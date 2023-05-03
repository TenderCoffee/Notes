# DrawCall 概念

遵循 Camera 渲染原理  和  GPU 的工作原理

引擎在 CPU 端将模型信息传递给 GPU 进行渲染

  

## DrawCall 过高的性能瓶颈 - CPU 端

DrawCall ： CPU 调用图像编程接口

造成 DrawCall 过高的性能瓶颈 不是在GPU端 而是 **在CPU端** 

CPU 和 GPU 并行操作

CPU 要渲染对象的时候向命令缓冲区添加命令，当 GPU 完成上一次渲染任务后就会从命令队列中取出一个命令执行   



## Unity（UGUI）的 合批原理

**将相同的 Material （参数也需要一样，如 Shader 和 Texture）节点 进行 Mesh 的 Combine 操作**

UGUI 减少 DrawCall 的方式：尽可能的动态合批

**注意：合批是需要很大代价的**

合并 Mesh 之后，如果对 Renderer 进行设置执行 Render.material 会 new 出一个新的 Material ，这个 Material 会包含合并后的 Texture 信息



# UGUI DrawCall 计算

**深度值排序 => 材质ID 排序 => 图片 ID 排序 => RendererOrder 排序**



## 原则

1、以 Canvas 为单位， canvas.alpha = 0 时 不渲染

2、Canvas 可以嵌套 Canvas



## 合批之前



### 1、按照 Hierarchy 动态计算 Depth - 基本确定 DrawCall 的值

1、深度优先原则：

按照 Hierarchy 节点的顺序，从上向下进行 Depth 分析

2、跳过不渲染的节点

active = false、 Image 脚本 Disable、Canvas 不渲染的 Layer 等

3、

**如果处于渲染状态且没有任何其他渲染元素跟它相交，depth = 0**

是否相交 不取决于 RectTransform 的 Rect 是否相交，而是渲染元素本身有没有重叠

![](.\images\UGUI DrawCall.png)

4、

**如果处于渲染状态且有任何其他渲染元素跟它相交**

1. **找到相交的渲染元素中 最大 Depth 值 Max Depth 的渲染 元素**
1. **判断是否能够与其 Batch（合批），如果能 Batch，则 Depth = Max Depth、否则 Depth = Max Depth + 1**



5、一个渲染元素同时盖住多个其他元素，取 Depth 值最高的元素，返回步骤4

6、元素A 盖住了 元素B，需要先计算好 元素B 的 Depth，才能计算 元素A 的Depth

**注意：Depth 的值与 是否相交有关，与是否为子节点无关**

![](.\images\UGUI DrawCall_Depth 升序排列.png)



### 2、在 Depth值相同的情况下，按照 材质球 Material 的 Instance ID 进行升序排列

注意：很多情况下 Image、Text 在没有指定纹理的情况下，都是使用的 Unity 的默认纹理： Default UI Material

所以一般情况下，节点的 Material ID 在是一样的

![](.\images\UGUI DrawCall_Material 升序排列.png)





### 3、使用纹理 Texture ID 的 Instance ID 进行升序排序

1、使用 Sprite Atlas 的图集或者使用 TexturePacker 打的图集，需要游戏运行起来，Texture ID 就是图集的 ID

2、使用 单个图片 没有图集， TextureID 就是 图片本身ID

3、使用 同个字体的 Text，Texture ID 就是 字体的ID

4、 进入到这个阶段，Mat ID 和 Depth 都相同，但是可以通过 Texture ID 影响 的节点排序

![](.\images\UGUI DrawCall_Texture 升序排列.png)



最后得到 DrawCall 数量，638/ 10138 / 10414 / 638

一共产生了 4 个 DrawCall 



## 关于 PosZ  的值不为0



如果 PosZ  的值不为0 的合批计算，节点需要满足条件才可以和别的节点合批：

父节点 PosZ 不为 0，子节点 PosZ 为 0 时，子节点满足以下条件可以可批：

1、图集一样，材质一样 等基本条件满足

2、在 Hierarchy 中是相邻的 ，可以是父子节点

3、跟 Depth 的值以及是否与别的节点是否相交 没有关系



父节点 PosZ 为0， 子节点 PosZ 不为0：

1、父节点下的 所有子节点的元素 满足基本合批条件（图集、材质一样）时，无论子节点的 PosZ 是否为0，都可以合批

2、父节点下的 所有子节点的元素 部分满足基本合批条件（部分元素的图集、材质一样），则此时合批对受 Hierarchy 的节点的顺序有所影响



## 注意

1、一个 Canvas 组件下的元素才会合批，不同 Canvas下即使 Order in layer 相同也不会合批

2、Tag、 Layer 的不同，是否添加了 outline shadow 脚本都不会影响合批

3、不使用 UnityWhite 等默认的 Unity 的图片

4、有时候为了合并层级，需要在 Text 下面放一个其他图集中的透明图片，将 Text 垫高层级，层级整齐可以与其他的 Text 层级相同从而合批

5、一个节点的 RectTransform 的 Value 的值不同 不会影响合批



# Mask 和 RectMask2D

裁剪的原理 和 计算 DrawCall 的方式完全不同



## Mask 的 合批规则 - GPU 实现

1、Mask 内的元素 不会和 Mask外的元素合批

2、Mask 内的元素 可以跟其他 Mask 内的元素进行合批

3、Mask 计算的时候 会在节点的前后生成2个 Mask 的 DrawCall 



Mask 的 2个 Mask 原因 - GPU 的 Shader 实现

第一个 Mask 会在底层模版绘制一个区域的指令，根据 Image 传进来的图片的 Alpha 值 确定 裁剪区域，之后 Mask 节点下的元素会根据这个区域计算 Alpha 的值

第二个 Mask 绘制区域结束的指令，用于结束计算裁剪的操作



### 注意

尽量让 Mask 放到一个 Canvas 下，使其不影响外面的正常的合批



## RectMask2D 的 合批规则 - CPU 实现

1、RectMask2D 的合批规则 和 正常的合批规则是一样的

2、RectMask2D 不会与 RectMask2D 外的其他任何元素合批，也不会和 其他 RectMask2D 内的元素合批

3、RectMask2D 不会和 Mask 一样产生额外的 DrawCall



RectMask2D 不需要 Image 的图片作为裁剪区域，所以裁剪区域永远是 矩形大小

进而 CPU 计算元素是否在矩形区域内，

如果在区域内，则会将节点下的常规合批，之后进行顶点裁剪

如果不在区域内，则不会被渲染

因此 当一个元素完全脱离 RectMask2D 矩形区域之外，这个节点的 DrawCall 会变为 0 





# 影响性能更大的元凶 - Rebuild



Canvas：

​		将子节点的UI元素的网格合并，生成渲染命令发送到 Unity 的图形管道。**UI 变化 执行一次 Batch**，给 GPU 进行渲染，一次 Batch 只会影响子节点，不会影响 子Canvas。



CanvasRenderer：

​		提供了 网格 数据 给 Canvas 使用，但是不包括 子Canvas



Rebuild：

​		重新计算 Graphic 组件的布局和网格情况 的过程



## Rebuild 分类1 - Layout Rebuilds

Layout Rebuilds  怎么触发

1、计算一个 Layout 组件子节点的适当位置，或者 可能的大小

​	HorizonLayoutGroup

2、LayoutGroup 在 以下情况 标记 SetDirty ，再触发 Rebuild

​	OnEnable

​	OnDidApplyAnimationProperties

​	OnRectTransformDimensionsChange

​	OnTransformChildrenChanged

3、如果 LayoutGroup 的直接子节点是 Graphic 类型（Text 和 Image），被 SetLayoutDirty 的时候，LayoutGroup 也会被加入到 Rebuild 的队列中



## Rebuild 分类2 - Graphic Rebuilds

缩放、旋转、文字变化、图片修改 都会触发 Graphic 中 的

SetAllDirty、SetLayoutDirty、SetVerticesDirty、SetMaterialDirty 都会触发 Rebuild



UGUI 会在一帧中手机所有的修改然后统一进行处理



## 优化

1、Cavnas 动静分离：

加载进来之后一直不变的 Graphic 放进一个 Canvas

一直变化（跑马灯、有 Animator动画效果的） 放到 另外一个 Canvas

增加了 DrawCall 但是降低了 Rebuild 的更大伤害

2、尽量不对 Canvas 的子节点 进行删除/增加 显示/隐藏

3、尽量不对 Canvas 的子节点 进行 Vertex、Rect、Color、Material、Texture 的改变 - 如果需要就使用 动静分离

4、不要设计 UI 元素数目过多或者层次结构过于复杂的节点结构，对 Batch 更新速度的影响很大



# UI 粒子特效

本身粒子特效不归 Canvas 管理，不会影响 UI 的 DrawCall

但如果特效 勾选了 **Render 属性** 则特效会有 Order In Layer 的概念，会与 Canvas 的 Order 进行混合影响显示层级



## 优化

**尽量简单，尽量用序列帧动画实现**



# Graphic 的 Raycast Target



## 优化

1、Graphic 的 Raycast Target 会迭代所有 Raycast 

2、不需要交互就取消勾选

3、不使用 富文本的 Text 则取消勾选 RichText

4、尽量不使用 Best Fit



# Overdraw - 多次深度测试 - 手机发热的主要元凶

Overdraw是指**屏幕上的某个像素在同一帧的时间内被绘制了多次**

过多Overdraw 引起GPU过载，影响动画的播放和界面响应速度

 GPU 渲染 **大量的粒子特效和透明材质的多层叠加覆盖** 的时候，**需要进行 多次的深度测试**



## 优化

1、避免短时间内大量特效产生

2、让特效的产生在屏幕上平均一些

3、避免使用内置的 Outline、Shadow 的组件，使用 TextMeshPro 替换

4、直接将特效渲染到 MaskableGraphic 中，以 UI 的方式展示



## 影响 Overdraw 的组件

Image：的 九宫格 的 Fill center

Mask：自带两层 Overdraw .

RectMask 2D：有一层 Overdraw，如果有一个完全不显示 那么完全没有overdraw，比 Mask 组件好

Text： OutLine 和 Shadow，

​		Shadow会增加一层 OverDraw，

​		OutLine是复制了四份Shadow实现的 = 4层OverDraw

​		替换为 Text Mesh Pro 插件

​		字体在更新的时候不要做成每帧都更新

​		



# 粒子特效 与  UGUI 的层级处理

 如果将粒子挂到 Image 上面 或者 Image 下面



## 方案1 - 修改 粒子效果的 Renderer 中的 Order In Layer

Canvas 的Order 默认为 0 

缺点：

​		需要手动设置每个粒子的 Order

优点：

​		可以实现一个 Prefab 有多个层级的粒子



## 方案2 - SortingGroup

在 Prefab 根节点上挂载 Sorting Group 然后设置 Order In Layer

即使设置了 粒子效果 的 Order In Layer 多大的值也无用，因为被 Sorting Group 强制控制

缺点：

​		如果需要实现一个预设的两个特效，则需要2个Prefab

优点：

​		特效制作简单不需要关心粒子的层级



## 方案3 - 滚动列表中的粒子特效 - 使用 UGUIParticleMask



 UGUIParticleMask ：

​		获取 RectMask2D 或者 Mask 的 RectTransform，然后调用 GetWorldCorners 获得该 UI 在世界坐标的信息坐标然后设置给 Shader，再让Shader 根据 Rect 坐标进行裁剪



缺点：

​		需要修改 Material

优点：

​		挂脚本方便快捷



## 方案4 - 滚动列表中的粒子特效 - 粒子挂载 UIParticles

1、将 Particle 挂上 UIParticles，设置相关的 Material 同时调整参数

2、有多少 粒子就需要挂载多少 UIParticles



UIParticles 继承了 MaskableGraphic 重写了 OnPopulateMesh 函数，模拟了粒子，将粒子转换为 UGUI 的 Graphic 融入到 UGUI 中



缺点：

​		制作会麻烦点，有多少粒子挂多少脚本

优点： 

​		可以裁剪不规则，不用手写代码设置 Order

性能：

​		特效的更新会导致UI 的 Rebuild ，不能使用太多，堆内存需要注意单个粒子系统的 MaxParticle 数量设置



# 优化

CPU的性能瓶颈往往是Drawcall

GPU的性能瓶颈往往是像素点填充率（Overdraw导致）



关注优先级应该是：Drawcall > Overdraw > 面片





Text Mesh Pro 的优点：

​	使用 了 SDF（有效距离场）渲染记录了字体的矢量形状进行渲染，渲染之前就已经把描边计算出来



Text Mesh Pro 的缺点：

​	1、写的字会对现有的资源里的贴图进行修改，贴图占据空间大

​	2、贴图发生变化，会需要将贴图重新打ab包更新，现在可以使用 创建动态字体



### 弹出窗口的遮挡

底层的窗口被上层遮挡，需要将 Camera渲染层级里移除并将不可见的Canvas设置enable = false

如果 不在Camera渲染层级里的 Canvas.enable = true，则 Canvas 下的UI 仍然会产生 OverDraw



### 图片资源本身带有大圈透明

这样大片渲染都是无效的 增加了over draw

1、可以让图片设置成 Polygon（多边形）

2、在 SpriteEditor 中设置图片的 轮廓Snap 自动生成

3、最后在使用的时候勾选 Use Sprite Mesh



### 不要使用不可见的Image 作为交互的空间

Image 不可见 但仍然占用显卡资源

解决：实现一个只在逻辑上响应Raycast但是不参与绘制的组件即可

```c#
using UnityEngine;
using System.Collections;

namespace UnityEngine.UI
{
    public class Empty4Raycast : MaskableGraphic
    {
        protected Empty4Raycast()
        {
            useLegacyMeshGeneration = false;
        }

        protected override void OnPopulateMesh(VertexHelper toFill)
        {
            toFill.Clear();
        }
    }
}
```



### 隐藏UI 的正确方法

1、**不要使用 SetActive**

​		因为 SetActive=false 的时候 会导致 Canvas 的 VBO（顶点缓冲对象） 数据失效，

​		重启 SetActive = true 的时候，会导致整个 Canvas rebuild 和 rebatch

​		对 CPU 造成 负担

2、**使用 子画布.enabled 并禁用面板逻辑**

​	推荐使用 Sub-Canvas，要添加 Canvas 组件 + Graphic Raycaster 组件

​	隐藏的时候 CanvasRenderer.enabled = false

​	注意：要禁用被隐藏的脚本逻辑，因为虽然看不到，实际上 Graphic Raycast 还是可以检测到

​				可以实现一个 manager 管理 CanvasRenderer.enabled  下的脚本行为

3、**降低 Overdraw 的方法 使用 CanvasGroup** 

​		1、全屏遮挡的时候，被遮挡的 Canvas 添加 CanvasGroup 组件 去管理这个节点以及节点下的所有子节点

​		2、被遮挡的 Canvas 在被挡住的时候设置 alpha 值= 0，这样就不会传递给 GPU 渲染 

​		





