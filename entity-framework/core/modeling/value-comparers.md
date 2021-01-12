---
title: 值比较器-EF Core
description: 使用值比较器来控制 EF Core 比较属性值的方式
author: ajcvickers
ms.date: 03/20/2020
uid: core/modeling/value-comparers
ms.openlocfilehash: 618341315de05f6efae8f43384809ed72226e18b
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128506"
---
# <a name="value-comparers"></a>值比较器

> [!NOTE]
> EF Core 3.0 中已引入此功能。

> [!TIP]
> 可在 GitHub 上找到此文档中的代码作为可 [运行示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/ValueConversions/)。

## <a name="background"></a>背景

更改跟踪是指 EF Core 自动确定已加载的实体实例上的应用程序所执行的更改，以便在调用时可以将这些更改保存回数据库 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。 EF Core 通常会在从数据库中加载实例时获取实例的 *快照* ，并将该快照与向应用程序传递的实例进行 *比较* 。

EF Core 附带了用于快照和比较数据库中使用的大多数标准类型的内置逻辑，因此用户通常不需要担心本主题。 但是，当通过 [值转换器](xref:core/modeling/value-conversions)映射属性时，EF Core 需要对任意用户类型执行比较，这可能很复杂。 默认情况下，EF Core 使用由类型定义的默认相等比较 (例如 `Equals` ，) 的方法; 对于快照，会复制值类型以生成快照，而对于引用类型，则不进行复制，同一实例用作快照。

如果内置的比较行为不合适，则用户可以提供 *值比较器*，其中包含快照的逻辑、比较和计算哈希代码。 例如，下面的属性的值转换设置为要 `List<int>` 转换为数据库中的 JSON 字符串的值，并定义适当的值比较器。

[!code-csharp[ListProperty](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ConfigureListProperty)]

有关更多详细信息，请参阅下面的 [可变类](#mutable-classes) 。

请注意，在确定解析关系时两个键值是否相同时，还会使用值比较器。下面对此进行了说明。

## <a name="shallow-vs-deep-comparison"></a>浅比较与深层比较

对于不变的可变值类型（例如 `int` ），EF Core 的默认逻辑工作正常：在进行快照时，值将按原样复制，并与类型的内置相等比较进行比较。 实现自己的值比较器时，请务必考虑深度或浅比较 (和快照) 逻辑是否合适。

请考虑字节数组，该数组可以任意大。 可以比较以下内容：

* 通过引用，因此只有在使用新的字节数组时才会检测到差异
* 进行深层比较时，将检测到数组中字节的变化

默认情况下，EF Core 使用这些方法中的第一个方法来实现非键字节数组。 也就是说，仅对引用进行比较，并且仅当现有字节数组替换为新的字节数组时，才会检测到更改。 这是一项切实的决策，可避免在执行时复制整个数组并对它们进行字节到字节的比较，这种情况下，将 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 以高性能的方式处理替换为一个映像的常见方案。

另一方面，如果使用字节数组表示二进制键，则引用相等性将不起作用，因为 FK 属性不太可能设置为与需要进行比较的 PK 属性 _相同的实例_ 。 因此，EF Core 对用作键的字节数组使用深层比较;这不太可能会产生大的性能，因为二进制密钥通常很短。

请注意，所选的比较和快照逻辑必须彼此对应： deep 比较要求深度快照才能正常工作。

## <a name="simple-immutable-classes"></a>简单的不可变类

考虑使用值转换器映射简单的不可变类的属性。

[!code-csharp[SimpleImmutableClass](../../../samples/core/Modeling/ValueConversions/MappingImmutableClassProperty.cs?name=SimpleImmutableClass)]

[!code-csharp[ConfigureImmutableClassProperty](../../../samples/core/Modeling/ValueConversions/MappingImmutableClassProperty.cs?name=ConfigureImmutableClassProperty)]

此类型的属性不需要特殊比较或快照，原因如下：

* 重写相等性，以使不同的实例正确比较
* 类型是不可变的，因此不可能改变快照值

因此，在这种情况下，EF Core 的默认行为是正确的。

## <a name="simple-immutable-structs"></a>简单的不可变结构

简单结构的映射也很简单，并且不需要特殊的比较器或快照。

[!code-csharp[SimpleImmutableStruct](../../../samples/core/Modeling/ValueConversions/MappingImmutableStructProperty.cs?name=SimpleImmutableStruct)]

[!code-csharp[ConfigureImmutableStructProperty](../../../samples/core/Modeling/ValueConversions/MappingImmutableStructProperty.cs?name=ConfigureImmutableStructProperty)]

EF Core 提供对结构属性的编译按成员比较的内置支持。
这意味着结构无需重写 EF Core 的相等性，但出于 [其他原因](/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-define-value-equality-for-a-type)，你仍可以选择执行此操作。
此外，不需要特殊快照，因为结构是不可变的，并且始终复制按成员。
 (对于可变结构也是如此，但 [通常应避免可变结构](/dotnet/csharp/write-safe-efficient-code)。 ) 

## <a name="mutable-classes"></a>可变类

建议尽可能使用值转换器 (类或结构) 的不可变类型。
这通常更高效，并具有比使用可变类型更清晰的语义。

但这种情况下，通常使用应用程序无法更改的类型的属性。
例如，映射包含数字列表的属性：

[!code-csharp[ListProperty](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ListProperty)]

<xref:System.Collections.Generic.List%601> 类：

* 具有引用相等性;包含相同值的两个列表被视为不同的。
* 可变;可以添加和删除列表中的值。

列表属性上的典型值转换可能会在列表与 JSON 之间进行转换：

### <a name="ef-core-50"></a>[EF Core 5.0](#tab/ef5)

[!code-csharp[ListProperty](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ConfigureListProperty&highlight=7-10)]

### <a name="older-versions"></a>[旧版本](#tab/older-versions)

[!code-csharp[ListProperty](../../../samples/core/Modeling/ValueConversions/MappingListPropertyOld.cs?name=ConfigureListProperty&highlight=8-11,17)]

***

<xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601>构造函数接受三个表达式：

* 用于检查相等性的表达式
* 用于生成哈希代码的表达式
* 用于对值进行快照的表达式

在这种情况下，比较是通过检查数字序列是否相同来完成的。

同样，哈希代码是根据此相同的顺序生成的。
 (请注意，这是一个基于可变值的哈希代码，因此可能 [会导致问题](https://ericlippert.com/2011/02/28/guidelines-and-rules-for-gethashcode/)。
如果可以，则改为不可变。 ) 

通过使用克隆列表创建快照 `ToList` 。
同样，仅当要转变列表时，才需要这样做。
如果可以，则改为不可变。

> [!NOTE]
> 值转换器和比较器使用表达式而不是简单委托来构造。
> 这是因为 EF Core 会将这些表达式插入到更复杂的表达式树中，然后将其编译到实体整形程序委托中。
> 从概念上讲，这类似于编译器内联。
> 例如，简单转换可能是在强制转换中编译的，而不是调用其他方法来执行转换。

## <a name="key-comparers"></a>密钥比较器

背景部分介绍了为何密钥比较可能需要特殊语义。
在 primary、principal 或 foreign key 属性上设置键时，请确保创建的比较器适用于键。

<xref:Microsoft.EntityFrameworkCore.MutablePropertyExtensions.SetKeyValueComparer%2A>在相同的属性上需要不同语义的罕见情况下使用。

> [!NOTE]
> <xref:Microsoft.EntityFrameworkCore.MutablePropertyExtensions.SetStructuralValueComparer%2A> 在 EF Core 5.0 中已过时。 请改用 <xref:Microsoft.EntityFrameworkCore.MutablePropertyExtensions.SetKeyValueComparer%2A>。

## <a name="overriding-the-default-comparer"></a>重写默认比较器

有时 EF Core 使用的默认比较可能不合适。
例如，默认情况下，不会在 EF Core 中检测到字节数组的变化。
这可以通过对属性设置不同的比较器来重写：

[!code-csharp[OverrideComparer](../../../samples/core/Modeling/ValueConversions/OverridingByteArrayComparisons.cs?name=OverrideComparer)]

EF Core 现在会比较字节序列并因此将检测字节数组突变。
