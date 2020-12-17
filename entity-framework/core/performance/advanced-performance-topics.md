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
# <a name="advanced-performance-topics"></a><span data-ttu-id="f3802-103">高级性能主题</span><span class="sxs-lookup"><span data-stu-id="f3802-103">Advanced Performance Topics</span></span>

## <a name="dbcontext-pooling"></a><span data-ttu-id="f3802-104">DbContext 池</span><span class="sxs-lookup"><span data-stu-id="f3802-104">DbContext pooling</span></span>

<span data-ttu-id="f3802-105">`AddDbContextPool` 启用实例的池 `DbContext` 。</span><span class="sxs-lookup"><span data-stu-id="f3802-105">`AddDbContextPool` enables pooling of `DbContext` instances.</span></span> <span data-ttu-id="f3802-106">上下文池可以通过重复使用上下文实例，而不是为每个请求创建新实例，从而提高大规模方案（如 web 服务器）的吞吐量。</span><span class="sxs-lookup"><span data-stu-id="f3802-106">Context pooling can increase throughput in high-scale scenarios such as web servers by reusing context instances, rather than creating new instances for each request.</span></span>

<span data-ttu-id="f3802-107">使用 EF Core 的 ASP.NET Core 应用程序中的典型模式涉及将自定义 <xref:Microsoft.EntityFrameworkCore.DbContext> 类型注册到 [依赖关系注入](/aspnet/core/fundamentals/dependency-injection) 容器，并通过控制器或 Razor Pages 中的构造函数参数获取该类型的实例。</span><span class="sxs-lookup"><span data-stu-id="f3802-107">The typical pattern in an ASP.NET Core app using EF Core involves registering a custom <xref:Microsoft.EntityFrameworkCore.DbContext> type into the [dependency injection](/aspnet/core/fundamentals/dependency-injection) container and obtaining instances of that type through constructor parameters in controllers or Razor Pages.</span></span> <span data-ttu-id="f3802-108">使用构造函数注入，将为每个请求创建一个新的上下文实例。</span><span class="sxs-lookup"><span data-stu-id="f3802-108">Using constructor injection, a new context instance is created for each request.</span></span>

<span data-ttu-id="f3802-109"><xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContextPool%2A> 启用可重用上下文实例的池。</span><span class="sxs-lookup"><span data-stu-id="f3802-109"><xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContextPool%2A> enables a pool of reusable context instances.</span></span> <span data-ttu-id="f3802-110">若要使用上下文池，请使用 `AddDbContextPool` 方法，而不是 `AddDbContext` 在服务注册期间使用：</span><span class="sxs-lookup"><span data-stu-id="f3802-110">To use context pooling, use the `AddDbContextPool` method instead of `AddDbContext` during service registration:</span></span>

```csharp
services.AddDbContextPool<BloggingContext>(
    options => options.UseSqlServer(connectionString));
```

<span data-ttu-id="f3802-111">`AddDbContextPool`使用时，在请求上下文实例时，EF 首先检查池中是否有可用的实例。</span><span class="sxs-lookup"><span data-stu-id="f3802-111">When `AddDbContextPool` is used, at the time a context instance is requested, EF first checks if there is an instance available in the pool.</span></span> <span data-ttu-id="f3802-112">请求处理完成后，实例的任何状态都将被重置，并且实例本身会返回池中。</span><span class="sxs-lookup"><span data-stu-id="f3802-112">Once the request processing finalizes, any state on the instance is reset and the instance is itself returned to the pool.</span></span>

<span data-ttu-id="f3802-113">从概念上讲，这类似于连接池在 ADO.NET 提供程序中的工作方式，并且具有保存上下文实例的某些初始化开销的优点。</span><span class="sxs-lookup"><span data-stu-id="f3802-113">This is conceptually similar to how connection pooling operates in ADO.NET providers and has the advantage of saving some of the cost of initialization of the context instance.</span></span>

<span data-ttu-id="f3802-114">的 `poolSize` 参数 <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContextPool%2A> 设置池保留的最大实例数。</span><span class="sxs-lookup"><span data-stu-id="f3802-114">The `poolSize` parameter of <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContextPool%2A> sets the maximum number of instances retained by the pool.</span></span> <span data-ttu-id="f3802-115">一旦 `poolSize` 超出，就不会缓存新的上下文实例，EF 会回退到按需创建实例的非池行为。</span><span class="sxs-lookup"><span data-stu-id="f3802-115">Once `poolSize` is exceeded, new context instances are not cached and  EF falls back to the non-pooling behavior of creating instances on demand.</span></span>

### <a name="limitations"></a><span data-ttu-id="f3802-116">限制</span><span class="sxs-lookup"><span data-stu-id="f3802-116">Limitations</span></span>

<span data-ttu-id="f3802-117">应该对应用进行分析和测试，以表明上下文初始化非常重要。</span><span class="sxs-lookup"><span data-stu-id="f3802-117">Apps should be profiled and tested to show that context initialization is a significant cost.</span></span>

<span data-ttu-id="f3802-118">`AddDbContextPool` 对可在上下文的方法中完成的操作有一些限制 `OnConfiguring` 。</span><span class="sxs-lookup"><span data-stu-id="f3802-118">`AddDbContextPool` has a few limitations on what can be done in the `OnConfiguring` method of the context.</span></span>

> [!WARNING]
> <span data-ttu-id="f3802-119">避免在维护状态的应用程序中使用上下文池。</span><span class="sxs-lookup"><span data-stu-id="f3802-119">Avoid using context pooling in apps that maintain state.</span></span> <span data-ttu-id="f3802-120">例如，不应在请求之间共享的上下文中的私有字段。</span><span class="sxs-lookup"><span data-stu-id="f3802-120">For example, private fields in the context that shouldn't be shared across requests.</span></span> <span data-ttu-id="f3802-121">在将上下文实例添加到池中之前，EF Core 仅重置它知道的状态。</span><span class="sxs-lookup"><span data-stu-id="f3802-121">EF Core only resets the state that it is aware of before adding a context instance to the pool.</span></span>

<span data-ttu-id="f3802-122">上下文池的工作方式是跨请求重复使用同一上下文实例。</span><span class="sxs-lookup"><span data-stu-id="f3802-122">Context pooling works by reusing the same context instance across requests.</span></span> <span data-ttu-id="f3802-123">这意味着，它可以有效地根据实例本身注册为 [单一](/aspnet/core/fundamentals/dependency-injection#service-lifetimes) 实例，以便能够持久保存。</span><span class="sxs-lookup"><span data-stu-id="f3802-123">This means that it's effectively registered as a [Singleton](/aspnet/core/fundamentals/dependency-injection#service-lifetimes) in terms of the instance itself so that it's able to persist.</span></span>

<span data-ttu-id="f3802-124">上下文池适用于在请求之间修复了上下文配置（包括服务已解决）的情况。</span><span class="sxs-lookup"><span data-stu-id="f3802-124">Context pooling is intended for scenarios where the context configuration, which includes services resolved, is fixed between requests.</span></span> <span data-ttu-id="f3802-125">对于需要 [范围内](/aspnet/core/fundamentals/dependency-injection#service-lifetimes) 的服务或需要更改配置的情况，请不要使用 pooling。</span><span class="sxs-lookup"><span data-stu-id="f3802-125">For cases where [Scoped](/aspnet/core/fundamentals/dependency-injection#service-lifetimes) services are required, or configuration needs to be changed, don't use pooling.</span></span> <span data-ttu-id="f3802-126">除高度优化的方案外，池的性能提升通常可以忽略不计。</span><span class="sxs-lookup"><span data-stu-id="f3802-126">The performance gain from pooling is usually negligible except in highly optimized scenarios.</span></span>

## <a name="query-caching-and-parameterization"></a><span data-ttu-id="f3802-127">查询缓存和参数化</span><span class="sxs-lookup"><span data-stu-id="f3802-127">Query caching and parameterization</span></span>

<span data-ttu-id="f3802-128">当 EF 收到用于执行的 LINQ 查询树时，它必须首先将该树 "编译" 到 SQL 查询中。</span><span class="sxs-lookup"><span data-stu-id="f3802-128">When EF receives a LINQ query tree for execution, it must first "compile" that tree into a SQL query.</span></span> <span data-ttu-id="f3802-129">由于这是一个很大的过程，EF 会按查询树 *形状* 缓存查询：具有相同结构的查询重用内部缓存的编译输出，并可以跳过重复编译。</span><span class="sxs-lookup"><span data-stu-id="f3802-129">Because this is a heavy process, EF caches queries by the query tree *shape*: queries with the same structure reuse internally-cached compilation outputs, and can skip repeated compilation.</span></span> <span data-ttu-id="f3802-130">不同的查询可能仍引用不同的 *值*，但只要这些值参数正确，结构就是相同的，并且缓存将正常运行。</span><span class="sxs-lookup"><span data-stu-id="f3802-130">The different queries may still reference different *values*, but as long as these values are properly parameterized, the structure is the same and caching will function properly.</span></span>

<span data-ttu-id="f3802-131">请考虑以下两个查询：</span><span class="sxs-lookup"><span data-stu-id="f3802-131">Consider the following two queries:</span></span>

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#QueriesWithConstants)]

<span data-ttu-id="f3802-132">由于表达式树包含不同的常量，因此表达式树不同，每个查询将由 EF Core 分别进行编译。</span><span class="sxs-lookup"><span data-stu-id="f3802-132">Since the expression trees contains different constants, the expression tree differs and each of these queries will be compiled separately by EF Core.</span></span> <span data-ttu-id="f3802-133">此外，每个查询还会生成一个略有不同的 SQL 命令：</span><span class="sxs-lookup"><span data-stu-id="f3802-133">In addition, each query produces a slightly different SQL command:</span></span>

```sql
SELECT TOP(1) [b].[Id], [b].[Name]
FROM [Blogs] AS [b]
WHERE [b].[Name] = N'blog1'

SELECT TOP(1) [b].[Id], [b].[Name]
FROM [Blogs] AS [b]
WHERE [b].[Name] = N'blog2'
```

<span data-ttu-id="f3802-134">由于 SQL 不同，数据库服务器可能还需要为这两个查询生成查询计划，而不是重复使用同一个计划。</span><span class="sxs-lookup"><span data-stu-id="f3802-134">Because the SQL differs, your database server will likely also need to produce a query plan for both queries, rather than reusing the same plan.</span></span>

<span data-ttu-id="f3802-135">对查询进行的小修改可能会显著变化：</span><span class="sxs-lookup"><span data-stu-id="f3802-135">A small modification to your queries can change things considerably:</span></span>

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#QueriesWithParameterization)]

<span data-ttu-id="f3802-136">由于博客名称现已 *参数化*，因此两个查询具有相同的树形状，而 EF 只需编译一次。</span><span class="sxs-lookup"><span data-stu-id="f3802-136">Since the blog name is now *parameterized*, both queries have the same tree shape, and EF only needs to be compiled once.</span></span> <span data-ttu-id="f3802-137">生成的 SQL 也是参数化的，使数据库能够重复使用相同的查询计划：</span><span class="sxs-lookup"><span data-stu-id="f3802-137">The SQL produced is also parameterized, allowing the database to reuse the same query plan:</span></span>

```sql
SELECT TOP(1) [b].[Id], [b].[Name]
FROM [Blogs] AS [b]
WHERE [b].[Name] = @__blogName_0
```

<span data-ttu-id="f3802-138">请注意，无需对每个查询进行参数化：对常量进行一些查询是非常好的方法，事实上，数据库 (和 EF) 有时可能会对查询进行参数化时的常量执行某些优化。</span><span class="sxs-lookup"><span data-stu-id="f3802-138">Note that there is no need to parameterize each and every query: it's perfectly fine to have some queries with constants, and indeed, databases (and EF) can sometimes perform certain optimization around constants which aren't possible when the query is parameterized.</span></span> <span data-ttu-id="f3802-139">请参阅有关 [动态构造的查询](#dynamically-constructed-queries) 的部分，了解正确参数化至关重要的示例。</span><span class="sxs-lookup"><span data-stu-id="f3802-139">See the section on [dynamically-constructed queries](#dynamically-constructed-queries) for an example where proper parameterization is crucial.</span></span>

> [!NOTE]
> <span data-ttu-id="f3802-140">EF Core 的 [事件计数器](xref:core/logging-events-diagnostics/event-counters) 报告查询缓存命中率。</span><span class="sxs-lookup"><span data-stu-id="f3802-140">EF Core's [event counters](xref:core/logging-events-diagnostics/event-counters) report the Query Cache Hit Rate.</span></span> <span data-ttu-id="f3802-141">在正常的应用程序中，在程序启动后，此计数器不久就会达到100%，一旦大多数查询执行至少一次。</span><span class="sxs-lookup"><span data-stu-id="f3802-141">In a normal application, this counter reaches 100% soon after program startup, once most queries have executed at least once.</span></span> <span data-ttu-id="f3802-142">如果此计数器的值低于100%，则表示应用程序可能正在执行与查询缓存不一致的操作，这是一个很好的想法。</span><span class="sxs-lookup"><span data-stu-id="f3802-142">If this counter remains stable below 100%, that is an indication that your application may be doing something which defeats the query cache - it's a good idea to investigate that.</span></span>

> [!NOTE]
> <span data-ttu-id="f3802-143">数据库如何管理缓存查询计划依赖于数据库。</span><span class="sxs-lookup"><span data-stu-id="f3802-143">How the database manages caches query plans is database-dependent.</span></span> <span data-ttu-id="f3802-144">例如，SQL Server 隐式维护 LRU 查询计划缓存，而 PostgreSQL 不 (但已准备好的语句可能会产生非常类似的最终效果) 。</span><span class="sxs-lookup"><span data-stu-id="f3802-144">For example, SQL Server implicitly maintains an LRU query plan cache, whereas PostgreSQL does not (but prepared statements can produce a very similar end effect).</span></span> <span data-ttu-id="f3802-145">有关更多详细信息，请参阅数据库文档。</span><span class="sxs-lookup"><span data-stu-id="f3802-145">Consult your database documentation for more details.</span></span>

## <a name="dynamically-constructed-queries"></a><span data-ttu-id="f3802-146">动态构造的查询</span><span class="sxs-lookup"><span data-stu-id="f3802-146">Dynamically-constructed queries</span></span>

<span data-ttu-id="f3802-147">在某些情况下，需要动态构造 LINQ 查询，而不是在源代码中进行完全指定。</span><span class="sxs-lookup"><span data-stu-id="f3802-147">In some situations, it is necessary to dynamically construct LINQ queries rather than specifying them outright in source code.</span></span> <span data-ttu-id="f3802-148">例如，在从客户端接收任意查询详细信息的网站中，具有开放查询运算符 (排序、筛选、分页 ... ) ，就会发生这种情况。原则上，如有必要，动态构造的查询可以像常规的查询一样高效 (尽管不能将已编译的查询优化用于动态查询) 。</span><span class="sxs-lookup"><span data-stu-id="f3802-148">This can happen, for example, in a website which receives arbitrary query details from a client, with open-ended query operators (sorting, filtering, paging...). In principle, if done correctly, dynamically-constructed queries can be just as efficient as regular ones (although it's not possible to use the compiled query optimization with dynamic queries).</span></span> <span data-ttu-id="f3802-149">但实际上，它们常常是性能问题的根源，因为这样可以很容易地生成具有每次不同的形状的表达式树。</span><span class="sxs-lookup"><span data-stu-id="f3802-149">In practice, however, they are frequently the source of performance issues, since it's easy to accidentally produce expression trees with shapes that differ every time.</span></span>

<span data-ttu-id="f3802-150">下面的示例使用两种方法来动态构造查询; `Where` 仅当给定参数不为 null 时，才向查询添加运算符。</span><span class="sxs-lookup"><span data-stu-id="f3802-150">The following example uses two techniques to dynamically construct a query; we add a `Where` operator to the query only if the given parameter is not null.</span></span> <span data-ttu-id="f3802-151">请注意，这并不适合用于动态构造查询-但为了简单起见，我们将使用它：</span><span class="sxs-lookup"><span data-stu-id="f3802-151">Note that this isn't a good use case for dynamically constructing a query - but we're using it for simplicity:</span></span>

### <a name="with-constant"></a>[<span data-ttu-id="f3802-152">带有常量</span><span class="sxs-lookup"><span data-stu-id="f3802-152">With constant</span></span>](#tab/with-constant)

[!code-csharp[Main](../../../samples/core/Benchmarks/DynamicallyConstructedQueries.cs?name=WithConstant&highlight=14-24)]

### <a name="with-parameter"></a>[<span data-ttu-id="f3802-153">With 参数</span><span class="sxs-lookup"><span data-stu-id="f3802-153">With parameter</span></span>](#tab/with-parameter)

[!code-csharp[Main](../../../samples/core/Benchmarks/DynamicallyConstructedQueries.cs?name=WithParameter&highlight=14)]

***

<span data-ttu-id="f3802-154">这两种方法的基准测试可提供以下结果：</span><span class="sxs-lookup"><span data-stu-id="f3802-154">Benchmarking these two techniques gives the following results:</span></span>

|        <span data-ttu-id="f3802-155">方法</span><span class="sxs-lookup"><span data-stu-id="f3802-155">Method</span></span> |       <span data-ttu-id="f3802-156">平均值</span><span class="sxs-lookup"><span data-stu-id="f3802-156">Mean</span></span> |    <span data-ttu-id="f3802-157">错误</span><span class="sxs-lookup"><span data-stu-id="f3802-157">Error</span></span> |    <span data-ttu-id="f3802-158">标准偏差</span><span class="sxs-lookup"><span data-stu-id="f3802-158">StdDev</span></span> |   <span data-ttu-id="f3802-159">第0代</span><span class="sxs-lookup"><span data-stu-id="f3802-159">Gen 0</span></span> |  <span data-ttu-id="f3802-160">第1代</span><span class="sxs-lookup"><span data-stu-id="f3802-160">Gen 1</span></span> | <span data-ttu-id="f3802-161">第2代</span><span class="sxs-lookup"><span data-stu-id="f3802-161">Gen 2</span></span> | <span data-ttu-id="f3802-162">已分配</span><span class="sxs-lookup"><span data-stu-id="f3802-162">Allocated</span></span> |
|-------------- |-----------:|---------:|----------:|--------:|-------:|------:|----------:|
|  <span data-ttu-id="f3802-163">WithConstant</span><span class="sxs-lookup"><span data-stu-id="f3802-163">WithConstant</span></span> | <span data-ttu-id="f3802-164">1096.7 美国</span><span class="sxs-lookup"><span data-stu-id="f3802-164">1,096.7 us</span></span> | <span data-ttu-id="f3802-165">12.54 美国</span><span class="sxs-lookup"><span data-stu-id="f3802-165">12.54 us</span></span> |  <span data-ttu-id="f3802-166">11.12 美国</span><span class="sxs-lookup"><span data-stu-id="f3802-166">11.12 us</span></span> | <span data-ttu-id="f3802-167">13.6719</span><span class="sxs-lookup"><span data-stu-id="f3802-167">13.6719</span></span> | <span data-ttu-id="f3802-168">1.9531</span><span class="sxs-lookup"><span data-stu-id="f3802-168">1.9531</span></span> |     - |  <span data-ttu-id="f3802-169">83.91 KB</span><span class="sxs-lookup"><span data-stu-id="f3802-169">83.91 KB</span></span> |
| <span data-ttu-id="f3802-170">WithParameter</span><span class="sxs-lookup"><span data-stu-id="f3802-170">WithParameter</span></span> |   <span data-ttu-id="f3802-171">570.8 美国</span><span class="sxs-lookup"><span data-stu-id="f3802-171">570.8 us</span></span> | <span data-ttu-id="f3802-172">42.43 美国</span><span class="sxs-lookup"><span data-stu-id="f3802-172">42.43 us</span></span> | <span data-ttu-id="f3802-173">124.43 美国</span><span class="sxs-lookup"><span data-stu-id="f3802-173">124.43 us</span></span> |  <span data-ttu-id="f3802-174">5.8594</span><span class="sxs-lookup"><span data-stu-id="f3802-174">5.8594</span></span> |      - |     - |  <span data-ttu-id="f3802-175">37.16 KB</span><span class="sxs-lookup"><span data-stu-id="f3802-175">37.16 KB</span></span> |

<span data-ttu-id="f3802-176">即使子毫秒差异看起来很小，也请记住，恒定版本会持续 pollutes 缓存，并导致重新编译其他查询，同时降低其速度。</span><span class="sxs-lookup"><span data-stu-id="f3802-176">Even if the sub-millisecond difference seems small, keep in mind that the constant version continuously pollutes the cache and causes other queries to be re-compiled, slowing them down as well.</span></span>

> [!NOTE]
> <span data-ttu-id="f3802-177">除非确实需要，否则请避免构造带有表达式树 API 的查询。</span><span class="sxs-lookup"><span data-stu-id="f3802-177">Avoid constructing queries with the expression tree API unless you really need to.</span></span> <span data-ttu-id="f3802-178">除了 API 的复杂性，在使用它们时非常容易导致严重的性能问题。</span><span class="sxs-lookup"><span data-stu-id="f3802-178">Aside from the API's complexity, it's very easy to inadvertently cause significant performance issues when using them.</span></span>
