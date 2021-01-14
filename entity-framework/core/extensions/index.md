---
title: 工具和扩展 - EF Core
description: Entity Framework Core 的外部工具和扩展
author: ErikEJ
ms.date: 01/06/2021
uid: core/extensions/index
ms.openlocfilehash: 1198cd586902cd6222a94225056d076c847c9197
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129013"
---
# <a name="ef-core-tools--extensions"></a><span data-ttu-id="a1bbd-103">EF Core 工具和扩展</span><span class="sxs-lookup"><span data-stu-id="a1bbd-103">EF Core Tools & Extensions</span></span>

<span data-ttu-id="a1bbd-104">这些工具和扩展为 Framework Core 2.1 及更高版本提供了附加功能。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-104">These tools and extensions provide additional functionality for Entity Framework Core 2.1 and later.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="a1bbd-105">扩展由各种源构建，不作为 Entity Framework Core 项目的一部分进行维护。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-105">Extensions are built by a variety of sources and aren't maintained as part of the Entity Framework Core project.</span></span> <span data-ttu-id="a1bbd-106">考虑使用第三方扩展时，请务必评估其质量、授权、兼容性和支持等因素，确保其符合要求。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-106">When considering a third party extension, be sure to evaluate its quality, licensing, compatibility, support, etc. to ensure it meets your requirements.</span></span> <span data-ttu-id="a1bbd-107">具体而言，为更早版本的 EF Core 构建的扩展可能需要更新，然后才适用于最新版本。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-107">In particular, an extension built for an older version of EF Core may need updating before it will work with the latest versions.</span></span>

## <a name="tools"></a><span data-ttu-id="a1bbd-108">工具</span><span class="sxs-lookup"><span data-stu-id="a1bbd-108">Tools</span></span>

### <a name="llblgen-pro"></a><span data-ttu-id="a1bbd-109">LLBLGen Pro</span><span class="sxs-lookup"><span data-stu-id="a1bbd-109">LLBLGen Pro</span></span>

<span data-ttu-id="a1bbd-110">LLBLGen Pro 是一种实体建模解决方案，包含对 Entity Framework 和 Entity Framework Core 的支持。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-110">LLBLGen Pro is an entity modeling solution with support for Entity Framework and Entity Framework Core.</span></span> <span data-ttu-id="a1bbd-111">借助它可轻松通过 Database First 或 Model First 定义实体模型并将其映射到数据库中，使你可以立即开始编写查询。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-111">It lets you easily define your entity model and map it to your database, using database first or model first, so you can get started writing queries right away.</span></span> <span data-ttu-id="a1bbd-112">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-112">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="a1bbd-113">网站</span><span class="sxs-lookup"><span data-stu-id="a1bbd-113">Website</span></span>](https://www.llblgen.com/)

### <a name="devart-entity-developer"></a><span data-ttu-id="a1bbd-114">Devart Entity Developer</span><span class="sxs-lookup"><span data-stu-id="a1bbd-114">Devart Entity Developer</span></span>

<span data-ttu-id="a1bbd-115">Entity Developer 是一种用于 ADO.NET Entity Framework、NHibernate、LinqConnect、Telerik 数据访问和 LINQ to SQL 的强大 O/RM 设计器。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-115">Entity Developer is a powerful O/RM designer for ADO.NET Entity Framework, NHibernate, LinqConnect, Telerik Data Access, and LINQ to SQL.</span></span> <span data-ttu-id="a1bbd-116">它支持 EF Core 模型的直观设计、使用“模型优先”或“数据库优先”的方式，还支持 C# 或 Visual Basic 代码生成。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-116">It supports designing EF Core models visually, using model first or database first approaches, and C# or Visual Basic code generation.</span></span> <span data-ttu-id="a1bbd-117">对于 EF Core：2、3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-117">For EF Core: 2, 3, 5.</span></span>

[<span data-ttu-id="a1bbd-118">网站</span><span class="sxs-lookup"><span data-stu-id="a1bbd-118">Website</span></span>](https://www.devart.com/entitydeveloper/)

### <a name="nhydrate-orm-for-entity-framework"></a><span data-ttu-id="a1bbd-119">用于 Entity Framework 的 nHydrate ORM</span><span class="sxs-lookup"><span data-stu-id="a1bbd-119">nHydrate ORM for Entity Framework</span></span>

<span data-ttu-id="a1bbd-120">为 Entity Framework 创建强类型的可扩展类的 O/RM。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-120">An O/RM that creates strongly-typed, extendable classes for Entity Framework.</span></span> <span data-ttu-id="a1bbd-121">生成的代码为 Entity Framework Core。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-121">The generated code is Entity Framework Core.</span></span> <span data-ttu-id="a1bbd-122">二者没有任何区别。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-122">There is no difference.</span></span> <span data-ttu-id="a1bbd-123">这不能替代 EF 或自定义 O/RM。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-123">This is not a replacement for EF or a custom O/RM.</span></span> <span data-ttu-id="a1bbd-124">它是一种视觉对象建模层，可让团队管理复杂的数据库架构。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-124">It is a visual, modeling layer that allows a team to manage complex database schemas.</span></span> <span data-ttu-id="a1bbd-125">它适用于 Git 等 SCM 软件，允许多用户访问你的模型，并最大限度减少冲突。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-125">It works well with SCM software like Git, allowing multi-user access to your model with minimal conflicts.</span></span> <span data-ttu-id="a1bbd-126">安装程序可跟踪模型更改并创建升级脚本。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-126">The installer tracks model changes and creates upgrade scripts.</span></span> <span data-ttu-id="a1bbd-127">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-127">For EF Core: 3.</span></span>

[<span data-ttu-id="a1bbd-128">Github 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-128">Github repository</span></span>](https://github.com/nHydrate/nHydrate)

### <a name="ef-core-power-tools"></a><span data-ttu-id="a1bbd-129">EF Core Power Tools</span><span class="sxs-lookup"><span data-stu-id="a1bbd-129">EF Core Power Tools</span></span>

<span data-ttu-id="a1bbd-130">EF Core Power Tools 是一种 Visual Studio 扩展，它在简单用户界面中公开各种 EF Core 设计时任务。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-130">EF Core Power Tools is a Visual Studio extension that exposes various EF Core design-time tasks in a simple user interface.</span></span> <span data-ttu-id="a1bbd-131">其中包括对现有数据库和 [SQL Server DACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications) 中的 DbContext 和实体类的反向工程、对数据库迁移的管理，以及模型可视化效果。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-131">It includes reverse engineering of DbContext and entity classes from existing databases and [SQL Server DACPACs](/sql/relational-databases/data-tier-applications/data-tier-applications), management of database migrations, and model visualizations.</span></span> <span data-ttu-id="a1bbd-132">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-132">For EF Core: 3, 5.</span></span>

[<span data-ttu-id="a1bbd-133">GitHub wiki</span><span class="sxs-lookup"><span data-stu-id="a1bbd-133">GitHub wiki</span></span>](https://github.com/ErikEJ/EFCorePowerTools/wiki)

### <a name="entity-framework-visual-editor"></a><span data-ttu-id="a1bbd-134">实体框架可视化编辑器</span><span class="sxs-lookup"><span data-stu-id="a1bbd-134">Entity Framework Visual Editor</span></span>

<span data-ttu-id="a1bbd-135">Entity Framework Visual Editor 是一种 Visual Studio 扩展，其中增添了 O/RM 设计器用于 EF 6 和 EF Core 类的可视化设计。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-135">Entity Framework Visual Editor is a Visual Studio extension that adds an O/RM designer for visual design of EF 6, and EF Core classes.</span></span> <span data-ttu-id="a1bbd-136">代码是通过 T4 模板生成的，因此可自定义来满足任意需求。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-136">Code is generated using T4 templates so can be customized to suit any needs.</span></span> <span data-ttu-id="a1bbd-137">它支持继承、单向和双向关联，支持枚举，还能用颜色标识类并添加文本块来解释潜在不可预测的设计部分。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-137">It supports inheritance, unidirectional and bidirectional associations, enumerations, and the ability to color-code your classes and add text blocks to explain potentially arcane parts of your design.</span></span> <span data-ttu-id="a1bbd-138">对于 EF Core：2、3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-138">For EF Core: 2, 3, 5.</span></span>

[<span data-ttu-id="a1bbd-139">市场</span><span class="sxs-lookup"><span data-stu-id="a1bbd-139">Marketplace</span></span>](https://marketplace.visualstudio.com/items?itemName=michaelsawczyn.EFDesigner)

### <a name="catfactory"></a><span data-ttu-id="a1bbd-140">CatFactory</span><span class="sxs-lookup"><span data-stu-id="a1bbd-140">CatFactory</span></span>

<span data-ttu-id="a1bbd-141">CatFactory 是一种面向 .NET Core 的基架引擎，它可自动基于 SQL Server 数据库生成 DbContext 类、实体、映射配置和存储库类。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-141">CatFactory is a scaffolding engine for .NET Core that can automate the generation of DbContext classes, entities, mapping configurations, and repository classes from a SQL Server database.</span></span> <span data-ttu-id="a1bbd-142">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-142">For EF Core: 2.</span></span>

[<span data-ttu-id="a1bbd-143">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-143">GitHub repository</span></span>](https://github.com/hherzl/CatFactory.EntityFrameworkCore)

### <a name="loresofts-entity-framework-core-generator"></a><span data-ttu-id="a1bbd-144">LoreSoft 的 Entity Framework Core 生成器</span><span class="sxs-lookup"><span data-stu-id="a1bbd-144">LoreSoft's Entity Framework Core Generator</span></span>

<span data-ttu-id="a1bbd-145">Entity Framework Core Generator (efg) 是一种 .NET Core CLI 工具，可基于现有数据库生成 EF Core 模型，其功能与 `dotnet ef dbcontext scaffold` 很相似，但它还支持通过区域替换或分析映射文件来实现安全代码的[重新生成](https://efg.loresoft.com/en/latest/regeneration/)。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-145">Entity Framework Core Generator (efg) is a .NET Core CLI tool that can generate EF Core models from an existing database, much like `dotnet ef dbcontext scaffold`, but it also supports safe code [regeneration](https://efg.loresoft.com/en/latest/regeneration/) via region replacement or by parsing mapping files.</span></span> <span data-ttu-id="a1bbd-146">此工具支持生成视图模型、验证和对象映射器代码。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-146">This tool supports generating view models, validation, and object mapper code.</span></span> <span data-ttu-id="a1bbd-147">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-147">For EF Core: 2.</span></span>

<span data-ttu-id="a1bbd-148">[教程](https://www.loresoft.com/Generate-ASP-NET-Web-API)
[文档](https://efg.loresoft.com/en/latest/)</span><span class="sxs-lookup"><span data-stu-id="a1bbd-148">[Tutorial](https://www.loresoft.com/Generate-ASP-NET-Web-API)
[Documentation](https://efg.loresoft.com/en/latest/)</span></span>

## <a name="extensions"></a><span data-ttu-id="a1bbd-149">扩展</span><span class="sxs-lookup"><span data-stu-id="a1bbd-149">Extensions</span></span>

### <a name="microsoftentityframeworkcoreautohistory"></a><span data-ttu-id="a1bbd-150">Microsoft.EntityFrameworkCore.AutoHistory</span><span class="sxs-lookup"><span data-stu-id="a1bbd-150">Microsoft.EntityFrameworkCore.AutoHistory</span></span>

<span data-ttu-id="a1bbd-151">一个插件库，它可用于将 EF Core 执行的数据更改自动记录到历史记录表中。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-151">A plugin library that enables automatically recording the data changes performed by EF Core into a history table.</span></span> <span data-ttu-id="a1bbd-152">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-152">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="a1bbd-153">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-153">GitHub repository</span></span>](https://github.com/Arch/AutoHistory/)

### <a name="efcoresecondlevelcacheinterceptor"></a><span data-ttu-id="a1bbd-154">EFCoreSecondLevelCacheInterceptor</span><span class="sxs-lookup"><span data-stu-id="a1bbd-154">EFCoreSecondLevelCacheInterceptor</span></span>

<span data-ttu-id="a1bbd-155">二级缓存是一个查询缓存。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-155">Second level caching is a query cache.</span></span> <span data-ttu-id="a1bbd-156">EF 命令的结果将存储在该缓存中，这样相同的 EF 命令将从该缓存检索其数据，而不是再次针对数据库进行执行。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-156">The results of EF commands will be stored in the cache, so that the same EF commands will retrieve their data from the cache rather than executing them against the database again.</span></span> <span data-ttu-id="a1bbd-157">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-157">For EF Core: 3, 5.</span></span>

[<span data-ttu-id="a1bbd-158">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-158">GitHub repository</span></span>](https://github.com/VahidN/EFCoreSecondLevelCacheInterceptor)

### <a name="geco"></a><span data-ttu-id="a1bbd-159">Geco</span><span class="sxs-lookup"><span data-stu-id="a1bbd-159">Geco</span></span>

<span data-ttu-id="a1bbd-160">Geco（生成器控制台）是一个基于控制台项目的简单代码生成器，它在 .NET Core 上运行并使用 C# 内插字符串来生成代码。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-160">Geco (Generator Console) is a simple code generator based on a console project, that runs on .NET Core and uses C# interpolated strings for code generation.</span></span> <span data-ttu-id="a1bbd-161">Geco 提供面向 EF Core 的反向模型生成器，并支持复数形式、单数形式和可编辑的模板。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-161">Geco includes a reverse model generator for EF Core with support for pluralization, singularization, and editable templates.</span></span> <span data-ttu-id="a1bbd-162">它还支持种子数据脚本生成器、脚本运行器和数据库清理器。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-162">It also provides a seed data script generator, a script runner, and a database cleaner.</span></span> <span data-ttu-id="a1bbd-163">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-163">For EF Core: 2.</span></span>

[<span data-ttu-id="a1bbd-164">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-164">GitHub repository</span></span>](https://github.com/iQuarc/Geco)

### <a name="entityframeworkcorescaffoldinghandlebars"></a><span data-ttu-id="a1bbd-165">EntityFrameworkCore.Scaffolding.Handlebars</span><span class="sxs-lookup"><span data-stu-id="a1bbd-165">EntityFrameworkCore.Scaffolding.Handlebars</span></span>

<span data-ttu-id="a1bbd-166">允许结合使用 Entity Framework Core 工具链和 Handlebars 模板对基于现有数据库反向工程处理的类进行自定义。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-166">Allows customization of classes reverse engineered from an existing database using the Entity Framework Core toolchain with Handlebars templates.</span></span> <span data-ttu-id="a1bbd-167">对于 EF Core：2、3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-167">For EF Core: 2, 3, 5.</span></span>

[<span data-ttu-id="a1bbd-168">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-168">GitHub repository</span></span>](https://github.com/TrackableEntities/EntityFrameworkCore.Scaffolding.Handlebars)

### <a name="neinlinqentityframeworkcore"></a><span data-ttu-id="a1bbd-169">NeinLinq.EntityFrameworkCore</span><span class="sxs-lookup"><span data-stu-id="a1bbd-169">NeinLinq.EntityFrameworkCore</span></span>

<span data-ttu-id="a1bbd-170">NeinLinq 扩展了 Entity Framewor 等 LINQ 提供程序，让用户能够使用可转换谓词和选择器重复使用函数、重新编写查询并构建动态查询。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-170">NeinLinq extends LINQ providers such as Entity Framework to enable reusing functions, rewriting queries, and building dynamic queries using translatable predicates and selectors.</span></span> <span data-ttu-id="a1bbd-171">对于 EF Core：2、3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-171">For EF Core: 2, 3, 5.</span></span>

[<span data-ttu-id="a1bbd-172">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-172">GitHub repository</span></span>](https://github.com/axelheer/nein-linq/)

### <a name="microsoftentityframeworkcoreunitofwork"></a><span data-ttu-id="a1bbd-173">Microsoft.EntityFrameworkCore.UnitOfWork</span><span class="sxs-lookup"><span data-stu-id="a1bbd-173">Microsoft.EntityFrameworkCore.UnitOfWork</span></span>

<span data-ttu-id="a1bbd-174">Microsoft.EntityFrameworkCore 的一个插件，它支持存储库、工作模式单元，并支持多个具有具有所支持分布式事务的数据库。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-174">A plugin for Microsoft.EntityFrameworkCore to support repository, unit of work patterns, and multiple databases with distributed transaction supported.</span></span> <span data-ttu-id="a1bbd-175">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-175">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="a1bbd-176">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-176">GitHub repository</span></span>](https://github.com/Arch/UnitOfWork/)

### <a name="efcorebulkextensions"></a><span data-ttu-id="a1bbd-177">EFCore.BulkExtensions</span><span class="sxs-lookup"><span data-stu-id="a1bbd-177">EFCore.BulkExtensions</span></span>

<span data-ttu-id="a1bbd-178">用于批量操作（插入、更新和删除）的 EF Core 插件。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-178">EF Core extensions for Bulk operations (Insert, Update, Delete).</span></span> <span data-ttu-id="a1bbd-179">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-179">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="a1bbd-180">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-180">GitHub repository</span></span>](https://github.com/borisdj/EFCore.BulkExtensions)

### <a name="bricelamentityframeworkcorepluralizer"></a><span data-ttu-id="a1bbd-181">Bricelam.EntityFrameworkCore.Pluralizer</span><span class="sxs-lookup"><span data-stu-id="a1bbd-181">Bricelam.EntityFrameworkCore.Pluralizer</span></span>

<span data-ttu-id="a1bbd-182">添加设计时复数形式。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-182">Adds design-time pluralization.</span></span> <span data-ttu-id="a1bbd-183">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-183">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="a1bbd-184">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-184">GitHub repository</span></span>](https://github.com/bricelam/EFCore.Pluralizer)

### <a name="toolbeltentityframeworkcoreindexattribute"></a><span data-ttu-id="a1bbd-185">Toolbelt.EntityFrameworkCore.IndexAttribute</span><span class="sxs-lookup"><span data-stu-id="a1bbd-185">Toolbelt.EntityFrameworkCore.IndexAttribute</span></span>

<span data-ttu-id="a1bbd-186">恢复 [Index] 属性（带有用于模型构建的扩展）。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-186">Revival of [Index] attribute (with extension for model building).</span></span> <span data-ttu-id="a1bbd-187">对于 EF Core：2、3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-187">For EF Core: 2, 3, 5.</span></span>

[<span data-ttu-id="a1bbd-188">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-188">GitHub repository</span></span>](https://github.com/jsakamoto/EntityFrameworkCore.IndexAttribute)

### <a name="verifyentityframework"></a><span data-ttu-id="a1bbd-189">Verify.EntityFramework</span><span class="sxs-lookup"><span data-stu-id="a1bbd-189">Verify.EntityFramework</span></span>

<span data-ttu-id="a1bbd-190">扩展 [Verify](https://github.com/VerifyTests/Verify)，以允许使用 EntityFramework 进行快照测试。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-190">Extends [Verify](https://github.com/VerifyTests/Verify) to allow snapshot testing with EntityFramework.</span></span> <span data-ttu-id="a1bbd-191">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-191">For EF Core: 3, 5.</span></span>

[<span data-ttu-id="a1bbd-192">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-192">GitHub repository</span></span>](https://github.com/VerifyTests/Verify.EntityFramework)

### <a name="localdb"></a><span data-ttu-id="a1bbd-193">LocalDb</span><span class="sxs-lookup"><span data-stu-id="a1bbd-193">LocalDb</span></span>

<span data-ttu-id="a1bbd-194">提供围绕 [SQL Server Express LocalDB](https://docs.microsoft.com/sql/database-engine/configure-windows/sql-server-express-localdb) 的包装器，以简化针对实体框架的运行测试。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-194">Provides a wrapper around [SQL Server Express LocalDB](https://docs.microsoft.com/sql/database-engine/configure-windows/sql-server-express-localdb) to simplify running tests against Entity Framework.</span></span> <span data-ttu-id="a1bbd-195">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-195">For EF Core: 3, 5.</span></span>

[<span data-ttu-id="a1bbd-196">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-196">GitHub repository</span></span>](https://github.com/SimonCropp/LocalDb)

### <a name="effluentvalidation"></a><span data-ttu-id="a1bbd-197">EfFluentValidation</span><span class="sxs-lookup"><span data-stu-id="a1bbd-197">EfFluentValidation</span></span>

<span data-ttu-id="a1bbd-198">向实体框架添加 [FluentValidation](https://fluentvalidation.net/) 支持。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-198">Adds [FluentValidation](https://fluentvalidation.net/) support to Entity Framework.</span></span> <span data-ttu-id="a1bbd-199">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-199">For EF Core: 3, 5.</span></span>

[<span data-ttu-id="a1bbd-200">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-200">GitHub repository</span></span>](https://github.com/SimonCropp/EfFluentValidation)

### <a name="efcoretemporalsupport"></a><span data-ttu-id="a1bbd-201">EFCore.TemporalSupport</span><span class="sxs-lookup"><span data-stu-id="a1bbd-201">EFCore.TemporalSupport</span></span>

<span data-ttu-id="a1bbd-202">对时态支持的实现。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-202">An implementation of temporal support.</span></span> <span data-ttu-id="a1bbd-203">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-203">For EF Core: 2.</span></span>

[<span data-ttu-id="a1bbd-204">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-204">GitHub repository</span></span>](https://github.com/cpoDesign/EFCore.TemporalSupport)

### <a name="efcoretemporaltable"></a><span data-ttu-id="a1bbd-205">EfCoreTemporalTable</span><span class="sxs-lookup"><span data-stu-id="a1bbd-205">EfCoreTemporalTable</span></span>

<span data-ttu-id="a1bbd-206">使用下列引入的扩展方法对你喜爱的数据库轻松执行时态查询：`AsTemporalAll()`、`AsTemporalAsOf(date)`、`AsTemporalFrom(startDate, endDate)`、`AsTemporalBetween(startDate, endDate)`、`AsTemporalContained(startDate, endDate)`。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-206">Easily perform temporal queries on your favourite database using introduced extension methods: `AsTemporalAll()`, `AsTemporalAsOf(date)`, `AsTemporalFrom(startDate, endDate)`, `AsTemporalBetween(startDate, endDate)`, `AsTemporalContained(startDate, endDate)`.</span></span> <span data-ttu-id="a1bbd-207">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-207">For EF Core: 3.</span></span>

[<span data-ttu-id="a1bbd-208">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-208">GitHub repository</span></span>](https://github.com/glautrou/EfCoreTemporalTable)

### <a name="entityframeworkcoretemporaltables"></a><span data-ttu-id="a1bbd-209">EntityFrameworkCore.TemporalTables</span><span class="sxs-lookup"><span data-stu-id="a1bbd-209">EntityFrameworkCore.TemporalTables</span></span>

<span data-ttu-id="a1bbd-210">适用于 Entity Framework Core 的扩展库，使用 SQL Server 的开发人员可通过它轻松使用时态表。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-210">Extension library for Entity Framework Core which allows developers who use SQL Server to easily use temporal tables.</span></span> <span data-ttu-id="a1bbd-211">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-211">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="a1bbd-212">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-212">GitHub repository</span></span>](https://github.com/findulov/EntityFrameworkCore.TemporalTables)

### <a name="entityframeworkcorecacheable"></a><span data-ttu-id="a1bbd-213">EntityFrameworkCore.Cacheable</span><span class="sxs-lookup"><span data-stu-id="a1bbd-213">EntityFrameworkCore.Cacheable</span></span>

<span data-ttu-id="a1bbd-214">高性能二级查询缓存。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-214">A high-performance second-level query cache.</span></span> <span data-ttu-id="a1bbd-215">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-215">For EF Core: 2.</span></span>

[<span data-ttu-id="a1bbd-216">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-216">GitHub repository</span></span>](https://github.com/SteffenMangold/EntityFrameworkCore.Cacheable)

### <a name="entityframeworkcorencache"></a><span data-ttu-id="a1bbd-217">EntityFrameworkCore.NCache</span><span class="sxs-lookup"><span data-stu-id="a1bbd-217">EntityFrameworkCore.NCache</span></span>

<span data-ttu-id="a1bbd-218">NCache Entity Framework Core 提供程序是一个分布式二级缓存提供程序，用于缓存查询结果。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-218">NCache Entity Framework Core Provider is a distributed second level cache provider for caching query results.</span></span> <span data-ttu-id="a1bbd-219">分布式 NCache 体系结构使其更具伸缩性和高可用性。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-219">The distributed architecture of NCache makes it more scalable and highly available.</span></span> <span data-ttu-id="a1bbd-220">对于 EF Core 2、3。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-220">For EF Core 2, 3.</span></span>

[<span data-ttu-id="a1bbd-221">网站</span><span class="sxs-lookup"><span data-stu-id="a1bbd-221">Website</span></span>](https://www.alachisoft.com/ncache/ef-core-cache.html)

### <a name="entityframeworkcoretriggered"></a><span data-ttu-id="a1bbd-222">EntityFrameworkCore.Triggered</span><span class="sxs-lookup"><span data-stu-id="a1bbd-222">EntityFrameworkCore.Triggered</span></span>

<span data-ttu-id="a1bbd-223">EF Core 的触发器。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-223">Triggers for EF Core.</span></span> <span data-ttu-id="a1bbd-224">在将 DbContext 中的更改提交到数据库之前和之后对其进行响应。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-224">Respond to changes in your DbContext before and after they are committed to the database.</span></span> <span data-ttu-id="a1bbd-225">触发器是完全异步的，支持依赖关系注入、继承、级联等。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-225">Triggers are fully asynchronous and support dependency injection, inheritance, cascading and more.</span></span> <span data-ttu-id="a1bbd-226">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-226">For EF Core: 3, 5.</span></span>

[<span data-ttu-id="a1bbd-227">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-227">GitHub repository</span></span>](https://github.com/koenbeuk/EntityFrameworkCore.Triggered)

### <a name="entity-framework-plus"></a><span data-ttu-id="a1bbd-228">Entity Framework Plus</span><span class="sxs-lookup"><span data-stu-id="a1bbd-228">Entity Framework Plus</span></span>

<span data-ttu-id="a1bbd-229">扩展 DbContext 的功能，例如：包括筛选器、审核、缓存、查询未来、成批删除、批量更新等。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-229">Extends your DbContext with features such as: Include Filter, Auditing, Caching, Query Future, Batch Delete, Batch Update, and more.</span></span> <span data-ttu-id="a1bbd-230">对于 EF Core：2、3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-230">For EF Core: 2, 3, 5.</span></span>

<span data-ttu-id="a1bbd-231">[网站](https://entityframework-plus.net/)
[GitHub 存储库](https://github.com/zzzprojects/EntityFramework-Plus)</span><span class="sxs-lookup"><span data-stu-id="a1bbd-231">[Website](https://entityframework-plus.net/)
[GitHub repository](https://github.com/zzzprojects/EntityFramework-Plus)</span></span>

### <a name="entity-framework-extensions"></a><span data-ttu-id="a1bbd-232">实体框架扩展</span><span class="sxs-lookup"><span data-stu-id="a1bbd-232">Entity Framework Extensions</span></span>

<span data-ttu-id="a1bbd-233">通过高性能批量操作扩展 DbContext：BulkSaveChanges、BulkInsert、BulkUpdate、BulkDelete、BulkMerge 等。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-233">Extends your DbContext with high-performance bulk operations: BulkSaveChanges, BulkInsert, BulkUpdate, BulkDelete, BulkMerge, and more.</span></span> <span data-ttu-id="a1bbd-234">对于 EF Core：2、3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-234">For EF Core: 2, 3, 5.</span></span>

[<span data-ttu-id="a1bbd-235">网站</span><span class="sxs-lookup"><span data-stu-id="a1bbd-235">Website</span></span>](https://entityframework-extensions.net/)

### <a name="expressionify"></a><span data-ttu-id="a1bbd-236">Expressionify</span><span class="sxs-lookup"><span data-stu-id="a1bbd-236">Expressionify</span></span>

<span data-ttu-id="a1bbd-237">添加了对在 LINQ lambda 中调用扩展方法的支持。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-237">Add support for calling extension methods in LINQ lambdas.</span></span> <span data-ttu-id="a1bbd-238">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-238">For EF Core: 3.</span></span>

[<span data-ttu-id="a1bbd-239">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-239">GitHub repository</span></span>](https://github.com/ClaveConsulting/Expressionify)

### <a name="elinq"></a><span data-ttu-id="a1bbd-240">ELinq</span><span class="sxs-lookup"><span data-stu-id="a1bbd-240">ELinq</span></span>

<span data-ttu-id="a1bbd-241">适用于关系数据库的语言集成查询 (LINQ) 技术。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-241">Language Integrated Query (LINQ) technology for relational databases.</span></span> <span data-ttu-id="a1bbd-242">它允许你使用 C# 编写强类型查询。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-242">It allows you to use C# to write strongly typed queries.</span></span> <span data-ttu-id="a1bbd-243">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-243">For EF Core: 3.</span></span>

- <span data-ttu-id="a1bbd-244">完全支持使用 C# 创建查询：可在 lambda 表达式内使用多个语句，还可使用变量、函数等。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-244">Full C# support for query creation: multiple statements inside lambda, variables, functions, etc.</span></span>
- <span data-ttu-id="a1bbd-245">与 SQL 之间不存在语义缺口。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-245">No semantic gap with SQL.</span></span> <span data-ttu-id="a1bbd-246">ELinq 将 SQL 语句（如 `SELECT`、`FROM`、`WHERE`）声明为第一类 C# 方法，将熟悉的语法与 intellisense、类型安全性和重构结合起来。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-246">ELinq declares SQL statements (like `SELECT`, `FROM`, `WHERE`) as first class C# methods, combining familiar syntax with intellisense, type safety and refactoring.</span></span>

<span data-ttu-id="a1bbd-247">因此，SQL 成为了“又一个”本地公开其 API 的类库，可以说是“集成了语言的 SQL”。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-247">As a result SQL becomes just "another" class library exposing its API locally, literally *"Language Integrated SQL"*.</span></span>

[<span data-ttu-id="a1bbd-248">网站</span><span class="sxs-lookup"><span data-stu-id="a1bbd-248">Website</span></span>](https://entitylinq.com/)

### <a name="ramses"></a><span data-ttu-id="a1bbd-249">Ramses</span><span class="sxs-lookup"><span data-stu-id="a1bbd-249">Ramses</span></span>

<span data-ttu-id="a1bbd-250">生命周期挂钩（用于 SaveChanges）。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-250">Lifecycle hooks (for SaveChanges).</span></span> <span data-ttu-id="a1bbd-251">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-251">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="a1bbd-252">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-252">GitHub repository</span></span>](https://github.com/JValck/Ramses)

### <a name="efcorenamingconventions"></a><span data-ttu-id="a1bbd-253">EFCore.NamingConventions</span><span class="sxs-lookup"><span data-stu-id="a1bbd-253">EFCore.NamingConventions</span></span>

<span data-ttu-id="a1bbd-254">这会自动使所有表和列的名称都有 snake_case、全部大写或全部小写命名。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-254">This will automatically make all your table and column names have snake_case, all UPPER or all lower case naming.</span></span> <span data-ttu-id="a1bbd-255">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-255">For EF Core: 3.</span></span>

[<span data-ttu-id="a1bbd-256">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-256">GitHub repository</span></span>](https://github.com/efcore/EFCore.NamingConventions)

### <a name="simplersoftwareentityframeworkcoresqlservernodatime"></a><span data-ttu-id="a1bbd-257">SimplerSoftware.EntityFrameworkCore.SqlServer.NodaTime</span><span class="sxs-lookup"><span data-stu-id="a1bbd-257">SimplerSoftware.EntityFrameworkCore.SqlServer.NodaTime</span></span>

<span data-ttu-id="a1bbd-258">为 NodaTime 类型的 SQL Server 添加对 EntityFrameworkCore 的本机支持。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-258">Adds native support to EntityFrameworkCore for SQL Server for the NodaTime types.</span></span> <span data-ttu-id="a1bbd-259">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-259">For EF Core: 3, 5.</span></span>

[<span data-ttu-id="a1bbd-260">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-260">GitHub repository</span></span>](https://github.com/StevenRasmussen/EFCore.SqlServer.NodaTime)

### <a name="dabbleentityframeworkcoretemporalquery"></a><span data-ttu-id="a1bbd-261">Dabble.EntityFrameworkCore.Temporal.Query</span><span class="sxs-lookup"><span data-stu-id="a1bbd-261">Dabble.EntityFrameworkCore.Temporal.Query</span></span>

<span data-ttu-id="a1bbd-262">Entity Framework Core 3.1 的 LINQ 扩展，目的是支持 Microsoft SQL Server 临时表查询。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-262">LINQ extensions to Entity Framework Core 3.1 to support Microsoft SQL Server Temporal Table Querying.</span></span> <span data-ttu-id="a1bbd-263">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-263">For EF Core: 3.</span></span>

[<span data-ttu-id="a1bbd-264">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-264">GitHub repository</span></span>](https://github.com/Adam-Langley/efcore-temporal-query)

### <a name="entityframeworkcoresqlserverhierarchyid"></a><span data-ttu-id="a1bbd-265">EntityFrameworkCore.SqlServer.HierarchyId</span><span class="sxs-lookup"><span data-stu-id="a1bbd-265">EntityFrameworkCore.SqlServer.HierarchyId</span></span>

<span data-ttu-id="a1bbd-266">向 SQL Server EF Core 提供程序添加 hierarchyid 支持。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-266">Adds hierarchyid support to the SQL Server EF Core provider.</span></span> <span data-ttu-id="a1bbd-267">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-267">For EF Core: 3.</span></span>

[<span data-ttu-id="a1bbd-268">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-268">GitHub repository</span></span>](https://github.com/efcore/EFCore.SqlServer.HierarchyId)

### <a name="linq2dbentityframeworkcore"></a><span data-ttu-id="a1bbd-269">linq2db.EntityFrameworkCore</span><span class="sxs-lookup"><span data-stu-id="a1bbd-269">linq2db.EntityFrameworkCore</span></span>

<span data-ttu-id="a1bbd-270">将 LINQ 查询转换为 SQL 表达式的替换转换器。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-270">Alternative translator of LINQ queries to SQL expressions.</span></span> <span data-ttu-id="a1bbd-271">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-271">For EF Core: 3, 5.</span></span>

<span data-ttu-id="a1bbd-272">现已开始支持高级 SQL 功能，如 CTE、大容量复制、表提示、窗口函数、临时表和数据库端创建/更新/删除操作。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-272">Includes support for advanced SQL features such as CTEs, bulk copy, table hints, windowed functions, temporary tables, and database-side create/update/delete operations.</span></span>

[<span data-ttu-id="a1bbd-273">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-273">GitHub repository</span></span>](https://github.com/linq2db/linq2db.EntityFrameworkCore)

### <a name="efcoresoftdelete"></a><span data-ttu-id="a1bbd-274">EFCore.SoftDelete</span><span class="sxs-lookup"><span data-stu-id="a1bbd-274">EFCore.SoftDelete</span></span>

<span data-ttu-id="a1bbd-275">软删除实体的实现。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-275">An implementation for soft deleting entities.</span></span> <span data-ttu-id="a1bbd-276">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-276">For EF Core: 3.</span></span>

[<span data-ttu-id="a1bbd-277">NuGet</span><span class="sxs-lookup"><span data-stu-id="a1bbd-277">NuGet</span></span>](https://www.nuget.org/packages/EFCore.SoftDelete)

### <a name="entityframeworkcoreconfigurationmanager"></a><span data-ttu-id="a1bbd-278">EntityFrameworkCore.ConfigurationManager</span><span class="sxs-lookup"><span data-stu-id="a1bbd-278">EntityFrameworkCore.ConfigurationManager</span></span>

<span data-ttu-id="a1bbd-279">扩展 EF Core 以通过 App.config 解析连接字符串。对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-279">Extends EF Core to resolve connection strings from App.config. For EF Core: 3.</span></span>

[<span data-ttu-id="a1bbd-280">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-280">GitHub repository</span></span>](https://github.com/efcore/EFCore.ConfigurationManager)

### <a name="detached-mapper"></a><span data-ttu-id="a1bbd-281">分离的映射器</span><span class="sxs-lookup"><span data-stu-id="a1bbd-281">Detached Mapper</span></span>

<span data-ttu-id="a1bbd-282">包含组合/聚合处理的 DTO 实体映射器（类似于 GraphDiff）。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-282">A DTO-Entity mapper with composition/aggregation handling (similar to GraphDiff).</span></span> <span data-ttu-id="a1bbd-283">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-283">For EF Core: 3, 5.</span></span>

[<span data-ttu-id="a1bbd-284">NuGet</span><span class="sxs-lookup"><span data-stu-id="a1bbd-284">NuGet</span></span>](https://www.nuget.org/packages/Detached.Mappers.EntityFramework)

### <a name="entityframeworkcoresqlitenodatime"></a><span data-ttu-id="a1bbd-285">EntityFrameworkCore.Sqlite.NodaTime</span><span class="sxs-lookup"><span data-stu-id="a1bbd-285">EntityFrameworkCore.Sqlite.NodaTime</span></span>

<span data-ttu-id="a1bbd-286">使用 [SQLite](https://sqlite.org) 时，添加对 [NodaTime](https://nodatime.org) 类型的支持。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-286">Adds support for [NodaTime](https://nodatime.org) types when using [SQLite](https://sqlite.org).</span></span> <span data-ttu-id="a1bbd-287">对于 EF Core：5.</span><span class="sxs-lookup"><span data-stu-id="a1bbd-287">For EF Core: 5.</span></span>

[<span data-ttu-id="a1bbd-288">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="a1bbd-288">GitHub repository</span></span>](https://github.com/khellang/EFCore.Sqlite.NodaTime)

### <a name="erikejentityframeworkcoresqlserverdacpac"></a><span data-ttu-id="a1bbd-289">ErikEJ.EntityFrameworkCore.SqlServer.Dacpac</span><span class="sxs-lookup"><span data-stu-id="a1bbd-289">ErikEJ.EntityFrameworkCore.SqlServer.Dacpac</span></span>

<span data-ttu-id="a1bbd-290">从 SQL Server 数据层应用程序包 (.dacpac) 对 EF Core 模型启用反向工程。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-290">Enables reverse engineering an EF Core model from a SQL Server data-tier application package (.dacpac).</span></span> <span data-ttu-id="a1bbd-291">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-291">For EF Core: 3, 5.</span></span>

[<span data-ttu-id="a1bbd-292">GitHub wiki</span><span class="sxs-lookup"><span data-stu-id="a1bbd-292">GitHub wiki</span></span>](https://github.com/ErikEJ/EFCorePowerTools/wiki/ErikEJ.EntityFrameworkCore.SqlServer.Dacpac)

### <a name="erikejentityframeworkcoredgmlbuilder"></a><span data-ttu-id="a1bbd-293">ErikEJ.EntityFrameworkCore.DgmlBuilder</span><span class="sxs-lookup"><span data-stu-id="a1bbd-293">ErikEJ.EntityFrameworkCore.DgmlBuilder</span></span>

<span data-ttu-id="a1bbd-294">生成可视化 DbContext 的 DGML（图形）内容。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-294">Generate DGML (Graph) content that visualizes your DbContext.</span></span> <span data-ttu-id="a1bbd-295">将 AsDgml() 扩展方法添加到 DbContext 类。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-295">Adds the AsDgml() extension method to the DbContext class.</span></span> <span data-ttu-id="a1bbd-296">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="a1bbd-296">For EF Core: 3, 5.</span></span>

[<span data-ttu-id="a1bbd-297">GitHub wiki</span><span class="sxs-lookup"><span data-stu-id="a1bbd-297">GitHub wiki</span></span>](https://github.com/ErikEJ/EFCorePowerTools/wiki/Inspect-your-DbContext-model)
