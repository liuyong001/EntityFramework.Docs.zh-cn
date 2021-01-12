---
title: 密钥-EF Core
description: 如何在使用 Entity Framework Core 时配置实体类型的密钥
author: AndriySvyryd
ms.date: 11/06/2019
uid: core/modeling/keys
ms.openlocfilehash: 805396a13227aa62ed86ac17c742d055d7a22bbf
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129182"
---
# <a name="keys"></a>键

键充当每个实体实例的唯一标识符。 EF 中的大多数实体都有一个键，此键映射到关系数据库中 *主键* 的概念 (对于没有键的实体，请参阅 [无键实体](xref:core/modeling/keyless-entity-types)) 。 实体可以有超过主键的其他键 (参阅 [备用键](#alternate-keys) 以获取详细信息) 。

按照约定，将名为 `Id` 或的属性 `<type name>Id` 配置为实体的主键。

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/KeyId.cs?name=KeyId&highlight=3,11)]

> [!NOTE]
> [拥有的实体类型](xref:core/modeling/owned-entities) 使用不同的规则来定义密钥。

可以将单个属性配置为实体的主键，如下所示：

## <a name="data-annotations"></a>[数据批注](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/KeySingle.cs?name=KeySingle&highlight=3)]

## <a name="fluent-api"></a>[Fluent API](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/KeySingle.cs?name=KeySingle&highlight=4)]

***

你还可以将多个属性配置为实体的键，这称为组合键。 复合密钥只能使用熟知的 API 进行配置;约定将永远不会设置组合键，你不能使用数据批注来配置它。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/KeyComposite.cs?name=KeyComposite&highlight=4)]

## <a name="primary-key-name"></a>主键名称

按照约定，使用名称创建关系数据库主键 `PK_<type name>` 。 可以按如下所示配置 primary key 约束的名称：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/KeyName.cs?name=KeyName&highlight=5)]

## <a name="key-types-and-values"></a>键类型和值

虽然 EF Core 支持使用任何基元类型的属性作为主键（包括 `string` 、 `Guid` 和其他类型），但 `byte[]` 并不是所有数据库都支持将所有类型作为键。 在某些情况下，可以自动将键值转换为支持的类型，否则应 [手动指定](xref:core/modeling/value-conversions)转换。

在将新实体添加到上下文时，键属性必须始终具有非默认值，但某些类型将 [由数据库生成](xref:core/modeling/generated-properties)。 在这种情况下，当添加实体进行跟踪时，EF 会尝试生成一个临时值。 在调用 [SaveChanges](/dotnet/api/Microsoft.EntityFrameworkCore.DbContext.SaveChanges) 后，临时值将替换为数据库生成的值。

> [!Important]
> 如果某个键属性的值由数据库生成，而在添加实体时指定了一个非默认值，则 EF 将假定该实体在数据库中已存在，并且将尝试对其进行更新而不是插入一个新的值。 若要避免这种情况，请禁用值生成或了解 [如何为生成的属性指定显式值](xref:core/saving/explicit-values-generated-properties)。

## <a name="alternate-keys"></a>备用键

除了主键外，备用键还可用作每个实体实例的替代唯一标识符;它可用作关系的目标。 当使用关系数据库时，这将映射到) 的替代键 (列上的唯一索引/约束的概念，以及引用 () 的列的一个或多个 foreign key 约束。

> [!TIP]
> 如果只是想要在列上强制唯一性，请定义唯一索引而不是备用键 (参阅 [索引](xref:core/modeling/indexes)) 。 在 EF 中，备用键为只读，并在唯一索引之上提供附加语义，因为它们可用作外键的目标。

通常会在需要时为您引入备用密钥，而不需要手动配置。 按照约定，当您标识的属性不是作为关系目标的主键时，将为您引入备用密钥。

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/AlternateKey.cs?name=AlternateKey&highlight=12)]

你还可以将单个属性配置为备用密钥：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/AlternateKeySingle.cs?name=AlternateKeySingle&highlight=4)]

你还可以将多个属性配置为替代密钥， (称为复合备用键) ：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/AlternateKeyComposite.cs?name=AlternateKeyComposite&highlight=4)]

最后，按照约定，为替代键引入的索引和约束将被命名为 `AK_<type name>_<property name>` (，以便复合备用键 `<property name>` 成为) 的属性名称列表。 您可以配置备用密钥的 index 和 unique 约束的名称：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/AlternateKeyName.cs?name=AlternateKeyName&highlight=5)]
