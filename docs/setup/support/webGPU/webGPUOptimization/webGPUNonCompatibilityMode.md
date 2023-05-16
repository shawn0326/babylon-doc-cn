# WebGPU 非兼容模式

WebGPU 可以在非兼容模式（NCM - 也称为快速路径）下运行，这可以根据您的场景提供更好的性能。

WebGPUEngine.compatibilityMode 属性是一个开关，默认情况下为 true。在此模式下，WebGPU 引擎与 WebGL 处于完全兼容模式（CM），这意味着所有可以使用 WebGL 渲染的场景也可以由 WebGPU 渲染，而无需您进行任何更改。

将此属性设置为 false 时，您会切换到非兼容模式 (NCM)，从而允许引擎使用特殊的代码路径，这可能会以重新组织代码为代价来提高性能。

## 描述

此模式的作用是构建一个名为 bundle 的对象，该对象包含着在给定上下文中绘制Mesh（更准确地说是子Mesh）的命令，然后每次在此上下文中绘制Mesh时可以重用该 bundle。构建这个 bundle 是有成本的，所以 bundle 必须有足够长的生命周期来补偿这个成本：如果在每一帧上重新创建 bundle，你实际上会因为使用 NCM 而遭受性能损失！

在许多情况下必须重新创建一个包：比如你改变了材质，或者直接在引擎上改变了一些全局上下文，更新了一些顶点数据等等。通过查看 engine.countersLastFrame 属性，你可以知道有多少包被重新创建以及有多少被重用于最后一帧：

````
> engine.countersLastFrame
> {numEnableEffects: 0, numEnableDrawWrapper: 2, numBundleCreationNonCompatMode: 0, numBundleReuseNonCompatMode: 2}
````

您感兴趣的是 numBundleXXX 这两个计数器：

* numBundleReuseNonCompatMode 是在最后一帧中被重用的包的数量
* numBundleCreationNonCompatMode 是在最后一帧中创建的包数

numBundleCreationNonCompatMode 越低越好：最好的情况是该计数器为 0。当然，根据您的操作，不可能使该计数器始终等于 0，但您应该尝试将其设置得尽可能低。

注意：如果两个计数器都为 0，则表示您不在 NCM 中，您应该执行 engine.compatibilityMode = false 以切换到 NCM。

请参阅本页的最后一部分，了解可能导致捆绑（重新）创建的注意事项以及如何修改代码以避免这种情况。

## 注意事项

在 NCM 模式下，从理论上讲，某些事情可能无法按预期工作（例如，更改材质不起作用），在这种情况下调用 scene.resetDrawCache() 将解决问题（或者，如果问题只影响单个Mesh，可以使用mesh.resetDrawCache()）。

然而，这在实践中应该很少发生（见下一节）：如果发生，请向论坛报告，以便我们分析案例并查看是否可以使其自动工作而无需调用 resetDrawCache。

此外，即使您能够实现每帧低（甚至 0）的捆绑重新创建，您也可能看不到与兼容模式相比的性能改进。这将取决于您的场景，您应该进行一些基准测试，看看 NCM 是否为您改进了一些东西。

## 在非兼容模式 (NCM) 下做/不做

* 如果两个或多个相机正在渲染到相同的 RTT（使用 Camera.outputRenderTarget 属性），请设置 rtt.renderPassId = undefined 以便 camera.renderPassId 用于每个相机而不是 rtt.renderPassId 用于两个相机，从而导致捆绑重新创建。
* 如果您正在使用骨架，请确保一种材质不会链接到多个不同的骨架，这意味着一种材质不会被多个不使用相同骨架的Mesh体使用。在这种情况下，根据需要多次克隆材质。morph targets也一样。
* Material.needDepthPrePass 在 NCM 中确实有效，但总是会在每一帧创建新的包。
* 如果在切换到 NCM 后更新这些材质属性之一，则必须重置绘制缓存以使这些更改生效（通过调用 mesh.resetDrawCache() 或 scene.resetDrawCache()）：sideOrientation、disableDepthWrite、forceDepthWrite、depthFunction , disableColorWrite, zOffset, 模板
* 如果您更新后处理/RTT 的示例属性，则必须在之后调用 scene.resetDrawCache() 以避免使用新值进行渲染（但使用旧示例值创建的旧包），否则您将收到类似的错误renderBundles[0] ([RenderBundle]) 的附件状态与 [RenderPassEncoder] 的附件状态不兼容。
* 不在 NCM 中工作（意思是：如果您想使用它们，请不要切换到 NCM，否则您的程序将无法按预期工作）：
    * 遮挡查询。它们不起作用，因为必须保留绘制和查询的顺序，这在 NCM 中是不可能的，因为在调用 executeBundles() 时所有绘制都被推迟到帧的末尾，而查询在它们被调用时执行。
    * Material.separateCullingPass = true 由于当前的实施方式而不起作用。

## 案例分析

下面是我们验证测试套件中所有 playground 的列表，当我们第一次在非兼容模式下测试它们时，它们每帧重新创建一个或多个包。我们解释了他们为什么要创建捆绑包以及我们如何修复它们（在可能的情况下）。您可以通过将 PG 编号（在 PG 名称后的括号内给出）附加到 Playground url (https://playground.babylonjs.com/) 来查看固定代码。

### WebGPU 列表 (webgpu.json)

* 带有自定义着色器的球体使用发光层显示线框(#Y05E2C#6)
    * 原因：材质的属性在一个帧中更改两次，以根据绘制Mesh的位置（发光层或常规渲染）更改颜色
    * 解决方案：使用两种材质，每种情况一种
* 粒子系统矩阵，如(#WL44T7)
    * 原因：重新创建了一个新的粒子系统来替换已经达到生存时间的粒子系统
    * 解决方案：无。作为 PG 正常工作的一部分，新的粒子系统会定期重新创建，因此预计束也会定期创建

### 默认列表 (config.json)

* Nested BBG, Chibi Rex, Yeti, Bones, GLTF Serializer Morph Target Animation Group（#ZG0C8B#5、#QATUCH#18、#QATUCH#19、#7EC27T#3、#T087A8#29）
    * 原因：对使用不同骨架的不同Mesh使用相同的材​​质
    * 解决方案：使用与骨架数量一样多的材质
* 固体粒子系统（#WCDZS#92）
    * 原因：SPS Mesh是可挑选的。当Mesh可拾取时，顶点属性（位置、法线）通过调用更新，mesh.updateVerticesData这会弄脏材质并最终重新创建快速路径代码使用的束
    * 解决方案：将Mesh设置为不可拾取
* 色带变形（#ACKC2#1）
    * 为什么：每帧更新位置属性
    * 解决方案：当时没有，CreateTube（和所有功能MeshBuilder）正在使用getVerticesData/updateVerticesData更新现有实例，后一种方法触发markAsAttributeDirty对材质的调用。我们需要在不使用 的情况下直接更新 GPU 缓冲区updateVerticesData，但这样做会丢失Mesh生成器方法工作所必需的烘焙 CPU 数组（该方法Buffer.updateDirectly正在清除_data属性，但getVerticesData仅当此属性不是时才有效null）
* 自定义渲染目标(#TQCEBF#3)
    * 为什么：使用RenderTargetTexture时，在onBeforeRender中更改使用的材质并在onAfterRender中将其重置
    * 解决方案：使用RenderTargetTexture.setMaterialForRendering代替onBeforeRender/onAfterRender
* 高级阴影、高级阴影（右手）、反向深度缓冲区和阴影（#SLV8LW#3、#B48X7G#64、#WL4Q8J#20）
    * 原因：8个楼层使用相同的材质（对于反向深度缓冲区和阴影，它是重复使用相同材质的盒子/球体/结）。每个楼层 + 盒子都由一个特定的灯点亮，该灯具有自己的着色器生成器。绘制地板时，阴影生成器对应的阴影采样器将绑定到着色器，并且由于所有地板都使用相同的材​​质，因此设置新的阴影采样器会重置缓存
    * 解决方案：为每层楼使用不同的（克隆的）材质
* 运动模糊，实例 + GBR + 运动模糊（#E5YGEL#20，#YB006J#403）
    * 为什么：每帧可见的球体数量（实例中的树 + GBR + 运动模糊）永远不会相同，因为它们在移动。为每个球体存储运动模糊参数的统一LeftOver缓冲区在内部有许多 GPU 缓冲区，每个球体一个。创建快速路径束时，一个缓冲区与该束相关联（显示的第一个球体的缓冲区 #0，第二个球体的缓冲区 #1，依此类推）。但是，下一帧，将用于给定球体的 GPU 缓冲区可能会有所不同，因为缓冲区从第一个开始重复使用：第一个缓冲区用于显示的第一个球体，第二个缓冲区用于第二个球体显示等。因此，如果球体未以相同顺序显示，则将重新创建包
    * 解决办法：暂无。对于 PG，我们简单地设置alwaysSelectAsActiveMesh=true，使系统处理的球体数量不变
* 瘦实例 + 运动模糊 + 手动(#HJGC2G#132)
    * 为什么：thinInstanceSetBuffer 每帧都被调用，这会（重新）创建一个顶点缓冲区，这反过来又将材质标“脏”
    * 解决方案：使用thinInstanceBufferUpdated代替
* 多相机和输出渲染目标(#BCYE7J#31)
    * 为什么：cameraRTT1并且cameraRTT2正在渲染到相同的 RTT（使用它们的outputRenderTarget属性），但是在 RTT 中渲染时，使用的渲染通道RTT.renderPassIdIDrenderPassId是（因为场景 ubo 在两种情况下都不同，因为它存储相机视图/投影）
    * 解决方案：删除RTT.renderPassID属性 ( RTT.renderPassID = undefined) => 在这种情况下，camera.renderPassId将使用该值代替
* OIT（#1PLV5Z#104）
    * 为什么：没在深度剥离渲染器上使用useRenderPasses = true
    * 解决方案：设置scene.depthPeelingRenderer.useRenderPasses = true;