---
title: 'Blazor 应用内局部刷新'
slug: 'blazor-reload-partial'
date: 2024-08-12
tags: [.NET, Blazor]
toc: true
summary: Blazor 如何实现应用内刷新？即只刷新局部内容，不需要浏览刷新。
---

在大多数企业应用中，都需要页面刷新的功能。因为用户想获取最新数据，但页面上暂时没实现数据刷新功能，所以需要手动刷新页面。但又不想重新从服务器加载，那速度会很慢。

实现以下组件：

```
    public class ReloadContainer : IComponent
    {
        private RenderHandle _renderHandle;

        [Parameter] public RenderFragment? ChildContent { get; set; }

        public void Attach(RenderHandle renderHandle)
        {
            _renderHandle = renderHandle;
        }

        public Task SetParametersAsync(ParameterView parameters)
        {
            parameters.SetParameterProperties(this);
            if (ChildContent != null)
            {
                _renderHandle.Render(ChildContent);
            }

            return Task.CompletedTask;
        }

        public void Reload()
        {
            _renderHandle.Render(b => { });
            if (ChildContent != null)
            {
                _renderHandle.Render(ChildContent);
            }
        }
    }
```

在需要的地方使用，比如布局组件中：

```
@inherits LayoutComponentBase

<div class="page">
    <div class="sidebar">
        <NavMenu />
    </div>

    <main>
        <div class="top-row px-4">
            <a @onclick="Reload">Reload</a>
            <a href="https://learn.microsoft.com/aspnet/core/" target="_blank">About</a>
        </div>

        <article class="content px-4">
            <ReloadContainer @ref="reloadContainer">
                  @Body
            </ReloadContainer>
        </article>
    </main>
</div>

<div id="blazor-error-ui">
    An unhandled error has occurred.
    <a href="" class="reload">Reload</a>
    <a class="dismiss">🗙</a>
</div>

@code{
    ReloadContainer reloadContainer;

    void Reload()
    {
        reloadContainer.Reload();
    }
}
```

这样点击右上角的Reload，即可看到页面刷新。特别是在有数据要加载的页面，会看到数据会重新加载，组件也重新渲染。