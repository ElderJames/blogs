---
title: 'Blazor 模版方式复用渲染片段'
slug: 'blazor-reload-partial'
date: 2025-01-04
tags: [.NET, Blazor]
toc: true
summary: 本文介绍一个官方文档上没有的隐藏技巧，去复用渲染片段
---

我们在开发现代前端项目时，跟代码复用一样，也经常需要复用一些参数不同、模版相同的代码。React和Vue可以用 txs/jsx，Angular 用 ng-template 直接在模版上定义一个命名模版。而 Blazor 中，就要用 RenderFragment 类型的变量来定义“渲染片段”。

官方文档中有关于 [定义可重用 RenderFragment](https://learn.microsoft.com/zh-cn/aspnet/core/blazor/performance?view=aspnetcore-9.0#define-reusable-renderfragments-in-code) 的方式。

```razor
@RenderWelcomeInfo

@code {
    private RenderFragment RenderWelcomeInfo = @<p>Welcome to your new app!</p>;
}
```

要传参数时，用泛型，但是值要用委托了：

```razor
@RenderWelcomeInfo(123)

@code {
    private RenderFragment<int> RenderWelcomeInfo = index => @<p>Welcome to your new app @index! </p>;
}
```

当然你愿意也可以在 C# 代码中用 Builder 方式构建渲染片段。

```cs
private RenderFragment RenderWelcomeInfo = builder => builder.AddContent(0, "Welcome to your new app!");
```

这是学是定义渲染片段最简单的方式，但是强制要用标签开头，对于复杂的模版不太友好，比如我们想在开头加 @if 条件。另外对 VS 的格式化也不友好。

因此本文介绍一个官方文档上都没有的隐藏的技巧，就是用 __builder 这个 razor 文件在编译时才生成的变量，作为模版的上下文。这样就可以用普通方式写 razor 模版了。

在 razor 文件中定义一个方法，入参需要包含类型是 `RenderTreeBuilder` 参数名是 `__builder` 的参数。这样就能直接在方法内些 razor 模版代码了。如果无需访问当前组件实例的引用，可以定义为 `static` 方法。

```razor
@RenderWelcomeInfo

@code {

    void RenderWelcomeInfo(RenderTreeBuilder __builder)
    {
        @if (someCondition)
        {
            <p>Welcome to your new app!</p>
        }
    }
}
```

当需要传参数时，可以在方法上再加参数。使用时则需要用调用方法的方式，并传入隐藏的固定变量 __builder。

```razor
@{ RenderWelcomeInfo(__builder, 1); }

@code {

    void RenderWelcomeInfo(RenderTreeBuilder __builder, int index)
    {
        @if (someCondition)
        {
            <p>Welcome to your new app @index !</p>
        }
    }
}
```

可以看出 __builder 是个神奇的魔法变量，正常用户都不了解。学会这个技巧，你也就成为 Blazor 的高手了！