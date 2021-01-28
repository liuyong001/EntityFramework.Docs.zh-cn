---
title: 更改外键和导航-EF Core
description: 如何通过操作外键和导航来更改实体之间的关系
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/relationship-changes
ms.openlocfilehash: b1ebe77ed29291beeef3708b603db026c38bbbec
ms.sourcegitcommit: 7700840119b1639275f3b64836e7abb59103f2e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2021
ms.locfileid: "98983607"
---
# <a name="changing-foreign-keys-and-navigations"></a><span data-ttu-id="80ee4-103">更改外键和导航</span><span class="sxs-lookup"><span data-stu-id="80ee4-103">Changing Foreign Keys and Navigations</span></span>

## <a name="overview-of-foreign-keys-and-navigations"></a><span data-ttu-id="80ee4-104">外键和导航概述</span><span class="sxs-lookup"><span data-stu-id="80ee4-104">Overview of foreign keys and navigations</span></span>

<span data-ttu-id="80ee4-105">EF Core) 模型的 Entity Framework Core (中的关系使用 (Fk) 的外键来表示。</span><span class="sxs-lookup"><span data-stu-id="80ee4-105">Relationships in an Entity Framework Core (EF Core) model are represented using foreign keys (FKs).</span></span> <span data-ttu-id="80ee4-106">FK 由关系中依赖实体或子实体上的一个或多个属性组成。</span><span class="sxs-lookup"><span data-stu-id="80ee4-106">An FK consists of one or more properties on the dependent or child entity in the relationship.</span></span> <span data-ttu-id="80ee4-107">当 dependent/child 上的外键属性的值与其他或主键的值相匹配时，此从属/子实体将与给定的主体/父实体相关联 (PK) 主体/父项的属性。</span><span class="sxs-lookup"><span data-stu-id="80ee4-107">This dependent/child entity is associated with a given principal/parent entity when the values of the foreign key properties on the dependent/child match the values of the alternate or primary key (PK) properties on the principal/parent.</span></span>

<span data-ttu-id="80ee4-108">外键是一种在数据库中存储和处理关系的好方法，但在应用程序代码中处理多个相关实体时，它们并不是很好的方法。</span><span class="sxs-lookup"><span data-stu-id="80ee4-108">Foreign keys are a good way to store and manipulate relationships in the database, but are not very friendly when working with multiple related entities in application code.</span></span> <span data-ttu-id="80ee4-109">因此，大多数 EF Core 模型还会将 "导航" 置于 FK 表示形式之上。</span><span class="sxs-lookup"><span data-stu-id="80ee4-109">Therefore, most EF Core models also layer "navigations" over the FK representation.</span></span> <span data-ttu-id="80ee4-110">导航窗体 c #/.NET 实体实例间的引用，这些实体实例反映通过将外键值与主键值或备用键值匹配而找到的关联。</span><span class="sxs-lookup"><span data-stu-id="80ee4-110">Navigations form C#/.NET references between entity instances that reflect the associations found by matching foreign key values to primary or alternate key values.</span></span>

<span data-ttu-id="80ee4-111">导航既可以在关系的双方上使用，也可以在一侧上使用，也可以根本不会使用，只留下 FK 属性。</span><span class="sxs-lookup"><span data-stu-id="80ee4-111">Navigations can be used on both sides of the relationship, on one side only, or not at all, leaving only the FK property.</span></span> <span data-ttu-id="80ee4-112">可以通过将 FK 属性设为 [阴影属性](xref:core/modeling/shadow-properties)来隐藏该属性。</span><span class="sxs-lookup"><span data-stu-id="80ee4-112">The FK property can be hidden by making it a [shadow property](xref:core/modeling/shadow-properties).</span></span> <span data-ttu-id="80ee4-113">有关建模关系的详细信息，请参阅 [关系](xref:core/modeling/relationships) 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-113">See [Relationships](xref:core/modeling/relationships) for more information on modelling relationships.</span></span>

> [!TIP]
> <span data-ttu-id="80ee4-114">本文档假设了解 EF Core 更改跟踪的实体状态和基础知识。</span><span class="sxs-lookup"><span data-stu-id="80ee4-114">This document assumes that entity states and the basics of EF Core change tracking are understood.</span></span> <span data-ttu-id="80ee4-115">有关这些主题的详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-115">See [Change Tracking in EF Core](xref:core/change-tracking/index) for more information on these topics.</span></span>

> [!TIP]
> <span data-ttu-id="80ee4-116">通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangingFKsAndNavigations)，你可运行并调试到本文档中的所有代码。</span><span class="sxs-lookup"><span data-stu-id="80ee4-116">You can run and debug into all the code in this document by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangingFKsAndNavigations).</span></span>

### <a name="example-model"></a><span data-ttu-id="80ee4-117">示例模型</span><span class="sxs-lookup"><span data-stu-id="80ee4-117">Example model</span></span>

<span data-ttu-id="80ee4-118">下面的模型包含四个实体类型，它们之间有关系。</span><span class="sxs-lookup"><span data-stu-id="80ee4-118">The following model contains four entity types with relationships between them.</span></span> <span data-ttu-id="80ee4-119">代码中的注释指示哪些属性为外键、主键和导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-119">The comments in the code indicate which properties are foreign keys, primary keys, and navigations.</span></span>

<!--
public class Blog
{
    public int Id { get; set; } // Primary key
    public string Name { get; set; }

    public IList<Post> Posts { get; } = new List<Post>(); // Collection navigation
    public BlogAssets Assets { get; set; } // Reference navigation
}

public class BlogAssets
{
    public int Id { get; set; } // Primary key
    public byte[] Banner { get; set; }

    public int BlogId { get; set; } // Foreign key
    public Blog Blog { get; set; } // Reference navigation
}

public class Post
{
    public int Id { get; set; } // Primary key
    public string Title { get; set; }
    public string Content { get; set; }

    public int? BlogId { get; set; } // Foreign key
    public Blog Blog { get; set; } // Reference navigation

    public IList<Tag> Tags { get; } = new List<Tag>(); // Skip collection navigation
}

public class Tag
{
    public int Id { get; set; } // Primary key
    public string Text { get; set; }

    public IList<Post> Posts { get; } = new List<Post>(); // Skip collection navigation
}
-->
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Model)]

<span data-ttu-id="80ee4-120">此模型中的三个关系为：</span><span class="sxs-lookup"><span data-stu-id="80ee4-120">The three relationships in this model are:</span></span>

- <span data-ttu-id="80ee4-121">每个博客可以有多个 (一对多) ：</span><span class="sxs-lookup"><span data-stu-id="80ee4-121">Each blog can have many posts (one-to-many):</span></span>
  - <span data-ttu-id="80ee4-122">`Blog` 是主体/父级。</span><span class="sxs-lookup"><span data-stu-id="80ee4-122">`Blog` is the principal/parent.</span></span>
  - <span data-ttu-id="80ee4-123">`Post` 依赖项/子级。</span><span class="sxs-lookup"><span data-stu-id="80ee4-123">`Post` is the dependent/child.</span></span> <span data-ttu-id="80ee4-124">它包含 FK 属性 `Post.BlogId` ，其值必须与 `Blog.Id` 相关博客的 PK 值匹配。</span><span class="sxs-lookup"><span data-stu-id="80ee4-124">It contains the FK property `Post.BlogId`, the value of which must match the `Blog.Id` PK value of the related blog.</span></span>
  - <span data-ttu-id="80ee4-125">`Post.Blog` 是从发布到相关博客的引用导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-125">`Post.Blog` is a reference navigation from a post to the associated blog.</span></span> <span data-ttu-id="80ee4-126">`Post.Blog` 是的反向导航 `Blog.Posts` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-126">`Post.Blog` is the inverse navigation for `Blog.Posts`.</span></span>
  - <span data-ttu-id="80ee4-127">`Blog.Posts` 是从博客到所有关联帖子的集合导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-127">`Blog.Posts` is a collection navigation from a blog to all the associated posts.</span></span> <span data-ttu-id="80ee4-128">`Blog.Posts` 是的反向导航 `Post.Blog` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-128">`Blog.Posts` is the inverse navigation for `Post.Blog`.</span></span>
- <span data-ttu-id="80ee4-129">每个博客可以有一个资产 (一对一) ：</span><span class="sxs-lookup"><span data-stu-id="80ee4-129">Each blog can have one assets (one-to-one):</span></span>
  - <span data-ttu-id="80ee4-130">`Blog` 是主体/父级。</span><span class="sxs-lookup"><span data-stu-id="80ee4-130">`Blog` is the principal/parent.</span></span>
  - <span data-ttu-id="80ee4-131">`BlogAssets` 依赖项/子级。</span><span class="sxs-lookup"><span data-stu-id="80ee4-131">`BlogAssets` is the dependent/child.</span></span> <span data-ttu-id="80ee4-132">它包含 FK 属性 `BlogAssets.BlogId` ，其值必须与 `Blog.Id` 相关博客的 PK 值匹配。</span><span class="sxs-lookup"><span data-stu-id="80ee4-132">It contains the FK property `BlogAssets.BlogId`, the value of which must match the `Blog.Id` PK value of the related blog.</span></span>
  - <span data-ttu-id="80ee4-133">`BlogAssets.Blog` 是从资源到关联博客的引用导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-133">`BlogAssets.Blog` is a reference navigation from the assets to the associated blog.</span></span> <span data-ttu-id="80ee4-134">`BlogAssets.Blog` 是的反向导航 `Blog.Assets` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-134">`BlogAssets.Blog` is the inverse navigation for `Blog.Assets`.</span></span>
  - <span data-ttu-id="80ee4-135">`Blog.Assets` 是从博客到相关资产的引用导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-135">`Blog.Assets` is a reference navigation from the blog to the associated assets.</span></span> <span data-ttu-id="80ee4-136">`Blog.Assets` 是的反向导航 `BlogAssets.Blog` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-136">`Blog.Assets` is the inverse navigation for `BlogAssets.Blog`.</span></span>
- <span data-ttu-id="80ee4-137">每篇文章都可以有多个标记，每个标记可有多个 post (多对多) ：</span><span class="sxs-lookup"><span data-stu-id="80ee4-137">Each post can have many tags and each tag can have many posts (many-to-many):</span></span>
  - <span data-ttu-id="80ee4-138">多对多关系是比 2 1 到多关系的更多层关系。</span><span class="sxs-lookup"><span data-stu-id="80ee4-138">Many-to-many relationships are a further layer over two one-to-many relationships.</span></span> <span data-ttu-id="80ee4-139">多对多关系将在本文档的后面部分介绍。</span><span class="sxs-lookup"><span data-stu-id="80ee4-139">Many-to-many relationships are covered later in this document.</span></span>
  - <span data-ttu-id="80ee4-140">`Post.Tags` 是从发布到所有关联标记的集合导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-140">`Post.Tags` is a collection navigation from a post to all the associated tags.</span></span> <span data-ttu-id="80ee4-141">`Post.Tags` 是的反向导航 `Tag.Posts` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-141">`Post.Tags` is the inverse navigation for `Tag.Posts`.</span></span>
  - <span data-ttu-id="80ee4-142">`Tag.Posts` 是从标记到所有关联的发布的集合导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-142">`Tag.Posts` is a collection navigation from a tag to all the associated posts.</span></span> <span data-ttu-id="80ee4-143">`Tag.Posts` 是的反向导航 `Post.Tags` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-143">`Tag.Posts` is the inverse navigation for `Post.Tags`.</span></span>

<span data-ttu-id="80ee4-144">有关如何建模和配置关系的详细信息，请参阅 [关系](xref:core/modeling/relationships) 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-144">See [Relationships](xref:core/modeling/relationships) for more information on how to model and configure relationships.</span></span>

## <a name="relationship-fixup"></a><span data-ttu-id="80ee4-145">关系修正</span><span class="sxs-lookup"><span data-stu-id="80ee4-145">Relationship fixup</span></span>

<span data-ttu-id="80ee4-146">EF Core 使导航与外键值保持一致，反之亦然。</span><span class="sxs-lookup"><span data-stu-id="80ee4-146">EF Core keeps navigations in alignment with foreign key values and vice versa.</span></span> <span data-ttu-id="80ee4-147">也就是说，如果外键值发生更改，使其现在引用其他主体/父实体，则会更新导航以反映此更改。</span><span class="sxs-lookup"><span data-stu-id="80ee4-147">That is, if a foreign key value changes such that it now refers to a different principal/parent entity, then the navigations are updated to reflect this change.</span></span> <span data-ttu-id="80ee4-148">同样，如果更改了导航，则所涉及的实体的外键值将更新以反映此更改。</span><span class="sxs-lookup"><span data-stu-id="80ee4-148">Likewise, if a navigation is changed, then the foreign key values of the entities involved are updated to reflect this change.</span></span> <span data-ttu-id="80ee4-149">这称为 "关系修正"。</span><span class="sxs-lookup"><span data-stu-id="80ee4-149">This is called "relationship fixup".</span></span>

### <a name="fixup-by-query"></a><span data-ttu-id="80ee4-150">按查询修正</span><span class="sxs-lookup"><span data-stu-id="80ee4-150">Fixup by query</span></span>

<span data-ttu-id="80ee4-151">如果从数据库中查询实体，则会首先出现修正。</span><span class="sxs-lookup"><span data-stu-id="80ee4-151">Fixup first occurs when entities are queried from the database.</span></span> <span data-ttu-id="80ee4-152">数据库仅具有外键值，因此当 EF Core 从数据库创建实体实例时，它将使用外键值来设置引用导航，并根据需要将实体添加到集合导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-152">The database has only foreign key values, so when EF Core creates an entity instance from the database it uses the foreign key values to set reference navigations and add entities to collection navigations as appropriate.</span></span> <span data-ttu-id="80ee4-153">例如，请考虑对博客及其相关的文章和资产进行查询：</span><span class="sxs-lookup"><span data-stu-id="80ee4-153">For example, consider a query for blogs and its associated posts and assets:</span></span>

<!--
        using var context = new BlogsContext();

        var blogs = context.Blogs
            .Include(e => e.Posts)
            .Include(e => e.Assets)
            .ToList();

        Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Relationship_fixup_1](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Relationship_fixup_1)]

<span data-ttu-id="80ee4-154">对于每个博客，EF Core 将首先创建一个 `Blog` 实例。</span><span class="sxs-lookup"><span data-stu-id="80ee4-154">For each blog, EF Core will first create a `Blog` instance.</span></span> <span data-ttu-id="80ee4-155">然后，在从数据库加载每个发布时，其 `Post.Blog` 引用导航设置为指向关联的博客。</span><span class="sxs-lookup"><span data-stu-id="80ee4-155">Then, as each post is loaded from the database its `Post.Blog` reference navigation is set to point to the associated blog.</span></span> <span data-ttu-id="80ee4-156">同样，将 post 添加到 `Blog.Posts` 集合导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-156">Likewise, the post is added to the `Blog.Posts` collection navigation.</span></span> <span data-ttu-id="80ee4-157">与相同 `BlogAssets` ，但在这种情况下，这两个导航都是引用。</span><span class="sxs-lookup"><span data-stu-id="80ee4-157">The same thing happens with `BlogAssets`, except in this case both navigations are references.</span></span> <span data-ttu-id="80ee4-158">`Blog.Assets`导航设置为指向资产实例， `BlogAsserts.Blog` 导航设置为指向博客实例。</span><span class="sxs-lookup"><span data-stu-id="80ee4-158">The `Blog.Assets` navigation is set to point to the assets instance, and the `BlogAsserts.Blog` navigation is set to point to the blog instance.</span></span>

<span data-ttu-id="80ee4-159">在此查询显示两个博客后查看 " [更改跟踪器" 调试视图](xref:core/change-tracking/debug-views) ，每个博客包含一个资产和两篇文章：</span><span class="sxs-lookup"><span data-stu-id="80ee4-159">Looking at the [change tracker debug view](xref:core/change-tracking/debug-views) after this query shows two blogs, each with one assets and two posts being tracked:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Assets: {Id: 1}
  Posts: [{Id: 1}, {Id: 2}]
Blog {Id: 2} Unchanged
  Id: 2 PK
  Name: 'Visual Studio Blog'
  Assets: {Id: 2}
  Posts: [{Id: 3}, {Id: 4}]
BlogAssets {Id: 1} Unchanged
  Id: 1 PK
  Banner: <null>
  BlogId: 1 FK
  Blog: {Id: 1}
BlogAssets {Id: 2} Unchanged
  Id: 2 PK
  Banner: <null>
  BlogId: 2 FK
  Blog: {Id: 2}
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
  Tags: []
Post {Id: 2} Unchanged
  Id: 2 PK
  BlogId: 1 FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: {Id: 1}
  Tags: []
Post {Id: 3} Unchanged
  Id: 3 PK
  BlogId: 2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: {Id: 2}
  Tags: []
Post {Id: 4} Unchanged
  Id: 4 PK
  BlogId: 2 FK
  Content: 'Examine when database queries were executed and measure how ...'
  Title: 'Database Profiling with Visual Studio'
  Blog: {Id: 2}
  Tags: []
```

<span data-ttu-id="80ee4-160">"调试" 视图显示键值和导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-160">The debug view shows both key values and navigations.</span></span> <span data-ttu-id="80ee4-161">使用相关实体的主键值显示导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-161">Navigations are shown using the primary key values of the related entities.</span></span> <span data-ttu-id="80ee4-162">例如， `Posts: [{Id: 1}, {Id: 2}]` 在上面的输出中，表示 `Blog.Posts` 集合导航分别包含两个主键分别为1和2的相关文章。</span><span class="sxs-lookup"><span data-stu-id="80ee4-162">For example, `Posts: [{Id: 1}, {Id: 2}]` in the output above indicates that the `Blog.Posts` collection navigation contains two related posts with primary keys 1 and 2 respectively.</span></span> <span data-ttu-id="80ee4-163">同样，对于与第一个博客关联的每个帖子， `Blog: {Id: 1}` 该行指示 `Post.Blog` 导航引用了主键为1的博客。</span><span class="sxs-lookup"><span data-stu-id="80ee4-163">Similarly, for each post associated with the first blog, the `Blog: {Id: 1}` line indicates that the `Post.Blog` navigation references the Blog with primary key 1.</span></span>

### <a name="fixup-to-locally-tracked-entities"></a><span data-ttu-id="80ee4-164">本地跟踪实体的链接</span><span class="sxs-lookup"><span data-stu-id="80ee4-164">Fixup to locally tracked entities</span></span>

<span data-ttu-id="80ee4-165">关系修正还会在从跟踪查询返回的实体与 DbContext 已跟踪的实体之间发生。</span><span class="sxs-lookup"><span data-stu-id="80ee4-165">Relationship fixup also happens between entities returned from a tracking query and entities already tracked by the DbContext.</span></span> <span data-ttu-id="80ee4-166">例如，考虑对博客、文章和资产执行三个单独的查询：</span><span class="sxs-lookup"><span data-stu-id="80ee4-166">For example, consider executing three separate queries for blogs, posts, and assets:</span></span>

<!--
        using var context = new BlogsContext();

        var blogs = context.Blogs.ToList();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);

        var assets = context.Assets.ToList();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);

        var posts = context.Posts.ToList();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Relationship_fixup_2](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Relationship_fixup_2)]
<span data-ttu-id="80ee4-167">再次查看调试视图，在第一次查询后，只跟踪这两个博客：</span><span class="sxs-lookup"><span data-stu-id="80ee4-167">Looking again at the debug views, after the first query only the two blogs are tracked:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Assets: <null>
  Posts: []
Blog {Id: 2} Unchanged
  Id: 2 PK
  Name: 'Visual Studio Blog'
  Assets: <null>
  Posts: []
```

<span data-ttu-id="80ee4-168">`Blog.Assets`引用导航为 null，并且 `Blog.Posts` 集合导航为空，因为当前上下文未跟踪关联的实体。</span><span class="sxs-lookup"><span data-stu-id="80ee4-168">The `Blog.Assets` reference navigations are null, and the `Blog.Posts` collection navigations are empty because no associated entities are currently being tracked by the context.</span></span>

<span data-ttu-id="80ee4-169">在第二次查询后， `Blogs.Assets` 引用导航已固定到新跟踪的 `BlogAsset` 实例。</span><span class="sxs-lookup"><span data-stu-id="80ee4-169">After the second query, the `Blogs.Assets` reference navigations have been fixed up to point to the newly tracked `BlogAsset` instances.</span></span> <span data-ttu-id="80ee4-170">同样， `BlogAssets.Blog` 引用导航设置为指向适当的已跟踪 `Blog` 实例。</span><span class="sxs-lookup"><span data-stu-id="80ee4-170">Likewise, the `BlogAssets.Blog` reference navigations are set to point to the appropriate already tracked `Blog` instance.</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Assets: {Id: 1}
  Posts: []
Blog {Id: 2} Unchanged
  Id: 2 PK
  Name: 'Visual Studio Blog'
  Assets: {Id: 2}
  Posts: []
BlogAssets {Id: 1} Unchanged
  Id: 1 PK
  Banner: <null>
  BlogId: 1 FK
  Blog: {Id: 1}
BlogAssets {Id: 2} Unchanged
  Id: 2 PK
  Banner: <null>
  BlogId: 2 FK
  Blog: {Id: 2}
```

<span data-ttu-id="80ee4-171">最后，在第三次查询后， `Blog.Posts` 集合导航现在包含所有相关的帖子， `Post.Blog` 引用指向相应的 `Blog` 实例：</span><span class="sxs-lookup"><span data-stu-id="80ee4-171">Finally, after the third query, the `Blog.Posts` collection navigations now contain all related posts, and the `Post.Blog` references point to the appropriate `Blog` instance:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Assets: {Id: 1}
  Posts: [{Id: 1}, {Id: 2}]
Blog {Id: 2} Unchanged
  Id: 2 PK
  Name: 'Visual Studio Blog'
  Assets: {Id: 2}
  Posts: [{Id: 3}, {Id: 4}]
BlogAssets {Id: 1} Unchanged
  Id: 1 PK
  Banner: <null>
  BlogId: 1 FK
  Blog: {Id: 1}
BlogAssets {Id: 2} Unchanged
  Id: 2 PK
  Banner: <null>
  BlogId: 2 FK
  Blog: {Id: 2}
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
  Tags: []
Post {Id: 2} Unchanged
  Id: 2 PK
  BlogId: 1 FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: {Id: 1}
  Tags: []
Post {Id: 3} Unchanged
  Id: 3 PK
  BlogId: 2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: {Id: 2}
  Tags: []
Post {Id: 4} Unchanged
  Id: 4 PK
  BlogId: 2 FK
  Content: 'Examine when database queries were executed and measure how ...'
  Title: 'Database Profiling with Visual Studio'
  Blog: {Id: 2}
  Tags: []
```

<span data-ttu-id="80ee4-172">这与通过原始单个查询实现的结束状态相同，因为在跟踪实体时 EF Core 固定的导航，即使是在来自多个不同的查询时也是如此。</span><span class="sxs-lookup"><span data-stu-id="80ee4-172">This is the same end-state as was achieved with the original single query, since EF Core fixed up navigations as entities were tracked, even when coming from multiple different queries.</span></span>

> [!NOTE]
> <span data-ttu-id="80ee4-173">修正永不导致从数据库返回更多数据。</span><span class="sxs-lookup"><span data-stu-id="80ee4-173">Fixup never causes more data to be returned from the database.</span></span> <span data-ttu-id="80ee4-174">它仅连接已由查询返回或已由 DbContext 跟踪的实体。</span><span class="sxs-lookup"><span data-stu-id="80ee4-174">It only connects entities that are already returned by the query or already tracked by the DbContext.</span></span> <span data-ttu-id="80ee4-175">有关在序列化实体时处理重复项的信息，请参阅 [中的标识解析 EF Core](xref:core/change-tracking/identity-resolution) 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-175">See [Identity Resolution in EF Core](xref:core/change-tracking/identity-resolution) for information about handling duplicates when serializing entities.</span></span>

## <a name="changing-relationships-using-navigations"></a><span data-ttu-id="80ee4-176">使用导航更改关系</span><span class="sxs-lookup"><span data-stu-id="80ee4-176">Changing relationships using navigations</span></span>

<span data-ttu-id="80ee4-177">更改两个实体之间的关系的最简单方法是操作导航，同时保留 EF Core 适当地修正反向导航和 FK 值。</span><span class="sxs-lookup"><span data-stu-id="80ee4-177">The easiest way to change the relationship between two entities is by manipulating a navigation, while leaving EF Core to fixup the inverse navigation and FK values appropriately.</span></span> <span data-ttu-id="80ee4-178">可通过以下方法完成此操作：</span><span class="sxs-lookup"><span data-stu-id="80ee4-178">This can be done by:</span></span>

- <span data-ttu-id="80ee4-179">添加或删除集合导航中的实体。</span><span class="sxs-lookup"><span data-stu-id="80ee4-179">Adding or removing an entity from a collection navigation.</span></span>
- <span data-ttu-id="80ee4-180">更改引用导航以指向不同的实体，或将其设置为 null。</span><span class="sxs-lookup"><span data-stu-id="80ee4-180">Changing a reference navigation to point to a different entity, or setting it to null.</span></span>

### <a name="adding-or-removing-from-collection-navigations"></a><span data-ttu-id="80ee4-181">添加或删除集合导航</span><span class="sxs-lookup"><span data-stu-id="80ee4-181">Adding or removing from collection navigations</span></span>

<span data-ttu-id="80ee4-182">例如，让我们将其中一篇文章从 Visual Studio 博客移到 .NET 博客。</span><span class="sxs-lookup"><span data-stu-id="80ee4-182">For example, let's move one of the posts from the Visual Studio blog to the .NET blog.</span></span> <span data-ttu-id="80ee4-183">这需要首先加载博客和文章，然后将张贴内容从一个博客上的导航集合移到其他博客上的导航集合：</span><span class="sxs-lookup"><span data-stu-id="80ee4-183">This requires first loading the blogs and posts, and then moving the post from the navigation collection on one blog to the navigation collection on the other blog:</span></span>

<!--
        using var context = new BlogsContext();

        var dotNetBlog = context.Blogs.Include(e => e.Posts).Single(e => e.Name == ".NET Blog");
        var vsBlog = context.Blogs.Include(e => e.Posts).Single(e => e.Name == "Visual Studio Blog");

        Console.WriteLine(context.ChangeTracker.DebugView.LongView);

        var post = vsBlog.Posts.Single(e => e.Title.StartsWith("Disassembly improvements"));
        vsBlog.Posts.Remove(post);
        dotNetBlog.Posts.Add(post);

        context.ChangeTracker.DetectChanges();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);

        context.SaveChanges();
-->
[!code-csharp[Changing_relationships_using_navigations_1](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Changing_relationships_using_navigations_1)]

> [!TIP]
> <span data-ttu-id="80ee4-184">此处需要对的调用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> ，因为访问 "调试" 视图不会导致 [更改自动检测](xref:core/change-tracking/change-detection)。</span><span class="sxs-lookup"><span data-stu-id="80ee4-184">A call to <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> is needed here because accessing the debug view does not cause [automatic detection of changes](xref:core/change-tracking/change-detection).</span></span>

<span data-ttu-id="80ee4-185">这是运行上述代码后打印的 "调试" 视图：</span><span class="sxs-lookup"><span data-stu-id="80ee4-185">This is the debug view printed after running the code above:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Assets: <null>
  Posts: [{Id: 1}, {Id: 2}, {Id: 3}]
Blog {Id: 2} Unchanged
  Id: 2 PK
  Name: 'Visual Studio Blog'
  Assets: <null>
  Posts: [{Id: 4}]
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
  Tags: []
Post {Id: 2} Unchanged
  Id: 2 PK
  BlogId: 1 FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: {Id: 1}
  Tags: []
Post {Id: 3} Modified
  Id: 3 PK
  BlogId: 1 FK Modified Originally 2
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: {Id: 1}
  Tags: []
Post {Id: 4} Unchanged
  Id: 4 PK
  BlogId: 2 FK
  Content: 'Examine when database queries were executed and measure how ...'
  Title: 'Database Profiling with Visual Studio'
  Blog: {Id: 2}
  Tags: []
```

<span data-ttu-id="80ee4-186">`Blog.Posts`.Net 博客上的导航现在包含三个发布 (`Posts: [{Id: 1}, {Id: 2}, {Id: 3}]`) 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-186">The `Blog.Posts` navigation on the .NET Blog now has three posts (`Posts: [{Id: 1}, {Id: 2}, {Id: 3}]`).</span></span> <span data-ttu-id="80ee4-187">同样， `Blog.Posts` Visual Studio 博客上的导航仅有一篇文章 (`Posts: [{Id: 4}]`) 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-187">Likewise, the `Blog.Posts` navigation on the Visual Studio blog only has one post (`Posts: [{Id: 4}]`).</span></span> <span data-ttu-id="80ee4-188">这是预期的，因为代码显式更改了这些集合。</span><span class="sxs-lookup"><span data-stu-id="80ee4-188">This is to be expected since the code explicitly changed these collections.</span></span>

<span data-ttu-id="80ee4-189">更有趣的是，尽管代码未显式更改 `Post.Blog` 导航，但它已得到修复，可指向 Visual Studio 博客 (`Blog: {Id: 1}`) 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-189">More interestingly, even though the code did not explicitly change the `Post.Blog` navigation, it has been fixed-up to point to the Visual Studio blog (`Blog: {Id: 1}`).</span></span> <span data-ttu-id="80ee4-190">此外， `Post.BlogId` 外键值已更新，以匹配 .net 博客的主键值。</span><span class="sxs-lookup"><span data-stu-id="80ee4-190">Also, the `Post.BlogId` foreign key value has been updated to match the primary key value of the .NET blog.</span></span> <span data-ttu-id="80ee4-191">在调用 SaveChanges 时，此更改将保留到数据库中的 FK 值：</span><span class="sxs-lookup"><span data-stu-id="80ee4-191">This change to the FK value in then persisted to the database when SaveChanges is called:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p1='3' (DbType = String), @p0='1' (Nullable = true) (DbType = String)], CommandType='Text', CommandTimeout='30']
UPDATE "Posts" SET "BlogId" = @p0
WHERE "Id" = @p1;
SELECT changes();
```

### <a name="changing-reference-navigations"></a><span data-ttu-id="80ee4-192">更改引用导航</span><span class="sxs-lookup"><span data-stu-id="80ee4-192">Changing reference navigations</span></span>

<span data-ttu-id="80ee4-193">在上面的示例中，通过操作每个博客上的文章的集合导航，将文章从一个博客移到另一个博客。</span><span class="sxs-lookup"><span data-stu-id="80ee4-193">In the previous example, a post was moved from one blog to another by manipulating the collection navigation of posts on each blog.</span></span> <span data-ttu-id="80ee4-194">可以通过更改 `Post.Blog` 引用导航来指向新博客来实现相同的目的。</span><span class="sxs-lookup"><span data-stu-id="80ee4-194">The same thing can be achieved by instead changing the `Post.Blog` reference navigation to point to the new blog.</span></span> <span data-ttu-id="80ee4-195">例如：</span><span class="sxs-lookup"><span data-stu-id="80ee4-195">For example:</span></span>

<!--
        var post = vsBlog.Posts.Single(e => e.Title.StartsWith("Disassembly improvements"));
        post.Blog = dotNetBlog;
-->
[!code-csharp[Changing_relationships_using_navigations_2](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Changing_relationships_using_navigations_2)]

<span data-ttu-id="80ee4-196">此更改之后的调试视图与上一示例中的调试视图 _完全相同_ 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-196">The debug view after this change is _exactly the same_ as it was in the previous example.</span></span> <span data-ttu-id="80ee4-197">这是因为 EF Core 检测到引用导航更改，然后将集合导航和 FK 值固定到匹配项。</span><span class="sxs-lookup"><span data-stu-id="80ee4-197">This because EF Core detected the reference navigation change and then fixed up the collection navigations and FK value to match.</span></span>

## <a name="changing-relationships-using-foreign-key-values"></a><span data-ttu-id="80ee4-198">使用外键值更改关系</span><span class="sxs-lookup"><span data-stu-id="80ee4-198">Changing relationships using foreign key values</span></span>

<span data-ttu-id="80ee4-199">在上一节中，关系由可自动更新的外键值的导航操作。</span><span class="sxs-lookup"><span data-stu-id="80ee4-199">In the previous section, relationships were manipulated by navigations leaving foreign key values to be updated automatically.</span></span> <span data-ttu-id="80ee4-200">这是在 EF Core 中操作关系的建议方法。</span><span class="sxs-lookup"><span data-stu-id="80ee4-200">This is the recommended way to manipulate relationships in EF Core.</span></span> <span data-ttu-id="80ee4-201">但是，也可以直接处理 FK 值。</span><span class="sxs-lookup"><span data-stu-id="80ee4-201">However, it is also possible to manipulate FK values directly.</span></span> <span data-ttu-id="80ee4-202">例如，可以通过更改外键值将 post 从一个博客移到另一个博客 `Post.BlogId` ：</span><span class="sxs-lookup"><span data-stu-id="80ee4-202">For example, we can move a post from one blog to another by changing the `Post.BlogId` foreign key value:</span></span>

<!--
        var post = vsBlog.Posts.Single(e => e.Title.StartsWith("Disassembly improvements"));
        post.BlogId = dotNetBlog.Id;
-->
[!code-csharp[Changing_relationships_using_foreign_key_values_1](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Changing_relationships_using_foreign_key_values_1)]

<span data-ttu-id="80ee4-203">请注意，这与更改引用导航的方式非常类似，如前面的示例所示。</span><span class="sxs-lookup"><span data-stu-id="80ee4-203">Notice how this is very similar to changing the reference navigation, as shown in the previous example.</span></span>

<span data-ttu-id="80ee4-204">此更改之后的 "调试" 视图与前面两个示例的大小写 _完全相同_ 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-204">The debug view after this change is again _exactly the same_ as was the case for the previous two examples.</span></span> <span data-ttu-id="80ee4-205">这是因为 EF Core 检测到 FK 值发生变化，并同时修复了要匹配的引用导航和集合导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-205">This because EF Core detected the FK value change and then fixed up both the reference and collection navigations to match.</span></span>

> [!TIP]
> <span data-ttu-id="80ee4-206">请勿在每次关系发生更改时编写代码来操作所有导航和 FK 值。</span><span class="sxs-lookup"><span data-stu-id="80ee4-206">Do not write code to manipulate all navigations and FK values each time a relationship changes.</span></span> <span data-ttu-id="80ee4-207">此类代码更复杂，必须确保在每种情况下对外键和导航进行一致的更改。</span><span class="sxs-lookup"><span data-stu-id="80ee4-207">Such code is more complicated and must ensure consistent changes to foreign keys and navigations in every case.</span></span> <span data-ttu-id="80ee4-208">如果可能，只需处理一个导航，或者同时处理两个导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-208">If possible, just manipulate a single navigation, or maybe both navigations.</span></span> <span data-ttu-id="80ee4-209">如果需要，只需操作 FK 值。</span><span class="sxs-lookup"><span data-stu-id="80ee4-209">If needed, just manipulate FK values.</span></span> <span data-ttu-id="80ee4-210">避免处理导航和 FK 值。</span><span class="sxs-lookup"><span data-stu-id="80ee4-210">Avoid manipulating both navigations and FK values.</span></span>

## <a name="fixup-for-added-or-deleted-entities"></a><span data-ttu-id="80ee4-211">添加或删除实体的修正</span><span class="sxs-lookup"><span data-stu-id="80ee4-211">Fixup for added or deleted entities</span></span>

### <a name="adding-to-a-collection-navigation"></a><span data-ttu-id="80ee4-212">添加到集合导航</span><span class="sxs-lookup"><span data-stu-id="80ee4-212">Adding to a collection navigation</span></span>

<span data-ttu-id="80ee4-213">EF Core 在 [检测](xref:core/change-tracking/change-detection) 到已向集合导航中添加了一个新的依赖/子实体时执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="80ee4-213">EF Core performs the following actions when it [detects](xref:core/change-tracking/change-detection) that a new dependent/child entity has been added to a collection navigation:</span></span>

- <span data-ttu-id="80ee4-214">如果未跟踪实体，则跟踪该实体。</span><span class="sxs-lookup"><span data-stu-id="80ee4-214">If the entity is not tracked, then it is tracked.</span></span> <span data-ttu-id="80ee4-215">实体 (通常将处于 `Added` 状态。</span><span class="sxs-lookup"><span data-stu-id="80ee4-215">(The entity will usually be in the `Added` state.</span></span> <span data-ttu-id="80ee4-216">但是，如果将实体类型配置为使用生成的密钥并设置了主键值，则会在状态中跟踪该实体 `Unchanged` 。 ) </span><span class="sxs-lookup"><span data-stu-id="80ee4-216">However, if the entity type is configured to use generated keys and the primary key value is set, then the entity is tracked in the `Unchanged` state.)</span></span>
- <span data-ttu-id="80ee4-217">如果该实体与其他主体/父对象相关联，则该关系将被断开。</span><span class="sxs-lookup"><span data-stu-id="80ee4-217">If the entity is associated with a different principal/parent, then that relationship is severed.</span></span>
- <span data-ttu-id="80ee4-218">实体将与拥有集合导航的主体/父对象关联。</span><span class="sxs-lookup"><span data-stu-id="80ee4-218">The entity becomes associated with the principal/parent that owns the collection navigation.</span></span>
- <span data-ttu-id="80ee4-219">对于涉及的所有实体，导航和外键值都是固定的。</span><span class="sxs-lookup"><span data-stu-id="80ee4-219">Navigations and foreign key values are fixed up for all entities involved.</span></span>

<span data-ttu-id="80ee4-220">基于此，我们可以看到，从一个博客向另一个博客移动一篇文章，实际上不需要将其从旧的集合导航中删除，然后再将其添加到新的中。</span><span class="sxs-lookup"><span data-stu-id="80ee4-220">Based on this we can see that to move a post from one blog to another we don't actually need to remove it from the old collection navigation before adding it to the new.</span></span> <span data-ttu-id="80ee4-221">因此，可以从上述示例中的代码更改：</span><span class="sxs-lookup"><span data-stu-id="80ee4-221">So the code from the example above can be changed from:</span></span>

<!--
        var post = vsBlog.Posts.Single(e => e.Title.StartsWith("Disassembly improvements"));
        vsBlog.Posts.Remove(post);
        dotNetBlog.Posts.Add(post);
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_1](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_1)]

<span data-ttu-id="80ee4-222">更改为：</span><span class="sxs-lookup"><span data-stu-id="80ee4-222">To:</span></span>

<!--
        var post = vsBlog.Posts.Single(e => e.Title.StartsWith("Disassembly improvements"));
        dotNetBlog.Posts.Add(post);
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_2](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_2)]

<span data-ttu-id="80ee4-223">EF Core 看到张贴内容已添加到新博客，并自动将其从第一个博客上的集合中删除。</span><span class="sxs-lookup"><span data-stu-id="80ee4-223">EF Core sees that the post has been added to a new blog and automatically removes it from the collection on the first blog.</span></span>

### <a name="removing-from-a-collection-navigation"></a><span data-ttu-id="80ee4-224">从集合导航中移除</span><span class="sxs-lookup"><span data-stu-id="80ee4-224">Removing from a collection navigation</span></span>

<span data-ttu-id="80ee4-225">从主体/父对象的集合导航中删除从属/子实体会导致与该主体/父对象的断开。</span><span class="sxs-lookup"><span data-stu-id="80ee4-225">Removing a dependent/child entity from the collection navigation of the principal/parent causes severing of the relationship to that principal/parent.</span></span> <span data-ttu-id="80ee4-226">接下来发生的情况取决于关系是可选的还是必需的。</span><span class="sxs-lookup"><span data-stu-id="80ee4-226">What happens next depends on whether the relationship is optional or required.</span></span>

#### <a name="optional-relationships"></a><span data-ttu-id="80ee4-227">可选关系</span><span class="sxs-lookup"><span data-stu-id="80ee4-227">Optional relationships</span></span>

<span data-ttu-id="80ee4-228">默认情况下，对于可选关系，外键值设置为 null。</span><span class="sxs-lookup"><span data-stu-id="80ee4-228">By default for optional relationships, the foreign key value is set to null.</span></span> <span data-ttu-id="80ee4-229">这意味着依赖/子级不再与 _任何_ 主体/父项关联。</span><span class="sxs-lookup"><span data-stu-id="80ee4-229">This means that the dependent/child is no longer associated with _any_ principal/parent.</span></span> <span data-ttu-id="80ee4-230">例如，让我们加载一个博客和文章，然后从集合导航中删除其中一个帖子 `Blog.Posts` ：</span><span class="sxs-lookup"><span data-stu-id="80ee4-230">For example, let's load a blog and posts and then remove one of the posts from the `Blog.Posts` collection navigation:</span></span>

<!--
        var post = dotNetBlog.Posts.Single(e => e.Title == "Announcing F# 5");
        dotNetBlog.Posts.Remove(post);
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_3](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_3)]

<span data-ttu-id="80ee4-231">在此更改后查看 [更改跟踪调试视图](xref:core/change-tracking/debug-views) 显示：</span><span class="sxs-lookup"><span data-stu-id="80ee4-231">Looking at the [change tracking debug view](xref:core/change-tracking/debug-views) after this change shows that:</span></span>

- <span data-ttu-id="80ee4-232">`Post.BlogId`FK 已设置为 null (`BlogId: <null> FK Modified Originally 1`) </span><span class="sxs-lookup"><span data-stu-id="80ee4-232">The `Post.BlogId` FK has been set to null (`BlogId: <null> FK Modified Originally 1`)</span></span>
- <span data-ttu-id="80ee4-233">`Post.Blog`已将引用导航设置为 null (`Blog: <null>`) </span><span class="sxs-lookup"><span data-stu-id="80ee4-233">The `Post.Blog` reference navigation has been set to null (`Blog: <null>`)</span></span>
- <span data-ttu-id="80ee4-234">已从 `Blog.Posts` 集合导航 (中删除了 post `Posts: [{Id: 1}]`) </span><span class="sxs-lookup"><span data-stu-id="80ee4-234">The post has been removed from `Blog.Posts` collection navigation (`Posts: [{Id: 1}]`)</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Assets: <null>
  Posts: [{Id: 1}]
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
  Tags: []
Post {Id: 2} Modified
  Id: 2 PK
  BlogId: <null> FK Modified Originally 1
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: <null>
  Tags: []
```

<span data-ttu-id="80ee4-235">请注意，post _未_ 标记为 `Deleted` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-235">Notice that the post is _not_ marked as `Deleted`.</span></span> <span data-ttu-id="80ee4-236">它被标记为， `Modified` 以便在调用 SaveChanges 时，数据库中的 FK 值将设置为 null。</span><span class="sxs-lookup"><span data-stu-id="80ee4-236">It is marked as `Modified` so that the FK value in the database will be set to null when SaveChanges is called.</span></span>

#### <a name="required-relationships"></a><span data-ttu-id="80ee4-237">必选关系</span><span class="sxs-lookup"><span data-stu-id="80ee4-237">Required relationships</span></span>

<span data-ttu-id="80ee4-238"> (不允许将 FK 值设置为 null，并且通常不能) 所需的关系。</span><span class="sxs-lookup"><span data-stu-id="80ee4-238">Setting the FK value to null is not allowed (and is usually not possible) for required relationships.</span></span> <span data-ttu-id="80ee4-239">因此，断开是必需的关系，这意味着依赖/子实体必须对新的主体/父实体具有重新父级，或在调用 SaveChanges 时从数据库中删除，以避免引用约束冲突。</span><span class="sxs-lookup"><span data-stu-id="80ee4-239">Therefore, severing a required relationship means that the dependent/child entity must be either re-parented to a new principal/parent, or removed from the database when SaveChanges is called to avoid a referential constraint violation.</span></span> <span data-ttu-id="80ee4-240">这称为 "删除孤立项"，并且是所需关系 EF Core 中的默认行为。</span><span class="sxs-lookup"><span data-stu-id="80ee4-240">This is known as "deleting orphans", and is the default behavior in EF Core for required relationships.</span></span>

<span data-ttu-id="80ee4-241">例如，让我们将博客和帖子之间的关系更改为必需，然后运行与上一示例相同的代码：</span><span class="sxs-lookup"><span data-stu-id="80ee4-241">For example, let's change the relationship between blog and posts to be required and then run the same code as in the previous example:</span></span>

<!--
        var post = dotNetBlog.Posts.Single(e => e.Title == "Announcing F# 5");
        dotNetBlog.Posts.Remove(post);
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_4](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/RequiredRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_4)]

<span data-ttu-id="80ee4-242">在此更改后查看调试视图显示：</span><span class="sxs-lookup"><span data-stu-id="80ee4-242">Looking at the debug view after this change shows that:</span></span>

- <span data-ttu-id="80ee4-243">此 post 已标记为 `Deleted` ，在调用 SaveChanges 时，将从数据库中删除它。</span><span class="sxs-lookup"><span data-stu-id="80ee4-243">The post has been marked as `Deleted` such that it will be deleted from the database when SaveChanges is called.</span></span>
- <span data-ttu-id="80ee4-244">`Post.Blog`已将引用导航设置为 null (`Blog: <null>`) 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-244">The `Post.Blog` reference navigation has been set to null (`Blog: <null>`).</span></span>
- <span data-ttu-id="80ee4-245">已从 `Blog.Posts` 集合导航 () 中删除了 post `Posts: [{Id: 1}]` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-245">The post has been removed from `Blog.Posts` collection navigation (`Posts: [{Id: 1}]`).</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Assets: <null>
  Posts: [{Id: 1}]
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
  Tags: []
Post {Id: 2} Deleted
  Id: 2 PK
  BlogId: 1 FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: <null>
  Tags: []
```

<span data-ttu-id="80ee4-246">请注意， `Post.BlogId` 由于所需的关系，不能将保持不变，因为它不能设置为 null。</span><span class="sxs-lookup"><span data-stu-id="80ee4-246">Notice that the `Post.BlogId` remains unchanged since for a required relationship it cannot be set to null.</span></span>

<span data-ttu-id="80ee4-247">正在删除孤立的日志中的调用 SaveChanges 结果：</span><span class="sxs-lookup"><span data-stu-id="80ee4-247">Calling SaveChanges results in the orphaned post being deleted:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='2' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Posts"
WHERE "Id" = @p0;
SELECT changes();
```

#### <a name="delete-orphans-timing-and-re-parenting"></a><span data-ttu-id="80ee4-248">删除孤立计时并重新进行父级</span><span class="sxs-lookup"><span data-stu-id="80ee4-248">Delete orphans timing and re-parenting</span></span>

<span data-ttu-id="80ee4-249">默认情况下，会在 `Deleted` [检测到](xref:core/change-tracking/change-detection)关系更改时立即标记遗孤。</span><span class="sxs-lookup"><span data-stu-id="80ee4-249">By default, marking orphans as `Deleted` happens as soon as the relationship change is [detected](xref:core/change-tracking/change-detection).</span></span> <span data-ttu-id="80ee4-250">但是，此过程可以延迟到实际调用 SaveChanges。</span><span class="sxs-lookup"><span data-stu-id="80ee4-250">However, this process can be delayed until SaveChanges is actually called.</span></span> <span data-ttu-id="80ee4-251">这有助于避免对已从一个主体/父节点中删除的实体进行孤立，但在调用 SaveChanges 之前，将使用新的主体/父项对其进行重新设置父级。</span><span class="sxs-lookup"><span data-stu-id="80ee4-251">This can be useful to avoid making orphans of entities that have been removed from one principal/parent, but will be re-parented with a new principal/parent before SaveChanges is called.</span></span> <span data-ttu-id="80ee4-252"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DeleteOrphansTiming?displayProperty=nameWithType> 用于设置此计时。</span><span class="sxs-lookup"><span data-stu-id="80ee4-252"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DeleteOrphansTiming?displayProperty=nameWithType> is used to set this timing.</span></span> <span data-ttu-id="80ee4-253">例如：</span><span class="sxs-lookup"><span data-stu-id="80ee4-253">For example:</span></span>

<!--
        context.ChangeTracker.DeleteOrphansTiming = CascadeTiming.OnSaveChanges;

        var post = vsBlog.Posts.Single(e => e.Title.StartsWith("Disassembly improvements"));
        vsBlog.Posts.Remove(post);

        context.ChangeTracker.DetectChanges();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);

        dotNetBlog.Posts.Add(post);

        context.ChangeTracker.DetectChanges();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);

        context.SaveChanges();
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_5](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/RequiredRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_5)]

<span data-ttu-id="80ee4-254">从第一个集合中删除 post 后，该对象不会标记为 `Deleted` 上一示例中的。</span><span class="sxs-lookup"><span data-stu-id="80ee4-254">After removing the post from the first collection the object is not marked as `Deleted` as it was in the previous example.</span></span> <span data-ttu-id="80ee4-255">相反，EF Core 会跟踪关系是否被断开， _即使这是必需的关系_ 也是如此。</span><span class="sxs-lookup"><span data-stu-id="80ee4-255">Instead, EF Core is tracking that the relationship is severed _even though this is a required relationship_.</span></span> <span data-ttu-id="80ee4-256"> (FK 值被 EF Core 视为 null，即使该类型不可为 null，也是如此。</span><span class="sxs-lookup"><span data-stu-id="80ee4-256">(The FK value is considered null by EF Core even though it cannot really be null because the type is not nullable.</span></span> <span data-ttu-id="80ee4-257">这称为 "概念 null"。 ) </span><span class="sxs-lookup"><span data-stu-id="80ee4-257">This is known as a "conceptual null".)</span></span>

```output
Post {Id: 3} Modified
  Id: 3 PK
  BlogId: <null> FK Modified Originally 2
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: <null>
  Tags: []
```

<span data-ttu-id="80ee4-258">此时调用 SaveChanges 会导致删除孤立的 post。</span><span class="sxs-lookup"><span data-stu-id="80ee4-258">Calling SaveChanges at this time would result in the orphaned post being deleted.</span></span> <span data-ttu-id="80ee4-259">但是，如果上面的示例中所示，post 将在调用 SaveChanges 之前与新的博客相关联，然后它将相应地固定到新博客并不再被视为孤立：</span><span class="sxs-lookup"><span data-stu-id="80ee4-259">However, if as in the example above, post is associated with a new blog before SaveChanges is called, then it will be fixed up appropriately to that new blog and is no longer considered an orphan:</span></span>

```output
Post {Id: 3} Modified
  Id: 3 PK
  BlogId: 1 FK Modified Originally 2
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: {Id: 1}
  Tags: []
```

<span data-ttu-id="80ee4-260">此时调用的 SaveChanges 会更新数据库中的 post，而不是将其删除。</span><span class="sxs-lookup"><span data-stu-id="80ee4-260">SaveChanges called at this point will update the post in the database rather than deleting it.</span></span>

<span data-ttu-id="80ee4-261">还可以关闭自动删除孤立项。</span><span class="sxs-lookup"><span data-stu-id="80ee4-261">It is also possible to turn off automatic deletion of orphans.</span></span> <span data-ttu-id="80ee4-262">如果在跟踪孤立时调用 SaveChanges，这将导致异常。</span><span class="sxs-lookup"><span data-stu-id="80ee4-262">This will result in an exception if SaveChanges is called while an orphan is being tracked.</span></span> <span data-ttu-id="80ee4-263">例如，以下代码：</span><span class="sxs-lookup"><span data-stu-id="80ee4-263">For example, this code:</span></span>

<!--
                var dotNetBlog = context.Blogs.Include(e => e.Posts).Single(e => e.Name == ".NET Blog");

                context.ChangeTracker.DeleteOrphansTiming = CascadeTiming.Never;

                var post = dotNetBlog.Posts.Single(e => e.Title == "Announcing F# 5");
                dotNetBlog.Posts.Remove(post);

                context.SaveChanges(); // Throws
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_6](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/RequiredRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_6)]

<span data-ttu-id="80ee4-264">将引发此异常：</span><span class="sxs-lookup"><span data-stu-id="80ee4-264">Will throw this exception:</span></span>

> <span data-ttu-id="80ee4-265">InvalidOperationException：实体 "博客" 和 "Post" （其键值为 "{BlogId： 1}"）之间的关联已被断开，但该关系已标记为必需或被隐式要求，因为外键不可为 null。</span><span class="sxs-lookup"><span data-stu-id="80ee4-265">System.InvalidOperationException: The association between entities 'Blog' and 'Post' with the key value '{BlogId: 1}' has been severed, but the relationship is either marked as required or is implicitly required because the foreign key is not nullable.</span></span> <span data-ttu-id="80ee4-266">如果在所需的关系被切断时应删除从属/子实体，请将关系配置为使用级联删除。</span><span class="sxs-lookup"><span data-stu-id="80ee4-266">If the dependent/child entity should be deleted when a required relationship is severed, configure the relationship to use cascade deletes.</span></span>

<span data-ttu-id="80ee4-267">可以通过调用来随时强制删除孤立项以及级联删除 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeChanges?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-267">Deletion of orphans, as well as cascade deletes, can be forced at any time by calling <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeChanges?displayProperty=nameWithType>.</span></span> <span data-ttu-id="80ee4-268">将此与 "删除孤立计时" 设置相结合 `Never` 将确保不会删除孤立项，除非明确指示 EF Core。</span><span class="sxs-lookup"><span data-stu-id="80ee4-268">Combining this with setting the delete orphan timing to `Never` will ensure orphans are never deleted unless EF Core is explicitly instructed to do so.</span></span>

### <a name="changing-a-reference-navigation"></a><span data-ttu-id="80ee4-269">更改引用导航</span><span class="sxs-lookup"><span data-stu-id="80ee4-269">Changing a reference navigation</span></span>

<span data-ttu-id="80ee4-270">更改一对多关系的引用导航与在关系的另一端更改集合导航的效果相同。</span><span class="sxs-lookup"><span data-stu-id="80ee4-270">Changing the reference navigation of a one-to-many relationship has the same effect as changing the collection navigation on the other end of the relationship.</span></span> <span data-ttu-id="80ee4-271">将从属/子的引用导航设置为 null 等效于从主体/父对象的集合导航中删除实体。</span><span class="sxs-lookup"><span data-stu-id="80ee4-271">Setting the reference navigation of dependent/child to null is equivalent to removing the entity from the collection navigation of the principal/parent.</span></span> <span data-ttu-id="80ee4-272">如前一部分中所述，所有修复和数据库更改都会发生，包括使实体成为孤立实体（如果需要关系）。</span><span class="sxs-lookup"><span data-stu-id="80ee4-272">All fixup and database changes happen as described in the previous section, including making the entity an orphan if the relationship is required.</span></span>

#### <a name="optional-one-to-one-relationships"></a><span data-ttu-id="80ee4-273">可选的一对一关系</span><span class="sxs-lookup"><span data-stu-id="80ee4-273">Optional one-to-one relationships</span></span>

<span data-ttu-id="80ee4-274">对于一对一关系，更改引用导航将导致任何以前的关系被断开。</span><span class="sxs-lookup"><span data-stu-id="80ee4-274">For one-to-one relationships, changing a reference navigation causes any previous relationship to be severed.</span></span> <span data-ttu-id="80ee4-275">对于可选关系，这意味着前面相关的依赖项/子级上的 FK 值设置为 null。</span><span class="sxs-lookup"><span data-stu-id="80ee4-275">For optional relationships, this means that the FK value on the previously related dependent/child is set to null.</span></span> <span data-ttu-id="80ee4-276">例如：</span><span class="sxs-lookup"><span data-stu-id="80ee4-276">For example:</span></span>

<!--
        using var context = new BlogsContext();

        var dotNetBlog = context.Blogs.Include(e => e.Assets).Single(e => e.Name == ".NET Blog");
        dotNetBlog.Assets = new BlogAssets();

        context.ChangeTracker.DetectChanges();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);

        context.SaveChanges();
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_7](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_7)]

<span data-ttu-id="80ee4-277">在调用 SaveChanges 之前，"调试" 视图会显示新资产已替换现有资产，该资产现在标为 `Modified` 空 `BlogAssets.BlogId` FK 值：</span><span class="sxs-lookup"><span data-stu-id="80ee4-277">The debug view before calling SaveChanges shows that the new assets has replaced the existing assets, which is now marked as `Modified` with a null `BlogAssets.BlogId` FK value:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Assets: {Id: -2147482629}
  Posts: []
BlogAssets {Id: -2147482629} Added
  Id: -2147482629 PK Temporary
  Banner: <null>
  BlogId: 1 FK
  Blog: {Id: 1}
BlogAssets {Id: 1} Modified
  Id: 1 PK
  Banner: <null>
  BlogId: <null> FK Modified Originally 1
  Blog: <null>
```

<span data-ttu-id="80ee4-278">这会导致在调用 SaveChanges 时执行更新和插入操作：</span><span class="sxs-lookup"><span data-stu-id="80ee4-278">This results in an update and an insert when SaveChanges is called:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p1='1' (DbType = String), @p0=NULL], CommandType='Text', CommandTimeout='30']
UPDATE "Assets" SET "BlogId" = @p0
WHERE "Id" = @p1;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p2=NULL, @p3='1' (Nullable = true) (DbType = String)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Assets" ("Banner", "BlogId")
VALUES (@p2, @p3);
SELECT "Id"
FROM "Assets"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();
```

#### <a name="required-one-to-one-relationships"></a><span data-ttu-id="80ee4-279">需要一对一关系</span><span class="sxs-lookup"><span data-stu-id="80ee4-279">Required one-to-one relationships</span></span>

<span data-ttu-id="80ee4-280">与上一示例中的代码运行相同的代码，但这一次具有所需的一对一关系，将显示先前关联的 `BlogAssets` 现在已标记为 `Deleted` ，因为在新的位置时，它会变得孤立 `BlogAssets` ：</span><span class="sxs-lookup"><span data-stu-id="80ee4-280">Running the same code as in the previous example, but this time with a required one-to-one relationship, shows that the previously associated `BlogAssets` is now marked as `Deleted`, since it becomes an orphan when the new `BlogAssets` takes its place:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Assets: {Id: -2147482639}
  Posts: []
BlogAssets {Id: -2147482639} Added
  Id: -2147482639 PK Temporary
  Banner: <null>
  BlogId: 1 FK
  Blog: {Id: 1}
BlogAssets {Id: 1} Deleted
  Id: 1 PK
  Banner: <null>
  BlogId: 1 FK
  Blog: <null>
```

<span data-ttu-id="80ee4-281">这样，当调用 SaveChanges 时，将会出现删除并插入操作：</span><span class="sxs-lookup"><span data-stu-id="80ee4-281">This then results in an delete an and insert when SaveChanges is called:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='1' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Assets"
WHERE "Id" = @p0;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p1=NULL, @p2='1' (DbType = String)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Assets" ("Banner", "BlogId")
VALUES (@p1, @p2);
SELECT "Id"
FROM "Assets"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();
```

<span data-ttu-id="80ee4-282">将孤立项标记为已删除的时间可以通过与收集导航显示的相同方式进行更改，并且具有相同的效果。</span><span class="sxs-lookup"><span data-stu-id="80ee4-282">The timing of marking orphans as deleted can be changed in the same way as shown for collection navigations and has the same effects.</span></span>

### <a name="deleting-an-entity"></a><span data-ttu-id="80ee4-283">删除实体</span><span class="sxs-lookup"><span data-stu-id="80ee4-283">Deleting an entity</span></span>

#### <a name="optional-relationships"></a><span data-ttu-id="80ee4-284">可选关系</span><span class="sxs-lookup"><span data-stu-id="80ee4-284">Optional relationships</span></span>

<span data-ttu-id="80ee4-285">当某个实体标记为 `Deleted` （例如通过调用）时，将 <xref:Microsoft.EntityFrameworkCore.DbContext.Remove%2A?displayProperty=nameWithType> 从其他实体的导航中删除对已删除实体的引用。</span><span class="sxs-lookup"><span data-stu-id="80ee4-285">When an entity is marked as `Deleted`, for example by calling <xref:Microsoft.EntityFrameworkCore.DbContext.Remove%2A?displayProperty=nameWithType>, then references to the deleted entity are removed from the navigations of other entities.</span></span> <span data-ttu-id="80ee4-286">对于可选关系，从属实体中的 FK 值将设置为 null。</span><span class="sxs-lookup"><span data-stu-id="80ee4-286">For optional relationships, the FK values in dependent entities are set to null.</span></span>

<span data-ttu-id="80ee4-287">例如，让我们将 Visual Studio 博客标记为 `Deleted` ：</span><span class="sxs-lookup"><span data-stu-id="80ee4-287">For example, let's mark the Visual Studio blog as `Deleted`:</span></span>

<!--
        using var context = new BlogsContext();

        var vsBlog = context.Blogs
            .Include(e => e.Posts)
            .Include(e => e.Assets)
            .Single(e => e.Name == "Visual Studio Blog");

        context.Remove(vsBlog);

        Console.WriteLine(context.ChangeTracker.DebugView.LongView);

        context.SaveChanges();
-->
[!code-csharp[Deleting_an_entity_1](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Deleting_an_entity_1)]

<span data-ttu-id="80ee4-288">在调用 SaveChanges 之前，查看 " [更改跟踪器" 调试视图](xref:core/change-tracking/debug-views) ：</span><span class="sxs-lookup"><span data-stu-id="80ee4-288">Looking at the [change tracker debug view](xref:core/change-tracking/debug-views) before calling SaveChanges shows:</span></span>

```output
Blog {Id: 2} Deleted
  Id: 2 PK
  Name: 'Visual Studio Blog'
  Assets: {Id: 2}
  Posts: [{Id: 3}, {Id: 4}]
BlogAssets {Id: 2} Modified
  Id: 2 PK
  Banner: <null>
  BlogId: <null> FK Modified Originally 2
  Blog: <null>
Post {Id: 3} Modified
  Id: 3 PK
  BlogId: <null> FK Modified Originally 2
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: <null>
  Tags: []
Post {Id: 4} Modified
  Id: 4 PK
  BlogId: <null> FK Modified Originally 2
  Content: 'Examine when database queries were executed and measure how ...'
  Title: 'Database Profiling with Visual Studio'
  Blog: <null>
  Tags: []
```

<span data-ttu-id="80ee4-289">请注意：</span><span class="sxs-lookup"><span data-stu-id="80ee4-289">Notice that:</span></span>

- <span data-ttu-id="80ee4-290">博客标记为 `Deleted`。</span><span class="sxs-lookup"><span data-stu-id="80ee4-290">The blog is marked as `Deleted`.</span></span>
- <span data-ttu-id="80ee4-291">与已删除博客相关的资产具有 null FK 值 (`BlogId: <null> FK Modified Originally 2`) 和空引用导航 (`Blog: <null>`) </span><span class="sxs-lookup"><span data-stu-id="80ee4-291">The assets related to the deleted blog has a null FK value (`BlogId: <null> FK Modified Originally 2`) and a null reference navigation (`Blog: <null>`)</span></span>
- <span data-ttu-id="80ee4-292">与已删除博客相关的每个帖子都具有 null FK 值 (`BlogId: <null> FK Modified Originally 2`) 和空引用导航 (`Blog: <null>`) </span><span class="sxs-lookup"><span data-stu-id="80ee4-292">Each post related to the deleted blog has a null FK value (`BlogId: <null> FK Modified Originally 2`) and a null reference navigation (`Blog: <null>`)</span></span>

#### <a name="required-relationships"></a><span data-ttu-id="80ee4-293">必选关系</span><span class="sxs-lookup"><span data-stu-id="80ee4-293">Required relationships</span></span>

<span data-ttu-id="80ee4-294">所需关系的修复行为与可选关系相同，不同之处在于：依赖/子实体被标记为， `Deleted` 因为它们不能在没有主体/父级的情况下存在，并且在调用 SaveChanges 时必须从数据库中删除，以避免引用约束异常。</span><span class="sxs-lookup"><span data-stu-id="80ee4-294">The fixup behavior for required relationships is the same as for optional relationships except that the dependent/child entities are marked as `Deleted` since they cannot exist without a principal/parent and must be removed from the database when SaveChanges is called to avoid a referential constraint exception.</span></span> <span data-ttu-id="80ee4-295">这称为 "级联删除"，是所需关系 EF Core 中的默认行为。</span><span class="sxs-lookup"><span data-stu-id="80ee4-295">This is known as "cascade delete", and is the default behavior in EF Core for required relationships.</span></span> <span data-ttu-id="80ee4-296">例如，在前面的示例中运行与前面示例中相同的代码，但在调用 SaveChanges 之前，会生成以下调试视图：</span><span class="sxs-lookup"><span data-stu-id="80ee4-296">For example, running the same code as in the previous example but with a required relationship results in the following debug view before SaveChanges is called:</span></span>

```output
Blog {Id: 2} Deleted
  Id: 2 PK
  Name: 'Visual Studio Blog'
  Assets: {Id: 2}
  Posts: [{Id: 3}, {Id: 4}]
BlogAssets {Id: 2} Deleted
  Id: 2 PK
  Banner: <null>
  BlogId: 2 FK
  Blog: {Id: 2}
Post {Id: 3} Deleted
  Id: 3 PK
  BlogId: 2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: {Id: 2}
  Tags: []
Post {Id: 4} Deleted
  Id: 4 PK
  BlogId: 2 FK
  Content: 'Examine when database queries were executed and measure how ...'
  Title: 'Database Profiling with Visual Studio'
  Blog: {Id: 2}
  Tags: []
```

<span data-ttu-id="80ee4-297">按预期，从属项/子项现在标记为 `Deleted` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-297">As expected, the dependents/children are now marked as `Deleted`.</span></span> <span data-ttu-id="80ee4-298">但请注意，已删除的实体上的导航 _未_ 更改。</span><span class="sxs-lookup"><span data-stu-id="80ee4-298">However, notice that the navigations on the deleted entities have _not_ changed.</span></span> <span data-ttu-id="80ee4-299">这似乎很奇怪，但它可以通过清除所有导航来避免完全清除已删除的实体图。</span><span class="sxs-lookup"><span data-stu-id="80ee4-299">This may seem strange, but it avoids completely shredding a deleted graph of entities by clearing all navigations.</span></span> <span data-ttu-id="80ee4-300">也就是说，即使在删除后，博客、资产和帖子仍会形成实体图。</span><span class="sxs-lookup"><span data-stu-id="80ee4-300">That is, the blog, asset, and posts still form a graph of entities even after having been deleted.</span></span> <span data-ttu-id="80ee4-301">这样，就可以更轻松地删除实体关系图，而不是在 EF6 中拆分关系图。</span><span class="sxs-lookup"><span data-stu-id="80ee4-301">This makes it much easier to un-delete a graph of entities than was the case in EF6 where the graph was shredded.</span></span>

#### <a name="cascade-delete-timing-and-re-parenting"></a><span data-ttu-id="80ee4-302">级联删除计时和重新父级</span><span class="sxs-lookup"><span data-stu-id="80ee4-302">Cascade delete timing and re-parenting</span></span>

<span data-ttu-id="80ee4-303">默认情况下，当父/主体标记为时，将立即执行级联删除 `Deleted` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-303">By default, cascade delete happens as soon as the parent/principal is marked as `Deleted`.</span></span> <span data-ttu-id="80ee4-304">这与删除孤立项相同，如前文所述。</span><span class="sxs-lookup"><span data-stu-id="80ee4-304">This is the same as for deleting orphans, as described previously.</span></span> <span data-ttu-id="80ee4-305">与删除孤立项一样，此过程可能会延迟，直到通过适当的设置调用 SaveChanges 或完全禁用它 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeDeleteTiming?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-305">As with deleting orphans, this process can be delayed until SaveChanges is called, or even disabled entirely, by setting <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeDeleteTiming?displayProperty=nameWithType> appropriately.</span></span> <span data-ttu-id="80ee4-306">这种方法的使用方式与删除孤立对象的方式相同，包括删除主体/父项后用于重新父级子/依赖项。</span><span class="sxs-lookup"><span data-stu-id="80ee4-306">This is useful in the same way as it is for deleting orphans, including for re-parenting children/dependents after deletion of a principal/parent.</span></span>

<span data-ttu-id="80ee4-307">通过调用，可以随时强制执行级联删除操作，以及删除孤立项 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeChanges?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-307">Cascade deletes, as well as deleting orphans, can be forced at any time by calling <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeChanges?displayProperty=nameWithType>.</span></span> <span data-ttu-id="80ee4-308">将此组合到将级联删除计时设置为 `Never` 将确保级联删除永远不会发生，除非显式指示 EF Core。</span><span class="sxs-lookup"><span data-stu-id="80ee4-308">Combining this with setting the cascade delete timing to `Never` will ensure cascade deletes never happen unless EF Core is explicitly instructed to do so.</span></span>

> [!TIP]
> <span data-ttu-id="80ee4-309">级联删除和删除孤立项是密切相关的。</span><span class="sxs-lookup"><span data-stu-id="80ee4-309">Cascade delete and deleting orphans are closely related.</span></span> <span data-ttu-id="80ee4-310">当断开与所需的主体/父实体之间的关系时，两者都将导致删除依赖实体/子实体。</span><span class="sxs-lookup"><span data-stu-id="80ee4-310">Both result in deleting dependent/child entities when the relationship to their required principal/parent is severed.</span></span> <span data-ttu-id="80ee4-311">对于级联删除，由于主体/父实体本身已删除，因此发生了这种断开。</span><span class="sxs-lookup"><span data-stu-id="80ee4-311">For cascade delete, this severing happens because the principal/parent is itself deleted.</span></span> <span data-ttu-id="80ee4-312">对于孤立项，主体/父实体仍然存在，但不再与依赖实体/子实体相关。</span><span class="sxs-lookup"><span data-stu-id="80ee4-312">For orphans, the principal/parent entity still exists, but is no longer related to the dependent/child entities.</span></span>

## <a name="many-to-many-relationships"></a><span data-ttu-id="80ee4-313">多对多关系</span><span class="sxs-lookup"><span data-stu-id="80ee4-313">Many-to-many relationships</span></span>

<span data-ttu-id="80ee4-314">EF Core 中的多对多关系是使用联接实体实现的。</span><span class="sxs-lookup"><span data-stu-id="80ee4-314">Many-to-many relationships in EF Core are implemented using a join entity.</span></span> <span data-ttu-id="80ee4-315">多对多关系的每一方都与具有一对多关系的此联接实体相关。</span><span class="sxs-lookup"><span data-stu-id="80ee4-315">Each side the many-to-many relationship is related to this join entity with a one-to-many relationship.</span></span> <span data-ttu-id="80ee4-316">在 EF Core 5.0 之前，必须显式定义和映射此联接实体。</span><span class="sxs-lookup"><span data-stu-id="80ee4-316">Before EF Core 5.0, this join entity had to explicitly defined and mapped.</span></span> <span data-ttu-id="80ee4-317">从 EF Core 5.0 开始，可以隐式创建并隐藏。</span><span class="sxs-lookup"><span data-stu-id="80ee4-317">Starting with EF Core 5.0, it can be created implicitly and hidden.</span></span> <span data-ttu-id="80ee4-318">但是，在这两种情况下，基础行为都是相同的。</span><span class="sxs-lookup"><span data-stu-id="80ee4-318">However, in both cases the underlying behavior is the same.</span></span> <span data-ttu-id="80ee4-319">首先，我们将介绍此基础行为，以了解如何跟踪多对多关系的工作方式。</span><span class="sxs-lookup"><span data-stu-id="80ee4-319">We will look at this underlying behavior first to understand how tracking of many-to-many relationships works.</span></span>

### <a name="how-many-to-many-relationships-work"></a><span data-ttu-id="80ee4-320">多对多关系的工作方式</span><span class="sxs-lookup"><span data-stu-id="80ee4-320">How many-to-many relationships work</span></span>

<span data-ttu-id="80ee4-321">请考虑这 EF Core 模型，该模型使用显式定义的联接实体类型在 post 和标记之间创建多对多关系：</span><span class="sxs-lookup"><span data-stu-id="80ee4-321">Consider this EF Core model that creates a many-to-many relationship between posts and tags using an explicitly defined join entity type:</span></span>

<!--
    public class Post
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }

        public int? BlogId { get; set; }
        public Blog Blog { get; set; }

        public IList<PostTag> PostTags { get; } = new List<PostTag>(); // Collection navigation
    }

    public class Tag
    {
        public int Id { get; set; }
        public string Text { get; set; }

        public IList<PostTag> PostTags { get; } = new List<PostTag>(); // Collection navigation
    }

    public class PostTag
    {
        public int PostId { get; set; } // First part of composite PK; FK to Post
        public int TagId { get; set; } // Second part of composite PK; FK to Tag

        public Post Post { get; set; } // Reference navigation
        public Tag Tag { get; set; } // Reference navigation
    }
    -->
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntitySamples.cs?name=Model)]

<span data-ttu-id="80ee4-322">请注意， `PostTag` 联接实体类型包含两个外键属性。</span><span class="sxs-lookup"><span data-stu-id="80ee4-322">Notice that the `PostTag` join entity type contains two foreign key properties.</span></span> <span data-ttu-id="80ee4-323">在此模型中，对于要与标记相关的 post，必须有一个 PostTag 联接实体 `PostTag.PostId` ，其中外键值与 `Post.Id` 主键值匹配， `PostTag.TagId` 外键值与 `Tag.Id` 主键值匹配。</span><span class="sxs-lookup"><span data-stu-id="80ee4-323">In this model, for a post to be related to a tag, there must be a PostTag join entity where the `PostTag.PostId` foreign key value matches the `Post.Id` primary key value, and where the `PostTag.TagId` foreign key value matches the `Tag.Id` primary key value.</span></span> <span data-ttu-id="80ee4-324">例如：</span><span class="sxs-lookup"><span data-stu-id="80ee4-324">For example:</span></span>

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            context.Add(new PostTag { PostId = post.Id, TagId = tag.Id });

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Many_to_many_relationships_1](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntitySamples.cs?name=Many_to_many_relationships_1)]

<span data-ttu-id="80ee4-325">运行此代码后查看 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图，以显示 post 和标记是由新的 "联接" 实体相关联的 `PostTag` ：</span><span class="sxs-lookup"><span data-stu-id="80ee4-325">Looking at the [change tracker debug view](xref:core/change-tracking/debug-views) after running this code shows that the post and tag are related by the new `PostTag` join entity:</span></span>

```output
Post {Id: 3} Unchanged
  Id: 3 PK
  BlogId: 2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: <null>
  PostTags: [{PostId: 3, TagId: 1}]
PostTag {PostId: 3, TagId: 1} Added
  PostId: 3 PK FK
  TagId: 1 PK FK
  Post: {Id: 3}
  Tag: {Id: 1}
Tag {Id: 1} Unchanged
  Id: 1 PK
  Text: '.NET'
  PostTags: [{PostId: 3, TagId: 1}]
```

<span data-ttu-id="80ee4-326">请注意，上的集合导航已 `Post` `Tag` 修复，并已在上进行引用导航 `PostTag` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-326">Notice that the collection navigations on `Post` and `Tag` have been fixed up, as have the reference navigations on `PostTag`.</span></span> <span data-ttu-id="80ee4-327">可以通过导航而不是 FK 值来操作这些关系，就像前面的示例中所述。</span><span class="sxs-lookup"><span data-stu-id="80ee4-327">These relationships can be manipulated by navigations instead of FK values, just as in all the preceding examples.</span></span> <span data-ttu-id="80ee4-328">例如，可以通过在联接实体上设置引用导航来修改上述代码以添加关系：</span><span class="sxs-lookup"><span data-stu-id="80ee4-328">For example, the code above can be modified to add the relationship by setting the reference navigations on the join entity:</span></span>

<!--
            context.Add(new PostTag { Post = post, Tag = tag });
-->
[!code-csharp[Many_to_many_relationships_2](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntitySamples.cs?name=Many_to_many_relationships_2)]

<span data-ttu-id="80ee4-329">这会导致与上一个示例中的 Fk 和导航完全相同的更改。</span><span class="sxs-lookup"><span data-stu-id="80ee4-329">This results in exactly the same change to FKs and navigations as in the previous example.</span></span>

### <a name="skip-navigations"></a><span data-ttu-id="80ee4-330">跳过导航</span><span class="sxs-lookup"><span data-stu-id="80ee4-330">Skip navigations</span></span>

> [!NOTE]
> <span data-ttu-id="80ee4-331">EF Core 5.0 中引入了跳过导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-331">Skip navigations were introduced in EF Core 5.0.</span></span>

<span data-ttu-id="80ee4-332">手动操作联接表可能比较繁琐。</span><span class="sxs-lookup"><span data-stu-id="80ee4-332">Manipulating the join table manually can be cumbersome.</span></span> <span data-ttu-id="80ee4-333">从 EF Core 5.0 开始，可以使用 "跳过" 联接实体的特殊集合导航直接操作多对多关系。</span><span class="sxs-lookup"><span data-stu-id="80ee4-333">Starting with EF Core 5.0, many-to-many relationships can be manipulated directly using special collection navigations that "skip over" the join entity.</span></span> <span data-ttu-id="80ee4-334">例如，可以将两个 skip 导航添加到上述模型;一个从 Post 到标记，另一个从标记到 post：</span><span class="sxs-lookup"><span data-stu-id="80ee4-334">For example, two skip navigations can be added to the model above; one from Post to Tags, and the other from Tag to Posts:</span></span>

<!--
    public class Post
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }

        public int? BlogId { get; set; }
        public Blog Blog { get; set; }

        public IList<Tag> Tags { get; } = new List<Tag>(); // Skip collection navigation
        public IList<PostTag> PostTags { get; } = new List<PostTag>(); // Collection navigation
    }

    public class Tag
    {
        public int Id { get; set; }
        public string Text { get; set; }

        public IList<Post> Posts { get; } = new List<Post>(); // Skip collection navigation
        public IList<PostTag> PostTags { get; } = new List<PostTag>(); // Collection navigation
    }

    public class PostTag
    {
        public int PostId { get; set; } // First part of composite PK; FK to Post
        public int TagId { get; set; } // Second part of composite PK; FK to Tag

        public Post Post { get; set; } // Reference navigation
        public Tag Tag { get; set; } // Reference navigation
    }
-->
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityAndSkipsSamples.cs?name=Model)]

<span data-ttu-id="80ee4-335">这种多对多关系要求使用以下配置，以确保跳过导航和普通导航均用于相同的多对多关系：</span><span class="sxs-lookup"><span data-stu-id="80ee4-335">This many-to-many relationship requires the following configuration to ensure the skip navigations and normal navigations are all used for the same many-to-many relationship:</span></span>

<!--
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Post>()
                .HasMany(p => p.Tags)
                .WithMany(p => p.Posts)
                .UsingEntity<PostTag>(
                    j => j.HasOne(t => t.Tag).WithMany(p => p.PostTags),
                    j => j.HasOne(t => t.Post).WithMany(p => p.PostTags));
        }
-->
[!code-csharp[OnModelCreating](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityAndSkipsSamples.cs?name=OnModelCreating)]

<span data-ttu-id="80ee4-336">有关映射多对多关系的详细信息，请参阅 [关系](xref:core/modeling/relationships) 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-336">See [Relationships](xref:core/modeling/relationships) for more information on mapping many-to-many relationships.</span></span>

<span data-ttu-id="80ee4-337">跳过导航外观并表现为正常的集合导航。</span><span class="sxs-lookup"><span data-stu-id="80ee4-337">Skip navigations look and behave like normal collection navigations.</span></span> <span data-ttu-id="80ee4-338">但是，它们使用外键值的方式不同。</span><span class="sxs-lookup"><span data-stu-id="80ee4-338">However, the way they work with foreign key values is different.</span></span> <span data-ttu-id="80ee4-339">让我们将帖子与标记相关联，但这次使用的是 skip 导航：</span><span class="sxs-lookup"><span data-stu-id="80ee4-339">Let's associate a post with a tag, but this time using a skip navigation:</span></span>

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            post.Tags.Add(tag);

            context.ChangeTracker.DetectChanges();
            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Many_to_many_relationships_3](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityAndSkipsSamples.cs?name=Many_to_many_relationships_3)]

<span data-ttu-id="80ee4-340">请注意，此代码不使用 join 实体。</span><span class="sxs-lookup"><span data-stu-id="80ee4-340">Notice that this code doesn't use the join entity.</span></span> <span data-ttu-id="80ee4-341">而只是将实体添加到导航集合中，就像在这是一种一对多关系时所做的一样。</span><span class="sxs-lookup"><span data-stu-id="80ee4-341">It instead just adds an entity to a navigation collection in the same way as would be done if this were a one-to-many relationship.</span></span> <span data-ttu-id="80ee4-342">生成的调试视图实质上与以前相同：</span><span class="sxs-lookup"><span data-stu-id="80ee4-342">The resulting debug view is essentially the same as before:</span></span>

```output
Post {Id: 3} Unchanged
  Id: 3 PK
  BlogId: 2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: <null>
  PostTags: [{PostId: 3, TagId: 1}]
  Tags: [{Id: 1}]
PostTag {PostId: 3, TagId: 1} Added
  PostId: 3 PK FK
  TagId: 1 PK FK
  Post: {Id: 3}
  Tag: {Id: 1}
Tag {Id: 1} Unchanged
  Id: 1 PK
  Text: '.NET'
  PostTags: [{PostId: 3, TagId: 1}]
  Posts: [{Id: 3}]
```

<span data-ttu-id="80ee4-343">请注意， `PostTag` 会自动创建一个联接实体实例，并将 FK 值设置为现在关联的标记和 post 的 PK 值。</span><span class="sxs-lookup"><span data-stu-id="80ee4-343">Notice that an instance of the `PostTag` join entity was created automatically with FK values set to the PK values of the tag and post that are now associated.</span></span> <span data-ttu-id="80ee4-344">所有常规引用和集合导航都已修复，可与这些 FK 值匹配。</span><span class="sxs-lookup"><span data-stu-id="80ee4-344">All the normal reference and collection navigations have been fixed up to match these FK values.</span></span> <span data-ttu-id="80ee4-345">此外，由于此模型包含跳过导航，因此它们也已修复。</span><span class="sxs-lookup"><span data-stu-id="80ee4-345">Also, since this model contains skip navigations, these have also been fixed up.</span></span> <span data-ttu-id="80ee4-346">具体而言，即使我们将标记添加到了 " `Post.Tags` 跳过导航"， `Tag.Posts` 也已将此关系的另一端上的反向跳过导航固定到包含关联的 post。</span><span class="sxs-lookup"><span data-stu-id="80ee4-346">Specifically, even though we added the tag to the `Post.Tags` skip navigation, the `Tag.Posts` inverse skip navigation on the other side of this relationship has also been fixed up to contain the associated post.</span></span>

<span data-ttu-id="80ee4-347">值得一提的是，即使已将跳过导航分层在顶部，也仍可以直接操作基础多对多关系。</span><span class="sxs-lookup"><span data-stu-id="80ee4-347">It is worth noting that the underlying many-to-many relationships can still be manipulated directly even when skip navigations have been layered on top.</span></span> <span data-ttu-id="80ee4-348">例如，标记和帖子可能会与我们在引入 skip 导航之前所做的关联：</span><span class="sxs-lookup"><span data-stu-id="80ee4-348">For example, the tag and Post could be associated as we did before introducing skip navigations:</span></span>

<!--
            context.Add(new PostTag { Post = post, Tag = tag });
-->
[!code-csharp[Many_to_many_relationships_4](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityAndSkipsSamples.cs?name=Many_to_many_relationships_4)]

<span data-ttu-id="80ee4-349">或使用 FK 值：</span><span class="sxs-lookup"><span data-stu-id="80ee4-349">Or using FK values:</span></span>

<!--
            context.Add(new PostTag { PostId = post.Id, TagId = tag.Id });
-->
[!code-csharp[Many_to_many_relationships_5](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityAndSkipsSamples.cs?name=Many_to_many_relationships_5)]

<span data-ttu-id="80ee4-350">这仍将导致正确地固定 "跳过" 导航，结果与上一示例中的调试视图输出相同。</span><span class="sxs-lookup"><span data-stu-id="80ee4-350">This will still result in the skip navigations being fixed up correctly, resulting in the same debug view output as in the previous example.</span></span>

### <a name="skip-navigations-only"></a><span data-ttu-id="80ee4-351">仅跳过导航</span><span class="sxs-lookup"><span data-stu-id="80ee4-351">Skip navigations only</span></span>

<span data-ttu-id="80ee4-352">在上一部分中，我们添加了跳过导航 _，并_ 完全定义了两个基础一对多关系。</span><span class="sxs-lookup"><span data-stu-id="80ee4-352">In the previous section we added skip navigations _in addition to_ fully defining the two underlying one-to-many relationships.</span></span> <span data-ttu-id="80ee4-353">这非常适用于说明 FK 值发生的情况，但通常不需要这样做。</span><span class="sxs-lookup"><span data-stu-id="80ee4-353">This is useful to illustrate what happens to FK values, but is often unnecessary.</span></span> <span data-ttu-id="80ee4-354">相反，可以使用 " _跳过导航_" 定义多对多关系。</span><span class="sxs-lookup"><span data-stu-id="80ee4-354">Instead, the many-to-many relationship can be defined using _only skip navigations_.</span></span> <span data-ttu-id="80ee4-355">这是在本文档最顶部的模型中定义多对多关系的方式。</span><span class="sxs-lookup"><span data-stu-id="80ee4-355">This is how the many-to-many relationship is defined in the model at the very top of this document.</span></span> <span data-ttu-id="80ee4-356">使用此模型，我们可以通过将 post 添加到 " `Tag.Posts` 跳过导航" (或者将标记添加到 "跳过导航") 来关联 post 和标记 `Post.Tags` ：</span><span class="sxs-lookup"><span data-stu-id="80ee4-356">Using this model, we can again associate a Post and a Tag by adding a post to the `Tag.Posts` skip navigation (or, alternately, adding a tag to the `Post.Tags` skip navigation):</span></span>

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            post.Tags.Add(tag);

            context.ChangeTracker.DetectChanges();
            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Many_to_many_relationships_6](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Many_to_many_relationships_6)]

<span data-ttu-id="80ee4-357">做出此更改后，查看 "调试" 视图会显示 EF Core 已创建了一个 `Dictionary<string, object>` 用于表示联接实体的实例。</span><span class="sxs-lookup"><span data-stu-id="80ee4-357">Looking at the debug view after making this change reveals that EF Core has created an instance of `Dictionary<string, object>` to represent the join entity.</span></span> <span data-ttu-id="80ee4-358">此联接实体包含 `PostsId` 和 `TagsId` 外键属性，这些属性已设置为与关联的 post 和标记的 PK 值相匹配。</span><span class="sxs-lookup"><span data-stu-id="80ee4-358">This join entity contains both `PostsId` and `TagsId` foreign key properties which have been set to match the PK values of the post and tag that are associated.</span></span>

```output
Post {Id: 3} Unchanged
  Id: 3 PK
  BlogId: 2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: <null>
  Tags: [{Id: 1}]
Tag {Id: 1} Unchanged
  Id: 1 PK
  Text: '.NET'
  Posts: [{Id: 3}]
PostTag (Dictionary<string, object>) {PostsId: 3, TagsId: 1} Added
  PostsId: 3 PK FK
  TagsId: 1 PK FK
```

<span data-ttu-id="80ee4-359">有关隐式联接实体和实体类型的使用的详细信息，请参阅 [关系](xref:core/modeling/relationships) `Dictionary<string, object>` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-359">See [Relationships](xref:core/modeling/relationships) for more information about implicit join entities and the use of `Dictionary<string, object>` entity types.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="80ee4-360">用于按约定联接实体类型的 CLR 类型在将来的版本中可能会更改以提高性能。</span><span class="sxs-lookup"><span data-stu-id="80ee4-360">The CLR type used for join entity types by convention may change in future releases to improve performance.</span></span> <span data-ttu-id="80ee4-361">不要依赖于联接类型， `Dictionary<string, object>` 除非显式配置了此类型。</span><span class="sxs-lookup"><span data-stu-id="80ee4-361">Do not depend on the join type being `Dictionary<string, object>` unless this has been explicitly configured.</span></span>

### <a name="join-entities-with-payloads"></a><span data-ttu-id="80ee4-362">将实体与负载联接</span><span class="sxs-lookup"><span data-stu-id="80ee4-362">Join entities with payloads</span></span>

<span data-ttu-id="80ee4-363">到目前为止，所有示例均使用联接实体类型 (显式或隐式) 只包含多对多关系所需的两个外键属性。</span><span class="sxs-lookup"><span data-stu-id="80ee4-363">So far all the examples have used a join entity type (whether explicit or implicit) that contains only the two foreign key properties needed for the many-to-many relationship.</span></span> <span data-ttu-id="80ee4-364">当处理关系时，这两个 FK 值都需要由应用程序显式设置，因为它们的值来自相关实体的主键属性。</span><span class="sxs-lookup"><span data-stu-id="80ee4-364">Neither of these FK values need to be explicitly set by the application when manipulating relationships because their values come from the primary key properties of the related entities.</span></span> <span data-ttu-id="80ee4-365">这允许 EF Core 创建联接实体的实例，而不会丢失数据。</span><span class="sxs-lookup"><span data-stu-id="80ee4-365">This allows EF Core to create instances of the join entity without missing data.</span></span>

#### <a name="payloads-with-generated-values"></a><span data-ttu-id="80ee4-366">具有生成值的负载</span><span class="sxs-lookup"><span data-stu-id="80ee4-366">Payloads with generated values</span></span>

<span data-ttu-id="80ee4-367">EF Core 支持向联接实体类型添加其他属性。</span><span class="sxs-lookup"><span data-stu-id="80ee4-367">EF Core supports adding additional properties to the join entity type.</span></span> <span data-ttu-id="80ee4-368">这就是所谓的 "有效负载" 实体。</span><span class="sxs-lookup"><span data-stu-id="80ee4-368">This is known as giving the join entity a "payload".</span></span> <span data-ttu-id="80ee4-369">例如，让我们将 `TaggedOn` 属性添加到 `PostTag` 联接实体：</span><span class="sxs-lookup"><span data-stu-id="80ee4-369">For example, let's add `TaggedOn` property to the `PostTag` join entity:</span></span>

<!--
    public class PostTag
    {
        public int PostId { get; set; } // Foreign key to Post
        public int TagId { get; set; } // Foreign key to Tag

        public DateTime TaggedOn { get; set; } // Payload
    }
-->
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityWithPayloadSamples.cs?name=Model)]

<span data-ttu-id="80ee4-370">EF Core 创建联接实体实例时，将不会设置此负载属性。</span><span class="sxs-lookup"><span data-stu-id="80ee4-370">This payload property will not be set when EF Core creates a join entity instance.</span></span> <span data-ttu-id="80ee4-371">处理此情况的最常见方法是使用负载属性和自动生成的值。</span><span class="sxs-lookup"><span data-stu-id="80ee4-371">The most common way to deal with this is to use payload properties with automatically generated values.</span></span> <span data-ttu-id="80ee4-372">例如，在 `TaggedOn` 插入每个新实体时，可以将属性配置为使用存储生成的时间戳：</span><span class="sxs-lookup"><span data-stu-id="80ee4-372">For example, the `TaggedOn` property can be configured to use a store-generated timestamp when each new entity is inserted:</span></span>

<!--
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Post>()
                .HasMany(p => p.Tags)
                .WithMany(p => p.Posts)
                .UsingEntity<PostTag>(
                    j => j.HasOne<Tag>().WithMany(),
                    j => j.HasOne<Post>().WithMany(),
                    j => j.Property(e => e.TaggedOn).HasDefaultValueSql("CURRENT_TIMESTAMP"));
        }
-->
[!code-csharp[OnModelCreating](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityWithPayloadSamples.cs?name=OnModelCreating)]

<span data-ttu-id="80ee4-373">现在，可以使用与之前相同的方式来标记 post：</span><span class="sxs-lookup"><span data-stu-id="80ee4-373">A post can now be tagged in the same way as before:</span></span>

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            post.Tags.Add(tag);

            context.SaveChanges();

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Many_to_many_relationships_7](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityWithPayloadSamples.cs?name=Many_to_many_relationships_7)]

<span data-ttu-id="80ee4-374">在调用 SaveChanges 后查看 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图会显示已正确设置了负载属性：</span><span class="sxs-lookup"><span data-stu-id="80ee4-374">Looking at the [change tracker debug view](xref:core/change-tracking/debug-views) after calling SaveChanges shows that the payload property has been set appropriately:</span></span>

```output
Post {Id: 3} Unchanged
  Id: 3 PK
  BlogId: 2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: <null>
  Tags: [{Id: 1}]
PostTag {PostId: 3, TagId: 1} Unchanged
  PostId: 3 PK FK
  TagId: 1 PK FK
  TaggedOn: '12/29/2020 8:13:21 PM'
Tag {Id: 1} Unchanged
  Id: 1 PK
  Text: '.NET'
  Posts: [{Id: 3}]
```

#### <a name="explicitly-setting-payload-values"></a><span data-ttu-id="80ee4-375">显式设置负载值</span><span class="sxs-lookup"><span data-stu-id="80ee4-375">Explicitly setting payload values</span></span>

<span data-ttu-id="80ee4-376">在前面的示例中，让我们添加一个不使用自动生成的值的负载属性：</span><span class="sxs-lookup"><span data-stu-id="80ee4-376">Following on from the previous example, let's add a payload property that does not use an automatically generated value:</span></span>

<!--
    public class PostTag
    {
        public int PostId { get; set; } // First part of composite PK; FK to Post
        public int TagId { get; set; } // Second part of composite PK; FK to Tag

        public DateTime TaggedOn { get; set; } // Auto-generated payload property
        public string TaggedBy { get; set; } // Not-generated payload property
    }
-->
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityWithStringPayloadSamples.cs?name=Model)]

<span data-ttu-id="80ee4-377">现在，可以使用与之前相同的方式来标记 post，并且仍将自动创建联接实体。</span><span class="sxs-lookup"><span data-stu-id="80ee4-377">A post can now be tagged in the same way as before, and the join entity will still be created automatically.</span></span> <span data-ttu-id="80ee4-378">然后，可以使用 [访问跟踪的实体](xref:core/change-tracking/entity-entries)中所述的一种机制来访问此实体。</span><span class="sxs-lookup"><span data-stu-id="80ee4-378">This entity can then be accessed using one of the mechanisms described in [Accessing Tracked Entities](xref:core/change-tracking/entity-entries).</span></span> <span data-ttu-id="80ee4-379">例如，下面的代码使用 <xref:Microsoft.EntityFrameworkCore.DbSet%601.Find%2A?displayProperty=nameWithType> 访问联接实体实例：</span><span class="sxs-lookup"><span data-stu-id="80ee4-379">For example, the code below uses <xref:Microsoft.EntityFrameworkCore.DbSet%601.Find%2A?displayProperty=nameWithType> to access the join entity instance:</span></span>

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            post.Tags.Add(tag);

            context.ChangeTracker.DetectChanges();

            var joinEntity = context.Set<PostTag>().Find(post.Id, tag.Id);

            joinEntity.TaggedBy = "ajcvickers";

            context.SaveChanges();

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Many_to_many_relationships_8](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityWithStringPayloadSamples.cs?name=Many_to_many_relationships_8)]

<span data-ttu-id="80ee4-380">找到联接实体后，可以通过以下方式进行操作：在此示例中，在 `TaggedBy` 调用 SaveChanges 之前设置负载属性。</span><span class="sxs-lookup"><span data-stu-id="80ee4-380">Once the join entity has been located it can be manipulated in the normal way--in this example, to set the `TaggedBy` payload property before calling SaveChanges.</span></span>

> [!NOTE]
> <span data-ttu-id="80ee4-381">请注意，若要在 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> 使用之前允许 EF Core 检测导航属性更改和创建联接实体实例，需要在此处进行调用 `Find` 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-381">Note that a call to <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> is required here to give EF Core a chance to detect the navigation property change and create the join entity instance before `Find` is used.</span></span> <span data-ttu-id="80ee4-382">有关详细信息，请参阅 [更改检测和通知](xref:core/change-tracking/change-detection) 。</span><span class="sxs-lookup"><span data-stu-id="80ee4-382">See [Change Detection and Notifications](xref:core/change-tracking/change-detection) for more information.</span></span>

<span data-ttu-id="80ee4-383">或者，可以显式创建联接实体，以将 post 与标记相关联。</span><span class="sxs-lookup"><span data-stu-id="80ee4-383">Alternately, the join entity can be created explicitly to associate a post with a tag.</span></span> <span data-ttu-id="80ee4-384">例如：</span><span class="sxs-lookup"><span data-stu-id="80ee4-384">For example:</span></span>

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            context.Add(new PostTag()
            {
                PostId = post.Id,
                TagId = tag.Id,
                TaggedBy = "ajcvickers"
            });

            context.SaveChanges();

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Many_to_many_relationships_9](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityWithStringPayloadSamples.cs?name=Many_to_many_relationships_9)]

<span data-ttu-id="80ee4-385">最后，设置负载数据的另一种方法是重写 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 或使用 <xref:Microsoft.EntityFrameworkCore.DbContext.SavingChanges?displayProperty=nameWithType> 事件来处理实体，然后再更新数据库。</span><span class="sxs-lookup"><span data-stu-id="80ee4-385">Finally, another way to set payload data is by either overriding <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> or using the <xref:Microsoft.EntityFrameworkCore.DbContext.SavingChanges?displayProperty=nameWithType> event to process entities before updating the database.</span></span> <span data-ttu-id="80ee4-386">例如：</span><span class="sxs-lookup"><span data-stu-id="80ee4-386">For example:</span></span>

<!--
        public override int SaveChanges()
        {
            foreach (var entityEntry in ChangeTracker.Entries<PostTag>())
            {
                if (entityEntry.State == EntityState.Added)
                {
                    entityEntry.Entity.TaggedBy = "ajcvickers";
                }
            }

            return base.SaveChanges();
        }
-->
[!code-csharp[SaveChanges](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityWithStringPayloadSamples.cs?name=SaveChanges)]
