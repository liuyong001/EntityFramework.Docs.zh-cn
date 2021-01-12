---
title: 使用诊断侦听器-EF Core
description: 使用 DiagnosticListener 进行 EF Core 诊断的全局使用
author: ajcvickers
ms.date: 10/16/2020
uid: core/logging-events-diagnostics/diagnostic-listeners
ms.openlocfilehash: afb80aa8f05f70761e423f58653f681938079858
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129260"
---
# <a name="using-diagnostic-listeners-in-ef-core"></a>在 EF Core 中使用诊断侦听器

> [!TIP]
> 可以从 GitHub [下载此文章的示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/DiagnosticListeners) 。

诊断侦听器允许侦听当前 .NET 进程中发生的任何 EF Core 事件。 <xref:System.Diagnostics.DiagnosticListener>类是一种[跨 .net 的通用机制](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md)的组成部分，用于从运行的应用程序中获取诊断信息。

诊断侦听器不适合从单个 DbContext 实例获取事件。 EF Core [拦截](xref:core/logging-events-diagnostics/interceptors) 程序使用每个上下文注册提供对相同事件的访问。

诊断侦听器不能用于日志记录。 请考虑使用 [简单日志记录](xref:core/logging-events-diagnostics/simple-logging) 或用于日志记录 [的日志记录](xref:core/logging-events-diagnostics/extensions-logging) 。

## <a name="example-observing-diagnostic-events"></a>示例：观察诊断事件

解析 EF Core 事件的过程分为两个步骤。 首先， [](/dotnet/standard/events/observer-design-pattern) `DiagnosticListener` 必须创建其自身的观察程序：

<!--
public class DiagnosticObserver : IObserver<DiagnosticListener>
{
    public void OnCompleted()
        => throw new NotImplementedException();

    public void OnError(Exception error)
        => throw new NotImplementedException();

    public void OnNext(DiagnosticListener value)
    {
        if (value.Name == DbLoggerCategory.Name) // "Microsoft.EntityFrameworkCore"
        {
            value.Subscribe(new KeyValueObserver());
        }
    }
}
-->
[!code-csharp[DiagnosticObserver](../../../samples/core/Miscellaneous/DiagnosticListeners/Program.cs?name=DiagnosticObserver)]

`OnNext`方法查找来自 EF Core 的 DiagnosticListener。 此侦听器的名称为 "Microsoft.entityframeworkcore"，可从类中获得， <xref:Microsoft.EntityFrameworkCore.DbLoggerCategory> 如所示。

然后，必须在应用程序的方法中全局注册此观察程序 `Main` ：

<!--
        DiagnosticListener.AllListeners.Subscribe(new DiagnosticObserver());
-->
[!code-csharp[RegisterDiagnosticListener](../../../samples/core/Miscellaneous/DiagnosticListeners/Program.cs?name=RegisterDiagnosticListener)]

其次，一旦找到 EF Core DiagnosticListener，就会创建一个新的键-值观察器来订阅实际 EF Core 事件。 例如：

<!--
public class KeyValueObserver : IObserver<KeyValuePair<string, object>>
{
    public void OnCompleted()
        => throw new NotImplementedException();

    public void OnError(Exception error)
        => throw new NotImplementedException();

    public void OnNext(KeyValuePair<string, object> value)
    {
        if (value.Key == CoreEventId.ContextInitialized.Name)
        {
            var payload = (ContextInitializedEventData)value.Value;
            Console.WriteLine($"EF is initializing {payload.Context.GetType().Name} ");
        }

        if (value.Key == RelationalEventId.ConnectionOpening.Name)
        {
            var payload = (ConnectionEventData)value.Value;
            Console.WriteLine($"EF is opening a connection to {payload.Connection.ConnectionString} ");
        }
    }
}
-->
[!code-csharp[KeyValueObserver](../../../samples/core/Miscellaneous/DiagnosticListeners/Program.cs?name=KeyValueObserver)]

`OnNext`此方法是使用每个 EF Core 事件的键/值对来调用的。 密钥是事件的名称，可以从以下项之一获得：

* <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId> 对于所有 EF Core 数据库提供程序所共有的事件
* <xref:Microsoft.EntityFrameworkCore.Diagnostics.RelationalEventId> 对于所有关系数据库提供程序通用的事件
* 特定于当前数据库提供程序的事件的类似类。 例如， <xref:Microsoft.EntityFrameworkCore.Diagnostics.SqlServerEventId> 对于 SQL Server 提供程序。

键/值对的值是特定于事件的负载类型。 预期的负载类型在这些事件类中定义的每个事件上记录。

例如，上面的代码处理 <xref:Microsoft.EntityFrameworkCore.Diagnostics.CoreEventId.ContextInitialized> 和 <xref:Microsoft.EntityFrameworkCore.Diagnostics.RelationalEventId.ConnectionOpening> 事件。 对于其中的第一种，负载是 <xref:Microsoft.EntityFrameworkCore.Diagnostics.ContextInitializedEventData> 。 对于第二个，它是 <xref:Microsoft.EntityFrameworkCore.Diagnostics.ConnectionEventData> 。

> [!TIP]
> 在每个 EF Core 事件数据类中重写 ToString，以生成事件的等效日志消息。 例如，调用会 `ContextInitializedEventData.ToString` 生成 "Entity Framework Core 5.0.0，并使用提供程序" microsoft.entityframeworkcore "（选项为" 无 "）初始化" BlogsContext "。

该 [示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/DiagnosticListeners) 包含一个简单的控制台应用程序，该应用程序对博客数据库进行更改并输出遇到的诊断事件。

<!--
    public static void Main()
    {
        #region RegisterDiagnosticListener
        DiagnosticListener.AllListeners.Subscribe(new DiagnosticObserver());
        #endregion

        using (var context = new BlogsContext())
        {
            context.Database.EnsureDeleted();
            context.Database.EnsureCreated();

            context.Add(
                new Blog
                {
                    Name = "EF Blog",
                    Posts =
                    {
                        new Post { Title = "EF Core 3.1!" },
                        new Post { Title = "EF Core 5.0!" }
                    }
                });

            context.SaveChanges();
        }

        using (var context = new BlogsContext())
        {
            var blog = context.Blogs.Include(e => e.Posts).Single();

            blog.Name = "EF Core Blog";
            context.Remove(blog.Posts.First());
            blog.Posts.Add(new Post { Title = "EF Core 6.0!" });

            context.SaveChanges();
        }
        #endregion
    }
-->
[!code-csharp[Program](../../../samples/core/Miscellaneous/DiagnosticListeners/Program.cs?name=Program)]

此代码的输出显示检测到的事件：

```output
EF is initializing BlogsContext
EF is opening a connection to Data Source=blogs.db;Mode=ReadOnly
EF is opening a connection to DataSource=blogs.db
EF is opening a connection to Data Source=blogs.db;Mode=ReadOnly
EF is opening a connection to DataSource=blogs.db
EF is opening a connection to DataSource=blogs.db
EF is opening a connection to DataSource=blogs.db
EF is initializing BlogsContext
EF is opening a connection to DataSource=blogs.db
EF is opening a connection to DataSource=blogs.db
```
