# 游戏引擎的分层

1.tool layer(编辑器这一层).
2.function layer(渲染,动画这一层)
3.resource layer(图片,3d模型,音乐等文件的管理层)
4.core layer(数学,以及硬件资源).
5.platform layer(输入设备)

相当于游戏中的逻辑时间
tick每一帧进行一次运算.logic(逻辑),render(渲染).

# 构建游戏世界

1.动态物(比如游戏的主角和敌人)
2.静态物(建筑)
3.环境(天气,植被,地形)

游戏的中的object一般用:
name
property
behavior
三者描述

功能去做模块化,而不是面向对象去描述实体.

基于component的流水线tick,批处理加速计算.
基于事件的处理,object直接不直接进行交互,而是生成事件发送给对应的object,object处理接收到的事件
改变自身的属性的和行为.

基于多叉树的场景管理.防止过多计算(
说白了就是基于业务,只计算与当前object相关的object,这种相关可以基于逻辑距离,也可以基于因果关系)

为了多线程使用中保证顺序,最终会写成类似生产者消费者的模式,消息最终会使用一个类似队列的东西集中管理.

# 游戏引擎中的渲染实战

simt在simd的基础上再扩展多核心,使计算效率变成simd*核数

渲染的时候一个mesh由多个三角形构成,一个三角形由三个顶点构成,一个三角形的顶点可能是另一个三角形的顶点,所以我们可以将顶点存在一个数组里面,达到顶点的复用
这就是vertex buffer.

shader就是mesh和材质的的组合.我要渲染一个面,这个面要渲染成虎皮花纹的

但是一个大的mesh可能由不同的mesh组成,所以就有了submesh的概念,每个submesh有自己的shader和texture.

给mesh换材质需要进行一次gpu的通信,所以游戏里经常给人换皮的会卡一下,这相当于给一个mesh重新着色.一种优化就是将所有的mesh通过素材grouping这样,就可以通过一个通信重新渲染所有
同样的mesh(批处理的思维).

只绘制摄像机可见范围内的画面
包围盒的概念:因为一个物体可能有几万个面,通过每一个面的计算去判断有没有碰撞或者击中效率太低了,所以用一个大致贴合物体且面更少的盒来包围物体,即命中盒子就命中了物体.

# 渲染中光和材质的数学魔法

环境贴图简化的光反射的计算.
基本的shading = simple light(简化的光源,点光,方向光,圆锥光等等)+ambient(除去主光外的物体表面光场# 游戏引擎的分层

1.tool layer(编辑器这一层).
2.function layer(渲染,动画这一层)
3.resource layer(图片,3d模型,音乐等文件的管理层)
4.core layer(数学,以及硬件资源).
5.platform layer(输入设备)

相当于游戏中的逻辑时间
tick每一帧进行一次运算.logic(逻辑),render(渲染).

# 构建游戏世界

1.动态物(比如游戏的主角和敌人)
2.静态物(建筑)
3.环境(天气,植被,地形)

游戏的中的object一般用:
name
property
behavior
三者描述

功能去做模块化,而不是面向对象去描述实体.

基于component的流水线tick,批处理加速计算.
基于事件的处理,object直接不直接进行交互,而是生成事件发送给对应的object,object处理接收到的事件
改变自身的属性的和行为.

基于多叉树的场景管理.防止过多计算(
说白了就是基于业务,只计算与当前object相关的object,这种相关可以基于逻辑距离,也可以基于因果关系)

为了多线程使用中保证顺序,最终会写成类似生产者消费者的模式,消息最终会使用一个类似队列的东西集中管理.

# 游戏引擎中的渲染实战

simt在simd的基础上再扩展多核心,使计算效率变成simd*核数

渲染的时候一个mesh由多个三角形构成,一个三角形由三个顶点构成,一个三角形的顶点可能是另一个三角形的顶点,所以我们可以将顶点存在一个数组里面,达到顶点的复用
这就是vertex buffer.

shader就是mesh和材质的的组合.我要渲染一个面,这个面要渲染成虎皮花纹的

但是一个大的mesh可能由不同的mesh组成,所以就有了submesh的概念,每个submesh有自己的shader和texture.

给mesh换材质需要进行一次gpu的通信,所以游戏里经常给人换皮的会卡一下,这相当于给一个mesh重新着色.一种优化就是将所有的mesh通过素材grouping这样,就可以通过一个通信重新渲染所有
同样的mesh(批处理的思维).

只绘制摄像机可见范围内的画面
包围盒的概念:因为一个物体可能有几万个面,通过每一个面的计算去判断有没有碰撞或者击中效率太低了,所以用一个大致贴合物体且面更少的盒来包围物体,即命中盒子就命中了物体.

# 渲染中光和材质的数学魔法

环境贴图简化的光反射的计算.
基本的shading = simple light(简化的光源,点光,方向光,圆锥光等等)+ambient(除去主光外的物体表面光场的平均值)+Environment map(环境贴图)
计算阴影,光源积分和视线积分的交集就是阴影.
Blinn-Phong模型=Ambient（环境）+ Diffuse（漫反射）+ Specular（高光）.缺点能量不守衡,出射光可能大于入射光.表现不佳,有一种塑料感




