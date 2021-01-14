---
title: 保存相关数据 - EF Core
description: 关于在 Entity Framework Core 中保存相关实体图形和管理关系的信息
author: ajcvickers
ms.date: 10/27/2016
uid: core/saving/related-data
ms.openlocfilehash: f1cc752587e6a3eb58d070f5f573450b51986f70
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129091"
---
# <a name="saving-related-data"></a>保存相关数据

除了独立实体以外，还可以使用模型中定义的关系。

> [!TIP]
> 可在 GitHub 上查看此文章的[示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Saving/RelatedData/)。

## <a name="adding-a-graph-of-new-entities"></a>添加新实体的关系图

如果创建多个新的相关实体，则将其中一个添加到上下文时也会添加其他实体。

在下面的示例中，博客和三个相关文章会全部插入到数据库中。 找到并添加这些文章，因为它们可以通过 `Blog.Posts` 导航属性访问。

[!code-csharp[Main](../../../samples/core/Saving/RelatedData/Sample.cs#AddingGraphOfEntities)]

> [!TIP]
> 使用 EntityEntry.State 属性仅设置单个实体的状态。 例如，`context.Entry(blog).State = EntityState.Modified`。

## <a name="adding-a-related-entity"></a>添加相关实体

如果从已由上下文跟踪的实体的导航属性中引用新实体，则将发现该实体并将其插入到数据库中。

在下面的示例中，插入 `post` 实体，因为该实体会添加到已从数据库中提取的 `blog` 实体的 `Posts` 属性。

[!code-csharp[Main](../../../samples/core/Saving/RelatedData/Sample.cs#AddingRelatedEntity)]

## <a name="changing-relationships"></a>更改关系

如果更改实体的导航属性，则将对数据库中的外键列进行相应的更改。

在下面的示例中，`post` 实体更新为属于新的 `blog` 实体，因为其 `Blog` 导航属性设置为指向 `blog`。 请注意，`blog` 也会插入到数据库中，因为它是已由上下文 (`post`) 跟踪的实体的导航属性引用的新实体。

[!code-csharp[Main](../../../samples/core/Saving/RelatedData/Sample.cs#ChangingRelationships)]

## <a name="removing-relationships"></a>删除关系

可以通过将引用导航设置为 `null` 或从集合导航中删除相关实体来删除关系。

根据关系中配置的级联删除行为，删除关系可能会对依赖实体产生副作用。

默认情况下，对于必选关系，将配置级联删除行为，并将从数据库中删除子实体/依赖实体。 对于可选关系，默认情况下不会配置级联删除，但会将外键属性设置为 null。

要了解有关如何配置关系必要性的信息，请参阅[必选关系和可选关系](xref:core/modeling/relationships#required-and-optional-relationships)。

有关级联删除行为的工作原理、如何显式配置这些行为以及如何按照约定选择这些行为的详细信息，请参阅[级联删除](xref:core/saving/cascade-delete)。

在下面的示例中，将在 `Blog` 和 `Post` 之间的关系上配置级联删除，以便从数据库中删除 `post` 实体。

[!code-csharp[Main](../../../samples/core/Saving/RelatedData/Sample.cs#RemovingRelationships)]
