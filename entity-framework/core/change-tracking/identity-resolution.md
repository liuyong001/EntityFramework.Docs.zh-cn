---
title: 标识解析-EF Core
description: 使用主键值将多个实体实例解析为单个实例
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/identity-resolution
ms.openlocfilehash: d4c8f935c8d0ab92eaecd8fc7a4156bd824713d4
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543609"
---
# <a name="identity-resolution-in-ef-core"></a>EF Core 中的标识解析

<xref:Microsoft.EntityFrameworkCore.DbContext>只能使用任意给定的主键值跟踪一个实体实例。 这意味着，必须将具有相同键值的实体的多个实例解析为单个实例。 这称为 "标识解析"。 标识解析可确保 Entity Framework Core (EF Core) 跟踪一致的关系图，而不考虑实体的关系或属性值。

> [!TIP]
> 本文档假设了解 EF Core 更改跟踪的实体状态和基础知识。 有关这些主题的详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。

> [!TIP]
> 通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/IdentityResolutionInEFCore)，你可运行并调试到本文档中的所有代码。

## <a name="introduction"></a>简介

下面的代码查询实体，然后尝试附加具有相同的主键值的不同实例：

<!--
            using var context = new BlogsContext();

            var blogA = context.Blogs.Single(e => e.Id == 1);
            var blogB = new Blog { Id = 1, Name = ".NET Blog (All new!)" };

            try
            {
                context.Update(blogB); // This will throw
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
            }
-->
[!code-csharp[Identity_Resolution_in_EF_Core_1](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Identity_Resolution_in_EF_Core_1)]

运行此代码将导致以下异常：

> InvalidOperationException：无法跟踪实体类型 "博客" 的实例，因为已在跟踪具有密钥值 "{Id： 1}" 的另一个实例。 附加现有实体时，请确保只附加一个具有给定键值的实体实例。

EF Core 需要单个实例，原因如下：

- 多个实例的属性值可能不同。 更新数据库时，EF Core 需要知道要使用的属性值。
- 与其他实体的关系在多个实例之间可能不同。 例如，"blogA" 可能与 "blogB" 不同的发布集合相关。

在以下情况下通常会遇到上述异常：

- 尝试更新实体时
- 尝试跟踪实体的序列化图形时
- 未能设置不自动生成的键值时
- 为多个工作单元重复使用 DbContext 实例时

以下各节将讨论其中的每种情况。

## <a name="updating-an-entity"></a>更新实体

有多种不同的方法可使用新值更新实体，如 [EF Core 中更改跟踪中](xref:core/change-tracking/index) 所述，并 [显式跟踪实体](xref:core/change-tracking/explicit-tracking)。 下面的标识解析上下文中概述了这些方法。 需要注意的一点是，每个方法都使用查询或对或的调用 `Update` `Attach` ，但 **_永远不会同时使用这两_** 种方法。

### <a name="call-update"></a>调用更新

通常，要更新的实体不会来自要用于 SaveChanges 的 DbContext 上的查询。 例如，在 web 应用程序中，可以根据 POST 请求中的信息创建实体实例。 处理此情况的最简单方法是使用 <xref:Microsoft.EntityFrameworkCore.DbContext.Update%2A?displayProperty=nameWithType> 或 <xref:Microsoft.EntityFrameworkCore.DbSet%601.Update%2A?displayProperty=nameWithType> 。 例如：

<!--
    public static void UpdateFromHttpPost1(Blog blog)
    {
        using var context = new BlogsContext();

        context.Update(blog);

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_1](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_1)]

在这种情况下：

- 仅创建实体的单个实例。
- 在进行更新的过程中， **不** 会从数据库中查询实体实例。
- 所有属性值都将在数据库中更新，不管它们是否确实已更改。
- 进行一个数据库往返。

### <a name="query-then-apply-changes"></a>查询，然后应用更改

通常不知道在通过 POST 请求中的信息创建实体时，哪些属性值实际上已更改，或类似。 通常，只需更新数据库中的所有值，就像在前面的示例中一样。 但是，如果应用程序正在处理多个实体，并且只有少量这些实体具有实际更改，则可以限制发送的更新。 这可以通过执行查询来跟踪当前存在于数据库中的实体，然后将更改应用于这些被跟踪的实体，来实现此目的。 例如：

<!--
    public static void UpdateFromHttpPost2(Blog blog)
    {
        using var context = new BlogsContext();

        var trackedBlog = context.Blogs.Find(blog.Id);

        trackedBlog.Name = blog.Name;
        trackedBlog.Summary = blog.Summary;

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_2](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_2)]

在这种情况下：

- 只跟踪实体的单个实例;查询从数据库返回的 `Find` 。
- `Update``Attach`**不** 使用、等。
- 仅在数据库中更新实际更改的属性值。
- 进行两次数据库往返。

EF Core 具有一些用于传输属性值的帮助器。 例如， <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyValues.SetValues%2A?displayProperty=nameWithType> 将复制给定对象中的所有值，并在跟踪的对象上设置这些值：

<!--
    public static void UpdateFromHttpPost3(Blog blog)
    {
        using var context = new BlogsContext();

        var trackedBlog = context.Blogs.Find(blog.Id);

        context.Entry(trackedBlog).CurrentValues.SetValues(blog);

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_3](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_3)]

`SetValues` 接受各种对象类型，包括数据传输对象 (Dto) ，其属性名称与实体类型的属性相匹配。 例如：

<!--
    public static void UpdateFromHttpPost4(BlogDto dto)
    {
        using var context = new BlogsContext();

        var trackedBlog = context.Blogs.Find(dto.Id);

        context.Entry(trackedBlog).CurrentValues.SetValues(dto);

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_4](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_4)]

或包含属性值的名称/值项的字典：

<!--
    public static void UpdateFromHttpPost5(Dictionary<string, object> propertyValues)
    {
        using var context = new BlogsContext();

        var trackedBlog = context.Blogs.Find(propertyValues["Id"]);

        context.Entry(trackedBlog).CurrentValues.SetValues(propertyValues);

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_5](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_5)]

有关使用此类属性值的详细信息，请参阅 [访问跟踪的实体](xref:core/change-tracking/entity-entries) 。

### <a name="use-original-values"></a>使用原始值

到目前为止，每种方法都已在进行更新之前执行了查询，或者更新了所有属性值，无论它们是否已更改。 若要更新不作为更新的一部分进行查询更改的值，需要有关哪些属性值已更改的特定信息。 获取此信息的常见方法是在 HTTP Post 或类似中发送回当前值和原始值。 例如：

<!--
    public static void UpdateFromHttpPost6(Blog blog, Dictionary<string, object> originalValues)
    {
        using var context = new BlogsContext();

        context.Attach(blog);
        context.Entry(blog).OriginalValues.SetValues(originalValues);

        context.SaveChanges();
    }
-->
[!code-csharp[Updating_an_entity_6](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Updating_an_entity_6)]

在此代码中，首先附加带有修改值的实体。 这会导致 EF Core 跟踪状态中的实体 `Unchanged` ; 即，不将属性值标记为已修改。 然后，将原始值的字典应用到此跟踪实体。 这会将不同的当前值和原始值标记为已修改的属性。 具有相同的当前值和原始值的属性将不会标记为已修改。

在这种情况下：

- 只跟踪实体的单个实例，并使用 Attach。
- 在进行更新的过程中， **不** 会从数据库中查询实体实例。
- 应用原始值可确保在数据库中仅更新实际更改的属性值。
- 进行一个数据库往返。

与上一节中的示例一样，原始值不必作为字典传递;实体实例或 DTO 也将起作用。

> [!TIP]
> 虽然这种方法有吸引力，但它确实要求向 web 客户端发送和从 web 客户端发送实体的原始值。 仔细考虑这种额外的复杂性是否值得。对于许多应用程序而言，一种更简单的方法更实用。

## <a name="attaching-a-serialized-graph"></a>附加序列化关系图

EF Core 使用通过外键和导航属性连接的实体图形，如 [更改外键和](xref:core/change-tracking/relationship-changes)导航中所述。 如果这些图形是使用在 EF Core 之外创建的，例如，在 JSON 文件中，则它们可以具有同一实体的多个实例。 在跟踪图形之前，需要将这些重复项解析为单个实例。

### <a name="graphs-with-no-duplicates"></a>不包含重复项的关系图

在进一步操作之前，请务必确认：

- 序列化程序通常具有在关系图中处理循环和重复实例的选项。
- 选择用作图形根的对象通常有助于减少或删除重复项。

如果可能，请使用序列化选项，并选择不会导致重复的根。 例如，下面的代码使用 [Json.NET](https://www.nuget.org/packages/Newtonsoft.Json/) 来序列化一个博客列表，其中每个博客都包含其关联的文章：

<!--
            using var context = new BlogsContext();

            var blogs = context.Blogs.Include(e => e.Posts).ToList();

            var serialized = JsonConvert.SerializeObject(
                blogs,
                new JsonSerializerSettings
                {
                    ReferenceLoopHandling = ReferenceLoopHandling.Ignore,
                    Formatting = Formatting.Indented
                });

            Console.WriteLine(serialized);
-->
[!code-csharp[Attaching_a_serialized_graph_1a](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_1a)]

此代码生成的 JSON 是：

```json
[
  {
    "Id": 1,
    "Name": ".NET Blog",
    "Summary": "Posts about .NET",
    "Posts": [
      {
        "Id": 1,
        "Title": "Announcing the Release of EF Core 5.0",
        "Content": "Announcing the release of EF Core 5.0, a full featured cross-platform...",
        "BlogId": 1
      },
      {
        "Id": 2,
        "Title": "Announcing F# 5",
        "Content": "F# 5 is the latest version of F#, the functional programming language...",
        "BlogId": 1
      }
    ]
  },
  {
    "Id": 2,
    "Name": "Visual Studio Blog",
    "Summary": "Posts about Visual Studio",
    "Posts": [
      {
        "Id": 3,
        "Title": "Disassembly improvements for optimized managed debugging",
        "Content": "If you are focused on squeezing out the last bits of performance for your .NET service or...",
        "BlogId": 2
      },
      {
        "Id": 4,
        "Title": "Database Profiling with Visual Studio",
        "Content": "Examine when database queries were executed and measure how long the take using...",
        "BlogId": 2
      }
    ]
  }
]
```

请注意，JSON 中没有重复的博客或文章。 这意味着，对的简单调用将可用于 `Update` 更新数据库中的这些实体：

<!--
        public static void UpdateBlogsFromJson(string json)
        {
            using var context = new BlogsContext();

            var blogs = JsonConvert.DeserializeObject<List<Blog>>(json);

            foreach (var blog in blogs)
            {
                context.Update(blog);
            }

            context.SaveChanges();
        }
-->
[!code-csharp[Attaching_a_serialized_graph_1b](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_1b)]

### <a name="handling-duplicates"></a>处理重复项

上一示例中的代码将每个博客及其关联的文章序列化。 如果更改此项以序列化每个 post 及其关联的博客，则会将重复项引入序列化的 JSON。 例如：

<!--
            using var context = new BlogsContext();

            var posts = context.Posts.Include(e => e.Blog).ToList();

            var serialized = JsonConvert.SerializeObject(
                posts,
                new JsonSerializerSettings
                {
                    ReferenceLoopHandling = ReferenceLoopHandling.Ignore,
                    Formatting = Formatting.Indented
                });

            Console.WriteLine(serialized);
-->
[!code-csharp[Attaching_a_serialized_graph_2](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_2)]

序列化的 JSON 现在如下所示：

```json
[
  {
    "Id": 1,
    "Title": "Announcing the Release of EF Core 5.0",
    "Content": "Announcing the release of EF Core 5.0, a full featured cross-platform...",
    "BlogId": 1,
    "Blog": {
      "Id": 1,
      "Name": ".NET Blog",
      "Summary": "Posts about .NET",
      "Posts": [
        {
          "Id": 2,
          "Title": "Announcing F# 5",
          "Content": "F# 5 is the latest version of F#, the functional programming language...",
          "BlogId": 1
        }
      ]
    }
  },
  {
    "Id": 2,
    "Title": "Announcing F# 5",
    "Content": "F# 5 is the latest version of F#, the functional programming language...",
    "BlogId": 1,
    "Blog": {
      "Id": 1,
      "Name": ".NET Blog",
      "Summary": "Posts about .NET",
      "Posts": [
        {
          "Id": 1,
          "Title": "Announcing the Release of EF Core 5.0",
          "Content": "Announcing the release of EF Core 5.0, a full featured cross-platform...",
          "BlogId": 1
        }
      ]
    }
  },
  {
    "Id": 3,
    "Title": "Disassembly improvements for optimized managed debugging",
    "Content": "If you are focused on squeezing out the last bits of performance for your .NET service or...",
    "BlogId": 2,
    "Blog": {
      "Id": 2,
      "Name": "Visual Studio Blog",
      "Summary": "Posts about Visual Studio",
      "Posts": [
        {
          "Id": 4,
          "Title": "Database Profiling with Visual Studio",
          "Content": "Examine when database queries were executed and measure how long the take using...",
          "BlogId": 2
        }
      ]
    }
  },
  {
    "Id": 4,
    "Title": "Database Profiling with Visual Studio",
    "Content": "Examine when database queries were executed and measure how long the take using...",
    "BlogId": 2,
    "Blog": {
      "Id": 2,
      "Name": "Visual Studio Blog",
      "Summary": "Posts about Visual Studio",
      "Posts": [
        {
          "Id": 3,
          "Title": "Disassembly improvements for optimized managed debugging",
          "Content": "If you are focused on squeezing out the last bits of performance for your .NET service or...",
          "BlogId": 2
        }
      ]
    }
  }
]
```

请注意，该图形现在包含多个具有相同键值的博客实例，还包括多个具有相同密钥值的 Post 实例。 尝试跟踪此关系图（如前面的示例所做的操作）将引发：

> InvalidOperationException：无法跟踪实体类型 "Post" 的实例，因为已在跟踪具有键值 "{Id： 2}" 的另一个实例。 附加现有实体时，请确保只附加一个具有给定键值的实体实例。

可以通过两种方式解决此问题：

- 使用保留引用的 JSON 序列化选项
- 跟踪图形时执行标识解析

#### <a name="preserve-references"></a>保留引用

Json.NET 提供了 `PreserveReferencesHandling` 用于处理此的选项。 例如：

<!--
            var serialized = JsonConvert.SerializeObject(
                posts,
                new JsonSerializerSettings
                {
                    PreserveReferencesHandling = PreserveReferencesHandling.All,
                    Formatting = Formatting.Indented
                });
-->
[!code-csharp[Attaching_a_serialized_graph_3](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_3)]

生成的 JSON 现在如下所示：

```json
{
  "$id": "1",
  "$values": [
    {
      "$id": "2",
      "Id": 1,
      "Title": "Announcing the Release of EF Core 5.0",
      "Content": "Announcing the release of EF Core 5.0, a full featured cross-platform...",
      "BlogId": 1,
      "Blog": {
        "$id": "3",
        "Id": 1,
        "Name": ".NET Blog",
        "Summary": "Posts about .NET",
        "Posts": [
          {
            "$ref": "2"
          },
          {
            "$id": "4",
            "Id": 2,
            "Title": "Announcing F# 5",
            "Content": "F# 5 is the latest version of F#, the functional programming language...",
            "BlogId": 1,
            "Blog": {
              "$ref": "3"
            }
          }
        ]
      }
    },
    {
      "$ref": "4"
    },
    {
      "$id": "5",
      "Id": 3,
      "Title": "Disassembly improvements for optimized managed debugging",
      "Content": "If you are focused on squeezing out the last bits of performance for your .NET service or...",
      "BlogId": 2,
      "Blog": {
        "$id": "6",
        "Id": 2,
        "Name": "Visual Studio Blog",
        "Summary": "Posts about Visual Studio",
        "Posts": [
          {
            "$ref": "5"
          },
          {
            "$id": "7",
            "Id": 4,
            "Title": "Database Profiling with Visual Studio",
            "Content": "Examine when database queries were executed and measure how long the take using...",
            "BlogId": 2,
            "Blog": {
              "$ref": "6"
            }
          }
        ]
      }
    },
    {
      "$ref": "7"
    }
  ]
}
```

请注意，此 JSON 已将重复项替换为类似于 `"$ref": "5"` 图中现有实例的引用。 此图可再次使用对的简单调用进行跟踪 `Update` ，如上所示。

<xref:System.Text.Json> (BCL) 的 .net 基类库中的支持具有类似的选项，该选项将产生相同的结果。 例如：

<!--
            var serialized = System.Text.Json.JsonSerializer.Serialize(posts, new System.Text.Json.JsonSerializerOptions
            {
                ReferenceHandler = ReferenceHandler.Preserve,
                WriteIndented = true
            });
-->
[!code-csharp[Attaching_a_serialized_graph_4](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_4)]

#### <a name="resolve-duplicates"></a>解决重复项

如果无法在序列化过程中消除重复项，则 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ChangeTracker.TrackGraph%2A?displayProperty=nameWithType> 提供一种方法来处理这种情况。 TrackGraph 的工作方式类似于 `Add` `Attach` 和， `Update` 只不过它在跟踪之前为每个实体实例生成一个回调。 此回调可用于跟踪实体或忽略它。 例如：

<!--
        public static void UpdatePostsFromJsonWithIdentityResolution(string json)
        {
            using var context = new BlogsContext();

            var posts = JsonConvert.DeserializeObject<List<Post>>(json);

            foreach (var post in posts)
            {
                context.ChangeTracker.TrackGraph(
                    post, node =>
                        {
                            var keyValue = node.Entry.Property("Id").CurrentValue;
                            var entityType = node.Entry.Metadata;

                            var existingEntity = node.Entry.Context.ChangeTracker.Entries()
                                .FirstOrDefault(
                                    e => Equals(e.Metadata, entityType)
                                         && Equals(e.Property("Id").CurrentValue, keyValue));

                            if (existingEntity == null)
                            {
                                Console.WriteLine($"Tracking {entityType} entity with key value {keyValue}");

                                node.Entry.State = EntityState.Modified;
                            }
                            else
                            {
                                Console.WriteLine($"Discarding duplicate {entityType} entity with key value {keyValue}");
                            }
                        });
            }

            context.SaveChanges();
        }
-->
[!code-csharp[Attaching_a_serialized_graph_5](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/SerializedGraphExamples.cs?name=Attaching_a_serialized_graph_5)]

对于图形中的每个实体，此代码将：

- 查找实体的实体类型和键值
- 在更改跟踪器中查找具有此密钥的实体
  - 如果找到该实体，则不采取进一步的操作，因为该实体是重复的
  - 如果找不到该实体，则通过将状态设置为来进行跟踪。 `Modified`

运行此代码的输出为：

```output
Tracking EntityType: Post entity with key value 1
Tracking EntityType: Blog entity with key value 1
Tracking EntityType: Post entity with key value 2
Discarding duplicate EntityType: Post entity with key value 2
Tracking EntityType: Post entity with key value 3
Tracking EntityType: Blog entity with key value 2
Tracking EntityType: Post entity with key value 4
Discarding duplicate EntityType: Post entity with key value 4
```

> [!IMPORTANT]
> 此代码 **假定所有重复项都相同**。 这使得随意选择其中一个重复项来跟踪，同时丢弃其他副本。 如果重复项可能不同，则代码需要决定如何确定使用哪一个，以及如何将属性和导航值组合在一起。

> [!NOTE]
> 为简单起见，此代码假定每个实体都有一个名为的主键属性 `Id` 。 这可能会整理到抽象基类或接口中。 或者，可以从元数据中获取主键属性， <xref:Microsoft.EntityFrameworkCore.Metadata.IEntityType> 以便此代码适用于任何类型的实体。

## <a name="failing-to-set-key-values"></a>未能设置键值

实体类型通常配置为使用 [自动生成的键值](xref:core/modeling/generated-properties)。 这是非复合键的整数和 GUID 属性的默认值。 但是，如果未将该实体类型配置为使用自动生成的键值，则在跟踪该实体之前必须设置显式键值。 例如，使用以下实体类型：

<!--
    public class Pet
    {
        [DatabaseGenerated(DatabaseGeneratedOption.None)]
        public int Id { get; set; }

        public string Name { get; set; }
    }
-->
[!code-csharp[Pet](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Pet)]

考虑在不设置键值的情况下尝试跟踪两个新实体实例的代码：

<!--
            using var context = new BlogsContext();

            context.Add(new Pet { Name = "Smokey" });

            try
            {
                context.Add(new Pet { Name = "Clippy" }); // This will throw
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
            }
-->
[!code-csharp[Failing_to_set_key_values_1](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=Failing_to_set_key_values_1)]

此代码将引发：

> InvalidOperationException：无法跟踪实体类型 "Pet" 的实例，因为已在跟踪具有密钥值 "{Id： 0}" 的另一个实例。 附加现有实体时，请确保只附加一个具有给定键值的实体实例。

解决此问题的方法是显式设置键值，或将 key 属性配置为使用生成的键值。 有关详细信息，请参阅 [生成的值](xref:core/modeling/generated-properties) 。

## <a name="overusing-a-single-dbcontext-instance"></a>过度使用单个 DbContext 实例

<xref:Microsoft.EntityFrameworkCore.DbContext> 旨在表示短期的工作单元，如 [DbContext 初始化和配置](xref:core/dbcontext-configuration/index)中所述，并详细介绍 EF Core 中的 [更改跟踪](xref:core/change-tracking/index)。 如果未遵循本指南，则很容易遇到尝试跟踪同一实体的多个实例的情况。 常见示例包括：

- 使用相同的 DbContext 实例设置测试状态，然后执行测试。 这通常会导致 DbContext 仍跟踪测试设置中的一个实体实例，同时尝试在测试中附加一个新实例。 请改用不同的 DbContext 实例来设置测试状态和测试代码。
- 在存储库或类似的代码中使用共享的 DbContext 实例。 相反，请确保存储库对每个工作单元使用单个 DbContext 实例。

## <a name="identity-resolution-and-queries"></a>标识解析和查询

当从查询跟踪实体时，将自动进行标识解析。 这意味着，如果已跟踪具有给定键值的实体实例，则使用此现有跟踪实例而不是创建新的实例。 这有一个重要的结果：如果数据在数据库中发生了更改，则不会在查询结果中反映出来。 这是将新的 DbContext 实例用于每个工作单元的好理由，如 [DbContext 初始化和配置](xref:core/dbcontext-configuration/index)中所述，以及 [EF Core 更改跟踪](xref:core/change-tracking/index)的详细说明。

> [!IMPORTANT]
> 务必了解 EF Core 始终针对数据库执行 LINQ 查询，并根据数据库中的内容仅返回结果。 但是，对于跟踪查询，如果已跟踪返回的实体，则使用跟踪的实例，而不是从数据库中的数据创建实例。

<xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.Reload><xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.GetDatabaseValues>当需要使用数据库中的最新数据刷新跟踪的实体时，可以使用或。 有关详细信息，请参阅 [访问跟踪的实体](xref:core/change-tracking/entity-entries) 。

与跟踪查询不同，无跟踪查询不会执行标识解析。 这意味着，无跟踪查询可以像前面所述的 JSON 序列化案例一样返回重复项。 如果查询结果将被序列化并发送到客户端，这通常不是问题。

> [!TIP]
> 不要定期执行无跟踪查询，然后将返回的实体附加到同一个上下文。 这会比使用跟踪查询更慢且更难。

无跟踪查询不会执行标识解析，因为这样做会影响从查询对大量实体进行流式处理的性能。 这是因为，标识解析需要跟踪每个返回的实例，以便可以使用它，而不是以后创建重复项。

从 EF Core 5.0 开始，无跟踪查询可以通过使用强制执行标识解析 <xref:Microsoft.EntityFrameworkCore.EntityFrameworkQueryableExtensions.AsNoTrackingWithIdentityResolution%60%601(System.Linq.IQueryable{%60%600})> 。 然后，该查询将跟踪返回 (的实例，而无需按正常方式跟踪它们) 并确保在查询结果中不会创建重复项。

## <a name="overriding-object-equality"></a>重写对象相等性

EF Core 在比较实体实例时使用 [引用相等性](/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons) 。 即使实体类型重写 <xref:System.Object.Equals(System.Object)?displayProperty=nameWithType> 或更改对象相等性，也是如此。 但是，重写相等性可能会影响 EF Core 行为：当集合导航使用重写的相等性而不是引用相等性时，因此将多个实例报告为同一。

因此，建议应避免重写实体相等性。 如果使用此方法，请确保创建强制引用相等的集合导航。 例如，创建使用引用相等性的相等比较器：

<!--
    public sealed class ReferenceEqualityComparer : IEqualityComparer<object>
    {
        private ReferenceEqualityComparer()
        {
        }

        public static ReferenceEqualityComparer Instance { get; } = new ReferenceEqualityComparer();

        bool IEqualityComparer<object>.Equals(object x, object y) => x == y;

        int IEqualityComparer<object>.GetHashCode(object obj) => RuntimeHelpers.GetHashCode(obj);
    }
-->
[!code-csharp[ReferenceEqualityComparer](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/ReferenceEqualityComparer.cs?name=ReferenceEqualityComparer)]

从 .NET 5 开始 (，它将作为中包含在 BCL 中 <xref:System.Collections.Generic.ReferenceEqualityComparer> 。 ) 

然后，可以在创建集合导航时使用此比较器。 例如：

<!--
        public ICollection<Order> Orders { get; set; }
            = new HashSet<Order>(ReferenceEqualityComparer.Instance);
-->
[!code-csharp[OrdersCollection](../../../samples/core/ChangeTracking/IdentityResolutionInEFCore/IdentityResolutionSamples.cs?name=OrdersCollection)]

### <a name="comparing-key-properties"></a>比较键属性

除了相等比较以外，还需要对键值进行排序。 这对于在对 SaveChanges 的单个调用中更新多个实体时避免死锁非常重要。 用于 primary、代用或外键属性以及用于唯一索引的所有类型都必须实现 <xref:System.IComparable%601> 和 <xref:System.IEquatable%601> 。 通常用作键 (int、Guid、string 等的类型 ) 已支持这些接口。 自定义密钥类型可以添加这些接口。
