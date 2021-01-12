---
title: 序列-EF Core
description: 在 Entity Framework Core 模型中配置序列
author: roji
ms.date: 12/18/2019
uid: core/modeling/sequences
ms.openlocfilehash: e388bde11b1251564b37227e8884d2b8c34dc02e
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128519"
---
# <a name="sequences"></a><span data-ttu-id="5543e-103">序列</span><span class="sxs-lookup"><span data-stu-id="5543e-103">Sequences</span></span>

> [!NOTE]
> <span data-ttu-id="5543e-104">序列是通常仅由关系数据库支持的功能。</span><span class="sxs-lookup"><span data-stu-id="5543e-104">Sequences are a feature typically supported only by relational databases.</span></span> <span data-ttu-id="5543e-105">如果使用的是非关系数据库（如 Cosmos），请查看数据库文档以生成唯一值。</span><span class="sxs-lookup"><span data-stu-id="5543e-105">If you're using a non-relational database such as Cosmos, check your database documentation on generating unique values.</span></span>

<span data-ttu-id="5543e-106">序列将在数据库中生成唯一的顺序数值。</span><span class="sxs-lookup"><span data-stu-id="5543e-106">A sequence generates unique, sequential numeric values in the database.</span></span> <span data-ttu-id="5543e-107">序列不与特定表相关联，并且可以将多个表设置为从同一序列中绘制值。</span><span class="sxs-lookup"><span data-stu-id="5543e-107">Sequences are not associated with a specific table, and multiple tables can be set up to draw values from the same sequence.</span></span>

## <a name="basic-usage"></a><span data-ttu-id="5543e-108">基本用法</span><span class="sxs-lookup"><span data-stu-id="5543e-108">Basic usage</span></span>

<span data-ttu-id="5543e-109">您可以在模型中设置一个序列，然后使用它来生成属性值：</span><span class="sxs-lookup"><span data-stu-id="5543e-109">You can set up a sequence in the model, and then use it to generate values for properties:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Sequence.cs?name=Sequence&highlight=3,7)]

<span data-ttu-id="5543e-110">请注意，用于从序列生成值的特定 SQL 是特定于数据库的;上面的示例可在 SQL Server 上运行，但在其他数据库中将失败。</span><span class="sxs-lookup"><span data-stu-id="5543e-110">Note that the specific SQL used to generate a value from a sequence is database-specific; the above example works on SQL Server but will fail on other databases.</span></span> <span data-ttu-id="5543e-111">有关详细信息，请参阅特定数据库的文档。</span><span class="sxs-lookup"><span data-stu-id="5543e-111">Consult your specific database's documentation for more information.</span></span>

## <a name="configuring-sequence-settings"></a><span data-ttu-id="5543e-112">配置序列设置</span><span class="sxs-lookup"><span data-stu-id="5543e-112">Configuring sequence settings</span></span>

<span data-ttu-id="5543e-113">你还可以配置序列的其他方面，例如其架构、起始值、增量等：</span><span class="sxs-lookup"><span data-stu-id="5543e-113">You can also configure additional aspects of the sequence, such as its schema, start value, increment, etc.:</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/SequenceConfiguration.cs?name=SequenceConfiguration&highlight=3-5)]
