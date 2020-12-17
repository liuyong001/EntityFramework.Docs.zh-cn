---
title: 性能诊断-EF Core
description: 诊断 Entity Framework Core 性能并识别瓶颈
author: roji
ms.date: 12/1/2020
uid: core/performance/performance-diagnosis
ms.openlocfilehash: 9416acf3326056ef7a5d732c4bd456dac751167b
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97657707"
---
# <a name="performance-diagnosis"></a>性能诊断

本部分讨论检测 EF 应用程序中的性能问题的方法，并在确定有问题的区域后，如何进一步分析它们以确定根本问题。 在跳转到任何结论之前，请务必仔细诊断并调查任何问题，以避免假设问题的根源。

## <a name="identifying-slow-database-commands-via-logging"></a>通过日志记录标识慢速数据库命令

在一天结束时，EF 将准备并执行要对数据库执行的命令;对于关系数据库，这意味着通过 ADO.NET 数据库 API 执行 SQL 语句。 如果某个查询所花的时间太长 (例如，因为缺少索引) ，则可以通过检查命令执行日志并观察它们实际执行的时间来发现。

EF 使你可以通过 [简单日志记录](xref:core/logging-events-diagnostics/simple-logging) 或 Microsoft 来轻松地捕获命令执行时间 [。日志记录](xref:core/logging-events-diagnostics/extensions-logging)：

### <a name="simple-logging"></a>[简单的日志记录](#tab/simple-logging)

[!code-csharp[Main](../../../samples/core/Performance/BloggingContext.cs#SimpleLogging)]

### <a name="microsoftextensionslogging"></a>[Microsoft.Extensions.Logging](#tab/microsoft-extensions-logging)

[!code-csharp[Main](../../../samples/core/Performance/ExtensionsLoggingContext.cs#ExtensionsLogging)]

***

如果日志记录级别设置为 `LogLevel.Information` ，则 EF 会使用所用时间为每个命令执行发出一条日志消息：

```log
info: 06/12/2020 09:12:36.117 RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command)
      Executed DbCommand (4ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      SELECT [b].[Id], [b].[Name]
      FROM [Blogs] AS [b]
      WHERE [b].[Name] = N'foo'
```

上述命令花费了4毫秒。 如果某个命令的执行时间超过预期，则可能是由于性能问题而发现可能的问题，现在可以集中精力来了解它运行缓慢的原因。 命令日志记录还可以揭示发生意外的数据库往返的情况;这会显示为多个只需要一个命令的命令。

> [!WARNING]
> 在生产环境中启用命令执行日志记录通常是一种不好的做法。 日志记录本身会降低应用程序的速度，并可快速创建大量日志文件，从而填满服务器的磁盘空间。 建议只将日志记录时间缩短为收集数据的时间间隔，同时仔细监视应用程序或捕获预生产系统上的日志记录数据。

## <a name="correlating-database-commands-to-linq-queries"></a>将数据库命令关联到 LINQ 查询

命令执行日志记录的一个问题是，有时很难关联 SQL 查询和 LINQ 查询： EF 执行的 SQL 命令可能与生成它们的 LINQ 查询看起来非常不同。 为了帮助解决此问题，你可能需要使用 EF 的 " [查询标记](xref:core/querying/tags) " 功能，该功能允许你将一个小型的标识注释注入到 SQL 查询中：

[!code-csharp[Main](../../../samples/core/Querying/Tags/Program.cs#BasicQueryTag)]

此标记显示在日志中：

```sql
-- This is my spatial query!

SELECT TOP(@__p_1) [p].[Id], [p].[Location]
FROM [People] AS [p]
ORDER BY [p].[Location].STDistance(@__myLocation_0) DESC
```

通常，使用这种方法标记应用程序的主要查询会使命令执行日志的可读性更高。

## <a name="other-interfaces-for-capturing-performance-data"></a>用于捕获性能数据的其他接口

EF 的日志记录功能有多种替代方法，用于捕获命令执行时间，这可能更加强大。 数据库通常附带自己的跟踪和性能分析工具，这些工具通常提供比简单的执行时间更丰富的数据库特定的信息;每个数据库的实际设置、功能和使用情况各不相同。

例如， [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) 是一种功能强大的客户端，可连接到 SQL Server 实例，并提供有价值的管理和性能信息。 此部分超出了详细信息的范围，但有两个值得提的功能是 [活动监视器](/sql/relational-databases/performance-monitor/open-activity-monitor-sql-server-management-studio)，它提供了服务器活动的实时仪表板， (包括最昂贵的查询) 和 [扩展事件 (XEvent) ](/sql/relational-databases/extended-events/quick-start-extended-events-in-sql-server) 功能，这允许定义可根据你的确切需求进行定制的任意数据捕获会话。 [有关监视的 SQL Server 文档](/sql/relational-databases/performance/monitor-and-tune-for-performance) 提供了有关这些功能以及其他功能的详细信息。

捕获性能数据的另一种方法是，通过接口收集由 EF 或数据库驱动程序自动发出的信息 `DiagnosticSource` ，然后分析该数据或将其显示在仪表板上。 如果你使用的是 Azure，则 [Azure 应用程序 Insights](https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-performance) 提供现成的强大监视功能，从而在分析你的 web 请求的速度时，集成数据库性能和查询执行时间。 有关此功能的详细信息，请访问 [Application Insights 性能教程](/azure/azure-monitor/learn/tutorial-performance)和 [Azure SQL analytics 页面](/azure/azure-monitor/insights/azure-sql)。

## <a name="inspecting-query-execution-plans"></a>检查查询执行计划

在您查明了需要优化的问题查询后，下一步通常是分析查询的 *执行计划*。 当数据库收到 SQL 语句时，它们通常会生成计划的执行方式，这有时需要根据已定义的索引、表中存在的数据量等来做出复杂的决策， (顺便说一下，计划本身应缓存在服务器上，以获得最佳性能) 。 关系数据库通常为用户提供了一种方式来查看查询计划，以及查询的不同部分的计算成本;这对于改进查询非常有用。

若要开始 SQL Server，请参阅有关 [查询执行计划](/sql/relational-databases/performance/execution-plans)的文档。 典型的分析工作流将使用 [SQL Server Management Studio](/sql/relational-databases/performance/display-an-actual-execution-plan)，粘贴通过上述方法之一标识的慢速查询的 SQL，并 [生成图形执行计划](/sql/relational-databases/performance/display-an-actual-execution-plan)：

![显示 SQL Server 执行计划](_static/actualexecplan.png)

尽管执行计划在最初看起来很复杂，但一定要花费很长时间来熟悉它们。 尤其重要的是，请注意与计划的每个节点关联的成本，并确定如何在不同节点 (或不) 使用索引。

虽然以上信息特定于 SQL Server，但其他数据库通常提供与可视化效果相同的工具。

> [!IMPORTANT]
> 根据数据库中的实际数据，数据库有时会生成不同的查询计划。 例如，如果表只包含几行，则数据库可以选择不使用该表的索引，而是改为执行全表扫描。 如果在测试数据库中分析查询计划，请始终确保它包含类似于生产系统的数据。

## <a name="event-counters"></a>事件计数器

以上章节重点介绍如何获取有关命令的信息，以及如何在数据库中执行这些命令。 除此之外，EF 还公开了一组 *事件计数器* ，这些计数器提供有关在 EF 本身内发生的情况以及应用程序使用它的方式的更低级别信息。 这些计数器对于诊断特定性能问题和性能异常非常有用，例如导致进行持续重新编译的 [查询缓存问题](xref:core/performance/advanced-performance-topics#dynamically-constructed-queries) 、未释放 DbContext 和其他计数器。

有关详细信息，请参阅 [EF 的事件计数器](xref:core/logging-events-diagnostics/event-counters) 上的专用页面。

## <a name="benchmarking-with-ef-core"></a>EF Core 的基准测试

在一天结束时，有时需要知道编写或执行查询的特定方式是否比另一种方法更快。 一定要永远不要假设或推测答案，同时将快速基准组合到一起以获得答案非常简单。 编写基准时，强烈建议使用众所周知的 [BenchmarkDotNet](https://benchmarkdotnet.org/index.html) 库，此库处理用户在尝试编写自己的基准时遇到的许多缺陷：是否已执行一些预热迭代？ 基准测试实际运行了多少次？ 让我们看看 EF Core 的基准的外观。

> [!TIP]
> [下面提供了](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Benchmarks/AverageBlogRanking.cs)下面的源的完整基准项目。 建议你复制它，并将其用作模板来实现你自己的基准测试。

作为一个简单的基准方案，让我们比较一下计算数据库中所有博客的平均排名的不同方法：

* 加载所有实体，对其各个排名求和，并计算平均值。
* 与上面相同，只使用非跟踪查询。 这应该更快，因为不执行标识解析，并且由于更改跟踪目的而不会对实体进行快照。
* 通过仅投影排名，避免加载整个博客实体实例。 这使我们无法转移其他不需要的博客实体类型列。
* 通过使其成为查询的一部分来计算数据库中的平均值。 这应该是最快的方法，因为所有内容都是在数据库中计算的，并且只会将结果传输回客户端。

使用 BenchmarkDotNet，可以编写要作为简单方法（就像单元测试一样）进行基准处理的代码，并且 BenchmarkDotNet 会自动运行每个方法来获得足够的迭代数，并可靠地度量所需的时间和分配的内存量。 下面是 ([完整基准代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Benchmarks/AverageBlogRanking.cs) 的不同方法) ：

### <a name="load-entities"></a>[加载实体](#tab/load-entities)

[!code-csharp[Main](../../../samples/core/Benchmarks/AverageBlogRanking.cs?name=LoadEntities)]

### <a name="load-entities-no-tracking"></a>[加载实体，无跟踪](#tab/load-entities-no-tracking)

[!code-csharp[Main](../../../samples/core/Benchmarks/AverageBlogRanking.cs?name=LoadEntitiesNoTracking)]

### <a name="project-only-ranking"></a>[仅限项目排名](#tab/project-only-ranking)

[!code-csharp[Main](../../../samples/core/Benchmarks/AverageBlogRanking.cs?name=ProjectOnlyRanking)]

### <a name="calculate-in-database"></a>[在数据库中计算](#tab/calculate-in-database)

[!code-csharp[Main](../../../samples/core/Benchmarks/AverageBlogRanking.cs?name=CalculateInDatabase)]

***

结果如下所示：

|                 方法 |       平均值 |    错误 |   标准偏差 |     中值 | 比率 | RatioSD |    第0代 |   第1代 | 第2代 |  已分配 |
|----------------------- |-----------:|---------:|---------:|-----------:|------:|--------:|---------:|--------:|------:|-----------:|
|           LoadEntities | 2860.4 美国 | 54.31 美国 | 93.68 美国 | 2844.5 美国 |  4.55 |    0.33 | 210.9375 | 70.3125 |     - | 1309.56 KB |
| LoadEntitiesNoTracking | 1353.0 美国 | 21.26 美国 | 18.85 美国 | 1355.6 美国 |  2.10 |    0.14 |  87.8906 |  3.9063 |     - |  540.09 KB |
|     ProjectOnlyRanking |   910.9 美国 | 20.91 美国 | 61.65 美国 |   892.9 美国 |  1.46 |    0.14 |  41.0156 |  0.9766 |     - |  252.08 KB |
|    CalculateInDatabase |   627.1 美国 | 14.58 美国 | 42.54 美国 |   626.4 美国 |  1.00 |    0.00 |   4.8828 |       - |     - |   33.27 KB |

> [!NOTE]
> 当方法在方法中实例化和释放上下文时，将对这些操作进行计算，而不是查询过程的一部分。 如果目标是将两种替代方法与另一个 (进行比较，则这一点不重要，因为上下文实例化和处置都是相同的) ，并为整个操作提供更全面的度量。

BenchmarkDotNet 的一个限制是，它测量你所提供的方法的简单的单线程性能，因此不太适合用于基准并发方案。

> [!IMPORTANT]
> 请始终确保在进行基准测试时数据库中的数据与生产数据相似，否则基准测试结果可能不表示生产中的实际性能。
