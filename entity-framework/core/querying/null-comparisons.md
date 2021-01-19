---
title: 比较查询中的 null 值
description: 介绍 Entity Framework Core 如何处理查询中的 null 比较
author: maumar
ms.date: 11/11/2020
uid: core/querying/null-comparisons
ms.openlocfilehash: d1235eb8df7fd22c7a930b3661ec38a99f75e5fa
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129531"
---
# <a name="query-null-semantics"></a><span data-ttu-id="ec4b2-103">查询 null 语义</span><span class="sxs-lookup"><span data-stu-id="ec4b2-103">Query null semantics</span></span>

## <a name="introduction"></a><span data-ttu-id="ec4b2-104">简介</span><span class="sxs-lookup"><span data-stu-id="ec4b2-104">Introduction</span></span>

<span data-ttu-id="ec4b2-105">SQL 数据库在执行比较时基于三值逻辑（`true`、`false`、`null`）进行操作，而不是基于 C# 的布尔逻辑。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-105">SQL databases operate on 3-valued logic (`true`, `false`, `null`) when performing comparisons, as opposed to the boolean logic of C#.</span></span> <span data-ttu-id="ec4b2-106">将 LINQ 查询转换为 SQL 时，EF Core 会尝试通过为某些查询元素引入额外的 null 检查来补偿差异。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-106">When translating LINQ queries to SQL, EF Core tries to compensate for the difference by introducing additional null checks for some elements of the query.</span></span>
<span data-ttu-id="ec4b2-107">为了说明这一点，我们定义以下实体：</span><span class="sxs-lookup"><span data-stu-id="ec4b2-107">To illustrate this, let's define the following entity:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/NullSemantics/NullSemanticsEntity.cs#Entity)]

<span data-ttu-id="ec4b2-108">并发出几个查询：</span><span class="sxs-lookup"><span data-stu-id="ec4b2-108">and issue several queries:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/NullSemantics/Program.cs#BasicExamples)]

<span data-ttu-id="ec4b2-109">前两个查询生成简单的比较。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-109">The first two queries produce simple comparisons.</span></span> <span data-ttu-id="ec4b2-110">在第一个查询中，这两个列都不可为 null，因此不需要进行 null 检查。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-110">In the first query, both columns are non-nullable so null checks are not needed.</span></span> <span data-ttu-id="ec4b2-111">在第二个查询中，`NullableInt` 可能包含 `null`，但 `Id` 不可为 null；将 `null` 与非 null 进行比较会生成 `null`，将通过 `WHERE` 操作对其进行筛除。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-111">In the second query, `NullableInt` could contain `null`, but `Id` is non-nullable; comparing `null` to non-null yields `null` as a result, which would be filtered out by `WHERE` operation.</span></span> <span data-ttu-id="ec4b2-112">因此，无需任何其他条款。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-112">So no additional terms are needed either.</span></span>

```sql
SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE [e].[Id] = [e].[Int]

SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE [e].[Id] = [e].[NullableInt]
```

<span data-ttu-id="ec4b2-113">第三个查询引入了 null 检查。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-113">The third query introduces a null check.</span></span> <span data-ttu-id="ec4b2-114">如果 `NullableInt` 为 `null`，则比较 `Id <> NullableInt` 会生成 `null`，将通过 `WHERE` 操作对其进行筛除。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-114">When `NullableInt` is `null` the comparison `Id <> NullableInt` yields `null`, which would be filtered out by `WHERE` operation.</span></span> <span data-ttu-id="ec4b2-115">但是，从布尔逻辑的角度来看，此案例应作为结果的一部分返回。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-115">However, from the boolean logic perspective this case should be returned as part of the result.</span></span> <span data-ttu-id="ec4b2-116">因此 EF Core 添加了必要的检查来确保这一点。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-116">Hence EF Core adds the necessary check to ensure that.</span></span>

```sql
SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE ([e].[Id] <> [e].[NullableInt]) OR [e].[NullableInt] IS NULL
```

<span data-ttu-id="ec4b2-117">当两个列都可为 null 时，查询 4 和 5 会显示该模式。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-117">Queries four and five show the pattern when both columns are nullable.</span></span> <span data-ttu-id="ec4b2-118">值得注意的是，与 `==` 操作相比，`<>` 操作产生的查询更为复杂（并且可能更慢）。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-118">It's worth noting that the `<>` operation produces more complicated (and potentially slower) query than the `==` operation.</span></span>

```sql
SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE ([e].[String1] = [e].[String2]) OR ([e].[String1] IS NULL AND [e].[String2] IS NULL)

SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE (([e].[String1] <> [e].[String2]) OR ([e].[String1] IS NULL OR [e].[String2] IS NULL)) AND ([e].[String1] IS NOT NULL OR [e].[String2] IS NOT NULL)
```

## <a name="treatment-of-nullable-values-in-functions"></a><span data-ttu-id="ec4b2-119">函数中可为 null 的值的处理</span><span class="sxs-lookup"><span data-stu-id="ec4b2-119">Treatment of nullable values in functions</span></span>

<span data-ttu-id="ec4b2-120">对于 SQL 中的许多函数，若其某些参数为 `null`，则它们只能返回 `null` 结果。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-120">Many functions in SQL can only return a `null` result if some of their arguments are `null`.</span></span> <span data-ttu-id="ec4b2-121">EF Core 利用这一点来生成更高效的查询。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-121">EF Core takes advantage of this to produce more efficient queries.</span></span>
<span data-ttu-id="ec4b2-122">下面的查询说明了优化：</span><span class="sxs-lookup"><span data-stu-id="ec4b2-122">The query below illustrates the optimization:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/NullSemantics/Program.cs#Functions)]

<span data-ttu-id="ec4b2-123">生成的 SQL 如下所示（我们不需要计算 `SUBSTRING` 函数，因为只有当它的任一参数为 null 时，它才将为 null。）：</span><span class="sxs-lookup"><span data-stu-id="ec4b2-123">The generated SQL is as follows (we don't need to evaluate the `SUBSTRING` function since it will be only null when either of the arguments to it is null.):</span></span>

```sql
SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE [e].[String1] IS NULL OR [e].[String2] IS NULL
```

<span data-ttu-id="ec4b2-124">优化还可用于用户定义的函数。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-124">The optimization can also be used for user-defined functions.</span></span> <span data-ttu-id="ec4b2-125">有关更多详细信息，请参阅[用户定义的函数映射](xref:core/querying/user-defined-function-mapping#configuring-nullability-of-user-defined-function-based-on-its-arguments)页。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-125">See [user defined function mapping](xref:core/querying/user-defined-function-mapping#configuring-nullability-of-user-defined-function-based-on-its-arguments) page for more details.</span></span>

## <a name="writing-performant-queries"></a><span data-ttu-id="ec4b2-126">编写高性能查询</span><span class="sxs-lookup"><span data-stu-id="ec4b2-126">Writing performant queries</span></span>

- <span data-ttu-id="ec4b2-127">比较不可为 null 的列比比较可为 null 的列更简单且更快。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-127">Comparing non-nullable columns is simpler and faster than comparing nullable columns.</span></span> <span data-ttu-id="ec4b2-128">如果可能，请考虑将列标记为不可为 null。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-128">Consider marking columns as non-nullable whenever possible.</span></span>

- <span data-ttu-id="ec4b2-129">检查相等性 (`==`) 比检查不相等 (`!=`) 更简单且更快，因为查询无需区分 `null` 和 `false` 结果。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-129">Checking for equality (`==`) is simpler and faster than checking for non-equality (`!=`), because query doesn't need to distinguish between `null` and `false` result.</span></span> <span data-ttu-id="ec4b2-130">尽可能使用相等性比较。不过，只是否定 `==` 比较这一点实际上与 `!=` 等效，因此不会提高性能。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-130">Use equality comparison whenever possible.However, simply negating `==` comparison is effectively the same as `!=`, so it doesn't result in performance improvement.</span></span>

- <span data-ttu-id="ec4b2-131">在某些情况下，可通过从列中显式筛选出 `null` 值来简化复杂比较，例如，当不存在 `null` 值或这些值在结果中不相关时。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-131">In some cases, it is possible to simplify a complex comparison by filtering out `null` values from a column explicitly - for example when no `null` values are present or these values are not relevant in the result.</span></span> <span data-ttu-id="ec4b2-132">请看下面的示例：</span><span class="sxs-lookup"><span data-stu-id="ec4b2-132">Consider the following example:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/NullSemantics/Program.cs#ManualOptimization)]

<span data-ttu-id="ec4b2-133">这些查询生成以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="ec4b2-133">These queries produce the following SQL:</span></span>

```sql
SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE ((([e].[String1] <> [e].[String2]) OR ([e].[String1] IS NULL OR [e].[String2] IS NULL)) AND ([e].[String1] IS NOT NULL OR [e].[String2] IS NOT NULL)) OR ((CAST(LEN([e].[String1]) AS int) = CAST(LEN([e].[String2]) AS int)) OR ([e].[String1] IS NULL AND [e].[String2] IS NULL))

SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE ([e].[String1] IS NOT NULL AND [e].[String2] IS NOT NULL) AND (([e].[String1] <> [e].[String2]) OR (CAST(LEN([e].[String1]) AS int) = CAST(LEN([e].[String2]) AS int)))
```

<span data-ttu-id="ec4b2-134">在第二个查询中，`null` 结果从 `String1` 列中显式筛选出来。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-134">In the second query, `null` results are filtered out from `String1` column explicitly.</span></span> <span data-ttu-id="ec4b2-135">在比较过程中，EF Core 可安全地将 `String1` 列视为不可为 null 的列，从而生成更简单的查询。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-135">EF Core can safely treat the `String1` column as non-nullable during comparison, resulting in a simpler query.</span></span>

## <a name="using-relational-null-semantics"></a><span data-ttu-id="ec4b2-136">使用关系 null 语义</span><span class="sxs-lookup"><span data-stu-id="ec4b2-136">Using relational null semantics</span></span>

<span data-ttu-id="ec4b2-137">可禁用 null 比较补偿并直接使用关系 null 语义。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-137">It's possible to disable the null comparison compensation and use relational null semantics directly.</span></span> <span data-ttu-id="ec4b2-138">这可通过在 `OnConfiguring` 方法中的选项生成器上调用 `UseRelationalNulls(true)` 方法来实现：</span><span class="sxs-lookup"><span data-stu-id="ec4b2-138">This can be done by calling `UseRelationalNulls(true)` method on the options builder inside `OnConfiguring` method:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/NullSemantics/NullSemanticsContext.cs#UseRelationalNulls)]

> [!WARNING]
> <span data-ttu-id="ec4b2-139">使用关系 null 语义时，LINQ 查询的含义不再与 C# 中的含义相同，并且可能会产生与预期不同的结果。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-139">When using relational null semantics, your LINQ queries no longer have the same meaning as they do in C#, and may yield different results than expected.</span></span> <span data-ttu-id="ec4b2-140">使用此模式时应多加小心。</span><span class="sxs-lookup"><span data-stu-id="ec4b2-140">Exercise caution when using this mode.</span></span>
