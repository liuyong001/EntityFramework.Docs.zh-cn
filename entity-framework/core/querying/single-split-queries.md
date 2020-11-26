---
title: 单个查询与拆分查询 - EF Core
description: 通过 Entity Framework Core 将查询转换为 SQL 中的单个查询和拆分查询
author: smitpatel
ms.date: 10/03/2019
uid: core/querying/single-split-queries
ms.openlocfilehash: ba282a0c5242b2eb87d681906571036d4751f6ac
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003557"
---
# <a name="single-vs-split-queries"></a>单个查询和拆分查询

## <a name="single-queries"></a>单个查询

在关系数据库中，所有相关实体通过在单个查询中引入 JOIN 来加载。

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [Blogs] AS [b]
LEFT JOIN [Post] AS [p] ON [b].[BlogId] = [p].[BlogId]
ORDER BY [b].[BlogId], [p].[PostId]
```

如果典型博客有多篇相关文章，这些文章对应的行会复制博客的信息。 这种复制会导致所谓的“笛卡尔爆炸”问题发生。 随着加载更多的一对多关系，重复的数据量可能会增长，并对应用程序性能产生负面影响。

## <a name="split-queries"></a>拆分查询

> [!NOTE]
> EF Core 5.0 中已引入此功能。 此功能仅在使用 `Include` 时可用。 [此问题](https://github.com/dotnet/efcore/issues/21234)跟踪了在不使用 `Include` 加载投影中相关数据的情况下对拆分查询的支持。

通过 EF，可以指定应将给定 LINQ 查询拆分为多个 SQL 查询。 与 JOIN 不同，拆分查询为包含的每个集合导航生成额外的 SQL 查询：

[!code-csharp[Main](../../../samples/core/Querying/RelatedData/Program.cs?name=AsSplitQuery&highlight=5)]

这会生成以下 SQL：

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
> 一对一相关实体始终都是通过 JOIN 在同一查询中进行加载的，因为这对性能没有影响。

## <a name="enabling-split-queries-globally"></a>全局启用拆分查询

还可以将拆分查询配置为应用程序上下文的默认查询：

[!code-csharp[Main](../../../samples/core/Querying/RelatedData/SplitQueriesBloggingContext.cs?name=QuerySplittingBehaviorSplitQuery&highlight=6)]

将拆分查询配置为默认查询后，仍然可以将特定查询配置为以单个查询的形式执行：

[!code-csharp[Main](../../../samples/core/Querying/RelatedData/Program.cs?name=AsSingleQuery&highlight=5)]

如果没有任何配置，默认情况下，EF Core 使用单个查询模式。 由于这可能会导致性能问题，因此，只要满足以下条件，EF Core 就会生成警告：

- EF Core 检测到查询加载了多个集合。
- 用户未全局配置查询拆分模式。
- 用户未在查询上使用 `AsSingleQuery`/`AsSplitQuery` 运算符。

若要关闭警告，请全局配置查询拆分模式，或在查询级别将其配置为适当的值。

## <a name="characteristics-of-split-queries"></a>拆分查询的特征

虽然拆分查询避免了与 JOIN 和笛卡尔爆炸相关的性能问题，但它也有一些缺点：

- 虽然大多数数据库对单个查询保证数据一致性，但对多个查询不存在这样的保证。 如果在执行查询时同时更新数据库，生成的数据可能会不一致。 这可以通过将查询包装在可序列化或快照事务中来缓解，尽管这样做本身可能会产生性能问题。 有关详细信息，请参见数据库器文档。
- 当前，每个查询都意味着对数据库进行一次额外的网络往返。 多次网络往返会降低性能，尤其是在数据库延迟很高的情况下（例如云服务）。
- 虽然有些数据库（带有 MARS 的 SQL Server、Sqlite）允许同时使用多个查询的结果，但大多数数据库在任何给定时间点只允许一个查询处于活动状态。 因此，在执行以后的查询之前，必须先在应用程序的内存中缓冲先前查询的所有结果，这将增加内存需求。

遗憾的是，没有一种加载相关实体的策略可以适用于所有情况。 请仔细考虑单个查询和拆分查询的优缺点，以便选择能够满足你需求的策略。
