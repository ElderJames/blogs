---
title: 'ASP.NET Core 在 .NET 6 Preview 2 中的更新'
slug: 'asp-net-core-updates-in-net-6-preview-2'
date: 2021-03-14
tags: [.NET]
toc: true
description: .NET 6 预览版 2 现已推出！快来看看那些让人激动不已的新东西！
---

.NET 6 预览版 2 现已推出！快来看看那些让人激动不已的新东西！

<!--more-->

> 原文：[《ASP.NET Core updates in .NET 6 Preview 2》](https://devblogs.microsoft.com/aspnet/asp-net-core-updates-in-net-6-preview-2/?WT.mc_id=DT-MVP-5003987)，作者 [Daniel Roth](https://devblogs.microsoft.com/aspnet/author/danroth27/?WT.mc_id=DT-MVP-5003987)

---

[.NET 6 预览版 2 现已推出](https://devblogs.microsoft.com/dotnet/announcing-net-6-preview-2/?WT.mc_id=DT-MVP-5003987)，其中包括许多对 ASP.NET Core 的新改进。

以下是本次预览版的新内容：

- Razor 编译器更新为使用 Source Generators
- Blazor 支持自定义事件参数
- 增加 MVC 视图和 Razor 页面的 CSS 隔离
- Blazor 支持从祖先组件中推断组件的泛型类型
- Blazor 应用程序支持保留预渲染时的状态
- SignalR – 支持 Nullable 标注

# 马上开始

想开始在 .NET 6 Preview 2 中使用 ASP.NET Core，请先安装 [.NET 6 SDK](https://dotnet.microsoft.com/download/dotnet/6.0?WT.mc_id=DT-MVP-5003987)。

如果您正在 Windows 上使用 Visual Studio，我们建议安装 [Visual Studio 2019 16.10 的最新预览版](http://visualstudio.com/preview?WT.mc_id=DT-MVP-5003987)。如果您在 macOS 上，我们建议安装 [Visual Studio 2019 for Mac 8.10](https://docs.microsoft.com/visualstudio/releasenotes/vs2019-mac-preview-relnotes?WT.mc_id=DT-MVP-5003987) 的最新预览版。

# 升级现有项目

要将现有的 ASP.NET Core 应用程序从 .NET 6 Preview 1 升级到.NET 6 Preview 2，您需要：

- 更新所有 Microsoft.AspNetCore._ 包的引用到 6.0.0-preview.2._.
- 更新所有 Microsoft.Extensions._ 包的引用到 6.0.0-preview.2._.

再查看完整的 ASP.NET Core 在 .NET 6 中的[破坏性改动](https://docs.microsoft.com/dotnet/core/compatibility/6.0?WT.mc_id=DT-MVP-5003987#aspnet-core)列表。

# Razor 编译器更新为使用 Source Generators

我们在这个预览版中更新了 Razor 编译器，使用 C# Source Generators 来实现。Source Generators 在编译过程中运行，并能检查正在编译的内容，以生成额外的文件，与项目的其余部分一起编译。我们使用 Source Generators 简化了 Razor 编译器，并显著加快了构建的时间。

下图显示了使用新的 Razor 编译器构建默认的 Blazor Server 和 MVC 模板时的构建时间改进。

![Razor 构建的性能改进](https://devblogs.microsoft.com/aspnet/wp-content/uploads/sites/16/2021/03/razor-build-perf-improvements.png?WT.mc_id=DT-MVP-5003987)

# Blazor 支持自定义事件参数

Blazor 对自定义事件的支持现已扩展到支持自定义事件参数。这允许通过自定义事件向 .NET 事件处理程序传递任意数据。

例如，您可能希望在用户粘贴文本的同时接收剪贴板粘贴事件。要做到这一点，首先要为您的事件声明一个自定义的名称，以及一个 .NET ，该类将持有该事件的事件参数，通过添加以下类到您的项目中：

```cs
[EventHandler("oncustompaste", typeof(CustomPasteEventArgs), enableStopPropagation: true, enablePreventDefault: true)]
public static class EventHandlers
{
    // 这个静态类不需要包含任何成员。
    // 它只是一个让我们可以把 [EventHandler] 属性放在 Razor 编译器上配置事件类型的地方。
    // 这样将影响编译器的输出以及编辑器中的代码完成。
}

public class CustomPasteEventArgs : EventArgs
{
    // 这些属性的数据将由自定义的 JavaScript 逻辑提供。
    public DateTime EventTimestamp { get; set; }
    public string PastedData { get; set; }
}
```

一旦这些都到位了，您就会在您的 Razor 组件中得到一个叫做 `@oncustompaste` 的新事件的智能提醒。例如，在 `Index.razor` 中，您可以按以下方式使用它:

```cs
@page "/"

<p>Try pasting into the following text box:</p>
<input @oncustompaste="HandleCustomPaste" />
<p>@message</p>

@code {
    string message;

    void HandleCustomPaste(CustomPasteEventArgs eventArgs)
    {
        message = $"At {eventArgs.EventTimestamp.ToShortTimeString()}, you pasted: {eventArgs.PastedData}";
    }
}
```

然而，如果您现在实际运行代码，事件将不会触发。因为还剩下一个步骤，就是添加一些 JavaScript 代码，实际为您的新 `EventArgs` 子类提供数据。在您的 `index.html` 或 `_Host.cshtml` 文件中，添加以下内容：

```html
<!-- 您需要将这段代码直接添加到 blazor.server.js 或 blazor.webassembly.js 的 <script> 标签后面。-->
<script>
  Blazor.registerCustomEventType('custompaste', {
    browserEventName: 'paste',
    createEventArgs: (event) => {
      // 这个例子只处理粘贴文本，但你可以使用任意的 JavaScript API
      // 来处理用户粘贴其他类型的数据，如图片。
      return {
        eventTimestamp: new Date(),
        pastedData: event.clipboardData.getData('text'),
      };
    },
  });
</script>
```

这是告诉浏览器，每当本地粘贴事件发生时，它也应该引发一个自定义粘贴事件，并使用您的自定义逻辑提供事件参数数据。请注意，事件名称的约定在 .NET（事件名称前缀为 on）和 JavaScript（事件名称没有任何前缀）之间有所不同。

# 增加 MVC 视图和 Razor 页面的 CSS 隔离

现在，MVC 视图 和 Razor 页面跟 Blazor 组件一样支持 CSS 隔离。想要添加一个视图或页面特定的 CSS 文件，只需添加一个与 `.cshtml`文件名称相匹配的 `.cshtml.css` 文件即可。

`Index.cshtml.css`

```css
h1 {
  color: red;
}
```

在你的布局中添加一个链接到 {项目名}.styles.css 来引用捆绑的样式。

```html
<link rel="stylesheet" href="MyWebApp.styles.css" />
```

然后，这些样式将只应用于各自的视图和页面。

![MVC CSS isolation](https://devblogs.microsoft.com/aspnet/wp-content/uploads/sites/16/2021/03/mvc-css-isolation.png?WT.mc_id=DT-MVP-5003987)

# Blazor 支持从祖先组件中推断组件的泛型类型

在使用 Blazor 的泛型组件（如 `Grid<TItem>` 或 `ListView<TItem>`）时，Blazor 通常可以根据传递给组件的参数来推断泛型类型参数，因此您不必明确指定它们。但在更复杂的组件中，您可能会有多个泛型组件一起使用，而这些组件的类型参数是要匹配的，比如 `Grid<TItem>` 和 `Column<TItem>` 。在这些复合场景中，泛型类型参数通常需要显式指定，就像这样：

```cs
<Grid Items="@people">
    <Column TItem="Person" Name="Full name">@context.FirstName @context.LastName</Column>
    <Column TItem="Person" Name="E-mail address">@context.Email</Column>
</Grid>
```

但你真正想做的是：

```cs
<Grid Items="@people">
    <Column Name="Full name">@context.FirstName @context.LastName</Column>
    <Column Name="E-mail address">@context.Email</Column>
</Grid>
```

有必要在每个 `<Column>` 上重新指定 `TItem`，因为每个 `<Column>` 都被视为一个独立的组件，没有其他方法知道它应该与什么类型的数据一起工作。

在 .NET 6 Preview 2 中，Blazor 可以从祖先组件中推断出泛型类型参数。祖先组件必须选择性地加入这种行为，使用 `[CascadingTypeParameter]` 特性（attribute）来通过名称将类型参数级联到后代。这个特性（attribute）允许泛型类型推断出，对于有同名类型参数的子代，可以自动使用指定的类型参数。

例如，你可以定义像这样的 `Grid` 和 `Column` 组件：

`Grid.razor`

```razor
@typeparam TItem
@attribute [CascadingTypeParameter(nameof(TItem))]

...

@code {
    [Parameter] public IEnumerable<TItem> Items { get; set; }
    [Parameter] public RenderFragment ChildContent { get; set; }
}
```

`Column.razor`

```razor
@typeparam TItem

...

@code {
    [Parameter] public string Title { get; set; }
}
```

然后你可以像这样使用 `Grid` 和 `Column` 组件：

```razor
<Grid Items="@GetItems()">
    <Column Title="Product name" />
    <Column Title="Num sales" />
</Grid>

@code {
    IEnumerable<SaleRecord> GetItems() { ... }
}
```

> 注意：Visual Studio Code 中的 Razor 支持尚未更新以支持该功能，因此即使项目正确构建，你也可能遇到误判的错误提醒。这将在即将发布的工具版本中解决。

# Blazor 应用程序支持保留预渲染时的状态

Blazor 应用程序可从服务端进行预渲染，以加快应用程序可感知的加载时间。当应用程序在后台进行交互性设置时，可立即渲染预渲染的 HTML。但是，预渲染期间使用的任何状态都会丢失，必须在应用程序完全加载时重新创建。如果异步设置任何状态，那么当预渲染的 UI 被临时占位符替换，然后再次完全渲染时，UI 可能会闪烁。

为了解决这个问题，我们增加了新的 `<preserve-component-state />` tag helper，它可以支持把状态持久化到预渲染页面。在你的应用程序中，你可以使用新的 `ComponentApplicationState` 服务来决定你要持久化的状态。当状态即将被持久化到预渲染页面中时，`ComponentApplicationState.OnPersisting` 事件会被触发。然后你就可以在初始化你的组件时恢复（retrieved）任何已被持久化的状态。

下面的示例展示了如何在预渲染期间持久化默认的 `FetchData` 组件中的天气预报，然后在 `Blazor` 服务端应用程序中恢复状态以初始化该组件。

`_Host.cshtml`

```html
<body>
  <component type="typeof(App)" render-mode="ServerPrerendered" />
  ... @* 当所有组件调用后持久化组件状态 *@
  <persist-component-state />
</body>
```

`FetchData.razor`

```razor
@page "/fetchdata"
@implements IDisposable
@inject ComponentApplicationState ApplicationState

...

@code {
    protected override async Task OnInitializedAsync()
    {
        ApplicationState.OnPersisting += PersistForecasts;
        if (!ApplicationState.TryRedeemPersistedState("fetchdata", out var data))
        {
            forecasts = await ForecastService.GetForecastAsync(DateTime.Now);
        }
        else
        {
            var options = new JsonSerializerOptions
            {
                PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
                PropertyNameCaseInsensitive = true,
            };
            forecasts = JsonSerializer.Deserialize<WeatherForecast[]>(data, options);
        }
    }

    private Task PersistForecasts()
    {
        ApplicationState.PersistAsJson("fetchdata", forecasts);
        return Task.CompletedTask;
    }

    void IDisposable.Dispose()
    {
        ApplicationState.OnPersisting -= PersistForecasts;
    }
}
```

> 注意：这个预览版中提供了一个 `TryRedeemFromJson<T>` 辅助方法，但有一个已知的问题，将会在未来的更新中解决。为了解决这个问题，请先使用 `TryRedeemPersistedState` 和手动 JSON 反序列化，如上例所示。

通过使用与预渲染时相同的状态来初始化你的组件，任何昂贵的初始化步骤都只需要执行一次。新渲染的 UI 也与预渲染的 UI 相匹配，因此不会发生闪烁。

# SignalR - Nullable 注解

[ASP.NET Cpre SignalR 客户端](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client/)包中已经[启用 Nullability](https://github.com/dotnet/aspnetcore/pull/29219)。这意味着，当您 [启用 Nullability](https://docs.microsoft.com/dotnet/csharp/language-reference/builtin-types/nullable-value-types?WT.mc_id=DT-MVP-5003987)时，C# 编译器将根据您在 SignalR API 中对空值的处理提供适当的反馈。在 .NET 5 中，SignalR 服务器已经更新了 Nullability，但在 .NET 6 中做了一些修改。您可以在 [dotnet/aspnetcore #27389](https://github.com/dotnet/aspnetcore/issues/27389) 中跟踪 ASP.NET Core 对 Nullable 注解的支持。

# 给予反馈

我们希望您喜欢这个 .NET 6 中 ASP.NET Core 的预览版。我们渴望听到您对这个版本的体验。请在 [GitHub](https://github.com/dotnet/aspnetcore/issues) 上提交问题，让我们知道你的想法。

感谢你试用 ASP.NET Core!
