# WebGPU 快照渲染

快照渲染是一种新的渲染模式，从 Babylon.js v5.0 开始可用。只有 WebGPU 支持它：在 WebGL 中激活此模式无效。

# 描述

快照渲染 (SR) 功能提高了下述某些特定场景中的性能。

它通过在一帧期间记录绘制调用并为所有后续帧重放此记录来工作。因此，为了让此模式按预期工作，场景应该大部分是静态的。请注意，您可以随时启用或禁用此模式。因此，如果在某些时候必须更改 SR 不支持的场景中的某些内容，您可以禁用该模式，应用更改并在之后重新启用 SR。

请注意，性能改进仅在 javascript 方面：GPU 性能在启用或不启用 SR 的情况下或多或少相同（不过，您仍然可以看到一些小的 GPU 性能改进，具体取决于浏览器）。

性能改进可能非常大，尤其是在使用快速 SR 模式时（有关 SR 模式的说明，请参见下文）。以下是示例部分中列出的第一个 PG 收集的一些数据：

|SR禁用|标准模式|快速模式|
|-|-|-|
|![](https://doc.babylonjs.com/img/resources/snapshot_rendering/sr_disabled.png)|![](https://doc.babylonjs.com/img/resources/snapshot_rendering/sr_standard.png)|![](https://doc.babylonjs.com/img/resources/snapshot_rendering/sr_fast.png)|

## 启用快照渲染模式

通过设置启用快照渲染模式：

````javascript
engine.snapshotRendering = true;
````

执行此操作时，下一帧将被记录（当然也会照常显示！）并且记录的快照将在所有后续帧中重播，直到您禁用该模式或使用 engine.snapshotRenderingReset()。如果调用后者，将在下一帧创建一个新快照来替换当前快照。

调用 engine.snapshotRenderingReset() 是应用 SR 当前模式不支持的更改的方法（请参阅下一节中的可用模式）：应用您的更改并调用此函数。这将破坏当前快照并指示系统通过记录下一帧（将包含您的更改）的绘制调用来创建新快照。实际上，它与执行 engine.snapshotRendering = false 相同； engine.snapshotRendering = true;。第一个任务将停止/销毁当前快照，第二个任务将要求系统在渲染下一帧时记录绘制调用。当然，要使其生效，您的更改应该在下一帧生效！例如，如果您导入一个新Mesh，则可能需要不止一个帧才能添加到场景中。因此，一旦您知道下一帧将使用新状态渲染场景，您应该确保调用 engine.snapshotRenderingReset()。

## 标准和快速模式

启用 SR 时有两种不同的模式可用：

* 标准模式 (Constants.SNAPSHOTRENDERING_STANDARD)。在此模式下，uniform buffer仍会更新，因此仍支持某种动态性：例如，您可以更新材质的某些参数，它会起作用。
* 快速模式 (Constants.SNAPSHOTRENDERING_FAST)。在这种模式下，只有场景uniform buffer由系统自动更新（意味着移动相机按预期工作）。您仍然可以“手动”更新mesh的uniform buffer，因此也支持移动/旋转mesh（请参见下面的示例）。

无论何种模式，由于给定帧的绘制调用会被记录下来并为所有后续帧重播，因此添加或删除Mesh将不起作用。如果要添加/删除Mesh并在之后重新启用它，则需要禁用 SR 模式。

## 注意事项

### 始终将 alwaysSelectAsActiveMesh 设置为 true

鉴于 SR 的工作方式，您可能总是希望为所有Mesh设置 alwaysSelectAsActiveMesh = true，因为如果此属性为 false（默认值）并且在录制快照时该Mesh并没有显示，则不会为该Mesh记录绘制调用，这意味着如果您稍后移动相机应该会使该Mesh可见才对，但它仍然不可见。

### Inspector 中的统计数据

在快速 SR 模式下，大部分常规 javascript 代码被跳过，因此检查器在 COUNT 部分显示的统计数据将不准确：所有 Active XXX 计数器以及 Total vertices 将保持为 0。

### 何时使用

很难根据模式列出所有会工作/不会工作的东西，所以使用这个新功能最简单的方法是启用它并查看启用后是否一切都按预期工作（首先尝试快速模式，然后尝试标准模式） .如上所述，如果您需要在某个时间点更新某些内容而当前 SR 模式无法处理，您始终可以禁用 SR，应用更改并重新启用 SR。

电子商务网站可能会极大地受益于此功能，因为场景通常很小，屏幕上的所有内容都可见。此外，通常没有太多动态性，当需要更新某些内容时，它是“一次”更新，因此调用 snapshotRenderingReset 或暂时禁用该功能应该有效。

### 适时开启快照渲染模式

在设置 engine.snapshotRendering = true 后，确保场景中的一切都已准备就绪，可以在下一帧渲染！事实上，一旦您将 snapshotRendering 设置为 true，下一帧就会被记录下来并在之后重播。如果某些纹理（例如）当时没有准备好，Mesh将不会在记录的帧中渲染，因此永远不会可见。您可能应该始终在 scene.executeWhenReady(...) 回调中设置 engine.snapshotRendering = true。

## 例子

这是一个演示使用快照渲染功能的PG：[Snapshot rendering](https://playground.babylonjs.com/?webgpu#SYQW69#1092)

您可以选择禁用或启用标准/快速 SR 模式。根据模式，您将看到渲染一帧所需的 javascript 时间（总帧数）和虚拟 fps（如果没有 GPU 渲染/fps 不受浏览器限制，您将拥有的 fps - 它只是1000/帧总数）。

在快速模式下，由于上述原因，灯光动画/更新bias将不起作用。更新时：

* bias，PG 调用 engine.snapshotRenderingReset() 以便在下一帧考虑bias并在那时重新创建快照
* Animate light 复选框，PG 切换到标准 SR 模式，直到您取消选中该框。碰巧移动灯在标准 SR 模式下确实有效：如果切换到 SR 禁用模式后，它将不起作用。

另请注意，在快速 SR 模式下，您必须自己处理天空位置的更新，因为系统不会更新uniform buffer（scene buffer除外）。有关更多详细信息，请参阅快速模式部分的高级用法。

这是另一个使用发光层的 PG：[Snapshot rendering with glow layer](https://playground.babylonjs.com/?webgpu#LRFB2D#182)

此 PG 使用标准 SR 模式，因为快速模式不起作用（尝试设置快速模式并亲自查看 - 但是，请参阅下一节以了解使其工作的方法）。此外，当开始调整大小时，我们需要禁用 SR 模式并仅在发光层有时间使用新大小重新创建其内部纹理时才重新启用它。这就是我们使用 setTimeout(..., 1) 重新启用 SR 模式的原因。

## 快速 SR 模式的高级用法

快速 SR 模式是最有趣的模式，因为您的场景处理速度比禁用 SR（或使用标准 SR 模式）快几个数量级。

这里有许多方法可以克服它的一些限制。

### 更新 Mesh 的位置/旋转/缩放/可见性属性

Mesh的世界矩阵和可见性属性存储在特定的 Mesh uniform buffer中。在快速 SR 模式下，此uniform buffer不会自动更新，因此如果您更新位置/旋转/缩放或可见性属性，它不会对屏幕产生任何影响。

您应该调用 mesh.transferToEffect(world) 来更新uniform buffer。

这里是一个例子：[Update mesh matrix in fast SR mode](https://playground.babylonjs.com/?webgpu#7YW416#7)

### 使用发光层

如上面的示例部分所示，在快速 SR 模式下，发光层并非开箱即用。但是，通过一些手动工作，它可以工作：

[Use glow layer in fast SR mode](https://playground.babylonjs.com/?webgpu#LRFB2D#218)

每次相机或发光层的Mesh移动/旋转时，您都需要调用 updateEffectLayer 方法。如果发光层中只有Mesh的一个子集，您可以更改方法以循环遍历此简化列表，而不是循环遍历场景的所有Mesh。

### 动画骨骼

要使骨骼动画在快速 SR 模式下工作，您只需在要设置动画的骨骼上调用 prepare 方法：

[Use bones in fast SR mode](https://playground.babylonjs.com/?webgpu#WGZLGJ#4072)