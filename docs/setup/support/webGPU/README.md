# WebGPU 支持

## 介绍

根据 Web 工作组的 GPU 规范，我们已经开始支持这项新技术的旅程。我们的计划是在 Babylon 5 中提供对 WebGPU 和 WebGL 功能对等的支持。

![](https://doc.babylonjs.com/img/extensions/WebGPU.png)

### WebGPU 的优势

WebGPU 背后的承诺是通过从 JavaScript 对系统图形资源进行较低级别的控制来提供速度异常快的 API。我们希望为开发人员带来这些额外的性能改进，以便通过他们已经熟悉的工具 Babylon.js 创建更高质量的 3D 网页游戏和体验。

WebGPU 带来的一些特性是：

* 计算管线
* 光线追踪（[进行中](https://github.com/gpuweb/gpuweb/issues/535)）
* 全面提升性能
* 更多...

## 进度

请参阅[专门的进度页面](https://doc.babylonjs.com/setup/support/webGPU/webGPUStatus)。

您还可以关注我们在 [GitHub 关于该问题的进展](https://github.com/BabylonJS/Babylon.js/issues/6443)。

WebGPU 的当前实现现已合并到 [Babylon.js GitHub 存储库](https://github.com/BabylonJS/Babylon.js)的主分支中。

我们在下一个版本中付出了巨大的努力来实现全面支持。我们非常欢迎每一份贡献，所以如果您有兴趣为网络上高性能游戏开发的未来做出贡献，请随时创建 PR。

## 潜在问题

即使我们现在已经深入开发过程，我们仍然会受到潜在变化和规格不确定性的影响。我们将一次处理这些更改，但因此我们可能不得不相应地调整发布时间表。

我们得到了将 WebGPU 实现到浏览器中的出色团队的全力支持，因此我们可以根据需要一起工作，使所有人的开发更加顺畅。

## 现有游戏和应用程序的迁移

由于向后兼容性是我们的支柱之一，我们希望拥有的唯一区别是需要异步的引擎初始化：

````javascript
const engine = new BABYLON.WebGPUEngine(canvas);
await engine.initAsync();
````

## 仍会支持 WebGL?

是的！在可预见的未来，对 WebGL 和 WebGPU 的支持将同时保持。

## 测试 WebGPU

您可以参考[此页面](https://github.com/gpuweb/gpuweb/wiki/Implementation-Status)以获取有关浏览器支持的详细信息。

假设你使用的是支持 WebGPU 的浏览器，你可以在 [Playground](https://playground.babylonjs.com/) 中自己尝试一下。

![](https://doc.babylonjs.com/img/extensions/webGPUPlayground.jpg)

从专门的 [Chrome 状态平台页面](https://chromestatus.com/feature/6213121689518080)关注状态。

所有演示代码都可以在 [Github](https://github.com/BabylonJS/Website/tree/master/build/Demos/WebGPU) 上找到，因此您可以比较 WebGL 和 WebGPU 版本，并注意目前除了初始化之外没有任何区别。我们将努力保持这种状态。 :-)