---
title: 迁移概述 - EF Core
description: 使用迁移通过 Entity Framework Core 管理数据库架构的概述
author: bricelam
ms.date: 10/28/2020
uid: core/managing-schemas/migrations/index
ms.openlocfilehash: b9547298714af59453aeae6d05742a03c067708b
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/19/2020
ms.locfileid: "95003388"
---
# <a name="migrations-overview"></a>迁移概述

在实际项目中，数据模型随着功能的实现而变化：添加和删除新的实体或属性，并且需要相应地更改数据库架构，使其与应用程序保持同步。 EF Core 中的迁移功能能够以递增方式更新数据库架构，使其与应用程序的数据模型保持同步，同时保留数据库中的现有数据。

简要地说，迁移的方式如下：

* 当引入数据模型更改时，开发人员使用 EF Core 工具添加相应的迁移，以描述使数据库架构保持同步所需的更新。EF Core 将当前模型与旧模型的快照进行比较，以确定差异，并生成迁移源文件；文件可在项目的源代码管理中进行跟踪，如任何其他源文件。
* 生成新的迁移后，可通过多种方式将其应用于数据库。 EF Core 在一个特殊的历史记录表中记录所有应用的迁移，使其知道哪些迁移已应用，哪些迁移尚未应用。

本页的其余部分是使用迁移的分步式入门指南。 有关更深入的信息，请参阅本节中的其他页面。

## <a name="getting-started"></a>入门

假设你刚刚完成了第一个 EF Core 应用程序，其中包含以下简单模型：

```csharp
public class Blog
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

在开发过程中，你可能已使用[创建和删除 API ](xref:core/managing-schemas/ensure-created)以快速迭代，并根据需要更改模型；但现在，你的应用程序将进入生产环境，你需要一种方法来安全地演化架构，而无需删除整个数据库。

### <a name="install-the-tools"></a>安装工具

首先，必须安装 [EF Core 命令行工具](xref:core/cli/index)：

* 我们通常建议使用 [.NET Core CLI 工具](xref:core/cli/dotnet)，该工具适用于所有平台。
* 如果你更喜欢在 Visual Studio 内工作或你具有 EF6 迁移经验，还可以使用[程序包管理器控制台工具](xref:core/cli/powershell)。

### <a name="create-your-first-migration"></a>创建第一个迁移

你现在已准备好添加第一个迁移！ 指示 EF Core 创建名为 InitialCreate 的迁移：

#### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef migrations add InitialCreate
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

```powershell
Add-Migration InitialCreate
```

**_

EF Core 将在项目中创建一个名为“Migrations”_*的目录，并生成一些文件。 最好检查 EF Core 生成的内容，并在可能的情况下修改它，但现在我们跳过此操作。

### <a name="create-your-database-and-schema"></a>创建数据库和架构

此时，可以让 EF 创建数据库并从迁移中创建架构。 可以通过以下操作实现这一点：

#### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef database update
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

```powershell
Update-Database
```

**_

就是这么回事 - 你的应用程序已准备好在新数据库上运行，你无需编写任何 SQL 代码。 请注意，这种应用迁移的方法非常适合本地开发，但不太适用于生产环境 - 有关详细信息，请参阅[应用迁移页面](xref:core/managing-schemas/migrations/applying)。

### <a name="evolving-your-model"></a>发展模型

几天后，系统会要求你将创建时间戳添加到博客。 你已完成针对应用程序的必要更改，模型现在如下所示：

```csharp
public class Blog
{
    public int Id { get; set; }
    public string Name { get; set; }
    public DateTime CreatedTimestamp { get; set; }
}
```

模型和生产数据库现在不同步，我们必须向数据库架构中添加一个新列。 让我们为此创建新迁移：

#### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef migrations add AddBlogCreatedTimestamp
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

```powershell
Add-Migration AddBlogCreatedTimestamp
```

_*_

请注意，我们为迁移提供了一个描述性名称，以便以后更容易了解项目历史记录。

由于这不是项目的第一次迁移，EF Core 现在会在添加列之前将更新的模型与旧模型的快照进行比较；模型快照是 EF Core 在你添加迁移时生成的文件之一，并签入到源代码管理中。 基于该比较，EF Core 检测到已添加一列，并添加适当的迁移。

现在，你可以像以前一样应用迁移：

<!--markdownlint-disable MD024-->

#### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef database update
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

```powershell
Update-Database
```

<!--markdownlint-enable MD024-->

_*_

请注意，这次 EF 检测到数据库已存在。 此外，在之前第一次应用迁移时，此事实记录在数据库中的特殊迁移历史记录表中；这允许 EF 自动仅应用新的迁移。

### <a name="excluding-parts-of-your-model"></a>排除模型的各个部分

> [!NOTE]
> EF Core 5.0 中已引入此功能。

有时，你可能希望引用其他 DbContext 中的类型。 这可能会导致迁移冲突。 若要防止出现这种情况，请从 DbContext 之一的迁移中排除该类型。

[!code-csharp[](../../../../samples/core/Modeling/FluentAPI/TableExcludeFromMigrations.cs#TableExcludeFromMigrations)]

### <a name="next-steps"></a>后续步骤

以上只是迁移的简短简介。 请参阅其他文档页面，详细了解如何[管理迁移](xref:core/managing-schemas/migrations/managing)、[应用迁移](xref:core/managing-schemas/migrations/applying)等。 [.NET Core CLI 工具参考](xref:core/cli/index)还包含有关不同命令的有用信息

## <a name="additional-resources"></a>其他资源

_ [EF Core Community Standup 会话](https://www.youtube.com/watch?v=mSsGERmrhnE&list=PLdo4fOcmZ0oX-DBuRG4u58ZTAJgBAeQ-t&index=20)介绍 EF Core 5.0 中的新迁移功能。
