---
title: 关联数据和序列化 - EF Core
description: 关于使用 Entity Framework Core 的相关数据中的周期如何影响序列化框架的信息
author: roji
ms.date: 9/8/2020
uid: core/querying/related-data/serialization
ms.openlocfilehash: 3e1152f09493d039032d9d88493b52a5ef425eb9
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129130"
---
# <a name="related-data-and-serialization"></a>关联数据和序列化

由于 EF Core 会自动修正导航属性，因此在对象图中可能会产生循环引用。 例如，加载博客及其关联文章会生成引用文章集合的博客对象。 而其中每篇文章又会引用该博客。

某些序列化框架不允许使用循环引用。 例如，Json.NET 在发现循环引用的情况下，会引发以下异常。

> Newtonsoft.Json.JsonSerializationException：为“MyApplication.Models.Blog”类型的“Blog”属性检测到自引用循环。

如果正在使用 ASP.NET Core，则可以将 Json.NET 配置为忽略在对象图中找到的循环引用。 此配置是通过 `Startup.cs` 中的 `ConfigureServices(...)` 方法完成的。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...

    services.AddMvc()
        .AddJsonOptions(
            options => options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore
        );

    ...
}
```

另一种方法是使用 `[JsonIgnore]` 特性修饰其中一个导航属性，该特性指示 Json.NET 在序列化时不遍历该导航属性。
