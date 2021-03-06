---
title: 预生成的映射视图-EF6
description: 实体框架6中预生成的映射视图
author: ajcvickers
ms.date: 10/23/2016
uid: ef6/fundamentals/performance/pre-generated-views
ms.openlocfilehash: bea0cdc59161068a8186ad2106516ba4f34910a9
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543141"
---
# <a name="pre-generated-mapping-views"></a>预生成的映射视图
在实体框架可以执行查询或将更改保存到数据源之前，它必须生成一组用于访问数据库的映射视图。 这些映射视图是一组实体 SQL 语句，它们以抽象的方式表示数据库，是每个应用程序域缓存的元数据的一部分。 如果在同一个应用程序域中创建同一个上下文的多个实例，则它们将重用缓存的元数据中的映射视图，而不是重新生成它们。 由于映射视图生成是执行第一个查询的总体成本的重要部分，因此实体框架允许您预先生成映射视图，并将它们包含在已编译的项目中。 有关详细信息，请参阅  [性能注意事项 (实体框架) ](xref:ef6/fundamentals/performance/perf-whitepaper)。

## <a name="generating-mapping-views-with-the-ef-power-tools-community-edition"></a>用 EF Power Tools 社区版生成映射视图

预生成视图的最简单方法是使用 [EF Power Tools 社区版](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EntityFramework6PowerToolsCommunityEdition)。 安装了 Power Tools 后，将有一个用于生成视图的菜单选项，如下所示。

-   对于 **Code First** 模型，右键单击包含 DbContext 类的代码文件。
-   对于 **EF 设计器** 模型，右键单击 EDMX 文件。

![生成视图](~/ef6/media/generateviews.png)

完成此过程后，您将拥有一个类似于下面生成的类

![生成的视图](~/ef6/media/generatedviews.png)

现在，当你运行应用程序 EF 时，将使用此类根据需要加载视图。 如果模型发生更改，并且不重新生成此类，则 EF 会引发异常。

## <a name="generating-mapping-views-from-code---ef6-onwards"></a>从代码生成映射视图-EF6 向前

生成视图的另一种方法是使用 EF 提供的 Api。 使用此方法时，您可以自由地序列化视图，但您也需要自行加载视图。

> [!NOTE]
> **EF6 仅向前** -本部分中所示的 api 在实体框架6中引入。 如果你使用的是早期版本，则不会应用此信息。

### <a name="generating-views"></a>生成视图

用于生成视图的 Api 位于 StorageMappingItemCollection 类中。 ""。 可以使用 ObjectContext 的 MetadataWorkspace 检索上下文的 StorageMappingCollection。 如果你使用的是较新的 DbContext API，则可以使用如下所示的 IObjectContextAdapter 进行访问：在此代码中，我们有一个名为 dbContext 的派生 DbContext 实例：

``` csharp
    var objectContext = ((IObjectContextAdapter) dbContext).ObjectContext;
    var  mappingCollection = (StorageMappingItemCollection)objectContext.MetadataWorkspace
                                                                        .GetItemCollection(DataSpace.CSSpace);
```

获得 StorageMappingItemCollection 后，可以访问 GenerateViews 和 ComputeMappingHashValue 方法。

``` csharp
    public Dictionary<EntitySetBase, DbMappingView> GenerateViews(IList<EdmSchemaError> errors)
    public string ComputeMappingHashValue()
```

第一种方法为容器映射中的每个视图创建一个包含条目的字典。 第二种方法为单个容器映射计算哈希值，并在运行时使用该方法验证模型是否自生成视图以来未发生更改。 对于涉及多个容器映射的复杂方案，会提供两种方法的替代。

生成视图时，将调用 GenerateViews 方法，然后写出生成的 EntitySetBase 和 DbMappingView。 还需要存储 ComputeMappingHashValue 方法生成的哈希。

### <a name="loading-views"></a>正在加载视图

为了加载 GenerateViews 方法生成的视图，你可以为 EF 提供从 DbMappingViewCache 抽象类继承的类。 DbMappingViewCache 指定必须实现的两个方法：

``` csharp
    public abstract string MappingHashValue { get; }
    public abstract DbMappingView GetView(EntitySetBase extent);
```

MappingHashValue 属性必须返回 ComputeMappingHashValue 方法生成的哈希值。 当 EF 打算请求视图时，它将首先生成并将模型的哈希值与此属性返回的哈希值进行比较。 如果二者不匹配，则 EF 将引发 EntityCommandCompilationException 异常。

GetView 方法将接受 EntitySetBase，并且需要返回一个 DbMappingVIew，其中包含为与在 EntitySetBase 方法生成的字典中的给定 GenerateViews 关联的 EntitySql。 如果 EF 要求你提供不具有的视图，则 GetView 应返回 null。

下面是 DbMappingViewCache 中的一种提取功能，如上文所述，使用 Power Tools 生成，其中有一种方法可以存储和检索所需的 EntitySql。

``` csharp
    public override string MappingHashValue
    {
        get { return "a0b843f03dd29abee99789e190a6fb70ce8e93dc97945d437d9a58fb8e2afd2e"; }
    }

    public override DbMappingView GetView(EntitySetBase extent)
    {
        if (extent == null)
        {
            throw new ArgumentNullException("extent");
        }

        var extentName = extent.EntityContainer.Name + "." + extent.Name;

        if (extentName == "BlogContext.Blogs")
        {
            return GetView2();
        }

        if (extentName == "BlogContext.Posts")
        {
            return GetView3();
        }

        return null;
    }

    private static DbMappingView GetView2()
    {
        return new DbMappingView(@"
            SELECT VALUE -- Constructing Blogs
            [BlogApp.Models.Blog](T1.Blog_BlogId, T1.Blog_Test, T1.Blog_title, T1.Blog_Active, T1.Blog_SomeDecimal)
            FROM (
            SELECT
                T.BlogId AS Blog_BlogId,
                T.Test AS Blog_Test,
                T.title AS Blog_title,
                T.Active AS Blog_Active,
                T.SomeDecimal AS Blog_SomeDecimal,
                True AS _from0
            FROM CodeFirstDatabase.Blog AS T
            ) AS T1");
    }
```

若要让 EF 使用 DbMappingViewCache，请使用 DbMappingViewCacheTypeAttribute，同时指定为其创建的上下文。 在下面的代码中，我们将 BlogContext 与 MyMappingViewCache 类相关联。

``` csharp
    [assembly: DbMappingViewCacheType(typeof(BlogContext), typeof(MyMappingViewCache))]
```

对于更复杂的方案，可以通过指定映射视图缓存工厂来提供映射视图缓存实例。 此操作可通过实现抽象类 MappingViews. DbMappingViewCacheFactory 来完成。 可以使用 StorageMappingItemCollection. MappingViewCacheFactoryproperty 检索或设置所使用的映射视图缓存工厂的实例。
