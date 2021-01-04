---
title: Entity Framework Core 工具参考 - EF Core
description: Entity Framework Core CLI 工具和 Visual Studio 包管理器控制台的参考指南
author: bricelam
ms.date: 09/19/2018
uid: core/cli/index
ms.openlocfilehash: 1ffc773cb8ed30516d682b90bbd9accef634ae6a
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97635375"
---
# <a name="entity-framework-core-tools-reference"></a>Entity Framework Core 工具参考

Entity Framework Core 工具可帮助完成设计时的开发任务。 它们主要用于通过对数据库架构进行反向工程来管理迁移和搭建 `DbContext` 和实体类型的基架。

可以安装下列两个工具之一，因为它们公开相同的功能：

* 在 Visual Studio 中，[EF Core 程序包管理器控制台工具](xref:core/cli/powershell) 在[程序包管理器控制台](/nuget/tools/package-manager-console)中运行。 如果使用 Visual Studio 进行开发，建议使用这些工具，因为它们可以提供更一体化的体验。

* [EF Core.NET 命令行接口 (CLI) 工具](xref:core/cli/dotnet)是对跨平台 [.NET Core CLI 工具](/dotnet/core/tools/)的扩展。 这些工具需要 .NET Core SDK 项目（具有 `Sdk="Microsoft.NET.Sdk"` 的项目或项目文件中的相似项目）。

## <a name="next-steps"></a>后续步骤

* [EF Core 程序包管理器控制台工具参考](xref:core/cli/powershell)
* [EF Core.NET CLI 工具参考](xref:core/cli/dotnet)
