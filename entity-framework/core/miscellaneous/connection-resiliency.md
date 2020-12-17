---
title: 连接复原-EF Core
description: 使用连接复原功能通过 Entity Framework Core 自动重试失败的命令
author: AndriySvyryd
ms.date: 11/15/2016
uid: core/miscellaneous/connection-resiliency
ms.openlocfilehash: bcafdf5de26ecfd7539d426388154550a39332ab
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97635778"
---
# <a name="connection-resiliency"></a>连接复原

连接复原会自动重试失败的数据库命令。 此功能可用于任何数据库，方法是提供 "执行策略"，它封装了检测失败和重试命令所需的逻辑。 EF Core 提供程序可以提供针对其特定数据库失败情况和最佳重试策略量身定制的执行策略。

例如，SQL Server 提供程序包括专门针对 SQL Server (包含 SQL Azure) 的执行策略。 它可以识别可重试的异常类型，并具有可用于最大重试次数、重试间隔时间等的合理默认值。

为上下文配置选项时，指定执行策略。 这通常在 `OnConfiguring` 派生上下文的方法中：

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#OnConfiguring)]

`Startup.cs`对于 ASP.NET Core 的应用程序，请执行以下操作：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<PicnicContext>(
        options => options.UseSqlServer(
            "<connection string>",
            providerOptions => providerOptions.EnableRetryOnFailure()));
}
```

> [!NOTE]
> 启用 "失败时重试" 会导致 EF 内部缓冲结果集，这可能会显著提高返回大型结果集的查询的内存需求。 有关更多详细信息，请参阅 [缓冲和流式处理](xref:core/performance/efficient-querying#buffering-and-streaming) 。

## <a name="custom-execution-strategy"></a>自定义执行策略

如果要更改任何默认值，则可以使用一种机制来注册自己的自定义执行策略。

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseMyProvider(
            "<connection string>",
            options => options.ExecutionStrategy(...));
}
```

## <a name="execution-strategies-and-transactions"></a>执行策略和事务

在失败时自动重试的执行策略需要能够在失败的重试块中播放每个操作。 启用重试后，通过 EF Core 执行的每个操作都会成为其自己的可重试操作。 也就是说，如果发生暂时性故障，每个查询和每个调用 `SaveChanges()` 都将作为一个单元重试。

但是，如果您的代码使用 `BeginTransaction()` 您定义自己的一组操作，这些操作需要被视为一个单元，则需要播放该事务中的所有内容时，都将发生故障。 如果在使用执行策略时尝试执行此操作，将收到如下所示的异常：

> InvalidOperationException：配置的执行策略 "SqlServerRetryingExecutionStrategy" 不支持用户启动的事务。 使用由“DbContext.Database.CreateExecutionStrategy()”返回的执行策略执行事务（作为一个可回溯单元）中的所有操作。

解决方案是使用代表需要执行的所有内容的委托手动调用执行策略。 如果发生暂时性故障，执行策略会再次调用委托。

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#ManualTransaction)]

此方法还可用于环境事务。

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#AmbientTransaction)]

## <a name="transaction-commit-failure-and-the-idempotency-issue"></a>事务提交失败和幂等性问题

通常，如果连接失败，当前事务将回滚。 但是，如果在提交事务时断开连接，则事务的生成状态是未知的。

默认情况下，执行策略将重试该操作，就像事务已回滚一样，但如果不是这样，则这会导致在新数据库状态不兼容时导致异常，或者如果操作不依赖于特定状态（例如，使用自动生成的键值插入新行时），可能会导致 **数据损坏** 。

可以通过多种方式来处理此情况。

### <a name="option-1---do-almost-nothing"></a>选项 1- (几乎) 无

在事务提交期间连接失败的可能性较低，因此，如果实际发生此情况，应用程序就会失败。

但是，你需要避免使用存储生成的密钥，以确保引发了异常，而不是添加重复的行。 请考虑使用客户端生成的 GUID 值或客户端值生成器。

### <a name="option-2---rebuild-application-state"></a>选项 2-重建应用程序状态

1. 放弃当前的 `DbContext` 。
2. 创建新的 `DbContext` 并从数据库还原应用程序的状态。
3. 通知用户上一次操作可能未成功完成。

### <a name="option-3---add-state-verification"></a>选项 3-添加状态验证

对于大多数更改数据库状态的操作，可以添加代码来检查它是否成功。 EF 提供扩展方法来简化此过程 `IExecutionStrategy.ExecuteInTransaction` 。

此方法会启动并提交事务，还会接受参数中的函数，该 `verifySucceeded` 参数在事务提交期间发生暂时性错误时被调用。

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#Verification)]

> [!NOTE]
> 在调用时，将 `SaveChanges` `acceptAllChangesOnSuccess` 设置为 `false` ，以避免在成功时将实体的状态更改 `Blog` 为 `Unchanged` `SaveChanges` 。 如果提交失败并且事务已回滚，则允许重试相同的操作。

### <a name="option-4---manually-track-the-transaction"></a>选项 4-手动跟踪事务

如果需要使用存储生成的密钥，或者需要一种常规方法来处理不依赖于执行的操作的提交失败，则可以为其分配一个在提交失败时检查的 ID。

1. 向数据库添加一个用于跟踪事务状态的表。
2. 在每个事务开头的表中插入一行。
3. 如果在提交期间连接失败，请检查数据库中是否存在相应的行。
4. 如果提交成功，则删除相应的行以避免表增长。

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#Tracking)]

> [!NOTE]
> 请确保用于验证的上下文具有定义的执行策略，因为如果在事务提交过程中失败，连接可能会在验证期间再次失败。

## <a name="additional-resources"></a>其他资源

* [排查 Azure SQL 数据库和 SQL 托管实例中的暂时性连接错误](/azure/azure-sql/database/troubleshoot-common-connectivity-issues)
