---
title: 数据库函数 - EF Core
description: 有关 EF Core 查询转换中支持的数据库函数的信息
author: smitpatel
ms.date: 11/10/2020
uid: core/querying/database-functions
ms.openlocfilehash: 34ced2a602168f6ed84763c046ef581cb87026e8
ms.sourcegitcommit: 42bbf7f68e92c364c5fff63092d3eb02229f568d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/11/2020
ms.locfileid: "94503779"
---
# <a name="database-functions"></a><span data-ttu-id="5a162-103">数据库函数</span><span class="sxs-lookup"><span data-stu-id="5a162-103">Database Functions</span></span>

<span data-ttu-id="5a162-104">数据库函数是 [C# 方法](/dotnet/csharp/programming-guide/classes-and-structs/methods)的数据库等效项。</span><span class="sxs-lookup"><span data-stu-id="5a162-104">Database functions are the database equivalent of [C# methods](/dotnet/csharp/programming-guide/classes-and-structs/methods).</span></span> <span data-ttu-id="5a162-105">数据库函数可以使用零个或更多个参数调用，它会根据参数值计算结果。</span><span class="sxs-lookup"><span data-stu-id="5a162-105">A database function can be invoked with zero or more parameters and it computes the result based on the parameter values.</span></span> <span data-ttu-id="5a162-106">大多数使用 SQL 进行查询的数据库都支持数据库函数。</span><span class="sxs-lookup"><span data-stu-id="5a162-106">Most databases, which use SQL for querying have support for database functions.</span></span> <span data-ttu-id="5a162-107">因此，EF Core 查询转换生成的 SQL 也允许调用数据库函数。</span><span class="sxs-lookup"><span data-stu-id="5a162-107">So SQL generated by EF Core query translation also allows invoking database functions.</span></span> <span data-ttu-id="5a162-108">在 EF Core 中，C# 方法不必严格地转换为数据库函数。</span><span class="sxs-lookup"><span data-stu-id="5a162-108">C# methods don't have to translate strictly to database functions in EF Core.</span></span>

- <span data-ttu-id="5a162-109">C# 方法可能没有等效的数据库函数。</span><span class="sxs-lookup"><span data-stu-id="5a162-109">A C# method may not have an equivalent database function.</span></span>
  - <span data-ttu-id="5a162-110"><xref:System.String.IsNullOrEmpty%2A?displayProperty=nameWithType> 方法会转换为 null 检查和与数据库中空字符串的比较，而不会转换为一个函数。</span><span class="sxs-lookup"><span data-stu-id="5a162-110"><xref:System.String.IsNullOrEmpty%2A?displayProperty=nameWithType> method translates to a null check and a comparison with an empty string in the database rather than a function.</span></span>
  - <span data-ttu-id="5a162-111"><xref:System.String.Equals(System.String,System.StringComparison)?displayProperty=nameWithType> 方法没有数据库等效项，因为在数据库中表示或模拟字符串比较并非易事。</span><span class="sxs-lookup"><span data-stu-id="5a162-111"><xref:System.String.Equals(System.String,System.StringComparison)?displayProperty=nameWithType> method doesn't have database equivalent since string comparison can't be represented or mimicked easily in a database.</span></span>
- <span data-ttu-id="5a162-112">数据库函数可能没有等效的 C# 方法。</span><span class="sxs-lookup"><span data-stu-id="5a162-112">A database function may not have an equivalent C# method.</span></span> <span data-ttu-id="5a162-113">C# 中的 `??` 运算符没有任何方法，它会转换为数据库中的 `COALESCE` 函数。</span><span class="sxs-lookup"><span data-stu-id="5a162-113">The `??` operator in C#, which doesn't have any method, translates to the `COALESCE` function in the database.</span></span>

## <a name="types-of-database-functions"></a><span data-ttu-id="5a162-114">数据库函数的类型</span><span class="sxs-lookup"><span data-stu-id="5a162-114">Types of database functions</span></span>

<span data-ttu-id="5a162-115">EF Core SQL 生成支持可在数据库中使用的部分函数。</span><span class="sxs-lookup"><span data-stu-id="5a162-115">EF Core SQL generation supports a subset of functions that can be used in databases.</span></span> <span data-ttu-id="5a162-116">此限制源自采用给定数据库函数的 LINQ 表示查询的能力。</span><span class="sxs-lookup"><span data-stu-id="5a162-116">This limitation comes from the ability to represent a query in LINQ for the given database function.</span></span> <span data-ttu-id="5a162-117">而且，每个数据库对数据库函数的支持各异，因此 EF Core 提供了一个通用子集。</span><span class="sxs-lookup"><span data-stu-id="5a162-117">Further, each database has varying support of database functions, so EF Core provides a common subset.</span></span> <span data-ttu-id="5a162-118">数据库提供程序可免费将 EF Core SQL 生成扩展为支持更多的模式。</span><span class="sxs-lookup"><span data-stu-id="5a162-118">A database provider is free to extend EF Core SQL generation to support more patterns.</span></span> <span data-ttu-id="5a162-119">下面是 EF Core 支持并唯一标识的数据库函数类型。</span><span class="sxs-lookup"><span data-stu-id="5a162-119">Following are the types of database functions EF Core supports and uniquely identifies.</span></span> <span data-ttu-id="5a162-120">这些条目也有助于了解哪些转换是 EF Core 提供程序内置的。</span><span class="sxs-lookup"><span data-stu-id="5a162-120">These terms also help in understanding what translations come built in with EF Core providers.</span></span>

### <a name="built-in-vs-user-defined-functions"></a><span data-ttu-id="5a162-121">内置函数与用户定义的函数</span><span class="sxs-lookup"><span data-stu-id="5a162-121">Built-in vs user-defined functions</span></span>

<span data-ttu-id="5a162-122">内置函数是数据库中预定义的，而用户定义的函数是由数据库用户显式定义的。</span><span class="sxs-lookup"><span data-stu-id="5a162-122">Built-in functions come with database predefined, but user-defined functions are explicitly defined by the user in the database.</span></span> <span data-ttu-id="5a162-123">EF Core 将查询转换为使用数据库函数时，它使用内置函数来确保该函数在数据库中始终可用。</span><span class="sxs-lookup"><span data-stu-id="5a162-123">When EF Core translates queries to use database functions, it uses built-in functions to make sure that the function is always available on the database.</span></span> <span data-ttu-id="5a162-124">在某些数据库中，需要了解内置函数的特征，才能正确生成 SQL。</span><span class="sxs-lookup"><span data-stu-id="5a162-124">The distinction of built-in functions is necessary in some databases to generate SQL correctly.</span></span> <span data-ttu-id="5a162-125">例如 SqlServer 要求使用架构限定的名称调用各个用户定义的函数。</span><span class="sxs-lookup"><span data-stu-id="5a162-125">For example SqlServer requires that every user-defined function is invoked with a schema-qualified name.</span></span> <span data-ttu-id="5a162-126">但 SqlServer 中的内置函数没有架构。</span><span class="sxs-lookup"><span data-stu-id="5a162-126">But built-in functions in SqlServer don't have a schema.</span></span> <span data-ttu-id="5a162-127">PostgreSQL 使用 `public` 架构定义内置函数，但可使用架构限定的名称调用它们。</span><span class="sxs-lookup"><span data-stu-id="5a162-127">PostgreSQL defines built-in function in the `public` schema but they can be invoked with schema-qualified names.</span></span>

### <a name="aggregate-vs-scalar-vs-table-valued-functions"></a><span data-ttu-id="5a162-128">聚合函数、标量函数和表值函数</span><span class="sxs-lookup"><span data-stu-id="5a162-128">Aggregate vs scalar vs table-valued functions</span></span>

- <span data-ttu-id="5a162-129">标量函数将标量值（如整数或字符串）用作参数，并返回标量值作为结果。</span><span class="sxs-lookup"><span data-stu-id="5a162-129">Scalar functions take scalar values - like integers or strings - as parameters and return a scalar value as the result.</span></span> <span data-ttu-id="5a162-130">可在 SQL 中任意可以传递标量值的地方使用标量函数。</span><span class="sxs-lookup"><span data-stu-id="5a162-130">Scalar functions can be used anywhere in SQL where a scalar value can be passed.</span></span>
- <span data-ttu-id="5a162-131">聚合函数将一系列的标量值用作参数，并返回标量值作为结果。</span><span class="sxs-lookup"><span data-stu-id="5a162-131">Aggregate functions take a stream of scalar values as parameters and return a scalar value as the result.</span></span> <span data-ttu-id="5a162-132">聚合函数应用于整个查询结果集或应用 `GROUP BY` 运算符所生成的一组值。</span><span class="sxs-lookup"><span data-stu-id="5a162-132">Aggregate functions are applied on the whole query result set or on a group of values generated by applying `GROUP BY` operator.</span></span>
- <span data-ttu-id="5a162-133">表值函数将标量值用作参数，并返回一系列的行作为结果。</span><span class="sxs-lookup"><span data-stu-id="5a162-133">Table-valued functions take scalar values as parameter(s) and return a stream of rows as the result.</span></span> <span data-ttu-id="5a162-134">表值函数用作 `FROM` 子句中的表源。</span><span class="sxs-lookup"><span data-stu-id="5a162-134">Table-valued functions are used as a table source in `FROM` clause.</span></span>

### <a name="niladic-functions"></a><span data-ttu-id="5a162-135">Niladic函数</span><span class="sxs-lookup"><span data-stu-id="5a162-135">Niladic functions</span></span>

<span data-ttu-id="5a162-136">Niladic 函数是特殊的数据库函数，没有任何参数，并且不得使用括号调用。</span><span class="sxs-lookup"><span data-stu-id="5a162-136">Niladic functions are special database functions that don't have any parameters and must be invoked without parenthesis.</span></span> <span data-ttu-id="5a162-137">它们类似于 C# 实例上的属性/字段访问。</span><span class="sxs-lookup"><span data-stu-id="5a162-137">They're similar to property/field access on an instance in C#.</span></span> <span data-ttu-id="5a162-138">Niladic 函数不同于无参数函数，因为后者需要使用空白括号。</span><span class="sxs-lookup"><span data-stu-id="5a162-138">Niladic functions differ from parameter-less functions as the latter require empty parenthesis.</span></span> <span data-ttu-id="5a162-139">始终需要使用括号的数据库函数没有特殊的名称。</span><span class="sxs-lookup"><span data-stu-id="5a162-139">There's no special name for database functions that requires parenthesis always.</span></span> <span data-ttu-id="5a162-140">另外一部分基于参数计数的数据库函数是可变参数函数。</span><span class="sxs-lookup"><span data-stu-id="5a162-140">Another subset of database functions based on parameter count is variadic functions.</span></span> <span data-ttu-id="5a162-141">可变参数函数可以采用不同的参数数量进行调用。</span><span class="sxs-lookup"><span data-stu-id="5a162-141">Variadic functions can take varying number of parameters when invoked.</span></span>

## <a name="database-function-mappings-in-ef-core"></a><span data-ttu-id="5a162-142">EF Core 中的数据库函数映射</span><span class="sxs-lookup"><span data-stu-id="5a162-142">Database function mappings in EF Core</span></span>

<span data-ttu-id="5a162-143">EF Core 支持三种不同的方式，来实现 C# 函数和数据库函数之间的映射。</span><span class="sxs-lookup"><span data-stu-id="5a162-143">EF Core supports three different ways of mapping between C# functions and database functions.</span></span>

### <a name="built-in-function-mapping"></a><span data-ttu-id="5a162-144">内置函数映射</span><span class="sxs-lookup"><span data-stu-id="5a162-144">Built-in function mapping</span></span>

<span data-ttu-id="5a162-145">默认情况下，EF Core 提供程序通过基元类型为各种内置函数提供映射。</span><span class="sxs-lookup"><span data-stu-id="5a162-145">By default EF Core providers provide mappings for various built-in functions over primitive types.</span></span> <span data-ttu-id="5a162-146">例如，在 SqlServer 中，<xref:System.String.ToLower?displayProperty=nameWithType> 会转换为 `LOWER`。</span><span class="sxs-lookup"><span data-stu-id="5a162-146">For example, <xref:System.String.ToLower?displayProperty=nameWithType> translates to `LOWER` in SqlServer.</span></span> <span data-ttu-id="5a162-147">用户可以通过此功能无缝地使用 LINQ 编写查询。</span><span class="sxs-lookup"><span data-stu-id="5a162-147">This functionality allows users to write queries in LINQ seamlessly.</span></span> <span data-ttu-id="5a162-148">通常，我们提供的数据库转换生成的结果要与客户端的 C# 函数所生成的内容相同。</span><span class="sxs-lookup"><span data-stu-id="5a162-148">We usually provide a translation in the database that gives the same result as what the C# function provides on the client side.</span></span> <span data-ttu-id="5a162-149">为实现此目的，有时实际的转换可能比数据库函数更为复杂。</span><span class="sxs-lookup"><span data-stu-id="5a162-149">Sometimes, to achieve that, the actual translation could be something more complicated than a database function.</span></span> <span data-ttu-id="5a162-150">在某些情况下，我们也会提供最适当的转换，而不是提供匹配的 C# 语义。</span><span class="sxs-lookup"><span data-stu-id="5a162-150">In some scenarios, we also provide the most appropriate translation rather than matching C# semantics.</span></span> <span data-ttu-id="5a162-151">同样的功能也可用于为某些 C# 成员访问提供通用的转换。</span><span class="sxs-lookup"><span data-stu-id="5a162-151">The same feature is also used to provide common translations for some of the C# member accesses.</span></span> <span data-ttu-id="5a162-152">例如，在 SqlServer 中，<xref:System.String.Length?displayProperty=nameWithType> 会转换为 `LEN`。</span><span class="sxs-lookup"><span data-stu-id="5a162-152">For example, <xref:System.String.Length?displayProperty=nameWithType> translates to `LEN` in SqlServer.</span></span> <span data-ttu-id="5a162-153">除了提供程序外，插件编写器也可以添加更多的转换。</span><span class="sxs-lookup"><span data-stu-id="5a162-153">Apart from providers, plugin writers can also add additional translations.</span></span> <span data-ttu-id="5a162-154">当插件添加对将更多类型作为基元类型的支持，并想要通过这些类型转换方法时，这种扩展性非常有用。</span><span class="sxs-lookup"><span data-stu-id="5a162-154">This extensibility is useful when plugins add support for more types as primitive types and want to translate methods over them.</span></span>

### <a name="effunctions-mapping"></a><span data-ttu-id="5a162-155">EF.Functions 映射</span><span class="sxs-lookup"><span data-stu-id="5a162-155">EF.Functions mapping</span></span>

<span data-ttu-id="5a162-156">由于并非所有数据库函数都有等效的 C# 函数，因此 EF Core 提供程序提供了特殊的 C# 方法来调用某些数据库函数。</span><span class="sxs-lookup"><span data-stu-id="5a162-156">Since not all database functions have equivalent C# functions, EF Core providers have special C# methods to invoke certain database functions.</span></span> <span data-ttu-id="5a162-157">这些方法通过 `EF.Functions` 定义为扩展方法来用于 LINQ 查询中。</span><span class="sxs-lookup"><span data-stu-id="5a162-157">These methods are defined as extension methods over `EF.Functions` to be used in LINQ queries.</span></span> <span data-ttu-id="5a162-158">这些方法是特定于提供程序的，因为它们与特定数据库函数密切相关。</span><span class="sxs-lookup"><span data-stu-id="5a162-158">These methods are provider-specific as they're closely tied with particular database functions.</span></span> <span data-ttu-id="5a162-159">因此，适用于某个提供程序的方法很可能不适用于任何其他提供程序。</span><span class="sxs-lookup"><span data-stu-id="5a162-159">So a method that works for one provider will likely not work for any other provider.</span></span> <span data-ttu-id="5a162-160">此外，由于这些方法旨在调用所转换查询中的数据库函数，因此尝试在客户端上计算这些方法会引发异常。</span><span class="sxs-lookup"><span data-stu-id="5a162-160">Further, since the intention of these methods is to invoke a database function in the translated query, trying to evaluate them on the client results in an exception.</span></span>

### <a name="user-defined-function-mapping"></a><span data-ttu-id="5a162-161">用户定义的函数映射</span><span class="sxs-lookup"><span data-stu-id="5a162-161">User-defined function mapping</span></span>

<span data-ttu-id="5a162-162">除了 EF Core 提供程序提供的映射以外，用户还可以定义自定义映射。</span><span class="sxs-lookup"><span data-stu-id="5a162-162">Apart from mappings provided by EF Core providers, users can also define custom mapping.</span></span> <span data-ttu-id="5a162-163">用户定义的映射会根据用户需求扩展查询转换。</span><span class="sxs-lookup"><span data-stu-id="5a162-163">A user-defined mapping extends the query translation according to the user needs.</span></span> <span data-ttu-id="5a162-164">当数据库中有用户想要从 LINQ 查询调用的用户定义的函数时，此功能非常有用。</span><span class="sxs-lookup"><span data-stu-id="5a162-164">This functionality is useful when there are user-defined functions in the database, which the user wants to invoke from their LINQ query.</span></span>

## <a name="see-also"></a><span data-ttu-id="5a162-165">另请参阅</span><span class="sxs-lookup"><span data-stu-id="5a162-165">See also</span></span>

- [<span data-ttu-id="5a162-166">SqlServer 内置函数映射</span><span class="sxs-lookup"><span data-stu-id="5a162-166">SqlServer built-in function mappings</span></span>](xref:core/providers/sql-server/functions)
- [<span data-ttu-id="5a162-167">Sqlite 内置函数映射</span><span class="sxs-lookup"><span data-stu-id="5a162-167">Sqlite built-in function mappings</span></span>](xref:core/providers/sqlite/functions)
- [<span data-ttu-id="5a162-168">Cosmos 内置函数映射</span><span class="sxs-lookup"><span data-stu-id="5a162-168">Cosmos built-in function mappings</span></span>](xref:core/providers/cosmos/functions)