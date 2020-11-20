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
# <a name="interceptors"></a>侦听器

Entity Framework Core (EF Core) 拦截允许拦截、修改和/或禁止 EF Core 操作。 这包括低级数据库操作（例如执行命令）以及高级别操作（例如对 SaveChanges 的调用）。

侦听器与日志记录和诊断的不同之处在于，它们允许修改或抑制所侦听的操作。 [简单日志记录](xref:core/logging-events-diagnostics/simple-logging)或 [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) 是日志记录的更好选择。

配置上下文时，将按 DbContext 实例注册侦听器。 使用[诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)获取相同的信息，但要获取进程中所有 DbContext 实例的信息。

## <a name="registering-interceptors"></a>正在注册侦听器

<xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.AddInterceptors%2A>在[配置 DbContext 实例](xref:core/dbcontext-configuration/index)时，将使用注册侦听器。 通常在重写中完成此操作 <xref:Microsoft.EntityFrameworkCore.DbContext.OnConfiguring%2A?displayProperty=nameWithType> 。 例如：

<!--
public class ExampleContext : BlogsContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder) 
        => optionsBuilder.AddInterceptors(new TaggedQueryCommandInterceptor());
}
-->
[!code-csharp[RegisterInterceptor](../../../samples/core/Miscellaneous/CommandInterception/Program.cs?name=RegisterInterceptor)]

另外， `AddInterceptors` 还可以在 <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> 创建 <xref:Microsoft.EntityFrameworkCore.DbContextOptions> 要传递到 DbContext 构造函数的实例时作为或的一部分来调用。

> [!TIP]
> 当使用 AddDbContext 或将 DbContextOptions 实例传递到 DbContext 构造函数时，仍将调用 OnConfiguring。 这使得它成为应用上下文配置的理想位置，而不考虑如何构造 DbContext。

拦截程序通常是无状态的，这意味着单个侦听器实例可用于所有 DbContext 实例。 例如：

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

每个侦听器实例都必须实现一个或多个派生自的接口 <xref:Microsoft.EntityFrameworkCore.Diagnostics.IInterceptor> 。 每个实例只能注册一次，即使它实现了多个截获接口;EF Core 将根据需要为每个接口路由事件。

## <a name="database-interception"></a>数据库拦截

> [!NOTE]
> 数据库拦截是在 EF Core 3.0 中引入的，并且仅适用于关系数据库提供程序。
> EF Core 5.0 中引入了保存点支持。

低级别数据库截取拆分为下表中显示的三个接口。

| 截获                                                            | 已截获数据库操作
|:-----------------------------------------------------------------------|-------------------------------------------------
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbCommandInterceptor> | 创建命令</br>执行命令</br>命令失败</br>释放命令的 DbDataReader
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbConnectionInterceptor> | 打开和关闭连接</br>连接失败
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbTransactionInterceptor> | 创建事务</br>使用现有事务</br>提交事务</br>回退事务</br>创建和使用保存点</br>事务失败

基类 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor> 、 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbConnectionInterceptor> 和 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbTransactionInterceptor> 包含对应接口中每个方法的无操作实现。 使用基类，以避免实现未使用的截取方法。

每个侦听器类型上的方法都成对出现，第一种方法是在数据库操作开始之前调用，第二个是在操作完成后调用。 例如， 例如，在 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor.ReaderExecuting%2A?displayProperty=nameWithType> 执行查询之前调用，并 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor.ReaderExecuted%2A?displayProperty=nameWithType> 在将查询发送到数据库后调用。

每对方法都有同步和异步变体。 这允许异步 i/o，如请求访问令牌，作为截获异步数据库操作的一部分发生。

### <a name="example-command-interception-to-add-query-hints"></a>示例：命令侦听以添加查询提示

> [!TIP]  
> 可以从 GitHub [下载命令侦听器示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/CommandInterception) 。

在将 <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbCommandInterceptor> SQL 发送到数据库之前，可以使用对其进行修改。 此示例演示如何修改 SQL 以包含查询提示。

通常，截获的最棘手部分是确定命令与需要修改的查询的对应情况。 分析 SQL 是一种方法，但这种做法往往非常脆弱。 另一种方法是使用 [EF Core 查询标记](xref:core/querying/tags) 来标记每个应修改的查询。 例如：

<!--
            var blogs1 = context.Blogs.TagWith("Use hint: robust plan").ToList();
-->
[!code-csharp[TaggedQuery](../../../samples/core/Miscellaneous/CommandInterception/Program.cs?name=TaggedQuery)]

然后，可在侦听器中检测到此标记，因为它将始终作为注释包含在命令文本的第一行中。 检测到标记时，会修改查询 SQL 以添加相应的提示：

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

注意：

* 侦听器从继承 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbCommandInterceptor> ，以避免必须实现侦听器接口中的每个方法。
* 侦听器同时实现同步和异步方法。 这可确保将相同的查询提示应用于同步和异步查询。
* 侦听器通过在将 `Executing` 生成的 SQL 发送到数据库 _之前_ ，实现 EF Core 调用的方法。 与 `Executed` 方法相比，在数据库调用返回之后调用这些方法。

在此示例中运行代码将在标记查询时生成以下内容：

```sql
-- Use hint: robust plan

SELECT [b].[Id], [b].[Name]
FROM [Blogs] AS [b] OPTION (ROBUST PLAN)
```

另一方面，如果查询未标记，则会将其发送到未修改的数据库：

```sql
SELECT [b].[Id], [b].[Name]
FROM [Blogs] AS [b]
```

### <a name="example-connection-interception-for-sql-azure-authentication-using-add"></a>示例：使用 ADD 进行 SQL Azure 身份验证的连接拦截

> [!TIP]  
> 可以从 GitHub [下载连接侦听器示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/ConnectionInterception) 。

在 <xref:Microsoft.EntityFrameworkCore.Diagnostics.IDbConnectionInterceptor> <xref:System.Data.Common.DbConnection> 用于连接到数据库之前，可以使用来操作。 这可用于获取 AAD) 访问令牌 (Azure Active Directory。 例如：

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
> [SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient/) 现在支持通过连接字符串进行 AAD 身份验证。 有关更多信息，请参见<xref:Microsoft.Data.SqlClient.SqlAuthenticationMethod>。

> [!WARNING]
> 请注意，如果对打开连接进行了同步调用，则侦听器将引发。 这是因为不存在用于获取访问令牌的非异步方法，并且没有 [从非异步上下文调用异步方法而不会造成死锁的通用](https://devblogs.microsoft.com/dotnet/configureawait-faq/)方法。

> [!WARNING]
> 在某些情况下，无法自动缓存访问令牌。 根据所请求的令牌类型，可能需要在此处实现自己的缓存。

### <a name="example-advanced-command-interception-for-caching"></a>示例：缓存的高级命令拦截

> [!TIP]  
> 可以从 GitHub [下载高级命令侦听器示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/CachingInterception) 。

EF Core 侦听器可以：

* 告诉 EF Core 取消执行正在截获的操作
* 将报告的操作结果改回 EF Core

此示例显示了一个侦听器，该侦听器使用这些功能作为基元二级缓存的行为。 为特定查询返回缓存查询结果，避免数据库往返。

> [!WARNING]
> 以这种方式更改 EF Core 默认行为时要格外小心。 如果出现无法正确处理的异常结果，EF Core 可能会以意外的方式运行。 此外，此示例还演示侦听器概念;它不是为了提供强大的二级缓存实现的模板。

在此示例中，应用程序经常执行查询以获取最新的 "每天消息"：

<!--
        async Task<string> GetDailyMessage(DailyMessageContext context)
            => (await context.DailyMessages.TagWith("Get_Daily_Message").OrderBy(e => e.Id).LastAsync()).Message;
-->
[!code-csharp[GetDailyMessage](../../../samples/core/Miscellaneous/CachingInterception/Program.cs?name=GetDailyMessage)]

此查询已 [标记](xref:core/querying/tags) 为可在侦听器中轻松检测到它。 其思路是每天只查询一次新消息的数据库。 在其他情况，应用程序将使用缓存的结果。  (示例使用示例中的延迟10秒来模拟新的一天。 ) 

#### <a name="interceptor-state"></a>侦听器状态

此拦截程序是有状态的：它存储查询的最近一次消息的 ID 和消息文本，以及执行该查询的时间。 由于此状态，我们还需要一个 [锁](/dotnet/csharp/language-reference/keywords/lock-statement) ，因为缓存要求相同的侦听器必须由多个上下文实例使用。

<!--
    private readonly object _lock = new object();
    private int _id;
    private string _message;
    private DateTime _queriedAt;
-->
[!code-csharp[InterceptorState](../../../samples/core/Miscellaneous/CachingInterception/CachingCommandInterceptor.cs?name=InterceptorState)]

#### <a name="before-execution"></a>执行之前

在 `Executing` 方法 (即，在对数据库调用进行) 之前，侦听器会检测标记的查询，然后检查是否有缓存的结果。 如果找到这样的结果，则会取消查询并改为使用缓存的结果。

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

请注意，代码如何调用 <xref:Microsoft.EntityFrameworkCore.Diagnostics.InterceptionResult%601.SuppressWithResult%2A?displayProperty=nameWithType> 并传递 <xref:System.Data.Common.DbDataReader> 包含缓存数据的替换。 然后，将返回此 InterceptionResult，从而导致执行查询。 替换读取器将作为查询结果 EF Core 使用。

此侦听器还处理命令文本。 此操作不是必需的，但会提高日志消息的清晰度。 命令文本不一定是有效的 SQL，因为现在不会执行查询。

#### <a name="after-execution"></a>执行之后

如果没有可用的缓存消息，或者如果它已过期，则上面的代码不会显示结果。 因此 EF Core 将以正常方式执行查询。 然后，它将在执行后返回到侦听器的 `Executed` 方法。 此时，如果该结果不是已缓存的读取器，则将从真正的读取器中 exacted 新的消息 ID 和字符串，并将其缓存起来，以便下一次使用此查询。

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

#### <a name="demonstration"></a>演示

[缓存侦听器示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/CachingInterception)包含一个简单的控制台应用程序，该应用程序查询每日消息以测试缓存：

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

这会导致生成以下输出：

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

从日志输出中可以看到，应用程序继续使用缓存消息，直到超时过期，此时将再次查询数据库以获取任何新的消息。

## <a name="savechanges-interception"></a>SaveChanges 拦截

> [!NOTE]
> EF Core 5.0 中引入了 SaveChanges 拦截。

> [!TIP]  
> 可以从 GitHub [下载 SaveChanges 侦听器示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SaveChangesInterception) 。

<xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 和 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A> 侦听点由 `ISaveChangesInterceptor` <!-- Issue #2748 --> 交互. 对于其他侦听器， `SaveChangesInterceptor` <!-- Issue #2748 --> 为方便起见，提供了具有无操作方法的基类。

> [!TIP]
> 拦截功能强大。 但是，在许多情况下，重写 SaveChanges 方法或使用 DbContext 上公开的 [savechanges 的 .net 事件](xref:core/logging-events-diagnostics/events) 可能更容易。

### <a name="example-savechanges-interception-for-auditing"></a>示例：用于审核的 SaveChanges 侦听

可以截获 SaveChanges 来创建所进行更改的独立审核记录。

> [!NOTE]
> 这并不是一种可靠的审核解决方案。 这是一个简化的示例，用于演示截获的功能。

#### <a name="the-application-context"></a>应用程序上下文

[审核示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SaveChangesInterception)对博客和帖子使用简单 DbContext。

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

请注意，会为每个 DbContext 实例注册侦听器的新实例。 这是因为审核侦听器包含链接到当前上下文实例的状态。

#### <a name="the-audit-context"></a>审核上下文

该示例还包含用于审核数据库的第二个 DbContext 和模型。

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

#### <a name="the-interceptor"></a>侦听器

与侦听器审核的一般思路如下：

* 在 SaveChanges 开始时创建审核消息，并将其写入审核数据库
* 允许使用 SaveChanges 继续
* 如果 SaveChanges 成功，则更新审核消息以指示成功
* 如果 SaveChanges 失败，则更新审核消息以指示失败

在使用的替代将任何更改发送到数据库之前处理第一个阶段 `ISaveChangesInterceptor.SavingChanges` <!-- Issue #2748 -->  和 `ISaveChangesInterceptor.SavingChangesAsync`<!-- Issue #2748 -->.

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

同时替代同步和异步方法可确保无论是否调用 SaveChanges 或 SaveChangesAsync，都将发生审核。 另请注意，异步重载本身能够对审核数据库执行非阻塞异步 i/o。 你可能希望从 sync SavingChanges 方法中引发，以确保所有数据库 i/o 都是异步的。 这就要求应用程序始终调用 SaveChangesAsync 而绝不会调用。

#### <a name="the-audit-message"></a>审核消息

每个侦听器方法都有一个 `eventData` 参数，该参数提供有关被截获事件的上下文信息。 在这种情况下，将在事件数据中包含当前应用程序 DbContext，然后将该数据用于创建审核消息。

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

结果是 `SaveChangesAudit` 具有实体集合的实体 `EntityAudit` ，每个实体对应于插入、更新或删除。 然后，侦听器会将这些实体插入到审核数据库中。

> [!TIP]
> 在每个 EF Core 事件数据类中重写 ToString，以生成事件的等效日志消息。 例如，调用会 `ContextInitializedEventData.ToString` 生成 "Entity Framework Core 5.0.0，并使用提供程序" microsoft.entityframeworkcore "（选项为" 无 "）初始化" BlogsContext "。

#### <a name="detecting-success"></a>检测成功

审核实体存储在侦听器上，以便一旦 SaveChanges 成功或失败，就可以再次访问该实体。 如果成功， `ISaveChangesInterceptor.SavedChanges` <!-- Issue #2748 --> 或 `ISaveChangesInterceptor.SavedChangesAsync` <!-- Issue #2748 --> 调用 。

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

审核实体附加到审核上下文，因为它已存在于数据库中，需要更新。 然后，设置 `Succeeded` 和 `EndTime` ，这会将这些属性标记为已修改，因此 SaveChanges 会将更新发送到审核数据库。

#### <a name="detecting-failure"></a>检测失败

处理故障的方式与成功的方式几乎相同，但在 `ISaveChangesInterceptor.SaveChangesFailed` <!-- Issue #2748 --> 或 `ISaveChangesInterceptor.SaveChangesFailedAsync` <!-- Issue #2748 --> 方法。 事件数据包含引发的异常。

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

#### <a name="demonstration"></a>演示

[审核示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SaveChangesInterception)包含一个简单的控制台应用程序，该应用程序对博客数据库进行更改，然后显示已创建的审核。

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

结果显示审核数据库的内容：

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
