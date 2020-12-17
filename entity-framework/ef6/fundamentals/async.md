---
title: Async query 和 EF6
description: 异步查询并保存实体框架6
author: ajcvickers
ms.date: 10/23/2016
uid: ef6/fundamentals/async
ms.openlocfilehash: 77204f56e4dca63322c8ae2e1117318262f16f83
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97635726"
---
# <a name="async-query-and-save"></a><span data-ttu-id="fecc0-103">异步查询并保存</span><span class="sxs-lookup"><span data-stu-id="fecc0-103">Async query and save</span></span>
> [!NOTE]
> <span data-ttu-id="fecc0-104">**仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。</span><span class="sxs-lookup"><span data-stu-id="fecc0-104">**EF6 Onwards Only** - The features, APIs, etc. discussed in this page were introduced in Entity Framework 6.</span></span> <span data-ttu-id="fecc0-105">如果使用的是早期版本，则部分或全部信息不适用。</span><span class="sxs-lookup"><span data-stu-id="fecc0-105">If you are using an earlier version, some or all of the information does not apply.</span></span>

<span data-ttu-id="fecc0-106">EF6 引入了对异步查询的支持，并使用 .NET 4.5 中引入的 [async 和 await 关键字](https://msdn.microsoft.com/library/vstudio/hh191443.aspx) 进行保存。</span><span class="sxs-lookup"><span data-stu-id="fecc0-106">EF6 introduced support for asynchronous query and save using the [async and await keywords](https://msdn.microsoft.com/library/vstudio/hh191443.aspx) that were introduced in .NET 4.5.</span></span> <span data-ttu-id="fecc0-107">虽然并非所有应用程序都可以受益于异步，但它可用于在处理长时间运行的、网络或 i/o 限制的任务时提高客户端的响应能力和服务器的可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="fecc0-107">While not all applications may benefit from asynchrony, it can be used to improve client responsiveness and server scalability when handling long-running, network or I/O-bound tasks.</span></span>

## <a name="when-to-really-use-async"></a><span data-ttu-id="fecc0-108">何时真正使用 async</span><span class="sxs-lookup"><span data-stu-id="fecc0-108">When to really use async</span></span>

<span data-ttu-id="fecc0-109">本演练的目的是以一种可以轻松地观察异步和同步程序执行之间的差异的方式引入异步概念。</span><span class="sxs-lookup"><span data-stu-id="fecc0-109">The purpose of this walkthrough is to introduce the async concepts in a way that makes it easy to observe the difference between asynchronous and synchronous program execution.</span></span> <span data-ttu-id="fecc0-110">本演练并不旨在说明异步编程提供了好处的关键方案。</span><span class="sxs-lookup"><span data-stu-id="fecc0-110">This walkthrough is not intended to illustrate any of the key scenarios where async programming provides benefits.</span></span>

<span data-ttu-id="fecc0-111">异步编程主要侧重于释放当前托管线程 (运行 .NET 代码的线程) 在等待不需要托管线程的任何计算时间的操作时执行其他工作。</span><span class="sxs-lookup"><span data-stu-id="fecc0-111">Async programming is primarily focused on freeing up the current managed thread (thread running .NET code) to do other work while it waits for an operation that does not require any compute time from a managed thread.</span></span> <span data-ttu-id="fecc0-112">例如，当数据库引擎正在处理查询时，.NET 代码不会执行任何操作。</span><span class="sxs-lookup"><span data-stu-id="fecc0-112">For example, whilst the database engine is processing a query there is nothing to be done by .NET code.</span></span>

<span data-ttu-id="fecc0-113">在客户端应用程序中 (WinForms、WPF 等 ) 当前线程可用于在执行异步操作时保持 UI 的响应能力。</span><span class="sxs-lookup"><span data-stu-id="fecc0-113">In client applications (WinForms, WPF, etc.) the current thread can be used to keep the UI responsive while the async operation is performed.</span></span> <span data-ttu-id="fecc0-114">在服务器应用程序中 (ASP.NET 等 ) 线程可用于处理其他传入请求-这可以减少内存使用量和/或提高服务器的吞吐量。</span><span class="sxs-lookup"><span data-stu-id="fecc0-114">In server applications (ASP.NET etc.) the thread can be used to process other incoming requests - this can reduce memory usage and/or increase throughput of the server.</span></span>

<span data-ttu-id="fecc0-115">在大多数使用 async 的应用程序中，没有明显的好处，甚至可能会造成不利影响。</span><span class="sxs-lookup"><span data-stu-id="fecc0-115">In most applications using async will have no noticeable benefits and even could be detrimental.</span></span> <span data-ttu-id="fecc0-116">在提交到特定方案之前，请使用测试、分析和常见意义来度量异步的影响。</span><span class="sxs-lookup"><span data-stu-id="fecc0-116">Use tests, profiling and common sense to measure the impact of async in your particular scenario before committing to it.</span></span>

<span data-ttu-id="fecc0-117">下面是一些用于了解 async 的更多资源：</span><span class="sxs-lookup"><span data-stu-id="fecc0-117">Here are some more resources to learn about async:</span></span>

-   [<span data-ttu-id="fecc0-118">.NET 4.5 中的 Brandon Bray 概述</span><span class="sxs-lookup"><span data-stu-id="fecc0-118">Brandon Bray’s overview of async/await in .NET 4.5</span></span>](https://devblogs.microsoft.com/dotnet/async-in-4-5-worth-the-await/)
-   <span data-ttu-id="fecc0-119">MSDN Library 中的[异步编程](https://msdn.microsoft.com/library/hh191443.aspx)页</span><span class="sxs-lookup"><span data-stu-id="fecc0-119">[Asynchronous Programming](https://msdn.microsoft.com/library/hh191443.aspx) pages in the MSDN Library</span></span>
-   <span data-ttu-id="fecc0-120">[如何使用 Async 构建 ASP.NET Web 应用程序](https://channel9.msdn.com/events/teched/northamerica/2013/dev-b337) (包括提高服务器吞吐量的演示) </span><span class="sxs-lookup"><span data-stu-id="fecc0-120">[How to Build ASP.NET Web Applications Using Async](https://channel9.msdn.com/events/teched/northamerica/2013/dev-b337) (includes a demo of increased server throughput)</span></span>

## <a name="create-the-model"></a><span data-ttu-id="fecc0-121">创建模型</span><span class="sxs-lookup"><span data-stu-id="fecc0-121">Create the model</span></span>

<span data-ttu-id="fecc0-122">我们将使用 [Code First 工作流](xref:ef6/modeling/code-first/workflows/new-database) 来创建模型并生成数据库，但异步功能适用于所有 ef 模型，包括使用 EF 设计器创建的那些模型。</span><span class="sxs-lookup"><span data-stu-id="fecc0-122">We’ll be using the [Code First workflow](xref:ef6/modeling/code-first/workflows/new-database) to create our model and generate the database, however the asynchronous functionality will work with all EF models including those created with the EF Designer.</span></span>

-   <span data-ttu-id="fecc0-123">创建控制台应用程序并将其调用 **AsyncDemo**</span><span class="sxs-lookup"><span data-stu-id="fecc0-123">Create a Console Application and call it **AsyncDemo**</span></span>
-   <span data-ttu-id="fecc0-124">添加 EntityFramework NuGet 包</span><span class="sxs-lookup"><span data-stu-id="fecc0-124">Add the EntityFramework NuGet package</span></span>
    -   <span data-ttu-id="fecc0-125">在解决方案资源管理器中，右键单击 **AsyncDemo** 项目</span><span class="sxs-lookup"><span data-stu-id="fecc0-125">In Solution Explorer, right-click on the **AsyncDemo** project</span></span>
    -   <span data-ttu-id="fecc0-126">选择 "**管理 NuGet 包 ...** "</span><span class="sxs-lookup"><span data-stu-id="fecc0-126">Select **Manage NuGet Packages…**</span></span>
    -   <span data-ttu-id="fecc0-127">在 "管理 NuGet 包" 对话框中，选择 " **联机** " 选项卡，然后选择 " **EntityFramework** " 包</span><span class="sxs-lookup"><span data-stu-id="fecc0-127">In the Manage NuGet Packages dialog, Select the **Online** tab and choose the **EntityFramework** package</span></span>
    -   <span data-ttu-id="fecc0-128">单击“安装” </span><span class="sxs-lookup"><span data-stu-id="fecc0-128">Click **Install**</span></span>
-   <span data-ttu-id="fecc0-129">添加具有以下实现的 **Model.cs** 类</span><span class="sxs-lookup"><span data-stu-id="fecc0-129">Add a **Model.cs** class with the following implementation</span></span>

``` csharp
    using System.Collections.Generic;
    using System.Data.Entity;

    namespace AsyncDemo
    {
        public class BloggingContext : DbContext
        {
            public DbSet<Blog> Blogs { get; set; }
            public DbSet<Post> Posts { get; set; }
        }

        public class Blog
        {
            public int BlogId { get; set; }
            public string Name { get; set; }

            public virtual List<Post> Posts { get; set; }
        }

        public class Post
        {
            public int PostId { get; set; }
            public string Title { get; set; }
            public string Content { get; set; }

            public int BlogId { get; set; }
            public virtual Blog Blog { get; set; }
        }
    }
```

 

## <a name="create-a-synchronous-program"></a><span data-ttu-id="fecc0-130">创建同步程序</span><span class="sxs-lookup"><span data-stu-id="fecc0-130">Create a synchronous program</span></span>

<span data-ttu-id="fecc0-131">现在我们有了 EF 模型，接下来我们编写一些代码，用它来执行某些数据访问。</span><span class="sxs-lookup"><span data-stu-id="fecc0-131">Now that we have an EF model, let's write some code that uses it to perform some data access.</span></span>

-   <span data-ttu-id="fecc0-132">将 **Program.cs** 的内容替换为以下代码</span><span class="sxs-lookup"><span data-stu-id="fecc0-132">Replace the contents of **Program.cs** with the following code</span></span>

``` csharp
    using System;
    using System.Linq;

    namespace AsyncDemo
    {
        class Program
        {
            static void Main(string[] args)
            {
                PerformDatabaseOperations();

                Console.WriteLine("Quote of the day");
                Console.WriteLine(" Don't worry about the world coming to an end today... ");
                Console.WriteLine(" It's already tomorrow in Australia.");

                Console.WriteLine();
                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }

            public static void PerformDatabaseOperations()
            {
                using (var db = new BloggingContext())
                {
                    // Create a new blog and save it
                    db.Blogs.Add(new Blog
                    {
                        Name = "Test Blog #" + (db.Blogs.Count() + 1)
                    });
                    Console.WriteLine("Calling SaveChanges.");
                    db.SaveChanges();
                    Console.WriteLine("SaveChanges completed.");

                    // Query for all blogs ordered by name
                    Console.WriteLine("Executing query.");
                    var blogs = (from b in db.Blogs
                                orderby b.Name
                                select b).ToList();

                    // Write all blogs out to Console
                    Console.WriteLine("Query completed with following results:");
                    foreach (var blog in blogs)
                    {
                        Console.WriteLine(" " + blog.Name);
                    }
                }
            }
        }
    }
```

<span data-ttu-id="fecc0-133">此代码调用方法，该 `PerformDatabaseOperations` 方法将新的 **博客** 保存到数据库，然后从数据库中检索所有 **博客** 并将其打印到 **控制台**。</span><span class="sxs-lookup"><span data-stu-id="fecc0-133">This code calls the `PerformDatabaseOperations` method which saves a new **Blog** to the database and then retrieves all **Blogs** from the database and prints them to the **Console**.</span></span> <span data-ttu-id="fecc0-134">完成此操作后，程序将一天的报价写入 **控制台**。</span><span class="sxs-lookup"><span data-stu-id="fecc0-134">After this, the program writes a quote of the day to the **Console**.</span></span>

<span data-ttu-id="fecc0-135">由于代码是同步的，因此，当我们运行程序时，可以观察到以下执行流：</span><span class="sxs-lookup"><span data-stu-id="fecc0-135">Since the code is synchronous, we can observe the following execution flow when we run the program:</span></span>

1.  <span data-ttu-id="fecc0-136">`SaveChanges` 开始将新 **博客** 推送到数据库</span><span class="sxs-lookup"><span data-stu-id="fecc0-136">`SaveChanges` begins to push the new **Blog** to the database</span></span>
2.  <span data-ttu-id="fecc0-137">`SaveChanges` 表示</span><span class="sxs-lookup"><span data-stu-id="fecc0-137">`SaveChanges` completes</span></span>
3.  <span data-ttu-id="fecc0-138">所有 **博客** 的查询都发送到数据库</span><span class="sxs-lookup"><span data-stu-id="fecc0-138">Query for all **Blogs** is sent to the database</span></span>
4.  <span data-ttu-id="fecc0-139">查询返回并将结果写入 **控制台**</span><span class="sxs-lookup"><span data-stu-id="fecc0-139">Query returns and results are written to **Console**</span></span>
5.  <span data-ttu-id="fecc0-140">将日报价写入 **控制台**</span><span class="sxs-lookup"><span data-stu-id="fecc0-140">Quote of the day is written to **Console**</span></span>

![同步输出](~/ef6/media/syncoutput.png) 

 

## <a name="making-it-asynchronous"></a><span data-ttu-id="fecc0-142">使其异步</span><span class="sxs-lookup"><span data-stu-id="fecc0-142">Making it asynchronous</span></span>

<span data-ttu-id="fecc0-143">现在，我们已启动并运行程序，接下来可以开始使用新的 async 和 await 关键字。</span><span class="sxs-lookup"><span data-stu-id="fecc0-143">Now that we have our program up and running, we can begin making use of the new async and await keywords.</span></span> <span data-ttu-id="fecc0-144">我们已对 Program.cs 进行了以下更改</span><span class="sxs-lookup"><span data-stu-id="fecc0-144">We've made the following changes to Program.cs</span></span>

1.  <span data-ttu-id="fecc0-145">第2行：命名空间的 using 语句 `System.Data.Entity` 使我们能够访问 EF async extension 方法。</span><span class="sxs-lookup"><span data-stu-id="fecc0-145">Line 2: The using statement for the `System.Data.Entity` namespace gives us access to the EF async extension methods.</span></span>
2.  <span data-ttu-id="fecc0-146">第4行：命名空间的 using 语句 `System.Threading.Tasks` 允许使用该 `Task` 类型。</span><span class="sxs-lookup"><span data-stu-id="fecc0-146">Line 4: The using statement for the `System.Threading.Tasks` namespace allows us to use the `Task` type.</span></span>
3.  <span data-ttu-id="fecc0-147">第12行 & 18：我们正在捕获到监视 `PerformSomeDatabaseOperations` (行 12) 进度的任务，然后在程序的所有工作完成 (第) 18 行后阻止程序执行完成此任务完成。</span><span class="sxs-lookup"><span data-stu-id="fecc0-147">Line 12 & 18: We are capturing as task that monitors the progress of `PerformSomeDatabaseOperations` (line 12) and then blocking program execution for this task to complete once all the work for the program is done (line 18).</span></span>
4.  <span data-ttu-id="fecc0-148">第25行：我们更新 `PerformSomeDatabaseOperations` 为标记为 `async` ，并返回 `Task` 。</span><span class="sxs-lookup"><span data-stu-id="fecc0-148">Line 25: We've update `PerformSomeDatabaseOperations` to be marked as `async` and return a `Task`.</span></span>
5.  <span data-ttu-id="fecc0-149">第35行：我们正在调用的异步版本 `SaveChanges` ，并等待其完成。</span><span class="sxs-lookup"><span data-stu-id="fecc0-149">Line 35: We're now calling the Async version of `SaveChanges` and awaiting it's completion.</span></span>
6.  <span data-ttu-id="fecc0-150">第42行：我们正在调用的异步版本 `ToList` ，并等待结果。</span><span class="sxs-lookup"><span data-stu-id="fecc0-150">Line 42: We're now calling the Async version of `ToList` and awaiting on the result.</span></span>

<span data-ttu-id="fecc0-151">有关命名空间中可用扩展方法的完整列表 `System.Data.Entity` ，请参阅 `QueryableExtensions` 类。</span><span class="sxs-lookup"><span data-stu-id="fecc0-151">For a comprehensive list of available extension methods in the `System.Data.Entity` namespace, refer to the `QueryableExtensions` class.</span></span> <span data-ttu-id="fecc0-152">*还需要将添加 `using System.Data.Entity` 到 using 语句。*</span><span class="sxs-lookup"><span data-stu-id="fecc0-152">*You’ll also need to add `using System.Data.Entity` to your using statements.*</span></span>

``` csharp
    using System;
    using System.Data.Entity;
    using System.Linq;
    using System.Threading.Tasks;

    namespace AsyncDemo
    {
        class Program
        {
            static void Main(string[] args)
            {
                var task = PerformDatabaseOperations();

                Console.WriteLine("Quote of the day");
                Console.WriteLine(" Don't worry about the world coming to an end today... ");
                Console.WriteLine(" It's already tomorrow in Australia.");

                task.Wait();

                Console.WriteLine();
                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }

            public static async Task PerformDatabaseOperations()
            {
                using (var db = new BloggingContext())
                {
                    // Create a new blog and save it
                    db.Blogs.Add(new Blog
                    {
                        Name = "Test Blog #" + (db.Blogs.Count() + 1)
                    });
                    Console.WriteLine("Calling SaveChanges.");
                    await db.SaveChangesAsync();
                    Console.WriteLine("SaveChanges completed.");

                    // Query for all blogs ordered by name
                    Console.WriteLine("Executing query.");
                    var blogs = await (from b in db.Blogs
                                orderby b.Name
                                select b).ToListAsync();

                    // Write all blogs out to Console
                    Console.WriteLine("Query completed with following results:");
                    foreach (var blog in blogs)
                    {
                        Console.WriteLine(" - " + blog.Name);
                    }
                }
            }
        }
    }
```

<span data-ttu-id="fecc0-153">由于代码是异步的，因此，我们可以在运行程序时观察到不同的执行流程：</span><span class="sxs-lookup"><span data-stu-id="fecc0-153">Now that the code is asynchronous, we can observe a different execution flow when we run the program:</span></span>

1. <span data-ttu-id="fecc0-154">`SaveChanges` 开始将新 **博客** 推送到数据库</span><span class="sxs-lookup"><span data-stu-id="fecc0-154">`SaveChanges` begins to push the new **Blog** to the database</span></span>  
    <span data-ttu-id="fecc0-155">*将命令发送到数据库后，当前托管线程不需要更多计算时间。 `PerformDatabaseOperations` 即使未完成执行 Main 方法中的) 和程序流，方法也会返回 (。*</span><span class="sxs-lookup"><span data-stu-id="fecc0-155">*Once the command is sent to the database no more compute time is needed on the current managed thread. The `PerformDatabaseOperations` method returns (even though it hasn't finished executing) and program flow in the Main method continues.*</span></span>
2. <span data-ttu-id="fecc0-156">**将日报价写入控制台**</span><span class="sxs-lookup"><span data-stu-id="fecc0-156">**Quote of the day is written to Console**</span></span>  
    <span data-ttu-id="fecc0-157">*由于在 Main 方法中没有更多的工作要做，因此 `Wait` 直到数据库操作完成后，调用中的托管线程才会被阻止。完成后，我们将执行的其余部分 `PerformDatabaseOperations` 。*</span><span class="sxs-lookup"><span data-stu-id="fecc0-157">*Since there is no more work to do in the Main method, the managed thread is blocked on the `Wait` call until the database operation completes. Once it completes, the remainder of our `PerformDatabaseOperations` will be executed.*</span></span>
3.  <span data-ttu-id="fecc0-158">`SaveChanges` 表示</span><span class="sxs-lookup"><span data-stu-id="fecc0-158">`SaveChanges` completes</span></span>  
4.  <span data-ttu-id="fecc0-159">所有 **博客** 的查询都发送到数据库</span><span class="sxs-lookup"><span data-stu-id="fecc0-159">Query for all **Blogs** is sent to the database</span></span>  
    <span data-ttu-id="fecc0-160">*同样，在数据库中处理查询时，托管线程可以自由地执行其他工作。由于所有其他执行都已完成，线程只会在等待调用时停止。*</span><span class="sxs-lookup"><span data-stu-id="fecc0-160">*Again, the managed thread is free to do other work while the query is processed in the database. Since all other execution has completed, the thread will just halt on the Wait call though.*</span></span>
5.  <span data-ttu-id="fecc0-161">查询返回并将结果写入 **控制台**</span><span class="sxs-lookup"><span data-stu-id="fecc0-161">Query returns and results are written to **Console**</span></span>  

![异步输出](~/ef6/media/asyncoutput.png) 

 

## <a name="the-takeaway"></a><span data-ttu-id="fecc0-163">要点在于</span><span class="sxs-lookup"><span data-stu-id="fecc0-163">The takeaway</span></span>

<span data-ttu-id="fecc0-164">现在，我们看到了使用 EF 的异步方法是多么简单。</span><span class="sxs-lookup"><span data-stu-id="fecc0-164">We now saw how easy it is to make use of EF’s asynchronous methods.</span></span> <span data-ttu-id="fecc0-165">尽管 async 的优点在简单的控制台应用程序中可能并不是很明显，但在运行长时间运行或网络绑定的活动可能会阻止应用程序，或者导致大量线程增加内存占用量的情况下，可以应用这些相同的策略。</span><span class="sxs-lookup"><span data-stu-id="fecc0-165">Although the advantages of async may not be very apparent with a simple console app, these same strategies can be applied in situations where long-running or network-bound activities might otherwise block the application, or cause a large number of threads to increase the memory footprint.</span></span>
