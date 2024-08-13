---
title: 'Blazor 本地化之共享资源与局部刷新'
slug: 'blazor-localization-shared-resource'
date: 2024-08-13
tags: [.NET, Blazor]
toc: true
summary: '本文介绍如何为 Blazor 应用添加共享本地化资源，并实现局部刷新'
---

本文参考 [Shared resources - ASP.NET Core Blazor globalization and localization](https://learn.microsoft.com/en-us/aspnet/core/blazor/globalization-localization?view=aspnetcore-8.0#shared-resources)

## 注册本地化服务

在 `Program.cs` 中注册本地化服务：

```csharp
builder.Services.AddLocalization();
```

画重点：共享资源是通过一个空类（dummy class）来定位，不需要设置 `ResourcesPath` 的。

## 添加空类

在项目一个目录定义一个空类，命名随意，但需要注意的是，这个类的命名空间需要与路径匹配。

如在详谬根目录里的 Localization 目录下，创建名为 `SharedResource.cs` 的

`Localization/SharedResource.cs:`

```csharp
namespace BlazorSample.Localization;

public class SharedResource
{
}
```

## 添加资源文件

然后在同一个目录下（Localization/）创建资源文件，文件名前缀要与空类一样。

`Localization/SharedResource.resx`

默认是英文，我们继续添加其他语言，如 zh-CN：

`Localization/SharedResource.zh-CN.resx`

打开这些资源文件的属性，把 `生成操作(Build Action)` 设置为 `嵌入的资源（Embedded resource）`。

## 使用

直接修改 `CultureInfo.DefaultThreadCurrentUICulture` 即可实现切换语言。切换后，利用之前的文章中介绍的[局部刷新方法](./blazor-reload-partial)，实现 @Body 内容的刷新，这样就能让页面重新显示对应语言的字符串。但是要注意的是，这种实现方式在静态渲染模式下无效，静态渲染的实现方式将在下篇文章中介绍。


```csharp
@using System.Globalization
@inherits LayoutComponentBase

<div class="page">
    <div class="sidebar">
        <NavMenu />
    </div>

    <main>
        <div class="top-row px-4">
            @if (CultureInfo.DefaultThreadCurrentUICulture?.Name == "en-US")
            {
                <a @onclick="@(()=>ChangeLanuge("zh-CN"))">中文</a>
            }
            else
            {
                <a @onclick="@(()=>ChangeLanuge("en-US"))">English</a>
            }
            <a href="https://learn.microsoft.com/aspnet/core/" target="_blank">About</a>
        </div>

        <article class="content px-4">
            <ReloadContainer @ref="reload">
                @Body
            </ReloadContainer>
        </article>
    </main>
</div>


@code {
    ReloadContainer reload;

    void ChangeLanuge(string culture)
    {
        CultureInfo.DefaultThreadCurrentUICulture = new CultureInfo(culture);
        reload.Reload();
    }
}
```

## 总结

本文介绍了如何在 Blazor 应用中添加共享本地化资源，并利用局部刷新实现切换语言。核心思路是通过一个空类来定位资源文件，并利用 `CultureInfo.DefaultThreadCurrentUICulture` 来切换语言。


