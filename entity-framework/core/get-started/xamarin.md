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
# <a name="getting-started-with-ef-core-and-xamarin"></a><span data-ttu-id="69a2d-103">EF Core 和 Xamarin 入门</span><span class="sxs-lookup"><span data-stu-id="69a2d-103">Getting Started with EF Core and Xamarin</span></span>

<span data-ttu-id="69a2d-104">在本教程中，你将创建一个 [Xamarin.Forms](/xamarin/get-started/what-is-xamarin-forms) 应用，该应用使用 Entity Framework Core 对 SQLite 数据库执行数据访问。</span><span class="sxs-lookup"><span data-stu-id="69a2d-104">In this tutorial, you create a [Xamarin.Forms](/xamarin/get-started/what-is-xamarin-forms) application that performs data access against a SQLite database using Entity Framework Core.</span></span>

<span data-ttu-id="69a2d-105">可使用 Windows 版 Visual Studio 或 Visual Studio for Mac 来执行本教程中的操作。</span><span class="sxs-lookup"><span data-stu-id="69a2d-105">You can follow the tutorial by using Visual Studio on Windows or Visual Studio for Mac.</span></span>

> [!TIP]
> <span data-ttu-id="69a2d-106">可在 [GitHub 示例](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Xamarin)中查看此文章的示例。</span><span class="sxs-lookup"><span data-stu-id="69a2d-106">You can view this article's [sample on GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/master/samples/core/Xamarin).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="69a2d-107">先决条件</span><span class="sxs-lookup"><span data-stu-id="69a2d-107">Prerequisites</span></span>

<span data-ttu-id="69a2d-108">安装以下工具之一：</span><span class="sxs-lookup"><span data-stu-id="69a2d-108">Install one of the below:</span></span>

* <span data-ttu-id="69a2d-109">对于此工作负载，建议采用 [Visual Studio 2019 版本 16.3 或更高版本](https://www.visualstudio.com/downloads/)：</span><span class="sxs-lookup"><span data-stu-id="69a2d-109">[Visual Studio 2019 version 16.3 or later](https://www.visualstudio.com/downloads/) with this workload:</span></span>
  * <span data-ttu-id="69a2d-110">使用 .NET 的移动开发</span><span class="sxs-lookup"><span data-stu-id="69a2d-110">**Mobile Development with .NET**</span></span>
* [<span data-ttu-id="69a2d-111">Visual Studio for Mac</span><span class="sxs-lookup"><span data-stu-id="69a2d-111">Visual Studio for Mac</span></span>](https://visualstudio.microsoft.com/vs/mac/)

<span data-ttu-id="69a2d-112">该[文档提供每个平台的详细分步安装说明](/xamarin/get-started/installation)。</span><span class="sxs-lookup"><span data-stu-id="69a2d-112">This [documentation provides detailed step-by-step installation instructions](/xamarin/get-started/installation) for each platform.</span></span>

## <a name="download-and-run-the-sample-project"></a><span data-ttu-id="69a2d-113">下载并运行示例项目</span><span class="sxs-lookup"><span data-stu-id="69a2d-113">Download and run the sample project</span></span>

<span data-ttu-id="69a2d-114">若要运行和浏览此示例应用程序，请下载 GitHub 上的代码。</span><span class="sxs-lookup"><span data-stu-id="69a2d-114">To run and explore this sample application, download the code on GitHub.</span></span>

<span data-ttu-id="69a2d-115">下载后，在 Visual Studio 或 Visual Studio for Mac 中打开解决方案文件 `EFGettingStarted.sln`，并在所选平台上运行应用程序。</span><span class="sxs-lookup"><span data-stu-id="69a2d-115">Once downloaded, open the solution file `EFGettingStarted.sln` in Visual Studio or Visual Studio for Mac and run the application on the platform of your choice.</span></span>

<span data-ttu-id="69a2d-116">应用首次启动时，将填充本地 SQLite 数据库（其中包含两个表示博客的条目）。</span><span class="sxs-lookup"><span data-stu-id="69a2d-116">When the app first starts, it will populate the local SQLite database with two entries representing blogs.</span></span>

![包含所有博客的列表页的屏幕截图](_static/xamarin-tutorial-1.png)

<span data-ttu-id="69a2d-118">单击工具栏中的“添加”按钮。</span><span class="sxs-lookup"><span data-stu-id="69a2d-118">Click the **Add** button in the toolbar.</span></span>

<span data-ttu-id="69a2d-119">随即将显示一个新的页面，你可在该页面输入新博客的相关信息。</span><span class="sxs-lookup"><span data-stu-id="69a2d-119">A new page will appear that allows you to enter information about a new blog.</span></span>

![新博客编辑页的屏幕截图](_static/xamarin-tutorial-2.png)

<span data-ttu-id="69a2d-121">填写所有信息，然后单击工具栏中的“保存”。</span><span class="sxs-lookup"><span data-stu-id="69a2d-121">Fill out all the info and click **Save** from the toolbar.</span></span> <span data-ttu-id="69a2d-122">新博客将保存到应用的 SQLite 数据库中，并显示在列表中。</span><span class="sxs-lookup"><span data-stu-id="69a2d-122">The new blog will save to the app's SQLite database and will show in the list.</span></span>

<span data-ttu-id="69a2d-123">你可单击列表中的一个博客条目，并查看该博客的任何帖子。</span><span class="sxs-lookup"><span data-stu-id="69a2d-123">You can click on one of the blog entries in the list and see any posts for that blog.</span></span>

![博客文章列表页的屏幕截图](_static/xamarin-tutorial-3.png)

<span data-ttu-id="69a2d-125">在工具栏中，单击“添加”。</span><span class="sxs-lookup"><span data-stu-id="69a2d-125">Click **Add** in the toolbar.</span></span>

<span data-ttu-id="69a2d-126">此时将显示一个页面，你可在该页面中填写新博客文章的相关信息。</span><span class="sxs-lookup"><span data-stu-id="69a2d-126">A page then appears that allows you to fill out information about a new blog post.</span></span>

![“添加新的文章”页面的屏幕截图](_static/xamarin-tutorial-4.png)

<span data-ttu-id="69a2d-128">填写所有信息，然后单击工具栏中的“保存”。</span><span class="sxs-lookup"><span data-stu-id="69a2d-128">Fill out all the information and click **Save** in the toolbar.</span></span>

<span data-ttu-id="69a2d-129">新文章将与上一步中单击的博客文章关联，它将保存到应用的 SQLite 数据库并在列表中显示。</span><span class="sxs-lookup"><span data-stu-id="69a2d-129">The new post will be associated to the blog post you clicked on in a previous step and will save to the app's SQLite database and show in the list.</span></span>

<span data-ttu-id="69a2d-130">返回到博客列表页。</span><span class="sxs-lookup"><span data-stu-id="69a2d-130">Go back to the blog list page.</span></span> <span data-ttu-id="69a2d-131">然后单击工具栏中的“全部删除”。</span><span class="sxs-lookup"><span data-stu-id="69a2d-131">And click **Delete All** in the toolbar.</span></span> <span data-ttu-id="69a2d-132">所有博客及其对应的文章随后都将从应用的 SQLite 数据库中删除。</span><span class="sxs-lookup"><span data-stu-id="69a2d-132">All blogs and their corresponding posts will then be deleted from the app's SQLite database.</span></span>

![博客被全部删除的应用的屏幕截图](_static/xamarin-tutorial-5.png)

## <a name="explore-the-code"></a><span data-ttu-id="69a2d-134">浏览代码</span><span class="sxs-lookup"><span data-stu-id="69a2d-134">Explore the code</span></span>

<span data-ttu-id="69a2d-135">以下各部分将展示示例项目中的代码，该项目会搭配 EF Core 使用 Xamarin.Forms 从 SQLite 数据库读取、创建、更新和删除数据。</span><span class="sxs-lookup"><span data-stu-id="69a2d-135">The following sections will walk you through the code in the sample project that reads, creates, updates, and deletes data from a SQLite database using EF Core with Xamarin.Forms.</span></span>

<span data-ttu-id="69a2d-136">本文假设你熟悉 Xamarin. Forms 的[显示数据](/xamarin/xamarin-forms/app-fundamentals/data-binding/)和[在页面之间导航](/xamarin/xamarin-forms/app-fundamentals/navigation/)主题。</span><span class="sxs-lookup"><span data-stu-id="69a2d-136">It is assumed that you are familiar with the Xamarin.Forms topics of [displaying data](/xamarin/xamarin-forms/app-fundamentals/data-binding/) and [navigating between pages](/xamarin/xamarin-forms/app-fundamentals/navigation/).</span></span>

> [!IMPORTANT]
> <span data-ttu-id="69a2d-137">Entity Framework Core 使用反射来调用 Xamarin.iOS 链接器在发布模式配置中可能会去除的函数。</span><span class="sxs-lookup"><span data-stu-id="69a2d-137">Entity Framework Core uses reflection to invoke functions which the Xamarin.iOS linker may strip out while in **Release** mode configurations.</span></span> <span data-ttu-id="69a2d-138">可以通过以下两种方法之一来避免这种情况。</span><span class="sxs-lookup"><span data-stu-id="69a2d-138">You can avoid that in one of two ways.</span></span>
>
> * <span data-ttu-id="69a2d-139">第一种方法是将 `--linkskip System.Core` 添加到“iOS 生成”选项中的“其他 mtouch 参数”。</span><span class="sxs-lookup"><span data-stu-id="69a2d-139">The first is to add `--linkskip System.Core` to the **Additional mtouch arguments** in the **iOS Build** options.</span></span>
> * <span data-ttu-id="69a2d-140">或者，将 Xamarin.iOS“链接器行为”设置为“iOS 生成”选项中的 `Don't Link`。</span><span class="sxs-lookup"><span data-stu-id="69a2d-140">Alternatively set the Xamarin.iOS **Linker behavior** to `Don't Link` in the **iOS Build** options.</span></span>
> <span data-ttu-id="69a2d-141">[本文详细介绍了 Xamarin.iOS 链接器](/xamarin/ios/deploy-test/linker)，包括如何在 Xamarin.iOS 上设置行为。</span><span class="sxs-lookup"><span data-stu-id="69a2d-141">[This article explains more about the Xamarin.iOS linker](/xamarin/ios/deploy-test/linker) including how to set the behavior on Xamarin.iOS.</span></span>
>

## <a name="entity-framework-core-nuget-packages"></a><span data-ttu-id="69a2d-142">Entity Framework Core NuGet 包</span><span class="sxs-lookup"><span data-stu-id="69a2d-142">Entity Framework Core NuGet packages</span></span>

<span data-ttu-id="69a2d-143">若要使用 EF Core 创建 Xamarin. Forms 应用，请将要以其为目标的 EF Core 数据库提供程序的包安装到 Xamarin.Forms 解决方案中的所有项目。</span><span class="sxs-lookup"><span data-stu-id="69a2d-143">To create Xamarin.Forms apps with EF Core, you install the package for the EF Core database provider(s) you want to target into all of the projects in the Xamarin.Forms solution.</span></span> <span data-ttu-id="69a2d-144">本教程使用 SQLite 提供程序。</span><span class="sxs-lookup"><span data-stu-id="69a2d-144">This tutorial uses the SQLite provider.</span></span>

<span data-ttu-id="69a2d-145">Xamarin.Forms 解决方案的每个项目中都需要以下 NuGet 包。</span><span class="sxs-lookup"><span data-stu-id="69a2d-145">The following NuGet package is needed in each of the projects in the Xamarin.Forms solution.</span></span>

* `Microsoft.EntityFrameworkCore.Sqlite`

## <a name="model-classes"></a><span data-ttu-id="69a2d-146">模型类</span><span class="sxs-lookup"><span data-stu-id="69a2d-146">Model classes</span></span>

<span data-ttu-id="69a2d-147">通过 EF Core 访问的 SQLite 数据库中的每个表都在某个类中进行建模。</span><span class="sxs-lookup"><span data-stu-id="69a2d-147">Each table in the SQLite database accessed through EF Core is modeled in a class.</span></span> <span data-ttu-id="69a2d-148">在本例中使用了两个类 - `Blog` 和 `Post`，它们位于 `Models` 文件夹。</span><span class="sxs-lookup"><span data-stu-id="69a2d-148">In this sample, two classes are used: `Blog` and `Post` which can be found in the `Models` folder.</span></span>

<span data-ttu-id="69a2d-149">模型类仅由属性（在数据库中对列建模）组成。</span><span class="sxs-lookup"><span data-stu-id="69a2d-149">The model classes are composed only of properties, which model columns in the database.</span></span>

* <span data-ttu-id="69a2d-150">**Blog.cs**</span><span class="sxs-lookup"><span data-stu-id="69a2d-150">**Blog.cs**</span></span>

  [!code-csharp[](../../../samples/core/Xamarin/EFGetStarted/Models/Blog.cs)]

* <span data-ttu-id="69a2d-151">`Posts` 属性会定义 `Blog` 与 `Post` 之间的父子关系。</span><span class="sxs-lookup"><span data-stu-id="69a2d-151">The `Posts` property defines a parent-child relationship between `Blog` and `Post`.</span></span>

* <span data-ttu-id="69a2d-152">**Post.cs**</span><span class="sxs-lookup"><span data-stu-id="69a2d-152">**Post.cs**</span></span>

  [!code-csharp[](../../../samples/core/Xamarin/EFGetStarted/Models/Post.cs)]

* <span data-ttu-id="69a2d-153">`BlogId` 和 `Blog` 属性与 `Post` 的实例的父级 `Blog` 对象相关联。</span><span class="sxs-lookup"><span data-stu-id="69a2d-153">The `BlogId` and `Blog` properties relate back to the parent `Blog` object for the instance of the `Post`.</span></span>

## <a name="data-context"></a><span data-ttu-id="69a2d-154">数据上下文</span><span class="sxs-lookup"><span data-stu-id="69a2d-154">Data context</span></span>

<span data-ttu-id="69a2d-155">`BloggingContext` 类位于 `Services` 文件夹中，并继承自 EF Core `DbContext` 类。</span><span class="sxs-lookup"><span data-stu-id="69a2d-155">The `BloggingContext` class is located in the `Services` folder and inherits from the EF Core `DbContext` class.</span></span> <span data-ttu-id="69a2d-156">`DbContext` 用于将数据库查询和更改组合到一起。</span><span class="sxs-lookup"><span data-stu-id="69a2d-156">A `DbContext` is used to group together database queries and changes.</span></span>

  [!code-csharp[](../../../samples/core/Xamarin/EFGetStarted/Services/BloggingContext.cs)]

* <span data-ttu-id="69a2d-157">`DbSet` 类中的两个属性都用于对表示博客和文章的基础表进行操作。</span><span class="sxs-lookup"><span data-stu-id="69a2d-157">Both properties in this class of type `DbSet` are used to operate on the underlying tables representing Blogs and Posts.</span></span>
* <span data-ttu-id="69a2d-158">构造函数中需要 `SQLitePCL.Batteries_V2.Init()` 才能在 iOS 上启动 SQLite。</span><span class="sxs-lookup"><span data-stu-id="69a2d-158">The `SQLitePCL.Batteries_V2.Init()` is needed in the constructor to initiate SQLite on iOS.</span></span>
* <span data-ttu-id="69a2d-159">`OnConfiguring` 函数可在物理设备上设置 SQLite 数据库的位置。</span><span class="sxs-lookup"><span data-stu-id="69a2d-159">The `OnConfiguring` function sets up the location of the SQLite database on the physical device.</span></span>

## <a name="create-read-update--delete"></a><span data-ttu-id="69a2d-160">创建、读取、更新和删除</span><span class="sxs-lookup"><span data-stu-id="69a2d-160">Create, read, update & delete</span></span>

<span data-ttu-id="69a2d-161">下面是应用中的一些实例，其中 EF Core 用于访问 SQLite。</span><span class="sxs-lookup"><span data-stu-id="69a2d-161">The following are some instances in the app where EF Core is used to access SQLite.</span></span>

### <a name="read"></a><span data-ttu-id="69a2d-162">读取</span><span class="sxs-lookup"><span data-stu-id="69a2d-162">Read</span></span>

* <span data-ttu-id="69a2d-163">返回所有记录。</span><span class="sxs-lookup"><span data-stu-id="69a2d-163">Return all records.</span></span>
  * <span data-ttu-id="69a2d-164">`BlogsPage.xaml.cs` 的 `OnAppearing` 函数可返回所有 `Blog` 记录，并将它们存储在 `List` 变量中。</span><span class="sxs-lookup"><span data-stu-id="69a2d-164">The `OnAppearing` function of `BlogsPage.xaml.cs` returns all `Blog` records and stores them into a `List` variable.</span></span>

```csharp
using (var blogContext = new BloggingContext())
{
    var theBlogs = blogContext.Blogs.ToList();
}
```

* <span data-ttu-id="69a2d-165">返回特定记录。</span><span class="sxs-lookup"><span data-stu-id="69a2d-165">Return specific records.</span></span>
  * <span data-ttu-id="69a2d-166">`PostsPage.xaml.cs` 的 `OnAppearing` 函数可返回包含特定 `BlogId` 的 `Post` 记录。</span><span class="sxs-lookup"><span data-stu-id="69a2d-166">The `OnAppearing` function of `PostsPage.xaml.cs` returns `Post` records that contain a specific `BlogId`.</span></span>

```csharp
using (var blogContext = new BloggingContext())
{
    var postList = blogContext.Posts
        .Where(p => p.BlogId == BlogId)
        .ToList();
}
```

### <a name="create"></a><span data-ttu-id="69a2d-167">创建</span><span class="sxs-lookup"><span data-stu-id="69a2d-167">Create</span></span>

* <span data-ttu-id="69a2d-168">插入新记录。</span><span class="sxs-lookup"><span data-stu-id="69a2d-168">Insert a new record.</span></span>
  * <span data-ttu-id="69a2d-169">`AddBlogPage.xaml.cs` 的 `Save_Clicked` 函数可将新的 `Blog` 对象插入到 SQLite 数据库中。</span><span class="sxs-lookup"><span data-stu-id="69a2d-169">The `Save_Clicked` function of `AddBlogPage.xaml.cs` inserts a new `Blog` object into the SQLite database.</span></span>

```csharp
var blog = new Blog { Url = blogUrl.Text };

using (var blogContext = new BloggingContext())
{
    blogContext.Add(blog);

    await blogContext.SaveChangesAsync();
}
```

### <a name="update"></a><span data-ttu-id="69a2d-170">更新</span><span class="sxs-lookup"><span data-stu-id="69a2d-170">Update</span></span>

* <span data-ttu-id="69a2d-171">更新现有记录。</span><span class="sxs-lookup"><span data-stu-id="69a2d-171">Update an existing record.</span></span>
  * <span data-ttu-id="69a2d-172">`AddPostPage.xaml.cs` 的 `Save_Clicked` 函数使用新的 `Post` 更新现有的 `Blog` 对象。</span><span class="sxs-lookup"><span data-stu-id="69a2d-172">The `Save_Clicked` function of `AddPostPage.xaml.cs` updates an existing `Blog` object with a new `Post`.</span></span>

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

### <a name="delete"></a><span data-ttu-id="69a2d-173">删除</span><span class="sxs-lookup"><span data-stu-id="69a2d-173">Delete</span></span>

* <span data-ttu-id="69a2d-174">删除所有记录并级联删除其子记录。</span><span class="sxs-lookup"><span data-stu-id="69a2d-174">Delete all records with cascade to child records.</span></span>
  * <span data-ttu-id="69a2d-175">`BlogsPage.xaml.cs` 的 `DeleteAll_Clicked` 函数可删除 SQLite 数据库中的所有 `Blog` 记录，并级联删除 `Blog` 的所有 `Post` 子记录。</span><span class="sxs-lookup"><span data-stu-id="69a2d-175">The `DeleteAll_Clicked` function of `BlogsPage.xaml.cs` deletes all the `Blog` records in the SQLite database and cascades the deletes to all of the `Blog` child `Post` records.</span></span>

```csharp
using (var blogContext = new BloggingContext())
{
    blogContext.RemoveRange(blogContext.Blogs);

    await blogContext.SaveChangesAsync();
}
```

## <a name="next-steps"></a><span data-ttu-id="69a2d-176">后续步骤</span><span class="sxs-lookup"><span data-stu-id="69a2d-176">Next steps</span></span>

<span data-ttu-id="69a2d-177">在本入门教程中，你已了解如何使用 Xamarin.Forms 应用程序通过 Entity Framework Core 访问 SQLite 数据库。</span><span class="sxs-lookup"><span data-stu-id="69a2d-177">In this getting started you have learned how to use a Xamarin.Forms application to access a SQLite database using Entity Framework Core.</span></span>

<span data-ttu-id="69a2d-178">Xamarin 开发人员感兴趣的其他 Entity Framework Core 主题：</span><span class="sxs-lookup"><span data-stu-id="69a2d-178">Other Entity Framework Core topics of interest to Xamarin developers:</span></span>

* [<span data-ttu-id="69a2d-179">配置 `DbContext`</span><span class="sxs-lookup"><span data-stu-id="69a2d-179">Configuring a `DbContext`</span></span>](xref:core/dbcontext-configuration/index)
* <span data-ttu-id="69a2d-180">了解有关 [LINQ 查询表达式](/dotnet/csharp/programming-guide/concepts/linq/basic-linq-query-operations)的详细信息</span><span class="sxs-lookup"><span data-stu-id="69a2d-180">Learn more about [LINQ query expressions](/dotnet/csharp/programming-guide/concepts/linq/basic-linq-query-operations)</span></span>
* <span data-ttu-id="69a2d-181">[配置模型](xref:core/modeling/index)指定[必需项](xref:core/modeling/entity-properties#required-and-optional-properties)和[最大长度](xref:core/modeling/entity-properties#maximum-length)等内容</span><span class="sxs-lookup"><span data-stu-id="69a2d-181">[Configure your model](xref:core/modeling/index) to specify things like [required](xref:core/modeling/entity-properties#required-and-optional-properties) and [maximum length](xref:core/modeling/entity-properties#maximum-length)</span></span>
