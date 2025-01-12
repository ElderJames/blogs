---
title: '如何解决 Blazor 导航时页面加载慢的问题'
slug: 'how-to-fix-slow-page-loads-when-navigating-blazor'
date: 2025-01-12
tags: [.NET, Blazor]
toc: true
summary: '大部 Blazor 分新手可能会遇到点击菜单跳转页面时，页面加载明细很慢的情况。'
---

大部 Blazor 分新手可能会遇到点击菜单跳转页面时，页面加载明细很慢的情况。其实这是对组件的生命周期不够了解。本文列举几个场景的误区，以便大家在以后的开发过程中避免。

## 1. 避免在 Oninitialized{Async} 方法中加载数据。

当路由导航到一个页面组件，就会进入这个页面组件的生命周期。在组件首次渲染时，会先调用一次Oninitialized{Async}方法。等这个方法执行完毕，才会渲染页面。如果在这个方法中加载数据，那么页面加载时间会比较长。

![](https://learn.microsoft.com/zh-cn/aspnet/core/blazor/components/lifecycle/_static/lifecycle1.png?view=aspnetcore-9.0)

因此，这个方法需要尽快执行完成，不宜在这里执行数据库查询或者 Http 请求等时间时间很长的操作，否则就会让用户有一种进入页面都很慢的感觉。那应该怎么加载数据呢？请看下节。

## 2. 执行时间较长的操作应放到后台线程。

习惯使用异步方法的开发者，可能习惯了async/await的方式调用异步方法，虽然这是标准的做法，也更好的让运行时管理线程。但是，在 Blazor 组件中的各个生命周期异步方法中，去执行异步方法时也会使UI线程的阻塞，体验上就会有卡顿的感觉。所以建议将这些操作放到后台线程，做法也简单，就是用 `Task.Run()`。要注意的是，当操作执行完需要重新渲染时，记得要在 `InvokeAsync()` 中调用`StateHasnChanged()` 方法。另外，不只是异步方法需要再后台运行，同步但耗时的方法也需要。以下是一个执行同步但耗时的方法的示例。

```csharp

@text

@code {
    private string text = "loading...";

    protected override void OnInitialized()
    {
        _ = Task.Run(() =>
        {
            for (int i = 0; i < 10000000; i++)
            {
                title += " ";
            }

            text = "done!";
            InvokeAsync(StateHasChanged);
        });
    }
}
```

## 3. 复杂组件需延迟渲染

对于复杂的组件，它们的构建初始化过程会比较耗时，所以当页面上有大量耗时的组件时，建议让这些组件延迟渲染。延迟渲染什么意思？就是优先、尽快地把页面组件渲染出来，再去渲染局部的耗时组件。

那怎么知道页面何时才被渲染？在 OnAfterRendered 方法中执行一些操作，比如记录页面加载时间，或者调用一个方法，让页面组件去执行一些操作。 

```csharp

@if (isLoad)
{
    <a>Loading...</a>
}
else
{
    <ComplexComponent />
}

@code {
    private bool isLoad = false;

    protected override void OnAfterRender(bool firstRender)
    {
        isLoad = true;
    }
}

```

如果页面中复杂组件比较多，如果统一的渲染，也会造成渲染卡顿，那么我们就需要延迟加载一些不需要展示的组件，当需要展示时再渲染。也可以先用 Spin 组件或者骨架屏组件渲染一个占位符，再一个一个地渲染这些组件。如何一个个地渲染呢？以下是一种方式：用一个集合来保存需要渲染的组件，然后边加入到集合，边调用`StateHasChanged()`方法来先渲染，而不是等到最后一起渲染。


```cs

@foreach (var item in showComponents)
{
    <DynamicComponent Type="item" />
}

@code {
    private List<Type> components = [typeof(ComplexComponent1), typeof(ComplexComponent2)];
    private List<Type> showComponents=[];

    protected override void OnAfterRender(bool firstRender)
    {
        for (var item in components)
        {
            _demos.Add(item);
            StateHasChanged();
        }
    }
}
       
```

当组件被添加到渲染树时，就会执行其内部的初始化方法，这就会造成UI线程的阻塞。所以，这个方法的原理就是将多个组件的初始化方法逐个拆分，逐个渲染。虽然最终的页面完整渲染时间会变长（执行渲染会消耗一点时间），但能够更快地渲染那些重要的部分（例如页面第一屏要显示的部分、Tabs 中第一页的部分），总体的体验反而是更好的。

当然，也不是所以场景都是顺序渲染的，这就需要用别的方式，但原理都是一样，就是做到小步快跑，让页面整体上的体验更好。

## 4. 将复杂的页面元素封装为组件

基于上一节的内容，很容易得出这个思路。因为将一个复杂的页面拆分为多个组件，是有利于延迟渲染的。否则，这个组件中的渲染树是一个整体，会一起执行渲染的，这样就不利于页面的加载速度。