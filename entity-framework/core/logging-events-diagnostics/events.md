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
# <a name="net-events-in-ef-core"></a>EF Core 中的 .NET 事件

> [!TIP]  
> 可以从 GitHub [下载事件示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Events) 。

Entity Framework Core (EF Core) 公开 [.net 事件](/dotnet/standard/events/) ，以在 EF Core 代码中发生某些事情时充当回调。 事件比 [侦听器](xref:core/logging-events-diagnostics/interceptors) 简单，并且允许更灵活的注册。 但是，它们只是同步，因此不能执行非阻塞异步 I/O。

每个实例都注册事件 `DbContext` 。 使用[诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)获取相同的信息，但要获取进程中所有 DbContext 实例的信息。

## <a name="events-raised-by-ef-core"></a>EF Core 引发的事件

EF Core 引发以下事件：

| 事件 | 引入的版本 | 引发时间
|:------|--------------------|-------
| <xref:Microsoft.EntityFrameworkCore.DbContext.SavingChanges?displayProperty=nameWithType> | 5.0 | 在或的开头 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A><xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A>
| <xref:Microsoft.EntityFrameworkCore.DbContext.SavedChanges?displayProperty=nameWithType> | 5.0 | 在成功 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 或 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A>
| <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesFailed?displayProperty=nameWithType> | 5.0 | 在失败 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 或 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A>
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Tracked?displayProperty=nameWithType> | 2.1 | 当上下文跟踪实体时
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.StateChanged?displayProperty=nameWithType> | 2.1 | 当跟踪的实体更改其状态时

### <a name="example-timestamp-state-changes"></a>示例：时间戳状态更改

DbContext 跟踪的每个实体都有一个 <xref:Microsoft.EntityFrameworkCore.EntityState> 。 例如， `Added` 状态指示将实体插入到数据库中。

此示例使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Tracked> 和 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.StateChanged> 事件来检测实体更改状态的时间。 然后，它会将实体标记为当前时间，指示此更改发生的时间。 这会导致时间戳，指示实体的插入、删除和/或上次更新的时间。

在此示例中，实体类型实现了一个接口，该接口定义时间戳属性：

<!--
public interface IHasTimestamps
{
    DateTime? Added { get; set; }
    DateTime? Deleted { get; set; }
    DateTime? Modified { get; set; }
}
-->
[!code-csharp[IHasTimestamps](../../../samples/core/Miscellaneous/Events/Program.cs?name=IHasTimestamps)]

然后，应用程序的 DbContext 上的方法可以为实现此接口的任何实体设置时间戳：

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

此方法具有用于和事件的事件处理程序的适当签名 `Tracked` `StateChanged` 。 为 DbContext 构造函数中的两个事件注册处理程序。 请注意，可以随时将事件附加到 DbContext;这不是必需的。

<!--
    public BlogsContext()
    {
        ChangeTracker.StateChanged += UpdateTimestamps;
        ChangeTracker.Tracked += UpdateTimestamps;
    }
-->
[!code-csharp[ContextConstructor](../../../samples/core/Miscellaneous/Events/Program.cs?name=ContextConstructor)]

这两个事件都是必需的，因为 `Tracked` 当首次跟踪事件时，新实体会激发事件。 `StateChanged` 仅 _在跟踪_ 事件时更改状态的实体才会触发事件。

此示例的 [示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Events) 包含一个简单的控制台应用程序，可对博客数据库进行更改：

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

此代码的输出显示发生的状态更改和要应用的时间戳：

```output
Stamped for insert: Blog 1 Added on: 10/15/2020 11:01:26 PM
Stamped for insert: Post 1 Added on: 10/15/2020 11:01:26 PM
Stamped for insert: Post 2 Added on: 10/15/2020 11:01:26 PM
Stamped for delete: Post 1 Added on: 10/15/2020 11:01:26 PM Deleted on: 10/15/2020 11:01:26 PM
Stamped for update: Blog 1 Added on: 10/15/2020 11:01:26 PM Modified on: 10/15/2020 11:01:26 PM
Stamped for insert: Post 3 Added on: 10/15/2020 11:01:26 PM
```
