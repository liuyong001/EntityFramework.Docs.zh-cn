---
title: 创建并配置模型 - EF Core
description: 使用 Entity Framework Core 创建并配置模型的概述
author: AndriySvyryd
ms.date: 10/13/2020
uid: core/modeling/index
ms.openlocfilehash: 709e2bde60c8e2c31f0a39390624c5d31a9cfa08
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129195"
---
# <a name="creating-and-configuring-a-model"></a><span data-ttu-id="29bf8-103">创建并配置模型</span><span class="sxs-lookup"><span data-stu-id="29bf8-103">Creating and configuring a model</span></span>

<span data-ttu-id="29bf8-104">Entity Framework 使用一组约定基于实体类的形状构建模型。</span><span class="sxs-lookup"><span data-stu-id="29bf8-104">Entity Framework uses a set of conventions to build a model based on the shape of your entity classes.</span></span> <span data-ttu-id="29bf8-105">可指定其他配置以补充和/或替代约定的内容。</span><span class="sxs-lookup"><span data-stu-id="29bf8-105">You can specify additional configuration to supplement and/or override what was discovered by convention.</span></span>

<span data-ttu-id="29bf8-106">本文介绍可应用于面向任何数据存储的模型的配置，以及面向任意关系数据库时可应用的配置。</span><span class="sxs-lookup"><span data-stu-id="29bf8-106">This article covers configuration that can be applied to a model targeting any data store and that which can be applied when targeting any relational database.</span></span> <span data-ttu-id="29bf8-107">提供程序还可支持特定于具体数据存储的配置。</span><span class="sxs-lookup"><span data-stu-id="29bf8-107">Providers may also enable configuration that is specific to a particular data store.</span></span> <span data-ttu-id="29bf8-108">有关提供程序特定配置的文档，请参阅[数据库提供程序](xref:core/providers/index)部分。</span><span class="sxs-lookup"><span data-stu-id="29bf8-108">For documentation on provider specific configuration see the [Database Providers](xref:core/providers/index) section.</span></span>

> [!TIP]
> <span data-ttu-id="29bf8-109">可在 GitHub 上查看此文章的[示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples)。</span><span class="sxs-lookup"><span data-stu-id="29bf8-109">You can view this article’s [sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples) on GitHub.</span></span>

## <a name="use-fluent-api-to-configure-a-model"></a><span data-ttu-id="29bf8-110">使用 fluent API 配置模型</span><span class="sxs-lookup"><span data-stu-id="29bf8-110">Use fluent API to configure a model</span></span>

<span data-ttu-id="29bf8-111">可在派生上下文中替代 `OnModelCreating` 方法，并使用 `ModelBuilder API` 来配置模型。</span><span class="sxs-lookup"><span data-stu-id="29bf8-111">You can override the `OnModelCreating` method in your derived context and use the `ModelBuilder API` to configure your model.</span></span> <span data-ttu-id="29bf8-112">此配置方法最为有效，并可在不修改实体类的情况下指定配置。</span><span class="sxs-lookup"><span data-stu-id="29bf8-112">This is the most powerful method of configuration and allows configuration to be specified without modifying your entity classes.</span></span> <span data-ttu-id="29bf8-113">Fluent API 配置具有最高优先级，并将替代约定和数据注释。</span><span class="sxs-lookup"><span data-stu-id="29bf8-113">Fluent API configuration has the highest precedence and will override conventions and data annotations.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/Required.cs?highlight=12-14)]

### <a name="grouping-configuration"></a><span data-ttu-id="29bf8-114">分组配置</span><span class="sxs-lookup"><span data-stu-id="29bf8-114">Grouping configuration</span></span>

<span data-ttu-id="29bf8-115">为了减小 <xref:System.Data.Entity.DbContext.OnModelCreating%2A> 方法的大小，可以将实体类型的所有配置提取到实现 <xref:Microsoft.EntityFrameworkCore.IEntityTypeConfiguration%601> 的单独类中。</span><span class="sxs-lookup"><span data-stu-id="29bf8-115">To reduce the size of the <xref:System.Data.Entity.DbContext.OnModelCreating%2A> method all configuration for an entity type can be extracted to a separate class implementing <xref:Microsoft.EntityFrameworkCore.IEntityTypeConfiguration%601>.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/EntityTypeConfiguration.cs?Name=IEntityTypeConfiguration)]

<span data-ttu-id="29bf8-116">然后，只需从 `OnModelCreating` 调用 `Configure` 方法。</span><span class="sxs-lookup"><span data-stu-id="29bf8-116">Then just invoke the `Configure` method from `OnModelCreating`.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/EntityTypeConfiguration.cs?Name=ApplyIEntityTypeConfiguration)]

<span data-ttu-id="29bf8-117">可以在给定程序集中应用实现 `IEntityTypeConfiguration` 的类型中指定的所有配置。</span><span class="sxs-lookup"><span data-stu-id="29bf8-117">It is possible to apply all configuration specified in types implementing `IEntityTypeConfiguration` in a given assembly.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/FluentAPI/EntityTypeConfiguration.cs?Name=ApplyConfigurationsFromAssembly)]

> [!NOTE]
> <span data-ttu-id="29bf8-118">应用配置的顺序是不确定的，因此仅当顺序不重要时才应使用此方法。</span><span class="sxs-lookup"><span data-stu-id="29bf8-118">The order in which the configurations will be applied is undefined, therefore this method should only be used when the order doesn't matter.</span></span>

## <a name="use-data-annotations-to-configure-a-model"></a><span data-ttu-id="29bf8-119">使用数据注释来配置模型</span><span class="sxs-lookup"><span data-stu-id="29bf8-119">Use data annotations to configure a model</span></span>

<span data-ttu-id="29bf8-120">也可将特性（称为数据注释）应用于类和属性。</span><span class="sxs-lookup"><span data-stu-id="29bf8-120">You can also apply attributes (known as Data Annotations) to your classes and properties.</span></span> <span data-ttu-id="29bf8-121">数据注释会替代约定，但会被 Fluent API 配置替代。</span><span class="sxs-lookup"><span data-stu-id="29bf8-121">Data annotations will override conventions, but will be overridden by Fluent API configuration.</span></span>

[!code-csharp[Main](../../../samples/core/Modeling/DataAnnotations/Required.cs?highlight=15)]
