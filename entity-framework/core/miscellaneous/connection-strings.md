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
# <a name="connection-strings"></a><span data-ttu-id="8bca1-103">连接字符串</span><span class="sxs-lookup"><span data-stu-id="8bca1-103">Connection Strings</span></span>

<span data-ttu-id="8bca1-104">大多数数据库提供程序都需要某种形式的连接字符串才能连接到数据库。</span><span class="sxs-lookup"><span data-stu-id="8bca1-104">Most database providers require some form of connection string to connect to the database.</span></span> <span data-ttu-id="8bca1-105">有时，此连接字符串包含需要保护的敏感信息。</span><span class="sxs-lookup"><span data-stu-id="8bca1-105">Sometimes this connection string contains sensitive information that needs to be protected.</span></span> <span data-ttu-id="8bca1-106">在开发、测试和生产等环境之间移动应用程序时，可能还需要更改连接字符串。</span><span class="sxs-lookup"><span data-stu-id="8bca1-106">You may also need to change the connection string as you move your application between environments, such as development, testing, and production.</span></span>

## <a name="aspnet-core"></a><span data-ttu-id="8bca1-107">ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="8bca1-107">ASP.NET Core</span></span>

<span data-ttu-id="8bca1-108">在 ASP.NET Core 配置系统非常灵活，并且可以将连接字符串存储在 `appsettings.json` 、环境变量、用户密钥存储或其他配置源中。</span><span class="sxs-lookup"><span data-stu-id="8bca1-108">In ASP.NET Core the configuration system is very flexible, and the connection string could be stored in `appsettings.json`, an environment variable, the user secret store, or another configuration source.</span></span> <span data-ttu-id="8bca1-109">有关更多详细信息，请参阅 [ASP.NET Core 文档](/aspnet/core/fundamentals/configuration) 的 "配置" 部分。</span><span class="sxs-lookup"><span data-stu-id="8bca1-109">See the [Configuration section of the ASP.NET Core documentation](/aspnet/core/fundamentals/configuration) for more details.</span></span>

<span data-ttu-id="8bca1-110">例如，你可以使用 [机密管理器工具](/aspnet/core/security/app-secrets#secret-manager) 存储数据库密码，然后在基架中，使用只包含的连接字符串 `Name=<database-alias>` 。</span><span class="sxs-lookup"><span data-stu-id="8bca1-110">For instance, you can use the [Secret Manager tool](/aspnet/core/security/app-secrets#secret-manager) to store your database password and then, in scaffolding, use a connection string that simply consists of `Name=<database-alias>`.</span></span>

```dotnetcli
dotnet user-secrets set ConnectionStrings:YourDatabaseAlias "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=YourDatabase"
dotnet ef dbcontext scaffold Name=ConnectionStrings:YourDatabaseAlias Microsoft.EntityFrameworkCore.SqlServer
```

<span data-ttu-id="8bca1-111">下面的示例显示了中存储的连接字符串 `appsettings.json` 。</span><span class="sxs-lookup"><span data-stu-id="8bca1-111">Or the following example shows the connection string stored in `appsettings.json`.</span></span>

```json
{
  "ConnectionStrings": {
    "BloggingDatabase": "Server=(localdb)\\mssqllocaldb;Database=EFGetStarted.ConsoleApp.NewDb;Trusted_Connection=True;"
  },
}
```

<span data-ttu-id="8bca1-112">然后，上下文通常在中配置为在 `Startup.cs` 从配置中读取的连接字符串中。</span><span class="sxs-lookup"><span data-stu-id="8bca1-112">Then the context is typically configured in `Startup.cs` with the connection string being read from configuration.</span></span> <span data-ttu-id="8bca1-113">请注意， `GetConnectionString()` 方法查找其键为的配置值 `ConnectionStrings:<connection string name>` 。</span><span class="sxs-lookup"><span data-stu-id="8bca1-113">Note the `GetConnectionString()` method looks for a configuration value whose key is `ConnectionStrings:<connection string name>`.</span></span> <span data-ttu-id="8bca1-114">需要导入 [Microsoft.Extensions.Configu](/dotnet/api/microsoft.extensions.configuration) 命名空间才能使用此扩展方法。</span><span class="sxs-lookup"><span data-stu-id="8bca1-114">You need to import the [Microsoft.Extensions.Configuration](/dotnet/api/microsoft.extensions.configuration) namespace to use this extension method.</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<BloggingContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("BloggingDatabase")));
}
```

## <a name="winforms--wpf-applications"></a><span data-ttu-id="8bca1-115">WinForms & WPF 应用程序</span><span class="sxs-lookup"><span data-stu-id="8bca1-115">WinForms & WPF Applications</span></span>

<span data-ttu-id="8bca1-116">WinForms、WPF 和 ASP.NET 4 应用程序都有一个已尝试并经过测试的连接字符串模式。</span><span class="sxs-lookup"><span data-stu-id="8bca1-116">WinForms, WPF, and ASP.NET 4 applications have a tried and tested connection string pattern.</span></span> <span data-ttu-id="8bca1-117">如果使用 ASP.NET) ，则应将连接字符串添加到应用程序的 App.config 文件中 ( # A1。</span><span class="sxs-lookup"><span data-stu-id="8bca1-117">The connection string should be added to your application's App.config file (Web.config if you are using ASP.NET).</span></span> <span data-ttu-id="8bca1-118">如果你的连接字符串包含敏感信息（例如用户名和密码），则可以使用 [受保护的配置](/dotnet/framework/data/adonet/connection-strings-and-configuration-files#encrypting-configuration-file-sections-using-protected-configuration)来保护配置文件的内容。</span><span class="sxs-lookup"><span data-stu-id="8bca1-118">If your connection string contains sensitive information, such as username and password, you can protect the contents of the configuration file using [Protected Configuration](/dotnet/framework/data/adonet/connection-strings-and-configuration-files#encrypting-configuration-file-sections-using-protected-configuration).</span></span>

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
> <span data-ttu-id="8bca1-119">`providerName`由于数据库提供程序是通过代码配置的，因此在 App.config 中存储的 EF Core 连接字符串上不需要此设置。</span><span class="sxs-lookup"><span data-stu-id="8bca1-119">The `providerName` setting is not required on EF Core connection strings stored in App.config because the database provider is configured via code.</span></span>

<span data-ttu-id="8bca1-120">然后，你可以 `ConfigurationManager` 在上下文的方法中使用 API 来读取连接字符串 `OnConfiguring` 。</span><span class="sxs-lookup"><span data-stu-id="8bca1-120">You can then read the connection string using the `ConfigurationManager` API in your context's `OnConfiguring` method.</span></span> <span data-ttu-id="8bca1-121">你可能还需要添加对 `System.Configuration` 框架程序集的引用才能使用此 API。</span><span class="sxs-lookup"><span data-stu-id="8bca1-121">You may need to add a reference to the `System.Configuration` framework assembly to be able to use this API.</span></span>

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

## <a name="universal-windows-platform-uwp"></a><span data-ttu-id="8bca1-122">通用 Windows 平台 (UWP)</span><span class="sxs-lookup"><span data-stu-id="8bca1-122">Universal Windows Platform (UWP)</span></span>

<span data-ttu-id="8bca1-123">UWP 应用程序中的连接字符串通常是一个 SQLite 连接，只需指定本地文件名。</span><span class="sxs-lookup"><span data-stu-id="8bca1-123">Connection strings in a UWP application are typically a SQLite connection that just specifies a local filename.</span></span> <span data-ttu-id="8bca1-124">它们通常不包含敏感信息，并且在部署应用程序时无需更改。</span><span class="sxs-lookup"><span data-stu-id="8bca1-124">They typically do not contain sensitive information, and do not need to be changed as an application is deployed.</span></span> <span data-ttu-id="8bca1-125">因此，这些连接字符串通常可以保留在代码中，如下所示。</span><span class="sxs-lookup"><span data-stu-id="8bca1-125">As such, these connection strings are usually fine to be left in code, as shown below.</span></span> <span data-ttu-id="8bca1-126">如果希望将它们移出代码，则 UWP 支持设置概念，有关详细信息，请参阅 [uwp 文档的 "应用设置" 部分](/windows/uwp/app-settings/store-and-retrieve-app-data) 。</span><span class="sxs-lookup"><span data-stu-id="8bca1-126">If you wish to move them out of code then UWP supports the concept of settings, see the [App Settings section of the UWP documentation](/windows/uwp/app-settings/store-and-retrieve-app-data) for details.</span></span>

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
