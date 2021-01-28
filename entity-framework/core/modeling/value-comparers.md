---
title: 值比较器-EF Core
description: 使用值比较器来控制 EF Core 比较属性值的方式
author: ajcvickers
ms.date: 01/16/2021
uid: core/modeling/value-comparers
ms.openlocfilehash: 5c5e5beee72230a331a8e1c88a2020dc5ad88ecf
ms.sourcegitcommit: 7700840119b1639275f3b64836e7abb59103f2e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2021
ms.locfileid: "98983477"
---
# <a name="value-comparers"></a><span data-ttu-id="40322-103">值比较器</span><span class="sxs-lookup"><span data-stu-id="40322-103">Value Comparers</span></span>

> [!NOTE]
> <span data-ttu-id="40322-104">EF Core 3.0 中已引入此功能。</span><span class="sxs-lookup"><span data-stu-id="40322-104">This feature was introduced in EF Core 3.0.</span></span>

> [!TIP]
> <span data-ttu-id="40322-105">可在 GitHub 上找到此文档中的代码作为可 [运行示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/ValueConversions/)。</span><span class="sxs-lookup"><span data-stu-id="40322-105">The code in this document can be found on GitHub as a [runnable sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Modeling/ValueConversions/).</span></span>

## <a name="background"></a><span data-ttu-id="40322-106">背景</span><span class="sxs-lookup"><span data-stu-id="40322-106">Background</span></span>

<span data-ttu-id="40322-107">[更改跟踪](xref:core/change-tracking/index) 是指 EF Core 自动确定已加载的实体实例上的应用程序所执行的更改，以便在调用时可以将这些更改保存回数据库 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。</span><span class="sxs-lookup"><span data-stu-id="40322-107">[Change tracking](xref:core/change-tracking/index) means that EF Core automatically determines what changes were performed by the application on a loaded entity instance, so that those changes can be saved back to the database when <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> is called.</span></span> <span data-ttu-id="40322-108">EF Core 通常会在从数据库中加载实例时获取实例的 *快照* ，并将该快照与向应用程序传递的实例进行 *比较* 。</span><span class="sxs-lookup"><span data-stu-id="40322-108">EF Core usually performs this by taking a *snapshot* of the instance when it's loaded from the database, and *comparing* that snapshot to the instance handed out to the application.</span></span>

<span data-ttu-id="40322-109">EF Core 附带了用于快照和比较数据库中使用的大多数标准类型的内置逻辑，因此用户通常不需要担心本主题。</span><span class="sxs-lookup"><span data-stu-id="40322-109">EF Core comes with built-in logic for snapshotting and comparing most standard types used in databases, so users don't usually need to worry about this topic.</span></span> <span data-ttu-id="40322-110">但是，当通过 [值转换器](xref:core/modeling/value-conversions)映射属性时，EF Core 需要对任意用户类型执行比较，这可能很复杂。</span><span class="sxs-lookup"><span data-stu-id="40322-110">However, when a property is mapped through a [value converter](xref:core/modeling/value-conversions), EF Core needs to perform comparison on arbitrary user types, which may be complex.</span></span> <span data-ttu-id="40322-111">默认情况下，EF Core 使用由类型定义的默认相等比较 (例如 `Equals` ，) 的方法; 对于快照，会复制 [值类型](/dotnet/csharp/language-reference/builtin-types/value-types) 以生成快照，而对于 [引用类型](/dotnet/csharp/language-reference/keywords/reference-types) ，则不进行复制，同一实例用作快照。</span><span class="sxs-lookup"><span data-stu-id="40322-111">By default, EF Core uses the default equality comparison defined by types (e.g. the `Equals` method); for snapshotting, [value types](/dotnet/csharp/language-reference/builtin-types/value-types) are copied to produce the snapshot, while for [reference types](/dotnet/csharp/language-reference/keywords/reference-types) no copying occurs, and the same instance is used as the snapshot.</span></span>

<span data-ttu-id="40322-112">如果内置的比较行为不合适，则用户可以提供 *值比较器*，其中包含快照的逻辑、比较和计算哈希代码。</span><span class="sxs-lookup"><span data-stu-id="40322-112">In cases where the built-in comparison behavior isn't appropriate, users may provide a *value comparer*, which contains logic for snapshotting, comparing and calculating a hash code.</span></span> <span data-ttu-id="40322-113">例如，下面的属性的值转换设置为要 `List<int>` 转换为数据库中的 JSON 字符串的值，并定义适当的值比较器。</span><span class="sxs-lookup"><span data-stu-id="40322-113">For example, the following sets up value conversion for `List<int>` property to be value converted to a JSON string in the database, and defines an appropriate value comparer as well:</span></span>

[!code-csharp[ListProperty](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ConfigureListProperty)]

<span data-ttu-id="40322-114">有关更多详细信息，请参阅下面的 [可变类](#mutable-classes) 。</span><span class="sxs-lookup"><span data-stu-id="40322-114">See [mutable classes](#mutable-classes) below for further details.</span></span>

<span data-ttu-id="40322-115">请注意，在确定解析关系时两个键值是否相同时，还会使用值比较器。下面对此进行了说明。</span><span class="sxs-lookup"><span data-stu-id="40322-115">Note that value comparers are also used when determining whether two key values are the same when resolving relationships; this is explained below.</span></span>

## <a name="shallow-vs-deep-comparison"></a><span data-ttu-id="40322-116">浅比较与深层比较</span><span class="sxs-lookup"><span data-stu-id="40322-116">Shallow vs. deep comparison</span></span>

<span data-ttu-id="40322-117">对于不变的可变值类型（例如 `int` ），EF Core 的默认逻辑工作正常：在进行快照时，值将按原样复制，并与类型的内置相等比较进行比较。</span><span class="sxs-lookup"><span data-stu-id="40322-117">For small, immutable value types such as `int`, EF Core's default logic works well: the value is copied as-is when snapshotted, and compared with the type's built-in equality comparison.</span></span> <span data-ttu-id="40322-118">实现自己的值比较器时，请务必考虑深度或浅比较 (和快照) 逻辑是否合适。</span><span class="sxs-lookup"><span data-stu-id="40322-118">When implementing your own value comparer, it's important to consider whether deep or shallow comparison (and snapshotting) logic is appropriate.</span></span>

<span data-ttu-id="40322-119">请考虑字节数组，该数组可以任意大。</span><span class="sxs-lookup"><span data-stu-id="40322-119">Consider byte arrays, which can be arbitrarily large.</span></span> <span data-ttu-id="40322-120">可以比较以下内容：</span><span class="sxs-lookup"><span data-stu-id="40322-120">These could be compared:</span></span>

* <span data-ttu-id="40322-121">通过引用，因此只有在使用新的字节数组时才会检测到差异</span><span class="sxs-lookup"><span data-stu-id="40322-121">By reference, such that a difference is only detected if a new byte array is used</span></span>
* <span data-ttu-id="40322-122">进行深层比较时，将检测到数组中字节的变化</span><span class="sxs-lookup"><span data-stu-id="40322-122">By deep comparison, such that mutation of the bytes in the array is detected</span></span>

<span data-ttu-id="40322-123">默认情况下，EF Core 使用这些方法中的第一个方法来实现非键字节数组。</span><span class="sxs-lookup"><span data-stu-id="40322-123">By default, EF Core uses the first of these approaches for non-key byte arrays.</span></span> <span data-ttu-id="40322-124">也就是说，仅对引用进行比较，并且仅当现有字节数组替换为新的字节数组时，才会检测到更改。</span><span class="sxs-lookup"><span data-stu-id="40322-124">That is, only references are compared and a change is detected only when an existing byte array is replaced with a new one.</span></span> <span data-ttu-id="40322-125">这是一项切实的决策，可避免在执行时复制整个数组，并将它们与字节进行比较 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。</span><span class="sxs-lookup"><span data-stu-id="40322-125">This is a pragmatic decision that avoids copying entire arrays and comparing them byte-to-byte when executing <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A>.</span></span> <span data-ttu-id="40322-126">这意味着，将一个图像替换为另一个映像的常见方案是以高性能的方式进行处理。</span><span class="sxs-lookup"><span data-stu-id="40322-126">It means that the common scenario of replacing, say, one image with another is handled in a performant way.</span></span>

<span data-ttu-id="40322-127">另一方面，如果使用字节数组表示二进制键，则引用相等性将不起作用，因为 FK 属性不太可能设置为与需要进行比较的 PK 属性 _相同的实例_ 。</span><span class="sxs-lookup"><span data-stu-id="40322-127">On the other hand, reference equality would not work when byte arrays are used to represent binary keys, since it's very unlikely that an FK property is set to the _same instance_ as a PK property to which it needs to be compared.</span></span> <span data-ttu-id="40322-128">因此，EF Core 对用作键的字节数组使用深层比较;这不太可能会产生大的性能，因为二进制密钥通常很短。</span><span class="sxs-lookup"><span data-stu-id="40322-128">Therefore, EF Core uses deep comparisons for byte arrays acting as keys; this is unlikely to have a big performance hit since binary keys are usually short.</span></span>

<span data-ttu-id="40322-129">请注意，所选的比较和快照逻辑必须彼此对应： deep 比较要求深度快照才能正常工作。</span><span class="sxs-lookup"><span data-stu-id="40322-129">Note that the chosen comparison and snapshotting logic must correspond to each other: deep comparison requires deep snapshotting to function correctly.</span></span>

## <a name="simple-immutable-classes"></a><span data-ttu-id="40322-130">简单的不可变类</span><span class="sxs-lookup"><span data-stu-id="40322-130">Simple immutable classes</span></span>

<span data-ttu-id="40322-131">考虑使用值转换器映射简单的不可变类的属性。</span><span class="sxs-lookup"><span data-stu-id="40322-131">Consider a property that uses a value converter to map a simple, immutable class.</span></span>

[!code-csharp[SimpleImmutableClass](../../../samples/core/Modeling/ValueConversions/MappingImmutableClassProperty.cs?name=SimpleImmutableClass)]

[!code-csharp[ConfigureImmutableClassProperty](../../../samples/core/Modeling/ValueConversions/MappingImmutableClassProperty.cs?name=ConfigureImmutableClassProperty)]

<span data-ttu-id="40322-132">此类型的属性不需要特殊比较或快照，原因如下：</span><span class="sxs-lookup"><span data-stu-id="40322-132">Properties of this type do not need special comparisons or snapshots because:</span></span>

* <span data-ttu-id="40322-133">重写相等性，以使不同的实例正确比较</span><span class="sxs-lookup"><span data-stu-id="40322-133">Equality is overridden so that different instances will compare correctly</span></span>
* <span data-ttu-id="40322-134">类型是不可变的，因此不可能改变快照值</span><span class="sxs-lookup"><span data-stu-id="40322-134">The type is immutable, so there is no chance of mutating a snapshot value</span></span>

<span data-ttu-id="40322-135">因此，在这种情况下，EF Core 的默认行为是正确的。</span><span class="sxs-lookup"><span data-stu-id="40322-135">So in this case the default behavior of EF Core is fine as it is.</span></span>

## <a name="simple-immutable-structs"></a><span data-ttu-id="40322-136">简单的不可变结构</span><span class="sxs-lookup"><span data-stu-id="40322-136">Simple immutable structs</span></span>

<span data-ttu-id="40322-137">简单结构的映射也很简单，并且不需要特殊的比较器或快照。</span><span class="sxs-lookup"><span data-stu-id="40322-137">The mapping for simple structs is also simple and requires no special comparers or snapshotting.</span></span>

[!code-csharp[SimpleImmutableStruct](../../../samples/core/Modeling/ValueConversions/MappingImmutableStructProperty.cs?name=SimpleImmutableStruct)]

[!code-csharp[ConfigureImmutableStructProperty](../../../samples/core/Modeling/ValueConversions/MappingImmutableStructProperty.cs?name=ConfigureImmutableStructProperty)]

<span data-ttu-id="40322-138">EF Core 提供对结构属性的编译按成员比较的内置支持。</span><span class="sxs-lookup"><span data-stu-id="40322-138">EF Core has built-in support for generating compiled, memberwise comparisons of struct properties.</span></span> <span data-ttu-id="40322-139">这意味着结构无需重写 EF Core 的相等性，但出于 [其他原因](/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-define-value-equality-for-a-type)，你仍可以选择执行此操作。</span><span class="sxs-lookup"><span data-stu-id="40322-139">This means structs don't need to have equality overridden for EF Core, but you may still choose to do this for [other reasons](/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-define-value-equality-for-a-type).</span></span> <span data-ttu-id="40322-140">此外，不需要特殊快照，因为结构是不可变的，并且始终复制按成员。</span><span class="sxs-lookup"><span data-stu-id="40322-140">Also, special snapshotting is not needed since structs are immutable and are always copied memberwise anyway.</span></span> <span data-ttu-id="40322-141"> (对于可变结构也是如此，但 [通常应避免可变结构](/dotnet/csharp/write-safe-efficient-code)。 ) </span><span class="sxs-lookup"><span data-stu-id="40322-141">(This is also true for mutable structs, but [mutable structs should in general be avoided](/dotnet/csharp/write-safe-efficient-code).)</span></span>

## <a name="mutable-classes"></a><span data-ttu-id="40322-142">可变类</span><span class="sxs-lookup"><span data-stu-id="40322-142">Mutable classes</span></span>

<span data-ttu-id="40322-143">建议尽可能使用值转换器 (类或结构) 的不可变类型。</span><span class="sxs-lookup"><span data-stu-id="40322-143">It is recommended that you use immutable types (classes or structs) with value converters when possible.</span></span> <span data-ttu-id="40322-144">这通常更高效，并具有比使用可变类型更清晰的语义。</span><span class="sxs-lookup"><span data-stu-id="40322-144">This is usually more efficient and has cleaner semantics than using a mutable type.</span></span> <span data-ttu-id="40322-145">但这种情况下，通常使用应用程序无法更改的类型的属性。</span><span class="sxs-lookup"><span data-stu-id="40322-145">However, that being said, it is common to use properties of types that the application cannot change.</span></span> <span data-ttu-id="40322-146">例如，映射包含数字列表的属性：</span><span class="sxs-lookup"><span data-stu-id="40322-146">For example, mapping a property containing a list of numbers:</span></span>

[!code-csharp[ListProperty](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ListProperty)]

<span data-ttu-id="40322-147"><xref:System.Collections.Generic.List%601> 类：</span><span class="sxs-lookup"><span data-stu-id="40322-147">The <xref:System.Collections.Generic.List%601> class:</span></span>

* <span data-ttu-id="40322-148">具有引用相等性;包含相同值的两个列表被视为不同的。</span><span class="sxs-lookup"><span data-stu-id="40322-148">Has reference equality; two lists containing the same values are treated as different.</span></span>
* <span data-ttu-id="40322-149">可变;可以添加和删除列表中的值。</span><span class="sxs-lookup"><span data-stu-id="40322-149">Is mutable; values in the list can be added and removed.</span></span>

<span data-ttu-id="40322-150">列表属性上的典型值转换可能会在列表与 JSON 之间进行转换：</span><span class="sxs-lookup"><span data-stu-id="40322-150">A typical value conversion on a list property might convert the list to and from JSON:</span></span>

### <a name="ef-core-50"></a>[<span data-ttu-id="40322-151">EF Core 5.0</span><span class="sxs-lookup"><span data-stu-id="40322-151">EF Core 5.0</span></span>](#tab/ef5)

[!code-csharp[ListProperty](../../../samples/core/Modeling/ValueConversions/MappingListProperty.cs?name=ConfigureListProperty&highlight=7-10)]

### <a name="older-versions"></a>[<span data-ttu-id="40322-152">旧版本</span><span class="sxs-lookup"><span data-stu-id="40322-152">Older versions</span></span>](#tab/older-versions)

[!code-csharp[ListProperty](../../../samples/core/Modeling/ValueConversions/MappingListPropertyOld.cs?name=ConfigureListProperty&highlight=8-11,17)]

***

<span data-ttu-id="40322-153"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601>构造函数接受三个表达式：</span><span class="sxs-lookup"><span data-stu-id="40322-153">The <xref:Microsoft.EntityFrameworkCore.ChangeTracking.ValueComparer%601> constructor accepts three expressions:</span></span>

* <span data-ttu-id="40322-154">用于检查相等性的表达式</span><span class="sxs-lookup"><span data-stu-id="40322-154">An expression for checking equality</span></span>
* <span data-ttu-id="40322-155">用于生成哈希代码的表达式</span><span class="sxs-lookup"><span data-stu-id="40322-155">An expression for generating a hash code</span></span>
* <span data-ttu-id="40322-156">用于对值进行快照的表达式</span><span class="sxs-lookup"><span data-stu-id="40322-156">An expression to snapshot a value</span></span>

<span data-ttu-id="40322-157">在这种情况下，比较是通过检查数字序列是否相同来完成的。</span><span class="sxs-lookup"><span data-stu-id="40322-157">In this case the comparison is done by checking if the sequences of numbers are the same.</span></span>

<span data-ttu-id="40322-158">同样，哈希代码是根据此相同的顺序生成的。</span><span class="sxs-lookup"><span data-stu-id="40322-158">Likewise, the hash code is built from this same sequence.</span></span> <span data-ttu-id="40322-159"> (请注意，这是一个基于可变值的哈希代码，因此可能 [会导致问题](https://ericlippert.com/2011/02/28/guidelines-and-rules-for-gethashcode/)。</span><span class="sxs-lookup"><span data-stu-id="40322-159">(Note that this is a hash code over mutable values and hence can [cause problems](https://ericlippert.com/2011/02/28/guidelines-and-rules-for-gethashcode/).</span></span> <span data-ttu-id="40322-160">如果可以，则改为不可变。 ) </span><span class="sxs-lookup"><span data-stu-id="40322-160">Be immutable instead if you can.)</span></span>

<span data-ttu-id="40322-161">通过使用克隆列表创建快照 `ToList` 。</span><span class="sxs-lookup"><span data-stu-id="40322-161">The snapshot is created by cloning the list with `ToList`.</span></span> <span data-ttu-id="40322-162">同样，仅当要转变列表时，才需要这样做。</span><span class="sxs-lookup"><span data-stu-id="40322-162">Again, this is only needed if the lists are going to be mutated.</span></span> <span data-ttu-id="40322-163">如果可以，则改为不可变。</span><span class="sxs-lookup"><span data-stu-id="40322-163">Be immutable instead if you can.</span></span>

> [!NOTE]
> <span data-ttu-id="40322-164">值转换器和比较器使用表达式而不是简单委托来构造。</span><span class="sxs-lookup"><span data-stu-id="40322-164">Value converters and comparers are constructed using expressions rather than simple delegates.</span></span> <span data-ttu-id="40322-165">这是因为 EF Core 会将这些表达式插入到更复杂的表达式树中，然后将其编译到实体整形程序委托中。</span><span class="sxs-lookup"><span data-stu-id="40322-165">This is because EF Core inserts these expressions into a much more complex expression tree that is then compiled into an entity shaper delegate.</span></span> <span data-ttu-id="40322-166">从概念上讲，这类似于编译器内联。</span><span class="sxs-lookup"><span data-stu-id="40322-166">Conceptually, this is similar to compiler inlining.</span></span> <span data-ttu-id="40322-167">例如，简单转换可能是在强制转换中编译的，而不是调用其他方法来执行转换。</span><span class="sxs-lookup"><span data-stu-id="40322-167">For example, a simple conversion may just be a compiled in cast, rather than a call to another method to do the conversion.</span></span>

## <a name="key-comparers"></a><span data-ttu-id="40322-168">密钥比较器</span><span class="sxs-lookup"><span data-stu-id="40322-168">Key comparers</span></span>

<span data-ttu-id="40322-169">背景部分介绍了为何密钥比较可能需要特殊语义。</span><span class="sxs-lookup"><span data-stu-id="40322-169">The background section covers why key comparisons may require special semantics.</span></span> <span data-ttu-id="40322-170">在 primary、principal 或 foreign key 属性上设置键时，请确保创建的比较器适用于键。</span><span class="sxs-lookup"><span data-stu-id="40322-170">Make sure to create a comparer that is appropriate for keys when setting it on a primary, principal, or foreign key property.</span></span>

<span data-ttu-id="40322-171"><xref:Microsoft.EntityFrameworkCore.MutablePropertyExtensions.SetKeyValueComparer%2A>在相同的属性上需要不同语义的罕见情况下使用。</span><span class="sxs-lookup"><span data-stu-id="40322-171">Use <xref:Microsoft.EntityFrameworkCore.MutablePropertyExtensions.SetKeyValueComparer%2A> in the rare cases where different semantics is required on the same property.</span></span>

> [!NOTE]
> <span data-ttu-id="40322-172"><xref:Microsoft.EntityFrameworkCore.MutablePropertyExtensions.SetStructuralValueComparer%2A> 在 EF Core 5.0 中已过时。</span><span class="sxs-lookup"><span data-stu-id="40322-172"><xref:Microsoft.EntityFrameworkCore.MutablePropertyExtensions.SetStructuralValueComparer%2A> has been obsoleted in EF Core 5.0.</span></span> <span data-ttu-id="40322-173">请改用 <xref:Microsoft.EntityFrameworkCore.MutablePropertyExtensions.SetKeyValueComparer%2A>。</span><span class="sxs-lookup"><span data-stu-id="40322-173">Use <xref:Microsoft.EntityFrameworkCore.MutablePropertyExtensions.SetKeyValueComparer%2A> instead.</span></span>

## <a name="overriding-the-default-comparer"></a><span data-ttu-id="40322-174">重写默认比较器</span><span class="sxs-lookup"><span data-stu-id="40322-174">Overriding the default comparer</span></span>

<span data-ttu-id="40322-175">有时 EF Core 使用的默认比较可能不合适。</span><span class="sxs-lookup"><span data-stu-id="40322-175">Sometimes the default comparison used by EF Core may not be appropriate.</span></span> <span data-ttu-id="40322-176">例如，默认情况下，不会在 EF Core 中检测到字节数组的变化。</span><span class="sxs-lookup"><span data-stu-id="40322-176">For example, mutation of byte arrays is not, by default, detected in EF Core.</span></span> <span data-ttu-id="40322-177">这可以通过对属性设置不同的比较器来重写：</span><span class="sxs-lookup"><span data-stu-id="40322-177">This can be overridden by setting a different comparer on the property:</span></span>

[!code-csharp[OverrideComparer](../../../samples/core/Modeling/ValueConversions/OverridingByteArrayComparisons.cs?name=OverrideComparer)]

<span data-ttu-id="40322-178">EF Core 现在会比较字节序列并因此将检测字节数组突变。</span><span class="sxs-lookup"><span data-stu-id="40322-178">EF Core will now compare byte sequences and will therefore detect byte array mutations.</span></span>
