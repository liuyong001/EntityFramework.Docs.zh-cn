---
title: 高效更新-EF Core
description: 使用 Entity Framework Core 进行高效更新的性能指南
author: roji
ms.date: 12/1/2020
uid: core/performance/efficient-updating
ms.openlocfilehash: 92766d2339fb04ed5ebc3123429171cc9be424b1
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97657716"
---
# <a name="efficient-updating"></a>有效更新

## <a name="batching"></a>批处理

EF Core 通过在单个往返中自动对所有更新进行批处理，有助于最大程度地减少往返次数。 考虑以下情况：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#SaveChangesBatching)]

上面的内容从数据库中加载博客，更改其名称，然后添加两个新博客：若要应用此语句，请将两个 SQL INSERT 语句和一个 UPDATE 语句发送到数据库。 不是逐个发送，而是添加博客实例，EF Core 在内部跟踪这些更改，并在调用时在单次往返中执行这些更改 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。

EF 在单次往返中分批批处理的语句数取决于所使用的数据库提供程序。 例如，在涉及到4条以上的语句时，性能分析显示了对 SQL Server 效率较低的批处理。 同样，在 SQL Server 的40语句之后，批处理的优点会下降，因此 EF Core 默认情况下，仅在单个批处理中执行最多42条语句，并在单独的往返中执行其他语句。

用户还可以调整这些阈值以实现可能更高的性能，但在修改它们之前应认真进行基准测试：

[!code-csharp[Main](../../../samples/core/Performance/BatchTweakingContext.cs#BatchTweaking)]

## <a name="bulk-updates"></a>大容量更新

假设你想要为所有员工带来一个举手。 EF Core 的典型实现如下所示：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#UpdateWithoutBulk)]

虽然这是非常有效的代码，但让我们从性能角度分析这一点：

* 执行数据库往返，以加载所有相关员工;请注意，这会将所有员工的行数据引入客户端，即使只需要薪金。
* EF Core 的更改跟踪会在加载实体时创建快照，然后将这些快照与实例进行比较，以确定更改了哪些属性。
* 执行第二个数据库往返以保存所有更改。 尽管所有更改都是在单次往返中完成的，但由于进行了批处理，因此 EF Core 仍将为每个雇员发送一个 UPDATE 语句，必须由数据库执行。

关系数据库还支持 *大容量更新*，因此可以将上面的单个 SQL 语句重写为以下形式：

```sql
UPDATE [Employees] SET [Salary] = [Salary] + 1000;
```

这会在单个往返过程中执行整个操作，而不会将任何实际数据加载或发送到数据库，并且不使用 EF 的更改跟踪机制，这会产生额外的开销。

遗憾的是，EF 当前不提供用于执行大容量更新的 Api。 在引入这些内容之前，可以使用原始 SQL 来执行性能敏感的操作：

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#UpdateWithBulk)]
