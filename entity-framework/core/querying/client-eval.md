---
title: 客户端与服务器评估 - EF Core
description: 使用 Entity Framework Core 进行客户端和服务器查询评估
author: smitpatel
ms.date: 11/09/2020
uid: core/querying/client-eval
ms.openlocfilehash: a1ddfb625be36cb05f01da08eb3be29512c54ab5
ms.sourcegitcommit: f3512e3a98e685a3ba409c1d0157ce85cc390cf4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2020
ms.locfileid: "94430139"
---
# <a name="client-vs-server-evaluation"></a>客户端与服务器评估

作为一般规则，Entity Framework Core 会尝试尽可能全面地评估服务器上的查询。 EF Core 将查询的一部分转换为可在客户端评估的参数。 系统将查询的其余部分（及生成的参数）提供给数据库提供程序，以确定要在服务器上评估的等效数据库查询。 EF Core 支持在顶级投影中进行部分客户端评估（基本上为最后一次调用 `Select()`）。 如果查询中的顶级投影无法转换为服务器，EF Core 将从服务器中提取任何所需的数据，并在客户端上评估查询的其余部分。 如果 EF Core 在顶级投影之外的任何位置检测到不能转换为服务器的表达式，则会引发运行时异常。 请参阅[查询工作原理](xref:core/querying/how-query-works)，了解 EF Core 如何确定哪些表达式无法转换为服务器。

> [!NOTE]
> 在 3.0 版之前，Entity Framework Core 支持在查询中的任何位置进行客户端评估。 有关详细信息，请参阅[历史版本部分](#previous-versions)。

> [!TIP]
> 可在 GitHub 上查看此文章的[示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Querying/ClientEvaluation)。

## <a name="client-evaluation-in-the-top-level-projection"></a>顶级投影中的客户端评估

在下面的示例中，一个辅助方法用于标准化从 SQL Server 数据库中返回的博客的 URL。 由于 SQL Server 提供程序不了解此方法的实现方式，因此无法将其转换为 SQL。 查询的所有其余部分是在数据库中评估的，但通过此方法传递返回的 `URL` 却是在客户端上完成。

[!code-csharp[Main](../../../samples/core/Querying/ClientEvaluation/Program.cs#ClientProjection)]

[!code-csharp[Main](../../../samples/core/Querying/ClientEvaluation/Program.cs#ClientMethod)]

## <a name="unsupported-client-evaluation"></a>不支持的客户端评估

尽管客户端评估非常有用，但有时会减弱性能。 请看以下查询，其中的 where 筛选器现已使用辅助方法。 由于数据库中不能应用筛选器，因此需要将所有数据提取到内存中，以便在客户端上应用筛选器。 根据服务器上的筛选器和数据量，客户端评估可能会减弱性能。 因此 Entity Framework Core 会阻止此类客户端评估，并引发运行时异常。

[!code-csharp[Main](../../../samples/core/Querying/ClientEvaluation/Program.cs#ClientWhere)]

## <a name="explicit-client-evaluation"></a>显式客户端评估

在某些情况下，可能需要以显式方式强制进行客户端评估，如下所示

- 由于数据量小，因此在进行客户端评估时才不会大幅减弱性能。
- 所用的 LINQ 运算符不会进行任何服务器端转换。

在这种情况下，通过调用 `AsEnumerable` 或 `ToList` 等方法（若为异步，则调用 `AsAsyncEnumerable` 或 `ToListAsync`），以显式方式选择进行客户端评估。 使用 `AsEnumerable` 将对结果进行流式传输，但使用 `ToList` 将通过创建列表来进行缓冲，因此也会占用额外的内存。 但如果枚举多次，则将结果存储到列表中可以带来更大的帮助，因为只有一个对数据库的查询。 根据具体的使用情况，你应该评估哪种方法更适合。

[!code-csharp[Main](../../../samples/core/Querying/ClientEvaluation/Program.cs#ExplicitClientEvaluation)]

> [!TIP]
> 如果你正在使用 `AsAsyncEnumerable` 并希望在客户端进一步编写查询，可以使用用于定义异步枚举运算符的 [System.Interactive.Async](https://www.nuget.org/packages/System.Interactive.Async/) 库。 有关详细信息，请参阅[客户端 linq 运算符](xref:core/miscellaneous/async#client-side-async-linq-operators)。

## <a name="potential-memory-leak-in-client-evaluation"></a>客户端评估中潜在的内存泄漏

由于查询转换和编译的开销高昂，因此 EF Core 会缓存已编译的查询计划。 缓存的委托在对顶级投影进行客户端评估时可能会使用客户端代码。 EF Core 为树型结构中客户端评估的部分生成参数，并通过替换参数值重用查询计划。 但表达式树中的某些常数无法转换为参数。 如果缓存的委托包含此类常数，则无法将这些对象垃圾回收，因为它们仍被引用。 如果此类对象包含 DbContext 或其中的其他服务，则会导致应用的内存使用量逐渐增多。 此行为通常是内存泄漏的标志。 只要遇到的常数为不能使用当前数据库提供程序映射的类型，EF Core 就会引发异常。 常见原因及其解决方案如下所示：

- **使用实例方法**：在客户端投影中使用实例方法时，表达式树包含实例的常数。 如果你的方法不使用该实例中的任何数据，请考虑将该方法设为静态方法。 如果需要方法主体中的实例数据，则将特定数据作为实参传递给方法。
- **将常数实参传递给方法**：这种情况通常是由于在客户端方法的实参中使用 `this` 引起的。 请考虑将实参拆分为多个标量实参，可由数据库提供程序进行映射。
- **其他常数**：如果在任何其他情况下都出现常数，则可以评估在处理过程中是否需要该常数。 如果必须具有常数，或者如果无法使用上述情况中的解决方案，则创建本地变量来存储值，并在查询中使用局部变量。 EF Core 会将局部变量转换为形参。

## <a name="previous-versions"></a>早期版本

以下部分适用于 3.0 以前的 EF Core 版本。

旧的 EF Core 版本支持在查询的任何部分中进行客户端评估，而不仅仅是顶级投影。 因此，与[不支持的客户评估](#unsupported-client-evaluation)部分下发布的查询类似的查询可以正常工作。 由于此行为可能引起不易觉察的性能问题，EF Core 记录了客户端评估警告。 有关如何查看日志记录输出的详细信息，请参阅[日志记录](xref:core/logging-events-diagnostics/index)。

（可选）借助 EF Core，你可以将默认行为更改为在执行客户端评估时引发异常或不执行任何操作（在投影中除外）。 引发异常的行为会使其类似于 3.0 中的行为。 若要更改该行为，你需要在设置上下文选项时配置警告。上下文选项一般在 `DbContext.OnConfiguring` 中设置，如果使用 ASP.NET Core，则在 `Startup.cs` 中设置。

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=EFQuerying;Trusted_Connection=True;")
        .ConfigureWarnings(warnings => warnings.Throw(RelationalEventId.QueryClientEvaluationWarning));
}
```
