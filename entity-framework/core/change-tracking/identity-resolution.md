---
title: 标识解析-EF Core
description: 使用主键值将多个实体实例解析为单个实例
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/identity-resolution
ms.openlocfilehash: d4c8f935c8d0ab92eaecd8fc7a4156bd824713d4
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543609"
---
# <a name="identity-resolution-in-ef-core"></a><span data-ttu-id="a2471-103">EF Core 中的标识解析</span><span class="sxs-lookup"><span data-stu-id="a2471-103">Identity Resolution in EF Core</span></span>

<span data-ttu-id="a2471-104"><xref:Microsoft.EntityFrameworkCore.DbContext>只能使用任意给定的主键值跟踪一个实体实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-104">A <xref:Microsoft.EntityFrameworkCore.DbContext> can only track one entity instance with any given primary key value.</span></span> <span data-ttu-id="a2471-105">这意味着，必须将具有相同键值的实体的多个实例解析为单个实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-105">This means multiple instances of an entity with the same key value must be resolved to a single instance.</span></span> <span data-ttu-id="a2471-106">这称为 "标识解析"。</span><span class="sxs-lookup"><span data-stu-id="a2471-106">This is called "identity resolution".</span></span> <span data-ttu-id="a2471-107">标识解析可确保 Entity Framework Core (EF Core) 跟踪一致的关系图，而不考虑实体的关系或属性值。</span><span class="sxs-lookup"><span data-stu-id="a2471-107">Identity resolution ensures Entity Framework Core (EF Core) is tracking a consistent graph with no ambiguities about the relationships or property values of the entities.</span></span>

> [!TIP]
> <span data-ttu-id="a2471-108">本文档假设了解 EF Core 更改跟踪的实体状态和基础知识。</span><span class="sxs-lookup"><span data-stu-id="a2471-108">This document assumes that entity states and the basics of EF Core change tracking are understood.</span></span> <span data-ttu-id="a2471-109">有关这些主题的详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。</span><span class="sxs-lookup"><span data-stu-id="a2471-109">See [Change Tracking in EF Core](xref:core/change-tracking/index) for more information on these topics.</span></span>

> [!TIP]
> <span data-ttu-id="a2471-110">通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/IdentityResolutionInEFCore)，你可运行并调试到本文档中的所有代码。</span><span class="sxs-lookup"><span data-stu-id="a2471-110">You can run and debug into all the code in this document by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/IdentityResolutionInEFCore).</span></span>

## <a name="introduction"></a><span data-ttu-id="a2471-111">简介</span><span class="sxs-lookup"><span data-stu-id="a2471-111">Introduction</span></span>

<span data-ttu-id="a2471-112">下面的代码查询实体，然后尝试附加具有相同的主键值的不同实例：</span><span class="sxs-lookup"><span data-stu-id="a2471-112">The following code queries for an entity and then attempts to attach a different instance with the same primary key value:</span></span>

<!--
            using var context = new BlogsContext();

            var blogA = context.Blogs.Single(e => e.Id == 1);
            var blogB = new Blog { Id = 1, Name = ".NET Blog (All new!)" };

            try
            {
                context.Update(blogB); // This will throw
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
            }
-->
[!code-csharp[Identity_Resolution_in_EF_Core_1](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Identity_Resolution_in_EF_Core_1)]

<span data-ttu-id="a2471-113">运行此代码将导致以下异常：</span><span class="sxs-lookup"><span data-stu-id="a2471-113">Running this code results in the following exception:</span></span>

> <span data-ttu-id="a2471-114">InvalidOperationException：无法跟踪实体类型 "博客" 的实例，因为已在跟踪具有密钥值 "{Id： 1}" 的另一个实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-114">System.InvalidOperationException: The instance of entity type 'Blog' cannot be tracked because another instance with the key value '{Id: 1}' is already being tracked.</span></span> <span data-ttu-id="a2471-115">附加现有实体时，请确保只附加一个具有给定键值的实体实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-115">When attaching existing entities, ensure that only one entity instance with a given key value is attached.</span></span>

<span data-ttu-id="a2471-116">EF Core 需要单个实例，原因如下：</span><span class="sxs-lookup"><span data-stu-id="a2471-116">EF Core requires a single instance because:</span></span>

- <span data-ttu-id="a2471-117">多个实例的属性值可能不同。</span><span class="sxs-lookup"><span data-stu-id="a2471-117">Property values may be different between multiple instances.</span></span> <span data-ttu-id="a2471-118">更新数据库时，EF Core 需要知道要使用的属性值。</span><span class="sxs-lookup"><span data-stu-id="a2471-118">When updating the database, EF Core needs to know which property values to use.</span></span>
- <span data-ttu-id="a2471-119">与其他实体的关系在多个实例之间可能不同。</span><span class="sxs-lookup"><span data-stu-id="a2471-119">Relationships with other entities may be different between multiple instances.</span></span> <span data-ttu-id="a2471-120">例如，"blogA" 可能与 "blogB" 不同的发布集合相关。</span><span class="sxs-lookup"><span data-stu-id="a2471-120">For example, "blogA" may be related to a different collection of posts than "blogB".</span></span>

<span data-ttu-id="a2471-121">在以下情况下通常会遇到上述异常：</span><span class="sxs-lookup"><span data-stu-id="a2471-121">The exception above is commonly encountered in these situations:</span></span>

- <span data-ttu-id="a2471-122">尝试更新实体时</span><span class="sxs-lookup"><span data-stu-id="a2471-122">When attempting to update an entity</span></span>
- <span data-ttu-id="a2471-123">尝试跟踪实体的序列化图形时</span><span class="sxs-lookup"><span data-stu-id="a2471-123">When attempting to track a serialized graph of entities</span></span>
- <span data-ttu-id="a2471-124">未能设置不自动生成的键值时</span><span class="sxs-lookup"><span data-stu-id="a2471-124">When failing to set a key value that is not automatically generated</span></span>
- <span data-ttu-id="a2471-125">为多个工作单元重复使用 DbContext 实例时</span><span class="sxs-lookup"><span data-stu-id="a2471-125">When reusing a DbContext instance for multiple units-of-work</span></span>

<span data-ttu-id="a2471-126">以下各节将讨论其中的每种情况。</span><span class="sxs-lookup"><span data-stu-id="a2471-126">Each of these situations is discussed in the following sections.</span></span>

## <a name="updating-an-entity"></a><span data-ttu-id="a2471-127">更新实体</span><span class="sxs-lookup"><span data-stu-id="a2471-127">Updating an entity</span></span>

<span data-ttu-id="a2471-128">有多种不同的方法可使用新值更新实体，如 [EF Core 中更改跟踪中](xref:core/change-tracking/index) 所述，并 [显式跟踪实体](xref:core/change-tracking/explicit-tracking)。</span><span class="sxs-lookup"><span data-stu-id="a2471-128">There are several different approaches to update an entity with new values, as covered in [Change Tracking in EF Core](xref:core/change-tracking/index) and [Explicitly Tracking Entities](xref:core/change-tracking/explicit-tracking).</span></span> <span data-ttu-id="a2471-129">下面的标识解析上下文中概述了这些方法。</span><span class="sxs-lookup"><span data-stu-id="a2471-129">These approaches are outlined below in the context of identity resolution.</span></span> <span data-ttu-id="a2471-130">需要注意的一点是，每个方法都使用查询或对或的调用 `Update` `Attach` ，但 **_永远不会同时使用这两_** 种方法。</span><span class="sxs-lookup"><span data-stu-id="a2471-130">An important point to notice is that each of the approaches use either a query or a call to one of `Update` or `Attach`, but **_never both_**.</span></span>

### <a name="call-update"></a><span data-ttu-id="a2471-131">调用更新</span><span class="sxs-lookup"><span data-stu-id="a2471-131">Call Update</span></span>

<span data-ttu-id="a2471-132">通常，要更新的实体不会来自要用于 SaveChanges 的 DbContext 上的查询。</span><span class="sxs-lookup"><span data-stu-id="a2471-132">Often the entity to update does not come from a query on the DbContext that we are going to use for SaveChanges.</span></span> <span data-ttu-id="a2471-133">例如，在 web 应用程序中，可以根据 POST 请求中的信息创建实体实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-133">For example, in a web application, an entity instance may be created from the information in a POST request.</span></span> <span data-ttu-id="a2471-134">处理此情况的最简单方法是使用 <xref:Microsoft.EntityFrameworkCore.DbContext.Update%2A?displayProperty=nameWithType> 或 <xref:Microsoft.EntityFrameworkCore.DbSet%601.Update%2A?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="a2471-134">The simplest way to handle this is to use <xref:Microsoft.EntityFrameworkCore.DbContext.Update%2A?displayProperty=nameWithType> or <xref:Microsoft.EntityFrameworkCore.DbSet%601.Update%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="a2471-135">例如：</span><span class="sxs-lookup"><span data-stu-id="a2471-135">For example:</span></span>

<!--
    public static void UpdateFromHttpPost1(Blog blog)
    {
        using var context = new BlogsContext();

        context.Update(blog);

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_1](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_1)]

<span data-ttu-id="a2471-136">在这种情况下：</span><span class="sxs-lookup"><span data-stu-id="a2471-136">In this case:</span></span>

- <span data-ttu-id="a2471-137">仅创建实体的单个实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-137">Only a single instance of the entity is created.</span></span>
- <span data-ttu-id="a2471-138">在进行更新的过程中， **不** 会从数据库中查询实体实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-138">The entity instance is **not** queried from the database as part of making the update.</span></span>
- <span data-ttu-id="a2471-139">所有属性值都将在数据库中更新，不管它们是否确实已更改。</span><span class="sxs-lookup"><span data-stu-id="a2471-139">All property values will be updated in the database, regardless of whether they have actually changed or not.</span></span>
- <span data-ttu-id="a2471-140">进行一个数据库往返。</span><span class="sxs-lookup"><span data-stu-id="a2471-140">One database round-trip is made.</span></span>

### <a name="query-then-apply-changes"></a><span data-ttu-id="a2471-141">查询，然后应用更改</span><span class="sxs-lookup"><span data-stu-id="a2471-141">Query then apply changes</span></span>

<span data-ttu-id="a2471-142">通常不知道在通过 POST 请求中的信息创建实体时，哪些属性值实际上已更改，或类似。</span><span class="sxs-lookup"><span data-stu-id="a2471-142">Usually it is not known which property values have actually been changed when an entity is created from information in a POST request or similar.</span></span> <span data-ttu-id="a2471-143">通常，只需更新数据库中的所有值，就像在前面的示例中一样。</span><span class="sxs-lookup"><span data-stu-id="a2471-143">Often it is fine to just update all values in the database, as we did in the previous example.</span></span> <span data-ttu-id="a2471-144">但是，如果应用程序正在处理多个实体，并且只有少量这些实体具有实际更改，则可以限制发送的更新。</span><span class="sxs-lookup"><span data-stu-id="a2471-144">However, if the application is handling many entities and only a small number of those have actual changes, then it may be useful to limit the updates sent.</span></span> <span data-ttu-id="a2471-145">这可以通过执行查询来跟踪当前存在于数据库中的实体，然后将更改应用于这些被跟踪的实体，来实现此目的。</span><span class="sxs-lookup"><span data-stu-id="a2471-145">This can be achieved by executing a query to track the entities as they currently exist in the database, and then applying changes to these tracked entities.</span></span> <span data-ttu-id="a2471-146">例如：</span><span class="sxs-lookup"><span data-stu-id="a2471-146">For example:</span></span>

<!--
    public static void UpdateFromHttpPost2(Blog blog)
    {
        using var context = new BlogsContext();

        var trackedBlog = context.Blogs.Find(blog.Id);

        trackedBlog.Name = blog.Name;
        trackedBlog.Summary = blog.Summary;

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_2](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_2)]

<span data-ttu-id="a2471-147">在这种情况下：</span><span class="sxs-lookup"><span data-stu-id="a2471-147">In this case:</span></span>

- <span data-ttu-id="a2471-148">只跟踪实体的单个实例;查询从数据库返回的 `Find` 。</span><span class="sxs-lookup"><span data-stu-id="a2471-148">Only a single instance of the entity is tracked; the one that is returned from the database by the `Find` query.</span></span>
- <span data-ttu-id="a2471-149">`Update``Attach`**不** 使用、等。</span><span class="sxs-lookup"><span data-stu-id="a2471-149">`Update`, `Attach`, etc. are **not** used.</span></span>
- <span data-ttu-id="a2471-150">仅在数据库中更新实际更改的属性值。</span><span class="sxs-lookup"><span data-stu-id="a2471-150">Only property values that have actually changed are updated in the database.</span></span>
- <span data-ttu-id="a2471-151">进行两次数据库往返。</span><span class="sxs-lookup"><span data-stu-id="a2471-151">Two database round-trips are made.</span></span>

<span data-ttu-id="a2471-152">EF Core 具有一些用于传输属性值的帮助器。</span><span class="sxs-lookup"><span data-stu-id="a2471-152">EF Core has some helpers for transferring property values like this.</span></span> <span data-ttu-id="a2471-153">例如， <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues.SetValues%2A?displayProperty=nameWithType> 将复制给定对象中的所有值，并在跟踪的对象上设置这些值：</span><span class="sxs-lookup"><span data-stu-id="a2471-153">For example, <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues.SetValues%2A?displayProperty=nameWithType> will copy all the values from the given object and set them on the tracked object:</span></span>

<!--
    public static void UpdateFromHttpPost3(Blog blog)
    {
        using var context = new BlogsContext();

        var trackedBlog = context.Blogs.Find(blog.Id);

        context.Entry(trackedBlog).CurrentValues.SetValues(blog);

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_3](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_3)]

<span data-ttu-id="a2471-154">`SetValues` 接受各种对象类型，包括数据传输对象 (Dto) ，其属性名称与实体类型的属性相匹配。</span><span class="sxs-lookup"><span data-stu-id="a2471-154">`SetValues` accepts various object types, including data transfer objects (DTOs) with property names that match the properties of the entity type.</span></span> <span data-ttu-id="a2471-155">例如：</span><span class="sxs-lookup"><span data-stu-id="a2471-155">For example:</span></span>

<!--
    public static void UpdateFromHttpPost4(BlogDto dto)
    {
        using var context = new BlogsContext();

        var trackedBlog = context.Blogs.Find(dto.Id);

        context.Entry(trackedBlog).CurrentValues.SetValues(dto);

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_4](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_4)]

<span data-ttu-id="a2471-156">或包含属性值的名称/值项的字典：</span><span class="sxs-lookup"><span data-stu-id="a2471-156">Or a dictionary with name/value entries for the property values:</span></span>

<!--
    public static void UpdateFromHttpPost5(Dictionary<string, object> propertyValues)
    {
        using var context = new BlogsContext();

        var trackedBlog = context.Blogs.Find(propertyValues["Id"]);

        context.Entry(trackedBlog).CurrentValues.SetValues(propertyValues);

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_5](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_5)]

<span data-ttu-id="a2471-157">有关使用此类属性值的详细信息，请参阅 [访问跟踪的实体](xref:core/change-tracking/entity-entries) 。</span><span class="sxs-lookup"><span data-stu-id="a2471-157">See [Accessing tracked entities](xref:core/change-tracking/entity-entries) for more information on working with property values like this.</span></span>

### <a name="use-original-values"></a><span data-ttu-id="a2471-158">使用原始值</span><span class="sxs-lookup"><span data-stu-id="a2471-158">Use original values</span></span>

<span data-ttu-id="a2471-159">到目前为止，每种方法都已在进行更新之前执行了查询，或者更新了所有属性值，无论它们是否已更改。</span><span class="sxs-lookup"><span data-stu-id="a2471-159">So far each approach has either executed a query before making the update, or updated all property values regardless of whether or not they have changed.</span></span> <span data-ttu-id="a2471-160">若要更新不作为更新的一部分进行查询更改的值，需要有关哪些属性值已更改的特定信息。</span><span class="sxs-lookup"><span data-stu-id="a2471-160">To update only values that have changed without querying as part of the update requires specific information about which property values have changed.</span></span> <span data-ttu-id="a2471-161">获取此信息的常见方法是在 HTTP Post 或类似中发送回当前值和原始值。</span><span class="sxs-lookup"><span data-stu-id="a2471-161">A common way to get this information is to send back both the current and original values in the HTTP Post or similar.</span></span> <span data-ttu-id="a2471-162">例如：</span><span class="sxs-lookup"><span data-stu-id="a2471-162">For example:</span></span>

<!--
    public static void UpdateFromHttpPost6(Blog blog, Dictionary<string, object> originalValues)
    {
        using var context = new BlogsContext();

        context.Attach(blog);
        context.Entry(blog).OriginalValues.SetValues(originalValues);

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_6](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_6)]

<span data-ttu-id="a2471-163">在此代码中，首先附加带有修改值的实体。</span><span class="sxs-lookup"><span data-stu-id="a2471-163">In this code the entity with modified values is first attached.</span></span> <span data-ttu-id="a2471-164">这会导致 EF Core 跟踪状态中的实体 `Unchanged` ; 即，不将属性值标记为已修改。</span><span class="sxs-lookup"><span data-stu-id="a2471-164">This causes EF Core to track the entity in the `Unchanged` state; that is, with no property values marked as modified.</span></span> <span data-ttu-id="a2471-165">然后，将原始值的字典应用到此跟踪实体。</span><span class="sxs-lookup"><span data-stu-id="a2471-165">The dictionary of original values is then applied to this tracked entity.</span></span> <span data-ttu-id="a2471-166">这会将不同的当前值和原始值标记为已修改的属性。</span><span class="sxs-lookup"><span data-stu-id="a2471-166">This will mark as modified properties with different current and original values.</span></span> <span data-ttu-id="a2471-167">具有相同的当前值和原始值的属性将不会标记为已修改。</span><span class="sxs-lookup"><span data-stu-id="a2471-167">Properties that have the same current and original values will not be marked as modified.</span></span>

<span data-ttu-id="a2471-168">在这种情况下：</span><span class="sxs-lookup"><span data-stu-id="a2471-168">In this case:</span></span>

- <span data-ttu-id="a2471-169">只跟踪实体的单个实例，并使用 Attach。</span><span class="sxs-lookup"><span data-stu-id="a2471-169">Only a single instance of the entity is tracked, using Attach.</span></span>
- <span data-ttu-id="a2471-170">在进行更新的过程中， **不** 会从数据库中查询实体实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-170">The entity instance is **not** queried from the database as part of making the update.</span></span>
- <span data-ttu-id="a2471-171">应用原始值可确保在数据库中仅更新实际更改的属性值。</span><span class="sxs-lookup"><span data-stu-id="a2471-171">Applying the original values ensures that only property values that have actually changed are updated in the database.</span></span>
- <span data-ttu-id="a2471-172">进行一个数据库往返。</span><span class="sxs-lookup"><span data-stu-id="a2471-172">One database round-trip is made.</span></span>

<span data-ttu-id="a2471-173">与上一节中的示例一样，原始值不必作为字典传递;实体实例或 DTO 也将起作用。</span><span class="sxs-lookup"><span data-stu-id="a2471-173">As with the examples in the previous section, the original values do not have to passed as a dictionary; an entity instance or DTO will also work.</span></span>

> [!TIP]
> <span data-ttu-id="a2471-174">虽然这种方法有吸引力，但它确实要求向 web 客户端发送和从 web 客户端发送实体的原始值。</span><span class="sxs-lookup"><span data-stu-id="a2471-174">While this approach has appealing characteristics, it does require sending the entity's original values to and from the web client.</span></span> <span data-ttu-id="a2471-175">仔细考虑这种额外的复杂性是否值得。对于许多应用程序而言，一种更简单的方法更实用。</span><span class="sxs-lookup"><span data-stu-id="a2471-175">Carefully consider whether this extra complexity is worth the benefits; for many applications one of the simpler approaches is more pragmatic.</span></span>

## <a name="attaching-a-serialized-graph"></a><span data-ttu-id="a2471-176">附加序列化关系图</span><span class="sxs-lookup"><span data-stu-id="a2471-176">Attaching a serialized graph</span></span>

<span data-ttu-id="a2471-177">EF Core 使用通过外键和导航属性连接的实体图形，如 [更改外键和](xref:core/change-tracking/relationship-changes)导航中所述。</span><span class="sxs-lookup"><span data-stu-id="a2471-177">EF Core works with graphs of entities connected via foreign keys and navigation properties, as described in [Changing Foreign Keys and Navigations](xref:core/change-tracking/relationship-changes).</span></span> <span data-ttu-id="a2471-178">如果这些图形是使用在 EF Core 之外创建的，例如，在 JSON 文件中，则它们可以具有同一实体的多个实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-178">If these graphs are created outside of EF Core using, for example, from a JSON file, then they can have multiple instances of the same entity.</span></span> <span data-ttu-id="a2471-179">在跟踪图形之前，需要将这些重复项解析为单个实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-179">These duplicates need to be resolved into single instances before the graph can be tracked.</span></span>

### <a name="graphs-with-no-duplicates"></a><span data-ttu-id="a2471-180">不包含重复项的关系图</span><span class="sxs-lookup"><span data-stu-id="a2471-180">Graphs with no duplicates</span></span>

<span data-ttu-id="a2471-181">在进一步操作之前，请务必确认：</span><span class="sxs-lookup"><span data-stu-id="a2471-181">Before going any further it is important to recognize that:</span></span>

- <span data-ttu-id="a2471-182">序列化程序通常具有在关系图中处理循环和重复实例的选项。</span><span class="sxs-lookup"><span data-stu-id="a2471-182">Serializers often have options for handling loops and duplicate instances in the graph.</span></span>
- <span data-ttu-id="a2471-183">选择用作图形根的对象通常有助于减少或删除重复项。</span><span class="sxs-lookup"><span data-stu-id="a2471-183">The choice of object used as the graph root can often help reduce or remove duplicates.</span></span>

<span data-ttu-id="a2471-184">如果可能，请使用序列化选项，并选择不会导致重复的根。</span><span class="sxs-lookup"><span data-stu-id="a2471-184">If possible, use serialization options and choose roots that do not result in duplicates.</span></span> <span data-ttu-id="a2471-185">例如，下面的代码使用 [Json.NET](https://www.nuget.org/packages/Newtonsoft.Json/) 来序列化一个博客列表，其中每个博客都包含其关联的文章：</span><span class="sxs-lookup"><span data-stu-id="a2471-185">For example, the following code uses [Json.NET](https://www.nuget.org/packages/Newtonsoft.Json/) to serialize a list of blogs each with its associated posts:</span></span>

<!--
            using var context = new BlogsContext();

            var blogs = context.Blogs.Include(e => e.Posts).ToList();

            var serialized = JsonConvert.SerializeObject(
                blogs,
                new JsonSerializerSettings
                {
                    ReferenceLoopHandling = ReferenceLoopHandling.Ignore,
                    Formatting = Formatting.Indented
                });

            Console.WriteLine(serialized);
-->
[!code-csharp[Attaching_a_serialized_graph_1a](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_1a)]

<span data-ttu-id="a2471-186">此代码生成的 JSON 是：</span><span class="sxs-lookup"><span data-stu-id="a2471-186">The JSON generated from this code is:</span></span>

```json
[
  {
    "Id": 1,
    "Name": ".NET Blog",
    "Summary": "Posts about .NET",
    "Posts": [
      {
        "Id": 1,
        "Title": "Announcing the Release of EF Core 5.0",
        "Content": "Announcing the release of EF Core 5.0, a full featured cross-platform...",
        "BlogId": 1
      },
      {
        "Id": 2,
        "Title": "Announcing F# 5",
        "Content": "F# 5 is the latest version of F#, the functional programming language...",
        "BlogId": 1
      }
    ]
  },
  {
    "Id": 2,
    "Name": "Visual Studio Blog",
    "Summary": "Posts about Visual Studio",
    "Posts": [
      {
        "Id": 3,
        "Title": "Disassembly improvements for optimized managed debugging",
        "Content": "If you are focused on squeezing out the last bits of performance for your .NET service or...",
        "BlogId": 2
      },
      {
        "Id": 4,
        "Title": "Database Profiling with Visual Studio",
        "Content": "Examine when database queries were executed and measure how long the take using...",
        "BlogId": 2
      }
    ]
  }
]
```

<span data-ttu-id="a2471-187">请注意，JSON 中没有重复的博客或文章。</span><span class="sxs-lookup"><span data-stu-id="a2471-187">Notice that there are no duplicate blogs or posts in the JSON.</span></span> <span data-ttu-id="a2471-188">这意味着，对的简单调用将可用于 `Update` 更新数据库中的这些实体：</span><span class="sxs-lookup"><span data-stu-id="a2471-188">This means that simple calls to `Update` will work to update these entities in the database:</span></span>

<!--
        public static void UpdateBlogsFromJson(string json)
        {
            using var context = new BlogsContext();

            var blogs = JsonConvert.DeserializeObject<List<Blog>>(json);

            foreach (var blog in blogs)
            {
                context.Update(blog);
            }

            context.SaveChanges();
        }
-->
[!code-csharp[Attaching_a_serialized_graph_1b](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_1b)]

### <a name="handling-duplicates"></a><span data-ttu-id="a2471-189">处理重复项</span><span class="sxs-lookup"><span data-stu-id="a2471-189">Handling duplicates</span></span>

<span data-ttu-id="a2471-190">上一示例中的代码将每个博客及其关联的文章序列化。</span><span class="sxs-lookup"><span data-stu-id="a2471-190">The code in the previous example serialized each blog with its associated posts.</span></span> <span data-ttu-id="a2471-191">如果更改此项以序列化每个 post 及其关联的博客，则会将重复项引入序列化的 JSON。</span><span class="sxs-lookup"><span data-stu-id="a2471-191">If this is changed to serialize each post with its associated blog, then duplicates are introduced into the serialized JSON.</span></span> <span data-ttu-id="a2471-192">例如：</span><span class="sxs-lookup"><span data-stu-id="a2471-192">For example:</span></span>

<!--
            using var context = new BlogsContext();

            var posts = context.Posts.Include(e => e.Blog).ToList();

            var serialized = JsonConvert.SerializeObject(
                posts,
                new JsonSerializerSettings
                {
                    ReferenceLoopHandling = ReferenceLoopHandling.Ignore,
                    Formatting = Formatting.Indented
                });

            Console.WriteLine(serialized);
-->
[!code-csharp[Attaching_a_serialized_graph_2](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_2)]

<span data-ttu-id="a2471-193">序列化的 JSON 现在如下所示：</span><span class="sxs-lookup"><span data-stu-id="a2471-193">The serialized JSON now looks like this:</span></span>

```json
[
  {
    "Id": 1,
    "Title": "Announcing the Release of EF Core 5.0",
    "Content": "Announcing the release of EF Core 5.0, a full featured cross-platform...",
    "BlogId": 1,
    "Blog": {
      "Id": 1,
      "Name": ".NET Blog",
      "Summary": "Posts about .NET",
      "Posts": [
        {
          "Id": 2,
          "Title": "Announcing F# 5",
          "Content": "F# 5 is the latest version of F#, the functional programming language...",
          "BlogId": 1
        }
      ]
    }
  },
  {
    "Id": 2,
    "Title": "Announcing F# 5",
    "Content": "F# 5 is the latest version of F#, the functional programming language...",
    "BlogId": 1,
    "Blog": {
      "Id": 1,
      "Name": ".NET Blog",
      "Summary": "Posts about .NET",
      "Posts": [
        {
          "Id": 1,
          "Title": "Announcing the Release of EF Core 5.0",
          "Content": "Announcing the release of EF Core 5.0, a full featured cross-platform...",
          "BlogId": 1
        }
      ]
    }
  },
  {
    "Id": 3,
    "Title": "Disassembly improvements for optimized managed debugging",
    "Content": "If you are focused on squeezing out the last bits of performance for your .NET service or...",
    "BlogId": 2,
    "Blog": {
      "Id": 2,
      "Name": "Visual Studio Blog",
      "Summary": "Posts about Visual Studio",
      "Posts": [
        {
          "Id": 4,
          "Title": "Database Profiling with Visual Studio",
          "Content": "Examine when database queries were executed and measure how long the take using...",
          "BlogId": 2
        }
      ]
    }
  },
  {
    "Id": 4,
    "Title": "Database Profiling with Visual Studio",
    "Content": "Examine when database queries were executed and measure how long the take using...",
    "BlogId": 2,
    "Blog": {
      "Id": 2,
      "Name": "Visual Studio Blog",
      "Summary": "Posts about Visual Studio",
      "Posts": [
        {
          "Id": 3,
          "Title": "Disassembly improvements for optimized managed debugging",
          "Content": "If you are focused on squeezing out the last bits of performance for your .NET service or...",
          "BlogId": 2
        }
      ]
    }
  }
]
```

<span data-ttu-id="a2471-194">请注意，该图形现在包含多个具有相同键值的博客实例，还包括多个具有相同密钥值的 Post 实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-194">Notice that the graph now includes multiple Blog instances with the same key value, as well as multiple Post instance with the same key value.</span></span> <span data-ttu-id="a2471-195">尝试跟踪此关系图（如前面的示例所做的操作）将引发：</span><span class="sxs-lookup"><span data-stu-id="a2471-195">Attempting to track this graph like we did in the previous example will throw:</span></span>

> <span data-ttu-id="a2471-196">InvalidOperationException：无法跟踪实体类型 "Post" 的实例，因为已在跟踪具有键值 "{Id： 2}" 的另一个实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-196">System.InvalidOperationException: The instance of entity type 'Post' cannot be tracked because another instance with the key value '{Id: 2}' is already being tracked.</span></span> <span data-ttu-id="a2471-197">附加现有实体时，请确保只附加一个具有给定键值的实体实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-197">When attaching existing entities, ensure that only one entity instance with a given key value is attached.</span></span>

<span data-ttu-id="a2471-198">可以通过两种方式解决此问题：</span><span class="sxs-lookup"><span data-stu-id="a2471-198">We can fix this in two ways:</span></span>

- <span data-ttu-id="a2471-199">使用保留引用的 JSON 序列化选项</span><span class="sxs-lookup"><span data-stu-id="a2471-199">Use JSON serialization options that preserve references</span></span>
- <span data-ttu-id="a2471-200">跟踪图形时执行标识解析</span><span class="sxs-lookup"><span data-stu-id="a2471-200">Perform identity resolution while the graph is being tracked</span></span>

#### <a name="preserve-references"></a><span data-ttu-id="a2471-201">保留引用</span><span class="sxs-lookup"><span data-stu-id="a2471-201">Preserve references</span></span>

<span data-ttu-id="a2471-202">Json.NET 提供了 `PreserveReferencesHandling` 用于处理此的选项。</span><span class="sxs-lookup"><span data-stu-id="a2471-202">Json.NET provides the `PreserveReferencesHandling` option to handle this.</span></span> <span data-ttu-id="a2471-203">例如：</span><span class="sxs-lookup"><span data-stu-id="a2471-203">For example:</span></span>

<!--
            var serialized = JsonConvert.SerializeObject(
                posts,
                new JsonSerializerSettings
                {
                    PreserveReferencesHandling = PreserveReferencesHandling.All,
                    Formatting = Formatting.Indented
                });
-->
[!code-csharp[Attaching_a_serialized_graph_3](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_3)]

<span data-ttu-id="a2471-204">生成的 JSON 现在如下所示：</span><span class="sxs-lookup"><span data-stu-id="a2471-204">The resulting JSON now looks like this:</span></span>

```json
{
  "$id": "1",
  "$values": [
    {
      "$id": "2",
      "Id": 1,
      "Title": "Announcing the Release of EF Core 5.0",
      "Content": "Announcing the release of EF Core 5.0, a full featured cross-platform...",
      "BlogId": 1,
      "Blog": {
        "$id": "3",
        "Id": 1,
        "Name": ".NET Blog",
        "Summary": "Posts about .NET",
        "Posts": [
          {
            "$ref": "2"
          },
          {
            "$id": "4",
            "Id": 2,
            "Title": "Announcing F# 5",
            "Content": "F# 5 is the latest version of F#, the functional programming language...",
            "BlogId": 1,
            "Blog": {
              "$ref": "3"
            }
          }
        ]
      }
    },
    {
      "$ref": "4"
    },
    {
      "$id": "5",
      "Id": 3,
      "Title": "Disassembly improvements for optimized managed debugging",
      "Content": "If you are focused on squeezing out the last bits of performance for your .NET service or...",
      "BlogId": 2,
      "Blog": {
        "$id": "6",
        "Id": 2,
        "Name": "Visual Studio Blog",
        "Summary": "Posts about Visual Studio",
        "Posts": [
          {
            "$ref": "5"
          },
          {
            "$id": "7",
            "Id": 4,
            "Title": "Database Profiling with Visual Studio",
            "Content": "Examine when database queries were executed and measure how long the take using...",
            "BlogId": 2,
            "Blog": {
              "$ref": "6"
            }
          }
        ]
      }
    },
    {
      "$ref": "7"
    }
  ]
}
```

<span data-ttu-id="a2471-205">请注意，此 JSON 已将重复项替换为类似于 `"$ref": "5"` 图中现有实例的引用。</span><span class="sxs-lookup"><span data-stu-id="a2471-205">Notice that this JSON has replaced duplicates with references like `"$ref": "5"` that refer to the already existing instance in the graph.</span></span> <span data-ttu-id="a2471-206">此图可再次使用对的简单调用进行跟踪 `Update` ，如上所示。</span><span class="sxs-lookup"><span data-stu-id="a2471-206">This graph can again be tracked using the simple calls to `Update`, as shown above.</span></span>

<span data-ttu-id="a2471-207"><xref:System.Text.Json> (BCL) 的 .net 基类库中的支持具有类似的选项，该选项将产生相同的结果。</span><span class="sxs-lookup"><span data-stu-id="a2471-207">The <xref:System.Text.Json> support in the .NET base class libraries (BCL) has a similar option which produces the same result.</span></span> <span data-ttu-id="a2471-208">例如：</span><span class="sxs-lookup"><span data-stu-id="a2471-208">For example:</span></span>

<!--
            var serialized = System.Text.Json.JsonSerializer.Serialize(posts, new System.Text.Json.JsonSerializerOptions
            {
                ReferenceHandler = ReferenceHandler.Preserve,
                WriteIndented = true
            });
-->
[!code-csharp[Attaching_a_serialized_graph_4](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_4)]

#### <a name="resolve-duplicates"></a><span data-ttu-id="a2471-209">解决重复项</span><span class="sxs-lookup"><span data-stu-id="a2471-209">Resolve duplicates</span></span>

<span data-ttu-id="a2471-210">如果无法在序列化过程中消除重复项，则 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.TrackGraph%2A?displayProperty=nameWithType> 提供一种方法来处理这种情况。</span><span class="sxs-lookup"><span data-stu-id="a2471-210">If it is not possible to eliminate duplicates in the serialization process, then <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.TrackGraph%2A?displayProperty=nameWithType> provides a way to handle this.</span></span> <span data-ttu-id="a2471-211">TrackGraph 的工作方式类似于 `Add` `Attach` 和， `Update` 只不过它在跟踪之前为每个实体实例生成一个回调。</span><span class="sxs-lookup"><span data-stu-id="a2471-211">TrackGraph works like `Add`, `Attach` and `Update` except that it generates a callback for every entity instance before tracking it.</span></span> <span data-ttu-id="a2471-212">此回调可用于跟踪实体或忽略它。</span><span class="sxs-lookup"><span data-stu-id="a2471-212">This callback can be used to either track the entity or ignore it.</span></span> <span data-ttu-id="a2471-213">例如：</span><span class="sxs-lookup"><span data-stu-id="a2471-213">For example:</span></span>

<!--
        public static void UpdatePostsFromJsonWithIdentityResolution(string json)
        {
            using var context = new BlogsContext();

            var posts = JsonConvert.DeserializeObject<List<Post>>(json);

            foreach (var post in posts)
            {
                context.ChangeTracker.TrackGraph(
                    post, node =>
                        {
                            var keyValue = node.Entry.Property("Id").CurrentValue;
                            var entityType = node.Entry.Metadata;

                            var existingEntity = node.Entry.Context.ChangeTracker.Entries()
                                .FirstOrDefault(
                                    e => Equals(e.Metadata, entityType)
                                         && Equals(e.Property("Id").CurrentValue, keyValue));

                            if (existingEntity == null)
                            {
                                Console.WriteLine($"Tracking {entityType} entity with key value {keyValue}");

                                node.Entry.State = EntityState.Modified;
                            }
                            else
                            {
                                Console.WriteLine($"Discarding duplicate {entityType} entity with key value {keyValue}");
                            }
                        });
            }

            context.SaveChanges();
        }
-->
[!code-csharp[Attaching_a_serialized_graph_5](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_5)]

<span data-ttu-id="a2471-214">对于图形中的每个实体，此代码将：</span><span class="sxs-lookup"><span data-stu-id="a2471-214">For each entity in the graph, this code will:</span></span>

- <span data-ttu-id="a2471-215">查找实体的实体类型和键值</span><span class="sxs-lookup"><span data-stu-id="a2471-215">Find the entity type and key value of the entity</span></span>
- <span data-ttu-id="a2471-216">在更改跟踪器中查找具有此密钥的实体</span><span class="sxs-lookup"><span data-stu-id="a2471-216">Lookup the entity with this key in the change tracker</span></span>
  - <span data-ttu-id="a2471-217">如果找到该实体，则不采取进一步的操作，因为该实体是重复的</span><span class="sxs-lookup"><span data-stu-id="a2471-217">If the entity is found, then no further action is taken as the entity is a duplicate</span></span>
  - <span data-ttu-id="a2471-218">如果找不到该实体，则通过将状态设置为来进行跟踪。 `Modified`</span><span class="sxs-lookup"><span data-stu-id="a2471-218">If the entity is not found, then it is tracked by setting the state to `Modified`</span></span>

<span data-ttu-id="a2471-219">运行此代码的输出为：</span><span class="sxs-lookup"><span data-stu-id="a2471-219">The output from running this code is:</span></span>

```output
Tracking EntityType: Post entity with key value 1
Tracking EntityType: Blog entity with key value 1
Tracking EntityType: Post entity with key value 2
Discarding duplicate EntityType: Post entity with key value 2
Tracking EntityType: Post entity with key value 3
Tracking EntityType: Blog entity with key value 2
Tracking EntityType: Post entity with key value 4
Discarding duplicate EntityType: Post entity with key value 4
```

> [!IMPORTANT]
> <span data-ttu-id="a2471-220">此代码 **假定所有重复项都相同**。</span><span class="sxs-lookup"><span data-stu-id="a2471-220">This code **assumes that all duplicates are identical**.</span></span> <span data-ttu-id="a2471-221">这使得随意选择其中一个重复项来跟踪，同时丢弃其他副本。</span><span class="sxs-lookup"><span data-stu-id="a2471-221">This makes it safe to arbitrarily choose one of the duplicates to track while discarding the others.</span></span> <span data-ttu-id="a2471-222">如果重复项可能不同，则代码需要决定如何确定使用哪一个，以及如何将属性和导航值组合在一起。</span><span class="sxs-lookup"><span data-stu-id="a2471-222">If the duplicates can differ, then the code will need to decide how to determine which one to use, and how to combine property and navigation values together.</span></span>

> [!NOTE]
> <span data-ttu-id="a2471-223">为简单起见，此代码假定每个实体都有一个名为的主键属性 `Id` 。</span><span class="sxs-lookup"><span data-stu-id="a2471-223">For simplicity, this code assumes each entity has a primary key property called `Id`.</span></span> <span data-ttu-id="a2471-224">这可能会整理到抽象基类或接口中。</span><span class="sxs-lookup"><span data-stu-id="a2471-224">This could be codified into an abstract base class or interface.</span></span> <span data-ttu-id="a2471-225">或者，可以从元数据中获取主键属性， <xref:Microsoft.EntityFrameworkCore.Metadata.IEntityType> 以便此代码适用于任何类型的实体。</span><span class="sxs-lookup"><span data-stu-id="a2471-225">Alternately, the primary key property or properties could be obtained from the <xref:Microsoft.EntityFrameworkCore.Metadata.IEntityType> metadata such that this code would work with any type of entity.</span></span>

## <a name="failing-to-set-key-values"></a><span data-ttu-id="a2471-226">未能设置键值</span><span class="sxs-lookup"><span data-stu-id="a2471-226">Failing to set key values</span></span>

<span data-ttu-id="a2471-227">实体类型通常配置为使用 [自动生成的键值](xref:core/modeling/generated-properties)。</span><span class="sxs-lookup"><span data-stu-id="a2471-227">Entity types are often configured to use [automatically generated key values](xref:core/modeling/generated-properties).</span></span> <span data-ttu-id="a2471-228">这是非复合键的整数和 GUID 属性的默认值。</span><span class="sxs-lookup"><span data-stu-id="a2471-228">This is the default for integer and GUID properties of non-composite keys.</span></span> <span data-ttu-id="a2471-229">但是，如果未将该实体类型配置为使用自动生成的键值，则在跟踪该实体之前必须设置显式键值。</span><span class="sxs-lookup"><span data-stu-id="a2471-229">However, if the entity type is not configured to use automatically generated key values, then an explicit key value must be set before tracking the entity.</span></span> <span data-ttu-id="a2471-230">例如，使用以下实体类型：</span><span class="sxs-lookup"><span data-stu-id="a2471-230">For example, using the following entity type:</span></span>

<!--
    public class Pet
    {
        [DatabaseGenerated(DatabaseGeneratedOption.None)]
        public int Id { get; set; }

        public string Name { get; set; }
    }
-->
[!code-csharp[Pet](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Pet)]

<span data-ttu-id="a2471-231">考虑在不设置键值的情况下尝试跟踪两个新实体实例的代码：</span><span class="sxs-lookup"><span data-stu-id="a2471-231">Consider code that attempts to tracker two new entity instances without setting key values:</span></span>

<!--
            using var context = new BlogsContext();

            context.Add(new Pet { Name = "Smokey" });

            try
            {
                context.Add(new Pet { Name = "Clippy" }); // This will throw
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
            }
-->
[!code-csharp[Failing_to_set_key_values_1](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Failing_to_set_key_values_1)]

<span data-ttu-id="a2471-232">此代码将引发：</span><span class="sxs-lookup"><span data-stu-id="a2471-232">This code will throw:</span></span>

> <span data-ttu-id="a2471-233">InvalidOperationException：无法跟踪实体类型 "Pet" 的实例，因为已在跟踪具有密钥值 "{Id： 0}" 的另一个实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-233">System.InvalidOperationException: The instance of entity type 'Pet' cannot be tracked because another instance with the key value '{Id: 0}' is already being tracked.</span></span> <span data-ttu-id="a2471-234">附加现有实体时，请确保只附加一个具有给定键值的实体实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-234">When attaching existing entities, ensure that only one entity instance with a given key value is attached.</span></span>

<span data-ttu-id="a2471-235">解决此问题的方法是显式设置键值，或将 key 属性配置为使用生成的键值。</span><span class="sxs-lookup"><span data-stu-id="a2471-235">The fix for this is to either to set key values explicitly or configure the key property to use generated key values.</span></span> <span data-ttu-id="a2471-236">有关详细信息，请参阅 [生成的值](xref:core/modeling/generated-properties) 。</span><span class="sxs-lookup"><span data-stu-id="a2471-236">See [Generated Values](xref:core/modeling/generated-properties) for more information.</span></span>

## <a name="overusing-a-single-dbcontext-instance"></a><span data-ttu-id="a2471-237">过度使用单个 DbContext 实例</span><span class="sxs-lookup"><span data-stu-id="a2471-237">Overusing a single DbContext instance</span></span>

<span data-ttu-id="a2471-238"><xref:Microsoft.EntityFrameworkCore.DbContext> 旨在表示短期的工作单元，如 [DbContext 初始化和配置](xref:core/dbcontext-configuration/index)中所述，并详细介绍 EF Core 中的 [更改跟踪](xref:core/change-tracking/index)。</span><span class="sxs-lookup"><span data-stu-id="a2471-238"><xref:Microsoft.EntityFrameworkCore.DbContext> is designed to represent a short-lived unit-of-work, as described in [DbContext Initialization and Configuration](xref:core/dbcontext-configuration/index), and elaborated on in [Change Tracking in EF Core](xref:core/change-tracking/index).</span></span> <span data-ttu-id="a2471-239">如果未遵循本指南，则很容易遇到尝试跟踪同一实体的多个实例的情况。</span><span class="sxs-lookup"><span data-stu-id="a2471-239">Not following this guidance makes it is easy to run into situations where an attempt is made to track multiple instances of the same entity.</span></span> <span data-ttu-id="a2471-240">常见示例包括：</span><span class="sxs-lookup"><span data-stu-id="a2471-240">Common examples are:</span></span>

- <span data-ttu-id="a2471-241">使用相同的 DbContext 实例设置测试状态，然后执行测试。</span><span class="sxs-lookup"><span data-stu-id="a2471-241">Using the same DbContext instance to both set up test state and then execute the test.</span></span> <span data-ttu-id="a2471-242">这通常会导致 DbContext 仍跟踪测试设置中的一个实体实例，同时尝试在测试中附加一个新实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-242">This often results in the DbContext still tracking one entity instance from test setup, while then attempting to attach a new instance in the test proper.</span></span> <span data-ttu-id="a2471-243">请改用不同的 DbContext 实例来设置测试状态和测试代码。</span><span class="sxs-lookup"><span data-stu-id="a2471-243">Instead, use a different DbContext instance for setting up test state and the test code proper.</span></span>
- <span data-ttu-id="a2471-244">在存储库或类似的代码中使用共享的 DbContext 实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-244">Using a shared DbContext instance in a repository or similar code.</span></span> <span data-ttu-id="a2471-245">相反，请确保存储库对每个工作单元使用单个 DbContext 实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-245">Instead, make sure your repository uses a single DbContext instance for each unit-of-work.</span></span>

## <a name="identity-resolution-and-queries"></a><span data-ttu-id="a2471-246">标识解析和查询</span><span class="sxs-lookup"><span data-stu-id="a2471-246">Identity resolution and queries</span></span>

<span data-ttu-id="a2471-247">当从查询跟踪实体时，将自动进行标识解析。</span><span class="sxs-lookup"><span data-stu-id="a2471-247">Identity resolution happens automatically when entities are tracked from a query.</span></span> <span data-ttu-id="a2471-248">这意味着，如果已跟踪具有给定键值的实体实例，则使用此现有跟踪实例而不是创建新的实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-248">This means that if an entity instance with a given key value is already tracked, then this existing tracked instance is used instead of creating a new instance.</span></span> <span data-ttu-id="a2471-249">这有一个重要的结果：如果数据在数据库中发生了更改，则不会在查询结果中反映出来。</span><span class="sxs-lookup"><span data-stu-id="a2471-249">This has an important consequence: if the data has changed in the database, then this will not be reflected in the results of the query.</span></span> <span data-ttu-id="a2471-250">这是将新的 DbContext 实例用于每个工作单元的好理由，如 [DbContext 初始化和配置](xref:core/dbcontext-configuration/index)中所述，以及 [EF Core 更改跟踪](xref:core/change-tracking/index)的详细说明。</span><span class="sxs-lookup"><span data-stu-id="a2471-250">This is a good reason to use a new DbContext instance for each unit-of-work, as described in [DbContext Initialization and Configuration](xref:core/dbcontext-configuration/index), and elaborated on in [Change Tracking in EF Core](xref:core/change-tracking/index).</span></span>

> [!IMPORTANT]
> <span data-ttu-id="a2471-251">务必了解 EF Core 始终针对数据库执行 LINQ 查询，并根据数据库中的内容仅返回结果。</span><span class="sxs-lookup"><span data-stu-id="a2471-251">It is important to understand that EF Core always executes a LINQ query on a DbSet against the database and only returns results based on what is in the database.</span></span> <span data-ttu-id="a2471-252">但是，对于跟踪查询，如果已跟踪返回的实体，则使用跟踪的实例，而不是从数据库中的数据创建实例。</span><span class="sxs-lookup"><span data-stu-id="a2471-252">However, for a tracking query, if the entities returned are already tracked, then the tracked instances are used instead of creating a instances from the data in the database.</span></span>

<span data-ttu-id="a2471-253"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Reload><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.GetDatabaseValues>当需要使用数据库中的最新数据刷新跟踪的实体时，可以使用或。</span><span class="sxs-lookup"><span data-stu-id="a2471-253"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Reload> or <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.GetDatabaseValues> can be used when tracked entities need to be refreshed with the latest data from the database.</span></span> <span data-ttu-id="a2471-254">有关详细信息，请参阅 [访问跟踪的实体](xref:core/change-tracking/entity-entries) 。</span><span class="sxs-lookup"><span data-stu-id="a2471-254">See [Accessing Tracked Entities](xref:core/change-tracking/entity-entries) for more information.</span></span>

<span data-ttu-id="a2471-255">与跟踪查询不同，无跟踪查询不会执行标识解析。</span><span class="sxs-lookup"><span data-stu-id="a2471-255">In contrast to tracking queries, no-tracking queries do not perform identity resolution.</span></span> <span data-ttu-id="a2471-256">这意味着，无跟踪查询可以像前面所述的 JSON 序列化案例一样返回重复项。</span><span class="sxs-lookup"><span data-stu-id="a2471-256">This means that no-tracking queries can return duplicates just like in the JSON serialization case described earlier.</span></span> <span data-ttu-id="a2471-257">如果查询结果将被序列化并发送到客户端，这通常不是问题。</span><span class="sxs-lookup"><span data-stu-id="a2471-257">This is usually not an issue if the query results are going to be serialized and sent to the client.</span></span>

> [!TIP]
> <span data-ttu-id="a2471-258">不要定期执行无跟踪查询，然后将返回的实体附加到同一个上下文。</span><span class="sxs-lookup"><span data-stu-id="a2471-258">Do not routinely perform a no-tracking query and then attach the returned entities to the same context.</span></span> <span data-ttu-id="a2471-259">这会比使用跟踪查询更慢且更难。</span><span class="sxs-lookup"><span data-stu-id="a2471-259">This will be both slower and harder to get right than using a tracking query.</span></span>

<span data-ttu-id="a2471-260">无跟踪查询不会执行标识解析，因为这样做会影响从查询对大量实体进行流式处理的性能。</span><span class="sxs-lookup"><span data-stu-id="a2471-260">No-tracking queries do not perform identity resolution because doing so impacts the performance of streaming a large number of entities from a query.</span></span> <span data-ttu-id="a2471-261">这是因为，标识解析需要跟踪每个返回的实例，以便可以使用它，而不是以后创建重复项。</span><span class="sxs-lookup"><span data-stu-id="a2471-261">This is because identity resolution requires keeping track of each instance returned so that it can be used instead of later creating a duplicate.</span></span>

<span data-ttu-id="a2471-262">从 EF Core 5.0 开始，无跟踪查询可以通过使用强制执行标识解析 <xref:Microsoft.EntityFrameworkCore.EntityFrameworkQueryableExtensions.AsNoTrackingWithIdentityResolution%60%601(System.Linq.IQueryable{%60%600})> 。</span><span class="sxs-lookup"><span data-stu-id="a2471-262">Starting with EF Core 5.0, no-tracking queries can be forced to perform identity resolution by using <xref:Microsoft.EntityFrameworkCore.EntityFrameworkQueryableExtensions.AsNoTrackingWithIdentityResolution%60%601(System.Linq.IQueryable{%60%600})>.</span></span> <span data-ttu-id="a2471-263">然后，该查询将跟踪返回 (的实例，而无需按正常方式跟踪它们) 并确保在查询结果中不会创建重复项。</span><span class="sxs-lookup"><span data-stu-id="a2471-263">The query will then keep track of returned instances (without tracking them in the normal way) and ensure no duplicates are created in the query results.</span></span>

## <a name="overriding-object-equality"></a><span data-ttu-id="a2471-264">重写对象相等性</span><span class="sxs-lookup"><span data-stu-id="a2471-264">Overriding object equality</span></span>

<span data-ttu-id="a2471-265">EF Core 在比较实体实例时使用 [引用相等性](/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons) 。</span><span class="sxs-lookup"><span data-stu-id="a2471-265">EF Core uses [reference equality](/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons) when comparing entity instances.</span></span> <span data-ttu-id="a2471-266">即使实体类型重写 <xref:System.Object.Equals(System.Object)?displayProperty=nameWithType> 或更改对象相等性，也是如此。</span><span class="sxs-lookup"><span data-stu-id="a2471-266">This is the case even if the entity types override <xref:System.Object.Equals(System.Object)?displayProperty=nameWithType> or otherwise change object equality.</span></span> <span data-ttu-id="a2471-267">但是，重写相等性可能会影响 EF Core 行为：当集合导航使用重写的相等性而不是引用相等性时，因此将多个实例报告为同一。</span><span class="sxs-lookup"><span data-stu-id="a2471-267">However, there is one place where overriding equality can impact EF Core behavior: when collection navigations use the overridden equality instead of reference equality, and hence report multiple instances as the same.</span></span>

<span data-ttu-id="a2471-268">因此，建议应避免重写实体相等性。</span><span class="sxs-lookup"><span data-stu-id="a2471-268">Because of this it is recommended that overriding entity equality should be avoided.</span></span> <span data-ttu-id="a2471-269">如果使用此方法，请确保创建强制引用相等的集合导航。</span><span class="sxs-lookup"><span data-stu-id="a2471-269">If it is used, then make sure to create collection navigations that force reference equality.</span></span> <span data-ttu-id="a2471-270">例如，创建使用引用相等性的相等比较器：</span><span class="sxs-lookup"><span data-stu-id="a2471-270">For example, create an equality comparer that uses reference equality:</span></span>

<!--
    public sealed class ReferenceEqualityComparer : IEqualityComparer<object>
    {
        private ReferenceEqualityComparer()
        {
        }

        public static ReferenceEqualityComparer Instance { get; } = new ReferenceEqualityComparer();

        bool IEqualityComparer<object>.Equals(object x, object y) => x == y;

        int IEqualityComparer<object>.GetHashCode(object obj) => RuntimeHelpers.GetHashCode(obj);
    }
-->
[!code-csharp[ReferenceEqualityComparer](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/ReferenceEqualityComparer.cs?name=ReferenceEqualityComparer)]

<span data-ttu-id="a2471-271">从 .NET 5 开始 (，它将作为中包含在 BCL 中 <xref:System.Collections.Generic.ReferenceEqualityComparer> 。 ) </span><span class="sxs-lookup"><span data-stu-id="a2471-271">(Starting with .NET 5, this is included in the BCL as <xref:System.Collections.Generic.ReferenceEqualityComparer>.)</span></span>

<span data-ttu-id="a2471-272">然后，可以在创建集合导航时使用此比较器。</span><span class="sxs-lookup"><span data-stu-id="a2471-272">This comparer can then be used when creating collection navigations.</span></span> <span data-ttu-id="a2471-273">例如：</span><span class="sxs-lookup"><span data-stu-id="a2471-273">For example:</span></span>

<!--
        public ICollection<Order> Orders { get; set; }
            = new HashSet<Order>(ReferenceEqualityComparer.Instance);
-->
[!code-csharp[OrdersCollection](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=OrdersCollection)]

### <a name="comparing-key-properties"></a><span data-ttu-id="a2471-274">比较键属性</span><span class="sxs-lookup"><span data-stu-id="a2471-274">Comparing key properties</span></span>

<span data-ttu-id="a2471-275">除了相等比较以外，还需要对键值进行排序。</span><span class="sxs-lookup"><span data-stu-id="a2471-275">In addition to equality comparisons, key values also need to be ordered.</span></span> <span data-ttu-id="a2471-276">这对于在对 SaveChanges 的单个调用中更新多个实体时避免死锁非常重要。</span><span class="sxs-lookup"><span data-stu-id="a2471-276">This is important for avoiding deadlocks when updating multiple entities in a single call to SaveChanges.</span></span> <span data-ttu-id="a2471-277">用于 primary、代用或外键属性以及用于唯一索引的所有类型都必须实现 <xref:System.IComparable%601> 和 <xref:System.IEquatable%601> 。</span><span class="sxs-lookup"><span data-stu-id="a2471-277">All types used for primary, alternate, or foreign key properties, as well as those used for unique indexes, must implement <xref:System.IComparable%601> and <xref:System.IEquatable%601>.</span></span> <span data-ttu-id="a2471-278">通常用作键 (int、Guid、string 等的类型 ) 已支持这些接口。</span><span class="sxs-lookup"><span data-stu-id="a2471-278">Types normally used as keys (int, Guid, string, etc.) already support these interfaces.</span></span> <span data-ttu-id="a2471-279">自定义密钥类型可以添加这些接口。</span><span class="sxs-lookup"><span data-stu-id="a2471-279">Custom key types may to add these interfaces.</span></span>
