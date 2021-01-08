---
title: 相关数据的延迟加载 - EF Core
description: 使用 Entity Framework Core 延迟加载相关数据
author: roji
ms.date: 9/8/2020
uid: core/querying/related-data/lazy
ms.openlocfilehash: 55622b9c5a8f70ef4e7246d6eb14678036948f18
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97635453"
---
# <a name="lazy-loading-of-related-data"></a><span data-ttu-id="10b92-103">相关数据的延迟加载</span><span class="sxs-lookup"><span data-stu-id="10b92-103">Lazy Loading of Related Data</span></span>

## <a name="lazy-loading-with-proxies"></a><span data-ttu-id="10b92-104">使用代理的延迟加载</span><span class="sxs-lookup"><span data-stu-id="10b92-104">Lazy loading with proxies</span></span>

<span data-ttu-id="10b92-105">使用延迟加载的最简单方式是通过安装 [Microsoft.EntityFrameworkCore.Proxies](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Proxies/) 包，并通过调用 `UseLazyLoadingProxies` 来启用该包。</span><span class="sxs-lookup"><span data-stu-id="10b92-105">The simplest way to use lazy-loading is by installing the [Microsoft.EntityFrameworkCore.Proxies](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Proxies/) package and enabling it with a call to `UseLazyLoadingProxies`.</span></span> <span data-ttu-id="10b92-106">例如：</span><span class="sxs-lookup"><span data-stu-id="10b92-106">For example:</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .UseLazyLoadingProxies()
        .UseSqlServer(myConnectionString);
```

<span data-ttu-id="10b92-107">或在使用 AddDbContext 时：</span><span class="sxs-lookup"><span data-stu-id="10b92-107">Or when using AddDbContext:</span></span>

```csharp
.AddDbContext<BloggingContext>(
    b => b.UseLazyLoadingProxies()
          .UseSqlServer(myConnectionString));
```

<span data-ttu-id="10b92-108">EF Core 接着会为可重写的任何导航属性（即，必须是 `virtual` 且在可被继承的类上）启用延迟加载。</span><span class="sxs-lookup"><span data-stu-id="10b92-108">EF Core will then enable lazy loading for any navigation property that can be overridden--that is, it must be `virtual` and on a class that can be inherited from.</span></span> <span data-ttu-id="10b92-109">例如，在以下实体中，`Post.Blog` 和 `Blog.Posts` 导航属性将被延迟加载。</span><span class="sxs-lookup"><span data-stu-id="10b92-109">For example, in the following entities, the `Post.Blog` and `Blog.Posts` navigation properties will be lazy-loaded.</span></span>

```csharp
public class Blog
{
    public int Id { get; set; }
    public string Name { get; set; }

    public virtual ICollection<Post> Posts { get; set; }
}

public class Post
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public virtual Blog Blog { get; set; }
}
```

> [!WARNING]
> <span data-ttu-id="10b92-110">延迟加载可能会导致发生不必要的额外数据库往返（即 N+1 问题），应注意避免此问题。</span><span class="sxs-lookup"><span data-stu-id="10b92-110">Lazy loading can cause unneeded extra database roundtrips to occur (the so-called N+1 problem), and care should be taken to avoid this.</span></span> <span data-ttu-id="10b92-111">有关详细信息，请参阅[性能部分](xref:core/performance/efficient-querying#beware-of-lazy-loading)。</span><span class="sxs-lookup"><span data-stu-id="10b92-111">See the [performance section](xref:core/performance/efficient-querying#beware-of-lazy-loading) for more details.</span></span>

## <a name="lazy-loading-without-proxies"></a><span data-ttu-id="10b92-112">不使用代理的延迟加载</span><span class="sxs-lookup"><span data-stu-id="10b92-112">Lazy loading without proxies</span></span>

<span data-ttu-id="10b92-113">使用代理进行延迟加载的工作方式是将 `ILazyLoader` 注入到实体中，如[实体类型构造函数](xref:core/modeling/constructors)中所述。</span><span class="sxs-lookup"><span data-stu-id="10b92-113">Lazy-loading proxies work by injecting the `ILazyLoader` service into an entity, as described in [Entity Type Constructors](xref:core/modeling/constructors).</span></span> <span data-ttu-id="10b92-114">例如：</span><span class="sxs-lookup"><span data-stu-id="10b92-114">For example:</span></span>

```csharp
public class Blog
{
    private ICollection<Post> _posts;

    public Blog()
    {
    }

    private Blog(ILazyLoader lazyLoader)
    {
        LazyLoader = lazyLoader;
    }

    private ILazyLoader LazyLoader { get; set; }

    public int Id { get; set; }
    public string Name { get; set; }

    public ICollection<Post> Posts
    {
        get => LazyLoader.Load(this, ref _posts);
        set => _posts = value;
    }
}

public class Post
{
    private Blog _blog;

    public Post()
    {
    }

    private Post(ILazyLoader lazyLoader)
    {
        LazyLoader = lazyLoader;
    }

    private ILazyLoader LazyLoader { get; set; }

    public int Id { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public Blog Blog
    {
        get => LazyLoader.Load(this, ref _blog);
        set => _blog = value;
    }
}
```

<span data-ttu-id="10b92-115">此方法不要求实体类型为可继承的类型，也不要求导航属性必须是虚拟的，且允许通过 `new` 创建的实体实例在附加到上下文后可进行延迟加载。</span><span class="sxs-lookup"><span data-stu-id="10b92-115">This method doesn't require entity types to be inherited from or navigation properties to be virtual, and allows entity instances created with `new` to lazy-load once attached to a context.</span></span> <span data-ttu-id="10b92-116">但它需要对 [Microsoft.EntityFrameworkCore.Abstractions](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Abstractions/) 包中定义的 `ILazyLoader` 服务的引用。</span><span class="sxs-lookup"><span data-stu-id="10b92-116">However, it requires a reference to the `ILazyLoader` service, which is defined in the [Microsoft.EntityFrameworkCore.Abstractions](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Abstractions/) package.</span></span> <span data-ttu-id="10b92-117">此包包含所允许的最少的一组类型，以便将依赖此包时所产生的影响降至最低。</span><span class="sxs-lookup"><span data-stu-id="10b92-117">This package contains a minimal set of types so that there is little impact in depending on it.</span></span> <span data-ttu-id="10b92-118">不过，可以将 `ILazyLoader.Load` 方法以委托的形式注入，这样就可以完全避免依赖于实体类型的任何 EF Core 包。</span><span class="sxs-lookup"><span data-stu-id="10b92-118">However, to completely avoid depending on any EF Core packages in the entity types, it's possible to inject the `ILazyLoader.Load` method as a delegate.</span></span> <span data-ttu-id="10b92-119">例如：</span><span class="sxs-lookup"><span data-stu-id="10b92-119">For example:</span></span>

```csharp
public class Blog
{
    private ICollection<Post> _posts;

    public Blog()
    {
    }

    private Blog(Action<object, string> lazyLoader)
    {
        LazyLoader = lazyLoader;
    }

    private Action<object, string> LazyLoader { get; set; }

    public int Id { get; set; }
    public string Name { get; set; }

    public ICollection<Post> Posts
    {
        get => LazyLoader.Load(this, ref _posts);
        set => _posts = value;
    }
}

public class Post
{
    private Blog _blog;

    public Post()
    {
    }

    private Post(Action<object, string> lazyLoader)
    {
        LazyLoader = lazyLoader;
    }

    private Action<object, string> LazyLoader { get; set; }

    public int Id { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public Blog Blog
    {
        get => LazyLoader.Load(this, ref _blog);
        set => _blog = value;
    }
}
```

<span data-ttu-id="10b92-120">上述代码使用 `Load` 扩展方法，以便更干净地使用委托：</span><span class="sxs-lookup"><span data-stu-id="10b92-120">The code above uses a `Load` extension method to make using the delegate a bit cleaner:</span></span>

```csharp
public static class PocoLoadingExtensions
{
    public static TRelated Load<TRelated>(
        this Action<object, string> loader,
        object entity,
        ref TRelated navigationField,
        [CallerMemberName] string navigationName = null)
        where TRelated : class
    {
        loader?.Invoke(entity, navigationName);

        return navigationField;
    }
}
```

> [!NOTE]
> <span data-ttu-id="10b92-121">延迟加载委托的构造函数参数必须名为“lazyLoader”。</span><span class="sxs-lookup"><span data-stu-id="10b92-121">The constructor parameter for the lazy-loading delegate must be called "lazyLoader".</span></span> <span data-ttu-id="10b92-122">未来的一个版本中的配置将计划采用另一个名称。</span><span class="sxs-lookup"><span data-stu-id="10b92-122">Configuration to use a different name than this is planned for a future release.</span></span>
