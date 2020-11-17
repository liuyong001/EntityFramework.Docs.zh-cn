---
title: 相关数据的预先加载 - EF Core
description: 使用 Entity Framework Core 预先加载相关数据
author: roji
ms.date: 9/8/2020
uid: core/querying/related-data/eager
ms.openlocfilehash: bd9c9045c1c2707d69ee4070bea59ad8066789f3
ms.sourcegitcommit: f3512e3a98e685a3ba409c1d0157ce85cc390cf4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2020
ms.locfileid: "94430100"
---
# <a name="eager-loading-of-related-data"></a>相关数据的预先加载

## <a name="eager-loading"></a>预先加载

可以使用 `Include` 方法来指定要包含在查询结果中的关联数据。 在以下示例中，结果中返回的blogs将使用关联的posts填充其 `Posts` 属性。

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#SingleInclude)]

> [!TIP]
> Entity Framework Core 会根据之前已加载到上下文实例中的实体自动填充导航属性。 因此，即使不显式包含导航属性的数据，如果先前加载了部分或所有关联实体，则仍可能填充该属性。

可以在单个查询中包含多个关系的关联数据。

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#MultipleIncludes)]

> [!CAUTION]
> 急于在单个查询中加载集合导航可能会导致性能问题。 有关详细信息，请参阅[单个查询和拆分查询](xref:core/querying/single-split-queries)。

## <a name="including-multiple-levels"></a>包含多个层级

使用 `ThenInclude` 方法可以依循关系包含多个层级的关联数据。 以下示例加载了所有博客、其关联文章及每篇文章的作者。

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#SingleThenInclude)]

可通过链式调用 `ThenInclude`，进一步包含更深级别的关联数据。

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#MultipleThenIncludes)]

可以将对来自多个级别和多个根的关联数据的所有调用合并到同一查询中。

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#IncludeTree)]

你可能希望将已包含的某个实体的多个关联实体都包含进来。 例如，当查询 `Blogs` 时，你会包含 `Posts`，然后希望同时包含 `Posts` 的 `Author` 和 `Tags`。 为了包含这两项内容，需要从根级别开始指定每个包含路径。 例如，`Blog -> Posts -> Author` 和 `Blog -> Posts -> Tags`。 这并不意味着会获得冗余联接查询，在大多数情况下，EF 会在生成 SQL 时合并相应的联接查询。

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#MultipleLeafIncludes)]

## <a name="filtered-include"></a>经过筛选的包含

> [!NOTE]
> EF Core 5.0 中已引入了此功能。

在应用包含功能来加载相关数据时，可对已包含的集合导航应用某些可枚举的操作，这样就可对结果进行筛选和排序。

支持的操作包括：`Where`、`OrderBy`、`OrderByDescending`、`ThenBy`、`ThenByDescending`、`Skip` 和 `Take`。

应对传递到 Include 方法的 Lambda 中的集合导航应用这类操作，如下例所示：

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#FilteredInclude)]

只能对每个包含的导航执行一组唯一的筛选器操作。 如果为某个给定的集合导航应用了多个包含操作（下例中为 `blog.Posts`），则只能对其中一个导航指定筛选器操作：

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#MultipleLeafIncludesFiltered1)]

可对多次包含的每个导航应用相同的操作：

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#MultipleLeafIncludesFiltered2)]

> [!CAUTION]
> 在跟踪查询时，由于[导航修正](xref:core/querying/tracking)，Filtered Include 的结果可能不符合预期。 之前已查询且已存储在更改跟踪器的所有相关实体都将在 Filtered Include 查询的结果中显示，即使它们不符合筛选器的要求也是如此。 在这些情况下使用 Filtered Include 时，请考虑使用 `NoTracking` 查询或重新创建 DbContext。

示例：

```csharp
var orders = context.Orders.Where(o => o.Id > 1000).ToList();

// customer entities will have references to all orders where Id > 1000, rather than > 5000
var filtered = context.Customers.Include(c => c.Orders.Where(o => o.Id > 5000)).ToList();
```

> [!NOTE]
> 在跟踪查询的情况下，应用了经过筛选的包含的导航被视为已加载。 这意味着 EF Core 不会尝试使用[显式加载](xref:core/querying/related-data/explicit)或[延迟加载](xref:core/querying/related-data/lazy)来重新加载其值，即使某些元素仍然可能缺失也不会尝试。

## <a name="include-on-derived-types"></a>派生类型上的包含

可以使用 `Include` 和 `ThenInclude` 包含仅在派生类型上定义的导航的关联数据。

给定以下模型：

```csharp
public class SchoolContext : DbContext
{
    public DbSet<Person> People { get; set; }
    public DbSet<School> Schools { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<School>().HasMany(s => s.Students).WithOne(s => s.School);
    }
}

public class Person
{
    public int Id { get; set; }
    public string Name { get; set; }
}

public class Student : Person
{
    public School School { get; set; }
}

public class School
{
    public int Id { get; set; }
    public string Name { get; set; }

    public List<Student> Students { get; set; }
}
```

所有人员（可以使用许多模式预先加载的学生）的 `School` 导航的内容：

* 使用强制转换

  ```csharp
  context.People.Include(person => ((Student)person).School).ToList()
  ```

* 使用 `as` 运算符

  ```csharp
  context.People.Include(person => (person as Student).School).ToList()
  ```

* 使用 `Include` 的重载，该方法采用 `string` 类型的参数

  ```csharp
  context.People.Include("School").ToList()
  ```
