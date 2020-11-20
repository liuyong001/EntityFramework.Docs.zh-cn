---
title: Microsoft SQL Server 数据库提供程序-索引-EF Core
description: 特定于 Entity Framework Core SQL Server 提供程序的索引功能
author: roji
ms.date: 9/1/2020
uid: core/providers/sql-server/indexes
ms.openlocfilehash: 66b4e3ce5ab1d4da855c106a6a2d2e75c43081f7
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003155"
---
# <a name="index-features-specific-to-the-entity-framework-core-sql-server-provider"></a>特定于 Entity Framework Core SQL Server 提供程序的索引功能

此页详细介绍了特定于 SQL Server 提供程序的索引配置选项。

## <a name="clustering"></a>群集

聚集索引根据数据行的键值在表或视图中排序和存储这些数据行。 为表创建适当的聚集索引可以显著提高查询的速度，因为数据已经按最佳顺序进行布局。 每个表只能有一个聚集索引，因为数据行本身只能按一个顺序存储。 有关详细信息，请参阅 [有关聚集索引和非聚集索引的 SQL Server 文档](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described)。

默认情况下，表的主键列由聚集索引隐式支持，所有其他索引为非聚集索引。

可以按如下所示配置要聚集的索引或键：

[!code-csharp[ClusteredIndex](../../../../samples/core/SqlServer/Indexes/ClusteredIndexContext.cs?name=ClusteredIndex)]

## <a name="fill-factor"></a>填充因子

> [!NOTE]
> EF Core 5.0 中引入了此功能。

提供索引填充因子选项是为了优化索引数据存储和性能。 有关详细信息，请参阅 [有关填充因子的 SQL Server 文档](/sql/relational-databases/indexes/specify-fill-factor-for-an-index)。

您可以按如下所示配置索引的填充因子：

[!code-csharp[IndexFillFactor](../../../../samples/core/SqlServer/Indexes/IndexFillFactorContext.cs?name=IndexFillFactor)]

## <a name="online-creation"></a>联机创建

ONLINE 选项允许在创建索引期间并发用户访问基础表或聚集索引数据以及任何关联的非聚集索引，以便用户可以继续更新和查询基础数据。 当脱机执行数据定义语言 (DDL) 操作（例如，生成或重新生成聚集索引）时，这些操作对基础数据和关联索引持有排他锁。 有关详细信息，请参阅 [联机索引选项上的 SQL Server 文档](/sql/relational-databases/indexes/perform-index-operations-online)。

可以使用 ONLINE 选项配置索引，如下所示：

[!code-csharp[IndexOnline](../../../../samples/core/SqlServer/Indexes/IndexOnlineContext.cs?name=IndexOnline)]
