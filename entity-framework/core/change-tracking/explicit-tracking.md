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
# <a name="explicitly-tracking-entities"></a>显式跟踪实体

每个 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例跟踪对实体所做的更改。 在调用 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 时，这些跟踪的实体会相应地驱动对数据库的更改。

Entity Framework Core (EF Core 当同一 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例同时用于查询实体并通过调用更新实体时，) 更改跟踪的效果最佳。 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 这是因为 EF Core 会自动跟踪已查询实体的状态，然后在调用 SaveChanges 时检测对这些实体所做的任何更改。 [EF Core 更改跟踪中](xref:core/change-tracking/index)介绍了这种方法。

> [!TIP]
> 本文档假设了解 EF Core 更改跟踪的实体状态和基础知识。 有关这些主题的详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。

> [!TIP]
> [从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangeTrackingInEFCore)，你可以运行并调试到本文档中的所有代码。

> [!TIP]
> 为简单起见，本文档使用和引用同步方法（如 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A>），而不是它们的异步等效方法（如 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A>）。 除非另有说明，否则可以替换调用并等待异步方法。

## <a name="introduction"></a>简介

实体可以显式 "附加" 到，以便 <xref:Microsoft.EntityFrameworkCore.DbContext> 上下文随后跟踪这些实体。 这主要适用于以下情况：

1. 创建将插入到数据库中的新实体。
2. 重新附加先前由 _其他_ DbContext 实例查询的已断开连接的实体。

大多数应用程序都需要其中的第一个应用程序，并且主要由方法进行处理 <xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType> 。

第二个只是在 **_未跟踪实体时_** 更改实体或其关系的应用程序所必需的。 例如，web 应用程序可能会将实体发送到 web 客户端，用户在该客户端进行更改并将其发送回。 这些实体被称为 "断开连接"，因为最初从 DbContext 查询它们，但在发送到客户端时，这些实体会与该上下文断开连接。

Web 应用程序现在必须重新附加这些实体，以便再次跟踪这些实体，并指示已进行的更改，以便对 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 数据库进行相应的更新。 这主要由 <xref:Microsoft.EntityFrameworkCore.DbContext.Attach%2A?displayProperty=nameWithType> 和方法来处理 <xref:Microsoft.EntityFrameworkCore.DbContext.Update%2A?displayProperty=nameWithType> 。

> [!TIP]
> 通常不需要将实体附加到从中查询它们的 _同一 DbContext 实例_ 。 不要定期执行无跟踪查询，然后将返回的实体附加到同一个上下文。 这会比使用跟踪查询慢，并且可能还会导致诸如缺少影子属性值之类的问题，从而使其更难正确地使用。

### <a name="generated-versus-explicit-key-values"></a>生成的值与显式键值

默认情况下，integer 和 GUID [键属性](xref:core/modeling/keys) 配置为使用 [自动生成的键值](xref:core/modeling/generated-properties)。 这 **对于更改跟踪具有主要优势：未设置的键值指示该实体为 "new"**。 通过 "新"，我们表示尚未将其插入到数据库中。

以下各节将使用两个模型。 第一个配置为 **不** 使用生成的键值：

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

在每个示例中，将在每个示例中首先显示非生成的 (，即显式设置) 键值，因为所有内容都非常明确且易于理解。 然后，在此示例中，将使用生成的键值：

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

请注意，此模型中的键属性不需要进行额外配置，因为使用生成的键值是 [简单整数键的默认](xref:core/modeling/generated-properties)值。

## <a name="inserting-new-entities"></a>插入新实体

### <a name="explicit-key-values"></a>显式键值

必须在 `Added` 要插入的状态中跟踪实体 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。 实体通常通过调用、、、 <xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType> <xref:Microsoft.EntityFrameworkCore.DbContext.AddRange%2A?displayProperty=nameWithType> 或中 <xref:Microsoft.EntityFrameworkCore.DbContext.AddAsync%2A?displayProperty=nameWithType> <xref:Microsoft.EntityFrameworkCore.DbContext.AddRangeAsync%2A?displayProperty=nameWithType> 的等效方法 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 之一置于已添加状态。

> [!TIP]
> 这些方法在更改跟踪上下文中的工作方式相同。 有关详细信息，请参阅 [其他更改跟踪功能](xref:core/change-tracking/miscellaneous) 。

例如，若要开始跟踪新博客：

<!--
            context.Add(
                new Blog
                {
                    Id = 1,
                    Name = ".NET Blog",
                });
-->
[!code-csharp[Inserting_new_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Inserting_new_entities_1)]

检查此调用后的 [更改跟踪器调试视图](xref:core/change-tracking/debug-views) 显示上下文正在跟踪状态中的新实体 `Added` ：

```output
Blog {Id: 1} Added
  Id: 1 PK
  Name: '.NET Blog'
  Posts: []
```

但是，Add 方法并不只是处理单个实体。 它们实际开始跟踪 _相关实体的整个关系图_，并将其全部置于 `Added` 状态。 例如，若要插入新的博客和关联的新文章，请执行以下操作：

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

上下文现在正在跟踪所有这些实体，如下所示 `Added` ：

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

请注意，已为 `Id` 上述示例中的键属性设置了显式值。 这是因为，此处的模型已配置为使用显式设置的键值，而不是自动生成的键值。 在不使用生成的键时，必须在调用 _之前_ 显式设置键属性 `Add` 。 然后，在调用 SaveChanges 时插入这些键值。 例如，使用 SQLite 时：

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

在 SaveChanges 完成后，所有这些实体都将在状态中跟踪 `Unchanged` ，因为这些实体现在存在于数据库中：

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

### <a name="generated-key-values"></a>生成的键值

如上所述，默认情况下，integer 和 GUID [键属性](xref:core/modeling/keys) 配置为使用 [自动生成的键值](xref:core/modeling/generated-properties) 。 这意味着应用程序 _不能显式设置任何键值_。 例如，若要插入新的博客并使用生成的键值发送所有内容，请执行以下操作：

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

对于显式键值，上下文现在正在跟踪所有这些实体，如下所示 `Added` ：

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

请注意，在这种情况下，已为每个实体生成了 [临时键值](xref:core/change-tracking/miscellaneous#temporary-values) 。 在调用 SaveChanges 之前，EF Core 会使用这些值，此时将从数据库中读回实际键值。 例如，使用 SQLite 时：

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

SaveChanges 完成后，所有实体都已使用其真实密钥值进行了更新，并在状态下进行跟踪， `Unchanged` 因为它们现在与数据库中的状态匹配：

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

这与上一个使用显式键值的示例的最终状态完全相同。

> [!TIP]
> 即使使用生成的键值，也仍可以设置显式键值。 然后 EF Core 将尝试使用此键值进行插入。 某些数据库配置（包括带有标识列的 SQL Server）不支持此类插入，并将引发 ([请参阅这些文档，了解解决方法](xref:core/providers/sql-server/value-generation#inserting-explicit-values-into-identity-columns)) 。

## <a name="attaching-existing-entities"></a>附加现有实体

### <a name="explicit-key-values"></a>显式键值

从查询返回的实体将在状态中跟踪 `Unchanged` 。 `Unchanged`状态表示该实体自查询后未进行修改。 断开连接的实体（可能从 HTTP 请求中的 web 客户端返回）可以使用 <xref:Microsoft.EntityFrameworkCore.DbContext.Attach%2A?displayProperty=nameWithType> 、 <xref:Microsoft.EntityFrameworkCore.DbContext.AttachRange%2A?displayProperty=nameWithType> 或中的等效方法进入此状态 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 。 例如，若要开始跟踪现有博客：

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
> 此处的示例是为了简单起见，显式创建实体 `new` 。 通常情况下，实体实例将来自其他源，如从客户端反序列化，或者是从 HTTP Post 中的数据创建的。

检查此调用后的 [更改跟踪](xref:core/change-tracking/debug-views) 器 "调试" 视图会显示在状态下跟踪实体 `Unchanged` ：

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Posts: []
```

就像一样 `Add` ， `Attach` 实际将连接的实体的整个关系图设置为 `Unchanged` 状态。 例如，要附加现有博客和关联的现有文章：

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

上下文现在正在跟踪所有这些实体，如下所示 `Unchanged` ：

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

此时调用 SaveChanges 将不起作用。 所有实体都标记为 `Unchanged` ，因此数据库中没有要更新的内容。

### <a name="generated-key-values"></a>生成的键值

如上所述，默认情况下，integer 和 GUID [键属性](xref:core/modeling/keys) 配置为使用 [自动生成的键值](xref:core/modeling/generated-properties) 。 当使用断开连接的实体时，这有一个主要优势：未设置的键值指示该实体尚未插入到数据库中。 这允许更改跟踪器自动检测新实体并使其处于 `Added` 状态。 例如，请考虑附加博客和文章的此图：

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

博客的键值为1，表示它已存在于数据库中。 两篇文章中的键值也已设置，但第三篇文章并未设置。 EF Core 会将此键值视为0，这是一个整数的 CLR 默认值。 这会导致 EF Core 将新实体标记为 `Added` 而不是 `Unchanged` ：

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

此时调用 SaveChanges 不会对实体执行任何操作 `Unchanged` ，但会将新实体插入到数据库中。 例如，使用 SQLite 时：

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='1' (DbType = String), @p1='.NET 5.0 includes many enhancements, including single file applications, more...' (Size = 80), @p2='Announcing .NET 5.0' (Size = 19)], CommandType='Text', CommandTimeout='30']
INSERT INTO "Posts" ("BlogId", "Content", "Title")
VALUES (@p0, @p1, @p2);
SELECT "Id"
FROM "Posts"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();
```

此处需要注意的要点是，使用生成的键值，EF Core 能够 **自动区分断开连接图形中的现有实体**。 简而言之，使用生成的密钥时，当实体没有设置键值时，EF Core 将始终插入实体。

## <a name="updating-existing-entities"></a>更新现有实体

### <a name="explicit-key-values"></a>显式键值

<xref:Microsoft.EntityFrameworkCore.DbContext.Update%2A?displayProperty=nameWithType>、 <xref:Microsoft.EntityFrameworkCore.DbContext.UpdateRange%2A?displayProperty=nameWithType> 和的等效方法的 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 行为与 `Attach` 上述方法完全相同，只是将实体放入 `Modfied` 而不是 `Unchanged` 状态。 例如，若要开始跟踪现有博客，请 `Modified` 执行以下操作：

<!--
            context.Update(
                new Blog
                {
                    Id = 1,
                    Name = ".NET Blog",
                });
-->
[!code-csharp[Updating_existing_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Updating_existing_entities_1)]

检查此调用后的 [更改跟踪器调试视图](xref:core/change-tracking/debug-views) 表明上下文正在跟踪此实体的 `Modified` 状态：

```output
Blog {Id: 1} Modified
  Id: 1 PK
  Name: '.NET Blog' Modified
  Posts: []
```

与和一样 `Add` `Attach` ， `Update` 实际上将相关实体的 _整个关系图_ 标记为 `Modified` 。 例如，要将现有博客和关联的现有帖子附加为 `Modified` ：

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

上下文现在正在跟踪所有这些实体，如下所示 `Modified` ：

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

此时调用 SaveChanges 将导致为所有这些实体将更新发送到数据库。 例如，使用 SQLite 时：

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

### <a name="generated-key-values"></a>生成的键值

与一样 `Attach` ，生成的键值与相同的主要优点是 `Update` ：未设置的键值指示该实体是新实体，尚未插入到数据库中。 与一样 `Attach` ，这允许 DbContext 自动检测新实体并使其处于 `Added` 状态。 例如，请考虑 `Update` 使用博客和文章的此图调用：

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

与示例一样 `Attach` ，不包含键值的 post 被检测为 new，并设置为 `Added` 状态。 其他实体标记为 `Modified` ：

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

`SaveChanges`此时调用将导致在插入新实体时将更新发送到数据库中的所有现有实体。 例如，使用 SQLite 时：

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

这是一种从断开连接的图形生成更新和插入的非常简单的方法。 但是，这将导致为每个被跟踪实体的每个属性将更新或插入发送到数据库，即使某些属性值可能未更改也是如此。 这不太恐惧;对于包含小型图形的许多应用程序，这是生成更新的一种简单且实用的方法。 换句话说，其他更复杂的模式有时会导致更新的更新，如 EF Core 中的 [标识解析](xref:core/change-tracking/identity-resolution)中所述。

## <a name="deleting-existing-entities"></a>删除现有实体

对于要通过 SaveChanges 删除的实体，必须在状态中跟踪该实体 `Deleted` 。 实体通常 `Deleted` 通过调用 <xref:Microsoft.EntityFrameworkCore.DbContext.Remove%2A?displayProperty=nameWithType> 、 <xref:Microsoft.EntityFrameworkCore.DbContext.RemoveRange%2A?displayProperty=nameWithType> 或中的等效方法之一置于状态 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 。 例如，要将现有 post 标记为 `Deleted` ：

<!--
            context.Remove(
                new Post
                {
                    Id = 2
                });
-->
[!code-csharp[Deleting_existing_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Deleting_existing_entities_1)]

检查此调用后的 [更改跟踪器调试视图](xref:core/change-tracking/debug-views) 表明上下文正在跟踪状态中的实体 `Deleted` ：

```output
Post {Id: 2} Deleted
  Id: 2 PK
  BlogId: <null> FK
  Content: <null>
  Title: <null>
  Blog: <null>
```

此实体将在调用 SaveChanges 时删除。 例如，使用 SQLite 时：

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='2' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Posts"
WHERE "Id" = @p0;
SELECT changes();
```

SaveChanges 完成后，删除的实体将从 DbContext 中分离出来，因为它不再存在于数据库中。 因此，调试视图为空，因为没有要跟踪的实体。

## <a name="deleting-dependentchild-entities"></a>删除从属/子实体

从关系图中删除从属/子实体比删除主体/父实体更直接。 有关详细信息，请参阅下一部分和 [更改外键和导航](xref:core/change-tracking/relationship-changes) 。

对 `Remove` 使用创建的实体调用是异常的 `new` 。 此外，与和不同，对尚未 `Add` `Attach` `Update` `Remove` 在或状态中跟踪的实体调用是罕见的 `Unchanged` `Modified` 。 通常，可以跟踪相关实体的单个实体或图形，然后对 `Remove` 应删除的实体调用。 此跟踪实体的图形通常通过以下方法之一创建：

1. 针对实体运行查询
2. `Attach` `Update` 如前面几节中所述，在断开连接的实体图上使用或方法。

例如，上一部分中的代码更有可能从客户端获得 post，然后执行如下所示的操作：

<!--
            context.Attach(post);
            context.Remove(post);
-->
[!code-csharp[Deleting_dependent_child_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Deleting_dependent_child_entities_1)]

此行为与上一个示例的行为完全相同，因为 `Remove` 对未跟踪的实体调用会使其首先附加，然后标记为 `Deleted` 。

在更逼真的示例中，首先附加实体的图形，然后将其中一些实体标记为已删除。 例如：

<!--
            // Attach a blog and associated posts
            context.Attach(blog);

            // Mark one post as Deleted
            context.Remove(blog.Posts[1]);
-->
[!code-csharp[Deleting_dependent_child_entities_2](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Deleting_dependent_child_entities_2)]

所有实体都标记为 `Unchanged` ，但调用的实体除外 `Remove` ：

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

此实体将在调用 SaveChanges 时删除。 例如，使用 SQLite 时：

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='2' (DbType = String)], CommandType='Text', CommandTimeout='30']
DELETE FROM "Posts"
WHERE "Id" = @p0;
SELECT changes();
```

SaveChanges 完成后，删除的实体将从 DbContext 中分离出来，因为它不再存在于数据库中。 其他实体保持 `Unchanged` 状态：

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

## <a name="deleting-principalparent-entities"></a>删除主体/父实体

连接两个实体类型的每个关系都具有一个主体或父端以及一个依赖项或子端。 从属实体或子实体是具有外键属性的实体。 在一对多关系中，主体/父对象位于 "一" 方，从属/子级位于 "多" 端。 有关详细信息，请参阅 [关系](xref:core/modeling/relationships) 。

在前面的示例中，我们删除了 post，这是博客文章一对多关系中的依赖/子实体。 这比较简单，因为删除依赖/子实体不会对其他实体产生任何影响。 另一方面，删除主体/父实体还必须影响任何从属/子实体。 如果不这样做，则会保留一个外键值，该外键值引用不再存在的主键值。 这是无效的模型状态，在大多数数据库中导致引用约束错误。

此无效模型状态可以通过两种方式进行处理：

1. 将 FK 值设置为 null。 这表明从属项/子项不再与任何主体/父对象相关。 这是可选关系的默认值，其中外键必须可以为 null。 将 FK 设置为 null 对于必需的关系无效，其中外键通常不可为 null。
2. 删除依赖项/子项。 这是所需关系的默认值，对于可选关系也是有效的。

有关更改跟踪和关系的详细信息，请参阅 [更改外键和导航](xref:core/change-tracking/relationship-changes) 。

### <a name="optional-relationships"></a>可选关系

`Post.BlogId`外键属性在我们所使用的模型中是可以为 null 的。 这意味着关系是可选的，因此，EF Core 的默认行为是在 `BlogId` 删除博客时将外键属性设置为 null。 例如：

<!--
            // Attach a blog and associated posts
            context.Attach(blog);

            // Mark one post as Deleted
            context.Remove(blog.Posts[1]);
-->
[!code-csharp[Deleting_principal_parent_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysSamples.cs?name=Deleting_principal_parent_entities_1)]

按照预期，在调用后检查 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图以 `Remove` 显示，博客现在标记为 `Deleted` ：

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

更有趣的是，所有相关文章现在都标记为 `Modified` 。 这是因为每个实体中的外键属性都设置为 null。 在删除博客之前，调用 SaveChanges 会将数据库中每个发布的外键值更新为 null：

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

SaveChanges 完成后，删除的实体将从 DbContext 中分离出来，因为它不再存在于数据库中。 其他实体现在标记为 `Unchanged` 具有 null 外键值，这与数据库的状态相匹配：

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

### <a name="required-relationships"></a>必选关系

如果 `Post.BlogId` 外键属性不可为 null，则博客和文章之间的关系将变为 "必需"。 在这种情况下，默认情况下，EF Core 将在删除主体/父实体时删除从属实体/子实体。 例如，删除包含相关文章的博客，如前一示例中所示：

<!--
            // Attach a blog and associated posts
            context.Attach(blog);

            // Mark one post as Deleted
            context.Remove(blog.Posts[1]);
-->
[!code-csharp[Deleting_principal_parent_entities_1](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/ExplicitKeysRequiredSamples.cs?name=Deleting_principal_parent_entities_1)]

按照预期，在调用后检查 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图以 `Remove` 显示，博客再次标记为 `Deleted` ：

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

更有趣的是，在这种情况下，所有相关的文章也已标记为 `Deleted` 。 调用 SaveChanges 会导致从数据库中删除博客和所有相关的发布：

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

SaveChanges 完成后，所有删除的实体将从 DbContext 中分离出来，因为它们不再存在于数据库中。 因此，"调试" 视图的输出为空。

> [!NOTE]
> 本文档仅对使用 EF Core 中的关系的图面进行了划痕。 若要详细了解如何在调用 SaveChanges 时更新/删除从属/子实体，请参阅 [关系](xref:core/modeling/relationships) 了解建模关系的详细信息，以及 [更改外键和导航](xref:core/change-tracking/relationship-changes) 。

## <a name="custom-tracking-with-trackgraph"></a>自定义跟踪与 TrackGraph

<xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.TrackGraph%2A?displayProperty=nameWithType> 的工作方式类似于 `Add` `Attach` 和， `Update` 只不过它在跟踪之前为每个实体实例生成一个回调。 这允许在确定如何跟踪图形中的各个实体时使用自定义逻辑。

例如，请考虑在使用生成的键值跟踪实体时使用的规则 EF Core：如果键值为零，则实体为 new，应插入该实体。 让我们扩展此规则，以指示键值是否为负，然后应删除该实体。 这样，我们便可以更改断开连接图形的实体中的主键值，以标记已删除的实体：

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

然后，可以使用 TrackGraph 跟踪此断开连接的图形：

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

对于图形中的每个实体，上面的代码会 _在跟踪实体之前_ 检查主键值。 对于 unset (零) 键值，代码执行 EF Core 通常会执行的操作。 也就是说，如果未设置密钥，则实体将标记为 `Added` 。 如果设置了该键并且值为非负值，则实体将标记为 `Modified` 。 但是，如果找到负键值，则会还原其真实的非负值，并将实体作为进行跟踪 `Deleted` 。

运行此代码的输出为：

```output
Tracking Blog with key value 1 as Modified
Tracking Post with key value 1 as Modified
Tracking Post with key value -2 as Deleted
Tracking Post with key value 0 as Added
```

> [!NOTE]
> 为简单起见，此代码假定每个实体都有一个名为的整数主键属性 `Id` 。 这可能会整理到抽象基类或接口中。 或者，可以从元数据中获取主键属性， <xref:Microsoft.EntityFrameworkCore.Metadata.IEntityType> 以便此代码适用于任何类型的实体。

TrackGraph 有两个重载。 在上面使用的简单重载中，EF Core 确定何时停止遍历关系图。 具体而言，它将停止从给定实体访问新的相关实体（如果该实体已被跟踪）或回调不会开始跟踪实体。

高级重载 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.TrackGraph%60%601(System.Object,%60%600,System.Func{Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntryGraphNode{%60%600},System.Boolean})?displayProperty=nameWithType> 具有返回布尔值的回调。 如果回调返回 false，则图形遍历将停止，否则将继续。 使用此重载时，必须小心避免出现无限循环。

高级重载还允许向 TrackGraph 提供状态，并将此状态传递给每个回调。
