---
order: 5
title: Transform
type: Core
---

<playground src="transform-basic.ts"></playground>

## Transform
Transform is closely related to the whole rendering process of webgl. Each entity in the scene, including the camera, has its own transform.
<br>
When rendering a scene, it is necessary to prepare the object and put it in a proper position. This process is model transformation.
<br>
Then we need to prepare the camera and put it in the right position. This process is view transformation.
<br>
After that, adjust the parameters of the camera, such as focal length and scale. This process is projection transformation.
<br>
The final result of this transformation is to determine the overall size of the drawing.
<br>
Among them, model transformation, view transformation and projection transformation are defined and implemented in our own shaders.
<br>
When you create a new entity, including the camera, a transform component is automatically added to the entity.The transformation component can help us realize the transformation process mentioned above very conveniently.
<br>
In the process of transformation,we often encounter the transformation of coordinate system space.
<br>
Usually, we make the root coordinate system of the whole scene into the world coordinate system. If we do not transform the model after creating the entity in the scene, the model will be placed at the origin.Similarly, if you create a camera without model transformation,the camera will be placed at the origin.
<br>
In order to make the camera capture things, we need to model the camera or other entities in the scene, move or rotate, so that the object appears in the camera's shooting range. At the same time, adjust the camera parameters (projection transformation) so that the camera can capture things~
<br>
In addition to directly adding entities in the scene, you can also add entities to an entity, that is, to establish a parent-child relationship between two entities.
<br>
At this time, the coordinate system of the child entity is no longer the world coordinate system, but the model coordinate system of the parent entity. The transformation of a child entity is affected by the parent entity.
<br>
Although it sounds complicated, the transformation component has done all the calculations for us.The following describes the basic usage of transform components.

## Usage

```typescript
// create entity
const scene = engine.sceneManager.activeScene;
const root = scene.createRootEntity('root');
const cubeEntity = root.createChild('cube');

// The entity will have its own transform component by default after it is created
// You can do geometrical transformation through the transform component api

// Modify displacement, rotation, scaling
transform.position = new Vector3();
// or transform.setPosition(0, 0, 0);

transform.rotation = new Vector3(90, 0, 0);
// or transform.setRotation(90, 0, 0);

// You can alose get transform component from entity object directly.
cubeEntity.transform.scale = new Vector3(2, 1, 1);
// or cubeEntity.transform.setScale(2, 1, 1);

// translate cube entity in local space
cubeEntity.transform.translate(new Vector3(10, 0, 0), true);

// rotate cube entity in local space
cubeEntity.transform.rotate(new Vector3(45, 0, 0), true);
```

## Component properties
| property name | interpretation |
| --- | --- |
| [position]($%7Bapi%7Dcore/Transform#position) | local position |
| [rotation]($%7Bapi%7Dcore/Transform#rotation) | local rotation - in euler |
| [rotationQuaternion]($%7Bapi%7Dcore/Transform#rotationquaternion) | local rotation - in quaternion |
| [scale]($%7Bapi%7Dcore/Transform#scale) | local scaling |
| [worldPosition]($%7Bapi%7Dcore/Transform#worldPosition) | world position |
| [worldRotation]($%7Bapi%7Dcore/Transform#worldRotation) | world rotation - in euler |
| [worldRotationQuaternion]($%7Bapi%7Dcore/Transform#worldRotationQuaternion) | world rotation - in quaternion |
| [lossyWorldScale]($%7Bapi%7Dcore/Transform#lossyWorldScale) | world lossy scaling - When the parent node is zoomed and the child node is rotated, the zoom will be tilted and cannot be represented correctly by Vector3, and must be represented by Matrix3x3 matrix. |
| [localMatrix]($%7Bapi%7Dcore/Transform#localMatrix) | local matrix |
| [worldMatrix]($%7Bapi%7Dcore/Transform#worldMatrix) | world matrix |



## 组件方法
| method name | interpretation |
| --- | --- |
| [getWorldUp]($%7Bapi%7Dcore/Transform#getWorldUp) | get world up vector |
| [getWorldRight]($%7Bapi%7Dcore/Transform#getWorldRight) | get world right vector |
| [getWorldForward]($%7Bapi%7Dcore/Transform#getWorldForward) | get world forward vector |
| [lookAt]($%7Bapi%7Dcore/Transform#lookAt) | Rotate and ensure that the world front vector points to the target world position |
| [registerWorldChangeFlag]($%7Bapi%7Dcore/Transform#registerWorldChangeFlag) | Register the world change change mark|
| [rotate]($%7Bapi%7Dcore/Transform#rotate) | Rotate according to the specified Euler angle |
| [rotateByAxis]($%7Bapi%7Dcore/Transform#rotateByAxis) | Rotate around the specified axis according to the specified angle |
| [translate]($%7Bapi%7Dcore/Transform#translate) | Translate according to the specified direction and distance |



## Common QA

- Why add the `transform` component?

In a 3D scene, the geometric transform of objects should be managed uniformly, rather than mixing transform-related data into `Entity` classes.
The new `transform` component has been heavily optimized with dirty flag to optimize calculations and improve performance.
​

- Why does modify the x value of transform position not work?

The transform component uses dirty flag to make a lot of optimizations. The updates of displacement, rotation, scale, world matrix and other attributes will all be affected by dirty flag. Only by modifying the attribute value of the `position` object, the object cannot be displaced. Therefore, using tweenjs to modify properties cannot achieve interpolation animation.
If you need to modify the displacement of the object, you can re-assign the value of `position` or call the `setPosition` method.


- What is the function of `registerWorldChangeFlag`?

Because the `worldMatrix` property of `transform` is also optimized with dirty marks~
Therefore, if you want to know if `worldMatrix` has changed, you need to get the current status of dirty flag for `worldMatrix`.
So, `transform` component provides the `registerWorldChangeFlag` method: this method will return an update flag, and the current `transform` `worldMatrix` will trigger the flag change when the `worldMatrix` is modified.
For specific usage, please refer to the camera component:

```typescript
class Camera {
	onAwake() {
  	this._transform = this.entity.transform;
    // Register change flag
    this._isViewMatrixDirty = this._transform.registerWorldChangeFlag();
  }
  get viewMatrix() {
    // When the mark is updated, the viewMatrix is ​​obtained according to the worldMatrix~
  	if (this._isViewMatrixDirty.flag) {
      this._isViewMatrixDirty.flag = false;
      Matrix.invert(this._transform.worldMatrix, this._viewMatrix);
    }
    return this._viewMatrix;
  }
}
```
