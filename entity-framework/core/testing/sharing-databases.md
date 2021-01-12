---
title: 在测试之间共享数据库-EF Core
description: 示例演示如何在多个测试之间共享数据库
author: ajcvickers
ms.date: 04/25/2020
uid: core/testing/sharing-databases
ms.openlocfilehash: 7a90a144271d5c34e9d5043aa439f84db805c6af
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128831"
---
# <a name="sharing-databases-between-tests"></a><span data-ttu-id="295f7-103">在测试之间共享数据库</span><span class="sxs-lookup"><span data-stu-id="295f7-103">Sharing databases between tests</span></span>

<span data-ttu-id="295f7-104">[EF Core 测试示例](xref:core/testing/testing-sample)演示了如何针对不同的数据库系统测试应用程序。</span><span class="sxs-lookup"><span data-stu-id="295f7-104">The [EF Core testing sample](xref:core/testing/testing-sample) showed how to test applications against different database systems.</span></span>
<span data-ttu-id="295f7-105">对于该示例，每个测试都会创建一个新数据库。</span><span class="sxs-lookup"><span data-stu-id="295f7-105">For that sample, each test created a new database.</span></span>
<span data-ttu-id="295f7-106">当使用 SQLite 或 EF 内存中数据库时，这是一个很好的模式，但在使用其他数据库系统时，这种模式可能会造成很大的开销。</span><span class="sxs-lookup"><span data-stu-id="295f7-106">This is a good pattern when using SQLite or the EF in-memory database, but it can involve significant overhead when using other database systems.</span></span>

<span data-ttu-id="295f7-107">此示例通过将数据库创建移到测试装置中来构建前面的示例。</span><span class="sxs-lookup"><span data-stu-id="295f7-107">This sample builds on the previous sample by moving database creation into a test fixture.</span></span>
<span data-ttu-id="295f7-108">这允许创建单个 SQL Server 数据库，并只为所有测试创建一次。</span><span class="sxs-lookup"><span data-stu-id="295f7-108">This allows a single SQL Server database to be created and seeded only once for all tests.</span></span>

> [!TIP]
> <span data-ttu-id="295f7-109">在继续操作之前，请务必完成 [EF Core 测试示例](xref:core/testing/testing-sample) 。</span><span class="sxs-lookup"><span data-stu-id="295f7-109">Make sure to work through the [EF Core testing sample](xref:core/testing/testing-sample) before continuing here.</span></span>

<span data-ttu-id="295f7-110">为同一个数据库编写多个测试并不困难。</span><span class="sxs-lookup"><span data-stu-id="295f7-110">It's not difficult to write multiple tests against the same database.</span></span>
<span data-ttu-id="295f7-111">这一技巧以测试在运行时不会相互行程的方式进行操作。</span><span class="sxs-lookup"><span data-stu-id="295f7-111">The trick is doing it in a way that the tests don't trip over each other as they run.</span></span>
<span data-ttu-id="295f7-112">这需要了解：</span><span class="sxs-lookup"><span data-stu-id="295f7-112">This requires understanding:</span></span>

* <span data-ttu-id="295f7-113">如何在测试之间安全共享对象</span><span class="sxs-lookup"><span data-stu-id="295f7-113">How to safely share objects between tests</span></span>
* <span data-ttu-id="295f7-114">当测试框架并行运行测试时</span><span class="sxs-lookup"><span data-stu-id="295f7-114">When the test framework runs tests in parallel</span></span>
* <span data-ttu-id="295f7-115">如何使数据库处于每个测试的干净状态</span><span class="sxs-lookup"><span data-stu-id="295f7-115">How to keep the database in a clean state for every test</span></span>

## <a name="the-fixture"></a><span data-ttu-id="295f7-116">装置</span><span class="sxs-lookup"><span data-stu-id="295f7-116">The fixture</span></span>

<span data-ttu-id="295f7-117">我们将使用测试装置在测试之间共享对象。</span><span class="sxs-lookup"><span data-stu-id="295f7-117">We will use a test fixture for sharing objects between tests.</span></span>
<span data-ttu-id="295f7-118">当你想要创建单个测试上下文并将其共享到类中的所有测试中并在类中的所有测试都完成后， [XUnit 文档](https://xunit.net/docs/shared-context.html) 将指出应使用的装置。 "</span><span class="sxs-lookup"><span data-stu-id="295f7-118">The [XUnit documentation](https://xunit.net/docs/shared-context.html) states that a fixture should be used "when you want to create a single test context and share it among all the tests in the class, and have it cleaned up after all the tests in the class have finished."</span></span>

> [!TIP]
> <span data-ttu-id="295f7-119">此示例使用的是 [XUnit](https://xunit.net/)，但其他测试框架中也存在类似的概念，包括 [NUnit](https://nunit.org/)。</span><span class="sxs-lookup"><span data-stu-id="295f7-119">This sample uses [XUnit](https://xunit.net/), but similar concepts exist in other testing frameworks, including [NUnit](https://nunit.org/).</span></span>

<span data-ttu-id="295f7-120">这意味着我们需要将数据库创建和种子设定转移到夹具类。</span><span class="sxs-lookup"><span data-stu-id="295f7-120">This means that we need to move database creation and seeding to a fixture class.</span></span>
<span data-ttu-id="295f7-121">它的外观如下所示：</span><span class="sxs-lookup"><span data-stu-id="295f7-121">Here's what it looks like:</span></span>

[!code-csharp[SharedDatabaseFixture](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/SharedDatabaseTests/SharedDatabaseFixture.cs?name=SharedDatabaseFixture)]

<span data-ttu-id="295f7-122">现在，请注意构造函数的方式：</span><span class="sxs-lookup"><span data-stu-id="295f7-122">For now, notice how the constructor:</span></span>

* <span data-ttu-id="295f7-123">在装置的生存期内创建单个数据库连接</span><span class="sxs-lookup"><span data-stu-id="295f7-123">Creates a single database connection for the lifetime of the fixture</span></span>
* <span data-ttu-id="295f7-124">通过调用方法创建并设定该数据库的种子 `Seed`</span><span class="sxs-lookup"><span data-stu-id="295f7-124">Creates and seeds that database by calling the `Seed` method</span></span>

<span data-ttu-id="295f7-125">立即忽略锁定;稍后我们将返回到它。</span><span class="sxs-lookup"><span data-stu-id="295f7-125">Ignore the locking for now; we will come back to it later.</span></span>

> [!TIP]
> <span data-ttu-id="295f7-126">创建和播种代码不需要是异步的。</span><span class="sxs-lookup"><span data-stu-id="295f7-126">The creation and seeding code does not need to be async.</span></span>
> <span data-ttu-id="295f7-127">将其设为异步会使代码变得复杂，而不会提高测试的性能或吞吐量。</span><span class="sxs-lookup"><span data-stu-id="295f7-127">Making it async will complicate the code and will not improve performance or throughput of tests.</span></span>

<span data-ttu-id="295f7-128">数据库是通过先删除所有现有数据库，然后创建新的数据库创建的。</span><span class="sxs-lookup"><span data-stu-id="295f7-128">The database is created by first deleting any existing database and then creating a new database.</span></span>
<span data-ttu-id="295f7-129">这可确保数据库与当前 EF 模型匹配，即使自上次测试运行以来已更改。</span><span class="sxs-lookup"><span data-stu-id="295f7-129">This ensures that the database matches the current EF model even if it has been changed since the last test run.</span></span>

> [!TIP]
> <span data-ttu-id="295f7-130">使用 [respawn](https://jimmybogard.com/tag/respawn/) 之类的内容（而不是每次重新创建），可以更快地 "清理" 现有数据库。</span><span class="sxs-lookup"><span data-stu-id="295f7-130">It can be faster to "clean" the existing database using something like [respawn](https://jimmybogard.com/tag/respawn/) rather than re-create it each time.</span></span>
> <span data-ttu-id="295f7-131">但是，在执行此操作时，必须注意确保数据库架构与 EF 模型保持最新。</span><span class="sxs-lookup"><span data-stu-id="295f7-131">However, care must be taken to ensure that the database schema is up-to-date with the EF model when doing this.</span></span>

<span data-ttu-id="295f7-132">释放装置后，数据库连接会被释放。</span><span class="sxs-lookup"><span data-stu-id="295f7-132">The database connection is disposed when the fixture is disposed.</span></span>
<span data-ttu-id="295f7-133">此时，您还可以考虑删除测试数据库。</span><span class="sxs-lookup"><span data-stu-id="295f7-133">You may also consider deleting the test database at this point.</span></span>
<span data-ttu-id="295f7-134">但是，如果多个测试类正在共享该装置，则这将需要额外的锁定和引用计数。</span><span class="sxs-lookup"><span data-stu-id="295f7-134">However, this will require additional locking and reference counting if the fixture is being shared by multiple test classes.</span></span>
<span data-ttu-id="295f7-135">此外，使测试数据库仍可用于调试失败的测试通常是很有用的。</span><span class="sxs-lookup"><span data-stu-id="295f7-135">Also, it is often useful to have the test database still available for debugging failed tests.</span></span>

## <a name="using-the-fixture"></a><span data-ttu-id="295f7-136">使用装置</span><span class="sxs-lookup"><span data-stu-id="295f7-136">Using the fixture</span></span>

<span data-ttu-id="295f7-137">XUnit 具有一个通用模式，用于将测试装置与一类测试进行关联：</span><span class="sxs-lookup"><span data-stu-id="295f7-137">XUnit has a common pattern for associating a test fixture with a class of tests:</span></span>

[!code-csharp[UsingTheFixture](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/SharedDatabaseTests/SharedDatabaseTest.cs?name=UsingTheFixture)]

<span data-ttu-id="295f7-138">XUnit 现在将创建单个夹具实例，并将其传递给测试类的每个实例。</span><span class="sxs-lookup"><span data-stu-id="295f7-138">XUnit will now create a single fixture instance and pass it to each instance of the test class.</span></span>
<span data-ttu-id="295f7-139"> (请 [注意，XUnit](xref:core/testing/testing-sample) 将在每次运行测试时创建新的测试类实例。 ) 这意味着将创建数据库并对其进行种子设定，然后每个测试都将使用此数据库。</span><span class="sxs-lookup"><span data-stu-id="295f7-139">(Remember from the first [testing sample](xref:core/testing/testing-sample) that XUnit creates a new test class instance every time it runs a test.) This means that the database will be created and seeded once and then each test will use this database.</span></span>

<span data-ttu-id="295f7-140">请注意，不会并行运行单个类中的测试。</span><span class="sxs-lookup"><span data-stu-id="295f7-140">Note that tests within a single class will not be run in parallel.</span></span>
<span data-ttu-id="295f7-141">这意味着，即使 `DbConnection` 对象不是线程安全的，每个测试都可以安全地使用同一个数据库连接。</span><span class="sxs-lookup"><span data-stu-id="295f7-141">This means it is safe for each test to use the same database connection, even though the `DbConnection` object is not thread-safe.</span></span>

## <a name="maintaining-database-state"></a><span data-ttu-id="295f7-142">维护数据库状态</span><span class="sxs-lookup"><span data-stu-id="295f7-142">Maintaining database state</span></span>

<span data-ttu-id="295f7-143">测试通常需要对测试数据进行插入、更新和删除操作。</span><span class="sxs-lookup"><span data-stu-id="295f7-143">Tests often need to mutate the test data with inserts, updates, and deletes.</span></span>
<span data-ttu-id="295f7-144">但这些更改会影响到需要干净的种子数据库的其他测试。</span><span class="sxs-lookup"><span data-stu-id="295f7-144">But these changes will then impact other tests which are expecting a clean, seeded database.</span></span>

<span data-ttu-id="295f7-145">这可以通过在事务中运行转变测试来处理。</span><span class="sxs-lookup"><span data-stu-id="295f7-145">This can be dealt with by running mutating tests inside a transaction.</span></span>
<span data-ttu-id="295f7-146">例如：</span><span class="sxs-lookup"><span data-stu-id="295f7-146">For example:</span></span>

[!code-csharp[CanAddItem](../../../samples/core/Miscellaneous/Testing/ItemsWebApi/SharedDatabaseTests/SharedDatabaseTest.cs?name=CanAddItem)]

<span data-ttu-id="295f7-147">请注意，在测试完成后将创建事务，并将其释放。</span><span class="sxs-lookup"><span data-stu-id="295f7-147">Notice that the transaction is created as the test starts and disposed when it is finished.</span></span>
<span data-ttu-id="295f7-148">释放事务将导致回滚该事务，因此其他测试将看不到任何更改。</span><span class="sxs-lookup"><span data-stu-id="295f7-148">Disposing the transaction causes it to be rolled back, so none of the changes will be seen by other tests.</span></span>

<span data-ttu-id="295f7-149">用于创建上下文的帮助器方法 (参阅上述装置代码) 接受此交易并使用它来 DbContext。</span><span class="sxs-lookup"><span data-stu-id="295f7-149">The helper method for creating a context (see the fixture code above) accepts this transaction and opts the DbContext into using it.</span></span>

## <a name="sharing-the-fixture"></a><span data-ttu-id="295f7-150">共享装置</span><span class="sxs-lookup"><span data-stu-id="295f7-150">Sharing the fixture</span></span>

<span data-ttu-id="295f7-151">您可能已注意到锁定了有关数据库创建和播种的代码。</span><span class="sxs-lookup"><span data-stu-id="295f7-151">You may have noticed locking code around database creation and seeding.</span></span>
<span data-ttu-id="295f7-152">这不是此示例所必需的，因为只有一类测试使用装置，因此仅创建单个装置实例。</span><span class="sxs-lookup"><span data-stu-id="295f7-152">This is not needed for this sample since only one class of tests use the fixture, so only a single fixture instance is created.</span></span>

<span data-ttu-id="295f7-153">但是，你可能想要将相同的装置用于多个测试类别。</span><span class="sxs-lookup"><span data-stu-id="295f7-153">However, you may want to use the same fixture with multiple classes of tests.</span></span>
<span data-ttu-id="295f7-154">XUnit 将为这些类中的每个类创建一个装置实例。</span><span class="sxs-lookup"><span data-stu-id="295f7-154">XUnit will create one fixture instance for each of these classes.</span></span>
<span data-ttu-id="295f7-155">并行运行测试的不同线程可能会使用这些方法。</span><span class="sxs-lookup"><span data-stu-id="295f7-155">These may be used by different threads running tests in parallel.</span></span>
<span data-ttu-id="295f7-156">因此，请务必使用适当的锁定，以确保只有一个线程可以创建和播种数据库。</span><span class="sxs-lookup"><span data-stu-id="295f7-156">Therefore, it is important to have appropriate locking to ensure only one thread does the database creation and seeding.</span></span>

> [!TIP]
> <span data-ttu-id="295f7-157">这里简单明了 `lock` 。</span><span class="sxs-lookup"><span data-stu-id="295f7-157">A simple `lock` is fine here.</span></span>
> <span data-ttu-id="295f7-158">无需尝试更复杂的操作，如任何无锁模式。</span><span class="sxs-lookup"><span data-stu-id="295f7-158">There is no need to attempt anything more complex, such as any lock-free patterns.</span></span>
