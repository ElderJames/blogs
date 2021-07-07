---
title: 'Blazor 面试 18 问'
slug: 'blazor-interview-questions'
date: 2021-03-16
tags: [.NET, Blazor]
toc: true
description: 以问题的形式介绍 Blazor 的基础概念。
---

> 本文翻译自[Blazor Interview Questions And Answers
> ](https://www.c-sharpcorner.com/article/introduction-to-blazor-with-net-core/)

# 问题一、 什么是 Blazor

Blazor 是一个免费的、开源的、跨平台的 Web 框架，它允许开发人员使用 C# 和 .NET 构建现代的、可扩展的、跨平台的 Web 应用程序。Blazor 是由微软和开源社区开发的，其最初设计目的是为了让 C# 和 .NET 开发人员使用 C# 语言构建 Web 客户端应用程序。Blazor 是现代的、快速的、快速发展的。

大多数 web 客户端应用程序都是使用 JavaScript 编写的，其中代码主要在浏览器中运行。Blazor 框架允许开发人员使用 C# 代替 JavaScript 创建丰富的交互式用户界面。

Blazor 同时支持客户端和服务器端编码。服务器端和客户端的应用逻辑都是用.NET 编写的。

尽管代码是用.NET 和 C#编写的，但 Blazor 将用户界面渲染为 HTML 和 CSS，以获得广泛的浏览器支持，包括移动浏览器。

下面是一个简单的 Blazor 代码示例，展示了 HTML 和 C# 如何在同一个文件中，以及如何从 HTML 代码中调用一个函数。

![Blazor Interview Questions And Answers](/photos/blazor-interview-questions/photo-1.jpg)

Blazor 还可以与现代托管平台（如 Docker）集成。

想要了解更多关于 Blazor 的信息，请从这里开始:使用。net Core 介绍 Blazor

# 问题二、 为什么要使用 Blazor?

Blazor 是为那些不熟悉 JavaScript 的开发人员开发的，他们大多拥有 c#和。net 背景。Blazor 提供了以下优点：

- 用 C#代替 JavaScript 编写代码。
- 利用现有.NET 库的.NET 生态系统。
- 在服务器和客户端之间共享应用程序逻辑。
- 受益于.NET 的性能、可靠性和安全性。
- 在 Windows、Linux 和 macOS 上使用 Visual Studio 保持生产力。
- 建立在一套通用的语言、框架和工具之上，这些语言、框架和工具稳定、功能丰富且易于使用。

# 问题三、什么类型的应用程序可以使用 Blazor 来构建？

Blazor 支持两种类型的应用，Blazor Sever 应用和 Blazor WebAssembly 应用。

![Blazor Interview Questions And Answers1](/photos/blazor-interview-questions/photo-2.jpg)

Blazor Server 应用程序在运行在服务端的 ASP.NET Core 应用程序中，并通过 SignalR 连接处理用户交互。

Blazor WebAssembly 应用程序通过 WebAssembly 在客户端浏览器中运行。

# 问题四、Blazor 中的组件是什么？

组件是 Blazor 的一个基本概念，所有 Blazor 应用都基于组件。Blazor 中的所有 UI 元素都是组件，比如页面、对话框或数据输入表单。从代码的角度来看，Blazor 中的组件是内置于.Net 程序集中的 c#类，它们定义了 UI 呈现逻辑并处理控件和页面事件。

Blazor 组件使用 Razor、HTML 和 C# 代码的组合。组件是 Blazor 应用的基础元素，也就是说，在 Blazor 中，每个页面都被视为一个组件。下面是一个简单的客户端 Blazor 应用，展示了默认的页面组件代码。

```
@page "/"
<h2>@Title</h2>
@functions {
    const string Title = "Blazor";
}
```

在上面的代码中，HTML 和后面的代码写在同一个 HTML 文件中。组件的 HTML 部分包含 Razor 语法和 HTML 标记，后面的代码包含实际的逻辑。简而言之，在这个方法中，我们可以在同一个页面上添加视图标记和逻辑。逻辑通过使用一个函数块来分隔。

在函数块中，我们可以定义视图标记中使用的所有属性，并将方法作为事件与控件绑定。

在 Blazor 中，我们还可以使用 code-behind 的方式来编写组件，让 C# 代码编写在一个单独的文件中。

---

未完待续...

```
# 问题五、Blazor 的生命周期与生命周期方法分别是什么？
# 问题六、什么是 Blazor WebAssembly？
# 问题七、什么是 Blazor Server？
# 问题八、Blazor 支持那些平台？
# 问题九、Blazor 中的模板组件是什么？
# 问题十、Blazor 中的路由是什么？
# 问题十一、Blazor 中的数据绑定是什么？
# 问题十二、如何将 Blazor 应用程序部署到 Azure 中？
# 问题十三、如何将 Blazor 应用程序部署到 IIS 中？
# 问题十si、Blazor 中的授权是什么？
# 问题十五、Blazor Server 和 Blazor WebAssembly 之间有什么不同？
# 问题十六、如何知道什么时候该用 Blazor Server 还是 Blazor WebAssembly 呢？
# 问题十七、可以同时使用 Blazor WebAssembly 来构建全栈应用程序吗？
# 问题十八、如何调试 Blazor WebAssembly？
```
