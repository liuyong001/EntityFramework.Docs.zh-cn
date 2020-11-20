---
title: 拦截程序-EF Core
description: 截获数据库操作和其他事件
author: ajcvickers
ms.date: 10/08/2020
uid: core/logging-events-diagnostics/interceptors
ms.openlocfilehash: 22d860a083c5ece9be109be630c3ce01dd742bf2
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003401"
---
# <a name="interceptors"></a><span data-ttu-id="143d6-103">侦听器</span><span class="sxs-lookup"><span data-stu-id="143d6-103">Interceptors</span></span>

<span data-ttu-id="143d6-104">Entity Framework Core (EF Core) 拦截允许拦截、修改和/或禁止 EF Core 操作。</span><span class="sxs-lookup"><span data-stu-id="143d6-104">Entity Framework Core (EF Core) interceptors enable interception, modification, and/or suppression of EF Core operations.</span></span> <span data-ttu-id="143d6-105">这包括低级数据库操作（例如执行命令）以及高级别操作（例如对 SaveChanges 的调用）。</span><span class="sxs-lookup"><span data-stu-id="143d6-105">This includes low-level database operations such as executing a command, as well as higher-level operations, such as calls to SaveChanges.</span></span>

<span data-ttu-id="143d6-106">侦听器与日志记录和诊断的不同之处在于，它们允许修改或抑制所侦听的操作。</span><span class="sxs-lookup"><span data-stu-id="143d6-106">Interceptors are different from logging and diagnostics in that they allow modification or suppression of the operation being intercepted.</span></span> <span data-ttu-id="143d6-107">[简单日志记录](xref:core/logging-events-diagnostics/simple-logging)或 [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) 是日志记录的更好选择。</span><span class="sxs-lookup"><span data-stu-id="143d6-107">[Simple logging](xref:core/logging-events-diagnostics/simple-logging) or [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) are better choices for logging.</span></span>

<span data-ttu-id="143d6-108">配置上下文时，将按 DbContext 实例注册侦听器。</span><span class="sxs-lookup"><span data-stu-id="143d6-108">Interceptors are registered per DbContext instance when the context is configured.</span></span> <span data-ttu-id="143d6-109">使用[诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)获取相同的信息，但要获取进程中所有 DbContext 实例的信息。</span><span class="sxs-lookup"><span data-stu-id="143d6-109">Use a [diagnostic listener](xref:core/logging-events-diagnostics/diagnostic-listeners) to get the same information but for all DbContext instances in the process.</span></span>

## <a name="registering-interceptors"></a><span data-ttu-id="143d6-110">正在注册侦听器</span><span class="sxs-lookup"><span data-stu-id="143d6-110">Registering interceptors</span></span>

<span data-ttu-id="143d6-111"><xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.AddInterceptors%2A>在[配置 DbContext 实例](xref:core/dbcontext-configuration/index)时，将使用注册侦听器。</span><span class="sxs-lookup"><span data-stu-id="143d6-111">Interceptors are registered using <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.AddInterceptors%2A> when [configuring a DbContext instance](xref:core/dbcontext-configuration/index).</span></span> <span data-ttu-id="143d6-112">通常在重写中完成此操作 <xref:Microsoft.EntityFrameworkCore.DbContext.OnConfiguring%2A?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="143d6-112">This is commonly done in an override of <xref:Microsoft.EntityFrameworkCore.DbContext.OnConfiguring%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="143d6-113">例如：</span><span class="sxs-lookup"><span data-stu-id="143d6-113">For example:</span></span>

<!--
public class ExampleContext : BlogsContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder) 
        => optionsBuilder.AddInterceptors(new TaggedQueryCommandInterceptor());
}
-->
[!code-csharp[RegisterInterceptor](../../../samples/core/Miscellaneous/CommandInterception/Program.cs?name=RegisterInterceptor)]

<span data-ttu-id="143d6-114">另外， `AddInterceptors` 还可以在 <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> 创建 <xref:Microsoft.EntityFrameworkCore.DbContextOptions> 要传递到 DbContext 构造函数的实例时作为或的一部分来调用。</span><span class="sxs-lookup"><span data-stu-id="143d6-114">Alternately, `AddInterceptors` can be called as part of <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> or when creating a <xref:Microsoft.EntityFrameworkCore.DbContextOptions> instance to pass to the DbContext constructor.</span></span>

> [!TIP]
> <span data-ttu-id="143d6-115">当使用 AddDbContext 或将 DbContextOptions 实例传递到 DbContext 构造函数时，仍将调用 OnConfiguring。</span><span class="sxs-lookup"><span data-stu-id="143d6-115">OnConfiguring is still called when AddDbContext is used or a DbContextOptions instance is passed to the DbContext constructor.</span></span> <span data-ttu-id="143d6-116">这使得它成为应用上下文配置的理想位置，而不考虑如何构造 DbContext。</span><span class="sxs-lookup"><span data-stu-id="143d6-116">This makes it the ideal place to apply context configuration regardless of how the DbContext is constructed.</span></span>

<span data-ttu-id="143d6-117">拦截程序通常是无状态的，这意味着单个侦听器实例可用于所有 DbContext 实例。</span><span class="sxs-lookup"><span data-stu-id="143d6-117">Interceptors are often stateless, which means that a single interceptor instance can be used for all DbContext instances.</span></span> <span data-ttu-id="143d6-118">例如：</span><span class="sxs-lookup"><span data-stu-id="143d6-118">For example:</span></span>

<!--
public class TaggedQueryCommandInterceptorContext : BlogsContext
{
    private static readonly TaggedQueryCommandInterceptor _interceptor 
        = new TaggedQueryCommandInterceptor();

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder) 
        => optionsBuilder.AddInterceptors(_interceptor);
}
-->
[!code-csharp[RegisterStatelessInterceptor](../../../samples/core/Miscellaneous/CommandInterception/Program.cs?name=RegisterStatelessInterceptor)]

<span data-ttu-id="143d6-119">每个侦听器实例都必须实现一个或多个派生自的接口 <xref:Microsoft.EntityFrameworkCore.Diagnostics.IInterceptor> 。</span><span class="sxs-lookup"><span data-stu-id="143d6-119">Every interceptor instance must implement one or more interface derived from <xref:Microsoft.EntityFrameworkCore.Diagnostics.IInterceptor>.</span></span> <span data-ttu-id="143d6-120">每个实例只能注册一次，即使它实现了多个截获接口;EF Core 将根据需要为每个接口路由事件。</span><span class="sxs-lookup"><span data-stu-id="143d6-120">Each instance should only be registered once even if it implements multiple interception interfaces; EF Core will route events for each interface as appropriate.</span></span>

## <a name="database-interception"></a><span data-ttu-id="143d6-121">数据库拦截</span><span class="sxs-lookup"><span data-stu-id="143d6-121">Database interception</span></span>

> [!NOTE]
> <span data-ttu-id="143d6-122">数据库拦截是在 EF Core 3.0 中引入的，并且仅适用于关系数据库提供程序。</span><span class="sxs-lookup"><span data-stu-id="143d6-122">Database interception was introduced in EF Core 3.0 and is only available for relational database providers.</span></span>
> <span data-ttu-id="143d6-123">EF Core 5.0 中引入了保存点支持。</span><span class="sxs-lookup"><span data-stu-id="143d6-123">Savepoint support was introduced in EF Core 5.0.</span></span>

<span data-ttu-id="143d6-124">低级别数据库截取拆分为下表中显示的三个接口。</span><span class="sxs-lookup"><span data-stu-id="143d6-124">Low-level database interception is split into the three interfaces shown in the following table.</span></span>

| <span data-ttu-id="143d6-125">截获</span><span class="sxs-lookup"><span data-stu-id="143d6-125">Interceptor</span></span>                                                            | <span data-ttu-id="143d6-126">已截获数据库操作</span><span class="sxs-lookup"><span data-stu-id="143d6-126">Database operations intercepted</span></span>
|:-----------------------------------------------------------------------|-------------------------------------------------
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbCommandInterceptor> | <span data-ttu-id="143d6-127">创建命令</span><span class="sxs-lookup"><span data-stu-id="143d6-127">Creating commands</span></span></br><span data-ttu-id="143d6-128">执行命令</span><span class="sxs-lookup"><span data-stu-id="143d6-128">Executing commands</span></span></br><span data-ttu-id="143d6-129">命令失败</span><span class="sxs-lookup"><span data-stu-id="143d6-129">Command failures</span></span></br><span data-ttu-id="143d6-130">释放命令的 DbDataReader</span><span class="sxs-lookup"><span data-stu-id="143d6-130">Disposing the command's DbDataReader</span></span>
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbConnectionInterceptor> | <span data-ttu-id="143d6-131">打开和关闭连接</span><span class="sxs-lookup"><span data-stu-id="143d6-131">Opening and closing connections</span></span></br><span data-ttu-id="143d6-132">连接失败</span><span class="sxs-lookup"><span data-stu-id="143d6-132">Connection failures</span></span>
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbTransactionInterceptor> | <span data-ttu-id="143d6-133">创建事务</span><span class="sxs-lookup"><span data-stu-id="143d6-133">Creating transactions</span></span></br><span data-ttu-id="143d6-134">使用现有事务</span><span class="sxs-lookup"><span data-stu-id="143d6-134">Using existing transactions</span></span></br><span data-ttu-id="143d6-135">提交事务</span><span class="sxs-lookup"><span data-stu-id="143d6-135">Committing transactions</span></span></br><span data-ttu-id="143d6-136">回退事务</span><span class="sxs-lookup"><span data-stu-id="143d6-136">Rolling back transactions</span></span></br><span data-ttu-id="143d6-137">创建和使用保存点</span><span class="sxs-lookup"><span data-stu-id="143d6-137">Creating and using savepoints</span></span></br><span data-ttu-id="143d6-138">事务失败</span><span class="sxs-lookup"><span data-stu-id="143d6-138">Transaction failures</span></span>

<span data-ttu-id="143d6-139">基类 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor> 、 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbConnectionInterceptor> 和 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbTransactionInterceptor> 包含对应接口中每个方法的无操作实现。</span><span class="sxs-lookup"><span data-stu-id="143d6-139">The base classes <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor>, <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbConnectionInterceptor>, and <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbTransactionInterceptor> contain no-op implementations for each method in the corresponding interface.</span></span> <span data-ttu-id="143d6-140">使用基类，以避免实现未使用的截取方法。</span><span class="sxs-lookup"><span data-stu-id="143d6-140">Use the base classes to avoid the need to implement unused interception methods.</span></span>

<span data-ttu-id="143d6-141">每个侦听器类型上的方法都成对出现，第一种方法是在数据库操作开始之前调用，第二个是在操作完成后调用。</span><span class="sxs-lookup"><span data-stu-id="143d6-141">The methods on each interceptor type come in pairs, with the first being called before the database operation is started, and the second after the operation has completed.</span></span> <span data-ttu-id="143d6-142">例如，</span><span class="sxs-lookup"><span data-stu-id="143d6-142">For example.</span></span> <span data-ttu-id="143d6-143">例如，在 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor.ReaderExecuting%2A?displayProperty=nameWithType> 执行查询之前调用，并 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor.ReaderExecuted%2A?displayProperty=nameWithType> 在将查询发送到数据库后调用。</span><span class="sxs-lookup"><span data-stu-id="143d6-143">For example, <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor.ReaderExecuting%2A?displayProperty=nameWithType> is called before a query is executed, and <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor.ReaderExecuted%2A?displayProperty=nameWithType> is called after query has been sent to the database.</span></span>

<span data-ttu-id="143d6-144">每对方法都有同步和异步变体。</span><span class="sxs-lookup"><span data-stu-id="143d6-144">Each pair of methods have both sync and async variations.</span></span> <span data-ttu-id="143d6-145">这允许异步 i/o，如请求访问令牌，作为截获异步数据库操作的一部分发生。</span><span class="sxs-lookup"><span data-stu-id="143d6-145">This allows for asynchronous I/O, such as requesting an access token, to happen as part of intercepting an async database operation.</span></span>

### <a name="example-command-interception-to-add-query-hints"></a><span data-ttu-id="143d6-146">示例：命令侦听以添加查询提示</span><span class="sxs-lookup"><span data-stu-id="143d6-146">Example: Command interception to add query hints</span></span>

> [!TIP]  
> <span data-ttu-id="143d6-147">可以从 GitHub [下载命令侦听器示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/CommandInterception) 。</span><span class="sxs-lookup"><span data-stu-id="143d6-147">You can [download the command interceptor sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/CommandInterception) from GitHub.</span></span>

<span data-ttu-id="143d6-148">在将 <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbCommandInterceptor> SQL 发送到数据库之前，可以使用对其进行修改。</span><span class="sxs-lookup"><span data-stu-id="143d6-148">An <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbCommandInterceptor> can be used to modify SQL before it is sent to the database.</span></span> <span data-ttu-id="143d6-149">此示例演示如何修改 SQL 以包含查询提示。</span><span class="sxs-lookup"><span data-stu-id="143d6-149">This example shows how to modify the SQL to include a query hint.</span></span>

<span data-ttu-id="143d6-150">通常，截获的最棘手部分是确定命令与需要修改的查询的对应情况。</span><span class="sxs-lookup"><span data-stu-id="143d6-150">Often, the trickiest part of the interception is determining when the command corresponds to the query that needs to be modified.</span></span> <span data-ttu-id="143d6-151">分析 SQL 是一种方法，但这种做法往往非常脆弱。</span><span class="sxs-lookup"><span data-stu-id="143d6-151">Parsing the SQL is one option, but tends to be fragile.</span></span> <span data-ttu-id="143d6-152">另一种方法是使用 [EF Core 查询标记](xref:core/querying/tags) 来标记每个应修改的查询。</span><span class="sxs-lookup"><span data-stu-id="143d6-152">Another option is to use [EF Core query tags](xref:core/querying/tags) to tag each query that should be modified.</span></span> <span data-ttu-id="143d6-153">例如：</span><span class="sxs-lookup"><span data-stu-id="143d6-153">For example:</span></span>

<!--
            var blogs1 = context.Blogs.TagWith("Use hint: robust plan").ToList();
-->
[!code-csharp[TaggedQuery](../../../samples/core/Miscellaneous/CommandInterception/Program.cs?name=TaggedQuery)]

<span data-ttu-id="143d6-154">然后，可在侦听器中检测到此标记，因为它将始终作为注释包含在命令文本的第一行中。</span><span class="sxs-lookup"><span data-stu-id="143d6-154">This tag can then be detected in the interceptor as it will always be included as a comment in the first line of the command text.</span></span> <span data-ttu-id="143d6-155">检测到标记时，会修改查询 SQL 以添加相应的提示：</span><span class="sxs-lookup"><span data-stu-id="143d6-155">On detecting the tag, the query SQL is modified to add the appropriate hint:</span></span>

<!--
public class TaggedQueryCommandInterceptor : DbCommandInterceptor
{
    public override InterceptionResult<DbDataReader> ReaderExecuting(
        DbCommand command,
        CommandEventData eventData,
        InterceptionResult<DbDataReader> result)
    {
        ManipulateCommand(command);

        return result;
    }

    public override ValueTask<InterceptionResult<DbDataReader>> ReaderExecutingAsync(
        DbCommand command,
        CommandEventData eventData,
        InterceptionResult<DbDataReader> result,
        CancellationToken cancellationToken = default)
    {
        ManipulateCommand(command);

        return new ValueTask<InterceptionResult<DbDataReader>>(result);
    }

    private static void ManipulateCommand(DbCommand command)
    {
        if (command.CommandText.StartsWith("-- Use hint: robust plan", StringComparison.Ordinal))
        {
            command.CommandText += " OPTION (ROBUST PLAN)";
        }
    }
}
-->
[!code-csharp[TaggedQueryCommandInterceptor](../../../samples/core/Miscellaneous/CommandInterception/TaggedQueryCommandInterceptor.cs?name=TaggedQueryCommandInterceptor)]

<span data-ttu-id="143d6-156">注意：</span><span class="sxs-lookup"><span data-stu-id="143d6-156">Notice:</span></span>

* <span data-ttu-id="143d6-157">侦听器从继承 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor> ，以避免必须实现侦听器接口中的每个方法。</span><span class="sxs-lookup"><span data-stu-id="143d6-157">The interceptor inherits from <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor> to avoid having to implement every method in the interceptor interface.</span></span>
* <span data-ttu-id="143d6-158">侦听器同时实现同步和异步方法。</span><span class="sxs-lookup"><span data-stu-id="143d6-158">The interceptor implements both sync and async methods.</span></span> <span data-ttu-id="143d6-159">这可确保将相同的查询提示应用于同步和异步查询。</span><span class="sxs-lookup"><span data-stu-id="143d6-159">This ensures that the same query hint is applied to sync and async queries.</span></span>
* <span data-ttu-id="143d6-160">侦听器通过在将 `Executing` 生成的 SQL 发送到数据库 _之前_ ，实现 EF Core 调用的方法。</span><span class="sxs-lookup"><span data-stu-id="143d6-160">The interceptor implements the `Executing` methods which are called by EF Core with the generated SQL _before_ it is sent to the database.</span></span> <span data-ttu-id="143d6-161">与 `Executed` 方法相比，在数据库调用返回之后调用这些方法。</span><span class="sxs-lookup"><span data-stu-id="143d6-161">Contrast this with the `Executed` methods, which are called after the database call has returned.</span></span>

<span data-ttu-id="143d6-162">在此示例中运行代码将在标记查询时生成以下内容：</span><span class="sxs-lookup"><span data-stu-id="143d6-162">Running the code in this example generates the following when a query is tagged:</span></span>

```sql
-- Use hint: robust plan

SELECT [b].[Id], [b].[Name]
FROM [Blogs] AS [b] OPTION (ROBUST PLAN)
```

<span data-ttu-id="143d6-163">另一方面，如果查询未标记，则会将其发送到未修改的数据库：</span><span class="sxs-lookup"><span data-stu-id="143d6-163">On the other hand, when a query is not tagged, then it is sent to the database unmodified:</span></span>

```sql
SELECT [b].[Id], [b].[Name]
FROM [Blogs] AS [b]
```

### <a name="example-connection-interception-for-sql-azure-authentication-using-add"></a><span data-ttu-id="143d6-164">示例：使用 ADD 进行 SQL Azure 身份验证的连接拦截</span><span class="sxs-lookup"><span data-stu-id="143d6-164">Example: Connection interception for SQL Azure authentication using ADD</span></span>

> [!TIP]  
> <span data-ttu-id="143d6-165">可以从 GitHub [下载连接侦听器示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/ConnectionInterception) 。</span><span class="sxs-lookup"><span data-stu-id="143d6-165">You can [download the connection interceptor sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/ConnectionInterception) from GitHub.</span></span>

<span data-ttu-id="143d6-166">在 <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbConnectionInterceptor> <xref:System.Data.Common.DbConnection> 用于连接到数据库之前，可以使用来操作。</span><span class="sxs-lookup"><span data-stu-id="143d6-166">An <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbConnectionInterceptor> can be used to manipulate the <xref:System.Data.Common.DbConnection> before it is used to connect to the database.</span></span> <span data-ttu-id="143d6-167">这可用于获取 AAD) 访问令牌 (Azure Active Directory。</span><span class="sxs-lookup"><span data-stu-id="143d6-167">This can be used to obtain an Azure Active Directory (AAD) access token.</span></span> <span data-ttu-id="143d6-168">例如：</span><span class="sxs-lookup"><span data-stu-id="143d6-168">For example:</span></span>

<!--
public class AadAuthenticationInterceptor : DbConnectionInterceptor
{
    public override InterceptionResult ConnectionOpening(
        DbConnection connection,
        ConnectionEventData eventData,
        InterceptionResult result)
        => throw new InvalidOperationException("Open connections asynchronously when using AAD authentication.");

    public override async ValueTask<InterceptionResult> ConnectionOpeningAsync(
        DbConnection connection,
        ConnectionEventData eventData,
        InterceptionResult result,
        CancellationToken cancellationToken = default)
    {
        var sqlConnection = (SqlConnection)connection;

        var provider = new AzureServiceTokenProvider();
        // Note: in some situations the access token may not be cached automatically the Azure Token Provider.
        // Depending on the kind of token requested, you may need to implement your own caching here.
        sqlConnection.AccessToken = await provider.GetAccessTokenAsync("https://database.windows.net/", null, cancellationToken);

        return result;
    }
}
-->
[!code-csharp[AadAuthenticationInterceptor](../../../samples/core/Miscellaneous/ConnectionInterception/AadAuthenticationInterceptor.cs?name=AadAuthenticationInterceptor)]

> [!TIP]
> <span data-ttu-id="143d6-169">[SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient/) 现在支持通过连接字符串进行 AAD 身份验证。</span><span class="sxs-lookup"><span data-stu-id="143d6-169">[Microsoft.Data.SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient/) now supports AAD authentication via connection string.</span></span> <span data-ttu-id="143d6-170">有关更多信息，请参见<xref:Microsoft.Data.SqlClient.SqlAuthenticationMethod>。</span><span class="sxs-lookup"><span data-stu-id="143d6-170">See <xref:Microsoft.Data.SqlClient.SqlAuthenticationMethod> for more information.</span></span>

> [!WARNING]
> <span data-ttu-id="143d6-171">请注意，如果对打开连接进行了同步调用，则侦听器将引发。</span><span class="sxs-lookup"><span data-stu-id="143d6-171">Notice that the interceptor throws if a sync call is made to open the connection.</span></span> <span data-ttu-id="143d6-172">这是因为不存在用于获取访问令牌的非异步方法，并且没有 [从非异步上下文调用异步方法而不会造成死锁的通用](https://devblogs.microsoft.com/dotnet/configureawait-faq/)方法。</span><span class="sxs-lookup"><span data-stu-id="143d6-172">This is because there is no non-async method to obtain the access token and there is [no universal and simple way to call an async method from non-async context without risking deadlock](https://devblogs.microsoft.com/dotnet/configureawait-faq/).</span></span>

> [!WARNING]
> <span data-ttu-id="143d6-173">在某些情况下，无法自动缓存访问令牌。</span><span class="sxs-lookup"><span data-stu-id="143d6-173">in some situations the access token may not be cached automatically the Azure Token Provider.</span></span> <span data-ttu-id="143d6-174">根据所请求的令牌类型，可能需要在此处实现自己的缓存。</span><span class="sxs-lookup"><span data-stu-id="143d6-174">Depending on the kind of token requested, you may need to implement your own caching here.</span></span>

### <a name="example-advanced-command-interception-for-caching"></a><span data-ttu-id="143d6-175">示例：缓存的高级命令拦截</span><span class="sxs-lookup"><span data-stu-id="143d6-175">Example: Advanced command interception for caching</span></span>

> [!TIP]  
> <span data-ttu-id="143d6-176">可以从 GitHub [下载高级命令侦听器示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/CachingInterception) 。</span><span class="sxs-lookup"><span data-stu-id="143d6-176">You can [download the advanced command interceptor sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/CachingInterception) from GitHub.</span></span>

<span data-ttu-id="143d6-177">EF Core 侦听器可以：</span><span class="sxs-lookup"><span data-stu-id="143d6-177">EF Core interceptors can:</span></span>

* <span data-ttu-id="143d6-178">告诉 EF Core 取消执行正在截获的操作</span><span class="sxs-lookup"><span data-stu-id="143d6-178">Tell EF Core to suppress executing the operation being intercepted</span></span>
* <span data-ttu-id="143d6-179">将报告的操作结果改回 EF Core</span><span class="sxs-lookup"><span data-stu-id="143d6-179">Change the result of the operation reported back to EF Core</span></span>

<span data-ttu-id="143d6-180">此示例显示了一个侦听器，该侦听器使用这些功能作为基元二级缓存的行为。</span><span class="sxs-lookup"><span data-stu-id="143d6-180">This example shows an interceptor that uses these features to behave like a primitive second-level cache.</span></span> <span data-ttu-id="143d6-181">为特定查询返回缓存查询结果，避免数据库往返。</span><span class="sxs-lookup"><span data-stu-id="143d6-181">Cached query results are returned for a specific query, avoiding a database roundtrip.</span></span>

> [!WARNING]
> <span data-ttu-id="143d6-182">以这种方式更改 EF Core 默认行为时要格外小心。</span><span class="sxs-lookup"><span data-stu-id="143d6-182">Take care when changing the EF Core default behavior in this way.</span></span> <span data-ttu-id="143d6-183">如果出现无法正确处理的异常结果，EF Core 可能会以意外的方式运行。</span><span class="sxs-lookup"><span data-stu-id="143d6-183">EF Core may behave in unexpected ways if it gets an abnormal result that it cannot process correctly.</span></span> <span data-ttu-id="143d6-184">此外，此示例还演示侦听器概念;它不是为了提供强大的二级缓存实现的模板。</span><span class="sxs-lookup"><span data-stu-id="143d6-184">Also, this example demonstrates interceptor concepts; it is not intended as a template for a robust second-level cache implementation.</span></span>

<span data-ttu-id="143d6-185">在此示例中，应用程序经常执行查询以获取最新的 "每天消息"：</span><span class="sxs-lookup"><span data-stu-id="143d6-185">In this example, the application frequently executes a query to obtain the most recent "daily message":</span></span>

<!--
        async Task<string> GetDailyMessage(DailyMessageContext context)
            => (await context.DailyMessages.TagWith("Get_Daily_Message").OrderBy(e => e.Id).LastAsync()).Message;
-->
[!code-csharp[GetDailyMessage](../../../samples/core/Miscellaneous/CachingInterception/Program.cs?name=GetDailyMessage)]

<span data-ttu-id="143d6-186">此查询已 [标记](xref:core/querying/tags) 为可在侦听器中轻松检测到它。</span><span class="sxs-lookup"><span data-stu-id="143d6-186">This query is [tagged](xref:core/querying/tags) so that it can be easily detected in the interceptor.</span></span> <span data-ttu-id="143d6-187">其思路是每天只查询一次新消息的数据库。</span><span class="sxs-lookup"><span data-stu-id="143d6-187">The idea is to only query the database for a new message once every day.</span></span> <span data-ttu-id="143d6-188">在其他情况，应用程序将使用缓存的结果。</span><span class="sxs-lookup"><span data-stu-id="143d6-188">At other times the application will use a cached result.</span></span> <span data-ttu-id="143d6-189"> (示例使用示例中的延迟10秒来模拟新的一天。 ) </span><span class="sxs-lookup"><span data-stu-id="143d6-189">(The sample uses delay of 10 seconds in the sample to simulate a new day.)</span></span>

#### <a name="interceptor-state"></a><span data-ttu-id="143d6-190">侦听器状态</span><span class="sxs-lookup"><span data-stu-id="143d6-190">Interceptor state</span></span>

<span data-ttu-id="143d6-191">此拦截程序是有状态的：它存储查询的最近一次消息的 ID 和消息文本，以及执行该查询的时间。</span><span class="sxs-lookup"><span data-stu-id="143d6-191">This interceptor is stateful: it stores the ID and message text of the most recent daily message queried, plus the time when that query was executed.</span></span> <span data-ttu-id="143d6-192">由于此状态，我们还需要一个 [锁](/dotnet/csharp/language-reference/keywords/lock-statement) ，因为缓存要求相同的侦听器必须由多个上下文实例使用。</span><span class="sxs-lookup"><span data-stu-id="143d6-192">Because of this state we also need a [lock](/dotnet/csharp/language-reference/keywords/lock-statement) since the caching requires that same interceptor must be used by multiple context instances.</span></span>

<!--
    private readonly object _lock = new object();
    private int _id;
    private string _message;
    private DateTime _queriedAt;
-->
[!code-csharp[InterceptorState](../../../samples/core/Miscellaneous/CachingInterception/CachingCommandInterceptor.cs?name=InterceptorState)]

#### <a name="before-execution"></a><span data-ttu-id="143d6-193">执行之前</span><span class="sxs-lookup"><span data-stu-id="143d6-193">Before execution</span></span>

<span data-ttu-id="143d6-194">在 `Executing` 方法 (即，在对数据库调用进行) 之前，侦听器会检测标记的查询，然后检查是否有缓存的结果。</span><span class="sxs-lookup"><span data-stu-id="143d6-194">In the `Executing` method (i.e. before making a database call), the interceptor detects the tagged query and then checks if there is a cached result.</span></span> <span data-ttu-id="143d6-195">如果找到这样的结果，则会取消查询并改为使用缓存的结果。</span><span class="sxs-lookup"><span data-stu-id="143d6-195">If such a result is found, then the query is suppressed and cached results are used instead.</span></span>

<!--
    public override ValueTask<InterceptionResult<DbDataReader>> ReaderExecutingAsync(
        DbCommand command,
        CommandEventData eventData,
        InterceptionResult<DbDataReader> result,
        CancellationToken cancellationToken = default)
    {
        if (command.CommandText.StartsWith("-- Get_Daily_Message", StringComparison.Ordinal))
        {
            lock (_lock)
            {
                if (_message != null
                    && DateTime.UtcNow < _queriedAt + new TimeSpan(0, 0, 10))
                {
                    command.CommandText = "-- Get_Daily_Message: Skipping DB call; using cache.";
                    result = InterceptionResult<DbDataReader>.SuppressWithResult(new CachedDailyMessageDataReader(_id, _message));
                }
            }
        }

        return new ValueTask<InterceptionResult<DbDataReader>>(result);
    }
-->
[!code-csharp[ReaderExecutingAsync](../../../samples/core/Miscellaneous/CachingInterception/CachingCommandInterceptor.cs?name=ReaderExecutingAsync)]

<span data-ttu-id="143d6-196">请注意，代码如何调用 <xref:Microsoft.EntityFrameworkCore.Diagnostics.InterceptionResult%601.SuppressWithResult%2A?displayProperty=nameWithType> 并传递 <xref:System.Data.Common.DbDataReader> 包含缓存数据的替换。</span><span class="sxs-lookup"><span data-stu-id="143d6-196">Notice how the code calls <xref:Microsoft.EntityFrameworkCore.Diagnostics.InterceptionResult%601.SuppressWithResult%2A?displayProperty=nameWithType> and passes a replacement <xref:System.Data.Common.DbDataReader> containing the cached data.</span></span> <span data-ttu-id="143d6-197">然后，将返回此 InterceptionResult，从而导致执行查询。</span><span class="sxs-lookup"><span data-stu-id="143d6-197">This InterceptionResult is then returned, causing suppression of query execution.</span></span> <span data-ttu-id="143d6-198">替换读取器将作为查询结果 EF Core 使用。</span><span class="sxs-lookup"><span data-stu-id="143d6-198">The replacement reader is instead used by EF Core as the results of the query.</span></span>

<span data-ttu-id="143d6-199">此侦听器还处理命令文本。</span><span class="sxs-lookup"><span data-stu-id="143d6-199">This interceptor also manipulates the command text.</span></span> <span data-ttu-id="143d6-200">此操作不是必需的，但会提高日志消息的清晰度。</span><span class="sxs-lookup"><span data-stu-id="143d6-200">This manipulation is not required, but improves clarity in log messages.</span></span> <span data-ttu-id="143d6-201">命令文本不一定是有效的 SQL，因为现在不会执行查询。</span><span class="sxs-lookup"><span data-stu-id="143d6-201">The command text does not need to be valid SQL since the query is now not going to be executed.</span></span>

#### <a name="after-execution"></a><span data-ttu-id="143d6-202">执行之后</span><span class="sxs-lookup"><span data-stu-id="143d6-202">After execution</span></span>

<span data-ttu-id="143d6-203">如果没有可用的缓存消息，或者如果它已过期，则上面的代码不会显示结果。</span><span class="sxs-lookup"><span data-stu-id="143d6-203">If no cached message is available, or if it has expired, then the code above does not suppress the result.</span></span> <span data-ttu-id="143d6-204">因此 EF Core 将以正常方式执行查询。</span><span class="sxs-lookup"><span data-stu-id="143d6-204">EF Core will therefore execute the query as normal.</span></span> <span data-ttu-id="143d6-205">然后，它将在执行后返回到侦听器的 `Executed` 方法。</span><span class="sxs-lookup"><span data-stu-id="143d6-205">It will then return to the interceptor's `Executed` method after execution.</span></span> <span data-ttu-id="143d6-206">此时，如果该结果不是已缓存的读取器，则将从真正的读取器中 exacted 新的消息 ID 和字符串，并将其缓存起来，以便下一次使用此查询。</span><span class="sxs-lookup"><span data-stu-id="143d6-206">At this point if the result is not already a cached reader, then the new message ID and string is exacted from the real reader and cached ready for the next use of this query.</span></span>

<!--
    public override async ValueTask<DbDataReader> ReaderExecutedAsync(
        DbCommand command,
        CommandExecutedEventData eventData,
        DbDataReader result,
        CancellationToken cancellationToken = default)
    {
        if (command.CommandText.StartsWith("-- Get_Daily_Message", StringComparison.Ordinal)
            && !(result is CachedDailyMessageDataReader))
        {
            try
            {
                await result.ReadAsync(cancellationToken);

                lock (_lock)
                {
                    _id = result.GetInt32(0);
                    _message = result.GetString(1);
                    _queriedAt = DateTime.UtcNow;
                    return new CachedDailyMessageDataReader(_id, message);
                }
            }
            finally
            {
                await result.DisposeAsync();
            }
        }

        return result;
    }
-->
[!code-csharp[ReaderExecutedAsync](../../../samples/core/Miscellaneous/CachingInterception/CachingCommandInterceptor.cs?name=ReaderExecutedAsync)]

#### <a name="demonstration"></a><span data-ttu-id="143d6-207">演示</span><span class="sxs-lookup"><span data-stu-id="143d6-207">Demonstration</span></span>

<span data-ttu-id="143d6-208">[缓存侦听器示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/CachingInterception)包含一个简单的控制台应用程序，该应用程序查询每日消息以测试缓存：</span><span class="sxs-lookup"><span data-stu-id="143d6-208">The [caching interceptor sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/CachingInterception) contains a simple console application that queries for daily messages to test the caching:</span></span>

<!--
        // 1. Initialize the database with some daily messages.
        using (var context = new DailyMessageContext())
        {
            await context.Database.EnsureDeletedAsync();
            await context.Database.EnsureCreatedAsync();

            context.AddRange(
                new DailyMessage { Message = "Remember: All builds are GA; no builds are RTM." },
                new DailyMessage { Message = "Keep calm and drink tea" });

            await context.SaveChangesAsync();
        }

        // 2. Query for the most recent daily message. It will be cached for 10 seconds.
        using (var context = new DailyMessageContext())
        {
            Console.WriteLine(await GetDailyMessage(context));
        }

        // 3. Insert a new daily message.
        using (var context = new DailyMessageContext())
        {
            context.Add(new DailyMessage { Message = "Free beer for unicorns" });

            await context.SaveChangesAsync();
        }

        // 4. Cached message is used until cache expires.
        using (var context = new DailyMessageContext())
        {
            Console.WriteLine(await GetDailyMessage(context));
        }

        // 5. Pretend it's the next day.
        Thread.Sleep(10000);

        // 6. Cache is expired, so the last message will noe be queried again.
        using (var context = new DailyMessageContext())
        {
            Console.WriteLine(await GetDailyMessage(context));
        }
        
        #region GetDailyMessage
        async Task<string> GetDailyMessage(DailyMessageContext context)
            => (await context.DailyMessages.TagWith("Get_Daily_Message").OrderBy(e => e.Id).LastAsync()).Message;
        #endregion
-->
[!code-csharp[Main](../../../samples/core/Miscellaneous/CachingInterception/Program.cs?name=Main)]

<span data-ttu-id="143d6-209">这会导致生成以下输出：</span><span class="sxs-lookup"><span data-stu-id="143d6-209">This results in the following output:</span></span>

```output
info: 10/15/2020 12:32:11.801 RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command)
      Executed DbCommand (0ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      -- Get_Daily_Message

      SELECT "d"."Id", "d"."Message"
      FROM "DailyMessages" AS "d"
      ORDER BY "d"."Id" DESC
      LIMIT 1

Keep calm and drink tea

info: 10/15/2020 12:32:11.821 RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command)
      Executed DbCommand (0ms) [Parameters=[@p0='Free beer for unicorns' (Size = 22)], CommandType='Text', CommandTimeout='30']
      INSERT INTO "DailyMessages" ("Message")
      VALUES (@p0);
      SELECT "Id"
      FROM "DailyMessages"
      WHERE changes() = 1 AND "rowid" = last_insert_rowid();

info: 10/15/2020 12:32:11.826 RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command)
      Executed DbCommand (0ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      -- Get_Daily_Message: Skipping DB call; using cache.

Keep calm and drink tea

info: 10/15/2020 12:32:21.833 RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command)
      Executed DbCommand (0ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      -- Get_Daily_Message

      SELECT "d"."Id", "d"."Message"
      FROM "DailyMessages" AS "d"
      ORDER BY "d"."Id" DESC
      LIMIT 1

Free beer for unicorns
```

<span data-ttu-id="143d6-210">从日志输出中可以看到，应用程序继续使用缓存消息，直到超时过期，此时将再次查询数据库以获取任何新的消息。</span><span class="sxs-lookup"><span data-stu-id="143d6-210">Notice from the log output that the application continues to use the cached message until the timeout expires, at which point the database is queried again for any new message.</span></span>

## <a name="savechanges-interception"></a><span data-ttu-id="143d6-211">SaveChanges 拦截</span><span class="sxs-lookup"><span data-stu-id="143d6-211">SaveChanges interception</span></span>

> [!NOTE]
> <span data-ttu-id="143d6-212">EF Core 5.0 中引入了 SaveChanges 拦截。</span><span class="sxs-lookup"><span data-stu-id="143d6-212">SaveChanges interception was introduced in EF Core 5.0.</span></span>

> [!TIP]  
> <span data-ttu-id="143d6-213">可以从 GitHub [下载 SaveChanges 侦听器示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SaveChangesInterception) 。</span><span class="sxs-lookup"><span data-stu-id="143d6-213">You can [download the SaveChanges interceptor sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SaveChangesInterception) from GitHub.</span></span>

<span data-ttu-id="143d6-214"><xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 和 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A> 侦听点由 `ISaveChangesInterceptor`</span><span class="sxs-lookup"><span data-stu-id="143d6-214"><xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> and <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A> interception points are defined by the `ISaveChangesInterceptor`</span></span> <!-- Issue #2748 --> <span data-ttu-id="143d6-215">交互.</span><span class="sxs-lookup"><span data-stu-id="143d6-215">interface.</span></span> <span data-ttu-id="143d6-216">对于其他侦听器， `SaveChangesInterceptor`</span><span class="sxs-lookup"><span data-stu-id="143d6-216">As for other interceptors, the `SaveChangesInterceptor`</span></span> <!-- Issue #2748 --> <span data-ttu-id="143d6-217">为方便起见，提供了具有无操作方法的基类。</span><span class="sxs-lookup"><span data-stu-id="143d6-217">base class with no-op methods is provided as a convenience.</span></span>

> [!TIP]
> <span data-ttu-id="143d6-218">拦截功能强大。</span><span class="sxs-lookup"><span data-stu-id="143d6-218">Interceptors are powerful.</span></span> <span data-ttu-id="143d6-219">但是，在许多情况下，重写 SaveChanges 方法或使用 DbContext 上公开的 [savechanges 的 .net 事件](xref:core/logging-events-diagnostics/events) 可能更容易。</span><span class="sxs-lookup"><span data-stu-id="143d6-219">However, in many cases it may be easier to override the SaveChanges method or use the [.NET events for SaveChanges](xref:core/logging-events-diagnostics/events) exposed on DbContext.</span></span>

### <a name="example-savechanges-interception-for-auditing"></a><span data-ttu-id="143d6-220">示例：用于审核的 SaveChanges 侦听</span><span class="sxs-lookup"><span data-stu-id="143d6-220">Example: SaveChanges interception for auditing</span></span>

<span data-ttu-id="143d6-221">可以截获 SaveChanges 来创建所进行更改的独立审核记录。</span><span class="sxs-lookup"><span data-stu-id="143d6-221">SaveChanges can be intercepted to create an independent audit record of the changes made.</span></span>

> [!NOTE]
> <span data-ttu-id="143d6-222">这并不是一种可靠的审核解决方案。</span><span class="sxs-lookup"><span data-stu-id="143d6-222">This is not intended to be a robust auditing solution.</span></span> <span data-ttu-id="143d6-223">这是一个简化的示例，用于演示截获的功能。</span><span class="sxs-lookup"><span data-stu-id="143d6-223">Rather it is a simplistic example used to demonstrate the features of interception.</span></span>

#### <a name="the-application-context"></a><span data-ttu-id="143d6-224">应用程序上下文</span><span class="sxs-lookup"><span data-stu-id="143d6-224">The application context</span></span>

<span data-ttu-id="143d6-225">[审核示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SaveChangesInterception)对博客和帖子使用简单 DbContext。</span><span class="sxs-lookup"><span data-stu-id="143d6-225">The [sample for auditing](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SaveChangesInterception) uses a simple DbContext with blogs and posts.</span></span>

<!--
public class BlogsContext : DbContext
{
    private readonly AuditingInterceptor _auditingInterceptor = new AuditingInterceptor("DataSource=audit.db");

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .AddInterceptors(_auditingInterceptor)
            .UseSqlite("DataSource=blogs.db");

    public DbSet<Blog> Blogs { get; set; }
}

public class Blog
{
    public int Id { get; set; }
    public string Name { get; set; }

    public ICollection<Post> Posts { get; } = new List<Post>();
}

public class Post
{
    public int Id { get; set; }
    public string Title { get; set; }

    public Blog Blog { get; set; }
}
-->
[!code-csharp[BlogsContext](../../../samples/core/Miscellaneous/SaveChangesInterception/BlogsContext.cs?name=BlogsContext)]

<span data-ttu-id="143d6-226">请注意，会为每个 DbContext 实例注册侦听器的新实例。</span><span class="sxs-lookup"><span data-stu-id="143d6-226">Notice that a new instance of the interceptor is registered for each DbContext instance.</span></span> <span data-ttu-id="143d6-227">这是因为审核侦听器包含链接到当前上下文实例的状态。</span><span class="sxs-lookup"><span data-stu-id="143d6-227">This is because the auditing interceptor contains state linked to the current context instance.</span></span>

#### <a name="the-audit-context"></a><span data-ttu-id="143d6-228">审核上下文</span><span class="sxs-lookup"><span data-stu-id="143d6-228">The audit context</span></span>

<span data-ttu-id="143d6-229">该示例还包含用于审核数据库的第二个 DbContext 和模型。</span><span class="sxs-lookup"><span data-stu-id="143d6-229">The sample also contains a second DbContext and model used for the auditing database.</span></span>

<!--
public class AuditContext : DbContext
{
    private readonly string _connectionString;

    public AuditContext(string connectionString)
    {
        _connectionString = connectionString;
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.UseSqlite(_connectionString);

    public DbSet<SaveChangesAudit> SaveChangesAudits { get; set; }
}

public class SaveChangesAudit
{
    public int Id { get; set; }
    public Guid AuditId { get; set; }
    public DateTime StartTime { get; set; }
    public DateTime EndTime { get; set; }
    public bool Succeeded { get; set; }
    public string ErrorMessage { get; set; }

    public ICollection<EntityAudit> Entities { get; } = new List<EntityAudit>();
}

public class EntityAudit
{
    public int Id { get; set; }
    public EntityState State { get; set; }
    public string AuditMessage { get; set; }

    public SaveChangesAudit SaveChangesAudit { get; set; }
}
-->
[!code-csharp[AuditContext](../../../samples/core/Miscellaneous/SaveChangesInterception/AuditContext.cs?name=AuditContext)]

#### <a name="the-interceptor"></a><span data-ttu-id="143d6-230">侦听器</span><span class="sxs-lookup"><span data-stu-id="143d6-230">The interceptor</span></span>

<span data-ttu-id="143d6-231">与侦听器审核的一般思路如下：</span><span class="sxs-lookup"><span data-stu-id="143d6-231">The general idea for auditing with the interceptor is:</span></span>

* <span data-ttu-id="143d6-232">在 SaveChanges 开始时创建审核消息，并将其写入审核数据库</span><span class="sxs-lookup"><span data-stu-id="143d6-232">An audit message is created at the beginning of SaveChanges and is written to the auditing database</span></span>
* <span data-ttu-id="143d6-233">允许使用 SaveChanges 继续</span><span class="sxs-lookup"><span data-stu-id="143d6-233">SaveChanges is allowed to continue</span></span>
* <span data-ttu-id="143d6-234">如果 SaveChanges 成功，则更新审核消息以指示成功</span><span class="sxs-lookup"><span data-stu-id="143d6-234">If SaveChanges succeeds, then the audit message is updated to indicate success</span></span>
* <span data-ttu-id="143d6-235">如果 SaveChanges 失败，则更新审核消息以指示失败</span><span class="sxs-lookup"><span data-stu-id="143d6-235">If SaveChanges fails, then the audit message is updated to indicate the failure</span></span>

<span data-ttu-id="143d6-236">在使用的替代将任何更改发送到数据库之前处理第一个阶段 `ISaveChangesInterceptor.SavingChanges`</span><span class="sxs-lookup"><span data-stu-id="143d6-236">The first stage is handled before any changes are sent to the database using overrides of `ISaveChangesInterceptor.SavingChanges`</span></span> <!-- Issue #2748 --> <span data-ttu-id="143d6-237"> 和 `ISaveChangesInterceptor.SavingChangesAsync`</span><span class="sxs-lookup"><span data-stu-id="143d6-237">and `ISaveChangesInterceptor.SavingChangesAsync`</span></span><!-- Issue #2748 --><span data-ttu-id="143d6-238">.</span><span class="sxs-lookup"><span data-stu-id="143d6-238">.</span></span>

<!--
    public async ValueTask<InterceptionResult<int>> SavingChangesAsync(
        DbContextEventData eventData,
        InterceptionResult<int> result,
        CancellationToken cancellationToken = default)
    {
        _audit = CreateAudit(eventData.Context);

        using (var auditContext = new AuditContext(_connectionString))
        {
            auditContext.Add(_audit);
            await auditContext.SaveChangesAsync();
        }

        return result;
    }

    public InterceptionResult<int> SavingChanges(
        DbContextEventData eventData,
        InterceptionResult<int> result)
    {
        _audit = CreateAudit(eventData.Context);

        using (var auditContext = new AuditContext(_connectionString))
        {
            auditContext.Add(_audit);
            auditContext.SaveChanges();
        }

        return result;
    }
-->
[!code-csharp[SavingChanges](../../../samples/core/Miscellaneous/SaveChangesInterception/AuditingInterceptor.cs?name=SavingChanges)]

<span data-ttu-id="143d6-239">同时替代同步和异步方法可确保无论是否调用 SaveChanges 或 SaveChangesAsync，都将发生审核。</span><span class="sxs-lookup"><span data-stu-id="143d6-239">Overriding both sync and async methods ensures that auditing will happen regardless of whether SaveChanges or SaveChangesAsync are called.</span></span> <span data-ttu-id="143d6-240">另请注意，异步重载本身能够对审核数据库执行非阻塞异步 i/o。</span><span class="sxs-lookup"><span data-stu-id="143d6-240">Notice also that the async overload is itself able to perform non-blocking async I/O to the auditing database.</span></span> <span data-ttu-id="143d6-241">你可能希望从 sync SavingChanges 方法中引发，以确保所有数据库 i/o 都是异步的。</span><span class="sxs-lookup"><span data-stu-id="143d6-241">You may wish to throw from the sync SavingChanges method to ensure that all database I/O is async.</span></span> <span data-ttu-id="143d6-242">这就要求应用程序始终调用 SaveChangesAsync 而绝不会调用。</span><span class="sxs-lookup"><span data-stu-id="143d6-242">This then requires that the application always calls SaveChangesAsync and never SaveChanges.</span></span>

#### <a name="the-audit-message"></a><span data-ttu-id="143d6-243">审核消息</span><span class="sxs-lookup"><span data-stu-id="143d6-243">The audit message</span></span>

<span data-ttu-id="143d6-244">每个侦听器方法都有一个 `eventData` 参数，该参数提供有关被截获事件的上下文信息。</span><span class="sxs-lookup"><span data-stu-id="143d6-244">Every interceptor method has an `eventData` parameter providing contextual information about the event being intercepted.</span></span> <span data-ttu-id="143d6-245">在这种情况下，将在事件数据中包含当前应用程序 DbContext，然后将该数据用于创建审核消息。</span><span class="sxs-lookup"><span data-stu-id="143d6-245">In this case the current application DbContext is included in the event data, which is then used to create an audit message.</span></span>

<!--
    private static SaveChangesAudit CreateAudit(DbContext context)
    {
        context.ChangeTracker.DetectChanges();

        var audit = new SaveChangesAudit
        {
            AuditId = Guid.NewGuid(),
            StartTime = DateTime.UtcNow
        };

        foreach (var entry in context.ChangeTracker.Entries())
        {
            var auditMessage = entry.State switch
            {
                EntityState.Deleted => CreateDeletedMessage(entry),
                EntityState.Modified => CreateModifiedMessage(entry),
                EntityState.Added => CreateAddedMessage(entry),
                _ => null
            };

            if (auditMessage != null)
            {
                audit.Entities.Add(new EntityAudit { State = entry.State, AuditMessage = auditMessage });
            }
        }

        return audit;

        string CreateAddedMessage(EntityEntry entry)
            => entry.Properties.Aggregate(
                $"Inserting {entry.Metadata.DisplayName()} with ",
                (auditString, property) => auditString + $"{property.Metadata.Name}: '{property.CurrentValue}' ");

        string CreateModifiedMessage(EntityEntry entry)
            => entry.Properties.Where(property => property.IsModified || property.Metadata.IsPrimaryKey()).Aggregate(
                $"Updating {entry.Metadata.DisplayName()} with ",
                (auditString, property) => auditString + $"{property.Metadata.Name}: '{property.CurrentValue}' ");

        string CreateDeletedMessage(EntityEntry entry)
            => entry.Properties.Where(property => property.Metadata.IsPrimaryKey()).Aggregate(
                $"Deleting {entry.Metadata.DisplayName()} with ",
                (auditString, property) => auditString + $"{property.Metadata.Name}: '{property.CurrentValue}' ");
    }
-->
[!code-csharp[CreateAudit](../../../samples/core/Miscellaneous/SaveChangesInterception/AuditingInterceptor.cs?name=CreateAudit)]

<span data-ttu-id="143d6-246">结果是 `SaveChangesAudit` 具有实体集合的实体 `EntityAudit` ，每个实体对应于插入、更新或删除。</span><span class="sxs-lookup"><span data-stu-id="143d6-246">The result is a `SaveChangesAudit` entity with a collection of `EntityAudit` entities, one for each insert, update, or delete.</span></span> <span data-ttu-id="143d6-247">然后，侦听器会将这些实体插入到审核数据库中。</span><span class="sxs-lookup"><span data-stu-id="143d6-247">The interceptor then inserts these entities into the audit database.</span></span>

> [!TIP]
> <span data-ttu-id="143d6-248">在每个 EF Core 事件数据类中重写 ToString，以生成事件的等效日志消息。</span><span class="sxs-lookup"><span data-stu-id="143d6-248">ToString is overridden in every EF Core event data class to generate the equivalent log message for the event.</span></span> <span data-ttu-id="143d6-249">例如，调用会 `ContextInitializedEventData.ToString` 生成 "Entity Framework Core 5.0.0，并使用提供程序" microsoft.entityframeworkcore "（选项为" 无 "）初始化" BlogsContext "。</span><span class="sxs-lookup"><span data-stu-id="143d6-249">For example, calling `ContextInitializedEventData.ToString` generates "Entity Framework Core 5.0.0 initialized 'BlogsContext' using provider 'Microsoft.EntityFrameworkCore.Sqlite' with options: None".</span></span>

#### <a name="detecting-success"></a><span data-ttu-id="143d6-250">检测成功</span><span class="sxs-lookup"><span data-stu-id="143d6-250">Detecting success</span></span>

<span data-ttu-id="143d6-251">审核实体存储在侦听器上，以便一旦 SaveChanges 成功或失败，就可以再次访问该实体。</span><span class="sxs-lookup"><span data-stu-id="143d6-251">The audit entity is stored on the interceptor so that it can be accessed again once SaveChanges either succeeds or fails.</span></span> <span data-ttu-id="143d6-252">如果成功， `ISaveChangesInterceptor.SavedChanges`</span><span class="sxs-lookup"><span data-stu-id="143d6-252">For success, `ISaveChangesInterceptor.SavedChanges`</span></span> <!-- Issue #2748 --> <span data-ttu-id="143d6-253">或 `ISaveChangesInterceptor.SavedChangesAsync`</span><span class="sxs-lookup"><span data-stu-id="143d6-253">or `ISaveChangesInterceptor.SavedChangesAsync`</span></span> <!-- Issue #2748 --> <span data-ttu-id="143d6-254">调用 。</span><span class="sxs-lookup"><span data-stu-id="143d6-254">is called.</span></span>

<!--
    public int SavedChanges(SaveChangesCompletedEventData eventData, int result)
    {
        using (var auditContext = new AuditContext(_connectionString))
        {
            auditContext.Attach(_audit);
            _audit.Succeeded = true;
            _audit.EndTime = DateTime.UtcNow;

            auditContext.SaveChanges();
        }

        return result;
    }

    public async ValueTask<int> SavedChangesAsync(
        SaveChangesCompletedEventData eventData,
        int result,
        CancellationToken cancellationToken = new CancellationToken())
    {
        using (var auditContext = new AuditContext(_connectionString))
        {
            auditContext.Attach(_audit);
            _audit.Succeeded = true;
            _audit.EndTime = DateTime.UtcNow;

            await auditContext.SaveChangesAsync(cancellationToken);
        }

        return result;
    }
-->
[!code-csharp[SavedChanges](../../../samples/core/Miscellaneous/SaveChangesInterception/AuditingInterceptor.cs?name=SavedChanges)]

<span data-ttu-id="143d6-255">审核实体附加到审核上下文，因为它已存在于数据库中，需要更新。</span><span class="sxs-lookup"><span data-stu-id="143d6-255">The audit entity is attached to the audit context, since it already exists in the database and needs to be updated.</span></span> <span data-ttu-id="143d6-256">然后，设置 `Succeeded` 和 `EndTime` ，这会将这些属性标记为已修改，因此 SaveChanges 会将更新发送到审核数据库。</span><span class="sxs-lookup"><span data-stu-id="143d6-256">We then set `Succeeded` and `EndTime`, which marks these properties as modified so SaveChanges will send an update to the audit database.</span></span>

#### <a name="detecting-failure"></a><span data-ttu-id="143d6-257">检测失败</span><span class="sxs-lookup"><span data-stu-id="143d6-257">Detecting failure</span></span>

<span data-ttu-id="143d6-258">处理故障的方式与成功的方式几乎相同，但在 `ISaveChangesInterceptor.SaveChangesFailed`</span><span class="sxs-lookup"><span data-stu-id="143d6-258">Failure is handled in much the same way as success, but in the `ISaveChangesInterceptor.SaveChangesFailed`</span></span> <!-- Issue #2748 --> <span data-ttu-id="143d6-259">或 `ISaveChangesInterceptor.SaveChangesFailedAsync`</span><span class="sxs-lookup"><span data-stu-id="143d6-259">or `ISaveChangesInterceptor.SaveChangesFailedAsync`</span></span> <!-- Issue #2748 --> <span data-ttu-id="143d6-260">方法。</span><span class="sxs-lookup"><span data-stu-id="143d6-260">method.</span></span> <span data-ttu-id="143d6-261">事件数据包含引发的异常。</span><span class="sxs-lookup"><span data-stu-id="143d6-261">The event data contains the exception that was thrown.</span></span>

<!--
    public void SaveChangesFailed(DbContextErrorEventData eventData)
    {
        using (var auditContext = new AuditContext(_connectionString))
        {
            auditContext.Attach(_audit);
            _audit.Succeeded = false;
            _audit.EndTime = DateTime.UtcNow;
            _audit.ErrorMessage = eventData.Exception.Message;

            auditContext.SaveChanges();
        }
    }

    public async Task SaveChangesFailedAsync(
        DbContextErrorEventData eventData,
        CancellationToken cancellationToken = new CancellationToken())
    {
        using (var auditContext = new AuditContext(_connectionString))
        {
            auditContext.Attach(_audit);
            _audit.Succeeded = false;
            _audit.EndTime = DateTime.UtcNow;
            _audit.ErrorMessage = eventData.Exception.InnerException?.Message;

            await auditContext.SaveChangesAsync(cancellationToken);
        }
    }
-->
[!code-csharp[SaveChangesFailed](../../../samples/core/Miscellaneous/SaveChangesInterception/AuditingInterceptor.cs?name=SaveChangesFailed)]

#### <a name="demonstration"></a><span data-ttu-id="143d6-262">演示</span><span class="sxs-lookup"><span data-stu-id="143d6-262">Demonstration</span></span>

<span data-ttu-id="143d6-263">[审核示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SaveChangesInterception)包含一个简单的控制台应用程序，该应用程序对博客数据库进行更改，然后显示已创建的审核。</span><span class="sxs-lookup"><span data-stu-id="143d6-263">The [auditing sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SaveChangesInterception) contains a simple console application that makes changes to the blogging database and then shows the auditing that was created.</span></span>

<!--
        // Insert, update, and delete some entities

        using (var context = new BlogsContext())
        {
            context.Add(
                new Blog
                {
                    Name = "EF Blog",
                    Posts =
                    {
                        new Post { Title = "EF Core 3.1!" },
                        new Post { Title = "EF Core 5.0!" }
                    }
                });

            await context.SaveChangesAsync();
        }

        using (var context = new BlogsContext())
        {
            var blog = context.Blogs.Include(e => e.Posts).Single();

            blog.Name = "EF Core Blog";
            context.Remove(blog.Posts.First());
            blog.Posts.Add(new Post { Title = "EF Core 6.0!" });

            context.SaveChanges();
        }

        // Do an insert that will fail

        using (var context = new BlogsContext())
        {
            try
            {
                context.Add(new Post { Id = 3, Title = "EF Core 3.1!" });

                await context.SaveChangesAsync();
            }
            catch (DbUpdateException)
            {
            }
        }

        // Look at the audit trail

        using (var context = new AuditContext("DataSource=audit.db"))
        {
            foreach (var audit in context.SaveChangesAudits.Include(e => e.Entities).ToList())
            {
                Console.WriteLine(
                    $"Audit {audit.AuditId} from {audit.StartTime} to {audit.EndTime} was{(audit.Succeeded ? "" : " not")} successful.");

                foreach (var entity in audit.Entities)
                {
                    Console.WriteLine($"  {entity.AuditMessage}");
                }

                if (!audit.Succeeded)
                {
                    Console.WriteLine($"  Error: {audit.ErrorMessage}");
                }
            }
        }
-->
[!code-csharp[Program](../../../samples/core/Miscellaneous/SaveChangesInterception/Program.cs?name=Program)]

<span data-ttu-id="143d6-264">结果显示审核数据库的内容：</span><span class="sxs-lookup"><span data-stu-id="143d6-264">The result shows the contents of the auditing database:</span></span>

```output
Audit 52e94327-1767-4046-a3ca-4c6b1eecbca6 from 10/14/2020 9:10:17 PM to 10/14/2020 9:10:17 PM was successful.
  Inserting Blog with Id: '-2147482647' Name: 'EF Blog'
  Inserting Post with Id: '-2147482647' BlogId: '-2147482647' Title: 'EF Core 3.1!'
  Inserting Post with Id: '-2147482646' BlogId: '-2147482647' Title: 'EF Core 5.0!'
Audit 8450f57a-5030-4211-a534-eb66b8da7040 from 10/14/2020 9:10:17 PM to 10/14/2020 9:10:17 PM was successful.
  Inserting Post with Id: '-2147482645' BlogId: '1' Title: 'EF Core 6.0!'
  Updating Blog with Id: '1' Name: 'EF Core Blog'
  Deleting Post with Id: '1'
Audit 201fef4d-66a7-43ad-b9b6-b57e9d3f37b3 from 10/14/2020 9:10:17 PM to 10/14/2020 9:10:17 PM was not successful.
  Inserting Post with Id: '3' BlogId: '' Title: 'EF Core 3.1!'
  Error: SQLite Error 19: 'UNIQUE constraint failed: Post.Id'.
```
