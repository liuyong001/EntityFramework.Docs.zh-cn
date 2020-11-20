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
# <a name="value-comparers"></a><span data-ttu-id="8dfe2-103">值比较器</span><span class="sxs-lookup"><span data-stu-id="8dfe2-103">Value Comparers</span></span>

> [!NOTE]  
> <span data-ttu-id="8dfe2-104">EF Core 3.0 中引入了此功能。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-104">This feature was introduced in EF Core 3.0.</span></span>

> [!TIP]  
> <span data-ttu-id="8dfe2-105">可在 GitHub 上找到此文档中的代码作为可 [运行示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/ValueConversions/)。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-105">The code in this document can be found on GitHub as a [runnable sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/ValueConversions/).</span></span>

## <a name="background"></a><span data-ttu-id="8dfe2-106">背景</span><span class="sxs-lookup"><span data-stu-id="8dfe2-106">Background</span></span>

<span data-ttu-id="8dfe2-107">EF Core 需要在以下情况中比较属性值：</span><span class="sxs-lookup"><span data-stu-id="8dfe2-107">EF Core needs to compare property values when:</span></span>

* <span data-ttu-id="8dfe2-108">确定在[检测更新更改](xref:core/saving/basic)时是否已更改属性</span><span class="sxs-lookup"><span data-stu-id="8dfe2-108">Determining whether a property has been changed as part of [detecting changes for updates](xref:core/saving/basic)</span></span>
* <span data-ttu-id="8dfe2-109">确定解析关系时两个键值是否相同</span><span class="sxs-lookup"><span data-stu-id="8dfe2-109">Determining whether two key values are the same when resolving relationships</span></span>

<span data-ttu-id="8dfe2-110">这会自动处理常见基元类型，如 int、bool、DateTime 等。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-110">This is handled automatically for common primitive types such as int, bool, DateTime, etc.</span></span>

<span data-ttu-id="8dfe2-111">对于更复杂的类型，需要选择如何进行比较。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-111">For more complex types, choices need to be made as to how to do the comparison.</span></span>
<span data-ttu-id="8dfe2-112">例如，可以对字节数组进行比较：</span><span class="sxs-lookup"><span data-stu-id="8dfe2-112">For example, a byte array could be compared:</span></span>

* <span data-ttu-id="8dfe2-113">通过引用，因此只有在使用新的字节数组时才会检测到差异</span><span class="sxs-lookup"><span data-stu-id="8dfe2-113">By reference, such that a difference is only detected if a new byte array is used</span></span>
* <span data-ttu-id="8dfe2-114">进行深层比较时，将检测到数组中字节的变化</span><span class="sxs-lookup"><span data-stu-id="8dfe2-114">By deep comparison, such that mutation of the bytes in the array is detected</span></span>

<span data-ttu-id="8dfe2-115">默认情况下，EF Core 使用这些方法中的第一个方法来实现非键字节数组。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-115">By default, EF Core uses the first of these approaches for non-key byte arrays.</span></span>
<span data-ttu-id="8dfe2-116">也就是说，仅对引用进行比较，并且仅当现有字节数组替换为新的字节数组时，才会检测到更改。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-116">That is, only references are compared and a change is detected only when an existing byte array is replaced with a new one.</span></span>
<span data-ttu-id="8dfe2-117">这是一种切实的决策，可避免在执行 SaveChanges 时对多个大字节数组进行深层比较。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-117">This is a pragmatic decision that avoids deep comparison of many large byte arrays when executing SaveChanges.</span></span>
<span data-ttu-id="8dfe2-118">但替换具有不同图像的映像的常见情况是以高性能的方式处理的。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-118">But the common scenario of replacing, say, an image with a different image is handled in a performant way.</span></span>

<span data-ttu-id="8dfe2-119">另一方面，将字节数组用于表示二进制键时，引用相等性将不起作用。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-119">On the other hand, reference equality would not work when byte arrays are used to represent binary keys.</span></span>
<span data-ttu-id="8dfe2-120">FK 属性不太可能设置为与需要进行比较的 PK 属性 _相同的实例_ 。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-120">It's very unlikely that an FK property is set to the _same instance_ as a PK property to which it needs to be compared.</span></span>
<span data-ttu-id="8dfe2-121">因此，EF Core 对用作键的字节数组使用深层比较。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-121">Therefore, EF Core uses deep comparisons for byte arrays acting as keys.</span></span>
<span data-ttu-id="8dfe2-122">这不太可能会产生大的性能，因为二进制密钥通常很短。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-122">This is unlikely to have a big performance hit since binary keys are usually short.</span></span>

### <a name="snapshots"></a><span data-ttu-id="8dfe2-123">快照</span><span class="sxs-lookup"><span data-stu-id="8dfe2-123">Snapshots</span></span>

<span data-ttu-id="8dfe2-124">可变类型的深层比较意味着 EF Core 需要能够创建属性值的深度 "快照"。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-124">Deep comparisons on mutable types means that EF Core needs the ability to create a deep "snapshot" of the property value.</span></span>
<span data-ttu-id="8dfe2-125">只复制引用会导致同时改变当前值和快照，因为它们是 _相同的对象_。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-125">Just copying the reference instead would result in mutating both the current value and the snapshot, since they are _the same object_.</span></span>
<span data-ttu-id="8dfe2-126">因此，在可变类型上使用深层比较时，还需要深层快照。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-126">Therefore, when deep comparisons are used on mutable types, deep snapshotting is also required.</span></span>

## <a name="properties-with-value-converters"></a><span data-ttu-id="8dfe2-127">具有值转换器的属性</span><span class="sxs-lookup"><span data-stu-id="8dfe2-127">Properties with value converters</span></span>

<span data-ttu-id="8dfe2-128">在上面的示例中，EF Core 具有对字节数组的本机映射支持，因此可自动选择适当的默认值。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-128">In the case above, EF Core has native mapping support for byte arrays and so can automatically choose appropriate defaults.</span></span>
<span data-ttu-id="8dfe2-129">但是，如果通过 [值转换器](xref:core/modeling/value-conversions)映射属性，则 EF Core 无法始终确定要使用的适当比较。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-129">However, if the property is mapped through a [value converter](xref:core/modeling/value-conversions), then EF Core can't always determine the appropriate comparison to use.</span></span>
<span data-ttu-id="8dfe2-130">相反，EF Core 始终使用属性类型定义的默认相等比较。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-130">Instead, EF Core always uses the default equality comparison defined by the type of the property.</span></span>
<span data-ttu-id="8dfe2-131">这通常是正确的，但在映射更复杂的类型时可能需要重写。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-131">This is often correct, but may need to be overridden when mapping more complex types.</span></span>

### <a name="simple-immutable-classes"></a><span data-ttu-id="8dfe2-132">简单的不可变类</span><span class="sxs-lookup"><span data-stu-id="8dfe2-132">Simple immutable classes</span></span>

<span data-ttu-id="8dfe2-133">考虑使用值转换器映射简单的不可变类的属性。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-133">Consider a property that uses a value converter to map a simple, immutable class.</span></span>

[!code-csharp[SimpleImmutableClass](../../../samples/core/Modeling/ValueConversions/MappingImmutableClassProperty.cs?name=SimpleImmutableClass)]

[!code-csharp[ConfigureImmutableClassProperty](../../../samples/core/Modeling/ValueConversions/MappingImmutableClassProperty.cs?name=ConfigureImmutableClassProperty)]

<span data-ttu-id="8dfe2-134">此类型的属性不需要特殊比较或快照，原因如下：</span><span class="sxs-lookup"><span data-stu-id="8dfe2-134">Properties of this type do not need special comparisons or snapshots because:</span></span>

* <span data-ttu-id="8dfe2-135">重写相等性，以使不同的实例正确比较</span><span class="sxs-lookup"><span data-stu-id="8dfe2-135">Equality is overridden so that different instances will compare correctly</span></span>
* <span data-ttu-id="8dfe2-136">类型是不可变的，因此不可能改变快照值</span><span class="sxs-lookup"><span data-stu-id="8dfe2-136">The type is immutable, so there is no chance of mutating a snapshot value</span></span>

<span data-ttu-id="8dfe2-137">因此，在这种情况下，EF Core 的默认行为是正确的。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-137">So in this case the default behavior of EF Core is fine as it is.</span></span>

### <a name="simple-immutable-structs"></a><span data-ttu-id="8dfe2-138">简单的不可变结构</span><span class="sxs-lookup"><span data-stu-id="8dfe2-138">Simple immutable Structs</span></span>

<span data-ttu-id="8dfe2-139">简单结构的映射也很简单，并且不需要特殊的比较器或快照。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-139">The mapping for simple structs is also simple and requires no special comparers or snapshotting.</span></span>

[!code-csharp[SimpleImmutableStruct](../../../samples/core/Modeling/ValueConversions/MappingImmutableStructProperty.cs?name=SimpleImmutableStruct)]

[!code-csharp[ConfigureImmutableStructProperty](../../../samples/core/Modeling/ValueConversions/MappingImmutableStructProperty.cs?name=ConfigureImmutableStructProperty)]

<span data-ttu-id="8dfe2-140">EF Core 提供对结构属性的编译按成员比较的内置支持。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-140">EF Core has built-in support for generating compiled, memberwise comparisons of struct properties.</span></span>
<span data-ttu-id="8dfe2-141">这意味着，结构不需要为 EF 重写相等，但出于 [其他原因](/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-define-value-equality-for-a-type)，你仍可以选择执行此操作。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-141">This means structs don't need to have equality overridden for EF, but you may still choose to do this for [other reasons](/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-define-value-equality-for-a-type).</span></span>
<span data-ttu-id="8dfe2-142">此外，不需要特殊快照，因为结构是不可变的，始终按成员复制。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-142">Also, special snapshotting is not needed since structs immutable and are always memberwise copied anyway.</span></span>
<span data-ttu-id="8dfe2-143"> (对于可变结构也是如此，但 [通常应避免可变结构](/dotnet/csharp/write-safe-efficient-code)。 ) </span><span class="sxs-lookup"><span data-stu-id="8dfe2-143">(This is also true for mutable structs, but [mutable structs should in general be avoided](/dotnet/csharp/write-safe-efficient-code).)</span></span>

### <a name="mutable-classes"></a><span data-ttu-id="8dfe2-144">可变类</span><span class="sxs-lookup"><span data-stu-id="8dfe2-144">Mutable classes</span></span>

<span data-ttu-id="8dfe2-145">建议尽可能使用值转换器 (类或结构) 的不可变类型。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-145">It is recommended that you use immutable types (classes or structs) with value converters when possible.</span></span>
<span data-ttu-id="8dfe2-146">这通常更高效，并具有比使用可变类型更清晰的语义。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-146">This is usually more efficient and has cleaner semantics than using a mutable type.</span></span>

<span data-ttu-id="8dfe2-147">但这种情况下，通常使用应用程序无法更改的类型的属性。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-147">However, that being said, it is common to use properties of types that the application cannot change.</span></span>
<span data-ttu-id="8dfe2-148">例如，映射包含数字列表的属性：</span><span class="sxs-lookup"><span data-stu-id="8dfe2-148">For example, mapping a property containing a list of numbers:</span></span>

[!code-csharp[ListProperty](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ListProperty)]

<span data-ttu-id="8dfe2-149">[ `List<T>` 类](/dotnet/api/system.collections.generic.list-1)：</span><span class="sxs-lookup"><span data-stu-id="8dfe2-149">The [`List<T>` class](/dotnet/api/system.collections.generic.list-1):</span></span>

* <span data-ttu-id="8dfe2-150">具有引用相等性;包含相同值的两个列表被视为不同的。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-150">Has reference equality; two lists containing the same values are treated as different.</span></span>
* <span data-ttu-id="8dfe2-151">可变;可以添加和删除列表中的值。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-151">Is mutable; values in the list can be added and removed.</span></span>

<span data-ttu-id="8dfe2-152">列表属性上的典型值转换可能会在列表与 JSON 之间进行转换：</span><span class="sxs-lookup"><span data-stu-id="8dfe2-152">A typical value conversion on a list property might convert the list to and from JSON:</span></span>

[!code-csharp[ConfigureListProperty](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ConfigureListProperty)]

<span data-ttu-id="8dfe2-153">这需要 `ValueComparer<T>` 在属性上设置，以强制 EF Core 在此转换中使用正确的比较：</span><span class="sxs-lookup"><span data-stu-id="8dfe2-153">This then requires setting a `ValueComparer<T>` on the property to force EF Core use correct comparisons with this conversion:</span></span>

[!code-csharp[ConfigureListPropertyComparer](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ConfigureListPropertyComparer)]

> [!NOTE]  
> <span data-ttu-id="8dfe2-154">模型生成器 ( "熟知" ) API 设置值比较器尚未实现。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-154">The model builder ("fluent") API to set a value comparer has not yet been implemented.</span></span>
> <span data-ttu-id="8dfe2-155">相反，上面的代码会对生成器作为 "Metadata" 公开的低级 IMutableProperty 调用 SetValueComparer。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-155">Instead, the code above calls SetValueComparer on the lower-level IMutableProperty exposed by the builder as 'Metadata'.</span></span>

<span data-ttu-id="8dfe2-156">`ValueComparer<T>`构造函数接受三个表达式：</span><span class="sxs-lookup"><span data-stu-id="8dfe2-156">The `ValueComparer<T>` constructor accepts three expressions:</span></span>

* <span data-ttu-id="8dfe2-157">用于检查相等性的表达式</span><span class="sxs-lookup"><span data-stu-id="8dfe2-157">An expression for checking equality</span></span>
* <span data-ttu-id="8dfe2-158">用于生成哈希代码的表达式</span><span class="sxs-lookup"><span data-stu-id="8dfe2-158">An expression for generating a hash code</span></span>
* <span data-ttu-id="8dfe2-159">用于对值进行快照的表达式</span><span class="sxs-lookup"><span data-stu-id="8dfe2-159">An expression to snapshot a value</span></span>  

<span data-ttu-id="8dfe2-160">在这种情况下，比较是通过检查数字序列是否相同来完成的。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-160">In this case the comparison is done by checking if the sequences of numbers are the same.</span></span>

<span data-ttu-id="8dfe2-161">同样，哈希代码是根据此相同的顺序生成的。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-161">Likewise, the hash code is built from this same sequence.</span></span>
<span data-ttu-id="8dfe2-162"> (请注意，这是一个基于可变值的哈希代码，因此可能 [会导致问题](https://ericlippert.com/2011/02/28/guidelines-and-rules-for-gethashcode/)。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-162">(Note that this is a hash code over mutable values and hence can [cause problems](https://ericlippert.com/2011/02/28/guidelines-and-rules-for-gethashcode/).</span></span>
<span data-ttu-id="8dfe2-163">如果可以，则改为不可变。 ) </span><span class="sxs-lookup"><span data-stu-id="8dfe2-163">Be immutable instead if you can.)</span></span>

<span data-ttu-id="8dfe2-164">快照是通过使用 System.linq.enumerable.tolist 克隆列表来创建的。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-164">The snapshot is created by cloning the list with ToList.</span></span>
<span data-ttu-id="8dfe2-165">同样，仅当要转变列表时，才需要这样做。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-165">Again, this is only needed if the lists are going to be mutated.</span></span>
<span data-ttu-id="8dfe2-166">如果可以，则改为不可变。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-166">Be immutable instead if you can.</span></span>

> [!NOTE]  
> <span data-ttu-id="8dfe2-167">值转换器和比较器使用表达式而不是简单委托来构造。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-167">Value converters and comparers are constructed using expressions rather than simple delegates.</span></span>
> <span data-ttu-id="8dfe2-168">这是因为 EF 将这些表达式插入更复杂的表达式树中，然后将其编译到实体整形程序委托中。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-168">This is because EF inserts these expressions into a much more complex expression tree that is then compiled into an entity shaper delegate.</span></span>
> <span data-ttu-id="8dfe2-169">从概念上讲，这类似于编译器内联。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-169">Conceptually, this is similar to compiler inlining.</span></span>
> <span data-ttu-id="8dfe2-170">例如，简单转换可能是在强制转换中编译的，而不是调用其他方法来执行转换。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-170">For example, a simple conversion may just be a compiled in cast, rather than a call to another method to do the conversion.</span></span>

### <a name="key-comparers"></a><span data-ttu-id="8dfe2-171">密钥比较器</span><span class="sxs-lookup"><span data-stu-id="8dfe2-171">Key comparers</span></span>

<span data-ttu-id="8dfe2-172">背景部分介绍了为何密钥比较可能需要特殊语义。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-172">The background section covers why key comparisons may require special semantics.</span></span>
<span data-ttu-id="8dfe2-173">在 primary、principal 或 foreign key 属性上设置键时，请确保创建的比较器适用于键。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-173">Make sure to create a comparer that is appropriate for keys when setting it on a primary, principal, or foreign key property.</span></span>

<span data-ttu-id="8dfe2-174">在相同的属性中需要不同语义的罕见情况下，使用 [SetKeyValueComparer](/dotnet/api/microsoft.entityframeworkcore.mutablepropertyextensions.setkeyvaluecomparer) 。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-174">Use [SetKeyValueComparer](/dotnet/api/microsoft.entityframeworkcore.mutablepropertyextensions.setkeyvaluecomparer) in the rare cases where different semantics is required on the same property.</span></span>

> [!NOTE]  
> <span data-ttu-id="8dfe2-175">EF Core 5.0 中的 SetStructuralComparer 已过时。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-175">SetStructuralComparer has been obsoleted in EF Core 5.0.</span></span>
> <span data-ttu-id="8dfe2-176">改用 SetKeyValueComparer。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-176">Use SetKeyValueComparer instead.</span></span>

### <a name="overriding-defaults"></a><span data-ttu-id="8dfe2-177">替代默认值</span><span class="sxs-lookup"><span data-stu-id="8dfe2-177">Overriding defaults</span></span>

<span data-ttu-id="8dfe2-178">有时 EF Core 使用的默认比较可能不合适。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-178">Sometimes the default comparison used by EF Core may not be appropriate.</span></span>
<span data-ttu-id="8dfe2-179">例如，默认情况下，不会在 EF Core 中检测到字节数组的变化。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-179">For example, mutation of byte arrays is not, by default, detected in EF Core.</span></span>
<span data-ttu-id="8dfe2-180">这可以通过对属性设置不同的比较器来重写：</span><span class="sxs-lookup"><span data-stu-id="8dfe2-180">This can be overridden by setting a different comparer on the property:</span></span>

[!code-csharp[OverrideComparer](../../../samples/core/Modeling/ValueConversions/OverridingByteArrayComparisons.cs?name=OverrideComparer)]

<span data-ttu-id="8dfe2-181">EF Core 现在会比较字节序列并因此将检测字节数组突变。</span><span class="sxs-lookup"><span data-stu-id="8dfe2-181">EF Core will now compare byte sequences and will therefore detect byte array mutations.</span></span>
