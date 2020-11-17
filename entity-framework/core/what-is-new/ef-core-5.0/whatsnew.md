---
title: EF Core 5.0 中的新增功能
description: EF Core 5.0 中的新功能概述
author: ajcvickers
ms.date: 09/10/2020
uid: core/what-is-new/ef-core-5.0/whatsnew
ms.openlocfilehash: 3efa883cdfac1ecd412112ef06c7763f1a7e12f1
ms.sourcegitcommit: f3512e3a98e685a3ba409c1d0157ce85cc390cf4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2020
ms.locfileid: "94429242"
---
# <a name="whats-new-in-ef-core-50"></a><span data-ttu-id="bc72a-103">EF Core 5.0 中的新增功能</span><span class="sxs-lookup"><span data-stu-id="bc72a-103">What's New in EF Core 5.0</span></span>

<span data-ttu-id="bc72a-104">为 EF Core 5.0 计划的所有功能现已完成。</span><span class="sxs-lookup"><span data-stu-id="bc72a-104">All features planned for EF Core 5.0 have now been completed.</span></span> <span data-ttu-id="bc72a-105">此页面包含每个预览版中引入的令人关注的更改的简要介绍。</span><span class="sxs-lookup"><span data-stu-id="bc72a-105">This page contains an overview of interesting changes introduced in each preview.</span></span>

<span data-ttu-id="bc72a-106">此页面不会复制 [EF Core 5.0](xref:core/what-is-new/ef-core-5.0/plan) 的计划。</span><span class="sxs-lookup"><span data-stu-id="bc72a-106">This page does not duplicate the [plan for EF Core 5.0](xref:core/what-is-new/ef-core-5.0/plan).</span></span> <span data-ttu-id="bc72a-107">计划中介绍了 EF Core 5.0 的整体主题，其中包括我们在交付最终版本之前打算包含的所有内容。</span><span class="sxs-lookup"><span data-stu-id="bc72a-107">The plan describes the overall themes for EF Core 5.0, including everything we are planning to include before shipping the final release.</span></span>

## <a name="rc1"></a><span data-ttu-id="bc72a-108">RC1</span><span class="sxs-lookup"><span data-stu-id="bc72a-108">RC1</span></span>

### <a name="many-to-many"></a><span data-ttu-id="bc72a-109">多对多</span><span class="sxs-lookup"><span data-stu-id="bc72a-109">Many-to-many</span></span>

<span data-ttu-id="bc72a-110">EF Core 5.0 支持多对多关系，而无需显式映射联接表。</span><span class="sxs-lookup"><span data-stu-id="bc72a-110">EF Core 5.0 supports many-to-many relationships without explicitly mapping the join table.</span></span>

<span data-ttu-id="bc72a-111">以下面这些实体类型为例：</span><span class="sxs-lookup"><span data-stu-id="bc72a-111">For example, consider these entity types:</span></span>

```csharp
public class Post
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Tag> Tags { get; set; }
}

public class Tag
{
    public int Id { get; set; }
    public string Text { get; set; }
    public ICollection<Post> Posts { get; set; }
}
```

<span data-ttu-id="bc72a-112">请注意，`Post` 包含 `Tags` 的集合，`Tag` 包含 `Posts` 的集合。</span><span class="sxs-lookup"><span data-stu-id="bc72a-112">Notice that `Post` contains a collection of `Tags`, and `Tag` contains a collection of `Posts`.</span></span> <span data-ttu-id="bc72a-113">EF Core 5.0 按照约定将此识别为多对多关系。</span><span class="sxs-lookup"><span data-stu-id="bc72a-113">EF Core 5.0 recognizes this as a many-to-many relationship by convention.</span></span> <span data-ttu-id="bc72a-114">这意味着在 `OnModelCreating` 中不需要任何代码：</span><span class="sxs-lookup"><span data-stu-id="bc72a-114">This means no code is required in `OnModelCreating`:</span></span>

```csharp
public class BlogContext : DbContext
{
    public DbSet<Post> Posts { get; set; }
    public DbSet<Tag> Tags { get; set; }
}
```

<span data-ttu-id="bc72a-115">当使用迁移（或 `EnsureCreated`）创建数据库时，EF Core 将自动创建联接表。</span><span class="sxs-lookup"><span data-stu-id="bc72a-115">When Migrations (or `EnsureCreated`) are used to create the database, EF Core will automatically create the join table.</span></span> <span data-ttu-id="bc72a-116">例如，在此模型的 SQL Server 上，EF Core 生成：</span><span class="sxs-lookup"><span data-stu-id="bc72a-116">For example, on SQL Server for this model, EF Core generates:</span></span>

```sql
CREATE TABLE [Posts] (
    [Id] int NOT NULL IDENTITY,
    [Name] nvarchar(max) NULL,
    CONSTRAINT [PK_Posts] PRIMARY KEY ([Id])
);

CREATE TABLE [Tag] (
    [Id] int NOT NULL IDENTITY,
    [Text] nvarchar(max) NULL,
    CONSTRAINT [PK_Tag] PRIMARY KEY ([Id])
);

CREATE TABLE [PostTag] (
    [PostsId] int NOT NULL,
    [TagsId] int NOT NULL,
    CONSTRAINT [PK_PostTag] PRIMARY KEY ([PostsId], [TagsId]),
    CONSTRAINT [FK_PostTag_Posts_PostsId] FOREIGN KEY ([PostsId]) REFERENCES [Posts] ([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_PostTag_Tag_TagsId] FOREIGN KEY ([TagsId]) REFERENCES [Tag] ([Id]) ON DELETE CASCADE
);

CREATE INDEX [IX_PostTag_TagsId] ON [PostTag] ([TagsId]);
```

<span data-ttu-id="bc72a-117">创建和关联 `Tag` 和 `Post` 实体会导致联接表更新自动发生。</span><span class="sxs-lookup"><span data-stu-id="bc72a-117">Creating and associating `Tag` and `Post` entities results in join table updates happening automatically.</span></span> <span data-ttu-id="bc72a-118">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-118">For example:</span></span>

```csharp
var beginnerTag = new Tag {Text = "Beginner"};
var advancedTag = new Tag {Text = "Advanced"};
var efCoreTag = new Tag {Text = "EF Core"};

context.AddRange(
    new Post {Name = "EF Core 101", Tags = new List<Tag> {beginnerTag, efCoreTag}},
    new Post {Name = "Writing an EF database provider", Tags = new List<Tag> {advancedTag, efCoreTag}},
    new Post {Name = "Savepoints in EF Core", Tags = new List<Tag> {beginnerTag, efCoreTag}});

context.SaveChanges();
```

<span data-ttu-id="bc72a-119">插入帖子和标记后，EF 将自动在联接表中创建行。</span><span class="sxs-lookup"><span data-stu-id="bc72a-119">After inserting the Posts and Tags, EF will then automatically create rows in the join table.</span></span> <span data-ttu-id="bc72a-120">例如，在 SQL Server 上：</span><span class="sxs-lookup"><span data-stu-id="bc72a-120">For example, on SQL Server:</span></span>

```sql
SET NOCOUNT ON;
INSERT INTO [PostTag] ([PostsId], [TagsId])
VALUES (@p6, @p7),
(@p8, @p9),
(@p10, @p11),
(@p12, @p13),
(@p14, @p15),
(@p16, @p17);
```

<span data-ttu-id="bc72a-121">对于查询，Include 和其他查询操作的工作方式与任何其他关系一样。</span><span class="sxs-lookup"><span data-stu-id="bc72a-121">For queries, Include and other query operations work just like for any other relationship.</span></span> <span data-ttu-id="bc72a-122">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-122">For example:</span></span>

```csharp
foreach (var post in context.Posts.Include(e => e.Tags))
{
    Console.Write($"Post \"{post.Name}\" has tags");

    foreach (var tag in post.Tags)
    {
        Console.Write($" '{tag.Text}'");
    }
}
```

<span data-ttu-id="bc72a-123">生成的 SQL 自动使用联接表来恢复所有相关的标记：</span><span class="sxs-lookup"><span data-stu-id="bc72a-123">The SQL generated uses the join table automatically to bring back all related Tags:</span></span>

```sql
SELECT [p].[Id], [p].[Name], [t0].[PostsId], [t0].[TagsId], [t0].[Id], [t0].[Text]
FROM [Posts] AS [p]
LEFT JOIN (
    SELECT [p0].[PostsId], [p0].[TagsId], [t].[Id], [t].[Text]
    FROM [PostTag] AS [p0]
    INNER JOIN [Tag] AS [t] ON [p0].[TagsId] = [t].[Id]
) AS [t0] ON [p].[Id] = [t0].[PostsId]
ORDER BY [p].[Id], [t0].[PostsId], [t0].[TagsId], [t0].[Id]
```

<span data-ttu-id="bc72a-124">与 EF6 不同，EF Core 允许完全自定义联接表。</span><span class="sxs-lookup"><span data-stu-id="bc72a-124">Unlike EF6, EF Core allows full customization of the join table.</span></span> <span data-ttu-id="bc72a-125">例如，下面的代码配置了多对多关系，该关系也具有联接实体的导航，其中联接实体包含有效负载属性：</span><span class="sxs-lookup"><span data-stu-id="bc72a-125">For example, the code below configures a many-to-many relationship that also has navigations to the join entity, and in which the join entity contains a payload property:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<Post>()
        .HasMany(p => p.Tags)
        .WithMany(p => p.Posts)
        .UsingEntity<PostTag>(
            j => j
                .HasOne(pt => pt.Tag)
                .WithMany()
                .HasForeignKey(pt => pt.TagId),
            j => j
                .HasOne(pt => pt.Post)
                .WithMany()
                .HasForeignKey(pt => pt.PostId),
            j =>
            {
                j.Property(pt => pt.PublicationDate).HasDefaultValueSql("CURRENT_TIMESTAMP");
                j.HasKey(t => new { t.PostId, t.TagId });
            });
}
```

> [!NOTE]
> <span data-ttu-id="bc72a-126">尚未添加对数据库中多对多关系搭建基架的支持。</span><span class="sxs-lookup"><span data-stu-id="bc72a-126">Support for scaffolding many-to-many relationships from the database is not yet added.</span></span> <span data-ttu-id="bc72a-127">请参阅[跟踪问题](https://github.com/dotnet/efcore/issues/22475)。</span><span class="sxs-lookup"><span data-stu-id="bc72a-127">See [tracking issue](https://github.com/dotnet/efcore/issues/22475).</span></span>

### <a name="map-entity-types-to-queries"></a><span data-ttu-id="bc72a-128">将实体类型映射到查询</span><span class="sxs-lookup"><span data-stu-id="bc72a-128">Map entity types to queries</span></span>

<span data-ttu-id="bc72a-129">实体类型通常映射到表或视图，以便 EF Core 在查询该类型时将拉回表或视图的内容。</span><span class="sxs-lookup"><span data-stu-id="bc72a-129">Entity types are commonly mapped to tables or views such that EF Core will pull back the contents of the table or view when querying for that type.</span></span> <span data-ttu-id="bc72a-130">EF Core 5.0 允许实体类型映射到“定义查询”。</span><span class="sxs-lookup"><span data-stu-id="bc72a-130">EF Core 5.0 allows an entity type to mapped to a "defining query".</span></span> <span data-ttu-id="bc72a-131">（这在以前的版本中部分支持，但在 EF Core 5.0 中已显著改进，并使用不同的语法。）</span><span class="sxs-lookup"><span data-stu-id="bc72a-131">(This was partially supported in previous versions, but is much improved and has different syntax in EF Core 5.0.)</span></span>

<span data-ttu-id="bc72a-132">例如，考虑两个表：一个具有新式帖子；另一个具有旧式帖子。</span><span class="sxs-lookup"><span data-stu-id="bc72a-132">For example, consider two tables; one with modern posts; the other with legacy posts.</span></span> <span data-ttu-id="bc72a-133">新式帖子表有一些额外的列，但为了我们的应用程序，我们希望将新式和旧式帖子合并和映射到具有所有必需属性的实体类型：</span><span class="sxs-lookup"><span data-stu-id="bc72a-133">The modern posts table has some additional columns, but for the purpose of our application we want both modern and legacy posts to be combined and mapped to an entity type with all necessary properties:</span></span>

```csharp
public class Post
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Category { get; set; }
    public int BlogId { get; set; }
    public Blog Blog { get; set; }
}
```

<span data-ttu-id="bc72a-134">在 EF Core 5.0 中，`ToSqlQuery` 可用于将此实体类型映射到从两个表中提取和合并行的查询：</span><span class="sxs-lookup"><span data-stu-id="bc72a-134">In EF Core 5.0, `ToSqlQuery` can be used to map this entity type to a query that pulls and combines rows from both tables:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Post>().ToSqlQuery(
        @"SELECT Id, Name, Category, BlogId FROM posts
          UNION ALL
          SELECT Id, Name, ""Legacy"", BlogId from legacy_posts");
}
```

<span data-ttu-id="bc72a-135">请注意，`legacy_posts` 表没有 `Category` 列，因此我们改为合成所有旧帖子的默认值。</span><span class="sxs-lookup"><span data-stu-id="bc72a-135">Notice that the `legacy_posts` table does not have a `Category` column, so we instead synthesize a default value for all legacy posts.</span></span>

<span data-ttu-id="bc72a-136">然后，此实体类型可以正常用于 LINQ 查询。</span><span class="sxs-lookup"><span data-stu-id="bc72a-136">This entity type can then be used in the normal way for LINQ queries.</span></span> <span data-ttu-id="bc72a-137">例如，</span><span class="sxs-lookup"><span data-stu-id="bc72a-137">For example.</span></span> <span data-ttu-id="bc72a-138">LINQ 查询：</span><span class="sxs-lookup"><span data-stu-id="bc72a-138">the LINQ query:</span></span>

```csharp
var posts = context.Posts.Where(e => e.Blog.Name.Contains("Unicorn")).ToList();
```

<span data-ttu-id="bc72a-139">在 SQLite 上生成以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="bc72a-139">Generates the following SQL on SQLite:</span></span>

```sql
SELECT "p"."Id", "p"."BlogId", "p"."Category", "p"."Name"
FROM (
    SELECT Id, Name, Category, BlogId FROM posts
    UNION ALL
    SELECT Id, Name, "Legacy", BlogId from legacy_posts
) AS "p"
INNER JOIN "Blogs" AS "b" ON "p"."BlogId" = "b"."Id"
WHERE ('Unicorn' = '') OR (instr("b"."Name", 'Unicorn') > 0)
```

<span data-ttu-id="bc72a-140">请注意，为实体类型配置的查询用作撰写完整 LINQ 查询的起始。</span><span class="sxs-lookup"><span data-stu-id="bc72a-140">Notice that the query configured for the entity type is used as a starting for composing the full LINQ query.</span></span>

### <a name="event-counters"></a><span data-ttu-id="bc72a-141">事件计数器</span><span class="sxs-lookup"><span data-stu-id="bc72a-141">Event counters</span></span>

<span data-ttu-id="bc72a-142">[.NET 事件计数器](https://devblogs.microsoft.com/dotnet/introducing-diagnostics-improvements-in-net-core-3-0/)是有效公开应用程序性能指标的一种方法。</span><span class="sxs-lookup"><span data-stu-id="bc72a-142">[.NET event counters](https://devblogs.microsoft.com/dotnet/introducing-diagnostics-improvements-in-net-core-3-0/) are a way to efficiently expose performance metrics from an application.</span></span> <span data-ttu-id="bc72a-143">EF Core 5.0 包括 `Microsoft.EntityFrameworkCore` 类别下的事件计数器。</span><span class="sxs-lookup"><span data-stu-id="bc72a-143">EF Core 5.0 includes event counters under the `Microsoft.EntityFrameworkCore` category.</span></span> <span data-ttu-id="bc72a-144">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-144">For example:</span></span>

```console
dotnet counters monitor Microsoft.EntityFrameworkCore -p 49496
```

<span data-ttu-id="bc72a-145">这指示 dotnet 计数器开始收集进程 49496 的 EF Core 事件。</span><span class="sxs-lookup"><span data-stu-id="bc72a-145">This tells dotnet counters to start collecting EF Core events for process 49496.</span></span> <span data-ttu-id="bc72a-146">这将在控制台中生成如下所示的输出：</span><span class="sxs-lookup"><span data-stu-id="bc72a-146">This generates output like this in the console:</span></span>

```console
[Microsoft.EntityFrameworkCore]
    Active DbContexts                                               1
    Execution Strategy Operation Failures (Count / 1 sec)           0
    Execution Strategy Operation Failures (Total)                   0
    Optimistic Concurrency Failures (Count / 1 sec)                 0
    Optimistic Concurrency Failures (Total)                         0
    Queries (Count / 1 sec)                                     1,755
    Queries (Total)                                            98,402
    Query Cache Hit Rate (%)                                      100
    SaveChanges (Count / 1 sec)                                     0
    SaveChanges (Total)                                             1
```

### <a name="property-bags"></a><span data-ttu-id="bc72a-147">属性包</span><span class="sxs-lookup"><span data-stu-id="bc72a-147">Property bags</span></span>

<span data-ttu-id="bc72a-148">EF Core 5.0 允许将相同的 CLR 类型映射到多个不同的实体类型。</span><span class="sxs-lookup"><span data-stu-id="bc72a-148">EF Core 5.0 allows the same CLR type to be mapped to multiple different entity types.</span></span> <span data-ttu-id="bc72a-149">此类类型称为共享类型实体类型。</span><span class="sxs-lookup"><span data-stu-id="bc72a-149">Such types are known as shared-type entity types.</span></span> <span data-ttu-id="bc72a-150">此功能与索引器属性（包含在预览 1 中）相结合，允许将属性包用作实体类型。</span><span class="sxs-lookup"><span data-stu-id="bc72a-150">This feature combined with indexer properties (included in preview 1) allows property bags to be used as entity type.</span></span>

<span data-ttu-id="bc72a-151">例如，下面的 DbContext 将 BCL 类型 `Dictionary<string, object>` 配置为产品和类别的共享类型实体类型。</span><span class="sxs-lookup"><span data-stu-id="bc72a-151">For example, the DbContext below configures the BCL type `Dictionary<string, object>` as a shared-type entity type for both products and categories.</span></span>

```csharp
public class ProductsContext : DbContext
{
    public DbSet<Dictionary<string, object>> Products => Set<Dictionary<string, object>>("Product");
    public DbSet<Dictionary<string, object>> Categories => Set<Dictionary<string, object>>("Category");

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.SharedTypeEntity<Dictionary<string, object>>("Category", b =>
        {
            b.IndexerProperty<string>("Description");
            b.IndexerProperty<int>("Id");
            b.IndexerProperty<string>("Name").IsRequired();
        });

        modelBuilder.SharedTypeEntity<Dictionary<string, object>>("Product", b =>
        {
            b.IndexerProperty<int>("Id");
            b.IndexerProperty<string>("Name").IsRequired();
            b.IndexerProperty<string>("Description");
            b.IndexerProperty<decimal>("Price");
            b.IndexerProperty<int?>("CategoryId");

            b.HasOne("Category", null).WithMany();
        });
    }
}
```

<span data-ttu-id="bc72a-152">字典对象（“属性包”）现在可以作为实体实例添加到上下文中并保存。</span><span class="sxs-lookup"><span data-stu-id="bc72a-152">Dictionary objects ("property bags") can now be added to the context as entity instances and saved.</span></span> <span data-ttu-id="bc72a-153">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-153">For example:</span></span>

```csharp
var beverages = new Dictionary<string, object>
{
    ["Name"] = "Beverages",
    ["Description"] = "Stuff to sip on"
};

context.Categories.Add(beverages);

context.SaveChanges();
```

<span data-ttu-id="bc72a-154">然后，可以以正常方式查询和更新这些实体：</span><span class="sxs-lookup"><span data-stu-id="bc72a-154">These entities can then be queried and updated in the normal way:</span></span>

```csharp
var foods = context.Categories.Single(e => e["Name"] == "Foods");
var marmite = context.Products.Single(e => e["Name"] == "Marmite");

marmite["CategoryId"] = foods["Id"];
marmite["Description"] = "Yummy when spread _thinly_ on buttered Toast!";

context.SaveChanges();
```

### <a name="savechanges-interception-and-events"></a><span data-ttu-id="bc72a-155">SaveChanges 拦截和事件</span><span class="sxs-lookup"><span data-stu-id="bc72a-155">SaveChanges interception and events</span></span>

<span data-ttu-id="bc72a-156">EF Core 5.0 引入调用 SaveChanges 时触发的 .NET 事件和 EF Core 拦截器。</span><span class="sxs-lookup"><span data-stu-id="bc72a-156">EF Core 5.0 introduces both .NET events and an EF Core interceptor triggered when SaveChanges is called.</span></span>

<span data-ttu-id="bc72a-157">事件简单易用；例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-157">The events are simple to use; for example:</span></span>

```csharp
context.SavingChanges += (sender, args) =>
{
    Console.WriteLine($"Saving changes for {((DbContext)sender).Database.GetConnectionString()}");
};

context.SavedChanges += (sender, args) =>
{
    Console.WriteLine($"Saved {args.EntitiesSavedCount} changes for {((DbContext)sender).Database.GetConnectionString()}");
};
```

<span data-ttu-id="bc72a-158">请注意：</span><span class="sxs-lookup"><span data-stu-id="bc72a-158">Notice that:</span></span>

* <span data-ttu-id="bc72a-159">事件发送方是 `DbContext` 实例</span><span class="sxs-lookup"><span data-stu-id="bc72a-159">The event sender is the `DbContext` instance</span></span>
* <span data-ttu-id="bc72a-160">`SavedChanges` 事件的参数包含保存到数据库的实体数</span><span class="sxs-lookup"><span data-stu-id="bc72a-160">The args for the `SavedChanges` event contains the number of entities saved to the database</span></span>

<span data-ttu-id="bc72a-161">拦截器由 `ISaveChangesInterceptor` 定义，但通常从 `SaveChangesInterceptor` 继承，以便避免实现每个方法。</span><span class="sxs-lookup"><span data-stu-id="bc72a-161">The interceptor is defined by `ISaveChangesInterceptor`, but it is often convienient to inherit from `SaveChangesInterceptor` to avoid implementing every method.</span></span> <span data-ttu-id="bc72a-162">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-162">For example:</span></span>

```csharp
public class MySaveChangesInterceptor : SaveChangesInterceptor
{
    public override InterceptionResult<int> SavingChanges(
        DbContextEventData eventData,
        InterceptionResult<int> result)
    {
        Console.WriteLine($"Saving changes for {eventData.Context.Database.GetConnectionString()}");

        return result;
    }

    public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
        DbContextEventData eventData,
        InterceptionResult<int> result,
        CancellationToken cancellationToken = new CancellationToken())
    {
        Console.WriteLine($"Saving changes asynchronously for {eventData.Context.Database.GetConnectionString()}");

        return new ValueTask<InterceptionResult<int>>(result);
    }
}
```

<span data-ttu-id="bc72a-163">请注意：</span><span class="sxs-lookup"><span data-stu-id="bc72a-163">Notice that:</span></span>

* <span data-ttu-id="bc72a-164">拦截器具有同步和异步方法。</span><span class="sxs-lookup"><span data-stu-id="bc72a-164">The interceptor has both sync and async methods.</span></span> <span data-ttu-id="bc72a-165">如果需要执行异步 I/O（例如写入审核服务器），这非常有用。</span><span class="sxs-lookup"><span data-stu-id="bc72a-165">This can be useful if you need to perform async I/O, such as writing to an audit server.</span></span>
* <span data-ttu-id="bc72a-166">拦截器允许使用所有拦截器通用的 `InterceptionResult` 机制跳过 SaveChanges。</span><span class="sxs-lookup"><span data-stu-id="bc72a-166">The interceptor allows SaveChanges to be skipped using the `InterceptionResult` mechanism common to all interceptors.</span></span>

<span data-ttu-id="bc72a-167">拦截器的缺点是，构建 DbContext 时，必须在 DbContext 上注册它们。</span><span class="sxs-lookup"><span data-stu-id="bc72a-167">The downside of interceptors is that they must be registered on the DbContext when it is being constructed.</span></span> <span data-ttu-id="bc72a-168">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-168">For example:</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .AddInterceptors(new MySaveChangesInterceptor())
        .UseSqlite("Data Source = test.db");
```

<span data-ttu-id="bc72a-169">相反，可以随时在 DbContext 实例上注册事件。</span><span class="sxs-lookup"><span data-stu-id="bc72a-169">In contrast, the events can be registered on the DbContext instance at any time.</span></span>

### <a name="exclude-tables-from-migrations"></a><span data-ttu-id="bc72a-170">从迁移中排除表</span><span class="sxs-lookup"><span data-stu-id="bc72a-170">Exclude tables from migrations</span></span>

<span data-ttu-id="bc72a-171">有时在多个 DbContext 中映射单个实体类型很有用。</span><span class="sxs-lookup"><span data-stu-id="bc72a-171">It is sometimes useful to have a single entity type mapped in multiple DbContexts.</span></span> <span data-ttu-id="bc72a-172">在使用[有界上下文](https://www.martinfowler.com/bliki/BoundedContext.html)时尤其如此，对于这些上下文，针对每段有界上下文使用不同 DbContext 类型的情况很常见。</span><span class="sxs-lookup"><span data-stu-id="bc72a-172">This is especially true when using [bounded contexts](https://www.martinfowler.com/bliki/BoundedContext.html), for which it is common to have a different DbContext type for each bounded context.</span></span>

<span data-ttu-id="bc72a-173">例如，授权上下文和报告上下文都可能需要 `User` 类型。</span><span class="sxs-lookup"><span data-stu-id="bc72a-173">For example, a `User` type may be needed by both an authorization context and a reporting context.</span></span> <span data-ttu-id="bc72a-174">如果对 `User` 类型进行了更改，则两个 DbContext 的迁移都将尝试更新数据库。</span><span class="sxs-lookup"><span data-stu-id="bc72a-174">If a change is made to the `User` type, then migrations for both DbContexts will attempt to update the database.</span></span> <span data-ttu-id="bc72a-175">为了防止这种情况，可将其中一个上下文的模型配置为从其迁移中排除表。</span><span class="sxs-lookup"><span data-stu-id="bc72a-175">To prevent this, the model for one of the contexts can be configured to exclude the table from its migrations.</span></span>

<span data-ttu-id="bc72a-176">在下面的代码中，`AuthorizationContext` 将生成对 `Users` 表的更改的迁移，但 `ReportingContext` 不会，从而防止迁移冲突。</span><span class="sxs-lookup"><span data-stu-id="bc72a-176">In the code below, the `AuthorizationContext` will generate migrations for changes to the `Users` table, but the `ReportingContext` will not, preventing the migrations from clashing.</span></span>

```csharp
public class AuthorizationContext : DbContext
{
    public DbSet<User> Users { get; set; }
}

public class ReportingContext : DbContext
{
    public DbSet<User> Users { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>().ToTable("Users", t => t.ExcludeFromMigrations());
    }
}
```

### <a name="required-11-dependents"></a><span data-ttu-id="bc72a-177">必需的一对一依赖项</span><span class="sxs-lookup"><span data-stu-id="bc72a-177">Required 1:1 dependents</span></span>

<span data-ttu-id="bc72a-178">在 EF Core 3.1 中，一对一关系的依赖端始终被视为可选。</span><span class="sxs-lookup"><span data-stu-id="bc72a-178">In EF Core 3.1, the dependent end of a one-to-one relationship was always considered optional.</span></span> <span data-ttu-id="bc72a-179">这在使用从属实体时最明显。</span><span class="sxs-lookup"><span data-stu-id="bc72a-179">This was most apparent when using owned entities.</span></span> <span data-ttu-id="bc72a-180">例如，请考虑以下模型和配置：</span><span class="sxs-lookup"><span data-stu-id="bc72a-180">For example, consider the following model and configuration:</span></span>

```csharp
public class Person
{
    public int Id { get; set; }
    public string Name { get; set; }

    public Address HomeAddress { get; set; }
    public Address WorkAddress { get; set; }
}

public class Address
{
    public string Line1 { get; set; }
    public string Line2 { get; set; }
    public string City { get; set; }
    public string Region { get; set; }
    public string Country { get; set; }
    public string Postcode { get; set; }
}
```

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Person>(b =>
    {
        b.OwnsOne(e => e.HomeAddress,
            b =>
            {
                b.Property(e => e.Line1).IsRequired();
                b.Property(e => e.City).IsRequired();
                b.Property(e => e.Region).IsRequired();
                b.Property(e => e.Postcode).IsRequired();
            });

        b.OwnsOne(e => e.WorkAddress);
    });
}
```

<span data-ttu-id="bc72a-181">这导致迁移为 SQLite 创建以下表：</span><span class="sxs-lookup"><span data-stu-id="bc72a-181">This results in Migrations creating the following table for SQLite:</span></span>

```sql
CREATE TABLE "People" (
    "Id" INTEGER NOT NULL CONSTRAINT "PK_People" PRIMARY KEY AUTOINCREMENT,
    "Name" TEXT NULL,
    "HomeAddress_Line1" TEXT NULL,
    "HomeAddress_Line2" TEXT NULL,
    "HomeAddress_City" TEXT NULL,
    "HomeAddress_Region" TEXT NULL,
    "HomeAddress_Country" TEXT NULL,
    "HomeAddress_Postcode" TEXT NULL,
    "WorkAddress_Line1" TEXT NULL,
    "WorkAddress_Line2" TEXT NULL,
    "WorkAddress_City" TEXT NULL,
    "WorkAddress_Region" TEXT NULL,
    "WorkAddress_Country" TEXT NULL,
    "WorkAddress_Postcode" TEXT NULL
);
```

<span data-ttu-id="bc72a-182">请注意，所有列都可为 null，即使其中某些 `HomeAddress` 属性已按要求配置。</span><span class="sxs-lookup"><span data-stu-id="bc72a-182">Notice that all the columns are nullable, even though some of the `HomeAddress` properties have been configured as required.</span></span> <span data-ttu-id="bc72a-183">此外，在查询 `Person` 时，如果家庭或工作地址的所有列都为 null，则 EF Core 将 `HomeAddress` 和/或 `WorkAddress` 属性保留为 null，而不是设置 `Address` 的空实例。</span><span class="sxs-lookup"><span data-stu-id="bc72a-183">Also, when querying for a `Person`, if all the columns for either the home or work address are null, then EF Core will leave the `HomeAddress` and/or `WorkAddress` properties as null, rather than setting an empty instance of `Address`.</span></span>

<span data-ttu-id="bc72a-184">在 EF Core 5.0 中，`HomeAddress` 导航现在可以配置为必需的依赖项。</span><span class="sxs-lookup"><span data-stu-id="bc72a-184">In EF Core 5.0, the `HomeAddress` navigation can now be configured as as a required dependent.</span></span> <span data-ttu-id="bc72a-185">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-185">For example:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Person>(b =>
    {
        b.OwnsOne(e => e.HomeAddress,
            b =>
            {
                b.Property(e => e.Line1).IsRequired();
                b.Property(e => e.City).IsRequired();
                b.Property(e => e.Region).IsRequired();
                b.Property(e => e.Postcode).IsRequired();
            });
        b.Navigation(e => e.HomeAddress).IsRequired();

        b.OwnsOne(e => e.WorkAddress);
    });
}
```

<span data-ttu-id="bc72a-186">对于必需依赖项的必需属性，由迁移创建的表现在将包含不可为 null 的列：</span><span class="sxs-lookup"><span data-stu-id="bc72a-186">The table created by Migrations will now included non-nullable columns for the required properties of the required dependent:</span></span>

```sql
CREATE TABLE "People" (
    "Id" INTEGER NOT NULL CONSTRAINT "PK_People" PRIMARY KEY AUTOINCREMENT,
    "Name" TEXT NULL,
    "HomeAddress_Line1" TEXT NOT NULL,
    "HomeAddress_Line2" TEXT NULL,
    "HomeAddress_City" TEXT NOT NULL,
    "HomeAddress_Region" TEXT NOT NULL,
    "HomeAddress_Country" TEXT NULL,
    "HomeAddress_Postcode" TEXT NOT NULL,
    "WorkAddress_Line1" TEXT NULL,
    "WorkAddress_Line2" TEXT NULL,
    "WorkAddress_City" TEXT NULL,
    "WorkAddress_Region" TEXT NULL,
    "WorkAddress_Country" TEXT NULL,
    "WorkAddress_Postcode" TEXT NULL
);
```

<span data-ttu-id="bc72a-187">此外，如果尝试保存具有需要 null 的依赖项的所有者，则 EF Core 现在将引发异常。</span><span class="sxs-lookup"><span data-stu-id="bc72a-187">In addition, EF Core will now throw an exception if an attempt is made to save an owner which has a null required dependent.</span></span> <span data-ttu-id="bc72a-188">在此示例中，EF Core 将在尝试使用 null `HomeAddress` 保存 `Person` 时引发。</span><span class="sxs-lookup"><span data-stu-id="bc72a-188">In this example, EF Core will throw when attempting to save a `Person` with a null `HomeAddress`.</span></span>

<span data-ttu-id="bc72a-189">最后，即使所需依赖项的所有列都具有 null 值，EF Core 仍将创建所需依赖项的实例。</span><span class="sxs-lookup"><span data-stu-id="bc72a-189">Finally, EF Core will still create an instance of a required dependent even when all the columns for the required dependent have null values.</span></span>

### <a name="options-for-migration-generation"></a><span data-ttu-id="bc72a-190">用于迁移生成的选项</span><span class="sxs-lookup"><span data-stu-id="bc72a-190">Options for migration generation</span></span>

<span data-ttu-id="bc72a-191">EF Core 5.0 引入了对出于不同目的的迁移生成的更大控制。</span><span class="sxs-lookup"><span data-stu-id="bc72a-191">EF Core 5.0 introduces greater control over generation of migrations for different purposes.</span></span> <span data-ttu-id="bc72a-192">包括以下功能：</span><span class="sxs-lookup"><span data-stu-id="bc72a-192">This includes the ability to:</span></span>

* <span data-ttu-id="bc72a-193">了解是为脚本生成迁移还是立即执行</span><span class="sxs-lookup"><span data-stu-id="bc72a-193">Know if the migration is being generated for a script or for immediate execution</span></span>
* <span data-ttu-id="bc72a-194">了解是否生成了幂等脚本</span><span class="sxs-lookup"><span data-stu-id="bc72a-194">Know if an idempotent script is being generated</span></span>
* <span data-ttu-id="bc72a-195">了解脚本是否应该排除事务语句（请参阅下面的“使用事务迁移脚本”。）</span><span class="sxs-lookup"><span data-stu-id="bc72a-195">Know if the script should exclude transaction statements (See _Migrations scripts with transactions_ below.)</span></span>

<span data-ttu-id="bc72a-196">此行为由 `MigrationsSqlGenerationOptions` 枚举指定，此枚举现可传递到 `IMigrator.GenerateScript`。</span><span class="sxs-lookup"><span data-stu-id="bc72a-196">This behavior is specified by an the `MigrationsSqlGenerationOptions` enum, which can now be passed to `IMigrator.GenerateScript`.</span></span>

<span data-ttu-id="bc72a-197">这项工作中还包括更好地生成幂等脚本，如果需要，该脚本会在 SQL Server 上调用 `EXEC`。</span><span class="sxs-lookup"><span data-stu-id="bc72a-197">Also included in this work is better generation of idempotent scripts with calls to `EXEC` on SQL Server when needed.</span></span> <span data-ttu-id="bc72a-198">此工作还支持对其他数据库提供程序（包括 PostgreSQL）生成的脚本进行类似的改进。</span><span class="sxs-lookup"><span data-stu-id="bc72a-198">This work also enables similar improvements to the scripts generated by other database providers, including PostgreSQL.</span></span>

### <a name="migrations-scripts-with-transactions"></a><span data-ttu-id="bc72a-199">使用事务迁移脚本</span><span class="sxs-lookup"><span data-stu-id="bc72a-199">Migrations scripts with transactions</span></span>

<span data-ttu-id="bc72a-200">从迁移生成的 SQL 脚本现在包含要开始和提交适合迁移的事务的语句。</span><span class="sxs-lookup"><span data-stu-id="bc72a-200">SQL scripts generated from migrations now contain statements to begin and commit transactions as appropriate for the migration.</span></span> <span data-ttu-id="bc72a-201">例如，下面的迁移脚本是两次迁移生成的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-201">For example, the migration script below was generated from two migrations.</span></span> <span data-ttu-id="bc72a-202">请注意，每次迁移现在都在事务中应用。</span><span class="sxs-lookup"><span data-stu-id="bc72a-202">Notice that each migration is now applied inside a transaction.</span></span>

```sql
BEGIN TRANSACTION;
GO

CREATE TABLE [Groups] (
    [Id] int NOT NULL IDENTITY,
    [Name] nvarchar(max) NULL,
    CONSTRAINT [PK_Groups] PRIMARY KEY ([Id])
);
GO

CREATE TABLE [Members] (
    [Id] int NOT NULL IDENTITY,
    [Name] nvarchar(max) NULL,
    [GroupId] int NULL,
    CONSTRAINT [PK_Members] PRIMARY KEY ([Id]),
    CONSTRAINT [FK_Members_Groups_GroupId] FOREIGN KEY ([GroupId]) REFERENCES [Groups] ([Id]) ON DELETE NO ACTION
);
GO

CREATE INDEX [IX_Members_GroupId] ON [Members] ([GroupId]);
GO

INSERT INTO [__EFMigrationsHistory] ([MigrationId], [ProductVersion])
VALUES (N'20200910194835_One', N'6.0.0-alpha.1.20460.2');
GO

COMMIT;
GO

BEGIN TRANSACTION;
GO

EXEC sp_rename N'[Groups].[Name]', N'GroupName', N'COLUMN';
GO

INSERT INTO [__EFMigrationsHistory] ([MigrationId], [ProductVersion])
VALUES (N'20200910195234_Two', N'6.0.0-alpha.1.20460.2');
GO

COMMIT;
```

<span data-ttu-id="bc72a-203">如前一部分所述，如果需要以不同的方式处理事务，可以禁用对事务的这种使用。</span><span class="sxs-lookup"><span data-stu-id="bc72a-203">As mentioned in the previous section, this use of transactions can be disabled if transactions need to be handled differently.</span></span>

### <a name="see-pending-migrations"></a><span data-ttu-id="bc72a-204">查看挂起的迁移</span><span class="sxs-lookup"><span data-stu-id="bc72a-204">See pending migrations</span></span>

<span data-ttu-id="bc72a-205">此功能是 [@Psypher9](https://github.com/Psypher9) 在社区中贡献的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-205">This feature was contributed from the community by [@Psypher9](https://github.com/Psypher9).</span></span> <span data-ttu-id="bc72a-206">非常感谢贡献此功能！</span><span class="sxs-lookup"><span data-stu-id="bc72a-206">Many thanks for the contribution!</span></span>

<span data-ttu-id="bc72a-207">`dotnet ef migrations list` 命令现在显示哪些迁移尚未应用于数据库。</span><span class="sxs-lookup"><span data-stu-id="bc72a-207">The `dotnet ef migrations list` command now shows which migrations have not yet been applied to the database.</span></span> <span data-ttu-id="bc72a-208">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-208">For example:</span></span>

```console
ajcvickers@avickers420u:~/AllTogetherNow/Daily$ dotnet ef migrations list
Build started...
Build succeeded.
20200910201647_One
20200910201708_Two
20200910202050_Three (Pending)
ajcvickers@avickers420u:~/AllTogetherNow/Daily$
```

<span data-ttu-id="bc72a-209">此外，现在具有相同功能的包管理器控制台具有 `Get-Migration` 命令。</span><span class="sxs-lookup"><span data-stu-id="bc72a-209">In addition, there is now a `Get-Migration` command for the Package Manager Console with the same functionality.</span></span>

### <a name="modelbuilder-api-for-value-comparers"></a><span data-ttu-id="bc72a-210">值比较器的 ModelBuilder API</span><span class="sxs-lookup"><span data-stu-id="bc72a-210">ModelBuilder API for value comparers</span></span>

<span data-ttu-id="bc72a-211">自定义可变类型的 EF Core 属性[需要值比较器](xref:core/modeling/value-comparers)以便正确检测属性更改。</span><span class="sxs-lookup"><span data-stu-id="bc72a-211">EF Core properties for custom mutable types [require a value comparer](xref:core/modeling/value-comparers) for property changes to be detected correctly.</span></span> <span data-ttu-id="bc72a-212">现在可以将此指定为配置类型的值转换的一部分。</span><span class="sxs-lookup"><span data-stu-id="bc72a-212">This can now be specified as part of configuring the value conversion for the type.</span></span> <span data-ttu-id="bc72a-213">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-213">For example:</span></span>

```csharp
modelBuilder
    .Entity<EntityType>()
    .Property(e => e.MyProperty)
    .HasConversion(
        v => JsonSerializer.Serialize(v, null),
        v => JsonSerializer.Deserialize<List<int>>(v, null),
        new ValueComparer<List<int>>(
            (c1, c2) => c1.SequenceEqual(c2),
            c => c.Aggregate(0, (a, v) => HashCode.Combine(a, v.GetHashCode())),
            c => c.ToList()));
```

### <a name="entityentry-trygetvalue-methods"></a><span data-ttu-id="bc72a-214">EntityEntry TryGetValue 方法</span><span class="sxs-lookup"><span data-stu-id="bc72a-214">EntityEntry TryGetValue methods</span></span>

<span data-ttu-id="bc72a-215">此功能是 [@m4ss1m0g](https://github.com/m4ss1m0g) 在社区中贡献的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-215">This feature was contributed from the community by [@m4ss1m0g](https://github.com/m4ss1m0g).</span></span> <span data-ttu-id="bc72a-216">非常感谢贡献此功能！</span><span class="sxs-lookup"><span data-stu-id="bc72a-216">Many thanks for the contribution!</span></span>

<span data-ttu-id="bc72a-217">`TryGetValue` 方法已添加到 `EntityEntry.CurrentValues` 和 `EntityEntry.OriginalValues`。</span><span class="sxs-lookup"><span data-stu-id="bc72a-217">A `TryGetValue` method has been added to `EntityEntry.CurrentValues` and `EntityEntry.OriginalValues`.</span></span> <span data-ttu-id="bc72a-218">这允许请求属性的值，而无需首先检查该属性是否映射在 EF 模型中。</span><span class="sxs-lookup"><span data-stu-id="bc72a-218">This allows the value of a property to be requested without first checking if the property is mapped in the EF model.</span></span> <span data-ttu-id="bc72a-219">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-219">For example:</span></span>

```csharp
if (entry.CurrentValues.TryGetValue(propertyName, out var value))
{
    Console.WriteLine(value);
}
```

### <a name="default-max-batch-size-for-sql-server"></a><span data-ttu-id="bc72a-220">SQL Server 的默认最大批处理大小</span><span class="sxs-lookup"><span data-stu-id="bc72a-220">Default max batch size for SQL Server</span></span>

<span data-ttu-id="bc72a-221">从 EF Core 5.0 开始，SQL Server 上的 SaveChanges 的默认最大批处理大小现在是 42。</span><span class="sxs-lookup"><span data-stu-id="bc72a-221">Starting with EF Core 5.0, the default maximum batch size for SaveChanges on SQL Server is now 42.</span></span> <span data-ttu-id="bc72a-222">众所周知，这也是对生命、宇宙和一切终极问题的答案。</span><span class="sxs-lookup"><span data-stu-id="bc72a-222">As is well known, this is also the answer to the Ultimate Question of Life, the Universe, and Everything.</span></span> <span data-ttu-id="bc72a-223">但是，这可能是巧合，因为该值是通过[批处理性能的分析](https://github.com/dotnet/efcore/issues/9270)获得的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-223">However, this is probably a coincidence, since the value was obtained through [analysis of batching performance](https://github.com/dotnet/efcore/issues/9270).</span></span> <span data-ttu-id="bc72a-224">我们不相信我们已经发现了终极问题的形式，尽管地球被创造出来是为了理解 SQL Server 为什么以它的方式工作似乎也有些道理。</span><span class="sxs-lookup"><span data-stu-id="bc72a-224">We do not believe that we have discovered a form of the Ultimate Question, although it does seem somewhat plausible that the Earth was created to understand why SQL Server works the way it does.</span></span>

### <a name="default-environment-to-development"></a><span data-ttu-id="bc72a-225">用于开发的默认环境</span><span class="sxs-lookup"><span data-stu-id="bc72a-225">Default environment to Development</span></span>

<span data-ttu-id="bc72a-226">EF Core 命令行工具现在会自动将 `ASPNETCORE_ENVIRONMENT` 和`DOTNET_ENVIRONMENT` 环境变量配置为“开发”。</span><span class="sxs-lookup"><span data-stu-id="bc72a-226">The EF Core command line tools now automatically configure the `ASPNETCORE_ENVIRONMENT` _and_ `DOTNET_ENVIRONMENT` environment variables to "Development".</span></span> <span data-ttu-id="bc72a-227">这带来了在开发过程中使用通用主机时与 ASP.NET Core 体验一样。</span><span class="sxs-lookup"><span data-stu-id="bc72a-227">This brings the experience when using the generic host in line with the experience for ASP.NET Core during development.</span></span> <span data-ttu-id="bc72a-228">查看 [#19903](https://github.com/dotnet/efcore/issues/19903)。</span><span class="sxs-lookup"><span data-stu-id="bc72a-228">See [#19903](https://github.com/dotnet/efcore/issues/19903).</span></span>

### <a name="better-migrations-column-ordering"></a><span data-ttu-id="bc72a-229">更好的迁移列排序</span><span class="sxs-lookup"><span data-stu-id="bc72a-229">Better migrations column ordering</span></span>

<span data-ttu-id="bc72a-230">未映射基类的列现在按映射实体类型的其他列排序。</span><span class="sxs-lookup"><span data-stu-id="bc72a-230">The columns for unmapped base classes are now ordered after other columns for mapped entity types.</span></span> <span data-ttu-id="bc72a-231">请注意，这仅会影响新创建的表。</span><span class="sxs-lookup"><span data-stu-id="bc72a-231">Note this only impacts newly created tables.</span></span> <span data-ttu-id="bc72a-232">现有表的列顺序保持不变。</span><span class="sxs-lookup"><span data-stu-id="bc72a-232">The column order for existing tables remains unchanged.</span></span> <span data-ttu-id="bc72a-233">查看 [#11314](https://github.com/dotnet/efcore/issues/11314)。</span><span class="sxs-lookup"><span data-stu-id="bc72a-233">See [#11314](https://github.com/dotnet/efcore/issues/11314).</span></span>

### <a name="query-improvements"></a><span data-ttu-id="bc72a-234">查询改进</span><span class="sxs-lookup"><span data-stu-id="bc72a-234">Query improvements</span></span>

<span data-ttu-id="bc72a-235">EF Core 5.0 RC1 包含一些其他查询转换改进：</span><span class="sxs-lookup"><span data-stu-id="bc72a-235">EF Core 5.0 RC1 contains some additional query translation improvements:</span></span>

* <span data-ttu-id="bc72a-236">Cosmos 上的 `is` 转换 - 请参阅 [#16391](https://github.com/dotnet/efcore/issues/16391)</span><span class="sxs-lookup"><span data-stu-id="bc72a-236">Translation of `is` on Cosmos--see [#16391](https://github.com/dotnet/efcore/issues/16391)</span></span>
* <span data-ttu-id="bc72a-237">现在可以注释用户映射的函数以控制 null 传播 - 请参阅 [#19609](https://github.com/dotnet/efcore/issues/19609)</span><span class="sxs-lookup"><span data-stu-id="bc72a-237">User-mapped functions can now be annotated to control null propagation--see [#19609](https://github.com/dotnet/efcore/issues/19609)</span></span>
* <span data-ttu-id="bc72a-238">支持使用条件聚合翻译 GroupBy - 请参阅 [#11711](https://github.com/dotnet/efcore/issues/11711)</span><span class="sxs-lookup"><span data-stu-id="bc72a-238">Support for translation of GroupBy with conditional aggregates--see [#11711](https://github.com/dotnet/efcore/issues/11711)</span></span>
* <span data-ttu-id="bc72a-239">聚合之前对组元素的 Distinct 运算符转换 - 请参阅 [#17376](https://github.com/dotnet/efcore/issues/17376)</span><span class="sxs-lookup"><span data-stu-id="bc72a-239">Translation of Distinct operator over group element before aggregate--see [#17376](https://github.com/dotnet/efcore/issues/17376)</span></span>

### <a name="model-building-for-fields"></a><span data-ttu-id="bc72a-240">字段的模型生成</span><span class="sxs-lookup"><span data-stu-id="bc72a-240">Model building for fields</span></span>

<span data-ttu-id="bc72a-241">最后，对于 RC1，EF Core 现在允许在 ModelBuilder 中对字段和属性使用 lambda 方法。</span><span class="sxs-lookup"><span data-stu-id="bc72a-241">Finally for RC1, EF Core now allows use of the lambda methods in the ModelBuilder for fields as well as properties.</span></span> <span data-ttu-id="bc72a-242">例如，如果你由于某种原因而反对属性并决定使用公共字段，那么现在可以使用 lambda 生成器映射这些字段：</span><span class="sxs-lookup"><span data-stu-id="bc72a-242">For example, if you are averse to properties for some reason and decide to use public fields, then these fields can now be mapped using the lambda builders:</span></span>

```csharp
public class Post
{
    public int Id;
    public string Name;
    public string Category;
    public int BlogId;
    public Blog Blog;
}

public class Blog
{
    public int Id;
    public string Name;
    public ICollection<Post> Posts;
}
```

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>(b =>
    {
        b.Property(e => e.Id);
        b.Property(e => e.Name);
    });

    modelBuilder.Entity<Post>(b =>
    {
        b.Property(e => e.Id);
        b.Property(e => e.Name);
        b.Property(e => e.Category);
        b.Property(e => e.BlogId);
        b.HasOne(e => e.Blog).WithMany(e => e.Posts);
    });
}
```

<span data-ttu-id="bc72a-243">虽然现在可以这样做，但我们强烈建议不要这样做。</span><span class="sxs-lookup"><span data-stu-id="bc72a-243">While this is now possible, we are certainly not recommending that you do this.</span></span> <span data-ttu-id="bc72a-244">此外，请注意，这不会向 EF Core 添加任何其他字段映射功能，它只允许使用 lambda 方法，而不是始终要求字符串方法。</span><span class="sxs-lookup"><span data-stu-id="bc72a-244">Also, note that this does not add any additional field mapping capabilities to EF Core, it only allows the lambda methods to be used instead of always requiring the string methods.</span></span> <span data-ttu-id="bc72a-245">这作用不大，因为字段很少是公共的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-245">This is seldom useful since fields are rarely public.</span></span>

## <a name="preview-8"></a><span data-ttu-id="bc72a-246">预览版 8</span><span class="sxs-lookup"><span data-stu-id="bc72a-246">Preview 8</span></span>

### <a name="table-per-type-tpt-mapping"></a><span data-ttu-id="bc72a-247">每个类型一张表 (TPT) 映射</span><span class="sxs-lookup"><span data-stu-id="bc72a-247">Table-per-type (TPT) mapping</span></span>

<span data-ttu-id="bc72a-248">默认情况下，EF Core 会将 .NET 类型的继承层次结构映射到单个数据库表。</span><span class="sxs-lookup"><span data-stu-id="bc72a-248">By default, EF Core maps an inheritance hierarchy of .NET types to a single database table.</span></span> <span data-ttu-id="bc72a-249">这称为每个层次结构一张表 (TPH) 映射。</span><span class="sxs-lookup"><span data-stu-id="bc72a-249">This is known as table-per-hierarchy (TPH) mapping.</span></span> <span data-ttu-id="bc72a-250">EF Core 5.0 还允许将继承层次结构中的每个 .NET 类型映射到另一个数据库表，这称为每个类型一张表 (TPT) 映射。</span><span class="sxs-lookup"><span data-stu-id="bc72a-250">EF Core 5.0 also allows mapping each .NET type in an inheritance hierarchy to a different database table; known as table-per-type (TPT) mapping.</span></span>

<span data-ttu-id="bc72a-251">例如，请考虑具有映射的层次结构的此模型：</span><span class="sxs-lookup"><span data-stu-id="bc72a-251">For example, consider this model with a mapped hierarchy:</span></span>

```csharp
public class Animal
{
    public int Id { get; set; }
    public string Species { get; set; }
}

public class Pet : Animal
{
    public string Name { get; set; }
}

public class Cat : Pet
{
    public string EducationLevel { get; set; }
}

public class Dog : Pet
{
    public string FavoriteToy { get; set; }
}
```

<span data-ttu-id="bc72a-252">默认情况下，EF Core 会将此模型映射到单个表：</span><span class="sxs-lookup"><span data-stu-id="bc72a-252">By default, EF Core will map this to a single table:</span></span>

```sql
CREATE TABLE [Animals] (
    [Id] int NOT NULL IDENTITY,
    [Species] nvarchar(max) NULL,
    [Discriminator] nvarchar(max) NOT NULL,
    [Name] nvarchar(max) NULL,
    [EdcuationLevel] nvarchar(max) NULL,
    [FavoriteToy] nvarchar(max) NULL,
    CONSTRAINT [PK_Animals] PRIMARY KEY ([Id])
);
```

<span data-ttu-id="bc72a-253">但是，将每个实体类型映射到不同的表，会得到每个类型一张表的结果：</span><span class="sxs-lookup"><span data-stu-id="bc72a-253">However, mapping each entity type to a different table will instead result in one table per type:</span></span>

```sql
CREATE TABLE [Animals] (
    [Id] int NOT NULL IDENTITY,
    [Species] nvarchar(max) NULL,
    CONSTRAINT [PK_Animals] PRIMARY KEY ([Id])
);

CREATE TABLE [Pets] (
    [Id] int NOT NULL,
    [Name] nvarchar(max) NULL,
    CONSTRAINT [PK_Pets] PRIMARY KEY ([Id]),
    CONSTRAINT [FK_Pets_Animals_Id] FOREIGN KEY ([Id]) REFERENCES [Animals] ([Id]) ON DELETE NO ACTION
);

CREATE TABLE [Cats] (
    [Id] int NOT NULL,
    [EdcuationLevel] nvarchar(max) NULL,
    CONSTRAINT [PK_Cats] PRIMARY KEY ([Id]),
    CONSTRAINT [FK_Cats_Animals_Id] FOREIGN KEY ([Id]) REFERENCES [Animals] ([Id]) ON DELETE NO ACTION,
    CONSTRAINT [FK_Cats_Pets_Id] FOREIGN KEY ([Id]) REFERENCES [Pets] ([Id]) ON DELETE NO ACTION
);

CREATE TABLE [Dogs] (
    [Id] int NOT NULL,
    [FavoriteToy] nvarchar(max) NULL,
    CONSTRAINT [PK_Dogs] PRIMARY KEY ([Id]),
    CONSTRAINT [FK_Dogs_Animals_Id] FOREIGN KEY ([Id]) REFERENCES [Animals] ([Id]) ON DELETE NO ACTION,
    CONSTRAINT [FK_Dogs_Pets_Id] FOREIGN KEY ([Id]) REFERENCES [Pets] ([Id]) ON DELETE NO ACTION
);
```

<span data-ttu-id="bc72a-254">请注意，上述外键约束的创建是在对预览版 8 的代码创建分支后添加的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-254">Note that creation of the foreign key constraints shown above were added after branching the code for preview 8.</span></span>

<span data-ttu-id="bc72a-255">可使用映射特性将实体类型映射到不同的表：</span><span class="sxs-lookup"><span data-stu-id="bc72a-255">Entity types can be mapped to different tables using mapping attributes:</span></span>

```csharp
[Table("Animals")]
public class Animal
{
    public int Id { get; set; }
    public string Species { get; set; }
}

[Table("Pets")]
public class Pet : Animal
{
    public string Name { get; set; }
}

[Table("Cats")]
public class Cat : Pet
{
    public string EdcuationLevel { get; set; }
}

[Table("Dogs")]
public class Dog : Pet
{
    public string FavoriteToy { get; set; }
}
```

<span data-ttu-id="bc72a-256">或使用 `ModelBuilder` 配置：</span><span class="sxs-lookup"><span data-stu-id="bc72a-256">Or using `ModelBuilder` configuration:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Animal>().ToTable("Animals");
    modelBuilder.Entity<Pet>().ToTable("Pets");
    modelBuilder.Entity<Cat>().ToTable("Cats");
    modelBuilder.Entity<Dog>().ToTable("Dogs");
}
```

<span data-ttu-id="bc72a-257">记录信息可通过问题 [#1979](https://github.com/dotnet/EntityFramework.Docs/issues/1979) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-257">Documentation is tracked by issue [#1979](https://github.com/dotnet/EntityFramework.Docs/issues/1979).</span></span>

### <a name="migrations-rebuild-sqlite-tables"></a><span data-ttu-id="bc72a-258">迁移：重新生成 SQLite 表</span><span class="sxs-lookup"><span data-stu-id="bc72a-258">Migrations: Rebuild SQLite tables</span></span>

<span data-ttu-id="bc72a-259">与其他数据库相比，SQLite 的架构操作功能相对有限。</span><span class="sxs-lookup"><span data-stu-id="bc72a-259">Compared to other database, SQLite is relatively limited in its schema manipulation capabilities.</span></span> <span data-ttu-id="bc72a-260">例如，从现有表中删除列需要删除并重新创建整个表。</span><span class="sxs-lookup"><span data-stu-id="bc72a-260">For example, dropping a column from an existing table requires that the entire table be dropped and re-created.</span></span> <span data-ttu-id="bc72a-261">EF Core 5.0 迁移现在支持自动重新生成表，以便实现所需的架构更改。</span><span class="sxs-lookup"><span data-stu-id="bc72a-261">EF Core 5.0 Migrations now supports automatic rebuilding the table for schema changes that require it.</span></span>

<span data-ttu-id="bc72a-262">例如，假设为 `Unicorn` 实体类型创建了 `Unicorns` 表：</span><span class="sxs-lookup"><span data-stu-id="bc72a-262">For example, imagine we have a `Unicorns` table created for a `Unicorn` entity type:</span></span>

```csharp
public class Unicorn
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
}
```

```sql
CREATE TABLE "Unicorns" (
    "Id" INTEGER NOT NULL CONSTRAINT "PK_Unicorns" PRIMARY KEY AUTOINCREMENT,
    "Name" TEXT NULL,
    "Age" INTEGER NOT NULL
);
```

<span data-ttu-id="bc72a-263">然后我们了解到，存储独角兽的年龄被认为非常不礼貌，因此让我们删除该属性，添加新的迁移，并更新数据库。</span><span class="sxs-lookup"><span data-stu-id="bc72a-263">We then learn that storing the age of a unicorn is considered very rude, so let's remove that property, add a new migration, and update the database.</span></span> <span data-ttu-id="bc72a-264">使用 EF Core 3.1 时，此更新将失败，因为无法删除该列。</span><span class="sxs-lookup"><span data-stu-id="bc72a-264">This update will fail when using EF Core 3.1 because the column cannot be dropped.</span></span> <span data-ttu-id="bc72a-265">在 EF Core 5.0 中，迁移将改为重新生成表：</span><span class="sxs-lookup"><span data-stu-id="bc72a-265">In EF Core 5.0, Migrations will instead rebuild the table:</span></span>

```sql
CREATE TABLE "ef_temp_Unicorns" (
    "Id" INTEGER NOT NULL CONSTRAINT "PK_Unicorns" PRIMARY KEY AUTOINCREMENT,
    "Name" TEXT NULL
);

INSERT INTO "ef_temp_Unicorns" ("Id", "Name")
SELECT "Id", "Name"
FROM Unicorns;

PRAGMA foreign_keys = 0;

DROP TABLE "Unicorns";

ALTER TABLE "ef_temp_Unicorns" RENAME TO "Unicorns";

PRAGMA foreign_keys = 1;
```

<span data-ttu-id="bc72a-266">请注意：</span><span class="sxs-lookup"><span data-stu-id="bc72a-266">Notice that:</span></span>

* <span data-ttu-id="bc72a-267">创建一个临时表，其中包含新表所需的架构</span><span class="sxs-lookup"><span data-stu-id="bc72a-267">A temporary table is created with the desired schema for the new table</span></span>
* <span data-ttu-id="bc72a-268">将数据从当前表复制到临时表中</span><span class="sxs-lookup"><span data-stu-id="bc72a-268">Data is copied from the current table into the temporary table</span></span>
* <span data-ttu-id="bc72a-269">关闭外键强制执行</span><span class="sxs-lookup"><span data-stu-id="bc72a-269">Foreign key enforcement is switched off</span></span>
* <span data-ttu-id="bc72a-270">删除当前表</span><span class="sxs-lookup"><span data-stu-id="bc72a-270">The current table is dropped</span></span>
* <span data-ttu-id="bc72a-271">将临时表重命名为新表</span><span class="sxs-lookup"><span data-stu-id="bc72a-271">The temporary table is renamed to be the new table</span></span>

<span data-ttu-id="bc72a-272">记录信息可通过问题 [#2583](https://github.com/dotnet/EntityFramework.Docs/issues/2583) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-272">Documentation is tracked by issue [#2583](https://github.com/dotnet/EntityFramework.Docs/issues/2583).</span></span>

### <a name="table-valued-functions"></a><span data-ttu-id="bc72a-273">表值函数</span><span class="sxs-lookup"><span data-stu-id="bc72a-273">Table-valued functions</span></span>

<span data-ttu-id="bc72a-274">此功能是 [@pmiddleton](https://github.com/pmiddleton) 在社区中贡献的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-274">This feature was contributed from the community by [@pmiddleton](https://github.com/pmiddleton).</span></span> <span data-ttu-id="bc72a-275">非常感谢贡献此功能！</span><span class="sxs-lookup"><span data-stu-id="bc72a-275">Many thanks for the contribution!</span></span>

<span data-ttu-id="bc72a-276">EF Core 5.0 为 .NET 方法到表值函数 (TVF) 的映射提供一流支持。</span><span class="sxs-lookup"><span data-stu-id="bc72a-276">EF Core 5.0 includes first-class support for mapping .NET methods to table-valued functions (TVFs).</span></span> <span data-ttu-id="bc72a-277">然后可在 LINQ 查询中使用这些函数，在此类查询中，函数结果上的其他组合也将转换为 SQL。</span><span class="sxs-lookup"><span data-stu-id="bc72a-277">These functions can then be used in LINQ queries where additional composition on the results of the function will also be translated to SQL.</span></span>

<span data-ttu-id="bc72a-278">例如，请考虑在 SQL Server 数据库中定义的以下 TVF：</span><span class="sxs-lookup"><span data-stu-id="bc72a-278">For example, consider this TVF defined in a SQL Server database:</span></span>

```sql
CREATE FUNCTION GetReports(@employeeId int)
RETURNS @reports TABLE
(
    Name nvarchar(50) NOT NULL,
    IsDeveloper bit NOT NULL
)
AS
BEGIN
    WITH cteEmployees AS
    (
        SELECT Id, Name, ManagerId, IsDeveloper
        FROM Employees
        WHERE Id = @employeeId
        UNION ALL
        SELECT e.Id, e.Name, e.ManagerId, e.IsDeveloper
        FROM Employees e
        INNER JOIN cteEmployees cteEmp ON cteEmp.Id = e.ManagerId
    )
    INSERT INTO @reports
    SELECT Name, IsDeveloper
    FROM cteEmployees
    WHERE Id != @employeeId

    RETURN
END
```

<span data-ttu-id="bc72a-279">EF Core 模型需要两种实体类型才能使用此 TVF：</span><span class="sxs-lookup"><span data-stu-id="bc72a-279">The EF Core model requires two entity types to use this TVF:</span></span>

* <span data-ttu-id="bc72a-280">以正常方式映射到 Employees 表的 `Employee` 类型</span><span class="sxs-lookup"><span data-stu-id="bc72a-280">An `Employee` type that maps to the Employees table in the normal way</span></span>
* <span data-ttu-id="bc72a-281">与 TVF 返回的形状相匹配的 `Report` 类型</span><span class="sxs-lookup"><span data-stu-id="bc72a-281">A `Report` type that matches the shape returned by the TVF</span></span>

```csharp
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public bool IsDeveloper { get; set; }

    public int? ManagerId { get; set; }
    public virtual Employee Manager { get; set; }
}
```

```csharp
public class Report
{
    public string Name { get; set; }
    public bool IsDeveloper { get; set; }
}
```

<span data-ttu-id="bc72a-282">这些类型必须包含在 EF Core 模型中：</span><span class="sxs-lookup"><span data-stu-id="bc72a-282">These types must be included in the EF Core model:</span></span>

```csharp
modelBuilder.Entity<Employee>();
modelBuilder.Entity(typeof(Report)).HasNoKey();
```

<span data-ttu-id="bc72a-283">请注意，`Report` 没有主键，因此必须这样配置。</span><span class="sxs-lookup"><span data-stu-id="bc72a-283">Notice that `Report` has no primary key and so must be configured as such.</span></span>

<span data-ttu-id="bc72a-284">最后，必须将 .NET 方法映射到数据库中的 TVF。</span><span class="sxs-lookup"><span data-stu-id="bc72a-284">Finally, a .NET method must be mapped to the TVF in the database.</span></span> <span data-ttu-id="bc72a-285">可以使用新的 `FromExpression` 方法在 DbContext 上定义此方法：</span><span class="sxs-lookup"><span data-stu-id="bc72a-285">This method can be defined on the DbContext using the new `FromExpression` method:</span></span>

```csharp
public IQueryable<Report> GetReports(int managerId)
    => FromExpression(() => GetReports(managerId));
```

<span data-ttu-id="bc72a-286">此方法使用与上面定义的 TVF 匹配的参数和返回类型。</span><span class="sxs-lookup"><span data-stu-id="bc72a-286">This method uses a parameter and return type that match the TVF defined above.</span></span> <span data-ttu-id="bc72a-287">然后在 OnModelCreating 中将该方法添加到 EF Core 模型：</span><span class="sxs-lookup"><span data-stu-id="bc72a-287">The method is then added to the EF Core model in OnModelCreating:</span></span>

```csharp
modelBuilder.HasDbFunction(() => GetReports(default));
```

<span data-ttu-id="bc72a-288">（在此处使用 lambda 可轻松将 `MethodInfo` 传递到 EF Core。</span><span class="sxs-lookup"><span data-stu-id="bc72a-288">(Using a lambda here is an easy way to pass the `MethodInfo` to EF Core.</span></span> <span data-ttu-id="bc72a-289">将忽略传递给该方法的参数。）</span><span class="sxs-lookup"><span data-stu-id="bc72a-289">The arguments passed to the method are ignored.)</span></span>

<span data-ttu-id="bc72a-290">现在可以编写查询，以调用 `GetReports` 并对结果进行组合。</span><span class="sxs-lookup"><span data-stu-id="bc72a-290">We can now write queries that call `GetReports` and compose over the results.</span></span> <span data-ttu-id="bc72a-291">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-291">For example:</span></span>

```csharp
from e in context.Employees
from rc in context.GetReports(e.Id)
where rc.IsDeveloper == true
select new
{
  ManagerName = e.Name,
  EmployeeName = rc.Name,
})
```

<span data-ttu-id="bc72a-292">在 SQL Server 上，这将转换为：</span><span class="sxs-lookup"><span data-stu-id="bc72a-292">On SQL Server, this translates to:</span></span>

```sql
SELECT [e].[Name] AS [ManagerName], [g].[Name] AS [EmployeeName]
FROM [Employees] AS [e]
CROSS APPLY [dbo].[GetReports]([e].[Id]) AS [g]
WHERE [g].[IsDeveloper] = CAST(1 AS bit)
```

<span data-ttu-id="bc72a-293">请注意，SQL 以 `Employees` 表为根，调用 `GetReports`，然后在函数的结果上添加一个额外的 WHERE 子句。</span><span class="sxs-lookup"><span data-stu-id="bc72a-293">Notice that the SQL is rooted in the `Employees` table, calls `GetReports`, and then adds an additional WHERE clause on the results of the function.</span></span>

### <a name="flexible-queryupdate-mapping"></a><span data-ttu-id="bc72a-294">灵活查询/更新映射</span><span class="sxs-lookup"><span data-stu-id="bc72a-294">Flexible query/update mapping</span></span>

<span data-ttu-id="bc72a-295">EF Core 5.0 允许将同一实体类型映射到不同的数据库对象。</span><span class="sxs-lookup"><span data-stu-id="bc72a-295">EF Core 5.0 allows mapping the same entity type to different database objects.</span></span> <span data-ttu-id="bc72a-296">这些对象可以是表、视图或函数。</span><span class="sxs-lookup"><span data-stu-id="bc72a-296">These objects may be tables, views, or functions.</span></span>

<span data-ttu-id="bc72a-297">例如，可将一个实体类型同时映射到数据库视图和数据库表：</span><span class="sxs-lookup"><span data-stu-id="bc72a-297">For example, an entity type can be mapped to both a database view and a database table:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<Blog>()
        .ToTable("Blogs")
        .ToView("BlogsView");
}
```

<span data-ttu-id="bc72a-298">默认情况下，EF Core 随后将从视图进行查询，将更新发送到表。</span><span class="sxs-lookup"><span data-stu-id="bc72a-298">By default, EF Core will then query from the view and send updates to the table.</span></span> <span data-ttu-id="bc72a-299">例如，执行以下代码：</span><span class="sxs-lookup"><span data-stu-id="bc72a-299">For example, executing the following code:</span></span>

```csharp
var blog = context.Set<Blog>().Single(e => e.Name == "One Unicorn");

blog.Name = "1unicorn2";

context.SaveChanges();
```

<span data-ttu-id="bc72a-300">生成针对视图的查询，然后对表进行更新：</span><span class="sxs-lookup"><span data-stu-id="bc72a-300">Results in a query against the view, and then an update to the table:</span></span>

```sql
SELECT TOP(2) [b].[Id], [b].[Name], [b].[Url]
FROM [BlogsView] AS [b]
WHERE [b].[Name] = N'One Unicorn'

SET NOCOUNT ON;
UPDATE [Blogs] SET [Name] = @p0
WHERE [Id] = @p1;
SELECT @@ROWCOUNT;
```

### <a name="context-wide-split-query-configuration"></a><span data-ttu-id="bc72a-301">上下文范围的拆分查询配置</span><span class="sxs-lookup"><span data-stu-id="bc72a-301">Context-wide split-query configuration</span></span>

<span data-ttu-id="bc72a-302">现在可以将拆分查询（见下文）配置为 DbContext 执行的任何查询的默认值。</span><span class="sxs-lookup"><span data-stu-id="bc72a-302">Split queries (see below) can now be configured as the default for any query executed by the DbContext.</span></span> <span data-ttu-id="bc72a-303">此配置仅适用于关系提供程序，因此必须将其指定为 `UseProvider` 配置的一部分。</span><span class="sxs-lookup"><span data-stu-id="bc72a-303">This configuration is only available for relational providers, and so must be specified as part of the `UseProvider` configuration.</span></span> <span data-ttu-id="bc72a-304">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-304">For example:</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .UseSqlServer(
            Your.SqlServerConnectionString,
            b => b.UseQuerySplittingBehavior(QuerySplittingBehavior.SplitQuery));
```

<span data-ttu-id="bc72a-305">记录信息可通过问题 [#2407](https://github.com/dotnet/EntityFramework.Docs/issues/2407) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-305">Documentation is tracked by issue [#2407](https://github.com/dotnet/EntityFramework.Docs/issues/2407).</span></span>

### <a name="physicaladdress-mapping"></a><span data-ttu-id="bc72a-306">PhysicalAddress 映射</span><span class="sxs-lookup"><span data-stu-id="bc72a-306">PhysicalAddress mapping</span></span>

<span data-ttu-id="bc72a-307">此功能是 [@ralmsdeveloper](https://github.com/ralmsdeveloper) 在社区中贡献的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-307">This feature was contributed from the community by [@ralmsdeveloper](https://github.com/ralmsdeveloper).</span></span> <span data-ttu-id="bc72a-308">非常感谢贡献此功能！</span><span class="sxs-lookup"><span data-stu-id="bc72a-308">Many thanks for the contribution!</span></span>

<span data-ttu-id="bc72a-309">标准 .NET [PhysicalAddress 类](/dotnet/api/system.net.networkinformation.physicaladdress)现自动映射到尚不具备原生支持的数据库的字符串列中。</span><span class="sxs-lookup"><span data-stu-id="bc72a-309">The standard .NET [PhysicalAddress class](/dotnet/api/system.net.networkinformation.physicaladdress) is now automatically mapped to a string column for databases that do not already have native support.</span></span> <span data-ttu-id="bc72a-310">有关详细信息，请参阅下面的 `IPAddress` 示例。</span><span class="sxs-lookup"><span data-stu-id="bc72a-310">For more information, see the examples for `IPAddress` below.</span></span>

## <a name="preview-7"></a><span data-ttu-id="bc72a-311">预览版 7</span><span class="sxs-lookup"><span data-stu-id="bc72a-311">Preview 7</span></span>

### <a name="dbcontextfactory"></a><span data-ttu-id="bc72a-312">DbContextFactory</span><span class="sxs-lookup"><span data-stu-id="bc72a-312">DbContextFactory</span></span>

<span data-ttu-id="bc72a-313">EF Core 5.0 引入了 `AddDbContextFactory` 和 `AddPooledDbContextFactory`，可用于在应用程序的依赖关系注入 (D.I.) 容器中注册工厂来创建 DbContext 实例。</span><span class="sxs-lookup"><span data-stu-id="bc72a-313">EF Core 5.0 introduces `AddDbContextFactory` and `AddPooledDbContextFactory` to register a factory for creating DbContext instances in the application's dependency injection (D.I.) container.</span></span> <span data-ttu-id="bc72a-314">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-314">For example:</span></span>

```csharp
services.AddDbContextFactory<SomeDbContext>(b =>
    b.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test"));
```

<span data-ttu-id="bc72a-315">应用程序服务（例如 ASP.NET Core 控制器）之后可依赖于服务构造函数中的 `IDbContextFactory<TContext>`。</span><span class="sxs-lookup"><span data-stu-id="bc72a-315">Application services such as ASP.NET Core controllers can then depend on `IDbContextFactory<TContext>` in the service constructor.</span></span> <span data-ttu-id="bc72a-316">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-316">For example:</span></span>

```csharp
public class MyController
{
    private readonly IDbContextFactory<SomeDbContext> _contextFactory;

    public MyController(IDbContextFactory<SomeDbContext> contextFactory)
    {
        _contextFactory = contextFactory;
    }
}
```

<span data-ttu-id="bc72a-317">之后可根据需要创建和使用 DbContext 实例。</span><span class="sxs-lookup"><span data-stu-id="bc72a-317">DbContext instances can then be created and used as needed.</span></span> <span data-ttu-id="bc72a-318">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-318">For example:</span></span>

```csharp
public void DoSomeThing()
{
    using (var context = _contextFactory.CreateDbContext())
    {
        // ...
    }
}
```

<span data-ttu-id="bc72a-319">请注意，以这种方式创建的 DbContext 实例并非由应用程序的服务提供程序进行管理，因此必须由应用程序释放。</span><span class="sxs-lookup"><span data-stu-id="bc72a-319">Note that the DbContext instances created in this way are _not_ managed by the application's service provider and therefore must be disposed by the application.</span></span> <span data-ttu-id="bc72a-320">此类分离对于 Blazor 应用程序非常有用（此情况下建议使用 `IDbContextFactory`），但在其他方案中也很有用。</span><span class="sxs-lookup"><span data-stu-id="bc72a-320">This decoupling is very useful for Blazor applications, where using `IDbContextFactory` is recommended, but may also be useful in other scenarios.</span></span>

<span data-ttu-id="bc72a-321">你可通过调用 `AddPooledDbContextFactory` 来共用 DbContext 实例。</span><span class="sxs-lookup"><span data-stu-id="bc72a-321">DbContext instances can be pooled by calling `AddPooledDbContextFactory`.</span></span> <span data-ttu-id="bc72a-322">此类共用的原理与 `AddDbContextPool` 相同，且限制也相同。</span><span class="sxs-lookup"><span data-stu-id="bc72a-322">This pooling works the same way as for `AddDbContextPool`, and also has the same limitations.</span></span>

<span data-ttu-id="bc72a-323">记录信息可通过问题 [#2523](https://github.com/dotnet/EntityFramework.Docs/issues/2523) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-323">Documentation is tracked by issue [#2523](https://github.com/dotnet/EntityFramework.Docs/issues/2523).</span></span>

### <a name="reset-dbcontext-state"></a><span data-ttu-id="bc72a-324">重置 DbContext 状态</span><span class="sxs-lookup"><span data-stu-id="bc72a-324">Reset DbContext state</span></span>

<span data-ttu-id="bc72a-325">EF Core 5.0 引入了 `ChangeTracker.Clear()`，它可清除所有被跟踪实体的 DbContext。</span><span class="sxs-lookup"><span data-stu-id="bc72a-325">EF Core 5.0 introduces `ChangeTracker.Clear()` which clears the DbContext of all tracked entities.</span></span> <span data-ttu-id="bc72a-326">当使用为每个工作单元创建生存期较短的新上下文实例的最佳做法时，通常不需要这样做。</span><span class="sxs-lookup"><span data-stu-id="bc72a-326">This should usually not be needed when using the best practice of creating a new, short-lived context instance for each unit-of-work.</span></span> <span data-ttu-id="bc72a-327">但如果需要重置 DbContext 实例的状态，则使用新的 `Clear()` 方法比大量分离所有实体的性能和可靠性更强。</span><span class="sxs-lookup"><span data-stu-id="bc72a-327">However, if there is a need to reset the state of a DbContext instance, then using the new `Clear()` method is more performant and robust than mass-detaching all entities.</span></span>

<span data-ttu-id="bc72a-328">记录信息可通过问题 [#2524](https://github.com/dotnet/EntityFramework.Docs/issues/2524) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-328">Documentation is tracked by issue [#2524](https://github.com/dotnet/EntityFramework.Docs/issues/2524).</span></span>

### <a name="new-pattern-for-store-generated-defaults"></a><span data-ttu-id="bc72a-329">存储生成的默认值的新模式</span><span class="sxs-lookup"><span data-stu-id="bc72a-329">New pattern for store-generated defaults</span></span>

<span data-ttu-id="bc72a-330">EF Core 允许为可能还有默认值约束的列设置显式值。</span><span class="sxs-lookup"><span data-stu-id="bc72a-330">EF Core allows an explicit value to be set for a column that may also have default value constraint.</span></span> <span data-ttu-id="bc72a-331">EF Core 使用 type 属性类型的 CLR 默认值作为此项的 sentinel；如果该值不是 CLR 默认值，则会将其插入，否则将使用数据库默认值。</span><span class="sxs-lookup"><span data-stu-id="bc72a-331">EF Core uses the CLR default of type property type as a sentinel for this; if the value is not the CLR default, then it is inserted, otherwise the database default is used.</span></span>

<span data-ttu-id="bc72a-332">这会使 CLR 默认值不是良好的 sentinel（最值得注意的是 `bool` 属性）的类型出现问题。</span><span class="sxs-lookup"><span data-stu-id="bc72a-332">This creates problems for types where the CLR default is not a good sentinel--most notably, `bool` properties.</span></span> <span data-ttu-id="bc72a-333">EF Core 5.0 现允许支持字段在此类情况下可为 null。</span><span class="sxs-lookup"><span data-stu-id="bc72a-333">EF Core 5.0 now allows the backing field to be nullable for cases like this.</span></span> <span data-ttu-id="bc72a-334">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-334">For example:</span></span>

```csharp
public class Blog
{
    private bool? _isValid;

    public bool IsValid
    {
        get => _isValid ?? false;
        set => _isValid = value;
    }
}
```

<span data-ttu-id="bc72a-335">请注意，支持字段可为 null，但公开属性不可为 null。</span><span class="sxs-lookup"><span data-stu-id="bc72a-335">Note that the backing field is nullable, but the publicly exposed property is not.</span></span> <span data-ttu-id="bc72a-336">这允许 sentinel 值为 `null`，而不影响实体类型的公开内容。</span><span class="sxs-lookup"><span data-stu-id="bc72a-336">This allows the sentinel value to be `null` without impacting the public surface of the entity type.</span></span> <span data-ttu-id="bc72a-337">在本例中，如果绝不设置 `IsValid`，则将使用数据库默认值，因为支持字段仍为 null。</span><span class="sxs-lookup"><span data-stu-id="bc72a-337">In this case, if the `IsValid` is never set, then the database default will be used since the backing field remains null.</span></span> <span data-ttu-id="bc72a-338">如果设置 `true` 或 `false`，则此值将显式保存到数据库中。</span><span class="sxs-lookup"><span data-stu-id="bc72a-338">If either `true` or `false` are set, then this value is saved explicitly to the database.</span></span>

<span data-ttu-id="bc72a-339">记录信息可通过问题 [#2525](https://github.com/dotnet/EntityFramework.Docs/issues/2525) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-339">Documentation is tracked by issue [#2525](https://github.com/dotnet/EntityFramework.Docs/issues/2525).</span></span>

### <a name="cosmos-partition-keys"></a><span data-ttu-id="bc72a-340">Cosmos 分区键</span><span class="sxs-lookup"><span data-stu-id="bc72a-340">Cosmos partition keys</span></span>

<span data-ttu-id="bc72a-341">EF Core 允许将 Cosmos 分区键包含在 EF 模型中。</span><span class="sxs-lookup"><span data-stu-id="bc72a-341">EF Core allows the Cosmos partition key is included in the EF model.</span></span> <span data-ttu-id="bc72a-342">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-342">For example:</span></span>

```csharp
modelBuilder.Entity<Customer>().HasPartitionKey(b => b.AlternateKey)
```

<span data-ttu-id="bc72a-343">从预览版 7 开始，分区键将包含在实体类型的 PK 中，用于改进某些查询的性能。</span><span class="sxs-lookup"><span data-stu-id="bc72a-343">Starting with preview 7, the partition key is included in the entity type's PK and is used to improved performance in some queries.</span></span>

<span data-ttu-id="bc72a-344">记录信息可通过问题 [#2471](https://github.com/dotnet/EntityFramework.Docs/issues/2471) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-344">Documentation is tracked by issue [#2471](https://github.com/dotnet/EntityFramework.Docs/issues/2471).</span></span>

### <a name="cosmos-configuration"></a><span data-ttu-id="bc72a-345">Cosmos 配置</span><span class="sxs-lookup"><span data-stu-id="bc72a-345">Cosmos configuration</span></span>

<span data-ttu-id="bc72a-346">EF Core 5.0 改进了 Cosmos 和 Cosmos 连接的配置。</span><span class="sxs-lookup"><span data-stu-id="bc72a-346">EF Core 5.0 improves configuration of Cosmos and Cosmos connections.</span></span>

<span data-ttu-id="bc72a-347">之前，EF Core 需要在连接到 Cosmos 数据库时显式指定终结点和密钥。</span><span class="sxs-lookup"><span data-stu-id="bc72a-347">Previously, EF Core required the end-point and key to be specified explicitly when connecting to a Cosmos database.</span></span> <span data-ttu-id="bc72a-348">而 EF Core 5.0 允许使用连接字符串。</span><span class="sxs-lookup"><span data-stu-id="bc72a-348">EF Core 5.0 allows use of a connection string instead.</span></span> <span data-ttu-id="bc72a-349">此外，EF Core 5.0 还允许显式设置 WebProxy 实例。</span><span class="sxs-lookup"><span data-stu-id="bc72a-349">In addition, EF Core 5.0 allows the WebProxy instance to be explicitly set.</span></span> <span data-ttu-id="bc72a-350">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-350">For example:</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .UseCosmos("my-cosmos-connection-string", "MyDb",
            cosmosOptionsBuilder =>
            {
                cosmosOptionsBuilder.WebProxy(myProxyInstance);
            });
```

<span data-ttu-id="bc72a-351">现在还可配置许多其他超时值、限制等。</span><span class="sxs-lookup"><span data-stu-id="bc72a-351">Many other timeout values, limits, etc. can now also be configured.</span></span> <span data-ttu-id="bc72a-352">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-352">For example:</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .UseCosmos("my-cosmos-connection-string", "MyDb",
            cosmosOptionsBuilder =>
            {
                cosmosOptionsBuilder.LimitToEndpoint();
                cosmosOptionsBuilder.RequestTimeout(requestTimeout);
                cosmosOptionsBuilder.OpenTcpConnectionTimeout(timeout);
                cosmosOptionsBuilder.IdleTcpConnectionTimeout(timeout);
                cosmosOptionsBuilder.GatewayModeMaxConnectionLimit(connectionLimit);
                cosmosOptionsBuilder.MaxTcpConnectionsPerEndpoint(connectionLimit);
                cosmosOptionsBuilder.MaxRequestsPerTcpConnection(requestLimit);
            });
```

<span data-ttu-id="bc72a-353">最后，默认的连接模式现为 `ConnectionMode.Gateway`，兼容性通常更好。</span><span class="sxs-lookup"><span data-stu-id="bc72a-353">Finally, the default connection mode is now `ConnectionMode.Gateway`, which is generally more compatible.</span></span>

<span data-ttu-id="bc72a-354">记录信息可通过问题 [#2471](https://github.com/dotnet/EntityFramework.Docs/issues/2471) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-354">Documentation is tracked by issue [#2471](https://github.com/dotnet/EntityFramework.Docs/issues/2471).</span></span>

### <a name="scaffold-dbcontext-now-singularizes"></a><span data-ttu-id="bc72a-355">Scaffold-DbContext 现为单数形式</span><span class="sxs-lookup"><span data-stu-id="bc72a-355">Scaffold-DbContext now singularizes</span></span>

<span data-ttu-id="bc72a-356">此前，当从现有数据库搭建 DbContext 时，EF Core 将创建与数据库中的表名称匹配的实体类型名称。</span><span class="sxs-lookup"><span data-stu-id="bc72a-356">Previously when scaffolding a DbContext from an existing database, EF Core will create entity type names that match the table names in the database.</span></span> <span data-ttu-id="bc72a-357">例如，表 `People` 和 `Addresses` 对应的实体类型名称为 `People` 和 `Addresses`。</span><span class="sxs-lookup"><span data-stu-id="bc72a-357">For example, tables `People` and `Addresses` resulted in entity types named `People` and `Addresses`.</span></span>

<span data-ttu-id="bc72a-358">在之前的版本中，此行为是通过注册复数化服务来配置的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-358">In previous releases, this behavior was configurable through registration of a pluralization service.</span></span> <span data-ttu-id="bc72a-359">现在，在 EF Core 5.0 中，[Humanizer](https://www.nuget.org/packages/Humanizer.Core/) 包用作默认复数化服务。</span><span class="sxs-lookup"><span data-stu-id="bc72a-359">Now in EF Core 5.0, the [Humanizer](https://www.nuget.org/packages/Humanizer.Core/) package is used as a default pluralization service.</span></span> <span data-ttu-id="bc72a-360">这意味着 `People` 和 `Addresses` 现将被反向工程处理为名叫 `Person` 和 `Address` 的实体类型。</span><span class="sxs-lookup"><span data-stu-id="bc72a-360">This means tables `People` and `Addresses` will now be reverse engineered to entity types named `Person` and `Address`.</span></span>

### <a name="savepoints"></a><span data-ttu-id="bc72a-361">保存点</span><span class="sxs-lookup"><span data-stu-id="bc72a-361">Savepoints</span></span>

<span data-ttu-id="bc72a-362">EF Core 现支持[保存点](/sql/t-sql/language-elements/save-transaction-transact-sql#remarks)，它们可更好地控制执行多个操作的事务。</span><span class="sxs-lookup"><span data-stu-id="bc72a-362">EF Core now supports [savepoints](/sql/t-sql/language-elements/save-transaction-transact-sql#remarks) for greater control over transactions that execute multiple operations.</span></span>

<span data-ttu-id="bc72a-363">你可手动创建、发布和回滚保存点。</span><span class="sxs-lookup"><span data-stu-id="bc72a-363">Savepoints can be manually created, released, and rolled back.</span></span> <span data-ttu-id="bc72a-364">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-364">For example:</span></span>

```csharp
context.Database.CreateSavepoint("MySavePoint");
```

<span data-ttu-id="bc72a-365">此外，在执行 `SaveChanges` 失败时，EF Core 现在会回滚到上一个保存点。</span><span class="sxs-lookup"><span data-stu-id="bc72a-365">In addition, EF Core will now roll back to the last savepoint when executing `SaveChanges` fails.</span></span> <span data-ttu-id="bc72a-366">因此，可重试 SaveChanges，而不用重试整个事务。</span><span class="sxs-lookup"><span data-stu-id="bc72a-366">This allows SaveChanges to be re-tried without re-trying the entire transaction.</span></span>

<span data-ttu-id="bc72a-367">记录信息可通过问题 [#2429](https://github.com/dotnet/EntityFramework.Docs/issues/2429) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-367">Documentation is tracked by issue [#2429](https://github.com/dotnet/EntityFramework.Docs/issues/2429).</span></span>

## <a name="preview-6"></a><span data-ttu-id="bc72a-368">预览版 6</span><span class="sxs-lookup"><span data-stu-id="bc72a-368">Preview 6</span></span>

### <a name="split-queries-for-related-collections"></a><span data-ttu-id="bc72a-369">适合相关集合的拆分查询</span><span class="sxs-lookup"><span data-stu-id="bc72a-369">Split queries for related collections</span></span>

<span data-ttu-id="bc72a-370">从 EF Core 3.0 开始，EF Core 始终会为每个 LINQ 查询生成一个 SQL 查询。</span><span class="sxs-lookup"><span data-stu-id="bc72a-370">Starting with EF Core 3.0, EF Core always generates a single SQL query for each LINQ query.</span></span> <span data-ttu-id="bc72a-371">这可确保在使用中的事务模式的约束内返回的数据保持一致。</span><span class="sxs-lookup"><span data-stu-id="bc72a-371">This ensures consistency of the data returned within the constraints of the transaction mode in use.</span></span> <span data-ttu-id="bc72a-372">但是，当查询使用 `Include` 或投影来返回多个相关集合时，速度可能会变得非常慢。</span><span class="sxs-lookup"><span data-stu-id="bc72a-372">However, this can become very slow when the query uses `Include` or a projection to bring back multiple related collections.</span></span>

<span data-ttu-id="bc72a-373">EF Core 5.0 现允许将包含相关集合的一个 LINQ 查询拆分成多个 SQL 查询。</span><span class="sxs-lookup"><span data-stu-id="bc72a-373">EF Core 5.0 now allows a single LINQ query including related collections to be split into multiple SQL queries.</span></span> <span data-ttu-id="bc72a-374">这可能明显提升性能，但如果在两次查询之间数据发生了变化，则可能导致返回的结果不一致。</span><span class="sxs-lookup"><span data-stu-id="bc72a-374">This can significantly improve performance, but can result in inconsistency in the results returned if the data changes between the two queries.</span></span> <span data-ttu-id="bc72a-375">你能使用可序列化的事务或快照事务来缓解这种情况并通过拆分查询实现一致性，但这可能会带来其他性能成本并导致行为差异。</span><span class="sxs-lookup"><span data-stu-id="bc72a-375">Serializable or snapshot transactions can be used to mitigate this and achieve consistency with split queries, but that may bring other performance costs and behavioral difference.</span></span>

#### <a name="split-queries-with-include"></a><span data-ttu-id="bc72a-376">用 Include 拆分查询</span><span class="sxs-lookup"><span data-stu-id="bc72a-376">Split queries with Include</span></span>

<span data-ttu-id="bc72a-377">例如，请考虑使用 `Include` 提取两个级别的相关集合的查询：</span><span class="sxs-lookup"><span data-stu-id="bc72a-377">For example, consider a query that pulls in two levels of related collections using `Include`:</span></span>

```csharp
var artists = context.Artists
    .Include(e => e.Albums).ThenInclude(e => e.Tags)
    .ToList();
```

<span data-ttu-id="bc72a-378">默认情况下，EF Core 将使用 SQLite 提供程序生成以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="bc72a-378">By default, EF Core will generate the following SQL when using the SQLite provider:</span></span>

```sql
SELECT "a"."Id", "a"."Name", "t0"."Id", "t0"."ArtistId", "t0"."Title", "t0"."Id0", "t0"."AlbumId", "t0"."Name"
FROM "Artists" AS "a"
LEFT JOIN (
    SELECT "a0"."Id", "a0"."ArtistId", "a0"."Title", "t"."Id" AS "Id0", "t"."AlbumId", "t"."Name"
    FROM "Album" AS "a0"
    LEFT JOIN "Tag" AS "t" ON "a0"."Id" = "t"."AlbumId"
) AS "t0" ON "a"."Id" = "t0"."ArtistId"
ORDER BY "a"."Id", "t0"."Id", "t0"."Id0"
```

<span data-ttu-id="bc72a-379">可使用新的 `AsSplitQuery` API 来更改此行为。</span><span class="sxs-lookup"><span data-stu-id="bc72a-379">The new `AsSplitQuery` API can be used to change this behavior.</span></span> <span data-ttu-id="bc72a-380">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-380">For example:</span></span>

```csharp
var artists = context.Artists
    .AsSplitQuery()
    .Include(e => e.Albums).ThenInclude(e => e.Tags)
    .ToList();
```

<span data-ttu-id="bc72a-381">AsSplitQuery 适用于所有关系数据库提供程序，且如同 AsNoTracking 一样，可在查询中的任何位置使用。</span><span class="sxs-lookup"><span data-stu-id="bc72a-381">AsSplitQuery is available for all relational database providers and can be used anywhere in the query, just like AsNoTracking.</span></span> <span data-ttu-id="bc72a-382">EF Core 现将生成以下 3 个 SQL 查询：</span><span class="sxs-lookup"><span data-stu-id="bc72a-382">EF Core will now generate the following three SQL queries:</span></span>

```sql
SELECT "a"."Id", "a"."Name"
FROM "Artists" AS "a"
ORDER BY "a"."Id"

SELECT "a0"."Id", "a0"."ArtistId", "a0"."Title", "a"."Id"
FROM "Artists" AS "a"
INNER JOIN "Album" AS "a0" ON "a"."Id" = "a0"."ArtistId"
ORDER BY "a"."Id", "a0"."Id"

SELECT "t"."Id", "t"."AlbumId", "t"."Name", "a"."Id", "a0"."Id"
FROM "Artists" AS "a"
INNER JOIN "Album" AS "a0" ON "a"."Id" = "a0"."ArtistId"
INNER JOIN "Tag" AS "t" ON "a0"."Id" = "t"."AlbumId"
ORDER BY "a"."Id", "a0"."Id"
```

<span data-ttu-id="bc72a-383">支持对查询根目录执行各种操作。</span><span class="sxs-lookup"><span data-stu-id="bc72a-383">All operations on the query root are supported.</span></span> <span data-ttu-id="bc72a-384">其中包括 OrderBy/Skip/Take、联接操作、FirstOrDefault 和类似的单结果选择操作。</span><span class="sxs-lookup"><span data-stu-id="bc72a-384">This includes OrderBy/Skip/Take, Join operations, FirstOrDefault and similar single result selecting operations.</span></span>

<span data-ttu-id="bc72a-385">请注意，预览版 6 中不支持带 OrderBy/Skip/Take 的 Filtered Include，但它们可在日常版本中使用，且将包含在预览版 7 中。</span><span class="sxs-lookup"><span data-stu-id="bc72a-385">Note that filtered Includes with OrderBy/Skip/Take are not supported in preview 6, but are available in the daily builds and will be included in preview 7.</span></span>

#### <a name="split-queries-with-collection-projections"></a><span data-ttu-id="bc72a-386">用集合投影来拆分查询</span><span class="sxs-lookup"><span data-stu-id="bc72a-386">Split queries with collection projections</span></span>

<span data-ttu-id="bc72a-387">在投影中加载集合时，也可使用 `AsSplitQuery`。</span><span class="sxs-lookup"><span data-stu-id="bc72a-387">`AsSplitQuery` can also be used when collections are loaded in projections.</span></span> <span data-ttu-id="bc72a-388">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-388">For example:</span></span>

```csharp
context.Artists
    .AsSplitQuery()
    .Select(e => new
    {
        Artist = e,
        Albums = e.Albums,
    }).ToList();
```

<span data-ttu-id="bc72a-389">使用 SQLite 提供程序时，此 LINQ 查询会生成以下两个 SQL 查询：</span><span class="sxs-lookup"><span data-stu-id="bc72a-389">This LINQ query generates the following two SQL queries when using the SQLite provider:</span></span>

```sql
SELECT "a"."Id", "a"."Name"
FROM "Artists" AS "a"
ORDER BY "a"."Id"

SELECT "a0"."Id", "a0"."ArtistId", "a0"."Title", "a"."Id"
FROM "Artists" AS "a"
INNER JOIN "Album" AS "a0" ON "a"."Id" = "a0"."ArtistId"
ORDER BY "a"."Id"
```

<span data-ttu-id="bc72a-390">请注意，仅支持将集合具体化。</span><span class="sxs-lookup"><span data-stu-id="bc72a-390">Note that only materialization of the collection is supported.</span></span> <span data-ttu-id="bc72a-391">上例中 `e.Albums` 之后的任何组合都不会产生拆分查询。</span><span class="sxs-lookup"><span data-stu-id="bc72a-391">Any composition after `e.Albums` in above case won't result in a split query.</span></span> <span data-ttu-id="bc72a-392">这方面的改进由 [#21234](https://github.com/dotnet/efcore/issues/21234) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-392">Improvements in this area are tracked by [#21234](https://github.com/dotnet/efcore/issues/21234).</span></span>

### <a name="indexattribute"></a><span data-ttu-id="bc72a-393">IndexAttribute</span><span class="sxs-lookup"><span data-stu-id="bc72a-393">IndexAttribute</span></span>

<span data-ttu-id="bc72a-394">这个新的 IndexAttribute 可置于实体类型上来指定单列的索引。</span><span class="sxs-lookup"><span data-stu-id="bc72a-394">The new IndexAttribute can be placed on an entity type to specify an index for a single column.</span></span> <span data-ttu-id="bc72a-395">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-395">For example:</span></span>

```csharp
[Index(nameof(FullName), IsUnique = true)]
public class User
{
    public int Id { get; set; }

    [MaxLength(128)]
    public string FullName { get; set; }
}
```

<span data-ttu-id="bc72a-396">对于 SQL Server，迁移随后将生成以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="bc72a-396">For SQL Server, Migrations will then generate the following SQL:</span></span>

```sql
CREATE UNIQUE INDEX [IX_Users_FullName]
    ON [Users] ([FullName])
    WHERE [FullName] IS NOT NULL;
```

<span data-ttu-id="bc72a-397">IndexAttribute 也可用于指定横跨多个列的索引。</span><span class="sxs-lookup"><span data-stu-id="bc72a-397">IndexAttribute can also be used to specify an index spanning multiple columns.</span></span> <span data-ttu-id="bc72a-398">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-398">For example:</span></span>

```csharp
[Index(nameof(FirstName), nameof(LastName), IsUnique = true)]
public class User
{
    public int Id { get; set; }

    [MaxLength(64)]
    public string FirstName { get; set; }

    [MaxLength(64)]
    public string LastName { get; set; }
}
```

<span data-ttu-id="bc72a-399">对于 SQL Server，这会导致：</span><span class="sxs-lookup"><span data-stu-id="bc72a-399">For SQL Server, this results in:</span></span>

```sql
CREATE UNIQUE INDEX [IX_Users_FirstName_LastName]
    ON [Users] ([FirstName], [LastName])
    WHERE [FirstName] IS NOT NULL AND [LastName] IS NOT NULL;
```

<span data-ttu-id="bc72a-400">记录信息可通过问题 [#2407](https://github.com/dotnet/EntityFramework.Docs/issues/2407) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-400">Documentation is tracked by issue [#2407](https://github.com/dotnet/EntityFramework.Docs/issues/2407).</span></span>

### <a name="improved-query-translation-exceptions"></a><span data-ttu-id="bc72a-401">改进了查询转换异常时显示的消息</span><span class="sxs-lookup"><span data-stu-id="bc72a-401">Improved query translation exceptions</span></span>

<span data-ttu-id="bc72a-402">我们将继续改进在查询转换失败时生成的异常消息。</span><span class="sxs-lookup"><span data-stu-id="bc72a-402">We are continuing to improve the exception messages generated when query translation fails.</span></span> <span data-ttu-id="bc72a-403">例如，以下查询使用未映射的属性 `IsSigned`：</span><span class="sxs-lookup"><span data-stu-id="bc72a-403">For example, this query uses the unmapped property `IsSigned`:</span></span>

```csharp
var artists = context.Artists.Where(e => e.IsSigned).ToList();
```

<span data-ttu-id="bc72a-404">EF Core 将引发以下异常，指出由于 `IsSigned` 未映射而导致转换失败：</span><span class="sxs-lookup"><span data-stu-id="bc72a-404">EF Core will throw the following exception indicating that translation failed because `IsSigned` is not mapped:</span></span>

```exception
Unhandled exception. System.InvalidOperationException: The LINQ expression 'DbSet<Artist>()
   .Where(a => a.IsSigned)' could not be translated. Additional information: Translation of member 'IsSigned' on entity type 'Artist' failed. Possibly the specified member is not mapped. Either rewrite the query in a form that can be translated, or switch to client evaluation explicitly by inserting a call to either AsEnumerable(), AsAsyncEnumerable(), ToList(), or ToListAsync(). See <https://go.microsoft.com/fwlink/?linkid=2101038> for more information.
```

<span data-ttu-id="bc72a-405">同样地，在尝试用依赖区域性的语义来转换字符串比较时，现将生成信息更丰富的异常消息。</span><span class="sxs-lookup"><span data-stu-id="bc72a-405">Similarly, better exception messages are now generated when attempting to translate string comparisons with culture-dependent semantics.</span></span> <span data-ttu-id="bc72a-406">例如，以下查询尝试使用 `StringComparison.CurrentCulture`：</span><span class="sxs-lookup"><span data-stu-id="bc72a-406">For example, this query attempts to use `StringComparison.CurrentCulture`:</span></span>

```csharp
var artists = context.Artists
    .Where(e => e.Name.Equals("The Unicorns", StringComparison.CurrentCulture))
    .ToList();
```

<span data-ttu-id="bc72a-407">EF Core 现将引发以下异常：</span><span class="sxs-lookup"><span data-stu-id="bc72a-407">EF Core will now throw the following exception:</span></span>

```exception
Unhandled exception. System.InvalidOperationException: The LINQ expression 'DbSet<Artist>()
     .Where(a => a.Name.Equals(
         value: "The Unicorns",
         comparisonType: CurrentCulture))' could not be translated. Additional information: Translation of 'string.Equals' method which takes 'StringComparison' argument is not supported. See <https://go.microsoft.com/fwlink/?linkid=2129535> for more information. Either rewrite the query in a form that can be translated, or switch to client evaluation explicitly by inserting a call to either AsEnumerable(), AsAsyncEnumerable(), ToList(), or ToListAsync(). See <https://go.microsoft.com/fwlink/?linkid=2101038> for more information.
```

### <a name="specify-transaction-id"></a><span data-ttu-id="bc72a-408">指定事务 ID</span><span class="sxs-lookup"><span data-stu-id="bc72a-408">Specify transaction ID</span></span>

<span data-ttu-id="bc72a-409">此功能是 [@Marusyk](https://github.com/Marusyk) 在社区中贡献的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-409">This feature was contributed from the community by [@Marusyk](https://github.com/Marusyk).</span></span> <span data-ttu-id="bc72a-410">非常感谢贡献此功能！</span><span class="sxs-lookup"><span data-stu-id="bc72a-410">Many thanks for the contribution!</span></span>

<span data-ttu-id="bc72a-411">EF Core 公开一个事务 ID 来跨调用关联事务。</span><span class="sxs-lookup"><span data-stu-id="bc72a-411">EF Core exposes a transaction ID for correlation of transactions across calls.</span></span> <span data-ttu-id="bc72a-412">此 ID 通常是在启动事务时由 EF Core 设置的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-412">This ID typically set by EF Core when a transaction is started.</span></span> <span data-ttu-id="bc72a-413">如果转而由应用程序启动事务，则应用程序可使用此功能来显式设置事务 ID，使其在所用的每个位置都正确关联。</span><span class="sxs-lookup"><span data-stu-id="bc72a-413">If the application starts the transaction instead, then this feature allows the application to explicitly set the transaction ID so it is correlated correctly everywhere it is used.</span></span> <span data-ttu-id="bc72a-414">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-414">For example:</span></span>

```csharp
using (context.Database.UseTransaction(myTransaction, myId))
{
   ...
}
```

### <a name="ipaddress-mapping"></a><span data-ttu-id="bc72a-415">IPAddress 映射</span><span class="sxs-lookup"><span data-stu-id="bc72a-415">IPAddress mapping</span></span>

<span data-ttu-id="bc72a-416">此功能是 [@ralmsdeveloper](https://github.com/ralmsdeveloper) 在社区中贡献的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-416">This feature was contributed from the community by [@ralmsdeveloper](https://github.com/ralmsdeveloper).</span></span> <span data-ttu-id="bc72a-417">非常感谢贡献此功能！</span><span class="sxs-lookup"><span data-stu-id="bc72a-417">Many thanks for the contribution!</span></span>

<span data-ttu-id="bc72a-418">标准 .NET [IPAddress 类](/dotnet/api/system.net.ipaddress) 现自动映射到尚不具备原生支持的数据库的字符串列中。</span><span class="sxs-lookup"><span data-stu-id="bc72a-418">The standard .NET [IPAddress class](/dotnet/api/system.net.ipaddress) is now automatically mapped to a string column for databases that do not already have native support.</span></span> <span data-ttu-id="bc72a-419">例如，请考虑映射以下实体类型：</span><span class="sxs-lookup"><span data-stu-id="bc72a-419">For example, consider mapping this entity type:</span></span>

```csharp
public class Host
{
    public int Id { get; set; }
    public IPAddress Address { get; set; }
}
```

<span data-ttu-id="bc72a-420">在 SQL Server 上，这将导致迁移创建以下表：</span><span class="sxs-lookup"><span data-stu-id="bc72a-420">On SQL Server, this will result in Migrations creating the following table:</span></span>

```sql
CREATE TABLE [Host] (
    [Id] int NOT NULL,
    [Address] nvarchar(45) NULL,
    CONSTRAINT [PK_Host] PRIMARY KEY ([Id]));
```

<span data-ttu-id="bc72a-421">之后，可按正常的方式添加实体：</span><span class="sxs-lookup"><span data-stu-id="bc72a-421">Entities can then be added in the normal way:</span></span>

```csharp
context.AddRange(
    new Host { Address = IPAddress.Parse("127.0.0.1")},
    new Host { Address = IPAddress.Parse("0000:0000:0000:0000:0000:0000:0000:0001")});
```

<span data-ttu-id="bc72a-422">所生成的 SQL 将插入规范化的 IPv4 或 IPv6 地址：</span><span class="sxs-lookup"><span data-stu-id="bc72a-422">And the resulting SQL will insert the normalized IPv4 or IPv6 address:</span></span>

```sql
Executed DbCommand (14ms) [Parameters=[@p0='1', @p1='127.0.0.1' (Size = 45), @p2='2', @p3='::1' (Size = 45)], CommandType='Text', CommandTimeout='30']
      SET NOCOUNT ON;
      INSERT INTO [Host] ([Id], [Address])
      VALUES (@p0, @p1), (@p2, @p3);
```

### <a name="exclude-onconfiguring-when-scaffolding"></a><span data-ttu-id="bc72a-423">在创建基架时排除 OnConfiguring</span><span class="sxs-lookup"><span data-stu-id="bc72a-423">Exclude OnConfiguring when scaffolding</span></span>

<span data-ttu-id="bc72a-424">在基于现有数据库创建 DbContext 的基架时，EF Core 会默认创建一个 OnConfiguring 重载，该重载有一个连接字符串，以便上下文可立即供用户使用。</span><span class="sxs-lookup"><span data-stu-id="bc72a-424">When a DbContext is scaffolded from an existing database, EF Core by default creates an OnConfiguring overload with a connection string so that the context is immediately usable.</span></span> <span data-ttu-id="bc72a-425">不过，如果你只有带 OnConfiguring 的分部类，或者你要用其他方式部署上下文，则此方法将不起作用。</span><span class="sxs-lookup"><span data-stu-id="bc72a-425">However, this is not useful if you already have a partial class with OnConfiguring, or if you are configuring the context some other way.</span></span>

<span data-ttu-id="bc72a-426">若要解决这种情况，现可指示基架命令不生成 OnConfiguring。</span><span class="sxs-lookup"><span data-stu-id="bc72a-426">To address this, the scaffolding commands can now be instructed to omit generation of OnConfiguring.</span></span> <span data-ttu-id="bc72a-427">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-427">For example:</span></span>

```console
dotnet ef dbcontext scaffold "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Chinook" Microsoft.EntityFrameworkCore.SqlServer --no-onconfiguring
```

<span data-ttu-id="bc72a-428">或者在包管理器控制台中：</span><span class="sxs-lookup"><span data-stu-id="bc72a-428">Or in the Package Manager Console:</span></span>

```console
Scaffold-DbContext 'Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Chinook' Microsoft.EntityFrameworkCore.SqlServer -NoOnConfiguring
```

<span data-ttu-id="bc72a-429">请注意，建议使用[命名的连接字符串和安全存储（如用户机密）](xref:core/managing-schemas/scaffolding#configuration-and-user-secrets)。</span><span class="sxs-lookup"><span data-stu-id="bc72a-429">Note that we recommend using [a named connection string and secure storage like User Secrets](xref:core/managing-schemas/scaffolding#configuration-and-user-secrets).</span></span>

### <a name="translations-for-firstordefault-on-strings"></a><span data-ttu-id="bc72a-430">转换字符串中的 FirstOrDefault</span><span class="sxs-lookup"><span data-stu-id="bc72a-430">Translations for FirstOrDefault on strings</span></span>

<span data-ttu-id="bc72a-431">此功能是 [@dvoreckyaa](https://github.com/dvoreckyaa) 在社区中贡献的。</span><span class="sxs-lookup"><span data-stu-id="bc72a-431">This feature was contributed from the community by [@dvoreckyaa](https://github.com/dvoreckyaa).</span></span> <span data-ttu-id="bc72a-432">非常感谢贡献此功能！</span><span class="sxs-lookup"><span data-stu-id="bc72a-432">Many thanks for the contribution!</span></span>

<span data-ttu-id="bc72a-433">现将转换字符串中字符的 FirstOrDefault 和类似运算符。</span><span class="sxs-lookup"><span data-stu-id="bc72a-433">FirstOrDefault and similar operators for characters in strings are now translated.</span></span> <span data-ttu-id="bc72a-434">例如，使用 SQL Server 时，以下 LINQ 查询：</span><span class="sxs-lookup"><span data-stu-id="bc72a-434">For example, this LINQ query:</span></span>

```csharp
context.Customers.Where(c => c.ContactName.FirstOrDefault() == 'A').ToList();
```

<span data-ttu-id="bc72a-435">将转换为以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="bc72a-435">Will be translated to the following SQL when using SQL Server:</span></span>

```sql
SELECT [c].[Id], [c].[ContactName]
FROM [Customer] AS [c]
WHERE SUBSTRING([c].[ContactName], 1, 1) = N'A'
```

### <a name="simplify-case-blocks"></a><span data-ttu-id="bc72a-436">简化 CASE 块</span><span class="sxs-lookup"><span data-stu-id="bc72a-436">Simplify case blocks</span></span>

<span data-ttu-id="bc72a-437">EF Core 现使用 CASE 块生成效果更佳的查询。</span><span class="sxs-lookup"><span data-stu-id="bc72a-437">EF Core now generates better queries with CASE blocks.</span></span> <span data-ttu-id="bc72a-438">例如，以下 LINQ 查询：</span><span class="sxs-lookup"><span data-stu-id="bc72a-438">For example, this LINQ query:</span></span>

```csharp
context.Weapons
    .OrderBy(w => w.Name.CompareTo("Marcus' Lancer") == 0)
    .ThenBy(w => w.Id)
```

<span data-ttu-id="bc72a-439">之前在 SQL Server 上正式转换为：</span><span class="sxs-lookup"><span data-stu-id="bc72a-439">Was on SQL Server formally translated to:</span></span>

```sql
SELECT [w].[Id], [w].[AmmunitionType], [w].[IsAutomatic], [w].[Name], [w].[OwnerFullName], [w].[SynergyWithId]
FROM [Weapons] AS [w]
ORDER BY CASE
    WHEN (CASE
        WHEN [w].[Name] = N'Marcus'' Lancer' THEN 0
        WHEN [w].[Name] > N'Marcus'' Lancer' THEN 1
        WHEN [w].[Name] < N'Marcus'' Lancer' THEN -1
    END = 0) AND CASE
        WHEN [w].[Name] = N'Marcus'' Lancer' THEN 0
        WHEN [w].[Name] > N'Marcus'' Lancer' THEN 1
        WHEN [w].[Name] < N'Marcus'' Lancer' THEN -1
    END IS NOT NULL THEN CAST(1 AS bit)
    ELSE CAST(0 AS bit)
END, [w].[Id]");
```

<span data-ttu-id="bc72a-440">但它现在转换为：</span><span class="sxs-lookup"><span data-stu-id="bc72a-440">But is now translated to:</span></span>

```sql
SELECT [w].[Id], [w].[AmmunitionType], [w].[IsAutomatic], [w].[Name], [w].[OwnerFullName], [w].[SynergyWithId]
FROM [Weapons] AS [w]
ORDER BY CASE
    WHEN ([w].[Name] = N'Marcus'' Lancer') AND [w].[Name] IS NOT NULL THEN CAST(1 AS bit)
    ELSE CAST(0 AS bit)
END, [w].[Id]");
```

## <a name="preview-5"></a><span data-ttu-id="bc72a-441">预览版 5</span><span class="sxs-lookup"><span data-stu-id="bc72a-441">Preview 5</span></span>

### <a name="database-collations"></a><span data-ttu-id="bc72a-442">数据库排序规则</span><span class="sxs-lookup"><span data-stu-id="bc72a-442">Database collations</span></span>

<span data-ttu-id="bc72a-443">现可在 EF 模式中指定数据库的默认排序规则。</span><span class="sxs-lookup"><span data-stu-id="bc72a-443">The default collation for a database can now be specified in the EF model.</span></span> <span data-ttu-id="bc72a-444">这将传输到生成的迁移，在创建数据库时设置排序规则。</span><span class="sxs-lookup"><span data-stu-id="bc72a-444">This will flow through to generated migrations to set the collation when the database is created.</span></span> <span data-ttu-id="bc72a-445">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-445">For example:</span></span>

```csharp
modelBuilder.UseCollation("German_PhoneBook_CI_AS");
```

<span data-ttu-id="bc72a-446">然后，迁移会生成以下内容，在 SQL Server 上创建数据库：</span><span class="sxs-lookup"><span data-stu-id="bc72a-446">Migrations then generates the following to create the database on SQL Server:</span></span>

```sql
CREATE DATABASE [Test]
COLLATE German_PhoneBook_CI_AS;
```

<span data-ttu-id="bc72a-447">可指定用于特定数据库列的排序规则。</span><span class="sxs-lookup"><span data-stu-id="bc72a-447">The collation to use for specific database columns can also be specified.</span></span> <span data-ttu-id="bc72a-448">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-448">For example:</span></span>

```csharp
modelBuilder
    .Entity<User>()
    .Property(e => e.Name)
    .UseCollation("German_PhoneBook_CI_AS");
```

<span data-ttu-id="bc72a-449">对于不使用迁移的列，可在创建 DbContext 基架时从数据库对排序规则进行反向工程处理。</span><span class="sxs-lookup"><span data-stu-id="bc72a-449">For those not using migrations, collations are now reverse-engineered from the database when scaffolding a DbContext.</span></span>

<span data-ttu-id="bc72a-450">最后，可通过 `EF.Functions.Collate()` 使用不同的排序规则进行即席查询。</span><span class="sxs-lookup"><span data-stu-id="bc72a-450">Finally, the `EF.Functions.Collate()` allows for ad-hoc queries using different collations.</span></span> <span data-ttu-id="bc72a-451">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-451">For example:</span></span>

```csharp
context.Users.Single(e => EF.Functions.Collate(e.Name, "French_CI_AS") == "Jean-Michel Jarre");
```

<span data-ttu-id="bc72a-452">这将为 SQL Server 生成以下查询：</span><span class="sxs-lookup"><span data-stu-id="bc72a-452">This will generate the following query for SQL Server:</span></span>

```sql
SELECT TOP(2) [u].[Id], [u].[Name]
FROM [Users] AS [u]
WHERE [u].[Name] COLLATE French_CI_AS = N'Jean-Michel Jarre'
```

<span data-ttu-id="bc72a-453">请注意，应谨慎使用即席排序规则，因为它们可能会对数据库性能造成不良影响。</span><span class="sxs-lookup"><span data-stu-id="bc72a-453">Note that ad-hoc collations should be used with care as they can negatively impact database performance.</span></span>

<span data-ttu-id="bc72a-454">文档可通过问题 [#2273](https://github.com/dotnet/EntityFramework.Docs/issues/2273) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-454">Documentation is tracked by issue [#2273](https://github.com/dotnet/EntityFramework.Docs/issues/2273).</span></span>

### <a name="flow-arguments-into-idesigntimedbcontextfactory"></a><span data-ttu-id="bc72a-455">将参数传输到 IDesignTimeDbContextFactory</span><span class="sxs-lookup"><span data-stu-id="bc72a-455">Flow arguments into IDesignTimeDbContextFactory</span></span>

<span data-ttu-id="bc72a-456">参数现从命令行传输到 [IDesignTimeDbContextFactory](/dotnet/api/microsoft.entityframeworkcore.design.idesigntimedbcontextfactory-1) 的 `CreateDbContext` 方法。</span><span class="sxs-lookup"><span data-stu-id="bc72a-456">Arguments are now flowed from the command line into the `CreateDbContext` method of [IDesignTimeDbContextFactory](/dotnet/api/microsoft.entityframeworkcore.design.idesigntimedbcontextfactory-1).</span></span> <span data-ttu-id="bc72a-457">例如，若要指明这是开发生成，可以在命令行上传递自定义参数（例如 `dev`）：</span><span class="sxs-lookup"><span data-stu-id="bc72a-457">For example, to indicate this is a dev build, a custom argument (e.g. `dev`) can be passed on the command line:</span></span>

```console
dotnet ef migrations add two --verbose --dev
```

<span data-ttu-id="bc72a-458">然后，该参数将传输到工厂，它在这里可用于控制如何创建和初始化上下文。</span><span class="sxs-lookup"><span data-stu-id="bc72a-458">This argument will then flow into the factory, where it can be used to control how the context is created and initialized.</span></span> <span data-ttu-id="bc72a-459">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-459">For example:</span></span>

```csharp
public class MyDbContextFactory : IDesignTimeDbContextFactory<SomeDbContext>
{
    public SomeDbContext CreateDbContext(string[] args)
        => new SomeDbContext(args.Contains("--dev"));
}
```

<span data-ttu-id="bc72a-460">文档可通过问题 [#2419](https://github.com/dotnet/EntityFramework.Docs/issues/2419) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-460">Documentation is tracked by issue [#2419](https://github.com/dotnet/EntityFramework.Docs/issues/2419).</span></span>

### <a name="no-tracking-queries-with-identity-resolution"></a><span data-ttu-id="bc72a-461">具有标识解析的非跟踪查询</span><span class="sxs-lookup"><span data-stu-id="bc72a-461">No-tracking queries with identity resolution</span></span>

<span data-ttu-id="bc72a-462">现可将非跟踪查询配置来执行标识解析。</span><span class="sxs-lookup"><span data-stu-id="bc72a-462">No-tracking queries can now be configured to perform identity resolution.</span></span> <span data-ttu-id="bc72a-463">例如，以下查询将为每个 Post 创建一个新的 Blog 实例，即使每个 Blog 的主键相同也是如此。</span><span class="sxs-lookup"><span data-stu-id="bc72a-463">For example, the following query will create a new Blog instance for each Post, even if each Blog has the same primary key.</span></span>

```csharp
context.Posts.AsNoTracking().Include(e => e.Blog).ToList();
```

<span data-ttu-id="bc72a-464">然而，可更改此查询来确保只创建一个 Blog 实例，代价通常是稍微拖慢一些速度和总是使用更多的内存：</span><span class="sxs-lookup"><span data-stu-id="bc72a-464">However, at the expense of usually being slightly slower and always using more memory, this query can be changed to ensure only a single Blog instance is created:</span></span>

```csharp
context.Posts.AsNoTracking().PerformIdentityResolution().Include(e => e.Blog).ToList();
```

<span data-ttu-id="bc72a-465">请注意，这仅适用于非跟踪查询，因为所有跟踪查询已显示此行为。</span><span class="sxs-lookup"><span data-stu-id="bc72a-465">Note that this is only useful for no-tracking queries since all tracking queries already exhibit this behavior.</span></span> <span data-ttu-id="bc72a-466">此外，在 API 评审后，将更改 `PerformIdentityResolution` 语法。</span><span class="sxs-lookup"><span data-stu-id="bc72a-466">Also, following API review, the `PerformIdentityResolution` syntax will be changed.</span></span> <span data-ttu-id="bc72a-467">请查看 [#19877](https://github.com/dotnet/efcore/issues/19877#issuecomment-637371073)。</span><span class="sxs-lookup"><span data-stu-id="bc72a-467">See [#19877](https://github.com/dotnet/efcore/issues/19877#issuecomment-637371073).</span></span>

<span data-ttu-id="bc72a-468">文档可通过问题 [#1895](https://github.com/dotnet/EntityFramework.Docs/issues/1895) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-468">Documentation is tracked by issue [#1895](https://github.com/dotnet/EntityFramework.Docs/issues/1895).</span></span>

### <a name="stored-persisted-computed-columns"></a><span data-ttu-id="bc72a-469">存储的（持久化）计算列</span><span class="sxs-lookup"><span data-stu-id="bc72a-469">Stored (persisted) computed columns</span></span>

<span data-ttu-id="bc72a-470">大多数数据库都允许在计算后存储计算得到的列值。</span><span class="sxs-lookup"><span data-stu-id="bc72a-470">Most databases allow computed column values to be stored after computation.</span></span> <span data-ttu-id="bc72a-471">虽然这会占用磁盘空间，但仅在更新时对计算的列计算一次，而不是在每次检索它的值时都计算。</span><span class="sxs-lookup"><span data-stu-id="bc72a-471">While this takes up disk space, the computed column is calculated only once on update, instead of each time its value is retrieved.</span></span> <span data-ttu-id="bc72a-472">这样也可对某些数据库编制列的索引。</span><span class="sxs-lookup"><span data-stu-id="bc72a-472">This also allows the column to be indexed for some databases.</span></span>

<span data-ttu-id="bc72a-473">EF Core 5.0 允许将计算列配置为存储计算列。</span><span class="sxs-lookup"><span data-stu-id="bc72a-473">EF Core 5.0 allows computed columns to be configured as stored.</span></span> <span data-ttu-id="bc72a-474">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-474">For example:</span></span>

```csharp
modelBuilder
    .Entity<User>()
    .Property(e => e.SomethingComputed)
    .HasComputedColumnSql("my sql", stored: true);
```

### <a name="sqlite-computed-columns"></a><span data-ttu-id="bc72a-475">SQLite 计算列</span><span class="sxs-lookup"><span data-stu-id="bc72a-475">SQLite computed columns</span></span>

<span data-ttu-id="bc72a-476">EF Core 现支持在 SQLite 数据库中使用计算列。</span><span class="sxs-lookup"><span data-stu-id="bc72a-476">EF Core now supports computed columns in SQLite databases.</span></span>

## <a name="preview-4"></a><span data-ttu-id="bc72a-477">预览版 4</span><span class="sxs-lookup"><span data-stu-id="bc72a-477">Preview 4</span></span>

### <a name="configure-database-precisionscale-in-model"></a><span data-ttu-id="bc72a-478">在模型中配置数据库精度/小数位数</span><span class="sxs-lookup"><span data-stu-id="bc72a-478">Configure database precision/scale in model</span></span>

<span data-ttu-id="bc72a-479">现在，可以使用模型生成器指定属性的精度和小数位数。</span><span class="sxs-lookup"><span data-stu-id="bc72a-479">Precision and scale for a property can now be specified using the model builder.</span></span> <span data-ttu-id="bc72a-480">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-480">For example:</span></span>

```csharp
modelBuilder
    .Entity<Blog>()
    .Property(b => b.Numeric)
    .HasPrecision(16, 4);
```

<span data-ttu-id="bc72a-481">还可以通过完整的数据库类型（如“decimal(16,4)”）设置精度和小数位数。</span><span class="sxs-lookup"><span data-stu-id="bc72a-481">Precision and scale can still be set via the full database type, such as "decimal(16,4)".</span></span>

<span data-ttu-id="bc72a-482">文档可通过问题 [#527](https://github.com/dotnet/EntityFramework.Docs/issues/527) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-482">Documentation is tracked by issue [#527](https://github.com/dotnet/EntityFramework.Docs/issues/527).</span></span>

### <a name="specify-sql-server-index-fill-factor"></a><span data-ttu-id="bc72a-483">指定 SQL Server 索引填充因子</span><span class="sxs-lookup"><span data-stu-id="bc72a-483">Specify SQL Server index fill factor</span></span>

<span data-ttu-id="bc72a-484">现在可以在 SQL Server 上创建索引时指定填充因子。</span><span class="sxs-lookup"><span data-stu-id="bc72a-484">The fill factor can now be specified when creating an index on SQL Server.</span></span> <span data-ttu-id="bc72a-485">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-485">For example:</span></span>

```csharp
modelBuilder
    .Entity<Customer>()
    .HasIndex(e => e.Name)
    .HasFillFactor(90);
```

## <a name="preview-3"></a><span data-ttu-id="bc72a-486">预览版 3</span><span class="sxs-lookup"><span data-stu-id="bc72a-486">Preview 3</span></span>

### <a name="filtered-include"></a><span data-ttu-id="bc72a-487">经过筛选的包含</span><span class="sxs-lookup"><span data-stu-id="bc72a-487">Filtered Include</span></span>

<span data-ttu-id="bc72a-488">Include 方法现在支持筛选包含的实体。</span><span class="sxs-lookup"><span data-stu-id="bc72a-488">The Include method now supports filtering of the entities included.</span></span> <span data-ttu-id="bc72a-489">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-489">For example:</span></span>

```csharp
var blogs = context.Blogs
    .Include(e => e.Posts.Where(p => p.Title.Contains("Cheese")))
    .ToList();
```

<span data-ttu-id="bc72a-490">此查询将一并返回包含每个关联文章的博客（仅当文章标题包含“Cheese”时）。</span><span class="sxs-lookup"><span data-stu-id="bc72a-490">This query will return blogs together with each associated post, but only when the post title contains "Cheese".</span></span>

<span data-ttu-id="bc72a-491">Skip 和 Take 也可用于减少包含的实体数量。</span><span class="sxs-lookup"><span data-stu-id="bc72a-491">Skip and Take can also be used to reduce the number of included entities.</span></span> <span data-ttu-id="bc72a-492">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-492">For example:</span></span>

```csharp
var blogs = context.Blogs
    .Include(e => e.Posts.OrderByDescending(post => post.Title).Take(5)))
    .ToList();
```

<span data-ttu-id="bc72a-493">此查询将返回博客，每个博客最多包含 5 篇文章。</span><span class="sxs-lookup"><span data-stu-id="bc72a-493">This query will return blogs with at most five posts included on each blog.</span></span>

<span data-ttu-id="bc72a-494">有关完整详细信息，请参阅 [Include 文档](xref:core/querying/related-data#filtered-include)。</span><span class="sxs-lookup"><span data-stu-id="bc72a-494">See the [Include documentation](xref:core/querying/related-data#filtered-include) for full details.</span></span>

### <a name="new-modelbuilder-api-for-navigation-properties"></a><span data-ttu-id="bc72a-495">用于导航属性的新 ModelBuilder API</span><span class="sxs-lookup"><span data-stu-id="bc72a-495">New ModelBuilder API for navigation properties</span></span>

<span data-ttu-id="bc72a-496">导航属性主要在[定义关系](xref:core/modeling/relationships)时配置。</span><span class="sxs-lookup"><span data-stu-id="bc72a-496">Navigation properties are primarily configured when [defining relationships](xref:core/modeling/relationships).</span></span> <span data-ttu-id="bc72a-497">但是，在导航属性需要额外配置的情况下，可以使用新的 `Navigation` 方法。</span><span class="sxs-lookup"><span data-stu-id="bc72a-497">However, the new `Navigation` method can be used in the cases where navigation properties need additional configuration.</span></span> <span data-ttu-id="bc72a-498">例如，如需在根据约定找不到支持字段时设置导航的支持字段，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="bc72a-498">For example, to set a backing field for the navigation when the field would not be found by convention:</span></span>

```csharp
modelBuilder.Entity<Blog>().Navigation(e => e.Posts).HasField("_myposts");
```

<span data-ttu-id="bc72a-499">请注意：`Navigation` API 不会替换关系配置。</span><span class="sxs-lookup"><span data-stu-id="bc72a-499">Note that the `Navigation` API does not replace relationship configuration.</span></span> <span data-ttu-id="bc72a-500">但是，它允许在已发现或定义的关系中进行其他导航属性配置。</span><span class="sxs-lookup"><span data-stu-id="bc72a-500">Instead it allows additional configuration of navigation properties in already discovered or defined relationships.</span></span>

<span data-ttu-id="bc72a-501">请参阅[配置导航属性文档](xref:core/modeling/relationships#configuring-navigation-properties)。</span><span class="sxs-lookup"><span data-stu-id="bc72a-501">See the [Configuring Navigation Properties documentation](xref:core/modeling/relationships#configuring-navigation-properties).</span></span>

### <a name="new-command-line-parameters-for-namespaces-and-connection-strings"></a><span data-ttu-id="bc72a-502">用于命名空间和连接字符串的新命令行参数</span><span class="sxs-lookup"><span data-stu-id="bc72a-502">New command-line parameters for namespaces and connection strings</span></span>

<span data-ttu-id="bc72a-503">迁移和基架现在允许在命令行上指定命名空间。</span><span class="sxs-lookup"><span data-stu-id="bc72a-503">Migrations and scaffolding now allow namespaces to be specified on the command line.</span></span> <span data-ttu-id="bc72a-504">例如，如需对数据库进行反向工程，将上下文和模型类放在不同的命名空间中：</span><span class="sxs-lookup"><span data-stu-id="bc72a-504">For example, to reverse engineer a database putting the context and model classes in different namespaces:</span></span>

```dotnetcli
dotnet ef dbcontext scaffold "connection string" Microsoft.EntityFrameworkCore.SqlServer --context-namespace "My.Context" --namespace "My.Model"
```

<span data-ttu-id="bc72a-505">有关完整详细信息，请参阅[迁移](xref:core/managing-schemas/migrations/index#namespaces)和[反向工程](xref:core/managing-schemas/scaffolding#directories-and-namespaces)文档。</span><span class="sxs-lookup"><span data-stu-id="bc72a-505">See the [Migrations](xref:core/managing-schemas/migrations/index#namespaces) and [Reverse Engineering](xref:core/managing-schemas/scaffolding#directories-and-namespaces) documentation for full details.</span></span>

---
<span data-ttu-id="bc72a-506">此外，连接字符串现在可以传递到 `database-update` 命令：</span><span class="sxs-lookup"><span data-stu-id="bc72a-506">Also, a connection string can now be passed to the `database-update` command:</span></span>

```dotnetcli
dotnet ef database update --connection "connection string"
```

<span data-ttu-id="bc72a-507">有关完整的详细信息，请参阅[工具文档](xref:core/cli/dotnet#dotnet-ef-database-update)。</span><span class="sxs-lookup"><span data-stu-id="bc72a-507">See the [Tools documentation](xref:core/cli/dotnet#dotnet-ef-database-update) for full details.</span></span>

<span data-ttu-id="bc72a-508">等效参数已添加到 VS 程序包管理器控制台中使用的 PowerShell 命令中。</span><span class="sxs-lookup"><span data-stu-id="bc72a-508">Equivalent parameters have also been added to the PowerShell commands used in the VS Package Manager Console.</span></span>

### <a name="enabledetailederrors-has-returned"></a><span data-ttu-id="bc72a-509">EnableDetailedErrors 已返回</span><span class="sxs-lookup"><span data-stu-id="bc72a-509">EnableDetailedErrors has returned</span></span>

<span data-ttu-id="bc72a-510">出于性能原因，在从数据库读取值时，EF 不会执行额外的 null 检查。</span><span class="sxs-lookup"><span data-stu-id="bc72a-510">For performance reasons, EF doesn't do additional null-checks when reading values from the database.</span></span> <span data-ttu-id="bc72a-511">当遇到意外的 null 时，这可能会导致难以查找根本原因的异常。</span><span class="sxs-lookup"><span data-stu-id="bc72a-511">This can result in exceptions that are hard to root-cause when an unexpected null is encountered.</span></span>

<span data-ttu-id="bc72a-512">使用 `EnableDetailedErrors` 会将额外的 null 检查添加到查询中，这样一来，对于较小的性能开销，这些错误就更容易追溯到根本原因。</span><span class="sxs-lookup"><span data-stu-id="bc72a-512">Using `EnableDetailedErrors` will add extra null checking to queries such that, for a small performance overhead, these errors are easier to trace back to a root cause.</span></span>

<span data-ttu-id="bc72a-513">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-513">For example:</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .EnableDetailedErrors()
        .EnableSensitiveDataLogging() // Often also useful with EnableDetailedErrors
        .UseSqlServer(Your.SqlServerConnectionString);
```

<span data-ttu-id="bc72a-514">文档可通过问题 [#955](https://github.com/dotnet/EntityFramework.Docs/issues/955) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-514">Documentation is tracked by issue [#955](https://github.com/dotnet/EntityFramework.Docs/issues/955).</span></span>

### <a name="cosmos-partition-keys"></a><span data-ttu-id="bc72a-515">Cosmos 分区键</span><span class="sxs-lookup"><span data-stu-id="bc72a-515">Cosmos partition keys</span></span>

<span data-ttu-id="bc72a-516">现在可以在查询中指定用于给定查询的分区键。</span><span class="sxs-lookup"><span data-stu-id="bc72a-516">The partition key to use for a given query can now be specified in the query.</span></span> <span data-ttu-id="bc72a-517">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-517">For example:</span></span>

```csharp
await context.Set<Customer>()
             .WithPartitionKey(myPartitionKey)
             .FirstAsync();
```

<span data-ttu-id="bc72a-518">文档可通过问题 [#2199](https://github.com/dotnet/EntityFramework.Docs/issues/2199) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-518">Documentation is tracked by issue [#2199](https://github.com/dotnet/EntityFramework.Docs/issues/2199).</span></span>

### <a name="support-for-the-sql-server-datalength-function"></a><span data-ttu-id="bc72a-519">支持 SQL Server DATALENGTH 函数</span><span class="sxs-lookup"><span data-stu-id="bc72a-519">Support for the SQL Server DATALENGTH function</span></span>

<span data-ttu-id="bc72a-520">可以使用新的 `EF.Functions.DataLength` 方法访问它。</span><span class="sxs-lookup"><span data-stu-id="bc72a-520">This can be accessed using the new `EF.Functions.DataLength` method.</span></span> <span data-ttu-id="bc72a-521">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-521">For example:</span></span>

```csharp
var count = context.Orders.Count(c => 100 < EF.Functions.DataLength(c.OrderDate));
```

## <a name="preview-2"></a><span data-ttu-id="bc72a-522">预览版 2</span><span class="sxs-lookup"><span data-stu-id="bc72a-522">Preview 2</span></span>

### <a name="use-a-c-attribute-to-specify-a-property-backing-field"></a><span data-ttu-id="bc72a-523">使用 C# 特性指定属性支持字段</span><span class="sxs-lookup"><span data-stu-id="bc72a-523">Use a C# attribute to specify a property backing field</span></span>

<span data-ttu-id="bc72a-524">现在，可以使用 C# 特性指定属性的支持字段。</span><span class="sxs-lookup"><span data-stu-id="bc72a-524">A C# attribute can now be used to specify the backing field for a property.</span></span> <span data-ttu-id="bc72a-525">使用此特性，即使在不能自动找到支持字段时，EF Core 也能正常读写支持字段。</span><span class="sxs-lookup"><span data-stu-id="bc72a-525">This attribute allows EF Core to still write to and read from the backing field as would normally happen, even when the backing field cannot be found automatically.</span></span> <span data-ttu-id="bc72a-526">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-526">For example:</span></span>

```csharp
public class Blog
{
    private string _mainTitle;

    public int Id { get; set; }

    [BackingField(nameof(_mainTitle))]
    public string Title
    {
        get => _mainTitle;
        set => _mainTitle = value;
    }
}
```

<span data-ttu-id="bc72a-527">文档可通过问题 [#2230](https://github.com/dotnet/EntityFramework.Docs/issues/2230) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-527">Documentation is tracked by issue [#2230](https://github.com/dotnet/EntityFramework.Docs/issues/2230).</span></span>

### <a name="complete-discriminator-mapping"></a><span data-ttu-id="bc72a-528">完成鉴别器映射</span><span class="sxs-lookup"><span data-stu-id="bc72a-528">Complete discriminator mapping</span></span>

<span data-ttu-id="bc72a-529">EF Core 使用鉴别器列进行 [继承层次结构的 TPH 映射](xref:core/modeling/inheritance)。</span><span class="sxs-lookup"><span data-stu-id="bc72a-529">EF Core uses a discriminator column for [TPH mapping of an inheritance hierarchy](xref:core/modeling/inheritance).</span></span> <span data-ttu-id="bc72a-530">只要 EF Core 知道该鉴别器的所有可能的值，就可能实现某些性能改进。</span><span class="sxs-lookup"><span data-stu-id="bc72a-530">Some performance enhancements are possible so long as EF Core knows all possible values for the discriminator.</span></span> <span data-ttu-id="bc72a-531">EF Core 5.0 现在可实现这些改进。</span><span class="sxs-lookup"><span data-stu-id="bc72a-531">EF Core 5.0 now implements these enhancements.</span></span>

<span data-ttu-id="bc72a-532">例如，对于返回层次结构中所有类型的查询，旧版 EF Core 总是会生成以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="bc72a-532">For example, previous versions of EF Core would always generate this SQL for a query returning all types in a hierarchy:</span></span>

```sql
SELECT [a].[Id], [a].[Discriminator], [a].[Name]
FROM [Animal] AS [a]
WHERE [a].[Discriminator] IN (N'Animal', N'Cat', N'Dog', N'Human')
```

<span data-ttu-id="bc72a-533">配置完整的鉴别器映射后，EF Core 5.0 现在将生成以下内容：</span><span class="sxs-lookup"><span data-stu-id="bc72a-533">EF Core 5.0 will now generate the following when a complete discriminator mapping is configured:</span></span>

```sql
SELECT [a].[Id], [a].[Discriminator], [a].[Name]
FROM [Animal] AS [a]
```

<span data-ttu-id="bc72a-534">从预览版 3 开始，这将成为默认行为。</span><span class="sxs-lookup"><span data-stu-id="bc72a-534">It will be the default behavior starting with preview 3.</span></span>

### <a name="performance-improvements-in-microsoftdatasqlite"></a><span data-ttu-id="bc72a-535">Microsoft.Data.Sqlite 中的性能改进</span><span class="sxs-lookup"><span data-stu-id="bc72a-535">Performance improvements in Microsoft.Data.Sqlite</span></span>

<span data-ttu-id="bc72a-536">我们对 SQLIte 进行了两项性能改进：</span><span class="sxs-lookup"><span data-stu-id="bc72a-536">We have made two performance improvements for SQLIte:</span></span>

* <span data-ttu-id="bc72a-537">现在，利用 SqliteBlob 和流，可以更高效地使用 GetBytes、GetChars 和 GetTextReader 检索二进制和字符串数据。</span><span class="sxs-lookup"><span data-stu-id="bc72a-537">Retrieving binary and string data with GetBytes, GetChars, and GetTextReader is now more efficient by making use of SqliteBlob and streams.</span></span>
* <span data-ttu-id="bc72a-538">SqliteConnection 的初始化现在很慢。</span><span class="sxs-lookup"><span data-stu-id="bc72a-538">Initialization of SqliteConnection is now lazy.</span></span>

<span data-ttu-id="bc72a-539">这些改进已实施到 ADO.NET Microsoft.Data.Sqlite 提供程序中，因此还可以在 EF Core 之外提升性能。</span><span class="sxs-lookup"><span data-stu-id="bc72a-539">These improvements are in the ADO.NET Microsoft.Data.Sqlite provider and hence also improve performance outside of EF Core.</span></span>

## <a name="preview-1"></a><span data-ttu-id="bc72a-540">预览版 1</span><span class="sxs-lookup"><span data-stu-id="bc72a-540">Preview 1</span></span>

### <a name="simple-logging"></a><span data-ttu-id="bc72a-541">简单的日志记录</span><span class="sxs-lookup"><span data-stu-id="bc72a-541">Simple logging</span></span>

<span data-ttu-id="bc72a-542">此特性会添加类似于 EF6 中 `Database.Log` 的功能。</span><span class="sxs-lookup"><span data-stu-id="bc72a-542">This feature adds functionality similar to `Database.Log` in EF6.</span></span> <span data-ttu-id="bc72a-543">也就是说，它提供了一种从 EF Core 获取日志的简单方法，而不需要配置任何类型的外部日志记录框架。</span><span class="sxs-lookup"><span data-stu-id="bc72a-543">That is, it provides a simple way to get logs from EF Core without the need to configure any kind of external logging framework.</span></span>

<span data-ttu-id="bc72a-544">初步文档包含在 [2019 年 12 月 5 日发布的 EF 每周状态](https://github.com/dotnet/efcore/issues/15403#issuecomment-562332863)中。</span><span class="sxs-lookup"><span data-stu-id="bc72a-544">Preliminary documentation is included in the [EF weekly status for December 5, 2019](https://github.com/dotnet/efcore/issues/15403#issuecomment-562332863).</span></span>

<span data-ttu-id="bc72a-545">其他文档可通过问题 [#2085](https://github.com/dotnet/EntityFramework.Docs/issues/2085) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-545">Additional documentation is tracked by issue [#2085](https://github.com/dotnet/EntityFramework.Docs/issues/2085).</span></span>

### <a name="simple-way-to-get-generated-sql"></a><span data-ttu-id="bc72a-546">获取生成的 SQL 的简单方法</span><span class="sxs-lookup"><span data-stu-id="bc72a-546">Simple way to get generated SQL</span></span>

<span data-ttu-id="bc72a-547">EF Core 5.0 引入了 `ToQueryString` 扩展方法，该方法会返回执行 LINQ 查询时 EF Core 生成的 SQL。</span><span class="sxs-lookup"><span data-stu-id="bc72a-547">EF Core 5.0 introduces the `ToQueryString` extension method, which will return the SQL that EF Core will generate when executing a LINQ query.</span></span>

<span data-ttu-id="bc72a-548">初步文档包含在 [2020 年 1 月 9 日发布的 EF 每周状态](https://github.com/dotnet/efcore/issues/19549#issuecomment-572823246)中。</span><span class="sxs-lookup"><span data-stu-id="bc72a-548">Preliminary documentation is included in the [EF weekly status for January 9, 2020](https://github.com/dotnet/efcore/issues/19549#issuecomment-572823246).</span></span>

<span data-ttu-id="bc72a-549">其他文档可通过问题 [#1331](https://github.com/dotnet/EntityFramework.Docs/issues/1331) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-549">Additional documentation is tracked by issue [#1331](https://github.com/dotnet/EntityFramework.Docs/issues/1331).</span></span>

### <a name="use-a-c-attribute-to-indicate-that-an-entity-has-no-key"></a><span data-ttu-id="bc72a-550">使用 C# 特性指示实体没有键</span><span class="sxs-lookup"><span data-stu-id="bc72a-550">Use a C# attribute to indicate that an entity has no key</span></span>

<span data-ttu-id="bc72a-551">现在可以将实体类型配置为不使用新 `KeylessAttribute`的键。</span><span class="sxs-lookup"><span data-stu-id="bc72a-551">An entity type can now be configured as having no key using the new `KeylessAttribute`.</span></span> <span data-ttu-id="bc72a-552">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-552">For example:</span></span>

```csharp
[Keyless]
public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public int Zip { get; set; }
}
```

<span data-ttu-id="bc72a-553">文档可通过问题 [#2186](https://github.com/dotnet/EntityFramework.Docs/issues/2186) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-553">Documentation is tracked by issue [#2186](https://github.com/dotnet/EntityFramework.Docs/issues/2186).</span></span>

### <a name="connection-or-connection-string-can-be-changed-on-initialized-dbcontext"></a><span data-ttu-id="bc72a-554">可在初始化的 DbContext 上更改连接或连接字符串</span><span class="sxs-lookup"><span data-stu-id="bc72a-554">Connection or connection string can be changed on initialized DbContext</span></span>

<span data-ttu-id="bc72a-555">现在可以更轻松地创建 DbContext 实例，而无需任何连接或连接字符串。</span><span class="sxs-lookup"><span data-stu-id="bc72a-555">It is now easier to create a DbContext instance without any connection or connection string.</span></span> <span data-ttu-id="bc72a-556">而且，现在可以在上下文实例上更改连接或连接字符串。</span><span class="sxs-lookup"><span data-stu-id="bc72a-556">Also, the connection or connection string can now be mutated on the context instance.</span></span> <span data-ttu-id="bc72a-557">此功能使得同一个上下文实例能够动态地连接到不同的数据库。</span><span class="sxs-lookup"><span data-stu-id="bc72a-557">This feature allows the same context instance to dynamically connect to different databases.</span></span>

<span data-ttu-id="bc72a-558">文档可通过问题 [#2075](https://github.com/dotnet/EntityFramework.Docs/issues/2075) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-558">Documentation is tracked by issue [#2075](https://github.com/dotnet/EntityFramework.Docs/issues/2075).</span></span>

### <a name="change-tracking-proxies"></a><span data-ttu-id="bc72a-559">更改跟踪代理</span><span class="sxs-lookup"><span data-stu-id="bc72a-559">Change-tracking proxies</span></span>

<span data-ttu-id="bc72a-560">EF Core 现在可以生成自动实现 [INotifyPropertyChanging](/dotnet/api/system.componentmodel.inotifypropertychanging) 和 [INotifyPropertyChanged](/dotnet/api/system.componentmodel.inotifypropertychanged) 的运行时代理。</span><span class="sxs-lookup"><span data-stu-id="bc72a-560">EF Core can now generate runtime proxies that automatically implement [INotifyPropertyChanging](/dotnet/api/system.componentmodel.inotifypropertychanging) and [INotifyPropertyChanged](/dotnet/api/system.componentmodel.inotifypropertychanged).</span></span> <span data-ttu-id="bc72a-561">这些代理会将实体属性的值更改直接报告给 EF Core，从而无需扫描更改。</span><span class="sxs-lookup"><span data-stu-id="bc72a-561">These then report value changes on entity properties directly to EF Core, avoiding the need to scan for changes.</span></span> <span data-ttu-id="bc72a-562">不过，代理有其自身的一组限制，因此并不适合所有人使用。</span><span class="sxs-lookup"><span data-stu-id="bc72a-562">However, proxies come with their own set of limitations, so they are not for everyone.</span></span>

<span data-ttu-id="bc72a-563">文档可通过问题 [#2076](https://github.com/dotnet/EntityFramework.Docs/issues/2076) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-563">Documentation is tracked by issue [#2076](https://github.com/dotnet/EntityFramework.Docs/issues/2076).</span></span>

### <a name="enhanced-debug-views"></a><span data-ttu-id="bc72a-564">增强的调试视图</span><span class="sxs-lookup"><span data-stu-id="bc72a-564">Enhanced debug views</span></span>

<span data-ttu-id="bc72a-565">调试视图是调试问题时查看 EF Core 内部情况的一种简单方法。</span><span class="sxs-lookup"><span data-stu-id="bc72a-565">Debug views are an easy way to look at the internals of EF Core when debugging issues.</span></span> <span data-ttu-id="bc72a-566">在一段时间之前，我们就已经实现了模型的调试视图。</span><span class="sxs-lookup"><span data-stu-id="bc72a-566">A debug view for the Model was implemented some time ago.</span></span> <span data-ttu-id="bc72a-567">对于 EF Core 5.0，我们使模型视图更易于读取，并在状态管理器中为跟踪的实体添加了新的调试视图。</span><span class="sxs-lookup"><span data-stu-id="bc72a-567">For EF Core 5.0, we have made the model view easier to read and added a new debug view for tracked entities in the state manager.</span></span>

<span data-ttu-id="bc72a-568">初步文档包含在 [2019 年 12 月 12 日发布的 EF 每周状态](https://github.com/dotnet/efcore/issues/15403#issuecomment-565196206)中。</span><span class="sxs-lookup"><span data-stu-id="bc72a-568">Preliminary documentation is included in the [EF weekly status for December 12, 2019](https://github.com/dotnet/efcore/issues/15403#issuecomment-565196206).</span></span>

<span data-ttu-id="bc72a-569">其他文档可通过问题 [#2086](https://github.com/dotnet/EntityFramework.Docs/issues/2086) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-569">Additional documentation is tracked by issue [#2086](https://github.com/dotnet/EntityFramework.Docs/issues/2086).</span></span>

### <a name="improved-handling-of-database-null-semantics"></a><span data-ttu-id="bc72a-570">改进了对数据库 null 语义的处理</span><span class="sxs-lookup"><span data-stu-id="bc72a-570">Improved handling of database null semantics</span></span>

<span data-ttu-id="bc72a-571">关系数据库通常将 NULL 视为未知值，因此它不等于任何其他 NULL 值。</span><span class="sxs-lookup"><span data-stu-id="bc72a-571">Relational databases typically treat NULL as an unknown value and therefore not equal to any other NULL.</span></span> <span data-ttu-id="bc72a-572">而 C# 将 null 视为与其他任何 null 相比均相等的定义值。</span><span class="sxs-lookup"><span data-stu-id="bc72a-572">While C# treats null as a defined value, which compares equal to any other null.</span></span> <span data-ttu-id="bc72a-573">默认情况下，EF Core 会转换查询，使查询使用 C# null 语义。</span><span class="sxs-lookup"><span data-stu-id="bc72a-573">EF Core by default translates queries so that they use C# null semantics.</span></span> <span data-ttu-id="bc72a-574">EF Core 5.0 极大地提升了这些转换操作的效率。</span><span class="sxs-lookup"><span data-stu-id="bc72a-574">EF Core 5.0 greatly improves the efficiency of these translations.</span></span>

<span data-ttu-id="bc72a-575">文档可通过问题 [#1612](https://github.com/dotnet/EntityFramework.Docs/issues/1612) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-575">Documentation is tracked by issue [#1612](https://github.com/dotnet/EntityFramework.Docs/issues/1612).</span></span>

### <a name="indexer-properties"></a><span data-ttu-id="bc72a-576">索引器属性</span><span class="sxs-lookup"><span data-stu-id="bc72a-576">Indexer properties</span></span>

<span data-ttu-id="bc72a-577">EF Core 5.0 支持 C# 索引器属性的映射。</span><span class="sxs-lookup"><span data-stu-id="bc72a-577">EF Core 5.0 supports mapping of C# indexer properties.</span></span> <span data-ttu-id="bc72a-578">这写属性使得实体能够充当属性包，实体中的列被映射为包中的命名属性。</span><span class="sxs-lookup"><span data-stu-id="bc72a-578">These properties allow entities to act as property bags where columns are mapped to named properties in the bag.</span></span>

<span data-ttu-id="bc72a-579">文档可通过问题 [#2018](https://github.com/dotnet/EntityFramework.Docs/issues/2018) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-579">Documentation is tracked by issue [#2018](https://github.com/dotnet/EntityFramework.Docs/issues/2018).</span></span>

### <a name="generation-of-check-constraints-for-enum-mappings"></a><span data-ttu-id="bc72a-580">为枚举映射生成检查约束</span><span class="sxs-lookup"><span data-stu-id="bc72a-580">Generation of check constraints for enum mappings</span></span>

<span data-ttu-id="bc72a-581">EF Core 5.0 迁移现可为枚举属性映射生成检查约束。</span><span class="sxs-lookup"><span data-stu-id="bc72a-581">EF Core 5.0 Migrations can now generate CHECK constraints for enum property mappings.</span></span> <span data-ttu-id="bc72a-582">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-582">For example:</span></span>

```sql
MyEnumColumn VARCHAR(10) NOT NULL CHECK (MyEnumColumn IN ('Useful', 'Useless', 'Unknown'))
```

<span data-ttu-id="bc72a-583">文档可通过问题 [#2082](https://github.com/dotnet/EntityFramework.Docs/issues/2082) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-583">Documentation is tracked by issue [#2082](https://github.com/dotnet/EntityFramework.Docs/issues/2082).</span></span>

### <a name="isrelational"></a><span data-ttu-id="bc72a-584">IsRelational</span><span class="sxs-lookup"><span data-stu-id="bc72a-584">IsRelational</span></span>

<span data-ttu-id="bc72a-585">除了现有 `IsSqlServer`、`IsSqlite` 和 `IsInMemory` 外，还添加了新的 `IsRelational` 方法。</span><span class="sxs-lookup"><span data-stu-id="bc72a-585">A new `IsRelational` method has been added in addition to the existing `IsSqlServer`, `IsSqlite`, and `IsInMemory`.</span></span> <span data-ttu-id="bc72a-586">此方法可用于测试 DbContext 是否使用任何关系数据库提供程序。</span><span class="sxs-lookup"><span data-stu-id="bc72a-586">This method can be used to test if the DbContext is using any relational database provider.</span></span> <span data-ttu-id="bc72a-587">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-587">For example:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    if (Database.IsRelational())
    {
        // Do relational-specific model configuration.
    }
}
```

<span data-ttu-id="bc72a-588">文档可通过问题 [#2185](https://github.com/dotnet/EntityFramework.Docs/issues/2185) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-588">Documentation is tracked by issue [#2185](https://github.com/dotnet/EntityFramework.Docs/issues/2185).</span></span>

### <a name="cosmos-optimistic-concurrency-with-etags"></a><span data-ttu-id="bc72a-589">使用 ETag 的 Cosmos 开放式并发</span><span class="sxs-lookup"><span data-stu-id="bc72a-589">Cosmos optimistic concurrency with ETags</span></span>

<span data-ttu-id="bc72a-590">Azure Cosmos DB 数据库提供程序现在支持使用 ETag 的开放式并发。</span><span class="sxs-lookup"><span data-stu-id="bc72a-590">The Azure Cosmos DB database provider now supports optimistic concurrency using ETags.</span></span> <span data-ttu-id="bc72a-591">使用 OnModelCreating 中的模型生成器配置 ETag：</span><span class="sxs-lookup"><span data-stu-id="bc72a-591">Use the model builder in OnModelCreating to configure an ETag:</span></span>

```csharp
builder.Entity<Customer>().Property(c => c.ETag).IsEtagConcurrency();
```

<span data-ttu-id="bc72a-592">然后，SaveChanges 将在并发冲突上引发 `DbUpdateConcurrencyException`，[可以处理](xref:core/saving/concurrency)它来实现重试等。</span><span class="sxs-lookup"><span data-stu-id="bc72a-592">SaveChanges will then throw an `DbUpdateConcurrencyException` on a concurrency conflict, which [can be handled](xref:core/saving/concurrency) to implement retries, etc.</span></span>

<span data-ttu-id="bc72a-593">文档可通过问题 [#2099](https://github.com/dotnet/EntityFramework.Docs/issues/2099) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-593">Documentation is tracked by issue [#2099](https://github.com/dotnet/EntityFramework.Docs/issues/2099).</span></span>

### <a name="query-translations-for-more-datetime-constructs"></a><span data-ttu-id="bc72a-594">查询转换以获取更多 DateTime 构造</span><span class="sxs-lookup"><span data-stu-id="bc72a-594">Query translations for more DateTime constructs</span></span>

<span data-ttu-id="bc72a-595">现在，包含新 DataTime 构造的查询会被转换。</span><span class="sxs-lookup"><span data-stu-id="bc72a-595">Queries containing new DateTime construction are now translated.</span></span>

<span data-ttu-id="bc72a-596">此外，现在映射了以下 SQL Server 函数：</span><span class="sxs-lookup"><span data-stu-id="bc72a-596">In addition, the following SQL Server functions are now mapped:</span></span>

* <span data-ttu-id="bc72a-597">DateDiffWeek</span><span class="sxs-lookup"><span data-stu-id="bc72a-597">DateDiffWeek</span></span>
* <span data-ttu-id="bc72a-598">DateFromParts</span><span class="sxs-lookup"><span data-stu-id="bc72a-598">DateFromParts</span></span>

<span data-ttu-id="bc72a-599">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-599">For example:</span></span>

```csharp
var count = context.Orders.Count(c => date > EF.Functions.DateFromParts(DateTime.Now.Year, 12, 25));

```

<span data-ttu-id="bc72a-600">文档可通过问题 [#2079](https://github.com/dotnet/EntityFramework.Docs/issues/2079) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-600">Documentation is tracked by issue [#2079](https://github.com/dotnet/EntityFramework.Docs/issues/2079).</span></span>

### <a name="query-translations-for-more-byte-array-constructs"></a><span data-ttu-id="bc72a-601">查询转换以获取更多字节数组构造</span><span class="sxs-lookup"><span data-stu-id="bc72a-601">Query translations for more byte array constructs</span></span>

<span data-ttu-id="bc72a-602">现在，使用 Contains、Length、SequenceEqual 等对 byte[] 属性进行的查询会转换成 SQL。</span><span class="sxs-lookup"><span data-stu-id="bc72a-602">Queries using Contains, Length, SequenceEqual, etc. on byte[] properties are now translated to SQL.</span></span>

<span data-ttu-id="bc72a-603">初步文档包含在 [2019 年 12 月 5 日发布的 EF 每周状态](https://github.com/dotnet/efcore/issues/15403#issuecomment-562332863)中。</span><span class="sxs-lookup"><span data-stu-id="bc72a-603">Preliminary documentation is included in the [EF weekly status for December 5, 2019](https://github.com/dotnet/efcore/issues/15403#issuecomment-562332863).</span></span>

<span data-ttu-id="bc72a-604">更多文档可通过问题 [#2079](https://github.com/dotnet/EntityFramework.Docs/issues/2079) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-604">Additional documentation is tracked by issue [#2079](https://github.com/dotnet/EntityFramework.Docs/issues/2079).</span></span>

### <a name="query-translation-for-reverse"></a><span data-ttu-id="bc72a-605">反向查询转换</span><span class="sxs-lookup"><span data-stu-id="bc72a-605">Query translation for Reverse</span></span>

<span data-ttu-id="bc72a-606">现在，使用 `Reverse` 的查询会被转换。</span><span class="sxs-lookup"><span data-stu-id="bc72a-606">Queries using `Reverse` are now translated.</span></span> <span data-ttu-id="bc72a-607">例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-607">For example:</span></span>

```csharp
context.Employees.OrderBy(e => e.EmployeeID).Reverse()
```

<span data-ttu-id="bc72a-608">文档可通过问题 [#2079](https://github.com/dotnet/EntityFramework.Docs/issues/2079) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-608">Documentation is tracked by issue [#2079](https://github.com/dotnet/EntityFramework.Docs/issues/2079).</span></span>

### <a name="query-translation-for-bitwise-operators"></a><span data-ttu-id="bc72a-609">按位运算符的查询转换</span><span class="sxs-lookup"><span data-stu-id="bc72a-609">Query translation for bitwise operators</span></span>

<span data-ttu-id="bc72a-610">现在，使用按位运算符的查询会在更多的情况下被转换，例如：</span><span class="sxs-lookup"><span data-stu-id="bc72a-610">Queries using bitwise operators are now translated in more cases For example:</span></span>

```csharp
context.Orders.Where(o => ~o.OrderID == negatedId)
```

<span data-ttu-id="bc72a-611">文档可通过问题 [#2079](https://github.com/dotnet/EntityFramework.Docs/issues/2079) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-611">Documentation is tracked by issue [#2079](https://github.com/dotnet/EntityFramework.Docs/issues/2079).</span></span>

### <a name="query-translation-for-strings-on-cosmos"></a><span data-ttu-id="bc72a-612">Cosmos 上的字符串的查询转换</span><span class="sxs-lookup"><span data-stu-id="bc72a-612">Query translation for strings on Cosmos</span></span>

<span data-ttu-id="bc72a-613">现在，在使用 Azure Cosmos DB 提供程序的情况下，使用字符串方法 Contains、StartsWith 和 EndsWith 的查询将被转换。</span><span class="sxs-lookup"><span data-stu-id="bc72a-613">Queries that use the string methods Contains, StartsWith, and EndsWith are now translated when using the Azure Cosmos DB provider.</span></span>

<span data-ttu-id="bc72a-614">文档可通过问题 [#2079](https://github.com/dotnet/EntityFramework.Docs/issues/2079) 进行跟踪。</span><span class="sxs-lookup"><span data-stu-id="bc72a-614">Documentation is tracked by issue [#2079](https://github.com/dotnet/EntityFramework.Docs/issues/2079).</span></span>
