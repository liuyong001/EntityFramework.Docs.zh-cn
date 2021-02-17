---
title: 显式跟踪实体-EF Core
description: 使用 "添加"、"附加"、"更新" 和 "删除" 通过 DbContext 显式跟踪实体
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/explicit-tracking
ms.openlocfilehash: 3d9142cecf272c635c3a041fe6c5d9c49a26c33d
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543180"
---
# <a name="explicitly-tracking-entities"></a><span data-ttu-id="7fee9-103">显式跟踪实体</span><span class="sxs-lookup"><span data-stu-id="7fee9-103">Explicitly Tracking Entities</span></span>

<span data-ttu-id="7fee9-104">每个 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例跟踪对实体所做的更改。</span><span class="sxs-lookup"><span data-stu-id="7fee9-104">Each <xref:Microsoft.EntityFrameworkCore.DbContext> instance tracks changes made to entities.</span></span> <span data-ttu-id="7fee9-105">在调用 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 时，这些跟踪的实体会相应地驱动对数据库的更改。</span><span class="sxs-lookup"><span data-stu-id="7fee9-105">These tracked entities in turn drive the changes to the database when <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> is called.</span></span>

<span data-ttu-id="7fee9-106">Entity Framework Core (EF Core 当同一 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例同时用于查询实体并通过调用更新实体时，) 更改跟踪的效果最佳。 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A></span><span class="sxs-lookup"><span data-stu-id="7fee9-106">Entity Framework Core (EF Core) change tracking works best when the same <xref:Microsoft.EntityFrameworkCore.DbContext> instance is used to both query for entities and update them by calling <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A>.</span></span> <span data-ttu-id="7fee9-107">这是因为 EF Core 会自动跟踪已查询实体的状态，然后在调用 SaveChanges 时检测对这些实体所做的任何更改。</span><span class="sxs-lookup"><span data-stu-id="7fee9-107">This is because EF Core automatically tracks the state of queried entities and then detects any changes made to these entities when SaveChanges is called.</span></span> <span data-ttu-id="7fee9-108">[EF Core 更改跟踪中](xref:core/change-tracking/index)介绍了这种方法。</span><span class="sxs-lookup"><span data-stu-id="7fee9-108">This approach is covered in [Change Tracking in EF Core](xref:core/change-tracking/index).</span></span>

> [!TIP]
> <span data-ttu-id="7fee9-109">本文档假设了解 EF Core 更改跟踪的实体状态和基础知识。</span><span class="sxs-lookup"><span data-stu-id="7fee9-109">This document assumes that entity states and the basics of EF Core change tracking are understood.</span></span> <span data-ttu-id="7fee9-110">有关这些主题的详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-110">See [Change Tracking in EF Core](xref:core/change-tracking/index) for more information on these topics.</span></span>

> [!TIP]
> <span data-ttu-id="7fee9-111">[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangeTrackingInEFCore)，你可以运行并调试到本文档中的所有代码。</span><span class="sxs-lookup"><span data-stu-id="7fee9-111">You can run and debug into all the code in this document by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangeTrackingInEFCore).</span></span>

> [!TIP]
> <span data-ttu-id="7fee9-112">为简单起见，本文档使用和引用同步方法（如 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A>），而不是它们的异步等效方法（如 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A>）。</span><span class="sxs-lookup"><span data-stu-id="7fee9-112">For simplicity, this document uses and references synchronous methods such as <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> rather their async equivalents such as <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A>.</span></span> <span data-ttu-id="7fee9-113">除非另有说明，否则可以替换调用并等待异步方法。</span><span class="sxs-lookup"><span data-stu-id="7fee9-113">Calling and awaiting the async method can be substituted unless otherwise noted.</span></span>

## <a name="introduction"></a><span data-ttu-id="7fee9-114">简介</span><span class="sxs-lookup"><span data-stu-id="7fee9-114">Introduction</span></span>

<span data-ttu-id="7fee9-115">实体可以显式 "附加" 到，以便 <xref:Microsoft.EntityFrameworkCore.DbContext> 上下文随后跟踪这些实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-115">Entities can be explicitly "attached" to a <xref:Microsoft.EntityFrameworkCore.DbContext> such that the context then tracks those entities.</span></span> <span data-ttu-id="7fee9-116">这主要适用于以下情况：</span><span class="sxs-lookup"><span data-stu-id="7fee9-116">This is primarily useful when:</span></span>

1. <span data-ttu-id="7fee9-117">创建将插入到数据库中的新实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-117">Creating new entities that will be inserted into the database.</span></span>
2. <span data-ttu-id="7fee9-118">重新附加先前由 _其他_ DbContext 实例查询的已断开连接的实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-118">Re-attaching disconnected entities that were previously queried by a _different_ DbContext instance.</span></span>

<span data-ttu-id="7fee9-119">大多数应用程序都需要其中的第一个应用程序，并且主要由方法进行处理 <xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-119">The first of these will be needed by most applications, and is primary handled by the <xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType> methods.</span></span>

<span data-ttu-id="7fee9-120">第二个只是在 **_未跟踪实体时_** 更改实体或其关系的应用程序所必需的。</span><span class="sxs-lookup"><span data-stu-id="7fee9-120">The second is only needed by applications that change entities or their relationships **_while the entities are not being tracked_**.</span></span> <span data-ttu-id="7fee9-121">例如，web 应用程序可能会将实体发送到 web 客户端，用户在该客户端进行更改并将其发送回。</span><span class="sxs-lookup"><span data-stu-id="7fee9-121">For example, a web application may send entities to the web client where the user makes changes and sends the entities back.</span></span> <span data-ttu-id="7fee9-122">这些实体被称为 "断开连接"，因为最初从 DbContext 查询它们，但在发送到客户端时，这些实体会与该上下文断开连接。</span><span class="sxs-lookup"><span data-stu-id="7fee9-122">These entities are referred to as "disconnected" since they were originally queried from a DbContext, but were then disconnected from that context when sent to the client.</span></span>

<span data-ttu-id="7fee9-123">Web 应用程序现在必须重新附加这些实体，以便再次跟踪这些实体，并指示已进行的更改，以便对 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 数据库进行相应的更新。</span><span class="sxs-lookup"><span data-stu-id="7fee9-123">The web application must now re-attach these entities so that they are again tracked and indicate the changes that have been made such that <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> can make appropriate updates to the database.</span></span> <span data-ttu-id="7fee9-124">这主要由 <xref:Microsoft.EntityFrameworkCore.DbContext.Attach%2A?displayProperty=nameWithType> 和方法来处理 <xref:Microsoft.EntityFrameworkCore.DbContext.Update%2A?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-124">This is primarily handled by the <xref:Microsoft.EntityFrameworkCore.DbContext.Attach%2A?displayProperty=nameWithType> and <xref:Microsoft.EntityFrameworkCore.DbContext.Update%2A?displayProperty=nameWithType> methods.</span></span>

> [!TIP]
> <span data-ttu-id="7fee9-125">通常不需要将实体附加到从中查询它们的 _同一 DbContext 实例_ 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-125">Attaching entities to the _same DbContext instance_ that they were queried from should not normally be needed.</span></span> <span data-ttu-id="7fee9-126">不要定期执行无跟踪查询，然后将返回的实体附加到同一个上下文。</span><span class="sxs-lookup"><span data-stu-id="7fee9-126">Do not routinely perform a no-tracking query and then attach the returned entities to the same context.</span></span> <span data-ttu-id="7fee9-127">这会比使用跟踪查询慢，并且可能还会导致诸如缺少影子属性值之类的问题，从而使其更难正确地使用。</span><span class="sxs-lookup"><span data-stu-id="7fee9-127">This will be slower than using a tracking query, and may also result in issues such as missing shadow property values, making it harder to get right.</span></span>

### <a name="generated-versus-explicit-key-values"></a><span data-ttu-id="7fee9-128">生成的值与显式键值</span><span class="sxs-lookup"><span data-stu-id="7fee9-128">Generated versus explicit key values</span></span>

<span data-ttu-id="7fee9-129">默认情况下，integer 和 GUID [键属性](xref:core/modeling/keys) 配置为使用 [自动生成的键值](xref:core/modeling/generated-properties)。</span><span class="sxs-lookup"><span data-stu-id="7fee9-129">By default, integer and GUID [key properties](xref:core/modeling/keys) are configured to use [automatically generated key values](xref:core/modeling/generated-properties).</span></span> <span data-ttu-id="7fee9-130">这 **对于更改跟踪具有主要优势：未设置的键值指示该实体为 "new"**。</span><span class="sxs-lookup"><span data-stu-id="7fee9-130">This has a **major advantage for change tracking: an unset key value indicates that the entity is "new"**.</span></span> <span data-ttu-id="7fee9-131">通过 "新"，我们表示尚未将其插入到数据库中。</span><span class="sxs-lookup"><span data-stu-id="7fee9-131">By "new", we mean that it has not yet been inserted into the database.</span></span>

<span data-ttu-id="7fee9-132">以下各节将使用两个模型。</span><span class="sxs-lookup"><span data-stu-id="7fee9-132">Two models are used in the following sections.</span></span> <span data-ttu-id="7fee9-133">第一个配置为 **不** 使用生成的键值：</span><span class="sxs-lookup"><span data-stu-id="7fee9-133">The first is configured to **not** use generated key values:</span></span>

<!--
    public class Blog
    {
        [DatabaseGenerated(DatabaseGeneratedOption.None)]
        public int Id { get; set; }

        public string Name { get; set; }

        public IList<Post> Posts { get; } = new List<Post>();
    }

    public class Post
    {
        [DatabaseGenerated(DatabaseGeneratedOption.None)]
        public int Id { get; set; }

        public string Title { get; set; }
        public string Content { get; set; }

        public int? BlogId { get; set; }
        public Blog Blog { get; set; }
    }
-->
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Model)]

<span data-ttu-id="7fee9-134">在每个示例中，将在每个示例中首先显示非生成的 (，即显式设置) 键值，因为所有内容都非常明确且易于理解。</span><span class="sxs-lookup"><span data-stu-id="7fee9-134">Non-generated (i.e. explicitly set) key values are shown first in each example because everything is very explicit and easy to follow.</span></span> <span data-ttu-id="7fee9-135">然后，在此示例中，将使用生成的键值：</span><span class="sxs-lookup"><span data-stu-id="7fee9-135">This is then followed by an example where generated key values are used:</span></span>

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

<span data-ttu-id="7fee9-136">请注意，此模型中的键属性不需要进行额外配置，因为使用生成的键值是 [简单整数键的默认](xref:core/modeling/generated-properties)值。</span><span class="sxs-lookup"><span data-stu-id="7fee9-136">Notice that the key properties in this model need no additional configuration here since using generated key values is the [default for simple integer keys](xref:core/modeling/generated-properties).</span></span>

## <a name="inserting-new-entities"></a><span data-ttu-id="7fee9-137">插入新实体</span><span class="sxs-lookup"><span data-stu-id="7fee9-137">Inserting new entities</span></span>

### <a name="explicit-key-values"></a><span data-ttu-id="7fee9-138">显式键值</span><span class="sxs-lookup"><span data-stu-id="7fee9-138">Explicit key values</span></span>

<span data-ttu-id="7fee9-139">必须在 `Added` 要插入的状态中跟踪实体 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-139">An entity must be tracked in the `Added` state to be inserted by <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A>.</span></span> <span data-ttu-id="7fee9-140">实体通常通过调用、、、 <xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType> <xref:Microsoft.EntityFrameworkCore.DbContext.AddRange%2A?displayProperty=nameWithType> 或中 <xref:Microsoft.EntityFrameworkCore.DbContext.AddAsync%2A?displayProperty=nameWithType> <xref:Microsoft.EntityFrameworkCore.DbContext.AddRangeAsync%2A?displayProperty=nameWithType> 的等效方法 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 之一置于已添加状态。</span><span class="sxs-lookup"><span data-stu-id="7fee9-140">Entities are typically put in the Added state by calling one of <xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType>, <xref:Microsoft.EntityFrameworkCore.DbContext.AddRange%2A?displayProperty=nameWithType>, <xref:Microsoft.EntityFrameworkCore.DbContext.AddAsync%2A?displayProperty=nameWithType>, <xref:Microsoft.EntityFrameworkCore.DbContext.AddRangeAsync%2A?displayProperty=nameWithType>, or the equivalent methods on <xref:Microsoft.EntityFrameworkCore.DbSet%601>.</span></span>

> [!TIP]
> <span data-ttu-id="7fee9-141">这些方法在更改跟踪上下文中的工作方式相同。</span><span class="sxs-lookup"><span data-stu-id="7fee9-141">These methods all work in the same way in the context of change tracking.</span></span> <span data-ttu-id="7fee9-142">有关详细信息，请参阅 [其他更改跟踪功能](xref:core/change-tracking/miscellaneous) 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-142">See [Additional Change Tracking Features](xref:core/change-tracking/miscellaneous) for more information.</span></span>

<span data-ttu-id="7fee9-143">例如，若要开始跟踪新博客：</span><span class="sxs-lookup"><span data-stu-id="7fee9-143">For example, to start tracking a new blog:</span></span>

<!--
            context.Add(
                new Blog
                {
                    Id = 1,
                    Name = ".NET Blog",
                });
-->
[!code-csharp[Inserting_new_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Inserting_new_entities_1)]

<span data-ttu-id="7fee9-144">检查此调用后的 [更改跟踪器调试视图](xref:core/change-tracking/debug-views) 显示上下文正在跟踪状态中的新实体 `Added` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-144">Inspecting the [change tracker debug view](xref:core/change-tracking/debug-views) following this call shows that the context is tracking the new entity in the `Added` state:</span></span>

```output
Blog {Id: 1} Added
  Id: 1 PK
  Name: '.NET Blog'
  Posts: []
```

<span data-ttu-id="7fee9-145">但是，Add 方法并不只是处理单个实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-145">However, the Add methods don't just work on an individual entity.</span></span> <span data-ttu-id="7fee9-146">它们实际开始跟踪 _相关实体的整个关系图_，并将其全部置于 `Added` 状态。</span><span class="sxs-lookup"><span data-stu-id="7fee9-146">They actually start tracking an _entire graph of related entities_, putting them all to the `Added` state.</span></span> <span data-ttu-id="7fee9-147">例如，若要插入新的博客和关联的新文章，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="7fee9-147">For example, to insert a new blog and associated new posts:</span></span>

<!--
            context.Add(
                new Blog
                {
                    Id = 1,
                    Name = ".NET Blog",
                    Posts =
                    {
                        new Post
                        {
                            Id = 1,
                            Title = "Announcing the Release of EF Core 5.0",
                            Content = "Announcing the release of EF Core 5.0, a full featured cross-platform..."
                        },
                        new Post
                        {
                            Id = 2,
                            Title = "Announcing F# 5",
                            Content = "F# 5 is the latest version of F#, the functional programming language..."
                        }
                    }
                });
-->
[!code-csharp[Inserting_new_entities_2](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Inserting_new_entities_2)]

<span data-ttu-id="7fee9-148">上下文现在正在跟踪所有这些实体，如下所示 `Added` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-148">The context is now tracking all these entities as `Added`:</span></span>

```output
Blog {Id: 1} Added
  Id: 1 PK
  Name: '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}]
Post {Id: 1} Added
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
Post {Id: 2} Added
  Id: 2 PK
  BlogId: 1 FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: {Id: 1}
```

<span data-ttu-id="7fee9-149">请注意，已为 `Id` 上述示例中的键属性设置了显式值。</span><span class="sxs-lookup"><span data-stu-id="7fee9-149">Notice that explicit values have been set for the `Id` key properties in the examples above.</span></span> <span data-ttu-id="7fee9-150">这是因为，此处的模型已配置为使用显式设置的键值，而不是自动生成的键值。</span><span class="sxs-lookup"><span data-stu-id="7fee9-150">This is because the model here has been configured to use explicitly set key values, rather than automatically generated key values.</span></span> <span data-ttu-id="7fee9-151">在不使用生成的键时，必须在调用 _之前_ 显式设置键属性 `Add` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-151">When not using generated keys, the key properties must be explicitly set _before_ calling `Add`.</span></span> <span data-ttu-id="7fee9-152">然后，在调用 SaveChanges 时插入这些键值。</span><span class="sxs-lookup"><span data-stu-id="7fee9-152">These key values are then inserted when SaveChanges is called.</span></span> <span data-ttu-id="7fee9-153">例如，使用 SQLite 时：</span><span class="sxs-lookup"><span data-stu-id="7fee9-153">For example, when using SQLite:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='1' (DbType = String), @p1='.NET Blog' (Size = 9)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Blogs" ("Id", "Name")
VALUES (@p0, @p1);

-- Executed DbCommand (0ms) [Parameters=[@p2='1' (DbType = String), @p3='1' (DbType = String), @p4='Announcing the release of EF Core 5.0, a full featured cross-platform...' (Size = 72), @p5='Announcing the Release of EF Core 5.0' (Size = 37)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Posts" ("Id", "BlogId", "Content", "Title")
VALUES (@p2, @p3, @p4, @p5);

-- Executed DbCommand (0ms) [Parameters=[@p0='2' (DbType = String), @p1='1' (DbType = String), @p2='F# 5 is the latest version of F#, the functional programming language...' (Size = 72), @p3='Announcing F# 5' (Size = 15)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Posts" ("Id", "BlogId", "Content", "Title")
VALUES (@p0, @p1, @p2, @p3);
```

<span data-ttu-id="7fee9-154">在 SaveChanges 完成后，所有这些实体都将在状态中跟踪 `Unchanged` ，因为这些实体现在存在于数据库中：</span><span class="sxs-lookup"><span data-stu-id="7fee9-154">All of these entities are tracked in the `Unchanged` state after SaveChanges completes, since these entities now exist in the database:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}]
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

### <a name="generated-key-values"></a><span data-ttu-id="7fee9-155">生成的键值</span><span class="sxs-lookup"><span data-stu-id="7fee9-155">Generated key values</span></span>

<span data-ttu-id="7fee9-156">如上所述，默认情况下，integer 和 GUID [键属性](xref:core/modeling/keys) 配置为使用 [自动生成的键值](xref:core/modeling/generated-properties) 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-156">As mentioned above, integer and GUID [key properties](xref:core/modeling/keys) are configured to use [automatically generated key values](xref:core/modeling/generated-properties) by default.</span></span> <span data-ttu-id="7fee9-157">这意味着应用程序 _不能显式设置任何键值_。</span><span class="sxs-lookup"><span data-stu-id="7fee9-157">This means that the application _must not set any key value explicitly_.</span></span> <span data-ttu-id="7fee9-158">例如，若要插入新的博客并使用生成的键值发送所有内容，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="7fee9-158">For example, to insert a new blog and posts all with generated key values:</span></span>

<!--
            context.Add(
                new Blog
                {
                    Name = ".NET Blog",
                    Posts =
                    {
                        new Post
                        {
                            Title = "Announcing the Release of EF Core 5.0",
                            Content = "Announcing the release of EF Core 5.0, a full featured cross-platform..."
                        },
                        new Post
                        {
                            Title = "Announcing F# 5",
                            Content = "F# 5 is the latest version of F#, the functional programming language..."
                        }
                    }
                });
-->
[!code-csharp[Inserting_new_entities_3](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/GeneratedKeysSamples.cs?name=Inserting_new_entities_3)]

<span data-ttu-id="7fee9-159">对于显式键值，上下文现在正在跟踪所有这些实体，如下所示 `Added` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-159">As with explicit key values, the context is now tracking all these entities as `Added`:</span></span>

```output
Blog {Id: -2147482644} Added
  Id: -2147482644 PK Temporary
  Name: '.NET Blog'
  Posts: [{Id: -2147482637}, {Id: -2147482636}]
Post {Id: -2147482637} Added
  Id: -2147482637 PK Temporary
  BlogId: -2147482644 FK Temporary
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: -2147482644}
Post {Id: -2147482636} Added
  Id: -2147482636 PK Temporary
  BlogId: -2147482644 FK Temporary
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: {Id: -2147482644}
```

<span data-ttu-id="7fee9-160">请注意，在这种情况下，已为每个实体生成了 [临时键值](xref:core/change-tracking/miscellaneous#temporary-values) 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-160">Notice in this case that [temporary key values](xref:core/change-tracking/miscellaneous#temporary-values) have been generated for each entity.</span></span> <span data-ttu-id="7fee9-161">在调用 SaveChanges 之前，EF Core 会使用这些值，此时将从数据库中读回实际键值。</span><span class="sxs-lookup"><span data-stu-id="7fee9-161">These values are used by EF Core until SaveChanges is called, at which point real key values are read back from the database.</span></span> <span data-ttu-id="7fee9-162">例如，使用 SQLite 时：</span><span class="sxs-lookup"><span data-stu-id="7fee9-162">For example, when using SQLite:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='.NET Blog' (Size = 9)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Blogs" ("Name")
VALUES (@p0);
SELECT "Id"
FROM "Blogs"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();

-- Executed DbCommand (0ms) [Parameters=[@p1='1' (DbType = String), @p2='Announcing the release of EF Core 5.0, a full featured cross-platform...' (Size = 72), @p3='Announcing the Release of EF Core 5.0' (Size = 37)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Posts" ("BlogId", "Content", "Title")
VALUES (@p1, @p2, @p3);
SELECT "Id"
FROM "Posts"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();

-- Executed DbCommand (0ms) [Parameters=[@p0='1' (DbType = String), @p1='F# 5 is the latest version of F#, the functional programming language...' (Size = 72), @p2='Announcing F# 5' (Size = 15)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Posts" ("BlogId", "Content", "Title")
VALUES (@p0, @p1, @p2);
SELECT "Id"
FROM "Posts"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();
```

<span data-ttu-id="7fee9-163">SaveChanges 完成后，所有实体都已使用其真实密钥值进行了更新，并在状态下进行跟踪， `Unchanged` 因为它们现在与数据库中的状态匹配：</span><span class="sxs-lookup"><span data-stu-id="7fee9-163">After SaveChanges completes, all of the entities have been updated with their real key values and are tracked in the `Unchanged` state since they now match the state in the database:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}]
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

<span data-ttu-id="7fee9-164">这与上一个使用显式键值的示例的最终状态完全相同。</span><span class="sxs-lookup"><span data-stu-id="7fee9-164">This is exactly the same end-state as the previous example that used explicit key values.</span></span>

> [!TIP]
> <span data-ttu-id="7fee9-165">即使使用生成的键值，也仍可以设置显式键值。</span><span class="sxs-lookup"><span data-stu-id="7fee9-165">An explicit key value can still be set even when using generated key values.</span></span> <span data-ttu-id="7fee9-166">然后 EF Core 将尝试使用此键值进行插入。</span><span class="sxs-lookup"><span data-stu-id="7fee9-166">EF Core will then attempt to insert using this key value.</span></span> <span data-ttu-id="7fee9-167">某些数据库配置（包括带有标识列的 SQL Server）不支持此类插入，并将引发 ([请参阅这些文档，了解解决方法](xref:core/providers/sql-server/value-generation#inserting-explicit-values-into-identity-columns)) 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-167">Some database configurations, including SQL Server with Identity columns, do not support such inserts and will throw ([see these docs for a workaround](xref:core/providers/sql-server/value-generation#inserting-explicit-values-into-identity-columns)).</span></span>

## <a name="attaching-existing-entities"></a><span data-ttu-id="7fee9-168">附加现有实体</span><span class="sxs-lookup"><span data-stu-id="7fee9-168">Attaching existing entities</span></span>

### <a name="explicit-key-values"></a><span data-ttu-id="7fee9-169">显式键值</span><span class="sxs-lookup"><span data-stu-id="7fee9-169">Explicit key values</span></span>

<span data-ttu-id="7fee9-170">从查询返回的实体将在状态中跟踪 `Unchanged` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-170">Entities returned from queries are tracked in the `Unchanged` state.</span></span> <span data-ttu-id="7fee9-171">`Unchanged`状态表示该实体自查询后未进行修改。</span><span class="sxs-lookup"><span data-stu-id="7fee9-171">The `Unchanged` state means that the entity has not been modified since it was queried.</span></span> <span data-ttu-id="7fee9-172">断开连接的实体（可能从 HTTP 请求中的 web 客户端返回）可以使用 <xref:Microsoft.EntityFrameworkCore.DbContext.Attach%2A?displayProperty=nameWithType> 、 <xref:Microsoft.EntityFrameworkCore.DbContext.AttachRange%2A?displayProperty=nameWithType> 或中的等效方法进入此状态 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-172">A disconnected entity, perhaps returned from a web client in an HTTP request, can be put into this state using either <xref:Microsoft.EntityFrameworkCore.DbContext.Attach%2A?displayProperty=nameWithType>, <xref:Microsoft.EntityFrameworkCore.DbContext.AttachRange%2A?displayProperty=nameWithType>, or the equivalent methods on <xref:Microsoft.EntityFrameworkCore.DbSet%601>.</span></span> <span data-ttu-id="7fee9-173">例如，若要开始跟踪现有博客：</span><span class="sxs-lookup"><span data-stu-id="7fee9-173">For example, to start tracking an existing blog:</span></span>

<!--
            context.Attach(
                new Blog
                {
                    Id = 1,
                    Name = ".NET Blog",
                });
-->
[!code-csharp[Attaching_existing_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Attaching_existing_entities_1)]

> [!NOTE]
> <span data-ttu-id="7fee9-174">此处的示例是为了简单起见，显式创建实体 `new` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-174">The examples here are creating entities explicitly with `new` for simplicity.</span></span> <span data-ttu-id="7fee9-175">通常情况下，实体实例将来自其他源，如从客户端反序列化，或者是从 HTTP Post 中的数据创建的。</span><span class="sxs-lookup"><span data-stu-id="7fee9-175">Normally the entity instances will have come from another source, such as being deserialized from a client, or being created from data in an HTTP Post.</span></span>

<span data-ttu-id="7fee9-176">检查此调用后的 [更改跟踪](xref:core/change-tracking/debug-views) 器 "调试" 视图会显示在状态下跟踪实体 `Unchanged` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-176">Inspecting the [change tracker debug view](xref:core/change-tracking/debug-views) following this call shows that the entity is tracked in the `Unchanged` state:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Posts: []
```

<span data-ttu-id="7fee9-177">就像一样 `Add` ， `Attach` 实际将连接的实体的整个关系图设置为 `Unchanged` 状态。</span><span class="sxs-lookup"><span data-stu-id="7fee9-177">Just like `Add`, `Attach` actually sets an entire graph of connected entities to the `Unchanged` state.</span></span> <span data-ttu-id="7fee9-178">例如，要附加现有博客和关联的现有文章：</span><span class="sxs-lookup"><span data-stu-id="7fee9-178">For example, to attach an existing blog and associated existing posts:</span></span>

<!--
            context.Attach(
                new Blog
                {
                    Id = 1,
                    Name = ".NET Blog",
                    Posts =
                    {
                        new Post
                        {
                            Id = 1,
                            Title = "Announcing the Release of EF Core 5.0",
                            Content = "Announcing the release of EF Core 5.0, a full featured cross-platform..."
                        },
                        new Post
                        {
                            Id = 2,
                            Title = "Announcing F# 5",
                            Content = "F# 5 is the latest version of F#, the functional programming language..."
                        }
                    }
                });
-->
[!code-csharp[Attaching_existing_entities_2](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Attaching_existing_entities_2)]

<span data-ttu-id="7fee9-179">上下文现在正在跟踪所有这些实体，如下所示 `Unchanged` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-179">The context is now tracking all these entities as `Unchanged`:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}]
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

<span data-ttu-id="7fee9-180">此时调用 SaveChanges 将不起作用。</span><span class="sxs-lookup"><span data-stu-id="7fee9-180">Calling SaveChanges at this point will have no effect.</span></span> <span data-ttu-id="7fee9-181">所有实体都标记为 `Unchanged` ，因此数据库中没有要更新的内容。</span><span class="sxs-lookup"><span data-stu-id="7fee9-181">All the entities are marked as `Unchanged`, so there is nothing to update in the database.</span></span>

### <a name="generated-key-values"></a><span data-ttu-id="7fee9-182">生成的键值</span><span class="sxs-lookup"><span data-stu-id="7fee9-182">Generated key values</span></span>

<span data-ttu-id="7fee9-183">如上所述，默认情况下，integer 和 GUID [键属性](xref:core/modeling/keys) 配置为使用 [自动生成的键值](xref:core/modeling/generated-properties) 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-183">As mentioned above, integer and GUID [key properties](xref:core/modeling/keys) are configured to use [automatically generated key values](xref:core/modeling/generated-properties) by default.</span></span> <span data-ttu-id="7fee9-184">当使用断开连接的实体时，这有一个主要优势：未设置的键值指示该实体尚未插入到数据库中。</span><span class="sxs-lookup"><span data-stu-id="7fee9-184">This has a major advantage when working with disconnected entities: an unset key value indicates that the entity has not yet been inserted into the database.</span></span> <span data-ttu-id="7fee9-185">这允许更改跟踪器自动检测新实体并使其处于 `Added` 状态。</span><span class="sxs-lookup"><span data-stu-id="7fee9-185">This allows the change tracker to automatically detect new entities and put them in the `Added` state.</span></span> <span data-ttu-id="7fee9-186">例如，请考虑附加博客和文章的此图：</span><span class="sxs-lookup"><span data-stu-id="7fee9-186">For example, consider attaching this graph of a blog and posts:</span></span>

<!--
            context.Attach(
                new Blog
                {
                    Id = 1,
                    Name = ".NET Blog",
                    Posts =
                    {
                        new Post
                        {
                            Id = 1,
                            Title = "Announcing the Release of EF Core 5.0",
                            Content = "Announcing the release of EF Core 5.0, a full featured cross-platform..."
                        },
                        new Post
                        {
                            Id = 2,
                            Title = "Announcing F# 5",
                            Content = "F# 5 is the latest version of F#, the functional programming language..."
                        },
                        new Post
                        {
                            Title = "Announcing .NET 5.0",
                            Content = ".NET 5.0 includes many enhancements, including single file applications, more..."
                        },
                    }
                });
-->
[!code-csharp[Attaching_existing_entities_3](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/GeneratedKeysSamples.cs?name=Attaching_existing_entities_3)]

<span data-ttu-id="7fee9-187">博客的键值为1，表示它已存在于数据库中。</span><span class="sxs-lookup"><span data-stu-id="7fee9-187">The blog has a key value of 1, indicating that it already exists in the database.</span></span> <span data-ttu-id="7fee9-188">两篇文章中的键值也已设置，但第三篇文章并未设置。</span><span class="sxs-lookup"><span data-stu-id="7fee9-188">Two of the posts also have key values set, but the third does not.</span></span> <span data-ttu-id="7fee9-189">EF Core 会将此键值视为0，这是一个整数的 CLR 默认值。</span><span class="sxs-lookup"><span data-stu-id="7fee9-189">EF Core will see this key value as 0, the CLR default for an integer.</span></span> <span data-ttu-id="7fee9-190">这会导致 EF Core 将新实体标记为 `Added` 而不是 `Unchanged` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-190">This results in EF Core marking the new entity as `Added` instead of `Unchanged`:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}, {Id: -2147482636}]
Post {Id: -2147482636} Added
  Id: -2147482636 PK Temporary
  BlogId: 1 FK
  Content: '.NET 5.0 includes many enhancements, including single file a...'
  Title: 'Announcing .NET 5.0'
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
```

<span data-ttu-id="7fee9-191">此时调用 SaveChanges 不会对实体执行任何操作 `Unchanged` ，但会将新实体插入到数据库中。</span><span class="sxs-lookup"><span data-stu-id="7fee9-191">Calling SaveChanges at this point does nothing with the `Unchanged` entities, but inserts the new entity into the database.</span></span> <span data-ttu-id="7fee9-192">例如，使用 SQLite 时：</span><span class="sxs-lookup"><span data-stu-id="7fee9-192">For example, when using SQLite:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='1' (DbType = String), @p1='.NET 5.0 includes many enhancements, including single file applications, more...' (Size = 80), @p2='Announcing .NET 5.0' (Size = 19)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Posts" ("BlogId", "Content", "Title")
VALUES (@p0, @p1, @p2);
SELECT "Id"
FROM "Posts"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();
```

<span data-ttu-id="7fee9-193">此处需要注意的要点是，使用生成的键值，EF Core 能够 **自动区分断开连接图形中的现有实体**。</span><span class="sxs-lookup"><span data-stu-id="7fee9-193">The important point to notice here is that, with generated key values, EF Core is able to **automatically distinguish new from existing entities in a disconnected graph**.</span></span> <span data-ttu-id="7fee9-194">简而言之，使用生成的密钥时，当实体没有设置键值时，EF Core 将始终插入实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-194">In a nutshell, when using generated keys, EF Core will always insert an entity when that entity has no key value set.</span></span>

## <a name="updating-existing-entities"></a><span data-ttu-id="7fee9-195">更新现有实体</span><span class="sxs-lookup"><span data-stu-id="7fee9-195">Updating existing entities</span></span>

### <a name="explicit-key-values"></a><span data-ttu-id="7fee9-196">显式键值</span><span class="sxs-lookup"><span data-stu-id="7fee9-196">Explicit key values</span></span>

<span data-ttu-id="7fee9-197"><xref:Microsoft.EntityFrameworkCore.DbContext.Update%2A?displayProperty=nameWithType>、 <xref:Microsoft.EntityFrameworkCore.DbContext.UpdateRange%2A?displayProperty=nameWithType> 和的等效方法的 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 行为与 `Attach` 上述方法完全相同，只是将实体放入 `Modfied` 而不是 `Unchanged` 状态。</span><span class="sxs-lookup"><span data-stu-id="7fee9-197"><xref:Microsoft.EntityFrameworkCore.DbContext.Update%2A?displayProperty=nameWithType>, <xref:Microsoft.EntityFrameworkCore.DbContext.UpdateRange%2A?displayProperty=nameWithType>, and the equivalent methods on <xref:Microsoft.EntityFrameworkCore.DbSet%601> behave exactly as the `Attach` methods described above, except that entities are put into the `Modfied` instead of the `Unchanged` state.</span></span> <span data-ttu-id="7fee9-198">例如，若要开始跟踪现有博客，请 `Modified` 执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="7fee9-198">For example, to start tracking an existing blog as `Modified`:</span></span>

<!--
            context.Update(
                new Blog
                {
                    Id = 1,
                    Name = ".NET Blog",
                });
-->
[!code-csharp[Updating_existing_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Updating_existing_entities_1)]

<span data-ttu-id="7fee9-199">检查此调用后的 [更改跟踪器调试视图](xref:core/change-tracking/debug-views) 表明上下文正在跟踪此实体的 `Modified` 状态：</span><span class="sxs-lookup"><span data-stu-id="7fee9-199">Inspecting the [change tracker debug view](xref:core/change-tracking/debug-views) following this call shows that the context is tracking this entity in the `Modified` state:</span></span>

```output
Blog {Id: 1} Modified
  Id: 1 PK
  Name: '.NET Blog' Modified
  Posts: []
```

<span data-ttu-id="7fee9-200">与和一样 `Add` `Attach` ， `Update` 实际上将相关实体的 _整个关系图_ 标记为 `Modified` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-200">Just like with `Add` and `Attach`, `Update` actually marks an _entire graph_ of related entities as `Modified`.</span></span> <span data-ttu-id="7fee9-201">例如，要将现有博客和关联的现有帖子附加为 `Modified` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-201">For example, to attach an existing blog and associated existing posts as `Modified`:</span></span>

<!--
            context.Update(
                new Blog
                {
                    Id = 1,
                    Name = ".NET Blog",
                    Posts =
                    {
                        new Post
                        {
                            Id = 1,
                            Title = "Announcing the Release of EF Core 5.0",
                            Content = "Announcing the release of EF Core 5.0, a full featured cross-platform..."
                        },
                        new Post
                        {
                            Id = 2,
                            Title = "Announcing F# 5",
                            Content = "F# 5 is the latest version of F#, the functional programming language..."
                        }
                    }
                });
-->
[!code-csharp[Updating_existing_entities_2](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Updating_existing_entities_2)]

<span data-ttu-id="7fee9-202">上下文现在正在跟踪所有这些实体，如下所示 `Modified` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-202">The context is now tracking all these entities as `Modified`:</span></span>

```output
Blog {Id: 1} Modified
  Id: 1 PK
  Name: '.NET Blog' Modified
  Posts: [{Id: 1}, {Id: 2}]
Post {Id: 1} Modified
  Id: 1 PK
  BlogId: 1 FK Modified Originally <null>
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...' Modified
  Title: 'Announcing the Release of EF Core 5.0' Modified
  Blog: {Id: 1}
Post {Id: 2} Modified
  Id: 2 PK
  BlogId: 1 FK Modified Originally <null>
  Content: 'F# 5 is the latest version of F#, the functional programming...' Modified
  Title: 'Announcing F# 5' Modified
  Blog: {Id: 1}
```

<span data-ttu-id="7fee9-203">此时调用 SaveChanges 将导致为所有这些实体将更新发送到数据库。</span><span class="sxs-lookup"><span data-stu-id="7fee9-203">Calling SaveChanges at this point will cause updates to be sent to the database for all these entities.</span></span> <span data-ttu-id="7fee9-204">例如，使用 SQLite 时：</span><span class="sxs-lookup"><span data-stu-id="7fee9-204">For example, when using SQLite:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p1='1' (DbType = String), @p0='.NET Blog' (Size = 9)], CommandType='Text', CommandTimeout='30']
UPDATE "Blogs" SET "Name" = @p0
WHERE "Id" = @p1;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p3='1' (DbType = String), @p0='1' (DbType = String), @p1='Announcing the release of EF Core 5.0, a full featured cross-platform...' (Size = 72), @p2='Announcing the Release of EF Core 5.0' (Size = 37)], CommandType='Text', CommandTimeout='30']
UPDATE "Posts" SET "BlogId" = @p0, "Content" = @p1, "Title" = @p2
WHERE "Id" = @p3;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p3='2' (DbType = String), @p0='1' (DbType = String), @p1='F# 5 is the latest version of F#, the functional programming language...' (Size = 72), @p2='Announcing F# 5' (Size = 15)], CommandType='Text', CommandTimeout='30']
UPDATE "Posts" SET "BlogId" = @p0, "Content" = @p1, "Title" = @p2
WHERE "Id" = @p3;
SELECT changes();
```

### <a name="generated-key-values"></a><span data-ttu-id="7fee9-205">生成的键值</span><span class="sxs-lookup"><span data-stu-id="7fee9-205">Generated key values</span></span>

<span data-ttu-id="7fee9-206">与一样 `Attach` ，生成的键值与相同的主要优点是 `Update` ：未设置的键值指示该实体是新实体，尚未插入到数据库中。</span><span class="sxs-lookup"><span data-stu-id="7fee9-206">As with `Attach`, generated key values have the same major benefit for `Update`: an unset key value indicates that the entity is new and has not yet been inserted into the database.</span></span> <span data-ttu-id="7fee9-207">与一样 `Attach` ，这允许 DbContext 自动检测新实体并使其处于 `Added` 状态。</span><span class="sxs-lookup"><span data-stu-id="7fee9-207">As with `Attach`, this allows the DbContext to automatically detect new entities and put them in the `Added` state.</span></span> <span data-ttu-id="7fee9-208">例如，请考虑 `Update` 使用博客和文章的此图调用：</span><span class="sxs-lookup"><span data-stu-id="7fee9-208">For example, consider calling `Update` with this graph of a blog and posts:</span></span>

<!--
            context.Update(
                new Blog
                {
                    Id = 1,
                    Name = ".NET Blog",
                    Posts =
                    {
                        new Post
                        {
                            Id = 1,
                            Title = "Announcing the Release of EF Core 5.0",
                            Content = "Announcing the release of EF Core 5.0, a full featured cross-platform..."
                        },
                        new Post
                        {
                            Id = 2,
                            Title = "Announcing F# 5",
                            Content = "F# 5 is the latest version of F#, the functional programming language..."
                        }
                    }
                });
-->
[!code-csharp[Updating_existing_entities_3](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/GeneratedKeysSamples.cs?name=Updating_existing_entities_3)]

<span data-ttu-id="7fee9-209">与示例一样 `Attach` ，不包含键值的 post 被检测为 new，并设置为 `Added` 状态。</span><span class="sxs-lookup"><span data-stu-id="7fee9-209">As with the `Attach` example, the post with no key value is detected as new and set to the `Added` state.</span></span> <span data-ttu-id="7fee9-210">其他实体标记为 `Modified` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-210">The other entities are marked as `Modified`:</span></span>

```output
Blog {Id: 1} Modified
  Id: 1 PK
  Name: '.NET Blog' Modified
  Posts: [{Id: 1}, {Id: 2}, {Id: -2147482633}]
Post {Id: -2147482633} Added
  Id: -2147482633 PK Temporary
  BlogId: 1 FK
  Content: '.NET 5.0 includes many enhancements, including single file a...'
  Title: 'Announcing .NET 5.0'
  Blog: {Id: 1}
Post {Id: 1} Modified
  Id: 1 PK
  BlogId: 1 FK Modified Originally <null>
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...' Modified
  Title: 'Announcing the Release of EF Core 5.0' Modified
  Blog: {Id: 1}
Post {Id: 2} Modified
  Id: 2 PK
  BlogId: 1 FK Modified Originally <null>
  Content: 'F# 5 is the latest version of F#, the functional programming...' Modified
  Title: 'Announcing F# 5' Modified
  Blog: {Id: 1}
```

<span data-ttu-id="7fee9-211">`SaveChanges`此时调用将导致在插入新实体时将更新发送到数据库中的所有现有实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-211">Calling `SaveChanges` at this point will cause updates to be sent to the database for all the existing entities, while the new entity is inserted.</span></span> <span data-ttu-id="7fee9-212">例如，使用 SQLite 时：</span><span class="sxs-lookup"><span data-stu-id="7fee9-212">For example, when using SQLite:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p1='1' (DbType = String), @p0='.NET Blog' (Size = 9)], CommandType='Text', CommandTimeout='30']
UPDATE "Blogs" SET "Name" = @p0
WHERE "Id" = @p1;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p3='1' (DbType = String), @p0='1' (DbType = String), @p1='Announcing the release of EF Core 5.0, a full featured cross-platform...' (Size = 72), @p2='Announcing the Release of EF Core 5.0' (Size = 37)], CommandType='Text', CommandTimeout='30']
UPDATE "Posts" SET "BlogId" = @p0, "Content" = @p1, "Title" = @p2
WHERE "Id" = @p3;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p3='2' (DbType = String), @p0='1' (DbType = String), @p1='F# 5 is the latest version of F#, the functional programming language...' (Size = 72), @p2='Announcing F# 5' (Size = 15)], CommandType='Text', CommandTimeout='30']
UPDATE "Posts" SET "BlogId" = @p0, "Content" = @p1, "Title" = @p2
WHERE "Id" = @p3;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p0='1' (DbType = String), @p1='.NET 5.0 includes many enhancements, including single file applications, more...' (Size = 80), @p2='Announcing .NET 5.0' (Size = 19)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Posts" ("BlogId", "Content", "Title")
VALUES (@p0, @p1, @p2);
SELECT "Id"
FROM "Posts"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();
```

<span data-ttu-id="7fee9-213">这是一种从断开连接的图形生成更新和插入的非常简单的方法。</span><span class="sxs-lookup"><span data-stu-id="7fee9-213">This is a very easy way to generate updates and inserts from a disconnected graph.</span></span> <span data-ttu-id="7fee9-214">但是，这将导致为每个被跟踪实体的每个属性将更新或插入发送到数据库，即使某些属性值可能未更改也是如此。</span><span class="sxs-lookup"><span data-stu-id="7fee9-214">However, it results in updates or inserts being sent to the database for every property of every tracked entity, even when some property values may not have been changed.</span></span> <span data-ttu-id="7fee9-215">这不太恐惧;对于包含小型图形的许多应用程序，这是生成更新的一种简单且实用的方法。</span><span class="sxs-lookup"><span data-stu-id="7fee9-215">Don't be too scared by this; for many applications with small graphs, this can be an easy and pragmatic way of generating updates.</span></span> <span data-ttu-id="7fee9-216">换句话说，其他更复杂的模式有时会导致更新的更新，如 EF Core 中的 [标识解析](xref:core/change-tracking/identity-resolution)中所述。</span><span class="sxs-lookup"><span data-stu-id="7fee9-216">That being said, other more complex patterns can sometimes result in more efficient updates, as described in [Identity Resolution in EF Core](xref:core/change-tracking/identity-resolution).</span></span>

## <a name="deleting-existing-entities"></a><span data-ttu-id="7fee9-217">删除现有实体</span><span class="sxs-lookup"><span data-stu-id="7fee9-217">Deleting existing entities</span></span>

<span data-ttu-id="7fee9-218">对于要通过 SaveChanges 删除的实体，必须在状态中跟踪该实体 `Deleted` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-218">For an entity to be deleted by SaveChanges it must be tracked in the `Deleted` state.</span></span> <span data-ttu-id="7fee9-219">实体通常 `Deleted` 通过调用 <xref:Microsoft.EntityFrameworkCore.DbContext.Remove%2A?displayProperty=nameWithType> 、 <xref:Microsoft.EntityFrameworkCore.DbContext.RemoveRange%2A?displayProperty=nameWithType> 或中的等效方法之一置于状态 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-219">Entities are typically put in the `Deleted` state by calling one of <xref:Microsoft.EntityFrameworkCore.DbContext.Remove%2A?displayProperty=nameWithType>, <xref:Microsoft.EntityFrameworkCore.DbContext.RemoveRange%2A?displayProperty=nameWithType>, or the equivalent methods on <xref:Microsoft.EntityFrameworkCore.DbSet%601>.</span></span> <span data-ttu-id="7fee9-220">例如，要将现有 post 标记为 `Deleted` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-220">For example, to mark an existing post as `Deleted`:</span></span>

<!--
            context.Remove(
                new Post
                {
                    Id = 2
                });
-->
[!code-csharp[Deleting_existing_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Deleting_existing_entities_1)]

<span data-ttu-id="7fee9-221">检查此调用后的 [更改跟踪器调试视图](xref:core/change-tracking/debug-views) 表明上下文正在跟踪状态中的实体 `Deleted` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-221">Inspecting the [change tracker debug view](xref:core/change-tracking/debug-views) following this call shows that the context is tracking the entity in the `Deleted` state:</span></span>

```output
Post {Id: 2} Deleted
  Id: 2 PK
  BlogId: <null> FK
  Content: <null>
  Title: <null>
  Blog: <null>
```

<span data-ttu-id="7fee9-222">此实体将在调用 SaveChanges 时删除。</span><span class="sxs-lookup"><span data-stu-id="7fee9-222">This entity will be deleted when SaveChanges is called.</span></span> <span data-ttu-id="7fee9-223">例如，使用 SQLite 时：</span><span class="sxs-lookup"><span data-stu-id="7fee9-223">For example, when using SQLite:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='2' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Posts"
WHERE "Id" = @p0;
SELECT changes();
```

<span data-ttu-id="7fee9-224">SaveChanges 完成后，删除的实体将从 DbContext 中分离出来，因为它不再存在于数据库中。</span><span class="sxs-lookup"><span data-stu-id="7fee9-224">After SaveChanges completes, the deleted entity is detached from the DbContext since it no longer exists in the database.</span></span> <span data-ttu-id="7fee9-225">因此，调试视图为空，因为没有要跟踪的实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-225">The debug view is therefore empty because no entities are being tracked.</span></span>

## <a name="deleting-dependentchild-entities"></a><span data-ttu-id="7fee9-226">删除从属/子实体</span><span class="sxs-lookup"><span data-stu-id="7fee9-226">Deleting dependent/child entities</span></span>

<span data-ttu-id="7fee9-227">从关系图中删除从属/子实体比删除主体/父实体更直接。</span><span class="sxs-lookup"><span data-stu-id="7fee9-227">Deleting dependent/child entities from a graph is more straightforward than deleting principal/parent entities.</span></span> <span data-ttu-id="7fee9-228">有关详细信息，请参阅下一部分和 [更改外键和导航](xref:core/change-tracking/relationship-changes) 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-228">See the next section and [Changing Foreign Keys and Navigations](xref:core/change-tracking/relationship-changes) for more information.</span></span>

<span data-ttu-id="7fee9-229">对 `Remove` 使用创建的实体调用是异常的 `new` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-229">It is unusual to call `Remove` on an entity created with `new`.</span></span> <span data-ttu-id="7fee9-230">此外，与和不同，对尚未 `Add` `Attach` `Update` `Remove` 在或状态中跟踪的实体调用是罕见的 `Unchanged` `Modified` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-230">Further, unlike `Add`, `Attach` and `Update`, it is uncommon to call `Remove` on an entity that isn't already tracked in the `Unchanged` or `Modified` state.</span></span> <span data-ttu-id="7fee9-231">通常，可以跟踪相关实体的单个实体或图形，然后对 `Remove` 应删除的实体调用。</span><span class="sxs-lookup"><span data-stu-id="7fee9-231">Instead it is typical to track a single entity or graph of related entities, and then call `Remove` on the entities that should be deleted.</span></span> <span data-ttu-id="7fee9-232">此跟踪实体的图形通常通过以下方法之一创建：</span><span class="sxs-lookup"><span data-stu-id="7fee9-232">This graph of tracked entities is typically created by either:</span></span>

1. <span data-ttu-id="7fee9-233">针对实体运行查询</span><span class="sxs-lookup"><span data-stu-id="7fee9-233">Running a query for the entities</span></span>
2. <span data-ttu-id="7fee9-234">`Attach` `Update` 如前面几节中所述，在断开连接的实体图上使用或方法。</span><span class="sxs-lookup"><span data-stu-id="7fee9-234">Using the `Attach` or `Update` methods on a graph of disconnected entities, as described in the preceding sections.</span></span>

<span data-ttu-id="7fee9-235">例如，上一部分中的代码更有可能从客户端获得 post，然后执行如下所示的操作：</span><span class="sxs-lookup"><span data-stu-id="7fee9-235">For example, the code in the previous section is more likely obtain a post from a client and then do something like this:</span></span>

<!--
            context.Attach(post);
            context.Remove(post);
-->
[!code-csharp[Deleting_dependent_child_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Deleting_dependent_child_entities_1)]

<span data-ttu-id="7fee9-236">此行为与上一个示例的行为完全相同，因为 `Remove` 对未跟踪的实体调用会使其首先附加，然后标记为 `Deleted` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-236">This behaves exactly the same way as the previous example, since calling `Remove` on an un-tracked entity causes it to first be attached and then marked as `Deleted`.</span></span>

<span data-ttu-id="7fee9-237">在更逼真的示例中，首先附加实体的图形，然后将其中一些实体标记为已删除。</span><span class="sxs-lookup"><span data-stu-id="7fee9-237">In more realistic examples, a graph of entities is first attached, and then some of those entities are marked as deleted.</span></span> <span data-ttu-id="7fee9-238">例如：</span><span class="sxs-lookup"><span data-stu-id="7fee9-238">For example:</span></span>

<!--
            // Attach a blog and associated posts
            context.Attach(blog);

            // Mark one post as Deleted
            context.Remove(blog.Posts[1]);
-->
[!code-csharp[Deleting_dependent_child_entities_2](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Deleting_dependent_child_entities_2)]

<span data-ttu-id="7fee9-239">所有实体都标记为 `Unchanged` ，但调用的实体除外 `Remove` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-239">All entities are marked as `Unchanged`, except the one on which `Remove` was called:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}]
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

<span data-ttu-id="7fee9-240">此实体将在调用 SaveChanges 时删除。</span><span class="sxs-lookup"><span data-stu-id="7fee9-240">This entity will be deleted when SaveChanges is called.</span></span> <span data-ttu-id="7fee9-241">例如，使用 SQLite 时：</span><span class="sxs-lookup"><span data-stu-id="7fee9-241">For example, when using SQLite:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='2' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Posts"
WHERE "Id" = @p0;
SELECT changes();
```

<span data-ttu-id="7fee9-242">SaveChanges 完成后，删除的实体将从 DbContext 中分离出来，因为它不再存在于数据库中。</span><span class="sxs-lookup"><span data-stu-id="7fee9-242">After SaveChanges completes, the deleted entity is detached from the DbContext since it no longer exists in the database.</span></span> <span data-ttu-id="7fee9-243">其他实体保持 `Unchanged` 状态：</span><span class="sxs-lookup"><span data-stu-id="7fee9-243">Other entities remain in the `Unchanged` state:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Posts: [{Id: 1}]
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
```

## <a name="deleting-principalparent-entities"></a><span data-ttu-id="7fee9-244">删除主体/父实体</span><span class="sxs-lookup"><span data-stu-id="7fee9-244">Deleting principal/parent entities</span></span>

<span data-ttu-id="7fee9-245">连接两个实体类型的每个关系都具有一个主体或父端以及一个依赖项或子端。</span><span class="sxs-lookup"><span data-stu-id="7fee9-245">Each relationship that connects two entity types has a principal or parent end, and a dependent or child end.</span></span> <span data-ttu-id="7fee9-246">从属实体或子实体是具有外键属性的实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-246">The dependent/child entity is the one with the foreign key property.</span></span> <span data-ttu-id="7fee9-247">在一对多关系中，主体/父对象位于 "一" 方，从属/子级位于 "多" 端。</span><span class="sxs-lookup"><span data-stu-id="7fee9-247">In a one-to-many relationship, the principal/parent is on the "one" side, and the dependent/child is on the "many" side.</span></span> <span data-ttu-id="7fee9-248">有关详细信息，请参阅 [关系](xref:core/modeling/relationships) 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-248">See [Relationships](xref:core/modeling/relationships) for more information.</span></span>

<span data-ttu-id="7fee9-249">在前面的示例中，我们删除了 post，这是博客文章一对多关系中的依赖/子实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-249">In the preceding examples we were deleting a post, which is a dependent/child entity in the blog-posts one-to-many relationship.</span></span> <span data-ttu-id="7fee9-250">这比较简单，因为删除依赖/子实体不会对其他实体产生任何影响。</span><span class="sxs-lookup"><span data-stu-id="7fee9-250">This is relatively straightforward since removal of a dependent/child entity does not have any impact on other entities.</span></span> <span data-ttu-id="7fee9-251">另一方面，删除主体/父实体还必须影响任何从属/子实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-251">On the other hand, deleting a principal/parent entity must also impact any dependent/child entities.</span></span> <span data-ttu-id="7fee9-252">如果不这样做，则会保留一个外键值，该外键值引用不再存在的主键值。</span><span class="sxs-lookup"><span data-stu-id="7fee9-252">Not doing so would leave a foreign key value referencing a primary key value that no longer exists.</span></span> <span data-ttu-id="7fee9-253">这是无效的模型状态，在大多数数据库中导致引用约束错误。</span><span class="sxs-lookup"><span data-stu-id="7fee9-253">This is an invalid model state and results in a referential constraint error in most databases.</span></span>

<span data-ttu-id="7fee9-254">此无效模型状态可以通过两种方式进行处理：</span><span class="sxs-lookup"><span data-stu-id="7fee9-254">This invalid model state can be handled in two ways:</span></span>

1. <span data-ttu-id="7fee9-255">将 FK 值设置为 null。</span><span class="sxs-lookup"><span data-stu-id="7fee9-255">Setting FK values to null.</span></span> <span data-ttu-id="7fee9-256">这表明从属项/子项不再与任何主体/父对象相关。</span><span class="sxs-lookup"><span data-stu-id="7fee9-256">This indicates that the dependents/children are no longer related to any principal/parent.</span></span> <span data-ttu-id="7fee9-257">这是可选关系的默认值，其中外键必须可以为 null。</span><span class="sxs-lookup"><span data-stu-id="7fee9-257">This is the default for optional relationships where the foreign key must be nullable.</span></span> <span data-ttu-id="7fee9-258">将 FK 设置为 null 对于必需的关系无效，其中外键通常不可为 null。</span><span class="sxs-lookup"><span data-stu-id="7fee9-258">Setting the FK to null is not valid for required relationships, where the foreign key is typically non-nullable.</span></span>
2. <span data-ttu-id="7fee9-259">删除依赖项/子项。</span><span class="sxs-lookup"><span data-stu-id="7fee9-259">Deleting the the dependents/children.</span></span> <span data-ttu-id="7fee9-260">这是所需关系的默认值，对于可选关系也是有效的。</span><span class="sxs-lookup"><span data-stu-id="7fee9-260">This is the default for required relationships, and is also valid for optional relationships.</span></span>

<span data-ttu-id="7fee9-261">有关更改跟踪和关系的详细信息，请参阅 [更改外键和导航](xref:core/change-tracking/relationship-changes) 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-261">See [Changing Foreign Keys and Navigations](xref:core/change-tracking/relationship-changes) for detailed information on change tracking and relationships.</span></span>

### <a name="optional-relationships"></a><span data-ttu-id="7fee9-262">可选关系</span><span class="sxs-lookup"><span data-stu-id="7fee9-262">Optional relationships</span></span>

<span data-ttu-id="7fee9-263">`Post.BlogId`外键属性在我们所使用的模型中是可以为 null 的。</span><span class="sxs-lookup"><span data-stu-id="7fee9-263">The `Post.BlogId` foreign key property is nullable in the model we have been using.</span></span> <span data-ttu-id="7fee9-264">这意味着关系是可选的，因此，EF Core 的默认行为是在 `BlogId` 删除博客时将外键属性设置为 null。</span><span class="sxs-lookup"><span data-stu-id="7fee9-264">This means the relationship is optional, and hence the default behavior of EF Core is to set `BlogId` foreign key properties to null when the blog is deleted.</span></span> <span data-ttu-id="7fee9-265">例如：</span><span class="sxs-lookup"><span data-stu-id="7fee9-265">For example:</span></span>

<!--
            // Attach a blog and associated posts
            context.Attach(blog);

            // Mark one post as Deleted
            context.Remove(blog.Posts[1]);
-->
[!code-csharp[Deleting_principal_parent_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Deleting_principal_parent_entities_1)]

<span data-ttu-id="7fee9-266">按照预期，在调用后检查 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图以 `Remove` 显示，博客现在标记为 `Deleted` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-266">Inspecting the [change tracker debug view](xref:core/change-tracking/debug-views) following the call to `Remove` shows that, as expected, the blog is now marked as `Deleted`:</span></span>

```output
Blog {Id: 1} Deleted
  Id: 1 PK
  Name: '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}]
Post {Id: 1} Modified
  Id: 1 PK
  BlogId: <null> FK Modified Originally 1
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: <null>
Post {Id: 2} Modified
  Id: 2 PK
  BlogId: <null> FK Modified Originally 1
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: <null>
```

<span data-ttu-id="7fee9-267">更有趣的是，所有相关文章现在都标记为 `Modified` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-267">More interestingly, all the related posts are now marked as `Modified`.</span></span> <span data-ttu-id="7fee9-268">这是因为每个实体中的外键属性都设置为 null。</span><span class="sxs-lookup"><span data-stu-id="7fee9-268">This is because the foreign key property in each entity has been set to null.</span></span> <span data-ttu-id="7fee9-269">在删除博客之前，调用 SaveChanges 会将数据库中每个发布的外键值更新为 null：</span><span class="sxs-lookup"><span data-stu-id="7fee9-269">Calling SaveChanges updates the foreign key value for each post to null in the database, before then deleting the blog:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p1='1' (DbType = String), @p0=NULL], CommandType='Text', CommandTimeout='30']
UPDATE "Posts" SET "BlogId" = @p0
WHERE "Id" = @p1;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p1='2' (DbType = String), @p0=NULL], CommandType='Text', CommandTimeout='30']
UPDATE "Posts" SET "BlogId" = @p0
WHERE "Id" = @p1;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p2='1' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Blogs"
WHERE "Id" = @p2;
SELECT changes();
```

<span data-ttu-id="7fee9-270">SaveChanges 完成后，删除的实体将从 DbContext 中分离出来，因为它不再存在于数据库中。</span><span class="sxs-lookup"><span data-stu-id="7fee9-270">After SaveChanges completes, the deleted entity is detached from the DbContext since it no longer exists in the database.</span></span> <span data-ttu-id="7fee9-271">其他实体现在标记为 `Unchanged` 具有 null 外键值，这与数据库的状态相匹配：</span><span class="sxs-lookup"><span data-stu-id="7fee9-271">Other entities are now marked as `Unchanged` with null foreign key values, which matches the state of the database:</span></span>

```output
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: <null> FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: <null>
Post {Id: 2} Unchanged
  Id: 2 PK
  BlogId: <null> FK
  Content: 'F# 5 is the latest version of F#, the functional programming...'
  Title: 'Announcing F# 5'
  Blog: <null>
```

### <a name="required-relationships"></a><span data-ttu-id="7fee9-272">必选关系</span><span class="sxs-lookup"><span data-stu-id="7fee9-272">Required relationships</span></span>

<span data-ttu-id="7fee9-273">如果 `Post.BlogId` 外键属性不可为 null，则博客和文章之间的关系将变为 "必需"。</span><span class="sxs-lookup"><span data-stu-id="7fee9-273">If the `Post.BlogId` foreign key property is non-nullable, then the relationship between blogs and posts becomes "required".</span></span> <span data-ttu-id="7fee9-274">在这种情况下，默认情况下，EF Core 将在删除主体/父实体时删除从属实体/子实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-274">In this situation, EF Core will, by default, delete dependent/child entities when the principal/parent is deleted.</span></span> <span data-ttu-id="7fee9-275">例如，删除包含相关文章的博客，如前一示例中所示：</span><span class="sxs-lookup"><span data-stu-id="7fee9-275">For example, deleting a blog with related posts as in the previous example:</span></span>

<!--
            // Attach a blog and associated posts
            context.Attach(blog);

            // Mark one post as Deleted
            context.Remove(blog.Posts[1]);
-->
[!code-csharp[Deleting_principal_parent_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysRequiredSamples.cs?name=Deleting_principal_parent_entities_1)]

<span data-ttu-id="7fee9-276">按照预期，在调用后检查 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图以 `Remove` 显示，博客再次标记为 `Deleted` ：</span><span class="sxs-lookup"><span data-stu-id="7fee9-276">Inspecting the [change tracker debug view](xref:core/change-tracking/debug-views) following the call to `Remove` shows that, as expected, the blog is again marked as `Deleted`:</span></span>

```output
Blog {Id: 1} Deleted
  Id: 1 PK
  Name: '.NET Blog'
  Posts: [{Id: 1}, {Id: 2}]
Post {Id: 1} Deleted
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

<span data-ttu-id="7fee9-277">更有趣的是，在这种情况下，所有相关的文章也已标记为 `Deleted` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-277">More interestingly in this case is that all related posts have also been marked as `Deleted`.</span></span> <span data-ttu-id="7fee9-278">调用 SaveChanges 会导致从数据库中删除博客和所有相关的发布：</span><span class="sxs-lookup"><span data-stu-id="7fee9-278">Calling SaveChanges causes the blog and all related posts to be deleted from the database:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='1' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Posts"
WHERE "Id" = @p0;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p0='2' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Posts"
WHERE "Id" = @p0;
SELECT changes();

-- Executed DbCommand (0ms) [Parameters=[@p1='1' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Blogs"
WHERE "Id" = @p1;
```

<span data-ttu-id="7fee9-279">SaveChanges 完成后，所有删除的实体将从 DbContext 中分离出来，因为它们不再存在于数据库中。</span><span class="sxs-lookup"><span data-stu-id="7fee9-279">After SaveChanges completes, all the deleted entities are detached from the DbContext since they no longer exist in the database.</span></span> <span data-ttu-id="7fee9-280">因此，"调试" 视图的输出为空。</span><span class="sxs-lookup"><span data-stu-id="7fee9-280">Output from the debug view is therefore empty.</span></span>

> [!NOTE]
> <span data-ttu-id="7fee9-281">本文档仅对使用 EF Core 中的关系的图面进行了划痕。</span><span class="sxs-lookup"><span data-stu-id="7fee9-281">This document only scratches the surface on working with relationships in EF Core.</span></span> <span data-ttu-id="7fee9-282">若要详细了解如何在调用 SaveChanges 时更新/删除从属/子实体，请参阅 [关系](xref:core/modeling/relationships) 了解建模关系的详细信息，以及 [更改外键和导航](xref:core/change-tracking/relationship-changes) 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-282">See [Relationships](xref:core/modeling/relationships) for more information on modeling relationships, and [Changing Foreign Keys and Navigations](xref:core/change-tracking/relationship-changes) for more information on updating/deleting dependent/child entities when calling SaveChanges.</span></span>

## <a name="custom-tracking-with-trackgraph"></a><span data-ttu-id="7fee9-283">自定义跟踪与 TrackGraph</span><span class="sxs-lookup"><span data-stu-id="7fee9-283">Custom tracking with TrackGraph</span></span>

<span data-ttu-id="7fee9-284"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.TrackGraph%2A?displayProperty=nameWithType> 的工作方式类似于 `Add` `Attach` 和， `Update` 只不过它在跟踪之前为每个实体实例生成一个回调。</span><span class="sxs-lookup"><span data-stu-id="7fee9-284"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.TrackGraph%2A?displayProperty=nameWithType> works like `Add`, `Attach` and `Update` except that it generates a callback for every entity instance before tracking it.</span></span> <span data-ttu-id="7fee9-285">这允许在确定如何跟踪图形中的各个实体时使用自定义逻辑。</span><span class="sxs-lookup"><span data-stu-id="7fee9-285">This allows custom logic to be used when determining how to track individual entities in a graph.</span></span>

<span data-ttu-id="7fee9-286">例如，请考虑在使用生成的键值跟踪实体时使用的规则 EF Core：如果键值为零，则实体为 new，应插入该实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-286">For example, consider the rule EF Core uses when tracking entities with generated key values: if the key value is zero, then the entity is new and should be inserted.</span></span> <span data-ttu-id="7fee9-287">让我们扩展此规则，以指示键值是否为负，然后应删除该实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-287">Let's extend this rule to say if the key value is negative, then the entity should be deleted.</span></span> <span data-ttu-id="7fee9-288">这样，我们便可以更改断开连接图形的实体中的主键值，以标记已删除的实体：</span><span class="sxs-lookup"><span data-stu-id="7fee9-288">This allows us to change the primary key values in entities of a disconnected graph to mark deleted entities:</span></span>

<!--
            blog.Posts.Add(
                new Post
                {
                    Title = "Announcing .NET 5.0",
                    Content = ".NET 5.0 includes many enhancements, including single file applications, more..."
                }
            );

            var toDelete = blog.Posts.Single(e => e.Title == "Announcing F# 5");
            toDelete.Id = -toDelete.Id;
-->
[!code-csharp[Custom_tracking_with_TrackGraph_1a](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/GeneratedKeysSamples.cs?name=Custom_tracking_with_TrackGraph_1a)]

<span data-ttu-id="7fee9-289">然后，可以使用 TrackGraph 跟踪此断开连接的图形：</span><span class="sxs-lookup"><span data-stu-id="7fee9-289">This disconnected graph can then be tracked using TrackGraph:</span></span>

<!--
        public static void UpdateBlog(Blog blog)
        {
            using var context = new BlogsContext();

            context.ChangeTracker.TrackGraph(
                blog, node =>
                    {
                        var propertyEntry = node.Entry.Property("Id");
                        var keyValue = (int)propertyEntry.CurrentValue;

                        if (keyValue == 0)
                        {
                            node.Entry.State = EntityState.Added;
                        }
                        else if (keyValue < 0)
                        {
                            propertyEntry.CurrentValue = -keyValue;
                            node.Entry.State = EntityState.Deleted;
                        }
                        else
                        {
                            node.Entry.State = EntityState.Modified;
                        }

                        Console.WriteLine($"Tracking {node.Entry.Metadata.DisplayName()} with key value {keyValue} as {node.Entry.State}");

                    });

            context.SaveChanges();
        }
-->
[!code-csharp[Custom_tracking_with_TrackGraph_1b](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/GeneratedKeysSamples.cs?name=Custom_tracking_with_TrackGraph_1b)]

<span data-ttu-id="7fee9-290">对于图形中的每个实体，上面的代码会 _在跟踪实体之前_ 检查主键值。</span><span class="sxs-lookup"><span data-stu-id="7fee9-290">For each entity in the graph, the code above checks the primary key value _before tracking the entity_.</span></span> <span data-ttu-id="7fee9-291">对于 unset (零) 键值，代码执行 EF Core 通常会执行的操作。</span><span class="sxs-lookup"><span data-stu-id="7fee9-291">For unset (zero) key values, the code does what EF Core would normally do.</span></span> <span data-ttu-id="7fee9-292">也就是说，如果未设置密钥，则实体将标记为 `Added` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-292">That is, if the key is not set, then the entity is marked as `Added`.</span></span> <span data-ttu-id="7fee9-293">如果设置了该键并且值为非负值，则实体将标记为 `Modified` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-293">If the key is set and the value is non-negative, then the entity is marked as `Modified`.</span></span> <span data-ttu-id="7fee9-294">但是，如果找到负键值，则会还原其真实的非负值，并将实体作为进行跟踪 `Deleted` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-294">However, if a negative key value is found, then its real, non-negative value is restored and the entity is tracked as `Deleted`.</span></span>

<span data-ttu-id="7fee9-295">运行此代码的输出为：</span><span class="sxs-lookup"><span data-stu-id="7fee9-295">The output from running this code is:</span></span>

```output
Tracking Blog with key value 1 as Modified
Tracking Post with key value 1 as Modified
Tracking Post with key value -2 as Deleted
Tracking Post with key value 0 as Added
```

> [!NOTE]
> <span data-ttu-id="7fee9-296">为简单起见，此代码假定每个实体都有一个名为的整数主键属性 `Id` 。</span><span class="sxs-lookup"><span data-stu-id="7fee9-296">For simplicity, this code assumes each entity has an integer primary key property called `Id`.</span></span> <span data-ttu-id="7fee9-297">这可能会整理到抽象基类或接口中。</span><span class="sxs-lookup"><span data-stu-id="7fee9-297">This could be codified into an abstract base class or interface.</span></span> <span data-ttu-id="7fee9-298">或者，可以从元数据中获取主键属性， <xref:Microsoft.EntityFrameworkCore.Metadata.IEntityType> 以便此代码适用于任何类型的实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-298">Alternately, the primary key property or properties could be obtained from the <xref:Microsoft.EntityFrameworkCore.Metadata.IEntityType> metadata such that this code would work with any type of entity.</span></span>

<span data-ttu-id="7fee9-299">TrackGraph 有两个重载。</span><span class="sxs-lookup"><span data-stu-id="7fee9-299">TrackGraph has two overloads.</span></span> <span data-ttu-id="7fee9-300">在上面使用的简单重载中，EF Core 确定何时停止遍历关系图。</span><span class="sxs-lookup"><span data-stu-id="7fee9-300">In the simple overload used above, EF Core determines when to stop traversing the graph.</span></span> <span data-ttu-id="7fee9-301">具体而言，它将停止从给定实体访问新的相关实体（如果该实体已被跟踪）或回调不会开始跟踪实体。</span><span class="sxs-lookup"><span data-stu-id="7fee9-301">Specifically, it stops visiting new related entities from a given entity when that entity is either already tracked, or when the callback does not start tracking the entity.</span></span>

<span data-ttu-id="7fee9-302">高级重载 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.TrackGraph%60%601(System.Object,%60%600,System.Func{Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntryGraphNode{%60%600},System.Boolean})?displayProperty=nameWithType> 具有返回布尔值的回调。</span><span class="sxs-lookup"><span data-stu-id="7fee9-302">The advanced overload, <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.TrackGraph%60%601(System.Object,%60%600,System.Func{Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntryGraphNode{%60%600},System.Boolean})?displayProperty=nameWithType>, has a callback that returns a bool.</span></span> <span data-ttu-id="7fee9-303">如果回调返回 false，则图形遍历将停止，否则将继续。</span><span class="sxs-lookup"><span data-stu-id="7fee9-303">If the callback returns false, then graph traversal stops, otherwise it continues.</span></span> <span data-ttu-id="7fee9-304">使用此重载时，必须小心避免出现无限循环。</span><span class="sxs-lookup"><span data-stu-id="7fee9-304">Care must be taken to avoid infinite loops when using this overload.</span></span>

<span data-ttu-id="7fee9-305">高级重载还允许向 TrackGraph 提供状态，并将此状态传递给每个回调。</span><span class="sxs-lookup"><span data-stu-id="7fee9-305">The advanced overload also allows state to be supplied to TrackGraph and this state is then passed to each callback.</span></span>
