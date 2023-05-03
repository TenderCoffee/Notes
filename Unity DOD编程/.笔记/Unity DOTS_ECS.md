# 各数据类型存储的字节数

整型：

byte:1个字节 8位 -128~127

short ：2个字节 16位 (-2^15~2^15-1)

**int ：4个字节 32位 (-2^31~2^31-1)**

long：8个字节 64位 (-2^63~2^63-1)

浮点型：

**float：4个字节 32 位**

double ：8个字节 64位

【注】像java等默认的是double类型，如3.14是double类型的；3.14f（即加后缀f），为float类型

char型：

char：2个字节。所以一个char类型的可以存储一个汉字。

boolean型：具体分析，不同的虚拟机实现1个字节、4个字节都是有可能的



# CPU 缓存友好

CPU 访问内存 的速度是最慢的

访问速度：L1 > L2 > L3 > RAM 

容量大小：L1 < L2 < L3 < RAM 

![](C:\Users\TenderCoffee\Desktop\Unity DOD编程\.笔记\images\缓存友好.png)



# 缓存行（Cache line）常见64个字节（2的整数幂个连续字节）

缓存是由若干缓存行组成，每个缓存行大概占用64字节

比如：一个角色：坐标 Vector3(x,y,z) + 旋转四元数 Quaternion(x,y,z,w)  = 7个浮点数

1个浮点数占4字节 

4*7=28个字节

**64 - 28 = 36：有36个字节的缓存是完全浪费的**

![](C:\Users\TenderCoffee\Desktop\Unity DOD编程\.笔记\images\缓存行.png)

**这些浪费的内存会造成缓存命中率直线下降**



缓存不命中的原因：

1、传统游戏对象排序问题

传统的游戏对象在内存中的排列方式：内存排列凌乱无序

游戏对象1 - 游戏组件1 - 游戏组件2 - 游戏对象2 - ...

假如：

​	有一个游戏对象身上挂了1024个脚本，每个脚本上都有一个长度为1024的int 数组，如果忽略游戏对象本身内存和托管堆指针内存，那么 4字节×1024×1024 = 4MB

​	比如手机缓存是 4MB，那么当每个游戏对象加载后，全部都会被占满，再试图加载其他对象缓存都不会命中



2、无用数据也加载到了缓存

比如：

​	public Transform t;

​	这里有可能只用到了 Transform 中的 Position 坐标，其他的比如旋转数据都没有用到，但是整个Transform对象全部都被加载到了缓存中，而缓存的容量有限

​	在 检视面板中 public 关联游戏对象，代价是将整个关联的游戏对象全部进行了缓存





# ECS（优化CPU缓存行）相同组件在内存中都排列在一起，让缓存更容易命中内存

**要求所有相同组件在内存中都排列在一起**

如：1024个怪物，写了个一个组件只保存坐标XY 和 旋转Y，所以 每个怪物 3*4字节=12字节

​	1024*12b = 12KB，在内存中都是连续的保存在一起

​	因为每个缓存行是64个字节，由于是连续的，所以可以同时保存 64KB / 12KB = 5 个怪物 + 浪费4个字节

​	这样就大幅度利用了原本浪费掉的内存

仅优化缓存可以提升10倍的效率



# 18版本 ECS



旧版本：18版本 - 需要 PackageManager 安装 Entitas、Mathematics、Hybrid Render、Jobs、Burst

1、组件

2、系统

3、Entity管理器



RotationCubeComponent.cs 组件 - **Component** -- **定义变量**

​	继承 MonoBehavior

​	内部定义 Cube 的旋转速度

​	拖拽到控制的游戏对象上

```c#
public class RotationCubeComponent : MonoBehaviour
{
    //存放了变量
    public float speed;
}
```



RotationCubeSystem.cs 系统 - **System** - **决定逻辑行为**

​	继承 **ComponentSystem 抽象类**（引入 Unity.Entities 命名空间）

 1. 需要定义一个 Group 的 **结构体**

    ```c#
    struct Group
    {
        //一个 Entity 可以包含多个 Component
        
        //RotationCubeComponent 组件
        public RotationCubeComponent rotation;
        //Transform 组件
        public Transform transform;
    }
    ```

    

 2. 重写 OnUpdate() ，定义 foreach 循环，使 Cube 旋转

   ```c#
   protected override void OnUpdate()
   {
       //将原本应该在 MonoBehavior 中的 Update 放到了 System 中更新
       //2019 GetEntities 被废弃掉了
       foreach(var en in GetEntities<Group>())
       {
           en.transform.Rotate(Vector3.down, en.rotation.speed * Time.delatTime);
       }
   }
   ```

   

给游戏对象添加 Unity.Entities 组件 GameObjectEntity.cs - **Entity** -- **连接 C 和 S**

层级视图中的对象添加 Entities 内置控制脚本

**管理脚本：把 组件 和 系统 结合起来**

直接运行即可



# 19版本 ECS - 混合ECS开发模式



最新版本：Unity 2019版本

在 PackageManager - 需要 Package Manager 安装 Entitas、Mathematics、Hybrid Render、Jobs、Burst



## 组件 - Component - 没有逻辑 用于定义修改数据

RotationCubeComponent.cs 文件

结构体（值类型，没有GC），不能有逻辑

值类型数据在进行拷贝时候的代价较高，只要超出作用域后系统可以自己回收

引用类型数据拷贝相对容易但是需要GC垃圾回收



继承 **IComponentData** 接口 

```c#
using Unity.Entities;

public struct RotateCubeComponent : IComponentData
{
    //实体旋转的速度
    public float Speed;
}
```



游戏脚本转组件，可以把Component 挂载在对象上，可以修改数据

```c#

[GenerateAuthoringComponent]
public class MyComponent : IComponent
{
    public int a;
    public string b;
}

```



把GameObject 在运行期间转换成 Component组件

```c#

public class SpawnerAuthoring_FromEntity : MonoBehaviour, IDeclareReferencedPrefabs, IConvertGameObjectToEntity
{
    public GameObject Prefab;
    public int CountX;
    public int CountY;
    
    //Prefab 转换系统，将Prefab 转换成 Entity
    public void DeclareRefercencedPrefabs(List<GameObject> referencedPrefabs)
    {
        refercencedPrefabs.Add(Prefab);
    }
    
    public void Convert(Entity entity, EntityManager dstManager, GameObjectConversionSystem conversionSystem)
    {
        //创建Entity
        var spawnerData = new Spawner_FromEntity
        {
            Prefab = conversionSystem.GetPrimaryEntity(Prefab),
            CountX = CountX,
            CountY = CountY
        };
        //给 Entity 添加组件
        dstManager.AddComponentData(entity, spawnerData);
	}
    
    
}



```



### 共享组件

坐标值、旋转值、缩放值，几乎每个角色都不一样，比较适合值类型结构体

材质、网格 很多模型都是完全一样的，不适合结构体，引用类型 不能写到通用组件中

![](images\共享组件.png)



实体组件和共享组件的区别：

IComponentData 和 ISharedComponentData

IComponentData ：值类型

ISharedComponentData：引用类型

```c#
public struct RenderMesh : ISharedComponentData, IEquatable<RendererMsh>
{
    public Mesh mesh;
    public Material material;
    public bool Equals(RendererMesh other)
    {
        return mesh == other.mesh && material == other.material;
    }
    public override int GetHashCode()
    {
        int hash = 0;
        if(!ReferenceEquals(mesh, null)) hash ^= mesh.GetHashCode();
        if(!ReferenceEquals(material, null)) hash ^= material.GetHashCode();
        return hash;
	}
}
```



## 系统 - System - OnUpdate 中使用实体遍历获取组件数据

继承 **ComponentSystem** 抽象类

实现 **OnUpdate** 抽象方法

使用 **Entities.ForEach()** 遍历更新所有“实体”中的组件（即 Component）

不关心Entity，首先找到自己关心的 Component（如 实体上的组件），进行遍历更新

```c#
using Unity.Entities;
using Unity.Mathematic;
using Unity.Transforms;

public class RotateCubeSystem : ComponentSystem
{
    protected override void OnUpdate()
    {
        //这里的 Rotation 是 Unity.Transforms 下的
        Entities.ForEach((ref RotationCubComponent rotationSpeed, ref Rotation rotation) =>{
            //math 是 Unity.Mathematic 下的
			rotation.Value = math.mul(math.normalize(rotation.Value), quaternion.AxisAngle(math.up() , rotationSpeed.Speed * Time.deltaTime));
        });
        
        Entities.WithName("RotationSpeedSystem_ForEach")
            //找到必须同时有 Rotation和RotationSpeed_ForEach组件 的实体
            //ref=可写可读 in=只读
            .ForEach((ref Rotation rotation, in RotationSpeed_ForEach rotationSpeed) =>
                     {
                         rotation.Value = math.mul(
                         	math.normalize(rotation.Value),
                             quaternion.AxisAngle(math.up(), rotationSpeed.RadiansPerSecond * deltaTime)
                         );
                     })
            //并行执行
            .ScheduleParallel();
    }
}
```



Unity.Transforms

Translation 负责处理物体移动的视图组件

WriteGroup 负责将 Translation 和 LocalToWorld 和 LocalToParent 组件关联起来，迫使关心 LocalToWorld 和 LocalToParent 组件的系统不再执行

```c#
namespace Unity.Transforms
{
    [Serializable]
    [WriteGroup(typeof(LocalToWorld))]
    [WriteGroup(typeof(LocalToParent))]
    public struct Translation : IComponentData
    {
        public float3 Value;
    }
}
```



每个System 可以去找自己关心的组件，System 不关心 Entity，只关心 Component

![](images\ECS System.png)





## 实体控制器 - Entity -  管理 CS - 将游戏对象转换为实体并创建组件

EntitiesManager.cs 文件

继承 MonoBehavior，实体管理器 脚本

其实是个 整型 Int

混合 ECS 开发模式：都是纯粹的 ECS 实体 而不是对象

1. 实现接口： **IConvertGameObjectToEntity**：通过  Convert 方法把游戏对象转换成实体
2. 在类的前面加入 **[RequiresEntityConversion]** 标记，即本脚本所挂载的对象上必须加入 **ConvertToEntity.cs** 的系统脚本
3. 必须实现的 **IConvertGameObjectToEntity.Convert** 方法中实现 EntityManger 实例，把所有组件加入到实体中（即： AddComponentData() 方法）

```c#
using Unity.Entities;

[RequiresEntityConversion]
public class EntitiesManager : MonoBehaviour, IConvertGameObjectToEntity
{
    //cube 的旋转速度
    public float FloatCubSpeed = 10f;
    
    void IConvertGameObjectToEntity.Convert(Entity entity, EntityManager dstManager, GameObjectConvert convert)
    {
        //通过  Convert 方法把游戏对象转换成实体
        
        //创建组件
        RotationCubeComponent data = new RotationCubeComponent{ Speed = FloatCubSpeed };
        //组件加入 EntityManager 中 让Unity 内置的环境实体管理器 进行管理
        dstManager.AddComponentData(entity, data);
        
        
    }
}
```

给游戏对象挂载组件 Entities Manager.cs 和 ConvertToEntity.cs



**注意：ECS 将游戏对象 转换为 实体后，Hierarchy 面板不会出现游戏对象**

需要打开实体面板 **Window - Analysis - Entity Debugger**

![](images\ECS Entity Debugger.png)



### ArcheType - 原型

即使不同的实体 Entity，但是只要 **组件** 相同，都会保存在原型 ArcheType 中

**ArcheType 是数组容器**，长度是固定的16KB，如果一个组件占16字节，那么就能够保存1024个这样的组件

如果16KB的容量不够会再开一个 ArcheType 并且依然连续保存





# ECS + Job System

假设游戏中有成千上万的树，石头，敌人、大敌人 等等

树、石头 作为场景中的元素不需要关心

敌人、大敌人 战斗角色 需要关心



传统方式：在主线程中一个个去查找 和 更新每个敌人，速度慢

ECS：原型块的容量是16KB，包含了所有的实体，Query 出符合条件的**实体组件**，大敌人、小敌人统一 Update，**Query** 是多线程查找速度快，**Update** 是多线程更新速度快



