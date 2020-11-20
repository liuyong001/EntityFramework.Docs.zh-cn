---
title: Microsoft SQL Server 数据库提供程序-Azure SQL 数据库选项-EF Core
description: 如何指定具有 SQL Server Entity Framework Core 数据库提供程序的 Azure SQL 数据库的服务层和性能级别
author: AndriySvyryd
ms.date: 11/05/2019
uid: core/providers/sql-server/azure-sql-database
ms.openlocfilehash: ad202336c2c2efdfe17776952f2a65e98222ecc0
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003583"
---
# <a name="specifying-azure-sql-database-options"></a>指定 Azure SQL 数据库选项

>[!NOTE]
> 此 API 是在 EF Core 3.1 中引入的。

Azure SQL 数据库提供了 [各种定价选项](https://azure.microsoft.com/pricing/details/sql-database/single/) ，这些选项通常通过 Azure 门户进行配置。 但是，如果使用 [EF Core 迁移](xref:core/managing-schemas/migrations/index) 来管理架构，则可以在模型本身中指定所需的选项。

您可以使用 [HasServiceTier](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasServiceTier)指定数据库 (版) 的服务层：

[!code-csharp[HasServiceTier](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasServiceTier)]

您可以使用 [HasDatabaseMaxSize](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasDatabaseMaxSize)指定数据库的最大大小：

[!code-csharp[HasDatabaseMaxSize](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasDatabaseMaxSize)]

您可以使用 [HasPerformanceLevel](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasPerformanceLevel))  (SERVICE_OBJECTIVE 指定数据库的性能级别：

[!code-csharp[HasPerformanceLevel](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasPerformanceLevel)]

使用 [HasPerformanceLevelSql](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasPerformanceLevelSql) 配置弹性池，因为该值不是字符串文本：

[!code-csharp[HasPerformanceLevel](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasPerformanceLevelSql)]

>[!TIP]
> 可以在 [ALTER DATABASE 文档](/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current&preserve-view=true)中找到所有支持的值。
