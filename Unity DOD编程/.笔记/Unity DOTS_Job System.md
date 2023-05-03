# Job System：充分利用多核的优势



修改前代码：

在Update中 

1、更新设置目标方向

2、渲染目标

3、设置位置



当需要使用 for 循环计算一些复杂的运算（比如数学求平方根）

帧率会发生严重的降低



目标：稳定在60帧 = 每帧16.6ms

即：必须在短时间内完成所有的逻辑与所有的渲染

如果花费时间长，将会导致帧率下降



Unity：在单线程中运行所有的更新逻辑

每个游戏对象的 Update 逻辑都会被一个接一个的调用



JobSystem：利用可用的核心，在多线程上高效运行 逻辑 和 Job

即：创建一个包含所有游戏对象逻辑的 Job = 并行处理多个对象 = 提高性能



# 实现 Job System

复制出2份原本的对象更新逻辑的代码文件

## 1、Job 文件 - 包含对象的所有逻辑的Job

```c#
//引入命名空间
using Unity.Jobs;
using Unity.Mathematics;
using Unity.Collections;
```

​	1、不继承 MonoBehavior ，而是继承 IJob

​	2、不使用 class，而改为使用 结构体 struct

​	3、将原来的 Update 方法 改为  Execute 方法，并设置为 public

​	4、通过构造函数递参数，让 Job 执行的时候，拥有之前的逻辑数据

​		注意：

​			1、Job 无法使用任何 引用类型 - 无法使用 Camera、Transform 组件

​			2、Job 是无状态 =  相互独立的 = 无法在帧之间保留例如 方向一类的 值

​			3、Job 无法访问 Time.deltaTime ，因此需要通过 构造函数 外部传入 float 值去使用

​			4、Job 不允许使用 Random 方法，需要外部传入随机种子，使用 Unity.Mathematics.Random 去用随机种子进行随机，random.Range 修改为 random.NextFloat

​	5、使用 NativeArray  将 Job 里计算后的值传出去给主线程访问

​			1、在构造函数中 设置 NativeArray  传入

​			2、设置值：给 NativeArray  数组索引为0 的位置设置值



## 2、驱动 Job 文件 - 用于安排 Job 在另一个线程中运行

```
//引入命名空间
using Unity.Jobs;
using Unity.Collections;
```

1、删除 Update 中原来的逻辑，

2、在Update 中 创建新的 Job 实例 来代替执行逻辑

3、在 Job的构造函数 中 设置参数

​		随机数种子：(uint)Random.Range(1,1000) 

​		记得将 NativeArray 通过构造函数传入

```c#
//第1个参数 = 需要数组包含多少个值
//第2个参数 = 设置持久性 Allocator.Persistent = 可以跨多个帧使用数组
NativeArray<float> test = new NativeArray<float>(1, Allocator.Persistent);
```

4、创建完成实例后 安排 Job 作业

```
//返回一个 JobHandle 可以跟踪 Job
JobHandle handle = job.Schedule();
```

5、因为 Job 中有返回结果给主线程，所以需要在 Schedule 后等待 Job 完成

​	让 Job 完成每一帧更新

​	在 LateUpdate 中调用 JobHandle.Complete  完成

​	这样执行后，所有实例的Job 都会被安排在 Update 周期中，并且每个实例都会在 LateUpdate 中等待完成

6、在  JobHandle.Complete 后面开始获取 Job 中的返回数据

```
//得到Job中的 返回的使用val值
float val = test[0];
```

7、需要在销毁游戏对象的时候，对 NativeArray  也进行销毁操作 Dispose

```c#
private void OnDestroy()
{
	test.Dispose();
}
```

8、最后将 驱动 Job 文件 挂载到 游戏对象上



## 3、Job 文件 挂载 Burst 编译指令

Burst 编译器会将代码转换为高度优化的本机代码，并且 Burst 主要用于与 Job System 一起使用

Job 文件：

```c#
//引入命名空间
using Unity.Burst;

//使用 Burst 编译
[BurstCompile]
public struct xxx: IJob
{
    
}
```



## 4、越多核 核心越充分利用 = 帧率越高





# 缺点：调度 Job 和 将值传入传出主线程都是有成本的

Job 不应该用于所有事情

**调度 Job 和 将值传入传出主线程都是有成本的** = Job 中的逻辑需要足够复杂 才能 利大于弊，成本才不算高

太简单的 Job = 管理 Job 的成本 可能 比主线程运行逻辑要更高





=======================================================================================





# 传统的 Component System

单线程。必须每个 System 的Update 走完才执行下一个



# Job Component System

![](images\Job Component System.png)

在多线程完成

在主线程分配多个 JCS（Job Component System），把任务交给子线程接着完成，不影响主线程



## Sync Point - 硬性同步点

比如后面的 Component System 用到了前面的 JCS 中的某个值，所以需要等前面所有的 Job 都执行完成



# Job System 的应用

假如 Job System 开启 16个子任务，通过 **Query** 找到感兴趣的组件，CPU 如果是有4个核心，则每个核心会分配4个任务，并行执行 。

假如某个核心任务繁重，就会造成别的核心完成后出现一直空闲等待的状态，就需要将原本任务繁重的核心里的任务，分配给那些空闲空闲下来的核心，一起执行加快处理速度。



# 使用

```c#
//Query
public class EnemyMovementSystem : JobComponentSystem
{
    private EntityQuery m_Group;
    protected override void OnCreate()
    {
        base.OnCreate();
        var query = new EntityQueryDesc
        {
            All = new []
            {
                typeof(EnemyTransformData),
                ComponentType.ReadOnly<TargetPositionData>()
            },
        };
        m_Group = GetEntityQuery(query);
    }
}

//Schedule
protected override JobHandle OnUpdate(JobHandle inputDeps)
{
    var transformDataType = GetArchetypeChunkComponentType<EnemyTransformData>();
    var targetPositionDataType = GetArchetypeChunkComponentType<TargetPositionData>(true);
    
    var updatePosAndHeadingJob = new UpdatePositionAndHeadingJob
    {
      	TransformDataType = transformDataType,
        TargetPositionDataType = targetPositionDataType,
        DeltaTime = Time.deltaTime,
        RotationLerpSpeed = 2.0f,
        MovementLerpSpeed = 4.0f,
    };
    return updatePosAndHeadingJob.Schedule(m_Group, inputDeps);
}

//Execute
public void Execute(ArchetypeChunk chunk, int chunkIndex, int firstEntityIndex)
{
    var chunkTransformData = chunk.GetNativeArray(TransformDataType);
    var chunkTargetPositionData = chunk.GetNativeArray(TargetPositionDataType);
    
    for(int i = 0; i < chunk.Count; i++)
    {
        var target = chunkTargetPositionData[i];
        var transformData = chunkTransformData[i];
        float2 toTarget = target.TargetPosition - transformData.Position;
        float distance = math.length(toTarget);
        transformData.Heading = math.select(
        	transformData.Heading,
            math.lerp(transformData.Heading,
                     math.normalize(toTarget),
                      math.mul(DeltaTime, RotationLerpSpeed)
                     ),
            distance > 0.008
        );
        
        transformData.Position = math.select(
        	target.TargetPosition,
            math.lerp(
                transformData.Position,
                target.TargetPosition,
                math.mul(DeltaTime, MovementLerpSpeed)
            ), distance <= 2
        );
        chunkTransformData[i] = transformData;
    }
    
}

```



Job 多线程计算

1、开启一个子线程计算并返回结果

2、

IJob：只开了一个核去运行多线程代码

IJobFor：会开多个核并行执行多线程代码，子线程 里 做多个数据的运算（两个数组），保证顺序

IJobParallelFor：会开多个核并行执行多线程代码。完全的多线程 Execute(int index) 不保证顺序

```c#
[BurstCompile]
public struct MyJob1 : IJob
{
    [ReadOnly] public int left;
    [ReadOnly] public int right;
    [WriteOnly] public NativeArray<int> @out;
    public void Execute()
    {
        @out[0] = left * right;
    }
}

[BurstCompile]
public struct MyJob2 : IJobFor
{
    public NativeArray<int> left;
    [ReadOnly] public NativeArray<int> right;
    public void Execute(int index)
    {
        left[index] = left[index] * right[index];
    }
}

[BurstCompile]
public struct MyJob3 : IJobParallelFor
{
    public NativeArray<int> left;
    [ReadOnly] public NativeArray<int> right;
    public void Execute(int index)
    {
        left[index] = left[index] * right[index];
    }
}


private void Start()
{
    MyJob1 myJob = new MyJob1();
    myJob.left = 2;
    myJob.right = 3;
    myJob.@out = new NativeArray<int>(1, Allocator.TempJob);
    myJob.Schedule().Complete();
    Debug.Log(myJob.@out[0]);	//6
    myJob.@out.Dispose();
    
    MyJob2 myJob2 = new MyJob2();
    myJob2.left = new NativeArray<int>(100, Allocator.TempJob);
    myJob2.right = new NativeArray<int>(100, Allocator.TempJob);
    
    MyJob3 myJob3= new MyJob3();
    myJob3.left = new NativeArray<int>(100, Allocator.TempJob);
    myJob3.right = new NativeArray<int>(100, Allocator.TempJob);
    
    //Schedule是在一个子线程中执行 可以保证顺序
    myJob.Schedule(myJob.left.Length, new JobHandle()).Complete();
    
    //ScheduleParallel 可以在多个子线程中并行运行，不保证顺序但效率更高
    myJob2.ScheduleParallel(myJob2.left.Length, 64, new JobHandle()).Complete();
    
    //完全并行执行不保证顺序
    myJob3.Schedule(myJob3.left.Length, 64).Complete();
    
    myJob.left.Dispose();
    myJob.right.Dispose();
}

```



1、 IJob、IJobFor、IJobParallelFor都提供了 Run 接口

2、Run 接口是完全在主线程中执行的，效率会大打折扣

3、Schedule 是多线程执行

4、Complete 可以实现在主线程等待执行的结果



IJobChunk - 推荐使用

1、每个 ArcheType 有 16KB，也就是一个 Chunk 块

2、使用 IJobChunk 来访问实体数据 是最灵活的

​	比如：修改了某个实体的坐标，最好的办法是只遍历被修改的这个实体的 Chunk

3、Chunk 中的每一类型的数据 都由 一个 VersionNumber 标记，如果变化了就说明数据发生了变化

```c#

[BurstCompile]
struct MyJob : IJobChunk
{
    [ReadOnly] public ArchetypeChunkComponentType<Transform> translationType;
    [ReadOnly] public ArchetypeChunkEntityType entityType;
    
    public float3 heroPos;
    public EntityComponentBuffer.Concurrent CommandBuffer;
    
    //系统的版本号
    public uint LastSystemVersion;
    
    public void Execute(ArchetypeChunk chunk, int chunkIndex, int firstEntityIndex)
    {
        //通过系统版本号和组件类型比较是否发生变化
        var translationsChanged = chunk.DidChange(translationType, LastSystemVersion);
        //如果块中的 Translation 数组没有变化则不进行遍历
        if(!translationChanged)
            return;
        var translations = chunk.GetNativeArray(translationType);
        var entities = chunk.GetNativeArray(entityType);
        for(var i = 0; i < translations.Length; i++)
        {
            if(math.distance(translations[i].Value, heroPos) <= 0.1f)
            {
                //删除实体
                CommandBuffer.DestroyEntity(chunkIndex, entities[i]);
            }
        }
        
    }
}


//Schedule
protected override JobHandle OnUpdate(JobHandle inputDeps)
{
    float3 heroPos = heroTransform.position;
    var myJob = new MyJob()
    {
        CommandBuffer = m_EndSimulationEcbSystem.CreateCommandBuffer().ToConcurrent(),
        entityType = this.GetArchetypeChunkEntityType(),
        translationType = this.GetArchetypeChunkComponentType<Translation>(true),
        heroPos = heroPos,
        LastSystemVersion = this.LastSystemVersion
    };
    
    var jobHandle = myJob.Schedule(m_Query, inputDeps);
    m_EndSimulationEcbSystem.AddJobHandleForProducer(jobHandle);
    return jobHandle;
    
}

```





## Job 的处理依赖关系

设置依赖后，不需要在主线程等待



```c#
MyJob2 myJob1 = new MyJob2();
MyJob3 myJob2 = new MyJob3();

//Job1 和 Job2 同时并行执行
myJob1.Schedule(100, 64);
myJob2.Schedule(100, 64);

//Job1执行完毕后再并行执行Job2
//缺点是要在主线程 等 Job1 结束
myJob1.Schedule(100, 64).Complete();
myJob2.Schedule(100, 64);

//设置 Job2 依赖 Job1，这样就不需要在主线程等待
JobHandle jobHandle = new JobHandle();
JobHandle scheduleJobDependencyJob = myJob1.Schedule(100, 64, jobHandle);
myJob2.Schedule(100, 64, scheduleJobDependencyJob).Complete();

```



**Job 的多线程，只能处理数据**



# Hybrid Renderer - 多线程游戏对象

PackageManager添加 包

绑定 ConvertToEntity 组件：可以将游戏对象动态 动态转换为实体对象

​	Convert And Destroy：转换后会把对象删除并且只保留Entity对象



Prefab动态转成实体对象：

代码中引用了Prefab，在运行时动态转换成实体对象

```c#

[AddComponentMenu("DOTS Samples/SpawnFromMonoBehaviour/Spawner")]
public class Spawner_FromMonoBehaviour : MonoBehaviour
{
    public GameObject Prefab;
    public int CountX = 100;
    public int COuntY = 100;
	void Start()
    {
        var settings = GameObjectConvertsionSettings.FromWorld(World.DefaultGameObjectInjectionWorld, null);
        var prefab = GameObjectConversionUtility.ConvertGameObjectHierachy(Prefab, settings);
        var entityManager = World.DefaultGameObjectInjectionWorld.EntityManager;
        
        for(var x = 0; x < CountX; x++)
        {
        	for(var y = 0; y < CountY; y++ )
            {
                var instance = entityManager.Instantiate(prefab);
                var position = transform.TransformPoint(
                    new float3(
                        x * 1.3F, 
                        noise.cnoise(new float2(x, y) * 0.21F) * 2,
                        0
                );
                entityManager.SetComponentData(instance, new Translation{Value = position});
			}
        }
    }
}

```



给空对象绑定 绑定组件：

空对象添加渲染器：RenderMeshProxy 组件

位置：Translation-Deprecated 组件

旋转：Rotation-Deprecated 组件

缩放：NonUniformScaleProxy-Deprecated 组件



运行时将整个场景转换成 DOTS：

用于美术制作了成功场景后

选中场景中的对象，右键 New SubScene From Selection，转换成功后 在 Sub Scene 组件中勾选 Auto Load Scene

 









