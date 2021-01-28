---
title: Microsoft SQL Server 数据库提供程序-值生成-EF Core
description: 特定于 SQL Server Entity Framework Core 数据库提供程序的值生成模式
author: roji
ms.date: 1/10/2020
uid: core/providers/sql-server/value-generation
ms.openlocfilehash: 90608f254a3d20e3c36586ae8325e0a982a7fa27
ms.sourcegitcommit: 7700840119b1639275f3b64836e7abb59103f2e7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2021
ms.locfileid: "98987135"
---
# <a name="sql-server-value-generation"></a><span data-ttu-id="4bb55-103">SQL Server 值生成</span><span class="sxs-lookup"><span data-stu-id="4bb55-103">SQL Server Value Generation</span></span>

<span data-ttu-id="4bb55-104">此页详细介绍了特定于 SQL Server 提供程序的值生成配置和模式。</span><span class="sxs-lookup"><span data-stu-id="4bb55-104">This page details value generation configuration  and patterns that are specific to the SQL Server provider.</span></span> <span data-ttu-id="4bb55-105">建议先阅读 [值生成的 "常规" 页](xref:core/modeling/generated-properties)。</span><span class="sxs-lookup"><span data-stu-id="4bb55-105">It's recommended to first read [the general page on value generation](xref:core/modeling/generated-properties).</span></span>

## <a name="identity-columns"></a><span data-ttu-id="4bb55-106">IDENTITY 列</span><span class="sxs-lookup"><span data-stu-id="4bb55-106">IDENTITY columns</span></span>

<span data-ttu-id="4bb55-107">按照约定，配置为在添加时生成值的数字列将设置为 [SQL SERVER 标识列](https://docs.microsoft.com/sql/t-sql/statements/create-table-transact-sql-identity-property)。</span><span class="sxs-lookup"><span data-stu-id="4bb55-107">By convention, numeric columns that are configured to have their values generated on add are set up as [SQL Server IDENTITY columns](https://docs.microsoft.com/sql/t-sql/statements/create-table-transact-sql-identity-property).</span></span>

### <a name="seed-and-increment"></a><span data-ttu-id="4bb55-108">种子和增量</span><span class="sxs-lookup"><span data-stu-id="4bb55-108">Seed and increment</span></span>

<span data-ttu-id="4bb55-109">默认情况下，标识列从 1 (种子) 开始，每次在增量)  (添加行时递增1。</span><span class="sxs-lookup"><span data-stu-id="4bb55-109">By default, IDENTITY columns start off at 1 (the seed), and increment by 1 each time a row is added (the increment).</span></span> <span data-ttu-id="4bb55-110">可以配置不同的种子和增量，如下所示：</span><span class="sxs-lookup"><span data-stu-id="4bb55-110">You can configure a different seed and increment as follows:</span></span>

[!code-csharp[Main](../../../../samples/core/SqlServer/ValueGeneration/IdentityOptionsContext.cs?name=IdentityOptions&highlight=5)]

> [!NOTE]
> <span data-ttu-id="4bb55-111">EF Core 3.0 中引入了配置标识种子和增量的功能。</span><span class="sxs-lookup"><span data-stu-id="4bb55-111">The ability to configure IDENTITY seed and increment was introduced in EF Core 3.0.</span></span>

### <a name="inserting-explicit-values-into-identity-columns"></a><span data-ttu-id="4bb55-112">将显式值插入到标识列中</span><span class="sxs-lookup"><span data-stu-id="4bb55-112">Inserting explicit values into IDENTITY columns</span></span>

<span data-ttu-id="4bb55-113">默认情况下，SQL Server 不允许将显式值插入到标识列中。</span><span class="sxs-lookup"><span data-stu-id="4bb55-113">By default, SQL Server doesn't allow inserting explicit values into IDENTITY columns.</span></span> <span data-ttu-id="4bb55-114">为此，必须在 `IDENTITY_INSERT` 调用之前手动启用 `SaveChanges()` ，如下所示：</span><span class="sxs-lookup"><span data-stu-id="4bb55-114">To do so, you must manually enable `IDENTITY_INSERT` before calling `SaveChanges()`, as follows:</span></span>

[!code-csharp[Main](../../../../samples/core/SqlServer/ValueGeneration/ExplicitIdentityValues.cs?name=ExplicitIdentityValues)]

> [!NOTE]
> <span data-ttu-id="4bb55-115">积压工作中有[功能请求](https://github.com/aspnet/EntityFramework/issues/703)，用来在 SQL Server 提供程序内自动执行此操作。</span><span class="sxs-lookup"><span data-stu-id="4bb55-115">We have a [feature request](https://github.com/aspnet/EntityFramework/issues/703) on our backlog to do this automatically within the SQL Server provider.</span></span>

## <a name="sequences"></a><span data-ttu-id="4bb55-116">序列</span><span class="sxs-lookup"><span data-stu-id="4bb55-116">Sequences</span></span>

<span data-ttu-id="4bb55-117">作为标识列的替代方法，可以使用标准序列。</span><span class="sxs-lookup"><span data-stu-id="4bb55-117">As an alternative to IDENTITY columns, you can use standard sequences.</span></span> <span data-ttu-id="4bb55-118">这在各种情况下都很有用;例如，您可能希望多个列从单个序列绘制其默认值。</span><span class="sxs-lookup"><span data-stu-id="4bb55-118">This can be useful in various scenarios; for example, you may want to have multiple columns drawing their default values from a single sequence.</span></span>

<span data-ttu-id="4bb55-119">SQL Server 允许你创建序列并按 [序列的 "常规" 页上的](xref:core/modeling/sequences)详细信息使用它们。</span><span class="sxs-lookup"><span data-stu-id="4bb55-119">SQL Server allows you to create sequences and use them as detailed in [the general page on sequences](xref:core/modeling/sequences).</span></span> <span data-ttu-id="4bb55-120">你需要将属性配置为通过使用序列 `HasDefaultValueSql()` 。</span><span class="sxs-lookup"><span data-stu-id="4bb55-120">It's up to you to configure your properties to use sequences via `HasDefaultValueSql()`.</span></span>
