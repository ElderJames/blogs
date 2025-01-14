---
title: 'Blazor 如何避免组件内部修改过的属性值被外部覆写'
slug: 'blazor-prevents-property-be-overwritten-externally'
date: 2025-01-15
tags: [.NET, Blazor]
toc: true
summary: '在开发组件时，我们经常会在组件内部修改组件的某个属性，来达到控制组件功能的目的。但是属性值被修改后，又会在组件外部重新渲染时被覆盖为外部的值。那么，组件内部修改的属性值，如何避免被外部覆写呢？'
---

在开发组件时，我们经常会在组件内部修改组件的某个属性，来达到控制组件功能的目的。但是属性值被修改后，又会在组件外部重新渲染时被覆盖为外部的值。那么，组件内部修改的属性值，如何避免被外部覆写呢？

我们比较熟悉 Vue 的朋友可能会知道，Vue 也有遇到这种问题，因此 props 是有约束不可以被组件内部修改的。往往需要用一个计算属性或者data来另外存储组件内部修改的属性值，然后再使用这个计算属性或者data来接受外部修改的新值。

在 Blazor 中，没有属性修改约束，大家在开发时可以自由地修改属性值。但是从内部修改了属性之后，根据组件的生命周期，当父级组件执行渲染时，会重新设置组件的属性，在 `SetParametersAsync` 方法中会逐一对比组件内当前的属性值与外部传入的值，如果不相等就会赋值给属性，所以内部值就被覆盖了。
举个例子，当要开发一个组件，里面有一个 `Visible` 属性用于控制组件的显示/隐藏状态。要求支持在组件外部用 `Visible` 属性控制，又需要组件内部本身有按钮/图标等来控制打开/关闭时，会出现以下一些情况：

1. 默认Visible=false，点击组件内部的按钮变为 Visible=true，然后组件外部重新渲染，则会重新赋值 Visible=false（因为外部属性赋值就是false）。
2. 外部将Visible从false改为true，组件显示了，内部按钮点击关闭，将Visible=false，然后组件外部重新渲染，则会重新赋值 Visible=true（因为外部属性赋值就是true）。

... 以此类推，总之就是因为**内外属性值不同步**导致的内部无法正常改变状态的问题。

那应该怎么解决呢？我目前有两种方法，欢迎有更好的思路的朋友赐教。

### 1. 使用 @bind-value 双向绑定

我们知道双向绑定就是在组件内部修改组件外部通过 `@bind` 绑定的变量，以达到属性值在组件内外的同步。

但是要实现双向绑定，组件的外部和内部都增加了不少的代码。比如组件内要增加属性 VisibleChanged，并注意在属性值需要被修改时调用 `VisibleChanged.InvokeAsync(newValue)`。然后在使用组件时，又要用户自己注意用 `@bind-Visible` 的方式来绑定值，不看文档及其容易漏掉。如果没有双向绑定的写法，一样会遇到同样的问题。


### 2. 增加内部字段，使用属性setter同步外部的值

我认为这种方式更加适合组件的开发，让组件本身更有自主性。

首先，我们先在组件内定义一个字段例如 `_isVisible`，在组件内部只依据 `_isVisible` 来控制显示/隐藏，组件内部的按钮/图标等点击时，修改 `_isVisible` 即可。

这样就解决了组件内部状态自治的问题，那怎么解决外部通过属性来控制这个状态呢？

当然还是要定义属性 `Visible` 并且是完整属性，即包含了 `get` 和 `set` 方法和承接属性的字段 `_visible`。然后在 `set` 方法中，判断 `_visible!= value` 时更新 `_isVisible`。

```

private bool _visible;
private bool _isVisible;

[Parameter] 
public bool Visible
{
    get => _visible;
    set
    {
        if (_visible != value)
        {
            _visible = value;
            _isVisible = value;
        }
    }
}
```

这样，`_visible` 负责存储外部传入的属性值，`_isVisible` 负责存储组件内部修改的属性值，判断 `visible != value` 则达到了避免覆写的目的。

当用户没有用 `Visible` 来控制时，就是默认 false，就算被重新赋值，也会被这个判断拦住，不去修改 `_isVisible`。

当用户要使用 `Visible` 属性来控制组件时， `_isVisible` 的值会跟着 `_visible` 变化的。就是不同步，最多是当 `_visible` 变为 `_isVisible` 相等，外部看起来操作无效，但是最终状态的一致的。

这个写法我自己认为很像汽车的离合器，可以让组件自转，又能在需要时从外部控制。