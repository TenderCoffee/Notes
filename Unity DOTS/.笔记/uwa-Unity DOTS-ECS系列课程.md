# Dots 涉及的 C# 基础



## Blittable 类型 - 基元类型

大多数数据类型在托管和非托管内存中具有共同的表示形式，而且不需要互操作封送处理程序进行特殊处理。 这些类型称为 blittable 类型，因为它们**在托管和非托管代码之间传递时不需要进行转换。而且由于这样可以提高性能，因此应首选这些类型**。

blittable类型：

- [System.Byte](https://learn.microsoft.com/zh-cn/dotnet/api/system.byte)
- [System.SByte](https://learn.microsoft.com/zh-cn/dotnet/api/system.sbyte)
- [System.Int16](https://learn.microsoft.com/zh-cn/dotnet/api/system.int16)
- [System.UInt16](https://learn.microsoft.com/zh-cn/dotnet/api/system.uint16)
- [System.Int32](https://learn.microsoft.com/zh-cn/dotnet/api/system.int32)
- [System.UInt32](https://learn.microsoft.com/zh-cn/dotnet/api/system.uint32)
- [System.Int64](https://learn.microsoft.com/zh-cn/dotnet/api/system.int64)
- [System.UInt64](https://learn.microsoft.com/zh-cn/dotnet/api/system.uint64)
- [System.IntPtr](https://learn.microsoft.com/zh-cn/dotnet/api/system.intptr)
- [System.UIntPtr](https://learn.microsoft.com/zh-cn/dotnet/api/system.uintptr)
- [System.Single](https://learn.microsoft.com/zh-cn/dotnet/api/system.single)
- [System.Double](https://learn.microsoft.com/zh-cn/dotnet/api/system.double)

属于blittable类型的复杂类型：

blittable 基元类型的一维数组，如整数数组。 但是，包含 blittable 类型变量数组的类型本身不是 blittable 类型。
所有只包含 blittable 类型（和作为格式化类型进行封送的类）的格式化的值类型。



## non-blittable类型

在非托管环境中，某些托管数据类型要求具有不同的表示形式。 **必须将这些非 blittable 数据类型转换为可以封送的形式**。 例如，托管字符串就是非 blittable 类型，因为这些字符串必须转换为字符串对象后才能进行封送。

| 转换为 C 样式数组或 SAFEARRAY。                   |                                  |
| ---------------------------------------- | -------------------------------- |
| [System.Boolean](https://learn.microsoft.com/zh-cn/previous-versions/dotnet/netframework-4.0/t2t3725f(v=vs.100)) | 转换为 1、2 或 4 字节的值，true 表示 1 或 -1。 |
| [System.Char](https://learn.microsoft.com/zh-cn/previous-versions/dotnet/netframework-4.0/6tyybbf2(v=vs.100)) | 转换为 Unicode 或 ANSI 字符。           |
| [System.Class](https://learn.microsoft.com/zh-cn/previous-versions/dotnet/netframework-4.0/s0968xy8(v=vs.100)) | 转换为类接口。                          |
| [System.Object](https://learn.microsoft.com/zh-cn/dotnet/framework/interop/default-marshalling-for-objects) | 转换为变量或接口。                        |
| [System.Mdarray](https://learn.microsoft.com/zh-cn/dotnet/framework/interop/default-marshalling-for-arrays) | 转换为 C 样式数组或 SAFEARRAY。           |
| [System.String](https://learn.microsoft.com/zh-cn/dotnet/framework/interop/default-marshalling-for-strings) | 转换为空引用中的终止字符串或转换为 BSTR。          |
| [System.Valuetype](https://learn.microsoft.com/zh-cn/previous-versions/dotnet/netframework-4.0/0t2cwe11(v=vs.100)) | 转换为具有固定内存布局的结构。                  |
| [System.Szarray](https://learn.microsoft.com/zh-cn/dotnet/framework/interop/default-marshalling-for-arrays) | 转换为 C 样式数组或 SAFEARRAY。           |

委托是引用静态方法或类实例的数据结构，也是non-blittable类型。



## unmanaged 类型 - 非托管的类型

不涉及托管对象引用的值类型

- 14种基元类型+Decimal(decimal)
- 枚举类型
- 指针类型（比如int*， long*）
- 只包含Unmanaged类型字段的结构体




## C# 类型

**int** 值类型

**int?** 值类型 实现 null 的判定是 C# hack 了判等（==）的操作，就像 Unity  hack 了GameObject 的判等操作一样

**string** 引用类型。声明后就不能修改，被C#潜规则处理过（线程调用的时候不会有问题），用起来像值类型 Dots比较少用

**bool** 值类型（但不是 Blittable） 有些场合下 burst 不支持使用 bool，用 byte 代替 bool

**IntPtr** 值类型，Dots比较少用

**枚举 值类型**，**但是枚举作为字典 Key 的时候有消耗**

**object** 标准引用类型 注意拆转箱的问题

**int*** 值类型，指针本身是一个值

**struct 结构体，值类型， Dots常用。嵌套的结构体 都是 Blittable（纯 Struct）**

```c#
//Blittable类型的 struct
public struct NeastedStruct_Blittable
{
    //注意：只要有一个不是 Blittable 则这个结构体就不是 Blittable 类型（害群之马）
	public Vector3 vectorValue;
}

//Not Blittable 类型的 struct
//值类型 但不是 Blittable 因为里面含有一个 引用类型 的东西
public struct NeastedStruct_NotBlittable
{
    public GameObject gameObjetc;
}
```

**interface** 接口 既有可能是 class 也有可能是 struct，利用接口的时候是面向接口编程，但是拿到接口没有办法区分是 struct 还是 class，**所以 Dots 中 只会拿 interface 作为一个约束（where 限制 泛型类型）**

interface 的精髓是 函数，**在 Dots中 绝对不要用面向接口的方式去编写**，因为 Burst 不知道继承接口的是 struct 还是 class，就会一律报错，所以要把 interface 更多的当作一个 限制

```c#
public interface UnKnownInterface
{
    //注意：这种设计在 Dots 上被拒绝
    //不是 C# 层面上的错误 而是在 Unity中的编译错误
    //所以 不要用这种函数的方式去编写
	string GetName();
}
```

**在 Dots 下 接口的正确用法：**

拿一个容器，往里面装 class 和 struct

```c#
public List<UnKnownInterface> unKnownInterfaceInstances = new List<UnKnownInterface>()
{
	new ClassUnKnownInterface(),
	new StructUnKnownInterface(),
};
```



## Dots 中的类型



### **NativeContainer **

**要求 T 必须为 unmanaged**

unmanaged 是 泛型约束的一种，要求 “不可以为 null 值的类型”，即 编译错误会提示 xxx 必须是 不可以为 null 值的类型

**泛型 T 需要用 unmanaged 类型描述**



**注意：这里 不推荐 下面的这种 扩展方法（HybridCLR 会有错误）**

```c#
public static class ArrayHelper
{
    //不推荐这种 扩展方法
    //原因： IL2CPP 有bug，IL2CPP中 obj.Func() 非虚调用不符合规范
    //导致 array Dispose 的话，在 Editor 模式下虽然会正常运行 但是在真机上 IL2CPP 会报错
    public static void DoHelp<T>(this NativeArray<T> array) where T : unmanaged
    {
        
	}
}
```

因为 不可以为 null 值的类型，这样声明会在编译的时候报错

```c#
//这样会编译报错
public NativeArray<int?> nativeArrayNullableInt;
```

GameObject 不是一个 unmanaged 类型，这样声明也会报错

```c#
//这样会编译报错
public NativeArray<GameObject> nativeArrayGameObject;
```



### Job

**Job 必须是一个 值类型 才可以使用，不可以使用 Class**

```c#
//必须使用 struct 去继承 Job
public struct TestJob : IJob
{
    //在 Job 中必须要用 纯的值类型 所以这里不可以这么写
    //public Dictionary<int, int> dicParam;    
    public NativeArray<int> intArrayParam;
    
    //注意： 在这里的 bool 也可以使用，比较特殊 只是 Burst在静态函数的情况下比较抗拒使用 bool
    public bool t;
    
    void IJob.Execute(){}
}

//注意：下面这种方式会在运行的时候报错 不可以用 class
//public class TestClassJob : IJob
//{
//    void IJob.Execute(){}
//}

```



## 托管内存 和 非托管内存

GC：C# 的 垃圾回收

托管：把使用内存的操作 交给 GC 去操作，只想 GC 内存堆

**非托管内存：结构体，保留了指针，指针指向了 非托管内存堆（C++ 层面申请出来的内存，这块内存不归 GC 管理，而不是指向所谓的 栈内存），释放需要手动调用 Dispose 进行释放**

结构体 虽然是在栈上分配的内存，但是 NativeArray 在栈上只是存储了一个指针，指针指向了 非托管内存，所以哪怕有再多的东西，也是分配到了 非托管内存中，栈不会因为 array 过大而导致 暴栈



using 的使用 和  结构体赋值（=） 值拷贝(浅层次拷贝) 的情况：

```c#
//托管内存
public List<int> intList;

//非托管内存
public NativeArray<int> intNativeArray;


public void NativeArrayUsage()
{
    //会在 非托管内存上 开辟一片连续的内存
    //intNativeArray 是一个壳 里面保留了一个指针而已
    intNativeArray = new NativeArray<int>(16, Allocator.TempJob);
    //非托管内存 需要手动 Dispose GC不管释放的操作
    intNativeArray.Dispose();
    
    using(var tempIntArray = new NativeArray<int>(16, Allocator.Temp))
    {
        //这里会产生编译错误
        //因为 using 语法糖在这里只能读不能写
        //tempIntArray[0] = 1;
        
    }// using 语法糖 在结束之后会自动调用 Dispose
    
    //因为 NativeArray 是一个结构体 所以在这里赋值的操作会产生“值拷贝”(浅层次拷贝)
    //即：= 这个操作会自动复制右操作数的所有值类型字段
    //因为 壳 里面的指针也是值类型，所以就可以复制给 valueCopyNativeArray
    //但是仅仅只是复制了指针，而不是复制指针所指得到内存片
	var valueCopyNativeArray = intNativeArray;    
    
    //TODO：注意：修改了 valueCopyNativeArray 但是 intNativeArray 不会跟着被修改
    
    //浅拷贝只会拷贝对象的第一层属性，如果这些属性是对象，则不会对这些对象进行拷贝，而是直接复制对象的引用。这意味着，对于浅拷贝后的对象，如果原对象的属性值发生了变化，浅拷贝后的对象的属性值也会跟着发生变化。
    //如果是深拷贝：会把指针里的所有的内容都拷贝出一份（new出来一份新的内存 然后把原来的内存转存过来（dunp））
}
```



## 栈内存 和 堆内存

**NativeContainer 虽然是 struct，但是主要内存区 是“非托管内存”**

```c#

public unsafe void StackAllocUsage()
{
    var allocCount = 100000;
    //在调用栈上开辟了内存 但是allocCount太大导致暴栈
    //在栈内存上的内存在函数运行之后会被回收（栈都弹完了就内存没了）
    var stackAllocArray = stackAlloc int[allocCount];
}

```





# NativeContainer 



## 几种常用的 NativeContainer 

```c#

//最常用 但不好用 用之前需要知道多长 
//NativeArray 不支持嵌套 无法描述二维数组
//连续内存访问，速度最快 Dots 很多 API 只接受这个类型
public NativeArray<int> mostCommonNativeContainer = new NativeArray<int>(12, Allocator.Temp);

//唯一一个不需要知道长度就可以使用
//能在 Job 中使用的 容器，支持 Job 的并发写入，在 Job 中出来之后转成 NativeArray 去遍历 = NativeQueue.ToArray 非常非常常用的操作
//内部实现是一个链表 但是不支持 index 索引（即 [x]） 需要遍历
public NativeQueue<int> mostLovelyNatvieContainer;

//虽然有变长属性 但是 【只有在主线程】中可以变长 但是在 Job 中不能变长（有点鸡肋）
//在主线程 resize 的时候会保证内存连续，是一个 NativeArray 的加强版
//TODO：因为投入在 Job后就是一个 ？？？ ，而 ？？？ 只有一个 API
public NativeList<int> mostDisappointmentNativeContainer;

//需要提前知道长度 与 Dictionary 不同
//除了不能 resize 和 Dict 差不多
public NativeParallelHashMap<int, int> nativeHashMap;

//在线程中解决嵌套数组
//可以描述 二维数组，一键对多值的容器
//遍历的代码非常丑陋
//使用在 NativeContainer 嵌套的情况下
public NativeParallelMultiHashMap<int, int> mostAglyNativeContainer;

//从 NativeArray 中截取一部分片段 - 面向数组的切片
//在底层代码的时候使用
//比如在 NativeArray 获取指定的截取范围 
public NativeSlice<int> mostTransparentNativeContainer;

```



## NativeContainer 具体分类

**最大的用法在 Job 中使用**



### NativeArray



注意：NativeArray 的嵌套 无法在 Job 中使用：

```
public NativeArray<NativeArray<int>> nestedArray;
```



**带有 NativeContainer 标签 的都不能嵌套**

![NativeContainer](.\images\NativeContainer.png)



解决限制嵌套的办法：

使用 Unsafe：解决禁止嵌套的问题：

但是注意：Unsafe 使用了之后 就会减少一些检查，不建议使用

**如果需要使用二维数组，就去使用 NativeParallelMultiHashMap**

```c#
//引入命名空间
using Unity.Collections.LowLevel.Unsafe;

public NativeArray<Unsafe<int>> nestedArray;

```



#### Allocator 分类

1、**Allocator.Temp**：不需要 Dispose **自动 Dispose** ，速度最快

在 Job 中使用容器， Temp 是最好的

```c#
var nativeArray = new NativeArray<Vector2>(16, Allocator.Temp);

//可以进行 Dispose（不进行 Dispose 也可以）
nativeArray.Dispose();
```



2、Allocator.TempJob： **常用**

**只要是 NativeContainer 和 NativeArray 想要进入 Job 的 Execute ，就必须使用 TempJob**

否则会报错

```c#
var nativeArray = new NativeArray<Vector2>(16, Allocator.TempJob);

//可以进行 Dispose（不进行 Dispose 也可以）
nativeArray.Dispose();

//================================================ 主线程

//主线程中的做法
public void NativeUsage()
{
    var nativeArray = new NativeArray<Vector2>(16, Allocator.TempJob);
    for(int index = 0; index < nativeArray.Length; index++)
    {
        var nativeArrayElement = nativeArray[index];	//index 的 element 发生了值拷贝
        nativeArrayElement.x += 10;						//由于是值拷贝 NativeArrayElement 作为一个独立的个体，修改之后无法反馈给 NativeArray
        nativeArray[index] = nativeArrayElement;		//必须回填
    }
}

// ================== 如果在主线程中 不想进行回填 使用 unsafe
public unsafe void NativeUsage()
{
    var nativeArray = new NativeArray<Vector2>(16, Allocator.TempJob);
    
    //操作指针 就不需要回填
    //这个方式主要用在 结构体很大的情况下 可以省下一次值拷贝 只修改一个小属性
	var nativeArrayPtr = (Vector2*)nativeArray.GetUnsafePrt();
    for(int index = 0; index < nativeArray.Length; index++)
    {
		var currentElementPtr = nativeArrayPtr + index;
        //操作指针 就不需要回填
        currentElementPrt->x += 10;
    }
}

//================================================ Job 线程

//Job 线程
//IJobParallelFor 
public struct NativeArrayJob : IJobParallelFor
{
    public NativeArray<Vector2> nativeArray;
    void IJobParallelFor.Execute(int index)
    {
        //对成员字段容器的操作必须记住三个步骤
		//1、读值
        //2、修改
        //3、回填（最重要）
        var nativeArrayElement = nativeArray[index];	//index 的 element 发生了值拷贝
        nativeArrayElement.x += 10;						//由于是值拷贝 NativeArrayElement 作为一个独立的个体，修改之后无法反馈给 NativeArray
        nativeArray[index] = nativeArrayElement;		//必须回填
	}
}

//主线程
public unsafe void NativeUsage()
{
    //主线程中的 nativeArray
    var nativeArray = new NativeArray<Vector2>(6, Allocator.TempJob);
    
    //假设有 nativeArray 有 6 个元素
    //有4个线程（worker线程）的情况下  - worker线程 是 Unity进行封装的
    // 1号线程 处理 0 1 2
    // 2号线程 处理 3 4 5
    // 平行时间下 = 1号线程 处理 0 的时候 2号线程 在处理 3
    // 因为线程是并发执行的  所以 原本for循环顺序遍历会被拆开 - 产生乱序
    
    // 因此 IJobParallelFor 中的 Execute 的 index 是乱序的
    //for(int index = 0; index < nativeArray.Length; index++)
    //{
    //    var nativeArrayElement = nativeArray[index];	//index 的 element 发生了值拷贝
    //    nativeArrayElement.x += 10;						//由于是值拷贝 NativeArrayElement 作为一个独立的个体，修改之后无法反馈给 NativeArray
    //    nativeArray[index] = nativeArrayElement;		//必须回填
    //}
    
    //Job 中的赋值会修改到原来的引用
   	var nativeArrayJob = new NativeArrayJob()
    {
        //值拷贝 拷贝了指针
        //在 Job 中 Execute 修改后 主线程中的 nativeArray 的值也会对应的修改
        nativeArray = nativeArray
	};
}

```



3、Allocator.Persistent： 用在生命周期很长的对象（System、静态数据部分）

**在 System 中保留某个 State（即：数据）全局生命周期**

```c#
private NativeArray<Vector2> allMonsterCurrentPosition;

public void NativeArrayUsage()
{
    //创建的时候
    allMonsterCurrentPosition = new NativeArray<Vector2>(16, Allocator.Persistent);
    
}

public void Dispose()
{
    //释放的时候
    allMonsterCurrentPosition.Dispose();
}
```





### NativeQueue

使用场景：在一串未知长度的 NativeArray 数据中筛选出条件下的所有元素给到下游的函数

**NativeQueue 满足多少条件的就可以申请多少的内存**

```c#
//Job 线程
private struct NativeQueueJob : IJobParallelFor
{
    [ReadOnly] public NativeArray<Vector2>.ReadOnly input;
    //注意：在 Job 线程中需要写入的时候  必须有 ParallelWriter
    public NativeQueue<Vector2>.ParallelWriter nativeQueue;
    
    void IJobParallelFor.Execute(int index)
    {
        var oneIndex = input[index];
        if(oneInput.x > 0)
        {
            //筛选条件后
            nativeQueue.Enqueue(oneInput);
        }
    }
}

//主线程
public void NativeQueueUsage(in NativeArray<int> inputDataArray)
{
    var nativeQueue = new NativeQueue<Vector2>(Allocator.TempJob);
    var nativeQueueJob = new NativeQueueJob()
    {
   		//只读的 Array 标记为这样可以增强代码可读性提升性能
        input = inputDataArray.AsReadObly(),
        //AsParallelWriter 是重点，因为这样 Queue 才可以支持多线程写入（但是这样也会损失 Dequeue功能）
        nativeQueue = nativeQueue.AsParallelWriter(),
    };
    nativeQueueJob.Schedule(inputDataArray.Length, 32).Complete();
    
    
    //怎么讲结果传递给下游的函数 ProcessAfterNativeQueueUsage
    var nativeQueueToArray = nativeQueue.ToArray(Allocator.TempJob);
    nativeQueue.Dispose();//转成Array 之后可以立马将 Queue 释放，因为 Array 是新开辟的内存
    ProcessAfterNativeQueueUsage(nativeQueueToArray);
    nativeQueueToArray.Dispose();	//新开辟的内存也需要释放掉
}

private void ProcessAfterNativeQueueUsage(in NativeArray<Vector2> xGreaterThenOElement)
{
    
}


```





### NativeParallelMultiHashMap

2种遍历方法

```c#
//====================================== 遍历方法1 - 建议使用

//假如 NativeParallelMultiHashMap<int,int>
//里面的数据是 (1,1) (1,2) (2,3)
//数据不是链表，而是平坦的存储着
//假如获取的时候 key=1 则会获取到 两个值 (1,1) (1,2) 这时候重复的Key 也被返回出来
//如果使用的是 GetUniqueKey 除了会给到 value 之外 还会给对应的 Key 以及 一共不重复的有多少个 = 比如这里会返回  1 2 1 2
	//GetUniqueKeyArray
    //uniqueIdArray = 1 2 1 = value + Key
    //uniqueIdCount = 2 = 不重复的数量
public void WalkNativeParallelMultiHashMap(in NativeParallelMultiHashMap<int,Vector2> footPositionMap)
{
    var (uniqueIdArray, uniqueIdCount) = footPositionMap.GetUniqueKeyArray(Allocator.TempJob);
    for(int i = 0; i < uniqueIdCount; i++)
    {
        var uniqueKey = uniqueIdArray[i];
        var ienumer = footPositionMap.GetValueForKey(uniqueKey);
        //遍历迭代器
        while(ienumer.MoveNext())
        {
            var footPosition = ienumer.Current;
            if(footPosition.x > 0)
            {
                //找到结果
                var luckyDogId = uniqueKey;
                break;
            }
        }
    }
    uniqueIdArray.Dispose();//需要 Dispose
    
}


//====================================== 遍历方法2 - 不友好的写法

public void WalkNativeParallelMultiHashMap2(in NativeParallelMultiHashMap<int,Vector2> footPositionMap)
{
    var (uniqueIdArray, uniqueIdCount) = footPositionMap.GetUniqueKeyArray(Allocator.TempJob);
    for(int i = 0; i < uniqueIdCount; i++)
    {
        var uniqueKey = uniqueIdArray[i];
        var ienumer = footPositionMap.GetValueForKey(uniqueKey);
        //TryGetFirstValue
        if(footPositionMap.TryGetFirstValue(uniqueKey, out Vector2 footPosition, out NativeParallelMultiHashMapIterator<int> iter))
        {
            do
            {
              	if(footPosition.x > 0)  
                {
					var luckyDogId = uniqueKey;
                    //break 到 goto 的位置
                    break;
                }
            }while(footPositionMap.TryGetNextValue(out footPosition, ref iter));
        }
        //[goto]
    }
    uniqueIdArray.Dispose();//需要 Dispose
    
}

```



# 非ECS下也可以使用的 Job

使用多线程去优化项目



## Job 的限制 - 字段采用struct，不能使用 class 

1、因为 无法在 Job 多线程访问 class类，而 Job多线程下载和解压资源的 API 是 class 实现，所以无法实现  Job多线程下载和解压资源

2、因为 无法在 Job 多线程访问 class类，所以无法实现在 Job 多线程中实例化 GameObject 或 批量设置自定义脚本的属性

3、因为在 Unity后台 已经实现了 在 Job 多线程中加载资源，禁止了多线程加载资源，所以无法实现



## Job 的使用场景 

1、加减乘除 的大规模运算，复杂伤害公式的计算结果

2、C++实现的 NavigationMesh 地图寻路的结果

3、简单行为树的 AI 决策

4、搭配和  Burst 一起使用，将 struct 抽成为 Job + Burst标签，**对于 Dots 而言，Burst 的提升是最大的**



提升程度排名：Burst  > ECS > Job

## Burst  + 内存连续 可以实现大幅度提升优化

**如果一个函数成为性能瓶颈，可以将这个函数抽出去成为 IJob。试试 Burst**



## Job 的使用方法论

**方法论 1：确定需求是否可分布式，如果不是是否有办法可以拆解为分布式**

**分布式：每个东西单独进行运算，互不影响**，算法可分布式下就可以对每个个体实现算法，使用 **IJobParallelFor** 进行并发操作

​	**落地准则：**

​		**1、牵扯到排序的不属于可分布式，不能使用 Job**

​		**2、个体之间的结果有影响的不属于可分布式**

​	应用：A* 寻路算法的 分布式（拆解为分布式）

​	寻路之间互不干扰，虽然小兵之间相互阻挡会互相影响，但可以做一个专门处理小兵阻挡的 Job，第一个 Job 处理小兵的阻挡把地图上抠出洞，第二个 Job 走A* 寻路，所以还是一个分布式的



**方法论 2：在不是分布式的情况下，需求是否可以拆成 N 个并发的子流程，变为多个 IJob 去解决问题 - 重点**

​	应用：大区的排行榜 排序 本身不是一个 分布式，但是可以将大区划分为多个小区（子流程），每个小区在自己线程里的排序，所有小区就可以排序完毕



**如果无法拆分，就只能在主线程中实现**



寻路 Job

```c#
//Burst编译
[BurstCompile]
private struct FindPathJob : IJobParallelFor
{
	[ReadOnly] public NativeArray<int>.ReadOnly agentIdArray;
    [ReadOnly] public NativeArray<Vector2>.ReadOnly currentAgentPosArray;
    [ReadOnly] public NativeArray<Vector2>.ReadOnly agentDestArray;
    
    public NativeParallelMultiHashMap<int, Vector2>.ParallelWriter outputPathPointsMap;
    
    void IJobParallelFor.Execute(int index)
    {
        var currentPoint = currentAgentPosArray[index];
        var destPoint = agentDestArray[index];
        var agentId = agentIdArray[index];
        //注意：pathPoints 需要自己释放
        if(CalcFinalPosition(currentPoint, destPoint, out var pathPoints, Allocator.Temp))
        {
            foreach(var pathPoint in pathPoints)
            {
                outputPathPointsMap.Add(agentId, pathPoint);
			}
            pathPoints.Dispose();
        }
	}
    
    private bool CalcFinalPosition(in Vector2 currentPosition, in Vector2 destPosition, out NativeArray<Vector2> pathPoints, Allocator allocator)
    {
        pathPoints = new NativeArray<Vector2>(2, allocator);
        pathPoints[0] = currentPosition;
        pathPoints[1] = destPosition;
        return true;
    }
}
```



大区的玩家，按照区号排序+战力排序的排行榜

1、先将大区玩家分成小区，n 个 list

2、list 相互不影响 单独排序

3、list的个数是小区的个数，所以小区数目越多越合适，最差的情况是只有一个小区

4、用 n 个 IJob 并发来做

```c#

private struct PlayerInfo
{
    public int subZoneId;
    public int score;
    public struct Sorter : IComparer<PlayerInfo>
    {
        int IComparer<PlayerInfo>.Compare(PlayerInfo x, PlayerInfo y)
        {
            var scoreX = x.score;
            var scoreY = y.score;
            if(scoreX > scoreY){	return 1; }
            else if(scoreX < scoreY) {return -1;}
            else {return 0;}
        }
    }
}

//Allocator 写在参数列表是为了提醒外边的 sorted 需要释放
private bool SortMainZonePlayers(in NativeArray<PlayerInfo> mainZonePlayers, out NativeArray<PlayerInfo> sorted, Allocator allocator)
{
    sorted = default;
    return false;
}

// ========================================= IJobParallelFor 写法

// 分小区是一个分布式的行为：把我分到我所在的小区，把你分到你所在的小区，跟其他人没有影响
// 所以分小区可以使用 IJobParallelFor
[BurstCompile]
private struct SeperateMainZonePlayerIntoSubZoneJob : IJobParallelFor
{
    //所有的玩家
    [ReadOnly] public NativeArray<PlayerInfo>.ReadOnly allMainZonePlayers;
    //小区为key 的player 列表
    public NativeParallelMultiHashMap<int, PlayerInfo>.ParallelWriter subZones;
    
    void IJobParallelFor.Execute(int index)
    {
        var player = allMainZonePlayers[index];
        subZones.Add(player.subZoneId, player);
    }
}

[BurstCompile]
private struct SortPlayerScoreJob : IJob
{
    public int startWriteIndex;
    
    //通过 Index 写入到共享的 Buffer 中 
    //Unity会报错认为是一个非安全的操作
    //需要加上2个标签
    [NativeDisableParallelForRestriction]
    [NativeDisableContainerSafetyRestriction]
    public NativeArray<PlayerInfo> outPutArray;

    public PlayerInfo.Sorter sorter;
    
    [ReadOnly]public NativeParallelMultiHashMap<int, PlayerInfo> inputMap;
    public int subzoneId;
    
    void IJob.Execute()
    {
        var subZonePlayerCount = inputMap.CountValuesForKey(subzoneId);
        var sortPlayers = new NativeArray<PlayerInfo>(subZonePlayerCount, Allocator.Temp);
        var ienumer = inputMap.GetValuesForKey(subzoneId);
        var subZonePlayersIndex = 0;
        while(ienumer.MoveNext())
        {
            sortPlayers[subZonePlayersIndex++] = ienumer.Current;
        }
        
        sortPlayers.Sort(sorter);
        //把排序的结果直接写入outPutArray
        NativeArray<PlayerInfo>.Copy(sortPlayers, 0, outPutArray, startWriteIndex, sortPlayers.Length);
    }
}


//Allocator 写在参数列表是为了提醒外边的 sorted 需要释放
private bool SortMainZonePlayers(in NativeArray<PlayerInfo> mainZonePlayers, out NativeArray<PlayerInfo> sorted, Allocator allocator)
{
    var count = mainZonePlayers.Length;
    if(count == 0){
        sorted = default;
        return false;
    }
    
    sorted = new NativeArray<PlayerInfo>(count, allocator);
    using var subZonePlayersMap = new NativeParallelMultiHashMap<int, PlayerInfo>(mainZonePlayers.Length, Allocator.TempJob);
    
    //分离
    var seperateMainZonePlayerIntoSubZoneJob = new SeperateMainZonePlayerIntoSubZoneJob()
    {
      	allMainZonePlayers = mainZonePlayers.AsReadOnly();
        subZones = subZonePlayersMap.AsParallelWriter();
    };
    //Job 有浓重的多线程色彩，异步下 Complete 后才会继续往下执行
    seperateMainZonePlayerIntoSubZoneJob.Schedule(count, 64).Complete();
    
    //遍历
    var (uniqueSubZoneIdArray, uniqueSubZoneIdCount) = subZonePlayersMap.GetUniqueKeyArray(Allocator.TempJob);
    //用一个容器 收集 JobHandle
    var sortHandles = new NativeList<JobHandle>(uniqueSubZoneIdCount, Allocator.TempJob);
    var writeIndex= 0;
    for(int i=0; i < uniqueSubZoneIdCount; i++)
    {
        var subZoneId = uniqueSubZoneIdArray[i];
        var subZonePlayerCount = subZonePlayersMap.CountValuesForKey(subZoneId);
        
        //由于 Job 中标记了 [DeallocateOnJobCompletion] 因此不用管
        //容器收纳整个区里所有的玩家
        //var subZonePlayers = new NativeArray<PlayerInfo>(subZonePlayerCount, Allocator.TempJob);
        
        //抽出玩家填充到容器中
        //var ienumer = subZonePlayersMap.GetValuesForKey(subZoneId);
        //var subZonePlayersIndex = 0;
        //while(ienumer.MoveNext())
        //{
        //    subZonePlayers[subZonePlayersIndex++] = ienumer.Current;
        //}
        
        //因为排序不是分布式的行为 使用方法论2 建立 iJOb 进行排序
        var sortJob = new SortPlayerScoreJob()
        {
            sorter = new PlayerInfo.Sorter(),
            startWriterIndex = writeIndex,
            outPutArray = sorted,
        };
        //注意 不能使用 Complete 因为会阻塞
        //JobHandle 实现多次之间的并发 拿到多个 JobHandle 之后统一进行 Complete
        //sortJob.Schedule().Complete();
        var sortJobHandle = sortJob.Schedule();
        sortHandles.Add(sortJobHandle);
        writeIndex += subZonePlayerCount;
    }
    
    //异步转同步
    JobHandle.CompleteAll(sortHandles);
    
    uniqueSubZoneIdArray.Dispose();
    return false;
    
}

```



## Job 的依赖关系 - 做好约束

避免出现非必现时序的Bug，建议：

**1、Job 的 Complete 绝对不能跨帧，否则会变成纯粹的多线程异步无法控制**

**2、在第一条的前提下，Complete 最好能在函数内部执行**



**串行，但是在线程中执行**

但是在第一个Job 中，如果有一个需要被write写入的成员变量

但是因为第一个Job 没有 Complete，所以这个变量没有写入完毕，只有整个链都执行完毕之后才能拿到这个成员变量

在这样的情况下，无法进行 Debug

```c#

[BurstCompile]
private struct Step1Job : IJob
{
	//需要被写入的变量 想要进行数据断点查看
	public NativeArray<int> outPut;
	void IJob.Execute(){}
}

[BurstCompile]
private struct Step2Job : IJob
{
	void IJob.Execute(){}
}

private JobHandle ExecuteRelativeJob(in JobHandle prevPipelineJobHandle)
{
  var step1Job = new Step1Job();
  var step1JobHandle = step1Job.Schedule(prevPipelineJobHandle);
  var step2Job = new Step2Job();
  var step2JobHandle = step1Job.Schedule(step1JobHandle);
  return step2JobHandle;
}


// 换成下面这种做法 去 强制 必须进行 Complete 保证正常
private JobHandle ExecuteRelativeJob(in JobHandle prevPipelineJobHandle)
{
  prevPipelineJobHandle.Complete();
  
  var step1Job = new Step1Job();
  step1Job.Schedule().Complete();
  
  var step2Job = new Step2Job();
  step2Job.Schedule().Complete();
  
  return default;
}

```





# OOP 和 ECS



OOP：

1、面向个体（new 出来的单个对象）

2、数据和方法要写在一起：方法是对象的方法，目的是去修改对象的数据

3、多态：以继承的方式对功能分层重点在于对基类进行统筹



ECS：

**面向批次：面向有相同特诊的一批数据**（不论是猫的尾巴还是狗的尾巴，只要尾巴会摇动的就可以实现）

数据，方法分离：数据是赤裸裸的数据，方法中不能含有 **状态（即 数据** - 声明周期超过其调用函数，或者会影响到下一帧计算的数据都可以被称为状态）

组合优先于继承：通过组件的排列组合达到功能的实现

 ![](.\images\ECS 倾向的方案.png)



# DOTS-ECS 入门知识

Entity 是作为一个 **ID 索引** 存在，而不是作为一个对象，不要用 Entity.方法



## Entity 



### 在 DOTS 中 Entity 是什么索引

比如有个 尾巴数组，数据在 DOTS 中 被称呼为 Chunk

Entity ：Chunk 的索引，ID 索引，数组角标

:1 ：表示的是 版本，用于检验是否合法



**Entity: Version**



[尾巴A,     尾巴B,     尾巴C ]					

​      ↑               ↑               ↑

柴犬Entity:0:1

​                 柯基Entity:1:1

​                                  比鲁斯猫Entity:2:1

​      ↓               ↓               ↓

[爪子A,     爪子B,     爪子C ]

​                       

 比鲁斯猫Entity:2   被销毁，但是 尾巴C 和 爪子C 还是指向  比鲁斯猫

这时候如果是 象帕猫，则 Version + 1，检验是否合法



[尾巴A,     尾巴B,     尾巴C**=> 尾巴D** ]					

​      ↑               ↑               ↑

柴犬Entity:0:1

​                 柯基Entity:1:1

​                                  象帕猫Entity:2:**2**

​      ↓               ↓               ↓

[爪子A,     爪子B,     爪子C**=> 爪子D** ]



**判断一个 Entity 是否为空（有效）：**

```c#

public void WhatInsideEntity(in EntityManager em)
{
  //如果是被 new 出来的 Entity 空的情况下则会是 Entity.Null
  //EntityManager里有 根据 Version 可以判断是否有效
  Entity mayByNullEntity = default;
  if(mayByNullEntity == Entity.Null || em.Exists(mayByNullEntity) == false)
  {
    Debug.LogError("空");
  }
  
  //使用 EntityManager 去 创建Entity
  Entity newEntity = EntityManager.CreateEntity()
  //不能直接 newEntity = new Entity(); C# 要求 Struct 有自己的默认构造函数，虽然编译可以正常，但不应该这么写
    
}



```

EcsWorld 是一个沙盒，每个沙盒里都有一个专门负责管理  Entity 的 EntityManager

EntityManager：World 管理器

EntityManager 不是静态量 而是有生命周期的一个对象实例，并且这个生命周期和 World 沙盒生命周期绑定



### 注意点



**注意1：Entity 绝对不要作为字典的Key**

因为 字典的Key 是取的 HashCode 

而 Entity 的  GetHashCode是直接使用了 Index返回，如果作为字典的 Key，那么失效的情况下（比鲁斯猫 变为 象帕猫），则会出现很大的 隐患，**除非能保证字典的有效期内 Entity 从不销毁**



**注意2：虽然 EntityCommandBuffer 可以在线程中 批量创建Entity，但是 创建ntity 其实还是放在了主线程中**



**注意：ECS 的 UUID 绝对不要考虑 Index**





## Components

并不是 Unity 的 Component ，与 Unity 的 Component  有下面的差异：

1、ECS中 对于 Entity 来说，同一个类型的 ComponentData 只能有1个，但在 Unity 的 对于 GameObject 来说可以有多个Component （GetComponentsInChildren）

2、ECS中 ComponentData 必须是 struct，没办法实现继承，没办法实现多态，Unity 的 Component 比较支持继承和多态



### IComponentData

最常用最核心的组件类型

用处：能携带数据，在内存中平坦连续，在批处理中能处理

处理数据其实就是在处理 Components

1、只是一个数据块

```c#
public struct MostCommUseComponent : IComponentData
{
	public int dataNumber;
}
```



2、可以描述一类数据

没有字段，只是做为逻辑区分的 Component

柴犬 有自己 柴犬 Tag

上面的数据块 也可以作为 Tag 使用

```c#
public struct MosterComponent : IComponentData
{

}
```



3、基于前2个，所以可以集中处理一类数据，这个集中的过程称为 **Query**





### ISystemStateComponent - 系统

用处：贴有 ISystemStateComponent 的 Entity 在销毁的时候会阻止销毁，只有当所有 ISystemStateComponent 全部被销毁之后才会自动销毁

可以用在资源系统的管理资源释放

1、监听某个东西被销毁的时候做逻辑，可以做但尽量少做

2、一个东西被销毁的时候数据本身就不完整了，销毁应该决绝点



```c#

public struct SystemStateComponent : ISystemStateComponentData {}

public void SimulateDestroyEntity(in EntityManager em)
{
  var entity = em.CrateEntity();
  em.AddComponent<MonsterTag>(entity);
  em.AddComponent<SystemStateComponent>(entity);
  em.DestroyEntity(entity);	//执行这句代码后 MonsterTag 会被销毁，但是因为 SystemStateComponent 贴在上面，所以 Entity 还是会存活
  em.RemoveComponent<SystemStateComponent>(entity);//执行完这句话后，Entity 才正式被销毁
  Debug.Assert(em.Exists(entity) == false);
}

public void UseageOfSystemStateComponent(in Entity monsterEntity, in EntityManager em)
{
  if(!em.HasComponemt<MonsterTag>(monsterTag) && em.HasComponemt<SystemStateComponent>(monsterTag))
  {
    	//Entity 被 Destroy 过
    	//在这一步可以去释放对应的怪物的美术资源
    	
  }
}

```





### ISharedComponent - 共享

1、本身必须是一个 struct

2、**字段可以是 class，但只要是 class 字段，就绝对不可以 BrustCompile， 会导致 真机崩溃**

3、SharedComponent 最大的用处是 为 Query 而生，**（真正的用途）**



用处：

1、在不得不去存储一些 Class 类型的字段的时候

2、想筛选 一类字段相等的 Entity 的时候**（重要的用法）** 如果有 N个 Sprite 想根据他们所使用的材质球进行抓取

```c#

[Unity.Burst.BurstCompile]
public struct PureStructSharedComponent : ISharedComponent 
{
	public int intValue;
}

//注意：这里不可以 Burst
//[Unity.Burst.BurstCompile]
public struct HybridSharedComponent : ISharedComponent 
{
	//要让 Unity 使用 过滤器，非值类型 要抽象为 值类型，需要实现 IEqultable 
	public GameObject gameObject;
}

public struct SpriteTag : IComponentData 
{

}

//需要实现 IEqultable
public struct SpriteMatUsage : ISharedComponentData, IEqultable<SpriteMatUsage>
{
	public Material mat;
	
	bool IEqultable<SpriteMatUsage>.Equals(SpriteMatUsage other)
    {
    	return mat == other.mat;
    }
    
    public override int GetHashCode() 
    {
    	return mat == null ? 0 : mat.GetHashCode(); 
    }
}

public void SharedComponentDataUsage(in EntityManager em, in NativeArray<Entity> spriteEntity, Material selectedMaterial)
{
  EntityQuery query;
  using(var builder = new EntityQueryDescBuilder(Allocator.Temp))
  {
    builder.AddAll(typeof(SpriteTag));
    builder.AddAll(typeof(SpriteMatUsage));
    builder.FinalizeQuery();
    query = em.CreateEntityQuery(builder);
  }
  
  //使用 query 快速获得所有 贴有SpriteTag 的 Entity
  //高性能 是 ECS 框架的基石
  var allSpriteEntity = query.ToEntityArray(Allocator.Temp);
  
  //改造 query - SharedComponent的最大用处
  query.AddSharedComponentFilter(new SpriteMatUsage(){mat = selectedMaterial});
  
  //到了这一步后，这些 Sprite 都是只有 selectedMaterial 的 sprite 了
  //这个 query 非常迅速 比 for 去判定要快速还简洁
  var weWantSprites = query.ToEntityArray(Allocator.Temp);
  
}

```



### DynamicBuffer

像List 一样，但是没有List好用，解决对于一个 Entity 想要贴上多个 ComponentData 的需求下使用：

解决 Component 唯一性

可以堆叠，能同个类型存在多个的 ComponentData（一般是 贴重复 IComponentData，只有 IComponentData 解决不了的时候才用）

不要大规模使用这个组件

```c#

//虽然是变长，但是超过这个指定值就会在堆里
[InternalBufferCapacity(8)]
public struct DynamicBufferElement : IBufferElementData {}

public void DynamicBufferUsage(in EntityManager em, in Entity entity)
{
  em.HasComponent<DynamicBufferElement>(entity);
  
  var buffer = em.AddBuffer<DynamicBufferElement>(entity);
  //在主线程中操作这个buffer就像在操作 List 一样
  buffer.RemoveAt(0);
  
  em.RemoveComponent<DynamicBufferElement>(entity);
  
  
}

```





# Query Job

Query 是指针运算，针对数组级别的操作，批量抓取某个组件

如果没有 ECS，for 循环  Entity 然后把又组件的 查找出来

而 ECS 的 Query 可以高效抓取某个或者多个组件 ，可以搭配上 Tag，然后将 Query 传递给 EntityManager 去查找所有 有这些组件 与 Tag 交集的 Entity ，也可以在 Query 的时候 加上 withouut 的剔除过滤的部分

一个沙盒一个 EntityManager，而 Query 从 EntityManager 来的，属于沙盒中的一部分，只能抓取沙盒中的信息



1、引入 Unity.Enities 的命名空间

2、继承 SystemBase，注意需要加上 partial，因为 ECS 会在另一个 SystemBase 里生成 partial 的代码

3、定义指定的组件

4、定义 EntityQuery

5、一般都是在  SystemBase 的 OnCreate 函数中实现 Query，因为 Query 在生命周期中只执行一次

6、创建 EntityQueryDescBuilder 的查询描述的实例  builder，如果不需要在Job中直接使用 Temp就可以，使用 AddXXX 去添加需要的描述

7、AddAll - 必须具有的     AddAny - 有或者没有都可以被抓取    最后调用 FinilizeQuery

8、调用 SystemBase 的 GetEntityQuery 将 builder 传入得到带有过滤条件后的 Query

9、创建 EntityJob 继承 IJobEntityBatch，去批处理某些 Entity 的组件

10、在 Update 中 将 Query 和 Job 关联上，让 Job 批量处理 符合 Query 描述的东西

11、在 Job 中访问到指定的 Query 后的东西，定义 ComponentTypeHandle<XXX> 对应 Quey 中的 AddXXX，ComponentTypeHandle 可以取到数据

12、使用 SystemBase 的GetComponentTypeHandle<XXX>(true or false) 的方式 去赋值到 Job的 定义的变量

如果是只读 [ReadOnly] 的，就传入 true ，可读可写就传入 false， [ReadOnly] 对性能有很好的提升，读写权限一定要分配好，否则 Unity 会报错

13、ArchetypeChunk 的 GetNativeArray 可以获取到数组

14、注意遍历的时候要使用  batchInChunk 的 Count 进行遍历，处理某个字段的值 并且 将数据回填，使用 batchInChunk 的 Has 判断是否有某个 Tag

```c#

[BurstCompile]
public struct CdTail : IComponentData { }

[BurstCompile]
public struct CdClow : IComponentData
{
	public float nailLength;
}

[BurstCompile]
public struct CdCatTag : IComponentData { }


//面向批次的 Job
[BurstCompile]
private struct TrimCatNailJob : IJobEntityBatch
{
	//因为无法使用 GetNativeArray 获取 所以这里也要去掉
	//[ReadOnly] public ComponentTypeHandle<CdCatTag> catTagsHandle;
	public ComponentTypeHandle<CdClow> clowsHandle;

	/*
	batchInChunk :
	[爪子A,       爪子B,       爪子C,       空]
	[猫A,         猫B,         猫C,        机器猫]
	*/
	void IJobEntityBatch.Execute(ArchetypeChunk batchInChunk, int batchIndex)
    {
    	//判断是否有爪子
    	var hasClows = batchInChunk.Has(clowsHandle);
    	var clows = hasClows ? batchInChunk.GetNativeArray(clowsHandle) : default;
    
    	//获取到数组 [爪子A,       爪子B,       爪子C,       空]
    	//var clows = batchInChunk.GetNativeArray(clowsHandle);
    	
    	
    	//获取到数组 [猫A,         猫B,         猫C,        机器猫]
    	//注意 因为 CdCatTag 里没有字段，结构体在内存上排布的时候没有长度，长度为0，Unity 无法取出，会报错
    	//var tags = batchInChunk.GetNativeArray(catTagsHandle);
    	    	
    	
    	//注意不能使用 clows 进行遍历 而是使用 batchInChunk 的 Count 进行遍历
    	for(int bIndex = 0, bIndex < batchInChunk.Count; bIndex++)
        {
        	//注意这里不能直接使用是否为空做判断
        	//因为 ArchiveType 为 2个，Execute 会执行2次，只有 batchInChunk 才知道是否有某个 Tag
        	// 
        	/*
        	第一次 
        	[爪子A,       爪子B,       爪子C]
        	[猫A,         猫B,         猫C]
        	
        	第二次 
        	[空]
        	[机器猫]
        	
        	*/
        	if(hasClows)
            {
                var clow = clows[bIndex];

                //处理某个字段的值
                clow.nailLength = Mathf.Min(1, clow.nailLength)；

                //将数据回填
                clow[bIndex] = clow;        
            }	
        }
    }
}


using Unity.Entities;

public partial class QueryJobSystem : SystemBase
{

	//定义 Query 查找带有爪子的猫
	private EntityQuery catMaybeWithClawQuery;
	
	protected override void OnCreate()
	{
		//如果不需要在Job中直接使用 Temp就可以
		using(var builder = new EntityQueryDescBuilder(Unity.Collections.Allocator.Temp))
        {
        	builder.AddAll(typeof(CdCatTag));
        	builder.AddAny(typeof(CdClow)));
        	builder.FinilizeQuery();
        	
        	catMaybeWithClawQuery = GetEntityQuery(builder);
        }
	}

	protected override void OnUpdate()
	{
		var trimCatNailJob = new TrimCatNailJob()
        {
        	catTagsHandle = GetComponentTypeHandle<CdCatTag>(true),
        	clowsHandle = GetComponentTypeHandle<CdClow>(false)
        };
        trimCatNailJob.ScheduleParallel(catMaybeWithClawQuery).Complete();
	}
}

```





## 实战

需求：实现一个可以对主人自动治疗的宠物

1、主人的HP<50的时候，将主人的HP恢复到50，并且添加一个每秒钟恢复5HP，持续5秒的 buff

【实现】

​	宠物 在自己的job中访问并检测别人的 Component 【 读取别人的 Component】的值

​	宠物 在自己的job中访问修改别人的 Component【写别人的 Component】的值

​	宠物 在自己的job中访问给别人的 DynamicBuffer 添加元素【增加别人DynamicBuffer 的元素】

​	主人 遍历并且修改自身 DynamicBuffer中的元素【修改自己的DynamicBuffer 的元素】

​	主人 根据条件，移除某些符合条件的 DynamicBuffer 的元素【删除自己的 DynamicBuffer 的元素】

2、宠物检测到主人被销毁的时候，宠物也会将自身销毁

【实现】

​	宠物 在自己的Job中检测别人是否被销毁【检测别人的 Entity 是否被销毁】

​	宠物 在自己的job中销毁自己【在自己的job中销毁自己】

3、主人在被销毁的时候，会将所有自己的宠物销毁

【实现】

​	主人 非SystemStateComponent 检测摧毁的方法

​	主人 在自己的job中摧毁别人【在自己的job中销毁别人的Entity】



```c#

// Cd = ComponentData
[BurstCompile]
public struct CdHp : IComponentData
{
	public float value;
}

[BurstCompile]
public struct CdPet : IComponentData
{
	public Entity ownerEntity;
}

//BE = BufferElementData
//宠物列表
[BurstCompile]
public struct BEPet : IBufferElementData
{
	public Entity petEntity;
}

//Buff列表
[BurstCompile]
public struct BEBuff : IBufferElementData
{
	public float preRecoverHpTime;
	public float remainLifeSeconds;
}

[BurstCompile]
public struct CdWatchOwnerHPAbilityTag : IComponentData
{

}

[BurstCompile]
public struct CdWatchOwnerDestroyAbilityTag : IComponentData
{

}


[BurstCompile]
public struct HeroTag : IComponentData
{
}

[BurstCompile]
public struct DestroyTag : IComponentData
{
}


public partial class QueryJobSystem2 : SystemBase
{

    //面向批次的 Job
    [BurstCompile]
    private struct PetProcessAddBuffLogicJob : IJobEntityBatch
    {
        [ReadOnly] public ComponentTypeHandle<CdPet> petsHandle;
        [ReadOnly] public EntityTypeHandle eHandle;
        
        //内存级别的随机访问
        //把 Entity 传入后 可以得到 CdHp 组件
        [ReadOnly] public ComponentDataFromEntity<CdHp> hpLookUp;
        
        //在自己的Job中 设置别人的数据
        //EntityCommandBuffer 功能：在 Job中缓存指令
        //SetComponent 【不会立即执行】 只是会把API调用转换为一条指令  等出了job之后才会执行
        public EntityCommandBuffer.ParallelWriter ecb;
        
        
        //查询别人的 DynamicBuffer
        [ReadOnly] public BufferFromEntity<BEBuff> buffListLookUp;

		//Execute是乱序的 多个Job并发Execute 所以写入EntityCommandBuffer是乱序的，所以需要传入 batchIndex
        void IJobEntityBatch.Execute(ArchetypeChunk batchInChunk, int batchIndex)
        {
            //判断是否有爪子
            var pets = batchInChunk.GetNativeArray(petsHandle);
            var es = batchInChunk.GetNativeArray(eHandle);
            for(int bIndex=0; bIndex < batchInChunk.Count; bIndex++)
            {
            	var pEntity = es[bIndex];
            	var pet = pets[bIndex];
				
				//随机访问
				//在Query中没有加入 但是要怎么获取其他没有在Query中加入的组件
				if(hpLookUp.TryGetComponent(pet.ownerEntity, out var ownerHpComp))
                {
                	if(ownerHpComp.value < 50)
                    {
                    	//恢复Hp
                    	//注意 无法直接赋值 值类型值拷贝 无法生效 需要回填
                    	ownerHpComp.value = 50;
                    	//在自己的Job中设置别人的属性 使用 EntityCommandBuffer
                    	ecb.SetComponent(batchIndex, pet.ownerEntity, ownerHpComp);
                    	
                    	//+buff
                    	if(buffListLookUp.TeyGetBuffer(pet.ownerEntity, out var ownerBuffList))
                        {
                          ecb.AppendToBuffer(batchIndex, pet.ownerEntity, new BEBuff()
                          {
                              remainLifeSeconds = 5f
                          });
                        }
                    }
                }
            }

        }
    }
    
    
    //面向批次的 Job
    [BurstCompile]
    private struct TickBuffJob : IJobEntityBatch
    {
        public float dt;
        public float time;
        public BufferTypeHandle<BEBuff> buffsListHandle;
        public ComponentTypeHandle<CdHp> hpsHandle;
        
        void IJobEntityBatch.Execute(ArchetypeChunk batchInChunk, int batchIndex)
        {
        	//Acc装满了DynamicBuffer
        	var buffsListAcc = batchInChunk.GetBufferAccessor(buffsListHandle);
        	var hps = batchInChunk.GetNativeArray(hpsHandle);
        	for(int bIndex=0; bIndex < batchInChunk.Count; bIndex++)
            {
            	var buffsList = buffsListAcc[bIndex];
            	for(int i=buffList.Length-1; i>=0; i--)
                {
                	//把值类型的引用返回出来
                	ref var buff = ref buffsList.ElementAt(i);
                	buff.remainLifeSeconds -= dt;
                	if(buff.remainLifeSeconds < 0)
                    {
                    	buffsList.RemoveAt(i);
                    }
                    else
                    {
                    	if(time - buff.preRecoverHpTime > 1)
                        {
                        	//+5hp
                        	buff.preRecoverHpTime = time;
                        }
                    }
                }
            }
        }
        
        
    }
    
    
    //面向批次的 Job
    [BurstCompile]
    private struct PetProcessOwnerDestroyLogicJob : IJobEntityBatch
    {
    	[ReadOnly] public ComponentTypeHandle<CdPet> petHandle;
    	public StorageInfoFromEntity storageInfoFromEntity;
        
        void IJobEntityBatch.Execute(ArchetypeChunk batchInChunk, int batchIndex)
        {
        	for(int bIndex=0; bIndex < batchInChunk.Count; bIndex++)
            {
            	var pet = pets[bIndex];
            	var petOwnerEntity = pet.ownerEntity;
            	if(petOwnerEntity == Entity.Null || !storageInfoFromEntity.Exists(petOwnerEntity))
                {
                	//主人被摧毁
                	ecb.DestroyEntity(batchIndex, es[bIndex]);
                }
            }
        }
        
    }
    
    
    //面向批次的 Job
    [BurstCompile]
    private struct HeroProcessDestroyLogicJob : IJobEntityBatch
    {
    	
    	public EntityCommandBuffer.ParallelWriter ecb;
    	[ReadOnly]public BufferTypeHandle<BEPet> petsListsHandle;
    	public StorageInfoFromEntity storageInfoFromEntity;
    	[ReadOnly] public EntityTypeHandle eHandle;
    	
        
        void IJobEntityBatch.Execute(ArchetypeChunk batchInChunk, int batchIndex)
        {
        	var es = batchInChunk.GetNatvieArray(eHandle);
        	var hasPetsLists = batchInChunk.Has(petsListsHandle);
        	var petsAcc = hasPetsLists ? batchInChunk.GetBufferAccessor(petsListsHandle) : default;
        	
        	for(int bIndex = 0; bIndex < batchInChunk.Count; bIndex++)
            {
            	if(hasPetsLists)
                {
                  var pets = petsAcc[bIndex];
                  for(int i=0; i<pets.Length; i++)
                  {
                      var petEntity = pets[i].petEntity;
                      if(petEntity != Entity.Null && storageInfoFromEntity.Exists(petEntity))
                      {
                          //在自己的Job 中给别人添加组件
                          //ecb.AddComponent<CdDestroyTag>(batchIndex, pets[i].petEntity);
                          ecb.DestroyEntity(batchIndex, pets[i].petEntity);
                      }
                  }
                }
                ecb.DestroyEntity(batchIndex, es[bIndex]);
            }
        }
        
    }


	private EntityQuery watchOwnerHpPetQuery;
	
	private EntityQuery buffListOwnerQuery;
	
	private EntityQuery watchOwnerDestroyPetQuery;
	
	private EntityQuery needDestroyHerosQuery;
	

	protected override void OnCreate()
    {
    	using(var builder = new EntityQueryDescBuilder(Unity.Collections.Allocator.Temp))
        {
        	builder.AddAll(typeof(CdPet));
        	builder.AddAll(typeof(CdWatchOwnerHPAbilityTag)));
        	builder.FinilizeQuery();
        	
        	watchOwnerHpPetQuery = GetEntityQuery(builder);
        }
        
        using(var builder = new EntityQueryDescBuilder(Unity.Collections.Allocator.Temp))
        {
        	builder.AddAll(typeof(BEBuff));
        	builder.AddAll(typeof(CdHp));
        	builder.FinilizeQuery();
        	
        	buffListOwnerQuery = GetEntityQuery(builder);
        }
        
        using(var builder = new EntityQueryDescBuilder(Unity.Collections.Allocator.Temp))
        {
        	builder.AddAll(typeof(CdPet));
        	builder.AddAll(typeof(CdWatchOwnerDestroyAbilityTag)));
        	builder.FinilizeQuery();
        	
        	watchOwnerDestroyPetQuery = GetEntityQuery(builder);
        }
        
        
        using(var builder = new EntityQueryDescBuilder(Unity.Collections.Allocator.Temp))
        {
        	builder.AddAll(typeof(CdHeroTag));
        	builder.AddAll(typeof(CdDestroyTag));
        	builder.AddAny(typeof(BEPet));
        	builder.FinilizeQuery();
        	
        	needDestroyHerosQuery = GetEntityQuery(builder);
        }
    }

	protected override void OnUpdate() 
	{ 
		//不能两个Job写入同一个 ecb，Clear 复用也不可行，直接new就可以
		var ecb = new EntityCommandBuffer(Allocator.TempJob);
		var petProcessAddBuffLogicJob = new PetProcessAddBuffLogicJob()
        {
        	ecb = ecb.AsParallelWriter(),
            eHandle = GetEntityTypeHandle(),
            petsHandle = GetComponentTypeHandle<CdPet>(true),
            //一般都是true 然后用 ecb去修改值
            hpLookup = GetComponentDataFromEntity<CdHp>(true),
            buffListLookUp = GetBufferFromEntity<BEBuff>(true),
        };
		petProcessAddBuffLogicJob.ScheduleParallel(watchOwnerHpPetQuery).Complete();
		if(ecb.ShouldPlayback)
        {
        	//让ecb的修改生效
        	ecb.Playback(EntityManager);
        }
        ecb.Dispose();
        
        
        var tickBuffJob = new TickBuffJob()
        {
        	dt = Time.DeltaTime,
        	time = (float)Time.ElapsedTime,
        	buffsListHandle = GetBufferTypeHandle<BEHandle>(false),
        	HpsHandle = GetComponentTypeHandle<CdHp>(false),
        };
        tickBuffJob.ScheduleParallel(buffListOwnerQuery).Complete();

        
        var ecb2 = new EntityCommandBuffer(Allocator.TempJob);
		var petProcessOwnerDestroyLogicJob = new PetProcessOwnerDestroyLogicJob()
        {
        	ecb2 = ecb2.AsParallelWriter(),
            eHandle = GetEntityTypeHandle(),
            petsHandle = GetComponentTypeHandle<CdPet>(true),
            buffListLookUp = GetStorageInfoFromEntity(),
        };
		petProcessOwnerDestroyLogicJob.ScheduleParallel(watchOwnerDestroyPetQuery).Complete();
		if(ecb2.ShouldPlayback)
        {
        	//让ecb的修改生效
        	ecb2.Playback(EntityManager);
        }
        ecb2.Dispose();
        
        
        
        var ecb3 = new EntityCommandBuffer(Allocator.TempJob);
		var heroProcessDestroyLogicJob = new HeroProcessDestroyLogicJob()
        {
            eHandle = GetEntityTypeHandle(),
            petsListHandle = GetBufferTypeHandle<BEPet>(true),
            storageInfoFromEntity = GetStorageInfoFromEntity(),
        };
		heroProcessDestroyLogicJob.ScheduleParallel(needDestroyHerosQuery).Complete();
		if(ecb3.ShouldPlayback)
        {
        	//让ecb的修改生效
        	ecb3.Playback(EntityManager);
        }
        ecb3.Dispose();
        
	}
  
}


```



# System

2种 System 的方式

方式1：继承 SystemBase

```c#
public partial class JobSystem : SystemBase
{
	protected override void OnUpdate()
    {
    }
}
```



方式2：struct 继承 ISystem接口

优点：函数是高性能函数

缺点：struct 不能继续被继承

```c#
[BurstCompile]
public partial struct JobSystem : ISystem
{
	[BurstCompile]
	void ISystem.OnCreate(ref Unity.Entities.SystemState state) { }
	
	
	[BurstCompile]
	void ISystem.OnDestroy(ref Unity.Entities.SystemState state) { }
	
	
	[BurstCompile]
	void ISystem.OnUpdate(ref Unity.Entities.SystemState state) { }
	
	
}
```





 























