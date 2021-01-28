---
title: EF Core 工具参考 (程序包管理器控制台) -EF Core
description: Entity Framework Core Visual Studio 包管理器控制台的参考指南
author: bricelam
ms.date: 10/27/2020
uid: core/cli/powershell
ms.openlocfilehash: 51f2dbcad0606a0d5571e96d7a7951cf595e2b9e
ms.sourcegitcommit: 7700840119b1639275f3b64836e7abb59103f2e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2021
ms.locfileid: "98983503"
---
# <a name="entity-framework-core-tools-reference---package-manager-console-in-visual-studio"></a><span data-ttu-id="45e1a-103">Entity Framework Core 工具参考-Visual Studio 中的包管理器控制台</span><span class="sxs-lookup"><span data-stu-id="45e1a-103">Entity Framework Core tools reference - Package Manager Console in Visual Studio</span></span>

<span data-ttu-id="45e1a-104">程序包管理器控制台 (用于 Entity Framework Core 执行设计时开发任务的 PMC) 工具。</span><span class="sxs-lookup"><span data-stu-id="45e1a-104">The Package Manager Console (PMC) tools for Entity Framework Core perform design-time development tasks.</span></span> <span data-ttu-id="45e1a-105">例如，他们基于现有数据库创建 [迁移](/aspnet/core/data/ef-mvc/migrations)、应用迁移和生成模型的代码。</span><span class="sxs-lookup"><span data-stu-id="45e1a-105">For example, they create [migrations](/aspnet/core/data/ef-mvc/migrations), apply migrations, and generate code for a model based on an existing database.</span></span> <span data-ttu-id="45e1a-106">命令使用 [包管理器控制台](/nuget/tools/package-manager-console)在 Visual Studio 中运行。</span><span class="sxs-lookup"><span data-stu-id="45e1a-106">The commands run inside of Visual Studio using the [Package Manager Console](/nuget/tools/package-manager-console).</span></span> <span data-ttu-id="45e1a-107">这些工具同时适用于 .NET Framework 和 .NET Core 项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-107">These tools work with both .NET Framework and .NET Core projects.</span></span>

<span data-ttu-id="45e1a-108">如果未使用 Visual Studio，则建议改为使用 [EF Core 命令行工具](xref:core/cli/dotnet) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-108">If you aren't using Visual Studio, we recommend the [EF Core Command-line Tools](xref:core/cli/dotnet) instead.</span></span> <span data-ttu-id="45e1a-109">.NET Core CLI 工具是跨平台的，在命令提示符下运行。</span><span class="sxs-lookup"><span data-stu-id="45e1a-109">The .NET Core CLI tools are cross-platform and run inside a command prompt.</span></span>

## <a name="installing-the-tools"></a><span data-ttu-id="45e1a-110">安装工具</span><span class="sxs-lookup"><span data-stu-id="45e1a-110">Installing the tools</span></span>

<span data-ttu-id="45e1a-111">在 **Package Manager console** 中运行以下命令，安装包管理器控制台工具：</span><span class="sxs-lookup"><span data-stu-id="45e1a-111">Install the Package Manager Console tools by running the following command in **Package Manager Console**:</span></span>

```powershell
Install-Package Microsoft.EntityFrameworkCore.Tools
```

<span data-ttu-id="45e1a-112">在 **Package Manager Console** 中运行以下命令，更新这些工具。</span><span class="sxs-lookup"><span data-stu-id="45e1a-112">Update the tools by running the following command in **Package Manager Console**.</span></span>

```powershell
Update-Package Microsoft.EntityFrameworkCore.Tools
```

### <a name="verify-the-installation"></a><span data-ttu-id="45e1a-113">验证安装</span><span class="sxs-lookup"><span data-stu-id="45e1a-113">Verify the installation</span></span>

<span data-ttu-id="45e1a-114">通过运行以下命令验证是否已安装这些工具：</span><span class="sxs-lookup"><span data-stu-id="45e1a-114">Verify that the tools are installed by running this command:</span></span>

```powershell
Get-Help about_EntityFrameworkCore
```

<span data-ttu-id="45e1a-115">输出如下所示 (它不会告诉你使用的工具版本) ：</span><span class="sxs-lookup"><span data-stu-id="45e1a-115">The output looks like this (it doesn't tell you which version of the tools you're using):</span></span>

```output

                     _/\__
               ---==/    \\
         ___  ___   |.    \|\
        | __|| __|  |  )   \\\
        | _| | _|   \_/ |  //|\\
        |___||_|       /   \\\/\\

TOPIC
    about_EntityFrameworkCore

SHORT DESCRIPTION
    Provides information about the Entity Framework Core Package Manager Console Tools.

<A list of available commands follows, omitted here.>
```

## <a name="using-the-tools"></a><span data-ttu-id="45e1a-116">使用工具</span><span class="sxs-lookup"><span data-stu-id="45e1a-116">Using the tools</span></span>

<span data-ttu-id="45e1a-117">使用这些工具之前：</span><span class="sxs-lookup"><span data-stu-id="45e1a-117">Before using the tools:</span></span>

* <span data-ttu-id="45e1a-118">了解目标项目和启动项目之间的差异。</span><span class="sxs-lookup"><span data-stu-id="45e1a-118">Understand the difference between target and startup project.</span></span>
* <span data-ttu-id="45e1a-119">了解如何将工具与 .NET Standard 类库一起使用。</span><span class="sxs-lookup"><span data-stu-id="45e1a-119">Learn how to use the tools with .NET Standard class libraries.</span></span>
* <span data-ttu-id="45e1a-120">对于 ASP.NET Core 项目，请设置环境。</span><span class="sxs-lookup"><span data-stu-id="45e1a-120">For ASP.NET Core projects, set the environment.</span></span>

### <a name="target-and-startup-project"></a><span data-ttu-id="45e1a-121">目标和启动项目</span><span class="sxs-lookup"><span data-stu-id="45e1a-121">Target and startup project</span></span>

<span data-ttu-id="45e1a-122">命令引用 *项目* 和 *启动项目*。</span><span class="sxs-lookup"><span data-stu-id="45e1a-122">The commands refer to a *project* and a *startup project*.</span></span>

* <span data-ttu-id="45e1a-123">该 *项目* 也称为 *目标项目* ，因为它是命令在其中添加或删除文件的位置。</span><span class="sxs-lookup"><span data-stu-id="45e1a-123">The *project* is also known as the *target project* because it's where the commands add or remove files.</span></span> <span data-ttu-id="45e1a-124">默认情况下，在 "**程序包管理器控制台**" 中选择的 **默认项目** 是目标项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-124">By default, the **Default project** selected in **Package Manager Console** is the target project.</span></span> <span data-ttu-id="45e1a-125">可以使用选项指定其他项目作为目标项目 <nobr>`--project`</nobr> 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-125">You can specify a different project as target project by using the <nobr>`--project`</nobr> option.</span></span>

* <span data-ttu-id="45e1a-126">*启动项目* 是工具生成和运行的项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-126">The *startup project* is the one that the tools build and run.</span></span> <span data-ttu-id="45e1a-127">这些工具必须在设计时执行应用程序代码，以获取有关项目的信息，例如数据库连接字符串和模型的配置。</span><span class="sxs-lookup"><span data-stu-id="45e1a-127">The tools have to execute application code at design time to get information about the project, such as the database connection string and the configuration of the model.</span></span> <span data-ttu-id="45e1a-128">默认情况下，**解决方案资源管理器** 中的 **启动项目** 是启动项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-128">By default, the **Startup Project** in **Solution Explorer** is the startup project.</span></span> <span data-ttu-id="45e1a-129">您可以使用选项指定其他项目作为启动项目 <nobr>`--startup-project`</nobr> 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-129">You can specify a different project as startup project by using the <nobr>`--startup-project`</nobr> option.</span></span>

<span data-ttu-id="45e1a-130">启动项目和目标项目通常是同一个项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-130">The startup project and target project are often the same project.</span></span> <span data-ttu-id="45e1a-131">它们是不同的项目的典型方案是：</span><span class="sxs-lookup"><span data-stu-id="45e1a-131">A typical scenario where they are separate projects is when:</span></span>

* <span data-ttu-id="45e1a-132">EF Core 的上下文和实体类位于 .NET Core 类库中。</span><span class="sxs-lookup"><span data-stu-id="45e1a-132">The EF Core context and entity classes are in a .NET Core class library.</span></span>
* <span data-ttu-id="45e1a-133">.NET Core 控制台应用程序或 web 应用程序引用类库。</span><span class="sxs-lookup"><span data-stu-id="45e1a-133">A .NET Core console app or web app references the class library.</span></span>

<span data-ttu-id="45e1a-134">还可以 [将迁移代码放在与 EF Core 上下文分离](xref:core/managing-schemas/migrations/projects)的类库中。</span><span class="sxs-lookup"><span data-stu-id="45e1a-134">It's also possible to [put migrations code in a class library separate from the EF Core context](xref:core/managing-schemas/migrations/projects).</span></span>

### <a name="other-target-frameworks"></a><span data-ttu-id="45e1a-135">其他目标框架</span><span class="sxs-lookup"><span data-stu-id="45e1a-135">Other target frameworks</span></span>

<span data-ttu-id="45e1a-136">包管理器控制台工具适用于 .NET Core 或 .NET Framework 项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-136">The Package Manager Console tools work with .NET Core or .NET Framework projects.</span></span> <span data-ttu-id="45e1a-137">.NET Standard 类库中具有 EF Core 模型的应用可能没有 .NET Core 或 .NET Framework 项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-137">Apps that have the EF Core model in a .NET Standard class library might not have a .NET Core or .NET Framework project.</span></span> <span data-ttu-id="45e1a-138">例如，这适用于 Xamarin 和通用 Windows 平台应用。</span><span class="sxs-lookup"><span data-stu-id="45e1a-138">For example, this is true of Xamarin and Universal Windows Platform apps.</span></span> <span data-ttu-id="45e1a-139">在这种情况下，你可以创建一个 .NET Core 或 .NET Framework 的控制台应用程序项目，该项目的唯一用途是充当工具的启动项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-139">In such cases, you can create a .NET Core or .NET Framework console app project whose only purpose is to act as startup project for the tools.</span></span> <span data-ttu-id="45e1a-140">项目可以是不包含实际代码的虚拟项目， &mdash; 只需为工具提供目标。</span><span class="sxs-lookup"><span data-stu-id="45e1a-140">The project can be a dummy project with no real code &mdash; it is only needed to provide a target for the tooling.</span></span>

<span data-ttu-id="45e1a-141">为什么需要虚拟项目？</span><span class="sxs-lookup"><span data-stu-id="45e1a-141">Why is a dummy project required?</span></span> <span data-ttu-id="45e1a-142">如前文所述，这些工具必须在设计时执行应用程序代码。</span><span class="sxs-lookup"><span data-stu-id="45e1a-142">As mentioned earlier, the tools have to execute application code at design time.</span></span> <span data-ttu-id="45e1a-143">为此，需要使用 .NET Core 或 .NET Framework 运行时。</span><span class="sxs-lookup"><span data-stu-id="45e1a-143">To do that, they need to use the .NET Core or .NET Framework runtime.</span></span> <span data-ttu-id="45e1a-144">当 EF Core 模型位于面向 .NET Core 或 .NET Framework 的项目中时，EF Core 工具会借用项目中的运行时。</span><span class="sxs-lookup"><span data-stu-id="45e1a-144">When the EF Core model is in a project that targets .NET Core or .NET Framework, the EF Core tools borrow the runtime from the project.</span></span> <span data-ttu-id="45e1a-145">如果 EF Core 模型在 .NET Standard 类库中，则无法执行此操作。</span><span class="sxs-lookup"><span data-stu-id="45e1a-145">They can't do that if the EF Core model is in a .NET Standard class library.</span></span> <span data-ttu-id="45e1a-146">.NET Standard 不是实际的 .NET 实现;它是 .NET 实现所必须支持的一组 Api 的规范。</span><span class="sxs-lookup"><span data-stu-id="45e1a-146">The .NET Standard is not an actual .NET implementation; it's a specification of a set of APIs that .NET implementations must support.</span></span> <span data-ttu-id="45e1a-147">因此 .NET Standard 不足以执行应用程序代码 EF Core 工具。</span><span class="sxs-lookup"><span data-stu-id="45e1a-147">Therefore .NET Standard is not sufficient for the EF Core tools to execute application code.</span></span> <span data-ttu-id="45e1a-148">你创建的用于启动项目的虚拟项目提供了一个具体的目标平台，工具可在其中加载 .NET Standard 类库。</span><span class="sxs-lookup"><span data-stu-id="45e1a-148">The dummy project you create to use as startup project provides a concrete target platform into which the tools can load the .NET Standard class library.</span></span>

### <a name="aspnet-core-environment"></a><span data-ttu-id="45e1a-149">ASP.NET Core 环境</span><span class="sxs-lookup"><span data-stu-id="45e1a-149">ASP.NET Core environment</span></span>

<span data-ttu-id="45e1a-150">若要为 ASP.NET Core 项目指定 [环境](/aspnet/core/fundamentals/environments) ，请在运行命令之前设置 **env： ASPNETCORE_ENVIRONMENT** 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-150">To specify [the environment](/aspnet/core/fundamentals/environments) for ASP.NET Core projects, set **env:ASPNETCORE_ENVIRONMENT** before running commands.</span></span>

<span data-ttu-id="45e1a-151">从 EF Core 5.0 开始，还可以将其他参数传递到 CreateHostBuilder，以便在命令行上指定环境：</span><span class="sxs-lookup"><span data-stu-id="45e1a-151">Starting in EF Core 5.0, additional arguments can also be passed into Program.CreateHostBuilder allowing you to specify the environment on the command-line:</span></span>

```powershell
Update-Database -Args '--environment Production'
```

## <a name="common-parameters"></a><span data-ttu-id="45e1a-152">通用参数</span><span class="sxs-lookup"><span data-stu-id="45e1a-152">Common parameters</span></span>

<span data-ttu-id="45e1a-153">下表显示了所有 EF Core 命令共有的参数：</span><span class="sxs-lookup"><span data-stu-id="45e1a-153">The following table shows parameters that are common to all of the EF Core commands:</span></span>

| <span data-ttu-id="45e1a-154">参数</span><span class="sxs-lookup"><span data-stu-id="45e1a-154">Parameter</span></span>                 | <span data-ttu-id="45e1a-155">说明</span><span class="sxs-lookup"><span data-stu-id="45e1a-155">Description</span></span>                                                                                                                                                                                                          |
|:--------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <span data-ttu-id="45e1a-156">-上下文 \<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-156">-Context \<String></span></span>        | <span data-ttu-id="45e1a-157">要使用的 `DbContext` 类。</span><span class="sxs-lookup"><span data-stu-id="45e1a-157">The `DbContext` class to use.</span></span> <span data-ttu-id="45e1a-158">仅命名空间或完全限定类名。</span><span class="sxs-lookup"><span data-stu-id="45e1a-158">Class name only or fully qualified with namespaces.</span></span>  <span data-ttu-id="45e1a-159">如果省略此参数，EF Core 将查找上下文类。</span><span class="sxs-lookup"><span data-stu-id="45e1a-159">If this parameter is omitted, EF Core finds the context class.</span></span> <span data-ttu-id="45e1a-160">如果有多个上下文类，则此参数是必需的。</span><span class="sxs-lookup"><span data-stu-id="45e1a-160">If there are multiple context classes, this parameter is required.</span></span> |
| <span data-ttu-id="45e1a-161">-项目 \<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-161">-Project \<String></span></span>        | <span data-ttu-id="45e1a-162">目标项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-162">The target project.</span></span> <span data-ttu-id="45e1a-163">如果省略此参数，则 **包管理器控制台** 的 **默认项目** 将用作目标项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-163">If this parameter is omitted, the **Default project** for **Package Manager Console** is used as the target project.</span></span>                                                                             |
| <span data-ttu-id="45e1a-164"><nobr>-StartupProject</nobr>\<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-164"><nobr>-StartupProject</nobr> \<String></span></span> | <span data-ttu-id="45e1a-165">启动项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-165">The startup project.</span></span> <span data-ttu-id="45e1a-166">如果省略此参数，则使用 **解决方案属性** 中的 **启动项目** 作为目标项目。</span><span class="sxs-lookup"><span data-stu-id="45e1a-166">If this parameter is omitted, the **Startup project** in **Solution properties** is used as the target project.</span></span>                                                                                 |
| <span data-ttu-id="45e1a-167">-参数 \<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-167">-Args \<String></span></span>           | <span data-ttu-id="45e1a-168">传递给应用程序的参数。</span><span class="sxs-lookup"><span data-stu-id="45e1a-168">Arguments passed to the application.</span></span> <span data-ttu-id="45e1a-169">在 EF Core 5.0 中添加。</span><span class="sxs-lookup"><span data-stu-id="45e1a-169">Added in EF Core 5.0.</span></span>                                                                                                                                                           |
| <span data-ttu-id="45e1a-170">-Verbose</span><span class="sxs-lookup"><span data-stu-id="45e1a-170">-Verbose</span></span>                  | <span data-ttu-id="45e1a-171">显示详细输出。</span><span class="sxs-lookup"><span data-stu-id="45e1a-171">Show verbose output.</span></span>                                                                                                                                                                                                 |

<span data-ttu-id="45e1a-172">若要显示有关命令的帮助信息，请使用 PowerShell `Get-Help` 命令。</span><span class="sxs-lookup"><span data-stu-id="45e1a-172">To show help information about a command, use PowerShell's `Get-Help` command.</span></span>

> [!TIP]
> <span data-ttu-id="45e1a-173">上下文、项目和 StartupProject 参数支持选项卡扩展。</span><span class="sxs-lookup"><span data-stu-id="45e1a-173">The Context, Project, and StartupProject parameters support tab-expansion.</span></span>

## <a name="add-migration"></a><span data-ttu-id="45e1a-174">Add-Migration</span><span class="sxs-lookup"><span data-stu-id="45e1a-174">Add-Migration</span></span>

<span data-ttu-id="45e1a-175">添加新的迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-175">Adds a new migration.</span></span>

<span data-ttu-id="45e1a-176">参数：</span><span class="sxs-lookup"><span data-stu-id="45e1a-176">Parameters:</span></span>

| <span data-ttu-id="45e1a-177">参数</span><span class="sxs-lookup"><span data-stu-id="45e1a-177">Parameter</span></span>                         | <span data-ttu-id="45e1a-178">说明</span><span class="sxs-lookup"><span data-stu-id="45e1a-178">Description</span></span>                                                                                                             |
|:----------------------------------|:------------------------------------------------------------------------------------------------------------------------|
| <span data-ttu-id="45e1a-179"><nobr>-Name \<String><nobr></span><span class="sxs-lookup"><span data-stu-id="45e1a-179"><nobr>-Name \<String><nobr></span></span>       | <span data-ttu-id="45e1a-180">迁移的名称。</span><span class="sxs-lookup"><span data-stu-id="45e1a-180">The name of the migration.</span></span> <span data-ttu-id="45e1a-181">这是一个位置参数，并且是必需的。</span><span class="sxs-lookup"><span data-stu-id="45e1a-181">This is a positional parameter and is required.</span></span>                                              |
| <span data-ttu-id="45e1a-182"><nobr>-OutputDir \<String></nobr></span><span class="sxs-lookup"><span data-stu-id="45e1a-182"><nobr>-OutputDir \<String></nobr></span></span> | <span data-ttu-id="45e1a-183">用于输出文件的目录。</span><span class="sxs-lookup"><span data-stu-id="45e1a-183">The directory use to output the files.</span></span> <span data-ttu-id="45e1a-184">路径相对于目标项目目录。</span><span class="sxs-lookup"><span data-stu-id="45e1a-184">Paths are relative to the target project directory.</span></span> <span data-ttu-id="45e1a-185">默认值为 "迁移"。</span><span class="sxs-lookup"><span data-stu-id="45e1a-185">Defaults to "Migrations".</span></span> |
| <span data-ttu-id="45e1a-186"><nobr>-命名空间 \<String></nobr></span><span class="sxs-lookup"><span data-stu-id="45e1a-186"><nobr>-Namespace \<String></nobr></span></span> | <span data-ttu-id="45e1a-187">要用于生成的类的命名空间。</span><span class="sxs-lookup"><span data-stu-id="45e1a-187">The namespace to use for the generated classes.</span></span> <span data-ttu-id="45e1a-188">默认为从输出目录生成。</span><span class="sxs-lookup"><span data-stu-id="45e1a-188">Defaults to generated from the output directory.</span></span> <span data-ttu-id="45e1a-189">在 EF Core 5.0 中添加。</span><span class="sxs-lookup"><span data-stu-id="45e1a-189">Added in EF Core 5.0.</span></span>  |

<span data-ttu-id="45e1a-190">上面列出了 [常见参数](#common-parameters) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-190">The [common parameters](#common-parameters) are listed above.</span></span>

## <a name="drop-database"></a><span data-ttu-id="45e1a-191">Drop-Database</span><span class="sxs-lookup"><span data-stu-id="45e1a-191">Drop-Database</span></span>

<span data-ttu-id="45e1a-192">删除数据库。</span><span class="sxs-lookup"><span data-stu-id="45e1a-192">Drops the database.</span></span>

<span data-ttu-id="45e1a-193">参数：</span><span class="sxs-lookup"><span data-stu-id="45e1a-193">Parameters:</span></span>

| <span data-ttu-id="45e1a-194">参数</span><span class="sxs-lookup"><span data-stu-id="45e1a-194">Parameter</span></span> | <span data-ttu-id="45e1a-195">说明</span><span class="sxs-lookup"><span data-stu-id="45e1a-195">Description</span></span>                                              |
|:----------|:---------------------------------------------------------|
| <span data-ttu-id="45e1a-196">-WhatIf</span><span class="sxs-lookup"><span data-stu-id="45e1a-196">-WhatIf</span></span>   | <span data-ttu-id="45e1a-197">显示要删除的数据库，但不删除它。</span><span class="sxs-lookup"><span data-stu-id="45e1a-197">Show which database would be dropped, but don't drop it.</span></span> |

<span data-ttu-id="45e1a-198">上面列出了 [常见参数](#common-parameters) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-198">The [common parameters](#common-parameters) are listed above.</span></span>

## <a name="get-dbcontext"></a><span data-ttu-id="45e1a-199">Get-DbContext</span><span class="sxs-lookup"><span data-stu-id="45e1a-199">Get-DbContext</span></span>

<span data-ttu-id="45e1a-200">列出并获取有关可用 `DbContext` 类型的信息。</span><span class="sxs-lookup"><span data-stu-id="45e1a-200">Lists and gets information about available `DbContext` types.</span></span>

<span data-ttu-id="45e1a-201">上面列出了 [常见参数](#common-parameters) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-201">The [common parameters](#common-parameters) are listed above.</span></span>

## <a name="get-migration"></a><span data-ttu-id="45e1a-202">Get-Migration</span><span class="sxs-lookup"><span data-stu-id="45e1a-202">Get-Migration</span></span>

<span data-ttu-id="45e1a-203">列出可用迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-203">Lists available migrations.</span></span> <span data-ttu-id="45e1a-204">在 EF Core 5.0 中添加。</span><span class="sxs-lookup"><span data-stu-id="45e1a-204">Added in EF Core 5.0.</span></span>

<span data-ttu-id="45e1a-205">参数：</span><span class="sxs-lookup"><span data-stu-id="45e1a-205">Parameters:</span></span>

| <span data-ttu-id="45e1a-206">参数</span><span class="sxs-lookup"><span data-stu-id="45e1a-206">Parameter</span></span>                          | <span data-ttu-id="45e1a-207">说明</span><span class="sxs-lookup"><span data-stu-id="45e1a-207">Description</span></span>                                                                                            |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------ |
| <span data-ttu-id="45e1a-208"><nobr>-连接 \<String></nobr></span><span class="sxs-lookup"><span data-stu-id="45e1a-208"><nobr>-Connection \<String></nobr></span></span> | <span data-ttu-id="45e1a-209">用于连接到数据库的连接字符串。</span><span class="sxs-lookup"><span data-stu-id="45e1a-209">The connection string to the database.</span></span> <span data-ttu-id="45e1a-210">默认值为 AddDbContext 或 OnConfiguring 中指定的值。</span><span class="sxs-lookup"><span data-stu-id="45e1a-210">Defaults to the one specified in AddDbContext or OnConfiguring.</span></span> |
| <span data-ttu-id="45e1a-211">-NoConnect</span><span class="sxs-lookup"><span data-stu-id="45e1a-211">-NoConnect</span></span>                         | <span data-ttu-id="45e1a-212">请勿连接到数据库。</span><span class="sxs-lookup"><span data-stu-id="45e1a-212">Don't connect to the database.</span></span>                                                                         |

<span data-ttu-id="45e1a-213">上面列出了 [常见参数](#common-parameters) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-213">The [common parameters](#common-parameters) are listed above.</span></span>

## <a name="remove-migration"></a><span data-ttu-id="45e1a-214">Remove-Migration</span><span class="sxs-lookup"><span data-stu-id="45e1a-214">Remove-Migration</span></span>

<span data-ttu-id="45e1a-215">删除上一次迁移 (回滚对迁移) 所做的代码更改。</span><span class="sxs-lookup"><span data-stu-id="45e1a-215">Removes the last migration (rolls back the code changes that were done for the migration).</span></span>

<span data-ttu-id="45e1a-216">参数：</span><span class="sxs-lookup"><span data-stu-id="45e1a-216">Parameters:</span></span>

| <span data-ttu-id="45e1a-217">参数</span><span class="sxs-lookup"><span data-stu-id="45e1a-217">Parameter</span></span> | <span data-ttu-id="45e1a-218">说明</span><span class="sxs-lookup"><span data-stu-id="45e1a-218">Description</span></span>                                                                     |
|:----------|:--------------------------------------------------------------------------------|
| <span data-ttu-id="45e1a-219">-Force</span><span class="sxs-lookup"><span data-stu-id="45e1a-219">-Force</span></span>    | <span data-ttu-id="45e1a-220">还原迁移 (回滚应用于数据库的更改) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-220">Revert the migration (roll back the changes that were applied to the database).</span></span> |

<span data-ttu-id="45e1a-221">上面列出了 [常见参数](#common-parameters) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-221">The [common parameters](#common-parameters) are listed above.</span></span>

## <a name="scaffold-dbcontext"></a><span data-ttu-id="45e1a-222">Scaffold-DbContext</span><span class="sxs-lookup"><span data-stu-id="45e1a-222">Scaffold-DbContext</span></span>

<span data-ttu-id="45e1a-223">为 `DbContext` 数据库的和实体类型生成代码。</span><span class="sxs-lookup"><span data-stu-id="45e1a-223">Generates code for a `DbContext` and entity types for a database.</span></span> <span data-ttu-id="45e1a-224">为了使 `Scaffold-DbContext` 生成实体类型，数据库表必须具有主键。</span><span class="sxs-lookup"><span data-stu-id="45e1a-224">In order for `Scaffold-DbContext` to generate an entity type, the database table must have a primary key.</span></span>

<span data-ttu-id="45e1a-225">参数：</span><span class="sxs-lookup"><span data-stu-id="45e1a-225">Parameters:</span></span>

| <span data-ttu-id="45e1a-226">参数</span><span class="sxs-lookup"><span data-stu-id="45e1a-226">Parameter</span></span>                          | <span data-ttu-id="45e1a-227">说明</span><span class="sxs-lookup"><span data-stu-id="45e1a-227">Description</span></span>                                                                                                                                                                                                                                                             |
|:-----------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <span data-ttu-id="45e1a-228"><nobr>-连接 \<String></nobr></span><span class="sxs-lookup"><span data-stu-id="45e1a-228"><nobr>-Connection \<String></nobr></span></span> | <span data-ttu-id="45e1a-229">用于连接到数据库的连接字符串。</span><span class="sxs-lookup"><span data-stu-id="45e1a-229">The connection string to the database.</span></span> <span data-ttu-id="45e1a-230">对于 ASP.NET Core 2.x 项目，值可以是 *name = \<name of connection string>*。</span><span class="sxs-lookup"><span data-stu-id="45e1a-230">For ASP.NET Core 2.x projects, the value can be *name=\<name of connection string>*.</span></span> <span data-ttu-id="45e1a-231">在这种情况下，该名称来自为项目设置的配置源。</span><span class="sxs-lookup"><span data-stu-id="45e1a-231">In that case the name comes from the configuration sources that are set up for the project.</span></span> <span data-ttu-id="45e1a-232">这是一个位置参数，并且是必需的。</span><span class="sxs-lookup"><span data-stu-id="45e1a-232">This is a positional parameter and is required.</span></span> |
| <span data-ttu-id="45e1a-233"><nobr>-提供程序 \<String></nobr></span><span class="sxs-lookup"><span data-stu-id="45e1a-233"><nobr>-Provider \<String></nobr></span></span>   | <span data-ttu-id="45e1a-234">要使用的提供程序。</span><span class="sxs-lookup"><span data-stu-id="45e1a-234">The provider to use.</span></span> <span data-ttu-id="45e1a-235">通常，这是 NuGet 包的名称，例如： `Microsoft.EntityFrameworkCore.SqlServer` 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-235">Typically this is the name of the NuGet package, for example: `Microsoft.EntityFrameworkCore.SqlServer`.</span></span> <span data-ttu-id="45e1a-236">这是一个位置参数，并且是必需的。</span><span class="sxs-lookup"><span data-stu-id="45e1a-236">This is a positional parameter and is required.</span></span>                                                                                           |
| <span data-ttu-id="45e1a-237">-OutputDir \<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-237">-OutputDir \<String></span></span>               | <span data-ttu-id="45e1a-238">要在其中放置文件的目录。</span><span class="sxs-lookup"><span data-stu-id="45e1a-238">The directory to put files in.</span></span> <span data-ttu-id="45e1a-239">路径相对于项目目录。</span><span class="sxs-lookup"><span data-stu-id="45e1a-239">Paths are relative to the project directory.</span></span>                                                                                                                                                                                             |
| <span data-ttu-id="45e1a-240">-ContextDir \<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-240">-ContextDir \<String></span></span>              | <span data-ttu-id="45e1a-241">要在其中放置文件的目录 `DbContext` 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-241">The directory to put the `DbContext` file in.</span></span> <span data-ttu-id="45e1a-242">路径相对于项目目录。</span><span class="sxs-lookup"><span data-stu-id="45e1a-242">Paths are relative to the project directory.</span></span>                                                                                                                                                               |
| <span data-ttu-id="45e1a-243">-命名空间 \<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-243">-Namespace \<String></span></span>               | <span data-ttu-id="45e1a-244">要用于所有生成的类的命名空间。</span><span class="sxs-lookup"><span data-stu-id="45e1a-244">The namespace to use for all generated classes.</span></span> <span data-ttu-id="45e1a-245">默认值为从根命名空间和输出目录生成。</span><span class="sxs-lookup"><span data-stu-id="45e1a-245">Defaults to generated from the root namespace and the output directory.</span></span> <span data-ttu-id="45e1a-246">在 EF Core 5.0 中添加。</span><span class="sxs-lookup"><span data-stu-id="45e1a-246">Added in EF Core 5.0.</span></span>                                                                                                                           |
| <span data-ttu-id="45e1a-247">-ContextNamespace \<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-247">-ContextNamespace \<String></span></span>        | <span data-ttu-id="45e1a-248">要用于生成的类的命名空间 `DbContext` 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-248">The namespace to use for the generated `DbContext` class.</span></span> <span data-ttu-id="45e1a-249">注意：重写 `-Namespace` 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-249">Note: overrides `-Namespace`.</span></span> <span data-ttu-id="45e1a-250">在 EF Core 5.0 中添加。</span><span class="sxs-lookup"><span data-stu-id="45e1a-250">Added in EF Core 5.0.</span></span>                                                                                                                                                           |
| <span data-ttu-id="45e1a-251">-上下文 \<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-251">-Context \<String></span></span>                 | <span data-ttu-id="45e1a-252">`DbContext`要生成的类的名称。</span><span class="sxs-lookup"><span data-stu-id="45e1a-252">The name of the `DbContext` class to generate.</span></span>                                                                                                                                                                                                                          |
| <span data-ttu-id="45e1a-253">-架构 \<String[]></span><span class="sxs-lookup"><span data-stu-id="45e1a-253">-Schemas \<String[]></span></span>               | <span data-ttu-id="45e1a-254">要为其生成实体类型的表的架构。</span><span class="sxs-lookup"><span data-stu-id="45e1a-254">The schemas of tables to generate entity types for.</span></span> <span data-ttu-id="45e1a-255">如果省略此参数，则包括所有架构。</span><span class="sxs-lookup"><span data-stu-id="45e1a-255">If this parameter is omitted, all schemas are included.</span></span>                                                                                                                                                             |
| <span data-ttu-id="45e1a-256">-表 \<String[]></span><span class="sxs-lookup"><span data-stu-id="45e1a-256">-Tables \<String[]></span></span>                | <span data-ttu-id="45e1a-257">要为其生成实体类型的表。</span><span class="sxs-lookup"><span data-stu-id="45e1a-257">The tables to generate entity types for.</span></span> <span data-ttu-id="45e1a-258">如果省略此参数，则包括所有表。</span><span class="sxs-lookup"><span data-stu-id="45e1a-258">If this parameter is omitted, all tables are included.</span></span>                                                                                                                                                                         |
| <span data-ttu-id="45e1a-259">-DataAnnotations</span><span class="sxs-lookup"><span data-stu-id="45e1a-259">-DataAnnotations</span></span>                   | <span data-ttu-id="45e1a-260">使用属性可在可能) 的情况下配置模型 (。</span><span class="sxs-lookup"><span data-stu-id="45e1a-260">Use attributes to configure the model (where possible).</span></span> <span data-ttu-id="45e1a-261">如果省略此参数，则只使用 Fluent API。</span><span class="sxs-lookup"><span data-stu-id="45e1a-261">If this parameter is omitted, only the fluent API is used.</span></span>                                                                                                                                                      |
| <span data-ttu-id="45e1a-262">-UseDatabaseNames</span><span class="sxs-lookup"><span data-stu-id="45e1a-262">-UseDatabaseNames</span></span>                  | <span data-ttu-id="45e1a-263">使用表和列的名称与数据库中显示的名称完全相同。</span><span class="sxs-lookup"><span data-stu-id="45e1a-263">Use table and column names exactly as they appear in the database.</span></span> <span data-ttu-id="45e1a-264">如果省略此参数，则更改数据库名称以更严格地符合 c # 名称样式约定。</span><span class="sxs-lookup"><span data-stu-id="45e1a-264">If this parameter is omitted, database names are changed to more closely conform to C# name style conventions.</span></span>                                                                                       |
| <span data-ttu-id="45e1a-265">-Force</span><span class="sxs-lookup"><span data-stu-id="45e1a-265">-Force</span></span>                             | <span data-ttu-id="45e1a-266">覆盖现有文件。</span><span class="sxs-lookup"><span data-stu-id="45e1a-266">Overwrite existing files.</span></span>                                                                                                                                                                                                                                               |
| <span data-ttu-id="45e1a-267">-NoOnConfiguring</span><span class="sxs-lookup"><span data-stu-id="45e1a-267">-NoOnConfiguring</span></span>                   | <span data-ttu-id="45e1a-268">不生成 `DbContext.OnConfiguring` 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-268">Don't generate `DbContext.OnConfiguring`.</span></span> <span data-ttu-id="45e1a-269">在 EF Core 5.0 中添加。</span><span class="sxs-lookup"><span data-stu-id="45e1a-269">Added in EF Core 5.0.</span></span>                                                                                                                                                                                                         |
| <span data-ttu-id="45e1a-270">-NoPluralize</span><span class="sxs-lookup"><span data-stu-id="45e1a-270">-NoPluralize</span></span>                       | <span data-ttu-id="45e1a-271">请勿使用复数化程序。</span><span class="sxs-lookup"><span data-stu-id="45e1a-271">Don't use the pluralizer.</span></span> <span data-ttu-id="45e1a-272">在 EF Core 5.0 中添加。</span><span class="sxs-lookup"><span data-stu-id="45e1a-272">Added in EF Core 5.0.</span></span>                                                                                                                                                                                                                         |

<span data-ttu-id="45e1a-273">上面列出了 [常见参数](#common-parameters) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-273">The [common parameters](#common-parameters) are listed above.</span></span>

<span data-ttu-id="45e1a-274">示例：</span><span class="sxs-lookup"><span data-stu-id="45e1a-274">Example:</span></span>

```powershell
Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models
```

<span data-ttu-id="45e1a-275">示例：仅基架选定的表，并在具有指定名称和命名空间的单独文件夹中创建上下文：</span><span class="sxs-lookup"><span data-stu-id="45e1a-275">Example that scaffolds only selected tables and creates the context in a separate folder with a specified name and namespace:</span></span>

```powershell
Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Tables "Blog","Post" -ContextDir Context -Context BlogContext -ContextNamespace New.Namespace
```

<span data-ttu-id="45e1a-276">下面的示例使用 [机密管理器工具](/aspnet/core/security/app-secrets#secret-manager)从项目的配置中读取连接字符串。</span><span class="sxs-lookup"><span data-stu-id="45e1a-276">The following example reads the connection string from the project's configuration possibly set using the [Secret Manager tool](/aspnet/core/security/app-secrets#secret-manager).</span></span>

```powershell
Scaffold-DbContext "Name=ConnectionStrings:Blogging" Microsoft.EntityFrameworkCore.SqlServer
```

## <a name="script-dbcontext"></a><span data-ttu-id="45e1a-277">Script-DbContext</span><span class="sxs-lookup"><span data-stu-id="45e1a-277">Script-DbContext</span></span>

<span data-ttu-id="45e1a-278">从 DbContext 生成 SQL 脚本。</span><span class="sxs-lookup"><span data-stu-id="45e1a-278">Generates a SQL script from the DbContext.</span></span> <span data-ttu-id="45e1a-279">绕过任何迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-279">Bypasses any migrations.</span></span> <span data-ttu-id="45e1a-280">在 EF Core 3.0 中添加。</span><span class="sxs-lookup"><span data-stu-id="45e1a-280">Added in EF Core 3.0.</span></span>

<span data-ttu-id="45e1a-281">参数：</span><span class="sxs-lookup"><span data-stu-id="45e1a-281">Parameters:</span></span>

| <span data-ttu-id="45e1a-282">参数</span><span class="sxs-lookup"><span data-stu-id="45e1a-282">Parameter</span></span>                      | <span data-ttu-id="45e1a-283">说明</span><span class="sxs-lookup"><span data-stu-id="45e1a-283">Description</span></span>                      |
| ------------------------------ | -------------------------------- |
| <span data-ttu-id="45e1a-284"><nobr>-输出 \<String></nobr></span><span class="sxs-lookup"><span data-stu-id="45e1a-284"><nobr>-Output \<String></nobr></span></span> | <span data-ttu-id="45e1a-285">要向其写入结果的文件。</span><span class="sxs-lookup"><span data-stu-id="45e1a-285">The file to write the result to.</span></span> |

<span data-ttu-id="45e1a-286">上面列出了 [常见参数](#common-parameters) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-286">The [common parameters](#common-parameters) are listed above.</span></span>

## <a name="script-migration"></a><span data-ttu-id="45e1a-287">Script-Migration</span><span class="sxs-lookup"><span data-stu-id="45e1a-287">Script-Migration</span></span>

<span data-ttu-id="45e1a-288">生成一个 SQL 脚本，该脚本将所选迁移中的所有更改应用于另一个选定的迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-288">Generates a SQL script that applies all of the changes from one selected migration to another selected migration.</span></span>

<span data-ttu-id="45e1a-289">参数：</span><span class="sxs-lookup"><span data-stu-id="45e1a-289">Parameters:</span></span>

| <span data-ttu-id="45e1a-290">参数</span><span class="sxs-lookup"><span data-stu-id="45e1a-290">Parameter</span></span>                    | <span data-ttu-id="45e1a-291">说明</span><span class="sxs-lookup"><span data-stu-id="45e1a-291">Description</span></span>                                                                                                                                                                                                                |
|:---------------------------- |:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <span data-ttu-id="45e1a-292">*-来自*\<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-292">*-From* \<String></span></span>            | <span data-ttu-id="45e1a-293">开始迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-293">The starting migration.</span></span> <span data-ttu-id="45e1a-294">可以按名称或 ID 识别迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-294">Migrations may be identified by name or by ID.</span></span> <span data-ttu-id="45e1a-295">数字0是一个特殊情况，表示在 *第一次迁移之前*。</span><span class="sxs-lookup"><span data-stu-id="45e1a-295">The number 0 is a special case that means *before the first migration*.</span></span> <span data-ttu-id="45e1a-296">默认值为 0。</span><span class="sxs-lookup"><span data-stu-id="45e1a-296">Defaults to 0.</span></span>                                                              |
| <span data-ttu-id="45e1a-297">*-到*\<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-297">*-To* \<String></span></span>              | <span data-ttu-id="45e1a-298">结束迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-298">The ending migration.</span></span> <span data-ttu-id="45e1a-299">默认为上次迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-299">Defaults to the last migration.</span></span>                                                                                                                                                                      |
| <span data-ttu-id="45e1a-300">-幂等</span><span class="sxs-lookup"><span data-stu-id="45e1a-300">-Idempotent</span></span>                  | <span data-ttu-id="45e1a-301">生成可用于任何迁移的数据库的脚本。</span><span class="sxs-lookup"><span data-stu-id="45e1a-301">Generate a script that can be used on a database at any migration.</span></span>                                                                                                                                                         |
| <span data-ttu-id="45e1a-302"><nobr>-NoTransactions</nobr></span><span class="sxs-lookup"><span data-stu-id="45e1a-302"><nobr>-NoTransactions</nobr></span></span> | <span data-ttu-id="45e1a-303">不生成 SQL transaction 语句。</span><span class="sxs-lookup"><span data-stu-id="45e1a-303">Don't generate SQL transaction statements.</span></span> <span data-ttu-id="45e1a-304">在 EF Core 5.0 中添加。</span><span class="sxs-lookup"><span data-stu-id="45e1a-304">Added in EF Core 5.0.</span></span>                                                                                                                                                           |
| <span data-ttu-id="45e1a-305">-输出 \<String></span><span class="sxs-lookup"><span data-stu-id="45e1a-305">-Output \<String></span></span>            | <span data-ttu-id="45e1a-306">要向其写入结果的文件。</span><span class="sxs-lookup"><span data-stu-id="45e1a-306">The file to write the result to.</span></span> <span data-ttu-id="45e1a-307">如果省略此参数，则会在创建应用的运行时文件所在的同一文件夹中创建具有生成名称的文件，例如： */obj/Debug/netcoreapp2.1/ghbkztfz.sql/*。</span><span class="sxs-lookup"><span data-stu-id="45e1a-307">IF this parameter is omitted, the file is created with a generated name in the same folder as the app's runtime files are created, for example: */obj/Debug/netcoreapp2.1/ghbkztfz.sql/*.</span></span> |

<span data-ttu-id="45e1a-308">上面列出了 [常见参数](#common-parameters) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-308">The [common parameters](#common-parameters) are listed above.</span></span>

> [!TIP]
> <span data-ttu-id="45e1a-309">To、From 和 Output 参数支持选项卡扩展。</span><span class="sxs-lookup"><span data-stu-id="45e1a-309">The To, From, and Output parameters support tab-expansion.</span></span>

<span data-ttu-id="45e1a-310">下面的示例使用迁移名称，为 InitialCreate 迁移 (从数据库创建一个脚本，而无需迁移) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-310">The following example creates a script for the InitialCreate migration (from a database without any migrations), using the migration name.</span></span>

```powershell
Script-Migration 0 InitialCreate
```

<span data-ttu-id="45e1a-311">以下示例使用迁移 ID，为 InitialCreate 迁移后的所有迁移创建一个脚本。</span><span class="sxs-lookup"><span data-stu-id="45e1a-311">The following example creates a script for all migrations after the InitialCreate migration, using the migration ID.</span></span>

```powershell
Script-Migration 20180904195021_InitialCreate
```

## <a name="update-database"></a><span data-ttu-id="45e1a-312">Update-Database</span><span class="sxs-lookup"><span data-stu-id="45e1a-312">Update-Database</span></span>

<span data-ttu-id="45e1a-313">将数据库更新到上次迁移或指定迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-313">Updates the database to the last migration or to a specified migration.</span></span>

| <span data-ttu-id="45e1a-314">参数</span><span class="sxs-lookup"><span data-stu-id="45e1a-314">Parameter</span></span>                           | <span data-ttu-id="45e1a-315">说明</span><span class="sxs-lookup"><span data-stu-id="45e1a-315">Description</span></span>                                                                                                                                                                                                                                                     |
|:------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <span data-ttu-id="45e1a-316"><nobr>*-迁移*\<String></nobr></span><span class="sxs-lookup"><span data-stu-id="45e1a-316"><nobr>*-Migration* \<String></nobr></span></span> | <span data-ttu-id="45e1a-317">目标迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-317">The target migration.</span></span> <span data-ttu-id="45e1a-318">可以按名称或 ID 识别迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-318">Migrations may be identified by name or by ID.</span></span> <span data-ttu-id="45e1a-319">数字0是一种特殊情况，表示在 *第一次迁移之前* ，并导致还原所有迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-319">The number 0 is a special case that means *before the first migration* and causes all migrations to be reverted.</span></span> <span data-ttu-id="45e1a-320">如果未指定迁移，则该命令默认为上一次迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-320">If no migration is specified, the command defaults to the last migration.</span></span> |
| <span data-ttu-id="45e1a-321"><nobr>-连接 \<String></nobr></span><span class="sxs-lookup"><span data-stu-id="45e1a-321"><nobr>-Connection \<String></nobr></span></span>  | <span data-ttu-id="45e1a-322">用于连接到数据库的连接字符串。</span><span class="sxs-lookup"><span data-stu-id="45e1a-322">The connection string to the database.</span></span> <span data-ttu-id="45e1a-323">默认为或中指定的 `AddDbContext` 一个 `OnConfiguring` 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-323">Defaults to the one specified in `AddDbContext` or `OnConfiguring`.</span></span> <span data-ttu-id="45e1a-324">在 EF Core 5.0 中添加。</span><span class="sxs-lookup"><span data-stu-id="45e1a-324">Added in EF Core 5.0.</span></span>                                                                                                                                |

<span data-ttu-id="45e1a-325">上面列出了 [常见参数](#common-parameters) 。</span><span class="sxs-lookup"><span data-stu-id="45e1a-325">The [common parameters](#common-parameters) are listed above.</span></span>

> [!TIP]
> <span data-ttu-id="45e1a-326">迁移参数支持选项卡扩展。</span><span class="sxs-lookup"><span data-stu-id="45e1a-326">The Migration parameter supports tab-expansion.</span></span>

<span data-ttu-id="45e1a-327">下面的示例将还原所有迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-327">The following example reverts all migrations.</span></span>

```powershell
Update-Database 0
```

<span data-ttu-id="45e1a-328">下面的示例将数据库更新为指定的迁移。</span><span class="sxs-lookup"><span data-stu-id="45e1a-328">The following examples update the database to a specified migration.</span></span> <span data-ttu-id="45e1a-329">第一个使用迁移名称，第二个使用迁移 ID 和指定的连接：</span><span class="sxs-lookup"><span data-stu-id="45e1a-329">The first uses the migration name and the second uses the migration ID and a specified connection:</span></span>

```powershell
Update-Database InitialCreate
Update-Database 20180904195021_InitialCreate -Connection your_connection_string
```

## <a name="additional-resources"></a><span data-ttu-id="45e1a-330">其他资源</span><span class="sxs-lookup"><span data-stu-id="45e1a-330">Additional resources</span></span>

* [<span data-ttu-id="45e1a-331">迁移</span><span class="sxs-lookup"><span data-stu-id="45e1a-331">Migrations</span></span>](xref:core/managing-schemas/migrations/index)
* [<span data-ttu-id="45e1a-332">反向工程</span><span class="sxs-lookup"><span data-stu-id="45e1a-332">Reverse Engineering</span></span>](xref:core/managing-schemas/scaffolding)
