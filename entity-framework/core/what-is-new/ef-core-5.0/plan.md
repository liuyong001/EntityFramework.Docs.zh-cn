---
title: 针对 Entity Framework Core 5.0 的计划
description: 为 Entity Framework Core 5.0 规划的功能
author: ajcvickers
ms.date: 08/22/2020
uid: core/what-is-new/ef-core-5.0/plan
ms.openlocfilehash: ba56a5f3c79dacc25b3402be114c57067da49433
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129039"
---
# <a name="plan-for-entity-framework-core-50"></a>针对 Entity Framework Core 5.0 的计划

> [!IMPORTANT]
> EF Core 5.0 [现已发布](xref:core/what-is-new/index)。 本页面保留作为计划的历史记录。

如[计划过程](xref:core/what-is-new/release-planning)中所述，我们已来自利益干系人的输入收集到针对 EF Core 5.0 版的暂定计划中。

> [!IMPORTANT]
> 此计划仍是半成品。 这里不进行任何承诺。 此计划是一个起点，会随着我们了解更多信息而发展。 当前未针对 5.0 进行计划的某些内容可能会被纳入。 当前已针对 5.0 进行计划的某些内容可能会被淘汰。

## <a name="general-information"></a>常规信息

### <a name="version-number-and-release-date"></a>版本号和发布日期

EF Core 5.0 当前计划[与 .NET 5.0 同时](https://devblogs.microsoft.com/dotnet/introducing-net-5/)发布。 已选择版本“5.0”，以便与 .NET 5.0 保持一致。

### <a name="supported-platforms"></a>受支持的平台

EF Core 5.0 计划在任何 .NET Standard 2.1 平台（包括 .NET 5.0）上运行。 这是更常见的 .NET 范围[平台与 .NET Core 的聚合](https://devblogs.microsoft.com/dotnet/introducing-net-5/)的一部分。

EF Core 5.0 不会在 .NET Framework 上运行。

### <a name="breaking-changes"></a>重大更改

EF Core 5.0 将包含一些[中断性变更](xref:core/what-is-new/ef-core-5.0/breaking-changes)，但与 EF Core 3.0 的情况相比，这些更改的严重性要小得多。 我们的目标是允许大多数应用程序进行更新而不会中断。

预计会对数据库提供程序进行一些重大更改，尤其是在 TPT 支持方面。 但是，我们预计为 5.0 更新提供程序的工作会少于为 3.0 更新所需的工作。

## <a name="themes"></a>主题

我们提取了几个主要领域或主题，它们将构成对 EF Core 5.0 进行大量投入的基础。

## <a name="fully-transparent-many-to-many-mapping-by-convention"></a>以完全透明的方式按约定进行多对多映射

主要开发人员：@smitpatel、@AndriySvyryd 和 @lajones

通过 [#10508](https://github.com/dotnet/efcore/issues/10508) 进行跟踪

T 恤大小：L

状态：完成

多对多是 GitHub 积压工作 (backlog) 中[请求最多的功能](https://github.com/dotnet/efcore/issues/1368)（大约 506 张投票）。

多对多关系的支持分为三个主要方面：

* 跳过导航属性 - 将在下个主题中讲解它们。
* 属性包实体类型。 这些类型使标准 CLR 类型（例如 `Dictionary`）可以用于实体实例，使得每种实体类型都不需要显式 CLR 类型。 通过 [#9914](https://github.com/dotnet/efcore/issues/9914) 进行跟踪。
* Sugar 可用于轻松配置多对多关系。

除了支持跳过导航，我们现还将多对多关系的这些其他方面引入到 EF Core 5.0 来提供完整的体验。

## <a name="many-to-many-navigation-properties-aka-skip-navigations"></a>多对多导航属性（即“跳过导航”）

开发人员负责人：@smitpatel 和 @AndriySvyryd

通过 [#19003](https://github.com/dotnet/efcore/issues/19003) 进行跟踪

T 恤大小：L

状态：完成

如第一个主题中所述，多对多支持具有多个方面。
该主题专门跟踪对跳过导航功能的使用。
我们认为，对于需要多对多支持的人员而言，最重要的阻止因素是无法在业务逻辑（如查询）中使用“自然”关系，而无需引用联接表。
联接表实体类型可能仍然存在，但不应妨碍业务逻辑。

## <a name="table-per-type-tpt-inheritance-mapping"></a>每个类型一张表 (TPT) 继承映射

开发人员主管：@AndriySvyryd 和 @smitpatel

通过 [#2266](https://github.com/dotnet/efcore/issues/2266) 进行跟踪

T 恤大小：XL

状态：完成

我们要实现 TPT 是因为它是经常请求的功能（大约 289 张投票；第三名），并且它需要一些低级更改，我们认为这些更改适合于总体 .NET 5 计划的基础性质。 我们预计这会形成数据库提供程序的重大更改，但与 3.0 所需的更改相比，这些更改的严重性要小得多。

## <a name="filtered-include"></a>经过筛选的包含

开发人员负责人：@maumar

通过 [#1833](https://github.com/dotnet/efcore/issues/1833) 进行跟踪

T 恤大小：M

状态：完成

经过筛选的包含是经常请求的功能（大约 376 张投票；第二名），其工作量不大，我们认为这会使当前需要模型级筛选器或更复杂查询的许多方案不受阻碍或更加容易。

## <a name="split-include"></a>拆分 Include

开发人员负责人：@smitpatel

通过 [#20892](https://github.com/dotnet/efcore/issues/20892) 进行跟踪

T 恤大小：L

状态：完成

EF Core 3.0 更改了默认行为，它为给定的 LINQ 查询创建一个 SQL 查询。
对于使用 Include 来处理多个集合的查询来说，这导致性能大幅下降。

在 EF Core 5.0 中，我们将保留新的默认行为。
不过，EF Core 5.0 现在允许为集合 Includes 生成多个查询，因为使用单个查询会导致性能不佳。

## <a name="required-one-to-one-dependents"></a>需要一对一依赖项

开发人员负责人：@AndriySvyryd 和 @smitpatel

通过 [#12100](https://github.com/dotnet/efcore/issues/12100) 进行跟踪

T 恤大小：M

状态：完成

在 EF Core 3.0 中，所有依赖项（包括所拥有的类型）都是可选的（例如 Person.Address 可为 null）。 在 EF Core 5.0 中，可根据需要配置依赖项。

## <a name="rationalize-totable-toquery-toview-fromsql-etc"></a>合理化 ToTable、ToQuery、ToView、FromSql 等

开发人员负责人：@AndriySvyryd 和 @smitpatel

通过 [#17270](https://github.com/dotnet/efcore/issues/17270) 进行跟踪

T 恤大小：L

状态：完成

在以前的版本中，我们在支持原始 SQL、无键类型和相关领域方面取得了进展。 但是，在所有内容作为一个整体协同工作的方式上存在差距和不一致。 5\.0 的目标是修复这些问题，并为定义、迁移和使用不同类型的实体及其关联查询和数据库项目创造良好体验。 这也可能涉及到已编译查询 API 的更新。

请注意，此项可能会导致某些应用程序级重大更改，因为我们当前拥有的某些功能过于宽松，以至于它可能很快导致人们陷入困境。 我们可能最终会阻止其中一些功能，并改为提供有关应执行的操作的指导。

## <a name="general-query-enhancements"></a>常规查询增强功能

开发人员负责人：@smitpatel 和 @maumar

通过 [5.0 里程碑中使用 `area-query` 标记的问题](https://github.com/dotnet/efcore/issues?utf8=%E2%9C%93&q=is%3Aissue+label%3Aarea-query+milestone%3A5.0.0+)进行跟踪

T 恤大小：XL

状态：完成

查询转换代码已针对 EF Core 3.0 进行了广泛重写。 因此，查询代码一般处于更可靠的状态。 对于 5.0，在支持 TPT 和 skip 导航属性所需的更改范围之外，我们未计划进行重大查询更改。 但是，仍然需要大量工作来修复 3.0 全面修改中遗留的一些技术债务。 我们还计划修复许多 bug 并实现少量的增强功能，以进一步改进总体查询体验。

## <a name="migrations-and-deployment-experience"></a>迁移和部署体验

开发人员负责人：@bricelam

通过 [#19587](https://github.com/dotnet/efcore/issues/19587) 进行跟踪

T 恤大小：L

状态：作用域/完成

范围问题：[迁移捆绑功能](https://github.com/dotnet/efcore/issues/19693)已推迟到 EF Core 5.0 发布之后。 但是，EF Core 5.0 中将包含[与迁移相关的一些其他目标改进](https://github.com/dotnet/efcore/issues/19587#issuecomment-668794460)

当前，许多开发人员在应用程序启动时迁移其数据库。 这十分简单，但不建议这样做，因为：

* 多个线程/进程/服务器可能会并发尝试迁移数据库
* 发生这种情况时，应用程序可能会尝试访问不一致的状态
* 通常，不应为应用程序执行授予修改架构的数据库权限
* 如果出现问题，则难以还原为干净状态

我们希望在这里提供更好的体验，从而可以轻松地在部署时迁移数据库。 这应该：

* 可在 Linux、Mac 和 Windows 上正常工作
* 在命令行上具有良好体验
* 支持采用容器的方案
* 适用于常用的实际部署工具/流
* 至少集成到 Visual Studio 中

结果可能是在 EF Core 中进行许多小改进（例如，SQLite 上更好的迁移），并与其他团队一起进行指导和长期协作，以改进不仅限于 EF 的端到端体验。

## <a name="ef-core-platforms-experience"></a>EF Core 平台体验

开发人员负责人：@roji 和 @bricelam

通过 [#19588](https://github.com/dotnet/efcore/issues/19588) 进行跟踪

T 恤大小：L

状态：作用域/完成

范围问题：为 Blazor、Xamarin、WinForms 和 WPF 发布了平台指南和示例。 现已计划为 EF Core 6.0 的发行进行 Xamarin 和其他 AOT/链接器工作。

我们提供了有关在类似传统 MVC 的 Web 应用程序中使用 EF Core 的良好指导。 适用于其他平台和应用程序模型的指导缺失或过期。 对于 EF Core 5.0，我们计划调查、改进和记录在以下方面使用 EF Core 的体验：

* Blazor
* Xamarin，包括使用 AOT/链接器情景
* WinForms/WPF/WinUI，可能还有其他 U.I. 框架

这可能是在 EF Core 中进行许多小改进，并与其他团队一起进行指导和长期协作，以改进不仅限于 EF 的端到端体验。

我们计划研究的特定领域有：

* 部署，包括使用 EF 工具（例如用于迁移）的体验
* 应用程序模型（包括 Xamarin 和 Blazor），可能还有其他模型
* SQLite 体验，包括空间体验和表重新生成
* AOT 和链接体验
* 诊断集成，包括性能计数器

## <a name="performance"></a>性能

开发人员负责人：@roji

通过 [5.0 里程碑中使用 `area-perf` 标记的问题](https://github.com/dotnet/efcore/issues?utf8=%E2%9C%93&q=is%3Aissue+label%3Aarea-perf+milestone%3A5.0.0+)进行跟踪

T 恤大小：L

状态：作用域/完成

范围问题：Npgsql 提供程序的主要性能改进已完成。 现已计划为 EF Core 6.0 的发行进行其他性能工作。

对于 EF Core，我们计划改进性能基准套件，并对运行时进行定向性能改进。 此外，我们计划完成在 3.0 发布周期内进行原型设计的新 ADO.NET 批处理 API。 同样在 ADO.NET 层，我们计划对 Npgsql 提供程序进行其他性能改进。

作为此工作的一部分，我们还计划根据需要添加 ADO.NET/EF Core 性能计数器和其他诊断。

## <a name="architecturalcontributor-documentation"></a>体系结构/参与者文档

文档管理人员主管：@ajcvickers

通过 [#1920](https://github.com/dotnet/EntityFramework.Docs/issues/1920) 进行跟踪

T 恤大小：L

状态：剪切

这里的思路是使人们更容易了解 EF Core 的内部情况。 这可能对于使用 EF Core 的任何人都十分有用，但主要动机是使外部人员更容易：

* 为 EF Core 代码做出贡献
* 创建数据库提供程序
* 构建其他扩展

更新：遗憾的是，该计划过于宏大。
我们仍相信这很重要，但遗憾的是，EF Core 5.0 未包含它。

## <a name="microsoftdatasqlite-documentation"></a>Microsoft.Data.Sqlite 文档

文档管理人员主管：@bricelam

通过 [#1675](https://github.com/dotnet/EntityFramework.Docs/issues/1675) 进行跟踪

T 恤大小：M

状态：已完成。 新文档在 [Microsoft 文档站点上提供](/dotnet/standard/data/sqlite/?tabs=netcore-cli)。

EF 团队还拥有 Microsoft.Data.Sqlite ADO.NET 提供程序。 我们计划在 5.0 版本中完整记录此提供程序。

## <a name="general-documentation"></a>常规文档

文档管理人员主管：@ajcvickers

通过 [5.0 里程碑的文档存储库中的问题](https://github.com/dotnet/EntityFramework.Docs/issues?utf8=%E2%9C%93&q=is%3Aissue+milestone%3A5.0.0+)进行跟踪

T 恤大小：L

状态：正在进行

我们已在针对 3.0 和 3.1 版本更新文档。 我们还在致力于：

* 全面修改入门文档，使它们更易于理解/更易于遵循
* 重新组织文档，使内容更易于查找并添加交叉引用
* 向现有文档添加更多详细信息和说明
* 更新示例并添加更多示例

## <a name="fixing-bugs"></a>修复 bug

通过 [5.0 里程碑中使用 `type-bug` 标记的问题](https://github.com/dotnet/efcore/issues?utf8=%E2%9C%93&q=is%3Aissue+milestone%3A5.0.0+label%3Atype-bug+)进行跟踪

开发人员：@roji、@maumar、@bricelam、@smitpatel、@AndriySvyryd、@ajcvickers

T 恤大小：L

状态：正在进行

撰写本文时，我们已将 135 个 bug 会审为在 5.0 版本中进行修复（已修复了 62 个），但与上面的常规查询增强功能部分存在很大的重叠。

在 3.0 版本期间，传入率（最终成为里程碑工作的问题）大约为每月 23 个问题。 并非所有这些问题都需要在 5.0 中进行修复。 作为大致估计，我们计划修复在 5.0 时间范围内修复其他 150 个问题。

## <a name="small-enhancements"></a>小型增强功能

通过 [5.0 里程碑中使用 `type-enhancement` 标记的问题](https://github.com/dotnet/efcore/issues?utf8=%E2%9C%93&q=is%3Aissue+milestone%3A5.0.0+label%3Atype-enhancement+)进行跟踪

开发人员：@roji、@maumar、@bricelam、@smitpatel、@AndriySvyryd、@ajcvickers

T 恤大小：L

状态：完成

除了上面概述的较大功能之外，我们还计划对 5.0 进行许多较小的改进，以修复“小问题”。 请注意，上面概述的更一般主题也涵盖了其中许多增强功能。

## <a name="below-the-line"></a>非计划

通过[使用 `consider-for-next-release` 标记的问题](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+label%3Aconsider-for-next-release)进行跟踪

这些是当前未针对 5.0 版本计划的 bug 修复和增强功能，但我们将根据以上工作的进度将它们视为延伸目标。

此外，我们始终会在计划时考虑[投票最多的问题](https://github.com/dotnet/efcore/issues?q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc)。 从版本中去除其中任何问题总是很痛苦的，但是我们确实需要针对所拥有的资源制定切合实际的计划。

## <a name="suggestions"></a>建议

你对计划的反馈非常重要。 指示问题重要性的最佳方式是在 GitHub 上为该问题投票（竖起大拇指）。 然后，此数据将进入下一个版本的[计划过程](xref:core/what-is-new/release-planning)。
