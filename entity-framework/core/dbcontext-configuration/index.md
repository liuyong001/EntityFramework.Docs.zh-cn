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
# <a name="dbcontext-lifetime-configuration-and-initialization"></a><span data-ttu-id="e3f24-103">DbContext 生存期、配置和初始化</span><span class="sxs-lookup"><span data-stu-id="e3f24-103">DbContext Lifetime, Configuration, and Initialization</span></span>

<span data-ttu-id="e3f24-104">本文介绍初始化和配置 <xref:Microsoft.EntityFrameworkCore.DbContext> 实例的基本模式。</span><span class="sxs-lookup"><span data-stu-id="e3f24-104">This article shows basic patterns for initialization and configuration of a <xref:Microsoft.EntityFrameworkCore.DbContext> instance.</span></span>

## <a name="the-dbcontext-lifetime"></a><span data-ttu-id="e3f24-105">DbContext 生存期</span><span class="sxs-lookup"><span data-stu-id="e3f24-105">The DbContext lifetime</span></span>

<span data-ttu-id="e3f24-106">`DbContext` 的生存期从创建实例时开始，并在[释放](/dotnet/standard/garbage-collection/unmanaged)实例时结束。</span><span class="sxs-lookup"><span data-stu-id="e3f24-106">The lifetime of a `DbContext` begins when the instance is created and ends when the instance is [disposed](/dotnet/standard/garbage-collection/unmanaged).</span></span> <span data-ttu-id="e3f24-107">`DbContext` 实例旨在用于单个[工作单元](https://www.martinfowler.com/eaaCatalog/unitOfWork.html)。</span><span class="sxs-lookup"><span data-stu-id="e3f24-107">A `DbContext` instance is designed to be used for a _single_ [unit-of-work](https://www.martinfowler.com/eaaCatalog/unitOfWork.html).</span></span> <span data-ttu-id="e3f24-108">这意味着 `DbContext` 实例的生存期通常很短。</span><span class="sxs-lookup"><span data-stu-id="e3f24-108">This means that the lifetime of a `DbContext` instance is usually very short.</span></span>

> [!TIP]
> <span data-ttu-id="e3f24-109">引用上述链接中 Martin Fowler 的话，“工作单元将持续跟踪在可能影响数据库的业务事务中执行的所有操作。</span><span class="sxs-lookup"><span data-stu-id="e3f24-109">To quote Martin Fowler from the link above, "A Unit of Work keeps track of everything you do during a business transaction that can affect the database.</span></span> <span data-ttu-id="e3f24-110">当你完成操作后，它将找出更改数据库作为工作结果时需要执行的所有操作。”</span><span class="sxs-lookup"><span data-stu-id="e3f24-110">When you're done, it figures out everything that needs to be done to alter the database as a result of your work."</span></span>

<span data-ttu-id="e3f24-111">使用 Entity Framework Core (EF Core) 时的典型工作单元包括：</span><span class="sxs-lookup"><span data-stu-id="e3f24-111">A typical unit-of-work when using Entity Framework Core (EF Core) involves:</span></span>

- <span data-ttu-id="e3f24-112">创建 `DbContext` 实例</span><span class="sxs-lookup"><span data-stu-id="e3f24-112">Creation of a `DbContext` instance</span></span>
- <span data-ttu-id="e3f24-113">根据上下文跟踪实体实例。</span><span class="sxs-lookup"><span data-stu-id="e3f24-113">Tracking of entity instances by the context.</span></span> <span data-ttu-id="e3f24-114">实体将在以下情况下被跟踪</span><span class="sxs-lookup"><span data-stu-id="e3f24-114">Entities become tracked by</span></span>
  - <span data-ttu-id="e3f24-115">正在[从查询返回](xref:core/querying/tracking)</span><span class="sxs-lookup"><span data-stu-id="e3f24-115">Being [returned from a query](xref:core/querying/tracking)</span></span>
  - <span data-ttu-id="e3f24-116">正在[添加或附加到上下文](xref:core/saving/disconnected-entities)</span><span class="sxs-lookup"><span data-stu-id="e3f24-116">Being [added or attached to the context](xref:core/saving/disconnected-entities)</span></span>
- <span data-ttu-id="e3f24-117">根据需要对所跟踪的实体进行更改以实现业务规则</span><span class="sxs-lookup"><span data-stu-id="e3f24-117">Changes are made to the tracked entities as needed to implement the business rule</span></span>
- <span data-ttu-id="e3f24-118">调用 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 或 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A>。</span><span class="sxs-lookup"><span data-stu-id="e3f24-118"><xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> or <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChangesAsync%2A> is called.</span></span> <span data-ttu-id="e3f24-119">EF Core 检测所做的更改，并将这些更改写入数据库。</span><span class="sxs-lookup"><span data-stu-id="e3f24-119">EF Core detects the changes made and writes them to the database.</span></span>
- <span data-ttu-id="e3f24-120">释放 `DbContext` 实例</span><span class="sxs-lookup"><span data-stu-id="e3f24-120">The `DbContext` instance is disposed</span></span>

> [!IMPORTANT]
>
> - <span data-ttu-id="e3f24-121">使用后释放 <xref:Microsoft.EntityFrameworkCore.DbContext> 非常重要。</span><span class="sxs-lookup"><span data-stu-id="e3f24-121">It is very important to dispose the <xref:Microsoft.EntityFrameworkCore.DbContext> after use.</span></span> <span data-ttu-id="e3f24-122">这可确保释放所有非托管资源，并注销任何事件或其他挂钩，以防止在实例保持引用时出现内存泄漏。</span><span class="sxs-lookup"><span data-stu-id="e3f24-122">This ensures both that any unmanaged resources are freed, and that any events or other hooks are unregistered so as to prevent memory leaks in case the instance remains referenced.</span></span>
> - <span data-ttu-id="e3f24-123">[DbContext 不是线程安全的](#avoiding-dbcontext-threading-issues)。</span><span class="sxs-lookup"><span data-stu-id="e3f24-123">[DbContext is **not thread-safe**](#avoiding-dbcontext-threading-issues).</span></span> <span data-ttu-id="e3f24-124">不要在线程之间共享上下文。</span><span class="sxs-lookup"><span data-stu-id="e3f24-124">Do not share contexts between threads.</span></span> <span data-ttu-id="e3f24-125">请确保在继续使用上下文实例之前，[等待](/dotnet/csharp/language-reference/operators/await)所有异步调用。</span><span class="sxs-lookup"><span data-stu-id="e3f24-125">Make sure to [await](/dotnet/csharp/language-reference/operators/await) all async calls before continuing to use the context instance.</span></span>
> - <span data-ttu-id="e3f24-126">EF Core 代码引发的 <xref:System.InvalidOperationException> 可以使上下文进入不可恢复的状态。</span><span class="sxs-lookup"><span data-stu-id="e3f24-126">An <xref:System.InvalidOperationException> thrown by EF Core code can put the context into an unrecoverable state.</span></span> <span data-ttu-id="e3f24-127">此类异常指示程序错误，并且不旨在从其中恢复。</span><span class="sxs-lookup"><span data-stu-id="e3f24-127">Such exceptions indicate a program error and are not designed to be recovered from.</span></span>

## <a name="dbcontext-in-dependency-injection-for-aspnet-core"></a><span data-ttu-id="e3f24-128">ASP.NET Core 依赖关系注入中的 DbContext</span><span class="sxs-lookup"><span data-stu-id="e3f24-128">DbContext in dependency injection for ASP.NET Core</span></span>

<span data-ttu-id="e3f24-129">在许多 Web 应用程序中，每个 HTTP 请求都对应于单个工作单元。</span><span class="sxs-lookup"><span data-stu-id="e3f24-129">In many web applications, each HTTP request corresponds to a single unit-of-work.</span></span> <span data-ttu-id="e3f24-130">这使得上下文生存期与请求的生存期相关，成为 Web 应用程序的一个良好默认值。</span><span class="sxs-lookup"><span data-stu-id="e3f24-130">This makes tying the context lifetime to that of the request a good default for web applications.</span></span>

<span data-ttu-id="e3f24-131">[使用依赖关系注入配置](/aspnet/core/fundamentals/startup) ASP.NET Core 应用程序。</span><span class="sxs-lookup"><span data-stu-id="e3f24-131">ASP.NET Core applications are [configured using dependency injection](/aspnet/core/fundamentals/startup).</span></span> <span data-ttu-id="e3f24-132">可以使用 `Startup.cs` 的 [`ConfigureServices`](/aspnet/core/fundamentals/startup#the-configureservices-method) 方法中的 <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> 将 EF Core 添加到此配置。</span><span class="sxs-lookup"><span data-stu-id="e3f24-132">EF Core can be added to this configuration using <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> in the [`ConfigureServices`](/aspnet/core/fundamentals/startup#the-configureservices-method) method of `Startup.cs`.</span></span> <span data-ttu-id="e3f24-133">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-133">For example:</span></span>

<!--
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();
            
            services.AddDbContext<ApplicationDbContext>(
                options => options.UseSqlServer("name=ConnectionStrings:DefaultConnection"));
        }
-->
[!code-csharp[ConfigureServices](../../../samples/core/Miscellaneous/ConfiguringDbContext/WebApp/Startup.cs?name=ConfigureServices)]

<span data-ttu-id="e3f24-134">此示例将名为 `ApplicationDbContext` 的 `DbContext` 子类注册为 ASP.NET Core 应用程序服务提供程序（也称为</span><span class="sxs-lookup"><span data-stu-id="e3f24-134">This example registers a `DbContext` subclass called `ApplicationDbContext` as a scoped service in the ASP.NET Core application service provider (a.k.a.</span></span> <span data-ttu-id="e3f24-135">依赖关系注入容器）中的作用域服务。</span><span class="sxs-lookup"><span data-stu-id="e3f24-135">the dependency injection container).</span></span> <span data-ttu-id="e3f24-136">上下文配置为使用 SQL Server 数据库提供程序，并将从 ASP.NET Core 配置读取连接字符串。</span><span class="sxs-lookup"><span data-stu-id="e3f24-136">The context is configured to use the SQL Server database provider and will read the connection string from ASP.NET Core configuration.</span></span> <span data-ttu-id="e3f24-137">在 `ConfigureServices` 中的何处调用 `AddDbContext` 通常不重要。</span><span class="sxs-lookup"><span data-stu-id="e3f24-137">It typically does not matter _where_ in `ConfigureServices` the call to `AddDbContext` is made.</span></span>

<span data-ttu-id="e3f24-138">`ApplicationDbContext` 类必须公开具有 `DbContextOptions<ApplicationDbContext>` 参数的公共构造函数。</span><span class="sxs-lookup"><span data-stu-id="e3f24-138">The `ApplicationDbContext` class must expose a public constructor with a `DbContextOptions<ApplicationDbContext>` parameter.</span></span> <span data-ttu-id="e3f24-139">这是将 `AddDbContext` 的上下文配置传递到 `DbContext` 的方式。</span><span class="sxs-lookup"><span data-stu-id="e3f24-139">This is how context configuration from `AddDbContext` is passed to the `DbContext`.</span></span> <span data-ttu-id="e3f24-140">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-140">For example:</span></span>

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

<span data-ttu-id="e3f24-141">然后，`ApplicationDbContext` 可以通过构造函数注入在 ASP.NET Core 控制器或其他服务中使用。</span><span class="sxs-lookup"><span data-stu-id="e3f24-141">`ApplicationDbContext` can then be used in ASP.NET Core controllers or other services through constructor injection.</span></span> <span data-ttu-id="e3f24-142">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-142">For example:</span></span>

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

<span data-ttu-id="e3f24-143">最终结果是为每个请求创建一个 `ApplicationDbContext` 实例，并传递给控制器，以在请求结束后释放前执行工作单元。</span><span class="sxs-lookup"><span data-stu-id="e3f24-143">The final result is an `ApplicationDbContext` instance created for each request and passed to the controller to perform a unit-of-work before being disposed when the request ends.</span></span>

<span data-ttu-id="e3f24-144">有关配置选项的详细信息，请进一步阅读本文。</span><span class="sxs-lookup"><span data-stu-id="e3f24-144">Read further in this article to learn more about configuration options.</span></span> <span data-ttu-id="e3f24-145">此外，有关 ASP.NET Core 中的配置和依赖关系注入的详细信息，请参阅 [ASP.NET Core 中的应用启动](/aspnet/core/fundamentals/startup)和 [ASP.NET Core 中的依赖关系注入](/aspnet/core/fundamentals/dependency-injection)。</span><span class="sxs-lookup"><span data-stu-id="e3f24-145">In addition, see [App startup in ASP.NET Core](/aspnet/core/fundamentals/startup) and [Dependency injection in ASP.NET Core](/aspnet/core/fundamentals/dependency-injection) for more information on configuration and dependency injection in ASP.NET Core.</span></span>

<!-- See also [Using Dependency Injection](TODO) for advanced dependency injection configuration with EF Core. -->

## <a name="simple-dbcontext-initialization-with-new"></a><span data-ttu-id="e3f24-146">使用“new”的简单的 DbContext 初始化</span><span class="sxs-lookup"><span data-stu-id="e3f24-146">Simple DbContext initialization with 'new'</span></span>

<span data-ttu-id="e3f24-147">可以按照常规的 .NET 方式构造 `DbContext` 实例，例如，使用 C# 中的 `new`。</span><span class="sxs-lookup"><span data-stu-id="e3f24-147">`DbContext` instances can be constructed in the normal .NET way, for example with `new` in C#.</span></span> <span data-ttu-id="e3f24-148">可以通过重写 `OnConfiguring` 方法或通过将选项传递给构造函数来执行配置。</span><span class="sxs-lookup"><span data-stu-id="e3f24-148">Configuration can be performed by overriding the `OnConfiguring` method, or by passing options to the constructor.</span></span> <span data-ttu-id="e3f24-149">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-149">For example:</span></span>

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

<span data-ttu-id="e3f24-150">通过此模式，还可以轻松地通过 `DbContext` 构造函数传递配置（如连接字符串）。</span><span class="sxs-lookup"><span data-stu-id="e3f24-150">This pattern also makes it easy to pass configuration like the connection string via the `DbContext` constructor.</span></span> <span data-ttu-id="e3f24-151">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-151">For example:</span></span>

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

<span data-ttu-id="e3f24-152">或者，可以使用 `DbContextOptionsBuilder` 创建 `DbContextOptions` 对象，然后将该对象传递到 `DbContext` 构造函数。</span><span class="sxs-lookup"><span data-stu-id="e3f24-152">Alternately, `DbContextOptionsBuilder` can be used to create a `DbContextOptions` object that is then passed to the `DbContext` constructor.</span></span> <span data-ttu-id="e3f24-153">这使得为依赖关系注入配置的 `DbContext` 也能显式构造。</span><span class="sxs-lookup"><span data-stu-id="e3f24-153">This allows a `DbContext` configured for dependency injection to also be constructed explicitly.</span></span> <span data-ttu-id="e3f24-154">例如，使用上述为 ASP.NET Core 的 Web 应用定义的 `ApplicationDbContext` 时：</span><span class="sxs-lookup"><span data-stu-id="e3f24-154">For example, when using `ApplicationDbContext` defined for ASP.NET Core web apps above:</span></span>

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

<span data-ttu-id="e3f24-155">可以创建 `DbContextOptions`，并可以显式调用构造函数：</span><span class="sxs-lookup"><span data-stu-id="e3f24-155">The `DbContextOptions` can be created and the constructor can be called explicitly:</span></span>

<!--
            var contextOptions = new DbContextOptionsBuilder<ApplicationDbContext>()
                .UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test")
                .Options;

            using var context = new ApplicationDbContext(contextOptions);
-->
[!code-csharp[UseNewForWebApp](../../../samples/core/Miscellaneous/ConfiguringDbContext/WebApp/UseNewForWebApp.cs?name=UseNewForWebApp)]

## <a name="using-a-dbcontext-factory-eg-for-blazor"></a><span data-ttu-id="e3f24-156">使用 DbContext 工厂（例如对于 Blazor）</span><span class="sxs-lookup"><span data-stu-id="e3f24-156">Using a DbContext factory (e.g. for Blazor)</span></span>

<span data-ttu-id="e3f24-157">某些应用程序类型（例如 [ASP.NET Core Blazor](/aspnet/core/blazor/)）使用依赖关系注入，但不创建与所需的 `DbContext` 生存期一致的服务作用域。</span><span class="sxs-lookup"><span data-stu-id="e3f24-157">Some application types (e.g. [ASP.NET Core Blazor](/aspnet/core/blazor/)) use dependency injection but do not create a service scope that aligns with the desired `DbContext` lifetime.</span></span> <span data-ttu-id="e3f24-158">即使存在这样的对齐方式，应用程序也可能需要在此作用域内执行多个工作单元。</span><span class="sxs-lookup"><span data-stu-id="e3f24-158">Even where such an alignment does exist, the application may need to perform multiple units-of-work within this scope.</span></span> <span data-ttu-id="e3f24-159">例如，单个 HTTP 请求中的多个工作单元。</span><span class="sxs-lookup"><span data-stu-id="e3f24-159">For example, multiple units-of-work within a single HTTP request.</span></span>

<span data-ttu-id="e3f24-160">在这些情况下，可以使用 <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContextFactory%2A> 来注册工厂以创建 `DbContext` 实例。</span><span class="sxs-lookup"><span data-stu-id="e3f24-160">In these cases, <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContextFactory%2A> can be used to register a factory for creation of `DbContext` instances.</span></span> <span data-ttu-id="e3f24-161">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-161">For example:</span></span>

<!--
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDbContextFactory<ApplicationDbContext>(options =>
                options.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test"));
        }
-->
[!code-csharp[ConfigureServices](../../../samples/core/Miscellaneous/ConfiguringDbContext/WithContextFactory/FactoryServicesExample.cs?name=ConfigureServices)]

<span data-ttu-id="e3f24-162">`ApplicationDbContext` 类必须公开具有 `DbContextOptions<ApplicationDbContext>` 参数的公共构造函数。</span><span class="sxs-lookup"><span data-stu-id="e3f24-162">The `ApplicationDbContext` class must expose a public constructor with a `DbContextOptions<ApplicationDbContext>` parameter.</span></span> <span data-ttu-id="e3f24-163">此模式与上面传统 ASP.NET Core 部分中使用的模式相同。</span><span class="sxs-lookup"><span data-stu-id="e3f24-163">This is the same pattern as used in the traditional ASP.NET Core section above.</span></span>

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

<span data-ttu-id="e3f24-164">然后，可以通过构造函数注入在其他服务中使用 `DbContextFactory` 工厂。</span><span class="sxs-lookup"><span data-stu-id="e3f24-164">The `DbContextFactory` factory can then be used in other services through constructor injection.</span></span> <span data-ttu-id="e3f24-165">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-165">For example:</span></span>

<!--
        private readonly IDbContextFactory<ApplicationDbContext> _contextFactory;

        public MyController(IDbContextFactory<ApplicationDbContext> contextFactory)
        {
            _contextFactory = contextFactory;
        }
-->
[!code-csharp[Construct](../../../samples/core/Miscellaneous/ConfiguringDbContext/WithContextFactory/MyController.cs?name=Construct)]

<span data-ttu-id="e3f24-166">然后，可以使用注入的工厂在服务代码中构造 DbContext 实例。</span><span class="sxs-lookup"><span data-stu-id="e3f24-166">The injected factory can then be used to construct DbContext instances in the service code.</span></span> <span data-ttu-id="e3f24-167">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-167">For example:</span></span>

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

<span data-ttu-id="e3f24-168">请注意，以这种方式创建的 `DbContext` 实例并非由应用程序的服务提供程序进行管理，因此必须由应用程序释放。</span><span class="sxs-lookup"><span data-stu-id="e3f24-168">Notice that the `DbContext` instances created in this way are **not** managed by the application's service provider and therefore must be disposed by the application.</span></span>

<span data-ttu-id="e3f24-169">有关将 EF Core 与 Blazor 配合使用的详细信息，请参阅 [ASP.NET Core Blazor Server 与 Entity Framework Core](/aspnet/core/blazor/blazor-server-ef-core)。</span><span class="sxs-lookup"><span data-stu-id="e3f24-169">See [ASP.NET Core Blazor Server with Entity Framework Core](/aspnet/core/blazor/blazor-server-ef-core) for more information on using EF Core with Blazor.</span></span>

## <a name="dbcontextoptions"></a><span data-ttu-id="e3f24-170">DbContextOptions</span><span class="sxs-lookup"><span data-stu-id="e3f24-170">DbContextOptions</span></span>

<span data-ttu-id="e3f24-171">所有 `DbContext` 配置的起始点都是 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder>。</span><span class="sxs-lookup"><span data-stu-id="e3f24-171">The starting point for all `DbContext` configuration is <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder>.</span></span> <span data-ttu-id="e3f24-172">可以通过三种方式获取此生成器：</span><span class="sxs-lookup"><span data-stu-id="e3f24-172">There are three ways to get this builder:</span></span>

- <span data-ttu-id="e3f24-173">在 `AddDbContext` 和相关方法中</span><span class="sxs-lookup"><span data-stu-id="e3f24-173">In `AddDbContext` and related methods</span></span>
- <span data-ttu-id="e3f24-174">在 `OnConfiguring` 中</span><span class="sxs-lookup"><span data-stu-id="e3f24-174">In `OnConfiguring`</span></span>
- <span data-ttu-id="e3f24-175">使用 `new` 显式构造</span><span class="sxs-lookup"><span data-stu-id="e3f24-175">Constructed explicitly with `new`</span></span>

<span data-ttu-id="e3f24-176">上述各节显示了其中每个示例。</span><span class="sxs-lookup"><span data-stu-id="e3f24-176">Examples of each of these are shown in the preceding sections.</span></span> <span data-ttu-id="e3f24-177">无论生成器来自何处，都可以应用相同的配置。</span><span class="sxs-lookup"><span data-stu-id="e3f24-177">The same configuration can be applied regardless of where the builder comes from.</span></span> <span data-ttu-id="e3f24-178">此外，无论如何构造上下文，都将始终调用 `OnConfiguring`。</span><span class="sxs-lookup"><span data-stu-id="e3f24-178">In addition, `OnConfiguring` is always called regardless of how the context is constructed.</span></span> <span data-ttu-id="e3f24-179">这意味着即使使用 `AddDbContext`，`OnConfiguring` 也可用于执行其他配置。</span><span class="sxs-lookup"><span data-stu-id="e3f24-179">This means `OnConfiguring` can be used to perform additional configuration even when `AddDbContext` is being used.</span></span>

### <a name="configuring-the-database-provider"></a><span data-ttu-id="e3f24-180">配置数据库提供程序</span><span class="sxs-lookup"><span data-stu-id="e3f24-180">Configuring the database provider</span></span>

<span data-ttu-id="e3f24-181">每个 `DbContext` 实例都必须配置为使用一个且仅一个数据库提供程序。</span><span class="sxs-lookup"><span data-stu-id="e3f24-181">Each `DbContext` instance must be configured to use one and only one database provider.</span></span> <span data-ttu-id="e3f24-182">（`DbContext` 子类型的不同实例可用于不同的数据库提供程序，但单个实例只能使用一个。）使用特定的 `Use*`" 调用配置数据库提供程序。</span><span class="sxs-lookup"><span data-stu-id="e3f24-182">(Different instances of a `DbContext` subtype can be used with different database providers, but a single instance must only use one.) A database provider is configured using a specific `Use*`" call.</span></span> <span data-ttu-id="e3f24-183">例如，若要使用 SQL Server 数据库提供程序：</span><span class="sxs-lookup"><span data-stu-id="e3f24-183">For example, to use the SQL Server database provider:</span></span>

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

<span data-ttu-id="e3f24-184">这些 `Use*`" 方法是由数据库提供程序实现的扩展方法。</span><span class="sxs-lookup"><span data-stu-id="e3f24-184">These `Use*`" methods are extension methods implemented by the database provider.</span></span> <span data-ttu-id="e3f24-185">这意味着必须先安装数据库提供程序 NuGet 包，然后才能使用扩展方法。</span><span class="sxs-lookup"><span data-stu-id="e3f24-185">This means that the database provider NuGet package must be installed before the extension method can be used.</span></span>

> [!TIP]
> <span data-ttu-id="e3f24-186">EF Core 数据库提供程序广泛使用[扩展方法](/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)。</span><span class="sxs-lookup"><span data-stu-id="e3f24-186">EF Core database providers make extensive use of [extension methods](/dotnet/csharp/programming-guide/classes-and-structs/extension-methods).</span></span> <span data-ttu-id="e3f24-187">如果编译器指示找不到方法，请确保已安装提供程序的 NuGet 包，并且你在代码中已有 `using Microsoft.EntityFrameworkCore;`。</span><span class="sxs-lookup"><span data-stu-id="e3f24-187">If the compiler indicates that a method cannot be found, then make sure that the provider's NuGet package is installed and that you have `using Microsoft.EntityFrameworkCore;` in your code.</span></span>

<span data-ttu-id="e3f24-188">下表包含常见数据库提供程序的示例。</span><span class="sxs-lookup"><span data-stu-id="e3f24-188">The following table contains examples for common database providers.</span></span>

| <span data-ttu-id="e3f24-189">数据库系统</span><span class="sxs-lookup"><span data-stu-id="e3f24-189">Database system</span></span>              | <span data-ttu-id="e3f24-190">配置示例</span><span class="sxs-lookup"><span data-stu-id="e3f24-190">Example configuration</span></span>                         | <span data-ttu-id="e3f24-191">NuGet 程序包</span><span class="sxs-lookup"><span data-stu-id="e3f24-191">NuGet package</span></span>
|:-----------------------------|-----------------------------------------------|--------------
| <span data-ttu-id="e3f24-192">SQL Server 或 Azure SQL</span><span class="sxs-lookup"><span data-stu-id="e3f24-192">SQL Server or Azure SQL</span></span>      | `.UseSqlServer(connectionString)`             | [<span data-ttu-id="e3f24-193">Microsoft.EntityFrameworkCore.SqlServer</span><span class="sxs-lookup"><span data-stu-id="e3f24-193">Microsoft.EntityFrameworkCore.SqlServer</span></span>](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/)
| <span data-ttu-id="e3f24-194">Azure Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="e3f24-194">Azure Cosmos DB</span></span>              | `.UseCosmos(connectionString, databaseName)`  | [<span data-ttu-id="e3f24-195">Microsoft.EntityFrameworkCore.Cosmos</span><span class="sxs-lookup"><span data-stu-id="e3f24-195">Microsoft.EntityFrameworkCore.Cosmos</span></span>](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Cosmos/)
| <span data-ttu-id="e3f24-196">SQLite</span><span class="sxs-lookup"><span data-stu-id="e3f24-196">SQLite</span></span>                       | `.UseSqlite(connectionString)`                | [<span data-ttu-id="e3f24-197">Microsoft.EntityFrameworkCore.Sqlite</span><span class="sxs-lookup"><span data-stu-id="e3f24-197">Microsoft.EntityFrameworkCore.Sqlite</span></span>](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite/)
| <span data-ttu-id="e3f24-198">EF Core 内存中数据库</span><span class="sxs-lookup"><span data-stu-id="e3f24-198">EF Core in-memory database</span></span>   | `.UseInMemoryDatabase(databaseName)`          | [<span data-ttu-id="e3f24-199">Microsoft.EntityFrameworkCore.InMemory</span><span class="sxs-lookup"><span data-stu-id="e3f24-199">Microsoft.EntityFrameworkCore.InMemory</span></span>](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.InMemory/)
| <span data-ttu-id="e3f24-200">PostgreSQL\*</span><span class="sxs-lookup"><span data-stu-id="e3f24-200">PostgreSQL\*</span></span>                  | `.UseNpgsql(connectionString)`                | [<span data-ttu-id="e3f24-201">Npgsql.EntityFrameworkCore.PostgreSQL</span><span class="sxs-lookup"><span data-stu-id="e3f24-201">Npgsql.EntityFrameworkCore.PostgreSQL</span></span>](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL/)
| <span data-ttu-id="e3f24-202">MySQL/MariaDB\*</span><span class="sxs-lookup"><span data-stu-id="e3f24-202">MySQL/MariaDB\*</span></span>               | `.UseMySql((connectionString)`                | [<span data-ttu-id="e3f24-203">Pomelo.EntityFrameworkCore.MySql</span><span class="sxs-lookup"><span data-stu-id="e3f24-203">Pomelo.EntityFrameworkCore.MySql</span></span>](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MySql/)
| <span data-ttu-id="e3f24-204">Oracle\*</span><span class="sxs-lookup"><span data-stu-id="e3f24-204">Oracle\*</span></span>                      | `.UseOracle(connectionString)`                | [<span data-ttu-id="e3f24-205">Oracle.EntityFrameworkCore</span><span class="sxs-lookup"><span data-stu-id="e3f24-205">Oracle.EntityFrameworkCore</span></span>](https://www.nuget.org/packages/Oracle.EntityFrameworkCore/)

<span data-ttu-id="e3f24-206">\*这些数据库提供程序不由 Microsoft 提供。</span><span class="sxs-lookup"><span data-stu-id="e3f24-206">\*These database providers are not shipped by Microsoft.</span></span> <span data-ttu-id="e3f24-207">有关数据库提供程序的详细信息，请参阅[数据库提供程序](xref:core/providers/index)。</span><span class="sxs-lookup"><span data-stu-id="e3f24-207">See [Database Providers](xref:core/providers/index) for more information about database providers.</span></span>

> [!WARNING]
> <span data-ttu-id="e3f24-208">EF Core 内存中数据库不是为生产用途设计的。</span><span class="sxs-lookup"><span data-stu-id="e3f24-208">The EF Core in-memory database is not designed for production use.</span></span> <span data-ttu-id="e3f24-209">此外，它可能不是测试的最佳选择。</span><span class="sxs-lookup"><span data-stu-id="e3f24-209">In addition, it may not be the best choice even for testing.</span></span> <span data-ttu-id="e3f24-210">有关详细信息，请参阅[使用 EF Core 的测试代码](xref:core/testing/index)。</span><span class="sxs-lookup"><span data-stu-id="e3f24-210">See [Testing Code That Uses EF Core](xref:core/testing/index) for more information.</span></span>

<span data-ttu-id="e3f24-211">有关将连接字符串与 EF Core 结合使用的详细信息，请参阅[连接字符串](xref:core/miscellaneous/connection-strings)。</span><span class="sxs-lookup"><span data-stu-id="e3f24-211">See [Connection Strings](xref:core/miscellaneous/connection-strings) for more information on using connection strings with EF Core.</span></span>

<span data-ttu-id="e3f24-212">特定于数据库提供程序的可选配置是在其他特定于提供程序的生成器中执行的。</span><span class="sxs-lookup"><span data-stu-id="e3f24-212">Optional configuration specific to the database provider is performed in an additional provider-specific builder.</span></span> <span data-ttu-id="e3f24-213">例如，在连接到 Azure SQL 时，使用 <xref:Microsoft.EntityFrameworkCore.Infrastructure.SqlServerDbContextOptionsBuilder.EnableRetryOnFailure%2A> 为连接复原配置重试：</span><span class="sxs-lookup"><span data-stu-id="e3f24-213">For example, using <xref:Microsoft.EntityFrameworkCore.Infrastructure.SqlServerDbContextOptionsBuilder.EnableRetryOnFailure%2A> to configure retries for connection resiliency when connecting to Azure SQL:</span></span>

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
> <span data-ttu-id="e3f24-214">同一数据库提供程序用于 SQL Server 和 Azure SQL。</span><span class="sxs-lookup"><span data-stu-id="e3f24-214">The same database provider is used for SQL Server and Azure SQL.</span></span> <span data-ttu-id="e3f24-215">但是，建议在连接到 SQL Azure 时使用[连接复原](xref:core/miscellaneous/connection-resiliency)。</span><span class="sxs-lookup"><span data-stu-id="e3f24-215">However, it is recommended that [connection resiliency](xref:core/miscellaneous/connection-resiliency) be used when connecting to SQL Azure.</span></span>

<span data-ttu-id="e3f24-216">有关特定于提供程序的配置的详细信息，请参阅[数据库提供程序](xref:core/providers/index)。</span><span class="sxs-lookup"><span data-stu-id="e3f24-216">See [Database Providers](xref:core/providers/index) for more information on provider-specific configuration.</span></span>

### <a name="other-dbcontext-configuration"></a><span data-ttu-id="e3f24-217">其他 DbContext 配置</span><span class="sxs-lookup"><span data-stu-id="e3f24-217">Other DbContext configuration</span></span>

<span data-ttu-id="e3f24-218">其他 `DbContext` 配置可以链接到 `Use*` 调用之前或之后（这不会有任何差别）。</span><span class="sxs-lookup"><span data-stu-id="e3f24-218">Other `DbContext` configuration can be chained either before or after (it makes no difference which) the `Use*` call.</span></span> <span data-ttu-id="e3f24-219">例如，若要启用敏感数据日志记录：</span><span class="sxs-lookup"><span data-stu-id="e3f24-219">For example, to turn on sensitive-data logging:</span></span>

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

<span data-ttu-id="e3f24-220">下表包含 `DbContextOptionsBuilder` 调用的常见方法的示例。</span><span class="sxs-lookup"><span data-stu-id="e3f24-220">The following table contains examples of common methods called on `DbContextOptionsBuilder`.</span></span>

| <span data-ttu-id="e3f24-221">DbContextOptionsBuilder 方法</span><span class="sxs-lookup"><span data-stu-id="e3f24-221">DbContextOptionsBuilder method</span></span>                                                             | <span data-ttu-id="e3f24-222">作用</span><span class="sxs-lookup"><span data-stu-id="e3f24-222">What it does</span></span>                                                | <span data-ttu-id="e3f24-223">了解更多</span><span class="sxs-lookup"><span data-stu-id="e3f24-223">Learn more</span></span>
|:-------------------------------------------------------------------------------------------|-------------------------------------------------------------|--------------
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.UseQueryTrackingBehavior%2A>   | <span data-ttu-id="e3f24-224">设置查询的默认跟踪行为</span><span class="sxs-lookup"><span data-stu-id="e3f24-224">Sets the default tracking behavior for queries</span></span>              | [<span data-ttu-id="e3f24-225">查询跟踪行为</span><span class="sxs-lookup"><span data-stu-id="e3f24-225">Query Tracking Behavior</span></span>](xref:core/querying/tracking)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.LogTo%2A>                      | <span data-ttu-id="e3f24-226">获取 EF Core 日志的一种简单方法（EF Core 5.0 及更高版本）</span><span class="sxs-lookup"><span data-stu-id="e3f24-226">A simple way to get EF Core logs (EF Core 5.0 and later)</span></span>    | [<span data-ttu-id="e3f24-227">日志记录、事件和诊断</span><span class="sxs-lookup"><span data-stu-id="e3f24-227">Logging, Events, and Diagnostics</span></span>](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.UseLoggerFactory%2A>           | <span data-ttu-id="e3f24-228">注册 `Micrsofot.Extensions.Logging` 工厂</span><span class="sxs-lookup"><span data-stu-id="e3f24-228">Registers an `Micrsofot.Extensions.Logging` factory</span></span>         | [<span data-ttu-id="e3f24-229">日志记录、事件和诊断</span><span class="sxs-lookup"><span data-stu-id="e3f24-229">Logging, Events, and Diagnostics</span></span>](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableSensitiveDataLogging%2A> | <span data-ttu-id="e3f24-230">在异常和日志记录中包括应用程序数据</span><span class="sxs-lookup"><span data-stu-id="e3f24-230">Includes application data in exceptions and logging</span></span>         | [<span data-ttu-id="e3f24-231">日志记录、事件和诊断</span><span class="sxs-lookup"><span data-stu-id="e3f24-231">Logging, Events, and Diagnostics</span></span>](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.EnableDetailedErrors%2A>       | <span data-ttu-id="e3f24-232">更详细的查询错误（以性能为代价）</span><span class="sxs-lookup"><span data-stu-id="e3f24-232">More detailed query errors (at the expense of performance)</span></span>  | [<span data-ttu-id="e3f24-233">日志记录、事件和诊断</span><span class="sxs-lookup"><span data-stu-id="e3f24-233">Logging, Events, and Diagnostics</span></span>](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.ConfigureWarnings%2A>          | <span data-ttu-id="e3f24-234">忽略或引发警告和其他事件</span><span class="sxs-lookup"><span data-stu-id="e3f24-234">Ignore or throw for warnings and other events</span></span>               | [<span data-ttu-id="e3f24-235">日志记录、事件和诊断</span><span class="sxs-lookup"><span data-stu-id="e3f24-235">Logging, Events, and Diagnostics</span></span>](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.AddInterceptors%2A>            | <span data-ttu-id="e3f24-236">注册 EF Core 侦听器</span><span class="sxs-lookup"><span data-stu-id="e3f24-236">Registers EF Core interceptors</span></span>                              | [<span data-ttu-id="e3f24-237">日志记录、事件和诊断</span><span class="sxs-lookup"><span data-stu-id="e3f24-237">Logging, Events, and Diagnostics</span></span>](xref:core/logging-events-diagnostics/index)
| <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseLazyLoadingProxies%2A>            | <span data-ttu-id="e3f24-238">使用动态代理进行延迟加载</span><span class="sxs-lookup"><span data-stu-id="e3f24-238">Use dynamic proxies for lazy-loading</span></span>                        | [<span data-ttu-id="e3f24-239">延迟加载</span><span class="sxs-lookup"><span data-stu-id="e3f24-239">Lazy Loading</span></span>](xref:core/querying/related-data/lazy)
| <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseChangeTrackingProxies%2A>         | <span data-ttu-id="e3f24-240">使用动态代理进行更改跟踪</span><span class="sxs-lookup"><span data-stu-id="e3f24-240">Use dynamic proxies for change-tracking</span></span>                     | <span data-ttu-id="e3f24-241">即将推出...</span><span class="sxs-lookup"><span data-stu-id="e3f24-241">Coming soon...</span></span>

> [!NOTE]
> <span data-ttu-id="e3f24-242"><xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseLazyLoadingProxies%2A> 和 <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseChangeTrackingProxies%2A> 是 [Microsoft.EntityFrameworkCore.Proxies](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Proxies/) NuGet 包中的扩展方法。</span><span class="sxs-lookup"><span data-stu-id="e3f24-242"><xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseLazyLoadingProxies%2A> and <xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseChangeTrackingProxies%2A> are extension methods from the [Microsoft.EntityFrameworkCore.Proxies](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Proxies/) NuGet package.</span></span> <span data-ttu-id="e3f24-243">建议的方式是使用这种类型的“.UseSomething()”调用来配置和/或使用其他包中包含的 EF Core 扩展。</span><span class="sxs-lookup"><span data-stu-id="e3f24-243">This kind of ".UseSomething()" call is the recommended way to configure and/or use EF Core extensions contained in other packages.</span></span>

### <a name="dbcontextoptions-verses-dbcontextoptionstcontext"></a><span data-ttu-id="e3f24-244">`DbContextOptions` 与 `DbContextOptions<TContext>`</span><span class="sxs-lookup"><span data-stu-id="e3f24-244">`DbContextOptions` verses `DbContextOptions<TContext>`</span></span>

<span data-ttu-id="e3f24-245">大多数接受 `DbContextOptions` 的 `DbContext` 子类应使用 [泛型](/dotnet/csharp/programming-guide/generics/) `DbContextOptions<TContext>` 变体。</span><span class="sxs-lookup"><span data-stu-id="e3f24-245">Most `DbContext` subclasses that accept a `DbContextOptions` should use the [generic](/dotnet/csharp/programming-guide/generics/) `DbContextOptions<TContext>` variation.</span></span> <span data-ttu-id="e3f24-246">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-246">For example:</span></span>

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

<span data-ttu-id="e3f24-247">这可确保从依赖关系注入中解析特定 `DbContext` 子类型的正确选项，即使注册了多个 `DbContext` 子类型也是如此。</span><span class="sxs-lookup"><span data-stu-id="e3f24-247">This ensures that the correct options for the specific `DbContext` subtype are resolved from dependency injection, even when multiple `DbContext` subtypes are registered.</span></span>

> [!TIP]
> <span data-ttu-id="e3f24-248">你的 DbContext 不需要密封，但对于没有被设计为继承的类，密封是最佳做法。</span><span class="sxs-lookup"><span data-stu-id="e3f24-248">Your DbContext does not need to be sealed, but sealing is best practice to do so for classes not designed to be inherited from.</span></span>

<span data-ttu-id="e3f24-249">但是，如果 `DbContext` 子类型本身旨在继承，则它应公开采用非泛型 `DbContextOptions` 的受保护构造函数。</span><span class="sxs-lookup"><span data-stu-id="e3f24-249">However, if the `DbContext` subtype is itself intended to be inherited from, then it should expose a protected constructor taking a non-generic `DbContextOptions`.</span></span> <span data-ttu-id="e3f24-250">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-250">For example:</span></span>

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

<span data-ttu-id="e3f24-251">这允许多个具体子类使用其不同的泛型 `DbContextOptions<TContext>` 实例来调用此基构造函数。</span><span class="sxs-lookup"><span data-stu-id="e3f24-251">This allows multiple concrete subclasses to call this base constructor using their different generic `DbContextOptions<TContext>` instances.</span></span> <span data-ttu-id="e3f24-252">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-252">For example:</span></span>

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

<span data-ttu-id="e3f24-253">请注意，这与直接从 `DbContext` 继承的模式完全相同。</span><span class="sxs-lookup"><span data-stu-id="e3f24-253">Notice that this is exactly the same pattern as when inheriting from `DbContext` directly.</span></span> <span data-ttu-id="e3f24-254">也就是说，出于此原因，`DbContext` 构造函数本身将接受非泛型 `DbContextOptions`。</span><span class="sxs-lookup"><span data-stu-id="e3f24-254">That is, the `DbContext` constructor itself accepts a non-generic `DbContextOptions` for this reason.</span></span>

<span data-ttu-id="e3f24-255">旨在同时进行实例化和继承的 `DbContext` 子类应公开构造函数的两种形式。</span><span class="sxs-lookup"><span data-stu-id="e3f24-255">A `DbContext` subclass intended to be both instantiated and inherited from should expose both forms of constructor.</span></span> <span data-ttu-id="e3f24-256">例如：</span><span class="sxs-lookup"><span data-stu-id="e3f24-256">For example:</span></span>

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

## <a name="design-time-dbcontext-configuration"></a><span data-ttu-id="e3f24-257">设计时 DbContext 配置</span><span class="sxs-lookup"><span data-stu-id="e3f24-257">Design-time DbContext configuration</span></span>

<span data-ttu-id="e3f24-258">EF Core 设计时工具（如 [EF Core迁移](xref:core/managing-schemas/migrations/index)）需要能够发现并创建 `DbContext` 类型的工作实例，以收集有关应用程序的实体类型及其如何映射到数据库架构的详细信息。</span><span class="sxs-lookup"><span data-stu-id="e3f24-258">EF Core design-time tools such as those for [EF Core migrations](xref:core/managing-schemas/migrations/index) need to be able to discover and create a working instance of a `DbContext` type in order to gather details about the application's entity types and how they map to a database schema.</span></span> <span data-ttu-id="e3f24-259">只要该工具可以通过与在运行时的配置方式类似的方式轻松地创建 `DbContext`，就可以自动执行此过程。</span><span class="sxs-lookup"><span data-stu-id="e3f24-259">This process can be automatic as long as the tool can easily create the `DbContext` in such a way that it will be configured similarly to how it would be configured at run-time.</span></span>

<span data-ttu-id="e3f24-260">虽然向 `DbContext` 提供必要配置信息的任何模式都可以在运行时正常运行，但在设计时需要使用 `DbContext` 的工具只能使用有限数量的模式。</span><span class="sxs-lookup"><span data-stu-id="e3f24-260">While any pattern that provides the necessary configuration information to the `DbContext` can work at run-time, tools that require using a `DbContext` at design-time can only work with a limited number of patterns.</span></span> <span data-ttu-id="e3f24-261">[设计时上下文创建](xref:core/cli/dbcontext-creation)中包含更多详细信息。</span><span class="sxs-lookup"><span data-stu-id="e3f24-261">These are covered in more detail in [Design-Time Context Creation](xref:core/cli/dbcontext-creation).</span></span>

## <a name="avoiding-dbcontext-threading-issues"></a><span data-ttu-id="e3f24-262">避免 DbContext 线程处理问题</span><span class="sxs-lookup"><span data-stu-id="e3f24-262">Avoiding DbContext threading issues</span></span>

<span data-ttu-id="e3f24-263">Entity Framework Core 不支持在同一 `DbContext` 实例上运行多个并行操作。</span><span class="sxs-lookup"><span data-stu-id="e3f24-263">Entity Framework Core does not support multiple parallel operations being run on the same `DbContext` instance.</span></span> <span data-ttu-id="e3f24-264">这包括异步查询的并行执行以及从多个线程进行的任何显式并发使用。</span><span class="sxs-lookup"><span data-stu-id="e3f24-264">This includes both parallel execution of async queries and any explicit concurrent use from multiple threads.</span></span> <span data-ttu-id="e3f24-265">因此，始终立即 `await` 异步调用，或对并行执行的操作使用单独的 `DbContext` 实例。</span><span class="sxs-lookup"><span data-stu-id="e3f24-265">Therefore, always `await` async calls immediately, or use separate `DbContext` instances for operations that execute in parallel.</span></span>

<span data-ttu-id="e3f24-266">当 EF Core 检测到尝试同时使用 `DbContext` 实例的情况时，你将看到 `InvalidOperationException`，其中包含类似于以下内容的消息：</span><span class="sxs-lookup"><span data-stu-id="e3f24-266">When EF Core detects an attempt to use a `DbContext` instance concurrently, you'll see an `InvalidOperationException` with a message like this:</span></span>

> <span data-ttu-id="e3f24-267">在上一个操作完成之前，第二个操作已在此上下文中启动。</span><span class="sxs-lookup"><span data-stu-id="e3f24-267">A second operation started on this context before a previous operation completed.</span></span> <span data-ttu-id="e3f24-268">这通常是由使用同一个 DbContext 实例的不同线程引起的，但不保证实例成员是线程安全的。</span><span class="sxs-lookup"><span data-stu-id="e3f24-268">This is usually caused by different threads using the same instance of DbContext, however instance members are not guaranteed to be thread safe.</span></span>

<span data-ttu-id="e3f24-269">检测不到并发访问时，可能会导致未定义的行为、应用程序崩溃和数据损坏。</span><span class="sxs-lookup"><span data-stu-id="e3f24-269">When concurrent access goes undetected, it can result in undefined behavior, application crashes and data corruption.</span></span>

<span data-ttu-id="e3f24-270">一些常见错误可能会无意中导致并发访问同一 `DbContext` 实例：</span><span class="sxs-lookup"><span data-stu-id="e3f24-270">There are common mistakes that can inadvertently cause concurrent access on the same `DbContext` instance:</span></span>

### <a name="asynchronous-operation-pitfalls"></a><span data-ttu-id="e3f24-271">异步操作缺陷</span><span class="sxs-lookup"><span data-stu-id="e3f24-271">Asynchronous operation pitfalls</span></span>

<span data-ttu-id="e3f24-272">使用异步方法，EF Core 可以启动以非阻挡式访问数据库的操作。</span><span class="sxs-lookup"><span data-stu-id="e3f24-272">Asynchronous methods enable EF Core to initiate operations that access the database in a non-blocking way.</span></span> <span data-ttu-id="e3f24-273">但是，如果调用方不等待其中一个方法完成，而是继续对 `DbContext` 执行其他操作，则 `DbContext` 的状态可能会（并且很可能会）损坏。</span><span class="sxs-lookup"><span data-stu-id="e3f24-273">But if a caller does not await the completion of one of these methods, and proceeds to perform other operations on the `DbContext`, the state of the `DbContext` can be, (and very likely will be) corrupted.</span></span>

<span data-ttu-id="e3f24-274">始终立即等待 EF Core 异步方法。</span><span class="sxs-lookup"><span data-stu-id="e3f24-274">Always await EF Core asynchronous methods immediately.</span></span>

### <a name="implicitly-sharing-dbcontext-instances-via-dependency-injection"></a><span data-ttu-id="e3f24-275">通过依赖关系注入隐式共享 DbContext 实例</span><span class="sxs-lookup"><span data-stu-id="e3f24-275">Implicitly sharing DbContext instances via dependency injection</span></span>

<span data-ttu-id="e3f24-276">默认情况下 [`AddDbContext`](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) 扩展方法使用[范围内生存期](/aspnet/core/fundamentals/dependency-injection#service-lifetimes)来注册 `DbContext` 类型。</span><span class="sxs-lookup"><span data-stu-id="e3f24-276">The [`AddDbContext`](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) extension method registers `DbContext` types with a [scoped lifetime](/aspnet/core/fundamentals/dependency-injection#service-lifetimes) by default.</span></span>

<span data-ttu-id="e3f24-277">这样可以避免在大多数 ASP.NET Core 应用程序中出现并发访问问题，因为在给定时间内只有一个线程在执行每个客户端请求，并且每个请求都有单独的依赖关系注入范围（因此有单独的 `DbContext` 实例）。</span><span class="sxs-lookup"><span data-stu-id="e3f24-277">This is safe from concurrent access issues in most ASP.NET Core applications because there is only one thread executing each client request at a given time, and because each request gets a separate dependency injection scope (and therefore a separate `DbContext` instance).</span></span> <span data-ttu-id="e3f24-278">对于 Blazor Server 托管模型，一个逻辑请求用来维护 Blazor 用户线路，因此，如果使用默认注入范围，则每个用户线路只能提供一个范围内的 DbContext 实例。</span><span class="sxs-lookup"><span data-stu-id="e3f24-278">For Blazor Server hosting model, one logical request is used for maintaining the Blazor user circuit, and thus only one scoped DbContext instance is available per user circuit if the default injection scope is used.</span></span>

<span data-ttu-id="e3f24-279">任何并行显式执行多个线程的代码都应确保 `DbContext` 实例不会同时访问。</span><span class="sxs-lookup"><span data-stu-id="e3f24-279">Any code that explicitly executes multiple threads in parallel should ensure that `DbContext` instances aren't ever accessed concurrently.</span></span>

<span data-ttu-id="e3f24-280">使用依赖关系注入可以通过以下方式实现：将上下文注册为范围内，并为每个线程创建范围（使用 `IServiceScopeFactory`），或将 `DbContext` 注册为暂时性（使用采用 `ServiceLifetime` 参数的 `AddDbContext` 的重载）。</span><span class="sxs-lookup"><span data-stu-id="e3f24-280">Using dependency injection, this can be achieved by either registering the context as scoped, and creating scopes (using `IServiceScopeFactory`) for each thread, or by registering the `DbContext` as transient (using the overload of `AddDbContext` which takes a `ServiceLifetime` parameter).</span></span>

## <a name="more-reading"></a><span data-ttu-id="e3f24-281">进一步阅读</span><span class="sxs-lookup"><span data-stu-id="e3f24-281">More reading</span></span>

- <span data-ttu-id="e3f24-282">请阅读[依赖关系注入](/aspnet/core/fundamentals/dependency-injection)以了解有关使用 DI 的详细信息。</span><span class="sxs-lookup"><span data-stu-id="e3f24-282">Read [Dependency Injection](/aspnet/core/fundamentals/dependency-injection) to learn more about using DI.</span></span>
- <span data-ttu-id="e3f24-283">请阅读[测试](xref:core/testing/index)以了解详细信息。</span><span class="sxs-lookup"><span data-stu-id="e3f24-283">Read [Testing](xref:core/testing/index) for more information.</span></span>
