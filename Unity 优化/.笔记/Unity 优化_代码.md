

# 性能定位步骤 - 代码

1、检查脚本是否存在场景中

​	在 Hierarchy 窗口中输入：

​	t:<monobehaviour name>

2、验证脚本执行次数

​	编写自定义编辑器辅助函数

3、验证事件顺序

​	IDE逐步调试

​	Debug.Log 打印 - 消耗大，对 CPU 和 内存有影响

​	注意：协程 的  WaitForSeconds yield 类型难以调试/不可预测

4、使用变量开关 控制调试日志输出

5、编辑器下调试过程中 禁用 Vsync

6、任务管理器检查后台没有其他进程消耗 CPU 周期和占用大量内存

​	内存不足会导致缓存丢失，应用程序对虚拟内存页文件交换的磁盘访问响应速度较慢

7、UnityEngine.Profiling.Profiler 的 BeginSample  和 EndSample

8、自定义计时工具 .Net 的 Stopwatch 类不完全准确，但是可以 多次运行相同测试/测试运行测试次数 = 平均时间

9、应用程序达到稳定状态后再开始测试，避开预热时间



# 优化



## 使用 GetComponent<T>

GetComponent<T> > GetComponent(typeof(T) > GetComponent(string)



## 移除空函数

MonoBehaviour 在场景中第一次实例化的时候，Unity 会将任何定义好的回调添加到一个函数指针列表中

即使是空函数，也会挂接到这些回调中

如果空函数分散在整个代码库中，调用会产生少量的CPU浪费

只要使用 GameObject.Instantiate 创建预设，空函数的存在（Start 、Awake）会使 场景初始化变慢，浪费CPU时间



Notepad++使用 正则表达式查找出空Update函数回调

```
void\s*Update\s*?\(\s*?\)\s*?\n*?\{\n*?\s*?\}
```



注意：基类的空函数调用也会影响



### 关于 GameObject 和 MonoBehavior

GameObject 和 MonoBehavior 是特殊对象

在内存中有两个表示：

1、存在管理 C#代码（编写的托管代码）的相同系统管理的内存中

2、存在于另一个单独处理的内存空间中（本机代码）



### 跨越本机-托管的桥接

数据可以在 托管代码 和 本机代码 之间移动，会产生额外的CPU开销和可能的额外内存分配以便跨桥复制，这需要垃圾收集器最终执行一些内存自动清理操作

#### 空引用检查 - gameObject != null

```c#
if(gameObject != null)
{
    //这里会产生桥接 的内存浪费
    //虽然有消耗，但项目中使用 利大于弊
}
```

#### 引用比较 - Object.ReferenceEquals

```c#
if(!System.Object.ReferenceEquals(gameObject, null))
{
    //这里会产生桥接 的内存浪费
}
```

#### GameObject 中取出字符串属性 - tag 和 name

```c#
for(int i = 0; i < listOfObjects.Count; i++)
{
    //tag属性这里会产生桥接 的内存浪费
    //解决：CompareTag 避免了 本机 - 托管桥接
    if(listOfObjects[i].tag == "Player")
    {
        //对对象执行了操作
    }
        
}
```



## 缓存组件引用

代价：少量的额外内存消耗



## 共享计算输出

如果多个位置调用一个消耗昂贵的操作总是得到同个输出，则可以将结果分发给其他需要这个结果的调用处

代价：牺牲代码简洁性 + 传递值的额外开销



## 控制调用频率

Update、Coroutines、InvokeRepeating



在Update回调中容易超出需要的频率重复调用某段代码（如 执行AI逻辑）

解决：将逻辑 转换成协程



### 关于协程

1、**开启协程的额外成本是标准函数的3倍**，会分配一些内存，将当前状态存储到内存中直到下一次调用

​	协程经常不断的调用yield 会造成相同开销成本，在 Update 降低频率 有时候比协程好处大

2、初始化协程后，协程的运行独立于 MonoBehaviour 组件的 Update回调的触发

​	不管组件是否禁用都会继续调用协程

3、协程会在包含它的 GameObject 变成不活动的时候停止，再次设置 GameObject 为活动，协程也不会自动重启

4、方法转换为协程可以减少大部分帧中的性能损失

​	但如果单次调用超出帧率预算则协程无效，因为调用次数的变化没办法解决问题

5、WaitForSeconds 是一个缩放的时间（受到 Time.timeScale 影响），不是一个精确的计数器 

​	WaitForSecondsRealTime - 未缩放的时间

​	WaitForEndOfFrame - 下一个 Update 结束时继续

​	WaitForFixedUpdate - 下一个 FixedUpdate 结束时继续

​	WaitUntil、WaitWhile - 根据委托返回 true 或 false 分别暂停或继续

6、简单的协程可替换为 InvokeRepating 调用

​	建立更简单，成本更小



### 关于 InvokeRepeating

1、完全独立于 MonoBehaviour 和 GameObject 的状态

2、CancelInvoke：停止

3、销毁关联的 MonoBehaviour 或者 父对象GameObject（禁用无法停止 ）



## 避免从GameObject中取出字符串属性 - tag和name属性

从对象中检索字符串属性、检索C#中任何其他引用类型属性：不增加内存成本

GameObject 中检索字符串属性也是 跨越本机-托管的桥接 的消耗

tag 和 name

解决：CompareTag 避免了 本机 - 托管桥接



## 合适的数据结构

数组：

​	列表 本质是 动态数组

​	在遍历中使用

​	数组中的元素，在内存中彼此相邻，在迭代查找中丢失CPU高速缓存 最小



字典：

​	两个对象相互关联，且希望快速获取、插入或删除关联



## 避免运行时修改 Transform的父节点

Transform 组件只存储与其父组件相关的数据

1、访问和修改 Transform 组件的 position、rotation、scale 会导致大量矩阵乘法计算

解决：

​	1、使用 localPosition、localRotation、localScale 成本较小，不需要任何额外的矩阵乘法

​	2、将 Vector3 数据缓存为变量，在 FixedUpdate（帧的末尾） 中提交对 position 的修改

2、修改 Transform 组件会向其他组件（Collider、Rigidbody、Light、Camera）发送 内部通知更新



## 避免在运行时使用 Find 和 SendMessage

Find   和 SendMessage 消耗大 甚至 可能丢失帧

使用 GetComponent 替换 SendMessage 



## 对象之间的通信

### Unity 内置的序列化系统

分配引用，将对象拖拽到组件引用的面板中



注意：不是所有对象都可以序列化到 Inspector 面板中

Unity 可以序列化所有的基本数据类型（int、float、string、bool）、各种内置类型（Vector3、Quaternion 等）、枚举、类、结构、包含其他可序列化的各种数据结构

但是无法序列化静态字段、只读字段、属性和字典



### 全局可访问静态类

单例设计模式

单例模式对管理共享资源或者繁重的数据流量（如文件访问、下载、数据解析和消息传递）非常有用

静态类的每个方法、属性和字段都必须加 static 关键字：在内存中永远只主流对象的一个实例



静态类字段的内部初始化

```c#
private static List<Enemy> _enemies = new List<Enemy>();
```



### 单例组件

提供静态方法授予全局访问权

注意：不需要在单例组件的类定义中实现静态方法



### 全局消息传递系统

消息广播



## 禁用未使用的脚本和对象

在 Update 回调中调用代码越多伸缩性越差，游戏越慢



### 通过可见性禁用对象

OnBecameVisiable：对任何一个摄像机可见

OnBecameInvisible：对所有摄像机不可见

可见性回调必须与渲染管线通信，因此GameObject 必须附加一个可渲染组件（MeshRenderer、SkinnedMeshRenderer）



### 通过距离禁用对象

AI 角色



## 使用平方计算

CPU 擅长浮点数相乘，不擅长计算平方根

使用 sqrMagnitude 属性替代 Distance 方法

```c#
float distance = (transform.position - other.transform.position).Distance();
if(distance < targetDistance)
{
    //
}

//替换为
float distanceSqrd = (transform.position - other.transform.position).sqrMagnitude;
if(distanceSqrd < (targetDistance * targetDistance))
{
    //
}

```



## 最小化反序列化行为

Unity 的序列化系统：场景、预制件、ScriptableObjects、各种资产类型

对象类型的保存：使用 YAML 格式将这些对象转换为文本文件，后面也可以将这些文件反序列化为原始对象类型



序列化文件：序列化数据捆绑在大型二进制数据文件中

反序列化：在运行时磁盘读取和反序列化数据（非常慢的过程）

​	Resources.Load 

​	反序列化的数据集越大，过程越长，预置组件的每个组件都是序列化的，层次结构越深需要反序列化的数据越多

​	加载大型序列化数据集在第一次加载时造成 CPU的峰值，运行时加载甚至可能导致掉帧



### 减小序列化对象

将序列化对象分割成更小的数据块

比如：UI 的加载 不需要一整个UI 一起加载，可以一次加载一个



### 异步加载序列化对象

Resources.LoadAsync



### 内存中保存之前加载的序列化对象

保存在内存中，实例化更多的预设副本

Resources.UnLoad



### 公共数据放到 ScriptableObject

组件有共享数据的属性（命中率、力量、速度等游戏设计值）

公共数据序列化到 ScriptableObject 中加载使用，可以减少存储在预置文件中的序列化数据量



## 叠加、异步加载和卸载场景

SceneManager.LoadScene：同步加载，阻塞主线程

异步叠加式加载：让场景逐渐加载

异步加载可以将加载分散到几个帧上



卸载场景：

SceneManager.UnloadScene：同步卸载

SceneManager.UnloadSceneAsync：异步卸载



## 创建自定义的Update层

不使用Unity自带的 MonoBehaviour 的 Update，最小化 Unity 需要跨越桥接的频率

使用接口：解耦

```c#
public interface IUpdateable{
	void OnUpdate(float dt);
}

public class UpdateableComponent : MonoBehaviour, IUpdateable
{
    //dt：时间增量 = Time.deltaTime
    public virtual void OnUpdate(float dt){ }
}

```

因为虚函数需要调用重定向，所以 虚函数的开销 比 非虚函数的略多

虽然有一定消耗，但这个虚函数的更新节省了大量的性能开销
