---
title: 针对 Entity Framework Core 6.0 的计划
description: 为 EF Core 6.0 计划的主题和功能
author: ajcvickers
ms.date: 01/12/2021
uid: core/what-is-new/ef-core-6.0/plan
ms.openlocfilehash: 612461bc6ad30778baa5c6d10dda5cabac91dcb2
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129533"
---
# <a name="plan-for-entity-framework-core-60"></a><span data-ttu-id="36987-103">针对 Entity Framework Core 6.0 的计划</span><span class="sxs-lookup"><span data-stu-id="36987-103">Plan for Entity Framework Core 6.0</span></span>

<span data-ttu-id="36987-104">如[计划过程](xref:core/what-is-new/release-planning)中所述，我们已将利益干系人输入的内容收集到针对 Entity Framework Core (EF Core) 6.0 版的计划中。</span><span class="sxs-lookup"><span data-stu-id="36987-104">As described in the [planning process](xref:core/what-is-new/release-planning), we have gathered input from stakeholders into a plan for the Entity Framework Core (EF Core) 6.0 release.</span></span>

<span data-ttu-id="36987-105">与以前的版本不同，此计划不会尝试覆盖 6.0 版的各项工作。</span><span class="sxs-lookup"><span data-stu-id="36987-105">Unlike previous releases, this plan does not attempt to cover all work for the 6.0 release.</span></span> <span data-ttu-id="36987-106">相反，它指出了我们打算在此版本中投入的方面和方式，而在我们一边使用该版本一边收集反馈和进行学习时，可灵活地调整范围或引入新工作。</span><span class="sxs-lookup"><span data-stu-id="36987-106">Instead, it indicates where and how we intend to invest in this release, but with flexibility to adjust scope or pull in new work as we gather feedback and learn while working on the release.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="36987-107">此计划不是承诺。</span><span class="sxs-lookup"><span data-stu-id="36987-107">This plan is not a commitment.</span></span> <span data-ttu-id="36987-108">它是一个起点，会随着我们了解更多信息而发展。</span><span class="sxs-lookup"><span data-stu-id="36987-108">It is a starting point that will evolve as we learn more.</span></span> <span data-ttu-id="36987-109">其中可能会纳入当前未针对 6.0 计划的某些内容。</span><span class="sxs-lookup"><span data-stu-id="36987-109">Some things not currently planned for 6.0 may get pulled in.</span></span> <span data-ttu-id="36987-110">而当前已针对 6.0 计划的某些内容可能会被淘汰。</span><span class="sxs-lookup"><span data-stu-id="36987-110">Some things currently planned for 6.0 may get punted out.</span></span>

## <a name="general-information"></a><span data-ttu-id="36987-111">常规信息</span><span class="sxs-lookup"><span data-stu-id="36987-111">General information</span></span>

### <a name="version-number-and-release-date"></a><span data-ttu-id="36987-112">版本号和发布日期</span><span class="sxs-lookup"><span data-stu-id="36987-112">Version number and release date</span></span>

<span data-ttu-id="36987-113">EF Core 6.0 是 EF Core 5.0 之后的下一版本，目前计划在 2021 年 11 月与 .NET 6 同时发布。</span><span class="sxs-lookup"><span data-stu-id="36987-113">EF Core 6.0 is the next release after EF Core 5.0 and is currently scheduled for release in November 2021 at the same time as .NET 6.</span></span>

### <a name="supported-platforms"></a><span data-ttu-id="36987-114">受支持的平台</span><span class="sxs-lookup"><span data-stu-id="36987-114">Supported platforms</span></span>

<span data-ttu-id="36987-115">EF Core 6.0 当前面向 .NET 5。</span><span class="sxs-lookup"><span data-stu-id="36987-115">EF Core 6.0 currently targets .NET 5.</span></span> <span data-ttu-id="36987-116">在我们快要发布时，这可能会更新到 .NET 6。</span><span class="sxs-lookup"><span data-stu-id="36987-116">This will likely be updated to .NET 6 as we near the release.</span></span> <span data-ttu-id="36987-117">EF Core 6.0 不面向任何 .NET Standard 版本；有关详细信息，请参阅 [.NET Standard 的未来](https://devblogs.microsoft.com/dotnet/the-future-of-net-standard/)。</span><span class="sxs-lookup"><span data-stu-id="36987-117">EF Core 6.0 does not target any .NET Standard version; for more information see [the future of .NET Standard](https://devblogs.microsoft.com/dotnet/the-future-of-net-standard/).</span></span>

<span data-ttu-id="36987-118">EF Core 6.0 不会在 .NET Framework 上运行。</span><span class="sxs-lookup"><span data-stu-id="36987-118">EF Core 6.0 will not run on .NET Framework.</span></span>

<span data-ttu-id="36987-119">EF Core 6.0 将作为[长期支持 (LTS) 版本](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)与 .NET 6 保持一致。</span><span class="sxs-lookup"><span data-stu-id="36987-119">EF Core 6.0 will align with .NET 6 as a [long-term support (LTS) release](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).</span></span>

### <a name="breaking-changes"></a><span data-ttu-id="36987-120">中断性变更</span><span class="sxs-lookup"><span data-stu-id="36987-120">Breaking changes</span></span>

<span data-ttu-id="36987-121">我们将继续改进 EF Core 和 .NET 平台，因此 EF Core 6.0 将包含少量[中断性变更](xref:core/what-is-new/ef-core-6.0/breaking-changes)。</span><span class="sxs-lookup"><span data-stu-id="36987-121">EF Core 6.0 will contain a small number of [breaking changes](xref:core/what-is-new/ef-core-6.0/breaking-changes) as we continue to evolve both EF Core and the .NET platform.</span></span> <span data-ttu-id="36987-122">我们的目标是允许大多数应用程序进行更新而不会中断。</span><span class="sxs-lookup"><span data-stu-id="36987-122">Our goal is to allow the vast majority of applications to update without breaking.</span></span>

## <a name="themes"></a><span data-ttu-id="36987-123">主题</span><span class="sxs-lookup"><span data-stu-id="36987-123">Themes</span></span>

<span data-ttu-id="36987-124">以下方面将构成 EF Core 6.0 中多数投资的基础。</span><span class="sxs-lookup"><span data-stu-id="36987-124">The following areas will form the basis for the large investments in EF Core 6.0.</span></span>

## <a name="highly-requested-features"></a><span data-ttu-id="36987-125">被强烈要求的功能</span><span class="sxs-lookup"><span data-stu-id="36987-125">Highly requested features</span></span>

<span data-ttu-id="36987-126">与往常一样，[计划过程](xref:core/what-is-new/release-planning)中的主要输入内容来自[对 GitHub 上功能的投票 (👍)](https://github.com/dotnet/efcore/issues?q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc)。</span><span class="sxs-lookup"><span data-stu-id="36987-126">As always, a major input into the [planning process](xref:core/what-is-new/release-planning) comes from the [voting (👍) for features on GitHub](https://github.com/dotnet/efcore/issues?q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc).</span></span> <span data-ttu-id="36987-127">对于 EF Core 6.0，我们计划处理以下被强烈要求的功能：</span><span class="sxs-lookup"><span data-stu-id="36987-127">For EF Core 6.0 we plan to work on the following highly requested features:</span></span>

### <a name="sql-server-temporal-tables"></a><span data-ttu-id="36987-128">SQL Server 临时表</span><span class="sxs-lookup"><span data-stu-id="36987-128">SQL Server temporal tables</span></span>

<span data-ttu-id="36987-129">通过 [#4693](https://github.com/dotnet/efcore/issues/4693) 进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-129">Tracked by [#4693](https://github.com/dotnet/efcore/issues/4693)</span></span>

<span data-ttu-id="36987-130">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-130">Status: Not started</span></span>

<span data-ttu-id="36987-131">T 恤大小：大</span><span class="sxs-lookup"><span data-stu-id="36987-131">T-shirt size: Large</span></span>

<span data-ttu-id="36987-132">临时表支持查询在任何时间点存储在表中的数据，而不是像普通表那样仅限存储的最新数据。</span><span class="sxs-lookup"><span data-stu-id="36987-132">Temporal tables support queries for data stored in the table at _any point in time_, as opposed to only the most recent data stored as is the case for normal tables.</span></span> <span data-ttu-id="36987-133">EF Core 6.0 将允许通过迁移来创建临时表，并允许通过 LINQ 查询访问数据。</span><span class="sxs-lookup"><span data-stu-id="36987-133">EF Core 6.0 will allow temporal tables to be created via Migrations, as well as allowing access to the data through LINQ queries.</span></span>

<span data-ttu-id="36987-134">这项工作最初作用的范围[如问题所述](https://github.com/dotnet/efcore/issues/4693#issuecomment-625048974)。</span><span class="sxs-lookup"><span data-stu-id="36987-134">This work is initially scoped as [described on the issue](https://github.com/dotnet/efcore/issues/4693#issuecomment-625048974).</span></span> <span data-ttu-id="36987-135">我们可能会根据发布期间的反馈引入其他支持。</span><span class="sxs-lookup"><span data-stu-id="36987-135">We may pull in additional support based on feedback during the release.</span></span>

### <a name="json-columns"></a><span data-ttu-id="36987-136">JSON 列</span><span class="sxs-lookup"><span data-stu-id="36987-136">JSON columns</span></span>

<span data-ttu-id="36987-137">通过 [#4021](https://github.com/dotnet/efcore/issues/4021) 进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-137">Tracked by [#4021](https://github.com/dotnet/efcore/issues/4021)</span></span>

<span data-ttu-id="36987-138">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-138">Status: Not started</span></span>

<span data-ttu-id="36987-139">T 恤大小：中等</span><span class="sxs-lookup"><span data-stu-id="36987-139">T-shirt size: Medium</span></span>

<span data-ttu-id="36987-140">此功能将为 JSON 支持引入可由任何数据库提供程序实现的通用机制和模式。</span><span class="sxs-lookup"><span data-stu-id="36987-140">This feature will introduce a common mechanism and patterns for JSON support that can be implemented by any database provider.</span></span> <span data-ttu-id="36987-141">我们将与社区合作来调整 [Npgsql](https://github.com/npgsql/efcore.pg) 和 [Pomelo MySQL](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql) 的现有实现，并添加对 SQL Server 和 SQLite 的支持。</span><span class="sxs-lookup"><span data-stu-id="36987-141">We will work with the community to align existing implementations for [Npgsql](https://github.com/npgsql/efcore.pg) and [Pomelo MySQL](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql), and also add support for SQL Server and SQLite.</span></span>

### <a name="columnattributeorder"></a><span data-ttu-id="36987-142">ColumnAttribute.Order</span><span class="sxs-lookup"><span data-stu-id="36987-142">ColumnAttribute.Order</span></span>

<span data-ttu-id="36987-143">通过 [#10059](https://github.com/dotnet/efcore/issues/10059) 进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-143">Tracked by [#10059](https://github.com/dotnet/efcore/issues/10059)</span></span>

<span data-ttu-id="36987-144">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-144">Status: Not started</span></span>

<span data-ttu-id="36987-145">T 恤大小：小</span><span class="sxs-lookup"><span data-stu-id="36987-145">T-shirt size: Small</span></span>

<span data-ttu-id="36987-146">此功能将允许在使用迁移或 `EnsureCreated` 创建表时对列进行任意排序。</span><span class="sxs-lookup"><span data-stu-id="36987-146">This feature will allow arbitrary ordering of columns when **creating a table** with Migrations or `EnsureCreated`.</span></span> <span data-ttu-id="36987-147">请注意，更改现有表中列的顺序需要重新生成表，而我们并不计划在任何 EF Core 版本中支持这项功能。</span><span class="sxs-lookup"><span data-stu-id="36987-147">Note that changing the order of columns in an existing tables requires that the table be rebuilt, and this is not something that we plan to support in any EF Core release.</span></span>

## <a name="performance"></a><span data-ttu-id="36987-148">性能</span><span class="sxs-lookup"><span data-stu-id="36987-148">Performance</span></span>

<span data-ttu-id="36987-149">虽然 EF Core 的速度通常比 EF6 更快，但某些方面仍存在大幅提升性能的空间。</span><span class="sxs-lookup"><span data-stu-id="36987-149">While EF Core is generally faster than EF6, there are still areas where significant improvements in performance are possible.</span></span> <span data-ttu-id="36987-150">我们计划在 EF Core 6.0 中处理其中的几个方面，同时改进我们的性能基础结构和测试。</span><span class="sxs-lookup"><span data-stu-id="36987-150">We plan to tackle several of these areas in EF Core 6.0, while also improving our perf infrastructure and testing.</span></span>

<span data-ttu-id="36987-151">本主题将涉及大量迭代调查，它将告知我们要将资源集中到哪些方面。</span><span class="sxs-lookup"><span data-stu-id="36987-151">This theme will involve a lot of iterative investigation, which will inform where we focus resources.</span></span> <span data-ttu-id="36987-152">我们计划从以下方面开始：</span><span class="sxs-lookup"><span data-stu-id="36987-152">We plan to begin with:</span></span>

### <a name="performance-infrastructure-and-new-tests"></a><span data-ttu-id="36987-153">性能基础结构和新测试</span><span class="sxs-lookup"><span data-stu-id="36987-153">Performance infrastructure and new tests</span></span>

<span data-ttu-id="36987-154">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-154">Status: Not started</span></span>

<span data-ttu-id="36987-155">T 恤大小：中等</span><span class="sxs-lookup"><span data-stu-id="36987-155">T-shirt size: Medium</span></span>

<span data-ttu-id="36987-156">EF Core 代码库已包含一组性能基准，它们每日都会执行。</span><span class="sxs-lookup"><span data-stu-id="36987-156">The EF Core codebase already contains a set of performance benchmarks that are executed every day.</span></span> <span data-ttu-id="36987-157">对于 6.0 版，我们计划改进这些测试的基础结构并添加新测试。</span><span class="sxs-lookup"><span data-stu-id="36987-157">For 6.0, we plan to improve the infrastructure for these tests as well as adding new tests.</span></span> <span data-ttu-id="36987-158">我们还将分析主线性能方案，并修复发现的任何可轻松解决的问题。</span><span class="sxs-lookup"><span data-stu-id="36987-158">We will also profile mainline perf scenarios and fix any low-hanging fruit found.</span></span>

### <a name="compiled-models"></a><span data-ttu-id="36987-159">已编译的模型</span><span class="sxs-lookup"><span data-stu-id="36987-159">Compiled models</span></span>

<span data-ttu-id="36987-160">通过 [#1906](https://github.com/dotnet/efcore/issues/1906) 进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-160">Tracked by [#1906](https://github.com/dotnet/efcore/issues/1906)</span></span>

<span data-ttu-id="36987-161">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-161">Status: Not started</span></span>

<span data-ttu-id="36987-162">T 恤大小：加大</span><span class="sxs-lookup"><span data-stu-id="36987-162">T-shirt size: X-Large</span></span>

<span data-ttu-id="36987-163">已编译的模型将允许生成 EF 模型的编译形式。</span><span class="sxs-lookup"><span data-stu-id="36987-163">Compiled models will allow the generation of a compiled form of the EF model.</span></span> <span data-ttu-id="36987-164">这将提供更好的启动性能，通常还在访问模型时提供更好的性能。</span><span class="sxs-lookup"><span data-stu-id="36987-164">This will provide both better startup performance, as well as generally better performance when accessing the model.</span></span>

### <a name="techempower-fortunes"></a><span data-ttu-id="36987-165">TechEmpower Fortunes</span><span class="sxs-lookup"><span data-stu-id="36987-165">TechEmpower Fortunes</span></span>

<span data-ttu-id="36987-166">通过 [#23611](https://github.com/dotnet/efcore/issues/23611) 进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-166">Tracked by [#23611](https://github.com/dotnet/efcore/issues/23611)</span></span>

<span data-ttu-id="36987-167">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-167">Status: Not started</span></span>

<span data-ttu-id="36987-168">T 恤大小：加大</span><span class="sxs-lookup"><span data-stu-id="36987-168">T-shirt size: X-Large</span></span>

<span data-ttu-id="36987-169">几年来，我们一直在 .NET 上针对 PostgreSQL 数据库运行行业标准的 [TechEmpower 基准](https://www.techempower.com/benchmarks/)。</span><span class="sxs-lookup"><span data-stu-id="36987-169">We have been running the industry standard [TechEmpower benchmarks](https://www.techempower.com/benchmarks/) on .NET against a PostgreSQL database for several years.</span></span> <span data-ttu-id="36987-170">[Fortunes 基准](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=fortune) 与 EF 方案尤为相关。</span><span class="sxs-lookup"><span data-stu-id="36987-170">The [Fortunes benchmark](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=fortune) is particularly relevant to EF scenarios.</span></span> <span data-ttu-id="36987-171">我们有此基准的多个变体，包括：</span><span class="sxs-lookup"><span data-stu-id="36987-171">We have multiple variations of this benchmark, including:</span></span>

- <span data-ttu-id="36987-172">直接使用 ADO.NET 的实现。</span><span class="sxs-lookup"><span data-stu-id="36987-172">An implementation that uses ADO.NET directly.</span></span> <span data-ttu-id="36987-173">这是此处列出的三个实现中最快的一个。</span><span class="sxs-lookup"><span data-stu-id="36987-173">This is the fastest implementation of the three listed here.</span></span>
- <span data-ttu-id="36987-174">使用 [Dapper](https://www.nuget.org/packages/Dapper/) 的实现。</span><span class="sxs-lookup"><span data-stu-id="36987-174">An implementation that uses [Dapper](https://www.nuget.org/packages/Dapper/).</span></span> <span data-ttu-id="36987-175">与直接使用 ADO.NET 相比，该实现更慢，但仍然很快。</span><span class="sxs-lookup"><span data-stu-id="36987-175">This is slower than using ADO.NET directly, but still fast.</span></span>
- <span data-ttu-id="36987-176">使用 EF Core 的实现。</span><span class="sxs-lookup"><span data-stu-id="36987-176">An implementation that uses EF Core.</span></span> <span data-ttu-id="36987-177">这是当前三个实现中最慢的一个。</span><span class="sxs-lookup"><span data-stu-id="36987-177">This is currently the slowest implementation of the three.</span></span>

<span data-ttu-id="36987-178">EF Core 6.0 的目标是使 EF Core 的性能与 TechEmpower Fortunes 基准上的 Dapper 不相上下。</span><span class="sxs-lookup"><span data-stu-id="36987-178">The goal for EF Core 6.0 is to get the EF Core performance to match that of Dapper on the TechEmpower Fortunes benchmark.</span></span> <span data-ttu-id="36987-179">（这是一项重大挑战，但我们将尽最大努力尽量实现。）</span><span class="sxs-lookup"><span data-stu-id="36987-179">(This is a significant challenge but we will do our best to get as close as we can.)</span></span>

### <a name="linkeraot"></a><span data-ttu-id="36987-180">链接器/AOT</span><span class="sxs-lookup"><span data-stu-id="36987-180">Linker/AOT</span></span>

<span data-ttu-id="36987-181">通过 [#10963](https://github.com/dotnet/efcore/issues/10963) 进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-181">Tracked by [#10963](https://github.com/dotnet/efcore/issues/10963)</span></span>

<span data-ttu-id="36987-182">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-182">Status: Not started</span></span>

<span data-ttu-id="36987-183">T 恤大小：中等</span><span class="sxs-lookup"><span data-stu-id="36987-183">T-shirt size: Medium</span></span>

<span data-ttu-id="36987-184">EF Core 执行大量的运行时代码生成。</span><span class="sxs-lookup"><span data-stu-id="36987-184">EF Core performs large amounts of runtime code generation.</span></span> <span data-ttu-id="36987-185">对于依赖链接器树握手（如 Xamarin 和 Blazor）的应用模型以及不允许动态编译的平台（如 iOS）来说，这非常困难。</span><span class="sxs-lookup"><span data-stu-id="36987-185">This is challenging for app models that depend on linker tree shaking, such as Xamarin and Blazor, and platforms that don't allow dynamic compilation, such as iOS.</span></span> <span data-ttu-id="36987-186">作为 EF Core 6.0 的一部分，我们将继续调查该方面，并尽可能进行有针对性的改进。</span><span class="sxs-lookup"><span data-stu-id="36987-186">We will continue investigating in this space as part of EF Core 6.0 and make targeted improvements as we can.</span></span> <span data-ttu-id="36987-187">不过，我们并不期望在 6.0 版期限内完全消除差距。</span><span class="sxs-lookup"><span data-stu-id="36987-187">However, we do not expect to fully close the gap in the 6.0 time frame.</span></span>

## <a name="migrations-and-deployment"></a><span data-ttu-id="36987-188">迁移和部署</span><span class="sxs-lookup"><span data-stu-id="36987-188">Migrations and deployment</span></span>

<span data-ttu-id="36987-189">根据对 [EF Core 5.0 的调查](xref:core/what-is-new/ef-core-5.0/plan#migrations-and-deployment-experience)，我们计划引入对管理迁移和部署数据库的改进支持。</span><span class="sxs-lookup"><span data-stu-id="36987-189">Following on from the [investigations done for EF Core 5.0](xref:core/what-is-new/ef-core-5.0/plan#migrations-and-deployment-experience), we plan to introduce improved support for managing migrations and deploying databases.</span></span> <span data-ttu-id="36987-190">这包括两个主要方面：</span><span class="sxs-lookup"><span data-stu-id="36987-190">This includes two major areas:</span></span>

### <a name="migrations-bundles"></a><span data-ttu-id="36987-191">迁移捆绑包</span><span class="sxs-lookup"><span data-stu-id="36987-191">Migrations bundles</span></span>

<span data-ttu-id="36987-192">通过 [#19693](https://github.com/dotnet/efcore/issues/19693) 进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-192">Tracked by [#19693](https://github.com/dotnet/efcore/issues/19693)</span></span>

<span data-ttu-id="36987-193">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-193">Status: Not started</span></span>

<span data-ttu-id="36987-194">T 恤大小：中等</span><span class="sxs-lookup"><span data-stu-id="36987-194">T-shirt size: Medium</span></span>

<span data-ttu-id="36987-195">迁移捆绑包是一种独立的可执行文件，用于将迁移应用到生产数据库。</span><span class="sxs-lookup"><span data-stu-id="36987-195">A migrations bundle is a self-contained executable that applies migrations to a production database.</span></span> <span data-ttu-id="36987-196">此行为将与 `dotnet ef database update` 匹配，但由于所需的所有内容都包含在单个可执行文件中，因此它会使 SSH/Docker/PowerShell 部署更容易。</span><span class="sxs-lookup"><span data-stu-id="36987-196">The behavior will match `dotnet ef database update`, but should make SSH/Docker/PowerShell deployment much easier, since everything needed is contained in the single executable.</span></span>

### <a name="managing-migrations"></a><span data-ttu-id="36987-197">管理迁移</span><span class="sxs-lookup"><span data-stu-id="36987-197">Managing migrations</span></span>

<span data-ttu-id="36987-198">通过 [#22945](https://github.com/dotnet/efcore/issues/22945) 进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-198">Tracked by [#22945](https://github.com/dotnet/efcore/issues/22945)</span></span>

<span data-ttu-id="36987-199">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-199">Status: Not started</span></span>

<span data-ttu-id="36987-200">T 恤大小：大</span><span class="sxs-lookup"><span data-stu-id="36987-200">T-shirt size: Large</span></span>

<span data-ttu-id="36987-201">为应用程序创建的迁移数可能会增加，从而成为负担。</span><span class="sxs-lookup"><span data-stu-id="36987-201">The number of migrations created for an application can grow to become a burden.</span></span> <span data-ttu-id="36987-202">此外，即使不需要，这些迁移也经常与应用程序一起部署。</span><span class="sxs-lookup"><span data-stu-id="36987-202">In addition, these migrations are frequently deployed with the application even when this is not needed.</span></span> <span data-ttu-id="36987-203">在 EF Core 6.0 中，我们计划通过更好的工具和项目/程序集管理来改进这一点。</span><span class="sxs-lookup"><span data-stu-id="36987-203">In EF Core 6.0, we plan to improve this through better tooling and project/assembly management.</span></span> <span data-ttu-id="36987-204">我们计划解决的两个具体问题是[将多个迁移压缩为一个迁移](https://github.com/dotnet/efcore/issues/2174)和[重新生成干净的模型快照](https://github.com/dotnet/efcore/issues/18557)。</span><span class="sxs-lookup"><span data-stu-id="36987-204">Two specific issues we plan to address are [squash many migrations into one](https://github.com/dotnet/efcore/issues/2174) and [regenerate a clean model snapshot](https://github.com/dotnet/efcore/issues/18557).</span></span>

## <a name="improve-existing-features-and-fix-bugs"></a><span data-ttu-id="36987-205">改进现有功能并修复 bug</span><span class="sxs-lookup"><span data-stu-id="36987-205">Improve existing features and fix bugs</span></span>

<span data-ttu-id="36987-206">[分配给 6.0.0 里程碑的任何问题或 bug](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0) 目前都计划在此版本中解决。</span><span class="sxs-lookup"><span data-stu-id="36987-206">Any [issue or bug assigned to the 6.0.0 milestone](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0) is currently planned for this release.</span></span> <span data-ttu-id="36987-207">这包括许多小型改进和 bug 修复。</span><span class="sxs-lookup"><span data-stu-id="36987-207">This includes many small enhancements and bug fixes.</span></span>

### <a name="ef6-query-parity"></a><span data-ttu-id="36987-208">EF6 查询奇偶校验</span><span class="sxs-lookup"><span data-stu-id="36987-208">EF6 query parity</span></span>

<span data-ttu-id="36987-209">通过[使用“ef6-parity”标记的问题和 6.0 里程碑中的问题](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+label%3Aef6-parity+milestone%3A6.0.0)进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-209">Tracked by [issues labeled with 'ef6-parity' and in the 6.0 milestone](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+label%3Aef6-parity+milestone%3A6.0.0)</span></span>

<span data-ttu-id="36987-210">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-210">Status: Not started</span></span>

<span data-ttu-id="36987-211">T 恤大小：大</span><span class="sxs-lookup"><span data-stu-id="36987-211">T-shirt size: Large</span></span>

<span data-ttu-id="36987-212">EF Core 5.0 支持 EF6 支持的大多数查询模式，也支持 EF6 不支持的模式。</span><span class="sxs-lookup"><span data-stu-id="36987-212">EF Core 5.0 supports most query patterns supported by EF6, in addition to patterns not supported in EF6.</span></span> <span data-ttu-id="36987-213">对于 EF Core 6.0，我们计划缩小差距，使受支持的 EF Core 查询成为受支持的 EF6 查询的真正超集。</span><span class="sxs-lookup"><span data-stu-id="36987-213">For EF Core 6.0, we plan to close the gap and make supported EF Core queries a true superset of supported EF6 queries.</span></span> <span data-ttu-id="36987-214">这将由对差异的调查来推动，但已包含 GroupBy 问题（例如[转换后接 FirstOrDefault 的 GroupBy](https://github.com/dotnet/efcore/issues/12088)），以及针对[基元](https://github.com/dotnet/efcore/issues/11624)和[未映射](https://github.com/dotnet/efcore/issues/10753)类型的原始 SQL 查询。</span><span class="sxs-lookup"><span data-stu-id="36987-214">This will be driven by investigation of the gaps, but already includes GroupBy issues such as [translate GroupBy followed by FirstOrDefault](https://github.com/dotnet/efcore/issues/12088), and raw SQL queries for [primitive](https://github.com/dotnet/efcore/issues/11624) and [unmapped](https://github.com/dotnet/efcore/issues/10753) types.</span></span>  

### <a name="value-objects"></a><span data-ttu-id="36987-215">值对象</span><span class="sxs-lookup"><span data-stu-id="36987-215">Value objects</span></span>

<span data-ttu-id="36987-216">通过 [#9906](https://github.com/dotnet/efcore/issues/9906) 进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-216">Tracked by [#9906](https://github.com/dotnet/efcore/issues/9906)</span></span>

<span data-ttu-id="36987-217">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-217">Status: Not started</span></span>

<span data-ttu-id="36987-218">T 恤大小：中等</span><span class="sxs-lookup"><span data-stu-id="36987-218">T-shirt size: Medium</span></span>

<span data-ttu-id="36987-219">以前，拥有实体的团队视图（旨在[提供聚合支持](https://www.martinfowler.com/bliki/DDD_Aggregate.html)）也是[值对象](https://www.martinfowler.com/bliki/ValueObject.html)的合理近似值。</span><span class="sxs-lookup"><span data-stu-id="36987-219">It was previously the team view that owned entities, intended for [aggregate support](https://www.martinfowler.com/bliki/DDD_Aggregate.html), would also be a reasonable approximation to [value objects](https://www.martinfowler.com/bliki/ValueObject.html).</span></span> <span data-ttu-id="36987-220">经验表明事实并非如此。</span><span class="sxs-lookup"><span data-stu-id="36987-220">Experience has shown this not to be the case.</span></span> <span data-ttu-id="36987-221">因此，我们计划在 EF Core 6.0 引入一种更好的体验，重点关注域驱动设计中值对象的需求。</span><span class="sxs-lookup"><span data-stu-id="36987-221">Therefore, in EF Core 6.0 we plan to introduce a better experience focused on the needs of value objects in domain-driven design.</span></span> <span data-ttu-id="36987-222">此方法将基于值转换器而不是拥有的实体。</span><span class="sxs-lookup"><span data-stu-id="36987-222">This approach will be based on value converters rather than owned entities.</span></span>

<span data-ttu-id="36987-223">这项工作最初作用的范围是允许[映射到多个列的值转换器](https://github.com/dotnet/efcore/issues/13947)。</span><span class="sxs-lookup"><span data-stu-id="36987-223">This work is initially scoped to allow [value converters which map to multiple columns](https://github.com/dotnet/efcore/issues/13947).</span></span> <span data-ttu-id="36987-224">我们可能会根据发布期间的反馈引入其他支持。</span><span class="sxs-lookup"><span data-stu-id="36987-224">We may pull in additional support based on feedback during the release.</span></span>

### <a name="cosmos-database-provider"></a><span data-ttu-id="36987-225">Cosmos 数据库提供程序</span><span class="sxs-lookup"><span data-stu-id="36987-225">Cosmos database provider</span></span>

<span data-ttu-id="36987-226">通过[使用“area-cosmos”标记的问题和 6.0 里程碑中的问题](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0+label%3Aarea-cosmos)进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-226">Tracked by [issues labeled with 'area-cosmos' and in the 6.0 milestone](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0+label%3Aarea-cosmos)</span></span>

<span data-ttu-id="36987-227">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-227">Status: Not started</span></span>

<span data-ttu-id="36987-228">T 恤大小：大</span><span class="sxs-lookup"><span data-stu-id="36987-228">T-shirt size: Large</span></span>

<span data-ttu-id="36987-229">我们正在积极收集有关对 EF Core 6.0 中的 Cosmos 提供程序进行哪些改进的反馈。</span><span class="sxs-lookup"><span data-stu-id="36987-229">We are actively gathering feedback on which improvements to make to the Cosmos provider in EF Core 6.0.</span></span> <span data-ttu-id="36987-230">我们会在了解更多信息后更新此文档。</span><span class="sxs-lookup"><span data-stu-id="36987-230">We will update this document as we learn more.</span></span> <span data-ttu-id="36987-231">现在，请务必为你需要的 Cosmos 功能投票 (👍)。</span><span class="sxs-lookup"><span data-stu-id="36987-231">For now, please make sure to vote (👍) for the Cosmos features that you need.</span></span>

### <a name="expose-model-building-conventions-to-applications"></a><span data-ttu-id="36987-232">向应用程序公开模型构建约定</span><span class="sxs-lookup"><span data-stu-id="36987-232">Expose model building conventions to applications</span></span>

<span data-ttu-id="36987-233">通过 [#214](https://github.com/dotnet/efcore/issues/214) 进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-233">Tracked by [#214](https://github.com/dotnet/efcore/issues/214)</span></span>

<span data-ttu-id="36987-234">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-234">Status: Not started</span></span>

<span data-ttu-id="36987-235">T 恤大小：中</span><span class="sxs-lookup"><span data-stu-id="36987-235">T-shirt size: medium</span></span>

<span data-ttu-id="36987-236">EF Core 使用一组约定来从 .NET 类型构建模型。</span><span class="sxs-lookup"><span data-stu-id="36987-236">EF Core uses a set of conventions for building a model from .NET types.</span></span> <span data-ttu-id="36987-237">这些约定当前由数据库提供程序控制。</span><span class="sxs-lookup"><span data-stu-id="36987-237">These conventions are currently controlled by the database provider.</span></span> <span data-ttu-id="36987-238">在 EF Core 6.0 中，我们打算允许应用程序与这些约定挂钩并更改这些约定。</span><span class="sxs-lookup"><span data-stu-id="36987-238">In EF Core 6.0, we intend to allow applications to hook into and change these conventions.</span></span>

### <a name="zero-bug-balance-zbb"></a><span data-ttu-id="36987-239">零 bug 平衡 (ZBB)</span><span class="sxs-lookup"><span data-stu-id="36987-239">Zero bug balance (ZBB)</span></span>

<span data-ttu-id="36987-240">通过 [6.0 里程碑中使用 `type-bug` 标记的问题](https://github.com/dotnet/efcore/issues?utf8=%E2%9C%93&q=is%3Aissue+milestone%3A6.0.0+label%3Atype-bug+)进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-240">Tracked by [issues labeled with `type-bug` in the 6.0 milestone](https://github.com/dotnet/efcore/issues?utf8=%E2%9C%93&q=is%3Aissue+milestone%3A6.0.0+label%3Atype-bug+)</span></span>

<span data-ttu-id="36987-241">状态：正在进行</span><span class="sxs-lookup"><span data-stu-id="36987-241">Status: In-progress</span></span>

<span data-ttu-id="36987-242">T 恤大小：大</span><span class="sxs-lookup"><span data-stu-id="36987-242">T-shirt size: Large</span></span>

<span data-ttu-id="36987-243">我们计划在 EF Core 6.0 版期限内修复所有待解决的 bug。</span><span class="sxs-lookup"><span data-stu-id="36987-243">We plan to fix all outstanding bugs during the EF Core 6.0 time frame.</span></span> <span data-ttu-id="36987-244">需谨记以下几点：</span><span class="sxs-lookup"><span data-stu-id="36987-244">Some things to keep in mind:</span></span>

- <span data-ttu-id="36987-245">这特别适用于标记了 [type-bug](https://github.com/dotnet/efcore/issues?q=is%3Aissue+label%3Atype-bug) 的问题。</span><span class="sxs-lookup"><span data-stu-id="36987-245">This specifically applies to issues labeled [type-bug](https://github.com/dotnet/efcore/issues?q=is%3Aissue+label%3Atype-bug).</span></span>
- <span data-ttu-id="36987-246">存在一些例外，例如当需要设计更改或新功能才能正确修复 bug 时。</span><span class="sxs-lookup"><span data-stu-id="36987-246">There will be exceptions, such as when the bug requires a design change or new feature to fix properly.</span></span> <span data-ttu-id="36987-247">这些问题将使用 `blocked` 标签进行标记。</span><span class="sxs-lookup"><span data-stu-id="36987-247">These issues will be marked with the `blocked` label.</span></span>
- <span data-ttu-id="36987-248">当我们即将发布 GA/RTM 版本时，我们会在需要时根据风险来修复 bug。</span><span class="sxs-lookup"><span data-stu-id="36987-248">We will punt bugs based on risk when needed as is normal as we get close to a GA/RTM release.</span></span>

### <a name="miscellaneous-features"></a><span data-ttu-id="36987-249">其他功能</span><span class="sxs-lookup"><span data-stu-id="36987-249">Miscellaneous features</span></span>

<span data-ttu-id="36987-250">通过 [6.0 里程碑中使用 `type-enhancement` 标记的问题](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0+label%3Atype-enhancement)进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-250">Tracked by [issues labeled with `type-enhancement` in the 6.0 milestone](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0+label%3Atype-enhancement)</span></span>

<span data-ttu-id="36987-251">状态：正在进行</span><span class="sxs-lookup"><span data-stu-id="36987-251">Status: In-progress</span></span>

<span data-ttu-id="36987-252">T 恤大小：大</span><span class="sxs-lookup"><span data-stu-id="36987-252">T-shirt size: Large</span></span>

<span data-ttu-id="36987-253">为 EF 6.0 计划的其他功能包括但不限于：</span><span class="sxs-lookup"><span data-stu-id="36987-253">Miscellaneous features planned for EF 6.0 include, but are not limited to:</span></span>

- [<span data-ttu-id="36987-254">针对非导航集合的拆分查询</span><span class="sxs-lookup"><span data-stu-id="36987-254">Split query for non-navigation collections</span></span>](https://github.com/dotnet/efcore/issues/21234)
- [<span data-ttu-id="36987-255">检测反向工程中的简单联接表并创建多对多关系</span><span class="sxs-lookup"><span data-stu-id="36987-255">Detect simple join tables in reverse engineering and create many-to-many relationships</span></span>](https://github.com/dotnet/efcore/issues/22475)
- [<span data-ttu-id="36987-256">在 SQLite 和 SQL Server 上完成完整/自由文本搜索</span><span class="sxs-lookup"><span data-stu-id="36987-256">Complete full/free-text search on SQLite and SQL Server</span></span>](https://github.com/dotnet/efcore/issues/4823)
- [<span data-ttu-id="36987-257">SQL Server 空间索引</span><span class="sxs-lookup"><span data-stu-id="36987-257">SQL Server spatial indexes</span></span>](https://github.com/dotnet/efcore/issues/12538)
- [<span data-ttu-id="36987-258">用于对模型中给定类型的任何属性指定默认转换的机制/API</span><span class="sxs-lookup"><span data-stu-id="36987-258">Mechanism/API to specify a default conversion for any property of a given type in the model</span></span>](https://github.com/dotnet/efcore/issues/10784)
- [<span data-ttu-id="36987-259">使用 ADO.NET 中新的批处理 API</span><span class="sxs-lookup"><span data-stu-id="36987-259">Use the new batching API from ADO.NET</span></span>](https://github.com/dotnet/efcore/issues/18990)

## <a name="net-integration"></a><span data-ttu-id="36987-260">.NET 集成</span><span class="sxs-lookup"><span data-stu-id="36987-260">.NET integration</span></span>

<span data-ttu-id="36987-261">EF Core 团队还致力于几种相关但独立的技术。</span><span class="sxs-lookup"><span data-stu-id="36987-261">The EF Core team also works on several related but independent technologies.</span></span> <span data-ttu-id="36987-262">具体而言，我们计划在以下方面开展工作：</span><span class="sxs-lookup"><span data-stu-id="36987-262">In particular, we plan to work on:</span></span>

### <a name="enhancements-to-systemdata"></a><span data-ttu-id="36987-263">对 System.Data 进行改进</span><span class="sxs-lookup"><span data-stu-id="36987-263">Enhancements to System.Data</span></span>

<span data-ttu-id="36987-264">通过 [6.0 里程碑中使用 `area-System.Data` 标记的 dotnet\runtime 存储库中的问题](https://github.com/dotnet/runtime/issues?q=is%3Aopen+is%3Aissue+label%3Aarea-System.Data+milestone%3A6.0.0)进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-264">Tracked by [issues in the dotnet\runtime repo labeled with `area-System.Data` in the 6.0 milestone](https://github.com/dotnet/runtime/issues?q=is%3Aopen+is%3Aissue+label%3Aarea-System.Data+milestone%3A6.0.0)</span></span>

<span data-ttu-id="36987-265">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-265">Status: Not started</span></span>

<span data-ttu-id="36987-266">T 恤大小：大</span><span class="sxs-lookup"><span data-stu-id="36987-266">T-shirt size: Large</span></span>

<span data-ttu-id="36987-267">这项工作包括：</span><span class="sxs-lookup"><span data-stu-id="36987-267">This work includes:</span></span>

- <span data-ttu-id="36987-268">实现新的[批处理 API](https://github.com/dotnet/runtime/issues/28633)。</span><span class="sxs-lookup"><span data-stu-id="36987-268">Implementation of the new [batching API](https://github.com/dotnet/runtime/issues/28633).</span></span>
- <span data-ttu-id="36987-269">继续与其他 .NET 团队和社区合作，来了解和发展 ADO.NET。</span><span class="sxs-lookup"><span data-stu-id="36987-269">Continued work with other .NET teams and the community to understand and evolve ADO.NET.</span></span>
- <span data-ttu-id="36987-270">[标准化 DiagnosticSource 以跟踪 System.Data.\* 组件](https://github.com/dotnet/runtime/issues/22336)。</span><span class="sxs-lookup"><span data-stu-id="36987-270">[Standardize on DiagnosticSource for tracing in System.Data.\* components](https://github.com/dotnet/runtime/issues/22336).</span></span>

### <a name="enhancements-to-microsoftdatasqlite"></a><span data-ttu-id="36987-271">对 Microsoft.Data.Sqlite 进行改进</span><span class="sxs-lookup"><span data-stu-id="36987-271">Enhancements to Microsoft.Data.Sqlite</span></span>

<span data-ttu-id="36987-272">通过 [6.0 里程碑中使用 `type-enhancement` 和 `area-adonet-sqlite` 标记的问题](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0+label%3Atype-enhancement+label%3Aarea-adonet-sqlite)进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-272">Tracked by [issues labeled with `type-enhancement` and `area-adonet-sqlite` in the 6.0 milestone](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0+label%3Atype-enhancement+label%3Aarea-adonet-sqlite)</span></span>

<span data-ttu-id="36987-273">状态：正在进行</span><span class="sxs-lookup"><span data-stu-id="36987-273">Status: In-progress</span></span>

<span data-ttu-id="36987-274">T 恤大小：中等</span><span class="sxs-lookup"><span data-stu-id="36987-274">T-shirt size: Medium</span></span>

<span data-ttu-id="36987-275">为 Microsoft.Data.Sqlite 规划了几项细微改进（包括[连接池](https://github.com/dotnet/efcore/issues/13837)和[预定义语句](https://github.com/dotnet/efcore/issues/14044)）来提高性能。</span><span class="sxs-lookup"><span data-stu-id="36987-275">Several small improvements are planned for the Microsoft.Data.Sqlite, including [connection pooling](https://github.com/dotnet/efcore/issues/13837) and [prepared statements](https://github.com/dotnet/efcore/issues/14044) for performance.</span></span>

### <a name="nullable-reference-types"></a><span data-ttu-id="36987-276">可为空引用类型</span><span class="sxs-lookup"><span data-stu-id="36987-276">Nullable reference types</span></span>

<span data-ttu-id="36987-277">通过 [#14150](https://github.com/dotnet/efcore/issues/14150) 进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-277">Tracked by [#14150](https://github.com/dotnet/efcore/issues/14150)</span></span>

<span data-ttu-id="36987-278">状态：正在进行</span><span class="sxs-lookup"><span data-stu-id="36987-278">Status: In-progress</span></span>

<span data-ttu-id="36987-279">T 恤大小：大</span><span class="sxs-lookup"><span data-stu-id="36987-279">T-shirt size: Large</span></span>

<span data-ttu-id="36987-280">我们将对 EF Core 代码进行批注来使用[可为空引用类型](/dotnet/csharp/nullable-references)。</span><span class="sxs-lookup"><span data-stu-id="36987-280">We will annotate the EF Core code to use [nullable reference types](/dotnet/csharp/nullable-references).</span></span>

## <a name="experiments-and-investigations"></a><span data-ttu-id="36987-281">试验和调查</span><span class="sxs-lookup"><span data-stu-id="36987-281">Experiments and investigations</span></span>

<span data-ttu-id="36987-282">EF 团队计划在 EF Core 6.0 版期限内投入时间，在两个方面进行试验和调查。</span><span class="sxs-lookup"><span data-stu-id="36987-282">The EF team is planning to invest time during the EF Core 6.0 timeframe experimenting and investigating in two areas.</span></span> <span data-ttu-id="36987-283">这是一个学习过程，因此没有针对 6.0 版本计划具体的可交付成果。</span><span class="sxs-lookup"><span data-stu-id="36987-283">This is a learning process and as such no concrete deliverables are planned for the 6.0 release.</span></span>

### <a name="sqlservercore"></a><span data-ttu-id="36987-284">SqlServer.Core</span><span class="sxs-lookup"><span data-stu-id="36987-284">SqlServer.Core</span></span>

<span data-ttu-id="36987-285">在 [.NET Data Lab 存储库中](https://github.com/dotnet/datalab/)进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-285">Tracked in the [.NET Data Lab repo](https://github.com/dotnet/datalab/)</span></span>

<span data-ttu-id="36987-286">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-286">Status: Not started</span></span>

<span data-ttu-id="36987-287">T 恤大小：进行中</span><span class="sxs-lookup"><span data-stu-id="36987-287">T-shirt size: Ongoing</span></span>

<span data-ttu-id="36987-288">[Microsoft.Data.SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient/) 是 SQL Server 的功能齐全的 ADO.NET 数据库提供程序。</span><span class="sxs-lookup"><span data-stu-id="36987-288">[Microsoft.Data.SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient/) is a fully-featured ADO.NET database provider for SQL Server.</span></span> <span data-ttu-id="36987-289">它在 .NET Core 和 .NET Framework 上都支持多种 SQL Server 功能。</span><span class="sxs-lookup"><span data-stu-id="36987-289">It supports a broad range of SQL Server features on both .NET Core and .NET Framework.</span></span> <span data-ttu-id="36987-290">不过，它也是一个较大的旧代码库，其行为之间存在很多复杂的交互。</span><span class="sxs-lookup"><span data-stu-id="36987-290">However, it is also a large and old codebase with many complex interactions between its behaviors.</span></span> <span data-ttu-id="36987-291">这使得很难调查使用较新的 .NET Core 功能可能带来的潜在好处。</span><span class="sxs-lookup"><span data-stu-id="36987-291">This makes it difficult to investigate the potential gains the could be made using newer .NET Core features.</span></span> <span data-ttu-id="36987-292">因此，我们将与社区协作开展一项试验，以确定 .NET 的高性能 SQL Server 驱动程序有何潜力。</span><span class="sxs-lookup"><span data-stu-id="36987-292">Therefore, we are starting an experiment in collaboration with the community to determine what potential there is for a highly performing SQL Server driver for .NET.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="36987-293">对 Microsoft.Data.SqlClient 的投资不会改变。</span><span class="sxs-lookup"><span data-stu-id="36987-293">Investment in Microsoft.Data.SqlClient is not changing.</span></span> <span data-ttu-id="36987-294">无论是否使用 EF Core，它都将继续作为连接 SQL Server 和 SQL Azure 的建议方法。</span><span class="sxs-lookup"><span data-stu-id="36987-294">It will continue to be the recommended way to connect to SQL Server and SQL Azure, both with and without EF Core.</span></span> <span data-ttu-id="36987-295">引入新的 SQL Server 功能后，它将继续支持新功能。</span><span class="sxs-lookup"><span data-stu-id="36987-295">It will continue to support new SQL Server features as they are introduced.</span></span>

### <a name="graphql"></a><span data-ttu-id="36987-296">GraphQL</span><span class="sxs-lookup"><span data-stu-id="36987-296">GraphQL</span></span>

<span data-ttu-id="36987-297">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-297">Status: Not started</span></span>

<span data-ttu-id="36987-298">T 恤大小：进行中</span><span class="sxs-lookup"><span data-stu-id="36987-298">T-shirt size: Ongoing</span></span>

<span data-ttu-id="36987-299">在过去几年里，[GraphQL](https://graphql.org/) 在各种平台上获得了广泛的关注。</span><span class="sxs-lookup"><span data-stu-id="36987-299">[GraphQL](https://graphql.org/) has been gaining traction over the last few years across a variety of platforms.</span></span> <span data-ttu-id="36987-300">我们计划对该方面进行调查，并找到改进 .NET 体验的方法。</span><span class="sxs-lookup"><span data-stu-id="36987-300">We plan to investigate the space and find ways to improve the experience with .NET.</span></span> <span data-ttu-id="36987-301">这涉及到与社区协作来了解和支持现有生态系统。</span><span class="sxs-lookup"><span data-stu-id="36987-301">This will involve working with the community on understanding and supporting the existing ecosystem.</span></span> <span data-ttu-id="36987-302">它还可能涉及来自 Microsoft 的特定投入，可能是对现有工作进行贡献，也可能是在 Microsoft 堆栈中开发补充部分。</span><span class="sxs-lookup"><span data-stu-id="36987-302">It may also involve specific investment from Microsoft, either in the form of contributions to existing work or in developing complimentary pieces in the Microsoft stack.</span></span>

### <a name="dataverse-formerly-common-data-services"></a><span data-ttu-id="36987-303">DataVerse（以前称为 Common Data Service）</span><span class="sxs-lookup"><span data-stu-id="36987-303">DataVerse (formerly Common Data Services)</span></span>

<span data-ttu-id="36987-304">状态：尚未开始</span><span class="sxs-lookup"><span data-stu-id="36987-304">Status: Not started</span></span>

<span data-ttu-id="36987-305">T 恤大小：进行中</span><span class="sxs-lookup"><span data-stu-id="36987-305">T-shirt size: Ongoing</span></span>

<span data-ttu-id="36987-306">[DataVerse](/powerapps/maker/data-platform/data-platform-intro) 是一种基于列的数据存储，旨在快速开发商业应用程序。</span><span class="sxs-lookup"><span data-stu-id="36987-306">[DataVerse](/powerapps/maker/data-platform/data-platform-intro) is a column-based data store designed for rapid development of business applications.</span></span> <span data-ttu-id="36987-307">它会自动处理复杂的数据类型，如二进制对象 (BLOB)，并具有内置的实体和关系，如组织和联系人。</span><span class="sxs-lookup"><span data-stu-id="36987-307">It automatically handles complex data types like binary objects (BLOBs) and has built-in entities and relationships like organizations and contacts.</span></span> <span data-ttu-id="36987-308">虽然存在 SDK，但具有 EF Core 提供程序可能有利于开发人员，他们可由此使用高级 LINQ 查询、利用工作单元，并具有一致的数据访问 API。</span><span class="sxs-lookup"><span data-stu-id="36987-308">An SDK exists but developers may benefit from having an EF Core provider to use advanced LINQ queries, take advantage of unit of work and have a consistent data access API.</span></span> <span data-ttu-id="36987-309">团队将考虑使用不同的方法来改进 .NET 开发人员连接 DataVerse 的体验。</span><span class="sxs-lookup"><span data-stu-id="36987-309">The team will consider different ways to improve the .NET developer experience connecting to DataVerse.</span></span>

## <a name="below-the-cut-line"></a><span data-ttu-id="36987-310">Below-the-cut-line</span><span class="sxs-lookup"><span data-stu-id="36987-310">Below-the-cut-line</span></span>

<span data-ttu-id="36987-311">通过[使用 `consider-for-current-release` 标记的问题](https://github.com/aspnet/EntityFrameworkCore/issues?q=is%3Aopen+is%3Aissue+label%3Aconsider-for-current-release)进行跟踪</span><span class="sxs-lookup"><span data-stu-id="36987-311">Tracked by [issues labeled with `consider-for-current-release`](https://github.com/aspnet/EntityFrameworkCore/issues?q=is%3Aopen+is%3Aissue+label%3Aconsider-for-current-release)</span></span>

<span data-ttu-id="36987-312">当前未针对 6.0 版本计划这些 bug 修复和增强功能，但我们将根据整个版本使用过程中的反馈以及在上述工作中所取得的进展来进行研究。</span><span class="sxs-lookup"><span data-stu-id="36987-312">These are bug fixes and enhancements that are **not** currently scheduled for the 6.0 release, but we will look at based on feedback throughout the release together with progress made on the work above.</span></span> <span data-ttu-id="36987-313">这些问题可能会引入到 EF Core 6.0，并将自动成为下一个版本的候选项。</span><span class="sxs-lookup"><span data-stu-id="36987-313">These issues may be pulled in to EF Core 6.0, and automatically become candidates for the next release.</span></span>

<span data-ttu-id="36987-314">此外，我们始终会在计划时考虑[投票最多的问题](https://github.com/dotnet/efcore/issues?q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc)。</span><span class="sxs-lookup"><span data-stu-id="36987-314">In addition, we always consider the [most voted issues](https://github.com/dotnet/efcore/issues?q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc) when planning.</span></span> <span data-ttu-id="36987-315">从版本中去除其中任何问题总是很痛苦的，但是我们确实需要针对所拥有的资源制定切合实际的计划。</span><span class="sxs-lookup"><span data-stu-id="36987-315">Cutting any of these issues from a release is always painful, but we do need a realistic plan for the resources we have.</span></span> <span data-ttu-id="36987-316">请务必对你需要的功能投票 (👍)。</span><span class="sxs-lookup"><span data-stu-id="36987-316">Make sure you have voted (👍) for the features you need.</span></span>

## <a name="suggestions"></a><span data-ttu-id="36987-317">建议</span><span class="sxs-lookup"><span data-stu-id="36987-317">Suggestions</span></span>

<span data-ttu-id="36987-318">你对计划的反馈非常重要。</span><span class="sxs-lookup"><span data-stu-id="36987-318">Your feedback on planning is important.</span></span> <span data-ttu-id="36987-319">指出问题重要性的最佳方式是在 GitHub 上为该问题投票 (👍)。</span><span class="sxs-lookup"><span data-stu-id="36987-319">The best way to indicate the importance of an issue is to vote (👍) for that issue on GitHub.</span></span> <span data-ttu-id="36987-320">然后，此数据将进入下一个版本的[计划过程](xref:core/what-is-new/release-planning)。</span><span class="sxs-lookup"><span data-stu-id="36987-320">This data will then feed into the [planning process](xref:core/what-is-new/release-planning) for the next release.</span></span>
