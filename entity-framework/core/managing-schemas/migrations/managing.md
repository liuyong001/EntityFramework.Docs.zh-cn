---
title: 管理迁移-EF Core
description: 添加、删除和管理 Entity Framework Core 的数据库架构迁移
author: bricelam
ms.date: 10/27/2020
uid: core/managing-schemas/migrations/managing
ms.openlocfilehash: ef5e1b9bb10d6f1cd428db2fee327ec513c3f528
ms.sourcegitcommit: 7700840119b1639275f3b64836e7abb59103f2e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/05/2021
ms.locfileid: "95003271"
---
# <a name="managing-migrations"></a>管理迁移

当模型发生更改时，将在正常开发过程中添加和删除迁移，并将迁移文件签入项目的源代码管理。 若要管理迁移，必须首先安装 [EF Core 命令行工具](xref:core/cli/index)。

> [!TIP]
> 如果 `DbContext` 与启动项目位于不同程序集中，可以在[包管理器控制台工具](xref:core/cli/powershell#target-and-startup-project)或 [.NET Core CLI 工具](xref:core/cli/dotnet#target-project-and-startup-project)中显式指定目标和启动项目。

## <a name="add-a-migration"></a>添加迁移

更改模型后，可以为该更改添加迁移：

### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef migrations add AddBlogCreatedTimestamp
```

### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

```powershell
Add-Migration AddBlogCreatedTimestamp
```

***

迁移名称的用途与版本控制系统中的提交消息类似。 例如，如果更改是实体上的新属性，则可以选择名称（如 *AddBlogCreatedTimestamp* ） `CreatedTimestamp` `Blog` 。

向 **Migrations** 目录下的项目添加以下三个文件：

* **XXXXXXXXXXXXXX_AddCreatedTimestamp**-主迁移文件。 包含应用迁移所需的操作（在 `Up` 中）和还原迁移所需的操作（在 `Down` 中）。
* **XXXXXXXXXXXXXX_AddCreatedTimestamp**：迁移元数据文件。 包含 EF 所用的信息。
* **MyContextModelSnapshot.cs**--当前模型的快照。 用于确定添加下一迁移时的更改内容。

文件名中的时间戳有助于保证文件按时间顺序排列，以便你查看更改情况。

### <a name="namespaces"></a>命名空间

可以手动移动 Migrations 文件并更改其命名空间。 新建的迁移和上个迁移同级。 或者，你可以在生成时指定目录，如下所示：

#### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef migrations add InitialCreate --output-dir Your/Directory
```

> [!NOTE]
> 在 EF Core 5.0 中，你还可以使用更改独立于目录的命名空间 `--namespace` 。

#### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

```powershell
Add-Migration InitialCreate -OutputDir Your\Directory
```

> [!NOTE]
> 在 EF Core 5.0 中，你还可以使用更改独立于目录的命名空间 `-Namespace` 。

***

## <a name="customize-migration-code"></a>自定义迁移代码

尽管 EF Core 通常会创建准确的迁移，但应始终查看代码，并确保其对应于所需的更改;在某些情况下，它甚至是必需的。

### <a name="column-renames"></a>列重命名

在重命名属性时，需要自定义迁移的一个值得注意的例子就是。 例如，如果将属性重命名 `Name` 为 `FullName` ，EF Core 将生成以下迁移：

```csharp
migrationBuilder.DropColumn(
    name: "Name",
    table: "Customers");

migrationBuilder.AddColumn<string>(
    name: "FullName",
    table: "Customers",
    nullable: true);
```

EF Core 通常无法知道何时要删除某一列，并创建一个新的 (两个不同的更改) ，以及应重命名列。 如果以上迁移按原样应用，则所有客户名称都将丢失。 若要重命名列，请将上面生成的迁移替换为以下内容：

```csharp
migrationBuilder.RenameColumn(
    name: "Name",
    table: "Customers",
    newName: "FullName");
```

> [!TIP]
> 当某个操作可能会导致数据丢失（例如删除某列），搭建迁移基架过程将对此发出警告。 如果看到此警告，务必检查迁移代码的准确性。

### <a name="adding-raw-sql"></a>添加原始 SQL

重命名列时，可以通过内置 API 来实现，在许多情况下，这是不可能的。 例如，我们可能希望将现有的 `FirstName` 和属性替换为 `LastName` 一个新的 `FullName` 属性。 EF Core 生成的迁移如下：

```csharp
migrationBuilder.DropColumn(
    name: "FirstName",
    table: "Customer");

migrationBuilder.DropColumn(
    name: "LastName",
    table: "Customer");

migrationBuilder.AddColumn<string>(
    name: "FullName",
    table: "Customer",
    nullable: true);
```

如前所述，这会导致不需要的数据丢失。 为了传输旧列中的数据，我们会重新排列迁移并引入原始 SQL 操作，如下所示：

```csharp
migrationBuilder.AddColumn<string>(
    name: "FullName",
    table: "Customer",
    nullable: true);

migrationBuilder.Sql(
@"
    UPDATE Customer
    SET FullName = FirstName + ' ' + LastName;
");

migrationBuilder.DropColumn(
    name: "FirstName",
    table: "Customer");

migrationBuilder.DropColumn(
    name: "LastName",
    table: "Customer");
```

### <a name="arbitrary-changes-via-raw-sql"></a>通过原始 SQL 进行任意更改

原始 SQL 还可用于管理 EF Core 不知道的数据库对象。 为此，请在不进行任何模型更改的情况下添加迁移;将生成一个空迁移，然后可以使用原始 SQL 操作填充该迁移。

例如，以下迁移创建一个 SQL Server 存储过程：

```csharp
migrationBuilder.Sql(
@"
    EXEC ('CREATE PROCEDURE getFullName
        @LastName nvarchar(50),
        @FirstName nvarchar(50)
    AS
        RETURN @LastName + @FirstName;')");
```

> [!TIP]
> `EXEC` 当语句必须是 SQL 批处理中的第一个或唯一一个语句时，使用。 它还可用于解决在引用的列当前不存在于表中时可能发生的幂等迁移脚本中的分析器错误。

这可用于管理数据库的任何方面，包括：

* 存储过程
* 全文搜索
* 函数
* 触发器
* 视图

在大多数情况下，在应用迁移时，EF Core 会在其自己的事务中自动包装每个迁移。 遗憾的是，某些迁移操作无法在某些数据库的事务中执行;对于这些情况，你可以通过传递到来选择不使用 `suppressTransaction: true` 事务 `migrationBuilder.Sql` 。

如果 `DbContext` 与启动项目位于不同程序集中，可以在[包管理器控制台工具](xref:core/cli/powershell#target-and-startup-project)或 [.NET Core CLI 工具](xref:core/cli/dotnet#target-project-and-startup-project)中显式指定目标和启动项目。

## <a name="remove-a-migration"></a>删除迁移

有时，你可能在添加迁移后意识到需要在应用迁移前对 EF Core 模型作出其他更改。 要删除上个迁移，请使用如下命令。

### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef migrations remove
```

### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

```powershell
Remove-Migration
```

***

删除迁移后，可对模型作出其他更改，然后再次添加迁移。

> [!WARNING]
> 请注意，不要删除已应用于生产数据库的任何迁移。 如果不这样做，将会阻止你恢复该操作，并且可能会破坏后续迁移所做的假设。

## <a name="listing-migrations"></a>列出迁移

可以按如下所示列出所有现有迁移：

### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef migrations list
```

### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

> [!NOTE]
> 此命令是在 EF Core 5.0 中引入的。

```powershell
Get-Migration
```

***

## <a name="resetting-all-migrations"></a>正在重置所有迁移

在某些极端情况下，可能需要删除所有迁移并重新开始。 这可以通过删除 **迁移** 文件夹并删除数据库来轻松完成;此时，你可以创建新的初始迁移，其中将包含整个当前架构。

还可以重置所有迁移并创建单个迁移，而不会丢失数据。 这有时称为 "squashing"，包括一些手动工作：

* 删除 **迁移** 文件夹
* 创建新迁移并为其生成 SQL 脚本
* 在数据库中，删除迁移历史记录表中的所有行
* 将单个行插入到迁移历史记录中，记录已应用的第一个迁移，因为表已经存在。 Insert SQL 是上面生成的 SQL 脚本中的最后一个操作。

> [!WARNING]
> 删除 **迁移** 文件夹时，任何 [自定义迁移代码](#customize-migration-code)都将丢失。  若要保留任何自定义，必须手动将其应用到新的初始迁移。
