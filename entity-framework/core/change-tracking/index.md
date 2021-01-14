---
title: 更改跟踪 - EF Core
description: EF Core 的更改跟踪概述
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/index
ms.openlocfilehash: 52223e5472b09271d19ac9449a3989b4a0e277f7
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129517"
---
# <a name="change-tracking-in-ef-core"></a><span data-ttu-id="20560-103">EF Core 中的更改跟踪</span><span class="sxs-lookup"><span data-stu-id="20560-103">Change Tracking in EF Core</span></span>

<span data-ttu-id="20560-104">每个 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例跟踪对实体所做的更改。</span><span class="sxs-lookup"><span data-stu-id="20560-104">Each <xref:Microsoft.EntityFrameworkCore.DbContext> instance tracks changes made to entities.</span></span> <span data-ttu-id="20560-105">在调用 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 时，这些跟踪的实体会相应地驱动对数据库的更改。</span><span class="sxs-lookup"><span data-stu-id="20560-105">These tracked entities in turn drive the changes to the database when <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> is called.</span></span>

<span data-ttu-id="20560-106">本文档概述了 Entity Framework Core (EF Core) 更改跟踪，以及它如何与查询和更新相关。</span><span class="sxs-lookup"><span data-stu-id="20560-106">This document presents an overview of Entity Framework Core (EF Core) change tracking and how it relates to queries and updates.</span></span>

> [!TIP]
> <span data-ttu-id="20560-107">[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangeTrackingInEFCore)，你可以运行并调试到本文档中的所有代码。</span><span class="sxs-lookup"><span data-stu-id="20560-107">You can run and debug into all the code in this document by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangeTrackingInEFCore).</span></span>

> [!TIP]
> <span data-ttu-id="20560-108">为简单起见，本文档使用和引用同步方法（如 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A>），而不是它们的异步等效方法（如 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A>）。</span><span class="sxs-lookup"><span data-stu-id="20560-108">For simplicity, this document uses and references synchronous methods such as <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> rather their async equivalents such as <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A>.</span></span> <span data-ttu-id="20560-109">除非另有说明，否则可以替换调用并等待异步方法。</span><span class="sxs-lookup"><span data-stu-id="20560-109">Calling and awaiting the async method can be substituted unless otherwise noted.</span></span>

## <a name="how-to-track-entities"></a><span data-ttu-id="20560-110">如何跟踪实体</span><span class="sxs-lookup"><span data-stu-id="20560-110">How to track entities</span></span>

<span data-ttu-id="20560-111">实体实例在以下情况下会被跟踪：</span><span class="sxs-lookup"><span data-stu-id="20560-111">Entity instances become tracked when they are:</span></span>

- <span data-ttu-id="20560-112">从针对数据库执行的查询返回</span><span class="sxs-lookup"><span data-stu-id="20560-112">Returned from a query executed against the database</span></span>
- <span data-ttu-id="20560-113">通过 `Add`、`Attach`、`Update` 或类似方法显示附加到 DbContext</span><span class="sxs-lookup"><span data-stu-id="20560-113">Explicitly attached to the DbContext by `Add`, `Attach`, `Update`, or similar methods</span></span>
- <span data-ttu-id="20560-114">检测为连接到现有跟踪实体的新实体</span><span class="sxs-lookup"><span data-stu-id="20560-114">Detected as new entities connected to existing tracked entities</span></span>

<span data-ttu-id="20560-115">实体实例在以下情况下不再被跟踪：</span><span class="sxs-lookup"><span data-stu-id="20560-115">Entity instances are no longer tracked when:</span></span>

- <span data-ttu-id="20560-116">DbContext 已释放</span><span class="sxs-lookup"><span data-stu-id="20560-116">The DbContext is disposed</span></span>
- <span data-ttu-id="20560-117">更改跟踪器已被清除（EF Core 5.0 及更高版本）</span><span class="sxs-lookup"><span data-stu-id="20560-117">The change tracker is cleared (EF Core 5.0 and later)</span></span>
- <span data-ttu-id="20560-118">显式拆离实体</span><span class="sxs-lookup"><span data-stu-id="20560-118">The entities are explicitly detached</span></span>

<span data-ttu-id="20560-119">DbContext 旨在表示短期工作单元，如 [DbContext 初始化和配置](xref:core/dbcontext-configuration/index)中所述。</span><span class="sxs-lookup"><span data-stu-id="20560-119">DbContext is designed to represent a short-lived unit-of-work, as described in [DbContext Initialization and Configuration](xref:core/dbcontext-configuration/index).</span></span> <span data-ttu-id="20560-120">这意味着释放 DbContext 是停止跟踪实体的正常方式。</span><span class="sxs-lookup"><span data-stu-id="20560-120">This means that disposing the DbContext is _the normal way_ to stop tracking entities.</span></span> <span data-ttu-id="20560-121">换句话说，DbContext 的生存期应为：</span><span class="sxs-lookup"><span data-stu-id="20560-121">In other words, the lifetime of a DbContext should be:</span></span>

1. <span data-ttu-id="20560-122">创建 DbContext 实例</span><span class="sxs-lookup"><span data-stu-id="20560-122">Create the DbContext instance</span></span>
2. <span data-ttu-id="20560-123">跟踪某些实体</span><span class="sxs-lookup"><span data-stu-id="20560-123">Track some entities</span></span>
3. <span data-ttu-id="20560-124">对实体进行一些更改</span><span class="sxs-lookup"><span data-stu-id="20560-124">Make some changes to the entities</span></span>
4. <span data-ttu-id="20560-125">调用 SaveChanges 以更新数据库</span><span class="sxs-lookup"><span data-stu-id="20560-125">Call SaveChanges to update the database</span></span>
5. <span data-ttu-id="20560-126">释放 DbContext 实例</span><span class="sxs-lookup"><span data-stu-id="20560-126">Dispose the DbContext instance</span></span>

> [!TIP]
> <span data-ttu-id="20560-127">采用此方法时，无需清除更改跟踪器或显式拆离实体实例。</span><span class="sxs-lookup"><span data-stu-id="20560-127">It is not necessary to clear the change tracker or explicitly detach entity instances when taking this approach.</span></span> <span data-ttu-id="20560-128">但是，如果确实需要拆离实体，则调用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Clear%2A?displayProperty=nameWithType> 比逐个拆离实体更有效。</span><span class="sxs-lookup"><span data-stu-id="20560-128">However, if you do need to detach entities, then calling <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Clear%2A?displayProperty=nameWithType> is more efficient than detaching entities one-by-one.</span></span>

## <a name="entity-states"></a><span data-ttu-id="20560-129">实体状态</span><span class="sxs-lookup"><span data-stu-id="20560-129">Entity states</span></span>

<span data-ttu-id="20560-130">每个实体都与给定 <xref:Microsoft.EntityFrameworkCore.EntityState> 相关联：</span><span class="sxs-lookup"><span data-stu-id="20560-130">Every entity is is associated with a given <xref:Microsoft.EntityFrameworkCore.EntityState>:</span></span>

- <span data-ttu-id="20560-131">`Detached` 实体未被 <xref:Microsoft.EntityFrameworkCore.DbContext> 跟踪。</span><span class="sxs-lookup"><span data-stu-id="20560-131">`Detached` entities are not being tracked by the <xref:Microsoft.EntityFrameworkCore.DbContext>.</span></span>
- <span data-ttu-id="20560-132">`Added` 实体是新实体，并且尚未插入到数据库中。</span><span class="sxs-lookup"><span data-stu-id="20560-132">`Added` entities are new and have not yet been inserted into the database.</span></span> <span data-ttu-id="20560-133">这意味着它们将在调用 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 时插入。</span><span class="sxs-lookup"><span data-stu-id="20560-133">This means they will be inserted when <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> is called.</span></span>
- <span data-ttu-id="20560-134">`Unchanged` 实体自从数据库中查询以来尚未进行更改。</span><span class="sxs-lookup"><span data-stu-id="20560-134">`Unchanged` entities have _not_ been changed since they were queried from the database.</span></span> <span data-ttu-id="20560-135">从查询返回的所有实体最初都处于此状态。</span><span class="sxs-lookup"><span data-stu-id="20560-135">All entities returned from queries are initially in this state.</span></span>
- <span data-ttu-id="20560-136">`Modified` 实体自从数据库中查询以来已进行更改。</span><span class="sxs-lookup"><span data-stu-id="20560-136">`Modified` entities have been changed since they were queried from the database.</span></span> <span data-ttu-id="20560-137">这意味着它们将在调用 SaveChanges 时更新。</span><span class="sxs-lookup"><span data-stu-id="20560-137">This means they will be updated when SaveChanges is called.</span></span>
- <span data-ttu-id="20560-138">`Deleted` 实体存在于数据库中，但标记为在调用 SaveChanges 时删除。</span><span class="sxs-lookup"><span data-stu-id="20560-138">`Deleted` entities exist in the database, but are marked to be deleted when SaveChanges is called.</span></span>

<span data-ttu-id="20560-139">EF Core 跟踪属性级别的更改。</span><span class="sxs-lookup"><span data-stu-id="20560-139">EF Core tracks changes at the property level.</span></span> <span data-ttu-id="20560-140">例如，如果只修改单个属性值，则数据库更新将仅更改该值。</span><span class="sxs-lookup"><span data-stu-id="20560-140">For example, if only a single property value is modified, then a database update will change only that value.</span></span> <span data-ttu-id="20560-141">但是，当实体本身处于“已修改”状态时，只能将属性标记为已修改。</span><span class="sxs-lookup"><span data-stu-id="20560-141">However, properties can only be marked as modified when the entity itself is in the Modified state.</span></span> <span data-ttu-id="20560-142">（或者，从另一角度来看，“已修改”状态意味着至少有一个属性值已标记为已修改。）</span><span class="sxs-lookup"><span data-stu-id="20560-142">(Or, from an alternate perspective, the Modified state means that at least one property value has been marked as modified.)</span></span>

<span data-ttu-id="20560-143">下表汇总了不同的状态：</span><span class="sxs-lookup"><span data-stu-id="20560-143">The following table summarizes the different states:</span></span>

| <span data-ttu-id="20560-144">实体状态</span><span class="sxs-lookup"><span data-stu-id="20560-144">Entity state</span></span>     | <span data-ttu-id="20560-145">由 DbContext 跟踪</span><span class="sxs-lookup"><span data-stu-id="20560-145">Tracked by DbContext</span></span> | <span data-ttu-id="20560-146">存在于数据库中</span><span class="sxs-lookup"><span data-stu-id="20560-146">Exists in database</span></span> | <span data-ttu-id="20560-147">属性已修改</span><span class="sxs-lookup"><span data-stu-id="20560-147">Properties modified</span></span> | <span data-ttu-id="20560-148">SaveChanges 上的操作</span><span class="sxs-lookup"><span data-stu-id="20560-148">Action on SaveChanges</span></span>
|:-----------------|----------------------|--------------------|---------------------|-----------------------
| `Detached`       | <span data-ttu-id="20560-149">否</span><span class="sxs-lookup"><span data-stu-id="20560-149">No</span></span>                   | -                  | -                   | -
| `Added`          | <span data-ttu-id="20560-150">是</span><span class="sxs-lookup"><span data-stu-id="20560-150">Yes</span></span>                  | <span data-ttu-id="20560-151">否</span><span class="sxs-lookup"><span data-stu-id="20560-151">No</span></span>                 | -                   | <span data-ttu-id="20560-152">插入</span><span class="sxs-lookup"><span data-stu-id="20560-152">Insert</span></span>
| `Unchanged`      | <span data-ttu-id="20560-153">是</span><span class="sxs-lookup"><span data-stu-id="20560-153">Yes</span></span>                  | <span data-ttu-id="20560-154">是</span><span class="sxs-lookup"><span data-stu-id="20560-154">Yes</span></span>                | <span data-ttu-id="20560-155">否</span><span class="sxs-lookup"><span data-stu-id="20560-155">No</span></span>                  | -
| `Modified`       | <span data-ttu-id="20560-156">是</span><span class="sxs-lookup"><span data-stu-id="20560-156">Yes</span></span>                  | <span data-ttu-id="20560-157">是</span><span class="sxs-lookup"><span data-stu-id="20560-157">Yes</span></span>                | <span data-ttu-id="20560-158">是</span><span class="sxs-lookup"><span data-stu-id="20560-158">Yes</span></span>                 | <span data-ttu-id="20560-159">更新</span><span class="sxs-lookup"><span data-stu-id="20560-159">Update</span></span>
| `Deleted`        | <span data-ttu-id="20560-160">是</span><span class="sxs-lookup"><span data-stu-id="20560-160">Yes</span></span>                  | <span data-ttu-id="20560-161">是</span><span class="sxs-lookup"><span data-stu-id="20560-161">Yes</span></span>                | -                   | <span data-ttu-id="20560-162">删除</span><span class="sxs-lookup"><span data-stu-id="20560-162">Delete</span></span>

> [!NOTE]
> <span data-ttu-id="20560-163">为清楚起见，此文本使用了关系数据库术语。</span><span class="sxs-lookup"><span data-stu-id="20560-163">This text uses relational database terms for clarity.</span></span> <span data-ttu-id="20560-164">NoSQL 数据库通常支持类似操作，但可能具有不同的名称。</span><span class="sxs-lookup"><span data-stu-id="20560-164">NoSQL databases typically support similar operations but possibly with different names.</span></span> <span data-ttu-id="20560-165">有关详细信息，请查阅数据库提供程序文档。</span><span class="sxs-lookup"><span data-stu-id="20560-165">Consult your database provider documentation for more information.</span></span>

## <a name="tracking-from-queries"></a><span data-ttu-id="20560-166">从查询跟踪</span><span class="sxs-lookup"><span data-stu-id="20560-166">Tracking from queries</span></span>

<span data-ttu-id="20560-167">当同一个 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例同时用于查询实体并通过调用 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 更新它们时，EF Core 更改跟踪的效果最佳。</span><span class="sxs-lookup"><span data-stu-id="20560-167">EF Core change tracking works best when the same <xref:Microsoft.EntityFrameworkCore.DbContext> instance is used to both query for entities and update them by calling <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A>.</span></span> <span data-ttu-id="20560-168">这是因为 EF Core 会自动跟踪已查询实体的状态，然后在调用 SaveChanges 时检测对这些实体所做的任何更改。</span><span class="sxs-lookup"><span data-stu-id="20560-168">This is because EF Core automatically tracks the state of queried entities and then detects any changes made to these entities when SaveChanges is called.</span></span>

<span data-ttu-id="20560-169">与[显式跟踪实体实例](xref:core/change-tracking/explicit-tracking)相比，此方法具有多个优点：</span><span class="sxs-lookup"><span data-stu-id="20560-169">This approach has several advantages over [explicitly tracking entity instances](xref:core/change-tracking/explicit-tracking):</span></span>

- <span data-ttu-id="20560-170">该实例为简单类型。</span><span class="sxs-lookup"><span data-stu-id="20560-170">It is simple.</span></span> <span data-ttu-id="20560-171">实体状态极少需要显式操作，EF Core 负责处理状态更改。</span><span class="sxs-lookup"><span data-stu-id="20560-171">Entity states rarely need to be manipulated explicitly--EF Core takes care of state changes.</span></span>
- <span data-ttu-id="20560-172">更新仅限于那些实际已更改的值。</span><span class="sxs-lookup"><span data-stu-id="20560-172">Updates are limited to only those values that have actually changed.</span></span>
- <span data-ttu-id="20560-173">[卷影属性](xref:core/modeling/shadow-properties)的值将保留，并根据需要使用。</span><span class="sxs-lookup"><span data-stu-id="20560-173">The values of [shadow properties](xref:core/modeling/shadow-properties) are preserved and used as needed.</span></span> <span data-ttu-id="20560-174">当外键以卷影状态存储时，这一点尤其重要。</span><span class="sxs-lookup"><span data-stu-id="20560-174">This is especially relevant when foreign keys are stored in shadow state.</span></span>
- <span data-ttu-id="20560-175">自动保留属性的原始值，并将其用于有效更新。</span><span class="sxs-lookup"><span data-stu-id="20560-175">The original values of properties are preserved automatically and used for efficient updates.</span></span>

## <a name="simple-query-and-update"></a><span data-ttu-id="20560-176">简单查询和更新</span><span class="sxs-lookup"><span data-stu-id="20560-176">Simple query and update</span></span>

<span data-ttu-id="20560-177">例如，请考虑一个简单的博客/帖子模型：</span><span class="sxs-lookup"><span data-stu-id="20560-177">For example, consider a simple blog/posts model:</span></span>

<!--
public class Blog
{
    public int Id { get; set; }

    public string Name { get; set; }

    public ICollection<Post> Posts { get; } = new List<Post>();
}

public class Post
{
    public int Id { get; set; }

    public string Title { get; set; }
    public string Content { get; set; }

    public int? BlogId { get; set; }
    public Blog Blog { get; set; }
}
-->
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/GeneratedKeysSamples.cs?name=Model)]

<span data-ttu-id="20560-178">我们可以使用此模型来查询博客和帖子，然后对数据库进行一些更新：</span><span class="sxs-lookup"><span data-stu-id="20560-178">We can use this model to query for blogs and posts and then make some updates to the database:</span></span>

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.Include(e => e.Posts).First(e => e.Name == ".NET Blog");

            blog.Name = ".NET Blog (Updated!)";

            foreach (var post in blog.Posts.Where(e => !e.Title.Contains("5.0")))
            {
                post.Title = post.Title.Replace("5", "5.0");
            }

            context.SaveChanges();
-->
[!code-csharp[Simple_query_and_update_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/GeneratedKeysSamples.cs?name=Simple_query_and_update_1)]

<span data-ttu-id="20560-179">使用 SQLite 作为示例数据库，调用 SaveChanges 会生成以下数据库更新：</span><span class="sxs-lookup"><span data-stu-id="20560-179">Calling SaveChanges results in the following database updates, using SQLite as an example database:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p1='1' (DbType = String), @p0='.NET Blog (Updated!)' (Size = 20)], CommandType='Text', CommandTimeout='30']
UPDATE "Blogs" SET "Name" = @p0
WHERE "Id" = @p1;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p1='2' (DbType = String), @p0='Announcing F# 5.0' (Size = 17)], CommandType='Text', CommandTimeout='30']
UPDATE "Posts" SET "Title" = @p0
WHERE "Id" = @p1;
SELECT changes();
```

<span data-ttu-id="20560-180">[更改跟踪器调试视图](xref:core/change-tracking/debug-views)是一种可以直观看到正在被跟踪的实体及其状态的极佳方式。</span><span class="sxs-lookup"><span data-stu-id="20560-180">The [change tracker debug view](xref:core/change-tracking/debug-views) is a great way visualize which entities are being tracked and what their states are.</span></span> <span data-ttu-id="20560-181">例如，在调用 SaveChanges 之前，将以下代码插入到上面的示例中：</span><span class="sxs-lookup"><span data-stu-id="20560-181">For example, inserting the following code into the sample above before calling SaveChanges:</span></span>

<!--
                context.ChangeTracker.DetectChanges();
                Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Simple_query_and_update_2](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/GeneratedKeysSamples.cs?name=Simple_query_and_update_2)]

<span data-ttu-id="20560-182">生成以下输出：</span><span class="sxs-lookup"><span data-stu-id="20560-182">Generates the following output:</span></span>

```output
Blog {Id: 1} Modified
  Id: 1 PK
  Name: '.NET Blog (Updated!)' Modified Originally '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}, {Id: 3}]
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
Post {Id: 2} Modified
  Id: 2 PK
  BlogId: 1 FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5.0' Modified Originally 'Announcing F# 5'
  Blog: {Id: 1}
```

<span data-ttu-id="20560-183">具体而言，请注意：</span><span class="sxs-lookup"><span data-stu-id="20560-183">Notice specifically:</span></span>

- <span data-ttu-id="20560-184">`Blog.Name` 属性标记为已修改 (`Name: '.NET Blog (Updated!)' Modified Originally '.NET Blog'`)，这会导致博客处于 `Modified` 状态。</span><span class="sxs-lookup"><span data-stu-id="20560-184">The `Blog.Name` property is marked as modified (`Name: '.NET Blog (Updated!)' Modified Originally '.NET Blog'`), and this results in the blog being in the `Modified` state.</span></span>
- <span data-ttu-id="20560-185">帖子 2 的 `Post.Title` 属性标记为已修改 (`Title: 'Announcing F# 5.0' Modified Originally 'Announcing F# 5'`)，这会导致此帖子处于 `Modified` 状态。</span><span class="sxs-lookup"><span data-stu-id="20560-185">The `Post.Title` property of post 2 is marked as modified (`Title: 'Announcing F# 5.0' Modified Originally 'Announcing F# 5'`), and this results in this post being in the `Modified` state.</span></span>
- <span data-ttu-id="20560-186">帖子 2 的其他属性值尚未更改，因此未标记为已修改。</span><span class="sxs-lookup"><span data-stu-id="20560-186">The other property values of post 2 have not changed and are therefore not marked as modified.</span></span> <span data-ttu-id="20560-187">这就是这些值不包含在数据库更新中的原因。</span><span class="sxs-lookup"><span data-stu-id="20560-187">This is why these values are not included in the database update.</span></span>
- <span data-ttu-id="20560-188">不会以任何方式修改另一个帖子。</span><span class="sxs-lookup"><span data-stu-id="20560-188">The other post was not modified in any way.</span></span> <span data-ttu-id="20560-189">这就是它仍处于 `Unchanged` 状态且不包括在数据库更新中的原因。</span><span class="sxs-lookup"><span data-stu-id="20560-189">This is why it is still in the `Unchanged` state and is not included in the database update.</span></span>

## <a name="query-then-insert-update-and-delete"></a><span data-ttu-id="20560-190">查询，然后插入、更新和删除</span><span class="sxs-lookup"><span data-stu-id="20560-190">Query then insert, update, and delete</span></span>

<span data-ttu-id="20560-191">与上一示例中的更新类似，可以将更新与插入和删除合并在同一工作单元中。</span><span class="sxs-lookup"><span data-stu-id="20560-191">Updates like those in the previous example can be combined with inserts and deletes in the same unit-of-work.</span></span> <span data-ttu-id="20560-192">例如：</span><span class="sxs-lookup"><span data-stu-id="20560-192">For example:</span></span>

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.Include(e => e.Posts).First(e => e.Name == ".NET Blog");

            // Modify property values
            blog.Name = ".NET Blog (Updated!)";

            // Insert a new Post
            blog.Posts.Add(new Post
            {
                Title = "What’s next for System.Text.Json?",
                Content = ".NET 5.0 was released recently and has come with many..."
            });

            // Mark an existing Post as Deleted
            var postToDelete = blog.Posts.Single(e => e.Title == "Announcing F# 5");
            context.Remove(postToDelete);

            context.ChangeTracker.DetectChanges();
            Console.WriteLine(context.ChangeTracker.DebugView.LongView);

            context.SaveChanges();
-->
[!code-csharp[Query_then_insert_update_and_delete_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/GeneratedKeysSamples.cs?name=Query_then_insert_update_and_delete_1)]

<span data-ttu-id="20560-193">在此示例中：</span><span class="sxs-lookup"><span data-stu-id="20560-193">In this example:</span></span>

- <span data-ttu-id="20560-194">从数据库查询博客和相关帖子并进行跟踪</span><span class="sxs-lookup"><span data-stu-id="20560-194">A blog and related posts are queried from the database and tracked</span></span>
- <span data-ttu-id="20560-195">`Blog.Name` 属性已更改</span><span class="sxs-lookup"><span data-stu-id="20560-195">The `Blog.Name` property is changed</span></span>
- <span data-ttu-id="20560-196">新帖子将添加到博客的现有帖子集合中</span><span class="sxs-lookup"><span data-stu-id="20560-196">A new post is added to the collection of existing posts for the blog</span></span>
- <span data-ttu-id="20560-197">通过调用 <xref:Microsoft.EntityFrameworkCore.DbContext.Remove%2A?displayProperty=nameWithType> 将现有帖子标记为要删除</span><span class="sxs-lookup"><span data-stu-id="20560-197">An existing post is marked for deletion by calling <xref:Microsoft.EntityFrameworkCore.DbContext.Remove%2A?displayProperty=nameWithType></span></span>

<span data-ttu-id="20560-198">在调用 SaveChanges 之前，再次查看[更改跟踪器调试视图](xref:core/change-tracking/debug-views)，会显示 EF Core 跟踪这些更改的方式：</span><span class="sxs-lookup"><span data-stu-id="20560-198">Looking again at the [change tracker debug view](xref:core/change-tracking/debug-views) before calling SaveChanges shows how EF Core is tracking these changes:</span></span>

```output
Blog {Id: 1} Modified
  Id: 1 PK
  Name: '.NET Blog (Updated!)' Modified Originally '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}, {Id: 3}, {Id: -2147482638}]
Post {Id: -2147482638} Added
  Id: -2147482638 PK Temporary
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
Post {Id: 2} Deleted
  Id: 2 PK
  BlogId: 1 FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: {Id: 1}
```

<span data-ttu-id="20560-199">请注意：</span><span class="sxs-lookup"><span data-stu-id="20560-199">Notice that:</span></span>

- <span data-ttu-id="20560-200">博客标记为 `Modified`。</span><span class="sxs-lookup"><span data-stu-id="20560-200">The blog is marked as `Modified`.</span></span> <span data-ttu-id="20560-201">这将生成一个数据库更新。</span><span class="sxs-lookup"><span data-stu-id="20560-201">This will generate a database update.</span></span>
- <span data-ttu-id="20560-202">帖子 2 标记为 `Deleted`。</span><span class="sxs-lookup"><span data-stu-id="20560-202">Post 2 is marked as `Deleted`.</span></span> <span data-ttu-id="20560-203">这将生成一个数据库删除。</span><span class="sxs-lookup"><span data-stu-id="20560-203">This will generate a database delete.</span></span>
- <span data-ttu-id="20560-204">具有临时 ID 的新帖子与博客 1 相关联，并标记为 `Added`。</span><span class="sxs-lookup"><span data-stu-id="20560-204">A new post with a temporary ID is associated with blog 1 and is marked as `Added`.</span></span> <span data-ttu-id="20560-205">这将生成一个数据库插入。</span><span class="sxs-lookup"><span data-stu-id="20560-205">This will generate a database insert.</span></span>

<span data-ttu-id="20560-206">在调用 SaveChanges 时，这会生成以下数据库命令（使用 SQLite）：</span><span class="sxs-lookup"><span data-stu-id="20560-206">This results in the following database commands (using SQLite) when SaveChanges is called:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p1='1' (DbType = String), @p0='.NET Blog (Updated!)' (Size = 20)], CommandType='Text', CommandTimeout='30']
UPDATE "Blogs" SET "Name" = @p0
WHERE "Id" = @p1;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p0='2' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Posts"
WHERE "Id" = @p0;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p0='1' (DbType = String), @p1='.NET 5.0 was released recently and has come with many...' (Size = 56), @p2='What's next for System.Text.Json?' (Size = 33)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Posts" ("BlogId", "Content", "Title")
VALUES (@p0, @p1, @p2);
SELECT "Id"
FROM "Posts"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();
```

<span data-ttu-id="20560-207">有关插入和删除实体的详细信息，请参阅[显式跟踪实体](xref:core/change-tracking/explicit-tracking)。</span><span class="sxs-lookup"><span data-stu-id="20560-207">See [Explicitly Tracking Entities](xref:core/change-tracking/explicit-tracking) for more information on inserting and deleting entities.</span></span> <span data-ttu-id="20560-208">有关 EF Core 如何自动检测此类更改的详细信息，请参阅[更改检测和通知](xref:core/change-tracking/change-detection)。</span><span class="sxs-lookup"><span data-stu-id="20560-208">See [Change Detection and Notifications](xref:core/change-tracking/change-detection) for more information on how EF Core automatically detects changes like this.</span></span>

> [!TIP]
> <span data-ttu-id="20560-209">调用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.HasChanges?displayProperty=nameWithType> 来确定是否进行了任何更改，这将导致 SaveChanges 对数据库进行更新。</span><span class="sxs-lookup"><span data-stu-id="20560-209">Call <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.HasChanges?displayProperty=nameWithType> to determine whether any changes have been made that will cause SaveChanges to make updates to the database.</span></span> <span data-ttu-id="20560-210">如果 HasChanges 返回 false，则 SaveChanges 将为无选项。</span><span class="sxs-lookup"><span data-stu-id="20560-210">If HasChanges return false, then SaveChanges will be a no-op.</span></span>
