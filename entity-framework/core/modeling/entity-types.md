---
title: 实体类型-EF Core
description: 如何使用 Entity Framework Core 配置和映射实体类型
author: roji
ms.date: 10/06/2020
uid: core/modeling/entity-types
ms.openlocfilehash: 9d86b959b5e0360df6d782d8d1c1c2f9393fdf8b
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003492"
---
# <a name="entity-types"></a><span data-ttu-id="c9250-103">实体类型</span><span class="sxs-lookup"><span data-stu-id="c9250-103">Entity Types</span></span>

<span data-ttu-id="c9250-104">在上下文中包含一种类型的 DbSet，这意味着它包含在 EF Core 的模型中;我们通常将此类类型称为 *实体*。</span><span class="sxs-lookup"><span data-stu-id="c9250-104">Including a DbSet of a type on your context means that it is included in EF Core's model; we usually refer to such a type as an *entity*.</span></span> <span data-ttu-id="c9250-105">EF Core 可以从/向数据库中读取和写入实体实例，如果使用的是关系数据库，EF Core 可以通过迁移为实体创建表。</span><span class="sxs-lookup"><span data-stu-id="c9250-105">EF Core can read and write entity instances from/to the database, and if you're using a relational database, EF Core can create tables for your entities via migrations.</span></span>

## <a name="including-types-in-the-model"></a><span data-ttu-id="c9250-106">在模型中包含类型</span><span class="sxs-lookup"><span data-stu-id="c9250-106">Including types in the model</span></span>

<span data-ttu-id="c9250-107">按照约定，在上下文中的 DbSet 属性中公开的类型作为实体包含在模型中。</span><span class="sxs-lookup"><span data-stu-id="c9250-107">By convention, types that are exposed in DbSet properties on your context are included in the model as entities.</span></span> <span data-ttu-id="c9250-108">还包括方法中指定的实体类型 `OnModelCreating` ，就像通过递归方式浏览其他发现的实体类型的导航属性找到的任何类型一样。</span><span class="sxs-lookup"><span data-stu-id="c9250-108">Entity types that are specified in the `OnModelCreating` method are also included, as are any types that are found by recursively exploring the navigation properties of other discovered entity types.</span></span>

<span data-ttu-id="c9250-109">在下面的代码示例中，包含了所有类型：</span><span class="sxs-lookup"><span data-stu-id="c9250-109">In the code sample below, all types are included:</span></span>

* <span data-ttu-id="c9250-110">`Blog` 包含，因为它在上下文的 DbSet 属性中公开。</span><span class="sxs-lookup"><span data-stu-id="c9250-110">`Blog` is included because it's exposed in a DbSet property on the context.</span></span>
* <span data-ttu-id="c9250-111">`Post` 包含，因为它通过 `Blog.Posts` 导航属性发现。</span><span class="sxs-lookup"><span data-stu-id="c9250-111">`Post` is included because it's discovered via the `Blog.Posts` navigation property.</span></span>
* <span data-ttu-id="c9250-112">`AuditEntry` 因为它是在中指定的 `OnModelCreating` 。</span><span class="sxs-lookup"><span data-stu-id="c9250-112">`AuditEntry` because it is specified in `OnModelCreating`.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/EntityTypes.cs?name=EntityTypes&highlight=3,7,16)]

## <a name="excluding-types-from-the-model"></a><span data-ttu-id="c9250-113">从模型中排除类型</span><span class="sxs-lookup"><span data-stu-id="c9250-113">Excluding types from the model</span></span>

<span data-ttu-id="c9250-114">如果您不希望在模型中包含某一类型，则可以排除它：</span><span class="sxs-lookup"><span data-stu-id="c9250-114">If you don't want a type to be included in the model, you can exclude it:</span></span>

### <a name="data-annotations"></a>[<span data-ttu-id="c9250-115">数据批注</span><span class="sxs-lookup"><span data-stu-id="c9250-115">Data Annotations</span></span>](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/IgnoreType.cs?name=IgnoreType&highlight=1)]

### <a name="fluent-api"></a>[<span data-ttu-id="c9250-116">Fluent API</span><span class="sxs-lookup"><span data-stu-id="c9250-116">Fluent API</span></span>](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/IgnoreType.cs?name=IgnoreType&highlight=3)]

***

### <a name="excluding-from-migrations"></a><span data-ttu-id="c9250-117">从迁移中排除</span><span class="sxs-lookup"><span data-stu-id="c9250-117">Excluding from migrations</span></span>

> [!NOTE]
> <span data-ttu-id="c9250-118">EF Core 5.0 中引入了从迁移中排除表的功能。</span><span class="sxs-lookup"><span data-stu-id="c9250-118">The ability to exclude tables from migrations was introduced in EF Core 5.0.</span></span>

<span data-ttu-id="c9250-119">有时在多个类型中映射相同的实体类型会很有用 `DbContext` 。</span><span class="sxs-lookup"><span data-stu-id="c9250-119">It is sometimes useful to have the same entity type mapped in multiple `DbContext` types.</span></span> <span data-ttu-id="c9250-120">当使用 [绑定上下文](https://www.martinfowler.com/bliki/BoundedContext.html)时，尤其是对于 `DbContext` 每个边界上下文都有不同类型的情况。</span><span class="sxs-lookup"><span data-stu-id="c9250-120">This is especially true when using [bounded contexts](https://www.martinfowler.com/bliki/BoundedContext.html), for which it is common to have a different `DbContext` type for each bounded context.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/TableExcludeFromMigrations.cs?name=TableExcludeFromMigrations&highlight=4)]

<span data-ttu-id="c9250-121">此配置迁移不会创建 `AspNetUsers` 该表，但 `IdentityUser` 仍包含在模型中，并且可正常使用。</span><span class="sxs-lookup"><span data-stu-id="c9250-121">With this configuration migrations will not create the `AspNetUsers` table, but `IdentityUser` is still included in the model and can be used normally.</span></span>

<span data-ttu-id="c9250-122">如果需要再次使用迁移来管理表，则应创建不包括的新迁移 `AspNetUsers` 。</span><span class="sxs-lookup"><span data-stu-id="c9250-122">If you need to start managing the table using migrations again then a new migration should be created where `AspNetUsers` is not excluded.</span></span> <span data-ttu-id="c9250-123">下一次迁移将包含对表所做的任何更改。</span><span class="sxs-lookup"><span data-stu-id="c9250-123">The next migration will now contain any changes made to the table.</span></span>

## <a name="table-name"></a><span data-ttu-id="c9250-124">表名称</span><span class="sxs-lookup"><span data-stu-id="c9250-124">Table name</span></span>

<span data-ttu-id="c9250-125">按照约定，每个实体类型将设置为映射到与公开实体的 DbSet 属性同名的数据库表。</span><span class="sxs-lookup"><span data-stu-id="c9250-125">By convention, each entity type will be set up to map to a database table with the same name as the DbSet property that exposes the entity.</span></span> <span data-ttu-id="c9250-126">如果给定实体不存在 DbSet，则使用类名称。</span><span class="sxs-lookup"><span data-stu-id="c9250-126">If no DbSet exists for the given entity, the class name is used.</span></span>

<span data-ttu-id="c9250-127">可以手动配置表名：</span><span class="sxs-lookup"><span data-stu-id="c9250-127">You can manually configure the table name:</span></span>

### <a name="data-annotations"></a>[<span data-ttu-id="c9250-128">数据批注</span><span class="sxs-lookup"><span data-stu-id="c9250-128">Data Annotations</span></span>](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/TableName.cs?Name=TableName&highlight=1)]

### <a name="fluent-api"></a>[<span data-ttu-id="c9250-129">Fluent API</span><span class="sxs-lookup"><span data-stu-id="c9250-129">Fluent API</span></span>](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/TableName.cs?Name=TableName&highlight=3-4)]

<span data-ttu-id="c9250-130">\*\*_</span><span class="sxs-lookup"><span data-stu-id="c9250-130">\*\*_</span></span>

## <a name="table-schema"></a><span data-ttu-id="c9250-131">表架构</span><span class="sxs-lookup"><span data-stu-id="c9250-131">Table schema</span></span>

<span data-ttu-id="c9250-132">使用关系数据库时，表按约定在数据库的默认架构中创建。</span><span class="sxs-lookup"><span data-stu-id="c9250-132">When using a relational database, tables are by convention created in your database's default schema.</span></span> <span data-ttu-id="c9250-133">例如，Microsoft SQL Server 将使用 `dbo` 架构 (SQLite 不支持) 的架构。</span><span class="sxs-lookup"><span data-stu-id="c9250-133">For example, Microsoft SQL Server will use the `dbo` schema (SQLite does not support schemas).</span></span>

<span data-ttu-id="c9250-134">你可以配置要在特定架构中创建的表，如下所示：</span><span class="sxs-lookup"><span data-stu-id="c9250-134">You can configure tables to be created in a specific schema as follows:</span></span>

### <a name="data-annotations"></a>[<span data-ttu-id="c9250-135">数据批注</span><span class="sxs-lookup"><span data-stu-id="c9250-135">Data Annotations</span></span>](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/TableNameAndSchema.cs?name=TableNameAndSchema&highlight=1)]

### <a name="fluent-api"></a>[<span data-ttu-id="c9250-136">Fluent API</span><span class="sxs-lookup"><span data-stu-id="c9250-136">Fluent API</span></span>](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/TableNameAndSchema.cs?name=TableNameAndSchema&highlight=3-4)]

<span data-ttu-id="c9250-137">_\*\*</span><span class="sxs-lookup"><span data-stu-id="c9250-137">_\*\*</span></span>

<span data-ttu-id="c9250-138">您还可以在模型级别定义 Fluent API 的默认架构，而不是为每个表指定架构：</span><span class="sxs-lookup"><span data-stu-id="c9250-138">Rather than specifying the schema for each table, you can also define the default schema at the model level with the fluent API:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DefaultSchema.cs?name=DefaultSchema&highlight=3)]

<span data-ttu-id="c9250-139">请注意，设置默认架构也会影响其他数据库对象，如序列。</span><span class="sxs-lookup"><span data-stu-id="c9250-139">Note that setting the default schema will also affect other database objects, such as sequences.</span></span>

## <a name="view-mapping"></a><span data-ttu-id="c9250-140">查看映射</span><span class="sxs-lookup"><span data-stu-id="c9250-140">View mapping</span></span>

<span data-ttu-id="c9250-141">实体类型可以使用熟知的 API 映射到数据库视图。</span><span class="sxs-lookup"><span data-stu-id="c9250-141">Entity types can be mapped to database views using the Fluent API.</span></span>

> [!Note]
> <span data-ttu-id="c9250-142">EF 假设数据库中已存在引用的视图，它不会在迁移中自动创建它。</span><span class="sxs-lookup"><span data-stu-id="c9250-142">EF will assume that the referenced view already exists in the database, it will not create it automatically in a migration.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ViewNameAndSchema.cs?name=ViewNameAndSchema&highlight=1)]

 <span data-ttu-id="c9250-143">映射到视图将删除默认表映射，但从 EF 5.0 开始，实体类型也可以显式映射到表。</span><span class="sxs-lookup"><span data-stu-id="c9250-143">Mapping to a view will remove the default table mapping, but starting with EF 5.0 the entity type can also be mapped to a table explicitly.</span></span> <span data-ttu-id="c9250-144">在这种情况下，查询映射将用于查询，表映射将用于更新。</span><span class="sxs-lookup"><span data-stu-id="c9250-144">In this case the query mapping will be used for queries and the table mapping will be used for updates.</span></span>

> [!TIP]
> <span data-ttu-id="c9250-145">若要测试使用内存中提供程序映射到视图的实体类型，请通过将它们映射到查询 `ToInMemoryQuery` 。</span><span class="sxs-lookup"><span data-stu-id="c9250-145">To test entity types mapped to views using the in-memory provider map them to a query via `ToInMemoryQuery`.</span></span> <span data-ttu-id="c9250-146">有关更多详细信息，请参阅使用此技术的可 [运行示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Testing/ItemsWebApi/) 。</span><span class="sxs-lookup"><span data-stu-id="c9250-146">See a [runnable sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Testing/ItemsWebApi/) using this technique for more details.</span></span>
