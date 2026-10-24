+++
date = '2026-10-17T14:13:16+08:00'
draft = false
title = '无需 JavaScript 的对话框'
authors = [
  "Chen Pan",
]
+++

## 背景

在完成[为我的博客添加友链]({{< relref "2026-10-16-为我的博客添加友链/index.md" >}})功能后，我愈发对自己博客的移动端布局
感到不满意，不满意的点主要是 `<header>` 导航菜单不会收缩折叠，如果后续导航菜单越来越多，移动端 `<header>` 会 `overflow`，
为了解决这个问题，我决定为移动端添加**可折叠菜单**。

过去我曾经为 [Forgejo](https://forgejo.org/) / [Codeberg](https://codeberg.org/) 贡献过代码，主要是前端方面。偶尔也会参
与到他们的 issues 讨论中，在交流过程中我大致了解到他们的前端在往 js-less 方向上走，即减少 JavaScript 在 Web 页面中的使用。

受此影响，我在开始制作我的博客主题时，我也尝试将这一理念融入其中。相比于依赖 JavaScript 实现各种交互效果，我更倾向于优先
利用原生 HTML 和 CSS 来解决问题。在现代浏览器中，HTML 本身提供了许多实用的交互机制，CSS 也在不断发展，许多过去
需要 JavaScript 才能实现的功能，如今已经可以通过原生 Web 技术完成。就比如我这次要分享的原生 [dialog](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dialog)。

在我的印象里，可折叠菜单通常需要 JavaScript 来控制菜单的展开与收起，例如监听按钮的点击事件，再通过修改 CSS 类名来切换菜单
的显示状态。

而这次，我希望尝试一种不同的实现方式：使用原生的 `<dialog>` 元素。

目前 Forgejo 的部分下拉菜单和模态框也采用了原生对话框实现。这让我意识到，`<dialog>` 并不只是用来展示传统意义上的模态窗口，
它同样可以作为菜单等交互组件的基础。借助 HTML 提供的 `show-modal` 和 `close` 命令，我们甚至可以在不编写 JavaScript 的情况
下，实现菜单的打开与关闭。

因此，我决定尝试利用原生 `<dialog>` 为博客添加移动端可折叠菜单，并借此探索现代 HTML 和 CSS 在交互设计中的更多可能性。

## 设计

我的设计原则之一是简洁。在保留现有移动端布局的基础上，我将导航菜单折叠起来，并在 header 的右上角放置一个菜单按钮。用户点击
按钮后即可展开导航菜单，再通过关闭按钮或其他方式将其收起。这样既能避免导航项过多导致的布局溢出，也能尽可能减少对现有页面布
局的影响，同时也为将来的拓展提供了更多的空间，桌面端则继续保持原有的横向导航形式，不额外引入不必要的交互。

考虑到菜单按钮位于屏幕右上角，我希望菜单能够从视口底部呼出（就像 toast 从面包机中弹出来的效果），让整个交互过程更加自然、
连贯。同时，我也希望为菜单添加适当的过渡动画，使其在展开和收起时不会显得过于突兀。

## 实现

> Modal dialogs can be declaratively opened and closed using the `Invoker Commands API` HTML attributes `commandfor` and
> `command`, which can be set on `<button>` elements.
> {lang="en"}
> The `command` attribute sets the particular command that is to be sent when the `<button>` element is clicked, while
> commandfor sets the `id` of the target dialog. The commands that can be sent for dialogs are `"show-modal"`, `"close"`,
> and `"request-close"`.
> {lang="en"}
> The HTML below demonstrates how to apply the attributes to a `<button>` element so it can be pressed to open a modal
> `<dialog>` with an `id` of `"my-dialog"`.
> {lang="en"}

确定了设计目标并查阅相关文档后，实现起来就容易多了。按照文档的说明，首先准备两个 `<button>` 元素和一个 `<dialog>` 元素，
具体布局可以根据自己的需求决定。

接下来，为菜单按钮添加 `command="show-modal"`，表示点击按钮时，以模态方式打开目标对话框。与此同时，通过 `commandfor` 属性
指定需要操作的目标元素，也就是前面创建的 `<dialog>`，填入 `<dialog>` 的 `id` 即可。

相应地，为关闭按钮添加 `command="close"`，并通过 `commandfor` 指向同一个对话框。这样，点击关闭按钮时，就可以关闭已经打开的
菜单。

```html
<header>
  <div class="menu-btn">
    <button id="open-menu" command="show-modal" commandfor="menu-modal">
      打开菜单
    </button>
    <button id="close-menu" command="close" commandfor="menu-modal">
      关闭菜单
    </button>
  </div>
</header>
<dialog id="menu-modal" closedby="any">
  <nav>
    <a href="#about"  >关于</a>
    <a href="#friends">友链</a>
    <a href="#posts"  >文章</a>
  </nav>
</dialog>
```

看看效果，似乎还不错。

{{< video src="./Screencast.webm" >}}

有一处小细节（一点小巧思），不知道大家发现没有：菜单按钮和关闭按钮会根据菜单的打开状态交替显示。这里我利用了 CSS 的 `:has()` 选择器，
检测 dialog 是否处于 `[open]` 状态，再切换两个按钮的 `visibility` 属性。

另外，我将两个按钮放在同一个 CSS Grid 单元格中，使它们能够重叠显示。这样切换按钮时，就不会因为两个按钮分别占据布局空间
而导致位置发生偏移。我觉得这种方法比使用 `position: absolute / fixed` 等定位方式更加简单直观，不需要额外处理元素的定位
参照、偏移量等问题，也不必担心脱离文档流后对布局产生的影响。对于这种只需要让两个元素占据同一位置的场景，CSS Grid 无疑
是一个简洁的选择。

不过，这里使用 visibility 而不是 display 还有一个好处：隐藏的按钮仍然保留在布局中，因此按钮切换时不会引起布局变化。

## 写在最后

最后，还有一个值得记录的问题。在实际测试过程中，我发现 Firefox 在触摸模拟模式下存在一个问题：当 `<dialog>` 设置了 `closedby="any"` 时，
点击对话框外部的 backdrop 不仅会关闭对话框，触摸事件还可能继续传播到下层页面的元素，导致原本不应该触发的点击事件被执行。

我随后在 Mozilla Bugzilla 上找到了相关的 [Bug 2022750](https://bugzilla.mozilla.org/show_bug.cgi?id=2022750)。这并不是 Firefox 独有的
问题，报告中也提到了 Chrome 存在相同的行为。根据该 issue 最近的更新，Chrome 154 已经修复这一问题。

这也提醒我，即使使用的是标准化的原生 Web API，也不意味着所有浏览器都能提供完全一致的行为。尤其是在涉及触摸事件、模态对话框
和焦点管理等交互细节时，实际测试仍然不可或缺。不过，这并不影响我继续探索 js-less 的实现方式。对我而言，减少 JavaScript 依赖
并不是目的本身，而是在充分利用浏览器原生能力的同时，让代码保持简洁、易于维护。


---

**2026-10-23：** 我给菜单/关闭按钮添加切换过渡动画，在 Firefox 上对话框不能关闭的问题就解决了，但是当对话框被关闭时，还是缺少我设置的
退场动画。
