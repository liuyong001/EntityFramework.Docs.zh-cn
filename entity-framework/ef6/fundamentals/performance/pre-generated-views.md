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
# <a name="pre-generated-mapping-views"></a><span data-ttu-id="eee65-103">预生成的映射视图</span><span class="sxs-lookup"><span data-stu-id="eee65-103">Pre-generated mapping views</span></span>
<span data-ttu-id="eee65-104">在实体框架可以执行查询或将更改保存到数据源之前，它必须生成一组用于访问数据库的映射视图。</span><span class="sxs-lookup"><span data-stu-id="eee65-104">Before the Entity Framework can execute a query or save changes to the data source, it must generate a set of mapping views to access the database.</span></span> <span data-ttu-id="eee65-105">这些映射视图是一组实体 SQL 语句，它们以抽象的方式表示数据库，是每个应用程序域缓存的元数据的一部分。</span><span class="sxs-lookup"><span data-stu-id="eee65-105">These mapping views are a set of Entity SQL statement that represent the database in an abstract way, and are part of the metadata which is cached per application domain.</span></span> <span data-ttu-id="eee65-106">如果在同一个应用程序域中创建同一个上下文的多个实例，则它们将重用缓存的元数据中的映射视图，而不是重新生成它们。</span><span class="sxs-lookup"><span data-stu-id="eee65-106">If you create multiple instances of the same context in the same application domain, they will reuse mapping views from the cached metadata rather than regenerating them.</span></span> <span data-ttu-id="eee65-107">由于映射视图生成是执行第一个查询的总体成本的重要部分，因此实体框架允许您预先生成映射视图，并将它们包含在已编译的项目中。</span><span class="sxs-lookup"><span data-stu-id="eee65-107">Because mapping view generation is a significant part of the overall cost of executing the first query, the Entity Framework enables you to pre-generate mapping views and include them in the compiled project.</span></span> <span data-ttu-id="eee65-108">有关详细信息，请参阅  [性能注意事项 (实体框架) ](xref:ef6/fundamentals/performance/perf-whitepaper)。</span><span class="sxs-lookup"><span data-stu-id="eee65-108">For more information, see  [Performance Considerations (Entity Framework)](xref:ef6/fundamentals/performance/perf-whitepaper).</span></span>

## <a name="generating-mapping-views-with-the-ef-power-tools-community-edition"></a><span data-ttu-id="eee65-109">用 EF Power Tools 社区版生成映射视图</span><span class="sxs-lookup"><span data-stu-id="eee65-109">Generating Mapping Views with the EF Power Tools Community Edition</span></span>

<span data-ttu-id="eee65-110">预生成视图的最简单方法是使用 [EF Power Tools 社区版](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EntityFramework6PowerToolsCommunityEdition)。</span><span class="sxs-lookup"><span data-stu-id="eee65-110">The easiest way to pre-generate views is to use the [EF Power Tools Community Edition](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EntityFramework6PowerToolsCommunityEdition).</span></span> <span data-ttu-id="eee65-111">安装了 Power Tools 后，将有一个用于生成视图的菜单选项，如下所示。</span><span class="sxs-lookup"><span data-stu-id="eee65-111">Once you have the Power Tools installed you will have a menu option to Generate Views, as below.</span></span>

-   <span data-ttu-id="eee65-112">对于 **Code First** 模型，右键单击包含 DbContext 类的代码文件。</span><span class="sxs-lookup"><span data-stu-id="eee65-112">For **Code First** models right-click on the code file that contains your DbContext class.</span></span>
-   <span data-ttu-id="eee65-113">对于 **EF 设计器** 模型，右键单击 EDMX 文件。</span><span class="sxs-lookup"><span data-stu-id="eee65-113">For **EF Designer** models right-click on your EDMX file.</span></span>

![生成视图](~/ef6/media/generateviews.png)

<span data-ttu-id="eee65-115">完成此过程后，您将拥有一个类似于下面生成的类</span><span class="sxs-lookup"><span data-stu-id="eee65-115">Once the process is finished you will have a class similar to the following generated</span></span>

![生成的视图](~/ef6/media/generatedviews.png)

<span data-ttu-id="eee65-117">现在，当你运行应用程序 EF 时，将使用此类根据需要加载视图。</span><span class="sxs-lookup"><span data-stu-id="eee65-117">Now when you run your application EF will use this class to load views as required.</span></span> <span data-ttu-id="eee65-118">如果模型发生更改，并且不重新生成此类，则 EF 会引发异常。</span><span class="sxs-lookup"><span data-stu-id="eee65-118">If your model changes and you do not re-generate this class then EF will throw an exception.</span></span>

## <a name="generating-mapping-views-from-code---ef6-onwards"></a><span data-ttu-id="eee65-119">从代码生成映射视图-EF6 向前</span><span class="sxs-lookup"><span data-stu-id="eee65-119">Generating Mapping Views from Code - EF6 Onwards</span></span>

<span data-ttu-id="eee65-120">生成视图的另一种方法是使用 EF 提供的 Api。</span><span class="sxs-lookup"><span data-stu-id="eee65-120">The other way to generate views is to use the APIs that EF provides.</span></span> <span data-ttu-id="eee65-121">使用此方法时，您可以自由地序列化视图，但您也需要自行加载视图。</span><span class="sxs-lookup"><span data-stu-id="eee65-121">When using this method you have the freedom to serialize the views however you like, but you also need to load the views yourself.</span></span>

> [!NOTE]
> <span data-ttu-id="eee65-122">**EF6 仅向前** -本部分中所示的 api 在实体框架6中引入。</span><span class="sxs-lookup"><span data-stu-id="eee65-122">**EF6 Onwards Only** - The APIs shown in this section were introduced in Entity Framework 6.</span></span> <span data-ttu-id="eee65-123">如果你使用的是早期版本，则不会应用此信息。</span><span class="sxs-lookup"><span data-stu-id="eee65-123">If you are using an earlier version this information does not apply.</span></span>

### <a name="generating-views"></a><span data-ttu-id="eee65-124">生成视图</span><span class="sxs-lookup"><span data-stu-id="eee65-124">Generating Views</span></span>

<span data-ttu-id="eee65-125">用于生成视图的 Api 位于 StorageMappingItemCollection 类中。 ""。</span><span class="sxs-lookup"><span data-stu-id="eee65-125">The APIs to generate views are on the System.Data.Entity.Core.Mapping.StorageMappingItemCollection class.</span></span> <span data-ttu-id="eee65-126">可以使用 ObjectContext 的 MetadataWorkspace 检索上下文的 StorageMappingCollection。</span><span class="sxs-lookup"><span data-stu-id="eee65-126">You can retrieve a StorageMappingCollection for a Context by using the MetadataWorkspace of an ObjectContext.</span></span> <span data-ttu-id="eee65-127">如果你使用的是较新的 DbContext API，则可以使用如下所示的 IObjectContextAdapter 进行访问：在此代码中，我们有一个名为 dbContext 的派生 DbContext 实例：</span><span class="sxs-lookup"><span data-stu-id="eee65-127">If you are using the newer DbContext API then you can access this by using the IObjectContextAdapter like below, in this code we have an instance of your derived DbContext called dbContext:</span></span>

``` csharp
    var objectContext = ((IObjectContextAdapter) dbContext).ObjectContext;
    var  mappingCollection = (StorageMappingItemCollection)objectContext.MetadataWorkspace
                                                                        .GetItemCollection(DataSpace.CSSpace);
```

<span data-ttu-id="eee65-128">获得 StorageMappingItemCollection 后，可以访问 GenerateViews 和 ComputeMappingHashValue 方法。</span><span class="sxs-lookup"><span data-stu-id="eee65-128">Once you have the StorageMappingItemCollection then you can get access to the GenerateViews and ComputeMappingHashValue methods.</span></span>

``` csharp
    public Dictionary<EntitySetBase, DbMappingView> GenerateViews(IList<EdmSchemaError> errors)
    public string ComputeMappingHashValue()
```

<span data-ttu-id="eee65-129">第一种方法为容器映射中的每个视图创建一个包含条目的字典。</span><span class="sxs-lookup"><span data-stu-id="eee65-129">The first method creates a dictionary with an entry for each view in the container mapping.</span></span> <span data-ttu-id="eee65-130">第二种方法为单个容器映射计算哈希值，并在运行时使用该方法验证模型是否自生成视图以来未发生更改。</span><span class="sxs-lookup"><span data-stu-id="eee65-130">The second method computes a hash value for the single container mapping and is used at runtime to validate that the model has not changed since the views were pre-generated.</span></span> <span data-ttu-id="eee65-131">对于涉及多个容器映射的复杂方案，会提供两种方法的替代。</span><span class="sxs-lookup"><span data-stu-id="eee65-131">Overrides of the two methods are provided for complex scenarios involving multiple container mappings.</span></span>

<span data-ttu-id="eee65-132">生成视图时，将调用 GenerateViews 方法，然后写出生成的 EntitySetBase 和 DbMappingView。</span><span class="sxs-lookup"><span data-stu-id="eee65-132">When generating views you will call the GenerateViews method and then write out the resulting EntitySetBase and DbMappingView.</span></span> <span data-ttu-id="eee65-133">还需要存储 ComputeMappingHashValue 方法生成的哈希。</span><span class="sxs-lookup"><span data-stu-id="eee65-133">You will also need to store the hash generated by the ComputeMappingHashValue method.</span></span>

### <a name="loading-views"></a><span data-ttu-id="eee65-134">正在加载视图</span><span class="sxs-lookup"><span data-stu-id="eee65-134">Loading Views</span></span>

<span data-ttu-id="eee65-135">为了加载 GenerateViews 方法生成的视图，你可以为 EF 提供从 DbMappingViewCache 抽象类继承的类。</span><span class="sxs-lookup"><span data-stu-id="eee65-135">In order to load the views generated by the GenerateViews method, you can provide EF with a class that inherits from the DbMappingViewCache abstract class.</span></span> <span data-ttu-id="eee65-136">DbMappingViewCache 指定必须实现的两个方法：</span><span class="sxs-lookup"><span data-stu-id="eee65-136">DbMappingViewCache specifies two methods that you must implement:</span></span>

``` csharp
    public abstract string MappingHashValue { get; }
    public abstract DbMappingView GetView(EntitySetBase extent);
```

<span data-ttu-id="eee65-137">MappingHashValue 属性必须返回 ComputeMappingHashValue 方法生成的哈希值。</span><span class="sxs-lookup"><span data-stu-id="eee65-137">The MappingHashValue property must return the hash generated by the ComputeMappingHashValue method.</span></span> <span data-ttu-id="eee65-138">当 EF 打算请求视图时，它将首先生成并将模型的哈希值与此属性返回的哈希值进行比较。</span><span class="sxs-lookup"><span data-stu-id="eee65-138">When EF is going to ask for views it will first generate and compare the hash value of the model with the hash returned by this property.</span></span> <span data-ttu-id="eee65-139">如果二者不匹配，则 EF 将引发 EntityCommandCompilationException 异常。</span><span class="sxs-lookup"><span data-stu-id="eee65-139">If they do not match then EF will throw an EntityCommandCompilationException exception.</span></span>

<span data-ttu-id="eee65-140">GetView 方法将接受 EntitySetBase，并且需要返回一个 DbMappingVIew，其中包含为与在 EntitySetBase 方法生成的字典中的给定 GenerateViews 关联的 EntitySql。</span><span class="sxs-lookup"><span data-stu-id="eee65-140">The GetView method will accept an EntitySetBase and you need to return a DbMappingVIew containing the EntitySql that was generated for that was associated with the given EntitySetBase in the dictionary generated by the GenerateViews method.</span></span> <span data-ttu-id="eee65-141">如果 EF 要求你提供不具有的视图，则 GetView 应返回 null。</span><span class="sxs-lookup"><span data-stu-id="eee65-141">If EF asks for a view that you do not have then GetView should return null.</span></span>

<span data-ttu-id="eee65-142">下面是 DbMappingViewCache 中的一种提取功能，如上文所述，使用 Power Tools 生成，其中有一种方法可以存储和检索所需的 EntitySql。</span><span class="sxs-lookup"><span data-stu-id="eee65-142">The following is an extract from the DbMappingViewCache that is generated with the Power Tools as described above, in it we see one way to store and retrieve the EntitySql required.</span></span>

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

<span data-ttu-id="eee65-143">若要让 EF 使用 DbMappingViewCache，请使用 DbMappingViewCacheTypeAttribute，同时指定为其创建的上下文。</span><span class="sxs-lookup"><span data-stu-id="eee65-143">To have EF use your DbMappingViewCache you add use the DbMappingViewCacheTypeAttribute, specifying the context that it was created for.</span></span> <span data-ttu-id="eee65-144">在下面的代码中，我们将 BlogContext 与 MyMappingViewCache 类相关联。</span><span class="sxs-lookup"><span data-stu-id="eee65-144">In the code below we associate the BlogContext with the MyMappingViewCache class.</span></span>

``` csharp
    [assembly: DbMappingViewCacheType(typeof(BlogContext), typeof(MyMappingViewCache))]
```

<span data-ttu-id="eee65-145">对于更复杂的方案，可以通过指定映射视图缓存工厂来提供映射视图缓存实例。</span><span class="sxs-lookup"><span data-stu-id="eee65-145">For more complex scenarios, mapping view cache instances can be provided by specifying a mapping view cache factory.</span></span> <span data-ttu-id="eee65-146">此操作可通过实现抽象类 MappingViews. DbMappingViewCacheFactory 来完成。</span><span class="sxs-lookup"><span data-stu-id="eee65-146">This can be done by implementing the abstract class System.Data.Entity.Infrastructure.MappingViews.DbMappingViewCacheFactory.</span></span> <span data-ttu-id="eee65-147">可以使用 StorageMappingItemCollection. MappingViewCacheFactoryproperty 检索或设置所使用的映射视图缓存工厂的实例。</span><span class="sxs-lookup"><span data-stu-id="eee65-147">The instance of the mapping view cache factory that is used can be retrieved or set using the StorageMappingItemCollection.MappingViewCacheFactoryproperty.</span></span>
