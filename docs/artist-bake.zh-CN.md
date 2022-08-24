---
order: 2
title: C4D 烘焙
type: 美术
group: 教程
---

以 C4D-OC 渲染器的烘培（windows）为例讲解。

### 什么是烘焙

关于烘培就是将所有渲染好的材质颜色信息用一张贴图的形式表达出来。
 
烘培首先需要两组模型：一个高模，一个低模。高模用于烘培精细程度较高的贴图，低模用于在进入引擎后使用贴图两者在制作时要保证 uv 的统一，所以先产生低模排布好 uv 然后进行细节加工产生高模，高模可以较多的制作细节来烘培出细节更丰富的贴图。


![1.gif](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*pbduQosyOJwAAAAAAAAAAAAAARQnAQ)

左边为低模 右边为高模 从布线信息可以看得出来，并且高模细的细节会多一些。

![2.gif](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*SgbzSKngA2IAAAAAAAAAAAAAARQnAQ)

随便布线细节有差异但是两者的可以被看到的部分要保持一致 被遮盖的部分不做考虑所以高模必须由低模产生来保证uv的一致。


### 具体的烘培过程

1.将准备好的高模用C4D进行调节材质渲染出来需要的效果，像面部使用的贴图也需要按照整体的uv排布来绘制贴图。调节好材质之后就可以准备进行烘培了。

![image.png](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*u81UTYTkSVMAAAAAAAAAAAAAARQnAQ)


2.  烘培重要的一点是需要对摄像机的模式进行选择，对需要进行输出的摄像机进行标签指定，加入OC渲染器特有的摄像机标签。

![](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*gRWvSK1MoTMAAAAAAAAAAAAAARQnAQ)                                      


3. 点击添加好的摄像机标签进入标签属性摄像机类型会有很多选项其中一项为烘培，选择烘培。

![image.png](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*7XApTKsQy9wAAAAAAAAAAAAAARQnAQ)


4. 在烘培菜单中将烘培群组ID设置为除1以外的数字这里设置为2。

![image.png](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*n_1qRIkFtdAAAAAAAAAAAAAAARQnAQ)


5. 然后需要将需要烘培的模型并为一组，如下图所示将所有需要的模型打一个组并且加入OC对象标签。

![image.png](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*_iMOSaTyfroAAAAAAAAAAAAAARQnAQ)


6. 点击标签出现标签属性选择对象图层，然后将里面的烘培ID设置为和烘培群组ID一样的数值这里是2，然后点击渲染即可,这样就可以烘培出需要的图片。

![image.png](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*lP1pQqZWZC8AAAAAAAAAAAAAARQnAQ)

![image.png](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*gsxbTZBSKGQAAAAAAAAAAAAAARQnAQ)


如果对烘焙的结果不是非常满意，C4D、Substance Painter 都能够涂刷修改贴图。真实感渲染不是唯一的选择，涂刷贴图也可以用来还原一些特殊的风格。

![image.png](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*PCz8TpYJd5wAAAAAAAAAAAAAARQnAQ)

![image.png](https://gw.alipayobjects.com/mdn/rms_d27172/afts/img/A*8mwtRY6YdiIAAAAAAAAAAAAAARQnAQ)
