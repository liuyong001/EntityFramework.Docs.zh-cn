---
title: 在测试之间共享数据库-EF Core
description: 示例演示如何在多个测试之间共享数据库
author: ajcvickers
ms.date: 04/25/2020
uid: core/testing/sharing-databases
ms.openlocfilehash: 7a90a144271d5c34e9d5043aa439f84db805c6af
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128831"
---
# <a name="sharing-databases-between-tests"></a>在测试之间共享数据库

[EF Core 测试示例](xref:core/testing/testing-sample)演示了如何针对不同的数据库系统测试应用程序。
对于该示例，每个测试都会创建一个新数据库。
当使用 SQLite 或 EF 内存中数据库时，这是一个很好的模式，但在使用其他数据库系统时，这种模式可能会造成很大的开销。

此示例通过将数据库创建移到测试装置中来构建前面的示例。
这允许创建单个 SQL Server 数据库，并只为所有测试创建一次。

> [!TIP]
> 在继续操作之前，请务必完成 [EF Core 测试示例](xref:core/testing/testing-sample) 。

为同一个数据库编写多个测试并不困难。
这一技巧以测试在运行时不会相互行程的方式进行操作。
这需要了解：

* 如何在测试之间安全共享对象
* 当测试框架并行运行测试时
* 如何使数据库处于每个测试的干净状态

## <a name="the-fixture"></a>装置

我们将使用测试装置在测试之间共享对象。
当你想要创建单个测试上下文并将其共享到类中的所有测试中并在类中的所有测试都完成后， [XUnit 文档](https://xunit.net/docs/shared-context.html) 将指出应使用的装置。 "

> [!TIP]
> 此示例使用的是 [XUnit](https://xunit.net/)，但其他测试框架中也存在类似的概念，包括 [NUnit](https://nunit.org/)。

这意味着我们需要将数据库创建和种子设定转移到夹具类。
它的外观如下所示：

[!code-csharp[SharedDatabaseFixture](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/SharedDatabaseTests/SharedDatabaseFixture.cs?name=SharedDatabaseFixture)]

现在，请注意构造函数的方式：

* 在装置的生存期内创建单个数据库连接
* 通过调用方法创建并设定该数据库的种子 `Seed`

立即忽略锁定;稍后我们将返回到它。

> [!TIP]
> 创建和播种代码不需要是异步的。
> 将其设为异步会使代码变得复杂，而不会提高测试的性能或吞吐量。

数据库是通过先删除所有现有数据库，然后创建新的数据库创建的。
这可确保数据库与当前 EF 模型匹配，即使自上次测试运行以来已更改。

> [!TIP]
> 使用 [respawn](https://jimmybogard.com/tag/respawn/) 之类的内容（而不是每次重新创建），可以更快地 "清理" 现有数据库。
> 但是，在执行此操作时，必须注意确保数据库架构与 EF 模型保持最新。

释放装置后，数据库连接会被释放。
此时，您还可以考虑删除测试数据库。
但是，如果多个测试类正在共享该装置，则这将需要额外的锁定和引用计数。
此外，使测试数据库仍可用于调试失败的测试通常是很有用的。

## <a name="using-the-fixture"></a>使用装置

XUnit 具有一个通用模式，用于将测试装置与一类测试进行关联：

[!code-csharp[UsingTheFixture](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/SharedDatabaseTests/SharedDatabaseTest.cs?name=UsingTheFixture)]

XUnit 现在将创建单个夹具实例，并将其传递给测试类的每个实例。
 (请 [注意，XUnit](xref:core/testing/testing-sample) 将在每次运行测试时创建新的测试类实例。 ) 这意味着将创建数据库并对其进行种子设定，然后每个测试都将使用此数据库。

请注意，不会并行运行单个类中的测试。
这意味着，即使 `DbConnection` 对象不是线程安全的，每个测试都可以安全地使用同一个数据库连接。

## <a name="maintaining-database-state"></a>维护数据库状态

测试通常需要对测试数据进行插入、更新和删除操作。
但这些更改会影响到需要干净的种子数据库的其他测试。

这可以通过在事务中运行转变测试来处理。
例如：

[!code-csharp[CanAddItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/SharedDatabaseTests/SharedDatabaseTest.cs?name=CanAddItem)]

请注意，在测试完成后将创建事务，并将其释放。
释放事务将导致回滚该事务，因此其他测试将看不到任何更改。

用于创建上下文的帮助器方法 (参阅上述装置代码) 接受此交易并使用它来 DbContext。

## <a name="sharing-the-fixture"></a>共享装置

您可能已注意到锁定了有关数据库创建和播种的代码。
这不是此示例所必需的，因为只有一类测试使用装置，因此仅创建单个装置实例。

但是，你可能想要将相同的装置用于多个测试类别。
XUnit 将为这些类中的每个类创建一个装置实例。
并行运行测试的不同线程可能会使用这些方法。
因此，请务必使用适当的锁定，以确保只有一个线程可以创建和播种数据库。

> [!TIP]
> 这里简单明了 `lock` 。
> 无需尝试更复杂的操作，如任何无锁模式。
