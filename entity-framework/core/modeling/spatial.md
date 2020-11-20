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
# <a name="spatial-data"></a>空间数据

> [!NOTE]
> EF Core 2.2 中引入了此功能。

空间数据表示对象的物理位置和形状。 许多数据库提供对此类数据的支持，以便能够与其他数据一起进行索引和查询。 常见方案包括从位置在给定距离内查询对象，或选择其边框包含给定位置的对象。 EF Core 支持使用 NetTopologySuite 空间库映射到空间数据类型。

## <a name="installing"></a>安装

若要在 EF Core 中使用空间数据，需要安装相应的支持 NuGet 包。 需要安装哪个包取决于所使用的提供程序。

EF Core 提供程序                        | 空间 NuGet 包
--------------------------------------- | ---------------------
Microsoft.EntityFrameworkCore.SqlServer | [Microsoft.entityframeworkcore. NetTopologySuite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer.NetTopologySuite)
Microsoft.EntityFrameworkCore.Sqlite    | [Microsoft.entityframeworkcore. NetTopologySuite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite.NetTopologySuite)
Microsoft.EntityFrameworkCore.InMemory  | [NetTopologySuite](https://www.nuget.org/packages/NetTopologySuite)
Npgsql.EntityFrameworkCore.PostgreSQL   | [Npgsql. Microsoft.entityframeworkcore. PostgreSQL. NetTopologySuite](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL.NetTopologySuite)
Pomelo.EntityFrameworkCore.MySql        | [Pomelo. Microsoft.entityframeworkcore. NetTopologySuite](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MySql.NetTopologySuite)
Devart.Data.MySql.EFCore                | [Devart. EFCore. NetTopologySuite](https://www.nuget.org/packages/Devart.Data.MySql.EFCore.NetTopologySuite)
Devart.Data.PostgreSql.EFCore           | [Devart. PostgreSql. EFCore. NetTopologySuite](https://www.nuget.org/packages/Devart.Data.PostgreSql.EFCore.NetTopologySuite)
Devart.Data.SQLite.EFCore               | [Devart. EFCore. NetTopologySuite](https://www.nuget.org/packages/Devart.Data.SQLite.EFCore.NetTopologySuite)
Teradata.EntityFrameworkCore            | [Teradata. Microsoft.entityframeworkcore. NetTopologySuite](https://www.nuget.org/packages/Teradata.EntityFrameworkCore.NetTopologySuite)

## <a name="nettopologysuite"></a>NetTopologySuite

[NetTopologySuite](https://nettopologysuite.github.io/NetTopologySuite/) (NTS) 是适用于 .net 的空间库。 EF Core 使用模型中的 NTS 类型启用映射到数据库中的空间数据类型。

若要通过 NTS 启用到空间类型的映射，请在提供程序的 DbContext 选项生成器上调用 UseNetTopologySuite 方法。 例如，对于 SQL Server，你应将其称为。

[!code-csharp[](../../../samples/core/Spatial/SqlServer/Models/WideWorldImportersContext.cs?name=snippet_UseNetTopologySuite)]

有几种空间数据类型。 使用哪种类型取决于您想要允许的形状的类型。 下面是可用于模型中的属性的 NTS 类型的层次结构。 它们位于 `NetTopologySuite.Geometries` 命名空间内。

* 几何图形
  * 点
  * LineString
  * Polygon
  * GeometryCollection
    * MultiPoint
    * MultiLineString
    * MultiPolygon

> [!WARNING]
> NTS 不支持 CircularString、CompoundCurve 和 CurePolygon。

使用基本几何图形类型允许属性指定任意类型的形状。

## <a name="longitude-and-latitude"></a>经度和纬度

NTS 中的坐标采用 X 和 Y 值。 若要表示经度和纬度，请将 X 用于经度，将 Y 用于纬度。 请注意，这 **backwards** 是从 `latitude, longitude` 通常会看到这些值的格式反向进行的。

## <a name="querying-data"></a>查询数据

以下实体类可用于映射到 [广角导入示例数据库](https://go.microsoft.com/fwlink/?LinkID=800630)中的表。

[!code-csharp[](../../../samples/core/Spatial/SqlServer/Models/City.cs?name=snippet_City)]

[!code-csharp[](../../../samples/core/Spatial/SqlServer/Models/Country.cs?name=snippet_Country)]

在 LINQ 中，可用作数据库函数的 NTS 方法和属性将转换为 SQL。 例如，在以下查询中转换距离和包含方法。 有关支持的方法，请参阅提供程序的文档。

[!code-csharp[](../../../samples/core/Spatial/SqlServer/Program.cs?name=snippet_Distance)]

[!code-csharp[](../../../samples/core/Spatial/SqlServer/Program.cs?name=snippet_Contains)]

## <a name="reverse-engineering"></a>反向工程

空间 NuGet 包还启用具有空间属性的 [反向工程](xref:core/managing-schemas/scaffolding) 模型，但需要在运行或 *_之前_* 安装包 * `Scaffold-DbContext` `dotnet ef dbcontext scaffold` 。 否则，你将收到有关找不到列的类型映射的警告，将跳过这些列。

## <a name="srid-ignored-during-client-operations"></a>在客户端操作过程中忽略 SRID

NTS 在操作过程中忽略 SRID 值。 它假定为平面坐标系统。 这意味着，如果在经度和纬度方面指定了坐标，则某些客户端计算的值（例如，距离、长度和区域）将为度数，而不是计量。 若要获得更有意义的值，首先需要使用库（如 [ProjNet4GeoAPI](https://github.com/NetTopologySuite/ProjNet4GeoAPI) ）在计算这些值之前投影到另一个坐标系统的坐标。

如果通过 EF Core 通过 SQL 对操作进行服务器计算，则该结果的单元将由数据库确定。

下面是一个示例，说明如何使用 ProjNet4GeoAPI 来计算两个城市之间的距离。

[!code-csharp[](../../../samples/core/Spatial/Projections/GeometryExtensions.cs?name=snippet_GeometryExtensions)]

[!code-csharp[](../../../samples/core/Spatial/Projections/Program.cs?name=snippet_ProjectTo)]

> [!NOTE]
> 4326指的是 WGS 84，是 GPS 和其他地理系统中使用的标准。

## <a name="additional-resources"></a>其他资源

### <a name="database-specific-information"></a>数据库特定的信息

有关使用空间数据的其他信息，请务必阅读提供程序的文档。

_ [SQL Server 提供程序中的空间数据](xref:core/providers/sql-server/spatial)
* [SQLite 提供程序中的空间数据](xref:core/providers/sqlite/spatial)
* [Npgsql 提供程序中的空间数据](https://www.npgsql.org/efcore/mapping/nts.html)

### <a name="other-resources"></a>其他资源

* [NetTopologySuite 文档](https://nettopologysuite.github.io/NetTopologySuite/)
* [EF Core 社区 Standup 会话](https://www.youtube.com/watch?v=IHslY5rrxD0&list=PLdo4fOcmZ0oX-DBuRG4u58ZTAJgBAeQ-t&index=15)，侧重于空间数据和 NetTopologySuite。
