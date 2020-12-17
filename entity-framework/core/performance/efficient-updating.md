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
# <a name="efficient-updating"></a><span data-ttu-id="1d1a8-103">有效更新</span><span class="sxs-lookup"><span data-stu-id="1d1a8-103">Efficient Updating</span></span>

## <a name="batching"></a><span data-ttu-id="1d1a8-104">批处理</span><span class="sxs-lookup"><span data-stu-id="1d1a8-104">Batching</span></span>

<span data-ttu-id="1d1a8-105">EF Core 通过在单个往返中自动对所有更新进行批处理，有助于最大程度地减少往返次数。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-105">EF Core helps minimize roundtrips by automatically batching together all updates in a single roundtrip.</span></span> <span data-ttu-id="1d1a8-106">考虑以下情况：</span><span class="sxs-lookup"><span data-stu-id="1d1a8-106">Consider the following:</span></span>

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#SaveChangesBatching)]

<span data-ttu-id="1d1a8-107">上面的内容从数据库中加载博客，更改其名称，然后添加两个新博客：若要应用此语句，请将两个 SQL INSERT 语句和一个 UPDATE 语句发送到数据库。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-107">The above loads a blog from the database, changes its name, and then adds two new blogs; to apply this, two SQL INSERT statements and one UPDATE statement are sent to the database.</span></span> <span data-ttu-id="1d1a8-108">不是逐个发送，而是添加博客实例，EF Core 在内部跟踪这些更改，并在调用时在单次往返中执行这些更改 <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> 。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-108">Rather than sending them one by one, as Blog instances are added, EF Core tracks these changes internally, and executes them in a single roundtrip when <xref:Microsoft.EntityFrameworkCore.DbContext.SaveChanges%2A> is called.</span></span>

<span data-ttu-id="1d1a8-109">EF 在单次往返中分批批处理的语句数取决于所使用的数据库提供程序。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-109">The number of statements that EF batches in a single roundtrip depends on the database provider being used.</span></span> <span data-ttu-id="1d1a8-110">例如，在涉及到4条以上的语句时，性能分析显示了对 SQL Server 效率较低的批处理。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-110">For example, performance analysis has shown batching to be generally less efficient for SQL Server when less than 4 statements are involved.</span></span> <span data-ttu-id="1d1a8-111">同样，在 SQL Server 的40语句之后，批处理的优点会下降，因此 EF Core 默认情况下，仅在单个批处理中执行最多42条语句，并在单独的往返中执行其他语句。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-111">Similarly, the benefits of batching degrade after around 40 statements for SQL Server, so EF Core will by default only execute up to 42 statements in a single batch, and execute additional statements in separate roundtrips.</span></span>

<span data-ttu-id="1d1a8-112">用户还可以调整这些阈值以实现可能更高的性能，但在修改它们之前应认真进行基准测试：</span><span class="sxs-lookup"><span data-stu-id="1d1a8-112">Users can also tweak these thresholds to achieve potentially higher performance - but benchmark carefully before modifying these:</span></span>

[!code-csharp[Main](../../../samples/core/Performance/BatchTweakingContext.cs#BatchTweaking)]

## <a name="bulk-updates"></a><span data-ttu-id="1d1a8-113">大容量更新</span><span class="sxs-lookup"><span data-stu-id="1d1a8-113">Bulk updates</span></span>

<span data-ttu-id="1d1a8-114">假设你想要为所有员工带来一个举手。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-114">Let's assume you want to give all your employees a raise.</span></span> <span data-ttu-id="1d1a8-115">EF Core 的典型实现如下所示：</span><span class="sxs-lookup"><span data-stu-id="1d1a8-115">A typical implementation for this in EF Core would look like the following:</span></span>

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#UpdateWithoutBulk)]

<span data-ttu-id="1d1a8-116">虽然这是非常有效的代码，但让我们从性能角度分析这一点：</span><span class="sxs-lookup"><span data-stu-id="1d1a8-116">While this is perfectly valid code, let's analyze what it does from a performance perspective:</span></span>

* <span data-ttu-id="1d1a8-117">执行数据库往返，以加载所有相关员工;请注意，这会将所有员工的行数据引入客户端，即使只需要薪金。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-117">A database roundtrip is performed, to load all the relevant employees; note that this brings all the Employees' row data to the client, even if only the salary will be needed.</span></span>
* <span data-ttu-id="1d1a8-118">EF Core 的更改跟踪会在加载实体时创建快照，然后将这些快照与实例进行比较，以确定更改了哪些属性。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-118">EF Core's change tracking creates snapshots when loading the entities, and then compares those snapshots to the instances to find out which properties changed.</span></span>
* <span data-ttu-id="1d1a8-119">执行第二个数据库往返以保存所有更改。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-119">A second database roundtrip is performed to save all the changes.</span></span> <span data-ttu-id="1d1a8-120">尽管所有更改都是在单次往返中完成的，但由于进行了批处理，因此 EF Core 仍将为每个雇员发送一个 UPDATE 语句，必须由数据库执行。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-120">While all changes are done in a single roundtrip thanks to batching, EF Core still sends an UPDATE statement per employee, which must be executed by the database.</span></span>

<span data-ttu-id="1d1a8-121">关系数据库还支持 *大容量更新*，因此可以将上面的单个 SQL 语句重写为以下形式：</span><span class="sxs-lookup"><span data-stu-id="1d1a8-121">Relational databases also support *bulk updates*, so the above could be rewritten as the following single SQL statement:</span></span>

```sql
UPDATE [Employees] SET [Salary] = [Salary] + 1000;
```

<span data-ttu-id="1d1a8-122">这会在单个往返过程中执行整个操作，而不会将任何实际数据加载或发送到数据库，并且不使用 EF 的更改跟踪机制，这会产生额外的开销。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-122">This performs the entire operation in a single roundtrip, without loading or sending any actual data to the database, and without making use of EF's change tracking machinery, which imposes an additional overhead.</span></span>

<span data-ttu-id="1d1a8-123">遗憾的是，EF 当前不提供用于执行大容量更新的 Api。</span><span class="sxs-lookup"><span data-stu-id="1d1a8-123">Unfortunately, EF doesn't currently provide APIs for performing bulk updates.</span></span> <span data-ttu-id="1d1a8-124">在引入这些内容之前，可以使用原始 SQL 来执行性能敏感的操作：</span><span class="sxs-lookup"><span data-stu-id="1d1a8-124">Until these are introduced, you can use raw SQL to perform the operation where performance is sensitive:</span></span>

[!code-csharp[Main](../../../samples/core/Performance/Program.cs#UpdateWithBulk)]
