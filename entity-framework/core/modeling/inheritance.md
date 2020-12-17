---
title: 继承-EF Core
description: 如何使用 Entity Framework Core 配置实体类型继承
author: AndriySvyryd
ms.date: 10/01/2020
uid: core/modeling/inheritance
ms.openlocfilehash: 11bd653a53767aa732790b1222da1beff8ad26a9
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97635752"
---
# <a name="inheritance"></a><span data-ttu-id="9599c-103">继承</span><span class="sxs-lookup"><span data-stu-id="9599c-103">Inheritance</span></span>

<span data-ttu-id="9599c-104">EF 可以将 .NET 类型层次结构映射到数据库。</span><span class="sxs-lookup"><span data-stu-id="9599c-104">EF can map a .NET type hierarchy to a database.</span></span> <span data-ttu-id="9599c-105">这使你可以像平常一样使用基类型和派生类型在代码中编写 .NET 实体，并让 EF 无缝创建适当的数据库架构、发出查询等。类型层次结构的映射方式的实际详细信息与提供程序相关;本页介绍关系数据库上下文中的继承支持。</span><span class="sxs-lookup"><span data-stu-id="9599c-105">This allows you to write your .NET entities in code as usual, using base and derived types, and have EF seamlessly create the appropriate database schema, issue queries, etc. The actual details of how a type hierarchy is mapped are provider-dependent; this page describes inheritance support in the context of a relational database.</span></span>

## <a name="entity-type-hierarchy-mapping"></a><span data-ttu-id="9599c-106">实体类型层次结构映射</span><span class="sxs-lookup"><span data-stu-id="9599c-106">Entity type hierarchy mapping</span></span>

<span data-ttu-id="9599c-107">按照约定，EF 不会自动扫描基类型或派生类型;这意味着，如果要映射层次结构中的 CLR 类型，则必须在模型上显式指定该类型。</span><span class="sxs-lookup"><span data-stu-id="9599c-107">By convention, EF will not automatically scan for base or derived types; this means that if you want a CLR type in your hierarchy to be mapped, you must explicitly specify that type on your model.</span></span> <span data-ttu-id="9599c-108">例如，仅指定层次结构的基类型将不会导致 EF Core 隐式包含其所有子类型。</span><span class="sxs-lookup"><span data-stu-id="9599c-108">For example, specifying only the base type of a hierarchy will not cause EF Core to implicitly include all of its sub-types.</span></span>

<span data-ttu-id="9599c-109">下面的示例为及其子类公开了 DbSet `Blog` `RssBlog` 。</span><span class="sxs-lookup"><span data-stu-id="9599c-109">The following sample exposes a DbSet for `Blog` and its subclass `RssBlog`.</span></span> <span data-ttu-id="9599c-110">如果 `Blog` 有任何其他子类，则不会将其包含在模型中。</span><span class="sxs-lookup"><span data-stu-id="9599c-110">If `Blog` has any other subclass, it will not be included in the model.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/InheritanceDbSets.cs?name=InheritanceDbSets&highlight=3-4)]

> [!NOTE]
> <span data-ttu-id="9599c-111">使用 TPH 映射时，数据库列会根据需要自动进行为 null。</span><span class="sxs-lookup"><span data-stu-id="9599c-111">Database columns are automatically made nullable as necessary when using TPH mapping.</span></span> <span data-ttu-id="9599c-112">例如， `RssUrl` 列可为 null，因为正则 `Blog` 实例不具有该属性。</span><span class="sxs-lookup"><span data-stu-id="9599c-112">For example, the `RssUrl` column is nullable because regular `Blog` instances do not have that property.</span></span>

<span data-ttu-id="9599c-113">如果你不希望 `DbSet` 为层次结构中的一个或多个实体公开，则还可以使用熟知的 API 来确保它们包含在模型中。</span><span class="sxs-lookup"><span data-stu-id="9599c-113">If you don't want to expose a `DbSet` for one or more entities in the hierarchy, you can also use the Fluent API to ensure they are included in the model.</span></span>

> [!TIP]
> <span data-ttu-id="9599c-114">如果不依赖约定，可以使用显式指定基类型 `HasBaseType` 。</span><span class="sxs-lookup"><span data-stu-id="9599c-114">If you don't rely on conventions, you can specify the base type explicitly using `HasBaseType`.</span></span> <span data-ttu-id="9599c-115">你还可以使用 `.HasBaseType((Type)null)` 从层次结构中删除实体类型。</span><span class="sxs-lookup"><span data-stu-id="9599c-115">You can also use `.HasBaseType((Type)null)` to remove an entity type from the hierarchy.</span></span>

## <a name="table-per-hierarchy-and-discriminator-configuration"></a><span data-ttu-id="9599c-116">每个层次结构一个表和鉴别器配置</span><span class="sxs-lookup"><span data-stu-id="9599c-116">Table-per-hierarchy and discriminator configuration</span></span>

<span data-ttu-id="9599c-117">默认情况下，EF 使用 *每个层次结构一个表* (TPH) 模式映射继承。</span><span class="sxs-lookup"><span data-stu-id="9599c-117">By default, EF maps the inheritance using the *table-per-hierarchy* (TPH) pattern.</span></span> <span data-ttu-id="9599c-118">TPH 使用单个表来存储层次结构中所有类型的数据，而鉴别器列用于标识每行所表示的类型。</span><span class="sxs-lookup"><span data-stu-id="9599c-118">TPH uses a single table to store the data for all types in the hierarchy, and a discriminator column is used to identify which type each row represents.</span></span>

<span data-ttu-id="9599c-119">上述模型映射到下面的数据库架构 (请注意隐式创建的 `Discriminator` 列，该列标识 `Blog` 每个行中存储) 的类型。</span><span class="sxs-lookup"><span data-stu-id="9599c-119">The model above is mapped to the following database schema (note the implicitly created `Discriminator` column, which identifies which type of `Blog` is stored in each row).</span></span>

![使用 "每个层次结构一个表" 模式查询博客实体层次结构的结果的屏幕截图](_static/inheritance-tph-data.png)

<span data-ttu-id="9599c-121">您可以配置鉴别器列的名称和类型以及用于标识层次结构中的每种类型的值：</span><span class="sxs-lookup"><span data-stu-id="9599c-121">You can configure the name and type of the discriminator column and the values that are used to identify each type in the hierarchy:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DiscriminatorConfiguration.cs?name=DiscriminatorConfiguration&highlight=4-6)]

<span data-ttu-id="9599c-122">在上述示例中，EF 在层次结构的基实体上将鉴别器隐式添加为 [影子属性](xref:core/modeling/shadow-properties) 。</span><span class="sxs-lookup"><span data-stu-id="9599c-122">In the examples above, EF added the discriminator implicitly as a [shadow property](xref:core/modeling/shadow-properties) on the base entity of the hierarchy.</span></span> <span data-ttu-id="9599c-123">此属性可以配置为类似于任何其他属性：</span><span class="sxs-lookup"><span data-stu-id="9599c-123">This property can be configured like any other:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DiscriminatorPropertyConfiguration.cs?name=DiscriminatorPropertyConfiguration&highlight=4-5)]

<span data-ttu-id="9599c-124">最后，鉴别器还可以映射到实体中的常规 .NET 属性：</span><span class="sxs-lookup"><span data-stu-id="9599c-124">Finally, the discriminator can also be mapped to a regular .NET property in your entity:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/NonShadowDiscriminator.cs?name=NonShadowDiscriminator&highlight=4)]

<span data-ttu-id="9599c-125">查询使用 TPH 模式的派生实体时 EF Core 会在查询中的鉴别器列上添加谓词。</span><span class="sxs-lookup"><span data-stu-id="9599c-125">When querying for derived entities, which use the TPH pattern, EF Core adds a predicate over discriminator column in the query.</span></span> <span data-ttu-id="9599c-126">此筛选器可确保不会为不在结果中的基类型或同级类型获取任何其他行。</span><span class="sxs-lookup"><span data-stu-id="9599c-126">This filter makes sure that we don't get any additional rows for base types or sibling types not in the result.</span></span> <span data-ttu-id="9599c-127">对于基实体类型，将跳过此筛选器谓词，因为查询基实体将获取层次结构中的所有实体的结果。</span><span class="sxs-lookup"><span data-stu-id="9599c-127">This filter predicate is skipped for the base entity type since querying for the base entity will get results for all the entities in the hierarchy.</span></span> <span data-ttu-id="9599c-128">当从查询中具体化结果时，如果跨一个不映射到模型中任何实体类型的鉴别器值，我们会引发异常，因为我们不知道如何具体化结果。</span><span class="sxs-lookup"><span data-stu-id="9599c-128">When materializing results from a query, if we come across a discriminator value, which isn't mapped to any entity type in the model, we throw an exception since we don't know how to materialize the results.</span></span> <span data-ttu-id="9599c-129">仅当数据库包含的行的鉴别器值未在 EF 模型中映射时，才会出现此错误。</span><span class="sxs-lookup"><span data-stu-id="9599c-129">This error only occurs if your database contains rows with discriminator values, which aren't mapped in the EF model.</span></span> <span data-ttu-id="9599c-130">如果有这样的数据，则可以将 EF Core 模型中的鉴别器映射标记为 "不完整"，以指示应始终添加筛选器谓词以查询层次结构中的任何类型。</span><span class="sxs-lookup"><span data-stu-id="9599c-130">If you have such data, then you can mark the discriminator mapping in EF Core model as incomplete to indicate that we should always add filter predicate for querying any type in the hierarchy.</span></span> <span data-ttu-id="9599c-131">`IsComplete(false)` 鉴别器配置上的调用将映射标记为不完整。</span><span class="sxs-lookup"><span data-stu-id="9599c-131">`IsComplete(false)` call on the discriminator configuration marks the mapping to be incomplete.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DiscriminatorMappingIncomplete.cs?name=DiscriminatorMappingIncomplete&highlight=5)]

### <a name="shared-columns"></a><span data-ttu-id="9599c-132">共享列</span><span class="sxs-lookup"><span data-stu-id="9599c-132">Shared columns</span></span>

<span data-ttu-id="9599c-133">默认情况下，当层次结构中的两个同级实体类型具有相同的属性时，它们将映射到两个单独的列。</span><span class="sxs-lookup"><span data-stu-id="9599c-133">By default, when two sibling entity types in the hierarchy have a property with the same name, they will be mapped to two separate columns.</span></span> <span data-ttu-id="9599c-134">但是，如果它们的类型相同，则可以映射到相同的数据库列：</span><span class="sxs-lookup"><span data-stu-id="9599c-134">However, if their type is identical they can be mapped to the same database column:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/SharedTPHColumns.cs?name=SharedTPHColumns&highlight=9,13)]

## <a name="table-per-type-configuration"></a><span data-ttu-id="9599c-135">每种类型一个表配置</span><span class="sxs-lookup"><span data-stu-id="9599c-135">Table-per-type configuration</span></span>

> [!NOTE]
> <span data-ttu-id="9599c-136">EF Core 5.0 中引入了每种类型一个表 (TPT) 功能。</span><span class="sxs-lookup"><span data-stu-id="9599c-136">The table-per-type (TPT) feature was introduced in EF Core 5.0.</span></span> <span data-ttu-id="9599c-137">EF6 支持每个具体的表类型 (TPC) ，但 EF Core 尚不支持。</span><span class="sxs-lookup"><span data-stu-id="9599c-137">Table-per-concrete-type (TPC) is supported by EF6, but is not yet supported by EF Core.</span></span>

<span data-ttu-id="9599c-138">在 TPT 映射模式下，所有类型都映射到各个表。</span><span class="sxs-lookup"><span data-stu-id="9599c-138">In the TPT mapping pattern, all the types are mapped to individual tables.</span></span> <span data-ttu-id="9599c-139">仅属于某个基类型或派生类型的属性存储在映射到该类型的一个表中。</span><span class="sxs-lookup"><span data-stu-id="9599c-139">Properties that belong solely to a base type or derived type are stored in a table that maps to that type.</span></span> <span data-ttu-id="9599c-140">映射到派生类型的表还存储将派生表与基表联接的外键。</span><span class="sxs-lookup"><span data-stu-id="9599c-140">Tables that map to derived types also store a foreign key that joins the derived table with the base table.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/TPTConfiguration.cs?name=TPTConfiguration)]

<span data-ttu-id="9599c-141">EF 将为上述模型创建下列数据库架构。</span><span class="sxs-lookup"><span data-stu-id="9599c-141">EF will create the following database schema for the model above.</span></span>

```sql
CREATE TABLE [Blogs] (
    [BlogId] int NOT NULL IDENTITY,
    [Url] nvarchar(max) NULL,
    CONSTRAINT [PK_Blogs] PRIMARY KEY ([BlogId])
);

CREATE TABLE [RssBlogs] (
    [BlogId] int NOT NULL,
    [RssUrl] nvarchar(max) NULL,
    CONSTRAINT [PK_RssBlogs] PRIMARY KEY ([BlogId]),
    CONSTRAINT [FK_RssBlogs_Blogs_BlogId] FOREIGN KEY ([BlogId]) REFERENCES [Blogs] ([BlogId]) ON DELETE NO ACTION
);
```

> [!NOTE]
> <span data-ttu-id="9599c-142">如果重命名了 primary key 约束，则新名称将应用于映射到层次结构的所有表，以后 EF 版本将允许在解决 [问题 19970](https://github.com/dotnet/efcore/issues/19970) 时为特定表重命名约束。</span><span class="sxs-lookup"><span data-stu-id="9599c-142">If the primary key constraint is renamed the new name will be applied to all the tables mapped to the hierarchy, future EF versions will allow renaming the constraint only for a particular table when [issue 19970](https://github.com/dotnet/efcore/issues/19970) is fixed.</span></span>

<span data-ttu-id="9599c-143">如果要使用批量配置，可以通过调用来检索特定表的列名称 <xref:Microsoft.EntityFrameworkCore.RelationalPropertyExtensions.GetColumnName(Microsoft.EntityFrameworkCore.Metadata.IProperty,Microsoft.EntityFrameworkCore.Metadata.StoreObjectIdentifier@)> 。</span><span class="sxs-lookup"><span data-stu-id="9599c-143">If you are employing bulk configuration you can retrieve the column name for a specific table by calling <xref:Microsoft.EntityFrameworkCore.RelationalPropertyExtensions.GetColumnName(Microsoft.EntityFrameworkCore.Metadata.IProperty,Microsoft.EntityFrameworkCore.Metadata.StoreObjectIdentifier@)>.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/TPTConfiguration.cs?name=Metadata&highlight=10)]

> [!WARNING]
> <span data-ttu-id="9599c-144">在许多情况下，与 TPH 相比，TPT 显示的性能差。</span><span class="sxs-lookup"><span data-stu-id="9599c-144">In many cases, TPT shows inferior performance when compared to TPH.</span></span> <span data-ttu-id="9599c-145">[有关详细信息，请参阅性能文档](xref:core/performance/modeling-for-performance#inheritance-mapping)。</span><span class="sxs-lookup"><span data-stu-id="9599c-145">[See the performance docs for more information](xref:core/performance/modeling-for-performance#inheritance-mapping).</span></span>
