# OpenGL介绍



相关资料

OpenGL 3.3

https://registry.khronos.org/OpenGL/specs/gl/glspec33.core.pdf

OpenGL 官方网站：

https://www.opengl.org/

OpenGL 各个版本的规范和扩展

https://www.opengl.org/registry/



## OpenGL 是 规范 而不是 API

一般被认为是 API：有一系列可以操作图形、图像的函数

OpenGL  仅仅是一个由 Khronos组织制定的 **规范**



**严格规定每个函数如何执行以及输出值**

内部具体的每个函数的实现由 OpenGL 库的开发者自行决定

OpenGL 规范 没有规定实现的细节，允许使用不同的实现，只要功能和结果与规范相匹配



## 显卡的生产商 - OpenGL 库的开发者

显卡支持的OpenGL版本都为这个系列的显卡专门开发

OpenGL 库表现的行为与规范规定的不一致时，基本都是库的开发者留下的bug

解决：升级显卡驱动 = 更新了显卡能支持的最新版本的 OpenGL，解决Bug



## 立即渲染模式（旧） 和 核心模式（新）

立即渲染模式（即 固定渲染管线） - OpenGL 早期

OpenGL 大多数功能被库隐藏，开发者没有灵活性没有绘图细节掌控

效率太低

OpenGL 3.2 废弃立即渲染模式



**Core-profile 核心模式 - OpenGL 鼓励**

强制使用现代函数，使用废弃的函数会抛出错误停止绘图

更高的灵活性和效率，但更难学习

需要真正理解OpenGL 和 图形编程

新版本的OpenGL 都是在  **OpenGL 3.3 的基础**上引入额外功能没有改动核心架构，只是更有效率或更有用的方式去完成同样的功能

开发者：基于较低版本的OpenGL 编写程序，并提供选项启用新版本的特性



## OpenGL 的特性 - 显卡公司使用 扩展 实现更有效的图形

在驱动中实现扩展：显卡公司提出新特性或者渲染上大优化

扩展可以实现更先进更有效的图形功能，也不需要等待新的 OpenGL 规范面世就可以使用新的渲染特性



```
if(GL_ARB_extension_name)
{
	//使用硬件支持的全新的现代特性
}
else
{
	//不支持此扩展：用旧的方式去做
	
}
```



## OpenGL 本质是一个巨大的 状态机



### 上下文变量 - 告诉 OpenGL 如何绘图

一系列的变量 描述OpenGL 此刻应当如何运行



### OpenGL 的上下文（状态）- Context

**改变上下文变量改变 OpenGL 的状态，告诉 OpenGL 如何绘图**



修改 OpenGL 的状态的方式：设置选项，操作缓冲，最后使用 OpenGL 上下文来渲染



### 状态设置函数 - 改变上下文

### 状态使用函数 - 根据状态执行操作



### OpenGL 需要翻译到其他高级语言，引入了抽象层 - 对象 

OpenGL 内核是一个C语言的库，语言结构需要被翻译到其他的高级语言，引入了一些抽象层

每个对象可以是不同的设置，执行使用 OpenGL 状态的操作的时候只需要绑定含有设置的对象即可



#### OpenGL 的一个对象是指选项的集合

如：可以用一个对象代表绘图窗口的设置

可以把对象看成一个 C 风格 的结构体（Struct）

```
struct object_name{
	float option1;
	int option2;
	char[] name;
};
```



#### OpenGL 的基本类型 - 保证了不同平台的类型大小统一

也使用使用 定宽类型（Fixed-width Type）来显示



#### OpenGL 常见工作流

1、创建对象，ID保存引用

2、将对象绑定到上下文的目标位置

3、设置窗口选项，这些选项会保存在 ObjectID引用的对象中

4、将目标位置的对象ID设置回0，解绑对象

```
//OpenGL 的状态
struct OpenGL_Context{
	...
	object* object_Window_Target;
	...
};

//创建对象
unsigned int objectId = 0;
glGenObject(1, &objectId);
//绑定对象至上下文
glBindObject(GL_WINDOW_TARGET, objectId);
//设置当前绑定到 GL_WINDOW_TARGET 的对象的一些选项
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH, 800);
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600);
//将上下文对象设置回默认
glBindObject(GL_WINDOW_TARGET, 0);
```



# 创建窗口

创建 OpenGL 的上下文（Context） 和 一个用于显示的窗口 **在每个操作系统上都是不一样的**

解决：

​	OpenGL 的库 提供了节省书写操作系统相关的代码，提供了窗口和上下文用于渲染

​	GLUT、SDL、SFML、GLFW



## CMake - 工程文件生成工具

源代码编译：可以保证生成的库完全适合操作系统和 CPU

预编译的二进制文件 也可以实现，但不可能所有的库都提供 预编译的二进制文件，而且预编译的二进制文件也可能不适用于系统

但是如果开放了源代码，就需要 团队协作里的每个人使用相同的 IDE 或者 构建系统来搞开发，因为提供的 项目/解决方案文件 可能与一些人的 IDE 不兼容，必须给定的 .c/.cpp 和 .h/.hpp 文件来自己建立项目/解决方案（CMake 解决了这个枯燥的工作）



CMake 是一个 工程文件生成工具

使用 CMake 后，用户可以使用预定义好的 CMake 脚本，根据自己的 IDE 的不同 生成不同的 IDE 的工程文件



### CMake - GUI  全流程

1、下载源码包 并 解压

2、CMake - GUI 编译对应 IDE 的 项目源代码

3、用对应的 IDE 打开 Build 之后的 项目源代码工程

4、在 IDE 中 生成解决方案，然后在 build/src/Debug 文件夹中找到 编译出的 库文件 xxx.lib

5、让 IDE 知道库和头文件的位置

		1. 找到 IDE 或者编译器的 /lib 和 /include 文件夹，添加 生成的库的 include 文件夹里的文件到 IDE 的 /include 文件夹里 - 做法不推荐
		1. 建立一个新文件夹存放所有第三方库文件和头文件，在 IDE 或编译器中指定这些文件夹 - 推荐做法

6、如果IDE 是 VS，在 项目的 VC++ 目录添加 lib 文件夹，然后在 链接器中 选择 输入 - 附加依赖项 中 添加 xxx.lib 文件

7、还需要在 链接器中 选择 输入 - 附加依赖项 中 添加 opengl32.lib



## GLFW 库 

针对 OpenGL 的 C 语言库

提供一些渲染物体所需的最低限度的接口

允许用户创建 OpenGL 上下文、定义窗口参数 以及 处理用户输入



### CMake 构建 GLFW 的 lib库

注意：除了 GLFW 之外还需要添加一个链接条目到 OpenGL 的库（ 链接器中 选择 输入 - 附加依赖项 ）

Windows 上的 OpenGL 库：

​	opengl32.lib 已经包含在 Microsoft SDK 里， 安装 visual studio 的时候默认安装

Linux 上的 OpenGL 库：

​	链接 libGL.so 库文件



然后就可以正常添加头文件使用

```c
#include <GLFW\glfw3.h>
```





## GLAD 库

OpenGL 只是一个规范，具体实现是驱动开发商针对特定显卡实现

驱动版本太多无法在编译的时候就确定函数的位置，需要在运行时获取函数地址并且将其保存在一个函数指针中供以后使用

需要对每个可能使用的函数重复这个非常复杂且繁琐的过程

```
//定义函数原型
typeof void (*GL_GENBUFFERS)(GLsizei, GLuint*);
//找到正确的函数并且赋值给函数指针
GL_GENBUFFERS glGenBuffers = (GL_GENBUFFERS)wglGetProcAddress("glGenBuffers");
//现在函数可以被正常使用
GLuint buffers;
glGenBuffers(1, &buffer);
```



GLAD 解决了这个繁琐的问题

GLAD 使用了**在线服务**，可以告诉 GLAD 需要定义的 OpenGL 版本，GLAD 会根据这个版本加载所有相关的 OpenGL 函数

https://glad.dav1d.de/



将 Generate 生成的 zip 文件下载后，解压将 glad 和 KHR 两个头文件目录复制到 include 文件夹中，并添加  glad.c 文件到工程中

```
//使用
#include <glad/glad.h>
```









































