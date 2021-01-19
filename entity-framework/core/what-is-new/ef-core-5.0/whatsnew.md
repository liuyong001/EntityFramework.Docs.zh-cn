---
title: EF Core 5.0 中的新增功能
description: EF Core 5.0 中的新功能概述
author: ajcvickers
ms.date: 09/10/2020
uid: core/what-is-new/ef-core-5.0/whatsnew
ms.openlocfilehash: 64b72ba2a6f752b9d71ea9b64ab08f4cf92ef03d
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129273"
---
# <a name="whats-new-in-ef-core-50"></a><span data-ttu-id="e4dfc-103">EF Core 5.0 中的新增功能</span><span class="sxs-lookup"><span data-stu-id="e4dfc-103">What's New in EF Core 5.0</span></span>

<span data-ttu-id="e4dfc-104">以下列表包括 EF Core 5.0 的主要新功能。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-104">The following list includes the major new features in EF Core 5.0.</span></span> <span data-ttu-id="e4dfc-105">有关发布中的问题的完整列表，请查看我们的[问题跟踪程序](https://github.com/dotnet/efcore/issues?q=is%3Aissue+milestone%3A5.0.0)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-105">For the full list of issues in the release, see our [issue tracker](https://github.com/dotnet/efcore/issues?q=is%3Aissue+milestone%3A5.0.0).</span></span>

<span data-ttu-id="e4dfc-106">EF Core 5.0 是一个主要版本，还包含多项[中断性变更](xref:core/what-is-new/ef-core-5.0/breaking-changes)，也就是可能会对现有应用程序产生负面影响的 API 改进或行为更改。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-106">As a major release, EF Core 5.0 also contains several [breaking changes](xref:core/what-is-new/ef-core-5.0/breaking-changes), which are API improvements or behavioral changes that may have negative impact on existing applications.</span></span>

## <a name="many-to-many"></a><span data-ttu-id="e4dfc-107">多对多</span><span class="sxs-lookup"><span data-stu-id="e4dfc-107">Many-to-many</span></span>

<span data-ttu-id="e4dfc-108">EF Core 5.0 支持多对多关系，而无需显式映射联接表。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-108">EF Core 5.0 supports many-to-many relationships without explicitly mapping the join table.</span></span>

<span data-ttu-id="e4dfc-109">以下面这些实体类型为例：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-109">For example, consider these entity types:</span></span>

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

<span data-ttu-id="e4dfc-110">EF Core 5.0 按照约定将其识别为多对多关系，并在数据库中自动创建一个 `PostTag` 联接表。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-110">EF Core 5.0 recognizes this as a many-to-many relationship by convention, and automatically creates a `PostTag` join table in the database.</span></span> <span data-ttu-id="e4dfc-111">无需显式引用联接表即可查询和更新数据，这大大简化了代码。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-111">Data can be queried and updated without explicitly referencing the join table, considerably simplifying code.</span></span> <span data-ttu-id="e4dfc-112">仍可自定义联接表，并在需要时显式查询它。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-112">The join table can still be customized and queried explicitly if needed.</span></span>

<span data-ttu-id="e4dfc-113">有关详细信息，请[参阅有关多对多的完整文档](xref:core/modeling/relationships#many-to-many)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-113">For further information, [see the full documentation on many-to-many](xref:core/modeling/relationships#many-to-many).</span></span>

## <a name="split-queries"></a><span data-ttu-id="e4dfc-114">拆分查询</span><span class="sxs-lookup"><span data-stu-id="e4dfc-114">Split queries</span></span>

<span data-ttu-id="e4dfc-115">从 EF Core 3.0 开始，EF Core 始终会为每个 LINQ 查询生成一个 SQL 查询。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-115">Starting with EF Core 3.0, EF Core always generates a single SQL query for each LINQ query.</span></span> <span data-ttu-id="e4dfc-116">这可确保在使用中的事务模式的约束内返回的数据保持一致。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-116">This ensures consistency of the data returned within the constraints of the transaction mode in use.</span></span> <span data-ttu-id="e4dfc-117">但是，当查询使用 `Include` 或投影来返回多个相关集合时，速度可能会变得非常慢。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-117">However, this can become very slow when the query uses `Include` or a projection to bring back multiple related collections.</span></span>

<span data-ttu-id="e4dfc-118">EF Core 5.0 现允许将包含相关集合的一个 LINQ 查询拆分成多个 SQL 查询。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-118">EF Core 5.0 now allows a single LINQ query including related collections to be split into multiple SQL queries.</span></span> <span data-ttu-id="e4dfc-119">这可能明显提升性能，但如果在两次查询之间数据发生了变化，则可能导致返回的结果不一致。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-119">This can significantly improve performance, but can result in inconsistency in the results returned if the data changes between the two queries.</span></span> <span data-ttu-id="e4dfc-120">你能使用可序列化的事务或快照事务来缓解这种情况并通过拆分查询实现一致性，但这可能会带来其他性能成本并导致行为差异。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-120">Serializable or snapshot transactions can be used to mitigate this and achieve consistency with split queries, but that may bring other performance costs and behavioral difference.</span></span>

<span data-ttu-id="e4dfc-121">例如，请考虑使用 `Include` 提取两个级别的相关集合的查询：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-121">For example, consider a query that pulls in two levels of related collections using `Include`:</span></span>

```csharp
var artists = context.Artists
    .Include(e => e.Albums)
    .ToList();
```

<span data-ttu-id="e4dfc-122">默认情况下，EF Core 将使用 SQLite 提供程序生成以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-122">By default, EF Core will generate the following SQL when using the SQLite provider:</span></span>

```sql
SELECT a."Id", a."Name", a0."Id", a0."ArtistId", a0."Title"
FROM "Artists" AS a
LEFT JOIN "Album" AS a0 ON a."Id" = a0."ArtistId"
ORDER BY a."Id", a0."Id"
```

<span data-ttu-id="e4dfc-123">使用拆分查询时，将改为生成以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-123">With split queries, the following SQL is generated instead:</span></span>

```sql
SELECT a."Id", a."Name"
FROM "Artists" AS a
ORDER BY a."Id"

SELECT a0."Id", a0."ArtistId", a0."Title", a."Id"
FROM "Artists" AS a
INNER JOIN "Album" AS a0 ON a."Id" = a0."ArtistId"
ORDER BY a."Id"
```

<span data-ttu-id="e4dfc-124">可通过将新的 `AsSplitQuery` 运算符放置在 LINQ 查询中的任何位置或全局放置在模型的 `OnConfiguring` 中来启用拆分查询。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-124">Split queries can be enabled by placing the new `AsSplitQuery` operator anywhere in your LINQ query, or globally in your model's `OnConfiguring`.</span></span> <span data-ttu-id="e4dfc-125">有关详细信息，请[参阅有关拆分查询的完整文档](xref:core/querying/single-split-queries)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-125">For further information, [see the full documentation on split queries](xref:core/querying/single-split-queries).</span></span>

## <a name="simple-logging-and-improved-diagnostics"></a><span data-ttu-id="e4dfc-126">简单的日志记录和改进的诊断</span><span class="sxs-lookup"><span data-stu-id="e4dfc-126">Simple logging and improved diagnostics</span></span>

<span data-ttu-id="e4dfc-127">EF Core 5.0 引入了一种通过新的 `LogTo` 方法来设置日志记录的简单方法。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-127">EF Core 5.0 introduces a simple way to set up logging via the new `LogTo` method.</span></span> <span data-ttu-id="e4dfc-128">以下内容会导致将日志记录消息写入控制台，包括 EF Core 生成的所有 SQL：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-128">The following will cause logging messages to be written to the console, including all SQL generated by EF Core:</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder.LogTo(Console.WriteLine);
```

<span data-ttu-id="e4dfc-129">此外，现可在任何 LINQ 查询上调用 `ToQueryString`，以检索该查询将执行的 SQL：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-129">In addition, it is now possible to call `ToQueryString` on any LINQ query, retrieving the SQL that the query would execute:</span></span>

```csharp
Console.WriteLine(
    ctx.Artists
    .Where(a => a.Name == "Pink Floyd")
    .ToQueryString());
```

<span data-ttu-id="e4dfc-130">最后，各种 EF Core 类型都配备了增强的 `DebugView` 属性，它针对内部项提供详细视图。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-130">Finally, various EF Core types have been fitted with an enhanced `DebugView` property which provides a detailed view into the internals.</span></span> <span data-ttu-id="e4dfc-131">例如，可参考 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DebugView%2A?displayProperty=nameWithType> 来查看在给定时间正在被跟踪的确切实体。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-131">For example, <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DebugView%2A?displayProperty=nameWithType> can be consulted to see exactly which entities are being tracked in a given moment.</span></span>

<span data-ttu-id="e4dfc-132">有关详细信息，请[参阅有关日志记录和拦截的文档](xref:core/logging-events-diagnostics/index)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-132">For further information, [see the documentation on logging and interception](xref:core/logging-events-diagnostics/index).</span></span>

## <a name="filtered-include"></a><span data-ttu-id="e4dfc-133">经过筛选的包含</span><span class="sxs-lookup"><span data-stu-id="e4dfc-133">Filtered include</span></span>

<span data-ttu-id="e4dfc-134">`Include` 方法现支持筛选包含的实体：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-134">The `Include` method now supports filtering of the entities included:</span></span>

```csharp
var blogs = context.Blogs
    .Include(e => e.Posts.Where(p => p.Title.Contains("Cheese")))
    .ToList();
```

<span data-ttu-id="e4dfc-135">此查询将一并返回包含每个关联文章的博客（仅当文章标题包含“Cheese”时）。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-135">This query will return blogs together with each associated post, but only when the post title contains "Cheese".</span></span>

<span data-ttu-id="e4dfc-136">有关详细信息，请[参阅有关拆分查询的完整文档](xref:core/querying/related-data/eager#filtered-include)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-136">For further information, [see the full documentation on split queries](xref:core/querying/related-data/eager#filtered-include).</span></span>

## <a name="table-per-type-tpt-mapping"></a><span data-ttu-id="e4dfc-137">每个类型一张表 (TPT) 映射</span><span class="sxs-lookup"><span data-stu-id="e4dfc-137">Table-per-type (TPT) mapping</span></span>

<span data-ttu-id="e4dfc-138">默认情况下，EF Core 会将 .NET 类型的继承层次结构映射到单个数据库表。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-138">By default, EF Core maps an inheritance hierarchy of .NET types to a single database table.</span></span> <span data-ttu-id="e4dfc-139">这称为每个层次结构一张表 (TPH) 映射。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-139">This is known as table-per-hierarchy (TPH) mapping.</span></span> <span data-ttu-id="e4dfc-140">EF Core 5.0 还允许将继承层次结构中的每个 .NET 类型映射到另一个数据库表，这称为每个类型一张表 (TPT) 映射。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-140">EF Core 5.0 also allows mapping each .NET type in an inheritance hierarchy to a different database table; known as table-per-type (TPT) mapping.</span></span>

<span data-ttu-id="e4dfc-141">例如，请考虑具有映射的层次结构的此模型：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-141">For example, consider this model with a mapped hierarchy:</span></span>

```csharp
public class Animal
{
    public int Id { get; set; }
    public string Name { get; set; }
}

public class Cat : Animal
{
    public string EducationLevel { get; set; }
}

public class Dog : Animal
{
    public string FavoriteToy { get; set; }
}
```

<span data-ttu-id="e4dfc-142">使用 TPT 时，将为层次结构中的每种类型创建一个数据库表：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-142">With TPT, a database table is created for each type in the hierarchy:</span></span>

```sql
CREATE TABLE [Animals] (
    [Id] int NOT NULL IDENTITY,
    [Name] nvarchar(max) NULL,
    CONSTRAINT [PK_Animals] PRIMARY KEY ([Id])
);

CREATE TABLE [Cats] (
    [Id] int NOT NULL,
    [EdcuationLevel] nvarchar(max) NULL,
    CONSTRAINT [PK_Cats] PRIMARY KEY ([Id]),
    CONSTRAINT [FK_Cats_Animals_Id] FOREIGN KEY ([Id]) REFERENCES [Animals] ([Id]) ON DELETE NO ACTION,
);

CREATE TABLE [Dogs] (
    [Id] int NOT NULL,
    [FavoriteToy] nvarchar(max) NULL,
    CONSTRAINT [PK_Dogs] PRIMARY KEY ([Id]),
    CONSTRAINT [FK_Dogs_Animals_Id] FOREIGN KEY ([Id]) REFERENCES [Animals] ([Id]) ON DELETE NO ACTION,
);
```

<span data-ttu-id="e4dfc-143">有关详细信息，请[参阅有关 TPT 的完整文档](xref:core/modeling/inheritance)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-143">For further information, [see the full documentation on TPT](xref:core/modeling/inheritance).</span></span>

## <a name="flexible-entity-mapping"></a><span data-ttu-id="e4dfc-144">灵活实体映射</span><span class="sxs-lookup"><span data-stu-id="e4dfc-144">Flexible entity mapping</span></span>

<span data-ttu-id="e4dfc-145">实体类型通常映射到表或视图，以便 EF Core 在查询该类型时将拉回表或视图的内容。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-145">Entity types are commonly mapped to tables or views such that EF Core will pull back the contents of the table or view when querying for that type.</span></span> <span data-ttu-id="e4dfc-146">EF Core 5.0 添加了其他映射选项，其中可将实体映射到 SQL 查询（称为“定义查询”）或表值函数 (TVF)：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-146">EF Core 5.0 adds additional mapping options, where an entity can be mapped to a SQL query (called a "defining query"), or to a table-valued function (TVF):</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Post>().ToSqlQuery(
        @"SELECT Id, Name, Category, BlogId FROM posts
          UNION ALL
          SELECT Id, Name, ""Legacy"", BlogId from legacy_posts");

    modelBuilder.Entity<Blog>().ToFunction("BlogsReturningFunction");
}
```

<span data-ttu-id="e4dfc-147">表值函数也可映射到 .NET 方法而不是 DbSet，从而允许传递参数。可使用 <xref:Microsoft.EntityFrameworkCore.RelationalModelBuilderExtensions.HasDbFunction%2A> 设置映射。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-147">Table-valued functions can also be mapped to a .NET method rather than to a DbSet, allowing parameters to be passed; the mapping can be set up with <xref:Microsoft.EntityFrameworkCore.RelationalModelBuilderExtensions.HasDbFunction%2A>.</span></span>

<span data-ttu-id="e4dfc-148">最后，现可在查询时将实体映射到视图（或映射到函数或定义查询中），而在更新时将实体映射到表：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-148">Finally, it is now possible to map an entity to a view when querying (or to a function or defining query), but to a table when updating:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<Blog>()
        .ToTable("Blogs")
        .ToView("BlogsView");
}
```

## <a name="shared-type-entity-types-and-property-bags"></a><span data-ttu-id="e4dfc-149">共享型实体类型和属性包</span><span class="sxs-lookup"><span data-stu-id="e4dfc-149">Shared-type entity types and property bags</span></span>

<span data-ttu-id="e4dfc-150">EF Core 5.0 允许将相同的 CLR 类型映射到多个不同的实体类型，这种类型称为共享型实体类型。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-150">EF Core 5.0 allows the same CLR type to be mapped to multiple different entity types; such types are known as shared-type entity types.</span></span> <span data-ttu-id="e4dfc-151">尽管任何 CLR 类型都可用于此功能，但 .NET `Dictionary` 提供了一个特别引人注目的用例，我们称其为“属性包”：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-151">While any CLR type can be used with this feature, .NET `Dictionary` offers a particularly compelling use-case which we call "property bags":</span></span>

```csharp
public class ProductsContext : DbContext
{
    public DbSet<Dictionary<string, object>> Products => Set<Dictionary<string, object>>("Product");

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.SharedTypeEntity<Dictionary<string, object>>("Product", b =>
        {
            b.IndexerProperty<int>("Id");
            b.IndexerProperty<string>("Name").IsRequired();
            b.IndexerProperty<decimal>("Price");
        });
    }
}
```

<span data-ttu-id="e4dfc-152">随后，这些实体可像普通实体类型一样使用它们自己的专用 CLR 类型来进行查询和更新。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-152">These entities can then be queried and updated just like normal entity types with their own, dedicated CLR type.</span></span> <span data-ttu-id="e4dfc-153">有关详细信息，可查看关于[属性包](xref:core/modeling/shadow-properties)的文档。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-153">More information can be found in the documentation on [property bags](xref:core/modeling/shadow-properties).</span></span>

## <a name="required-11-dependents"></a><span data-ttu-id="e4dfc-154">必需的一对一依赖项</span><span class="sxs-lookup"><span data-stu-id="e4dfc-154">Required 1:1 dependents</span></span>

<span data-ttu-id="e4dfc-155">在 EF Core 3.1 中，一对一关系的依赖端始终被视为可选。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-155">In EF Core 3.1, the dependent end of a one-to-one relationship was always considered optional.</span></span> <span data-ttu-id="e4dfc-156">使用拥有的实体时，这一点最明显，因为拥有的实体的所有列在数据库中都被创建为可为 null，即使它们已按模型中的要求进行配置也是这样。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-156">This was most apparent when using owned entities, as all the owned entity's column were created as nullable in the database, even if they were configured as required in the model.</span></span>

<span data-ttu-id="e4dfc-157">在 EF Core 5.0 中，可将到拥有的实体的导航配置为必需依赖项。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-157">In EF Core 5.0, a navigation to an owned entity can be configured as as a required dependent.</span></span> <span data-ttu-id="e4dfc-158">例如：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-158">For example:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Person>(b =>
    {
        b.OwnsOne(e => e.HomeAddress,
            b =>
            {
                b.Property(e => e.City).IsRequired();
                b.Property(e => e.Postcode).IsRequired();
            });
        b.Navigation(e => e.HomeAddress).IsRequired();
    });
}
```

## <a name="dbcontextfactory"></a><span data-ttu-id="e4dfc-159">DbContextFactory</span><span class="sxs-lookup"><span data-stu-id="e4dfc-159">DbContextFactory</span></span>

<span data-ttu-id="e4dfc-160">EF Core 5.0 引入了 `AddDbContextFactory` 和 `AddPooledDbContextFactory` 来注册工厂，以便在应用程序的依赖项注入 (D.I.) 容器中创建 DbContext 实例；当应用程序代码需要手动创建和处理上下文实例时，这很有用。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-160">EF Core 5.0 introduces `AddDbContextFactory` and `AddPooledDbContextFactory` to register a factory for creating DbContext instances in the application's dependency injection (D.I.) container; this can be useful when application code needs to create and dispose context instances manually.</span></span>

```csharp
services.AddDbContextFactory<SomeDbContext>(b =>
    b.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test"));
```

<span data-ttu-id="e4dfc-161">此时，可使用 `IDbContextFactory<TContext>` 注入 ASP.NET Core 控制器等应用程序服务，并用它来实例化上下文实例：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-161">At this point, application services such as ASP.NET Core controllers can then be injected with `IDbContextFactory<TContext>`, and use it to instantiate context instances:</span></span>

```csharp
public class MyController
{
    private readonly IDbContextFactory<SomeDbContext> _contextFactory;

    public MyController(IDbContextFactory<SomeDbContext> contextFactory)
        => _contextFactory = contextFactory;

    public void DoSomeThing()
    {
        using (var context = _contextFactory.CreateDbContext())
        {
            // ...
        }
    }
}
```

<span data-ttu-id="e4dfc-162">有关详细信息，请[参阅有关 DbContextFactory 的完整文档](xref:core/dbcontext-configuration/index#using-a-dbcontext-factory-eg-for-blazor)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-162">For further information, [see the full documentation on DbContextFactory](xref:core/dbcontext-configuration/index#using-a-dbcontext-factory-eg-for-blazor).</span></span>

## <a name="sqlite-table-rebuilds"></a><span data-ttu-id="e4dfc-163">重新生成 SQLite 表</span><span class="sxs-lookup"><span data-stu-id="e4dfc-163">SQLite table rebuilds</span></span>

<span data-ttu-id="e4dfc-164">与其他数据库相比，SQLite 的架构操作能力相对有限；例如，不支持从现有表中删除列。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-164">Compared to other databases, SQLite is relatively limited in its schema manipulation capabilities; for example, dropping a column from an existing table is not supported.</span></span> <span data-ttu-id="e4dfc-165">EF Core 5.0 解决这些限制的方式是自动创建一个新表、从旧表复制数据、删除旧表并重命名新表。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-165">EF Core 5.0 works around these limitations by automatically creating a new table, copying the data from the old table, dropping the old table and renaming the new one.</span></span> <span data-ttu-id="e4dfc-166">这会“重新生成”表，而且可安全地应用之前不受支持的迁移操作。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-166">This "rebuilds" the table, and allows previously unsupported migration operations to be safely applied.</span></span>

<span data-ttu-id="e4dfc-167">若要详细了解现在通过重新生成表而支持的具体迁移操作，请[参阅此文档页面](xref:core/providers/sqlite/limitations#migrations-limitations)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-167">For details on which migration operations are now supported via table rebuilds, [see this documentation page](xref:core/providers/sqlite/limitations#migrations-limitations).</span></span>

## <a name="database-collations"></a><span data-ttu-id="e4dfc-168">数据库排序规则</span><span class="sxs-lookup"><span data-stu-id="e4dfc-168">Database collations</span></span>

<span data-ttu-id="e4dfc-169">EF Core 5.0 现支持在数据库、列或查询级别指定文本排序规则。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-169">EF Core 5.0 introduces support for specifying text collations at the database, column or query level.</span></span> <span data-ttu-id="e4dfc-170">这样就可以灵活但不影响查询性能的方式配置大小写区分和其他文本方面。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-170">This allows case sensitivity and other textual aspects to be configured in a way that is both flexible and does not compromise query performance.</span></span>

<span data-ttu-id="e4dfc-171">例如，以下内容将 `Name` 列配置为在 SQL Server 上区分大小写，并且在该列上创建的任何索引都将相应地起作用：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-171">For example, the following will configure the `Name` column to be case-sensitive on SQL Server, and any indexes created on the column will function accordingly:</span></span>

```csharp
modelBuilder
    .Entity<User>()
    .Property(e => e.Name)
    .UseCollation("SQL_Latin1_General_CP1_CS_AS");
```

<span data-ttu-id="e4dfc-172">有关详细信息，请[参阅有关排序规则和大小写区分的完整文档](xref:core/miscellaneous/collations-and-case-sensitivity)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-172">For further information, [see the full documentation on collations and case sensitivity](xref:core/miscellaneous/collations-and-case-sensitivity).</span></span>

## <a name="event-counters"></a><span data-ttu-id="e4dfc-173">事件计数器</span><span class="sxs-lookup"><span data-stu-id="e4dfc-173">Event counters</span></span>

<span data-ttu-id="e4dfc-174">EF Core 5.0 公开了[事件计数器](https://devblogs.microsoft.com/dotnet/introducing-diagnostics-improvements-in-net-core-3-0)，它可用于跟踪应用程序的性能并发现各种异常情况。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-174">EF Core 5.0 exposes [event counters](https://devblogs.microsoft.com/dotnet/introducing-diagnostics-improvements-in-net-core-3-0) which can be used to track your application's performance and spot various anomalies.</span></span> <span data-ttu-id="e4dfc-175">只需使用 [dotnet-counters](https://docs.microsoft.com/dotnet/core/diagnostics/dotnet-counters) 工具将其附加到运行 EF 的进程即可：</span><span class="sxs-lookup"><span data-stu-id="e4dfc-175">Simply attach to a process running EF with the [dotnet-counters](https://docs.microsoft.com/dotnet/core/diagnostics/dotnet-counters) tool:</span></span>

```console
> dotnet counters monitor Microsoft.EntityFrameworkCore -p 49496

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

<span data-ttu-id="e4dfc-176">有关详细信息，请[参阅有关事件计数器的完整文档](xref:core/logging-events-diagnostics/event-counters)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-176">For further information, [see the full documentation on event counters](xref:core/logging-events-diagnostics/event-counters).</span></span>

## <a name="other-features"></a><span data-ttu-id="e4dfc-177">其他功能</span><span class="sxs-lookup"><span data-stu-id="e4dfc-177">Other features</span></span>

### <a name="model-building"></a><span data-ttu-id="e4dfc-178">模型构建</span><span class="sxs-lookup"><span data-stu-id="e4dfc-178">Model building</span></span>

* <span data-ttu-id="e4dfc-179">为了更轻松地配置[值比较器](xref:core/modeling/value-comparers)，引入了模型构建 API。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-179">Model building APIs have been introduced for easier configuration of [value comparers](xref:core/modeling/value-comparers).</span></span>
* <span data-ttu-id="e4dfc-180">现可将计算列配置为[存储或虚拟](xref:core/modeling/generated-properties#computed-columns) 。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-180">Computed columns can now be configured as [*stored* or *virtual*](xref:core/modeling/generated-properties#computed-columns).</span></span>
* <span data-ttu-id="e4dfc-181">现可[通过 Fluent API](xref:core/modeling/entity-properties#precision-and-scale) 配置精度和规模。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-181">Precision and scale can now be configured [via the Fluent API](xref:core/modeling/entity-properties#precision-and-scale).</span></span>
* <span data-ttu-id="e4dfc-182">针对[导航属性](xref:core/modeling/relationships#configuring-navigation-properties)引入了新的模型构建 API。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-182">New model building APIs have been introduced for [navigation properties](xref:core/modeling/relationships#configuring-navigation-properties).</span></span>
* <span data-ttu-id="e4dfc-183">与属性类似，针对字段引入了新的模型构建 API。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-183">New model building APIs have been introduced for fields, similar to properties.</span></span>
* <span data-ttu-id="e4dfc-184">现可将 .NET [PhysicalAddress](/dotnet/api/system.net.networkinformation.physicaladdress) 和 [IPAddress](/dotnet/api/system.net.ipaddress) 类型映射到数据库字符串列。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-184">The .NET [PhysicalAddress](/dotnet/api/system.net.networkinformation.physicaladdress) and [IPAddress](/dotnet/api/system.net.ipaddress) types can now be mapped to database string columns.</span></span>
* <span data-ttu-id="e4dfc-185">现可通过[新的 `[BackingField]` 属性](xref:core/modeling/backing-field)来配置支持字段。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-185">A backing field can now be configured via [the new `[BackingField]` attribute](xref:core/modeling/backing-field).</span></span>
* <span data-ttu-id="e4dfc-186">现可使用可为 null 的支持字段，在 CLR 默认值不是一个好的 sentinel 值（`bool` 值得注意）的情况下，这更好地支持了商店生成的默认值。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-186">Nullable backing fields are now allowed, providing better support for store-generated defaults where the CLR default isn't a good sentinel value (notable `bool`).</span></span>
* <span data-ttu-id="e4dfc-187">可在实体类型上使用新的 `[Index]` 属性来指定索引，而不是使用 Fluent API。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-187">A new `[Index]` attribute can be used on an entity type to specify an index, instead of using the Fluent API.</span></span>
* <span data-ttu-id="e4dfc-188">可使用新的 `[Keyless]` 属性将实体类型配置为[没有键](xref:core/modeling/keyless-entity-types)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-188">A new `[Keyless]` attribute can be used to configure an entity type [as having no key](xref:core/modeling/keyless-entity-types).</span></span>
* <span data-ttu-id="e4dfc-189">默认情况下，[EF Core 现将鉴别器视为完整](xref:core/modeling/inheritance#table-per-hierarchy-and-discriminator-configuration)，这意味着它预期永远不会看到模型中应用程序未配置的鉴别器值。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-189">By default, [EF Core now regards discriminators as *complete*](xref:core/modeling/inheritance#table-per-hierarchy-and-discriminator-configuration), meaning that it expects to never see discriminator values not configured by the application in the model.</span></span> <span data-ttu-id="e4dfc-190">这样可一定程度地提高性能；如果鉴别器列可能包含未知值，可将其禁用。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-190">This allows for some performance improvements, and can be disabled if your discriminator column might hold unknown values.</span></span>

### <a name="query"></a><span data-ttu-id="e4dfc-191">查询</span><span class="sxs-lookup"><span data-stu-id="e4dfc-191">Query</span></span>

* <span data-ttu-id="e4dfc-192">查询转换失败异常现包含更明确的失败原因，以帮助查明问题。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-192">Query translation failure exceptions now contain more explicit reasons about the reasons for the failure, to help pinpoint the issue.</span></span>
* <span data-ttu-id="e4dfc-193">非跟踪查询现可执行[标识解析](xref:core/querying/tracking#identity-resolution)，从而避免为同一数据库对象返回多个实体实例。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-193">No-tracking queries can now perform [identity resolution](xref:core/querying/tracking#identity-resolution), avoiding multiple entity instances being returned for the same database object.</span></span>
* <span data-ttu-id="e4dfc-194">现支持使用条件聚合进行 GroupBy 操作（例如 `GroupBy(o => o.OrderDate).Select(g => g.Count(i => i.OrderDate != null))`）。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-194">Added support for GroupBy with conditional aggregates (e.g. `GroupBy(o => o.OrderDate).Select(g => g.Count(i => i.OrderDate != null))`).</span></span>
* <span data-ttu-id="e4dfc-195">现支持在聚合之前针对组元素转换 Distinct 运算符。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-195">Added support for translating the Distinct operator over group elements before aggregate.</span></span>
* <span data-ttu-id="e4dfc-196">对 [`Reverse`](/dotnet/api/system.linq.queryable.reverse) 的转换。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-196">Translation of [`Reverse`](/dotnet/api/system.linq.queryable.reverse).</span></span>
* <span data-ttu-id="e4dfc-197">针对 SQL Server 改进了对 `DateTime` 的转换（例如 [`DateDiffWeek`](xref:Microsoft.EntityFrameworkCore.SqlServerDbFunctionsExtensions.DateDiffWeek%2A)、[`DateFromParts`](xref:Microsoft.EntityFrameworkCore.SqlServerDbFunctionsExtensions.DateFromParts%2A)）。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-197">Improved translation around `DateTime` for SQL Server (e.g. [`DateDiffWeek`](xref:Microsoft.EntityFrameworkCore.SqlServerDbFunctionsExtensions.DateDiffWeek%2A), [`DateFromParts`](xref:Microsoft.EntityFrameworkCore.SqlServerDbFunctionsExtensions.DateFromParts%2A)).</span></span>
* <span data-ttu-id="e4dfc-198">字节数组上对新方法的转换（例如 [`Contains`](/dotnet/api/system.linq.enumerable.contains)、[`Length`](/dotnet/api/system.array.length)、[`SequenceEqual`](/dotnet/api/system.linq.enumerable.sequenceequal)）。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-198">Translation of new methods on byte arrays (e.g. [`Contains`](/dotnet/api/system.linq.enumerable.contains), [`Length`](/dotnet/api/system.array.length), [`SequenceEqual`](/dotnet/api/system.linq.enumerable.sequenceequal)).</span></span>
* <span data-ttu-id="e4dfc-199">对一些其他按位运算符的转换，例如二进制补码。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-199">Translation of some additional bitwise operators, such as two's complement.</span></span>
* <span data-ttu-id="e4dfc-200">针对字符串上 `FirstOrDefault` 的转换。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-200">Translation of `FirstOrDefault` over strings.</span></span>
* <span data-ttu-id="e4dfc-201">围绕 NULL 语义改进了查询转换，使查询更紧密、更高效。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-201">Improved query translation around null semantics, resulting in tighter, more efficient queries.</span></span>
* <span data-ttu-id="e4dfc-202">现可对用户映射的函数进行批注以控制 Null 传播，这又使得查询更紧密、更高效。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-202">User-mapped functions can now be annotated to control null propagation, again resulting in tighter, more efficient queries.</span></span>
* <span data-ttu-id="e4dfc-203">包含 CASE 块的 SQL 现在明显简洁得多。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-203">SQL containing CASE blocks is now considerably more concise.</span></span>
* <span data-ttu-id="e4dfc-204">现可通过新的 [`EF.Functions.DataLength`](xref:Microsoft.EntityFrameworkCore.SqlServerDbFunctionsExtensions.DataLength%2A) 方法在查询中调用 SQL Server [`DATELENGTH`](https://docs.microsoft.com/sql/t-sql/functions/datalength-transact-sql) 函数。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-204">The SQL Server [`DATELENGTH`](https://docs.microsoft.com/sql/t-sql/functions/datalength-transact-sql) function can now be called in queries via the new [`EF.Functions.DataLength`](xref:Microsoft.EntityFrameworkCore.SqlServerDbFunctionsExtensions.DataLength%2A) method.</span></span>
* <span data-ttu-id="e4dfc-205">`EnableDetailedErrors` 将[ 其他详细信息添加到异常](xref:core/logging-events-diagnostics/simple-logging#detailed-query-exceptions)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-205">`EnableDetailedErrors` adds [additional details to exceptions](xref:core/logging-events-diagnostics/simple-logging#detailed-query-exceptions).</span></span>

### <a name="saving"></a><span data-ttu-id="e4dfc-206">保存</span><span class="sxs-lookup"><span data-stu-id="e4dfc-206">Saving</span></span>

* <span data-ttu-id="e4dfc-207">SaveChanges [拦截](xref:core/logging-events-diagnostics/interceptors#savechanges-interception)和[事件](xref:core/logging-events-diagnostics/events)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-207">SaveChanges [interception](xref:core/logging-events-diagnostics/interceptors#savechanges-interception) and [events](xref:core/logging-events-diagnostics/events).</span></span>
* <span data-ttu-id="e4dfc-208">引入了用于控制[事务保存点](xref:core/saving/transactions#savepoints)的 API。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-208">APIs have been introduced for controlling [transaction savepoints](xref:core/saving/transactions#savepoints).</span></span> <span data-ttu-id="e4dfc-209">此外，当调用 `SaveChanges` 并且事务已经在进行时，EF Core 将自动创建一个保存点，并在失败时回滚到该保存点。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-209">In addition, EF Core will automatically create a savepoint when `SaveChanges` is called and a transaction is already in progress, and roll back to it in case of failure.</span></span>
* <span data-ttu-id="e4dfc-210">事务 ID 可由应用程序显式设置，从而可更轻松地在日志记录和其他位置关联事务事件。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-210">A transaction ID can be explicitly set by the application, allowing for easier correlation of transaction events in logging and elsewhere.</span></span>
* <span data-ttu-id="e4dfc-211">根据对批处理性能的分析，SQL Server 的默认最大批处理大小已更改为 42。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-211">The default maximum batch size for SQL Server has been changed to 42 based on an analysis of batching performance.</span></span>

### <a name="migrations-and-scaffolding"></a><span data-ttu-id="e4dfc-212">迁移和基架</span><span class="sxs-lookup"><span data-stu-id="e4dfc-212">Migrations and scaffolding</span></span>

* <span data-ttu-id="e4dfc-213">现可[从迁移中排除](xref:core/modeling/entity-types#excluding-from-migrations)表。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-213">Tables can now be [excluded from migrations](xref:core/modeling/entity-types#excluding-from-migrations).</span></span>
* <span data-ttu-id="e4dfc-214">新的 [`dotnet ef migrations list`](xref:core/cli/dotnet#dotnet-ef-migrations-list) 命令现将显示哪些迁移尚未应用于数据库（[`Get-Migration`](xref:core/cli/powershell#get-migration) 在包管理控制台中执行相同的操作）。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-214">A new [`dotnet ef migrations list`](xref:core/cli/dotnet#dotnet-ef-migrations-list) command now shows which migrations have not yet been applied to the database ([`Get-Migration`](xref:core/cli/powershell#get-migration) does the same in the Package Management Console).</span></span>
* <span data-ttu-id="e4dfc-215">迁移脚本现在适当的位置包含事务语句，以改善迁移应用程序失败时的处理情况。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-215">Migrations scripts now contain transaction statements where appropriate to improve handling cases where migration application fails.</span></span>
* <span data-ttu-id="e4dfc-216">未映射基类的列现在按映射实体类型的其他列排序。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-216">The columns for unmapped base classes are now ordered after other columns for mapped entity types.</span></span> <span data-ttu-id="e4dfc-217">请注意，这仅影响新创建的表。 现有表的列顺序保持不变。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-217">Note this only impacts newly created tables; the column order for existing tables remains unchanged.</span></span>
* <span data-ttu-id="e4dfc-218">迁移生成现可知道正在生成的迁移是否是幂等的，以及输出是立即执行还是生成用作脚本。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-218">Migration generation can now be aware if the migration being generated is idempotent, and whether the output will be executed immediately or generated as a script.</span></span>
* <span data-ttu-id="e4dfc-219">添加了新的命令行参数，用于在[迁移](xref:core/managing-schemas/migrations/index#namespaces)和[基架](xref:core/managing-schemas/scaffolding#directories-and-namespaces)中指定命名空间。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-219">New command-line parameters have been added for specifying namespaces in [Migrations](xref:core/managing-schemas/migrations/index#namespaces) and [scaffolding](xref:core/managing-schemas/scaffolding#directories-and-namespaces).</span></span>
* <span data-ttu-id="e4dfc-220">[dotnet ef database update](xref:core/cli/dotnet#dotnet-ef-database-update) 命令现接受一个新的 `--connection` 参数来指定连接字符串。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-220">The [dotnet ef database update](xref:core/cli/dotnet#dotnet-ef-database-update) command now accepts a new `--connection` parameter for specifying the connection string.</span></span>
* <span data-ttu-id="e4dfc-221">现在，对现有数据库搭建基架会将表名单数化，因此名为 `People` 和 `Addresses` 的表将搭建基架成为称作 `Person` 和 `Address` 的实体类型。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-221">Scaffolding existing databases now singularizes table names, so tables named `People` and `Addresses` will be scaffolded to entity types called `Person` and `Address`.</span></span> <span data-ttu-id="e4dfc-222">[仍可保留原始数据库名称](xref:core/managing-schemas/scaffolding#preserving-names)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-222">[Original database names can still be preserved](xref:core/managing-schemas/scaffolding#preserving-names).</span></span>
* <span data-ttu-id="e4dfc-223">新的 [`--no-onconfiguring`](xref:core/cli/dotnet#dotnet-ef-dbcontext-scaffold) 选项可指示 EF Core 在搭建模型基架时排除 `OnModelConfiguring`。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-223">The new [`--no-onconfiguring`](xref:core/cli/dotnet#dotnet-ef-dbcontext-scaffold) option can instruct EF Core to exclude `OnModelConfiguring` when scaffolding a model.</span></span>

### <a name="cosmos"></a><span data-ttu-id="e4dfc-224">Cosmos</span><span class="sxs-lookup"><span data-stu-id="e4dfc-224">Cosmos</span></span>

* <span data-ttu-id="e4dfc-225">[Cosmos 连接设置](xref:core/providers/cosmos/index#cosmos-options)已扩展。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-225">[Cosmos connection settings](xref:core/providers/cosmos/index#cosmos-options) have been expanded.</span></span>
* <span data-ttu-id="e4dfc-226">现支持在 Cosmos 上[使用 ETag](xref:core/providers/cosmos/index#optimistic-concurrency-with-etags) 来进行乐观并发。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-226">Optimistic concurrency is now [supported on Cosmos via the use of ETags](xref:core/providers/cosmos/index#optimistic-concurrency-with-etags).</span></span>
* <span data-ttu-id="e4dfc-227">新的 `WithPartitionKey` 方法使 Cosmos [分区键](xref:core/providers/cosmos/index#partition-keys)既可包含在模型中，也可包含在查询中。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-227">The new `WithPartitionKey` method allows the Cosmos [partition key](xref:core/providers/cosmos/index#partition-keys) to be included both in the model and in queries.</span></span>
* <span data-ttu-id="e4dfc-228">现针对 Cosmos 转换了字符串方法 [`Contains`](/dotnet/api/system.string.contains)、[`StartsWith`](/dotnet/api/system.string.startswith) 和 [`EndsWith`](/dotnet/api/system.string.endswith)。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-228">The string methods [`Contains`](/dotnet/api/system.string.contains), [`StartsWith`](/dotnet/api/system.string.startswith) and [`EndsWith`](/dotnet/api/system.string.endswith) are now translated for Cosmos.</span></span>
* <span data-ttu-id="e4dfc-229">现在 Cosmos 上转换了 C# `is` 运算符。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-229">The C# `is` operator is now translated on Cosmos.</span></span>

### <a name="sqlite"></a><span data-ttu-id="e4dfc-230">Sqlite</span><span class="sxs-lookup"><span data-stu-id="e4dfc-230">Sqlite</span></span>

* <span data-ttu-id="e4dfc-231">现支持计算列。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-231">Computed columns are now supported.</span></span>
* <span data-ttu-id="e4dfc-232">现在，利用 SqliteBlob 和流，可以更高效地使用 GetBytes、GetChars 和 GetTextReader 检索二进制和字符串数据。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-232">Retrieving binary and string data with GetBytes, GetChars, and GetTextReader is now more efficient by making use of SqliteBlob and streams.</span></span>
* <span data-ttu-id="e4dfc-233">SqliteConnection 的初始化现在很慢。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-233">Initialization of SqliteConnection is now lazy.</span></span>

### <a name="other"></a><span data-ttu-id="e4dfc-234">其他</span><span class="sxs-lookup"><span data-stu-id="e4dfc-234">Other</span></span>

* <span data-ttu-id="e4dfc-235">可生成自动执行 [INotifyPropertyChanging](/dotnet/api/system.componentmodel.inotifypropertychanging) 和 [INotifyPropertyChanged](/dotnet/api/system.componentmodel.inotifypropertychanged) 的更改跟踪代理。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-235">Change-tracking proxies can be generated that automatically implement [INotifyPropertyChanging](/dotnet/api/system.componentmodel.inotifypropertychanging) and [INotifyPropertyChanged](/dotnet/api/system.componentmodel.inotifypropertychanged).</span></span> <span data-ttu-id="e4dfc-236">对于调用 `SaveChanges` 时不扫描更改的更改跟踪而言，这提供了一种替代方法。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-236">This provides an alternative approach to change-tracking that doesn't scan for changes when `SaveChanges` is called.</span></span>
* <span data-ttu-id="e4dfc-237">现可在已初始化的 DbContext 上更改 <xref:System.Data.Common.DbConnection> 或连接字符串。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-237">A <xref:System.Data.Common.DbConnection> or connection string can now be changed on an already-initialized DbContext.</span></span>
* <span data-ttu-id="e4dfc-238">新的 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Clear%2A?displayProperty=nameWithType>.0#Microsoft_EntityFrameworkCore_ChangeTracking_ChangeTracker_Clear) 方法会清除所有跟踪实体的 DbContext。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-238">The new <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Clear%2A?displayProperty=nameWithType>.0#Microsoft_EntityFrameworkCore_ChangeTracking_ChangeTracker_Clear) method clears the DbContext of all tracked entities.</span></span> <span data-ttu-id="e4dfc-239">当使用为每个工作单元创建生存期较短的新上下文实例的最佳做法时，通常不需要这样做。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-239">This should usually not be needed when using the best practice of creating a new, short-lived context instance for each unit-of-work.</span></span> <span data-ttu-id="e4dfc-240">但如果需要重置 DbContext 实例的状态，则使用新的 `Clear()` 方法比大量分离所有实体更高效可靠。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-240">However, if there is a need to reset the state of a DbContext instance, then using the new `Clear()` method is more efficient and robust than mass-detaching all entities.</span></span>
* <span data-ttu-id="e4dfc-241">EF Core 命令行工具现在会自动将 `ASPNETCORE_ENVIRONMENT` 和`DOTNET_ENVIRONMENT` 环境变量配置为“开发”。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-241">The EF Core command line tools now automatically configure the `ASPNETCORE_ENVIRONMENT` _and_ `DOTNET_ENVIRONMENT` environment variables to "Development".</span></span> <span data-ttu-id="e4dfc-242">这带来了在开发过程中使用通用主机时与 ASP.NET Core 体验一样。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-242">This brings the experience when using the generic host in line with the experience for ASP.NET Core during development.</span></span>
* <span data-ttu-id="e4dfc-243">自定义命令行参数可流入 <xref:Microsoft.EntityFrameworkCore.Design.IDesignTimeDbContextFactory%601>，这使得应用程序可控制如何创建和初始化上下文。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-243">Custom command-line arguments can be flowed into <xref:Microsoft.EntityFrameworkCore.Design.IDesignTimeDbContextFactory%601>, allowing applications to control how the context is created and initialized.</span></span>
* <span data-ttu-id="e4dfc-244">现可[在 SQL Server 上配置](xref:core/providers/sql-server/indexes#fill-factor)索引填充因子。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-244">The index fill factor can now be [configured on SQL Server](xref:core/providers/sql-server/indexes#fill-factor).</span></span>
* <span data-ttu-id="e4dfc-245">新的 <xref:Microsoft.EntityFrameworkCore.RelationalDatabaseFacadeExtensions.IsRelational%2A> 属性可用于区分使用关系提供程序和非关系提供程序（例如 InMemory）的时间。</span><span class="sxs-lookup"><span data-stu-id="e4dfc-245">The new <xref:Microsoft.EntityFrameworkCore.RelationalDatabaseFacadeExtensions.IsRelational%2A> property can be used to distinguish when using a relational provider and a non-relation provider (such as InMemory).</span></span>
