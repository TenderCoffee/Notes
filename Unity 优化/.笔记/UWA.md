# 名词解释



## 帧率

测量单位 - 每秒显示帧数、FPS（**f**rame **p**er **s**econd）



### 画面撕裂

显示卡效能大幅提高，帧率太高出现画面撕裂。屏幕更新频率是固定的，通常是60Hz，如果显示卡的输出高于60fps，**两者不同步，画面便会撕裂**。通常游戏内选项内的垂直同步(V Sync)开启后便可解决画面撕裂的问题。



## VSS - 虚拟耗用内存

VSS：Virtual Set Size，虚拟耗用内存。它是一个进程能访问的所有内存空间地址的大小。这个大小包含了一些没有驻留在RAM中的内存，就像mallocs已经被分配，但还没有写入。VSS很少用来测量程序的实际使用内存。



## RSS - 实际使用的物理内存

RSS：Resident Set Size，实际使用物理内存。RSS是一个进程在RAM中实际持有的内存大小。RSS可能会产生误导，因为它包含了所有该进程使用的共享库所占用的内存，一个被加载到内存中的共享库可能有很多进程会使用它。RSS不是单个进程使用内存量的精确表示。



## PSS - 实际使用的物理内存（按比例分配共享库所占用的内存）

PSS：Proportional Set Size，实际使用的物理内存，它与RSS不同，它会按比例分配共享库所占用的内存。例如，如果有三个进程共享一个占30页内存控件的共享库，每个进程在计算PSS的时候，只会计算10页。PSS是一个非常有用的数值，如果系统中所有的进程的PSS相加，所得和即为系统占用内存的总和。当一个进程被杀死后，它所占用的共享库内存将会被其他仍然使用该共享库的进程所分担。在这种方式下，PSS也会带来误导，因为当一个进程被杀后，PSS并不代表系统回收的内存大小。



## USS - 进程独自占用的物理内存

USS：Unique Set Size，进程独自占用的物理内存。这部分内存完全是该进程独享的。USS是一个非常有用的数值，因为它表明了运行一个特定进程所需的真正内存成本。当一个进程被杀死，USS就是所有系统回收的内存。USS是用来检查进程中是否有内存泄露的最好选择。



## Shader 变体

在Unity上写的shader其实并不是真正的shader语言，而是Unity提供给我们的中间的抽象层

代码重复性太高：两个效果差不多的shader ，代码只相差一行或者几行

解决：宏定义

```text
//可以将两个差不多效果的shader写在一个文件了
//既减少了重复代码的编写，代码也比较清晰
#ifdef UNITY_UI_ALPHACLIP
clip (color.a - 0.001);
#endif
```

Unity 的 Shader 中的宏定义：

两个关键字来定义宏 **multi_compile** 和 **shader_feature**

**Unity 会根据宏定义来生成不同种类的shader，这些不同种类的shader就是所谓的变体**



**在代码中调用 EnableKeyword = 动态启用变体**



### multi_compile

缺点：

1、multi_compile 会造成组合爆炸

```
#pragma multi_compile A B
#pragma multi_compile C D
//生成4种变体 （A,C） (A,D) (B,C) (B,D)
//multi_compile定义的越多，生成的变体组合数就会非常非常多
```

2、不论是否代码中调用EnableKeyword，变体都会打进包中

multi_compile 控制不好会导致打包慢，还生成很多没用的shader



### shader_feature

优点：

没用到的shader关键字不会生成变体被打进包中，也就不会造成冗余资源浪费，生成没用的shader



缺点：

使用 动态启用变体（代码调用 EnableKeyword）会因为没有直接饮用而导致想要的变体没有打进去，最后找不到对应的 Shader



### 解决方案

*1、使用multi_complie 代替 shader_feature，multi_complie 会把所有变体build出来；*

*2、把这个Shader加入“always included shaders”中 (Project Settings -> Graphic)；*

*3、创造一个使用变体B的Material，强行说明变体B有用；*

*4、将用到的变体用ShaderVariantCollection收集起来*



### 变体生效

1、材质中勾选

[Toggle(UNITY_UI_ALPHACLIP)] _UseUIAlphaClip ("Use Alpha Clip", Float) = 0

UNITY_UI_ALPHACLIP 就是变体的宏定义，在材质中可以勾选使用

2、代码调用 Material.EnableKeyword 或者 Shader.EnableKeyword 启用



### 变体收集(ShaderVariantCollection)

**1、可以shader_feature类型的变体收集，以避免打不进包中**

**2、可以通过ShaderVariantCollection.WarmUp();进行预加载，避免卡顿。**





# 性能标准





## 耗时推荐值 - 帧率

流畅性最直接的指标就是帧率（即时战斗的动作类游戏更高要求）

需要将CPU耗时控制住一定的数值范围内（30帧 = 总耗时33ms以下）

![img](https://public1.uwa4d.com/toc/edu/content/2.jpg)



## 内存推荐值 - PSS

USS = 进程独占的内存

RSS = USS + 共享内存

PSS = USS + 共享内存/共享这段内存的进程数量



避免闪退的重点 - 控制 PSS 内存峰值

PSS 内存的大头：Reserved Total中的资源内存 和 Mono堆内存

Lua的项目：除了上面两个还需要额外关注Lua 内存



当PSS内存峰值控制在硬件总内存的0.5-0.6倍以下的时候，闪退风险才较低

如：2G 设备内存 - PSS 应该控制在 1G 以下



大多数项目，PSS 内存大约高于Reserved Total 200MB-300MB左右

即：2G 设备的Reserved Total应控制在700MB以下

3G设备则控制在1G以下

![img](https://public1.uwa4d.com/toc/edu/content/3.jpg)



## 渲染模块推荐值

画质效果

GPU的压力会受到帧率，分辨率，三角形面片数，后处理，Shader复杂度，Overdraw等多方面的影响

在不同档位的机型上要做较为细致的区分（绿色的数值如果能达到的话会更优）

![img](https://public1.uwa4d.com/toc/edu/content/4.jpg)



# 常见的优化工具



## 1、Unity Profiler

![uwa_1.png](.\images\uwa_1.png)



构建到目标平台

Unity 端设置：

1、勾选 Development Build：在打包的应用中画面右下角有水印

2、勾选 Autoconnect Profiler（可选）：Autoconnect Profiler 勾选后增加了10s 的初始启动时间，可以自动连接到Profiler

有线 和 无线 连接：





Android端设置：

1、使用 ADB：开启开发者模式，使用线缆将设备连接至计算机

2、远程连接：连接 WIFI 网络，手机与计算机位于同一个子网即可



### 最常使用和最关注的三个：CPU、渲染、内存

CPU：分析可以得到在不同方面的耗时，定位性能瓶颈

渲染：找到带来渲染压力的因素

内存：各类资源的内存使用情况，判断某些内存是否过大



### CPU 模块

#### (Raw)Hierarchy - 原始层级结构

​	列出已进行性能分析的所有样本，并被共享的调用栈和 Profiler Marker 层级视图将样本一起分组（Raw Hierarchy 不会）



#### Timeline - 时间轴

​	包含应用程序中时间花费的位置以及时间之间的关系的概述

​	显示特定帧的时间细分信息以及该帧长度的时间轴

​	可以一次性查看所有线程上的时间以及在帧内运行的线程的时间，可以关联各个线程的时间

​	![](.\images\uwa_4.png)



​	快捷键：
​		A：快速将当前帧的整体时长自适应到窗口中间

​		F：将选中的具体项进行聚焦，缩放至较为合适的大小（方便查看较短时间）

​		鼠标滚轮键：缩放视图大小，按住滚轮键：左右平移视图

​	

##### 	Gfx.WaitForPresentOnGfxThread

​	主线程准备好，但主线程一直在等待渲染线程完成上一帧的工作，造成帧率下降

##### 	UniversalRenderPipeline.RenderSingleCamera

​	CameraLoop 花费耗时，后处理效果，Bloom 辉光效果



#### Hierarchy - 层级结构

​	Total：Unity 在特定函数上花费的总时间百分比

​	Self：Unity 在除去调用子函数的特定函数上花费的总时间百分比

​	Calls：此帧中调用此函数的次数

​	GC Alloc：垃圾收集

​	Time ms：总时间

​	Self ms：除去子函数的总时间



右上角开启额外的数据可以进一步了解应用程序在何处调用和使用了接受性能分析的函数

![](.\images\uwa_2.png)



切换别的线程：渲染线程、工作线程 等等

![](.\images\uwa_3.png)



### Rendering 渲染 模块

本质：相机视角的移动造成需要渲染的内容发生变化



**衡量场景中不同区域的资源强度，这些统计信息 对优化很有用**

Batches Count：批次数

SetPass Calls Count：切换通道的次数

Triangle Count：三角形数

Vertices Count：顶点数



![](.\images\uwa_5.png)



详细信息面板显示更多的渲染统计信息：

**Open Frame Debugger：查看渲染帧的各个绘制调用的信息**

![](.\images\uwa_6.png)

Rendering Statistics 窗口：

![](.\images\uwa_7.png)



### Memory 内存模块

应用程序中分配的总内存的计数器

可以看到每一帧的GC分配数量



![](.\images\uwa_8.png)

Total Used Memory：使用的内存量

Texture Memory：纹理内存量

Mesh Memory：网格内存量

Material Count：材质实例数

Object Count：原生对象实例数

GC Used Memory：GC 的内存量

GC Allocated in Frame：GC 堆中每帧分配的内存量



**详细的数据：Simple 视图 和 Detailed 详细视图**

#### Simple 

显示了 Unity 如何按帧实时使用内存的概述，将应用程序使用的总内存分成了几个主要类别

​		Total：基于 System Used Memory Profiler 计数器，指示操作系统报告的应用程序正在使用多少内存

​		找到占据最高内存的模块



#### Detailed 

查看具体的使用情况

​		应用程序当前的快照

​		Take Sample：获得某一帧内存的详细信息，展开可以具体到每一个资源文件

​		Unity编辑器中进行分析，Play Mode 下的警告 = 真实的内存使用与实际不一致

![](.\images\uwa_9.png)



### 其他模块

Audio 音频：可跟踪应用程序中花费在音频上的时间

Video 视频：显示有关应用程序中的视频所用资源的信息

Physics 物理：物理系统在项目场景中处理过的物理信息



## 2、 FrameDebugger - 检查合批

检查Batch 是否正常合批，检查不合批的原因

能够将正在运行的游戏的状态冻结到特定帧来自由回放，并查看用于渲染该帧的各个 **DrawCall绘制调用**

还可以逐个单步执行这些绘制调用，可以查看到场景是如何从场景的图形元素构建的



Enable - 点击后即可捕捉当前帧的画面

Editor：正在编辑器模式下的分析，也可以选择连接设备远程调试（需要支持多线程渲染）

![](.\images\uwa_10.png)



能够显示为什么无法进行合批的原因：（可以检查合批失败的情况）

![](.\images\uwa_11.png)



Preview 分页：可以查看顶点数、网格等信息

​	**顶点/索引数量可以间接衡量场景的渲染代价**

Shader 分页：可以查看使用的贴图、向量等数据，点击的如果是工程中的素材贴图（也有动态生成贴图的情况），则会直接定位，Ctrl+点击 可以预览贴图 

![](.\images\uwa_12.png)



## 3、Mali Offline Compiler - 计算Shader复杂度

Shader 对性能的影响非常重要，经常要在游戏视觉表现和性能之间做出平衡



静态分析的命令行工具，不需要了解项目的详细信息也不需要打包发布到真机，提供静态分析 GPU着色器

**只需要获取要测试的 Shader 片段，**选定目标 GPU 即可

可以用于验证着色器的语法、识别性能瓶颈、衡量任何变化对性能的影响



主要用来计算 Shader 的复杂度，结合分档的高中低数值来判断 Shader 是否过于复杂

![](.\images\uwa_13.png)



### 1、找到要测试的 Shader 文件

选择 GLES3x 作为编译语言，点击 Compile and show code 就可以得到这个API 对应的 Shader 代码

![](.\images\uwa_14.png)



### 2、搜索 Keywords 保存 shader 文件

从编译后的代码中找到真正使用的一段

 	1. 使用 Unity Frame Debugger 找到目标 Drawcall
 	2. 查看当前使用的 Keywords：DIRECTIONAL LIGHTPROBE_SH
 	3. 使用该 Keywords 就可以在编译后的Shader中 定位到实际使用的 Shader 片段
 	4. 使用 Keywords 进行搜索就可以找到多段对应不同硬件等级的代码
 	5. 将 #ifdef xxx 到 #endif 之间的代码复制出来，**分别保存为 .vert 和 .frag类型的文件**，注意不要包括 #ifdef xxx 和 #endif
 	6. **一般更加关注片元着色器的性能**



![](.\images\uwa_15.png)

​	![](.\images\uwa_16.png)



### 3、开始编译 Shader

到 cmd 命令行中进行测试

需要填写要测试的目标机型 的GPU型号，不填写的话会默认使用最新的一款

![](.\images\uwa_17.png)



### 4、测试分析报告

![](.\images\uwa_18.png)

####  **Configuration** - GPU 信息：

​		Hardware：GPU 的名称 

​		Architecture：GPU 的架构

​		Driver：GPU 的驱动

​		Shader type：当前的 Shader 的语言和种类

控制台输入：

查看更多该 GPU 的信息

```
malioc --info -c Mali-G71
```



#### Main Shader 报告分析

Work registers：这个 shader 工作使用的寄存器数量。可用的物理寄存器池在正在执行的着色器线程之间分配。因此，减少工作寄存器的使用可以增加可以同时执行的线程数，有助于保持GPU工作忙碌。

Uniform registers 是只读寄存器，用于存储着色器可能需要的常量以及时间常量。如果这里使用 100% 的话，就需要返回到每个线程的内存负载以获得常量值。

Stack spilling 是看是否有溢出到堆栈，如果有的话对 GPU 读取是性能消耗较大的。通过降低变量精度、减少变量的活动范围、简化着色器程序 可以降低寄存器压力。

16-bit arithmetic：以 16位或者更低精度执行的算术运算的百分比，数值 越高代表 shader 的性能越好，使用16位操作的速度是32位的两倍。即使在整体性能没有提高的情况下，更高比例的16位操作也可以提高能源效率。



性能表：给出单个着色器核心的着色器程序的潜在性能的指示

Total Instruction Cycles：为程序生成的所有指令的累计执行周期数，与程序控制流无关。

Shortest Path Cycles：通过着色器程序的最短控制流路径的周期数的估计。这一行根据设计中出现的功能单元的数量规范化周期成本。

Longest Path Cycles：通过着色器程序的最长控制流路径的周期数的估计。这一行根据设计中出现的功能单元的数量规范化周期成本。根据静态分析确定最长路径并不总是可能的，例如，如果一个统一变量控制一个循环迭代限制。所以这一行可能表示一个未知的循环计数（“N/A”）

![](.\images\uwa_19.png)





