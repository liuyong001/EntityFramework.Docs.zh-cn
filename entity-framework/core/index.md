---
title: Entity Framework Core 概述 - EF Core
description: Entity Framework Core 的一般介绍概述
author: ajcvickers
ms.date: 9/20/2020
uid: core/index
ms.openlocfilehash: f2905fc55419e196a6f5b8883cf2a2a76b8a1ab2
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128597"
---
# <a name="entity-framework-core"></a><span data-ttu-id="45800-103">Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="45800-103">Entity Framework Core</span></span>

<span data-ttu-id="45800-104">Entity Framework (EF) Core 是轻量化、可扩展、[开源](https://github.com/dotnet/efcore)和跨平台版的常用 Entity Framework 数据访问技术。</span><span class="sxs-lookup"><span data-stu-id="45800-104">Entity Framework (EF) Core is a lightweight, extensible, [open source](https://github.com/dotnet/efcore) and cross-platform version of the popular Entity Framework data access technology.</span></span>

<span data-ttu-id="45800-105">EF Core 可用作对象关系映射程序 (O/RM)，这可以实现以下两点：</span><span class="sxs-lookup"><span data-stu-id="45800-105">EF Core can serve as an object-relational mapper (O/RM), which:</span></span>

* <span data-ttu-id="45800-106">使 .NET 开发人员能够使用 .NET 对象处理数据库。</span><span class="sxs-lookup"><span data-stu-id="45800-106">Enables .NET developers to work with a database using .NET objects.</span></span>
* <span data-ttu-id="45800-107">无需再像通常那样编写大部分数据访问代码。</span><span class="sxs-lookup"><span data-stu-id="45800-107">Eliminates the need for most of the data-access code that typically needs to be written.</span></span>

<span data-ttu-id="45800-108">EF Core 支持多个数据库引擎，请参阅[数据库提供程序](xref:core/providers/index)了解详细信息。</span><span class="sxs-lookup"><span data-stu-id="45800-108">EF Core supports many database engines, see [Database Providers](xref:core/providers/index) for details.</span></span>

## <a name="the-model"></a><span data-ttu-id="45800-109">模型</span><span class="sxs-lookup"><span data-stu-id="45800-109">The model</span></span>

<span data-ttu-id="45800-110">对于 EF Core，使用模型执行数据访问。</span><span class="sxs-lookup"><span data-stu-id="45800-110">With EF Core, data access is performed using a model.</span></span> <span data-ttu-id="45800-111">模型由实体类和表示数据库会话的上下文对象构成。</span><span class="sxs-lookup"><span data-stu-id="45800-111">A model is made up of entity classes and a context object that represents a session with the database.</span></span> <span data-ttu-id="45800-112">上下文对象允许查询并保存数据。</span><span class="sxs-lookup"><span data-stu-id="45800-112">The context object allows querying and saving data.</span></span> <span data-ttu-id="45800-113">有关详细信息，请参阅[创建模型](xref:core/modeling/index)。</span><span class="sxs-lookup"><span data-stu-id="45800-113">For more information, see [Creating a Model](xref:core/modeling/index).</span></span>

<span data-ttu-id="45800-114">EF 支持以下模型开发方法：</span><span class="sxs-lookup"><span data-stu-id="45800-114">EF supports the following model development approaches:</span></span>

* <span data-ttu-id="45800-115">从现有数据库生成模型。</span><span class="sxs-lookup"><span data-stu-id="45800-115">Generate a model from an existing database.</span></span>
* <span data-ttu-id="45800-116">对模型手动编码，使其符合数据库。</span><span class="sxs-lookup"><span data-stu-id="45800-116">Hand code a model to match the database.</span></span>
* <span data-ttu-id="45800-117">创建模型后，使用 [EF 迁移](xref:core/managing-schemas/migrations/index)从模型创建数据库。</span><span class="sxs-lookup"><span data-stu-id="45800-117">Once a model is created, use [EF Migrations](xref:core/managing-schemas/migrations/index) to create a database from the model.</span></span> <span data-ttu-id="45800-118">模型发生变化时，迁移可让数据库不断演进。</span><span class="sxs-lookup"><span data-stu-id="45800-118">Migrations allow evolving the database as the model changes.</span></span>

[!code-csharp[Main](../../samples/core/Intro/Model.cs)]

## <a name="querying"></a><span data-ttu-id="45800-119">查询</span><span class="sxs-lookup"><span data-stu-id="45800-119">Querying</span></span>

<span data-ttu-id="45800-120">使用[语言集成查询 (LINQ)](/dotnet/csharp/programming-guide/concepts/linq/) 从数据库检索实体类的实例。</span><span class="sxs-lookup"><span data-stu-id="45800-120">Instances of your entity classes are retrieved from the database using [Language Integrated Query (LINQ)](/dotnet/csharp/programming-guide/concepts/linq/).</span></span> <span data-ttu-id="45800-121">有关详细信息，请参阅[查询数据](xref:core/querying/index)。</span><span class="sxs-lookup"><span data-stu-id="45800-121">For more information, see [Querying Data](xref:core/querying/index).</span></span>

[!code-csharp[Main](../../samples/core/Intro/Program.cs#Querying)]

## <a name="saving-data"></a><span data-ttu-id="45800-122">保存数据</span><span class="sxs-lookup"><span data-stu-id="45800-122">Saving data</span></span>

<span data-ttu-id="45800-123">使用实体类的实例在数据库中创建、删除和修改数据。</span><span class="sxs-lookup"><span data-stu-id="45800-123">Data is created, deleted, and modified in the database using instances of your entity classes.</span></span> <span data-ttu-id="45800-124">有关详细信息，请参阅[保存数据](xref:core/saving/index)。</span><span class="sxs-lookup"><span data-stu-id="45800-124">See [Saving Data](xref:core/saving/index) to learn more.</span></span>

[!code-csharp[Main](../../samples/core/Intro/Program.cs#SavingData)]

## <a name="ef-orm-considerations"></a><span data-ttu-id="45800-125">EF O/RM 注意事项</span><span class="sxs-lookup"><span data-stu-id="45800-125">EF O/RM considerations</span></span>

<span data-ttu-id="45800-126">虽然 EF Core 善长提取许多编程详细信息，但还是有一些适用于任何 O/RM 的最佳做法，可帮助避免生产应用中的常见陷阱：</span><span class="sxs-lookup"><span data-stu-id="45800-126">While EF Core is good at abstracting many programming details, there are some best practices applicable to any O/RM that help to avoid common pitfalls in production apps:</span></span>

* <span data-ttu-id="45800-127">若要在高性能生产应用中构建、调试、分析和迁移数据，必须具备基础数据库服务器的中级知识或更高级别的知识。</span><span class="sxs-lookup"><span data-stu-id="45800-127">Intermediate-level knowledge or higher of the underlying database server is essential to architect, debug, profile, and migrate data in high performance production apps.</span></span> <span data-ttu-id="45800-128">例如，有关主键和外键、约束、索引、标准化、DML 和 DDL 语句、数据类型、分析等方面的知识。</span><span class="sxs-lookup"><span data-stu-id="45800-128">For example, knowledge of primary and foreign keys, constraints, indexes, normalization, DML and DDL statements, data types, profiling, etc.</span></span>
* <span data-ttu-id="45800-129">功能和集成测试：请务必尽可能严密地复制生产环境，以便：</span><span class="sxs-lookup"><span data-stu-id="45800-129">Functional and integration testing:  It's important to replicate the production environment as closely as possible to:</span></span>
  * <span data-ttu-id="45800-130">查找仅在使用特定版本的数据库服务器时才出现的问题。</span><span class="sxs-lookup"><span data-stu-id="45800-130">Find issues in the app that only show up when using a specific versions or edition of the database server .</span></span>
  * <span data-ttu-id="45800-131">在升级 EF Core 和其他依赖项时捕获中断性变更。</span><span class="sxs-lookup"><span data-stu-id="45800-131">Catch breaking changes when upgrading EF Core and other dependencies.</span></span> <span data-ttu-id="45800-132">例如，添加或升级 ASP.NET Core、OData 或 AutoMapper 等框架。</span><span class="sxs-lookup"><span data-stu-id="45800-132">For example, adding or upgrading frameworks like ASP.NET Core, OData, or AutoMapper.</span></span> <span data-ttu-id="45800-133">这些依赖项可能以多种意外方式影响 EF Core。</span><span class="sxs-lookup"><span data-stu-id="45800-133">These dependencies can affect EF Core in unexpected ways.</span></span>
* <span data-ttu-id="45800-134">通过代表性负载进行性能和压力测试。</span><span class="sxs-lookup"><span data-stu-id="45800-134">Performance and stress testing with representative loads.</span></span> <span data-ttu-id="45800-135">某些功能的不成熟用法缩放性不佳。</span><span class="sxs-lookup"><span data-stu-id="45800-135">The naïve usage of some features doesn't scale well.</span></span> <span data-ttu-id="45800-136">例如，多项集合包含内容、大量使用延迟加载、对未编制索引的列执行条件查询、对存储生成的值进行大规模更新和插入、缺乏并发处理、大型模型、缓存策略不充分。</span><span class="sxs-lookup"><span data-stu-id="45800-136">For example, multiple collections Includes, heavy use of lazy loading, conditional queries on non-indexed columns, massive updates and inserts with store-generated values, lack of concurrency handling, large models, inadequate cache policy.</span></span>
* <span data-ttu-id="45800-137">安全评审：例如，连接字符串和其他机密处理、非部署操作的数据库权限、原始 SQL 的输入验证、敏感数据加密。</span><span class="sxs-lookup"><span data-stu-id="45800-137">Security review: For example, handling of connection strings and other secrets, database permissions for non-deployment operation, input validation for raw SQL, encryption for sensitive data.</span></span>
* <span data-ttu-id="45800-138">确保日志记录和诊断充足且可用。</span><span class="sxs-lookup"><span data-stu-id="45800-138">Make sure logging and diagnostics are sufficient and usable.</span></span> <span data-ttu-id="45800-139">例如，适当的日志记录配置、查询标记和 Application Insights。</span><span class="sxs-lookup"><span data-stu-id="45800-139">For example, appropriate logging configuration, query tags, and Application Insights.</span></span>
* <span data-ttu-id="45800-140">错误恢复。</span><span class="sxs-lookup"><span data-stu-id="45800-140">Error recovery.</span></span> <span data-ttu-id="45800-141">为常见故障场景（如版本回退、回退服务器、横向扩展和负载平衡、DoS 缓解和数据备份）准备应急计划。</span><span class="sxs-lookup"><span data-stu-id="45800-141">Prepare contingencies for common failure scenarios such as version rollback, fallback servers, scale-out and load balancing, DoS mitigation, and data backups.</span></span>
* <span data-ttu-id="45800-142">应用程序部署和迁移。</span><span class="sxs-lookup"><span data-stu-id="45800-142">Application deployment and migration.</span></span> <span data-ttu-id="45800-143">规划如何在部署过程中应用迁移；在应用程序启动时执行此操作可能会导致并发问题，并且对于常规操作，这所需的权限比必要权限更高。</span><span class="sxs-lookup"><span data-stu-id="45800-143">Plan out how migrations are going to be applied during deployment; doing it at application start can suffer from concurrency issues and requires higher permissions than necessary for normal operation.</span></span> <span data-ttu-id="45800-144">在迁移期间，使用暂存来辅助从错误中恢复。</span><span class="sxs-lookup"><span data-stu-id="45800-144">Use staging to facilitate recovery from fatal errors during migration.</span></span> <span data-ttu-id="45800-145">有关详细信息，请参阅[应用迁移](xref:core/managing-schemas/migrations/applying)。</span><span class="sxs-lookup"><span data-stu-id="45800-145">For more information, see [Applying Migrations](xref:core/managing-schemas/migrations/applying).</span></span>
* <span data-ttu-id="45800-146">生成的迁移的详细检查和测试。</span><span class="sxs-lookup"><span data-stu-id="45800-146">Detailed examination and testing of generated migrations.</span></span> <span data-ttu-id="45800-147">将迁移应用于生产数据前，应对其进行全面测试。</span><span class="sxs-lookup"><span data-stu-id="45800-147">Migrations should be thoroughly tested before being applied to production data.</span></span> <span data-ttu-id="45800-148">若表中包含生产数据，架构的形状和列类型就不能轻易更改。</span><span class="sxs-lookup"><span data-stu-id="45800-148">The shape of the schema and the column types cannot be easily changed once the tables contain production data.</span></span> <span data-ttu-id="45800-149">例如，在 SQL Server 上，对于映射到字符串和十进制属性的列，`nvarchar(max)` 和 `decimal(18, 2)` 极少成为最佳类型，但这些是 EF 使用的默认值，因为 EF 不了解你的具体情况。</span><span class="sxs-lookup"><span data-stu-id="45800-149">For example, on SQL Server, `nvarchar(max)` and `decimal(18, 2)` are rarely the best types for columns mapped to string and decimal properties, but those are the defaults that EF uses because it doesn't have knowledge of your specific scenario.</span></span>

## <a name="next-steps"></a><span data-ttu-id="45800-150">后续步骤</span><span class="sxs-lookup"><span data-stu-id="45800-150">Next steps</span></span>

<span data-ttu-id="45800-151">有关介绍性教程，请参阅 [Entity Framework Core 入门](xref:core/get-started/overview/first-app)。</span><span class="sxs-lookup"><span data-stu-id="45800-151">For introductory tutorials, see [Getting Started with Entity Framework Core](xref:core/get-started/overview/first-app).</span></span>
