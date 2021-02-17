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
# <a name="function-mappings-of-the-azure-cosmos-db-ef-core-provider"></a>Azure Cosmos DB EF Core 提供程序的函数映射

此页显示使用 Azure Cosmos DB 提供程序时，哪些 .NET 成员转换为哪些 SQL 函数。

.NET                          | SQL                              | 在
----------------------------- | -------------------------------- | --------
集合.包含 (项)      | @item 中 @collection
EF.函数。随机 ()          | RAND ()                            | EF Core 6.0
stringValue 包含 (值)    | 包含 (@stringValue 、 @value)    | EF Core 5.0
stringValue. EndsWith (值)    | ENDSWITH (@stringValue ， @value)    | EF Core 5.0
stringValue. FirstOrDefault ()   | 左 (@stringValue ，1)             | EF Core 5.0
stringValue. LastOrDefault ()    | RIGHT (@stringValue ，1)            | EF Core 5.0
stringValue. StartsWith (值)  | STARTSWITH (@stringValue ， @value)  | EF Core 5.0
