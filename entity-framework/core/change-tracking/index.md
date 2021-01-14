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
# <a name="change-tracking-in-ef-core"></a>EF Core 中的更改跟踪

每个 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例跟踪对实体所做的更改。 在调用 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 时，这些跟踪的实体会相应地驱动对数据库的更改。

本文档概述了 Entity Framework Core (EF Core) 更改跟踪，以及它如何与查询和更新相关。

> [!TIP]
> [从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/ChangeTrackingInEFCore)，你可以运行并调试到本文档中的所有代码。

> [!TIP]
> 为简单起见，本文档使用和引用同步方法（如 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A>），而不是它们的异步等效方法（如 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A>）。 除非另有说明，否则可以替换调用并等待异步方法。

## <a name="how-to-track-entities"></a>如何跟踪实体

实体实例在以下情况下会被跟踪：

- 从针对数据库执行的查询返回
- 通过 `Add`、`Attach`、`Update` 或类似方法显示附加到 DbContext
- 检测为连接到现有跟踪实体的新实体

实体实例在以下情况下不再被跟踪：

- DbContext 已释放
- 更改跟踪器已被清除（EF Core 5.0 及更高版本）
- 显式拆离实体

DbContext 旨在表示短期工作单元，如 [DbContext 初始化和配置](xref:core/dbcontext-configuration/index)中所述。 这意味着释放 DbContext 是停止跟踪实体的正常方式。 换句话说，DbContext 的生存期应为：

1. 创建 DbContext 实例
2. 跟踪某些实体
3. 对实体进行一些更改
4. 调用 SaveChanges 以更新数据库
5. 释放 DbContext 实例

> [!TIP]
> 采用此方法时，无需清除更改跟踪器或显式拆离实体实例。 但是，如果确实需要拆离实体，则调用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Clear%2A?displayProperty=nameWithType> 比逐个拆离实体更有效。

## <a name="entity-states"></a>实体状态

每个实体都与给定 <xref:Microsoft.EntityFrameworkCore.EntityState> 相关联：

- `Detached` 实体未被 <xref:Microsoft.EntityFrameworkCore.DbContext> 跟踪。
- `Added` 实体是新实体，并且尚未插入到数据库中。 这意味着它们将在调用 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 时插入。
- `Unchanged` 实体自从数据库中查询以来尚未进行更改。 从查询返回的所有实体最初都处于此状态。
- `Modified` 实体自从数据库中查询以来已进行更改。 这意味着它们将在调用 SaveChanges 时更新。
- `Deleted` 实体存在于数据库中，但标记为在调用 SaveChanges 时删除。

EF Core 跟踪属性级别的更改。 例如，如果只修改单个属性值，则数据库更新将仅更改该值。 但是，当实体本身处于“已修改”状态时，只能将属性标记为已修改。 （或者，从另一角度来看，“已修改”状态意味着至少有一个属性值已标记为已修改。）

下表汇总了不同的状态：

| 实体状态     | 由 DbContext 跟踪 | 存在于数据库中 | 属性已修改 | SaveChanges 上的操作
|:-----------------|----------------------|--------------------|---------------------|-----------------------
| `Detached`       | 否                   | -                  | -                   | -
| `Added`          | 是                  | 否                 | -                   | 插入
| `Unchanged`      | 是                  | 是                | 否                  | -
| `Modified`       | 是                  | 是                | 是                 | 更新
| `Deleted`        | 是                  | 是                | -                   | 删除

> [!NOTE]
> 为清楚起见，此文本使用了关系数据库术语。 NoSQL 数据库通常支持类似操作，但可能具有不同的名称。 有关详细信息，请查阅数据库提供程序文档。

## <a name="tracking-from-queries"></a>从查询跟踪

当同一个 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例同时用于查询实体并通过调用 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 更新它们时，EF Core 更改跟踪的效果最佳。 这是因为 EF Core 会自动跟踪已查询实体的状态，然后在调用 SaveChanges 时检测对这些实体所做的任何更改。

与[显式跟踪实体实例](xref:core/change-tracking/explicit-tracking)相比，此方法具有多个优点：

- 该实例为简单类型。 实体状态极少需要显式操作，EF Core 负责处理状态更改。
- 更新仅限于那些实际已更改的值。
- [卷影属性](xref:core/modeling/shadow-properties)的值将保留，并根据需要使用。 当外键以卷影状态存储时，这一点尤其重要。
- 自动保留属性的原始值，并将其用于有效更新。

## <a name="simple-query-and-update"></a>简单查询和更新

例如，请考虑一个简单的博客/帖子模型：

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

我们可以使用此模型来查询博客和帖子，然后对数据库进行一些更新：

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

使用 SQLite 作为示例数据库，调用 SaveChanges 会生成以下数据库更新：

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

[更改跟踪器调试视图](xref:core/change-tracking/debug-views)是一种可以直观看到正在被跟踪的实体及其状态的极佳方式。 例如，在调用 SaveChanges 之前，将以下代码插入到上面的示例中：

<!--
                context.ChangeTracker.DetectChanges();
                Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Simple_query_and_update_2](../../../samples/core/ChangeTracking/ChangeTrackingInEFCore/GeneratedKeysSamples.cs?name=Simple_query_and_update_2)]

生成以下输出：

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

具体而言，请注意：

- `Blog.Name` 属性标记为已修改 (`Name: '.NET Blog (Updated!)' Modified Originally '.NET Blog'`)，这会导致博客处于 `Modified` 状态。
- 帖子 2 的 `Post.Title` 属性标记为已修改 (`Title: 'Announcing F# 5.0' Modified Originally 'Announcing F# 5'`)，这会导致此帖子处于 `Modified` 状态。
- 帖子 2 的其他属性值尚未更改，因此未标记为已修改。 这就是这些值不包含在数据库更新中的原因。
- 不会以任何方式修改另一个帖子。 这就是它仍处于 `Unchanged` 状态且不包括在数据库更新中的原因。

## <a name="query-then-insert-update-and-delete"></a>查询，然后插入、更新和删除

与上一示例中的更新类似，可以将更新与插入和删除合并在同一工作单元中。 例如：

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

在此示例中：

- 从数据库查询博客和相关帖子并进行跟踪
- `Blog.Name` 属性已更改
- 新帖子将添加到博客的现有帖子集合中
- 通过调用 <xref:Microsoft.EntityFrameworkCore.DbContext.Remove%2A?displayProperty=nameWithType> 将现有帖子标记为要删除

在调用 SaveChanges 之前，再次查看[更改跟踪器调试视图](xref:core/change-tracking/debug-views)，会显示 EF Core 跟踪这些更改的方式：

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

请注意：

- 博客标记为 `Modified`。 这将生成一个数据库更新。
- 帖子 2 标记为 `Deleted`。 这将生成一个数据库删除。
- 具有临时 ID 的新帖子与博客 1 相关联，并标记为 `Added`。 这将生成一个数据库插入。

在调用 SaveChanges 时，这会生成以下数据库命令（使用 SQLite）：

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

有关插入和删除实体的详细信息，请参阅[显式跟踪实体](xref:core/change-tracking/explicit-tracking)。 有关 EF Core 如何自动检测此类更改的详细信息，请参阅[更改检测和通知](xref:core/change-tracking/change-detection)。

> [!TIP]
> 调用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.HasChanges?displayProperty=nameWithType> 来确定是否进行了任何更改，这将导致 SaveChanges 对数据库进行更新。 如果 HasChanges 返回 false，则 SaveChanges 将为无选项。
