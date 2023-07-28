---
title: '跟进 .NET 8 Blazor 之 ReuseTabs 支持 Query 属性绑定'
slug: 'asp-net-core-updates-in-net-6-preview-2'
date: 2023-07-28
tags: [AntDesign,Blazor]
toc: true
summary: 'ASP.NET 团队在 .NET 8 继续全力投入 Blazor，为 Blazor 带来了非常多的新特性，AntDesign Blazor也会继续佛系跟进。本篇主要介绍第一个在AntDesign Blazor 上应用的新特性，也就是 ReuseTabs 自 2021 年发布两年后，仍然未支持的 Query String 属性绑定功能。'
---

ASP.NET 团队和社区在 .NET 8 继续全力投入 Blazor，为它带来了非常多的新特性，特别是在服务端渲染（SSR）方面，一定程度解决之前 WASM 加载慢，Server 性能不理想等局限性，也跟原来的 MVC，Razor Pages 框架在底层完成了统一。

AntDesign Blazor 作为 Blazor 最受欢迎的开源组件库之一，自然也会继续佛系跟进。本篇主要介绍第一个在 AntDesign Blazor 上应用的 .NET 8 新特性—— `CascadingModelBinder`，我利用它实现了 ReuseTabs 自 2021 年发布两年后，一直未支持的 Query String 属性绑定。

ReuseTabs 是 AntDesign Blazor 在 2021 年 7 月增加的组件，也是 Blazor 目前唯一真正实现路由复用的组件。它只需在 App.razor 增加 RouteData 级联值，就可以在任何 Blazor 项目中独立使用（其文档上的例子就是官方模板），不依赖菜单配置就能够会主动识别路由，渲染页面组件，并保持每个 Tab 页面的状态。不像其他组件库的实现，只能在他们指定的配套模板上才能使用...

它支持绑定模板属性值，但是当页面组件 Query string 值绑定在 .NET 6 加入，ReuseTabs 却不支持了。 2022 年 5 月便有用户提了 issue。

我知道这个特性，也是在前段时间 .NET 官方博客中发布的文章 [ASP.NET Core 在 .NET 8 Preview 6 中的更新](https://devblogs.microsoft.com/dotnet/asp-net-core-updates-in-dotnet-8-preview-6/?WT.mc_id=DT-MVP-5003987)，里面提到了一个特性，级联 query string 值到 Blazor 组件，意思是不再让Query string 值绑定局限于页面组件了。