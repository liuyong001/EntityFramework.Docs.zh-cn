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
# <a name="whats-new-in-ef-core-50"></a>EF Core 5.0 中的新增功能

以下列表包括 EF Core 5.0 的主要新功能。 有关发布中的问题的完整列表，请查看我们的[问题跟踪程序](https://github.com/dotnet/efcore/issues?q=is%3Aissue+milestone%3A5.0.0)。

EF Core 5.0 是一个主要版本，还包含多项[中断性变更](xref:core/what-is-new/ef-core-5.0/breaking-changes)，也就是可能会对现有应用程序产生负面影响的 API 改进或行为更改。

## <a name="many-to-many"></a>多对多

EF Core 5.0 支持多对多关系，而无需显式映射联接表。

以下面这些实体类型为例：

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

EF Core 5.0 按照约定将其识别为多对多关系，并在数据库中自动创建一个 `PostTag` 联接表。 无需显式引用联接表即可查询和更新数据，这大大简化了代码。 仍可自定义联接表，并在需要时显式查询它。

有关详细信息，请[参阅有关多对多的完整文档](xref:core/modeling/relationships#many-to-many)。

## <a name="split-queries"></a>拆分查询

从 EF Core 3.0 开始，EF Core 始终会为每个 LINQ 查询生成一个 SQL 查询。 这可确保在使用中的事务模式的约束内返回的数据保持一致。 但是，当查询使用 `Include` 或投影来返回多个相关集合时，速度可能会变得非常慢。

EF Core 5.0 现允许将包含相关集合的一个 LINQ 查询拆分成多个 SQL 查询。 这可能明显提升性能，但如果在两次查询之间数据发生了变化，则可能导致返回的结果不一致。 你能使用可序列化的事务或快照事务来缓解这种情况并通过拆分查询实现一致性，但这可能会带来其他性能成本并导致行为差异。

例如，请考虑使用 `Include` 提取两个级别的相关集合的查询：

```csharp
var artists = context.Artists
    .Include(e => e.Albums)
    .ToList();
```

默认情况下，EF Core 将使用 SQLite 提供程序生成以下 SQL：

```sql
SELECT a."Id", a."Name", a0."Id", a0."ArtistId", a0."Title"
FROM "Artists" AS a
LEFT JOIN "Album" AS a0 ON a."Id" = a0."ArtistId"
ORDER BY a."Id", a0."Id"
```

使用拆分查询时，将改为生成以下 SQL：

```sql
SELECT a."Id", a."Name"
FROM "Artists" AS a
ORDER BY a."Id"

SELECT a0."Id", a0."ArtistId", a0."Title", a."Id"
FROM "Artists" AS a
INNER JOIN "Album" AS a0 ON a."Id" = a0."ArtistId"
ORDER BY a."Id"
```

可通过将新的 `AsSplitQuery` 运算符放置在 LINQ 查询中的任何位置或全局放置在模型的 `OnConfiguring` 中来启用拆分查询。 有关详细信息，请[参阅有关拆分查询的完整文档](xref:core/querying/single-split-queries)。

## <a name="simple-logging-and-improved-diagnostics"></a>简单的日志记录和改进的诊断

EF Core 5.0 引入了一种通过新的 `LogTo` 方法来设置日志记录的简单方法。 以下内容会导致将日志记录消息写入控制台，包括 EF Core 生成的所有 SQL：

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder.LogTo(Console.WriteLine);
```

此外，现可在任何 LINQ 查询上调用 `ToQueryString`，以检索该查询将执行的 SQL：

```csharp
Console.WriteLine(
    ctx.Artists
    .Where(a => a.Name == "Pink Floyd")
    .ToQueryString());
```

最后，各种 EF Core 类型都配备了增强的 `DebugView` 属性，它针对内部项提供详细视图。 例如，可参考 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.DebugView%2A?displayProperty=nameWithType> 来查看在给定时间正在被跟踪的确切实体。

有关详细信息，请[参阅有关日志记录和拦截的文档](xref:core/logging-events-diagnostics/index)。

## <a name="filtered-include"></a>经过筛选的包含

`Include` 方法现支持筛选包含的实体：

```csharp
var blogs = context.Blogs
    .Include(e => e.Posts.Where(p => p.Title.Contains("Cheese")))
    .ToList();
```

此查询将一并返回包含每个关联文章的博客（仅当文章标题包含“Cheese”时）。

有关详细信息，请[参阅有关拆分查询的完整文档](xref:core/querying/related-data/eager#filtered-include)。

## <a name="table-per-type-tpt-mapping"></a>每个类型一张表 (TPT) 映射

默认情况下，EF Core 会将 .NET 类型的继承层次结构映射到单个数据库表。 这称为每个层次结构一张表 (TPH) 映射。 EF Core 5.0 还允许将继承层次结构中的每个 .NET 类型映射到另一个数据库表，这称为每个类型一张表 (TPT) 映射。

例如，请考虑具有映射的层次结构的此模型：

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

使用 TPT 时，将为层次结构中的每种类型创建一个数据库表：

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

有关详细信息，请[参阅有关 TPT 的完整文档](xref:core/modeling/inheritance)。

## <a name="flexible-entity-mapping"></a>灵活实体映射

实体类型通常映射到表或视图，以便 EF Core 在查询该类型时将拉回表或视图的内容。 EF Core 5.0 添加了其他映射选项，其中可将实体映射到 SQL 查询（称为“定义查询”）或表值函数 (TVF)：

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

表值函数也可映射到 .NET 方法而不是 DbSet，从而允许传递参数。可使用 <xref:Microsoft.EntityFrameworkCore.RelationalModelBuilderExtensions.HasDbFunction%2A> 设置映射。

最后，现可在查询时将实体映射到视图（或映射到函数或定义查询中），而在更新时将实体映射到表：

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<Blog>()
        .ToTable("Blogs")
        .ToView("BlogsView");
}
```

## <a name="shared-type-entity-types-and-property-bags"></a>共享型实体类型和属性包

EF Core 5.0 允许将相同的 CLR 类型映射到多个不同的实体类型，这种类型称为共享型实体类型。 尽管任何 CLR 类型都可用于此功能，但 .NET `Dictionary` 提供了一个特别引人注目的用例，我们称其为“属性包”：

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

随后，这些实体可像普通实体类型一样使用它们自己的专用 CLR 类型来进行查询和更新。 有关详细信息，可查看关于[属性包](xref:core/modeling/shadow-properties)的文档。

## <a name="required-11-dependents"></a>必需的一对一依赖项

在 EF Core 3.1 中，一对一关系的依赖端始终被视为可选。 使用拥有的实体时，这一点最明显，因为拥有的实体的所有列在数据库中都被创建为可为 null，即使它们已按模型中的要求进行配置也是这样。

在 EF Core 5.0 中，可将到拥有的实体的导航配置为必需依赖项。 例如：

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

## <a name="dbcontextfactory"></a>DbContextFactory

EF Core 5.0 引入了 `AddDbContextFactory` 和 `AddPooledDbContextFactory` 来注册工厂，以便在应用程序的依赖项注入 (D.I.) 容器中创建 DbContext 实例；当应用程序代码需要手动创建和处理上下文实例时，这很有用。

```csharp
services.AddDbContextFactory<SomeDbContext>(b =>
    b.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test"));
```

此时，可使用 `IDbContextFactory<TContext>` 注入 ASP.NET Core 控制器等应用程序服务，并用它来实例化上下文实例：

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

有关详细信息，请[参阅有关 DbContextFactory 的完整文档](xref:core/dbcontext-configuration/index#using-a-dbcontext-factory-eg-for-blazor)。

## <a name="sqlite-table-rebuilds"></a>重新生成 SQLite 表

与其他数据库相比，SQLite 的架构操作能力相对有限；例如，不支持从现有表中删除列。 EF Core 5.0 解决这些限制的方式是自动创建一个新表、从旧表复制数据、删除旧表并重命名新表。 这会“重新生成”表，而且可安全地应用之前不受支持的迁移操作。

若要详细了解现在通过重新生成表而支持的具体迁移操作，请[参阅此文档页面](xref:core/providers/sqlite/limitations#migrations-limitations)。

## <a name="database-collations"></a>数据库排序规则

EF Core 5.0 现支持在数据库、列或查询级别指定文本排序规则。 这样就可以灵活但不影响查询性能的方式配置大小写区分和其他文本方面。

例如，以下内容将 `Name` 列配置为在 SQL Server 上区分大小写，并且在该列上创建的任何索引都将相应地起作用：

```csharp
modelBuilder
    .Entity<User>()
    .Property(e => e.Name)
    .UseCollation("SQL_Latin1_General_CP1_CS_AS");
```

有关详细信息，请[参阅有关排序规则和大小写区分的完整文档](xref:core/miscellaneous/collations-and-case-sensitivity)。

## <a name="event-counters"></a>事件计数器

EF Core 5.0 公开了[事件计数器](https://devblogs.microsoft.com/dotnet/introducing-diagnostics-improvements-in-net-core-3-0)，它可用于跟踪应用程序的性能并发现各种异常情况。 只需使用 [dotnet-counters](https://docs.microsoft.com/dotnet/core/diagnostics/dotnet-counters) 工具将其附加到运行 EF 的进程即可：

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

有关详细信息，请[参阅有关事件计数器的完整文档](xref:core/logging-events-diagnostics/event-counters)。

## <a name="other-features"></a>其他功能

### <a name="model-building"></a>模型构建

* 为了更轻松地配置[值比较器](xref:core/modeling/value-comparers)，引入了模型构建 API。
* 现可将计算列配置为[存储或虚拟](xref:core/modeling/generated-properties#computed-columns) 。
* 现可[通过 Fluent API](xref:core/modeling/entity-properties#precision-and-scale) 配置精度和规模。
* 针对[导航属性](xref:core/modeling/relationships#configuring-navigation-properties)引入了新的模型构建 API。
* 与属性类似，针对字段引入了新的模型构建 API。
* 现可将 .NET [PhysicalAddress](/dotnet/api/system.net.networkinformation.physicaladdress) 和 [IPAddress](/dotnet/api/system.net.ipaddress) 类型映射到数据库字符串列。
* 现可通过[新的 `[BackingField]` 属性](xref:core/modeling/backing-field)来配置支持字段。
* 现可使用可为 null 的支持字段，在 CLR 默认值不是一个好的 sentinel 值（`bool` 值得注意）的情况下，这更好地支持了商店生成的默认值。
* 可在实体类型上使用新的 `[Index]` 属性来指定索引，而不是使用 Fluent API。
* 可使用新的 `[Keyless]` 属性将实体类型配置为[没有键](xref:core/modeling/keyless-entity-types)。
* 默认情况下，[EF Core 现将鉴别器视为完整](xref:core/modeling/inheritance#table-per-hierarchy-and-discriminator-configuration)，这意味着它预期永远不会看到模型中应用程序未配置的鉴别器值。 这样可一定程度地提高性能；如果鉴别器列可能包含未知值，可将其禁用。

### <a name="query"></a>查询

* 查询转换失败异常现包含更明确的失败原因，以帮助查明问题。
* 非跟踪查询现可执行[标识解析](xref:core/querying/tracking#identity-resolution)，从而避免为同一数据库对象返回多个实体实例。
* 现支持使用条件聚合进行 GroupBy 操作（例如 `GroupBy(o => o.OrderDate).Select(g => g.Count(i => i.OrderDate != null))`）。
* 现支持在聚合之前针对组元素转换 Distinct 运算符。
* 对 [`Reverse`](/dotnet/api/system.linq.queryable.reverse) 的转换。
* 针对 SQL Server 改进了对 `DateTime` 的转换（例如 [`DateDiffWeek`](xref:Microsoft.EntityFrameworkCore.SqlServerDbFunctionsExtensions.DateDiffWeek%2A)、[`DateFromParts`](xref:Microsoft.EntityFrameworkCore.SqlServerDbFunctionsExtensions.DateFromParts%2A)）。
* 字节数组上对新方法的转换（例如 [`Contains`](/dotnet/api/system.linq.enumerable.contains)、[`Length`](/dotnet/api/system.array.length)、[`SequenceEqual`](/dotnet/api/system.linq.enumerable.sequenceequal)）。
* 对一些其他按位运算符的转换，例如二进制补码。
* 针对字符串上 `FirstOrDefault` 的转换。
* 围绕 NULL 语义改进了查询转换，使查询更紧密、更高效。
* 现可对用户映射的函数进行批注以控制 Null 传播，这又使得查询更紧密、更高效。
* 包含 CASE 块的 SQL 现在明显简洁得多。
* 现可通过新的 [`EF.Functions.DataLength`](xref:Microsoft.EntityFrameworkCore.SqlServerDbFunctionsExtensions.DataLength%2A) 方法在查询中调用 SQL Server [`DATELENGTH`](https://docs.microsoft.com/sql/t-sql/functions/datalength-transact-sql) 函数。
* `EnableDetailedErrors` 将[ 其他详细信息添加到异常](xref:core/logging-events-diagnostics/simple-logging#detailed-query-exceptions)。

### <a name="saving"></a>保存

* SaveChanges [拦截](xref:core/logging-events-diagnostics/interceptors#savechanges-interception)和[事件](xref:core/logging-events-diagnostics/events)。
* 引入了用于控制[事务保存点](xref:core/saving/transactions#savepoints)的 API。 此外，当调用 `SaveChanges` 并且事务已经在进行时，EF Core 将自动创建一个保存点，并在失败时回滚到该保存点。
* 事务 ID 可由应用程序显式设置，从而可更轻松地在日志记录和其他位置关联事务事件。
* 根据对批处理性能的分析，SQL Server 的默认最大批处理大小已更改为 42。

### <a name="migrations-and-scaffolding"></a>迁移和基架

* 现可[从迁移中排除](xref:core/modeling/entity-types#excluding-from-migrations)表。
* 新的 [`dotnet ef migrations list`](xref:core/cli/dotnet#dotnet-ef-migrations-list) 命令现将显示哪些迁移尚未应用于数据库（[`Get-Migration`](xref:core/cli/powershell#get-migration) 在包管理控制台中执行相同的操作）。
* 迁移脚本现在适当的位置包含事务语句，以改善迁移应用程序失败时的处理情况。
* 未映射基类的列现在按映射实体类型的其他列排序。 请注意，这仅影响新创建的表。 现有表的列顺序保持不变。
* 迁移生成现可知道正在生成的迁移是否是幂等的，以及输出是立即执行还是生成用作脚本。
* 添加了新的命令行参数，用于在[迁移](xref:core/managing-schemas/migrations/index#namespaces)和[基架](xref:core/managing-schemas/scaffolding#directories-and-namespaces)中指定命名空间。
* [dotnet ef database update](xref:core/cli/dotnet#dotnet-ef-database-update) 命令现接受一个新的 `--connection` 参数来指定连接字符串。
* 现在，对现有数据库搭建基架会将表名单数化，因此名为 `People` 和 `Addresses` 的表将搭建基架成为称作 `Person` 和 `Address` 的实体类型。 [仍可保留原始数据库名称](xref:core/managing-schemas/scaffolding#preserving-names)。
* 新的 [`--no-onconfiguring`](xref:core/cli/dotnet#dotnet-ef-dbcontext-scaffold) 选项可指示 EF Core 在搭建模型基架时排除 `OnModelConfiguring`。

### <a name="cosmos"></a>Cosmos

* [Cosmos 连接设置](xref:core/providers/cosmos/index#cosmos-options)已扩展。
* 现支持在 Cosmos 上[使用 ETag](xref:core/providers/cosmos/index#optimistic-concurrency-with-etags) 来进行乐观并发。
* 新的 `WithPartitionKey` 方法使 Cosmos [分区键](xref:core/providers/cosmos/index#partition-keys)既可包含在模型中，也可包含在查询中。
* 现针对 Cosmos 转换了字符串方法 [`Contains`](/dotnet/api/system.string.contains)、[`StartsWith`](/dotnet/api/system.string.startswith) 和 [`EndsWith`](/dotnet/api/system.string.endswith)。
* 现在 Cosmos 上转换了 C# `is` 运算符。

### <a name="sqlite"></a>Sqlite

* 现支持计算列。
* 现在，利用 SqliteBlob 和流，可以更高效地使用 GetBytes、GetChars 和 GetTextReader 检索二进制和字符串数据。
* SqliteConnection 的初始化现在很慢。

### <a name="other"></a>其他

* 可生成自动执行 [INotifyPropertyChanging](/dotnet/api/system.componentmodel.inotifypropertychanging) 和 [INotifyPropertyChanged](/dotnet/api/system.componentmodel.inotifypropertychanged) 的更改跟踪代理。 对于调用 `SaveChanges` 时不扫描更改的更改跟踪而言，这提供了一种替代方法。
* 现可在已初始化的 DbContext 上更改 <xref:System.Data.Common.DbConnection> 或连接字符串。
* 新的 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Clear%2A?displayProperty=nameWithType>.0#Microsoft_EntityFrameworkCore_ChangeTracking_ChangeTracker_Clear) 方法会清除所有跟踪实体的 DbContext。 当使用为每个工作单元创建生存期较短的新上下文实例的最佳做法时，通常不需要这样做。 但如果需要重置 DbContext 实例的状态，则使用新的 `Clear()` 方法比大量分离所有实体更高效可靠。
* EF Core 命令行工具现在会自动将 `ASPNETCORE_ENVIRONMENT` 和`DOTNET_ENVIRONMENT` 环境变量配置为“开发”。 这带来了在开发过程中使用通用主机时与 ASP.NET Core 体验一样。
* 自定义命令行参数可流入 <xref:Microsoft.EntityFrameworkCore.Design.IDesignTimeDbContextFactory%601>，这使得应用程序可控制如何创建和初始化上下文。
* 现可[在 SQL Server 上配置](xref:core/providers/sql-server/indexes#fill-factor)索引填充因子。
* 新的 <xref:Microsoft.EntityFrameworkCore.RelationalDatabaseFacadeExtensions.IsRelational%2A> 属性可用于区分使用关系提供程序和非关系提供程序（例如 InMemory）的时间。
