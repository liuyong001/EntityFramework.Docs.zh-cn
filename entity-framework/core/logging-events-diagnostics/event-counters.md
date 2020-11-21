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
# <a name="event-counters"></a>事件计数器

> [!NOTE]
> EF Core 5.0 中添加了此功能。

Entity Framework Core (EF Core) 会公开连续数值指标，这可以提供程序健康状况的良好指示。 这些指标可用于以下目的：

* 在应用程序运行时实时跟踪常规数据库加载
* 公开可能导致性能下降的问题编码做法
* 跟踪和隔离异常程序行为

通过标准 .NET 事件计数器功能 EF Core 报告指标;建议阅读 [此博客文章](https://devblogs.microsoft.com/dotnet/introducing-diagnostics-improvements-in-net-core-3-0/) ，了解计数器如何工作的简要概述。

## <a name="attach-to-a-process-using-dotnet-counters"></a>使用 dotnet 附加到进程-计数器

[Dotnet 工具](https://docs.microsoft.com/dotnet/core/diagnostics/dotnet-counters)可用于附加到正在运行的进程，并定期报告 EF Core 事件计数器;无需在程序中执行任何特殊操作即可使用这些计数器。

首先，安装 `dotnet-counters` 工具： `dotnet tool install --global dotnet-counters` 。

接下来，查找运行 EF Core 应用程序的 .NET 进程的进程 ID (PID) ：

### <a name="windows"></a>[Windows](#tab/windows)

1. 右键单击任务栏，然后选择 "任务管理器"，打开 Windows 任务管理器。
2. 请确保在窗口底部选择 "更多详细信息" 选项。
3. 在 "进程" 选项卡中，右键单击某一列并确保 PID 列已启用。
4. 在进程列表中找到你的应用程序，并从 PID 列获取其进程 ID。

### <a name="linux-or-macos"></a>[Linux 或 macOS](#tab/fluent-api)

1. 使用 `ps` 命令列出你的用户正在运行的所有进程。
2. 在进程列表中找到你的应用程序，并从 PID 列获取其进程 ID。

***

在 .NET 应用程序中，进程 ID 可用作 `Process.GetCurrentProcess().Id` ; 这对于在启动时打印 PID 非常有用。

最后， `dotnet-counters` 按如下所示启动：

```console
dotnet counters monitor Microsoft.EntityFrameworkCore -p <PID>
```

`dotnet-counters` 现在会附加到正在运行的进程，并开始报告连续计数器数据：

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

## <a name="counters-and-their-meaning"></a>计数器及其含义

计数器名称                          | 说明
------------------------------------- | ----
活动 Dbcontext                     | 当前应用程序中的活动的未释放 DbContext 实例数。 如果此数字持续增长，则可能存在泄漏，因为 DbContext 实例没有正确释放。 请注意，如果启用了 [上下文池](xref:core/miscellaneous/context-pooling) ，则此数目包括当前未使用的共用 DbContext 实例。
执行策略操作失败 | 数据库操作执行失败的次数。 如果启用重试执行策略，则会在同一操作的多次尝试中包括每个单独的故障。 这可用于检测基础结构的暂时性问题。
开放式并发故障       | `SaveChanges`由于出现了开放式并发错误而失败的次数，因为数据存储区中的数据自代码加载后发生了更改。 这对应于要 <xref:Microsoft.EntityFrameworkCore.DbUpdateConcurrencyException> 引发的。
查询                               | 执行的查询数。
查询缓存命中率 (% )               | 查询缓存命中与未命中的比率。 第一次执行给定 LINQ 查询时，EF Core (排除) 参数，则必须在相对较高的进程中进行编译。 在普通的应用程序中，将重复使用所有查询，在初始预热期后，查询缓存命中率应该稳定在100%。 如果此数字在一段时间内小于100%，则可能会由于重复编译导致性能下降，这可能是由于不理想的动态查询生成导致的。
SaveChanges                           | 已调用的次数 `SaveChanges` 。 请注意， `SaveChanges` 在单个批处理中保存多个更改，因此这并不一定表示对单个实体执行的每个更新。

## <a name="additional-resources"></a>其他资源

* [有关事件计数器的 .NET 文档](https://docs.microsoft.com/dotnet/core/diagnostics/event-counters)
