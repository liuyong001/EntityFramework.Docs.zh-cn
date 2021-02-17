---
title: 函数映射-Microsoft SQL Server 数据库提供程序-EF Core
description: Microsoft SQL Server 数据库提供程序的函数映射
author: bricelam
ms.date: 1/26/2021
uid: core/providers/sql-server/functions
ms.openlocfilehash: c39e81fca52854e7fa7d5b2e81b80511e8184b90
ms.sourcegitcommit: 704240349e18b6404e5a809f5b7c9d365b152e2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/16/2021
ms.locfileid: "100543219"
---
# <a name="function-mappings-of-the-microsoft-sql-server-provider"></a>Microsoft SQL Server 提供程序的函数映射

此页显示使用 SQL Server 提供程序时，哪些 .NET 成员转换为哪些 SQL 函数。

## <a name="binary-functions"></a>二进制函数

.NET                         | SQL                           | 在
---------------------------- | ----------------------------- | --------
字节.包含 (值)         | CHARINDEX (@value ， @bytes) > 0 | EF Core 5.0
字节.第一个 ()                 | SUBSTRING (@bytes ，1，1)        | EF Core 6.0
字节.长短                 | DATALENGTH (@bytes)             | EF Core 5.0
字节.SequenceEqual (second)   | @bytes = @second              | EF Core 5.0
字节 [i]                     | SUBSTRING (@bytes ， @i + 1，1)   | EF Core 6.0
EF.DataLength (arg)  | DATALENGTH (@arg)               | EF Core 5.0

## <a name="conversion-functions"></a>转换函数

.NET                      | SQL                                    | 在
------------------------- | -------------------------------------- | --------
字节.ToString ()           | 转换 (varchar (100) ， @bytes) 
byteValue ()       | CONVERT (varchar (3) ， @byteValue) 
charValue ()       | 转换 (varchar (1) ， @charValue) 
ToBoolean (值)   | 转换 (位， @value)                    | EF Core 5.0
ToByte (值)      | 转换 (tinyint， @value) 
ToDecimal (值)   | 将 (decimal (18，2) ， @value) 
ToDouble (值)    | 转换 (float、 @value) 
ToInt16 (值)     | 转换 (smallint， @value) 
ToInt32 (值)     | 转换 (int、 @value) 
ToInt64 (值)     | 转换 (bigint、 @value) 
将 ToString (值转换)    | 将 (nvarchar 转换 (max) ， @value) 
dateTime ()        | 转换 (varchar (100) ， @dateTime) 
dateTimeOffset ()  | 转换 (varchar (100) ， @dateTimeOffset) 
decimalValue ()    | 转换 (varchar (100) ， @decimalValue) 
doubleValue ()     | 转换 (varchar (100) ， @doubleValue) 
floatValue ()      | 转换 (varchar (100) ， @floatValue) 
guid.empty.ToString ()            | 转换 (varchar (36) ， @guid) 
intValue ()        | CONVERT (varchar (11) ， @intValue) 
longValue ()       | 转换 (varchar (20) ， @longValue) 
sbyteValue ()      | 转换 (varchar (4) ， @sbyteValue) 
shortValue ()      | CONVERT (varchar (6) ， @shortValue) 
timeSpan ()        | 转换 (varchar (100) ， @timeSpan) 
uintValue ()       | 转换 (varchar (10) ， @uintValue) 
ulongValue ()      | CONVERT (varchar (19) ， @ulongValue) 
ushortValue ()     | CONVERT (varchar (5) ， @ushortValue) 

## <a name="date-and-time-functions"></a>日期和时间函数

.NET                                                        | SQL                                                  | 在
----------------------------------------------------------- | ---------------------------------------------------- | --------
DateTime.Now                                                | GETDATE () 
日期时间. 今天                                              | 转换 (date、GETDATE () ) 
DateTime.UtcNow                                             | GETUTCDATE () 
AddDays (值)                                      | DATEADD (day， @value ， @dateTime) 
AddHours (值)                                     | DATEADD (小时， @value ， @dateTime) 
AddMilliseconds (值)                              | DATEADD (毫秒， @value ， @dateTime) 
AddMinutes (值)                                   | DATEADD (minute， @value ， @dateTime) 
AddMonths (月)                                   | DATEADD (month、 @months @dateTime) 
AddSeconds (值)                                   | DATEADD (second， @value ， @dateTime) 
AddYears (值)                                     | DATEADD (year， @value ， @dateTime) 
日期时间                                               | 转换 (日期 @dateTime) 
dateTime. Day                                                | DATEPART (day @dateTime) 
DayOfYear                                          | DATEPART (dayofyear， @dateTime) 
dateTime. 小时                                               | DATEPART (小时， @dateTime) 
dateTime. 毫秒                                        | DATEPART (毫秒， @dateTime) 
dateTime. 分钟                                             | DATEPART (minute @dateTime) 
日期时间月                                              | DATEPART (month @dateTime) 
dateTime。秒                                             | DATEPART (second @dateTime) 
dateTime. TimeOfDay                                          | 转换 (时间， @dateTime)                               | EF Core 2.2
日期时间年份                                               | DATEPART (year， @dateTime) 
DateTimeOffset.Now                                          | SYSDATETIMEOFFSET () 
DateTimeOffset. UtcNow                                       | SYSUTCDATETIME () 
AddDays (天)                                 | DATEADD (day， @days ， @dateTimeOffset) 
AddHours (小时)                               | DATEADD (小时， @hours ， @dateTimeOffset) 
AddMilliseconds (毫秒)                 | DATEADD (毫秒， @milliseconds ， @dateTimeOffset) 
AddMinutes (分钟)                           | DATEADD (minute， @minutes ， @dateTimeOffset) 
AddMonths (月)                             | DATEADD (month、 @months @dateTimeOffset) 
AddSeconds (秒)                           | DATEADD (second， @seconds ， @dateTimeOffset) 
AddYears (年)                               | DATEADD (year， @years ， @dateTimeOffset) 
dateTimeOffset. 日期                                         | 转换 (日期 @dateTimeOffset) 
dateTimeOffset. 日                                          | DATEPART (day @dateTimeOffset) 
dateTimeOffset. DayOfYear                                    | DATEPART (dayofyear， @dateTimeOffset) 
dateTimeOffset. 小时                                         | DATEPART (小时， @dateTimeOffset) 
dateTimeOffset. 毫秒                                  | DATEPART (毫秒， @dateTimeOffset) 
dateTimeOffset. 分钟                                       | DATEPART (minute @dateTimeOffset) 
dateTimeOffset 月                                        | DATEPART (month @dateTimeOffset) 
dateTimeOffset。第二                                       | DATEPART (second @dateTimeOffset) 
dateTimeOffset                                    | 转换 (时间， @dateTimeOffset)                         | EF Core 2.2
dateTimeOffset 年                                         | DATEPART (year， @dateTimeOffset) 
EF.DateDiffDay (开始、结束)                         | DATEDIFF (day， @start ， @end) 
EF.DateDiffHour (开始、结束)                        | DATEDIFF (小时， @start ， @end) 
EF.DateDiffMicrosecond (开始、结束)                 | DATEDIFF (微秒， @start @end) 
EF.DateDiffMillisecond (开始、结束)                 | DATEDIFF (毫秒， @start ， @end) 
EF.DateDiffMinute (开始、结束)                      | DATEDIFF (minute， @start ， @d2) 
EF.DateDiffMonth (开始、结束)                       | DATEDIFF (month、 @start @end) 
EF.DateDiffNanosecond (开始、结束)                  | DATEDIFF (毫微秒， @start ， @end) 
EF.DateDiffSecond (开始、结束)                      | DATEDIFF (second， @start ， @end) 
EF.DateDiffWeek (开始、结束)                        | DATEDIFF (周， @start ， @end)                          | EF Core 5.0
EF.DateDiffYear (开始、结束)                        | DATEDIFF (year， @start ， @end) 
EF.DateFromParts (year，month，day)                 | DATEFROMPARTS (@year 、 @month @day)                    | EF Core 5.0
EF.DateTime2FromParts (year，month，day，... )       | DATETIME2FROMPARTS (@year ，，， @month @day ... )          | EF Core 5.0
EF.DateTimeFromParts (year，month，day，... )        | DATETIMEFROMPARTS (@year ，，， @month @day ... )           | EF Core 5.0
EF.DateTimeOffsetFromParts (year，month，day，... )  | DATETIMEOFFSETFROMPARTS (@year ，，， @month @day ... )     | EF Core 5.0
EF.IsDate (表达式)                              | ISDATE (@expression)                                   | EF Core 3.0
EF.SmallDateTimeFromParts (year，month，day，... )   | SMALLDATETIMEFROMPARTS (@year ，，， @month @day ... )      | EF Core 5.0
EF.TimeFromParts (hour，minute，second，... )        | TIMEFROMPARTS (@hour ，，， @minute @second ... )           | EF Core 5.0
timeSpan 小时数                                              | DATEPART (小时， @timeSpan)                             | EF Core 5.0
timeSpan 毫秒                                       | DATEPART (毫秒， @timeSpan)                      | EF Core 5.0
timeSpan 分钟数                                            | DATEPART (minute @timeSpan)                           | EF Core 5.0
timeSpan。秒                                            | DATEPART (second @timeSpan)                           | EF Core 5.0

## <a name="numeric-functions"></a>数值函数

.NET                    | SQL                  | 在
----------------------- | -------------------- | --------
EF.函数。随机 ()    | RAND ()                | EF Core 6.0
Math (值)          | ABS (@value) 
Acos (d)             | ACOS (@d) 
Asin (d)             | ASIN (@d) 
Atan (d)             | ATAN (@d) 
Atan2 (y，x)         | ATN2 (@y ， @x) 
 (d) 的数学天花板         | 天花板 (@d) 
 (d) 的数学 Cos             | COS (@d) 
数学 .Exp (d)              | EXP (@d) 
Math (d)            | 楼层 (@d) 
Math (d)              | 日志 (@d) 
 (，newBase)     | 日志 (@a ， @newBase) 
Log10 (d)            | LOG10 (@d) 
Pow (x，y)           | POWER (@x ， @y) 
 (d) 的数学运算           |  (舍入 @d ，0) 
Math (d，小数)  | 舍入 (@d ， @decimals) 
数学符号 (值)         | 签名 (@value) 
数学 Sin ()              | SIN (@a) 
数学 Sqrt (d)             | SQRT (@d) 
 () 的数学茶色             | 茶色 (@a) 
Math (d)         |  (舍入 @d ，0，1) 

## <a name="string-functions"></a>字符串函数

.NET                                                                    | SQL                                                                    | 在
----------------------------------------------------------------------- | ---------------------------------------------------------------------- | --------
EF.函数 (操作数，排序规则)                                 | @operand COLLATE @collation                                            | EF Core 5.0
EF.函数。包含 (propertyReference，searchCondition)                | 包含 (@propertyReference 、 @searchCondition)                          | EF Core 2.2
EF.函数。包含 (propertyReference、searchCondition、languageTerm)  | 包含 (@propertyReference 、 @searchCondition 、语言 @languageTerm)  | EF Core 2.2
EF.函数 FreeText (propertyReference、freeText)                       | FREETEXT (@propertyReference ， @freeText) 
EF.函数 FreeText (propertyReference、freeText、languageTerm)         | FREETEXT (@propertyReference 、 @freeText 、LANGUAGE @languageTerm) 
EF.IsNumeric (表达式)                                       | ISNUMERIC (@expression)                                                  | EF Core 6.0
EF.函数。例如 (matchExpression，pattern)                              | @matchExpression 外观 @pattern
EF.函数。例如 (matchExpression、pattern、escapeCharacter)             | @matchExpression LIKE @pattern ESCAPE @escapeCharacter
string. 比较 (strA，strB)                                               | @strA等于 = @strB THEN 0 .。。端面
类似.Concat (str0，str1)                                                | @str0 + @str1
类似.IsNullOrEmpty (值)                                              | @value 为 NULL 或 @value 类似 N ""
类似.String.isnullorwhitespace (值)                                         | @value 为 NULL 或 LTRIM (RTRIM (@value) ) = N ' '
stringValue. CompareTo (strB)                                              | @stringValue等于 = @strB THEN 0 .。。端面
stringValue 包含 (值)                                              | @stringValue 如 N "%" + @value + N "%"
stringValue. EndsWith (值)                                              | @stringValue 如 N '% ' + @value
stringValue. FirstOrDefault ()                                             | SUBSTRING (@stringValue ，1，1)                                           | EF Core 5.0
stringValue. IndexOf (值)                                               | CHARINDEX (@value ， @stringValue) -1
stringValue. LastOrDefault ()                                              | 子字符串 (@stringValue ，LEN (@stringValue) ，1)                           | EF Core 5.0
stringValue                                                      | LEN (@stringValue) 
stringValue (@oldValue ， @newValue)                                | 替换 (@stringValue 、 @oldValue @newValue) 
stringValue. StartsWith (值)                                            | @stringValue LIKE @value + N "%"
stringValue (startIndex，长度)                                | SUBSTRING (@stringValue ， @startIndex + 1 @length) 
stringValue. ToLower ()                                                    | 降低 (@stringValue) 
stringValue. ToUpper ()                                                    |  (的 @stringValue) 
stringValue ()                                                       | LTRIM (RTRIM (@stringValue) ) 
stringValue. TrimEnd ()                                                    | RTRIM (@stringValue) 
stringValue. TrimStart ()                                                  | LTRIM (@stringValue) 

## <a name="miscellaneous-functions"></a>杂项函数

.NET                                     | SQL                                | 在
---------------------------------------- | ---------------------------------- | --------
集合.包含 (项)                 | @item 中 @collection               | EF Core 3.0
属于 enumvalue. Enum.hasflag\ (标志)                   | @enumValue & @flag = @flag
Guid.NewGuid()                           | NEWID()
可以为 null.Context.request.headers.getvalueordefault ()              | 合并 (@nullable ，0) 
可以为 null.Context.request.headers.getvalueordefault (defaultValue)  | 合并 (@nullable ， @defaultValue) 

> [!NOTE]
> 出于说明目的，已经简化了一些 SQL。 实际 SQL 更复杂，可以处理更大范围的值。

## <a name="see-also"></a>请参阅

* [空间函数映射](xref:core/providers/sql-server/spatial#spatial-function-mappings)
