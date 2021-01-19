---
title: EF Core 5.0 中的中断性变更 - EF Core
description: Entity Framework Core 5.0 中引入的中断性变更的完整列表
author: bricelam
ms.date: 11/07/2020
uid: core/what-is-new/ef-core-5.0/breaking-changes
ms.openlocfilehash: 4a463e785edaceaf5dd96164c39e2cc9b5f86de4
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128740"
---
# <a name="breaking-changes-in-ef-core-50"></a><span data-ttu-id="e7f27-103">EF Core 5.0 中的中断性变更</span><span class="sxs-lookup"><span data-stu-id="e7f27-103">Breaking changes in EF Core 5.0</span></span>

<span data-ttu-id="e7f27-104">API 和行为的下列更改有可能导致现有应用程序在更新到 EF Core 5.0.0 时中断。</span><span class="sxs-lookup"><span data-stu-id="e7f27-104">The following API and behavior changes have the potential to break existing applications updating to EF Core 5.0.0.</span></span>

## <a name="summary"></a><span data-ttu-id="e7f27-105">摘要</span><span class="sxs-lookup"><span data-stu-id="e7f27-105">Summary</span></span>

| <span data-ttu-id="e7f27-106">**中断性变更**</span><span class="sxs-lookup"><span data-stu-id="e7f27-106">**Breaking change**</span></span>                                                                                                                   | <span data-ttu-id="e7f27-107">**影响**</span><span class="sxs-lookup"><span data-stu-id="e7f27-107">**Impact**</span></span> |
|:--------------------------------------------------------------------------------------------------------------------------------------|------------|
| [<span data-ttu-id="e7f27-108">EF Core 5.0 不支持 .NET Framework</span><span class="sxs-lookup"><span data-stu-id="e7f27-108">EF Core 5.0 does not support .NET Framework</span></span>](#netstandard21)                                                                         | <span data-ttu-id="e7f27-109">中</span><span class="sxs-lookup"><span data-stu-id="e7f27-109">Medium</span></span>     |
| [<span data-ttu-id="e7f27-110">IProperty.GetColumnName() 现已过时</span><span class="sxs-lookup"><span data-stu-id="e7f27-110">IProperty.GetColumnName() is now obsolete</span></span>](#getcolumnname-obsolete)                                                                  | <span data-ttu-id="e7f27-111">中</span><span class="sxs-lookup"><span data-stu-id="e7f27-111">Medium</span></span>     |
| [<span data-ttu-id="e7f27-112">小数要求精度和小数位数</span><span class="sxs-lookup"><span data-stu-id="e7f27-112">Precision and scale are required for decimals</span></span>](#decimals)                                                                            | <span data-ttu-id="e7f27-113">中</span><span class="sxs-lookup"><span data-stu-id="e7f27-113">Medium</span></span>     |
| [<span data-ttu-id="e7f27-114">具有不同语义的从主体到依赖项的导航中必需</span><span class="sxs-lookup"><span data-stu-id="e7f27-114">Required on the navigation from principal to dependent has different semantics</span></span>](#required-dependent)                                 | <span data-ttu-id="e7f27-115">中</span><span class="sxs-lookup"><span data-stu-id="e7f27-115">Medium</span></span>     |
| [<span data-ttu-id="e7f27-116">定义查询替换为特定于提供程序的方法</span><span class="sxs-lookup"><span data-stu-id="e7f27-116">Defining query is replaced with provider-specific methods</span></span>](#defining-query)                                                          | <span data-ttu-id="e7f27-117">中</span><span class="sxs-lookup"><span data-stu-id="e7f27-117">Medium</span></span>     |
| [<span data-ttu-id="e7f27-118">查询不覆盖非 null 引用导航</span><span class="sxs-lookup"><span data-stu-id="e7f27-118">Non-null reference navigations are not overwritten by queries</span></span>](#nonnullreferences)                                                   | <span data-ttu-id="e7f27-119">中</span><span class="sxs-lookup"><span data-stu-id="e7f27-119">Medium</span></span>     |
| [<span data-ttu-id="e7f27-120">ToView() 因迁移具有不同的处理方式</span><span class="sxs-lookup"><span data-stu-id="e7f27-120">ToView() is treated differently by migrations</span></span>](#toview)                                                                              | <span data-ttu-id="e7f27-121">中型</span><span class="sxs-lookup"><span data-stu-id="e7f27-121">Medium</span></span>     |
| [<span data-ttu-id="e7f27-122">ToTable(null) 将实体类型标记为未映射到表</span><span class="sxs-lookup"><span data-stu-id="e7f27-122">ToTable(null) marks the entity type as not mapped to a table</span></span>](#totable)                                                              | <span data-ttu-id="e7f27-123">中</span><span class="sxs-lookup"><span data-stu-id="e7f27-123">Medium</span></span>     |
| [<span data-ttu-id="e7f27-124">从 SQLite NTS 扩展中删除了 HasGeometricDimension 方法</span><span class="sxs-lookup"><span data-stu-id="e7f27-124">Removed HasGeometricDimension method from SQLite NTS extension</span></span>](#geometric-sqlite)                                                   | <span data-ttu-id="e7f27-125">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-125">Low</span></span>        |
| [<span data-ttu-id="e7f27-126">Cosmos：分区键现已添加到主键</span><span class="sxs-lookup"><span data-stu-id="e7f27-126">Cosmos: Partition key is now added to the primary key</span></span>](#cosmos-partition-key)                                                        | <span data-ttu-id="e7f27-127">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-127">Low</span></span>        |
| [<span data-ttu-id="e7f27-128">Cosmos：`id` 属性重命名为 `__id`</span><span class="sxs-lookup"><span data-stu-id="e7f27-128">Cosmos: `id` property renamed to `__id`</span></span>](#cosmos-id)                                                                                 | <span data-ttu-id="e7f27-129">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-129">Low</span></span>        |
| <span data-ttu-id="e7f27-130">[Cosmos：byte[] 现在存储为 base64 字符串而不是数字数组](#cosmos-byte)</span><span class="sxs-lookup"><span data-stu-id="e7f27-130">[Cosmos: byte[] is now stored as a base64 string instead of a number array](#cosmos-byte)</span></span>                                             | <span data-ttu-id="e7f27-131">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-131">Low</span></span>        |
| [<span data-ttu-id="e7f27-132">Cosmos：GetPropertyName 和 SetPropertyName 已重命名</span><span class="sxs-lookup"><span data-stu-id="e7f27-132">Cosmos: GetPropertyName and SetPropertyName were renamed</span></span>](#cosmos-metadata)                                                          | <span data-ttu-id="e7f27-133">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-133">Low</span></span>        |
| [<span data-ttu-id="e7f27-134">当实体状态从“已分离”更改为“未更改”、“已更新”或“已删除”时，将调用值生成器</span><span class="sxs-lookup"><span data-stu-id="e7f27-134">Value generators are called when the entity state is changed from Detached to Unchanged, Updated, or Deleted</span></span>](#non-added-generation) | <span data-ttu-id="e7f27-135">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-135">Low</span></span>        |
| [<span data-ttu-id="e7f27-136">IMigrationsModelDiffer 当前使用 IRelationalModel</span><span class="sxs-lookup"><span data-stu-id="e7f27-136">IMigrationsModelDiffer now uses IRelationalModel</span></span>](#relational-model)                                                                 | <span data-ttu-id="e7f27-137">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-137">Low</span></span>        |
| [<span data-ttu-id="e7f27-138">鉴别器是只读的</span><span class="sxs-lookup"><span data-stu-id="e7f27-138">Discriminators are read-only</span></span>](#read-only-discriminators)                                                                             | <span data-ttu-id="e7f27-139">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-139">Low</span></span>        |
| [<span data-ttu-id="e7f27-140">特定于提供程序的 EF.Functions 方法针对 InMemory 提供程序引发</span><span class="sxs-lookup"><span data-stu-id="e7f27-140">Provider-specific EF.Functions methods throw for InMemory provider</span></span>](#no-client-methods)                                              | <span data-ttu-id="e7f27-141">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-141">Low</span></span>        |
| [<span data-ttu-id="e7f27-142">IndexBuilder.HasName 现已过时</span><span class="sxs-lookup"><span data-stu-id="e7f27-142">IndexBuilder.HasName is now obsolete</span></span>](#index-obsolete)                                                                               | <span data-ttu-id="e7f27-143">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-143">Low</span></span>        |
| [<span data-ttu-id="e7f27-144">现已包括用于搭建实施了反向工程的模型的复数化程序</span><span class="sxs-lookup"><span data-stu-id="e7f27-144">A pluralizer is now included for scaffolding reverse engineered models</span></span>](#pluralizer)                                                 | <span data-ttu-id="e7f27-145">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-145">Low</span></span>        |
| [<span data-ttu-id="e7f27-146">INavigationBase 替换某些 API 中的 INavigation 以支持跳过导航</span><span class="sxs-lookup"><span data-stu-id="e7f27-146">INavigationBase replaces INavigation in some APIs to support skip navigations</span></span>](#inavigationbase)                                     | <span data-ttu-id="e7f27-147">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-147">Low</span></span>        |
| [<span data-ttu-id="e7f27-148">不再支持使用相关集合同时还使用 `Distinct` 或 `GroupBy` 的查询</span><span class="sxs-lookup"><span data-stu-id="e7f27-148">Some queries with correlated collection that also use `Distinct` or `GroupBy` are no longer supported</span></span>](#collection-distinct-groupby) | <span data-ttu-id="e7f27-149">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-149">Low</span></span>        |
| [<span data-ttu-id="e7f27-150">不支持在投影中使用可查询类型的集合</span><span class="sxs-lookup"><span data-stu-id="e7f27-150">Using a collection of Queryable type in projection is not supported</span></span>](#queryable-projection)                                          | <span data-ttu-id="e7f27-151">低</span><span class="sxs-lookup"><span data-stu-id="e7f27-151">Low</span></span>        |

## <a name="medium-impact-changes"></a><span data-ttu-id="e7f27-152">影响中等的更改</span><span class="sxs-lookup"><span data-stu-id="e7f27-152">Medium-impact changes</span></span>

<a name="netstandard21"></a>

### <a name="ef-core-50-does-not-support-net-framework"></a><span data-ttu-id="e7f27-153">EF Core 5.0 不支持 .NET Framework</span><span class="sxs-lookup"><span data-stu-id="e7f27-153">EF Core 5.0 does not support .NET Framework</span></span>

[<span data-ttu-id="e7f27-154">跟踪问题 #15498</span><span class="sxs-lookup"><span data-stu-id="e7f27-154">Tracking Issue #15498</span></span>](https://github.com/dotnet/efcore/issues/15498)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-155">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-155">Old behavior</span></span>

<span data-ttu-id="e7f27-156">EF Core 3.1 面向 .NET Framework 支持的 .NET Standard 2.0。</span><span class="sxs-lookup"><span data-stu-id="e7f27-156">EF Core 3.1 targets .NET Standard 2.0, which is supported by .NET Framework.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-157">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-157">New behavior</span></span>

<span data-ttu-id="e7f27-158">EF Core 5.0 面向 .NET Framework 不支持的 .NET Standard 2.1。</span><span class="sxs-lookup"><span data-stu-id="e7f27-158">EF Core 5.0 targets .NET Standard 2.1, which is not supported by .NET Framework.</span></span> <span data-ttu-id="e7f27-159">这意味着 EF Core 5.0 无法与 .NET Framework 应用程序一起使用。</span><span class="sxs-lookup"><span data-stu-id="e7f27-159">This means EF Core 5.0 cannot be used with .NET Framework applications.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-160">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-160">Why</span></span>

<span data-ttu-id="e7f27-161">这是整个 .NET 团队的更广泛行动的一部分，旨在统一到一个 .NET 目标框架。</span><span class="sxs-lookup"><span data-stu-id="e7f27-161">This is part of the wider movement across .NET teams aimed at unification to a single .NET target framework.</span></span> <span data-ttu-id="e7f27-162">有关详细信息，请参阅 [.NET Standard 的未来](https://devblogs.microsoft.com/dotnet/the-future-of-net-standard/)。</span><span class="sxs-lookup"><span data-stu-id="e7f27-162">For more information see [the future of .NET Standard](https://devblogs.microsoft.com/dotnet/the-future-of-net-standard/).</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-163">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-163">Mitigations</span></span>

<span data-ttu-id="e7f27-164">.NET Framework 应用程序可继续使用 EF Core 3.1，这是一个[长期支持 (LTS) 版本](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)。</span><span class="sxs-lookup"><span data-stu-id="e7f27-164">.NET Framework applications can continue to use EF Core 3.1, which is a [long-term support (LTS) release](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).</span></span> <span data-ttu-id="e7f27-165">或者，可将应用程序更新为使用 .NET Core 2.1、.NET Core 3.1 或 .NET 5，所有这些都支持 .NET Standard 2.1。</span><span class="sxs-lookup"><span data-stu-id="e7f27-165">Alternately, applications can be updated to use .NET Core 2.1, .NET Core 3.1, or .NET 5, all of which support .NET Standard 2.1.</span></span>

<a name="getcolumnname-obsolete"></a>

### <a name="ipropertygetcolumnname-is-now-obsolete"></a><span data-ttu-id="e7f27-166">IProperty.GetColumnName() 现已过时</span><span class="sxs-lookup"><span data-stu-id="e7f27-166">IProperty.GetColumnName() is now obsolete</span></span>

[<span data-ttu-id="e7f27-167">跟踪问题 #2266</span><span class="sxs-lookup"><span data-stu-id="e7f27-167">Tracking Issue #2266</span></span>](https://github.com/dotnet/efcore/issues/2266)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-168">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-168">Old behavior</span></span>

<span data-ttu-id="e7f27-169">`GetColumnName()` 返回属性将映射到的列的名称。</span><span class="sxs-lookup"><span data-stu-id="e7f27-169">`GetColumnName()` returned the name of the column that a property is mapped to.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-170">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-170">New behavior</span></span>

<span data-ttu-id="e7f27-171">`GetColumnName()` 仍返回属性所映射到的列的名称，但此行为现在是不明确的，因为 EF Core 5 支持 TPT 并支持同时映射到视图或函数，在视图或函数中，这些映射可对同一属性使用不同的列名。</span><span class="sxs-lookup"><span data-stu-id="e7f27-171">`GetColumnName()` still returns the name of a column that a property is mapped to, but this behavior is now ambiguous since EF Core 5 supports TPT and simultaneous mapping to a view or a function where these mappings could use different column names for the same property.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-172">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-172">Why</span></span>

<span data-ttu-id="e7f27-173">我们将此方法标记为已过时，以指导用户更准确地重载 - <xref:Microsoft.EntityFrameworkCore.RelationalPropertyExtensions.GetColumnName(Microsoft.EntityFrameworkCore.Metadata.IProperty,Microsoft.EntityFrameworkCore.Metadata.StoreObjectIdentifier@)>。</span><span class="sxs-lookup"><span data-stu-id="e7f27-173">We marked this method as obsolete to guide users to a more accurate overload - <xref:Microsoft.EntityFrameworkCore.RelationalPropertyExtensions.GetColumnName(Microsoft.EntityFrameworkCore.Metadata.IProperty,Microsoft.EntityFrameworkCore.Metadata.StoreObjectIdentifier@)>.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-174">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-174">Mitigations</span></span>

<span data-ttu-id="e7f27-175">使用以下代码获取特定表的列名：</span><span class="sxs-lookup"><span data-stu-id="e7f27-175">Use the following code to get the column name for a specific table:</span></span>

```csharp
var columnName = property.GetColumnName(StoreObjectIdentifier.Table("Users", null)));
```

<a name="decimals"></a>

### <a name="precision-and-scale-are-required-for-decimals"></a><span data-ttu-id="e7f27-176">小数要求精度和小数位数</span><span class="sxs-lookup"><span data-stu-id="e7f27-176">Precision and scale are required for decimals</span></span>

[<span data-ttu-id="e7f27-177">跟踪问题 #19293</span><span class="sxs-lookup"><span data-stu-id="e7f27-177">Tracking Issue #19293</span></span>](https://github.com/dotnet/efcore/issues/19293)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-178">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-178">Old behavior</span></span>

<span data-ttu-id="e7f27-179">EF Core 通常不会对 <xref:Microsoft.Data.SqlClient.SqlParameter> 对象设置精度和小数位数。</span><span class="sxs-lookup"><span data-stu-id="e7f27-179">EF Core did not normally set precision and scale on <xref:Microsoft.Data.SqlClient.SqlParameter> objects.</span></span> <span data-ttu-id="e7f27-180">这意味着会将完整的精度和小数位数发送到 SQL Server，此时 SQL Server 会根据数据库列的精度和小数位数进行舍入。</span><span class="sxs-lookup"><span data-stu-id="e7f27-180">This means the full precision and scale was sent to SQL Server, at which point SQL Server would round based on the precision and scale of the database column.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-181">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-181">New behavior</span></span>

<span data-ttu-id="e7f27-182">EF Core 现使用为 EF Core 模型中的属性配置的值设置参数的精度和小数位数。</span><span class="sxs-lookup"><span data-stu-id="e7f27-182">EF Core now sets precision and scale on parameters using the values configured for properties in the EF Core model.</span></span> <span data-ttu-id="e7f27-183">这意味着舍入现在发生在 SqlClient 中。</span><span class="sxs-lookup"><span data-stu-id="e7f27-183">This means rounding now happens in SqlClient.</span></span> <span data-ttu-id="e7f27-184">因此，如果配置的精度和小数位数与数据库的精度和小数位数不符，则看到的舍入可能会发生更改。</span><span class="sxs-lookup"><span data-stu-id="e7f27-184">Consequentially, if the configured precision and scale do not match the database precision and scale, then the rounding seen may change.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-185">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-185">Why</span></span>

<span data-ttu-id="e7f27-186">较新的 SQL Server 功能（包括 Always Encrypted）要求完全指定参数方面。</span><span class="sxs-lookup"><span data-stu-id="e7f27-186">Newer SQL Server features, including Always Encrypted, require that parameter facets are fully specified.</span></span> <span data-ttu-id="e7f27-187">此外，SqlClient 更改舍入值而不是截断小数值，从而匹配 SQL Server 行为。</span><span class="sxs-lookup"><span data-stu-id="e7f27-187">In addition, SqlClient made a change to round instead of truncate decimal values, thereby matching the SQL Server behavior.</span></span> <span data-ttu-id="e7f27-188">这使得 EF Core 无需更改正确配置的小数的行为即可设置这些方面。</span><span class="sxs-lookup"><span data-stu-id="e7f27-188">This made it possible for EF Core to set these facets without changing the behavior for correctly configured decimals.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-189">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-189">Mitigations</span></span>

<span data-ttu-id="e7f27-190">使用包含精度和小数位数的类型名称映射小数属性。</span><span class="sxs-lookup"><span data-stu-id="e7f27-190">Map your decimal properties using a type name that includes precision and scale.</span></span> <span data-ttu-id="e7f27-191">例如：</span><span class="sxs-lookup"><span data-stu-id="e7f27-191">For example:</span></span>

```csharp
public class Blog
{
    public int Id { get; set; }

    [Column(TypeName = "decimal(16, 5")]
    public decimal Score { get; set; }
}
```

<span data-ttu-id="e7f27-192">或在模型构建 API 中使用 `HasPrecision`。</span><span class="sxs-lookup"><span data-stu-id="e7f27-192">Or use `HasPrecision` in the model building APIs.</span></span> <span data-ttu-id="e7f27-193">例如：</span><span class="sxs-lookup"><span data-stu-id="e7f27-193">For example:</span></span>

```csharp
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>().Property(e => e.Score).HasPrecision(16, 5);
    }
```

<a name="required-dependent"></a>

### <a name="required-on-the-navigation-from-principal-to-dependent-has-different-semantics"></a><span data-ttu-id="e7f27-194">具有不同语义的从主体到依赖项的导航中必需</span><span class="sxs-lookup"><span data-stu-id="e7f27-194">Required on the navigation from principal to dependent has different semantics</span></span>

[<span data-ttu-id="e7f27-195">跟踪问题 #17286</span><span class="sxs-lookup"><span data-stu-id="e7f27-195">Tracking Issue #17286</span></span>](https://github.com/dotnet/efcore/issues/17286)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-196">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-196">Old behavior</span></span>

<span data-ttu-id="e7f27-197">只有到主体的导航才能配置为“必需”。</span><span class="sxs-lookup"><span data-stu-id="e7f27-197">Only the navigations to principal could be configured as required.</span></span> <span data-ttu-id="e7f27-198">因此，在对到依赖项（包含外键的实体）的导航使用 `RequiredAttribute` 时，将会改为在定义实体类型上创建外键。</span><span class="sxs-lookup"><span data-stu-id="e7f27-198">Therefore using `RequiredAttribute` on the navigation to the dependent (the entity containing the foreign key) would instead create the foreign key on the defining entity type.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-199">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-199">New behavior</span></span>

<span data-ttu-id="e7f27-200">借助对所需依赖项新增的支持后，现在可以将任何引用导航标记为“必需”，这意味着在上述情况下，外键将在关系的另一端进行定义，并且这些属性不会标记为“必需”。</span><span class="sxs-lookup"><span data-stu-id="e7f27-200">With the added support for required dependents, it is now possible to mark any reference navigation as required, meaning that in the case shown above the foreign key will be defined on the other side of the relationship and the properties won't be marked as required.</span></span>

<span data-ttu-id="e7f27-201">目前在指定依赖端之前是否调用 `IsRequired` 尚不明确：</span><span class="sxs-lookup"><span data-stu-id="e7f27-201">Calling `IsRequired` before specifying the dependent end is now ambiguous:</span></span>

```csharp
modelBuilder.Entity<Blog>()
    .HasOne(b => b.BlogImage)
    .WithOne(i => i.Blog)
    .IsRequired()
    .HasForeignKey<BlogImage>(b => b.BlogForeignKey);
```

#### <a name="why"></a><span data-ttu-id="e7f27-202">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-202">Why</span></span>

<span data-ttu-id="e7f27-203">为了支持所需的依赖项，需要新行为（[请参阅 #12100](https://github.com/dotnet/efcore/issues/12100)）。</span><span class="sxs-lookup"><span data-stu-id="e7f27-203">The new behavior is necessary to enable support for required dependents ([see #12100](https://github.com/dotnet/efcore/issues/12100)).</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-204">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-204">Mitigations</span></span>

<span data-ttu-id="e7f27-205">从到依赖项的导航中删除 `RequiredAttribute`，转而将其放置在到主体的导航中或在 `OnModelCreating` 中配置关系：</span><span class="sxs-lookup"><span data-stu-id="e7f27-205">Remove `RequiredAttribute` from the navigation to the dependent and place it instead on the navigation to the principal or configure the relationship in `OnModelCreating`:</span></span>

```csharp
modelBuilder.Entity<Blog>()
    .HasOne(b => b.BlogImage)
    .WithOne(i => i.Blog)
    .HasForeignKey<BlogImage>(b => b.BlogForeignKey)
    .IsRequired();
```

<a name="defining-query"></a>

### <a name="defining-query-is-replaced-with-provider-specific-methods"></a><span data-ttu-id="e7f27-206">定义查询替换为特定于提供程序的方法</span><span class="sxs-lookup"><span data-stu-id="e7f27-206">Defining query is replaced with provider-specific methods</span></span>

[<span data-ttu-id="e7f27-207">跟踪问题 #18903</span><span class="sxs-lookup"><span data-stu-id="e7f27-207">Tracking Issue #18903</span></span>](https://github.com/dotnet/efcore/issues/18903)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-208">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-208">Old behavior</span></span>

<span data-ttu-id="e7f27-209">实体类型映射到定义核心级别的查询。</span><span class="sxs-lookup"><span data-stu-id="e7f27-209">Entity types were mapped to defining queries at the Core level.</span></span> <span data-ttu-id="e7f27-210">每当在查询中使用实体类型时，实体类型的根将被替换为任何提供程序的定义查询。</span><span class="sxs-lookup"><span data-stu-id="e7f27-210">Anytime the entity type was used in the query root of the entity type was replaced by the defining query for any provider.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-211">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-211">New behavior</span></span>

<span data-ttu-id="e7f27-212">已弃用用于定义查询的 API。</span><span class="sxs-lookup"><span data-stu-id="e7f27-212">APIs for defining query are deprecated.</span></span> <span data-ttu-id="e7f27-213">引入了特定于提供程序的新 API。</span><span class="sxs-lookup"><span data-stu-id="e7f27-213">New provider-specific APIs were introduced.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-214">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-214">Why</span></span>

<span data-ttu-id="e7f27-215">在查询中使用查询根时，定义查询作为替换查询实现，但它有一些问题：</span><span class="sxs-lookup"><span data-stu-id="e7f27-215">While defining queries were implemented as replacement query whenever query root is used in the query, it had a few issues:</span></span>

- <span data-ttu-id="e7f27-216">如果定义查询使用 `Select` 方法中的 `new { ... }` 预测实体类型，则将查询标识为实体需要额外的工作，并且与 EF Core 在查询中处理名义类型的方式不一致。</span><span class="sxs-lookup"><span data-stu-id="e7f27-216">If defining query is projecting entity type using `new { ... }` in `Select` method, then identifying that as an entity required additional work and made it inconsistent with how EF Core treats nominal types in the query.</span></span>
- <span data-ttu-id="e7f27-217">对于关系提供程序 `FromSql`，仍然需要以 LINQ 表达式格式传递 SQL 字符串。</span><span class="sxs-lookup"><span data-stu-id="e7f27-217">For relational providers `FromSql` is still needed to pass the SQL string in LINQ expression form.</span></span>

<span data-ttu-id="e7f27-218">定义查询最初是作为客户端视图引入的，用于无键实体的内存中提供程序（类似于关系数据库中的数据库视图）。</span><span class="sxs-lookup"><span data-stu-id="e7f27-218">Initially defining queries were introduced as client-side views to be used with In-Memory provider for keyless entities (similar to database views in relational databases).</span></span> <span data-ttu-id="e7f27-219">这种定义使得对内存中数据库测试应用程序变得容易。</span><span class="sxs-lookup"><span data-stu-id="e7f27-219">Such definition makes it easy to test application against in-memory database.</span></span> <span data-ttu-id="e7f27-220">之后，它们变得广泛适用，这很有用，但带来了不一致和难以理解的行为。</span><span class="sxs-lookup"><span data-stu-id="e7f27-220">Afterwards they became broadly applicable, which was useful but brought inconsistent and hard to understand behavior.</span></span> <span data-ttu-id="e7f27-221">因此，我们决定简化概念。</span><span class="sxs-lookup"><span data-stu-id="e7f27-221">So we decided to simplify the concept.</span></span> <span data-ttu-id="e7f27-222">我们使基于 LINQ 的定义查询独占内存中提供程序，并以不同的方式对其进行处理。</span><span class="sxs-lookup"><span data-stu-id="e7f27-222">We made LINQ based defining query exclusive to In-Memory provider and treat them differently.</span></span> <span data-ttu-id="e7f27-223">有关详细信息，[请参阅此问题](https://github.com/dotnet/efcore/issues/20023)。</span><span class="sxs-lookup"><span data-stu-id="e7f27-223">For more information, [see this issue](https://github.com/dotnet/efcore/issues/20023).</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-224">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-224">Mitigations</span></span>

<span data-ttu-id="e7f27-225">对于关系提供程序，在 `OnModelCreating` 中使用 `ToSqlQuery` 方法，然后传递 SQL 字符串以用于实体类型。</span><span class="sxs-lookup"><span data-stu-id="e7f27-225">For relational providers, use `ToSqlQuery` method in `OnModelCreating` and pass in a SQL string to use for the entity type.</span></span>
<span data-ttu-id="e7f27-226">对于内存中提供程序，在 `OnModelCreating` 中使用 `ToInMemoryQuery` 方法，然后传递要用于实体类型 LINQ 查询。</span><span class="sxs-lookup"><span data-stu-id="e7f27-226">For the In-Memory provider, use `ToInMemoryQuery` method in `OnModelCreating` and pass in a LINQ query to use for the entity type.</span></span>

<a name="nonnullreferences"></a>

### <a name="non-null-reference-navigations-are-not-overwritten-by-queries"></a><span data-ttu-id="e7f27-227">查询不覆盖非 null 引用导航</span><span class="sxs-lookup"><span data-stu-id="e7f27-227">Non-null reference navigations are not overwritten by queries</span></span>

[<span data-ttu-id="e7f27-228">跟踪问题 #2693</span><span class="sxs-lookup"><span data-stu-id="e7f27-228">Tracking Issue #2693</span></span>](https://github.com/dotnet/EntityFramework.Docs/issues/2693)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-229">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-229">Old behavior</span></span>

<span data-ttu-id="e7f27-230">在 EF Core 3.1 中，无论键值是否匹配，数据库中的实体实例有时会覆盖预先初始化为非 null 值的引用导航。</span><span class="sxs-lookup"><span data-stu-id="e7f27-230">In EF Core 3.1, reference navigations eagerly initialized to non-null values would sometimes be overwritten by entity instances from the database, regardless of whether or not key values matched.</span></span> <span data-ttu-id="e7f27-231">而在其他情况下，EF Core 3.1 会执行相反的操作，保留现有的非 null 值。</span><span class="sxs-lookup"><span data-stu-id="e7f27-231">However, in other cases, EF Core 3.1 would do the opposite and leave the existing non-null value.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-232">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-232">New behavior</span></span>

<span data-ttu-id="e7f27-233">从 EF Core 5.0 开始，在任何情况下查询返回的实例都不会覆盖非 null 引用导航。</span><span class="sxs-lookup"><span data-stu-id="e7f27-233">Starting with EF Core 5.0, non-null reference navigations are never overwritten by instances returned from a query.</span></span>

<span data-ttu-id="e7f27-234">请注意，仍支持将集合导航预先初始化为一个空集合。</span><span class="sxs-lookup"><span data-stu-id="e7f27-234">Note that eager initialization of a _collection_ navigation to an empty collection is still supported.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-235">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-235">Why</span></span>

<span data-ttu-id="e7f27-236">将引用导航属性初始化为“空”实体实例将导致状态模糊。</span><span class="sxs-lookup"><span data-stu-id="e7f27-236">Initialization of a reference navigation property to an "empty" entity instance results in an ambiguous state.</span></span> <span data-ttu-id="e7f27-237">例如：</span><span class="sxs-lookup"><span data-stu-id="e7f27-237">For example:</span></span>

```csharp
public class Blog
{
     public int Id { get; set; }
     public Author Author { get; set; ) = new Author();
}
```

<span data-ttu-id="e7f27-238">一个针对博客和作者的查询通常首先会创建 `Blog` 实例，然后根据数据库返回的数据设置适当的 `Author` 实例。</span><span class="sxs-lookup"><span data-stu-id="e7f27-238">Normally a query for Blogs and Authors will first create `Blog` instances and then set the appropriate `Author` instances based on the data returned from the database.</span></span> <span data-ttu-id="e7f27-239">但是，在这种情况下，每个 `Blog.Author` 属性都已初始化为空 `Author`。</span><span class="sxs-lookup"><span data-stu-id="e7f27-239">However, in this case every `Blog.Author` property is already initialized to an empty `Author`.</span></span> <span data-ttu-id="e7f27-240">不过 EF Core 无法知道此实例是否为“空”。</span><span class="sxs-lookup"><span data-stu-id="e7f27-240">Except EF Core has no way to know that this instance is "empty".</span></span> <span data-ttu-id="e7f27-241">因此覆盖此实例可能会悄无声息地丢弃一个有效的 `Author`。</span><span class="sxs-lookup"><span data-stu-id="e7f27-241">So overwriting this instance could potentially silently throw away a valid `Author`.</span></span> <span data-ttu-id="e7f27-242">因而，EF Core 5.0 现在始终不会覆盖已初始化的导航。</span><span class="sxs-lookup"><span data-stu-id="e7f27-242">Therefore, EF Core 5.0 now consistently does not overwrite a navigation that is already initialized.</span></span>

<span data-ttu-id="e7f27-243">尽管经过调查发现，此新行为有时与 EF6 的行为不一致，但在大多数情况下它都与 EF6 一致。</span><span class="sxs-lookup"><span data-stu-id="e7f27-243">This new behavior also aligns with the behavior of EF6 in most cases, although upon investigation we also found some cases of inconsistency in EF6.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-244">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-244">Mitigations</span></span>

<span data-ttu-id="e7f27-245">如果遇到此中断，解决方法是停止预先初始化引用导航属性。</span><span class="sxs-lookup"><span data-stu-id="e7f27-245">If this break is encountered, then the fix is to stop eagerly initializing reference navigation properties.</span></span>

<a name="toview"></a>

### <a name="toview-is-treated-differently-by-migrations"></a><span data-ttu-id="e7f27-246">ToView() 因迁移具有不同的处理方式</span><span class="sxs-lookup"><span data-stu-id="e7f27-246">ToView() is treated differently by migrations</span></span>

[<span data-ttu-id="e7f27-247">跟踪问题 #2725</span><span class="sxs-lookup"><span data-stu-id="e7f27-247">Tracking Issue #2725</span></span>](https://github.com/dotnet/efcore/issues/2725)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-248">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-248">Old behavior</span></span>

<span data-ttu-id="e7f27-249">如果调用 `ToView(string)`，迁移会将实体类型映射到视图，但会忽略该类型。</span><span class="sxs-lookup"><span data-stu-id="e7f27-249">Calling `ToView(string)` made the migrations ignore the entity type in addition to mapping it to a view.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-250">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-250">New behavior</span></span>

<span data-ttu-id="e7f27-251">现在，`ToView(string)` 除了将实体类型映射到视图，还会将其标记为未映射到表。</span><span class="sxs-lookup"><span data-stu-id="e7f27-251">Now `ToView(string)` marks the entity type as not mapped to a table in addition to mapping it to a view.</span></span> <span data-ttu-id="e7f27-252">这会导致升级到 EF Core 5 之后的第一次迁移尝试删除此实体类型的默认表，因为它不会被忽略。</span><span class="sxs-lookup"><span data-stu-id="e7f27-252">This results in the first migration after upgrading to EF Core 5 to try to drop the default table for this entity type as it's not longer ignored.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-253">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-253">Why</span></span>

<span data-ttu-id="e7f27-254">EF Core 现在允许同时映射到表和视图的实体类型，因此 `ToView` 不再是应由迁移忽略的有效指示器。</span><span class="sxs-lookup"><span data-stu-id="e7f27-254">EF Core now allows an entity type to be mapped to both a table and a view simultaneously, so `ToView` is no longer a valid indicator that it should be ignored by migrations.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-255">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-255">Mitigations</span></span>

<span data-ttu-id="e7f27-256">使用以下代码将映射的表标记为从迁移中排除：</span><span class="sxs-lookup"><span data-stu-id="e7f27-256">Use the following code to mark the mapped table as excluded from migrations:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>().ToTable("UserView", t => t.ExcludeFromMigrations());
}
```

<a name="totable"></a>

### <a name="totablenull-marks-the-entity-type-as-not-mapped-to-a-table"></a><span data-ttu-id="e7f27-257">ToTable(null) 将实体类型标记为未映射到表</span><span class="sxs-lookup"><span data-stu-id="e7f27-257">ToTable(null) marks the entity type as not mapped to a table</span></span>

[<span data-ttu-id="e7f27-258">跟踪问题 #21172</span><span class="sxs-lookup"><span data-stu-id="e7f27-258">Tracking Issue #21172</span></span>](https://github.com/dotnet/efcore/issues/21172)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-259">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-259">Old behavior</span></span>

<span data-ttu-id="e7f27-260">`ToTable(null)` 会将表名称重置为默认值。</span><span class="sxs-lookup"><span data-stu-id="e7f27-260">`ToTable(null)` would reset the table name to the default.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-261">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-261">New behavior</span></span>

<span data-ttu-id="e7f27-262">`ToTable(null)` 现在会将实体类型标记为未映射到任何表。</span><span class="sxs-lookup"><span data-stu-id="e7f27-262">`ToTable(null)` now marks the entity type as not mapped to any table.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-263">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-263">Why</span></span>

<span data-ttu-id="e7f27-264">EF Core 现在允许同时映射到表和视图的实体类型，因此使用 `ToTable(null)` 指示其未映射到任何表。</span><span class="sxs-lookup"><span data-stu-id="e7f27-264">EF Core now allows an entity type to be mapped to both a table and a view simultaneously, so `ToTable(null)` is used to indicate that it isn't mapped to any table.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-265">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-265">Mitigations</span></span>

<span data-ttu-id="e7f27-266">如果表名称未映射到视图或 DbFunction，请使用以下代码将其重置为默认值：</span><span class="sxs-lookup"><span data-stu-id="e7f27-266">Use the following code to reset the table name to the default if it's not mapped to a view or a DbFunction:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>().Metadata.RemoveAnnotation(RelationalAnnotationNames.TableName);
}
```

## <a name="low-impact-changes"></a><span data-ttu-id="e7f27-267">影响较小的更改</span><span class="sxs-lookup"><span data-stu-id="e7f27-267">Low-impact changes</span></span>

<a name="geometric-sqlite"></a>

### <a name="removed-hasgeometricdimension-method-from-sqlite-nts-extension"></a><span data-ttu-id="e7f27-268">从 SQLite NTS 扩展中删除了 HasGeometricDimension 方法</span><span class="sxs-lookup"><span data-stu-id="e7f27-268">Removed HasGeometricDimension method from SQLite NTS extension</span></span>

[<span data-ttu-id="e7f27-269">跟踪问题 #14257</span><span class="sxs-lookup"><span data-stu-id="e7f27-269">Tracking Issue #14257</span></span>](https://github.com/dotnet/efcore/issues/14257)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-270">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-270">Old behavior</span></span>

<span data-ttu-id="e7f27-271">HasGeometricDimension 过去用于在几何列上启用其他维度（Z 和 M）。</span><span class="sxs-lookup"><span data-stu-id="e7f27-271">HasGeometricDimension was used to enable additional dimensions (Z and M) on geometry columns.</span></span> <span data-ttu-id="e7f27-272">但是，之前它只影响数据库创建。</span><span class="sxs-lookup"><span data-stu-id="e7f27-272">However, it only ever affected database creation.</span></span> <span data-ttu-id="e7f27-273">不需要指定它来查询具有其他维度的值。</span><span class="sxs-lookup"><span data-stu-id="e7f27-273">It was unnecessary to specify it to query values with additional dimensions.</span></span> <span data-ttu-id="e7f27-274">之前，在插入或更新具有其他维度的值时，它也无法正常工作（请参见 [see #14257](https://github.com/dotnet/efcore/issues/14257)）。</span><span class="sxs-lookup"><span data-stu-id="e7f27-274">It also didn't work correctly when inserting or updating values with additional dimensions ([see #14257](https://github.com/dotnet/efcore/issues/14257)).</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-275">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-275">New behavior</span></span>

<span data-ttu-id="e7f27-276">要能够插入和更新具有其他维度（Z 和 M）的几何值，需要将维度指定为列类型名称的一部分。</span><span class="sxs-lookup"><span data-stu-id="e7f27-276">To enable inserting and updating geometry values with additional dimensions (Z and M), the dimension needs to be specified as part of the column type name.</span></span> <span data-ttu-id="e7f27-277">该 API 与 SpatiaLite 的 AddGeometryColumn 函数的基本行为匹配度更高。</span><span class="sxs-lookup"><span data-stu-id="e7f27-277">This API matches more closely to the underlying behavior of SpatiaLite's AddGeometryColumn function.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-278">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-278">Why</span></span>

<span data-ttu-id="e7f27-279">在列类型中指定维度后，不需要使用 HasGeometricDimension，该方法也很多余，因此我们已将它彻底删除。</span><span class="sxs-lookup"><span data-stu-id="e7f27-279">Using HasGeometricDimension after specifying the dimension in the column type is unnecessary and redundant, so we removed HasGeometricDimension entirely.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-280">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-280">Mitigations</span></span>

<span data-ttu-id="e7f27-281">使用 `HasColumnType` 指定维度：</span><span class="sxs-lookup"><span data-stu-id="e7f27-281">Use `HasColumnType` to specify the dimension:</span></span>

```csharp
modelBuilder.Entity<GeoEntity>(
    x =>
    {
        // Allow any GEOMETRY value with optional Z and M values
        x.Property(e => e.Geometry).HasColumnType("GEOMETRYZM");

        // Allow only POINT values with an optional Z value
        x.Property(e => e.Point).HasColumnType("POINTZ");
    });
```

<a name="cosmos-partition-key"></a>

### <a name="cosmos-partition-key-is-now-added-to-the-primary-key"></a><span data-ttu-id="e7f27-282">Cosmos：分区键现已添加到主键</span><span class="sxs-lookup"><span data-stu-id="e7f27-282">Cosmos: Partition key is now added to the primary key</span></span>

[<span data-ttu-id="e7f27-283">跟踪问题 #15289</span><span class="sxs-lookup"><span data-stu-id="e7f27-283">Tracking Issue #15289</span></span>](https://github.com/dotnet/efcore/issues/15289)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-284">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-284">Old behavior</span></span>

<span data-ttu-id="e7f27-285">分区键仅添加到包含 `id` 的备用键。</span><span class="sxs-lookup"><span data-stu-id="e7f27-285">The partition key property was only added to the alternate key that includes `id`.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-286">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-286">New behavior</span></span>

<span data-ttu-id="e7f27-287">分区键现在还按约定添加到主键。</span><span class="sxs-lookup"><span data-stu-id="e7f27-287">The partition key property is now also added to the primary key by convention.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-288">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-288">Why</span></span>

<span data-ttu-id="e7f27-289">此更改使模型更好地与 Azure Cosmos DB 语义对齐，并改进 `Find` 和某些查询的性能。</span><span class="sxs-lookup"><span data-stu-id="e7f27-289">This change makes the model better aligned with Azure Cosmos DB semantics and improves the performance of `Find` and some queries.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-290">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-290">Mitigations</span></span>

<span data-ttu-id="e7f27-291">若要防止将分区键属性添加到主键，请在 `OnModelCreating` 中配置它。</span><span class="sxs-lookup"><span data-stu-id="e7f27-291">To prevent the partition key property to be added to the primary key, configure it in `OnModelCreating`.</span></span>

```csharp
modelBuilder.Entity<Blog>()
    .HasKey(b => b.Id);
```

<a name="cosmos-id"></a>

### <a name="cosmos-id-property-renamed-to-__id"></a><span data-ttu-id="e7f27-292">Cosmos：`id` 属性重命名为 `__id`</span><span class="sxs-lookup"><span data-stu-id="e7f27-292">Cosmos: `id` property renamed to `__id`</span></span>

[<span data-ttu-id="e7f27-293">跟踪问题 #17751</span><span class="sxs-lookup"><span data-stu-id="e7f27-293">Tracking Issue #17751</span></span>](https://github.com/dotnet/efcore/issues/17751)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-294">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-294">Old behavior</span></span>

<span data-ttu-id="e7f27-295">映射到 `id` JSON 属性的阴影属性也称为 `id`。</span><span class="sxs-lookup"><span data-stu-id="e7f27-295">The shadow property mapped to the `id` JSON property was also named `id`.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-296">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-296">New behavior</span></span>

<span data-ttu-id="e7f27-297">按照约定创建的阴影属性现在命名为 `__id`。</span><span class="sxs-lookup"><span data-stu-id="e7f27-297">The shadow property created by convention is now named `__id`.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-298">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-298">Why</span></span>

<span data-ttu-id="e7f27-299">此更改使 `id` 属性与实体类型上的现有属性冲突的可能性更小。</span><span class="sxs-lookup"><span data-stu-id="e7f27-299">This change makes it less likely that the `id` property clashes with an existing property on the entity type.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-300">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-300">Mitigations</span></span>

<span data-ttu-id="e7f27-301">若要返回到 3.x 行为，请在 `OnModelCreating` 中配置 `id` 属性。</span><span class="sxs-lookup"><span data-stu-id="e7f27-301">To go back to the 3.x behavior, configure the `id` property in `OnModelCreating`.</span></span>

```csharp
modelBuilder.Entity<Blog>()
    .Property<string>("id")
    .ToJsonProperty("id");
```

<a name="cosmos-byte"></a>

### <a name="cosmos-byte-is-now-stored-as-a-base64-string-instead-of-a-number-array"></a><span data-ttu-id="e7f27-302">Cosmos：byte[] 现在存储为 base64 字符串而不是数字数组</span><span class="sxs-lookup"><span data-stu-id="e7f27-302">Cosmos: byte[] is now stored as a base64 string instead of a number array</span></span>

[<span data-ttu-id="e7f27-303">跟踪问题 #17306</span><span class="sxs-lookup"><span data-stu-id="e7f27-303">Tracking Issue #17306</span></span>](https://github.com/dotnet/efcore/issues/17306)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-304">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-304">Old behavior</span></span>

<span data-ttu-id="e7f27-305">类型 byte[] 的属性存储为数字数组。</span><span class="sxs-lookup"><span data-stu-id="e7f27-305">Properties of type byte[] were stored as a number array.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-306">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-306">New behavior</span></span>

<span data-ttu-id="e7f27-307">类型 byte[] 的属性现存储为 base64 字符串。</span><span class="sxs-lookup"><span data-stu-id="e7f27-307">Properties of type byte[] are now stored as a base64 string.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-308">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-308">Why</span></span>

<span data-ttu-id="e7f27-309">byte[] 的这种表示形式与预期更好地对齐，并且是主要 JSON 序列化库的默认行为。</span><span class="sxs-lookup"><span data-stu-id="e7f27-309">This representation of byte[] aligns better with expectations and is the default behavior of the major JSON serialization libraries.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-310">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-310">Mitigations</span></span>

<span data-ttu-id="e7f27-311">仍将正确查询作为数字数组存储的现有数据，但目前没有支持更改插入行为的方法。</span><span class="sxs-lookup"><span data-stu-id="e7f27-311">Existing data stored as number arrays will still be queried correctly, but currently there isn't a supported way to change back the insert behavior.</span></span> <span data-ttu-id="e7f27-312">如果此限制对于你的方案是一种阻碍，请对[此问题](https://github.com/dotnet/efcore/issues/17306)发表评论</span><span class="sxs-lookup"><span data-stu-id="e7f27-312">If this limitation is blocking your scenario, comment on [this issue](https://github.com/dotnet/efcore/issues/17306)</span></span>

<a name="cosmos-metadata"></a>

### <a name="cosmos-getpropertyname-and-setpropertyname-were-renamed"></a><span data-ttu-id="e7f27-313">Cosmos：GetPropertyName 和 SetPropertyName 已重命名</span><span class="sxs-lookup"><span data-stu-id="e7f27-313">Cosmos: GetPropertyName and SetPropertyName were renamed</span></span>

[<span data-ttu-id="e7f27-314">跟踪问题 #17874</span><span class="sxs-lookup"><span data-stu-id="e7f27-314">Tracking Issue #17874</span></span>](https://github.com/dotnet/efcore/issues/17874)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-315">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-315">Old behavior</span></span>

<span data-ttu-id="e7f27-316">扩展方法此前称为 `GetPropertyName` 和 `SetPropertyName`</span><span class="sxs-lookup"><span data-stu-id="e7f27-316">Previously the extension methods were called `GetPropertyName` and `SetPropertyName`</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-317">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-317">New behavior</span></span>

<span data-ttu-id="e7f27-318">旧 API 已删除，添加了以下新方法：`GetJsonPropertyName`、`SetJsonPropertyName`</span><span class="sxs-lookup"><span data-stu-id="e7f27-318">The old API was removed and new methods added: `GetJsonPropertyName`, `SetJsonPropertyName`</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-319">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-319">Why</span></span>

<span data-ttu-id="e7f27-320">此更改消除了有关这些方法配置方式的歧义。</span><span class="sxs-lookup"><span data-stu-id="e7f27-320">This change removes the ambiguity around what these methods are configuring.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-321">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-321">Mitigations</span></span>

<span data-ttu-id="e7f27-322">使用新 API。</span><span class="sxs-lookup"><span data-stu-id="e7f27-322">Use the new API.</span></span>

<a name="non-added-generation"></a>

### <a name="value-generators-are-called-when-the-entity-state-is-changed-from-detached-to-unchanged-updated-or-deleted"></a><span data-ttu-id="e7f27-323">当实体状态从“已分离”更改为“未更改”、“已更新”或“已删除”时，将调用值生成器</span><span class="sxs-lookup"><span data-stu-id="e7f27-323">Value generators are called when the entity state is changed from Detached to Unchanged, Updated, or Deleted</span></span>

[<span data-ttu-id="e7f27-324">跟踪问题 #15289</span><span class="sxs-lookup"><span data-stu-id="e7f27-324">Tracking Issue #15289</span></span>](https://github.com/dotnet/efcore/issues/15289)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-325">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-325">Old behavior</span></span>

<span data-ttu-id="e7f27-326">只有当实体状态更改为“已添加”时，才调用值生成器。</span><span class="sxs-lookup"><span data-stu-id="e7f27-326">Value generators were only called when the entity state changed to Added.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-327">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-327">New behavior</span></span>

<span data-ttu-id="e7f27-328">目前，当实体状态从“已分离”更改为“未更改”、“已更新”或“已删除”，且属性包含默认值时，将调用值生成器。</span><span class="sxs-lookup"><span data-stu-id="e7f27-328">Value generators are now called when the entity state is changed from Detached to Unchanged, Updated, or Deleted and the property contains the default values.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-329">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-329">Why</span></span>

<span data-ttu-id="e7f27-330">此更改是必要的，以便改进未保存到数据存储并始终在客户端上生成其值的属性方面的体验。</span><span class="sxs-lookup"><span data-stu-id="e7f27-330">This change was necessary to improve the experience with properties that are not persisted to the data store and have their value generated always on the client.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-331">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-331">Mitigations</span></span>

<span data-ttu-id="e7f27-332">若要防止调用值生成器，请在状态更改前向属性分配一个非默认值。</span><span class="sxs-lookup"><span data-stu-id="e7f27-332">To prevent the value generator from being called, assign a non-default value to the property before the state is changed.</span></span>

<a name="relational-model"></a>

### <a name="imigrationsmodeldiffer-now-uses-irelationalmodel"></a><span data-ttu-id="e7f27-333">IMigrationsModelDiffer 当前使用 IRelationalModel</span><span class="sxs-lookup"><span data-stu-id="e7f27-333">IMigrationsModelDiffer now uses IRelationalModel</span></span>

[<span data-ttu-id="e7f27-334">跟踪问题 #20305</span><span class="sxs-lookup"><span data-stu-id="e7f27-334">Tracking Issue #20305</span></span>](https://github.com/dotnet/efcore/issues/20305)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-335">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-335">Old behavior</span></span>

<span data-ttu-id="e7f27-336">`IMigrationsModelDiffer` API 使用了 `IModel` 进行定义。</span><span class="sxs-lookup"><span data-stu-id="e7f27-336">`IMigrationsModelDiffer` API was defined using `IModel`.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-337">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-337">New behavior</span></span>

<span data-ttu-id="e7f27-338">`IMigrationsModelDiffer` API 当前使用 `IRelationalModel`。</span><span class="sxs-lookup"><span data-stu-id="e7f27-338">`IMigrationsModelDiffer` API now uses `IRelationalModel`.</span></span> <span data-ttu-id="e7f27-339">不过，模型快照仍只包含 `IModel`，因为此代码是应用程序的一部分，并且实体框架无法在不做出较大中断性变更的情况下对其进行更改。</span><span class="sxs-lookup"><span data-stu-id="e7f27-339">However the model snapshot still contains only `IModel` as this code is part of the application and Entity Framework can't change it without making a bigger breaking change.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-340">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-340">Why</span></span>

<span data-ttu-id="e7f27-341">`IRelationalModel` 是新添加的数据库架构的表示形式。</span><span class="sxs-lookup"><span data-stu-id="e7f27-341">`IRelationalModel` is a newly added representation of the database schema.</span></span> <span data-ttu-id="e7f27-342">使用它可以更快速、更精确地查找差异。</span><span class="sxs-lookup"><span data-stu-id="e7f27-342">Using it to find differences is faster and more accurate.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-343">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-343">Mitigations</span></span>

<span data-ttu-id="e7f27-344">使用以下代码将 `context` 中的模型与 `snapshot` 中的模型进行比较：</span><span class="sxs-lookup"><span data-stu-id="e7f27-344">Use the following code to compare the model from `snapshot` with the model from `context`:</span></span>

```csharp
var dependencies = context.GetService<ProviderConventionSetBuilderDependencies>();
var relationalDependencies = context.GetService<RelationalConventionSetBuilderDependencies>();

var typeMappingConvention = new TypeMappingConvention(dependencies);
typeMappingConvention.ProcessModelFinalizing(((IConventionModel)modelSnapshot.Model).Builder, null);

var relationalModelConvention = new RelationalModelConvention(dependencies, relationalDependencies);
var sourceModel = relationalModelConvention.ProcessModelFinalized(snapshot.Model);

var modelDiffer = context.GetService<IMigrationsModelDiffer>();
var hasDifferences = modelDiffer.HasDifferences(
    ((IMutableModel)sourceModel).FinalizeModel().GetRelationalModel(),
    context.Model.GetRelationalModel());
```

<span data-ttu-id="e7f27-345">我们计划在 6.0 中改进这种体验（[请参阅 #22031](https://github.com/dotnet/efcore/issues/22031)）</span><span class="sxs-lookup"><span data-stu-id="e7f27-345">We are planning to improve this experience in 6.0 ([see #22031](https://github.com/dotnet/efcore/issues/22031))</span></span>

<a name="read-only-discriminators"></a>

### <a name="discriminators-are-read-only"></a><span data-ttu-id="e7f27-346">鉴别器是只读的</span><span class="sxs-lookup"><span data-stu-id="e7f27-346">Discriminators are read-only</span></span>

[<span data-ttu-id="e7f27-347">跟踪问题 #21154</span><span class="sxs-lookup"><span data-stu-id="e7f27-347">Tracking Issue #21154</span></span>](https://github.com/dotnet/efcore/issues/21154)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-348">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-348">Old behavior</span></span>

<span data-ttu-id="e7f27-349">在调用 `SaveChanges` 之前，可以更改鉴别器值</span><span class="sxs-lookup"><span data-stu-id="e7f27-349">It was possible to change the discriminator value before calling `SaveChanges`</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-350">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-350">New behavior</span></span>

<span data-ttu-id="e7f27-351">在上述情况下，将引发异常。</span><span class="sxs-lookup"><span data-stu-id="e7f27-351">An exception will be throws in the above case.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-352">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-352">Why</span></span>

<span data-ttu-id="e7f27-353">EF 不希望实体类型仍在受到跟踪时就发生更改，因此更改鉴别器值会使上下文处于不一致的状态，这可能导致意外行为。</span><span class="sxs-lookup"><span data-stu-id="e7f27-353">EF doesn't expect the entity type to change while it is still being tracked, so changing the discriminator value leaves the context in an inconsistent state, which might result in unexpected behavior.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-354">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-354">Mitigations</span></span>

<span data-ttu-id="e7f27-355">如果更改鉴别器值是必需的，并且在调用 `SaveChanges` 后将立即处理上下文，则可将鉴别器设置为可变：</span><span class="sxs-lookup"><span data-stu-id="e7f27-355">If changing the discriminator value is necessary and the context will be disposed immediately after calling `SaveChanges`, the discriminator can be made mutable:</span></span>

```csharp
modelBuilder.Entity<BaseEntity>()
    .Property<string>("Discriminator")
    .Metadata.SetAfterSaveBehavior(PropertySaveBehavior.Save);
```

<a name="no-client-methods"></a>

### <a name="provider-specific-effunctions-methods-throw-for-inmemory-provider"></a><span data-ttu-id="e7f27-356">特定于提供程序的 EF.Functions 方法针对 InMemory 提供程序引发</span><span class="sxs-lookup"><span data-stu-id="e7f27-356">Provider-specific EF.Functions methods throw for InMemory provider</span></span>

[<span data-ttu-id="e7f27-357">跟踪问题 #20294</span><span class="sxs-lookup"><span data-stu-id="e7f27-357">Tracking Issue #20294</span></span>](https://github.com/dotnet/efcore/issues/20294)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-358">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-358">Old behavior</span></span>

<span data-ttu-id="e7f27-359">特定于提供程序的 EF.Functions 方法包含对客户端执行的实现，从而使这些方法可以在 InMemory 提供程序上执行。</span><span class="sxs-lookup"><span data-stu-id="e7f27-359">Provider-specific EF.Functions methods contained implementation for client execution, which allowed them to be executed on the InMemory provider.</span></span> <span data-ttu-id="e7f27-360">例如，`EF.Functions.DateDiffDay` 是特定于 SQL Server 的方法，它在 InMemory 提供程序上运行。</span><span class="sxs-lookup"><span data-stu-id="e7f27-360">For example, `EF.Functions.DateDiffDay` is a Sql Server specific method, which worked on InMemory provider.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-361">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-361">New behavior</span></span>

<span data-ttu-id="e7f27-362">特定于提供程序的方法已更新为在其方法主体中引发异常，以阻止在客户端上对它们进行评估。</span><span class="sxs-lookup"><span data-stu-id="e7f27-362">Provider-specific methods have been updated to throw an exception in their method body to block evaluating them on client side.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-363">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-363">Why</span></span>

<span data-ttu-id="e7f27-364">特定于提供程序的方法映射到数据库函数。</span><span class="sxs-lookup"><span data-stu-id="e7f27-364">Provider-specific methods map to a database function.</span></span> <span data-ttu-id="e7f27-365">在 LINQ 中，映射的数据库函数执行的计算无法每次都在客户端上进行复制。</span><span class="sxs-lookup"><span data-stu-id="e7f27-365">The computation done by the mapped database function can't always be replicated on the client side in LINQ.</span></span> <span data-ttu-id="e7f27-366">当在客户端执行相同的方法时，这可能会导致来自服务器的结果有所不同。</span><span class="sxs-lookup"><span data-stu-id="e7f27-366">It may cause the result from the server to differ when executing the same method on client.</span></span> <span data-ttu-id="e7f27-367">由于这些方法会用于 LINQ，来转换到特定数据库函数，因此无需在客户端上评估这些方法。</span><span class="sxs-lookup"><span data-stu-id="e7f27-367">Since these methods are used in LINQ to translate to specific database functions, they don't need to be evaluated on client side.</span></span> <span data-ttu-id="e7f27-368">由于 InMemory 提供程序是另一种数据库，因此这些方法不能用于此提供程序。</span><span class="sxs-lookup"><span data-stu-id="e7f27-368">As InMemory provider is a different *database*, these methods aren't available for this provider.</span></span> <span data-ttu-id="e7f27-369">如果尝试为 InMemory 提供程序或任何其他无法转换这些方法的提供程序执行它们时，将引发异常。</span><span class="sxs-lookup"><span data-stu-id="e7f27-369">Trying to execute them for InMemory provider, or any other provider that doesn't translate these methods, throws an exception.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-370">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-370">Mitigations</span></span>

<span data-ttu-id="e7f27-371">由于无法准确模拟数据库函数的行为，因此应根据生产中的同一种数据库测试包含这些函数的查询。</span><span class="sxs-lookup"><span data-stu-id="e7f27-371">Since there's no way to mimic behavior of database functions accurately, you should test the queries containing them against same kind of database as in production.</span></span>

<a name="index-obsolete"></a>

### <a name="indexbuilderhasname-is-now-obsolete"></a><span data-ttu-id="e7f27-372">IndexBuilder.HasName 现已过时</span><span class="sxs-lookup"><span data-stu-id="e7f27-372">IndexBuilder.HasName is now obsolete</span></span>

[<span data-ttu-id="e7f27-373">跟踪问题 #21089</span><span class="sxs-lookup"><span data-stu-id="e7f27-373">Tracking Issue #21089</span></span>](https://github.com/dotnet/efcore/issues/21089)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-374">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-374">Old behavior</span></span>

<span data-ttu-id="e7f27-375">以前，只能在给定的一组属性上定义一个索引。</span><span class="sxs-lookup"><span data-stu-id="e7f27-375">Previously, only one index could be defined over a given set of properties.</span></span> <span data-ttu-id="e7f27-376">索引的数据库名称是使用 IndexBuilder.HasName 配置的。</span><span class="sxs-lookup"><span data-stu-id="e7f27-376">The database name of an index was configured using IndexBuilder.HasName.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-377">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-377">New behavior</span></span>

<span data-ttu-id="e7f27-378">现在，允许在同一组属性上使用多个索引。</span><span class="sxs-lookup"><span data-stu-id="e7f27-378">Multiple indexes are now allowed on the same set or properties.</span></span> <span data-ttu-id="e7f27-379">这些索引现在可以通过模型中的名称来区分。</span><span class="sxs-lookup"><span data-stu-id="e7f27-379">These indexes are now distinguished by a name in the model.</span></span> <span data-ttu-id="e7f27-380">按照约定，模型名称将用作数据库名称；不过，也可以使用 HasDatabaseName 单独进行配置。</span><span class="sxs-lookup"><span data-stu-id="e7f27-380">By convention, the model name is used as the database name; however it can also be configured independently using HasDatabaseName.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-381">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-381">Why</span></span>

<span data-ttu-id="e7f27-382">将来，我们希望使用同一组属性上的不同排序规则对索引启用升序和降序。</span><span class="sxs-lookup"><span data-stu-id="e7f27-382">In the future, we'd like to enable both ascending and descending indexes or indexes with different collations on the same set of properties.</span></span> <span data-ttu-id="e7f27-383">此更改将沿该方向转至其他步骤。</span><span class="sxs-lookup"><span data-stu-id="e7f27-383">This change moves us another step in that direction.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-384">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-384">Mitigations</span></span>

<span data-ttu-id="e7f27-385">之前调用 IndexBuilder.HasName 的任何代码都应更新为调用 HasDatabaseName。</span><span class="sxs-lookup"><span data-stu-id="e7f27-385">Any code that was previously calling IndexBuilder.HasName should be updated to call HasDatabaseName instead.</span></span>

<span data-ttu-id="e7f27-386">如果你的项目包含 EF Core 版本 2.0.0 之前生成的迁移，则可以安全地忽略这些文件中的警告，并通过添加 `#pragma warning disable 612, 618` 将其取消。</span><span class="sxs-lookup"><span data-stu-id="e7f27-386">If your project includes migrations generated prior to EF Core version 2.0.0, you can safely ignore the warning in those files and suppress it by adding `#pragma warning disable 612, 618`.</span></span>

<a name="pluralizer"></a>

### <a name="a-pluralizer-is-now-included-for-scaffolding-reverse-engineered-models"></a><span data-ttu-id="e7f27-387">现已包括用于搭建实施了反向工程的模型的复数化程序</span><span class="sxs-lookup"><span data-stu-id="e7f27-387">A pluralizer is now included for scaffolding reverse engineered models</span></span>

[<span data-ttu-id="e7f27-388">跟踪问题 #11160</span><span class="sxs-lookup"><span data-stu-id="e7f27-388">Tracking Issue #11160</span></span>](https://github.com/dotnet/efcore/issues/11160)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-389">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-389">Old behavior</span></span>

<span data-ttu-id="e7f27-390">以前，必须安装单独的复数化程序包，才能在通过对数据库架构实施反向工程来搭建 DbContext 和实体类型时，设置 DbSet 和集合导航名称的复数形式并设置表名称的单数形式。</span><span class="sxs-lookup"><span data-stu-id="e7f27-390">Previously, you had to install a separate pluralizer package in order to pluralize DbSet and collection navigation names and singularize table names when scaffolding a DbContext and entity types by reverse engineering a database schema.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-391">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-391">New behavior</span></span>

<span data-ttu-id="e7f27-392">EF Core 现在包括使用 [Humanizer](https://github.com/Humanizr/Humanizer) 库的复数化程序。</span><span class="sxs-lookup"><span data-stu-id="e7f27-392">EF Core now includes a pluralizer that uses the [Humanizer](https://github.com/Humanizr/Humanizer) library.</span></span> <span data-ttu-id="e7f27-393">这是 Visual Studio 用来推荐变量名称的同一个库。</span><span class="sxs-lookup"><span data-stu-id="e7f27-393">This is the same library Visual Studio uses to recommend variable names.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-394">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-394">Why</span></span>

<span data-ttu-id="e7f27-395">对集合属性的单词使用复数形式并对类型和引用属性的单词使用单数形式在 .NET 中是惯用的。</span><span class="sxs-lookup"><span data-stu-id="e7f27-395">Using plural forms of words for collection properties and singular forms for types and reference properties is idiomatic in .NET.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-396">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-396">Mitigations</span></span>

<span data-ttu-id="e7f27-397">若要禁用复数化程序，请使用 `dotnet ef dbcontext scaffold` 上的 `--no-pluralize` 选项或 `Scaffold-DbContext` 上的 `-NoPluralize` 开关。</span><span class="sxs-lookup"><span data-stu-id="e7f27-397">To disable the pluralizer, use the `--no-pluralize` option on `dotnet ef dbcontext scaffold` or the `-NoPluralize` switch on `Scaffold-DbContext`.</span></span>

<a name="inavigationbase"></a>

### <a name="inavigationbase-replaces-inavigation-in-some-apis-to-support-skip-navigations"></a><span data-ttu-id="e7f27-398">INavigationBase 替换某些 API 中的 INavigation 以支持跳过导航</span><span class="sxs-lookup"><span data-stu-id="e7f27-398">INavigationBase replaces INavigation in some APIs to support skip navigations</span></span>

[<span data-ttu-id="e7f27-399">跟踪问题 #2568</span><span class="sxs-lookup"><span data-stu-id="e7f27-399">Tracking Issue #2568</span></span>](https://github.com/dotnet/EntityFramework.Docs/issues/2568)

#### <a name="old-behavior"></a><span data-ttu-id="e7f27-400">旧行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-400">Old behavior</span></span>

<span data-ttu-id="e7f27-401">5\.0 之前的 EF Core 仅支持由 `INavigation` 接口表示的一种导航属性形式。</span><span class="sxs-lookup"><span data-stu-id="e7f27-401">EF Core prior to 5.0 supported only one form of navigation property, represented by the `INavigation` interface.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="e7f27-402">新行为</span><span class="sxs-lookup"><span data-stu-id="e7f27-402">New behavior</span></span>

<span data-ttu-id="e7f27-403">EF Core 5.0 使用“跳过导航”引入了多对多关系。</span><span class="sxs-lookup"><span data-stu-id="e7f27-403">EF Core 5.0 introduces many-to-many relationships which use "skip navigations".</span></span> <span data-ttu-id="e7f27-404">这些关系由 `ISkipNavigation` 接口表示，`INavigation` 的大多数功能都向下推送到通用的基接口 `INavigationBase`。</span><span class="sxs-lookup"><span data-stu-id="e7f27-404">These are represented by the `ISkipNavigation` interface, and most of the functionality of `INavigation` has been pushed down to a common base interface: `INavigationBase`.</span></span>

#### <a name="why"></a><span data-ttu-id="e7f27-405">原因</span><span class="sxs-lookup"><span data-stu-id="e7f27-405">Why</span></span>

<span data-ttu-id="e7f27-406">常规导航和跳过导航的多数功能都相同。</span><span class="sxs-lookup"><span data-stu-id="e7f27-406">Most of the functionality between normal and skip navigations is the same.</span></span> <span data-ttu-id="e7f27-407">但是，跳过导航与外键的关系不同于常规导航，因为涉及的 FK 不直接在关系的任意一端，而是在联接实体中。</span><span class="sxs-lookup"><span data-stu-id="e7f27-407">However, skip navigations have a different relationship to foreign keys than normal navigations, since the FKs involved are not directly on either end of the relationship, but rather in the join entity.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="e7f27-408">缓解措施</span><span class="sxs-lookup"><span data-stu-id="e7f27-408">Mitigations</span></span>

<span data-ttu-id="e7f27-409">在许多情况下，应用程序都可以切换为使用新的基接口，并且不会发生任何其他更改。</span><span class="sxs-lookup"><span data-stu-id="e7f27-409">In many cases applications can switch to using the new base interface with no other changes.</span></span> <span data-ttu-id="e7f27-410">但是，如果使用导航访问外键属性，应将应用程序代码限制为仅使用常规导航，或者更新此代码以便执行适当的操作来实现常规导航和跳过导航。</span><span class="sxs-lookup"><span data-stu-id="e7f27-410">However, in cases where the navigation is used to access foreign key properties, application code should either be constrained to only normal navigations, or updated to do the appropriate thing for both normal and skip navigations.</span></span>

<a name="collection-distinct-groupby"></a>

### <a name="some-queries-with-correlated-collection-that-also-use-distinct-or-groupby-are-no-longer-supported"></a><span data-ttu-id="e7f27-411">不再支持使用相关集合同时还使用 `Distinct` 或 `GroupBy` 的查询</span><span class="sxs-lookup"><span data-stu-id="e7f27-411">Some queries with correlated collection that also use `Distinct` or `GroupBy` are no longer supported</span></span>

[<span data-ttu-id="e7f27-412">跟踪问题 #15873</span><span class="sxs-lookup"><span data-stu-id="e7f27-412">Tracking Issue #15873</span></span>](https://github.com/dotnet/efcore/issues/15873)

<span data-ttu-id="e7f27-413">**旧行为**</span><span class="sxs-lookup"><span data-stu-id="e7f27-413">**Old behavior**</span></span>

<span data-ttu-id="e7f27-414">以前我们允许执行其中包含相关集合并且之后紧跟 `GroupBy` 的查询以及一些使用 `Distinct` 的查询。</span><span class="sxs-lookup"><span data-stu-id="e7f27-414">Previously, queries involving correlated collections followed by `GroupBy`, as well as some queries using `Distinct` we allowed to execute.</span></span>

<span data-ttu-id="e7f27-415">GroupBy 示例：</span><span class="sxs-lookup"><span data-stu-id="e7f27-415">GroupBy example:</span></span>

```csharp
context.Parents
    .Select(p => p.Children
        .GroupBy(c => c.School)
        .Select(g => g.Key))
```

<span data-ttu-id="e7f27-416">`Distinct` 示例 - 特别是内部集合投影不包含主键的 `Distinct` 查询：</span><span class="sxs-lookup"><span data-stu-id="e7f27-416">`Distinct` example - specifically `Distinct` queries where inner collection projection doesn't contain the primary key:</span></span>

```csharp
context.Parents
    .Select(p => p.Children
        .Select(c => c.School)
        .Distinct())
```

<span data-ttu-id="e7f27-417">如果内部集合包含任何重复项，则这些查询可能会返回不正确的结果；如果内部集合中的所有元素都是唯一的，则这些查询可能会正常工作。</span><span class="sxs-lookup"><span data-stu-id="e7f27-417">These queries could return incorrect results if the inner collection contained any duplicates, but worked correctly if all the elements in the inner collection were unique.</span></span>

<span data-ttu-id="e7f27-418">**新行为**</span><span class="sxs-lookup"><span data-stu-id="e7f27-418">**New behavior**</span></span>

<span data-ttu-id="e7f27-419">不再支持这些查询。</span><span class="sxs-lookup"><span data-stu-id="e7f27-419">These queries are no longer supported.</span></span> <span data-ttu-id="e7f27-420">将引发异常，指示信息不足，因此无法正确生成结果。</span><span class="sxs-lookup"><span data-stu-id="e7f27-420">Exception is thrown indicating that we don't have enough information to correctly build the results.</span></span>

<span data-ttu-id="e7f27-421">**为什么**</span><span class="sxs-lookup"><span data-stu-id="e7f27-421">**Why**</span></span>

<span data-ttu-id="e7f27-422">如果使用相关集合，我们需要了解实体的主键，才能将集合实体分配到正确的父级中。</span><span class="sxs-lookup"><span data-stu-id="e7f27-422">For correlated collection scenarios we need to know entity's primary key in order to assign collection entities to the correct parent.</span></span> <span data-ttu-id="e7f27-423">如果内部集合不使用 `GroupBy` 或 `Distinct`，只需将丢失的主键添加到投影中。</span><span class="sxs-lookup"><span data-stu-id="e7f27-423">When inner collection doesn't use `GroupBy` or `Distinct`, the missing primary key can simply be added to the projection.</span></span> <span data-ttu-id="e7f27-424">但是，如果使用 `GroupBy` 和 `Distinct`，则无法执行此操作，因为此操作会更改 `GroupBy` 或 `Distinct` 操作的结果。</span><span class="sxs-lookup"><span data-stu-id="e7f27-424">However in case of `GroupBy` and `Distinct` it can't be done because it would change the result of `GroupBy` or `Distinct` operation.</span></span>

<span data-ttu-id="e7f27-425">**缓解措施**</span><span class="sxs-lookup"><span data-stu-id="e7f27-425">**Mitigations**</span></span>

<span data-ttu-id="e7f27-426">将查询重写为不在内部集合上使用 `GroupBy` 或 `Distinct` 操作，改为在客户端执行这些操作。</span><span class="sxs-lookup"><span data-stu-id="e7f27-426">Rewrite the query to not use `GroupBy` or `Distinct` operations on the inner collection, and perform these operations on the client instead.</span></span>

```csharp
context.Parents
    .Select(p => p.Children.Select(c => c.School))
    .ToList()
    .Select(x => x.GroupBy(c => c).Select(g => g.Key))
```

```csharp
context.Parents
    .Select(p => p.Children.Select(c => c.School))
    .ToList()
    .Select(x => x.Distinct())
```

<a name="queryable-projection"></a>

### <a name="using-a-collection-of-queryable-type-in-projection-is-not-supported"></a><span data-ttu-id="e7f27-427">不支持在投影中使用可查询类型的集合</span><span class="sxs-lookup"><span data-stu-id="e7f27-427">Using a collection of Queryable type in projection is not supported</span></span>

[<span data-ttu-id="e7f27-428">跟踪问题 #16314</span><span class="sxs-lookup"><span data-stu-id="e7f27-428">Tracking Issue #16314</span></span>](https://github.com/dotnet/efcore/issues/16314)

<span data-ttu-id="e7f27-429">**旧行为**</span><span class="sxs-lookup"><span data-stu-id="e7f27-429">**Old behavior**</span></span>

<span data-ttu-id="e7f27-430">以前，在某些情况下能够在投影中使用可查询类型的集合，例如用作 `List<T>` 构造函数的参数：</span><span class="sxs-lookup"><span data-stu-id="e7f27-430">Previously, it was possible to use collection of a Queryable type inside the projection in some cases, for example as an argument to a `List<T>` constructor:</span></span>

```csharp
context.Blogs
    .Select(b => new List<Post>(context.Posts.Where(p => p.BlogId == b.Id)))
```

<span data-ttu-id="e7f27-431">**新行为**</span><span class="sxs-lookup"><span data-stu-id="e7f27-431">**New behavior**</span></span>

<span data-ttu-id="e7f27-432">不再支持这些查询。</span><span class="sxs-lookup"><span data-stu-id="e7f27-432">These queries are no longer supported.</span></span> <span data-ttu-id="e7f27-433">将引发异常，指示无法创建可查询类型的对象，并建议如何解决此问题。</span><span class="sxs-lookup"><span data-stu-id="e7f27-433">Exception is thrown indicating that we can't create an object of Queryable type and suggesting how this could be fixed.</span></span>

<span data-ttu-id="e7f27-434">**为什么**</span><span class="sxs-lookup"><span data-stu-id="e7f27-434">**Why**</span></span>

<span data-ttu-id="e7f27-435">无法将可查询类型的对象具体化，因此它们会自动改用 `List<T>` 类型生成。</span><span class="sxs-lookup"><span data-stu-id="e7f27-435">We can't materialize an object of a Queryable type, so they would automatically be created using `List<T>` type instead.</span></span> <span data-ttu-id="e7f27-436">这样通常会由于类型不匹配而引发异常，一些用户对此异常不太了解，并且可能会感到非常意外。</span><span class="sxs-lookup"><span data-stu-id="e7f27-436">This would often cause an exception due to type mismatch which was not very clear and could be surprising to some users.</span></span> <span data-ttu-id="e7f27-437">我们决定识别此模式，并引发更有意义的异常。</span><span class="sxs-lookup"><span data-stu-id="e7f27-437">We decided to recognize the pattern and throw a more meaningful exception.</span></span>

<span data-ttu-id="e7f27-438">**缓解措施**</span><span class="sxs-lookup"><span data-stu-id="e7f27-438">**Mitigations**</span></span>

<span data-ttu-id="e7f27-439">在投影中的可查询对象之后添加 `ToList()` 调用：</span><span class="sxs-lookup"><span data-stu-id="e7f27-439">Add `ToList()` call after the Queryable object in the projection:</span></span>

```csharp
context.Blogs.Select(b => context.Posts.Where(p => p.BlogId == b.Id).ToList())
```
