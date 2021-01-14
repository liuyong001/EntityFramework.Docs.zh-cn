---
title: EF Core 和 Xamarin 入门 - EF Core
description: 将 Entity Framework Core 和 Xamarin 结合使用来创建移动应用的入门教程
author: codemillmatt
ms.date: 07/07/2020
ms.author: masoucou
uid: core/get-started/xamarin
ms.openlocfilehash: c19c70903926c2c8ff4b51cca7781859af7e3567
ms.sourcegitcommit: 032a1767d7a6e42052a005f660b80372c6521e7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2021
ms.locfileid: "98128779"
---
# <a name="getting-started-with-ef-core-and-xamarin"></a>EF Core 和 Xamarin 入门

在本教程中，你将创建一个 [Xamarin.Forms](/xamarin/get-started/what-is-xamarin-forms) 应用，该应用使用 Entity Framework Core 对 SQLite 数据库执行数据访问。

可使用 Windows 版 Visual Studio 或 Visual Studio for Mac 来执行本教程中的操作。

> [!TIP]
> 可在 [GitHub 示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Xamarin)中查看此文章的示例。

## <a name="prerequisites"></a>先决条件

安装以下工具之一：

* 对于此工作负载，建议采用 [Visual Studio 2019 版本 16.3 或更高版本](https://www.visualstudio.com/downloads/)：
  * 使用 .NET 的移动开发
* [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/)

该[文档提供每个平台的详细分步安装说明](/xamarin/get-started/installation)。

## <a name="download-and-run-the-sample-project"></a>下载并运行示例项目

若要运行和浏览此示例应用程序，请下载 GitHub 上的代码。

下载后，在 Visual Studio 或 Visual Studio for Mac 中打开解决方案文件 `EFGettingStarted.sln`，并在所选平台上运行应用程序。

应用首次启动时，将填充本地 SQLite 数据库（其中包含两个表示博客的条目）。

![包含所有博客的列表页的屏幕截图](_static/xamarin-tutorial-1.png)

单击工具栏中的“添加”按钮。

随即将显示一个新的页面，你可在该页面输入新博客的相关信息。

![新博客编辑页的屏幕截图](_static/xamarin-tutorial-2.png)

填写所有信息，然后单击工具栏中的“保存”。 新博客将保存到应用的 SQLite 数据库中，并显示在列表中。

你可单击列表中的一个博客条目，并查看该博客的任何帖子。

![博客文章列表页的屏幕截图](_static/xamarin-tutorial-3.png)

在工具栏中，单击“添加”。

此时将显示一个页面，你可在该页面中填写新博客文章的相关信息。

![“添加新的文章”页面的屏幕截图](_static/xamarin-tutorial-4.png)

填写所有信息，然后单击工具栏中的“保存”。

新文章将与上一步中单击的博客文章关联，它将保存到应用的 SQLite 数据库并在列表中显示。

返回到博客列表页。 然后单击工具栏中的“全部删除”。 所有博客及其对应的文章随后都将从应用的 SQLite 数据库中删除。

![博客被全部删除的应用的屏幕截图](_static/xamarin-tutorial-5.png)

## <a name="explore-the-code"></a>浏览代码

以下各部分将展示示例项目中的代码，该项目会搭配 EF Core 使用 Xamarin.Forms 从 SQLite 数据库读取、创建、更新和删除数据。

本文假设你熟悉 Xamarin. Forms 的[显示数据](/xamarin/xamarin-forms/app-fundamentals/data-binding/)和[在页面之间导航](/xamarin/xamarin-forms/app-fundamentals/navigation/)主题。

> [!IMPORTANT]
> Entity Framework Core 使用反射来调用 Xamarin.iOS 链接器在发布模式配置中可能会去除的函数。 可以通过以下两种方法之一来避免这种情况。
>
> * 第一种方法是将 `--linkskip System.Core` 添加到“iOS 生成”选项中的“其他 mtouch 参数”。
> * 或者，将 Xamarin.iOS“链接器行为”设置为“iOS 生成”选项中的 `Don't Link`。
> [本文详细介绍了 Xamarin.iOS 链接器](/xamarin/ios/deploy-test/linker)，包括如何在 Xamarin.iOS 上设置行为。
>

## <a name="entity-framework-core-nuget-packages"></a>Entity Framework Core NuGet 包

若要使用 EF Core 创建 Xamarin. Forms 应用，请将要以其为目标的 EF Core 数据库提供程序的包安装到 Xamarin.Forms 解决方案中的所有项目。 本教程使用 SQLite 提供程序。

Xamarin.Forms 解决方案的每个项目中都需要以下 NuGet 包。

* `Microsoft.EntityFrameworkCore.Sqlite`

## <a name="model-classes"></a>模型类

通过 EF Core 访问的 SQLite 数据库中的每个表都在某个类中进行建模。 在本例中使用了两个类 - `Blog` 和 `Post`，它们位于 `Models` 文件夹。

模型类仅由属性（在数据库中对列建模）组成。

* **Blog.cs**

  [!code-csharp[](../../../samples/core/Xamarin/EFGetStarted/Models/Blog.cs)]

* `Posts` 属性会定义 `Blog` 与 `Post` 之间的父子关系。

* **Post.cs**

  [!code-csharp[](../../../samples/core/Xamarin/EFGetStarted/Models/Post.cs)]

* `BlogId` 和 `Blog` 属性与 `Post` 的实例的父级 `Blog` 对象相关联。

## <a name="data-context"></a>数据上下文

`BloggingContext` 类位于 `Services` 文件夹中，并继承自 EF Core `DbContext` 类。 `DbContext` 用于将数据库查询和更改组合到一起。

  [!code-csharp[](../../../samples/core/Xamarin/EFGetStarted/Services/BloggingContext.cs)]

* `DbSet` 类中的两个属性都用于对表示博客和文章的基础表进行操作。
* 构造函数中需要 `SQLitePCL.Batteries_V2.Init()` 才能在 iOS 上启动 SQLite。
* `OnConfiguring` 函数可在物理设备上设置 SQLite 数据库的位置。

## <a name="create-read-update--delete"></a>创建、读取、更新和删除

下面是应用中的一些实例，其中 EF Core 用于访问 SQLite。

### <a name="read"></a>读取

* 返回所有记录。
  * `BlogsPage.xaml.cs` 的 `OnAppearing` 函数可返回所有 `Blog` 记录，并将它们存储在 `List` 变量中。

```csharp
using (var blogContext = new BloggingContext())
{
    var theBlogs = blogContext.Blogs.ToList();
}
```

* 返回特定记录。
  * `PostsPage.xaml.cs` 的 `OnAppearing` 函数可返回包含特定 `BlogId` 的 `Post` 记录。

```csharp
using (var blogContext = new BloggingContext())
{
    var postList = blogContext.Posts
        .Where(p => p.BlogId == BlogId)
        .ToList();
}
```

### <a name="create"></a>创建

* 插入新记录。
  * `AddBlogPage.xaml.cs` 的 `Save_Clicked` 函数可将新的 `Blog` 对象插入到 SQLite 数据库中。

```csharp
var blog = new Blog { Url = blogUrl.Text };

using (var blogContext = new BloggingContext())
{
    blogContext.Add(blog);

    await blogContext.SaveChangesAsync();
}
```

### <a name="update"></a>更新

* 更新现有记录。
  * `AddPostPage.xaml.cs` 的 `Save_Clicked` 函数使用新的 `Post` 更新现有的 `Blog` 对象。

```csharp
var newPost = new Post
{
    BlogId = BlogId,
    Content = postCell.Text,
    Title = titleCell.Text
};

using (var blogContext = new BloggingContext())
{
    var blog = await blogContext
        .Blogs
        .FirstAsync(b => b.BlogId == BlogId);

    blog.Posts.Add(newPost);

    await blogContext.SaveChangesAsync();
}
```

### <a name="delete"></a>删除

* 删除所有记录并级联删除其子记录。
  * `BlogsPage.xaml.cs` 的 `DeleteAll_Clicked` 函数可删除 SQLite 数据库中的所有 `Blog` 记录，并级联删除 `Blog` 的所有 `Post` 子记录。

```csharp
using (var blogContext = new BloggingContext())
{
    blogContext.RemoveRange(blogContext.Blogs);

    await blogContext.SaveChangesAsync();
}
```

## <a name="next-steps"></a>后续步骤

在本入门教程中，你已了解如何使用 Xamarin.Forms 应用程序通过 Entity Framework Core 访问 SQLite 数据库。

Xamarin 开发人员感兴趣的其他 Entity Framework Core 主题：

* [配置 `DbContext`](xref:core/dbcontext-configuration/index)
* 了解有关 [LINQ 查询表达式](/dotnet/csharp/programming-guide/concepts/linq/basic-linq-query-operations)的详细信息
* [配置模型](xref:core/modeling/index)指定[必需项](xref:core/modeling/entity-properties#required-and-optional-properties)和[最大长度](xref:core/modeling/entity-properties#maximum-length)等内容
