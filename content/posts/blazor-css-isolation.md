---
title: 'Blazor CSS 隔离'
slug: 'blazor-css-isolation'
date: 2024-08-20
tags: [.NET, Blazor]
toc: true
summary: 很多人遇到Blazor 样式隔离的问题，本文再解说下 Blazor 官方文档中的 CSS 隔离
---

## 样式隔离如何生效

Blazor 默认开启了样式隔离，只要把 css 文件命名为与 razor 组件文件名相同，后缀改为 razor.css，就可以了。例如组件名是 Home.razor，那css文件的文件名就是 Home.razor.css。

然后，如果不是通过官方模板创建的项目，需要在入口文件中引入 Blazor 给样式隔离生成的 css 文件，这个文件的文件名是固定的，为文件所在程序集名称加`.styles.css`后缀。例如，当前项目叫做 MyBlazorApp，那么 css 文件就是 MyBlazorApp.styles.css。

用一下代码引入这个文件：

```html
<link href="{ASSEMBLY NAME}.styles.css" rel="stylesheet">
```

在 Blazor Web 应用中，在 Components/App.razor 文件中添加。

在 Blazor WebAssembly 应用中，在 wwwroot/index.html 文件中添加。

## 样式隔离的原理

Blazor 的样式隔离，是在渲染 html 标签时，自动加上一个 Attribute ,如 `bl_1234`，然后在上文所说的隔离css文件中，给我们写的样式选择器都加这个Attribue，如 `div[bl_1234] { }`，这样就能利用属性选择器（Attribute Selector）的限制达到隔离的效果。生成的策略也是简单粗暴，

## 如何覆盖子组件或者组件库组件的样式？

Blazor 的样式隔离不会作用到子组件中，即不会给子组件的 html 标签加 Attribute。只能当前组件标签上 Attribute ，然后再用这个 Attribute 来隔离对子组件的覆盖样式（也就是可以实现不同页面的样式覆盖都不同）。

所以，要达到这个目的，有两个要求：

1. 需要被覆盖样式的子组件，需要用原生html元素包围。
2. 需要利用特定的标记 `::deep` 来连接父组件和子组件的元素，等于给父组件的元素加一个优先级，而不给子组件的元素加Attribute，否则也隔离了就无法作用到子组件上了。

看官方文档上的示例：

```
@page "/parent"

<div>
    <h1>Parent component</h1>

    <Child />
</div>
```

这里 `<Child />` 是一个子组件，被 `<div>` 包围了。它里面也是一个元素的元素。

```
<h1>Child Component</h1>
```

那么，要在父级组件覆盖这个h1元素的样式，如果直接用 div h1，就只能作用在父组件上的 h1，而 <Child /> 中的 h1 则毫无反应。因此，需要加一个 ::deep 来关联起来。

```css
duv ::deep h1 {
    color: red;
}
```

这样就能覆盖子组件的样式，而又不会影响当前组件的样式了。


## 组件库（RCL）上使用样式隔离的不同点

组件库的隔离样式生成的文件名和引入的路径与普通 Blazor 应用的有所区别。

```
_content/ClassLib/ClassLib.bundle.scp.css
```

这个文件会被自动导入到引用了这个RCL的应用的隔离css文件中，所以如果应用引用的组件库用了样式隔离，那么应用必须引入它自己的 styles.css 文件才能让组件库样式生效。