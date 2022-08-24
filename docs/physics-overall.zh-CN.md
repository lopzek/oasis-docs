---
order: 1
title: 物理总览 
type: 物理
---

物理引擎是游戏引擎中非常重要的组成部分。 业界普遍采用 PhysX 引入相关功能。 但是对于轻量级的场景，PhysX 使得最终的应用体积非常大，超出了这些项目的限制。 Oasis 基于多后端设计。 一方面，它通过 WebAssembly
编译得到 [PhysX.js](https://github.com/oasis-engine/physX.js) ; 另一方面，它也提供了轻量级的物理引擎。
两者在 [API](https://github.com/oasis-engine/engine/tree/main/packages/design/src/physics) 设计上是一致的。 用户只需要在初始化引擎时选择特定的物理后端。
可以满足轻量级应用、重量级游戏等各种场景的需求。有关物理组件的总体设计，可以参考 [Wiki](https://github.com/oasis-engine/engine/wiki/Physical-system-design).

对于需要使用各种物理组件，以及 `InputManager` 等需要 Raycast 拾取的场景，都需要在使用之前初始化物理引擎。目前 Oasis 引擎提供两种内置的物理引擎后端实现：

1. [physics-lite](https://github.com/oasis-engine/engine/tree/main/packages/physics-lite)
2. [physics-physx](https://github.com/oasis-engine/engine/tree/main/packages/physics-physx)

初始化只需要将这两个后端的静态对象，绑定到 `physicsManager` 中即可：

```ts
import {LitePhysics} from "@oasis-engine/physics-lite";

const engine = new WebGLEngine("canvas");
engine.physicsManager.initialize(LitePhysics);
```

## Wasm 版物理引擎加载与初始化

由于WASM需要异步加载，因此引擎的初始化需要放在 Promise 的回调中进行。

```ts
import {PhysXPhysics} from "@oasis-engine/physics-physx";

PhysXPhysics.initialize().then(() => {
  const engine = new WebGLEngine("canvas");
  engine.physicsManager.initialize(PhysXPhysics);

  engine.run();
})
```

## 选择物理后端
选择物理后端需要考虑到功能，性能和包尺寸这三个因素：
1. 功能：追求完整物理引擎功能以及高性能的物理模拟，推荐选择 PhysX 后端，Lite 后端只支持碰撞检测。
2. 性能：PhysX 会在不支持 WebAssembly 的平台自动降级为纯 JavaScript 的代码，因此性能也会随之降低。但由于内置了用于场景搜索的数据结构，性能比 Lite 后端还是要更加好。
3. 包尺寸：选择 PhysX 后端会额外引入接近 2.5mb 的 wasm 文件（纯 JavaScript 版的大小接近），增加包的大小的同时降低应用初始化的速度。

## 物理组件的调试
物理碰撞器由基础物理外形复合而成，包括了球，盒，胶囊和无限大的平面。在实际应用中，这些碰撞器外形很少与渲染的物体刚好是完全重合的，这位可视化调试带来了很大的困难。
有两种调试方法：
1. 借助 PhysX Visual Debugger(PVD)，是 Nvidia 官方开发的调试工具，但是用这一工具需要自行编译 debug 版本的PhysX，并且借助 WebSocket 串联浏览器和该调试工具。
具体的使用方法，可以参考 [physx.js](https://github.com/oasis-engine/physX.js) 的Readme种的介绍。
2. 我们还提供了轻量级的[辅助线工具](https://github.com/oasis-engine/engine-toolkit/tree/main/packages/auxiliary-lines)，该工具根据物理组件的配置绘制对应的线框，辅助配置和调试物理组件。
使用起来也非常容易，只需要在挂载 `WireframeManager` 脚本，然后设置其关联各种物理组件，或者直接关联节点即可：
```ts
const wireframe = rootEntity.addComponent(WireframeManager);
wireframe.addCollideWireframe(collider);
```
<playground src="physics-debug-draw.ts"></playground>
