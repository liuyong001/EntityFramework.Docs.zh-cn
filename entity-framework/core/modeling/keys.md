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
# <a name="keys"></a><span data-ttu-id="d4a61-103">键</span><span class="sxs-lookup"><span data-stu-id="d4a61-103">Keys</span></span>

<span data-ttu-id="d4a61-104">键充当每个实体实例的唯一标识符。</span><span class="sxs-lookup"><span data-stu-id="d4a61-104">A key serves as a unique identifier for each entity instance.</span></span> <span data-ttu-id="d4a61-105">EF 中的大多数实体都有一个键，此键映射到关系数据库中 *主键* 的概念 (对于没有键的实体，请参阅 [无键实体](xref:core/modeling/keyless-entity-types)) 。</span><span class="sxs-lookup"><span data-stu-id="d4a61-105">Most entities in EF have a single key, which maps to the concept of a *primary key* in relational databases (for entities without keys, see [Keyless entities](xref:core/modeling/keyless-entity-types)).</span></span> <span data-ttu-id="d4a61-106">实体可以有超过主键的其他键 (参阅 [备用键](#alternate-keys) 以获取详细信息) 。</span><span class="sxs-lookup"><span data-stu-id="d4a61-106">Entities can have additional keys beyond the primary key (see [Alternate Keys](#alternate-keys) for more information).</span></span>

<span data-ttu-id="d4a61-107">按照约定，将名为 `Id` 或的属性 `<type name>Id` 配置为实体的主键。</span><span class="sxs-lookup"><span data-stu-id="d4a61-107">By convention, a property named `Id` or `<type name>Id` will be configured as the primary key of an entity.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/KeyId.cs?name=KeyId&highlight=3,11)]

> [!NOTE]
> <span data-ttu-id="d4a61-108">[拥有的实体类型](xref:core/modeling/owned-entities) 使用不同的规则来定义密钥。</span><span class="sxs-lookup"><span data-stu-id="d4a61-108">[Owned entity types](xref:core/modeling/owned-entities) use different rules to define keys.</span></span>

<span data-ttu-id="d4a61-109">可以将单个属性配置为实体的主键，如下所示：</span><span class="sxs-lookup"><span data-stu-id="d4a61-109">You can configure a single property to be the primary key of an entity as follows:</span></span>

## <a name="data-annotations"></a>[<span data-ttu-id="d4a61-110">数据批注</span><span class="sxs-lookup"><span data-stu-id="d4a61-110">Data Annotations</span></span>](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/KeySingle.cs?name=KeySingle&highlight=3)]

## <a name="fluent-api"></a>[<span data-ttu-id="d4a61-111">Fluent API</span><span class="sxs-lookup"><span data-stu-id="d4a61-111">Fluent API</span></span>](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/KeySingle.cs?name=KeySingle&highlight=4)]

***

<span data-ttu-id="d4a61-112">你还可以将多个属性配置为实体的键，这称为组合键。</span><span class="sxs-lookup"><span data-stu-id="d4a61-112">You can also configure multiple properties to be the key of an entity - this is known as a composite key.</span></span> <span data-ttu-id="d4a61-113">复合密钥只能使用熟知的 API 进行配置;约定将永远不会设置组合键，你不能使用数据批注来配置它。</span><span class="sxs-lookup"><span data-stu-id="d4a61-113">Composite keys can only be configured using the Fluent API; conventions will never set up a composite key, and you can not use Data Annotations to configure one.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/KeyComposite.cs?name=KeyComposite&highlight=4)]

## <a name="primary-key-name"></a><span data-ttu-id="d4a61-114">主键名称</span><span class="sxs-lookup"><span data-stu-id="d4a61-114">Primary key name</span></span>

<span data-ttu-id="d4a61-115">按照约定，使用名称创建关系数据库主键 `PK_<type name>` 。</span><span class="sxs-lookup"><span data-stu-id="d4a61-115">By convention, on relational databases primary keys are created with the name `PK_<type name>`.</span></span> <span data-ttu-id="d4a61-116">可以按如下所示配置 primary key 约束的名称：</span><span class="sxs-lookup"><span data-stu-id="d4a61-116">You can configure the name of the primary key constraint as follows:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/KeyName.cs?name=KeyName&highlight=5)]

## <a name="key-types-and-values"></a><span data-ttu-id="d4a61-117">键类型和值</span><span class="sxs-lookup"><span data-stu-id="d4a61-117">Key types and values</span></span>

<span data-ttu-id="d4a61-118">虽然 EF Core 支持使用任何基元类型的属性作为主键（包括 `string` 、 `Guid` 和其他类型），但 `byte[]` 并不是所有数据库都支持将所有类型作为键。</span><span class="sxs-lookup"><span data-stu-id="d4a61-118">While EF Core supports using properties of any primitive type as the primary key, including `string`, `Guid`, `byte[]` and others, not all databases support all types as keys.</span></span> <span data-ttu-id="d4a61-119">在某些情况下，可以自动将键值转换为支持的类型，否则应 [手动指定](xref:core/modeling/value-conversions)转换。</span><span class="sxs-lookup"><span data-stu-id="d4a61-119">In some cases the key values can be converted to a supported type automatically, otherwise the conversion should be [specified manually](xref:core/modeling/value-conversions).</span></span>

<span data-ttu-id="d4a61-120">在将新实体添加到上下文时，键属性必须始终具有非默认值，但某些类型将 [由数据库生成](xref:core/modeling/generated-properties)。</span><span class="sxs-lookup"><span data-stu-id="d4a61-120">Key properties must always have a non-default value when adding a new entity to the context, but some types will be [generated by the database](xref:core/modeling/generated-properties).</span></span> <span data-ttu-id="d4a61-121">在这种情况下，当添加实体进行跟踪时，EF 会尝试生成一个临时值。</span><span class="sxs-lookup"><span data-stu-id="d4a61-121">In that case EF will try to generate a temporary value when the entity is added for tracking purposes.</span></span> <span data-ttu-id="d4a61-122">在调用 [SaveChanges](/dotnet/api/Microsoft.EntityFrameworkCore.DbContext.SaveChanges) 后，临时值将替换为数据库生成的值。</span><span class="sxs-lookup"><span data-stu-id="d4a61-122">After [SaveChanges](/dotnet/api/Microsoft.EntityFrameworkCore.DbContext.SaveChanges) is called the temporary value will be replaced by the value generated by the database.</span></span>

> [!Important]
> <span data-ttu-id="d4a61-123">如果某个键属性的值由数据库生成，而在添加实体时指定了一个非默认值，则 EF 将假定该实体在数据库中已存在，并且将尝试对其进行更新而不是插入一个新的值。</span><span class="sxs-lookup"><span data-stu-id="d4a61-123">If a key property has its value generated by the database and a non-default value is specified when an entity is added, then EF will assume that the entity already exists in the database and will try to update it instead of inserting a new one.</span></span> <span data-ttu-id="d4a61-124">若要避免这种情况，请禁用值生成或了解 [如何为生成的属性指定显式值](xref:core/saving/explicit-values-generated-properties)。</span><span class="sxs-lookup"><span data-stu-id="d4a61-124">To avoid this turn off value generation or see [how to specify explicit values for generated properties](xref:core/saving/explicit-values-generated-properties).</span></span>

## <a name="alternate-keys"></a><span data-ttu-id="d4a61-125">备用键</span><span class="sxs-lookup"><span data-stu-id="d4a61-125">Alternate Keys</span></span>

<span data-ttu-id="d4a61-126">除了主键外，备用键还可用作每个实体实例的替代唯一标识符;它可用作关系的目标。</span><span class="sxs-lookup"><span data-stu-id="d4a61-126">An alternate key serves as an alternate unique identifier for each entity instance in addition to the primary key; it can be used as the target of a relationship.</span></span> <span data-ttu-id="d4a61-127">当使用关系数据库时，这将映射到) 的替代键 (列上的唯一索引/约束的概念，以及引用 () 的列的一个或多个 foreign key 约束。</span><span class="sxs-lookup"><span data-stu-id="d4a61-127">When using a relational database this maps to the concept of a unique index/constraint on the alternate key column(s) and one or more foreign key constraints that reference the column(s).</span></span>

> [!TIP]
> <span data-ttu-id="d4a61-128">如果只是想要在列上强制唯一性，请定义唯一索引而不是备用键 (参阅 [索引](xref:core/modeling/indexes)) 。</span><span class="sxs-lookup"><span data-stu-id="d4a61-128">If you just want to enforce uniqueness on a column, define a unique index rather than an alternate key (see [Indexes](xref:core/modeling/indexes)).</span></span> <span data-ttu-id="d4a61-129">在 EF 中，备用键为只读，并在唯一索引之上提供附加语义，因为它们可用作外键的目标。</span><span class="sxs-lookup"><span data-stu-id="d4a61-129">In EF, alternate keys are read-only and provide additional semantics over unique indexes because they can be used as the target of a foreign key.</span></span>

<span data-ttu-id="d4a61-130">通常会在需要时为您引入备用密钥，而不需要手动配置。</span><span class="sxs-lookup"><span data-stu-id="d4a61-130">Alternate keys are typically introduced for you when needed and you do not need to manually configure them.</span></span> <span data-ttu-id="d4a61-131">按照约定，当您标识的属性不是作为关系目标的主键时，将为您引入备用密钥。</span><span class="sxs-lookup"><span data-stu-id="d4a61-131">By convention, an alternate key is introduced for you when you identify a property which isn't the primary key as the target of a relationship.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/AlternateKey.cs?name=AlternateKey&highlight=12)]

<span data-ttu-id="d4a61-132">你还可以将单个属性配置为备用密钥：</span><span class="sxs-lookup"><span data-stu-id="d4a61-132">You can also configure a single property to be an alternate key:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/AlternateKeySingle.cs?name=AlternateKeySingle&highlight=4)]

<span data-ttu-id="d4a61-133">你还可以将多个属性配置为替代密钥， (称为复合备用键) ：</span><span class="sxs-lookup"><span data-stu-id="d4a61-133">You can also configure multiple properties to be an alternate key (known as a composite alternate key):</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/AlternateKeyComposite.cs?name=AlternateKeyComposite&highlight=4)]

<span data-ttu-id="d4a61-134">最后，按照约定，为替代键引入的索引和约束将被命名为 `AK_<type name>_<property name>` (，以便复合备用键 `<property name>` 成为) 的属性名称列表。</span><span class="sxs-lookup"><span data-stu-id="d4a61-134">Finally, by convention, the index and constraint that are introduced for an alternate key will be named `AK_<type name>_<property name>` (for composite alternate keys `<property name>` becomes an underscore separated list of property names).</span></span> <span data-ttu-id="d4a61-135">您可以配置备用密钥的 index 和 unique 约束的名称：</span><span class="sxs-lookup"><span data-stu-id="d4a61-135">You can configure the name of the alternate key's index and unique constraint:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/AlternateKeyName.cs?name=AlternateKeyName&highlight=5)]
