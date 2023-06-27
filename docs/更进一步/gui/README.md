# GUI

## 图形用户界面

有许多方式向 Babylon.js 添加 GUI。本节介绍 Babylon.GUI。

它允许您将button和label放置在 3D 空间或 2D 屏幕空间中。当您想要一个在 VR 或 3D 空间中工作的 GUI 时，它是唯一的选择。

它集成在PlayGround内。对于您自己的项目，则必须单独加载它。

[简单的GUI滑动条示例](https://playground.babylonjs.com/#NGS9AU)

其他可用的 GUI 有：

1. CastorGUI，覆盖场景之上的 Babylon.js 社区扩展
2. Dat.GUI，一个外部扩展
3. HTML GUI

## CastorGUI

另一种 2D GUI 是扩展 CastorGUI。必须为PlayGround和您自己的项目加载它。

可以在 [Github](https://github.com/dad72/CastorGUI) 上找到。

[CastorGUI示例](https://playground.babylonjs.com/#S34THY#14)

## Dat.GUI

外部扩展 [dat.GUI](https://github.com/dataarts/dat.gui) 集成在 Playground 中。对于您自己的项目，必须加载它以及 Babylon.js。

[dat.GUI示例](https://playground.babylonjs.com/#NGS9AU#1)

## HTML GUI

由于 Babylon.js 采用 JavaScript，因此可以使用 HTML 和 CSS 来覆盖 Babylon.js 场景。

[HTML GUI 示例](https://playground.babylonjs.com/#1AHPN5)

## GUI 方案的比较

以下列出了使用不同类型 GUI 的优缺点。值得注意的是，这些选项并不相互排斥 - 它们可以根据要求一起使用，例如：

* 在过渡到 Babylon 2D 或 3D GUI 之前，使用简单的 HTML GUI 进行初始快速原型设计
* 对复杂、动态、文本较多的信息面板使用 HTML GUI 覆盖层，而对其他所有内容使用 Babylon 2D GUI

一切皆有可能！您可以尝试任何您喜欢的方式。

### HTML GUI

#### 优点

* 使用熟悉的 HTML、CSS 和前端框架，如 Bootstrap、Tailwind、React、Vue、Svelte 和 Angular 等
* 近乎无限的灵活性和移动响应能力
* 高性能（由本机浏览器而不是 3D 引擎渲染）
* 更容易使 WCAG 可访问性合规

#### 缺点

* 与 3D 场景的松散集成
* 不能直接在 3D 场景中使用 GUI 元素（例如应用于网格）
* 无法将 3D 后处理效果应用于重叠的 HTML GUI 元素
* 无法用于全屏/原生 VR

### Babylon 2D GUI

#### 优点

* 现在有一个很棒的 GUI 编辑器，可以让界面创建变得更容易！
* 与3D场景紧密结合
* 还可以选择将场景后处理效果应用于 GUI
* 能够应用/链接 GUI 元素和网格
* 一些独特、有用的功能，例如九块拉伸和 sprite-sheet 动画，这些功能在原始 HTML GUI 中不可用

#### 缺点

* 不如 HTML GUI 全面/灵活（但仍然足以满足大多数要求）
* 根据高级动态纹理分辨率，GUI 可能看起来有点模糊
* 可能是一些性能方面的考虑
* 无法用于全屏/原生 VR

### Babylon 3D GUI

#### 优点

* 支持全屏/原生 VR
* 与3D场景紧密结合
* 还可以选择将场景后处理效果应用于 GUI

#### 缺点

* 不如 Babylon 2D GUI 和 HTML GUI 全面/灵活
