---
order: 1
title: Physics Overall
type: Physics
---

The physics engine is a very important part of the game engine. The industry generally adopts PhysX to introduce related
functions. But for lightweight scenarios, PhysX makes the size of final application very large, beyond the constraints
of
these projects. Oasis is based on a multi-backend design. On the one hand, it uses WebAssembly Compile to
get [PhysX.js](https://github.com/oasis-engine/physX.js) ; on the other hand, it also provides a lightweight physics
engine. Both are identical in [API](https://github.com/oasis-engine/engine/tree/main/packages/design/src/physics)
design. Users only need to select a specific physical backend when initializing the engine. It can meet the needs of
various scenarios such as lightweight applications and heavyweight games. For the overall design of physical components,
you can refer to [Wiki](https://github.com/oasis-engine/engine/wiki/Physical-system-design)

For scenes that need to use various physics components, `InputManager` and other scenes that need to be picked up by
Raycast, you need to initialize the physics engine before use. Currently, the Oasis engine provides two built-in physics
engine backend implementations:

1. [physics-lite](https://github.com/oasis-engine/engine/tree/main/packages/physics-lite)
2. [physics-physx](https://github.com/oasis-engine/engine/tree/main/packages/physics-physx)

Initialization only needs to bind the static objects of these two backends to `physicsManager`:

```ts
import {LitePhysics} from "@oasis-engine/physics-lite";

const engine = new WebGLEngine("canvas");
engine.physicsManager.initialize(LitePhysics);
```

## Wasm version physics engine loading and initialization

Since WASM needs to be loaded asynchronously, the initialization of the engine needs to be done in the callback of
Promise.

```ts
import {PhysXPhysics} from "@oasis-engine/physics-physx";

PhysXPhysics.initialize().then(() => {
  const engine = new WebGLEngine("canvas");
  engine.physicsManager.initialize(PhysXPhysics);

  engine.run();
})
```

## Select physical backend

Choosing a physical backend requires considering three factors: functionality, performance, and package size:

1. Functionality: In pursuit of complete physics engine functions and high-performance physics simulation,
   the `PhysX backend`
   is recommended, and the `Lite backend` only supports collision detection.
2. Performance: `PhysX` will automatically downgrade to pure JavaScript code on platforms that do not support
   WebAssembly,
   so performance will also decrease. But because of the built-in data structure for scene search, the performance is
   still better than the Lite backend.
3. Package size: Selecting the `PhysX backend` will introduce an additional wasm file close to 2.5mb (the size of the
   pure
   JavaScript version is similar), which increases the size of the package and reduces the speed of application
   initialization.

## Debugging of physical components

Physics colliders are composites of basic physics shapes, including spheres, boxes, capsules, and infinite planes. In
practical applications, the shapes of these colliders rarely exactly coincide with the rendered objects, which brings
great difficulties to visual debugging.
There are two debugging methods:

1. With the help of PhysX Visual Debugger (PVD), it is an official debugging tool developed by Nvidia, but to use this
   tool, you need to compile the debug version of PhysX yourself, and use WebSocket to connect the browser and the
   debugging tool.
   For specific usage, please refer to the introduction of Readme
   in [physx.js](https://github.com/oasis-engine/physX.js).
2. We also provide a
   lightweight [auxiliary-lines tool](https://github.com/oasis-engine/engine-toolkit/tree/main/packages/auxiliary-lines)
   , which is based on the configuration of physical components Draw the corresponding wireframes to assist in
   configuring and debugging physical components.
   It is also very easy to use, just mount the `WireframeManager` script, and then set it to associate various physical
   components, or directly associate nodes:

```ts
const wireframe = rootEntity.addComponent(WireframeManager);
wireframe.addCollideWireframe(collider);
````
<playground src="physics-debug-draw.ts"></playground>
