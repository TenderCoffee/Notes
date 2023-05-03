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

