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
# <a name="raw-sql-queries"></a><span data-ttu-id="18745-103">原生 SQL 查询</span><span class="sxs-lookup"><span data-stu-id="18745-103">Raw SQL Queries</span></span>

<span data-ttu-id="18745-104">通过 Entity Framework Core 可以在使用关系数据库时下降到原始 SQL 查询。</span><span class="sxs-lookup"><span data-stu-id="18745-104">Entity Framework Core allows you to drop down to raw SQL queries when working with a relational database.</span></span> <span data-ttu-id="18745-105">所需查询不能使用 LINQ 来表示时，可以使用原始 SQL 查询。</span><span class="sxs-lookup"><span data-stu-id="18745-105">Raw SQL queries are useful if the query you want can't be expressed using LINQ.</span></span> <span data-ttu-id="18745-106">如果使用 LINQ 查询导致 SQL 查询效率低下，也可以使用原始 SQL 查询。</span><span class="sxs-lookup"><span data-stu-id="18745-106">Raw SQL queries are also used if using a LINQ query is resulting in an inefficient SQL query.</span></span> <span data-ttu-id="18745-107">原始 SQL 查询可返回一般实体类型或者模型中的[无键实体类型](xref:core/modeling/keyless-entity-types)。</span><span class="sxs-lookup"><span data-stu-id="18745-107">Raw SQL queries can return regular entity types or [keyless entity types](xref:core/modeling/keyless-entity-types) that are part of your model.</span></span>

> [!TIP]
> <span data-ttu-id="18745-108">可在 GitHub 上查看此文章的[示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Querying/RawSQL)。</span><span class="sxs-lookup"><span data-stu-id="18745-108">You can view this article's [sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Querying/RawSQL) on GitHub.</span></span>

## <a name="basic-raw-sql-queries"></a><span data-ttu-id="18745-109">基本原生 SQL 查询</span><span class="sxs-lookup"><span data-stu-id="18745-109">Basic raw SQL queries</span></span>

<span data-ttu-id="18745-110">可使用 `FromSqlRaw` 扩展方法基于原始 SQL 查询开始 LINQ 查询。</span><span class="sxs-lookup"><span data-stu-id="18745-110">You can use the `FromSqlRaw` extension method to begin a LINQ query based on a raw SQL query.</span></span> <span data-ttu-id="18745-111">`FromSqlRaw` 只能在直接位于 `DbSet<>` 上的查询根上使用。</span><span class="sxs-lookup"><span data-stu-id="18745-111">`FromSqlRaw` can only be used on query roots, that is directly on the `DbSet<>`.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlRaw)]

<span data-ttu-id="18745-112">原生 SQL 查询可用于执行存储过程。</span><span class="sxs-lookup"><span data-stu-id="18745-112">Raw SQL queries can be used to execute a stored procedure.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlRawStoredProcedure)]

## <a name="passing-parameters"></a><span data-ttu-id="18745-113">快速参考</span><span class="sxs-lookup"><span data-stu-id="18745-113">Passing parameters</span></span>

> [!WARNING]
> <span data-ttu-id="18745-114">**始终对原始 SQL 查询使用参数化**</span><span class="sxs-lookup"><span data-stu-id="18745-114">**Always use parameterization for raw SQL queries**</span></span>
>
> <span data-ttu-id="18745-115">向原始 SQL 查询引入任何用户提供的值时，必须注意防范 SQL 注入攻击。</span><span class="sxs-lookup"><span data-stu-id="18745-115">When introducing any user-provided values into a raw SQL query, care must be taken to avoid SQL injection attacks.</span></span> <span data-ttu-id="18745-116">除了验证确保此类值不包含无效字符，请始终使用会将值与 SQL 文本分开发送的参数化处理。</span><span class="sxs-lookup"><span data-stu-id="18745-116">In addition to validating that such values don't contain invalid characters, always use parameterization which sends the values separate from the SQL text.</span></span>
>
> <span data-ttu-id="18745-117">具体而言，如果连接和内插的字符串 (`$""`) 带有用户提供的未经验证的值，则切勿将其传递到 `FromSqlRaw` 或 `ExecuteSqlRaw`。</span><span class="sxs-lookup"><span data-stu-id="18745-117">In particular, never pass a concatenated or interpolated string (`$""`) with non-validated user-provided values into `FromSqlRaw` or `ExecuteSqlRaw`.</span></span> <span data-ttu-id="18745-118">通过 `FromSqlInterpolated` 和 `ExecuteSqlInterpolated` 方法，可采用一种能抵御 SQL 注入攻击的方式使用字符串内插语法。</span><span class="sxs-lookup"><span data-stu-id="18745-118">The `FromSqlInterpolated` and `ExecuteSqlInterpolated` methods allow using string interpolation syntax in a way that protects against SQL injection attacks.</span></span>

<span data-ttu-id="18745-119">下面的示例通过在 SQL 查询字符串中包含形参占位符并提供额外的实参，将单个形参传递到存储过程。</span><span class="sxs-lookup"><span data-stu-id="18745-119">The following example passes a single parameter to a stored procedure by including a parameter placeholder in the SQL query string and providing an additional argument.</span></span> <span data-ttu-id="18745-120">虽然此语法可能看上去像 `String.Format` 语法，但提供的值包装在 `DbParameter` 中，且生成的参数名称插入到指定 `{0}` 占位符的位置。</span><span class="sxs-lookup"><span data-stu-id="18745-120">While this syntax may look like `String.Format` syntax, the supplied value is wrapped in a `DbParameter` and the generated parameter name inserted where the `{0}` placeholder was specified.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlRawStoredProcedureParameter)]

<span data-ttu-id="18745-121">虽然 `FromSqlInterpolated` 类似于 `FromSqlRaw`，但你可以借助它使用字符串内插语法。</span><span class="sxs-lookup"><span data-stu-id="18745-121">`FromSqlInterpolated` is similar to `FromSqlRaw` but allows you to use string interpolation syntax.</span></span> <span data-ttu-id="18745-122">与 `FromSqlRaw` 一样，`FromSqlInterpolated` 只能在查询根上使用。</span><span class="sxs-lookup"><span data-stu-id="18745-122">Just like `FromSqlRaw`, `FromSqlInterpolated` can only be used on query roots.</span></span> <span data-ttu-id="18745-123">与上述示例一样，该值会转换为 `DbParameter`，且不易受到 SQL 注入攻击。</span><span class="sxs-lookup"><span data-stu-id="18745-123">As with the previous example, the value is converted to a `DbParameter` and isn't vulnerable to SQL injection.</span></span>

> [!NOTE]
> <span data-ttu-id="18745-124">在版本 3.0 之前，`FromSqlRaw` 和 `FromSqlInterpolated` 是名为 `FromSql` 的两个重载。</span><span class="sxs-lookup"><span data-stu-id="18745-124">Prior to version 3.0, `FromSqlRaw` and `FromSqlInterpolated` were two overloads named `FromSql`.</span></span> <span data-ttu-id="18745-125">有关详细信息，请参阅[历史版本部分](#previous-versions)。</span><span class="sxs-lookup"><span data-stu-id="18745-125">For more information, see the [previous versions section](#previous-versions).</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlInterpolatedStoredProcedureParameter)]

<span data-ttu-id="18745-126">还可以构造 DbParameter 并将其作为参数值提供。</span><span class="sxs-lookup"><span data-stu-id="18745-126">You can also construct a DbParameter and supply it as a parameter value.</span></span> <span data-ttu-id="18745-127">由于使用了常规 SQL 参数占位符而不是字符串占位符，因此可安全地使用 `FromSqlRaw`：</span><span class="sxs-lookup"><span data-stu-id="18745-127">Since a regular SQL parameter placeholder is used, rather than a string placeholder, `FromSqlRaw` can be safely used:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlRawStoredProcedureSqlParameter)]

<span data-ttu-id="18745-128">借助 `FromSqlRaw`，可以在 SQL 查询字符串中使用已命名的参数，这在存储的流程具有可选参数时非常有用：</span><span class="sxs-lookup"><span data-stu-id="18745-128">`FromSqlRaw` allows you to use named parameters in the SQL query string, which is useful when a stored procedure has optional parameters:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlRawStoredProcedureNamedSqlParameter)]

> [!NOTE]
> <span data-ttu-id="18745-129">**参数排序** Entity Framework Core 根据 `SqlParameter[]` 数组的顺序传递参数。</span><span class="sxs-lookup"><span data-stu-id="18745-129">**Parameter Ordering** Entity Framework Core passes parameters based on the order of the `SqlParameter[]` array.</span></span> <span data-ttu-id="18745-130">传递多个 `SqlParameter` 时，SQL 字符串中的顺序必须与存储过程定义中参数的顺序相匹配。</span><span class="sxs-lookup"><span data-stu-id="18745-130">When passing multiple `SqlParameter`s, the ordering in the SQL string must match the order of the parameters in the stored procedure's definition.</span></span> <span data-ttu-id="18745-131">如果不匹配，在执行此过程时，就可能会导致类型转换异常和/或异常行为。</span><span class="sxs-lookup"><span data-stu-id="18745-131">Failure to do this may result in type conversion exceptions and/or unexpected behavior when the procedure is executed.</span></span>

## <a name="composing-with-linq"></a><span data-ttu-id="18745-132">使用 LINQ 编写</span><span class="sxs-lookup"><span data-stu-id="18745-132">Composing with LINQ</span></span>

<span data-ttu-id="18745-133">可使用 LINQ 运算符在初始的原始 SQL 查询基础上进行组合。</span><span class="sxs-lookup"><span data-stu-id="18745-133">You can compose on top of the initial raw SQL query using LINQ operators.</span></span> <span data-ttu-id="18745-134">EF Core 将其视为子查询，并在数据库中对其进行组合。</span><span class="sxs-lookup"><span data-stu-id="18745-134">EF Core will treat it as subquery and compose over it in the database.</span></span> <span data-ttu-id="18745-135">下面的示例使用原始 SQL 查询，该查询从表值函数 (TVF) 中进行选择。</span><span class="sxs-lookup"><span data-stu-id="18745-135">The following example uses a raw SQL query that selects from a Table-Valued Function (TVF).</span></span> <span data-ttu-id="18745-136">然后，使用 LINQ 进行筛选和排序，从而对其进行组合。</span><span class="sxs-lookup"><span data-stu-id="18745-136">And then composes on it using LINQ to do filtering and sorting.</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlInterpolatedComposed)]

<span data-ttu-id="18745-137">上面的查询生成以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="18745-137">Above query generates following SQL:</span></span>

```sql
SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url]
FROM (
    SELECT * FROM dbo.SearchBlogs(@p0)
) AS [b]
WHERE [b].[Rating] > 3
ORDER BY [b].[Rating] DESC
```

### <a name="including-related-data"></a><span data-ttu-id="18745-138">包含关联数据</span><span class="sxs-lookup"><span data-stu-id="18745-138">Including related data</span></span>

<span data-ttu-id="18745-139">`Include` 方法可用于添加相关数据，就像对其他 LINQ 查询那样：</span><span class="sxs-lookup"><span data-stu-id="18745-139">The `Include` method can be used to include related data, just like with any other LINQ query:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlInterpolatedInclude)]

<span data-ttu-id="18745-140">使用 LINQ 进行组合要求原始 SQL 查询是可组合的，因为 EF Core 会将提供的 SQL 视为子查询。</span><span class="sxs-lookup"><span data-stu-id="18745-140">Composing with LINQ requires your raw SQL query to be composable since EF Core will treat the supplied SQL as a subquery.</span></span> <span data-ttu-id="18745-141">以 `SELECT` 关键字开始的 SQL 查询一般是可组合的。</span><span class="sxs-lookup"><span data-stu-id="18745-141">SQL queries that can be composed on begin with the `SELECT` keyword.</span></span> <span data-ttu-id="18745-142">此外，传递的 SQL 不应包含子查询上无效的任何字符或选项，如：</span><span class="sxs-lookup"><span data-stu-id="18745-142">Further, SQL passed shouldn't contain any characters or options that aren't valid on a subquery, such as:</span></span>

- <span data-ttu-id="18745-143">结尾分号</span><span class="sxs-lookup"><span data-stu-id="18745-143">A trailing semicolon</span></span>
- <span data-ttu-id="18745-144">在 SQL Server 上，结尾处的查询级提示（例如，`OPTION (HASH JOIN)`）</span><span class="sxs-lookup"><span data-stu-id="18745-144">On SQL Server, a trailing query-level hint (for example, `OPTION (HASH JOIN)`)</span></span>
- <span data-ttu-id="18745-145">在 SQL Server 上，`SELECT` 子句中不与 `OFFSET 0` 或 `TOP 100 PERCENT` 配合使用的 `ORDER BY` 子句</span><span class="sxs-lookup"><span data-stu-id="18745-145">On SQL Server, an `ORDER BY` clause that isn't used with `OFFSET 0` OR `TOP 100 PERCENT` in the `SELECT` clause</span></span>

<span data-ttu-id="18745-146">SQL Server 不允许对存储过程调用进行组合，因此任何尝试向此类调用应用其他查询运算符的操作都将导致无效的 SQL。</span><span class="sxs-lookup"><span data-stu-id="18745-146">SQL Server doesn't allow composing over stored procedure calls, so any attempt to apply additional query operators to such a call will result in invalid SQL.</span></span> <span data-ttu-id="18745-147">请在 `FromSqlRaw` 或 `FromSqlInterpolated` 方法之后立即使用 `AsEnumerable` 或 `AsAsyncEnumerable` 方法，确保 EF Core 不会尝试对存储过程进行组合。</span><span class="sxs-lookup"><span data-stu-id="18745-147">Use `AsEnumerable` or `AsAsyncEnumerable` method right after `FromSqlRaw` or `FromSqlInterpolated` methods to make sure that EF Core doesn't try to compose over a stored procedure.</span></span>

## <a name="change-tracking"></a><span data-ttu-id="18745-148">更改跟踪</span><span class="sxs-lookup"><span data-stu-id="18745-148">Change Tracking</span></span>

<span data-ttu-id="18745-149">使用 `FromSqlRaw` 或 `FromSqlInterpolated` 方法的查询遵循与 EF Core 中所有其他 LINQ 查询完全相同的更改跟踪规则。</span><span class="sxs-lookup"><span data-stu-id="18745-149">Queries that use the `FromSqlRaw` or `FromSqlInterpolated` methods follow the exact same change tracking rules as any other LINQ query in EF Core.</span></span> <span data-ttu-id="18745-150">例如，如果该查询投影实体类型，默认情况下会跟踪结果。</span><span class="sxs-lookup"><span data-stu-id="18745-150">For example, if the query projects entity types, the results will be tracked by default.</span></span>

<span data-ttu-id="18745-151">下面的示例使用原始 SQL 查询，该查询从表值函数 (TVF) 中进行选择，然后禁用通过对 `AsNoTracking` 的调用来更改跟踪：</span><span class="sxs-lookup"><span data-stu-id="18745-151">The following example uses a raw SQL query that selects from a Table-Valued Function (TVF), then disables change tracking with the call to `AsNoTracking`:</span></span>

[!code-csharp[Main](../../../samples/core/Querying/RawSQL/Program.cs#FromSqlInterpolatedAsNoTracking)]

## <a name="limitations"></a><span data-ttu-id="18745-152">限制</span><span class="sxs-lookup"><span data-stu-id="18745-152">Limitations</span></span>

<span data-ttu-id="18745-153">使用原生 SQL 查询时需注意以下几个限制：</span><span class="sxs-lookup"><span data-stu-id="18745-153">There are a few limitations to be aware of when using raw SQL queries:</span></span>

- <span data-ttu-id="18745-154">SQL 查询必须返回实体类型的所有属性的数据。</span><span class="sxs-lookup"><span data-stu-id="18745-154">The SQL query must return data for all properties of the entity type.</span></span>
- <span data-ttu-id="18745-155">结果集中的列名必须与属性映射到的列名称匹配。</span><span class="sxs-lookup"><span data-stu-id="18745-155">The column names in the result set must match the column names that properties are mapped to.</span></span> <span data-ttu-id="18745-156">请注意，此行为不同于 EF6。</span><span class="sxs-lookup"><span data-stu-id="18745-156">Note this behavior is different from EF6.</span></span> <span data-ttu-id="18745-157">EF6 中忽略了原始 SQL 查询的属性/列映射关系，结果集列名必须与属性名相匹配。</span><span class="sxs-lookup"><span data-stu-id="18745-157">EF6 ignored property to column mapping for raw SQL queries and result set column names had to match the property names.</span></span>
- <span data-ttu-id="18745-158">SQL 查询不能包含关联数据。</span><span class="sxs-lookup"><span data-stu-id="18745-158">The SQL query can't contain related data.</span></span> <span data-ttu-id="18745-159">但是，在许多情况下你可以在查询后面紧跟着使用 `Include` 方法以返回关联数据（请参阅[包含关联数据](#including-related-data)）。</span><span class="sxs-lookup"><span data-stu-id="18745-159">However, in many cases you can compose on top of the query using the `Include` operator to return related data (see [Including related data](#including-related-data)).</span></span>

## <a name="previous-versions"></a><span data-ttu-id="18745-160">旧版</span><span class="sxs-lookup"><span data-stu-id="18745-160">Previous versions</span></span>

<span data-ttu-id="18745-161">EF Core 2.2 及更低版本具有两个名为 `FromSql` 的方法重载，它们的行为方式与较新的 `FromSqlRaw` 和 `FromSqlInterpolated` 的相同。</span><span class="sxs-lookup"><span data-stu-id="18745-161">EF Core version 2.2 and earlier had two overloads of method named `FromSql`, which behaved in the same way as the newer `FromSqlRaw` and `FromSqlInterpolated`.</span></span> <span data-ttu-id="18745-162">因此，在意图调用内插字符串方法时很容易意外调用原始字符串方法，反之亦然。</span><span class="sxs-lookup"><span data-stu-id="18745-162">It was easy to accidentally call the raw string method when the intent was to call the interpolated string method, and the other way around.</span></span> <span data-ttu-id="18745-163">意外调用错误的重载会导致本该参数化的查询没有参数化。</span><span class="sxs-lookup"><span data-stu-id="18745-163">Calling wrong overload accidentally could result in queries not being parameterized when they should have been.</span></span>
