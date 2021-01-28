---
title: 反向工程-EF Core
description: 使用 Entity Framework Core 从现有数据库反向工程模型
author: bricelam
ms.date: 11/13/2018
uid: core/managing-schemas/scaffolding
ms.openlocfilehash: 4cad43f4b1300e1ef3cb31e60d3e9890826d974d
ms.sourcegitcommit: 7700840119b1639275f3b64836e7abb59103f2e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2021
ms.locfileid: "98983568"
---
# <a name="reverse-engineering"></a>反向工程

反向工程是基架实体类型类的过程，以及基于数据库架构的 DbContext 类。 可以使用 `Scaffold-DbContext` EF Core 程序包管理器控制台的命令来执行该命令 (PMC) 工具或 `dotnet ef dbcontext scaffold` .net 命令行界面的命令 (CLI) 工具。

## <a name="installing"></a>安装

在进行反向工程之前，你需要 (仅) 或[CLI 工具](xref:core/cli/dotnet)将[PMC 工具](xref:core/cli/powershell)安装在 Visual Studio 中。 有关详细信息，请参阅链接。

还需要为要进行反向工程的数据库架构安装适当的 [数据库提供程序](xref:core/providers/index) 。

## <a name="connection-string"></a>连接字符串

该命令的第一个参数是指向数据库的连接字符串。 工具将使用此连接字符串来读取数据库架构。

引号和转义连接字符串的方式取决于您使用哪个 shell 执行命令。 有关详细信息，请参阅 shell 的文档。 例如，PowerShell 要求转义 `$` 字符，而不是 `\` 。

### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef dbcontext scaffold "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Chinook" Microsoft.EntityFrameworkCore.SqlServer
```

### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

```powershell
Scaffold-DbContext 'Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Chinook' Microsoft.EntityFrameworkCore.SqlServer
```

***

### <a name="configuration-and-user-secrets"></a>配置和用户机密

如果有 ASP.NET Core 项目，则可以使用 `Name=<connection-string>` 语法从配置中读取连接字符串。

这很适合用于 [机密管理器工具](/aspnet/core/security/app-secrets#secret-manager) ，使数据库密码与代码库保持分离。

```dotnetcli
dotnet user-secrets set ConnectionStrings:Chinook "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Chinook"
dotnet ef dbcontext scaffold Name=ConnectionStrings:Chinook Microsoft.EntityFrameworkCore.SqlServer
```

## <a name="provider-name"></a>提供程序名称

第二个参数是提供程序名称。 提供程序名称通常与提供程序的 NuGet 包名称相同。

## <a name="specifying-tables"></a>指定表

默认情况下，将数据库架构中的所有表反向工程为实体类型。 您可以通过指定架构和表来限制对哪些表进行反向工程。

### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

`--schema`选项可用于包含架构中的每个表，并 `--table` 可用于包含特定表。

若要包括多个表，请多次指定选项：

```dotnetcli
dotnet ef dbcontext scaffold ... --table Artist --table Album
```

### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

`-Schemas`选项可用于包含架构中的每个表，并 `-Tables` 可用于包含特定表。

若要包括多个表，请使用数组：

```powershell
Scaffold-DbContext ... -Tables Artist, Album
```

**_

## <a name="preserving-names"></a>保留名称

默认情况下，表和列名已固定，以便更好地匹配类型和属性的 .NET 命名约定。 如果 `-UseDatabaseNames` 在 PMC 中指定开关或 `--use-database-names` .NET Core CLI 中的选项，则将禁用此行为，从而尽可能保留原始数据库名称。 无效的 .NET 标识符仍然是固定的，而合成的名称（如导航属性）仍符合 .NET 命名约定。

## <a name="fluent-api-or-data-annotations"></a>熟知 API 或数据批注

默认情况下，实体类型是使用熟知 API 配置的。 指定 `-DataAnnotations` (PMC) 或 `--data-annotations` ( .NET Core CLI) 改为使用数据批注（如果可能）。

例如，使用熟知的 API 将基架：

```csharp
entity.Property(e => e.Title)
    .IsRequired()
    .HasMaxLength(160);
```

使用数据注释时，将基架：

```csharp
[Required]
[StringLength(160)]
public string Title { get; set; }
```

## <a name="dbcontext-name"></a>DbContext 名称

默认情况下，基架 DbContext 类名称将为带有后缀 _Context * 的数据库的名称。 若要指定其他项，请 `-Context` 在 PMC 中使用， `--context` 在 .NET Core CLI 中使用。

## <a name="directories-and-namespaces"></a>目录和命名空间

实体类和 DbContext 类将基架到项目的根目录中，并使用项目的默认命名空间。

### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

你可以使用指定基架类的目录 `--output-dir` ，并 `--context-dir` 可用于将 DbContext 类基架到不同于实体类型类的目录中：

```dotnetcli
dotnet ef dbcontext scaffold ... --context-dir Data --output-dir Models
```

默认情况下，命名空间将为根命名空间加上项目根目录下的任何子目录的名称。 但从 EFCore 5.0 开始，你可以使用重写所有输出类的命名空间 `--namespace` 。 你还可以使用以下方式仅重写 DbContext 类的命名空间 `--context-namespace` ：

```dotnetcli
dotnet ef dbcontext scaffold ... --namespace Your.Namespace --context-namespace Your.DbContext.Namespace
```

### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

你可以使用指定基架类的目录 `-OutputDir` ，并 `-ContextDir` 可用于将 DbContext 类基架到不同于实体类型类的目录中：

```powershell
Scaffold-DbContext ... -ContextDir Data -OutputDir Models
```

默认情况下，命名空间将为根命名空间加上项目根目录下的任何子目录的名称。 但从 EFCore 5.0 开始，你可以使用重写所有输出类的命名空间 `-Namespace` 。 你还可以使用替代 DbContext 类的命名空间 `-ContextNamespace` 。

```powershell
Scaffold-DbContext ... -Namespace Your.Namespace -ContextNamespace Your.DbContext.Namespace
```

***

## <a name="how-it-works"></a>工作原理

反向工程从读取数据库架构开始。 它读取有关表、列、约束和索引的信息。

接下来，它使用架构信息创建 EF Core 模型。 表用于创建实体类型;列用于创建属性;和外键用于创建关系。

最后，模型用于生成代码。 为了从应用程序重新创建相同的模型，相应的实体类型类、熟知 API 和数据批注都是基架的。

## <a name="limitations"></a>限制

* 不是有关模型的所有内容都可以使用数据库架构来表示。 例如，有关 [**继承层次结构**](xref:core/modeling/inheritance)、 [**附属类型**](xref:core/modeling/owned-entities)和 [**表拆分**](xref:core/modeling/table-splitting) 的信息在数据库架构中不存在。 因此，这些构造永远不会经过反向工程。
* 此外，EF Core 提供程序可能不支持 **某些列类型** 。 这些列不会包含在模型中。
* 可以在 EF Core 模型中定义 [**并发标记**](xref:core/modeling/concurrency)，以防止两个用户同时更新同一实体。 某些数据库具有特殊类型来表示此类型的列 (例如，SQL Server 中的 rowversion) ，在这种情况下，我们可以对此信息进行反向工程;但是，其他并发令牌不会进行反向工程。
* 反向工程当前不支持[c # 8 可为 Null 的引用类型功能](/dotnet/csharp/tutorials/nullable-reference-types)： EF Core 始终会生成假定禁用该功能的 c # 代码。 例如，可以将可为 null 的文本列基架为类型为的属性 `string` ，而不是 `string?` 用于配置是否需要属性的熟知 API 或数据批注。 您可以编辑基架代码并将其替换为 c # 为空批注。 [#15520](https://github.com/dotnet/efcore/issues/15520)的问题跟踪了可为 null 的引用类型的基架支持。

## <a name="customizing-the-model"></a>自定义模型

EF Core 生成的代码是您的代码。 随意更改。 仅当您再次对同一模型进行反向工程时，才会重新生成它。 基架代码表示 *一个* 可用于访问数据库的模型，但它当然不是 *唯一* 可以使用的模型。

根据需要自定义实体类型类和 DbContext 类。 例如，你可以选择重命名类型和属性、引入继承层次结构或将表拆分为多个实体。 还可以从模型中删除非唯一索引、未使用的序列和导航属性、可选的标量属性和约束名称。

还可以添加其他构造函数、方法、属性等。 在单独的文件中使用另一个分部类。 即使您要再次对模型进行反向工程，此方法也能正常工作。

## <a name="updating-the-model"></a>更新模型

更改数据库后，可能需要更新 EF Core 模型以反映这些更改。 如果数据库更改很简单，只需手动对 EF Core 模型进行更改即可。 例如，对表或列进行重命名、删除列或更新列的类型是在代码中进行的一些简单的更改。

但是，更重要的更改并不容易手动完成。 一个常见的工作流是通过使用 `-Force` (PMC) 或 `--force` (CLI) ，使用已更新的模型覆盖现有模型，从数据库反向对模型进行反向工程。

另一个常请求的功能是能够从数据库更新模型，同时保留自定义项（如重命名、类型层次结构等）。使用问题 [#831](https://github.com/dotnet/efcore/issues/831) 跟踪此功能的进度。

> [!WARNING]
> 如果您从数据库中再次对模型进行反向工程，则对这些文件所做的任何更改都将丢失。
