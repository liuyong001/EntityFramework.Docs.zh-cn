---
title: EF Core 6.0 中的新增功能
description: EF Core 6.0 中的新功能概述
author: ajcvickers
ms.date: 01/28/2021
uid: core/what-is-new/ef-core-6.0/whatsnew
ms.openlocfilehash: bcc2b3ce9047a2c6b5a89e99b96919914bcf42fe
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543193"
---
# <a name="whats-new-in-ef-core-60"></a><span data-ttu-id="5def1-103">EF Core 6.0 中的新增功能</span><span class="sxs-lookup"><span data-stu-id="5def1-103">What's New in EF Core 6.0</span></span>

<span data-ttu-id="5def1-104">EF Core 6.0 目前正在开发中。</span><span class="sxs-lookup"><span data-stu-id="5def1-104">EF Core 6.0 is currently in development.</span></span> <span data-ttu-id="5def1-105">这包含每个预览版中引入的令人关注的更改的简要介绍。</span><span class="sxs-lookup"><span data-stu-id="5def1-105">This contains an overview of interesting changes introduced in each preview.</span></span>

<span data-ttu-id="5def1-106">此页面不会复制 [EF Core 6.0 的计划](xref:core/what-is-new/ef-core-6.0/plan)。</span><span class="sxs-lookup"><span data-stu-id="5def1-106">This page does not duplicate the [plan for EF Core 6.0](xref:core/what-is-new/ef-core-6.0/plan).</span></span> <span data-ttu-id="5def1-107">计划介绍了 EF Core 6.0 的整体主题，其中包括我们在交付最终版本之前打算包含的所有内容。</span><span class="sxs-lookup"><span data-stu-id="5def1-107">The plan describes the overall themes for EF Core 6.0, including everything we are planning to include before shipping the final release.</span></span>

## <a name="ef-core-60-preview-1"></a><span data-ttu-id="5def1-108">EF Core 6.0 预览版 1</span><span class="sxs-lookup"><span data-stu-id="5def1-108">EF Core 6.0 Preview 1</span></span>

> [!TIP]
> <span data-ttu-id="5def1-109">通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/NewInEFCore6)，你可运行并调试如下所示的预览版 1 示例。</span><span class="sxs-lookup"><span data-stu-id="5def1-109">You can run and debug into all the preview 1 samples shown below by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/NewInEFCore6).</span></span>

### <a name="unicodeattribute"></a><span data-ttu-id="5def1-110">UnicodeAttribute</span><span class="sxs-lookup"><span data-stu-id="5def1-110">UnicodeAttribute</span></span>

<span data-ttu-id="5def1-111">GitHub 问题：[#19794](https://github.com/dotnet/efcore/issues/19794)。</span><span class="sxs-lookup"><span data-stu-id="5def1-111">GitHub Issue: [#19794](https://github.com/dotnet/efcore/issues/19794).</span></span> <span data-ttu-id="5def1-112">这一功能由 [@RaymondHuy](https://github.com/RaymondHuy) 提供。</span><span class="sxs-lookup"><span data-stu-id="5def1-112">This feature was contributed by [@RaymondHuy](https://github.com/RaymondHuy).</span></span>

<span data-ttu-id="5def1-113">从 EF Core 6.0 开始，现在可以使用映射特性将字符串特性映射到非 Unicode 列，而无需直接指定数据库类型。</span><span class="sxs-lookup"><span data-stu-id="5def1-113">Starting with EF Core 6.0, a string property can now be mapped to a non-Unicode column using a mapping attribute _without specifying the database type directly_.</span></span> <span data-ttu-id="5def1-114">例如，考虑 `Book` 实体类型，该实体类型具有[国际标准书号 (ISBN)](https://en.wikipedia.org/wiki/International_Standard_Book_Number) 的属性，格式为“ISBN 978-3-16-148410-0”：</span><span class="sxs-lookup"><span data-stu-id="5def1-114">For example, consider a `Book` entity type with a property for the [International Standard Book Number (ISBN)](https://en.wikipedia.org/wiki/International_Standard_Book_Number) in the form "ISBN 978-3-16-148410-0":</span></span>

<!--
    public class Book
    {
        public int Id { get; set; }
        public string Title { get; set; }

        [Unicode(false)]
        [MaxLength(22)]
        public string Isbn { get; set; }
    }
-->
[!code-csharp[BookEntityType](../../../../samples/core/Miscellaneous/NewInEFCore6/UnicodeAttributeSample.cs?name=BookEntityType)]

<span data-ttu-id="5def1-115">由于 ISBN 不能包含任何非 unicode 字符，因此 `Unicode` 特性将导致使用非 unicode 字符串类型。</span><span class="sxs-lookup"><span data-stu-id="5def1-115">Since ISBNs cannot contain any non-unicode characters, the `Unicode` attribute will cause a non-Unicode string type to be used.</span></span> <span data-ttu-id="5def1-116">此外，`MaxLength` 用于限制数据库列的大小。</span><span class="sxs-lookup"><span data-stu-id="5def1-116">In addition, `MaxLength` is used to limit the size of the database column.</span></span> <span data-ttu-id="5def1-117">例如，使用 SQL Server 时，这将产生 `varchar(22)` 的数据库列：</span><span class="sxs-lookup"><span data-stu-id="5def1-117">For example, when using SQL Server, this results in a database column of `varchar(22)`:</span></span>

```sql
CREATE TABLE [Book] (
    [Id] int NOT NULL IDENTITY,
    [Title] nvarchar(max) NULL,
    [Isbn] varchar(22) NULL,
    CONSTRAINT [PK_Book] PRIMARY KEY ([Id]));
```

> [!NOTE]
> <span data-ttu-id="5def1-118">默认情况下，EF Core 将字符串属性映射到 Unicode 列。</span><span class="sxs-lookup"><span data-stu-id="5def1-118">EF Core maps string properties to Unicode columns by default.</span></span> <span data-ttu-id="5def1-119">当数据库系统仅支持 Unicode 类型时，`UnicodeAttribute` 会被忽略。</span><span class="sxs-lookup"><span data-stu-id="5def1-119">`UnicodeAttribute` is ignored when the database system supports only Unicode types.</span></span>

### <a name="precisionattribute"></a><span data-ttu-id="5def1-120">PrecisionAttribute</span><span class="sxs-lookup"><span data-stu-id="5def1-120">PrecisionAttribute</span></span>

<span data-ttu-id="5def1-121">GitHub 问题：[#17914](https://github.com/dotnet/efcore/issues/17914)。</span><span class="sxs-lookup"><span data-stu-id="5def1-121">GitHub Issue: [#17914](https://github.com/dotnet/efcore/issues/17914).</span></span> <span data-ttu-id="5def1-122">这一功能由 [@RaymondHuy](https://github.com/RaymondHuy) 提供。</span><span class="sxs-lookup"><span data-stu-id="5def1-122">This feature was contributed by [@RaymondHuy](https://github.com/RaymondHuy).</span></span>

<span data-ttu-id="5def1-123">现在可以使用映射特性配置数据库列的精度和小数位数，而无需直接指定数据库类型。</span><span class="sxs-lookup"><span data-stu-id="5def1-123">The precision and scale of a database column can now be configured using mapping attributes _without specifying the database type directly_.</span></span> <span data-ttu-id="5def1-124">例如，考虑具有小数 `Price` 属性的 `Product` 实体类型：</span><span class="sxs-lookup"><span data-stu-id="5def1-124">For example, consider a `Product` entity type with a decimal `Price` property:</span></span>

<!--
    public class Product
    {
        public int Id { get; set; }

        [Precision(precision: 10, scale: 2)]
        public decimal Price { get; set; }
    }
-->
[!code-csharp[ProductEntityType](../../../../samples/core/Miscellaneous/NewInEFCore6/PrecisionAttributeSample.cs?name=ProductEntityType)]

<span data-ttu-id="5def1-125">EF Core 会将此属性映射到精度为 10 且小数位数为 2 的数据库列。</span><span class="sxs-lookup"><span data-stu-id="5def1-125">EF Core will map this property to a database column with precision 10 and scale 2.</span></span> <span data-ttu-id="5def1-126">例如，在 SQL Server 上：</span><span class="sxs-lookup"><span data-stu-id="5def1-126">For example, on SQL Server:</span></span>

```sql
CREATE TABLE [Product] (
    [Id] int NOT NULL IDENTITY,
    [Price] decimal(10,2) NOT NULL,
    CONSTRAINT [PK_Product] PRIMARY KEY ([Id]));
```

### <a name="entitytypeconfigurationattribute"></a><span data-ttu-id="5def1-127">EntityTypeConfigurationAttribute</span><span class="sxs-lookup"><span data-stu-id="5def1-127">EntityTypeConfigurationAttribute</span></span>

<span data-ttu-id="5def1-128">GitHub 问题：[#23163](https://github.com/dotnet/efcore/issues/23163)。</span><span class="sxs-lookup"><span data-stu-id="5def1-128">GitHub Issue: [#23163](https://github.com/dotnet/efcore/issues/23163).</span></span> <span data-ttu-id="5def1-129">这一功能由 [@KaloyanIT](https://github.com/KaloyanIT) 提供。</span><span class="sxs-lookup"><span data-stu-id="5def1-129">This feature was contributed by [@KaloyanIT](https://github.com/KaloyanIT).</span></span>

<span data-ttu-id="5def1-130"><xref:Microsoft.EntityFrameworkCore.IEntityTypeConfiguration%601> 实例允许将每个实体类型的 <xref:Microsoft.EntityFrameworkCore.ModelBuilder> 配置包含在其各自的配置类中。</span><span class="sxs-lookup"><span data-stu-id="5def1-130"><xref:Microsoft.EntityFrameworkCore.IEntityTypeConfiguration%601> instances allow <xref:Microsoft.EntityFrameworkCore.ModelBuilder> configuration for a each entity type to be contained in its own configuration class.</span></span> <span data-ttu-id="5def1-131">例如：</span><span class="sxs-lookup"><span data-stu-id="5def1-131">For example:</span></span>

<!--
public class BookConfiguration : IEntityTypeConfiguration<Book>
{
    public void Configure(EntityTypeBuilder<Book> builder)
    {
        builder
            .Property(e => e.Isbn)
            .IsUnicode(false)
            .HasMaxLength(22);
    }
}
-->
[!code-csharp[BookConfiguration](../../../../samples/core/Miscellaneous/NewInEFCore6/EntityTypeConfigurationAttributeSample.cs?name=BookConfiguration)]

<span data-ttu-id="5def1-132">通常，此配置类必须实例化，并从 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A?displayProperty=nameWithType> 调用。</span><span class="sxs-lookup"><span data-stu-id="5def1-132">Normally, this configuration class must be instantiated and called into from <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="5def1-133">例如：</span><span class="sxs-lookup"><span data-stu-id="5def1-133">For example:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    new BookConfiguration().Configure(modelBuilder.Entity<Book>());
}
```

<span data-ttu-id="5def1-134">从 EF Core 6.0 开始，可以在实体类型上放置 `EntityTypeConfigurationAttribute`，以便 EF Core 可以查找并使用适当的配置。</span><span class="sxs-lookup"><span data-stu-id="5def1-134">Starting with EF Core 6.0, an `EntityTypeConfigurationAttribute` can be placed on the entity type such that EF Core can find and use appropriate configuration.</span></span> <span data-ttu-id="5def1-135">例如：</span><span class="sxs-lookup"><span data-stu-id="5def1-135">For example:</span></span>

<!--
[EntityTypeConfiguration(typeof(BookConfiguration))]
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Isbn { get; set; }
}
-->
[!code-csharp[BookEntityType](../../../../samples/core/Miscellaneous/NewInEFCore6/EntityTypeConfigurationAttributeSample.cs?name=BookEntityType)]

<span data-ttu-id="5def1-136">此特性意味着，每当模型中包含 `Book` 实体类型时，EF Core 都将使用指定的 `IEntityTypeConfiguration` 实现。</span><span class="sxs-lookup"><span data-stu-id="5def1-136">This attribute means that EF Core will use the specified `IEntityTypeConfiguration` implementation whenever the `Book` entity type is included in a model.</span></span> <span data-ttu-id="5def1-137">实体类型包含在使用普通机制其中一种机制的模型中。</span><span class="sxs-lookup"><span data-stu-id="5def1-137">The entity type is included in a model using one of the normal mechanisms.</span></span> <span data-ttu-id="5def1-138">例如，通过为实体类型创建 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 属性：</span><span class="sxs-lookup"><span data-stu-id="5def1-138">For example, by creating a <xref:Microsoft.EntityFrameworkCore.DbSet%601> property for the entity type:</span></span>

<!--
public class BooksContext : DbContext
{
    public DbSet<Book> Books { get; set; }

    //...
-->
[!code-csharp[DbContext](../../../../samples/core/Miscellaneous/NewInEFCore6/EntityTypeConfigurationAttributeSample.cs?name=DbContext)]

<span data-ttu-id="5def1-139">或者将其注册到 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A>：</span><span class="sxs-lookup"><span data-stu-id="5def1-139">Or by registering it in <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A>:</span></span>

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Book>();
}
```

> [!NOTE]
> <span data-ttu-id="5def1-140">程序集中不会自动发现 `EntityTypeConfigurationAttribute` 类型。</span><span class="sxs-lookup"><span data-stu-id="5def1-140">`EntityTypeConfigurationAttribute` types will not be automatically discovered in an assembly.</span></span> <span data-ttu-id="5def1-141">实体类型必须添加到模型中，然后才能在该实体类型上发现特性。</span><span class="sxs-lookup"><span data-stu-id="5def1-141">Entity types must be added to the model before the attribute will be discovered on that entity type.</span></span>

### <a name="translate-tostring-on-sqlite"></a><span data-ttu-id="5def1-142">在 SQLite 上转换 ToString</span><span class="sxs-lookup"><span data-stu-id="5def1-142">Translate ToString on SQLite</span></span>

<span data-ttu-id="5def1-143">GitHub 问题：[#17223](https://github.com/dotnet/efcore/issues/17223)。</span><span class="sxs-lookup"><span data-stu-id="5def1-143">GitHub Issue: [#17223](https://github.com/dotnet/efcore/issues/17223).</span></span> <span data-ttu-id="5def1-144">这一功能由 [@ralmsdeveloper](https://github.com/ralmsdeveloper) 提供。</span><span class="sxs-lookup"><span data-stu-id="5def1-144">This feature was contributed by [@ralmsdeveloper](https://github.com/ralmsdeveloper).</span></span>

<span data-ttu-id="5def1-145">使用 SQLite 数据库提供程序时，对 <xref:System.Object.ToString> 的调用现已转换为 SQL。</span><span class="sxs-lookup"><span data-stu-id="5def1-145">Calls to <xref:System.Object.ToString> are now translated to SQL when using the SQLite database provider.</span></span> <span data-ttu-id="5def1-146">这对于涉及非字符串列的文本搜索十分有用。</span><span class="sxs-lookup"><span data-stu-id="5def1-146">This can be useful for text searches involving non-string columns.</span></span> <span data-ttu-id="5def1-147">例如，考虑将电话号码存储为数字值的 `User` 实体类型：</span><span class="sxs-lookup"><span data-stu-id="5def1-147">For example, consider a `User` entity type that stores phone numbers as numeric values:</span></span>

<!--
    public class User
    {
        public int Id { get; set; }
        public string Username { get; set; }
        public long PhoneNumber { get; set; }
    }
-->
[!code-csharp[UserEntityType](../../../../samples/core/Miscellaneous/NewInEFCore6/ToStringTranslationSample.cs?name=UserEntityType)]

<span data-ttu-id="5def1-148">`ToString` 可用于将数字转换为数据库中的字符串。</span><span class="sxs-lookup"><span data-stu-id="5def1-148">`ToString` can be used to convert the number to a string in the database.</span></span> <span data-ttu-id="5def1-149">然后，我们可以将此字符串与 `LIKE` 等函数一起使用，以查找与模式匹配的数字。</span><span class="sxs-lookup"><span data-stu-id="5def1-149">We can then use this string with a function such as `LIKE` to find numbers that match a pattern.</span></span> <span data-ttu-id="5def1-150">例如，要查找包含 555 的所有数字：</span><span class="sxs-lookup"><span data-stu-id="5def1-150">For example, to find all numbers containing 555:</span></span>

<!--
var users = context.Users.Where(u => EF.Functions.Like(u.PhoneNumber.ToString(), "%555%")).ToList();
-->
[!code-csharp[Query](../../../../samples/core/Miscellaneous/NewInEFCore6/ToStringTranslationSample.cs?name=Query)]

<span data-ttu-id="5def1-151">使用 SQLite 数据库时，这会转换为以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="5def1-151">This translates to the following SQL when using a SQLite database:</span></span>

```sql
SELECT COUNT(*)
FROM "Users" AS "u"
WHERE CAST("u"."PhoneNumber" AS TEXT) LIKE '%555%'
```

<span data-ttu-id="5def1-152">请注意，EF Core 5.0 中已经支持 SQL Server 的 <xref:System.Object.ToString> 转换，其他数据库提供程序也可能支持。</span><span class="sxs-lookup"><span data-stu-id="5def1-152">Note that translation of <xref:System.Object.ToString> for SQL Server is already supported in EF Core 5.0, and may also be supported by other database providers.</span></span>

### <a name="effunctionsrandom"></a><span data-ttu-id="5def1-153">EF.Functions.Random</span><span class="sxs-lookup"><span data-stu-id="5def1-153">EF.Functions.Random</span></span>

<span data-ttu-id="5def1-154">GitHub 问题：[#16141](https://github.com/dotnet/efcore/issues/16141)。</span><span class="sxs-lookup"><span data-stu-id="5def1-154">GitHub Issue: [#16141](https://github.com/dotnet/efcore/issues/16141).</span></span> <span data-ttu-id="5def1-155">这一功能由 [@RaymondHuy](https://github.com/RaymondHuy) 提供。</span><span class="sxs-lookup"><span data-stu-id="5def1-155">This feature was contributed by [@RaymondHuy](https://github.com/RaymondHuy).</span></span>

<span data-ttu-id="5def1-156">`EF.Functions.Random` 映射到可返回介于 0 和 1（不含）之间的伪随机数的数据库函数。</span><span class="sxs-lookup"><span data-stu-id="5def1-156">`EF.Functions.Random` maps to a database function returning a pseudo-random number between 0 and 1 exclusive.</span></span> <span data-ttu-id="5def1-157">已在 SQL Server、SQLite 和 Cosmos 的 EF Core 存储库中实现转换。</span><span class="sxs-lookup"><span data-stu-id="5def1-157">Translations have been implemented in the EF Core repo for SQL Server, SQLite, and Cosmos.</span></span> <span data-ttu-id="5def1-158">例如，考虑具有 `Popularity` 属性的 `User` 实体类型：</span><span class="sxs-lookup"><span data-stu-id="5def1-158">For example, consider a `User` entity type with a `Popularity` property:</span></span>

<!--
    public class User
    {
        public int Id { get; set; }
        public string Username { get; set; }
        public int Popularity { get; set; }
    }
-->
[!code-csharp[UserEntityType](../../../../samples/core/Miscellaneous/NewInEFCore6/RandomFunctionSample.cs?name=UserEntityType)]

<span data-ttu-id="5def1-159">`Popularity` 的值可为 1 到 5（含）。</span><span class="sxs-lookup"><span data-stu-id="5def1-159">`Popularity` can have values from 1 to 5 inclusive.</span></span> <span data-ttu-id="5def1-160">使用 `EF.Functions.Random`，我们可以编写一个查询，以随机选择的热门程度返回所有用户：</span><span class="sxs-lookup"><span data-stu-id="5def1-160">Using `EF.Functions.Random` we can write a query to return all users with a randomly chosen popularity:</span></span>

<!--
var users = context.Users.Where(u => u.Popularity == (int)(EF.Functions.Random() * 5.0) + 1).ToList();
-->
[!code-csharp[Query](../../../../samples/core/Miscellaneous/NewInEFCore6/RandomFunctionSample.cs?name=Query)]

<span data-ttu-id="5def1-161">使用 SQL Server 数据库时，这会转换为以下 SQL：</span><span class="sxs-lookup"><span data-stu-id="5def1-161">This translates to the following SQL when using a SQL Server database:</span></span>

```sql
SELECT [u].[Id], [u].[Popularity], [u].[Username]
FROM [Users] AS [u]
WHERE [u].[Popularity] = (CAST((RAND() * 5.0E0) AS int) + 1)
```

### <a name="support-for-sql-server-sparse-columns"></a><span data-ttu-id="5def1-162">支持 SQL Server 稀疏列</span><span class="sxs-lookup"><span data-stu-id="5def1-162">Support for SQL Server sparse columns</span></span>

<span data-ttu-id="5def1-163">GitHub 问题：[#8023](https://github.com/dotnet/efcore/issues/8023)。</span><span class="sxs-lookup"><span data-stu-id="5def1-163">GitHub Issue: [#8023](https://github.com/dotnet/efcore/issues/8023).</span></span>

<span data-ttu-id="5def1-164">SQL Server [稀疏列](/sql/relational-databases/tables/use-sparse-columns)是优化为存储 null 值的普通列。</span><span class="sxs-lookup"><span data-stu-id="5def1-164">SQL Server [sparse columns](/sql/relational-databases/tables/use-sparse-columns) are ordinary columns that are optimized to store null values.</span></span> <span data-ttu-id="5def1-165">这在使用 [TPH 继承映射](xref:core/modeling/inheritance)时非常有用，其中很少使用的子类型的属性将导致表中大多数行的列值为 null。</span><span class="sxs-lookup"><span data-stu-id="5def1-165">This can be useful when using [TPH inheritance mapping](xref:core/modeling/inheritance) where properties of a rarely used subtype will result in null column values for most rows in the table.</span></span> <span data-ttu-id="5def1-166">例如，考虑从 `ForumUser` 扩展而来的 `ForumModerator` 类：</span><span class="sxs-lookup"><span data-stu-id="5def1-166">For example, consider a `ForumModerator` class that extends from `ForumUser`:</span></span>

<!--
    public class ForumUser
    {
        public int Id { get; set; }
        public string Username { get; set; }
    }

    public class ForumModerator : ForumUser
    {
        public string ForumName { get; set; }
    }
-->
[!code-csharp[UserEntityType](../../../../samples/core/Miscellaneous/NewInEFCore6/SparseColumnsSample.cs?name=UserEntityType)]

<span data-ttu-id="5def1-167">用户数可能以数百万计，而其中只有少数人是版主。</span><span class="sxs-lookup"><span data-stu-id="5def1-167">There may be millions of users, with only a handful of these being moderators.</span></span> <span data-ttu-id="5def1-168">这意味着将 `ForumName` 映射为稀疏在此处可能会有意义。</span><span class="sxs-lookup"><span data-stu-id="5def1-168">This means mapping the `ForumName` as sparse might make sense here.</span></span> <span data-ttu-id="5def1-169">现在可以使用 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> 中的 `IsSparse` 对此进行配置。</span><span class="sxs-lookup"><span data-stu-id="5def1-169">This can now be configured using `IsSparse` in <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A>.</span></span> <span data-ttu-id="5def1-170">例如：</span><span class="sxs-lookup"><span data-stu-id="5def1-170">For example:</span></span>

<!--
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder
                .Entity<ForumModerator>()
                .Property(e => e.ForumName)
                .IsSparse();
        }
-->
[!code-csharp[OnModelCreating](../../../../samples/core/Miscellaneous/NewInEFCore6/SparseColumnsSample.cs?name=OnModelCreating)]

<span data-ttu-id="5def1-171">然后 EF Core 迁移会将该列标记为稀疏列。</span><span class="sxs-lookup"><span data-stu-id="5def1-171">EF Core migrations will then mark the column as sparse.</span></span> <span data-ttu-id="5def1-172">例如：</span><span class="sxs-lookup"><span data-stu-id="5def1-172">For example:</span></span>

```sql
CREATE TABLE [ForumUser] (
    [Id] int NOT NULL IDENTITY,
    [Username] nvarchar(max) NULL,
    [Discriminator] nvarchar(max) NOT NULL,
    [ForumName] nvarchar(max) SPARSE NULL,
    CONSTRAINT [PK_ForumUser] PRIMARY KEY ([Id]));
```

> [!NOTE]
> <span data-ttu-id="5def1-173">稀疏列具有限制。</span><span class="sxs-lookup"><span data-stu-id="5def1-173">Sparse columns have limitations.</span></span> <span data-ttu-id="5def1-174">请务必阅读 [SQL Server 稀疏列文档](/sql/relational-databases/tables/use-sparse-columns)，以确保稀疏列适用于你的场景。</span><span class="sxs-lookup"><span data-stu-id="5def1-174">Make sure to read the [SQL Server sparse columns documentation](/sql/relational-databases/tables/use-sparse-columns) to ensure that sparse columns are the right choice for your scenario.</span></span>

### <a name="in-memory-database-validate-required-properties-are-not-null"></a><span data-ttu-id="5def1-175">内存中数据库：验证必需的属性不为 null</span><span class="sxs-lookup"><span data-stu-id="5def1-175">In-memory database: validate required properties are not null</span></span>

<span data-ttu-id="5def1-176">GitHub 问题：[#10613](https://github.com/dotnet/efcore/issues/10613)。</span><span class="sxs-lookup"><span data-stu-id="5def1-176">GitHub Issue: [#10613](https://github.com/dotnet/efcore/issues/10613).</span></span> <span data-ttu-id="5def1-177">这一功能由 [@fagnercarvalho](https://github.com/fagnercarvalho) 提供。</span><span class="sxs-lookup"><span data-stu-id="5def1-177">This feature was contributed by [@fagnercarvalho](https://github.com/fagnercarvalho).</span></span>

<span data-ttu-id="5def1-178">如果尝试为标记为“必需”的属性保存 null 值，则 EF Core 内存中数据库将引发异常。</span><span class="sxs-lookup"><span data-stu-id="5def1-178">The EF Core in-memory database will now throw an exception if an attempt is made to save a null value for a property marked as required.</span></span> <span data-ttu-id="5def1-179">例如，考虑具有必需的 `Username` 属性的 `User` 类型：</span><span class="sxs-lookup"><span data-stu-id="5def1-179">For example, consider a `User` type with a required `Username` property:</span></span>

<!--
    public class User
    {
        public int Id { get; set; }

        [Required]
        public string Username { get; set; }
    }
-->
[!code-csharp[UserEntityType](../../../../samples/core/Miscellaneous/NewInEFCore6/InMemoryRequiredPropertiesSample.cs?name=UserEntityType)]

<span data-ttu-id="5def1-180">如果尝试保存一个具有空 `Username` 的实体，将导致以下异常出现：</span><span class="sxs-lookup"><span data-stu-id="5def1-180">Attempting to save an entity with a null `Username` will result in the following exception:</span></span>

> <span data-ttu-id="5def1-181">Microsoft.EntityFrameworkCore.DbUpdateException：对于键值为“{Id: 1}”的实体类型“User”的实例，缺少必需的属性“{'Username'}”。</span><span class="sxs-lookup"><span data-stu-id="5def1-181">Microsoft.EntityFrameworkCore.DbUpdateException: Required properties '{'Username'}' are missing for the instance of entity type 'User' with the key value '{Id: 1}'.</span></span>

<span data-ttu-id="5def1-182">如果需要，可以禁用此验证。</span><span class="sxs-lookup"><span data-stu-id="5def1-182">This validation can be disabled if necessary.</span></span> <span data-ttu-id="5def1-183">例如：</span><span class="sxs-lookup"><span data-stu-id="5def1-183">For example:</span></span>

<!--
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder
                .LogTo(Console.WriteLine, new[] { InMemoryEventId.ChangesSaved })
                .UseInMemoryDatabase("UserContextWithNullCheckingDisabled")
                .EnableNullabilityCheck(false);
        }
-->
[!code-csharp[OnConfiguring](../../../../samples/core/Miscellaneous/NewInEFCore6/InMemoryRequiredPropertiesSample.cs?name=OnConfiguring)]

### <a name="improved-sql-server-translation-for-isnullorwhitespace"></a><span data-ttu-id="5def1-184">改进了 IsNullOrWhitespace 的 SQL Server 转换</span><span class="sxs-lookup"><span data-stu-id="5def1-184">Improved SQL Server translation for IsNullOrWhitespace</span></span>

<span data-ttu-id="5def1-185">GitHub 问题：[#22916](https://github.com/dotnet/efcore/issues/22916)。</span><span class="sxs-lookup"><span data-stu-id="5def1-185">GitHub Issue: [#22916](https://github.com/dotnet/efcore/issues/22916).</span></span> <span data-ttu-id="5def1-186">这一功能由 [@Marusyk](https://github.com/Marusyk) 提供。</span><span class="sxs-lookup"><span data-stu-id="5def1-186">This feature was contributed by [@Marusyk](https://github.com/Marusyk).</span></span>

<span data-ttu-id="5def1-187">请考虑下列查询：</span><span class="sxs-lookup"><span data-stu-id="5def1-187">Consider the following query:</span></span>

<!--
        var users = context.Users.Where(
            e => string.IsNullOrWhiteSpace(e.FirstName)
                 || string.IsNullOrWhiteSpace(e.LastName)).ToList();
-->
[!code-csharp[Query](../../../../samples/core/Miscellaneous/NewInEFCore6/IsNullOrWhitespaceSample.cs?name=Query)]

<span data-ttu-id="5def1-188">在 EF Core 6.0 之前，此项已在 SQL Server 转换为以下内容：</span><span class="sxs-lookup"><span data-stu-id="5def1-188">Before EF Core 6.0, this was translated to the following on SQL Server:</span></span>

```sql
SELECT [u].[Id], [u].[FirstName], [u].[LastName]
FROM [Users] AS [u]
WHERE ([u].[FirstName] IS NULL OR (LTRIM(RTRIM([u].[FirstName])) = N'')) OR ([u].[LastName] IS NULL OR (LTRIM(RTRIM([u].[LastName])) = N''))
```

<span data-ttu-id="5def1-189">已针对 EF Core 6.0 将这一转换改进为：</span><span class="sxs-lookup"><span data-stu-id="5def1-189">This translation has been improved for EF Core 6.0 to:</span></span>

```sql
SELECT [u].[Id], [u].[FirstName], [u].[LastName]
FROM [Users] AS [u]
WHERE ([u].[FirstName] IS NULL OR ([u].[FirstName] = N'')) OR ([u].[LastName] IS NULL OR ([u].[LastName] = N''))
```

### <a name="database-comments-are-scaffolded-to-code-comments"></a><span data-ttu-id="5def1-190">数据库注释已搭建为代码注释</span><span class="sxs-lookup"><span data-stu-id="5def1-190">Database comments are scaffolded to code comments</span></span>

<span data-ttu-id="5def1-191">GitHub 问题：[#19113](https://github.com/dotnet/efcore/issues/19113)。</span><span class="sxs-lookup"><span data-stu-id="5def1-191">GitHub Issue: [#19113](https://github.com/dotnet/efcore/issues/19113).</span></span> <span data-ttu-id="5def1-192">这一功能由 [@ErikEJ](https://github.com/ErikEJ) 提供。</span><span class="sxs-lookup"><span data-stu-id="5def1-192">This feature was contributed by [@ErikEJ](https://github.com/ErikEJ).</span></span>

<span data-ttu-id="5def1-193">对 SQL 表和列的注释现在搭建为从现有 SQL Server 数据库[进行 EF Core 模型反向工程](xref:core/managing-schemas/scaffolding)时创建的实体类型。</span><span class="sxs-lookup"><span data-stu-id="5def1-193">Comments on SQL tables and columns are now scaffolded into the entity types created when [reverse-engineering an EF Core model](xref:core/managing-schemas/scaffolding) from an existing SQL Server database.</span></span> <span data-ttu-id="5def1-194">例如：</span><span class="sxs-lookup"><span data-stu-id="5def1-194">For example:</span></span>

```csharp
/// <summary>
/// The Blog table.
/// </summary>
public partial class Blog
{
    /// <summary>
    /// The primary key.
    /// </summary>
    [Key]
    public int Id { get; set; }
}
```

## <a name="microsoftdatasqlite-60-preview-1"></a><span data-ttu-id="5def1-195">Microsoft.Data.Sqlite 6.0 预览版 1</span><span class="sxs-lookup"><span data-stu-id="5def1-195">Microsoft.Data.Sqlite 6.0 Preview 1</span></span>

> [!TIP]
> <span data-ttu-id="5def1-196">通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/NewInEFCore6)，你可运行并调试如下所示的预览版 1 示例。</span><span class="sxs-lookup"><span data-stu-id="5def1-196">You can run and debug into all the preview 1 samples shown below by [downloading the sample code from GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/NewInEFCore6).</span></span>

### <a name="savepoints-api"></a><span data-ttu-id="5def1-197">保存点 API</span><span class="sxs-lookup"><span data-stu-id="5def1-197">Savepoints API</span></span>

<span data-ttu-id="5def1-198">GitHub 问题：[#20228](https://github.com/dotnet/efcore/issues/20228)。</span><span class="sxs-lookup"><span data-stu-id="5def1-198">GitHub Issue: [#20228](https://github.com/dotnet/efcore/issues/20228).</span></span>

<span data-ttu-id="5def1-199">我们一直在对 [ADO.NET 提供程序中保存点的常见 API](https://github.com/dotnet/runtime/issues/33397) 进行标准化。</span><span class="sxs-lookup"><span data-stu-id="5def1-199">We have been standardizing on [a common API for savepoints in ADO.NET providers](https://github.com/dotnet/runtime/issues/33397).</span></span> <span data-ttu-id="5def1-200">Microsoft.Data.Sqlite 现支持此 API，包括：</span><span class="sxs-lookup"><span data-stu-id="5def1-200">Microsoft.Data.Sqlite now supports this API, including:</span></span>

- <span data-ttu-id="5def1-201"><xref:System.Data.Common.DbTransaction.Save(System.String)>，用于在事务中创建保存点</span><span class="sxs-lookup"><span data-stu-id="5def1-201"><xref:System.Data.Common.DbTransaction.Save(System.String)> to create a savepoint in the transaction</span></span>
- <span data-ttu-id="5def1-202"><xref:System.Data.Common.DbTransaction.Rollback(System.String)>，用于回滚到以前的保存点</span><span class="sxs-lookup"><span data-stu-id="5def1-202"><xref:System.Data.Common.DbTransaction.Rollback(System.String)> to roll back to a previous savepoint</span></span>
- <span data-ttu-id="5def1-203"><xref:System.Data.Common.DbTransaction.Release(System.String)>，用于释放保存点</span><span class="sxs-lookup"><span data-stu-id="5def1-203"><xref:System.Data.Common.DbTransaction.Release(System.String)> to release a savepoint</span></span>

<span data-ttu-id="5def1-204">使用保存点允许回滚事务的一部分，而不是回滚整个事务。</span><span class="sxs-lookup"><span data-stu-id="5def1-204">Using a savepoint allows part of a transaction to be rolled back without rolling back the entire transaction.</span></span> <span data-ttu-id="5def1-205">例如，以下代码可执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="5def1-205">For example, the code below:</span></span>

- <span data-ttu-id="5def1-206">创建事务</span><span class="sxs-lookup"><span data-stu-id="5def1-206">Creates a transaction</span></span>
- <span data-ttu-id="5def1-207">将更新发送到数据库</span><span class="sxs-lookup"><span data-stu-id="5def1-207">Sends an update to the database</span></span>
- <span data-ttu-id="5def1-208">创建保存点</span><span class="sxs-lookup"><span data-stu-id="5def1-208">Creates a savepoint</span></span>
- <span data-ttu-id="5def1-209">将另一个更新发送到数据库</span><span class="sxs-lookup"><span data-stu-id="5def1-209">Sends another update to the database</span></span>
- <span data-ttu-id="5def1-210">回滚到之前创建的保存点</span><span class="sxs-lookup"><span data-stu-id="5def1-210">Rolls back to the savepoint previous created</span></span>
- <span data-ttu-id="5def1-211">提交事务</span><span class="sxs-lookup"><span data-stu-id="5def1-211">Commits the transaction</span></span>

<!--
        using var connection = new SqliteConnection("DataSource=test.db");
        connection.Open();

        using var transaction = connection.BeginTransaction();

        using (var command = connection.CreateCommand())
        {
            command.CommandText = @"UPDATE Users SET Username = 'ajcvickers' WHERE Id = 1";
            command.ExecuteNonQuery();
        }

        transaction.Save("MySavepoint");

        using (var command = connection.CreateCommand())
        {
            command.CommandText = @"UPDATE Users SET Username = 'wfvickers' WHERE Id = 2";
            command.ExecuteNonQuery();
        }

        transaction.Rollback("MySavepoint");

        transaction.Commit();
-->
[!code-csharp[PerformUpdates](../../../../samples/core/Miscellaneous/NewInEFCore6/SqliteSamples.cs?name=PerformUpdates)]

<span data-ttu-id="5def1-212">这将使第一次更新提交到数据库，而第二次更新不会提交，因为在提交事务之前回滚了保存点。</span><span class="sxs-lookup"><span data-stu-id="5def1-212">This will result in the first update being committed to the database, while the second update is not committed since the savepoint was rolled back before committing the transaction.</span></span>

### <a name="command-timeout-in-the-connection-string"></a><span data-ttu-id="5def1-213">连接字符串中的命令超时</span><span class="sxs-lookup"><span data-stu-id="5def1-213">Command timeout in the connection string</span></span>

<span data-ttu-id="5def1-214">GitHub 问题：[#22505](https://github.com/dotnet/efcore/issues/22505)。</span><span class="sxs-lookup"><span data-stu-id="5def1-214">GitHub Issue: [#22505](https://github.com/dotnet/efcore/issues/22505).</span></span> <span data-ttu-id="5def1-215">这一功能由 [@nmichels](https://github.com/nmichels) 提供。</span><span class="sxs-lookup"><span data-stu-id="5def1-215">This feature was contributed by [@nmichels](https://github.com/nmichels).</span></span>

<span data-ttu-id="5def1-216">ADO.NET 提供程序支持两种不同的超时：</span><span class="sxs-lookup"><span data-stu-id="5def1-216">ADO.NET providers support two distinct timeouts:</span></span>

- <span data-ttu-id="5def1-217">连接超时，这决定了连接到数据库时等待的最长时间。</span><span class="sxs-lookup"><span data-stu-id="5def1-217">The connection timeout, which determines the maximum time to wait when making a connection to the database.</span></span>
- <span data-ttu-id="5def1-218">命令超时，这决定了等待命令完成执行所用的最长时间。</span><span class="sxs-lookup"><span data-stu-id="5def1-218">The command timeout, which determines the maximum time to wait for a command to complete executing.</span></span>

<span data-ttu-id="5def1-219">命令超时可以使用 <xref:System.Data.Common.DbCommand.CommandTimeout?displayProperty=nameWithType> 从代码进行设置。</span><span class="sxs-lookup"><span data-stu-id="5def1-219">The command timeout can be set from code using <xref:System.Data.Common.DbCommand.CommandTimeout?displayProperty=nameWithType>.</span></span> <span data-ttu-id="5def1-220">许多提供程序现在还在连接字符串中公开此命令超时。</span><span class="sxs-lookup"><span data-stu-id="5def1-220">Many providers are now also exposing this command timeout in the connection string.</span></span> <span data-ttu-id="5def1-221">Microsoft.Data.Sqlite 使用 `Command Timeout` 连接字符串关键字跟随这一趋势。</span><span class="sxs-lookup"><span data-stu-id="5def1-221">Microsoft.Data.Sqlite is following this trend with the `Command Timeout` connection string keyword.</span></span> <span data-ttu-id="5def1-222">例如，`"Command Timeout=60;DataSource=test.db"` 将使用 60 秒作为连接创建的命令的超时默认值。</span><span class="sxs-lookup"><span data-stu-id="5def1-222">For example, `"Command Timeout=60;DataSource=test.db"` will use 60 seconds as the default timeout for commands created by the connection.</span></span>

> [!TIP]
> <span data-ttu-id="5def1-223">Sqlite 将 `Default Timeout` 视为 `Command Timeout` 的同义词，因此可以改为使用前者（如果愿意）。</span><span class="sxs-lookup"><span data-stu-id="5def1-223">Sqlite treats `Default Timeout` as a synonym for `Command Timeout` and so can be used instead if preferred.</span></span>
