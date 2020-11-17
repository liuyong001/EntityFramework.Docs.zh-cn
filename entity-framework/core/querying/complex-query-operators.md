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
# <a name="complex-query-operators"></a><span data-ttu-id="f8af4-103">复杂查询运算符</span><span class="sxs-lookup"><span data-stu-id="f8af4-103">Complex Query Operators</span></span>

<span data-ttu-id="f8af4-104">语言集成查询 (LINQ) 包含许多用于组合多个数据源或执行复杂处理的复杂运算符。</span><span class="sxs-lookup"><span data-stu-id="f8af4-104">Language Integrated Query (LINQ) contains many complex operators, which combine multiple data sources or does complex processing.</span></span> <span data-ttu-id="f8af4-105">并非所有 LINQ 运算符都会在服务器端进行适当转换。</span><span class="sxs-lookup"><span data-stu-id="f8af4-105">Not all LINQ operators have suitable translations on the server side.</span></span> <span data-ttu-id="f8af4-106">有时，采用一种形式的查询会转换为服务器，但如果采用另一种形式，即使结果相同，也不会转换。</span><span class="sxs-lookup"><span data-stu-id="f8af4-106">Sometimes, a query in one form translates to the server but if written in a different form doesn't translate even if the result is the same.</span></span> <span data-ttu-id="f8af4-107">本页介绍部分复杂运算符及其支持的变体。</span><span class="sxs-lookup"><span data-stu-id="f8af4-107">This page describes some of the complex operators and their supported variations.</span></span> <span data-ttu-id="f8af4-108">在将来的版本中，我们可能会认识更多的模式并添加其相应的转换。</span><span class="sxs-lookup"><span data-stu-id="f8af4-108">In future releases, we may recognize more patterns and add their corresponding translations.</span></span> <span data-ttu-id="f8af4-109">另请牢记，转换支持在提供程序之间各所不同。</span><span class="sxs-lookup"><span data-stu-id="f8af4-109">It's also important to keep in mind that translation support varies between providers.</span></span> <span data-ttu-id="f8af4-110">在 SqlServer 中转换的特定查询可能无法用于 SQLite 数据库。</span><span class="sxs-lookup"><span data-stu-id="f8af4-110">A particular query, which is translated in SqlServer, may not work for SQLite databases.</span></span>

> [!TIP]
> <span data-ttu-id="f8af4-111">可在 GitHub 上查看此文章的[示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Querying/ComplexQuery)。</span><span class="sxs-lookup"><span data-stu-id="f8af4-111">You can view this article's [sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Querying/ComplexQuery) on GitHub.</span></span>

## <a name="join"></a><span data-ttu-id="f8af4-112">联接</span><span class="sxs-lookup"><span data-stu-id="f8af4-112">Join</span></span>

<span data-ttu-id="f8af4-113">借助 LINQ Join 运算符，可根据每个源的键选择器连接两个数据源，并在键匹配时生成值的元组。</span><span class="sxs-lookup"><span data-stu-id="f8af4-113">The LINQ Join operator allows you to connect two data sources based on the key selector for each source, generating a tuple of values when the key matches.</span></span> <span data-ttu-id="f8af4-114">该运算符在关系数据库中自然而然地转换为 `INNER JOIN`。</span><span class="sxs-lookup"><span data-stu-id="f8af4-114">It naturally translates to `INNER JOIN` on relational databases.</span></span> <span data-ttu-id="f8af4-115">虽然 LINQ Join 具有外部和内部键选择器，但数据库只需要一个联接条件。</span><span class="sxs-lookup"><span data-stu-id="f8af4-115">While the LINQ Join has outer and inner key selectors, the database requires a single join condition.</span></span> <span data-ttu-id="f8af4-116">因此 EF Core 通过比较外部键选择器和内部键选择器是否相等，来生成联接条件。</span><span class="sxs-lookup"><span data-stu-id="f8af4-116">So EF Core generates a join condition by comparing the outer key selector to the inner key selector for equality.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#Join)]

```sql
SELECT [p].[PersonId], [p].[Name], [p].[PhotoId], [p0].[PersonPhotoId], [p0].[Caption], [p0].[Photo]
FROM [PersonPhoto] AS [p0]
INNER JOIN [Person] AS [p] ON [p0].[PersonPhotoId] = [p].[PhotoId]
```

<span data-ttu-id="f8af4-117">此外，如果键选择器是匿名类型，则 EF Core 会生成一个联接条件，以比较组件是否相等。</span><span class="sxs-lookup"><span data-stu-id="f8af4-117">Further, if the key selectors are anonymous types, EF Core generates a join condition to compare equality component-wise.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#JoinComposite)]

```sql
SELECT [p].[PersonId], [p].[Name], [p].[PhotoId], [p0].[PersonPhotoId], [p0].[Caption], [p0].[Photo]
FROM [PersonPhoto] AS [p0]
INNER JOIN [Person] AS [p] ON ([p0].[PersonPhotoId] = [p].[PhotoId] AND ([p0].[Caption] = N'SN'))
```

## <a name="groupjoin"></a><span data-ttu-id="f8af4-118">GroupJoin</span><span class="sxs-lookup"><span data-stu-id="f8af4-118">GroupJoin</span></span>

<span data-ttu-id="f8af4-119">借助 LINQ GroupJoin 运算符，可以采用与 Join 类似的方式连接两个数据源，但它会创建一组内部值，用于匹配外部元素。</span><span class="sxs-lookup"><span data-stu-id="f8af4-119">The LINQ GroupJoin operator allows you to connect two data sources similar to Join, but it creates a group of inner values for matching outer elements.</span></span> <span data-ttu-id="f8af4-120">执行与以下示例类似的查询将生成 `Blog` & `IEnumerable<Post>` 的结果。</span><span class="sxs-lookup"><span data-stu-id="f8af4-120">Executing a query like the following example generates a result of `Blog` & `IEnumerable<Post>`.</span></span> <span data-ttu-id="f8af4-121">由于数据库（特别是关系数据库）无法表示一组客户端对象，因此在许多情况下，GroupJoin 不会转换为服务器。</span><span class="sxs-lookup"><span data-stu-id="f8af4-121">Since databases (especially relational databases) don't have a way to represent a collection of client-side objects, GroupJoin doesn't translate to the server in many cases.</span></span> <span data-ttu-id="f8af4-122">它需要从服务器获取所有数据来进行 GroupJoin，无需使用特殊选择器（下面的第一个查询）。</span><span class="sxs-lookup"><span data-stu-id="f8af4-122">It requires you to get all of the data from the server to do GroupJoin without a special selector (first query below).</span></span> <span data-ttu-id="f8af4-123">但如果选择器限制选定的数据，则从服务器提取所有数据可能会导致出现性能问题（下面的第二个查询）。</span><span class="sxs-lookup"><span data-stu-id="f8af4-123">But if the selector is limiting data being selected then fetching all of the data from the server may cause performance issues (second query below).</span></span> <span data-ttu-id="f8af4-124">这就是 EF Core 不转换 GroupJoin 的原因。</span><span class="sxs-lookup"><span data-stu-id="f8af4-124">That's why EF Core doesn't translate GroupJoin.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#GroupJoin)]

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#GroupJoinComposed)]

## <a name="selectmany"></a><span data-ttu-id="f8af4-125">SelectMany</span><span class="sxs-lookup"><span data-stu-id="f8af4-125">SelectMany</span></span>

<span data-ttu-id="f8af4-126">借助 LINQ SelectMany 运算符，可为每个外部元素枚举集合选择器，并从每个数据源生成值的元组。</span><span class="sxs-lookup"><span data-stu-id="f8af4-126">The LINQ SelectMany operator allows you to enumerate over a collection selector for each outer element and generate tuples of values from each data source.</span></span> <span data-ttu-id="f8af4-127">在某种程度上，它是没有任何条件的联接，因此每个外部元素都与来自集合源的元素连接。</span><span class="sxs-lookup"><span data-stu-id="f8af4-127">In a way, it's a join but without any condition so every outer element is connected with an element from the collection source.</span></span> <span data-ttu-id="f8af4-128">根据集合选择器与外部数据源的关系，SelectMany 可在服务器端转换为各种不同的查询。</span><span class="sxs-lookup"><span data-stu-id="f8af4-128">Depending on how the collection selector is related to the outer data source, SelectMany can translate into various different queries on the server side.</span></span>

### <a name="collection-selector-doesnt-reference-outer"></a><span data-ttu-id="f8af4-129">集合选择器不引用外部</span><span class="sxs-lookup"><span data-stu-id="f8af4-129">Collection selector doesn't reference outer</span></span>

<span data-ttu-id="f8af4-130">如果集合选择器不引用外部源中的任何内容，则结果是这两个数据源的笛卡尔乘积。</span><span class="sxs-lookup"><span data-stu-id="f8af4-130">When the collection selector isn't referencing anything from the outer source, the result is a cartesian product of both data sources.</span></span> <span data-ttu-id="f8af4-131">它在关系数据库中转换为 `CROSS JOIN`。</span><span class="sxs-lookup"><span data-stu-id="f8af4-131">It translates to `CROSS JOIN` in relational databases.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#SelectManyConvertedToCrossJoin)]

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [Blogs] AS [b]
CROSS JOIN [Posts] AS [p]
```

### <a name="collection-selector-references-outer-in-a-where-clause"></a><span data-ttu-id="f8af4-132">集合选择器引用 where 子句中的外部</span><span class="sxs-lookup"><span data-stu-id="f8af4-132">Collection selector references outer in a where clause</span></span>

<span data-ttu-id="f8af4-133">如果集合选择器具有引用外部元素的 where 子句，则 EF Core 会将其转换为数据库联接，并将谓词用作联接条件。</span><span class="sxs-lookup"><span data-stu-id="f8af4-133">When the collection selector has a where clause, which references the outer element, then EF Core translates it to a database join and uses the predicate as the join condition.</span></span> <span data-ttu-id="f8af4-134">在对外部元素使用集合导航作为集合选择器时，通常会出现这种情况。</span><span class="sxs-lookup"><span data-stu-id="f8af4-134">Normally this case arises when using collection navigation on the outer element as the collection selector.</span></span> <span data-ttu-id="f8af4-135">如果外部元素的集合为空，则不会为该外部元素生成任何结果。</span><span class="sxs-lookup"><span data-stu-id="f8af4-135">If the collection is empty for an outer element, then no results would be generated for that outer element.</span></span> <span data-ttu-id="f8af4-136">但如果在集合选择器上应用 `DefaultIfEmpty`，则外部元素将与内部元素的默认值连接。</span><span class="sxs-lookup"><span data-stu-id="f8af4-136">But if `DefaultIfEmpty` is applied on the collection selector then the outer element will be connected with a default value of the inner element.</span></span> <span data-ttu-id="f8af4-137">由于这种区别，应用 `DefaultIfEmpty` 时，如果缺少 `DefaultIfEmpty` 和 `LEFT JOIN`，此类查询会转换为 `INNER JOIN`。</span><span class="sxs-lookup"><span data-stu-id="f8af4-137">Because of this distinction, this kind of queries translates to `INNER JOIN` in the absence of `DefaultIfEmpty` and `LEFT JOIN` when `DefaultIfEmpty` is applied.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#SelectManyConvertedToJoin)]

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [Blogs] AS [b]
INNER JOIN [Posts] AS [p] ON [b].[BlogId] = [p].[BlogId]

SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [Blogs] AS [b]
LEFT JOIN [Posts] AS [p] ON [b].[BlogId] = [p].[BlogId]
```

### <a name="collection-selector-references-outer-in-a-non-where-case"></a><span data-ttu-id="f8af4-138">集合选择器引用非 where 情况下的外部</span><span class="sxs-lookup"><span data-stu-id="f8af4-138">Collection selector references outer in a non-where case</span></span>

<span data-ttu-id="f8af4-139">如果集合选择器引用的外部元素不在 where 子句中（如上所述），则它不会转换为数据库联接。</span><span class="sxs-lookup"><span data-stu-id="f8af4-139">When the collection selector references the outer element, which isn't in a where clause (as the case above), it doesn't translate to a database join.</span></span> <span data-ttu-id="f8af4-140">这就是我们需要为每个外部元素评估集合选择器的原因。</span><span class="sxs-lookup"><span data-stu-id="f8af4-140">That's why we need to evaluate the collection selector for each outer element.</span></span> <span data-ttu-id="f8af4-141">它在许多关系数据库中转换为 `APPLY` 操作。</span><span class="sxs-lookup"><span data-stu-id="f8af4-141">It translates to `APPLY` operations in many relational databases.</span></span> <span data-ttu-id="f8af4-142">如果外部元素的集合为空，则不会为该外部元素生成任何结果。</span><span class="sxs-lookup"><span data-stu-id="f8af4-142">If the collection is empty for an outer element, then no results would be generated for that outer element.</span></span> <span data-ttu-id="f8af4-143">但如果在集合选择器上应用 `DefaultIfEmpty`，则外部元素将与内部元素的默认值连接。</span><span class="sxs-lookup"><span data-stu-id="f8af4-143">But if `DefaultIfEmpty` is applied on the collection selector then the outer element will be connected with a default value of the inner element.</span></span> <span data-ttu-id="f8af4-144">由于这种区别，应用 `DefaultIfEmpty` 时，如果缺少 `DefaultIfEmpty` 和 `OUTER APPLY`，此类查询会转换为 `CROSS APPLY`。</span><span class="sxs-lookup"><span data-stu-id="f8af4-144">Because of this distinction, this kind of queries translates to `CROSS APPLY` in the absence of `DefaultIfEmpty` and `OUTER APPLY` when `DefaultIfEmpty` is applied.</span></span> <span data-ttu-id="f8af4-145">像 SQLite 之类的某些数据库不支持 `APPLY` 运算符，因此此类查询可能不会进行转换。</span><span class="sxs-lookup"><span data-stu-id="f8af4-145">Certain databases like SQLite don't support `APPLY` operators so this kind of query may not be translated.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#SelectManyConvertedToApply)]

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], ([b].[Url] + N'=>') + [p].[Title] AS [p]
FROM [Blogs] AS [b]
CROSS APPLY [Posts] AS [p]

SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], ([b].[Url] + N'=>') + [p].[Title] AS [p]
FROM [Blogs] AS [b]
OUTER APPLY [Posts] AS [p]
```

## <a name="groupby"></a><span data-ttu-id="f8af4-146">GroupBy</span><span class="sxs-lookup"><span data-stu-id="f8af4-146">GroupBy</span></span>

<span data-ttu-id="f8af4-147">LINQ GroupBy 运算符创建 `IGrouping<TKey, TElement>` 类型的结果，其中 `TKey` 和 `TElement` 可以是任意类型。</span><span class="sxs-lookup"><span data-stu-id="f8af4-147">LINQ GroupBy operators create a result of type `IGrouping<TKey, TElement>` where `TKey` and `TElement` could be any arbitrary type.</span></span> <span data-ttu-id="f8af4-148">此外，`IGrouping` 实现了 `IEnumerable<TElement>`，这意味着可在分组后使用任意 LINQ 运算符来对其进行组合。</span><span class="sxs-lookup"><span data-stu-id="f8af4-148">Furthermore, `IGrouping` implements `IEnumerable<TElement>`, which means you can compose over it using any LINQ operator after the grouping.</span></span> <span data-ttu-id="f8af4-149">由于任何数据库结构都无法表示 `IGrouping`，因此在大多数情况下 GroupBy 运算符不会进行任何转换。</span><span class="sxs-lookup"><span data-stu-id="f8af4-149">Since no database structure can represent an `IGrouping`, GroupBy operators have no translation in most cases.</span></span> <span data-ttu-id="f8af4-150">聚合运算符应用于返回标量的每个组时，该运算符可在关系数据库中转换为 SQL `GROUP BY`。</span><span class="sxs-lookup"><span data-stu-id="f8af4-150">When an aggregate operator is applied to each group, which returns a scalar, it can be translated to SQL `GROUP BY` in relational databases.</span></span> <span data-ttu-id="f8af4-151">SQL `GROUP BY` 也会受到限制。</span><span class="sxs-lookup"><span data-stu-id="f8af4-151">The SQL `GROUP BY` is restrictive too.</span></span> <span data-ttu-id="f8af4-152">它要求只按标量值进行分组。</span><span class="sxs-lookup"><span data-stu-id="f8af4-152">It requires you to group only by scalar values.</span></span> <span data-ttu-id="f8af4-153">投影只能包含分组键列或对列应用的任何聚合。</span><span class="sxs-lookup"><span data-stu-id="f8af4-153">The projection can only contain grouping key columns or any aggregate applied over a column.</span></span> <span data-ttu-id="f8af4-154">EF Core 标识此模式并将其转换为服务器，如以下示例中所示：</span><span class="sxs-lookup"><span data-stu-id="f8af4-154">EF Core identifies this pattern and translates it to the server, as in the following example:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#GroupBy)]

```sql
SELECT [p].[AuthorId] AS [Key], COUNT(*) AS [Count]
FROM [Posts] AS [p]
GROUP BY [p].[AuthorId]
```

<span data-ttu-id="f8af4-155">EF Core 还会转换符合以下条件的查询：分组的聚合运算符出现在 Where 或 OrderBy（或其他排序方式）LINQ 运算符中。</span><span class="sxs-lookup"><span data-stu-id="f8af4-155">EF Core also translates queries where an aggregate operator on the grouping appears in a Where or OrderBy (or other ordering) LINQ operator.</span></span> <span data-ttu-id="f8af4-156">它在 SQL 中将 `HAVING` 子句用于 where 子句。</span><span class="sxs-lookup"><span data-stu-id="f8af4-156">It uses `HAVING` clause in SQL for the where clause.</span></span> <span data-ttu-id="f8af4-157">在应用 GroupBy 运算符之前的查询部分可以是任何复杂查询，只要它可转换为服务器即可。</span><span class="sxs-lookup"><span data-stu-id="f8af4-157">The part of the query before applying the GroupBy operator can be any complex query as long as it can be translated to server.</span></span> <span data-ttu-id="f8af4-158">此外，将聚合运算符应用于分组查询以从生成的源中移除分组后，可以像使用任何其他查询一样，在它的基础上进行组合。</span><span class="sxs-lookup"><span data-stu-id="f8af4-158">Furthermore, once you apply aggregate operators on a grouping query to remove groupings from the resulting source, you can compose on top of it like any other query.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#GroupByFilter)]

```sql
SELECT [p].[AuthorId] AS [Key], COUNT(*) AS [Count]
FROM [Posts] AS [p]
GROUP BY [p].[AuthorId]
HAVING COUNT(*) > 0
ORDER BY [p].[AuthorId]
```

<span data-ttu-id="f8af4-159">EF Core 支持的聚合运算符如下所示</span><span class="sxs-lookup"><span data-stu-id="f8af4-159">The aggregate operators EF Core supports are as follows</span></span>

- <span data-ttu-id="f8af4-160">平均值</span><span class="sxs-lookup"><span data-stu-id="f8af4-160">Average</span></span>
- <span data-ttu-id="f8af4-161">计数</span><span class="sxs-lookup"><span data-stu-id="f8af4-161">Count</span></span>
- <span data-ttu-id="f8af4-162">LongCount</span><span class="sxs-lookup"><span data-stu-id="f8af4-162">LongCount</span></span>
- <span data-ttu-id="f8af4-163">最大值</span><span class="sxs-lookup"><span data-stu-id="f8af4-163">Max</span></span>
- <span data-ttu-id="f8af4-164">Min</span><span class="sxs-lookup"><span data-stu-id="f8af4-164">Min</span></span>
- <span data-ttu-id="f8af4-165">Sum</span><span class="sxs-lookup"><span data-stu-id="f8af4-165">Sum</span></span>

## <a name="left-join"></a><span data-ttu-id="f8af4-166">Left Join</span><span class="sxs-lookup"><span data-stu-id="f8af4-166">Left Join</span></span>

<span data-ttu-id="f8af4-167">虽然 Left Join 不是 LINQ 运算符，但关系数据库具有常用于查询的 Left Join 的概念。</span><span class="sxs-lookup"><span data-stu-id="f8af4-167">While Left Join isn't a LINQ operator, relational databases have the concept of a Left Join which is frequently used in queries.</span></span> <span data-ttu-id="f8af4-168">LINQ 查询中的特定模式提供与服务器上的 `LEFT JOIN` 相同的结果。</span><span class="sxs-lookup"><span data-stu-id="f8af4-168">A particular pattern in LINQ queries gives the same result as a `LEFT JOIN` on the server.</span></span> <span data-ttu-id="f8af4-169">EF Core 标识此类模式，并在服务器端生成等效的 `LEFT JOIN`。</span><span class="sxs-lookup"><span data-stu-id="f8af4-169">EF Core identifies such patterns and generates the equivalent `LEFT JOIN` on the server side.</span></span> <span data-ttu-id="f8af4-170">该模式包括在两个数据源之间创建 GroupJoin，然后通过对分组源使用 SelectMany 运算符与 DefaultIfEmpty 来平展分组，从而在内部不具有相关元素时匹配 null。</span><span class="sxs-lookup"><span data-stu-id="f8af4-170">The pattern involves creating a GroupJoin between both the data sources and then flattening out the grouping by using the SelectMany operator with DefaultIfEmpty on the grouping source to match null when the inner doesn't have a related element.</span></span> <span data-ttu-id="f8af4-171">下面的示例显示该模式的样式及其生成的内容。</span><span class="sxs-lookup"><span data-stu-id="f8af4-171">The following example shows what that pattern looks like and what it generates.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/ComplexQuery/Program.cs#LeftJoin)]

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url], [p].[PostId], [p].[AuthorId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [Blogs] AS [b]
LEFT JOIN [Posts] AS [p] ON [b].[BlogId] = [p].[BlogId]
```

<span data-ttu-id="f8af4-172">以上模式在表达式树中创建复杂的结构。</span><span class="sxs-lookup"><span data-stu-id="f8af4-172">The above pattern creates a complex structure in the expression tree.</span></span> <span data-ttu-id="f8af4-173">因此，EF Core 要求在紧随运算符的步骤中将 GroupJoin 运算符的分组结果平展。</span><span class="sxs-lookup"><span data-stu-id="f8af4-173">Because of that, EF Core requires you to flatten out the grouping results of the GroupJoin operator in a step immediately following the operator.</span></span> <span data-ttu-id="f8af4-174">即使使用了 GroupJoin-DefaultIfEmpty-SelectMany，但采用其他的模式，也不能将其标识为 Left Join。</span><span class="sxs-lookup"><span data-stu-id="f8af4-174">Even if the GroupJoin-DefaultIfEmpty-SelectMany is used but in a different pattern, we may not identify it as a Left Join.</span></span>
