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
# <a name="ef-core-tools--extensions"></a><span data-ttu-id="96994-103">EF Core 工具和扩展</span><span class="sxs-lookup"><span data-stu-id="96994-103">EF Core Tools & Extensions</span></span>

<span data-ttu-id="96994-104">这些工具和扩展为 Framework Core 2.1 及更高版本提供了附加功能。</span><span class="sxs-lookup"><span data-stu-id="96994-104">These tools and extensions provide additional functionality for Entity Framework Core 2.1 and later.</span></span>

> [!IMPORTANT]  
> <span data-ttu-id="96994-105">扩展由各种源构建，不作为 Entity Framework Core 项目的一部分进行维护。</span><span class="sxs-lookup"><span data-stu-id="96994-105">Extensions are built by a variety of sources and aren't maintained as part of the Entity Framework Core project.</span></span> <span data-ttu-id="96994-106">考虑使用第三方扩展时，请务必评估其质量、授权、兼容性和支持等因素，确保其符合要求。</span><span class="sxs-lookup"><span data-stu-id="96994-106">When considering a third party extension, be sure to evaluate its quality, licensing, compatibility, support, etc. to ensure it meets your requirements.</span></span> <span data-ttu-id="96994-107">具体而言，为更早版本的 EF Core 构建的扩展可能需要更新，然后才适用于最新版本。</span><span class="sxs-lookup"><span data-stu-id="96994-107">In particular, an extension built for an older version of EF Core may need updating before it will work with the latest versions.</span></span>

## <a name="tools"></a><span data-ttu-id="96994-108">工具</span><span class="sxs-lookup"><span data-stu-id="96994-108">Tools</span></span>

### <a name="llblgen-pro"></a><span data-ttu-id="96994-109">LLBLGen Pro</span><span class="sxs-lookup"><span data-stu-id="96994-109">LLBLGen Pro</span></span>

<span data-ttu-id="96994-110">LLBLGen Pro 是一种实体建模解决方案，包含对 Entity Framework 和 Entity Framework Core 的支持。</span><span class="sxs-lookup"><span data-stu-id="96994-110">LLBLGen Pro is an entity modeling solution with support for Entity Framework and Entity Framework Core.</span></span> <span data-ttu-id="96994-111">借助它可轻松通过 Database First 或 Model First 定义实体模型并将其映射到数据库中，使你可以立即开始编写查询。</span><span class="sxs-lookup"><span data-stu-id="96994-111">It lets you easily define your entity model and map it to your database, using database first or model first, so you can get started writing queries right away.</span></span> <span data-ttu-id="96994-112">对于 EF Core：2、3</span><span class="sxs-lookup"><span data-stu-id="96994-112">For EF Core: 2, 3</span></span>

[<span data-ttu-id="96994-113">网站</span><span class="sxs-lookup"><span data-stu-id="96994-113">Website</span></span>](https://www.llblgen.com/)

### <a name="devart-entity-developer"></a><span data-ttu-id="96994-114">Devart Entity Developer</span><span class="sxs-lookup"><span data-stu-id="96994-114">Devart Entity Developer</span></span>

<span data-ttu-id="96994-115">Entity Developer 是一种用于 ADO.NET Entity Framework、NHibernate、LinqConnect、Telerik 数据访问和 LINQ to SQL 的强大 O/RM 设计器。</span><span class="sxs-lookup"><span data-stu-id="96994-115">Entity Developer is a powerful O/RM designer for ADO.NET Entity Framework, NHibernate, LinqConnect, Telerik Data Access, and LINQ to SQL.</span></span> <span data-ttu-id="96994-116">它支持 EF Core 模型的直观设计、使用“模型优先”或“数据库优先”的方式，还支持 C# 或 Visual Basic 代码生成。</span><span class="sxs-lookup"><span data-stu-id="96994-116">It supports designing EF Core models visually, using model first or database first approaches, and C# or Visual Basic code generation.</span></span> <span data-ttu-id="96994-117">对于 EF Core：1、2、3、5。</span><span class="sxs-lookup"><span data-stu-id="96994-117">For EF Core: 1, 2, 3, 5.</span></span>

[<span data-ttu-id="96994-118">网站</span><span class="sxs-lookup"><span data-stu-id="96994-118">Website</span></span>](https://www.devart.com/entitydeveloper/)

### <a name="nhydrate-orm-for-entity-framework"></a><span data-ttu-id="96994-119">用于 Entity Framework 的 nHydrate ORM</span><span class="sxs-lookup"><span data-stu-id="96994-119">nHydrate ORM for Entity Framework</span></span>

<span data-ttu-id="96994-120">为 Entity Framework 创建强类型的可扩展类的 O/RM。</span><span class="sxs-lookup"><span data-stu-id="96994-120">An O/RM that creates strongly-typed, extendable classes for Entity Framework.</span></span> <span data-ttu-id="96994-121">生成的代码为 Entity Framework Core。</span><span class="sxs-lookup"><span data-stu-id="96994-121">The generated code is Entity Framework Core.</span></span> <span data-ttu-id="96994-122">二者没有任何区别。</span><span class="sxs-lookup"><span data-stu-id="96994-122">There is no difference.</span></span> <span data-ttu-id="96994-123">这不能替代 EF 或自定义 O/RM。</span><span class="sxs-lookup"><span data-stu-id="96994-123">This is not a replacement for EF or a custom O/RM.</span></span> <span data-ttu-id="96994-124">它是一种视觉对象建模层，可让团队管理复杂的数据库架构。</span><span class="sxs-lookup"><span data-stu-id="96994-124">It is a visual, modeling layer that allows a team to manage complex database schemas.</span></span> <span data-ttu-id="96994-125">它适用于 Git 等 SCM 软件，允许多用户访问你的模型，并最大限度减少冲突。</span><span class="sxs-lookup"><span data-stu-id="96994-125">It works well with SCM software like Git, allowing multi-user access to your model with minimal conflicts.</span></span> <span data-ttu-id="96994-126">安装程序可跟踪模型更改并创建升级脚本。</span><span class="sxs-lookup"><span data-stu-id="96994-126">The installer tracks model changes and creates upgrade scripts.</span></span> <span data-ttu-id="96994-127">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-127">For EF Core: 3.</span></span>

[<span data-ttu-id="96994-128">Github 站点</span><span class="sxs-lookup"><span data-stu-id="96994-128">Github site</span></span>](https://github.com/nHydrate/nHydrate)

### <a name="ef-core-power-tools"></a><span data-ttu-id="96994-129">EF Core Power Tools</span><span class="sxs-lookup"><span data-stu-id="96994-129">EF Core Power Tools</span></span>

<span data-ttu-id="96994-130">EF Core Power Tools 是一种 Visual Studio 扩展，它在简单用户界面中公开各种 EF Core 设计时任务。</span><span class="sxs-lookup"><span data-stu-id="96994-130">EF Core Power Tools is a Visual Studio extension that exposes various EF Core design-time tasks in a simple user interface.</span></span> <span data-ttu-id="96994-131">其中包括对现有数据库和 [SQL Server DACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications) 中的 DbContext 和实体类的反向工程、对数据库迁移的管理，以及模型可视化效果。</span><span class="sxs-lookup"><span data-stu-id="96994-131">It includes reverse engineering of DbContext and entity classes from existing databases and [SQL Server DACPACs](/sql/relational-databases/data-tier-applications/data-tier-applications), management of database migrations, and model visualizations.</span></span> <span data-ttu-id="96994-132">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="96994-132">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="96994-133">GitHub wiki</span><span class="sxs-lookup"><span data-stu-id="96994-133">GitHub wiki</span></span>](https://github.com/ErikEJ/EFCorePowerTools/wiki)

### <a name="entity-framework-visual-editor"></a><span data-ttu-id="96994-134">实体框架可视化编辑器</span><span class="sxs-lookup"><span data-stu-id="96994-134">Entity Framework Visual Editor</span></span>

<span data-ttu-id="96994-135">Entity Framework Visual Editor 是一种 Visual Studio 扩展，其中增添了 O/RM 设计器用于 EF 6 和 EF Core 类的可视化设计。</span><span class="sxs-lookup"><span data-stu-id="96994-135">Entity Framework Visual Editor is a Visual Studio extension that adds an O/RM designer for visual design of EF 6, and EF Core classes.</span></span> <span data-ttu-id="96994-136">代码是通过 T4 模板生成的，因此可自定义来满足任意需求。</span><span class="sxs-lookup"><span data-stu-id="96994-136">Code is generated using T4 templates so can be customized to suit any needs.</span></span> <span data-ttu-id="96994-137">它支持继承、单向和双向关联，支持枚举，还能用颜色标识类并添加文本块来解释潜在不可预测的设计部分。</span><span class="sxs-lookup"><span data-stu-id="96994-137">It supports inheritance, unidirectional and bidirectional associations, enumerations, and the ability to color-code your classes and add text blocks to explain potentially arcane parts of your design.</span></span> <span data-ttu-id="96994-138">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="96994-138">For EF Core: 2.</span></span>

[<span data-ttu-id="96994-139">市场</span><span class="sxs-lookup"><span data-stu-id="96994-139">Marketplace</span></span>](https://marketplace.visualstudio.com/items?itemName=michaelsawczyn.EFDesigner)

### <a name="catfactory"></a><span data-ttu-id="96994-140">CatFactory</span><span class="sxs-lookup"><span data-stu-id="96994-140">CatFactory</span></span>

<span data-ttu-id="96994-141">CatFactory 是一种面向 .NET Core 的基架引擎，它可自动基于 SQL Server 数据库生成 DbContext 类、实体、映射配置和存储库类。</span><span class="sxs-lookup"><span data-stu-id="96994-141">CatFactory is a scaffolding engine for .NET Core that can automate the generation of DbContext classes, entities, mapping configurations, and repository classes from a SQL Server database.</span></span> <span data-ttu-id="96994-142">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="96994-142">For EF Core: 2.</span></span>

[<span data-ttu-id="96994-143">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-143">GitHub repository</span></span>](https://github.com/hherzl/CatFactory.EntityFrameworkCore)

### <a name="loresofts-entity-framework-core-generator"></a><span data-ttu-id="96994-144">LoreSoft 的 Entity Framework Core 生成器</span><span class="sxs-lookup"><span data-stu-id="96994-144">LoreSoft's Entity Framework Core Generator</span></span>

<span data-ttu-id="96994-145">Entity Framework Core Generator (efg) 是一种 .NET Core CLI 工具，可基于现有数据库生成 EF Core 模型，其功能与 `dotnet ef dbcontext scaffold` 很相似，但它还支持通过区域替换或分析映射文件来实现安全代码的[重新生成](https://efg.loresoft.com/en/latest/regeneration/)。</span><span class="sxs-lookup"><span data-stu-id="96994-145">Entity Framework Core Generator (efg) is a .NET Core CLI tool that can generate EF Core models from an existing database, much like `dotnet ef dbcontext scaffold`, but it also supports safe code [regeneration](https://efg.loresoft.com/en/latest/regeneration/) via region replacement or by parsing mapping files.</span></span> <span data-ttu-id="96994-146">此工具支持生成视图模型、验证和对象映射器代码。</span><span class="sxs-lookup"><span data-stu-id="96994-146">This tool supports generating view models, validation, and object mapper code.</span></span> <span data-ttu-id="96994-147">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="96994-147">For EF Core: 2.</span></span>

<span data-ttu-id="96994-148">[教程](https://www.loresoft.com/Generate-ASP-NET-Web-API)
[文档](https://efg.loresoft.com/en/latest/)</span><span class="sxs-lookup"><span data-stu-id="96994-148">[Tutorial](https://www.loresoft.com/Generate-ASP-NET-Web-API)
[Documentation](https://efg.loresoft.com/en/latest/)</span></span>

## <a name="extensions"></a><span data-ttu-id="96994-149">扩展</span><span class="sxs-lookup"><span data-stu-id="96994-149">Extensions</span></span>

### <a name="microsoftentityframeworkcoreautohistory"></a><span data-ttu-id="96994-150">Microsoft.EntityFrameworkCore.AutoHistory</span><span class="sxs-lookup"><span data-stu-id="96994-150">Microsoft.EntityFrameworkCore.AutoHistory</span></span>

<span data-ttu-id="96994-151">一个插件库，它可用于将 EF Core 执行的数据更改自动记录到历史记录表中。</span><span class="sxs-lookup"><span data-stu-id="96994-151">A plugin library that enables automatically recording the data changes performed by EF Core into a history table.</span></span> <span data-ttu-id="96994-152">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="96994-152">For EF Core: 2.</span></span>

[<span data-ttu-id="96994-153">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-153">GitHub repository</span></span>](https://github.com/Arch/AutoHistory/)

### <a name="efcoresecondlevelcacheinterceptor"></a><span data-ttu-id="96994-154">EFCoreSecondLevelCacheInterceptor</span><span class="sxs-lookup"><span data-stu-id="96994-154">EFCoreSecondLevelCacheInterceptor</span></span>

<span data-ttu-id="96994-155">二级缓存是一个查询缓存。</span><span class="sxs-lookup"><span data-stu-id="96994-155">Second level caching is a query cache.</span></span> <span data-ttu-id="96994-156">EF 命令的结果将存储在该缓存中，这样相同的 EF 命令将从该缓存检索其数据，而不是再次针对数据库进行执行。</span><span class="sxs-lookup"><span data-stu-id="96994-156">The results of EF commands will be stored in the cache, so that the same EF commands will retrieve their data from the cache rather than executing them against the database again.</span></span> <span data-ttu-id="96994-157">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-157">For EF Core: 3.</span></span>

[<span data-ttu-id="96994-158">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-158">GitHub repository</span></span>](https://github.com/VahidN/EFCoreSecondLevelCacheInterceptor)

### <a name="geco"></a><span data-ttu-id="96994-159">Geco</span><span class="sxs-lookup"><span data-stu-id="96994-159">Geco</span></span>

<span data-ttu-id="96994-160">Geco（生成器控制台）是一个基于控制台项目的简单代码生成器，它在 .NET Core 上运行并使用 C# 内插字符串来生成代码。</span><span class="sxs-lookup"><span data-stu-id="96994-160">Geco (Generator Console) is a simple code generator based on a console project, that runs on .NET Core and uses C# interpolated strings for code generation.</span></span> <span data-ttu-id="96994-161">Geco 提供面向 EF Core 的反向模型生成器，并支持复数形式、单数形式和可编辑的模板。</span><span class="sxs-lookup"><span data-stu-id="96994-161">Geco includes a reverse model generator for EF Core with support for pluralization, singularization, and editable templates.</span></span> <span data-ttu-id="96994-162">它还支持种子数据脚本生成器、脚本运行器和数据库清理器。</span><span class="sxs-lookup"><span data-stu-id="96994-162">It also provides a seed data script generator, a script runner, and a database cleaner.</span></span> <span data-ttu-id="96994-163">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="96994-163">For EF Core: 2.</span></span>

[<span data-ttu-id="96994-164">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-164">GitHub repository</span></span>](https://github.com/iQuarc/Geco)

### <a name="entityframeworkcorescaffoldinghandlebars"></a><span data-ttu-id="96994-165">EntityFrameworkCore.Scaffolding.Handlebars</span><span class="sxs-lookup"><span data-stu-id="96994-165">EntityFrameworkCore.Scaffolding.Handlebars</span></span>

<span data-ttu-id="96994-166">允许结合使用 Entity Framework Core 工具链和 Handlebars 模板对基于现有数据库反向工程处理的类进行自定义。</span><span class="sxs-lookup"><span data-stu-id="96994-166">Allows customization of classes reverse engineered from an existing database using the Entity Framework Core toolchain with Handlebars templates.</span></span> <span data-ttu-id="96994-167">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="96994-167">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="96994-168">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-168">GitHub repository</span></span>](https://github.com/TrackableEntities/EntityFrameworkCore.Scaffolding.Handlebars)

### <a name="neinlinqentityframeworkcore"></a><span data-ttu-id="96994-169">NeinLinq.EntityFrameworkCore</span><span class="sxs-lookup"><span data-stu-id="96994-169">NeinLinq.EntityFrameworkCore</span></span>

<span data-ttu-id="96994-170">NeinLinq 扩展了 Entity Framewor 等 LINQ 提供程序，让用户能够使用可转换谓词和选择器重复使用函数、重新编写查询并构建动态查询。</span><span class="sxs-lookup"><span data-stu-id="96994-170">NeinLinq extends LINQ providers such as Entity Framework to enable reusing functions, rewriting queries, and building dynamic queries using translatable predicates and selectors.</span></span> <span data-ttu-id="96994-171">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="96994-171">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="96994-172">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-172">GitHub repository</span></span>](https://github.com/axelheer/nein-linq/)

### <a name="microsoftentityframeworkcoreunitofwork"></a><span data-ttu-id="96994-173">Microsoft.EntityFrameworkCore.UnitOfWork</span><span class="sxs-lookup"><span data-stu-id="96994-173">Microsoft.EntityFrameworkCore.UnitOfWork</span></span>

<span data-ttu-id="96994-174">Microsoft.EntityFrameworkCore 的一个插件，它支持存储库、工作模式单元，并支持多个具有具有所支持分布式事务的数据库。</span><span class="sxs-lookup"><span data-stu-id="96994-174">A plugin for Microsoft.EntityFrameworkCore to support repository, unit of work patterns, and multiple databases with distributed transaction supported.</span></span> <span data-ttu-id="96994-175">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="96994-175">For EF Core: 2.</span></span>

[<span data-ttu-id="96994-176">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-176">GitHub repository</span></span>](https://github.com/Arch/UnitOfWork/)

### <a name="efcorebulkextensions"></a><span data-ttu-id="96994-177">EFCore.BulkExtensions</span><span class="sxs-lookup"><span data-stu-id="96994-177">EFCore.BulkExtensions</span></span>

<span data-ttu-id="96994-178">用于批量操作（插入、更新和删除）的 EF Core 插件。</span><span class="sxs-lookup"><span data-stu-id="96994-178">EF Core extensions for Bulk operations (Insert, Update, Delete).</span></span> <span data-ttu-id="96994-179">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="96994-179">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="96994-180">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-180">GitHub repository</span></span>](https://github.com/borisdj/EFCore.BulkExtensions)

### <a name="bricelamentityframeworkcorepluralizer"></a><span data-ttu-id="96994-181">Bricelam.EntityFrameworkCore.Pluralizer</span><span class="sxs-lookup"><span data-stu-id="96994-181">Bricelam.EntityFrameworkCore.Pluralizer</span></span>

<span data-ttu-id="96994-182">添加设计时复数形式。</span><span class="sxs-lookup"><span data-stu-id="96994-182">Adds design-time pluralization.</span></span> <span data-ttu-id="96994-183">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="96994-183">For EF Core: 2.</span></span>

[<span data-ttu-id="96994-184">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-184">GitHub repository</span></span>](https://github.com/bricelam/EFCore.Pluralizer)

### <a name="toolbeltentityframeworkcoreindexattribute"></a><span data-ttu-id="96994-185">Toolbelt.EntityFrameworkCore.IndexAttribute</span><span class="sxs-lookup"><span data-stu-id="96994-185">Toolbelt.EntityFrameworkCore.IndexAttribute</span></span>

<span data-ttu-id="96994-186">恢复 [Index] 属性（带有用于模型构建的扩展）。</span><span class="sxs-lookup"><span data-stu-id="96994-186">Revival of [Index] attribute (with extension for model building).</span></span> <span data-ttu-id="96994-187">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="96994-187">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="96994-188">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-188">GitHub repository</span></span>](https://github.com/jsakamoto/EntityFrameworkCore.IndexAttribute)

### <a name="efcoreinmemoryhelpers"></a><span data-ttu-id="96994-189">EfCore.InMemoryHelpers</span><span class="sxs-lookup"><span data-stu-id="96994-189">EfCore.InMemoryHelpers</span></span>

<span data-ttu-id="96994-190">提供一个面向 EF Core 内存中数据库提供程序的包装器。</span><span class="sxs-lookup"><span data-stu-id="96994-190">Provides a wrapper around the EF Core In-Memory Database Provider.</span></span> <span data-ttu-id="96994-191">使其功能与关系提供程序更类似。</span><span class="sxs-lookup"><span data-stu-id="96994-191">Makes it act more like a relational provider.</span></span> <span data-ttu-id="96994-192">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="96994-192">For EF Core: 2.</span></span>

[<span data-ttu-id="96994-193">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-193">GitHub repository</span></span>](https://github.com/SimonCropp/EfCore.InMemoryHelpers)

### <a name="efcoretemporalsupport"></a><span data-ttu-id="96994-194">EFCore.TemporalSupport</span><span class="sxs-lookup"><span data-stu-id="96994-194">EFCore.TemporalSupport</span></span>

<span data-ttu-id="96994-195">对时态支持的实现。</span><span class="sxs-lookup"><span data-stu-id="96994-195">An implementation of temporal support.</span></span> <span data-ttu-id="96994-196">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="96994-196">For EF Core: 2.</span></span>

[<span data-ttu-id="96994-197">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-197">GitHub repository</span></span>](https://github.com/cpoDesign/EFCore.TemporalSupport)

### <a name="efcoretemporaltable"></a><span data-ttu-id="96994-198">EfCoreTemporalTable</span><span class="sxs-lookup"><span data-stu-id="96994-198">EfCoreTemporalTable</span></span>

<span data-ttu-id="96994-199">使用下列引入的扩展方法对你喜爱的数据库轻松执行时态查询：`AsTemporalAll()`、`AsTemporalAsOf(date)`、`AsTemporalFrom(startDate, endDate)`、`AsTemporalBetween(startDate, endDate)`、`AsTemporalContained(startDate, endDate)`。</span><span class="sxs-lookup"><span data-stu-id="96994-199">Easily perform temporal queries on your favourite database using introduced extension methods: `AsTemporalAll()`, `AsTemporalAsOf(date)`, `AsTemporalFrom(startDate, endDate)`, `AsTemporalBetween(startDate, endDate)`, `AsTemporalContained(startDate, endDate)`.</span></span> <span data-ttu-id="96994-200">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-200">For EF Core: 3.</span></span>

[<span data-ttu-id="96994-201">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-201">GitHub repository</span></span>](https://github.com/glautrou/EfCoreTemporalTable)

### <a name="efcoretimetraveler"></a><span data-ttu-id="96994-202">EFCore.TimeTraveler</span><span class="sxs-lookup"><span data-stu-id="96994-202">EFCore.TimeTraveler</span></span>

<span data-ttu-id="96994-203">允许使用你已定义的 EF Core 代码、实体和映射对 [SQL Server 时态历史记录](/sql/relational-databases/tables/temporal-table-usage-scenarios#point-in-time-analysis-time-travel)执行功能齐全的 Entity Framework Core 查询。</span><span class="sxs-lookup"><span data-stu-id="96994-203">Allow full-featured Entity Framework Core queries against [SQL Server Temporal History](/sql/relational-databases/tables/temporal-table-usage-scenarios#point-in-time-analysis-time-travel) using the EF Core code, entities, and mappings you already have defined.</span></span>  <span data-ttu-id="96994-204">通过将代码包装到 `using (TemporalQuery.AsOf(targetDateTime)) {...}` 中按时间顺序查看。</span><span class="sxs-lookup"><span data-stu-id="96994-204">Travel through time by wrapping your code in `using (TemporalQuery.AsOf(targetDateTime)) {...}`.</span></span> <span data-ttu-id="96994-205">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-205">For EF Core: 3.</span></span>

[<span data-ttu-id="96994-206">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-206">GitHub repository</span></span>](https://github.com/VantageSoftware/EFCore.TimeTraveler)

### <a name="entityframeworkcoretemporaltables"></a><span data-ttu-id="96994-207">EntityFrameworkCore.TemporalTables</span><span class="sxs-lookup"><span data-stu-id="96994-207">EntityFrameworkCore.TemporalTables</span></span>

<span data-ttu-id="96994-208">适用于 Entity Framework Core 的扩展库，使用 SQL Server 的开发人员可通过它轻松使用时态表。</span><span class="sxs-lookup"><span data-stu-id="96994-208">Extension library for Entity Framework Core which allows developers who use SQL Server to easily use temporal tables.</span></span> <span data-ttu-id="96994-209">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="96994-209">For EF Core: 2.</span></span>

[<span data-ttu-id="96994-210">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-210">GitHub repository</span></span>](https://github.com/findulov/EntityFrameworkCore.TemporalTables)

### <a name="entityframeworkcorecacheable"></a><span data-ttu-id="96994-211">EntityFrameworkCore.Cacheable</span><span class="sxs-lookup"><span data-stu-id="96994-211">EntityFrameworkCore.Cacheable</span></span>

<span data-ttu-id="96994-212">高性能二级查询缓存。</span><span class="sxs-lookup"><span data-stu-id="96994-212">A high-performance second-level query cache.</span></span> <span data-ttu-id="96994-213">对于 EF Core：2.</span><span class="sxs-lookup"><span data-stu-id="96994-213">For EF Core: 2.</span></span>

[<span data-ttu-id="96994-214">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-214">GitHub repository</span></span>](https://github.com/SteffenMangold/EntityFrameworkCore.Cacheable)

### <a name="entityframeworkcorencache"></a><span data-ttu-id="96994-215">EntityFrameworkCore.NCache</span><span class="sxs-lookup"><span data-stu-id="96994-215">EntityFrameworkCore.NCache</span></span>

<span data-ttu-id="96994-216">NCache Entity Framework Core 提供程序是一个分布式二级缓存提供程序，用于缓存查询结果。</span><span class="sxs-lookup"><span data-stu-id="96994-216">NCache Entity Framework Core Provider is a distributed second level cache provider for caching query results.</span></span> <span data-ttu-id="96994-217">分布式 NCache 体系结构使其更具伸缩性和高可用性。</span><span class="sxs-lookup"><span data-stu-id="96994-217">The distributed architecture of NCache makes it more scalable and highly available.</span></span> <span data-ttu-id="96994-218">适用于 EF Core 2。</span><span class="sxs-lookup"><span data-stu-id="96994-218">For EF Core 2.</span></span>

[<span data-ttu-id="96994-219">网站</span><span class="sxs-lookup"><span data-stu-id="96994-219">Website</span></span>](https://www.alachisoft.com/ncache/ef-core-cache.html)

### <a name="entity-framework-plus"></a><span data-ttu-id="96994-220">Entity Framework Plus</span><span class="sxs-lookup"><span data-stu-id="96994-220">Entity Framework Plus</span></span>

<span data-ttu-id="96994-221">扩展 DbContext 的功能，例如：包括筛选器、审核、缓存、查询未来、成批删除、批量更新等。</span><span class="sxs-lookup"><span data-stu-id="96994-221">Extends your DbContext with features such as: Include Filter, Auditing, Caching, Query Future, Batch Delete, Batch Update, and more.</span></span> <span data-ttu-id="96994-222">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="96994-222">For EF Core: 2, 3.</span></span>

<span data-ttu-id="96994-223">[网站](https://entityframework-plus.net/)
[GitHub 存储库](https://github.com/zzzprojects/EntityFramework-Plus)</span><span class="sxs-lookup"><span data-stu-id="96994-223">[Website](https://entityframework-plus.net/)
[GitHub repository](https://github.com/zzzprojects/EntityFramework-Plus)</span></span>

### <a name="entity-framework-extensions"></a><span data-ttu-id="96994-224">实体框架扩展</span><span class="sxs-lookup"><span data-stu-id="96994-224">Entity Framework Extensions</span></span>

<span data-ttu-id="96994-225">通过高性能批量操作扩展 DbContext：BulkSaveChanges、BulkInsert、BulkUpdate、BulkDelete、BulkMerge 等。</span><span class="sxs-lookup"><span data-stu-id="96994-225">Extends your DbContext with high-performance bulk operations: BulkSaveChanges, BulkInsert, BulkUpdate, BulkDelete, BulkMerge, and more.</span></span> <span data-ttu-id="96994-226">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="96994-226">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="96994-227">网站</span><span class="sxs-lookup"><span data-stu-id="96994-227">Website</span></span>](https://entityframework-extensions.net/)

### <a name="expressionify"></a><span data-ttu-id="96994-228">Expressionify</span><span class="sxs-lookup"><span data-stu-id="96994-228">Expressionify</span></span>

<span data-ttu-id="96994-229">添加了对在 LINQ lambda 中调用扩展方法的支持。</span><span class="sxs-lookup"><span data-stu-id="96994-229">Add support for calling extension methods in LINQ lambdas.</span></span> <span data-ttu-id="96994-230">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-230">For EF Core: 3.</span></span>

[<span data-ttu-id="96994-231">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-231">GitHub repository</span></span>](https://github.com/ClaveConsulting/Expressionify)

### <a name="xlinq"></a><span data-ttu-id="96994-232">XLinq</span><span class="sxs-lookup"><span data-stu-id="96994-232">XLinq</span></span>

<span data-ttu-id="96994-233">适用于关系数据库的语言集成查询 (LINQ) 技术。</span><span class="sxs-lookup"><span data-stu-id="96994-233">Language Integrated Query (LINQ) technology for relational databases.</span></span> <span data-ttu-id="96994-234">它允许你使用 C# 编写强类型查询。</span><span class="sxs-lookup"><span data-stu-id="96994-234">It allows you to use C# to write strongly typed queries.</span></span> <span data-ttu-id="96994-235">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-235">For EF Core: 3.</span></span>

- <span data-ttu-id="96994-236">完全支持使用 C# 创建查询：可在 lambda 表达式内使用多个语句，还可使用变量、函数等。</span><span class="sxs-lookup"><span data-stu-id="96994-236">Full C# support for query creation: multiple statements inside lambda, variables, functions, etc.</span></span>
- <span data-ttu-id="96994-237">与 SQL 之间不存在语义缺口。</span><span class="sxs-lookup"><span data-stu-id="96994-237">No semantic gap with SQL.</span></span> <span data-ttu-id="96994-238">XLinq 将 SQL 语句（如 `SELECT`、`FROM`、`WHERE`）声明为第一类 C# 方法，将熟悉的语法与 intellisense、类型安全性和重构结合起来。</span><span class="sxs-lookup"><span data-stu-id="96994-238">XLinq declares SQL statements (like `SELECT`, `FROM`, `WHERE`) as first class C# methods, combining familiar syntax with intellisense, type safety and refactoring.</span></span>

<span data-ttu-id="96994-239">因此，SQL 成为了“又一个”本地公开其 API 的类库，可以说是“集成了语言的 SQL”。</span><span class="sxs-lookup"><span data-stu-id="96994-239">As a result SQL becomes just "another" class library exposing its API locally, literally *"Language Integrated SQL"*.</span></span>

[<span data-ttu-id="96994-240">网站</span><span class="sxs-lookup"><span data-stu-id="96994-240">Website</span></span>](http://xlinq.live/)

### <a name="ramses"></a><span data-ttu-id="96994-241">Ramses</span><span class="sxs-lookup"><span data-stu-id="96994-241">Ramses</span></span>

<span data-ttu-id="96994-242">生命周期挂钩（用于 SaveChanges）。</span><span class="sxs-lookup"><span data-stu-id="96994-242">Lifecycle hooks (for SaveChanges).</span></span> <span data-ttu-id="96994-243">对于 EF Core：2、3。</span><span class="sxs-lookup"><span data-stu-id="96994-243">For EF Core: 2, 3.</span></span>

[<span data-ttu-id="96994-244">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-244">GitHub repository</span></span>](https://github.com/JValck/Ramses)

### <a name="efcorenamingconventions"></a><span data-ttu-id="96994-245">EFCore.NamingConventions</span><span class="sxs-lookup"><span data-stu-id="96994-245">EFCore.NamingConventions</span></span>

<span data-ttu-id="96994-246">这会自动使所有表和列的名称都有 snake_case、全部大写或全部小写命名。</span><span class="sxs-lookup"><span data-stu-id="96994-246">This will automatically make all your table and column names have snake_case, all UPPER or all lower case naming.</span></span> <span data-ttu-id="96994-247">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-247">For EF Core: 3.</span></span>

[<span data-ttu-id="96994-248">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-248">GitHub repository</span></span>](https://github.com/efcore/EFCore.NamingConventions)

### <a name="simplersoftwareentityframeworkcoresqlservernodatime"></a><span data-ttu-id="96994-249">SimplerSoftware.EntityFrameworkCore.SqlServer.NodaTime</span><span class="sxs-lookup"><span data-stu-id="96994-249">SimplerSoftware.EntityFrameworkCore.SqlServer.NodaTime</span></span>

<span data-ttu-id="96994-250">为 NodaTime 类型的 SQL Server 添加对 EntityFrameworkCore 的本机支持。</span><span class="sxs-lookup"><span data-stu-id="96994-250">Adds native support to EntityFrameworkCore for SQL Server for the NodaTime types.</span></span> <span data-ttu-id="96994-251">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-251">For EF Core: 3.</span></span>

[<span data-ttu-id="96994-252">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-252">GitHub repository</span></span>](https://github.com/StevenRasmussen/EFCore.SqlServer.NodaTime)

### <a name="dabbleentityframeworkcoretemporalquery"></a><span data-ttu-id="96994-253">Dabble.EntityFrameworkCore.Temporal.Query</span><span class="sxs-lookup"><span data-stu-id="96994-253">Dabble.EntityFrameworkCore.Temporal.Query</span></span>

<span data-ttu-id="96994-254">Entity Framework Core 3.1 的 LINQ 扩展，目的是支持 Microsoft SQL Server 临时表查询。</span><span class="sxs-lookup"><span data-stu-id="96994-254">LINQ extensions to Entity Framework Core 3.1 to support Microsoft SQL Server Temporal Table Querying.</span></span> <span data-ttu-id="96994-255">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-255">For EF Core: 3.</span></span>

[<span data-ttu-id="96994-256">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-256">GitHub repository</span></span>](https://github.com/Adam-Langley/efcore-temporal-query)

### <a name="entityframeworkcoresqlserverhierarchyid"></a><span data-ttu-id="96994-257">EntityFrameworkCore.SqlServer.HierarchyId</span><span class="sxs-lookup"><span data-stu-id="96994-257">EntityFrameworkCore.SqlServer.HierarchyId</span></span>

<span data-ttu-id="96994-258">向 SQL Server EF Core 提供程序添加 hierarchyid 支持。</span><span class="sxs-lookup"><span data-stu-id="96994-258">Adds hierarchyid support to the SQL Server EF Core provider.</span></span> <span data-ttu-id="96994-259">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-259">For EF Core: 3.</span></span>

[<span data-ttu-id="96994-260">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-260">GitHub repository</span></span>](https://github.com/efcore/EFCore.SqlServer.HierarchyId)

### <a name="linq2dbentityframeworkcore"></a><span data-ttu-id="96994-261">linq2db.EntityFrameworkCore</span><span class="sxs-lookup"><span data-stu-id="96994-261">linq2db.EntityFrameworkCore</span></span>

<span data-ttu-id="96994-262">将 LINQ 查询转换为 SQL 表达式的替换转换器。</span><span class="sxs-lookup"><span data-stu-id="96994-262">Alternative translator of LINQ queries to SQL expressions.</span></span> <span data-ttu-id="96994-263">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-263">For EF Core: 3.</span></span>

<span data-ttu-id="96994-264">现已开始支持高级 SQL 功能，如 CTE、大容量复制、表提示、窗口函数、临时表和数据库端创建/更新/删除操作。</span><span class="sxs-lookup"><span data-stu-id="96994-264">Includes support for advanced SQL features such as CTEs, bulk copy, table hints, windowed functions, temporary tables, and database-side create/update/delete operations.</span></span>

[<span data-ttu-id="96994-265">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-265">GitHub repository</span></span>](https://github.com/linq2db/linq2db.EntityFrameworkCore)

### <a name="efcoresoftdelete"></a><span data-ttu-id="96994-266">EFCore.SoftDelete</span><span class="sxs-lookup"><span data-stu-id="96994-266">EFCore.SoftDelete</span></span>

<span data-ttu-id="96994-267">软删除实体的实现。</span><span class="sxs-lookup"><span data-stu-id="96994-267">An implementation for soft deleting entities.</span></span> <span data-ttu-id="96994-268">对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-268">For EF Core: 3.</span></span>

[<span data-ttu-id="96994-269">NuGet</span><span class="sxs-lookup"><span data-stu-id="96994-269">NuGet</span></span>](https://www.nuget.org/packages/EFCore.SoftDelete)

### <a name="entityframeworkcoreconfigurationmanager"></a><span data-ttu-id="96994-270">EntityFrameworkCore.ConfigurationManager</span><span class="sxs-lookup"><span data-stu-id="96994-270">EntityFrameworkCore.ConfigurationManager</span></span>

<span data-ttu-id="96994-271">扩展 EF Core 以通过 App.config 解析连接字符串。对于 EF Core：3.</span><span class="sxs-lookup"><span data-stu-id="96994-271">Extends EF Core to resolve connection strings from App.config. For EF Core: 3.</span></span>

[<span data-ttu-id="96994-272">GitHub 存储库</span><span class="sxs-lookup"><span data-stu-id="96994-272">GitHub repository</span></span>](https://github.com/efcore/EFCore.ConfigurationManager)

### <a name="detached-mapper"></a><span data-ttu-id="96994-273">分离的映射器</span><span class="sxs-lookup"><span data-stu-id="96994-273">Detached Mapper</span></span>

<span data-ttu-id="96994-274">包含组合/聚合处理的 DTO 实体映射器（类似于 GraphDiff）。</span><span class="sxs-lookup"><span data-stu-id="96994-274">A DTO-Entity mapper with composition/aggregation handling (similar to GraphDiff).</span></span> <span data-ttu-id="96994-275">对于 EF Core：3、5。</span><span class="sxs-lookup"><span data-stu-id="96994-275">For EF Core: 3, 5.</span></span>

[<span data-ttu-id="96994-276">NuGet</span><span class="sxs-lookup"><span data-stu-id="96994-276">NuGet</span></span>](https://www.nuget.org/packages/Detached.Mappers.EntityFramework)
