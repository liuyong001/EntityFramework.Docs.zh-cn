---
title: 查询的工作原理 - EF Core
description: 关于 Entity Framework Core 如何在内部编译和执行查询的常规信息
author: ajcvickers
ms.date: 03/17/2020
uid: core/querying/how-query-works
ms.openlocfilehash: 4f856bf7e084d020edee1585d5c6c64873a5ed15
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129000"
---
# <a name="how-queries-work"></a>查询的工作原理

Entity Framework Core 使用语言集成查询 (LINQ) 来查询数据库中的数据。 通过 LINQ 可使用 C#（或你选择的 .NET 语言）基于派生上下文和实体类编写强类型查询。

> [!NOTE]
> 本文已过时，其中的某些部分需要更新，以解释查询管道设计中发生的更改。 如果对此处提到的任何行为有任何疑问，请[提问](https://github.com/dotnet/efcore/issues/new/choose)。

## <a name="the-life-of-a-query"></a>查询的寿命

下面的描述是每个查询所经历的过程的综合概述。

1. LINQ 查询由 Entity Framework Core 处理，用于生成已准备好由数据库提供程序处理的表示形式
   1. 结果会被缓存，以便每次执行查询时无需进行此处理
2. 结果会传递到数据库提供程序
   1. 数据库提供程序确定可以在数据库中评估查询的哪些部分
   2. 查询的这些部分会转换为特定数据库的查询语言（例如关系数据库的 SQL）
   3. 查询会发送到数据库并返回结果集（返回的是数据库中的值，而不是实体实例中的）
3. 对于结果集中的每一项
   1. 如果该查询是跟踪查询，EF 会检查数据是否表示上下文实例内更改跟踪器中的现有实体
      * 如果是，则会返回现有实体
      * 如果不是，则会创建新实体、设置更改跟踪并返回该新实体
   2. 如果该查询是非跟踪查询，将始终创建并返回新实体

## <a name="when-queries-are-executed"></a>执行查询时

调用 LINQ 运算符时，只会构建查询在内存中的表示形式。 使用结果时，查询只会发送到数据库。

导致查询发送到数据库的最常见操作如下：

* 在 `for` 循环中循环访问结果
* 使用 `ToList`、`ToArray`、`Single`、`Count` 等操作或等效的异步重载

> [!WARNING]
> 始终验证用户输入：虽然 EF Core通过在查询中使用参数和转义文字来防止 SQL 注入攻击，但它不会验证输入。 根据应用程序的要求，在将 LINQ 查询中使用的来自不受信任的源的值分配给实体属性或传递给其他 EF Core API 之前，应执行相应的验证。 这包括用于动态构造查询的所有用户输入。 即使在使用 LINQ 时，如果接受用于生成表达式的用户输入，也会需要确保只能构造预期表达式。
