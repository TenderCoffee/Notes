# 手游性能问题 



## CPU-bound（CPU受限）

CPU 执行的 21ms 中

1、CPU执行引擎代码

2、逻辑代码

3、准备提交渲染数据到GPU

4、GPU开始工作的同时 CPU进入下一帧

（重复以上工作）

假如：GPU 提前完成了上一帧的渲染，CPU迟迟没有准备好这一帧需要提交的数据，造成了 GPU等待（Wait）



出现 CPU 受限主要有以下原因：

1、逻辑代码过于耗时

2、Application.targetFrameRate 限帧



![](images\CPU-bound.png)



## GPU-bound（GPU受限）

CPU 卡住了，GPU 等待



GPU-bound：

GPU 上一帧的计算迟迟没有结束

CPU 即使准备好提交 GPU的数据 也得等着



Gfx.WaitForPresent in profiler



![](images\GPU-bound.png)





最完美的结果是 CPU 和 GPU 都不需要等待对方

缺点：帧率过高会导致机器发热

需要适当降帧



# DOD编程：面向数据设计

OOP：面向对象设计。GameObject + OOP 会造成大量对象存在

DOD：面向数据设计，对机器友好，对内存利用率高，可以用于大量游戏对象（万人同屏）



# DOTS：面向数据技术栈

DOTS ：面向数据技术栈，具体 DOD 的解决方式

充分利用多核处理器，多线程处理 让游戏的运行速度更快更高效

由三部分构成：C# Job System、Burst 编译器、实体组件系统（ECS）

![](images\DOTS性能数据.png)



**CPU 缓存友好 + SIMD + 多线程 + struct去掉GC + LLVM burst 编辑器优化**



## 1. Burst Compiler：新编译器

以前：需要转换为中间代码，比较慢

Burst 编译器：不需要经过中间代码，直接变成汇编语言/机器码，效率更高

生成高度优化的本地代码



## 2. Job System：多线程



以前：单线程。一个CPU有很多的核，但是只有一个主线程在使用，其他的核都处于空闲的状态

Job System：多线程编程。当海量数据来临的时候，CPU 所有的核心都在一起工作，系统效率得到提高。Unity 内部 处理了 死锁 的问题。



## 3. ECS （Entity/Component/System）



**ECS：优化缓存行么，提高缓存命中率**

ECS 将所有相同的组件在内存中都排列在一起



性能强悍，展示海量游戏对象（百万同屏）



以前：基于对象的运作，如：每一个GameObject 都有自己的刚体、渲染组件组成，在内存中都是分散的，同一时间对大量游戏对象进行内存处理的话效率很低。

ECS：流水线式运作，在实例化实体的时候，都是在内存中紧密的排列，可以海量数据的处理





# HPC# 高性能C#

IL2CPP 与 Net Core 的效率相当 但是依然比 C++ 慢2倍

Unity 使用 Burst 编译后，可以让 C# 代码运行效率比 C++更快



操作原生：

1、C# class 类型数据的内存分配在堆上，程序员无法主动释放，必须等到 .Net 垃圾回收才可以真正的清理

2、IL2CPP虽然可以将 IL 转成 C++ 代码，但是 垃圾回收 使用的是 Boehm（贝姆），所以效率不等同于 C++

3、HPC# 就是 NativeArray<T> 可代替数组 T[] 数据类型包括值类型（float、int、uint、short、bool ...），enums，structs 和 其他类型的指针

4、NativeArray、NativeList 系列 API可以在 C# 层分配 C++ 中的对象，可以主动释放不需要进行 C#  的垃圾回收

5、Job System 中使用的就是 NativeArray



# 面向数据的技术栈

Unity 通过 5个核心包定义 + 额外包 的全新的Unity代码编写模型



# 5个核心包



## Job System 

​	多线程代码 更容易利用多核处理并行任务



## Burst

​	优化C#代码的编译器，比 Mono、IL2CPP 更快，不止可以编译DOTS代码，也可以编译Unity中的任何代码



## Mathematics

​	可以在 Job System 中使用的数学库 经过特别优化



## Collections

​	常见集合类 的内存分配 属于非 C# 的托管类型，可以在 Burst 编译代码中的 Job System 中使用，并且这些类型支持安全检查（能在 Job System 中安全使用）



## Entities（ECS）

​	Entity 对象比 GameObject 更轻量 更高效的替代品，本身不承担任何代码，

​	Component 只是数据片段集合

​	E、C 都由对应的 System 代码单元进行处理



# 扩展包

## Entities.Graphics

​	即 DOTS 1.0 之前的 Hybrid Renderer

​	支持 URP 与 HDRP 的 Entities 的渲染解决方案

​	注意：这个包 不优化 GPU 的性能，而是优化 CPU 上的 性能



## NetCode

​	建立五个核心包基础上的 - DOTS 网络解决方案

​	网络多人连线的服务器功能，与客户端预测等功能



## Physics

​	建立五个核心包基础上的 - 物理解决方案

​	支持2个后端：

​	1、默认 Unity Physics 包，无状态的确定物理库（适合网络多人游戏）

​	2、Havok Physics 有状态但没有确定性的物理库，但更稳定和功能更强大



## Animation(WIP)

支持 DOTS



## Audio(WIP)

支持 DOTS



# DOTS 用在哪里（重要）

多线程加载、通讯、充分利用多核并行计算的游戏类型



1、具有大世界流式加载的游戏

2、具有复杂的大规模模拟的游戏

3、具有多种网络类型的多人联线游戏

4、具有需要客户端模拟预测的网络游戏（FPS游戏）



# 为什么需要 DOTS

程序性能的提升 远没有硬件提升的快 游戏主机硬件是固定的硬件 DOTS 提供了一套更简单的面向数据的代码编写模型

DOTS 编写出的程序充分利用现代CPU的多核并行设计



## 很多游戏的性能瓶颈不在渲染

很多游戏低分辨率用最新显卡也没有帧率提升。因为性能瓶颈在 CPU。

目前计算机硬件，CPU、GPU、内存 的发展不均衡，带宽限制

GPU 有独显的显存，但 CPU 和 内存 是添加高速缓存的内存层级结构去弥补 （ L1、L2、L3）

虽然有 高速缓存，但程序的设计不合理，还是会导致 Cache 使用的低效，会导致大型缓存收益效益递减



## 面向数据设计的本身是面向缓存友好的

可以极大增加缓存 Cache 的命中，提高效率



## 程序性能的提升 远没有硬件提升的快

摩尔定律的延续 依赖于 越来越好的工艺（2纳米）

提高CPU的速度主要依赖于提高主频、提高功率的时候降低发热，在占用空间越来越小的时候通过增加核数做并行处理提高处理能力

需要做指令并行，改进编译技术的支持才能发挥现代CPU设计的真正效率



## 其它并行编程库 - 依赖特定的硬件、针对科学计算

Intel TBB、OpenGL CUDA、OpenCL、MPI/OpenMPI 依赖特定的硬件、针对科学计算

集成到Unity中几乎不可用

DOTS 充分考虑游戏设计的需求，在 Unity 中兼容多平台多硬件的支持

