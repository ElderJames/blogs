---
title: 'Ant Design Blazor 组件库的路由复用多标签页介绍'
slug: 'implement-reuse-tabs-using-antblazor'
date: 2021-07-07
tags: [.NET, Blazor]
toc: true
summary: 最近，在 Ant Design Blazor 组件库中实现多标签页组件的呼声日益高涨。于是，我利用周末时间，基于 `BlazorDemoMultiPagesTab` 中提供的思路，结合 Blazor 内置路由组件实现了基于 `Tabs` 组件的 `ReuseTabs` 组件。
---

最近，在 Ant Design Blazor 组件库中实现多标签页组件的呼声日益高涨。于是，我利用周末时间，结合 Blazor 内置路由组件实现了基于 `Tabs` 组件的 `ReuseTabs` 组件。

<!--more-->

# 前言

Blazor 是 .NET 最新的前端框架，可以基于 WebAssembly 或 SignalR （WebSocket）构建前端应用程序，基于 WebAssembly 托管模型的 Blazor 甚至可以离线运行。再加上可以共用 .NET 类库，能使代码量比以往的基于 JS 的前后端分离模型少 1/3。而且现在 .NET 开发者也可以使用自己熟悉的技术和经验来开发前端应用了，不同技术栈的开发人员之间的沟通成本也大大减少，从而再一次解放了生产力。

所以，Blazor 是 .NET 开发者的又一生产力技术！

通过使用 Blazor 社区生态开源的 UI 组件库，常用的组件都被封装了起来，用户再也不需要写 JS 和 CSS 了，使得 .NET 开发人员在社区里连连称赞。目前，已经有大量的基于 Blazor 构建的企业级应用程序被部署上线，逐渐被企业认可。

# 正文

## 什么是路由复用多标签页

![Ant Design Blazor 组件库中的多标签页](/photos/reuse-tabs/reuse-tabs-demo1.gif)

本文标题中的路由复用，是 Angular 的 `RouteReuseStrategy` 中的概念，在中文社区也常被称作“多标签页”，主要的功能是当切换页面时，保持页面的状态，并且可以通过任意切换页面，当前展示的页面状态能够恢复。通常是被用在比较复杂的后台管理系统，用户可以在筛选了表格后，进入记录的详情页，再回到列表页后，仍然能保持原来的搜索条件、翻页数等等；也或者是填写表单时，需要去别的页面查看正确的信息，在回到表单时，表达上已填过的内容不会丢失。

而由于天然的能复用 C# 代码的优势， Blazor 通常被用于构建后台管理系统，所以使用标签页就成为了普遍的需求。然鹅，Blazor 官方团队并没有给我们直接提供这样的组件，所以就需要社区的小伙伴来实现了。

但是纵观社区中的几个开源组件库，都只是实现了通用的 Tabs 标签页组件，只能当作切换面板来使用。要用来实现“多标签页”的功能，要么不支持，要么还得要直接或间接地依赖自己菜单组件和布局组件，再要依赖页面文件路径，拼接出页面组件的类型，最后用反射来创建页面组件……

虽然说它们确实实现了多标签页的功能，但是实现方式不甚优雅。耦合度非常高，只能在组件库自己的框架布局中使用，而不能单独拎出来使用。另外，反射的性能也不好，还要把页面按照约定放置，对用户种种制约。

当然，社区中还流传一个比较认可的方案，就是 `BlazorDemoMultiPagesTab` 项目。它提供了一个原型，通过结合 Blazor 内置的路由组件实现了路由多标签页。但问题是它只是一个 demo，只实现了原理，代码比较凌乱，作者也没有再做整理，也没有封装成组件，如果想在自己项目中使用起来，肯定会薅秃自己的头发的。

## Ant Design Blazor 中的 ReuseTabs 组件

最近，在 Ant Design Blazor 组件库中实现多标签页组件的呼声日益高涨。于是，我利用周末时间，基于 `BlazorDemoMultiPagesTab` 中提供的思路，结合 Blazor 内置路由组件实现了基于 `Tabs` 组件的 `ReuseTabs` 组件。

`ReuseTabs` 组件只包含两个子组件，`ReuseTabsRouteView` 和 `ReuseTabs`，其中 `ReuseTabsRouteView` 继承自内置的 `RouteView` 组件，负责接管页面组件的生命周期，使页面组件能够保持状态或被销毁；`ReuseTabs` 负责标签的展示和交互。除此之外，没有再依赖菜单、布局之类的其他组件，因此可以直接用于任何 Blazor 应用程序，也可以很好地与其他组件库一起使用。

### 主要的特点

- 只需修改两处代码即可应用
- 支持静态或动态地设置标签名
- 与路由同步，支持 `<a>` 标签、`NavigationManger` 等各种方式的跳转

下面介绍一下基础的使用方法，以 dotnet new 模板项目为例。

### 使用方法

1. 首先，按照 Ant Design Blazor 文档中介绍的方式安装 AntDesign 包和服务注册。

2. 然后，修改项目中的 `App.razor` 文件，把 `RouteView` 替换成 `ReuseTabsRouteView`。

   ```diff
   <Router AppAssembly="@typeof(Program).Assembly">
       <Found Context="routeData">
   -       <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" / >
   +       <ReuseTabsRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
       </Found>
       ...
   </Router>
   ```

3. 修改项目中的 `MainLayout.razor` 文件，

   ```diff
   @inherits LayoutComponentBase

   <div class="page">
       <div class="sidebar">
           <NavMenu />
       </div>

       <div class="main">
   -       <div class="top-row px-4">
   -           <a href="http://blazor.net" target="_blank" class="ml-md-auto">About</a>
   -       </div>
   -       <div class="content px-4">
   -           @Body
   -       </div>
   +       <ReuseTabs Class="top-row px-4" TabPaneClass="content px-4" / >
       </div>
   </div>
   ```

4. 两种方式自定义标签名

   - 如果是纯文本，可以在页面组件使用 `ReuseTabsPageTitle` 特性。

   ```diff
   @page "/counter"
   + @attribute [ReuseTabsPageTitle("Counter")]
   ```

   - 如果需要使用模板来获取动态的标签名，比如添加另一个组件，或者从页面内容中获取标题，需要实现 `IReuseTabsPage` 接口：

   ```diff
   @page "/"
   + @implements IReuseTabsPage

   <h1>Hello, world!</h1>

   @code{
   +   public RenderFragment GetPageTitle() =>
   +       @<div>
   +           <Icon Type="home"/> Home
   +       </div>
   +   ;
   }
   ```

_注意：当前 `ReuseTabs` 组件在 0.9.0 版本的每日构建包中，在正式发布之前，需要参考[文档中介绍的方式](https://antblazor.com/zh-CN/docs/nightly-build)来安装。_

### 后续功能

目前该组件只实现了基本的功能，还有一些功能在后续的计划当中：

- 右键菜单
- 代码方式关闭
- 缓存策略
- 路由守卫与权限控制

# 后记

Blazor 社区中对多标签页的呼声有一年多了，这几天终于实现了，国内外的社区都一片欢腾，也是颇有成就感的。

对于实现的细节，感兴趣的同学可以到 Ant Design Blazor 的源码中查看，也只是几个文件。当然，如果感兴趣的同学比较多，我也可以写一篇详细的文章来介绍。

其实我是比较希望社区中能有更多的爱好者站出来，分享心得、贡献开源项目，这样才能让社区健康发展起来。
Ant Design Blazor 发展至今已有一年有多，但是说实话相对于其他组件库项目的作者，我自己的投入的时间和贡献并没有很多。其中除了贡献代码，一大部分精力都花在了社区的运营。
为项目作宣传，把路人变成用户，再把用户变成贡献者，让更多人各自花少量精力，达到众人拾柴的效果，才是开源项目得以长期活跃的长久之计。

最后还是非常感激支持我们的用户和贡献者！他们的每个 issue、案例和 PR 都是对我们的肯定，也是让我们坚持的动力！

# 参考链接

- https://github.com/BlazorPlus/BlazorDemoMultiPagesTab
- https://github.com/ant-design-blazor/demo-reuse-tabs
- https://antblazor.com/docs/nightly-build
