---
title: 拥有的实体类型-EF Core
description: 如何在使用时配置拥有的实体类型或聚合 Entity Framework Core
author: AndriySvyryd
ms.date: 11/06/2019
uid: core/modeling/owned-entities
ms.openlocfilehash: 0cd6bfd25d4462509a3e6c112b892d652d29e45e
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128623"
---
# <a name="owned-entity-types"></a><span data-ttu-id="ce095-103">从属实体类型</span><span class="sxs-lookup"><span data-stu-id="ce095-103">Owned Entity Types</span></span>

<span data-ttu-id="ce095-104">EF Core 允许您为只能出现在其他实体类型的导航属性中的实体类型建模。</span><span class="sxs-lookup"><span data-stu-id="ce095-104">EF Core allows you to model entity types that can only ever appear on navigation properties of other entity types.</span></span> <span data-ttu-id="ce095-105">它们称为 _拥有的实体类型_。</span><span class="sxs-lookup"><span data-stu-id="ce095-105">These are called _owned entity types_.</span></span> <span data-ttu-id="ce095-106">包含拥有的实体类型的实体是其 _所有者_。</span><span class="sxs-lookup"><span data-stu-id="ce095-106">The entity containing an owned entity type is its _owner_.</span></span>

<span data-ttu-id="ce095-107">拥有的实体实质上是所有者的一部分，并且在没有它的情况下不存在，它们在概念上类似于 [聚合](https://martinfowler.com/bliki/DDD_Aggregate.html)。</span><span class="sxs-lookup"><span data-stu-id="ce095-107">Owned entities are essentially a part of the owner and cannot exist without it, they are conceptually similar to [aggregates](https://martinfowler.com/bliki/DDD_Aggregate.html).</span></span> <span data-ttu-id="ce095-108">这意味着，拥有的实体由与所有者的关系的从属方定义。</span><span class="sxs-lookup"><span data-stu-id="ce095-108">This means that the owned entity is by definition on the dependent side of the relationship with the owner.</span></span>

## <a name="explicit-configuration"></a><span data-ttu-id="ce095-109">显式配置</span><span class="sxs-lookup"><span data-stu-id="ce095-109">Explicit configuration</span></span>

<span data-ttu-id="ce095-110">所有实体类型永远不会通过约定 EF Core 在模型中。</span><span class="sxs-lookup"><span data-stu-id="ce095-110">Owned entity types are never included by EF Core in the model by convention.</span></span> <span data-ttu-id="ce095-111">您可以使用 `OwnsOne` 中的方法 `OnModelCreating` 或使用批注该类型 `OwnedAttribute` ，以将类型配置为拥有类型。</span><span class="sxs-lookup"><span data-stu-id="ce095-111">You can use the `OwnsOne` method in `OnModelCreating` or annotate the type with `OwnedAttribute` to configure the type as an owned type.</span></span>

<span data-ttu-id="ce095-112">在此示例中， `StreetAddress` 是一个无标识属性的类型。</span><span class="sxs-lookup"><span data-stu-id="ce095-112">In this example, `StreetAddress` is a type with no identity property.</span></span> <span data-ttu-id="ce095-113">它用作 Order 类型的属性来指定特定订单的发货地址。</span><span class="sxs-lookup"><span data-stu-id="ce095-113">It is used as a property of the Order type to specify the shipping address for a particular order.</span></span>

<span data-ttu-id="ce095-114">`OwnedAttribute`从另一个实体类型引用时，可以使用将其视为拥有的实体：</span><span class="sxs-lookup"><span data-stu-id="ce095-114">We can use the `OwnedAttribute` to treat it as an owned entity when referenced from another entity type:</span></span>

[!code-csharp[StreetAddress](../../../samples/core/Modeling/OwnedEntities/StreetAddress.cs?name=StreetAddress)]

[!code-csharp[Order](../../../samples/core/Modeling/OwnedEntities/Order.cs?name=Order)]

<span data-ttu-id="ce095-115">还可以使用 `OwnsOne` 中的方法 `OnModelCreating` 来指定 `ShippingAddress` 属性是实体类型的拥有实体 `Order` ，并根据需要配置其他方面。</span><span class="sxs-lookup"><span data-stu-id="ce095-115">It is also possible to use the `OwnsOne` method in `OnModelCreating` to specify that the `ShippingAddress` property is an Owned Entity of the `Order` entity type and to configure additional facets if needed.</span></span>

[!code-csharp[OwnsOne](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOne)]

<span data-ttu-id="ce095-116">如果该 `ShippingAddress` 属性是类型中的私有属性 `Order` ，则可以使用该方法的字符串版本 `OwnsOne` ：</span><span class="sxs-lookup"><span data-stu-id="ce095-116">If the `ShippingAddress` property is private in the `Order` type, you can use the string version of the `OwnsOne` method:</span></span>

[!code-csharp[OwnsOneString](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOneString)]

<span data-ttu-id="ce095-117">以上模型映射到以下数据库架构：</span><span class="sxs-lookup"><span data-stu-id="ce095-117">The model above is mapped to the following database schema:</span></span>

![包含 Sceenshot 的实体的数据库模型的数据库模型](_static/owned-entities-ownsone.png)

<span data-ttu-id="ce095-119">有关更多上下文，请参阅 [完整的示例项目](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/OwnedEntities) 。</span><span class="sxs-lookup"><span data-stu-id="ce095-119">See the [full sample project](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/OwnedEntities) for more context.</span></span>

> [!TIP]
> <span data-ttu-id="ce095-120">可以将拥有的实体类型标记为必需，有关详细信息，请参阅 [所需的一对一依赖项](xref:core/modeling/relationships#one-to-one) 。</span><span class="sxs-lookup"><span data-stu-id="ce095-120">The owned entity type can be marked as required, see [Required one-to-one dependents](xref:core/modeling/relationships#one-to-one) for more information.</span></span>

## <a name="implicit-keys"></a><span data-ttu-id="ce095-121">隐式键</span><span class="sxs-lookup"><span data-stu-id="ce095-121">Implicit keys</span></span>

<span data-ttu-id="ce095-122">`OwnsOne`通过引用导航配置的或通过引用导航发现的拥有的类型与所有者始终具有一对一的关系，因此，它们不需要其自己的键值，因为外键值是唯一的。</span><span class="sxs-lookup"><span data-stu-id="ce095-122">Owned types configured with `OwnsOne` or discovered through a reference navigation always have a one-to-one relationship with the owner, therefore they don't need their own key values as the foreign key values are unique.</span></span> <span data-ttu-id="ce095-123">在上面的示例中， `StreetAddress` 类型不需要定义键属性。</span><span class="sxs-lookup"><span data-stu-id="ce095-123">In the previous example, the `StreetAddress` type does not need to define a key property.</span></span>

<span data-ttu-id="ce095-124">为了理解 EF Core 如何跟踪这些对象，知道主键是作为所属类型的 [影子属性](xref:core/modeling/shadow-properties) 创建的，这会很有用。</span><span class="sxs-lookup"><span data-stu-id="ce095-124">In order to understand how EF Core tracks these objects, it is useful to know that a primary key is created as a [shadow property](xref:core/modeling/shadow-properties) for the owned type.</span></span> <span data-ttu-id="ce095-125">所拥有类型的实例的键值将与所有者实例的键的值相同。</span><span class="sxs-lookup"><span data-stu-id="ce095-125">The value of the key of an instance of the owned type will be the same as the value of the key of the owner instance.</span></span>

## <a name="collections-of-owned-types"></a><span data-ttu-id="ce095-126">拥有的类型的集合</span><span class="sxs-lookup"><span data-stu-id="ce095-126">Collections of owned types</span></span>

<span data-ttu-id="ce095-127">若要配置拥有类型的集合，请使用 `OwnsMany` 中的 `OnModelCreating` 。</span><span class="sxs-lookup"><span data-stu-id="ce095-127">To configure a collection of owned types use `OwnsMany` in `OnModelCreating`.</span></span>

<span data-ttu-id="ce095-128">拥有的类型需要主键。</span><span class="sxs-lookup"><span data-stu-id="ce095-128">Owned types need a primary key.</span></span> <span data-ttu-id="ce095-129">如果 .NET 类型上没有合适的候选属性，EF Core 可以尝试创建一个。</span><span class="sxs-lookup"><span data-stu-id="ce095-129">If there are no good candidates properties on the .NET type, EF Core can try to create one.</span></span> <span data-ttu-id="ce095-130">但是，当所有类型是通过集合定义的时，只需创建一个影子属性以同时充当所有者的外键和该拥有实例的主键，就像我们所做的那样 `OwnsOne` ：对于每个所有者，可以有多个拥有的类型实例，因此，所有者的密钥不足以为每个拥有的实例提供唯一标识。</span><span class="sxs-lookup"><span data-stu-id="ce095-130">However, when owned types are defined through a collection, it isn't enough to just create a shadow property to act as both the foreign key into the owner and the primary key of the owned instance, as we do for `OwnsOne`: there can be multiple owned type instances for each owner, and hence the key of the owner isn't enough to provide a unique identity for each owned instance.</span></span>

<span data-ttu-id="ce095-131">这两个最直接的解决方案是：</span><span class="sxs-lookup"><span data-stu-id="ce095-131">The two most straightforward solutions to this are:</span></span>

- <span data-ttu-id="ce095-132">在独立于指向所有者的外键的新属性上定义代理项主键。</span><span class="sxs-lookup"><span data-stu-id="ce095-132">Defining a surrogate primary key on a new property independent of the foreign key that points to the owner.</span></span> <span data-ttu-id="ce095-133">所有所有者都需要唯一的包含值 (例如，如果父项 {1} 具有子级 {1} ，则父项 {2} 不能具有子 {1}) ，因此，该值没有任何固有含义。</span><span class="sxs-lookup"><span data-stu-id="ce095-133">The contained values would need to be unique across all owners (e.g. if Parent {1} has Child {1}, then Parent {2} cannot have Child {1}), so the value doesn't have any inherent meaning.</span></span> <span data-ttu-id="ce095-134">由于外键不是主键的一部分，因此可以更改其值，因此，您可以将子级从一个父级移到另一个父级，但这通常会针对聚合语义进行。</span><span class="sxs-lookup"><span data-stu-id="ce095-134">Since the foreign key is not part of the primary key its values can be changed, so you could move a child from one parent to another one, however this usually goes against aggregate semantics.</span></span>
- <span data-ttu-id="ce095-135">使用外键和附加属性作为组合键。</span><span class="sxs-lookup"><span data-stu-id="ce095-135">Using the foreign key and an additional property as a composite key.</span></span> <span data-ttu-id="ce095-136">现在，附加属性值只需对于给定父 (是唯一的，因此，如果 Parent {1} 具有子级， {1,1} 则父项 {2} 仍可以具有子 {2,1}) 。</span><span class="sxs-lookup"><span data-stu-id="ce095-136">The additional property value now only needs to be unique for a given parent (so if Parent {1} has Child {1,1} then Parent {2} can still have Child {2,1}).</span></span> <span data-ttu-id="ce095-137">通过创建主键的外键部分，所有者和拥有的实体之间的关系将变为不可变的，并且更好地反映了聚合语义。</span><span class="sxs-lookup"><span data-stu-id="ce095-137">By making the foreign key part of the primary key the relationship between the owner and the owned entity becomes immutable and reflects aggregate semantics better.</span></span> <span data-ttu-id="ce095-138">这是 EF Core 默认情况下执行的操作。</span><span class="sxs-lookup"><span data-stu-id="ce095-138">This is what EF Core does by default.</span></span>

<span data-ttu-id="ce095-139">在此示例中，我们将使用 `Distributor` 类。</span><span class="sxs-lookup"><span data-stu-id="ce095-139">In this example we'll use the `Distributor` class.</span></span>

[!code-csharp[Distributor](../../../samples/core/Modeling/OwnedEntities/Distributor.cs?name=Distributor)]

<span data-ttu-id="ce095-140">默认情况下，在通过导航属性引用的所拥有的类型中使用的主键 `ShippingCenters` 将是 `("DistributorId", "Id")` `"DistributorId"` FK，而 `"Id"` 是唯一 `int` 值。</span><span class="sxs-lookup"><span data-stu-id="ce095-140">By default the primary key used for the owned type referenced through the `ShippingCenters` navigation property will be `("DistributorId", "Id")` where `"DistributorId"` is the FK and `"Id"` is a unique `int` value.</span></span>

<span data-ttu-id="ce095-141">配置不同的主键调用 `HasKey` 。</span><span class="sxs-lookup"><span data-stu-id="ce095-141">To configure a different primary key call `HasKey`.</span></span>

[!code-csharp[OwnsMany](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsMany)]

<span data-ttu-id="ce095-142">以上模型映射到以下数据库架构：</span><span class="sxs-lookup"><span data-stu-id="ce095-142">The model above is mapped to the following database schema:</span></span>

![包含拥有的集合的实体的数据库模型的 Sceenshot](_static/owned-entities-ownsmany.png)

## <a name="mapping-owned-types-with-table-splitting"></a><span data-ttu-id="ce095-144">将拥有的类型映射到表拆分</span><span class="sxs-lookup"><span data-stu-id="ce095-144">Mapping owned types with table splitting</span></span>

<span data-ttu-id="ce095-145">使用关系数据库时，默认情况下，引用拥有的类型将映射到与所有者相同的表。</span><span class="sxs-lookup"><span data-stu-id="ce095-145">When using relational databases, by default reference owned types are mapped to the same table as the owner.</span></span> <span data-ttu-id="ce095-146">这需要将表拆分为两个：某些列将用于存储所有者的数据，某些列将用于存储拥有实体的数据。</span><span class="sxs-lookup"><span data-stu-id="ce095-146">This requires splitting the table in two: some columns will be used to store the data of the owner, and some columns will be used to store data of the owned entity.</span></span> <span data-ttu-id="ce095-147">这是一种称为 [表拆分](xref:core/modeling/table-splitting)的常见功能。</span><span class="sxs-lookup"><span data-stu-id="ce095-147">This is a common feature known as [table splitting](xref:core/modeling/table-splitting).</span></span>

<span data-ttu-id="ce095-148">默认情况下，EF Core 会按照模式 _Navigation_OwnedEntityProperty_ 为拥有的实体类型的属性命名数据库列。</span><span class="sxs-lookup"><span data-stu-id="ce095-148">By default, EF Core will name the database columns for the properties of the owned entity type following the pattern _Navigation_OwnedEntityProperty_.</span></span> <span data-ttu-id="ce095-149">因此，这些 `StreetAddress` 属性将显示在 "Orders" 表中，名称为 "ShippingAddress_Street" 和 "ShippingAddress_City"。</span><span class="sxs-lookup"><span data-stu-id="ce095-149">Therefore the `StreetAddress` properties will appear in the 'Orders' table with the names 'ShippingAddress_Street' and 'ShippingAddress_City'.</span></span>

<span data-ttu-id="ce095-150">您可以使用 `HasColumnName` 方法重命名这些列。</span><span class="sxs-lookup"><span data-stu-id="ce095-150">You can use the `HasColumnName` method to rename those columns.</span></span>

[!code-csharp[ColumnNames](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=ColumnNames)]

> [!NOTE]
> <span data-ttu-id="ce095-151">大多数正常的实体类型配置方法（如 [Ignore](/dotnet/api/microsoft.entityframeworkcore.metadata.builders.ownednavigationbuilder.ignore) ）都可以通过相同的方式进行调用。</span><span class="sxs-lookup"><span data-stu-id="ce095-151">Most of the normal entity type configuration methods like [Ignore](/dotnet/api/microsoft.entityframeworkcore.metadata.builders.ownednavigationbuilder.ignore) can be called in the same way.</span></span>

## <a name="sharing-the-same-net-type-among-multiple-owned-types"></a><span data-ttu-id="ce095-152">在多个所拥有的类型之间共享相同的 .NET 类型</span><span class="sxs-lookup"><span data-stu-id="ce095-152">Sharing the same .NET type among multiple owned types</span></span>

<span data-ttu-id="ce095-153">一个拥有的实体类型可以是与另一个拥有的实体类型相同的 .NET 类型，因此，.NET 类型可能不足以标识某个所有者的类型。</span><span class="sxs-lookup"><span data-stu-id="ce095-153">An owned entity type can be of the same .NET type as another owned entity type, therefore the .NET type may not be enough to identify an owned type.</span></span>

<span data-ttu-id="ce095-154">在这些情况下，从所有者指向拥有的实体的属性将成为所拥有实体类型的 _定义导航_ 。</span><span class="sxs-lookup"><span data-stu-id="ce095-154">In those cases, the property pointing from the owner to the owned entity becomes the _defining navigation_ of the owned entity type.</span></span> <span data-ttu-id="ce095-155">从 EF Core 的角度来看，定义导航是类型标识与 .NET 类型的一部分。</span><span class="sxs-lookup"><span data-stu-id="ce095-155">From the perspective of EF Core, the defining navigation is part of the type's identity alongside the .NET type.</span></span>

<span data-ttu-id="ce095-156">例如，在下面的类中， `ShippingAddress` 和 `BillingAddress` 都是相同的 .net 类型 `StreetAddress` 。</span><span class="sxs-lookup"><span data-stu-id="ce095-156">For example, in the following class `ShippingAddress` and `BillingAddress` are both of the same .NET type, `StreetAddress`.</span></span>

[!code-csharp[OrderDetails](../../../samples/core/Modeling/OwnedEntities/OrderDetails.cs?name=OrderDetails)]

<span data-ttu-id="ce095-157">为了理解 EF Core 如何区分跟踪的这些对象的实例，可能会认为定义导航已成为实例的键的一部分，以及所有者的键的值和拥有的类型的 .NET 类型。</span><span class="sxs-lookup"><span data-stu-id="ce095-157">In order to understand how EF Core will distinguish tracked instances of these objects, it may be useful to think that the defining navigation has become part of the key of the instance alongside the value of the key of the owner and the .NET type of the owned type.</span></span>

## <a name="nested-owned-types"></a><span data-ttu-id="ce095-158">嵌套的所属类型</span><span class="sxs-lookup"><span data-stu-id="ce095-158">Nested owned types</span></span>

<span data-ttu-id="ce095-159">在此示例 `OrderDetails` 中 `BillingAddress` ，拥有和 `ShippingAddress` ，这两种类型都是 `StreetAddress` 。</span><span class="sxs-lookup"><span data-stu-id="ce095-159">In this example `OrderDetails` owns `BillingAddress` and `ShippingAddress`, which are both `StreetAddress` types.</span></span> <span data-ttu-id="ce095-160">然后 `OrderDetails` 归 `DetailedOrder` 类型所有。</span><span class="sxs-lookup"><span data-stu-id="ce095-160">Then `OrderDetails` is owned by the `DetailedOrder` type.</span></span>

[!code-csharp[DetailedOrder](../../../samples/core/Modeling/OwnedEntities/DetailedOrder.cs?name=DetailedOrder)]

[!code-csharp[OrderStatus](../../../samples/core/Modeling/OwnedEntities/OrderStatus.cs?name=OrderStatus)]

<span data-ttu-id="ce095-161">每个指向所拥有的类型的导航都定义一个具有完全独立配置的单独实体类型。</span><span class="sxs-lookup"><span data-stu-id="ce095-161">Each navigation to an owned type defines a separate entity type with completely independent configuration.</span></span>

<span data-ttu-id="ce095-162">除了嵌套的类型外，拥有的类型还可以引用常规实体，该实体可以是所有者，也可以是其他实体，前提是拥有的实体在依赖方。</span><span class="sxs-lookup"><span data-stu-id="ce095-162">In addition to nested owned types, an owned type can reference a regular entity which can be either the owner or a different entity as long as the owned entity is on the dependent side.</span></span> <span data-ttu-id="ce095-163">此功能在 EF6 中除复杂类型外，会设置拥有的实体类型。</span><span class="sxs-lookup"><span data-stu-id="ce095-163">This capability sets owned entity types apart from complex types in EF6.</span></span>

[!code-csharp[OrderDetails](../../../samples/core/Modeling/OwnedEntities/OrderDetails.cs?name=OrderDetails)]

## <a name="configuring-owned-types"></a><span data-ttu-id="ce095-164">配置拥有类型</span><span class="sxs-lookup"><span data-stu-id="ce095-164">Configuring owned types</span></span>

<span data-ttu-id="ce095-165">可以 `OwnsOne` 通过在流畅调用中将方法链接在一起来配置此模型：</span><span class="sxs-lookup"><span data-stu-id="ce095-165">It is possible to chain the `OwnsOne` method in a fluent call to configure this model:</span></span>

[!code-csharp[OwnsOneNested](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOneNested)]

<span data-ttu-id="ce095-166">请注意 `WithOwner` 用于定义在所有者返回导航属性的调用。</span><span class="sxs-lookup"><span data-stu-id="ce095-166">Notice the `WithOwner` call used to define the navigation property pointing back at the owner.</span></span> <span data-ttu-id="ce095-167">若要定义对不属于所有权关系的所有者实体类型的导航， `WithOwner()` 应不使用任何参数调用。</span><span class="sxs-lookup"><span data-stu-id="ce095-167">To define a navigation to the owner entity type that's not part of the ownership relationship `WithOwner()` should be called without any arguments.</span></span>

<span data-ttu-id="ce095-168">还可以在和上使用来实现此结果 `OwnedAttribute` `OrderDetails` `StreetAddress` 。</span><span class="sxs-lookup"><span data-stu-id="ce095-168">It is also possible to achieve this result using `OwnedAttribute` on both `OrderDetails` and `StreetAddress`.</span></span>

<span data-ttu-id="ce095-169">此外，请注意 `Navigation` 调用。</span><span class="sxs-lookup"><span data-stu-id="ce095-169">In addition, notice the `Navigation` call.</span></span> <span data-ttu-id="ce095-170">在 EFCore 5.0 中，可以将导航属性进一步配置为 [非拥有的导航属性](xref:core/modeling/relationships#configuring-navigation-properties)。</span><span class="sxs-lookup"><span data-stu-id="ce095-170">In EFCore 5.0, navigation properties to owned types can be further configured [as for non-owned navigation properties](xref:core/modeling/relationships#configuring-navigation-properties).</span></span>

<span data-ttu-id="ce095-171">以上模型映射到以下数据库架构：</span><span class="sxs-lookup"><span data-stu-id="ce095-171">The model above is mapped to the following database schema:</span></span>

![包含嵌套的引用的实体的数据库模型的屏幕截图](_static/owned-entities-nested.png)

## <a name="storing-owned-types-in-separate-tables"></a><span data-ttu-id="ce095-173">将拥有的类型存储在单独的表中</span><span class="sxs-lookup"><span data-stu-id="ce095-173">Storing owned types in separate tables</span></span>

<span data-ttu-id="ce095-174">与 EF6 复杂类型不同的是，拥有的类型可以存储在所有者的单独表中。</span><span class="sxs-lookup"><span data-stu-id="ce095-174">Also unlike EF6 complex types, owned types can be stored in a separate table from the owner.</span></span> <span data-ttu-id="ce095-175">若要重写将拥有的类型映射到与所有者相同的表的约定，可以直接调用 `ToTable` 并提供不同的表名。</span><span class="sxs-lookup"><span data-stu-id="ce095-175">In order to override the convention that maps an owned type to the same table as the owner, you can simply call `ToTable` and provide a different table name.</span></span> <span data-ttu-id="ce095-176">下面的示例将映射以下 `OrderDetails` 两个地址，并将其作为一个单独的表 `DetailedOrder` ：</span><span class="sxs-lookup"><span data-stu-id="ce095-176">The following example will map `OrderDetails` and its two addresses to a separate table from `DetailedOrder`:</span></span>

[!code-csharp[OwnsOneTable](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOneTable)]

<span data-ttu-id="ce095-177">还可以使用 `TableAttribute` 来实现此目的，但请注意，如果有多个导航到拥有的类型，则这会失败，因为在这种情况下，多个实体类型会映射到同一个表。</span><span class="sxs-lookup"><span data-stu-id="ce095-177">It is also possible to use the `TableAttribute` to accomplish this, but note that this would fail if there are multiple navigations to the owned type since in that case multiple entity types would be mapped to the same table.</span></span>

## <a name="querying-owned-types"></a><span data-ttu-id="ce095-178">查询拥有的类型</span><span class="sxs-lookup"><span data-stu-id="ce095-178">Querying owned types</span></span>

<span data-ttu-id="ce095-179">查询所有者时，固有类型将默认包含在内。</span><span class="sxs-lookup"><span data-stu-id="ce095-179">When querying the owner the owned types will be included by default.</span></span> <span data-ttu-id="ce095-180">不需要使用 `Include` 方法，即使所有类型都存储在单独的表中。</span><span class="sxs-lookup"><span data-stu-id="ce095-180">It is not necessary to use the `Include` method, even if the owned types are stored in a separate table.</span></span> <span data-ttu-id="ce095-181">根据前面所述的模型，下面的查询将获得 `Order` ，其中 `OrderDetails` 两个数据库都拥有 `StreetAddresses` ：</span><span class="sxs-lookup"><span data-stu-id="ce095-181">Based on the model described before, the following query will get `Order`, `OrderDetails` and the two owned `StreetAddresses` from the database:</span></span>

[!code-csharp[DetailedOrderQuery](../../../samples/core/Modeling/OwnedEntities/Program.cs?name=DetailedOrderQuery)]

## <a name="limitations"></a><span data-ttu-id="ce095-182">限制</span><span class="sxs-lookup"><span data-stu-id="ce095-182">Limitations</span></span>

<span data-ttu-id="ce095-183">其中一些限制对于拥有的实体类型的工作方式很重要，但其他一些限制是我们可以在未来版本中删除的限制：</span><span class="sxs-lookup"><span data-stu-id="ce095-183">Some of these limitations are fundamental to how owned entity types work, but some others are restrictions that we may be able to remove in future releases:</span></span>

### <a name="by-design-restrictions"></a><span data-ttu-id="ce095-184">按设计限制</span><span class="sxs-lookup"><span data-stu-id="ce095-184">By-design restrictions</span></span>

- <span data-ttu-id="ce095-185">不能 `DbSet<T>` 为拥有的类型创建。</span><span class="sxs-lookup"><span data-stu-id="ce095-185">You cannot create a `DbSet<T>` for an owned type.</span></span>
- <span data-ttu-id="ce095-186">不能 `Entity<T>()` 对拥有的类型调用 `ModelBuilder` 。</span><span class="sxs-lookup"><span data-stu-id="ce095-186">You cannot call `Entity<T>()` with an owned type on `ModelBuilder`.</span></span>
- <span data-ttu-id="ce095-187">多个所有者不能共享拥有的实体类型的实例 (这是一个已知的值对象方案，不能使用) 的拥有实体类型实现。</span><span class="sxs-lookup"><span data-stu-id="ce095-187">Instances of owned entity types cannot be shared by multiple owners (this is a well-known scenario for value objects that cannot be implemented using owned entity types).</span></span>

### <a name="current-shortcomings"></a><span data-ttu-id="ce095-188">当前缺陷</span><span class="sxs-lookup"><span data-stu-id="ce095-188">Current shortcomings</span></span>

- <span data-ttu-id="ce095-189">拥有的实体类型不能具有继承层次结构</span><span class="sxs-lookup"><span data-stu-id="ce095-189">Owned entity types cannot have inheritance hierarchies</span></span>

### <a name="shortcomings-in-previous-versions"></a><span data-ttu-id="ce095-190">以前版本中的缺点</span><span class="sxs-lookup"><span data-stu-id="ce095-190">Shortcomings in previous versions</span></span>

- <span data-ttu-id="ce095-191">在 EF Core 中，对拥有的实体类型的引用导航不能为 null，除非它们显式映射到与所有者不同的表。</span><span class="sxs-lookup"><span data-stu-id="ce095-191">In EF Core 2.x reference navigations to owned entity types cannot be null unless they are explicitly mapped to a separate table from the owner.</span></span>
- <span data-ttu-id="ce095-192">在 EF Core 1.x 中，所有与所有者映射到同一个表的实体类型的列都将始终标记为可以为 null。</span><span class="sxs-lookup"><span data-stu-id="ce095-192">In EF Core 3.x the columns for owned entity types mapped to the same table as the owner are always marked as nullable.</span></span>
