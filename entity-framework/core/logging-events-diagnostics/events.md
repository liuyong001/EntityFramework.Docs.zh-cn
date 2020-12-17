---
title: .NET 事件-EF Core
description: EF Core 定义的 .NET 事件
author: ajcvickers
ms.date: 10/15/2020
uid: core/logging-events-diagnostics/events
ms.openlocfilehash: 51c0bba5cf25e1d9ddd1fd9aebea50b9a03481a3
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97635687"
---
# <a name="net-events-in-ef-core"></a><span data-ttu-id="a0fce-103">EF Core 中的 .NET 事件</span><span class="sxs-lookup"><span data-stu-id="a0fce-103">.NET Events in EF Core</span></span>

> [!TIP]  
> <span data-ttu-id="a0fce-104">可以从 GitHub [下载事件示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Events) 。</span><span class="sxs-lookup"><span data-stu-id="a0fce-104">You can [download the events sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Events) from GitHub.</span></span>

<span data-ttu-id="a0fce-105">Entity Framework Core (EF Core) 公开 [.net 事件](/dotnet/standard/events/) ，以在 EF Core 代码中发生某些事情时充当回调。</span><span class="sxs-lookup"><span data-stu-id="a0fce-105">Entity Framework Core (EF Core) exposes [.NET events](/dotnet/standard/events/) to act as callbacks when certain things happen in the EF Core code.</span></span> <span data-ttu-id="a0fce-106">事件比 [侦听器](xref:core/logging-events-diagnostics/interceptors) 简单，并且允许更灵活的注册。</span><span class="sxs-lookup"><span data-stu-id="a0fce-106">Events are simpler than [interceptors](xref:core/logging-events-diagnostics/interceptors) and allow more flexible registration.</span></span> <span data-ttu-id="a0fce-107">但是，它们只是同步，因此不能执行非阻塞异步 I/O。</span><span class="sxs-lookup"><span data-stu-id="a0fce-107">However, they are sync only and so cannot perform non-blocking async I/O.</span></span>

<span data-ttu-id="a0fce-108">每个实例都注册事件 `DbContext` 。</span><span class="sxs-lookup"><span data-stu-id="a0fce-108">Events are registered per `DbContext` instance.</span></span> <span data-ttu-id="a0fce-109">使用[诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)获取相同的信息，但要获取进程中所有 DbContext 实例的信息。</span><span class="sxs-lookup"><span data-stu-id="a0fce-109">Use a [diagnostic listener](xref:core/logging-events-diagnostics/diagnostic-listeners) to get the same information but for all DbContext instances in the process.</span></span>

## <a name="events-raised-by-ef-core"></a><span data-ttu-id="a0fce-110">EF Core 引发的事件</span><span class="sxs-lookup"><span data-stu-id="a0fce-110">Events raised by EF Core</span></span>

<span data-ttu-id="a0fce-111">EF Core 引发以下事件：</span><span class="sxs-lookup"><span data-stu-id="a0fce-111">The following events are raised by EF Core:</span></span>

| <span data-ttu-id="a0fce-112">事件</span><span class="sxs-lookup"><span data-stu-id="a0fce-112">Event</span></span> | <span data-ttu-id="a0fce-113">引入的版本</span><span class="sxs-lookup"><span data-stu-id="a0fce-113">Version introduced</span></span> | <span data-ttu-id="a0fce-114">引发时间</span><span class="sxs-lookup"><span data-stu-id="a0fce-114">When raised</span></span>
|:------|--------------------|-------
| <xref:Microsoft.EntityFrameworkCore.DbContext.SavingChanges?displayProperty=nameWithType> | <span data-ttu-id="a0fce-115">5.0</span><span class="sxs-lookup"><span data-stu-id="a0fce-115">5.0</span></span> | <span data-ttu-id="a0fce-116">在或的开头 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A><xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A></span><span class="sxs-lookup"><span data-stu-id="a0fce-116">At the start of <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> or <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A></span></span>
| <xref:Microsoft.EntityFrameworkCore.DbContext.SavedChanges?displayProperty=nameWithType> | <span data-ttu-id="a0fce-117">5.0</span><span class="sxs-lookup"><span data-stu-id="a0fce-117">5.0</span></span> | <span data-ttu-id="a0fce-118">在成功 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 或 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A></span><span class="sxs-lookup"><span data-stu-id="a0fce-118">At the end of a successful <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> or <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A></span></span>
| <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesFailed?displayProperty=nameWithType> | <span data-ttu-id="a0fce-119">5.0</span><span class="sxs-lookup"><span data-stu-id="a0fce-119">5.0</span></span> | <span data-ttu-id="a0fce-120">在失败 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 或 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A></span><span class="sxs-lookup"><span data-stu-id="a0fce-120">At the end of a failed <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> or <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A></span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Tracked?displayProperty=nameWithType> | <span data-ttu-id="a0fce-121">2.1</span><span class="sxs-lookup"><span data-stu-id="a0fce-121">2.1</span></span> | <span data-ttu-id="a0fce-122">当上下文跟踪实体时</span><span class="sxs-lookup"><span data-stu-id="a0fce-122">When an entity is tracked by the context</span></span>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.StateChanged?displayProperty=nameWithType> | <span data-ttu-id="a0fce-123">2.1</span><span class="sxs-lookup"><span data-stu-id="a0fce-123">2.1</span></span> | <span data-ttu-id="a0fce-124">当跟踪的实体更改其状态时</span><span class="sxs-lookup"><span data-stu-id="a0fce-124">When a tracked entity changes its state</span></span>

### <a name="example-timestamp-state-changes"></a><span data-ttu-id="a0fce-125">示例：时间戳状态更改</span><span class="sxs-lookup"><span data-stu-id="a0fce-125">Example: Timestamp state changes</span></span>

<span data-ttu-id="a0fce-126">DbContext 跟踪的每个实体都有一个 <xref:Microsoft.EntityFrameworkCore.EntityState> 。</span><span class="sxs-lookup"><span data-stu-id="a0fce-126">Each entity tracked by a DbContext has an <xref:Microsoft.EntityFrameworkCore.EntityState>.</span></span> <span data-ttu-id="a0fce-127">例如， `Added` 状态指示将实体插入到数据库中。</span><span class="sxs-lookup"><span data-stu-id="a0fce-127">For example, the `Added` state indicates that the entity will be inserted into the database.</span></span>

<span data-ttu-id="a0fce-128">此示例使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Tracked> 和 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.StateChanged> 事件来检测实体更改状态的时间。</span><span class="sxs-lookup"><span data-stu-id="a0fce-128">This example uses the <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Tracked> and <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.StateChanged> events to detect when an entity changes state.</span></span> <span data-ttu-id="a0fce-129">然后，它会将实体标记为当前时间，指示此更改发生的时间。</span><span class="sxs-lookup"><span data-stu-id="a0fce-129">It then stamps the entity with the current time indicating when this change happened.</span></span> <span data-ttu-id="a0fce-130">这会导致时间戳，指示实体的插入、删除和/或上次更新的时间。</span><span class="sxs-lookup"><span data-stu-id="a0fce-130">This results in timestamps indicating when the entity was inserted, deleted, and/or last updated.</span></span>

<span data-ttu-id="a0fce-131">在此示例中，实体类型实现了一个接口，该接口定义时间戳属性：</span><span class="sxs-lookup"><span data-stu-id="a0fce-131">The entity types in this example implement an interface that defines the timestamp properties:</span></span>

<!--
public interface IHasTimestamps
{
    DateTime? Added { get; set; }
    DateTime? Deleted { get; set; }
    DateTime? Modified { get; set; }
}
-->
[!code-csharp[IHasTimestamps](../../../samples/core/Miscellaneous/Events/Program.cs?name=IHasTimestamps)]

<span data-ttu-id="a0fce-132">然后，应用程序的 DbContext 上的方法可以为实现此接口的任何实体设置时间戳：</span><span class="sxs-lookup"><span data-stu-id="a0fce-132">A method on the application's DbContext can then set timestamps for any entity that implements this interface:</span></span>

<!--
    private static void UpdateTimestamps(object sender, EntityEntryEventArgs e)
    {
        if (e.Entry.Entity is IHasTimestamps entityWithTimestamps)
        {
            switch (e.Entry.State)
            {
                case EntityState.Deleted:
                    entityWithTimestamps.Deleted = DateTime.UtcNow;
                    Console.WriteLine($"Stamped for delete: {e.Entry.Entity}");
                    break;
                case EntityState.Modified:
                    entityWithTimestamps.Modified = DateTime.UtcNow;
                    Console.WriteLine($"Stamped for update: {e.Entry.Entity}");
                    break;
                case EntityState.Added:
                    entityWithTimestamps.Added = DateTime.UtcNow;
                    Console.WriteLine($"Stamped for insert: {e.Entry.Entity}");
                    break;
            }
        }
    }
-->
[!code-csharp[UpdateTimestamps](../../../samples/core/Miscellaneous/Events/Program.cs?name=UpdateTimestamps)]

<span data-ttu-id="a0fce-133">此方法具有用于和事件的事件处理程序的适当签名 `Tracked` `StateChanged` 。</span><span class="sxs-lookup"><span data-stu-id="a0fce-133">This method has the appropriate signature to use as an event handler for both the `Tracked` and `StateChanged` events.</span></span> <span data-ttu-id="a0fce-134">为 DbContext 构造函数中的两个事件注册处理程序。</span><span class="sxs-lookup"><span data-stu-id="a0fce-134">The handler is registered for both events in the DbContext constructor.</span></span> <span data-ttu-id="a0fce-135">请注意，可以随时将事件附加到 DbContext;这不是必需的。</span><span class="sxs-lookup"><span data-stu-id="a0fce-135">Note that events can be attached to a DbContext at any time; it is not required that this happen in the context constructor.</span></span>

<!--
    public BlogsContext()
    {
        ChangeTracker.StateChanged += UpdateTimestamps;
        ChangeTracker.Tracked += UpdateTimestamps;
    }
-->
[!code-csharp[ContextConstructor](../../../samples/core/Miscellaneous/Events/Program.cs?name=ContextConstructor)]

<span data-ttu-id="a0fce-136">这两个事件都是必需的，因为 `Tracked` 当首次跟踪事件时，新实体会激发事件。</span><span class="sxs-lookup"><span data-stu-id="a0fce-136">Both events are needed because new entities fire `Tracked` events when they are first tracked.</span></span> <span data-ttu-id="a0fce-137">`StateChanged` 仅 _在跟踪_ 事件时更改状态的实体才会触发事件。</span><span class="sxs-lookup"><span data-stu-id="a0fce-137">`StateChanged` events are only fired for entities that change state while they are _already_ being tracked.</span></span>

<span data-ttu-id="a0fce-138">此示例的 [示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Events) 包含一个简单的控制台应用程序，可对博客数据库进行更改：</span><span class="sxs-lookup"><span data-stu-id="a0fce-138">The [sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Events) for this example contains a simple console application that makes changes to the blogging database:</span></span>

<!--
        using (var context = new BlogsContext())
        {
            context.Database.EnsureDeleted();
            context.Database.EnsureCreated();
            
            context.Add(
                new Blog
                {
                    Id = 1,
                    Name = "EF Blog",
                    Posts =
                    {
                        new Post { Id = 1, Title = "EF Core 3.1!" },
                        new Post { Id = 2, Title = "EF Core 5.0!" }
                    }
                });

            context.SaveChanges();
        }

        using (var context = new BlogsContext())
        {
            var blog = context.Blogs.Include(e => e.Posts).Single();

            blog.Name = "EF Core Blog";
            context.Remove(blog.Posts.First());
            blog.Posts.Add(new Post { Id = 3, Title = "EF Core 6.0!" });

            context.SaveChanges();
        }
-->
[!code-csharp[Demonstration](../../../samples/core/Miscellaneous/Events/Program.cs?name=Demonstration)]

<span data-ttu-id="a0fce-139">此代码的输出显示发生的状态更改和要应用的时间戳：</span><span class="sxs-lookup"><span data-stu-id="a0fce-139">The output from this code shows the state changes happening and the timestamps being applied:</span></span>

```output
Stamped for insert: Blog 1 Added on: 10/15/2020 11:01:26 PM
Stamped for insert: Post 1 Added on: 10/15/2020 11:01:26 PM
Stamped for insert: Post 2 Added on: 10/15/2020 11:01:26 PM
Stamped for delete: Post 1 Added on: 10/15/2020 11:01:26 PM Deleted on: 10/15/2020 11:01:26 PM
Stamped for update: Blog 1 Added on: 10/15/2020 11:01:26 PM Modified on: 10/15/2020 11:01:26 PM
Stamped for insert: Post 3 Added on: 10/15/2020 11:01:26 PM
```
