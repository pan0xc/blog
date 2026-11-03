+++
date = '2026-11-03T23:04:39+08:00'
draft = false
title = '修复 NixOS 中 QQ 的 Desktop Entry 路径'
authors = [
  "Chen Pan",
]
+++

今天无意间在 [search.nixos.org](https://search.nixos.org/packages) 搜索 QQ 的包信息，在翻阅打包源码的过程中，
发现其存在一个看起来很小、但值得记录的问题：上游提供的 `qq.desktop` 文件使用了硬编码的 FHS 路径，而这与 Nix 的
软件包组织方式并不完全契合。

## 问题的探讨

QQ 上游提供的 `qq.desktop` 大致如下：

```sh
[Desktop Entry]
...
Exec=/opt/QQ/qq %U
Icon=/usr/share/icons/hicolor/512x512/apps/qq.png
...
```
这里有两个值得注意的地方：
- Exec 使用了 `/opt/QQ/qq` 作为可执行文件路径。
- Icon 使用了 `/usr/share/icons/...` 作为图标路径。

完整地指定可执行程序的路径和图标的路径，显得不那么灵活，我查看了诸如 VSCode、Ghostty 及 LibreWolf，发现他们
都只给出程序名称，这么做有什么好处呢？

这里先说明一下，这里讨论的完整地指定与之相对的是是否将可执行程序加入到 `$PATH` 环境变量中。

对于可执行程序，加入到 `$PATH` 环境变量中，意味着调用方不必关心程序实际安装在哪里，只需要知道它的命令名称即可，
所以在 Desktop Entry 中的 Exec 可以直接填写命令名称。

对于图标，如果完整地指定了路径，那么用户的主题图标对该程序就不会生效了。

查看用户 Nix profile 中的 `qq.desktop`：

```sh
panic@nixos /e/p/p/panic> cat /etc/profiles/per-user/panic/share/applications/qq.desktop
[Desktop Entry]
...
Exec=/nix/store/wh392xpxl87h2n6z1apgqbq4nggljk78-qq-3.2.34-2026-09-24/bin/qq %U
Icon=/nix/store/wh392xpxl87h2n6z1apgqbq4nggljk78-qq-3.2.34-2026-09-24/share/icons/hicolor/512x512/apps/qq.png
...
```

不难发现，Exec 和 Icon 都使用了 `/nix/store` 中的绝对地址，这显然是不够灵活和优雅的。

另外，我们认真去看它的包源码，从 `makeShellWrapper $out/opt/QQ/qq $out/bin/qq` 来看，它其实已经把可执行程序加入
到了 `/bin` 目录中（是否意味着已经加入到了`$PATH` 环境变量中？），因此我们只需要填写命令名称即可。

```nix
mkdir -p $out/bin
cp -r opt $out/opt
cp -r usr/share $out/share
substituteInPlace $out/share/applications/qq.desktop \
  --replace-fail "/opt/QQ/qq" "$out/bin/qq" \
  --replace-fail "/usr/share" "$out/share"
makeShellWrapper $out/opt/QQ/qq $out/bin/qq \
```

图标也是一样的，从 `cp -r usr/share $out/share` 来看，它肯定也是把图标复制过去了，所以只需要填写图标名称就好了（试试看？）。

讨论到这里，我对 NixOS 的软件包组织方式也有了进一步的认识。

NixOS 并不遵循传统 Linux 发行版的 FHS（Filesystem Hierarchy Standard）目录布局。软件包不会像传统发行版那样，将文件
直接安装到 `/usr/bin`、`/usr/share` 或 `/usr/lib` 等公共目录中，而是被独立地构建到 `/nix/store` 下的专属路径。

不过，当我们进入用户的 Nix profile 时，会发现另一番景象：

```sh
/etc/profiles/per-user/panic/
├── bin/
├── etc/
├── lib/
├── sbin/
└── share/
    ├── applications/
    ├── icons/
    └── ...
```

这里依然保留了许多熟悉的 Unix/Linux/FHS 目录结构。

但是实际上，他们只是一些符号链接，指向了各自的 `/nix/store` 路径。例如，在 `bin/` 目录中的：

``` sh
bin/
├── amberol -> /nix/store/ff2skwl5vaq9x59db2y5mpwlgc5ql9sl-amberol-2026.1/bin/amberol
├── hx -> /nix/store/2h9dsm7dxfh1lwhcvjqia601g8chlyrw-helix-25.07.1/bin/hx
└── ...
```

因此，Nix profile 在一定程度上保留了传统 Unix/Linux/FHS 软件生态熟悉的目录约定，同时又不需要将软件包实际安装到这些全局目录中。

这也解释了为什么 Desktop Entry 可以使用 Exec=qq 和 Icon=qq：前者可以通过 Nix profile 暴露的 `bin/` 目录查找程序，后者则
可以通过 Nix profile 暴露的 `share/icons/` 目录进行图标主题查找。

## 在 nixpkgs 中修复

对其中两个不适合 Nix 的字段进行替换：

```nix
substituteInPlace $out/share/applications/qq.desktop \
  --replace-fail "Exec=/opt/QQ/qq" "Exec=qq" \
  --replace-fail "Icon=/usr/share/icons/hicolor/512x512/apps/qq.png" "Icon=qq"
```

相比于原来直接替换路径前缀，我的修改明确指定 Desktop Entry 中需要修改的完整字段，对于后续维护者而言，能够直接从代码中辨
认出 Exec 和 Icon 分别代表什么，以及为什么需要修改它们，是很有必要的。

## 不严谨的验证

在着手修改之前，我向 NixOS 群群友请教了如何测试自己修改的 nixpkgs，群友 [Justin Chen](https://blog.starryreverie.cc/) 及
其他群友向我提供了帮助，教我如何进行 nix build，非常感谢。

修改完成后，我使用本地 nixpkgs 工作树构建 QQ：

```sh
NIXPKGS_ALLOW_UNFREE=1 nix build --impure .#qq
```

然后执行 `cat result/share/applications/qq.desktop` 检查生成的 Desktop Entry：

```sh
[Desktop Entry]
...
Exec=qq %U
Icon=qq
...
```

符合预期，看起来没问题了，程序也能跑。

## 结束了吗？

我仍然是 NixOS 用户中的新手或是低手。

NixOS 的浩瀚星空，仍然值得我亲自去探索。
