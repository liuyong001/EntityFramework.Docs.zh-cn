---
title: 查询的工作原理 - EF Core
description: 关于 Entity Framework Core 如何在内部编译和执行查询的常规信息
author: ajcvickers
ms.date: 03/17/2020
uid: core/querying/how-query-works
ms.openlocfilehash: 7b3014cf64f8467ccbec10598ea1bb47304dfe43
ms.sourcegitcommit: f3512e3a98e685a3ba409c1d0157ce85cc390cf4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2020
ms.locfileid: "94430464"
---
# <a name="how-queries-work"></a><span data-ttu-id="e9351-103">查询的工作原理</span><span class="sxs-lookup"><span data-stu-id="e9351-103">How Queries Work</span></span>

<span data-ttu-id="e9351-104">Entity Framework Core 使用语言集成查询 (LINQ) 来查询数据库中的数据。</span><span class="sxs-lookup"><span data-stu-id="e9351-104">Entity Framework Core uses Language Integrated Query (LINQ) to query data from the database.</span></span> <span data-ttu-id="e9351-105">通过 LINQ 可使用 C#（或你选择的 .NET 语言）基于派生上下文和实体类编写强类型查询。</span><span class="sxs-lookup"><span data-stu-id="e9351-105">LINQ allows you to use C# (or your .NET language of choice) to write strongly typed queries based on your derived context and entity classes.</span></span>

> [!NOTE]
> <span data-ttu-id="e9351-106">本文已过时，其中的某些部分需要更新，以解释查询管道设计中发生的更改。</span><span class="sxs-lookup"><span data-stu-id="e9351-106">This article is out of date and some parts of it needs to be updated to account for changes happened in design of query pipeline.</span></span> <span data-ttu-id="e9351-107">如果对此处提到的任何行为有任何疑问，请[提问](https://github.com/dotnet/efcore/issues/new/choose)。</span><span class="sxs-lookup"><span data-stu-id="e9351-107">If you have any doubts about any behavior mentioned here, please [ask a question](https://github.com/dotnet/efcore/issues/new/choose).</span></span>

## <a name="the-life-of-a-query"></a><span data-ttu-id="e9351-108">查询的寿命</span><span class="sxs-lookup"><span data-stu-id="e9351-108">The life of a query</span></span>

<span data-ttu-id="e9351-109">下面的描述是每个查询所经历的过程的综合概述。</span><span class="sxs-lookup"><span data-stu-id="e9351-109">The following description is a high-level overview of the process each query goes through.</span></span>

1. <span data-ttu-id="e9351-110">LINQ 查询由 Entity Framework Core 处理，用于生成已准备好由数据库提供程序处理的表示形式</span><span class="sxs-lookup"><span data-stu-id="e9351-110">The LINQ query is processed by Entity Framework Core to build a representation that is ready to be processed by the database provider</span></span>
   1. <span data-ttu-id="e9351-111">结果会被缓存，以便每次执行查询时无需进行此处理</span><span class="sxs-lookup"><span data-stu-id="e9351-111">The result is cached so that this processing does not need to be done every time the query is executed</span></span>
2. <span data-ttu-id="e9351-112">结果会传递到数据库提供程序</span><span class="sxs-lookup"><span data-stu-id="e9351-112">The result is passed to the database provider</span></span>
   1. <span data-ttu-id="e9351-113">数据库提供程序确定可以在数据库中评估查询的哪些部分</span><span class="sxs-lookup"><span data-stu-id="e9351-113">The database provider identifies which parts of the query can be evaluated in the database</span></span>
   2. <span data-ttu-id="e9351-114">查询的这些部分会转换为特定数据库的查询语言（例如关系数据库的 SQL）</span><span class="sxs-lookup"><span data-stu-id="e9351-114">These parts of the query are translated to database-specific query language (for example, SQL for a relational database)</span></span>
   3. <span data-ttu-id="e9351-115">查询会发送到数据库并返回结果集（返回的是数据库中的值，而不是实体实例中的）</span><span class="sxs-lookup"><span data-stu-id="e9351-115">A query is sent to the database and the result set returned (results are values from the database, not entity instances)</span></span>
3. <span data-ttu-id="e9351-116">对于结果集中的每一项</span><span class="sxs-lookup"><span data-stu-id="e9351-116">For each item in the result set</span></span>
   1. <span data-ttu-id="e9351-117">如果该查询是跟踪查询，EF 会检查数据是否表示上下文实例内更改跟踪器中的现有实体</span><span class="sxs-lookup"><span data-stu-id="e9351-117">If the query is a tracking query, EF checks if the data represents an entity already in the change tracker for the context instance</span></span>
      * <span data-ttu-id="e9351-118">如果是，则会返回现有实体</span><span class="sxs-lookup"><span data-stu-id="e9351-118">If so, the existing entity is returned</span></span>
      * <span data-ttu-id="e9351-119">如果不是，则会创建新实体、设置更改跟踪并返回该新实体</span><span class="sxs-lookup"><span data-stu-id="e9351-119">If not, a new entity is created, change tracking is set up, and the new entity is returned</span></span>
   2. <span data-ttu-id="e9351-120">如果该查询是非跟踪查询，将始终创建并返回新实体</span><span class="sxs-lookup"><span data-stu-id="e9351-120">If the query is a no-tracking query, then a new entity is always created and returned</span></span>

## <a name="when-queries-are-executed"></a><span data-ttu-id="e9351-121">执行查询时</span><span class="sxs-lookup"><span data-stu-id="e9351-121">When queries are executed</span></span>

<span data-ttu-id="e9351-122">调用 LINQ 运算符时，只会构建查询在内存中的表示形式。</span><span class="sxs-lookup"><span data-stu-id="e9351-122">When you call LINQ operators, you're simply building up an in-memory representation of the query.</span></span> <span data-ttu-id="e9351-123">使用结果时，查询只会发送到数据库。</span><span class="sxs-lookup"><span data-stu-id="e9351-123">The query is only sent to the database when the results are consumed.</span></span>

<span data-ttu-id="e9351-124">导致查询发送到数据库的最常见操作如下：</span><span class="sxs-lookup"><span data-stu-id="e9351-124">The most common operations that result in the query being sent to the database are:</span></span>

* <span data-ttu-id="e9351-125">在 `for` 循环中循环访问结果</span><span class="sxs-lookup"><span data-stu-id="e9351-125">Iterating the results in a `for` loop</span></span>
* <span data-ttu-id="e9351-126">使用 `ToList`、`ToArray`、`Single`、`Count` 等操作或等效的异步重载</span><span class="sxs-lookup"><span data-stu-id="e9351-126">Using an operator such as `ToList`, `ToArray`, `Single`, `Count`, or the equivalent async overloads</span></span>

> [!WARNING]  
> <span data-ttu-id="e9351-127">始终验证用户输入：虽然 EF Core通过在查询中使用参数和转义文字来防止 SQL 注入攻击，但它不会验证输入。</span><span class="sxs-lookup"><span data-stu-id="e9351-127">**Always validate user input:** While EF Core protects against SQL injection attacks by using parameters and escaping literals in queries, it does not validate inputs.</span></span> <span data-ttu-id="e9351-128">根据应用程序的要求，在将 LINQ 查询中使用的来自不受信任的源的值分配给实体属性或传递给其他 EF Core API 之前，应执行相应的验证。</span><span class="sxs-lookup"><span data-stu-id="e9351-128">Appropriate validation, per the application's requirements, should be performed before values from un-trusted sources are used in LINQ queries, assigned to entity properties, or passed to other EF Core APIs.</span></span> <span data-ttu-id="e9351-129">这包括用于动态构造查询的所有用户输入。</span><span class="sxs-lookup"><span data-stu-id="e9351-129">This includes any user input used to dynamically construct queries.</span></span> <span data-ttu-id="e9351-130">即使在使用 LINQ 时，如果接受用于生成表达式的用户输入，也会需要确保只能构造预期表达式。</span><span class="sxs-lookup"><span data-stu-id="e9351-130">Even when using LINQ, if you are accepting user input to build expressions, you need to make sure that only intended expressions can be constructed.</span></span>
