---
title: 索引-EF Core
description: 在 Entity Framework Core 模型中配置索引
author: roji
ms.date: 12/16/2019
uid: core/modeling/indexes
ms.openlocfilehash: ab81b108c4ff518cf98b7e835da3553c0c41efed
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128532"
---
# <a name="indexes"></a>索引

索引是跨多个数据存储区的常见概念。 尽管它们在数据存储中的实现可能会有所不同，但也可用于基于列 (或一组列来进行查找，) 效率更高。 有关良好索引使用的详细信息，请参阅性能文档中的 " [索引" 一节](xref:core/performance/efficient-querying#use-indexes-properly) 。

您可以对列指定索引，如下所示：

## <a name="data-annotations"></a>[数据批注](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/Index.cs?name=Index&highlight=1)]

> [!NOTE]
> EF Core 5.0 中引入了通过数据批注配置索引。

## <a name="fluent-api"></a>[Fluent API](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Index.cs?name=Index&highlight=4)]

***

> [!NOTE]
> 按照约定，会在每个属性中创建一个索引 (或一组用作外键的属性) 。
>
> EF Core 每个不同的属性集仅支持一个索引。 如果对已定义索引的一组属性（按约定或以前的配置）配置索引，则会更改该索引的定义。 如果要进一步配置由约定创建的索引，则此操作非常有用。

## <a name="composite-index"></a>复合索引

索引还可以跨多个列：

### <a name="data-annotations"></a>[数据批注](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/IndexComposite.cs?name=Composite&highlight=1)]

### <a name="fluent-api"></a>[Fluent API](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexComposite.cs?name=Composite&highlight=4)]

**_

跨多个列的索引，也称为 _composite 索引 *，加速对索引列进行筛选的查询，以及仅筛选索引覆盖的 *第一* 列的查询。 有关详细信息，请参阅 [性能文档](xref:core/performance/efficient-querying#use-indexes-properly) 。

## <a name="index-uniqueness"></a>索引唯一性

默认情况下，索引不唯一：对于索引的列集，允许多行具有相同的值 (s) 。 可以使索引唯一，如下所示：

### <a name="data-annotations"></a>[数据批注](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/IndexUnique.cs?name=IndexUnique&highlight=1)]

### <a name="fluent-api"></a>[Fluent API](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexUnique.cs?name=IndexUnique&highlight=5)]

***

尝试为索引的列集插入多个具有相同值的实体将导致引发异常。

## <a name="index-name"></a>索引名称

按照约定，在关系数据库中创建的索引命名为 `IX_<type name>_<property name>` 。 对于复合索引， `<property name>` 将成为以下划线分隔的属性名称列表。

可以设置在数据库中创建的索引的名称：

### <a name="data-annotations"></a>[数据批注](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/IndexName.cs?name=IndexName&highlight=1)]

### <a name="fluent-api"></a>[Fluent API](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexName.cs?name=IndexName&highlight=5)]

***

## <a name="index-filter"></a>索引筛选器

某些关系数据库允许您指定筛选索引或部分索引。 这使您可以只为列的值的一个子集编制索引，从而减少索引的大小并改善性能和磁盘空间的使用情况。 有关 SQL Server 筛选索引的详细信息， [请参阅文档](/sql/relational-databases/indexes/create-filtered-indexes)。

您可以使用熟知的 API 来指定索引的筛选器，作为 SQL 表达式提供：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexFilter.cs?name=IndexFilter&highlight=5)]

当使用 SQL Server 提供程序 EF 时 `'IS NOT NULL'` ，将为唯一索引中包含的所有可以为 null 的列添加筛选器。 若要重写此约定，可以提供一个 `null` 值。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexNoFilter.cs?name=IndexNoFilter&highlight=6)]

## <a name="included-columns"></a>包含列

某些关系数据库允许配置一组列，这些列包含在索引中，但不是其 "键" 的一部分。 当查询中的所有列都作为键列或非键列包含在索引中时，这可以显著提高查询性能，因为表本身无需访问。 有关 SQL Server 包含列的详细信息， [请参阅文档](/sql/relational-databases/indexes/create-indexes-with-included-columns)。

在下面的示例中， `Url` 列是索引键的一部分，因此对该列的任何查询筛选都可以使用索引。 但此外，仅访问 `Title` 和列的查询 `PublishedOn` 将不需要访问表并更有效地运行：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexInclude.cs?name=IndexInclude&highlight=5-9)]
