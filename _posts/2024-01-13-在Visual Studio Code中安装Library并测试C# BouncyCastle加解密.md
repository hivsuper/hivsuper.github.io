---
title: 在Visual Studio Code中安装Library并测试C# BouncyCastle加解密  
date: 2024-01-13 18:47:00 +0800  
categories: [Technology]  
tags: [c#, xUnit, NuGet, vscode]  
---
在[前文](/posts/在Visual-Studio-Code中创建一个带xUnit的简单C-项目/)基础上，增加一些特性：
- 支持多个`Main`方法
- 安装`NuGet`插件
- 测试`BouncyCastle`加密及解密

## 参考资料
+ [高级 C# 编译器选项](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/compiler-options/advanced#mainentrypoint-or-startupobject)
+ [常用的 MSBuild 项目属性](https://learn.microsoft.com/zh-cn/visualstudio/msbuild/common-msbuild-project-properties?view=vs-2022)
+ [mattosaurus/PgpCore](https://github.com/mattosaurus/PgpCore/issues/194)

## 支持多个`Main`方法
在`Utility.csproj`中添加`<OutputType>Exe</OutputType>`后项目可直接运行
```
PS D:\Test\STUDY-CSHARP> dotnet run --project .\Utility\Utility.csproj
```
若项目中有多个`Main`方法，会遇到报“<u>error CS0017: 程序定义了多个入口点。使用 /main (指定包含入口点的类型)进行编译。</u>”错误，那么需要再添加`StartupObject`指定入口，例如
```
<StartupObject>Utility.Example.HelperExample</StartupObject>
```

## 安装`NuGet`插件
运行`Ctrl + Shift + X`命令调出插件列表，安装`NuGet Package Manager GUI`  
![Visual Studio Code Extensions](/assets/img/202401/2024-01-13-01.png){: width="232" height="308" .normal }  
插件使用方法  
![NuGet Package Manager GUI](/assets/img/202401/2024-01-13-02.png){: width="800" .normal }  

## 测试`BouncyCastle`加密及解密
运行`Ctrl+Shift+P`，打开`NuGet Package Manager GUI`  
![NuGet Package Manager GUI](/assets/img/202401/2024-01-13-03.png){: width="449" .normal }  
参考下图安装`BouncyCastle`依赖包  
![NuGet Package Manager GUI](/assets/img/202401/2024-01-13-04.png){: width="800" .normal }  
添加[OpenPGPHelper.cs](https://github.com/hivsuper/learning-journey/blob/master/STUDY-CSHARP/Utility/OpenPGPHelper.cs)和[OpenPGPHelperExample.cs](https://github.com/hivsuper/learning-journey/blob/master/STUDY-CSHARP/Utility/Example/OpenPGPHelperExample.cs)，从[https://pgpkeygen.com/](https://pgpkeygen.com/)生成需要的密钥并下载[private_key.asc](https://github.com/hivsuper/learning-journey/blob/master/STUDY-CSHARP/Resources/private_key.asc)和[public_key.asc](https://github.com/hivsuper/learning-journey/blob/master/STUDY-CSHARP/Resources/public_key.asc)，修改`Utility.csproj`
```
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    ...
    <!-- Make the project runnable -->
    <OutputType>Exe</OutputType>
    <!-- Specify a startup object by changing the class here when there are multiple Main methods -->
    <StartupObject>Utility.Example.OpenPGPHelperExample</StartupObject>
  </PropertyGroup>
  <ItemGroup>
    ...
    <!-- Include embedded resources -->
    <EmbeddedResource Include="..\Resources\*.asc" />
  </ItemGroup>

</Project>
```
运行测试
```
PS D:\Test\STUDY-CSHARP> dotnet run --project .\Utility\Utility.csproj
...
LS0tLS1CRUdJTiBQR1AgTUVTU0FHRS0tLS0tDQpWZXJzaW9uOiBCb3VuY3lDYXN0bGUuTkVUIENyeXB0b2dyYXBoeSAoT3BlblBHUC1vbmx5LCBuZXQ2LjApIHYyLjAuMC4xDQoNCmhJd0R0eXFucHJPK3JWNEJBLzkzcndiK2pFTnBDdlhHWlpMZ2ptd2xWam9hdEJLUXZLck1iTStwOVFGUEkrcS8NCndWWjA0S1BSU2Z4NUgyWlhHWW52VjJPMkIycDE3b0lDU284R2R6OGpCN2FTRFJkVHovdnk3T21iL0dTeVJxOHcNCjZlV3dUamxXbVZZMExXc3EwcVBMT2gwaGU3OTZuZ25SU29tMVdYVjlFNXJiMFN6OXJ4aFFVV1BxL2lBQ3lkTEENClJBSGpkNGtNMXVFMFE1WnZhYUxpYWJ6U3YvTVZMcU9kazZlWlV6UTAyd3gySVo2cDRodnZqTElRNExzMEU2VlQNClNNMlJhOTBBSGd2M1dDcXVZNzVyZ3g1Q1hkVU12dmE0ZHVCZ2t6elpOZmtlQkUzTm5SdVVNREdnSFU3NEtqcDENCm5JbDZSZ2t0cDhiV1doSGUyS25OSDlKYWQxSS9aQ0M4NE9BSzgyVDlUS0tQL0JzMlBuV1BXTEFLaFdLQ2FRUnUNCnVJNUJPZ1BHNm1SYVZUU05JQ3c1MWRXMW1qRzRtTURlNkJuZVREMTQ4S21BU0NWODdPZGlkellnSHNzWFFYdnANClhhSm1KMmJ6SHkyMjZYZkVIQ1ByeHAxSjNCRmZoRlRJR3pJdkYvazNSbzZJVS9ETVVHaS9CdmZLOUhLZFBDam8NCnJoSGxqcEMyMHcrRDVnejR3UzdVTEtKMTZvZHINCj1ac2N4DQotLS0tLUVORCBQR1AgTUVTU0FHRS0tLS0tDQo=
test
PS D:\Test\STUDY-CSHARP>
```