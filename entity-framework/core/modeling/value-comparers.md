---
title: 值比较器-EF Core
description: 使用值比较器来控制 EF Core 比较属性值的方式
author: ajcvickers
ms.date: 03/20/2020
uid: core/modeling/value-comparers
ms.openlocfilehash: d59ab093c7e9d251aac80420972d3e35edabae0b
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003609"
---
# <a name="value-comparers"></a>值比较器

> [!NOTE]  
> EF Core 3.0 中引入了此功能。

> [!TIP]  
> 可在 GitHub 上找到此文档中的代码作为可 [运行示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/ValueConversions/)。

## <a name="background"></a>背景

EF Core 需要在以下情况中比较属性值：

* 确定在[检测更新更改](xref:core/saving/basic)时是否已更改属性
* 确定解析关系时两个键值是否相同

这会自动处理常见基元类型，如 int、bool、DateTime 等。

对于更复杂的类型，需要选择如何进行比较。
例如，可以对字节数组进行比较：

* 通过引用，因此只有在使用新的字节数组时才会检测到差异
* 进行深层比较时，将检测到数组中字节的变化

默认情况下，EF Core 使用这些方法中的第一个方法来实现非键字节数组。
也就是说，仅对引用进行比较，并且仅当现有字节数组替换为新的字节数组时，才会检测到更改。
这是一种切实的决策，可避免在执行 SaveChanges 时对多个大字节数组进行深层比较。
但替换具有不同图像的映像的常见情况是以高性能的方式处理的。

另一方面，将字节数组用于表示二进制键时，引用相等性将不起作用。
FK 属性不太可能设置为与需要进行比较的 PK 属性 _相同的实例_ 。
因此，EF Core 对用作键的字节数组使用深层比较。
这不太可能会产生大的性能，因为二进制密钥通常很短。

### <a name="snapshots"></a>快照

可变类型的深层比较意味着 EF Core 需要能够创建属性值的深度 "快照"。
只复制引用会导致同时改变当前值和快照，因为它们是 _相同的对象_。
因此，在可变类型上使用深层比较时，还需要深层快照。

## <a name="properties-with-value-converters"></a>具有值转换器的属性

在上面的示例中，EF Core 具有对字节数组的本机映射支持，因此可自动选择适当的默认值。
但是，如果通过 [值转换器](xref:core/modeling/value-conversions)映射属性，则 EF Core 无法始终确定要使用的适当比较。
相反，EF Core 始终使用属性类型定义的默认相等比较。
这通常是正确的，但在映射更复杂的类型时可能需要重写。

### <a name="simple-immutable-classes"></a>简单的不可变类

考虑使用值转换器映射简单的不可变类的属性。

[!code-csharp[SimpleImmutableClass](../../../samples/core/Modeling/ValueConversions/MappingImmutableClassProperty.cs?name=SimpleImmutableClass)]

[!code-csharp[ConfigureImmutableClassProperty](../../../samples/core/Modeling/ValueConversions/MappingImmutableClassProperty.cs?name=ConfigureImmutableClassProperty)]

此类型的属性不需要特殊比较或快照，原因如下：

* 重写相等性，以使不同的实例正确比较
* 类型是不可变的，因此不可能改变快照值

因此，在这种情况下，EF Core 的默认行为是正确的。

### <a name="simple-immutable-structs"></a>简单的不可变结构

简单结构的映射也很简单，并且不需要特殊的比较器或快照。

[!code-csharp[SimpleImmutableStruct](../../../samples/core/Modeling/ValueConversions/MappingImmutableStructProperty.cs?name=SimpleImmutableStruct)]

[!code-csharp[ConfigureImmutableStructProperty](../../../samples/core/Modeling/ValueConversions/MappingImmutableStructProperty.cs?name=ConfigureImmutableStructProperty)]

EF Core 提供对结构属性的编译按成员比较的内置支持。
这意味着，结构不需要为 EF 重写相等，但出于 [其他原因](/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-define-value-equality-for-a-type)，你仍可以选择执行此操作。
此外，不需要特殊快照，因为结构是不可变的，始终按成员复制。
 (对于可变结构也是如此，但 [通常应避免可变结构](/dotnet/csharp/write-safe-efficient-code)。 ) 

### <a name="mutable-classes"></a>可变类

建议尽可能使用值转换器 (类或结构) 的不可变类型。
这通常更高效，并具有比使用可变类型更清晰的语义。

但这种情况下，通常使用应用程序无法更改的类型的属性。
例如，映射包含数字列表的属性：

[!code-csharp[ListProperty](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ListProperty)]

[ `List<T>` 类](/dotnet/api/system.collections.generic.list-1)：

* 具有引用相等性;包含相同值的两个列表被视为不同的。
* 可变;可以添加和删除列表中的值。

列表属性上的典型值转换可能会在列表与 JSON 之间进行转换：

[!code-csharp[ConfigureListProperty](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ConfigureListProperty)]

这需要 `ValueComparer<T>` 在属性上设置，以强制 EF Core 在此转换中使用正确的比较：

[!code-csharp[ConfigureListPropertyComparer](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ConfigureListPropertyComparer)]

> [!NOTE]  
> 模型生成器 ( "熟知" ) API 设置值比较器尚未实现。
> 相反，上面的代码会对生成器作为 "Metadata" 公开的低级 IMutableProperty 调用 SetValueComparer。

`ValueComparer<T>`构造函数接受三个表达式：

* 用于检查相等性的表达式
* 用于生成哈希代码的表达式
* 用于对值进行快照的表达式  

在这种情况下，比较是通过检查数字序列是否相同来完成的。

同样，哈希代码是根据此相同的顺序生成的。
 (请注意，这是一个基于可变值的哈希代码，因此可能 [会导致问题](https://ericlippert.com/2011/02/28/guidelines-and-rules-for-gethashcode/)。
如果可以，则改为不可变。 ) 

快照是通过使用 System.linq.enumerable.tolist 克隆列表来创建的。
同样，仅当要转变列表时，才需要这样做。
如果可以，则改为不可变。

> [!NOTE]  
> 值转换器和比较器使用表达式而不是简单委托来构造。
> 这是因为 EF 将这些表达式插入更复杂的表达式树中，然后将其编译到实体整形程序委托中。
> 从概念上讲，这类似于编译器内联。
> 例如，简单转换可能是在强制转换中编译的，而不是调用其他方法来执行转换。

### <a name="key-comparers"></a>密钥比较器

背景部分介绍了为何密钥比较可能需要特殊语义。
在 primary、principal 或 foreign key 属性上设置键时，请确保创建的比较器适用于键。

在相同的属性中需要不同语义的罕见情况下，使用 [SetKeyValueComparer](/dotnet/api/microsoft.entityframeworkcore.mutablepropertyextensions.setkeyvaluecomparer) 。

> [!NOTE]  
> EF Core 5.0 中的 SetStructuralComparer 已过时。
> 改用 SetKeyValueComparer。

### <a name="overriding-defaults"></a>替代默认值

有时 EF Core 使用的默认比较可能不合适。
例如，默认情况下，不会在 EF Core 中检测到字节数组的变化。
这可以通过对属性设置不同的比较器来重写：

[!code-csharp[OverrideComparer](../../../samples/core/Modeling/ValueConversions/OverridingByteArrayComparisons.cs?name=OverrideComparer)]

EF Core 现在会比较字节序列并因此将检测字节数组突变。
