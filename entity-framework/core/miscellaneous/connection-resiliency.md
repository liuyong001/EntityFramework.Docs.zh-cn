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
# <a name="connection-resiliency"></a><span data-ttu-id="a0ce2-103">连接复原</span><span class="sxs-lookup"><span data-stu-id="a0ce2-103">Connection Resiliency</span></span>

<span data-ttu-id="a0ce2-104">连接复原会自动重试失败的数据库命令。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-104">Connection resiliency automatically retries failed database commands.</span></span> <span data-ttu-id="a0ce2-105">此功能可用于任何数据库，方法是提供 "执行策略"，它封装了检测失败和重试命令所需的逻辑。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-105">The feature can be used with any database by supplying an "execution strategy", which encapsulates the logic necessary to detect failures and retry commands.</span></span> <span data-ttu-id="a0ce2-106">EF Core 提供程序可以提供针对其特定数据库失败情况和最佳重试策略量身定制的执行策略。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-106">EF Core providers can supply execution strategies tailored to their specific database failure conditions and optimal retry policies.</span></span>

<span data-ttu-id="a0ce2-107">例如，SQL Server 提供程序包括专门针对 SQL Server (包含 SQL Azure) 的执行策略。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-107">As an example, the SQL Server provider includes an execution strategy that is specifically tailored to SQL Server (including SQL Azure).</span></span> <span data-ttu-id="a0ce2-108">它可以识别可重试的异常类型，并具有可用于最大重试次数、重试间隔时间等的合理默认值。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-108">It is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, etc.</span></span>

<span data-ttu-id="a0ce2-109">为上下文配置选项时，指定执行策略。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-109">An execution strategy is specified when configuring the options for your context.</span></span> <span data-ttu-id="a0ce2-110">这通常在 `OnConfiguring` 派生上下文的方法中：</span><span class="sxs-lookup"><span data-stu-id="a0ce2-110">This is typically in the `OnConfiguring` method of your derived context:</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#OnConfiguring)]

<span data-ttu-id="a0ce2-111">`Startup.cs`对于 ASP.NET Core 的应用程序，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="a0ce2-111">or in `Startup.cs` for an ASP.NET Core application:</span></span>

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
> <span data-ttu-id="a0ce2-112">启用 "失败时重试" 会导致 EF 内部缓冲结果集，这可能会显著提高返回大型结果集的查询的内存需求。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-112">Enabling retry on failure causes EF to internally buffer the resultset, which may significantly increase memory requirements for queries returning large resultsets.</span></span> <span data-ttu-id="a0ce2-113">有关更多详细信息，请参阅 [缓冲和流式处理](xref:core/performance/efficient-querying#buffering-and-streaming) 。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-113">See [buffering and streaming](xref:core/performance/efficient-querying#buffering-and-streaming) for more details.</span></span>

## <a name="custom-execution-strategy"></a><span data-ttu-id="a0ce2-114">自定义执行策略</span><span class="sxs-lookup"><span data-stu-id="a0ce2-114">Custom execution strategy</span></span>

<span data-ttu-id="a0ce2-115">如果要更改任何默认值，则可以使用一种机制来注册自己的自定义执行策略。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-115">There is a mechanism to register a custom execution strategy of your own if you wish to change any of the defaults.</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseMyProvider(
            "<connection string>",
            options => options.ExecutionStrategy(...));
}
```

## <a name="execution-strategies-and-transactions"></a><span data-ttu-id="a0ce2-116">执行策略和事务</span><span class="sxs-lookup"><span data-stu-id="a0ce2-116">Execution strategies and transactions</span></span>

<span data-ttu-id="a0ce2-117">在失败时自动重试的执行策略需要能够在失败的重试块中播放每个操作。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-117">An execution strategy that automatically retries on failures needs to be able to play back each operation in a retry block that fails.</span></span> <span data-ttu-id="a0ce2-118">启用重试后，通过 EF Core 执行的每个操作都会成为其自己的可重试操作。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-118">When retries are enabled, each operation you perform via EF Core becomes its own retriable operation.</span></span> <span data-ttu-id="a0ce2-119">也就是说，如果发生暂时性故障，每个查询和每个调用 `SaveChanges()` 都将作为一个单元重试。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-119">That is, each query and each call to `SaveChanges()` will be retried as a unit if a transient failure occurs.</span></span>

<span data-ttu-id="a0ce2-120">但是，如果您的代码使用 `BeginTransaction()` 您定义自己的一组操作，这些操作需要被视为一个单元，则需要播放该事务中的所有内容时，都将发生故障。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-120">However, if your code initiates a transaction using `BeginTransaction()` you are defining your own group of operations that need to be treated as a unit, and everything inside the transaction would need to be played back shall a failure occur.</span></span> <span data-ttu-id="a0ce2-121">如果在使用执行策略时尝试执行此操作，将收到如下所示的异常：</span><span class="sxs-lookup"><span data-stu-id="a0ce2-121">You will receive an exception like the following if you attempt to do this when using an execution strategy:</span></span>

> <span data-ttu-id="a0ce2-122">InvalidOperationException：配置的执行策略 "SqlServerRetryingExecutionStrategy" 不支持用户启动的事务。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-122">InvalidOperationException: The configured execution strategy 'SqlServerRetryingExecutionStrategy' does not support user initiated transactions.</span></span> <span data-ttu-id="a0ce2-123">使用由“DbContext.Database.CreateExecutionStrategy()”返回的执行策略执行事务（作为一个可回溯单元）中的所有操作。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-123">Use the execution strategy returned by 'DbContext.Database.CreateExecutionStrategy()' to execute all the operations in the transaction as a retriable unit.</span></span>

<span data-ttu-id="a0ce2-124">解决方案是使用代表需要执行的所有内容的委托手动调用执行策略。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-124">The solution is to manually invoke the execution strategy with a delegate representing everything that needs to be executed.</span></span> <span data-ttu-id="a0ce2-125">如果发生暂时性故障，执行策略会再次调用委托。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-125">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#ManualTransaction)]

<span data-ttu-id="a0ce2-126">此方法还可用于环境事务。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-126">This approach can also be used with ambient transactions.</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#AmbientTransaction)]

## <a name="transaction-commit-failure-and-the-idempotency-issue"></a><span data-ttu-id="a0ce2-127">事务提交失败和幂等性问题</span><span class="sxs-lookup"><span data-stu-id="a0ce2-127">Transaction commit failure and the idempotency issue</span></span>

<span data-ttu-id="a0ce2-128">通常，如果连接失败，当前事务将回滚。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-128">In general, when there is a connection failure the current transaction is rolled back.</span></span> <span data-ttu-id="a0ce2-129">但是，如果在提交事务时断开连接，则事务的生成状态是未知的。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-129">However, if the connection is dropped while the transaction is being committed the resulting state of the transaction is unknown.</span></span>

<span data-ttu-id="a0ce2-130">默认情况下，执行策略将重试该操作，就像事务已回滚一样，但如果不是这样，则这会导致在新数据库状态不兼容时导致异常，或者如果操作不依赖于特定状态（例如，使用自动生成的键值插入新行时），可能会导致 **数据损坏** 。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-130">By default, the execution strategy will retry the operation as if the transaction was rolled back, but if it's not the case this will result in an exception if the new database state is incompatible or could lead to **data corruption** if the operation does not rely on a particular state, for example when inserting a new row with auto-generated key values.</span></span>

<span data-ttu-id="a0ce2-131">可以通过多种方式来处理此情况。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-131">There are several ways to deal with this.</span></span>

### <a name="option-1---do-almost-nothing"></a><span data-ttu-id="a0ce2-132">选项 1- (几乎) 无</span><span class="sxs-lookup"><span data-stu-id="a0ce2-132">Option 1 - Do (almost) nothing</span></span>

<span data-ttu-id="a0ce2-133">在事务提交期间连接失败的可能性较低，因此，如果实际发生此情况，应用程序就会失败。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-133">The likelihood of a connection failure during transaction commit is low so it may be acceptable for your application to just fail if this condition actually occurs.</span></span>

<span data-ttu-id="a0ce2-134">但是，你需要避免使用存储生成的密钥，以确保引发了异常，而不是添加重复的行。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-134">However, you need to avoid using store-generated keys in order to ensure that an exception is thrown instead of adding a duplicate row.</span></span> <span data-ttu-id="a0ce2-135">请考虑使用客户端生成的 GUID 值或客户端值生成器。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-135">Consider using a client-generated GUID value or a client-side value generator.</span></span>

### <a name="option-2---rebuild-application-state"></a><span data-ttu-id="a0ce2-136">选项 2-重建应用程序状态</span><span class="sxs-lookup"><span data-stu-id="a0ce2-136">Option 2 - Rebuild application state</span></span>

1. <span data-ttu-id="a0ce2-137">放弃当前的 `DbContext` 。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-137">Discard the current `DbContext`.</span></span>
2. <span data-ttu-id="a0ce2-138">创建新的 `DbContext` 并从数据库还原应用程序的状态。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-138">Create a new `DbContext` and restore the state of your application from the database.</span></span>
3. <span data-ttu-id="a0ce2-139">通知用户上一次操作可能未成功完成。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-139">Inform the user that the last operation might not have been completed successfully.</span></span>

### <a name="option-3---add-state-verification"></a><span data-ttu-id="a0ce2-140">选项 3-添加状态验证</span><span class="sxs-lookup"><span data-stu-id="a0ce2-140">Option 3 - Add state verification</span></span>

<span data-ttu-id="a0ce2-141">对于大多数更改数据库状态的操作，可以添加代码来检查它是否成功。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-141">For most of the operations that change the database state it is possible to add code that checks whether it succeeded.</span></span> <span data-ttu-id="a0ce2-142">EF 提供扩展方法来简化此过程 `IExecutionStrategy.ExecuteInTransaction` 。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-142">EF provides an extension method to make this easier - `IExecutionStrategy.ExecuteInTransaction`.</span></span>

<span data-ttu-id="a0ce2-143">此方法会启动并提交事务，还会接受参数中的函数，该 `verifySucceeded` 参数在事务提交期间发生暂时性错误时被调用。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-143">This method begins and commits a transaction and also accepts a function in the `verifySucceeded` parameter that is invoked when a transient error occurs during the transaction commit.</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#Verification)]

> [!NOTE]
> <span data-ttu-id="a0ce2-144">在调用时，将 `SaveChanges` `acceptAllChangesOnSuccess` 设置为 `false` ，以避免在成功时将实体的状态更改 `Blog` 为 `Unchanged` `SaveChanges` 。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-144">Here `SaveChanges` is invoked with `acceptAllChangesOnSuccess` set to `false` to avoid changing the state of the `Blog` entity to `Unchanged` if `SaveChanges` succeeds.</span></span> <span data-ttu-id="a0ce2-145">如果提交失败并且事务已回滚，则允许重试相同的操作。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-145">This allows to retry the same operation if the commit fails and the transaction is rolled back.</span></span>

### <a name="option-4---manually-track-the-transaction"></a><span data-ttu-id="a0ce2-146">选项 4-手动跟踪事务</span><span class="sxs-lookup"><span data-stu-id="a0ce2-146">Option 4 - Manually track the transaction</span></span>

<span data-ttu-id="a0ce2-147">如果需要使用存储生成的密钥，或者需要一种常规方法来处理不依赖于执行的操作的提交失败，则可以为其分配一个在提交失败时检查的 ID。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-147">If you need to use store-generated keys or need a generic way of handling commit failures that doesn't depend on the operation performed each transaction could be assigned an ID that is checked when the commit fails.</span></span>

1. <span data-ttu-id="a0ce2-148">向数据库添加一个用于跟踪事务状态的表。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-148">Add a table to the database used to track the status of the transactions.</span></span>
2. <span data-ttu-id="a0ce2-149">在每个事务开头的表中插入一行。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-149">Insert a row into the table at the beginning of each transaction.</span></span>
3. <span data-ttu-id="a0ce2-150">如果在提交期间连接失败，请检查数据库中是否存在相应的行。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-150">If the connection fails during the commit, check for the presence of the corresponding row in the database.</span></span>
4. <span data-ttu-id="a0ce2-151">如果提交成功，则删除相应的行以避免表增长。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-151">If the commit is successful, delete the corresponding row to avoid the growth of the table.</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#Tracking)]

> [!NOTE]
> <span data-ttu-id="a0ce2-152">请确保用于验证的上下文具有定义的执行策略，因为如果在事务提交过程中失败，连接可能会在验证期间再次失败。</span><span class="sxs-lookup"><span data-stu-id="a0ce2-152">Make sure that the context used for the verification has an execution strategy defined as the connection is likely to fail again during verification if it failed during transaction commit.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="a0ce2-153">其他资源</span><span class="sxs-lookup"><span data-stu-id="a0ce2-153">Additional resources</span></span>

* [<span data-ttu-id="a0ce2-154">排查 Azure SQL 数据库和 SQL 托管实例中的暂时性连接错误</span><span class="sxs-lookup"><span data-stu-id="a0ce2-154">Troubleshoot transient connection errors in Azure SQL Database and SQL Managed Instance</span></span>](/azure/azure-sql/database/troubleshoot-common-connectivity-issues)
