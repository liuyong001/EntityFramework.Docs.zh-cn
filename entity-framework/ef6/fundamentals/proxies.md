---
title: 使用代理-EF6
description: 使用实体框架6中的代理
author: ajcvickers
ms.date: 10/23/2016
uid: ef6/fundamentals/proxies
ms.openlocfilehash: de44e891c074a8811e466d040f6451bea0f950f4
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97635843"
---
# <a name="working-with-proxies"></a>使用代理
创建 POCO 实体类型实例时，实体框架通常会创建动态生成的派生类型的实例，该类型充当实体的代理。 此代理会重写实体的某些虚拟属性，以插入挂钩，以便在访问属性时自动执行操作。 例如，此机制用于支持关系的延迟加载。 本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

## <a name="disabling-proxy-creation"></a>禁用代理创建  

有时，阻止实体框架创建代理实例会很有用。 例如，序列化非代理实例比序列化代理实例要容易得多。 可以通过清除标志关闭代理创建 `ProxyCreationEnabled` 。 您可以在上下文的构造函数中执行此操作。 例如：  

``` csharp
public class BloggingContext : DbContext
{
    public BloggingContext()
    {
        this.Configuration.ProxyCreationEnabled = false;
    }  

    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}
```  

请注意，EF 不会为代理创建没有的类型的代理。 这意味着，你还可以通过使用密封的类型和/或没有虚拟属性来避免代理。  

## <a name="explicitly-creating-an-instance-of-a-proxy"></a>显式创建代理的实例  

如果使用 new 运算符创建实体的实例，则不会创建代理实例。 这可能不是问题，但如果你需要创建代理实例 (例如，使延迟加载或代理更改跟踪工作) ，则可以使用的方法来执行此操作 `Create` `DbSet` 。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Create();
}
```  

`Create`如果要创建派生实体类型的实例，则可以使用的泛型版本。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var admin = context.Users.Create<Administrator>();
}
```  

请注意， `Create` 方法不会将创建的实体添加或附加到上下文。  

请注意， `Create` 如果为实体创建代理类型，该方法将仅创建实体类型本身的实例，因为它不会执行任何操作。 例如，如果实体类型是密封的并且/或没有虚拟属性，则 `Create` 只需创建实体类型的实例。  

## <a name="getting-the-actual-entity-type-from-a-proxy-type"></a>从代理类型获取实际实体类型  

代理类型的名称如下所示：  

```
System.Data.Entity.DynamicProxies.Blog_5E43C6C196972BF0754973E48C9C941092D86818CD94005E9A759B70BF6E48E6
```

你可以使用中的方法来查找此代理类型的实体类型 `GetObjectType` `ObjectContext` 。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);
    var entityType = ObjectContext.GetObjectType(blog.GetType());
}
```  

请注意，如果传递给的类型 `GetObjectType` 是不是代理类型的实体类型的实例，则仍将返回实体的类型。 这意味着，始终可以使用此方法获取实际的实体类型，而无需进行任何其他检查即可查看该类型是否为代理类型。  
