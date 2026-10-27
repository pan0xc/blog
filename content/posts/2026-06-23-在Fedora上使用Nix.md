+++
date = '2026-06-23T23:29:55+08:00'
draft = false
title = '在 Fedora 上使用 Nix'
authors = [
  "Chen Pan",
]
+++

## 介绍

> Nix is a tool that takes a unique approach to package management and system configuration.
> Learn how to make reproducible, declarative and reliable systems.
> {lang="en"}
{cite="https://nixos.org" caption="[nixos.org](nixos.org)"}

[Nix](https://nixos.org/) 是一个在软件包管理和系统配置方面采用独特方法的工具。官方声称使用 Nix 可以构建可重复、可声明式且可靠的系统。

Nix 虽然很好，但是我是忠实的 Fedora 用户，是不可以沉迷 Nix 的，沉迷的后果非常严重。

## 安装

在 Fedora 上安装 Nix 上安装很容易，我这里不使用 Nix 官网提供的脚本进行安装，而是直接使用 Fedora 的包管理器进行安装，执行以下命令即可
安装然后立即启动：

```sh
sudo dnf install nix nix-daemon
sudo systemctl enable --now nix-daemon
```

检查是否安装成功，可以试试查看 Nix 的版本:

```sh
panc@fedora ~> nix --version
nix (Nix) 2.34.7
```

如果你的输出和我的类似，那么先恭喜你踏入了 Nix 世界的第一步。

---

写这篇博客不久我就从 **Fedora** 换到了 **NixOS**，NixOS 实在太舒服了，所以呢，这篇博客就到此结尾了。
哈哈😄！
