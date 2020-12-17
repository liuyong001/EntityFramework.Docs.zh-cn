---
title: 高级性能主题
description: Entity Framework Core 的高级性能主题
author: rick-anderson
ms.author: riande
ms.date: 12/9/2020
uid: core/performance/advanced-performance-topics
ms.openlocfilehash: 3c0340e1b36cbbb96d23db0633cb2eebc04dd970
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97657719"
---
# <a name="advanced-performance-topics"></a>高级性能主题

## <a name="dbcontext-pooling"></a>DbContext 池

`AddDbContextPool` 启用实例的池 `DbContext` 。 上下文池可以通过重复使用上下文实例，而不是为每个请求创建新实例，从而提高大规模方案（如 web 服务器）的吞吐量。

使用 EF Core 的 ASP.NET Core 应用程序中的典型模式涉及将自定义 <xref:Microsoft.EntityFrameworkCore.DbContext> 类型注册到 [依赖关系注入](/aspnet/core/fundamentals/dependency-injection) 容器，并通过控制器或 Razor Pages 中的构造函数参数获取该类型的实例。 使用构造函数注入，将为每个请求创建一个新的上下文实例。

<xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContextPool%2A> 启用可重用上下文实例的池。 若要使用上下文池，请使用 `AddDbContextPool` 方法，而不是 `AddDbContext` 在服务注册期间使用：

```csharp
services.AddDbContextPool<BloggingContext>(
    options => options.UseSqlServer(connectionString));
```

`AddDbContextPool`使用时，在请求上下文实例时，EF 首先检查池中是否有可用的实例。 请求处理完成后，实例的任何状态都将被重置，并且实例本身会返回池中。

从概念上讲，这类似于连接池在 ADO.NET 提供程序中的工作方式，并且具有保存上下文实例的某些初始化开销的优点。

的 `poolSize` 参数 <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContextPool%2A> 设置池保留的最大实例数。 一旦 `poolSize` 超出，就不会缓存新的上下文实例，EF 会回退到按需创建实例的非池行为。

### <a name="limitations"></a>限制

应该对应用进行分析和测试，以表明上下文初始化非常重要。

`AddDbContextPool` 对可在上下文的方法中完成的操作有一些限制 `OnConfiguring` 。

> [!WARNING]
> 避免在维护状态的应用程序中使用上下文池。 例如，不应在请求之间共享的上下文中的私有字段。 在将上下文实例添加到池中之前，EF Core 仅重置它知道的状态。

上下文池的工作方式是跨请求重复使用同一上下文实例。 这意味着，它可以有效地根据实例本身注册为 [单一](/aspnet/core/fundamentals/dependency-injection#service-lifetimes) 实例，以便能够持久保存。

上下文池适用于在请求之间修复了上下文配置（包括服务已解决）的情况。 对于需要 [范围内](/aspnet/core/fundamentals/dependency-injection#service-lifetimes) 的服务或需要更改配置的情况，请不要使用 pooling。 除高度优化的方案外，池的性能提升通常可以忽略不计。

## <a name="query-caching-and-parameterization"></a>查询缓存和参数化

当 EF 收到用于执行的 LINQ 查询树时，它必须首先将该树 "编译" 到 SQL 查询中。 由于这是一个很大的过程，EF 会按查询树 *形状* 缓存查询：具有相同结构的查询重用内部缓存的编译输出，并可以跳过重复编译。 不同的查询可能仍引用不同的 *值*，但只要这些值参数正确，结构就是相同的，并且缓存将正常运行。

请考虑以下两个查询：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#QueriesWithConstants)]

由于表达式树包含不同的常量，因此表达式树不同，每个查询将由 EF Core 分别进行编译。 此外，每个查询还会生成一个略有不同的 SQL 命令：

```sql
SELECT TOP(1) [b].[Id], [b].[Name]
FROM [Blogs] AS [b]
WHERE [b].[Name] = N'blog1'

SELECT TOP(1) [b].[Id], [b].[Name]
FROM [Blogs] AS [b]
WHERE [b].[Name] = N'blog2'
```

由于 SQL 不同，数据库服务器可能还需要为这两个查询生成查询计划，而不是重复使用同一个计划。

对查询进行的小修改可能会显著变化：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#QueriesWithParameterization)]

由于博客名称现已 *参数化*，因此两个查询具有相同的树形状，而 EF 只需编译一次。 生成的 SQL 也是参数化的，使数据库能够重复使用相同的查询计划：

```sql
SELECT TOP(1) [b].[Id], [b].[Name]
FROM [Blogs] AS [b]
WHERE [b].[Name] = @__blogName_0
```

请注意，无需对每个查询进行参数化：对常量进行一些查询是非常好的方法，事实上，数据库 (和 EF) 有时可能会对查询进行参数化时的常量执行某些优化。 请参阅有关 [动态构造的查询](#dynamically-constructed-queries) 的部分，了解正确参数化至关重要的示例。

> [!NOTE]
> EF Core 的 [事件计数器](xref:core/logging-events-diagnostics/event-counters) 报告查询缓存命中率。 在正常的应用程序中，在程序启动后，此计数器不久就会达到100%，一旦大多数查询执行至少一次。 如果此计数器的值低于100%，则表示应用程序可能正在执行与查询缓存不一致的操作，这是一个很好的想法。

> [!NOTE]
> 数据库如何管理缓存查询计划依赖于数据库。 例如，SQL Server 隐式维护 LRU 查询计划缓存，而 PostgreSQL 不 (但已准备好的语句可能会产生非常类似的最终效果) 。 有关更多详细信息，请参阅数据库文档。

## <a name="dynamically-constructed-queries"></a>动态构造的查询

在某些情况下，需要动态构造 LINQ 查询，而不是在源代码中进行完全指定。 例如，在从客户端接收任意查询详细信息的网站中，具有开放查询运算符 (排序、筛选、分页 ... ) ，就会发生这种情况。原则上，如有必要，动态构造的查询可以像常规的查询一样高效 (尽管不能将已编译的查询优化用于动态查询) 。 但实际上，它们常常是性能问题的根源，因为这样可以很容易地生成具有每次不同的形状的表达式树。

下面的示例使用两种方法来动态构造查询; `Where` 仅当给定参数不为 null 时，才向查询添加运算符。 请注意，这并不适合用于动态构造查询-但为了简单起见，我们将使用它：

### <a name="with-constant"></a>[带有常量](#tab/with-constant)

[!code-csharp[Main](../../../samples/core/Benchmarks/DynamicallyConstructedQueries.cs?name=WithConstant&highlight=14-24)]

### <a name="with-parameter"></a>[With 参数](#tab/with-parameter)

[!code-csharp[Main](../../../samples/core/Benchmarks/DynamicallyConstructedQueries.cs?name=WithParameter&highlight=14)]

***

这两种方法的基准测试可提供以下结果：

|        方法 |       平均值 |    错误 |    标准偏差 |   第0代 |  第1代 | 第2代 | 已分配 |
|-------------- |-----------:|---------:|----------:|--------:|-------:|------:|----------:|
|  WithConstant | 1096.7 美国 | 12.54 美国 |  11.12 美国 | 13.6719 | 1.9531 |     - |  83.91 KB |
| WithParameter |   570.8 美国 | 42.43 美国 | 124.43 美国 |  5.8594 |      - |     - |  37.16 KB |

即使子毫秒差异看起来很小，也请记住，恒定版本会持续 pollutes 缓存，并导致重新编译其他查询，同时降低其速度。

> [!NOTE]
> 除非确实需要，否则请避免构造带有表达式树 API 的查询。 除了 API 的复杂性，在使用它们时非常容易导致严重的性能问题。
