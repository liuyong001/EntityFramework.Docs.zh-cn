---
title: 设计时 DbContext 创建-EF Core
description: 使用 Entity Framework Core 创建设计时 DbContext 的策略
author: bricelam
ms.date: 10/27/2020
uid: core/cli/dbcontext-creation
ms.openlocfilehash: 1a2c0e853047cf4ab54a320d0bef413a114e90bc
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543401"
---
# <a name="design-time-dbcontext-creation"></a>设计时 DbContext 创建

某些 EF Core 工具命令 (例如，) [迁移][1] 命令需要在 `DbContext` 设计时创建一个派生实例，以便收集有关该应用程序的实体类型及其如何映射到数据库架构的详细信息。 在大多数情况下，最好 `DbContext` 按照类似的方式配置创建，使其在 [运行时配置][2]。

工具可通过多种方式来创建 `DbContext` ：

## <a name="from-application-services"></a>从应用程序服务

如果启动项目使用 [ASP.NET Core Web 主机][3] 或 [.Net Core 泛型主机][4]，则这些工具将尝试从应用程序的服务提供程序获取 DbContext 对象。

工具首先尝试通过调用 `Program.CreateHostBuilder()` 、调用 `Build()` ，然后访问属性来获取服务提供程序 `Services` 。

[!code-csharp[Main](../../../samples/core/Miscellaneous/CommandLine/ApplicationService.cs#ApplicationService)]

> [!NOTE]
> 创建新的 ASP.NET Core 应用程序时，默认情况下会包括此挂接。

`DbContext`自身及其构造函数中的任何依赖项都需要在应用程序的服务提供程序中注册为服务。 通过在[将 `DbContext` 实例 `DbContextOptions<TContext>` 作为参数][5]并使用[ `AddDbContext<TContext>` 方法][6]的上，可以轻松实现这一点。

## <a name="using-a-constructor-with-no-parameters"></a>使用不带参数的构造函数

如果无法从应用程序服务提供程序获得 DbContext，则工具将查找 `DbContext` 项目中的派生类型。 然后，它们尝试使用不带参数的构造函数创建实例。 如果使用方法配置，则这可能是默认构造函数 `DbContext` [`OnConfiguring`][7] 。

## <a name="from-a-design-time-factory"></a>从设计时工厂

你还可以通过实现接口来告诉工具如何创建 DbContext `IDesignTimeDbContextFactory<TContext>` ：如果实现此接口的类在与派生的项目相同的项目中 `DbContext` 或在应用程序的启动项目中找到，则这些工具将绕过创建 DbContext 的其他方法，并改用设计时工厂。

[!code-csharp[Main](../../../samples/core/Miscellaneous/CommandLine/BloggingContextFactory.cs#BloggingContextFactory)]

> [!NOTE]
> 在 EFCore 5.0 之前， `args` 未使用参数 () ，请参阅 [此问题][8] 。
> 这在 EFCore 5.0 中是固定的，任何其他设计时参数都将通过该参数传递到应用程序。

如果需要在 `DbContext` 设计时与运行时的配置不同， `DbContext` 并且如果构造函数采用其他参数，而不是在 di 中注册，如果根本不使用 di，或者出于某种原因而不是使用 `CreateHostBuilder` ASP.NET Core 应用程序的类中的方法，设计时工厂就特别有用 `Main` 。

## <a name="args"></a>Args

<xref:Microsoft.EntityFrameworkCore.Design.IDesignTimeDbContextFactory%601.CreateDbContext%2A?displayProperty=nameWithType>和均 `Program.CreateHostBuilder` 接受命令行参数。

从 EF Core 5.0 开始，可以从工具中指定以下参数：

### <a name="net-core-cli"></a>[.NET Core CLI](#tab/dotnet-core-cli)

```dotnetcli
dotnet ef database update -- --environment Production
```

该 `--` 令牌指示 `dotnet ef` 将后面的所有内容视为一个参数，而不会尝试将它们作为选项进行分析。 不使用的任何额外参数 `dotnet ef` 都将转发到应用程序。

### <a name="visual-studio"></a>[Visual Studio](#tab/vs)

```powershell
Update-Database -Args '--environment Production'
```

***

  [1]: xref:core/managing-schemas/migrations/index
  [2]: xref:core/dbcontext-configuration/index
  [3]: /aspnet/core/fundamentals/host/web-host
  [4]: /aspnet/core/fundamentals/host/generic-host
  [5]: xref:core/dbcontext-configuration/index#constructor-argument
  [6]: xref:core/dbcontext-configuration/index#using-dbcontext-with-dependency-injection
  [7]: xref:core/dbcontext-configuration/index#onconfiguring
  [8]: https://github.com/dotnet/efcore/issues/8332
