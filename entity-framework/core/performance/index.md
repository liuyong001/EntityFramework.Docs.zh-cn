---
title: 性能简介 - EF Core
description: 关于有效使用 Entity Framework Core 的性能指南
author: roji
ms.date: 12/1/2020
uid: core/miscellaneous/performance/index
ms.openlocfilehash: 14400d81ea3c93e2ebf40e8e585a457abf31478f
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97657683"
---
# <a name="introduction-to-performance"></a>性能简介

数据库性能是一个宏大而复杂的主题，涉及整个组件堆栈：数据库、网络、数据库驱动程序和数据访问层（如 EF Core）。 尽管高级层和 O/RM（如 EF Core）大大简化了应用程序开发并改善了可维护性，但它们有时可能是不透明的，隐藏了性能关键的内部详细信息，例如正在执行的 SQL。 本部分将尝试概述如何通过 EF Core 实现良好的性能，以及如何避免可能会降低应用程序性能的常见缺陷。

## <a name="identify-bottlenecks-and-measure-measure-measure"></a>确定瓶颈并再三衡量

对于性能而言始终不变的是，没有数据显示问题就不要急着优化。正如高德纳所说，“过早优化是万恶之源”。 [性能诊断](xref:core/performance/performance-diagnosis)部分介绍了各种方法，可用于了解应用程序在数据库逻辑中的哪些位置最耗时，还介绍了如何确定有问题的具体部分。 确定缓慢查询后，可以考虑使用解决方案：数据库是否缺少索引？ 是否应尝试其他查询模式？

始终对你的代码和可能的替代项进行基准测试 - 性能诊断部分包含 BenchmarkDotNet 的示例基准测试，可将其用作你的基准测试的模板。 不要假定通用的公共基准会适用于你的特定用例；各种因素（例如数据库延迟、查询复杂性和表中的实际数据量）都可能对最佳解决方案的选择产生深远影响。 例如，很多公共基准测试都是在理想网络条件下执行的，到数据库的延迟几乎为零，并且使用极轻量的查询，在数据库端几乎不需要任何处理（或磁盘 I/O）。 虽然这些基准测试对于比较不同数据访问层的运行时开销很有价值，但它们所揭示的差异在实际应用程序中通常可以忽略不计，因为在实际情况下，由数据库执行实际工作，到数据库的延迟是非常重要的性能因素。

## <a name="aspects-of-data-access-performance"></a>数据访问性能的各个方面

总体数据访问性能可细分为以下几个大类：

* **纯数据库性能**。 对于关系数据库，EF 将应用程序的 LINQ 查询转换为 SQL 语句供数据库执行；这些 SQL 语句本身的效率可以更高，也可以更低。 适当位置的适当索引可能对 SQL 性能产生天翻地覆的影响，重写 LINQ 查询也可能使 EF 生成更好的 SQL 查询。
* **网络数据传输**。 就像在任何网络系统中一样，必须限制通过网线来回传输的数据量。 这涉及到确保只发送和加载实际需要的数据，还有避免在加载相关实体时出现所谓的“笛卡尔爆炸”效应。
* **网络往返**。 除了来回传输的数据量，还要考虑网络往返，因为与在应用程序和数据库之间来回传输数据包的时间相比，在数据库中执行查询所用的时间是微不足道的。 往返开销在很大程度上取决于环境：数据库服务器越远，延迟越高，越次往返的成本越高。 随着云的出现，应用程序与数据库的距离越来越远，而“健谈”的应用程序（即进行往返过多的应用程序）会出现性能下降。 因此，请务必了解应用程序与数据库联系的确切时间、它所执行的往返次数，以及是否可以最大限度地减少该数值。
* **EF 运行时开销**。 最后，EF 本身也会为数据库操作增加一些运行时开销：EF 需要将查询从 LINQ 编译为 SQL（尽管通常只会执行一次），更改跟踪也会增加一些开销（但可以禁用），此外还有其他开销。在实践中，大多数情况下可以忽略实际应用程序的 EF 开销，因为数据库中的查询执行时间和网络延迟时间占总时间的很大一部分；但了解你有哪些选项以及如何避免某些缺陷很重要。

## <a name="know-whats-happening-under-the-hood"></a>了解幕后发生的情况

EF 允许开发人员生成 SQL、具体化结果和执行其他任务，以便专注于业务逻辑。 与任何层或抽象一样，它也会隐藏幕后发生的情况，例如执行的实际 SQL 查询。 性能对某些应用程序而言并非关键，但在性能关键的应用程序中，开发人员一定要了解 EF 在幕后进行的工作：检查传出的 SQL 查询，跟进往返以确保没有发生 N+1 问题等。

## <a name="cache-outside-the-database"></a>数据库外的缓存

最后，与数据库交互最高效的方法是，完全不与它交互。 换句话说，如果数据库访问表现为应用程序中的性能瓶颈，则可能需要在数据库外缓存某些结果，以便将请求数量降到最低。 虽然缓存增加了复杂性，但它对任何可缩放应用程序都特别重要：尽管可以添加更多服务器来轻松扩展应用程序层以处理增加的负载，但缩放数据库层通常要复杂得多。