---
title: 附加更改跟踪功能-EF Core
description: 涉及 EF Core 更改跟踪的其他功能和方案
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/miscellaneous
ms.openlocfilehash: 9eb3186f4eef300e4824dc86700497444ece4a2c
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543414"
---
# <a name="additional-change-tracking-features"></a>其他更改跟踪功能

本文档介绍涉及更改跟踪的其他功能和方案。

> [!TIP]
> 本文档假设了解 EF Core 更改跟踪的实体状态和基础知识。 有关这些主题的详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。

> [!TIP]
> 通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/AdditionalChangeTrackingFeatures)，你可运行并调试到本文档中的所有代码。

## <a name="add-versus-addasync"></a>`Add` 与 `AddAsync`

Entity Framework Core (EF Core) 在使用该方法时提供异步方法可能会导致数据库交互。 还提供了同步方法，以避免使用不支持高性能异步访问的数据库时的开销。

<xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType><xref:Microsoft.EntityFrameworkCore.DbSet%601.Add%2A?displayProperty=nameWithType>通常不会访问数据库，因为这些方法本质上只是开始跟踪实体。 但某些形式的值生成 _可以_ 访问数据库，以便生成键值。 唯一这样做并随 EF Core 提供的值生成器为 <xref:Microsoft.EntityFrameworkCore.ValueGeneration.HiLoValueGenerator%601> 。 使用此生成器不常见;默认情况下，它不配置。 这意味着大多数应用程序应使用 `Add` ，而不是 `AddAsync` 。

诸如、和之类的其他类似方法没有 `Update` `Attach` `Remove` 异步重载，因为它们永远不会生成新的键值，因此永远不需要访问数据库。

## <a name="addrange-updaterange-attachrange-and-removerange"></a>`AddRange`、`UpdateRange`、`AttachRange` 和 `RemoveRange`

<xref:Microsoft.EntityFrameworkCore.DbSet%601> 和 <xref:Microsoft.EntityFrameworkCore.DbContext> 提供了、、和的备用版本 `Add` ， `Update` `Attach` `Remove` 可在单个调用中接受多个实例。 这些方法 <xref:Microsoft.EntityFrameworkCore.DbSet%601.AddRange%2A> 分别为、 <xref:Microsoft.EntityFrameworkCore.DbSet%601.UpdateRange%2A> 、 <xref:Microsoft.EntityFrameworkCore.DbSet%601.AttachRange%2A> 和 <xref:Microsoft.EntityFrameworkCore.DbSet%601.RemoveRange%2A> 。

这些方法是提供方便的。 使用 "范围" 方法与对等效的非范围方法的多个调用具有相同的功能。 这两种方法之间没有明显的性能差异。

> [!NOTE]
> 这不同于 EF6，其中 `AddRange` 和 `Add` 都是自动调用的 `DetectChanges` ，但调用多次 `Add` 将导致多次调用 DetectChanges，而不是一次。 这 `AddRange` 在 EF6 中提高了效率。 在 EF Core 中，这两种方法都不会自动调用 `DetectChanges` 。

## <a name="dbcontext-versus-dbset-methods"></a>DbContext 与 DbSet 方法

许多方法（包括 `Add` 、 `Update` 、 `Attach` 和 `Remove` ）在和上都具有 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 实现 <xref:Microsoft.EntityFrameworkCore.DbContext> 。 对于普通实体类型，这些方法具有 _完全相同的行为_ 。 这是因为实体的 CLR 类型会映射到 EF Core 模型中的一个实体类型。 因此，CLR 类型会完全定义实体在模型中的适合位置，因此，可以隐式确定要使用的 DbSet。

此规则的例外情况是使用在 EF Core 5.0 中引入的共享类型实体类型，主要用于多对多联接实体。 使用共享类型的实体类型时，必须首先为正在使用的 EF Core 模型类型创建 DbSet。 `Add`然后，可以在 DbSet 上使用类似于、、和的方法， `Update` `Attach` `Remove` 而不会有任何不明确的方法可用于所使用的 EF Core 模型类型。

默认情况下，对于多对多关系中的联接实体，将使用共享类型的实体类型。 还可以显式配置共享类型的实体类型，以便在多对多关系中使用。 例如，下面的代码将配置 `Dictionary<string, int>` 为联接实体类型：

<!--
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder
                .SharedTypeEntity<Dictionary<string, int>>(
                    "PostTag",
                    b =>
                        {
                            b.IndexerProperty<int>("TagId");
                            b.IndexerProperty<int>("PostId");
                        });

            modelBuilder.Entity<Post>()
                .HasMany(p => p.Tags)
                .WithMany(p => p.Posts)
                .UsingEntity<Dictionary<string, int>>(
                    "PostTag",
                    j => j.HasOne<Tag>().WithMany(),
                    j => j.HasOne<Post>().WithMany());
        }
-->
[!code-csharp[OnModelCreating](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/Samples.cs?name=OnModelCreating)]

[更改外键和导航](xref:core/change-tracking/relationship-changes) 显示了如何通过跟踪新的联接实体实例来关联两个实体。 下面的代码针对 `Dictionary<string, int>` 用于联接实体的共享类型实体类型执行此操作：

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            var joinEntitySet = context.Set<Dictionary<string, int>>("PostTag");
            var joinEntity = new Dictionary<string, int>
            {
                ["PostId"] = post.Id,
                ["TagId"] = tag.Id
            };
            joinEntitySet.Add(joinEntity);

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);

            context.SaveChanges();
-->
[!code-csharp[DbContext_versus_DbSet_methods_1](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/Samples.cs?name=DbContext_versus_DbSet_methods_1)]

请注意， <xref:Microsoft.EntityFrameworkCore.DbContext.Set%60%601(System.String)?displayProperty=nameWithType> 用于为 `PostTag` 实体类型创建 DbSet。 然后，可以使用此 DbSet 调用 `Add` 新的联接实体实例。

> [!IMPORTANT]
> 用于按约定联接实体类型的 CLR 类型在将来的版本中可能会更改以提高性能。 不依赖于任何特定的联接实体类型，除非已将其显式配置为 `Dictionary<string, int>` 在上面的代码中完成。

## <a name="property-versus-field-access"></a>属性和字段访问

从 EF Core 3.0 开始，默认情况下，对实体属性的访问将使用属性的支持字段。 这是高效的，避免了调用属性 getter 和 setter 的副作用。 例如，这是延迟加载如何避免触发无限循环。 有关在模型中配置支持字段的详细信息，请参阅 [支持字段](xref:core/modeling/backing-field) 。

有时 EF Core 需要在修改属性值时生成副作用。 例如，将数据绑定到实体时，设置属性可能会向 U.I. 生成通知。 直接设置字段时不会发生这种情况。 为此，可将更改 <xref:Microsoft.EntityFrameworkCore.PropertyAccessMode> 为：

- 模型中使用的所有实体类型 <xref:Microsoft.EntityFrameworkCore.ModelBuilder.UsePropertyAccessMode%2A?displayProperty=nameWithType>
- 使用特定实体类型的所有属性和导航 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.EntityTypeBuilder%601.UsePropertyAccessMode%2A?displayProperty=nameWithType>
- 使用的特定属性 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.UsePropertyAccessMode%2A?displayProperty=nameWithType>
- 使用的特定导航 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.NavigationBuilder.UsePropertyAccessMode%2A?displayProperty=nameWithType>

属性访问模式 `Field` `PreferField` 将导致 EF Core 通过其支持字段访问属性值。 同样， `Property` 和 `PreferProperty` 将导致 EF Core 通过其 getter 和 setter 访问属性值。

如果 `Field` 使用或， `Property` 并且 EF Core 无法分别通过字段或属性 getter/setter 访问该值，则 EF Core 会引发异常。 这可确保在您认为 EF Core 始终使用字段/属性访问。

另一方面， `PreferField` `PreferProperty` 如果不能使用首选访问权限，和模式将回退到使用属性或支持字段。 `PreferField` 是 EF Core 3.0 的默认值。 这意味着 EF Core 将尽可能使用字段，但如果必须通过其 getter 或 setter 访问属性，则不会失败。

`FieldDuringConstruction` 并 `PreferFieldDuringConstruction` 将 EF Core 配置为 _仅在创建实体实例时_ 使用支持字段。 这允许执行无 getter 和 setter 副作用的查询，而 EF Core 的属性更改将导致这些副作用。 `PreferFieldDuringConstruction` EF Core 3.0 之前的默认值。

下表汇总了不同的属性访问模式：

| PropertyAccessMode              | 首选项 | 首选项创建实体 | 回退 | 回退创建实体
|:--------------------------------|------------|------------------------------|----------|---------------------------
| `Field`                         | 字段      | 字段                        | 引发   | 引发
| `Property`                      | 属性   | 属性                     | 引发   | 引发
| `PreferField`                   | 字段      | 字段                        | 属性 | 属性
| `PreferProperty`                | 属性   | 属性                     | 字段    | 字段
| `FieldDuringConstruction`       | 属性   | 字段                        | 字段    | 引发
| `PreferFieldDuringConstruction` | 属性   | 字段                        | 字段    | 属性

## <a name="temporary-values"></a>临时值

EF Core 在跟踪新实体时创建临时密钥值，当调用 SaveChanges 时，将具有数据库生成的实际密钥值。 有关如何使用这些临时值的概述，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。

### <a name="accessing-temporary-values"></a>访问临时值

从 EF Core 3.0 开始，临时值存储在更改跟踪器中，而不是直接设置到实体实例。 但是，当使用各种机制 [访问跟踪的实体](xref:core/change-tracking/entity-entries)时，_会_ 公开这些临时值。 例如，以下代码使用访问临时值 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.CurrentValues%2A?displayProperty=nameWithType> ：

<!--
        using var context = new BlogsContext();

        var blog = new Blog { Name = ".NET Blog" };

        context.Add(blog);

        Console.WriteLine($"Blog.Id set on entity is {blog.Id}");
        Console.WriteLine($"Blog.Id tracked by EF is {context.Entry(blog).Property(e => e.Id).CurrentValue}");
-->
[!code-csharp[Temporary_values_1](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/Samples.cs?name=Temporary_values_1)]

此代码的输出为：

```output
Blog.Id set on entity is 0
Blog.Id tracked by EF is -2147482643
```

<xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsTemporary%2A?displayProperty=nameWithType> 可用于检查临时值。

### <a name="manipulating-temporary-values"></a>操作临时值

对于显式处理临时值，有时很有用。 例如，可以在 web 客户端上创建新实体的集合，然后将其序列化回服务器。 外键值是在这些实体之间建立关系的一种方法。 下面的代码使用此方法将新实体的关系图与外键关联，同时仍然允许在调用 SaveChanges 时生成实际键值。

<!--
            var blogs = new List<Blog>
            {
                new Blog { Id = -1, Name = ".NET Blog" },
                new Blog { Id = -2, Name = "Visual Studio Blog" }
            };

            var posts = new List<Post>()
            {
                new Post
                {
                    Id = -1,
                    BlogId = -1,
                    Title = "Announcing the Release of EF Core 5.0",
                    Content = "Announcing the release of EF Core 5.0, a full featured cross-platform..."
                },
                new Post
                {
                    Id = -2,
                    BlogId = -2,
                    Title = "Disassembly improvements for optimized managed debugging",
                    Content = "If you are focused on squeezing out the last bits of performance for your .NET service or..."
                }
            };

            using var context = new BlogsContext();

            foreach (var blog in blogs)
            {
                context.Add(blog).Property(e => e.Id).IsTemporary = true;
            }

            foreach (var post in posts)
            {
                context.Add(post).Property(e => e.Id).IsTemporary = true;
            }

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);

            context.SaveChanges();

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Temporary_values_2](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/Samples.cs?name=Temporary_values_2)]

请注意：

- 负数作为临时键值使用;这不是必需的，但它是防止密钥冲突的常见约定。
- 为 `Post.BlogId` FK 属性分配与关联博客的 PK 相同的负值。
- 跟踪每个实体后，将通过设置将 PK 值标记为临时值 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsTemporary%2A> 。 这是必需的，因为应用程序提供的任何密钥值都假定为实际键值。

在调用 SaveChanges 之前查看 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图会显示 PK 值标记为临时，并与正确的博客关联，其中包括导航的修正：

```output
Blog {Id: -2} Added
  Id: -2 PK Temporary
  Name: 'Visual Studio Blog'
  Posts: [{Id: -2}]
Blog {Id: -1} Added
  Id: -1 PK Temporary
  Name: '.NET Blog'
  Posts: [{Id: -1}]
Post {Id: -2} Added
  Id: -2 PK Temporary
  BlogId: -2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: {Id: -2}
  Tags: []
Post {Id: -1} Added
  Id: -1 PK Temporary
  BlogId: -1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: -1}
```

调用后 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> ，这些临时值被数据库生成的真实值所替换：

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Posts: [{Id: 1}]
Blog {Id: 2} Unchanged
  Id: 2 PK
  Name: 'Visual Studio Blog'
  Posts: [{Id: 2}]
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
  Tags: []
Post {Id: 2} Unchanged
  Id: 2 PK
  BlogId: 2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: {Id: 2}
  Tags: []
```

## <a name="working-with-default-values"></a>使用默认值

当调用时，EF Core 允许属性从数据库获取其默认值 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。 与生成的键值一样，如果没有显式设置值，EF Core 将仅从数据库使用默认值。 例如，请考虑以下实体类型：

<!--
    public class Token
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public DateTime ValidFrom { get; set; }
    }
-->
[!code-csharp[Token](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Token)]

此 `ValidFrom` 属性配置为从数据库中获取默认值：

<!--
        modelBuilder
            .Entity<Token>()
            .Property(e => e.ValidFrom)
            .HasDefaultValueSql("CURRENT_TIMESTAMP");
-->
[!code-csharp[OnModelCreating_Token](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=OnModelCreating_Token)]

插入此类型的实体时，EF Core 将使数据库生成值，除非已设置了显式值。 例如：

<!--
            using var context = new BlogsContext();

            context.AddRange(
                new Token { Name = "A" },
                new Token { Name = "B", ValidFrom = new DateTime(1111, 11, 11, 11, 11, 11)});

            context.SaveChanges();

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Working_with_default_values_1](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Working_with_default_values_1)]

查看 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图会显示数据库生成的第一个标记 `ValidFrom` ，而第二个标记使用了显式设置的值：

```output
Token {Id: 1} Unchanged
  Id: 1 PK
  Name: 'A'
  ValidFrom: '12/30/2020 6:36:06 PM'
Token {Id: 2} Unchanged
  Id: 2 PK
  Name: 'B'
  ValidFrom: '11/11/1111 11:11:11 AM'
```

> [!NOTE]
> 使用数据库默认值要求数据库列已配置默认值约束。 使用或时 EF Core 迁移，会自动完成此操作 <xref:Microsoft.EntityFrameworkCore.RelationalPropertyBuilderExtensions.HasDefaultValueSql%2A> <xref:Microsoft.EntityFrameworkCore.RelationalPropertyBuilderExtensions.HasDefaultValue%2A> 。 在不使用 EF Core 迁移时，请确保以其他某种方式在列上创建默认约束。

### <a name="using-nullable-properties"></a>使用可以为 null 的属性

EF Core 可以通过将属性值与该类型的 CLR 默认值进行比较来确定是否已设置了该属性。 这在大多数情况下非常有效，但意味着无法将 CLR 默认值显式插入数据库中。 例如，请考虑一个具有整数属性的实体：

<!--
public class Foo1
{
    public int Id { get; set; }
    public int Count { get; set; }
}
-->
[!code-csharp[Foo1](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Foo1)]

其中，该属性配置为具有数据库默认值-1：

<!--
        modelBuilder
            .Entity<Foo1>()
            .Property(e => e.Count)
            .HasDefaultValue(-1);
-->
[!code-csharp[OnModelCreating_Foo1](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=OnModelCreating_Foo1)]

目的是在未设置显式值时，将使用默认值-1。 但是，如果将值设置为0，则 () 的 CLR 默认值不能区分 EF Core 不设置任何值，这意味着不能为此属性插入0。 例如：

<!--
        using var context = new BlogsContext();

        var fooA = new Foo1 { Count = 10 };
        var fooB = new Foo1 { Count = 0 };
        var fooC = new Foo1 { };

        context.AddRange(fooA, fooB, fooC);
        context.SaveChanges();

        Debug.Assert(fooA.Count == 10);
        Debug.Assert(fooB.Count == -1); // Not what we want!
        Debug.Assert(fooC.Count == -1);
-->
[!code-csharp[Working_with_default_values_2](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Working_with_default_values_2)]

请注意， `Count` 显式设置为0的实例仍从数据库中获取默认值，而这不是我们预期的值。 处理此操作的一种简单方法是将属性设置为 `Count` 可以为 null：

<!--
public class Foo2
{
    public int Id { get; set; }
    public int? Count { get; set; }
}
-->
[!code-csharp[Foo2](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Foo2)]

这会使 CLR 默认值为 null，而不是0，这意味着在显式设置时，将插入0：

<!--
        using var context = new BlogsContext();

        var fooA = new Foo2 { Count = 10 };
        var fooB = new Foo2 { Count = 0 };
        var fooC = new Foo2 { };

        context.AddRange(fooA, fooB, fooC);
        context.SaveChanges();

        Debug.Assert(fooA.Count == 10);
        Debug.Assert(fooB.Count == 0);
        Debug.Assert(fooC.Count == -1);
-->
[!code-csharp[Working_with_default_values_3](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Working_with_default_values_3)]

### <a name="using-nullable-backing-fields"></a>使用可以为 null 的支持字段

> [!NOTE]
> EF Core 5.0 及更高版本支持此可为 null 的支持字段模式。

使属性可以为 null 的问题，它在域模型中可能不能是可以为 null 的。 如果强制属性可以为 null，则会对模型进行折衷。

从 EF Core 5.0 开始，属性可以保留为不可为 null，只有支持字段可以为 null。 例如：

<!--
public class Foo3
{
    public int Id { get; set; }

    private int? _count;
    public int Count
    {
        get => _count ?? -1;
        set => _count = value;
    }
}
-->
[!code-csharp[Foo3](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Foo3)]

这允许在将属性显式设置为0的情况下插入 CLR 默认 (0) ，而无需在域模型中将属性公开为可为 null。 例如：

<!--
            using var context = new BlogsContext();

            var fooA = new Foo3 { Count = 10 };
            var fooB = new Foo3 { Count = 0 };
            var fooC = new Foo3 { };

            context.AddRange(fooA, fooB, fooC);
            context.SaveChanges();

            Debug.Assert(fooA.Count == 10);
            Debug.Assert(fooB.Count == 0);
            Debug.Assert(fooC.Count == -1);
-->
[!code-csharp[Working_with_default_values_4](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Working_with_default_values_4)]

#### <a name="nullable-backing-fields-for-bool-properties"></a>布尔属性的可为 null 的支持字段

当使用具有存储生成的默认值的 bool 属性时，此模式特别有用。 由于的 CLR 默认值 `bool` 为 "false"，这意味着不能使用 normal 模式显式插入 "false"。 例如，假设有一个 `User` 实体类型：

<!--
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }

    private bool? _isAuthorized;
    public bool IsAuthorized
    {
        get => _isAuthorized ?? true;
        set => _isAuthorized = value;
    }
}
-->
[!code-csharp[User](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=User)]

此 `IsAuthorized` 属性配置为数据库默认值为 "true"：

<!--
        modelBuilder
            .Entity<User>()
            .Property(e => e.IsAuthorized)
            .HasDefaultValue(true);
-->
[!code-csharp[OnModelCreating_User](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=OnModelCreating_User)]

在 `IsAuthorized` 插入之前，可以将属性显式设置为 "true" 或 "false"，也可以将其设置为 "false"，在这种情况下，将使用数据库默认值：

<!--
        using var context = new BlogsContext();

        var userA = new User { Name = "Mac" };
        var userB = new User { Name = "Alice", IsAuthorized = true };
        var userC = new User { Name = "Baxter", IsAuthorized = false }; // Always deny Baxter access!

        context.AddRange(userA, userB, userC);

        context.SaveChanges();
-->
[!code-csharp[Working_with_default_values_5](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Working_with_default_values_5)]

使用 SQLite 时，SaveChanges 的输出显示数据库默认值用于 Mac，而为 Alice 和 Baxter 设置了显式值：

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='Mac' (Size = 3)], CommandType='Text', CommandTimeout='30']
INSERT INTO "User" ("Name")
VALUES (@p0);
SELECT "Id", "IsAuthorized"
FROM "User"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();

-- Executed DbCommand (0ms) [Parameters=[@p0='True' (DbType = String), @p1='Alice' (Size = 5)], CommandType='Text', CommandTimeout='30']
INSERT INTO "User" ("IsAuthorized", "Name")
VALUES (@p0, @p1);
SELECT "Id"
FROM "User"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();

-- Executed DbCommand (0ms) [Parameters=[@p0='False' (DbType = String), @p1='Baxter' (Size = 6)], CommandType='Text', CommandTimeout='30']
INSERT INTO "User" ("IsAuthorized", "Name")
VALUES (@p0, @p1);
SELECT "Id"
FROM "User"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();
```

### <a name="schema-defaults-only"></a>仅限架构默认值

有时，通过 EF Core 迁移创建的数据库架构中的默认值非常有用，EF Core 不会使用这些值进行插入。 为此，可以配置属性，例如 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.ValueGeneratedNever%2A?displayProperty=nameWithType> ：

<!--
        modelBuilder
            .Entity<Bar>()
            .Property(e => e.Count)
            .HasDefaultValue(-1)
            .ValueGeneratedNever();
-->
[!code-csharp[OnModelCreating_Bar](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=OnModelCreating_Bar)]
