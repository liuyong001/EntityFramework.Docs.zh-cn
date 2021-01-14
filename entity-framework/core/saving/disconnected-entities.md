---
title: 断开连接的实体 - EF Core
description: 在 Entity Framework Core 中跨多个上下文实例使用断开连接的、未跟踪的实体
author: ajcvickers
ms.date: 10/27/2016
uid: core/saving/disconnected-entities
ms.openlocfilehash: 4ddae08b68aa99fb81de77fed8a195c60927e0bb
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128844"
---
# <a name="disconnected-entities"></a><span data-ttu-id="de76e-103">断开连接的实体</span><span class="sxs-lookup"><span data-stu-id="de76e-103">Disconnected entities</span></span>

<span data-ttu-id="de76e-104">DbContext 实例将自动跟踪从数据库返回的实体。</span><span class="sxs-lookup"><span data-stu-id="de76e-104">A DbContext instance will automatically track entities returned from the database.</span></span> <span data-ttu-id="de76e-105">调用 SaveChanges 时，系统将检测对这些实体所做的更改并根据需要更新数据库。</span><span class="sxs-lookup"><span data-stu-id="de76e-105">Changes made to these entities will then be detected when SaveChanges is called and the database will be updated as needed.</span></span> <span data-ttu-id="de76e-106">有关详细信息，请参阅[基本保存](xref:core/saving/basic)和[相关数据](xref:core/saving/related-data)。</span><span class="sxs-lookup"><span data-stu-id="de76e-106">See [Basic Save](xref:core/saving/basic) and [Related Data](xref:core/saving/related-data) for details.</span></span>

<span data-ttu-id="de76e-107">但是，有时会使用一个上下文实例查询实体，然后使用其他实例对其进行保存。</span><span class="sxs-lookup"><span data-stu-id="de76e-107">However, sometimes entities are queried using one context instance and then saved using a different instance.</span></span> <span data-ttu-id="de76e-108">这通常在“断开连接”的情况下发生，例如 Web 应用程序，此情况下实体被查询、发送到客户端、被修改、在请求中发送回服务器，然后进行保存。</span><span class="sxs-lookup"><span data-stu-id="de76e-108">This often happens in "disconnected" scenarios such as a web application where the entities are queried, sent to the client, modified, sent back to the server in a request, and then saved.</span></span> <span data-ttu-id="de76e-109">在这种情况下，第二个上下文实例需要知道实体是新的（应插入）还是现有的（应更新）。</span><span class="sxs-lookup"><span data-stu-id="de76e-109">In this case, the second context instance needs to know whether the entities are new (should be inserted) or existing (should be updated).</span></span>

<!-- markdownlint-disable MD028 -->
> [!TIP]
> <span data-ttu-id="de76e-110">可在 GitHub 上查看此文章的[示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Saving/Disconnected/)。</span><span class="sxs-lookup"><span data-stu-id="de76e-110">You can view this article's [sample](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Saving/Disconnected/) on GitHub.</span></span>

> [!TIP]
> <span data-ttu-id="de76e-111">EF Core 只能跟踪具有给定主键值的任何实体的一个实例。</span><span class="sxs-lookup"><span data-stu-id="de76e-111">EF Core can only track one instance of any entity with a given primary key value.</span></span> <span data-ttu-id="de76e-112">为避免这种情况成为一个问题，最好为每个工作单元使用短期上下文，使上下文一开始为空、向其附加实体、保存这些实体，然后释放并放弃该上下文。</span><span class="sxs-lookup"><span data-stu-id="de76e-112">The best way to avoid this being an issue is to use a short-lived context for each unit-of-work such that the context starts empty, has entities attached to it, saves those entities, and then the context is disposed and discarded.</span></span>
<!-- markdownlint-enable MD028 -->

## <a name="identifying-new-entities"></a><span data-ttu-id="de76e-113">标识新实体</span><span class="sxs-lookup"><span data-stu-id="de76e-113">Identifying new entities</span></span>

### <a name="client-identifies-new-entities"></a><span data-ttu-id="de76e-114">客户端标识新实体</span><span class="sxs-lookup"><span data-stu-id="de76e-114">Client identifies new entities</span></span>

<span data-ttu-id="de76e-115">由客户端通知服务器实体是新实体还是现有实体，这是最简单的情况。</span><span class="sxs-lookup"><span data-stu-id="de76e-115">The simplest case to deal with is when the client informs the server whether the entity is new or existing.</span></span> <span data-ttu-id="de76e-116">例如，通常插入新实体的请求与更新现有实体的请求不同。</span><span class="sxs-lookup"><span data-stu-id="de76e-116">For example, often the request to insert a new entity is different from the request to update an existing entity.</span></span>

<span data-ttu-id="de76e-117">本节的其余部分介绍了需要以其他某种方式确定是插入还是更新的情况。</span><span class="sxs-lookup"><span data-stu-id="de76e-117">The remainder of this section covers the cases where it necessary to determine in some other way whether to insert or update.</span></span>

### <a name="with-auto-generated-keys"></a><span data-ttu-id="de76e-118">使用自动生成的键</span><span class="sxs-lookup"><span data-stu-id="de76e-118">With auto-generated keys</span></span>

<span data-ttu-id="de76e-119">自动生成的键的值通常用于确定是需要插入实体还是需要更新实体。</span><span class="sxs-lookup"><span data-stu-id="de76e-119">The value of an automatically generated key can often be used to determine whether an entity needs to be inserted or updated.</span></span> <span data-ttu-id="de76e-120">如果未设置键（即 CLR 默认值仍为 null、零等），则实体必须为新的，且需要插入。</span><span class="sxs-lookup"><span data-stu-id="de76e-120">If the key has not been set (that is, it still has the CLR default value of null, zero, etc.), then the entity must be new and needs inserting.</span></span> <span data-ttu-id="de76e-121">另一方面，如果已设置键值，则必须之前已保存该实体，且现在需要更新。</span><span class="sxs-lookup"><span data-stu-id="de76e-121">On the other hand, if the key value has been set, then it must have already been previously saved and now needs updating.</span></span> <span data-ttu-id="de76e-122">换而言之，如果键具有值，则实体被查询、发送到客户端、现在返回进行更新。</span><span class="sxs-lookup"><span data-stu-id="de76e-122">In other words, if the key has a value, then the entity was queried, sent to the client, and has now come back to be updated.</span></span>

<span data-ttu-id="de76e-123">实体类型为未知时，可以轻松检查未设置的键：</span><span class="sxs-lookup"><span data-stu-id="de76e-123">It is easy to check for an unset key when the entity type is known:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#IsItNewSimple)]

<span data-ttu-id="de76e-124">但是，EF 还有一种适用于任何实体类型和键类型的内置方法可用于执行此操作：</span><span class="sxs-lookup"><span data-stu-id="de76e-124">However, EF also has a built-in way to do this for any entity type and key type:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#IsItNewGeneral)]

> [!TIP]
> <span data-ttu-id="de76e-125">即使实体处于“Added”状态，只要实体由上下文跟踪，就会设置键。</span><span class="sxs-lookup"><span data-stu-id="de76e-125">Keys are set as soon as entities are tracked by the context, even if the entity is in the Added state.</span></span> <span data-ttu-id="de76e-126">这有助于遍历实体图并决定如何处理每个实体（例如在使用 TrackGraph API 时）。</span><span class="sxs-lookup"><span data-stu-id="de76e-126">This helps when traversing a graph of entities and deciding what to do with each, such as when using the TrackGraph API.</span></span> <span data-ttu-id="de76e-127">键值只能以此处显示的方式使用，然后才能执行任何调用以跟踪实体。</span><span class="sxs-lookup"><span data-stu-id="de76e-127">The key value should only be used in the way shown here _before_ any call is made to track the entity.</span></span>

### <a name="with-other-keys"></a><span data-ttu-id="de76e-128">使用其他键</span><span class="sxs-lookup"><span data-stu-id="de76e-128">With other keys</span></span>

<span data-ttu-id="de76e-129">未自动生成键值时，需要使用其他某种机制来确定新实体。</span><span class="sxs-lookup"><span data-stu-id="de76e-129">Some other mechanism is needed to identify new entities when key values are not generated automatically.</span></span> <span data-ttu-id="de76e-130">有以下两种常规方法：</span><span class="sxs-lookup"><span data-stu-id="de76e-130">There are two general approaches to this:</span></span>

* <span data-ttu-id="de76e-131">查询实体</span><span class="sxs-lookup"><span data-stu-id="de76e-131">Query for the entity</span></span>
* <span data-ttu-id="de76e-132">从客户端传递标志</span><span class="sxs-lookup"><span data-stu-id="de76e-132">Pass a flag from the client</span></span>

<span data-ttu-id="de76e-133">若要查询实体，只需使用 Find 方法：</span><span class="sxs-lookup"><span data-stu-id="de76e-133">To query for the entity, just use the Find method:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#IsItNewQuery)]

<span data-ttu-id="de76e-134">无法显示用于从客户端传递标志的完整代码，这超出了本文档的范围。</span><span class="sxs-lookup"><span data-stu-id="de76e-134">It is beyond the scope of this document to show the full code for passing a flag from a client.</span></span> <span data-ttu-id="de76e-135">在 Web 应用中，这通常意味着对不同操作发出不同请求，或者在请求中传递某些状态，然后在控制器中进行提取。</span><span class="sxs-lookup"><span data-stu-id="de76e-135">In a web app, it usually means making different requests for different actions, or passing some state in the request then extracting it in the controller.</span></span>

## <a name="saving-single-entities"></a><span data-ttu-id="de76e-136">保存单个实体</span><span class="sxs-lookup"><span data-stu-id="de76e-136">Saving single entities</span></span>

<span data-ttu-id="de76e-137">如果知道是需要插入还是需要更新，则可以相应地使用 Add 或 Update：</span><span class="sxs-lookup"><span data-stu-id="de76e-137">If it is known whether or not an insert or update is needed, then either Add or Update can be used appropriately:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#InsertAndUpdateSingleEntity)]

<span data-ttu-id="de76e-138">但是，如果实体使用自动生成的键值，则 Update 方法可以用于以下两种情况：</span><span class="sxs-lookup"><span data-stu-id="de76e-138">However, if the entity uses auto-generated key values, then the Update method can be used for both cases:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#InsertOrUpdateSingleEntity)]

<span data-ttu-id="de76e-139">Update 方法通常将实体标记为更新，而不是插入。</span><span class="sxs-lookup"><span data-stu-id="de76e-139">The Update method normally marks the entity for update, not insert.</span></span> <span data-ttu-id="de76e-140">但是，如果实体具有自动生成的键且未设置任何键值，则实体会自动标记为插入。</span><span class="sxs-lookup"><span data-stu-id="de76e-140">However, if the entity has a auto-generated key, and no key value has been set, then the entity is instead automatically marked for insert.</span></span>

<span data-ttu-id="de76e-141">如果实体不使用自动生成的键，则应用程序必须确定是应插入实体还是应更新实体：例如：</span><span class="sxs-lookup"><span data-stu-id="de76e-141">If the entity is not using auto-generated keys, then the application must decide whether the entity should be inserted or updated: For example:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#InsertOrUpdateSingleEntityWithFind)]

<span data-ttu-id="de76e-142">步骤如下：</span><span class="sxs-lookup"><span data-stu-id="de76e-142">The steps here are:</span></span>

* <span data-ttu-id="de76e-143">如果 Find 返回 null，则数据库尚未包含具有此 ID 的博客，因此对 Add 的调用会将其标记为插入。</span><span class="sxs-lookup"><span data-stu-id="de76e-143">If Find returns null, then the database doesn't already contain the blog with this ID, so we call Add mark it for insertion.</span></span>
* <span data-ttu-id="de76e-144">如果 Find 返回实体，则该实体存在于数据库中，且上下文现在正在跟踪现有实体</span><span class="sxs-lookup"><span data-stu-id="de76e-144">If Find returns an entity, then it exists in the database and the context is now tracking the existing entity</span></span>
  * <span data-ttu-id="de76e-145">然后，使用 SetValues 将此实体上所有属性的值设置为来自客户端的值。</span><span class="sxs-lookup"><span data-stu-id="de76e-145">We then use SetValues to set the values for all properties on this entity to those that came from the client.</span></span>
  * <span data-ttu-id="de76e-146">SetValues 调用将根据需要标记要更新的实体。</span><span class="sxs-lookup"><span data-stu-id="de76e-146">The SetValues call will mark the entity to be updated as needed.</span></span>

> [!TIP]
> <span data-ttu-id="de76e-147">SetValues 仅将与跟踪实体中的属性具有不同值的属性标记为“已修改”。</span><span class="sxs-lookup"><span data-stu-id="de76e-147">SetValues will only mark as modified the properties that have different values to those in the tracked entity.</span></span> <span data-ttu-id="de76e-148">这意味着当发送更新时，这意味着当发送更新时，只会更新实际发生更改的列。</span><span class="sxs-lookup"><span data-stu-id="de76e-148">This means that when the update is sent, only those columns that have actually changed will be updated.</span></span> <span data-ttu-id="de76e-149">（如果未发生更改，则根本不会发送任何更新。）</span><span class="sxs-lookup"><span data-stu-id="de76e-149">(And if nothing has changed, then no update will be sent at all.)</span></span>

## <a name="working-with-graphs"></a><span data-ttu-id="de76e-150">使用图形</span><span class="sxs-lookup"><span data-stu-id="de76e-150">Working with graphs</span></span>

### <a name="identity-resolution"></a><span data-ttu-id="de76e-151">标识解析</span><span class="sxs-lookup"><span data-stu-id="de76e-151">Identity resolution</span></span>

<span data-ttu-id="de76e-152">如上所述，EF Core 只能跟踪具有给定主键值的任何实体的一个实例。</span><span class="sxs-lookup"><span data-stu-id="de76e-152">As noted above, EF Core can only track one instance of any entity with a given primary key value.</span></span> <span data-ttu-id="de76e-153">使用图形时，理想情况下应创建图形，以便保留此固定对象，且上下文应仅用于一个工作单元。</span><span class="sxs-lookup"><span data-stu-id="de76e-153">When working with graphs the graph should ideally be created such that this invariant is maintained, and the context should be used for only one unit-of-work.</span></span> <span data-ttu-id="de76e-154">如果图形中包含重复项，则必须先处理该图形，然后再将其发送到 EF 以合并多个实例。</span><span class="sxs-lookup"><span data-stu-id="de76e-154">If the graph does contain duplicates, then it will be necessary to process the graph before sending it to EF to consolidate multiple instances into one.</span></span> <span data-ttu-id="de76e-155">如果实例具有冲突值和关系，则以上操作可能并不轻松，因此应尽快在应用程序管道中完成重复项合并以避免冲突解决。</span><span class="sxs-lookup"><span data-stu-id="de76e-155">This may not be trivial where instances have conflicting values and relationships, so consolidating duplicates should be done as soon as possible in your application pipeline to avoid conflict resolution.</span></span>

### <a name="all-newall-existing-entities"></a><span data-ttu-id="de76e-156">所有新实体/所有现有实体</span><span class="sxs-lookup"><span data-stu-id="de76e-156">All new/all existing entities</span></span>

<span data-ttu-id="de76e-157">插入或更新博客及其相关文章的集合是使用图形的一个示例。</span><span class="sxs-lookup"><span data-stu-id="de76e-157">An example of working with graphs is inserting or updating a blog together with its collection of associated posts.</span></span> <span data-ttu-id="de76e-158">如果应插入图形中的所有实体，或应更新所有这些实体，则该过程与上述单个实体的过程相同。</span><span class="sxs-lookup"><span data-stu-id="de76e-158">If all the entities in the graph should be inserted, or all should be updated, then the process is the same as described above for single entities.</span></span> <span data-ttu-id="de76e-159">例如，博客和文章的图形创建如下：</span><span class="sxs-lookup"><span data-stu-id="de76e-159">For example, a graph of blogs and posts created like this:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#CreateBlogAndPosts)]

<span data-ttu-id="de76e-160">可按如下方式插入：</span><span class="sxs-lookup"><span data-stu-id="de76e-160">can be inserted like this:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#InsertGraph)]

<span data-ttu-id="de76e-161">对 Add 的调用会将博客和所有文章标记为插入。</span><span class="sxs-lookup"><span data-stu-id="de76e-161">The call to Add will mark the blog and all the posts to be inserted.</span></span>

<span data-ttu-id="de76e-162">同样，如果图形中的所有实体都需要更新，则可以使用 Update：</span><span class="sxs-lookup"><span data-stu-id="de76e-162">Likewise, if all the entities in a graph need to be updated, then Update can be used:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#UpdateGraph)]

<span data-ttu-id="de76e-163">博客及其所有文章将标记为更新。</span><span class="sxs-lookup"><span data-stu-id="de76e-163">The blog and all its posts will be marked to be updated.</span></span>

### <a name="mix-of-new-and-existing-entities"></a><span data-ttu-id="de76e-164">新实体和现有实体的组合</span><span class="sxs-lookup"><span data-stu-id="de76e-164">Mix of new and existing entities</span></span>

<span data-ttu-id="de76e-165">即使图形包含需要插入的实体和需要更新的实体的组合，使用自动生成的键，Update 也可以再次用于插入和更新：</span><span class="sxs-lookup"><span data-stu-id="de76e-165">With auto-generated keys, Update can again be used for both inserts and updates, even if the graph contains a mix of entities that require inserting and those that require updating:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#InsertOrUpdateGraph)]

<span data-ttu-id="de76e-166">如果图形、博客或文章中的任何实体未设置键值，Update 会将其标记为插入，而其他所有实体会标记为更新。</span><span class="sxs-lookup"><span data-stu-id="de76e-166">Update will mark any entity in the graph, blog or post, for insertion if it does not have a key value set, while all other entities are marked for update.</span></span>

<span data-ttu-id="de76e-167">如前所述，不使用自动生成的键时，可以使用查询并进行一些处理：</span><span class="sxs-lookup"><span data-stu-id="de76e-167">As before, when not using auto-generated keys, a query and some processing can be used:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#InsertOrUpdateGraphWithFind)]

## <a name="handling-deletes"></a><span data-ttu-id="de76e-168">处理删除</span><span class="sxs-lookup"><span data-stu-id="de76e-168">Handling deletes</span></span>

<span data-ttu-id="de76e-169">由于实体不存在通常表示应删除该实体，因此删除可能很难处理。</span><span class="sxs-lookup"><span data-stu-id="de76e-169">Delete can be tricky to handle since often the absence of an entity means that it should be deleted.</span></span> <span data-ttu-id="de76e-170">解决此问题的一种方法是使用“软删除”，以便将该实体标记为已删除，而不是实际删除。</span><span class="sxs-lookup"><span data-stu-id="de76e-170">One way to deal with this is to use "soft deletes" such that the entity is marked as deleted rather than actually being deleted.</span></span> <span data-ttu-id="de76e-171">然后，删除将变得与更新相同。</span><span class="sxs-lookup"><span data-stu-id="de76e-171">Deletes then becomes the same as updates.</span></span> <span data-ttu-id="de76e-172">可以使用[查询筛选器](xref:core/querying/filters)实现软删除。</span><span class="sxs-lookup"><span data-stu-id="de76e-172">Soft deletes can be implemented in using [query filters](xref:core/querying/filters).</span></span>

<span data-ttu-id="de76e-173">对于真删除，常见模式是使用查询模式的扩展来执行图形差异真正的内容。</span><span class="sxs-lookup"><span data-stu-id="de76e-173">For true deletes, a common pattern is to use an extension of the query pattern to perform what is essentially a graph diff.</span></span> <span data-ttu-id="de76e-174">例如：</span><span class="sxs-lookup"><span data-stu-id="de76e-174">For example:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#InsertUpdateOrDeleteGraphWithFind)]

## <a name="trackgraph"></a><span data-ttu-id="de76e-175">TrackGraph</span><span class="sxs-lookup"><span data-stu-id="de76e-175">TrackGraph</span></span>

<span data-ttu-id="de76e-176">在内部，Add、Attach 和 Update 使用图形遍历，为每个实体就是否应将其标记为 Added（若要插入）、Modified（若要更新）、Unchanged（不执行任何操作）或 Deleted（若要删除）作出决定。</span><span class="sxs-lookup"><span data-stu-id="de76e-176">Internally, Add, Attach, and Update use graph-traversal with a determination made for each entity as to whether it should be marked as Added (to insert), Modified (to update), Unchanged (do nothing), or Deleted (to delete).</span></span> <span data-ttu-id="de76e-177">此机制是通过 TrackGraph API 公开的。</span><span class="sxs-lookup"><span data-stu-id="de76e-177">This mechanism is exposed via the TrackGraph API.</span></span> <span data-ttu-id="de76e-178">例如，假设当客户端发送回实体图形时，会在每个实体上设置某些标志，指示应如何处理实体。</span><span class="sxs-lookup"><span data-stu-id="de76e-178">For example, let's assume that when the client sends back a graph of entities it sets some flag on each entity indicating how it should be handled.</span></span> <span data-ttu-id="de76e-179">然后，TrackGraph 可用于处理此标志：</span><span class="sxs-lookup"><span data-stu-id="de76e-179">TrackGraph can then be used to process this flag:</span></span>

[!code-csharp[Main](../../../samples/core/Saving/Disconnected/Sample.cs#TrackGraph)]

<span data-ttu-id="de76e-180">为了简化示例，标志仅作为实体的一部分显示。</span><span class="sxs-lookup"><span data-stu-id="de76e-180">The flags are only shown as part of the entity for simplicity of the example.</span></span> <span data-ttu-id="de76e-181">通常，标志是 DTO 或包含在请求中的其他某个状态的一部分。</span><span class="sxs-lookup"><span data-stu-id="de76e-181">Typically the flags would be part of a DTO or some other state included in the request.</span></span>
