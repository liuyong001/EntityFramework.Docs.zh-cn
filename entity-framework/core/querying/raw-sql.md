---
title: 原生 SQL 查询 - EF Core
description: 在 Entity Framework Core 中使用原生 SQL 进行查询
author: smitpatel
ms.date: 10/08/2019
uid: core/querying/raw-sql
ms.openlocfilehash: 56724f9fddc9126fd4dfe0f348a0c525b43a3478
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128376"
---
# <a name="raw-sql-queries"></a>原生 SQL 查询

通过 Entity Framework Core 可以在使用关系数据库时下降到原始 SQL 查询。 所需查询不能使用 LINQ 来表示时，可以使用原始 SQL 查询。 如果使用 LINQ 查询导致 SQL 查询效率低下，也可以使用原始 SQL 查询。 原始 SQL 查询可返回一般实体类型或者模型中的[无键实体类型](xref:core/modeling/keyless-entity-types)。

> [!TIP]
> 可在 GitHub 上查看此文章的[示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Querying/RawSQL)。

## <a name="basic-raw-sql-queries"></a>基本原生 SQL 查询

可使用 `FromSqlRaw` 扩展方法基于原始 SQL 查询开始 LINQ 查询。 `FromSqlRaw` 只能在直接位于 `DbSet<>` 上的查询根上使用。

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlRaw)]

原生 SQL 查询可用于执行存储过程。

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlRawStoredProcedure)]

## <a name="passing-parameters"></a>快速参考

> [!WARNING]
> **始终对原始 SQL 查询使用参数化**
>
> 向原始 SQL 查询引入任何用户提供的值时，必须注意防范 SQL 注入攻击。 除了验证确保此类值不包含无效字符，请始终使用会将值与 SQL 文本分开发送的参数化处理。
>
> 具体而言，如果连接和内插的字符串 (`$""`) 带有用户提供的未经验证的值，则切勿将其传递到 `FromSqlRaw` 或 `ExecuteSqlRaw`。 通过 `FromSqlInterpolated` 和 `ExecuteSqlInterpolated` 方法，可采用一种能抵御 SQL 注入攻击的方式使用字符串内插语法。

下面的示例通过在 SQL 查询字符串中包含形参占位符并提供额外的实参，将单个形参传递到存储过程。 虽然此语法可能看上去像 `String.Format` 语法，但提供的值包装在 `DbParameter` 中，且生成的参数名称插入到指定 `{0}` 占位符的位置。

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlRawStoredProcedureParameter)]

虽然 `FromSqlInterpolated` 类似于 `FromSqlRaw`，但你可以借助它使用字符串内插语法。 与 `FromSqlRaw` 一样，`FromSqlInterpolated` 只能在查询根上使用。 与上述示例一样，该值会转换为 `DbParameter`，且不易受到 SQL 注入攻击。

> [!NOTE]
> 在版本 3.0 之前，`FromSqlRaw` 和 `FromSqlInterpolated` 是名为 `FromSql` 的两个重载。 有关详细信息，请参阅[历史版本部分](#previous-versions)。

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlInterpolatedStoredProcedureParameter)]

还可以构造 DbParameter 并将其作为参数值提供。 由于使用了常规 SQL 参数占位符而不是字符串占位符，因此可安全地使用 `FromSqlRaw`：

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlRawStoredProcedureSqlParameter)]

借助 `FromSqlRaw`，可以在 SQL 查询字符串中使用已命名的参数，这在存储的流程具有可选参数时非常有用：

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlRawStoredProcedureNamedSqlParameter)]

> [!NOTE]
> **参数排序** Entity Framework Core 根据 `SqlParameter[]` 数组的顺序传递参数。 传递多个 `SqlParameter` 时，SQL 字符串中的顺序必须与存储过程定义中参数的顺序相匹配。 如果不匹配，在执行此过程时，就可能会导致类型转换异常和/或异常行为。

## <a name="composing-with-linq"></a>使用 LINQ 编写

可使用 LINQ 运算符在初始的原始 SQL 查询基础上进行组合。 EF Core 将其视为子查询，并在数据库中对其进行组合。 下面的示例使用原始 SQL 查询，该查询从表值函数 (TVF) 中进行选择。 然后，使用 LINQ 进行筛选和排序，从而对其进行组合。

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlInterpolatedComposed)]

上面的查询生成以下 SQL：

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url]
FROM (
    SELECT * FROM dbo.SearchBlogs(@p0)
) AS [b]
WHERE [b].[Rating] > 3
ORDER BY [b].[Rating] DESC
```

### <a name="including-related-data"></a>包含关联数据

`Include` 方法可用于添加相关数据，就像对其他 LINQ 查询那样：

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlInterpolatedInclude)]

使用 LINQ 进行组合要求原始 SQL 查询是可组合的，因为 EF Core 会将提供的 SQL 视为子查询。 以 `SELECT` 关键字开始的 SQL 查询一般是可组合的。 此外，传递的 SQL 不应包含子查询上无效的任何字符或选项，如：

- 结尾分号
- 在 SQL Server 上，结尾处的查询级提示（例如，`OPTION (HASH JOIN)`）
- 在 SQL Server 上，`SELECT` 子句中不与 `OFFSET 0` 或 `TOP 100 PERCENT` 配合使用的 `ORDER BY` 子句

SQL Server 不允许对存储过程调用进行组合，因此任何尝试向此类调用应用其他查询运算符的操作都将导致无效的 SQL。 请在 `FromSqlRaw` 或 `FromSqlInterpolated` 方法之后立即使用 `AsEnumerable` 或 `AsAsyncEnumerable` 方法，确保 EF Core 不会尝试对存储过程进行组合。

## <a name="change-tracking"></a>更改跟踪

使用 `FromSqlRaw` 或 `FromSqlInterpolated` 方法的查询遵循与 EF Core 中所有其他 LINQ 查询完全相同的更改跟踪规则。 例如，如果该查询投影实体类型，默认情况下会跟踪结果。

下面的示例使用原始 SQL 查询，该查询从表值函数 (TVF) 中进行选择，然后禁用通过对 `AsNoTracking` 的调用来更改跟踪：

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlInterpolatedAsNoTracking)]

## <a name="limitations"></a>限制

使用原生 SQL 查询时需注意以下几个限制：

- SQL 查询必须返回实体类型的所有属性的数据。
- 结果集中的列名必须与属性映射到的列名称匹配。 请注意，此行为不同于 EF6。 EF6 中忽略了原始 SQL 查询的属性/列映射关系，结果集列名必须与属性名相匹配。
- SQL 查询不能包含关联数据。 但是，在许多情况下你可以在查询后面紧跟着使用 `Include` 方法以返回关联数据（请参阅[包含关联数据](#including-related-data)）。

## <a name="previous-versions"></a>旧版

EF Core 2.2 及更低版本具有两个名为 `FromSql` 的方法重载，它们的行为方式与较新的 `FromSqlRaw` 和 `FromSqlInterpolated` 的相同。 因此，在意图调用内插字符串方法时很容易意外调用原始字符串方法，反之亦然。 意外调用错误的重载会导致本该参数化的查询没有参数化。
