---
title: 附加更改跟踪功能-EF Core
description: 涉及 EF Core 更改跟踪的其他功能和方案
author: ajcvickers
ms.date: 12/30/2020
uid: core/change-tracking/miscellaneous
ms.openlocfilehash: db1e32948b2a60ad1b85e300bbbccd54d49a84e5
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129581"
---
# <a name="additional-change-tracking-features"></a><span data-ttu-id="e31bf-103">其他更改跟踪功能</span><span class="sxs-lookup"><span data-stu-id="e31bf-103">Additional Change Tracking Features</span></span>

<span data-ttu-id="e31bf-104">本文档介绍涉及更改跟踪的其他功能和方案。</span><span class="sxs-lookup"><span data-stu-id="e31bf-104">This document covers miscellaneous features and scenarios involving change tracking.</span></span>

> [!TIP]
> <span data-ttu-id="e31bf-105">本文档假设了解 EF Core 更改跟踪的实体状态和基础知识。</span><span class="sxs-lookup"><span data-stu-id="e31bf-105">This document assumes that entity states and the basics of EF Core change tracking are understood.</span></span> <span data-ttu-id="e31bf-106">有关这些主题的详细信息，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。</span><span class="sxs-lookup"><span data-stu-id="e31bf-106">See [Change Tracking in EF Core](xref:core/change-tracking/index) for more information on these topics.</span></span>

> [!TIP]
> <span data-ttu-id="e31bf-107">通过 [从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/AdditionalChangeTrackingFeatures)，你可以运行并调试到本文档中的所有代码。</span><span class="sxs-lookup"><span data-stu-id="e31bf-107">You can run and debug into all the code in this document by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/ChangeTracking/AdditionalChangeTrackingFeatures).</span></span>

## <a name="add-verses-addasync"></a><span data-ttu-id="e31bf-108">添加辞 AddAsync</span><span class="sxs-lookup"><span data-stu-id="e31bf-108">Add verses AddAsync</span></span>

<span data-ttu-id="e31bf-109">Entity Framework Core (EF Core) 在使用该方法时提供异步方法可能会导致数据库交互。</span><span class="sxs-lookup"><span data-stu-id="e31bf-109">Entity Framework Core (EF Core) provides async methods whenever using that method may result in a database interaction.</span></span> <span data-ttu-id="e31bf-110">还提供了同步方法，以避免使用不支持高性能异步访问的数据库时的开销。</span><span class="sxs-lookup"><span data-stu-id="e31bf-110">Synchronous methods are also provided to avoid overhead when using databases that do not support high performance asynchronous access.</span></span>

<span data-ttu-id="e31bf-111"><xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType><xref:Microsoft.EntityFrameworkCore.DbSet%601.Add%2A?displayProperty=nameWithType>通常不会访问数据库，因为这些方法本质上只是开始跟踪实体。</span><span class="sxs-lookup"><span data-stu-id="e31bf-111"><xref:Microsoft.EntityFrameworkCore.DbContext.Add%2A?displayProperty=nameWithType> and <xref:Microsoft.EntityFrameworkCore.DbSet%601.Add%2A?displayProperty=nameWithType> do not normally access the database, since these methods inherently just start tracking entities.</span></span> <span data-ttu-id="e31bf-112">但某些形式的值生成 _可以_ 访问数据库，以便生成键值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-112">However, some forms of value generation _may_ access the database in order to generate a key value.</span></span> <span data-ttu-id="e31bf-113">唯一这样做并随 EF Core 提供的值生成器为 <xref:Microsoft.EntityFrameworkCore.ValueGeneration.HiLoValueGenerator%601> 。</span><span class="sxs-lookup"><span data-stu-id="e31bf-113">The only value generator that does this and ships with EF Core is <xref:Microsoft.EntityFrameworkCore.ValueGeneration.HiLoValueGenerator%601>.</span></span> <span data-ttu-id="e31bf-114">使用此生成器不常见;默认情况下，它不配置。</span><span class="sxs-lookup"><span data-stu-id="e31bf-114">Using this generator is uncommon; it is never configured by default.</span></span> <span data-ttu-id="e31bf-115">这意味着大多数应用程序应使用 `Add` ，而不是 `AddAsync` 。</span><span class="sxs-lookup"><span data-stu-id="e31bf-115">This means that the vast majority of applications should use `Add`, and not `AddAsync`.</span></span>

<span data-ttu-id="e31bf-116">诸如、和之类的其他类似方法没有 `Update` `Attach` `Remove` 异步重载，因为它们永远不会生成新的键值，因此永远不需要访问数据库。</span><span class="sxs-lookup"><span data-stu-id="e31bf-116">Other similar methods like `Update`, `Attach`, and `Remove` do not have async overloads because they never generate new key values, and hence never need to access the database.</span></span>

## <a name="addrange-updaterange-attachrange-and-removerange"></a><span data-ttu-id="e31bf-117">AddRange、UpdateRange、AttachRange 和 RemoveRange</span><span class="sxs-lookup"><span data-stu-id="e31bf-117">AddRange, UpdateRange, AttachRange, and RemoveRange</span></span>

<span data-ttu-id="e31bf-118"><xref:Microsoft.EntityFrameworkCore.DbSet%601> 和 <xref:Microsoft.EntityFrameworkCore.DbContext> 提供了、、和的备用版本 `Add` ， `Update` `Attach` `Remove` 可在单个调用中接受多个实例。</span><span class="sxs-lookup"><span data-stu-id="e31bf-118"><xref:Microsoft.EntityFrameworkCore.DbSet%601> and <xref:Microsoft.EntityFrameworkCore.DbContext> provide alternate versions of `Add`, `Update`, `Attach`, and `Remove` that accept multiple instances in a single call.</span></span> <span data-ttu-id="e31bf-119">这些方法分别称为 `AddRange` 、 `UpdateRange` 、 `AttachRange` 和 `RemoveRange` 。</span><span class="sxs-lookup"><span data-stu-id="e31bf-119">These methods are called `AddRange`, `UpdateRange`, `AttachRange`, and `RemoveRange` respectively.</span></span>

<span data-ttu-id="e31bf-120">这些方法是提供方便的。</span><span class="sxs-lookup"><span data-stu-id="e31bf-120">These methods are provided as a convenience.</span></span> <span data-ttu-id="e31bf-121">使用 "范围" 方法与对等效的非范围方法的多个调用具有相同的功能。</span><span class="sxs-lookup"><span data-stu-id="e31bf-121">Using a "range" method has the same functionality as multiple calls to the equivalent non-range method.</span></span> <span data-ttu-id="e31bf-122">这两种方法之间没有明显的性能差异。</span><span class="sxs-lookup"><span data-stu-id="e31bf-122">There is no significant performance difference between the two approaches.</span></span>

> [!NOTE]
> <span data-ttu-id="e31bf-123">这不同于 EF6，其中 AddRange 并添加了自动调用的 DetectChanges，但调用 Add 多次导致 DetectChanges 多次调用，而不是一次。</span><span class="sxs-lookup"><span data-stu-id="e31bf-123">This is different from EF6, where AddRange and Add both automatically called DetectChanges, but calling Add multiple times caused DetectChanges to be called multiple times instead of once.</span></span> <span data-ttu-id="e31bf-124">这使 AddRange 在 EF6 中更有效。</span><span class="sxs-lookup"><span data-stu-id="e31bf-124">This made AddRange more efficient in EF6.</span></span> <span data-ttu-id="e31bf-125">在 EF Core 中，这两种方法都不会自动调用 DetectChanges。</span><span class="sxs-lookup"><span data-stu-id="e31bf-125">In EF Core, neither of these methods automatically call DetectChanges.</span></span>

## <a name="dbcontext-verses-dbset-methods"></a><span data-ttu-id="e31bf-126">DbContext 辞句 DbSet 方法</span><span class="sxs-lookup"><span data-stu-id="e31bf-126">DbContext verses DbSet methods</span></span>

<span data-ttu-id="e31bf-127">许多方法（包括 `Add` 、 `Update` 、 `Attach` 和 `Remove` ）在和上都具有 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 实现 <xref:Microsoft.EntityFrameworkCore.DbContext> 。</span><span class="sxs-lookup"><span data-stu-id="e31bf-127">Many methods, including `Add`, `Update`, `Attach`, and `Remove`, have implementations on both <xref:Microsoft.EntityFrameworkCore.DbSet%601> and <xref:Microsoft.EntityFrameworkCore.DbContext>.</span></span> <span data-ttu-id="e31bf-128">对于普通实体类型，这些方法具有 _完全相同的行为_ 。</span><span class="sxs-lookup"><span data-stu-id="e31bf-128">These methods have _exactly the same behavior_ for normal entity types.</span></span> <span data-ttu-id="e31bf-129">这是因为实体的 CLR 类型会映射到 EF Core 模型中的一个实体类型。</span><span class="sxs-lookup"><span data-stu-id="e31bf-129">This is because the CLR type of the entity is mapped onto one and only one entity type in the EF Core model.</span></span> <span data-ttu-id="e31bf-130">因此，CLR 类型会完全定义实体在模型中的适合位置，因此，可以隐式确定要使用的 DbSet。</span><span class="sxs-lookup"><span data-stu-id="e31bf-130">Therefore, the CLR type fully defines where the entity fits in the model, and so the DbSet to use can be determined implicitly.</span></span>

<span data-ttu-id="e31bf-131">此规则的例外情况是使用在 EF Core 5.0 中引入的共享类型实体类型，主要用于多对多联接实体。</span><span class="sxs-lookup"><span data-stu-id="e31bf-131">The exception to this rule is when using shared-type entity types, which were introduced in EF Core 5.0, primarily for many-to-many join entities.</span></span> <span data-ttu-id="e31bf-132">使用共享类型的实体类型时，必须首先为正在使用的 EF Core 模型类型创建 DbSet。</span><span class="sxs-lookup"><span data-stu-id="e31bf-132">When using a shared-type entity type, a DbSet must first be created for the EF Core model type that is being used.</span></span> <span data-ttu-id="e31bf-133">`Add`然后，可以在 DbSet 上使用类似于、、和的方法， `Update` `Attach` `Remove` 而不会有任何不明确的方法可用于所使用的 EF Core 模型类型。</span><span class="sxs-lookup"><span data-stu-id="e31bf-133">Methods like `Add`, `Update`, `Attach`, and `Remove` can then be used on the DbSet without any ambiguity as to which EF Core model type is being used.</span></span>

<span data-ttu-id="e31bf-134">默认情况下，对于多对多关系中的联接实体，将使用共享类型的实体类型。</span><span class="sxs-lookup"><span data-stu-id="e31bf-134">Shared-type entity types are used by default for the join entities in many-to-many relationships.</span></span> <span data-ttu-id="e31bf-135">还可以显式配置共享类型的实体类型，以便在多对多关系中使用。</span><span class="sxs-lookup"><span data-stu-id="e31bf-135">A shared-type entity type can also be explicitly configured for use in a many-to-many relationship.</span></span> <span data-ttu-id="e31bf-136">例如，下面的代码将配置 `Dictionary<string, int>` 为联接实体类型：</span><span class="sxs-lookup"><span data-stu-id="e31bf-136">For example, the code below configures `Dictionary<string, int>` as a join entity type:</span></span>

<!--
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder
                .SharedTypeEntity<Dictionary<string, int>>(
                    "PostTag",
                    b =>
                        {
                            b.IndexerProperty<int>("TagId");
                            b.IndexerProperty<int>("PostId");
                        });

            modelBuilder.Entity<Post>()
                .HasMany(p => p.Tags)
                .WithMany(p => p.Posts)
                .UsingEntity<Dictionary<string, int>>(
                    "PostTag",
                    j => j.HasOne<Tag>().WithMany(),
                    j => j.HasOne<Post>().WithMany());
        }
-->
[!code-csharp[OnModelCreating](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/Samples.cs?name=OnModelCreating)]

<span data-ttu-id="e31bf-137">[更改外键和导航](xref:core/change-tracking/relationship-changes) 显示了如何通过跟踪新的联接实体实例来关联两个实体。</span><span class="sxs-lookup"><span data-stu-id="e31bf-137">[Changing Foreign Keys and Navigations](xref:core/change-tracking/relationship-changes) shows how to associate two entities by tracking a new join entity instance.</span></span> <span data-ttu-id="e31bf-138">下面的代码针对 `Dictionary<string, int>` 用于联接实体的共享类型实体类型执行此操作：</span><span class="sxs-lookup"><span data-stu-id="e31bf-138">The code below does this for the `Dictionary<string, int>` shared-type entity type used for the join entity:</span></span>

<!--
            using var context = new BlogsContext();

            var post = context.Posts.Single(e => e.Id == 3);
            var tag = context.Tags.Single(e => e.Id == 1);

            var joinEntitySet = context.Set<Dictionary<string, int>>("PostTag");
            var joinEntity = new Dictionary<string, int>
            {
                ["PostId"] = post.Id,
                ["TagId"] = tag.Id
            };
            joinEntitySet.Add(joinEntity);

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);

            context.SaveChanges();
-->
[!code-csharp[DbContext_verses_DbSet_methods_1](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/Samples.cs?name=DbContext_verses_DbSet_methods_1)]

<span data-ttu-id="e31bf-139">请注意， <xref:Microsoft.EntityFrameworkCore.DbContext.Set%60%601(System.String)?displayProperty=nameWithType> 用于为 `PostTag` 实体类型创建 DbSet。</span><span class="sxs-lookup"><span data-stu-id="e31bf-139">Notice that <xref:Microsoft.EntityFrameworkCore.DbContext.Set%60%601(System.String)?displayProperty=nameWithType> is used to create a DbSet for the `PostTag` entity type.</span></span> <span data-ttu-id="e31bf-140">然后，可以使用此 DbSet 调用 `Add` 新的联接实体实例。</span><span class="sxs-lookup"><span data-stu-id="e31bf-140">This DbSet can then be used to call `Add` with the new join entity instance.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="e31bf-141">用于按约定联接实体类型的 CLR 类型在将来的版本中可能会更改以提高性能。</span><span class="sxs-lookup"><span data-stu-id="e31bf-141">The CLR type used for join entity types by convention may change in future releases to improve performance.</span></span> <span data-ttu-id="e31bf-142">不依赖于任何特定的联接实体类型，除非已将其显式配置为 `Dictionary<string, int>` 在上面的代码中完成。</span><span class="sxs-lookup"><span data-stu-id="e31bf-142">Do not depend on any specific join entity type unless it has been explicitly configured as is done for `Dictionary<string, int>` in the code above.</span></span>

## <a name="property-verses-field-access"></a><span data-ttu-id="e31bf-143">属性辞字段访问</span><span class="sxs-lookup"><span data-stu-id="e31bf-143">Property verses field access</span></span>

<span data-ttu-id="e31bf-144">从 EF Core 3.0 开始，默认情况下，对实体属性的访问将使用属性的支持字段。</span><span class="sxs-lookup"><span data-stu-id="e31bf-144">Starting with EF Core 3.0, access to entity properties uses the backing field of the property by default.</span></span> <span data-ttu-id="e31bf-145">这是高效的，避免了调用属性 getter 和 setter 的副作用。</span><span class="sxs-lookup"><span data-stu-id="e31bf-145">This is efficient and avoids triggering side effects from calling property getters and setters.</span></span> <span data-ttu-id="e31bf-146">例如，这是延迟加载如何避免触发无限循环。</span><span class="sxs-lookup"><span data-stu-id="e31bf-146">For example, this is how lazy-loading is able to avoid triggering infinite loops.</span></span> <span data-ttu-id="e31bf-147">有关在模型中配置支持字段的详细信息，请参阅 [支持字段](xref:core/modeling/backing-field) 。</span><span class="sxs-lookup"><span data-stu-id="e31bf-147">See [Backing Fields](xref:core/modeling/backing-field) for more information on configuring backing fields in the model.</span></span>

<span data-ttu-id="e31bf-148">有时 EF Core 需要在修改属性值时生成副作用。</span><span class="sxs-lookup"><span data-stu-id="e31bf-148">Sometimes it may be desirable for EF Core to generate side-effects when it modifies property values.</span></span> <span data-ttu-id="e31bf-149">例如，将数据绑定到实体时，设置属性可能会向 U.I. 生成通知。</span><span class="sxs-lookup"><span data-stu-id="e31bf-149">For example, when data binding to entities, setting a property may generate notifications to the U.I.</span></span> <span data-ttu-id="e31bf-150">直接设置字段时不会发生这种情况。</span><span class="sxs-lookup"><span data-stu-id="e31bf-150">which do not happen when setting the field directly.</span></span> <span data-ttu-id="e31bf-151">为此，可将更改 <xref:Microsoft.EntityFrameworkCore.PropertyAccessMode> 为：</span><span class="sxs-lookup"><span data-stu-id="e31bf-151">This can be achieved by changing the <xref:Microsoft.EntityFrameworkCore.PropertyAccessMode> for:</span></span>

- <span data-ttu-id="e31bf-152">模型中使用的所有实体类型 <xref:Microsoft.EntityFrameworkCore.ModelBuilder.UsePropertyAccessMode%2A?displayProperty=nameWithType></span><span class="sxs-lookup"><span data-stu-id="e31bf-152">All entity types in the model using <xref:Microsoft.EntityFrameworkCore.ModelBuilder.UsePropertyAccessMode%2A?displayProperty=nameWithType></span></span>
- <span data-ttu-id="e31bf-153">使用特定实体类型的所有属性和导航 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.EntityTypeBuilder%601.UsePropertyAccessMode%2A?displayProperty=nameWithType></span><span class="sxs-lookup"><span data-stu-id="e31bf-153">All properties and navigations of a specific entity type using <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.EntityTypeBuilder%601.UsePropertyAccessMode%2A?displayProperty=nameWithType></span></span>
- <span data-ttu-id="e31bf-154">使用的特定属性 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.UsePropertyAccessMode%2A?displayProperty=nameWithType></span><span class="sxs-lookup"><span data-stu-id="e31bf-154">A specific property using <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.UsePropertyAccessMode%2A?displayProperty=nameWithType></span></span>
- <span data-ttu-id="e31bf-155">使用的特定导航 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.NavigationBuilder.UsePropertyAccessMode%2A?displayProperty=nameWithType></span><span class="sxs-lookup"><span data-stu-id="e31bf-155">A specific navigation using <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.NavigationBuilder.UsePropertyAccessMode%2A?displayProperty=nameWithType></span></span>

<span data-ttu-id="e31bf-156">属性访问模式 `Field` `PreferField` 将导致 EF Core 通过其支持字段访问属性值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-156">Property access modes `Field` and `PreferField` will cause EF Core to access the property value through its backing field.</span></span> <span data-ttu-id="e31bf-157">同样， `Property` 和 `PreferProperty` 将导致 EF Core 通过其 getter 和 setter 访问属性值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-157">Likewise, `Property` and `PreferProperty` will cause EF Core to access the property value through its getter and setter.</span></span>

<span data-ttu-id="e31bf-158">如果 `Field` 使用或， `Property` 并且 EF Core 无法分别通过字段或属性 getter/setter 访问该值，则 EF Core 会引发异常。</span><span class="sxs-lookup"><span data-stu-id="e31bf-158">If `Field` or `Property` are used and EF Core cannot access the value through the field or property getter/setter respectively, then EF Core will throw an exception.</span></span> <span data-ttu-id="e31bf-159">这可确保在您认为 EF Core 始终使用字段/属性访问。</span><span class="sxs-lookup"><span data-stu-id="e31bf-159">This ensures EF Core is always using field/property access when you think it is.</span></span>

<span data-ttu-id="e31bf-160">另一方面， `PreferField` `PreferProperty` 如果不能使用首选访问权限，和模式将回退到使用属性或支持字段。</span><span class="sxs-lookup"><span data-stu-id="e31bf-160">On the other hand, the `PreferField` and `PreferProperty` modes will fall back to using the property or backing field respectively if it is not possible to use the preferred access.</span></span> <span data-ttu-id="e31bf-161">`PreferField` 是 EF Core 3.0 的默认值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-161">`PreferField` is the default from EF Core 3.0 onwards.</span></span> <span data-ttu-id="e31bf-162">这意味着 EF Core 将尽可能使用字段，但如果必须通过其 getter 或 setter 访问属性，则不会失败。</span><span class="sxs-lookup"><span data-stu-id="e31bf-162">This means EF Core will use fields whenever it can, but will not fail if a property must be accessed through its getter or setter instead.</span></span>

<span data-ttu-id="e31bf-163">`FieldDuringConstruction` 并 `PreferFieldDuringConstruction` 将 EF Core 配置为 _仅在创建实体实例时_ 使用支持字段。</span><span class="sxs-lookup"><span data-stu-id="e31bf-163">`FieldDuringConstruction` and `PreferFieldDuringConstruction` configure EF Core to use of backing fields _only when creating entity instances_.</span></span> <span data-ttu-id="e31bf-164">这允许执行无 getter 和 setter 副作用的查询，而 EF Core 的属性更改将导致这些副作用。</span><span class="sxs-lookup"><span data-stu-id="e31bf-164">This allows queries to be executed without getter and setter side effects, while later property changes by EF Core will cause these side effects.</span></span> <span data-ttu-id="e31bf-165">`PreferFieldDuringConstruction` EF Core 3.0 之前的默认值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-165">`PreferFieldDuringConstruction` was the default prior to EF Core 3.0.</span></span>

<span data-ttu-id="e31bf-166">下表汇总了不同的属性访问模式：</span><span class="sxs-lookup"><span data-stu-id="e31bf-166">The different property access modes are summarized in the following table:</span></span>

| <span data-ttu-id="e31bf-167">PropertyAccessMode</span><span class="sxs-lookup"><span data-stu-id="e31bf-167">PropertyAccessMode</span></span>              | <span data-ttu-id="e31bf-168">首选项</span><span class="sxs-lookup"><span data-stu-id="e31bf-168">Preference</span></span> | <span data-ttu-id="e31bf-169">首选项创建实体</span><span class="sxs-lookup"><span data-stu-id="e31bf-169">Preference creating entities</span></span> | <span data-ttu-id="e31bf-170">回退</span><span class="sxs-lookup"><span data-stu-id="e31bf-170">Fallback</span></span> | <span data-ttu-id="e31bf-171">回退创建实体</span><span class="sxs-lookup"><span data-stu-id="e31bf-171">Fallback creating entities</span></span>
|:--------------------------------|------------|------------------------------|----------|---------------------------
| `Field`                         | <span data-ttu-id="e31bf-172">字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-172">Field</span></span>      | <span data-ttu-id="e31bf-173">字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-173">Field</span></span>                        | <span data-ttu-id="e31bf-174">引发</span><span class="sxs-lookup"><span data-stu-id="e31bf-174">Throws</span></span>   | <span data-ttu-id="e31bf-175">引发</span><span class="sxs-lookup"><span data-stu-id="e31bf-175">Throws</span></span>
| `Property`                      | <span data-ttu-id="e31bf-176">属性</span><span class="sxs-lookup"><span data-stu-id="e31bf-176">Property</span></span>   | <span data-ttu-id="e31bf-177">属性</span><span class="sxs-lookup"><span data-stu-id="e31bf-177">Property</span></span>                     | <span data-ttu-id="e31bf-178">引发</span><span class="sxs-lookup"><span data-stu-id="e31bf-178">Throws</span></span>   | <span data-ttu-id="e31bf-179">引发</span><span class="sxs-lookup"><span data-stu-id="e31bf-179">Throws</span></span>
| `PreferField`                   | <span data-ttu-id="e31bf-180">字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-180">Field</span></span>      | <span data-ttu-id="e31bf-181">字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-181">Field</span></span>                        | <span data-ttu-id="e31bf-182">属性</span><span class="sxs-lookup"><span data-stu-id="e31bf-182">Property</span></span> | <span data-ttu-id="e31bf-183">属性</span><span class="sxs-lookup"><span data-stu-id="e31bf-183">Property</span></span>
| `PreferProperty`                | <span data-ttu-id="e31bf-184">属性</span><span class="sxs-lookup"><span data-stu-id="e31bf-184">Property</span></span>   | <span data-ttu-id="e31bf-185">属性</span><span class="sxs-lookup"><span data-stu-id="e31bf-185">Property</span></span>                     | <span data-ttu-id="e31bf-186">字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-186">Field</span></span>    | <span data-ttu-id="e31bf-187">字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-187">Field</span></span>
| `FieldDuringConstruction`       | <span data-ttu-id="e31bf-188">属性</span><span class="sxs-lookup"><span data-stu-id="e31bf-188">Property</span></span>   | <span data-ttu-id="e31bf-189">字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-189">Field</span></span>                        | <span data-ttu-id="e31bf-190">字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-190">Field</span></span>    | <span data-ttu-id="e31bf-191">引发</span><span class="sxs-lookup"><span data-stu-id="e31bf-191">Throws</span></span>
| `PreferFieldDuringConstruction` | <span data-ttu-id="e31bf-192">属性</span><span class="sxs-lookup"><span data-stu-id="e31bf-192">Property</span></span>   | <span data-ttu-id="e31bf-193">字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-193">Field</span></span>                        | <span data-ttu-id="e31bf-194">字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-194">Field</span></span>    | <span data-ttu-id="e31bf-195">属性</span><span class="sxs-lookup"><span data-stu-id="e31bf-195">Property</span></span>

## <a name="temporary-values"></a><span data-ttu-id="e31bf-196">临时值</span><span class="sxs-lookup"><span data-stu-id="e31bf-196">Temporary values</span></span>

<span data-ttu-id="e31bf-197">EF Core 在跟踪新实体时创建临时密钥值，当调用 SaveChanges 时，将具有数据库生成的实际密钥值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-197">EF Core creates temporary key values when tracking new entities that will have real key values generated by the database when SaveChanges is called.</span></span> <span data-ttu-id="e31bf-198">有关如何使用这些临时值的概述，请参阅 [EF Core 中的更改跟踪](xref:core/change-tracking/index) 。</span><span class="sxs-lookup"><span data-stu-id="e31bf-198">See [Change Tracking in EF Core](xref:core/change-tracking/index) for an overview of how these temporary values are used.</span></span>

### <a name="accessing-temporary-values"></a><span data-ttu-id="e31bf-199">访问临时值</span><span class="sxs-lookup"><span data-stu-id="e31bf-199">Accessing temporary values</span></span>

<span data-ttu-id="e31bf-200">从 EF Core 3.0 开始，临时值存储在更改跟踪器中，而不是直接设置到实体实例。</span><span class="sxs-lookup"><span data-stu-id="e31bf-200">Starting with EF Core 3.0, temporary values are stored in the change tracker and not set onto entity instances directly.</span></span> <span data-ttu-id="e31bf-201">但是，当使用各种机制 [访问跟踪的实体](xref:core/change-tracking/entity-entries)时，_会_ 公开这些临时值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-201">However, these temporary values _are_ exposed when using the various mechanisms for [Accessing Tracked Entities](xref:core/change-tracking/entity-entries).</span></span> <span data-ttu-id="e31bf-202">例如，以下代码使用访问临时值 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.CurrentValues%2A?displayProperty=nameWithType> ：</span><span class="sxs-lookup"><span data-stu-id="e31bf-202">For example, the following code accesses a temporary value using <xref:Microsoft.EntityFrameworkCore.ChangeTracking.EntityEntry.CurrentValues%2A?displayProperty=nameWithType>:</span></span>

<!--
        using var context = new BlogsContext();

        var blog = new Blog { Name = ".NET Blog" };

        context.Add(blog);

        Console.WriteLine($"Blog.Id set on entity is {blog.Id}");
        Console.WriteLine($"Blog.Id tracked by EF is {context.Entry(blog).Property(e => e.Id).CurrentValue}");
-->
[!code-csharp[Temporary_values_1](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/Samples.cs?name=Temporary_values_1)]

<span data-ttu-id="e31bf-203">此代码的输出为：</span><span class="sxs-lookup"><span data-stu-id="e31bf-203">The output from this code is:</span></span>

```output
Blog.Id set on entity is 0
Blog.Id tracked by EF is -2147482643
```

<span data-ttu-id="e31bf-204"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsTemporary%2A?displayProperty=nameWithType> 可用于检查临时值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-204"><xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsTemporary%2A?displayProperty=nameWithType> can be used to check for temporary values.</span></span>

### <a name="manipulating-temporary-values"></a><span data-ttu-id="e31bf-205">操作临时值</span><span class="sxs-lookup"><span data-stu-id="e31bf-205">Manipulating temporary values</span></span>

<span data-ttu-id="e31bf-206">对于显式处理临时值，有时很有用。</span><span class="sxs-lookup"><span data-stu-id="e31bf-206">It is sometimes useful to explicitly work with temporary values.</span></span> <span data-ttu-id="e31bf-207">例如，可以在 web 客户端上创建新实体的集合，然后将其序列化回服务器。</span><span class="sxs-lookup"><span data-stu-id="e31bf-207">For example, a collection of new entities might be created on a web client and then serialized back to the server.</span></span> <span data-ttu-id="e31bf-208">外键值是在这些实体之间建立关系的一种方法。</span><span class="sxs-lookup"><span data-stu-id="e31bf-208">Foreign key values are one way to set up relationships between these entities.</span></span> <span data-ttu-id="e31bf-209">下面的代码使用此方法将新实体的关系图与外键关联，同时仍然允许在调用 SaveChanges 时生成实际键值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-209">The following code uses this approach to associate a graph of new entities by foreign key, while still allowing real key values to be generated when SaveChanges is called.</span></span>

<!--
            var blogs = new List<Blog>
            {
                new Blog { Id = -1, Name = ".NET Blog" },
                new Blog { Id = -2, Name = "Visual Studio Blog" }
            };

            var posts = new List<Post>()
            {
                new Post
                {
                    Id = -1,
                    BlogId = -1,
                    Title = "Announcing the Release of EF Core 5.0",
                    Content = "Announcing the release of EF Core 5.0, a full featured cross-platform..."
                },
                new Post
                {
                    Id = -2,
                    BlogId = -2,
                    Title = "Disassembly improvements for optimized managed debugging",
                    Content = "If you are focused on squeezing out the last bits of performance for your .NET service or..."
                }
            };

            using var context = new BlogsContext();

            foreach (var blog in blogs)
            {
                context.Add(blog).Property(e => e.Id).IsTemporary = true;
            }

            foreach (var post in posts)
            {
                context.Add(post).Property(e => e.Id).IsTemporary = true;
            }

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);

            context.SaveChanges();

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Temporary_values_2](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/Samples.cs?name=Temporary_values_2)]

<span data-ttu-id="e31bf-210">请注意：</span><span class="sxs-lookup"><span data-stu-id="e31bf-210">Notice that:</span></span>

- <span data-ttu-id="e31bf-211">负数作为临时键值使用;这不是必需的，但它是防止密钥冲突的常见约定。</span><span class="sxs-lookup"><span data-stu-id="e31bf-211">Negative numbers are used as temporary key values; this is not required, but is a common convention to prevent key clashes.</span></span>
- <span data-ttu-id="e31bf-212">为 `Post.BlogId` FK 属性分配与关联博客的 PK 相同的负值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-212">The `Post.BlogId` FK property is assigned the same negative value as the PK of the associated blog.</span></span>
- <span data-ttu-id="e31bf-213">跟踪每个实体后，将通过设置将 PK 值标记为临时值 <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsTemporary%2A> 。</span><span class="sxs-lookup"><span data-stu-id="e31bf-213">The PK values are marked as temporary by setting <xref:Microsoft.EntityFrameworkCore.ChangeTracking.PropertyEntry.IsTemporary%2A> after each entity is tracked.</span></span> <span data-ttu-id="e31bf-214">这是必需的，因为应用程序提供的任何密钥值都假定为实际键值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-214">This is necessary because any key value supplied by the application is assumed to be a real key value.</span></span>

<span data-ttu-id="e31bf-215">在调用 SaveChanges 之前查看 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图会显示 PK 值标记为临时，并与正确的博客关联，其中包括导航的修正：</span><span class="sxs-lookup"><span data-stu-id="e31bf-215">Looking at the [change tracker debug view](xref:core/change-tracking/debug-views) before calling SaveChanges shows that the PK values are marked as temporary and posts are associated with the correct blogs, including fixup of navigations:</span></span>

```output
Blog {Id: -2} Added
  Id: -2 PK Temporary
  Name: 'Visual Studio Blog'
  Posts: [{Id: -2}]
Blog {Id: -1} Added
  Id: -1 PK Temporary
  Name: '.NET Blog'
  Posts: [{Id: -1}]
Post {Id: -2} Added
  Id: -2 PK Temporary
  BlogId: -2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: {Id: -2}
  Tags: []
Post {Id: -1} Added
  Id: -1 PK Temporary
  BlogId: -1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: -1}
```

<span data-ttu-id="e31bf-216">调用后 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> ，这些临时值被数据库生成的真实值所替换：</span><span class="sxs-lookup"><span data-stu-id="e31bf-216">After calling <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A>, these temporary values have been replaced by real values generated by the database:</span></span>

```output
Blog {Id: 1} Unchanged
  Id: 1 PK
  Name: '.NET Blog'
  Posts: [{Id: 1}]
Blog {Id: 2} Unchanged
  Id: 2 PK
  Name: 'Visual Studio Blog'
  Posts: [{Id: 2}]
Post {Id: 1} Unchanged
  Id: 1 PK
  BlogId: 1 FK
  Content: 'Announcing the release of EF Core 5.0, a full featured cross...'
  Title: 'Announcing the Release of EF Core 5.0'
  Blog: {Id: 1}
  Tags: []
Post {Id: 2} Unchanged
  Id: 2 PK
  BlogId: 2 FK
  Content: 'If you are focused on squeezing out the last bits of perform...'
  Title: 'Disassembly improvements for optimized managed debugging'
  Blog: {Id: 2}
  Tags: []
```

## <a name="working-with-default-values"></a><span data-ttu-id="e31bf-217">使用默认值</span><span class="sxs-lookup"><span data-stu-id="e31bf-217">Working with default values</span></span>

<span data-ttu-id="e31bf-218">当调用时，EF Core 允许属性从数据库获取其默认值 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。</span><span class="sxs-lookup"><span data-stu-id="e31bf-218">EF Core allows a property to get its default value from the database when <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> is called.</span></span> <span data-ttu-id="e31bf-219">与生成的键值一样，如果没有显式设置值，EF Core 将仅从数据库使用默认值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-219">Like with generated key values, EF Core will only use a default from the database if no value has been explicitly set.</span></span> <span data-ttu-id="e31bf-220">例如，请考虑以下实体类型：</span><span class="sxs-lookup"><span data-stu-id="e31bf-220">For example, consider the following entity type:</span></span>

<!--
    public class Token
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public DateTime ValidFrom { get; set; }
    }
-->
[!code-csharp[Token](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Token)]

<span data-ttu-id="e31bf-221">此 `ValidFrom` 属性配置为从数据库中获取默认值：</span><span class="sxs-lookup"><span data-stu-id="e31bf-221">The `ValidFrom` property is configured to get a default value from the database:</span></span>

<!--
        modelBuilder
            .Entity<Token>()
            .Property(e => e.ValidFrom)
            .HasDefaultValueSql("CURRENT_TIMESTAMP");
-->
[!code-csharp[OnModelCreating_Token](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=OnModelCreating_Token)]

<span data-ttu-id="e31bf-222">插入此类型的实体时，EF Core 将使数据库生成值，除非已设置了显式值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-222">When inserting an entity of this type, EF Core will let the database generate the value unless an explicit value has been set instead.</span></span> <span data-ttu-id="e31bf-223">例如：</span><span class="sxs-lookup"><span data-stu-id="e31bf-223">For example:</span></span>

<!--
            using var context = new BlogsContext();

            context.AddRange(
                new Token { Name = "A" },
                new Token { Name = "B", ValidFrom = new DateTime(1111, 11, 11, 11, 11, 11)});

            context.SaveChanges();

            Console.WriteLine(context.ChangeTracker.DebugView.LongView);
-->
[!code-csharp[Working_with_default_values_1](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Working_with_default_values_1)]

<span data-ttu-id="e31bf-224">查看 [更改跟踪](xref:core/change-tracking/debug-views) 器的 "调试" 视图会显示数据库生成的第一个标记 `ValidFrom` ，而第二个标记使用了显式设置的值：</span><span class="sxs-lookup"><span data-stu-id="e31bf-224">Looking at the [change tracker debug view](xref:core/change-tracking/debug-views) shows that the first token had `ValidFrom` generated by the database, while the second token used the value explicitly set:</span></span>

```output
Token {Id: 1} Unchanged
  Id: 1 PK
  Name: 'A'
  ValidFrom: '12/30/2020 6:36:06 PM'
Token {Id: 2} Unchanged
  Id: 2 PK
  Name: 'B'
  ValidFrom: '11/11/1111 11:11:11 AM'
```

> [!NOTE]
> <span data-ttu-id="e31bf-225">使用数据库默认值要求数据库列已配置默认值约束。</span><span class="sxs-lookup"><span data-stu-id="e31bf-225">Using database default values requires that the database column has a default value constraint configured.</span></span> <span data-ttu-id="e31bf-226">使用或时 EF Core 迁移，会自动完成此操作 <xref:Microsoft.EntityFrameworkCore.RelationalPropertyBuilderExtensions.HasDefaultValueSql%2A> <xref:Microsoft.EntityFrameworkCore.RelationalPropertyBuilderExtensions.HasDefaultValue%2A> 。</span><span class="sxs-lookup"><span data-stu-id="e31bf-226">This is done automatically by EF Core migrations when using <xref:Microsoft.EntityFrameworkCore.RelationalPropertyBuilderExtensions.HasDefaultValueSql%2A> or <xref:Microsoft.EntityFrameworkCore.RelationalPropertyBuilderExtensions.HasDefaultValue%2A>.</span></span> <span data-ttu-id="e31bf-227">在不使用 EF Core 迁移时，请确保以其他某种方式在列上创建默认约束。</span><span class="sxs-lookup"><span data-stu-id="e31bf-227">Make sure to create the default constraint on the column in some other way when not using EF Core migrations.</span></span>

### <a name="using-nullable-properties"></a><span data-ttu-id="e31bf-228">使用可以为 null 的属性</span><span class="sxs-lookup"><span data-stu-id="e31bf-228">Using nullable properties</span></span>

<span data-ttu-id="e31bf-229">EF Core 可以通过将属性值与该类型的 CLR 默认值进行比较来确定是否已设置了该属性。</span><span class="sxs-lookup"><span data-stu-id="e31bf-229">EF Core is able to determine whether or not a property has been set by comparing the property value to the CLR default for the that type.</span></span> <span data-ttu-id="e31bf-230">这在大多数情况下非常有效，但意味着无法将 CLR 默认值显式插入数据库中。</span><span class="sxs-lookup"><span data-stu-id="e31bf-230">This works well in most cases, but means that the CLR default cannot be explicitly inserted into the database.</span></span> <span data-ttu-id="e31bf-231">例如，请考虑一个具有整数属性的实体：</span><span class="sxs-lookup"><span data-stu-id="e31bf-231">For example, consider an entity with an integer property:</span></span>

<!--
public class Foo1
{
    public int Id { get; set; }
    public int Count { get; set; }
}
-->
[!code-csharp[Foo1](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Foo1)]

<span data-ttu-id="e31bf-232">其中，该属性配置为具有数据库默认值-1：</span><span class="sxs-lookup"><span data-stu-id="e31bf-232">Where that property is configured to have a database default of -1:</span></span>

<!--
        modelBuilder
            .Entity<Foo1>()
            .Property(e => e.Count)
            .HasDefaultValue(-1);
-->
[!code-csharp[OnModelCreating_Foo1](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=OnModelCreating_Foo1)]

<span data-ttu-id="e31bf-233">目的是在未设置显式值时，将使用默认值-1。</span><span class="sxs-lookup"><span data-stu-id="e31bf-233">The intention is that the default of -1 will be used whenever an explicit value is not set.</span></span> <span data-ttu-id="e31bf-234">但是，如果将值设置为0，则 () 的 CLR 默认值不能区分 EF Core 不设置任何值，这意味着不能为此属性插入0。</span><span class="sxs-lookup"><span data-stu-id="e31bf-234">However, setting the value to 0 (the CLR default for integers) is indistinguishable to EF Core from not setting any value, this means that it is not possible to insert 0 for this property.</span></span> <span data-ttu-id="e31bf-235">例如：</span><span class="sxs-lookup"><span data-stu-id="e31bf-235">For example:</span></span>

<!--
        using var context = new BlogsContext();

        var fooA = new Foo1 { Count = 10 };
        var fooB = new Foo1 { Count = 0 };
        var fooC = new Foo1 { };

        context.AddRange(fooA, fooB, fooC);
        context.SaveChanges();

        Debug.Assert(fooA.Count == 10);
        Debug.Assert(fooB.Count == -1); // Not what we want!
        Debug.Assert(fooC.Count == -1);
-->
[!code-csharp[Working_with_default_values_2](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Working_with_default_values_2)]

<span data-ttu-id="e31bf-236">请注意， `Count` 显式设置为0的实例仍从数据库中获取默认值，而这不是我们预期的值。</span><span class="sxs-lookup"><span data-stu-id="e31bf-236">Notice that the instance where `Count` was explicitly set to 0 is still gets the default value from the database, which is not what we intended.</span></span> <span data-ttu-id="e31bf-237">处理此操作的一种简单方法是将属性设置为 `Count` 可以为 null：</span><span class="sxs-lookup"><span data-stu-id="e31bf-237">An easy way to deal with this is to make the `Count` property nullable:</span></span>

<!--
public class Foo2
{
    public int Id { get; set; }
    public int? Count { get; set; }
}
-->
[!code-csharp[Foo2](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Foo2)]

<span data-ttu-id="e31bf-238">这会使 CLR 默认值为 null，而不是0，这意味着在显式设置时，将插入0：</span><span class="sxs-lookup"><span data-stu-id="e31bf-238">This makes the CLR default null, instead of 0, which means 0 will now be inserted when explicitly set:</span></span>

<!--
        using var context = new BlogsContext();

        var fooA = new Foo2 { Count = 10 };
        var fooB = new Foo2 { Count = 0 };
        var fooC = new Foo2 { };

        context.AddRange(fooA, fooB, fooC);
        context.SaveChanges();

        Debug.Assert(fooA.Count == 10);
        Debug.Assert(fooB.Count == 0);
        Debug.Assert(fooC.Count == -1);
-->
[!code-csharp[Working_with_default_values_3](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Working_with_default_values_3)]

### <a name="using-nullable-backing-fields"></a><span data-ttu-id="e31bf-239">使用可以为 null 的支持字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-239">Using nullable backing fields</span></span>

> [!NOTE]
> <span data-ttu-id="e31bf-240">EF Core 5.0 及更高版本支持此可为 null 的支持字段模式。</span><span class="sxs-lookup"><span data-stu-id="e31bf-240">This nullable backing field pattern is supported by EF Core 5.0 and later.</span></span>

<span data-ttu-id="e31bf-241">使属性可以为 null 的问题，它在域模型中可能不能是可以为 null 的。</span><span class="sxs-lookup"><span data-stu-id="e31bf-241">The problem with making the property nullable that it may not be conceptually nullable in the domain model.</span></span> <span data-ttu-id="e31bf-242">如果强制属性可以为 null，则会对模型进行折衷。</span><span class="sxs-lookup"><span data-stu-id="e31bf-242">Forcing the property to be nullable therefore compromises the model.</span></span>

<span data-ttu-id="e31bf-243">从 EF Core 5.0 开始，属性可以保留为不可为 null，只有支持字段可以为 null。</span><span class="sxs-lookup"><span data-stu-id="e31bf-243">Starting with EF Core 5.0, the property can be left non-nullable, with only the backing field being nullable.</span></span> <span data-ttu-id="e31bf-244">例如：</span><span class="sxs-lookup"><span data-stu-id="e31bf-244">For example:</span></span>

<!--
public class Foo3
{
    public int Id { get; set; }

    private int? _count;
    public int Count
    {
        get => _count ?? -1;
        set => _count = value;
    }
}
-->
[!code-csharp[Foo3](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Foo3)]

<span data-ttu-id="e31bf-245">这允许在将属性显式设置为0的情况下插入 CLR 默认 (0) ，而无需在域模型中将属性公开为可为 null。</span><span class="sxs-lookup"><span data-stu-id="e31bf-245">This allows the CLR default (0) to be inserted if the property is explicitly set to 0, while not needing to expose the property as nullable in the domain model.</span></span> <span data-ttu-id="e31bf-246">例如：</span><span class="sxs-lookup"><span data-stu-id="e31bf-246">For example:</span></span>

<!--
            using var context = new BlogsContext();

            var fooA = new Foo3 { Count = 10 };
            var fooB = new Foo3 { Count = 0 };
            var fooC = new Foo3 { };

            context.AddRange(fooA, fooB, fooC);
            context.SaveChanges();

            Debug.Assert(fooA.Count == 10);
            Debug.Assert(fooB.Count == 0);
            Debug.Assert(fooC.Count == -1);
-->
[!code-csharp[Working_with_default_values_4](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Working_with_default_values_4)]

#### <a name="nullable-backing-fields-for-bool-properties"></a><span data-ttu-id="e31bf-247">布尔属性的可为 null 的支持字段</span><span class="sxs-lookup"><span data-stu-id="e31bf-247">Nullable backing fields for bool properties</span></span>

<span data-ttu-id="e31bf-248">当使用具有存储生成的默认值的 bool 属性时，此模式特别有用。</span><span class="sxs-lookup"><span data-stu-id="e31bf-248">This pattern is especially useful when using bool properties with store-generated defaults.</span></span> <span data-ttu-id="e31bf-249">由于的 CLR 默认值 `bool` 为 "false"，这意味着不能使用 normal 模式显式插入 "false"。</span><span class="sxs-lookup"><span data-stu-id="e31bf-249">Since the CLR default for `bool` is "false", it means that "false" cannot be inserted explicitly using the normal pattern.</span></span> <span data-ttu-id="e31bf-250">例如，假设有一个 `User` 实体类型：</span><span class="sxs-lookup"><span data-stu-id="e31bf-250">For example, consider a `User` entity type:</span></span>

<!--
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }

    private bool? _isAuthorized;
    public bool IsAuthorized
    {
        get => _isAuthorized ?? true;
        set => _isAuthorized = value;
    }
}
-->
[!code-csharp[User](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=User)]

<span data-ttu-id="e31bf-251">此 `IsAuthorized` 属性配置为数据库默认值为 "true"：</span><span class="sxs-lookup"><span data-stu-id="e31bf-251">The `IsAuthorized` property is configured with a database default value of "true":</span></span>

<!--
        modelBuilder
            .Entity<User>()
            .Property(e => e.IsAuthorized)
            .HasDefaultValue(true);
-->
[!code-csharp[OnModelCreating_User](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=OnModelCreating_User)]

<span data-ttu-id="e31bf-252">在 `IsAuthorized` 插入之前，可以将属性显式设置为 "true" 或 "false"，也可以将其设置为 "false"，在这种情况下，将使用数据库默认值：</span><span class="sxs-lookup"><span data-stu-id="e31bf-252">The `IsAuthorized` property can be set to "true" or "false" explicitly before inserting, or can be left unset in which case the database default will be used:</span></span>

<!--
        using var context = new BlogsContext();

        var userA = new User { Name = "Mac" };
        var userB = new User { Name = "Alice", IsAuthorized = true };
        var userC = new User { Name = "Baxter", IsAuthorized = false }; // Always deny Baxter access!

        context.AddRange(userA, userB, userC);

        context.SaveChanges();
-->
[!code-csharp[Working_with_default_values_5](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=Working_with_default_values_5)]

<span data-ttu-id="e31bf-253">使用 SQLite 时，SaveChanges 的输出显示数据库默认值用于 Mac，而为 Alice 和 Baxter 设置了显式值：</span><span class="sxs-lookup"><span data-stu-id="e31bf-253">The output from SaveChanges when using SQLite shows that the database default is used for Mac, while explicit values are set for Alice and Baxter:</span></span>

```sql
-- Executed DbCommand (0ms) [Parameters=[@p0='Mac' (Size = 3)], CommandType='Text', CommandTimeout='30']
INSERT INTO "User" ("Name")
VALUES (@p0);
SELECT "Id", "IsAuthorized"
FROM "User"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();

-- Executed DbCommand (0ms) [Parameters=[@p0='True' (DbType = String), @p1='Alice' (Size = 5)], CommandType='Text', CommandTimeout='30']
INSERT INTO "User" ("IsAuthorized", "Name")
VALUES (@p0, @p1);
SELECT "Id"
FROM "User"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();

-- Executed DbCommand (0ms) [Parameters=[@p0='False' (DbType = String), @p1='Baxter' (Size = 6)], CommandType='Text', CommandTimeout='30']
INSERT INTO "User" ("IsAuthorized", "Name")
VALUES (@p0, @p1);
SELECT "Id"
FROM "User"
WHERE changes() = 1 AND "rowid" = last_insert_rowid();
```

### <a name="schema-defaults-only"></a><span data-ttu-id="e31bf-254">仅限架构默认值</span><span class="sxs-lookup"><span data-stu-id="e31bf-254">Schema defaults only</span></span>

<span data-ttu-id="e31bf-255">有时，通过 EF Core 迁移创建的数据库架构中的默认值非常有用，EF Core 不会使用这些值进行插入。</span><span class="sxs-lookup"><span data-stu-id="e31bf-255">Sometimes it is useful to have defaults in the database schema created by EF Core migrations without EF Core ever using these values for inserts.</span></span> <span data-ttu-id="e31bf-256">为此，可以配置属性，例如 <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.ValueGeneratedNever%2A?displayProperty=nameWithType> ：</span><span class="sxs-lookup"><span data-stu-id="e31bf-256">This can be achieved by configuring the property as <xref:Microsoft.EntityFrameworkCore.Metadata.Builders.PropertyBuilder.ValueGeneratedNever%2A?displayProperty=nameWithType> For example:</span></span>

<!--
        modelBuilder
            .Entity<Bar>()
            .Property(e => e.Count)
            .HasDefaultValue(-1)
            .ValueGeneratedNever();
-->
[!code-csharp[OnModelCreating_Bar](../../../samples/core/ChangeTracking/AdditionalChangeTrackingFeatures/DefaultValueSamples.cs?name=OnModelCreating_Bar)]
