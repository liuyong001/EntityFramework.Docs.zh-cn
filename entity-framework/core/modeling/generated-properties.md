---
title: 生成的值-EF Core
description: 如何在使用时配置属性的值生成 Entity Framework Core
author: AndriySvyryd
ms.date: 1/10/2021
uid: core/modeling/generated-properties
ms.openlocfilehash: 76fa4454c88a5ef7afb9864c2a4b1063ac75e37e
ms.sourcegitcommit: 7700840119b1639275f3b64836e7abb59103f2e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2021
ms.locfileid: "98983542"
---
# <a name="generated-values"></a><span data-ttu-id="0d770-103">生成的值</span><span class="sxs-lookup"><span data-stu-id="0d770-103">Generated Values</span></span>

<span data-ttu-id="0d770-104">数据库列的值可以通过各种方式生成：主键列经常是自动递增的整数，其他列具有默认值或计算的值等。此页详细介绍了 EF Core 的配置值生成的各种模式。</span><span class="sxs-lookup"><span data-stu-id="0d770-104">Database columns can have their values generated in various ways: primary key columns are frequently auto-incrementing integers, other columns have default or computed values, etc. This page details various patterns for configuration value generation with EF Core.</span></span>

## <a name="default-values"></a><span data-ttu-id="0d770-105">默认值</span><span class="sxs-lookup"><span data-stu-id="0d770-105">Default values</span></span>

<span data-ttu-id="0d770-106">在关系数据库中，可以使用默认值来配置列;如果插入的行没有该列的值，将使用默认值。</span><span class="sxs-lookup"><span data-stu-id="0d770-106">On relational databases, a column can be configured with a default value; if a row is inserted without a value for that column, the default value will be used.</span></span>

<span data-ttu-id="0d770-107">可以在属性上配置默认值：</span><span class="sxs-lookup"><span data-stu-id="0d770-107">You can configure a default value on a property:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DefaultValue.cs?name=DefaultValue&highlight=5)]

<span data-ttu-id="0d770-108">还可以指定用于计算默认值的 SQL 片段：</span><span class="sxs-lookup"><span data-stu-id="0d770-108">You can also specify a SQL fragment that is used to calculate the default value:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DefaultValueSql.cs?name=DefaultValueSql&highlight=5)]

## <a name="computed-columns"></a><span data-ttu-id="0d770-109">计算列</span><span class="sxs-lookup"><span data-stu-id="0d770-109">Computed columns</span></span>

<span data-ttu-id="0d770-110">在大多数关系数据库上，列可以配置为在数据库中计算其值，通常具有引用其他列的表达式：</span><span class="sxs-lookup"><span data-stu-id="0d770-110">On most relational databases, a column can be configured to have its value computed in the database, typically with an expression referring to other columns:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ComputedColumn.cs?name=DefaultComputedColumn&highlight=3)]

<span data-ttu-id="0d770-111">上面创建了一个 *虚拟* 计算列，每次从数据库中提取值时都会计算其值。</span><span class="sxs-lookup"><span data-stu-id="0d770-111">The above creates a *virtual* computed column, whose value is computed every time it is fetched from the database.</span></span> <span data-ttu-id="0d770-112">您还可以指定将计算列 *存储* (有时称为 *持久化*) ，这意味着在每次更新行时计算，并将磁盘与常规列一起存储：</span><span class="sxs-lookup"><span data-stu-id="0d770-112">You may also specify that a computed column be *stored* (sometimes called *persisted*), meaning that it is computed on every update of the row, and is stored on disk alongside regular columns:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ComputedColumn.cs?name=StoredComputedColumn&highlight=3)]

> [!NOTE]
> <span data-ttu-id="0d770-113">EF Core 5.0 中增加了对创建存储计算列的支持。</span><span class="sxs-lookup"><span data-stu-id="0d770-113">Support for creating stored computed columns was added in EF Core 5.0.</span></span>

## <a name="primary-keys"></a><span data-ttu-id="0d770-114">主键</span><span class="sxs-lookup"><span data-stu-id="0d770-114">Primary keys</span></span>

<span data-ttu-id="0d770-115">按照约定，如果应用程序不提供值，则将 "short"、"int"、"long" 或 "Guid" 类型的非复合主键设置为为插入的实体生成值。</span><span class="sxs-lookup"><span data-stu-id="0d770-115">By convention, non-composite primary keys of type short, int, long, or Guid are set up to have values generated for inserted entities if a value isn't provided by the application.</span></span> <span data-ttu-id="0d770-116">您的数据库提供程序通常会负责必要的配置;例如，SQL Server 中的数字主键将自动设置为标识列。</span><span class="sxs-lookup"><span data-stu-id="0d770-116">Your database provider typically takes care of the necessary configuration; for example, a numeric primary key in SQL Server is automatically set up to be an IDENTITY column.</span></span>

<span data-ttu-id="0d770-117">有关详细信息，请 [参阅有关密钥的文档](xref:core/modeling/keys)。</span><span class="sxs-lookup"><span data-stu-id="0d770-117">For more information, [see the documentation about keys](xref:core/modeling/keys).</span></span>

## <a name="explicitly-configuring-value-generation"></a><span data-ttu-id="0d770-118">显式配置值生成</span><span class="sxs-lookup"><span data-stu-id="0d770-118">Explicitly configuring value generation</span></span>

<span data-ttu-id="0d770-119">上述 EF Core 会自动设置主键的值生成，但对于非键属性，我们可能要执行相同操作。</span><span class="sxs-lookup"><span data-stu-id="0d770-119">We saw above that EF Core automatically sets up value generation for primary keys - but we may want to do the same for non-key properties.</span></span> <span data-ttu-id="0d770-120">可以将任何属性配置为为插入的实体生成其值，如下所示：</span><span class="sxs-lookup"><span data-stu-id="0d770-120">You can configure any property to have its value generated for inserted entities as follows:</span></span>

### <a name="data-annotations"></a>[<span data-ttu-id="0d770-121">数据批注</span><span class="sxs-lookup"><span data-stu-id="0d770-121">Data Annotations</span></span>](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/ValueGeneratedOnAdd.cs?name=ValueGeneratedOnAdd&highlight=5)]

### <a name="fluent-api"></a>[<span data-ttu-id="0d770-122">Fluent API</span><span class="sxs-lookup"><span data-stu-id="0d770-122">Fluent API</span></span>](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedOnAdd.cs?name=ValueGeneratedOnAdd&highlight=5)]

***

<span data-ttu-id="0d770-123">同样，可以将属性配置为在添加或更新时生成其值：</span><span class="sxs-lookup"><span data-stu-id="0d770-123">Similarly, a property can be configured to have its value generated on add or update:</span></span>

### <a name="data-annotations"></a>[<span data-ttu-id="0d770-124">数据批注</span><span class="sxs-lookup"><span data-stu-id="0d770-124">Data Annotations</span></span>](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/ValueGeneratedOnAddOrUpdate.cs?name=ValueGeneratedOnAddOrUpdate&highlight=5)]

### <a name="fluent-api"></a>[<span data-ttu-id="0d770-125">Fluent API</span><span class="sxs-lookup"><span data-stu-id="0d770-125">Fluent API</span></span>](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedOnAddOrUpdate.cs?name=ValueGeneratedOnAddOrUpdate&highlight=5)]

<span data-ttu-id="0d770-126">\*\*_</span><span class="sxs-lookup"><span data-stu-id="0d770-126">\*\*_</span></span>

> [!WARNING]
> <span data-ttu-id="0d770-127">与默认值或计算列不同的是，我们不会指定要生成的值 _how \*;这取决于所使用的数据库提供程序。</span><span class="sxs-lookup"><span data-stu-id="0d770-127">Unlike with default values or computed columns, we are not specifying _how\* the values are to be generated; that depends on the database provider being used.</span></span> <span data-ttu-id="0d770-128">数据库提供程序可以为某些属性类型自动设置值生成，但其他提供程序可能要求您手动设置如何生成值。</span><span class="sxs-lookup"><span data-stu-id="0d770-128">Database providers may automatically set up value generation for some property types, but others may require you to manually set up how the value is generated.</span></span>
>
> <span data-ttu-id="0d770-129">例如，在 SQL Server 上，当 GUID 属性配置为 "添加时生成的值" 时，提供程序将使用算法生成最佳顺序 GUID 值，自动执行值生成客户端。</span><span class="sxs-lookup"><span data-stu-id="0d770-129">For example, on SQL Server, when a GUID property is configured as value generated on add, the provider automatically performs value generation client-side, using an algorithm to generate optimal sequential GUID values.</span></span> <span data-ttu-id="0d770-130">但是， `ValueGeneratedOnAdd()` 对 datetime 属性指定将不起任何作用 ([请参阅下面的部分，了解是否) 生成日期值](#datetime-value-generation) 。</span><span class="sxs-lookup"><span data-stu-id="0d770-130">However, specifying `ValueGeneratedOnAdd()` on a DateTime property will have no effect ([see the section below for DateTime value generation](#datetime-value-generation)).</span></span>
>
> <span data-ttu-id="0d770-131">同样，配置为在添加或更新时生成并标记为并发令牌的 byte [] 属性将设置为 rowversion 数据类型，以便在数据库中自动生成值。</span><span class="sxs-lookup"><span data-stu-id="0d770-131">Similarly, byte[] properties that are configured as generated on add or update and marked as concurrency tokens are set up with the rowversion data type, so that values are automatically generated in the database.</span></span> <span data-ttu-id="0d770-132">但再次指定 `ValueGeneratedOnAddOrUpdate()` 将不起作用。</span><span class="sxs-lookup"><span data-stu-id="0d770-132">However, specifying `ValueGeneratedOnAddOrUpdate()` will again have no effect.</span></span>
>
> [!NOTE]
> <span data-ttu-id="0d770-133">根据所使用的数据库提供程序，值可能是由 EF 或数据库中的客户端生成的。</span><span class="sxs-lookup"><span data-stu-id="0d770-133">Depending on the database provider being used, values may be generated client side by EF or in the database.</span></span> <span data-ttu-id="0d770-134">如果值是由数据库生成的，则在将实体添加到上下文时，EF 可能会分配临时值;在期间，此临时值将替换为数据库生成的值 `SaveChanges()` 。</span><span class="sxs-lookup"><span data-stu-id="0d770-134">If the value is generated by the database, then EF may assign a temporary value when you add the entity to the context; this temporary value will then be replaced by the database generated value during `SaveChanges()`.</span></span> <span data-ttu-id="0d770-135">有关详细信息， [请参阅临时值上的文档](xref:core/change-tracking/explicit-tracking#temporary-values)。</span><span class="sxs-lookup"><span data-stu-id="0d770-135">For more information, [see the docs on temporary values](xref:core/change-tracking/explicit-tracking#temporary-values).</span></span>

## <a name="datetime-value-generation"></a><span data-ttu-id="0d770-136">日期/时间值生成</span><span class="sxs-lookup"><span data-stu-id="0d770-136">Date/time value generation</span></span>

<span data-ttu-id="0d770-137">常见的请求是包含一个数据库列，其中包含第一次插入列时的日期/时间 (添加) 上生成的值，或上次更新的时间 (添加或更新) 生成的值。</span><span class="sxs-lookup"><span data-stu-id="0d770-137">A common request is to have a database column which contains the date/time for when the column was first inserted (value generated on add), or for when it was last updated (value generated on add or update).</span></span> <span data-ttu-id="0d770-138">由于有各种策略来执行此操作，因此 EF Core 提供商通常不会为日期/时间列自动设置值生成-您必须自行配置。</span><span class="sxs-lookup"><span data-stu-id="0d770-138">As there are various strategies to do this, EF Core providers usually don't set up value generation automatically for date/time columns - you have to configure this yourself.</span></span>

### <a name="creation-timestamp"></a><span data-ttu-id="0d770-139">创建时间戳</span><span class="sxs-lookup"><span data-stu-id="0d770-139">Creation timestamp</span></span>

<span data-ttu-id="0d770-140">将日期/时间列配置为具有行的创建时间戳通常是使用适当的 SQL 函数配置默认值的问题。</span><span class="sxs-lookup"><span data-stu-id="0d770-140">Configuring a date/time column to have the creation timestamp of the row is usually a matter of configuring a default value with the appropriate SQL function.</span></span> <span data-ttu-id="0d770-141">例如，在 SQL Server 可以使用以下内容：</span><span class="sxs-lookup"><span data-stu-id="0d770-141">For example, on SQL Server you may use the following:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/DefaultValueSql.cs?name=DefaultValueSql&highlight=5)]

<span data-ttu-id="0d770-142">请确保选择适当的函数，因为可能存在多个 (例如 `GETDATE()` `GETUTCDATE()`) 。</span><span class="sxs-lookup"><span data-stu-id="0d770-142">Be sure to select the appropriate function, as several may exist (e.g. `GETDATE()` vs. `GETUTCDATE()`).</span></span>

### <a name="update-timestamp"></a><span data-ttu-id="0d770-143">更新时间戳</span><span class="sxs-lookup"><span data-stu-id="0d770-143">Update timestamp</span></span>

<span data-ttu-id="0d770-144">尽管存储计算列看起来像是用于管理上次更新时间戳的良好解决方案，但数据库通常不允许 `GETDATE()` 在计算列中指定函数，例如。</span><span class="sxs-lookup"><span data-stu-id="0d770-144">Although stored computed columns seem like a good solution for managing last-updated timestamps, databases usually don't allow specifying functions such as `GETDATE()` in a computed column.</span></span> <span data-ttu-id="0d770-145">作为替代方法，您可以设置一个数据库触发器来实现同样的效果：</span><span class="sxs-lookup"><span data-stu-id="0d770-145">As an alternative, you can set up a database trigger to achieve the same effect:</span></span>

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

<span data-ttu-id="0d770-146">有关创建触发器的信息， [请参阅有关在迁移中使用原始 SQL 的文档](xref:core/managing-schemas/migrations/managing#adding-raw-sql)。</span><span class="sxs-lookup"><span data-stu-id="0d770-146">For information on creating triggers, [see the documentation on using raw SQL in migrations](xref:core/managing-schemas/migrations/managing#adding-raw-sql).</span></span>

## <a name="overriding-value-generation"></a><span data-ttu-id="0d770-147">重写值生成</span><span class="sxs-lookup"><span data-stu-id="0d770-147">Overriding value generation</span></span>

<span data-ttu-id="0d770-148">尽管为值生成配置了属性，但在许多情况下，你仍可以为其显式指定一个值。</span><span class="sxs-lookup"><span data-stu-id="0d770-148">Although a property is configured for value generation, in many cases you may still explicitly specify a value for it.</span></span> <span data-ttu-id="0d770-149">这是否会实际工作取决于已配置的特定值生成机制;尽管可以指定显式值而不是使用列的默认值，但不能对计算列执行相同的操作。</span><span class="sxs-lookup"><span data-stu-id="0d770-149">Whether this will actually work depends on the specific value generation mechanism that has been configured; while you may specify an explicit value instead of using a column's default value, the same cannot be done with computed columns.</span></span>

<span data-ttu-id="0d770-150">若要使用显式值覆盖值生成，只需将属性设置为任何不是该属性类型的 CLR 默认值的值， (、for、 `null` `string` `0` `int` `Guid.Empty` `Guid` 等等 ) 。</span><span class="sxs-lookup"><span data-stu-id="0d770-150">To override value generation with an explicit value, simply set the property to any value that is not the CLR default value for that property's type (`null` for `string`, `0` for `int`, `Guid.Empty` for `Guid`, etc.).</span></span>

> [!NOTE]
> <span data-ttu-id="0d770-151">默认情况下，尝试将显式值插入 SQL Server 标识失败; [有关解决方法，请参阅以下文档](xref:core/providers/sql-server/value-generation#inserting-explicit-values-into-identity-columns)。</span><span class="sxs-lookup"><span data-stu-id="0d770-151">Trying to insert explicit values into SQL Server IDENTITY fails by default; [see these docs for a workaround](xref:core/providers/sql-server/value-generation#inserting-explicit-values-into-identity-columns).</span></span>

<span data-ttu-id="0d770-152">若要为已配置为在添加或更新时生成的值的属性提供显式值，还必须按如下所示配置属性：</span><span class="sxs-lookup"><span data-stu-id="0d770-152">To provide an explicit value for properties that have been configured as value generated on add or update, you must also configure the property as follows:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedOnAddOrUpdateWithPropertySaveBehavior.cs?name=ValueGeneratedOnAddOrUpdateWithPropertySaveBehavior&highlight=5)]

## <a name="no-value-generation"></a><span data-ttu-id="0d770-153">无值生成</span><span class="sxs-lookup"><span data-stu-id="0d770-153">No value generation</span></span>

<span data-ttu-id="0d770-154">除了前面所述的特定方案以外，属性通常不会配置任何值;这意味着，应用程序始终提供要保存到数据库中的值。</span><span class="sxs-lookup"><span data-stu-id="0d770-154">Apart from specific scenarios such as those described above, properties typically have no value generation configured; this means that it's up to the application to always supply a value to be saved to the database.</span></span> <span data-ttu-id="0d770-155">必须先将此值分配给新实体，然后才能将其添加到上下文中。</span><span class="sxs-lookup"><span data-stu-id="0d770-155">This value must be assigned to new entities before they are added to the context.</span></span>

<span data-ttu-id="0d770-156">但是，在某些情况下，你可能想要禁用已经按约定设置的值生成。</span><span class="sxs-lookup"><span data-stu-id="0d770-156">However, in some cases you may want to disable value generation that has been set up by convention.</span></span> <span data-ttu-id="0d770-157">例如，int 类型的主键通常隐式配置为基于值生成的 (，例如 SQL Server) 上的标识列。</span><span class="sxs-lookup"><span data-stu-id="0d770-157">For example, a primary key of type int is usually implicitly configured as value-generated-on-add (e.g. identity column on SQL Server).</span></span> <span data-ttu-id="0d770-158">可以通过以下方式禁用此操作：</span><span class="sxs-lookup"><span data-stu-id="0d770-158">You can disable this via the following:</span></span>

### <a name="data-annotations"></a>[<span data-ttu-id="0d770-159">数据批注</span><span class="sxs-lookup"><span data-stu-id="0d770-159">Data Annotations</span></span>](#tab/data-annotations)

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/ValueGeneratedNever.cs?name=ValueGeneratedNever&highlight=3)]

### <a name="fluent-api"></a>[<span data-ttu-id="0d770-160">Fluent API</span><span class="sxs-lookup"><span data-stu-id="0d770-160">Fluent API</span></span>](#tab/fluent-api)

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/ValueGeneratedNever.cs?name=ValueGeneratedNever&highlight=5)]

***
