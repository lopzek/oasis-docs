---
order: 5
title: Model Mesh
type: 图形
group: 网格
---

[ModelMesh](${api}core/ModelMesh) 是一个用于描述几何体轮廓的网状渲染数据类，主要包含了顶点（位置、法线和 UV 等）、索引和混合形状等数据。不仅可以使用建模软件制作并导出 glTF 在引擎中解析还原，还可以方便的使用脚本直接写入数据创建。

<playground src="obj-loader.ts"></playground>

## 脚本创建

### 代码示例

```TypeScript
const entity = rootEntity.createChild('mesh-example');
const meshRenderer = entity.addComponent(MeshRenderer);

const modelMesh = new ModelMesh(engine);

// Set vertieces data
const positions = [
  new Vector3(-1.0, -1.0,  1.0),
  new Vector3( 1.0, -1.0,  1.0),
  new Vector3( 1.0,  1.0,  1.0),
  new Vector3( 1.0,  1.0,  1.0),
  new Vector3(-1.0,  1.0,  1.0),
  new Vector3(-1.0, -1.0,  1.0)
];
modelMesh.setPositions(positions);

// Add SubMesh
modelMesh.addSubMesh(0, 6);

// Upload data
modelMesh.uploadData(false);

meshRenderer.mesh = modelMesh;
meshRenderer.setMaterial(new UnlitMaterial(engine));
```

### 详细介绍

`ModelMesh` 的使用分为三步：

1. **设置数据**

通过 `setPositions（）`、 `setColors（）`等方法设置顶点数据

```TypeScript
modelMesh.setPositions([
  new Vector3(-1.0, -1.0,  1.0),
  new Vector3( 1.0, -1.0,  1.0),
  new Vector3( 1.0,  1.0,  1.0),
  new Vector3( 1.0,  1.0,  1.0),
  new Vector3(-1.0,  1.0,  1.0),
  new Vector3(-1.0, -1.0,  1.0)
]);

modelMesh.setColors([
    new Color(1, 0, 0),
    new Color(1, 1, 0),
    new Color(0, 1, 1),
    new Color(0, 1, 0),
    new Color(0, 1, 1),
    new Color(1, 0, 1)
]);
```

设置数据的 API 有：

| API                                                   | 说明                   |
| ----------------------------------------------------- | ---------------------- |
| [setPositions](${api}core/ModelMesh#setPositions)     | 设置顶点坐标           |
| [setIndices](${api}core/ModelMesh#setIndices)         | 设置索引数据           |
| [setNormals](${api}core/ModelMesh#setNormals)         | 设置逐顶点法线数据     |
| [setColors](${api}core/ModelMesh#setColors)           | 设置逐顶点颜色数据     |
| [setTangents](${api}core/ModelMesh#setTangents)       | 设置逐顶点切线         |
| [setBoneWeights](${api}core/ModelMesh#setBoneWeights) | 设置逐顶点骨骼权重     |
| [setBoneIndices](${api}core/ModelMesh#setBoneIndices) | 设置逐顶点骨骼索引数据 |
| [setUVs](${api}core/ModelMesh#setUVs)                 | 设置逐顶点 uv 数据     |

可以根据需求选择性设置（注意位置是必要数据且需要最先设置）。

2. **添加 SubMesh**

[SubMesh](${api}core/SubMesh) 主要包含了绘制范围和绘制方式等信息。调用 [addSubMesh](${api}core/ModelMesh#addSubMesh) 添加。

```TypeScript
modelMesh.addSubMesh(0, 2, MeshTopology.Triangles);
```

3. **上传数据**

调用 [uploadData()](${api}core/ModelMesh#uploadData) 方法。

如果不再需要修改 `ModelMesh` 数据，`noLongerAccessible` 参数设置为 `true`：

```TypeScript
modelMesh.uploadData(true);
```

如果需要持续修改 `ModelMesh` 数据，`noLongerAccessible` 参数设置为 `false`：

```TypeScript
modelMesh.uploadData(false);
```

<playground src="model-mesh.ts"></playground>

## 脚本添加 BlendShape 动画

`BlendShape` 通常用于制作精细程度非常高的动画，比如表情动画等。其原理也比较简单，主要通过权重混合基础形状和目标形状的网格数据来表现形状之间过度的动画效果。

**glTF 导入 BlendShape 动画案例：**
<playground src="skeleton-animation-blendShape.ts"></playground>

**脚本自定义 BlendShape 动画案例：**
<playground src="skeleton-animation-customBlendShape.ts"></playground>

### 详细步骤

1. **组织`BlendShape`数据**

   首先我们先创建一个`BlendShape` 对象，然后调用 [addFrame()](${api}core/ModelMesh#addFrame)添加混合形状的帧数据，一个 `BlendShape` 可以添加多个关键帧，每一帧由**权重**和**几何体偏移数据**组成 其中**偏移位置**是必要数据，**偏移法线**和**偏移切线**为可选数据。

   然后我们通过`Mesh`的`addBlendShape()` 方法添加创建好的`BlendShape`。

   ```typescript
   // Add BlendShape
   const deltaPositions = [
      new Vector3(0.0, 0.0, 0.0),
      new Vector3(0.0, 0.0, 0.0),
      new Vector3(-1.0, 0.0, 0.0),
      new Vector3(-1.0, 0.0, 0.0),
      new Vector3(1.0, 0.0, 0.0),
      new Vector3(0.0, 0.0, 0.0)
    ];
    const blendShape = new BlendShape("BlendShapeA");
    blendShape.addFrame(1.0, deltaPositions);
    modelMesh.addBlendShape(blendShape);
   ```

   

2. **通过权重调整至目标 `BlendShape`**

   现在我们要将网格的形状完全调整为刚才添加的`BlendShape`，我们需要设置一个权重数组，由于我们只添加了一个`BlendShape`，所以权重数组长度为1即可，并把第一个元素的值设置为1.0。

   ```typescript
   // Use `blendShapeWeights` property to adjust the mesh to the target BlendShape
   skinnedMeshRenderer.blendShapeWeights = new Float32Array([1.0]);
   ```

   
