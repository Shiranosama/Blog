---
title: 「折腾日记」Win SDK APP, but VSCode
date: 2026-09-17T20:28:21+08:00
slug: b612c25
draft: false
description: Microsoft，你离开Visual Studio能开发Win SDK APP吗？我反正是尽力了
keywords:
weight: 0
categories:
  - 折腾日记
tags:
  - Windows
  - .NET
summary: Microsoft，你离开Visual Studio能开发Win SDK APP吗？我反正是尽力了
---

## 环境准备
- Visual Studio Code
    - C# 拓展
- Visual Studio Build Tools

> 关于C# dev kit拓展，实际上这拓展不好用，每次都要登录不说，交互逻辑还烂，不然你猜它为啥评分2.5/5

## 启动环境的正确姿势
- 启动Windwos Terminal配置文件——**Developer Command/PowerShell Prompt for VS 18**
    - 如果没有Windows Terminal，可以在Visual Studio Installer中点击Build Tools的启动按钮
- 在终端会话中执行`Code`命令以打开VS Code，这会正确继承Build Tools的环境变量

### 上下文菜单集成
- 文件夹右键菜单：
    ```reg
    Windows Registry Editor Version 5.00

    [HKEY_CLASSES_ROOT\Directory\shell\MSVCVSCode]
    @="Code with MSVC"
    "Icon"="C:\\Program Files\\Microsoft VS Code\\Code.exe"

    [HKEY_CLASSES_ROOT\Directory\shell\MSVCVSCode\Command]
    @="\"C:\\Program Files\\PowerShell\\7\\pwsh.exe\" -NoExit -WorkingDirectory \"%V\" -Command \"&{Import-Module \"\"\"C:\\Program Files (x86)\\Microsoft Visual Studio\\18\\BuildTools\\Common7\\Tools\\Microsoft.VisualStudio.DevShell.dll\"\"\"; Enter-VsDevShell xxxxxxxx -SkipAutomaticLocation -DevCmdArguments \"\"\"-arch=x64 -host_arch=x64\"\"\"; code -n \"%V\"}\""
    ```
- 文件夹背景右键菜单
    ```reg
    Windows Registry Editor Version 5.00

    [HKEY_CLASSES_ROOT\Directory\Background\shell\MSVCVSCode]
    @="Code with MSVC"
    "Icon"="C:\\Program Files\\Microsoft VS Code\\Code.exe"

    [HKEY_CLASSES_ROOT\Directory\Background\shell\MSVCVSCode\Command]
    @="\"C:\\Program Files\\PowerShell\\7\\pwsh.exe\" -NoExit -WorkingDirectory \"%V\" -Command \"&{Import-Module \"\"\"C:\\Program Files (x86)\\Microsoft Visual Studio\\18\\BuildTools\\Common7\\Tools\\Microsoft.VisualStudio.DevShell.dll\"\"\"; Enter-VsDevShell xxxxxxxx -SkipAutomaticLocation -DevCmdArguments \"\"\"-arch=x64 -host_arch=x64\"\"\"; code -n \"%V\"}\""
    ```
- 其中：
    - `MSVCVSCode`是键名，可随意更改
    - `Code with MSVC`是菜单项名
    - `Icon`是菜单图标，这里直接指向VsCode获取图标
    - Command键的值来源于Windows Terminal配置文件的**命令行**项，将其中的pwsh替换为绝对路径并添加了`-WorkingDirectory %V`用于指定工作目录，在-Command选项参数的结尾添加`code -n %V`以启动VsCode
        - 其中的**路径**、**xxxxxxxxx**并非固定，取决于Visual Studio Installer生成的具体值，请自行查看
        - %V在传递给shell时会被替换为右键点击的目录
        - code不使用绝对路径是因为pwsh会自己加载PATH

### Don't Do This
- 不要将Build Tools直接硬塞进环境变量，这将彻底污染PATH变量，不是一个明智的选择

## CLI工具
## DotNet
- dotnet命令负责.NET项目的编译、调试、发布，执行时，只需将当前项目的.sln文件传递给CLI即可：
    ```powershell
    # 一例：编译项目
    dotnet build ./Thisissln.sln /property:GenerateFullPaths=true /p:Configuration=Debug /p:Platform=x64 /consoleloggerparameters:NoSummary

    # 一例：发布项目为MSIX
    dotnet publish ./Thisissln.sln /property:GenerateFullPaths=true /p:Configuration=Release /p:Platform=x64 /consoleloggerparameters:NoSummary /p:GenerateAppxPackageOnBuild=true
    ```
    - `Configuration`对应项目配置
    - `Platform`对应平台架构
    - `GenerateAppxPackageOnBuild`告诉MSBuild生成AppxPackage

    > 为什么使用/p而不是-c/-r？

    > 因为/p是直接传递MSBuild参数，而后者是进行映射，有时候会出现与MSBuild当前参数不一致而导致报错，解决办法就是写两遍，一遍-r win-x64，一遍/p:Platform=x64。既然如此，那我为何不直接传MSBuild参数跳过中间商呢


### makeappx
- makeappx命令负责将dotnet生成的产物打包成.msix格式，将产物文件夹传递即可：
    ```powershell
    # 一例：打包项目
    makeappx pack /d .\win-x64\ /p Reorld.msix /o
    ```
- 注：
    - 确保你的产物文件夹包含msix打包所需文件

### signtool
- signtool命令负责签名程序包，需要提供一个可用的代码签名证书：
    ```powershell
    # 一例：签名MSIX
    # signtool默认在Certificate::CurrentUser\My下寻找证书
    # /s 参数可指定要在命令搜索证书时打开的存储
    signtool sign /sha1 '证书Sha1' /fd Sha256 /td Sha256 /tr http://timestamp.digicert.com .\Reorld.msix
    ```

注：证书的Subject必须与项目manifest里Identity的Publisher属性一致
### 找PowerShell给你画一张证书
- 下面的命令生成一个用于代码签名的证书，并储存在`PSParentPath: Microsoft.PowerShell.Security\Certificate::CurrentUser\My`下
    ```powershell
    New-SelfSignedCertificate -Type CodeSigningCert -Subject "CN=SuiBianTian" -CertStoreLocation Cert:\CurrentUser\My
    ```
- 导出证书
    ```powershell
    # 获取证书Hash
    Get-ChildItem -Path Cert:\CurrentUser\My\

    # 设置临时变量储存密码
    $mypwd = Read-Host -AsSecureString

    # 导出为pfx用于签名（一般不需要）
    Export-PfxCertificate -Cert "Cert:\CurrentUser\My\证书Hash" -FilePath "用来储存pfx的路径/Yourpfx.pfx" -Password $mypwd

    # 导出为cer用于信任/安装
    Export-Certificate -Cert "Cert:\CurrentUser\My\证书Hash" -FilePath "用来储存cer的路径/Yourcer.cer"
    ```
    - 如果你导出了pfx，在输入密码时强烈建议同样通过`Read-Host -AsSecureString`获取安全字符串临时变量后使用