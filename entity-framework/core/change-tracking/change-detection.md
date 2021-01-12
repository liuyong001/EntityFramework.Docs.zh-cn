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
# <a name="change-detection-and-notifications"></a>更改检测和通知

每个 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例跟踪对实体所做的更改。 在调用时，这些跟踪的实体会驱动对数据库的更改 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。 [EF Core 的更改跟踪](xref:core/change-tracking/index)中介绍了这一点，并且本文档假设了解 Entity Framework Core (EF Core) 更改跟踪的实体状态和基础知识。

跟踪属性和关系更改要求 DbContext 能够检测到这些更改。 本文档介绍了如何进行这种检测，以及如何使用属性通知或更改跟踪代理强制立即检测更改。

> [!TIP]
> 通过 [从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangeDetectionAndNotifications)，你可以运行并调试到本文档中的所有代码。

## <a name="snapshot-change-tracking"></a>快照更改跟踪

默认情况下，在 DbContext 实例首次跟踪每个实体的属性值时，EF Core 会创建其快照。 然后，将存储在此快照中的值与实体的当前值进行比较，以确定更改了哪些属性值。

在调用 SaveChanges 时，会发生更改，以确保在将更新发送到数据库之前检测到所有更改的值。 不过，还会在其他时间进行更改检测，以确保应用程序能够使用最新的跟踪信息。 可以通过调用随时强制执行更改检测 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> 。

### <a name="when-change-detection-is-needed"></a>需要更改检测时

如果在 _不使用 EF Core 进行此更改的情况下_ 更改了属性或导航，则需要检测更改。 例如，考虑加载博客和文章，然后对这些实体进行更改：

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

在调用之前查看 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图会 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> 显示未检测到所做的更改，因此不会在实体状态和修改后的属性数据中反映这些更改：

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

具体而言，博客条目的状态仍为 `Unchanged` ，而新的帖子不会显示为跟踪的实体。  (敏锐将会看到 "属性" 报告其新值，即使 EF Core 尚未检测到这些更改。 这是因为 "调试" 视图直接从实体实例读取当前值。 ) 

在调用 DetectChanges 后，将此与调试视图进行比较：

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

现在，博客已正确标记为 `Modified` ，并已检测到新文章，并将其作为跟踪 `Added` 。

在本部分开始时，我们指出在不使用 _EF Core 进行更改_ 时需要检测更改。 这就是上述代码所发生的情况。 也就是说，对属性和导航的更改 _直接对实体实例_ 进行，而不是使用任何 EF Core 方法。

将此与以下代码进行比较，以相同方式修改实体，但这次使用 EF Core 方法：

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

在这种情况下，更改跟踪器的 "调试" 视图会显示所有实体状态和属性修改都是已知的，即使未发生更改检测也是如此。 这是因为 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.CurrentValue?displayProperty=nameWithType> 是 EF Core 方法，这意味着 EF Core 立即知道此方法所做的更改。 同样，通过调用， <xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType> EF Core 立即了解新实体并进行相应跟踪。

> [!TIP]
> 不要尝试通过始终使用 EF Core 方法进行实体更改来避免检测更改。 这样做通常更繁琐，而且执行方式不如在正常情况中对实体进行更改。 本文档的目的是在需要检测更改时通知，如果不需要，则通知。 目的不是鼓励避免更改检测。

### <a name="methods-that-automatically-detect-changes"></a>自动检测更改的方法

<xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges> 方法会自动调用此方法，这可能会影响结果。 这些方法包括：

- <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A?displayProperty=nameWithType> ，以确保在更新数据库之前检测到所有更改。
- <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%60%601?displayProperty=nameWithType> ，以确保实体状态和修改的属性是最新的。
- <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.HasChanges?displayProperty=nameWithType>，以确保结果准确无误。
- <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeChanges?displayProperty=nameWithType>，用于在级联之前确保主体/父实体的实体状态正确。
- <xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType>，以确保跟踪的关系图是最新的。

在某些位置，只会对单个实体实例进行更改检测，而不是在整个所跟踪实体关系图上发生更改。 这些位置包括：

- 使用时 <xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType> ，确保实体的状态和修改的属性是最新的。
- 使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry> 、或等方法 `Property` `Collection` `Reference` `Member` 确保属性修改、当前值等是最新的。
- 当要删除依赖/子实体时，因为所需的关系已被断开。 这会检测不应删除实体的时间，因为它已重新获得父级。

可以通过调用，显式触发单个实体的更改的本地检测 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.DetectChanges?displayProperty=nameWithType> 。

> [!NOTE]
> 本地检测更改可能会丢失完全检测会发现的一些更改。 如果因未检测到对其他实体进行的更改而导致的级联操作对相关实体产生影响，则会发生这种情况。 在这种情况下，应用程序可能需要通过显式调用强制对所有实体进行完全扫描 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> 。

### <a name="disabling-automatic-change-detection"></a>禁用自动更改检测

对于大多数应用程序而言，检测更改的性能并不是瓶颈。 但对于跟踪上千个实体的某些应用程序而言，检测更改可能会成为性能问题。  (的确切数字将取决于许多因素，如实体中属性的数目。 ) 为此，可以使用禁用更改的自动检测 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.AutoDetectChangesEnabled?displayProperty=nameWithType> 。 例如，考虑使用有效负载处理多对多关系中的联接实体：

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

正如我们在上一节中所知道的，和都将 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%60%601?displayProperty=nameWithType> <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A?displayProperty=nameWithType> 自动检测更改。 但是，在调用项后，代码不会更改任何实体或属性状态。  (设置添加的实体的普通属性值不会导致任何状态更改。 ) 代码因此，在向下调用基本 SaveChanges 方法时，将禁用不必要的自动更改检测。 此代码还利用 try/finally 块来确保即使 SaveChanges 失败，也会还原默认设置。

> [!TIP]
> 不要假设你的代码必须禁用自动更改检测，才能正常运行。 这只是在分析应用程序跟踪时需要的，很多实体表示更改检测的性能问题。

### <a name="detecting-changes-and-value-conversions"></a>检测更改和值转换

若要对实体类型使用快照更改跟踪，EF Core 必须能够：

- 跟踪实体时创建每个属性值的快照
- 将此值与属性的当前值进行比较
- 为值生成哈希代码

对于可以直接映射到数据库的类型 EF Core，会自动处理这种情况。 但是，在 [使用值转换器来映射属性](xref:core/modeling/value-conversions)时，转换器必须指定如何执行这些操作。 这是通过值比较器实现的，在 [值](xref:core/modeling/value-comparers) 比较器文档中进行了详细说明。

## <a name="notification-entities"></a>通知实体

对于大多数应用程序，建议使用快照更改跟踪。 但是，跟踪多个实体和/或对这些实体进行很多更改的应用程序可能会受益于实现实体，这些实体会在属性和导航值改变时自动通知 EF Core。 它们称为 "通知实体"。

### <a name="implementing-notification-entities"></a>实现通知实体

通知实体使用 <xref:System.ComponentModel.INotifyPropertyChanging> 和 <xref:System.ComponentModel.INotifyPropertyChanged> 接口，这些接口是 .net 基类库 (BCL) 的一部分。 这些接口定义在更改属性值之前和之后必须激发的事件。 例如：

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

此外，任何集合导航都必须实现 `INotifyCollectionChanged` ; 在上面的示例中，通过使用 post 来满足这一要求 <xref:System.Collections.ObjectModel.ObservableCollection%601> 。 EF Core 还附带了一个 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ObservableHashSet%601> 实现，该实现具有更高效的查找，但代价是稳定的排序。

大多数此类通知代码通常会移到未映射的基类中。 例如：

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

### <a name="configuring-notification-entities"></a>配置通知实体

无法 EF Core 验证是否已 `INotifyPropertyChanging` `INotifyPropertyChanged` 完全实现或使用 EF Core。 具体而言，这些接口的某些用途只对某些属性使用通知，而不是根据 EF Core 所需 (包括导航) 的所有属性。 出于此原因，EF Core 不会自动挂接到这些事件。

相反，EF Core 必须配置为使用这些通知实体。 通常通过调用对所有实体类型执行此操作 <xref:Microsoft.EntityFrameworkCore.ModelBuilder.HasChangeTrackingStrategy%2A?displayProperty=nameWithType> 。 例如：

<!--
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.HasChangeTrackingStrategy(ChangeTrackingStrategy.ChangingAndChangedNotifications);
        }
-->
[!code-csharp[OnModelCreating](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/NotificationWithBaseSamples.cs?name=OnModelCreating)]

 (还可以使用不同的实体类型为不同的实体类型设置该策略 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.EntityTypeBuilder.HasChangeTrackingStrategy%2A?displayProperty=nameWithType> ，但这通常是适得其反的，因为那些不是通知实体的类型仍然需要 DetectChanges。 ) 

完全通知更改跟踪要求 `INotifyPropertyChanging` `INotifyPropertyChanged` 实现和。 这样就可以在属性值更改之前保存原始值，从而无需 EF Core 在跟踪实体时创建快照。 实现的实体类型 `INotifyPropertyChanged` 还可用于 EF Core。 在这种情况下，EF 仍会在跟踪实体时创建快照以跟踪原始值，但随后使用这些通知来检测更改，而不需要调用 DetectChanges。

<xref:Microsoft.EntityFrameworkCore.ChangeTrackingStrategy>下表汇总了不同的值。

| ChangeTrackingStrategy                              | 需要的接口                                      | 需要 DetectChanges | 快照原始值
|:----------------------------------------------------|--------------------------------------------------------|---------------------|--------------------------
| 快照                                            | None                                                   | 是                 | 是
| ChangedNotifications                                | INotifyPropertyChanged                                 | 否                  | 是
| ChangingAndChangedNotifications                     | INotifyPropertyChanged 和 INotifyPropertyChanging     | 否                  | 否
| ChangingAndChangedNotificationsWithOriginalValues   | INotifyPropertyChanged 和 INotifyPropertyChanging     | 否                  | 是

### <a name="using-notification-entities"></a>使用通知实体

通知实体的行为与其他任何实体类似，只是对实体实例进行更改不需要调用来 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> 检测这些更改。 例如：

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

对于普通实体， [更改跟踪](xref:core/change-tracking/debug-views) 器 "调试" 视图显示在调用 DetectChanges 之前未检测到这些更改。 当使用通知实体时，查看调试视图表明已立即检测到这些更改：

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

## <a name="change-tracking-proxies"></a>更改跟踪代理

> [!NOTE]
> EF Core 5.0 中引入了更改跟踪代理。

EF Core 可以动态生成实现和的代理 <xref:System.ComponentModel.INotifyPropertyChanging> 类型 <xref:System.ComponentModel.INotifyPropertyChanged> 。 这需要安装 [Microsoft.entityframeworkcore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Proxies/) NuGet 包，并启用更改跟踪代理， <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseChangeTrackingProxies%2A> 例如：

<!--
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
            => optionsBuilder.UseChangeTrackingProxies();
-->
[!code-csharp[OnConfiguring](../../../samples/core/ChangeTracking/ChangeDetectionAndNotifications/ChangeTrackingProxiesSamples.cs?name=OnConfiguring)]

创建动态代理涉及到使用 [城堡](https://www.nuget.org/packages/Castle.Core/) 代理实现创建新的动态 .net 类型 () ，该实现继承自实体类型，然后覆盖所有属性资源库。 因此，代理的实体类型必须是可以从继承的类型，并且必须具有可重写的属性。 此外，显式创建的集合导航必须实现 <xref:System.Collections.Specialized.INotifyCollectionChanged> 示例：

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

更改跟踪代理的一个重大缺点是 EF Core 必须始终跟踪代理的实例，而不是基础实体类型的实例。 这是因为，基础实体类型的实例将不会生成通知，这意味着对这些实体所做的更改将丢失。

EF Core 在查询数据库时自动创建代理实例，因此，这种缺点通常仅限于跟踪新的实体实例。 这些实例必须使用 <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.CreateProxy%2A> 扩展方法创建，而 **不** 是使用来创建 `new` 。 这意味着，前面的示例中的代码现在必须使用 `CreateProxy` ：

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

## <a name="change-tracking-events"></a>更改跟踪事件

EF Core <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Tracked?displayProperty=nameWithType> 第一次跟踪实体时，会触发事件。 将来的实体状态更改会导致 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.StateChanged?displayProperty=nameWithType> 事件发生。 有关详细信息，请参阅 [EF Core 中的 .NET 事件](xref:core/logging-events-diagnostics/events)。

> [!NOTE]
> `StateChanged`第一次跟踪实体时不会触发事件，即使状态已从更改 `Detached` 为其他状态之一。 请确保侦听 `StateChanged` 和 `Tracked` 事件以获取所有相关的通知。
