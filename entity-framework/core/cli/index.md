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
# <a name="entity-framework-core-tools-reference"></a><span data-ttu-id="e5ef4-103">Entity Framework Core 工具参考</span><span class="sxs-lookup"><span data-stu-id="e5ef4-103">Entity Framework Core tools reference</span></span>

<span data-ttu-id="e5ef4-104">Entity Framework Core 工具可帮助完成设计时的开发任务。</span><span class="sxs-lookup"><span data-stu-id="e5ef4-104">The Entity Framework Core tools help with design-time development tasks.</span></span> <span data-ttu-id="e5ef4-105">它们主要用于通过对数据库架构进行反向工程来管理迁移和搭建 `DbContext` 和实体类型的基架。</span><span class="sxs-lookup"><span data-stu-id="e5ef4-105">They're primarily used to manage Migrations and to scaffold a `DbContext` and entity types by reverse engineering the schema of a database.</span></span>

<span data-ttu-id="e5ef4-106">可以安装下列两个工具之一，因为它们公开相同的功能：</span><span class="sxs-lookup"><span data-stu-id="e5ef4-106">Either of the following tools can be installed, as both tools expose the same functionality:</span></span>

* <span data-ttu-id="e5ef4-107">在 Visual Studio 中，[EF Core 程序包管理器控制台工具](xref:core/cli/powershell) 在[程序包管理器控制台](/nuget/tools/package-manager-console)中运行。</span><span class="sxs-lookup"><span data-stu-id="e5ef4-107">The [EF Core Package Manager Console tools](xref:core/cli/powershell) run in the [Package Manager Console](/nuget/tools/package-manager-console) in Visual Studio.</span></span> <span data-ttu-id="e5ef4-108">如果使用 Visual Studio 进行开发，建议使用这些工具，因为它们可以提供更一体化的体验。</span><span class="sxs-lookup"><span data-stu-id="e5ef4-108">We recommend using these tools if you are developing in Visual Studio as they provide a more integrated experience.</span></span>

* <span data-ttu-id="e5ef4-109">[EF Core.NET 命令行接口 (CLI) 工具](xref:core/cli/dotnet)是对跨平台 [.NET Core CLI 工具](/dotnet/core/tools/)的扩展。</span><span class="sxs-lookup"><span data-stu-id="e5ef4-109">The [EF Core .NET command-line interface (CLI) tools](xref:core/cli/dotnet) are an extension to the cross-platform [.NET Core CLI tools](/dotnet/core/tools/).</span></span> <span data-ttu-id="e5ef4-110">这些工具需要 .NET Core SDK 项目（具有 `Sdk="Microsoft.NET.Sdk"` 的项目或项目文件中的相似项目）。</span><span class="sxs-lookup"><span data-stu-id="e5ef4-110">These tools require a .NET Core SDK project (one with `Sdk="Microsoft.NET.Sdk"` or similar in the project file).</span></span>

## <a name="next-steps"></a><span data-ttu-id="e5ef4-111">后续步骤</span><span class="sxs-lookup"><span data-stu-id="e5ef4-111">Next steps</span></span>

* [<span data-ttu-id="e5ef4-112">EF Core 程序包管理器控制台工具参考</span><span class="sxs-lookup"><span data-stu-id="e5ef4-112">EF Core Package Manager Console tools reference</span></span>](xref:core/cli/powershell)
* [<span data-ttu-id="e5ef4-113">EF Core.NET CLI 工具参考</span><span class="sxs-lookup"><span data-stu-id="e5ef4-113">EF Core .NET CLI tools reference</span></span>](xref:core/cli/dotnet)
