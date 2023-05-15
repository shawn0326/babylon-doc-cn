# WebGPU 重大变化

此页面描述了 WebGL 和 WebGPU 之间的重大变化和行为差异。

我们尽量避免在核心库中进行重大更改，但有时我们无法避免它们，并且随着支持 WebGPU 作为新引擎的重大新发展，如果尝试，您需要注意很多事情将现有项目移植到 WebGPU。

## readPixels 现在是异步的

WebGPU 和 WebGL 之间最大的变化可能是从纹理读取在 WebGPU 中是异步的，没有办法绕过它。因此，所有从纹理读取像素的方法（即使在 WebGL 模式下）现在都返回一个Promise：

* BaseTexture.readPixels
* ProceduralTexture.getContent
* ThinEngine.readPixels
* CubeMapToSphericalPolynomialTools.
* ConvertCubeMapTextureToSphericalPolynomial
* CopyTools.GenerateBase64StringFromTexture

为了匹配 WebGL 的工作方式，在读取纹理之前会自动执行 flushFramebuffer 调用，以确保您获得最新数据。但是，如果您在调用 readPixels 方法时知道纹理是最新的，则可以通过将适当的参数传递给函数调用（flushRenderer = false，请参阅文档）来避免这种刷新（节省一些性能）。请注意，如果您正在 engine.onEndFrameObservable 中进行读取，则不需要刷新，因为此观察者会在当前帧的刷新完成后触发。

另请注意，如果宽度不能被 64 整除，则当前 readPixels 速度很慢！此外，从半浮点纹理读取数据时速度非常慢：如果可能，请改用全浮点纹理。加快这些事情在我们的路线图上。

## WebGPU 引擎的创建是异步的

在 WebGPU 中创建引擎也是异步的。如果浏览器支持，您可以像这样创建一个 WebGPU 引擎，或者一个 WebGL 引擎：

````javascript
async function createEngine() {
  const webGPUSupported = await BABYLON.WebGPUEngine.IsSupportedAsync;
  if (webGPUSupported) {
    const engine = new BABYLON.WebGPUEngine(document.getElementById("renderCanvas"));
    await engine.initAsync();
    return engine;
  }
  return new BABYLON.Engine(document.getElementById("renderCanvas"), true);
}
````

或者使用 EngineFactory 助手（如果支持，它将首先尝试创建 WebGPU 引擎，然后是 WebGL 引擎，然后是空引擎）：

````javascript
async function createEngine() {
  return BABYLON.EngineFactory.CreateAsync(document.getElementById("renderCanvas"));
}
````

## 着色器代码差异

### 纹理数组

着色器代码中的纹理数组不能使用可变索引访问，它必须是立即值。例如，myTextures[0] / myTextures[1] 确实有效，但 myTextures[i] 无效（例如，我是一个变量循环）。

### 将采样器传递给函数

在着色器中，您不能将采样器传递给函数：

````
vec4 getPixel(sampler2D sampler, vec2 uv) {
    return texture2D(sampler, uv);
}
````

在 WebGPU 中调用此函数将失败并出现编译错误。

为了简化现有代码的移植，我们添加了一个预传递着色器代码内联器，它将用函数本身的代码替换函数调用。您需要使用 #define inline 标记要内联的函数，以便内联器执行其处理：

````
#define inline
vec4 getPixel(sampler2D sampler, vec2 uv) {
    return texture2D(sampler, uv);
}
````

### 将值绑定到采样器

WebGPU 不如 WebGL 宽容，所有在着色器中声明的采样器变量都必须有一个值绑定，即使您不使用该变量，这与 WebGL 相反。如果您收到类似“numBindings 不匹配”的警告消息，这意味着您可能在着色器代码中定义了一个Uniform采样器变量，但没有将值绑定到它（通过调用 setTexture("myvar", texture)着色器/材质）。

### ShaderMaterial

如果在 ShaderMaterial（或 CustomMaterial / PBRCustomMaterial）中使用自定义属性，则必须在该着色器使用的属性列表中声明它。例如，对于 ShaderMaterial，您必须在传递给构造函数的选项的属性数组中传递它的名称。在 WebGL 中，你可以省略这个声明，它仍然可以工作（但作为副作用，它并不真正被支持）。

在 WebGL 中，您可以在创建 ShaderMaterial 时多次列出相同的属性并且它会起作用（就好像您只给这个属性一次），但在 WebGPU 中它会失败。

## 其它

与 WebGL 相反，视口不能扩展到帧缓冲区/纹理之外。所以，如果你调用类似的东西：

````javascript
new BABYLON.Viewport(x, y, w, h);
````

* x, y, w, h must be >= 0 and <= 1.
* x + w must be <= 1
* y + h must be <= 1

与 WebGL 相反，顶点缓冲区中属性的步幅不能小于 4 个字节。另外，position vertex kind暂时必须是number/float，不能是char/int (Int8Array/Int32Array)。试试这个 PG，它在 WebGL 中有效，但在 WebGPU 中无效：

[Stride of 3 bytes](https://playground.babylonjs.com/#U1CZV3#4)

WebGPU 不支持 TEXTUREFORMAT_LUMINANCE 格式。