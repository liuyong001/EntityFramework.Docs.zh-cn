---
title: 测试使用 EF Core 的代码 - EF Core
description: 测试使用 Entity Framework Core 的应用程序的不同方法
author: ajcvickers
ms.date: 04/22/2020
uid: core/testing/index
ms.openlocfilehash: d94c40bdb1082473ee88e7c3c1fbfecee90ba4ff
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98129065"
---
# <a name="testing-code-that-uses-ef-core"></a><span data-ttu-id="d8c5d-103">测试使用 EF Core 的代码</span><span class="sxs-lookup"><span data-stu-id="d8c5d-103">Testing code that uses EF Core</span></span>

<span data-ttu-id="d8c5d-104">测试访问数据库的代码需要满足以下任一条件：</span><span class="sxs-lookup"><span data-stu-id="d8c5d-104">Testing code that accesses a database requires either:</span></span>

* <span data-ttu-id="d8c5d-105">针对在生产环境中使用的相同数据库系统运行查询和更新。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-105">Running queries and updates against the same database system used in production.</span></span>
* <span data-ttu-id="d8c5d-106">对其他更易管理的数据库系统运行查询和更新。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-106">Running queries and updates against some other easier to manage database system.</span></span>
* <span data-ttu-id="d8c5d-107">使用测试替身或其他机制来避免使用数据库。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-107">Using test doubles or some other mechanism to avoid using a database at all.</span></span>

<span data-ttu-id="d8c5d-108">本文档概述了每个选项所涉及的折衷方案，并说明了如何在每种方法中使用 EF Core。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-108">This document outlines the trade-offs involved in each of these choices and shows how EF Core can be used with each approach.</span></span>

> [!TIP]
> <span data-ttu-id="d8c5d-109">请参阅 [EF Core 测试示例](xref:core/testing/testing-sample)查看演示此处引入的概念的代码。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-109">See [EF Core testing sample](xref:core/testing/testing-sample) for code demonstrating the concepts introduced here.</span></span>

## <a name="all-database-providers-are-not-equal"></a><span data-ttu-id="d8c5d-110">所有数据库提供程序都不等同</span><span class="sxs-lookup"><span data-stu-id="d8c5d-110">All database providers are not equal</span></span>

<span data-ttu-id="d8c5d-111">务必要理解的是，EF Core 不是为了抽象底层数据库系统的各个方面而设计的。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-111">It is very important to understand that EF Core is not designed to abstract every aspect of the underlying database system.</span></span>
<span data-ttu-id="d8c5d-112">EF Core 是一组通用的模式和概念，可用于任何数据库系统。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-112">Instead, EF Core is a common set of patterns and concepts that can be used with any database system.</span></span>
<span data-ttu-id="d8c5d-113">然后，EF Core 数据库提供程序在此通用框架的基础上叠加数据库特定的行为和功能。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-113">EF Core database providers then layer database-specific behavior and functionality over this common framework.</span></span>
<span data-ttu-id="d8c5d-114">这样，每个数据库系统都可以实现其最佳性能，并在适当的情况下维持与其他数据库系统的共性。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-114">This allows each database system to do what it does best while still maintaining commonality, where appropriate, with other database systems.</span></span>

<span data-ttu-id="d8c5d-115">从根本上说，这意味着切换数据库提供程序将更改 EF Core 行为，使应用程序不能正常工作，除非它明确解释行为的任何差异。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-115">Fundamentally, this means that switching out the database provider will change EF Core behavior and the application can't be expected to function correctly unless it explicitly accounts for any differences in behavior.</span></span>
<span data-ttu-id="d8c5d-116">虽然如此，但在许多情况下还是可以这么做，因为关系数据库之间存在高度的共性。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-116">That being said, in many cases doing this will work because there is a high degree of commonality amongst relational databases.</span></span>
<span data-ttu-id="d8c5d-117">这有利也有弊。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-117">This is good and bad.</span></span>
<span data-ttu-id="d8c5d-118">其有利之处在于，可以相对轻松地切换数据库系统。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-118">Good because moving between database systems can be relatively easy.</span></span>
<span data-ttu-id="d8c5d-119">而其弊端在于，如果未针对新的数据库系统彻底测试应用程序，会有一种虚假的安全感。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-119">Bad because it can give a false sense of security if the application is not fully tested against the new database system.</span></span>

## <a name="approach-1-production-database-system"></a><span data-ttu-id="d8c5d-120">方法 1：生产数据库系统</span><span class="sxs-lookup"><span data-stu-id="d8c5d-120">Approach 1: Production database system</span></span>

<span data-ttu-id="d8c5d-121">如前一部分所述，要确保测试在生产中运行的内容，唯一的方法是使用同一个数据库系统。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-121">As described in the previous section, the only way to be sure you are testing what runs in production is to use the same database system.</span></span>
<span data-ttu-id="d8c5d-122">例如，如果部署的应用程序使用 SQL Azure，则测试也应针对 SQL Azure 进行。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-122">For example, if the deployed application uses SQL Azure, then testing should also be done against SQL Azure.</span></span>

<span data-ttu-id="d8c5d-123">但是，要让每个开发者正在积极处理代码的同时针对 SQL Azure 运行测试，不仅工作速度慢而且代价高昂。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-123">However, having every developer run tests against SQL Azure while actively working on the code would be both slow and expensive.</span></span>
<span data-ttu-id="d8c5d-124">以下问题描述了这些方法中涉及的主要折衷方案：什么时候适合偏离生产数据库系统以提高测试效率？</span><span class="sxs-lookup"><span data-stu-id="d8c5d-124">This illustrates the main trade-off involved throughout these approaches: when is it appropriate to deviate from the production database system so as to improve test efficiency?</span></span>

<span data-ttu-id="d8c5d-125">幸运的是，在本例中，答案非常简单：使用本地 SQL Server 进行开发者测试。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-125">Luckily, in this case the answer is quite easy: use local or on-premises SQL Server for developer testing.</span></span>
<span data-ttu-id="d8c5d-126">SQL Azure 和 SQL Server 极其相似，因此针对 SQL Server 的测试通常是合理的折衷方案。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-126">SQL Azure and SQL Server are extremely similar, so testing against SQL Server is usually a reasonable trade-off.</span></span>
<span data-ttu-id="d8c5d-127">虽然如此，但在投入生产之前针对 SQL Azure 本身运行测试仍然是明智之举。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-127">That being said, it is still wise to run tests against SQL Azure itself before going into production.</span></span>

### <a name="localdb"></a><span data-ttu-id="d8c5d-128">LocalDB</span><span class="sxs-lookup"><span data-stu-id="d8c5d-128">LocalDB</span></span>

<span data-ttu-id="d8c5d-129">所有主要数据库系统都具有某种形式的“Developer Edition”本地测试。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-129">All the major database systems have some form of "Developer Edition" for local testing.</span></span>
<span data-ttu-id="d8c5d-130">SQL Server 还有一项名为 [LocalDB](/sql/database-engine/configure-windows/sql-server-express-localdb) 的功能。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-130">SQL Server also has a feature called [LocalDB](/sql/database-engine/configure-windows/sql-server-express-localdb).</span></span>
<span data-ttu-id="d8c5d-131">LocalDB 的主要优势在于可以按需增大数据库实例。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-131">The primary advantage of LocalDB is that it spins up the database instance on demand.</span></span>
<span data-ttu-id="d8c5d-132">这可以避免在计算机上运行数据库服务，即使在未运行测试时也是如此。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-132">This avoids having a database service running on your machine even when you're not running tests.</span></span>

<span data-ttu-id="d8c5d-133">LocalDB 也不是没有问题：</span><span class="sxs-lookup"><span data-stu-id="d8c5d-133">LocalDB is not without its issues:</span></span>

* <span data-ttu-id="d8c5d-134">它不支持 [SQL Server Developer Edition](/sql/sql-server/editions-and-components-of-sql-server-version-15?view=sql-server-ver15&preserve-view=true) 支持的部分内容。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-134">It doesn't support everything that [SQL Server Developer Edition](/sql/sql-server/editions-and-components-of-sql-server-version-15?view=sql-server-ver15&preserve-view=true) does.</span></span>
* <span data-ttu-id="d8c5d-135">它在 Linux 上不可用。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-135">It isn't available on Linux.</span></span>
* <span data-ttu-id="d8c5d-136">由于要启动服务，可能导致首次测试运行滞后。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-136">It can cause lag on first test run as the service is spun up.</span></span>

<span data-ttu-id="d8c5d-137">就个人而言，我从来不觉得在开发计算机上运行数据库服务有什么问题，所以一般建议使用 Developer Edition。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-137">Personally, I've never found it a problem having a database service running on my dev machine and I would generally recommend using Developer Edition instead.</span></span>
<span data-ttu-id="d8c5d-138">但是，LocalDB 对某些人而言可能适用，尤其是在处理能力不够强的开发计算机上。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-138">However, LocalDB may be appropriate for some people, especially on less powerful dev machines.</span></span>

<span data-ttu-id="d8c5d-139">在 Docker 容器（或类似容器）中[运行 SQL Server](/sql/linux/quickstart-install-connect-docker)（或任何其他数据库系统）是避免直接在开发计算机上运行数据库系统的另一种方法。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-139">[Running SQL Server](/sql/linux/quickstart-install-connect-docker) (or any other database system) in a Docker container (or similar) is another way to avoid running the database system directly on your development machine.</span></span>

## <a name="approach-2-sqlite"></a><span data-ttu-id="d8c5d-140">方法 2：SQLite</span><span class="sxs-lookup"><span data-stu-id="d8c5d-140">Approach 2: SQLite</span></span>

<span data-ttu-id="d8c5d-141">EF Core 主要通过针对本地 SQL Server 实例运行 SQL Server 提供程序来对其进行测试。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-141">EF Core tests the SQL Server provider primarily by running it against a local SQL Server instance.</span></span>
<span data-ttu-id="d8c5d-142">这些测试将在几分钟内在一台高速计算机上运行数万次查询。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-142">These tests run tens of thousands of queries in a couple of minutes on a fast machine.</span></span>
<span data-ttu-id="d8c5d-143">这说明，使用实际数据库系统可能是一种高性能的解决方案。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-143">This illustrates that using the real database system can be a performant solution.</span></span>
<span data-ttu-id="d8c5d-144">有一种错误的观念是，使用较小型的数据库是快速运行测试的唯一方法。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-144">It is a myth that using some lighter-weight database is the only way to run tests quickly.</span></span>

<span data-ttu-id="d8c5d-145">话虽如此，但如果由于某些原因，无法针对接近生产数据库系统的系统运行测试，应该怎么办？</span><span class="sxs-lookup"><span data-stu-id="d8c5d-145">That being said, what if for whatever reason you can't run tests against something close to your production database system?</span></span>
<span data-ttu-id="d8c5d-146">下一个最佳选择是使用具有相似功能的对象。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-146">The next best choice is to use something with similar functionality.</span></span>
<span data-ttu-id="d8c5d-147">这通常意味着另一个关系数据库，[SQLite](https://sqlite.org/index.html) 是显而易见的选择。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-147">This usually means another relational database, for which [SQLite](https://sqlite.org/index.html) is the obvious choice.</span></span>

<span data-ttu-id="d8c5d-148">SQLite 是不错的选择，因为：</span><span class="sxs-lookup"><span data-stu-id="d8c5d-148">SQLite is a good choice because:</span></span>

* <span data-ttu-id="d8c5d-149">它在处理应用程序的过程中运行，因此开销较低。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-149">It runs in-process with your application and so has low overhead.</span></span>
* <span data-ttu-id="d8c5d-150">它使用自动创建的简单数据库文件，因此不需要数据库管理。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-150">It uses simple, automatically created files for databases, and so doesn't require database management.</span></span>
* <span data-ttu-id="d8c5d-151">它有内存模式，甚至可以避免创建文件。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-151">It has an in-memory mode that avoids even the file creation.</span></span>

<span data-ttu-id="d8c5d-152">但是，请记住：</span><span class="sxs-lookup"><span data-stu-id="d8c5d-152">However, remember that:</span></span>

* <span data-ttu-id="d8c5d-153">SQLite 不一定支持你的生产数据库系统支持的所有内容，这是不可避免的。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-153">SQLite inevitability doesn't support everything that your production database system does.</span></span>
* <span data-ttu-id="d8c5d-154">对于某些查询，SQLite 的行为方式将与你的生产数据库系统不同。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-154">SQLite will behave differently than your production database system for some queries.</span></span>

<span data-ttu-id="d8c5d-155">因此，如果使用 SQLite 进行测试，还应确保针对实际数据库系统进行测试。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-155">So if you do use SQLite for some testing, make sure to also test against your real database system.</span></span>

<span data-ttu-id="d8c5d-156">有关 EF Core 特定指南，请参阅[用 SQLite 进行测试](xref:core/testing/sqlite)。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-156">See [Testing with SQLite](xref:core/testing/sqlite) for EF Core specific guidance.</span></span>

## <a name="approach-3-the-ef-core-in-memory-database"></a><span data-ttu-id="d8c5d-157">方法 3：EF Core 内存中数据库</span><span class="sxs-lookup"><span data-stu-id="d8c5d-157">Approach 3: The EF Core in-memory database</span></span>

<span data-ttu-id="d8c5d-158">EF Core 附带了内存中数据库，用于对 EF Core 本身进行内部测试。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-158">EF Core comes with an in-memory database that we use for internal testing of EF Core itself.</span></span>
<span data-ttu-id="d8c5d-159">此数据库一般不适合用于测试使用 EF Core 的应用。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-159">This database is in general **not suitable for testing applications that use EF Core**.</span></span> <span data-ttu-id="d8c5d-160">尤其是在下列情况下：</span><span class="sxs-lookup"><span data-stu-id="d8c5d-160">Specifically:</span></span>

* <span data-ttu-id="d8c5d-161">它不是关系数据库。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-161">It is not a relational database.</span></span>
* <span data-ttu-id="d8c5d-162">它不支持事务。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-162">It doesn't support transactions.</span></span>
* <span data-ttu-id="d8c5d-163">它无法运行原始 SQL 查询。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-163">It cannot run raw SQL queries.</span></span>
* <span data-ttu-id="d8c5d-164">它未针对性能进行优化。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-164">It is not optimized for performance.</span></span>

<span data-ttu-id="d8c5d-165">在测试 EF Core 内部机制时，这些都不重要，因为我们只在数据库与测试不相关时才会使用它。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-165">None of this is very important when testing EF Core internals because we use it specifically where the database is irrelevant to the test.</span></span>
<span data-ttu-id="d8c5d-166">另一方面，在测试使用 EF Core 的应用程序时，这些特性往往非常重要。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-166">On the other hand, these things tend to be very important when testing an application that uses EF Core.</span></span>

## <a name="unit-testing"></a><span data-ttu-id="d8c5d-167">单元测试</span><span class="sxs-lookup"><span data-stu-id="d8c5d-167">Unit testing</span></span>

<span data-ttu-id="d8c5d-168">考虑这种情况：你要测试可能需要使用数据库中的某些数据的业务逻辑，但该测试本身并不测试数据库交互。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-168">Consider testing a piece of business logic that might need to use some data from a database, but is not inherently testing the database interactions.</span></span>
<span data-ttu-id="d8c5d-169">可以选择使用[测试替身](https://en.wikipedia.org/wiki/Test_double)，例如模拟或虚构数据库。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-169">One option is to use a [test double](https://en.wikipedia.org/wiki/Test_double) such as a mock or fake.</span></span>

<span data-ttu-id="d8c5d-170">我们使用测试替身进行 EF Core 的内部测试。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-170">We use test doubles for internal testing of EF Core.</span></span>
<span data-ttu-id="d8c5d-171">但是，我们不会尝试模拟 DbContext 或 IQueryable。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-171">However, we never try to mock DbContext or IQueryable.</span></span>
<span data-ttu-id="d8c5d-172">这样做难度大、过程繁琐且结果不可靠。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-172">Doing so is difficult, cumbersome, and fragile.</span></span>
<span data-ttu-id="d8c5d-173">不要这样做。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-173">**Don't do it.**</span></span>

<span data-ttu-id="d8c5d-174">改为使用 EF 内存中数据库对使用 DbContext 的应用进行单元测试。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-174">Instead we use the EF in-memory database when unit testing something that uses DbContext.</span></span>
<span data-ttu-id="d8c5d-175">在这种情况下，使用 EF 内存中数据库是适当的，因为测试不依赖于数据库行为。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-175">In this case using the EF in-memory database is appropriate because the test is not dependent on database behavior.</span></span>
<span data-ttu-id="d8c5d-176">但是不要这样测试实际数据库查询或更新。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-176">Just don't do this to test actual database queries or updates.</span></span>

<span data-ttu-id="d8c5d-177">[EF Core 测试示例](xref:core/testing/testing-sample)演示使用 EF 内存中数据库以及 SQL Server 和 SQLite 进行的测试。</span><span class="sxs-lookup"><span data-stu-id="d8c5d-177">The [EF Core testing sample](xref:core/testing/testing-sample) demonstrates tests using the EF in-memory database, as well as SQL Server and SQLite.</span></span>
