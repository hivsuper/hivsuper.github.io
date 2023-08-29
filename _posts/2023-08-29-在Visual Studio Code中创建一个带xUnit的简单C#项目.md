---
title: 在Visual Studio Code中创建一个带xUnit的简单C#项目  
date: 2023-08-29 23:00:00 +0800  
categories: [Chinese, C# Learning Journey]  
tags: [c#, xUnit]  
---
最近几周一直在写C#，接触到了一些有意思的东西，于是记录一下。  

## 参考资料
+ [Unit testing C# in .NET Core using dotnet test and xUnit](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test)
+ [Organizing and testing projects with the .NET CLI](https://learn.microsoft.com/en-us/dotnet/core/tutorials/testing-with-cli)

## 利用终端创建项目结构
```
PS D:\Test> dotnet new sln -o STUDY-CSHARP
已成功创建模板“解决方案文件”。

PS D:\Test> cd .\STUDY-CSHARP\
PS D:\Test\STUDY-CSHARP> dotnet new classlib -o Utility
已成功创建模板“类库”。

正在处理创建后操作...
正在还原 D:\Test\STUDY-CSHARP\Utility\Utility.csproj:
  正在确定要还原的项目…
  已还原 D:\Test\STUDY-CSHARP\Utility\Utility.csproj (用时 45 ms)。
已成功还原。


PS D:\Test\STUDY-CSHARP> cd .\Utility\
PS D:\Test\STUDY-CSHARP\Utility> ls


    目录: D:\Test\STUDY-CSHARP\Utility


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
da----         2023/8/29     23:55                obj
-a----         2023/8/29     23:55             52 Class1.cs
-a----         2023/8/29     23:55            215 Utility.csproj


PS D:\Test\STUDY-CSHARP\Utility> mv .\Class1.cs Helper.cs
PS D:\Test\STUDY-CSHARP\Utility> ls


    目录: D:\Test\STUDY-CSHARP\Utility


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
da----         2023/8/29     23:55                obj
-a----         2023/8/29     23:55             52 Helper.cs
-a----         2023/8/29     23:55            215 Utility.csproj


PS D:\Test\STUDY-CSHARP\Utility> cd ..
PS D:\Test\STUDY-CSHARP> dotnet sln add .\Utility\Utility.csproj
已将项目“Utility\Utility.csproj”添加到解决方案中。
PS D:\Test\STUDY-CSHARP> dotnet new xunit -o Utility.Tests
已成功创建模板“xUnit Test Project”。

正在处理创建后操作...
正在还原 D:\Test\STUDY-CSHARP\Utility.Tests\Utility.Tests.csproj:
  正在确定要还原的项目…
  已还原 D:\Test\STUDY-CSHARP\Utility.Tests\Utility.Tests.csproj (用时 278 ms)。
已成功还原。

PS D:\Test\STUDY-CSHARP> ls


    目录: D:\Test\STUDY-CSHARP


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2023/8/29     23:57                Utility
d-----         2023/8/29     23:59                Utility.Tests
-a----         2023/8/29     23:58            997 STUDY-CSHARP.sln


PS D:\Test\STUDY-CSHARP> dotnet sln add .\Utility.Tests\Utility.Tests.csproj
已将项目“Utility.Tests\Utility.Tests.csproj”添加到解决方案中。
PS D:\Test\STUDY-CSHARP> dotnet add .\Utility.Tests\Utility.Tests.csproj reference .\Utility\Utility.csproj
已将引用“..\Utility\Utility.csproj”添加到项目。
PS D:\Test\STUDY-CSHARP> mv .\Utility.Tests\UnitTest1.cs .\Utility.Tests\HelperTest.cs
```

## 在`Visual Studio Code`中打开`STUDY-CSHARP`
添加代码，例如
```
    public static bool BoolTest(bool? input)
    {
        return input ?? false;
    }
```
测试代码，例如
```
    [Fact]
    public void TestBoolTest()
    {
        Assert.False(Program.BoolTest(null));
        Assert.True(Program.BoolTest(true));
        Assert.False(Program.BoolTest(false));
    }
```
在终端运行
```
PS D:\Test\study-csharp> dotnet test
  正在确定要还原的项目…
  所有项目均是最新的，无法还原。
  Utility -> D:\Test\study-csharp\Utility\bin\Debug\net7.0\Utility.dll
  Utility.Tests -> D:\Test\study-csharp\Utility.Tests\bin\Debug\net7.0\Utility.Tests.dll
D:\Test\study-csharp\Utility.Tests\bin\Debug\net7.0\Utility.Tests.dll (.NETCoreApp,Version=v7.0)的测试运行
Microsoft (R) 测试执行命令行工具版本 17.4.0 (x64)
版权所有 (C) Microsoft Corporation。保留所有权利。

正在启动测试执行，请稍候...
总共 1 个测试文件与指定模式相匹配。

已通过! - 失败:     0，通过:     7，已跳过:     0，总计:     7，持续时间: 2 ms - Utility.Tests.dll (net7.0)
PS D:\Test\study-csharp> 
```