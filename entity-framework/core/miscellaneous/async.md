---
title: 异步编程-EF Core
description: 通过 Entity Framework Core 异步查询和保存数据
author: roji
ms.date: 9/2/2020
uid: core/miscellaneous/async
ms.openlocfilehash: 52b3ac20a50babbed6937ebe3365ac1947dcaef1
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128571"
---
# <a name="asynchronous-programming"></a><span data-ttu-id="b976b-103">异步编程</span><span class="sxs-lookup"><span data-stu-id="b976b-103">Asynchronous Programming</span></span>

<span data-ttu-id="b976b-104">当在数据库中执行查询时，异步操作将避免阻止线程。</span><span class="sxs-lookup"><span data-stu-id="b976b-104">Asynchronous operations avoid blocking a thread while the query is executed in the database.</span></span> <span data-ttu-id="b976b-105">异步操作对于在丰富的客户端应用程序中保持响应式 UI 非常重要，并且还可以增加 web 应用程序中的吞吐量，在这些应用程序中，它们可释放线程以处理 web 应用程序中的其他请求。</span><span class="sxs-lookup"><span data-stu-id="b976b-105">Async operations are important for keeping a responsive UI in rich client applications, and can also increase throughput in web applications where they free up the thread to service other requests in web applications.</span></span>

<span data-ttu-id="b976b-106">遵循 .NET standard 后，EF Core 向执行 i/o 的所有同步方法提供异步对应项。</span><span class="sxs-lookup"><span data-stu-id="b976b-106">Following the .NET standard, EF Core provides asynchronous counterparts to all synchronous methods which perform I/O.</span></span> <span data-ttu-id="b976b-107">它们具有与同步方法相同的效果，并且可与 c # `async` 和关键字一起使用 `await` 。</span><span class="sxs-lookup"><span data-stu-id="b976b-107">These have the same effects as the sync methods, and can be used with the C# `async` and `await` keywords.</span></span> <span data-ttu-id="b976b-108">例如，在执行数据库 i/o 时，不使用 DbContext 来阻止线程，而是使用 DbContext。 SaveChangesAsync：</span><span class="sxs-lookup"><span data-stu-id="b976b-108">For example, instead of using DbContext.SaveChanges, which will block a thread while database I/O is performed, DbContext.SaveChangesAsync can be used:</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/Async/Program.cs#SaveChangesAsync)]

<span data-ttu-id="b976b-109">有关详细信息，请参阅 [常规 c # 异步编程文档](/dotnet/csharp/async)。</span><span class="sxs-lookup"><span data-stu-id="b976b-109">For more information, see [the general C# asynchronous programming docs](/dotnet/csharp/async).</span></span>

> [!WARNING]
> <span data-ttu-id="b976b-110">EF Core 不支持在同一上下文实例上运行多个并行操作。</span><span class="sxs-lookup"><span data-stu-id="b976b-110">EF Core doesn't support multiple parallel operations being run on the same context instance.</span></span> <span data-ttu-id="b976b-111">应始终等待操作完成，然后再开始下一个操作。</span><span class="sxs-lookup"><span data-stu-id="b976b-111">You should always wait for an operation to complete before beginning the next operation.</span></span> <span data-ttu-id="b976b-112">这通常是通过在每个异步操作上使用 `await` 关键字完成的。</span><span class="sxs-lookup"><span data-stu-id="b976b-112">This is typically done by using the `await` keyword on each async operation.</span></span>

> [!WARNING]
> <span data-ttu-id="b976b-113">[SqlClient](https://github.com/dotnet/SqlClient)的异步实现中存在一些已知问题 (例如[#593](https://github.com/dotnet/SqlClient/issues/593)、 [#601](https://github.com/dotnet/SqlClient/issues/601)和其他) 。</span><span class="sxs-lookup"><span data-stu-id="b976b-113">The async implementation of [Microsoft.Data.SqlClient](https://github.com/dotnet/SqlClient) unfortunately has some known issues (e.g. [#593](https://github.com/dotnet/SqlClient/issues/593), [#601](https://github.com/dotnet/SqlClient/issues/601), and others).</span></span>

> [!NOTE]
> <span data-ttu-id="b976b-114">EF Core 将取消标记传递给中使用的基础数据库提供程序 (例如 SqlClient) 。</span><span class="sxs-lookup"><span data-stu-id="b976b-114">EF Core passes cancellation tokens down to the underlying database provider in use (e.g. Microsoft.Data.SqlClient).</span></span> <span data-ttu-id="b976b-115">这些令牌可能会也可能不起作用-请参考数据库提供商的文档。</span><span class="sxs-lookup"><span data-stu-id="b976b-115">These tokens may or may not be honored - consult your database provider's documentation.</span></span>

## <a name="async-linq-operators"></a><span data-ttu-id="b976b-116">异步 LINQ 运算符</span><span class="sxs-lookup"><span data-stu-id="b976b-116">Async LINQ operators</span></span>

<span data-ttu-id="b976b-117">为了支持异步执行 LINQ 查询，EF Core 提供了一组异步扩展方法，这些方法执行查询并返回结果。</span><span class="sxs-lookup"><span data-stu-id="b976b-117">In order to support executing LINQ queries asynchronously, EF Core provides a set of async extension methods which execute the query and return results.</span></span> <span data-ttu-id="b976b-118">这些对应于标准的同步 LINQ 运算符包括 ToListAsync、SingleAsync、AsAsyncEnumerable 等：</span><span class="sxs-lookup"><span data-stu-id="b976b-118">These counterparts to the standard, synchronous LINQ operators include ToListAsync, SingleAsync, AsAsyncEnumerable, etc.:</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/Async/Program.cs#ToListAsync)]

<span data-ttu-id="b976b-119">请注意，有些 LINQ 运算符（如 Where 或 OrderBy）没有异步版本，因为这些运算符仅生成 LINQ 表达式树，而不会导致在数据库中执行查询。</span><span class="sxs-lookup"><span data-stu-id="b976b-119">Note that there are no async versions of some LINQ operators such as Where or OrderBy, because these only build up the LINQ expression tree and don't cause the query to be executed in the database.</span></span> <span data-ttu-id="b976b-120">只有导致执行查询的运算符具有 async 对应项。</span><span class="sxs-lookup"><span data-stu-id="b976b-120">Only operators which cause query execution have async counterparts.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="b976b-121">在 `Microsoft.EntityFrameworkCore` 命名空间中定义 EF Core 异步扩展方法。</span><span class="sxs-lookup"><span data-stu-id="b976b-121">The EF Core async extension methods are defined in the `Microsoft.EntityFrameworkCore` namespace.</span></span> <span data-ttu-id="b976b-122">必须导入此命名空间才能使这些方法可用。</span><span class="sxs-lookup"><span data-stu-id="b976b-122">This namespace must be imported for the methods to be available.</span></span>

## <a name="client-side-async-linq-operators"></a><span data-ttu-id="b976b-123">客户端异步 LINQ 运算符</span><span class="sxs-lookup"><span data-stu-id="b976b-123">Client-side async LINQ operators</span></span>

<span data-ttu-id="b976b-124">上面所述的异步 LINQ 运算符只能在 EF 查询上使用-不能将其与客户端 LINQ to Objects 查询一起使用。</span><span class="sxs-lookup"><span data-stu-id="b976b-124">The async LINQ operators discussed above can only be used on EF queries - you cannot use them with client-side LINQ to Objects query.</span></span> <span data-ttu-id="b976b-125">若要在 EF 之外执行客户端异步 LINQ 操作，请使用 system.exception [包](https://www.nuget.org/packages/System.Interactive.Async);此包对于在客户端上执行无法在服务器上进行求值的操作时特别有用。</span><span class="sxs-lookup"><span data-stu-id="b976b-125">To perform client-side async LINQ operations outside of EF, use the [System.Interactive.Async package](https://www.nuget.org/packages/System.Interactive.Async); this package can be especially useful for performing operations on the client that cannot be translated for evaluation at the server.</span></span>

<span data-ttu-id="b976b-126">遗憾的是，引用 system.exception 会导致应用于 EF Dbset 的 LINQ 运算符上出现不明确的调用编译错误;这使得难以在同一项目中使用 EF 和 System.object。</span><span class="sxs-lookup"><span data-stu-id="b976b-126">Unfortunately, referencing System.Interactive.Async causes ambiguous invocation compilation errors on LINQ operators applied to EF's DbSets; this makes it hard to use both EF and System.Interactive.Async in the same project.</span></span> <span data-ttu-id="b976b-127">若要解决此问题，请将 AsQueryable 添加到 DbSet：</span><span class="sxs-lookup"><span data-stu-id="b976b-127">To work around this issue, add AsQueryable to your DbSet:</span></span>

[!code-csharp[Main](../../../samples/core/Miscellaneous/AsyncWithSystemInteractive/Program.cs#SystemInteractiveAsync)]
