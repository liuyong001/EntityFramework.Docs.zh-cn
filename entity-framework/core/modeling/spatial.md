---
title: 空间数据-EF Core
description: 在 Entity Framework Core 模型中使用空间数据
author: bricelam
ms.date: 10/02/2020
uid: core/modeling/spatial
ms.openlocfilehash: a38e75e150b3d3404f2ed7619d05ba478c411117
ms.sourcegitcommit: 788a56c2248523967b846bcca0e98c2ed7ef0d6b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2020
ms.locfileid: "95003440"
---
# <a name="spatial-data"></a><span data-ttu-id="d5cc7-103">空间数据</span><span class="sxs-lookup"><span data-stu-id="d5cc7-103">Spatial Data</span></span>

> [!NOTE]
> <span data-ttu-id="d5cc7-104">EF Core 2.2 中引入了此功能。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-104">This feature was introduced in EF Core 2.2.</span></span>

<span data-ttu-id="d5cc7-105">空间数据表示对象的物理位置和形状。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-105">Spatial data represents the physical location and the shape of objects.</span></span> <span data-ttu-id="d5cc7-106">许多数据库提供对此类数据的支持，以便能够与其他数据一起进行索引和查询。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-106">Many databases provide support for this type of data so it can be indexed and queried alongside other data.</span></span> <span data-ttu-id="d5cc7-107">常见方案包括从位置在给定距离内查询对象，或选择其边框包含给定位置的对象。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-107">Common scenarios include querying for objects within a given distance from a location, or selecting the object whose border contains a given location.</span></span> <span data-ttu-id="d5cc7-108">EF Core 支持使用 NetTopologySuite 空间库映射到空间数据类型。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-108">EF Core supports mapping to spatial data types using the NetTopologySuite spatial library.</span></span>

## <a name="installing"></a><span data-ttu-id="d5cc7-109">安装</span><span class="sxs-lookup"><span data-stu-id="d5cc7-109">Installing</span></span>

<span data-ttu-id="d5cc7-110">若要在 EF Core 中使用空间数据，需要安装相应的支持 NuGet 包。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-110">In order to use spatial data with EF Core, you need to install the appropriate supporting NuGet package.</span></span> <span data-ttu-id="d5cc7-111">需要安装哪个包取决于所使用的提供程序。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-111">Which package you need to install depends on the provider you're using.</span></span>

<span data-ttu-id="d5cc7-112">EF Core 提供程序</span><span class="sxs-lookup"><span data-stu-id="d5cc7-112">EF Core Provider</span></span>                        | <span data-ttu-id="d5cc7-113">空间 NuGet 包</span><span class="sxs-lookup"><span data-stu-id="d5cc7-113">Spatial NuGet Package</span></span>
--------------------------------------- | ---------------------
<span data-ttu-id="d5cc7-114">Microsoft.EntityFrameworkCore.SqlServer</span><span class="sxs-lookup"><span data-stu-id="d5cc7-114">Microsoft.EntityFrameworkCore.SqlServer</span></span> | [<span data-ttu-id="d5cc7-115">Microsoft.entityframeworkcore. NetTopologySuite</span><span class="sxs-lookup"><span data-stu-id="d5cc7-115">Microsoft.EntityFrameworkCore.SqlServer.NetTopologySuite</span></span>](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer.NetTopologySuite)
<span data-ttu-id="d5cc7-116">Microsoft.EntityFrameworkCore.Sqlite</span><span class="sxs-lookup"><span data-stu-id="d5cc7-116">Microsoft.EntityFrameworkCore.Sqlite</span></span>    | [<span data-ttu-id="d5cc7-117">Microsoft.entityframeworkcore. NetTopologySuite</span><span class="sxs-lookup"><span data-stu-id="d5cc7-117">Microsoft.EntityFrameworkCore.Sqlite.NetTopologySuite</span></span>](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite.NetTopologySuite)
<span data-ttu-id="d5cc7-118">Microsoft.EntityFrameworkCore.InMemory</span><span class="sxs-lookup"><span data-stu-id="d5cc7-118">Microsoft.EntityFrameworkCore.InMemory</span></span>  | [<span data-ttu-id="d5cc7-119">NetTopologySuite</span><span class="sxs-lookup"><span data-stu-id="d5cc7-119">NetTopologySuite</span></span>](https://www.nuget.org/packages/NetTopologySuite)
<span data-ttu-id="d5cc7-120">Npgsql.EntityFrameworkCore.PostgreSQL</span><span class="sxs-lookup"><span data-stu-id="d5cc7-120">Npgsql.EntityFrameworkCore.PostgreSQL</span></span>   | [<span data-ttu-id="d5cc7-121">Npgsql. Microsoft.entityframeworkcore. PostgreSQL. NetTopologySuite</span><span class="sxs-lookup"><span data-stu-id="d5cc7-121">Npgsql.EntityFrameworkCore.PostgreSQL.NetTopologySuite</span></span>](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL.NetTopologySuite)
<span data-ttu-id="d5cc7-122">Pomelo.EntityFrameworkCore.MySql</span><span class="sxs-lookup"><span data-stu-id="d5cc7-122">Pomelo.EntityFrameworkCore.MySql</span></span>        | [<span data-ttu-id="d5cc7-123">Pomelo. Microsoft.entityframeworkcore. NetTopologySuite</span><span class="sxs-lookup"><span data-stu-id="d5cc7-123">Pomelo.EntityFrameworkCore.MySql.NetTopologySuite</span></span>](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MySql.NetTopologySuite)
<span data-ttu-id="d5cc7-124">Devart.Data.MySql.EFCore</span><span class="sxs-lookup"><span data-stu-id="d5cc7-124">Devart.Data.MySql.EFCore</span></span>                | [<span data-ttu-id="d5cc7-125">Devart. EFCore. NetTopologySuite</span><span class="sxs-lookup"><span data-stu-id="d5cc7-125">Devart.Data.MySql.EFCore.NetTopologySuite</span></span>](https://www.nuget.org/packages/Devart.Data.MySql.EFCore.NetTopologySuite)
<span data-ttu-id="d5cc7-126">Devart.Data.PostgreSql.EFCore</span><span class="sxs-lookup"><span data-stu-id="d5cc7-126">Devart.Data.PostgreSql.EFCore</span></span>           | [<span data-ttu-id="d5cc7-127">Devart. PostgreSql. EFCore. NetTopologySuite</span><span class="sxs-lookup"><span data-stu-id="d5cc7-127">Devart.Data.PostgreSql.EFCore.NetTopologySuite</span></span>](https://www.nuget.org/packages/Devart.Data.PostgreSql.EFCore.NetTopologySuite)
<span data-ttu-id="d5cc7-128">Devart.Data.SQLite.EFCore</span><span class="sxs-lookup"><span data-stu-id="d5cc7-128">Devart.Data.SQLite.EFCore</span></span>               | [<span data-ttu-id="d5cc7-129">Devart. EFCore. NetTopologySuite</span><span class="sxs-lookup"><span data-stu-id="d5cc7-129">Devart.Data.SQLite.EFCore.NetTopologySuite</span></span>](https://www.nuget.org/packages/Devart.Data.SQLite.EFCore.NetTopologySuite)
<span data-ttu-id="d5cc7-130">Teradata.EntityFrameworkCore</span><span class="sxs-lookup"><span data-stu-id="d5cc7-130">Teradata.EntityFrameworkCore</span></span>            | [<span data-ttu-id="d5cc7-131">Teradata. Microsoft.entityframeworkcore. NetTopologySuite</span><span class="sxs-lookup"><span data-stu-id="d5cc7-131">Teradata.EntityFrameworkCore.NetTopologySuite</span></span>](https://www.nuget.org/packages/Teradata.EntityFrameworkCore.NetTopologySuite)

## <a name="nettopologysuite"></a><span data-ttu-id="d5cc7-132">NetTopologySuite</span><span class="sxs-lookup"><span data-stu-id="d5cc7-132">NetTopologySuite</span></span>

<span data-ttu-id="d5cc7-133">[NetTopologySuite](https://nettopologysuite.github.io/NetTopologySuite/) (NTS) 是适用于 .net 的空间库。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-133">[NetTopologySuite](https://nettopologysuite.github.io/NetTopologySuite/) (NTS) is a spatial library for .NET.</span></span> <span data-ttu-id="d5cc7-134">EF Core 使用模型中的 NTS 类型启用映射到数据库中的空间数据类型。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-134">EF Core enables mapping to spatial data types in the database by using NTS types in your model.</span></span>

<span data-ttu-id="d5cc7-135">若要通过 NTS 启用到空间类型的映射，请在提供程序的 DbContext 选项生成器上调用 UseNetTopologySuite 方法。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-135">To enable mapping to spatial types via NTS, call the UseNetTopologySuite method on the provider's DbContext options builder.</span></span> <span data-ttu-id="d5cc7-136">例如，对于 SQL Server，你应将其称为。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-136">For example, with SQL Server you'd call it like this.</span></span>

[!code-csharp[](../../../samples/core/Spatial/SqlServer/Models/WideWorldImportersContext.cs?name=snippet_UseNetTopologySuite)]

<span data-ttu-id="d5cc7-137">有几种空间数据类型。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-137">There are several spatial data types.</span></span> <span data-ttu-id="d5cc7-138">使用哪种类型取决于您想要允许的形状的类型。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-138">Which type you use depends on the types of shapes you want to allow.</span></span> <span data-ttu-id="d5cc7-139">下面是可用于模型中的属性的 NTS 类型的层次结构。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-139">Here is the hierarchy of NTS types that you can use for properties in your model.</span></span> <span data-ttu-id="d5cc7-140">它们位于 `NetTopologySuite.Geometries` 命名空间内。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-140">They're located within the `NetTopologySuite.Geometries` namespace.</span></span>

* <span data-ttu-id="d5cc7-141">几何图形</span><span class="sxs-lookup"><span data-stu-id="d5cc7-141">Geometry</span></span>
  * <span data-ttu-id="d5cc7-142">点</span><span class="sxs-lookup"><span data-stu-id="d5cc7-142">Point</span></span>
  * <span data-ttu-id="d5cc7-143">LineString</span><span class="sxs-lookup"><span data-stu-id="d5cc7-143">LineString</span></span>
  * <span data-ttu-id="d5cc7-144">Polygon</span><span class="sxs-lookup"><span data-stu-id="d5cc7-144">Polygon</span></span>
  * <span data-ttu-id="d5cc7-145">GeometryCollection</span><span class="sxs-lookup"><span data-stu-id="d5cc7-145">GeometryCollection</span></span>
    * <span data-ttu-id="d5cc7-146">MultiPoint</span><span class="sxs-lookup"><span data-stu-id="d5cc7-146">MultiPoint</span></span>
    * <span data-ttu-id="d5cc7-147">MultiLineString</span><span class="sxs-lookup"><span data-stu-id="d5cc7-147">MultiLineString</span></span>
    * <span data-ttu-id="d5cc7-148">MultiPolygon</span><span class="sxs-lookup"><span data-stu-id="d5cc7-148">MultiPolygon</span></span>

> [!WARNING]
> <span data-ttu-id="d5cc7-149">NTS 不支持 CircularString、CompoundCurve 和 CurePolygon。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-149">CircularString, CompoundCurve, and CurePolygon aren't supported by NTS.</span></span>

<span data-ttu-id="d5cc7-150">使用基本几何图形类型允许属性指定任意类型的形状。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-150">Using the base Geometry type allows any type of shape to be specified by the property.</span></span>

## <a name="longitude-and-latitude"></a><span data-ttu-id="d5cc7-151">经度和纬度</span><span class="sxs-lookup"><span data-stu-id="d5cc7-151">Longitude and Latitude</span></span>

<span data-ttu-id="d5cc7-152">NTS 中的坐标采用 X 和 Y 值。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-152">Coordinates in NTS are in terms of X and Y values.</span></span> <span data-ttu-id="d5cc7-153">若要表示经度和纬度，请将 X 用于经度，将 Y 用于纬度。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-153">To represent longitude and latitude, use X for longitude and Y for latitude.</span></span> <span data-ttu-id="d5cc7-154">请注意，这 **backwards** 是从 `latitude, longitude` 通常会看到这些值的格式反向进行的。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-154">Note that this is **backwards** from the `latitude, longitude` format in which you typically see these values.</span></span>

## <a name="querying-data"></a><span data-ttu-id="d5cc7-155">查询数据</span><span class="sxs-lookup"><span data-stu-id="d5cc7-155">Querying Data</span></span>

<span data-ttu-id="d5cc7-156">以下实体类可用于映射到 [广角导入示例数据库](https://go.microsoft.com/fwlink/?LinkID=800630)中的表。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-156">The following entity classes could be used to map to tables in the [Wide World Importers sample database](https://go.microsoft.com/fwlink/?LinkID=800630).</span></span>

[!code-csharp[](../../../samples/core/Spatial/SqlServer/Models/City.cs?name=snippet_City)]

[!code-csharp[](../../../samples/core/Spatial/SqlServer/Models/Country.cs?name=snippet_Country)]

<span data-ttu-id="d5cc7-157">在 LINQ 中，可用作数据库函数的 NTS 方法和属性将转换为 SQL。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-157">In LINQ, the NTS methods and properties available as database functions will be translated to SQL.</span></span> <span data-ttu-id="d5cc7-158">例如，在以下查询中转换距离和包含方法。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-158">For example, the Distance and Contains methods are translated in the following queries.</span></span> <span data-ttu-id="d5cc7-159">有关支持的方法，请参阅提供程序的文档。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-159">See your provider's documentation for which methods are supported.</span></span>

[!code-csharp[](../../../samples/core/Spatial/SqlServer/Program.cs?name=snippet_Distance)]

[!code-csharp[](../../../samples/core/Spatial/SqlServer/Program.cs?name=snippet_Contains)]

## <a name="reverse-engineering"></a><span data-ttu-id="d5cc7-160">反向工程</span><span class="sxs-lookup"><span data-stu-id="d5cc7-160">Reverse engineering</span></span>

<span data-ttu-id="d5cc7-161">空间 NuGet 包还启用具有空间属性的 [反向工程](xref:core/managing-schemas/scaffolding) 模型，但需要在运行或 *_之前_* 安装包 \* `Scaffold-DbContext` `dotnet ef dbcontext scaffold` 。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-161">The spatial NuGet packages also enable [reverse engineering](xref:core/managing-schemas/scaffolding) models with spatial properties, but you need to install the package \**_before_* _ running `Scaffold-DbContext` or `dotnet ef dbcontext scaffold`.</span></span> <span data-ttu-id="d5cc7-162">否则，你将收到有关找不到列的类型映射的警告，将跳过这些列。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-162">If you don't, you'll receive warnings about not finding type mappings for the columns and the columns will be skipped.</span></span>

## <a name="srid-ignored-during-client-operations"></a><span data-ttu-id="d5cc7-163">在客户端操作过程中忽略 SRID</span><span class="sxs-lookup"><span data-stu-id="d5cc7-163">SRID Ignored during client operations</span></span>

<span data-ttu-id="d5cc7-164">NTS 在操作过程中忽略 SRID 值。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-164">NTS ignores SRID values during operations.</span></span> <span data-ttu-id="d5cc7-165">它假定为平面坐标系统。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-165">It assumes a planar coordinate system.</span></span> <span data-ttu-id="d5cc7-166">这意味着，如果在经度和纬度方面指定了坐标，则某些客户端计算的值（例如，距离、长度和区域）将为度数，而不是计量。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-166">This means that if you specify coordinates in terms of longitude and latitude, some client-evaluated values like distance, length, and area will be in degrees, not meters.</span></span> <span data-ttu-id="d5cc7-167">若要获得更有意义的值，首先需要使用库（如 [ProjNet4GeoAPI](https://github.com/NetTopologySuite/ProjNet4GeoAPI) ）在计算这些值之前投影到另一个坐标系统的坐标。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-167">For more meaningful values, you first need to project the coordinates to another coordinate system using a library like [ProjNet4GeoAPI](https://github.com/NetTopologySuite/ProjNet4GeoAPI) before calculating these values.</span></span>

<span data-ttu-id="d5cc7-168">如果通过 EF Core 通过 SQL 对操作进行服务器计算，则该结果的单元将由数据库确定。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-168">If an operation is server-evaluated by EF Core via SQL, the result's unit will be determined by the database.</span></span>

<span data-ttu-id="d5cc7-169">下面是一个示例，说明如何使用 ProjNet4GeoAPI 来计算两个城市之间的距离。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-169">Here is an example of using ProjNet4GeoAPI to calculate the distance between two cities.</span></span>

[!code-csharp[](../../../samples/core/Spatial/Projections/GeometryExtensions.cs?name=snippet_GeometryExtensions)]

[!code-csharp[](../../../samples/core/Spatial/Projections/Program.cs?name=snippet_ProjectTo)]

> [!NOTE]
> <span data-ttu-id="d5cc7-170">4326指的是 WGS 84，是 GPS 和其他地理系统中使用的标准。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-170">4326 refers to WGS 84, a standard used in GPS and other geographic systems.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="d5cc7-171">其他资源</span><span class="sxs-lookup"><span data-stu-id="d5cc7-171">Additional resources</span></span>

### <a name="database-specific-information"></a><span data-ttu-id="d5cc7-172">数据库特定的信息</span><span class="sxs-lookup"><span data-stu-id="d5cc7-172">Database-specific information</span></span>

<span data-ttu-id="d5cc7-173">有关使用空间数据的其他信息，请务必阅读提供程序的文档。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-173">Be sure to read your provider's documentation for additional information on working with spatial data.</span></span>

<span data-ttu-id="d5cc7-174">_ [SQL Server 提供程序中的空间数据](xref:core/providers/sql-server/spatial)</span><span class="sxs-lookup"><span data-stu-id="d5cc7-174">_ [Spatial Data in the SQL Server Provider](xref:core/providers/sql-server/spatial)</span></span>
* [<span data-ttu-id="d5cc7-175">SQLite 提供程序中的空间数据</span><span class="sxs-lookup"><span data-stu-id="d5cc7-175">Spatial Data in the SQLite Provider</span></span>](xref:core/providers/sqlite/spatial)
* [<span data-ttu-id="d5cc7-176">Npgsql 提供程序中的空间数据</span><span class="sxs-lookup"><span data-stu-id="d5cc7-176">Spatial Data in the Npgsql Provider</span></span>](https://www.npgsql.org/efcore/mapping/nts.html)

### <a name="other-resources"></a><span data-ttu-id="d5cc7-177">其他资源</span><span class="sxs-lookup"><span data-stu-id="d5cc7-177">Other resources</span></span>

* [<span data-ttu-id="d5cc7-178">NetTopologySuite 文档</span><span class="sxs-lookup"><span data-stu-id="d5cc7-178">NetTopologySuite Docs</span></span>](https://nettopologysuite.github.io/NetTopologySuite/)
* <span data-ttu-id="d5cc7-179">[EF Core 社区 Standup 会话](https://www.youtube.com/watch?v=IHslY5rrxD0&list=PLdo4fOcmZ0oX-DBuRG4u58ZTAJgBAeQ-t&index=15)，侧重于空间数据和 NetTopologySuite。</span><span class="sxs-lookup"><span data-stu-id="d5cc7-179">[EF Core Community Standup session](https://www.youtube.com/watch?v=IHslY5rrxD0&list=PLdo4fOcmZ0oX-DBuRG4u58ZTAJgBAeQ-t&index=15), focusing on spatial data and NetTopologySuite.</span></span>
