---
title: 访问跟踪的实体-EF Core
description: 使用 EntityEntry、DbContext 和 DbSet 访问跟踪的实体
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/entity-entries
ms.openlocfilehash: f385016aba61535f33e34c622dd43ce6dc823fc5
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129579"
---
# <a name="accessing-tracked-entities"></a><span data-ttu-id="83db8-103">访问跟踪的实体</span><span class="sxs-lookup"><span data-stu-id="83db8-103">Accessing Tracked Entities</span></span>

<span data-ttu-id="83db8-104">可使用四个主要 Api 来访问由跟踪的实体 <xref:Microsoft.EntityFrameworkCore.DbContext> ：</span><span class="sxs-lookup"><span data-stu-id="83db8-104">There are four main APIs for accessing entities tracked by a <xref:Microsoft.EntityFrameworkCore.DbContext>:</span></span>

- <span data-ttu-id="83db8-105"><xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType> 返回 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 给定实体实例的实例。</span><span class="sxs-lookup"><span data-stu-id="83db8-105"><xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType> returns an <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> instance for a given entity instance.</span></span>
- <span data-ttu-id="83db8-106"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%2A?displayProperty=nameWithType> 返回 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 所有已跟踪实体的实例，或返回给定类型的所有已跟踪实体的实例。</span><span class="sxs-lookup"><span data-stu-id="83db8-106"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%2A?displayProperty=nameWithType> returns <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> instances for all tracked entities, or for all tracked entities of a given type.</span></span>
- <span data-ttu-id="83db8-107"><xref:Microsoft.EntityFrameworkCore.DbContext.Find%2A?displayProperty=nameWithType>、 <xref:Microsoft.EntityFrameworkCore.DbContext.FindAsync%2A?displayProperty=nameWithType> 、 <xref:Microsoft.EntityFrameworkCore.DbSet%601.Find%2A?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.DbSet%601.FindAsync%2A?displayProperty=nameWithType> 按主键查找单个实体，首先在跟踪的实体中查找，然后在需要时查询数据库。</span><span class="sxs-lookup"><span data-stu-id="83db8-107"><xref:Microsoft.EntityFrameworkCore.DbContext.Find%2A?displayProperty=nameWithType>, <xref:Microsoft.EntityFrameworkCore.DbContext.FindAsync%2A?displayProperty=nameWithType>, <xref:Microsoft.EntityFrameworkCore.DbSet%601.Find%2A?displayProperty=nameWithType>, and <xref:Microsoft.EntityFrameworkCore.DbSet%601.FindAsync%2A?displayProperty=nameWithType> find a single entity by primary key, first looking in tracked entities, and then querying the database if needed.</span></span>
- <span data-ttu-id="83db8-108"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> 返回由 DbSet 表示的实体类型的实体 (不 EntityEntry) 实例的实际实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-108"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> returns actual entities (not EntityEntry instances) for entities of the entity type represented by the DbSet.</span></span>

<span data-ttu-id="83db8-109">下面的部分将对其中的每个进行更详细的介绍。</span><span class="sxs-lookup"><span data-stu-id="83db8-109">Each of these is described in more detail in the sections below.</span></span>

> [!TIP]
> <span data-ttu-id="83db8-110">本文档假设了解 EF Core 更改跟踪的实体状态和基础知识。</span><span class="sxs-lookup"><span data-stu-id="83db8-110">This document assumes that entity states and the basics of EF Core change tracking are understood.</span></span> <span data-ttu-id="83db8-111">有关这些主题的详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。</span><span class="sxs-lookup"><span data-stu-id="83db8-111">See [Change Tracking in EF Core](xref:core/change-tracking/index) for more information on these topics.</span></span>

> [!TIP]
> <span data-ttu-id="83db8-112">通过 [从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/AccessingTrackedEntities)，你可以运行并调试到本文档中的所有代码。</span><span class="sxs-lookup"><span data-stu-id="83db8-112">You can run and debug into all the code in this document by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/AccessingTrackedEntities).</span></span>

## <a name="using-dbcontextentry-and-entityentry-instances"></a><span data-ttu-id="83db8-113">使用 DbContext 和 EntityEntry 实例</span><span class="sxs-lookup"><span data-stu-id="83db8-113">Using DbContext.Entry and EntityEntry instances</span></span>

<span data-ttu-id="83db8-114">对于每个被跟踪的实体，Entity Framework Core (EF Core) 跟踪：</span><span class="sxs-lookup"><span data-stu-id="83db8-114">For each tracked entity, Entity Framework Core (EF Core) keeps track of:</span></span>

- <span data-ttu-id="83db8-115">实体的总体状态。</span><span class="sxs-lookup"><span data-stu-id="83db8-115">The overall state of the entity.</span></span> <span data-ttu-id="83db8-116">这是 `Unchanged` 、 `Modified` 、或中的一个， `Added` `Deleted` 有关详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。</span><span class="sxs-lookup"><span data-stu-id="83db8-116">This is one of `Unchanged`, `Modified`, `Added`, or `Deleted`; see [Change Tracking in EF Core](xref:core/change-tracking/index) for more information.</span></span>
- <span data-ttu-id="83db8-117">所跟踪的实体之间的关系。</span><span class="sxs-lookup"><span data-stu-id="83db8-117">The relationships between tracked entities.</span></span> <span data-ttu-id="83db8-118">例如，张贴内容所属的博客。</span><span class="sxs-lookup"><span data-stu-id="83db8-118">For example, the blog to which a post belongs.</span></span>
- <span data-ttu-id="83db8-119">属性的 "当前值"。</span><span class="sxs-lookup"><span data-stu-id="83db8-119">The "current values" of properties.</span></span>
- <span data-ttu-id="83db8-120">如果此信息可用，则为属性的 "原始值"。</span><span class="sxs-lookup"><span data-stu-id="83db8-120">The "original values" of properties, when this information is available.</span></span> <span data-ttu-id="83db8-121">原始值是从数据库中查询实体时存在的属性值。</span><span class="sxs-lookup"><span data-stu-id="83db8-121">Original values are the property values that existed when entity was queried from the database.</span></span>
- <span data-ttu-id="83db8-122">自查询后修改了哪些属性值。</span><span class="sxs-lookup"><span data-stu-id="83db8-122">Which property values have been modified since they were queried.</span></span>
- <span data-ttu-id="83db8-123">有关属性值的其他信息，例如，值是否是 [临时](xref:core/change-tracking/miscellaneous#temporary-values)的。</span><span class="sxs-lookup"><span data-stu-id="83db8-123">Other information about property values, such as whether or not the value is [temporary](xref:core/change-tracking/miscellaneous#temporary-values).</span></span>

<span data-ttu-id="83db8-124">传递实体实例将 <xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType> 导致为 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 给定实体提供对此信息的访问。</span><span class="sxs-lookup"><span data-stu-id="83db8-124">Passing an entity instance to <xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType> results in an <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> providing access to this information for the given entity.</span></span> <span data-ttu-id="83db8-125">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-125">For example:</span></span>

<!--
        using var context = new BlogsContext();

        var blog = context.Blogs.Single(e => e.Id == 1);
        var entityEntry = context.Entry(blog);

-->
[!code-csharp[Using_DbContext_Entry_and_EntityEntry_instances_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_DbContext_Entry_and_EntityEntry_instances_1)]

<span data-ttu-id="83db8-126">以下各节说明如何使用 EntityEntry 访问和操作实体状态以及实体的属性和导航状态。</span><span class="sxs-lookup"><span data-stu-id="83db8-126">The following sections show how to use an EntityEntry to access and manipulate entity state, as well as the state of the entity's properties and navigations.</span></span>

### <a name="working-with-the-entity"></a><span data-ttu-id="83db8-127">使用实体</span><span class="sxs-lookup"><span data-stu-id="83db8-127">Working with the entity</span></span>

<span data-ttu-id="83db8-128">最常见的用途 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 是访问实体的当前 <xref:Microsoft.EntityFrameworkCore.EntityState> 。</span><span class="sxs-lookup"><span data-stu-id="83db8-128">The most common use of <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> is to access the current <xref:Microsoft.EntityFrameworkCore.EntityState> of an entity.</span></span> <span data-ttu-id="83db8-129">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-129">For example:</span></span>

<!--
        var currentState = context.Entry(blog).State;
        if (currentState == EntityState.Unchanged)
        {
            context.Entry(blog).State = EntityState.Modified;
        }
-->
[!code-csharp[Work_with_the_entity_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_the_entity_1)]

<span data-ttu-id="83db8-130">条目方法还可用于尚未跟踪的实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-130">The Entry method can also be used on entities that are not yet tracked.</span></span> <span data-ttu-id="83db8-131">这不 _会开始跟踪实体_;实体的状态仍为 `Detatched` 。</span><span class="sxs-lookup"><span data-stu-id="83db8-131">This _does not start tracking the entity_; the state of the entity is still `Detatched`.</span></span> <span data-ttu-id="83db8-132">但是，随后可以使用返回的 EntityEntry 更改实体状态，此时将在给定状态下跟踪实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-132">However, the returned EntityEntry can then be used to change the entity state, at which point the entity will become tracked in the given state.</span></span> <span data-ttu-id="83db8-133">例如，以下代码将开始跟踪博客实例，如下所示 `Added` ：</span><span class="sxs-lookup"><span data-stu-id="83db8-133">For example, the following code will start tracking a Blog instance as `Added`:</span></span>

<!--
        var newBlog = new Blog();
        Debug.Assert(context.Entry(newBlog).State == EntityState.Detached);

        context.Entry(newBlog).State = EntityState.Added;
        Debug.Assert(context.Entry(newBlog).State == EntityState.Added);
-->
[!code-csharp[Work_with_the_entity_2](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_the_entity_2)]

> [!TIP]
> <span data-ttu-id="83db8-134">与在 EF6 中不同，设置单个实体的状态不会导致跟踪所有连接的实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-134">Unlike in EF6, setting the state of an individual entity will not cause all connected entities to be tracked.</span></span> <span data-ttu-id="83db8-135">这使得将此状态设置为较低级别操作的方式，而不是调用 `Add` 、 `Attach` 或，这会 `Update` 对整个实体图形执行操作。</span><span class="sxs-lookup"><span data-stu-id="83db8-135">This makes setting the state this way a lower-level operation than calling `Add`, `Attach`, or `Update`, which operate on an entire graph of entities.</span></span>

<span data-ttu-id="83db8-136">下表总结了使用 EntityEntry 来处理整个实体的方法：</span><span class="sxs-lookup"><span data-stu-id="83db8-136">The following table summarizes ways to use an EntityEntry to work with an entire entity:</span></span>

| <span data-ttu-id="83db8-137">EntityEntry 成员</span><span class="sxs-lookup"><span data-stu-id="83db8-137">EntityEntry member</span></span>                                                                                         | <span data-ttu-id="83db8-138">描述</span><span class="sxs-lookup"><span data-stu-id="83db8-138">Description</span></span>
|:-----------------------------------------------------------------------------------------------------------|----------------------
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.State?displayProperty=nameWithType>         | <span data-ttu-id="83db8-139">获取和设置 <xref:Microsoft.EntityFrameworkCore.EntityState> 实体的。</span><span class="sxs-lookup"><span data-stu-id="83db8-139">Gets and sets the <xref:Microsoft.EntityFrameworkCore.EntityState> of the entity.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Entity?displayProperty=nameWithType>        | <span data-ttu-id="83db8-140">获取实体实例。</span><span class="sxs-lookup"><span data-stu-id="83db8-140">Gets the entity instance.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Context?displayProperty=nameWithType>       | <span data-ttu-id="83db8-141"><xref:Microsoft.EntityFrameworkCore.DbContext>正在跟踪此实体的。</span><span class="sxs-lookup"><span data-stu-id="83db8-141">The <xref:Microsoft.EntityFrameworkCore.DbContext> that is tracking this entity.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Metadata?displayProperty=nameWithType>      | <span data-ttu-id="83db8-142"><xref:Microsoft.EntityFrameworkCore.Metadata.IEntityType> 实体类型的元数据。</span><span class="sxs-lookup"><span data-stu-id="83db8-142"><xref:Microsoft.EntityFrameworkCore.Metadata.IEntityType> metadata for the type of entity.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.IsKeySet?displayProperty=nameWithType>      | <span data-ttu-id="83db8-143">实体是否已设置其键值。</span><span class="sxs-lookup"><span data-stu-id="83db8-143">Whether or not the entity has had its key value set.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Reload?displayProperty=nameWithType>        | <span data-ttu-id="83db8-144">用从数据库中读取的值覆盖属性值。</span><span class="sxs-lookup"><span data-stu-id="83db8-144">Overwrites property values with values read from the database.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.DetectChanges?displayProperty=nameWithType> | <span data-ttu-id="83db8-145">仅强制检测此实体的更改;请参阅 [更改检测和通知](xref:core/change-tracking/change-detection)。</span><span class="sxs-lookup"><span data-stu-id="83db8-145">Forces detection of changes for this entity only; see [Change Detection and Notifications](xref:core/change-tracking/change-detection).</span></span>

### <a name="working-with-a-single-property"></a><span data-ttu-id="83db8-146">使用单个属性</span><span class="sxs-lookup"><span data-stu-id="83db8-146">Working with a single property</span></span>

<span data-ttu-id="83db8-147">的多个重载 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Property%2A?displayProperty=nameWithType> 允许访问有关实体的各个属性的信息。</span><span class="sxs-lookup"><span data-stu-id="83db8-147">Several overloads of <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Property%2A?displayProperty=nameWithType> allow access to information about an individual property of an entity.</span></span> <span data-ttu-id="83db8-148">例如，使用强类型的、熟知的 API：</span><span class="sxs-lookup"><span data-stu-id="83db8-148">For example, using a strongly-typed, fluent-like API:</span></span>

<!--
            PropertyEntry<Blog, string> propertyEntry = context.Entry(blog).Property(e => e.Name);
-->
[!code-csharp[Work_with_a_single_property_1a](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_property_1a)]

<span data-ttu-id="83db8-149">属性名称可以改为作为字符串传递。</span><span class="sxs-lookup"><span data-stu-id="83db8-149">The property name can instead be passed as a string.</span></span> <span data-ttu-id="83db8-150">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-150">For example:</span></span>

<!--
            PropertyEntry<Blog, string> propertyEntry = context.Entry(blog).Property<string>("Name");
-->
[!code-csharp[Work_with_a_single_property_1b](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_property_1b)]

<span data-ttu-id="83db8-151">然后， <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602> 可以使用返回的来访问有关属性的信息。</span><span class="sxs-lookup"><span data-stu-id="83db8-151">The returned <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602> can then be used to access information about the property.</span></span> <span data-ttu-id="83db8-152">例如，它可用于获取和设置此实体上的属性的当前值：</span><span class="sxs-lookup"><span data-stu-id="83db8-152">For example, it can be used to get and set the current value of the property on this entity:</span></span>

<!--
            string currentValue = context.Entry(blog).Property(e => e.Name).CurrentValue;
            context.Entry(blog).Property(e => e.Name).CurrentValue = "1unicorn2";
-->
[!code-csharp[Work_with_a_single_property_1d](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_property_1d)]

<span data-ttu-id="83db8-153">以上使用的两个属性方法都返回强类型的泛型 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602> 实例。</span><span class="sxs-lookup"><span data-stu-id="83db8-153">Both of the Property methods used above return a strongly-typed generic <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602> instance.</span></span> <span data-ttu-id="83db8-154">使用此泛型类型是首选方法，因为它允许无需 [装箱值类型](/dotnet/csharp/programming-guide/types/boxing-and-unboxing)即可访问属性值。</span><span class="sxs-lookup"><span data-stu-id="83db8-154">Using this generic type is preferred because it allows access to property values without [boxing value types](/dotnet/csharp/programming-guide/types/boxing-and-unboxing).</span></span> <span data-ttu-id="83db8-155">但是，如果实体或属性的类型在编译时未知，则 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry> 可以改为获取非泛型：</span><span class="sxs-lookup"><span data-stu-id="83db8-155">However, if the type of entity or property is not known at compile-time, then a non-generic <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry> can be obtained instead:</span></span>

<!--
            var propertyEntry = context.Entry(blog).Property("Name");
-->
[!code-csharp[Work_with_a_single_property_1c](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_property_1c)]

<span data-ttu-id="83db8-156">这样，无论属性的类型如何，都可以访问任何属性的属性信息，但需支付值类型的费用。</span><span class="sxs-lookup"><span data-stu-id="83db8-156">This allows access to property information for any property regardless of its type, at the expense of boxing value types.</span></span> <span data-ttu-id="83db8-157">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-157">For example:</span></span>

<!--
            object blog = context.Blogs.Single(e => e.Id == 1);

            object currentValue = context.Entry(blog).Property("Name").CurrentValue;
            context.Entry(blog).Property("Name").CurrentValue = "1unicorn2";
-->
[!code-csharp[Work_with_a_single_property_1e](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_property_1e)]

<span data-ttu-id="83db8-158">下表汇总了由 PropertyEntry 公开的属性信息：</span><span class="sxs-lookup"><span data-stu-id="83db8-158">The following table summarizes property information exposed by PropertyEntry:</span></span>

| <span data-ttu-id="83db8-159">PropertyEntry 成员</span><span class="sxs-lookup"><span data-stu-id="83db8-159">PropertyEntry member</span></span>                               | <span data-ttu-id="83db8-160">描述</span><span class="sxs-lookup"><span data-stu-id="83db8-160">Description</span></span>
|:-------------------------------------------------|----------------------
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602.CurrentValue?displayProperty=nameWithType>  | <span data-ttu-id="83db8-161">获取和设置属性的当前值。</span><span class="sxs-lookup"><span data-stu-id="83db8-161">Gets and sets the current value of the property.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602.OriginalValue?displayProperty=nameWithType> | <span data-ttu-id="83db8-162">获取并设置属性的原始值（如果可用）。</span><span class="sxs-lookup"><span data-stu-id="83db8-162">Gets and sets the original value of the property, if available.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602.EntityEntry?displayProperty=nameWithType>   | <span data-ttu-id="83db8-163">对实体的的反向引用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 。</span><span class="sxs-lookup"><span data-stu-id="83db8-163">A back reference to the <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> for the entity.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.Metadata?displayProperty=nameWithType>          | <span data-ttu-id="83db8-164"><xref:Microsoft.EntityFrameworkCore.Metadata.IProperty> 属性的元数据。</span><span class="sxs-lookup"><span data-stu-id="83db8-164"><xref:Microsoft.EntityFrameworkCore.Metadata.IProperty> metadata for the property.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsModified?displayProperty=nameWithType>        | <span data-ttu-id="83db8-165">指示此属性是否被标记为已修改，并允许更改此状态。</span><span class="sxs-lookup"><span data-stu-id="83db8-165">Indicates whether this property is marked as modified, and allows this state to be changed.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsTemporary?displayProperty=nameWithType>       | <span data-ttu-id="83db8-166">指示此属性是否标记为 [临时](xref:core/change-tracking/miscellaneous#temporary-values#temporary-values)，并允许更改此状态。</span><span class="sxs-lookup"><span data-stu-id="83db8-166">Indicates whether this property is marked as [temporary](xref:core/change-tracking/miscellaneous#temporary-values#temporary-values), and allows this state to be changed.</span></span>

<span data-ttu-id="83db8-167">说明：</span><span class="sxs-lookup"><span data-stu-id="83db8-167">Notes:</span></span>

- <span data-ttu-id="83db8-168">属性的原始值是从数据库中查询实体时属性具有的值。</span><span class="sxs-lookup"><span data-stu-id="83db8-168">The original value of a property is the value that the property had when the entity was queried from the database.</span></span> <span data-ttu-id="83db8-169">但是，如果实体已断开连接，然后显式附加到另一个 DbContext （例如，使用或），则原始值不可用 `Attach` `Update` 。</span><span class="sxs-lookup"><span data-stu-id="83db8-169">However, original values are not available if the entity was disconnected and then explicitly attached to another DbContext, for example with `Attach` or `Update`.</span></span> <span data-ttu-id="83db8-170">在这种情况下，返回的原始值将与当前的值相同。</span><span class="sxs-lookup"><span data-stu-id="83db8-170">In this case, the original value returned will be the same as the current value.</span></span>
- <span data-ttu-id="83db8-171"><xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 只会更新标记为已修改的属性。</span><span class="sxs-lookup"><span data-stu-id="83db8-171"><xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> will only update properties marked as modified.</span></span> <span data-ttu-id="83db8-172"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsModified>如果设置为 true，则强制 EF Core 更新给定属性值，或将其设置为 false 以防止 EF Core 更新属性值。</span><span class="sxs-lookup"><span data-stu-id="83db8-172">Set <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsModified> to true to force EF Core to update a given property value, or set it to false to prevent EF Core from updating the property value.</span></span>
- <span data-ttu-id="83db8-173">[临时值](xref:core/change-tracking/miscellaneous) 通常由 EF Core [值生成器](xref:core/modeling/generated-properties)生成。</span><span class="sxs-lookup"><span data-stu-id="83db8-173">[Temporary values](xref:core/change-tracking/miscellaneous) are typically generated by EF Core [value generators](xref:core/modeling/generated-properties).</span></span> <span data-ttu-id="83db8-174">设置属性的当前值会将临时值替换为给定的值，并将该属性标记为不是临时的。</span><span class="sxs-lookup"><span data-stu-id="83db8-174">Setting the current value of a property will replace the temporary value with the given value and mark the property as not temporary.</span></span> <span data-ttu-id="83db8-175"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsTemporary>如果设置为 true，则强制即使在显式设置值后也将值设置为 "临时"。</span><span class="sxs-lookup"><span data-stu-id="83db8-175">Set <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsTemporary> to true to force a value to be temporary even after it has been explicitly set.</span></span>

### <a name="working-with-a-single-navigation"></a><span data-ttu-id="83db8-176">使用单个导航</span><span class="sxs-lookup"><span data-stu-id="83db8-176">Working with a single navigation</span></span>

<span data-ttu-id="83db8-177">、和的多个重载 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Reference%2A?displayProperty=nameWithType> <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Collection%2A?displayProperty=nameWithType> <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Navigation%2A?displayProperty=nameWithType> 允许访问有关单个导航的信息。</span><span class="sxs-lookup"><span data-stu-id="83db8-177">Several overloads of <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Reference%2A?displayProperty=nameWithType>, <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Collection%2A?displayProperty=nameWithType>, and <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Navigation%2A?displayProperty=nameWithType> allow access to information about an individual navigation.</span></span>

<span data-ttu-id="83db8-178">通过方法访问指向单个相关实体的引用导航 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Reference%2A> 。</span><span class="sxs-lookup"><span data-stu-id="83db8-178">Reference navigations to a single related entity are accessed through the <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Reference%2A> methods.</span></span> <span data-ttu-id="83db8-179">引用导航点指向一对多关系的 "一" 方，并指向一对一关系的两侧。</span><span class="sxs-lookup"><span data-stu-id="83db8-179">Reference navigations point to the "one" sides of one-to-many relationships, and both sides of one-to-one relationships.</span></span> <span data-ttu-id="83db8-180">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-180">For example:</span></span>

<!--
        ReferenceEntry<Post, Blog> referenceEntry1 = context.Entry(post).Reference(e => e.Blog);
        ReferenceEntry<Post, Blog> referenceEntry2 = context.Entry(post).Reference<Blog>("Blog");
        ReferenceEntry referenceEntry3 = context.Entry(post).Reference("Blog");
-->
[!code-csharp[Work_with_a_single_navigation_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_navigation_1)]

<span data-ttu-id="83db8-181">当用于一对多和多对多关系的 "多" 方时，导航也可以是相关实体的集合。</span><span class="sxs-lookup"><span data-stu-id="83db8-181">Navigations can also be collections of related entities when used for the "many" sides of one-to-many and many-to-many relationships.</span></span> <span data-ttu-id="83db8-182"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Collection%2A>方法用于访问集合导航。</span><span class="sxs-lookup"><span data-stu-id="83db8-182">The <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Collection%2A> methods are used to access collection navigations.</span></span> <span data-ttu-id="83db8-183">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-183">For example:</span></span>

<!--
        CollectionEntry<Blog, Post> collectionEntry1 = context.Entry(blog).Collection(e => e.Posts);
        CollectionEntry<Blog, Post> collectionEntry2 = context.Entry(blog).Collection<Post>("Posts");
        CollectionEntry collectionEntry3 = context.Entry(blog).Collection("Posts");
-->
[!code-csharp[Work_with_a_single_navigation_2a](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_navigation_2a)]

<span data-ttu-id="83db8-184">某些操作对于所有导航都很常见。</span><span class="sxs-lookup"><span data-stu-id="83db8-184">Some operations are common for all navigations.</span></span> <span data-ttu-id="83db8-185">使用方法可同时访问引用和集合导航 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Navigation%2A?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="83db8-185">These can be accessed for both reference and collection navigations using the <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Navigation%2A?displayProperty=nameWithType> method.</span></span> <span data-ttu-id="83db8-186">请注意，在同时访问所有导航时，仅可使用非泛型访问。</span><span class="sxs-lookup"><span data-stu-id="83db8-186">Note that only non-generic access is available when accessing all navigations together.</span></span> <span data-ttu-id="83db8-187">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-187">For example:</span></span>

<!--
        NavigationEntry navigationEntry = context.Entry(blog).Navigation("Posts");
-->
[!code-csharp[Work_with_a_single_navigation_2b](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_navigation_2b)]

<span data-ttu-id="83db8-188">下表总结了使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ReferenceEntry%602> 、和的方法 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.CollectionEntry%602> <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry> ：</span><span class="sxs-lookup"><span data-stu-id="83db8-188">The following table summarizes ways to use <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ReferenceEntry%602>, <xref:Microsoft.EntityFrameworkCore.ChangeTracking.CollectionEntry%602>, and <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry>:</span></span>

| <span data-ttu-id="83db8-189">NavigationEntry 成员</span><span class="sxs-lookup"><span data-stu-id="83db8-189">NavigationEntry member</span></span>                                                                                    | <span data-ttu-id="83db8-190">描述</span><span class="sxs-lookup"><span data-stu-id="83db8-190">Description</span></span>
|:----------------------------------------------------------------------------------------------------------|----------------------
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.MemberEntry.CurrentValue?displayProperty=nameWithType> | <span data-ttu-id="83db8-191">获取和设置导航的当前值。</span><span class="sxs-lookup"><span data-stu-id="83db8-191">Gets and sets the current value of the navigation.</span></span> <span data-ttu-id="83db8-192">这是集合导航的整个集合。</span><span class="sxs-lookup"><span data-stu-id="83db8-192">This is the entire collection for collection navigations.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry.Metadata?displayProperty=nameWithType> | <span data-ttu-id="83db8-193"><xref:Microsoft.EntityFrameworkCore.Metadata.INavigationBase> 导航的元数据。</span><span class="sxs-lookup"><span data-stu-id="83db8-193"><xref:Microsoft.EntityFrameworkCore.Metadata.INavigationBase> metadata for the navigation.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry.IsLoaded?displayProperty=nameWithType> | <span data-ttu-id="83db8-194">获取或设置一个值，该值指示是否已从数据库完全加载相关实体或集合。</span><span class="sxs-lookup"><span data-stu-id="83db8-194">Gets or sets a value indicating whether the related entity or collection has been fully loaded from the database.</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry.Load?displayProperty=nameWithType>     | <span data-ttu-id="83db8-195">从数据库加载相关实体或集合;请参阅 [显式加载相关数据](xref:core/querying/related-data/explicit)。</span><span class="sxs-lookup"><span data-stu-id="83db8-195">Loads the related entity or collection from the database; see [Explicit Loading of Related Data](xref:core/querying/related-data/explicit).</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry.Query?displayProperty=nameWithType>    | <span data-ttu-id="83db8-196">查询 EF Core 将使用将此导航作为 `IQueryable` 可以进一步组合的来加载，请参阅 [显式加载相关数据](xref:core/querying/related-data/explicit)。</span><span class="sxs-lookup"><span data-stu-id="83db8-196">The query EF Core would use to load this navigation as an `IQueryable` that can be further composed; see [Explicit Loading of Related Data](xref:core/querying/related-data/explicit).</span></span>

### <a name="working-with-all-properties-of-an-entity"></a><span data-ttu-id="83db8-197">使用实体的所有属性</span><span class="sxs-lookup"><span data-stu-id="83db8-197">Working with all properties of an entity</span></span>

<span data-ttu-id="83db8-198"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Properties?displayProperty=nameWithType><xref:System.Collections.Generic.IEnumerable%601> <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry> 为实体的每个属性返回的。</span><span class="sxs-lookup"><span data-stu-id="83db8-198"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Properties?displayProperty=nameWithType> returns an <xref:System.Collections.Generic.IEnumerable%601> of <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry> for every property of the entity.</span></span> <span data-ttu-id="83db8-199">这可用于为实体的每个属性执行操作。</span><span class="sxs-lookup"><span data-stu-id="83db8-199">This can be used to perform an action for every property of the entity.</span></span> <span data-ttu-id="83db8-200">例如，若要将任何 DateTime 属性设置为 `DateTime.Now` ：</span><span class="sxs-lookup"><span data-stu-id="83db8-200">For example, to set any DateTime property to `DateTime.Now`:</span></span>

<!--
        foreach (var propertyEntry in context.Entry(blog).Properties)
        {
            if (propertyEntry.Metadata.ClrType == typeof(DateTime))
            {
                propertyEntry.CurrentValue = DateTime.Now;
            }
        }
-->
[!code-csharp[Work_with_all_properties_of_an_entity_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_1)]

<span data-ttu-id="83db8-201">此外，EntityEntry 包含多个用于同时获取和设置所有属性值的方法。</span><span class="sxs-lookup"><span data-stu-id="83db8-201">In addition, EntityEntry contains several methods to get and set all property values at the same time.</span></span> <span data-ttu-id="83db8-202">这些方法使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues> 类，该类表示属性及其值的集合。</span><span class="sxs-lookup"><span data-stu-id="83db8-202">These methods use the <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues> class, which represents a collection of properties and their values.</span></span> <span data-ttu-id="83db8-203">可以获取当前或原始值的 PropertyValues，也可以为数据库中当前存储的值获取。</span><span class="sxs-lookup"><span data-stu-id="83db8-203">PropertyValues can be obtained for current or original values, or for the values as currently stored in the database.</span></span> <span data-ttu-id="83db8-204">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-204">For example:</span></span>

<!--
        var currentValues = context.Entry(blog).CurrentValues;
        var originalValues = context.Entry(blog).OriginalValues;
        var databaseValues = context.Entry(blog).GetDatabaseValues();
-->
[!code-csharp[Work_with_all_properties_of_an_entity_2a](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_2a)]

<span data-ttu-id="83db8-205">这些 PropertyValues 对象本身并不十分有用。</span><span class="sxs-lookup"><span data-stu-id="83db8-205">These PropertyValues objects are not very useful on their own.</span></span> <span data-ttu-id="83db8-206">但是，它们可以组合起来执行操作实体时所需的常见操作。</span><span class="sxs-lookup"><span data-stu-id="83db8-206">However, they can be combined to perform common operations needed when manipulating entities.</span></span> <span data-ttu-id="83db8-207">当处理数据传输对象以及解决 [开放式并发冲突](xref:core/saving/concurrency)时，这非常有用。</span><span class="sxs-lookup"><span data-stu-id="83db8-207">This is useful when working with data transfer objects and when resolving [optimistic concurrency conflicts](xref:core/saving/concurrency).</span></span> <span data-ttu-id="83db8-208">以下部分演示了一些示例。</span><span class="sxs-lookup"><span data-stu-id="83db8-208">The following sections show some examples.</span></span>

#### <a name="setting-current-or-original-values-from-an-entity-or-dto"></a><span data-ttu-id="83db8-209">设置实体或 DTO 的当前值或原始值</span><span class="sxs-lookup"><span data-stu-id="83db8-209">Setting current or original values from an entity or DTO</span></span>

<span data-ttu-id="83db8-210">可以通过从另一个对象复制值来更新实体的当前值或原始值。</span><span class="sxs-lookup"><span data-stu-id="83db8-210">The current or original values of an entity can be updated by copying values from another object.</span></span> <span data-ttu-id="83db8-211">例如，请考虑一个 `BlogDto` 数据传输对象 (DTO) ，该对象具有与实体类型相同的属性：</span><span class="sxs-lookup"><span data-stu-id="83db8-211">For example, consider a `BlogDto` data transfer object (DTO) with the same properties as the entity type:</span></span>

<!--
public class BlogDto
{
    public int Id { get; set; }
    public string Name { get; set; }
}
-->
[!code-csharp[BlogDto](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=BlogDto)]

<span data-ttu-id="83db8-212">这可用于设置所跟踪实体的当前值，使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues.SetValues%2A?displayProperty=nameWithType> ：</span><span class="sxs-lookup"><span data-stu-id="83db8-212">This can be used to set the current values of a tracked entity using <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues.SetValues%2A?displayProperty=nameWithType>:</span></span>

<!--
        var blogDto = new BlogDto { Id = 1, Name = "1unicorn2" };

        context.Entry(blog).CurrentValues.SetValues(blogDto);
-->
[!code-csharp[Work_with_all_properties_of_an_entity_2b](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_2b)]

<span data-ttu-id="83db8-213">当使用从服务调用或 n 层应用程序中的客户端获取的值更新实体时，有时会使用此方法。</span><span class="sxs-lookup"><span data-stu-id="83db8-213">This technique is sometimes used when updating an entity with values obtained from a service call or a client in an n-tier application.</span></span> <span data-ttu-id="83db8-214">请注意，所使用的对象不必与实体具有相同的类型，但前提是它具有与实体的名称相匹配的属性。</span><span class="sxs-lookup"><span data-stu-id="83db8-214">Note that the object used does not have to be of the same type as the entity so long as it has properties whose names match those of the entity.</span></span> <span data-ttu-id="83db8-215">在上面的示例中，DTO 的实例 `BlogDto` 用于设置所跟踪实体的当前值 `Blog` 。</span><span class="sxs-lookup"><span data-stu-id="83db8-215">In the example above, an instance of the DTO `BlogDto` is used to set the current values of a tracked `Blog` entity.</span></span>

<span data-ttu-id="83db8-216">请注意，仅当值设置与当前值不同时，才将属性标记为已修改。</span><span class="sxs-lookup"><span data-stu-id="83db8-216">Note that properties will only be marked as modified if the value set differs from the current value.</span></span>

#### <a name="setting-current-or-original-values-from-a-dictionary"></a><span data-ttu-id="83db8-217">从字典设置当前值或原始值</span><span class="sxs-lookup"><span data-stu-id="83db8-217">Setting current or original values from a dictionary</span></span>

<span data-ttu-id="83db8-218">前面的示例从实体或 DTO 实例设置值。</span><span class="sxs-lookup"><span data-stu-id="83db8-218">The previous example set values from an entity or DTO instance.</span></span> <span data-ttu-id="83db8-219">当属性值作为名称/值对存储在字典中时，可以使用相同的行为。</span><span class="sxs-lookup"><span data-stu-id="83db8-219">The same behavior is available when property values are stored as name/value pairs in a dictionary.</span></span> <span data-ttu-id="83db8-220">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-220">For example:</span></span>

<!--
        var blogDictionary = new Dictionary<string, object>
        {
            ["Id"] = 1,
            ["Name"] = "1unicorn2"
        };

        context.Entry(blog).CurrentValues.SetValues(blogDictionary);
-->
[!code-csharp[Work_with_all_properties_of_an_entity_2d](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_2d)]

#### <a name="setting-current-or-original-values-from-the-database"></a><span data-ttu-id="83db8-221">从数据库设置当前值或原始值</span><span class="sxs-lookup"><span data-stu-id="83db8-221">Setting current or original values from the database</span></span>

<span data-ttu-id="83db8-222">通过调用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.GetDatabaseValues> 或 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.GetDatabaseValuesAsync%2A> 并使用返回的对象设置当前值或原始值，或者同时使用这两个值，可以使用数据库中的最新值更新实体的当前值或原始值。</span><span class="sxs-lookup"><span data-stu-id="83db8-222">The current or original values of an entity can be updated with the latest values from the database by calling <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.GetDatabaseValues> or <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.GetDatabaseValuesAsync%2A> and using the returned object to set current or original values, or both.</span></span> <span data-ttu-id="83db8-223">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-223">For example:</span></span>

<!--
        var databaseValues = context.Entry(blog).GetDatabaseValues();
        context.Entry(blog).CurrentValues.SetValues(databaseValues);
        context.Entry(blog).OriginalValues.SetValues(databaseValues);
-->
[!code-csharp[Work_with_all_properties_of_an_entity_2c](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_2c)]

#### <a name="creating-a-cloned-object-containing-current-original-or-database-values"></a><span data-ttu-id="83db8-224">创建包含当前、原始或数据库值的克隆对象</span><span class="sxs-lookup"><span data-stu-id="83db8-224">Creating a cloned object containing current, original, or database values</span></span>

<span data-ttu-id="83db8-225">从 CurrentValues、OriginalValues 或 GetDatabaseValues 返回的 PropertyValues 对象可用于通过创建实体的克隆 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues.ToObject?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="83db8-225">The PropertyValues object returned from CurrentValues, OriginalValues, or GetDatabaseValues can be used to create a clone of the entity using <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues.ToObject?displayProperty=nameWithType>.</span></span> <span data-ttu-id="83db8-226">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-226">For example:</span></span>

<!--
var clonedBlog = context.Entry(blog).GetDatabaseValues().ToObject();
-->
[!code-csharp[Work_with_all_properties_of_an_entity_2e](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_2e)]

<span data-ttu-id="83db8-227">请注意，将 `ToObject` 返回一个不由 DbContext 跟踪的新实例。</span><span class="sxs-lookup"><span data-stu-id="83db8-227">Note that `ToObject` returns a new instance that is not tracked by the DbContext.</span></span> <span data-ttu-id="83db8-228">返回的对象也不会将任何关系设置为其他实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-228">The returned object also does not have any relationships set to other entities.</span></span>

<span data-ttu-id="83db8-229">克隆的对象可用于解决与数据库的并发更新相关的问题，尤其是当数据绑定到特定类型的对象时。</span><span class="sxs-lookup"><span data-stu-id="83db8-229">The cloned object can be useful for resolving issues related to concurrent updates to the database, especially when data binding to objects of a certain type.</span></span> <span data-ttu-id="83db8-230">有关详细信息，请参阅 [乐观并发](xref:core/saving/concurrency) 。</span><span class="sxs-lookup"><span data-stu-id="83db8-230">See [optimistic concurrency](xref:core/saving/concurrency) for more information.</span></span>

### <a name="working-with-all-navigations-of-an-entity"></a><span data-ttu-id="83db8-231">使用实体的所有导航</span><span class="sxs-lookup"><span data-stu-id="83db8-231">Working with all navigations of an entity</span></span>

<span data-ttu-id="83db8-232"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Navigations?displayProperty=nameWithType><xref:System.Collections.Generic.IEnumerable%601> <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry> 对于实体的每个导航，返回的。</span><span class="sxs-lookup"><span data-stu-id="83db8-232"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Navigations?displayProperty=nameWithType> returns an <xref:System.Collections.Generic.IEnumerable%601> of <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry> for every navigation of the entity.</span></span> <span data-ttu-id="83db8-233"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.References?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Collections?displayProperty=nameWithType> 执行相同的操作，但限制为分别引用或收集导航。</span><span class="sxs-lookup"><span data-stu-id="83db8-233"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.References?displayProperty=nameWithType> and <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Collections?displayProperty=nameWithType> do the same thing, but restricted to reference or collection navigations respectively.</span></span> <span data-ttu-id="83db8-234">这可用于为实体的每个导航执行操作。</span><span class="sxs-lookup"><span data-stu-id="83db8-234">This can be used to perform an action for every navigation of the entity.</span></span> <span data-ttu-id="83db8-235">例如，若要强制加载所有相关实体：</span><span class="sxs-lookup"><span data-stu-id="83db8-235">For example, to force loading of all related entities:</span></span>

<!--
        foreach (var navigationEntry in context.Entry(blog).Navigations)
        {
            navigationEntry.Load();
        }
-->
[!code-csharp[Work_with_all_navigations_of_an_entity_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_navigations_of_an_entity_1)]

### <a name="working-with-all-members-of-an-entity"></a><span data-ttu-id="83db8-236">使用实体的所有成员</span><span class="sxs-lookup"><span data-stu-id="83db8-236">Working with all members of an entity</span></span>

<span data-ttu-id="83db8-237">常规属性和导航属性具有不同的状态和行为。</span><span class="sxs-lookup"><span data-stu-id="83db8-237">Regular properties and navigation properties have different state and behavior.</span></span> <span data-ttu-id="83db8-238">因此，可以单独处理导航和非导航，如以上部分所示。</span><span class="sxs-lookup"><span data-stu-id="83db8-238">It is therefore common to process navigations and non-navigations separately, as shown in the sections above.</span></span> <span data-ttu-id="83db8-239">但有时，对实体的任何成员执行某些操作都很有用，无论它是常规属性还是导航。</span><span class="sxs-lookup"><span data-stu-id="83db8-239">However, sometimes it can be useful to do something with any member of the entity, regardless of whether it is a regular property or navigation.</span></span> <span data-ttu-id="83db8-240"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Member%2A?displayProperty=nameWithType><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Members?displayProperty=nameWithType>出于此目的提供了和。</span><span class="sxs-lookup"><span data-stu-id="83db8-240"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Member%2A?displayProperty=nameWithType> and <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Members?displayProperty=nameWithType> are provided for this purpose.</span></span> <span data-ttu-id="83db8-241">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-241">For example:</span></span>

<!--
        foreach (var memberEntry in context.Entry(blog).Members)
        {
            Console.WriteLine(
                $"Member {memberEntry.Metadata.Name} is of type {memberEntry.Metadata.ClrType.ShortDisplayName()} and has value {memberEntry.CurrentValue}");
        }
-->
[!code-csharp[Work_with_all_members_of_an_entity_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_members_of_an_entity_1)]

<span data-ttu-id="83db8-242">在示例中，在博客上运行此代码将生成以下输出：</span><span class="sxs-lookup"><span data-stu-id="83db8-242">Running this code on a blog from the sample generates the following output:</span></span>

```output
Member Id is of type int and has value 1
Member Name is of type string and has value .NET Blog
Member Posts is of type IList<Post> and has value System.Collections.Generic.List`1[Post]
```

> [!TIP]
> <span data-ttu-id="83db8-243">[更改跟踪](xref:core/change-tracking/debug-views)器的 "调试" 视图将显示类似于下面的信息。</span><span class="sxs-lookup"><span data-stu-id="83db8-243">The [change tracker debug view](xref:core/change-tracking/debug-views) shows information like this.</span></span> <span data-ttu-id="83db8-244">整个更改跟踪器的 "调试" 视图是从 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.DebugView?displayProperty=nameWithType> 每个被跟踪实体的个体生成的。</span><span class="sxs-lookup"><span data-stu-id="83db8-244">The debug view for the entire change tracker is generated from the individual <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.DebugView?displayProperty=nameWithType> of each tracked entity.</span></span>

## <a name="find-and-findasync"></a><span data-ttu-id="83db8-245">Find 和 FindAsync</span><span class="sxs-lookup"><span data-stu-id="83db8-245">Find and FindAsync</span></span>

<span data-ttu-id="83db8-246"><xref:Microsoft.EntityFrameworkCore.DbContext.Find%2A?displayProperty=nameWithType>、 <xref:Microsoft.EntityFrameworkCore.DbContext.FindAsync%2A?displayProperty=nameWithType> 、 <xref:Microsoft.EntityFrameworkCore.DbSet%601.Find%2A?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.DbSet%601.FindAsync%2A?displayProperty=nameWithType> 旨在有效地查找其主键已知的单个实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-246"><xref:Microsoft.EntityFrameworkCore.DbContext.Find%2A?displayProperty=nameWithType>, <xref:Microsoft.EntityFrameworkCore.DbContext.FindAsync%2A?displayProperty=nameWithType>, <xref:Microsoft.EntityFrameworkCore.DbSet%601.Find%2A?displayProperty=nameWithType>, and <xref:Microsoft.EntityFrameworkCore.DbSet%601.FindAsync%2A?displayProperty=nameWithType> are designed for efficient lookup of a single entity when its primary key is known.</span></span> <span data-ttu-id="83db8-247">查找首先检查是否已跟踪实体，如果是，则立即返回实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-247">Find first checks if the entity is already tracked, and if so returns the entity immediately.</span></span> <span data-ttu-id="83db8-248">仅当未在本地跟踪实体时才会进行数据库查询。</span><span class="sxs-lookup"><span data-stu-id="83db8-248">A database query is only made if the entity is not tracked locally.</span></span> <span data-ttu-id="83db8-249">例如，请考虑下面这段代码：</span><span class="sxs-lookup"><span data-stu-id="83db8-249">For example, consider this code that calls Find twice for the same entity:</span></span>

<!--
        using var context = new BlogsContext();

        Console.WriteLine("First call to Find...");
        var blog1 = context.Blogs.Find(1);

        Console.WriteLine($"...found blog {blog1.Name}");

        Console.WriteLine();
        Console.WriteLine("Second call to Find...");
        var blog2 = context.Blogs.Find(1);
        Debug.Assert(blog1 == blog2);

        Console.WriteLine("...returned the same instance without executing a query.");
-->
[!code-csharp[Find_and_FindAsync_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Find_and_FindAsync_1)]

<span data-ttu-id="83db8-250">使用 SQLite 时，此代码的输出 (包括 EF Core 日志记录) ：</span><span class="sxs-lookup"><span data-stu-id="83db8-250">The output from this code (including EF Core logging) when using SQLite is:</span></span>

```output
First call to Find...
info: 12/29/2020 07:45:53.682 RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command)
      Executed DbCommand (1ms) [Parameters=[@__p_0='1' (DbType = String)], CommandType='Text', CommandTimeout='30']
      SELECT "b"."Id", "b"."Name"
      FROM "Blogs" AS "b"
      WHERE "b"."Id" = @__p_0
      LIMIT 1
...found blog .NET Blog

Second call to Find...
...returned the same instance without executing a query.
```

<span data-ttu-id="83db8-251">请注意，第一次调用在本地找不到实体，因此执行数据库查询。</span><span class="sxs-lookup"><span data-stu-id="83db8-251">Notice that the first call does not find the entity locally and so executes a database query.</span></span> <span data-ttu-id="83db8-252">相反，第二次调用返回相同的实例而不查询数据库，因为它已被跟踪。</span><span class="sxs-lookup"><span data-stu-id="83db8-252">Conversely, the second call returns the same instance without querying the database because it is already being tracked.</span></span>

<span data-ttu-id="83db8-253">如果具有给定键的实体未在本地跟踪并且数据库中不存在，则 Find 将返回 null。</span><span class="sxs-lookup"><span data-stu-id="83db8-253">Find returns null if an entity with the given key is not tracked locally and does not exist in the database.</span></span>

### <a name="composite-keys"></a><span data-ttu-id="83db8-254">组合键</span><span class="sxs-lookup"><span data-stu-id="83db8-254">Composite keys</span></span>

<span data-ttu-id="83db8-255">"查找" 还可用于组合键。</span><span class="sxs-lookup"><span data-stu-id="83db8-255">Find can also be used with composite keys.</span></span> <span data-ttu-id="83db8-256">例如，考虑一个 `OrderLine` 具有由订单 ID 和产品 ID 组成的组合键的实体：</span><span class="sxs-lookup"><span data-stu-id="83db8-256">For example, consider an `OrderLine` entity with a composite key consisting of the order ID and the product ID:</span></span>

<!--
public class OrderLine
{
    public int OrderId { get; set; }
    public int ProductId { get; set; }

    //...
}
-->
[!code-csharp[OrderLine](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=OrderLine)]

<span data-ttu-id="83db8-257">必须在中配置组合键 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A?displayProperty=nameWithType> ，才能定义关键部分 _及其顺序_。</span><span class="sxs-lookup"><span data-stu-id="83db8-257">The composite key must be configured in <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A?displayProperty=nameWithType> to define the key parts _and their order_.</span></span> <span data-ttu-id="83db8-258">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-258">For example:</span></span>

<!--
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder
            .Entity<OrderLine>()
            .HasKey(e => new { e.OrderId, e.ProductId });
    }
-->
[!code-csharp[OnModelCreating](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=OnModelCreating)]

<span data-ttu-id="83db8-259">请注意， `OrderId` 是键的第一部分， `ProductId` 是密钥的第二部分。</span><span class="sxs-lookup"><span data-stu-id="83db8-259">Notice that `OrderId` is the first part of the key and `ProductId` is the second part of the key.</span></span> <span data-ttu-id="83db8-260">传递要查找的键值时，必须使用此顺序。</span><span class="sxs-lookup"><span data-stu-id="83db8-260">This order must be used when passing key values to Find.</span></span> <span data-ttu-id="83db8-261">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-261">For example:</span></span>

<!--
        var orderline = context.OrderLines.Find(orderId, productId);
-->
[!code-csharp[Find_and_FindAsync_2](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Find_and_FindAsync_2)]

## <a name="using-changetrackerentries-to-access-all-tracked-entities"></a><span data-ttu-id="83db8-262">使用 ChangeTracker 访问所有跟踪的实体</span><span class="sxs-lookup"><span data-stu-id="83db8-262">Using ChangeTracker.Entries to access all tracked entities</span></span>

<span data-ttu-id="83db8-263">到目前为止，我们只是一次访问了一个 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry> 。</span><span class="sxs-lookup"><span data-stu-id="83db8-263">So far we have accessed only a single <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry> at a time.</span></span> <span data-ttu-id="83db8-264"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries?displayProperty=nameWithType> 为 DbContext 当前跟踪的每个实体返回 EntityEntry。</span><span class="sxs-lookup"><span data-stu-id="83db8-264"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries?displayProperty=nameWithType> returns an EntityEntry for every entity currently tracked by the DbContext.</span></span> <span data-ttu-id="83db8-265">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-265">For example:</span></span>

<!--
        using var context = new BlogsContext();
        var blogs = context.Blogs.Include(e => e.Posts).ToList();

        foreach (var entityEntry in context.ChangeTracker.Entries())
        {
            Console.WriteLine($"Found {entityEntry.Metadata.Name} entity with ID {entityEntry.Property("Id").CurrentValue}");
        }
-->
[!code-csharp[Using_ChangeTracker_Entries_to_access_all_tracked_entities_1a](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_ChangeTracker_Entries_to_access_all_tracked_entities_1a)]

<span data-ttu-id="83db8-266">此代码生成以下输出：</span><span class="sxs-lookup"><span data-stu-id="83db8-266">This code generates the following output:</span></span>

```output
Found Blog entity with ID 1
Found Post entity with ID 1
Found Post entity with ID 2
```

<span data-ttu-id="83db8-267">请注意，将返回博客和帖子的条目。</span><span class="sxs-lookup"><span data-stu-id="83db8-267">Notice that entries for both blogs and posts are returned.</span></span> <span data-ttu-id="83db8-268">可以改用泛型重载，将结果筛选为特定实体类型 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%60%601?displayProperty=nameWithType> ：</span><span class="sxs-lookup"><span data-stu-id="83db8-268">The results can instead be filtered to a specific entity type using the <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%60%601?displayProperty=nameWithType> generic overload:</span></span>

<!--
        foreach (var entityEntry in context.ChangeTracker.Entries<Post>())
        {
            Console.WriteLine(
                $"Found {entityEntry.Metadata.Name} entity with ID {entityEntry.Property(e => e.Id).CurrentValue}");
        }
-->
[!code-csharp[Using_ChangeTracker_Entries_to_access_all_tracked_entities_1b](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_ChangeTracker_Entries_to_access_all_tracked_entities_1b)]

<span data-ttu-id="83db8-269">此代码的输出显示仅返回 post：</span><span class="sxs-lookup"><span data-stu-id="83db8-269">The output from this code shows that only posts are returned:</span></span>

```output
Found Post entity with ID 1
Found Post entity with ID 2
```

<span data-ttu-id="83db8-270">此外，使用泛型重载会返回泛型 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 实例。</span><span class="sxs-lookup"><span data-stu-id="83db8-270">Also, using the generic overload returns generic <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> instances.</span></span> <span data-ttu-id="83db8-271">这就允许 `Id` 在此示例中对属性的访问权限。</span><span class="sxs-lookup"><span data-stu-id="83db8-271">This is what allows that fluent-like access to the `Id` property in this example.</span></span>

<span data-ttu-id="83db8-272">用于筛选的泛型类型不一定是映射实体类型;可以改为使用未映射的基类型或接口。</span><span class="sxs-lookup"><span data-stu-id="83db8-272">The generic type used for filtering does not have to be a mapped entity type; an unmapped base type or interface can be used instead.</span></span> <span data-ttu-id="83db8-273">例如，如果模型中的所有实体类型都实现定义其键属性的接口：</span><span class="sxs-lookup"><span data-stu-id="83db8-273">For example, if all the entity types in the model implement an interface defining their key property:</span></span>

<!--
public interface IEntityWithKey
{
    int Id { get; set; }
}
-->
[!code-csharp[IEntityWithKey](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=IEntityWithKey)]

<span data-ttu-id="83db8-274">然后，可以使用此接口以强类型方式处理任何跟踪实体的键。</span><span class="sxs-lookup"><span data-stu-id="83db8-274">Then this interface can be used to work with the key of any tracked entity in a strongly-typed manner.</span></span> <span data-ttu-id="83db8-275">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-275">For example:</span></span>

<!--
        foreach (var entityEntry in context.ChangeTracker.Entries<IEntityWithKey>())
        {
            Console.WriteLine(
                $"Found {entityEntry.Metadata.Name} entity with ID {entityEntry.Property(e => e.Id).CurrentValue}");
        }
-->
[!code-csharp[Using_ChangeTracker_Entries_to_access_all_tracked_entities_1c](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_ChangeTracker_Entries_to_access_all_tracked_entities_1c)]

## <a name="using-dbsetlocal-to-query-tracked-entities"></a><span data-ttu-id="83db8-276">使用 DbSet 查询跟踪的实体</span><span class="sxs-lookup"><span data-stu-id="83db8-276">Using DbSet.Local to query tracked entities</span></span>

<span data-ttu-id="83db8-277">EF Core 查询始终在数据库上执行，并且仅返回已保存到数据库的实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-277">EF Core queries are always executed on the database, and only return entities that have been saved to the database.</span></span> <span data-ttu-id="83db8-278"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> 提供了一种机制，用于在 DbContext 中查询本地跟踪实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-278"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> provides a mechanism to query the DbContext for local, tracked entities.</span></span>

<span data-ttu-id="83db8-279">由于 `DbSet.Local` 用于查询跟踪的实体，因此通常会将实体加载到 DbContext 中，然后使用已加载的实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-279">Since `DbSet.Local` is used to query tracked entities, it is typical to load entities into the DbContext and then work with those loaded entities.</span></span> <span data-ttu-id="83db8-280">这对于数据绑定尤其如此，但在其他情况下也很有用。</span><span class="sxs-lookup"><span data-stu-id="83db8-280">This is especially true for data binding, but can also be useful in other situations.</span></span> <span data-ttu-id="83db8-281">例如，在以下代码中，首先查询数据库以获取所有博客和帖子。</span><span class="sxs-lookup"><span data-stu-id="83db8-281">For example, in the following code the database is first queried for all blogs and posts.</span></span> <span data-ttu-id="83db8-282"><xref:Microsoft.EntityFrameworkCore.EntityFrameworkQueryableExtensions.Load%2A>扩展方法用于使用上下文跟踪的结果来执行此查询，而不会直接返回到应用程序。</span><span class="sxs-lookup"><span data-stu-id="83db8-282">The <xref:Microsoft.EntityFrameworkCore.EntityFrameworkQueryableExtensions.Load%2A> extension method is used to execute this query with the results tracked by the context without being returned directly to the application.</span></span> <span data-ttu-id="83db8-283">使用 `ToList` 或类似 (具有相同的效果，但会产生创建返回列表的开销，但此处不需要这样做。 ) 该示例然后使用 `DbSet.Local` 访问本地跟踪的实体：</span><span class="sxs-lookup"><span data-stu-id="83db8-283">(Using `ToList` or similar has the same effect but with the overhead of creating the returned list, which is not needed here.) The example then uses `DbSet.Local` to access the locally tracked entities:</span></span>

<!--
        using var context = new BlogsContext();

        context.Blogs.Include(e => e.Posts).Load();

        foreach (var blog in context.Blogs.Local)
        {
            Console.WriteLine($"Blog: {blog.Name}");
        }

        foreach (var post in context.Posts.Local)
        {
            Console.WriteLine($"Post: {post.Title}");
        }
-->
[!code-csharp[Using_DbSet_Local_to_query_tracked_entities_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_DbSet_Local_to_query_tracked_entities_1)]

<span data-ttu-id="83db8-284">请注意，与不同的是 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries?displayProperty=nameWithType> ， `DbSet.Local` 直接返回实体实例。</span><span class="sxs-lookup"><span data-stu-id="83db8-284">Notice that, unlike <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries?displayProperty=nameWithType>, `DbSet.Local` returns entity instances directly.</span></span> <span data-ttu-id="83db8-285">当然，EntityEntry 可以通过调用来获取返回的实体 <xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="83db8-285">An EntityEntry can, of course, always be obtained for the returned entity by calling <xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType>.</span></span>

### <a name="the-local-view"></a><span data-ttu-id="83db8-286">本地视图</span><span class="sxs-lookup"><span data-stu-id="83db8-286">The local view</span></span>

<span data-ttu-id="83db8-287"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> 返回用于反映这些实体当前的本地跟踪实体的视图 <xref:Microsoft.EntityFrameworkCore.EntityState> 。</span><span class="sxs-lookup"><span data-stu-id="83db8-287"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> returns a view of locally tracked entities that reflects the current <xref:Microsoft.EntityFrameworkCore.EntityState> of those entities.</span></span> <span data-ttu-id="83db8-288">具体而言，这意味着：</span><span class="sxs-lookup"><span data-stu-id="83db8-288">Specifically, this means that:</span></span>

- <span data-ttu-id="83db8-289">`Added` 实体包括在内。</span><span class="sxs-lookup"><span data-stu-id="83db8-289">`Added` entities are included.</span></span> <span data-ttu-id="83db8-290">请注意，对于普通 EF Core 查询，这种情况并非如此，因为 `Added` 实体不存在于数据库中，因此不会由数据库查询返回。</span><span class="sxs-lookup"><span data-stu-id="83db8-290">Note that this is not the case for normal EF Core queries, since `Added` entities do not yet exist in the database and so are therefore never returned by a database query.</span></span>
- <span data-ttu-id="83db8-291">`Deleted` 排除实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-291">`Deleted` entities are excluded.</span></span> <span data-ttu-id="83db8-292">请注意，对于普通 EF Core 查询，这种情况并不是这样，因为 `Deleted` 实体仍然存在于数据库中，并且由数据库查询返回。</span><span class="sxs-lookup"><span data-stu-id="83db8-292">Note that this is again not the case for normal EF Core queries, since `Deleted` entities still exist in the database and so _are_ returned by database queries.</span></span>

<span data-ttu-id="83db8-293">所有这些都是指对 `DbSet.Local` 反映实体关系图当前概念状态的数据的查看，其中 `Added` 包含实体和 `Deleted` 实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-293">All of this means that `DbSet.Local` is view over the data that reflects the current conceptual state of the entity graph, with `Added` entities included and `Deleted` entities excluded.</span></span> <span data-ttu-id="83db8-294">这与调用 SaveChanges 后预期的数据库状态一致。</span><span class="sxs-lookup"><span data-stu-id="83db8-294">This matches what database state is expected to be after SaveChanges is called.</span></span>

<span data-ttu-id="83db8-295">这通常是数据绑定的理想视图，因为它会根据应用程序所做的更改向用户提供数据。</span><span class="sxs-lookup"><span data-stu-id="83db8-295">This is typically the ideal view for data binding, since it presents to the user the data as they understand it based on the changes made by the application.</span></span>

<span data-ttu-id="83db8-296">下面的代码演示，我将一个 post 标记为 `Deleted` ，然后添加一个新帖子，将其标记为 `Added` ：</span><span class="sxs-lookup"><span data-stu-id="83db8-296">The following code demonstrates this my marking one post as `Deleted` and then adding a new post, marking it as `Added`:</span></span>

<!--
        using var context = new BlogsContext();

        var posts = context.Posts.Include(e => e.Blog).ToList();

        Console.WriteLine("Local view after loading posts:");

        foreach (var post in context.Posts.Local)
        {
            Console.WriteLine($"  Post: {post.Title}");
        }

        context.Remove(posts[1]);

        context.Add(new Post
        {
            Title = "What’s next for System.Text.Json?",
            Content = ".NET 5.0 was released recently and has come with many...",
            Blog = posts[0].Blog
        });

        Console.WriteLine("Local view after adding and deleting posts:");

        foreach (var post in context.Posts.Local)
        {
            Console.WriteLine($"  Post: {post.Title}");
        }
-->
[!code-csharp[Using_DbSet_Local_to_query_tracked_entities_2](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_DbSet_Local_to_query_tracked_entities_2)]

<span data-ttu-id="83db8-297">此代码的输出为：</span><span class="sxs-lookup"><span data-stu-id="83db8-297">The output from this code is:</span></span>

```output
Local view after loading posts:
  Post: Announcing the Release of EF Core 5.0
  Post: Announcing F# 5
  Post: Announcing .NET 5.0
Local view after adding and deleting posts:
  Post: What’s next for System.Text.Json?
  Post: Announcing the Release of EF Core 5.0
  Post: Announcing .NET 5.0
```

<span data-ttu-id="83db8-298">请注意，已删除的 post 将从本地视图中删除，并包含添加的帖子。</span><span class="sxs-lookup"><span data-stu-id="83db8-298">Notice that the deleted post is removed from the local view, and the added post is included.</span></span>

### <a name="using-local-to-add-and-remove-entities"></a><span data-ttu-id="83db8-299">使用本地添加和删除实体</span><span class="sxs-lookup"><span data-stu-id="83db8-299">Using Local to add and remove entities</span></span>

<span data-ttu-id="83db8-300"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> 返回 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.LocalView%601> 的实例。</span><span class="sxs-lookup"><span data-stu-id="83db8-300"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> returns an instance of <xref:Microsoft.EntityFrameworkCore.ChangeTracking.LocalView%601>.</span></span> <span data-ttu-id="83db8-301">这是 <xref:System.Collections.Generic.ICollection%601> 在从集合中添加或删除实体时生成并响应通知的实现。</span><span class="sxs-lookup"><span data-stu-id="83db8-301">This is an implementation of <xref:System.Collections.Generic.ICollection%601> that generates and responds to notifications when entities are added and removed from the collection.</span></span> <span data-ttu-id="83db8-302"> (这是与相同的概念 <xref:System.Collections.ObjectModel.ObservableCollection%601> ，但它是在现有 EF Core 更改跟踪条目的投影上实现的，而不是作为独立的集合实现的。 ) </span><span class="sxs-lookup"><span data-stu-id="83db8-302">(This is the same concept as <xref:System.Collections.ObjectModel.ObservableCollection%601>, but implemented as a projection over existing EF Core change tracking entries, rather than as an independent collection.)</span></span>

<span data-ttu-id="83db8-303">本地视图的通知挂钩到 DbContext 更改跟踪，使本地视图与 DbContext 保持同步。</span><span class="sxs-lookup"><span data-stu-id="83db8-303">The local view's notifications are hooked into DbContext change tracking such that the local view stays in sync with the DbContext.</span></span> <span data-ttu-id="83db8-304">具体而言：</span><span class="sxs-lookup"><span data-stu-id="83db8-304">Specifically:</span></span>

- <span data-ttu-id="83db8-305">添加新实体以 `DbSet.Local` 使其由 DbContext 跟踪，通常处于 `Added` 状态。</span><span class="sxs-lookup"><span data-stu-id="83db8-305">Adding a new entity to `DbSet.Local` causes it to be tracked by the DbContext, typically in the `Added` state.</span></span> <span data-ttu-id="83db8-306"> (如果实体已有生成的键值，则会改为跟踪该实体值 `Unchanged` 。 ) </span><span class="sxs-lookup"><span data-stu-id="83db8-306">(If the entity already has a generated key value, then it is tracked as `Unchanged` instead.)</span></span>
- <span data-ttu-id="83db8-307">从中移除实体 `DbSet.Local` 会使其标记为 `Deleted` 。</span><span class="sxs-lookup"><span data-stu-id="83db8-307">Removing an entity from `DbSet.Local` causes it to be marked as `Deleted`.</span></span>
- <span data-ttu-id="83db8-308">DbContext 跟踪的实体会自动出现在 `DbSet.Local` 集合中。</span><span class="sxs-lookup"><span data-stu-id="83db8-308">An entity that becomes tracked by the DbContext will automatically appear in the `DbSet.Local` collection.</span></span> <span data-ttu-id="83db8-309">例如，如果执行查询来自动引入多个实体，则会使本地视图更新。</span><span class="sxs-lookup"><span data-stu-id="83db8-309">For example, executing a query to bring in more entities automatically causes the local view to be updated.</span></span>
- <span data-ttu-id="83db8-310">标记为的实体 `Deleted` 将自动从本地集合中删除。</span><span class="sxs-lookup"><span data-stu-id="83db8-310">An entity that is marked as `Deleted` will be removed from the local collection automatically.</span></span>

<span data-ttu-id="83db8-311">这意味着，只需通过在集合中添加和移除，就可以使用本地视图来处理跟踪的实体。</span><span class="sxs-lookup"><span data-stu-id="83db8-311">This means the local view can be used to manipulate tracked entities simply by adding and removing from the collection.</span></span> <span data-ttu-id="83db8-312">例如，我们修改前面的示例代码以添加和删除本地集合中的帖子：</span><span class="sxs-lookup"><span data-stu-id="83db8-312">For example, lets modify the previous example code to add and remove posts from the local collection:</span></span>

<!--
        using var context = new BlogsContext();

        var posts = context.Posts.Include(e => e.Blog).ToList();

        Console.WriteLine("Local view after loading posts:");

        foreach (var post in context.Posts.Local)
        {
            Console.WriteLine($"  Post: {post.Title}");
        }

        context.Posts.Local.Remove(posts[1]);

        context.Posts.Local.Add(new Post
        {
            Title = "What’s next for System.Text.Json?",
            Content = ".NET 5.0 was released recently and has come with many...",
            Blog = posts[0].Blog
        });

        Console.WriteLine("Local view after adding and deleting posts:");

        foreach (var post in context.Posts.Local)
        {
            Console.WriteLine($"  Post: {post.Title}");
        }
-->
[!code-csharp[Using_DbSet_Local_to_query_tracked_entities_3](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_DbSet_Local_to_query_tracked_entities_3)]

<span data-ttu-id="83db8-313">由于对本地视图所做的更改与 DbContext 同步，因此输出在上一个示例中保持不变。</span><span class="sxs-lookup"><span data-stu-id="83db8-313">The output remains unchanged from the previous example because changes made to the local view are synced with the DbContext.</span></span>

### <a name="using-the-local-view-for-windows-forms-or-wpf-data-binding"></a><span data-ttu-id="83db8-314">使用 Windows 窗体或 WPF 数据绑定的本地视图</span><span class="sxs-lookup"><span data-stu-id="83db8-314">Using the local view for Windows Forms or WPF data binding</span></span>

<span data-ttu-id="83db8-315"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> 构成用于 EF Core 实体的数据绑定的基础。</span><span class="sxs-lookup"><span data-stu-id="83db8-315"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> forms the basis for data binding to EF Core entities.</span></span> <span data-ttu-id="83db8-316">不过，Windows 窗体和 WPF 在与他们期望的特定类型的通知集合一起使用时，效果最佳。</span><span class="sxs-lookup"><span data-stu-id="83db8-316">However, both Windows Forms and WPF work best when used with the specific type of notifying collection that they expect.</span></span> <span data-ttu-id="83db8-317">本地视图支持创建以下特定的集合类型：</span><span class="sxs-lookup"><span data-stu-id="83db8-317">The local view supports creating these specific collection types:</span></span>

- <span data-ttu-id="83db8-318"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.LocalView%601.ToObservableCollection?displayProperty=nameWithType> 返回 <xref:System.Collections.ObjectModel.ObservableCollection%601> WPF 数据绑定的。</span><span class="sxs-lookup"><span data-stu-id="83db8-318"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.LocalView%601.ToObservableCollection?displayProperty=nameWithType> returns an <xref:System.Collections.ObjectModel.ObservableCollection%601> for WPF data binding.</span></span>
- <span data-ttu-id="83db8-319"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.LocalView%601.ToBindingList?displayProperty=nameWithType> 返回 <xref:System.ComponentModel.BindingList%601> Windows 窗体数据绑定。</span><span class="sxs-lookup"><span data-stu-id="83db8-319"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.LocalView%601.ToBindingList?displayProperty=nameWithType> returns a <xref:System.ComponentModel.BindingList%601> for Windows Forms data binding.</span></span>

<span data-ttu-id="83db8-320">例如：</span><span class="sxs-lookup"><span data-stu-id="83db8-320">For example:</span></span>

<!--
        ObservableCollection<Post> observableCollection = context.Posts.Local.ToObservableCollection();
        BindingList<Post> bindingList = context.Posts.Local.ToBindingList();
-->
[!code-csharp[Using_DbSet_Local_to_query_tracked_entities_4](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_DbSet_Local_to_query_tracked_entities_4)]

<span data-ttu-id="83db8-321">有关与 EF Core 的 WPF 数据绑定的详细信息，请参阅 [Wpf 入门](xref:core/get-started/wpf) 。</span><span class="sxs-lookup"><span data-stu-id="83db8-321">See [Get Started with WPF](xref:core/get-started/wpf) for more information on WPF data binding with EF Core.</span></span>

> [!TIP]
> <span data-ttu-id="83db8-322">给定 DbSet 实例的本地视图在第一次访问后延迟创建，然后进行缓存。</span><span class="sxs-lookup"><span data-stu-id="83db8-322">The local view for a given DbSet instance is created lazily when first accessed and then cached.</span></span> <span data-ttu-id="83db8-323">LocalView 创建本身的速度很快，并且不会占用大量内存。</span><span class="sxs-lookup"><span data-stu-id="83db8-323">LocalView creation itself is fast and it does not use significant memory.</span></span> <span data-ttu-id="83db8-324">不过，它确实调用了 [DetectChanges](xref:core/change-tracking/change-detection)，这对于大量实体可能会很慢。</span><span class="sxs-lookup"><span data-stu-id="83db8-324">However, it does call [DetectChanges](xref:core/change-tracking/change-detection), which can be slow for large numbers of entities.</span></span> <span data-ttu-id="83db8-325">和创建的集合 `ToObservableCollection` `ToBindingList` 还会按延迟创建，然后进行缓存。</span><span class="sxs-lookup"><span data-stu-id="83db8-325">The collections created by `ToObservableCollection` and `ToBindingList` are also created lazily and and then cached.</span></span> <span data-ttu-id="83db8-326">这两种方法都创建新的集合，这可能会很慢，并且会在涉及数千个实体时使用大量内存。</span><span class="sxs-lookup"><span data-stu-id="83db8-326">Both of these methods create new collections, which can be slow and use a lot of memory when thousands of entities are involved.</span></span>
