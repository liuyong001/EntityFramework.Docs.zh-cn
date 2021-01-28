---
title: 连接字符串-EF Core
description: 用 Entity Framework Core 管理不同环境下的连接字符串
author: bricelam
ms.date: 10/27/2016
uid: core/miscellaneous/connection-strings
ms.openlocfilehash: 75e364eddd02087cffdffd1c152d1e988a99817b
ms.sourcegitcommit: 7700840119b1639275f3b64836e7abb59103f2e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2021
ms.locfileid: "98983555"
---
# <a name="connection-strings"></a>连接字符串

大多数数据库提供程序都需要某种形式的连接字符串才能连接到数据库。 有时，此连接字符串包含需要保护的敏感信息。 在开发、测试和生产等环境之间移动应用程序时，可能还需要更改连接字符串。

## <a name="aspnet-core"></a>ASP.NET Core

在 ASP.NET Core 配置系统非常灵活，并且可以将连接字符串存储在 `appsettings.json` 、环境变量、用户密钥存储或其他配置源中。 有关更多详细信息，请参阅 [ASP.NET Core 文档](/aspnet/core/fundamentals/configuration) 的 "配置" 部分。

例如，你可以使用 [机密管理器工具](/aspnet/core/security/app-secrets#secret-manager) 存储数据库密码，然后在基架中，使用只包含的连接字符串 `Name=<database-alias>` 。

```dotnetcli
dotnet user-secrets set ConnectionStrings:YourDatabaseAlias "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=YourDatabase"
dotnet ef dbcontext scaffold Name=ConnectionStrings:YourDatabaseAlias Microsoft.EntityFrameworkCore.SqlServer
```

下面的示例显示了中存储的连接字符串 `appsettings.json` 。

```json
{
  "ConnectionStrings": {
    "BloggingDatabase": "Server=(localdb)\\mssqllocaldb;Database=EFGetStarted.ConsoleApp.NewDb;Trusted_Connection=True;"
  },
}
```

然后，上下文通常在中配置为在 `Startup.cs` 从配置中读取的连接字符串中。 请注意， `GetConnectionString()` 方法查找其键为的配置值 `ConnectionStrings:<connection string name>` 。 需要导入 [Microsoft.Extensions.Configu](/dotnet/api/microsoft.extensions.configuration) 命名空间才能使用此扩展方法。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<BloggingContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("BloggingDatabase")));
}
```

## <a name="winforms--wpf-applications"></a>WinForms & WPF 应用程序

WinForms、WPF 和 ASP.NET 4 应用程序都有一个已尝试并经过测试的连接字符串模式。 如果使用 ASP.NET) ，则应将连接字符串添加到应用程序的 App.config 文件中 ( # A1。 如果你的连接字符串包含敏感信息（例如用户名和密码），则可以使用 [受保护的配置](/dotnet/framework/data/adonet/connection-strings-and-configuration-files#encrypting-configuration-file-sections-using-protected-configuration)来保护配置文件的内容。

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>

  <connectionStrings>
    <add name="BloggingDatabase"
         connectionString="Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" />
  </connectionStrings>
</configuration>
```

> [!TIP]
> `providerName`由于数据库提供程序是通过代码配置的，因此在 App.config 中存储的 EF Core 连接字符串上不需要此设置。

然后，你可以 `ConfigurationManager` 在上下文的方法中使用 API 来读取连接字符串 `OnConfiguring` 。 你可能还需要添加对 `System.Configuration` 框架程序集的引用才能使用此 API。

```csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
      optionsBuilder.UseSqlServer(ConfigurationManager.ConnectionStrings["BloggingDatabase"].ConnectionString);
    }
}
```

## <a name="universal-windows-platform-uwp"></a>通用 Windows 平台 (UWP)

UWP 应用程序中的连接字符串通常是一个 SQLite 连接，只需指定本地文件名。 它们通常不包含敏感信息，并且在部署应用程序时无需更改。 因此，这些连接字符串通常可以保留在代码中，如下所示。 如果希望将它们移出代码，则 UWP 支持设置概念，有关详细信息，请参阅 [uwp 文档的 "应用设置" 部分](/windows/uwp/app-settings/store-and-retrieve-app-data) 。

```csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
            optionsBuilder.UseSqlite("Data Source=blogging.db");
    }
}
```
