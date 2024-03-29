# WebGPU 限制

这里列出了截至 2022 年 1 月 16 日 WebGPU 规范/实施的当前限制。

## 32 位浮点纹理不可线性过滤

32 位浮点纹理不可线性过滤（请参阅[纹理格式列表](https://www.w3.org/TR/webgpu/#plain-color-formats)），这意味着您不能对它们使用双/三线性过滤，例如。您将需要改用 16 位浮点纹理，直到规范（或扩展）添加对它的支持。

## varying的支持数量少

Chrome（这是目前唯一受 Babylon.js 支持的浏览器，因为其他浏览器在 WebGPU 规范实现方面落后）仅支持 14 个“varyings”，意思是在顶点着色器中创建并在顶点着色器中重用的变量片段着色器。

这意味着 kernelBlur 着色器比 WebGL 更受限制，以防您的 GPU 支持超过 14 个变量（例如，GTX1080 支持多达 30 个变量）。

它还限制了节点材质的复杂性：一些可以在 WebGL 中使用的材质因此在 WebGPU 中无法使用。