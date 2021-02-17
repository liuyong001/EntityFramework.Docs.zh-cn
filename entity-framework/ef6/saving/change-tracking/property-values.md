---
title: 使用属性值-EF6
description: 使用实体框架6中的属性值
author: ajcvickers
ms.date: 10/23/2016
uid: ef6/saving/change-tracking/property-values
ms.openlocfilehash: 4e9912df960481232c492f8a83d15595800edec3
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543557"
---
# <a name="working-with-property-values"></a>使用属性值
大多数情况下实体框架将负责跟踪实体实例的属性的状态、原始值和当前值。 但是，在某些情况下（例如，已断开连接的情况下），你希望查看或操作有关属性的信息 EF。 本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

实体框架跟踪所跟踪实体的每个属性的两个值。 当前值为，如名称所示，是实体中属性的当前值。 原始值是从数据库中查询实体或将实体附加到上下文时属性所具有的值。  

使用属性值的一般机制有两种：  

- 单个属性的值可以使用属性方法以强类型方式获取。  
- 实体的所有属性的值都可以读取到 DbPropertyValues 对象中。 然后，DbPropertyValues 充当类似字典的对象，以允许读取和设置属性值。 DbPropertyValues 对象中的值可以从其他 DbPropertyValues 对象中的值或其他某个对象的值进行设置，如实体的另一个副本或简单的数据传输对象 (DTO) 。  

以下部分显示了使用上述两种机制的示例。  

## <a name="getting-and-setting-the-current-or-original-value-of-an-individual-property"></a>获取和设置单个属性的当前值或原始值  

下面的示例演示如何读取属性的当前值，然后将其设置为新值：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(3);

    // Read the current value of the Name property
    string currentName1 = context.Entry(blog).Property(u => u.Name).CurrentValue;

    // Set the Name property to a new value
    context.Entry(blog).Property(u => u.Name).CurrentValue = "My Fancy Blog";

    // Read the current value of the Name property using a string for the property name
    object currentName2 = context.Entry(blog).Property("Name").CurrentValue;

    // Set the Name property to a new value using a string for the property name
    context.Entry(blog).Property("Name").CurrentValue = "My Boring Blog";
}
```  

使用 OriginalValue 属性而非 CurrentValue 属性来读取或设置原始值。  

请注意，当使用字符串指定属性名时，返回的值将被类型化为 "object"。 另一方面，如果使用 lambda 表达式，则返回值为强类型。  

如果新值不同于旧值，则将属性值设置为时，只会将属性标记为已修改。  

以这种方式设置属性值时，即使关闭了 AutoDetectChanges，也会自动检测更改。  

## <a name="getting-and-setting-the-current-value-of-an-unmapped-property"></a>获取和设置未映射的属性的当前值  

还可以读取未映射到数据库的属性的当前值。 未映射的属性的一个示例可能是博客上的 .Rsslink 属性。 此值可以基于 BlogId 进行计算，因此无需存储在数据库中。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);
    // Read the current value of an unmapped property
    var rssLink = context.Entry(blog).Property(p => p.RssLink).CurrentValue;

    // Use a string to specify the property name
    var rssLinkAgain = context.Entry(blog).Property("RssLink").CurrentValue;
}
```  

如果属性公开了 setter，还可以设置当前值。  

在对未映射的属性执行实体框架验证时，读取未映射的属性的值非常有用。 出于相同原因，可以读取当前值并将其设置为当前未由上下文跟踪的实体的属性。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    // Create an entity that is not being tracked
    var blog = new Blog { Name = "ADO.NET Blog" };

    // Read and set the current value of Name as before
    var currentName1 = context.Entry(blog).Property(u => u.Name).CurrentValue;
    context.Entry(blog).Property(u => u.Name).CurrentValue = "My Fancy Blog";
    var currentName2 = context.Entry(blog).Property("Name").CurrentValue;
    context.Entry(blog).Property("Name").CurrentValue = "My Boring Blog";
}
```  

请注意，原始值不可用于未映射的属性或上下文未跟踪的实体属性。  

## <a name="checking-whether-a-property-is-marked-as-modified"></a>检查属性是否被标记为已修改  

下面的示例演示如何检查单个属性是否被标记为已修改：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    var nameIsModified1 = context.Entry(blog).Property(u => u.Name).IsModified;

    // Use a string for the property name
    var nameIsModified2 = context.Entry(blog).Property("Name").IsModified;
}
```  

调用 SaveChanges 时，已修改属性的值将作为更新发送到数据库。  

##  <a name="marking-a-property-as-modified"></a>将属性标记为已修改  

下面的示例演示如何强制将单个属性标记为已修改：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    context.Entry(blog).Property(u => u.Name).IsModified = true;

    // Use a string for the property name
    context.Entry(blog).Property("Name").IsModified = true;
}
```  

将属性标记为已修改会强制在调用 SaveChanges 时将更新发送到该属性的数据库，即使该属性的当前值与原始值相同也是如此。  

目前不能在标记为 "已修改" 后将单个属性重置为不修改。 这是我们计划在未来版本中提供支持的内容。  

## <a name="reading-current-original-and-database-values-for-all-properties-of-an-entity"></a>读取实体的所有属性的当前值、原始值和数据库值  

下面的示例演示如何在数据库中读取实体的所有映射属性的当前值、原始值和值。  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    // Make a modification to Name in the tracked entity
    blog.Name = "My Cool Blog";

    // Make a modification to Name in the database
    context.Database.SqlCommand("update dbo.Blogs set Name = 'My Boring Blog' where Id = 1");

    // Print out current, original, and database values
    Console.WriteLine("Current values:");
    PrintValues(context.Entry(blog).CurrentValues);

    Console.WriteLine("\nOriginal values:");
    PrintValues(context.Entry(blog).OriginalValues);

    Console.WriteLine("\nDatabase values:");
    PrintValues(context.Entry(blog).GetDatabaseValues());
}

public static void PrintValues(DbPropertyValues values)
{
    foreach (var propertyName in values.PropertyNames)
    {
        Console.WriteLine("Property {0} has value {1}",
                          propertyName, values[propertyName]);
    }
}
```  

当前值是实体的属性当前包含的值。 原始值是查询实体时从数据库中读取的值。 数据库值是当前存储在数据库中的值。 如果数据库中的值可能在查询实体后发生更改（例如，当另一个用户对数据库进行了并发编辑时），获取数据库值将很有用。  

## <a name="setting-current-or-original-values-from-another-object"></a>设置其他对象的当前值或原始值  

可以通过从另一个对象复制值来更新已跟踪实体的当前值或原始值。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);
    var coolBlog = new Blog { Id = 1, Name = "My Cool Blog" };
    var boringBlog = new BlogDto { Id = 1, Name = "My Boring Blog" };

    // Change the current and original values by copying the values from other objects
    var entry = context.Entry(blog);
    entry.CurrentValues.SetValues(coolBlog);
    entry.OriginalValues.SetValues(boringBlog);

    // Print out current and original values
    Console.WriteLine("Current values:");
    PrintValues(entry.CurrentValues);

    Console.WriteLine("\nOriginal values:");
    PrintValues(entry.OriginalValues);
}

public class BlogDto
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```  

运行上面的代码将输出：  

```console
Current values:
Property Id has value 1
Property Name has value My Cool Blog

Original values:
Property Id has value 1
Property Name has value My Boring Blog
```  

当使用从服务调用或 n 层应用程序中的客户端获取的值更新实体时，有时会使用此方法。 请注意，所使用的对象不必与实体具有相同的类型，但前提是它具有与实体的名称相匹配的属性。 在上面的示例中，BlogDTO 的实例用于更新原始值。  

请注意，从其他对象复制时，仅将设置为不同值的属性标记为已修改。  

## <a name="setting-current-or-original-values-from-a-dictionary"></a>从字典设置当前值或原始值  

可以通过从字典或其他一些数据结构复制值来更新已跟踪实体的当前值或原始值。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    var newValues = new Dictionary<string, object>
    {
        { "Name", "The New ADO.NET Blog" },
        { "Url", "blogs.msdn.com/adonet" },
    };

    var currentValues = context.Entry(blog).CurrentValues;

    foreach (var propertyName in newValues.Keys)
    {
        currentValues[propertyName] = newValues[propertyName];
    }

    PrintValues(currentValues);
}
```  

使用 OriginalValues 属性而非 CurrentValues 属性来设置原始值。  

## <a name="setting-current-or-original-values-from-a-dictionary-using-property"></a>使用属性设置字典中的当前值或原始值  

如上所述，使用 CurrentValues 或 OriginalValues 的一种替代方法是使用属性方法来设置每个属性的值。 当你需要设置复杂属性的值时，这可能更好。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var user = context.Users.Find("johndoe1987");

    var newValues = new Dictionary<string, object>
    {
        { "Name", "John Doe" },
        { "Location.City", "Redmond" },
        { "Location.State.Name", "Washington" },
        { "Location.State.Code", "WA" },
    };

    var entry = context.Entry(user);

    foreach (var propertyName in newValues.Keys)
    {
        entry.Property(propertyName).CurrentValue = newValues[propertyName];
    }
}
```  

在上面的示例中，使用点分名称访问复杂属性。 有关访问复杂属性的其他方法，请参阅本主题后面有关复杂属性的两个部分。  

## <a name="creating-a-cloned-object-containing-current-original-or-database-values"></a>创建包含当前、原始或数据库值的克隆对象  

从 CurrentValues、OriginalValues 或 GetDatabaseValues 返回的 DbPropertyValues 对象可用于创建实体的克隆。 此克隆将包含 DbPropertyValues 对象中用于创建它的属性值。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    var clonedBlog = context.Entry(blog).GetDatabaseValues().ToObject();
}
```  

请注意，返回的对象不是实体，也不是由上下文跟踪。 返回的对象也不会将任何关系设置为其他对象。  

克隆的对象可用于解决与数据库并发更新相关的问题，尤其是在使用涉及到特定类型的对象的数据绑定的 UI 时。  

## <a name="getting-and-setting-the-current-or-original-values-of-complex-properties"></a>获取和设置复杂属性的当前值或原始值  

可以使用属性方法读取和设置整个复杂对象的值，就像它可用于基元属性一样。 此外，还可以向下钻取到复杂对象，并读取或设置该对象的属性，甚至是嵌套的对象。 下面是一些示例：  

``` csharp
using (var context = new BloggingContext())
{
    var user = context.Users.Find("johndoe1987");

    // Get the Location complex object
    var location = context.Entry(user)
                       .Property(u => u.Location)
                       .CurrentValue;

    // Get the nested State complex object using chained calls
    var state1 = context.Entry(user)
                     .ComplexProperty(u => u.Location)
                     .Property(l => l.State)
                     .CurrentValue;

    // Get the nested State complex object using a single lambda expression
    var state2 = context.Entry(user)
                     .Property(u => u.Location.State)
                     .CurrentValue;

    // Get the nested State complex object using a dotted string
    var state3 = context.Entry(user)
                     .Property("Location.State")
                     .CurrentValue;

    // Get the value of the Name property on the nested State complex object using chained calls
    var name1 = context.Entry(user)
                       .ComplexProperty(u => u.Location)
                       .ComplexProperty(l => l.State)
                       .Property(s => s.Name)
                       .CurrentValue;

    // Get the value of the Name property on the nested State complex object using a single lambda expression
    var name2 = context.Entry(user)
                       .Property(u => u.Location.State.Name)
                       .CurrentValue;

    // Get the value of the Name property on the nested State complex object using a dotted string
    var name3 = context.Entry(user)
                       .Property("Location.State.Name")
                       .CurrentValue;
}
```  

使用 OriginalValue 属性而非 CurrentValue 属性来获取或设置原始值。  

请注意，可以使用属性或 ComplexProperty 方法来访问复杂属性。 但是，如果想要使用其他属性或 ComplexProperty 调用深化到复杂对象，则必须使用 ComplexProperty 方法。  

## <a name="using-dbpropertyvalues-to-access-complex-properties"></a>使用 DbPropertyValues 访问复杂属性  

使用 CurrentValues、OriginalValues 或 GetDatabaseValues 获取实体的所有当前值、原始值或数据库值时，任何复杂属性的值将作为嵌套 DbPropertyValues 对象返回。 然后，可以使用这些嵌套对象获取复杂对象的值。 例如，以下方法将打印出所有属性的值，包括任何复杂属性的值和嵌套的复杂属性。  

``` csharp
public static void WritePropertyValues(string parentPropertyName, DbPropertyValues propertyValues)
{
    foreach (var propertyName in propertyValues.PropertyNames)
    {
        var nestedValues = propertyValues[propertyName] as DbPropertyValues;
        if (nestedValues != null)
        {
            WritePropertyValues(parentPropertyName + propertyName + ".", nestedValues);
        }
        else
        {
            Console.WriteLine("Property {0}{1} has value {2}",
                              parentPropertyName, propertyName,
                              propertyValues[propertyName]);
        }
    }
}
```  

若要打印出所有当前属性值，将调用方法，如下所示：  

``` csharp
using (var context = new BloggingContext())
{
    var user = context.Users.Find("johndoe1987");

    WritePropertyValues("", context.Entry(user).CurrentValues);
}
```  
