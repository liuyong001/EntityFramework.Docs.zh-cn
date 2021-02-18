---
title: 测试使用 EF Core 的代码 - EF Core
description: 测试使用 Entity Framework Core 的应用程序的不同方法
author: ajcvickers
ms.date: 04/22/2020
uid: core/testing/index
ms.openlocfilehash: 5ac7954a87697e43b68845e925e2510c027b7135
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543570"
---
# <a name="testing-code-that-uses-ef-core"></a>测试使用 EF Core 的代码

测试访问数据库的代码需要满足以下任一条件：

* 针对在生产环境中使用的相同数据库系统运行查询和更新。
* 对其他更易管理的数据库系统运行查询和更新。
* 使用测试替身或其他机制来避免使用数据库。

本文档概述了每个选项所涉及的折衷方案，并说明了如何在每种方法中使用 EF Core。

> [!TIP]
> 请参阅 [EF Core 测试示例](xref:core/testing/testing-sample)查看演示此处引入的概念的代码。

## <a name="all-database-providers-are-not-equal"></a>所有数据库提供程序都不等同

务必要理解的是，EF Core 不是为了抽象底层数据库系统的各个方面而设计的。
EF Core 是一组通用的模式和概念，可用于任何数据库系统。
然后，EF Core 数据库提供程序在此通用框架的基础上叠加数据库特定的行为和功能。
这样，每个数据库系统都可以实现其最佳性能，并在适当的情况下维持与其他数据库系统的共性。

从根本上说，这意味着切换数据库提供程序将更改 EF Core 行为，使应用程序不能正常工作，除非它明确解释行为的任何差异。
虽然如此，但在许多情况下还是可以这么做，因为关系数据库之间存在高度的共性。
这有利也有弊。
其有利之处在于，可以相对轻松地切换数据库系统。
而其弊端在于，如果未针对新的数据库系统彻底测试应用程序，会有一种虚假的安全感。

## <a name="approach-1-production-database-system"></a>方法 1：生产数据库系统

如前一部分所述，要确保测试在生产中运行的内容，唯一的方法是使用同一个数据库系统。
例如，如果部署的应用程序使用 SQL Azure，则测试也应针对 SQL Azure 进行。

但是，要让每个开发者正在积极处理代码的同时针对 SQL Azure 运行测试，不仅工作速度慢而且代价高昂。
以下问题描述了这些方法中涉及的主要折衷方案：什么时候适合偏离生产数据库系统以提高测试效率？

幸运的是，在本例中，答案非常简单：使用本地 SQL Server 进行开发者测试。
SQL Azure 和 SQL Server 极其相似，因此针对 SQL Server 的测试通常是合理的折衷方案。
虽然如此，但在投入生产之前针对 SQL Azure 本身运行测试仍然是明智之举。

### <a name="localdb"></a>LocalDB

所有主要数据库系统都具有某种形式的“Developer Edition”本地测试。
SQL Server 还有一项名为 [LocalDB](/sql/database-engine/configure-windows/sql-server-express-localdb) 的功能。
LocalDB 的主要优势在于可以按需增大数据库实例。
这可以避免在计算机上运行数据库服务，即使在未运行测试时也是如此。

LocalDB 也不是没有问题：

* 它不支持 [SQL Server Developer Edition](/sql/sql-server/editions-and-components-of-sql-server-version-15?view=sql-server-ver15&preserve-view=true) 支持的部分内容。
* 它在 Linux 上不可用。
* 由于要启动服务，可能导致首次测试运行滞后。

就个人而言，我从来不觉得在开发计算机上运行数据库服务有什么问题，所以一般建议使用 Developer Edition。
但是，LocalDB 对某些人而言可能适用，尤其是在处理能力不够强的开发计算机上。

在 Docker 容器（或类似容器）中[运行 SQL Server](/sql/linux/quickstart-install-connect-docker)（或任何其他数据库系统）是避免直接在开发计算机上运行数据库系统的另一种方法。

## <a name="approach-2-sqlite"></a>方法 2：SQLite

EF Core 主要通过针对本地 SQL Server 实例运行 SQL Server 提供程序来对其进行测试。
这些测试将在几分钟内在一台高速计算机上运行数万次查询。
这说明，使用实际数据库系统可能是一种高性能的解决方案。
有一种错误的观念是，使用较小型的数据库是快速运行测试的唯一方法。

话虽如此，但如果由于某些原因，无法针对接近生产数据库系统的系统运行测试，应该怎么办？
下一个最佳选择是使用具有相似功能的对象。
这通常意味着另一个关系数据库，[SQLite](https://sqlite.org/index.html) 是显而易见的选择。

SQLite 是不错的选择，因为：

* 它在处理应用程序的过程中运行，因此开销较低。
* 它使用自动创建的简单数据库文件，因此不需要数据库管理。
* 它有内存模式，甚至可以避免创建文件。

但是，请记住：

* SQLite 不一定支持你的生产数据库系统支持的所有内容，这是不可避免的。
* 对于某些查询，SQLite 的行为方式将与你的生产数据库系统不同。

因此，如果使用 SQLite 进行测试，还应确保针对实际数据库系统进行测试。

有关 EF Core 特定指南，请参阅[用 SQLite 进行测试](xref:core/testing/sqlite)。

## <a name="approach-3-the-ef-core-in-memory-database"></a>方法 3：EF Core 内存中数据库

EF Core 附带了内存中数据库，用于对 EF Core 本身进行内部测试。
此数据库一般不适合用于测试使用 EF Core 的应用。 尤其是在下列情况下：

* 它不是关系数据库。
* 它不支持事务。
* 它无法运行原始 SQL 查询。
* 它未针对性能进行优化。

在测试 EF Core 内部机制时，这些都不重要，因为我们只在数据库与测试不相关时才会使用它。
另一方面，在测试使用 EF Core 的应用程序时，这些特性往往非常重要。

## <a name="unit-testing"></a>单元测试

考虑这种情况：你要测试可能需要使用数据库中的某些数据的业务逻辑，但该测试本身并不测试数据库交互。
可以选择使用[测试替身](https://en.wikipedia.org/wiki/Test_double)，例如模拟或虚构数据库。

我们使用测试替身进行 EF Core 的内部测试。
但是，我们不会尝试模拟 DbContext 或 IQueryable。
这样做难度大、过程繁琐且结果不可靠。
不要这样做。

改为使用 EF 内存中数据库对使用 DbContext 的应用进行单元测试。
在这种情况下，使用 EF 内存中数据库是适当的，因为测试不依赖于数据库行为。
但是不要这样测试实际数据库查询或更新。

[EF Core 测试示例](xref:core/testing/testing-sample)演示使用 EF 内存中数据库以及 SQL Server 和 SQLite 进行的测试。
