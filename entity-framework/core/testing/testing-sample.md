---
title: EF Core 测试示例-EF Core
description: 示例演示如何测试使用 Entity Framework Core 的应用程序
author: ajcvickers
ms.date: 04/22/2020
uid: core/testing/testing-sample
no-loc:
- Item
- Tag
- Items
- Tags
- items
- tags
ms.openlocfilehash: 7af516421a6ec3040b636507a6cd7976cabce3a9
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128792"
---
# <a name="ef-core-testing-sample"></a>EF Core 测试示例

> [!TIP]
> 可在 GitHub 上找到此文档中的代码作为可 [运行示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Testing/ItemsWebApi/)。
> 请注意，其中某些测试 **应该会失败**。 下面说明了这种情况的原因。

此文档介绍了用于测试使用 EF Core 的代码的示例。

## <a name="the-application"></a>应用程序

该 [示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Testing/ItemsWebApi/) 包含两个项目：

- ItemsWebApi：通过单个控制器 [ASP.NET Core 支持](/aspnet/core/tutorials/first-web-api) 的非常简单的 Web API
- 测试：用于测试控制器的 [XUnit](https://xunit.net/) 测试项目

### <a name="the-model-and-business-rules"></a>模型和业务规则

此 API 支持以下两种实体类型： Items 和 Tags 。

- Items 具有区分大小写的名称和的集合 Tags 。
- 每个 Tag 都有一个标签和一个计数，表示它应用到的次数 Item 。
- 每个 Item 仅应有一个 Tag 具有给定标签的。
  - 如果多次用相同标签标记项，则会增加带有该标签的现有标记的计数，而不会增加正在创建的新标记。
- 删除将 Item 删除所有关联的 Tags 。

#### <a name="the-no-locitem-entity-type"></a>Item实体类型

`Item`实体类型：

[!code-csharp[ItemEntityType](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Item.cs?name=ItemEntityType)]

及其在中的配置 `DbContext.OnModelCreating` ：

[!code-csharp[ConfigureItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/ItemsContext.cs?name=ConfigureItem)]

请注意，实体类型限制了它可用于反映域模型和业务规则的方式。 具体而言：

- 主键直接映射到 `_id` 字段，而不公开公开
  - EF 检测并使用接受主键值和名称的私有构造函数。
- 此 `Name` 属性是只读的，仅在构造函数中设置。
- Tags 公开为 `IReadOnlyList<Tag>` 以防止任意修改。
  - EF `Tags` 通过匹配属性的名称将属性与 `_tags` 支持字段相关联。
  - `AddTag`方法采用标签标签，并实现上面所述的业务规则。
    也就是说，只为新标签添加了标记。
    否则，现有标签上的计数将增加。
- 为 `Tags` 多对一关系配置导航属性
  - 不需要从到的导航属性 Tag Item ，因此不包含。
  - 此外，不 Tag 会定义外键属性。
    EF 将创建和管理卷影状态中的属性。

#### <a name="the-no-loctag-entity-type"></a>Tag实体类型

`Tag`实体类型：

[!code-csharp[TagEntityType](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Tag.cs?name=TagEntityType)]

及其在中的配置 `DbContext.OnModelCreating` ：

[!code-csharp[ConfigureTag](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/ItemsContext.cs?name=ConfigureTag)]

与类似 Item ， Tag 隐藏其主键并使属性成为 `Label` 只读的。

### <a name="the-no-locitemscontroller"></a>Items控制器

Web API 控制器非常基本。
它 `DbContext` 通过构造函数注入从依赖关系注入容器中获取：

[!code-csharp[Constructor](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Controllers/ItemsController.cs?name=Constructor)]

它具有获取所有 Items 或 Item 具有给定名称的方法：

[!code-csharp[Get](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Controllers/ItemsController.cs?name=Get)]

它有一个用于添加新的方法 Item ：

[!code-csharp[PostItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Controllers/ItemsController.cs?name=PostItem)]

用标签标记的方法 Item ：

[!code-csharp[PostTag](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Controllers/ItemsController.cs?name=PostTag)]

以及用于删除 Item 和全部关联的方法 Tags ：

[!code-csharp[DeleteItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Controllers/ItemsController.cs?name=DeleteItem)]

为了减少混乱，已经删除了大多数验证和错误处理。

## <a name="the-tests"></a>测试

这些测试被组织成通过多数据库提供程序配置运行：

- SQL Server 提供程序，它是应用程序使用的提供程序
- SQLite 提供程序
- 使用内存中 SQLite 数据库的 SQLite 提供程序
- EF 内存中数据库提供程序

为实现此目的，方法是将所有测试放在一个基类中，然后从此继承以测试每个提供程序。

> [!TIP]
> 如果使用的不是 LocalDB，则需要更改 SQL Server 连接字符串。
> 有关使用 SQLite 进行内存中测试的指南，请参阅 [使用 sqlite 进行测试](xref:core/testing/sqlite) 。

以下两个测试应该会失败：

- `Can_remove_item_and_all_associated_tags` 与 EF 内存中数据库提供程序一起运行时
- `Can_add_item_differing_only_by_case` 与 SQL Server 提供程序一起运行时

下面更详细地介绍了这种情况。

### <a name="setting-up-and-seeding-the-database"></a>设置和播种数据库

与大多数测试框架一样，XUnit 将为每个测试运行创建一个新的测试类实例。
此外，XUnit 不会并行运行给定测试类中的测试。
这意味着我们可以在测试构造函数中设置和配置数据库，并且它将处于每个测试的已知状态。

> [!TIP]
> 此示例将重新创建每个测试的数据库。
> 这适用于 SQLite 和 EF 内存中数据库测试，但会涉及到与其他数据库系统（包括 SQL Server）的巨大开销。
> [跨测试共享数据库](xref:core/testing/sharing-databases)中介绍了降低此开销的方法。

运行每个测试时：

- 为使用中的提供程序配置 DbContextOptions，并将其传递到基类构造函数
  - 这些选项存储在属性中，并在整个测试中用于创建 DbContext 实例
- 调用种子方法来创建和播种数据库
  - Seed 方法通过删除并重新创建数据库来确保数据库干净
  - 创建了一些已知的测试实体，并将其保存到数据库中

[!code-csharp[Seeding](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=Seeding)]

然后，每个具体的测试类都继承自此。
例如：

[!code-csharp[SqliteItemsControllerTest](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/SqliteItemsControllerTest.cs?name=SqliteItemsControllerTest)]

### <a name="test-structure"></a>测试结构

即使应用程序使用依赖关系注入，测试也不是这样。
在此处使用依赖项注入会很好，但它需要的附加代码却没有什么价值。
而是使用创建 DbContext， `new` 然后直接作为依赖关系传递到控制器。

然后，每个测试在控制器上执行受测方法，并断言结果与预期结果相同。
例如：

[!code-csharp[CanGetItems](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=CanGetItems)]

请注意，不同的 DbContext 实例用于对数据库进行种子设定并运行测试。
这可确保测试不使用 (，也不会在进行种子设定时接管上下文所跟踪) 实体。
它还更适合于 web 应用和服务中发生的情况。

用于改变数据库的测试在测试中创建第二个 DbContext 实例，原因类似。
也就是说，创建新的、干净的上下文，然后从数据库中读取该数据库，以确保将更改保存到数据库中。
例如：

[!code-csharp[CanAddItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=CanAddItem)]

两个稍微复杂的测试涵盖了有关添加的业务逻辑 tags 。

[!code-csharp[CanAddTag](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=CanAddTag)]

[!code-csharp[CanUpTagCount](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=CanUpTagCount)]

## <a name="issues-using-different-database-providers"></a>使用不同数据库提供程序时出现的问题

使用与生产应用程序中使用的不同的数据库系统进行测试可能导致出现问题。
它们在 [测试使用 EF Core 的代码](xref:core/testing/index)的概念级别中进行了介绍。
以下部分介绍了本示例中的测试演示的两个问题示例。

### <a name="test-passes-when-the-application-is-broken"></a>应用程序中断时的测试通过

应用程序的要求之一是 " Items 具有区分大小写的名称和集合 Tags "。
这非常简单，可以测试：

[!code-csharp[CanAddItemCaseInsensitive](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=CanAddItemCaseInsensitive)]

对 EF 内存中数据库运行此测试表明一切正常。
使用 SQLite 时，所有内容仍能正常工作。
但运行时测试会失败 SQL Server！

```output
System.InvalidOperationException : Sequence contains more than one element
   at System.Linq.ThrowHelper.ThrowMoreThanOneElementException()
   at System.Linq.Enumerable.Single[TSource](IEnumerable`1 source)
   at Microsoft.EntityFrameworkCore.Query.Internal.QueryCompiler.Execute[TResult](Expression query)
   at Microsoft.EntityFrameworkCore.Query.Internal.EntityQueryProvider.Execute[TResult](Expression expression)
   at System.Linq.Queryable.Single[TSource](IQueryable`1 source, Expression`1 predicate)
   at Tests.ItemsControllerTest.Can_add_item_differing_only_by_case()
```

这是因为在默认情况下，EF 内存中数据库数据库和 SQLite 数据库都是区分大小写的。
另一方面，SQL Server 不区分大小写！

按照设计，EF Core 不会更改这些行为，因为强制进行区分大小写可能会对性能产生重大影响。

知道这是一个问题，我们可以修复应用程序并在测试中进行补偿。
但是，此处的要点是，如果仅通过 EF 内存中数据库或 SQLite 提供程序进行测试，则可能会丢失此 bug。

### <a name="test-fails-when-the-application-is-correct"></a>当应用程序正确时测试失败

我们的应用程序的另一个要求是 "删除" Item 应删除所有关联的 Tags 。 "
同样，易于测试：

[!code-csharp[DeleteItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=DeleteItem)]

此测试通过 SQL Server 和 SQLite，但对于 EF 内存中数据库失败！

```output
Assert.False() Failure
Expected: False
Actual:   True
   at Tests.ItemsControllerTest.Can_remove_item_and_all_associated_tags()
```

在这种情况下，应用程序将正常工作，因为 SQL Server 支持 [级联删除](xref:core/saving/cascade-delete)。
SQLite 还支持级联删除，这与大多数关系数据库一样，因此，可以在 SQLite 上进行测试。
另一方面，EF 内存中数据库不 [支持级联删除](https://github.com/dotnet/efcore/issues/3924)。
这意味着应用程序的此部分不能通过 EF 内存中数据库提供程序进行测试。
