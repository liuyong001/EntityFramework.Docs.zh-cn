---
title: 事务 - EF Core
description: 在使用 Entity Framework Core 保存数据时管理事务的原子性
author: roji
ms.date: 9/26/2020
uid: core/saving/transactions
ms.openlocfilehash: b5e1fa2a0bcc466f22f03fee7ecaef9dcea1efaf
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003544"
---
# <a name="using-transactions"></a>使用事务

事务允许以原子方式处理多个数据库操作。 如果已提交事务，则所有操作都会成功应用到数据库。 如果已回滚事务，则所有操作都不会应用到数据库。

> [!TIP]
> 可在 GitHub 上查看此文章的[示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Saving/Transactions/)。

## <a name="default-transaction-behavior"></a>默认事务行为

默认情况下，如果数据库提供程序支持事务，则会在事务中应用对 `SaveChanges` 的单一调用中的所有更改。 如果其中有任何更改失败，则会回滚事务且所有更改都不会应用到数据库。 这意味着，`SaveChanges` 可保证完全成功，或在出现错误时不修改数据库。

对于大多数应用程序，此默认行为已足够。 如果应用程序要求被视为有必要，则应该仅手动控制事务。

## <a name="controlling-transactions"></a>控制事务

可以使用 `DbContext.Database` API 开始、提交和回滚事务。 以下示例显示了在单个事务中执行的两个 `SaveChanges` 操作以及一个 LINQ 查询：

[!code-csharp[Main](../../../samples/core/Saving/Transactions/ControllingTransaction.cs?name=Transaction&highlight=2,16-18)]

虽然所有关系数据库提供程序都支持事务，但在调用事务 API 时，可能会引发其他提供程序类型或不执行任何操作。

## <a name="savepoints"></a>保存点

> [!NOTE]
> EF Core 5.0 中已引入此功能。

如果调用 `SaveChanges` 且事务已在上下文中进行，则在保存任何数据之前，EF 会自动创建保存点。 保存点是数据库事务中的点，如果发生错误或出于任何其他原因，可能会回滚到这些点。 如果 `SaveChanges` 遇到任何错误，则会自动将事务回滚到保存点，使事务处于相同状态，就好像从未开始。 这样可以解决问题并重试保存，尤其是在出现[乐观并发](xref:core/saving/concurrency)问题时。

> [!WARNING]
> 保存点与 SQL Server 的多重活动结果集不兼容，因此不使用。 如果在 `SaveChanges`过程中出现错误，则该事务可能处于未知状态。

还可以手动管理保存点，就像管理事务一样。 以下示例在事务中创建保存点，并在失败时回滚到该保存点：

[!code-csharp[Main](../../../samples/core/Saving/Transactions/ManagingSavepoints.cs?name=Savepoints&highlight=9,19-20)]

## <a name="cross-context-transaction"></a>跨上下文事务

还可以跨多个上下文实例共享一个事务。 此功能仅在使用关系数据库提供程序时才可用，因为该提供程序需要使用特定于关系数据库的 `DbTransaction` 和 `DbConnection`。

若要共享事务，上下文必须共享 `DbConnection` 和 `DbTransaction`。

### <a name="allow-connection-to-be-externally-provided"></a>允许在外部提供连接

共享 `DbConnection` 需要在构造上下文时向其中传入连接的功能。

允许在外部提供 `DbConnection` 的最简单方式是，停止使用 `DbContext.OnConfiguring` 方法来配置上下文并在外部创建 `DbContextOptions`，然后将其传递到上下文构造函数。

> [!TIP]
> `DbContextOptionsBuilder` 是在 `DbContext.OnConfiguring` 中用于配置上下文的 API，现在即将在外部使用它来创建 `DbContextOptions`。

[!code-csharp[Main](../../../samples/core/Saving/Transactions/SharingTransaction.cs?name=Context&highlight=3,4,5)]

替代方法是继续使用 `DbContext.OnConfiguring`，但接受已保存并随后在 `DbContext.OnConfiguring` 中使用的 `DbConnection`。

```csharp
public class BloggingContext : DbContext
{
    private DbConnection _connection;

    public BloggingContext(DbConnection connection)
    {
      _connection = connection;
    }

    public DbSet<Blog> Blogs { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(_connection);
    }
}
```

### <a name="share-connection-and-transaction"></a>共享连接和事务

现在可以创建共享同一连接的多个上下文实例。 然后使用 `DbContext.Database.UseTransaction(DbTransaction)` API 在同一事务中登记两个上下文。

[!code-csharp[Main](../../../samples/core/Saving/Transactions/SharingTransaction.cs?name=Transaction&highlight=1-3,6,14,21-23)]

## <a name="using-external-dbtransactions-relational-databases-only"></a>使用外部 DbTransactions（仅限关系数据库）

如果使用多个数据访问技术来访问关系数据库，则可能希望在这些不同技术所执行的操作之间共享事务。

以下示例显示了如何在同一事务中执行 ADO.NET SqlClient 操作和 Entity Framework Core 操作。

[!code-csharp[Main](../../../samples/core/Saving/Transactions/ExternalDbTransaction.cs?name=Transaction&highlight=4,9,20,25-27)]

## <a name="using-systemtransactions"></a>使用 System.Transactions

如果需要跨较大作用域进行协调，则可以使用环境事务。

[!code-csharp[Main](../../../samples/core/Saving/Transactions/AmbientTransaction.cs?name=Transaction&highlight=1,2,3,26-28)]

还可以在显式事务中登记。

[!code-csharp[Main](../../../samples/core/Saving/Transactions/CommitableTransaction.cs?name=Transaction&highlight=1-2,15,28-30)]

### <a name="limitations-of-systemtransactions"></a>System.Transactions 的限制

1. EF Core 依赖数据库提供程序以实现对 System.Transactions 的支持。 如果提供程序未实现对 System.Transactions 的支持，则可能会完全忽略对这些 API 的调用。 SqlClient 支持它。

   > [!IMPORTANT]
   > 建议你测试在依赖提供程序以管理事务之前 API 与该提供程序的行为是否正确。 如果不正确，则建议你与数据库提供程序的维护人员联系。

2. 自 .NET Core 2.1 起，System.Transactions 实现不包括对分布式事务的支持，因此不能使用 `TransactionScope` 或 `CommittableTransaction` 来跨多个资源管理器协调事务。
