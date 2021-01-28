---
title: 值转换-EF Core
description: 在 Entity Framework Core 模型中配置值转换器
author: ajcvickers
ms.date: 01/16/2021
uid: core/modeling/value-conversions
ms.openlocfilehash: d9d3753c7f0b257a2109e4af1f587df913c15b44
ms.sourcegitcommit: 7700840119b1639275f3b64836e7abb59103f2e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2021
ms.locfileid: "98983438"
---
# <a name="value-conversions"></a><span data-ttu-id="7a509-103">值转换</span><span class="sxs-lookup"><span data-stu-id="7a509-103">Value Conversions</span></span>

<span data-ttu-id="7a509-104">值转换器允许在读取或写入数据库时转换属性值。</span><span class="sxs-lookup"><span data-stu-id="7a509-104">Value converters allow property values to be converted when reading from or writing to the database.</span></span> <span data-ttu-id="7a509-105">此转换可以从一个值转换为同一类型的另一个值 (例如，将字符串) 或从一种类型的值加密为另一种类型的值 (例如，在数据库中将枚举值与字符串相互转换。 ) </span><span class="sxs-lookup"><span data-stu-id="7a509-105">This conversion can be from one value to another of the same type (for example, encrypting strings) or from a value of one type to a value of another type (for example, converting enum values to and from strings in the database.)</span></span>

> [!TIP]
> <span data-ttu-id="7a509-106">通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/ValueConversions/)，你可运行并调试到本文档中的所有代码。</span><span class="sxs-lookup"><span data-stu-id="7a509-106">You can run and debug into all the code in this document by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/ValueConversions/).</span></span>

## <a name="overview"></a><span data-ttu-id="7a509-107">概述</span><span class="sxs-lookup"><span data-stu-id="7a509-107">Overview</span></span>

<span data-ttu-id="7a509-108">值转换器以和的形式指定 `ModelClrType` `ProviderClrType` 。</span><span class="sxs-lookup"><span data-stu-id="7a509-108">Value converters are specified in terms of a `ModelClrType` and a `ProviderClrType`.</span></span> <span data-ttu-id="7a509-109">模型类型是实体类型中的属性的 .NET 类型。</span><span class="sxs-lookup"><span data-stu-id="7a509-109">The model type is the .NET type of the property in the entity type.</span></span> <span data-ttu-id="7a509-110">提供程序类型是数据库提供程序理解的 .NET 类型。</span><span class="sxs-lookup"><span data-stu-id="7a509-110">The provider type is the .NET type understood by the database provider.</span></span> <span data-ttu-id="7a509-111">例如，若要将枚举作为字符串保存在数据库中，模型类型是枚举的类型，而提供程序类型为 `String` 。</span><span class="sxs-lookup"><span data-stu-id="7a509-111">For example, to save enums as strings in the database, the model type is the type of the enum, and the provider type is `String`.</span></span> <span data-ttu-id="7a509-112">这两种类型可以相同。</span><span class="sxs-lookup"><span data-stu-id="7a509-112">These two types can be the same.</span></span>

<span data-ttu-id="7a509-113">使用两个 `Func` 表达式树来定义转换：一个从 `ModelClrType` 到 `ProviderClrType` ，另一个由 `ProviderClrType` 到 `ModelClrType` 。</span><span class="sxs-lookup"><span data-stu-id="7a509-113">Conversions are defined using two `Func` expression trees: one from `ModelClrType` to `ProviderClrType` and the other from `ProviderClrType` to `ModelClrType`.</span></span> <span data-ttu-id="7a509-114">使用表达式树，以便可以将它们编译到数据库访问委托中以便进行有效的转换。</span><span class="sxs-lookup"><span data-stu-id="7a509-114">Expression trees are used so that they can be compiled into the database access delegate for efficient conversions.</span></span> <span data-ttu-id="7a509-115">对于复杂转换，表达式树可能包含对转换方法的简单调用。</span><span class="sxs-lookup"><span data-stu-id="7a509-115">The expression tree may contain a simple call to a conversion method for complex conversions.</span></span>

> [!NOTE]
> <span data-ttu-id="7a509-116">为值转换配置的属性可能还需要指定 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601> 。</span><span class="sxs-lookup"><span data-stu-id="7a509-116">A property that has been configured for value conversion may also need to specify a <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601>.</span></span> <span data-ttu-id="7a509-117">有关详细信息，请参阅下面的示例和 [值](xref:core/modeling/value-comparers) 比较器文档。</span><span class="sxs-lookup"><span data-stu-id="7a509-117">See the examples below, and the [Value Comparers](xref:core/modeling/value-comparers) documentation for more information.</span></span>

## <a name="configuring-a-value-converter"></a><span data-ttu-id="7a509-118">配置值转换器</span><span class="sxs-lookup"><span data-stu-id="7a509-118">Configuring a value converter</span></span>

<span data-ttu-id="7a509-119">值转换在中进行配置 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A?displayProperty=nameWithType> 。</span><span class="sxs-lookup"><span data-stu-id="7a509-119">Value conversions are configured in <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="7a509-120">例如，假设枚举和实体类型定义为：</span><span class="sxs-lookup"><span data-stu-id="7a509-120">For example, consider an enum and entity type defined as:</span></span>

<!--
        public class Rider
        {
            public int Id { get; set; }
            public EquineBeast Mount { get; set; }
        }

        public enum EquineBeast
        {
            Donkey,
            Mule,
            Horse,
            Unicorn
        }
-->
[!code-csharp[BeastAndRider](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=BeastAndRider)]

<span data-ttu-id="7a509-121">在中，可以将转换配置 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> 为在数据库中存储枚举值（如 "Donkey"、"Mule" 等），只需提供一个将从转换 `ModelClrType` 为的函数 `ProviderClrType` ，另一个函数用于相反转换：</span><span class="sxs-lookup"><span data-stu-id="7a509-121">Conversions can be configured in <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> to store the enum values as strings such as "Donkey", "Mule", etc. in the database; you simply need to provide one function which converts from the `ModelClrType` to the `ProviderClrType`, and another for the opposite conversion:</span></span>

<!--
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                modelBuilder
                    .Entity<Rider>()
                    .Property(e => e.Mount)
                    .HasConversion(
                        v => v.ToString(),
                        v => (EquineBeast)Enum.Parse(typeof(EquineBeast), v));
            }
-->
[!code-csharp[ExplicitConversion](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ExplicitConversion)]

> [!NOTE]
> <span data-ttu-id="7a509-122">`null`值绝不会传递到值转换器。</span><span class="sxs-lookup"><span data-stu-id="7a509-122">A `null` value will never be passed to a value converter.</span></span> <span data-ttu-id="7a509-123">数据库列中的 null 在实体实例中始终为 null，反之亦然。</span><span class="sxs-lookup"><span data-stu-id="7a509-123">A null in a database column is always a null in the entity instance, and vice-versa.</span></span> <span data-ttu-id="7a509-124">这使得转换的实现变得更简单，并使其能够在可以为 null 和不可为 null 的属性之间共享。</span><span class="sxs-lookup"><span data-stu-id="7a509-124">This makes the implementation of conversions easier and allows them to be shared amongst nullable and non-nullable properties.</span></span> <span data-ttu-id="7a509-125">有关详细信息，请参阅 [GitHub 问题 #13850](https://github.com/dotnet/efcore/issues/13850) 。</span><span class="sxs-lookup"><span data-stu-id="7a509-125">See [GitHub issue #13850](https://github.com/dotnet/efcore/issues/13850) for more information.</span></span>

## <a name="pre-defined-conversions"></a><span data-ttu-id="7a509-126">预定义的转换</span><span class="sxs-lookup"><span data-stu-id="7a509-126">Pre-defined conversions</span></span>

<span data-ttu-id="7a509-127">EF Core 包含许多预定义的转换，这些转换可避免手动编写转换函数。</span><span class="sxs-lookup"><span data-stu-id="7a509-127">EF Core contains many pre-defined conversions that avoid the need to write conversion functions manually.</span></span> <span data-ttu-id="7a509-128">相反，EF Core 会根据模型中的属性类型和请求的数据库提供程序类型选取要使用的转换。</span><span class="sxs-lookup"><span data-stu-id="7a509-128">Instead, EF Core will pick the conversion to use based on the property type in the model and the requested database provider type.</span></span>

<span data-ttu-id="7a509-129">例如，枚举到字符串的转换用作上面的示例，但在将提供程序类型配置为使用的泛型类型时，EF Core 实际上会自动执行此 `string` <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.HasConversion%2A> 操作：</span><span class="sxs-lookup"><span data-stu-id="7a509-129">For example, enum to string conversions are used as an example above, but EF Core will actually do this automatically when the provider type is configured as `string` using the generic type of <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.HasConversion%2A>:</span></span>

<!--
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                modelBuilder
                    .Entity<Rider>()
                    .Property(e => e.Mount)
                    .HasConversion<string>();
            }
-->
[!code-csharp[ConversionByClrType](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ConversionByClrType)]

<span data-ttu-id="7a509-130">可以通过显式指定数据库列类型来实现相同的目的。</span><span class="sxs-lookup"><span data-stu-id="7a509-130">The same thing can be achieved by explicitly specifying the database column type.</span></span> <span data-ttu-id="7a509-131">例如，如果定义了实体类型，如下所示：</span><span class="sxs-lookup"><span data-stu-id="7a509-131">For example, if the entity type is defined like so:</span></span>

### <a name="data-annotations"></a>[<span data-ttu-id="7a509-132">数据批注</span><span class="sxs-lookup"><span data-stu-id="7a509-132">Data Annotations</span></span>](#tab/data-annotations)

<!--
        public class Rider2
        {
            public int Id { get; set; }

            [Column(TypeName = "nvarchar(24)")]
            public EquineBeast Mount { get; set; }
        }
-->
[!code-csharp[ConversionByDatabaseType](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ConversionByDatabaseType)]

### <a name="fluent-api"></a>[<span data-ttu-id="7a509-133">Fluent API</span><span class="sxs-lookup"><span data-stu-id="7a509-133">Fluent API</span></span>](#tab/fluent-api)

<!--
                modelBuilder
                    .Entity<Rider2>()
                    .Property(e => e.Mount)
                    .HasColumnType("nvarchar(24)");
-->
[!code-csharp[ConversionByDatabaseTypeFluent](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ConversionByDatabaseTypeFluent)]

***

<span data-ttu-id="7a509-134">然后，枚举值将作为字符串保存在数据库中，而不会在中进行任何进一步的配置 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> 。</span><span class="sxs-lookup"><span data-stu-id="7a509-134">Then the enum values will be saved as strings in the database without any further configuration in <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A>.</span></span>

## <a name="the-valueconverter-class"></a><span data-ttu-id="7a509-135">ValueConverter 类</span><span class="sxs-lookup"><span data-stu-id="7a509-135">The ValueConverter class</span></span>

<span data-ttu-id="7a509-136"><xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.HasConversion%2A>如上所述调用会创建一个 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.ValueConverter%602> 实例，并在属性中设置该实例。</span><span class="sxs-lookup"><span data-stu-id="7a509-136">Calling <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.HasConversion%2A> as shown above will create a <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.ValueConverter%602> instance and set it on the property.</span></span> <span data-ttu-id="7a509-137">`ValueConverter`可以显式创建。</span><span class="sxs-lookup"><span data-stu-id="7a509-137">The `ValueConverter` can instead be created explicitly.</span></span> <span data-ttu-id="7a509-138">例如：</span><span class="sxs-lookup"><span data-stu-id="7a509-138">For example:</span></span>

<!--
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                var converter = new ValueConverter<EquineBeast, string>(
                    v => v.ToString(),
                    v => (EquineBeast)Enum.Parse(typeof(EquineBeast), v));

                modelBuilder
                    .Entity<Rider>()
                    .Property(e => e.Mount)
                    .HasConversion(converter);
            }
-->
[!code-csharp[ConversionByConverterInstance](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ConversionByConverterInstance)]

<span data-ttu-id="7a509-139">当多个属性使用相同的转换时，这可能很有用。</span><span class="sxs-lookup"><span data-stu-id="7a509-139">This can be useful when multiple properties use the same conversion.</span></span>

## <a name="built-in-converters"></a><span data-ttu-id="7a509-140">内置转换器</span><span class="sxs-lookup"><span data-stu-id="7a509-140">Built-in converters</span></span>

<span data-ttu-id="7a509-141">如上所述，EF Core 附带一组预定义的 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.ValueConverter%602> 类，这些类位于 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion> 命名空间中。</span><span class="sxs-lookup"><span data-stu-id="7a509-141">As mentioned above, EF Core ships with a set of pre-defined <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.ValueConverter%602> classes, found in the <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion> namespace.</span></span> <span data-ttu-id="7a509-142">在许多情况下，EF 会基于模型中的属性类型和数据库中所请求的类型选择适当的内置转换器，如上面的枚举所示。</span><span class="sxs-lookup"><span data-stu-id="7a509-142">In many cases EF will choose the appropriate built-in converter based on the type of the property in the model and the type requested in the database, as shown above for enums.</span></span> <span data-ttu-id="7a509-143">例如， `.HasConversion<int>()` 对 `bool` 属性使用会导致 EF Core 将布尔值转换为数字零和一个值：</span><span class="sxs-lookup"><span data-stu-id="7a509-143">For example, using `.HasConversion<int>()` on a `bool` property will cause EF Core to convert bool values to numerical zero and one values:</span></span>

<!--
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                modelBuilder
                    .Entity<User>()
                    .Property(e => e.IsActive)
                    .HasConversion<int>();
            }
-->
[!code-csharp[ConversionByBuiltInBoolToInt](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ConversionByBuiltInBoolToInt)]

<span data-ttu-id="7a509-144">此功能与创建内置的实例 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToZeroOneConverter%601> 并显式设置它的功能相同：</span><span class="sxs-lookup"><span data-stu-id="7a509-144">This is functionally the same as creating an instance of the built-in <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToZeroOneConverter%601> and setting it explicitly:</span></span>

<!--
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                var converter = new BoolToZeroOneConverter<int>();

                modelBuilder
                    .Entity<User>()
                    .Property(e => e.IsActive)
                    .HasConversion(converter);
            }
-->
[!code-csharp[ConversionByBuiltInBoolToIntExplicit](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ConversionByBuiltInBoolToIntExplicit)]

<span data-ttu-id="7a509-145">下表汇总了从模型/属性类型到数据库提供程序类型的常用预定义转换。</span><span class="sxs-lookup"><span data-stu-id="7a509-145">The following table summarizes commonly-used pre-defined conversions from model/property types to database provider types.</span></span> <span data-ttu-id="7a509-146">在表中 `any_numeric_type` ，是指 `int` 、、、 `short` `long` `byte` 、 `uint` 、 `ushort` `ulong` `sbyte` `char` `decimal` `float` `double` 、、、、、或中的一个。</span><span class="sxs-lookup"><span data-stu-id="7a509-146">In the table `any_numeric_type` means one of `int`, `short`, `long`, `byte`, `uint`, `ushort`, `ulong`, `sbyte`, `char`, `decimal`, `float`, or `double`.</span></span>

| <span data-ttu-id="7a509-147">模型/属性类型</span><span class="sxs-lookup"><span data-stu-id="7a509-147">Model/property type</span></span> | <span data-ttu-id="7a509-148">提供程序/数据库类型</span><span class="sxs-lookup"><span data-stu-id="7a509-148">Provider/database type</span></span> | <span data-ttu-id="7a509-149">转换</span><span class="sxs-lookup"><span data-stu-id="7a509-149">Conversion</span></span>                                                | <span data-ttu-id="7a509-150">使用情况</span><span class="sxs-lookup"><span data-stu-id="7a509-150">Usage</span></span>
|:--------------------|------------------------|-----------------------------------------------------------|------
| <span data-ttu-id="7a509-151">bool</span><span class="sxs-lookup"><span data-stu-id="7a509-151">bool</span></span>                | <span data-ttu-id="7a509-152">any_numeric_type</span><span class="sxs-lookup"><span data-stu-id="7a509-152">any_numeric_type</span></span>       | <span data-ttu-id="7a509-153">False/true 到0/1</span><span class="sxs-lookup"><span data-stu-id="7a509-153">False/true to 0/1</span></span>                                         | `.HasConversion<any_numeric_type>()`
|                     | <span data-ttu-id="7a509-154">any_numeric_type</span><span class="sxs-lookup"><span data-stu-id="7a509-154">any_numeric_type</span></span>       | <span data-ttu-id="7a509-155">对于任意两个数值为 False/true</span><span class="sxs-lookup"><span data-stu-id="7a509-155">False/true to any two numbers</span></span>                             | <span data-ttu-id="7a509-156">使用 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToTwoValuesConverter%601></span><span class="sxs-lookup"><span data-stu-id="7a509-156">Use <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToTwoValuesConverter%601></span></span>
|                     | <span data-ttu-id="7a509-157">字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-157">string</span></span>                 | <span data-ttu-id="7a509-158">False/true 到 "Y"/"N"</span><span class="sxs-lookup"><span data-stu-id="7a509-158">False/true to "Y"/"N"</span></span>                                     | `.HasConversion<string>()`
|                     | <span data-ttu-id="7a509-159">字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-159">string</span></span>                 | <span data-ttu-id="7a509-160">对于任意两个字符串为 False/true</span><span class="sxs-lookup"><span data-stu-id="7a509-160">False/true to any two strings</span></span>                             | <span data-ttu-id="7a509-161">使用 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToStringConverter></span><span class="sxs-lookup"><span data-stu-id="7a509-161">Use <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToStringConverter></span></span>
| <span data-ttu-id="7a509-162">any_numeric_type</span><span class="sxs-lookup"><span data-stu-id="7a509-162">any_numeric_type</span></span>    | <span data-ttu-id="7a509-163">bool</span><span class="sxs-lookup"><span data-stu-id="7a509-163">bool</span></span>                   | <span data-ttu-id="7a509-164">0/1 到 false/true</span><span class="sxs-lookup"><span data-stu-id="7a509-164">0/1 to false/true</span></span>                                         | `.HasConversion<bool>()`
|                     | <span data-ttu-id="7a509-165">any_numeric_type</span><span class="sxs-lookup"><span data-stu-id="7a509-165">any_numeric_type</span></span>       | <span data-ttu-id="7a509-166">简单强制转换</span><span class="sxs-lookup"><span data-stu-id="7a509-166">Simple cast</span></span>                                               | `.HasConversion<any_numeric_type>()`
|                     | <span data-ttu-id="7a509-167">字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-167">string</span></span>                 | <span data-ttu-id="7a509-168">字符串形式的数字</span><span class="sxs-lookup"><span data-stu-id="7a509-168">The number as a string</span></span>                                    | `.HasConversion<string>()`
| <span data-ttu-id="7a509-169">枚举</span><span class="sxs-lookup"><span data-stu-id="7a509-169">Enum</span></span>                | <span data-ttu-id="7a509-170">any_numeric_type</span><span class="sxs-lookup"><span data-stu-id="7a509-170">any_numeric_type</span></span>       | <span data-ttu-id="7a509-171">枚举的数值</span><span class="sxs-lookup"><span data-stu-id="7a509-171">The numeric value of the enum</span></span>                             | `.HasConversion<any_numeric_type>()`
|                     | <span data-ttu-id="7a509-172">字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-172">string</span></span>                 | <span data-ttu-id="7a509-173">枚举值的字符串表示形式</span><span class="sxs-lookup"><span data-stu-id="7a509-173">The string representation of the enum value</span></span>               | `.HasConversion<string>()`
| <span data-ttu-id="7a509-174">字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-174">string</span></span>              | <span data-ttu-id="7a509-175">bool</span><span class="sxs-lookup"><span data-stu-id="7a509-175">bool</span></span>                   | <span data-ttu-id="7a509-176">将字符串分析为布尔型</span><span class="sxs-lookup"><span data-stu-id="7a509-176">Parses the string as a bool</span></span>                               | `.HasConversion<bool>()`
|                     | <span data-ttu-id="7a509-177">any_numeric_type</span><span class="sxs-lookup"><span data-stu-id="7a509-177">any_numeric_type</span></span>       | <span data-ttu-id="7a509-178">将字符串分析为给定的数值类型</span><span class="sxs-lookup"><span data-stu-id="7a509-178">Parses the string as the given numeric type</span></span>               | `.HasConversion<any_numeric_type>()`
|                     | <span data-ttu-id="7a509-179">char</span><span class="sxs-lookup"><span data-stu-id="7a509-179">char</span></span>                   | <span data-ttu-id="7a509-180">字符串的第一个字符</span><span class="sxs-lookup"><span data-stu-id="7a509-180">The first character of the string</span></span>                         | `.HasConversion<char>()`
|                     | <span data-ttu-id="7a509-181">DateTime</span><span class="sxs-lookup"><span data-stu-id="7a509-181">DateTime</span></span>               | <span data-ttu-id="7a509-182">将字符串分析为日期时间</span><span class="sxs-lookup"><span data-stu-id="7a509-182">Parses the string as a DateTime</span></span>                           | `.HasConversion<DateTime>()`
|                     | <span data-ttu-id="7a509-183">DateTimeOffset</span><span class="sxs-lookup"><span data-stu-id="7a509-183">DateTimeOffset</span></span>         | <span data-ttu-id="7a509-184">将字符串分析为 DateTimeOffset</span><span class="sxs-lookup"><span data-stu-id="7a509-184">Parses the string as a DateTimeOffset</span></span>                     | `.HasConversion<DateTimeOffset>()`
|                     | <span data-ttu-id="7a509-185">TimeSpan</span><span class="sxs-lookup"><span data-stu-id="7a509-185">TimeSpan</span></span>               | <span data-ttu-id="7a509-186">将字符串分析为 TimeSpan</span><span class="sxs-lookup"><span data-stu-id="7a509-186">Parses the string as a TimeSpan</span></span>                           | `.HasConversion<TimeSpan>()`
|                     | <span data-ttu-id="7a509-187">Guid</span><span class="sxs-lookup"><span data-stu-id="7a509-187">Guid</span></span>                   | <span data-ttu-id="7a509-188">将字符串分析为 Guid</span><span class="sxs-lookup"><span data-stu-id="7a509-188">Parses the string as a Guid</span></span>                               | `.HasConversion<Guid>()`
|                     | <span data-ttu-id="7a509-189">byte[]</span><span class="sxs-lookup"><span data-stu-id="7a509-189">byte[]</span></span>                 | <span data-ttu-id="7a509-190">UTF8 字节形式的字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-190">The string as UTF8 bytes</span></span>                                  | `.HasConversion<byte[]>()`
| <span data-ttu-id="7a509-191">char</span><span class="sxs-lookup"><span data-stu-id="7a509-191">char</span></span>                | <span data-ttu-id="7a509-192">string</span><span class="sxs-lookup"><span data-stu-id="7a509-192">string</span></span>                 | <span data-ttu-id="7a509-193">单个字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-193">A single character string</span></span>                                 | `.HasConversion<string>()`
| <span data-ttu-id="7a509-194">DateTime</span><span class="sxs-lookup"><span data-stu-id="7a509-194">DateTime</span></span>            | <span data-ttu-id="7a509-195">long</span><span class="sxs-lookup"><span data-stu-id="7a509-195">long</span></span>                   | <span data-ttu-id="7a509-196">编码日期/时间保留日期时间类型</span><span class="sxs-lookup"><span data-stu-id="7a509-196">Encoded date/time preserving DateTime.Kind</span></span>                | `.HasConversion<long>()`
|                     | <span data-ttu-id="7a509-197">long</span><span class="sxs-lookup"><span data-stu-id="7a509-197">long</span></span>                   | <span data-ttu-id="7a509-198">刻度</span><span class="sxs-lookup"><span data-stu-id="7a509-198">Ticks</span></span>                                                     | <span data-ttu-id="7a509-199">使用 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToTicksConverter></span><span class="sxs-lookup"><span data-stu-id="7a509-199">Use <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToTicksConverter></span></span>
|                     | <span data-ttu-id="7a509-200">字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-200">string</span></span>                 | <span data-ttu-id="7a509-201">固定区域性日期/时间字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-201">Invariant culture date/time string</span></span>                        | `.HasConversion<string>()`
| <span data-ttu-id="7a509-202">DateTimeOffset</span><span class="sxs-lookup"><span data-stu-id="7a509-202">DateTimeOffset</span></span>      | <span data-ttu-id="7a509-203">long</span><span class="sxs-lookup"><span data-stu-id="7a509-203">long</span></span>                   | <span data-ttu-id="7a509-204">带偏移量的编码日期/时间</span><span class="sxs-lookup"><span data-stu-id="7a509-204">Encoded date/time with offset</span></span>                             | `.HasConversion<long>()`
|                     | <span data-ttu-id="7a509-205">字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-205">string</span></span>                 | <span data-ttu-id="7a509-206">具有偏移量的固定区域性日期/时间字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-206">Invariant culture date/time string with offset</span></span>            | `.HasConversion<string>()`
| <span data-ttu-id="7a509-207">TimeSpan</span><span class="sxs-lookup"><span data-stu-id="7a509-207">TimeSpan</span></span>            | <span data-ttu-id="7a509-208">long</span><span class="sxs-lookup"><span data-stu-id="7a509-208">long</span></span>                   | <span data-ttu-id="7a509-209">刻度</span><span class="sxs-lookup"><span data-stu-id="7a509-209">Ticks</span></span>                                                     | `.HasConversion<long>()`
|                     | <span data-ttu-id="7a509-210">字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-210">string</span></span>                 | <span data-ttu-id="7a509-211">固定区域性时间跨度字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-211">Invariant culture time span string</span></span>                        | `.HasConversion<string>()`
| <span data-ttu-id="7a509-212">URI</span><span class="sxs-lookup"><span data-stu-id="7a509-212">Uri</span></span>                 | <span data-ttu-id="7a509-213">string</span><span class="sxs-lookup"><span data-stu-id="7a509-213">string</span></span>                 | <span data-ttu-id="7a509-214">字符串形式的 URI</span><span class="sxs-lookup"><span data-stu-id="7a509-214">The URI as a string</span></span>                                       | `.HasConversion<string>()`
| <span data-ttu-id="7a509-215">PhysicalAddress</span><span class="sxs-lookup"><span data-stu-id="7a509-215">PhysicalAddress</span></span>     | <span data-ttu-id="7a509-216">字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-216">string</span></span>                 | <span data-ttu-id="7a509-217">字符串形式的地址</span><span class="sxs-lookup"><span data-stu-id="7a509-217">The address as a string</span></span>                                   | `.HasConversion<string>()`
|                     | <span data-ttu-id="7a509-218">byte[]</span><span class="sxs-lookup"><span data-stu-id="7a509-218">byte[]</span></span>                 | <span data-ttu-id="7a509-219">大字节序网络顺序中的字节数</span><span class="sxs-lookup"><span data-stu-id="7a509-219">Bytes in big-endian network order</span></span>                         | `.HasConversion<byte[]>()`
| <span data-ttu-id="7a509-220">IPAddress</span><span class="sxs-lookup"><span data-stu-id="7a509-220">IPAddress</span></span>           | <span data-ttu-id="7a509-221">字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-221">string</span></span>                 | <span data-ttu-id="7a509-222">字符串形式的地址</span><span class="sxs-lookup"><span data-stu-id="7a509-222">The address as a string</span></span>                                   | `.HasConversion<string>()`
|                     | <span data-ttu-id="7a509-223">byte[]</span><span class="sxs-lookup"><span data-stu-id="7a509-223">byte[]</span></span>                 | <span data-ttu-id="7a509-224">大字节序网络顺序中的字节数</span><span class="sxs-lookup"><span data-stu-id="7a509-224">Bytes in big-endian network order</span></span>                         | `.HasConversion<byte[]>()`
| <span data-ttu-id="7a509-225">Guid</span><span class="sxs-lookup"><span data-stu-id="7a509-225">Guid</span></span>                | <span data-ttu-id="7a509-226">字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-226">string</span></span>                 | <span data-ttu-id="7a509-227">采用 "dddddddd-dddd-dddd-dddd-dddddddddddd" 格式的 GUID</span><span class="sxs-lookup"><span data-stu-id="7a509-227">The GUID in 'dddddddd-dddd-dddd-dddd-dddddddddddd' format</span></span> | `.HasConversion<string>()`
|                     | <span data-ttu-id="7a509-228">byte[]</span><span class="sxs-lookup"><span data-stu-id="7a509-228">byte[]</span></span>                 | <span data-ttu-id="7a509-229">.NET 二进制序列化顺序中的字节数</span><span class="sxs-lookup"><span data-stu-id="7a509-229">Bytes in .NET binary serialization order</span></span>                  | `.HasConversion<byte[]>()`

<span data-ttu-id="7a509-230">请注意，这些转换假定值的格式适用于转换。</span><span class="sxs-lookup"><span data-stu-id="7a509-230">Note that these conversions assume that the format of the value is appropriate for the conversion.</span></span> <span data-ttu-id="7a509-231">例如，如果字符串值无法分析为数字，将字符串转换为数字将会失败。</span><span class="sxs-lookup"><span data-stu-id="7a509-231">For example, converting strings to numbers will fail if the string values cannot be parsed as numbers.</span></span>

<span data-ttu-id="7a509-232">内置转换器的完整列表如下：</span><span class="sxs-lookup"><span data-stu-id="7a509-232">The full list of built-in converters is:</span></span>

* <span data-ttu-id="7a509-233">转换 bool 属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-233">Converting bool properties:</span></span>
  * <span data-ttu-id="7a509-234"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToStringConverter> -Bool 到字符串（如 "Y" 和 "N"）</span><span class="sxs-lookup"><span data-stu-id="7a509-234"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToStringConverter> - Bool to strings such as "Y" and "N"</span></span>
  * <span data-ttu-id="7a509-235"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToTwoValuesConverter%601> -布尔值到任意两个值</span><span class="sxs-lookup"><span data-stu-id="7a509-235"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToTwoValuesConverter%601> - Bool to any two values</span></span>
  * <span data-ttu-id="7a509-236"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToZeroOneConverter%601> -布尔值为零和一个</span><span class="sxs-lookup"><span data-stu-id="7a509-236"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToZeroOneConverter%601> - Bool to zero and one</span></span>
* <span data-ttu-id="7a509-237">转换字节数组属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-237">Converting byte array properties:</span></span>
  * <span data-ttu-id="7a509-238"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BytesToStringConverter> -字节数组到 Base64 编码的字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-238"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BytesToStringConverter> - Byte array to Base64-encoded string</span></span>
* <span data-ttu-id="7a509-239">任何只需要类型强制转换的转换</span><span class="sxs-lookup"><span data-stu-id="7a509-239">Any conversion that requires only a type-cast</span></span>
  * <span data-ttu-id="7a509-240"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.CastingConverter%602> -只需要类型强制转换的转换</span><span class="sxs-lookup"><span data-stu-id="7a509-240"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.CastingConverter%602> - Conversions that require only a type cast</span></span>
* <span data-ttu-id="7a509-241">转换 char 属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-241">Converting char properties:</span></span>
  * <span data-ttu-id="7a509-242"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.CharToStringConverter> -Char 到单字符字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-242"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.CharToStringConverter> - Char to single character string</span></span>
* <span data-ttu-id="7a509-243">转换 <xref:System.DateTimeOffset> 属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-243">Converting <xref:System.DateTimeOffset> properties:</span></span>
  * <span data-ttu-id="7a509-244"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeOffsetToBinaryConverter> - <xref:System.DateTimeOffset> 到二进制编码的64位值</span><span class="sxs-lookup"><span data-stu-id="7a509-244"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeOffsetToBinaryConverter> - <xref:System.DateTimeOffset> to binary-encoded 64-bit value</span></span>
  * <span data-ttu-id="7a509-245"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeOffsetToBytesConverter> - <xref:System.DateTimeOffset> 到字节数组</span><span class="sxs-lookup"><span data-stu-id="7a509-245"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeOffsetToBytesConverter> - <xref:System.DateTimeOffset> to byte array</span></span>
  * <span data-ttu-id="7a509-246"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeOffsetToStringConverter> - <xref:System.DateTimeOffset> 到字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-246"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeOffsetToStringConverter> - <xref:System.DateTimeOffset> to string</span></span>
* <span data-ttu-id="7a509-247">转换 <xref:System.DateTime> 属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-247">Converting <xref:System.DateTime> properties:</span></span>
  * <span data-ttu-id="7a509-248"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToBinaryConverter> - <xref:System.DateTime> 到64位值（包括 Datetimekind.utc）</span><span class="sxs-lookup"><span data-stu-id="7a509-248"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToBinaryConverter> - <xref:System.DateTime> to 64-bit value including DateTimeKind</span></span>
  * <span data-ttu-id="7a509-249"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToStringConverter> - <xref:System.DateTime> 到字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-249"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToStringConverter> - <xref:System.DateTime> to string</span></span>
  * <span data-ttu-id="7a509-250"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToTicksConverter> - <xref:System.DateTime> 到刻度</span><span class="sxs-lookup"><span data-stu-id="7a509-250"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToTicksConverter> - <xref:System.DateTime> to ticks</span></span>
* <span data-ttu-id="7a509-251">转换枚举属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-251">Converting enum properties:</span></span>
  * <span data-ttu-id="7a509-252"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.EnumToNumberConverter%602> -枚举到基础数字</span><span class="sxs-lookup"><span data-stu-id="7a509-252"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.EnumToNumberConverter%602> - Enum to underlying number</span></span>
  * <span data-ttu-id="7a509-253"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.EnumToStringConverter%601> -枚举到字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-253"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.EnumToStringConverter%601> - Enum to string</span></span>
* <span data-ttu-id="7a509-254">转换 <xref:System.Guid> 属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-254">Converting <xref:System.Guid> properties:</span></span>
  * <span data-ttu-id="7a509-255"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.GuidToBytesConverter> - <xref:System.Guid> 到字节数组</span><span class="sxs-lookup"><span data-stu-id="7a509-255"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.GuidToBytesConverter> - <xref:System.Guid> to byte array</span></span>
  * <span data-ttu-id="7a509-256"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.GuidToStringConverter> - <xref:System.Guid> 到字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-256"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.GuidToStringConverter> - <xref:System.Guid> to string</span></span>
* <span data-ttu-id="7a509-257">转换 <xref:System.Net.IPAddress> 属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-257">Converting <xref:System.Net.IPAddress> properties:</span></span>
  * <span data-ttu-id="7a509-258"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.IPAddressToBytesConverter> - <xref:System.Net.IPAddress> 到字节数组</span><span class="sxs-lookup"><span data-stu-id="7a509-258"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.IPAddressToBytesConverter> - <xref:System.Net.IPAddress> to byte array</span></span>
  * <span data-ttu-id="7a509-259"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.IPAddressToStringConverter> - <xref:System.Net.IPAddress> 到字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-259"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.IPAddressToStringConverter> - <xref:System.Net.IPAddress> to string</span></span>
* <span data-ttu-id="7a509-260">将数值转换 (int、double、decimal 等 ) 属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-260">Converting numeric (int, double, decimal, etc.) properties:</span></span>
  * <span data-ttu-id="7a509-261"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.NumberToBytesConverter%601> -任何数值到字节数组</span><span class="sxs-lookup"><span data-stu-id="7a509-261"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.NumberToBytesConverter%601> - Any numerical value to byte array</span></span>
  * <span data-ttu-id="7a509-262"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.NumberToStringConverter%601> -任何数值到字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-262"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.NumberToStringConverter%601> - Any numerical value to string</span></span>
* <span data-ttu-id="7a509-263">转换 <xref:System.Net.NetworkInformation.PhysicalAddress> 属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-263">Converting <xref:System.Net.NetworkInformation.PhysicalAddress> properties:</span></span>
  * <span data-ttu-id="7a509-264"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.PhysicalAddressToBytesConverter> - <xref:System.Net.NetworkInformation.PhysicalAddress> 到字节数组</span><span class="sxs-lookup"><span data-stu-id="7a509-264"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.PhysicalAddressToBytesConverter> - <xref:System.Net.NetworkInformation.PhysicalAddress> to byte array</span></span>
  * <span data-ttu-id="7a509-265"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.PhysicalAddressToStringConverter> - <xref:System.Net.NetworkInformation.PhysicalAddress> 到字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-265"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.PhysicalAddressToStringConverter> - <xref:System.Net.NetworkInformation.PhysicalAddress> to string</span></span>
* <span data-ttu-id="7a509-266">转换字符串属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-266">Converting string properties:</span></span>
  * <span data-ttu-id="7a509-267"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToBoolConverter> -字符串（例如 "Y" 和 "N"）到 bool</span><span class="sxs-lookup"><span data-stu-id="7a509-267"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToBoolConverter> - Strings such as "Y" and "N" to bool</span></span>
  * <span data-ttu-id="7a509-268"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToBytesConverter> -字符串到 UTF8 字节</span><span class="sxs-lookup"><span data-stu-id="7a509-268"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToBytesConverter> - String to UTF8 bytes</span></span>
  * <span data-ttu-id="7a509-269"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToCharConverter> -字符串到字符</span><span class="sxs-lookup"><span data-stu-id="7a509-269"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToCharConverter> - String to character</span></span>
  * <span data-ttu-id="7a509-270"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToDateTimeConverter> -字符串到 <xref:System.DateTime></span><span class="sxs-lookup"><span data-stu-id="7a509-270"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToDateTimeConverter> - String to <xref:System.DateTime></span></span>
  * <span data-ttu-id="7a509-271"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToDateTimeOffsetConverter> -字符串到 <xref:System.DateTimeOffset></span><span class="sxs-lookup"><span data-stu-id="7a509-271"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToDateTimeOffsetConverter> - String to <xref:System.DateTimeOffset></span></span>
  * <span data-ttu-id="7a509-272"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToEnumConverter%601> -要枚举的字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-272"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToEnumConverter%601> - String to enum</span></span>
  * <span data-ttu-id="7a509-273"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToGuidConverter> -字符串到 <xref:System.Guid></span><span class="sxs-lookup"><span data-stu-id="7a509-273"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToGuidConverter> - String to <xref:System.Guid></span></span>
  * <span data-ttu-id="7a509-274"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToNumberConverter%601> -字符串到数值类型</span><span class="sxs-lookup"><span data-stu-id="7a509-274"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToNumberConverter%601> - String to numeric type</span></span>
  * <span data-ttu-id="7a509-275"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToTimeSpanConverter> -字符串到 <xref:System.TimeSpan></span><span class="sxs-lookup"><span data-stu-id="7a509-275"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToTimeSpanConverter> - String to <xref:System.TimeSpan></span></span>
  * <span data-ttu-id="7a509-276"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToUriConverter> -字符串到 <xref:System.Uri></span><span class="sxs-lookup"><span data-stu-id="7a509-276"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToUriConverter> - String to <xref:System.Uri></span></span>
* <span data-ttu-id="7a509-277">转换 <xref:System.TimeSpan> 属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-277">Converting <xref:System.TimeSpan> properties:</span></span>
  * <span data-ttu-id="7a509-278"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.TimeSpanToStringConverter> - <xref:System.TimeSpan> 到字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-278"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.TimeSpanToStringConverter> - <xref:System.TimeSpan> to string</span></span>
  * <span data-ttu-id="7a509-279"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.TimeSpanToTicksConverter> - <xref:System.TimeSpan> 到刻度</span><span class="sxs-lookup"><span data-stu-id="7a509-279"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.TimeSpanToTicksConverter> - <xref:System.TimeSpan> to ticks</span></span>
* <span data-ttu-id="7a509-280">转换 <xref:System.Uri> 属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-280">Converting <xref:System.Uri> properties:</span></span>
  * <span data-ttu-id="7a509-281"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.UriToStringConverter> - <xref:System.Uri> 到字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-281"><xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.UriToStringConverter> - <xref:System.Uri> to string</span></span>

<span data-ttu-id="7a509-282">请注意，所有内置的转换器都是无状态的，因此，多个属性可以安全地共享单个实例。</span><span class="sxs-lookup"><span data-stu-id="7a509-282">Note that all the built-in converters are stateless and so a single instance can be safely shared by multiple properties.</span></span>

## <a name="column-facets-and-mapping-hints"></a><span data-ttu-id="7a509-283">列 facet 和映射提示</span><span class="sxs-lookup"><span data-stu-id="7a509-283">Column facets and mapping hints</span></span>

<span data-ttu-id="7a509-284">某些数据库类型具有用于修改数据存储方式的方面。</span><span class="sxs-lookup"><span data-stu-id="7a509-284">Some database types have facets that modify how the data is stored.</span></span> <span data-ttu-id="7a509-285">其中包括:</span><span class="sxs-lookup"><span data-stu-id="7a509-285">These include:</span></span>

* <span data-ttu-id="7a509-286">"小数位数" 和 "日期/时间" 列的精度和小数位数</span><span class="sxs-lookup"><span data-stu-id="7a509-286">Precision and scale for decimals and date/time columns</span></span>
* <span data-ttu-id="7a509-287">二进制和字符串列的大小/长度</span><span class="sxs-lookup"><span data-stu-id="7a509-287">Size/length for binary and string columns</span></span>
* <span data-ttu-id="7a509-288">字符串列的 Unicode</span><span class="sxs-lookup"><span data-stu-id="7a509-288">Unicode for string columns</span></span>

<span data-ttu-id="7a509-289">对于使用值转换器的属性，可以按正常方式配置这些层面，并将其应用于转换后的数据库类型。</span><span class="sxs-lookup"><span data-stu-id="7a509-289">These facets can be configured in the normal way for a property that uses a value converter, and will apply to the converted database type.</span></span> <span data-ttu-id="7a509-290">例如，在将枚举转换为字符串时，可以指定数据库列应为非 Unicode，并且最多可存储20个字符：</span><span class="sxs-lookup"><span data-stu-id="7a509-290">For example, when converting from an enum to strings, we can specify that the database column should be non-Unicode and store up to 20 characters:</span></span>

<!--
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                modelBuilder
                    .Entity<Rider>()
                    .Property(e => e.Mount)
                    .HasConversion<string>()
                    .HasMaxLength(20)
                    .IsUnicode(false);
            }
-->
[!code-csharp[ConversionByClrTypeWithFacets](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ConversionByClrTypeWithFacets)]

<span data-ttu-id="7a509-291">或者，显式创建转换器：</span><span class="sxs-lookup"><span data-stu-id="7a509-291">Or, when creating the converter explicitly:</span></span>

<!--
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                var converter = new ValueConverter<EquineBeast, string>(
                    v => v.ToString(),
                    v => (EquineBeast)Enum.Parse(typeof(EquineBeast), v));

                modelBuilder
                    .Entity<Rider>()
                    .Property(e => e.Mount)
                    .HasConversion(converter)
                    .HasMaxLength(20)
                    .IsUnicode(false);
            }
-->
[!code-csharp[ConversionByConverterInstanceWithFacets](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ConversionByConverterInstanceWithFacets)]

<span data-ttu-id="7a509-292">这会在 `varchar(20)` 对 SQL Server 使用 EF Core 迁移时生成列：</span><span class="sxs-lookup"><span data-stu-id="7a509-292">This results in a `varchar(20)` column when using EF Core migrations against SQL Server:</span></span>

```sql
CREATE TABLE [Rider] (
    [Id] int NOT NULL IDENTITY,
    [Mount] varchar(20) NOT NULL,
    CONSTRAINT [PK_Rider] PRIMARY KEY ([Id]));
```

<span data-ttu-id="7a509-293">但是，如果默认情况下所有 `EquineBeast` 列都应为 `varchar(20)` ，则可以将此信息作为的值转换器提供给 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.ConverterMappingHints> 。</span><span class="sxs-lookup"><span data-stu-id="7a509-293">However, if by default all `EquineBeast` columns should be `varchar(20)`, then this information can be given to the value converter as a <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.ConverterMappingHints>.</span></span> <span data-ttu-id="7a509-294">例如：</span><span class="sxs-lookup"><span data-stu-id="7a509-294">For example:</span></span>

<!--
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                var converter = new ValueConverter<EquineBeast, string>(
                    v => v.ToString(),
                    v => (EquineBeast)Enum.Parse(typeof(EquineBeast), v),
                    new ConverterMappingHints(size: 20, unicode: false));

                modelBuilder
                    .Entity<Rider>()
                    .Property(e => e.Mount)
                    .HasConversion(converter);
            }
-->
[!code-csharp[ConversionByConverterInstanceWithMappingHints](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ConversionByConverterInstanceWithMappingHints)]

<span data-ttu-id="7a509-295">现在只要使用此转换器，数据库列就将为非 unicode，最大长度为20。</span><span class="sxs-lookup"><span data-stu-id="7a509-295">Now any time this converter is used, the database column will be non-unicode with a max length of 20.</span></span> <span data-ttu-id="7a509-296">但是，这只是一个提示，因为它们被显式设置在映射属性上的任何 facet 所覆盖。</span><span class="sxs-lookup"><span data-stu-id="7a509-296">However, these are only hints since they are be overridden by any facets explicitly set on the mapped property.</span></span>

## <a name="examples"></a><span data-ttu-id="7a509-297">示例</span><span class="sxs-lookup"><span data-stu-id="7a509-297">Examples</span></span>

### <a name="simple-value-objects"></a><span data-ttu-id="7a509-298">简单值对象</span><span class="sxs-lookup"><span data-stu-id="7a509-298">Simple value objects</span></span>

<span data-ttu-id="7a509-299">此示例使用简单类型来包装基元类型。</span><span class="sxs-lookup"><span data-stu-id="7a509-299">This example uses a simple type to wrap a primitive type.</span></span> <span data-ttu-id="7a509-300">当你希望模型中的类型更具体 (，因此与基元类型相比，更强的类型安全) 时，这会很有用。</span><span class="sxs-lookup"><span data-stu-id="7a509-300">This can be useful when you want the type in your model to be more specific (and hence more type-safe) than a primitive type.</span></span> <span data-ttu-id="7a509-301">在此示例中，该类型为 `Dollars` ，它将包装十进制基元：</span><span class="sxs-lookup"><span data-stu-id="7a509-301">In this example, that type is `Dollars`, which wraps the decimal primitive:</span></span>

<!--
        public readonly struct Dollars
        {
            public Dollars(decimal amount) 
                => Amount = amount;
            
            public decimal Amount { get; }

            public override string ToString() 
                => $"${Amount}";
        }
-->
[!code-csharp[SimpleValueObject](../../../samples/core/Modeling/ValueConversions/SimpleValueObject.cs?name=SimpleValueObject)]

<span data-ttu-id="7a509-302">这可用于实体类型：</span><span class="sxs-lookup"><span data-stu-id="7a509-302">This can be used in an entity type:</span></span>

<!--
        public class Order
        {
            public int Id { get; set; }

            public Dollars Price { get; set; }
        }
-->
[!code-csharp[SimpleValueObjectModel](../../../samples/core/Modeling/ValueConversions/SimpleValueObject.cs?name=SimpleValueObjectModel)]

<span data-ttu-id="7a509-303">并在 `decimal` 存储在数据库中时转换为基础：</span><span class="sxs-lookup"><span data-stu-id="7a509-303">And converted to the underlying `decimal` when stored in the database:</span></span>

<!--
                modelBuilder.Entity<Order>()
                    .Property(e => e.Price)
                    .HasConversion(
                        v => v.Amount,
                        v => new Dollars(v));
-->
[!code-csharp[ConfigureImmutableStructProperty](../../../samples/core/Modeling/ValueConversions/SimpleValueObject.cs?name=ConfigureImmutableStructProperty)]

> [!NOTE]
> <span data-ttu-id="7a509-304">此值对象作为 [readonly 结构](/dotnet/csharp/language-reference/builtin-types/struct)实现。</span><span class="sxs-lookup"><span data-stu-id="7a509-304">This value object is implemented as a [readonly struct](/dotnet/csharp/language-reference/builtin-types/struct).</span></span> <span data-ttu-id="7a509-305">这意味着 EF Core 可以快照和比较值而不会出现问题。</span><span class="sxs-lookup"><span data-stu-id="7a509-305">This means that EF Core can snapshot and compare values without issue.</span></span> <span data-ttu-id="7a509-306">有关详细信息，请参阅 [值](xref:core/modeling/value-comparers) 比较器。</span><span class="sxs-lookup"><span data-stu-id="7a509-306">See [Value Comparers](xref:core/modeling/value-comparers) for more information.</span></span>

### <a name="composite-value-objects"></a><span data-ttu-id="7a509-307">复合值对象</span><span class="sxs-lookup"><span data-stu-id="7a509-307">Composite value objects</span></span>

<span data-ttu-id="7a509-308">在上面的示例中，值对象类型只包含一个属性。</span><span class="sxs-lookup"><span data-stu-id="7a509-308">In the previous example, the value object type contained only a single property.</span></span> <span data-ttu-id="7a509-309">更常见的情况是，值对象类型组成多个属性，共同构成域概念。</span><span class="sxs-lookup"><span data-stu-id="7a509-309">It is more common for a value object type to compose multiple properties that together form a domain concept.</span></span> <span data-ttu-id="7a509-310">例如，一般 `Money` 类型包含数量和货币：</span><span class="sxs-lookup"><span data-stu-id="7a509-310">For example, a general `Money` type that contains both the amount and the currency:</span></span>

<!--
        public readonly struct Money
        {
            [JsonConstructor]
            public Money(decimal amount, Currency currency)
            {
                Amount = amount;
                Currency = currency;
            }

            public override string ToString()
                => (Currency == Currency.UsDollars ? "$" : "£") + Amount;

            public decimal Amount { get; }
            public Currency Currency { get; }
        }

        public enum Currency
        {
            UsDollars,
            PoundsStirling
        }
-->
[!code-csharp[CompositeValueObject](../../../samples/core/Modeling/ValueConversions/CompositeValueObject.cs?name=CompositeValueObject)]

<span data-ttu-id="7a509-311">此值对象可以像以前一样用于实体类型：</span><span class="sxs-lookup"><span data-stu-id="7a509-311">This value object can be used in an entity type as before:</span></span>

<!--
        public class Order
        {
            public int Id { get; set; }

            public Money Price { get; set; }
        }
-->
[!code-csharp[CompositeValueObjectModel](../../../samples/core/Modeling/ValueConversions/CompositeValueObject.cs?name=CompositeValueObjectModel)]

<span data-ttu-id="7a509-312">值转换器当前只能在单个数据库列之间转换值。</span><span class="sxs-lookup"><span data-stu-id="7a509-312">Value converters can currently only convert values to and from a single database column.</span></span> <span data-ttu-id="7a509-313">此限制意味着必须将对象中的所有属性值编码为单个列值。</span><span class="sxs-lookup"><span data-stu-id="7a509-313">This limitation means that all property values from the object must be encoded into a single column value.</span></span> <span data-ttu-id="7a509-314">这通常是通过在对象进入数据库时序列化来处理的，然后再次反序列化该对象。例如，使用 <xref:System.Text.Json> ：</span><span class="sxs-lookup"><span data-stu-id="7a509-314">This is typically handled by serializing the object as it goes into the database, and then deserializing it again on the way out. For example, using <xref:System.Text.Json>:</span></span>

<!--
                modelBuilder.Entity<Order>()
                    .Property(e => e.Price)
                    .HasConversion(
                        v => JsonSerializer.Serialize(v, null),
                        v => JsonSerializer.Deserialize<Money>(v, null));
-->
[!code-csharp[ConfigureCompositeValueObject](../../../samples/core/Modeling/ValueConversions/CompositeValueObject.cs?name=ConfigureCompositeValueObject)]

> [!NOTE]
> <span data-ttu-id="7a509-315">我们计划允许在 EF Core 6.0 中将对象映射到多个列，无需在此处使用序列化。</span><span class="sxs-lookup"><span data-stu-id="7a509-315">We plan to allow mapping an object to multiple columns in EF Core 6.0, removing the need to use serialization here.</span></span> <span data-ttu-id="7a509-316">此 [问题由 GitHub 问题 #13947](https://github.com/dotnet/efcore/issues/13947)跟踪。</span><span class="sxs-lookup"><span data-stu-id="7a509-316">This is tracked by [GitHub issue #13947](https://github.com/dotnet/efcore/issues/13947).</span></span>

> [!NOTE]
> <span data-ttu-id="7a509-317">与前面的示例一样，此值对象作为 [readonly 结构](/dotnet/csharp/language-reference/builtin-types/struct)实现。</span><span class="sxs-lookup"><span data-stu-id="7a509-317">As with the previous example, this value object is implemented as a [readonly struct](/dotnet/csharp/language-reference/builtin-types/struct).</span></span> <span data-ttu-id="7a509-318">这意味着 EF Core 可以快照和比较值而不会出现问题。</span><span class="sxs-lookup"><span data-stu-id="7a509-318">This means that EF Core can snapshot and compare values without issue.</span></span> <span data-ttu-id="7a509-319">有关详细信息，请参阅 [值](xref:core/modeling/value-comparers) 比较器。</span><span class="sxs-lookup"><span data-stu-id="7a509-319">See [Value Comparers](xref:core/modeling/value-comparers) for more information.</span></span>

### <a name="collections-of-primitives"></a><span data-ttu-id="7a509-320">基元集合</span><span class="sxs-lookup"><span data-stu-id="7a509-320">Collections of primitives</span></span>

<span data-ttu-id="7a509-321">序列化还可用于存储基元值的集合。</span><span class="sxs-lookup"><span data-stu-id="7a509-321">Serialization can also be used to store a collection of primitive values.</span></span> <span data-ttu-id="7a509-322">例如：</span><span class="sxs-lookup"><span data-stu-id="7a509-322">For example:</span></span>

<!--
        public class Post
        {
            public int Id { get; set; }
            public string Title { get; set; }
            public string Contents { get; set; }

            public ICollection<string> Tags { get; set; }
        }
-->
[!code-csharp[PrimitiveCollectionModel](../../../samples/core/Modeling/ValueConversions/PrimitiveCollection.cs?name=PrimitiveCollectionModel)]

<span data-ttu-id="7a509-323"><xref:System.Text.Json>再次使用：</span><span class="sxs-lookup"><span data-stu-id="7a509-323">Using <xref:System.Text.Json> again:</span></span>

<!--
                modelBuilder.Entity<Post>()
                    .Property(e => e.Tags)
                    .HasConversion(
                        v => JsonSerializer.Serialize(v, null),
                        v => JsonSerializer.Deserialize<List<string>>(v, null),
                        new ValueComparer<ICollection<string>>(
                            (c1, c2) => c1.SequenceEqual(c2),
                            c => c.Aggregate(0, (a, v) => HashCode.Combine(a, v.GetHashCode())),
                            c => (ICollection<string>)c.ToList()));
-->
[!code-csharp[ConfigurePrimitiveCollection](../../../samples/core/Modeling/ValueConversions/PrimitiveCollection.cs?name=ConfigurePrimitiveCollection)]

<span data-ttu-id="7a509-324">`ICollection<string>` 表示可变引用类型。</span><span class="sxs-lookup"><span data-stu-id="7a509-324">`ICollection<string>` represents a mutable reference type.</span></span> <span data-ttu-id="7a509-325">这意味着， <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601> 需要，以便 EF Core 可以正确地跟踪和检测更改。</span><span class="sxs-lookup"><span data-stu-id="7a509-325">This means that a <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601> is needed so that EF Core can track and detect changes correctly.</span></span> <span data-ttu-id="7a509-326">有关详细信息，请参阅 [值](xref:core/modeling/value-comparers) 比较器。</span><span class="sxs-lookup"><span data-stu-id="7a509-326">See [Value Comparers](xref:core/modeling/value-comparers) for more information.</span></span>

### <a name="collections-of-value-objects"></a><span data-ttu-id="7a509-327">值对象的集合</span><span class="sxs-lookup"><span data-stu-id="7a509-327">Collections of value objects</span></span>

<span data-ttu-id="7a509-328">结合上述两个示例，可以创建值对象的集合。</span><span class="sxs-lookup"><span data-stu-id="7a509-328">Combining the previous two examples together we can create a collection of value objects.</span></span> <span data-ttu-id="7a509-329">例如，请考虑一 `AnnualFinance` 种类型，该类型对一年的博客理财建模：</span><span class="sxs-lookup"><span data-stu-id="7a509-329">For example, consider an `AnnualFinance` type that models blog finances for a single year:</span></span>

<!--
        public readonly struct AnnualFinance
        {
            [JsonConstructor]
            public AnnualFinance(int year, Money income, Money expenses)
            {
                Year = year;
                Income = income;
                Expenses = expenses;
            }

            public int Year { get; }
            public Money Income { get; }
            public Money Expenses { get; }
            public Money Revenue => new Money(Income.Amount - Expenses.Amount, Income.Currency);
        }
-->
[!code-csharp[ValueObjectCollection](../../../samples/core/Modeling/ValueConversions/ValueObjectCollection.cs?name=ValueObjectCollection)]

<span data-ttu-id="7a509-330">此类型撰写了前面创建的几种 `Money` 类型：</span><span class="sxs-lookup"><span data-stu-id="7a509-330">This type composes several of the `Money` types we created previously:</span></span>

<!--
        public readonly struct Money
        {
            [JsonConstructor]
            public Money(decimal amount, Currency currency)
            {
                Amount = amount;
                Currency = currency;
            }

            public override string ToString()
                => (Currency == Currency.UsDollars ? "$" : "£") + Amount;

            public decimal Amount { get; }
            public Currency Currency { get; }
        }

        public enum Currency
        {
            UsDollars,
            PoundsStirling
        }
-->
[!code-csharp[ValueObjectCollectionMoney](../../../samples/core/Modeling/ValueConversions/ValueObjectCollection.cs?name=ValueObjectCollectionMoney)]

<span data-ttu-id="7a509-331">然后，可以将的集合添加 `AnnualFinance` 到实体类型：</span><span class="sxs-lookup"><span data-stu-id="7a509-331">We can then add a collection of `AnnualFinance` to our entity type:</span></span>

<!--
        public class Blog
        {
            public int Id { get; set; }
            public string Name { get; set; }
            
            public IList<AnnualFinance> Finances { get; set; }
        }
-->
[!code-csharp[ValueObjectCollectionModel](../../../samples/core/Modeling/ValueConversions/ValueObjectCollection.cs?name=ValueObjectCollectionModel)]

<span data-ttu-id="7a509-332">并再次使用序列化来存储以下内容：</span><span class="sxs-lookup"><span data-stu-id="7a509-332">And again use serialization to store this:</span></span>

<!--
                modelBuilder.Entity<Blog>()
                    .Property(e => e.Finances)
                    .HasConversion(
                        v => JsonSerializer.Serialize(v, null),
                        v => JsonSerializer.Deserialize<List<AnnualFinance>>(v, null),
                        new ValueComparer<IList<AnnualFinance>>(
                            (c1, c2) => c1.SequenceEqual(c2),
                            c => c.Aggregate(0, (a, v) => HashCode.Combine(a, v.GetHashCode())),
                            c => (IList<AnnualFinance>)c.ToList()));
-->
[!code-csharp[ConfigureValueObjectCollection](../../../samples/core/Modeling/ValueConversions/ValueObjectCollection.cs?name=ConfigureValueObjectCollection)]

> [!NOTE]
> <span data-ttu-id="7a509-333">与之前一样，此转换需要一个 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601> 。</span><span class="sxs-lookup"><span data-stu-id="7a509-333">As before, this conversion requires a <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601>.</span></span> <span data-ttu-id="7a509-334">有关详细信息，请参阅 [值](xref:core/modeling/value-comparers) 比较器。</span><span class="sxs-lookup"><span data-stu-id="7a509-334">See [Value Comparers](xref:core/modeling/value-comparers) for more information.</span></span>

### <a name="value-objects-as-keys"></a><span data-ttu-id="7a509-335">值对象作为键</span><span class="sxs-lookup"><span data-stu-id="7a509-335">Value objects as keys</span></span>

<span data-ttu-id="7a509-336">有时，可以在值对象中包装基元键属性，以在赋值时添加其他级别的类型安全。</span><span class="sxs-lookup"><span data-stu-id="7a509-336">Sometimes primitive key properties may be wrapped in value objects to add an additional level of type-safety in assigning values.</span></span> <span data-ttu-id="7a509-337">例如，我们可以实现博客的密钥类型和帖子的密钥类型：</span><span class="sxs-lookup"><span data-stu-id="7a509-337">For example, we could implement a key type for blogs, and a key type for posts:</span></span>

<!--
        public readonly struct BlogKey
        {
            public BlogKey(int id) => Id = id;
            public int Id { get; }
        }

        public readonly struct PostKey
        {
            public PostKey(int id) => Id = id;
            public int Id { get; }
        }
-->
[!code-csharp[KeyValueObjects](../../../samples/core/Modeling/ValueConversions/KeyValueObjects.cs?name=KeyValueObjects)]

<span data-ttu-id="7a509-338">然后，可以在域模型中使用这些内容：</span><span class="sxs-lookup"><span data-stu-id="7a509-338">These can then be used in the domain model:</span></span>

<!--
        public class Blog
        {
            public BlogKey Id { get; set; }
            public string Name { get; set; }

            public ICollection<Post> Posts { get; set; }
        }

        public class Post
        {
            public PostKey Id { get; set; }

            public string Title { get; set; }
            public string Content { get; set; }

            public BlogKey? BlogId { get; set; }
            public Blog Blog { get; set; }
        }
-->
[!code-csharp[KeyValueObjectsModel](../../../samples/core/Modeling/ValueConversions/KeyValueObjects.cs?name=KeyValueObjectsModel)]

<span data-ttu-id="7a509-339">请注意， `Blog.Id` 不会意外 `PostKey` 地分配，并且 `Post.Id` 不能意外地分配 `BlogKey` 。</span><span class="sxs-lookup"><span data-stu-id="7a509-339">Notice that `Blog.Id` cannot accidentally be assigned a `PostKey`, and `Post.Id` cannot accidentally be assigned a `BlogKey`.</span></span> <span data-ttu-id="7a509-340">同样， `Post.BlogId` 必须为外键属性赋值 `BlogKey` 。</span><span class="sxs-lookup"><span data-stu-id="7a509-340">Similarly, the `Post.BlogId` foreign key property must be assigned a `BlogKey`.</span></span>

> [!NOTE]
> <span data-ttu-id="7a509-341">显示此模式并不意味着我们建议这样做。</span><span class="sxs-lookup"><span data-stu-id="7a509-341">Showing this pattern does not mean we recommend it.</span></span> <span data-ttu-id="7a509-342">请仔细考虑此级别的抽象是否正在帮助或牵制你的开发体验。</span><span class="sxs-lookup"><span data-stu-id="7a509-342">Carefully consider whether this level of abstraction is helping or hampering your development experience.</span></span> <span data-ttu-id="7a509-343">另外，请考虑使用导航和生成的密钥，而不是直接处理键值。</span><span class="sxs-lookup"><span data-stu-id="7a509-343">Also, consider using navigations and generated keys instead of dealing with key values directly.</span></span>

<span data-ttu-id="7a509-344">然后，可以使用值转换器映射这些密钥属性：</span><span class="sxs-lookup"><span data-stu-id="7a509-344">These key properties can then be mapped using value converters:</span></span>

<!--
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                var blogKeyConverter = new ValueConverter<BlogKey, int>(
                    v => v.Id,
                    v => new BlogKey(v));

                modelBuilder.Entity<Blog>().Property(e => e.Id).HasConversion(blogKeyConverter);

                modelBuilder.Entity<Post>(
                    b =>
                        {
                            b.Property(e => e.Id).HasConversion(v => v.Id, v => new PostKey(v));
                            b.Property(e => e.BlogId).HasConversion(blogKeyConverter);
                        });
            }
-->
[!code-csharp[ConfigureKeyValueObjects](../../../samples/core/Modeling/ValueConversions/KeyValueObjects.cs?name=ConfigureKeyValueObjects)]

> [!NOTE]
> <span data-ttu-id="7a509-345">当前带有转换的键属性不能使用生成的键值。</span><span class="sxs-lookup"><span data-stu-id="7a509-345">Currently key properties with conversions cannot use generated key values.</span></span> <span data-ttu-id="7a509-346">对 [GitHub 问题 #11597](https://github.com/dotnet/efcore/issues/11597) 投票，以删除此限制。</span><span class="sxs-lookup"><span data-stu-id="7a509-346">Vote for [GitHub issue #11597](https://github.com/dotnet/efcore/issues/11597) to have this limitation removed.</span></span>

### <a name="use-ulong-for-timestamprowversion"></a><span data-ttu-id="7a509-347">将 ulong 用于 timestamp/rowversion</span><span class="sxs-lookup"><span data-stu-id="7a509-347">Use ulong for timestamp/rowversion</span></span>

<span data-ttu-id="7a509-348">SQL Server 使用[8 字节的二进制 `rowversion` / `timestamp` 列](/sql/t-sql/data-types/rowversion-transact-sql)支持自动[开放式并发](xref:core/saving/concurrency)。</span><span class="sxs-lookup"><span data-stu-id="7a509-348">SQL Server supports automatic [optimistic concurrency](xref:core/saving/concurrency) using [8-byte binary `rowversion`/`timestamp` columns](/sql/t-sql/data-types/rowversion-transact-sql).</span></span> <span data-ttu-id="7a509-349">它们始终使用8字节数组从数据库进行读取和写入。</span><span class="sxs-lookup"><span data-stu-id="7a509-349">These are always read from and written to the database using an 8-byte array.</span></span> <span data-ttu-id="7a509-350">但是，字节数组是可变引用类型，这使得处理起来有些困难。</span><span class="sxs-lookup"><span data-stu-id="7a509-350">However, byte arrays are a mutable reference type, which makes them somewhat painful to deal with.</span></span> <span data-ttu-id="7a509-351">值转换器允许 `rowversion` 改为映射到 `ulong` 属性，该属性比字节数组更合适且更易于使用。</span><span class="sxs-lookup"><span data-stu-id="7a509-351">Value converters allow the `rowversion` to instead be mapped to a `ulong` property, which is much more appropriate and easy to use than the byte array.</span></span> <span data-ttu-id="7a509-352">例如，请考虑一个 `Blog` 具有 ulong 并发令牌的实体：</span><span class="sxs-lookup"><span data-stu-id="7a509-352">For example, consider a `Blog` entity with a ulong concurrency token:</span></span>

<!--
        public class Blog
        {
            public int Id { get; set; }
            public string Name { get; set; }
            public ulong Version { get; set; }
        }
-->
[!code-csharp[ULongConcurrencyModel](../../../samples/core/Modeling/ValueConversions/ULongConcurrency.cs?name=ULongConcurrencyModel)]

<span data-ttu-id="7a509-353">这可以使用值转换器映射到 SQL server `rowversion` 列：</span><span class="sxs-lookup"><span data-stu-id="7a509-353">This can be mapped to a SQL server `rowversion` column using a value converter:</span></span>

<!--
                modelBuilder.Entity<Blog>()
                    .Property(e => e.Version)
                    .IsRowVersion()
                    .HasConversion<byte[]>();
-->
[!code-csharp[ConfigureULongConcurrency](../../../samples/core/Modeling/ValueConversions/ULongConcurrency.cs?name=ConfigureULongConcurrency)]

### <a name="specify-the-datetimekind-when-reading-dates"></a><span data-ttu-id="7a509-354">在读取日期时指定 DateTime. Kind</span><span class="sxs-lookup"><span data-stu-id="7a509-354">Specify the DateTime.Kind when reading dates</span></span>

<span data-ttu-id="7a509-355"><xref:System.DateTime.Kind%2A?displayProperty=nameWithType>将存储为或时，SQL Server 会丢弃标志 <xref:System.DateTime> [`datetime`](/sql/t-sql/data-types/datetime-transact-sql) [`datetime2`](/sql/t-sql/data-types/datetime2-transact-sql) 。</span><span class="sxs-lookup"><span data-stu-id="7a509-355">SQL Server discards the <xref:System.DateTime.Kind%2A?displayProperty=nameWithType> flag when storing a <xref:System.DateTime> as a [`datetime`](/sql/t-sql/data-types/datetime-transact-sql) or [`datetime2`](/sql/t-sql/data-types/datetime2-transact-sql).</span></span> <span data-ttu-id="7a509-356">这意味着，从数据库返回的 DateTime 值始终具有 <xref:System.DateTimeKind> 的 `Unspecified` 。</span><span class="sxs-lookup"><span data-stu-id="7a509-356">This means that DateTime values coming back from the database always have a <xref:System.DateTimeKind> of `Unspecified`.</span></span>

<span data-ttu-id="7a509-357">可以通过两种方式使用值转换器来处理这种情况。</span><span class="sxs-lookup"><span data-stu-id="7a509-357">Value converters can be used in two ways to deal with this.</span></span> <span data-ttu-id="7a509-358">首先，EF Core 具有一个值转换器，该转换器可创建可保留标志的8字节不透明值 `Kind` 。</span><span class="sxs-lookup"><span data-stu-id="7a509-358">First, EF Core has a value converter that creates an 8-byte opaque value which preserves the `Kind` flag.</span></span> <span data-ttu-id="7a509-359">例如：</span><span class="sxs-lookup"><span data-stu-id="7a509-359">For example:</span></span>

<!--
                modelBuilder.Entity<Post>()
                    .Property(e => e.PostedOn)
                    .HasConversion<long>();
-->
[!code-csharp[ConfigurePreserveDateTimeKind1](../../../samples/core/Modeling/ValueConversions/PreserveDateTimeKind.cs?name=ConfigurePreserveDateTimeKind1)]

<span data-ttu-id="7a509-360">这允许 `Kind` 在数据库中混合具有不同标志的日期时间值。</span><span class="sxs-lookup"><span data-stu-id="7a509-360">This allows DateTime values with different `Kind` flags to be mixed in the database.</span></span>

<span data-ttu-id="7a509-361">此方法的问题是数据库不再具有可识别的 `datetime` 或 `datetime2` 列。</span><span class="sxs-lookup"><span data-stu-id="7a509-361">The problem with this approach is that the database no longer has recognizable `datetime` or `datetime2` columns.</span></span> <span data-ttu-id="7a509-362">通常，通常情况下，通常会存储 UTC 时间 (或者，不太常见，总是本地时间) ，然后 `Kind` 使用值转换器忽略标志或将其设置为合适的值。</span><span class="sxs-lookup"><span data-stu-id="7a509-362">So instead it is common to always store UTC time (or, less commonly, always local time) and then either ignore the `Kind` flag or set it to the appropriate value using a value converter.</span></span> <span data-ttu-id="7a509-363">例如，下面的转换器确保 `DateTime` 从数据库中读取的值具有 <xref:System.DateTimeKind> `UTC` ：</span><span class="sxs-lookup"><span data-stu-id="7a509-363">For example, the converter below ensures that the `DateTime` value read from the database will have the <xref:System.DateTimeKind> `UTC`:</span></span>

<!--
                modelBuilder.Entity<Post>()
                    .Property(e => e.LastUpdated)
                    .HasConversion(
                        v => v,
                        v => new DateTime(v.Ticks, DateTimeKind.Utc));
-->
[!code-csharp[ConfigurePreserveDateTimeKind2](../../../samples/core/Modeling/ValueConversions/PreserveDateTimeKind.cs?name=ConfigurePreserveDateTimeKind2)]

<span data-ttu-id="7a509-364">如果在实体实例中设置了局部值和 UTC 值的组合，则在插入前可以使用转换器进行适当的转换。</span><span class="sxs-lookup"><span data-stu-id="7a509-364">If a mix of local and UTC values are being set in entity instances, then the converter can be used to convert appropriately before inserting.</span></span> <span data-ttu-id="7a509-365">例如：</span><span class="sxs-lookup"><span data-stu-id="7a509-365">For example:</span></span>

<!--
                modelBuilder.Entity<Post>()
                    .Property(e => e.LastUpdated)
                    .HasConversion(
                        v => v.ToUniversalTime(),
                        v => new DateTime(v.Ticks, DateTimeKind.Utc));
-->
[!code-csharp[ConfigurePreserveDateTimeKind3](../../../samples/core/Modeling/ValueConversions/PreserveDateTimeKind.cs?name=ConfigurePreserveDateTimeKind3)]

> [!NOTE]
> <span data-ttu-id="7a509-366">仔细考虑统一所有数据库访问代码以始终使用 UTC 时间，只需处理向用户呈现数据的本地时间。</span><span class="sxs-lookup"><span data-stu-id="7a509-366">Carefully consider unifying all database access code to use UTC time all the time, only dealing with local time when presenting data to users.</span></span>

### <a name="use-case-insensitive-string-keys"></a><span data-ttu-id="7a509-367">使用不区分大小写的字符串密钥</span><span class="sxs-lookup"><span data-stu-id="7a509-367">Use case-insensitive string keys</span></span>

<span data-ttu-id="7a509-368">默认情况下，某些数据库（包括 SQL Server）执行不区分大小写的字符串比较。</span><span class="sxs-lookup"><span data-stu-id="7a509-368">Some databases, including SQL Server, perform case-insensitive string comparisons by default.</span></span> <span data-ttu-id="7a509-369">另一方面，.NET 默认执行区分大小写的字符串比较。</span><span class="sxs-lookup"><span data-stu-id="7a509-369">.NET, on the other hand, performs case-sensitive string comparisons by default.</span></span> <span data-ttu-id="7a509-370">这意味着外键值（如 "DotNet"）将匹配 SQL Server 上的主键值 "DotNet"，但不会在 EF Core 中匹配。</span><span class="sxs-lookup"><span data-stu-id="7a509-370">This means that a foreign key value like "DotNet" will match the primary key value "dotnet" on SQL Server, but will not match it in EF Core.</span></span> <span data-ttu-id="7a509-371">键的值比较器可用于强制 EF Core 为类似于数据库中的不区分大小写的字符串比较。</span><span class="sxs-lookup"><span data-stu-id="7a509-371">A value comparer for keys can be used to force EF Core into case-insensitive string comparisons like in the database.</span></span> <span data-ttu-id="7a509-372">例如，请考虑使用字符串键的博客/文章模型：</span><span class="sxs-lookup"><span data-stu-id="7a509-372">For example, consider a blog/posts model with string keys:</span></span>

<!--
        public class Blog
        {
            public string Id { get; set; }
            public string Name { get; set; }

            public ICollection<Post> Posts { get; set; }
        }

        public class Post
        {
            public string Id { get; set; }
            public string Title { get; set; }
            public string Content { get; set; }

            public string BlogId { get; set; }
            public Blog Blog { get; set; }
        }
-->
[!code-csharp[CaseInsensitiveStringsModel](../../../samples/core/Modeling/ValueConversions/CaseInsensitiveStrings.cs?name=CaseInsensitiveStringsModel)]

<span data-ttu-id="7a509-373">如果某些 `Post.BlogId` 值具有不同的大小写，则此操作不会按预期方式工作。</span><span class="sxs-lookup"><span data-stu-id="7a509-373">This will not work as expected if some of the `Post.BlogId` values have different casing.</span></span> <span data-ttu-id="7a509-374">导致这种错误的原因将取决于应用程序正在执行的操作，但通常涉及未正确 [固定](xref:core/change-tracking/relationship-changes) 的对象的图形，以及/或由于 FK 值错误而导致失败的更新。</span><span class="sxs-lookup"><span data-stu-id="7a509-374">The errors caused by this will depend on what the application is doing, but typically involve graphs of objects that are not [fixed-up](xref:core/change-tracking/relationship-changes) correctly, and/or updates that fail because the FK value is wrong.</span></span> <span data-ttu-id="7a509-375">值比较器可用于更正此错误：</span><span class="sxs-lookup"><span data-stu-id="7a509-375">A value comparer can be used to correct this:</span></span>

<!--
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                var comparer = new ValueComparer<string>(
                    (l, r) => string.Equals(l, r, StringComparison.OrdinalIgnoreCase),
                    v => v.ToUpper().GetHashCode(),
                    v => v);

                modelBuilder.Entity<Blog>()
                    .Property(e => e.Id)
                    .Metadata.SetValueComparer(comparer);

                modelBuilder.Entity<Post>(
                    b =>
                        {
                            b.Property(e => e.Id).Metadata.SetValueComparer(comparer);
                            b.Property(e => e.BlogId).Metadata.SetValueComparer(comparer);
                        });
            }
-->
[!code-csharp[ConfigureCaseInsensitiveStrings](../../../samples/core/Modeling/ValueConversions/CaseInsensitiveStrings.cs?name=ConfigureCaseInsensitiveStrings)]

> [!NOTE]
> <span data-ttu-id="7a509-376">除了区分大小写，.NET 字符串比较和数据库字符串比较可能会有所不同。</span><span class="sxs-lookup"><span data-stu-id="7a509-376">.NET string comparisons and database string comparisons can differ in more than just case sensitivity.</span></span> <span data-ttu-id="7a509-377">此模式适用于简单的 ASCII 键，但对于任何类型的区域性特定字符，密钥可能会失败。</span><span class="sxs-lookup"><span data-stu-id="7a509-377">This pattern works for simple ASCII keys, but may fail for keys with any kind of culture-specific characters.</span></span> <span data-ttu-id="7a509-378">有关详细信息，请参阅 [排序规则和区分大小写](xref:core/miscellaneous/collations-and-case-sensitivity) 。</span><span class="sxs-lookup"><span data-stu-id="7a509-378">See [Collations and Case Sensitivity](xref:core/miscellaneous/collations-and-case-sensitivity) for more information.</span></span>

### <a name="handle-fixed-length-database-strings"></a><span data-ttu-id="7a509-379">处理固定长度的数据库字符串</span><span class="sxs-lookup"><span data-stu-id="7a509-379">Handle fixed-length database strings</span></span>

<span data-ttu-id="7a509-380">前面的示例不需要值转换器。</span><span class="sxs-lookup"><span data-stu-id="7a509-380">The previous example did not need a value converter.</span></span> <span data-ttu-id="7a509-381">但是，转换器可用于固定长度的数据库字符串类型（如或） `char(20)` `nchar(20)` 。</span><span class="sxs-lookup"><span data-stu-id="7a509-381">However, a converter can be useful for fixed-length database string types like `char(20)` or `nchar(20)`.</span></span> <span data-ttu-id="7a509-382">只要将值插入到数据库中，就会将固定长度的字符串填充到其完整长度。</span><span class="sxs-lookup"><span data-stu-id="7a509-382">Fixed-length strings are padded to their full length whenever a value is inserted into the database.</span></span> <span data-ttu-id="7a509-383">这意味着，将 "" 的键值 `dotnet` 作为 "" 读回数据库 `dotnet..............` ，其中 `.` 表示空格字符。</span><span class="sxs-lookup"><span data-stu-id="7a509-383">This means that a key value of "`dotnet`" will be read back from the database as "`dotnet..............`", where `.` represents a space character.</span></span> <span data-ttu-id="7a509-384">这样，就不会正确地将未填充的键值进行比较。</span><span class="sxs-lookup"><span data-stu-id="7a509-384">This will then not compare correctly with key values that are not padded.</span></span>

<span data-ttu-id="7a509-385">值转换器可用于在读取键值时剪裁填充。</span><span class="sxs-lookup"><span data-stu-id="7a509-385">A value converter can be used to trim the padding when reading key values.</span></span> <span data-ttu-id="7a509-386">这可以与上一示例中的值比较器结合，以正确比较固定长度不区分大小写的 ASCII 键。</span><span class="sxs-lookup"><span data-stu-id="7a509-386">This can be combined with the value comparer in the previous example to compare fixed length case-insensitive ASCII keys correctly.</span></span> <span data-ttu-id="7a509-387">例如：</span><span class="sxs-lookup"><span data-stu-id="7a509-387">For example:</span></span>

<!--
            protected override void OnModelCreating(ModelBuilder modelBuilder)
            {
                var converter = new ValueConverter<string, string>(
                    v => v,
                    v => v.Trim());
                
                var comparer = new ValueComparer<string>(
                    (l, r) => string.Equals(l, r, StringComparison.OrdinalIgnoreCase),
                    v => v.ToUpper().GetHashCode(),
                    v => v);

                modelBuilder.Entity<Blog>()
                    .Property(e => e.Id)
                    .HasColumnType("char(20)")
                    .HasConversion(converter, comparer);

                modelBuilder.Entity<Post>(
                    b =>
                        {
                            b.Property(e => e.Id).HasColumnType("char(20)").HasConversion(converter, comparer);
                            b.Property(e => e.BlogId).HasColumnType("char(20)").HasConversion(converter, comparer);
                        });
            }
-->
[!code-csharp[ConfigureFixedLengthStrings](../../../samples/core/Modeling/ValueConversions/FixedLengthStrings.cs?name=ConfigureFixedLengthStrings)]

### <a name="encrypt-property-values"></a><span data-ttu-id="7a509-388">加密属性值</span><span class="sxs-lookup"><span data-stu-id="7a509-388">Encrypt property values</span></span>

<span data-ttu-id="7a509-389">值转换器可用于在将属性值发送到数据库之前对其进行加密，然后将其解密。例如，使用字符串反转替代实加密算法：</span><span class="sxs-lookup"><span data-stu-id="7a509-389">Value converters can be used to encrypt property values before sending them to the database, and then decrypt them on the way out. For example, using string reversal as a substitute for a real encryption algorithm:</span></span>

<!--
                modelBuilder.Entity<User>().Property(e => e.Password).HasConversion(
                    v => new string(v.Reverse().ToArray()),
                    v => new string(v.Reverse().ToArray()));
-->
[!code-csharp[ConfigureEncryptPropertyValues](../../../samples/core/Modeling/ValueConversions/EncryptPropertyValues.cs?name=ConfigureEncryptPropertyValues)]

> [!NOTE]
> <span data-ttu-id="7a509-390">目前没有任何方法可以从值转换器内获取对当前 DbContext 或其他会话状态的引用。</span><span class="sxs-lookup"><span data-stu-id="7a509-390">There is currently no way to get a reference to the current DbContext, or other session state, from within a value converter.</span></span> <span data-ttu-id="7a509-391">这限制了可以使用的加密类型。</span><span class="sxs-lookup"><span data-stu-id="7a509-391">This limits the kinds of encryption that can be used.</span></span> <span data-ttu-id="7a509-392">对 [GitHub 问题 #11597](https://github.com/dotnet/efcore/issues/12205) 投票，以删除此限制。</span><span class="sxs-lookup"><span data-stu-id="7a509-392">Vote for [GitHub issue #11597](https://github.com/dotnet/efcore/issues/12205) to have this limitation removed.</span></span>

> [!WARNING]
> <span data-ttu-id="7a509-393">如果你滚动自己的加密以保护敏感数据，请确保了解所有影响。</span><span class="sxs-lookup"><span data-stu-id="7a509-393">Make sure to understand all the implications if you roll your own encryption to protect sensitive data.</span></span> <span data-ttu-id="7a509-394">请考虑使用预先生成的加密机制，如 SQL Server 上的 [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) 。</span><span class="sxs-lookup"><span data-stu-id="7a509-394">Consider instead using pre-built encryption mechanisms, such as [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) on SQL Server.</span></span>

## <a name="limitations"></a><span data-ttu-id="7a509-395">限制</span><span class="sxs-lookup"><span data-stu-id="7a509-395">Limitations</span></span>

<span data-ttu-id="7a509-396">值转换系统存在一些已知的当前限制：</span><span class="sxs-lookup"><span data-stu-id="7a509-396">There are a few known current limitations of the value conversion system:</span></span>

* <span data-ttu-id="7a509-397">当前无法在一个位置指定给定类型的每个属性都必须使用相同的值转换器。</span><span class="sxs-lookup"><span data-stu-id="7a509-397">There is currently no way to specify in one place that every property of a given type must use the same value converter.</span></span> <span data-ttu-id="7a509-398">请 (👍) [GitHub 问题](https://github.com/dotnet/efcore/issues/10784) 的投票 #10784 如果这是你需要的东西。</span><span class="sxs-lookup"><span data-stu-id="7a509-398">Please vote (👍) for [GitHub issue #10784](https://github.com/dotnet/efcore/issues/10784) if this is something you need.</span></span>
* <span data-ttu-id="7a509-399">如上所述， `null` 无法转换。</span><span class="sxs-lookup"><span data-stu-id="7a509-399">As noted above, `null` cannot be converted.</span></span> <span data-ttu-id="7a509-400">请 (👍) [GitHub 问题](https://github.com/dotnet/efcore/issues/13850) 的投票 #13850 如果这是你需要的东西。</span><span class="sxs-lookup"><span data-stu-id="7a509-400">Please vote (👍) for [GitHub issue #13850](https://github.com/dotnet/efcore/issues/13850) if this is something you need.</span></span>
* <span data-ttu-id="7a509-401">目前没有办法将一个属性转换为多个列，反之亦然。</span><span class="sxs-lookup"><span data-stu-id="7a509-401">There is currently no way to spread a conversion of one property to multiple columns or vice-versa.</span></span> <span data-ttu-id="7a509-402">请 (👍) [GitHub 问题](https://github.com/dotnet/efcore/issues/13947) 的投票 #13947 如果这是你需要的东西。</span><span class="sxs-lookup"><span data-stu-id="7a509-402">Please vote (👍) for [GitHub issue #13947](https://github.com/dotnet/efcore/issues/13947) if this is something you need.</span></span>
* <span data-ttu-id="7a509-403">对于通过值转换器映射的大多数密钥，不支持值生成。</span><span class="sxs-lookup"><span data-stu-id="7a509-403">Value generation is not supported for most keys mapped through value converters.</span></span> <span data-ttu-id="7a509-404">请 (👍) [GitHub 问题](https://github.com/dotnet/efcore/issues/11597) 的投票 #11597 如果这是你需要的东西。</span><span class="sxs-lookup"><span data-stu-id="7a509-404">Please vote (👍) for [GitHub issue #11597](https://github.com/dotnet/efcore/issues/11597) if this is something you need.</span></span>
* <span data-ttu-id="7a509-405">值转换无法引用当前的 DbContext 实例。</span><span class="sxs-lookup"><span data-stu-id="7a509-405">Value conversions cannot reference the current DbContext instance.</span></span> <span data-ttu-id="7a509-406">请 (👍) [GitHub 问题](https://github.com/dotnet/efcore/issues/12205) 的投票 #11597 如果这是你需要的东西。</span><span class="sxs-lookup"><span data-stu-id="7a509-406">Please vote (👍) for [GitHub issue #11597](https://github.com/dotnet/efcore/issues/12205) if this is something you need.</span></span>

<span data-ttu-id="7a509-407">将来的版本将考虑删除这些限制。</span><span class="sxs-lookup"><span data-stu-id="7a509-407">Removal of these limitations is being considered for future releases.</span></span>
