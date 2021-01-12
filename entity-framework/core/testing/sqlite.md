---
title: 用 SQLite 测试 EF Core
description: 使用 SQLite 测试 Entity Framework Core 应用程序
author: ajcvickers
ms.date: 04/24/2020
uid: core/testing/sqlite
ms.openlocfilehash: da2504cfe7997a10a5ee8c447b1c6ef00dd02369
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129052"
---
# <a name="using-sqlite-to-test-an-ef-core-application"></a>使用 SQLite 测试 EF Core 应用程序

> [!WARNING]
> 使用 SQLite 可以有效地测试 EF Core 应用程序。
> 但是，当 SQLite 的行为与其他数据库系统不同时，可能会出现问题。
> 有关问题和利弊权衡的讨论，请参阅 [使用 EF Core 的测试代码](xref:core/testing/index) 。

本文档在示例中所述的概念上生成了 [如何测试使用 EF Core 的应用程序](xref:core/testing/testing-sample)。
此处所示的代码示例来自于此示例。

## <a name="using-sqlite-in-memory-databases"></a>使用 SQLite 内存中数据库

通常情况下，SQLite 以简单文件的形式创建数据库，并在应用程序中访问文件。
这种速度非常快，尤其是在使用快速 [SSD](https://en.wikipedia.org/wiki/Solid-state_drive)时。

SQLite 还可以使用纯粹在内存中创建的数据库。
只要了解内存中数据库生存期，就可以轻松地在 EF Core 中使用：

* 当打开数据库的连接时，将创建该数据库
* 当数据库的连接关闭时，将删除该数据库

EF Core 将在给定连接时使用已打开的连接，并且不会尝试关闭该连接。
因此，将 EF Core 用于内存中 SQLite 数据库的关键是在将连接传递给 EF 之前打开连接。

该 [示例](xref:core/testing/testing-sample) 通过以下代码实现此操作：

[!code-csharp[SqliteInMemory](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/SqliteInMemoryItemsControllerTest.cs?name=SqliteInMemory)]

注意：

* `CreateInMemoryDatabase`方法创建一个 SQLite 内存中数据库，并打开与该数据库的连接。
* 创建的 `DbConnection` 从中提取 `ContextOptions` ，并保存。
* 当释放测试以便不泄漏资源时，将释放该连接。

> [!NOTE]
> [问题 #16103](https://github.com/dotnet/efcore/issues/16103) 正在跟踪使此连接管理更容易的方法。
