---
title: 用 EF In-Memory 数据库进行测试-EF Core
description: 使用 EF 内存中数据库测试 Entity Framework Core 应用程序
author: ajcvickers
ms.date: 10/27/2016
uid: core/testing/in-memory
ms.openlocfilehash: 78dcac3d0fd69110986c99a097a864104caa1951
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128805"
---
# <a name="testing-with-the-ef-in-memory-database"></a><span data-ttu-id="0a1f0-103">用 EF In-Memory 数据库进行测试</span><span class="sxs-lookup"><span data-stu-id="0a1f0-103">Testing with the EF In-Memory Database</span></span>

> [!WARNING]
> <span data-ttu-id="0a1f0-104">EF 内存中数据库的行为通常与关系数据库不同。</span><span class="sxs-lookup"><span data-stu-id="0a1f0-104">The EF in-memory database often behaves differently than relational databases.</span></span>
> <span data-ttu-id="0a1f0-105">在充分了解所涉及的问题和折衷后，只需使用 EF 内存中数据库，如 [测试使用 EF Core 的代码](xref:core/testing/index)中所述。</span><span class="sxs-lookup"><span data-stu-id="0a1f0-105">Only use the EF in-memory database after fully understanding the issues and trade-offs involved, as discussed in [Testing code that uses EF Core](xref:core/testing/index).</span></span>

> [!TIP]
> <span data-ttu-id="0a1f0-106">SQLite 是一个关系提供程序，还可以使用内存中数据库。</span><span class="sxs-lookup"><span data-stu-id="0a1f0-106">SQLite is a relational provider and can also use in-memory databases.</span></span>
> <span data-ttu-id="0a1f0-107">请考虑使用它进行测试，以便更密切地匹配常见的关系数据库行为。</span><span class="sxs-lookup"><span data-stu-id="0a1f0-107">Consider using this for testing to more closely match common relational database behaviors.</span></span>
> <span data-ttu-id="0a1f0-108">[使用 SQLite 测试 EF Core 的应用程序中对](xref:core/testing/sqlite)此进行了介绍。</span><span class="sxs-lookup"><span data-stu-id="0a1f0-108">This is covered in [Using SQLite to test an EF Core application](xref:core/testing/sqlite).</span></span>

<span data-ttu-id="0a1f0-109">此页上的信息现在驻留在其他位置：</span><span class="sxs-lookup"><span data-stu-id="0a1f0-109">The information on this page now lives in other locations:</span></span>

* <span data-ttu-id="0a1f0-110">有关使用 EF 内存中数据库进行测试的常规信息，请参阅 [测试使用 EF Core 的代码](xref:core/testing/index) 。</span><span class="sxs-lookup"><span data-stu-id="0a1f0-110">See [Testing code that uses EF Core](xref:core/testing/index) for general information on testing with the EF in-memory database.</span></span>
* <span data-ttu-id="0a1f0-111">请参阅示例，演示如何使用 EF 内存中数据库 [测试使用 EF Core 的示例应用程序](xref:core/testing/testing-sample) 。</span><span class="sxs-lookup"><span data-stu-id="0a1f0-111">See [Sample showing how to test applications that use EF Core](xref:core/testing/testing-sample) for a sample using the EF in-memory database.</span></span>
* <span data-ttu-id="0a1f0-112">有关 EF 内存中数据库的常规信息，请参阅 [ef 内存中数据库提供程序](xref:core/providers/in-memory/index) 。</span><span class="sxs-lookup"><span data-stu-id="0a1f0-112">See [The EF in-memory database provider](xref:core/providers/in-memory/index) for general information about the EF in-memory database.</span></span>
