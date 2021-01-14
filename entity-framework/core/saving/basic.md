---
title: 基本保存 - EF Core
description: 关于通过 Entity Framework Core 添加、更新和删除数据的基本信息
author: ajcvickers
ms.date: 10/27/2016
uid: core/saving/basic
ms.openlocfilehash: 8c98f95dee85641f11af716290333b47f340bff8
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129104"
---
# <a name="basic-save"></a><span data-ttu-id="bc368-103">基本保存</span><span class="sxs-lookup"><span data-stu-id="bc368-103">Basic Save</span></span>

<span data-ttu-id="bc368-104">了解如何使用上下文和实体类添加、修改和删除数据。</span><span class="sxs-lookup"><span data-stu-id="bc368-104">Learn how to add, modify, and remove data using your context and entity classes.</span></span>

> [!TIP]
> <span data-ttu-id="bc368-105">可在 GitHub 上查看此文章的[示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Saving/Basics/)。</span><span class="sxs-lookup"><span data-stu-id="bc368-105">You can view this article's [sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Saving/Basics/) on GitHub.</span></span>

## <a name="adding-data"></a><span data-ttu-id="bc368-106">添加数据</span><span class="sxs-lookup"><span data-stu-id="bc368-106">Adding Data</span></span>

<span data-ttu-id="bc368-107">使用 *DbSet.Add* 方法添加实体类的新实例。</span><span class="sxs-lookup"><span data-stu-id="bc368-107">Use the *DbSet.Add* method to add new instances of your entity classes.</span></span> <span data-ttu-id="bc368-108">调用 *SaveChanges* 时，数据将插入到数据库中。</span><span class="sxs-lookup"><span data-stu-id="bc368-108">The data will be inserted in the database when you call *SaveChanges*.</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Basics/Sample.cs#Add)]

> [!TIP]
> <span data-ttu-id="bc368-109">添加、附加和更新方法全部呈现在传递给这些方法的实体的完整关系图上，如[相关数据](xref:core/saving/related-data)部分中所述。</span><span class="sxs-lookup"><span data-stu-id="bc368-109">The Add, Attach, and Update methods all work on the full graph of entities passed to them, as described in the [Related Data](xref:core/saving/related-data) section.</span></span> <span data-ttu-id="bc368-110">此外，还可以使用 EntityEntry.State 属性仅设置单个实体的状态。</span><span class="sxs-lookup"><span data-stu-id="bc368-110">Alternately, the EntityEntry.State property can be used to set the state of just a single entity.</span></span> <span data-ttu-id="bc368-111">例如，`context.Entry(blog).State = EntityState.Modified`。</span><span class="sxs-lookup"><span data-stu-id="bc368-111">For example, `context.Entry(blog).State = EntityState.Modified`.</span></span>

## <a name="updating-data"></a><span data-ttu-id="bc368-112">更新数据</span><span class="sxs-lookup"><span data-stu-id="bc368-112">Updating Data</span></span>

<span data-ttu-id="bc368-113">EF 将自动检测对由上下文跟踪的现有实体所做的更改。</span><span class="sxs-lookup"><span data-stu-id="bc368-113">EF will automatically detect changes made to an existing entity that is tracked by the context.</span></span> <span data-ttu-id="bc368-114">这包括从数据库加载/查询的实体，以及之前添加并保存到数据库的实体。</span><span class="sxs-lookup"><span data-stu-id="bc368-114">This includes entities that you load/query from the database, and entities that were previously added and saved to the database.</span></span>

<span data-ttu-id="bc368-115">只需修改分配给属性的值，然后调用 *SaveChanges*。</span><span class="sxs-lookup"><span data-stu-id="bc368-115">Simply modify the values assigned to properties and then call *SaveChanges*.</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Basics/Sample.cs#Update)]

## <a name="deleting-data"></a><span data-ttu-id="bc368-116">删除数据</span><span class="sxs-lookup"><span data-stu-id="bc368-116">Deleting Data</span></span>

<span data-ttu-id="bc368-117">使用 DbSet.Remove 方法删除实体类的实例。</span><span class="sxs-lookup"><span data-stu-id="bc368-117">Use the *DbSet.Remove* method to delete instances of your entity classes.</span></span>

<span data-ttu-id="bc368-118">如果实体已存在于数据库中，则将在“SaveChanges”期间删除该实体。</span><span class="sxs-lookup"><span data-stu-id="bc368-118">If the entity already exists in the database, it will be deleted during *SaveChanges*.</span></span> <span data-ttu-id="bc368-119">如果实体尚未保存到数据库（即跟踪为“已添加”），则在调用“SaveChanges”时，该实体会从上下文中删除且不再插入。</span><span class="sxs-lookup"><span data-stu-id="bc368-119">If the entity has not yet been saved to the database (that is, it is tracked as added) then it will be removed from the context and will no longer be inserted when *SaveChanges* is called.</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Basics/Sample.cs#Remove)]

## <a name="multiple-operations-in-a-single-savechanges"></a><span data-ttu-id="bc368-120">单个 SaveChanges 中的多个操作</span><span class="sxs-lookup"><span data-stu-id="bc368-120">Multiple Operations in a single SaveChanges</span></span>

<span data-ttu-id="bc368-121">可以将多个添加/更新/删除操作合并到对“SaveChanges”的单个调用。</span><span class="sxs-lookup"><span data-stu-id="bc368-121">You can combine multiple Add/Update/Remove operations into a single call to *SaveChanges*.</span></span>

> [!NOTE]
> <span data-ttu-id="bc368-122">对于大多数数据库提供程序，“SaveChanges”是事务性的。</span><span class="sxs-lookup"><span data-stu-id="bc368-122">For most database providers, *SaveChanges* is transactional.</span></span> <span data-ttu-id="bc368-123">这意味着所有操作将成功或失败，决不部分应用这些操作。</span><span class="sxs-lookup"><span data-stu-id="bc368-123">This means  all the operations will either succeed or fail and the operations will never be left partially applied.</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Basics/Sample.cs#MultipleOperations)]
