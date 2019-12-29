---
title: Microsoft SQL Server 数据库提供程序-Azure SQL 数据库选项-EF Core
description: 如何指定具有 SQL Server Entity Framework Core 数据库提供程序的 Azure SQL 数据库的服务层和性能级别
author: AndriySvyryd
ms.author: ansvyryd
ms.date: 11/05/2019
uid: core/providers/sql-server/azure-sql-database
ms.openlocfilehash: c4f7b91110a0e700ed06130661e611cf45bee05f
ms.sourcegitcommit: 7a709ce4f77134782393aa802df5ab2718714479
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74824785"
---
# <a name="specifying-azure-sql-database-options"></a><span data-ttu-id="d649d-103">指定 Azure SQL 数据库选项</span><span class="sxs-lookup"><span data-stu-id="d649d-103">Specifying Azure SQL Database Options</span></span>

>[!NOTE]
> <span data-ttu-id="d649d-104">此 API 是 EF Core 3.1 中新增的。</span><span class="sxs-lookup"><span data-stu-id="d649d-104">This API is new in EF Core 3.1.</span></span>

<span data-ttu-id="d649d-105">Azure SQL 数据库提供了[各种定价选项](https://azure.microsoft.com/pricing/details/sql-database/single/)，这些选项通常通过 Azure 门户进行配置。</span><span class="sxs-lookup"><span data-stu-id="d649d-105">Azure SQL Database provides [a variety of pricing options](https://azure.microsoft.com/pricing/details/sql-database/single/) that are usually configured through the Azure Portal.</span></span> <span data-ttu-id="d649d-106">但是，如果使用[EF Core 迁移](xref:core/managing-schemas/migrations/index)来管理架构，则可以在模型本身中指定所需的选项。</span><span class="sxs-lookup"><span data-stu-id="d649d-106">However if you are managing the schema using [EF Core migrations](xref:core/managing-schemas/migrations/index) you can specify the desired options in the model itself.</span></span>

<span data-ttu-id="d649d-107">您可以使用[HasServiceTier](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasServiceTier)指定数据库的服务层：</span><span class="sxs-lookup"><span data-stu-id="d649d-107">You can specify the service tier of the database (EDITION) using [HasServiceTier](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasServiceTier):</span></span>

[!code-csharp[HasServiceTier](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasServiceTier)]

<span data-ttu-id="d649d-108">您可以使用[HasDatabaseMaxSize](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasDatabaseMaxSize)指定数据库的最大大小：</span><span class="sxs-lookup"><span data-stu-id="d649d-108">You can specify the maximum size of the database using [HasDatabaseMaxSize](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasDatabaseMaxSize):</span></span>

[!code-csharp[HasDatabaseMaxSize](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasDatabaseMaxSize)]

<span data-ttu-id="d649d-109">您可以使用[HasPerformanceLevel](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasPerformanceLevel)指定数据库的性能级别（SERVICE_OBJECTIVE）：</span><span class="sxs-lookup"><span data-stu-id="d649d-109">You can specify the performance level of the database (SERVICE_OBJECTIVE) using [HasPerformanceLevel](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasPerformanceLevel):</span></span>

[!code-csharp[HasPerformanceLevel](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasPerformanceLevel)]

<span data-ttu-id="d649d-110">使用[HasPerformanceLevelSql](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasPerformanceLevelSql)配置弹性池，因为该值不是字符串文本：</span><span class="sxs-lookup"><span data-stu-id="d649d-110">Use [HasPerformanceLevelSql](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasPerformanceLevelSql) to configure the elastic pool, since the value is not a string literal:</span></span>

[!code-csharp[HasPerformanceLevel](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasPerformanceLevelSql)]


>[!TIP]
> <span data-ttu-id="d649d-111">可以在[ALTER DATABASE 文档](/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current)中找到所有支持的值。</span><span class="sxs-lookup"><span data-stu-id="d649d-111">You can find all the supported values in the [ALTER DATABASE documentation](/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current).</span></span>