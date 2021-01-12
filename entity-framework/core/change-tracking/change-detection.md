---
title: 更改检测和通知-EF Core
description: 使用 DetectChanges 或通知检测属性和关系更改
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/change-detection
ms.openlocfilehash: 39dc66a3ba74be89d3e470cfe788a357401965d1
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129578"
---
# <a name="change-detection-and-notifications"></a><span data-ttu-id="ef79a-103">更改检测和通知</span><span class="sxs-lookup"><span data-stu-id="ef79a-103">Change Detection and Notifications</span></span>

<span data-ttu-id="ef79a-104">每个 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例跟踪对实体所做的更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-104">Each <xref:Microsoft.EntityFrameworkCore.DbContext> instance tracks changes made to entities.</span></span> <span data-ttu-id="ef79a-105">在调用时，这些跟踪的实体会驱动对数据库的更改 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。</span><span class="sxs-lookup"><span data-stu-id="ef79a-105">These tracked entities in turn drive the changes to the database when <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> is called.</span></span> <span data-ttu-id="ef79a-106">[EF Core 的更改跟踪](xref:core/change-tracking/index)中介绍了这一点，并且本文档假设了解 Entity Framework Core (EF Core) 更改跟踪的实体状态和基础知识。</span><span class="sxs-lookup"><span data-stu-id="ef79a-106">This is covered in [Change Tracking in EF Core](xref:core/change-tracking/index), and this document assumes that entity states and the basics of Entity Framework Core (EF Core) change tracking are understood.</span></span>

<span data-ttu-id="ef79a-107">跟踪属性和关系更改要求 DbContext 能够检测到这些更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-107">Tracking property and relationship changes requires that the DbContext is able to detect these changes.</span></span> <span data-ttu-id="ef79a-108">本文档介绍了如何进行这种检测，以及如何使用属性通知或更改跟踪代理强制立即检测更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-108">This document covers how this detection happens, as well as how to use property notifications or change-tracking proxies to force immediate detection of changes.</span></span>

> [!TIP]
> <span data-ttu-id="ef79a-109">通过 [从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangeDetectionAndNotifications)，你可以运行并调试到本文档中的所有代码。</span><span class="sxs-lookup"><span data-stu-id="ef79a-109">You can run and debug into all the code in this document by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangeDetectionAndNotifications).</span></span>

## <a name="snapshot-change-tracking"></a><span data-ttu-id="ef79a-110">快照更改跟踪</span><span class="sxs-lookup"><span data-stu-id="ef79a-110">Snapshot change tracking</span></span>

<span data-ttu-id="ef79a-111">默认情况下，在 DbContext 实例首次跟踪每个实体的属性值时，EF Core 会创建其快照。</span><span class="sxs-lookup"><span data-stu-id="ef79a-111">By default, EF Core creates a snapshot of every entity's property values when it is first tracked by a DbContext instance.</span></span> <span data-ttu-id="ef79a-112">然后，将存储在此快照中的值与实体的当前值进行比较，以确定更改了哪些属性值。</span><span class="sxs-lookup"><span data-stu-id="ef79a-112">The values stored in this snapshot are then compared against the current values of the entity in order to determine which property values have changed.</span></span>

<span data-ttu-id="ef79a-113">在调用 SaveChanges 时，会发生更改，以确保在将更新发送到数据库之前检测到所有更改的值。</span><span class="sxs-lookup"><span data-stu-id="ef79a-113">This detection of changes happens when SaveChanges is called to ensure all changed values are detected before sending updates to the database.</span></span> <span data-ttu-id="ef79a-114">不过，还会在其他时间进行更改检测，以确保应用程序能够使用最新的跟踪信息。</span><span class="sxs-lookup"><span data-stu-id="ef79a-114">However, the detection of changes also happens at other times to ensure the application is working with up-to-date tracking information.</span></span> <span data-ttu-id="ef79a-115">可以通过调用随时强制执行更改检测 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="ef79a-115">Detection of changes can be forced at any time by calling <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType>.</span></span>

### <a name="when-change-detection-is-needed"></a><span data-ttu-id="ef79a-116">需要更改检测时</span><span class="sxs-lookup"><span data-stu-id="ef79a-116">When change detection is needed</span></span>

<span data-ttu-id="ef79a-117">如果在 _不使用 EF Core 进行此更改的情况下_ 更改了属性或导航，则需要检测更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-117">Detection of changes is needed when a property or navigation has been changed _without using EF Core to make this change_.</span></span> <span data-ttu-id="ef79a-118">例如，考虑加载博客和文章，然后对这些实体进行更改：</span><span class="sxs-lookup"><span data-stu-id="ef79a-118">For example, consider loading blogs and posts and then making changes to these entities:</span></span>

<!--
        using var context = new BlogsContext();
        var blog = context.Blogs.Include(e => e.Posts).First(e => e.Name == ".NET Blog");

        // Change a property value
        blog.Name = ".NET Blog (Updated!)";

        // Add a new entity to a navigation
        blog.Posts.Add(new Post
        {
            Title = "What’s next for System.Text.Json?",
            Content = ".NET 5.0 was released recently and has come with many..."
        });

        Console.WriteLine(context.ChangeTracker.DebugView.LongView);
        context.ChangeTracker.DetectChanges();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Snapshot_change_tracking_1](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/SnapshotSamples.cs?name=Snapshot_change_tracking_1)]

<span data-ttu-id="ef79a-119">在调用之前查看 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图会 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> 显示未检测到所做的更改，因此不会在实体状态和修改后的属性数据中反映这些更改：</span><span class="sxs-lookup"><span data-stu-id="ef79a-119">Looking at the [change tracker debug view](xref:core/change-tracking/debug-views) before calling <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> shows that the changes made have not been detected and hence are not reflected in the entity states and modified property data:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog (Updated!)' Originally '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}, <not found>]
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
Post {Id: 2} Unchanged
  Id: 2 PK
  BlogId: 1 FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: {Id: 1}
```

<span data-ttu-id="ef79a-120">具体而言，博客条目的状态仍为 `Unchanged` ，而新的帖子不会显示为跟踪的实体。</span><span class="sxs-lookup"><span data-stu-id="ef79a-120">Specifically, the state of the blog entry is still `Unchanged`, and the new post does not appear as a tracked entity.</span></span> <span data-ttu-id="ef79a-121"> (敏锐将会看到 "属性" 报告其新值，即使 EF Core 尚未检测到这些更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-121">(The astute will notice properties report their new values, even though these changes have not yet been detected by EF Core.</span></span> <span data-ttu-id="ef79a-122">这是因为 "调试" 视图直接从实体实例读取当前值。 ) </span><span class="sxs-lookup"><span data-stu-id="ef79a-122">This is because the debug view is reading current values directly from the entity instance.)</span></span>

<span data-ttu-id="ef79a-123">在调用 DetectChanges 后，将此与调试视图进行比较：</span><span class="sxs-lookup"><span data-stu-id="ef79a-123">Contrast this with the debug view after calling DetectChanges:</span></span>

```output
Blog {Id: 1} Modified
  Id: 1 PK
  Name: '.NET Blog (Updated!)' Modified Originally '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}, {Id: -2147482643}]
Post {Id: -2147482643} Added
  Id: -2147482643 PK Temporary
  BlogId: 1 FK
  Content: '.NET 5.0 was released recently and has come with many...'
  Title: 'What's next for System.Text.Json?'
  Blog: {Id: 1}
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
Post {Id: 2} Unchanged
  Id: 2 PK
  BlogId: 1 FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: {Id: 1}
```

<span data-ttu-id="ef79a-124">现在，博客已正确标记为 `Modified` ，并已检测到新文章，并将其作为跟踪 `Added` 。</span><span class="sxs-lookup"><span data-stu-id="ef79a-124">Now the blog is correctly marked as `Modified` and the new post has been detected and is tracked as `Added`.</span></span>

<span data-ttu-id="ef79a-125">在本部分开始时，我们指出在不使用 _EF Core 进行更改_ 时需要检测更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-125">At the start of this section we stated that detecting changes is needed when not using _using EF Core to make the change_.</span></span> <span data-ttu-id="ef79a-126">这就是上述代码所发生的情况。</span><span class="sxs-lookup"><span data-stu-id="ef79a-126">This is what is happening in the code above.</span></span> <span data-ttu-id="ef79a-127">也就是说，对属性和导航的更改 _直接对实体实例_ 进行，而不是使用任何 EF Core 方法。</span><span class="sxs-lookup"><span data-stu-id="ef79a-127">That is, the changes to the property and navigation are made _directly on the entity instances_, and not by using any EF Core methods.</span></span>

<span data-ttu-id="ef79a-128">将此与以下代码进行比较，以相同方式修改实体，但这次使用 EF Core 方法：</span><span class="sxs-lookup"><span data-stu-id="ef79a-128">Contrast this to the following code which modifies the entities in the same way, but this time using EF Core methods:</span></span>

<!--
        using var context = new BlogsContext();
        var blog = context.Blogs.Include(e => e.Posts).First(e => e.Name == ".NET Blog");

        // Change a property value
        context.Entry(blog).Property(e => e.Name).CurrentValue = ".NET Blog (Updated!)";

        // Add a new entity to the DbContext
        context.Add(
            new Post
            {
                Blog = blog,
                Title = "What’s next for System.Text.Json?",
                Content = ".NET 5.0 was released recently and has come with many..."
            });

        Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Snapshot_change_tracking_2](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/SnapshotSamples.cs?name=Snapshot_change_tracking_2)]

<span data-ttu-id="ef79a-129">在这种情况下，更改跟踪器的 "调试" 视图会显示所有实体状态和属性修改都是已知的，即使未发生更改检测也是如此。</span><span class="sxs-lookup"><span data-stu-id="ef79a-129">In this case the change tracker debug view shows that all entity states and property modifications are known, even though detection of changes has not happened.</span></span> <span data-ttu-id="ef79a-130">这是因为 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.CurrentValue?displayProperty=nameWithType> 是 EF Core 方法，这意味着 EF Core 立即知道此方法所做的更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-130">This is because <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.CurrentValue?displayProperty=nameWithType> is an EF Core method, which means that EF Core immediately knows about the change made by this method.</span></span> <span data-ttu-id="ef79a-131">同样，通过调用， <xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType> EF Core 立即了解新实体并进行相应跟踪。</span><span class="sxs-lookup"><span data-stu-id="ef79a-131">Likewise, calling <xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType> allows EF Core to immediately know about the new entity and track it appropriately.</span></span>

> [!TIP]
> <span data-ttu-id="ef79a-132">不要尝试通过始终使用 EF Core 方法进行实体更改来避免检测更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-132">Don't attempt to avoid detecting changes by always using EF Core methods to make entity changes.</span></span> <span data-ttu-id="ef79a-133">这样做通常更繁琐，而且执行方式不如在正常情况中对实体进行更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-133">Doing so is often more cumbersome and performs less well than making changes to entities in the normal way.</span></span> <span data-ttu-id="ef79a-134">本文档的目的是在需要检测更改时通知，如果不需要，则通知。</span><span class="sxs-lookup"><span data-stu-id="ef79a-134">The intention of this document is to inform as to when detecting changes is needed and when it is not.</span></span> <span data-ttu-id="ef79a-135">目的不是鼓励避免更改检测。</span><span class="sxs-lookup"><span data-stu-id="ef79a-135">The intention is not to encourage avoidance of change detection.</span></span>

### <a name="methods-that-automatically-detect-changes"></a><span data-ttu-id="ef79a-136">自动检测更改的方法</span><span class="sxs-lookup"><span data-stu-id="ef79a-136">Methods that automatically detect changes</span></span>

<span data-ttu-id="ef79a-137"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges> 方法会自动调用此方法，这可能会影响结果。</span><span class="sxs-lookup"><span data-stu-id="ef79a-137"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges> is called automatically by methods where doing so is likely to impact the results.</span></span> <span data-ttu-id="ef79a-138">这些方法包括：</span><span class="sxs-lookup"><span data-stu-id="ef79a-138">These methods are:</span></span>

- <span data-ttu-id="ef79a-139"><xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A?displayProperty=nameWithType> ，以确保在更新数据库之前检测到所有更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-139"><xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A?displayProperty=nameWithType> and <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A?displayProperty=nameWithType>, to ensure that all changes are detected before updating the database.</span></span>
- <span data-ttu-id="ef79a-140"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%60%601?displayProperty=nameWithType> ，以确保实体状态和修改的属性是最新的。</span><span class="sxs-lookup"><span data-stu-id="ef79a-140"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries?displayProperty=nameWithType> and <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%60%601?displayProperty=nameWithType>, to ensure entity states and modified properties are up-to-date.</span></span>
- <span data-ttu-id="ef79a-141"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.HasChanges?displayProperty=nameWithType>，以确保结果准确无误。</span><span class="sxs-lookup"><span data-stu-id="ef79a-141"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.HasChanges?displayProperty=nameWithType>, to ensure that the result is accurate.</span></span>
- <span data-ttu-id="ef79a-142"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeChanges?displayProperty=nameWithType>，用于在级联之前确保主体/父实体的实体状态正确。</span><span class="sxs-lookup"><span data-stu-id="ef79a-142"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeChanges?displayProperty=nameWithType>, to ensure correct entity states for principal/parent entities before cascading.</span></span>
- <span data-ttu-id="ef79a-143"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType>，以确保跟踪的关系图是最新的。</span><span class="sxs-lookup"><span data-stu-id="ef79a-143"><xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType>, to ensure that the tracked graph is up-to-date.</span></span>

<span data-ttu-id="ef79a-144">在某些位置，只会对单个实体实例进行更改检测，而不是在整个所跟踪实体关系图上发生更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-144">There are also some places where detection of changes happens on only a single entity instance, rather than on the entire graph of tracked entities.</span></span> <span data-ttu-id="ef79a-145">这些位置包括：</span><span class="sxs-lookup"><span data-stu-id="ef79a-145">These places are:</span></span>

- <span data-ttu-id="ef79a-146">使用时 <xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType> ，确保实体的状态和修改的属性是最新的。</span><span class="sxs-lookup"><span data-stu-id="ef79a-146">When using <xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType>, to ensure that the entity's state and modified properties are up-to-date.</span></span>
- <span data-ttu-id="ef79a-147">使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry> 、或等方法 `Property` `Collection` `Reference` `Member` 确保属性修改、当前值等是最新的。</span><span class="sxs-lookup"><span data-stu-id="ef79a-147">When using <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry> methods such as `Property`, `Collection`, `Reference` or `Member` to ensure property modifications, current values, etc. are up-to-date.</span></span>
- <span data-ttu-id="ef79a-148">当要删除依赖/子实体时，因为所需的关系已被断开。</span><span class="sxs-lookup"><span data-stu-id="ef79a-148">When a dependent/child entity is going to be deleted because a required relationship has been severed.</span></span> <span data-ttu-id="ef79a-149">这会检测不应删除实体的时间，因为它已重新获得父级。</span><span class="sxs-lookup"><span data-stu-id="ef79a-149">This detects when an entity should not be deleted because it has been re-parented.</span></span>

<span data-ttu-id="ef79a-150">可以通过调用，显式触发单个实体的更改的本地检测 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.DetectChanges?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="ef79a-150">Local detection of changes for a single entity can be triggered explicitly by calling <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.DetectChanges?displayProperty=nameWithType>.</span></span>

> [!NOTE]
> <span data-ttu-id="ef79a-151">本地检测更改可能会丢失完全检测会发现的一些更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-151">Local detect changes can miss some changes that a full detection would find.</span></span> <span data-ttu-id="ef79a-152">如果因未检测到对其他实体进行的更改而导致的级联操作对相关实体产生影响，则会发生这种情况。</span><span class="sxs-lookup"><span data-stu-id="ef79a-152">This happens when cascading actions resulting from undetected changes to other entities have an impact on the entity in question.</span></span> <span data-ttu-id="ef79a-153">在这种情况下，应用程序可能需要通过显式调用强制对所有实体进行完全扫描 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="ef79a-153">In such situations the application may need to force a full scan of all entities by explicitly calling <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType>.</span></span>

### <a name="disabling-automatic-change-detection"></a><span data-ttu-id="ef79a-154">禁用自动更改检测</span><span class="sxs-lookup"><span data-stu-id="ef79a-154">Disabling automatic change detection</span></span>

<span data-ttu-id="ef79a-155">对于大多数应用程序而言，检测更改的性能并不是瓶颈。</span><span class="sxs-lookup"><span data-stu-id="ef79a-155">The performance of detecting changes is not a bottleneck for most applications.</span></span> <span data-ttu-id="ef79a-156">但对于跟踪上千个实体的某些应用程序而言，检测更改可能会成为性能问题。</span><span class="sxs-lookup"><span data-stu-id="ef79a-156">However, detecting changes can become a performance problem for some applications that track thousands of entities.</span></span> <span data-ttu-id="ef79a-157"> (的确切数字将取决于许多因素，如实体中属性的数目。 ) 为此，可以使用禁用更改的自动检测 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.AutoDetectChangesEnabled?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="ef79a-157">(The exact number will dependent on many things, such as the number of properties in the entity.) For this reason the automatic detection of changes can be disabled using <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.AutoDetectChangesEnabled?displayProperty=nameWithType>.</span></span> <span data-ttu-id="ef79a-158">例如，考虑使用有效负载处理多对多关系中的联接实体：</span><span class="sxs-lookup"><span data-stu-id="ef79a-158">For example, consider processing join entities in a many-to-many relationship with payloads:</span></span>

<!--
        public override int SaveChanges()
        {
            foreach (var entityEntry in ChangeTracker.Entries<PostTag>()) // Detects changes automatically
            {
                if (entityEntry.State == EntityState.Added)
                {
                    entityEntry.Entity.TaggedBy = "ajcvickers";
                    entityEntry.Entity.TaggedOn = DateTime.Now;
                }
            }

            try
            {
                ChangeTracker.AutoDetectChangesEnabled = false;
                return base.SaveChanges(); // Avoid automatically detecting changes again here
            }
            finally
            {
                ChangeTracker.AutoDetectChangesEnabled = true;
            }
        }
-->
[!code-csharp[SaveChanges](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/SnapshotSamples.cs?name=SaveChanges)]

<span data-ttu-id="ef79a-159">正如我们在上一节中所知道的，和都将 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%60%601?displayProperty=nameWithType> <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A?displayProperty=nameWithType> 自动检测更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-159">As we know from the previous section, both <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%60%601?displayProperty=nameWithType> and <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A?displayProperty=nameWithType> automatically detect changes.</span></span> <span data-ttu-id="ef79a-160">但是，在调用项后，代码不会更改任何实体或属性状态。</span><span class="sxs-lookup"><span data-stu-id="ef79a-160">However, after calling Entries, the code does not then make any entity or property state changes.</span></span> <span data-ttu-id="ef79a-161"> (设置添加的实体的普通属性值不会导致任何状态更改。 ) 代码因此，在向下调用基本 SaveChanges 方法时，将禁用不必要的自动更改检测。</span><span class="sxs-lookup"><span data-stu-id="ef79a-161">(Setting normal property values on Added entities does not cause any state changes.) The code therefore disables unnecessary automatic change detection when calling down into the base SaveChanges method.</span></span> <span data-ttu-id="ef79a-162">此代码还利用 try/finally 块来确保即使 SaveChanges 失败，也会还原默认设置。</span><span class="sxs-lookup"><span data-stu-id="ef79a-162">The code also makes use of a try/finally block to ensure that the default setting is restored even if SaveChanges fails.</span></span>

> [!TIP]
> <span data-ttu-id="ef79a-163">不要假设你的代码必须禁用自动更改检测，才能正常运行。</span><span class="sxs-lookup"><span data-stu-id="ef79a-163">Do not assume that your code must disable automatic change detection to to perform well.</span></span> <span data-ttu-id="ef79a-164">这只是在分析应用程序跟踪时需要的，很多实体表示更改检测的性能问题。</span><span class="sxs-lookup"><span data-stu-id="ef79a-164">This is only needed when profiling an application tracking many entities indicates that performance of change detection is an issue.</span></span>

### <a name="detecting-changes-and-value-conversions"></a><span data-ttu-id="ef79a-165">检测更改和值转换</span><span class="sxs-lookup"><span data-stu-id="ef79a-165">Detecting changes and value conversions</span></span>

<span data-ttu-id="ef79a-166">若要对实体类型使用快照更改跟踪，EF Core 必须能够：</span><span class="sxs-lookup"><span data-stu-id="ef79a-166">To use snapshot change tracking with an entity type, EF Core must be able to:</span></span>

- <span data-ttu-id="ef79a-167">跟踪实体时创建每个属性值的快照</span><span class="sxs-lookup"><span data-stu-id="ef79a-167">Make a snapshot of each property value when the entity is tracked</span></span>
- <span data-ttu-id="ef79a-168">将此值与属性的当前值进行比较</span><span class="sxs-lookup"><span data-stu-id="ef79a-168">Compare this value to the current value of the property</span></span>
- <span data-ttu-id="ef79a-169">为值生成哈希代码</span><span class="sxs-lookup"><span data-stu-id="ef79a-169">Generate a hash code for the value</span></span>

<span data-ttu-id="ef79a-170">对于可以直接映射到数据库的类型 EF Core，会自动处理这种情况。</span><span class="sxs-lookup"><span data-stu-id="ef79a-170">This is handled automatically by EF Core for types that can be directly mapped to the database.</span></span> <span data-ttu-id="ef79a-171">但是，在 [使用值转换器来映射属性](xref:core/modeling/value-conversions)时，转换器必须指定如何执行这些操作。</span><span class="sxs-lookup"><span data-stu-id="ef79a-171">However, when a [value converter is used to map a property](xref:core/modeling/value-conversions), then that converter must specify how to perform these actions.</span></span> <span data-ttu-id="ef79a-172">这是通过值比较器实现的，在 [值](xref:core/modeling/value-comparers) 比较器文档中进行了详细说明。</span><span class="sxs-lookup"><span data-stu-id="ef79a-172">This is achieved with a value comparer, and is described in detail in the [Value Comparers](xref:core/modeling/value-comparers) documentation.</span></span>

## <a name="notification-entities"></a><span data-ttu-id="ef79a-173">通知实体</span><span class="sxs-lookup"><span data-stu-id="ef79a-173">Notification entities</span></span>

<span data-ttu-id="ef79a-174">对于大多数应用程序，建议使用快照更改跟踪。</span><span class="sxs-lookup"><span data-stu-id="ef79a-174">Snapshot change tracking is recommended for most applications.</span></span> <span data-ttu-id="ef79a-175">但是，跟踪多个实体和/或对这些实体进行很多更改的应用程序可能会受益于实现实体，这些实体会在属性和导航值改变时自动通知 EF Core。</span><span class="sxs-lookup"><span data-stu-id="ef79a-175">However, applications that track many entities and/or make many changes to those entities may benefit from implementing entities that automatically notify EF Core when their property and navigation values change.</span></span> <span data-ttu-id="ef79a-176">它们称为 "通知实体"。</span><span class="sxs-lookup"><span data-stu-id="ef79a-176">These are known as "notification entities".</span></span>

### <a name="implementing-notification-entities"></a><span data-ttu-id="ef79a-177">实现通知实体</span><span class="sxs-lookup"><span data-stu-id="ef79a-177">Implementing notification entities</span></span>

<span data-ttu-id="ef79a-178">通知实体使用 <xref:System.ComponentModel.INotifyPropertyChanging> 和 <xref:System.ComponentModel.INotifyPropertyChanged> 接口，这些接口是 .net 基类库 (BCL) 的一部分。</span><span class="sxs-lookup"><span data-stu-id="ef79a-178">Notification entities make use of the <xref:System.ComponentModel.INotifyPropertyChanging> and <xref:System.ComponentModel.INotifyPropertyChanged> interfaces, which are part of the .NET base class library (BCL).</span></span> <span data-ttu-id="ef79a-179">这些接口定义在更改属性值之前和之后必须激发的事件。</span><span class="sxs-lookup"><span data-stu-id="ef79a-179">These interfaces define events that must be fired before and after changing a property value.</span></span> <span data-ttu-id="ef79a-180">例如：</span><span class="sxs-lookup"><span data-stu-id="ef79a-180">For example:</span></span>

<!--
    public class Blog : INotifyPropertyChanging, INotifyPropertyChanged
    {
        public event PropertyChangingEventHandler PropertyChanging;
        public event PropertyChangedEventHandler PropertyChanged;

        private int _id;
        public int Id
        {
            get => _id;
            set
            {
                PropertyChanging?.Invoke(this, new PropertyChangingEventArgs(nameof(Id)));
                _id = value;
                PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(Id)));
            }
        }

        private string _name;
        public string Name
        {
            get => _name;
            set
            {
                PropertyChanging?.Invoke(this, new PropertyChangingEventArgs(nameof(Name)));
                _name = value;
                PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(Name)));
            }
        }

        public IList<Post> Posts { get; } = new ObservableCollection<Post>();
    }
-->
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/NotificationEntitiesSamples.cs?name=Model)]

<span data-ttu-id="ef79a-181">此外，任何集合导航都必须实现 `INotifyCollectionChanged` ; 在上面的示例中，通过使用 post 来满足这一要求 <xref:System.Collections.ObjectModel.ObservableCollection%601> 。</span><span class="sxs-lookup"><span data-stu-id="ef79a-181">In addition, any collection navigations must implement `INotifyCollectionChanged`; in the example above this satisfied by using an <xref:System.Collections.ObjectModel.ObservableCollection%601> of posts.</span></span> <span data-ttu-id="ef79a-182">EF Core 还附带了一个 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ObservableHashSet%601> 实现，该实现具有更高效的查找，但代价是稳定的排序。</span><span class="sxs-lookup"><span data-stu-id="ef79a-182">EF Core also ships with an <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ObservableHashSet%601> implementation that has more efficient lookups at the expense of stable ordering.</span></span>

<span data-ttu-id="ef79a-183">大多数此类通知代码通常会移到未映射的基类中。</span><span class="sxs-lookup"><span data-stu-id="ef79a-183">Most of this notification code is typically moved into an unmapped base class.</span></span> <span data-ttu-id="ef79a-184">例如：</span><span class="sxs-lookup"><span data-stu-id="ef79a-184">For example:</span></span>

<!--
    public class Blog : NotifyingEntity
    {
        private int _id;
        public int Id
        {
            get => _id;
            set => SetWithNotify(value, out _id);
        }

        private string _name;
        public string Name
        {
            get => _name;
            set => SetWithNotify(value, out _name);
        }

        public IList<Post> Posts { get; } = new ObservableCollection<Post>();
    }

    public abstract class NotifyingEntity : INotifyPropertyChanging, INotifyPropertyChanged
    {
        protected void SetWithNotify<T>(T value, out T field, [CallerMemberName] string propertyName = "")
        {
            NotifyChanging(propertyName);
            field = value;
            NotifyChanged(propertyName);
        }

        public event PropertyChangingEventHandler PropertyChanging;
        public event PropertyChangedEventHandler PropertyChanged;

        private void NotifyChanged(string propertyName)
            => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));

        private void NotifyChanging(string propertyName)
            => PropertyChanging?.Invoke(this, new PropertyChangingEventArgs(propertyName));
    }
-->
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/NotificationWithBaseSamples.cs?name=Model)]

### <a name="configuring-notification-entities"></a><span data-ttu-id="ef79a-185">配置通知实体</span><span class="sxs-lookup"><span data-stu-id="ef79a-185">Configuring notification entities</span></span>

<span data-ttu-id="ef79a-186">无法 EF Core 验证是否已 `INotifyPropertyChanging` `INotifyPropertyChanged` 完全实现或使用 EF Core。</span><span class="sxs-lookup"><span data-stu-id="ef79a-186">There is no way for EF Core to validate that `INotifyPropertyChanging` or `INotifyPropertyChanged` are fully implemented for use with EF Core.</span></span> <span data-ttu-id="ef79a-187">具体而言，这些接口的某些用途只对某些属性使用通知，而不是根据 EF Core 所需 (包括导航) 的所有属性。</span><span class="sxs-lookup"><span data-stu-id="ef79a-187">In particular, some uses of these interfaces do so with notifications only on certain properties, rather than on all properties (including navigations) as required by EF Core.</span></span> <span data-ttu-id="ef79a-188">出于此原因，EF Core 不会自动挂接到这些事件。</span><span class="sxs-lookup"><span data-stu-id="ef79a-188">For this reason, EF Core does not automatically hook into these events.</span></span>

<span data-ttu-id="ef79a-189">相反，EF Core 必须配置为使用这些通知实体。</span><span class="sxs-lookup"><span data-stu-id="ef79a-189">Instead, EF Core must be configured to use these notification entities.</span></span> <span data-ttu-id="ef79a-190">通常通过调用对所有实体类型执行此操作 <xref:Microsoft.EntityFrameworkCore.ModelBuilder.HasChangeTrackingStrategy%2A?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="ef79a-190">This is usually done for all entity types by calling <xref:Microsoft.EntityFrameworkCore.ModelBuilder.HasChangeTrackingStrategy%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="ef79a-191">例如：</span><span class="sxs-lookup"><span data-stu-id="ef79a-191">For example:</span></span>

<!--
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.HasChangeTrackingStrategy(ChangeTrackingStrategy.ChangingAndChangedNotifications);
        }
-->
[!code-csharp[OnModelCreating](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/NotificationWithBaseSamples.cs?name=OnModelCreating)]

<span data-ttu-id="ef79a-192"> (还可以使用不同的实体类型为不同的实体类型设置该策略 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.EntityTypeBuilder.HasChangeTrackingStrategy%2A?displayProperty=nameWithType> ，但这通常是适得其反的，因为那些不是通知实体的类型仍然需要 DetectChanges。 ) </span><span class="sxs-lookup"><span data-stu-id="ef79a-192">(The strategy can also be set differently for different entity types using <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.EntityTypeBuilder.HasChangeTrackingStrategy%2A?displayProperty=nameWithType>, but this is usually counterproductive since DetectChanges is still required for those types that are not notification entities.)</span></span>

<span data-ttu-id="ef79a-193">完全通知更改跟踪要求 `INotifyPropertyChanging` `INotifyPropertyChanged` 实现和。</span><span class="sxs-lookup"><span data-stu-id="ef79a-193">Full notification change tracking requires that both `INotifyPropertyChanging` and `INotifyPropertyChanged` are implemented.</span></span> <span data-ttu-id="ef79a-194">这样就可以在属性值更改之前保存原始值，从而无需 EF Core 在跟踪实体时创建快照。</span><span class="sxs-lookup"><span data-stu-id="ef79a-194">This allows original values to be saved just before the property value is changed, avoiding the need for EF Core to create a snapshot when tracking the entity.</span></span> <span data-ttu-id="ef79a-195">实现的实体类型 `INotifyPropertyChanged` 还可用于 EF Core。</span><span class="sxs-lookup"><span data-stu-id="ef79a-195">Entity types that implement only `INotifyPropertyChanged` can also be used with EF Core.</span></span> <span data-ttu-id="ef79a-196">在这种情况下，EF 仍会在跟踪实体时创建快照以跟踪原始值，但随后使用这些通知来检测更改，而不需要调用 DetectChanges。</span><span class="sxs-lookup"><span data-stu-id="ef79a-196">In this case, EF still creates a snapshot when tracking an entity to keep track of original values, but then uses the notifications to detect changes immediately, rather than needing DetectChanges to be called.</span></span>

<span data-ttu-id="ef79a-197"><xref:Microsoft.EntityFrameworkCore.ChangeTrackingStrategy>下表汇总了不同的值。</span><span class="sxs-lookup"><span data-stu-id="ef79a-197">The different <xref:Microsoft.EntityFrameworkCore.ChangeTrackingStrategy> values are summarized in the the following table.</span></span>

| <span data-ttu-id="ef79a-198">ChangeTrackingStrategy</span><span class="sxs-lookup"><span data-stu-id="ef79a-198">ChangeTrackingStrategy</span></span>                              | <span data-ttu-id="ef79a-199">需要的接口</span><span class="sxs-lookup"><span data-stu-id="ef79a-199">Interfaces needed</span></span>                                      | <span data-ttu-id="ef79a-200">需要 DetectChanges</span><span class="sxs-lookup"><span data-stu-id="ef79a-200">Needs DetectChanges</span></span> | <span data-ttu-id="ef79a-201">快照原始值</span><span class="sxs-lookup"><span data-stu-id="ef79a-201">Snapshots original values</span></span>
|:----------------------------------------------------|--------------------------------------------------------|---------------------|--------------------------
| <span data-ttu-id="ef79a-202">快照</span><span class="sxs-lookup"><span data-stu-id="ef79a-202">Snapshot</span></span>                                            | <span data-ttu-id="ef79a-203">None</span><span class="sxs-lookup"><span data-stu-id="ef79a-203">None</span></span>                                                   | <span data-ttu-id="ef79a-204">是</span><span class="sxs-lookup"><span data-stu-id="ef79a-204">Yes</span></span>                 | <span data-ttu-id="ef79a-205">是</span><span class="sxs-lookup"><span data-stu-id="ef79a-205">Yes</span></span>
| <span data-ttu-id="ef79a-206">ChangedNotifications</span><span class="sxs-lookup"><span data-stu-id="ef79a-206">ChangedNotifications</span></span>                                | <span data-ttu-id="ef79a-207">INotifyPropertyChanged</span><span class="sxs-lookup"><span data-stu-id="ef79a-207">INotifyPropertyChanged</span></span>                                 | <span data-ttu-id="ef79a-208">否</span><span class="sxs-lookup"><span data-stu-id="ef79a-208">No</span></span>                  | <span data-ttu-id="ef79a-209">是</span><span class="sxs-lookup"><span data-stu-id="ef79a-209">Yes</span></span>
| <span data-ttu-id="ef79a-210">ChangingAndChangedNotifications</span><span class="sxs-lookup"><span data-stu-id="ef79a-210">ChangingAndChangedNotifications</span></span>                     | <span data-ttu-id="ef79a-211">INotifyPropertyChanged 和 INotifyPropertyChanging</span><span class="sxs-lookup"><span data-stu-id="ef79a-211">INotifyPropertyChanged and INotifyPropertyChanging</span></span>     | <span data-ttu-id="ef79a-212">否</span><span class="sxs-lookup"><span data-stu-id="ef79a-212">No</span></span>                  | <span data-ttu-id="ef79a-213">否</span><span class="sxs-lookup"><span data-stu-id="ef79a-213">No</span></span>
| <span data-ttu-id="ef79a-214">ChangingAndChangedNotificationsWithOriginalValues</span><span class="sxs-lookup"><span data-stu-id="ef79a-214">ChangingAndChangedNotificationsWithOriginalValues</span></span>   | <span data-ttu-id="ef79a-215">INotifyPropertyChanged 和 INotifyPropertyChanging</span><span class="sxs-lookup"><span data-stu-id="ef79a-215">INotifyPropertyChanged and INotifyPropertyChanging</span></span>     | <span data-ttu-id="ef79a-216">否</span><span class="sxs-lookup"><span data-stu-id="ef79a-216">No</span></span>                  | <span data-ttu-id="ef79a-217">是</span><span class="sxs-lookup"><span data-stu-id="ef79a-217">Yes</span></span>

### <a name="using-notification-entities"></a><span data-ttu-id="ef79a-218">使用通知实体</span><span class="sxs-lookup"><span data-stu-id="ef79a-218">Using notification entities</span></span>

<span data-ttu-id="ef79a-219">通知实体的行为与其他任何实体类似，只是对实体实例进行更改不需要调用来 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> 检测这些更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-219">Notification entities behave like any other entities, except that making changes to the entity instances do not require a call to <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> to detect these changes.</span></span> <span data-ttu-id="ef79a-220">例如：</span><span class="sxs-lookup"><span data-stu-id="ef79a-220">For example:</span></span>

<!--
            using var context = new BlogsContext();
            var blog = context.Blogs.Include(e => e.Posts).First(e => e.Name == ".NET Blog");

            // Change a property value
            blog.Name = ".NET Blog (Updated!)";

            // Add a new entity to a navigation
            blog.Posts.Add(new Post
            {
                Title = "What’s next for System.Text.Json?",
                Content = ".NET 5.0 was released recently and has come with many..."
            });

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Notification_entities_2](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/NotificationWithBaseSamples.cs?name=Notification_entities_2)]

<span data-ttu-id="ef79a-221">对于普通实体， [更改跟踪](xref:core/change-tracking/debug-views) 器 "调试" 视图显示在调用 DetectChanges 之前未检测到这些更改。</span><span class="sxs-lookup"><span data-stu-id="ef79a-221">With normal entities, the [change tracker debug view](xref:core/change-tracking/debug-views) showed that these changes were not detected until DetectChanges was called.</span></span> <span data-ttu-id="ef79a-222">当使用通知实体时，查看调试视图表明已立即检测到这些更改：</span><span class="sxs-lookup"><span data-stu-id="ef79a-222">Looking at the debug view when notification entities are used shows that these changes have been detected immediately:</span></span>

```output
Blog {Id: 1} Modified
  Id: 1 PK
  Name: '.NET Blog (Updated!)' Modified
  Posts: [{Id: 1}, {Id: 2}, {Id: -2147482643}]
Post {Id: -2147482643} Added
  Id: -2147482643 PK Temporary
  BlogId: 1 FK
  Content: '.NET 5.0 was released recently and has come with many...'
  Title: 'What's next for System.Text.Json?'
  Blog: {Id: 1}
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
Post {Id: 2} Unchanged
  Id: 2 PK
  BlogId: 1 FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: {Id: 1}
```

## <a name="change-tracking-proxies"></a><span data-ttu-id="ef79a-223">更改跟踪代理</span><span class="sxs-lookup"><span data-stu-id="ef79a-223">Change-tracking proxies</span></span>

> [!NOTE]
> <span data-ttu-id="ef79a-224">EF Core 5.0 中引入了更改跟踪代理。</span><span class="sxs-lookup"><span data-stu-id="ef79a-224">Change-tracking proxies were introduced in EF Core 5.0.</span></span>

<span data-ttu-id="ef79a-225">EF Core 可以动态生成实现和的代理 <xref:System.ComponentModel.INotifyPropertyChanging> 类型 <xref:System.ComponentModel.INotifyPropertyChanged> 。</span><span class="sxs-lookup"><span data-stu-id="ef79a-225">EF Core can dynamically generate proxy types that implement <xref:System.ComponentModel.INotifyPropertyChanging> and <xref:System.ComponentModel.INotifyPropertyChanged>.</span></span> <span data-ttu-id="ef79a-226">这需要安装 [Microsoft.entityframeworkcore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Proxies/) NuGet 包，并启用更改跟踪代理， <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseChangeTrackingProxies%2A> 例如：</span><span class="sxs-lookup"><span data-stu-id="ef79a-226">This requires installing the [Microsoft.EntityFrameworkCore.Proxies](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Proxies/) NuGet package, and enabling change-tracking proxies with <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseChangeTrackingProxies%2A> For example:</span></span>

<!--
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
            => optionsBuilder.UseChangeTrackingProxies();
-->
[!code-csharp[OnConfiguring](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/ChangeTrackingProxiesSamples.cs?name=OnConfiguring)]

<span data-ttu-id="ef79a-227">创建动态代理涉及到使用 [城堡](https://www.nuget.org/packages/Castle.Core/) 代理实现创建新的动态 .net 类型 () ，该实现继承自实体类型，然后覆盖所有属性资源库。</span><span class="sxs-lookup"><span data-stu-id="ef79a-227">Creating a dynamic proxy involves creating a new, dynamic .NET type (using the [Castle.Core](https://www.nuget.org/packages/Castle.Core/) proxies implementation), which inherits from the entity type and then overrides all property setters.</span></span> <span data-ttu-id="ef79a-228">因此，代理的实体类型必须是可以从继承的类型，并且必须具有可重写的属性。</span><span class="sxs-lookup"><span data-stu-id="ef79a-228">Entity types for proxies must therefore be types that can be inherited from and must have properties that can be overridden.</span></span> <span data-ttu-id="ef79a-229">此外，显式创建的集合导航必须实现 <xref:System.Collections.Specialized.INotifyCollectionChanged> 示例：</span><span class="sxs-lookup"><span data-stu-id="ef79a-229">Also, collection navigations created explicitly must implement <xref:System.Collections.Specialized.INotifyCollectionChanged> For example:</span></span>

<!--
    public class Blog
    {
        public virtual int Id { get; set; }
        public virtual string Name { get; set; }

        public virtual IList<Post> Posts { get; } = new ObservableCollection<Post>();
    }

    public class Post
    {
        public virtual int Id { get; set; }
        public virtual string Title { get; set; }
        public virtual string Content { get; set; }

        public virtual int BlogId { get; set; }
        public virtual Blog Blog { get; set; }
    }
-->
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/ChangeTrackingProxiesSamples.cs?name=Model)]

<span data-ttu-id="ef79a-230">更改跟踪代理的一个重大缺点是 EF Core 必须始终跟踪代理的实例，而不是基础实体类型的实例。</span><span class="sxs-lookup"><span data-stu-id="ef79a-230">One significant downside to change-tracking proxies is that EF Core must always track instances of the proxy, never instances of the underlying entity type.</span></span> <span data-ttu-id="ef79a-231">这是因为，基础实体类型的实例将不会生成通知，这意味着对这些实体所做的更改将丢失。</span><span class="sxs-lookup"><span data-stu-id="ef79a-231">This is because instances of the underlying entity type will not generate notifications, which means changes made to these entities will be missed.</span></span>

<span data-ttu-id="ef79a-232">EF Core 在查询数据库时自动创建代理实例，因此，这种缺点通常仅限于跟踪新的实体实例。</span><span class="sxs-lookup"><span data-stu-id="ef79a-232">EF Core creates proxy instances automatically when querying the database, so this downside is generally limited to tracking new entity instances.</span></span> <span data-ttu-id="ef79a-233">这些实例必须使用 <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.CreateProxy%2A> 扩展方法创建，而 **不** 是使用来创建 `new` 。</span><span class="sxs-lookup"><span data-stu-id="ef79a-233">These instances must be created using the <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.CreateProxy%2A> extension methods, and **not** in the normal way using `new`.</span></span> <span data-ttu-id="ef79a-234">这意味着，前面的示例中的代码现在必须使用 `CreateProxy` ：</span><span class="sxs-lookup"><span data-stu-id="ef79a-234">This means the code from the previous examples must now make use of `CreateProxy`:</span></span>

<!--
            using var context = new BlogsContext();
            var blog = context.Blogs.Include(e => e.Posts).First(e => e.Name == ".NET Blog");

            // Change a property value
            blog.Name = ".NET Blog (Updated!)";

            // Add a new entity to a navigation
            blog.Posts.Add(
                context.CreateProxy<Post>(
                    p =>
                        {
                            p.Title = "What’s next for System.Text.Json?";
                            p.Content = ".NET 5.0 was released recently and has come with many...";
                        }));

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Change_tracking_proxies_1](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/ChangeTrackingProxiesSamples.cs?name=Change_tracking_proxies_1)]

## <a name="change-tracking-events"></a><span data-ttu-id="ef79a-235">更改跟踪事件</span><span class="sxs-lookup"><span data-stu-id="ef79a-235">Change tracking events</span></span>

<span data-ttu-id="ef79a-236">EF Core <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Tracked?displayProperty=nameWithType> 第一次跟踪实体时，会触发事件。</span><span class="sxs-lookup"><span data-stu-id="ef79a-236">EF Core fires the <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Tracked?displayProperty=nameWithType> event when an entity is tracked for the first time.</span></span> <span data-ttu-id="ef79a-237">将来的实体状态更改会导致 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.StateChanged?displayProperty=nameWithType> 事件发生。</span><span class="sxs-lookup"><span data-stu-id="ef79a-237">Future entity state changes result in <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.StateChanged?displayProperty=nameWithType> events.</span></span> <span data-ttu-id="ef79a-238">有关详细信息，请参阅 [EF Core 中的 .NET 事件](xref:core/logging-events-diagnostics/events)。</span><span class="sxs-lookup"><span data-stu-id="ef79a-238">See [.NET Events in EF Core](xref:core/logging-events-diagnostics/events) for more information.</span></span>

> [!NOTE]
> <span data-ttu-id="ef79a-239">`StateChanged`第一次跟踪实体时不会触发事件，即使状态已从更改 `Detached` 为其他状态之一。</span><span class="sxs-lookup"><span data-stu-id="ef79a-239">The `StateChanged` event is not fired when an entity is first tracked, even though the state has changed from `Detached` to one of the other states.</span></span> <span data-ttu-id="ef79a-240">请确保侦听 `StateChanged` 和 `Tracked` 事件以获取所有相关的通知。</span><span class="sxs-lookup"><span data-stu-id="ef79a-240">Make sure to listen for both `StateChanged` and `Tracked` events to get all relevant notifications.</span></span>
