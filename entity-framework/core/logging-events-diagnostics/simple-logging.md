---
title: 简单日志记录-EF Core
description: 使用 LogTo 从 EF Core DbContext 进行日志记录
author: ajcvickers
ms.date: 10/03/2020
uid: core/logging-events-diagnostics/simple-logging
ms.openlocfilehash: 076c4b12aa033b51a2b839686c520a76520ee415
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97635609"
---
# <a name="simple-logging"></a><span data-ttu-id="27c6d-103">简单的日志记录</span><span class="sxs-lookup"><span data-stu-id="27c6d-103">Simple Logging</span></span>

> [!NOTE]
> <span data-ttu-id="27c6d-104">EF Core 5.0 中已引入此功能。</span><span class="sxs-lookup"><span data-stu-id="27c6d-104">This feature was introduced in EF Core 5.0.</span></span>

> [!TIP]  
> <span data-ttu-id="27c6d-105">可以从 GitHub [下载此文章的示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SimpleLogging) 。</span><span class="sxs-lookup"><span data-stu-id="27c6d-105">You can [download this article's sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SimpleLogging) from GitHub.</span></span>

<span data-ttu-id="27c6d-106">Entity Framework Core (EF Core) 简单日志记录可用于在开发和调试应用程序时轻松获取日志。</span><span class="sxs-lookup"><span data-stu-id="27c6d-106">Entity Framework Core (EF Core) simple logging can be used to easily obtain logs while developing and debugging applications.</span></span> <span data-ttu-id="27c6d-107">这种形式的日志记录需要最少的配置，而不需要其他 NuGet 包。</span><span class="sxs-lookup"><span data-stu-id="27c6d-107">This form of logging requires minimal configuration and no additional NuGet packages.</span></span>

> [!TIP]
> <span data-ttu-id="27c6d-108">EF Core 还与 [Microsoft. Extensions. 日志记录](xref:core/logging-events-diagnostics/extensions-logging)，这需要更多配置，但通常更适合用于在生产应用程序中进行日志记录。</span><span class="sxs-lookup"><span data-stu-id="27c6d-108">EF Core also integrates with [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging), which requires more configuration, but is often more suitable for logging in production applications.</span></span>

## <a name="configuration"></a><span data-ttu-id="27c6d-109">配置</span><span class="sxs-lookup"><span data-stu-id="27c6d-109">Configuration</span></span>

<span data-ttu-id="27c6d-110">在 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.LogTo%2A> [配置 DbContext 实例](xref:core/dbcontext-configuration/index)时，可以使用任何类型的应用程序访问 EF Core 日志。</span><span class="sxs-lookup"><span data-stu-id="27c6d-110">EF Core logs can be accessed from any type of application through the use of <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.LogTo%2A> when [configuring a DbContext instance](xref:core/dbcontext-configuration/index).</span></span> <span data-ttu-id="27c6d-111">此配置通常通过替代 <xref:Microsoft.EntityFrameworkCore.DbContext.OnConfiguring%2A?displayProperty=nameWithType> 来完成。</span><span class="sxs-lookup"><span data-stu-id="27c6d-111">This configuration is commonly done in an override of <xref:Microsoft.EntityFrameworkCore.DbContext.OnConfiguring%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="27c6d-112">例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-112">For example:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(Console.WriteLine);
-->
[!code-csharp[LogToConsole](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=LogToConsole)]

<span data-ttu-id="27c6d-113">或者， `LogTo` <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> 在创建 <xref:Microsoft.EntityFrameworkCore.DbContextOptions> 要传递给构造函数的实例时，可以将作为或的一部分来调用 `DbContext` 。</span><span class="sxs-lookup"><span data-stu-id="27c6d-113">Alternately, `LogTo` can be called as part of <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> or when creating a <xref:Microsoft.EntityFrameworkCore.DbContextOptions> instance to pass to the `DbContext` constructor.</span></span>

> [!TIP]
> <span data-ttu-id="27c6d-114">当使用 AddDbContext 或将 DbContextOptions 实例传递到 DbContext 构造函数时，仍将调用 OnConfiguring。</span><span class="sxs-lookup"><span data-stu-id="27c6d-114">OnConfiguring is still called when AddDbContext is used or a DbContextOptions instance is passed to the DbContext constructor.</span></span> <span data-ttu-id="27c6d-115">这使得它成为应用上下文配置的理想位置，而不考虑如何构造 DbContext。</span><span class="sxs-lookup"><span data-stu-id="27c6d-115">This makes it the ideal place to apply context configuration regardless of how the DbContext is constructed.</span></span>

## <a name="directing-the-logs"></a><span data-ttu-id="27c6d-116">定向日志</span><span class="sxs-lookup"><span data-stu-id="27c6d-116">Directing the logs</span></span>

### <a name="logging-to-the-console"></a><span data-ttu-id="27c6d-117">日志记录到控制台</span><span class="sxs-lookup"><span data-stu-id="27c6d-117">Logging to the console</span></span>

<span data-ttu-id="27c6d-118">`LogTo` 需要一个 <xref:System.Action%601> 接受字符串的委托。</span><span class="sxs-lookup"><span data-stu-id="27c6d-118">`LogTo` requires an <xref:System.Action%601> delegate that accepts a string.</span></span> <span data-ttu-id="27c6d-119">EF Core 将为每个生成的日志消息调用此委托和一个字符串。</span><span class="sxs-lookup"><span data-stu-id="27c6d-119">EF Core will call this delegate with a string for each log message generated.</span></span> <span data-ttu-id="27c6d-120">然后，将由委托来处理给定的消息。</span><span class="sxs-lookup"><span data-stu-id="27c6d-120">It is then up to the delegate to do something with the given message.</span></span>

<span data-ttu-id="27c6d-121"><xref:System.Console.WriteLine%2A?displayProperty=nameWithType>方法通常用于此委托，如上所示。</span><span class="sxs-lookup"><span data-stu-id="27c6d-121">The <xref:System.Console.WriteLine%2A?displayProperty=nameWithType> method is often used for this delegate, as shown above.</span></span> <span data-ttu-id="27c6d-122">这会导致每个日志消息都写入控制台。</span><span class="sxs-lookup"><span data-stu-id="27c6d-122">This results in each log message being written to the console.</span></span>

### <a name="logging-to-the-debug-window"></a><span data-ttu-id="27c6d-123">记录到调试窗口</span><span class="sxs-lookup"><span data-stu-id="27c6d-123">Logging to the debug window</span></span>

<span data-ttu-id="27c6d-124"><xref:System.Diagnostics.Debug.WriteLine%2A?displayProperty=nameWithType> 可用于将输出发送到 Visual Studio 或其他 Ide 中的 "调试" 窗口。</span><span class="sxs-lookup"><span data-stu-id="27c6d-124"><xref:System.Diagnostics.Debug.WriteLine%2A?displayProperty=nameWithType> can be used to send output to the Debug window in Visual Studio or other IDEs.</span></span> <span data-ttu-id="27c6d-125">在这种情况下，必须使用[Lambda 语法](/dotnet/csharp/language-reference/operators/lambda-expressions)，因为 `Debug` 该类是从发布版本编译而来的。</span><span class="sxs-lookup"><span data-stu-id="27c6d-125">[Lambda syntax](/dotnet/csharp/language-reference/operators/lambda-expressions) must be used in this case because the `Debug` class is compiled out of release builds.</span></span> <span data-ttu-id="27c6d-126">例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-126">For example:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(message => Debug.WriteLine(message));
-->
[!code-csharp[LogToDebug](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=LogToDebug)]

### <a name="logging-to-a-file"></a><span data-ttu-id="27c6d-127">记录到文件</span><span class="sxs-lookup"><span data-stu-id="27c6d-127">Logging to a file</span></span>

<span data-ttu-id="27c6d-128">写入文件时，需要 <xref:System.IO.StreamWriter> 为该文件创建一个或类似的。</span><span class="sxs-lookup"><span data-stu-id="27c6d-128">Writing to a file requires creating a <xref:System.IO.StreamWriter> or similar for the file.</span></span> <span data-ttu-id="27c6d-129"><xref:System.IO.StreamWriter.WriteLine%2A>然后，可以使用方法，如以上其他示例中所示。</span><span class="sxs-lookup"><span data-stu-id="27c6d-129">The <xref:System.IO.StreamWriter.WriteLine%2A> method can then be used as in the other examples above.</span></span> <span data-ttu-id="27c6d-130">请记住，在释放上下文时释放编写器，以确保文件完全关闭。</span><span class="sxs-lookup"><span data-stu-id="27c6d-130">Remember to ensure the file is closed cleanly by disposing the writer when the context is disposed.</span></span> <span data-ttu-id="27c6d-131">例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-131">For example:</span></span>

<!--
    private readonly StreamWriter _logStream = new StreamWriter("mylog.txt", append: true); 

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(_logStream.WriteLine);

    public override void Dispose()
    {
        base.Dispose();
        _logStream.Dispose();
    }
    
    public override async ValueTask DisposeAsync()
    {
        await base.DisposeAsync();
        await _logStream.DisposeAsync();
    }
-->
[!code-csharp[LogToFile](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=LogToFile)]

> [!TIP]
> <span data-ttu-id="27c6d-132">请考虑使用 [Microsoft Extensions](/aspnet/core/fundamentals/logging) 日志记录日志记录到生产应用程序中的文件。</span><span class="sxs-lookup"><span data-stu-id="27c6d-132">Consider using [Microsoft.Extensions.Logging](/aspnet/core/fundamentals/logging) for logging to files in production applications.</span></span>

## <a name="getting-detailed-messages"></a><span data-ttu-id="27c6d-133">获取详细消息</span><span class="sxs-lookup"><span data-stu-id="27c6d-133">Getting detailed messages</span></span>

### <a name="sensitive-data"></a><span data-ttu-id="27c6d-134">敏感数据</span><span class="sxs-lookup"><span data-stu-id="27c6d-134">Sensitive data</span></span>

<span data-ttu-id="27c6d-135">默认情况下，EF Core 不会将任何数据的值包含在异常消息中。</span><span class="sxs-lookup"><span data-stu-id="27c6d-135">By default, EF Core will not include the values of any data in exception messages.</span></span> <span data-ttu-id="27c6d-136">这是因为此类数据可能是机密的，如果未处理异常，则可能会在生产中显示这些数据。</span><span class="sxs-lookup"><span data-stu-id="27c6d-136">This is because such data may be confidential, and could be revealed in production use if an exception is not handled.</span></span>

<span data-ttu-id="27c6d-137">但是，在调试时，知道数据值（特别是对于密钥）会非常有用。</span><span class="sxs-lookup"><span data-stu-id="27c6d-137">However, knowing data values, especially for keys, can be very helpful when debugging.</span></span> <span data-ttu-id="27c6d-138">可以通过调用在 EF Core 中启用此功能 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging> 。</span><span class="sxs-lookup"><span data-stu-id="27c6d-138">This can be enabled in EF Core by calling <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging>.</span></span> <span data-ttu-id="27c6d-139">例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-139">For example:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .LogTo(Console.WriteLine)
            .EnableSensitiveDataLogging();
-->
[!code-csharp[SensitiveDataLogging](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=SensitiveDataLogging)]

### <a name="detailed-query-exceptions"></a><span data-ttu-id="27c6d-140">详细查询异常</span><span class="sxs-lookup"><span data-stu-id="27c6d-140">Detailed query exceptions</span></span>

<span data-ttu-id="27c6d-141">出于性能方面的考虑，EF Core 不会在 try-catch 块中包装每次调用以读取数据库提供程序的值。</span><span class="sxs-lookup"><span data-stu-id="27c6d-141">For performance reasons, EF Core does not wrap each call to read a value from the database provider in a try-catch block.</span></span> <span data-ttu-id="27c6d-142">但是，这有时会导致难以诊断的异常，尤其是在数据库不允许的情况下，当模型不允许时，它们会返回 NULL。</span><span class="sxs-lookup"><span data-stu-id="27c6d-142">However, this sometimes results in exceptions that are hard to diagnose, especially when the database returns a NULL when not allowed by the model.</span></span>

<span data-ttu-id="27c6d-143">启用 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableDetailedErrors%2A> 将导致 EF 引入这些 try-catch 块，从而提供更详细的错误。</span><span class="sxs-lookup"><span data-stu-id="27c6d-143">Turning on <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableDetailedErrors%2A> will cause EF to introduce these try-catch blocks and thereby provide more detailed errors.</span></span> <span data-ttu-id="27c6d-144">例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-144">For example:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .LogTo(Console.WriteLine)
            .EnableDetailedErrors();
-->
[!code-csharp[EnableDetailedErrors](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=EnableDetailedErrors)]

## <a name="filtering"></a><span data-ttu-id="27c6d-145">筛选</span><span class="sxs-lookup"><span data-stu-id="27c6d-145">Filtering</span></span>

### <a name="log-levels"></a><span data-ttu-id="27c6d-146">日志级别</span><span class="sxs-lookup"><span data-stu-id="27c6d-146">Log levels</span></span>

<span data-ttu-id="27c6d-147">每 EF Core 日志消息都分配给枚举定义的级别 <xref:Microsoft.Extensions.Logging.LogLevel> 。</span><span class="sxs-lookup"><span data-stu-id="27c6d-147">Every EF Core log message is assigned to a level defined by the <xref:Microsoft.Extensions.Logging.LogLevel> enum.</span></span> <span data-ttu-id="27c6d-148">默认情况下，EF Core 简单日志记录在 `Debug` 级别或更高级别包含每条消息。</span><span class="sxs-lookup"><span data-stu-id="27c6d-148">By default, EF Core simple logging includes every message at `Debug` level or above.</span></span> <span data-ttu-id="27c6d-149">`LogTo` 可向传递较高的最低级别以筛选出某些消息。</span><span class="sxs-lookup"><span data-stu-id="27c6d-149">`LogTo` can be passed a higher minimum level to filter out some messages.</span></span> <span data-ttu-id="27c6d-150">例如，传递将 `Information` 导致限制为数据库访问和某些内务处理消息的最小日志集。</span><span class="sxs-lookup"><span data-stu-id="27c6d-150">For example, passing `Information` results in a minimal set of logs limited to database access and some housekeeping messages.</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(Console.WriteLine, LogLevel.Information);
-->
[!code-csharp[InfoOnly](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=InfoOnly)]

### <a name="specific-messages"></a><span data-ttu-id="27c6d-151">特定消息</span><span class="sxs-lookup"><span data-stu-id="27c6d-151">Specific messages</span></span>

<span data-ttu-id="27c6d-152">每条日志消息都分配有一个 <xref:Microsoft.Extensions.Logging.EventId> 。</span><span class="sxs-lookup"><span data-stu-id="27c6d-152">Every log message is assigned an <xref:Microsoft.Extensions.Logging.EventId>.</span></span> <span data-ttu-id="27c6d-153">这些 Id 可以从 <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId> 类或 <xref:Microsoft.EntityFrameworkCore.Diagnostics.RelationalEventId> 类用于关系特定消息。</span><span class="sxs-lookup"><span data-stu-id="27c6d-153">These IDs can be accessed from the <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId> class or the <xref:Microsoft.EntityFrameworkCore.Diagnostics.RelationalEventId> class for relational-specific messages.</span></span> <span data-ttu-id="27c6d-154">数据库提供程序还可以在类似类中具有特定于提供程序的 Id。</span><span class="sxs-lookup"><span data-stu-id="27c6d-154">A database provider may also have provider-specific IDs in a similar class.</span></span> <span data-ttu-id="27c6d-155">例如， <xref:Microsoft.EntityFrameworkCore.Diagnostics.SqlServerEventId> 对于 SQL Server 提供程序。</span><span class="sxs-lookup"><span data-stu-id="27c6d-155">For example, <xref:Microsoft.EntityFrameworkCore.Diagnostics.SqlServerEventId> for the SQL Server provider.</span></span>

<span data-ttu-id="27c6d-156">`LogTo` 可以配置为仅记录与一个或多个事件 Id 相关联的消息。</span><span class="sxs-lookup"><span data-stu-id="27c6d-156">`LogTo` can be configured to only log the messages associated with one or more event IDs.</span></span> <span data-ttu-id="27c6d-157">例如，若要只记录要初始化或释放的上下文的消息，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="27c6d-157">For example, to log only messages for the context being initialized or disposed:</span></span>  

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .LogTo(Console.WriteLine, new[] { CoreEventId.ContextDisposed, CoreEventId.ContextInitialized });
-->
[!code-csharp[EventIds](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=EventIds)]

### <a name="message-categories"></a><span data-ttu-id="27c6d-158">消息类别</span><span class="sxs-lookup"><span data-stu-id="27c6d-158">Message categories</span></span>

<span data-ttu-id="27c6d-159">每条日志消息都分配到一个命名的分层记录器类别。</span><span class="sxs-lookup"><span data-stu-id="27c6d-159">Every log message is assigned to a named hierarchical logger category.</span></span> <span data-ttu-id="27c6d-160">类别为：</span><span class="sxs-lookup"><span data-stu-id="27c6d-160">The categories are:</span></span>

| <span data-ttu-id="27c6d-161">类别</span><span class="sxs-lookup"><span data-stu-id="27c6d-161">Category</span></span>                                             | <span data-ttu-id="27c6d-162">消息</span><span class="sxs-lookup"><span data-stu-id="27c6d-162">Messages</span></span>
|:-----------------------------------------------------|-------------------------------------------------
| <span data-ttu-id="27c6d-163">Microsoft.EntityFrameworkCore</span><span class="sxs-lookup"><span data-stu-id="27c6d-163">Microsoft.EntityFrameworkCore</span></span>                        | <span data-ttu-id="27c6d-164">所有 EF Core 消息</span><span class="sxs-lookup"><span data-stu-id="27c6d-164">All EF Core messages</span></span>
| <span data-ttu-id="27c6d-165">Microsoft.entityframeworkcore</span><span class="sxs-lookup"><span data-stu-id="27c6d-165">Microsoft.EntityFrameworkCore.Database</span></span>               | <span data-ttu-id="27c6d-166">所有数据库交互</span><span class="sxs-lookup"><span data-stu-id="27c6d-166">All database interactions</span></span>
| <span data-ttu-id="27c6d-167">Microsoft.entityframeworkcore。连接</span><span class="sxs-lookup"><span data-stu-id="27c6d-167">Microsoft.EntityFrameworkCore.Database.Connection</span></span>    | <span data-ttu-id="27c6d-168">使用数据库连接</span><span class="sxs-lookup"><span data-stu-id="27c6d-168">Uses of a database connection</span></span>
| <span data-ttu-id="27c6d-169">Microsoft.entityframeworkcore。</span><span class="sxs-lookup"><span data-stu-id="27c6d-169">Microsoft.EntityFrameworkCore.Database.Command</span></span>       | <span data-ttu-id="27c6d-170">使用数据库命令</span><span class="sxs-lookup"><span data-stu-id="27c6d-170">Uses of a database command</span></span>
| <span data-ttu-id="27c6d-171">Microsoft.entityframeworkcore 事务</span><span class="sxs-lookup"><span data-stu-id="27c6d-171">Microsoft.EntityFrameworkCore.Database.Transaction</span></span>   | <span data-ttu-id="27c6d-172">使用数据库事务</span><span class="sxs-lookup"><span data-stu-id="27c6d-172">Uses of a database transaction</span></span>
| <span data-ttu-id="27c6d-173">Microsoft.entityframeworkcore。更新</span><span class="sxs-lookup"><span data-stu-id="27c6d-173">Microsoft.EntityFrameworkCore.Update</span></span>                 | <span data-ttu-id="27c6d-174">保存实体，排除数据库交互</span><span class="sxs-lookup"><span data-stu-id="27c6d-174">Saving entities, excluding database interactions</span></span>
| <span data-ttu-id="27c6d-175">Microsoft.entityframeworkcore</span><span class="sxs-lookup"><span data-stu-id="27c6d-175">Microsoft.EntityFrameworkCore.Model</span></span>                  | <span data-ttu-id="27c6d-176">所有模型和元数据交互</span><span class="sxs-lookup"><span data-stu-id="27c6d-176">All model and metadata interactions</span></span>
| <span data-ttu-id="27c6d-177">Microsoft.entityframeworkcore。验证</span><span class="sxs-lookup"><span data-stu-id="27c6d-177">Microsoft.EntityFrameworkCore.Model.Validation</span></span>       | <span data-ttu-id="27c6d-178">模型验证</span><span class="sxs-lookup"><span data-stu-id="27c6d-178">Model validation</span></span>
| <span data-ttu-id="27c6d-179">Microsoft.entityframeworkcore</span><span class="sxs-lookup"><span data-stu-id="27c6d-179">Microsoft.EntityFrameworkCore.Query</span></span>                  | <span data-ttu-id="27c6d-180">查询，排除数据库交互</span><span class="sxs-lookup"><span data-stu-id="27c6d-180">Queries, excluding database interactions</span></span>
| <span data-ttu-id="27c6d-181">Microsoft.entityframeworkcore</span><span class="sxs-lookup"><span data-stu-id="27c6d-181">Microsoft.EntityFrameworkCore.Infrastructure</span></span>         | <span data-ttu-id="27c6d-182">常规事件，例如上下文创建</span><span class="sxs-lookup"><span data-stu-id="27c6d-182">General events, such as context creation</span></span>
| <span data-ttu-id="27c6d-183">Microsoft.entityframeworkcore 基架</span><span class="sxs-lookup"><span data-stu-id="27c6d-183">Microsoft.EntityFrameworkCore.Scaffolding</span></span>            | <span data-ttu-id="27c6d-184">数据库反向工程</span><span class="sxs-lookup"><span data-stu-id="27c6d-184">Database reverse engineering</span></span>
| <span data-ttu-id="27c6d-185">Microsoft.entityframeworkcore</span><span class="sxs-lookup"><span data-stu-id="27c6d-185">Microsoft.EntityFrameworkCore.Migrations</span></span>             | <span data-ttu-id="27c6d-186">迁移</span><span class="sxs-lookup"><span data-stu-id="27c6d-186">Migrations</span></span>
| <span data-ttu-id="27c6d-187">Microsoft.entityframeworkcore. 更改跟踪</span><span class="sxs-lookup"><span data-stu-id="27c6d-187">Microsoft.EntityFrameworkCore.ChangeTracking</span></span>         | <span data-ttu-id="27c6d-188">更改跟踪交互</span><span class="sxs-lookup"><span data-stu-id="27c6d-188">Change tracking interactions</span></span>

<span data-ttu-id="27c6d-189">`LogTo` 可以配置为仅记录来自一个或多个类别的消息。</span><span class="sxs-lookup"><span data-stu-id="27c6d-189">`LogTo` can be configured to only log the messages from one or more categories.</span></span> <span data-ttu-id="27c6d-190">例如，若要只记录数据库交互：</span><span class="sxs-lookup"><span data-stu-id="27c6d-190">For example, to log only database interactions:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .LogTo(Console.WriteLine, new[] { DbLoggerCategory.Database.Name });
-->
[!code-csharp[DatabaseCategory](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=DatabaseCategory)]

<span data-ttu-id="27c6d-191">请注意， <xref:Microsoft.EntityFrameworkCore.DbLoggerCategory> 类提供了一个分层 API 用于查找类别，并避免了对字符串进行硬编码。</span><span class="sxs-lookup"><span data-stu-id="27c6d-191">Notice that the <xref:Microsoft.EntityFrameworkCore.DbLoggerCategory> class provides a hierarchical API for finding a category and avoids the need to hard-code strings.</span></span>

<span data-ttu-id="27c6d-192">由于类别具有层次结构，因此使用类别的此示例 `Database` 将包括子类别、和的所有消息 `Database.Connection` `Database.Command` `Database.Transaction` 。</span><span class="sxs-lookup"><span data-stu-id="27c6d-192">Since categories are hierarchical, this example using the `Database` category will include all messages for the subcategories `Database.Connection`, `Database.Command`, and `Database.Transaction`.</span></span>

### <a name="custom-filters"></a><span data-ttu-id="27c6d-193">自定义筛选器</span><span class="sxs-lookup"><span data-stu-id="27c6d-193">Custom filters</span></span>

<span data-ttu-id="27c6d-194">`LogTo` 允许使用自定义筛选器，用于上述筛选选项均不足的情况。</span><span class="sxs-lookup"><span data-stu-id="27c6d-194">`LogTo` allows a custom filter to be used for cases where none of the filtering options above are sufficient.</span></span> <span data-ttu-id="27c6d-195">例如，若要记录任何级别 `Information` 或更高级别的消息，以及用于打开和关闭连接的消息：</span><span class="sxs-lookup"><span data-stu-id="27c6d-195">For example, to log any message at level `Information` or above, as well as messages for opening and closing a connection:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .LogTo(
                Console.WriteLine,
                (eventId, logLevel) => logLevel >= LogLevel.Information
                                       || eventId == RelationalEventId.ConnectionOpened
                                       || eventId == RelationalEventId.ConnectionClosed);
-->
[!code-csharp[CustomFilter](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=CustomFilter)]

> [!TIP]
> <span data-ttu-id="27c6d-196">使用自定义筛选器或使用此处所示的任何其他选项进行筛选比委托中的筛选更有效 `LogTo` 。</span><span class="sxs-lookup"><span data-stu-id="27c6d-196">Filtering using custom filters or using any of the other options shown here is more efficient than filtering in the `LogTo` delegate.</span></span> <span data-ttu-id="27c6d-197">这是因为如果筛选器确定不应记录消息，则不会创建日志消息。</span><span class="sxs-lookup"><span data-stu-id="27c6d-197">This is because if the filter determines the message should not be logged, then the log message is not even created.</span></span>

## <a name="configuration-for-specific-messages"></a><span data-ttu-id="27c6d-198">特定消息的配置</span><span class="sxs-lookup"><span data-stu-id="27c6d-198">Configuration for specific messages</span></span>

<span data-ttu-id="27c6d-199">EF Core <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.ConfigureWarnings%2A> API 允许应用程序更改遇到特定事件时所发生的情况。</span><span class="sxs-lookup"><span data-stu-id="27c6d-199">The EF Core <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.ConfigureWarnings%2A> API allows applications to change what happens when a specific event is encountered.</span></span> <span data-ttu-id="27c6d-200">这可用于：</span><span class="sxs-lookup"><span data-stu-id="27c6d-200">This can be used to:</span></span>

* <span data-ttu-id="27c6d-201">更改记录事件的日志级别</span><span class="sxs-lookup"><span data-stu-id="27c6d-201">Change the log level at which the event is logged</span></span>
* <span data-ttu-id="27c6d-202">完全跳过记录事件</span><span class="sxs-lookup"><span data-stu-id="27c6d-202">Skip logging the event altogether</span></span>
* <span data-ttu-id="27c6d-203">事件发生时引发异常</span><span class="sxs-lookup"><span data-stu-id="27c6d-203">Throw an exception when the event occurs</span></span>

### <a name="changing-the-log-level-for-an-event"></a><span data-ttu-id="27c6d-204">更改事件的日志级别</span><span class="sxs-lookup"><span data-stu-id="27c6d-204">Changing the log level for an event</span></span>

<span data-ttu-id="27c6d-205">前面的示例使用自定义筛选器来记录每条消息以及为 `LogLevel.Information` 定义的两个事件 `LogLevel.Debug` 。</span><span class="sxs-lookup"><span data-stu-id="27c6d-205">The previous example used a custom filter to log every message at `LogLevel.Information` as well as two events defined for `LogLevel.Debug`.</span></span> <span data-ttu-id="27c6d-206">可以通过将两个事件的日志级别更改 `Debug` 为来实现相同的 `Information` 操作：</span><span class="sxs-lookup"><span data-stu-id="27c6d-206">The same can be achieved by changing the log level of the two `Debug` events to `Information`:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .ConfigureWarnings(b => b.Log(
                (RelationalEventId.ConnectionOpened, LogLevel.Information),
                (RelationalEventId.ConnectionClosed, LogLevel.Information)))
            .LogTo(Console.WriteLine, LogLevel.Information);
-->
[!code-csharp[ChangeLogLevel](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=ChangeLogLevel)]

### <a name="suppress-logging-an-event"></a><span data-ttu-id="27c6d-207">禁止记录事件</span><span class="sxs-lookup"><span data-stu-id="27c6d-207">Suppress logging an event</span></span>

<span data-ttu-id="27c6d-208">同样，可以从日志记录中抑制单个事件。</span><span class="sxs-lookup"><span data-stu-id="27c6d-208">In a similar way, an individual event can be suppressed from logging.</span></span> <span data-ttu-id="27c6d-209">这对于忽略已评审和理解的警告特别有用。</span><span class="sxs-lookup"><span data-stu-id="27c6d-209">This is particularly useful for ignoring a warning that has been reviewed and understood.</span></span> <span data-ttu-id="27c6d-210">例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-210">For example:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .ConfigureWarnings(b => b.Ignore(CoreEventId.DetachedLazyLoadingWarning))
            .LogTo(Console.WriteLine);
-->
[!code-csharp[SuppressMessage](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=SuppressMessage)]

### <a name="throw-for-an-event"></a><span data-ttu-id="27c6d-211">引发事件</span><span class="sxs-lookup"><span data-stu-id="27c6d-211">Throw for an event</span></span>

<span data-ttu-id="27c6d-212">最后，可以将 EF Core 配置为引发给定事件。</span><span class="sxs-lookup"><span data-stu-id="27c6d-212">Finally, EF Core can be configured to throw for a given event.</span></span> <span data-ttu-id="27c6d-213">这对于将警告更改为错误特别有用。</span><span class="sxs-lookup"><span data-stu-id="27c6d-213">This is particularly useful for changing a warning into an error.</span></span> <span data-ttu-id="27c6d-214"> (确实，这是方法的原始用途 `ConfigureWarnings` ，因此是名称。 ) 例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-214">(Indeed, this was the original purpose of `ConfigureWarnings` method, hence the name.) For example:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .ConfigureWarnings(b => b.Throw(RelationalEventId.MultipleCollectionIncludeWarning))
            .LogTo(Console.WriteLine);
-->
[!code-csharp[ThrowForEvent](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=ThrowForEvent)]

## <a name="message-contents-and-formatting"></a><span data-ttu-id="27c6d-215">消息内容和格式设置</span><span class="sxs-lookup"><span data-stu-id="27c6d-215">Message contents and formatting</span></span>

<span data-ttu-id="27c6d-216">中的默认内容 `LogTo` 是跨多行格式设置的。</span><span class="sxs-lookup"><span data-stu-id="27c6d-216">The default content from `LogTo` is formatted across multiple lines.</span></span> <span data-ttu-id="27c6d-217">第一行包含消息元数据：</span><span class="sxs-lookup"><span data-stu-id="27c6d-217">The first line contains message metadata:</span></span>

* <span data-ttu-id="27c6d-218"><xref:Microsoft.Extensions.Logging.LogLevel>作为四字符前缀</span><span class="sxs-lookup"><span data-stu-id="27c6d-218">The <xref:Microsoft.Extensions.Logging.LogLevel> as a four-character prefix</span></span>
* <span data-ttu-id="27c6d-219">为当前区域性设置格式的本地时间戳</span><span class="sxs-lookup"><span data-stu-id="27c6d-219">A local timestamp, formatted for the current culture</span></span>
* <span data-ttu-id="27c6d-220"><xref:Microsoft.Extensions.Logging.EventId>可以复制/粘贴以便从或其他某个类获取成员的窗体中的 <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId> `EventId` ，以及原始 ID 值</span><span class="sxs-lookup"><span data-stu-id="27c6d-220">The <xref:Microsoft.Extensions.Logging.EventId> in the form that can be copy/pasted to get the member from <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId> or one of the other `EventId` classes, plus the raw ID value</span></span>
* <span data-ttu-id="27c6d-221">事件类别，如上所述。</span><span class="sxs-lookup"><span data-stu-id="27c6d-221">The event category, as described above.</span></span>

<span data-ttu-id="27c6d-222">例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-222">For example:</span></span>

```output
info: 10/6/2020 10:52:45.581 RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command)
      Executed DbCommand (0ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      CREATE TABLE "Blogs" (
          "Id" INTEGER NOT NULL CONSTRAINT "PK_Blogs" PRIMARY KEY AUTOINCREMENT,
          "Name" INTEGER NOT NULL
      );
dbug: 10/6/2020 10:52:45.582 RelationalEventId.TransactionCommitting[20210] (Microsoft.EntityFrameworkCore.Database.Transaction)
      Committing transaction.
dbug: 10/6/2020 10:52:45.585 RelationalEventId.TransactionCommitted[20202] (Microsoft.EntityFrameworkCore.Database.Transaction)
      Committed transaction.
```

<span data-ttu-id="27c6d-223">可以通过传递中的值来自定义此内容 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbContextLoggerOptions> ，如以下部分所示。</span><span class="sxs-lookup"><span data-stu-id="27c6d-223">This content can be customized by passing values from <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbContextLoggerOptions>, as shown in the following sections.</span></span>

> [!TIP]
> <span data-ttu-id="27c6d-224">请考虑使用 [Microsoft Extensions](/aspnet/core/fundamentals/logging) 日志来更好地控制日志格式。</span><span class="sxs-lookup"><span data-stu-id="27c6d-224">Consider using [Microsoft.Extensions.Logging](/aspnet/core/fundamentals/logging) for more control over log formatting.</span></span>

### <a name="using-utc-time"></a><span data-ttu-id="27c6d-225">使用 UTC 时间</span><span class="sxs-lookup"><span data-stu-id="27c6d-225">Using UTC time</span></span>

<span data-ttu-id="27c6d-226">默认情况下，时间戳在调试时用于本地使用。</span><span class="sxs-lookup"><span data-stu-id="27c6d-226">By default, timestamps are designed for local consumption while debugging.</span></span> <span data-ttu-id="27c6d-227">使用 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbContextLoggerOptions.DefaultWithUtcTime?displayProperty=nameWithType> 可以改为使用与区域性无关的 UTC 时间戳，但保留所有其他内容。</span><span class="sxs-lookup"><span data-stu-id="27c6d-227">Use <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbContextLoggerOptions.DefaultWithUtcTime?displayProperty=nameWithType> to use culture-agnostic UTC timestamps instead, but keep everything else the same.</span></span> <span data-ttu-id="27c6d-228">例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-228">For example:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(
            Console.WriteLine,
            LogLevel.Debug,
            DbContextLoggerOptions.DefaultWithUtcTime);
-->
[!code-csharp[Utc](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=Utc)]

<span data-ttu-id="27c6d-229">此示例将生成以下日志格式：</span><span class="sxs-lookup"><span data-stu-id="27c6d-229">This example results in the following log formatting:</span></span>

```output
info: 2020-10-06T17:55:39.0333701Z RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command)
      Executed DbCommand (0ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      CREATE TABLE "Blogs" (
          "Id" INTEGER NOT NULL CONSTRAINT "PK_Blogs" PRIMARY KEY AUTOINCREMENT,
          "Name" INTEGER NOT NULL
      );
dbug: 2020-10-06T17:55:39.0333892Z RelationalEventId.TransactionCommitting[20210] (Microsoft.EntityFrameworkCore.Database.Transaction)
      Committing transaction.
dbug: 2020-10-06T17:55:39.0351684Z RelationalEventId.TransactionCommitted[20202] (Microsoft.EntityFrameworkCore.Database.Transaction)
      Committed transaction.
```

### <a name="single-line-logging"></a><span data-ttu-id="27c6d-230">单行日志记录</span><span class="sxs-lookup"><span data-stu-id="27c6d-230">Single line logging</span></span>

<span data-ttu-id="27c6d-231">有时，每条日志消息只获取一行很有用。</span><span class="sxs-lookup"><span data-stu-id="27c6d-231">Sometimes it is useful to get exactly one line per log message.</span></span> <span data-ttu-id="27c6d-232">这可以由启用 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbContextLoggerOptions.SingleLine?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="27c6d-232">This can be enabled by <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbContextLoggerOptions.SingleLine?displayProperty=nameWithType>.</span></span> <span data-ttu-id="27c6d-233">例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-233">For example:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(
            Console.WriteLine,
            LogLevel.Debug,
            DbContextLoggerOptions.DefaultWithLocalTime | DbContextLoggerOptions.SingleLine);
-->
[!code-csharp[SingleLine](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=SingleLine)]

<span data-ttu-id="27c6d-234">此示例将生成以下日志格式：</span><span class="sxs-lookup"><span data-stu-id="27c6d-234">This example results in the following log formatting:</span></span>

```output
info: 10/6/2020 10:52:45.723 RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command) -> Executed DbCommand (0ms) [Parameters=[], CommandType='Text', CommandTimeout='30']CREATE TABLE "Blogs" (    "Id" INTEGER NOT NULL CONSTRAINT "PK_Blogs" PRIMARY KEY AUTOINCREMENT,    "Name" INTEGER NOT NULL);
dbug: 10/6/2020 10:52:45.723 RelationalEventId.TransactionCommitting[20210] (Microsoft.EntityFrameworkCore.Database.Transaction) -> Committing transaction.
dbug: 10/6/2020 10:52:45.725 RelationalEventId.TransactionCommitted[20202] (Microsoft.EntityFrameworkCore.Database.Transaction) -> Committed transaction.
```

### <a name="other-content-options"></a><span data-ttu-id="27c6d-235">其他内容选项</span><span class="sxs-lookup"><span data-stu-id="27c6d-235">Other content options</span></span>

<span data-ttu-id="27c6d-236">中的其他标志 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbContextLoggerOptions> 可用于修整日志中包含的元数据量。</span><span class="sxs-lookup"><span data-stu-id="27c6d-236">Other flags in <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbContextLoggerOptions> can be used to trim down the amount of metadata included in the log.</span></span> <span data-ttu-id="27c6d-237">这对于单行日志记录很有用。</span><span class="sxs-lookup"><span data-stu-id="27c6d-237">This is can be useful in conjunction with single-line logging.</span></span> <span data-ttu-id="27c6d-238">例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-238">For example:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(
            Console.WriteLine,
            LogLevel.Debug,
            DbContextLoggerOptions.UtcTime | DbContextLoggerOptions.SingleLine);
-->
[!code-csharp[TerseLogs](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=TerseLogs)]

<span data-ttu-id="27c6d-239">此示例将生成以下日志格式：</span><span class="sxs-lookup"><span data-stu-id="27c6d-239">This example results in the following log formatting:</span></span>

```output
2020-10-06T17:52:45.7320362Z -> Executed DbCommand (0ms) [Parameters=[], CommandType='Text', CommandTimeout='30']CREATE TABLE "Blogs" (    "Id" INTEGER NOT NULL CONSTRAINT "PK_Blogs" PRIMARY KEY AUTOINCREMENT,    "Name" INTEGER NOT NULL);
2020-10-06T17:52:45.7320531Z -> Committing transaction.
2020-10-06T17:52:45.7339441Z -> Committed transaction.
```

## <a name="moving-from-ef6"></a><span data-ttu-id="27c6d-240">从 EF6 移动</span><span class="sxs-lookup"><span data-stu-id="27c6d-240">Moving from EF6</span></span>

<span data-ttu-id="27c6d-241">EF Core 简单日志记录 <xref:System.Data.Entity.Database.Log?displayProperty=nameWithType> 在以下两个重要方面不同于 EF6：</span><span class="sxs-lookup"><span data-stu-id="27c6d-241">EF Core simple logging differs from <xref:System.Data.Entity.Database.Log?displayProperty=nameWithType> in EF6 in two important ways:</span></span>

* <span data-ttu-id="27c6d-242">日志消息不仅限于数据库交互</span><span class="sxs-lookup"><span data-stu-id="27c6d-242">Log messages are not limited to only database interactions</span></span>
* <span data-ttu-id="27c6d-243">必须在上下文初始化时配置日志记录</span><span class="sxs-lookup"><span data-stu-id="27c6d-243">The logging must be configured at context initialization time</span></span>

<span data-ttu-id="27c6d-244">对于第一个差异，可以使用上述筛选来限制要记录的消息。</span><span class="sxs-lookup"><span data-stu-id="27c6d-244">For the first difference, the filtering described above can be used to limit which messages are logged.</span></span>

<span data-ttu-id="27c6d-245">第二个差异是指在不需要日志消息时不生成日志消息，从而提高性能的故意更改。</span><span class="sxs-lookup"><span data-stu-id="27c6d-245">The second difference is an intentional change to improve performance by not generating log messages when they are not needed.</span></span> <span data-ttu-id="27c6d-246">但是，仍然可以通过 `Log` 在上创建一个属性 `DbContext` ，然后仅在设置该属性时使用该属性，来使 EF6 的行为类似。</span><span class="sxs-lookup"><span data-stu-id="27c6d-246">However, it is still possible to get a similar behavior to EF6 by creating a `Log` property on your `DbContext` and then using it only when it has been set.</span></span> <span data-ttu-id="27c6d-247">例如：</span><span class="sxs-lookup"><span data-stu-id="27c6d-247">For example:</span></span>

<!--
    public Action<string> Log { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(s => Log?.Invoke(s));
-->
[!code-csharp[DatabaseLog](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=DatabaseLog)]
