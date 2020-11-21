---
title: 事件计数器-EF Core
description: 利用 .NET 事件计数器跟踪 EF Core 性能和诊断异常
author: roji
ms.date: 11/17/2020
uid: core/logging-events-diagnostics/event-counters
ms.openlocfilehash: 46acfe82d8aeb7d16146bae0cc2cd4ff733e2831
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003686"
---
# <a name="event-counters"></a><span data-ttu-id="669ff-103">事件计数器</span><span class="sxs-lookup"><span data-stu-id="669ff-103">Event Counters</span></span>

> [!NOTE]
> <span data-ttu-id="669ff-104">EF Core 5.0 中添加了此功能。</span><span class="sxs-lookup"><span data-stu-id="669ff-104">This feature was added in EF Core 5.0.</span></span>

<span data-ttu-id="669ff-105">Entity Framework Core (EF Core) 会公开连续数值指标，这可以提供程序健康状况的良好指示。</span><span class="sxs-lookup"><span data-stu-id="669ff-105">Entity Framework Core (EF Core) exposes continuous numeric metrics which can provide a good indication of your program's health.</span></span> <span data-ttu-id="669ff-106">这些指标可用于以下目的：</span><span class="sxs-lookup"><span data-stu-id="669ff-106">These metrics can be used for the following purposes:</span></span>

* <span data-ttu-id="669ff-107">在应用程序运行时实时跟踪常规数据库加载</span><span class="sxs-lookup"><span data-stu-id="669ff-107">Track general database load in realtime as the application is running</span></span>
* <span data-ttu-id="669ff-108">公开可能导致性能下降的问题编码做法</span><span class="sxs-lookup"><span data-stu-id="669ff-108">Expose problematic coding practices which can lead to degraded performance</span></span>
* <span data-ttu-id="669ff-109">跟踪和隔离异常程序行为</span><span class="sxs-lookup"><span data-stu-id="669ff-109">Track down and isolate anomalous program behavior</span></span>

<span data-ttu-id="669ff-110">通过标准 .NET 事件计数器功能 EF Core 报告指标;建议阅读 [此博客文章](https://devblogs.microsoft.com/dotnet/introducing-diagnostics-improvements-in-net-core-3-0/) ，了解计数器如何工作的简要概述。</span><span class="sxs-lookup"><span data-stu-id="669ff-110">EF Core reports metrics via the standard .NET event counters feature; it's recommended to read [this blog post](https://devblogs.microsoft.com/dotnet/introducing-diagnostics-improvements-in-net-core-3-0/) for a quick overview of how counters work.</span></span>

## <a name="attach-to-a-process-using-dotnet-counters"></a><span data-ttu-id="669ff-111">使用 dotnet 附加到进程-计数器</span><span class="sxs-lookup"><span data-stu-id="669ff-111">Attach to a process using dotnet-counters</span></span>

<span data-ttu-id="669ff-112">[Dotnet 工具](https://docs.microsoft.com/dotnet/core/diagnostics/dotnet-counters)可用于附加到正在运行的进程，并定期报告 EF Core 事件计数器;无需在程序中执行任何特殊操作即可使用这些计数器。</span><span class="sxs-lookup"><span data-stu-id="669ff-112">The [dotnet-counters tool](https://docs.microsoft.com/dotnet/core/diagnostics/dotnet-counters) can be used to attach to a running process and report EF Core event counters regularly; nothing special needs to be done in the program for these counters to be available.</span></span>

<span data-ttu-id="669ff-113">首先，安装 `dotnet-counters` 工具： `dotnet tool install --global dotnet-counters` 。</span><span class="sxs-lookup"><span data-stu-id="669ff-113">First, install the `dotnet-counters` tool: `dotnet tool install --global dotnet-counters`.</span></span>

<span data-ttu-id="669ff-114">接下来，查找运行 EF Core 应用程序的 .NET 进程的进程 ID (PID) ：</span><span class="sxs-lookup"><span data-stu-id="669ff-114">Next, find the process ID (PID) of the .NET process running your EF Core application:</span></span>

### <a name="windows"></a>[<span data-ttu-id="669ff-115">Windows</span><span class="sxs-lookup"><span data-stu-id="669ff-115">Windows</span></span>](#tab/windows)

1. <span data-ttu-id="669ff-116">右键单击任务栏，然后选择 "任务管理器"，打开 Windows 任务管理器。</span><span class="sxs-lookup"><span data-stu-id="669ff-116">Open the Windows Task Manager by right-clicking on the task bar and selecting "Task Manager".</span></span>
2. <span data-ttu-id="669ff-117">请确保在窗口底部选择 "更多详细信息" 选项。</span><span class="sxs-lookup"><span data-stu-id="669ff-117">Make sure that the "More details" option is selected at the bottom of the window.</span></span>
3. <span data-ttu-id="669ff-118">在 "进程" 选项卡中，右键单击某一列并确保 PID 列已启用。</span><span class="sxs-lookup"><span data-stu-id="669ff-118">In the Processes tab, right-click a column and make sure that the PID column is enabled.</span></span>
4. <span data-ttu-id="669ff-119">在进程列表中找到你的应用程序，并从 PID 列获取其进程 ID。</span><span class="sxs-lookup"><span data-stu-id="669ff-119">Locate your application in the process list, and get its process ID from the PID column.</span></span>

### <a name="linux-or-macos"></a>[<span data-ttu-id="669ff-120">Linux 或 macOS</span><span class="sxs-lookup"><span data-stu-id="669ff-120">Linux or macOS</span></span>](#tab/fluent-api)

1. <span data-ttu-id="669ff-121">使用 `ps` 命令列出你的用户正在运行的所有进程。</span><span class="sxs-lookup"><span data-stu-id="669ff-121">Use the `ps` command to list all processes running for your user.</span></span>
2. <span data-ttu-id="669ff-122">在进程列表中找到你的应用程序，并从 PID 列获取其进程 ID。</span><span class="sxs-lookup"><span data-stu-id="669ff-122">Locate your application in the process list, and get its process ID from the PID column.</span></span>

***

<span data-ttu-id="669ff-123">在 .NET 应用程序中，进程 ID 可用作 `Process.GetCurrentProcess().Id` ; 这对于在启动时打印 PID 非常有用。</span><span class="sxs-lookup"><span data-stu-id="669ff-123">Inside your .NET application, the process ID is available as `Process.GetCurrentProcess().Id`; this can be useful for printing the PID upon startup.</span></span>

<span data-ttu-id="669ff-124">最后， `dotnet-counters` 按如下所示启动：</span><span class="sxs-lookup"><span data-stu-id="669ff-124">Finally, launch `dotnet-counters` as follows:</span></span>

```console
dotnet counters monitor Microsoft.EntityFrameworkCore -p <PID>
```

<span data-ttu-id="669ff-125">`dotnet-counters` 现在会附加到正在运行的进程，并开始报告连续计数器数据：</span><span class="sxs-lookup"><span data-stu-id="669ff-125">`dotnet-counters` will now attach to your running process and start reporting continuous counter data:</span></span>

```console
Press p to pause, r to resume, q to quit.
 Status: Running

[Microsoft.EntityFrameworkCore]
    Active DbContexts                                               1
    Execution Strategy Operation Failures (Count / 1 sec)           0
    Execution Strategy Operation Failures (Total)                   0
    Optimistic Concurrency Failures (Count / 1 sec)                 0
    Optimistic Concurrency Failures (Total)                         0
    Queries (Count / 1 sec)                                         1
    Queries (Total)                                               189
    Query Cache Hit Rate (%)                                      100
    SaveChanges (Count / 1 sec)                                     0
    SaveChanges (Total)                                             0
```

## <a name="counters-and-their-meaning"></a><span data-ttu-id="669ff-126">计数器及其含义</span><span class="sxs-lookup"><span data-stu-id="669ff-126">Counters and their meaning</span></span>

<span data-ttu-id="669ff-127">计数器名称</span><span class="sxs-lookup"><span data-stu-id="669ff-127">Counter name</span></span>                          | <span data-ttu-id="669ff-128">说明</span><span class="sxs-lookup"><span data-stu-id="669ff-128">Description</span></span>
------------------------------------- | ----
<span data-ttu-id="669ff-129">活动 Dbcontext</span><span class="sxs-lookup"><span data-stu-id="669ff-129">Active DbContexts</span></span>                     | <span data-ttu-id="669ff-130">当前应用程序中的活动的未释放 DbContext 实例数。</span><span class="sxs-lookup"><span data-stu-id="669ff-130">The number of active, undisposed DbContext instances currently in your application.</span></span> <span data-ttu-id="669ff-131">如果此数字持续增长，则可能存在泄漏，因为 DbContext 实例没有正确释放。</span><span class="sxs-lookup"><span data-stu-id="669ff-131">If this number grows continuously, you may have a leak because DbContext instances aren't being properly disposed.</span></span> <span data-ttu-id="669ff-132">请注意，如果启用了 [上下文池](xref:core/miscellaneous/context-pooling) ，则此数目包括当前未使用的共用 DbContext 实例。</span><span class="sxs-lookup"><span data-stu-id="669ff-132">Note that if [context pooling](xref:core/miscellaneous/context-pooling) is enabled, this number includes pooled DbContext instances not currently in use.</span></span>
<span data-ttu-id="669ff-133">执行策略操作失败</span><span class="sxs-lookup"><span data-stu-id="669ff-133">Execution Strategy Operation Failures</span></span> | <span data-ttu-id="669ff-134">数据库操作执行失败的次数。</span><span class="sxs-lookup"><span data-stu-id="669ff-134">The number of times a database operation failed to execute.</span></span> <span data-ttu-id="669ff-135">如果启用重试执行策略，则会在同一操作的多次尝试中包括每个单独的故障。</span><span class="sxs-lookup"><span data-stu-id="669ff-135">If a retrying execution strategy is enabled, this includes each individual failure within multiple attempts on the same operation.</span></span> <span data-ttu-id="669ff-136">这可用于检测基础结构的暂时性问题。</span><span class="sxs-lookup"><span data-stu-id="669ff-136">This can be used to detect transient issues with your infrastructure.</span></span>
<span data-ttu-id="669ff-137">开放式并发故障</span><span class="sxs-lookup"><span data-stu-id="669ff-137">Optimistic Concurrency Failures</span></span>       | <span data-ttu-id="669ff-138">`SaveChanges`由于出现了开放式并发错误而失败的次数，因为数据存储区中的数据自代码加载后发生了更改。</span><span class="sxs-lookup"><span data-stu-id="669ff-138">The number of times `SaveChanges` failed because of an optimistic concurrency error, because data in the data store was changed since your code loaded it.</span></span> <span data-ttu-id="669ff-139">这对应于要 <xref:Microsoft.EntityFrameworkCore.DbUpdateConcurrencyException> 引发的。</span><span class="sxs-lookup"><span data-stu-id="669ff-139">This corresponds to a <xref:Microsoft.EntityFrameworkCore.DbUpdateConcurrencyException> being thrown.</span></span>
<span data-ttu-id="669ff-140">查询</span><span class="sxs-lookup"><span data-stu-id="669ff-140">Queries</span></span>                               | <span data-ttu-id="669ff-141">执行的查询数。</span><span class="sxs-lookup"><span data-stu-id="669ff-141">The number of queries executed.</span></span>
<span data-ttu-id="669ff-142">查询缓存命中率 (% ) </span><span class="sxs-lookup"><span data-stu-id="669ff-142">Query Cache Hit Rate (%)</span></span>              | <span data-ttu-id="669ff-143">查询缓存命中与未命中的比率。</span><span class="sxs-lookup"><span data-stu-id="669ff-143">The ratio of query cache hits to misses.</span></span> <span data-ttu-id="669ff-144">第一次执行给定 LINQ 查询时，EF Core (排除) 参数，则必须在相对较高的进程中进行编译。</span><span class="sxs-lookup"><span data-stu-id="669ff-144">The first time a given LINQ query is executed by EF Core (excluding parameters), it must be compiled in what is a relatively heavy process.</span></span> <span data-ttu-id="669ff-145">在普通的应用程序中，将重复使用所有查询，在初始预热期后，查询缓存命中率应该稳定在100%。</span><span class="sxs-lookup"><span data-stu-id="669ff-145">In a normal application, all queries are reused, and the query cache hit rate should be stable at 100% after an initial warmup period.</span></span> <span data-ttu-id="669ff-146">如果此数字在一段时间内小于100%，则可能会由于重复编译导致性能下降，这可能是由于不理想的动态查询生成导致的。</span><span class="sxs-lookup"><span data-stu-id="669ff-146">If this number is less than 100% over time, you may experience degraded perf due to repeated compilations, which could be a result of suboptimal dynamic query generation.</span></span>
<span data-ttu-id="669ff-147">SaveChanges</span><span class="sxs-lookup"><span data-stu-id="669ff-147">SaveChanges</span></span>                           | <span data-ttu-id="669ff-148">已调用的次数 `SaveChanges` 。</span><span class="sxs-lookup"><span data-stu-id="669ff-148">The number of times `SaveChanges` has been called.</span></span> <span data-ttu-id="669ff-149">请注意， `SaveChanges` 在单个批处理中保存多个更改，因此这并不一定表示对单个实体执行的每个更新。</span><span class="sxs-lookup"><span data-stu-id="669ff-149">Note that `SaveChanges` saves multiple changes in a single batch, so this doesn't necessarily represent each individual update done on a single entity.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="669ff-150">其他资源</span><span class="sxs-lookup"><span data-stu-id="669ff-150">Additional resources</span></span>

* [<span data-ttu-id="669ff-151">有关事件计数器的 .NET 文档</span><span class="sxs-lookup"><span data-stu-id="669ff-151">.NET documentation on event counters</span></span>](https://docs.microsoft.com/dotnet/core/diagnostics/event-counters)
