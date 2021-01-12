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
# <a name="asynchronous-programming"></a>异步编程

当在数据库中执行查询时，异步操作将避免阻止线程。 异步操作对于在丰富的客户端应用程序中保持响应式 UI 非常重要，并且还可以增加 web 应用程序中的吞吐量，在这些应用程序中，它们可释放线程以处理 web 应用程序中的其他请求。

遵循 .NET standard 后，EF Core 向执行 i/o 的所有同步方法提供异步对应项。 它们具有与同步方法相同的效果，并且可与 c # `async` 和关键字一起使用 `await` 。 例如，在执行数据库 i/o 时，不使用 DbContext 来阻止线程，而是使用 DbContext。 SaveChangesAsync：

[!code-csharp[Main](../../../samples/core/Miscellaneous/Async/Program.cs#SaveChangesAsync)]

有关详细信息，请参阅 [常规 c # 异步编程文档](/dotnet/csharp/async)。

> [!WARNING]
> EF Core 不支持在同一上下文实例上运行多个并行操作。 应始终等待操作完成，然后再开始下一个操作。 这通常是通过在每个异步操作上使用 `await` 关键字完成的。

> [!WARNING]
> [SqlClient](https://github.com/dotnet/SqlClient)的异步实现中存在一些已知问题 (例如[#593](https://github.com/dotnet/SqlClient/issues/593)、 [#601](https://github.com/dotnet/SqlClient/issues/601)和其他) 。

> [!NOTE]
> EF Core 将取消标记传递给中使用的基础数据库提供程序 (例如 SqlClient) 。 这些令牌可能会也可能不起作用-请参考数据库提供商的文档。

## <a name="async-linq-operators"></a>异步 LINQ 运算符

为了支持异步执行 LINQ 查询，EF Core 提供了一组异步扩展方法，这些方法执行查询并返回结果。 这些对应于标准的同步 LINQ 运算符包括 ToListAsync、SingleAsync、AsAsyncEnumerable 等：

[!code-csharp[Main](../../../samples/core/Miscellaneous/Async/Program.cs#ToListAsync)]

请注意，有些 LINQ 运算符（如 Where 或 OrderBy）没有异步版本，因为这些运算符仅生成 LINQ 表达式树，而不会导致在数据库中执行查询。 只有导致执行查询的运算符具有 async 对应项。

> [!IMPORTANT]
> 在 `Microsoft.EntityFrameworkCore` 命名空间中定义 EF Core 异步扩展方法。 必须导入此命名空间才能使这些方法可用。

## <a name="client-side-async-linq-operators"></a>客户端异步 LINQ 运算符

上面所述的异步 LINQ 运算符只能在 EF 查询上使用-不能将其与客户端 LINQ to Objects 查询一起使用。 若要在 EF 之外执行客户端异步 LINQ 操作，请使用 system.exception [包](https://www.nuget.org/packages/System.Interactive.Async);此包对于在客户端上执行无法在服务器上进行求值的操作时特别有用。

遗憾的是，引用 system.exception 会导致应用于 EF Dbset 的 LINQ 运算符上出现不明确的调用编译错误;这使得难以在同一项目中使用 EF 和 System.object。 若要解决此问题，请将 AsQueryable 添加到 DbSet：

[!code-csharp[Main](../../../samples/core/Miscellaneous/AsyncWithSystemInteractive/Program.cs#SystemInteractiveAsync)]
