---
title: 函数映射-Azure Cosmos DB 提供程序-EF Core
description: Azure Cosmos DB EF Core 提供程序的函数映射
author: bricelam
ms.date: 1/26/2021
uid: core/providers/cosmos/functions
ms.openlocfilehash: d4d45ba7db50befb5eb775feed0af44468ee3f74
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543583"
---
# <a name="function-mappings-of-the-azure-cosmos-db-ef-core-provider"></a><span data-ttu-id="7791d-103">Azure Cosmos DB EF Core 提供程序的函数映射</span><span class="sxs-lookup"><span data-stu-id="7791d-103">Function Mappings of the Azure Cosmos DB EF Core Provider</span></span>

<span data-ttu-id="7791d-104">此页显示使用 Azure Cosmos DB 提供程序时，哪些 .NET 成员转换为哪些 SQL 函数。</span><span class="sxs-lookup"><span data-stu-id="7791d-104">This page shows which .NET members are translated into which SQL functions when using the Azure Cosmos DB provider.</span></span>

<span data-ttu-id="7791d-105">.NET</span><span class="sxs-lookup"><span data-stu-id="7791d-105">.NET</span></span>                          | <span data-ttu-id="7791d-106">SQL</span><span class="sxs-lookup"><span data-stu-id="7791d-106">SQL</span></span>                              | <span data-ttu-id="7791d-107">在</span><span class="sxs-lookup"><span data-stu-id="7791d-107">Added in</span></span>
----------------------------- | -------------------------------- | --------
<span data-ttu-id="7791d-108">集合.包含 (项) </span><span class="sxs-lookup"><span data-stu-id="7791d-108">collection.Contains(item)</span></span>     | <span data-ttu-id="7791d-109">@item 中 @collection</span><span class="sxs-lookup"><span data-stu-id="7791d-109">@item IN @collection</span></span>
<span data-ttu-id="7791d-110">EF.函数。随机 () </span><span class="sxs-lookup"><span data-stu-id="7791d-110">EF.Functions.Random()</span></span>         | <span data-ttu-id="7791d-111">RAND () </span><span class="sxs-lookup"><span data-stu-id="7791d-111">RAND()</span></span>                           | <span data-ttu-id="7791d-112">EF Core 6.0</span><span class="sxs-lookup"><span data-stu-id="7791d-112">EF Core 6.0</span></span>
<span data-ttu-id="7791d-113">stringValue 包含 (值) </span><span class="sxs-lookup"><span data-stu-id="7791d-113">stringValue.Contains(value)</span></span>   | <span data-ttu-id="7791d-114">包含 (@stringValue 、 @value) </span><span class="sxs-lookup"><span data-stu-id="7791d-114">CONTAINS(@stringValue, @value)</span></span>   | <span data-ttu-id="7791d-115">EF Core 5.0</span><span class="sxs-lookup"><span data-stu-id="7791d-115">EF Core 5.0</span></span>
<span data-ttu-id="7791d-116">stringValue. EndsWith (值) </span><span class="sxs-lookup"><span data-stu-id="7791d-116">stringValue.EndsWith(value)</span></span>   | <span data-ttu-id="7791d-117">ENDSWITH (@stringValue ， @value) </span><span class="sxs-lookup"><span data-stu-id="7791d-117">ENDSWITH(@stringValue, @value)</span></span>   | <span data-ttu-id="7791d-118">EF Core 5.0</span><span class="sxs-lookup"><span data-stu-id="7791d-118">EF Core 5.0</span></span>
<span data-ttu-id="7791d-119">stringValue. FirstOrDefault () </span><span class="sxs-lookup"><span data-stu-id="7791d-119">stringValue.FirstOrDefault()</span></span>  | <span data-ttu-id="7791d-120">左 (@stringValue ，1) </span><span class="sxs-lookup"><span data-stu-id="7791d-120">LEFT(@stringValue, 1)</span></span>            | <span data-ttu-id="7791d-121">EF Core 5.0</span><span class="sxs-lookup"><span data-stu-id="7791d-121">EF Core 5.0</span></span>
<span data-ttu-id="7791d-122">stringValue. LastOrDefault () </span><span class="sxs-lookup"><span data-stu-id="7791d-122">stringValue.LastOrDefault()</span></span>   | <span data-ttu-id="7791d-123">RIGHT (@stringValue ，1) </span><span class="sxs-lookup"><span data-stu-id="7791d-123">RIGHT(@stringValue, 1)</span></span>           | <span data-ttu-id="7791d-124">EF Core 5.0</span><span class="sxs-lookup"><span data-stu-id="7791d-124">EF Core 5.0</span></span>
<span data-ttu-id="7791d-125">stringValue. StartsWith (值) </span><span class="sxs-lookup"><span data-stu-id="7791d-125">stringValue.StartsWith(value)</span></span> | <span data-ttu-id="7791d-126">STARTSWITH (@stringValue ， @value) </span><span class="sxs-lookup"><span data-stu-id="7791d-126">STARTSWITH(@stringValue, @value)</span></span> | <span data-ttu-id="7791d-127">EF Core 5.0</span><span class="sxs-lookup"><span data-stu-id="7791d-127">EF Core 5.0</span></span>
