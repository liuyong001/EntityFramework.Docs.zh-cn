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
# <a name="ef-core-testing-sample"></a><span data-ttu-id="c3db0-103">EF Core 测试示例</span><span class="sxs-lookup"><span data-stu-id="c3db0-103">EF Core testing sample</span></span>

> [!TIP]
> <span data-ttu-id="c3db0-104">可在 GitHub 上找到此文档中的代码作为可 [运行示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Testing/ItemsWebApi/)。</span><span class="sxs-lookup"><span data-stu-id="c3db0-104">The code in this document can be found on GitHub as a [runnable sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Testing/ItemsWebApi/).</span></span>
> <span data-ttu-id="c3db0-105">请注意，其中某些测试 **应该会失败**。</span><span class="sxs-lookup"><span data-stu-id="c3db0-105">Note that some of these tests **are expected to fail**.</span></span> <span data-ttu-id="c3db0-106">下面说明了这种情况的原因。</span><span class="sxs-lookup"><span data-stu-id="c3db0-106">The reasons for this are explained below.</span></span>

<span data-ttu-id="c3db0-107">此文档介绍了用于测试使用 EF Core 的代码的示例。</span><span class="sxs-lookup"><span data-stu-id="c3db0-107">This doc walks through a sample for testing code that uses EF Core.</span></span>

## <a name="the-application"></a><span data-ttu-id="c3db0-108">应用程序</span><span class="sxs-lookup"><span data-stu-id="c3db0-108">The application</span></span>

<span data-ttu-id="c3db0-109">该 [示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Testing/ItemsWebApi/) 包含两个项目：</span><span class="sxs-lookup"><span data-stu-id="c3db0-109">The [sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Testing/ItemsWebApi/) contains two projects:</span></span>

- <span data-ttu-id="c3db0-110">ItemsWebApi：通过单个控制器 [ASP.NET Core 支持](/aspnet/core/tutorials/first-web-api) 的非常简单的 Web API</span><span class="sxs-lookup"><span data-stu-id="c3db0-110">ItemsWebApi: A very simple [Web API backed by ASP.NET Core](/aspnet/core/tutorials/first-web-api) with a single controller</span></span>
- <span data-ttu-id="c3db0-111">测试：用于测试控制器的 [XUnit](https://xunit.net/) 测试项目</span><span class="sxs-lookup"><span data-stu-id="c3db0-111">Tests: An [XUnit](https://xunit.net/) test project to test the controller</span></span>

### <a name="the-model-and-business-rules"></a><span data-ttu-id="c3db0-112">模型和业务规则</span><span class="sxs-lookup"><span data-stu-id="c3db0-112">The model and business rules</span></span>

<span data-ttu-id="c3db0-113">此 API 支持以下两种实体类型： Items 和 Tags 。</span><span class="sxs-lookup"><span data-stu-id="c3db0-113">The model backing this API has two entity types: Items and Tags.</span></span>

- <span data-ttu-id="c3db0-114">Items 具有区分大小写的名称和的集合 Tags 。</span><span class="sxs-lookup"><span data-stu-id="c3db0-114">Items have a case-sensitive name and a collection of Tags.</span></span>
- <span data-ttu-id="c3db0-115">每个 Tag 都有一个标签和一个计数，表示它应用到的次数 Item 。</span><span class="sxs-lookup"><span data-stu-id="c3db0-115">Each Tag has a label and a count representing the number of times it has been applied to the Item.</span></span>
- <span data-ttu-id="c3db0-116">每个 Item 仅应有一个 Tag 具有给定标签的。</span><span class="sxs-lookup"><span data-stu-id="c3db0-116">Each Item should only have one Tag with a given label.</span></span>
  - <span data-ttu-id="c3db0-117">如果多次用相同标签标记项，则会增加带有该标签的现有标记的计数，而不会增加正在创建的新标记。</span><span class="sxs-lookup"><span data-stu-id="c3db0-117">If an item is tagged with the same label more than once, then the count on the existing tag with that label is incremented instead of a new tag being created.</span></span>
- <span data-ttu-id="c3db0-118">删除将 Item 删除所有关联的 Tags 。</span><span class="sxs-lookup"><span data-stu-id="c3db0-118">Deleting an Item should delete all associated Tags.</span></span>

#### <a name="the-no-locitem-entity-type"></a><span data-ttu-id="c3db0-119">Item实体类型</span><span class="sxs-lookup"><span data-stu-id="c3db0-119">The Item entity type</span></span>

<span data-ttu-id="c3db0-120">`Item`实体类型：</span><span class="sxs-lookup"><span data-stu-id="c3db0-120">The `Item` entity type:</span></span>

[!code-csharp[ItemEntityType](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Item.cs?name=ItemEntityType)]

<span data-ttu-id="c3db0-121">及其在中的配置 `DbContext.OnModelCreating` ：</span><span class="sxs-lookup"><span data-stu-id="c3db0-121">And its configuration in `DbContext.OnModelCreating`:</span></span>

[!code-csharp[ConfigureItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/ItemsContext.cs?name=ConfigureItem)]

<span data-ttu-id="c3db0-122">请注意，实体类型限制了它可用于反映域模型和业务规则的方式。</span><span class="sxs-lookup"><span data-stu-id="c3db0-122">Notice that entity type constrains the way it can be used to reflect the domain model and business rules.</span></span> <span data-ttu-id="c3db0-123">具体而言：</span><span class="sxs-lookup"><span data-stu-id="c3db0-123">In particular:</span></span>

- <span data-ttu-id="c3db0-124">主键直接映射到 `_id` 字段，而不公开公开</span><span class="sxs-lookup"><span data-stu-id="c3db0-124">The primary key is mapped directly to the `_id` field and not exposed publicly</span></span>
  - <span data-ttu-id="c3db0-125">EF 检测并使用接受主键值和名称的私有构造函数。</span><span class="sxs-lookup"><span data-stu-id="c3db0-125">EF detects and uses the private constructor accepting the primary key value and name.</span></span>
- <span data-ttu-id="c3db0-126">此 `Name` 属性是只读的，仅在构造函数中设置。</span><span class="sxs-lookup"><span data-stu-id="c3db0-126">The `Name` property is read-only and set only in the constructor.</span></span>
- <span data-ttu-id="c3db0-127">Tags 公开为 `IReadOnlyList<Tag>` 以防止任意修改。</span><span class="sxs-lookup"><span data-stu-id="c3db0-127">Tags are exposed as a `IReadOnlyList<Tag>` to prevent arbitrary modification.</span></span>
  - <span data-ttu-id="c3db0-128">EF `Tags` 通过匹配属性的名称将属性与 `_tags` 支持字段相关联。</span><span class="sxs-lookup"><span data-stu-id="c3db0-128">EF associates the `Tags` property with the `_tags` backing field by matching their names.</span></span>
  - <span data-ttu-id="c3db0-129">`AddTag`方法采用标签标签，并实现上面所述的业务规则。</span><span class="sxs-lookup"><span data-stu-id="c3db0-129">The `AddTag` method takes a tag label and implements the business rule described above.</span></span>
    <span data-ttu-id="c3db0-130">也就是说，只为新标签添加了标记。</span><span class="sxs-lookup"><span data-stu-id="c3db0-130">That is, a tag is only added for new labels.</span></span>
    <span data-ttu-id="c3db0-131">否则，现有标签上的计数将增加。</span><span class="sxs-lookup"><span data-stu-id="c3db0-131">Otherwise the count on an existing label is incremented.</span></span>
- <span data-ttu-id="c3db0-132">为 `Tags` 多对一关系配置导航属性</span><span class="sxs-lookup"><span data-stu-id="c3db0-132">The `Tags` navigation property is configured for a many-to-one relationship</span></span>
  - <span data-ttu-id="c3db0-133">不需要从到的导航属性 Tag Item ，因此不包含。</span><span class="sxs-lookup"><span data-stu-id="c3db0-133">There is no need for a navigation property from Tag to Item, so it is not included.</span></span>
  - <span data-ttu-id="c3db0-134">此外，不 Tag 会定义外键属性。</span><span class="sxs-lookup"><span data-stu-id="c3db0-134">Also, Tag does not define a foreign key property.</span></span>
    <span data-ttu-id="c3db0-135">EF 将创建和管理卷影状态中的属性。</span><span class="sxs-lookup"><span data-stu-id="c3db0-135">Instead, EF will create and manage a property in shadow-state.</span></span>

#### <a name="the-no-loctag-entity-type"></a><span data-ttu-id="c3db0-136">Tag实体类型</span><span class="sxs-lookup"><span data-stu-id="c3db0-136">The Tag entity type</span></span>

<span data-ttu-id="c3db0-137">`Tag`实体类型：</span><span class="sxs-lookup"><span data-stu-id="c3db0-137">The `Tag` entity type:</span></span>

[!code-csharp[TagEntityType](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Tag.cs?name=TagEntityType)]

<span data-ttu-id="c3db0-138">及其在中的配置 `DbContext.OnModelCreating` ：</span><span class="sxs-lookup"><span data-stu-id="c3db0-138">And its configuration in `DbContext.OnModelCreating`:</span></span>

[!code-csharp[ConfigureTag](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/ItemsContext.cs?name=ConfigureTag)]

<span data-ttu-id="c3db0-139">与类似 Item ， Tag 隐藏其主键并使属性成为 `Label` 只读的。</span><span class="sxs-lookup"><span data-stu-id="c3db0-139">Similarly to Item, Tag hides its primary key and makes the `Label` property read-only.</span></span>

### <a name="the-no-locitemscontroller"></a><span data-ttu-id="c3db0-140">Items控制器</span><span class="sxs-lookup"><span data-stu-id="c3db0-140">The ItemsController</span></span>

<span data-ttu-id="c3db0-141">Web API 控制器非常基本。</span><span class="sxs-lookup"><span data-stu-id="c3db0-141">The Web API controller is pretty basic.</span></span>
<span data-ttu-id="c3db0-142">它 `DbContext` 通过构造函数注入从依赖关系注入容器中获取：</span><span class="sxs-lookup"><span data-stu-id="c3db0-142">It gets a `DbContext` from the dependency injection container through constructor injection:</span></span>

[!code-csharp[Constructor](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Controllers/ItemsController.cs?name=Constructor)]

<span data-ttu-id="c3db0-143">它具有获取所有 Items 或 Item 具有给定名称的方法：</span><span class="sxs-lookup"><span data-stu-id="c3db0-143">It has methods to get all Items or an Item with a given name:</span></span>

[!code-csharp[Get](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Controllers/ItemsController.cs?name=Get)]

<span data-ttu-id="c3db0-144">它有一个用于添加新的方法 Item ：</span><span class="sxs-lookup"><span data-stu-id="c3db0-144">It has a method to add a new Item:</span></span>

[!code-csharp[PostItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Controllers/ItemsController.cs?name=PostItem)]

<span data-ttu-id="c3db0-145">用标签标记的方法 Item ：</span><span class="sxs-lookup"><span data-stu-id="c3db0-145">A method to tag an Item with a label:</span></span>

[!code-csharp[PostTag](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Controllers/ItemsController.cs?name=PostTag)]

<span data-ttu-id="c3db0-146">以及用于删除 Item 和全部关联的方法 Tags ：</span><span class="sxs-lookup"><span data-stu-id="c3db0-146">And a method to delete an Item and all associated Tags:</span></span>

[!code-csharp[DeleteItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/ItemsWebApi/Controllers/ItemsController.cs?name=DeleteItem)]

<span data-ttu-id="c3db0-147">为了减少混乱，已经删除了大多数验证和错误处理。</span><span class="sxs-lookup"><span data-stu-id="c3db0-147">Most validation and error handling have been removed to reduce clutter.</span></span>

## <a name="the-tests"></a><span data-ttu-id="c3db0-148">测试</span><span class="sxs-lookup"><span data-stu-id="c3db0-148">The Tests</span></span>

<span data-ttu-id="c3db0-149">这些测试被组织成通过多数据库提供程序配置运行：</span><span class="sxs-lookup"><span data-stu-id="c3db0-149">The tests are organized to run with multiple database provider configurations:</span></span>

- <span data-ttu-id="c3db0-150">SQL Server 提供程序，它是应用程序使用的提供程序</span><span class="sxs-lookup"><span data-stu-id="c3db0-150">The SQL Server provider, which is the provider used by the application</span></span>
- <span data-ttu-id="c3db0-151">SQLite 提供程序</span><span class="sxs-lookup"><span data-stu-id="c3db0-151">The SQLite provider</span></span>
- <span data-ttu-id="c3db0-152">使用内存中 SQLite 数据库的 SQLite 提供程序</span><span class="sxs-lookup"><span data-stu-id="c3db0-152">The SQLite provider using in-memory SQLite databases</span></span>
- <span data-ttu-id="c3db0-153">EF 内存中数据库提供程序</span><span class="sxs-lookup"><span data-stu-id="c3db0-153">The EF in-memory database provider</span></span>

<span data-ttu-id="c3db0-154">为实现此目的，方法是将所有测试放在一个基类中，然后从此继承以测试每个提供程序。</span><span class="sxs-lookup"><span data-stu-id="c3db0-154">This is achieved by putting all the tests in a base class, then inheriting from this to test with each provider.</span></span>

> [!TIP]
> <span data-ttu-id="c3db0-155">如果使用的不是 LocalDB，则需要更改 SQL Server 连接字符串。</span><span class="sxs-lookup"><span data-stu-id="c3db0-155">You will need to change the SQL Server connection string if you're not using LocalDB.</span></span>
> <span data-ttu-id="c3db0-156">有关使用 SQLite 进行内存中测试的指南，请参阅 [使用 sqlite 进行测试](xref:core/testing/sqlite) 。</span><span class="sxs-lookup"><span data-stu-id="c3db0-156">See [Testing with SQLite](xref:core/testing/sqlite) for guidance on using SQLite for in-memory testing.</span></span>

<span data-ttu-id="c3db0-157">以下两个测试应该会失败：</span><span class="sxs-lookup"><span data-stu-id="c3db0-157">The following two tests are expected to fail:</span></span>

- <span data-ttu-id="c3db0-158">`Can_remove_item_and_all_associated_tags` 与 EF 内存中数据库提供程序一起运行时</span><span class="sxs-lookup"><span data-stu-id="c3db0-158">`Can_remove_item_and_all_associated_tags` when running with the EF in-memory database provider</span></span>
- <span data-ttu-id="c3db0-159">`Can_add_item_differing_only_by_case` 与 SQL Server 提供程序一起运行时</span><span class="sxs-lookup"><span data-stu-id="c3db0-159">`Can_add_item_differing_only_by_case` when running with the SQL Server provider</span></span>

<span data-ttu-id="c3db0-160">下面更详细地介绍了这种情况。</span><span class="sxs-lookup"><span data-stu-id="c3db0-160">This is covered in more detail below.</span></span>

### <a name="setting-up-and-seeding-the-database"></a><span data-ttu-id="c3db0-161">设置和播种数据库</span><span class="sxs-lookup"><span data-stu-id="c3db0-161">Setting up and seeding the database</span></span>

<span data-ttu-id="c3db0-162">与大多数测试框架一样，XUnit 将为每个测试运行创建一个新的测试类实例。</span><span class="sxs-lookup"><span data-stu-id="c3db0-162">XUnit, like most testing frameworks, will create a new test class instance for each test run.</span></span>
<span data-ttu-id="c3db0-163">此外，XUnit 不会并行运行给定测试类中的测试。</span><span class="sxs-lookup"><span data-stu-id="c3db0-163">Also, XUnit will not run tests within a given test class in parallel.</span></span>
<span data-ttu-id="c3db0-164">这意味着我们可以在测试构造函数中设置和配置数据库，并且它将处于每个测试的已知状态。</span><span class="sxs-lookup"><span data-stu-id="c3db0-164">This means that we can set up and configure the database in the test constructor and it will be in a well-known state for each test.</span></span>

> [!TIP]
> <span data-ttu-id="c3db0-165">此示例将重新创建每个测试的数据库。</span><span class="sxs-lookup"><span data-stu-id="c3db0-165">This sample recreates the database for each test.</span></span>
> <span data-ttu-id="c3db0-166">这适用于 SQLite 和 EF 内存中数据库测试，但会涉及到与其他数据库系统（包括 SQL Server）的巨大开销。</span><span class="sxs-lookup"><span data-stu-id="c3db0-166">This works well for SQLite and EF in-memory database testing but can involve significant overhead with other database systems, including SQL Server.</span></span>
> <span data-ttu-id="c3db0-167">[跨测试共享数据库](xref:core/testing/sharing-databases)中介绍了降低此开销的方法。</span><span class="sxs-lookup"><span data-stu-id="c3db0-167">Approaches for reducing this overhead are covered in [Sharing databases across tests](xref:core/testing/sharing-databases).</span></span>

<span data-ttu-id="c3db0-168">运行每个测试时：</span><span class="sxs-lookup"><span data-stu-id="c3db0-168">When each test is run:</span></span>

- <span data-ttu-id="c3db0-169">为使用中的提供程序配置 DbContextOptions，并将其传递到基类构造函数</span><span class="sxs-lookup"><span data-stu-id="c3db0-169">DbContextOptions are configured for the provider in use and passed to the base class constructor</span></span>
  - <span data-ttu-id="c3db0-170">这些选项存储在属性中，并在整个测试中用于创建 DbContext 实例</span><span class="sxs-lookup"><span data-stu-id="c3db0-170">These options are stored in a property and used throughout the tests for creating DbContext instances</span></span>
- <span data-ttu-id="c3db0-171">调用种子方法来创建和播种数据库</span><span class="sxs-lookup"><span data-stu-id="c3db0-171">A Seed method is called to create and seed the database</span></span>
  - <span data-ttu-id="c3db0-172">Seed 方法通过删除并重新创建数据库来确保数据库干净</span><span class="sxs-lookup"><span data-stu-id="c3db0-172">The Seed method ensures the database is clean by deleting it and then re-creating it</span></span>
  - <span data-ttu-id="c3db0-173">创建了一些已知的测试实体，并将其保存到数据库中</span><span class="sxs-lookup"><span data-stu-id="c3db0-173">Some well-known test entities are created and saved to the database</span></span>

[!code-csharp[Seeding](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=Seeding)]

<span data-ttu-id="c3db0-174">然后，每个具体的测试类都继承自此。</span><span class="sxs-lookup"><span data-stu-id="c3db0-174">Each concrete test class then inherits from this.</span></span>
<span data-ttu-id="c3db0-175">例如：</span><span class="sxs-lookup"><span data-stu-id="c3db0-175">For example:</span></span>

[!code-csharp[SqliteItemsControllerTest](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/SqliteItemsControllerTest.cs?name=SqliteItemsControllerTest)]

### <a name="test-structure"></a><span data-ttu-id="c3db0-176">测试结构</span><span class="sxs-lookup"><span data-stu-id="c3db0-176">Test structure</span></span>

<span data-ttu-id="c3db0-177">即使应用程序使用依赖关系注入，测试也不是这样。</span><span class="sxs-lookup"><span data-stu-id="c3db0-177">Even though the application uses dependency injection, the tests do not.</span></span>
<span data-ttu-id="c3db0-178">在此处使用依赖项注入会很好，但它需要的附加代码却没有什么价值。</span><span class="sxs-lookup"><span data-stu-id="c3db0-178">It would be fine to use dependency injection here, but the additional code it requires has little value.</span></span>
<span data-ttu-id="c3db0-179">而是使用创建 DbContext， `new` 然后直接作为依赖关系传递到控制器。</span><span class="sxs-lookup"><span data-stu-id="c3db0-179">Instead, a DbContext is created using `new` and then directly passed as the dependency to the controller.</span></span>

<span data-ttu-id="c3db0-180">然后，每个测试在控制器上执行受测方法，并断言结果与预期结果相同。</span><span class="sxs-lookup"><span data-stu-id="c3db0-180">Each test then executes the method under test on the controller and asserts the results are as expected.</span></span>
<span data-ttu-id="c3db0-181">例如：</span><span class="sxs-lookup"><span data-stu-id="c3db0-181">For example:</span></span>

[!code-csharp[CanGetItems](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=CanGetItems)]

<span data-ttu-id="c3db0-182">请注意，不同的 DbContext 实例用于对数据库进行种子设定并运行测试。</span><span class="sxs-lookup"><span data-stu-id="c3db0-182">Notice that different DbContext instances are used to seed the database and run the tests.</span></span>
<span data-ttu-id="c3db0-183">这可确保测试不使用 (，也不会在进行种子设定时接管上下文所跟踪) 实体。</span><span class="sxs-lookup"><span data-stu-id="c3db0-183">This ensures that the test is not using (or tripping over) entities tracked by the context when seeding.</span></span>
<span data-ttu-id="c3db0-184">它还更适合于 web 应用和服务中发生的情况。</span><span class="sxs-lookup"><span data-stu-id="c3db0-184">It also better matches what happens in web apps and services.</span></span>

<span data-ttu-id="c3db0-185">用于改变数据库的测试在测试中创建第二个 DbContext 实例，原因类似。</span><span class="sxs-lookup"><span data-stu-id="c3db0-185">Tests that mutate the database create a second DbContext instance in the test for similar reasons.</span></span>
<span data-ttu-id="c3db0-186">也就是说，创建新的、干净的上下文，然后从数据库中读取该数据库，以确保将更改保存到数据库中。</span><span class="sxs-lookup"><span data-stu-id="c3db0-186">That is, creating a new, clean, context and then reading into it from the database to ensure that the changes were saved to the database.</span></span>
<span data-ttu-id="c3db0-187">例如：</span><span class="sxs-lookup"><span data-stu-id="c3db0-187">For example:</span></span>

[!code-csharp[CanAddItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=CanAddItem)]

<span data-ttu-id="c3db0-188">两个稍微复杂的测试涵盖了有关添加的业务逻辑 tags 。</span><span class="sxs-lookup"><span data-stu-id="c3db0-188">Two slightly more involved tests cover the business logic around adding tags.</span></span>

[!code-csharp[CanAddTag](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=CanAddTag)]

[!code-csharp[CanUpTagCount](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=CanUpTagCount)]

## <a name="issues-using-different-database-providers"></a><span data-ttu-id="c3db0-189">使用不同数据库提供程序时出现的问题</span><span class="sxs-lookup"><span data-stu-id="c3db0-189">Issues using different database providers</span></span>

<span data-ttu-id="c3db0-190">使用与生产应用程序中使用的不同的数据库系统进行测试可能导致出现问题。</span><span class="sxs-lookup"><span data-stu-id="c3db0-190">Testing with a different database system than is used in the production application can lead to problems.</span></span>
<span data-ttu-id="c3db0-191">它们在 [测试使用 EF Core 的代码](xref:core/testing/index)的概念级别中进行了介绍。</span><span class="sxs-lookup"><span data-stu-id="c3db0-191">These are covered at the conceptual level in [Testing code that uses EF Core](xref:core/testing/index).</span></span>
<span data-ttu-id="c3db0-192">以下部分介绍了本示例中的测试演示的两个问题示例。</span><span class="sxs-lookup"><span data-stu-id="c3db0-192">The sections below cover two examples of such issues demonstrated by the tests in this sample.</span></span>

### <a name="test-passes-when-the-application-is-broken"></a><span data-ttu-id="c3db0-193">应用程序中断时的测试通过</span><span class="sxs-lookup"><span data-stu-id="c3db0-193">Test passes when the application is broken</span></span>

<span data-ttu-id="c3db0-194">应用程序的要求之一是 " Items 具有区分大小写的名称和集合 Tags "。</span><span class="sxs-lookup"><span data-stu-id="c3db0-194">One of the requirements for our application is that "Items have a case-sensitive name and a collection of Tags."</span></span>
<span data-ttu-id="c3db0-195">这非常简单，可以测试：</span><span class="sxs-lookup"><span data-stu-id="c3db0-195">This is pretty simple to test:</span></span>

[!code-csharp[CanAddItemCaseInsensitive](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=CanAddItemCaseInsensitive)]

<span data-ttu-id="c3db0-196">对 EF 内存中数据库运行此测试表明一切正常。</span><span class="sxs-lookup"><span data-stu-id="c3db0-196">Running this test against the EF in-memory database indicates that everything is fine.</span></span>
<span data-ttu-id="c3db0-197">使用 SQLite 时，所有内容仍能正常工作。</span><span class="sxs-lookup"><span data-stu-id="c3db0-197">Everything still looks fine when using SQLite.</span></span>
<span data-ttu-id="c3db0-198">但运行时测试会失败 SQL Server！</span><span class="sxs-lookup"><span data-stu-id="c3db0-198">But the test fails when run against SQL Server!</span></span>

```output
System.InvalidOperationException : Sequence contains more than one element
   at System.Linq.ThrowHelper.ThrowMoreThanOneElementException()
   at System.Linq.Enumerable.Single[TSource](IEnumerable`1 source)
   at Microsoft.EntityFrameworkCore.Query.Internal.QueryCompiler.Execute[TResult](Expression query)
   at Microsoft.EntityFrameworkCore.Query.Internal.EntityQueryProvider.Execute[TResult](Expression expression)
   at System.Linq.Queryable.Single[TSource](IQueryable`1 source, Expression`1 predicate)
   at Tests.ItemsControllerTest.Can_add_item_differing_only_by_case()
```

<span data-ttu-id="c3db0-199">这是因为在默认情况下，EF 内存中数据库数据库和 SQLite 数据库都是区分大小写的。</span><span class="sxs-lookup"><span data-stu-id="c3db0-199">This is because both the EF in-memory database and the SQLite database are case-sensitive by default.</span></span>
<span data-ttu-id="c3db0-200">另一方面，SQL Server 不区分大小写！</span><span class="sxs-lookup"><span data-stu-id="c3db0-200">SQL Server, on the other hand, is case-insensitive!</span></span>

<span data-ttu-id="c3db0-201">按照设计，EF Core 不会更改这些行为，因为强制进行区分大小写可能会对性能产生重大影响。</span><span class="sxs-lookup"><span data-stu-id="c3db0-201">EF Core, by design, does not change these behaviors because forcing a change in case-sensitivity can have a big performance impact.</span></span>

<span data-ttu-id="c3db0-202">知道这是一个问题，我们可以修复应用程序并在测试中进行补偿。</span><span class="sxs-lookup"><span data-stu-id="c3db0-202">Once we know this is a problem we can fix the application and compensate in tests.</span></span>
<span data-ttu-id="c3db0-203">但是，此处的要点是，如果仅通过 EF 内存中数据库或 SQLite 提供程序进行测试，则可能会丢失此 bug。</span><span class="sxs-lookup"><span data-stu-id="c3db0-203">However, the point here is that this bug could be missed if only testing with the EF in-memory database or SQLite providers.</span></span>

### <a name="test-fails-when-the-application-is-correct"></a><span data-ttu-id="c3db0-204">当应用程序正确时测试失败</span><span class="sxs-lookup"><span data-stu-id="c3db0-204">Test fails when the application is correct</span></span>

<span data-ttu-id="c3db0-205">我们的应用程序的另一个要求是 "删除" Item 应删除所有关联的 Tags 。 "</span><span class="sxs-lookup"><span data-stu-id="c3db0-205">Another of the requirements for our application is that "deleting an Item should delete all associated Tags."</span></span>
<span data-ttu-id="c3db0-206">同样，易于测试：</span><span class="sxs-lookup"><span data-stu-id="c3db0-206">Again, easy to test:</span></span>

[!code-csharp[DeleteItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/Tests/ItemsControllerTest.cs?name=DeleteItem)]

<span data-ttu-id="c3db0-207">此测试通过 SQL Server 和 SQLite，但对于 EF 内存中数据库失败！</span><span class="sxs-lookup"><span data-stu-id="c3db0-207">This test passes on SQL Server and SQLite, but fails with the EF in-memory database!</span></span>

```output
Assert.False() Failure
Expected: False
Actual:   True
   at Tests.ItemsControllerTest.Can_remove_item_and_all_associated_tags()
```

<span data-ttu-id="c3db0-208">在这种情况下，应用程序将正常工作，因为 SQL Server 支持 [级联删除](xref:core/saving/cascade-delete)。</span><span class="sxs-lookup"><span data-stu-id="c3db0-208">In this case, the application is working correctly because SQL Server supports [cascade deletes](xref:core/saving/cascade-delete).</span></span>
<span data-ttu-id="c3db0-209">SQLite 还支持级联删除，这与大多数关系数据库一样，因此，可以在 SQLite 上进行测试。</span><span class="sxs-lookup"><span data-stu-id="c3db0-209">SQLite also supports cascade deletes, as do most relational databases, so testing this on SQLite works.</span></span>
<span data-ttu-id="c3db0-210">另一方面，EF 内存中数据库不 [支持级联删除](https://github.com/dotnet/efcore/issues/3924)。</span><span class="sxs-lookup"><span data-stu-id="c3db0-210">On the other hand, the EF in-memory database [does not support cascade deletes](https://github.com/dotnet/efcore/issues/3924).</span></span>
<span data-ttu-id="c3db0-211">这意味着应用程序的此部分不能通过 EF 内存中数据库提供程序进行测试。</span><span class="sxs-lookup"><span data-stu-id="c3db0-211">This means that this part of the application cannot be tested with the EF in-memory database provider.</span></span>
