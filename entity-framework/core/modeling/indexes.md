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
# <a name="indexes"></a><span data-ttu-id="8a28a-103">索引</span><span class="sxs-lookup"><span data-stu-id="8a28a-103">Indexes</span></span>

<span data-ttu-id="8a28a-104">索引是跨多个数据存储区的常见概念。</span><span class="sxs-lookup"><span data-stu-id="8a28a-104">Indexes are a common concept across many data stores.</span></span> <span data-ttu-id="8a28a-105">尽管它们在数据存储中的实现可能会有所不同，但也可用于基于列 (或一组列来进行查找，) 效率更高。</span><span class="sxs-lookup"><span data-stu-id="8a28a-105">While their implementation in the data store may vary, they are used to make lookups based on a column (or set of columns) more efficient.</span></span> <span data-ttu-id="8a28a-106">有关良好索引使用的详细信息，请参阅性能文档中的 " [索引" 一节](xref:core/performance/efficient-querying#use-indexes-properly) 。</span><span class="sxs-lookup"><span data-stu-id="8a28a-106">See the [indexes section](xref:core/performance/efficient-querying#use-indexes-properly) in the performance documentation for more information on good index usage.</span></span>

<span data-ttu-id="8a28a-107">您可以对列指定索引，如下所示：</span><span class="sxs-lookup"><span data-stu-id="8a28a-107">You can specify an index over a column as follows:</span></span>

## <a name="data-annotations"></a>[<span data-ttu-id="8a28a-108">数据批注</span><span class="sxs-lookup"><span data-stu-id="8a28a-108">Data Annotations</span></span>](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/Index.cs?name=Index&highlight=1)]

> [!NOTE]
> <span data-ttu-id="8a28a-109">EF Core 5.0 中引入了通过数据批注配置索引。</span><span class="sxs-lookup"><span data-stu-id="8a28a-109">Configuring indexes via Data Annotations has been introduced in EF Core 5.0.</span></span>

## <a name="fluent-api"></a>[<span data-ttu-id="8a28a-110">Fluent API</span><span class="sxs-lookup"><span data-stu-id="8a28a-110">Fluent API</span></span>](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Index.cs?name=Index&highlight=4)]

***

> [!NOTE]
> <span data-ttu-id="8a28a-111">按照约定，会在每个属性中创建一个索引 (或一组用作外键的属性) 。</span><span class="sxs-lookup"><span data-stu-id="8a28a-111">By convention, an index is created in each property (or set of properties) that are used as a foreign key.</span></span>
>
> <span data-ttu-id="8a28a-112">EF Core 每个不同的属性集仅支持一个索引。</span><span class="sxs-lookup"><span data-stu-id="8a28a-112">EF Core only supports one index per distinct set of properties.</span></span> <span data-ttu-id="8a28a-113">如果对已定义索引的一组属性（按约定或以前的配置）配置索引，则会更改该索引的定义。</span><span class="sxs-lookup"><span data-stu-id="8a28a-113">If you configure an index on a set of properties that already has an index defined, either by convention or previous configuration, then you will be changing the definition of that index.</span></span> <span data-ttu-id="8a28a-114">如果要进一步配置由约定创建的索引，则此操作非常有用。</span><span class="sxs-lookup"><span data-stu-id="8a28a-114">This is useful if you want to further configure an index that was created by convention.</span></span>

## <a name="composite-index"></a><span data-ttu-id="8a28a-115">复合索引</span><span class="sxs-lookup"><span data-stu-id="8a28a-115">Composite index</span></span>

<span data-ttu-id="8a28a-116">索引还可以跨多个列：</span><span class="sxs-lookup"><span data-stu-id="8a28a-116">An index can also span more than one column:</span></span>

### <a name="data-annotations"></a>[<span data-ttu-id="8a28a-117">数据批注</span><span class="sxs-lookup"><span data-stu-id="8a28a-117">Data Annotations</span></span>](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/IndexComposite.cs?name=Composite&highlight=1)]

### <a name="fluent-api"></a>[<span data-ttu-id="8a28a-118">Fluent API</span><span class="sxs-lookup"><span data-stu-id="8a28a-118">Fluent API</span></span>](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexComposite.cs?name=Composite&highlight=4)]

<span data-ttu-id="8a28a-119">\*\*_</span><span class="sxs-lookup"><span data-stu-id="8a28a-119">\*\*_</span></span>

<span data-ttu-id="8a28a-120">跨多个列的索引，也称为 _composite 索引 \*，加速对索引列进行筛选的查询，以及仅筛选索引覆盖的 *第一* 列的查询。</span><span class="sxs-lookup"><span data-stu-id="8a28a-120">Indexes over multiple columns, also known as _composite indexes\*, speed up queries which filter on index's columns, but also queries which only filter on the *first* columns covered by the index.</span></span> <span data-ttu-id="8a28a-121">有关详细信息，请参阅 [性能文档](xref:core/performance/efficient-querying#use-indexes-properly) 。</span><span class="sxs-lookup"><span data-stu-id="8a28a-121">See the [performance docs](xref:core/performance/efficient-querying#use-indexes-properly) for more information.</span></span>

## <a name="index-uniqueness"></a><span data-ttu-id="8a28a-122">索引唯一性</span><span class="sxs-lookup"><span data-stu-id="8a28a-122">Index uniqueness</span></span>

<span data-ttu-id="8a28a-123">默认情况下，索引不唯一：对于索引的列集，允许多行具有相同的值 (s) 。</span><span class="sxs-lookup"><span data-stu-id="8a28a-123">By default, indexes aren't unique: multiple rows are allowed to have the same value(s) for the index's column set.</span></span> <span data-ttu-id="8a28a-124">可以使索引唯一，如下所示：</span><span class="sxs-lookup"><span data-stu-id="8a28a-124">You can make an index unique as follows:</span></span>

### <a name="data-annotations"></a>[<span data-ttu-id="8a28a-125">数据批注</span><span class="sxs-lookup"><span data-stu-id="8a28a-125">Data Annotations</span></span>](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/IndexUnique.cs?name=IndexUnique&highlight=1)]

### <a name="fluent-api"></a>[<span data-ttu-id="8a28a-126">Fluent API</span><span class="sxs-lookup"><span data-stu-id="8a28a-126">Fluent API</span></span>](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexUnique.cs?name=IndexUnique&highlight=5)]

***

<span data-ttu-id="8a28a-127">尝试为索引的列集插入多个具有相同值的实体将导致引发异常。</span><span class="sxs-lookup"><span data-stu-id="8a28a-127">Attempting to insert more than one entity with the same values for the index's column set will cause an exception to be thrown.</span></span>

## <a name="index-name"></a><span data-ttu-id="8a28a-128">索引名称</span><span class="sxs-lookup"><span data-stu-id="8a28a-128">Index name</span></span>

<span data-ttu-id="8a28a-129">按照约定，在关系数据库中创建的索引命名为 `IX_<type name>_<property name>` 。</span><span class="sxs-lookup"><span data-stu-id="8a28a-129">By convention, indexes created in a relational database are named `IX_<type name>_<property name>`.</span></span> <span data-ttu-id="8a28a-130">对于复合索引， `<property name>` 将成为以下划线分隔的属性名称列表。</span><span class="sxs-lookup"><span data-stu-id="8a28a-130">For composite indexes, `<property name>` becomes an underscore separated list of property names.</span></span>

<span data-ttu-id="8a28a-131">可以设置在数据库中创建的索引的名称：</span><span class="sxs-lookup"><span data-stu-id="8a28a-131">You can set the name of the index created in the database:</span></span>

### <a name="data-annotations"></a>[<span data-ttu-id="8a28a-132">数据批注</span><span class="sxs-lookup"><span data-stu-id="8a28a-132">Data Annotations</span></span>](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/IndexName.cs?name=IndexName&highlight=1)]

### <a name="fluent-api"></a>[<span data-ttu-id="8a28a-133">Fluent API</span><span class="sxs-lookup"><span data-stu-id="8a28a-133">Fluent API</span></span>](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexName.cs?name=IndexName&highlight=5)]

***

## <a name="index-filter"></a><span data-ttu-id="8a28a-134">索引筛选器</span><span class="sxs-lookup"><span data-stu-id="8a28a-134">Index filter</span></span>

<span data-ttu-id="8a28a-135">某些关系数据库允许您指定筛选索引或部分索引。</span><span class="sxs-lookup"><span data-stu-id="8a28a-135">Some relational databases allow you to specify a filtered or partial index.</span></span> <span data-ttu-id="8a28a-136">这使您可以只为列的值的一个子集编制索引，从而减少索引的大小并改善性能和磁盘空间的使用情况。</span><span class="sxs-lookup"><span data-stu-id="8a28a-136">This allows you to index only a subset of a column's values, reducing the index's size and improving both performance and disk space usage.</span></span> <span data-ttu-id="8a28a-137">有关 SQL Server 筛选索引的详细信息， [请参阅文档](/sql/relational-databases/indexes/create-filtered-indexes)。</span><span class="sxs-lookup"><span data-stu-id="8a28a-137">For more information on SQL Server filtered indexes, [see the documentation](/sql/relational-databases/indexes/create-filtered-indexes).</span></span>

<span data-ttu-id="8a28a-138">您可以使用熟知的 API 来指定索引的筛选器，作为 SQL 表达式提供：</span><span class="sxs-lookup"><span data-stu-id="8a28a-138">You can use the Fluent API to specify a filter on an index, provided as a SQL expression:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexFilter.cs?name=IndexFilter&highlight=5)]

<span data-ttu-id="8a28a-139">当使用 SQL Server 提供程序 EF 时 `'IS NOT NULL'` ，将为唯一索引中包含的所有可以为 null 的列添加筛选器。</span><span class="sxs-lookup"><span data-stu-id="8a28a-139">When using the SQL Server provider EF adds an `'IS NOT NULL'` filter for all nullable columns that are part of a unique index.</span></span> <span data-ttu-id="8a28a-140">若要重写此约定，可以提供一个 `null` 值。</span><span class="sxs-lookup"><span data-stu-id="8a28a-140">To override this convention you can supply a `null` value.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexNoFilter.cs?name=IndexNoFilter&highlight=6)]

## <a name="included-columns"></a><span data-ttu-id="8a28a-141">包含列</span><span class="sxs-lookup"><span data-stu-id="8a28a-141">Included columns</span></span>

<span data-ttu-id="8a28a-142">某些关系数据库允许配置一组列，这些列包含在索引中，但不是其 "键" 的一部分。</span><span class="sxs-lookup"><span data-stu-id="8a28a-142">Some relational databases allow you to configure a set of columns which get included in the index, but aren't part of its "key".</span></span> <span data-ttu-id="8a28a-143">当查询中的所有列都作为键列或非键列包含在索引中时，这可以显著提高查询性能，因为表本身无需访问。</span><span class="sxs-lookup"><span data-stu-id="8a28a-143">This can significantly improve query performance when all columns in the query are included in the index either as key or nonkey columns, as the table itself doesn't need to be accessed.</span></span> <span data-ttu-id="8a28a-144">有关 SQL Server 包含列的详细信息， [请参阅文档](/sql/relational-databases/indexes/create-indexes-with-included-columns)。</span><span class="sxs-lookup"><span data-stu-id="8a28a-144">For more information on SQL Server included columns, [see the documentation](/sql/relational-databases/indexes/create-indexes-with-included-columns).</span></span>

<span data-ttu-id="8a28a-145">在下面的示例中， `Url` 列是索引键的一部分，因此对该列的任何查询筛选都可以使用索引。</span><span class="sxs-lookup"><span data-stu-id="8a28a-145">In the following example, the `Url` column is part of the index key, so any query filtering on that column can use the index.</span></span> <span data-ttu-id="8a28a-146">但此外，仅访问 `Title` 和列的查询 `PublishedOn` 将不需要访问表并更有效地运行：</span><span class="sxs-lookup"><span data-stu-id="8a28a-146">But in addition, queries accessing only the `Title` and `PublishedOn` columns will not need to access the table and will run more efficiently:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IndexInclude.cs?name=IndexInclude&highlight=5-9)]
