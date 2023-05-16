# WebGPU 详解 - 概述

Babylon.js 有一个抽象层，可以帮助实现新型引擎。目前有 4 种类型的引擎：Null、Native、WebGPU 和 Engine（它实现了 WebGL 以及音频等附加组件）。

前 3 个扩展引擎和引擎扩展 ThinEngine，这是引擎的低级（仅图形）API。请注意，中期目标之一是让所有引擎直接扩展 ThinEngine（或特定的“瘦”版本）而不是 Engine 以更好地封装实现。

下面是用于实现 WebGPU 引擎的主要文件/类的图表：

![](https://doc.babylonjs.com/img/extensions/webgpu/classesChart.png)

## 核心类

这些是 WebGPU 实现中使用的核心类。

### WebGPUEngine

此类是 WebGPU 实现的主要入口点。它扩展了 Engine 并实现了更高层使用 WebGPU 所需的所有 API。实现的某些部分在 WebGPU/Extensions/ 文件中分派，其方式与为 WebGL 完成的方式大致相同（在 Engines/Extensions/ for WebGL 中）。

* 我们目前在 WebGPU 中使用与在 WebGL 中使用的相同的 GLSL 着色器。由于 GLSLlang，这些着色器被转换为 SpirV，而 SpirV 通过 Tint 到 WASM (TintWASM) 的端口被转换为 WGSL。
* 我们使用了 3 个 Command Encoder：Upload、RenderTarget 和 Render。当我们需要将数据上传到纹理时使用 Upload，RenderTarget 用于渲染到渲染目标，Render 用于主渲染通道（当渲染到swap chain texture时）。它们按以下顺序被推入队列：Upload -> RenderTarget -> Render。请注意，计算着色器通道也使用 RenderTarget Encoder。此外，在生成渲染目标的 mipmap 时，我们使用 RenderTarget 编码器而不是 Upload，这样可以确保 mips 生成是在渲染到渲染目标之后而不是之前完成的（Upload Encoder 的命令在 RenderTarget Encoder 命令之前被推入队列）
* GPU 计时（可以在 Inspector 的 Statistics / GPU Frame time 下看到）是使用时间戳查询完成的，但目前 Chrome 默认不允许这些查询：Chrome 必须使用 --disable-dawn- 启动features=disallow_unsafe_apis 标志使其工作
* _ubInvertY 和 _ubDontInvertY 是用于在没有 CSS yScale(-1) 技巧的情况下实现渲染的两个uniform buffer（因此在每个帧的末尾保存一个副本）：请参阅此[评论](https://github.com/BabylonJS/Babylon.js/pull/11616#issue-1077008145)以获取更多信息。
* 绑定 RenderTargetWrapper 时（通过调用 bindFramebuffer()），我们不会立即创建渲染通道（此渲染通道是在 _startRenderTargetRenderPass() 中创建的）。这是因为我们可能会在短时间内处理 clear() 调用：我们将在那时创建渲染通道，使用传递给 clear() 的清除值来创建渲染通道。这样做可以避免在调用 bindFramebuffer() 时创建渲染通道，然后在调用 clear() 时关闭它并重新创建一个渲染通道（因为清除是在通过 beginRenderPass API 调用创建渲染通道的同时完成的）。

### WebGPUBufferManager

此类处理与 GPU 缓冲区相关的所有内容（创建、删除、读取、写入）。

* 读取half float数据非常慢，因为必须在js代码中完成从half float到float的转换！为了加快速度，可以改用计算着色器
* 如果行字节大小未对齐（即不能被 256 整除），则 js 中有一个额外的复制过程来构建最终缓冲区，从而使整个过程变慢。计算着色器在这里也有帮助。
* 当调用 releaseBuffer() 时，传入的缓冲区不会立即释放，因为在队列处理期间（即调用 device.queue.submit() 时）使用的缓冲区必须有效。相反，缓冲区被推送到一个列表中，该列表将通过调用 destroyDeferredBuffers() 进行清理（WebGPUEngine 在调用 device.queue.submit() 之后，在帧末尾调用此方法）。

### WebGPUTextureHelper

此类处理与 GPU 纹理相关的所有内容（创建、删除、读取、写入、生成 mipmap 等）。

* 至于 GPU 缓冲区，我们使用延迟列表来释放纹理并等待队列提交后释放它们。
* 创建纹理时，我们总是设置 RenderAttachment | CopyDst 标志，因为我们事先不知道是否会使用 copyExternalImageToTexture() 来更新纹理（请参阅 updateTexture() 方法），并且此函数需要设置这些标志
* mipmap 生成是通过运行 n 个渲染通道（*6 用于立方体纹理）来完成的。使用一些优化的计算着色器可能会更快。此外，请参阅 [Lower perf when generating mipmaps compared to webgl](https://bugs.chromium.org/p/dawn/issues/detail?id=587) 以了解有关 mipmap 生成性能的一些讨论

### WebGPUSnapshotRendering

此类实现快照渲染优化：有关此优化的更多信息，请参阅[快照渲染](../webGPUOptimization/webGPUSnapshotRendering.md)。

它在录制时为每个渲染纹理（渲染目标纹理或交换链纹理）创建一个包，并为所有后续帧重放该包。

请注意，它正在创建一个包列表而不是单个包，因为包内不支持许多 API（如 setViewport、setScissorRect 等）：WebGPUBundleList 是管理与这些 API 调用交错的包列表的类。

### WebGPUTintWASM

此类是 TintWASM 包的轻型包装器，用于将 SpirV 转换为 WGSL。

### WebGPUOcclusionQuery

此类在 WebGPU 中实现遮挡查询。这是一个简单的实现，它使用 WebGPUQuerySet 帮助程序类来处理实际的查询内容。

### WebGPUTimestampQuery

此类通过在上传命令编码器的开头写入时间戳并将其作为渲染命令编码器的最后一个命令并减去两个时间戳的时间度量来实现帧的 GPU 计时。

如上所述，如果使用 --disable-dawn-features=disallow_unsafe_apis 标志启动，它目前只能在 Chrome 中运行。

## 用于缓存的类

为了避免每次需要时都重新创建一些对象并节省性能，WebGPU 实现使用了许多缓存。

### WebGPUCacheBindGroup

此类实现 GPU BindGroup 的缓存。有关实现的详细说明，请参阅 [BindGroup缓存](./webGPUCacheBindGroup.md)。

### WebGPUCacheRenderPipeline

此类实现 GPU RenderPipeline 的缓存。有关实现的详细说明，请参阅 [RenderPipeline缓存](./webGPUCacheRenderPipeline.md)。

### WebGPUCacheSampler

此类实现了 GPU 采样器的缓存。缓存是一个简单的map：

````javascript
export class WebGPUCacheSampler {

    private _samplers: { [hash: number]: GPUSampler } = {};
    ...
}
````

哈希值是根据采样器属性（采样模式、比较函数、包装模式等）计算的。

## 一些专门的 low level 类

WebGPU 对 Babylon.js 框架使用的一些低级类有自己的实现。

### WebGPURenderTargetWrapper

RenderTargetWrapper 的专门实现。在 WebGPU 中，我们只需要一个额外的 _defaultAttachments: number[] 属性，它在创建时保存附件列表（当调用 WebGPUEngine.createMultipleRenderTarget 时）。如果在开始新的渲染目标过程时没有提供明确的附件列表（通过调用 WebGPUEngine.bindAttachments），它将使用该列表。

### WebGPUHardwareTexture

它是 WebGPU 的 HardwareTextureWrapper 的专门实现。这是 InternalTexture 在 _hardwareTexture 属性中持有的对象。

请注意，此类包含一些缓存以提高性能：

* 生成 mipmap 时：_mipmapGenRenderPassDescr 和 _mipmapGenBindGroup 属性
* 调用 WebGPUTextureHelper.invertYPreMultiplyAlpha 时：_copyInvertYTempTexture、_copyInvertYRenderPassDescr、_copyInvertYBindGroup 和 _copyInvertYBindGroupWithOfst 属性

这些属性包含预构建的 WebGPU 对象，因此我们不必在每次调用相应处理（mipmap 生成/调用 invertYPreMultiplyAlpha）时重新创建它们 - 我们仅在第一次调用处理时创建它们。

### WebGPUDepthCullingState

这是 DepthCullingState 的专门实现。此类覆盖所有 setter 方法以调用相应的 WebGPUCacheRenderPipeline 方法，以便深度/剔除缓存状态始终是最新的。

### WebGPUStencilStateComposer

这是 StencilStateComposer 的专门实现。至于WebGPUDepthCullingState，这个类会覆盖所有的setter方法来调用相应的WebGPUCacheRenderPipeline方法，使模板缓存状态始终是最新的。

## 着色器/管线类

这些是处理着色器和着色器上下文的类。请注意，此上下文中的管线与规范中的 WebGPU 管线概念无关！这里指的是着色器上下文。

### WebGPUShaderProcessor, WebGPUShaderProcessorGLSL, WebGPUShaderProcessorWGSL

WebGPUShaderProcessor 是 WebGPUShaderProcessorGLSL 和 WebGPUShaderProcessorWGSL 使用的基类，系统使用它来解析着色器：用 GLSL 编写 WebGPUShaderProcessorGLSL，用 WGSL 编写 WebGPUShaderProcessorWGSL。

这些类的主要任务是收集buffer、uniform、attribute、纹理和采样器的列表，并创建相关的 WebGPU 对象，如绑定组布局条目和绑定组条目。这些对象存储在 WebGPUShaderProcessingContext 实例中。着色器代码也进行了修改，使其符合 glsllang 预期的语法。

这些类还创建了一个特殊的uniform buffer的描述，称为left over buffer，它包含所有使用uniform VarType VarName 声明的uniform变量；着色器中的语法（或uniform VarName : VarType；对于 WGSL 着色器 - 请参阅[在 WGSL 中编写着色器](https://doc.babylonjs.com/setup/support/webGPU/webGPUWGSL#special-syntax-used-in-wgsl-code)）。事实上，您不能在 WebGPU 中的uniform buffer之外声明uniform，因此系统会在后台为您创建一个并自动处理它。

为了避免在帧末尾使用 Y 反转进行额外的纹理复制，这些类在所有着色器中注入了一些特殊代码。此代码使用的参数（yFactor 和 textureOutputHeight）通过特定的uniform buffer传递，在着色器代码中称为 Internals。有关详细信息，请参阅此[评论](https://github.com/BabylonJS/Babylon.js/pull/11616#issue-1077008145)。

### WebGPUShaderProcessingContext

此类包含 WebGPUShaderProcessorXXX 解析着色器时提取/创建的所有数据。

请注意，我们使用已知 UBO 的 _SimplifiedKnownUBOs 定义而不是 _KnownUBOs 来保存一些 GPURenderPassEncoder.setBindGroup 调用：对于 _SimplifiedKnownUBOs，我们只使用两个绑定组，因此只发出两个调用。未来可以进行优化，如果场景 UBO 相同并且其内容自上次调用以来没有更改，我们将不会发出对场景 UBO（绑定组 0）的调用。然而，使用 WebGPU 的预期模式是非兼容模式，在这种模式下，setBindGroup 调用的次数并不重要，因为我们只在创建包时执行一次。

### WebGPUPipelineContext

此类是 Effect 类使用的主要类，它收集与构建该效果的着色器相关的所有数据。主要属性有：

* shaderProcessingContext. 用于解析着色器的 WebGPUShaderProcessingContext 实例
* bindGroupLayouts. 由 WebGPUCacheRenderPipeline 类构建的 GPUBindGroupLayout 对象
* stages. GPU 顶点和片段stage对象
* uniformBuffer. UniformBuffer 实例

再次重申，WebGPUPipelineContext 中的 Pipeline 与 WebGPU Pipeline 概念无关，您应该更像 WebGPUShaderContext（或 WebGPUEffectContext）一样来理解它。

### WebGPUComputePipelineContext

此类是 WebGPUPipelineContext 的镜像，但用于计算着色器而不是常规顶点/片段着色器。它由 ComputeEffect 类使用，并收集与构建效果的计算着色器相关的所有数据（仅限于计算阶段对象 (GPUProgrammableStage) 和计算管道对象 (GPUComputePipeline)）。

## 上下文类

上下文类是将一些数据从用户空间带到核心并在渲染时使用的类。

WebGPUMaterialContext 和 WebGPUDrawContext 由 Engine.enableEffect 调用通过 DrawWrapper 参数传递给核心。这两个类的主要目标是跟踪用于创建BindGroup（采样器、纹理、Uniform/存储缓冲区）的数据。

WebGPUComputeContext 直接传递给 Engine.computeDispatch 方法。

### WebGPUMaterialContext

此类跟踪材质使用的纹理和采样器列表。

纹理可以是内部或外部纹理。内部纹理是常规标准纹理（InternalTexture 类），而外部纹理（ExternalTexture 类）是 WebGPU 中的新纹理，并且（暂时）仅用于视频纹理。由于 WebGPU 处理外部纹理的方式，绑定组缓存不能用于它们（因为外部纹理仅对当前帧有效，您必须通过调用在每一帧检索新纹理（然后重新创建新的绑定组） device.importExternalTexture：请参阅 WebGPUMaterialContext.forceBindGroupCreation getter。

此外，我们必须知道是否至少有一个 32 位浮点纹理（请参阅 WebGPUMaterialContext.hasFloatTextures），因为目前 WebGPU 不支持过滤 32 位浮点纹理，这意味着我们必须在创建绑定组布局条目时设置一些特殊值此纹理：与纹理关联的采样器（如果有）必须设置为非过滤，并且纹理的采样类型必须设置为 unfilterable-float。请参阅 WebGPUCacheRenderPipeline._createPipelineLayoutWithTextureStage。

### WebGPUDrawContext

此类跟踪着色器使用的 Uniform/Storage 缓冲区列表。材质Uniform buffer可以存储在 WebGPUMaterialContext 上，但为了简化实现，我们没有拆分列表，所有缓冲区都由 WebGPUDrawContext 处理。

该类还拥有两个缓存：

* fastBundle - 非兼容模式下使用的bundle
* bindGroups - 它是BindGroup的缓存。如果 WebGPUDrawContext.isDirty==false（和 WebGPUMaterialContext.isDirty==false），它会被重新用于下一次绘制。请参见 WebGPUCacheBindGroup

最后，该类管理一个 GPU 缓冲区 (WebGPUDrawContext.indirectDrawBuffer)，它存储在非兼容模式下使用实例时在（间接）绘制调用中使用的参数。实际上，在那种模式下，绘制调用嵌入到bundle中，如果要绘制的实例数是每帧更新的，我们将需要重新创建bundle以更新实例计数参数。由于创建bundle会导致一些性能损失，因此我们使用间接绘制调用并更新 GPU 缓冲区中的实例计数，而不是在bundle中进行常规绘制。

关于 WebGPUMaterialContext.updateId 属性的注释：

修改材质中的纹理确实适用于所有使用该材质的Mesh，因为当您更新纹理属性时，所有使用该材质的子Mesh都被标记为“纹理脏”。这意味着当 Material.isReadyForSubMesh 运行时，我们将执行调用 subMesh.setEffect(effect, defines, this._materialContext) 的代码，它会重置上下文（参见 DrawWrapper.setEffect 的代码）。因此绑定组将使用新纹理重新生成。

但是，如果我们不使用材质而是使用 effect.setTexture(...) 直接更新纹理（例如深度剥离渲染器正在更新 oitDepthSampler 和 oitFrontColorSampler），它将不再起作用，因为上下文是不重置：没有什么会触发绑定组的重建。我们可以在所有子mesh上手动调用 _markAllSubMeshesAsTexturesDirty() 但它的性能不佳，因为将调用 Material.isReadyForSubMesh 而我们只想绑定一些纹理并且不需要调用此方法。此外，这是一个手动调用，因此可能会被遗忘......

为了解决这个问题，我们添加了一个 WebGPUMaterialContext.updateId 属性，每次在材质上下文中更改采样器/纹理时都会更新该属性。 WebGPUDrawContext 也更新了 WebGPUDrawContext.materialContextUpdateId 属性，该属性对应于生成绑定组时关联的 WebGPUMaterialContext.updateId 值（您可以在 WebGPUMaterialContext.bindGroups 中找到）。从 WebGPUMaterialContext.bindGroups 缓存中检索绑定组时，如果两个 updateId 值不同，则表示材质上下文自上次生成绑定组以来已发生更改，因此我们必须重新创建它们。

### WebGPUComputeContext

此类或多或少是 WebGPUDrawContext 的镜像，但用于计算着色器。它的任务是根据着色器使用的资源创建绑定组。绑定组会被缓存，直到至少有一个资源被更新：ComputeShader 类负责在资源更改时清除绑定组。

## 辅助类

WebGPU 实现中使用了许多辅助类。下面列出了主要的。

### WebGPUBundleList

此类用于管理Bundle列表。更准确地说，它处理一个item列表。一个item可以是：

* 包列表
* 视口设置
* scissor矩形范围设置
* 模板参考设置
* 混色设置
* 开始遮挡查询
* 结束遮挡查询

这是因为 bundle 本身不能包含许多 API 调用，如 setViewport、setScissorRect、setStencilReference 等：请参阅 LibDeclarations/webgpu.d.ts 中的 GPURenderBundleEncoder 类。

为了使事情更清楚，我们必须为其创建单独项目的 API 是 GPURenderPassEncoder 支持但 GPURenderBundleEncoder 不支持的 API：

* setViewport
* ssetScissorRect
* ssetStencilReference
* ssetBlendConstant
* sbeginOcclusionQuery
* sendOcclusionQuery

在帧期间记录 API 调用时（例如快照渲染所使用的），我们在 bundle item 列表中添加尽可能多的 bundle，但是当我们必须处理bundle encoder接口不支持的 API 调用时我们必须根据我们必须使用的 API 调用创建一个新item并将其添加到item列表中。在帧的末尾，通过调用 WebGPUBundleList.run() 将 bundle/API 调用发送到 GPU。

### WebGPUClearQuad

此类处理清除纹理中的矩形区域。当调用 WebGPUEngine.clear() 并且非全屏剪刀矩形生效时使用它。

它有自己的渲染管线缓存实例，以避免干扰主渲染管线缓存。实际上，WebGPUClearQuad 需要禁用深度测试并将模板读取掩码设置为 0xFF。如果我们将这些状态设置在主渲染管道缓存上，它会/可能会导致这些状态发生变化（通常启用深度测试），因此强制以比我们使用更低索引状态的索引状态开始缓存遍历单独的缓存实例（请参阅渲染管道缓存如何工作 [Cache Render Pipeline](https://doc.babylonjs.com/setup/support/webGPU/webGPUInternals/webGPUCacheRenderPipeline) ）。不确定这是一个很大的性能提升（因为 WebGPUClearQuad.clear() 很少被调用），但它仍然完成了......

### WebGPURenderPassWrapper

这个类是一个非常小的包装器，围绕着 GPU 渲染通道使用的主要对象：渲染通道描述符、渲染通道本身、颜色/深度描述符、输出纹理和深度纹理格式。主通道有一个实例，渲染目标通道有另一个实例，并允许分解主/渲染目标通道常用的一些方法（如 _setColorFormat 和 _setDepthTextureFormat）。

### WebGPUQuerySet

这是实现 GPU 查询集的类，由 WebGPUOcclusionQuery 和 WebGPUTimestampQuery 使用。请参阅 WebGPU 规范中的[Queries](https://www.w3.org/TR/webgpu/#queries)。