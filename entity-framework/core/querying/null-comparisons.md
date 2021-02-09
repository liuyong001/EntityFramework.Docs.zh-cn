---
title: 比较查询中的 null 值
description: 介绍 Entity Framework Core 如何处理查询中的 null 比较
author: maumar
ms.date: 11/11/2020
uid: core/querying/null-comparisons
ms.openlocfilehash: fc63d0e0e6aea09e46b1700152312d4b74270219
ms.sourcegitcommit: 7700840119b1639275f3b64836e7abb59103f2e7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2021
ms.locfileid: "98983347"
---
# <a name="query-null-semantics"></a>查询 null 语义

## <a name="introduction"></a>简介

SQL 数据库在执行比较时基于三值逻辑（`true`、`false`、`null`）进行操作，而不是基于 C# 的布尔逻辑。 将 LINQ 查询转换为 SQL 时，EF Core 会尝试通过为某些查询元素引入额外的 null 检查来补偿差异。
为了说明这一点，我们定义以下实体：

[!code-csharp[Main](../../../samples/core/Querying/NullSemantics/NullSemanticsEntity.cs#Entity)]

并发出几个查询：

[!code-csharp[Main](../../../samples/core/Querying/NullSemantics/Program.cs#BasicExamples)]

前两个查询生成简单的比较。 在第一个查询中，这两个列都不可为 null，因此不需要进行 null 检查。 在第二个查询中，`NullableInt` 可能包含 `null`，但 `Id` 不可为 null；将 `null` 与非 null 进行比较会生成 `null`，将通过 `WHERE` 操作对其进行筛除。 因此，无需任何其他条款。

```sql
SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE [e].[Id] = [e].[Int]

SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE [e].[Id] = [e].[NullableInt]
```

第三个查询引入了 null 检查。 如果 `NullableInt` 为 `null`，则比较 `Id <> NullableInt` 会生成 `null`，将通过 `WHERE` 操作对其进行筛除。 但是，从布尔逻辑的角度来看，此案例应作为结果的一部分返回。 因此 EF Core 添加了必要的检查来确保这一点。

```sql
SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE ([e].[Id] <> [e].[NullableInt]) OR [e].[NullableInt] IS NULL
```

当两个列都可为 null 时，查询 4 和 5 会显示该模式。 值得注意的是，与 `==` 操作相比，`<>` 操作产生的查询更为复杂（并且可能更慢）。

```sql
SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE ([e].[String1] = [e].[String2]) OR ([e].[String1] IS NULL AND [e].[String2] IS NULL)

SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE (([e].[String1] <> [e].[String2]) OR ([e].[String1] IS NULL OR [e].[String2] IS NULL)) AND ([e].[String1] IS NOT NULL OR [e].[String2] IS NOT NULL)
```

## <a name="treatment-of-nullable-values-in-functions"></a>函数中可为 null 的值的处理

对于 SQL 中的许多函数，若其某些参数为 `null`，则它们只能返回 `null` 结果。 EF Core 利用这一点来生成更高效的查询。
下面的查询说明了优化：

[!code-csharp[Main](../../../samples/core/Querying/NullSemantics/Program.cs#Functions)]

生成的 SQL 如下所示（我们不需要计算 `SUBSTRING` 函数，因为只有当它的任一参数为 null 时，它才将为 null。）：

```sql
SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE [e].[String1] IS NULL OR [e].[String2] IS NULL
```

优化还可用于用户定义的函数。 有关更多详细信息，请参阅[用户定义的函数映射](xref:core/querying/user-defined-function-mapping#configuring-nullability-of-user-defined-function-based-on-its-arguments)页。

## <a name="writing-performant-queries"></a>编写高性能查询

- 比较不可为 null 的列比比较可为 null 的列更简单且更快。 如果可能，请考虑将列标记为不可为 null。

- 检查相等性 (`==`) 比检查不相等 (`!=`) 更简单且更快，因为查询无需区分 `null` 和 `false` 结果。 尽可能使用相等性比较。 不过，只是否定 `==` 比较这一点实际上与 `!=` 等效，因此不会提高性能。

- 在某些情况下，可通过从列中显式筛选出 `null` 值来简化复杂比较，例如，当不存在 `null` 值或这些值在结果中不相关时。 请看下面的示例：

[!code-csharp[Main](../../../samples/core/Querying/NullSemantics/Program.cs#ManualOptimization)]

这些查询生成以下 SQL：

```sql
SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE ((([e].[String1] <> [e].[String2]) OR ([e].[String1] IS NULL OR [e].[String2] IS NULL)) AND ([e].[String1] IS NOT NULL OR [e].[String2] IS NOT NULL)) OR ((CAST(LEN([e].[String1]) AS int) = CAST(LEN([e].[String2]) AS int)) OR ([e].[String1] IS NULL AND [e].[String2] IS NULL))

SELECT [e].[Id], [e].[Int], [e].[NullableInt], [e].[String1], [e].[String2]
FROM [Entities] AS [e]
WHERE ([e].[String1] IS NOT NULL AND [e].[String2] IS NOT NULL) AND (([e].[String1] <> [e].[String2]) OR (CAST(LEN([e].[String1]) AS int) = CAST(LEN([e].[String2]) AS int)))
```

在第二个查询中，`null` 结果从 `String1` 列中显式筛选出来。 在比较过程中，EF Core 可安全地将 `String1` 列视为不可为 null 的列，从而生成更简单的查询。

## <a name="using-relational-null-semantics"></a>使用关系 null 语义

可禁用 null 比较补偿并直接使用关系 null 语义。 这可通过在 `OnConfiguring` 方法中的选项生成器上调用 `UseRelationalNulls(true)` 方法来实现：

[!code-csharp[Main](../../../samples/core/Querying/NullSemantics/NullSemanticsContext.cs#UseRelationalNulls)]

> [!WARNING]
> 使用关系 null 语义时，LINQ 查询的含义不再与 C# 中的含义相同，并且可能会产生与预期不同的结果。 使用此模式时应多加小心。
