---
title: EF Core 工具参考 ( .NET CLI) -EF Core
description: Entity Framework Core .NET Core CLI 工具的参考指南
author: bricelam
ms.date: 10/27/2020
uid: core/cli/dotnet
ms.openlocfilehash: 60655c03a7fc29137ccb8d9304c94dac6b803cb2
ms.sourcegitcommit: 7700840119b1639275f3b64836e7abb59103f2e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2021
ms.locfileid: "98983581"
---
# <a name="entity-framework-core-tools-reference---net-core-cli"></a>Entity Framework Core 工具参考-.NET Core CLI

命令行界面 (用于 Entity Framework Core 执行设计时开发任务的 CLI) 工具。 例如，他们基于现有数据库创建 [迁移](/aspnet/core/data/ef-mvc/migrations)、应用迁移和生成模型的代码。 命令是跨平台 [dotnet](/dotnet/core/tools) 命令的扩展，它是 [.NET Core SDK](https://www.microsoft.com/net/core)的一部分。 这些工具适用于 .NET Core 项目。

使用 Visual Studio 时，请考虑使用 [包管理器控制台工具](xref:core/cli/powershell) 代替 CLI 工具。 程序包管理器控制台工具自动：

* 适用于在 **包管理器控制台** 中选择的当前项目，而无需手动切换目录。
* 命令完成后，打开命令生成的文件。
* 提供命令、参数、项目名称、上下文类型和迁移名称的 tab 键完成。

## <a name="installing-the-tools"></a>安装工具

`dotnet ef` 可以安装为全局或本地工具。 大多数开发人员偏向于使用以下命令将 `dotnet ef` 安装为全局工具：

```dotnetcli
dotnet tool install --global dotnet-ef
```

若要将它用作本地工具，请使用[工具清单文件](/dotnet/core/tools/global-tools#install-a-local-tool)恢复声明为工具依赖项的项目的依赖项。

使用以下命令更新工具工具：

```dotnetcli
dotnet tool update --global dotnet-ef
```

你需要将包添加到特定项目上，然后才能使用这些工具 `Microsoft.EntityFrameworkCore.Design` 。

```dotnetcli
dotnet add package Microsoft.EntityFrameworkCore.Design
```

### <a name="verify-installation"></a>验证安装

运行以下命令以验证是否正确安装了 EF Core CLI 工具：

  ```dotnetcli
  dotnet ef
  ```

命令的输出标识所使用的工具的版本：

```output

                     _/\__
               ---==/    \\
         ___  ___   |.    \|\
        | __|| __|  |  )   \\\
        | _| | _|   \_/ |  //|\\
        |___||_|       /   \\\/\\

Entity Framework Core .NET Command-line Tools 2.1.3-rtm-32065

<Usage documentation follows, not shown.>
```

## <a name="update-the-tools"></a>更新工具

使用 `dotnet tool update --global dotnet-ef` 将全局工具更新到最新的可用版本。 如果你在项目中本地安装了这些工具，则使用 `dotnet tool update dotnet-ef` 。 通过追加 `--version <VERSION>` 到命令来安装特定版本。 有关更多详细信息，请参阅 dotnet 工具文档的 [更新](/dotnet/core/tools/dotnet-tool-update) 部分。

## <a name="using-the-tools"></a>使用工具

在使用这些工具之前，您可能需要创建一个启动项目或设置环境。

### <a name="target-project-and-startup-project"></a>目标项目和启动项目

命令引用 *项目* 和 *启动项目*。

* 该 *项目* 也称为 *目标项目* ，因为它是命令在其中添加或删除文件的位置。 默认情况下，当前目录中的项目是目标项目。 可以使用选项指定其他项目作为目标项目 <nobr>`--project`</nobr> 。

* *启动项目* 是工具生成和运行的项目。 这些工具必须在设计时执行应用程序代码，以获取有关项目的信息，例如数据库连接字符串和模型的配置。 默认情况下，当前目录中的项目是启动项目。 您可以使用选项指定其他项目作为启动项目 <nobr>`--startup-project`</nobr> 。

启动项目和目标项目通常是同一个项目。 它们是不同的项目的典型方案是：

* EF Core 的上下文和实体类位于 .NET Core 类库中。
* .NET Core 控制台应用程序或 web 应用程序引用类库。

还可以 [将迁移代码放在与 EF Core 上下文分离](xref:core/managing-schemas/migrations/projects)的类库中。

### <a name="other-target-frameworks"></a>其他目标框架

CLI 工具适用于 .NET Core 项目和 .NET Framework 项目。 .NET Standard 类库中具有 EF Core 模型的应用可能没有 .NET Core 或 .NET Framework 项目。 例如，这适用于 Xamarin 和通用 Windows 平台应用。 在这种情况下，你可以创建一个仅供其使用的 .NET Core 控制台应用项目，作为工具的启动项目。 项目可以是不包含实际代码的虚拟项目， &mdash; 只需为工具提供目标。

为什么需要虚拟项目？ 如前文所述，这些工具必须在设计时执行应用程序代码。 为此，需要使用 .NET Core 运行时。 当 EF Core 模型位于面向 .NET Core 或 .NET Framework 的项目中时，EF Core 工具会借用项目中的运行时。 如果 EF Core 模型在 .NET Standard 类库中，则无法执行此操作。 .NET Standard 不是实际的 .NET 实现;它是 .NET 实现所必须支持的一组 Api 的规范。 因此 .NET Standard 不足以执行应用程序代码 EF Core 工具。 你创建的用于启动项目的虚拟项目提供了一个具体的目标平台，工具可在其中加载 .NET Standard 类库。

### <a name="aspnet-core-environment"></a>ASP.NET Core 环境

若要为 ASP.NET Core 项目指定 [环境](/aspnet/core/fundamentals/environments) ，请在运行命令之前设置 **ASPNETCORE_ENVIRONMENT** 环境变量。

从 EF Core 5.0 开始，还可以将其他参数传递到 CreateHostBuilder，以便在命令行上指定环境：

```dotnetcli
dotnet ef database update -- --environment Production
```

> [!TIP]
> 该 `--` 令牌指示 `dotnet ef` 将后面的所有内容视为一个参数，而不会尝试将它们作为选项进行分析。 不使用的任何额外参数 `dotnet ef` 都将转发到应用程序。

## <a name="common-options"></a>常用选项

| 选项                                         | Short             | 说明                                                                                                                                                                                                                                                   |
|:-----------------------------------------------|:------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--json`                                       |                   | 显示 JSON 输出。                                                                                                                                                                                                                                             |
| `--context <DBCONTEXT>`                        | <nobr>`-c`</nobr> | 要使用的 `DbContext` 类。 仅命名空间或完全限定类名。  如果省略此选项，EF Core 将查找上下文类。 如果有多个上下文类，则需要此选项。                                            |
| `--project <PROJECT>`                          | `-p`              | 目标项目的项目文件夹的相对路径。  默认值为当前文件夹。                                                                                                                                                              |
| `--startup-project <PROJECT>`                  | `-s`              | 启动项目的项目文件夹的相对路径。 默认值为当前文件夹。                                                                                                                                                              |
| `--framework <FRAMEWORK>`                      |                   | [目标框架](/dotnet/standard/frameworks)的[目标框架名字对象](/dotnet/standard/frameworks#supported-target-framework-versions)。  当项目文件指定多个目标框架，并想要选择其中一个时，请使用。 |
| <nobr>`--configuration <CONFIGURATION>`</nobr> |                   | 生成配置，例如： `Debug` 或 `Release` 。                                                                                                                                                                                                   |
| `--runtime <IDENTIFIER>`                       |                   | 要为其还原包的目标运行时的标识符。 有关运行时标识符 (RID) 的列表，请参阅 [RID 目录](/dotnet/core/rid-catalog)。                                                                                                      |
| `--no-build`                                   |                   | 不要生成项目。 应在生成是最新版本时使用。                                                                                                                                                                                    |
| `--help`                                       | `-h`              | 显示帮助信息。                                                                                                                                                                                                                                        |
| `--verbose`                                    | `-v`              | 显示详细输出。                                                                                                                                                                                                                                          |
| `--no-color`                                   |                   | 不要为输出着色。                                                                                                                                                                                                                                        |
| `--prefix-output`                              |                   | 用 level 作为输出前缀。                                                                                                                                                                                                                                     |

从 EF Core 5.0 开始，将任何其他参数传递给应用程序。

## <a name="dotnet-ef-database-drop"></a>dotnet ef 数据库删除

删除数据库。

选项：

| 选项                   | Short             | 说明                                              |
|:-------------------------|:------------------|:---------------------------------------------------------|
| `--force`                | <nobr>`-f`</nobr> | 不要确认。                                           |
| <nobr>`--dry-run`</nobr> |                   | 显示要删除的数据库，但不删除它。 |

上面列出了 [常见选项](#common-options) 。

## <a name="dotnet-ef-database-update"></a>dotnet ef 数据库更新

将数据库更新到上次迁移或指定迁移。

参数：

| 参数                   | 说明                                                                                                                                                                                                                                                     |
|:---------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>`<MIGRATION>`</nobr> | 目标迁移。 可以按名称或 ID 识别迁移。 数字0是一种特殊情况，表示在 *第一次迁移之前* ，并导致还原所有迁移。 如果未指定迁移，则该命令默认为上一次迁移。 |

选项：

| 选项                                    | 说明                                                                                                                      |
|:------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------|
|  <nobr>`--connection <CONNECTION>`</nobr> | 用于连接到数据库的连接字符串。 默认为或中指定的 `AddDbContext` 一个 `OnConfiguring` 。 在 EF Core 5.0 中添加。 |

上面列出了 [常见选项](#common-options) 。

下面的示例将数据库更新为指定的迁移。 第一个使用迁移名称，第二个使用迁移 ID 和指定的连接：

```dotnetcli
dotnet ef database update InitialCreate
dotnet ef database update 20180904195021_InitialCreate --connection your_connection_string
```

## <a name="dotnet-ef-dbcontext-info"></a>dotnet ef dbcontext 信息

获取有关类型的信息 `DbContext` 。

上面列出了 [常见选项](#common-options) 。

## <a name="dotnet-ef-dbcontext-list"></a>dotnet ef dbcontext 列表

列出可用 `DbContext` 的类型。

上面列出了 [常见选项](#common-options) 。

## <a name="dotnet-ef-dbcontext-scaffold"></a>dotnet ef dbcontext 基架

为 `DbContext` 数据库的和实体类型生成代码。 为了使此命令生成实体类型，数据库表必须具有主键。

参数：

| 参数                    | 说明                                                                                                                                                                                                             |
|:----------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>`<CONNECTION>`</nobr> | 用于连接到数据库的连接字符串。 对于 ASP.NET Core 2.x 项目，值可以是 *name = \<name of connection string>*。 在这种情况下，该名称来自为项目设置的配置源。 |
| `<PROVIDER>`                | 要使用的提供程序。 通常，这是 NuGet 包的名称，例如： `Microsoft.EntityFrameworkCore.SqlServer` 。                                                                                           |

选项：

| 选项                                   | Short             | 说明                                                                                                                                                                    |
|:-----------------------------------------|:------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--data-annotations`                     | <nobr>`-d`</nobr> | 使用属性可在可能) 的情况下配置模型 (。 如果省略此选项，则只使用 Fluent API。                                                                |
| `--context <NAME>`                       | `-c`              | `DbContext`要生成的类的名称。                                                                                                                                 |
| `--context-dir <PATH>`                   |                   | 要放入 `DbContext` 类文件的目录。 路径相对于项目目录。 命名空间是从文件夹名称派生的。                                 |
| `--context-namespace <NAMESPACE>`        |                   | 要用于生成的类的命名空间 `DbContext` 。 注意：重写 `--namespace` 。 在 EF Core 5.0 中添加。                                                                 |
| `--force`                                | `-f`              | 覆盖现有文件。                                                                                                                                                      |
| `--output-dir <PATH>`                    | `-o`              | 要在其中放置实体类文件的目录。 路径相对于项目目录。                                                                                       |
| `--namespace <NAMESPACE>`                | `-n`              | 要用于所有生成的类的命名空间。 默认值为从根命名空间和输出目录生成。 在 EF Core 5.0 中添加。                                  |
| <nobr>`--schema <SCHEMA_NAME>...`</nobr> |                   | 要为其生成实体类型的表的架构。 若要指定多个架构，请重复上述操作 `--schema` 。 如果省略此选项，则包括所有架构。          |
| `--table <TABLE_NAME>`...                | `-t`              | 要为其生成实体类型的表。 若要指定多个表，请对 `-t` 每个表重复或 `--table` 。 如果省略此选项，则包括所有表。                |
| `--use-database-names`                   |                   | 使用表和列的名称与数据库中显示的名称完全相同。 如果省略此选项，则更改数据库名称以更严格地符合 c # 名称样式约定。 |
| `--no-onconfiguring`                     |                   | 禁止 `OnConfiguring` 在生成的类中生成方法 `DbContext` 。 在 EF Core 5.0 中添加。                                                                  |
| `--no-pluralize`                         |                   | 请勿使用复数化程序。 添加 EF Core 5。0                                                                                                                                 |

上面列出了 [常见选项](#common-options) 。

下面的示例基架所有架构和表，并将新文件放在 *模型* 文件夹中。

```dotnetcli
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models
```

以下示例仅基架选定的表，并在具有指定名称和命名空间的单独文件夹中创建上下文：

```dotnetcli
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models -t Blog -t Post --context-dir Context -c BlogContext --context-namespace New.Namespace
```

下面的示例使用 [机密管理器工具](/aspnet/core/security/app-secrets#secret-manager)从项目的配置集读取连接字符串。

```dotnetcli
dotnet user-secrets set ConnectionStrings:Blogging "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Blogging"
dotnet ef dbcontext scaffold Name=ConnectionStrings:Blogging Microsoft.EntityFrameworkCore.SqlServer
```

## <a name="dotnet-ef-dbcontext-script"></a>dotnet ef dbcontext 脚本

从 DbContext 生成 SQL 脚本。 绕过任何迁移。 在 EF Core 3.0 中添加。

选项：

| 选项                         | Short             | 说明                      |
| ------------------------------ | ----------------- | -------------------------------- |
| <nobr>`--output <FILE>`</nobr> | <nobr>`-o`</nobr> | 要向其写入结果的文件。 |

上面列出了 [常见选项](#common-options) 。

## <a name="dotnet-ef-migrations-add"></a>dotnet ef 迁移添加

添加新的迁移。

参数：

| 参数              | 说明                |
|:----------------------|:---------------------------|
| <nobr>`<NAME>`</nobr> | 迁移的名称。 |

选项：

| 选项                                 | Short             | 说明                                                                                                            |
|:---------------------------------------|:------------------|:-----------------------------------------------------------------------------------------------------------------------|
| `--output-dir <PATH>`                  | <nobr>`-o`</nobr> | 用于输出文件的目录。 路径相对于目标项目目录。 默认值为 "迁移"。   |
| <nobr>`--namespace <NAMESPACE>`</nobr> | `-n`              | 要用于生成的类的命名空间。 默认为从输出目录生成。 在 EF Core 5.0 中添加。 |

上面列出了 [常见选项](#common-options) 。

## <a name="dotnet-ef-migrations-list"></a>dotnet ef 迁移列表

列出可用迁移。

选项：

| 选项                                   | 说明                                                                                                                  |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| <nobr>`--connection <CONNECTION>`</nobr> | 用于连接到数据库的连接字符串。 默认值为 AddDbContext 或 OnConfiguring 中指定的值。 在 EF Core 5.0 中添加。 |
| `--no-connect`                           | 请勿连接到数据库。 在 EF Core 5.0 中添加。                                                                         |

上面列出了 [常见选项](#common-options) 。

## <a name="dotnet-ef-migrations-remove"></a>dotnet ef 迁移删除

删除上一次迁移 (回滚对迁移) 所做的代码更改。

选项：

| 选项                 | Short             | 说明                                                                     |
|:-----------------------|:------------------|:--------------------------------------------------------------------------------|
| <nobr>`--force`</nobr> | <nobr>`-f`</nobr> | 还原迁移 (回滚应用于数据库的更改) 。 |

上面列出了 [常见选项](#common-options) 。

## <a name="dotnet-ef-migrations-script"></a>dotnet ef 迁移脚本

从迁移生成 SQL 脚本。

参数：

| 参数              | 说明                                                                                                                                                   |
|:----------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>`<FROM>`</nobr> | 开始迁移。 可以按名称或 ID 识别迁移。 数字0是一个特殊情况，表示在 *第一次迁移之前*。 默认值为 0。 |
| `<TO>`                | 结束迁移。 默认为上次迁移。                                                                                                         |

选项：

| 选项                           | Short             | 说明                                                        |
|:---------------------------------|:------------------|:-------------------------------------------------------------------|
| `--output <FILE>`                | <nobr>`-o`</nobr> | 要写入脚本的文件。                                   |
| `--idempotent`                   | `-i`              | 生成可用于任何迁移的数据库的脚本。 |
| <nobr>`--no-transactions`</nobr> |                   | 不生成 SQL transaction 语句。 在 EF Core 5.0 中添加。   |

上面列出了 [常见选项](#common-options) 。

以下示例创建用于 InitialCreate 迁移的脚本：

```dotnetcli
dotnet ef migrations script 0 InitialCreate
```

以下示例在 InitialCreate 迁移之后为所有迁移创建一个脚本。

```dotnetcli
dotnet ef migrations script 20180904195021_InitialCreate
```

## <a name="additional-resources"></a>其他资源

* [迁移](xref:core/managing-schemas/migrations/index)
* [反向工程](xref:core/managing-schemas/scaffolding)
