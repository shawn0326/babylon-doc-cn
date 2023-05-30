# 计算着色器

以下是维基百科对计算着色器的描述：

    在计算中，计算内核是为高吞吐量加速器（例如图形处理单元 (GPU)、数字信号处理器 (DSP) 或现场可编程门阵列 (FPGA)）编译的例程，与主程序分开但由主程序使用（通常在中央处理器上运行）。它们有时被称为计算着色器，与 GPU 上的顶点着色器和像素着色器共享执行单元，但不限于在一类设备或图形 API 上执行。

请注意，在 Babylon.js 中，这只是一个 WebGPU 功能（从 v5.0 开始），WebGL 不支持计算着色器。

您可以通过以下方式查询引擎的计算着色器支持：

````javascript
engine.getCaps().supportComputeShaders
````

## 创建计算着色器

使用 ComputeShader 类创建计算机着色器，就像您使用 ShaderMaterial 为某些自定义着色器代码创建材质一样：

````javascript
const cs1 = new BABYLON.ComputeShader("myCompute", engine, { computeSource: copyTextureComputeShader }, { bindingsMapping:
    {
        "dest": { group: 0, binding: 0 },
        "src": { group: 0, binding: 2 }
    }
});
````

copyTextureComputeShader 是着色器代码，而 bindingsMapping 是一个将输入变量名称（来自着色器源代码）映射到其绑定位置的对象：请参阅下文了解更多说明。

创建计算着色器实例后，您可以使用适当的方法设置输入值（请参阅 ComputeShader 类）：

````javascript
cs1.setTexture("src", src);
cs1.setStorageTexture("dest", dest);
````

可以通过调用 dispatch() 或 dispatchWhenReady() 方法之一来执行计算着色器。

## 传递给计算着色器的变量类型

传递给计算着色器的变量可以是以下类型：

* texture. 使用 ComputeShader.setTexture() 将常规纹理传递给着色器。
* storage texture. 存储纹理是可以从计算着色器写入的纹理。请注意，您不能从存储纹理中读取，只能写入它。如果您在存储纹理中写了一些东西并需要在着色器中读取它，只需将其作为常规纹理传递即可。在 Babylon.js 中，存储纹理被创建为常规纹理，但为 creationFlag 参数传递了一个特殊标志：BABYLON.Constants.TEXTURE_CREATIONFLAG_STORAGE。还有两个辅助方法可用于创建存储纹理：BABYLON.RawTexture.CreateRGBAStorageTexture() 和 BABYLON.RawTexture.CreateRStorageTexture()。使用 ComputeShader.setStorageTexture() 将存储纹理传递给着色器。
* uniform buffer. 它是一个缓冲区，您可以通过实例化 UniformBuffer 类来创建，并且可以用来将一些常量值传递给着色器端（这些值仍然可以在您的程序过程中更新 - 它们是着色器内部的常量）。请注意，您需要首先按照属性在着色器代码中出现在缓冲区中的顺序调用 addUniform 方法来创建此缓冲区的布局！最后一点很重要，因为您创建的布局必须与着色器代码中使用的缓冲区布局相匹配。创建布局后，您可以通过调用 updateXXX() 方法来设置一些值。一旦您准备好更新 GPU 端的缓冲区，请调用 update()。使用 ComputeShader.setUniformBuffer() 将统一缓冲区传递给着色器。
* storage buffer. 这是一个可用于读取或写入值的任意缓冲区。使用 StorageBuffer 类创建此类缓冲区。使用 ComputeShader.setStorageBuffer() 将存储缓冲区传递给着色器。
* sampler. 这是一个自定义纹理采样器对象，可用于对纹理进行采样。使用 TextureSampler 类创建一个并使用 ComputeShader.setTexturesSampler() 将纹理采样器传递给着色器。有关纹理采样器的更多说明，请参见下一节。

## 着色器语言和输入绑定

计算着色器必须用 WGSL 编写，这是 WebGPU 使用的着色器语言。

由于 GLSL 着色器可以存储在 ShaderStore.ShadersStore 中，WGSL 着色器可以存储在 ShaderStore.ShadersStoreWGSL 中，您可以将用于存储此对象中着色器的键的名称传递给 ComputeShader 构造函数。您也可以直接将着色器代码传递给构造函数（如上例所示）。

浏览器目前不支持 WGSL 着色器的反射，这意味着我们无法自动检索输入变量的绑定和组值，如下所示：

````
@group(0) @binding(0) var dest : texture_storage_2d<rgba8unorm, write>;
@group(0) @binding(1) var srcSampler : sampler;
@group(0) @binding(2) var src : texture_2d<f32>;
````

这就是为什么您需要在创建新的 ComputeShader 实例时自己提供这些绑定：

````javascript
const cs1 = new BABYLON.ComputeShader("myCompute", engine, { computeSource: copyTextureComputeShader }, { bindingsMapping:
    {
        "dest": { group: 0, binding: 0 },
        "src": { group: 0, binding: 2 }
    }
});
````

请注意，对于上例中作为 src 的（采样的）纹理变量，您可以指示系统自动绑定与纹理对应的采样器。为此，您必须使用等于纹理绑定值减 1 的绑定值声明采样器，并且您应该将 true 作为 setTexture() 调用的第三个参数传递（或者不传递任何内容，因为 true 是默认值）。在这种情况下，您不得在 bindingsMapping 对象中添加此采样器。

如果您不需要/不想自动绑定与纹理对应的采样器，您可以通过将 false 作为第三个参数传递给 ComputeShader.setTexture() 来指示系统不绑定采样器。

您还可以通过创建一个采样器（请参阅 TextureSampler 类）并使用 ComputeShader.setTextureSampler() 方法来绑定您自己的采样器：

````javascript
const cs1 = new BABYLON.ComputeShader("myCompute", engine, { computeSource: copyTextureComputeShader }, { bindingsMapping:
    {
        "dest": { group: 0, binding: 0 },
        "srcSampler": { group: 0, binding: 1 },
        "src": { group: 0, binding: 2 }
    }
});
const sampler = new BABYLON.TextureSampler().setParameters();
cs1.setTextureSampler("samplerSrc", sampler);
````

在这种情况下，您必须将此采样器添加到 bindingsMapping 对象中。

## 示例

### 简单的计算着色器

[简单的计算着色器](https://playground.babylonjs.com/?webgpu#3URR7V#183)

这个 PlayGround 创建了 3 个计算着色器：

* 第一个是加载纹理并通过计算着色器将其复制到另一个纹理中。然后将此纹理应用于地面。警告：它仅用于演示目的，请勿在实际代码中使用此计算着色器，一个简单的复制缓冲区就足以实现相同的目的！

* 第二个是清除具有恒定值的纹理并将其应用于球体。再次强调，不要在实际代码中使用它！请注意，我们不使用 addUniform 来创建Uniform缓冲区的布局：当Uniform缓冲区中只有一个属性时，调用 updateXXX() 来设置属性的值也会创建布局。

* 第三个是计算两个矩阵的乘法。它使用 3 个存储缓冲区，2 个用于两个输入矩阵，1 个用于结果。然后读取结果缓冲区并将其转储到控制台日志。它是 Web 上 GPU 计算入门的直接端口。

### 图片模糊

[模糊计算着色器](https://playground.babylonjs.com/?webgpu#3URR7V#185)

这是 WebGPU 示例 imageBlur 的直接实现。

请注意，在示例中，我们为第一个计算着色器调用 dispatchWhenReady() 以确保在继续之前计算效果已准备就绪，但对于下一个着色器，我们只需调用 dispatch() 因为它们使用与我们相同的着色器代码确定效果会准备好（因为它是同一个）。

### 机器人模拟

[机器人模拟着色器](https://playground.babylonjs.com/?webgpu#3URR7V#186)

这是 WebGPU 示例 computeBoids 的直接实现。

它是一个计算着色器，可以更新两个存储粒子数据的乒乓缓冲区。该数据用于绘制实例粒子。

请注意，它正在使用（5.0 中的新功能）Mesh.forcedInstanceCount 属性为没有实例 (InstancedMesh) 的网格设置实例计数，但我们希望渲染多次，因为我们手动提供了适当的顶点缓冲区。

由于我们用来计算粒子位置和速度的存储缓冲区将用作（实例化的）顶点缓冲区，因此我们必须在创建时将它们标记为 BUFFER_CREATIONFLAG_VERTEX（请参阅代码中的新 BABYLON.StorageBuffer(...) 调用）。

### 水力侵蚀

[水力侵蚀](https://playground.babylonjs.com/?webgpu#C90R62#16)

这是伟大项目 [Hydraulic-Erosion](https://github.com/SebLague/Hydraulic-Erosion) 的一个实现：所有功劳归于 sebastlague@gmail.com！

地形的生成和侵蚀的模拟是通过使用两个不同的计算着色器完成的。

请注意，此示例也适用于计算着色器不可用的 WebGL2，但在设置参数时应小心：不要提高太多迭代次数、半径、最大寿命、分辨率，否则你可能会卡住你的浏览器，因为现在地形生成和腐蚀过程在 CPU 端处理！

### 史莱姆模拟

[史莱姆模拟](https://playground.babylonjs.com/?webgpu#GXJ3FZ#51)

这是伟大项目 [Slime-Simulation](https://github.com/SebLague/Slime-Simulation) 的一个实现：所有功劳归于 sebastlague@gmail.com！

WGSL 中的实现不如 HLSL 漂亮，因为在撰写本文时 WebGPU 不支持读/写纹理，因此我们不得不为 TrailMap 纹理使用存储缓冲区。这意味着我们需要一些复制缓冲区到纹理和纹理到缓冲函数，并且当我们需要获取 vec4（参见代码）时，我们必须从 TrailMap 读取 4 次而不是一次，这可能比它的 HLSL 副本性能低.

### 海洋示例

[海洋示例](https://playground.babylonjs.com/?webgpu#YX6IB8#152)

这是伟大项目 [FFT-Ocean](https://github.com/gasgiant/FFT-Ocean) 的一个实现：所有功劳都归功于 Ivan Pensionerov (https://github.com/gasgiant)！

此示例使用大量计算着色器运行：每帧大约有 200-250 个计算着色器运行！使用 F8 显示/隐藏 GUI（在您单击渲染区域中的任意位置以将焦点置于画布之后）并使用 WASD 移动。

---

## 延申阅读

* [Compute Shader in OpenGL](https://www.khronos.org/opengl/wiki/Compute_Shader)
* [Compute Shader in DirectX](https://docs.microsoft.com/en-gb/windows/win32/direct3d11/direct3d-11-advanced-stages-compute-shader?redirectedfrom=MSDN)
* [Example of compute shader optimization](https://on-demand.gputechconf.com/gtc/2010/presentations/S12312-DirectCompute-Pre-Conference-Tutorial.pdf)