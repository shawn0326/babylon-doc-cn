节点材质和PBR
===

### 创建PBR材质

得益于 PBRMettalic_Roughness 块，使用基于物理的渲染和节点材质非常容易。

您可以使用这些 playground 和材质作为您自己实验的起点，在 NME 中创建 PBR 材质（请注意，节点材质可能需要一些时间才能加载到 PlayGround 中——网格将保持黑色，直到材质加载完毕）：

* PBR块的用法：
  * PlayGround：[Playground](https://playground.babylonjs.com/#D8AK3Z#44)
  * 材质：[NME](https://nme.babylonjs.com/#EPY8BV#6)
* 带sheen的PBR材质:
  * PlayGround：[Playground](https://playground.babylonjs.com/#MUX769#4)
  * 材质：[NME](https://nme.babylonjs.com/#V3R0KJ)
* 带clear coat的PBR材质:
  * PlayGround：[Playground](https://playground.babylonjs.com/#0XSPF6#6)
  * 材质：[NME](https://nme.babylonjs.com/#C3NEY1#4)
* 带sub surface的PBR材质:
  * PlayGround：[Playground](https://playground.babylonjs.com/#7QAN2T#8)
  * 材质：[NME](https://nme.babylonjs.com/#100NDL#1)

PBR 块的输入名与 PBRMetallicRoughnessMaterial 类中属性名相同，因此您可以参考此文档以获取有关它们的解释。

单击 NME 中的块时，某些参数可作为属性使用。

例如，对于反射：

![](https://doc.babylonjs.com/img/how_to/Materials/nme_reflection_prop.png)

或者对于 PBRMetallicRoughness：

![](https://doc.babylonjs.com/img/how_to/Materials/nme_pbr_prop.png)

至于标准 PBRMaterial，如果没有为反射/折射纹理提供纹理，则使用在场景级别 (scene.environmentTexture) 声明的纹理。

默认情况下，如果某些东西连接到 FragmentOutput 块的输入，则会启用 alpha 混合。如果您不需要 alpha 混合，请不要连接此输入。

关于 PBRMetallicRoughness 块，如果需要，您可以单独访问每个输出组件（环境光、漫反射、镜面反射……），或者您可以直接使用光照来获得复合输出。在单独输出的名称中，dir 表示直接（来自直接光的分量），Ind 表示间接（来自间接照明的分量，表示环境）。

关于图像处理和手动合成的注意事项：请注意，PBRMetallicRoughness 块的合成照明输出还添加了来自场景的图像处理。如果您希望向标准照明设置添加额外的组件，您将需要使用分离的组件自己进行合成。分离组件的输出在线性颜色空间中。这很重要，因为如果您希望在手动合成中计算场景图像处理，则需要 ImageProcessing 块。默认情况下，此块采用伽玛颜色空间中的输入值，并运行与线性颜色空间输出的内部对话。您需要在 ImageProcessing 块属性中关闭此转换，以便在不进行转换的情况下线性传递。

<iframe width="501" height="282" src="https://www.youtube.com/embed/CRg8P1Af1M0" title="PBR Nodes in Node Materials Part 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>