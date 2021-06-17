创建一个TiledBox
===

### TiledBox

TiledBox只能通过*MeshBuilder*创建。一个TiledBox是由六个TiledPlane组成的，所以每个面的tile尺寸，模式，对齐等属性都是一样的。结合faceUV属性，每个面都可以使用漫反射贴图的不同部分，就像在[这里](./)描述的一样。

### MeshBuilder

使用：

````javascript
const tiledBox = BABYLON.MeshBuilder.CreateTiledBox("tiled box", options, scene); //scene is optional and defaults to the current scene
````

|options属性|值|默认值|
|--|--|--|
|size|(number)各方向的尺寸|1|
|height|(number)高度，覆盖size属性|size|
|width|(number)宽度，覆盖size属性|size|
|depth|(number)深度，覆盖size属性|size|
|tileSize|(number)每个tile的尺寸|1|
|tileHeight|(number)每个tile的高度，覆盖tileSize属性|tileSize|
|tileWidth|(number)每个tile的宽度，覆盖tileSize属性|tileSize|
|faceColors|(Color4[])6个Color4组成的数组，每个代表一个面的颜色|每个面都为Color4(1, 1, 1, 1)|
|faceUV|(Vector4[])6个Vector4组成的数组，每个代表一个面的UV|每个面都为Vector4(0, 0, 1, 1)|
|pattern|(number)tile重复或旋转的模式|NO_FLIP|
|alignVertical|(number)所有tile在当前的面居上，居下，或居中对齐|CENTER|
|alignHorizontal|(number)所有tile在当前的面居左，居右，或居中对齐|CENTER|
|updatable|(boolean)如果为true网格体可以被更新|false|
|sideOrientation|(number)面朝向|DEFAULTSIDE|

[创建一个TiledBox](https://playground.babylonjs.com/#FAP6ZC#3)

选项中的pattern属性，可以设置下列枚举值：

````javascript
BABYLON.Mesh.NO_FLIP, default
BABYLON.Mesh.FLIP_TILE,
BABYLON.Mesh.ROTATE_TILE,
BABYLON.Mesh.FLIP_ROW,
BABYLON.Mesh.ROTATE_ROW,
BABYLON.Mesh.FLIP_N_ROTATE_TILE,
BABYLON.Mesh.FLIP_N_ROTATE_ROW
````

TILE结尾的枚举值表示每一个tile格子都被翻转或者旋转。

ROW结尾的枚举表示每隔一整行tile被翻转或者旋转。

当平面的长度和宽度不恰好是tile尺寸的整数倍时，那么tile平铺的时候会有被切掉的情况。这时你可以规定被切掉的部分出现在平面的哪个位置，可以在平面的一边，也可以同时出现在两边。你可以通过设置选项中的*alignVertical*或者*alignHorizontal*属性，来实现上述设置。例如，如果设置*alignHorizontal*

// TODO