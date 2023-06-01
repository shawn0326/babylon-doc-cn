# 优化您的场景

## 如何优化场景

本教程将帮助您找到有关如何改进场景渲染性能的一些链接和信息。

## 使用 TransformNode 而不是 AbstractMesh 或 空Mesh

如果您需要节点容器或变换节点，请不要使用Mesh，而应使用 TransformNode。仅当与要呈现的内容关联时才使用Mesh。

Mesh需要经过一个评估过程，相机会检查它们是否在平截头体中。这是一个昂贵的过程，因此在可能的情况下使用 TransformNode 来减少Mesh的数量是一个很好的做法。

## 减少着色器开销

Babylon.js 使用先进的自动着色器引擎。该系统将使着色器保持最新的材质选项。如果您使用的是静态材质（即不可变材质），那么您可以使用以下代码将其告知 Babylon.js：

````javascript
material.freeze();
````

一旦冻结，着色器将保持不变，即使您更改材质的属性也是如此。您必须解冻它才能更新内部着色器：

````javascript
material.unfreeze();
````

## 减少世界矩阵计算

每个Mesh都有一个世界矩阵来指定它的位置/旋转/缩放。该矩阵在每一帧上进行评估。您可以通过冻结此矩阵来提高性能。然后将忽略对位置/旋转/缩放的任何后续更改：

````javascript
mesh.freezeWorldMatrix();
````

您可以使用以下方法解冻Mesh：

````javascript
mesh.unfreezeWorldMatrix();
````

## 冻结活动Mesh

如果您受 CPU 限制，您可以决定保持活动Mesh列表不变，然后释放 CPU 确定活动网格所花费的时间：

````javascript
scene.freezeActiveMeshes();
````

您可以使用以下方法解冻活动Mesh：

````javascript
scene.unfreezeActiveMeshes();
````

请注意，您可以在冻结列表之前设置 mesh.alwaysSelectAsActiveMesh = true 强制Mesh处于活动Mesh中。

冻结活动Mesh列表可能会导致场景中的某些事物停止更新。一个例子是 RenderTargetTexture，如果它被Mesh材质使用的话。对于这种情况，RTT 需要明确添加到活动相机的自定义渲染目标列表中，这将保证它最终出现在场景的渲染列表中：

````javascript
camera.customRenderTargets.push(renderTargetTexture);
````

## 不更新边界信息

结合 mesh.alwaysSelectAsActiveMesh，您还可以决定关闭边界信息同步。这样世界矩阵计算会更快，因为边界信息不会更新（如果你想使用拾取或碰撞，这可能是个问题）：

````javascript
mesh.doNotSyncBoundingInfo = true;
````

## 光标移动时不选择场景

在每次光标移动时，场景都会浏览Mesh列表以查看光标下的Mesh是否需要引发关联的动作/事件。为了避免这个过程，你可以设置scene.skipPointerMovePicking = true。

请注意，通过这样做，当光标移动时，您将不会在任何Mesh上触发任何事件（即使 scene.constantlyUpdateMeshUnderPointer === true，scene.meshUnderPointer 也不会更新。

## 减少 draw call

请尽量使用实例化渲染，因为它们是通过一次绘制调用绘制的。

如果不能共享相同的材质，那么您可以考虑使用与 mesh.clone("newName") 共享相同几何体的克隆。

关于实例化渲染的评论：如果其中一个实例的世界矩阵具有不同的行列式（例如，一个实例具有负缩放而其他实例没有），babylon.js 将被迫从其材质中移除back face culling。

## 减少对 gl.clear() 的调用

默认情况下，Babylon.js 在渲染场景之前自动清除颜色、深度和模板缓冲区。它还会在切换到新相机之后和渲染新 RenderingGroup 之前清除深度和模板缓冲区。在填充率低的系统上，这些会迅速加起来并对性能产生重大影响。

如果您的场景设置为视口始终 100% 填充不透明几何体（例如，如果您始终在天空盒内），您可以禁用默认场景清除行为：

````javascript
scene.autoClear = false; // Color buffer
scene.autoClearDepthAndStencil = false; // Depth and stencil, obviously
````

如果您知道特定 RenderingGroup 中的几何体将始终位于其他组的几何体之前，则可以使用以下命令禁用该组的缓冲区清除：

````javascript
scene.setRenderingAutoClearDepthStencil(renderingGroupIdx, autoClear, depth, stencil);
````

autoClear: true 启用自动清除。如果为false，则覆盖深度和模板

depth：默认为 true 以启用深度缓冲区的清除

stencil：默认为 true 以启用模板缓冲区的清除

继续并积极使用这些设置。如果您看到任何污迹，您就会知道它是否不适合您的应用！

## 使用 depth pre-pass

在处理复杂场景时，使用depth pre-pass可能很有用。该技术将仅在深度缓冲区中渲染指定的Mesh，以利用early depth test。例如，当场景包含具有高级着色器的Mesh时，可以使用它。要为Mesh启用depth pre-pass，只需调用 mesh.material.needDepthPrePass = true。

## 使用非索引的Mesh

默认情况下，Babylon.js 使用索引Mesh，其中顶点可以被面重用。当顶点重用率较低且顶点结构相当简单时（例如position和normal），您可能希望展开顶点并停止使用索引：

````javascript
mesh.convertToUnIndexedMesh();
````

例如，对于简单的立方体数据来说，传输 32 个position比 24 个position和 32 个index更有效。

## 关闭/打开 AdaptToDeviceRatio

默认情况下，Babylon.js 不再适配device ratio。在收到大量社区反馈后，它默认关注性能与质量。

缺点是这可能看起来不太真实。您可以使用 Engine 构造函数的第四个参数将其打开：

````javascript
var engine = new BABYLON.Engine(canvas, antialiasing, null, true);
````

## 阻塞标脏机制

默认情况下，当您更改可能影响它们的属性（alpha、纹理更新等）时，场景将使所有材质保持最新。为此，场景需要遍历所有材质并将它们标记为脏。如果您有很多材质，这可能是一个潜在的瓶颈。

要阻止这种自动更新，您可以执行：

````javascript
scene.blockMaterialDirtyMechanism = true;
````

完成批量更改后，不要忘记将其恢复为 false。

## 使用 Animation Ratio

Babylon.js 处理速度取决于当前帧速率。

在低端设备上，动画或相机移动可能与高端设备不同。为了弥补这一点，您可以使用：

````javascript
scene.getAnimationRatio();
````

低帧率的返回值更高。

## 处理 WebGL 上下文丢失

从 3.1 版本开始，Babylon.js 可以处理 WebGL 上下文丢失事件。当 GPU 需要从您的代码中移除时，浏览器会引发此事件。例如，在混合场景（具有多个 GPU）中使用 WebVR 时可能会发生这种情况。在这种情况下，Babylon.js 必须重新创建所有低级资源（包括纹理、着色器、程序、缓冲区等）。该过程完全透明，由 Babylon.js 在幕后完成。

作为开发人员，您不应该担心这种机制。但是，为了支持这种情况，Babylon.js 可能需要额外的内存量来跟踪资源创建。如果您不需要支持 WebGL 上下文丢失事件，您可以通过实例化您的引擎并将 doNotHandleContextLost 选项设置为 true 来关闭跟踪。

如果您创建了需要重建的资源（如顶点缓冲区或索引缓冲区），您可以使用 engine.onContextLostObservable 和 engine.onContextRestoredObservable 可观察对象来跟踪上下文丢失和上下文恢复事件。

## 有大量Mesh的场景

如果场景中有大量Mesh，并且需要减少在场景中添加/删除这些Mesh所花费的时间，Scene构造函数的几个选项可以提供帮助：

* 将选项 useGeometryIdsMap 设置为 true 将加速场景中 Geometry 的添加和删除。
* 将选项 useMaterialMeshMap 设置为 true 将通过减少查找绑定网格所花费的时间来加速处理材质。
* 将选项 useClonedMeshMap 设置为 true 将通过减少查找关联的克隆网格所花费的时间来加速网格的处理。

对于每个打开的选项，Babylon.js 都需要额外的内存量。

此外，如果您连续处理大量Mesh，您可以通过在处理Mesh之前将场景属性 blockfreeActiveMeshesAndRenderingGroups 设置为 true 并在之后将其设置回 false 来节省不必要的计算，如下所示：

````javascript
scene.blockfreeActiveMeshesAndRenderingGroups = true;
/*
 * Dispose all the meshes in a row here
 */
scene.blockfreeActiveMeshesAndRenderingGroups = false;
````

## 改变Mesh剔除策略

剔除是选择是否必须将Mesh传递给 GPU 进行渲染的过程。它在 CPU 端完成。
如果Mesh以某种方式与相机平截头体相交，则会将其传递给 GPU。
根据其准确性（仅检查Mesh的包围盒或包围球体，尝试从截锥体中快速包含或排除网格），此过程可能非常耗时。
另一方面，降低这个过程的准确性以使其更快可能会导致一些误报：一些网格被传递到 GPU，在那里计算并且最终不会在视口中可见。
默认情况下，BABYLON 应用“Bounding Sphere Only”排除测试来检查网格是否在相机视锥体中。
您可以随时为场景的任何Mesh更改此行为（如果需要，然后再更改）此属性 mesh.cullingStrategy。

````javascript
/**
* Possible values : 
* - BABYLON.AbstractMesh.CULLINGSTRATEGY_STANDARD  
* - BABYLON.AbstractMesh.CULLINGSTRATEGY_BOUNDINGSPHERE_ONLY  
* - BABYLON.AbstractMesh.CULLINGSTRATEGY_OPTIMISTIC_INCLUSION  
* - BABYLON.AbstractMesh.CULLINGSTRATEGY_OPTIMISTIC_INCLUSION_THEN_BSPHERE_ONLY  
*/

mesh.cullingStrategy = oneOfThePossibleValues;
````

* 标准：最准确和标准的。这是一个排除测试。测试顺序是：
    * 包围球在平截头体之外吗？
    * 如果不是，包围盒顶点是否在平截头体之外？
    * 如果不是，则可剔除对象位于截锥体中。
* 仅限包围球：更快但不太准确。这是一个排除测试。它比标准策略更快，因为没有测试包围盒。它也不如标准准确，因为仍然可以选择一些不可见的对象。测试顺序是：
    * 包围球在平截头体之外吗？
    * 如果不是，则可剔除对象位于截锥体中。
* 乐观包含：这首先是包含测试，然后是标准排除测试。当预期可剔除对象几乎总是位于相机视锥体中时，这会更快。当被测试的对象中心不是截锥体但其包围盒顶点之一仍在内部时，这也可能比标准测试慢一点。不管怎样，它和标准策略一样准确。测试顺序是：
    * 可剔除对象包围球的中心是否位于平截头体中？
    * 如果不是，则应用默认的剔除策略。
* Optimistic Inclusion Then Bounding Sphere Only ：这首先在包含测试中，然后是包围球仅排除测试。当预期可剔除对象几乎总是位于相机视锥体中时，这可能是最快的测试。这也可能比 BoundingSphereOnly 策略慢一点，当被测试的对象中心不在视锥体中但它的边界球体仍然与它相交时。它不如标准策略准确，但与 BoundingSphereOnly 策略一样准确。测试顺序是：
    * 可剔除对象包围球的中心是否位于平截头体中？
    * 如果不是，请应用 Bounding Sphere Only 策略。这里没有测试 Bounding Box。

乐观包容模式会带来一点好处。它们在所应用的内容（标准或 bSphereOnly）上保持与基本模式相同的准确性。
BoundingSphereOnly 模式，因为它们降低了很多精度，所以提供了良好的性能增益。这些不应与高多边形网格一起使用，因为向 GPU 发送误报会产生实际渲染成本。相反，这些对于许多低多边形网格来说可能非常有趣。*如果您受 CPU 限制，这真的很有用*。

## 性能优先模式

从 Babylon.js 5.22 开始，您现在可以更改场景在向后兼容性和易用性方面处理性能的方式。

### 向后兼容模式（默认）

默认情况下，scene.performancePriority 设置为 BABYLON.ScenePerformancePriority.BackwardCompatible。在这种模式下，根本没有变化。该场景将继续优先考虑易用性和向后兼容性。

### 中间模式

如果将 performancePriority 切换为 BABYLON.ScenePerformancePriority.Intermediate，场景将自动：

* 准备好后将材质freeze。如果您需要更改材质上的某些内容，则必须调用 material.unfreeze()，进行更改，然后再次调用 material.freeze()
* 新Mesh会将其 alwaysSelectAsActiveMesh 属性设置为 true。然后系统将跳过网格的截头体裁剪并始终将其设置为活动状态（节省复杂的 CPU 操作）。如果您的场景受 GPU 限制，请记住将其关闭
新Mesh会将其 isPickable 属性设置为 false。pick 和 action 管理器将不再工作。如果您需要为特定Mesh拾取，您可以随时重新打开该属性
* scene.skipPointerMovePicking 将被打开（意味着不会有 OnPointerMove 事件）
* scene.autoClear 将被关闭

### 激进模式

如果将 performancePriority 切换为 BABYLON.ScenePerformancePriority.Aggressive，场景将自动：

* 启用中级模式的所有功能

* 场景将完全跳过所有截头体裁剪阶段（scene.skipFrustumClipping 将设置为 true）

* 新Mesh会将其 doNotSyncBoundingInfo 设置为 true

* 管理器不会在帧之间重置（scene.renderingManager.maintainStateBetweenFrames 设置为 true）。这意味着如果Mesh变得不可见或透明，它将不可见，直到再次将此布尔值设置为 false

请注意，Intermediate 和 Aggressive 模式不会向后兼容，这意味着我们将来可能会在这些模式中添加更多功能以支持性能优先

这里是一个例子：[性能模式示例](https://playground.babylonjs.com/#6HWS9M)

## 仪器仪表

当您想要优化场景时，仪器是一个关键工具。它将帮助您找出瓶颈在哪里，以便您能够优化需要优化的内容。

### EngineInstrumentation

EngineInstrumentation 类允许您获取以下计数器：

* gpuFrameTimeCounter：GPU 渲染单帧所花费的时间（以纳秒为单位）。必须通过 instrumentation.captureGPUFrameTime = true 打开。
* shaderCompilationTimeCounter：CPU 编译所有着色器所花费的时间（以毫秒为单位）。必须通过 instrumentation.captureShaderCompilationTime = true 打开。

以下是如何使用引擎检测的示例：[引擎检测示例](https://playground.babylonjs.com/#HH8T00#1)

请注意，每个计数器都是 PerfCounter 对象，它可以提供多个属性，如平均值、总计、最小值、最大值、计数等。

GPU 计时器需要一个特殊的扩展（EXT_DISJOINT_TIMER_QUERY - 有时在 WebGL2 上下文中报告带有 _webgl2 前缀）才能工作。由于所有主流浏览器上的 Spectre 和 Meltdown，此扩展已被禁用，但有些已将其重新添加，例如 Chrome 和 Edge。您可以在 Khronos SDK 测试页面或 WebGL 报告中检查您的浏览器是否支持此扩展。

* 注意：在 Chrome 移动设备上，可以在 chrome://flags#enable-webgl-developer-extensions 中启用此扩展，然后重新启动浏览器

### SceneInstrumentation

SceneInstrumentation 类允许您获得以下计数器（每个场景）：

* activeMeshesEvaluationTimeCounter：评估活动Mesh（基于活动相机平截头体）所花费的时间（以毫秒为单位）。必须通过 instrumentation.captureActiveMeshesEvaluationTime = true 打开。
* renderTargetsRenderTimeCounter：渲染所有RenderTargetTexture所花费的时间（以毫秒为单位）。必须通过 instrumentation.captureRenderTargetsRenderTime = true 打开。
* drawCallsCounter：每帧的绘制调用次数（对 engine.draw 的实际调用）。一个好的建议是使这个数字尽可能小。
* frameTimeCounter：处理整个帧（包括动画、物理、渲染目标、特殊 fx 等）所花费的时间（以毫秒为单位）。必须通过 instrumentation.captureFrameTime = true 打开。
* renderTimeCounter：渲染一帧所花费的时间（以毫秒为单位）。必须通过 instrumentation.captureRenderTime = true 打开。
* interFrameTimeCounter：两帧之间花费的时间（以毫秒为单位）。必须通过 instrumentation.captureInterFrameTime = true 打开。
* particlesRenderTimeCounter：渲染粒子（包括动画）所花费的时间（以毫秒为单位）。必须通过 instrumentation.captureParticlesRenderTime = true 打开。
* spritesRenderTimeCounter：渲染精灵所花费的时间（以毫秒为单位）。必须通过 instrumentation.captureSpritesRenderTime = true 打开。
* physicsTimeCounter：模拟物理所花费的时间（以毫秒为单位）。必须通过 instrumentation.capturePhysicsTime = true 打开。
* cameraRenderTimeCounter：渲染相机所花费的时间（以毫秒为单位）。必须通过 instrumentation.captureCameraRenderTime = true 打开。

这些计数器在每帧开始时都重置为 0。因此，在 onAfterRender 回调或可观察对象中更容易访问它们。

## Inspector

从 Babylon.js v4.0 开始，您可以使用 Inspector 来分析您的场景或打开/关闭功能或调试工具。

## VR/XR场景

将 Babylon.js 与 WebVR 或 WebXR 结合使用时，启用多视图是一种几乎可以将渲染速度提高一倍的快速方法。

----

## 延申阅读

* [The scene Optimizer](./sceneOptimizer.md)
* [Optimizing With Octrees]
* [MultiViews Part 1]


