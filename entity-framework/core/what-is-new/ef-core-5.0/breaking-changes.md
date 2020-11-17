---
title: EF Core 5.0 中的中断性变更 - EF Core
description: Entity Framework Core 5.0 中引入的中断性变更的完整列表
author: bricelam
ms.date: 11/07/2020
uid: core/what-is-new/ef-core-5.0/breaking-changes
ms.openlocfilehash: e2537dbc1d5dba48450bd0fea7712054ba2fa622
ms.sourcegitcommit: 42bbf7f68e92c364c5fff63092d3eb02229f568d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/11/2020
ms.locfileid: "94503171"
---
# <a name="breaking-changes-in-ef-core-50"></a>EF Core 5.0 中的中断性变更

API 和行为的下列更改有可能导致现有应用程序在更新到 EF Core 5.0.0 时中断。

## <a name="summary"></a>摘要

| **中断性变更**                                                                                                                   | **影响** |
|:--------------------------------------------------------------------------------------------------------------------------------------|------------|
| [具有不同语义的从主体到依赖项的导航中必需](#required-dependent)                                 | 中     |
| [定义查询替换为特定于提供程序的方法](#defining-query)                                                          | 中     |
| [查询不覆盖非 null 引用导航](#nonnullreferences)                                                   | 中     |
| [从 SQLite NTS 扩展中删除了 HasGeometricDimension 方法](#geometric-sqlite)                                                   | 低        |
| [Cosmos：分区键现已添加到主键](#cosmos-partition-key)                                                        | 低        |
| [Cosmos：`id` 属性重命名为 `__id`](#cosmos-id)                                                                                 | 低        |
| [Cosmos：byte[] 现在存储为 base64 字符串而不是数字数组](#cosmos-byte)                                             | 低        |
| [Cosmos：GetPropertyName 和 SetPropertyName 已重命名](#cosmos-metadata)                                                          | 低        |
| [当实体状态从“已分离”更改为“未更改”、“已更新”或“已删除”时，将调用值生成器](#non-added-generation) | 低        |
| [IMigrationsModelDiffer 当前使用 IRelationalModel](#relational-model)                                                                 | 低        |
| [鉴别器是只读的](#read-only-discriminators)                                                                             | 低        |
| [特定于提供程序的 EF.Functions 方法针对 InMemory 提供程序引发](#no-client-methods)                                              | 低        |
| [IndexBuilder.HasName 现已过时](#index-obsolete)                                                                               | 低        |
| [现已包括用于搭建实施了反向工程的模型的复数化程序](#pluralizer)                                                 | 低        |
| [INavigationBase 替换某些 API 中的 INavigation 以支持跳过导航](#inavigationbase)                                     | 低        |
| [不再支持使用相关集合同时还使用 `Distinct` 或 `GroupBy` 的查询](#collection-distinct-groupby) | 低        |
| [不支持在投影中使用可查询类型的集合](#queryable-projection)                                          | 低        |

## <a name="medium-impact-changes"></a>影响中等的更改

<a name="required-dependent"></a>

### <a name="required-on-the-navigation-from-principal-to-dependent-has-different-semantics"></a>具有不同语义的从主体到依赖项的导航中必需

[跟踪问题 #17286](https://github.com/dotnet/efcore/issues/17286)

#### <a name="old-behavior"></a>旧行为

只有到主体的导航才能配置为“必需”。 因此，在对到依赖项（包含外键的实体）的导航使用 `RequiredAttribute` 时，将会改为在定义实体类型上创建外键。

#### <a name="new-behavior"></a>新行为

借助对所需依赖项新增的支持后，现在可以将任何引用导航标记为“必需”，这意味着在上述情况下，外键将在关系的另一端进行定义，并且这些属性不会标记为“必需”。

目前在指定依赖端之前是否调用 `IsRequired` 尚不明确：

```csharp
modelBuilder.Entity<Blog>()
    .HasOne(b => b.BlogImage)
    .WithOne(i => i.Blog)
    .IsRequired()
    .HasForeignKey<BlogImage>(b => b.BlogForeignKey);
```

#### <a name="why"></a>原因

为了支持所需的依赖项，需要新行为（[请参阅 #12100](https://github.com/dotnet/efcore/issues/12100)）。

#### <a name="mitigations"></a>缓解措施

从到依赖项的导航中删除 `RequiredAttribute`，转而将其放置在到主体的导航中或在 `OnModelCreating` 中配置关系：

```csharp
modelBuilder.Entity<Blog>()
    .HasOne(b => b.BlogImage)
    .WithOne(i => i.Blog)
    .HasForeignKey<BlogImage>(b => b.BlogForeignKey)
    .IsRequired();
```

<a name="defining-query"></a>

### <a name="defining-query-is-replaced-with-provider-specific-methods"></a>定义查询替换为特定于提供程序的方法

[跟踪问题 #18903](https://github.com/dotnet/efcore/issues/18903)

#### <a name="old-behavior"></a>旧行为

实体类型映射到定义核心级别的查询。 每当在查询中使用实体类型时，实体类型的根将被替换为任何提供程序的定义查询。

#### <a name="new-behavior"></a>新行为

已弃用用于定义查询的 API。 引入了特定于提供程序的新 API。

#### <a name="why"></a>原因

在查询中使用查询根时，定义查询作为替换查询实现，但它有一些问题：

- 如果定义查询使用 `Select` 方法中的 `new { ... }` 预测实体类型，则将查询标识为实体需要额外的工作，并且与 EF Core 在查询中处理名义类型的方式不一致。
- 对于关系提供程序 `FromSql`，仍然需要以 LINQ 表达式格式传递 SQL 字符串。

定义查询最初是作为客户端视图引入的，用于无键实体的内存中提供程序（类似于关系数据库中的数据库视图）。 这种定义使得对内存中数据库测试应用程序变得容易。 之后，它们变得广泛适用，这很有用，但带来了不一致和难以理解的行为。 因此，我们决定简化概念。 我们使基于 LINQ 的定义查询独占内存中提供程序，并以不同的方式对其进行处理。 有关详细信息，[请参阅此问题](https://github.com/dotnet/efcore/issues/20023)。

#### <a name="mitigations"></a>缓解措施

对于关系提供程序，在 `OnModelCreating` 中使用 `ToSqlQuery` 方法，然后传递 SQL 字符串以用于实体类型。
对于内存中提供程序，在 `OnModelCreating` 中使用 `ToInMemoryQuery` 方法，然后传递要用于实体类型 LINQ 查询。

<a name="nonnullreferences"></a>

### <a name="non-null-reference-navigations-are-not-overwritten-by-queries"></a>查询不覆盖非 null 引用导航

[跟踪问题 #2693](https://github.com/dotnet/EntityFramework.Docs/issues/2693)

#### <a name="old-behavior"></a>旧行为

在 EF Core 3.1 中，无论键值是否匹配，数据库中的实体实例有时会覆盖预先初始化为非 null 值的引用导航。 而在其他情况下，EF Core 3.1 会执行相反的操作，保留现有的非 null 值。

#### <a name="new-behavior"></a>新行为

从 EF Core 5.0 开始，在任何情况下查询返回的实例都不会覆盖非 null 引用导航。

请注意，仍支持将集合导航预先初始化为一个空集合。

#### <a name="why"></a>原因

将引用导航属性初始化为“空”实体实例将导致状态模糊。 例如：

```csharp
public class Blog
{
     public int Id { get; set; }
     public Author Author { get; set; ) = new Author();
}
```

一个针对博客和作者的查询通常首先会创建 `Blog` 实例，然后根据数据库返回的数据设置适当的 `Author` 实例。 但是，在这种情况下，每个 `Blog.Author` 属性都已初始化为空 `Author`。 不过 EF Core 无法知道此实例是否为“空”。 因此覆盖此实例可能会悄无声息地丢弃一个有效的 `Author`。 因而，EF Core 5.0 现在始终不会覆盖已初始化的导航。

尽管经过调查发现，此新行为有时与 EF6 的行为不一致，但在大多数情况下它都与 EF6 一致。  

#### <a name="mitigations"></a>缓解措施

如果遇到此中断，解决方法是停止预先初始化引用导航属性。

## <a name="low-impact-changes"></a>影响较小的更改

<a name="geometric-sqlite"></a>

### <a name="removed-hasgeometricdimension-method-from-sqlite-nts-extension"></a>从 SQLite NTS 扩展中删除了 HasGeometricDimension 方法

[跟踪问题 #14257](https://github.com/dotnet/efcore/issues/14257)

#### <a name="old-behavior"></a>旧行为

HasGeometricDimension 过去用于在几何列上启用其他维度（Z 和 M）。 但是，之前它只影响数据库创建。 不需要指定它来查询具有其他维度的值。 之前，在插入或更新具有其他维度的值时，它也无法正常工作（请参见 [see #14257](https://github.com/dotnet/efcore/issues/14257)）。

#### <a name="new-behavior"></a>新行为

要能够插入和更新具有其他维度（Z 和 M）的几何值，需要将维度指定为列类型名称的一部分。 该 API 与 SpatiaLite 的 AddGeometryColumn 函数的基本行为匹配度更高。

#### <a name="why"></a>原因

在列类型中指定维度后，不需要使用 HasGeometricDimension，该方法也很多余，因此我们已将它彻底删除。

#### <a name="mitigations"></a>缓解措施

使用 `HasColumnType` 指定维度：

```csharp
modelBuilder.Entity<GeoEntity>(
    x =>
    {
        // Allow any GEOMETRY value with optional Z and M values
        x.Property(e => e.Geometry).HasColumnType("GEOMETRYZM");

        // Allow only POINT values with an optional Z value
        x.Property(e => e.Point).HasColumnType("POINTZ");
    });
```

<a name="cosmos-partition-key"></a>

### <a name="cosmos-partition-key-is-now-added-to-the-primary-key"></a>Cosmos：分区键现已添加到主键

[跟踪问题 #15289](https://github.com/dotnet/efcore/issues/15289)

#### <a name="old-behavior"></a>旧行为

分区键仅添加到包含 `id` 的备用键。

#### <a name="new-behavior"></a>新行为

分区键现在还按约定添加到主键。

#### <a name="why"></a>原因

此更改使模型更好地与 Azure Cosmos DB 语义对齐，并改进 `Find` 和某些查询的性能。

#### <a name="mitigations"></a>缓解措施

若要防止将分区键属性添加到主键，请在 `OnModelCreating` 中配置它。

```csharp
modelBuilder.Entity<Blog>()
    .HasKey(b => b.Id);
```

<a name="cosmos-id"></a>

### <a name="cosmos-id-property-renamed-to-__id"></a>Cosmos：`id` 属性重命名为 `__id`

[跟踪问题 #17751](https://github.com/dotnet/efcore/issues/17751)

#### <a name="old-behavior"></a>旧行为

映射到 `id` JSON 属性的阴影属性也称为 `id`。

#### <a name="new-behavior"></a>新行为

按照约定创建的阴影属性现在命名为 `__id`。

#### <a name="why"></a>原因

此更改使 `id` 属性与实体类型上的现有属性冲突的可能性更小。

#### <a name="mitigations"></a>缓解措施

若要返回到 3.x 行为，请在 `OnModelCreating` 中配置 `id` 属性。

```csharp
modelBuilder.Entity<Blog>()
    .Property<string>("id")
    .ToJsonProperty("id");
```

<a name="cosmos-byte"></a>

### <a name="cosmos-byte-is-now-stored-as-a-base64-string-instead-of-a-number-array"></a>Cosmos：byte[] 现在存储为 base64 字符串而不是数字数组

[跟踪问题 #17306](https://github.com/dotnet/efcore/issues/17306)

#### <a name="old-behavior"></a>旧行为

类型 byte[] 的属性存储为数字数组。

#### <a name="new-behavior"></a>新行为

类型 byte[] 的属性现存储为 base64 字符串。

#### <a name="why"></a>原因

byte[] 的这种表示形式与预期更好地对齐，并且是主要 JSON 序列化库的默认行为。

#### <a name="mitigations"></a>缓解措施

仍将正确查询作为数字数组存储的现有数据，但目前没有支持更改插入行为的方法。 如果此限制对于你的方案是一种阻碍，请对[此问题](https://github.com/dotnet/efcore/issues/17306)发表评论

<a name="cosmos-metadata"></a>

### <a name="cosmos-getpropertyname-and-setpropertyname-were-renamed"></a>Cosmos：GetPropertyName 和 SetPropertyName 已重命名

[跟踪问题 #17874](https://github.com/dotnet/efcore/issues/17874)

#### <a name="old-behavior"></a>旧行为

扩展方法此前称为 `GetPropertyName` 和 `SetPropertyName`

#### <a name="new-behavior"></a>新行为

旧 API 已删除，添加了以下新方法：`GetJsonPropertyName`、`SetJsonPropertyName`

#### <a name="why"></a>原因

此更改消除了有关这些方法配置方式的歧义。

#### <a name="mitigations"></a>缓解措施

使用新 API。

<a name="non-added-generation"></a>

### <a name="value-generators-are-called-when-the-entity-state-is-changed-from-detached-to-unchanged-updated-or-deleted"></a>当实体状态从“已分离”更改为“未更改”、“已更新”或“已删除”时，将调用值生成器

[跟踪问题 #15289](https://github.com/dotnet/efcore/issues/15289)

#### <a name="old-behavior"></a>旧行为

只有当实体状态更改为“已添加”时，才调用值生成器。

#### <a name="new-behavior"></a>新行为

目前，当实体状态从“已分离”更改为“未更改”、“已更新”或“已删除”，且属性包含默认值时，将调用值生成器。

#### <a name="why"></a>原因

此更改是必要的，以便改进未保存到数据存储并始终在客户端上生成其值的属性方面的体验。

#### <a name="mitigations"></a>缓解措施

若要防止调用值生成器，请在状态更改前向属性分配一个非默认值。

<a name="relational-model"></a>

### <a name="imigrationsmodeldiffer-now-uses-irelationalmodel"></a>IMigrationsModelDiffer 当前使用 IRelationalModel

[跟踪问题 #20305](https://github.com/dotnet/efcore/issues/20305)

#### <a name="old-behavior"></a>旧行为

`IMigrationsModelDiffer` API 使用了 `IModel` 进行定义。

#### <a name="new-behavior"></a>新行为

`IMigrationsModelDiffer` API 当前使用 `IRelationalModel`。 不过，模型快照仍只包含 `IModel`，因为此代码是应用程序的一部分，并且实体框架无法在不做出较大中断性变更的情况下对其进行更改。

#### <a name="why"></a>原因

`IRelationalModel` 是新添加的数据库架构的表示形式。 使用它可以更快速、更精确地查找差异。

#### <a name="mitigations"></a>缓解措施

使用以下代码将 `context` 中的模型与 `snapshot` 中的模型进行比较：

```csharp
var dependencies = context.GetService<ProviderConventionSetBuilderDependencies>();
var relationalDependencies = context.GetService<RelationalConventionSetBuilderDependencies>();

var typeMappingConvention = new TypeMappingConvention(dependencies);
typeMappingConvention.ProcessModelFinalizing(((IConventionModel)modelSnapshot.Model).Builder, null);

var relationalModelConvention = new RelationalModelConvention(dependencies, relationalDependencies);
var sourceModel = relationalModelConvention.ProcessModelFinalized(snapshot.Model);

var modelDiffer = context.GetService<IMigrationsModelDiffer>();
var hasDifferences = modelDiffer.HasDifferences(
    ((IMutableModel)sourceModel).FinalizeModel().GetRelationalModel(),
    context.Model.GetRelationalModel());
```

我们计划在 6.0 中改进这种体验（[请参阅 #22031](https://github.com/dotnet/efcore/issues/22031)）

<a name="read-only-discriminators"></a>

### <a name="discriminators-are-read-only"></a>鉴别器是只读的

[跟踪问题 #21154](https://github.com/dotnet/efcore/issues/21154)

#### <a name="old-behavior"></a>旧行为

在调用 `SaveChanges` 之前，可以更改鉴别器值

#### <a name="new-behavior"></a>新行为

在上述情况下，将引发异常。

#### <a name="why"></a>原因

EF 不希望实体类型仍在受到跟踪时就发生更改，因此更改鉴别器值会使上下文处于不一致的状态，这可能导致意外行为。

#### <a name="mitigations"></a>缓解措施

如果更改鉴别器值是必需的，并且在调用 `SaveChanges` 后将立即处理上下文，则可将鉴别器设置为可变：

```csharp
modelBuilder.Entity<BaseEntity>()
    .Property<string>("Discriminator")
    .Metadata.SetAfterSaveBehavior(PropertySaveBehavior.Save);
```

<a name="no-client-methods"></a>

### <a name="provider-specific-effunctions-methods-throw-for-inmemory-provider"></a>特定于提供程序的 EF.Functions 方法针对 InMemory 提供程序引发

[跟踪问题 #20294](https://github.com/dotnet/efcore/issues/20294)

#### <a name="old-behavior"></a>旧行为

特定于提供程序的 EF.Functions 方法包含对客户端执行的实现，从而使这些方法可以在 InMemory 提供程序上执行。 例如，`EF.Functions.DateDiffDay` 是特定于 SQL Server 的方法，它在 InMemory 提供程序上运行。

#### <a name="new-behavior"></a>新行为

特定于提供程序的方法已更新为在其方法主体中引发异常，以阻止在客户端上对它们进行评估。

#### <a name="why"></a>原因

特定于提供程序的方法映射到数据库函数。 在 LINQ 中，映射的数据库函数执行的计算无法每次都在客户端上进行复制。 当在客户端执行相同的方法时，这可能会导致来自服务器的结果有所不同。 由于这些方法会用于 LINQ，来转换到特定数据库函数，因此无需在客户端上评估这些方法。 由于 InMemory 提供程序是另一种数据库，因此这些方法不能用于此提供程序。 如果尝试为 InMemory 提供程序或任何其他无法转换这些方法的提供程序执行它们时，将引发异常。

#### <a name="mitigations"></a>缓解措施

由于无法准确模拟数据库函数的行为，因此应根据生产中的同一种数据库测试包含这些函数的查询。

<a name="index-obsolete"></a>

### <a name="indexbuilderhasname-is-now-obsolete"></a>IndexBuilder.HasName 现已过时

[跟踪问题 #21089](https://github.com/dotnet/efcore/issues/21089)

#### <a name="old-behavior"></a>旧行为

以前，只能在给定的一组属性上定义一个索引。 索引的数据库名称是使用 IndexBuilder.HasName 配置的。

#### <a name="new-behavior"></a>新行为

现在，允许在同一组属性上使用多个索引。 这些索引现在可以通过模型中的名称来区分。 按照约定，模型名称将用作数据库名称；不过，也可以使用 HasDatabaseName 单独进行配置。

#### <a name="why"></a>原因

将来，我们希望使用同一组属性上的不同排序规则对索引启用升序和降序。 此更改将沿该方向转至其他步骤。

#### <a name="mitigations"></a>缓解措施

之前调用 IndexBuilder.HasName 的任何代码都应更新为调用 HasDatabaseName。

如果你的项目包含 EF Core 版本 2.0.0 之前生成的迁移，则可以安全地忽略这些文件中的警告，并通过添加 `#pragma warning disable 612, 618` 将其取消。

<a name="pluralizer"></a>

### <a name="a-pluralizer-is-now-included-for-scaffolding-reverse-engineered-models"></a>现已包括用于搭建实施了反向工程的模型的复数化程序

[跟踪问题 #11160](https://github.com/dotnet/efcore/issues/11160)

#### <a name="old-behavior"></a>旧行为

以前，必须安装单独的复数化程序包，才能在通过对数据库架构实施反向工程来搭建 DbContext 和实体类型时，设置 DbSet 和集合导航名称的复数形式并设置表名称的单数形式。

#### <a name="new-behavior"></a>新行为

EF Core 现在包括使用 [Humanizer](https://github.com/Humanizr/Humanizer) 库的复数化程序。 这是 Visual Studio 用来推荐变量名称的同一个库。

#### <a name="why"></a>原因

对集合属性的单词使用复数形式并对类型和引用属性的单词使用单数形式在 .NET 中是惯用的。

#### <a name="mitigations"></a>缓解措施

若要禁用复数化程序，请使用 `dotnet ef dbcontext scaffold` 上的 `--no-pluralize` 选项或 `Scaffold-DbContext` 上的 `-NoPluralize` 开关。

<a name="inavigationbase"></a>

### <a name="inavigationbase-replaces-inavigation-in-some-apis-to-support-skip-navigations"></a>INavigationBase 替换某些 API 中的 INavigation 以支持跳过导航

[跟踪问题 #2568](https://github.com/dotnet/EntityFramework.Docs/issues/2568)

#### <a name="old-behavior"></a>旧行为

5\.0 之前的 EF Core 仅支持由 `INavigation` 接口表示的一种导航属性形式。

#### <a name="new-behavior"></a>新行为

EF Core 5.0 使用“跳过导航”引入了多对多关系。 这些关系由 `ISkipNavigation` 接口表示，`INavigation` 的大多数功能都向下推送到通用的基接口 `INavigationBase`。

#### <a name="why"></a>原因

常规导航和跳过导航的多数功能都相同。 但是，跳过导航与外键的关系不同于常规导航，因为涉及的 FK 不直接在关系的任意一端，而是在联接实体中。

#### <a name="mitigations"></a>缓解措施

在许多情况下，应用程序都可以切换为使用新的基接口，并且不会发生任何其他更改。 但是，如果使用导航访问外键属性，应将应用程序代码限制为仅使用常规导航，或者更新此代码以便执行适当的操作来实现常规导航和跳过导航。

<a name="collection-distinct-groupby"></a>

### <a name="some-queries-with-correlated-collection-that-also-use-distinct-or-groupby-are-no-longer-supported"></a>不再支持使用相关集合同时还使用 `Distinct` 或 `GroupBy` 的查询

[跟踪问题 #15873](https://github.com/dotnet/efcore/issues/15873)

**旧行为**

以前我们允许执行其中包含相关集合并且之后紧跟 `GroupBy` 的查询以及一些使用 `Distinct` 的查询。

GroupBy 示例：

```csharp
context.Parents
    .Select(p => p.Children
        .GroupBy(c => c.School)
        .Select(g => g.Key))
```

`Distinct` 示例 - 特别是内部集合投影不包含主键的 `Distinct` 查询：

```csharp
context.Parents
    .Select(p => p.Children
        .Select(c => c.School)
        .Distinct())
```

如果内部集合包含任何重复项，则这些查询可能会返回不正确的结果；如果内部集合中的所有元素都是唯一的，则这些查询可能会正常工作。

**新行为**

不再支持这些查询。 将引发异常，指示信息不足，因此无法正确生成结果。

**为什么**

如果使用相关集合，我们需要了解实体的主键，才能将集合实体分配到正确的父级中。 如果内部集合不使用 `GroupBy` 或 `Distinct`，只需将丢失的主键添加到投影中。 但是，如果使用 `GroupBy` 和 `Distinct`，则无法执行此操作，因为此操作会更改 `GroupBy` 或 `Distinct` 操作的结果。

**缓解措施**

将查询重写为不在内部集合上使用 `GroupBy` 或 `Distinct` 操作，改为在客户端执行这些操作。

```csharp
context.Parents
    .Select(p => p.Children.Select(c => c.School))
    .ToList()
    .Select(x => x.GroupBy(c => c).Select(g => g.Key))
```

```csharp
context.Parents
    .Select(p => p.Children.Select(c => c.School))
    .ToList()
    .Select(x => x.Distinct())
```

<a name="queryable-projection"></a>

### <a name="using-a-collection-of-queryable-type-in-projection-is-not-supported"></a>不支持在投影中使用可查询类型的集合

[跟踪问题 #16314](https://github.com/dotnet/efcore/issues/16314)

**旧行为**

以前，在某些情况下能够在投影中使用可查询类型的集合，例如用作 `List<T>` 构造函数的参数：

```csharp
context.Blogs
    .Select(b => new List<Post>(context.Posts.Where(p => p.BlogId == b.Id)))
```

**新行为**

不再支持这些查询。 将引发异常，指示无法创建可查询类型的对象，并建议如何解决此问题。

**为什么**

无法将可查询类型的对象具体化，因此它们会自动改用 `List<T>` 类型生成。 这样通常会由于类型不匹配而引发异常，一些用户对此异常不太了解，并且可能会感到非常意外。 我们决定识别此模式，并引发更有意义的异常。

**缓解措施**

在投影中的可查询对象之后添加 `ToList()` 调用：

```csharp
context.Blogs.Select(b => context.Posts.Where(p => p.BlogId == b.Id).ToList())
```
