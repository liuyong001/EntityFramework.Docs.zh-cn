---
title: 日志记录和侦听的概述 - EF Core
description: 概要介绍了 EF Core 的日志记录、事件、侦听器和诊断
author: ajcvickers
ms.date: 10/01/2020
uid: core/logging-events-diagnostics/index
ms.openlocfilehash: 5ddbffc8d39e97065f2e06af14443c62b4a9465d
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129221"
---
# <a name="overview-of-logging-and-interception"></a>日志记录和侦听的概述

Entity Framework Core (EF Core) 包含一些用于生成日志、响应事件和获取诊断结果的机制。 其中每种机制都针对不同情况进行了定制，务必针对手头上的任务选择最佳机制，即使是多个机制都适用也是如此。 例如，数据库侦听器可用于记录 SQL，但使用针对日志记录定制的机制之一来处理，效果会更好。 本页面概要介绍了上述每种机制，描述了每种机制的使用场景。

## <a name="quick-reference"></a>快速参考

下表提供了有关此处所述机制之间的差异的快速参考。

| 机制 |  Async | 范围 | 已注册 | 预期用途
|:----------|--------|-------|------------|-------------
| 简单的日志记录 | 否 | 按上下文 | 上下文配置 | 开发时日志记录
| Microsoft.Extensions.Logging | 否 | 按上下文* | D.I. 或上下文配置 | 生产日志记录
| 事件 | 否 | 按上下文 | 任何时间 | 对 EF 事件做出反应
| 拦截器 | 是 | 按上下文 | 上下文配置 | 进行 EF 操作
| 诊断侦听器 | 否 | 进程 | 全局 | 应用程序诊断

*通常，`Microsoft.Extensions.Logging` 通过依赖项注入按应用程序进行配置，但是，在 EF 级别，可以根据需要使用不同的记录器配置每个上下文。

## <a name="simple-logging"></a>简单的日志记录

> [!NOTE]
> EF Core 5.0 中已引入此功能。

可以在[配置 DbContext 实例](xref:core/dbcontext-configuration/index)时使用 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.LogTo%2A> 从任意类型的应用程序访问 EF Core 日志。 此配置通常通过替代 <xref:Microsoft.EntityFrameworkCore.DbContext.OnConfiguring%2A?displayProperty=nameWithType> 来完成。 例如：

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(Console.WriteLine);
-->
[!code-csharp[LogToConsole](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=LogToConsole)]

此概念与 EF6 中的 <xref:System.Data.Entity.Database.Log?displayProperty=nameWithType> 相类似。

有关详细信息，请参阅[简单日志记录](xref:core/logging-events-diagnostics/simple-logging)。

## <a name="microsoftextensionslogging"></a>Microsoft.Extensions.Logging

[Microsoft.Extensions.Logging](/dotnet/core/extensions/logging) 是一种可扩展的日志记录机制，适用于很多常见日志记录系统的插件提供程序。 EF Core 与 `Microsoft.Extensions.Logging` 完全集成，并且 ASP.NET Core 应用程序默认使用这种形式的日志记录。

有关详细信息，请参阅[使用 EF Core 中的 Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging)。

## <a name="events"></a>事件

> [!NOTE]
> EF Core 5.0 中引入了其他事件。

当 EF Core 代码中发生某些事情时，EF Core 公开 [.NET 事件](/dotnet/standard/events/)以用作回调。 事件比侦听器简单，并且允许更灵活的注册。 但是，它们只是同步，因此不能执行非阻塞异步 I/O。

每个 DbContext 实例都会注册事件，并且可以随时进行此注册。 使用[诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)获取相同的信息，但要获取进程中所有 DbContext 实例的信息。

有关详细信息，请参阅 [EF Core 中的 .NET 事件](xref:core/logging-events-diagnostics/events)。

## <a name="interception"></a>Interception

> [!NOTE]
> EF Core 3.0 中已引入此功能。 EF Core 5.0 中引入了其他侦听器。

EF Core 侦听器支持 EF Core 操作的侦听、修改和/或抑制。 这包括低级数据库操作（例如执行命令）以及高级别操作（例如对 SaveChanges 的调用）。

侦听器与日志记录和诊断的不同之处在于，它们允许修改或抑制所侦听的操作。 [简单日志记录](xref:core/logging-events-diagnostics/simple-logging)或 [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) 是日志记录的更好选择。

配置上下文时，将按 DbContext 实例注册侦听器。 使用[诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)获取相同的信息，但要获取进程中所有 DbContext 实例的信息。

有关详细信息，请参阅[侦听](xref:core/logging-events-diagnostics/interceptors)。

## <a name="diagnostic-listeners"></a>诊断侦听器

诊断侦听器允许侦听当前 .NET 进程中发生的任何 EF Core 事件。

诊断侦听器不适合从单个 DbContext 实例获取事件。 EF Core 侦听器使用按个上下文注册提供对相同事件的访问。

诊断侦听器不能用于日志记录。 [简单日志记录](xref:core/logging-events-diagnostics/simple-logging)或 [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) 是日志记录的更好选择。

有关详细信息，请参阅[使用 EF Core 中的诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)。
