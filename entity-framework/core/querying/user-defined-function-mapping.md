---
title: 用户定义的函数映射 - EF Core
description: 将用户定义的函数映射到数据库函数
author: maumar
ms.date: 11/23/2020
uid: core/user-defined-function-mapping
ms.openlocfilehash: ba60abdc9c81b34b8f4ed8f501cf2f7e52ba9d7d
ms.sourcegitcommit: 4860d036ea0fb392c28799907bcc924c987d2d7b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/17/2020
ms.locfileid: "97657695"
---
# <a name="user-defined-function-mapping"></a>用户定义的函数映射

EF Core 允许在查询中使用用户定义的 SQL 函数。 为此，需要在模型配置过程中将函数映射到 CLR 方法。 将 LINQ 查询转换为 SQL 时，将调用用户定义的函数，而不是它已映射到的 CLR 函数。

## <a name="mapping-a-method-to-a-sql-function"></a>将方法映射到 SQL 函数

为了说明用户定义的函数映射的工作原理，让我们定义以下实体：

[!code-csharp[Main](../../../samples/core/Querying/UserDefinedFunctionMapping/Model.cs#Entities)]

以及以下模型配置：

[!code-csharp[Main](../../../samples/core/Querying/UserDefinedFunctionMapping/Model.cs#EntityConfiguration)]

博客可以有多篇帖文，每篇帖文可以有多条评论。

接下来，根据博客 `Id`，创建用户定义的函数 `CommentedPostCountForBlog`，该函数将返回针对给定博客至少具有一条评论的帖文计数：

```sql
CREATE FUNCTION dbo.CommentedPostCountForBlog(@id int)
RETURNS int
AS
BEGIN
    RETURN (SELECT COUNT(*)
        FROM [Posts] AS [p]
        WHERE ([p].[BlogId] = @id) AND ((
            SELECT COUNT(*)
            FROM [Comments] AS [c]
            WHERE [p].[PostId] = [c].[PostId]) > 0));
END
```

为了在 EF Core 中使用此函数，我们将定义以下 CLR 方法，并将其映射到用户定义的函数：

[!code-csharp[Main](../../../samples/core/Querying/UserDefinedFunctionMapping/Model.cs#BasicFunctionDefinition)]

CLR 方法的主体并不重要。 不会在客户端调用此方法，除非 EF Core 不能转换其参数。 如果可以转换参数，EF Core 只关心方法签名。

> [!NOTE]
> 在此示例中，方法是基于 `DbContext` 定义的，但也可以将其定义为其他类中的静态方法。

此函数定义现在可以与模型配置中用户定义的函数关联：

[!code-csharp[Main](../../../samples/core/Querying/UserDefinedFunctionMapping/Model.cs#BasicFunctionConfiguration)]

默认情况下，EF Core 尝试将 CLR 函数映射到具有相同名称的用户定义的函数。 如果名称不同，可以使用 `HasName` 为要映射到的用户定义的函数提供正确的名称。

接下来，执行下面的查询：

[!code-csharp[Main](../../../samples/core/Querying/UserDefinedFunctionMapping/Program.cs#BasicQuery)]

将生成此 SQL：

```sql
SELECT [b].[BlogId], [b].[Rating], [b].[Url]
FROM [Blogs] AS [b]
WHERE [dbo].[CommentedPostCountForBlog]([b].[BlogId]) > 1
```

## <a name="mapping-a-method-to-a-custom-sql"></a>将方法映射到自定义 SQL

EF Core 还允许将用户定义的函数转换为特定 SQL。 SQL 表达式是在配置用户定义的函数过程中使用 `HasTranslation` 方法提供的。

在下面的示例中，我们将创建一个函数，用于计算两个整数之间的百分比差异。

CLR 方法如下所示：

[!code-csharp[Main](../../../samples/core/Querying/UserDefinedFunctionMapping/Model.cs#HasTranslationFunctionDefinition)]

函数定义如下：

[!code-csharp[Main](../../../samples/core/Querying/UserDefinedFunctionMapping/Model.cs#HasTranslationFunctionConfiguration)]

定义函数后，可在查询中使用函数。 EF Core 会根据从 HasTranslation 构造的 SQL 表达式树将方法主体直接转换为 SQL，而不是调用数据库函数。 下面的 LINQ 查询可以：

[!code-csharp[Main](../../../samples/core/Querying/UserDefinedFunctionMapping/Program.cs#HasTranslationQuery)]

生成以下 SQL：

```sql
SELECT 100 * (ABS(CAST([p].[BlogId] AS float) - 3) / ((CAST([p].[BlogId] AS float) + 3) / 2))
FROM [Posts] AS [p]
```

## <a name="mapping-a-queryable-function-to-a-table-valued-function"></a>将可查询函数映射到表值函数

EF Core 还支持映射到表值函数，方法是使用返回实体类型的 `IQueryable` 的用户定义的 CLR 方法并允许 EF Core 映射带参数的 TVF。 此过程类似于将标量用户定义的函数映射到 SQL 函数：我们需要数据库中的 TVF、在 LINQ 查询中使用的 CLR 函数，以及两者之间的映射。

例如，我们将使用表值函数返回至少具有一个符合给定“赞”阈值的评论的所有帖文：

```sql
CREATE FUNCTION dbo.PostsWithPopularComments(@likeThreshold int)
RETURNS TABLE
AS
RETURN
(
    SELECT [p].[PostId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
    FROM [Posts] AS [p]
    WHERE (
        SELECT COUNT(*)
        FROM [Comments] AS [c]
        WHERE ([p].[PostId] = [c].[PostId]) AND ([c].[Likes] >= @likeThreshold)) > 0
)
```

CLR 方法签名如下所示：

[!code-csharp[Main](../../../samples/core/Querying/UserDefinedFunctionMapping/Model.cs#QueryableFunctionDefinition)]

> [!TIP]
> CLR 函数主体中的 `FromExpression` 调用允许使用函数而不是常规 DbSet。

下面是映射：

[!code-csharp[Main](../../../samples/core/Querying/UserDefinedFunctionMapping/Model.cs#QueryableFunctionConfigurationHasDbFunction)]

> [!CAUTION]
> 在[问题 23408](https://github.com/dotnet/efcore/issues/23408) 得到修复之前，映射到实体类型 `IQueryable` 时将替代 DbSet 的表的默认映射。 如有必要（例如，不是无键实体时），必须使用 `ToTable` 方法显式指定表映射。

> [!NOTE]
> 可查询函数必须映射到表值函数，并且不能使用 `HasTranslation`。

映射函数后，请考虑以下查询：

[!code-csharp[Main](../../../samples/core/Querying/UserDefinedFunctionMapping/Program.cs#TableValuedFunctionQuery)]

生成：

```sql
SELECT [p].[PostId], [p].[BlogId], [p].[Content], [p].[Rating], [p].[Title]
FROM [dbo].[PostsWithPopularComments](@likeThreshold) AS [p]
ORDER BY [p].[Rating]
```
