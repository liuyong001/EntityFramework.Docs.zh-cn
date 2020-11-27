---
title: 相关数据的预先加载 - EF Core
description: 使用 Entity Framework Core 预先加载相关数据
author: roji
ms.date: 9/8/2020
uid: core/querying/related-data/eager
ms.openlocfilehash: 66956fcd85bb21a08c69fa93b93c12382bbfc8eb
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003570"
---
# <a name="eager-loading-of-related-data"></a><span data-ttu-id="6ad08-103">相关数据的预先加载</span><span class="sxs-lookup"><span data-stu-id="6ad08-103">Eager Loading of Related Data</span></span>

## <a name="eager-loading"></a><span data-ttu-id="6ad08-104">预先加载</span><span class="sxs-lookup"><span data-stu-id="6ad08-104">Eager loading</span></span>

<span data-ttu-id="6ad08-105">可以使用 `Include` 方法来指定要包含在查询结果中的关联数据。</span><span class="sxs-lookup"><span data-stu-id="6ad08-105">You can use the `Include` method to specify related data to be included in query results.</span></span> <span data-ttu-id="6ad08-106">在以下示例中，结果中返回的blogs将使用关联的posts填充其 `Posts` 属性。</span><span class="sxs-lookup"><span data-stu-id="6ad08-106">In the following example, the blogs that are returned in the results will have their `Posts` property populated with the related posts.</span></span>

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#SingleInclude)]

> [!TIP]
> <span data-ttu-id="6ad08-107">Entity Framework Core 会根据之前已加载到上下文实例中的实体自动填充导航属性。</span><span class="sxs-lookup"><span data-stu-id="6ad08-107">Entity Framework Core will automatically fix-up navigation properties to any other entities that were previously loaded into the context instance.</span></span> <span data-ttu-id="6ad08-108">因此，即使不显式包含导航属性的数据，如果先前加载了部分或所有关联实体，则仍可能填充该属性。</span><span class="sxs-lookup"><span data-stu-id="6ad08-108">So even if you don't explicitly include the data for a navigation property, the property may still be populated if some or all of the related entities were previously loaded.</span></span>

<span data-ttu-id="6ad08-109">可以在单个查询中包含多个关系的关联数据。</span><span class="sxs-lookup"><span data-stu-id="6ad08-109">You can include related data from multiple relationships in a single query.</span></span>

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#MultipleIncludes)]

> [!CAUTION]
> <span data-ttu-id="6ad08-110">急于在单个查询中加载集合导航可能会导致性能问题。</span><span class="sxs-lookup"><span data-stu-id="6ad08-110">Eager loading a collection navigation in a single query may cause performance issues.</span></span> <span data-ttu-id="6ad08-111">有关详细信息，请参阅[单个查询和拆分查询](xref:core/querying/single-split-queries)。</span><span class="sxs-lookup"><span data-stu-id="6ad08-111">For more information, see [Single vs. split queries](xref:core/querying/single-split-queries).</span></span>

## <a name="including-multiple-levels"></a><span data-ttu-id="6ad08-112">包含多个层级</span><span class="sxs-lookup"><span data-stu-id="6ad08-112">Including multiple levels</span></span>

<span data-ttu-id="6ad08-113">使用 `ThenInclude` 方法可以依循关系包含多个层级的关联数据。</span><span class="sxs-lookup"><span data-stu-id="6ad08-113">You can drill down through relationships to include multiple levels of related data using the `ThenInclude` method.</span></span> <span data-ttu-id="6ad08-114">以下示例加载了所有博客、其关联文章及每篇文章的作者。</span><span class="sxs-lookup"><span data-stu-id="6ad08-114">The following example loads all blogs, their related posts, and the author of each post.</span></span>

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#SingleThenInclude)]

<span data-ttu-id="6ad08-115">可通过链式调用 `ThenInclude`，进一步包含更深级别的关联数据。</span><span class="sxs-lookup"><span data-stu-id="6ad08-115">You can chain multiple calls to `ThenInclude` to continue including further levels of related data.</span></span>

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#MultipleThenIncludes)]

<span data-ttu-id="6ad08-116">可以将对来自多个级别和多个根的关联数据的所有调用合并到同一查询中。</span><span class="sxs-lookup"><span data-stu-id="6ad08-116">You can combine all of the calls to include related data from multiple levels and multiple roots in the same query.</span></span>

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#IncludeTree)]

<span data-ttu-id="6ad08-117">你可能希望将已包含的某个实体的多个关联实体都包含进来。</span><span class="sxs-lookup"><span data-stu-id="6ad08-117">You may want to include multiple related entities for one of the entities that is being included.</span></span> <span data-ttu-id="6ad08-118">例如，当查询 `Blogs` 时，你会包含 `Posts`，然后希望同时包含 `Posts` 的 `Author` 和 `Tags`。</span><span class="sxs-lookup"><span data-stu-id="6ad08-118">For example, when querying `Blogs`, you include `Posts` and then want to include both the `Author` and `Tags` of the `Posts`.</span></span> <span data-ttu-id="6ad08-119">为了包含这两项内容，需要从根级别开始指定每个包含路径。</span><span class="sxs-lookup"><span data-stu-id="6ad08-119">To include both, you need to specify each include path starting at the root.</span></span> <span data-ttu-id="6ad08-120">例如，`Blog -> Posts -> Author` 和 `Blog -> Posts -> Tags`。</span><span class="sxs-lookup"><span data-stu-id="6ad08-120">For example, `Blog -> Posts -> Author` and `Blog -> Posts -> Tags`.</span></span> <span data-ttu-id="6ad08-121">这并不意味着会获得冗余联接查询，在大多数情况下，EF 会在生成 SQL 时合并相应的联接查询。</span><span class="sxs-lookup"><span data-stu-id="6ad08-121">It doesn't mean you'll get redundant joins; in most cases, EF will combine the joins when generating SQL.</span></span>

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#MultipleLeafIncludes)]

> [!TIP]
> <span data-ttu-id="6ad08-122">还可以使用单个 `Include` 方法加载多个导航。</span><span class="sxs-lookup"><span data-stu-id="6ad08-122">You can also load multiple navigations using a single `Include` method.</span></span> <span data-ttu-id="6ad08-123">对于作为所有引用的导航“链”，或者当它们以单个集合结尾时，这都是可能的。</span><span class="sxs-lookup"><span data-stu-id="6ad08-123">This is possible for navigation "chains" that are all references, or when they end with a single collection.</span></span>

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#IncludeMultipleNavigationsWithSingleInclude)]

## <a name="filtered-include"></a><span data-ttu-id="6ad08-124">经过筛选的包含</span><span class="sxs-lookup"><span data-stu-id="6ad08-124">Filtered include</span></span>

> [!NOTE]
> <span data-ttu-id="6ad08-125">EF Core 5.0 中已引入此功能。</span><span class="sxs-lookup"><span data-stu-id="6ad08-125">This feature was introduced in EF Core 5.0.</span></span>

<span data-ttu-id="6ad08-126">在应用包含功能来加载相关数据时，可对已包含的集合导航应用某些可枚举的操作，这样就可对结果进行筛选和排序。</span><span class="sxs-lookup"><span data-stu-id="6ad08-126">When applying Include to load related data, you can add certain enumerable operations to the included collection navigation, which allows for filtering and sorting of the results.</span></span>

<span data-ttu-id="6ad08-127">支持的操作包括：`Where`、`OrderBy`、`OrderByDescending`、`ThenBy`、`ThenByDescending`、`Skip` 和 `Take`。</span><span class="sxs-lookup"><span data-stu-id="6ad08-127">Supported operations are: `Where`, `OrderBy`, `OrderByDescending`, `ThenBy`, `ThenByDescending`, `Skip`, and `Take`.</span></span>

<span data-ttu-id="6ad08-128">应对传递到 Include 方法的 Lambda 中的集合导航应用这类操作，如下例所示：</span><span class="sxs-lookup"><span data-stu-id="6ad08-128">Such operations should be applied on the collection navigation in the lambda passed to the Include method, as shown in example below:</span></span>

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#FilteredInclude)]

<span data-ttu-id="6ad08-129">只能对每个包含的导航执行一组唯一的筛选器操作。</span><span class="sxs-lookup"><span data-stu-id="6ad08-129">Each included navigation allows only one unique set of filter operations.</span></span> <span data-ttu-id="6ad08-130">如果为某个给定的集合导航应用了多个包含操作（下例中为 `blog.Posts`），则只能对其中一个导航指定筛选器操作：</span><span class="sxs-lookup"><span data-stu-id="6ad08-130">In cases where multiple Include operations are applied for a given collection navigation (`blog.Posts` in the examples below), filter operations can only be specified on one of them:</span></span>

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#MultipleLeafIncludesFiltered1)]

<span data-ttu-id="6ad08-131">可对多次包含的每个导航应用相同的操作：</span><span class="sxs-lookup"><span data-stu-id="6ad08-131">Instead, identical operations can be applied for each navigation that is included multiple times:</span></span>

[!code-csharp[Main](../../../../samples/core/Querying/RelatedData/Program.cs#MultipleLeafIncludesFiltered2)]

> [!CAUTION]
> <span data-ttu-id="6ad08-132">在跟踪查询时，由于[导航修正](xref:core/querying/tracking)，Filtered Include 的结果可能不符合预期。</span><span class="sxs-lookup"><span data-stu-id="6ad08-132">In case of tracking queries, results of Filtered Include may be unexpected due to [navigation fixup](xref:core/querying/tracking).</span></span> <span data-ttu-id="6ad08-133">之前已查询且已存储在更改跟踪器的所有相关实体都将在 Filtered Include 查询的结果中显示，即使它们不符合筛选器的要求也是如此。</span><span class="sxs-lookup"><span data-stu-id="6ad08-133">All relevant entities that have been queried for previously and have been stored in the Change Tracker will be present in the results of Filtered Include query, even if they don't meet the requirements of the filter.</span></span> <span data-ttu-id="6ad08-134">在这些情况下使用 Filtered Include 时，请考虑使用 `NoTracking` 查询或重新创建 DbContext。</span><span class="sxs-lookup"><span data-stu-id="6ad08-134">Consider using `NoTracking` queries or re-create the DbContext when using Filtered Include in those situations.</span></span>

<span data-ttu-id="6ad08-135">示例：</span><span class="sxs-lookup"><span data-stu-id="6ad08-135">Example:</span></span>

```csharp
var orders = context.Orders.Where(o => o.Id > 1000).ToList();

// customer entities will have references to all orders where Id > 1000, rather than > 5000
var filtered = context.Customers.Include(c => c.Orders.Where(o => o.Id > 5000)).ToList();
```

> [!NOTE]
> <span data-ttu-id="6ad08-136">在跟踪查询的情况下，应用了经过筛选的包含的导航被视为已加载。</span><span class="sxs-lookup"><span data-stu-id="6ad08-136">In case of tracking queries, the navigation on which filtered include was applied is considered to be loaded.</span></span> <span data-ttu-id="6ad08-137">这意味着 EF Core 不会尝试使用[显式加载](xref:core/querying/related-data/explicit)或[延迟加载](xref:core/querying/related-data/lazy)来重新加载其值，即使某些元素仍然可能缺失也不会尝试。</span><span class="sxs-lookup"><span data-stu-id="6ad08-137">This means that EF Core will not attempt to re-load it's values using [explicit loading](xref:core/querying/related-data/explicit) or [lazy loading](xref:core/querying/related-data/lazy), even though some elements could still be missing.</span></span>

## <a name="include-on-derived-types"></a><span data-ttu-id="6ad08-138">派生类型上的包含</span><span class="sxs-lookup"><span data-stu-id="6ad08-138">Include on derived types</span></span>

<span data-ttu-id="6ad08-139">可以使用 `Include` 和 `ThenInclude` 包含仅在派生类型上定义的导航的关联数据。</span><span class="sxs-lookup"><span data-stu-id="6ad08-139">You can include related data from navigation defined only on a derived type using `Include` and `ThenInclude`.</span></span>

<span data-ttu-id="6ad08-140">给定以下模型：</span><span class="sxs-lookup"><span data-stu-id="6ad08-140">Given the following model:</span></span>

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

<span data-ttu-id="6ad08-141">所有人员（可以使用许多模式预先加载的学生）的 `School` 导航的内容：</span><span class="sxs-lookup"><span data-stu-id="6ad08-141">Contents of `School` navigation of all People who are Students can be eagerly loaded using a number of patterns:</span></span>

* <span data-ttu-id="6ad08-142">使用强制转换</span><span class="sxs-lookup"><span data-stu-id="6ad08-142">using cast</span></span>

  ```csharp
  context.People.Include(person => ((Student)person).School).ToList()
  ```

* <span data-ttu-id="6ad08-143">使用 `as` 运算符</span><span class="sxs-lookup"><span data-stu-id="6ad08-143">using `as` operator</span></span>

  ```csharp
  context.People.Include(person => (person as Student).School).ToList()
  ```

* <span data-ttu-id="6ad08-144">使用 `Include` 的重载，该方法采用 `string` 类型的参数</span><span class="sxs-lookup"><span data-stu-id="6ad08-144">using overload of `Include` that takes parameter of type `string`</span></span>

  ```csharp
  context.People.Include("School").ToList()
  ```
