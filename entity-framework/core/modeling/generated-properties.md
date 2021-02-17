---
title: 生成的值-EF Core
description: 如何在使用时配置属性的值生成 Entity Framework Core
author: AndriySvyryd
ms.date: 1/10/2021
uid: core/modeling/generated-properties
ms.openlocfilehash: a9e43f3b755bf028bc76581135988e831a42d0d1
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543336"
---
# <a name="generated-values"></a>生成的值

数据库列的值可以通过各种方式生成：主键列经常是自动递增的整数，其他列具有默认值或计算的值等。此页详细介绍了 EF Core 的配置值生成的各种模式。

## <a name="default-values"></a>默认值

在关系数据库中，可以使用默认值来配置列;如果插入的行没有该列的值，将使用默认值。

可以在属性上配置默认值：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DefaultValue.cs?name=DefaultValue&highlight=5)]

还可以指定用于计算默认值的 SQL 片段：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DefaultValueSql.cs?name=DefaultValueSql&highlight=5)]

## <a name="computed-columns"></a>计算列

在大多数关系数据库上，列可以配置为在数据库中计算其值，通常具有引用其他列的表达式：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ComputedColumn.cs?name=DefaultComputedColumn&highlight=3)]

上面创建了一个 *虚拟* 计算列，每次从数据库中提取值时都会计算其值。 您还可以指定将计算列 *存储* (有时称为 *持久化*) ，这意味着在每次更新行时计算，并将磁盘与常规列一起存储：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ComputedColumn.cs?name=StoredComputedColumn&highlight=3)]

> [!NOTE]
> EF Core 5.0 中增加了对创建存储计算列的支持。

## <a name="primary-keys"></a>主键

按照约定，如果应用程序不提供值，则将 "short"、"int"、"long" 或 "Guid" 类型的非复合主键设置为为插入的实体生成值。 您的数据库提供程序通常会负责必要的配置;例如，SQL Server 中的数字主键将自动设置为标识列。

有关详细信息，请 [参阅有关密钥的文档](xref:core/modeling/keys)。

## <a name="explicitly-configuring-value-generation"></a>显式配置值生成

上述 EF Core 会自动设置主键的值生成，但对于非键属性，我们可能要执行相同操作。 可以将任何属性配置为为插入的实体生成其值，如下所示：

### <a name="data-annotations"></a>[数据批注](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/ValueGeneratedOnAdd.cs?name=ValueGeneratedOnAdd&highlight=6)]

### <a name="fluent-api"></a>[Fluent API](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedOnAdd.cs?name=ValueGeneratedOnAdd&highlight=5)]

***

同样，可以将属性配置为在添加或更新时生成其值：

### <a name="data-annotations"></a>[数据批注](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/ValueGeneratedOnAddOrUpdate.cs?name=ValueGeneratedOnAddOrUpdate&highlight=6)]

### <a name="fluent-api"></a>[Fluent API](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedOnAddOrUpdate.cs?name=ValueGeneratedOnAddOrUpdate&highlight=5)]

***

> [!WARNING]
> 与默认值或计算列不同，我们不指定 *如何* 生成值;这取决于所使用的数据库提供程序。 数据库提供程序可以为某些属性类型自动设置值生成，但其他提供程序可能要求您手动设置如何生成值。
>
> 例如，在 SQL Server 上，当 GUID 属性配置为 "添加时生成的值" 时，提供程序将使用算法生成最佳顺序 GUID 值，自动执行值生成客户端。 但是， `ValueGeneratedOnAdd()` 对 datetime 属性指定将不起任何作用 ([请参阅下面的部分，了解是否) 生成日期值](#datetime-value-generation) 。
>
> 同样，配置为在添加或更新时生成并标记为并发令牌的 byte [] 属性将设置为 rowversion 数据类型，以便在数据库中自动生成值。 但再次指定 `ValueGeneratedOnAddOrUpdate()` 将不起作用。
>
> [!NOTE]
> 根据所使用的数据库提供程序，值可能是由 EF 或数据库中的客户端生成的。 如果值是由数据库生成的，则在将实体添加到上下文时，EF 可能会分配临时值;在期间，此临时值将替换为数据库生成的值 `SaveChanges()` 。 有关详细信息， [请参阅临时值上的文档](xref:core/change-tracking/explicit-tracking#temporary-values)。

## <a name="datetime-value-generation"></a>日期/时间值生成

常见的请求是包含一个数据库列，其中包含第一次插入列时的日期/时间 (添加) 上生成的值，或上次更新的时间 (添加或更新) 生成的值。 由于有各种策略来执行此操作，因此 EF Core 提供商通常不会为日期/时间列自动设置值生成-您必须自行配置。

### <a name="creation-timestamp"></a>创建时间戳

将日期/时间列配置为具有行的创建时间戳通常是使用适当的 SQL 函数配置默认值的问题。 例如，在 SQL Server 可以使用以下内容：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DefaultValueSql.cs?name=DefaultValueSql&highlight=5)]

请确保选择适当的函数，因为可能存在多个 (例如 `GETDATE()` `GETUTCDATE()`) 。

### <a name="update-timestamp"></a>更新时间戳

尽管存储计算列看起来像是用于管理上次更新时间戳的良好解决方案，但数据库通常不允许 `GETDATE()` 在计算列中指定函数，例如。 作为替代方法，您可以设置一个数据库触发器来实现同样的效果：

```sql
CREATE TRIGGER [dbo].[Blogs_UPDATE] ON [dbo].[Blogs]
    AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    IF ((SELECT TRIGGER_NESTLEVEL()) > 1) RETURN;

    DECLARE @Id INT

    SELECT @Id = INSERTED.BlogId
    FROM INSERTED

    UPDATE dbo.Blogs
    SET LastUpdated = GETDATE()
    WHERE BlogId = @Id
END
```

有关创建触发器的信息， [请参阅有关在迁移中使用原始 SQL 的文档](xref:core/managing-schemas/migrations/managing#adding-raw-sql)。

## <a name="overriding-value-generation"></a>重写值生成

尽管为值生成配置了属性，但在许多情况下，你仍可以为其显式指定一个值。 这是否会实际工作取决于已配置的特定值生成机制;尽管可以指定显式值而不是使用列的默认值，但不能对计算列执行相同的操作。

若要使用显式值覆盖值生成，只需将属性设置为任何不是该属性类型的 CLR 默认值的值， (、for、 `null` `string` `0` `int` `Guid.Empty` `Guid` 等等 ) 。

> [!NOTE]
> 默认情况下，尝试将显式值插入 SQL Server 标识失败; [有关解决方法，请参阅以下文档](xref:core/providers/sql-server/value-generation#inserting-explicit-values-into-identity-columns)。

若要为已配置为在添加或更新时生成的值的属性提供显式值，还必须按如下所示配置属性：

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedOnAddOrUpdateWithPropertySaveBehavior.cs?name=ValueGeneratedOnAddOrUpdateWithPropertySaveBehavior&highlight=5)]

## <a name="no-value-generation"></a>无值生成

除了前面所述的特定方案以外，属性通常不会配置任何值;这意味着，应用程序始终提供要保存到数据库中的值。 必须先将此值分配给新实体，然后才能将其添加到上下文中。

但是，在某些情况下，你可能想要禁用已经按约定设置的值生成。 例如，int 类型的主键通常隐式配置为基于值生成的 (，例如 SQL Server) 上的标识列。 可以通过以下方式禁用此操作：

### <a name="data-annotations"></a>[数据批注](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/ValueGeneratedNever.cs?name=ValueGeneratedNever&highlight=3)]

### <a name="fluent-api"></a>[Fluent API](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedNever.cs?name=ValueGeneratedNever&highlight=5)]

***
