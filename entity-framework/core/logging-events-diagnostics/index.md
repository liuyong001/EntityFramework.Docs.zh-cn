---
title: 日志记录和侦听的概述 - EF Core
description: 概要介绍了 EF Core 的日志记录、事件、侦听器和诊断
author: ajcvickers
ms.date: 10/01/2020
uid: core/logging-events-diagnostics/index
ms.openlocfilehash: 2c44772b22112645f85cf0bffa680bc510ea5afb
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003518"
---
# <a name="overview-of-logging-and-interception"></a><span data-ttu-id="fddc7-103">日志记录和侦听的概述</span><span class="sxs-lookup"><span data-stu-id="fddc7-103">Overview of Logging and Interception</span></span>

<span data-ttu-id="fddc7-104">Entity Framework Core (EF Core) 包含一些用于生成日志、响应事件和获取诊断结果的机制。</span><span class="sxs-lookup"><span data-stu-id="fddc7-104">Entity Framework Core (EF Core) contains several mechanisms for generating logs, responding to events, and obtaining diagnostics.</span></span> <span data-ttu-id="fddc7-105">其中每种机制都针对不同情况进行了定制，务必针对手头上的任务选择最佳机制，即使是多个机制都适用也是如此。</span><span class="sxs-lookup"><span data-stu-id="fddc7-105">Each of these is tailored to different situations, and it is important to select the best mechanism for the task in hand, even when multiple mechanisms could work.</span></span> <span data-ttu-id="fddc7-106">例如，数据库侦听器可用于记录 SQL，但使用针对日志记录定制的机制之一来处理，效果会更好。</span><span class="sxs-lookup"><span data-stu-id="fddc7-106">For example, a database interceptor could be used to log SQL, but this is better handled by one of the mechanisms tailored to logging.</span></span> <span data-ttu-id="fddc7-107">本页面概要介绍了上述每种机制，描述了每种机制的使用场景。</span><span class="sxs-lookup"><span data-stu-id="fddc7-107">This page presents an overview of each of these mechanisms and describes when each should be used.</span></span>

## <a name="quick-reference"></a><span data-ttu-id="fddc7-108">快速参考</span><span class="sxs-lookup"><span data-stu-id="fddc7-108">Quick reference</span></span>

<span data-ttu-id="fddc7-109">下表提供了有关此处所述机制之间的差异的快速参考。</span><span class="sxs-lookup"><span data-stu-id="fddc7-109">The table below provides a quick reference for the differences between the mechanisms described here.</span></span>

| <span data-ttu-id="fddc7-110">机制</span><span class="sxs-lookup"><span data-stu-id="fddc7-110">Mechanism</span></span> |  <span data-ttu-id="fddc7-111">Async</span><span class="sxs-lookup"><span data-stu-id="fddc7-111">Async</span></span> | <span data-ttu-id="fddc7-112">范围</span><span class="sxs-lookup"><span data-stu-id="fddc7-112">Scope</span></span> | <span data-ttu-id="fddc7-113">已注册</span><span class="sxs-lookup"><span data-stu-id="fddc7-113">Registered</span></span> | <span data-ttu-id="fddc7-114">预期用途</span><span class="sxs-lookup"><span data-stu-id="fddc7-114">Intended use</span></span>
|:----------|--------|-------|------------|-------------
| <span data-ttu-id="fddc7-115">简单的日志记录</span><span class="sxs-lookup"><span data-stu-id="fddc7-115">Simple Logging</span></span> | <span data-ttu-id="fddc7-116">否</span><span class="sxs-lookup"><span data-stu-id="fddc7-116">No</span></span> | <span data-ttu-id="fddc7-117">按上下文</span><span class="sxs-lookup"><span data-stu-id="fddc7-117">Per context</span></span> | <span data-ttu-id="fddc7-118">上下文配置</span><span class="sxs-lookup"><span data-stu-id="fddc7-118">Context configuration</span></span> | <span data-ttu-id="fddc7-119">开发时日志记录</span><span class="sxs-lookup"><span data-stu-id="fddc7-119">Development-time logging</span></span>
| <span data-ttu-id="fddc7-120">Microsoft.Extensions.Logging</span><span class="sxs-lookup"><span data-stu-id="fddc7-120">Microsoft.Extensions.Logging</span></span> | <span data-ttu-id="fddc7-121">否</span><span class="sxs-lookup"><span data-stu-id="fddc7-121">No</span></span> | <span data-ttu-id="fddc7-122">按上下文\*</span><span class="sxs-lookup"><span data-stu-id="fddc7-122">Per context\*</span></span> | <span data-ttu-id="fddc7-123">D.I.</span><span class="sxs-lookup"><span data-stu-id="fddc7-123">D.I.</span></span> <span data-ttu-id="fddc7-124">或上下文配置</span><span class="sxs-lookup"><span data-stu-id="fddc7-124">or context configuration</span></span> | <span data-ttu-id="fddc7-125">生产日志记录</span><span class="sxs-lookup"><span data-stu-id="fddc7-125">Production logging</span></span>
| <span data-ttu-id="fddc7-126">事件</span><span class="sxs-lookup"><span data-stu-id="fddc7-126">Events</span></span> | <span data-ttu-id="fddc7-127">否</span><span class="sxs-lookup"><span data-stu-id="fddc7-127">No</span></span> | <span data-ttu-id="fddc7-128">按上下文</span><span class="sxs-lookup"><span data-stu-id="fddc7-128">Per context</span></span> | <span data-ttu-id="fddc7-129">任何时间</span><span class="sxs-lookup"><span data-stu-id="fddc7-129">Any time</span></span> | <span data-ttu-id="fddc7-130">对 EF 事件做出反应</span><span class="sxs-lookup"><span data-stu-id="fddc7-130">Reacting to EF events</span></span>
| <span data-ttu-id="fddc7-131">拦截器</span><span class="sxs-lookup"><span data-stu-id="fddc7-131">Interceptors</span></span> | <span data-ttu-id="fddc7-132">是</span><span class="sxs-lookup"><span data-stu-id="fddc7-132">Yes</span></span> | <span data-ttu-id="fddc7-133">按上下文</span><span class="sxs-lookup"><span data-stu-id="fddc7-133">Per context</span></span> | <span data-ttu-id="fddc7-134">上下文配置</span><span class="sxs-lookup"><span data-stu-id="fddc7-134">Context configuration</span></span> | <span data-ttu-id="fddc7-135">进行 EF 操作</span><span class="sxs-lookup"><span data-stu-id="fddc7-135">Manipulating EF operations</span></span>
| <span data-ttu-id="fddc7-136">诊断侦听器</span><span class="sxs-lookup"><span data-stu-id="fddc7-136">Diagnostics listeners</span></span> | <span data-ttu-id="fddc7-137">否</span><span class="sxs-lookup"><span data-stu-id="fddc7-137">No</span></span> | <span data-ttu-id="fddc7-138">进程</span><span class="sxs-lookup"><span data-stu-id="fddc7-138">Process</span></span> | <span data-ttu-id="fddc7-139">全局</span><span class="sxs-lookup"><span data-stu-id="fddc7-139">Globally</span></span> | <span data-ttu-id="fddc7-140">应用程序诊断</span><span class="sxs-lookup"><span data-stu-id="fddc7-140">Application diagnostics</span></span>

<span data-ttu-id="fddc7-141">\*通常，`Microsoft.Extensions.Logging` 通过依赖项注入按应用程序进行配置，但是，在 EF 级别，可以根据需要使用不同的记录器配置每个上下文。</span><span class="sxs-lookup"><span data-stu-id="fddc7-141">\*Typically `Microsoft.Extensions.Logging` is configured per-application via dependency injection However, at the EF level, each context _can_ be configured with a different logger if needed.</span></span>

## <a name="simple-logging"></a><span data-ttu-id="fddc7-142">简单的日志记录</span><span class="sxs-lookup"><span data-stu-id="fddc7-142">Simple logging</span></span>

> [!NOTE]
> <span data-ttu-id="fddc7-143">EF Core 5.0 中已引入此功能。</span><span class="sxs-lookup"><span data-stu-id="fddc7-143">This feature was introduced in EF Core 5.0.</span></span>

<span data-ttu-id="fddc7-144">可以通过使用 [LogTo](https://github.com/dotnet/efcore/blob/ec3df8fd7e4ea4ebeebfa747619cef37b23ab2c6/src/EFCore/DbContextOptionsBuilder.cs#L135) 从任意类型的应用程序访问 EF Core 日志</span><span class="sxs-lookup"><span data-stu-id="fddc7-144">EF Core logs can be accessed from any type of application through the use of [LogTo](https://github.com/dotnet/efcore/blob/ec3df8fd7e4ea4ebeebfa747619cef37b23ab2c6/src/EFCore/DbContextOptionsBuilder.cs#L135)</span></span> <!-- Issue #2748 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.LogTo%2A> --> <span data-ttu-id="fddc7-145">[配置 DbContext 实例](xref:core/dbcontext-configuration/index)时。</span><span class="sxs-lookup"><span data-stu-id="fddc7-145">when [configuring a DbContext instance](xref:core/dbcontext-configuration/index).</span></span> <span data-ttu-id="fddc7-146">此配置通常通过替代 <xref:Microsoft.EntityFrameworkCore.DbContext.OnConfiguring%2A?displayProperty=nameWithType> 来完成。</span><span class="sxs-lookup"><span data-stu-id="fddc7-146">This configuration is commonly done in an override of <xref:Microsoft.EntityFrameworkCore.DbContext.OnConfiguring%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="fddc7-147">例如：</span><span class="sxs-lookup"><span data-stu-id="fddc7-147">For example:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(Console.WriteLine);
-->
[!code-csharp[LogToConsole](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=LogToConsole)]

<span data-ttu-id="fddc7-148">此概念与 EF6 中的 <xref:System.Data.Entity.Database.Log?displayProperty=nameWithType> 相类似。</span><span class="sxs-lookup"><span data-stu-id="fddc7-148">This concept is similar to <xref:System.Data.Entity.Database.Log?displayProperty=nameWithType> in EF6.</span></span>

<span data-ttu-id="fddc7-149">有关详细信息，请参阅[简单日志记录](xref:core/logging-events-diagnostics/simple-logging)。</span><span class="sxs-lookup"><span data-stu-id="fddc7-149">See [Simple Logging](xref:core/logging-events-diagnostics/simple-logging) for more information.</span></span>

## <a name="microsoftextensionslogging"></a><span data-ttu-id="fddc7-150">Microsoft.Extensions.Logging</span><span class="sxs-lookup"><span data-stu-id="fddc7-150">Microsoft.Extensions.Logging</span></span>

<span data-ttu-id="fddc7-151">[Microsoft.Extensions.Logging](/dotnet/core/extensions/logging) 是一种可扩展的日志记录机制，适用于很多常见日志记录系统的插件提供程序。</span><span class="sxs-lookup"><span data-stu-id="fddc7-151">[Microsoft.Extensions.Logging](/dotnet/core/extensions/logging) is an extensible logging mechanism with plug-in providers for many common logging systems.</span></span> <span data-ttu-id="fddc7-152">EF Core 与 `Microsoft.Extensions.Logging` 完全集成，并且 ASP.NET Core 应用程序默认使用这种形式的日志记录。</span><span class="sxs-lookup"><span data-stu-id="fddc7-152">EF Core fully integrates with `Microsoft.Extensions.Logging` and this form of logging is used by default for ASP.NET Core applications.</span></span>

<span data-ttu-id="fddc7-153">有关详细信息，请参阅[使用 EF Core 中的 Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging)。</span><span class="sxs-lookup"><span data-stu-id="fddc7-153">See [Using Microsoft.Extensions.Logging in EF Core](xref:core/logging-events-diagnostics/extensions-logging) for more information.</span></span>

## <a name="events"></a><span data-ttu-id="fddc7-154">事件</span><span class="sxs-lookup"><span data-stu-id="fddc7-154">Events</span></span>

> [!NOTE]
> <span data-ttu-id="fddc7-155">EF Core 5.0 中引入了其他事件。</span><span class="sxs-lookup"><span data-stu-id="fddc7-155">Additional events were introduced in EF Core 5.0.</span></span>

<span data-ttu-id="fddc7-156">当 EF Core 代码中发生某些事情时，EF Core 公开 [.NET 事件](/dotnet/standard/events/)以用作回调。</span><span class="sxs-lookup"><span data-stu-id="fddc7-156">EF Core exposes [.NET events](/dotnet/standard/events/) to act as callbacks when certain things happen in the EF Core code.</span></span> <span data-ttu-id="fddc7-157">事件比侦听器简单，并且允许更灵活的注册。</span><span class="sxs-lookup"><span data-stu-id="fddc7-157">Events are simpler than interceptors and allow more flexible registration.</span></span> <span data-ttu-id="fddc7-158">但是，它们只是同步，因此不能执行非阻塞异步 I/O。</span><span class="sxs-lookup"><span data-stu-id="fddc7-158">However, they are sync only and so cannot perform non-blocking async I/O.</span></span>

<span data-ttu-id="fddc7-159">每个 DbContext 实例都会注册事件，并且可以随时进行此注册。</span><span class="sxs-lookup"><span data-stu-id="fddc7-159">Events are registered per DbContext instance and this registration can be done at any time.</span></span> <span data-ttu-id="fddc7-160">使用[诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)获取相同的信息，但要获取进程中所有 DbContext 实例的信息。</span><span class="sxs-lookup"><span data-stu-id="fddc7-160">Use a [diagnostic listener](xref:core/logging-events-diagnostics/diagnostic-listeners) to get the same information but for all DbContext instances in the process.</span></span>

<span data-ttu-id="fddc7-161">有关详细信息，请参阅 [EF Core 中的 .NET 事件](xref:core/logging-events-diagnostics/events)。</span><span class="sxs-lookup"><span data-stu-id="fddc7-161">See [.NET Events in EF Core](xref:core/logging-events-diagnostics/events) for more information.</span></span>

## <a name="interception"></a><span data-ttu-id="fddc7-162">Interception</span><span class="sxs-lookup"><span data-stu-id="fddc7-162">Interception</span></span>

> [!NOTE]
> <span data-ttu-id="fddc7-163">EF Core 3.0 中已引入此功能。</span><span class="sxs-lookup"><span data-stu-id="fddc7-163">This feature was introduced in EF Core 3.0.</span></span> <span data-ttu-id="fddc7-164">EF Core 5.0 中引入了其他侦听器。</span><span class="sxs-lookup"><span data-stu-id="fddc7-164">Additional interceptors were introduced in EF Core 5.0.</span></span>

<span data-ttu-id="fddc7-165">EF Core 侦听器支持 EF Core 操作的侦听、修改和/或抑制。</span><span class="sxs-lookup"><span data-stu-id="fddc7-165">EF Core interceptors enable interception, modification, and/or suppression of EF Core operations.</span></span> <span data-ttu-id="fddc7-166">这包括低级数据库操作（例如执行命令）以及高级别操作（例如对 SaveChanges 的调用）。</span><span class="sxs-lookup"><span data-stu-id="fddc7-166">This includes low-level database operations such as executing a command, as well as higher-level operations, such as calls to SaveChanges.</span></span>

<span data-ttu-id="fddc7-167">侦听器与日志记录和诊断的不同之处在于，它们允许修改或抑制所侦听的操作。</span><span class="sxs-lookup"><span data-stu-id="fddc7-167">Interceptors are different from logging and diagnostics in that they allow modification or suppression of the operation being intercepted.</span></span> <span data-ttu-id="fddc7-168">[简单日志记录](xref:core/logging-events-diagnostics/simple-logging)或 [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) 是日志记录的更好选择。</span><span class="sxs-lookup"><span data-stu-id="fddc7-168">[Simple logging](xref:core/logging-events-diagnostics/simple-logging) or [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) are better choices for logging.</span></span>

<span data-ttu-id="fddc7-169">配置上下文时，将按 DbContext 实例注册侦听器。</span><span class="sxs-lookup"><span data-stu-id="fddc7-169">Interceptors are registered per DbContext instance when the context is configured.</span></span> <span data-ttu-id="fddc7-170">使用[诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)获取相同的信息，但要获取进程中所有 DbContext 实例的信息。</span><span class="sxs-lookup"><span data-stu-id="fddc7-170">Use a [diagnostic listener](xref:core/logging-events-diagnostics/diagnostic-listeners) to get the same information but for all DbContext instances in the process.</span></span>

<span data-ttu-id="fddc7-171">有关详细信息，请参阅[侦听](xref:core/logging-events-diagnostics/interceptors)。</span><span class="sxs-lookup"><span data-stu-id="fddc7-171">See [Interception](xref:core/logging-events-diagnostics/interceptors) for more information.</span></span>

## <a name="diagnostic-listeners"></a><span data-ttu-id="fddc7-172">诊断侦听器</span><span class="sxs-lookup"><span data-stu-id="fddc7-172">Diagnostic listeners</span></span>

<span data-ttu-id="fddc7-173">诊断侦听器允许侦听当前 .NET 进程中发生的任何 EF Core 事件。</span><span class="sxs-lookup"><span data-stu-id="fddc7-173">Diagnostic listeners allow listening for any EF Core event that occurs in the current .NET process.</span></span>

<span data-ttu-id="fddc7-174">诊断侦听器不适合从单个 DbContext 实例获取事件。</span><span class="sxs-lookup"><span data-stu-id="fddc7-174">Diagnostic listeners are not suitable for getting events from a single DbContext instance.</span></span> <span data-ttu-id="fddc7-175">EF Core 侦听器使用按个上下文注册提供对相同事件的访问。</span><span class="sxs-lookup"><span data-stu-id="fddc7-175">EF Core interceptors provide access to the same events with per-context registration.</span></span>

<span data-ttu-id="fddc7-176">诊断侦听器不能用于日志记录。</span><span class="sxs-lookup"><span data-stu-id="fddc7-176">Diagnostic listeners are not designed for logging.</span></span> <span data-ttu-id="fddc7-177">[简单日志记录](xref:core/logging-events-diagnostics/simple-logging)或 [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) 是日志记录的更好选择。</span><span class="sxs-lookup"><span data-stu-id="fddc7-177">[Simple logging](xref:core/logging-events-diagnostics/simple-logging) or [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) are better choices for logging.</span></span>

<span data-ttu-id="fddc7-178">有关详细信息，请参阅[使用 EF Core 中的诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)。</span><span class="sxs-lookup"><span data-stu-id="fddc7-178">See [Using diagnostic listeners in EF Core](xref:core/logging-events-diagnostics/diagnostic-listeners) for more information.</span></span>
