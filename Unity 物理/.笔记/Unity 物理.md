# Bounds(包围盒)

包围盒算法是一种求解离散点集最优包围空间的方法

用体积稍大且特性简单的几何体称为包围盒来近似地代替复杂的几何对象



# 包围盒分类

## AABB包围盒 - Axis-aligned bounding box 轴对齐边框

应用最早的包围盒

包含该对象且边平行于坐标轴的最小六面体（六个标量）

构造比较简单存储空间小但紧密性差

尤其对不规则几何形体冗余空间很大

当对象旋转时无法对其进行相应的旋转

不适合包含软体变形的复杂的虚拟环境情况



## 包围球 - Sphere

包含该对象的最小的球体





## 方向包围盒OBB - Oriented bounding box

常见的包围盒类型

最大特点：方向的任意性这使得它可以根据被包围对象的形状特点尽可能紧密的包围对象

缺点：相交测试变得复杂





## FDH 固定方向凸包 - Fixed directions hulls或k-DOP

继承了AABB简单性的特点，但其要具备良好的空间紧密度必须使用足够多的固定方向

被定义为包含该对象且它的所有面的法向量都取自一个固定的方向k个向量集合的凸包

FDH比其他包围体更紧密地包围原物体创建的层次树也就有更少的节点

求交检测时就会减少更多的冗余计算但相互间的求交运算较为复杂





# 包围盒的选择

对于以60pfs运行的游戏来说处理每一帧数据的时间只有0.0167s左右

对于不同的游戏碰撞检测大概需要占10～30%的时间

也就是说所有碰撞检测必须在0.002～0.005s内完成非常巨大的挑战。



1. 快速的碰撞检测
2. 能紧密覆盖所包围的对象
3. 包围盒应该非常容易计算
4. 能方便的旋转和变换坐标
5. 低内存占用。



AABB快只需要6条逻辑比较指令

用多个AABB来近似OBB或者使用代价相对较低的Vertical-agliened OB





# Unity 中的包围盒

Unity用Bounds这个**结构体struct**来描述AABB包围盒

获取一个物体AABB包围盒的API有三种 `RenderColliderMesh`

- Render`GetComponent<Renderer>().bounds`—世界坐标

- Collider`GetComponent<Collider>().bounds`—世界坐标

- Mesh`GetComponent<MeshFilter>().bounds`—本地坐标

  

把Mesh.bounds本地坐标换算成世界坐标bounds

```csharp
var centerPoint = transform.TransformPoint(bounds.center);
Bounds newBounds = new Bounds(centerPoint, bounds.size);
```

不管是2D还是3D碰撞以及精灵和都是有`bounds`属性



公共属性：

- center边界盒的中心世界坐标

- extents边界框的范围总是size的一半

- max世界坐标边界盒的最大点这个值总是等于center+extents

- min世界坐标边界盒的最小点这个值总是等于center-extents

- size边界盒的总大小。

  

```csharp
Bounds bounds = this.GetComponent<Collider>().bounds;
Debug.DrawLine(bounds.center, bounds.center + bounds.extents, Color.red);
```

这个向量的长度是从中心点到右上角的长度。
由于`extents`是`size`的一半所以我们这样画`size`

```csharp
//得到左下角的位置
Vector3 p1 = bounds.center - bounds.extents;
Debug.DrawLine(p1, p1 + bounds.size, Color.green);
```

`size`的长度刚好是从左下角到右上角的长度。
然后我们分别画出最小值和最大值和中心点的连线。

```csharp
Debug.DrawLine(bounds.center,  bounds.min, Color.gray);
Debug.DrawLine(bounds.center, bounds.max, Color.cyan);
```

最小值在左下角最大值在右上角



公共函数：

- Encapsulate重新计算最大最小点
- Contains可判断点是否包含在边界框内世界坐标如我们需判断你是否点击了某个精灵则可以用`Contains()`
- SetMinMax设置边界盒的最小最大值
- SqrDistance点和该边界盒之间的最小平方距离
- IntersectRay射线与改边界盒相交吗
- Intersects与另一个边界相交吗比如我们需要判断两个精灵是否有重叠在一起则就可以使用`Intersects()`。



## 旋转对Bounds的影响

```csharp
//后左下角
Vector3 backBottomLeft = bounds.min;
///后右下角
Vector3 backBottomRight = backBottomLeft + new Vector3(bounds.size.x, 0, 0);
///前左下角
Vector3 forwardBottomLeft = backBottomLeft + new Vector3(0, 0, bounds.size.z);
///前右下角
Vector3 forwardBottomRight = backBottomLeft + new Vector3(bounds.size.x, 0, bounds.size.z);
///后右上角
Vector3 backTopRight = backBottomLeft + new Vector3(bounds.size.x, bounds.size.y, 0);
///前左上角
Vector3 forwardTopLeft = backBottomLeft + new Vector3(0, bounds.size.y, bounds.size.z);
///后左上角
Vector3 backTopLeft = backBottomLeft + new Vector3(0, bounds.size.y, 0);
///前右上角
Vector3 forwardTopRight = bounds.max;

Debug.DrawLine(bounds.min, backBottomRight, Color.red);
Debug.DrawLine(backBottomRight, forwardBottomRight, Color.red);
Debug.DrawLine(forwardBottomRight, forwardBottomLeft, Color.red);
Debug.DrawLine(forwardBottomLeft, bounds.min, Color.red);

Debug.DrawLine(bounds.min, backTopLeft, Color.red);
Debug.DrawLine(backBottomRight, backTopRight, Color.red);
Debug.DrawLine(forwardBottomRight, bounds.max, Color.red);
Debug.DrawLine(forwardBottomLeft, forwardTopLeft, Color.red);

Debug.DrawLine(backTopRight, backTopLeft, Color.red);
Debug.DrawLine(backTopLeft, forwardTopLeft, Color.red);
Debug.DrawLine(forwardTopLeft, bounds.max, Color.red);
Debug.DrawLine(bounds.max, backTopRight, Color.red);
```

**没有随着小方块旋转但是它仍然完全包裹着小方块**

**是与世界坐标轴对齐完全包围的对象是它自身的预制体**



# Bounds和碰撞器Collider的区别

- 碰撞器Collider的方框始终跟着模型旋转移动缩放跟着模型的只要模型不缩放它也不缩放。
  属于obb包围盒他是**有向的检测精度较好**。
- Bounds跟随模型移动而不会跟模型着旋转而是随着模型旋转而缩放变大变小始终包裹模型。
  属于aabb包围盒他是**无向的检测精度较差**。

