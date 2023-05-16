# WebGPU 详解 - 杂项

## 验证测试

这些验证测试被排除在 WebGPU 之外，因为某些功能尚未在 Babylon.js 中实现：

* GLTF Mesh Primitive Mode (0) 和 GLTF Mesh Primitive Mode (1): line loop / triangle fan 尚未实现
* GLTF Buggy with Meshopt Compression：尚不支持顶点缓冲区的浮点格式以外的格式（位置，法线，uv，...）

实现这些功能后，应重新启用相应的验证测试。

自阴影验证测试会产生渲染错误（但仍然可以，因为错误率低于 2.5%），因为它使用指数阴影贴图，其参数（尤其是 depthScale）取决于深度贴图的精度。在 WebGL 中，我们使用 32 位浮点纹理，但在 WebGPU 中，它只是半浮点纹理，因为不支持 32 位浮点纹理的线性过滤（至少目前是这样）：

![](https://doc.babylonjs.com/img/extensions/webgpu/webgpuValidationTestSelfShadowing.jpg)

核心开发人员请注意：您应该经常在本地运行验证测试，因为它们不在 AZURE 服务器上运行！

运行：

* WebGPU tests: http://localhost:1338/tests/validation/?list=webgpu&engine=webgpu
* Standard tests: http://localhost:1338/tests/validation/?list=config&engine=webgpu

您还应该以“check resource creation”模式运行它们：请参阅 [检查资源创建 - 仅限 WebGPU](https://doc.babylonjs.com/contribute/toBabylon/validationTests#check-resource-creation---webgpu-only)。

## TintWASM

您应该定期更新 TWGSL (Tint WASM) 模块，以便它与 Tint 源代码保持同步。

* TintWASM module: https://github.com/syntheticmagus/twgsl
* Tint repository: https://dawn.googlesource.com/tint