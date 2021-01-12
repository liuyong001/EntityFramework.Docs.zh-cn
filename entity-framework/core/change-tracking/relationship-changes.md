---
title: 更改外键和导航-EF Core
description: 如何通过操作外键和导航来更改实体之间的关系
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/relationship-changes
ms.openlocfilehash: ac2110509b6748e85411dbb14989522465925ecf
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129582"
---
# <a name="changing-foreign-keys-and-navigations"></a>更改外键和导航

## <a name="overview-of-foreign-keys-and-navigations"></a>外键和导航概述

EF Core) 模型的 Entity Framework Core (中的关系使用 (Fk) 的外键来表示。 FK 由关系中依赖实体或子实体上的一个或多个属性组成。 当 dependent/child 上的外键属性的值与其他或主键的值相匹配时，此从属/子实体将与给定的主体/父实体相关联 (PK) 主体/父项的属性。

外键是一种在数据库中存储和处理关系的好方法，但在应用程序代码中处理多个相关实体时，它们并不是很好的方法。 因此，大多数 EF Core 模型还会将 "导航" 置于 FK 表示形式之上。 导航窗体 c #/.NET 实体实例间的引用，这些实体实例反映通过将外键值与主键值或备用键值匹配而找到的关联。

导航既可以在关系的双方上使用，也可以在一侧上使用，也可以根本不会使用，只留下 FK 属性。 可以通过将 FK 属性设为 [阴影属性](xref:core/modeling/shadow-properties)来隐藏该属性。 有关建模关系的详细信息，请参阅 [关系](xref:core/modeling/relationships) 。

> [!TIP]
> 本文档假设了解 EF Core 更改跟踪的实体状态和基础知识。 有关这些主题的详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。

> [!TIP]
> 通过 [从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangingFKsAndNavigations)，你可以运行并调试到本文档中的所有代码。

### <a name="example-model"></a>示例模型

下面的模型包含四个实体类型，它们之间有关系。 代码中的注释指示哪些属性为外键、主键和导航。

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

此模型中的三个关系为：

- 每个博客可以有多个 (一对多) ：
  - `Blog` 是主体/父级。
  - `Post` 依赖项/子级。 它包含 FK 属性 `Post.BlogId` ，其值必须与 `Blog.Id` 相关博客的 PK 值匹配。
  - `Post.Blog` 是从发布到相关博客的引用导航。 `Post.Blog` 是的反向导航 `Blog.Posts` 。
  - `Blog.Posts` 是从博客到所有关联帖子的集合导航。 `Blog.Posts` 是的反向导航 `Post.Blog` 。
- 每个博客可以有一个资产 (一对一) ：
  - `Blog` 是主体/父级。
  - `BlogAssets` 依赖项/子级。 它包含 FK 属性 `BlogAssets.BlogId` ，其值必须与 `Blog.Id` 相关博客的 PK 值匹配。
  - `BlogAssets.Blog` 是从资源到关联博客的引用导航。 `BlogAssets.Blog` 是的反向导航 `Blog.Assets` 。
  - `Blog.Assets` 是从博客到相关资产的引用导航。 `Blog.Assets` 是的反向导航 `BlogAssets.Blog` 。
- 每篇文章都可以有多个标记，每个标记可有多个 post (多对多) ：
  - 多对多关系是比 2 1 到多关系的更多层关系。 多对多关系将在本文档的后面部分介绍。
  - `Post.Tags` 是从发布到所有关联标记的集合导航。 `Post.Tags` 是的反向导航 `Tag.Posts` 。
  - `Tag.Posts` 是从标记到所有关联的发布的集合导航。 `Tag.Posts` 是的反向导航 `Post.Tags` 。

有关如何建模和配置关系的详细信息，请参阅 [关系](xref:core/modeling/relationships) 。

## <a name="relationship-fixup"></a>关系修正

EF Core 使导航与外键值保持一致，反之亦然。 也就是说，如果外键值发生更改，使其现在引用其他主体/父实体，则会更新导航以反映此更改。 同样，如果更改了导航，则所涉及的实体的外键值将更新以反映此更改。 这称为 "关系修正"。

### <a name="fixup-by-query"></a>按查询修正

如果从数据库中查询实体，则会首先出现修正。 数据库仅具有外键值，因此当 EF Core 从数据库创建实体实例时，它将使用外键值来设置引用导航，并根据需要将实体添加到集合导航。 例如，请考虑对博客及其相关的文章和资产进行查询：

<!--
        using var context = new BlogsContext();

        var blogs = context.Blogs
            .Include(e => e.Posts)
            .Include(e => e.Assets)
            .ToList();

        Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Relationship_fixup_1](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Relationship_fixup_1)]

对于每个博客，EF Core 将首先创建一个 `Blog` 实例。 然后，在从数据库加载每个发布时，其 `Post.Blog` 引用导航设置为指向关联的博客。 同样，将 post 添加到 `Blog.Posts` 集合导航。 与相同 `BlogAssets` ，但在这种情况下，这两个导航都是引用。 `Blog.Assets`导航设置为指向资产实例， `BlogAsserts.Blog` 导航设置为指向博客实例。

在此查询显示两个博客后查看 " [更改跟踪器" 调试视图](xref:core/change-tracking/debug-views) ，每个博客包含一个资产和两篇文章：

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

"调试" 视图显示键值和导航。 使用相关实体的主键值显示导航。 例如， `Posts: [{Id: 1}, {Id: 2}]` 在上面的输出中，表示 `Blog.Posts` 集合导航分别包含两个主键分别为1和2的相关文章。 同样，对于与第一个博客关联的每个帖子， `Blog: {Id: 1}` 该行指示 `Post.Blog` 导航引用了主键为1的博客。

### <a name="fixup-to-locally-tracked-entities"></a>本地跟踪实体的链接

关系修正还会在从跟踪查询返回的实体与 DbContext 已跟踪的实体之间发生。 例如，考虑对博客、文章和资产执行三个单独的查询：

<!--
        using var context = new BlogsContext();

        var blogs = context.Blogs.ToList();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);

        var assets = context.Assets.ToList();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);

        var posts = context.Posts.ToList();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[！ code-csharp[Relationship_fixup_2](../../../ samples / core / ChangeTracking / ChangingFKsAndNavigations / OptionalRelationshipsSamples.cs ? name = Relationship_fixup_2
) ]再次查看调试视图，在第一次查询后，只跟踪这两个博客：

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

`Blog.Assets`引用导航为 null，并且 `Blog.Posts` 集合导航为空，因为当前上下文未跟踪关联的实体。

在第二次查询后， `Blogs.Assets` 引用导航已固定到新跟踪的 `BlogAsset` 实例。 同样， `BlogAssets.Blog` 引用导航设置为指向适当的已跟踪 `Blog` 实例。

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

最后，在第三次查询后， `Blog.Posts` 集合导航现在包含所有相关的帖子， `Post.Blog` 引用指向相应的 `Blog` 实例：

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

这与通过原始单个查询实现的结束状态相同，因为在跟踪实体时 EF Core 固定的导航，即使是在来自多个不同的查询时也是如此。

> [!NOTE]
> 修正永不导致从数据库返回更多数据。 它仅连接已由查询返回或已由 DbContext 跟踪的实体。 有关在序列化实体时处理重复项的信息，请参阅 [中的标识解析 EF Core](xref:core/change-tracking/identity-resolution) 。

## <a name="changing-relationships-using-navigations"></a>使用导航更改关系

更改两个实体之间的关系的最简单方法是操作导航，同时保留 EF Core 适当地修正反向导航和 FK 值。 可通过以下方法完成此操作：

- 添加或删除集合导航中的实体。
- 更改引用导航以指向不同的实体，或将其设置为 null。

### <a name="adding-or-removing-from-collection-navigations"></a>添加或删除集合导航

例如，让我们将其中一篇文章从 Visual Studio 博客移到 .NET 博客。 这需要首先加载博客和文章，然后将张贴内容从一个博客上的导航集合移到其他博客上的导航集合：

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
> 此处需要对的调用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> ，因为访问 "调试" 视图不会导致 [更改自动检测](xref:core/change-tracking/change-detection)。

这是运行上述代码后打印的 "调试" 视图：

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

`Blog.Posts`.Net 博客上的导航现在包含三个发布 (`Posts: [{Id: 1}, {Id: 2}, {Id: 3}]`) 。 同样， `Blog.Posts` Visual Studio 博客上的导航仅有一篇文章 (`Posts: [{Id: 4}]`) 。 这是预期的，因为代码显式更改了这些集合。

更有趣的是，尽管代码未显式更改 `Post.Blog` 导航，但它已得到修复，可指向 Visual Studio 博客 (`Blog: {Id: 1}`) 。 此外， `Post.BlogId` 外键值已更新，以匹配 .net 博客的主键值。 在调用 SaveChanges 时，此更改将保留到数据库中的 FK 值：

```sql
-- Executed DbCommand (0ms) [Parameters=[@p1='3' (DbType = String), @p0='1' (Nullable = true) (DbType = String)], CommandType='Text', CommandTimeout='30']
UPDATE "Posts" SET "BlogId" = @p0
WHERE "Id" = @p1;
SELECT changes();
```

### <a name="changing-reference-navigations"></a>更改引用导航

在上面的示例中，通过操作每个博客上的文章的集合导航，将文章从一个博客移到另一个博客。 可以通过更改 `Post.Blog` 引用导航来指向新博客来实现相同的目的。 例如：

<!--
        var post = vsBlog.Posts.Single(e => e.Title.StartsWith("Disassembly improvements"));
        post.Blog = dotNetBlog;
-->
[!code-csharp[Changing_relationships_using_navigations_2](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Changing_relationships_using_navigations_2)]

此更改之后的调试视图与上一示例中的调试视图 _完全相同_ 。 这是因为 EF Core 检测到引用导航更改，然后将集合导航和 FK 值固定到匹配项。

## <a name="changing-relationships-using-foreign-key-values"></a>使用外键值更改关系

在上一节中，关系由可自动更新的外键值的导航操作。 这是在 EF Core 中操作关系的建议方法。 但是，也可以直接处理 FK 值。 例如，可以通过更改外键值将 post 从一个博客移到另一个博客 `Post.BlogId` ：

<!--
        var post = vsBlog.Posts.Single(e => e.Title.StartsWith("Disassembly improvements"));
        post.BlogId = dotNetBlog.Id;
-->
[!code-csharp[Changing_relationships_using_foreign_key_values_1](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Changing_relationships_using_foreign_key_values_1)]

请注意，这与更改引用导航的方式非常类似，如前面的示例所示。

此更改之后的 "调试" 视图与前面两个示例的大小写 _完全相同_ 。 这是因为 EF Core 检测到 FK 值发生变化，并同时修复了要匹配的引用导航和集合导航。

> [!TIP]
> 请勿在每次关系发生更改时编写代码来操作所有导航和 FK 值。 此类代码更复杂，必须确保在每种情况下对外键和导航进行一致的更改。 如果可能，只需处理一个导航，或者同时处理两个导航。 如果需要，只需操作 FK 值。 避免处理导航和 FK 值。

## <a name="fixup-for-added-or-deleted-entities"></a>添加或删除实体的修正

### <a name="adding-to-a-collection-navigation"></a>添加到集合导航

EF Core 在 [检测](xref:core/change-tracking/change-detection) 到已向集合导航中添加了一个新的依赖/子实体时执行以下操作：

- 如果未跟踪实体，则跟踪该实体。 实体 (通常将处于 `Added` 状态。 但是，如果将实体类型配置为使用生成的密钥并设置了主键值，则会在状态中跟踪该实体 `Unchanged` 。 ) 
- 如果该实体与其他主体/父对象相关联，则该关系将被断开。
- 实体将与拥有集合导航的主体/父对象关联。
- 对于涉及的所有实体，导航和外键值都是固定的。

基于此，我们可以看到，从一个博客向另一个博客移动一篇文章，实际上不需要将其从旧的集合导航中删除，然后再将其添加到新的中。 因此，可以从上述示例中的代码更改：

<!--
        var post = vsBlog.Posts.Single(e => e.Title.StartsWith("Disassembly improvements"));
        vsBlog.Posts.Remove(post);
        dotNetBlog.Posts.Add(post);
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_1](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_1)]

更改为：

<!--
        var post = vsBlog.Posts.Single(e => e.Title.StartsWith("Disassembly improvements"));
        dotNetBlog.Posts.Add(post);
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_2](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_2)]

EF Core 看到张贴内容已添加到新博客，并自动将其从第一个博客上的集合中删除。

### <a name="removing-from-a-collection-navigation"></a>从集合导航中移除

从主体/父对象的集合导航中删除从属/子实体会导致与该主体/父对象的断开。 接下来发生的情况取决于关系是可选的还是必需的。

#### <a name="optional-relationships"></a>可选关系

默认情况下，对于可选关系，外键值设置为 null。 这意味着依赖/子级不再与 _任何_ 主体/父项关联。 例如，让我们加载一个博客和文章，然后从集合导航中删除其中一个帖子 `Blog.Posts` ：

<!--
        var post = dotNetBlog.Posts.Single(e => e.Title == "Announcing F# 5");
        dotNetBlog.Posts.Remove(post);
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_3](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_3)]

在此更改后查看 [更改跟踪调试视图](xref:core/change-tracking/debug-views) 显示：

- `Post.BlogId`FK 已设置为 null (`BlogId: <null> FK Modified Originally 1`) 
- `Post.Blog`已将引用导航设置为 null (`Blog: <null>`) 
- 已从 `Blog.Posts` 集合导航 (中删除了 post `Posts: [{Id: 1}]`) 

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

请注意，post _未_ 标记为 `Deleted` 。 它被标记为， `Modified` 以便在调用 SaveChanges 时，数据库中的 FK 值将设置为 null。

#### <a name="required-relationships"></a>必选关系

 (不允许将 FK 值设置为 null，并且通常不能) 所需的关系。 因此，断开是必需的关系，这意味着依赖/子实体必须对新的主体/父实体具有重新父级，或在调用 SaveChanges 时从数据库中删除，以避免引用约束冲突。 这称为 "删除孤立项"，并且是所需关系 EF Core 中的默认行为。

例如，让我们将博客和帖子之间的关系更改为必需，然后运行与上一示例相同的代码：

<!--
        var post = dotNetBlog.Posts.Single(e => e.Title == "Announcing F# 5");
        dotNetBlog.Posts.Remove(post);
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_4](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/RequiredRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_4)]

在此更改后查看调试视图显示：

- 此 post 已标记为 `Deleted` ，在调用 SaveChanges 时，将从数据库中删除它。
- `Post.Blog`已将引用导航设置为 null (`Blog: <null>`) 。
- 已从 `Blog.Posts` 集合导航 () 中删除了 post `Posts: [{Id: 1}]` 。

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

请注意， `Post.BlogId` 由于所需的关系，不能将保持不变，因为它不能设置为 null。

正在删除孤立的日志中的调用 SaveChanges 结果：

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='2' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Posts"
WHERE "Id" = @p0;
SELECT changes();
```

#### <a name="delete-orphans-timing-and-re-parenting"></a>删除孤立计时并重新进行父级

默认情况下，会在 `Deleted` [检测到](xref:core/change-tracking/change-detection)关系更改时立即标记遗孤。 但是，此过程可以延迟到实际调用 SaveChanges。 这有助于避免对已从一个主体/父节点中删除的实体进行孤立，但在调用 SaveChanges 之前，将使用新的主体/父项对其进行重新设置父级。 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DeleteOrphansTiming?displayProperty=nameWithType> 用于设置此计时。 例如：

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

从第一个集合中删除 post 后，该对象不会标记为 `Deleted` 上一示例中的。 相反，EF Core 会跟踪关系是否被断开， _即使这是必需的关系_ 也是如此。  (FK 值被 EF Core 视为 null，即使该类型不可为 null，也是如此。 这称为 "概念 null"。 ) 

```output
Post {Id: 3} Modified
  Id: 3 PK
  BlogId: <null> FK Modified Originally 2
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: <null>
  Tags: []
```

此时调用 SaveChanges 会导致删除孤立的 post。 但是，如果上面的示例中所示，post 将在调用 SaveChanges 之前与新的博客相关联，然后它将相应地固定到新博客并不再被视为孤立：

```output
Post {Id: 3} Modified
  Id: 3 PK
  BlogId: 1 FK Modified Originally 2
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: {Id: 1}
  Tags: []
```

此时调用的 SaveChanges 会更新数据库中的 post，而不是将其删除。

还可以关闭自动删除孤立项。 如果在跟踪孤立时调用 SaveChanges，这将导致异常。 例如，以下代码：

<!--
                var dotNetBlog = context.Blogs.Include(e => e.Posts).Single(e => e.Name == ".NET Blog");

                context.ChangeTracker.DeleteOrphansTiming = CascadeTiming.Never;

                var post = dotNetBlog.Posts.Single(e => e.Title == "Announcing F# 5");
                dotNetBlog.Posts.Remove(post);

                context.SaveChanges(); // Throws
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_6](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/RequiredRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_6)]

将引发此异常：

> InvalidOperationException：实体 "博客" 和 "Post" （其键值为 "{BlogId： 1}"）之间的关联已被断开，但该关系已标记为必需或被隐式要求，因为外键不可为 null。 如果在所需的关系被切断时应删除从属/子实体，请将关系配置为使用级联删除。

可以通过调用来随时强制删除孤立项以及级联删除 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeChanges?displayProperty=nameWithType> 。 将此与 "删除孤立计时" 设置相结合 `Never` 将确保不会删除孤立项，除非明确指示 EF Core。

### <a name="changing-a-reference-navigation"></a>更改引用导航

更改一对多关系的引用导航与在关系的另一端更改集合导航的效果相同。 将从属/子的引用导航设置为 null 等效于从主体/父对象的集合导航中删除实体。 如前一部分中所述，所有修复和数据库更改都会发生，包括使实体成为孤立实体（如果需要关系）。

#### <a name="optional-one-to-one-relationships"></a>可选的一对一关系

对于一对一关系，更改引用导航将导致任何以前的关系被断开。 对于可选关系，这意味着前面相关的依赖项/子级上的 FK 值设置为 null。 例如：

<!--
        using var context = new BlogsContext();

        var dotNetBlog = context.Blogs.Include(e => e.Assets).Single(e => e.Name == ".NET Blog");
        dotNetBlog.Assets = new BlogAssets();

        context.ChangeTracker.DetectChanges();
        Console.WriteLine(context.ChangeTracker.DebugView.LongView);

        context.SaveChanges();
-->
[!code-csharp[Fixup_for_added_or_deleted_entities_7](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Fixup_for_added_or_deleted_entities_7)]

在调用 SaveChanges 之前，"调试" 视图会显示新资产已替换现有资产，该资产现在标为 `Modified` 空 `BlogAssets.BlogId` FK 值：

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

这会导致在调用 SaveChanges 时执行更新和插入操作：

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

#### <a name="required-one-to-one-relationships"></a>需要一对一关系

与上一示例中的代码运行相同的代码，但这一次具有所需的一对一关系，将显示先前关联的 `BlogAssets` 现在已标记为 `Deleted` ，因为在新的位置时，它会变得孤立 `BlogAssets` ：

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

这样，当调用 SaveChanges 时，将会出现删除并插入操作：

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

将孤立项标记为已删除的时间可以通过与收集导航显示的相同方式进行更改，并且具有相同的效果。

### <a name="deleting-an-entity"></a>删除实体

#### <a name="optional-relationships"></a>可选关系

当某个实体标记为 `Deleted` （例如通过调用）时，将 <xref:Microsoft.EntityFrameworkCore.DbContext.Remove%2A?displayProperty=nameWithType> 从其他实体的导航中删除对已删除实体的引用。 对于可选关系，从属实体中的 FK 值将设置为 null。

例如，让我们将 Visual Studio 博客标记为 `Deleted` ：

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

在调用 SaveChanges 之前，查看 " [更改跟踪器" 调试视图](xref:core/change-tracking/debug-views) ：

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

请注意：

- 博客标记为 `Deleted` 。
- 与已删除博客相关的资产具有 null FK 值 (`BlogId: <null> FK Modified Originally 2`) 和空引用导航 (`Blog: <null>`) 
- 与已删除博客相关的每个帖子都具有 null FK 值 (`BlogId: <null> FK Modified Originally 2`) 和空引用导航 (`Blog: <null>`) 

#### <a name="required-relationships"></a>必选关系

所需关系的修复行为与可选关系相同，不同之处在于：依赖/子实体被标记为， `Deleted` 因为它们不能在没有主体/父级的情况下存在，并且在调用 SaveChanges 时必须从数据库中删除，以避免引用约束异常。 这称为 "级联删除"，是所需关系 EF Core 中的默认行为。 例如，在前面的示例中运行与前面示例中相同的代码，但在调用 SaveChanges 之前，会生成以下调试视图：

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

按预期，从属项/子项现在标记为 `Deleted` 。 但请注意，已删除的实体上的导航 _未_ 更改。 这似乎很奇怪，但它可以通过清除所有导航来避免完全清除已删除的实体图。 也就是说，即使在删除后，博客、资产和帖子仍会形成实体图。 这样，就可以更轻松地删除实体关系图，而不是在 EF6 中拆分关系图。

#### <a name="cascade-delete-timing-and-re-parenting"></a>级联删除计时和重新父级

默认情况下，当父/主体标记为时，将立即执行级联删除 `Deleted` 。 这与删除孤立项相同，如前文所述。 与删除孤立项一样，此过程可能会延迟，直到通过适当的设置调用 SaveChanges 或完全禁用它 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeDeleteTiming?displayProperty=nameWithType> 。 这种方法的使用方式与删除孤立对象的方式相同，包括删除主体/父项后用于重新父级子/依赖项。

通过调用，可以随时强制执行级联删除操作，以及删除孤立项 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeChanges?displayProperty=nameWithType> 。 将此组合到将级联删除计时设置为 `Never` 将确保级联删除永远不会发生，除非显式指示 EF Core。

> [!TIP]
> 级联删除和删除孤立项是密切相关的。 当与所需的主体/父实体之间的关系断开时，两者都将导致删除从属实体。 对于级联删除，会发生此断开，因为主体/父项本身会被删除。 对于遗孤，主体/父实体仍然存在，但不再与从属/子实体相关。

## <a name="many-to-many-relationships"></a>多对多关系

EF Core 中的多对多关系是使用联接实体实现的。 多对多关系的每一方都与具有一对多关系的此联接实体相关。 在 EF Core 5.0 之前，必须显式定义和映射此联接实体。 从 EF Core 5.0 开始，可以隐式创建并隐藏。 但是，在这两种情况下，基础行为都是相同的。 首先，我们将介绍此基础行为，以了解如何跟踪多对多关系的工作方式。

### <a name="how-many-to-many-relationships-work"></a>多对多关系的工作方式

请考虑这 EF Core 模型，该模型使用显式定义的联接实体类型在 post 和标记之间创建多对多关系：

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

请注意， `PostTag` 联接实体类型包含两个外键属性。 在此模型中，对于要与标记相关的 post，必须有一个 PostTag 联接实体 `PostTag.PostId` ，其中外键值与 `Post.Id` 主键值匹配， `PostTag.TagId` 外键值与 `Tag.Id` 主键值匹配。 例如：

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            context.Add(new PostTag { PostId = post.Id, TagId = tag.Id });

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Many_to_many_relationships_1](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntitySamples.cs?name=Many_to_many_relationships_1)]

运行此代码后查看 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图，以显示 post 和标记是由新的 "联接" 实体相关联的 `PostTag` ：

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

请注意，上的集合导航已 `Post` `Tag` 修复，并已在上进行引用导航 `PostTag` 。 可以通过导航而不是 FK 值来操作这些关系，就像前面的示例中所述。 例如，可以通过在联接实体上设置引用导航来修改上述代码以添加关系：

<!--
            context.Add(new PostTag { Post = post, Tag = tag });
-->
[!code-csharp[Many_to_many_relationships_2](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntitySamples.cs?name=Many_to_many_relationships_2)]

这会导致与上一个示例中的 Fk 和导航完全相同的更改。

### <a name="skip-navigations"></a>跳过导航

> [!NOTE]
> EF Core 5.0 中引入了跳过导航。

手动操作联接表可能比较繁琐。 从 EF Core 5.0 开始，可以使用 "跳过" 联接实体的特殊集合导航直接操作多对多关系。 例如，可以将两个 skip 导航添加到上述模型;一个从 Post 到标记，另一个从标记到 post：

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

这种多对多关系要求使用以下配置，以确保跳过导航和普通导航均用于相同的多对多关系：

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

有关映射多对多关系的详细信息，请参阅 [关系](xref:core/modeling/relationships) 。

跳过导航外观并表现为正常的集合导航。 但是，它们使用外键值的方式不同。 让我们将帖子与标记相关联，但这次使用的是 skip 导航：

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            post.Tags.Add(tag);

            context.ChangeTracker.DetectChanges();
            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Many_to_many_relationships_3](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityAndSkipsSamples.cs?name=Many_to_many_relationships_3)]

请注意，此代码不使用 join 实体。 而只是将实体添加到导航集合中，就像在这是一种一对多关系时所做的一样。 生成的调试视图实质上与以前相同：

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

请注意， `PostTag` 会自动创建一个联接实体实例，并将 FK 值设置为现在关联的标记和 post 的 PK 值。 所有常规引用和集合导航都已修复，可与这些 FK 值匹配。 此外，由于此模型包含跳过导航，因此它们也已修复。 具体而言，即使我们将标记添加到了 " `Post.Tags` 跳过导航"， `Tag.Posts` 也已将此关系的另一端上的反向跳过导航固定到包含关联的 post。

值得一提的是，即使已将跳过导航分层在顶部，也仍可以直接操作基础多对多关系。 例如，标记和帖子可能会与我们在引入 skip 导航之前所做的关联：

<!--
            context.Add(new PostTag { Post = post, Tag = tag });
-->
[!code-csharp[Many_to_many_relationships_4](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityAndSkipsSamples.cs?name=Many_to_many_relationships_4)]

或使用 FK 值：

<!--
            context.Add(new PostTag { PostId = post.Id, TagId = tag.Id });
-->
[!code-csharp[Many_to_many_relationships_5](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityAndSkipsSamples.cs?name=Many_to_many_relationships_5)]

这仍将导致正确地固定 "跳过" 导航，结果与上一示例中的调试视图输出相同。

### <a name="skip-navigations-only"></a>仅跳过导航

在上一部分中，我们添加了跳过导航 _，并_ 完全定义了两个基础一对多关系。 这非常适用于说明 FK 值发生的情况，但通常不需要这样做。 相反，可以使用 " _跳过导航_" 定义多对多关系。 这是在本文档最顶部的模型中定义多对多关系的方式。 使用此模型，我们可以通过将 post 添加到 " `Tag.Posts` 跳过导航" (或者将标记添加到 "跳过导航") 来关联 post 和标记 `Post.Tags` ：

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            post.Tags.Add(tag);

            context.ChangeTracker.DetectChanges();
            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Many_to_many_relationships_6](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/OptionalRelationshipsSamples.cs?name=Many_to_many_relationships_6)]

做出此更改后，查看 "调试" 视图会显示 EF Core 已创建了一个 `Dictionary<string, object>` 用于表示联接实体的实例。 此联接实体包含 `PostsId` 和 `TagsId` 外键属性，这些属性已设置为与关联的 post 和标记的 PK 值相匹配。

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

有关隐式联接实体和实体类型的使用的详细信息，请参阅 [关系](xref:core/modeling/relationships) `Dictionary<string, object>` 。

> [!IMPORTANT]
> 用于按约定联接实体类型的 CLR 类型在将来的版本中可能会更改以提高性能。 不要依赖于联接类型， `Dictionary<string, object>` 除非显式配置了此类型。

### <a name="join-entities-with-payloads"></a>将实体与负载联接

到目前为止，所有示例均使用联接实体类型 (显式或隐式) 只包含多对多关系所需的两个外键属性。 当处理关系时，这两个 FK 值都需要由应用程序显式设置，因为它们的值来自相关实体的主键属性。 这允许 EF Core 创建联接实体的实例，而不会丢失数据。

#### <a name="payloads-with-generated-values"></a>具有生成值的负载

EF Core 支持向联接实体类型添加其他属性。 这就是所谓的 "有效负载" 实体。 例如，让我们将 `TaggedOn` 属性添加到 `PostTag` 联接实体：

<!--
    public class PostTag
    {
        public int PostId { get; set; } // Foreign key to Post
        public int TagId { get; set; } // Foreign key to Tag

        public DateTime TaggedOn { get; set; } // Payload
    }
-->
[!code-csharp[Model](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityWithPayloadSamples.cs?name=Model)]

EF Core 创建联接实体实例时，将不会设置此负载属性。 处理此情况的最常见方法是使用负载属性和自动生成的值。 例如，在 `TaggedOn` 插入每个新实体时，可以将属性配置为使用存储生成的时间戳：

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

现在，可以使用与之前相同的方式来标记 post：

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            post.Tags.Add(tag);

            context.SaveChanges();

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Many_to_many_relationships_7](../../../samples/core/ChangeTracking/ChangingFKsAndNavigations/ExplicitJoinEntityWithPayloadSamples.cs?name=Many_to_many_relationships_7)]

在调用 SaveChanges 后查看 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图会显示已正确设置了负载属性：

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

#### <a name="explicitly-setting-payload-values"></a>显式设置负载值

在前面的示例中，让我们添加一个不使用自动生成的值的负载属性：

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

现在，可以使用与之前相同的方式来标记 post，并且仍将自动创建联接实体。 然后，可以使用 [访问跟踪的实体](xref:core/change-tracking/entity-entries)中所述的一种机制来访问此实体。 例如，下面的代码使用 <xref:Microsoft.EntityFrameworkCore.DbSet%601.Find%2A?displayProperty=nameWithType> 访问联接实体实例：

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

找到联接实体后，可以通过以下方式进行操作：在此示例中，在 `TaggedBy` 调用 SaveChanges 之前设置负载属性。

> [!NOTE]
> 请注意，若要在 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DetectChanges?displayProperty=nameWithType> 使用之前允许 EF Core 检测导航属性更改和创建联接实体实例，需要在此处进行调用 `Find` 。 有关详细信息，请参阅 [更改检测和通知](xref:core/change-tracking/change-detection) 。

或者，可以显式创建联接实体，以将 post 与标记相关联。 例如：

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

最后，设置负载数据的另一种方法是重写 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 或使用 <xref:Microsoft.EntityFrameworkCore.DbContext.SavingChanges?displayProperty=nameWithType> 事件来处理实体，然后再更新数据库。 例如：

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
