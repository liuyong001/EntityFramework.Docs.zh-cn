---
title: EF Core 3.x 中的中断性变更 - EF Core
description: Entity Framework Core 3.x 中引入的中断性变更的完整列表
author: ajcvickers
ms.date: 09/05/2020
uid: core/what-is-new/ef-core-3.x/breaking-changes
ms.openlocfilehash: bfcfb7257091d1b6889f7c0af00ddab10e0e12e3
ms.sourcegitcommit: f3512e3a98e685a3ba409c1d0157ce85cc390cf4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2020
ms.locfileid: "94429307"
---
# <a name="breaking-changes-included-in-ef-core-3x"></a><span data-ttu-id="fdd39-103">EF Core 3.x 中包含的中断性变更</span><span class="sxs-lookup"><span data-stu-id="fdd39-103">Breaking changes included in EF Core 3.x</span></span>

<span data-ttu-id="fdd39-104">以下 API 和行为更改有可能导致现有应用程序在升级到 3.x 时中断。</span><span class="sxs-lookup"><span data-stu-id="fdd39-104">The following API and behavior changes have the potential to break existing applications when upgrading them to 3.x.</span></span>
<span data-ttu-id="fdd39-105">我们将仅影响数据库提供程序的更改记录在[提供程序更改](xref:core/providers/provider-log)下。</span><span class="sxs-lookup"><span data-stu-id="fdd39-105">Changes that we expect to only impact database providers are documented under [provider changes](xref:core/providers/provider-log).</span></span>

## <a name="summary"></a><span data-ttu-id="fdd39-106">摘要</span><span class="sxs-lookup"><span data-stu-id="fdd39-106">Summary</span></span>

| <span data-ttu-id="fdd39-107">**中断性变更**</span><span class="sxs-lookup"><span data-stu-id="fdd39-107">**Breaking change**</span></span>                                                                                               | <span data-ttu-id="fdd39-108">**影响**</span><span class="sxs-lookup"><span data-stu-id="fdd39-108">**Impact**</span></span> |
|:------------------------------------------------------------------------------------------------------------------|------------|
| [<span data-ttu-id="fdd39-109">不再在客户端上计算 LINQ 查询</span><span class="sxs-lookup"><span data-stu-id="fdd39-109">LINQ queries are no longer evaluated on the client</span></span>](#linq-queries-are-no-longer-evaluated-on-the-client)         | <span data-ttu-id="fdd39-110">高</span><span class="sxs-lookup"><span data-stu-id="fdd39-110">High</span></span>       |
| [<span data-ttu-id="fdd39-111">EF Core 命令行工具 dotnet ef 不再是 .NET Core SDK 的一部分</span><span class="sxs-lookup"><span data-stu-id="fdd39-111">The EF Core command-line tool, dotnet ef, is no longer part of the .NET Core SDK</span></span>](#dotnet-ef) | <span data-ttu-id="fdd39-112">高</span><span class="sxs-lookup"><span data-stu-id="fdd39-112">High</span></span>      |
| [<span data-ttu-id="fdd39-113">DetectChanges 遵循存储生成的键值</span><span class="sxs-lookup"><span data-stu-id="fdd39-113">DetectChanges honors store-generated key values</span></span>](#dc) | <span data-ttu-id="fdd39-114">高</span><span class="sxs-lookup"><span data-stu-id="fdd39-114">High</span></span>      |
| [<span data-ttu-id="fdd39-115">FromSql、ExecuteSql 和 ExecuteSqlAsync 已重命名</span><span class="sxs-lookup"><span data-stu-id="fdd39-115">FromSql, ExecuteSql, and ExecuteSqlAsync have been renamed</span></span>](#fromsql) | <span data-ttu-id="fdd39-116">高</span><span class="sxs-lookup"><span data-stu-id="fdd39-116">High</span></span>      |
| [<span data-ttu-id="fdd39-117">查询类型与实体类型合并</span><span class="sxs-lookup"><span data-stu-id="fdd39-117">Query types are consolidated with entity types</span></span>](#qt) | <span data-ttu-id="fdd39-118">高</span><span class="sxs-lookup"><span data-stu-id="fdd39-118">High</span></span>      |
| [<span data-ttu-id="fdd39-119">Entity Framework Core 不再是 ASP.NET Core 共享框架的一部分</span><span class="sxs-lookup"><span data-stu-id="fdd39-119">Entity Framework Core is no longer part of the ASP.NET Core shared framework</span></span>](#no-longer) | <span data-ttu-id="fdd39-120">中型</span><span class="sxs-lookup"><span data-stu-id="fdd39-120">Medium</span></span>      |
| [<span data-ttu-id="fdd39-121">默认情况下，现在会立即发生级联删除</span><span class="sxs-lookup"><span data-stu-id="fdd39-121">Cascade deletions now happen immediately by default</span></span>](#cascade) | <span data-ttu-id="fdd39-122">中型</span><span class="sxs-lookup"><span data-stu-id="fdd39-122">Medium</span></span>      |
| [<span data-ttu-id="fdd39-123">单个查询中现在开始预先加载相关实体</span><span class="sxs-lookup"><span data-stu-id="fdd39-123">Eager loading of related entities now happens in a single query</span></span>](#eager-loading-single-query) | <span data-ttu-id="fdd39-124">中型</span><span class="sxs-lookup"><span data-stu-id="fdd39-124">Medium</span></span>      |
| [<span data-ttu-id="fdd39-125">DeleteBehavior.Restrict 具有更简洁的语义</span><span class="sxs-lookup"><span data-stu-id="fdd39-125">DeleteBehavior.Restrict has cleaner semantics</span></span>](#deletebehavior) | <span data-ttu-id="fdd39-126">中型</span><span class="sxs-lookup"><span data-stu-id="fdd39-126">Medium</span></span>      |
| [<span data-ttu-id="fdd39-127">从属类型关系的配置 API 已更改</span><span class="sxs-lookup"><span data-stu-id="fdd39-127">Configuration API for owned type relationships has changed</span></span>](#config) | <span data-ttu-id="fdd39-128">中型</span><span class="sxs-lookup"><span data-stu-id="fdd39-128">Medium</span></span>      |
| [<span data-ttu-id="fdd39-129">每个属性使用独立的内存中整数键生成</span><span class="sxs-lookup"><span data-stu-id="fdd39-129">Each property uses independent in-memory integer key generation</span></span>](#each) | <span data-ttu-id="fdd39-130">中型</span><span class="sxs-lookup"><span data-stu-id="fdd39-130">Medium</span></span>      |
| [<span data-ttu-id="fdd39-131">无跟踪查询不再执行标识解析</span><span class="sxs-lookup"><span data-stu-id="fdd39-131">No-tracking queries no longer perform identity resolution</span></span>](#notrackingresolution) | <span data-ttu-id="fdd39-132">中型</span><span class="sxs-lookup"><span data-stu-id="fdd39-132">Medium</span></span>      |
| [<span data-ttu-id="fdd39-133">元数据 API 更改</span><span class="sxs-lookup"><span data-stu-id="fdd39-133">Metadata API changes</span></span>](#metadata-api-changes) | <span data-ttu-id="fdd39-134">中型</span><span class="sxs-lookup"><span data-stu-id="fdd39-134">Medium</span></span>      |
| [<span data-ttu-id="fdd39-135">特定于提供程序的元数据 API 更改</span><span class="sxs-lookup"><span data-stu-id="fdd39-135">Provider-specific Metadata API changes</span></span>](#provider) | <span data-ttu-id="fdd39-136">中型</span><span class="sxs-lookup"><span data-stu-id="fdd39-136">Medium</span></span>      |
| [<span data-ttu-id="fdd39-137">UseRowNumberForPaging 已删除</span><span class="sxs-lookup"><span data-stu-id="fdd39-137">UseRowNumberForPaging has been removed</span></span>](#urn) | <span data-ttu-id="fdd39-138">中型</span><span class="sxs-lookup"><span data-stu-id="fdd39-138">Medium</span></span>      |
| [<span data-ttu-id="fdd39-139">FromSql 方法在与存储过程配合使用时，无法进行组合</span><span class="sxs-lookup"><span data-stu-id="fdd39-139">FromSql method when used with stored procedure cannot be composed</span></span>](#fromsqlsproc) | <span data-ttu-id="fdd39-140">中型</span><span class="sxs-lookup"><span data-stu-id="fdd39-140">Medium</span></span>      |
| [<span data-ttu-id="fdd39-141">只能在查询根上指定 FromSql 方法</span><span class="sxs-lookup"><span data-stu-id="fdd39-141">FromSql methods can only be specified on query roots</span></span>](#fromsql) | <span data-ttu-id="fdd39-142">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-142">Low</span></span>      |
| [<span data-ttu-id="fdd39-143">不再在实体实例上设置临时键值</span><span class="sxs-lookup"><span data-stu-id="fdd39-143">Temporary key values are no longer set onto entity instances</span></span>](#tkv) | <span data-ttu-id="fdd39-144">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-144">Low</span></span>      |
| [<span data-ttu-id="fdd39-145">与主体共享表的依赖实体现为可选项</span><span class="sxs-lookup"><span data-stu-id="fdd39-145">Dependent entities sharing the table with the principal are now optional</span></span>](#de) | <span data-ttu-id="fdd39-146">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-146">Low</span></span>      |
| [<span data-ttu-id="fdd39-147">与并发标记列共享表的所有实体均必须将其映射到属性</span><span class="sxs-lookup"><span data-stu-id="fdd39-147">All entities sharing a table with a concurrency token column have to map it to a property</span></span>](#aes) | <span data-ttu-id="fdd39-148">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-148">Low</span></span>      |
| [<span data-ttu-id="fdd39-149">如果没有所有者，则无法使用跟踪查询来查询从属实体</span><span class="sxs-lookup"><span data-stu-id="fdd39-149">Owned entities cannot be queried without the owner using a tracking query</span></span>](#owned-query) | <span data-ttu-id="fdd39-150">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-150">Low</span></span>      |
| [<span data-ttu-id="fdd39-151">对于所有派生的类型而言，从未映射的类型继承的属性现在会映射到一个列中</span><span class="sxs-lookup"><span data-stu-id="fdd39-151">Inherited properties from unmapped types are now mapped to a single column for all derived types</span></span>](#ip) | <span data-ttu-id="fdd39-152">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-152">Low</span></span>      |
| [<span data-ttu-id="fdd39-153">外键属性约定不再匹配与主体属性相同的名称</span><span class="sxs-lookup"><span data-stu-id="fdd39-153">The foreign key property convention no longer matches same name as the principal property</span></span>](#fkp) | <span data-ttu-id="fdd39-154">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-154">Low</span></span>      |
| [<span data-ttu-id="fdd39-155">现在，如果在 TransactionScope 完成前不再使用数据库连接，则该连接会关闭</span><span class="sxs-lookup"><span data-stu-id="fdd39-155">Database connection is now closed if not used anymore before the TransactionScope has been completed</span></span>](#dbc) | <span data-ttu-id="fdd39-156">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-156">Low</span></span>      |
| [<span data-ttu-id="fdd39-157">默认情况下使用支持字段</span><span class="sxs-lookup"><span data-stu-id="fdd39-157">Backing fields are used by default</span></span>](#backing-fields-are-used-by-default) | <span data-ttu-id="fdd39-158">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-158">Low</span></span>      |
| [<span data-ttu-id="fdd39-159">如果找到多个兼容的支持字段，则引发</span><span class="sxs-lookup"><span data-stu-id="fdd39-159">Throw if multiple compatible backing fields are found</span></span>](#throw-if-multiple-compatible-backing-fields-are-found) | <span data-ttu-id="fdd39-160">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-160">Low</span></span>      |
| [<span data-ttu-id="fdd39-161">“仅字段”属性名应与字段名匹配</span><span class="sxs-lookup"><span data-stu-id="fdd39-161">Field-only property names should match the field name</span></span>](#field-only-property-names-should-match-the-field-name) | <span data-ttu-id="fdd39-162">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-162">Low</span></span>      |
| [<span data-ttu-id="fdd39-163">AddDbContext/AddDbContextPool 不再调用 AddLogging 和 AddMemoryCache</span><span class="sxs-lookup"><span data-stu-id="fdd39-163">AddDbContext/AddDbContextPool no longer call AddLogging and AddMemoryCache</span></span>](#adddbc) | <span data-ttu-id="fdd39-164">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-164">Low</span></span>      |
| [<span data-ttu-id="fdd39-165">AddEntityFramework\* 添加具有大小限制的 IMemoryCache</span><span class="sxs-lookup"><span data-stu-id="fdd39-165">AddEntityFramework\* adds IMemoryCache with a size limit</span></span>](#addentityframework-adds-imemorycache-with-a-size-limit) | <span data-ttu-id="fdd39-166">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-166">Low</span></span>      |
| [<span data-ttu-id="fdd39-167">DbContext.Entry 现在执行本地 DetectChanges</span><span class="sxs-lookup"><span data-stu-id="fdd39-167">DbContext.Entry now performs a local DetectChanges</span></span>](#dbe) | <span data-ttu-id="fdd39-168">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-168">Low</span></span>      |
| [<span data-ttu-id="fdd39-169">默认情况下，字符串和字节数组键不是客户端生成的</span><span class="sxs-lookup"><span data-stu-id="fdd39-169">String and byte array keys are not client-generated by default</span></span>](#string-and-byte-array-keys-are-not-client-generated-by-default) | <span data-ttu-id="fdd39-170">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-170">Low</span></span>      |
| [<span data-ttu-id="fdd39-171">ILoggerFactory 现在是一个在一定范围内有效的服务</span><span class="sxs-lookup"><span data-stu-id="fdd39-171">ILoggerFactory is now a scoped service</span></span>](#ilf) | <span data-ttu-id="fdd39-172">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-172">Low</span></span>      |
| [<span data-ttu-id="fdd39-173">延迟加载代理不再假定导航属性已完全加载</span><span class="sxs-lookup"><span data-stu-id="fdd39-173">Lazy-loading proxies no longer assume navigation properties are fully loaded</span></span>](#lazy-loading-proxies-no-longer-assume-navigation-properties-are-fully-loaded) | <span data-ttu-id="fdd39-174">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-174">Low</span></span>      |
| [<span data-ttu-id="fdd39-175">默认情况下，现在过度创建内部服务提供程序是一个错误</span><span class="sxs-lookup"><span data-stu-id="fdd39-175">Excessive creation of internal service providers is now an error by default</span></span>](#excessive-creation-of-internal-service-providers-is-now-an-error-by-default) | <span data-ttu-id="fdd39-176">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-176">Low</span></span>      |
| [<span data-ttu-id="fdd39-177">使用单个字符串调用 HasOne/HasMany 的新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-177">New behavior for HasOne/HasMany called with a single string</span></span>](#nbh) | <span data-ttu-id="fdd39-178">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-178">Low</span></span>      |
| [<span data-ttu-id="fdd39-179">多个异步方法的返回类型已从 Task 更改为 ValueTask</span><span class="sxs-lookup"><span data-stu-id="fdd39-179">The return type for several async methods has been changed from Task to ValueTask</span></span>](#rtnt) | <span data-ttu-id="fdd39-180">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-180">Low</span></span>      |
| [<span data-ttu-id="fdd39-181">关系式：TypeMapping 注释现在只是 TypeMapping</span><span class="sxs-lookup"><span data-stu-id="fdd39-181">The Relational:TypeMapping annotation is now just TypeMapping</span></span>](#rtt) | <span data-ttu-id="fdd39-182">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-182">Low</span></span>      |
| [<span data-ttu-id="fdd39-183">派生类型上的 ToTable 会引发异常</span><span class="sxs-lookup"><span data-stu-id="fdd39-183">ToTable on a derived type throws an exception</span></span>](#totable-on-a-derived-type-throws-an-exception) | <span data-ttu-id="fdd39-184">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-184">Low</span></span>      |
| [<span data-ttu-id="fdd39-185">EF Core 不再发送 pragma 来执行 SQLite FK</span><span class="sxs-lookup"><span data-stu-id="fdd39-185">EF Core no longer sends pragma for SQLite FK enforcement</span></span>](#pragma) | <span data-ttu-id="fdd39-186">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-186">Low</span></span>      |
| [<span data-ttu-id="fdd39-187">Microsoft.EntityFrameworkCore.Sqlite 现在依赖于 SQLitePCLRaw.bundle_e_sqlite3</span><span class="sxs-lookup"><span data-stu-id="fdd39-187">Microsoft.EntityFrameworkCore.Sqlite now depends on SQLitePCLRaw.bundle_e_sqlite3</span></span>](#sqlite3) | <span data-ttu-id="fdd39-188">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-188">Low</span></span>      |
| [<span data-ttu-id="fdd39-189">GUID 值现在以文本形式存储在 SQLite 上</span><span class="sxs-lookup"><span data-stu-id="fdd39-189">Guid values are now stored as TEXT on SQLite</span></span>](#guid) | <span data-ttu-id="fdd39-190">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-190">Low</span></span>      |
| [<span data-ttu-id="fdd39-191">Char 值现在以文本形式存储在 SQLite 上</span><span class="sxs-lookup"><span data-stu-id="fdd39-191">Char values are now stored as TEXT on SQLite</span></span>](#char) | <span data-ttu-id="fdd39-192">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-192">Low</span></span>      |
| [<span data-ttu-id="fdd39-193">现在使用固定区域性的日历生成迁移 ID</span><span class="sxs-lookup"><span data-stu-id="fdd39-193">Migration IDs are now generated using the invariant culture's calendar</span></span>](#migid) | <span data-ttu-id="fdd39-194">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-194">Low</span></span>      |
| [<span data-ttu-id="fdd39-195">已从 IDbContextOptionsExtension 中删除扩展信息/元数据</span><span class="sxs-lookup"><span data-stu-id="fdd39-195">Extension info/metadata has been removed from IDbContextOptionsExtension</span></span>](#xinfo) | <span data-ttu-id="fdd39-196">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-196">Low</span></span>      |
| [<span data-ttu-id="fdd39-197">已重命名 LogQueryPossibleExceptionWithAggregateOperator</span><span class="sxs-lookup"><span data-stu-id="fdd39-197">LogQueryPossibleExceptionWithAggregateOperator has been renamed</span></span>](#lqpe) | <span data-ttu-id="fdd39-198">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-198">Low</span></span>      |
| [<span data-ttu-id="fdd39-199">阐明 API 的外键约束名称</span><span class="sxs-lookup"><span data-stu-id="fdd39-199">Clarify API for foreign key constraint names</span></span>](#clarify) | <span data-ttu-id="fdd39-200">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-200">Low</span></span>      |
| [<span data-ttu-id="fdd39-201">IRelationalDatabaseCreator.HasTables/HasTablesAsync 已公开</span><span class="sxs-lookup"><span data-stu-id="fdd39-201">IRelationalDatabaseCreator.HasTables/HasTablesAsync have been made public</span></span>](#irdc2) | <span data-ttu-id="fdd39-202">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-202">Low</span></span>      |
| [<span data-ttu-id="fdd39-203">Microsoft.EntityFrameworkCore.Design 现在是 DevelopmentDependency 包</span><span class="sxs-lookup"><span data-stu-id="fdd39-203">Microsoft.EntityFrameworkCore.Design is now a DevelopmentDependency package</span></span>](#dip) | <span data-ttu-id="fdd39-204">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-204">Low</span></span>      |
| [<span data-ttu-id="fdd39-205">SQLitePCL.raw 已更新为版本 2.0.0</span><span class="sxs-lookup"><span data-stu-id="fdd39-205">SQLitePCL.raw updated to version 2.0.0</span></span>](#SQLitePCL) | <span data-ttu-id="fdd39-206">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-206">Low</span></span>      |
| [<span data-ttu-id="fdd39-207">NetTopologySuite 已更新为版本 2.0.0</span><span class="sxs-lookup"><span data-stu-id="fdd39-207">NetTopologySuite updated to version 2.0.0</span></span>](#NetTopologySuite) | <span data-ttu-id="fdd39-208">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-208">Low</span></span>      |
| [<span data-ttu-id="fdd39-209">使用 Microsoft.Data.SqlClient 而不是 System.Data.SqlClient</span><span class="sxs-lookup"><span data-stu-id="fdd39-209">Microsoft.Data.SqlClient is used instead of System.Data.SqlClient</span></span>](#SqlClient) | <span data-ttu-id="fdd39-210">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-210">Low</span></span>      |
| [<span data-ttu-id="fdd39-211">必须配置多个不明确的自引用关系</span><span class="sxs-lookup"><span data-stu-id="fdd39-211">Multiple ambiguous self-referencing relationships must be configured</span></span>](#mersa) | <span data-ttu-id="fdd39-212">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-212">Low</span></span>      |
| [<span data-ttu-id="fdd39-213">DbFunction.Schema 为 null 或者空字符串将其配置为位于模型的默认架构中</span><span class="sxs-lookup"><span data-stu-id="fdd39-213">DbFunction.Schema being null or empty string configures it to be in model's default schema</span></span>](#udf-empty-string) | <span data-ttu-id="fdd39-214">低</span><span class="sxs-lookup"><span data-stu-id="fdd39-214">Low</span></span>      |
| [<span data-ttu-id="fdd39-215">~~EF Core 3.0 面向 .NET Standard 2.1，而不是 .NET Standard 2.0~~ 已还原</span><span class="sxs-lookup"><span data-stu-id="fdd39-215">~~EF Core 3.0 targets .NET Standard 2.1 rather than .NET Standard 2.0~~ Reverted</span></span>](#netstandard21) | |
| [<span data-ttu-id="fdd39-216">~~在调试级别记录查询执行~~已还原</span><span class="sxs-lookup"><span data-stu-id="fdd39-216">~~Query execution is logged at Debug level~~ Reverted</span></span>](#qe) | |

## <a name="high-impact-changes"></a><span data-ttu-id="fdd39-217">影响较大的更改</span><span class="sxs-lookup"><span data-stu-id="fdd39-217">High-impact changes</span></span>

### <a name="linq-queries-are-no-longer-evaluated-on-the-client"></a><span data-ttu-id="fdd39-218">不再在客户端上计算 LINQ 查询</span><span class="sxs-lookup"><span data-stu-id="fdd39-218">LINQ queries are no longer evaluated on the client</span></span>

<span data-ttu-id="fdd39-219">[跟踪问题 #14935](https://github.com/dotnet/efcore/issues/14935)
[另请参阅问题 #12795](https://github.com/dotnet/efcore/issues/12795)</span><span class="sxs-lookup"><span data-stu-id="fdd39-219">[Tracking Issue #14935](https://github.com/dotnet/efcore/issues/14935)
[Also see issue #12795](https://github.com/dotnet/efcore/issues/12795)</span></span>

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-220">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-220">Old behavior</span></span>

<span data-ttu-id="fdd39-221">在 3.0 之前，当 EF Core 无法将查询中的表达式转换为 SQL 或参数时，它会在客户端上自动计算表达式的值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-221">Before 3.0, when EF Core couldn't convert an expression that was part of a query to either SQL or a parameter, it automatically evaluated the expression on the client.</span></span>
<span data-ttu-id="fdd39-222">默认情况下，客户端对潜在的昂贵表达式的计算仅触发警告。</span><span class="sxs-lookup"><span data-stu-id="fdd39-222">By default, client evaluation of potentially expensive expressions only triggered a warning.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-223">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-223">New behavior</span></span>

<span data-ttu-id="fdd39-224">从 3.0 开始，EF Core 仅允许在客户端上计算顶级投影中的表达式（查询中的最后一个 `Select()` 调用）。</span><span class="sxs-lookup"><span data-stu-id="fdd39-224">Starting with 3.0, EF Core only allows expressions in the top-level projection (the last `Select()` call in the query) to be evaluated on the client.</span></span>
<span data-ttu-id="fdd39-225">当查询的任何其他部分中的表达式无法转换为 SQL 或参数时，将引发异常。</span><span class="sxs-lookup"><span data-stu-id="fdd39-225">When expressions in any other part of the query can't be converted to either SQL or a parameter, an exception is thrown.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-226">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-226">Why</span></span>

<span data-ttu-id="fdd39-227">自动的客户端查询计算允许执行许多查询，即使它们的重要组成部分无法转换。</span><span class="sxs-lookup"><span data-stu-id="fdd39-227">Automatic client evaluation of queries allows many queries to be executed even if important parts of them can't be translated.</span></span>
<span data-ttu-id="fdd39-228">此行为可能导致意外且具有潜在破坏性的行为，这些行为可能仅在生产中变得明显。</span><span class="sxs-lookup"><span data-stu-id="fdd39-228">This behavior can result in unexpected and potentially damaging behavior that may only become evident in production.</span></span>
<span data-ttu-id="fdd39-229">例如，`Where()` 调用中无法转换的条件可能导致表中的所有行从数据库服务器传输且筛选器应用于客户端。</span><span class="sxs-lookup"><span data-stu-id="fdd39-229">For example, a condition in a `Where()` call which can't be translated can cause all rows from the table to be transferred from the database server, and the filter to be applied on the client.</span></span>
<span data-ttu-id="fdd39-230">如果在开发中表中只包含几行，则不容易检测到这种情况，但是当应用程序转入生产环节时，由于表中可能包含数百万行，这种情况会非常严重。</span><span class="sxs-lookup"><span data-stu-id="fdd39-230">This situation can easily go undetected if the table contains only a few rows in development, but hit hard when the application moves to production, where the table may contain millions of rows.</span></span>
<span data-ttu-id="fdd39-231">在开发过程中，客户端求值警告也很容易被忽视。</span><span class="sxs-lookup"><span data-stu-id="fdd39-231">Client evaluation warnings also proved too easy to ignore during development.</span></span>

<span data-ttu-id="fdd39-232">除此之外，自动客户端计算可能会导致问题，其中改进特定表达式的查询转换会导致版本之间发生意外中断性变更。</span><span class="sxs-lookup"><span data-stu-id="fdd39-232">Besides this, automatic client evaluation can lead to issues in which improving query translation for specific expressions caused unintended breaking changes between releases.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-233">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-233">Mitigations</span></span>

<span data-ttu-id="fdd39-234">如果无法完全转换查询，则以可转换的形式重写查询，或使用 `AsEnumerable()`、`ToList()` 或类似信息将数据显式返回客户端，然后可以进一步使用 LINQ 到对象处理。</span><span class="sxs-lookup"><span data-stu-id="fdd39-234">If a query can't be fully translated, then either rewrite the query in a form that can be translated, or use `AsEnumerable()`, `ToList()`, or similar to explicitly bring data back to the client where it can then be further processed using LINQ-to-Objects.</span></span>

<a name="no-longer"></a>

## <a name="medium-impact-changes"></a><span data-ttu-id="fdd39-235">影响中等的更改</span><span class="sxs-lookup"><span data-stu-id="fdd39-235">Medium-impact changes</span></span>

### <a name="entity-framework-core-is-no-longer-part-of-the-aspnet-core-shared-framework"></a><span data-ttu-id="fdd39-236">Entity Framework Core 不再是 ASP.NET Core 共享框架的一部分</span><span class="sxs-lookup"><span data-stu-id="fdd39-236">Entity Framework Core is no longer part of the ASP.NET Core shared framework</span></span>

[<span data-ttu-id="fdd39-237">跟踪问题公告#325</span><span class="sxs-lookup"><span data-stu-id="fdd39-237">Tracking Issue Announcements#325</span></span>](https://github.com/aspnet/Announcements/issues/325)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-238">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-238">Old behavior</span></span>

<span data-ttu-id="fdd39-239">在 ASP.NET Core 3.0 之前，当向 `Microsoft.AspNetCore.App` 或 `Microsoft.AspNetCore.All` 添加包引用时，它将包括 EF Core 和一些 EF Core 数据提供程序（如 SQL Server 提供程序）。</span><span class="sxs-lookup"><span data-stu-id="fdd39-239">Before ASP.NET Core 3.0, when you added a package reference to `Microsoft.AspNetCore.App` or `Microsoft.AspNetCore.All`, it would include EF Core and some of the EF Core data providers like the SQL Server provider.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-240">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-240">New behavior</span></span>

<span data-ttu-id="fdd39-241">从 3.0 开始，ASP.NET Core 共享框架不包括 EF Core 或任何 EF Core 数据提供程序。</span><span class="sxs-lookup"><span data-stu-id="fdd39-241">Starting in 3.0, the ASP.NET Core shared framework doesn't include EF Core or any EF Core data providers.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-242">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-242">Why</span></span>

<span data-ttu-id="fdd39-243">在此更改之前，获取 EF Core 需要不同的步骤，具体取决于应用程序是否是面向 ASP.NET Core 和 SQL Server。</span><span class="sxs-lookup"><span data-stu-id="fdd39-243">Before this change, getting EF Core required different steps depending on whether the application targeted ASP.NET Core and SQL Server or not.</span></span>
<span data-ttu-id="fdd39-244">此外，升级 ASP.NET Core 会强制升级 EF Core 和 SQL Server 提供程序，这并不总是可取的。</span><span class="sxs-lookup"><span data-stu-id="fdd39-244">Also, upgrading ASP.NET Core forced the upgrade of EF Core and the SQL Server provider, which isn't always desirable.</span></span>

<span data-ttu-id="fdd39-245">通过此更改，通过所有提供程序、支持的 .NET 实现和应用程序类型获取 EF Core 的体验都是一致的。</span><span class="sxs-lookup"><span data-stu-id="fdd39-245">With this change, the experience of getting EF Core is the same across all providers, supported .NET implementations and application types.</span></span>
<span data-ttu-id="fdd39-246">开发人员现在还可以准确控制何时升级 EF Core 和 EF Core 数据提供程序。</span><span class="sxs-lookup"><span data-stu-id="fdd39-246">Developers can also now control exactly when EF Core and EF Core data providers are upgraded.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-247">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-247">Mitigations</span></span>

<span data-ttu-id="fdd39-248">若要在 ASP.NET Core 3.0 应用程序或任何其他受支持的应用程序中使用 EF Core，请显式添加对应用程序将使用的 EF Core 数据库提供程序的包引用。</span><span class="sxs-lookup"><span data-stu-id="fdd39-248">To use EF Core in an ASP.NET Core 3.0 application or any other supported application, explicitly add a package reference to the EF Core database provider that your application will use.</span></span>

<a name="dotnet-ef"></a>

### <a name="the-ef-core-command-line-tool-dotnet-ef-is-no-longer-part-of-the-net-core-sdk"></a><span data-ttu-id="fdd39-249">EF Core 命令行工具 dotnet ef 不再是 .NET Core SDK 的一部分</span><span class="sxs-lookup"><span data-stu-id="fdd39-249">The EF Core command-line tool, dotnet ef, is no longer part of the .NET Core SDK</span></span>

[<span data-ttu-id="fdd39-250">跟踪问题 #14016</span><span class="sxs-lookup"><span data-stu-id="fdd39-250">Tracking Issue #14016</span></span>](https://github.com/dotnet/efcore/issues/14016)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-251">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-251">Old behavior</span></span>

<span data-ttu-id="fdd39-252">.NET Core SDK 3.0 以前的版本包含 `dotnet ef` 工具，可以随时从任何项目的命令行使用，无需额外的步骤。</span><span class="sxs-lookup"><span data-stu-id="fdd39-252">Before 3.0, the `dotnet ef` tool was included in the .NET Core SDK and was readily available to use from the command line from any project without requiring extra steps.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-253">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-253">New behavior</span></span>

<span data-ttu-id="fdd39-254">从 3.0 版开始，.NET SDK 不再包含 `dotnet ef` 工具，因此，在使用它之前，必须将其明确安装为本地或全局工具。</span><span class="sxs-lookup"><span data-stu-id="fdd39-254">Starting in 3.0, the .NET SDK does not include the `dotnet ef` tool, so before you can use it you have to explicitly install it as a local or global tool.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-255">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-255">Why</span></span>

<span data-ttu-id="fdd39-256">此更改允许我们在 NuGet 上将 `dotnet ef` 作为常规 .NET CLI 工具分发和更新，这与 EF Core 3.0 也始终作为 NuGet 包分发的事实一致。</span><span class="sxs-lookup"><span data-stu-id="fdd39-256">This change allows us to distribute and update `dotnet ef` as a regular .NET CLI tool on NuGet, consistent with the fact that the EF Core 3.0 is also always distributed as a NuGet package.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-257">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-257">Mitigations</span></span>

<span data-ttu-id="fdd39-258">为了能够管理迁移或构架 `DbContext`，请安装 `dotnet-ef` 作为全局工具：</span><span class="sxs-lookup"><span data-stu-id="fdd39-258">To be able to manage migrations or scaffold a `DbContext`, install `dotnet-ef` as a global tool:</span></span>

```dotnetcli
dotnet tool install --global dotnet-ef
```

<span data-ttu-id="fdd39-259">使用[工具清单文件](/dotnet/core/tools/global-tools#install-a-local-tool)恢复声明为工具依赖项的项目依赖项时，还可以将其作为本地工具获取。</span><span class="sxs-lookup"><span data-stu-id="fdd39-259">You can also obtain it a local tool when you restore the dependencies of a project that declares it as a tooling dependency using a [tool manifest file](/dotnet/core/tools/global-tools#install-a-local-tool).</span></span>

## <a name="low-impact-changes"></a><span data-ttu-id="fdd39-260">影响较小的更改</span><span class="sxs-lookup"><span data-stu-id="fdd39-260">Low-impact changes</span></span>

<a name="fromsql"></a>

### <a name="fromsql-executesql-and-executesqlasync-have-been-renamed"></a><span data-ttu-id="fdd39-261">FromSql、ExecuteSql 和 ExecuteSqlAsync 已重命名</span><span class="sxs-lookup"><span data-stu-id="fdd39-261">FromSql, ExecuteSql, and ExecuteSqlAsync have been renamed</span></span>

[<span data-ttu-id="fdd39-262">跟踪问题 #10996</span><span class="sxs-lookup"><span data-stu-id="fdd39-262">Tracking Issue #10996</span></span>](https://github.com/dotnet/efcore/issues/10996)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-263">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-263">Old behavior</span></span>

<span data-ttu-id="fdd39-264">在 EF Core 3.0 之前，这些方法名称是重载的，它们使用普通字符串或应内插到 SQL 和参数中的字符串。</span><span class="sxs-lookup"><span data-stu-id="fdd39-264">Before EF Core 3.0, these method names were overloaded to work with either a normal string or a string that should be interpolated into SQL and parameters.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-265">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-265">New behavior</span></span>

<span data-ttu-id="fdd39-266">自 EF Core 3.0 起，可使用 `FromSqlRaw`、`ExecuteSqlRaw` 和 `ExecuteSqlRawAsync` 创建一个参数化的查询，其中参数是从查询字符串中单独传递的。</span><span class="sxs-lookup"><span data-stu-id="fdd39-266">Starting with EF Core 3.0, use `FromSqlRaw`, `ExecuteSqlRaw`, and `ExecuteSqlRawAsync` to create a parameterized query where the parameters are passed separately from the query string.</span></span>
<span data-ttu-id="fdd39-267">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-267">For example:</span></span>

```csharp
context.Products.FromSqlRaw(
    "SELECT * FROM Products WHERE Name = {0}",
    product.Name);
```

<span data-ttu-id="fdd39-268">使用 `FromSqlInterpolated`、`ExecuteSqlInterpolated` 和 `ExecuteSqlInterpolatedAsync` 创建一个参数化的查询，其中参数作为内插查询字符串的一部分进行传递。</span><span class="sxs-lookup"><span data-stu-id="fdd39-268">Use `FromSqlInterpolated`, `ExecuteSqlInterpolated`, and `ExecuteSqlInterpolatedAsync` to create a parameterized query where the parameters are passed as part of an interpolated query string.</span></span>
<span data-ttu-id="fdd39-269">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-269">For example:</span></span>

```csharp
context.Products.FromSqlInterpolated(
    $"SELECT * FROM Products WHERE Name = {product.Name}");
```

<span data-ttu-id="fdd39-270">请注意，上述两个查询都将生成 SQL 参数相同的同一参数化的 SQL。</span><span class="sxs-lookup"><span data-stu-id="fdd39-270">Note that both of the queries above will produce the same parameterized SQL with the same SQL parameters.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-271">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-271">Why</span></span>

<span data-ttu-id="fdd39-272">此类方法重载使得在意图调用内插字符串方法时很容易意外调用原始字符串方法，反之亦然。</span><span class="sxs-lookup"><span data-stu-id="fdd39-272">Method overloads like this make it very easy to accidentally call the raw string method when the intent was to call the interpolated string method, and the other way around.</span></span>
<span data-ttu-id="fdd39-273">这会导致查询中的本该参数化的结果没有参数化。</span><span class="sxs-lookup"><span data-stu-id="fdd39-273">This could result in queries not being parameterized when they should have been.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-274">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-274">Mitigations</span></span>

<span data-ttu-id="fdd39-275">切换到使用新的方法名称。</span><span class="sxs-lookup"><span data-stu-id="fdd39-275">Switch to use the new method names.</span></span>

<a name="fromsqlsproc"></a>

### <a name="fromsql-method-when-used-with-stored-procedure-cannot-be-composed"></a><span data-ttu-id="fdd39-276">FromSql 方法在与存储过程配合使用时，无法进行组合</span><span class="sxs-lookup"><span data-stu-id="fdd39-276">FromSql method when used with stored procedure cannot be composed</span></span>

[<span data-ttu-id="fdd39-277">跟踪问题 #15392</span><span class="sxs-lookup"><span data-stu-id="fdd39-277">Tracking Issue #15392</span></span>](https://github.com/dotnet/efcore/issues/15392)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-278">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-278">Old behavior</span></span>

<span data-ttu-id="fdd39-279">在 EF Core 3.0 之前，FromSql 方法已尝试检测是否可对传入的 SQL 进行组合。</span><span class="sxs-lookup"><span data-stu-id="fdd39-279">Before EF Core 3.0, FromSql method tried to detect if the passed SQL can be composed upon.</span></span> <span data-ttu-id="fdd39-280">当 SQL 像存储过程那样不可组合时，该方法进行客户端评估。</span><span class="sxs-lookup"><span data-stu-id="fdd39-280">It did client evaluation when the SQL was non-composable like a stored procedure.</span></span> <span data-ttu-id="fdd39-281">以下查询在服务器上运行存储过程并在客户端执行 FirstOrDefault。</span><span class="sxs-lookup"><span data-stu-id="fdd39-281">The following query worked by running the stored procedure on the server and doing FirstOrDefault on the client side.</span></span>

```csharp
context.Products.FromSqlRaw("[dbo].[Ten Most Expensive Products]").FirstOrDefault();
```

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-282">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-282">New behavior</span></span>

<span data-ttu-id="fdd39-283">从 EF Core 3.0 开始，EF Core 将不再尝试分析 SQL。</span><span class="sxs-lookup"><span data-stu-id="fdd39-283">Starting with EF Core 3.0, EF Core will not try to parse the SQL.</span></span> <span data-ttu-id="fdd39-284">因此，如果在 FromSqlRaw/FromSqlInterpolated 之后组合，则 EF Core 会通过引发子查询来组合 SQL。</span><span class="sxs-lookup"><span data-stu-id="fdd39-284">So if you are composing after FromSqlRaw/FromSqlInterpolated, then EF Core will compose the SQL by causing sub query.</span></span> <span data-ttu-id="fdd39-285">因此，如果将存储过程用于组合，则出现无效 SQL 语法的异常。</span><span class="sxs-lookup"><span data-stu-id="fdd39-285">So if you are using a stored procedure with composition then you will get an exception for invalid SQL syntax.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-286">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-286">Why</span></span>

<span data-ttu-id="fdd39-287">EF Core 3.0 不支持自动客户端评估，因为容易出错，如[此处](#linq-queries-are-no-longer-evaluated-on-the-client)所述。</span><span class="sxs-lookup"><span data-stu-id="fdd39-287">EF Core 3.0 does not support automatic client evaluation, since it was error prone as explained [here](#linq-queries-are-no-longer-evaluated-on-the-client).</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-288">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-288">Mitigations</span></span>

<span data-ttu-id="fdd39-289">如果在 FromSqlRaw/FromSqlInterpolated 中使用存储过程，你了解无法对其进行组合，因此可以紧随 FromSql 方法调用添加 AsEnumerable/AsAsyncEnumerable，以避免在服务器端上进行任何组合。</span><span class="sxs-lookup"><span data-stu-id="fdd39-289">If you are using a stored procedure in FromSqlRaw/FromSqlInterpolated, you know that it cannot be composed upon, so you can add __AsEnumerable/AsAsyncEnumerable__ right after the FromSql method call to avoid any composition on server side.</span></span>

```csharp
context.Products.FromSqlRaw("[dbo].[Ten Most Expensive Products]").AsEnumerable().FirstOrDefault();
```

<a name="fromsql"></a>

### <a name="fromsql-methods-can-only-be-specified-on-query-roots"></a><span data-ttu-id="fdd39-290">只能在查询根上指定 FromSql 方法</span><span class="sxs-lookup"><span data-stu-id="fdd39-290">FromSql methods can only be specified on query roots</span></span>

[<span data-ttu-id="fdd39-291">跟踪问题 #15704</span><span class="sxs-lookup"><span data-stu-id="fdd39-291">Tracking Issue #15704</span></span>](https://github.com/dotnet/efcore/issues/15704)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-292">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-292">Old behavior</span></span>

<span data-ttu-id="fdd39-293">在 EF Core 3.0 之前，可以在查询中的任意位置指定 `FromSql` 方法。</span><span class="sxs-lookup"><span data-stu-id="fdd39-293">Before EF Core 3.0, the `FromSql` method could be specified anywhere in the query.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-294">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-294">New behavior</span></span>

<span data-ttu-id="fdd39-295">从 EF Core 3.0 开始，只能在查询根上（即，直接在 `DbSet<>` 上）指定新的 `FromSqlRaw` 和 `FromSqlInterpolated` 方法（替换 `FromSql`）。</span><span class="sxs-lookup"><span data-stu-id="fdd39-295">Starting with EF Core 3.0, the new `FromSqlRaw` and `FromSqlInterpolated` methods (which replace `FromSql`) can only be specified on query roots, i.e. directly on the `DbSet<>`.</span></span> <span data-ttu-id="fdd39-296">尝试在其他任何位置指定这些方法将导致编译错误。</span><span class="sxs-lookup"><span data-stu-id="fdd39-296">Attempting to specify them anywhere else will result in a compilation error.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-297">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-297">Why</span></span>

<span data-ttu-id="fdd39-298">在除 `DbSet` 之外的任意位置指定 `FromSql` 没有附加含义或附加值，并且可能在某些情况下存在多义性。</span><span class="sxs-lookup"><span data-stu-id="fdd39-298">Specifying `FromSql` anywhere other than on a `DbSet` had no added meaning or added value, and could cause ambiguity in certain scenarios.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-299">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-299">Mitigations</span></span>

<span data-ttu-id="fdd39-300">应移动 `FromSql` 调用以使其直接位于它们所应用的 `DbSet` 上。</span><span class="sxs-lookup"><span data-stu-id="fdd39-300">`FromSql` invocations should be moved to be directly on the `DbSet` to which they apply.</span></span>

<a name="notrackingresolution"></a>

### <a name="no-tracking-queries-no-longer-perform-identity-resolution"></a><span data-ttu-id="fdd39-301">无跟踪查询不再执行标识解析</span><span class="sxs-lookup"><span data-stu-id="fdd39-301">No-tracking queries no longer perform identity resolution</span></span>

[<span data-ttu-id="fdd39-302">跟踪问题 #13518</span><span class="sxs-lookup"><span data-stu-id="fdd39-302">Tracking Issue #13518</span></span>](https://github.com/dotnet/efcore/issues/13518)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-303">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-303">Old behavior</span></span>

<span data-ttu-id="fdd39-304">在 EF Core 3.0 之前，将对具有给定类型和 ID 的实体的每个匹配项使用同一个实体实例。</span><span class="sxs-lookup"><span data-stu-id="fdd39-304">Before EF Core 3.0, the same entity instance would be used for every occurrence of an entity with a given type and ID.</span></span> <span data-ttu-id="fdd39-305">这与跟踪查询的行为匹配。</span><span class="sxs-lookup"><span data-stu-id="fdd39-305">This matches the behavior of tracking queries.</span></span> <span data-ttu-id="fdd39-306">例如下面的查询：</span><span class="sxs-lookup"><span data-stu-id="fdd39-306">For example, this query:</span></span>

```csharp
var results = context.Products.Include(e => e.Category).AsNoTracking().ToList();
```

<span data-ttu-id="fdd39-307">会为与给定类别关联的每个 `Product` 返回相同的 `Category` 实例。</span><span class="sxs-lookup"><span data-stu-id="fdd39-307">would return the same `Category` instance for each `Product` that is associated with the given category.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-308">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-308">New behavior</span></span>

<span data-ttu-id="fdd39-309">从 EF Core 3.0 开始，当在返回的图中的不同位置遇到具有给定类型和 ID 的实体时，将创建不同的实体实例。</span><span class="sxs-lookup"><span data-stu-id="fdd39-309">Starting with EF Core 3.0, different entity instances will be created when an entity with a given type and ID is encountered at different places in the returned graph.</span></span> <span data-ttu-id="fdd39-310">例如，上面的查询现在将为每个 `Product` 返回新的 `Category` 实例，即使两个产品与同一类别关联。</span><span class="sxs-lookup"><span data-stu-id="fdd39-310">For example, the query above will now return a new `Category` instance for each `Product` even when two products are associated with the same category.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-311">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-311">Why</span></span>

<span data-ttu-id="fdd39-312">标识解析（即确定实体与先前遇到的实体具有相同的类型和 ID）会增加额外的性能和内存开销。</span><span class="sxs-lookup"><span data-stu-id="fdd39-312">Identity resolution (that is, determining that an entity has the same type and ID as a previously encountered entity) adds additional performance and memory overhead.</span></span> <span data-ttu-id="fdd39-313">这通常会运行计数器，原因是最初使用无跟踪查询。</span><span class="sxs-lookup"><span data-stu-id="fdd39-313">This usually runs counter to why no-tracking queries are used in the first place.</span></span> <span data-ttu-id="fdd39-314">此外，尽管标识解析有时会很有用，但如果要对实体进行序列化并将其发送到客户端（这对于无跟踪查询很常见），则不需要这样做。</span><span class="sxs-lookup"><span data-stu-id="fdd39-314">Also, while identity resolution can sometimes be useful, it is not needed if the entities are to be serialized and sent to a client, which is common for no-tracking queries.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-315">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-315">Mitigations</span></span>

<span data-ttu-id="fdd39-316">如果需要标识解析，请使用跟踪查询。</span><span class="sxs-lookup"><span data-stu-id="fdd39-316">Use a tracking query if identity resolution is required.</span></span>

<a name="tkv"></a>

### <a name="temporary-key-values-are-no-longer-set-onto-entity-instances"></a><span data-ttu-id="fdd39-317">不再在实体实例上设置临时键值</span><span class="sxs-lookup"><span data-stu-id="fdd39-317">Temporary key values are no longer set onto entity instances</span></span>

[<span data-ttu-id="fdd39-318">跟踪问题 #12378</span><span class="sxs-lookup"><span data-stu-id="fdd39-318">Tracking Issue #12378</span></span>](https://github.com/dotnet/efcore/issues/12378)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-319">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-319">Old behavior</span></span>

<span data-ttu-id="fdd39-320">在 EF Core 3.0 之前，临时值已分配给所有键属性，这些属性稍后将具有数据库生成的实际值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-320">Before EF Core 3.0, temporary values were assigned to all key properties that would later have a real value generated by the database.</span></span>
<span data-ttu-id="fdd39-321">通常这些临时值是较大负数。</span><span class="sxs-lookup"><span data-stu-id="fdd39-321">Usually these temporary values were large negative numbers.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-322">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-322">New behavior</span></span>

<span data-ttu-id="fdd39-323">从 3.0 开始，EF Core 将临时键值存储为实体跟踪信息的一部分，并保持键属性本身不变。</span><span class="sxs-lookup"><span data-stu-id="fdd39-323">Starting with 3.0, EF Core stores the temporary key value as part of the entity's tracking information, and leaves the key property itself unchanged.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-324">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-324">Why</span></span>

<span data-ttu-id="fdd39-325">此更改是为了防止当之前由某个 `DbContext` 实例跟踪的实体移动到另一个 `DbContext` 实例时，临时键值错误地变成永久值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-325">This change was made to prevent temporary key values from erroneously becoming permanent when an entity that has been previously tracked by some `DbContext` instance is moved to a different `DbContext` instance.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-326">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-326">Mitigations</span></span>

<span data-ttu-id="fdd39-327">如果主键是存储生成的并且属于 `Added` 状态的实体，则将主键值分配到外键以在实体之间形成关联的应用可能会依赖于旧行为。</span><span class="sxs-lookup"><span data-stu-id="fdd39-327">Applications that assign primary key values onto foreign keys to form associations between entities may depend on the old behavior if the primary keys are store-generated and belong to entities in the `Added` state.</span></span>
<span data-ttu-id="fdd39-328">可通过以下方式避免：</span><span class="sxs-lookup"><span data-stu-id="fdd39-328">This can be avoided by:</span></span>

* <span data-ttu-id="fdd39-329">不使用存储生成的密钥。</span><span class="sxs-lookup"><span data-stu-id="fdd39-329">Not using store-generated keys.</span></span>
* <span data-ttu-id="fdd39-330">设置导航属性以形成关系，而不是设置外键值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-330">Setting navigation properties to form relationships instead of setting foreign key values.</span></span>
* <span data-ttu-id="fdd39-331">从实体的跟踪信息中获取实际的临时键值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-331">Obtain the actual temporary key values from the entity's tracking information.</span></span>
<span data-ttu-id="fdd39-332">例如，即使 `blog.Id` 本身尚未设置，`context.Entry(blog).Property(e => e.Id).CurrentValue` 也将返回临时值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-332">For example, `context.Entry(blog).Property(e => e.Id).CurrentValue` will return the temporary value even though `blog.Id` itself hasn't been set.</span></span>

<a name="dc"></a>

### <a name="detectchanges-honors-store-generated-key-values"></a><span data-ttu-id="fdd39-333">DetectChanges 遵循存储生成的键值</span><span class="sxs-lookup"><span data-stu-id="fdd39-333">DetectChanges honors store-generated key values</span></span>

[<span data-ttu-id="fdd39-334">跟踪问题 #14616</span><span class="sxs-lookup"><span data-stu-id="fdd39-334">Tracking Issue #14616</span></span>](https://github.com/dotnet/efcore/issues/14616)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-335">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-335">Old behavior</span></span>

<span data-ttu-id="fdd39-336">在 EF Core 3.0 之前，`DetectChanges` 找到的未跟踪实体将在 `Added` 状态中被跟踪，并在调用 `SaveChanges` 时作为新行插入。</span><span class="sxs-lookup"><span data-stu-id="fdd39-336">Before EF Core 3.0, an untracked entity found by `DetectChanges` would be tracked in the `Added` state and inserted as a new row when `SaveChanges` is called.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-337">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-337">New behavior</span></span>

<span data-ttu-id="fdd39-338">从 EF Core 3.0 开始，如果实体使用生成的键值并设置了某个键值，则将在 `Modified` 状态下跟踪实体。</span><span class="sxs-lookup"><span data-stu-id="fdd39-338">Starting with EF Core 3.0, if an entity is using generated key values and some key value is set, then the entity will be tracked in the `Modified` state.</span></span>
<span data-ttu-id="fdd39-339">这意味着假定存在实体的行，并且在调用 `SaveChanges` 时将更新该行。</span><span class="sxs-lookup"><span data-stu-id="fdd39-339">This means that a row for the entity is assumed to exist and it will be updated when `SaveChanges` is called.</span></span>
<span data-ttu-id="fdd39-340">如果未设置键值，或者实体类型未使用生成的键，则新实体仍将像先前版本一样被作为 `Added` 跟踪。</span><span class="sxs-lookup"><span data-stu-id="fdd39-340">If the key value isn't set, or if the entity type isn't using generated keys, then the new entity will still be tracked as `Added` as in previous versions.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-341">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-341">Why</span></span>

<span data-ttu-id="fdd39-342">进行此更改是为了在使用存储生成的键时更轻松、更一致地使用断开连接的实体图。</span><span class="sxs-lookup"><span data-stu-id="fdd39-342">This change was made to make it easier and more consistent to work with disconnected entity graphs while using store-generated keys.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-343">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-343">Mitigations</span></span>

<span data-ttu-id="fdd39-344">如果将实体类型配置为使用生成的键，但为新实例显式设置了键值，则此更改可能会中断应用程序。</span><span class="sxs-lookup"><span data-stu-id="fdd39-344">This change can break an application if an entity type is configured to use generated keys but key values are explicitly set for new instances.</span></span>
<span data-ttu-id="fdd39-345">解决方案是显式配置键属性，使其不使用生成的值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-345">The fix is to explicitly configure the key properties to not use generated values.</span></span>
<span data-ttu-id="fdd39-346">例如，使用 Fluent API：</span><span class="sxs-lookup"><span data-stu-id="fdd39-346">For example, with the fluent API:</span></span>

```csharp
modelBuilder
    .Entity<Blog>()
    .Property(e => e.Id)
    .ValueGeneratedNever();
```

<span data-ttu-id="fdd39-347">或使用数据注释：</span><span class="sxs-lookup"><span data-stu-id="fdd39-347">Or with data annotations:</span></span>

```csharp
[DatabaseGenerated(DatabaseGeneratedOption.None)]
public string Id { get; set; }
```

<a name="cascade"></a>

### <a name="cascade-deletions-now-happen-immediately-by-default"></a><span data-ttu-id="fdd39-348">默认情况下，现在会立即发生级联删除</span><span class="sxs-lookup"><span data-stu-id="fdd39-348">Cascade deletions now happen immediately by default</span></span>

[<span data-ttu-id="fdd39-349">跟踪问题 #10114</span><span class="sxs-lookup"><span data-stu-id="fdd39-349">Tracking Issue #10114</span></span>](https://github.com/dotnet/efcore/issues/10114)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-350">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-350">Old behavior</span></span>

<span data-ttu-id="fdd39-351">在 3.0 之前，直到调用 SaveChanges 时，EF Core 才会应用级联操作（删除所需主体时或者在切断与所需主体的关系时删除依赖实体）。</span><span class="sxs-lookup"><span data-stu-id="fdd39-351">Before 3.0, EF Core applied cascading actions (deleting dependent entities when a required principal is deleted or when the relationship to a required principal is severed) did not happen until SaveChanges was called.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-352">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-352">New behavior</span></span>

<span data-ttu-id="fdd39-353">从 3.0 开始，一旦检测到触发条件，EF Core 就会应用级联操作。</span><span class="sxs-lookup"><span data-stu-id="fdd39-353">Starting with 3.0, EF Core applies cascading actions as soon as the triggering condition is detected.</span></span>
<span data-ttu-id="fdd39-354">例如，调用 `context.Remove()` 来删除主体实体将导致所有跟踪的相关必需依赖项也立即设置为 `Deleted`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-354">For example, calling `context.Remove()` to delete a principal entity will result in all tracked related required dependents also being set to `Deleted` immediately.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-355">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-355">Why</span></span>

<span data-ttu-id="fdd39-356">此更改是为了改善数据绑定和审核方案的体验；在相关体验中，有必要了解在调用 `SaveChanges` 之前会删除哪些实体。</span><span class="sxs-lookup"><span data-stu-id="fdd39-356">This change was made to improve the experience for data binding and auditing scenarios where it is important to understand which entities will be deleted _before_ `SaveChanges` is called.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-357">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-357">Mitigations</span></span>

<span data-ttu-id="fdd39-358">可以通过 `context.ChangeTracker` 上的设置还原以前的行为。</span><span class="sxs-lookup"><span data-stu-id="fdd39-358">The previous behavior can be restored through settings on `context.ChangeTracker`.</span></span>
<span data-ttu-id="fdd39-359">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-359">For example:</span></span>

```csharp
context.ChangeTracker.CascadeDeleteTiming = CascadeTiming.OnSaveChanges;
context.ChangeTracker.DeleteOrphansTiming = CascadeTiming.OnSaveChanges;
```

<a name="eager-loading-single-query"></a>

### <a name="eager-loading-of-related-entities-now-happens-in-a-single-query"></a><span data-ttu-id="fdd39-360">单个查询中现在开始预先加载相关实体</span><span class="sxs-lookup"><span data-stu-id="fdd39-360">Eager loading of related entities now happens in a single query</span></span>

[<span data-ttu-id="fdd39-361">跟踪问题 #18022</span><span class="sxs-lookup"><span data-stu-id="fdd39-361">Tracking issue #18022</span></span>](https://github.com/dotnet/efcore/issues/18022)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-362">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-362">Old behavior</span></span>

<span data-ttu-id="fdd39-363">在 3.0 之前，通过 `Include` 运算符预先加载集合导航会导致在关系数据库上生成多个查询，每个相关实体类型对应一个查询。</span><span class="sxs-lookup"><span data-stu-id="fdd39-363">Before 3.0, eagerly loading collection navigations via `Include` operators caused multiple queries to be generated on relational database, one for each related entity type.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-364">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-364">New behavior</span></span>

<span data-ttu-id="fdd39-365">从 3.0 开始，EF Core 会在关系数据库上使用 JOIN 生成单个查询。</span><span class="sxs-lookup"><span data-stu-id="fdd39-365">Starting with 3.0, EF Core generates a single query with JOINs on relational databases.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-366">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-366">Why</span></span>

<span data-ttu-id="fdd39-367">以发出多个查询的方式实现单个 LINQ 查询会导致出现许多问题，包括由于需要执行多次数据库往返而引起的性能不佳问题，以及每个查询都可能会观察到不同的数据库状态的数据不一致问题。</span><span class="sxs-lookup"><span data-stu-id="fdd39-367">Issuing multiple queries to implement a single LINQ query caused numerous issues, including negative performance as multiple database roundtrips were necessary, and data coherency issues as each query could observe a different state of the database.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-368">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-368">Mitigations</span></span>

<span data-ttu-id="fdd39-369">尽管从技术上讲，这不是一项中断性变更，但当单个查询在集合导航中包含大量 `Include` 运算符时，这可能对应用程序性能产生相当大的影响。</span><span class="sxs-lookup"><span data-stu-id="fdd39-369">While technically this is not a breaking change, it could have a considerable effect on application performance when a single query contains a large number of `Include` operator on collection navigations.</span></span> <span data-ttu-id="fdd39-370">[请参阅此评论](https://github.com/dotnet/efcore/issues/18022#issuecomment-542397085)，了解详细信息，以及如何以更有效的方式重写查询。</span><span class="sxs-lookup"><span data-stu-id="fdd39-370">[See this comment](https://github.com/dotnet/efcore/issues/18022#issuecomment-542397085) for more information and for rewriting queries in a more efficient way.</span></span>

**

<a name="deletebehavior"></a>

### <a name="deletebehaviorrestrict-has-cleaner-semantics"></a><span data-ttu-id="fdd39-371">DeleteBehavior.Restrict 具有更简洁的语义</span><span class="sxs-lookup"><span data-stu-id="fdd39-371">DeleteBehavior.Restrict has cleaner semantics</span></span>

[<span data-ttu-id="fdd39-372">跟踪问题 #12661</span><span class="sxs-lookup"><span data-stu-id="fdd39-372">Tracking Issue #12661</span></span>](https://github.com/dotnet/efcore/issues/12661)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-373">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-373">Old behavior</span></span>

<span data-ttu-id="fdd39-374">3\.0 之前，`DeleteBehavior.Restrict` 使用 `Restrict` 语义在数据库中创建外键，但也以不明显的方式更改了内部修复。</span><span class="sxs-lookup"><span data-stu-id="fdd39-374">Before 3.0, `DeleteBehavior.Restrict` created foreign keys in the database with `Restrict` semantics, but also changed internal fixup in a non-obvious way.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-375">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-375">New behavior</span></span>

<span data-ttu-id="fdd39-376">从 3.0 开始，`DeleteBehavior.Restrict` 确保使用 `Restrict` 语义创建外键--即无级联；在发生约束冲突时触发 - 但不会同时影响 EF 内部修复。</span><span class="sxs-lookup"><span data-stu-id="fdd39-376">Starting with 3.0, `DeleteBehavior.Restrict` ensures that foreign keys are created with `Restrict` semantics--that is, no cascades; throw on constraint violation--without also impacting EF internal fixup.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-377">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-377">Why</span></span>

<span data-ttu-id="fdd39-378">进行此更改是为了改进以直观方式使用 `DeleteBehavior` 的体验，且不产生意外副作用。</span><span class="sxs-lookup"><span data-stu-id="fdd39-378">This change was made to improve the experience for using `DeleteBehavior` in an intuitive manner, without unexpected side-effects.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-379">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-379">Mitigations</span></span>

<span data-ttu-id="fdd39-380">可使用 `DeleteBehavior.ClientNoAction` 还原以前的行为。</span><span class="sxs-lookup"><span data-stu-id="fdd39-380">The previous behavior can be restored by using `DeleteBehavior.ClientNoAction`.</span></span>

<a name="qt"></a>

### <a name="query-types-are-consolidated-with-entity-types"></a><span data-ttu-id="fdd39-381">查询类型与实体类型合并</span><span class="sxs-lookup"><span data-stu-id="fdd39-381">Query types are consolidated with entity types</span></span>

[<span data-ttu-id="fdd39-382">跟踪问题 #14194</span><span class="sxs-lookup"><span data-stu-id="fdd39-382">Tracking Issue #14194</span></span>](https://github.com/dotnet/efcore/issues/14194)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-383">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-383">Old behavior</span></span>

<span data-ttu-id="fdd39-384">在 EF Core 3.0 之前，[查询类型](xref:core/modeling/keyless-entity-types)是一种查询未以结构化方式定义主键的数据的方法。</span><span class="sxs-lookup"><span data-stu-id="fdd39-384">Before EF Core 3.0, [query types](xref:core/modeling/keyless-entity-types) were a means to query data that doesn't define a primary key in a structured way.</span></span>
<span data-ttu-id="fdd39-385">也就是说，查询类型用于映射没有键的实体类型（更可能来自视图，但也可能来自表），而当有可用的键时则使用常规实体类型（更可能来自表，但也可能来自视图）。</span><span class="sxs-lookup"><span data-stu-id="fdd39-385">That is, a query type was used for mapping entity types without keys (more likely from a view, but possibly from a table) while a regular entity type was used when a key was available (more likely from a table, but possibly from a view).</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-386">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-386">New behavior</span></span>

<span data-ttu-id="fdd39-387">现在，查询类型只是一个没有主键的实体类型。</span><span class="sxs-lookup"><span data-stu-id="fdd39-387">A query type now becomes just an entity type without a primary key.</span></span>
<span data-ttu-id="fdd39-388">无键实体类型与先前版本中的查询类型具有相同的功能。</span><span class="sxs-lookup"><span data-stu-id="fdd39-388">Keyless entity types have the same functionality as query types in previous versions.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-389">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-389">Why</span></span>

<span data-ttu-id="fdd39-390">这样做是为了减少对查询类型用途的混淆。</span><span class="sxs-lookup"><span data-stu-id="fdd39-390">This change was made to reduce the confusion around the purpose of query types.</span></span>
<span data-ttu-id="fdd39-391">具体来说，它们是无键实体类型，因此本质上是只读的，但是不应该仅仅因为实体类型需要是只读的就使用它们。</span><span class="sxs-lookup"><span data-stu-id="fdd39-391">Specifically, they are keyless entity types and they are inherently read-only because of this, but they should not be used just because an entity type needs to be read-only.</span></span>
<span data-ttu-id="fdd39-392">同样，它们通常映射到视图，但这只是因为视图通常不定义键。</span><span class="sxs-lookup"><span data-stu-id="fdd39-392">Likewise, they are often mapped to views, but this is only because views often don't define keys.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-393">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-393">Mitigations</span></span>

<span data-ttu-id="fdd39-394">API 的以下部分现已过时：</span><span class="sxs-lookup"><span data-stu-id="fdd39-394">The following parts of the API are now obsolete:</span></span>

* <span data-ttu-id="fdd39-395">**`ModelBuilder.Query<>()`** - 需要调用 `ModelBuilder.Entity<>().HasNoKey()` 来将实体类型标记为没有键。</span><span class="sxs-lookup"><span data-stu-id="fdd39-395">**`ModelBuilder.Query<>()`** - Instead `ModelBuilder.Entity<>().HasNoKey()` needs to be called to mark an entity type as having no keys.</span></span>
<span data-ttu-id="fdd39-396">这仍然不会按约定配置，以避免在需要主键但是与约定不匹配时发生配置错误。</span><span class="sxs-lookup"><span data-stu-id="fdd39-396">This would still not be configured by convention to avoid misconfiguration when a primary key is expected, but doesn't match the convention.</span></span>
* <span data-ttu-id="fdd39-397">**`DbQuery<>`** - 应使用 `DbSet<>`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-397">**`DbQuery<>`** - Instead `DbSet<>` should be used.</span></span>
* <span data-ttu-id="fdd39-398">**`DbContext.Query<>()`** - 应使用 `DbContext.Set<>()`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-398">**`DbContext.Query<>()`** - Instead `DbContext.Set<>()` should be used.</span></span>
* <span data-ttu-id="fdd39-399">**`IQueryTypeConfiguration<TQuery>`** - 应使用 `IEntityTypeConfiguration<TEntity>`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-399">**`IQueryTypeConfiguration<TQuery>`** - Instead `IEntityTypeConfiguration<TEntity>` should be used.</span></span>

> [!NOTE]
> <span data-ttu-id="fdd39-400">由于查询所有属性设置为 `null` 的无键实体时存在 [3.x 中的问题](https://github.com/dotnet/efcore/issues/19537)，将返回 `null` 而不是实体，如果此问题适用于你的方案，则还要添加逻辑来处理结果中的 `null`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-400">Due to [an issue in 3.x](https://github.com/dotnet/efcore/issues/19537) when querying keyless entities that have all properties set to `null` a `null` will be returned instead of an entity, if this issue is applicable to your scenario also add logic to handle `null` in results.</span></span>

<a name="config"></a>

### <a name="configuration-api-for-owned-type-relationships-has-changed"></a><span data-ttu-id="fdd39-401">从属类型关系的配置 API 已更改</span><span class="sxs-lookup"><span data-stu-id="fdd39-401">Configuration API for owned type relationships has changed</span></span>

<span data-ttu-id="fdd39-402">[跟踪问题 #12444](https://github.com/dotnet/efcore/issues/12444)
[跟踪问题 #9148](https://github.com/dotnet/efcore/issues/9148)
[跟踪问题 #14153](https://github.com/dotnet/efcore/issues/14153)</span><span class="sxs-lookup"><span data-stu-id="fdd39-402">[Tracking Issue #12444](https://github.com/dotnet/efcore/issues/12444)
[Tracking Issue #9148](https://github.com/dotnet/efcore/issues/9148)
[Tracking Issue #14153](https://github.com/dotnet/efcore/issues/14153)</span></span>

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-403">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-403">Old behavior</span></span>

<span data-ttu-id="fdd39-404">EF Core 3.0 之前，会在 `OwnsOne` 或 `OwnsMany` 调用之后直接执行所拥有关系的配置。</span><span class="sxs-lookup"><span data-stu-id="fdd39-404">Before EF Core 3.0, configuration of the owned relationship was performed directly after the `OwnsOne` or `OwnsMany` call.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-405">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-405">New behavior</span></span>

<span data-ttu-id="fdd39-406">从 EF Core 3.0 开始，Fluent API 会使用 `WithOwner()` 为所有者配置导航属性。</span><span class="sxs-lookup"><span data-stu-id="fdd39-406">Starting with EF Core 3.0, there is now fluent API to configure a navigation property to the owner using `WithOwner()`.</span></span>
<span data-ttu-id="fdd39-407">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-407">For example:</span></span>

```csharp
modelBuilder.Entity<Order>.OwnsOne(e => e.Details).WithOwner(e => e.Order);
```

<span data-ttu-id="fdd39-408">与所有者之间关系相关的配置现会在 `WithOwner()` 之后关联起来，就像配置其他关系一样。</span><span class="sxs-lookup"><span data-stu-id="fdd39-408">The configuration related to the relationship between owner and owned should now be chained after `WithOwner()` similarly to how other relationships are configured.</span></span>
<span data-ttu-id="fdd39-409">但从属类型本身的配置仍会在 `OwnsOne()/OwnsMany()` 之后关联。</span><span class="sxs-lookup"><span data-stu-id="fdd39-409">While the configuration for the owned type itself would still be chained after `OwnsOne()/OwnsMany()`.</span></span>
<span data-ttu-id="fdd39-410">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-410">For example:</span></span>

```csharp
modelBuilder.Entity<Order>.OwnsOne(e => e.Details, eb =>
    {
        eb.WithOwner()
            .HasForeignKey(e => e.AlternateId)
            .HasConstraintName("FK_OrderDetails");

        eb.ToTable("OrderDetails");
        eb.HasKey(e => e.AlternateId);
        eb.HasIndex(e => e.Id);

        eb.HasOne(e => e.Customer).WithOne();

        eb.HasData(
            new OrderDetails
            {
                AlternateId = 1,
                Id = -1
            });
    });
```

<span data-ttu-id="fdd39-411">此外，使用固有类型目标调用 `Entity()``HasOne()` 或 `Set()` 现将引发异常。</span><span class="sxs-lookup"><span data-stu-id="fdd39-411">Additionally calling `Entity()`, `HasOne()`, or `Set()` with an owned type target will now throw an exception.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-412">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-412">Why</span></span>

<span data-ttu-id="fdd39-413">此更改是为了更清晰的区分从属类型本身的配置和与从属类型的 _关系_ 的配置。</span><span class="sxs-lookup"><span data-stu-id="fdd39-413">This change was made to create a cleaner separation between configuring the owned type itself and the _relationship to_ the owned type.</span></span>
<span data-ttu-id="fdd39-414">这反过来消除了诸如 `HasForeignKey` 之类的方法的模糊性和混淆。</span><span class="sxs-lookup"><span data-stu-id="fdd39-414">This in turn removes ambiguity and confusion around methods like `HasForeignKey`.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-415">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-415">Mitigations</span></span>

<span data-ttu-id="fdd39-416">更改从属类型关系的配置以使用新的 API 曲面，如上例所示。</span><span class="sxs-lookup"><span data-stu-id="fdd39-416">Change configuration of owned type relationships to use the new API surface as shown in the example above.</span></span>

<a name="de"></a>

### <a name="dependent-entities-sharing-the-table-with-the-principal-are-now-optional"></a><span data-ttu-id="fdd39-417">与主体共享表的依赖实体现为可选项</span><span class="sxs-lookup"><span data-stu-id="fdd39-417">Dependent entities sharing the table with the principal are now optional</span></span>

[<span data-ttu-id="fdd39-418">跟踪问题 #9005</span><span class="sxs-lookup"><span data-stu-id="fdd39-418">Tracking Issue #9005</span></span>](https://github.com/dotnet/efcore/issues/9005)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-419">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-419">Old behavior</span></span>

<span data-ttu-id="fdd39-420">考虑下列模型：</span><span class="sxs-lookup"><span data-stu-id="fdd39-420">Consider the following model:</span></span>

```csharp
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public OrderDetails Details { get; set; }
}

public class OrderDetails
{
    public int Id { get; set; }
    public string ShippingAddress { get; set; }
}
```

<span data-ttu-id="fdd39-421">在 EF Core 3.0 之前，如果 `OrderDetails` 由 `Order` 拥有且显式映射到同一张表，则在添加新的 `Order` 时，始终需要 `OrderDetails` 实例。</span><span class="sxs-lookup"><span data-stu-id="fdd39-421">Before EF Core 3.0, if `OrderDetails` is owned by `Order` or explicitly mapped to the same table then an `OrderDetails` instance was always required when adding a new `Order`.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-422">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-422">New behavior</span></span>

<span data-ttu-id="fdd39-423">自 3.0 起，EF Core 允许添加 `Order` 而不添加 `OrderDetails`，并将主键之外的所有 `OrderDetails` 属性映射到不为 null 的列中。</span><span class="sxs-lookup"><span data-stu-id="fdd39-423">Starting with 3.0, EF Core allows to add an `Order` without an `OrderDetails` and maps all of the `OrderDetails` properties except the primary key to nullable columns.</span></span>
<span data-ttu-id="fdd39-424">查询时，如果其任意所需属性均没有值，或者它在主键之外没有任何必需属性且所有属性均为 `null`，则 EF Core 会将 `OrderDetails` 设置为 `null`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-424">When querying EF Core sets `OrderDetails` to `null` if any of its required properties doesn't have a value or if it has no required properties besides the primary key and all properties are `null`.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-425">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-425">Mitigations</span></span>

<span data-ttu-id="fdd39-426">如果你的模型具有依赖于所有可选列的表共享，但指向该共享的导航不应为 `null`，则应修改应用程序，使其处理导航为 `null` 的情况。</span><span class="sxs-lookup"><span data-stu-id="fdd39-426">If your model has a table sharing dependent with all optional columns, but the navigation pointing to it is not expected to be `null` then the application should be modified to handle cases when the navigation is `null`.</span></span> <span data-ttu-id="fdd39-427">如果此方法不可行，则应向实体类型添加一个必需属性，或者至少要有一个属性分配有非 `null` 值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-427">If this is not possible a required property should be added to the entity type or at least one property should have a non-`null` value assigned to it.</span></span>

<a name="aes"></a>

### <a name="all-entities-sharing-a-table-with-a-concurrency-token-column-have-to-map-it-to-a-property"></a><span data-ttu-id="fdd39-428">与并发标记列共享表的所有实体均必须将其映射到属性</span><span class="sxs-lookup"><span data-stu-id="fdd39-428">All entities sharing a table with a concurrency token column have to map it to a property</span></span>

[<span data-ttu-id="fdd39-429">跟踪问题 #14154</span><span class="sxs-lookup"><span data-stu-id="fdd39-429">Tracking Issue #14154</span></span>](https://github.com/dotnet/efcore/issues/14154)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-430">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-430">Old behavior</span></span>

<span data-ttu-id="fdd39-431">考虑下列模型：</span><span class="sxs-lookup"><span data-stu-id="fdd39-431">Consider the following model:</span></span>

```csharp
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public byte[] Version { get; set; }
    public OrderDetails Details { get; set; }
}

public class OrderDetails
{
    public int Id { get; set; }
    public string ShippingAddress { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Order>()
        .Property(o => o.Version).IsRowVersion().HasColumnName("Version");
}
```

<span data-ttu-id="fdd39-432">在 EF Core 3.0 之前，如果 `OrderDetails` 由 `Order` 拥有且显式映射到同一张表，则只更新 `OrderDetails` 时，将不更新客户端上的 `Version` 值且下次更新将失败。</span><span class="sxs-lookup"><span data-stu-id="fdd39-432">Before EF Core 3.0, if `OrderDetails` is owned by `Order` or explicitly mapped to the same table then updating just `OrderDetails` will not update `Version` value on client and the next update will fail.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-433">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-433">New behavior</span></span>

<span data-ttu-id="fdd39-434">自 3.0 起，如果 新的 `Version` 值拥有 `OrderDetails`则 EF Core 会将该值传播给 `Order`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-434">Starting with 3.0, EF Core propagates the new `Version` value to `Order` if it owns `OrderDetails`.</span></span> <span data-ttu-id="fdd39-435">否则，会在模型验证期间引发异常。</span><span class="sxs-lookup"><span data-stu-id="fdd39-435">Otherwise an exception is thrown during model validation.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-436">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-436">Why</span></span>

<span data-ttu-id="fdd39-437">进行此更改的目的是避免在仅更新映射到同一张表的其中一个实体时使用过时的并发标记值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-437">This change was made to avoid a stale concurrency token value when only one of the entities mapped to the same table is updated.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-438">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-438">Mitigations</span></span>

<span data-ttu-id="fdd39-439">共享表的所有实体都必须包含一个映射到并发标记列的属性。</span><span class="sxs-lookup"><span data-stu-id="fdd39-439">All entities sharing the table have to include a property that is mapped to the concurrency token column.</span></span> <span data-ttu-id="fdd39-440">可在影子状态中创建一个：</span><span class="sxs-lookup"><span data-stu-id="fdd39-440">It's possible the create one in shadow-state:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<OrderDetails>()
        .Property<byte[]>("Version").IsRowVersion().HasColumnName("Version");
}
```

<a name="owned-query"></a>

### <a name="owned-entities-cannot-be-queried-without-the-owner-using-a-tracking-query"></a><span data-ttu-id="fdd39-441">如果没有所有者，则无法使用跟踪查询来查询从属实体</span><span class="sxs-lookup"><span data-stu-id="fdd39-441">Owned entities cannot be queried without the owner using a tracking query</span></span>

[<span data-ttu-id="fdd39-442">跟踪问题 #18876</span><span class="sxs-lookup"><span data-stu-id="fdd39-442">Tracking Issue #18876</span></span>](https://github.com/dotnet/efcore/issues/18876)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-443">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-443">Old behavior</span></span>

<span data-ttu-id="fdd39-444">在 EF Core 3.0 之前，可像任何其他导航一样查询从属实体。</span><span class="sxs-lookup"><span data-stu-id="fdd39-444">Before EF Core 3.0, the owned entities could be queried as any other navigation.</span></span>

```csharp
context.People.Select(p => p.Address);
```

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-445">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-445">New behavior</span></span>

<span data-ttu-id="fdd39-446">自版本 3.0 起，如果跟踪查询在没有所有者的情况下投射一个从属实体，EF Core 将引发异常。</span><span class="sxs-lookup"><span data-stu-id="fdd39-446">Starting with 3.0, EF Core will throw if a tracking query projects an owned entity without the owner.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-447">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-447">Why</span></span>

<span data-ttu-id="fdd39-448">如果没有所有者，则无法操作从属实体，因此在绝大多数情况下，不该使用此方式查询它们。</span><span class="sxs-lookup"><span data-stu-id="fdd39-448">Owned entities cannot be manipulated without the owner, so in the vast majority of cases querying them in this way is an error.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-449">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-449">Mitigations</span></span>

<span data-ttu-id="fdd39-450">如果应跟踪从属实体，以便之后以任何方式进行修改，则应在查询中包含所有者。</span><span class="sxs-lookup"><span data-stu-id="fdd39-450">If the owned entity should be tracked to be modified in any way later then the owner should be included in the query.</span></span>

<span data-ttu-id="fdd39-451">否则，请添加一个 `AsNoTracking()` 调用：</span><span class="sxs-lookup"><span data-stu-id="fdd39-451">Otherwise add an `AsNoTracking()` call:</span></span>

```csharp
context.People.Select(p => p.Address).AsNoTracking();
```

<a name="ip"></a>

### <a name="inherited-properties-from-unmapped-types-are-now-mapped-to-a-single-column-for-all-derived-types"></a><span data-ttu-id="fdd39-452">对于所有派生的类型而言，从未映射的类型继承的属性现在会映射到一个列中</span><span class="sxs-lookup"><span data-stu-id="fdd39-452">Inherited properties from unmapped types are now mapped to a single column for all derived types</span></span>

[<span data-ttu-id="fdd39-453">跟踪问题 #13998</span><span class="sxs-lookup"><span data-stu-id="fdd39-453">Tracking Issue #13998</span></span>](https://github.com/dotnet/efcore/issues/13998)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-454">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-454">Old behavior</span></span>

<span data-ttu-id="fdd39-455">考虑下列模型：</span><span class="sxs-lookup"><span data-stu-id="fdd39-455">Consider the following model:</span></span>

```csharp
public abstract class EntityBase
{
    public int Id { get; set; }
}

public abstract class OrderBase : EntityBase
{
    public int ShippingAddress { get; set; }
}

public class BulkOrder : OrderBase
{
}

public class Order : OrderBase
{
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Ignore<OrderBase>();
    modelBuilder.Entity<EntityBase>();
    modelBuilder.Entity<BulkOrder>();
    modelBuilder.Entity<Order>();
}
```

<span data-ttu-id="fdd39-456">在 EF Core 3.0 之前，`ShippingAddress` 属性会为 `BulkOrder` 和 `Order` 默认映射到单独的列中。</span><span class="sxs-lookup"><span data-stu-id="fdd39-456">Before EF Core 3.0, the `ShippingAddress` property would be mapped to separate columns for `BulkOrder` and `Order` by default.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-457">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-457">New behavior</span></span>

<span data-ttu-id="fdd39-458">自 3.0 起，EF Core只会为 `ShippingAddress` 创建一个列。</span><span class="sxs-lookup"><span data-stu-id="fdd39-458">Starting with 3.0, EF Core only creates one column for `ShippingAddress`.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-459">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-459">Why</span></span>

<span data-ttu-id="fdd39-460">旧行为不是预期行为。</span><span class="sxs-lookup"><span data-stu-id="fdd39-460">The old behavoir was unexpected.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-461">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-461">Mitigations</span></span>

<span data-ttu-id="fdd39-462">属性仍可显式映射到所派生的类型上的单独的列中：</span><span class="sxs-lookup"><span data-stu-id="fdd39-462">The property can still be explicitly mapped to separate column on the derived types:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Ignore<OrderBase>();
    modelBuilder.Entity<EntityBase>();
    modelBuilder.Entity<BulkOrder>()
        .Property(o => o.ShippingAddress).HasColumnName("BulkShippingAddress");
    modelBuilder.Entity<Order>()
        .Property(o => o.ShippingAddress).HasColumnName("ShippingAddress");
}
```

<a name="fkp"></a>

### <a name="the-foreign-key-property-convention-no-longer-matches-same-name-as-the-principal-property"></a><span data-ttu-id="fdd39-463">外键属性约定不再匹配与主体属性相同的名称</span><span class="sxs-lookup"><span data-stu-id="fdd39-463">The foreign key property convention no longer matches same name as the principal property</span></span>

[<span data-ttu-id="fdd39-464">跟踪问题 #13274</span><span class="sxs-lookup"><span data-stu-id="fdd39-464">Tracking Issue #13274</span></span>](https://github.com/dotnet/efcore/issues/13274)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-465">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-465">Old behavior</span></span>

<span data-ttu-id="fdd39-466">考虑下列模型：</span><span class="sxs-lookup"><span data-stu-id="fdd39-466">Consider the following model:</span></span>

```csharp
public class Customer
{
    public int CustomerId { get; set; }
    public ICollection<Order> Orders { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
}
```

<span data-ttu-id="fdd39-467">在 EF Core 3.0 之前，`CustomerId` 属性将按约定用于外键。</span><span class="sxs-lookup"><span data-stu-id="fdd39-467">Before EF Core 3.0, the `CustomerId` property would be used for the foreign key by convention.</span></span>
<span data-ttu-id="fdd39-468">但是，如果 `Order` 是从属类型，那么这也会使 `CustomerId` 成为主键，这通常不是预期结果。</span><span class="sxs-lookup"><span data-stu-id="fdd39-468">However, if `Order` is an owned type, then this would also make `CustomerId` the primary key and this isn't usually the expectation.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-469">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-469">New behavior</span></span>

<span data-ttu-id="fdd39-470">自 3.0 起，如果外键的主体属性名称相同，EF Core 不会尝试通过转换来为外键使用属性。</span><span class="sxs-lookup"><span data-stu-id="fdd39-470">Starting with 3.0, EF Core doesn't try to use properties for foreign keys by convention if they have the same name as the principal property.</span></span>
<span data-ttu-id="fdd39-471">与主体属性名称关联的主体类型名称和与主体属性名称模式关联的导航名称仍然相匹配。</span><span class="sxs-lookup"><span data-stu-id="fdd39-471">Principal type name concatenated with principal property name, and navigation name concatenated with principal property name patterns are still matched.</span></span>
<span data-ttu-id="fdd39-472">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-472">For example:</span></span>

```csharp
public class Customer
{
    public int Id { get; set; }
    public ICollection<Order> Orders { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
}
```

```csharp
public class Customer
{
    public int Id { get; set; }
    public ICollection<Order> Orders { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public int BuyerId { get; set; }
    public Customer Buyer { get; set; }
}
```

#### <a name="why"></a><span data-ttu-id="fdd39-473">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-473">Why</span></span>

<span data-ttu-id="fdd39-474">此更改是为了避免错误地在从属类型上定义主键属性。</span><span class="sxs-lookup"><span data-stu-id="fdd39-474">This change was made to avoid erroneously defining a primary key property on the owned type.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-475">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-475">Mitigations</span></span>

<span data-ttu-id="fdd39-476">如果该属性将成为外键，即为主键的一部分，则显式配置它。</span><span class="sxs-lookup"><span data-stu-id="fdd39-476">If the property was intended to be the foreign key, and hence part of the primary key, then explicitly configure it as such.</span></span>

<a name="dbc"></a>

### <a name="database-connection-is-now-closed-if-not-used-anymore-before-the-transactionscope-has-been-completed"></a><span data-ttu-id="fdd39-477">现在，如果在 TransactionScope 完成前不再使用数据库连接，则该连接会关闭</span><span class="sxs-lookup"><span data-stu-id="fdd39-477">Database connection is now closed if not used anymore before the TransactionScope has been completed</span></span>

[<span data-ttu-id="fdd39-478">跟踪问题 #14218</span><span class="sxs-lookup"><span data-stu-id="fdd39-478">Tracking Issue #14218</span></span>](https://github.com/dotnet/efcore/issues/14218)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-479">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-479">Old behavior</span></span>

<span data-ttu-id="fdd39-480">在 EF Core 3.0 之前，如果上下文打开了 `TransactionScope` 中的连接，则该连接在 `TransactionScope` 处于活动期间仍保持打开状态。</span><span class="sxs-lookup"><span data-stu-id="fdd39-480">Before EF Core 3.0, if the context opens the connection inside a `TransactionScope`, the connection remains open while the current `TransactionScope` is active.</span></span>

```csharp
using (new TransactionScope())
{
    using (AdventureWorks context = new AdventureWorks())
    {
        context.ProductCategories.Add(new ProductCategory());
        context.SaveChanges();

        // Old behavior: Connection is still open at this point

        var categories = context.ProductCategories().ToList();
    }
}
```

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-481">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-481">New behavior</span></span>

<span data-ttu-id="fdd39-482">自 3.0 起，一旦不再使用连接，EF Core 就会将其关闭。</span><span class="sxs-lookup"><span data-stu-id="fdd39-482">Starting with 3.0, EF Core closes the connection as soon as it's done using it.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-483">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-483">Why</span></span>

<span data-ttu-id="fdd39-484">此更改让你能够在同一 `TransactionScope` 中使用多个上下文。</span><span class="sxs-lookup"><span data-stu-id="fdd39-484">This change allows to use multiple contexts in the same `TransactionScope`.</span></span> <span data-ttu-id="fdd39-485">新行为也与 EF6 一致。</span><span class="sxs-lookup"><span data-stu-id="fdd39-485">The new behavior also matches EF6.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-486">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-486">Mitigations</span></span>

<span data-ttu-id="fdd39-487">如果连接需要保持打开状态，则显式调用 `OpenConnection()` 将确保 EF Core 不永久关闭此连接：</span><span class="sxs-lookup"><span data-stu-id="fdd39-487">If the connection needs to remain open explicit call to `OpenConnection()` will ensure that EF Core doesn't close it prematurely:</span></span>

```csharp
using (new TransactionScope())
{
    using (AdventureWorks context = new AdventureWorks())
    {
        context.Database.OpenConnection();
        context.ProductCategories.Add(new ProductCategory());
        context.SaveChanges();

        var categories = context.ProductCategories().ToList();
        context.Database.CloseConnection();
    }
}
```

<a name="each"></a>

### <a name="each-property-uses-independent-in-memory-integer-key-generation"></a><span data-ttu-id="fdd39-488">每个属性使用独立的内存中整数键生成</span><span class="sxs-lookup"><span data-stu-id="fdd39-488">Each property uses independent in-memory integer key generation</span></span>

[<span data-ttu-id="fdd39-489">跟踪问题 #6872</span><span class="sxs-lookup"><span data-stu-id="fdd39-489">Tracking Issue #6872</span></span>](https://github.com/dotnet/efcore/issues/6872)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-490">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-490">Old behavior</span></span>

<span data-ttu-id="fdd39-491">在 EF Core 3.0 之前，一个共享值生成器用于所有内存中的整数键属性。</span><span class="sxs-lookup"><span data-stu-id="fdd39-491">Before EF Core 3.0, one shared value generator was used for all in-memory integer key properties.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-492">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-492">New behavior</span></span>

<span data-ttu-id="fdd39-493">从 EF Core 3.0 开始，每个整数键属性在使用内存数据库时都会获取其自己的值生成器。</span><span class="sxs-lookup"><span data-stu-id="fdd39-493">Starting with EF Core 3.0, each integer key property gets its own value generator when using the in-memory database.</span></span>
<span data-ttu-id="fdd39-494">此外，如果删除了数据库，则会为所有表重置键生成。</span><span class="sxs-lookup"><span data-stu-id="fdd39-494">Also, if the database is deleted, then key generation is reset for all tables.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-495">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-495">Why</span></span>

<span data-ttu-id="fdd39-496">此更改的目的是使内存中的键生成更接近于实际的数据库键生成，并改进在使用内存中的数据库时隔离测试的功能。</span><span class="sxs-lookup"><span data-stu-id="fdd39-496">This change was made to align in-memory key generation more closely to real database key generation and to improve the ability to isolate tests from each other when using the in-memory database.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-497">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-497">Mitigations</span></span>

<span data-ttu-id="fdd39-498">这可能会中断依赖于特定内存中键值的应用程序。</span><span class="sxs-lookup"><span data-stu-id="fdd39-498">This can break an application that is relying on specific in-memory key values to be set.</span></span>
<span data-ttu-id="fdd39-499">请考虑不依赖于特定键值，或者进行更新以匹配新行为。</span><span class="sxs-lookup"><span data-stu-id="fdd39-499">Consider instead not relying on specific key values, or updating to match the new behavior.</span></span>

### <a name="backing-fields-are-used-by-default"></a><span data-ttu-id="fdd39-500">默认情况下使用支持字段</span><span class="sxs-lookup"><span data-stu-id="fdd39-500">Backing fields are used by default</span></span>

[<span data-ttu-id="fdd39-501">跟踪问题 #12430</span><span class="sxs-lookup"><span data-stu-id="fdd39-501">Tracking Issue #12430</span></span>](https://github.com/dotnet/efcore/issues/12430)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-502">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-502">Old behavior</span></span>

<span data-ttu-id="fdd39-503">在 3.0 之前，即使已知属性的支持字段，EF Core 仍将默认使用属性 getter 和 setter 方法读取和写入属性值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-503">Before 3.0, even if the backing field for a property was known, EF Core would still by default read and write the property value using the property getter and setter methods.</span></span>
<span data-ttu-id="fdd39-504">例外情况是查询执行，如果已知，将直接设置支持字段。</span><span class="sxs-lookup"><span data-stu-id="fdd39-504">The exception to this was query execution, where the backing field would be set directly if known.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-505">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-505">New behavior</span></span>

<span data-ttu-id="fdd39-506">从 EF Core 3.0 开始，如果已知属性的支持字段，EF Core 将始终使用支持字段读取和写入该属性。</span><span class="sxs-lookup"><span data-stu-id="fdd39-506">Starting with EF Core 3.0, if the backing field for a property is known, then EF Core will always read and write that property using the backing field.</span></span>
<span data-ttu-id="fdd39-507">如果应用程序依赖于编码到 getter 或 setter 方法中的其他行为，则可能导致应用程序中断。</span><span class="sxs-lookup"><span data-stu-id="fdd39-507">This could cause an application break if the application is relying on additional behavior coded into the getter or setter methods.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-508">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-508">Why</span></span>

<span data-ttu-id="fdd39-509">此更改是为了防止 EF Core 在执行涉及实体的数据库操作时默认错误地触发业务逻辑。</span><span class="sxs-lookup"><span data-stu-id="fdd39-509">This change was made to prevent EF Core from erroneously triggering business logic by default when performing database operations involving the entities.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-510">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-510">Mitigations</span></span>

<span data-ttu-id="fdd39-511">可以通过在 `ModelBuilder` 上配置属性访问模式来恢复 3.0 之前的行为。</span><span class="sxs-lookup"><span data-stu-id="fdd39-511">The pre-3.0 behavior can be restored through configuration of the property access mode on `ModelBuilder`.</span></span>
<span data-ttu-id="fdd39-512">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-512">For example:</span></span>

```csharp
modelBuilder.UsePropertyAccessMode(PropertyAccessMode.PreferFieldDuringConstruction);
```

### <a name="throw-if-multiple-compatible-backing-fields-are-found"></a><span data-ttu-id="fdd39-513">如果找到多个兼容的支持字段，则引发</span><span class="sxs-lookup"><span data-stu-id="fdd39-513">Throw if multiple compatible backing fields are found</span></span>

[<span data-ttu-id="fdd39-514">跟踪问题 #12523</span><span class="sxs-lookup"><span data-stu-id="fdd39-514">Tracking Issue #12523</span></span>](https://github.com/dotnet/efcore/issues/12523)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-515">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-515">Old behavior</span></span>

<span data-ttu-id="fdd39-516">在 EF Core 3.0 之前，如果多个字段与查找属性的后备字段的规则匹配，则将基于某种优先顺序选择一个字段。</span><span class="sxs-lookup"><span data-stu-id="fdd39-516">Before EF Core 3.0, if multiple fields matched the rules for finding the backing field of a property, then one field would be chosen based on some precedence order.</span></span>
<span data-ttu-id="fdd39-517">这可能导致在不明确的情况下使用错误字段。</span><span class="sxs-lookup"><span data-stu-id="fdd39-517">This could cause the wrong field to be used in ambiguous cases.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-518">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-518">New behavior</span></span>

<span data-ttu-id="fdd39-519">从 EF Core 3.0 开始，如果多个字段与同一属性匹配，则引发异常。</span><span class="sxs-lookup"><span data-stu-id="fdd39-519">Starting with EF Core 3.0, if multiple fields are matched to the same property, then an exception is thrown.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-520">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-520">Why</span></span>

<span data-ttu-id="fdd39-521">此更改是为了避免在只有一个字段是正确的情况下无提示地使用另一个字段。</span><span class="sxs-lookup"><span data-stu-id="fdd39-521">This change was made to avoid silently using one field over another when only one can be correct.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-522">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-522">Mitigations</span></span>

<span data-ttu-id="fdd39-523">具有不明确的支持字段的属性必须具有显式指定的字段。</span><span class="sxs-lookup"><span data-stu-id="fdd39-523">Properties with ambiguous backing fields must have the field to use specified explicitly.</span></span>
<span data-ttu-id="fdd39-524">例如，使用 Fluent API：</span><span class="sxs-lookup"><span data-stu-id="fdd39-524">For example, using the fluent API:</span></span>

```csharp
modelBuilder
    .Entity<Blog>()
    .Property(e => e.Id)
    .HasField("_id");
```

### <a name="field-only-property-names-should-match-the-field-name"></a><span data-ttu-id="fdd39-525">“仅字段”属性名应与字段名匹配</span><span class="sxs-lookup"><span data-stu-id="fdd39-525">Field-only property names should match the field name</span></span>

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-526">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-526">Old behavior</span></span>

<span data-ttu-id="fdd39-527">在 EF Core 3.0 之前，可通过字符串值指定属性，如果在 .NET 类型上找不到具有该名称的属性，则 EF Core 将尝试使用约定规则将其与字段匹配。</span><span class="sxs-lookup"><span data-stu-id="fdd39-527">Before EF Core 3.0, a property could be specified by a string value and if no property with that name was found on the .NET type then EF Core would try to match it to a field using convention rules.</span></span>

```csharp
private class Blog
{
    private int _id;
    public string Name { get; set; }
}
```

```csharp
modelBuilder
    .Entity<Blog>()
    .Property("Id");
```

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-528">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-528">New behavior</span></span>

<span data-ttu-id="fdd39-529">从 EF Core 3.0 开始，“仅字段”属性必须与字段名完全匹配。</span><span class="sxs-lookup"><span data-stu-id="fdd39-529">Starting with EF Core 3.0, a field-only property must match the field name exactly.</span></span>

```csharp
modelBuilder
    .Entity<Blog>()
    .Property("_id");
```

#### <a name="why"></a><span data-ttu-id="fdd39-530">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-530">Why</span></span>

<span data-ttu-id="fdd39-531">进行此更改是为了避免对两个名称相似的属性使用相同的字段，这也使得仅字段属性的匹配规则与映射到 CLR 属性的属性相同。</span><span class="sxs-lookup"><span data-stu-id="fdd39-531">This change was made to avoid using the same field for two properties named similarly, it also makes the matching rules for field-only properties the same as for properties mapped to CLR properties.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-532">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-532">Mitigations</span></span>

<span data-ttu-id="fdd39-533">仅字段属性名必须与它们映射到的字段名相同。</span><span class="sxs-lookup"><span data-stu-id="fdd39-533">Field-only properties must be named the same as the field they are mapped to.</span></span>
<span data-ttu-id="fdd39-534">在 3.0 版之后的 EF Core 未来版本中，我们计划重新启用显式配置与属性名称不同的字段名称（请参阅问题 [15307](https://github.com/dotnet/efcore/issues/15307)）：</span><span class="sxs-lookup"><span data-stu-id="fdd39-534">In a future release of EF Core after 3.0, we plan to re-enable explicitly configuring a field name that is different from the property name (see issue [#15307](https://github.com/dotnet/efcore/issues/15307)):</span></span>

```csharp
modelBuilder
    .Entity<Blog>()
    .Property("Id")
    .HasField("_id");
```

<a name="adddbc"></a>

### <a name="adddbcontextadddbcontextpool-no-longer-call-addlogging-and-addmemorycache"></a><span data-ttu-id="fdd39-535">AddDbContext/AddDbContextPool 不再调用 AddLogging 和 AddMemoryCache</span><span class="sxs-lookup"><span data-stu-id="fdd39-535">AddDbContext/AddDbContextPool no longer call AddLogging and AddMemoryCache</span></span>

[<span data-ttu-id="fdd39-536">跟踪问题 #14756</span><span class="sxs-lookup"><span data-stu-id="fdd39-536">Tracking Issue #14756</span></span>](https://github.com/dotnet/efcore/issues/14756)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-537">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-537">Old behavior</span></span>

<span data-ttu-id="fdd39-538">在 EF Core 3.0 之前，调用 `AddDbContext` 或 `AddDbContextPool` 的操作也会通过调用 [AddLogging](/dotnet/api/microsoft.extensions.dependencyinjection.loggingservicecollectionextensions.addlogging) 和 [AddMemoryCache](/dotnet/api/microsoft.extensions.dependencyinjection.memorycacheservicecollectionextensions.addmemorycache) 向 DI 注册日志记录和内存缓存服务。</span><span class="sxs-lookup"><span data-stu-id="fdd39-538">Before EF Core 3.0, calling `AddDbContext` or `AddDbContextPool` would also register logging and memory caching services with DI through calls to [AddLogging](/dotnet/api/microsoft.extensions.dependencyinjection.loggingservicecollectionextensions.addlogging) and [AddMemoryCache](/dotnet/api/microsoft.extensions.dependencyinjection.memorycacheservicecollectionextensions.addmemorycache).</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-539">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-539">New behavior</span></span>

<span data-ttu-id="fdd39-540">从 EF Core 3.0 开始，`AddDbContext` 和 `AddDbContextPool` 将无法再在依赖注入 (DI) 中注册这些服务。</span><span class="sxs-lookup"><span data-stu-id="fdd39-540">Starting with EF Core 3.0, `AddDbContext` and `AddDbContextPool` will no longer register these services with Dependency Injection (DI).</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-541">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-541">Why</span></span>

<span data-ttu-id="fdd39-542">EF Core 3.0 不要求这些服务位于应用程序的 DI 容器中。</span><span class="sxs-lookup"><span data-stu-id="fdd39-542">EF Core 3.0 does not require that these services are in the application's DI container.</span></span> <span data-ttu-id="fdd39-543">但是，如果 `ILoggerFactory` 在应用程序的 DI 容器中注册，它仍然会由 EF Core 使用。</span><span class="sxs-lookup"><span data-stu-id="fdd39-543">However, if `ILoggerFactory` is registered in the application's DI container, then it will still be used by EF Core.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-544">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-544">Mitigations</span></span>

<span data-ttu-id="fdd39-545">如果应用程序需要这些服务，则使用 [AddLogging](/dotnet/api/microsoft.extensions.dependencyinjection.loggingservicecollectionextensions.addlogging) 或 [AddMemoryCache](/dotnet/api/microsoft.extensions.dependencyinjection.memorycacheservicecollectionextensions.addmemorycache) 将它们显式注册到 DI 容器中。</span><span class="sxs-lookup"><span data-stu-id="fdd39-545">If your application needs these services, then register them explicitly with the DI container using  [AddLogging](/dotnet/api/microsoft.extensions.dependencyinjection.loggingservicecollectionextensions.addlogging) or [AddMemoryCache](/dotnet/api/microsoft.extensions.dependencyinjection.memorycacheservicecollectionextensions.addmemorycache).</span></span>

### <a name="addentityframework-adds-imemorycache-with-a-size-limit"></a><span data-ttu-id="fdd39-546">AddEntityFramework\* 添加具有大小限制的 IMemoryCache</span><span class="sxs-lookup"><span data-stu-id="fdd39-546">AddEntityFramework\* adds IMemoryCache with a size limit</span></span>

[<span data-ttu-id="fdd39-547">跟踪问题 #12905</span><span class="sxs-lookup"><span data-stu-id="fdd39-547">Tracking Issue #12905</span></span>](https://github.com/dotnet/efcore/issues/12905)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-548">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-548">Old behavior</span></span>

<span data-ttu-id="fdd39-549">在 EF Core 3.0 之前，调用 `AddEntityFramework*` 方法还将向 DI 注册内存缓存服务，且没有大小限制。</span><span class="sxs-lookup"><span data-stu-id="fdd39-549">Before EF Core 3.0, calling `AddEntityFramework*` methods would also register memory caching services with DI without a size limit.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-550">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-550">New behavior</span></span>

<span data-ttu-id="fdd39-551">自 EF Core 3.0 起，`AddEntityFramework*` 注册的 IMemoryCache 服务具有大小限制。</span><span class="sxs-lookup"><span data-stu-id="fdd39-551">Starting with EF Core 3.0, `AddEntityFramework*` will register an IMemoryCache service with a size limit.</span></span> <span data-ttu-id="fdd39-552">如果随后添加的任何其他服务依赖于 IMemoryCache，则它们很快就会达到默认限制，从而导致异常或性能下降。</span><span class="sxs-lookup"><span data-stu-id="fdd39-552">If any other services added afterwards depend on IMemoryCache they can quickly reach the default limit causing exceptions or degraded performance.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-553">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-553">Why</span></span>

<span data-ttu-id="fdd39-554">如果查询缓存逻辑中存在 bug 或者查询是动态生成的，则无限制地使用 IMemoryCache 可能会导致内存使用量不受控制。</span><span class="sxs-lookup"><span data-stu-id="fdd39-554">Using IMemoryCache without a limit could result in uncontrolled memory usage if there is a bug in query caching logic or the queries are generated dynamically.</span></span> <span data-ttu-id="fdd39-555">设定默认限制可减少潜在的 DoS 攻击。</span><span class="sxs-lookup"><span data-stu-id="fdd39-555">Having a default limit mitigates a potential DoS attack.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-556">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-556">Mitigations</span></span>

<span data-ttu-id="fdd39-557">在大多数情况下，如果同时调用了 `AddDbContext` 或 `AddDbContextPool`，则无需调用 `AddEntityFramework*`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-557">In most cases calling `AddEntityFramework*` is not necessary if `AddDbContext` or `AddDbContextPool` is called as well.</span></span> <span data-ttu-id="fdd39-558">因此，最好的缓解措施是删除 `AddEntityFramework*` 调用。</span><span class="sxs-lookup"><span data-stu-id="fdd39-558">Therefore, the best mitigation is to remove the `AddEntityFramework*` call.</span></span>

<span data-ttu-id="fdd39-559">如果应用程序需要这些服务，则使用 [AddMemoryCache](/dotnet/api/microsoft.extensions.dependencyinjection.memorycacheservicecollectionextensions.addmemorycache) 预先向 DI 容器显式注册 IMemoryCache 实现。</span><span class="sxs-lookup"><span data-stu-id="fdd39-559">If your application needs these services, then register a IMemoryCache implementation explicitly with the DI container beforehand using [AddMemoryCache](/dotnet/api/microsoft.extensions.dependencyinjection.memorycacheservicecollectionextensions.addmemorycache).</span></span>

<a name="dbe"></a>

### <a name="dbcontextentry-now-performs-a-local-detectchanges"></a><span data-ttu-id="fdd39-560">DbContext.Entry 现在执行本地 DetectChanges</span><span class="sxs-lookup"><span data-stu-id="fdd39-560">DbContext.Entry now performs a local DetectChanges</span></span>

[<span data-ttu-id="fdd39-561">跟踪问题 #13552</span><span class="sxs-lookup"><span data-stu-id="fdd39-561">Tracking Issue #13552</span></span>](https://github.com/dotnet/efcore/issues/13552)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-562">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-562">Old behavior</span></span>

<span data-ttu-id="fdd39-563">在 EF Core 3.0 之前，调用 `DbContext.Entry` 将导致检测到所有被跟踪实体的更改。</span><span class="sxs-lookup"><span data-stu-id="fdd39-563">Before EF Core 3.0, calling `DbContext.Entry` would cause changes to be detected for all tracked entities.</span></span>
<span data-ttu-id="fdd39-564">这确保了 `EntityEntry` 中暴露的状态是最新的。</span><span class="sxs-lookup"><span data-stu-id="fdd39-564">This ensured that the state exposed in the `EntityEntry` was up-to-date.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-565">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-565">New behavior</span></span>

<span data-ttu-id="fdd39-566">从 EF Core 3.0 开始，调用 `DbContext.Entry` 现在只会尝试检测给定实体和与之相关的任何跟踪主体实体的更改。</span><span class="sxs-lookup"><span data-stu-id="fdd39-566">Starting with EF Core 3.0, calling `DbContext.Entry` will now only attempt to detect changes in the given entity and any tracked principal entities related to it.</span></span>
<span data-ttu-id="fdd39-567">这意味着可能无法通过调用此方法检测到其他位置的更改，这可能会影响应用程序状态。</span><span class="sxs-lookup"><span data-stu-id="fdd39-567">This means that changes elsewhere may not have been detected by calling this method, which could have implications on application state.</span></span>

<span data-ttu-id="fdd39-568">请注意，如果 `ChangeTracker.AutoDetectChangesEnabled` 设置为 `false`，则即使是本地更改检测也将被禁用。</span><span class="sxs-lookup"><span data-stu-id="fdd39-568">Note that if `ChangeTracker.AutoDetectChangesEnabled` is set to `false` then even this local change detection will be disabled.</span></span>

<span data-ttu-id="fdd39-569">导致更改检测的其他方法（例如 `ChangeTracker.Entries` 和 `SaveChanges`）仍然会导致所有被跟踪实体的完整 `DetectChanges`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-569">Other methods that cause change detection--for example `ChangeTracker.Entries` and `SaveChanges`--still cause a full `DetectChanges` of all tracked entities.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-570">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-570">Why</span></span>

<span data-ttu-id="fdd39-571">此更改是为了提高使用 `context.Entry` 的默认性能。</span><span class="sxs-lookup"><span data-stu-id="fdd39-571">This change was made to improve the default performance of using `context.Entry`.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-572">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-572">Mitigations</span></span>

<span data-ttu-id="fdd39-573">在调用 `Entry` 之前显式调用 `ChangeTracker.DetectChanges()` 以确保 3.0 之前的行为。</span><span class="sxs-lookup"><span data-stu-id="fdd39-573">Call `ChangeTracker.DetectChanges()` explicitly before calling `Entry` to ensure the pre-3.0 behavior.</span></span>

### <a name="string-and-byte-array-keys-are-not-client-generated-by-default"></a><span data-ttu-id="fdd39-574">默认情况下，字符串和字节数组键不是客户端生成的</span><span class="sxs-lookup"><span data-stu-id="fdd39-574">String and byte array keys are not client-generated by default</span></span>

[<span data-ttu-id="fdd39-575">跟踪问题 #14617</span><span class="sxs-lookup"><span data-stu-id="fdd39-575">Tracking Issue #14617</span></span>](https://github.com/dotnet/efcore/issues/14617)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-576">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-576">Old behavior</span></span>

<span data-ttu-id="fdd39-577">在 EF Core 3.0 之前，可以使用 `string` 和 `byte[]` 键属性，而不需要显式地设置非 null 值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-577">Before EF Core 3.0, `string` and `byte[]` key properties could be used without explicitly setting a non-null value.</span></span>
<span data-ttu-id="fdd39-578">在这种情况下，键值将在客户端上生成为 GUID，并序列化为 `byte[]` 的字节。</span><span class="sxs-lookup"><span data-stu-id="fdd39-578">In such a case, the key value would be generated on the client as a GUID, serialized to bytes for `byte[]`.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-579">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-579">New behavior</span></span>

<span data-ttu-id="fdd39-580">从 EF Core 3.0 开始，将引发异常，指示未设置任何键值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-580">Starting with EF Core 3.0 an exception will be thrown indicating that no key value has been set.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-581">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-581">Why</span></span>

<span data-ttu-id="fdd39-582">之所以进行此更改是因为客户端生成的 `string`/`byte[]` 值通常没有用，并且默认行为使得很难以通用方式推断生成的键值。</span><span class="sxs-lookup"><span data-stu-id="fdd39-582">This change was made because client-generated `string`/`byte[]` values generally aren't useful, and the default behavior made it hard to reason about generated key values in a common way.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-583">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-583">Mitigations</span></span>

<span data-ttu-id="fdd39-584">如果没有设置其他非 null 值，则可以通过显式指定键属性应使用生成的值来获得 3.0 之前的行为。</span><span class="sxs-lookup"><span data-stu-id="fdd39-584">The pre-3.0 behavior can be obtained by explicitly specifying that the key properties should use generated values if no other non-null value is set.</span></span>
<span data-ttu-id="fdd39-585">例如，使用 Fluent API：</span><span class="sxs-lookup"><span data-stu-id="fdd39-585">For example, with the fluent API:</span></span>

```csharp
modelBuilder
    .Entity<Blog>()
    .Property(e => e.Id)
    .ValueGeneratedOnAdd();
```

<span data-ttu-id="fdd39-586">或使用数据注释：</span><span class="sxs-lookup"><span data-stu-id="fdd39-586">Or with data annotations:</span></span>

```csharp
[DatabaseGenerated(DatabaseGeneratedOption.Identity)]
public string Id { get; set; }
```

<a name="ilf"></a>

### <a name="iloggerfactory-is-now-a-scoped-service"></a><span data-ttu-id="fdd39-587">ILoggerFactory 现在是一个在一定范围内有效的服务</span><span class="sxs-lookup"><span data-stu-id="fdd39-587">ILoggerFactory is now a scoped service</span></span>

[<span data-ttu-id="fdd39-588">跟踪问题 #14698</span><span class="sxs-lookup"><span data-stu-id="fdd39-588">Tracking Issue #14698</span></span>](https://github.com/dotnet/efcore/issues/14698)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-589">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-589">Old behavior</span></span>

<span data-ttu-id="fdd39-590">在 EF Core 3.0 之前，`ILoggerFactory` 被注册为单一实例服务。</span><span class="sxs-lookup"><span data-stu-id="fdd39-590">Before EF Core 3.0, `ILoggerFactory` was registered as a singleton service.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-591">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-591">New behavior</span></span>

<span data-ttu-id="fdd39-592">从 EF Core 3.0 开始，`ILoggerFactory` 现已注册为作用域。</span><span class="sxs-lookup"><span data-stu-id="fdd39-592">Starting with EF Core 3.0, `ILoggerFactory` is now registered as scoped.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-593">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-593">Why</span></span>

<span data-ttu-id="fdd39-594">此更改是为了允许记录器与 `DbContext` 实例关联，从而启用其他功能并删除一些反常行为，例如内部服务提供商爆炸式增长的情况。</span><span class="sxs-lookup"><span data-stu-id="fdd39-594">This change was made to allow association of a logger with a `DbContext` instance, which enables other functionality and removes some cases of pathological behavior such as an explosion of internal service providers.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-595">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-595">Mitigations</span></span>

<span data-ttu-id="fdd39-596">除非在 EF Core 内部服务提供商上注册和使用自定义服务，否则此更改不应影响应用程序代码。</span><span class="sxs-lookup"><span data-stu-id="fdd39-596">This change should not impact application code unless it is registering and using custom services on the EF Core internal service provider.</span></span>
<span data-ttu-id="fdd39-597">这并不常见。</span><span class="sxs-lookup"><span data-stu-id="fdd39-597">This isn't common.</span></span>
<span data-ttu-id="fdd39-598">在这些情况下，大多数事情仍然有效，但是需要更改依赖于 `ILoggerFactory` 的任何单一实例服务以便以不同的方式获取 `ILoggerFactory`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-598">In these cases, most things will still work, but any singleton service that was depending on `ILoggerFactory` will need to be changed to obtain the `ILoggerFactory` in a different way.</span></span>

<span data-ttu-id="fdd39-599">如果遇到此类情况，请在 [EF Core GitHub 问题跟踪程序](https://github.com/dotnet/efcore/issues)上提交一个问题，让我们知道你是如何使用 `ILoggerFactory` 的，以便我们更好地理解今后如何避免这种情况再次发生。</span><span class="sxs-lookup"><span data-stu-id="fdd39-599">If you run into situations like this, please file an issue at on the [EF Core GitHub issue tracker](https://github.com/dotnet/efcore/issues) to let us know how you are using `ILoggerFactory` such that we can better understand how not to break this again in the future.</span></span>

### <a name="lazy-loading-proxies-no-longer-assume-navigation-properties-are-fully-loaded"></a><span data-ttu-id="fdd39-600">延迟加载代理不再假定导航属性已完全加载</span><span class="sxs-lookup"><span data-stu-id="fdd39-600">Lazy-loading proxies no longer assume navigation properties are fully loaded</span></span>

[<span data-ttu-id="fdd39-601">跟踪问题 #12780</span><span class="sxs-lookup"><span data-stu-id="fdd39-601">Tracking Issue #12780</span></span>](https://github.com/dotnet/efcore/issues/12780)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-602">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-602">Old behavior</span></span>

<span data-ttu-id="fdd39-603">在 EF Core 3.0 之前，一旦 `DbContext` 被处置，就无法知道从该上下文获得的实体上的给定导航属性是否已完全加载。</span><span class="sxs-lookup"><span data-stu-id="fdd39-603">Before EF Core 3.0, once a `DbContext` was disposed there was no way of knowing if a given navigation property on an entity obtained from that context was fully loaded or not.</span></span>
<span data-ttu-id="fdd39-604">相反，如果导航有一个非 null 值，代理将假定加载一个引用导航，如果导航非空，则假定加载集合导航。</span><span class="sxs-lookup"><span data-stu-id="fdd39-604">Proxies would instead assume that a reference navigation is loaded if it has a non-null value, and that a collection navigation is loaded if it isn't empty.</span></span>
<span data-ttu-id="fdd39-605">在这些情况下，尝试延迟加载将是无效的。</span><span class="sxs-lookup"><span data-stu-id="fdd39-605">In these cases, attempting to lazy-load would be a no-op.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-606">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-606">New behavior</span></span>

<span data-ttu-id="fdd39-607">从 EF Core 3.0 开始，代理会跟踪是否加载了导航属性。</span><span class="sxs-lookup"><span data-stu-id="fdd39-607">Starting with EF Core 3.0, proxies keep track of whether or not a navigation property is loaded.</span></span>
<span data-ttu-id="fdd39-608">这意味着如果尝试访问在释放了上下文之后加载的导航属性，其结果始终是无操作，即使加载的导航为空或为 null。</span><span class="sxs-lookup"><span data-stu-id="fdd39-608">This means attempting to access a navigation property that is loaded after the context has been disposed will always be a no-op, even when the loaded navigation is empty or null.</span></span>
<span data-ttu-id="fdd39-609">相反，即使导航属性是非空集合，尝试访问未加载的导航属性也会引发异常。</span><span class="sxs-lookup"><span data-stu-id="fdd39-609">Conversely, attempting to access a navigation property that isn't loaded will throw an exception if the context is disposed even if the navigation property is a non-empty collection.</span></span>
<span data-ttu-id="fdd39-610">如果出现这种情况，则表示应用程序代码在无效时间尝试使用延迟加载，应将应用程序更改为不执行此操作。</span><span class="sxs-lookup"><span data-stu-id="fdd39-610">If this situation arises, it means the application code is attempting to use lazy-loading at an invalid time, and the application should be changed to not do this.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-611">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-611">Why</span></span>

<span data-ttu-id="fdd39-612">此更改是为了在尝试对已释放的 `DbContext` 实例进行延迟加载时使行为保持一致和正确。</span><span class="sxs-lookup"><span data-stu-id="fdd39-612">This change was made to make the behavior consistent and correct when attempting to lazy-load on a disposed `DbContext` instance.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-613">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-613">Mitigations</span></span>

<span data-ttu-id="fdd39-614">更新应用程序代码，以避免尝试对已释放的上下文进行延迟加载，或者将其配置为无操作，如异常消息中所述。</span><span class="sxs-lookup"><span data-stu-id="fdd39-614">Update application code to not attempt lazy-loading with a disposed context, or configure this to be a no-op as described in the exception message.</span></span>

### <a name="excessive-creation-of-internal-service-providers-is-now-an-error-by-default"></a><span data-ttu-id="fdd39-615">默认情况下，现在过度创建内部服务提供程序是一个错误</span><span class="sxs-lookup"><span data-stu-id="fdd39-615">Excessive creation of internal service providers is now an error by default</span></span>

[<span data-ttu-id="fdd39-616">跟踪问题 #10236</span><span class="sxs-lookup"><span data-stu-id="fdd39-616">Tracking Issue #10236</span></span>](https://github.com/dotnet/efcore/issues/10236)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-617">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-617">Old behavior</span></span>

<span data-ttu-id="fdd39-618">在 EF Core 3.0 之前，对于创建了大量内部服务提供程序的应用程序，将会记录一个警告。</span><span class="sxs-lookup"><span data-stu-id="fdd39-618">Before EF Core 3.0, a warning would be logged for an application creating a pathological number of internal service providers.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-619">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-619">New behavior</span></span>

<span data-ttu-id="fdd39-620">从 EF Core 3.0 开始，现在会考虑此警告，并引发错误和异常。</span><span class="sxs-lookup"><span data-stu-id="fdd39-620">Starting with EF Core 3.0, this warning is now considered and error and an exception is thrown.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-621">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-621">Why</span></span>

<span data-ttu-id="fdd39-622">此更改是为了通过更明显地暴露这个病态案例来驱动生成更好的应用程序代码。</span><span class="sxs-lookup"><span data-stu-id="fdd39-622">This change was made to drive better application code through exposing this pathological case more explicitly.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-623">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-623">Mitigations</span></span>

<span data-ttu-id="fdd39-624">遇到此错误时，最合适的操作是了解根本原因并停止创建如此多的内部服务提供程序。</span><span class="sxs-lookup"><span data-stu-id="fdd39-624">The most appropriate cause of action on encountering this error is to understand the root cause and stop creating so many internal service providers.</span></span>
<span data-ttu-id="fdd39-625">但是，可以通过 `DbContextOptionsBuilder` 上的配置将错误转换回警告（或忽略）。</span><span class="sxs-lookup"><span data-stu-id="fdd39-625">However, the error can be converted back to a warning (or ignored) via configuration on the `DbContextOptionsBuilder`.</span></span>
<span data-ttu-id="fdd39-626">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-626">For example:</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .ConfigureWarnings(w => w.Log(CoreEventId.ManyServiceProvidersCreatedWarning));
}
```

<a name="nbh"></a>

### <a name="new-behavior-for-hasonehasmany-called-with-a-single-string"></a><span data-ttu-id="fdd39-627">使用单个字符串调用 HasOne/HasMany 的新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-627">New behavior for HasOne/HasMany called with a single string</span></span>

[<span data-ttu-id="fdd39-628">跟踪问题 #9171</span><span class="sxs-lookup"><span data-stu-id="fdd39-628">Tracking Issue #9171</span></span>](https://github.com/dotnet/efcore/issues/9171)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-629">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-629">Old behavior</span></span>

<span data-ttu-id="fdd39-630">在 EF Core 3.0 之前，对通过单个字符串调用 `HasOne` 或 `HasMany` 的代码的解释令人困惑。</span><span class="sxs-lookup"><span data-stu-id="fdd39-630">Before EF Core 3.0, code calling `HasOne` or `HasMany` with a single string was interpreted in a confusing way.</span></span>
<span data-ttu-id="fdd39-631">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-631">For example:</span></span>

```csharp
modelBuilder.Entity<Samurai>().HasOne("Entrance").WithOne();
```

<span data-ttu-id="fdd39-632">该代码看起来是使用 `Entrance` 导航属性（可能是私有属性）将 `Samurai` 与某些其他实体类型关联起来的。</span><span class="sxs-lookup"><span data-stu-id="fdd39-632">The code looks like it is relating `Samurai` to some other entity type using the `Entrance` navigation property, which may be private.</span></span>

<span data-ttu-id="fdd39-633">实际上，此代码试图创建与某个名为 `Entrance` 的实体类型的关系，该实体类型没有导航属性。</span><span class="sxs-lookup"><span data-stu-id="fdd39-633">In reality, this code attempts to create a relationship to some entity type called `Entrance` with no navigation property.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-634">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-634">New behavior</span></span>

<span data-ttu-id="fdd39-635">从 EF Core 3.0 开始，上面的代码现执行它以前应执行的操作。</span><span class="sxs-lookup"><span data-stu-id="fdd39-635">Starting with EF Core 3.0, the code above now does what it looked like it should have been doing before.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-636">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-636">Why</span></span>

<span data-ttu-id="fdd39-637">这一旧行为令人非常困惑，尤其是在读取配置代码和查找错误时。</span><span class="sxs-lookup"><span data-stu-id="fdd39-637">The old behavior was very confusing, especially when reading the configuration code and looking for errors.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-638">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-638">Mitigations</span></span>

<span data-ttu-id="fdd39-639">这只会中断使用类型名称字符串显式配置关系而无需显式指定导航属性的应用程序。</span><span class="sxs-lookup"><span data-stu-id="fdd39-639">This will only break applications that are explicitly configuring relationships using strings for type names and without specifying the navigation property explicitly.</span></span>
<span data-ttu-id="fdd39-640">这并不常见。</span><span class="sxs-lookup"><span data-stu-id="fdd39-640">This is not common.</span></span>
<span data-ttu-id="fdd39-641">以前的行为可以通过显式传递导航属性名称的 `null` 获得。</span><span class="sxs-lookup"><span data-stu-id="fdd39-641">The previous behavior can be obtained through explicitly passing `null` for the navigation property name.</span></span>
<span data-ttu-id="fdd39-642">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-642">For example:</span></span>

```csharp
modelBuilder.Entity<Samurai>().HasOne("Some.Entity.Type.Name", null).WithOne();
```

<a name="rtnt"></a>

### <a name="the-return-type-for-several-async-methods-has-been-changed-from-task-to-valuetask"></a><span data-ttu-id="fdd39-643">多个异步方法的返回类型已从 Task 更改为 ValueTask</span><span class="sxs-lookup"><span data-stu-id="fdd39-643">The return type for several async methods has been changed from Task to ValueTask</span></span>

[<span data-ttu-id="fdd39-644">跟踪问题 #15184</span><span class="sxs-lookup"><span data-stu-id="fdd39-644">Tracking Issue #15184</span></span>](https://github.com/dotnet/efcore/issues/15184)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-645">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-645">Old behavior</span></span>

<span data-ttu-id="fdd39-646">以下异步方法之前返回的是 `Task<T>`：</span><span class="sxs-lookup"><span data-stu-id="fdd39-646">The following async methods previously returned a `Task<T>`:</span></span>

* `DbContext.FindAsync()`
* `DbSet.FindAsync()`
* `DbContext.AddAsync()`
* `DbSet.AddAsync()`
* <span data-ttu-id="fdd39-647">`ValueGenerator.NextValueAsync()`（及派生类）</span><span class="sxs-lookup"><span data-stu-id="fdd39-647">`ValueGenerator.NextValueAsync()` (and deriving classes)</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-648">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-648">New behavior</span></span>

<span data-ttu-id="fdd39-649">上述方法现返回一个 `ValueTask<T>`，其中 `T` 与前述相同。</span><span class="sxs-lookup"><span data-stu-id="fdd39-649">The aforementioned methods now return a `ValueTask<T>` over the same `T` as before.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-650">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-650">Why</span></span>

<span data-ttu-id="fdd39-651">此更改会减少在调用这些方法时发生的堆分配数量，从而提高整体性能。</span><span class="sxs-lookup"><span data-stu-id="fdd39-651">This change reduces the number of heap allocations incurred when invoking these methods, improving general performance.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-652">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-652">Mitigations</span></span>

<span data-ttu-id="fdd39-653">仅需重新编译只等待上述 API 的应用程序 -无需更改源。</span><span class="sxs-lookup"><span data-stu-id="fdd39-653">Applications simply awaiting the above APIs only need to be recompiled - no source changes are necessary.</span></span>
<span data-ttu-id="fdd39-654">更复杂的用法（例如，将返回的 `Task` 传递到 `Task.WhenAny()`）通常需要通过对返回的 `ValueTask<T>` 调用 `AsTask()` 来将其转换为 `Task<T>`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-654">A more complex usage (e.g. passing the returned `Task` to `Task.WhenAny()`) typically require that the returned `ValueTask<T>` be converted to a `Task<T>` by calling `AsTask()` on it.</span></span>
<span data-ttu-id="fdd39-655">请注意，这会抵消此更改所带来的分配数减少优势。</span><span class="sxs-lookup"><span data-stu-id="fdd39-655">Note that this negates the allocation reduction that this change brings.</span></span>

<a name="rtt"></a>

### <a name="the-relationaltypemapping-annotation-is-now-just-typemapping"></a><span data-ttu-id="fdd39-656">关系式：TypeMapping 注释现在只是 TypeMapping</span><span class="sxs-lookup"><span data-stu-id="fdd39-656">The Relational:TypeMapping annotation is now just TypeMapping</span></span>

[<span data-ttu-id="fdd39-657">跟踪问题 #9913</span><span class="sxs-lookup"><span data-stu-id="fdd39-657">Tracking Issue #9913</span></span>](https://github.com/dotnet/efcore/issues/9913)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-658">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-658">Old behavior</span></span>

<span data-ttu-id="fdd39-659">类型映射注释的注释名称是“Relational：TypeMapping”。</span><span class="sxs-lookup"><span data-stu-id="fdd39-659">The annotation name for type mapping annotations was "Relational:TypeMapping".</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-660">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-660">New behavior</span></span>

<span data-ttu-id="fdd39-661">类型映射注释的注释名称现在是“TypeMapping”。</span><span class="sxs-lookup"><span data-stu-id="fdd39-661">The annotation name for type mapping annotations is now "TypeMapping".</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-662">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-662">Why</span></span>

<span data-ttu-id="fdd39-663">类型映射现在不仅用于关系数据库提供程序。</span><span class="sxs-lookup"><span data-stu-id="fdd39-663">Type mappings are now used for more than just relational database providers.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-664">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-664">Mitigations</span></span>

<span data-ttu-id="fdd39-665">这只会中断直接作为注释访问类型映射的应用程序，这不常见。</span><span class="sxs-lookup"><span data-stu-id="fdd39-665">This will only break applications that access the type mapping directly as an annotation, which isn't common.</span></span>
<span data-ttu-id="fdd39-666">最合适的修复操作是使用 API 曲面来访问类型映射，而不是直接使用注释。</span><span class="sxs-lookup"><span data-stu-id="fdd39-666">The most appropriate action to fix is to use API surface to access type mappings rather than using the annotation directly.</span></span>

### <a name="totable-on-a-derived-type-throws-an-exception"></a><span data-ttu-id="fdd39-667">派生类型上的 ToTable 会引发异常</span><span class="sxs-lookup"><span data-stu-id="fdd39-667">ToTable on a derived type throws an exception</span></span>

[<span data-ttu-id="fdd39-668">跟踪问题 #11811</span><span class="sxs-lookup"><span data-stu-id="fdd39-668">Tracking Issue #11811</span></span>](https://github.com/dotnet/efcore/issues/11811)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-669">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-669">Old behavior</span></span>

<span data-ttu-id="fdd39-670">在 EF Core 3.0 之前，将忽略调用派生类型的 `ToTable()`，因为只有继承映射策略是 TPH，这是无效的。</span><span class="sxs-lookup"><span data-stu-id="fdd39-670">Before EF Core 3.0, `ToTable()` called on a derived type would be ignored since only inheritance mapping strategy was TPH where this isn't valid.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-671">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-671">New behavior</span></span>

<span data-ttu-id="fdd39-672">从 EF Core 3.0 开始，同时为在以后的版本中添加 TPT 和 TPC 支持做准备，调用派生类型的 `ToTable()` 现在将引发异常，以避免将来发生意外的映射更改。</span><span class="sxs-lookup"><span data-stu-id="fdd39-672">Starting with EF Core 3.0 and in preparation for adding TPT and TPC support in a later release, `ToTable()` called on a derived type will now throw an exception to avoid an unexpected mapping change in the future.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-673">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-673">Why</span></span>

<span data-ttu-id="fdd39-674">目前，将派生类型映射到不同的表是无效的。</span><span class="sxs-lookup"><span data-stu-id="fdd39-674">Currently it isn't valid to map a derived type to a different table.</span></span>
<span data-ttu-id="fdd39-675">这种改变避免了在将来当它变为有效时被中断。</span><span class="sxs-lookup"><span data-stu-id="fdd39-675">This change avoids breaking in the future when it becomes a valid thing to do.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-676">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-676">Mitigations</span></span>

<span data-ttu-id="fdd39-677">删除将派生类型映射到其他表的任何尝试。</span><span class="sxs-lookup"><span data-stu-id="fdd39-677">Remove any attempts to map derived types to other tables.</span></span>

### <a name="forsqlserverhasindex-replaced-with-hasindex"></a><span data-ttu-id="fdd39-678">用 HasIndex 替换 ForSqlServerHasIndex</span><span class="sxs-lookup"><span data-stu-id="fdd39-678">ForSqlServerHasIndex replaced with HasIndex</span></span>

[<span data-ttu-id="fdd39-679">跟踪问题 #12366</span><span class="sxs-lookup"><span data-stu-id="fdd39-679">Tracking Issue #12366</span></span>](https://github.com/dotnet/efcore/issues/12366)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-680">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-680">Old behavior</span></span>

<span data-ttu-id="fdd39-681">在 EF Core 3.0 之前，`ForSqlServerHasIndex().ForSqlServerInclude()` 提供了一种配置与 `INCLUDE` 一起使用的列的方法。</span><span class="sxs-lookup"><span data-stu-id="fdd39-681">Before EF Core 3.0, `ForSqlServerHasIndex().ForSqlServerInclude()` provided a way to configure columns used with `INCLUDE`.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-682">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-682">New behavior</span></span>

<span data-ttu-id="fdd39-683">从 EF Core 3.0 开始，现在支持在关系级别上对索引使用 `Include`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-683">Starting with EF Core 3.0, using `Include` on an index is now supported at the relational level.</span></span>
<span data-ttu-id="fdd39-684">请使用 `HasIndex().ForSqlServerInclude()`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-684">Use `HasIndex().ForSqlServerInclude()`.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-685">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-685">Why</span></span>

<span data-ttu-id="fdd39-686">此更改是为了将用于索引的 API 与 `Include` 合并到一个位置，以供所有数据库提供程序使用。</span><span class="sxs-lookup"><span data-stu-id="fdd39-686">This change was made to consolidate the API for indexes with `Include` into one place for all database providers.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-687">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-687">Mitigations</span></span>

<span data-ttu-id="fdd39-688">使用新的 API，如上所示。</span><span class="sxs-lookup"><span data-stu-id="fdd39-688">Use the new API, as shown above.</span></span>

### <a name="metadata-api-changes"></a><span data-ttu-id="fdd39-689">元数据 API 更改</span><span class="sxs-lookup"><span data-stu-id="fdd39-689">Metadata API changes</span></span>

[<span data-ttu-id="fdd39-690">跟踪问题 #214</span><span class="sxs-lookup"><span data-stu-id="fdd39-690">Tracking Issue #214</span></span>](https://github.com/dotnet/efcore/issues/214)

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-691">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-691">New behavior</span></span>

<span data-ttu-id="fdd39-692">以下属性已转换为扩展方法：</span><span class="sxs-lookup"><span data-stu-id="fdd39-692">The following properties were converted to extension methods:</span></span>

* `IEntityType.QueryFilter` -> `GetQueryFilter()`
* `IEntityType.DefiningQuery` -> `GetDefiningQuery()`
* `IProperty.IsShadowProperty` -> `IsShadowProperty()`
* `IProperty.BeforeSaveBehavior` -> `GetBeforeSaveBehavior()`
* `IProperty.AfterSaveBehavior` -> `GetAfterSaveBehavior()`

#### <a name="why"></a><span data-ttu-id="fdd39-693">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-693">Why</span></span>

<span data-ttu-id="fdd39-694">此更改简化了上述接口的实现。</span><span class="sxs-lookup"><span data-stu-id="fdd39-694">This change simplifies the implementation of the aforementioned interfaces.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-695">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-695">Mitigations</span></span>

<span data-ttu-id="fdd39-696">使用新的扩展方法。</span><span class="sxs-lookup"><span data-stu-id="fdd39-696">Use the new extension methods.</span></span>

<a name="provider"></a>

### <a name="provider-specific-metadata-api-changes"></a><span data-ttu-id="fdd39-697">特定于提供程序的元数据 API 更改</span><span class="sxs-lookup"><span data-stu-id="fdd39-697">Provider-specific Metadata API changes</span></span>

[<span data-ttu-id="fdd39-698">跟踪问题 #214</span><span class="sxs-lookup"><span data-stu-id="fdd39-698">Tracking Issue #214</span></span>](https://github.com/dotnet/efcore/issues/214)

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-699">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-699">New behavior</span></span>

<span data-ttu-id="fdd39-700">将展开特定于提供程序的扩展方法：</span><span class="sxs-lookup"><span data-stu-id="fdd39-700">The provider-specific extension methods will be flattened out:</span></span>

* `IProperty.Relational().ColumnName` -> `IProperty.GetColumnName()`
* `IEntityType.SqlServer().IsMemoryOptimized` -> `IEntityType.IsMemoryOptimized()`
* `PropertyBuilder.UseSqlServerIdentityColumn()` -> `PropertyBuilder.UseIdentityColumn()`

#### <a name="why"></a><span data-ttu-id="fdd39-701">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-701">Why</span></span>

<span data-ttu-id="fdd39-702">此更改简化了上述扩展方法的实现。</span><span class="sxs-lookup"><span data-stu-id="fdd39-702">This change simplifies the implementation of the aforementioned extension methods.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-703">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-703">Mitigations</span></span>

<span data-ttu-id="fdd39-704">使用新的扩展方法。</span><span class="sxs-lookup"><span data-stu-id="fdd39-704">Use the new extension methods.</span></span>

<a name="pragma"></a>

### <a name="ef-core-no-longer-sends-pragma-for-sqlite-fk-enforcement"></a><span data-ttu-id="fdd39-705">EF Core 不再发送 pragma 来执行 SQLite FK</span><span class="sxs-lookup"><span data-stu-id="fdd39-705">EF Core no longer sends pragma for SQLite FK enforcement</span></span>

[<span data-ttu-id="fdd39-706">跟踪问题 #12151</span><span class="sxs-lookup"><span data-stu-id="fdd39-706">Tracking Issue #12151</span></span>](https://github.com/dotnet/efcore/issues/12151)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-707">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-707">Old behavior</span></span>

<span data-ttu-id="fdd39-708">在 EF Core 3.0 之前，当打开与 SQLite 的连接时，EF Core 会发送 `PRAGMA foreign_keys = 1`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-708">Before EF Core 3.0, EF Core would send `PRAGMA foreign_keys = 1` when a connection to SQLite is opened.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-709">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-709">New behavior</span></span>

<span data-ttu-id="fdd39-710">从 EF Core 3.0 开始，当打开到 SQLite 的连接时，EF Core 不再发送 `PRAGMA foreign_keys = 1`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-710">Starting with EF Core 3.0, EF Core no longer sends `PRAGMA foreign_keys = 1` when a connection to SQLite is opened.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-711">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-711">Why</span></span>

<span data-ttu-id="fdd39-712">之所以进行此更改，是因为 EF Core 默认使用 `SQLitePCLRaw.bundle_e_sqlite3`，这意味着 FK 强制执行操作在默认情况下是打开的，并且不需要在每次打开连接时显式启用。</span><span class="sxs-lookup"><span data-stu-id="fdd39-712">This change was made because EF Core uses `SQLitePCLRaw.bundle_e_sqlite3` by default, which in turn means that FK enforcement is switched on by default and doesn't need to be explicitly enabled each time a connection is opened.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-713">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-713">Mitigations</span></span>

<span data-ttu-id="fdd39-714">默认情况下，SQLitePCLRaw.bundle_e_sqlite3 中启用了默认用于 EF Core 的外键。</span><span class="sxs-lookup"><span data-stu-id="fdd39-714">Foreign keys are enabled by default in SQLitePCLRaw.bundle_e_sqlite3, which is used by default for EF Core.</span></span>
<span data-ttu-id="fdd39-715">对于其他情况，可以通过在连接字符串中指定 `Foreign Keys=True` 来启用外键。</span><span class="sxs-lookup"><span data-stu-id="fdd39-715">For other cases, foreign keys can be enabled by specifying `Foreign Keys=True` in your connection string.</span></span>

<a name="sqlite3"></a>

### <a name="microsoftentityframeworkcoresqlite-now-depends-on-sqlitepclrawbundle_e_sqlite3"></a><span data-ttu-id="fdd39-716">Microsoft.EntityFrameworkCore.Sqlite 现在依赖于 SQLitePCLRaw.bundle_e_sqlite3</span><span class="sxs-lookup"><span data-stu-id="fdd39-716">Microsoft.EntityFrameworkCore.Sqlite now depends on SQLitePCLRaw.bundle_e_sqlite3</span></span>

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-717">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-717">Old behavior</span></span>

<span data-ttu-id="fdd39-718">在 EF Core 3.0 之前，EF Core 使用 `SQLitePCLRaw.bundle_green`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-718">Before EF Core 3.0, EF Core used `SQLitePCLRaw.bundle_green`.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-719">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-719">New behavior</span></span>

<span data-ttu-id="fdd39-720">从 EF Core 3.0 开始，EF Core 使用 `SQLitePCLRaw.bundle_e_sqlite3`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-720">Starting with EF Core 3.0, EF Core uses `SQLitePCLRaw.bundle_e_sqlite3`.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-721">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-721">Why</span></span>

<span data-ttu-id="fdd39-722">此更改是为了使 iOS 上使用的 SQLite 版本与其他平台一致。</span><span class="sxs-lookup"><span data-stu-id="fdd39-722">This change was made so that the version of SQLite used on iOS consistent with other platforms.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-723">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-723">Mitigations</span></span>

<span data-ttu-id="fdd39-724">若要在 iOS 上使用本机 SQLite 版本，请配置 `Microsoft.Data.Sqlite` 以使用其他 `SQLitePCLRaw` 捆绑包。</span><span class="sxs-lookup"><span data-stu-id="fdd39-724">To use the native SQLite version on iOS, configure `Microsoft.Data.Sqlite` to use a different `SQLitePCLRaw` bundle.</span></span>

<a name="guid"></a>

### <a name="guid-values-are-now-stored-as-text-on-sqlite"></a><span data-ttu-id="fdd39-725">GUID 值现在以文本形式存储在 SQLite 上</span><span class="sxs-lookup"><span data-stu-id="fdd39-725">Guid values are now stored as TEXT on SQLite</span></span>

[<span data-ttu-id="fdd39-726">跟踪问题 #15078</span><span class="sxs-lookup"><span data-stu-id="fdd39-726">Tracking Issue #15078</span></span>](https://github.com/dotnet/efcore/issues/15078)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-727">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-727">Old behavior</span></span>

<span data-ttu-id="fdd39-728">GUID 值之前以 BLOB 值形式存储在 SQLite 上。</span><span class="sxs-lookup"><span data-stu-id="fdd39-728">Guid values were previously stored as BLOB values on SQLite.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-729">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-729">New behavior</span></span>

<span data-ttu-id="fdd39-730">Guid 值现在以文本形式存储。</span><span class="sxs-lookup"><span data-stu-id="fdd39-730">Guid values are now stored as TEXT.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-731">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-731">Why</span></span>

<span data-ttu-id="fdd39-732">GUID 的二进制格式不会进行标准化。</span><span class="sxs-lookup"><span data-stu-id="fdd39-732">The binary format of Guids is not standardized.</span></span> <span data-ttu-id="fdd39-733">以文本形式存储值使数据库与其他技术更兼容。</span><span class="sxs-lookup"><span data-stu-id="fdd39-733">Storing the values as TEXT makes the database more compatible with other technologies.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-734">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-734">Mitigations</span></span>

<span data-ttu-id="fdd39-735">现在通过执行如下的 SQL，可以将现有数据库转成新的格式。</span><span class="sxs-lookup"><span data-stu-id="fdd39-735">You can migrate existing databases to the new format by executing SQL like the following.</span></span>

```sql
UPDATE MyTable
SET GuidColumn = hex(substr(GuidColumn, 4, 1)) ||
                 hex(substr(GuidColumn, 3, 1)) ||
                 hex(substr(GuidColumn, 2, 1)) ||
                 hex(substr(GuidColumn, 1, 1)) || '-' ||
                 hex(substr(GuidColumn, 6, 1)) ||
                 hex(substr(GuidColumn, 5, 1)) || '-' ||
                 hex(substr(GuidColumn, 8, 1)) ||
                 hex(substr(GuidColumn, 7, 1)) || '-' ||
                 hex(substr(GuidColumn, 9, 2)) || '-' ||
                 hex(substr(GuidColumn, 11, 6))
WHERE typeof(GuidColumn) == 'blob';
```

<span data-ttu-id="fdd39-736">在 EF Core 中，还可以通过为这些属性配置值转换器，继续使用以前的行为模式。</span><span class="sxs-lookup"><span data-stu-id="fdd39-736">In EF Core, you could also continue using the previous behavior by configuring a value converter on these properties.</span></span>

```csharp
modelBuilder
    .Entity<MyEntity>()
    .Property(e => e.GuidProperty)
    .HasConversion(
        g => g.ToByteArray(),
        b => new Guid(b));
```

<span data-ttu-id="fdd39-737">Microsoft.Data.Sqlite 仍然能够从“BLOB”和“文本”列读取 GUID 值；但是，由于参数和常量的默认格式已更改，可能需要针对涉及 GUID 的大多数情况采取措施。</span><span class="sxs-lookup"><span data-stu-id="fdd39-737">Microsoft.Data.Sqlite remains capable of reading Guid values from both BLOB and TEXT columns; however, since the default format for parameters and constants has changed you'll likely need to take action for most scenarios involving Guids.</span></span>

<a name="char"></a>

### <a name="char-values-are-now-stored-as-text-on-sqlite"></a><span data-ttu-id="fdd39-738">Char 值现在以文本形式存储在 SQLite 上</span><span class="sxs-lookup"><span data-stu-id="fdd39-738">Char values are now stored as TEXT on SQLite</span></span>

[<span data-ttu-id="fdd39-739">跟踪问题 #15020</span><span class="sxs-lookup"><span data-stu-id="fdd39-739">Tracking Issue #15020</span></span>](https://github.com/dotnet/efcore/issues/15020)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-740">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-740">Old behavior</span></span>

<span data-ttu-id="fdd39-741">Char 值之前以整数值形式存储在 SQLite 上。</span><span class="sxs-lookup"><span data-stu-id="fdd39-741">Char values were previously sored as INTEGER values on SQLite.</span></span> <span data-ttu-id="fdd39-742">例如，A 的 char 值存储为整数值 65。</span><span class="sxs-lookup"><span data-stu-id="fdd39-742">For example, a char value of *A* was stored as the integer value 65.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-743">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-743">New behavior</span></span>

<span data-ttu-id="fdd39-744">Char 值现在以文本形式存储。</span><span class="sxs-lookup"><span data-stu-id="fdd39-744">Char values are now stored as TEXT.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-745">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-745">Why</span></span>

<span data-ttu-id="fdd39-746">以文本形式存储值显得更加自然，并且使数据库与其他技术更兼容。</span><span class="sxs-lookup"><span data-stu-id="fdd39-746">Storing the values as TEXT is more natural and makes the database more compatible with other technologies.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-747">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-747">Mitigations</span></span>

<span data-ttu-id="fdd39-748">现在通过执行如下的 SQL，可以将现有数据库转成新的格式。</span><span class="sxs-lookup"><span data-stu-id="fdd39-748">You can migrate existing databases to the new format by executing SQL like the following.</span></span>

```sql
UPDATE MyTable
SET CharColumn = char(CharColumn)
WHERE typeof(CharColumn) = 'integer';
```

<span data-ttu-id="fdd39-749">在 EF Core 中，还可以通过为这些属性配置值转换器，继续使用以前的行为模式。</span><span class="sxs-lookup"><span data-stu-id="fdd39-749">In EF Core, you could also continue using the previous behavior by configuring a value converter on these properties.</span></span>

```csharp
modelBuilder
    .Entity<MyEntity>()
    .Property(e => e.CharProperty)
    .HasConversion(
        c => (long)c,
        i => (char)i);
```

<span data-ttu-id="fdd39-750">Microsoft.Data.Sqlite 也仍然能够读取整数列和文本列的字符值，因此某些情况可能不需要任何操作。</span><span class="sxs-lookup"><span data-stu-id="fdd39-750">Microsoft.Data.Sqlite also remains capable of reading character values from both INTEGER and TEXT columns, so certain scenarios may not require any action.</span></span>

<a name="migid"></a>

### <a name="migration-ids-are-now-generated-using-the-invariant-cultures-calendar"></a><span data-ttu-id="fdd39-751">现在使用固定区域性的日历生成迁移 ID</span><span class="sxs-lookup"><span data-stu-id="fdd39-751">Migration IDs are now generated using the invariant culture's calendar</span></span>

[<span data-ttu-id="fdd39-752">跟踪问题 #12978</span><span class="sxs-lookup"><span data-stu-id="fdd39-752">Tracking Issue #12978</span></span>](https://github.com/dotnet/efcore/issues/12978)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-753">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-753">Old behavior</span></span>

<span data-ttu-id="fdd39-754">以前使用当前区域性的日历无意生成迁移 ID。</span><span class="sxs-lookup"><span data-stu-id="fdd39-754">Migration IDs were inadvertently generated using the current culture's calendar.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-755">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-755">New behavior</span></span>

<span data-ttu-id="fdd39-756">现在始终使用固定区域性的日历（公历）生成迁移 ID。</span><span class="sxs-lookup"><span data-stu-id="fdd39-756">Migration IDs are now always generated using the invariant culture's calendar (Gregorian).</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-757">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-757">Why</span></span>

<span data-ttu-id="fdd39-758">更新数据库或解决合并冲突时，迁移的顺序非常重要。</span><span class="sxs-lookup"><span data-stu-id="fdd39-758">The order of migrations is important when updating the database or resolving merge conflicts.</span></span> <span data-ttu-id="fdd39-759">使用固定日历可以避免因团队成员采用不同系统日历而产生的顺序问题。</span><span class="sxs-lookup"><span data-stu-id="fdd39-759">Using the invariant calendar avoids ordering issues that can result from team members having different system calendars.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-760">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-760">Mitigations</span></span>

<span data-ttu-id="fdd39-761">如果有人使用年份时间大于公历日历的非公历日历（如泰国佛历），则会受到影响。</span><span class="sxs-lookup"><span data-stu-id="fdd39-761">This change affects anyone using a non-Gregorian calendar where the year is greater than the Gregorian calendar (like the Thai Buddhist calendar).</span></span> <span data-ttu-id="fdd39-762">现有迁移 ID 需要进行更新，以便新迁移排在现有迁移之后。</span><span class="sxs-lookup"><span data-stu-id="fdd39-762">Existing migration IDs will need to be updated so that new migrations are ordered after existing migrations.</span></span>

<span data-ttu-id="fdd39-763">在迁移的设计者文件的 Migration 属性中可以找到迁移 ID。</span><span class="sxs-lookup"><span data-stu-id="fdd39-763">The migration ID can be found in the Migration attribute in the migrations' designer files.</span></span>

```diff
 [DbContext(typeof(MyDbContext))]
-[Migration("25620318122820_MyMigration")]
+[Migration("20190318122820_MyMigration")]
 partial class MyMigration
 {
```

<span data-ttu-id="fdd39-764">迁移历史记录表还需要更新。</span><span class="sxs-lookup"><span data-stu-id="fdd39-764">The Migrations history table also needs to be updated.</span></span>

```sql
UPDATE __EFMigrationsHistory
SET MigrationId = CONCAT(LEFT(MigrationId, 4)  - 543, SUBSTRING(MigrationId, 4, 150))
```

<a name="urn"></a>

### <a name="userownumberforpaging-has-been-removed"></a><span data-ttu-id="fdd39-765">UseRowNumberForPaging 已删除</span><span class="sxs-lookup"><span data-stu-id="fdd39-765">UseRowNumberForPaging has been removed</span></span>

[<span data-ttu-id="fdd39-766">跟踪问题 #16400</span><span class="sxs-lookup"><span data-stu-id="fdd39-766">Tracking Issue #16400</span></span>](https://github.com/dotnet/efcore/issues/16400)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-767">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-767">Old behavior</span></span>

<span data-ttu-id="fdd39-768">在 EF Core 3.0 之前，`UseRowNumberForPaging` 可用于生成与 SQL Server 2008 兼容的分页 SQL。</span><span class="sxs-lookup"><span data-stu-id="fdd39-768">Before EF Core 3.0, `UseRowNumberForPaging` could be used to generate SQL for paging that is compatible with SQL Server 2008.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-769">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-769">New behavior</span></span>

<span data-ttu-id="fdd39-770">从 EF Core 3.0 开始，EF 将仅生成仅与更高的 SQL Server 版本兼容的分页 SQL。</span><span class="sxs-lookup"><span data-stu-id="fdd39-770">Starting with EF Core 3.0, EF will only generate SQL for paging that is only compatible with later SQL Server versions.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-771">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-771">Why</span></span>

<span data-ttu-id="fdd39-772">我们正在进行此更改，因为 [SQL Server 2008 不再是受支持的产品](https://blogs.msdn.microsoft.com/sqlreleaseservices/end-of-mainstream-support-for-sql-server-2008-and-sql-server-2008-r2/)，并且更新此功能以使用在 EF Core 3.0 中做出的查询更改是一项重要工作。</span><span class="sxs-lookup"><span data-stu-id="fdd39-772">We are making this change because [SQL Server 2008 is no longer a supported product](https://blogs.msdn.microsoft.com/sqlreleaseservices/end-of-mainstream-support-for-sql-server-2008-and-sql-server-2008-r2/) and updating this feature to work with the query changes made in EF Core 3.0 is significant work.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-773">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-773">Mitigations</span></span>

<span data-ttu-id="fdd39-774">建议更新到更高的 SQL Server 版本，或者使用更高的兼容性级别，以便支持生成的 SQL。</span><span class="sxs-lookup"><span data-stu-id="fdd39-774">We recommend updating to a newer version of SQL Server, or using a higher compatibility level, so that the generated SQL is supported.</span></span> <span data-ttu-id="fdd39-775">这就是说，如果无法执行此操作，请[在跟踪问题中做出详细注释](https://github.com/dotnet/efcore/issues/16400)。</span><span class="sxs-lookup"><span data-stu-id="fdd39-775">That being said, if you are unable to do this, then please [comment on the tracking issue](https://github.com/dotnet/efcore/issues/16400) with details.</span></span> <span data-ttu-id="fdd39-776">我们可能会根据反馈重新考虑这个决定。</span><span class="sxs-lookup"><span data-stu-id="fdd39-776">We may revisit this decision based on feedback.</span></span>

<a name="xinfo"></a>

### <a name="extension-infometadata-has-been-removed-from-idbcontextoptionsextension"></a><span data-ttu-id="fdd39-777">已从 IDbContextOptionsExtension 中删除扩展信息/元数据</span><span class="sxs-lookup"><span data-stu-id="fdd39-777">Extension info/metadata has been removed from IDbContextOptionsExtension</span></span>

[<span data-ttu-id="fdd39-778">跟踪问题 #16119</span><span class="sxs-lookup"><span data-stu-id="fdd39-778">Tracking Issue #16119</span></span>](https://github.com/dotnet/efcore/issues/16119)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-779">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-779">Old behavior</span></span>

<span data-ttu-id="fdd39-780">`IDbContextOptionsExtension` 包含用于提供扩展元数据的方法。</span><span class="sxs-lookup"><span data-stu-id="fdd39-780">`IDbContextOptionsExtension` contained methods for providing metadata about the extension.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-781">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-781">New behavior</span></span>

<span data-ttu-id="fdd39-782">这些方法已迁移到从新 `IDbContextOptionsExtension.Info` 属性返回的新 `DbContextOptionsExtensionInfo` 抽象基类。</span><span class="sxs-lookup"><span data-stu-id="fdd39-782">These methods have been moved onto a new `DbContextOptionsExtensionInfo` abstract base class, which is returned from a new `IDbContextOptionsExtension.Info` property.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-783">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-783">Why</span></span>

<span data-ttu-id="fdd39-784">在从版本 2.0 迁移到版本 3.0 的过程中，我们需要多次添加或更改这些方法。</span><span class="sxs-lookup"><span data-stu-id="fdd39-784">Over the releases from 2.0 to 3.0 we needed to add to or change these methods several times.</span></span>
<span data-ttu-id="fdd39-785">将它们拆分成新抽象基类可以更轻松地进行此类更改，而不会破坏现有扩展。</span><span class="sxs-lookup"><span data-stu-id="fdd39-785">Breaking them out into a new abstract base class will make it easier to make these kind of changes without breaking existing extensions.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-786">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-786">Mitigations</span></span>

<span data-ttu-id="fdd39-787">将扩展更新为采用新模式。</span><span class="sxs-lookup"><span data-stu-id="fdd39-787">Update extensions to follow the new pattern.</span></span>
<span data-ttu-id="fdd39-788">例如，许多用于 EF Core 源代码中不同种类扩展的 `IDbContextOptionsExtension` 实现。</span><span class="sxs-lookup"><span data-stu-id="fdd39-788">Examples are found in the many implementations of `IDbContextOptionsExtension` for different kinds of extensions in the EF Core source code.</span></span>

<a name="lqpe"></a>

### <a name="logquerypossibleexceptionwithaggregateoperator-has-been-renamed"></a><span data-ttu-id="fdd39-789">已重命名 LogQueryPossibleExceptionWithAggregateOperator</span><span class="sxs-lookup"><span data-stu-id="fdd39-789">LogQueryPossibleExceptionWithAggregateOperator has been renamed</span></span>

[<span data-ttu-id="fdd39-790">跟踪问题 #10985</span><span class="sxs-lookup"><span data-stu-id="fdd39-790">Tracking Issue #10985</span></span>](https://github.com/dotnet/efcore/issues/10985)

#### <a name="change"></a><span data-ttu-id="fdd39-791">更改</span><span class="sxs-lookup"><span data-stu-id="fdd39-791">Change</span></span>

<span data-ttu-id="fdd39-792">`RelationalEventId.LogQueryPossibleExceptionWithAggregateOperator` 已重命名为 `RelationalEventId.LogQueryPossibleExceptionWithAggregateOperatorWarning`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-792">`RelationalEventId.LogQueryPossibleExceptionWithAggregateOperator` has been renamed to `RelationalEventId.LogQueryPossibleExceptionWithAggregateOperatorWarning`.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-793">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-793">Why</span></span>

<span data-ttu-id="fdd39-794">将此警告事件的命名方式与所有其他警告事件保持一致。</span><span class="sxs-lookup"><span data-stu-id="fdd39-794">Aligns the naming of this warning event with all other warning events.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-795">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-795">Mitigations</span></span>

<span data-ttu-id="fdd39-796">使用新名称。</span><span class="sxs-lookup"><span data-stu-id="fdd39-796">Use the new name.</span></span> <span data-ttu-id="fdd39-797">（注意：事件 ID 号码未更改。）</span><span class="sxs-lookup"><span data-stu-id="fdd39-797">(Note that the event ID number has not changed.)</span></span>

<a name="clarify"></a>

### <a name="clarify-api-for-foreign-key-constraint-names"></a><span data-ttu-id="fdd39-798">阐明 API 的外键约束名称</span><span class="sxs-lookup"><span data-stu-id="fdd39-798">Clarify API for foreign key constraint names</span></span>

[<span data-ttu-id="fdd39-799">跟踪问题 #10730</span><span class="sxs-lookup"><span data-stu-id="fdd39-799">Tracking Issue #10730</span></span>](https://github.com/dotnet/efcore/issues/10730)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-800">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-800">Old behavior</span></span>

<span data-ttu-id="fdd39-801">在 EF Core 3.0 之前，外键约束名称被简单地称为“名称”。</span><span class="sxs-lookup"><span data-stu-id="fdd39-801">Before EF Core 3.0, foreign key constraint names were referred to as simply the "name".</span></span> <span data-ttu-id="fdd39-802">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-802">For example:</span></span>

```csharp
var constraintName = myForeignKey.Name;
```

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-803">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-803">New behavior</span></span>

<span data-ttu-id="fdd39-804">从 EF Core 3.0 开始，外键约束名称现在被称为“约束名称”。</span><span class="sxs-lookup"><span data-stu-id="fdd39-804">Starting with EF Core 3.0, foreign key constraint names are now referred to as the "constraint name".</span></span> <span data-ttu-id="fdd39-805">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-805">For example:</span></span>

```csharp
var constraintName = myForeignKey.ConstraintName;
```

#### <a name="why"></a><span data-ttu-id="fdd39-806">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-806">Why</span></span>

<span data-ttu-id="fdd39-807">这样不仅可以让此领域的命名方式保持一致，还阐明了这是外键约束的名称，不是定义外键所依据的列或属性名称。</span><span class="sxs-lookup"><span data-stu-id="fdd39-807">This change brings consistency to naming in this area, and also clarifies that this is the name of the foreign key constraint, and not the column or property name that the foreign key is defined on.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-808">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-808">Mitigations</span></span>

<span data-ttu-id="fdd39-809">使用新名称。</span><span class="sxs-lookup"><span data-stu-id="fdd39-809">Use the new name.</span></span>

<a name="irdc2"></a>

### <a name="irelationaldatabasecreatorhastableshastablesasync-have-been-made-public"></a><span data-ttu-id="fdd39-810">IRelationalDatabaseCreator.HasTables/HasTablesAsync 已公开</span><span class="sxs-lookup"><span data-stu-id="fdd39-810">IRelationalDatabaseCreator.HasTables/HasTablesAsync have been made public</span></span>

[<span data-ttu-id="fdd39-811">跟踪问题 #15997</span><span class="sxs-lookup"><span data-stu-id="fdd39-811">Tracking Issue #15997</span></span>](https://github.com/dotnet/efcore/issues/15997)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-812">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-812">Old behavior</span></span>

<span data-ttu-id="fdd39-813">在 EF Core 3.0 推出前，这些方法受保护。</span><span class="sxs-lookup"><span data-stu-id="fdd39-813">Before EF Core 3.0, these methods were protected.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-814">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-814">New behavior</span></span>

<span data-ttu-id="fdd39-815">自 EF Core 3.0 起，这些方法是公共的。</span><span class="sxs-lookup"><span data-stu-id="fdd39-815">Starting with EF Core 3.0, these methods are public.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-816">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-816">Why</span></span>

<span data-ttu-id="fdd39-817">EF 使用这些方法来确定数据库是否已创建但为空。</span><span class="sxs-lookup"><span data-stu-id="fdd39-817">These methods are used by EF to determine if a database is created but empty.</span></span> <span data-ttu-id="fdd39-818">这也适用于在 EF 外部确定是否要应用迁移。</span><span class="sxs-lookup"><span data-stu-id="fdd39-818">This can also be useful from outside EF when determining whether or not to apply migrations.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-819">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-819">Mitigations</span></span>

<span data-ttu-id="fdd39-820">更改任何重写的可访问性。</span><span class="sxs-lookup"><span data-stu-id="fdd39-820">Change the accessibility of any overrides.</span></span>

<a name="dip"></a>

### <a name="microsoftentityframeworkcoredesign-is-now-a-developmentdependency-package"></a><span data-ttu-id="fdd39-821">Microsoft.EntityFrameworkCore.Design 现在是 DevelopmentDependency 包</span><span class="sxs-lookup"><span data-stu-id="fdd39-821">Microsoft.EntityFrameworkCore.Design is now a DevelopmentDependency package</span></span>

[<span data-ttu-id="fdd39-822">跟踪问题 #11506</span><span class="sxs-lookup"><span data-stu-id="fdd39-822">Tracking Issue #11506</span></span>](https://github.com/dotnet/efcore/issues/11506)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-823">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-823">Old behavior</span></span>

<span data-ttu-id="fdd39-824">在 EF Core 3.0 推出前，Microsoft.EntityFrameworkCore.Design 是常规 NuGet 包，它的程序集可以由依赖它的项目引用。</span><span class="sxs-lookup"><span data-stu-id="fdd39-824">Before EF Core 3.0, Microsoft.EntityFrameworkCore.Design was a regular NuGet package whose assembly could be referenced by projects that depended on it.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-825">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-825">New behavior</span></span>

<span data-ttu-id="fdd39-826">自 EF Core 3.0 起，它是 DevelopmentDependency 包。</span><span class="sxs-lookup"><span data-stu-id="fdd39-826">Starting with EF Core 3.0, it is a DevelopmentDependency package.</span></span> <span data-ttu-id="fdd39-827">这意味着，依赖项不会过渡流动到其他项目中，并且你也无法再默认引用它的程序集。</span><span class="sxs-lookup"><span data-stu-id="fdd39-827">This means that the dependency won't flow transitively into other projects, and that you can no longer, by default, reference its assembly.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-828">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-828">Why</span></span>

<span data-ttu-id="fdd39-829">此包仅用于设计时。</span><span class="sxs-lookup"><span data-stu-id="fdd39-829">This package is only intended to be used at design time.</span></span> <span data-ttu-id="fdd39-830">已部署的应用程序不得引用它。</span><span class="sxs-lookup"><span data-stu-id="fdd39-830">Deployed applications shouldn't reference it.</span></span> <span data-ttu-id="fdd39-831">让它成为 DevelopmentDependency 包强化了此建议。</span><span class="sxs-lookup"><span data-stu-id="fdd39-831">Making the package a DevelopmentDependency reinforces this recommendation.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-832">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-832">Mitigations</span></span>

<span data-ttu-id="fdd39-833">如果需要引用此包来重写 EF Core 的设计时行为，则可更新项目中的 PackageReference 项元数据。</span><span class="sxs-lookup"><span data-stu-id="fdd39-833">If you need to reference this package to override EF Core's design-time behavior, then you can update PackageReference item metadata in your project.</span></span>

```xml
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="3.0.0">
  <PrivateAssets>all</PrivateAssets>
  <!-- Remove IncludeAssets to allow compiling against the assembly -->
  <!--<IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>-->
</PackageReference>
```

<span data-ttu-id="fdd39-834">如果正在通过 Microsoft.EntityFrameworkCore.Tools 过渡引用此包，必须向此包添加显式 PackageReference，以更改它的元数据。</span><span class="sxs-lookup"><span data-stu-id="fdd39-834">If the package is being referenced transitively via Microsoft.EntityFrameworkCore.Tools, you will need to add an explicit PackageReference to the package to change its metadata.</span></span> <span data-ttu-id="fdd39-835">必须在需要此包中的类型的任何项目下添加此类显式引用。</span><span class="sxs-lookup"><span data-stu-id="fdd39-835">Such an explicit reference must be added to any project where the types from the package are needed.</span></span>

<a name="SQLitePCL"></a>

### <a name="sqlitepclraw-updated-to-version-200"></a><span data-ttu-id="fdd39-836">SQLitePCL.raw 已更新为版本 2.0.0</span><span class="sxs-lookup"><span data-stu-id="fdd39-836">SQLitePCL.raw updated to version 2.0.0</span></span>

[<span data-ttu-id="fdd39-837">跟踪问题 #14824</span><span class="sxs-lookup"><span data-stu-id="fdd39-837">Tracking Issue #14824</span></span>](https://github.com/dotnet/efcore/issues/14824)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-838">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-838">Old behavior</span></span>

<span data-ttu-id="fdd39-839">Microsoft.EntityFrameworkCore.Sqlite 以前依赖 SQLitePCL.raw 版本 1.1.12。</span><span class="sxs-lookup"><span data-stu-id="fdd39-839">Microsoft.EntityFrameworkCore.Sqlite previously depended on version 1.1.12 of SQLitePCL.raw.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-840">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-840">New behavior</span></span>

<span data-ttu-id="fdd39-841">我们已将包更新为依赖版本 2.0.0。</span><span class="sxs-lookup"><span data-stu-id="fdd39-841">We've updated our package to depend on version 2.0.0.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-842">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-842">Why</span></span>

<span data-ttu-id="fdd39-843">SQLitePCL.raw 版本 2.0.0 定目标到 .NET Standard 2.0。</span><span class="sxs-lookup"><span data-stu-id="fdd39-843">Version 2.0.0 of SQLitePCL.raw targets .NET Standard 2.0.</span></span> <span data-ttu-id="fdd39-844">它以前定目标到 .NET Standard 1.1，这就要求必须将可传递的包用作大型收尾，才能正常运行。</span><span class="sxs-lookup"><span data-stu-id="fdd39-844">It previously targeted .NET Standard 1.1 which required a large closure of transitive packages to work.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-845">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-845">Mitigations</span></span>

<span data-ttu-id="fdd39-846">SQLitePCL.raw 版本 2.0.0 包括一些重大变化。</span><span class="sxs-lookup"><span data-stu-id="fdd39-846">SQLitePCL.raw version 2.0.0 includes some breaking changes.</span></span> <span data-ttu-id="fdd39-847">有关详细信息，请参阅[发行说明](https://github.com/ericsink/SQLitePCL.raw/blob/v2/v2.md)。</span><span class="sxs-lookup"><span data-stu-id="fdd39-847">See the [release notes](https://github.com/ericsink/SQLitePCL.raw/blob/v2/v2.md) for details.</span></span>

<a name="NetTopologySuite"></a>

### <a name="nettopologysuite-updated-to-version-200"></a><span data-ttu-id="fdd39-848">NetTopologySuite 已更新为版本 2.0.0</span><span class="sxs-lookup"><span data-stu-id="fdd39-848">NetTopologySuite updated to version 2.0.0</span></span>

[<span data-ttu-id="fdd39-849">跟踪问题 #14825</span><span class="sxs-lookup"><span data-stu-id="fdd39-849">Tracking Issue #14825</span></span>](https://github.com/dotnet/efcore/issues/14825)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-850">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-850">Old behavior</span></span>

<span data-ttu-id="fdd39-851">空间包以前依赖于 NetTopologySuite 的 1.15.1 版。</span><span class="sxs-lookup"><span data-stu-id="fdd39-851">The spatial packages previously depended on version 1.15.1 of NetTopologySuite.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-852">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-852">New behavior</span></span>

<span data-ttu-id="fdd39-853">我们已将包更新为依赖版本 2.0.0。</span><span class="sxs-lookup"><span data-stu-id="fdd39-853">We've update our package to depend on version 2.0.0.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-854">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-854">Why</span></span>

<span data-ttu-id="fdd39-855">NetTopologySuite 2.0.0 版旨在解决 EF Core 用户遇到的几个可用性问题。</span><span class="sxs-lookup"><span data-stu-id="fdd39-855">Version 2.0.0 of NetTopologySuite aims to address several usability issues encountered by EF Core users.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-856">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-856">Mitigations</span></span>

<span data-ttu-id="fdd39-857">NetTopologySuite 2.0.0 版包括一些重大更改。</span><span class="sxs-lookup"><span data-stu-id="fdd39-857">NetTopologySuite version 2.0.0 includes some breaking changes.</span></span> <span data-ttu-id="fdd39-858">有关详细信息，请参阅[发行说明](https://www.nuget.org/packages/NetTopologySuite/2.0.0-pre001)。</span><span class="sxs-lookup"><span data-stu-id="fdd39-858">See the [release notes](https://www.nuget.org/packages/NetTopologySuite/2.0.0-pre001) for details.</span></span>

<a name="SqlClient"></a>

### <a name="microsoftdatasqlclient-is-used-instead-of-systemdatasqlclient"></a><span data-ttu-id="fdd39-859">使用 Microsoft.Data.SqlClient 而不是 System.Data.SqlClient</span><span class="sxs-lookup"><span data-stu-id="fdd39-859">Microsoft.Data.SqlClient is used instead of System.Data.SqlClient</span></span>

[<span data-ttu-id="fdd39-860">跟踪问题 #15636</span><span class="sxs-lookup"><span data-stu-id="fdd39-860">Tracking Issue #15636</span></span>](https://github.com/dotnet/efcore/issues/15636)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-861">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-861">Old behavior</span></span>

<span data-ttu-id="fdd39-862">Microsoft.EntityFrameworkCore.SqlServer 以前依赖 System.Data.SqlClient。</span><span class="sxs-lookup"><span data-stu-id="fdd39-862">Microsoft.EntityFrameworkCore.SqlServer previously depended on System.Data.SqlClient.</span></span>

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-863">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-863">New behavior</span></span>

<span data-ttu-id="fdd39-864">我们已将包更新为依赖 Microsoft.Data.SqlClient。</span><span class="sxs-lookup"><span data-stu-id="fdd39-864">We've updated our package to depend on Microsoft.Data.SqlClient.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-865">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-865">Why</span></span>

<span data-ttu-id="fdd39-866">Microsoft.Data.SqlClient 是今后用于 SQL Server 的旗舰版数据访问驱动程序。而 System.Data.SqlClient 不再是开发的重点。</span><span class="sxs-lookup"><span data-stu-id="fdd39-866">Microsoft.Data.SqlClient is the flagship data access driver for SQL Server going forward, and System.Data.SqlClient no longer be the focus of development.</span></span>
<span data-ttu-id="fdd39-867">一些重要功能（例如 Always Encrypted）仅可在 Microsoft.Data.SqlClient 上使用。</span><span class="sxs-lookup"><span data-stu-id="fdd39-867">Some important features, such as Always Encrypted, are only available on Microsoft.Data.SqlClient.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-868">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-868">Mitigations</span></span>

<span data-ttu-id="fdd39-869">如果你的代码直接依赖于 System.Data.SqlClient，则必须将其更改为 Microsoft.Data.SqlClient；由于这两个包与 API 的兼容性都非常高，因此这只是简单的包和命名空间更改。</span><span class="sxs-lookup"><span data-stu-id="fdd39-869">If your code takes a direct dependency on System.Data.SqlClient, you must change it to reference Microsoft.Data.SqlClient instead; as the two packages maintain a very high degree of API compatibility, this should only be a simple package and namespace change.</span></span>

<a name="mersa"></a>

### <a name="multiple-ambiguous-self-referencing-relationships-must-be-configured"></a><span data-ttu-id="fdd39-870">必须配置多个不明确的自引用关系</span><span class="sxs-lookup"><span data-stu-id="fdd39-870">Multiple ambiguous self-referencing relationships must be configured</span></span>

[<span data-ttu-id="fdd39-871">跟踪问题 #13573</span><span class="sxs-lookup"><span data-stu-id="fdd39-871">Tracking Issue #13573</span></span>](https://github.com/dotnet/efcore/issues/13573)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-872">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-872">Old behavior</span></span>

<span data-ttu-id="fdd39-873">具有多个自引用单向导航属性和匹配的 FK 的实体类型被错误配置为单个关系。</span><span class="sxs-lookup"><span data-stu-id="fdd39-873">An entity type with multiple self-referencing uni-directional navigation properties and matching FKs was incorrectly configured as a single relationship.</span></span> <span data-ttu-id="fdd39-874">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-874">For example:</span></span>

```csharp
public class User
{
        public Guid Id { get; set; }
        public User CreatedBy { get; set; }
        public User UpdatedBy { get; set; }
        public Guid CreatedById { get; set; }
        public Guid? UpdatedById { get; set; }
}
```

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-875">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-875">New behavior</span></span>

<span data-ttu-id="fdd39-876">现已在模型构建中检测到此场景，引发了一个指示模型不明确的异常。</span><span class="sxs-lookup"><span data-stu-id="fdd39-876">This scenario is now detected in model building and an exception is thrown indicating that the model is ambiguous.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-877">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-877">Why</span></span>

<span data-ttu-id="fdd39-878">生成的模型是不明确的，在这种情况下可能会出现错误。</span><span class="sxs-lookup"><span data-stu-id="fdd39-878">The resultant model was ambiguous and will likely usually be wrong for this case.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-879">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-879">Mitigations</span></span>

<span data-ttu-id="fdd39-880">使用关系的完全配置。</span><span class="sxs-lookup"><span data-stu-id="fdd39-880">Use full configuration of the relationship.</span></span> <span data-ttu-id="fdd39-881">例如：</span><span class="sxs-lookup"><span data-stu-id="fdd39-881">For example:</span></span>

```csharp
modelBuilder
     .Entity<User>()
     .HasOne(e => e.CreatedBy)
     .WithMany();

 modelBuilder
     .Entity<User>()
     .HasOne(e => e.UpdatedBy)
     .WithMany();
```

<a name="udf-empty-string"></a>

### <a name="dbfunctionschema-being-null-or-empty-string-configures-it-to-be-in-models-default-schema"></a><span data-ttu-id="fdd39-882">DbFunction.Schema 为 null 或者空字符串将其配置为位于模型的默认架构中</span><span class="sxs-lookup"><span data-stu-id="fdd39-882">DbFunction.Schema being null or empty string configures it to be in model's default schema</span></span>

[<span data-ttu-id="fdd39-883">跟踪问题 #12757</span><span class="sxs-lookup"><span data-stu-id="fdd39-883">Tracking Issue #12757</span></span>](https://github.com/dotnet/efcore/issues/12757)

#### <a name="old-behavior"></a><span data-ttu-id="fdd39-884">旧行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-884">Old behavior</span></span>

<span data-ttu-id="fdd39-885">配置了实为空字符串的架构的 DbFunction 被视为不带架构的内置函数。</span><span class="sxs-lookup"><span data-stu-id="fdd39-885">A DbFunction configured with schema as an empty string was treated as built-in function without a schema.</span></span> <span data-ttu-id="fdd39-886">例如，下述代码会将 `DatePart` CLR 函数映射到 SqlServer 上的 `DATEPART` 内置函数。</span><span class="sxs-lookup"><span data-stu-id="fdd39-886">For example following code will map `DatePart` CLR function to `DATEPART` built-in function on SqlServer.</span></span>

```csharp
[DbFunction("DATEPART", Schema = "")]
public static int? DatePart(string datePartArg, DateTime? date) => throw new Exception();

```

#### <a name="new-behavior"></a><span data-ttu-id="fdd39-887">新行为</span><span class="sxs-lookup"><span data-stu-id="fdd39-887">New behavior</span></span>

<span data-ttu-id="fdd39-888">所有 DbFunction 映射都被视为映射到用户定义的函数。</span><span class="sxs-lookup"><span data-stu-id="fdd39-888">All DbFunction mappings are considered to be mapped to user defined functions.</span></span> <span data-ttu-id="fdd39-889">因此，空字符串值会将函数置于模型的默认架构中。</span><span class="sxs-lookup"><span data-stu-id="fdd39-889">Hence empty string value would put the function inside the default schema for the model.</span></span> <span data-ttu-id="fdd39-890">这可能是通过 Fluent API `modelBuilder.HasDefaultSchema()` 显式配置的架构，否则为 `dbo`。</span><span class="sxs-lookup"><span data-stu-id="fdd39-890">Which could be the schema configured explicitly via fluent API `modelBuilder.HasDefaultSchema()` or `dbo` otherwise.</span></span>

#### <a name="why"></a><span data-ttu-id="fdd39-891">原因</span><span class="sxs-lookup"><span data-stu-id="fdd39-891">Why</span></span>

<span data-ttu-id="fdd39-892">如果之前的架构为空，可以此将函数视为内置项，但此逻辑仅适用于内置函数不属于任何架构的 SqlServer。</span><span class="sxs-lookup"><span data-stu-id="fdd39-892">Previously schema being empty was a way to treat that function is built-in but that logic is only applicable for SqlServer where built-in functions do not belong to any schema.</span></span>

#### <a name="mitigations"></a><span data-ttu-id="fdd39-893">缓解措施</span><span class="sxs-lookup"><span data-stu-id="fdd39-893">Mitigations</span></span>

<span data-ttu-id="fdd39-894">手动配置 DbFunction 的转换，以将其映射到内置函数中。</span><span class="sxs-lookup"><span data-stu-id="fdd39-894">Configure DbFunction's translation manually to map it to a built-in function.</span></span>

```csharp
modelBuilder
    .HasDbFunction(typeof(MyContext).GetMethod(nameof(MyContext.DatePart)))
    .HasTranslation(args => SqlFunctionExpression.Create("DatePart", args, typeof(int?), null));
```

<a name="netstandard21"></a>

### <a name="ef-core-30-targets-net-standard-21-rather-than-net-standard-20-reverted"></a><span data-ttu-id="fdd39-895">~~EF Core 3.0 面向 .NET Standard 2.1，而不是 .NET Standard 2.0~~ 已还原</span><span class="sxs-lookup"><span data-stu-id="fdd39-895">~~EF Core 3.0 targets .NET Standard 2.1 rather than .NET Standard 2.0~~ Reverted</span></span>

[<span data-ttu-id="fdd39-896">跟踪问题 #15498</span><span class="sxs-lookup"><span data-stu-id="fdd39-896">Tracking Issue #15498</span></span>](https://github.com/dotnet/efcore/issues/15498)

<span data-ttu-id="fdd39-897">EF Core 3.0 面向 .NET Standard 2.1，这是一项重大更改，因为它不包括 .NET Framework 应用程序。</span><span class="sxs-lookup"><span data-stu-id="fdd39-897">EF Core 3.0 targets .NET Standard 2.1, which is a breaking change which excludes .NET Framework applications.</span></span> <span data-ttu-id="fdd39-898">EF Core 3.1 还原了此更改，再次面向 .NET Standard 2.0。</span><span class="sxs-lookup"><span data-stu-id="fdd39-898">EF Core 3.1 reverted this and targets .NET Standard 2.0 again.</span></span>

<a name="qe"></a>

### <a name="query-execution-is-logged-at-debug-level-reverted"></a><span data-ttu-id="fdd39-899">~~在调试级别记录查询执行~~已还原</span><span class="sxs-lookup"><span data-stu-id="fdd39-899">~~Query execution is logged at Debug level~~ Reverted</span></span>

[<span data-ttu-id="fdd39-900">跟踪问题 #14523</span><span class="sxs-lookup"><span data-stu-id="fdd39-900">Tracking Issue #14523</span></span>](https://github.com/dotnet/efcore/issues/14523)

<span data-ttu-id="fdd39-901">我们之所以还原此更改是因为，EF Core 3.0 中的新配置允许应用程序指定任何事件的日志级别。</span><span class="sxs-lookup"><span data-stu-id="fdd39-901">We reverted this change because new configuration in EF Core 3.0 allows the log level for any event to be specified by the application.</span></span> <span data-ttu-id="fdd39-902">例如，若要将 SQL 日志记录切换为 `Debug`，请在 `OnConfiguring` 或 `AddDbContext` 中显式配置级别：</span><span class="sxs-lookup"><span data-stu-id="fdd39-902">For example, to switch logging of SQL to `Debug`, explicitly configure the level in `OnConfiguring` or `AddDbContext`:</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .UseSqlServer(connectionString)
        .ConfigureWarnings(c => c.Log((RelationalEventId.CommandExecuting, LogLevel.Debug)));
```
