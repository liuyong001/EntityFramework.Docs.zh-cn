---
title: 单个查询与拆分查询 - EF Core
description: 通过 Entity Framework Core 将查询转换为 SQL 中的单个查询和拆分查询
author: smitpatel
ms.date: 10/03/2019
uid: core/querying/single-split-queries
ms.openlocfilehash: 1c99d931c01b99de199710ffe661e1aac7a37263
ms.sourcegitcommit: f3512e3a98e685a3ba409c1d0157ce85cc390cf4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2020
ms.locfileid: "94431174"
---
# <a name="single-vs-split-queries"></a><span data-ttu-id="a384b-103">单个查询和拆分查询</span><span class="sxs-lookup"><span data-stu-id="a384b-103">Single vs. split queries</span></span>

## <a name="single-queries"></a><span data-ttu-id="a384b-104">单个查询</span><span class="sxs-lookup"><span data-stu-id="a384b-104">Single queries</span></span>

<span data-ttu-id="a384b-105">在关系数据库中，所有相关实体通过在单个查询中引入 JOIN 来加载。</span><span class="sxs-lookup"><span data-stu-id="a384b-105">In relational databases, all related entities are loaded by introducing JOINs in single query.</span></span>

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [Blogs] AS [b]
LEFT JOIN [Post] AS [p] ON [b].[BlogId] = [p].[BlogId]
ORDER BY [b].[BlogId], [p].[PostId]
```

<span data-ttu-id="a384b-106">如果典型博客有多篇相关文章，这些文章对应的行会复制博客的信息。</span><span class="sxs-lookup"><span data-stu-id="a384b-106">If a typical blog has multiple related posts, rows for these posts will duplicate the blog's information.</span></span> <span data-ttu-id="a384b-107">这种复制会导致所谓的“笛卡尔爆炸”问题发生。</span><span class="sxs-lookup"><span data-stu-id="a384b-107">This duplication leads to the so-called "cartesian explosion" problem.</span></span> <span data-ttu-id="a384b-108">随着加载更多的一对多关系，重复的数据量可能会增长，并对应用程序性能产生负面影响。</span><span class="sxs-lookup"><span data-stu-id="a384b-108">As more one-to-many relationships are loaded, the amount of duplicated data may grow and adversely affect the performance of your application.</span></span>

## <a name="split-queries"></a><span data-ttu-id="a384b-109">拆分查询</span><span class="sxs-lookup"><span data-stu-id="a384b-109">Split queries</span></span>

> [!NOTE]
> <span data-ttu-id="a384b-110">EF Core 5.0 中已引入了此功能。</span><span class="sxs-lookup"><span data-stu-id="a384b-110">This feature is introduced in EF Core 5.0.</span></span> <span data-ttu-id="a384b-111">此功能仅在使用 `Include` 时可用。</span><span class="sxs-lookup"><span data-stu-id="a384b-111">It only works when using `Include`.</span></span> <span data-ttu-id="a384b-112">[此问题](https://github.com/dotnet/efcore/issues/21234)跟踪了在不使用 `Include` 加载投影中相关数据的情况下对拆分查询的支持。</span><span class="sxs-lookup"><span data-stu-id="a384b-112">[This issue](https://github.com/dotnet/efcore/issues/21234) is tracking support for split query when loading related data in projection without `Include`.</span></span>

<span data-ttu-id="a384b-113">通过 EF，可以指定应将给定 LINQ 查询拆分为多个 SQL 查询。</span><span class="sxs-lookup"><span data-stu-id="a384b-113">EF allows you to specify that a given LINQ query should be *split* into multiple SQL queries.</span></span> <span data-ttu-id="a384b-114">与 JOIN 不同，拆分查询为包含的每个集合导航生成额外的 SQL 查询：</span><span class="sxs-lookup"><span data-stu-id="a384b-114">Instead of JOINs, split queries generate an additional SQL query for each included collection navigation:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RelatedData/Program.cs?name=AsSplitQuery&highlight=5)]

<span data-ttu-id="a384b-115">这会生成以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="a384b-115">It will produce the following SQL:</span></span>

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url]
FROM [Blogs] AS [b]
ORDER BY [b].[BlogId]

SELECT [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title], [b].[BlogId]
FROM [Blogs] AS [b]
INNER JOIN [Post] AS [p] ON [b].[BlogId] = [p].[BlogId]
ORDER BY [b].[BlogId]
```

> [!NOTE]
> <span data-ttu-id="a384b-116">一对一相关实体始终都是通过 JOIN 在同一查询中进行加载的，因为这对性能没有影响。</span><span class="sxs-lookup"><span data-stu-id="a384b-116">One-to-one related entities are always loaded via JOINs in the same query, as it has no performance impact.</span></span>

## <a name="enabling-split-queries-globally"></a><span data-ttu-id="a384b-117">全局启用拆分查询</span><span class="sxs-lookup"><span data-stu-id="a384b-117">Enabling split queries globally</span></span>

<span data-ttu-id="a384b-118">还可以将拆分查询配置为应用程序上下文的默认查询：</span><span class="sxs-lookup"><span data-stu-id="a384b-118">You can also configure split queries as the default for your application's context:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RelatedData/SplitQueriesBloggingContext.cs?name=QuerySplittingBehaviorSplitQuery&highlight=6)]

<span data-ttu-id="a384b-119">将拆分查询配置为默认查询后，仍然可以将特定查询配置为以单个查询的形式执行：</span><span class="sxs-lookup"><span data-stu-id="a384b-119">When split queries are configured as the default, it's still possible to configure specific queries to execute as single queries:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RelatedData/Program.cs?name=AsSingleQuery&highlight=5)]

<span data-ttu-id="a384b-120">如果没有任何配置，默认情况下，EF Core 使用单个查询模式。</span><span class="sxs-lookup"><span data-stu-id="a384b-120">EF Core uses single query mode by default in the absence of any configuration.</span></span> <span data-ttu-id="a384b-121">由于这可能会导致性能问题，因此，只要满足以下条件，EF Core 就会生成警告：</span><span class="sxs-lookup"><span data-stu-id="a384b-121">Since it may cause performance issues, EF Core generates a warning whenever following conditions are met:</span></span>

- <span data-ttu-id="a384b-122">EF Core 检测到查询加载了多个集合。</span><span class="sxs-lookup"><span data-stu-id="a384b-122">EF Core detects that the query loads multiple collections.</span></span>
- <span data-ttu-id="a384b-123">用户未全局配置查询拆分模式。</span><span class="sxs-lookup"><span data-stu-id="a384b-123">User hasn't configured query splitting mode globally.</span></span>
- <span data-ttu-id="a384b-124">用户未在查询上使用 `AsSingleQuery`/`AsSplitQuery` 运算符。</span><span class="sxs-lookup"><span data-stu-id="a384b-124">User hasn't used `AsSingleQuery`/`AsSplitQuery` operator on the query.</span></span>

<span data-ttu-id="a384b-125">若要关闭警告，请全局配置查询拆分模式，或在查询级别将其配置为适当的值。</span><span class="sxs-lookup"><span data-stu-id="a384b-125">To turn off the warning, configure query splitting mode globally or at the query level to an appropriate value.</span></span>

## <a name="characteristics-of-split-queries"></a><span data-ttu-id="a384b-126">拆分查询的特征</span><span class="sxs-lookup"><span data-stu-id="a384b-126">Characteristics of split queries</span></span>

<span data-ttu-id="a384b-127">虽然拆分查询避免了与 JOIN 和笛卡尔爆炸相关的性能问题，但它也有一些缺点：</span><span class="sxs-lookup"><span data-stu-id="a384b-127">While split query avoids the performance issues associated with JOINs and cartesian explosion, it also has some drawbacks:</span></span>

- <span data-ttu-id="a384b-128">虽然大多数数据库对单个查询保证数据一致性，但对多个查询不存在这样的保证。</span><span class="sxs-lookup"><span data-stu-id="a384b-128">While most databases guarantee data consistency for single queries, no such guarantees exist for multiple queries.</span></span> <span data-ttu-id="a384b-129">如果在执行查询时同时更新数据库，生成的数据可能会不一致。</span><span class="sxs-lookup"><span data-stu-id="a384b-129">If the database is updated concurrently when executing your queries, resulting data may not be consistent.</span></span> <span data-ttu-id="a384b-130">这可以通过将查询包装在可序列化或快照事务中来缓解，尽管这样做本身可能会产生性能问题。</span><span class="sxs-lookup"><span data-stu-id="a384b-130">You can mitigate it by wrapping the queries in a serializable or snapshot transaction, although doing so may create performance issues of its own.</span></span> <span data-ttu-id="a384b-131">有关详细信息，请参见数据库器文档。</span><span class="sxs-lookup"><span data-stu-id="a384b-131">For more information, see your database's documentation.</span></span>
- <span data-ttu-id="a384b-132">当前，每个查询都意味着对数据库进行一次额外的网络往返。</span><span class="sxs-lookup"><span data-stu-id="a384b-132">Each query currently implies an additional network roundtrip to your database.</span></span> <span data-ttu-id="a384b-133">多次网络往返会降低性能，尤其是在数据库延迟很高的情况下（例如云服务）。</span><span class="sxs-lookup"><span data-stu-id="a384b-133">Multiple network roundtrip can degrade performance, especially where latency to the database is high (for example, cloud services).</span></span>
- <span data-ttu-id="a384b-134">虽然有些数据库（带有 MARS 的 SQL Server、Sqlite）允许同时使用多个查询的结果，但大多数数据库在任何给定时间点只允许一个查询处于活动状态。</span><span class="sxs-lookup"><span data-stu-id="a384b-134">While some databases allow consuming the results of multiple queries at the same time (SQL Server with MARS, Sqlite), most allow only a single query to be active at any given point.</span></span> <span data-ttu-id="a384b-135">因此，在执行以后的查询之前，必须先在应用程序的内存中缓冲先前查询的所有结果，这将增加内存需求。</span><span class="sxs-lookup"><span data-stu-id="a384b-135">So all results from earlier queries must be buffered in your application's memory before executing later queries, which leads to increased memory requirements.</span></span>

<span data-ttu-id="a384b-136">遗憾的是，没有一种加载相关实体的策略可以适用于所有情况。</span><span class="sxs-lookup"><span data-stu-id="a384b-136">Unfortunately, there isn't one strategy for loading related entities that fits all scenarios.</span></span> <span data-ttu-id="a384b-137">请仔细考虑单个查询和拆分查询的优缺点，以便选择能够满足你需求的策略。</span><span class="sxs-lookup"><span data-stu-id="a384b-137">Carefully consider the advantages and disadvantages of single and split queries to select the one that fits your needs.</span></span>
