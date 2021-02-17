---
title: 基于代码的配置-EF6
description: 实体框架6中基于代码的配置
author: ajcvickers
ms.date: 10/23/2016
uid: ef6/fundamentals/configuring/code-based
ms.openlocfilehash: b16cbcef85708730dcc6b82a38635cc60cb2206a
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543167"
---
# <a name="code-based-configuration"></a><span data-ttu-id="17c74-103">基于代码的配置</span><span class="sxs-lookup"><span data-stu-id="17c74-103">Code-based configuration</span></span>
> [!NOTE]
> <span data-ttu-id="17c74-104">**仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。</span><span class="sxs-lookup"><span data-stu-id="17c74-104">**EF6 Onwards Only** - The features, APIs, etc. discussed in this page were introduced in Entity Framework 6.</span></span> <span data-ttu-id="17c74-105">如果使用的是早期版本，则部分或全部信息不适用。</span><span class="sxs-lookup"><span data-stu-id="17c74-105">If you are using an earlier version, some or all of the information does not apply.</span></span>  

<span data-ttu-id="17c74-106">可以在配置文件中指定实体框架应用程序的配置 (app.config/web.config) 或通过代码。</span><span class="sxs-lookup"><span data-stu-id="17c74-106">Configuration for an Entity Framework application can be specified in a config file (app.config/web.config) or through code.</span></span> <span data-ttu-id="17c74-107">后者称为基于代码的配置。</span><span class="sxs-lookup"><span data-stu-id="17c74-107">The latter is known as code-based configuration.</span></span>  

<span data-ttu-id="17c74-108">在 [单独的文章](xref:ef6/fundamentals/configuring/config-file)中介绍了配置文件中的配置。</span><span class="sxs-lookup"><span data-stu-id="17c74-108">Configuration in a config file is described in a [separate article](xref:ef6/fundamentals/configuring/config-file).</span></span> <span data-ttu-id="17c74-109">配置文件优先于基于代码的配置。</span><span class="sxs-lookup"><span data-stu-id="17c74-109">The config file takes precedence over code-based configuration.</span></span> <span data-ttu-id="17c74-110">换句话说，如果在代码和配置文件中都设置了配置选项，则使用配置文件中的设置。</span><span class="sxs-lookup"><span data-stu-id="17c74-110">In other words, if a configuration option is set in both code and in the config file, then the setting in the config file is used.</span></span>  

## <a name="using-dbconfiguration"></a><span data-ttu-id="17c74-111">使用 `DbConfiguration`</span><span class="sxs-lookup"><span data-stu-id="17c74-111">Using `DbConfiguration`</span></span>

<span data-ttu-id="17c74-112">EF6 和更高版本中基于代码的配置是通过创建的子类实现的 `System.Data.Entity.Config.DbConfiguration` 。</span><span class="sxs-lookup"><span data-stu-id="17c74-112">Code-based configuration in EF6 and above is achieved by creating a subclass of `System.Data.Entity.Config.DbConfiguration`.</span></span> <span data-ttu-id="17c74-113">以下准则应遵循以下准则 `DbConfiguration` ：</span><span class="sxs-lookup"><span data-stu-id="17c74-113">The following guidelines should be followed when subclassing `DbConfiguration`:</span></span>  

- <span data-ttu-id="17c74-114">只 `DbConfiguration` 为应用程序创建一个类。</span><span class="sxs-lookup"><span data-stu-id="17c74-114">Create only one `DbConfiguration` class for your application.</span></span> <span data-ttu-id="17c74-115">此类指定应用域范围内的设置。</span><span class="sxs-lookup"><span data-stu-id="17c74-115">This class specifies app-domain wide settings.</span></span>  
- <span data-ttu-id="17c74-116">将 `DbConfiguration` 类放在与类相同的程序集中 `DbContext` 。</span><span class="sxs-lookup"><span data-stu-id="17c74-116">Place your `DbConfiguration` class in the same assembly as your `DbContext` class.</span></span> <span data-ttu-id="17c74-117"> (如果要更改此内容，请参阅 *移动 `DbConfiguration`* 部分。 ) </span><span class="sxs-lookup"><span data-stu-id="17c74-117">(See the *Moving `DbConfiguration`* section if you want to change this.)</span></span>  
- <span data-ttu-id="17c74-118">为您 `DbConfiguration` 的类指定一个公共的无参数构造函数。</span><span class="sxs-lookup"><span data-stu-id="17c74-118">Give your `DbConfiguration` class a public parameterless constructor.</span></span>  
- <span data-ttu-id="17c74-119">通过 `DbConfiguration` 从该构造函数内调用受保护的方法来设置配置选项。</span><span class="sxs-lookup"><span data-stu-id="17c74-119">Set configuration options by calling protected `DbConfiguration` methods from within this constructor.</span></span>  

<span data-ttu-id="17c74-120">遵循这些指导原则后，EF 可以通过需要访问模型和应用程序运行时的两个工具自动发现和使用配置。</span><span class="sxs-lookup"><span data-stu-id="17c74-120">Following these guidelines allows EF to discover and use your configuration automatically by both tooling that needs to access your model and when your application is run.</span></span>  

## <a name="example"></a><span data-ttu-id="17c74-121">示例</span><span class="sxs-lookup"><span data-stu-id="17c74-121">Example</span></span>  

<span data-ttu-id="17c74-122">派生自的类 `DbConfiguration` 可能如下所示：</span><span class="sxs-lookup"><span data-stu-id="17c74-122">A class derived from `DbConfiguration` might look like this:</span></span>  

``` csharp
using System.Data.Entity;
using System.Data.Entity.Infrastructure;
using System.Data.Entity.SqlServer;

namespace MyNamespace
{
    public class MyConfiguration : DbConfiguration
    {
        public MyConfiguration()
        {
            SetExecutionStrategy("System.Data.SqlClient", () => new SqlAzureExecutionStrategy());
            SetDefaultConnectionFactory(new LocalDbConnectionFactory("mssqllocaldb"));
        }
    }
}
```  

<span data-ttu-id="17c74-123">此类设置 EF 以使用 SQL Azure 执行策略-自动重试失败的数据库操作，并使用由 Code First 中的约定创建的数据库的本地数据库。</span><span class="sxs-lookup"><span data-stu-id="17c74-123">This class sets up EF to use the SQL Azure execution strategy - to automatically retry failed database operations - and to use Local DB for databases that are created by convention from Code First.</span></span>  

## <a name="moving-dbconfiguration"></a><span data-ttu-id="17c74-124">动 `DbConfiguration`</span><span class="sxs-lookup"><span data-stu-id="17c74-124">Moving `DbConfiguration`</span></span>  

<span data-ttu-id="17c74-125">在某些情况下，不能将 `DbConfiguration` 类放在与类相同的程序集中 `DbContext` 。</span><span class="sxs-lookup"><span data-stu-id="17c74-125">There are cases where it is not possible to place your `DbConfiguration` class in the same assembly as your `DbContext` class.</span></span> <span data-ttu-id="17c74-126">例如，你可能 `DbContext` 在不同的程序集中有两个类。</span><span class="sxs-lookup"><span data-stu-id="17c74-126">For example, you may have two `DbContext` classes each in different assemblies.</span></span> <span data-ttu-id="17c74-127">有两个选项可用于处理此操作。</span><span class="sxs-lookup"><span data-stu-id="17c74-127">There are two options for handling this.</span></span>  

<span data-ttu-id="17c74-128">第一种方法是使用配置文件指定 `DbConfiguration` 要使用的实例。</span><span class="sxs-lookup"><span data-stu-id="17c74-128">The first option is to use the config file to specify the `DbConfiguration` instance to use.</span></span> <span data-ttu-id="17c74-129">为此，请设置 entityFramework 部分的 codeConfigurationType 属性。</span><span class="sxs-lookup"><span data-stu-id="17c74-129">To do this, set the codeConfigurationType attribute of the entityFramework section.</span></span> <span data-ttu-id="17c74-130">例如：</span><span class="sxs-lookup"><span data-stu-id="17c74-130">For example:</span></span>  

``` xml
<entityFramework codeConfigurationType="MyNamespace.MyDbConfiguration, MyAssembly">
    ...Your EF config...
</entityFramework>
```  

<span data-ttu-id="17c74-131">CodeConfigurationType 的值必须是类的程序集和命名空间限定名称 `DbConfiguration` 。</span><span class="sxs-lookup"><span data-stu-id="17c74-131">The value of codeConfigurationType must be the assembly and namespace qualified name of your `DbConfiguration` class.</span></span>  

<span data-ttu-id="17c74-132">第二个选项是放置 `DbConfigurationTypeAttribute` 在上下文类中。</span><span class="sxs-lookup"><span data-stu-id="17c74-132">The second option is to place `DbConfigurationTypeAttribute` on your context class.</span></span> <span data-ttu-id="17c74-133">例如：</span><span class="sxs-lookup"><span data-stu-id="17c74-133">For example:</span></span>  

``` csharp  
[DbConfigurationType(typeof(MyDbConfiguration))]
public class MyContextContext : DbContext
{
}
```  

<span data-ttu-id="17c74-134">传递给特性的值可以是您的 `DbConfiguration` 类型（如上所示），也可以是程序集和命名空间限定的类型名称字符串。</span><span class="sxs-lookup"><span data-stu-id="17c74-134">The value passed to the attribute can either be your `DbConfiguration` type - as shown above - or the assembly and namespace qualified type name string.</span></span> <span data-ttu-id="17c74-135">例如：</span><span class="sxs-lookup"><span data-stu-id="17c74-135">For example:</span></span>  

``` csharp
[DbConfigurationType("MyNamespace.MyDbConfiguration, MyAssembly")]
public class MyContextContext : DbContext
{
}
```  

## <a name="setting-dbconfiguration-explicitly"></a><span data-ttu-id="17c74-136">`DbConfiguration`显式设置</span><span class="sxs-lookup"><span data-stu-id="17c74-136">Setting `DbConfiguration` explicitly</span></span>  

<span data-ttu-id="17c74-137">在某些情况下，在使用任何类型之前可能需要进行配置 `DbContext` 。</span><span class="sxs-lookup"><span data-stu-id="17c74-137">There are some situations where configuration may be needed before any `DbContext` type has been used.</span></span> <span data-ttu-id="17c74-138">这种情况的示例包括：</span><span class="sxs-lookup"><span data-stu-id="17c74-138">Examples of this include:</span></span>  

- <span data-ttu-id="17c74-139">`DbModelBuilder`用于生成没有上下文的模型</span><span class="sxs-lookup"><span data-stu-id="17c74-139">Using `DbModelBuilder` to build a model without a context</span></span>  
- <span data-ttu-id="17c74-140">在使用应用程序上下文之前，使用一些其他框架/实用工具代码，该代码利用在 `DbContext` 何处使用上下文</span><span class="sxs-lookup"><span data-stu-id="17c74-140">Using some other framework/utility code that utilizes a `DbContext` where that context is used before your application context is used</span></span>  

<span data-ttu-id="17c74-141">在这种情况下，EF 无法自动发现配置，而必须执行以下操作之一：</span><span class="sxs-lookup"><span data-stu-id="17c74-141">In such situations EF is unable to discover the configuration automatically and you must instead do one of the following:</span></span>  

- <span data-ttu-id="17c74-142">设置 `DbConfiguration` 配置文件中的类型，如上面的 *移动 `DbConfiguration`* 部分所述</span><span class="sxs-lookup"><span data-stu-id="17c74-142">Set the `DbConfiguration` type in the config file, as described in the *Moving `DbConfiguration`* section above</span></span>
- <span data-ttu-id="17c74-143">调用静态 `DbConfiguration` 。在应用程序启动过程中的 SetConfiguration 方法</span><span class="sxs-lookup"><span data-stu-id="17c74-143">Call the static `DbConfiguration`.SetConfiguration method during application startup</span></span>  

## <a name="overriding-dbconfiguration"></a><span data-ttu-id="17c74-144">取代 `DbConfiguration`</span><span class="sxs-lookup"><span data-stu-id="17c74-144">Overriding `DbConfiguration`</span></span>  

<span data-ttu-id="17c74-145">在某些情况下，你需要在中替代配置集 `DbConfiguration` 。</span><span class="sxs-lookup"><span data-stu-id="17c74-145">There are some situations where you need to override the configuration set in the `DbConfiguration`.</span></span> <span data-ttu-id="17c74-146">这通常不是由应用程序开发人员完成，而是由不能使用派生类的第三方提供程序和插件来完成 `DbConfiguration` 。</span><span class="sxs-lookup"><span data-stu-id="17c74-146">This is not typically done by application developers but rather by third party providers and plug-ins that cannot use a derived `DbConfiguration` class.</span></span>  

<span data-ttu-id="17c74-147">为此，EntityFramework 允许注册事件处理程序，该事件处理程序可以在锁定之前修改现有配置。</span><span class="sxs-lookup"><span data-stu-id="17c74-147">For this, EntityFramework allows an event handler to be registered that can modify existing configuration just before it is locked down.</span></span>  <span data-ttu-id="17c74-148">它还提供专门用于替换 EF 服务定位器返回的任何服务的一个糖方法。</span><span class="sxs-lookup"><span data-stu-id="17c74-148">It also provides a sugar method specifically for replacing any service returned by the EF service locator.</span></span> <span data-ttu-id="17c74-149">这就是如何使用它：</span><span class="sxs-lookup"><span data-stu-id="17c74-149">This is how it is intended to be used:</span></span>  

- <span data-ttu-id="17c74-150">在应用启动 (在使用 EF 之前) 插件或提供程序应为此事件注册事件处理程序方法。</span><span class="sxs-lookup"><span data-stu-id="17c74-150">At app startup (before EF is used) the plug-in or provider should register the event handler method for this event.</span></span> <span data-ttu-id="17c74-151"> (注意，此操作必须在应用程序使用 EF 之前发生。 ) </span><span class="sxs-lookup"><span data-stu-id="17c74-151">(Note that this must happen before the application uses EF.)</span></span>  
- <span data-ttu-id="17c74-152">对于需要替换的每个服务，事件处理程序都会调用 ReplaceService。</span><span class="sxs-lookup"><span data-stu-id="17c74-152">The event handler makes a call to ReplaceService for every service that needs to be replaced.</span></span>  

<span data-ttu-id="17c74-153">例如，要替换， `IDbConnectionFactory` `DbProviderService` 你需要注册类似于下面的处理程序：</span><span class="sxs-lookup"><span data-stu-id="17c74-153">For example, to replace `IDbConnectionFactory` and `DbProviderService` you would register a handler something like this:</span></span>  

``` csharp
DbConfiguration.Loaded += (_, a) =>
   {
       a.ReplaceService<DbProviderServices>((s, k) => new MyProviderServices(s));
       a.ReplaceService<IDbConnectionFactory>((s, k) => new MyConnectionFactory(s));
   };
```  

<span data-ttu-id="17c74-154">在上面的代码中 `MyProviderServices` ， `MyConnectionFactory` 表示服务的实现。</span><span class="sxs-lookup"><span data-stu-id="17c74-154">In the code above, `MyProviderServices` and `MyConnectionFactory` represent your implementations of the service.</span></span>  

<span data-ttu-id="17c74-155">你还可以添加其他依赖关系处理程序以获得相同的效果。</span><span class="sxs-lookup"><span data-stu-id="17c74-155">You can also add additional dependency handlers to get the same effect.</span></span>  

<span data-ttu-id="17c74-156">请注意，你也可以 `DbProviderFactory` 通过这种方式进行换行，但这样做只会影响 ef，而不会在 `DbProviderFactory` ef 之外使用。</span><span class="sxs-lookup"><span data-stu-id="17c74-156">Note that you could also wrap `DbProviderFactory` in this way, but doing so will only affect EF and not uses of the `DbProviderFactory` outside of EF.</span></span> <span data-ttu-id="17c74-157">出于此原因，你可能会希望继续 `DbProviderFactory` 像以前一样进行换行。</span><span class="sxs-lookup"><span data-stu-id="17c74-157">For this reason you’ll probably want to continue to wrap `DbProviderFactory` as you have before.</span></span>  

<span data-ttu-id="17c74-158">你还应记住你在应用程序外部运行的服务（例如，从包管理器控制台运行迁移时）。</span><span class="sxs-lookup"><span data-stu-id="17c74-158">You should also keep in mind the services that you run externally to your application - for example, when running migrations from the Package Manager Console.</span></span> <span data-ttu-id="17c74-159">当你从控制台运行迁移时，它将尝试查找你 `DbConfiguration` 的。</span><span class="sxs-lookup"><span data-stu-id="17c74-159">When you run migrate from the console it will attempt to find your `DbConfiguration`.</span></span> <span data-ttu-id="17c74-160">但是，无论是否获取包装服务，都取决于它所注册的事件处理程序的位置。</span><span class="sxs-lookup"><span data-stu-id="17c74-160">However, whether or not it will get the wrapped service depends on where the event handler it registered.</span></span> <span data-ttu-id="17c74-161">如果已将其注册为构造的一部分，则 `DbConfiguration` 应执行代码并且应包装服务。</span><span class="sxs-lookup"><span data-stu-id="17c74-161">If it is registered as part of the construction of your `DbConfiguration` then the code should execute and the service should get wrapped.</span></span> <span data-ttu-id="17c74-162">通常不会出现这种情况，这意味着工具不会获得包装服务。</span><span class="sxs-lookup"><span data-stu-id="17c74-162">Usually this won’t be the case and this means that tooling won’t get the wrapped service.</span></span>  
