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
# <a name="database-functions"></a>数据库函数

数据库函数是 [C# 方法](/dotnet/csharp/programming-guide/classes-and-structs/methods)的数据库等效项。 数据库函数可以使用零个或更多个参数调用，它会根据参数值计算结果。 大多数使用 SQL 进行查询的数据库都支持数据库函数。 因此，EF Core 查询转换生成的 SQL 也允许调用数据库函数。 在 EF Core 中，C# 方法不必严格地转换为数据库函数。

- C# 方法可能没有等效的数据库函数。
  - <xref:System.String.IsNullOrEmpty%2A?displayProperty=nameWithType> 方法会转换为 null 检查和与数据库中空字符串的比较，而不会转换为一个函数。
  - <xref:System.String.Equals(System.String,System.StringComparison)?displayProperty=nameWithType> 方法没有数据库等效项，因为在数据库中表示或模拟字符串比较并非易事。
- 数据库函数可能没有等效的 C# 方法。 C# 中的 `??` 运算符没有任何方法，它会转换为数据库中的 `COALESCE` 函数。

## <a name="types-of-database-functions"></a>数据库函数的类型

EF Core SQL 生成支持可在数据库中使用的部分函数。 此限制源自采用给定数据库函数的 LINQ 表示查询的能力。 而且，每个数据库对数据库函数的支持各异，因此 EF Core 提供了一个通用子集。 数据库提供程序可免费将 EF Core SQL 生成扩展为支持更多的模式。 下面是 EF Core 支持并唯一标识的数据库函数类型。 这些条目也有助于了解哪些转换是 EF Core 提供程序内置的。

### <a name="built-in-vs-user-defined-functions"></a>内置函数与用户定义的函数

内置函数是数据库中预定义的，而用户定义的函数是由数据库用户显式定义的。 EF Core 将查询转换为使用数据库函数时，它使用内置函数来确保该函数在数据库中始终可用。 在某些数据库中，需要了解内置函数的特征，才能正确生成 SQL。 例如 SqlServer 要求使用架构限定的名称调用各个用户定义的函数。 但 SqlServer 中的内置函数没有架构。 PostgreSQL 使用 `public` 架构定义内置函数，但可使用架构限定的名称调用它们。

### <a name="aggregate-vs-scalar-vs-table-valued-functions"></a>聚合函数、标量函数和表值函数

- 标量函数将标量值（如整数或字符串）用作参数，并返回标量值作为结果。 可在 SQL 中任意可以传递标量值的地方使用标量函数。
- 聚合函数将一系列的标量值用作参数，并返回标量值作为结果。 聚合函数应用于整个查询结果集或应用 `GROUP BY` 运算符所生成的一组值。
- 表值函数将标量值用作参数，并返回一系列的行作为结果。 表值函数用作 `FROM` 子句中的表源。

### <a name="niladic-functions"></a>Niladic函数

Niladic 函数是特殊的数据库函数，没有任何参数，并且不得使用括号调用。 它们类似于 C# 实例上的属性/字段访问。 Niladic 函数不同于无参数函数，因为后者需要使用空白括号。 始终需要使用括号的数据库函数没有特殊的名称。 另外一部分基于参数计数的数据库函数是可变参数函数。 可变参数函数可以采用不同的参数数量进行调用。

## <a name="database-function-mappings-in-ef-core"></a>EF Core 中的数据库函数映射

EF Core 支持三种不同的方式，来实现 C# 函数和数据库函数之间的映射。

### <a name="built-in-function-mapping"></a>内置函数映射

默认情况下，EF Core 提供程序通过基元类型为各种内置函数提供映射。 例如，在 SqlServer 中，<xref:System.String.ToLower?displayProperty=nameWithType> 会转换为 `LOWER`。 用户可以通过此功能无缝地使用 LINQ 编写查询。 通常，我们提供的数据库转换生成的结果要与客户端的 C# 函数所生成的内容相同。 为实现此目的，有时实际的转换可能比数据库函数更为复杂。 在某些情况下，我们也会提供最适当的转换，而不是提供匹配的 C# 语义。 同样的功能也可用于为某些 C# 成员访问提供通用的转换。 例如，在 SqlServer 中，<xref:System.String.Length?displayProperty=nameWithType> 会转换为 `LEN`。 除了提供程序外，插件编写器也可以添加更多的转换。 当插件添加对将更多类型作为基元类型的支持，并想要通过这些类型转换方法时，这种扩展性非常有用。

### <a name="effunctions-mapping"></a>EF.Functions 映射

由于并非所有数据库函数都有等效的 C# 函数，因此 EF Core 提供程序提供了特殊的 C# 方法来调用某些数据库函数。 这些方法通过 `EF.Functions` 定义为扩展方法来用于 LINQ 查询中。 这些方法是特定于提供程序的，因为它们与特定数据库函数密切相关。 因此，适用于某个提供程序的方法很可能不适用于任何其他提供程序。 此外，由于这些方法旨在调用所转换查询中的数据库函数，因此尝试在客户端上计算这些方法会引发异常。

### <a name="user-defined-function-mapping"></a>用户定义的函数映射

除了 EF Core 提供程序提供的映射以外，用户还可以定义自定义映射。 用户定义的映射会根据用户需求扩展查询转换。 当数据库中有用户想要从 LINQ 查询调用的用户定义的函数时，此功能非常有用。

## <a name="see-also"></a>另请参阅

- [SqlServer 内置函数映射](xref:core/providers/sql-server/functions)
- [Sqlite 内置函数映射](xref:core/providers/sqlite/functions)
- [Cosmos 内置函数映射](xref:core/providers/cosmos/functions)
