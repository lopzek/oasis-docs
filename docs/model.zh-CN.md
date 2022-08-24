---
order: 3
title: 3. 加载一个 3D 模型
type: 快速入门
group: 基础
---

glTF（GL Transmission Format）是 [khronos](https://www.khronos.org/) 发布的一种能高效传输和加载 3D 场景的规范，与 FBX、OBJ 等传统模型格式一样，基本支持 3D 场景中的所有特性，也是目前 Oasis 推荐的首选 3D 文件格式。

glTF 的产物一般分为（.gltf + .bin + png）或者 (.glb)，前者适合图片体积大的场景，所以将图片和模型拆分开来，可以异步加载模型和纹理；后者适合模型文件较大的场景，会将所有数据进行二进制保存，需要等所有数据解析完毕才能展示模型。

## 基本使用

加载一个 3D 模型只要调用引擎 [ResourceManager](${docs}resource-manager-cn) 实例的 [load](${api}core/ResourceManager/#load) 方法即可，如下：

```typescript
engine.resourceManager.load("{glTF source}").then((gltf) => {
  entity.addChild(gltf.defaultSceneRoot);
});
```

如果在一个异步函数体中使用，可以采用 `async/await` 语法：

```typescript
const gltf = await this.engine.resourceManager.load("{glTF source}");

entity.addChild(gltf.defaultSceneRoot);
```

如下 demo：

<playground src="gltf-basic.ts"></playground>

## 更多案例

glTF 拥有非常多的特性，官网提供了大量的[示例](https://github.com/KhronosGroup/glTF-Sample-Models/tree/master/2.0)进行参考，Oasis 也提供了一份复刻版本进行快速浏览，可以通过以下 **glTF List** 切换不同的 glTF 模型。

前往[ glTF 资源](${docs}gltf-cn) 了解更多 glTF 相关设计。

<playground src="gltf-loader.ts"></playground>
