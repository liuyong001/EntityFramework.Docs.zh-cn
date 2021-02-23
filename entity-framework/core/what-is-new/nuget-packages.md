---
title: EF Core NuGet 包
description: 不同 Entity Framework Core NuGet 包的概述
author: ajcvickers
ms.date: 01/21/2021
uid: core/what-is-new/nuget-packages
ms.openlocfilehash: 4b6e210f2324ea97e006d681d399bfdd6918d1b4
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100544580"
---
# <a name="ef-core-nuget-packages"></a>EF Core NuGet 包

Entity Framework Core (EF Core) 以 [NuGet](https://www.nuget.org/) 包的形式提供。 应用程序所需的包取决于：

- 所使用的数据库系统类型（SQL Server、SQLite 等）
- 所需的 EF Core 功能

安装包的常规过程是：

- 确定数据库提供程序并安装相应的包（[见下文](#database-providers)）
- 如果使用关系提供程序，还需安装 [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/) 和 [Microsoft.EntityFrameworkCore.Relational](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Relational/)。 这有助于确保使用一致的版本，同时也意味着 NuGet 会在新的包版本发布时通知你。
- 或者确定所需的工具类型，并为此安装相应的包（见下文）

请参阅 [Entity Framework Core 入门教程](xref:core/get-started/overview/first-app)，获取 EF Core 入门方面的帮助。

## <a name="package-versions"></a>包版本

请务必安装 Microsoft 提供的所有 EF Core 包的同一版本。 例如，如果安装了 5.0.3 版本的 Microsoft.EntityFrameworkCore.SqlServer，则所有其他 Microsoft.EntityFrameworkCore.* 包也必须为 5.0.3 版本。

此外，请确保所有外部包都与所使用的 EF Core 的版本兼容。 特别是，检查外部数据库提供程序是否支持你所使用的 EF Core 版本。 EF Core 的新主版本通常需要更新的数据库提供程序。

> [!WARNING]
> NuGet 不强制使用一致的包版本。 请始终仔细检查你在 `csproj` 文件或等效文件中引用的版本。

## <a name="database-providers"></a>数据库提供程序

EF Core 通过使用“数据库提供程序”支持不同的数据库系统。 每个系统都有自己的数据库提供程序，而提供程序以 NuGet 包的形式提供。 应用程序应安装其中一个或多个提供程序包。

下表列出了常见的数据库提供程序。 有关可用提供程序的更全面列表，请参阅[数据库提供程序](xref:core/providers/index)。

| 数据库系统                   | 包
|:----------------------------------|----------------------
| SQL Server 和 SQL Azure          | [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer)
| SQLite                            | [Microsoft.EntityFrameworkCore.Sqlite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite)
| Azure Cosmos DB                   | [Microsoft.EntityFrameworkCore.Cosmos](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Cosmos)
| PostgreSQL                        | [Npgsql.EntityFrameworkCore.PostgreSQL](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL/)*
| MySQL                             | [Pomelo.EntityFrameworkCore.MySql](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MySql/)*
| EF Core 内存中数据库**      | [Microsoft.EntityFrameworkCore.InMemory](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.InMemory)

*这些是由社区开发和提供的热门高质量开源提供程序。 列出的其他提供程序由 Microsoft 提供。

**仔细考虑是否使用内存中提供程序。 它不是为生产用途而设计的，也可能不是[用于测试的最佳解决方案](xref:core/testing/index)。

## <a name="tools"></a>工具

使用用于 [EF Core 迁移](xref:core/managing-schemas/migrations/index)和[现有数据库中的反向工程（基架）](xref:core/managing-schemas/scaffolding)的工具需要安装相应的工具包：

- 可在 Visual Studio [包管理器控制台](/nuget/consume-packages/install-use-packages-powershell)中使用的 PowerShell 工具的 [Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools/)
- 跨平台命令行工具的 [dotnet-ef](https://www.nuget.org/packages/dotnet-ef/) 和 [Microsoft.EntityFrameworkCore.Design](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Design/)

请参阅 [Entity Framework Core 工具参考](xref:core/cli/index)，详细了解如何使用 EF Core 工具，包括如何在项目中或在全局范围内正确安装 `dotnet-ef` 工具。

> [!TIP]
> 默认情况下，Microsoft.EntityFrameworkCore.Design 包的安装方式有所不同，它不会随应用程序一起部署。 这也意味着，其类型不能在其他项目中传递使用。 如果需要访问包中的类型，请在 `.csproj` 文件或等效文件中使用常规 `PackageReference`。 有关详细信息，请参阅[设计时服务](xref:core/cli/services)。

## <a name="extension-packages"></a>扩展包

Microsoft 和第三方以 NuGet 包的形式发布了许多 [EF Core 扩展](xref:core/extensions/index)。 常用包包括：

| 功能                                | 包 | 附加依赖项
|:---------------------------------------------|---------|------------------------
| 用于延迟加载和更改跟踪的代理 | [Microsoft.EntityFrameworkCore.Proxies](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Proxies/) | [Castle.Core](https://www.nuget.org/packages/Castle.Core/)
| 对 SQL Server 的空间支持               | [Microsoft.EntityFrameworkCore.SqlServer.NetTopologySuite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite.NetTopologySuite/) | [NetTopologySuite](https://www.nuget.org/packages/NetTopologySuite/) 和 [NetTopologySuite.IO.SqlServerBytes](https://www.nuget.org/packages/NetTopologySuite.IO.SqlServerBytes/)
| 对 SQLite 的空间支持                   | [Microsoft.EntityFrameworkCore.Sqlite.NetTopologySuite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite.NetTopologySuite/) | [NetTopologySuite](https://www.nuget.org/packages/NetTopologySuite/) 和 [NetTopologySuite.IO.SpatiaLite](https://www.nuget.org/packages/NetTopologySuite.IO.SpatiaLite/)
| 对 PostgreSQL 的空间支持               | [Npgsql.EntityFrameworkCore.PostgreSQL.NetTopologySuite](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL.NetTopologySuite) | [NetTopologySuite](https://www.nuget.org/packages/NetTopologySuite/) 和 [NetTopologySuite.IO.PostGIS](https://www.nuget.org/packages/NetTopologySuite.IO.PostGIS/)（通过 [Npgsql.NetTopologySuite](https://www.nuget.org/packages/Npgsql.NetTopologySuite/)）
| 对 MySQL 的空间支持                    | [Pomelo.EntityFrameworkCore.MySql.NetTopologySuite](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MySql.NetTopologySuite) | [NetTopologySuite](https://www.nuget.org/packages/NetTopologySuite/)

## <a name="other-packages"></a>其他包

其他 EF Core 包作为数据库提供程序包的依赖项进行拉取。 但是，建议为这些包添加显式包引用，这样 NuGet 在发布新版本时会提供通知。

| 功能                                              | 包
|:-----------------------------------------------------------|--------
| EF Core 基本功能                                | [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/)
| 通用关系数据库功能                   | [Microsoft.EntityFrameworkCore.Relational](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Relational/)
| 用于 EF Core 特性等的轻型包。           | [Microsoft.EntityFrameworkCore.Abstractions](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Abstractions/)
| EF Core 使用情况的 Roslyn 代码分析器                    | [Microsoft.EntityFrameworkCore.Analyzers](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Analyzers/)
| 没有原生 SQLite 依赖项的 EF Core SQLite 提供程序 | [Microsoft.EntityFrameworkCore.Sqlite.Core](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite.Core/)

## <a name="packages-for-database-provider-testing"></a>用于数据库提供程序测试的包

以下包用于测试外部 GitHub 存储库中内置的数据库提供程序。 有关示例，请参阅 [efcore.pg](https://github.com/npgsql/efcore.pg) 和 [Pomelo.EntityFrameworkCore.MySql](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql)。 应用程序不应安装这些包。

| 功能                                              | 包
|:-----------------------------------------------------------|--------
| 测试任何数据库提供程序                            | [Microsoft.EntityFrameworkCore.Specification.Tests](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Specification.Tests/)
| 测试关系数据库提供程序                    | [Microsoft.EntityFrameworkCore.Relational.Specification.Tests](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Relational.Specification.Tests/)

## <a name="obsolete-packages"></a>已过时的包

请勿安装以下已过时的包，如果项目中当前安装了这些包，请将其删除：

- Microsoft.EntityFrameworkCore.Relational.Design
- Microsoft.EntityFrameworkCore.Tools.DotNet
- Microsoft.EntityFrameworkCore.SqlServer.Design
- Microsoft.EntityFrameworkCore.Sqlite.Design
- Microsoft.EntityFrameworkCore.Relational.Design.Specification.Tests
