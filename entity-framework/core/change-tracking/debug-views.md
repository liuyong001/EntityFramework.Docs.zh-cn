---
title: 更改跟踪器调试-EF Core
description: 使用 ChangeTracker DebugView 和日志消息调试 EF Core 更改跟踪
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/debug-views
ms.openlocfilehash: 76108120cf7f532d085fef12121bb639b065add0
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129577"
---
# <a name="change-tracker-debugging"></a><span data-ttu-id="21986-103">更改跟踪器调试</span><span class="sxs-lookup"><span data-stu-id="21986-103">Change Tracker Debugging</span></span>

<span data-ttu-id="21986-104">Entity Framework Core (EF Core) 更改跟踪器会生成两种输出，以帮助进行调试：</span><span class="sxs-lookup"><span data-stu-id="21986-104">The Entity Framework Core (EF Core) change tracker generates two kinds of output to help with debugging:</span></span>

- <span data-ttu-id="21986-105"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DebugView%2A?displayProperty=nameWithType>提供要跟踪的所有实体的用户可读视图</span><span class="sxs-lookup"><span data-stu-id="21986-105">The <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DebugView%2A?displayProperty=nameWithType> provides a human-readable view of all entities being tracked</span></span>
- <span data-ttu-id="21986-106">当更改跟踪器检测状态并修复关系时，将生成调试级日志消息</span><span class="sxs-lookup"><span data-stu-id="21986-106">Debug-level log messages are generated when the change tracker detects state and fixes up relationships</span></span>

> [!TIP]
> <span data-ttu-id="21986-107">本文档假设了解 EF Core 更改跟踪的实体状态和基础知识。</span><span class="sxs-lookup"><span data-stu-id="21986-107">This document assumes that entity states and the basics of EF Core change tracking are understood.</span></span> <span data-ttu-id="21986-108">有关这些主题的详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。</span><span class="sxs-lookup"><span data-stu-id="21986-108">See [Change Tracking in EF Core](xref:core/change-tracking/index) for more information on these topics.</span></span>

> [!TIP]
> <span data-ttu-id="21986-109">通过 [从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangeTrackerDebugging)，你可以运行并调试到本文档中的所有代码。</span><span class="sxs-lookup"><span data-stu-id="21986-109">You can run and debug into all the code in this document by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangeTrackerDebugging).</span></span>

## <a name="change-tracker-debug-view"></a><span data-ttu-id="21986-110">更改跟踪器调试视图</span><span class="sxs-lookup"><span data-stu-id="21986-110">Change tracker debug view</span></span>

<span data-ttu-id="21986-111">可以在 IDE 的调试器中访问更改跟踪程序调试视图。</span><span class="sxs-lookup"><span data-stu-id="21986-111">The change tracker debug view can be accessed in the debugger of your IDE.</span></span> <span data-ttu-id="21986-112">例如，对于 Visual Studio：</span><span class="sxs-lookup"><span data-stu-id="21986-112">For example, with Visual Studio:</span></span>

![从 Visual Studio 调试器访问更改跟踪程序调试视图](_static/debug-view.png)

<span data-ttu-id="21986-114">它还可以直接从代码进行访问，例如将调试视图发送到控制台：</span><span class="sxs-lookup"><span data-stu-id="21986-114">It can also be accessed directly from code, for example to send the debug view to the console:</span></span>

<!--
        Console.WriteLine(context.ChangeTracker.DebugView.ShortView);
-->
[!code-csharp[Change_tracker_debug_view_1b](../../../samples/core/ChangeTracking/ChangeTrackerDebugging/Samples.cs?name=Change_tracker_debug_view_1b)]

<span data-ttu-id="21986-115">"调试" 视图具有缩写形式和长格式。</span><span class="sxs-lookup"><span data-stu-id="21986-115">The debug view has a short form and a long form.</span></span> <span data-ttu-id="21986-116">缩写形式显示跟踪的实体、其状态和键值。</span><span class="sxs-lookup"><span data-stu-id="21986-116">The short form shows tracked entities, their state, and key values.</span></span> <span data-ttu-id="21986-117">长格式还包括所有属性和导航值以及状态。</span><span class="sxs-lookup"><span data-stu-id="21986-117">The long form also includes all property and navigation values and state.</span></span>

### <a name="the-short-view"></a><span data-ttu-id="21986-118">简短视图</span><span class="sxs-lookup"><span data-stu-id="21986-118">The short view</span></span>

<span data-ttu-id="21986-119">让我们看一下使用本文档末尾显示的模型的调试视图示例。</span><span class="sxs-lookup"><span data-stu-id="21986-119">Let's look at a debug view example using the model shown at the end of this document.</span></span> <span data-ttu-id="21986-120">首先，我们将跟踪某些实体并将其放在一些不同的状态中，这样就可以查看更好的更改跟踪数据：</span><span class="sxs-lookup"><span data-stu-id="21986-120">First, we will track some entities and put them in some different states, just so we have good change tracking data to view:</span></span>

<!--
        using var context = new BlogsContext();

        var blogs = context.Blogs
            .Include(e => e.Posts).ThenInclude(e => e.Tags)
            .Include(e => e.Assets)
            .ToList();

        // Mark something Added
        blogs[0].Posts.Add(
            new Post
            {
                Title = "What’s next for System.Text.Json?",
                Content = ".NET 5.0 was released recently and has come with many new features and..."
            });

        // Mark something Deleted
        blogs[1].Posts.Remove(blogs[1].Posts[1]);

        // Make something Modified
        blogs[0].Name = ".NET Blog (All new!)";

        context.ChangeTracker.DetectChanges();
-->
[!code-csharp[Change_tracker_debug_view_1a](../../../samples/core/ChangeTracking/ChangeTrackerDebugging/Samples.cs?name=Change_tracker_debug_view_1a)]

<span data-ttu-id="21986-121">此时要打印简短视图，如上所述，会生成以下输出：</span><span class="sxs-lookup"><span data-stu-id="21986-121">Printing the short view at this point, as shown above, results in the following output:</span></span>

```output
Blog {Id: 1} Modified AK {AssetsId: ed727978-1ffe-4709-baee-73913e8e44a0}
Blog {Id: 2} Unchanged AK {AssetsId: 3a54b880-2b9d-486b-9403-dc2e52d36d65}
BlogAssets {Id: 3a54b880-2b9d-486b-9403-dc2e52d36d65} Unchanged FK {Id: 3a54b880-2b9d-486b-9403-dc2e52d36d65}
BlogAssets {Id: ed727978-1ffe-4709-baee-73913e8e44a0} Unchanged FK {Id: ed727978-1ffe-4709-baee-73913e8e44a0}
Post {Id: -2147482643} Added FK {BlogId: 1}
Post {Id: 1} Unchanged FK {BlogId: 1}
Post {Id: 2} Unchanged FK {BlogId: 1}
Post {Id: 3} Unchanged FK {BlogId: 2}
Post {Id: 4} Deleted FK {BlogId: 2}
PostTag (Dictionary<string, object>) {PostsId: 1, TagsId: 1} Unchanged FK {PostsId: 1} FK {TagsId: 1}
PostTag (Dictionary<string, object>) {PostsId: 1, TagsId: 3} Unchanged FK {PostsId: 1} FK {TagsId: 3}
PostTag (Dictionary<string, object>) {PostsId: 2, TagsId: 1} Unchanged FK {PostsId: 2} FK {TagsId: 1}
PostTag (Dictionary<string, object>) {PostsId: 3, TagsId: 2} Unchanged FK {PostsId: 3} FK {TagsId: 2}
PostTag (Dictionary<string, object>) {PostsId: 4, TagsId: 2} Deleted FK {PostsId: 4} FK {TagsId: 2}
Tag {Id: 1} Unchanged
Tag {Id: 2} Unchanged
Tag {Id: 3} Unchanged
```

<span data-ttu-id="21986-122">注意：</span><span class="sxs-lookup"><span data-stu-id="21986-122">Notice:</span></span>

- <span data-ttu-id="21986-123">列出的每个实体及其主键 (PK) 值。</span><span class="sxs-lookup"><span data-stu-id="21986-123">Each tracked entity is listed with its primary key (PK) value.</span></span> <span data-ttu-id="21986-124">例如，`Blog {Id: 1}`。</span><span class="sxs-lookup"><span data-stu-id="21986-124">For example, `Blog {Id: 1}`.</span></span>
- <span data-ttu-id="21986-125">如果实体为共享类型实体类型，则也会显示 CLR 类型。</span><span class="sxs-lookup"><span data-stu-id="21986-125">If the entity is a shared-type entity type, then it's CLR type is also shown.</span></span> <span data-ttu-id="21986-126">例如，`PostTag (Dictionary<string, object>)`。</span><span class="sxs-lookup"><span data-stu-id="21986-126">For example, `PostTag (Dictionary<string, object>)`.</span></span>
- <span data-ttu-id="21986-127"><xref:Microsoft.EntityFrameworkCore.EntityState>下图显示了。</span><span class="sxs-lookup"><span data-stu-id="21986-127">The <xref:Microsoft.EntityFrameworkCore.EntityState> is shown next.</span></span> <span data-ttu-id="21986-128">这将是 `Unchanged` 、 `Added` 、或之一 `Modified` `Deleted` 。</span><span class="sxs-lookup"><span data-stu-id="21986-128">This will be one of `Unchanged`, `Added`, `Modified`, or `Deleted`.</span></span>
- <span data-ttu-id="21986-129">接下来显示 (AKs) 的任何备用键的值。</span><span class="sxs-lookup"><span data-stu-id="21986-129">Values for any alternate keys (AKs) are shown next.</span></span> <span data-ttu-id="21986-130">例如，`AK {AssetsId: ed727978-1ffe-4709-baee-73913e8e44a0}`。</span><span class="sxs-lookup"><span data-stu-id="21986-130">For example, `AK {AssetsId: ed727978-1ffe-4709-baee-73913e8e44a0}`.</span></span>
- <span data-ttu-id="21986-131">最后，将显示 (Fk) 的任何外键的值。</span><span class="sxs-lookup"><span data-stu-id="21986-131">Finally, values for any foreign keys (FKs) are shown.</span></span> <span data-ttu-id="21986-132">例如，`FK {PostsId: 4} FK {TagsId: 2}`。</span><span class="sxs-lookup"><span data-stu-id="21986-132">For example, `FK {PostsId: 4} FK {TagsId: 2}`.</span></span>

### <a name="the-long-view"></a><span data-ttu-id="21986-133">长视图</span><span class="sxs-lookup"><span data-stu-id="21986-133">The long view</span></span>

<span data-ttu-id="21986-134">可以通过与简短视图相同的方式向控制台发送长视图：</span><span class="sxs-lookup"><span data-stu-id="21986-134">The long view can be sent to the console in the same way as the short view:</span></span>

<!--
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Change_tracker_debug_view_1c](../../../samples/core/ChangeTracking/ChangeTrackerDebugging/Samples.cs?name=Change_tracker_debug_view_1c)]

<span data-ttu-id="21986-135">与上面的 short 视图相同的状态的输出为：</span><span class="sxs-lookup"><span data-stu-id="21986-135">The output for the same state as the short view above is:</span></span>

```output
Blog {Id: 1} Modified
  Id: 1 PK
  AssetsId: 'ed727978-1ffe-4709-baee-73913e8e44a0' AK
  Name: '.NET Blog (All new!)' Modified Originally '.NET Blog'
  Assets: {Id: ed727978-1ffe-4709-baee-73913e8e44a0}
  Posts: [{Id: 1}, {Id: 2}, {Id: -2147482643}]
Blog {Id: 2} Unchanged
  Id: 2 PK
  AssetsId: '3a54b880-2b9d-486b-9403-dc2e52d36d65' AK
  Name: 'Visual Studio Blog'
  Assets: {Id: 3a54b880-2b9d-486b-9403-dc2e52d36d65}
  Posts: [{Id: 3}]
BlogAssets {Id: 3a54b880-2b9d-486b-9403-dc2e52d36d65} Unchanged
  Id: '3a54b880-2b9d-486b-9403-dc2e52d36d65' PK FK
  Banner: <null>
  Blog: {Id: 2}
BlogAssets {Id: ed727978-1ffe-4709-baee-73913e8e44a0} Unchanged
  Id: 'ed727978-1ffe-4709-baee-73913e8e44a0' PK FK
  Banner: <null>
  Blog: {Id: 1}
Post {Id: -2147482643} Added
  Id: -2147482643 PK Temporary
  BlogId: 1 FK
  Content: '.NET 5.0 was released recently and has come with many new fe...'
  Title: 'What's next for System.Text.Json?'
  Blog: {Id: 1}
  Tags: []
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
  Tags: [{Id: 1}, {Id: 3}]
Post {Id: 2} Unchanged
  Id: 2 PK
  BlogId: 1 FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: {Id: 1}
  Tags: [{Id: 1}]
Post {Id: 3} Unchanged
  Id: 3 PK
  BlogId: 2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: {Id: 2}
  Tags: [{Id: 2}]
Post {Id: 4} Deleted
  Id: 4 PK
  BlogId: 2 FK
  Content: 'Examine when database queries were executed and measure how ...'
  Title: 'Database Profiling with Visual Studio'
  Blog: <null>
  Tags: [{Id: 2}]
PostTag (Dictionary<string, object>) {PostsId: 1, TagsId: 1} Unchanged
  PostsId: 1 PK FK
  TagsId: 1 PK FK
PostTag (Dictionary<string, object>) {PostsId: 1, TagsId: 3} Unchanged
  PostsId: 1 PK FK
  TagsId: 3 PK FK
PostTag (Dictionary<string, object>) {PostsId: 2, TagsId: 1} Unchanged
  PostsId: 2 PK FK
  TagsId: 1 PK FK
PostTag (Dictionary<string, object>) {PostsId: 3, TagsId: 2} Unchanged
  PostsId: 3 PK FK
  TagsId: 2 PK FK
PostTag (Dictionary<string, object>) {PostsId: 4, TagsId: 2} Deleted
  PostsId: 4 PK FK
  TagsId: 2 PK FK
Tag {Id: 1} Unchanged
  Id: 1 PK
  Text: '.NET'
  Posts: [{Id: 1}, {Id: 2}]
Tag {Id: 2} Unchanged
  Id: 2 PK
  Text: 'Visual Studio'
  Posts: [{Id: 3}, {Id: 4}]
Tag {Id: 3} Unchanged
  Id: 3 PK
  Text: 'EF Core'
  Posts: [{Id: 1}]
```

<span data-ttu-id="21986-136">每个跟踪的实体及其状态显示为 "之前"。</span><span class="sxs-lookup"><span data-stu-id="21986-136">Each tracked entity and its state is shown as before.</span></span> <span data-ttu-id="21986-137">不过，长视图还显示属性和导航值。</span><span class="sxs-lookup"><span data-stu-id="21986-137">However, the long view also shows property and navigation values.</span></span>

#### <a name="property-values"></a><span data-ttu-id="21986-138">属性值</span><span class="sxs-lookup"><span data-stu-id="21986-138">Property values</span></span>

<span data-ttu-id="21986-139">对于每个属性，长视图都显示属性是否是主键 (PK) 、备用键 (AK) 或外键 (FK) 的一部分。</span><span class="sxs-lookup"><span data-stu-id="21986-139">For each property, the long view shows whether or not the property is part of a primary key (PK), alternate key (AK), or foreign key (FK).</span></span> <span data-ttu-id="21986-140">例如：</span><span class="sxs-lookup"><span data-stu-id="21986-140">For example:</span></span>

- <span data-ttu-id="21986-141">`Blog.Id` 为主键属性： `Id: 1 PK`</span><span class="sxs-lookup"><span data-stu-id="21986-141">`Blog.Id` is a primary key property: `Id: 1 PK`</span></span>
- <span data-ttu-id="21986-142">`Blog.AssetsId` 是备用键属性： `AssetsId: 'ed727978-1ffe-4709-baee-73913e8e44a0' AK`</span><span class="sxs-lookup"><span data-stu-id="21986-142">`Blog.AssetsId` is an alternate key property: `AssetsId: 'ed727978-1ffe-4709-baee-73913e8e44a0' AK`</span></span>
- <span data-ttu-id="21986-143">`Post.BlogId` 是外键属性： `BlogId: 2 FK`</span><span class="sxs-lookup"><span data-stu-id="21986-143">`Post.BlogId` is a foreign key property: `BlogId: 2 FK`</span></span>
- <span data-ttu-id="21986-144">`BlogAssets.Id` 为主键和外键属性： `Id: '3a54b880-2b9d-486b-9403-dc2e52d36d65' PK FK`</span><span class="sxs-lookup"><span data-stu-id="21986-144">`BlogAssets.Id` is both a primary key and a foreign key property: `Id: '3a54b880-2b9d-486b-9403-dc2e52d36d65' PK FK`</span></span>

<span data-ttu-id="21986-145">已修改的属性值将被标记为这样，同时还会显示该属性的原始值。</span><span class="sxs-lookup"><span data-stu-id="21986-145">Property values that have been modified are marked as such, and the original value of the property is also shown.</span></span> <span data-ttu-id="21986-146">例如，`Name: '.NET Blog (All new!)' Modified Originally '.NET Blog'`。</span><span class="sxs-lookup"><span data-stu-id="21986-146">For example, `Name: '.NET Blog (All new!)' Modified Originally '.NET Blog'`.</span></span>

<span data-ttu-id="21986-147">最后， `Added` 具有临时键值的实体指示值是临时的。</span><span class="sxs-lookup"><span data-stu-id="21986-147">Finally, `Added` entities with temporary key values indicate that the value is temporary.</span></span> <span data-ttu-id="21986-148">例如，`Id: -2147482643 PK Temporary`。</span><span class="sxs-lookup"><span data-stu-id="21986-148">For example, `Id: -2147482643 PK Temporary`.</span></span>

#### <a name="navigation-values"></a><span data-ttu-id="21986-149">导航值</span><span class="sxs-lookup"><span data-stu-id="21986-149">Navigation values</span></span>

<span data-ttu-id="21986-150">使用导航引用的实体的主键值显示导航值。</span><span class="sxs-lookup"><span data-stu-id="21986-150">Navigation values are displayed using the primary key values of the entities that the navigations reference.</span></span> <span data-ttu-id="21986-151">例如，在上面的输出中，post 3 与博客2相关。</span><span class="sxs-lookup"><span data-stu-id="21986-151">For example, in the output above, post 3 is related to blog 2.</span></span> <span data-ttu-id="21986-152">这意味着， `Post.Blog` 导航指向 `Blog` ID 为2的实例。</span><span class="sxs-lookup"><span data-stu-id="21986-152">This means that the `Post.Blog` navigation points to the `Blog` instance with ID 2.</span></span> <span data-ttu-id="21986-153">这显示为 `Blog: {Id: 2}` 。</span><span class="sxs-lookup"><span data-stu-id="21986-153">This is shown as `Blog: {Id: 2}`.</span></span>

<span data-ttu-id="21986-154">对于集合导航也是如此，只是在这种情况下，可以有多个相关实体。</span><span class="sxs-lookup"><span data-stu-id="21986-154">The same thing happens for collection navigations, except that in this case there can be multiple related entities.</span></span> <span data-ttu-id="21986-155">例如，集合导航 `Blog.Posts` 包含三个实体，键值分别为1、2和-2147482643。</span><span class="sxs-lookup"><span data-stu-id="21986-155">For example, the collection navigation `Blog.Posts` contains three entities, with key values 1, 2, and -2147482643 respectively.</span></span> <span data-ttu-id="21986-156">这显示为 `[{Id: 1}, {Id: 2}, {Id: -2147482643}]` 。</span><span class="sxs-lookup"><span data-stu-id="21986-156">This is shown as `[{Id: 1}, {Id: 2}, {Id: -2147482643}]`.</span></span>

## <a name="change-tracker-logging"></a><span data-ttu-id="21986-157">更改跟踪器日志记录</span><span class="sxs-lookup"><span data-stu-id="21986-157">Change tracker logging</span></span>

<span data-ttu-id="21986-158">更改跟踪器 `Debug` <xref:Microsoft.Extensions.Logging.LogLevel> 会在 [检测到属性或导航更改](xref:core/change-tracking/debug-views)时记录消息。</span><span class="sxs-lookup"><span data-stu-id="21986-158">The change tracker logs messages at the `Debug` <xref:Microsoft.Extensions.Logging.LogLevel> whenever it [detects property or navigation changes](xref:core/change-tracking/debug-views).</span></span> <span data-ttu-id="21986-159">例如， <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> 在本文档顶部的代码中调用时，如果 [启用了调试日志记录](xref:core/logging-events-diagnostics/index)，则会生成以下日志：</span><span class="sxs-lookup"><span data-stu-id="21986-159">For example, when <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> is called in the code at the top of this document and [debug logging is enabled](xref:core/logging-events-diagnostics/index), then the following logs are generated:</span></span>

```output
dbug: 12/30/2020 13:52:44.815 CoreEventId.DetectChangesStarting[10800] (Microsoft.EntityFrameworkCore.ChangeTracking)
      DetectChanges starting for 'BlogsContext'.
dbug: 12/30/2020 13:52:44.818 CoreEventId.PropertyChangeDetected[10802] (Microsoft.EntityFrameworkCore.ChangeTracking)
      The unchanged property 'Blog.Name' was detected as changed from '.NET Blog' to '.NET Blog (All new!)' and will be marked as modified for entity with key '{Id: 1}'.
dbug: 12/30/2020 13:52:44.820 CoreEventId.StateChanged[10807] (Microsoft.EntityFrameworkCore.ChangeTracking)
      The 'Blog' entity with key '{Id: 1}' tracked by 'BlogsContext' changed state from 'Unchanged' to 'Modified'.
dbug: 12/30/2020 13:52:44.821 CoreEventId.CollectionChangeDetected[10804] (Microsoft.EntityFrameworkCore.ChangeTracking)
      1 entities were added and 0 entities were removed from navigation 'Blog.Posts' on entity with key '{Id: 1}'.
dbug: 12/30/2020 13:52:44.822 CoreEventId.ValueGenerated[10808] (Microsoft.EntityFrameworkCore.ChangeTracking)
      'BlogsContext' generated temporary value '-2147482638' for the property 'Id.Post'.
dbug: 12/30/2020 13:52:44.822 CoreEventId.StartedTracking[10806] (Microsoft.EntityFrameworkCore.ChangeTracking)
      Context 'BlogsContext' started tracking 'Post' entity with key '{Id: -2147482638}'.
dbug: 12/30/2020 13:52:44.827 CoreEventId.CollectionChangeDetected[10804] (Microsoft.EntityFrameworkCore.ChangeTracking)
      0 entities were added and 1 entities were removed from navigation 'Blog.Posts' on entity with key '{Id: 2}'.
dbug: 12/30/2020 13:52:44.827 CoreEventId.StateChanged[10807] (Microsoft.EntityFrameworkCore.ChangeTracking)
      The 'Post' entity with key '{Id: 4}' tracked by 'BlogsContext' changed state from 'Unchanged' to 'Modified'.
dbug: 12/30/2020 13:52:44.829 CoreEventId.CascadeDeleteOrphan[10003] (Microsoft.EntityFrameworkCore.Update)
      An entity of type 'Post' with key '{Id: 4}' changed to 'Deleted' state due to severed required relationship to its parent entity of type 'Blog'.
dbug: 12/30/2020 13:52:44.829 CoreEventId.StateChanged[10807] (Microsoft.EntityFrameworkCore.ChangeTracking)
      The 'Post' entity with key '{Id: 4}' tracked by 'BlogsContext' changed state from 'Modified' to 'Deleted'.
dbug: 12/30/2020 13:52:44.829 CoreEventId.CollectionChangeDetected[10804] (Microsoft.EntityFrameworkCore.ChangeTracking)
      0 entities were added and 1 entities were removed from navigation 'Blog.Posts' on entity with key '{Id: 2}'.
dbug: 12/30/2020 13:52:44.831 CoreEventId.CascadeDelete[10002] (Microsoft.EntityFrameworkCore.Update)
      A cascade state change of an entity of type 'PostTag' with key '{PostsId: 4, TagsId: 2}' to 'Deleted' occurred due to the deletion of its parent entity of type 'Post' with key '{Id: 4}'.
dbug: 12/30/2020 13:52:44.831 CoreEventId.StateChanged[10807] (Microsoft.EntityFrameworkCore.ChangeTracking)
      The 'PostTag' entity with key '{PostsId: 4, TagsId: 2}' tracked by 'BlogsContext' changed state from 'Unchanged' to 'Deleted'.
dbug: 12/30/2020 13:52:44.831 CoreEventId.DetectChangesCompleted[10801] (Microsoft.EntityFrameworkCore.ChangeTracking)
      DetectChanges completed for 'BlogsContext'.
```

<span data-ttu-id="21986-160">下表总结了更改跟踪器日志记录消息：</span><span class="sxs-lookup"><span data-stu-id="21986-160">The following table summaries the change tracker logging messages:</span></span>

| <span data-ttu-id="21986-161">事件 ID</span><span class="sxs-lookup"><span data-stu-id="21986-161">Event ID</span></span>                                                                                                               | <span data-ttu-id="21986-162">描述</span><span class="sxs-lookup"><span data-stu-id="21986-162">Description</span></span>
|:-----------------------------------------------------------------------------------------------------------------------|----
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId.DetectChangesStarting?displayProperty=nameWithType>        | <span data-ttu-id="21986-163"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges> 正在启动</span><span class="sxs-lookup"><span data-stu-id="21986-163"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges> is starting</span></span>
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId.DetectChangesCompleted?displayProperty=nameWithType>       | <span data-ttu-id="21986-164"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges> 已完成</span><span class="sxs-lookup"><span data-stu-id="21986-164"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges> has completed</span></span>
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId.PropertyChangeDetected?displayProperty=nameWithType>       | <span data-ttu-id="21986-165">正常属性值已更改</span><span class="sxs-lookup"><span data-stu-id="21986-165">A normal property value has changed</span></span>
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId.ForeignKeyChangeDetected?displayProperty=nameWithType>     | <span data-ttu-id="21986-166">外键属性值已更改</span><span class="sxs-lookup"><span data-stu-id="21986-166">A foreign key property value has changed</span></span>
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId.CollectionChangeDetected?displayProperty=nameWithType>     | <span data-ttu-id="21986-167">非跳过集合导航已添加或移除相关实体。</span><span class="sxs-lookup"><span data-stu-id="21986-167">A non-skip collection navigation has had related entities added or removed.</span></span>
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId.ReferenceChangeDetected?displayProperty=nameWithType>      | <span data-ttu-id="21986-168">已将引用导航更改为指向另一个实体，或将其设置为 null</span><span class="sxs-lookup"><span data-stu-id="21986-168">A reference navigation has been changed to point to another entity, or set to null</span></span>
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId.StartedTracking?displayProperty=nameWithType>              | <span data-ttu-id="21986-169">EF Core 开始跟踪实体</span><span class="sxs-lookup"><span data-stu-id="21986-169">EF Core started tracking an entity</span></span>
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId.StateChanged?displayProperty=nameWithType>                 | <span data-ttu-id="21986-170"><xref:Microsoft.EntityFrameworkCore.EntityState>实体的已更改</span><span class="sxs-lookup"><span data-stu-id="21986-170">The <xref:Microsoft.EntityFrameworkCore.EntityState> of an entity has changed</span></span>
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId.ValueGenerated?displayProperty=nameWithType>               | <span data-ttu-id="21986-171">为属性生成了值</span><span class="sxs-lookup"><span data-stu-id="21986-171">A value was generated for a property</span></span>
| <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId.SkipCollectionChangeDetected?displayProperty=nameWithType> | <span data-ttu-id="21986-172">已添加或删除了 "跳过集合" 导航</span><span class="sxs-lookup"><span data-stu-id="21986-172">A skip collection navigation has had related entities added or removed</span></span>

## <a name="the-model"></a><span data-ttu-id="21986-173">模型</span><span class="sxs-lookup"><span data-stu-id="21986-173">The model</span></span>

<span data-ttu-id="21986-174">以上示例所使用的模型包含以下实体类型：</span><span class="sxs-lookup"><span data-stu-id="21986-174">The model used for the examples above contains the following entity types:</span></span>

<!--
public class Blog
{
    public int Id { get; set; } // Primary key
    public Guid AssetsId { get; set; } // Alternate key
    public string Name { get; set; }

    public IList<Post> Posts { get; } = new List<Post>(); // Collection navigation
    public BlogAssets Assets { get; set; } // Reference navigation
}

public class BlogAssets
{
    public Guid Id { get; set; } // Primary key and foreign key
    public byte[] Banner { get; set; }

    public Blog Blog { get; set; } // Reference navigation
}

public class Post
{
    public int Id { get; set; } // Primary key
    public string Title { get; set; }
    public string Content { get; set; }

    public int BlogId { get; set; } // Foreign key
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
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangeTrackerDebugging/Samples.cs?name=Model)]

<span data-ttu-id="21986-175">模型大多按惯例配置，只需几行 OnModelCreating：</span><span class="sxs-lookup"><span data-stu-id="21986-175">The model is mostly configured by convention, with just a few lines in OnModelCreating:</span></span>

<!--
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder
            .Entity<Blog>()
            .Property(e => e.AssetsId)
            .ValueGeneratedOnAdd();

        modelBuilder
            .Entity<BlogAssets>()
            .HasOne(e => e.Blog)
            .WithOne(e => e.Assets)
            .HasForeignKey<BlogAssets>(e => e.Id)
            .HasPrincipalKey<Blog>(e => e.AssetsId);
    }
-->
[!code-csharp[OnModelCreating](../../../samples/core/ChangeTracking/ChangeTrackerDebugging/Samples.cs?name=OnModelCreating)]
