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
# <a name="value-conversions"></a>值转换

值转换器允许在读取或写入数据库时转换属性值。 此转换可以从一个值转换为同一类型的另一个值 (例如，将字符串) 或从一种类型的值加密为另一种类型的值 (例如，在数据库中将枚举值与字符串相互转换。 ) 

> [!TIP]
> 通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/ValueConversions/)，你可运行并调试到本文档中的所有代码。

## <a name="overview"></a>概述

值转换器以和的形式指定 `ModelClrType` `ProviderClrType` 。 模型类型是实体类型中的属性的 .NET 类型。 提供程序类型是数据库提供程序理解的 .NET 类型。 例如，若要将枚举作为字符串保存在数据库中，模型类型是枚举的类型，而提供程序类型为 `String` 。 这两种类型可以相同。

使用两个 `Func` 表达式树来定义转换：一个从 `ModelClrType` 到 `ProviderClrType` ，另一个由 `ProviderClrType` 到 `ModelClrType` 。 使用表达式树，以便可以将它们编译到数据库访问委托中以便进行有效的转换。 对于复杂转换，表达式树可能包含对转换方法的简单调用。

> [!NOTE]
> 为值转换配置的属性可能还需要指定 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601> 。 有关详细信息，请参阅下面的示例和 [值](xref:core/modeling/value-comparers) 比较器文档。

## <a name="configuring-a-value-converter"></a>配置值转换器

值转换在中进行配置 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A?displayProperty=nameWithType> 。 例如，假设枚举和实体类型定义为：

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

在中，可以将转换配置 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> 为在数据库中存储枚举值（如 "Donkey"、"Mule" 等），只需提供一个将从转换 `ModelClrType` 为的函数 `ProviderClrType` ，另一个函数用于相反转换：

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
> `null`值绝不会传递到值转换器。 数据库列中的 null 在实体实例中始终为 null，反之亦然。 这使得转换的实现变得更简单，并使其能够在可以为 null 和不可为 null 的属性之间共享。 有关详细信息，请参阅 [GitHub 问题 #13850](https://github.com/dotnet/efcore/issues/13850) 。

## <a name="pre-defined-conversions"></a>预定义的转换

EF Core 包含许多预定义的转换，这些转换可避免手动编写转换函数。 相反，EF Core 会根据模型中的属性类型和请求的数据库提供程序类型选取要使用的转换。

例如，枚举到字符串的转换用作上面的示例，但在将提供程序类型配置为使用的泛型类型时，EF Core 实际上会自动执行此 `string` <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.HasConversion%2A> 操作：

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

可以通过显式指定数据库列类型来实现相同的目的。 例如，如果定义了实体类型，如下所示：

### <a name="data-annotations"></a>[数据批注](#tab/data-annotations)

<!--
        public class Rider2
        {
            public int Id { get; set; }

            [Column(TypeName = "nvarchar(24)")]
            public EquineBeast Mount { get; set; }
        }
-->
[!code-csharp[ConversionByDatabaseType](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ConversionByDatabaseType)]

### <a name="fluent-api"></a>[Fluent API](#tab/fluent-api)

<!--
                modelBuilder
                    .Entity<Rider2>()
                    .Property(e => e.Mount)
                    .HasColumnType("nvarchar(24)");
-->
[!code-csharp[ConversionByDatabaseTypeFluent](../../../samples/core/Modeling/ValueConversions/EnumToStringConversions.cs?name=ConversionByDatabaseTypeFluent)]

***

然后，枚举值将作为字符串保存在数据库中，而不会在中进行任何进一步的配置 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> 。

## <a name="the-valueconverter-class"></a>ValueConverter 类

<xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.HasConversion%2A>如上所述调用会创建一个 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.ValueConverter%602> 实例，并在属性中设置该实例。 `ValueConverter`可以显式创建。 例如：

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

当多个属性使用相同的转换时，这可能很有用。

## <a name="built-in-converters"></a>内置转换器

如上所述，EF Core 附带一组预定义的 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.ValueConverter%602> 类，这些类位于 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion> 命名空间中。 在许多情况下，EF 会基于模型中的属性类型和数据库中所请求的类型选择适当的内置转换器，如上面的枚举所示。 例如， `.HasConversion<int>()` 对 `bool` 属性使用会导致 EF Core 将布尔值转换为数字零和一个值：

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

此功能与创建内置的实例 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToZeroOneConverter%601> 并显式设置它的功能相同：

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

下表汇总了从模型/属性类型到数据库提供程序类型的常用预定义转换。 在表中 `any_numeric_type` ，是指 `int` 、、、 `short` `long` `byte` 、 `uint` 、 `ushort` `ulong` `sbyte` `char` `decimal` `float` `double` 、、、、、或中的一个。

| 模型/属性类型 | 提供程序/数据库类型 | 转换                                                | 使用情况
|:--------------------|------------------------|-----------------------------------------------------------|------
| bool                | any_numeric_type       | False/true 到0/1                                         | `.HasConversion<any_numeric_type>()`
|                     | any_numeric_type       | 对于任意两个数值为 False/true                             | 使用 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToTwoValuesConverter%601>
|                     | 字符串                 | False/true 到 "Y"/"N"                                     | `.HasConversion<string>()`
|                     | 字符串                 | 对于任意两个字符串为 False/true                             | 使用 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToStringConverter>
| any_numeric_type    | bool                   | 0/1 到 false/true                                         | `.HasConversion<bool>()`
|                     | any_numeric_type       | 简单强制转换                                               | `.HasConversion<any_numeric_type>()`
|                     | 字符串                 | 字符串形式的数字                                    | `.HasConversion<string>()`
| 枚举                | any_numeric_type       | 枚举的数值                             | `.HasConversion<any_numeric_type>()`
|                     | 字符串                 | 枚举值的字符串表示形式               | `.HasConversion<string>()`
| 字符串              | bool                   | 将字符串分析为布尔型                               | `.HasConversion<bool>()`
|                     | any_numeric_type       | 将字符串分析为给定的数值类型               | `.HasConversion<any_numeric_type>()`
|                     | char                   | 字符串的第一个字符                         | `.HasConversion<char>()`
|                     | DateTime               | 将字符串分析为日期时间                           | `.HasConversion<DateTime>()`
|                     | DateTimeOffset         | 将字符串分析为 DateTimeOffset                     | `.HasConversion<DateTimeOffset>()`
|                     | TimeSpan               | 将字符串分析为 TimeSpan                           | `.HasConversion<TimeSpan>()`
|                     | Guid                   | 将字符串分析为 Guid                               | `.HasConversion<Guid>()`
|                     | byte[]                 | UTF8 字节形式的字符串                                  | `.HasConversion<byte[]>()`
| char                | string                 | 单个字符串                                 | `.HasConversion<string>()`
| DateTime            | long                   | 编码日期/时间保留日期时间类型                | `.HasConversion<long>()`
|                     | long                   | 刻度                                                     | 使用 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToTicksConverter>
|                     | 字符串                 | 固定区域性日期/时间字符串                        | `.HasConversion<string>()`
| DateTimeOffset      | long                   | 带偏移量的编码日期/时间                             | `.HasConversion<long>()`
|                     | 字符串                 | 具有偏移量的固定区域性日期/时间字符串            | `.HasConversion<string>()`
| TimeSpan            | long                   | 刻度                                                     | `.HasConversion<long>()`
|                     | 字符串                 | 固定区域性时间跨度字符串                        | `.HasConversion<string>()`
| URI                 | string                 | 字符串形式的 URI                                       | `.HasConversion<string>()`
| PhysicalAddress     | 字符串                 | 字符串形式的地址                                   | `.HasConversion<string>()`
|                     | byte[]                 | 大字节序网络顺序中的字节数                         | `.HasConversion<byte[]>()`
| IPAddress           | 字符串                 | 字符串形式的地址                                   | `.HasConversion<string>()`
|                     | byte[]                 | 大字节序网络顺序中的字节数                         | `.HasConversion<byte[]>()`
| Guid                | 字符串                 | 采用 "dddddddd-dddd-dddd-dddd-dddddddddddd" 格式的 GUID | `.HasConversion<string>()`
|                     | byte[]                 | .NET 二进制序列化顺序中的字节数                  | `.HasConversion<byte[]>()`

请注意，这些转换假定值的格式适用于转换。 例如，如果字符串值无法分析为数字，将字符串转换为数字将会失败。

内置转换器的完整列表如下：

* 转换 bool 属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToStringConverter> -Bool 到字符串（如 "Y" 和 "N"）
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToTwoValuesConverter%601> -布尔值到任意两个值
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BoolToZeroOneConverter%601> -布尔值为零和一个
* 转换字节数组属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.BytesToStringConverter> -字节数组到 Base64 编码的字符串
* 任何只需要类型强制转换的转换
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.CastingConverter%602> -只需要类型强制转换的转换
* 转换 char 属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.CharToStringConverter> -Char 到单字符字符串
* 转换 <xref:System.DateTimeOffset> 属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeOffsetToBinaryConverter> - <xref:System.DateTimeOffset> 到二进制编码的64位值
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeOffsetToBytesConverter> - <xref:System.DateTimeOffset> 到字节数组
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeOffsetToStringConverter> - <xref:System.DateTimeOffset> 到字符串
* 转换 <xref:System.DateTime> 属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToBinaryConverter> - <xref:System.DateTime> 到64位值（包括 Datetimekind.utc）
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToStringConverter> - <xref:System.DateTime> 到字符串
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.DateTimeToTicksConverter> - <xref:System.DateTime> 到刻度
* 转换枚举属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.EnumToNumberConverter%602> -枚举到基础数字
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.EnumToStringConverter%601> -枚举到字符串
* 转换 <xref:System.Guid> 属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.GuidToBytesConverter> - <xref:System.Guid> 到字节数组
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.GuidToStringConverter> - <xref:System.Guid> 到字符串
* 转换 <xref:System.Net.IPAddress> 属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.IPAddressToBytesConverter> - <xref:System.Net.IPAddress> 到字节数组
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.IPAddressToStringConverter> - <xref:System.Net.IPAddress> 到字符串
* 将数值转换 (int、double、decimal 等 ) 属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.NumberToBytesConverter%601> -任何数值到字节数组
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.NumberToStringConverter%601> -任何数值到字符串
* 转换 <xref:System.Net.NetworkInformation.PhysicalAddress> 属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.PhysicalAddressToBytesConverter> - <xref:System.Net.NetworkInformation.PhysicalAddress> 到字节数组
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.PhysicalAddressToStringConverter> - <xref:System.Net.NetworkInformation.PhysicalAddress> 到字符串
* 转换字符串属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToBoolConverter> -字符串（例如 "Y" 和 "N"）到 bool
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToBytesConverter> -字符串到 UTF8 字节
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToCharConverter> -字符串到字符
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToDateTimeConverter> -字符串到 <xref:System.DateTime>
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToDateTimeOffsetConverter> -字符串到 <xref:System.DateTimeOffset>
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToEnumConverter%601> -要枚举的字符串
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToGuidConverter> -字符串到 <xref:System.Guid>
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToNumberConverter%601> -字符串到数值类型
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToTimeSpanConverter> -字符串到 <xref:System.TimeSpan>
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.StringToUriConverter> -字符串到 <xref:System.Uri>
* 转换 <xref:System.TimeSpan> 属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.TimeSpanToStringConverter> - <xref:System.TimeSpan> 到字符串
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.TimeSpanToTicksConverter> - <xref:System.TimeSpan> 到刻度
* 转换 <xref:System.Uri> 属性：
  * <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.UriToStringConverter> - <xref:System.Uri> 到字符串

请注意，所有内置的转换器都是无状态的，因此，多个属性可以安全地共享单个实例。

## <a name="column-facets-and-mapping-hints"></a>列 facet 和映射提示

某些数据库类型具有用于修改数据存储方式的方面。 其中包括:

* "小数位数" 和 "日期/时间" 列的精度和小数位数
* 二进制和字符串列的大小/长度
* 字符串列的 Unicode

对于使用值转换器的属性，可以按正常方式配置这些层面，并将其应用于转换后的数据库类型。 例如，在将枚举转换为字符串时，可以指定数据库列应为非 Unicode，并且最多可存储20个字符：

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

或者，显式创建转换器：

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

这会在 `varchar(20)` 对 SQL Server 使用 EF Core 迁移时生成列：

```sql
CREATE TABLE [Rider] (
    [Id] int NOT NULL IDENTITY,
    [Mount] varchar(20) NOT NULL,
    CONSTRAINT [PK_Rider] PRIMARY KEY ([Id]));
```

但是，如果默认情况下所有 `EquineBeast` 列都应为 `varchar(20)` ，则可以将此信息作为的值转换器提供给 <xref:Microsoft.EntityFrameworkCore.Storage.ValueConversion.ConverterMappingHints> 。 例如：

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

现在只要使用此转换器，数据库列就将为非 unicode，最大长度为20。 但是，这只是一个提示，因为它们被显式设置在映射属性上的任何 facet 所覆盖。

## <a name="examples"></a>示例

### <a name="simple-value-objects"></a>简单值对象

此示例使用简单类型来包装基元类型。 当你希望模型中的类型更具体 (，因此与基元类型相比，更强的类型安全) 时，这会很有用。 在此示例中，该类型为 `Dollars` ，它将包装十进制基元：

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

这可用于实体类型：

<!--
        public class Order
        {
            public int Id { get; set; }

            public Dollars Price { get; set; }
        }
-->
[!code-csharp[SimpleValueObjectModel](../../../samples/core/Modeling/ValueConversions/SimpleValueObject.cs?name=SimpleValueObjectModel)]

并在 `decimal` 存储在数据库中时转换为基础：

<!--
                modelBuilder.Entity<Order>()
                    .Property(e => e.Price)
                    .HasConversion(
                        v => v.Amount,
                        v => new Dollars(v));
-->
[!code-csharp[ConfigureImmutableStructProperty](../../../samples/core/Modeling/ValueConversions/SimpleValueObject.cs?name=ConfigureImmutableStructProperty)]

> [!NOTE]
> 此值对象作为 [readonly 结构](/dotnet/csharp/language-reference/builtin-types/struct)实现。 这意味着 EF Core 可以快照和比较值而不会出现问题。 有关详细信息，请参阅 [值](xref:core/modeling/value-comparers) 比较器。

### <a name="composite-value-objects"></a>复合值对象

在上面的示例中，值对象类型只包含一个属性。 更常见的情况是，值对象类型组成多个属性，共同构成域概念。 例如，一般 `Money` 类型包含数量和货币：

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

此值对象可以像以前一样用于实体类型：

<!--
        public class Order
        {
            public int Id { get; set; }

            public Money Price { get; set; }
        }
-->
[!code-csharp[CompositeValueObjectModel](../../../samples/core/Modeling/ValueConversions/CompositeValueObject.cs?name=CompositeValueObjectModel)]

值转换器当前只能在单个数据库列之间转换值。 此限制意味着必须将对象中的所有属性值编码为单个列值。 这通常是通过在对象进入数据库时序列化来处理的，然后再次反序列化该对象。例如，使用 <xref:System.Text.Json> ：

<!--
                modelBuilder.Entity<Order>()
                    .Property(e => e.Price)
                    .HasConversion(
                        v => JsonSerializer.Serialize(v, null),
                        v => JsonSerializer.Deserialize<Money>(v, null));
-->
[!code-csharp[ConfigureCompositeValueObject](../../../samples/core/Modeling/ValueConversions/CompositeValueObject.cs?name=ConfigureCompositeValueObject)]

> [!NOTE]
> 我们计划允许在 EF Core 6.0 中将对象映射到多个列，无需在此处使用序列化。 此 [问题由 GitHub 问题 #13947](https://github.com/dotnet/efcore/issues/13947)跟踪。

> [!NOTE]
> 与前面的示例一样，此值对象作为 [readonly 结构](/dotnet/csharp/language-reference/builtin-types/struct)实现。 这意味着 EF Core 可以快照和比较值而不会出现问题。 有关详细信息，请参阅 [值](xref:core/modeling/value-comparers) 比较器。

### <a name="collections-of-primitives"></a>基元集合

序列化还可用于存储基元值的集合。 例如：

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

<xref:System.Text.Json>再次使用：

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

`ICollection<string>` 表示可变引用类型。 这意味着， <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601> 需要，以便 EF Core 可以正确地跟踪和检测更改。 有关详细信息，请参阅 [值](xref:core/modeling/value-comparers) 比较器。

### <a name="collections-of-value-objects"></a>值对象的集合

结合上述两个示例，可以创建值对象的集合。 例如，请考虑一 `AnnualFinance` 种类型，该类型对一年的博客理财建模：

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

此类型撰写了前面创建的几种 `Money` 类型：

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

然后，可以将的集合添加 `AnnualFinance` 到实体类型：

<!--
        public class Blog
        {
            public int Id { get; set; }
            public string Name { get; set; }
            
            public IList<AnnualFinance> Finances { get; set; }
        }
-->
[!code-csharp[ValueObjectCollectionModel](../../../samples/core/Modeling/ValueConversions/ValueObjectCollection.cs?name=ValueObjectCollectionModel)]

并再次使用序列化来存储以下内容：

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
> 与之前一样，此转换需要一个 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601> 。 有关详细信息，请参阅 [值](xref:core/modeling/value-comparers) 比较器。

### <a name="value-objects-as-keys"></a>值对象作为键

有时，可以在值对象中包装基元键属性，以在赋值时添加其他级别的类型安全。 例如，我们可以实现博客的密钥类型和帖子的密钥类型：

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

然后，可以在域模型中使用这些内容：

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

请注意， `Blog.Id` 不会意外 `PostKey` 地分配，并且 `Post.Id` 不能意外地分配 `BlogKey` 。 同样， `Post.BlogId` 必须为外键属性赋值 `BlogKey` 。

> [!NOTE]
> 显示此模式并不意味着我们建议这样做。 请仔细考虑此级别的抽象是否正在帮助或牵制你的开发体验。 另外，请考虑使用导航和生成的密钥，而不是直接处理键值。

然后，可以使用值转换器映射这些密钥属性：

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
> 当前带有转换的键属性不能使用生成的键值。 对 [GitHub 问题 #11597](https://github.com/dotnet/efcore/issues/11597) 投票，以删除此限制。

### <a name="use-ulong-for-timestamprowversion"></a>将 ulong 用于 timestamp/rowversion

SQL Server 使用[8 字节的二进制 `rowversion` / `timestamp` 列](/sql/t-sql/data-types/rowversion-transact-sql)支持自动[开放式并发](xref:core/saving/concurrency)。 它们始终使用8字节数组从数据库进行读取和写入。 但是，字节数组是可变引用类型，这使得处理起来有些困难。 值转换器允许 `rowversion` 改为映射到 `ulong` 属性，该属性比字节数组更合适且更易于使用。 例如，请考虑一个 `Blog` 具有 ulong 并发令牌的实体：

<!--
        public class Blog
        {
            public int Id { get; set; }
            public string Name { get; set; }
            public ulong Version { get; set; }
        }
-->
[!code-csharp[ULongConcurrencyModel](../../../samples/core/Modeling/ValueConversions/ULongConcurrency.cs?name=ULongConcurrencyModel)]

这可以使用值转换器映射到 SQL server `rowversion` 列：

<!--
                modelBuilder.Entity<Blog>()
                    .Property(e => e.Version)
                    .IsRowVersion()
                    .HasConversion<byte[]>();
-->
[!code-csharp[ConfigureULongConcurrency](../../../samples/core/Modeling/ValueConversions/ULongConcurrency.cs?name=ConfigureULongConcurrency)]

### <a name="specify-the-datetimekind-when-reading-dates"></a>在读取日期时指定 DateTime. Kind

<xref:System.DateTime.Kind%2A?displayProperty=nameWithType>将存储为或时，SQL Server 会丢弃标志 <xref:System.DateTime> [`datetime`](/sql/t-sql/data-types/datetime-transact-sql) [`datetime2`](/sql/t-sql/data-types/datetime2-transact-sql) 。 这意味着，从数据库返回的 DateTime 值始终具有 <xref:System.DateTimeKind> 的 `Unspecified` 。

可以通过两种方式使用值转换器来处理这种情况。 首先，EF Core 具有一个值转换器，该转换器可创建可保留标志的8字节不透明值 `Kind` 。 例如：

<!--
                modelBuilder.Entity<Post>()
                    .Property(e => e.PostedOn)
                    .HasConversion<long>();
-->
[!code-csharp[ConfigurePreserveDateTimeKind1](../../../samples/core/Modeling/ValueConversions/PreserveDateTimeKind.cs?name=ConfigurePreserveDateTimeKind1)]

这允许 `Kind` 在数据库中混合具有不同标志的日期时间值。

此方法的问题是数据库不再具有可识别的 `datetime` 或 `datetime2` 列。 通常，通常情况下，通常会存储 UTC 时间 (或者，不太常见，总是本地时间) ，然后 `Kind` 使用值转换器忽略标志或将其设置为合适的值。 例如，下面的转换器确保 `DateTime` 从数据库中读取的值具有 <xref:System.DateTimeKind> `UTC` ：

<!--
                modelBuilder.Entity<Post>()
                    .Property(e => e.LastUpdated)
                    .HasConversion(
                        v => v,
                        v => new DateTime(v.Ticks, DateTimeKind.Utc));
-->
[!code-csharp[ConfigurePreserveDateTimeKind2](../../../samples/core/Modeling/ValueConversions/PreserveDateTimeKind.cs?name=ConfigurePreserveDateTimeKind2)]

如果在实体实例中设置了局部值和 UTC 值的组合，则在插入前可以使用转换器进行适当的转换。 例如：

<!--
                modelBuilder.Entity<Post>()
                    .Property(e => e.LastUpdated)
                    .HasConversion(
                        v => v.ToUniversalTime(),
                        v => new DateTime(v.Ticks, DateTimeKind.Utc));
-->
[!code-csharp[ConfigurePreserveDateTimeKind3](../../../samples/core/Modeling/ValueConversions/PreserveDateTimeKind.cs?name=ConfigurePreserveDateTimeKind3)]

> [!NOTE]
> 仔细考虑统一所有数据库访问代码以始终使用 UTC 时间，只需处理向用户呈现数据的本地时间。

### <a name="use-case-insensitive-string-keys"></a>使用不区分大小写的字符串密钥

默认情况下，某些数据库（包括 SQL Server）执行不区分大小写的字符串比较。 另一方面，.NET 默认执行区分大小写的字符串比较。 这意味着外键值（如 "DotNet"）将匹配 SQL Server 上的主键值 "DotNet"，但不会在 EF Core 中匹配。 键的值比较器可用于强制 EF Core 为类似于数据库中的不区分大小写的字符串比较。 例如，请考虑使用字符串键的博客/文章模型：

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

如果某些 `Post.BlogId` 值具有不同的大小写，则此操作不会按预期方式工作。 导致这种错误的原因将取决于应用程序正在执行的操作，但通常涉及未正确 [固定](xref:core/change-tracking/relationship-changes) 的对象的图形，以及/或由于 FK 值错误而导致失败的更新。 值比较器可用于更正此错误：

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
> 除了区分大小写，.NET 字符串比较和数据库字符串比较可能会有所不同。 此模式适用于简单的 ASCII 键，但对于任何类型的区域性特定字符，密钥可能会失败。 有关详细信息，请参阅 [排序规则和区分大小写](xref:core/miscellaneous/collations-and-case-sensitivity) 。

### <a name="handle-fixed-length-database-strings"></a>处理固定长度的数据库字符串

前面的示例不需要值转换器。 但是，转换器可用于固定长度的数据库字符串类型（如或） `char(20)` `nchar(20)` 。 只要将值插入到数据库中，就会将固定长度的字符串填充到其完整长度。 这意味着，将 "" 的键值 `dotnet` 作为 "" 读回数据库 `dotnet..............` ，其中 `.` 表示空格字符。 这样，就不会正确地将未填充的键值进行比较。

值转换器可用于在读取键值时剪裁填充。 这可以与上一示例中的值比较器结合，以正确比较固定长度不区分大小写的 ASCII 键。 例如：

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

### <a name="encrypt-property-values"></a>加密属性值

值转换器可用于在将属性值发送到数据库之前对其进行加密，然后将其解密。例如，使用字符串反转替代实加密算法：

<!--
                modelBuilder.Entity<User>().Property(e => e.Password).HasConversion(
                    v => new string(v.Reverse().ToArray()),
                    v => new string(v.Reverse().ToArray()));
-->
[!code-csharp[ConfigureEncryptPropertyValues](../../../samples/core/Modeling/ValueConversions/EncryptPropertyValues.cs?name=ConfigureEncryptPropertyValues)]

> [!NOTE]
> 目前没有任何方法可以从值转换器内获取对当前 DbContext 或其他会话状态的引用。 这限制了可以使用的加密类型。 对 [GitHub 问题 #11597](https://github.com/dotnet/efcore/issues/12205) 投票，以删除此限制。

> [!WARNING]
> 如果你滚动自己的加密以保护敏感数据，请确保了解所有影响。 请考虑使用预先生成的加密机制，如 SQL Server 上的 [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) 。

## <a name="limitations"></a>限制

值转换系统存在一些已知的当前限制：

* 当前无法在一个位置指定给定类型的每个属性都必须使用相同的值转换器。 请 (👍) [GitHub 问题](https://github.com/dotnet/efcore/issues/10784) 的投票 #10784 如果这是你需要的东西。
* 如上所述， `null` 无法转换。 请 (👍) [GitHub 问题](https://github.com/dotnet/efcore/issues/13850) 的投票 #13850 如果这是你需要的东西。
* 目前没有办法将一个属性转换为多个列，反之亦然。 请 (👍) [GitHub 问题](https://github.com/dotnet/efcore/issues/13947) 的投票 #13947 如果这是你需要的东西。
* 对于通过值转换器映射的大多数密钥，不支持值生成。 请 (👍) [GitHub 问题](https://github.com/dotnet/efcore/issues/11597) 的投票 #11597 如果这是你需要的东西。
* 值转换无法引用当前的 DbContext 实例。 请 (👍) [GitHub 问题](https://github.com/dotnet/efcore/issues/12205) 的投票 #11597 如果这是你需要的东西。

将来的版本将考虑删除这些限制。
