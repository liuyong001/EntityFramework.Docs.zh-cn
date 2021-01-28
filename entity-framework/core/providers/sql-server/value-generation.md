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
# <a name="sql-server-value-generation"></a>SQL Server 值生成

此页详细介绍了特定于 SQL Server 提供程序的值生成配置和模式。 建议先阅读 [值生成的 "常规" 页](xref:core/modeling/generated-properties)。

## <a name="identity-columns"></a>IDENTITY 列

按照约定，配置为在添加时生成值的数字列将设置为 [SQL SERVER 标识列](https://docs.microsoft.com/sql/t-sql/statements/create-table-transact-sql-identity-property)。

### <a name="seed-and-increment"></a>种子和增量

默认情况下，标识列从 1 (种子) 开始，每次在增量)  (添加行时递增1。 可以配置不同的种子和增量，如下所示：

[!code-csharp[Main](../../../../samples/core/SqlServer/ValueGeneration/IdentityOptionsContext.cs?name=IdentityOptions&highlight=5)]

> [!NOTE]
> EF Core 3.0 中引入了配置标识种子和增量的功能。

### <a name="inserting-explicit-values-into-identity-columns"></a>将显式值插入到标识列中

默认情况下，SQL Server 不允许将显式值插入到标识列中。 为此，必须在 `IDENTITY_INSERT` 调用之前手动启用 `SaveChanges()` ，如下所示：

[!code-csharp[Main](../../../../samples/core/SqlServer/ValueGeneration/ExplicitIdentityValues.cs?name=ExplicitIdentityValues)]

> [!NOTE]
> 积压工作中有[功能请求](https://github.com/aspnet/EntityFramework/issues/703)，用来在 SQL Server 提供程序内自动执行此操作。

## <a name="sequences"></a>序列

作为标识列的替代方法，可以使用标准序列。 这在各种情况下都很有用;例如，您可能希望多个列从单个序列绘制其默认值。

SQL Server 允许你创建序列并按 [序列的 "常规" 页上的](xref:core/modeling/sequences)详细信息使用它们。 你需要将属性配置为通过使用序列 `HasDefaultValueSql()` 。
