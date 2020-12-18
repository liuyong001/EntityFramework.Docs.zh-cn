---
title: DbContext 生存期、配置和初始化 - EF Core
description: 用于创建和管理包含或不包含依赖关系注入的 DbContext 实例的模式
author: ajcvickers
ms.date: 11/07/2020
uid: core/dbcontext-configuration/index
ms.openlocfilehash: 93d5942fbc81ee0ae9aeff0c5c8b9e20b160d512
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97635387"
---
# <a name="dbcontext-lifetime-configuration-and-initialization"></a>DbContext 生存期、配置和初始化

本文介绍初始化和配置 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例的基本模式。

## <a name="the-dbcontext-lifetime"></a>DbContext 生存期

`DbContext` 的生存期从创建实例时开始，并在[释放](/dotnet/standard/garbage-collection/unmanaged)实例时结束。 `DbContext` 实例旨在用于单个[工作单元](https://www.martinfowler.com/eaaCatalog/unitOfWork.html)。 这意味着 `DbContext` 实例的生存期通常很短。

> [!TIP]
> 引用上述链接中 Martin Fowler 的话，“工作单元将持续跟踪在可能影响数据库的业务事务中执行的所有操作。 当你完成操作后，它将找出更改数据库作为工作结果时需要执行的所有操作。”

使用 Entity Framework Core (EF Core) 时的典型工作单元包括：

- 创建 `DbContext` 实例
- 根据上下文跟踪实体实例。 实体将在以下情况下被跟踪
  - 正在[从查询返回](xref:core/querying/tracking)
  - 正在[添加或附加到上下文](xref:core/saving/disconnected-entities)
- 根据需要对所跟踪的实体进行更改以实现业务规则
- 调用 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 或 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A>。 EF Core 检测所做的更改，并将这些更改写入数据库。
- 释放 `DbContext` 实例

> [!IMPORTANT]
>
> - 使用后释放 <xref:Microsoft.EntityFrameworkCore.DbContext> 非常重要。 这可确保释放所有非托管资源，并注销任何事件或其他挂钩，以防止在实例保持引用时出现内存泄漏。
> - [DbContext 不是线程安全的](#avoiding-dbcontext-threading-issues)。 不要在线程之间共享上下文。 请确保在继续使用上下文实例之前，[等待](/dotnet/csharp/language-reference/operators/await)所有异步调用。
> - EF Core 代码引发的 <xref:System.InvalidOperationException> 可以使上下文进入不可恢复的状态。 此类异常指示程序错误，并且不旨在从其中恢复。

## <a name="dbcontext-in-dependency-injection-for-aspnet-core"></a>ASP.NET Core 依赖关系注入中的 DbContext

在许多 Web 应用程序中，每个 HTTP 请求都对应于单个工作单元。 这使得上下文生存期与请求的生存期相关，成为 Web 应用程序的一个良好默认值。

[使用依赖关系注入配置](/aspnet/core/fundamentals/startup) ASP.NET Core 应用程序。 可以使用 `Startup.cs` 的 [`ConfigureServices`](/aspnet/core/fundamentals/startup#the-configureservices-method) 方法中的 <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> 将 EF Core 添加到此配置。 例如：

<!--
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();
            
            services.AddDbContext<ApplicationDbContext>(
                options => options.UseSqlServer("name=ConnectionStrings:DefaultConnection"));
        }
-->
[!code-csharp[ConfigureServices](../../../samples/core/Miscellaneous/ConfiguringDbContext/WebApp/Startup.cs?name=ConfigureServices)]

此示例将名为 `ApplicationDbContext` 的 `DbContext` 子类注册为 ASP.NET Core 应用程序服务提供程序（也称为 依赖关系注入容器）中的作用域服务。 上下文配置为使用 SQL Server 数据库提供程序，并将从 ASP.NET Core 配置读取连接字符串。 在 `ConfigureServices` 中的何处调用 `AddDbContext` 通常不重要。

`ApplicationDbContext` 类必须公开具有 `DbContextOptions<ApplicationDbContext>` 参数的公共构造函数。 这是将 `AddDbContext` 的上下文配置传递到 `DbContext` 的方式。 例如：

<!--
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }
    }
-->
[!code-csharp[ApplicationDbContext](../../../samples/core/Miscellaneous/ConfiguringDbContext/WebApp/ApplicationDbContext.cs?name=ApplicationDbContext)]

然后，`ApplicationDbContext` 可以通过构造函数注入在 ASP.NET Core 控制器或其他服务中使用。 例如：

<!--
    public class MyController
    {
        private readonly ApplicationDbContext _context;

        public MyController(ApplicationDbContext context)
        {
            _context = context;
        }
    }
-->
[!code-csharp[MyController](../../../samples/core/Miscellaneous/ConfiguringDbContext/WebApp/Controllers/MyController.cs?name=MyController)]

最终结果是为每个请求创建一个 `ApplicationDbContext` 实例，并传递给控制器，以在请求结束后释放前执行工作单元。

有关配置选项的详细信息，请进一步阅读本文。 此外，有关 ASP.NET Core 中的配置和依赖关系注入的详细信息，请参阅 [ASP.NET Core 中的应用启动](/aspnet/core/fundamentals/startup)和 [ASP.NET Core 中的依赖关系注入](/aspnet/core/fundamentals/dependency-injection)。

<!-- See also [Using Dependency Injection](TODO) for advanced dependency injection configuration with EF Core. -->

## <a name="simple-dbcontext-initialization-with-new"></a>使用“new”的简单的 DbContext 初始化

可以按照常规的 .NET 方式构造 `DbContext` 实例，例如，使用 C# 中的 `new`。 可以通过重写 `OnConfiguring` 方法或通过将选项传递给构造函数来执行配置。 例如：

<!--
    public class ApplicationDbContext : DbContext
    {
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test");
        }
    }
-->
[!code-csharp[ApplicationDbContext](../../../samples/core/Miscellaneous/ConfiguringDbContext/WithNew/ApplicationDbContext.cs?name=ApplicationDbContext)]

通过此模式，还可以轻松地通过 `DbContext` 构造函数传递配置（如连接字符串）。 例如：

<!--
    public class ApplicationDbContext : DbContext
    {
        private readonly string _connectionString;

        public ApplicationDbContext(string connectionString)
        {
            _connectionString = connectionString;
        }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer(_connectionString);
        }
    }
-->
[!code-csharp[ApplicationDbContext](../../../samples/core/Miscellaneous/ConfiguringDbContext/WithNewAndArgs/ApplicationDbContext.cs?name=ApplicationDbContext)]

或者，可以使用 `DbContextOptionsBuilder` 创建 `DbContextOptions` 对象，然后将该对象传递到 `DbContext` 构造函数。 这使得为依赖关系注入配置的 `DbContext` 也能显式构造。 例如，使用上述为 ASP.NET Core 的 Web 应用定义的 `ApplicationDbContext` 时：

<!--
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }
    }
-->
[!code-csharp[ApplicationDbContext](../../../samples/core/Miscellaneous/ConfiguringDbContext/WebApp/ApplicationDbContext.cs?name=ApplicationDbContext)]

可以创建 `DbContextOptions`，并可以显式调用构造函数：

<!--
            var contextOptions = new DbContextOptionsBuilder<ApplicationDbContext>()
                .UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test")
                .Options;

            using var context = new ApplicationDbContext(contextOptions);
-->
[!code-csharp[UseNewForWebApp](../../../samples/core/Miscellaneous/ConfiguringDbContext/WebApp/UseNewForWebApp.cs?name=UseNewForWebApp)]

## <a name="using-a-dbcontext-factory-eg-for-blazor"></a>使用 DbContext 工厂（例如对于 Blazor）

某些应用程序类型（例如 [ASP.NET Core Blazor](/aspnet/core/blazor/)）使用依赖关系注入，但不创建与所需的 `DbContext` 生存期一致的服务作用域。 即使存在这样的对齐方式，应用程序也可能需要在此作用域内执行多个工作单元。 例如，单个 HTTP 请求中的多个工作单元。

在这些情况下，可以使用 <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContextFactory%2A> 来注册工厂以创建 `DbContext` 实例。 例如：

<!--
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDbContextFactory<ApplicationDbContext>(options =>
                options.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test"));
        }
-->
[!code-csharp[ConfigureServices](../../../samples/core/Miscellaneous/ConfiguringDbContext/WithContextFactory/FactoryServicesExample.cs?name=ConfigureServices)]

`ApplicationDbContext` 类必须公开具有 `DbContextOptions<ApplicationDbContext>` 参数的公共构造函数。 此模式与上面传统 ASP.NET Core 部分中使用的模式相同。

<!--
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }
    }
-->
[!code-csharp[ApplicationDbContext](../../../samples/core/Miscellaneous/ConfiguringDbContext/WebApp/ApplicationDbContext.cs?name=ApplicationDbContext)]

然后，可以通过构造函数注入在其他服务中使用 `DbContextFactory` 工厂。 例如：

<!--
        private readonly IDbContextFactory<ApplicationDbContext> _contextFactory;

        public MyController(IDbContextFactory<ApplicationDbContext> contextFactory)
        {
            _contextFactory = contextFactory;
        }
-->
[!code-csharp[Construct](../../../samples/core/Miscellaneous/ConfiguringDbContext/WithContextFactory/MyController.cs?name=Construct)]

然后，可以使用注入的工厂在服务代码中构造 DbContext 实例。 例如：

<!--
        public void DoSomething()
        {
            using (var context = _contextFactory.CreateDbContext())
            {
                // ...
            }
        }
-->
[!code-csharp[DoSomething](../../../samples/core/Miscellaneous/ConfiguringDbContext/WithContextFactory/MyController.cs?name=DoSomething)]

请注意，以这种方式创建的 `DbContext` 实例并非由应用程序的服务提供程序进行管理，因此必须由应用程序释放。

有关将 EF Core 与 Blazor 配合使用的详细信息，请参阅 [ASP.NET Core Blazor Server 与 Entity Framework Core](/aspnet/core/blazor/blazor-server-ef-core)。

## <a name="dbcontextoptions"></a>DbContextOptions

所有 `DbContext` 配置的起始点都是 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder>。 可以通过三种方式获取此生成器：

- 在 `AddDbContext` 和相关方法中
- 在 `OnConfiguring` 中
- 使用 `new` 显式构造

上述各节显示了其中每个示例。 无论生成器来自何处，都可以应用相同的配置。 此外，无论如何构造上下文，都将始终调用 `OnConfiguring`。 这意味着即使使用 `AddDbContext`，`OnConfiguring` 也可用于执行其他配置。

### <a name="configuring-the-database-provider"></a>配置数据库提供程序

每个 `DbContext` 实例都必须配置为使用一个且仅一个数据库提供程序。 （`DbContext` 子类型的不同实例可用于不同的数据库提供程序，但单个实例只能使用一个。）使用特定的 `Use*`" 调用配置数据库提供程序。 例如，若要使用 SQL Server 数据库提供程序：

<!--
    public class ApplicationDbContext : DbContext
    {
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test");
        }
    }
-->
[!code-csharp[ApplicationDbContext](../../../samples/core/Miscellaneous/ConfiguringDbContext/WithNew/ApplicationDbContext.cs?name=ApplicationDbContext)]

这些 `Use*`" 方法是由数据库提供程序实现的扩展方法。 这意味着必须先安装数据库提供程序 NuGet 包，然后才能使用扩展方法。

> [!TIP]
> EF Core 数据库提供程序广泛使用[扩展方法](/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)。 如果编译器指示找不到方法，请确保已安装提供程序的 NuGet 包，并且你在代码中已有 `using Microsoft.EntityFrameworkCore;`。

下表包含常见数据库提供程序的示例。

| 数据库系统              | 配置示例                         | NuGet 程序包
|:-----------------------------|-----------------------------------------------|--------------
| SQL Server 或 Azure SQL      | `.UseSqlServer(connectionString)`             | [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/)
| Azure Cosmos DB              | `.UseCosmos(connectionString, databaseName)`  | [Microsoft.EntityFrameworkCore.Cosmos](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Cosmos/)
| SQLite                       | `.UseSqlite(connectionString)`                | [Microsoft.EntityFrameworkCore.Sqlite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite/)
| EF Core 内存中数据库   | `.UseInMemoryDatabase(databaseName)`          | [Microsoft.EntityFrameworkCore.InMemory](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.InMemory/)
| PostgreSQL*                  | `.UseNpgsql(connectionString)`                | [Npgsql.EntityFrameworkCore.PostgreSQL](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL/)
| MySQL/MariaDB*               | `.UseMySql((connectionString)`                | [Pomelo.EntityFrameworkCore.MySql](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MySql/)
| Oracle*                      | `.UseOracle(connectionString)`                | [Oracle.EntityFrameworkCore](https://www.nuget.org/packages/Oracle.EntityFrameworkCore/)

*这些数据库提供程序不由 Microsoft 提供。 有关数据库提供程序的详细信息，请参阅[数据库提供程序](xref:core/providers/index)。

> [!WARNING]
> EF Core 内存中数据库不是为生产用途设计的。 此外，它可能不是测试的最佳选择。 有关详细信息，请参阅[使用 EF Core 的测试代码](xref:core/testing/index)。

有关将连接字符串与 EF Core 结合使用的详细信息，请参阅[连接字符串](xref:core/miscellaneous/connection-strings)。

特定于数据库提供程序的可选配置是在其他特定于提供程序的生成器中执行的。 例如，在连接到 Azure SQL 时，使用 <xref:Microsoft.EntityFrameworkCore.Infrastructure.SqlServerDbContextOptionsBuilder.EnableRetryOnFailure%2A> 为连接复原配置重试：

<!--
    public class ApplicationDbContext : DbContext
    {
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder
                .UseSqlServer(
                    @"Server=(localdb)\mssqllocaldb;Database=Test",
                    providerOptions =>
                        {
                            providerOptions.EnableRetryOnFailure();
                        });
        }
    }
-->
[!code-csharp[ApplicationDbContext](../../../samples/core/Miscellaneous/ConfiguringDbContext/AdditionalProviderConfiguration/ApplicationDbContext.cs?name=ApplicationDbContext)]

> [!TIP]
> 同一数据库提供程序用于 SQL Server 和 Azure SQL。 但是，建议在连接到 SQL Azure 时使用[连接复原](xref:core/miscellaneous/connection-resiliency)。

有关特定于提供程序的配置的详细信息，请参阅[数据库提供程序](xref:core/providers/index)。

### <a name="other-dbcontext-configuration"></a>其他 DbContext 配置

其他 `DbContext` 配置可以链接到 `Use*` 调用之前或之后（这不会有任何差别）。 例如，若要启用敏感数据日志记录：

<!--
    public class ApplicationDbContext : DbContext
    {
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder
                .EnableSensitiveDataLogging()
                .UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test");
        }
    }
-->
[!code-csharp[ApplicationDbContext](../../../samples/core/Miscellaneous/ConfiguringDbContext/AdditionalConfiguration/ApplicationDbContext.cs?name=ApplicationDbContext)]

下表包含 `DbContextOptionsBuilder` 调用的常见方法的示例。

| DbContextOptionsBuilder 方法                                                             | 作用                                                | 了解更多
|:-------------------------------------------------------------------------------------------|-------------------------------------------------------------|--------------
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.UseQueryTrackingBehavior%2A>   | 设置查询的默认跟踪行为              | [查询跟踪行为](xref:core/querying/tracking)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.LogTo%2A>                      | 获取 EF Core 日志的一种简单方法（EF Core 5.0 及更高版本）    | [日志记录、事件和诊断](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.UseLoggerFactory%2A>           | 注册 `Micrsofot.Extensions.Logging` 工厂         | [日志记录、事件和诊断](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> | 在异常和日志记录中包括应用程序数据         | [日志记录、事件和诊断](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableDetailedErrors%2A>       | 更详细的查询错误（以性能为代价）  | [日志记录、事件和诊断](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.ConfigureWarnings%2A>          | 忽略或引发警告和其他事件               | [日志记录、事件和诊断](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.AddInterceptors%2A>            | 注册 EF Core 侦听器                              | [日志记录、事件和诊断](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseLazyLoadingProxies%2A>            | 使用动态代理进行延迟加载                        | [延迟加载](xref:core/querying/related-data/lazy)
| <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseChangeTrackingProxies%2A>         | 使用动态代理进行更改跟踪                     | 即将推出...

> [!NOTE]
> <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseLazyLoadingProxies%2A> 和 <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseChangeTrackingProxies%2A> 是 [Microsoft.EntityFrameworkCore.Proxies](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Proxies/) NuGet 包中的扩展方法。 建议的方式是使用这种类型的“.UseSomething()”调用来配置和/或使用其他包中包含的 EF Core 扩展。

### <a name="dbcontextoptions-verses-dbcontextoptionstcontext"></a>`DbContextOptions` 与 `DbContextOptions<TContext>`

大多数接受 `DbContextOptions` 的 `DbContext` 子类应使用 [泛型](/dotnet/csharp/programming-guide/generics/) `DbContextOptions<TContext>` 变体。 例如：

<!--
    public sealed class SealedApplicationDbContext : DbContext
    {
        public SealedApplicationDbContext(DbContextOptions<SealedApplicationDbContext> contextOptions)
            : base(contextOptions)
        {
        }
    }
-->
[!code-csharp[SealedApplicationDbContext](../../../samples/core/Miscellaneous/ConfiguringDbContext/InheritDbContext/ApplicationDbContext.cs?name=SealedApplicationDbContext)]

这可确保从依赖关系注入中解析特定 `DbContext` 子类型的正确选项，即使注册了多个 `DbContext` 子类型也是如此。

> [!TIP]
> 你的 DbContext 不需要密封，但对于没有被设计为继承的类，密封是最佳做法。

但是，如果 `DbContext` 子类型本身旨在继承，则它应公开采用非泛型 `DbContextOptions` 的受保护构造函数。 例如：

<!--
    public abstract class ApplicationDbContextBase : DbContext
    {
        protected ApplicationDbContextBase(DbContextOptions contextOptions)
            : base(contextOptions)
        {
        }
    }
-->
[!code-csharp[ApplicationDbContextBase](../../../samples/core/Miscellaneous/ConfiguringDbContext/InheritDbContext/ApplicationDbContext.cs?name=ApplicationDbContextBase)]

这允许多个具体子类使用其不同的泛型 `DbContextOptions<TContext>` 实例来调用此基构造函数。 例如：

<!--
    public sealed class ApplicationDbContext1 : ApplicationDbContextBase
    {
        public ApplicationDbContext1(DbContextOptions<ApplicationDbContext1> contextOptions)
            : base(contextOptions)
        {
        }
    }

    public sealed class ApplicationDbContext2 : ApplicationDbContextBase
    {
        public ApplicationDbContext2(DbContextOptions<ApplicationDbContext2> contextOptions)
            : base(contextOptions)
        {
        }
    }
-->
[!code-csharp[ApplicationDbContext12](../../../samples/core/Miscellaneous/ConfiguringDbContext/InheritDbContext/ApplicationDbContext.cs?name=ApplicationDbContext12)]

请注意，这与直接从 `DbContext` 继承的模式完全相同。 也就是说，出于此原因，`DbContext` 构造函数本身将接受非泛型 `DbContextOptions`。

旨在同时进行实例化和继承的 `DbContext` 子类应公开构造函数的两种形式。 例如：

<!--
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> contextOptions)
            : base(contextOptions)
        {
        }

        protected ApplicationDbContext(DbContextOptions contextOptions)
            : base(contextOptions)
        {
        }
    }
-->
[!code-csharp[ApplicationDbContext](../../../samples/core/Miscellaneous/ConfiguringDbContext/InheritDbContext/ApplicationDbContext.cs?name=ApplicationDbContext)]

## <a name="design-time-dbcontext-configuration"></a>设计时 DbContext 配置

EF Core 设计时工具（如 [EF Core迁移](xref:core/managing-schemas/migrations/index)）需要能够发现并创建 `DbContext` 类型的工作实例，以收集有关应用程序的实体类型及其如何映射到数据库架构的详细信息。 只要该工具可以通过与在运行时的配置方式类似的方式轻松地创建 `DbContext`，就可以自动执行此过程。

虽然向 `DbContext` 提供必要配置信息的任何模式都可以在运行时正常运行，但在设计时需要使用 `DbContext` 的工具只能使用有限数量的模式。 [设计时上下文创建](xref:core/cli/dbcontext-creation)中包含更多详细信息。

## <a name="avoiding-dbcontext-threading-issues"></a>避免 DbContext 线程处理问题

Entity Framework Core 不支持在同一 `DbContext` 实例上运行多个并行操作。 这包括异步查询的并行执行以及从多个线程进行的任何显式并发使用。 因此，始终立即 `await` 异步调用，或对并行执行的操作使用单独的 `DbContext` 实例。

当 EF Core 检测到尝试同时使用 `DbContext` 实例的情况时，你将看到 `InvalidOperationException`，其中包含类似于以下内容的消息：

> 在上一个操作完成之前，第二个操作已在此上下文中启动。 这通常是由使用同一个 DbContext 实例的不同线程引起的，但不保证实例成员是线程安全的。

检测不到并发访问时，可能会导致未定义的行为、应用程序崩溃和数据损坏。

一些常见错误可能会无意中导致并发访问同一 `DbContext` 实例：

### <a name="asynchronous-operation-pitfalls"></a>异步操作缺陷

使用异步方法，EF Core 可以启动以非阻挡式访问数据库的操作。 但是，如果调用方不等待其中一个方法完成，而是继续对 `DbContext` 执行其他操作，则 `DbContext` 的状态可能会（并且很可能会）损坏。

始终立即等待 EF Core 异步方法。

### <a name="implicitly-sharing-dbcontext-instances-via-dependency-injection"></a>通过依赖关系注入隐式共享 DbContext 实例

默认情况下 [`AddDbContext`](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) 扩展方法使用[范围内生存期](/aspnet/core/fundamentals/dependency-injection#service-lifetimes)来注册 `DbContext` 类型。

这样可以避免在大多数 ASP.NET Core 应用程序中出现并发访问问题，因为在给定时间内只有一个线程在执行每个客户端请求，并且每个请求都有单独的依赖关系注入范围（因此有单独的 `DbContext` 实例）。 对于 Blazor Server 托管模型，一个逻辑请求用来维护 Blazor 用户线路，因此，如果使用默认注入范围，则每个用户线路只能提供一个范围内的 DbContext 实例。

任何并行显式执行多个线程的代码都应确保 `DbContext` 实例不会同时访问。

使用依赖关系注入可以通过以下方式实现：将上下文注册为范围内，并为每个线程创建范围（使用 `IServiceScopeFactory`），或将 `DbContext` 注册为暂时性（使用采用 `ServiceLifetime` 参数的 `AddDbContext` 的重载）。

## <a name="more-reading"></a>进一步阅读

- 请阅读[依赖关系注入](/aspnet/core/fundamentals/dependency-injection)以了解有关使用 DI 的详细信息。
- 请阅读[测试](xref:core/testing/index)以了解详细信息。
