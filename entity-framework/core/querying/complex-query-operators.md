---
title: 复杂查询运算符 - EF Core
description: 关于使用 Entity Framework Core 时更复杂的 LINQ 查询运算符的深度信息
author: smitpatel
ms.date: 10/03/2019
uid: core/querying/complex-query-operators
ms.openlocfilehash: 84c2518972355d31cf5a6a7bafc57b44162412c8
ms.sourcegitcommit: f3512e3a98e685a3ba409c1d0157ce85cc390cf4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2020
ms.locfileid: "94430477"
---
# <a name="complex-query-operators"></a>复杂查询运算符

语言集成查询 (LINQ) 包含许多用于组合多个数据源或执行复杂处理的复杂运算符。 并非所有 LINQ 运算符都会在服务器端进行适当转换。 有时，采用一种形式的查询会转换为服务器，但如果采用另一种形式，即使结果相同，也不会转换。 本页介绍部分复杂运算符及其支持的变体。 在将来的版本中，我们可能会认识更多的模式并添加其相应的转换。 另请牢记，转换支持在提供程序之间各所不同。 在 SqlServer 中转换的特定查询可能无法用于 SQLite 数据库。

> [!TIP]
> 可在 GitHub 上查看此文章的[示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Querying/ComplexQuery)。

## <a name="join"></a>联接

借助 LINQ Join 运算符，可根据每个源的键选择器连接两个数据源，并在键匹配时生成值的元组。 该运算符在关系数据库中自然而然地转换为 `INNER JOIN`。 虽然 LINQ Join 具有外部和内部键选择器，但数据库只需要一个联接条件。 因此 EF Core 通过比较外部键选择器和内部键选择器是否相等，来生成联接条件。

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#Join)]

```sql
SELECT [p].[PersonId], [p].[Name], [p].[PhotoId], [p0].[PersonPhotoId], [p0].[Caption], [p0].[Photo]
FROM [PersonPhoto] AS [p0]
INNER JOIN [Person] AS [p] ON [p0].[PersonPhotoId] = [p].[PhotoId]
```

此外，如果键选择器是匿名类型，则 EF Core 会生成一个联接条件，以比较组件是否相等。

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#JoinComposite)]

```sql
SELECT [p].[PersonId], [p].[Name], [p].[PhotoId], [p0].[PersonPhotoId], [p0].[Caption], [p0].[Photo]
FROM [PersonPhoto] AS [p0]
INNER JOIN [Person] AS [p] ON ([p0].[PersonPhotoId] = [p].[PhotoId] AND ([p0].[Caption] = N'SN'))
```

## <a name="groupjoin"></a>GroupJoin

借助 LINQ GroupJoin 运算符，可以采用与 Join 类似的方式连接两个数据源，但它会创建一组内部值，用于匹配外部元素。 执行与以下示例类似的查询将生成 `Blog` & `IEnumerable<Post>` 的结果。 由于数据库（特别是关系数据库）无法表示一组客户端对象，因此在许多情况下，GroupJoin 不会转换为服务器。 它需要从服务器获取所有数据来进行 GroupJoin，无需使用特殊选择器（下面的第一个查询）。 但如果选择器限制选定的数据，则从服务器提取所有数据可能会导致出现性能问题（下面的第二个查询）。 这就是 EF Core 不转换 GroupJoin 的原因。

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#GroupJoin)]

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#GroupJoinComposed)]

## <a name="selectmany"></a>SelectMany

借助 LINQ SelectMany 运算符，可为每个外部元素枚举集合选择器，并从每个数据源生成值的元组。 在某种程度上，它是没有任何条件的联接，因此每个外部元素都与来自集合源的元素连接。 根据集合选择器与外部数据源的关系，SelectMany 可在服务器端转换为各种不同的查询。

### <a name="collection-selector-doesnt-reference-outer"></a>集合选择器不引用外部

如果集合选择器不引用外部源中的任何内容，则结果是这两个数据源的笛卡尔乘积。 它在关系数据库中转换为 `CROSS JOIN`。

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#SelectManyConvertedToCrossJoin)]

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [Blogs] AS [b]
CROSS JOIN [Posts] AS [p]
```

### <a name="collection-selector-references-outer-in-a-where-clause"></a>集合选择器引用 where 子句中的外部

如果集合选择器具有引用外部元素的 where 子句，则 EF Core 会将其转换为数据库联接，并将谓词用作联接条件。 在对外部元素使用集合导航作为集合选择器时，通常会出现这种情况。 如果外部元素的集合为空，则不会为该外部元素生成任何结果。 但如果在集合选择器上应用 `DefaultIfEmpty`，则外部元素将与内部元素的默认值连接。 由于这种区别，应用 `DefaultIfEmpty` 时，如果缺少 `DefaultIfEmpty` 和 `LEFT JOIN`，此类查询会转换为 `INNER JOIN`。

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#SelectManyConvertedToJoin)]

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [Blogs] AS [b]
INNER JOIN [Posts] AS [p] ON [b].[BlogId] = [p].[BlogId]

SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [Blogs] AS [b]
LEFT JOIN [Posts] AS [p] ON [b].[BlogId] = [p].[BlogId]
```

### <a name="collection-selector-references-outer-in-a-non-where-case"></a>集合选择器引用非 where 情况下的外部

如果集合选择器引用的外部元素不在 where 子句中（如上所述），则它不会转换为数据库联接。 这就是我们需要为每个外部元素评估集合选择器的原因。 它在许多关系数据库中转换为 `APPLY` 操作。 如果外部元素的集合为空，则不会为该外部元素生成任何结果。 但如果在集合选择器上应用 `DefaultIfEmpty`，则外部元素将与内部元素的默认值连接。 由于这种区别，应用 `DefaultIfEmpty` 时，如果缺少 `DefaultIfEmpty` 和 `OUTER APPLY`，此类查询会转换为 `CROSS APPLY`。 像 SQLite 之类的某些数据库不支持 `APPLY` 运算符，因此此类查询可能不会进行转换。

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#SelectManyConvertedToApply)]

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], ([b].[Url] + N'=>') + [p].[Title] AS [p]
FROM [Blogs] AS [b]
CROSS APPLY [Posts] AS [p]

SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], ([b].[Url] + N'=>') + [p].[Title] AS [p]
FROM [Blogs] AS [b]
OUTER APPLY [Posts] AS [p]
```

## <a name="groupby"></a>GroupBy

LINQ GroupBy 运算符创建 `IGrouping<TKey, TElement>` 类型的结果，其中 `TKey` 和 `TElement` 可以是任意类型。 此外，`IGrouping` 实现了 `IEnumerable<TElement>`，这意味着可在分组后使用任意 LINQ 运算符来对其进行组合。 由于任何数据库结构都无法表示 `IGrouping`，因此在大多数情况下 GroupBy 运算符不会进行任何转换。 聚合运算符应用于返回标量的每个组时，该运算符可在关系数据库中转换为 SQL `GROUP BY`。 SQL `GROUP BY` 也会受到限制。 它要求只按标量值进行分组。 投影只能包含分组键列或对列应用的任何聚合。 EF Core 标识此模式并将其转换为服务器，如以下示例中所示：

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#GroupBy)]

```sql
SELECT [p].[AuthorId] AS [Key], COUNT(*) AS [Count]
FROM [Posts] AS [p]
GROUP BY [p].[AuthorId]
```

EF Core 还会转换符合以下条件的查询：分组的聚合运算符出现在 Where 或 OrderBy（或其他排序方式）LINQ 运算符中。 它在 SQL 中将 `HAVING` 子句用于 where 子句。 在应用 GroupBy 运算符之前的查询部分可以是任何复杂查询，只要它可转换为服务器即可。 此外，将聚合运算符应用于分组查询以从生成的源中移除分组后，可以像使用任何其他查询一样，在它的基础上进行组合。

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#GroupByFilter)]

```sql
SELECT [p].[AuthorId] AS [Key], COUNT(*) AS [Count]
FROM [Posts] AS [p]
GROUP BY [p].[AuthorId]
HAVING COUNT(*) > 0
ORDER BY [p].[AuthorId]
```

EF Core 支持的聚合运算符如下所示

- 平均值
- 计数
- LongCount
- 最大值
- Min
- Sum

## <a name="left-join"></a>Left Join

虽然 Left Join 不是 LINQ 运算符，但关系数据库具有常用于查询的 Left Join 的概念。 LINQ 查询中的特定模式提供与服务器上的 `LEFT JOIN` 相同的结果。 EF Core 标识此类模式，并在服务器端生成等效的 `LEFT JOIN`。 该模式包括在两个数据源之间创建 GroupJoin，然后通过对分组源使用 SelectMany 运算符与 DefaultIfEmpty 来平展分组，从而在内部不具有相关元素时匹配 null。 下面的示例显示该模式的样式及其生成的内容。

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#LeftJoin)]

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [Blogs] AS [b]
LEFT JOIN [Posts] AS [p] ON [b].[BlogId] = [p].[BlogId]
```

以上模式在表达式树中创建复杂的结构。 因此，EF Core 要求在紧随运算符的步骤中将 GroupJoin 运算符的分组结果平展。 即使使用了 GroupJoin-DefaultIfEmpty-SelectMany，但采用其他的模式，也不能将其标识为 Left Join。
