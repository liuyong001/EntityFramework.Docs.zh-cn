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
# <a name="owned-entity-types"></a>从属实体类型

EF Core 允许您为只能出现在其他实体类型的导航属性中的实体类型建模。 它们称为 _拥有的实体类型_。 包含拥有的实体类型的实体是其 _所有者_。

拥有的实体实质上是所有者的一部分，并且在没有它的情况下不存在，它们在概念上类似于 [聚合](https://martinfowler.com/bliki/DDD_Aggregate.html)。 这意味着，拥有的实体由与所有者的关系的从属方定义。

## <a name="explicit-configuration"></a>显式配置

所有实体类型永远不会通过约定 EF Core 在模型中。 您可以使用 `OwnsOne` 中的方法 `OnModelCreating` 或使用批注该类型 `OwnedAttribute` ，以将类型配置为拥有类型。

在此示例中， `StreetAddress` 是一个无标识属性的类型。 它用作 Order 类型的属性来指定特定订单的发货地址。

`OwnedAttribute`从另一个实体类型引用时，可以使用将其视为拥有的实体：

[!code-csharp[StreetAddress](../../../samples/core/Modeling/OwnedEntities/StreetAddress.cs?name=StreetAddress)]

[!code-csharp[Order](../../../samples/core/Modeling/OwnedEntities/Order.cs?name=Order)]

还可以使用 `OwnsOne` 中的方法 `OnModelCreating` 来指定 `ShippingAddress` 属性是实体类型的拥有实体 `Order` ，并根据需要配置其他方面。

[!code-csharp[OwnsOne](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOne)]

如果该 `ShippingAddress` 属性是类型中的私有属性 `Order` ，则可以使用该方法的字符串版本 `OwnsOne` ：

[!code-csharp[OwnsOneString](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOneString)]

以上模型映射到以下数据库架构：

![包含 Sceenshot 的实体的数据库模型的数据库模型](_static/owned-entities-ownsone.png)

有关更多上下文，请参阅 [完整的示例项目](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/OwnedEntities) 。

> [!TIP]
> 可以将拥有的实体类型标记为必需，有关详细信息，请参阅 [所需的一对一依赖项](xref:core/modeling/relationships#one-to-one) 。

## <a name="implicit-keys"></a>隐式键

`OwnsOne`通过引用导航配置的或通过引用导航发现的拥有的类型与所有者始终具有一对一的关系，因此，它们不需要其自己的键值，因为外键值是唯一的。 在上面的示例中， `StreetAddress` 类型不需要定义键属性。

为了理解 EF Core 如何跟踪这些对象，知道主键是作为所属类型的 [影子属性](xref:core/modeling/shadow-properties) 创建的，这会很有用。 所拥有类型的实例的键值将与所有者实例的键的值相同。

## <a name="collections-of-owned-types"></a>拥有的类型的集合

若要配置拥有类型的集合，请使用 `OwnsMany` 中的 `OnModelCreating` 。

拥有的类型需要主键。 如果 .NET 类型上没有合适的候选属性，EF Core 可以尝试创建一个。 但是，当所有类型是通过集合定义的时，只需创建一个影子属性以同时充当所有者的外键和该拥有实例的主键，就像我们所做的那样 `OwnsOne` ：对于每个所有者，可以有多个拥有的类型实例，因此，所有者的密钥不足以为每个拥有的实例提供唯一标识。

这两个最直接的解决方案是：

- 在独立于指向所有者的外键的新属性上定义代理项主键。 所有所有者都需要唯一的包含值 (例如，如果父项 {1} 具有子级 {1} ，则父项 {2} 不能具有子 {1}) ，因此，该值没有任何固有含义。 由于外键不是主键的一部分，因此可以更改其值，因此，您可以将子级从一个父级移到另一个父级，但这通常会针对聚合语义进行。
- 使用外键和附加属性作为组合键。 现在，附加属性值只需对于给定父 (是唯一的，因此，如果 Parent {1} 具有子级， {1,1} 则父项 {2} 仍可以具有子 {2,1}) 。 通过创建主键的外键部分，所有者和拥有的实体之间的关系将变为不可变的，并且更好地反映了聚合语义。 这是 EF Core 默认情况下执行的操作。

在此示例中，我们将使用 `Distributor` 类。

[!code-csharp[Distributor](../../../samples/core/Modeling/OwnedEntities/Distributor.cs?name=Distributor)]

默认情况下，在通过导航属性引用的所拥有的类型中使用的主键 `ShippingCenters` 将是 `("DistributorId", "Id")` `"DistributorId"` FK，而 `"Id"` 是唯一 `int` 值。

配置不同的主键调用 `HasKey` 。

[!code-csharp[OwnsMany](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsMany)]

以上模型映射到以下数据库架构：

![包含拥有的集合的实体的数据库模型的 Sceenshot](_static/owned-entities-ownsmany.png)

## <a name="mapping-owned-types-with-table-splitting"></a>将拥有的类型映射到表拆分

使用关系数据库时，默认情况下，引用拥有的类型将映射到与所有者相同的表。 这需要将表拆分为两个：某些列将用于存储所有者的数据，某些列将用于存储拥有实体的数据。 这是一种称为 [表拆分](xref:core/modeling/table-splitting)的常见功能。

默认情况下，EF Core 会按照模式 _Navigation_OwnedEntityProperty_ 为拥有的实体类型的属性命名数据库列。 因此，这些 `StreetAddress` 属性将显示在 "Orders" 表中，名称为 "ShippingAddress_Street" 和 "ShippingAddress_City"。

您可以使用 `HasColumnName` 方法重命名这些列。

[!code-csharp[ColumnNames](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=ColumnNames)]

> [!NOTE]
> 大多数正常的实体类型配置方法（如 [Ignore](/dotnet/api/microsoft.entityframeworkcore.metadata.builders.ownednavigationbuilder.ignore) ）都可以通过相同的方式进行调用。

## <a name="sharing-the-same-net-type-among-multiple-owned-types"></a>在多个所拥有的类型之间共享相同的 .NET 类型

一个拥有的实体类型可以是与另一个拥有的实体类型相同的 .NET 类型，因此，.NET 类型可能不足以标识某个所有者的类型。

在这些情况下，从所有者指向拥有的实体的属性将成为所拥有实体类型的 _定义导航_ 。 从 EF Core 的角度来看，定义导航是类型标识与 .NET 类型的一部分。

例如，在下面的类中， `ShippingAddress` 和 `BillingAddress` 都是相同的 .net 类型 `StreetAddress` 。

[!code-csharp[OrderDetails](../../../samples/core/Modeling/OwnedEntities/OrderDetails.cs?name=OrderDetails)]

为了理解 EF Core 如何区分跟踪的这些对象的实例，可能会认为定义导航已成为实例的键的一部分，以及所有者的键的值和拥有的类型的 .NET 类型。

## <a name="nested-owned-types"></a>嵌套的所属类型

在此示例 `OrderDetails` 中 `BillingAddress` ，拥有和 `ShippingAddress` ，这两种类型都是 `StreetAddress` 。 然后 `OrderDetails` 归 `DetailedOrder` 类型所有。

[!code-csharp[DetailedOrder](../../../samples/core/Modeling/OwnedEntities/DetailedOrder.cs?name=DetailedOrder)]

[!code-csharp[OrderStatus](../../../samples/core/Modeling/OwnedEntities/OrderStatus.cs?name=OrderStatus)]

每个指向所拥有的类型的导航都定义一个具有完全独立配置的单独实体类型。

除了嵌套的类型外，拥有的类型还可以引用常规实体，该实体可以是所有者，也可以是其他实体，前提是拥有的实体在依赖方。 此功能在 EF6 中除复杂类型外，会设置拥有的实体类型。

[!code-csharp[OrderDetails](../../../samples/core/Modeling/OwnedEntities/OrderDetails.cs?name=OrderDetails)]

## <a name="configuring-owned-types"></a>配置拥有类型

可以 `OwnsOne` 通过在流畅调用中将方法链接在一起来配置此模型：

[!code-csharp[OwnsOneNested](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOneNested)]

请注意 `WithOwner` 用于定义在所有者返回导航属性的调用。 若要定义对不属于所有权关系的所有者实体类型的导航， `WithOwner()` 应不使用任何参数调用。

还可以在和上使用来实现此结果 `OwnedAttribute` `OrderDetails` `StreetAddress` 。

此外，请注意 `Navigation` 调用。 在 EFCore 5.0 中，可以将导航属性进一步配置为 [非拥有的导航属性](xref:core/modeling/relationships#configuring-navigation-properties)。

以上模型映射到以下数据库架构：

![包含嵌套的引用的实体的数据库模型的屏幕截图](_static/owned-entities-nested.png)

## <a name="storing-owned-types-in-separate-tables"></a>将拥有的类型存储在单独的表中

与 EF6 复杂类型不同的是，拥有的类型可以存储在所有者的单独表中。 若要重写将拥有的类型映射到与所有者相同的表的约定，可以直接调用 `ToTable` 并提供不同的表名。 下面的示例将映射以下 `OrderDetails` 两个地址，并将其作为一个单独的表 `DetailedOrder` ：

[!code-csharp[OwnsOneTable](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOneTable)]

还可以使用 `TableAttribute` 来实现此目的，但请注意，如果有多个导航到拥有的类型，则这会失败，因为在这种情况下，多个实体类型会映射到同一个表。

## <a name="querying-owned-types"></a>查询拥有的类型

查询所有者时，固有类型将默认包含在内。 不需要使用 `Include` 方法，即使所有类型都存储在单独的表中。 根据前面所述的模型，下面的查询将获得 `Order` ，其中 `OrderDetails` 两个数据库都拥有 `StreetAddresses` ：

[!code-csharp[DetailedOrderQuery](../../../samples/core/Modeling/OwnedEntities/Program.cs?name=DetailedOrderQuery)]

## <a name="limitations"></a>限制

其中一些限制对于拥有的实体类型的工作方式很重要，但其他一些限制是我们可以在未来版本中删除的限制：

### <a name="by-design-restrictions"></a>按设计限制

- 不能 `DbSet<T>` 为拥有的类型创建。
- 不能 `Entity<T>()` 对拥有的类型调用 `ModelBuilder` 。
- 多个所有者不能共享拥有的实体类型的实例 (这是一个已知的值对象方案，不能使用) 的拥有实体类型实现。

### <a name="current-shortcomings"></a>当前缺陷

- 拥有的实体类型不能具有继承层次结构

### <a name="shortcomings-in-previous-versions"></a>以前版本中的缺点

- 在 EF Core 中，对拥有的实体类型的引用导航不能为 null，除非它们显式映射到与所有者不同的表。
- 在 EF Core 1.x 中，所有与所有者映射到同一个表的实体类型的列都将始终标记为可以为 null。
