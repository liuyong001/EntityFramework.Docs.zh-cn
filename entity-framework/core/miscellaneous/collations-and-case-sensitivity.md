---
title: 排序规则和区分大小写-EF Core
description: 在数据库和查询中配置排序规则和区分大小写的 Entity Framework Core
author: roji
ms.date: 04/27/2020
uid: core/miscellaneous/collations-and-case-sensitivity
ms.openlocfilehash: 48e0a6b316742dc1467134ae81f90086bb93d011
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543531"
---
# <a name="collations-and-case-sensitivity"></a>排序规则和区分大小写

> [!NOTE]
> EF Core 5.0 中已引入此功能。

数据库中的文本处理可能会很复杂，并且需要更多用户注意。 一方面，数据库处理文本的方式有很大差别;例如，尽管某些数据库在默认情况下是区分大小写的 (例如 Sqlite、PostgreSQL) 等，但其他数据库不区分大小写， (SQL Server MySQL) 。 此外，由于索引使用情况，区分大小写和类似方面可能会对查询性能产生很大的影响：尽管在区分大小写的 `string.Lower` 数据库中使用强制不区分大小写的比较可能会很有吸引力，但这样做可能会阻止应用程序使用索引。 本页详细说明了如何配置区分大小写或更一般的排序规则，以及如何在不影响查询性能的情况下有效地执行此操作。

## <a name="introduction-to-collations"></a>排序规则简介

文本处理中的基本概念是 *排序规则*，这是一组规则，用于确定文本值如何排序和比较是否相等。 例如，虽然不区分大小写的排序规则不会在大写字母和小写字母之间忽略差异，所以区分大小写的排序规则不会。 但是，因为区分大小写是区分区域性的 (例如， `i` 并 `I` 表示土耳其) 中的不同字母，所以存在多个不区分大小写的排序规则，每个排序规则都具有自己的一组规则。 排序规则的范围还超出了区分大小写的字符数据的其他方面;例如，在德语中，它有时 (，但并不总是) 需要将 `ä` 和视为 `ae` 相同。 最后，排序规则还定义文本值的 *排序* 方式：德语位置 `ä` 之后 `a` ，瑞典语会将其置于字母表的末尾。

数据库中的所有文本操作都使用排序规则，无论是显式还是隐式地确定操作比较和排序字符串的方式。 可用排序规则及其命名方案的实际列表是特定于数据库的;请参阅 [以下部分](#database-specific-information) ，了解各种数据库相关文档页的链接。 幸运的是，数据库通常允许在数据库级或列级定义默认排序规则，并显式指定哪个排序规则应该用于查询中的特定操作。

## <a name="database-collation"></a>数据库排序规则

在大多数数据库系统中，默认排序规则是在数据库级别定义的;除非重写，否则该排序规则将隐式应用于该数据库中发生的所有文本操作。 数据库排序规则通常是在创建数据库时通过 `CREATE DATABASE` DDL 语句)  (设置的，如果未指定，则默认为在安装时确定的某个服务器级别值。 例如，SQL Server 中的默认服务器级排序规则是 `SQL_Latin1_General_CP1_CI_AS` ，这是不区分大小写且区分重音的排序规则。 尽管数据库系统通常允许更改现有数据库的排序规则，但这样做可能会导致复杂;建议在创建数据库之前选择排序规则。

使用 EF Core 迁移来管理数据库架构时，模型的方法中的以下 `OnModelCreating` 内容将 SQL Server 数据库配置为使用区分大小写的排序规则：

[!code-csharp[Main](../../../samples/core/Miscellaneous/Collations/Program.cs?name=DatabaseCollation)]

## <a name="column-collation"></a>列排序规则

也可以在文本列上定义排序规则，以覆盖数据库默认值。 如果某些列需要区分大小写，而数据库的其余部分需要区分大小写，则这会很有用。

使用 EF Core 迁移来管理数据库架构时，以下内容将属性的列配置为在其他情况下被配置为区分大小写的 `Name` 数据库中的不区分大小写：

[!code-csharp[Main](../../../samples/core/Miscellaneous/Collations/Program.cs?name=ColumnCollation)]

## <a name="explicit-collation-in-a-query"></a>查询中的显式排序规则

在某些情况下，需要通过不同的查询使用不同的排序规则查询同一列。 例如，一个查询可能需要对列执行区分大小写的比较，而另一个查询可能需要对同一列执行不区分大小写的比较。 这可以通过在查询本身中显式指定排序规则来实现：

[!code-csharp[Main](../../../samples/core/Miscellaneous/Collations/Program.cs?name=SimpleQueryCollation)]

这会 `COLLATE` 在 SQL 查询中生成一个子句，该子句将应用区分大小写的排序规则，而不考虑在列或数据库级别定义的排序规则：

```sql
SELECT [c].[Id], [c].[Name]
FROM [Customers] AS [c]
WHERE [c].[Name] COLLATE SQL_Latin1_General_CP1_CS_AS = N'John'
```

### <a name="explicit-collations-and-indexes"></a>显式排序规则和索引

索引是数据库性能最重要的因素之一-使用索引高效运行的查询可以网格不使用该索引的停止。 索引隐式继承列的排序规则;这意味着，对列的所有查询都将自动符合使用该列定义的索引的条件，前提是该查询不指定其他排序规则。 在查询中指定显式排序规则通常会阻止该查询使用该列上定义的索引，因为排序规则将不再匹配;因此，建议在使用此功能时务必小心。 最好是在 (或数据库) 级别的列定义排序规则，以允许所有查询隐式使用该排序规则并受益于任何索引。

请注意，某些数据库允许在创建索引 (例如 PostgreSQL、Sqlite) 时定义排序规则。 这允许在同一列中定义多个索引，从而加速具有不同排序规则的操作 (例如，区分大小写和不区分大小写的比较) 。 有关更多详细信息，请参阅数据库提供程序的文档。

> [!WARNING]
> 请始终检查查询的查询计划，并确保在对大量数据执行的性能关键查询中使用正确的索引。 通过 (或通过调用) 来覆盖查询中的区分大小写 `EF.Functions.Collate` `string.ToLower` 会对应用程序的性能产生很大的影响。

## <a name="translation-of-built-in-net-string-operations"></a>内置 .NET 字符串操作的翻译

在 .NET 中，string 相等性默认情况下区分大小写： `s1 == s2` 执行要求字符串相同的序号比较。 因为数据库的默认排序规则会变化，并且因为需要简单的相等性来使用索引，所以 EF Core 不会尝试将简单的相等转换为区分大小写的数据库操作： c # 相等性会直接转换为 SQL 相等性，这可能会也可能不区分大小写，具体取决于所使用的特定数据库及其排序规则配置。

此外，.NET 还提供了 [`string.Equals`](/dotnet/api/system.string.equals#System_String_Equals_System_String_System_StringComparison_) 接受枚举的重载 [`StringComparison`](/dotnet/api/system.stringcomparison) ，这允许为比较指定区分大小写和文化。 按照设计，EF Core refrains 将这些重载转换为 SQL，尝试使用它们将导致异常。 一方面，EF Core 知道不应使用不区分大小写的排序规则或不区分大小写的排序规则。 更重要的是，在大多数情况下，应用排序规则会阻止索引的使用，对非常基本的常用 .NET 构造的性能产生显著影响。 若要强制查询使用区分大小写或不区分大小写的比较，请通过 `EF.Functions.Collate` [上面详细](#explicit-collations-and-indexes)说明显式指定排序规则。

## <a name="additional-resources"></a>其他资源

### <a name="database-specific-information"></a>数据库特定的信息

* [SQL Server 有关排序规则的文档](/sql/relational-databases/collations/collation-and-unicode-support)。
* [有关排序规则的数据表](/dotnet/standard/data/sqlite/collation)。
* [有关排序规则的 PostgreSQL 文档](https://www.postgresql.org/docs/current/collation.html)。
* [有关排序规则的 MySQL 文档](https://dev.mysql.com/doc/refman/en/charset-general.html)。

### <a name="other-resources"></a>其他资源

* [EF Core 社区 Standup 会话](https://www.youtube.com/watch?v=OgMhLVa_VfA&list=PLdo4fOcmZ0oX-DBuRG4u58ZTAJgBAeQ-t&index=1)，介绍排序规则和探索性能和索引方面。
