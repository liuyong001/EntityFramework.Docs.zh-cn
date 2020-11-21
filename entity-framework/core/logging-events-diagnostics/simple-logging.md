---
title: 简单日志记录-EF Core
description: 使用 LogTo 从 EF Core DbContext 进行日志记录
author: ajcvickers
ms.date: 10/03/2020
uid: core/logging-events-diagnostics/simple-logging
ms.openlocfilehash: 274fca39c45c4e2ccfd47f3c4eeb0834e00d18aa
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003505"
---
# <a name="simple-logging"></a>简单的日志记录

> [!NOTE]
> EF Core 5.0 中引入了此功能。

> [!TIP]  
> 可以从 GitHub [下载此文章的示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/SimpleLogging) 。

Entity Framework Core (EF Core) 简单日志记录可用于在开发和调试应用程序时轻松获取日志。 这种形式的日志记录需要最少的配置，而不需要其他 NuGet 包。

> [!TIP]
> EF Core 还与 [Microsoft. Extensions. 日志记录](xref:core/logging-events-diagnostics/extensions-logging)，这需要更多配置，但通常更适合用于在生产应用程序中进行日志记录。

## <a name="configuration"></a>Configuration

可以通过使用 [LogTo](https://github.com/dotnet/efcore/blob/ec3df8fd7e4ea4ebeebfa747619cef37b23ab2c6/src/EFCore/DbContextOptionsBuilder.cs#L135) 从任意类型的应用程序访问 EF Core 日志 <!-- Issue #2748 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.LogTo%2A> --> [配置 DbContext 实例](xref:core/dbcontext-configuration/index)时。 此配置通常通过替代 <xref:Microsoft.EntityFrameworkCore.DbContext.OnConfiguring%2A?displayProperty=nameWithType> 来完成。 例如：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(Console.WriteLine);
-->
[!code-csharp[LogToConsole](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=LogToConsole)]

或者， `LogTo` <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> 在创建 <xref:Microsoft.EntityFrameworkCore.DbContextOptions> 要传递给构造函数的实例时，可以将作为或的一部分来调用 `DbContext` 。

> [!TIP]
> 当使用 AddDbContext 或将 DbContextOptions 实例传递到 DbContext 构造函数时，仍将调用 OnConfiguring。 这使得它成为应用上下文配置的理想位置，而不考虑如何构造 DbContext。

## <a name="directing-the-logs"></a>定向日志

### <a name="logging-to-the-console"></a>日志记录到控制台

`LogTo` 需要一个 <xref:System.Action%601> 接受字符串的委托。 EF Core 将为每个生成的日志消息调用此委托和一个字符串。 然后，将由委托来处理给定的消息。

<xref:System.Console.WriteLine%2A?displayProperty=nameWithType>方法通常用于此委托，如上所示。 这会导致每个日志消息都写入控制台。

### <a name="logging-to-the-debug-window"></a>记录到调试窗口

<xref:System.Diagnostics.Debug.WriteLine%2A?displayProperty=nameWithType> 可用于将输出发送到 Visual Studio 或其他 Ide 中的 "调试" 窗口。 在这种情况下，必须使用[Lambda 语法](/dotnet/csharp/language-reference/operators/lambda-expressions)，因为 `Debug` 该类是从发布版本编译而来的。 例如：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(message => Debug.WriteLine(message));
-->
[!code-csharp[LogToDebug](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=LogToDebug)]

### <a name="logging-to-a-file"></a>记录到文件

写入文件时，需要 <xref:System.IO.StreamWriter> 为该文件创建一个或类似的。 <xref:System.IO.StreamWriter.WriteLine%2A>然后，可以使用方法，如以上其他示例中所示。 请记住，在释放上下文时释放编写器，以确保文件完全关闭。 例如：

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
> 请考虑使用 [Microsoft Extensions](/aspnet/core/fundamentals/logging) 日志记录日志记录到生产应用程序中的文件。

## <a name="getting-detailed-messages"></a>获取详细消息

### <a name="sensitive-data"></a>敏感数据

默认情况下，EF Core 不会将任何数据的值包含在异常消息中。 这是因为此类数据可能是机密的，如果未处理异常，则可能会在生产中显示这些数据。

但是，在调试时，知道数据值（特别是对于密钥）会非常有用。 可以通过调用在 EF Core 中启用此功能 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging> 。 例如：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .LogTo(Console.WriteLine)
            .EnableSensitiveDataLogging();
-->
[!code-csharp[SensitiveDataLogging](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=SensitiveDataLogging)]

### <a name="detailed-query-exceptions"></a>详细查询异常

出于性能方面的考虑，EF Core 不会在 try-catch 块中包装每次调用以读取数据库提供程序的值。 但是，这有时会导致难以诊断的异常，尤其是在数据库不允许的情况下，当模型不允许时，它们会返回 NULL。

启用 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableDetailedErrors%2A> 将导致 EF 引入这些 try-catch 块，从而提供更详细的错误。 例如：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .LogTo(Console.WriteLine)
            .EnableDetailedErrors();
-->
[!code-csharp[EnableDetailedErrors](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=EnableDetailedErrors)]

## <a name="filtering"></a>Filtering

### <a name="log-levels"></a>日志级别

每 EF Core 日志消息都分配给枚举定义的级别 <xref:Microsoft.Extensions.Logging.LogLevel> 。 默认情况下，EF Core 简单日志记录在 `Debug` 级别或更高级别包含每条消息。 `LogTo` 可向传递较高的最低级别以筛选出某些消息。 例如，传递将 `Information` 导致限制为数据库访问和某些内务处理消息的最小日志集。

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(Console.WriteLine, LogLevel.Information);
-->
[!code-csharp[InfoOnly](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=InfoOnly)]

### <a name="specific-messages"></a>特定消息

每条日志消息都分配有一个 <xref:Microsoft.Extensions.Logging.EventId> 。 这些 Id 可以从 <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId> 类或 <xref:Microsoft.EntityFrameworkCore.Diagnostics.RelationalEventId> 类用于关系特定消息。 数据库提供程序还可以在类似类中具有特定于提供程序的 Id。 例如， <xref:Microsoft.EntityFrameworkCore.Diagnostics.SqlServerEventId> 对于 SQL Server 提供程序。

`LogTo` 可以配置为仅记录与一个或多个事件 Id 相关联的消息。 例如，若要只记录要初始化或释放的上下文的消息，请执行以下操作：  

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .LogTo(Console.WriteLine, new[] { CoreEventId.ContextDisposed, CoreEventId.ContextInitialized });
-->
[!code-csharp[EventIds](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=EventIds)]

### <a name="message-categories"></a>消息类别

每条日志消息都分配到一个命名的分层记录器类别。 类别为：

| 类别                                             | 消息
|:-----------------------------------------------------|-------------------------------------------------
| Microsoft.EntityFrameworkCore                        | 所有 EF Core 消息
| Microsoft.entityframeworkcore               | 所有数据库交互
| Microsoft.entityframeworkcore。连接    | 使用数据库连接
| Microsoft.entityframeworkcore。       | 使用数据库命令
| Microsoft.entityframeworkcore 事务   | 使用数据库事务
| Microsoft.entityframeworkcore。更新                 | 保存实体，排除数据库交互
| Microsoft.entityframeworkcore                  | 所有模型和元数据交互
| Microsoft.entityframeworkcore。验证       | 模型验证
| Microsoft.entityframeworkcore                  | 查询，排除数据库交互
| Microsoft.entityframeworkcore         | 常规事件，例如上下文创建
| Microsoft.entityframeworkcore 基架            | 数据库反向工程
| Microsoft.entityframeworkcore             | 迁移
| Microsoft.entityframeworkcore. 更改跟踪         | 更改跟踪交互

`LogTo` 可以配置为仅记录来自一个或多个类别的消息。 例如，若要只记录数据库交互：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .LogTo(Console.WriteLine, new[] { DbLoggerCategory.Database.Name });
-->
[!code-csharp[DatabaseCategory](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=DatabaseCategory)]

请注意， <xref:Microsoft.EntityFrameworkCore.DbLoggerCategory> 类提供了一个分层 API 用于查找类别，并避免了对字符串进行硬编码。

由于类别具有层次结构，因此使用类别的此示例 `Database` 将包括子类别、和的所有消息 `Database.Connection` `Database.Command` `Database.Transaction` 。

### <a name="custom-filters"></a>自定义筛选器

`LogTo` 允许使用自定义筛选器，用于上述筛选选项均不足的情况。 例如，若要记录任何级别 `Information` 或更高级别的消息，以及用于打开和关闭连接的消息：

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
> 使用自定义筛选器或使用此处所示的任何其他选项进行筛选比 LogTo 委托中的筛选更有效。 这是因为如果筛选器确定不应记录消息，则不会创建日志消息。

## <a name="configuration-for-specific-messages"></a>特定消息的配置

EF Core <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.ConfigureWarnings%2A> API 允许应用程序更改遇到特定事件时所发生的情况。 这可用于：

* 更改记录事件的日志级别
* 完全跳过记录事件
* 事件发生时引发异常

### <a name="changing-the-log-level-for-an-event"></a>更改事件的日志级别

前面的示例使用自定义筛选器来记录每条消息以及为 `LogLevel.Information` 定义的两个事件 `LogLevel.Debug` 。 可以通过将两个事件的日志级别更改 `Debug` 为来实现相同的 `Information` 操作：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .ConfigureWarnings(b => b.Log(
                (RelationalEventId.ConnectionOpened, LogLevel.Information),
                (RelationalEventId.ConnectionClosed, LogLevel.Information)))
            .LogTo(Console.WriteLine, LogLevel.Information);
-->
[!code-csharp[ChangeLogLevel](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=ChangeLogLevel)]

### <a name="suppress-logging-an-event"></a>禁止记录事件

同样，可以从日志记录中抑制单个事件。 这对于忽略已评审和理解的警告特别有用。 例如：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .ConfigureWarnings(b => b.Ignore(CoreEventId.DetachedLazyLoadingWarning))
            .LogTo(Console.WriteLine);
-->
[!code-csharp[SuppressMessage](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=SuppressMessage)]

### <a name="throw-for-an-event"></a>引发事件

最后，可以将 EF Core 配置为引发给定事件。 这对于将警告更改为错误特别有用。  (确实，这是方法的原始用途 `ConfigureWarnings` ，因此是名称。 ) 例如：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder
            .ConfigureWarnings(b => b.Throw(RelationalEventId.MultipleCollectionIncludeWarning))
            .LogTo(Console.WriteLine);
-->
[!code-csharp[ThrowForEvent](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=ThrowForEvent)]

## <a name="message-contents-and-formatting"></a>消息内容和格式设置

中的默认内容 `LogTo` 是跨多行格式设置的。 第一行包含消息元数据：

* <xref:Microsoft.Extensions.Logging.LogLevel>作为四字符前缀
* 为当前区域性设置格式的本地时间戳
* <xref:Microsoft.Extensions.Logging.EventId>可以复制/粘贴以便从或其他某个类获取成员的窗体中的 <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId> `EventId` ，以及原始 ID 值
* 事件类别，如上所述。

例如：

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

可以通过从[DbContextLoggerOptions](https://github.com/dotnet/efcore/blob/ec3df8fd7e4ea4ebeebfa747619cef37b23ab2c6/src/EFCore/Diagnostics/DbContextLoggerOptions.cs#L15)传递值来自定义此内容 <!-- Issue #2748 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbContextLoggerOptions> -->，如以下部分所示。

> [!TIP]
> 请考虑使用 [Microsoft Extensions](/aspnet/core/fundamentals/logging) 日志来更好地控制日志格式。

### <a name="using-utc-time"></a>使用 UTC 时间

默认情况下，timestamnps 在调试时专用于本地消耗。 使用 `DbContextLoggerOptions.DefaultWithUtcTime` 可以改为使用与区域性无关的 UTC 时间戳，但保留所有其他内容。 例如：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(
            Console.WriteLine,
            LogLevel.Debug,
            DbContextLoggerOptions.DefaultWithUtcTime);
-->
[!code-csharp[Utc](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=Utc)]

此示例将生成以下日志格式：

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

### <a name="single-line-logging"></a>单行日志记录

有时，每条日志消息只获取一行很有用。 这可以由启用 `DbContextLoggerOptions.SingleLine` 。 例如：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(
            Console.WriteLine,
            LogLevel.Debug,
            DbContextLoggerOptions.DefaultWithLocalTime | DbContextLoggerOptions.SingleLine);
-->
[!code-csharp[SingleLine](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=SingleLine)]

此示例将生成以下日志格式：

```output
info: 10/6/2020 10:52:45.723 RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command) -> Executed DbCommand (0ms) [Parameters=[], CommandType='Text', CommandTimeout='30']CREATE TABLE "Blogs" (    "Id" INTEGER NOT NULL CONSTRAINT "PK_Blogs" PRIMARY KEY AUTOINCREMENT,    "Name" INTEGER NOT NULL);
dbug: 10/6/2020 10:52:45.723 RelationalEventId.TransactionCommitting[20210] (Microsoft.EntityFrameworkCore.Database.Transaction) -> Committing transaction.
dbug: 10/6/2020 10:52:45.725 RelationalEventId.TransactionCommitted[20202] (Microsoft.EntityFrameworkCore.Database.Transaction) -> Committed transaction.
```

### <a name="other-content-options"></a>其他内容选项

[DbContextLoggerOptions](https://github.com/dotnet/efcore/blob/ec3df8fd7e4ea4ebeebfa747619cef37b23ab2c6/src/EFCore/Diagnostics/DbContextLoggerOptions.cs#L15)中的其他标志 <!-- Issue #2748 <xref:Microsoft.EntityFrameworkCore.Diagnostics.DbContextLoggerOptions> --> 可用于修整日志中包含的元数据量。 这对于单行日志记录很有用。 例如：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(
            Console.WriteLine,
            LogLevel.Debug,
            DbContextLoggerOptions.UtcTime | DbContextLoggerOptions.SingleLine);
-->
[!code-csharp[TerseLogs](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=TerseLogs)]

此示例将生成以下日志格式：

```output
2020-10-06T17:52:45.7320362Z -> Executed DbCommand (0ms) [Parameters=[], CommandType='Text', CommandTimeout='30']CREATE TABLE "Blogs" (    "Id" INTEGER NOT NULL CONSTRAINT "PK_Blogs" PRIMARY KEY AUTOINCREMENT,    "Name" INTEGER NOT NULL);
2020-10-06T17:52:45.7320531Z -> Committing transaction.
2020-10-06T17:52:45.7339441Z -> Committed transaction.
```

## <a name="moving-from-ef6"></a>从 EF6 移动

EF Core 简单日志记录 <xref:System.Data.Entity.Database.Log?displayProperty=nameWithType> 在以下两个重要方面不同于 EF6：

* 日志消息不仅限于数据库交互
* 必须在上下文初始化时配置日志记录

对于第一个差异，可以使用上述筛选来限制要记录的消息。

第二个差异是指在不需要日志消息时不生成日志消息，从而提高性能的故意更改。 但是，仍然可以通过 `Log` 在上创建一个属性 `DbContext` ，然后仅在设置该属性时使用该属性，来使 EF6 的行为类似。 例如：

<!--
    public Action<string> Log { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(s => Log?.Invoke(s));
-->
[!code-csharp[DatabaseLog](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=DatabaseLog)]
