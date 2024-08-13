---
title: '如何无需 Dockerfile 直接发布 .NET 7 应用到 Docker'
slug: 'publish-net-apps-to-docker'
date: 2022-12-09
tags: [.NET, Blazor]
toc: true
summary: .NET 7 在今年 11 月发布了，带来了几百页的性能提升和大量的新特性，其中很多新特性都让人印象深刻。本文就给大家带来直接使用 .NET SDK 构建 Docker 镜像的实战介绍
---

.NET 7 在今年 11 月发布了，带来了几百页的性能提升和大量的新特性，其中很多新特性都让人印象深刻。本文就给大家带来直接使用 .NET SDK 构建 Docker 镜像的实战介绍。

在 .NET 7 中，无需显式地指定 Dockerfile 文件，利用 .NET 的构建基础设施可以直接将应用程序发布到 Docker 容器注册表。

容器化已经在我们身边无处不在，但是老实讲，我们中的许多人还不习惯使用 Docker 或者类似的工具。不管怎样，现在每个人都在谈论 .NET 7，我也想探索一下最新 SDK 提供给我们的一个新特性。

那么，让我们看看如何仅使用内置框架工具就能将 .NET 7 应用程序直接部署到 Docker 上。为此，我使用的是安装了 Docker 的 Windows 11 机器。

如果还没安装 Docker，可以在命令行执行以下命令：

```
winget install -e --id Docker.DockerDesktop
```

# 创建简单的应用程序

首先，让我们创建一个简单的应用程序，让我们在后面看到一切都在正确地工作。为了做到这一点，我在自己的临时目录中向 .NET CLI 发出 PowerShell 命令。使用下面的命令，我们将创建一个 Blazor 应用程序:

```
dotnet new blazorserver -o BlazorDocker --no-https
```

完成之后，切换到新创建的目录并运行:

```
cd BlazorDocker
dotnet run
```