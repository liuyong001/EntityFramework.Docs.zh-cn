---
title: 使用可以为 null 的引用类型-EF Core
description: '使用时使用 c # 可为 null 的引用类型 Entity Framework Core'
author: roji
ms.date: 09/09/2019
uid: core/miscellaneous/nullable-reference-types
ms.openlocfilehash: 749fef8560c6777dcb2314126b11d2dd6a3562f8
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128558"
---
# <a name="working-with-nullable-reference-types"></a><span data-ttu-id="09687-103">使用可以为 Null 的引用类型</span><span class="sxs-lookup"><span data-stu-id="09687-103">Working with Nullable Reference Types</span></span>

<span data-ttu-id="09687-104">C # 8 引入了一项名 [为 null 的引用类型 (NRT) ](/dotnet/csharp/tutorials/nullable-reference-types)的新功能，允许对引用类型进行批注，以指示它是否可用于包含 null。</span><span class="sxs-lookup"><span data-stu-id="09687-104">C# 8 introduced a new feature called [nullable reference types (NRT)](/dotnet/csharp/tutorials/nullable-reference-types), allowing reference types to be annotated, indicating whether it is valid for them to contain null or not.</span></span> <span data-ttu-id="09687-105">如果你不熟悉此功能，则建议你通过阅读 c # 文档来使自己熟悉该功能。</span><span class="sxs-lookup"><span data-stu-id="09687-105">If you are new to this feature, it is recommended that make yourself familiar with it by reading the C# docs.</span></span>

<span data-ttu-id="09687-106">此页介绍 EF Core 对可为 null 的引用类型的支持，并介绍了使用它们的最佳做法。</span><span class="sxs-lookup"><span data-stu-id="09687-106">This page introduces EF Core's support for nullable reference types, and describes best practices for working with them.</span></span>

## <a name="required-and-optional-properties"></a><span data-ttu-id="09687-107">必需属性和可选属性</span><span class="sxs-lookup"><span data-stu-id="09687-107">Required and optional properties</span></span>

<span data-ttu-id="09687-108">对于必需属性和可选属性及其与可为 null 的引用类型的交互，主要文档是 [必需的和可选的属性](xref:core/modeling/entity-properties#required-and-optional-properties) 页。</span><span class="sxs-lookup"><span data-stu-id="09687-108">The main documentation on required and optional properties and their interaction with nullable reference types is the [Required and Optional Properties](xref:core/modeling/entity-properties#required-and-optional-properties) page.</span></span> <span data-ttu-id="09687-109">建议首先阅读该页面。</span><span class="sxs-lookup"><span data-stu-id="09687-109">It is recommended you start out by reading that page first.</span></span>

> [!NOTE]
> <span data-ttu-id="09687-110">在现有项目上启用可以为 null 的引用类型时要格外小心：现在配置为可选的引用类型属性现在将配置为 "必需"，除非它们显式批注为可为 null。</span><span class="sxs-lookup"><span data-stu-id="09687-110">Exercise caution when enabling nullable reference types on an existing project: reference type properties which were previously configured as optional will now be configured as required, unless they are explicitly annotated to be nullable.</span></span> <span data-ttu-id="09687-111">管理关系数据库架构时，这可能会导致生成更改数据库列的为 null 性的迁移。</span><span class="sxs-lookup"><span data-stu-id="09687-111">When managing a relational database schema, this may cause migrations to be generated which alter the database column's nullability.</span></span>

## <a name="non-nullable-properties-and-initialization"></a><span data-ttu-id="09687-112">不可以为 null 的属性和初始化</span><span class="sxs-lookup"><span data-stu-id="09687-112">Non-nullable properties and initialization</span></span>

<span data-ttu-id="09687-113">如果启用了可以为 null 的引用类型，则 c # 编译器将为任何未初始化的不可为 null 的属性发出警告，因为这将包含 null。</span><span class="sxs-lookup"><span data-stu-id="09687-113">When nullable reference types are enabled, the C# compiler emits warnings for any uninitialized non-nullable property, as these would contain null.</span></span> <span data-ttu-id="09687-114">因此，不能使用以下常见方法来编写实体类型：</span><span class="sxs-lookup"><span data-stu-id="09687-114">As a result, the following, common way of writing entity types cannot be used:</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/NullableReferenceTypes/CustomerWithWarning.cs?name=CustomerWithWarning&highlight=4-5)]

<span data-ttu-id="09687-115">[构造函数绑定](xref:core/modeling/constructors) 是一项有用的技术，可确保初始化不可为 null 的属性：</span><span class="sxs-lookup"><span data-stu-id="09687-115">[Constructor binding](xref:core/modeling/constructors) is a useful technique to ensure that your non-nullable properties are initialized:</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/NullableReferenceTypes/CustomerWithConstructorBinding.cs?name=CustomerWithConstructorBinding&highlight=6-9)]

<span data-ttu-id="09687-116">遗憾的是，在某些情况下，构造函数绑定不是一个选项;例如，不能以这种方式初始化导航属性。</span><span class="sxs-lookup"><span data-stu-id="09687-116">Unfortunately, in some scenarios constructor binding isn't an option; navigation properties, for example, cannot be initialized in this way.</span></span>

<span data-ttu-id="09687-117">所需的导航属性会带来额外的难度：尽管某个给定主体的依赖项始终存在，但特定查询可能会或不加载该依赖项，具体取决于程序中该点的需求 (参阅) 的 [加载数据的不同模式](xref:core/querying/related-data) 。</span><span class="sxs-lookup"><span data-stu-id="09687-117">Required navigation properties present an additional difficulty: although a dependent will always exist for a given principal, it may or may not be loaded by a particular query, depending on the needs at that point in the program ([see the different patterns for loading data](xref:core/querying/related-data)).</span></span> <span data-ttu-id="09687-118">同时，不需要将这些属性设置为可以为 null，因为这会强制对它们的所有访问权限，以检查是否有 null，即使它们是必需的。</span><span class="sxs-lookup"><span data-stu-id="09687-118">At the same time, it is undesirable to make these properties nullable, since that would force all access to them to check for null, even if they are required.</span></span>

<span data-ttu-id="09687-119">处理这些方案的一种方法是使用一个可以为 null 的 [支持字段](xref:core/modeling/backing-field)的不可为 null 的属性：</span><span class="sxs-lookup"><span data-stu-id="09687-119">One way to deal with these scenarios, is to have a non-nullable property with a nullable [backing field](xref:core/modeling/backing-field):</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/NullableReferenceTypes/Order.cs?range=10-17)]

<span data-ttu-id="09687-120">由于导航属性不可为 null，因此配置了必需的导航;只要正确加载导航，就可以通过属性访问依赖项。</span><span class="sxs-lookup"><span data-stu-id="09687-120">Since the navigation property is non-nullable, a required navigation is configured; and as long as the navigation is properly loaded, the dependent will be accessible via the property.</span></span> <span data-ttu-id="09687-121">但是，如果在未事先正确加载相关实体的情况下访问属性，则会引发 InvalidOperationException，因为 API 协定的使用不正确。</span><span class="sxs-lookup"><span data-stu-id="09687-121">If, however, the property is accessed without first properly loading the related entity, an InvalidOperationException is thrown, since the API contract has been used incorrectly.</span></span> <span data-ttu-id="09687-122">请注意，必须将 EF 配置为始终访问支持字段而不是属性，因为它依赖于即使在未设置时也能读取值;请参阅有关如何执行此操作的 [支持字段](xref:core/modeling/backing-field) 的文档，并考虑指定 `PropertyAccessMode.Field` 以确保配置正确。</span><span class="sxs-lookup"><span data-stu-id="09687-122">Note that EF must be configured to always access the backing field and not the property, as it relies on being able to read the value even when unset; consult the documentation on [backing fields](xref:core/modeling/backing-field) on how to do this, and consider specifying `PropertyAccessMode.Field` to make sure the configuration is correct.</span></span>

<span data-ttu-id="09687-123">作为 terser 的替代方法，可以使用包容性运算符 (！ ) 的帮助简单地将属性初始化为 null：</span><span class="sxs-lookup"><span data-stu-id="09687-123">As a terser alternative, it is possible to simply initialize the property to null with the help of the null-forgiving operator (!):</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/NullableReferenceTypes/Order.cs?range=19)]

<span data-ttu-id="09687-124">实际的 null 值永远不会被视为除了编程错误之外的情况，例如，访问导航属性时，无需事先正确加载相关实体。</span><span class="sxs-lookup"><span data-stu-id="09687-124">An actual null value will never be observed except as a result of a programming bug, e.g. accessing the navigation property without properly loading the related entity beforehand.</span></span>

> [!NOTE]
> <span data-ttu-id="09687-125">包含对多个相关实体的引用的集合导航应始终不可为 null。</span><span class="sxs-lookup"><span data-stu-id="09687-125">Collection navigations, which contain references to multiple related entities, should always be non-nullable.</span></span> <span data-ttu-id="09687-126">空集合意味着不存在相关实体，但列表本身不应为 null。</span><span class="sxs-lookup"><span data-stu-id="09687-126">An empty collection means that no related entities exist, but the list itself should never be null.</span></span>

## <a name="dbcontext-and-dbset"></a><span data-ttu-id="09687-127">DbContext 和 DbSet</span><span class="sxs-lookup"><span data-stu-id="09687-127">DbContext and DbSet</span></span>

<span data-ttu-id="09687-128">在上下文类型上具有未初始化的 DbSet 属性的常见做法也有问题，因为编译器现在会发出警告。</span><span class="sxs-lookup"><span data-stu-id="09687-128">The common practice of having uninitialized DbSet properties on context types is also problematic, as the compiler will now emit warnings for them.</span></span> <span data-ttu-id="09687-129">可以按如下所示修复此问题：</span><span class="sxs-lookup"><span data-stu-id="09687-129">This can be fixed as follows:</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/NullableReferenceTypes/NullableReferenceTypesContext.cs?name=Context&highlight=3-4)]

<span data-ttu-id="09687-130">另一种策略是使用不可为 null 的自动属性，但要将其初始化为 null，请使用包容性运算符 (！ ) 使编译器警告无提示。</span><span class="sxs-lookup"><span data-stu-id="09687-130">Another strategy is to use non-nullable auto-properties, but to initialize them to null, using the null-forgiving operator (!) to silence the compiler warning.</span></span> <span data-ttu-id="09687-131">DbContext 基本构造函数可确保所有 DbSet 属性都将进行初始化，并且不会在这些属性上看到 null。</span><span class="sxs-lookup"><span data-stu-id="09687-131">The DbContext base constructor ensures that all DbSet properties will get initialized, and null will never be observed on them.</span></span>

## <a name="navigating-and-including-nullable-relationships"></a><span data-ttu-id="09687-132">导航和包括可以为 null 的关系</span><span class="sxs-lookup"><span data-stu-id="09687-132">Navigating and including nullable relationships</span></span>

<span data-ttu-id="09687-133">当处理可选关系时，可能会遇到编译器警告，但不可能出现实际的 null 引用异常。</span><span class="sxs-lookup"><span data-stu-id="09687-133">When dealing with optional relationships, it's possible to encounter compiler warnings where an actual null reference exception would be impossible.</span></span> <span data-ttu-id="09687-134">在转换和执行 LINQ 查询时，EF Core 确保在一个可选的相关实体不存在的情况下，将忽略对该实体的任何导航，而不是引发。</span><span class="sxs-lookup"><span data-stu-id="09687-134">When translating and executing your LINQ queries, EF Core guarantees that if an optional related entity does not exist, any navigation to it will simply be ignored, rather than throwing.</span></span> <span data-ttu-id="09687-135">但是，编译器不知道这 EF Core 确保，并生成警告，就好像 LINQ 查询是在内存中执行的，而 LINQ to Objects。</span><span class="sxs-lookup"><span data-stu-id="09687-135">However, the compiler is unaware of this EF Core guarantee, and produces warnings as if the LINQ query were executed in memory, with LINQ to Objects.</span></span> <span data-ttu-id="09687-136">因此，需要使用包容性运算符 (！ ) 来通知编译器无法实现实际的 null 值：</span><span class="sxs-lookup"><span data-stu-id="09687-136">As a result, it is necessary to use the null-forgiving operator (!) to inform the compiler that an actual null value isn't possible:</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/NullableReferenceTypes/Program.cs?range=46)]

<span data-ttu-id="09687-137">在可选导航中包含多个级别的关系时，会发生类似的问题：</span><span class="sxs-lookup"><span data-stu-id="09687-137">A similar issue occurs when including multiple levels of relationships across optional navigations:</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/NullableReferenceTypes/Program.cs?range=36-39&highlight=2)]

<span data-ttu-id="09687-138">如果你发现自己执行了大量操作，并且所涉及的实体类型主要 (或独占) 用于 EF Core 查询，请考虑使导航属性不可为 null，并将其配置为可通过流畅 API 或数据批注将它们配置为可选。</span><span class="sxs-lookup"><span data-stu-id="09687-138">If you find yourself doing this a lot, and the entity types in question are predominantly (or exclusively) used in EF Core queries, consider making the navigation properties non-nullable, and to configure them as optional via the Fluent API or Data Annotations.</span></span> <span data-ttu-id="09687-139">这将删除所有编译器警告，同时保持关系为可选;但是，如果你的实体是在 EF Core 之外遍历的，则你可能会观察到 null 值，尽管这些属性已批注为不可为 null。</span><span class="sxs-lookup"><span data-stu-id="09687-139">This will remove all compiler warnings while keeping the relationship optional; however, if your entities are traversed outside of EF Core, you may observe null values although the properties are annotated as non-nullable.</span></span>

## <a name="limitations"></a><span data-ttu-id="09687-140">限制</span><span class="sxs-lookup"><span data-stu-id="09687-140">Limitations</span></span>

* <span data-ttu-id="09687-141">反向工程当前不支持 [c # 8 可为 null 的引用类型 (NRTs) ](/dotnet/csharp/tutorials/nullable-reference-types)： EF Core 始终生成假定该功能处于关闭状态的 c # 代码。</span><span class="sxs-lookup"><span data-stu-id="09687-141">Reverse engineering does not currently support [C# 8 nullable reference types (NRTs)](/dotnet/csharp/tutorials/nullable-reference-types): EF Core always generates C# code that assumes the feature is off.</span></span> <span data-ttu-id="09687-142">例如，可以将可为 null 的文本列基架为类型为的属性 `string` ，而不是 `string?` 用于配置是否需要属性的熟知 API 或数据批注。</span><span class="sxs-lookup"><span data-stu-id="09687-142">For example, nullable text columns will be scaffolded as a property with type `string` , not `string?`, with either the Fluent API or Data Annotations used to configure whether a property is required or not.</span></span> <span data-ttu-id="09687-143">您可以编辑基架代码并将其替换为 c # 为空批注。</span><span class="sxs-lookup"><span data-stu-id="09687-143">You can edit the scaffolded code and replace these with C# nullability annotations.</span></span> <span data-ttu-id="09687-144">[#15520](https://github.com/dotnet/efcore/issues/15520)的问题跟踪了可为 null 的引用类型的基架支持。</span><span class="sxs-lookup"><span data-stu-id="09687-144">Scaffolding support for nullable reference types is tracked by issue [#15520](https://github.com/dotnet/efcore/issues/15520).</span></span>
* <span data-ttu-id="09687-145">EF Core 的公共 API 图面尚未批注为为空性 (公共 API 为 "在意" ) ，这使得在打开 NRT 功能时，使用此功能有时会很难使用。</span><span class="sxs-lookup"><span data-stu-id="09687-145">EF Core's public API surface has not yet been annotated for nullability (the public API is "null-oblivious"), making it sometimes awkward to use when the NRT feature is turned on.</span></span> <span data-ttu-id="09687-146">这特别包括 EF Core 公开的异步 LINQ 运算符，如 [FirstOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.firstordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_FirstOrDefaultAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_)。</span><span class="sxs-lookup"><span data-stu-id="09687-146">This notably includes the async LINQ operators exposed by EF Core, such as [FirstOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.firstordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_FirstOrDefaultAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_).</span></span> <span data-ttu-id="09687-147">我们计划为6.0 版本解决这一情况。</span><span class="sxs-lookup"><span data-stu-id="09687-147">We plan to address this for the 6.0 release.</span></span>
