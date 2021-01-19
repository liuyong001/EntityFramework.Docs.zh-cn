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
# <a name="cascade-delete"></a><span data-ttu-id="62b0d-103">级联删除</span><span class="sxs-lookup"><span data-stu-id="62b0d-103">Cascade Delete</span></span>

<span data-ttu-id="62b0d-104">Entity Framework Core (EF Core) 表示使用外键的关系。</span><span class="sxs-lookup"><span data-stu-id="62b0d-104">Entity Framework Core (EF Core) represents relationships using foreign keys.</span></span> <span data-ttu-id="62b0d-105">具有外键的实体是关系中的子实体或依赖实体。</span><span class="sxs-lookup"><span data-stu-id="62b0d-105">An entity with a foreign key is the child or dependent entity in the relationship.</span></span> <span data-ttu-id="62b0d-106">此实体的外键值必须与相关主体/父实体的主键值（或替换键值）匹配。</span><span class="sxs-lookup"><span data-stu-id="62b0d-106">This entity's foreign key value must match the primary key value (or an alternate key value) of the related principal/parent entity.</span></span>

<span data-ttu-id="62b0d-107">如果删除主体/父实体，则依赖项/子项的外键值将不再匹配任何主体/父实体的主键或替换键。</span><span class="sxs-lookup"><span data-stu-id="62b0d-107">If the principal/parent entity is deleted, then the foreign key values of the dependents/children will no longer match the primary or alternate key of _any_ principal/parent.</span></span> <span data-ttu-id="62b0d-108">这是无效状态，将导致在大多数数据库中出现引用约束冲突。</span><span class="sxs-lookup"><span data-stu-id="62b0d-108">This is an invalid state, and will cause a referential constraint violation in most databases.</span></span>

<span data-ttu-id="62b0d-109">可通过两种方法来避免此引用约束冲突：</span><span class="sxs-lookup"><span data-stu-id="62b0d-109">There are two options to avoid this referential constraint violation:</span></span>

1. <span data-ttu-id="62b0d-110">将外键值设置为 null</span><span class="sxs-lookup"><span data-stu-id="62b0d-110">Set the FK values to null</span></span>
2. <span data-ttu-id="62b0d-111">同时删除依赖实体/子实体</span><span class="sxs-lookup"><span data-stu-id="62b0d-111">Also delete the dependent/child entities</span></span>

<span data-ttu-id="62b0d-112">第一个选项仅适用于其中外键属性（及其映射到的数据库列）必须可为 null 的可选关系。</span><span class="sxs-lookup"><span data-stu-id="62b0d-112">The first option in only valid for optional relationships where the foreign key property (and the database column to which it is mapped) must be nullable.</span></span>

<span data-ttu-id="62b0d-113">第二个选项适用于任何类型的关系，它被称作“级联删除”。</span><span class="sxs-lookup"><span data-stu-id="62b0d-113">The second option is valid for any kind of relationship and is known as "cascade delete".</span></span>

> [!TIP]
> <span data-ttu-id="62b0d-114">本文档从更新数据库的角度介绍级联删除（和删除孤立项）。</span><span class="sxs-lookup"><span data-stu-id="62b0d-114">This document describes cascade deletes (and deleting orphans) from the perspective of updating the database.</span></span> <span data-ttu-id="62b0d-115">本文大量使用在[在 EF Core 中更改跟踪](xref:core/change-tracking/index)和[更改外键和导航](xref:core/change-tracking/relationship-changes)文章中介绍的概念。</span><span class="sxs-lookup"><span data-stu-id="62b0d-115">It makes heavy use of concepts introduced in [Change Tracking in EF Core](xref:core/change-tracking/index) and [Changing Foreign Keys and Navigations](xref:core/change-tracking/relationship-changes).</span></span> <span data-ttu-id="62b0d-116">请确保在此处处理材料之前充分了解这些概念。</span><span class="sxs-lookup"><span data-stu-id="62b0d-116">Make sure to fully understand these concepts before tackling the material here.</span></span>

> [!TIP]  
> <span data-ttu-id="62b0d-117">通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/CascadeDeletes)，你可运行并调试到本文档中的所有代码。</span><span class="sxs-lookup"><span data-stu-id="62b0d-117">You can run and debug into all the code in this document by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/CascadeDeletes).</span></span>

## <a name="when-cascading-behaviors-happen"></a><span data-ttu-id="62b0d-118">发生级联行为时</span><span class="sxs-lookup"><span data-stu-id="62b0d-118">When cascading behaviors happen</span></span>

<span data-ttu-id="62b0d-119">当依赖实体/子实体无法再与其当前主体/父实体关联时，需要执行级联删除。</span><span class="sxs-lookup"><span data-stu-id="62b0d-119">Cascading deletes are needed when a dependent/child entity can no longer be associated with its current principal/parent.</span></span> <span data-ttu-id="62b0d-120">发生这种情况的原因可能是主体/父实体已被删除，或者当主体/父实体仍存在，但依赖实体/子实体不再与其关联时。</span><span class="sxs-lookup"><span data-stu-id="62b0d-120">This can happen because the principal/parent is deleted, or it can happen when the principal/parent still exists but the dependent/child is no longer associated with it.</span></span>

### <a name="deleting-a-principalparent"></a><span data-ttu-id="62b0d-121">删除主体/父实体</span><span class="sxs-lookup"><span data-stu-id="62b0d-121">Deleting a principal/parent</span></span>

<span data-ttu-id="62b0d-122">请考虑此简单模型，其中 `Blog` 是与 `Post`（依赖实体/子实体）的关系中的主体/父实体。</span><span class="sxs-lookup"><span data-stu-id="62b0d-122">Consider this simple model where `Blog` is the principal/parent in a relationship with `Post`, which is the dependent/child.</span></span> <span data-ttu-id="62b0d-123">`Post.BlogId` 是外键属性，其值必须与该博客所属文章中的 `Post.Id` 主键匹配。</span><span class="sxs-lookup"><span data-stu-id="62b0d-123">`Post.BlogId` is a foreign key property, the value of which must match the `Post.Id` primary key of the post to which the blog belongs.</span></span>

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

<span data-ttu-id="62b0d-124">按照约定，由于 `Post.BlogId` 外键属性是不可为 null 的，因此该关系被配置为必需的。</span><span class="sxs-lookup"><span data-stu-id="62b0d-124">By convention, this relationship is configured as a required, since the `Post.BlogId` foreign key property is non-nullable.</span></span> <span data-ttu-id="62b0d-125">默认情况下，所需的关系配置为使用级联删除。</span><span class="sxs-lookup"><span data-stu-id="62b0d-125">Required relationships are configured to use cascade deletes by default.</span></span> <span data-ttu-id="62b0d-126">要详细了解建模关系，请参阅[关系](xref:core/modeling/relationships)。</span><span class="sxs-lookup"><span data-stu-id="62b0d-126">See [Relationships](xref:core/modeling/relationships) for more information on modeling relationships.</span></span>

<span data-ttu-id="62b0d-127">删除博客时，所有文章都将被级联删除。</span><span class="sxs-lookup"><span data-stu-id="62b0d-127">When deleting a blog, all posts are cascade deleted.</span></span> <span data-ttu-id="62b0d-128">例如：</span><span class="sxs-lookup"><span data-stu-id="62b0d-128">For example:</span></span>

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).Include(e => e.Posts).First();

            context.Remove(blog);
            
            context.SaveChanges();
-->
[!code-csharp[Deleting_principal_parent_1](../../../samples/core/CascadeDeletes/IntroRequiredSamples.cs?name=Deleting_principal_parent_1)]

<span data-ttu-id="62b0d-129">SaveChanges 以 SQL Server 为例，生成以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="62b0d-129">SaveChanges generates the following SQL, using SQL Server as an example:</span></span>

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

### <a name="severing-a-relationship"></a><span data-ttu-id="62b0d-130">断开关系</span><span class="sxs-lookup"><span data-stu-id="62b0d-130">Severing a relationship</span></span>

<span data-ttu-id="62b0d-131">我们不会删除博客，而是断开每篇文章与其博客之间的关系。</span><span class="sxs-lookup"><span data-stu-id="62b0d-131">Rather than deleting the blog, we could instead sever the relationship between each post and its blog.</span></span> <span data-ttu-id="62b0d-132">为此，可将每篇文章的引用导航 `Post.Blog` 设置为 null：</span><span class="sxs-lookup"><span data-stu-id="62b0d-132">This can be done by setting the reference navigation `Post.Blog` to null for each post:</span></span>

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

<span data-ttu-id="62b0d-133">还可通过从 `Blog.Posts` 集合导航中删除每篇文章内容来断开关系：</span><span class="sxs-lookup"><span data-stu-id="62b0d-133">The relationship can also be severed by removing each post from the `Blog.Posts` collection navigation:</span></span>

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).Include(e => e.Posts).First();

            blog.Posts.Clear();
            
            context.SaveChanges();
-->
[!code-csharp[Severing_a_relationship_2](../../../samples/core/CascadeDeletes/IntroRequiredSamples.cs?name=Severing_a_relationship_2)]

<span data-ttu-id="62b0d-134">无论哪种情况，结果都一样：没有删除博客，但是删除了不再与任何博客关联的文章：</span><span class="sxs-lookup"><span data-stu-id="62b0d-134">In either case the result is the same: the blog is not deleted, but the posts that are no longer associated with any blog are deleted:</span></span>

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

<span data-ttu-id="62b0d-135">删除不再与任何主体/依赖实体关联的实体这一行为被称作“删除孤立项”。</span><span class="sxs-lookup"><span data-stu-id="62b0d-135">Deleting entities that are no longer associated with any principal/dependent is known as "deleting orphans".</span></span>

> [!TIP]
> <span data-ttu-id="62b0d-136">级联删除和删除孤立项是密切相关的。</span><span class="sxs-lookup"><span data-stu-id="62b0d-136">Cascade delete and deleting orphans are closely related.</span></span> <span data-ttu-id="62b0d-137">当断开与所需的主体/父实体之间的关系时，两者都将导致删除依赖实体/子实体。</span><span class="sxs-lookup"><span data-stu-id="62b0d-137">Both result in deleting dependent/child entities when the relationship to their required principal/parent is severed.</span></span> <span data-ttu-id="62b0d-138">对于级联删除，由于主体/父实体本身已删除，因此发生了这种断开。</span><span class="sxs-lookup"><span data-stu-id="62b0d-138">For cascade delete, this severing happens because the principal/parent is itself deleted.</span></span> <span data-ttu-id="62b0d-139">对于孤立项，主体/父实体仍然存在，但不再与依赖实体/子实体相关。</span><span class="sxs-lookup"><span data-stu-id="62b0d-139">For orphans, the principal/parent entity still exists, but is no longer related to the dependent/child entities.</span></span>  

## <a name="where-cascading-behaviors-happen"></a><span data-ttu-id="62b0d-140">发生级联行为的位置</span><span class="sxs-lookup"><span data-stu-id="62b0d-140">Where cascading behaviors happen</span></span>

<span data-ttu-id="62b0d-141">可将级联行为应用于：</span><span class="sxs-lookup"><span data-stu-id="62b0d-141">Cascading behaviors can be applied to:</span></span>

- <span data-ttu-id="62b0d-142">当前 <xref:Microsoft.EntityFrameworkCore.DbContext> 跟踪的实体</span><span class="sxs-lookup"><span data-stu-id="62b0d-142">Entities tracked by the current <xref:Microsoft.EntityFrameworkCore.DbContext></span></span>
- <span data-ttu-id="62b0d-143">数据库中尚未加载到上下文中的实体</span><span class="sxs-lookup"><span data-stu-id="62b0d-143">Entities in the database that have not been loaded into the context</span></span>

### <a name="cascade-delete-of-tracked-entities"></a><span data-ttu-id="62b0d-144">级联删除被跟踪实体</span><span class="sxs-lookup"><span data-stu-id="62b0d-144">Cascade delete of tracked entities</span></span>

<span data-ttu-id="62b0d-145">EF Core 始终将配置的级联行为应用于跟踪的实体。</span><span class="sxs-lookup"><span data-stu-id="62b0d-145">EF Core always applies configured cascading behaviors to tracked entities.</span></span> <span data-ttu-id="62b0d-146">这意味着如上面的示例所示，如果应用程序将所有相关的依赖实体/子实体加载到 DbContext 中，则无论如何配置数据库，都将正确应用级联行为。</span><span class="sxs-lookup"><span data-stu-id="62b0d-146">This means that if the application loads all relevant dependent/child entities into the DbContext, as is shown in the examples above, then cascading behaviors will be correctly applied regardless of how the database is configured.</span></span>

> [!TIP]
> <span data-ttu-id="62b0d-147">可使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeDeleteTiming?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DeleteOrphansTiming?displayProperty=nameWithType> 控制在被跟踪实体上发生级联行为的确切时间。</span><span class="sxs-lookup"><span data-stu-id="62b0d-147">The exact timing of when cascading behaviors happen to tracked entities can be controlled using <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.CascadeDeleteTiming?displayProperty=nameWithType> and <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DeleteOrphansTiming?displayProperty=nameWithType>.</span></span> <span data-ttu-id="62b0d-148">有关详细信息，请参阅[更改外键和导航](xref:core/change-tracking/relationship-changes)。</span><span class="sxs-lookup"><span data-stu-id="62b0d-148">See [Changing Foreign Keys and Navigations](xref:core/change-tracking/relationship-changes) for more information.</span></span>

### <a name="cascade-delete-in-the-database"></a><span data-ttu-id="62b0d-149">数据库中的级联删除</span><span class="sxs-lookup"><span data-stu-id="62b0d-149">Cascade delete in the database</span></span>

<span data-ttu-id="62b0d-150">许多数据库系统还提供在数据库中删除实体时触发的级联行为。</span><span class="sxs-lookup"><span data-stu-id="62b0d-150">Many database systems also offer cascading behaviors that are triggered when an entity is deleted in the database.</span></span> <span data-ttu-id="62b0d-151">使用 <xref:Microsoft.EntityFrameworkCore.Infrastructure.DatabaseFacade.EnsureCreated%2A> 或 EF Core 迁移创建数据库时，EF Core 会根据 EF Core 模型中的级联删除行为来配置这些行为。</span><span class="sxs-lookup"><span data-stu-id="62b0d-151">EF Core configures these behaviors based on the cascade delete behavior in the EF Core model when a database is created using <xref:Microsoft.EntityFrameworkCore.Infrastructure.DatabaseFacade.EnsureCreated%2A> or EF Core migrations.</span></span> <span data-ttu-id="62b0d-152">例如，通过上述模型，使用 SQL Server 时将为文章创建下表：</span><span class="sxs-lookup"><span data-stu-id="62b0d-152">For example, using the model above, the following table is created for posts when using SQL Server:</span></span>

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

<span data-ttu-id="62b0d-153">请注意，定义博客和文章之间关系的外键约束是用 `ON DELETE CASCADE` 配置的。</span><span class="sxs-lookup"><span data-stu-id="62b0d-153">Notice that the foreign key constraint defining the relationship between blogs and posts is configured with `ON DELETE CASCADE`.</span></span>

<span data-ttu-id="62b0d-154">如果我们知道数据库是这样配置的，那么我们可以删除博客，而无需先加载文章，数据库将负责删除与此博客相关的所有文章。</span><span class="sxs-lookup"><span data-stu-id="62b0d-154">If we know that the database is configured like this, then we can delete a blog _without first loading posts_ and the database will take care of deleting all the posts that were related to that blog.</span></span> <span data-ttu-id="62b0d-155">例如：</span><span class="sxs-lookup"><span data-stu-id="62b0d-155">For example:</span></span>

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).First();

            context.Remove(blog);
            
            context.SaveChanges();
-->
[!code-csharp[Where_cascading_behaviors_happen_1](../../../samples/core/CascadeDeletes/IntroRequiredSamples.cs?name=Where_cascading_behaviors_happen_1)]

<span data-ttu-id="62b0d-156">请注意，文章没有 `Include`，因此它们不会被加载。</span><span class="sxs-lookup"><span data-stu-id="62b0d-156">Notice that there is no `Include` for posts, so they are not loaded.</span></span> <span data-ttu-id="62b0d-157">在这种情况下，SaveChanges 将仅删除博客，因为这是唯一正在跟踪的实体：</span><span class="sxs-lookup"><span data-stu-id="62b0d-157">SaveChanges in this case will delete just the blog, since that's the only entity being tracked:</span></span>

```sql
-- Executed DbCommand (6ms) [Parameters=[@p0='1'], CommandType='Text', CommandTimeout='30']
SET NOCOUNT ON;
DELETE FROM [Blogs]
WHERE [Id] = @p0;
SELECT @@ROWCOUNT;
```

<span data-ttu-id="62b0d-158">如果未针对级联删除配置数据库中的外键约束，则将导致异常。</span><span class="sxs-lookup"><span data-stu-id="62b0d-158">This would result in an exception if the foreign key constraint in the database is not configured for cascade deletes.</span></span> <span data-ttu-id="62b0d-159">但在这种情况下，数据库删除了文章，因为它在创建时是用 `ON DELETE CASCADE` 配置的。</span><span class="sxs-lookup"><span data-stu-id="62b0d-159">However, in this case the posts are deleted by the database because it has been configured with `ON DELETE CASCADE` when it was created.</span></span>

> [!NOTE]
> <span data-ttu-id="62b0d-160">数据库通常没有任何自动删除孤立项的方法。</span><span class="sxs-lookup"><span data-stu-id="62b0d-160">Databases don't typically have any way to automatically delete orphans.</span></span> <span data-ttu-id="62b0d-161">这是因为虽然 EF Core 使用导航以及外键来表示关系，但是数据库仅具有外键而没有导航。</span><span class="sxs-lookup"><span data-stu-id="62b0d-161">This is because while EF Core represents relationships using navigations as well of foreign keys, databases have only foreign keys and no navigations.</span></span> <span data-ttu-id="62b0d-162">这意味着通常无法在不将双方都加载到 DbContext 的情况下断开关系。</span><span class="sxs-lookup"><span data-stu-id="62b0d-162">This means that it is usually not possible to sever a relationship without loading both sides into the DbContext.</span></span>

> [!NOTE]
> <span data-ttu-id="62b0d-163">EF Core 内存中数据库当前不支持数据库中的级联删除。</span><span class="sxs-lookup"><span data-stu-id="62b0d-163">The EF Core in-memory database does not currently support cascade deletes in the database.</span></span>

> [!WARNING]
> <span data-ttu-id="62b0d-164">软删除实体时，请勿在数据库中配置级联删除。</span><span class="sxs-lookup"><span data-stu-id="62b0d-164">Do not configure cascade delete in the database when soft-deleting entities.</span></span> <span data-ttu-id="62b0d-165">这可能会导致实体被意外删除，而不是软删除。</span><span class="sxs-lookup"><span data-stu-id="62b0d-165">This may cause entities to be accidentally really deleted instead of soft-deleted.</span></span>

### <a name="database-cascade-limitations"></a><span data-ttu-id="62b0d-166">数据库级联限制</span><span class="sxs-lookup"><span data-stu-id="62b0d-166">Database cascade limitations</span></span>

<span data-ttu-id="62b0d-167">一些数据库（最突出的是 SQL Server）对形成周期的级联行为有限制。</span><span class="sxs-lookup"><span data-stu-id="62b0d-167">Some databases, most notably SQL Server, have limitations on the cascade behaviors that form cycles.</span></span> <span data-ttu-id="62b0d-168">例如，请考虑以下模型：</span><span class="sxs-lookup"><span data-stu-id="62b0d-168">For example, consider the following model:</span></span>

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

<span data-ttu-id="62b0d-169">该模型具有 3 个关系，所有这些关系都是必需的，因此按约定配置为级联删除：</span><span class="sxs-lookup"><span data-stu-id="62b0d-169">This model has three relationships, all required and therefore configured to cascade delete by convention:</span></span>

- <span data-ttu-id="62b0d-170">删除博客将级联删除所有相关文章</span><span class="sxs-lookup"><span data-stu-id="62b0d-170">Deleting a blog will cascade delete all the related posts</span></span>
- <span data-ttu-id="62b0d-171">删除文章的作者将导致作者的文章被级联删除</span><span class="sxs-lookup"><span data-stu-id="62b0d-171">Deleting the author of posts will cause the authored posts to be cascade deleted</span></span>
- <span data-ttu-id="62b0d-172">删除博客所有者将导致该博客被级联删除</span><span class="sxs-lookup"><span data-stu-id="62b0d-172">Deleting the owner of a blog will cause the blog to be cascade deleted</span></span>

<span data-ttu-id="62b0d-173">这一切都是合理的（不过在博客管理策略中有些苛刻！），但是尝试创建配置了这些级联的 SQL Server 数据库会导致以下异常：</span><span class="sxs-lookup"><span data-stu-id="62b0d-173">This is all reasonable (if a bit draconian in blog management policies!) but attempting to create a SQL Server database with these cascades configured results in the following exception:</span></span>

> <span data-ttu-id="62b0d-174">Microsoft.Data.SqlClient.SqlException (0x80131904):将 FOREIGN KEY 约束 "FK_Posts_Person_AuthorId" 引入表 "Posts" 可能会导致循环或多重级联路径。</span><span class="sxs-lookup"><span data-stu-id="62b0d-174">Microsoft.Data.SqlClient.SqlException (0x80131904): Introducing FOREIGN KEY constraint 'FK_Posts_Person_AuthorId' on table 'Posts' may cause cycles or multiple cascade paths.</span></span> <span data-ttu-id="62b0d-175">请指定 ON DELETE NO ACTION 或 ON UPDATE NO ACTION，或修改其他 FOREIGN KEY 约束。</span><span class="sxs-lookup"><span data-stu-id="62b0d-175">Specify ON DELETE NO ACTION or ON UPDATE NO ACTION, or modify other FOREIGN KEY constraints.</span></span>

<span data-ttu-id="62b0d-176">有两种方法可处理这种情况：</span><span class="sxs-lookup"><span data-stu-id="62b0d-176">There are two ways to handle this situation:</span></span>

1. <span data-ttu-id="62b0d-177">将一个或多个关系更改为不级联删除。</span><span class="sxs-lookup"><span data-stu-id="62b0d-177">Change one or more of the relationships to not cascade delete.</span></span>
2. <span data-ttu-id="62b0d-178">配置数据库，但不包含这些级联删除中的一个或多个，然后确保已加载所有依赖实体，以便 EF Core 可执行级联行为。</span><span class="sxs-lookup"><span data-stu-id="62b0d-178">Configure the database without one or more of these cascade deletes, then ensure all dependent entities are loaded so that EF Core can perform the cascading behavior.</span></span>

<span data-ttu-id="62b0d-179">在我们的示例中采用第一种方法，我们可通过为博客与所有者之间的关系赋予可为 null 的外键属性来使其成为可选关系：</span><span class="sxs-lookup"><span data-stu-id="62b0d-179">Taking the first approach with our example, we could make the blog-owner relationship optional by giving it a nullable foreign key property:</span></span>

<!--
            public int? BlogId { get; set; }
-->
[!code-csharp[NullableBlogId](../../../samples/core/CascadeDeletes/OptionalDependentsSamples.cs?name=NullableBlogId)]

<span data-ttu-id="62b0d-180">可选关系使得即使没有所有者，博客也可存在，这意味着默认情况下将不再配置级联删除。</span><span class="sxs-lookup"><span data-stu-id="62b0d-180">An optional relationship allows the blog to exist without an owner, which means cascade delete will no longer be configured by default.</span></span> <span data-ttu-id="62b0d-181">这表示级联操作不再循环，并且可以在 SQL Server 上创建数据库而不会出现错误。</span><span class="sxs-lookup"><span data-stu-id="62b0d-181">This means there is no longer a cycle in cascading actions, and the database can be created without error on SQL Server.</span></span>

<span data-ttu-id="62b0d-182">采取第二种方法，我们可以保持必需的博客所有者关系并对其配置来进行级联删除，但是使此配置仅适用于跟踪的实体，而不适用于数据库：</span><span class="sxs-lookup"><span data-stu-id="62b0d-182">Taking the second approach instead, we can keep the blog-owner relationship required and configured for cascade delete, but make this configuration only apply to tracked entities, not the database:</span></span>

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

<span data-ttu-id="62b0d-183">现在，如果我们同时加载某用户及其拥有的博客，然后删除该用户，会发生什么呢？</span><span class="sxs-lookup"><span data-stu-id="62b0d-183">Now what happens if we load both a person and the blog they own, then delete the person?</span></span>

<!--
            using var context = new BlogsContext();

            var owner = context.People.Single(e => e.Name == "ajcvickers");
            var blog = context.Blogs.Single(e => e.Owner == owner);

            context.Remove(owner);
            
            context.SaveChanges();
-->
[!code-csharp[Database_cascade_limitations_1](../../../samples/core/CascadeDeletes/WithDatabaseCycleSamples.cs?name=Database_cascade_limitations_1)]

<span data-ttu-id="62b0d-184">EF Core 将级联删除所有者，以便博客也被删除：</span><span class="sxs-lookup"><span data-stu-id="62b0d-184">EF Core will cascade the delete of the owner so that the blog is also deleted:</span></span>

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

<span data-ttu-id="62b0d-185">但是，如果在删除所有者时未加载博客：</span><span class="sxs-lookup"><span data-stu-id="62b0d-185">However, if the blog is not loaded when the owner is deleted:</span></span>

<!--
                using var context = new BlogsContext();

                var owner = context.People.Single(e => e.Name == "ajcvickers");

                context.Remove(owner);
            
                context.SaveChanges();
-->
[!code-csharp[Database_cascade_limitations_2](../../../samples/core/CascadeDeletes/WithDatabaseCycleSamples.cs?name=Database_cascade_limitations_2)]

<span data-ttu-id="62b0d-186">则由于违反数据库中的外键约束，将引发异常：</span><span class="sxs-lookup"><span data-stu-id="62b0d-186">Then an exception will be thrown due to violation of the foreign key constraint in the database:</span></span>

> <span data-ttu-id="62b0d-187">Microsoft.Data.SqlClient.SqlException:DELETE 语句与REFERENCE 约束 "FK_Blogs_People_OwnerId" 发生冲突。</span><span class="sxs-lookup"><span data-stu-id="62b0d-187">Microsoft.Data.SqlClient.SqlException: The DELETE statement conflicted with the REFERENCE constraint "FK_Blogs_People_OwnerId".</span></span> <span data-ttu-id="62b0d-188">数据库 "Scratch"、表 "dbo.Blogs"、列 "OwnerId" 中发生冲突。</span><span class="sxs-lookup"><span data-stu-id="62b0d-188">The conflict occurred in database "Scratch", table "dbo.Blogs", column 'OwnerId'.</span></span>
<span data-ttu-id="62b0d-189">语句已终止。</span><span class="sxs-lookup"><span data-stu-id="62b0d-189">The statement has been terminated.</span></span>

## <a name="cascading-nulls"></a><span data-ttu-id="62b0d-190">级联 NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-190">Cascading nulls</span></span>

<span data-ttu-id="62b0d-191">可选关系将可为 null 的外键属性映射到可为 null 的数据库列。</span><span class="sxs-lookup"><span data-stu-id="62b0d-191">Optional relationships have nullable foreign key properties mapped to nullable database columns.</span></span> <span data-ttu-id="62b0d-192">这意味着当删除当前主体/父实体或断开与依赖实体/子实体的关系时，可将外键值设置为 NULL。</span><span class="sxs-lookup"><span data-stu-id="62b0d-192">This means that the foreign key value can be set to null when the current principal/parent is deleted or is severed from the dependent/child.</span></span>

<span data-ttu-id="62b0d-193">让我们再看一下[发生级联行为时](#when-cascading-behaviors-happen)的示例，但这次可选关系由可为 null 的 `Post.BlogId` 外键属性表示：</span><span class="sxs-lookup"><span data-stu-id="62b0d-193">Let's look again at the examples from [When cascading behaviors happen](#when-cascading-behaviors-happen), but this time with an optional relationship represented by a nullable `Post.BlogId` foreign key property:</span></span>

<!--
            public int? BlogId { get; set; }
-->
[!code-csharp[NullableBlogId](../../../samples/core/CascadeDeletes/OptionalDependentsSamples.cs?name=NullableBlogId)]

<span data-ttu-id="62b0d-194">删除每篇文章的相关博客时，该文章的外键属性将设置为 NULL。</span><span class="sxs-lookup"><span data-stu-id="62b0d-194">This foreign key property will be set to null for each post when its related blog is deleted.</span></span> <span data-ttu-id="62b0d-195">例如，此代码与之前的代码相同：</span><span class="sxs-lookup"><span data-stu-id="62b0d-195">For example, this code, which is the same as before:</span></span>

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).Include(e => e.Posts).First();

            context.Remove(blog);
            
            context.SaveChanges();
-->
[!code-csharp[Deleting_principal_parent_1b](../../../samples/core/CascadeDeletes/IntroOptionalSamples.cs?name=Deleting_principal_parent_1b)]

<span data-ttu-id="62b0d-196">现将在调用 SaveChanges 时导致以下数据库更新：</span><span class="sxs-lookup"><span data-stu-id="62b0d-196">Will now result in the following database updates when SaveChanges is called:</span></span>

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

<span data-ttu-id="62b0d-197">同样，如果使用上述任一示例来断开关系：</span><span class="sxs-lookup"><span data-stu-id="62b0d-197">Likewise, if the relationship is severed using either of the examples from above:</span></span>

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

<span data-ttu-id="62b0d-198">或：</span><span class="sxs-lookup"><span data-stu-id="62b0d-198">Or:</span></span>

<!--
            using var context = new BlogsContext();

            var blog = context.Blogs.OrderBy(e => e.Name).Include(e => e.Posts).First();

            blog.Posts.Clear();
            
            context.SaveChanges();
-->
[!code-csharp[Severing_a_relationship_2b](../../../samples/core/CascadeDeletes/IntroOptionalSamples.cs?name=Severing_a_relationship_2b)]

<span data-ttu-id="62b0d-199">则在调用 SaveChanges 时，将使用 NULL 外键值更新文章：</span><span class="sxs-lookup"><span data-stu-id="62b0d-199">Then the posts are updated with null foreign key values when SaveChanges is called:</span></span>

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

<span data-ttu-id="62b0d-200">请参阅[更改外键和导航](xref:core/change-tracking/relationship-changes)，详细了解 EF Core 如何在外键和导航的值更改时管理外键和导航。</span><span class="sxs-lookup"><span data-stu-id="62b0d-200">See [Changing Foreign Keys and Navigations](xref:core/change-tracking/relationship-changes) for more information on how EF Core manages foreign keys and navigations as their values are changed.</span></span>

> [!NOTE]
> <span data-ttu-id="62b0d-201">自 2008 年首版以来，实体框架默认情况下都会修复这类关系。</span><span class="sxs-lookup"><span data-stu-id="62b0d-201">The fixup of relationships like this has been the default behavior of Entity Framework since the first version in 2008.</span></span> <span data-ttu-id="62b0d-202">在 EF Core 之前，它没有名称，且无法更改。</span><span class="sxs-lookup"><span data-stu-id="62b0d-202">Prior to EF Core it didn't have a name and was not possible to change.</span></span> <span data-ttu-id="62b0d-203">它现在称为 `ClientSetNull`，如下一部分所述。</span><span class="sxs-lookup"><span data-stu-id="62b0d-203">It is now known as `ClientSetNull` as described in the next section.</span></span>

<span data-ttu-id="62b0d-204">当删除可选关系中的主体/父实体时，数据库也可配置为级联 NULL。</span><span class="sxs-lookup"><span data-stu-id="62b0d-204">Databases can also be configured to cascade nulls like this when a principal/parent in an optional relationship is deleted.</span></span> <span data-ttu-id="62b0d-205">但是，与在数据库中使用级联删除相比，这种情况要少得多。</span><span class="sxs-lookup"><span data-stu-id="62b0d-205">However, this is much less common than using cascading deletes in the database.</span></span> <span data-ttu-id="62b0d-206">在使用 SQL Server 时，在数据库中同时使用级联删除和级联 NULL 几乎总是会导致关系循环。</span><span class="sxs-lookup"><span data-stu-id="62b0d-206">Using cascading deletes and cascading nulls in the database at the same time will almost always result in relationship cycles when using SQL Server.</span></span> <span data-ttu-id="62b0d-207">若要详细了解如何配置级联 NULL，请参阅下一部分。</span><span class="sxs-lookup"><span data-stu-id="62b0d-207">See the next section for more information on configuring cascading nulls.</span></span>

## <a name="configuring-cascading-behaviors"></a><span data-ttu-id="62b0d-208">配置级联行为</span><span class="sxs-lookup"><span data-stu-id="62b0d-208">Configuring cascading behaviors</span></span>

> [!TIP]
> <span data-ttu-id="62b0d-209">请务必阅读上述部分，然后再转到此处。</span><span class="sxs-lookup"><span data-stu-id="62b0d-209">Be sure to read sections above before coming here.</span></span> <span data-ttu-id="62b0d-210">如果不了解上述资料，那么配置选项可能没有意义。</span><span class="sxs-lookup"><span data-stu-id="62b0d-210">The configuration options will likely not make sense if the preceding material is not understood.</span></span>

<span data-ttu-id="62b0d-211">使用 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> 中的 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.ReferenceCollectionBuilder.OnDelete%2A> 方法按关系配置级联行为。</span><span class="sxs-lookup"><span data-stu-id="62b0d-211">Cascade behaviors are configured per relationship using the <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.ReferenceCollectionBuilder.OnDelete%2A> method in <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A>.</span></span> <span data-ttu-id="62b0d-212">例如：</span><span class="sxs-lookup"><span data-stu-id="62b0d-212">For example:</span></span>

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

<span data-ttu-id="62b0d-213">若要详细了解如何配置实体类型之间的关系，请参阅[关系](xref:core/modeling/relationships)。</span><span class="sxs-lookup"><span data-stu-id="62b0d-213">See [Relationships](xref:core/modeling/relationships) for more information on configuring relationships between entity types.</span></span>

<span data-ttu-id="62b0d-214">`OnDelete` 从公认地令人混淆的 <xref:Microsoft.EntityFrameworkCore.DeleteBehavior> 枚举中接受一个值。</span><span class="sxs-lookup"><span data-stu-id="62b0d-214">`OnDelete` accepts a value from the, admittedly confusing, <xref:Microsoft.EntityFrameworkCore.DeleteBehavior> enum.</span></span> <span data-ttu-id="62b0d-215">该枚举既定义了 EF Core 在跟踪实体上的行为，又定义了使用 EF 创建架构时数据库中级联删除的配置。</span><span class="sxs-lookup"><span data-stu-id="62b0d-215">This enum defines both the behavior of EF Core on tracked entities, and the configuration of cascade delete in the database when EF is used to create the schema.</span></span>

### <a name="impact-on-database-schema"></a><span data-ttu-id="62b0d-216">对数据库架构的影响</span><span class="sxs-lookup"><span data-stu-id="62b0d-216">Impact on database schema</span></span>

<span data-ttu-id="62b0d-217">下表显示了由 EF Core 迁移或 <xref:Microsoft.EntityFrameworkCore.Infrastructure.DatabaseFacade.EnsureCreated%2A> 创建的外键约束上每个 `OnDelete` 值的结果。</span><span class="sxs-lookup"><span data-stu-id="62b0d-217">The following table shows the result of each `OnDelete` value on the foreign key constraint created by EF Core migrations or <xref:Microsoft.EntityFrameworkCore.Infrastructure.DatabaseFacade.EnsureCreated%2A>.</span></span>

| <span data-ttu-id="62b0d-218">DeleteBehavior</span><span class="sxs-lookup"><span data-stu-id="62b0d-218">DeleteBehavior</span></span>        | <span data-ttu-id="62b0d-219">对数据库架构的影响</span><span class="sxs-lookup"><span data-stu-id="62b0d-219">Impact on database schema</span></span>
|:----------------------|--------------------------
| <span data-ttu-id="62b0d-220">Cascade</span><span class="sxs-lookup"><span data-stu-id="62b0d-220">Cascade</span></span>               | <span data-ttu-id="62b0d-221">ON DELETE CASCADE</span><span class="sxs-lookup"><span data-stu-id="62b0d-221">ON DELETE CASCADE</span></span>
| <span data-ttu-id="62b0d-222">限制</span><span class="sxs-lookup"><span data-stu-id="62b0d-222">Restrict</span></span>              | <span data-ttu-id="62b0d-223">ON DELETE NO ACTION</span><span class="sxs-lookup"><span data-stu-id="62b0d-223">ON DELETE NO ACTION</span></span>
| <span data-ttu-id="62b0d-224">NoAction</span><span class="sxs-lookup"><span data-stu-id="62b0d-224">NoAction</span></span>              | <database default>
| <span data-ttu-id="62b0d-225">SetNull</span><span class="sxs-lookup"><span data-stu-id="62b0d-225">SetNull</span></span>               | <span data-ttu-id="62b0d-226">ON DELETE SET NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-226">ON DELETE SET NULL</span></span>
| <span data-ttu-id="62b0d-227">ClientSetNull</span><span class="sxs-lookup"><span data-stu-id="62b0d-227">ClientSetNull</span></span>         | <span data-ttu-id="62b0d-228">ON DELETE NO ACTION</span><span class="sxs-lookup"><span data-stu-id="62b0d-228">ON DELETE NO ACTION</span></span>
| <span data-ttu-id="62b0d-229">ClientCascade</span><span class="sxs-lookup"><span data-stu-id="62b0d-229">ClientCascade</span></span>         | <span data-ttu-id="62b0d-230">ON DELETE NO ACTION</span><span class="sxs-lookup"><span data-stu-id="62b0d-230">ON DELETE NO ACTION</span></span>
| <span data-ttu-id="62b0d-231">ClientNoAction</span><span class="sxs-lookup"><span data-stu-id="62b0d-231">ClientNoAction</span></span>        | <database default>

> [!NOTE]
> <span data-ttu-id="62b0d-232">该表令人困惑，我们计划在将来的版本中重新进行介绍。</span><span class="sxs-lookup"><span data-stu-id="62b0d-232">This table is confusing and we plan to revisit this in a future release.</span></span> <span data-ttu-id="62b0d-233">请参阅 [GitHub 问题 #21252](https://github.com/dotnet/efcore/issues/21252).</span><span class="sxs-lookup"><span data-stu-id="62b0d-233">See [GitHub Issue #21252](https://github.com/dotnet/efcore/issues/21252).</span></span>

<span data-ttu-id="62b0d-234">关系数据库中 `ON DELETE NO ACTION` 和 `ON DELETE RESTRICT` 的行为通常相同或非常相似。</span><span class="sxs-lookup"><span data-stu-id="62b0d-234">The behaviors of `ON DELETE NO ACTION` and `ON DELETE RESTRICT` in relational databases are typically either identical or very similar.</span></span> <span data-ttu-id="62b0d-235">尽管 `NO ACTION` 可能意味着什么，但这两个选项都会导致强制执行引用约束。</span><span class="sxs-lookup"><span data-stu-id="62b0d-235">Despite what `NO ACTION` may imply, both of these options cause referential constraints to be enforced.</span></span> <span data-ttu-id="62b0d-236">区别是当有一个时，是影响数据库何时检查约束。</span><span class="sxs-lookup"><span data-stu-id="62b0d-236">The difference, when there is one, is _when_ the database checks the constraints.</span></span>  <span data-ttu-id="62b0d-237">请查看数据库文档，了解数据库系统上 `ON DELETE NO ACTION` 和 `ON DELETE RESTRICT` 之间的具体区别。</span><span class="sxs-lookup"><span data-stu-id="62b0d-237">Check your database documentation for the specific differences between `ON DELETE NO ACTION` and `ON DELETE RESTRICT` on your database system.</span></span>

<span data-ttu-id="62b0d-238">导致数据库级联行为的唯一值是 `Cascade` 和 `SetNull`。</span><span class="sxs-lookup"><span data-stu-id="62b0d-238">The only values that will cause cascading behaviors on the database are `Cascade` and `SetNull`.</span></span> <span data-ttu-id="62b0d-239">所有其他值会将数据库配置为不级联任何更改。</span><span class="sxs-lookup"><span data-stu-id="62b0d-239">All other values will configure the database to not cascade any changes.</span></span>

### <a name="impact-on-savechanges-behavior"></a><span data-ttu-id="62b0d-240">对 SaveChanges 行为的影响</span><span class="sxs-lookup"><span data-stu-id="62b0d-240">Impact on SaveChanges behavior</span></span>

<span data-ttu-id="62b0d-241">以下各部分中的表格介绍了删除主体/父实体或断开与主体/子实体的关系时，依赖实体/子实体所发生的情况。</span><span class="sxs-lookup"><span data-stu-id="62b0d-241">The tables in the following sections cover what happens to dependent/child entities when the principal/parent is deleted, or its relationship to the dependent/child entities is severed.</span></span> <span data-ttu-id="62b0d-242">每张表都涵盖下述内容之一：</span><span class="sxs-lookup"><span data-stu-id="62b0d-242">Each table covers one of:</span></span>

- <span data-ttu-id="62b0d-243">可选（可为 null 的外键）和必需（不可为 null 的外键）关系</span><span class="sxs-lookup"><span data-stu-id="62b0d-243">Optional (nullable FK) and required (non-nullable FK) relationships</span></span>
- <span data-ttu-id="62b0d-244">依赖项/子项何时由 DbContext 加载和跟踪，以及它们何时仅存在于数据库中</span><span class="sxs-lookup"><span data-stu-id="62b0d-244">When dependents/children are loaded and tracked by the DbContext and when they exist only in the database</span></span>

#### <a name="required-relationship-with-dependentschildren-loaded"></a><span data-ttu-id="62b0d-245">与已加载的依赖项/子项的必需关系</span><span class="sxs-lookup"><span data-stu-id="62b0d-245">Required relationship with dependents/children loaded</span></span>

| <span data-ttu-id="62b0d-246">DeleteBehavior</span><span class="sxs-lookup"><span data-stu-id="62b0d-246">DeleteBehavior</span></span>    | <span data-ttu-id="62b0d-247">删除主体/父实体时</span><span class="sxs-lookup"><span data-stu-id="62b0d-247">On deleting principal/parent</span></span>             | <span data-ttu-id="62b0d-248">断开与主体/父实体的关系时</span><span class="sxs-lookup"><span data-stu-id="62b0d-248">On severing from principal/parent</span></span>
|:------------------|------------------------------------------|----------------------------------------
| <span data-ttu-id="62b0d-249">Cascade</span><span class="sxs-lookup"><span data-stu-id="62b0d-249">Cascade</span></span>           | <span data-ttu-id="62b0d-250">EF Core 删除的依赖项</span><span class="sxs-lookup"><span data-stu-id="62b0d-250">Dependents deleted by EF Core</span></span>            | <span data-ttu-id="62b0d-251">EF Core 删除的依赖项</span><span class="sxs-lookup"><span data-stu-id="62b0d-251">Dependents deleted by EF Core</span></span>
| <span data-ttu-id="62b0d-252">限制</span><span class="sxs-lookup"><span data-stu-id="62b0d-252">Restrict</span></span>          | `InvalidOperationException`              | `InvalidOperationException`
| <span data-ttu-id="62b0d-253">NoAction</span><span class="sxs-lookup"><span data-stu-id="62b0d-253">NoAction</span></span>          | `InvalidOperationException`              | `InvalidOperationException`
| <span data-ttu-id="62b0d-254">SetNull</span><span class="sxs-lookup"><span data-stu-id="62b0d-254">SetNull</span></span>           | <span data-ttu-id="62b0d-255">`SqlException`（创建数据库时）</span><span class="sxs-lookup"><span data-stu-id="62b0d-255">`SqlException` on creating database</span></span>      | <span data-ttu-id="62b0d-256">`SqlException`（创建数据库时）</span><span class="sxs-lookup"><span data-stu-id="62b0d-256">`SqlException` on creating database</span></span>
| <span data-ttu-id="62b0d-257">ClientSetNull</span><span class="sxs-lookup"><span data-stu-id="62b0d-257">ClientSetNull</span></span>     | `InvalidOperationException`              | `InvalidOperationException`
| <span data-ttu-id="62b0d-258">ClientCascade</span><span class="sxs-lookup"><span data-stu-id="62b0d-258">ClientCascade</span></span>     | <span data-ttu-id="62b0d-259">EF Core 删除的依赖项</span><span class="sxs-lookup"><span data-stu-id="62b0d-259">Dependents deleted by EF Core</span></span>            | <span data-ttu-id="62b0d-260">EF Core 删除的依赖项</span><span class="sxs-lookup"><span data-stu-id="62b0d-260">Dependents deleted by EF Core</span></span>
| <span data-ttu-id="62b0d-261">ClientNoAction</span><span class="sxs-lookup"><span data-stu-id="62b0d-261">ClientNoAction</span></span>    | `DbUpdateException`                      | `InvalidOperationException`

<span data-ttu-id="62b0d-262">说明：</span><span class="sxs-lookup"><span data-stu-id="62b0d-262">Notes:</span></span>

- <span data-ttu-id="62b0d-263">这种必需关系的默认值为 `Cascade`。</span><span class="sxs-lookup"><span data-stu-id="62b0d-263">The default for required relationships like this is `Cascade`.</span></span>
- <span data-ttu-id="62b0d-264">调用 SaveChanges 时，对必需关系使用除级联删除以外的其他方法将导致异常。</span><span class="sxs-lookup"><span data-stu-id="62b0d-264">Using anything other than cascade delete for required relationships will result in an exception when SaveChanges is called.</span></span>
  - <span data-ttu-id="62b0d-265">通常，这是来自 EF Core 的 `InvalidOperationException`，因为在已加载的子项/依赖项中检测到无效状态。</span><span class="sxs-lookup"><span data-stu-id="62b0d-265">Typically, this is an `InvalidOperationException` from EF Core since the invalid state is detected in the loaded children/dependents.</span></span>
  - <span data-ttu-id="62b0d-266">`ClientNoAction` 会强制 EF Core 在将依赖项发送到数据库之前不检查修复它们，因此在这种情况下，数据库将引发异常，然后由 SaveChanges 将其包装在 `DbUpdateException` 中。</span><span class="sxs-lookup"><span data-stu-id="62b0d-266">`ClientNoAction` forces EF Core to not check fixup dependents before sending them to the database, so in this case the database throws an exception, which is then wrapped in a `DbUpdateException` by SaveChanges.</span></span>
  - <span data-ttu-id="62b0d-267">创建数据库时会拒绝 `SetNull`，因为外键列不可为 null。</span><span class="sxs-lookup"><span data-stu-id="62b0d-267">`SetNull` is rejected when creating the database since the foreign key column is not nullable.</span></span>
- <span data-ttu-id="62b0d-268">由于已加载依赖项/子项，因此它们始终会被 EF Core 删除，并且永远不会留下来等到数据库被删除。</span><span class="sxs-lookup"><span data-stu-id="62b0d-268">Since dependents/children are loaded, they are always deleted by EF Core, and never left for the database to delete.</span></span>

#### <a name="required-relationship-with-dependentschildren-not-loaded"></a><span data-ttu-id="62b0d-269">与未加载的依赖项/子项的必需关系</span><span class="sxs-lookup"><span data-stu-id="62b0d-269">Required relationship with dependents/children not loaded</span></span>

| <span data-ttu-id="62b0d-270">DeleteBehavior</span><span class="sxs-lookup"><span data-stu-id="62b0d-270">DeleteBehavior</span></span>    | <span data-ttu-id="62b0d-271">删除主体/父实体时</span><span class="sxs-lookup"><span data-stu-id="62b0d-271">On deleting principal/parent</span></span>             | <span data-ttu-id="62b0d-272">断开与主体/父实体的关系时</span><span class="sxs-lookup"><span data-stu-id="62b0d-272">On severing from principal/parent</span></span>
|:------------------|------------------------------------------|----------------------------------------
| <span data-ttu-id="62b0d-273">Cascade</span><span class="sxs-lookup"><span data-stu-id="62b0d-273">Cascade</span></span>           | <span data-ttu-id="62b0d-274">数据库删除的依赖项</span><span class="sxs-lookup"><span data-stu-id="62b0d-274">Dependents deleted by database</span></span>           | <span data-ttu-id="62b0d-275">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-275">N/A</span></span>
| <span data-ttu-id="62b0d-276">限制</span><span class="sxs-lookup"><span data-stu-id="62b0d-276">Restrict</span></span>          | `DbUpdateException`                      | <span data-ttu-id="62b0d-277">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-277">N/A</span></span>
| <span data-ttu-id="62b0d-278">NoAction</span><span class="sxs-lookup"><span data-stu-id="62b0d-278">NoAction</span></span>          | `DbUpdateException`                      | <span data-ttu-id="62b0d-279">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-279">N/A</span></span>
| <span data-ttu-id="62b0d-280">SetNull</span><span class="sxs-lookup"><span data-stu-id="62b0d-280">SetNull</span></span>           | <span data-ttu-id="62b0d-281">`SqlException`（创建数据库时）</span><span class="sxs-lookup"><span data-stu-id="62b0d-281">`SqlException` on creating database</span></span>      | <span data-ttu-id="62b0d-282">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-282">N/A</span></span>
| <span data-ttu-id="62b0d-283">ClientSetNull</span><span class="sxs-lookup"><span data-stu-id="62b0d-283">ClientSetNull</span></span>     | `DbUpdateException`                      | <span data-ttu-id="62b0d-284">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-284">N/A</span></span>
| <span data-ttu-id="62b0d-285">ClientCascade</span><span class="sxs-lookup"><span data-stu-id="62b0d-285">ClientCascade</span></span>     | `DbUpdateException`                      | <span data-ttu-id="62b0d-286">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-286">N/A</span></span>
| <span data-ttu-id="62b0d-287">ClientNoAction</span><span class="sxs-lookup"><span data-stu-id="62b0d-287">ClientNoAction</span></span>    | `DbUpdateException`                      | <span data-ttu-id="62b0d-288">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-288">N/A</span></span>

<span data-ttu-id="62b0d-289">说明：</span><span class="sxs-lookup"><span data-stu-id="62b0d-289">Notes:</span></span>

- <span data-ttu-id="62b0d-290">此处没法断开关系，因为未加载依赖项/子项。</span><span class="sxs-lookup"><span data-stu-id="62b0d-290">Severing a relationship is not valid here since the dependents/children are not loaded.</span></span>
- <span data-ttu-id="62b0d-291">这种必需关系的默认值为 `Cascade`。</span><span class="sxs-lookup"><span data-stu-id="62b0d-291">The default for required relationships like this is `Cascade`.</span></span>
- <span data-ttu-id="62b0d-292">调用 SaveChanges 时，对必需关系使用除级联删除以外的其他方法将导致异常。</span><span class="sxs-lookup"><span data-stu-id="62b0d-292">Using anything other than cascade delete for required relationships will result in an exception when SaveChanges is called.</span></span>
  - <span data-ttu-id="62b0d-293">通常，这是 `DbUpdateException`，理由是未加载依赖项/子项，因此数据库只能检测到无效状态。</span><span class="sxs-lookup"><span data-stu-id="62b0d-293">Typically, this is a `DbUpdateException` because the dependents/children are not loaded, and hence the invalid state can only be detected by the database.</span></span> <span data-ttu-id="62b0d-294">然后，SaveChanges 会将数据库异常包装在 `DbUpdateException` 中。</span><span class="sxs-lookup"><span data-stu-id="62b0d-294">SaveChanges then wraps the database exception in a `DbUpdateException`.</span></span>
  - <span data-ttu-id="62b0d-295">创建数据库时会拒绝 `SetNull`，因为外键列不可为 null。</span><span class="sxs-lookup"><span data-stu-id="62b0d-295">`SetNull` is rejected when creating the database since the foreign key column is not nullable.</span></span>

#### <a name="optional-relationship-with-dependentschildren-loaded"></a><span data-ttu-id="62b0d-296">与已加载的依赖项/子项的可选关系</span><span class="sxs-lookup"><span data-stu-id="62b0d-296">Optional relationship with dependents/children loaded</span></span>

| <span data-ttu-id="62b0d-297">DeleteBehavior</span><span class="sxs-lookup"><span data-stu-id="62b0d-297">DeleteBehavior</span></span>    | <span data-ttu-id="62b0d-298">删除主体/父实体时</span><span class="sxs-lookup"><span data-stu-id="62b0d-298">On deleting principal/parent</span></span>             | <span data-ttu-id="62b0d-299">断开与主体/父实体的关系时</span><span class="sxs-lookup"><span data-stu-id="62b0d-299">On severing from principal/parent</span></span>
|:------------------|------------------------------------------|----------------------------------------
| <span data-ttu-id="62b0d-300">Cascade</span><span class="sxs-lookup"><span data-stu-id="62b0d-300">Cascade</span></span>           | <span data-ttu-id="62b0d-301">EF Core 删除的依赖项</span><span class="sxs-lookup"><span data-stu-id="62b0d-301">Dependents deleted by EF Core</span></span>            | <span data-ttu-id="62b0d-302">EF Core 删除的依赖项</span><span class="sxs-lookup"><span data-stu-id="62b0d-302">Dependents deleted by EF Core</span></span>
| <span data-ttu-id="62b0d-303">限制</span><span class="sxs-lookup"><span data-stu-id="62b0d-303">Restrict</span></span>          | <span data-ttu-id="62b0d-304">EF Core 将依赖外键设置为 NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-304">Dependent FKs set to null by EF Core</span></span>     | <span data-ttu-id="62b0d-305">EF Core 将依赖外键设置为 NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-305">Dependent FKs set to null by EF Core</span></span>
| <span data-ttu-id="62b0d-306">NoAction</span><span class="sxs-lookup"><span data-stu-id="62b0d-306">NoAction</span></span>          | <span data-ttu-id="62b0d-307">EF Core 将依赖外键设置为 NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-307">Dependent FKs set to null by EF Core</span></span>     | <span data-ttu-id="62b0d-308">EF Core 将依赖外键设置为 NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-308">Dependent FKs set to null by EF Core</span></span>
| <span data-ttu-id="62b0d-309">SetNull</span><span class="sxs-lookup"><span data-stu-id="62b0d-309">SetNull</span></span>           | <span data-ttu-id="62b0d-310">EF Core 将依赖外键设置为 NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-310">Dependent FKs set to null by EF Core</span></span>     | <span data-ttu-id="62b0d-311">EF Core 将依赖外键设置为 NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-311">Dependent FKs set to null by EF Core</span></span>
| <span data-ttu-id="62b0d-312">ClientSetNull</span><span class="sxs-lookup"><span data-stu-id="62b0d-312">ClientSetNull</span></span>     | <span data-ttu-id="62b0d-313">EF Core 将依赖外键设置为 NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-313">Dependent FKs set to null by EF Core</span></span>     | <span data-ttu-id="62b0d-314">EF Core 将依赖外键设置为 NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-314">Dependent FKs set to null by EF Core</span></span>
| <span data-ttu-id="62b0d-315">ClientCascade</span><span class="sxs-lookup"><span data-stu-id="62b0d-315">ClientCascade</span></span>     | <span data-ttu-id="62b0d-316">EF Core 删除的依赖项</span><span class="sxs-lookup"><span data-stu-id="62b0d-316">Dependents deleted by EF Core</span></span>            | <span data-ttu-id="62b0d-317">EF Core 删除的依赖项</span><span class="sxs-lookup"><span data-stu-id="62b0d-317">Dependents deleted by EF Core</span></span>
| <span data-ttu-id="62b0d-318">ClientNoAction</span><span class="sxs-lookup"><span data-stu-id="62b0d-318">ClientNoAction</span></span>    | `DbUpdateException`                      | <span data-ttu-id="62b0d-319">EF Core 将依赖外键设置为 NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-319">Dependent FKs set to null by EF Core</span></span>

<span data-ttu-id="62b0d-320">说明：</span><span class="sxs-lookup"><span data-stu-id="62b0d-320">Notes:</span></span>

- <span data-ttu-id="62b0d-321">这种可选关系的默认值为 `ClientSetNull`。</span><span class="sxs-lookup"><span data-stu-id="62b0d-321">The default for optional relationships like this is `ClientSetNull`.</span></span>
- <span data-ttu-id="62b0d-322">永远不会删除依赖项/子项，除非配置了 `Cascade` 或 `ClientCascade`。</span><span class="sxs-lookup"><span data-stu-id="62b0d-322">Dependents/children are never deleted unless `Cascade` or `ClientCascade` are configured.</span></span>
- <span data-ttu-id="62b0d-323">所有其他值都会导致 EF Core 将依赖外键设置为 NULL...</span><span class="sxs-lookup"><span data-stu-id="62b0d-323">All other values cause the dependent FKs to be set to null by EF Core...</span></span>
  - <span data-ttu-id="62b0d-324">... `ClientNoAction` 除外，它指示 EF Core 在删除主体/父实体时不处理依赖项/子项的外键。</span><span class="sxs-lookup"><span data-stu-id="62b0d-324">...except `ClientNoAction` which tells EF Core not to touch the foreign keys of dependents/children when the principal/parent is deleted.</span></span> <span data-ttu-id="62b0d-325">因此，数据库会引发异常，由 SaveChanges 将其包装为 `DbUpdateException`。</span><span class="sxs-lookup"><span data-stu-id="62b0d-325">The database therefore throws an exception, which is wrapped as a `DbUpdateException` by SaveChanges.</span></span>

#### <a name="optional-relationship-with-dependentschildren-not-loaded"></a><span data-ttu-id="62b0d-326">与未加载的依赖项/子项的可选关系</span><span class="sxs-lookup"><span data-stu-id="62b0d-326">Optional relationship with dependents/children not loaded</span></span>

| <span data-ttu-id="62b0d-327">DeleteBehavior</span><span class="sxs-lookup"><span data-stu-id="62b0d-327">DeleteBehavior</span></span>    | <span data-ttu-id="62b0d-328">删除主体/父实体时</span><span class="sxs-lookup"><span data-stu-id="62b0d-328">On deleting principal/parent</span></span>             | <span data-ttu-id="62b0d-329">断开与主体/父实体的关系时</span><span class="sxs-lookup"><span data-stu-id="62b0d-329">On severing from principal/parent</span></span>
|:------------------|------------------------------------------|----------------------------------------
| <span data-ttu-id="62b0d-330">Cascade</span><span class="sxs-lookup"><span data-stu-id="62b0d-330">Cascade</span></span>           | <span data-ttu-id="62b0d-331">数据库删除的依赖项</span><span class="sxs-lookup"><span data-stu-id="62b0d-331">Dependents deleted by database</span></span>           | <span data-ttu-id="62b0d-332">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-332">N/A</span></span>
| <span data-ttu-id="62b0d-333">限制</span><span class="sxs-lookup"><span data-stu-id="62b0d-333">Restrict</span></span>          | `DbUpdateException`                      | <span data-ttu-id="62b0d-334">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-334">N/A</span></span>
| <span data-ttu-id="62b0d-335">NoAction</span><span class="sxs-lookup"><span data-stu-id="62b0d-335">NoAction</span></span>          | `DbUpdateException`                      | <span data-ttu-id="62b0d-336">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-336">N/A</span></span>
| <span data-ttu-id="62b0d-337">SetNull</span><span class="sxs-lookup"><span data-stu-id="62b0d-337">SetNull</span></span>           | <span data-ttu-id="62b0d-338">数据库将依赖外键设置为 NULL</span><span class="sxs-lookup"><span data-stu-id="62b0d-338">Dependent FKs set to null by database</span></span>    | <span data-ttu-id="62b0d-339">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-339">N/A</span></span>
| <span data-ttu-id="62b0d-340">ClientSetNull</span><span class="sxs-lookup"><span data-stu-id="62b0d-340">ClientSetNull</span></span>     | `DbUpdateException`                      | <span data-ttu-id="62b0d-341">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-341">N/A</span></span>
| <span data-ttu-id="62b0d-342">ClientCascade</span><span class="sxs-lookup"><span data-stu-id="62b0d-342">ClientCascade</span></span>     | `DbUpdateException`                      | <span data-ttu-id="62b0d-343">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-343">N/A</span></span>
| <span data-ttu-id="62b0d-344">ClientNoAction</span><span class="sxs-lookup"><span data-stu-id="62b0d-344">ClientNoAction</span></span>    | `DbUpdateException`                      | <span data-ttu-id="62b0d-345">不可用</span><span class="sxs-lookup"><span data-stu-id="62b0d-345">N/A</span></span>

<span data-ttu-id="62b0d-346">说明：</span><span class="sxs-lookup"><span data-stu-id="62b0d-346">Notes:</span></span>

- <span data-ttu-id="62b0d-347">此处没法断开关系，因为未加载依赖项/子项。</span><span class="sxs-lookup"><span data-stu-id="62b0d-347">Severing a relationship is not valid here since the dependents/children are not loaded.</span></span>
- <span data-ttu-id="62b0d-348">这种可选关系的默认值为 `ClientSetNull`。</span><span class="sxs-lookup"><span data-stu-id="62b0d-348">The default for optional relationships like this is `ClientSetNull`.</span></span>
- <span data-ttu-id="62b0d-349">除非已将数据库配置为级联删除或级联 NULL，否则必须加载依赖项/子项以避免数据库异常。</span><span class="sxs-lookup"><span data-stu-id="62b0d-349">Dependents/children must be loaded to avoid a database exception unless the database has been configured to cascade either deletes or nulls.</span></span>
