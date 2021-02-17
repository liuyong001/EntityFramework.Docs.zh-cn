---
title: 应用迁移-EF Core
description: 使用 Entity Framework Core 将架构迁移应用到生产和开发数据库的策略
author: bricelam
ms.date: 05/06/2020
uid: core/managing-schemas/migrations/applying
ms.openlocfilehash: cb4e6f719ba5ab4ef70e2e1d06760db1de5658b4
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543596"
---
# <a name="applying-migrations"></a>应用迁移

添加迁移后，需要对其进行部署并将其应用于数据库。 有多种策略可用于执行此操作，某些策略更适用于生产环境，而另一些则用于开发生命周期。

> [!NOTE]
> 无论部署策略如何，都应始终检查生成的迁移并在应用到生产数据库之前对其进行测试。 如果尝试对列进行重命名，则迁移可能会删除列，如果应用到数据库，则可能会因各种原因而失败。

## <a name="sql-scripts"></a>SQL 脚本

部署到生产数据库的建议方法是生成 SQL 脚本。 此策略的优点包括：

* 可以查看 SQL 脚本的准确性;这一点很重要，因为对生产数据库应用架构更改可能会导致数据丢失。
* 在某些情况下，可以根据生产数据库的特定需求调整这些脚本。
* SQL 脚本可以与部署技术结合使用，甚至可以作为 CI 过程的一部分生成。
* SQL 脚本可提供给 DBA，可以单独管理和存档。

### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

#### <a name="basic-usage"></a>基本用法

下面的内容生成一个从空白数据库到最新迁移的 SQL 脚本：

```dotnetcli
dotnet ef migrations script
```

#### <a name="with-from-to-implied"></a>使用 From（to 隐含）

以下生成从给定的迁移到最新迁移的 SQL 脚本。

```dotnetcli
dotnet ef migrations script AddNewTables
```

#### <a name="with-from-and-to"></a>使用 From 和 To

以下生成从指定的 `from` 迁移到指定迁移的 SQL 脚本 `to` 。

```dotnetcli
dotnet ef migrations script AddNewTables AddAuditTable
```

可以使用比 `to` 新的 `from` 来生成回退脚本。

> [!WARNING]
> 请记下潜在的数据丢失方案。

### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

#### <a name="basic-usage"></a>基本用法

下面的内容生成一个从空白数据库到最新迁移的 SQL 脚本：

```powershell
Script-Migration
```

#### <a name="with-from-to-implied"></a>使用 From（to 隐含）

以下生成从给定的迁移到最新迁移的 SQL 脚本。

```powershell
Script-Migration AddNewTables
```

#### <a name="with-from-and-to"></a>使用 From 和 To

以下生成从指定的 `from` 迁移到指定迁移的 SQL 脚本 `to` 。

```powershell
Script-Migration AddNewTables AddAuditTable
```

可以使用比 `to` 新的 `from` 来生成回退脚本。 请记下潜在的数据丢失方案。

***

脚本生成接受以下两个参数，以指示应生成的迁移范围：

* from 迁移应是运行该脚本前应用到数据库的最后一个迁移。 如果未应用任何迁移，请指定 `0`（默认值）。
* to 迁移是运行该脚本后应用到数据库的最后一个迁移。 它默认为项目中的最后一个迁移。

## <a name="idempotent-sql-scripts"></a>幂等 SQL 脚本

上面生成的 SQL 脚本只能应用于将架构从一个迁移更改到另一个迁移;你需要负责适当地应用脚本，并且仅适用于处于正确迁移状态的数据库。 EF Core 还支持生成 **幂等** 脚本，这些脚本在内部检查是否已通过迁移历史记录表) 应用了哪些迁移 (并仅应用缺少的迁移。 如果您不知道最后一次应用到数据库的迁移，或者要部署到多个数据库（每个数据库都在不同的迁移上），这会很有用。

以下生成幂等迁移：

### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef migrations script --idempotent
```

### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

```powershell
Script-Migration -Idempotent
```

***

## <a name="command-line-tools"></a>命令行工具

EF 命令行工具可用于将迁移应用到数据库。 尽管迁移的本地开发和测试工作效率，但这种方法不适合用于管理生产数据库：

* SQL 命令直接应用于该工具，而不允许开发人员检查或修改它们。 这在生产环境中可能会很危险。
* .NET SDK 和 EF 工具必须安装在生产服务器上。

### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

以下内容将数据库更新到最新迁移：

```dotnetcli
dotnet ef database update
```

以下操作会将数据库更新到给定的迁移：

```dotnetcli
dotnet ef database update AddNewTables
```

请注意，这也可用于回滚到之前的迁移。

> [!WARNING]
> 请记下潜在的数据丢失方案。

### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

以下内容将数据库更新到最新迁移：

```powershell
Update-Database
```

以下操作会将数据库更新到给定的迁移：

```powershell
Update-Database AddNewTables
```

请注意，这也可用于回滚到之前的迁移。

> [!WARNING]
> 请记下潜在的数据丢失方案。

***

有关通过命令行工具应用迁移的详细信息，请参阅 [EF Core 工具参考](xref:core/cli/index)。

## <a name="apply-migrations-at-runtime"></a>在运行时应用迁移

应用程序本身可能会以编程方式（通常是在启动期间）应用迁移。 尽管迁移的本地开发和测试工作效率，但由于以下原因，此方法不适用于管理生产数据库：

* 如果你的应用程序的多个实例正在运行，这两个应用程序都可以尝试同时应用迁移和失败 (或更糟，导致数据损坏) 。
* 同样，如果应用程序在另一个应用程序迁移它时访问数据库，这可能会导致严重问题。
* 应用程序必须具有提升的访问权限才能修改数据库架构。 在生产环境中限制应用程序的数据库权限通常是一种很好的做法。
* 在出现问题时，必须能够回滚应用的迁移。 其他策略可轻松地提供这种情况。
* SQL 命令直接应用于程序，无需为开发人员提供检查或修改的机会。 这在生产环境中可能会很危险。

若要以编程方式应用迁移，请调用 `context.Database.Migrate()` 。 例如，典型的 ASP.NET 应用程序可以执行以下操作：

```csharp
public static void Main(string[] args)
{
    var host = CreateHostBuilder(args).Build();

    using (var scope = host.Services.CreateScope())
    {
        var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
        db.Database.Migrate();
    }

    host.Run();
}
```

请注意， `Migrate()` 在服务的基础上构建 `IMigrator` ，可用于更高级的方案。 请使用 `myDbContext.GetInfrastructure().GetService<IMigrator>()` 进行访问。

> [!WARNING]
>
> * 在生产环境中使用此方法之前，请仔细考虑。 经验表明，此部署策略的简易性有点了它创建的问题。 请考虑从迁移生成 SQL 脚本。
> * 请勿在 `Migrate()` 前调用 `EnsureCreated()`。 `EnsureCreated()` 会绕过迁移创建架构，这会导致 `Migrate()` 失败。
