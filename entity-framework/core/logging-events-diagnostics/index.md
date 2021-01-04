---
title: 日志记录和侦听的概述 - EF Core
description: 概要介绍了 EF Core 的日志记录、事件、侦听器和诊断
author: ajcvickers
ms.date: 10/01/2020
uid: core/logging-events-diagnostics/index
ms.openlocfilehash: d85a506167661523bf70b62d3a075a6248180d11
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97635674"
---
# <a name="overview-of-logging-and-interception"></a><span data-ttu-id="4c031-103">日志记录和侦听的概述</span><span class="sxs-lookup"><span data-stu-id="4c031-103">Overview of Logging and Interception</span></span>

<span data-ttu-id="4c031-104">Entity Framework Core (EF Core) 包含一些用于生成日志、响应事件和获取诊断结果的机制。</span><span class="sxs-lookup"><span data-stu-id="4c031-104">Entity Framework Core (EF Core) contains several mechanisms for generating logs, responding to events, and obtaining diagnostics.</span></span> <span data-ttu-id="4c031-105">其中每种机制都针对不同情况进行了定制，务必针对手头上的任务选择最佳机制，即使是多个机制都适用也是如此。</span><span class="sxs-lookup"><span data-stu-id="4c031-105">Each of these is tailored to different situations, and it is important to select the best mechanism for the task in hand, even when multiple mechanisms could work.</span></span> <span data-ttu-id="4c031-106">例如，数据库侦听器可用于记录 SQL，但使用针对日志记录定制的机制之一来处理，效果会更好。</span><span class="sxs-lookup"><span data-stu-id="4c031-106">For example, a database interceptor could be used to log SQL, but this is better handled by one of the mechanisms tailored to logging.</span></span> <span data-ttu-id="4c031-107">本页面概要介绍了上述每种机制，描述了每种机制的使用场景。</span><span class="sxs-lookup"><span data-stu-id="4c031-107">This page presents an overview of each of these mechanisms and describes when each should be used.</span></span>

## <a name="quick-reference"></a><span data-ttu-id="4c031-108">快速参考</span><span class="sxs-lookup"><span data-stu-id="4c031-108">Quick reference</span></span>

<span data-ttu-id="4c031-109">下表提供了有关此处所述机制之间的差异的快速参考。</span><span class="sxs-lookup"><span data-stu-id="4c031-109">The table below provides a quick reference for the differences between the mechanisms described here.</span></span>

| <span data-ttu-id="4c031-110">机制</span><span class="sxs-lookup"><span data-stu-id="4c031-110">Mechanism</span></span> |  <span data-ttu-id="4c031-111">Async</span><span class="sxs-lookup"><span data-stu-id="4c031-111">Async</span></span> | <span data-ttu-id="4c031-112">范围</span><span class="sxs-lookup"><span data-stu-id="4c031-112">Scope</span></span> | <span data-ttu-id="4c031-113">已注册</span><span class="sxs-lookup"><span data-stu-id="4c031-113">Registered</span></span> | <span data-ttu-id="4c031-114">预期用途</span><span class="sxs-lookup"><span data-stu-id="4c031-114">Intended use</span></span>
|:----------|--------|-------|------------|-------------
| <span data-ttu-id="4c031-115">简单的日志记录</span><span class="sxs-lookup"><span data-stu-id="4c031-115">Simple Logging</span></span> | <span data-ttu-id="4c031-116">否</span><span class="sxs-lookup"><span data-stu-id="4c031-116">No</span></span> | <span data-ttu-id="4c031-117">按上下文</span><span class="sxs-lookup"><span data-stu-id="4c031-117">Per context</span></span> | <span data-ttu-id="4c031-118">上下文配置</span><span class="sxs-lookup"><span data-stu-id="4c031-118">Context configuration</span></span> | <span data-ttu-id="4c031-119">开发时日志记录</span><span class="sxs-lookup"><span data-stu-id="4c031-119">Development-time logging</span></span>
| <span data-ttu-id="4c031-120">Microsoft.Extensions.Logging</span><span class="sxs-lookup"><span data-stu-id="4c031-120">Microsoft.Extensions.Logging</span></span> | <span data-ttu-id="4c031-121">否</span><span class="sxs-lookup"><span data-stu-id="4c031-121">No</span></span> | <span data-ttu-id="4c031-122">按上下文\*</span><span class="sxs-lookup"><span data-stu-id="4c031-122">Per context\*</span></span> | <span data-ttu-id="4c031-123">D.I.</span><span class="sxs-lookup"><span data-stu-id="4c031-123">D.I.</span></span> <span data-ttu-id="4c031-124">或上下文配置</span><span class="sxs-lookup"><span data-stu-id="4c031-124">or context configuration</span></span> | <span data-ttu-id="4c031-125">生产日志记录</span><span class="sxs-lookup"><span data-stu-id="4c031-125">Production logging</span></span>
| <span data-ttu-id="4c031-126">事件</span><span class="sxs-lookup"><span data-stu-id="4c031-126">Events</span></span> | <span data-ttu-id="4c031-127">否</span><span class="sxs-lookup"><span data-stu-id="4c031-127">No</span></span> | <span data-ttu-id="4c031-128">按上下文</span><span class="sxs-lookup"><span data-stu-id="4c031-128">Per context</span></span> | <span data-ttu-id="4c031-129">任何时间</span><span class="sxs-lookup"><span data-stu-id="4c031-129">Any time</span></span> | <span data-ttu-id="4c031-130">对 EF 事件做出反应</span><span class="sxs-lookup"><span data-stu-id="4c031-130">Reacting to EF events</span></span>
| <span data-ttu-id="4c031-131">拦截器</span><span class="sxs-lookup"><span data-stu-id="4c031-131">Interceptors</span></span> | <span data-ttu-id="4c031-132">是</span><span class="sxs-lookup"><span data-stu-id="4c031-132">Yes</span></span> | <span data-ttu-id="4c031-133">按上下文</span><span class="sxs-lookup"><span data-stu-id="4c031-133">Per context</span></span> | <span data-ttu-id="4c031-134">上下文配置</span><span class="sxs-lookup"><span data-stu-id="4c031-134">Context configuration</span></span> | <span data-ttu-id="4c031-135">进行 EF 操作</span><span class="sxs-lookup"><span data-stu-id="4c031-135">Manipulating EF operations</span></span>
| <span data-ttu-id="4c031-136">诊断侦听器</span><span class="sxs-lookup"><span data-stu-id="4c031-136">Diagnostics listeners</span></span> | <span data-ttu-id="4c031-137">否</span><span class="sxs-lookup"><span data-stu-id="4c031-137">No</span></span> | <span data-ttu-id="4c031-138">进程</span><span class="sxs-lookup"><span data-stu-id="4c031-138">Process</span></span> | <span data-ttu-id="4c031-139">全局</span><span class="sxs-lookup"><span data-stu-id="4c031-139">Globally</span></span> | <span data-ttu-id="4c031-140">应用程序诊断</span><span class="sxs-lookup"><span data-stu-id="4c031-140">Application diagnostics</span></span>

<span data-ttu-id="4c031-141">\*通常，`Microsoft.Extensions.Logging` 通过依赖项注入按应用程序进行配置，但是，在 EF 级别，可以根据需要使用不同的记录器配置每个上下文。</span><span class="sxs-lookup"><span data-stu-id="4c031-141">\*Typically `Microsoft.Extensions.Logging` is configured per-application via dependency injection However, at the EF level, each context _can_ be configured with a different logger if needed.</span></span>

## <a name="simple-logging"></a><span data-ttu-id="4c031-142">简单的日志记录</span><span class="sxs-lookup"><span data-stu-id="4c031-142">Simple logging</span></span>

> [!NOTE]
> <span data-ttu-id="4c031-143">EF Core 5.0 中已引入此功能。</span><span class="sxs-lookup"><span data-stu-id="4c031-143">This feature was introduced in EF Core 5.0.</span></span>

<span data-ttu-id="4c031-144">可以在[配置 DbContext 实例](xref:core/dbcontext-configuration/index)时使用 <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.LogTo%2A> 从任意类型的应用程序访问 EF Core 日志。</span><span class="sxs-lookup"><span data-stu-id="4c031-144">EF Core logs can be accessed from any type of application through the use of <xref:Microsoft.EntityFrameworkCore.DbContextOptionsBuilder.LogTo%2A> when [configuring a DbContext instance](xref:core/dbcontext-configuration/index).</span></span> <span data-ttu-id="4c031-145">此配置通常通过替代 <xref:Microsoft.EntityFrameworkCore.DbContext.OnConfiguring%2A?displayProperty=nameWithType> 来完成。</span><span class="sxs-lookup"><span data-stu-id="4c031-145">This configuration is commonly done in an override of <xref:Microsoft.EntityFrameworkCore.DbContext.OnConfiguring%2A?displayProperty=nameWithType>.</span></span> <span data-ttu-id="4c031-146">例如：</span><span class="sxs-lookup"><span data-stu-id="4c031-146">For example:</span></span>

<!--
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.LogTo(Console.WriteLine);
-->
[!code-csharp[LogToConsole](../../../samples/core/Miscellaneous/Logging/SimpleLogging/Program.cs?name=LogToConsole)]

<span data-ttu-id="4c031-147">此概念与 EF6 中的 <xref:System.Data.Entity.Database.Log?displayProperty=nameWithType> 相类似。</span><span class="sxs-lookup"><span data-stu-id="4c031-147">This concept is similar to <xref:System.Data.Entity.Database.Log?displayProperty=nameWithType> in EF6.</span></span>

<span data-ttu-id="4c031-148">有关详细信息，请参阅[简单日志记录](xref:core/logging-events-diagnostics/simple-logging)。</span><span class="sxs-lookup"><span data-stu-id="4c031-148">See [Simple Logging](xref:core/logging-events-diagnostics/simple-logging) for more information.</span></span>

## <a name="microsoftextensionslogging"></a><span data-ttu-id="4c031-149">Microsoft.Extensions.Logging</span><span class="sxs-lookup"><span data-stu-id="4c031-149">Microsoft.Extensions.Logging</span></span>

<span data-ttu-id="4c031-150">[Microsoft.Extensions.Logging](/dotnet/core/extensions/logging) 是一种可扩展的日志记录机制，适用于很多常见日志记录系统的插件提供程序。</span><span class="sxs-lookup"><span data-stu-id="4c031-150">[Microsoft.Extensions.Logging](/dotnet/core/extensions/logging) is an extensible logging mechanism with plug-in providers for many common logging systems.</span></span> <span data-ttu-id="4c031-151">EF Core 与 `Microsoft.Extensions.Logging` 完全集成，并且 ASP.NET Core 应用程序默认使用这种形式的日志记录。</span><span class="sxs-lookup"><span data-stu-id="4c031-151">EF Core fully integrates with `Microsoft.Extensions.Logging` and this form of logging is used by default for ASP.NET Core applications.</span></span>

<span data-ttu-id="4c031-152">有关详细信息，请参阅[使用 EF Core 中的 Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging)。</span><span class="sxs-lookup"><span data-stu-id="4c031-152">See [Using Microsoft.Extensions.Logging in EF Core](xref:core/logging-events-diagnostics/extensions-logging) for more information.</span></span>

## <a name="events"></a><span data-ttu-id="4c031-153">事件</span><span class="sxs-lookup"><span data-stu-id="4c031-153">Events</span></span>

> [!NOTE]
> <span data-ttu-id="4c031-154">EF Core 5.0 中引入了其他事件。</span><span class="sxs-lookup"><span data-stu-id="4c031-154">Additional events were introduced in EF Core 5.0.</span></span>

<span data-ttu-id="4c031-155">当 EF Core 代码中发生某些事情时，EF Core 公开 [.NET 事件](/dotnet/standard/events/)以用作回调。</span><span class="sxs-lookup"><span data-stu-id="4c031-155">EF Core exposes [.NET events](/dotnet/standard/events/) to act as callbacks when certain things happen in the EF Core code.</span></span> <span data-ttu-id="4c031-156">事件比侦听器简单，并且允许更灵活的注册。</span><span class="sxs-lookup"><span data-stu-id="4c031-156">Events are simpler than interceptors and allow more flexible registration.</span></span> <span data-ttu-id="4c031-157">但是，它们只是同步，因此不能执行非阻塞异步 I/O。</span><span class="sxs-lookup"><span data-stu-id="4c031-157">However, they are sync only and so cannot perform non-blocking async I/O.</span></span>

<span data-ttu-id="4c031-158">每个 DbContext 实例都会注册事件，并且可以随时进行此注册。</span><span class="sxs-lookup"><span data-stu-id="4c031-158">Events are registered per DbContext instance and this registration can be done at any time.</span></span> <span data-ttu-id="4c031-159">使用[诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)获取相同的信息，但要获取进程中所有 DbContext 实例的信息。</span><span class="sxs-lookup"><span data-stu-id="4c031-159">Use a [diagnostic listener](xref:core/logging-events-diagnostics/diagnostic-listeners) to get the same information but for all DbContext instances in the process.</span></span>

<span data-ttu-id="4c031-160">有关详细信息，请参阅 [EF Core 中的 .NET 事件](xref:core/logging-events-diagnostics/events)。</span><span class="sxs-lookup"><span data-stu-id="4c031-160">See [.NET Events in EF Core](xref:core/logging-events-diagnostics/events) for more information.</span></span>

## <a name="interception"></a><span data-ttu-id="4c031-161">Interception</span><span class="sxs-lookup"><span data-stu-id="4c031-161">Interception</span></span>

> [!NOTE]
> <span data-ttu-id="4c031-162">EF Core 3.0 中已引入此功能。</span><span class="sxs-lookup"><span data-stu-id="4c031-162">This feature was introduced in EF Core 3.0.</span></span> <span data-ttu-id="4c031-163">EF Core 5.0 中引入了其他侦听器。</span><span class="sxs-lookup"><span data-stu-id="4c031-163">Additional interceptors were introduced in EF Core 5.0.</span></span>

<span data-ttu-id="4c031-164">EF Core 侦听器支持 EF Core 操作的侦听、修改和/或抑制。</span><span class="sxs-lookup"><span data-stu-id="4c031-164">EF Core interceptors enable interception, modification, and/or suppression of EF Core operations.</span></span> <span data-ttu-id="4c031-165">这包括低级数据库操作（例如执行命令）以及高级别操作（例如对 SaveChanges 的调用）。</span><span class="sxs-lookup"><span data-stu-id="4c031-165">This includes low-level database operations such as executing a command, as well as higher-level operations, such as calls to SaveChanges.</span></span>

<span data-ttu-id="4c031-166">侦听器与日志记录和诊断的不同之处在于，它们允许修改或抑制所侦听的操作。</span><span class="sxs-lookup"><span data-stu-id="4c031-166">Interceptors are different from logging and diagnostics in that they allow modification or suppression of the operation being intercepted.</span></span> <span data-ttu-id="4c031-167">[简单日志记录](xref:core/logging-events-diagnostics/simple-logging)或 [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) 是日志记录的更好选择。</span><span class="sxs-lookup"><span data-stu-id="4c031-167">[Simple logging](xref:core/logging-events-diagnostics/simple-logging) or [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) are better choices for logging.</span></span>

<span data-ttu-id="4c031-168">配置上下文时，将按 DbContext 实例注册侦听器。</span><span class="sxs-lookup"><span data-stu-id="4c031-168">Interceptors are registered per DbContext instance when the context is configured.</span></span> <span data-ttu-id="4c031-169">使用[诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)获取相同的信息，但要获取进程中所有 DbContext 实例的信息。</span><span class="sxs-lookup"><span data-stu-id="4c031-169">Use a [diagnostic listener](xref:core/logging-events-diagnostics/diagnostic-listeners) to get the same information but for all DbContext instances in the process.</span></span>

<span data-ttu-id="4c031-170">有关详细信息，请参阅[侦听](xref:core/logging-events-diagnostics/interceptors)。</span><span class="sxs-lookup"><span data-stu-id="4c031-170">See [Interception](xref:core/logging-events-diagnostics/interceptors) for more information.</span></span>

## <a name="diagnostic-listeners"></a><span data-ttu-id="4c031-171">诊断侦听器</span><span class="sxs-lookup"><span data-stu-id="4c031-171">Diagnostic listeners</span></span>

<span data-ttu-id="4c031-172">诊断侦听器允许侦听当前 .NET 进程中发生的任何 EF Core 事件。</span><span class="sxs-lookup"><span data-stu-id="4c031-172">Diagnostic listeners allow listening for any EF Core event that occurs in the current .NET process.</span></span>

<span data-ttu-id="4c031-173">诊断侦听器不适合从单个 DbContext 实例获取事件。</span><span class="sxs-lookup"><span data-stu-id="4c031-173">Diagnostic listeners are not suitable for getting events from a single DbContext instance.</span></span> <span data-ttu-id="4c031-174">EF Core 侦听器使用按个上下文注册提供对相同事件的访问。</span><span class="sxs-lookup"><span data-stu-id="4c031-174">EF Core interceptors provide access to the same events with per-context registration.</span></span>

<span data-ttu-id="4c031-175">诊断侦听器不能用于日志记录。</span><span class="sxs-lookup"><span data-stu-id="4c031-175">Diagnostic listeners are not designed for logging.</span></span> <span data-ttu-id="4c031-176">[简单日志记录](xref:core/logging-events-diagnostics/simple-logging)或 [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) 是日志记录的更好选择。</span><span class="sxs-lookup"><span data-stu-id="4c031-176">[Simple logging](xref:core/logging-events-diagnostics/simple-logging) or [Microsoft.Extensions.Logging](xref:core/logging-events-diagnostics/extensions-logging) are better choices for logging.</span></span>

<span data-ttu-id="4c031-177">有关详细信息，请参阅[使用 EF Core 中的诊断侦听器](xref:core/logging-events-diagnostics/diagnostic-listeners)。</span><span class="sxs-lookup"><span data-stu-id="4c031-177">See [Using diagnostic listeners in EF Core](xref:core/logging-events-diagnostics/diagnostic-listeners) for more information.</span></span>
