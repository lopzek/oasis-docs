---
order: 1
title: 动画组件
type: 动画
---

动画控制组件（[Animator](${api}core/Animator)）可以通过状态机组织动画片段（[AnimationClip](${api}core/AnimationClip)）实现更加灵活丰富的动画效果。

## 基本使用
在加载GLTF模型后引擎会自动为模型添加一个Animator组件，并将模型中的动画片段加入其中。可以直接在模型的根实体上获取Animator组件，并播放指定动画。
```typescript
engine.resourceManager
  .load<GLTFResource>("https://gw.alipayobjects.com/os/bmw-prod/5e3c1e4e-496e-45f8-8e05-f89f2bd5e4a4.glb")
  .then((asset) => {
    const { defaultSceneRoot } = asset;
    rootEntity.addChild(defaultSceneRoot);
    const animator = defaultSceneRoot.getComponent(Animator);
    animator.play("run");
});
```

### 控制播放速度

你可以通过 [speed](${api}core/Animator#speed)  属性来控制动画的播放速度。 `speed` 默认值为 `1.0` ，值越大播放的越快，越小播放的越慢。当值为负数时，进行倒播。


```typescript
animator.speed = 2.0；
```

### 设置动画数据

你可以通过 [animatorController](${api}core/Animator#animatorController)  属性来设置动画控制器的动画数据，加载完成的GLTF模型会自动添加一个默认的AnimatorController。


```typescript
animator.animatorController = new AnimatorController()；
```


### 播放指定动画状态

<playground src="skeleton-animation-play.ts"></playground>

你可以使用 [play](${api}core/Animator#play) 方法来播放指定的AnimatorState。参数为AnimatorState的`name`，其他参数说明详见[API文档](${api}core/Animator#play)。

```typescript
animator.play("run");
```

如果需要在动画中的某一时刻开始播放可以通过以下方式

```typescript
const layerIndex = 0;
const normalizedTimeOffset = 0.5; // 归一化的时间
animator.play("run", layerIndex, normalizedTimeOffset);
```

### 获取当前在播放的动画状态

你可以使用 [getCurrentAnimatorState](${api}core/Animator#getCurrentAnimatorState) 方法来获取当前正在播放的AnimatorState。参数为动画状态所在层的序号`layerIndex`, 详见[API文档](${api}core/Animator#getCurrentAnimatorState)。获取之后可以设置动画状态的属性，比如将默认的循环播放改为一次。

```typescript
const currentState = animator.getCurrentAnimatorState(0);
// 播放一次
currentState.wrapMode = WrapMode.Once;
// 循环播放
currentState.wrapMode = WrapMode.Loop;
```

### 获取动画状态

你可以通过如下方式获取指定的动画状态, 参考[API文档](${api}core/Animator#animatorController)

```typescript
const state = animator.animatorController.layers[layerIndex].stateMachine.findStateByName('xxx');
// 播放一次
state.wrapMode = WrapMode.Once;
// 循环播放
state.wrapMode = WrapMode.Loop;
```

### 动画过渡

<playground src="skeleton-animation-crossfade.ts"></playground>

你可以使用 [crossFade](${api}core/Animator#crossFade) 方法来使角色过渡到指定状态。第一个参数为要过渡到的动画状态的`name`，第二个参数为归一化的动画过渡时间，其他参数说明详见[API文档](${api}core/Animator#crossFade)。


### 动画事件

<playground src="animation-event.ts"></playground>

你可以使用 [AnimationEvent](${api}core/AnimationEvent) 来为AnimationClip添加事件，动画事件将在指定时间调用你在同一实体上绑定组件的指定回调函数。

```typescript
const event = new AnimationEvent();
event.functionName = "test";
event.time = 0.5;
clip.addEvent(event);
```

## 使用动画状态机控制动画

在进一步介绍使用方法之前，先简单介绍下动画系统的构成以便理解。动画系统的构成如下：


### 动画系统的构成

![image-20210830233452874](https://gw.alipayobjects.com/zos/OasisHub/b973418a-cca7-46c9-9298-a54e7d445f70/image-20210830233452874.png)

#### [Animator](${api}core/Animator)
动画控制器组件，用于控制动画的播放。Animator 组件读取 AnimatorController 作为动画数据。通过 AnimatorControllerParameter 设置该 Animator 中的变量。

#### [AnimatorController](${api}core/AnimatorController)
用于存储 Animator 组件的动画数据。一个 AnimatorController 包含多个 AnimatorControllerLayer，用于分层播放或动画叠加。

#### AnimatorControllerParameter（开发中）
动画控制器中使用的变量，使用户可以通过在脚本中改变变量以控制动画状态的切换。

#### [AnimatorControllerLayer](${api}core/AnimatorControllerLayer)
存储该层的动画状态机数据，混合模式以及混合的权重。多个 AnimatorControllerLayer 同时播放时可以通过设置 `blendingMode = AnimatorLayerBlendingMode.Additive` 实现动画叠加的效果。

#### [AnimatorStateMachine](${api}core/AnimatorStateMachine)
每个 AnimatorControllerLayer 中有一个 AnimatorStateMachine，用于控制动画状态的播放及状态间的切换及过渡。

#### [BlendingMode](${api}core/AnimatorControllerLayer#blendingMode)
动画层的混合模式，默认为 `AnimatorLayerBlendingMode.Override` 既覆盖模式，可以通过将下面的层设置为 `AnimatorLayerBlendingMode.Additive` 实现动画叠加的效果。

#### [AnimatorState](${api}core/AnimatorState)
AnimatorState 是 AnimatorStateMachine 的基本构成。可以控制 AnimationClip 的速度，是否循环，开始结束时间。每个 AnimatorState 需绑定一个 AnimationClip，当处于该状态时，则会播放该AnimationClip。

#### [AnimatorTransition](${api}core/AnimatorTransition)
AnimatorTransition定义了状态机何时以及如何从一个状态过渡到另一个状态。通过它可以设置两个动画状态的过渡开始时间 `exitTime`，目标状态的开始时间 `offset` 及过渡时长 `duration`。

#### [AnimationClip](${api}core/AnimationClip)
动画片段，存储设计师制作的基于关键帧的动画数据。一个 AnimationClip 一般对应一个模型的特定动作，每个 AnimationClip 包含多个 AnimationCurve。

#### [AnimationCurve](${api}core/AnimationCurve)
一个模型拥有多个骨骼，模型动画中每个骨骼实体的指定属性的动画关键帧数据存储于 AnimationCurve 中。一个 AnimationCurve 中包含多个 Keyframe 既关键帧数据。

#### [AnimationEvent](${api}core/AnimationEvent)
AnimationEvent 可以让你在指定时间调用其同一实体绑定的脚本的回调函数.

#### [Keyframe](${api}core/KeyFrame)
存储动画关键帧数据，既指定时间实体的属性的值应是多少。

#### [Interpolation](${api}core/AnimationCurve#interpolation)
动画曲线中关键帧的插值方式。既当时间在两个关键帧间时，属性的值该如何计算。

### 使用AnimatorTransition做动画过渡
你可以通过为AnimatorState添加AnimatorTransition实现动画状态间的过渡。

```typescript
const walkState = animatorStateMachine.addState('walk');
walkState.clip = walkClip;
const runState = animatorStateMachine.addState('run');
runState.clip = runClip;
const transition = new AnimatorStateTransition();
transition.duration = 1;
transition.offset = 0;
transition.exitTime = 0.5;
transition.destinationState = runState;
walkState.addTransition(runState);
animator.play("walk");
```
通过这样的方式你之后每次在该动画状态机所在的层播放 `walk` 动画时都会在播放一半时开始过渡到 `run` 动画。

### 动画叠加

<playground src="skeleton-animation-additive.ts"></playground>

动画叠加是通过AnimatorControllerLayer间的混合达到的效果。第一层是基础动画层，修改它的权重及混合模式将不会生效。将想要叠加的动画状态添加到其他层并将它的混合模式设置为 `AnimatorLayerBlendingMode.Additive` 即可实现动画叠加效果，Oasis引擎支持多层的动画叠加。

### 状态机脚本

<playground src="animation-stateMachineScript.ts"></playground>

状态机脚本为用户提供了动画状态的生命周期钩子函数来编写自己的游戏逻辑代码。用户可以通过继承 [StateMachineScript](${api}core/StateMachineScript) 类来使用状态机脚本。

状态机脚本提供了三个动画状态周期：

- `onStateEnter`：动画状态开始播放时回调。
- `onStateUpdate`：动画状态更新时回调。
- `onStateExit`：动画状态结束时回调。

```typescript
class theScript extends StateMachineScript {
  // onStateEnter is called when a transition starts and the state machine starts to evaluate this state
  onStateEnter(animator: Animator, stateInfo: any, layerIndex: number) {
    console.log('onStateEnter', animator, stateInfo, layerIndex);
  }

  // onStateUpdate is called on each Update frame between onStateEnter and onStateExit callbacks
  onStateUpdate(animator: Animator, stateInfo: any, layerIndex: number) {
    console.log('onStateUpdate', animator, stateInfo, layerIndex);
  }

  // onStateExit is called when a transition ends and the state machine finishes evaluating this state
  onStateExit(animator: Animator, stateInfo: any, layerIndex: number) {
    console.log('onStateExit', animator, stateInfo, layerIndex);
  }
}

animatorState.addStateMachineScript(theScript)
```

