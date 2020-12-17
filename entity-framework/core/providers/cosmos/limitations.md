---
title: Azure Cosmos DB 提供程序-限制-EF Core
description: 与其他提供程序相比，Entity Framework Core Azure Cosmos DB 提供程序的限制
author: AndriySvyryd
ms.date: 11/05/2020
uid: core/providers/cosmos/limitations
ms.openlocfilehash: 088f593dddd9b5691d87566d7e31a699ba90d7c5
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97635713"
---
# <a name="ef-core-azure-cosmos-db-provider-limitations"></a><span data-ttu-id="9b4e8-103">Azure Cosmos DB 提供程序限制 EF Core</span><span class="sxs-lookup"><span data-stu-id="9b4e8-103">EF Core Azure Cosmos DB Provider Limitations</span></span>

<span data-ttu-id="9b4e8-104">Cosmos 提供程序有很多限制。</span><span class="sxs-lookup"><span data-stu-id="9b4e8-104">The Cosmos provider has a number of limitations.</span></span> <span data-ttu-id="9b4e8-105">其中的许多限制是由基础 Cosmos 数据库引擎中的限制引起的，并不特定于 EF。</span><span class="sxs-lookup"><span data-stu-id="9b4e8-105">Many of these limitations are a result of limitations in the underlying Cosmos database engine and are not specific to EF.</span></span> <span data-ttu-id="9b4e8-106">但大多数情况还 [没有实现](https://github.com/dotnet/efcore/issues?page=1&q=is%3Aissue+is%3Aopen+Cosmos+in%3Atitle+label%3Atype-enhancement+sort%3Areactions-%2B1-desc)。</span><span class="sxs-lookup"><span data-stu-id="9b4e8-106">But most simply [haven't been implemented yet](https://github.com/dotnet/efcore/issues?page=1&q=is%3Aissue+is%3Aopen+Cosmos+in%3Atitle+label%3Atype-enhancement+sort%3Areactions-%2B1-desc).</span></span>

<span data-ttu-id="9b4e8-107">下面是一些常见的请求功能：</span><span class="sxs-lookup"><span data-stu-id="9b4e8-107">These are some of the commonly requested features:</span></span>

- [<span data-ttu-id="9b4e8-108">`Include` 支持</span><span class="sxs-lookup"><span data-stu-id="9b4e8-108">`Include` support</span></span>](https://github.com/dotnet/efcore/issues/16920)
- [<span data-ttu-id="9b4e8-109">`Join` 支持</span><span class="sxs-lookup"><span data-stu-id="9b4e8-109">`Join` support</span></span>](https://github.com/dotnet/efcore/issues/17314)
- [<span data-ttu-id="9b4e8-110">基元类型的集合支持</span><span class="sxs-lookup"><span data-stu-id="9b4e8-110">Collections of primitive types support</span></span>](https://github.com/dotnet/efcore/issues/14762)

## <a name="azure-cosmos-db-sdk-limitations"></a><span data-ttu-id="9b4e8-111">Azure Cosmos DB SDK 限制</span><span class="sxs-lookup"><span data-stu-id="9b4e8-111">Azure Cosmos DB SDK limitations</span></span>

- <span data-ttu-id="9b4e8-112">仅提供 async 方法</span><span class="sxs-lookup"><span data-stu-id="9b4e8-112">Only async methods are provided</span></span>

> [!WARNING]
> <span data-ttu-id="9b4e8-113">由于没有 EF Core 依赖的低级方法的同步版本，因此当前通过对返回的调用来实现相应的功能 `.Wait()` `Task` 。</span><span class="sxs-lookup"><span data-stu-id="9b4e8-113">Since there are no sync versions of the low level methods EF Core relies on, the corresponding functionality is currently implemented by calling `.Wait()` on the returned `Task`.</span></span> <span data-ttu-id="9b4e8-114">这意味着，使用或等方法（ `SaveChanges` `ToList` 而不是其异步对应项）可能会导致应用程序中出现死锁</span><span class="sxs-lookup"><span data-stu-id="9b4e8-114">This means that using methods like `SaveChanges`, or `ToList` instead of their async counterparts could lead to a deadlock in your application</span></span>

## <a name="azure-cosmos-db-limitations"></a><span data-ttu-id="9b4e8-115">Azure Cosmos DB 限制</span><span class="sxs-lookup"><span data-stu-id="9b4e8-115">Azure Cosmos DB limitations</span></span>

<span data-ttu-id="9b4e8-116">你可以查看 [Azure Cosmos DB 支持的功能](/azure/cosmos-db/modeling-data)的完整概述，与关系数据库相比，这些是最明显的区别：</span><span class="sxs-lookup"><span data-stu-id="9b4e8-116">You can see the full overview of [Azure Cosmos DB supported features](/azure/cosmos-db/modeling-data), these are the most notable differences compared to a relational database:</span></span>

- <span data-ttu-id="9b4e8-117">不支持客户端启动的事务</span><span class="sxs-lookup"><span data-stu-id="9b4e8-117">Client-initiated transactions are not supported</span></span>
- <span data-ttu-id="9b4e8-118">某些跨分区查询的速度较慢，具体取决于例如 (涉及的运算符 `Skip/Take` 或 `OFFSET LIMIT`) </span><span class="sxs-lookup"><span data-stu-id="9b4e8-118">Some cross-partition queries are slower depending on the operators involved (for example `Skip/Take` or `OFFSET LIMIT`)</span></span>
