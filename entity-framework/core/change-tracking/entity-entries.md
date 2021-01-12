---
title: 访问跟踪的实体-EF Core
description: 使用 EntityEntry、DbContext 和 DbSet 访问跟踪的实体
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/entity-entries
ms.openlocfilehash: f385016aba61535f33e34c622dd43ce6dc823fc5
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129579"
---
# <a name="accessing-tracked-entities"></a>访问跟踪的实体

可使用四个主要 Api 来访问由跟踪的实体 <xref:Microsoft.EntityFrameworkCore.DbContext> ：

- <xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType> 返回 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 给定实体实例的实例。
- <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%2A?displayProperty=nameWithType> 返回 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 所有已跟踪实体的实例，或返回给定类型的所有已跟踪实体的实例。
- <xref:Microsoft.EntityFrameworkCore.DbContext.Find%2A?displayProperty=nameWithType>、 <xref:Microsoft.EntityFrameworkCore.DbContext.FindAsync%2A?displayProperty=nameWithType> 、 <xref:Microsoft.EntityFrameworkCore.DbSet%601.Find%2A?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.DbSet%601.FindAsync%2A?displayProperty=nameWithType> 按主键查找单个实体，首先在跟踪的实体中查找，然后在需要时查询数据库。
- <xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> 返回由 DbSet 表示的实体类型的实体 (不 EntityEntry) 实例的实际实体。

下面的部分将对其中的每个进行更详细的介绍。

> [!TIP]
> 本文档假设了解 EF Core 更改跟踪的实体状态和基础知识。 有关这些主题的详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。

> [!TIP]
> 通过 [从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/AccessingTrackedEntities)，你可以运行并调试到本文档中的所有代码。

## <a name="using-dbcontextentry-and-entityentry-instances"></a>使用 DbContext 和 EntityEntry 实例

对于每个被跟踪的实体，Entity Framework Core (EF Core) 跟踪：

- 实体的总体状态。 这是 `Unchanged` 、 `Modified` 、或中的一个， `Added` `Deleted` 有关详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。
- 所跟踪的实体之间的关系。 例如，张贴内容所属的博客。
- 属性的 "当前值"。
- 如果此信息可用，则为属性的 "原始值"。 原始值是从数据库中查询实体时存在的属性值。
- 自查询后修改了哪些属性值。
- 有关属性值的其他信息，例如，值是否是 [临时](xref:core/change-tracking/miscellaneous#temporary-values)的。

传递实体实例将 <xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType> 导致为 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 给定实体提供对此信息的访问。 例如：

<!--
        using var context = new BlogsContext();

        var blog = context.Blogs.Single(e => e.Id == 1);
        var entityEntry = context.Entry(blog);

-->
[!code-csharp[Using_DbContext_Entry_and_EntityEntry_instances_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_DbContext_Entry_and_EntityEntry_instances_1)]

以下各节说明如何使用 EntityEntry 访问和操作实体状态以及实体的属性和导航状态。

### <a name="working-with-the-entity"></a>使用实体

最常见的用途 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 是访问实体的当前 <xref:Microsoft.EntityFrameworkCore.EntityState> 。 例如：

<!--
        var currentState = context.Entry(blog).State;
        if (currentState == EntityState.Unchanged)
        {
            context.Entry(blog).State = EntityState.Modified;
        }
-->
[!code-csharp[Work_with_the_entity_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_the_entity_1)]

条目方法还可用于尚未跟踪的实体。 这不 _会开始跟踪实体_;实体的状态仍为 `Detatched` 。 但是，随后可以使用返回的 EntityEntry 更改实体状态，此时将在给定状态下跟踪实体。 例如，以下代码将开始跟踪博客实例，如下所示 `Added` ：

<!--
        var newBlog = new Blog();
        Debug.Assert(context.Entry(newBlog).State == EntityState.Detached);

        context.Entry(newBlog).State = EntityState.Added;
        Debug.Assert(context.Entry(newBlog).State == EntityState.Added);
-->
[!code-csharp[Work_with_the_entity_2](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_the_entity_2)]

> [!TIP]
> 与在 EF6 中不同，设置单个实体的状态不会导致跟踪所有连接的实体。 这使得将此状态设置为较低级别操作的方式，而不是调用 `Add` 、 `Attach` 或，这会 `Update` 对整个实体图形执行操作。

下表总结了使用 EntityEntry 来处理整个实体的方法：

| EntityEntry 成员                                                                                         | 描述
|:-----------------------------------------------------------------------------------------------------------|----------------------
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.State?displayProperty=nameWithType>         | 获取和设置 <xref:Microsoft.EntityFrameworkCore.EntityState> 实体的。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Entity?displayProperty=nameWithType>        | 获取实体实例。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Context?displayProperty=nameWithType>       | <xref:Microsoft.EntityFrameworkCore.DbContext>正在跟踪此实体的。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Metadata?displayProperty=nameWithType>      | <xref:Microsoft.EntityFrameworkCore.Metadata.IEntityType> 实体类型的元数据。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.IsKeySet?displayProperty=nameWithType>      | 实体是否已设置其键值。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Reload?displayProperty=nameWithType>        | 用从数据库中读取的值覆盖属性值。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.DetectChanges?displayProperty=nameWithType> | 仅强制检测此实体的更改;请参阅 [更改检测和通知](xref:core/change-tracking/change-detection)。

### <a name="working-with-a-single-property"></a>使用单个属性

的多个重载 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Property%2A?displayProperty=nameWithType> 允许访问有关实体的各个属性的信息。 例如，使用强类型的、熟知的 API：

<!--
            PropertyEntry<Blog, string> propertyEntry = context.Entry(blog).Property(e => e.Name);
-->
[!code-csharp[Work_with_a_single_property_1a](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_property_1a)]

属性名称可以改为作为字符串传递。 例如：

<!--
            PropertyEntry<Blog, string> propertyEntry = context.Entry(blog).Property<string>("Name");
-->
[!code-csharp[Work_with_a_single_property_1b](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_property_1b)]

然后， <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602> 可以使用返回的来访问有关属性的信息。 例如，它可用于获取和设置此实体上的属性的当前值：

<!--
            string currentValue = context.Entry(blog).Property(e => e.Name).CurrentValue;
            context.Entry(blog).Property(e => e.Name).CurrentValue = "1unicorn2";
-->
[!code-csharp[Work_with_a_single_property_1d](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_property_1d)]

以上使用的两个属性方法都返回强类型的泛型 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602> 实例。 使用此泛型类型是首选方法，因为它允许无需 [装箱值类型](/dotnet/csharp/programming-guide/types/boxing-and-unboxing)即可访问属性值。 但是，如果实体或属性的类型在编译时未知，则 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry> 可以改为获取非泛型：

<!--
            var propertyEntry = context.Entry(blog).Property("Name");
-->
[!code-csharp[Work_with_a_single_property_1c](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_property_1c)]

这样，无论属性的类型如何，都可以访问任何属性的属性信息，但需支付值类型的费用。 例如：

<!--
            object blog = context.Blogs.Single(e => e.Id == 1);

            object currentValue = context.Entry(blog).Property("Name").CurrentValue;
            context.Entry(blog).Property("Name").CurrentValue = "1unicorn2";
-->
[!code-csharp[Work_with_a_single_property_1e](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_property_1e)]

下表汇总了由 PropertyEntry 公开的属性信息：

| PropertyEntry 成员                               | 描述
|:-------------------------------------------------|----------------------
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602.CurrentValue?displayProperty=nameWithType>  | 获取和设置属性的当前值。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602.OriginalValue?displayProperty=nameWithType> | 获取并设置属性的原始值（如果可用）。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry%602.EntityEntry?displayProperty=nameWithType>   | 对实体的的反向引用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.Metadata?displayProperty=nameWithType>          | <xref:Microsoft.EntityFrameworkCore.Metadata.IProperty> 属性的元数据。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsModified?displayProperty=nameWithType>        | 指示此属性是否被标记为已修改，并允许更改此状态。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsTemporary?displayProperty=nameWithType>       | 指示此属性是否标记为 [临时](xref:core/change-tracking/miscellaneous#temporary-values#temporary-values)，并允许更改此状态。

说明：

- 属性的原始值是从数据库中查询实体时属性具有的值。 但是，如果实体已断开连接，然后显式附加到另一个 DbContext （例如，使用或），则原始值不可用 `Attach` `Update` 。 在这种情况下，返回的原始值将与当前的值相同。
- <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 只会更新标记为已修改的属性。 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsModified>如果设置为 true，则强制 EF Core 更新给定属性值，或将其设置为 false 以防止 EF Core 更新属性值。
- [临时值](xref:core/change-tracking/miscellaneous) 通常由 EF Core [值生成器](xref:core/modeling/generated-properties)生成。 设置属性的当前值会将临时值替换为给定的值，并将该属性标记为不是临时的。 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsTemporary>如果设置为 true，则强制即使在显式设置值后也将值设置为 "临时"。

### <a name="working-with-a-single-navigation"></a>使用单个导航

、和的多个重载 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Reference%2A?displayProperty=nameWithType> <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Collection%2A?displayProperty=nameWithType> <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Navigation%2A?displayProperty=nameWithType> 允许访问有关单个导航的信息。

通过方法访问指向单个相关实体的引用导航 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Reference%2A> 。 引用导航点指向一对多关系的 "一" 方，并指向一对一关系的两侧。 例如：

<!--
        ReferenceEntry<Post, Blog> referenceEntry1 = context.Entry(post).Reference(e => e.Blog);
        ReferenceEntry<Post, Blog> referenceEntry2 = context.Entry(post).Reference<Blog>("Blog");
        ReferenceEntry referenceEntry3 = context.Entry(post).Reference("Blog");
-->
[!code-csharp[Work_with_a_single_navigation_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_navigation_1)]

当用于一对多和多对多关系的 "多" 方时，导航也可以是相关实体的集合。 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601.Collection%2A>方法用于访问集合导航。 例如：

<!--
        CollectionEntry<Blog, Post> collectionEntry1 = context.Entry(blog).Collection(e => e.Posts);
        CollectionEntry<Blog, Post> collectionEntry2 = context.Entry(blog).Collection<Post>("Posts");
        CollectionEntry collectionEntry3 = context.Entry(blog).Collection("Posts");
-->
[!code-csharp[Work_with_a_single_navigation_2a](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_navigation_2a)]

某些操作对于所有导航都很常见。 使用方法可同时访问引用和集合导航 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Navigation%2A?displayProperty=nameWithType> 。 请注意，在同时访问所有导航时，仅可使用非泛型访问。 例如：

<!--
        NavigationEntry navigationEntry = context.Entry(blog).Navigation("Posts");
-->
[!code-csharp[Work_with_a_single_navigation_2b](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_a_single_navigation_2b)]

下表总结了使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ReferenceEntry%602> 、和的方法 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.CollectionEntry%602> <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry> ：

| NavigationEntry 成员                                                                                    | 描述
|:----------------------------------------------------------------------------------------------------------|----------------------
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.MemberEntry.CurrentValue?displayProperty=nameWithType> | 获取和设置导航的当前值。 这是集合导航的整个集合。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry.Metadata?displayProperty=nameWithType> | <xref:Microsoft.EntityFrameworkCore.Metadata.INavigationBase> 导航的元数据。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry.IsLoaded?displayProperty=nameWithType> | 获取或设置一个值，该值指示是否已从数据库完全加载相关实体或集合。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry.Load?displayProperty=nameWithType>     | 从数据库加载相关实体或集合;请参阅 [显式加载相关数据](xref:core/querying/related-data/explicit)。
| <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry.Query?displayProperty=nameWithType>    | 查询 EF Core 将使用将此导航作为 `IQueryable` 可以进一步组合的来加载，请参阅 [显式加载相关数据](xref:core/querying/related-data/explicit)。

### <a name="working-with-all-properties-of-an-entity"></a>使用实体的所有属性

<xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Properties?displayProperty=nameWithType><xref:System.Collections.Generic.IEnumerable%601> <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry> 为实体的每个属性返回的。 这可用于为实体的每个属性执行操作。 例如，若要将任何 DateTime 属性设置为 `DateTime.Now` ：

<!--
        foreach (var propertyEntry in context.Entry(blog).Properties)
        {
            if (propertyEntry.Metadata.ClrType == typeof(DateTime))
            {
                propertyEntry.CurrentValue = DateTime.Now;
            }
        }
-->
[!code-csharp[Work_with_all_properties_of_an_entity_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_1)]

此外，EntityEntry 包含多个用于同时获取和设置所有属性值的方法。 这些方法使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues> 类，该类表示属性及其值的集合。 可以获取当前或原始值的 PropertyValues，也可以为数据库中当前存储的值获取。 例如：

<!--
        var currentValues = context.Entry(blog).CurrentValues;
        var originalValues = context.Entry(blog).OriginalValues;
        var databaseValues = context.Entry(blog).GetDatabaseValues();
-->
[!code-csharp[Work_with_all_properties_of_an_entity_2a](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_2a)]

这些 PropertyValues 对象本身并不十分有用。 但是，它们可以组合起来执行操作实体时所需的常见操作。 当处理数据传输对象以及解决 [开放式并发冲突](xref:core/saving/concurrency)时，这非常有用。 以下部分演示了一些示例。

#### <a name="setting-current-or-original-values-from-an-entity-or-dto"></a>设置实体或 DTO 的当前值或原始值

可以通过从另一个对象复制值来更新实体的当前值或原始值。 例如，请考虑一个 `BlogDto` 数据传输对象 (DTO) ，该对象具有与实体类型相同的属性：

<!--
public class BlogDto
{
    public int Id { get; set; }
    public string Name { get; set; }
}
-->
[!code-csharp[BlogDto](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=BlogDto)]

这可用于设置所跟踪实体的当前值，使用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues.SetValues%2A?displayProperty=nameWithType> ：

<!--
        var blogDto = new BlogDto { Id = 1, Name = "1unicorn2" };

        context.Entry(blog).CurrentValues.SetValues(blogDto);
-->
[!code-csharp[Work_with_all_properties_of_an_entity_2b](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_2b)]

当使用从服务调用或 n 层应用程序中的客户端获取的值更新实体时，有时会使用此方法。 请注意，所使用的对象不必与实体具有相同的类型，但前提是它具有与实体的名称相匹配的属性。 在上面的示例中，DTO 的实例 `BlogDto` 用于设置所跟踪实体的当前值 `Blog` 。

请注意，仅当值设置与当前值不同时，才将属性标记为已修改。

#### <a name="setting-current-or-original-values-from-a-dictionary"></a>从字典设置当前值或原始值

前面的示例从实体或 DTO 实例设置值。 当属性值作为名称/值对存储在字典中时，可以使用相同的行为。 例如：

<!--
        var blogDictionary = new Dictionary<string, object>
        {
            ["Id"] = 1,
            ["Name"] = "1unicorn2"
        };

        context.Entry(blog).CurrentValues.SetValues(blogDictionary);
-->
[!code-csharp[Work_with_all_properties_of_an_entity_2d](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_2d)]

#### <a name="setting-current-or-original-values-from-the-database"></a>从数据库设置当前值或原始值

通过调用 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.GetDatabaseValues> 或 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.GetDatabaseValuesAsync%2A> 并使用返回的对象设置当前值或原始值，或者同时使用这两个值，可以使用数据库中的最新值更新实体的当前值或原始值。 例如：

<!--
        var databaseValues = context.Entry(blog).GetDatabaseValues();
        context.Entry(blog).CurrentValues.SetValues(databaseValues);
        context.Entry(blog).OriginalValues.SetValues(databaseValues);
-->
[!code-csharp[Work_with_all_properties_of_an_entity_2c](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_2c)]

#### <a name="creating-a-cloned-object-containing-current-original-or-database-values"></a>创建包含当前、原始或数据库值的克隆对象

从 CurrentValues、OriginalValues 或 GetDatabaseValues 返回的 PropertyValues 对象可用于通过创建实体的克隆 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues.ToObject?displayProperty=nameWithType> 。 例如：

<!--
var clonedBlog = context.Entry(blog).GetDatabaseValues().ToObject();
-->
[!code-csharp[Work_with_all_properties_of_an_entity_2e](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_properties_of_an_entity_2e)]

请注意，将 `ToObject` 返回一个不由 DbContext 跟踪的新实例。 返回的对象也不会将任何关系设置为其他实体。

克隆的对象可用于解决与数据库的并发更新相关的问题，尤其是当数据绑定到特定类型的对象时。 有关详细信息，请参阅 [乐观并发](xref:core/saving/concurrency) 。

### <a name="working-with-all-navigations-of-an-entity"></a>使用实体的所有导航

<xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Navigations?displayProperty=nameWithType><xref:System.Collections.Generic.IEnumerable%601> <xref:Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry> 对于实体的每个导航，返回的。 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.References?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Collections?displayProperty=nameWithType> 执行相同的操作，但限制为分别引用或收集导航。 这可用于为实体的每个导航执行操作。 例如，若要强制加载所有相关实体：

<!--
        foreach (var navigationEntry in context.Entry(blog).Navigations)
        {
            navigationEntry.Load();
        }
-->
[!code-csharp[Work_with_all_navigations_of_an_entity_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_navigations_of_an_entity_1)]

### <a name="working-with-all-members-of-an-entity"></a>使用实体的所有成员

常规属性和导航属性具有不同的状态和行为。 因此，可以单独处理导航和非导航，如以上部分所示。 但有时，对实体的任何成员执行某些操作都很有用，无论它是常规属性还是导航。 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Member%2A?displayProperty=nameWithType><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Members?displayProperty=nameWithType>出于此目的提供了和。 例如：

<!--
        foreach (var memberEntry in context.Entry(blog).Members)
        {
            Console.WriteLine(
                $"Member {memberEntry.Metadata.Name} is of type {memberEntry.Metadata.ClrType.ShortDisplayName()} and has value {memberEntry.CurrentValue}");
        }
-->
[!code-csharp[Work_with_all_members_of_an_entity_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Work_with_all_members_of_an_entity_1)]

在示例中，在博客上运行此代码将生成以下输出：

```output
Member Id is of type int and has value 1
Member Name is of type string and has value .NET Blog
Member Posts is of type IList<Post> and has value System.Collections.Generic.List`1[Post]
```

> [!TIP]
> [更改跟踪](xref:core/change-tracking/debug-views)器的 "调试" 视图将显示类似于下面的信息。 整个更改跟踪器的 "调试" 视图是从 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.DebugView?displayProperty=nameWithType> 每个被跟踪实体的个体生成的。

## <a name="find-and-findasync"></a>Find 和 FindAsync

<xref:Microsoft.EntityFrameworkCore.DbContext.Find%2A?displayProperty=nameWithType>、 <xref:Microsoft.EntityFrameworkCore.DbContext.FindAsync%2A?displayProperty=nameWithType> 、 <xref:Microsoft.EntityFrameworkCore.DbSet%601.Find%2A?displayProperty=nameWithType> 和 <xref:Microsoft.EntityFrameworkCore.DbSet%601.FindAsync%2A?displayProperty=nameWithType> 旨在有效地查找其主键已知的单个实体。 查找首先检查是否已跟踪实体，如果是，则立即返回实体。 仅当未在本地跟踪实体时才会进行数据库查询。 例如，请考虑下面这段代码：

<!--
        using var context = new BlogsContext();

        Console.WriteLine("First call to Find...");
        var blog1 = context.Blogs.Find(1);

        Console.WriteLine($"...found blog {blog1.Name}");

        Console.WriteLine();
        Console.WriteLine("Second call to Find...");
        var blog2 = context.Blogs.Find(1);
        Debug.Assert(blog1 == blog2);

        Console.WriteLine("...returned the same instance without executing a query.");
-->
[!code-csharp[Find_and_FindAsync_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Find_and_FindAsync_1)]

使用 SQLite 时，此代码的输出 (包括 EF Core 日志记录) ：

```output
First call to Find...
info: 12/29/2020 07:45:53.682 RelationalEventId.CommandExecuted[20101] (Microsoft.EntityFrameworkCore.Database.Command)
      Executed DbCommand (1ms) [Parameters=[@__p_0='1' (DbType = String)], CommandType='Text', CommandTimeout='30']
      SELECT "b"."Id", "b"."Name"
      FROM "Blogs" AS "b"
      WHERE "b"."Id" = @__p_0
      LIMIT 1
...found blog .NET Blog

Second call to Find...
...returned the same instance without executing a query.
```

请注意，第一次调用在本地找不到实体，因此执行数据库查询。 相反，第二次调用返回相同的实例而不查询数据库，因为它已被跟踪。

如果具有给定键的实体未在本地跟踪并且数据库中不存在，则 Find 将返回 null。

### <a name="composite-keys"></a>组合键

"查找" 还可用于组合键。 例如，考虑一个 `OrderLine` 具有由订单 ID 和产品 ID 组成的组合键的实体：

<!--
public class OrderLine
{
    public int OrderId { get; set; }
    public int ProductId { get; set; }

    //...
}
-->
[!code-csharp[OrderLine](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=OrderLine)]

必须在中配置组合键 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A?displayProperty=nameWithType> ，才能定义关键部分 _及其顺序_。 例如：

<!--
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder
            .Entity<OrderLine>()
            .HasKey(e => new { e.OrderId, e.ProductId });
    }
-->
[!code-csharp[OnModelCreating](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=OnModelCreating)]

请注意， `OrderId` 是键的第一部分， `ProductId` 是密钥的第二部分。 传递要查找的键值时，必须使用此顺序。 例如：

<!--
        var orderline = context.OrderLines.Find(orderId, productId);
-->
[!code-csharp[Find_and_FindAsync_2](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Find_and_FindAsync_2)]

## <a name="using-changetrackerentries-to-access-all-tracked-entities"></a>使用 ChangeTracker 访问所有跟踪的实体

到目前为止，我们只是一次访问了一个 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry> 。 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries?displayProperty=nameWithType> 为 DbContext 当前跟踪的每个实体返回 EntityEntry。 例如：

<!--
        using var context = new BlogsContext();
        var blogs = context.Blogs.Include(e => e.Posts).ToList();

        foreach (var entityEntry in context.ChangeTracker.Entries())
        {
            Console.WriteLine($"Found {entityEntry.Metadata.Name} entity with ID {entityEntry.Property("Id").CurrentValue}");
        }
-->
[!code-csharp[Using_ChangeTracker_Entries_to_access_all_tracked_entities_1a](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_ChangeTracker_Entries_to_access_all_tracked_entities_1a)]

此代码生成以下输出：

```output
Found Blog entity with ID 1
Found Post entity with ID 1
Found Post entity with ID 2
```

请注意，将返回博客和帖子的条目。 可以改用泛型重载，将结果筛选为特定实体类型 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries%60%601?displayProperty=nameWithType> ：

<!--
        foreach (var entityEntry in context.ChangeTracker.Entries<Post>())
        {
            Console.WriteLine(
                $"Found {entityEntry.Metadata.Name} entity with ID {entityEntry.Property(e => e.Id).CurrentValue}");
        }
-->
[!code-csharp[Using_ChangeTracker_Entries_to_access_all_tracked_entities_1b](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_ChangeTracker_Entries_to_access_all_tracked_entities_1b)]

此代码的输出显示仅返回 post：

```output
Found Post entity with ID 1
Found Post entity with ID 2
```

此外，使用泛型重载会返回泛型 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry%601> 实例。 这就允许 `Id` 在此示例中对属性的访问权限。

用于筛选的泛型类型不一定是映射实体类型;可以改为使用未映射的基类型或接口。 例如，如果模型中的所有实体类型都实现定义其键属性的接口：

<!--
public interface IEntityWithKey
{
    int Id { get; set; }
}
-->
[!code-csharp[IEntityWithKey](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=IEntityWithKey)]

然后，可以使用此接口以强类型方式处理任何跟踪实体的键。 例如：

<!--
        foreach (var entityEntry in context.ChangeTracker.Entries<IEntityWithKey>())
        {
            Console.WriteLine(
                $"Found {entityEntry.Metadata.Name} entity with ID {entityEntry.Property(e => e.Id).CurrentValue}");
        }
-->
[!code-csharp[Using_ChangeTracker_Entries_to_access_all_tracked_entities_1c](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_ChangeTracker_Entries_to_access_all_tracked_entities_1c)]

## <a name="using-dbsetlocal-to-query-tracked-entities"></a>使用 DbSet 查询跟踪的实体

EF Core 查询始终在数据库上执行，并且仅返回已保存到数据库的实体。 <xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> 提供了一种机制，用于在 DbContext 中查询本地跟踪实体。

由于 `DbSet.Local` 用于查询跟踪的实体，因此通常会将实体加载到 DbContext 中，然后使用已加载的实体。 这对于数据绑定尤其如此，但在其他情况下也很有用。 例如，在以下代码中，首先查询数据库以获取所有博客和帖子。 <xref:Microsoft.EntityFrameworkCore.EntityFrameworkQueryableExtensions.Load%2A>扩展方法用于使用上下文跟踪的结果来执行此查询，而不会直接返回到应用程序。 使用 `ToList` 或类似 (具有相同的效果，但会产生创建返回列表的开销，但此处不需要这样做。 ) 该示例然后使用 `DbSet.Local` 访问本地跟踪的实体：

<!--
        using var context = new BlogsContext();

        context.Blogs.Include(e => e.Posts).Load();

        foreach (var blog in context.Blogs.Local)
        {
            Console.WriteLine($"Blog: {blog.Name}");
        }

        foreach (var post in context.Posts.Local)
        {
            Console.WriteLine($"Post: {post.Title}");
        }
-->
[!code-csharp[Using_DbSet_Local_to_query_tracked_entities_1](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_DbSet_Local_to_query_tracked_entities_1)]

请注意，与不同的是 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.Entries?displayProperty=nameWithType> ， `DbSet.Local` 直接返回实体实例。 当然，EntityEntry 可以通过调用来获取返回的实体 <xref:System.Data.Entity.DbContext.Entry%2A?displayProperty=nameWithType> 。

### <a name="the-local-view"></a>本地视图

<xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> 返回用于反映这些实体当前的本地跟踪实体的视图 <xref:Microsoft.EntityFrameworkCore.EntityState> 。 具体而言，这意味着：

- `Added` 实体包括在内。 请注意，对于普通 EF Core 查询，这种情况并非如此，因为 `Added` 实体不存在于数据库中，因此不会由数据库查询返回。
- `Deleted` 排除实体。 请注意，对于普通 EF Core 查询，这种情况并不是这样，因为 `Deleted` 实体仍然存在于数据库中，并且由数据库查询返回。

所有这些都是指对 `DbSet.Local` 反映实体关系图当前概念状态的数据的查看，其中 `Added` 包含实体和 `Deleted` 实体。 这与调用 SaveChanges 后预期的数据库状态一致。

这通常是数据绑定的理想视图，因为它会根据应用程序所做的更改向用户提供数据。

下面的代码演示，我将一个 post 标记为 `Deleted` ，然后添加一个新帖子，将其标记为 `Added` ：

<!--
        using var context = new BlogsContext();

        var posts = context.Posts.Include(e => e.Blog).ToList();

        Console.WriteLine("Local view after loading posts:");

        foreach (var post in context.Posts.Local)
        {
            Console.WriteLine($"  Post: {post.Title}");
        }

        context.Remove(posts[1]);

        context.Add(new Post
        {
            Title = "What’s next for System.Text.Json?",
            Content = ".NET 5.0 was released recently and has come with many...",
            Blog = posts[0].Blog
        });

        Console.WriteLine("Local view after adding and deleting posts:");

        foreach (var post in context.Posts.Local)
        {
            Console.WriteLine($"  Post: {post.Title}");
        }
-->
[!code-csharp[Using_DbSet_Local_to_query_tracked_entities_2](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_DbSet_Local_to_query_tracked_entities_2)]

此代码的输出为：

```output
Local view after loading posts:
  Post: Announcing the Release of EF Core 5.0
  Post: Announcing F# 5
  Post: Announcing .NET 5.0
Local view after adding and deleting posts:
  Post: What’s next for System.Text.Json?
  Post: Announcing the Release of EF Core 5.0
  Post: Announcing .NET 5.0
```

请注意，已删除的 post 将从本地视图中删除，并包含添加的帖子。

### <a name="using-local-to-add-and-remove-entities"></a>使用本地添加和删除实体

<xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> 返回 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.LocalView%601> 的实例。 这是 <xref:System.Collections.Generic.ICollection%601> 在从集合中添加或删除实体时生成并响应通知的实现。  (这是与相同的概念 <xref:System.Collections.ObjectModel.ObservableCollection%601> ，但它是在现有 EF Core 更改跟踪条目的投影上实现的，而不是作为独立的集合实现的。 ) 

本地视图的通知挂钩到 DbContext 更改跟踪，使本地视图与 DbContext 保持同步。 具体而言：

- 添加新实体以 `DbSet.Local` 使其由 DbContext 跟踪，通常处于 `Added` 状态。  (如果实体已有生成的键值，则会改为跟踪该实体值 `Unchanged` 。 ) 
- 从中移除实体 `DbSet.Local` 会使其标记为 `Deleted` 。
- DbContext 跟踪的实体会自动出现在 `DbSet.Local` 集合中。 例如，如果执行查询来自动引入多个实体，则会使本地视图更新。
- 标记为的实体 `Deleted` 将自动从本地集合中删除。

这意味着，只需通过在集合中添加和移除，就可以使用本地视图来处理跟踪的实体。 例如，我们修改前面的示例代码以添加和删除本地集合中的帖子：

<!--
        using var context = new BlogsContext();

        var posts = context.Posts.Include(e => e.Blog).ToList();

        Console.WriteLine("Local view after loading posts:");

        foreach (var post in context.Posts.Local)
        {
            Console.WriteLine($"  Post: {post.Title}");
        }

        context.Posts.Local.Remove(posts[1]);

        context.Posts.Local.Add(new Post
        {
            Title = "What’s next for System.Text.Json?",
            Content = ".NET 5.0 was released recently and has come with many...",
            Blog = posts[0].Blog
        });

        Console.WriteLine("Local view after adding and deleting posts:");

        foreach (var post in context.Posts.Local)
        {
            Console.WriteLine($"  Post: {post.Title}");
        }
-->
[!code-csharp[Using_DbSet_Local_to_query_tracked_entities_3](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_DbSet_Local_to_query_tracked_entities_3)]

由于对本地视图所做的更改与 DbContext 同步，因此输出在上一个示例中保持不变。

### <a name="using-the-local-view-for-windows-forms-or-wpf-data-binding"></a>使用 Windows 窗体或 WPF 数据绑定的本地视图

<xref:Microsoft.EntityFrameworkCore.DbSet%601.Local?displayProperty=nameWithType> 构成用于 EF Core 实体的数据绑定的基础。 不过，Windows 窗体和 WPF 在与他们期望的特定类型的通知集合一起使用时，效果最佳。 本地视图支持创建以下特定的集合类型：

- <xref:Microsoft.EntityFrameworkCore.ChangeTracking.LocalView%601.ToObservableCollection?displayProperty=nameWithType> 返回 <xref:System.Collections.ObjectModel.ObservableCollection%601> WPF 数据绑定的。
- <xref:Microsoft.EntityFrameworkCore.ChangeTracking.LocalView%601.ToBindingList?displayProperty=nameWithType> 返回 <xref:System.ComponentModel.BindingList%601> Windows 窗体数据绑定。

例如：

<!--
        ObservableCollection<Post> observableCollection = context.Posts.Local.ToObservableCollection();
        BindingList<Post> bindingList = context.Posts.Local.ToBindingList();
-->
[!code-csharp[Using_DbSet_Local_to_query_tracked_entities_4](../../../samples/core/ChangeTracking/AccessingTrackedEntities/Samples.cs?name=Using_DbSet_Local_to_query_tracked_entities_4)]

有关与 EF Core 的 WPF 数据绑定的详细信息，请参阅 [Wpf 入门](xref:core/get-started/wpf) 。

> [!TIP]
> 给定 DbSet 实例的本地视图在第一次访问后延迟创建，然后进行缓存。 LocalView 创建本身的速度很快，并且不会占用大量内存。 不过，它确实调用了 [DetectChanges](xref:core/change-tracking/change-detection)，这对于大量实体可能会很慢。 和创建的集合 `ToObservableCollection` `ToBindingList` 还会按延迟创建，然后进行缓存。 这两种方法都创建新的集合，这可能会很慢，并且会在涉及数千个实体时使用大量内存。
