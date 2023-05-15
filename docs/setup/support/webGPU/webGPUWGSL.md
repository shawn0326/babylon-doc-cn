# 使用 WGSL 为 WebGPU 编写着色器

目前，Babylon.js 使用的所有着色器都是用 GLSL 编写的，并通过一些特殊工具转换为 WGSL（WebGPU 唯一知道的着色器语言）。

因此，即使在 WebGPU 中，如果您使用 CustomMaterial 或 PBRCustomMaterial 注入一些自定义着色器代码，您也必须使用 GLSL 编写它。

如果你想用 WGSL 语言编写着色器代码，你可以编写计算着色器或使用 ShaderMaterial 类来包装顶点/片段着色器。后者是本页的主题。

## 使用ShaderMaterial编写WGSL代码

您可以使用 ShaderMaterial 类编写 WGSL 代码，其方式与使用它编写 GLSL 的方式大致相同，但有一些细微差别。

注意：如果您在着色器代码中使用“color”attribute，请不要将其添加到传递给 ShaderMaterial 构造函数的 attributes 属性中！如果名为“color”的顶点缓冲区附加到网格，将自动添加此属性。如果将“color”添加到attribute数组，您将收到类似“Attribute shader location (1) is used more than once”的错误。

### 设置正确的着色器语言

您必须在传递给构造函数的选项参数中将 shaderLanguage 属性设置为 BABYLON.ShaderLanguage.WGSL。例如：

````javascript
const mat = new BABYLON.ShaderMaterial("shader", scene, {
        vertex: "myShader",
        fragment: "myShader",
    },
    {
        attributes: ["position", "uv", "normal"],
        uniformBuffers: ["Scene", "Mesh"],
        shaderLanguage: BABYLON.ShaderLanguage.WGSL,
    }
);
````

### 使用 WGSL 着色器存储

如果使用着色器存储，则必须将 WGSL 代码放入 BABYLON.ShaderStore.ShadersStoreWGSL 而不是 BABYLON.ShaderStore.ShadersStore。

例如：

````javascript
BABYLON.ShaderStore.ShadersStoreWGSL["myShaderVertexShader"]=`   
    #include<sceneUboDeclaration>
    #include<meshUboDeclaration>
    ...
`;

BABYLON.ShaderStore.ShadersStoreWGSL["myShaderFragmentShader"]=`
    varying vPositionW : vec3<f32>;
    varying vUV : vec2<f32>;
    ...
`;
````

### 入口声明

您还必须以特殊方式声明顶点和片段着色器的入口点。

Vertex:

````
@vertex
fn main(input : VertexInputs) -> FragmentInputs {
    ...
}
````

Fragment:

````
@fragment
fn main(input : FragmentInputs) -> FragmentOutputs {
    ...
}
````

### 使用预定义的 Uniforms

要使用场景（view、viewProjection、projection、vEyePosition）和网格（world、visibility）的预定义制服，您必须在着色器代码中包含适当的文件：

````
#include<sceneUboDeclaration>
#include<meshUboDeclaration>
````

并将uniform缓冲区名称添加到 ShaderMaterial 类构造函数的 uniformBuffers 选项：

````
const mat = new BABYLON.ShaderMaterial("shader", scene, {
        vertex: "myShader",
        fragment: "myShader",
    },
    {
        attributes: ["position", "uv", "normal"],
        uniformBuffers: ["Scene", "Mesh"],
        shaderLanguage: BABYLON.ShaderLanguage.WGSL,
    }
);
````

在 WGSL 代码中，您可以通过在名称前加上`scene.`或`mesh.`前缀来访问对应的uniform。分别用于scene或mesh uniforms：

````
@vertex
fn main(input : VertexInputs) -> FragmentInputs {
    vertexOutputs.position = scene.viewProjection * mesh.world * vec4<f32>(vertexInputs.position, 1.0);
}
````

## WGSL 代码中使用的特殊语法

与使用普通 WGSL 代码的计算着色器不同，您为 ShaderMaterial 编写的着色器代码必须使用特殊语法才能与现有工作流程一起使用。为了方便开发人员，变量的声明与 GLSL 中使用的相同：

* 声明一个varying变量

````
varying varName : varType;
````

* 声明一个attribute变量

````
attribute varName : varType;
````

* 声明一个uniform变量

````
uniform varName : varType;
````

与 GLSL 相反，顶点/片段着色器的输入和输出在内部没有声明为单独的全局变量，而是在引擎为您管理的一些结构中定义。然而，这意味着您需要一种特殊的语法来访问这些变量。这是 GLSL 语法和 WGSL 语法之间的映射：

* 在顶点着色器
    * attribute 必须通过 vertexInputs.attributeName 引用
    * gl_VertexID => vertexInputs.vertexIndex
    * gl_InstanceID => vertexInputs.instanceIndex
    * varying 必须通过 vertexOutputs.varName 引用
    * gl_Position => vertexOutputs.position
* 在片元着色器
    * varying 必须通过 fragmentInputs.varName 引用
    * gl_FragCoord => fragmentInputs.position
    * gl_FrontFacing => fragmentInputs.frontFacing
    * gl_FragColor => fragmentOutputs.color
    * gl_FragDepth => fragmentOutputs.fragDepth

注意：
* 使用 uniform varName : varType 语法时，您可以通过执行 uniforms.varName 而不是简单的 varName 来访问变量。与 GLSL 一样，可以使用 ShaderMaterial 类的常规方法（setFloat、setInt 等）从 javascript 代码中设置以这种方式声明的变量
* varType 必须使用 WGSL 语法，而不是 GLSL！例如：变化的 vUV : vec2<f32>;
* 你不能添加 @group(X) @binding(Y) 装饰！系统会自动添加

## 使用 WGSL 中可用的新对象

您可以使用标准 WGSL 语法来声明：

* 自定义uniform buffer：
````
struct MyUBO {
    scale: f32,
};

var<uniform> myUBO: MyUBO;
````
* storage 纹理：
````
var storageTexture : texture_storage_2d<rgba8unorm,write>;
````
* storage buffers：
````
struct Buffer {
    items: array<f32>,
};
var<storage,read_write> storageBuffer : Buffer;
````
* external 纹理：
````
var videoTexture : texture_external;
````

同样，您不能添加 @group(X) @binding(Y) 装饰！系统会自动添加。

在 javascript 方面，您有相应的方法来为这些变量设置值：

* uniform buffer: setUniformBuffer(name, buffer)
* storage 纹理: 与普通纹理的设置方法相同 (setTexture(name, texture))
storage buffer: setStorageBuffer(name, buffer)
external 纹理: setExternalTexture(name, buffer)

## 示例

这个 playground 是在 ShaderMaterial 中使用 WGSL 的基本示例：

[Basic example of WGSL with ShaderMaterial](https://playground.babylonjs.com/?webgpu#6GFJNR#178)

与使用 GLSL 时一样，ShaderMaterial 支持 WGSL 中的变形、骨骼和实例化。您需要在代码中添加适当的包含以支持这些功能。在这个 playground 中看看它是如何完成的（这个例子还演示了如何使用存储纹理和存储缓冲区）：[Advanced usage of the ShaderMaterial class](https://playground.babylonjs.com/?webgpu#8RU8Q3#155)

您还可以使用 5.0 中新增的烘焙顶点动画功能以及剪辑平面。看：[Using BVA and clip planes in WGSL](https://playground.babylonjs.com/?webgpu#8RU8Q3#156)

在 WebGPU 中使用常规 VideoTexture 播放视频很慢，因为在浏览器的幕后发生了大量纹理副本。 texture_external 类型对象用于在 WebGPU 中快速播放视频。这个 playground 展示了如何使用 ShaderMaterial 类来实现带有 texture_external 的视频播放：[Video playing with the ShaderMaterial class](https://playground.babylonjs.com/?webgpu#6GFJNR#179)