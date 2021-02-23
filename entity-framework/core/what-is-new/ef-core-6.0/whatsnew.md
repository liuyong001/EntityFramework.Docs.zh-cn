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
# <a name="whats-new-in-ef-core-60"></a>EF Core 6.0 中的新增功能

EF Core 6.0 目前正在开发中。 这包含每个预览版中引入的令人关注的更改的简要介绍。

此页面不会复制 [EF Core 6.0 的计划](xref:core/what-is-new/ef-core-6.0/plan)。 计划介绍了 EF Core 6.0 的整体主题，其中包括我们在交付最终版本之前打算包含的所有内容。

## <a name="ef-core-60-preview-1"></a>EF Core 6.0 预览版 1

> [!TIP]
> 通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/NewInEFCore6)，你可运行并调试如下所示的预览版 1 示例。

### <a name="unicodeattribute"></a>UnicodeAttribute

GitHub 问题：[#19794](https://github.com/dotnet/efcore/issues/19794)。 这一功能由 [@RaymondHuy](https://github.com/RaymondHuy) 提供。

从 EF Core 6.0 开始，现在可以使用映射特性将字符串特性映射到非 Unicode 列，而无需直接指定数据库类型。 例如，考虑 `Book` 实体类型，该实体类型具有[国际标准书号 (ISBN)](https://en.wikipedia.org/wiki/International_Standard_Book_Number) 的属性，格式为“ISBN 978-3-16-148410-0”：

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

由于 ISBN 不能包含任何非 unicode 字符，因此 `Unicode` 特性将导致使用非 unicode 字符串类型。 此外，`MaxLength` 用于限制数据库列的大小。 例如，使用 SQL Server 时，这将产生 `varchar(22)` 的数据库列：

```sql
CREATE TABLE [Book] (
    [Id] int NOT NULL IDENTITY,
    [Title] nvarchar(max) NULL,
    [Isbn] varchar(22) NULL,
    CONSTRAINT [PK_Book] PRIMARY KEY ([Id]));
```

> [!NOTE]
> 默认情况下，EF Core 将字符串属性映射到 Unicode 列。 当数据库系统仅支持 Unicode 类型时，`UnicodeAttribute` 会被忽略。

### <a name="precisionattribute"></a>PrecisionAttribute

GitHub 问题：[#17914](https://github.com/dotnet/efcore/issues/17914)。 这一功能由 [@RaymondHuy](https://github.com/RaymondHuy) 提供。

现在可以使用映射特性配置数据库列的精度和小数位数，而无需直接指定数据库类型。 例如，考虑具有小数 `Price` 属性的 `Product` 实体类型：

<!--
    public class Product
    {
        public int Id { get; set; }

        [Precision(precision: 10, scale: 2)]
        public decimal Price { get; set; }
    }
-->
[!code-csharp[ProductEntityType](../../../../samples/core/Miscellaneous/NewInEFCore6/PrecisionAttributeSample.cs?name=ProductEntityType)]

EF Core 会将此属性映射到精度为 10 且小数位数为 2 的数据库列。 例如，在 SQL Server 上：

```sql
CREATE TABLE [Product] (
    [Id] int NOT NULL IDENTITY,
    [Price] decimal(10,2) NOT NULL,
    CONSTRAINT [PK_Product] PRIMARY KEY ([Id]));
```

### <a name="entitytypeconfigurationattribute"></a>EntityTypeConfigurationAttribute

GitHub 问题：[#23163](https://github.com/dotnet/efcore/issues/23163)。 这一功能由 [@KaloyanIT](https://github.com/KaloyanIT) 提供。

<xref:Microsoft.EntityFrameworkCore.IEntityTypeConfiguration%601> 实例允许将每个实体类型的 <xref:Microsoft.EntityFrameworkCore.ModelBuilder> 配置包含在其各自的配置类中。 例如：

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

通常，此配置类必须实例化，并从 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A?displayProperty=nameWithType> 调用。 例如：

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    new BookConfiguration().Configure(modelBuilder.Entity<Book>());
}
```

从 EF Core 6.0 开始，可以在实体类型上放置 `EntityTypeConfigurationAttribute`，以便 EF Core 可以查找并使用适当的配置。 例如：

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

此特性意味着，每当模型中包含 `Book` 实体类型时，EF Core 都将使用指定的 `IEntityTypeConfiguration` 实现。 实体类型包含在使用普通机制其中一种机制的模型中。 例如，通过为实体类型创建 <xref:Microsoft.EntityFrameworkCore.DbSet%601> 属性：

<!--
public class BooksContext : DbContext
{
    public DbSet<Book> Books { get; set; }

    //...
-->
[!code-csharp[DbContext](../../../../samples/core/Miscellaneous/NewInEFCore6/EntityTypeConfigurationAttributeSample.cs?name=DbContext)]

或者将其注册到 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A>：

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Book>();
}
```

> [!NOTE]
> 程序集中不会自动发现 `EntityTypeConfigurationAttribute` 类型。 实体类型必须添加到模型中，然后才能在该实体类型上发现特性。

### <a name="translate-tostring-on-sqlite"></a>在 SQLite 上转换 ToString

GitHub 问题：[#17223](https://github.com/dotnet/efcore/issues/17223)。 这一功能由 [@ralmsdeveloper](https://github.com/ralmsdeveloper) 提供。

使用 SQLite 数据库提供程序时，对 <xref:System.Object.ToString> 的调用现已转换为 SQL。 这对于涉及非字符串列的文本搜索十分有用。 例如，考虑将电话号码存储为数字值的 `User` 实体类型：

<!--
    public class User
    {
        public int Id { get; set; }
        public string Username { get; set; }
        public long PhoneNumber { get; set; }
    }
-->
[!code-csharp[UserEntityType](../../../../samples/core/Miscellaneous/NewInEFCore6/ToStringTranslationSample.cs?name=UserEntityType)]

`ToString` 可用于将数字转换为数据库中的字符串。 然后，我们可以将此字符串与 `LIKE` 等函数一起使用，以查找与模式匹配的数字。 例如，要查找包含 555 的所有数字：

<!--
var users = context.Users.Where(u => EF.Functions.Like(u.PhoneNumber.ToString(), "%555%")).ToList();
-->
[!code-csharp[Query](../../../../samples/core/Miscellaneous/NewInEFCore6/ToStringTranslationSample.cs?name=Query)]

使用 SQLite 数据库时，这会转换为以下 SQL：

```sql
SELECT COUNT(*)
FROM "Users" AS "u"
WHERE CAST("u"."PhoneNumber" AS TEXT) LIKE '%555%'
```

请注意，EF Core 5.0 中已经支持 SQL Server 的 <xref:System.Object.ToString> 转换，其他数据库提供程序也可能支持。

### <a name="effunctionsrandom"></a>EF.Functions.Random

GitHub 问题：[#16141](https://github.com/dotnet/efcore/issues/16141)。 这一功能由 [@RaymondHuy](https://github.com/RaymondHuy) 提供。

`EF.Functions.Random` 映射到可返回介于 0 和 1（不含）之间的伪随机数的数据库函数。 已在 SQL Server、SQLite 和 Cosmos 的 EF Core 存储库中实现转换。 例如，考虑具有 `Popularity` 属性的 `User` 实体类型：

<!--
    public class User
    {
        public int Id { get; set; }
        public string Username { get; set; }
        public int Popularity { get; set; }
    }
-->
[!code-csharp[UserEntityType](../../../../samples/core/Miscellaneous/NewInEFCore6/RandomFunctionSample.cs?name=UserEntityType)]

`Popularity` 的值可为 1 到 5（含）。 使用 `EF.Functions.Random`，我们可以编写一个查询，以随机选择的热门程度返回所有用户：

<!--
var users = context.Users.Where(u => u.Popularity == (int)(EF.Functions.Random() * 5.0) + 1).ToList();
-->
[!code-csharp[Query](../../../../samples/core/Miscellaneous/NewInEFCore6/RandomFunctionSample.cs?name=Query)]

使用 SQL Server 数据库时，这会转换为以下 SQL：

```sql
SELECT [u].[Id], [u].[Popularity], [u].[Username]
FROM [Users] AS [u]
WHERE [u].[Popularity] = (CAST((RAND() * 5.0E0) AS int) + 1)
```

### <a name="support-for-sql-server-sparse-columns"></a>支持 SQL Server 稀疏列

GitHub 问题：[#8023](https://github.com/dotnet/efcore/issues/8023)。

SQL Server [稀疏列](/sql/relational-databases/tables/use-sparse-columns)是优化为存储 null 值的普通列。 这在使用 [TPH 继承映射](xref:core/modeling/inheritance)时非常有用，其中很少使用的子类型的属性将导致表中大多数行的列值为 null。 例如，考虑从 `ForumUser` 扩展而来的 `ForumModerator` 类：

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

用户数可能以数百万计，而其中只有少数人是版主。 这意味着将 `ForumName` 映射为稀疏在此处可能会有意义。 现在可以使用 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> 中的 `IsSparse` 对此进行配置。 例如：

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

然后 EF Core 迁移会将该列标记为稀疏列。 例如：

```sql
CREATE TABLE [ForumUser] (
    [Id] int NOT NULL IDENTITY,
    [Username] nvarchar(max) NULL,
    [Discriminator] nvarchar(max) NOT NULL,
    [ForumName] nvarchar(max) SPARSE NULL,
    CONSTRAINT [PK_ForumUser] PRIMARY KEY ([Id]));
```

> [!NOTE]
> 稀疏列具有限制。 请务必阅读 [SQL Server 稀疏列文档](/sql/relational-databases/tables/use-sparse-columns)，以确保稀疏列适用于你的场景。

### <a name="in-memory-database-validate-required-properties-are-not-null"></a>内存中数据库：验证必需的属性不为 null

GitHub 问题：[#10613](https://github.com/dotnet/efcore/issues/10613)。 这一功能由 [@fagnercarvalho](https://github.com/fagnercarvalho) 提供。

如果尝试为标记为“必需”的属性保存 null 值，则 EF Core 内存中数据库将引发异常。 例如，考虑具有必需的 `Username` 属性的 `User` 类型：

<!--
    public class User
    {
        public int Id { get; set; }

        [Required]
        public string Username { get; set; }
    }
-->
[!code-csharp[UserEntityType](../../../../samples/core/Miscellaneous/NewInEFCore6/InMemoryRequiredPropertiesSample.cs?name=UserEntityType)]

如果尝试保存一个具有空 `Username` 的实体，将导致以下异常出现：

> Microsoft.EntityFrameworkCore.DbUpdateException：对于键值为“{Id: 1}”的实体类型“User”的实例，缺少必需的属性“{'Username'}”。

如果需要，可以禁用此验证。 例如：

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

### <a name="improved-sql-server-translation-for-isnullorwhitespace"></a>改进了 IsNullOrWhitespace 的 SQL Server 转换

GitHub 问题：[#22916](https://github.com/dotnet/efcore/issues/22916)。 这一功能由 [@Marusyk](https://github.com/Marusyk) 提供。

请考虑下列查询：

<!--
        var users = context.Users.Where(
            e => string.IsNullOrWhiteSpace(e.FirstName)
                 || string.IsNullOrWhiteSpace(e.LastName)).ToList();
-->
[!code-csharp[Query](../../../../samples/core/Miscellaneous/NewInEFCore6/IsNullOrWhitespaceSample.cs?name=Query)]

在 EF Core 6.0 之前，此项已在 SQL Server 转换为以下内容：

```sql
SELECT [u].[Id], [u].[FirstName], [u].[LastName]
FROM [Users] AS [u]
WHERE ([u].[FirstName] IS NULL OR (LTRIM(RTRIM([u].[FirstName])) = N'')) OR ([u].[LastName] IS NULL OR (LTRIM(RTRIM([u].[LastName])) = N''))
```

已针对 EF Core 6.0 将这一转换改进为：

```sql
SELECT [u].[Id], [u].[FirstName], [u].[LastName]
FROM [Users] AS [u]
WHERE ([u].[FirstName] IS NULL OR ([u].[FirstName] = N'')) OR ([u].[LastName] IS NULL OR ([u].[LastName] = N''))
```

### <a name="database-comments-are-scaffolded-to-code-comments"></a>数据库注释已搭建为代码注释

GitHub 问题：[#19113](https://github.com/dotnet/efcore/issues/19113)。 这一功能由 [@ErikEJ](https://github.com/ErikEJ) 提供。

对 SQL 表和列的注释现在搭建为从现有 SQL Server 数据库[进行 EF Core 模型反向工程](xref:core/managing-schemas/scaffolding)时创建的实体类型。 例如：

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

## <a name="microsoftdatasqlite-60-preview-1"></a>Microsoft.Data.Sqlite 6.0 预览版 1

> [!TIP]
> 通过[从 GitHub 下载示例代码](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/NewInEFCore6)，你可运行并调试如下所示的预览版 1 示例。

### <a name="savepoints-api"></a>保存点 API

GitHub 问题：[#20228](https://github.com/dotnet/efcore/issues/20228)。

我们一直在对 [ADO.NET 提供程序中保存点的常见 API](https://github.com/dotnet/runtime/issues/33397) 进行标准化。 Microsoft.Data.Sqlite 现支持此 API，包括：

- <xref:System.Data.Common.DbTransaction.Save(System.String)>，用于在事务中创建保存点
- <xref:System.Data.Common.DbTransaction.Rollback(System.String)>，用于回滚到以前的保存点
- <xref:System.Data.Common.DbTransaction.Release(System.String)>，用于释放保存点

使用保存点允许回滚事务的一部分，而不是回滚整个事务。 例如，以下代码可执行以下操作：

- 创建事务
- 将更新发送到数据库
- 创建保存点
- 将另一个更新发送到数据库
- 回滚到之前创建的保存点
- 提交事务

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

这将使第一次更新提交到数据库，而第二次更新不会提交，因为在提交事务之前回滚了保存点。

### <a name="command-timeout-in-the-connection-string"></a>连接字符串中的命令超时

GitHub 问题：[#22505](https://github.com/dotnet/efcore/issues/22505)。 这一功能由 [@nmichels](https://github.com/nmichels) 提供。

ADO.NET 提供程序支持两种不同的超时：

- 连接超时，这决定了连接到数据库时等待的最长时间。
- 命令超时，这决定了等待命令完成执行所用的最长时间。

命令超时可以使用 <xref:System.Data.Common.DbCommand.CommandTimeout?displayProperty=nameWithType> 从代码进行设置。 许多提供程序现在还在连接字符串中公开此命令超时。 Microsoft.Data.Sqlite 使用 `Command Timeout` 连接字符串关键字跟随这一趋势。 例如，`"Command Timeout=60;DataSource=test.db"` 将使用 60 秒作为连接创建的命令的超时默认值。

> [!TIP]
> Sqlite 将 `Default Timeout` 视为 `Command Timeout` 的同义词，因此可以改为使用前者（如果愿意）。
