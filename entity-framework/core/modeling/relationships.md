---
title: 关系-EF Core
description: 如何在使用 Entity Framework Core 时配置实体类型之间的关系
author: AndriySvyryd
ms.date: 10/01/2020
uid: core/modeling/relationships
ms.openlocfilehash: 716c034bd73d831996b727da18c2c1f83dd55290
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003258"
---
# <a name="relationships"></a>关系

关系定义两个实体之间的关系。 在关系数据库中，这由外键约束表示。

> [!NOTE]  
> 本文中的大多数示例都使用一对多关系来演示概念。 有关一对一关系和多对多关系的示例，请参阅文章末尾的 [其他关系模式](#other-relationship-patterns) 部分。

## <a name="definition-of-terms"></a>术语定义

有许多术语用于描述关系

* **相关实体：** 这是包含外键属性的实体。 有时称为关系的 "子级"。

* **主体实体：** 这是包含主/备用键属性的实体。 有时称为关系的 "父项"。

* **主体密钥：** 唯一标识主体实体的属性。 这可能是主键或备用密钥。

* **外键：** 用于存储相关实体的主体键值的依赖实体中的属性。

* **导航属性：** 在主体和/或从属实体上定义的属性，该属性引用相关实体。

  * **集合导航属性：** 一个导航属性，其中包含对多个相关实体的引用。

  * **引用导航属性：** 保存对单个相关实体的引用的导航属性。

  * **反向导航属性：** 讨论特定导航属性时，此术语是指关系另一端的导航属性。
  
* **自引用关系：** 依赖关系和主体实体类型相同的关系。

下面的代码显示与之间的一对多关系 `Blog``Post`

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/Relationships/Full.cs#Full)]

* `Post` 是依赖实体

* `Blog` 是主体实体

* `Blog.BlogId` 是主体键 (在本例中，它是主密钥而不是备用密钥) 

* `Post.BlogId` 为外键

* `Post.Blog` 是一个引用导航属性

* `Blog.Posts` 是集合导航属性

* `Post.Blog` (的反向导航属性 `Blog.Posts` ，反之亦然) 

## <a name="conventions"></a>约定

默认情况下，当在某个类型上发现导航属性时，将创建一个关系。 如果当前数据库提供程序无法将其指向的类型映射为标量类型，则该属性被视为导航属性。

> [!NOTE]  
> 按约定发现的关系将始终以主体实体的主键为目标。 若要以备用密钥为目标，则必须使用熟知的 API 执行其他配置。

### <a name="fully-defined-relationships"></a>完全定义的关系

关系最常见的模式是在关系两端定义导航属性，在依赖实体类中定义外键属性。

* 如果在两个类型之间找到一对导航属性，则这些属性将配置为同一关系的反向导航属性。

* 如果依赖实体包含名称与其中一种模式相匹配的属性，则该属性将被配置为外键：
  * `<navigation property name><principal key property name>`
  * `<navigation property name>Id`
  * `<principal entity name><principal key property name>`
  * `<principal entity name>Id`

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/Relationships/Full.cs?name=Full&highlight=6,15-16)]

在此示例中，突出显示的属性将用于配置关系。

> [!NOTE]
> 如果属性为主键，或者为与主体键不兼容的类型，则不会将其配置为外键。

> [!NOTE]
> 在 EF Core 3.0 之前，名为与主体键属性完全相同的属性 [也与外键匹配](https://github.com/dotnet/efcore/issues/13274)

### <a name="no-foreign-key-property"></a>无外键属性

尽管建议在依赖实体类中定义外键属性，但这并不是必需的。 如果未找到外键属性，则会使用名称引入 [阴影外键属性](xref:core/modeling/shadow-properties) ， `<navigation property name><principal key property name>` `<principal entity name><principal key property name>` 如果依赖类型上没有导航，则为。

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/Relationships/NoForeignKey.cs?name=NoForeignKey&highlight=6,15)]

在此示例中，阴影外键是 `BlogId` 因为预先计算导航名称将是冗余的。

> [!NOTE]
> 如果已存在具有相同名称的属性，则会以数字作为后缀的阴影属性名称。

### <a name="single-navigation-property"></a>单个导航属性

只包含一个导航属性 (不会反向导航，并且没有外键属性) 足以具有约定定义的关系。 还可以有一个导航属性和一个外键属性。

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/Relationships/OneNavigation.cs?name=OneNavigation&highlight=6)]

### <a name="limitations"></a>限制

如果在两个类型之间定义了多个导航属性 (也就是说，多个只是一对导航) 导航属性表示的关系是不明确的。 你将需要手动对其进行配置以解决歧义。

### <a name="cascade-delete"></a>级联删除

按照约定，级联删除将对所需的关系和 *ClientSetNull* 设置为 *cascade* ，以实现可选关系。 *Cascade* 表示也会删除依赖实体。 *ClientSetNull* 表示未加载到内存中的依赖实体将保持不变，必须手动删除，或将其更新为指向有效的主体实体。 对于加载到内存中的实体，EF Core 将尝试将外键属性设置为 null。

请参阅 required [和 optional](#required-and-optional-relationships) 关系部分，了解必需和可选关系之间的差异。

有关不同的删除行为和约定使用的默认值的详细信息，请参阅 [级联删除](xref:core/saving/cascade-delete) 。

## <a name="manual-configuration"></a>手动配置

### <a name="fluent-api"></a>[Fluent API](#tab/fluent-api)

若要在熟知的 API 中配置关系，请首先标识构成关系的导航属性。 `HasOne` 或 `HasMany` 标识要开始配置的实体类型上的导航属性。 然后，将调用链接到 `WithOne` 或 `WithMany` 以标识反向导航。 `HasOne`/`WithOne`用于引用导航属性，用于 `HasMany` / `WithMany` 集合导航属性。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/NoForeignKey.cs?name=NoForeignKey&highlight=8-10)]

### <a name="data-annotations"></a>[数据注释](#tab/data-annotations)

您可以使用数据批注来配置依赖项和主体实体上的导航属性如何配对。 这通常在两个实体类型之间存在多个导航属性对时执行。

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/Relationships/InverseProperty.cs?name=InverseProperty&highlight=20,23)]

> [!NOTE]
> 只能在依赖实体上的属性上使用 [Required] 来影响关系的 requiredness。 [必需] 在主体实体的导航中通常会忽略，但这可能会导致实体成为依赖实体。

> [!NOTE]
> 数据批注 `[ForeignKey]` 和 `[InverseProperty]` 在命名空间中可用 `System.ComponentModel.DataAnnotations.Schema` 。 `[Required]` 在 `System.ComponentModel.DataAnnotations` 命名空间中可用。

---

### <a name="single-navigation-property"></a>单个导航属性

如果只有一个导航属性，则和的无参数重载 `WithOne` `WithMany` 。 这表示在概念上，关系的另一端有一个引用或集合，但实体类中不包含导航属性。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/OneNavigation.cs?name=OneNavigation&highlight=8-10)]

---

### <a name="configuring-navigation-properties"></a>配置导航属性

> [!NOTE]
> EF Core 5.0 中引入了此功能。

创建导航属性后，你可能需要对其进行进一步配置。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/NavigationConfiguration.cs?name=NavigationConfiguration&highlight=7-9)]

> [!NOTE]
> 此调用不能用于创建导航属性。 它仅用于配置导航属性，该属性以前是通过定义关系或从约定创建的。

### <a name="foreign-key"></a>外键

#### <a name="fluent-api-simple-key"></a>[熟知 API (简单密钥) ](#tab/fluent-api-simple-key)

您可以使用熟知的 API 来配置应用作给定关系的外键属性的属性：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/ForeignKey.cs?name=ForeignKey&highlight=11)]

#### <a name="fluent-api-composite-key"></a>[熟知 API (组合键) ](#tab/fluent-api-composite-key)

您可以使用熟知的 API 来配置哪些属性应用作给定关系的复合外键属性：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/CompositeForeignKey.cs?name=CompositeForeignKey&highlight=13)]

#### <a name="data-annotations-simple-key"></a>[数据批注 (简单键) ](#tab/data-annotations-simple-key)

您可以使用数据批注来配置应用作给定关系的外键属性的属性。 通常，当不按约定发现外键属性时，会执行此操作：

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/Relationships/ForeignKey.cs?name=ForeignKey&highlight=17)]

> [!TIP]  
> `[ForeignKey]`批注可放置在关系中的任一导航属性上。 它不需要在依赖实体类中定位导航属性。

> [!NOTE]
> `[ForeignKey]`在导航属性上使用指定的属性不需要存在于依赖类型上。 在这种情况下，将使用指定的名称创建阴影外键。

---

#### <a name="shadow-foreign-key"></a>影子外键

您可以使用的字符串重载将 `HasForeignKey(...)` 影子属性配置为外键 (参阅 [阴影属性](xref:core/modeling/shadow-properties) 以了解详细信息) 。 建议先将影子属性显式添加到模型中，然后再将其用作外键 (如下) 所示。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/ShadowForeignKey.cs?name=ShadowForeignKey&highlight=10,16)]

#### <a name="foreign-key-constraint-name"></a>Foreign key 约束名称

按照约定，以关系数据库为目标时，外键约束命名为 FK \_ \<dependent type name> \_ \<principal type name> \_ \<foreign key property name> 。 对于复合外键， \<foreign key property name> 将成为外键属性名称的下划线分隔列表。

你还可以配置约束名称，如下所示：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/ConstraintName.cs?name=ConstraintName&highlight=6-7)]

### <a name="without-navigation-property"></a>无导航属性

不一定需要提供导航属性。 您可以直接在关系的一端提供外键。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/NoNavigation.cs?name=NoNavigation&highlight=8-11)]

### <a name="principal-key"></a>主体密钥

如果你希望外键引用主键之外的属性，则可以使用熟知的 API 来配置关系的主体键属性。 配置为主体密钥的属性将自动设置为 [备用密钥](xref:core/modeling/keys#alternate-keys)。

#### <a name="simple-key"></a>[简单键](#tab/simple-key)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/PrincipalKey.cs?name=PrincipalKey&highlight=11)]

#### <a name="composite-key"></a>[组合键](#tab/composite-key)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/CompositePrincipalKey.cs?name=CompositePrincipalKey&highlight=11)]

> [!WARNING]  
> 指定主体键属性的顺序必须与为外键指定这些属性的顺序一致。

---

### <a name="required-and-optional-relationships"></a>必需和可选的关系

您可以使用熟知的 API 来配置关系是必需的还是可选的。 最终，这会控制外键属性是必需的还是可选的。 当使用阴影状态外键时，这非常有用。 如果实体类中具有外键属性，则关系的 requiredness 取决于外键属性是必需还是可选 (查看) 的详细信息 [所需的属性和可选属性](xref:core/modeling/entity-properties#required-and-optional-properties) 。

外键属性位于依赖实体类型上，因此，如果这些属性配置为 "必需"，则意味着每个依赖实体都需要具有相应的主体实体。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/Required.cs?name=Required&highlight=6)]

> [!NOTE]
> `IsRequired(false)`如果未配置，则调用还会使外键属性为可选。

### <a name="cascade-delete"></a>级联删除

您可以使用熟知的 API 显式配置给定关系的级联删除行为。

有关每个选项的详细讨论，请参阅 [级联删除](xref:core/saving/cascade-delete) 。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/CascadeDelete.cs?name=CascadeDelete&highlight=6)]

## <a name="other-relationship-patterns"></a>其他关系模式

### <a name="one-to-one"></a>一对一

一对多关系在两侧都有一个引用导航属性。 它们遵循与一对多关系相同的约定，但在外键属性上引入了唯一索引，以确保只有一个依赖项与每个主体相关。

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/Relationships/OneToOne.cs?name=OneToOne&highlight=6,15-16)]

> [!NOTE]  
> EF 会根据其检测外键属性的能力，选择其中一个实体作为依赖项。 如果选择了错误的实体作为依赖项，则可以使用熟知的 API 来更正此问题。

使用 "流畅" API 配置关系时，请使用 `HasOne` 和 `WithOne` 方法。

配置外键时，需要指定依赖实体类型-请注意以下列表中提供的泛型参数 `HasForeignKey` 。 在一对多关系中，可以清楚地表明具有引用导航的实体是依赖项，并且具有集合的实体是主体。 但这并不是一对一的关系，因此需要显式定义它。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/OneToOne.cs?name=OneToOne&highlight=11)]

默认情况下，从属端被视为可选的，但可以根据需要进行配置。 但是，EF 不会验证是否提供了依赖实体，因此，只有在数据库映射允许强制执行此配置时，此配置才会产生差别。 这种情况的常见方案是默认情况下使用表拆分的引用拥有的类型。

[!code-csharp[Main](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=Required&highlight=11-12)]

对于此配置，与对应的列 `ShippingAddress` 在数据库中将标记为不可为 null。

> [!NOTE]
> 如果使用 [不可为 null 的引用类型](/dotnet/csharp/nullable-references) `IsRequired` ，则无需调用。

> [!NOTE]
> EF Core 5.0 中引入了配置依赖是否必需的功能。

### <a name="many-to-many"></a>多对多

多对多关系要求两侧都有集合导航属性。 它们将被类似于其他类型的关系的惯例来发现。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/ManyToManyShared.cs?name=ManyToManyShared)]

此关系在数据库中的实现方式是通过联接表，其中包含和的外键 `Post` `Tag` 。 例如，这是 EF 将在上述模型的关系数据库中创建的内容。

```sql
CREATE TABLE [Posts] (
    [PostId] int NOT NULL IDENTITY,
    [Title] nvarchar(max) NULL,
    [Content] nvarchar(max) NULL,
    CONSTRAINT [PK_Posts] PRIMARY KEY ([PostId])
);

CREATE TABLE [Tags] (
    [TagId] nvarchar(450) NOT NULL,
    CONSTRAINT [PK_Tags] PRIMARY KEY ([TagId])
);

CREATE TABLE [PostTag] (
    [PostsId] int NOT NULL,
    [TagsId] nvarchar(450) NOT NULL,
    CONSTRAINT [PK_PostTag] PRIMARY KEY ([PostsId], [TagsId]),
    CONSTRAINT [FK_PostTag_Posts_PostsId] FOREIGN KEY ([PostsId]) REFERENCES [Posts] ([PostId]) ON DELETE CASCADE,
    CONSTRAINT [FK_PostTag_Tags_TagsId] FOREIGN KEY ([TagsId]) REFERENCES [Tags] ([TagId]) ON DELETE CASCADE
);
```

在内部，EF 会创建一个实体类型来表示将被称为联接实体类型的联接表。 没有可用于此的特定 CLR 类型，因此 `Dictionary<string, object>` 使用。 模型中可能存在多个多对多关系，因此，在此情况下，必须为联接实体类型指定唯一名称 `PostTag` 。 此功能可用于共享类型的实体类型。

"多对多" 导航称为 "跳过导航"，因为它们实际上会跳过联接实体类型。 如果你正在运用大容量配置，则可以从获取所有 skip 导航 `GetSkipNavigations` 。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/ManyToManyShared.cs?name=Metadata)]

通常会将配置应用到联接实体类型。 可以通过完成此操作 `UsingEntity` 。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/ManyToManyShared.cs?name=SharedConfiguration)]

可以使用匿名类型为联接实体类型提供[模型种子数据](xref:core/modeling/data-seeding)。 您可以检查模型调试视图，以确定由约定创建的属性名称。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/ManyToManyShared.cs?name=Seeding)]

其他数据可以存储在联接实体类型中，但对于这种情况，最好是创建订购 CLR 类型。 在配置与自定义联接实体类型的关系时，需要显式指定两个外键。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/ManyToManyPayload.cs?name=ManyToManyPayload)]

> [!NOTE]
> EF Core 5.0 中引入了配置多对多关系的功能，但对于以前的版本，请使用以下方法。

您还可以通过只添加联接实体类型并映射两个单独的一对多关系来表示多对多关系。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Relationships/ManyToMany.cs?name=ManyToMany&highlight=16-19,21-24)]

> [!NOTE]
> 尚未添加对数据库中多对多关系搭建基架的支持。 请参阅[跟踪问题](https://github.com/dotnet/efcore/issues/22475)。

## <a name="additional-resources"></a>其他资源

* [EF Core 社区 Standup 会话](https://www.youtube.com/watch?v=W1sxepfIMRM&list=PLdo4fOcmZ0oX-DBuRG4u58ZTAJgBAeQ-t&index=32)，深入探讨多对多和基础结构。
