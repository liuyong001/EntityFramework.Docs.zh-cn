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
# <a name="plan-for-entity-framework-core-60"></a>针对 Entity Framework Core 6.0 的计划

如[计划过程](xref:core/what-is-new/release-planning)中所述，我们已将利益干系人输入的内容收集到针对 Entity Framework Core (EF Core) 6.0 版的计划中。

与以前的版本不同，此计划不会尝试覆盖 6.0 版的各项工作。 相反，它指出了我们打算在此版本中投入的方面和方式，而在我们一边使用该版本一边收集反馈和进行学习时，可灵活地调整范围或引入新工作。

> [!IMPORTANT]
> 此计划不是承诺。 它是一个起点，会随着我们了解更多信息而发展。 其中可能会纳入当前未针对 6.0 计划的某些内容。 而当前已针对 6.0 计划的某些内容可能会被淘汰。

## <a name="general-information"></a>常规信息

### <a name="version-number-and-release-date"></a>版本号和发布日期

EF Core 6.0 是 EF Core 5.0 之后的下一版本，目前计划在 2021 年 11 月与 .NET 6 同时发布。

### <a name="supported-platforms"></a>受支持的平台

EF Core 6.0 当前面向 .NET 5。 在我们快要发布时，这可能会更新到 .NET 6。 EF Core 6.0 不面向任何 .NET Standard 版本；有关详细信息，请参阅 [.NET Standard 的未来](https://devblogs.microsoft.com/dotnet/the-future-of-net-standard/)。

EF Core 6.0 不会在 .NET Framework 上运行。

EF Core 6.0 将作为[长期支持 (LTS) 版本](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)与 .NET 6 保持一致。

### <a name="breaking-changes"></a>中断性变更

我们将继续改进 EF Core 和 .NET 平台，因此 EF Core 6.0 将包含少量[中断性变更](xref:core/what-is-new/ef-core-6.0/breaking-changes)。 我们的目标是允许大多数应用程序进行更新而不会中断。

## <a name="themes"></a>主题

以下方面将构成 EF Core 6.0 中多数投资的基础。

## <a name="highly-requested-features"></a>被强烈要求的功能

与往常一样，[计划过程](xref:core/what-is-new/release-planning)中的主要输入内容来自[对 GitHub 上功能的投票 (👍)](https://github.com/dotnet/efcore/issues?q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc)。 对于 EF Core 6.0，我们计划处理以下被强烈要求的功能：

### <a name="sql-server-temporal-tables"></a>SQL Server 临时表

通过 [#4693](https://github.com/dotnet/efcore/issues/4693) 进行跟踪

状态：尚未开始

T 恤大小：大

临时表支持查询在任何时间点存储在表中的数据，而不是像普通表那样仅限存储的最新数据。 EF Core 6.0 将允许通过迁移来创建临时表，并允许通过 LINQ 查询访问数据。

这项工作最初作用的范围[如问题所述](https://github.com/dotnet/efcore/issues/4693#issuecomment-625048974)。 我们可能会根据发布期间的反馈引入其他支持。

### <a name="json-columns"></a>JSON 列

通过 [#4021](https://github.com/dotnet/efcore/issues/4021) 进行跟踪

状态：尚未开始

T 恤大小：中等

此功能将为 JSON 支持引入可由任何数据库提供程序实现的通用机制和模式。 我们将与社区合作来调整 [Npgsql](https://github.com/npgsql/efcore.pg) 和 [Pomelo MySQL](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql) 的现有实现，并添加对 SQL Server 和 SQLite 的支持。

### <a name="columnattributeorder"></a>ColumnAttribute.Order

通过 [#10059](https://github.com/dotnet/efcore/issues/10059) 进行跟踪

状态：尚未开始

T 恤大小：小

此功能将允许在使用迁移或 `EnsureCreated` 创建表时对列进行任意排序。 请注意，更改现有表中列的顺序需要重新生成表，而我们并不计划在任何 EF Core 版本中支持这项功能。

## <a name="performance"></a>性能

虽然 EF Core 的速度通常比 EF6 更快，但某些方面仍存在大幅提升性能的空间。 我们计划在 EF Core 6.0 中处理其中的几个方面，同时改进我们的性能基础结构和测试。

本主题将涉及大量迭代调查，它将告知我们要将资源集中到哪些方面。 我们计划从以下方面开始：

### <a name="performance-infrastructure-and-new-tests"></a>性能基础结构和新测试

状态：尚未开始

T 恤大小：中等

EF Core 代码库已包含一组性能基准，它们每日都会执行。 对于 6.0 版，我们计划改进这些测试的基础结构并添加新测试。 我们还将分析主线性能方案，并修复发现的任何可轻松解决的问题。

### <a name="compiled-models"></a>已编译的模型

通过 [#1906](https://github.com/dotnet/efcore/issues/1906) 进行跟踪

状态：尚未开始

T 恤大小：加大

已编译的模型将允许生成 EF 模型的编译形式。 这将提供更好的启动性能，通常还在访问模型时提供更好的性能。

### <a name="techempower-fortunes"></a>TechEmpower Fortunes

通过 [#23611](https://github.com/dotnet/efcore/issues/23611) 进行跟踪

状态：尚未开始

T 恤大小：加大

几年来，我们一直在 .NET 上针对 PostgreSQL 数据库运行行业标准的 [TechEmpower 基准](https://www.techempower.com/benchmarks/)。 [Fortunes 基准](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=fortune) 与 EF 方案尤为相关。 我们有此基准的多个变体，包括：

- 直接使用 ADO.NET 的实现。 这是此处列出的三个实现中最快的一个。
- 使用 [Dapper](https://www.nuget.org/packages/Dapper/) 的实现。 与直接使用 ADO.NET 相比，该实现更慢，但仍然很快。
- 使用 EF Core 的实现。 这是当前三个实现中最慢的一个。

EF Core 6.0 的目标是使 EF Core 的性能与 TechEmpower Fortunes 基准上的 Dapper 不相上下。 （这是一项重大挑战，但我们将尽最大努力尽量实现。）

### <a name="linkeraot"></a>链接器/AOT

通过 [#10963](https://github.com/dotnet/efcore/issues/10963) 进行跟踪

状态：尚未开始

T 恤大小：中等

EF Core 执行大量的运行时代码生成。 对于依赖链接器树握手（如 Xamarin 和 Blazor）的应用模型以及不允许动态编译的平台（如 iOS）来说，这非常困难。 作为 EF Core 6.0 的一部分，我们将继续调查该方面，并尽可能进行有针对性的改进。 不过，我们并不期望在 6.0 版期限内完全消除差距。

## <a name="migrations-and-deployment"></a>迁移和部署

根据对 [EF Core 5.0 的调查](xref:core/what-is-new/ef-core-5.0/plan#migrations-and-deployment-experience)，我们计划引入对管理迁移和部署数据库的改进支持。 这包括两个主要方面：

### <a name="migrations-bundles"></a>迁移捆绑包

通过 [#19693](https://github.com/dotnet/efcore/issues/19693) 进行跟踪

状态：尚未开始

T 恤大小：中等

迁移捆绑包是一种独立的可执行文件，用于将迁移应用到生产数据库。 此行为将与 `dotnet ef database update` 匹配，但由于所需的所有内容都包含在单个可执行文件中，因此它会使 SSH/Docker/PowerShell 部署更容易。

### <a name="managing-migrations"></a>管理迁移

通过 [#22945](https://github.com/dotnet/efcore/issues/22945) 进行跟踪

状态：尚未开始

T 恤大小：大

为应用程序创建的迁移数可能会增加，从而成为负担。 此外，即使不需要，这些迁移也经常与应用程序一起部署。 在 EF Core 6.0 中，我们计划通过更好的工具和项目/程序集管理来改进这一点。 我们计划解决的两个具体问题是[将多个迁移压缩为一个迁移](https://github.com/dotnet/efcore/issues/2174)和[重新生成干净的模型快照](https://github.com/dotnet/efcore/issues/18557)。

## <a name="improve-existing-features-and-fix-bugs"></a>改进现有功能并修复 bug

[分配给 6.0.0 里程碑的任何问题或 bug](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0) 目前都计划在此版本中解决。 这包括许多小型改进和 bug 修复。

### <a name="ef6-query-parity"></a>EF6 查询奇偶校验

通过[使用“ef6-parity”标记的问题和 6.0 里程碑中的问题](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+label%3Aef6-parity+milestone%3A6.0.0)进行跟踪

状态：尚未开始

T 恤大小：大

EF Core 5.0 支持 EF6 支持的大多数查询模式，也支持 EF6 不支持的模式。 对于 EF Core 6.0，我们计划缩小差距，使受支持的 EF Core 查询成为受支持的 EF6 查询的真正超集。 这将由对差异的调查来推动，但已包含 GroupBy 问题（例如[转换后接 FirstOrDefault 的 GroupBy](https://github.com/dotnet/efcore/issues/12088)），以及针对[基元](https://github.com/dotnet/efcore/issues/11624)和[未映射](https://github.com/dotnet/efcore/issues/10753)类型的原始 SQL 查询。  

### <a name="value-objects"></a>值对象

通过 [#9906](https://github.com/dotnet/efcore/issues/9906) 进行跟踪

状态：尚未开始

T 恤大小：中等

以前，拥有实体的团队视图（旨在[提供聚合支持](https://www.martinfowler.com/bliki/DDD_Aggregate.html)）也是[值对象](https://www.martinfowler.com/bliki/ValueObject.html)的合理近似值。 经验表明事实并非如此。 因此，我们计划在 EF Core 6.0 引入一种更好的体验，重点关注域驱动设计中值对象的需求。 此方法将基于值转换器而不是拥有的实体。

这项工作最初作用的范围是允许[映射到多个列的值转换器](https://github.com/dotnet/efcore/issues/13947)。 我们可能会根据发布期间的反馈引入其他支持。

### <a name="cosmos-database-provider"></a>Cosmos 数据库提供程序

通过[使用“area-cosmos”标记的问题和 6.0 里程碑中的问题](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0+label%3Aarea-cosmos)进行跟踪

状态：尚未开始

T 恤大小：大

我们正在积极收集有关对 EF Core 6.0 中的 Cosmos 提供程序进行哪些改进的反馈。 我们会在了解更多信息后更新此文档。 现在，请务必为你需要的 Cosmos 功能投票 (👍)。

### <a name="expose-model-building-conventions-to-applications"></a>向应用程序公开模型构建约定

通过 [#214](https://github.com/dotnet/efcore/issues/214) 进行跟踪

状态：尚未开始

T 恤大小：中

EF Core 使用一组约定来从 .NET 类型构建模型。 这些约定当前由数据库提供程序控制。 在 EF Core 6.0 中，我们打算允许应用程序与这些约定挂钩并更改这些约定。

### <a name="zero-bug-balance-zbb"></a>零 bug 平衡 (ZBB)

通过 [6.0 里程碑中使用 `type-bug` 标记的问题](https://github.com/dotnet/efcore/issues?utf8=%E2%9C%93&q=is%3Aissue+milestone%3A6.0.0+label%3Atype-bug+)进行跟踪

状态：正在进行

T 恤大小：大

我们计划在 EF Core 6.0 版期限内修复所有待解决的 bug。 需谨记以下几点：

- 这特别适用于标记了 [type-bug](https://github.com/dotnet/efcore/issues?q=is%3Aissue+label%3Atype-bug) 的问题。
- 存在一些例外，例如当需要设计更改或新功能才能正确修复 bug 时。 这些问题将使用 `blocked` 标签进行标记。
- 当我们即将发布 GA/RTM 版本时，我们会在需要时根据风险来修复 bug。

### <a name="miscellaneous-features"></a>其他功能

通过 [6.0 里程碑中使用 `type-enhancement` 标记的问题](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0+label%3Atype-enhancement)进行跟踪

状态：正在进行

T 恤大小：大

为 EF 6.0 计划的其他功能包括但不限于：

- [针对非导航集合的拆分查询](https://github.com/dotnet/efcore/issues/21234)
- [检测反向工程中的简单联接表并创建多对多关系](https://github.com/dotnet/efcore/issues/22475)
- [在 SQLite 和 SQL Server 上完成完整/自由文本搜索](https://github.com/dotnet/efcore/issues/4823)
- [SQL Server 空间索引](https://github.com/dotnet/efcore/issues/12538)
- [用于对模型中给定类型的任何属性指定默认转换的机制/API](https://github.com/dotnet/efcore/issues/10784)
- [使用 ADO.NET 中新的批处理 API](https://github.com/dotnet/efcore/issues/18990)

## <a name="net-integration"></a>.NET 集成

EF Core 团队还致力于几种相关但独立的技术。 具体而言，我们计划在以下方面开展工作：

### <a name="enhancements-to-systemdata"></a>对 System.Data 进行改进

通过 [6.0 里程碑中使用 `area-System.Data` 标记的 dotnet\runtime 存储库中的问题](https://github.com/dotnet/runtime/issues?q=is%3Aopen+is%3Aissue+label%3Aarea-System.Data+milestone%3A6.0.0)进行跟踪

状态：尚未开始

T 恤大小：大

这项工作包括：

- 实现新的[批处理 API](https://github.com/dotnet/runtime/issues/28633)。
- 继续与其他 .NET 团队和社区合作，来了解和发展 ADO.NET。
- [标准化 DiagnosticSource 以跟踪 System.Data.* 组件](https://github.com/dotnet/runtime/issues/22336)。

### <a name="enhancements-to-microsoftdatasqlite"></a>对 Microsoft.Data.Sqlite 进行改进

通过 [6.0 里程碑中使用 `type-enhancement` 和 `area-adonet-sqlite` 标记的问题](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+milestone%3A6.0.0+label%3Atype-enhancement+label%3Aarea-adonet-sqlite)进行跟踪

状态：正在进行

T 恤大小：中等

为 Microsoft.Data.Sqlite 规划了几项细微改进（包括[连接池](https://github.com/dotnet/efcore/issues/13837)和[预定义语句](https://github.com/dotnet/efcore/issues/14044)）来提高性能。

### <a name="nullable-reference-types"></a>可为空引用类型

通过 [#14150](https://github.com/dotnet/efcore/issues/14150) 进行跟踪

状态：正在进行

T 恤大小：大

我们将对 EF Core 代码进行批注来使用[可为空引用类型](/dotnet/csharp/nullable-references)。

## <a name="experiments-and-investigations"></a>试验和调查

EF 团队计划在 EF Core 6.0 版期限内投入时间，在两个方面进行试验和调查。 这是一个学习过程，因此没有针对 6.0 版本计划具体的可交付成果。

### <a name="sqlservercore"></a>SqlServer.Core

在 [.NET Data Lab 存储库中](https://github.com/dotnet/datalab/)进行跟踪

状态：尚未开始

T 恤大小：进行中

[Microsoft.Data.SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient/) 是 SQL Server 的功能齐全的 ADO.NET 数据库提供程序。 它在 .NET Core 和 .NET Framework 上都支持多种 SQL Server 功能。 不过，它也是一个较大的旧代码库，其行为之间存在很多复杂的交互。 这使得很难调查使用较新的 .NET Core 功能可能带来的潜在好处。 因此，我们将与社区协作开展一项试验，以确定 .NET 的高性能 SQL Server 驱动程序有何潜力。

> [!IMPORTANT]
> 对 Microsoft.Data.SqlClient 的投资不会改变。 无论是否使用 EF Core，它都将继续作为连接 SQL Server 和 SQL Azure 的建议方法。 引入新的 SQL Server 功能后，它将继续支持新功能。

### <a name="graphql"></a>GraphQL

状态：尚未开始

T 恤大小：进行中

在过去几年里，[GraphQL](https://graphql.org/) 在各种平台上获得了广泛的关注。 我们计划对该方面进行调查，并找到改进 .NET 体验的方法。 这涉及到与社区协作来了解和支持现有生态系统。 它还可能涉及来自 Microsoft 的特定投入，可能是对现有工作进行贡献，也可能是在 Microsoft 堆栈中开发补充部分。

### <a name="dataverse-formerly-common-data-services"></a>DataVerse（以前称为 Common Data Service）

状态：尚未开始

T 恤大小：进行中

[DataVerse](/powerapps/maker/data-platform/data-platform-intro) 是一种基于列的数据存储，旨在快速开发商业应用程序。 它会自动处理复杂的数据类型，如二进制对象 (BLOB)，并具有内置的实体和关系，如组织和联系人。 虽然存在 SDK，但具有 EF Core 提供程序可能有利于开发人员，他们可由此使用高级 LINQ 查询、利用工作单元，并具有一致的数据访问 API。 团队将考虑使用不同的方法来改进 .NET 开发人员连接 DataVerse 的体验。

## <a name="below-the-cut-line"></a>Below-the-cut-line

通过[使用 `consider-for-current-release` 标记的问题](https://github.com/aspnet/EntityFrameworkCore/issues?q=is%3Aopen+is%3Aissue+label%3Aconsider-for-current-release)进行跟踪

当前未针对 6.0 版本计划这些 bug 修复和增强功能，但我们将根据整个版本使用过程中的反馈以及在上述工作中所取得的进展来进行研究。 这些问题可能会引入到 EF Core 6.0，并将自动成为下一个版本的候选项。

此外，我们始终会在计划时考虑[投票最多的问题](https://github.com/dotnet/efcore/issues?q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc)。 从版本中去除其中任何问题总是很痛苦的，但是我们确实需要针对所拥有的资源制定切合实际的计划。 请务必对你需要的功能投票 (👍)。

## <a name="suggestions"></a>建议

你对计划的反馈非常重要。 指出问题重要性的最佳方式是在 GitHub 上为该问题投票 (👍)。 然后，此数据将进入下一个版本的[计划过程](xref:core/what-is-new/release-planning)。
