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
# <a name="managing-migrations"></a><span data-ttu-id="b0c4d-103">管理迁移</span><span class="sxs-lookup"><span data-stu-id="b0c4d-103">Managing Migrations</span></span>

<span data-ttu-id="b0c4d-104">当模型发生更改时，将在正常开发过程中添加和删除迁移，并将迁移文件签入项目的源代码管理。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-104">As your model changes, migrations are added and removed as part of normal development, and the migration files are checked into your project's source control.</span></span> <span data-ttu-id="b0c4d-105">若要管理迁移，必须首先安装 [EF Core 命令行工具](xref:core/cli/index)。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-105">To manage migrations, you must first install the [EF Core command-line tools](xref:core/cli/index).</span></span>

> [!TIP]
> <span data-ttu-id="b0c4d-106">如果 `DbContext` 与启动项目位于不同程序集中，可以在[包管理器控制台工具](xref:core/cli/powershell#target-and-startup-project)或 [.NET Core CLI 工具](xref:core/cli/dotnet#target-project-and-startup-project)中显式指定目标和启动项目。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-106">If the `DbContext` is in a different assembly than the startup project, you can explicitly specify the target and startup projects in either the [Package Manager Console tools](xref:core/cli/powershell#target-and-startup-project) or the [.NET Core CLI tools](xref:core/cli/dotnet#target-project-and-startup-project).</span></span>

## <a name="add-a-migration"></a><span data-ttu-id="b0c4d-107">添加迁移</span><span class="sxs-lookup"><span data-stu-id="b0c4d-107">Add a migration</span></span>

<span data-ttu-id="b0c4d-108">更改模型后，可以为该更改添加迁移：</span><span class="sxs-lookup"><span data-stu-id="b0c4d-108">After your model has been changed, you can add a migration for that change:</span></span>

### <a name="net-core-cli"></a>[<span data-ttu-id="b0c4d-109">.NET Core CLI</span><span class="sxs-lookup"><span data-stu-id="b0c4d-109">.NET Core CLI</span></span>](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef migrations add AddBlogCreatedTimestamp
```

### <a name="visual-studio"></a>[<span data-ttu-id="b0c4d-110">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="b0c4d-110">Visual Studio</span></span>](#tab/vs)

```powershell
Add-Migration AddBlogCreatedTimestamp
```

***

<span data-ttu-id="b0c4d-111">迁移名称的用途与版本控制系统中的提交消息类似。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-111">The migration name can be used like a commit message in a version control system.</span></span> <span data-ttu-id="b0c4d-112">例如，如果更改是实体上的新属性，则可以选择名称（如 *AddBlogCreatedTimestamp* ） `CreatedTimestamp` `Blog` 。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-112">For example, you might choose a name like *AddBlogCreatedTimestamp* if the change is a new `CreatedTimestamp` property on your `Blog` entity.</span></span>

<span data-ttu-id="b0c4d-113">向 **Migrations** 目录下的项目添加以下三个文件：</span><span class="sxs-lookup"><span data-stu-id="b0c4d-113">Three files are added to your project under the **Migrations** directory:</span></span>

* <span data-ttu-id="b0c4d-114">**XXXXXXXXXXXXXX_AddCreatedTimestamp**-主迁移文件。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-114">**XXXXXXXXXXXXXX_AddCreatedTimestamp.cs**--The main migrations file.</span></span> <span data-ttu-id="b0c4d-115">包含应用迁移所需的操作（在 `Up` 中）和还原迁移所需的操作（在 `Down` 中）。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-115">Contains the operations necessary to apply the migration (in `Up`) and to revert it (in `Down`).</span></span>
* <span data-ttu-id="b0c4d-116">**XXXXXXXXXXXXXX_AddCreatedTimestamp**：迁移元数据文件。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-116">**XXXXXXXXXXXXXX_AddCreatedTimestamp.Designer.cs**--The migrations metadata file.</span></span> <span data-ttu-id="b0c4d-117">包含 EF 所用的信息。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-117">Contains information used by EF.</span></span>
* <span data-ttu-id="b0c4d-118">**MyContextModelSnapshot.cs**--当前模型的快照。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-118">**MyContextModelSnapshot.cs**--A snapshot of your current model.</span></span> <span data-ttu-id="b0c4d-119">用于确定添加下一迁移时的更改内容。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-119">Used to determine what changed when adding the next migration.</span></span>

<span data-ttu-id="b0c4d-120">文件名中的时间戳有助于保证文件按时间顺序排列，以便你查看更改情况。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-120">The timestamp in the filename helps keep them ordered chronologically so you can see the progression of changes.</span></span>

### <a name="namespaces"></a><span data-ttu-id="b0c4d-121">命名空间</span><span class="sxs-lookup"><span data-stu-id="b0c4d-121">Namespaces</span></span>

<span data-ttu-id="b0c4d-122">可以手动移动 Migrations 文件并更改其命名空间。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-122">You are free to move Migrations files and change their namespace manually.</span></span> <span data-ttu-id="b0c4d-123">新建的迁移和上个迁移同级。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-123">New migrations are created as siblings of the last migration.</span></span> <span data-ttu-id="b0c4d-124">或者，你可以在生成时指定目录，如下所示：</span><span class="sxs-lookup"><span data-stu-id="b0c4d-124">Alternatively, you can specify the directory at generation time as follows:</span></span>

#### <a name="net-core-cli"></a>[<span data-ttu-id="b0c4d-125">.NET Core CLI</span><span class="sxs-lookup"><span data-stu-id="b0c4d-125">.NET Core CLI</span></span>](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef migrations add InitialCreate --output-dir Your/Directory
```

> [!NOTE]
> <span data-ttu-id="b0c4d-126">在 EF Core 5.0 中，你还可以使用更改独立于目录的命名空间 `--namespace` 。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-126">In EF Core 5.0, you can also change the namespace independently of the directory using `--namespace`.</span></span>

#### <a name="visual-studio"></a>[<span data-ttu-id="b0c4d-127">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="b0c4d-127">Visual Studio</span></span>](#tab/vs)

```powershell
Add-Migration InitialCreate -OutputDir Your\Directory
```

> [!NOTE]
> <span data-ttu-id="b0c4d-128">在 EF Core 5.0 中，你还可以使用更改独立于目录的命名空间 `-Namespace` 。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-128">In EF Core 5.0, you can also change the namespace independently of the directory using `-Namespace`.</span></span>

***

## <a name="customize-migration-code"></a><span data-ttu-id="b0c4d-129">自定义迁移代码</span><span class="sxs-lookup"><span data-stu-id="b0c4d-129">Customize migration code</span></span>

<span data-ttu-id="b0c4d-130">尽管 EF Core 通常会创建准确的迁移，但应始终查看代码，并确保其对应于所需的更改;在某些情况下，它甚至是必需的。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-130">While EF Core generally creates accurate migrations, you should always review the code and make sure it corresponds to the desired change; in some cases, it is even necessary to do so.</span></span>

### <a name="column-renames"></a><span data-ttu-id="b0c4d-131">列重命名</span><span class="sxs-lookup"><span data-stu-id="b0c4d-131">Column renames</span></span>

<span data-ttu-id="b0c4d-132">在重命名属性时，需要自定义迁移的一个值得注意的例子就是。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-132">One notable example where customizing migrations is required is when renaming a property.</span></span> <span data-ttu-id="b0c4d-133">例如，如果将属性重命名 `Name` 为 `FullName` ，EF Core 将生成以下迁移：</span><span class="sxs-lookup"><span data-stu-id="b0c4d-133">For example, if you rename a property from `Name` to `FullName`, EF Core will generate the following migration:</span></span>

```csharp
migrationBuilder.DropColumn(
    name: "Name",
    table: "Customers");

migrationBuilder.AddColumn<string>(
    name: "FullName",
    table: "Customers",
    nullable: true);
```

<span data-ttu-id="b0c4d-134">EF Core 通常无法知道何时要删除某一列，并创建一个新的 (两个不同的更改) ，以及应重命名列。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-134">EF Core is generally unable to know when the intention is to drop a column and create a new one (two separate changes), and when a column should be renamed.</span></span> <span data-ttu-id="b0c4d-135">如果以上迁移按原样应用，则所有客户名称都将丢失。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-135">If the above migration is applied as-is, all your customer names will be lost.</span></span> <span data-ttu-id="b0c4d-136">若要重命名列，请将上面生成的迁移替换为以下内容：</span><span class="sxs-lookup"><span data-stu-id="b0c4d-136">To rename a column, replace the above generated migration with the following:</span></span>

```csharp
migrationBuilder.RenameColumn(
    name: "Name",
    table: "Customers",
    newName: "FullName");
```

> [!TIP]
> <span data-ttu-id="b0c4d-137">当某个操作可能会导致数据丢失（例如删除某列），搭建迁移基架过程将对此发出警告。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-137">The migration scaffolding process warns when an operation might result in data loss (like dropping a column).</span></span> <span data-ttu-id="b0c4d-138">如果看到此警告，务必检查迁移代码的准确性。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-138">If you see that warning, be especially sure to review the migrations code for accuracy.</span></span>

### <a name="adding-raw-sql"></a><span data-ttu-id="b0c4d-139">添加原始 SQL</span><span class="sxs-lookup"><span data-stu-id="b0c4d-139">Adding raw SQL</span></span>

<span data-ttu-id="b0c4d-140">重命名列时，可以通过内置 API 来实现，在许多情况下，这是不可能的。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-140">While renaming a column can be achieved via a built-in API, in many cases that is not possible.</span></span> <span data-ttu-id="b0c4d-141">例如，我们可能希望将现有的 `FirstName` 和属性替换为 `LastName` 一个新的 `FullName` 属性。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-141">For example, we may want to replace existing `FirstName` and `LastName` properties with a single, new `FullName` property.</span></span> <span data-ttu-id="b0c4d-142">EF Core 生成的迁移如下：</span><span class="sxs-lookup"><span data-stu-id="b0c4d-142">The migration generated by EF Core will be the following:</span></span>

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

<span data-ttu-id="b0c4d-143">如前所述，这会导致不需要的数据丢失。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-143">As before, this would cause unwanted data loss.</span></span> <span data-ttu-id="b0c4d-144">为了传输旧列中的数据，我们会重新排列迁移并引入原始 SQL 操作，如下所示：</span><span class="sxs-lookup"><span data-stu-id="b0c4d-144">To transfer the data from the old columns, we rearrange the migrations and introduce a raw SQL operation as follows:</span></span>

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

### <a name="arbitrary-changes-via-raw-sql"></a><span data-ttu-id="b0c4d-145">通过原始 SQL 进行任意更改</span><span class="sxs-lookup"><span data-stu-id="b0c4d-145">Arbitrary changes via raw SQL</span></span>

<span data-ttu-id="b0c4d-146">原始 SQL 还可用于管理 EF Core 不知道的数据库对象。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-146">Raw SQL can also be used to manage database objects that EF Core isn't aware of.</span></span> <span data-ttu-id="b0c4d-147">为此，请在不进行任何模型更改的情况下添加迁移;将生成一个空迁移，然后可以使用原始 SQL 操作填充该迁移。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-147">To do this, add a migration without making any model change; an empty migration will be generated, which you can then populate with raw SQL operations.</span></span>

<span data-ttu-id="b0c4d-148">例如，以下迁移创建一个 SQL Server 存储过程：</span><span class="sxs-lookup"><span data-stu-id="b0c4d-148">For example, the following migration creates a SQL Server stored procedure:</span></span>

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
> <span data-ttu-id="b0c4d-149">`EXEC` 当语句必须是 SQL 批处理中的第一个或唯一一个语句时，使用。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-149">`EXEC` is used when a statement must be the first or only one in a SQL batch.</span></span> <span data-ttu-id="b0c4d-150">它还可用于解决在引用的列当前不存在于表中时可能发生的幂等迁移脚本中的分析器错误。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-150">It can also be used to work around parser errors in idempotent migration scripts that can occur when referenced columns don't currently exist on a table.</span></span>

<span data-ttu-id="b0c4d-151">这可用于管理数据库的任何方面，包括：</span><span class="sxs-lookup"><span data-stu-id="b0c4d-151">This can be used to manage any aspect of your database, including:</span></span>

* <span data-ttu-id="b0c4d-152">存储过程</span><span class="sxs-lookup"><span data-stu-id="b0c4d-152">Stored procedures</span></span>
* <span data-ttu-id="b0c4d-153">全文搜索</span><span class="sxs-lookup"><span data-stu-id="b0c4d-153">Full-Text Search</span></span>
* <span data-ttu-id="b0c4d-154">函数</span><span class="sxs-lookup"><span data-stu-id="b0c4d-154">Functions</span></span>
* <span data-ttu-id="b0c4d-155">触发器</span><span class="sxs-lookup"><span data-stu-id="b0c4d-155">Triggers</span></span>
* <span data-ttu-id="b0c4d-156">视图</span><span class="sxs-lookup"><span data-stu-id="b0c4d-156">Views</span></span>

<span data-ttu-id="b0c4d-157">在大多数情况下，在应用迁移时，EF Core 会在其自己的事务中自动包装每个迁移。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-157">In most cases, EF Core will automatically wrap each migration in its own transaction when applying migrations.</span></span> <span data-ttu-id="b0c4d-158">遗憾的是，某些迁移操作无法在某些数据库的事务中执行;对于这些情况，你可以通过传递到来选择不使用 `suppressTransaction: true` 事务 `migrationBuilder.Sql` 。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-158">Unfortunately, some migrations operations cannot be performed within a transaction in some databases; for these cases, you may opt out of the transaction by passing `suppressTransaction: true` to `migrationBuilder.Sql`.</span></span>

<span data-ttu-id="b0c4d-159">如果 `DbContext` 与启动项目位于不同程序集中，可以在[包管理器控制台工具](xref:core/cli/powershell#target-and-startup-project)或 [.NET Core CLI 工具](xref:core/cli/dotnet#target-project-and-startup-project)中显式指定目标和启动项目。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-159">If the `DbContext` is in a different assembly than the startup project, you can explicitly specify the target and startup projects in either the [Package Manager Console tools](xref:core/cli/powershell#target-and-startup-project) or the [.NET Core CLI tools](xref:core/cli/dotnet#target-project-and-startup-project).</span></span>

## <a name="remove-a-migration"></a><span data-ttu-id="b0c4d-160">删除迁移</span><span class="sxs-lookup"><span data-stu-id="b0c4d-160">Remove a migration</span></span>

<span data-ttu-id="b0c4d-161">有时，你可能在添加迁移后意识到需要在应用迁移前对 EF Core 模型作出其他更改。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-161">Sometimes you add a migration and realize you need to make additional changes to your EF Core model before applying it.</span></span> <span data-ttu-id="b0c4d-162">要删除上个迁移，请使用如下命令。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-162">To remove the last migration, use this command.</span></span>

### <a name="net-core-cli"></a>[<span data-ttu-id="b0c4d-163">.NET Core CLI</span><span class="sxs-lookup"><span data-stu-id="b0c4d-163">.NET Core CLI</span></span>](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef migrations remove
```

### <a name="visual-studio"></a>[<span data-ttu-id="b0c4d-164">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="b0c4d-164">Visual Studio</span></span>](#tab/vs)

```powershell
Remove-Migration
```

***

<span data-ttu-id="b0c4d-165">删除迁移后，可对模型作出其他更改，然后再次添加迁移。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-165">After removing the migration, you can make the additional model changes and add it again.</span></span>

> [!WARNING]
> <span data-ttu-id="b0c4d-166">请注意，不要删除已应用于生产数据库的任何迁移。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-166">Take care not to remove any migrations which are already applied to production databases.</span></span> <span data-ttu-id="b0c4d-167">如果不这样做，将会阻止你恢复该操作，并且可能会破坏后续迁移所做的假设。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-167">Not doing so will prevent you from being able to revert it, and may break the assumptions made by subsequent migrations.</span></span>

## <a name="listing-migrations"></a><span data-ttu-id="b0c4d-168">列出迁移</span><span class="sxs-lookup"><span data-stu-id="b0c4d-168">Listing migrations</span></span>

<span data-ttu-id="b0c4d-169">可以按如下所示列出所有现有迁移：</span><span class="sxs-lookup"><span data-stu-id="b0c4d-169">You can list all existing migrations as follows:</span></span>

### <a name="net-core-cli"></a>[<span data-ttu-id="b0c4d-170">.NET Core CLI</span><span class="sxs-lookup"><span data-stu-id="b0c4d-170">.NET Core CLI</span></span>](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef migrations list
```

### <a name="visual-studio"></a>[<span data-ttu-id="b0c4d-171">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="b0c4d-171">Visual Studio</span></span>](#tab/vs)

> [!NOTE]
> <span data-ttu-id="b0c4d-172">此命令是在 EF Core 5.0 中引入的。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-172">This command was introduced in EF Core 5.0.</span></span>

```powershell
Get-Migration
```

***

## <a name="resetting-all-migrations"></a><span data-ttu-id="b0c4d-173">正在重置所有迁移</span><span class="sxs-lookup"><span data-stu-id="b0c4d-173">Resetting all migrations</span></span>

<span data-ttu-id="b0c4d-174">在某些极端情况下，可能需要删除所有迁移并重新开始。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-174">In some extreme cases, it may be necessary to remove all migrations and start over.</span></span> <span data-ttu-id="b0c4d-175">这可以通过删除 **迁移** 文件夹并删除数据库来轻松完成;此时，你可以创建新的初始迁移，其中将包含整个当前架构。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-175">This can be easily done by deleting your **Migrations** folder and dropping your database; at that point you can create a new initial migration, which will contain you entire current schema.</span></span>

<span data-ttu-id="b0c4d-176">还可以重置所有迁移并创建单个迁移，而不会丢失数据。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-176">It's also possible to reset all migrations and create a single one without losing your data.</span></span> <span data-ttu-id="b0c4d-177">这有时称为 "squashing"，包括一些手动工作：</span><span class="sxs-lookup"><span data-stu-id="b0c4d-177">This is sometimes called "squashing", and involves some manual work:</span></span>

* <span data-ttu-id="b0c4d-178">删除 **迁移** 文件夹</span><span class="sxs-lookup"><span data-stu-id="b0c4d-178">Delete your **Migrations** folder</span></span>
* <span data-ttu-id="b0c4d-179">创建新迁移并为其生成 SQL 脚本</span><span class="sxs-lookup"><span data-stu-id="b0c4d-179">Create a new migration and generate a SQL script for it</span></span>
* <span data-ttu-id="b0c4d-180">在数据库中，删除迁移历史记录表中的所有行</span><span class="sxs-lookup"><span data-stu-id="b0c4d-180">In your database, delete all rows from the migrations history table</span></span>
* <span data-ttu-id="b0c4d-181">将单个行插入到迁移历史记录中，记录已应用的第一个迁移，因为表已经存在。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-181">Insert a single row into the migrations history, to record that the first migration has already been applied, since your tables are already there.</span></span> <span data-ttu-id="b0c4d-182">Insert SQL 是上面生成的 SQL 脚本中的最后一个操作。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-182">The insert SQL is the last operation in the SQL script generated above.</span></span>

> [!WARNING]
> <span data-ttu-id="b0c4d-183">删除 **迁移** 文件夹时，任何 [自定义迁移代码](#customize-migration-code)都将丢失。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-183">Any [custom migration code](#customize-migration-code) will be lost when the **Migrations** folder is deleted.</span></span>  <span data-ttu-id="b0c4d-184">若要保留任何自定义，必须手动将其应用到新的初始迁移。</span><span class="sxs-lookup"><span data-stu-id="b0c4d-184">Any customizations must be applied to the new initial migration manually in order to be preserved.</span></span>
