---
title: 工具和扩展 - EF Core
description: Entity Framework Core 的外部工具和扩展
author: ErikEJ
ms.date: 04/11/2020
uid: core/extensions/index
ms.openlocfilehash: dc8125ab522b5b0c5126fda2d7c8093c2babdc29
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003297"
---
# <a name="ef-core-tools--extensions"></a>EF Core 工具和扩展

这些工具和扩展为 Framework Core 2.1 及更高版本提供了附加功能。

> [!IMPORTANT]  
> 扩展由各种源构建，不作为 Entity Framework Core 项目的一部分进行维护。 考虑使用第三方扩展时，请务必评估其质量、授权、兼容性和支持等因素，确保其符合要求。 具体而言，为更早版本的 EF Core 构建的扩展可能需要更新，然后才适用于最新版本。

## <a name="tools"></a>工具

### <a name="llblgen-pro"></a>LLBLGen Pro

LLBLGen Pro 是一种实体建模解决方案，包含对 Entity Framework 和 Entity Framework Core 的支持。 借助它可轻松通过 Database First 或 Model First 定义实体模型并将其映射到数据库中，使你可以立即开始编写查询。 对于 EF Core：2、3

[网站](https://www.llblgen.com/)

### <a name="devart-entity-developer"></a>Devart Entity Developer

Entity Developer 是一种用于 ADO.NET Entity Framework、NHibernate、LinqConnect、Telerik 数据访问和 LINQ to SQL 的强大 O/RM 设计器。 它支持 EF Core 模型的直观设计、使用“模型优先”或“数据库优先”的方式，还支持 C# 或 Visual Basic 代码生成。 对于 EF Core：1、2、3、5。

[网站](https://www.devart.com/entitydeveloper/)

### <a name="nhydrate-orm-for-entity-framework"></a>用于 Entity Framework 的 nHydrate ORM

为 Entity Framework 创建强类型的可扩展类的 O/RM。 生成的代码为 Entity Framework Core。 二者没有任何区别。 这不能替代 EF 或自定义 O/RM。 它是一种视觉对象建模层，可让团队管理复杂的数据库架构。 它适用于 Git 等 SCM 软件，允许多用户访问你的模型，并最大限度减少冲突。 安装程序可跟踪模型更改并创建升级脚本。 对于 EF Core：3.

[Github 站点](https://github.com/nHydrate/nHydrate)

### <a name="ef-core-power-tools"></a>EF Core Power Tools

EF Core Power Tools 是一种 Visual Studio 扩展，它在简单用户界面中公开各种 EF Core 设计时任务。 其中包括对现有数据库和 [SQL Server DACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications) 中的 DbContext 和实体类的反向工程、对数据库迁移的管理，以及模型可视化效果。 对于 EF Core：2、3。

[GitHub wiki](https://github.com/ErikEJ/EFCorePowerTools/wiki)

### <a name="entity-framework-visual-editor"></a>实体框架可视化编辑器

Entity Framework Visual Editor 是一种 Visual Studio 扩展，其中增添了 O/RM 设计器用于 EF 6 和 EF Core 类的可视化设计。 代码是通过 T4 模板生成的，因此可自定义来满足任意需求。 它支持继承、单向和双向关联，支持枚举，还能用颜色标识类并添加文本块来解释潜在不可预测的设计部分。 对于 EF Core：2.

[市场](https://marketplace.visualstudio.com/items?itemName=michaelsawczyn.EFDesigner)

### <a name="catfactory"></a>CatFactory

CatFactory 是一种面向 .NET Core 的基架引擎，它可自动基于 SQL Server 数据库生成 DbContext 类、实体、映射配置和存储库类。 对于 EF Core：2.

[GitHub 存储库](https://github.com/hherzl/CatFactory.EntityFrameworkCore)

### <a name="loresofts-entity-framework-core-generator"></a>LoreSoft 的 Entity Framework Core 生成器

Entity Framework Core Generator (efg) 是一种 .NET Core CLI 工具，可基于现有数据库生成 EF Core 模型，其功能与 `dotnet ef dbcontext scaffold` 很相似，但它还支持通过区域替换或分析映射文件来实现安全代码的[重新生成](https://efg.loresoft.com/en/latest/regeneration/)。 此工具支持生成视图模型、验证和对象映射器代码。 对于 EF Core：2.

[教程](https://www.loresoft.com/Generate-ASP-NET-Web-API)
[文档](https://efg.loresoft.com/en/latest/)

## <a name="extensions"></a>扩展

### <a name="microsoftentityframeworkcoreautohistory"></a>Microsoft.EntityFrameworkCore.AutoHistory

一个插件库，它可用于将 EF Core 执行的数据更改自动记录到历史记录表中。 对于 EF Core：2.

[GitHub 存储库](https://github.com/Arch/AutoHistory/)

### <a name="efcoresecondlevelcacheinterceptor"></a>EFCoreSecondLevelCacheInterceptor

二级缓存是一个查询缓存。 EF 命令的结果将存储在该缓存中，这样相同的 EF 命令将从该缓存检索其数据，而不是再次针对数据库进行执行。 对于 EF Core：3.

[GitHub 存储库](https://github.com/VahidN/EFCoreSecondLevelCacheInterceptor)

### <a name="geco"></a>Geco

Geco（生成器控制台）是一个基于控制台项目的简单代码生成器，它在 .NET Core 上运行并使用 C# 内插字符串来生成代码。 Geco 提供面向 EF Core 的反向模型生成器，并支持复数形式、单数形式和可编辑的模板。 它还支持种子数据脚本生成器、脚本运行器和数据库清理器。 对于 EF Core：2.

[GitHub 存储库](https://github.com/iQuarc/Geco)

### <a name="entityframeworkcorescaffoldinghandlebars"></a>EntityFrameworkCore.Scaffolding.Handlebars

允许结合使用 Entity Framework Core 工具链和 Handlebars 模板对基于现有数据库反向工程处理的类进行自定义。 对于 EF Core：2、3。

[GitHub 存储库](https://github.com/TrackableEntities/EntityFrameworkCore.Scaffolding.Handlebars)

### <a name="neinlinqentityframeworkcore"></a>NeinLinq.EntityFrameworkCore

NeinLinq 扩展了 Entity Framewor 等 LINQ 提供程序，让用户能够使用可转换谓词和选择器重复使用函数、重新编写查询并构建动态查询。 对于 EF Core：2、3。

[GitHub 存储库](https://github.com/axelheer/nein-linq/)

### <a name="microsoftentityframeworkcoreunitofwork"></a>Microsoft.EntityFrameworkCore.UnitOfWork

Microsoft.EntityFrameworkCore 的一个插件，它支持存储库、工作模式单元，并支持多个具有具有所支持分布式事务的数据库。 对于 EF Core：2.

[GitHub 存储库](https://github.com/Arch/UnitOfWork/)

### <a name="efcorebulkextensions"></a>EFCore.BulkExtensions

用于批量操作（插入、更新和删除）的 EF Core 插件。 对于 EF Core：2、3。

[GitHub 存储库](https://github.com/borisdj/EFCore.BulkExtensions)

### <a name="bricelamentityframeworkcorepluralizer"></a>Bricelam.EntityFrameworkCore.Pluralizer

添加设计时复数形式。 对于 EF Core：2.

[GitHub 存储库](https://github.com/bricelam/EFCore.Pluralizer)

### <a name="toolbeltentityframeworkcoreindexattribute"></a>Toolbelt.EntityFrameworkCore.IndexAttribute

恢复 [Index] 属性（带有用于模型构建的扩展）。 对于 EF Core：2、3。

[GitHub 存储库](https://github.com/jsakamoto/EntityFrameworkCore.IndexAttribute)

### <a name="efcoreinmemoryhelpers"></a>EfCore.InMemoryHelpers

提供一个面向 EF Core 内存中数据库提供程序的包装器。 使其功能与关系提供程序更类似。 对于 EF Core：2.

[GitHub 存储库](https://github.com/SimonCropp/EfCore.InMemoryHelpers)

### <a name="efcoretemporalsupport"></a>EFCore.TemporalSupport

对时态支持的实现。 对于 EF Core：2.

[GitHub 存储库](https://github.com/cpoDesign/EFCore.TemporalSupport)

### <a name="efcoretemporaltable"></a>EfCoreTemporalTable

使用下列引入的扩展方法对你喜爱的数据库轻松执行时态查询：`AsTemporalAll()`、`AsTemporalAsOf(date)`、`AsTemporalFrom(startDate, endDate)`、`AsTemporalBetween(startDate, endDate)`、`AsTemporalContained(startDate, endDate)`。 对于 EF Core：3.

[GitHub 存储库](https://github.com/glautrou/EfCoreTemporalTable)

### <a name="efcoretimetraveler"></a>EFCore.TimeTraveler

允许使用你已定义的 EF Core 代码、实体和映射对 [SQL Server 时态历史记录](/sql/relational-databases/tables/temporal-table-usage-scenarios#point-in-time-analysis-time-travel)执行功能齐全的 Entity Framework Core 查询。  通过将代码包装到 `using (TemporalQuery.AsOf(targetDateTime)) {...}` 中按时间顺序查看。 对于 EF Core：3.

[GitHub 存储库](https://github.com/VantageSoftware/EFCore.TimeTraveler)

### <a name="entityframeworkcoretemporaltables"></a>EntityFrameworkCore.TemporalTables

适用于 Entity Framework Core 的扩展库，使用 SQL Server 的开发人员可通过它轻松使用时态表。 对于 EF Core：2.

[GitHub 存储库](https://github.com/findulov/EntityFrameworkCore.TemporalTables)

### <a name="entityframeworkcorecacheable"></a>EntityFrameworkCore.Cacheable

高性能二级查询缓存。 对于 EF Core：2.

[GitHub 存储库](https://github.com/SteffenMangold/EntityFrameworkCore.Cacheable)

### <a name="entityframeworkcorencache"></a>EntityFrameworkCore.NCache

NCache Entity Framework Core 提供程序是一个分布式二级缓存提供程序，用于缓存查询结果。 分布式 NCache 体系结构使其更具伸缩性和高可用性。 适用于 EF Core 2。

[网站](https://www.alachisoft.com/ncache/ef-core-cache.html)

### <a name="entity-framework-plus"></a>Entity Framework Plus

扩展 DbContext 的功能，例如：包括筛选器、审核、缓存、查询未来、成批删除、批量更新等。 对于 EF Core：2、3。

[网站](https://entityframework-plus.net/)
[GitHub 存储库](https://github.com/zzzprojects/EntityFramework-Plus)

### <a name="entity-framework-extensions"></a>实体框架扩展

通过高性能批量操作扩展 DbContext：BulkSaveChanges、BulkInsert、BulkUpdate、BulkDelete、BulkMerge 等。 对于 EF Core：2、3。

[网站](https://entityframework-extensions.net/)

### <a name="expressionify"></a>Expressionify

添加了对在 LINQ lambda 中调用扩展方法的支持。 对于 EF Core：3.

[GitHub 存储库](https://github.com/ClaveConsulting/Expressionify)

### <a name="xlinq"></a>XLinq

适用于关系数据库的语言集成查询 (LINQ) 技术。 它允许你使用 C# 编写强类型查询。 对于 EF Core：3.

- 完全支持使用 C# 创建查询：可在 lambda 表达式内使用多个语句，还可使用变量、函数等。
- 与 SQL 之间不存在语义缺口。 XLinq 将 SQL 语句（如 `SELECT`、`FROM`、`WHERE`）声明为第一类 C# 方法，将熟悉的语法与 intellisense、类型安全性和重构结合起来。

因此，SQL 成为了“又一个”本地公开其 API 的类库，可以说是“集成了语言的 SQL”。

[网站](http://xlinq.live/)

### <a name="ramses"></a>Ramses

生命周期挂钩（用于 SaveChanges）。 对于 EF Core：2、3。

[GitHub 存储库](https://github.com/JValck/Ramses)

### <a name="efcorenamingconventions"></a>EFCore.NamingConventions

这会自动使所有表和列的名称都有 snake_case、全部大写或全部小写命名。 对于 EF Core：3.

[GitHub 存储库](https://github.com/efcore/EFCore.NamingConventions)

### <a name="simplersoftwareentityframeworkcoresqlservernodatime"></a>SimplerSoftware.EntityFrameworkCore.SqlServer.NodaTime

为 NodaTime 类型的 SQL Server 添加对 EntityFrameworkCore 的本机支持。 对于 EF Core：3.

[GitHub 存储库](https://github.com/StevenRasmussen/EFCore.SqlServer.NodaTime)

### <a name="dabbleentityframeworkcoretemporalquery"></a>Dabble.EntityFrameworkCore.Temporal.Query

Entity Framework Core 3.1 的 LINQ 扩展，目的是支持 Microsoft SQL Server 临时表查询。 对于 EF Core：3.

[GitHub 存储库](https://github.com/Adam-Langley/efcore-temporal-query)

### <a name="entityframeworkcoresqlserverhierarchyid"></a>EntityFrameworkCore.SqlServer.HierarchyId

向 SQL Server EF Core 提供程序添加 hierarchyid 支持。 对于 EF Core：3.

[GitHub 存储库](https://github.com/efcore/EFCore.SqlServer.HierarchyId)

### <a name="linq2dbentityframeworkcore"></a>linq2db.EntityFrameworkCore

将 LINQ 查询转换为 SQL 表达式的替换转换器。 对于 EF Core：3.

现已开始支持高级 SQL 功能，如 CTE、大容量复制、表提示、窗口函数、临时表和数据库端创建/更新/删除操作。

[GitHub 存储库](https://github.com/linq2db/linq2db.EntityFrameworkCore)

### <a name="efcoresoftdelete"></a>EFCore.SoftDelete

软删除实体的实现。 对于 EF Core：3.

[NuGet](https://www.nuget.org/packages/EFCore.SoftDelete)

### <a name="entityframeworkcoreconfigurationmanager"></a>EntityFrameworkCore.ConfigurationManager

扩展 EF Core 以通过 App.config 解析连接字符串。对于 EF Core：3.

[GitHub 存储库](https://github.com/efcore/EFCore.ConfigurationManager)

### <a name="detached-mapper"></a>分离的映射器

包含组合/聚合处理的 DTO 实体映射器（类似于 GraphDiff）。 对于 EF Core：3、5。

[NuGet](https://www.nuget.org/packages/Detached.Mappers.EntityFramework)
