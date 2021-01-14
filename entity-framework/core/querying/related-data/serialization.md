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
# <a name="related-data-and-serialization"></a><span data-ttu-id="7d4ef-103">关联数据和序列化</span><span class="sxs-lookup"><span data-stu-id="7d4ef-103">Related data and serialization</span></span>

<span data-ttu-id="7d4ef-104">由于 EF Core 会自动修正导航属性，因此在对象图中可能会产生循环引用。</span><span class="sxs-lookup"><span data-stu-id="7d4ef-104">Because EF Core automatically does fix-up of navigation properties, you can end up with cycles in your object graph.</span></span> <span data-ttu-id="7d4ef-105">例如，加载博客及其关联文章会生成引用文章集合的博客对象。</span><span class="sxs-lookup"><span data-stu-id="7d4ef-105">For example, loading a blog and its related posts will result in a blog object that references a collection of posts.</span></span> <span data-ttu-id="7d4ef-106">而其中每篇文章又会引用该博客。</span><span class="sxs-lookup"><span data-stu-id="7d4ef-106">Each of those posts will have a reference back to the blog.</span></span>

<span data-ttu-id="7d4ef-107">某些序列化框架不允许使用循环引用。</span><span class="sxs-lookup"><span data-stu-id="7d4ef-107">Some serialization frameworks don't allow such cycles.</span></span> <span data-ttu-id="7d4ef-108">例如，Json.NET 在发现循环引用的情况下，会引发以下异常。</span><span class="sxs-lookup"><span data-stu-id="7d4ef-108">For example, Json.NET will throw the following exception if a cycle is found.</span></span>

> <span data-ttu-id="7d4ef-109">Newtonsoft.Json.JsonSerializationException：为“MyApplication.Models.Blog”类型的“Blog”属性检测到自引用循环。</span><span class="sxs-lookup"><span data-stu-id="7d4ef-109">Newtonsoft.Json.JsonSerializationException: Self referencing loop detected for property 'Blog' with type 'MyApplication.Models.Blog'.</span></span>

<span data-ttu-id="7d4ef-110">如果正在使用 ASP.NET Core，则可以将 Json.NET 配置为忽略在对象图中找到的循环引用。</span><span class="sxs-lookup"><span data-stu-id="7d4ef-110">If you're using ASP.NET Core, you can configure Json.NET to ignore cycles that it finds in the object graph.</span></span> <span data-ttu-id="7d4ef-111">此配置是通过 `Startup.cs` 中的 `ConfigureServices(...)` 方法完成的。</span><span class="sxs-lookup"><span data-stu-id="7d4ef-111">This configuration is done in the `ConfigureServices(...)` method in `Startup.cs`.</span></span>

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

<span data-ttu-id="7d4ef-112">另一种方法是使用 `[JsonIgnore]` 特性修饰其中一个导航属性，该特性指示 Json.NET 在序列化时不遍历该导航属性。</span><span class="sxs-lookup"><span data-stu-id="7d4ef-112">Another alternative is to decorate one of the navigation properties with the `[JsonIgnore]` attribute, which instructs Json.NET to not traverse that navigation property while serializing.</span></span>
