---
title: 级联删除 - EF Core
description: 配置从主体/父实体删除或断开时触发的级联行为
author: ajcvickers
ms.date: 01/07/2021
uid: core/saving/cascade-delete
ms.openlocfilehash: 7c35de900930cf42da0e534df76124b5fb19ca52
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128857"
---
# <a name="cascade-delete"></a>级联删除

Entity Framework Core (EF Core) 表示使用外键的关系。 具有外键的实体是关系中的子实体或依赖实体。 此实体的外键值必须与相关主体/父实体的主键值（或替换键值）匹配。

如果删除主体/父实体，则依赖项/子项的外键值将不再匹配任何主体/父实体的主键或替换键。 这是无效状态，将导致在大多数数据库中出现引用约束冲突。

可通过两种方法来避免此引用约束冲突：

1. 将外键值设置为 null
2. 同时删除依赖实体/子实体

第一个选项仅适用于其中外键属性（及其映射到的数据库列）必须可为 null 的可选关系。

第二个选项适用于任何类型的关系，它被称作“级联删除”。

> [!TIP]
> 本文档从更新数据库的角度介绍级联删除（和删除孤立项）。 本文大量使用在[在 EF Core 中更改跟踪](xref:core/change-tracking/index)和[更改外键和导航](xref:core/change-tracking/relationship-changes)文章中介绍的概念。 请确保在此处处理材料之前充分了解这些概念。

> [!TIP]  
> 通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/CascadeDeletes)，你可运行并调试到本文档中的所有代码。

## <a name="when-cascading-behaviors-happen"></a>发生级联行为时

当依赖实体/子实体无法再与其当前主体/父实体关联时，需要执行级联删除。 发生这种情况的原因可能是主体/父实体已被删除，或者当主体/父实体仍存在，但依赖实体/子实体不再与其关联时。

### <a name="deleting-a-principalparent"></a>删除主体/父实体

请考虑此简单模型，其中 `Blog` 是与 `Post`（依赖实体/子实体）的关系中的主体/父实体。 `Post.BlogId` 是外键属性，其值必须与该博客所属文章中的 `Post.Id` 主键匹配。

<!--
    public class Blog
    {
        public int Id { get; set; }

        public string Name { get; set; }

        public IList<Post> Posts { get; } = new List<Post>();
    }

    public class Post
    {
        public int Id { get; set; }

        public string Title { get; set; }
        public string Content { get; set; }

        public int BlogId { get; set; }
        public Blog Blog { get; set; }
    }
-->
[!code-csharp[Model](../../../samples/core/CascadeDeletes/IntroRequiredSamples.cs?name=Model)]

按照约定，由于 `Post.BlogId` 外键属性是不可为 null 的，因此该关系被配置为必需的。 默认情况下，所需的关系配置为使用级联删除。 要详细了解建模关系，请参阅[关系](xref:core/modeling/relationships)。

删除博客时，所有文章都将被级联删除。 例如：

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).Include(e => e.Posts).First();

            context.Remove(blog);
            
            context.SaveChanges();
-->
[!code-csharp[Deleting_principal_parent_1](../../../samples/core/CascadeDeletes/IntroRequiredSamples.cs?name=Deleting_principal_parent_1)]

SaveChanges 以 SQL Server 为例，生成以下 SQL：

```sql
-- Executed DbCommand (1ms) [Parameters=[@p0='1'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [Posts]
WHERE [Id] = @p0;
SELECT @@ROWCOUNT;

-- Executed DbCommand (0ms) [Parameters=[@p0='2'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [Posts]
WHERE [Id] = @p0;
SELECT @@ROWCOUNT;

-- Executed DbCommand (2ms) [Parameters=[@p1='1'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [Blogs]
WHERE [Id] = @p1;
SELECT @@ROWCOUNT;
```

### <a name="severing-a-relationship"></a>断开关系

我们不会删除博客，而是断开每篇文章与其博客之间的关系。 为此，可将每篇文章的引用导航 `Post.Blog` 设置为 null：

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).Include(e => e.Posts).First();

            foreach (var post in blog.Posts)
            {
                post.Blog = null;
            }
            
            context.SaveChanges();
-->
[!code-csharp[Severing_a_relationship_1](../../../samples/core/CascadeDeletes/IntroRequiredSamples.cs?name=Severing_a_relationship_1)]

还可通过从 `Blog.Posts` 集合导航中删除每篇文章内容来断开关系：

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).Include(e => e.Posts).First();

            blog.Posts.Clear();
            
            context.SaveChanges();
-->
[!code-csharp[Severing_a_relationship_2](../../../samples/core/CascadeDeletes/IntroRequiredSamples.cs?name=Severing_a_relationship_2)]

无论哪种情况，结果都一样：没有删除博客，但是删除了不再与任何博客关联的文章：

```sql
-- Executed DbCommand (1ms) [Parameters=[@p0='1'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [Posts]
WHERE [Id] = @p0;
SELECT @@ROWCOUNT;

-- Executed DbCommand (0ms) [Parameters=[@p0='2'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [Posts]
WHERE [Id] = @p0;
SELECT @@ROWCOUNT;
```

删除不再与任何主体/依赖实体关联的实体这一行为被称作“删除孤立项”。

> [!TIP]
> 级联删除和删除孤立项是密切相关的。 当断开与所需的主体/父实体之间的关系时，两者都将导致删除依赖实体/子实体。 对于级联删除，由于主体/父实体本身已删除，因此发生了这种断开。 对于孤立项，主体/父实体仍然存在，但不再与依赖实体/子实体相关。  

## <a name="where-cascading-behaviors-happen"></a>发生级联行为的位置

可将级联行为应用于：

- 当前 <xref:Microsoft.EntityFrameworkCore.DbContext> 跟踪的实体
- 数据库中尚未加载到上下文中的实体

### <a name="cascade-delete-of-tracked-entities"></a>级联删除被跟踪实体

EF Core 始终将配置的级联行为应用于跟踪的实体。 这意味着如上面的示例所示，如果应用程序将所有相关的依赖实体/子实体加载到 DbContext 中，则无论如何配置数据库，都将正确应用级联行为。

> [!TIP]
> 可使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeDeleteTiming?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DeleteOrphansTiming?displayProperty=nameWithType> 控制在被跟踪实体上发生级联行为的确切时间。 有关详细信息，请参阅[更改外键和导航](xref:core/change-tracking/relationship-changes)。

### <a name="cascade-delete-in-the-database"></a>数据库中的级联删除

许多数据库系统还提供在数据库中删除实体时触发的级联行为。 使用 <xref:Microsoft.EntityFrameworkCore.Infrastructure.DatabaseFacade.EnsureCreated%2A> 或 EF Core 迁移创建数据库时，EF Core 会根据 EF Core 模型中的级联删除行为来配置这些行为。 例如，通过上述模型，使用 SQL Server 时将为文章创建下表：

```sql
CREATE TABLE [Posts] (
    [Id] int NOT NULL IDENTITY,
    [Title] nvarchar(max) NULL,
    [Content] nvarchar(max) NULL,
    [BlogId] int NOT NULL,
    CONSTRAINT [PK_Posts] PRIMARY KEY ([Id]),
    CONSTRAINT [FK_Posts_Blogs_BlogId] FOREIGN KEY ([BlogId]) REFERENCES [Blogs] ([Id]) ON DELETE CASCADE
);
```

请注意，定义博客和文章之间关系的外键约束是用 `ON DELETE CASCADE` 配置的。

如果我们知道数据库是这样配置的，那么我们可以删除博客，而无需先加载文章，数据库将负责删除与此博客相关的所有文章。 例如：

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).First();

            context.Remove(blog);
            
            context.SaveChanges();
-->
[!code-csharp[Where_cascading_behaviors_happen_1](../../../samples/core/CascadeDeletes/IntroRequiredSamples.cs?name=Where_cascading_behaviors_happen_1)]

请注意，文章没有 `Include`，因此它们不会被加载。 在这种情况下，SaveChanges 将仅删除博客，因为这是唯一正在跟踪的实体：

```sql
-- Executed DbCommand (6ms) [Parameters=[@p0='1'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [Blogs]
WHERE [Id] = @p0;
SELECT @@ROWCOUNT;
```

如果未针对级联删除配置数据库中的外键约束，则将导致异常。 但在这种情况下，数据库删除了文章，因为它在创建时是用 `ON DELETE CASCADE` 配置的。

> [!NOTE]
> 数据库通常没有任何自动删除孤立项的方法。 这是因为虽然 EF Core 使用导航以及外键来表示关系，但是数据库仅具有外键而没有导航。 这意味着通常无法在不将双方都加载到 DbContext 的情况下断开关系。

> [!NOTE]
> EF Core 内存中数据库当前不支持数据库中的级联删除。

> [!WARNING]
> 软删除实体时，请勿在数据库中配置级联删除。 这可能会导致实体被意外删除，而不是软删除。

### <a name="database-cascade-limitations"></a>数据库级联限制

一些数据库（最突出的是 SQL Server）对形成周期的级联行为有限制。 例如，请考虑以下模型：

<!--
    public class Blog
    {
        public int Id { get; set; }
        public string Name { get; set; }

        public IList<Post> Posts { get; } = new List<Post>();
        
        public int OwnerId { get; set; }
        public Person Owner { get; set; }
    }

    public class Post
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }

        public int BlogId { get; set; }
        public Blog Blog { get; set; }
        
        public int AuthorId { get; set; }
        public Person Author { get; set; }
    }

    public class Person
    {
        public int Id { get; set; }
        public string Name { get; set; }
        
        public IList<Post> Posts { get; } = new List<Post>();

        public Blog OwnedBlog { get; set; }
    }
-->
[!code-csharp[Model](../../../samples/core/CascadeDeletes/WithDatabaseCycleSamples.cs?name=Model)]

该模型具有 3 个关系，所有这些关系都是必需的，因此按约定配置为级联删除：

- 删除博客将级联删除所有相关文章
- 删除文章的作者将导致作者的文章被级联删除
- 删除博客所有者将导致该博客被级联删除

这一切都是合理的（不过在博客管理策略中有些苛刻！），但是尝试创建配置了这些级联的 SQL Server 数据库会导致以下异常：

> Microsoft.Data.SqlClient.SqlException (0x80131904):将 FOREIGN KEY 约束 "FK_Posts_Person_AuthorId" 引入表 "Posts" 可能会导致循环或多重级联路径。 请指定 ON DELETE NO ACTION 或 ON UPDATE NO ACTION，或修改其他 FOREIGN KEY 约束。

有两种方法可处理这种情况：

1. 将一个或多个关系更改为不级联删除。
2. 配置数据库，但不包含这些级联删除中的一个或多个，然后确保已加载所有依赖实体，以便 EF Core 可执行级联行为。

在我们的示例中采用第一种方法，我们可通过为博客与所有者之间的关系赋予可为 null 的外键属性来使其成为可选关系：

<!--
            public int? BlogId { get; set; }
-->
[!code-csharp[NullableBlogId](../../../samples/core/CascadeDeletes/OptionalDependentsSamples.cs?name=NullableBlogId)]

可选关系使得即使没有所有者，博客也可存在，这意味着默认情况下将不再配置级联删除。 这表示级联操作不再循环，并且可以在 SQL Server 上创建数据库而不会出现错误。

采取第二种方法，我们可以保持必需的博客所有者关系并对其配置来进行级联删除，但是使此配置仅适用于跟踪的实体，而不适用于数据库：

<!--
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder
                .Entity<Blog>()
                .HasOne(e => e.Owner)
                .WithOne(e => e.OwnedBlog)
                .OnDelete(DeleteBehavior.ClientCascade);
        }
-->
[!code-csharp[OnModelCreating](../../../samples/core/CascadeDeletes/WithDatabaseCycleSamples.cs?name=OnModelCreating)]

现在，如果我们同时加载某用户及其拥有的博客，然后删除该用户，会发生什么呢？

<!--
            using var context = new BlogsContext();

            var owner = context.People.Single(e => e.Name == "ajcvickers");
            var blog = context.Blogs.Single(e => e.Owner == owner);

            context.Remove(owner);
            
            context.SaveChanges();
-->
[!code-csharp[Database_cascade_limitations_1](../../../samples/core/CascadeDeletes/WithDatabaseCycleSamples.cs?name=Database_cascade_limitations_1)]

EF Core 将级联删除所有者，以便博客也被删除：

```sql
-- Executed DbCommand (8ms) [Parameters=[@p0='1'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [Blogs]
WHERE [Id] = @p0;
SELECT @@ROWCOUNT;

-- Executed DbCommand (2ms) [Parameters=[@p1='1'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [People]
WHERE [Id] = @p1;
SELECT @@ROWCOUNT;
```

但是，如果在删除所有者时未加载博客：

<!--
                using var context = new BlogsContext();

                var owner = context.People.Single(e => e.Name == "ajcvickers");

                context.Remove(owner);
            
                context.SaveChanges();
-->
[!code-csharp[Database_cascade_limitations_2](../../../samples/core/CascadeDeletes/WithDatabaseCycleSamples.cs?name=Database_cascade_limitations_2)]

则由于违反数据库中的外键约束，将引发异常：

> Microsoft.Data.SqlClient.SqlException:DELETE 语句与REFERENCE 约束 "FK_Blogs_People_OwnerId" 发生冲突。 数据库 "Scratch"、表 "dbo.Blogs"、列 "OwnerId" 中发生冲突。
语句已终止。

## <a name="cascading-nulls"></a>级联 NULL

可选关系将可为 null 的外键属性映射到可为 null 的数据库列。 这意味着当删除当前主体/父实体或断开与依赖实体/子实体的关系时，可将外键值设置为 NULL。

让我们再看一下[发生级联行为时](#when-cascading-behaviors-happen)的示例，但这次可选关系由可为 null 的 `Post.BlogId` 外键属性表示：

<!--
            public int? BlogId { get; set; }
-->
[!code-csharp[NullableBlogId](../../../samples/core/CascadeDeletes/OptionalDependentsSamples.cs?name=NullableBlogId)]

删除每篇文章的相关博客时，该文章的外键属性将设置为 NULL。 例如，此代码与之前的代码相同：

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).Include(e => e.Posts).First();

            context.Remove(blog);
            
            context.SaveChanges();
-->
[!code-csharp[Deleting_principal_parent_1b](../../../samples/core/CascadeDeletes/IntroOptionalSamples.cs?name=Deleting_principal_parent_1b)]

现将在调用 SaveChanges 时导致以下数据库更新：

```sql
-- Executed DbCommand (2ms) [Parameters=[@p1='1', @p0=NULL (DbType = Int32)], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
UPDATE [Posts] SET [BlogId] = @p0
WHERE [Id] = @p1;
SELECT @@ROWCOUNT;

-- Executed DbCommand (0ms) [Parameters=[@p1='2', @p0=NULL (DbType = Int32)], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
UPDATE [Posts] SET [BlogId] = @p0
WHERE [Id] = @p1;
SELECT @@ROWCOUNT;

-- Executed DbCommand (1ms) [Parameters=[@p2='1'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [Blogs]
WHERE [Id] = @p2;
SELECT @@ROWCOUNT;
```

同样，如果使用上述任一示例来断开关系：

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).Include(e => e.Posts).First();

            foreach (var post in blog.Posts)
            {
                post.Blog = null;
            }
            
            context.SaveChanges();
-->
[!code-csharp[Severing_a_relationship_1b](../../../samples/core/CascadeDeletes/IntroOptionalSamples.cs?name=Severing_a_relationship_1b)]

或：

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).Include(e => e.Posts).First();

            blog.Posts.Clear();
            
            context.SaveChanges();
-->
[!code-csharp[Severing_a_relationship_2b](../../../samples/core/CascadeDeletes/IntroOptionalSamples.cs?name=Severing_a_relationship_2b)]

则在调用 SaveChanges 时，将使用 NULL 外键值更新文章：

```sql
-- Executed DbCommand (2ms) [Parameters=[@p1='1', @p0=NULL (DbType = Int32)], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
UPDATE [Posts] SET [BlogId] = @p0
WHERE [Id] = @p1;
SELECT @@ROWCOUNT;

-- Executed DbCommand (0ms) [Parameters=[@p1='2', @p0=NULL (DbType = Int32)], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
UPDATE [Posts] SET [BlogId] = @p0
WHERE [Id] = @p1;
SELECT @@ROWCOUNT;
```

请参阅[更改外键和导航](xref:core/change-tracking/relationship-changes)，详细了解 EF Core 如何在外键和导航的值更改时管理外键和导航。

> [!NOTE]
> 自 2008 年首版以来，实体框架默认情况下都会修复这类关系。 在 EF Core 之前，它没有名称，且无法更改。 它现在称为 `ClientSetNull`，如下一部分所述。

当删除可选关系中的主体/父实体时，数据库也可配置为级联 NULL。 但是，与在数据库中使用级联删除相比，这种情况要少得多。 在使用 SQL Server 时，在数据库中同时使用级联删除和级联 NULL 几乎总是会导致关系循环。 若要详细了解如何配置级联 NULL，请参阅下一部分。

## <a name="configuring-cascading-behaviors"></a>配置级联行为

> [!TIP]
> 请务必阅读上述部分，然后再转到此处。 如果不了解上述资料，那么配置选项可能没有意义。

使用 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> 中的 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.ReferenceCollectionBuilder.OnDelete%2A> 方法按关系配置级联行为。 例如：

<!--
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder
                .Entity<Blog>()
                .HasOne(e => e.Owner)
                .WithOne(e => e.OwnedBlog)
                .OnDelete(DeleteBehavior.ClientCascade);
        }
-->
[!code-csharp[OnModelCreating](../../../samples/core/CascadeDeletes/WithDatabaseCycleSamples.cs?name=OnModelCreating)]

若要详细了解如何配置实体类型之间的关系，请参阅[关系](xref:core/modeling/relationships)。

`OnDelete` 从公认地令人混淆的 <xref:Microsoft.EntityFrameworkCore.DeleteBehavior> 枚举中接受一个值。 该枚举既定义了 EF Core 在跟踪实体上的行为，又定义了使用 EF 创建架构时数据库中级联删除的配置。

### <a name="impact-on-database-schema"></a>对数据库架构的影响

下表显示了由 EF Core 迁移或 <xref:Microsoft.EntityFrameworkCore.Infrastructure.DatabaseFacade.EnsureCreated%2A> 创建的外键约束上每个 `OnDelete` 值的结果。

| DeleteBehavior        | 对数据库架构的影响
|:----------------------|--------------------------
| Cascade               | ON DELETE CASCADE
| 限制              | ON DELETE NO ACTION
| NoAction              | <database default>
| SetNull               | ON DELETE SET NULL
| ClientSetNull         | ON DELETE NO ACTION
| ClientCascade         | ON DELETE NO ACTION
| ClientNoAction        | <database default>

> [!NOTE]
> 该表令人困惑，我们计划在将来的版本中重新进行介绍。 请参阅 [GitHub 问题 #21252](https://github.com/dotnet/efcore/issues/21252).

关系数据库中 `ON DELETE NO ACTION` 和 `ON DELETE RESTRICT` 的行为通常相同或非常相似。 尽管 `NO ACTION` 可能意味着什么，但这两个选项都会导致强制执行引用约束。 区别是当有一个时，是影响数据库何时检查约束。  请查看数据库文档，了解数据库系统上 `ON DELETE NO ACTION` 和 `ON DELETE RESTRICT` 之间的具体区别。

导致数据库级联行为的唯一值是 `Cascade` 和 `SetNull`。 所有其他值会将数据库配置为不级联任何更改。

### <a name="impact-on-savechanges-behavior"></a>对 SaveChanges 行为的影响

以下各部分中的表格介绍了删除主体/父实体或断开与主体/子实体的关系时，依赖实体/子实体所发生的情况。 每张表都涵盖下述内容之一：

- 可选（可为 null 的外键）和必需（不可为 null 的外键）关系
- 依赖项/子项何时由 DbContext 加载和跟踪，以及它们何时仅存在于数据库中

#### <a name="required-relationship-with-dependentschildren-loaded"></a>与已加载的依赖项/子项的必需关系

| DeleteBehavior    | 删除主体/父实体时             | 断开与主体/父实体的关系时
|:------------------|------------------------------------------|----------------------------------------
| Cascade           | EF Core 删除的依赖项            | EF Core 删除的依赖项
| 限制          | `InvalidOperationException`              | `InvalidOperationException`
| NoAction          | `InvalidOperationException`              | `InvalidOperationException`
| SetNull           | `SqlException`（创建数据库时）      | `SqlException`（创建数据库时）
| ClientSetNull     | `InvalidOperationException`              | `InvalidOperationException`
| ClientCascade     | EF Core 删除的依赖项            | EF Core 删除的依赖项
| ClientNoAction    | `DbUpdateException`                      | `InvalidOperationException`

说明：

- 这种必需关系的默认值为 `Cascade`。
- 调用 SaveChanges 时，对必需关系使用除级联删除以外的其他方法将导致异常。
  - 通常，这是来自 EF Core 的 `InvalidOperationException`，因为在已加载的子项/依赖项中检测到无效状态。
  - `ClientNoAction` 会强制 EF Core 在将依赖项发送到数据库之前不检查修复它们，因此在这种情况下，数据库将引发异常，然后由 SaveChanges 将其包装在 `DbUpdateException` 中。
  - 创建数据库时会拒绝 `SetNull`，因为外键列不可为 null。
- 由于已加载依赖项/子项，因此它们始终会被 EF Core 删除，并且永远不会留下来等到数据库被删除。

#### <a name="required-relationship-with-dependentschildren-not-loaded"></a>与未加载的依赖项/子项的必需关系

| DeleteBehavior    | 删除主体/父实体时             | 断开与主体/父实体的关系时
|:------------------|------------------------------------------|----------------------------------------
| Cascade           | 数据库删除的依赖项           | 不可用
| 限制          | `DbUpdateException`                      | 不可用
| NoAction          | `DbUpdateException`                      | 不可用
| SetNull           | `SqlException`（创建数据库时）      | 不可用
| ClientSetNull     | `DbUpdateException`                      | 不可用
| ClientCascade     | `DbUpdateException`                      | 不可用
| ClientNoAction    | `DbUpdateException`                      | 不可用

说明：

- 此处没法断开关系，因为未加载依赖项/子项。
- 这种必需关系的默认值为 `Cascade`。
- 调用 SaveChanges 时，对必需关系使用除级联删除以外的其他方法将导致异常。
  - 通常，这是 `DbUpdateException`，理由是未加载依赖项/子项，因此数据库只能检测到无效状态。 然后，SaveChanges 会将数据库异常包装在 `DbUpdateException` 中。
  - 创建数据库时会拒绝 `SetNull`，因为外键列不可为 null。

#### <a name="optional-relationship-with-dependentschildren-loaded"></a>与已加载的依赖项/子项的可选关系

| DeleteBehavior    | 删除主体/父实体时             | 断开与主体/父实体的关系时
|:------------------|------------------------------------------|----------------------------------------
| Cascade           | EF Core 删除的依赖项            | EF Core 删除的依赖项
| 限制          | EF Core 将依赖外键设置为 NULL     | EF Core 将依赖外键设置为 NULL
| NoAction          | EF Core 将依赖外键设置为 NULL     | EF Core 将依赖外键设置为 NULL
| SetNull           | EF Core 将依赖外键设置为 NULL     | EF Core 将依赖外键设置为 NULL
| ClientSetNull     | EF Core 将依赖外键设置为 NULL     | EF Core 将依赖外键设置为 NULL
| ClientCascade     | EF Core 删除的依赖项            | EF Core 删除的依赖项
| ClientNoAction    | `DbUpdateException`                      | EF Core 将依赖外键设置为 NULL

说明：

- 这种可选关系的默认值为 `ClientSetNull`。
- 永远不会删除依赖项/子项，除非配置了 `Cascade` 或 `ClientCascade`。
- 所有其他值都会导致 EF Core 将依赖外键设置为 NULL...
  - ... `ClientNoAction` 除外，它指示 EF Core 在删除主体/父实体时不处理依赖项/子项的外键。 因此，数据库会引发异常，由 SaveChanges 将其包装为 `DbUpdateException`。

#### <a name="optional-relationship-with-dependentschildren-not-loaded"></a>与未加载的依赖项/子项的可选关系

| DeleteBehavior    | 删除主体/父实体时             | 断开与主体/父实体的关系时
|:------------------|------------------------------------------|----------------------------------------
| Cascade           | 数据库删除的依赖项           | 不可用
| 限制          | `DbUpdateException`                      | 不可用
| NoAction          | `DbUpdateException`                      | 不可用
| SetNull           | 数据库将依赖外键设置为 NULL    | 不可用
| ClientSetNull     | `DbUpdateException`                      | 不可用
| ClientCascade     | `DbUpdateException`                      | 不可用
| ClientNoAction    | `DbUpdateException`                      | 不可用

说明：

- 此处没法断开关系，因为未加载依赖项/子项。
- 这种可选关系的默认值为 `ClientSetNull`。
- 除非已将数据库配置为级联删除或级联 NULL，否则必须加载依赖项/子项以避免数据库异常。
