# Profiler 分类

Profiler 使用 54998 - 55511 端口广播分析数据

注意防火墙的拦截



## 指令注入

观察目标函数调用行为，分配的内存

方法：

​	启用附加的编译标志 

​	

缺点：

​	1、不高效，因为 启用附加的编译标志 会有额外的 性能消耗

​	2、Unity 编辑器本身有消耗CPU 和 内存

​	

## 基准分析

在目标硬件测试收集关注点（渲染帧率（FPS）、总体内存消耗、CPU峰值、CPU/GPU 温度）



分析目标硬件上独立运行的项目需要启用 Development Build 和 Autoconnect Profiler 标志

Profiler - Connected - 连接要测试的硬件设备

​	

### IOS 设备

1、应用程序启用  Development Build 和 Autoconnect Profiler 标志

2、将 IOS 和 Mac 设备 连接到本地 WiFi

3、通过 USB 将 IOS 设备连接到 Mac 上

4、Build & Run 选项构建应用程序

5、Unity - Profiler - Connected Player - 选择设备



### Android 设备

方式一：WiFi

1、应用程序启用  Development Build 和 Autoconnect Profiler 标志

2、将 Android 和 Mac/Widnows 连接到本地 WiFi

3、通过 USB 将 Android 连接到 Mac/Widnows

4、Build & Run 选项构建应用程序

5、Unity - Profiler - Connected Player - 选择设备



方式二：ADB

1、安装 Android SDK

2、通过 USB 将 Android 连接到 Mac/Widnows

3、应用程序启用  Development Build 和 Autoconnect Profiler 标志

4、Build & Run 选项构建应用程序

5、Unity - Profiler - Connected Player - 选择设备



## 建议

1、先基准分析，再研究注入的指令

2、不在编辑器模式下运行得到基准数据

3、将分析工具放到应用程序中



# Unity Profiler



## Deep Profiler

无差别统计整个调用栈

会更深层次指令重新编译脚本统计每个调用的方法，需要真用大量内存

大型项目不使用 Deep Profiler 



## CPU使用情况 区域

最复杂、最有用

三种模式（Hierarchy、Raw Hierarchy、Timeline）



### FPS - 基准

30 FPS = 33毫秒完成整个帧的内容

60 FPS = 16.667 毫秒完成整个帧的内容



### 辅助工具：.Net 的 Stopwatch 计时

自定义计时工具 

精度不完全准确

1、不能用在不同调用之间差异很大的场景

2、重复请求相同内存块导致认为提高缓存命中率也会降低平均时间

3、人为因素导致 JIT编译被隐藏，JIT只影响方法的第一次调用



**注意：监视 CPU 峰值的时候需要关闭 垂直同步（VSync）**

Edit - Project Settings - Quality

Vsync会将应用程序的帧率匹配到显示到的设备帧率：如果显示器帧率是60Hz，游戏渲染>60Hz，则游戏会等待直到输出渲染帧

作用：防止了屏幕撕裂

启用 Vsync 对性能检测的影响：

因为在编辑器模式下窗口很小，不需要很多CPU和GPU来渲染 ，Vsync开启后游戏会为了故意降低速度，所以会出现错误的CPU 峰值（**WaitForTargetFPS**）



### Hierarchy模式 - 第一步

显示调用栈的调用，确定执行哪个函数调用花费更多CPU时间



### Raw Hierarchy模式

会将全局 Unity 函数调用隔离到单独的条目，而不是合并到一个大条目中

1、统计某个全局方法调用多少次

2、确定这些调用的某次调用比预计消耗更多GPU 和 内存



### Timeline 模式

对CPU使用区域最有效的模式



Unity Job System：主线程、渲染线程、各种后台工作线程

用于加载场景和资源等活动

可以用来评估导致性能问题的最大原因

宽度类似 = 消耗处理时间类似



假设：预算游戏需要达到60FPS性能

60FPS = 0.06

1/0.06 ≈ 16.667

即：不能超出16.667毫秒的预算

如果发现有MonoBehavior 组件超出 16.667ms 则可以视为有性能问题



Main Thread 可能是感兴趣的内容



## GPU使用情况 区域

相关Unity调用、摄像机、绘制、不透明、透明的集合图形、光照和阴影 等 花费时间



## 渲染 区域

GPU 内部处理



CPU 上为渲染而准备的相关活动：

​	SetPass调用数量（DrawCall）

​	渲染到场景的批次总数

​	动态批处理和静态批处理节省的批次数量和生成方式

​	纹理的内存消耗



**使用 Frame Debugger**



## 内存 区域



### Simple 模式

Unity 底层引擎

Mono框架（由垃圾回收管理的整个堆的大小）

图形资源

音频资源

缓冲区

保存 Profiler 收集的数据的内存



### Detail 模式

每个GameObject 和 MonoBehavior 为 Native 和 Managed 表示所消耗的内存

Take Sample：手动采样



## 音频 区域

音频系统的 CPU 消耗

所有播放中或暂停的音频 和 音频剪辑的总内存消耗



音频需要大量潜在的磁盘访问和 CPU 处理



## 物理 区域

Physics 3D - Nvidia 的 PhysX

Physics 2D - Box2D



Physics Debugger



## UI 和 UI 详情 区域

洞察 使用 内建 UI 系统的应用程序



### 全局光照 区域

使用 GI 的前提下可以用于洞察性能

