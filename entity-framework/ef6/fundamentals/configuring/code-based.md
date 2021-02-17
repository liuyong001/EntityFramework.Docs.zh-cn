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
# <a name="code-based-configuration"></a>基于代码的配置
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。  

可以在配置文件中指定实体框架应用程序的配置 (app.config/web.config) 或通过代码。 后者称为基于代码的配置。  

在 [单独的文章](xref:ef6/fundamentals/configuring/config-file)中介绍了配置文件中的配置。 配置文件优先于基于代码的配置。 换句话说，如果在代码和配置文件中都设置了配置选项，则使用配置文件中的设置。  

## <a name="using-dbconfiguration"></a>使用 `DbConfiguration`

EF6 和更高版本中基于代码的配置是通过创建的子类实现的 `System.Data.Entity.Config.DbConfiguration` 。 以下准则应遵循以下准则 `DbConfiguration` ：  

- 只 `DbConfiguration` 为应用程序创建一个类。 此类指定应用域范围内的设置。  
- 将 `DbConfiguration` 类放在与类相同的程序集中 `DbContext` 。  (如果要更改此内容，请参阅 *移动 `DbConfiguration`* 部分。 )   
- 为您 `DbConfiguration` 的类指定一个公共的无参数构造函数。  
- 通过 `DbConfiguration` 从该构造函数内调用受保护的方法来设置配置选项。  

遵循这些指导原则后，EF 可以通过需要访问模型和应用程序运行时的两个工具自动发现和使用配置。  

## <a name="example"></a>示例  

派生自的类 `DbConfiguration` 可能如下所示：  

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

此类设置 EF 以使用 SQL Azure 执行策略-自动重试失败的数据库操作，并使用由 Code First 中的约定创建的数据库的本地数据库。  

## <a name="moving-dbconfiguration"></a>动 `DbConfiguration`  

在某些情况下，不能将 `DbConfiguration` 类放在与类相同的程序集中 `DbContext` 。 例如，你可能 `DbContext` 在不同的程序集中有两个类。 有两个选项可用于处理此操作。  

第一种方法是使用配置文件指定 `DbConfiguration` 要使用的实例。 为此，请设置 entityFramework 部分的 codeConfigurationType 属性。 例如：  

``` xml
<entityFramework codeConfigurationType="MyNamespace.MyDbConfiguration, MyAssembly">
    ...Your EF config...
</entityFramework>
```  

CodeConfigurationType 的值必须是类的程序集和命名空间限定名称 `DbConfiguration` 。  

第二个选项是放置 `DbConfigurationTypeAttribute` 在上下文类中。 例如：  

``` csharp  
[DbConfigurationType(typeof(MyDbConfiguration))]
public class MyContextContext : DbContext
{
}
```  

传递给特性的值可以是您的 `DbConfiguration` 类型（如上所示），也可以是程序集和命名空间限定的类型名称字符串。 例如：  

``` csharp
[DbConfigurationType("MyNamespace.MyDbConfiguration, MyAssembly")]
public class MyContextContext : DbContext
{
}
```  

## <a name="setting-dbconfiguration-explicitly"></a>`DbConfiguration`显式设置  

在某些情况下，在使用任何类型之前可能需要进行配置 `DbContext` 。 这种情况的示例包括：  

- `DbModelBuilder`用于生成没有上下文的模型  
- 在使用应用程序上下文之前，使用一些其他框架/实用工具代码，该代码利用在 `DbContext` 何处使用上下文  

在这种情况下，EF 无法自动发现配置，而必须执行以下操作之一：  

- 设置 `DbConfiguration` 配置文件中的类型，如上面的 *移动 `DbConfiguration`* 部分所述
- 调用静态 `DbConfiguration` 。在应用程序启动过程中的 SetConfiguration 方法  

## <a name="overriding-dbconfiguration"></a>取代 `DbConfiguration`  

在某些情况下，你需要在中替代配置集 `DbConfiguration` 。 这通常不是由应用程序开发人员完成，而是由不能使用派生类的第三方提供程序和插件来完成 `DbConfiguration` 。  

为此，EntityFramework 允许注册事件处理程序，该事件处理程序可以在锁定之前修改现有配置。  它还提供专门用于替换 EF 服务定位器返回的任何服务的一个糖方法。 这就是如何使用它：  

- 在应用启动 (在使用 EF 之前) 插件或提供程序应为此事件注册事件处理程序方法。  (注意，此操作必须在应用程序使用 EF 之前发生。 )   
- 对于需要替换的每个服务，事件处理程序都会调用 ReplaceService。  

例如，要替换， `IDbConnectionFactory` `DbProviderService` 你需要注册类似于下面的处理程序：  

``` csharp
DbConfiguration.Loaded += (_, a) =>
   {
       a.ReplaceService<DbProviderServices>((s, k) => new MyProviderServices(s));
       a.ReplaceService<IDbConnectionFactory>((s, k) => new MyConnectionFactory(s));
   };
```  

在上面的代码中 `MyProviderServices` ， `MyConnectionFactory` 表示服务的实现。  

你还可以添加其他依赖关系处理程序以获得相同的效果。  

请注意，你也可以 `DbProviderFactory` 通过这种方式进行换行，但这样做只会影响 ef，而不会在 `DbProviderFactory` ef 之外使用。 出于此原因，你可能会希望继续 `DbProviderFactory` 像以前一样进行换行。  

你还应记住你在应用程序外部运行的服务（例如，从包管理器控制台运行迁移时）。 当你从控制台运行迁移时，它将尝试查找你 `DbConfiguration` 的。 但是，无论是否获取包装服务，都取决于它所注册的事件处理程序的位置。 如果已将其注册为构造的一部分，则 `DbConfiguration` 应执行代码并且应包装服务。 通常不会出现这种情况，这意味着工具不会获得包装服务。  
