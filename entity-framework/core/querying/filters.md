---
title: 全局查询筛选器 - EF Core
description: 将 Entity Framework Core 与全局查询筛选器结合使用来筛选结果
author: maumar
ms.date: 11/03/2017
uid: core/querying/filters
ms.openlocfilehash: 6436f9f8e2e09d44ef9528fd2022720d40095fe0
ms.sourcegitcommit: f3512e3a98e685a3ba409c1d0157ce85cc390cf4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2020
ms.locfileid: "94430126"
---
# <a name="global-query-filters"></a>全局查询筛选器

全局查询筛选器是应用于元数据模型（通常为 `OnModelCreating`）中的实体类型的 LINQ 查询谓词。 查询谓词即通常传递给 LINQ `Where` 查询运算符的布尔表达式。  EF Core 会自动将此类筛选器应用于涉及这些实体类型的任何 LINQ 查询。  EF Core 还将其应用于使用 Include 或导航属性进行间接引用的实体类型。 此功能的一些常见应用如下：

* 软删除 - 实体类型定义 `IsDeleted` 属性。
* 多租户 - 实体类型定义 `TenantId` 属性。

## <a name="example"></a>示例

下面的示例演示了如何使用全局查询筛选器在简单的博客模型中实现多租户和软删除查询行为。

> [!TIP]
> 可在 GitHub 上查看此文章的[示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Querying/QueryFilters)。

首先，定义实体：

[!code-csharp[Main](../../../samples/core/Querying/QueryFilters/Entities.cs#Entities)]

请注意 `Blog` 实体上的 `_tenantId` 字段的声明。 此字段用于将每个 Blog 实例与特定租户相关联。 同时在 `Post` 实体类型上定义了 `IsDeleted` 属性。 此属性用于跟踪某个 Post 实例是否已“软删除”。 也就是说，实例只是被标记为已删除，而非真正删除了基础数据。

接下来，使用 `HasQueryFilter` API 在 `OnModelCreating` 中配置查询筛选器。

[!code-csharp[Main](../../../samples/core/Querying/QueryFilters/BloggingContext.cs#FilterConfiguration)]

传递给 `HasQueryFilter` 调用的谓词表达式将立即自动应用于这些类型的任何 LINQ 查询。

> [!TIP]
> 请注意 DbContext 实例级别字段的使用：`_tenantId` 用于设置当前租户。 模型级筛选器将使用正确上下文实例（即执行查询的实例）中的值。

> [!NOTE]
> 目前不能在同一个实体中定义多个查询筛选器，只会应用最后一个筛选器。 但是，可以使用逻辑 `AND` 运算符（[C# 中为 `&&`](/dotnet/csharp/language-reference/operators/boolean-logical-operators#conditional-logical-and-operator-)）定义含有多种条件的单个筛选器。

## <a name="use-of-navigations"></a>使用导航

还可以在定义全局查询筛选器时使用导航。 在查询筛选器中使用导航将导致以递归方式应用查询筛选器这一结果。 当 EF Core 展开查询筛选器中使用的导航时，它还会应用在引用的实体上定义的查询筛选器。

为了进行说明，请按以下方式在 `OnModelCreating` 中配置查询筛选器：[!code-csharp[Main](../../../samples/core/Querying/QueryFilters/FilteredBloggingContextRequired.cs#NavigationInFilter)]

接下来，查询所有 `Blog` 实体：[!code-csharp[Main](../../../samples/core/Querying/QueryFilters/FilteredBloggingContextRequired.cs#QueriesNavigation)]

此查询将生成以下 SQL，它将应用为 `Blog` 和 `Post` 实体定义的查询筛选器：

```sql
SELECT [b].[BlogId], [b].[Name], [b].[Url]
FROM [Blogs] AS [b]
WHERE (
    SELECT COUNT(*)
    FROM [Posts] AS [p]
    WHERE ([p].[Title] LIKE N'%fish%') AND ([b].[BlogId] = [p].[BlogId])) > 0
```

> [!NOTE]
> 目前，EF Core 不会检测全局查询筛选器定义中的循环，因此需在定义它们时小心谨慎。 如果指定错误，这些循环可能在查询转换期间导致无限循环。

## <a name="accessing-entity-with-query-filter-using-required-navigation"></a>使用必需的导航访问具有查询筛选器的实体

> [!CAUTION]
> 如果使用必需的导航访问定义了全局查询筛选器的实体，则可能导致意外结果。

必需的导航要求始终存在相关实体。 如果查询筛选器筛选出了必需的相关实体，则父实体不会出现在结果中。 因此，结果中的元素数量可能会低于预期。

为了说明此问题，我们可使用上面指定的 `Blog` 和 `Post` 实体，同时使用下面的 `OnModelCreating` 方法：

[!code-csharp[Main](../../../samples/core/Querying/QueryFilters/FilteredBloggingContextRequired.cs#IncorrectFilter)]

可通过以下数据对模型进行种子设定：

[!code-csharp[Main](../../../samples/core/Querying/QueryFilters/Program.cs#SeedData)]

执行两个查询时，可能会观察到此问题：

[!code-csharp[Main](../../../samples/core/Querying/QueryFilters/Program.cs#Queries)]

在上述设置中，第一个查询返回全部 6 个 `Post`，而第二个查询仅返回 3 个。 发生这种不匹配情况的原因是第二个查询中的 `Include` 方法会加载相关的 `Blog` 实体。 由于需要在 `Blog` 和 `Post` 之间导航，因此在构造查询时，EF Core 使用了 `INNER JOIN`：

```sql
SELECT [p].[PostId], [p].[BlogId], [p].[Content], [p].[IsDeleted], [p].[Title], [t].[BlogId], [t].[Name], [t].[Url]
FROM [Posts] AS [p]
INNER JOIN (
    SELECT [b].[BlogId], [b].[Name], [b].[Url]
    FROM [Blogs] AS [b]
    WHERE [b].[Url] LIKE N'%fish%'
) AS [t] ON [p].[BlogId] = [t].[BlogId]
```

使用 `INNER JOIN` 会筛选出其相关 `Blog` 已被全局查询筛选器删除的所有 `Post`。

可使用可选导航来解决此问题，而不使用必需导航。
这样一来，第一个查询与之前相同，但第二个查询现将生成 `LEFT JOIN` 并返回 6 个结果。

[!code-csharp[Main](../../../samples/core/Querying/QueryFilters/FilteredBloggingContextRequired.cs#OptionalNavigation)]

替代方法是在 `Blog` 和 `Post` 实体上指定一致的筛选器。
这样，匹配的筛选器就会同时应用于 `Blog` 和 `Post`。 可能导致出现意外状态的 `Post` 已被删除，且两个查询都返回 3 个结果。

[!code-csharp[Main](../../../samples/core/Querying/QueryFilters/FilteredBloggingContextRequired.cs#MatchingFilters)]

## <a name="disabling-filters"></a>禁用筛选器

可使用 <xref:Microsoft.EntityFrameworkCore.EntityFrameworkQueryableExtensions.IgnoreQueryFilters%2A> 运算符对各个 LINQ 查询禁用筛选器。

[!code-csharp[Main](../../../samples/core/Querying/QueryFilters/Program.cs#IgnoreFilters)]

## <a name="limitations"></a>限制

全局查询筛选器具有以下限制：

* 仅可为继承层次结构的根实体类型定义筛选器。
