---
title: 继承-EF Core
description: 如何使用 Entity Framework Core 配置实体类型继承
author: AndriySvyryd
ms.date: 10/01/2020
uid: core/modeling/inheritance
ms.openlocfilehash: 33429bbc4a9941ff8ea98a8f99cc652c8ea26455
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003622"
---
# <a name="inheritance"></a>继承

EF 可以将 .NET 类型层次结构映射到数据库。 这使你可以像平常一样使用基类型和派生类型在代码中编写 .NET 实体，并让 EF 无缝创建适当的数据库架构、发出查询等。类型层次结构的映射方式的实际详细信息与提供程序相关;本页介绍关系数据库上下文中的继承支持。

## <a name="entity-type-hierarchy-mapping"></a>实体类型层次结构映射

按照约定，EF 不会自动扫描基类型或派生类型;这意味着，如果要映射层次结构中的 CLR 类型，则必须在模型上显式指定该类型。 例如，仅指定层次结构的基类型将不会导致 EF Core 隐式包含其所有子类型。

下面的示例为及其子类公开了 DbSet `Blog` `RssBlog` 。 如果 `Blog` 有任何其他子类，则不会将其包含在模型中。

[!code-csharp[Main](../../../samples/core/Modeling/Conventions/InheritanceDbSets.cs?name=InheritanceDbSets&highlight=3-4)]

> [!NOTE]
> 使用 TPH 映射时，数据库列会根据需要自动进行为 null。 例如， `RssUrl` 列可为 null，因为正则 `Blog` 实例不具有该属性。

如果你不希望 `DbSet` 为层次结构中的一个或多个实体公开，则还可以使用熟知的 API 来确保它们包含在模型中。

> [!TIP]
> 如果不依赖约定，可以使用显式指定基类型 `HasBaseType` 。 你还可以使用 `.HasBaseType((Type)null)` 从层次结构中删除实体类型。

## <a name="table-per-hierarchy-and-discriminator-configuration"></a>每个层次结构一个表和鉴别器配置

默认情况下，EF 使用 *每个层次结构一个表* (TPH) 模式映射继承。 TPH 使用单个表来存储层次结构中所有类型的数据，而鉴别器列用于标识每行所表示的类型。

上述模型映射到下面的数据库架构 (请注意隐式创建的 `Discriminator` 列，该列标识 `Blog` 每个行中存储) 的类型。

![使用 "每个层次结构一个表" 模式查询博客实体层次结构的结果的屏幕截图](_static/inheritance-tph-data.png)

您可以配置鉴别器列的名称和类型以及用于标识层次结构中的每种类型的值：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DiscriminatorConfiguration.cs?name=DiscriminatorConfiguration&highlight=4-6)]

在上述示例中，EF 在层次结构的基实体上将鉴别器隐式添加为 [影子属性](xref:core/modeling/shadow-properties) 。 此属性可以配置为类似于任何其他属性：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DiscriminatorPropertyConfiguration.cs?name=DiscriminatorPropertyConfiguration&highlight=4-5)]

最后，鉴别器还可以映射到实体中的常规 .NET 属性：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/NonShadowDiscriminator.cs?name=NonShadowDiscriminator&highlight=4)]

查询使用 TPH 模式的派生实体时 EF Core 会在查询中的鉴别器列上添加谓词。 此筛选器可确保不会为不在结果中的基类型或同级类型获取任何其他行。 对于基实体类型，将跳过此筛选器谓词，因为查询基实体将获取层次结构中的所有实体的结果。 当从查询中具体化结果时，如果跨一个不映射到模型中任何实体类型的鉴别器值，我们会引发异常，因为我们不知道如何具体化结果。 仅当数据库包含的行的鉴别器值未在 EF 模型中映射时，才会出现此错误。 如果有这样的数据，则可以将 EF Core 模型中的鉴别器映射标记为 "不完整"，以指示应始终添加筛选器谓词以查询层次结构中的任何类型。 `IsComplete(false)` 鉴别器配置上的调用将映射标记为不完整。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DiscriminatorMappingIncomplete.cs?name=DiscriminatorMappingIncomplete&highlight=5)]

### <a name="shared-columns"></a>共享列

默认情况下，当层次结构中的两个同级实体类型具有相同的属性时，它们将映射到两个单独的列。 但是，如果它们的类型相同，则可以映射到相同的数据库列：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/SharedTPHColumns.cs?name=SharedTPHColumns&highlight=9,13)]

## <a name="table-per-type-configuration"></a>每种类型一个表配置

> [!NOTE]
> EF Core 5.0 中引入了每种类型一个表 (TPT) 功能。 EF6 支持每个具体的表类型 (TPC) ，但 EF Core 尚不支持。

在 TPT 映射模式下，所有类型都映射到各个表。 仅属于某个基类型或派生类型的属性存储在映射到该类型的一个表中。 映射到派生类型的表还存储将派生表与基表联接的外键。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/TPTConfiguration.cs?name=TPTConfiguration)]

EF 将为上述模型创建下列数据库架构。

```sql
CREATE TABLE [Blogs] (
    [BlogId] int NOT NULL IDENTITY,
    [Url] nvarchar(max) NULL,
    CONSTRAINT [PK_Blogs] PRIMARY KEY ([BlogId])
);

CREATE TABLE [RssBlogs] (
    [BlogId] int NOT NULL,
    [RssUrl] nvarchar(max) NULL,
    CONSTRAINT [PK_RssBlogs] PRIMARY KEY ([BlogId]),
    CONSTRAINT [FK_RssBlogs_Blogs_BlogId] FOREIGN KEY ([BlogId]) REFERENCES [Blogs] ([BlogId]) ON DELETE NO ACTION
);
```

> [!NOTE]
> 如果重命名了 primary key 约束，则新名称将应用于映射到层次结构的所有表，以后 EF 版本将允许在解决 [问题 19970](https://github.com/dotnet/efcore/issues/19970) 时为特定表重命名约束。

如果要使用批量配置，可以通过调用来检索特定表的列名称 <xref:Microsoft.EntityFrameworkCore.RelationalPropertyExtensions.GetColumnName%2A> 。

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/TPTConfiguration.cs?name=Metadata&highlight=10)]
