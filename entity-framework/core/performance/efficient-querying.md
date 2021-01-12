---
title: 高效查询-EF Core
description: 使用 Entity Framework Core 高效查询的性能指南
author: roji
ms.date: 12/1/2020
uid: core/performance/efficient-querying
ms.openlocfilehash: e945a1e0f734d62ce8948904bcbe819455fcbefa
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128480"
---
# <a name="efficient-querying"></a>高效查询

有效地进行查询是一种广泛的主题，涵盖作为索引、相关实体加载策略以及许多其他主题的主体。 本部分详细介绍了一些用于更快地进行查询的常见主题，以及用户通常会遇到的问题。

## <a name="use-indexes-properly"></a>正确使用索引

查询能否快速使用索引的主要决定因素是：数据库通常用于保存大量数据，而遍历整个表的查询通常是严重的性能问题的根源所在。 索引问题不太容易发现，因为给定的查询是否将使用索引。 例如：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#Indexes)]

发现索引问题的一个好方法是首先查明慢速查询，然后通过数据库的偏好工具检查其查询计划;有关如何执行此操作的详细信息，请参阅 " [性能诊断](xref:core/performance/performance-diagnosis) " 页。 查询计划显示查询是遍历整个表，还是使用索引。

作为一般规则，没有任何特殊的 EF 知识可用于使用索引或诊断与它们相关的性能问题;与索引相关的一般数据库知识与 EF 应用程序相关，与不使用 EF 的应用程序一样。 下表列出了在使用索引时要记住的一些一般准则：

* 尽管索引可以提高查询速度，但它们也会减缓更新，因为它们需要保持最新状态。 避免定义不需要的索引，并考虑使用 [索引筛选器](xref:core/modeling/indexes#index-filter) 将索引限制为行的子集，从而减少此开销。
* 复合索引可以加速对多个列进行筛选的查询，但也可以加速不筛选索引的所有列的查询，具体取决于排序。 例如，列 A 和列 B 上的索引加快了 A 和 B 筛选查询，并查询仅筛选了筛选器，但它并不能加速仅由 B 进行的查询筛选。
* 如果查询按表达式筛选列 (例如 `price / 2`) ，则无法使用简单索引。 但是，您可以为表达式定义 [存储的持久化列](xref:core/modeling/generated-properties#computed-columns) ，并对该列创建索引。 某些数据库还支持表达式索引，可以直接使用表达式索引来加快通过任何表达式筛选的查询。
* 不同数据库允许以各种方式配置索引，在许多情况下，EF Core 提供程序通过流畅的 API 公开这些索引。 例如，SQL Server 提供程序允许你配置索引是否为 [聚集](xref:core/providers/sql-server/indexes#clustering)索引，或设置其 [填充因子](xref:core/providers/sql-server/indexes#fill-factor)。 有关详细信息，请查阅提供商的文档。

## <a name="project-only-properties-you-need"></a>仅限项目需要的属性

EF Core 使得查询实体实例变得非常简单，然后在代码中使用这些实例。 但是，查询实体实例通常可以从数据库中提取比需要的更多数据。 考虑以下情况：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#ProjectEntities)]

尽管此代码实际上只需要每个博客的 `Url` 属性，但会提取整个博客实体，并且不需要的列将从数据库传输：

```sql
SELECT [b].[BlogId], [b].[CreationDate], [b].[Name], [b].[Rating], [b].[Url]
FROM [Blogs] AS [b]
```

这可以通过使用 `Select` 来告诉 EF 要投影哪些列：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#ProjectSingleProperty)]

生成的 SQL 仅拉取所需的列：

```csharp
SELECT [b].[Url]
FROM [Blogs] AS [b]
```

如果需要投影多个列，请使用所需的属性投影到 c # 匿名类型。

请注意，这种方法对只读查询非常有用，但如果您需要 *更新* 提取的博客，事情会变得更加复杂，因为 EF 的更改跟踪仅适用于实体实例。 通过附加修改后的博客实例，并告诉 EF 哪些属性已更改，可以执行更新而无需加载整个实体。

## <a name="limit-the-resultset-size"></a>限制结果集大小

默认情况下，查询将返回与筛选器匹配的所有行：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#NoLimit)]

由于返回的行数取决于数据库中的实际数据，因此不可能知道将从数据库中加载的数据量、结果占用的内存量，以及处理这些结果时将生成多少额外负载 (例如，通过网络) 发送到用户浏览器。 至关重要，测试数据库经常包含很少的数据，以便在测试时一切都能正常运行，但当查询开始在实际数据上运行并且返回了很多行时，会突然出现性能问题。

因此，通常有必要提供限制结果数的考虑：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#Limit25)]

你的 UI 至少可能会显示一条消息，指示数据库中可能存在更多行 (并允许) 的其他方式检索这些行。 全面的解决方案将实现 *分页*，其中 UI 一次只显示一定数量的行，并允许用户根据需要转到下一页。这通常会组合 <xref:System.Linq.Enumerable.Take%2A> 和 <xref:System.Linq.Enumerable.Skip%2A> 运算符，以便每次在结果集中选择特定范围。

## <a name="avoid-cartesian-explosion-when-loading-related-entities"></a>在加载相关实体时避免笛卡尔分解

在关系数据库中，所有相关实体通过在单个查询中引入 JOIN 来加载。

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [Blogs] AS [b]
LEFT JOIN [Post] AS [p] ON [b].[BlogId] = [p].[BlogId]
ORDER BY [b].[BlogId], [p].[PostId]
```

如果典型博客有多篇相关文章，这些文章对应的行会复制博客的信息。 这种复制会导致所谓的“笛卡尔爆炸”问题发生。 随着加载更多的一对多关系，重复的数据量可能会增长，并对应用程序性能产生负面影响。

EF 允许使用 "拆分查询" 来避免这种影响，后者通过单独的查询来加载相关实体。 有关详细信息，请阅读 [有关拆分和单个查询的文档](xref:core/querying/single-split-queries)。

> [!NOTE]
> [拆分查询](xref:core/querying/single-split-queries)的当前实现执行每个查询的往返。 我们计划在将来改进这一点，并在单次往返中执行所有查询。

## <a name="load-related-entities-eagerly-when-possible"></a>尽可能积极加载相关实体

建议在继续此部分之前，先阅读 [相关实体的专用页面](xref:core/querying/related-data) 。

在处理相关实体时，我们通常会提前了解需要加载的内容：典型的示例是加载一组特定的博客以及所有发布内容。 在这些情况下，最好使用 [预先加载](xref:core/querying/related-data/eager)，以便 EF 可以在一往返中提取所有必需的数据。 EF Core 5.0 中引入的 [筛选的包含](xref:core/querying/related-data/eager#filtered-include) 功能还允许您限制要加载的相关实体，同时使加载过程保持预先，进而可行单个往返：

[!code-csharp[Main](../../../samples/core/Querying/RelatedData/Program.cs#FilteredInclude)]

在其他情况下，我们可能不知道我们要在获取其主体实体之前需要哪些相关实体。 例如，在加载某些博客时，可能需要咨询其他一些数据源（可能是 webservice），以了解我们是否对该博客文章感兴趣。 在这些情况下，可以使用 [显式](xref:core/querying/related-data/explicit) 或 [延迟](xref:core/querying/related-data/lazy) 加载单独提取相关实体，并填充博客的文章导航。 请注意，由于这些方法不是预先的，因此需要对数据库进行额外的往返，这是一种缓慢的源;根据具体的方案，只需始终加载所有帖子，而不是执行额外的往返，并有选择地仅获取所需的内容，这种方法可能更高效。

### <a name="beware-of-lazy-loading"></a>注意延迟加载

[延迟加载](xref:core/querying/related-data/lazy) 通常是一种非常有用的方法来编写数据库逻辑，因为 EF Core 在代码访问数据库时，它们会从数据库中自动加载相关实体。 这可以避免加载 (（如 [显式加载](xref:core/querying/related-data/explicit)) ）所需的相关实体，似乎使程序员不必处理相关实体。 不过，延迟加载特别容易产生不必要的额外往返，这会降低应用程序的速度。

考虑以下情况：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#NPlusOne)]

这种看似合法的代码段会循环访问所有博客及其帖子，打印出来。启用 EF Core 的 [语句日志记录](xref:core/logging-events-diagnostics/index) 会显示以下内容：

```console
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      SELECT [b].[BlogId], [b].[Rating], [b].[Url]
      FROM [Blogs] AS [b]
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (5ms) [Parameters=[@__p_0='1'], CommandType='Text', CommandTimeout='30']
      SELECT [p].[PostId], [p].[BlogId], [p].[Content], [p].[Title]
      FROM [Post] AS [p]
      WHERE [p].[BlogId] = @__p_0
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (1ms) [Parameters=[@__p_0='2'], CommandType='Text', CommandTimeout='30']
      SELECT [p].[PostId], [p].[BlogId], [p].[Content], [p].[Title]
      FROM [Post] AS [p]
      WHERE [p].[BlogId] = @__p_0
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (1ms) [Parameters=[@__p_0='3'], CommandType='Text', CommandTimeout='30']
      SELECT [p].[PostId], [p].[BlogId], [p].[Content], [p].[Title]
      FROM [Post] AS [p]
      WHERE [p].[BlogId] = @__p_0

... and so on
```

这是怎么回事？ 为什么要为上述简单循环发送所有这些查询？ 使用延迟加载时，博客的张贴内容仅 (延迟) 在访问其 Post 属性时加载;因此，内部 foreach 中的每个迭代都将在其自身的往返中触发附加的数据库查询。 因此，在初始查询加载所有博客后，我们会在 *每个博客* 中创建另一个查询，并加载其所有文章;这有时称为 *N + 1* 问题，这可能会导致严重的性能问题。

假设我们将需要所有的博客文章，可以改为在此处使用预先加载。 我们可以使用 [Include](xref:core/querying/related-data/eager#eager-loading) 运算符来执行加载，但由于我们只需要博客的 url (并且应该只 [加载) 所需的内容](xref:core/performance/efficient-querying#project-only-properties-you-need) 。 我们将改为使用投影：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#EagerlyLoadRelatedAndProject)]

这会使 EF Core 在单个查询中提取所有博客及其帖子。 在某些情况下，使用 [拆分查询](xref:core/querying/single-split-queries)来避免笛卡尔分解效果也很有用。

> [!WARNING]
> 由于延迟加载使得无意中触发了 N + 1 问题，因此建议避免出现这种情况。 当发生数据库往返时，预先或显式加载会使源代码非常清楚。

## <a name="buffering-and-streaming"></a>缓冲和流式处理

缓冲是指将所有查询结果加载到内存中，而流式处理意味着 EF 每次都要将应用程序作为单个结果，绝不会将整个结果集包含在内存中。 原则上，流式处理查询的内存要求是固定的，无论查询返回1行还是1000，它们都是相同的。另一方面，缓冲查询需要更多的内存来返回更多的行。 对于产生大型结果集的查询，这可能是一个重要的性能因素。

查询缓冲区或流是否取决于其计算方式：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#BufferingAndStreaming)]

如果查询只返回几个结果，则可能无需担心。 但是，如果您的查询可能返回大量的行，则有必要为流式传输而不是缓冲。

> [!NOTE]
> 如果要 <xref:System.Linq.Enumerable.ToList%2A> <xref:System.Linq.Enumerable.ToArray%2A> 在结果中使用另一个 LINQ 运算符，请避免使用或，这将不必要地将所有结果缓冲到内存中。 请改用 <xref:System.Linq.Enumerable.AsEnumerable%2A>。

### <a name="internal-buffering-by-ef"></a>EF 的内部缓冲

在某些情况下，无论您对查询的评估方式如何，EF 都会在内部将结果集缓冲。 出现这种情况的两种情况是：

* 重试执行策略已准备就绪。 这样做是为了确保以后重试查询时返回相同的结果。
* 使用 [split 查询](xref:core/querying/single-split-queries) 时，将缓冲除最后一个查询之外的所有结果，除非在 SQL Server 上启用了 MARS。 这是因为通常无法同时激活多个查询结果集。

请注意，除通过 LINQ 运算符引起的任何缓冲外，还会发生此内部缓冲。 例如，如果在 <xref:System.Linq.Enumerable.ToList%2A> 查询中使用，并且正在重试执行策略，则会将结果集加载到内存中 *两* 次：一次由 EF 提供一次，一次加载 <xref:System.Linq.Enumerable.ToList%2A> 。

## <a name="tracking-no-tracking-and-identity-resolution"></a>跟踪，无跟踪和标识解析

在继续此部分之前，建议阅读 [有关跟踪和无跟踪的专用页面](xref:core/querying/tracking) 。

EF 默认跟踪实体实例，因此在调用时，将会检测并保存对实体实例所做的更改 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。 跟踪查询的另一个作用是 EF 检测是否已为你的数据加载了一个实例，并将自动返回该跟踪的实例，而不是返回新的实例;这称为 *标识解析*。 从性能的角度来看，"更改跟踪" 的含义如下：

* EF 在内部维护跟踪实例的字典。 加载新数据时，EF 会检查字典，以查看是否已为该实体的密钥跟踪了某个实例 (标识解析) 。 加载查询结果时，字典维护和查找会花费一些时间。
* 在将加载的实例处理到应用程序之前，将实例 *快照* 并在内部保留快照。 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A>调用时，应用程序的实例与快照进行比较，以发现要保存的更改。 快照占用更多内存，快照进程本身需要时间;有时可以通过 [值](xref:core/modeling/value-comparers)比较器指定不同的、可能更有效的快照行为，或使用更改跟踪代理来完全绕过快照进程， (虽然附带了自己的一组缺点) 。

在未将更改保存回数据库的只读方案中，可以通过使用 [无跟踪查询](xref:core/querying/tracking#no-tracking-queries)来避免上述开销。 但是，因为无跟踪查询不会执行标识解析，所以，由多个其他加载行引用的数据库行将被具体化为不同的实例。

为了说明这一点，假设我们在从数据库中加载大量的文章，以及每个帖子引用的博客。 如果发生了100发布以引用同一博客，则跟踪查询将通过标识解析来检测这种情况，并且所有 Post 实例都将引用相同的重复数据的博客实例。 相反，无跟踪查询与相同的博客100次重复，并且必须相应地编写应用程序代码。

下面是比较跟踪与不跟踪行为的基准比较的结果：加载10个博客，其中每个日志都包含20个帖子。 [源代码在此处提供](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Benchmarks/QueryTrackingBehavior.cs)，可将其用作自己度量的基础。

|       方法 | NumBlogs | NumPostsPerBlog |       平均值 |    错误 |   标准偏差 |     中值 | 比率 | RatioSD |   第0代 |   第1代 | 第2代 | 已分配 |
|------------- |--------- |---------------- |-----------:|---------:|---------:|-----------:|------:|--------:|--------:|--------:|------:|----------:|
|   AsTracking |       10 |              20 | 1414.7 美国 | 27.20 美国 | 45.44 美国 | 1405.5 美国 |  1.00 |    0.00 | 60.5469 | 13.6719 |     - | 380.11 KB |
| AsNoTracking |       10 |              20 |   993.3 美国 | 24.04 美国 | 65.40 美国 |   966.2 美国 |  0.71 |    0.05 | 37.1094 |  6.8359 |     - | 232.89 KB |

最后，通过利用无跟踪查询然后将返回的实例附加到上下文中，可以在不影响更改跟踪的开销的情况下执行更新，同时指定要进行的更改。 这会将更改跟踪的负担从 EF 传输到用户，并且仅当更改跟踪开销已通过分析或基准证明显示为不可接受时才应尝试。

## <a name="using-raw-sql"></a>使用原始 SQL

在某些情况下，你的查询存在更多优化的 SQL，而 EF 不会生成。 如果 SQL 构造是特定于您的数据库的扩展，或者是不受支持的，则可能会发生这种情况。 在这些情况下，手动编写 SQL 可以显著提高性能，而 EF 支持多种方法来实现此目的。

* [直接在查询中](xref:core/querying/raw-sql)使用原始 SQL，例如通过 <xref:Microsoft.EntityFrameworkCore.RelationalQueryableExtensions.FromSqlRaw%2A> 。 EF 甚至使你可以使用常规 LINQ 查询来编写原始 SQL，使你能够仅在原始 SQL 中表达部分查询。 当只需在代码库中的单个查询中使用原始 SQL 时，这是一种很好的方法。
* 定义 (UDF) 的 [用户定义函数](xref:core/querying/database-functions) ，然后从查询中调用该函数。 请注意，从5.0 开始，Udf 允许 Udf 返回完整结果集-这些函数称为表值函数 (Tvf) -并且还允许将映射 `DbSet` 到函数，使其看起来就像是另一个表。
* 定义数据库视图，并在查询中对其进行查询。 请注意，与函数不同，视图不能接受参数。

> [!NOTE]
> 通常，在确保 EF 无法生成所需的 SQL 时，以及当性能非常重要，足以使给定的查询进行调整时，通常应将原始 SQL 用作最后的手段。 使用原始 SQL 会带来相当大的维护缺点。

## <a name="asynchronous-programming"></a>异步编程

作为一般规则，若要使应用程序具有可伸缩性，必须始终使用异步 Api，而不是同步 (例如， <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A> 而不是 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A>) 。 同步 Api 在数据库 i/o 期间阻止线程，从而增加了对线程的需要和必须发生的线程上下文切换次数。

有关详细信息，请参阅 [异步编程](xref:core/miscellaneous/async)中的页面。

> [!WARNING]
> 避免在同一应用程序中混合同步和异步代码，这很容易意外触发微妙的线程池不足问题。

## <a name="additional-resources"></a>其他资源

请参阅 null 比较文档页的 " [性能" 部分](xref:core/querying/null-comparisons#writing-performant-queries) ，了解比较可为 null 值时的一些最佳实践。
