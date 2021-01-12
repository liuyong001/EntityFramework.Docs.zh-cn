---
title: Microsoft SQL Server 数据库提供程序-索引-EF Core
description: 特定于 Entity Framework Core SQL Server 提供程序的索引功能
author: roji
ms.date: 9/1/2020
uid: core/providers/sql-server/indexes
ms.openlocfilehash: 42411a562b4741ba39b4eb855bb84c66e100456b
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129156"
---
# <a name="index-features-specific-to-the-entity-framework-core-sql-server-provider"></a><span data-ttu-id="02522-103">特定于 Entity Framework Core SQL Server 提供程序的索引功能</span><span class="sxs-lookup"><span data-stu-id="02522-103">Index features specific to the Entity Framework Core SQL Server provider</span></span>

<span data-ttu-id="02522-104">此页详细介绍了特定于 SQL Server 提供程序的索引配置选项。</span><span class="sxs-lookup"><span data-stu-id="02522-104">This page details index configuration options that are specific to the SQL Server provider.</span></span>

## <a name="clustering"></a><span data-ttu-id="02522-105">群集功能</span><span class="sxs-lookup"><span data-stu-id="02522-105">Clustering</span></span>

<span data-ttu-id="02522-106">聚集索引根据数据行的键值在表或视图中排序和存储这些数据行。</span><span class="sxs-lookup"><span data-stu-id="02522-106">Clustered indexes sort and store the data rows in the table or view based on their key values.</span></span> <span data-ttu-id="02522-107">为表创建适当的聚集索引可以显著提高查询的速度，因为数据已经按最佳顺序进行布局。</span><span class="sxs-lookup"><span data-stu-id="02522-107">Creating the right clustered index for your table can significantly improve the speed of your queries, as data is already laid out in the optimal order.</span></span> <span data-ttu-id="02522-108">每个表只能有一个聚集索引，因为数据行本身只能按一个顺序存储。</span><span class="sxs-lookup"><span data-stu-id="02522-108">There can be only one clustered index per table, because the data rows themselves can be stored in only one order.</span></span> <span data-ttu-id="02522-109">有关详细信息，请参阅 [有关聚集索引和非聚集索引的 SQL Server 文档](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described)。</span><span class="sxs-lookup"><span data-stu-id="02522-109">For more information, see [the SQL Server documentation on clustered and non-clustered indexes](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described).</span></span>

<span data-ttu-id="02522-110">默认情况下，表的主键列由聚集索引隐式支持，所有其他索引为非聚集索引。</span><span class="sxs-lookup"><span data-stu-id="02522-110">By default, the primary key column of a table is implicitly backed by a clustered index, and all other indexes are non-clustered.</span></span>

<span data-ttu-id="02522-111">可以按如下所示配置要聚集的索引或键：</span><span class="sxs-lookup"><span data-stu-id="02522-111">You can configure an index or key to be clustered as follows:</span></span>

[!code-csharp[ClusteredIndex](../../../../samples/core/SqlServer/Indexes/ClusteredIndexContext.cs?name=ClusteredIndex)]

> [!NOTE]
> <span data-ttu-id="02522-112">SQL Server 仅支持每个表有一个聚集索引，并且在默认情况下，主密钥为聚集索引。</span><span class="sxs-lookup"><span data-stu-id="02522-112">SQL Server only supports one clustered index per table, and the primary key is by default clustered.</span></span> <span data-ttu-id="02522-113">如果要对非键列具有聚集索引，则必须将键显式设为非群集。</span><span class="sxs-lookup"><span data-stu-id="02522-113">If you'd like to have a clustered index on a non-key column, you must explicitly make your key non-clustered.</span></span>

## <a name="fill-factor"></a><span data-ttu-id="02522-114">填充因子</span><span class="sxs-lookup"><span data-stu-id="02522-114">Fill factor</span></span>

> [!NOTE]
> <span data-ttu-id="02522-115">EF Core 5.0 中已引入此功能。</span><span class="sxs-lookup"><span data-stu-id="02522-115">This feature was introduced in EF Core 5.0.</span></span>

<span data-ttu-id="02522-116">提供索引填充因子选项是为了优化索引数据存储和性能。</span><span class="sxs-lookup"><span data-stu-id="02522-116">The index fill-factor option is provided for fine-tuning index data storage and performance.</span></span> <span data-ttu-id="02522-117">有关详细信息，请参阅 [有关填充因子的 SQL Server 文档](/sql/relational-databases/indexes/specify-fill-factor-for-an-index)。</span><span class="sxs-lookup"><span data-stu-id="02522-117">For more information, see [the SQL Server documentation on fill factor](/sql/relational-databases/indexes/specify-fill-factor-for-an-index).</span></span>

<span data-ttu-id="02522-118">您可以按如下所示配置索引的填充因子：</span><span class="sxs-lookup"><span data-stu-id="02522-118">You can configure an index's fill factor as follows:</span></span>

[!code-csharp[IndexFillFactor](../../../../samples/core/SqlServer/Indexes/IndexFillFactorContext.cs?name=IndexFillFactor)]

## <a name="online-creation"></a><span data-ttu-id="02522-119">联机创建</span><span class="sxs-lookup"><span data-stu-id="02522-119">Online creation</span></span>

<span data-ttu-id="02522-120">ONLINE 选项允许在创建索引期间并发用户访问基础表或聚集索引数据以及任何关联的非聚集索引，以便用户可以继续更新和查询基础数据。</span><span class="sxs-lookup"><span data-stu-id="02522-120">The ONLINE option allows concurrent user access to the underlying table or clustered index data and any associated nonclustered indexes during index creation, so that users can continue to update and query the underlying data.</span></span> <span data-ttu-id="02522-121">当脱机执行数据定义语言 (DDL) 操作（例如，生成或重新生成聚集索引）时，这些操作对基础数据和关联索引持有排他锁。</span><span class="sxs-lookup"><span data-stu-id="02522-121">When you perform data definition language (DDL) operations offline, such as building or rebuilding a clustered index; these operations hold exclusive locks on the underlying data and associated indexes.</span></span> <span data-ttu-id="02522-122">有关详细信息，请参阅 [联机索引选项上的 SQL Server 文档](/sql/relational-databases/indexes/perform-index-operations-online)。</span><span class="sxs-lookup"><span data-stu-id="02522-122">For more information, see [the SQL Server documentation on the ONLINE index option](/sql/relational-databases/indexes/perform-index-operations-online).</span></span>

<span data-ttu-id="02522-123">可以使用 ONLINE 选项配置索引，如下所示：</span><span class="sxs-lookup"><span data-stu-id="02522-123">You can configure an index with the ONLINE option as follows:</span></span>

[!code-csharp[IndexOnline](../../../../samples/core/SqlServer/Indexes/IndexOnlineContext.cs?name=IndexOnline)]
