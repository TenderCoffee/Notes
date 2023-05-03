

# 基础了解 - IOS 的限制

禁止 JIT 并导致依赖 JIT 的官方CLR的虚拟机无法运行

必须用 AOT 将 Managed程序提前转换为目标平台的静态原生程序后再运行



两种方案：mono 和 Il2cpp

1. Mono 也支持 AOT ，但是新能较差且跨平台支持不佳
2. IL2CPP 转换工具，包含如下：
   1. 一套 AOT运行时
   2. 一套 DLL到 C++代码 和 元数据的转换



# 基础了解 - CLR 运行中做的事情

1. 执行简单的内存操作或者计算或者逻辑跳转。

   这部分与 CLI 的 Base 的指令集大致对应

2. 执行一个依赖于元数据信息的基础操作

   例如 a.x， arr[3] 这种，依赖于元数据信息才能正确工作的代码

   对应部分 CLI 的 Object Mode 指令集

3. 执行一个依赖元数据的比较复杂的操作。

   如 typeof(object)，a is string，(object)5 这种依赖于运行时提供的函数以及相应元数据才正确工作的代码，对应部分 CLI 的 Object Mode 指令集

4. 函数调用。

   包括且不限于被AOT函数调用以及调用 AOT函数，以及 Interpreter 之间的函数的调用。

   对应 CLI 指令集中的 call、callvir、newobj 等 Object Model 指令

   



# 基础了解 - 程序集 的划分

在不需要热更新的项目中，合理的规划和拆分代码模块，设置合理的引用关系，**可以解除基础框架-游戏模块-三方插件的耦合**。

![](images\程序集的划分.webp)

游戏模块作为需要经常改动的模块，它引用了基础框架模块和 三方插件（插件不会对游戏模块引用）



## 划分程序集后

**如果按上图划分程序集并设置引用关系**：

1. 代码的依赖关系会因为程序集依赖被强制限制。
2. 代码的依赖关系会更好理顺，不会过分依赖开发者的个人代码能力。



## 程序集与热更新



1、AOT程序集：不需要热更的程序集（自定义）

​	自定义程序集：需要设置对外部程序集的引用后，才可以在内部使用相关的接口代码



2、JIT程序集：需要进行热更的程序集（自定义）

​	自定义程序集：需要设置对外部程序集的引用后，才可以在内部使用相关的接口代码



3、Assembly-CSharp程序集：特殊的程序集（Unity 默认）

1. unity默认程序集，如果自己添加的代码，如果没有专门划分程序集，那么就会被设置为默认的**Assembly-CSharp**或**Assembly-CSharp-Editor**程序集
1. 遍布整个项目目录不便管理和统计
1. 拥有最大访问权力。可以访问项目内所有的程序集(在勾选Auto Referenced的时候)
1. 不能当做热更程序集，为了便于管理项目



## Unity 中定义程序集（Assembly Definition 文件 ）

程序集的生成和划分是由我们在unity中处理，然后unity自动生成，无法在vs中右键程序集(项目)条目然后打开属性，查看该程序集的相关设定。



Assembly Definition 文件 可以处理的事情：

1. 添加自定义的宏：**Define Constraints**;
2. 添加删除程序集的依赖：**Assembly Definition Refercences**;
3. 选择生效的平台：**Platforms**;



General 选项下 的选项解释：

1. **Allow ‘unsafe’ Code**：是否启用不安全的代码；
2. **Auto Referenced**：是否自动被依赖；勾选后会被默认的Assembly-CSharp程序集自动依赖。所以如果我们想在Assembly-CSharp中隔离对当前程序集的依赖，取消勾选。
3. **No Engine References**:不依赖于引擎提供的代码模块。适用于可以在unity或其他平台的项目中通用的程序集。
4. **Override References**：可以手动指定所依赖的预编译的程序集，因为unity项目中的预编译程序集可以被其他默认依赖，勾选后当前程序集可以选择(不)依赖某个预编译的程序集。
5. **Root Namespace**: 当前程序集的默认命名空间，填写后我们使用unity添加新代码文件，会自动添加命名空间。我测试只有在unity中创建才生效。





# 基础了解 - 解释执行

解释执行不依赖于平台，因为编译器会根据不同的平台进行解析。例如JS语言无论在windows平台还是在Linux平台都可以使用。故可移植性强。使用解释执行的程序我们一般称为解释程序。它将源语言直接作为源程序输入，解释执行解释一句后就提交计算机执行一句，并不形成目标程序。如在终端上打一条命令或语句，解释程序就立即将此语句解释成一条或几条指令并提交硬件立即执行且将执行结果反映到终端，从终端把命令打入后，就能立即得到计算结果。这种工作方式非常适合于人通过终端设备与计算机会话



因为热更新的脚本，**除了 执行代码 是 以解释模式 执行**，其他方式跟 AOT部分的类型是完全相同的，包括占用内存，所以 HybridCLR 是运行时级别的实现，



# 基础了解 - IL2CPP 基础



Mono：有 Hybrid mode execution，可以支持动态加载 DLL



IL2CPP：使得原始 C# 代码最终能在 IOS 平台上运行

是一个纯静态的 AOT 运行时，不支持运行时加载 DLL，无法支持热更新



IL2CPP 脚本编译流程：

![](images\IL2CPP 脚本编译流程.png)



IL2CPP 构建：

使用 IL2CPP 开始构建后，Unity 会自动执行以下步骤：

1. 将 Unity Scripting API 代码编译为 常规的 .NET DLL（托管程序集）
2. 应用托管字节码剥离。这个步骤可显著减小构建的游戏大小
3. 将所有托管程序集转换为标准的 C++ 代码
4. 使用本机平台编译器编译生成的C++代码 和 IL2CPP 的运行时部分
5. 将代码链接到可执行文件 或 DLL，具体取决于目标平台



C++ 不像 C# 有完整的类型信息，所以为了获取完整的类型信息， HybridCLR 需要对 IL2CPP 做改造

il2cpp_plus 项目工程：（HybridCLR 的 底层原理：修改了 IL2CPP 的一部分源码）

​	对 IL2CPP 做了改造，增加了泛型方法，泛型数据类型，其他一些支持，添加到 IL2CPP



勾选 IL2CPP 打包方式后的变化：

![](images\IL2CPP 打包方式 的变化.png)



# HybridCLR 开始

https://focus-creative-games.github.io/hybridclr/about/





# HybridCLR 和 其他热更 的不同之处

![](images\HybridCLR有哪些不同之处.png)



其他热更新：

1. 使用独立虚拟机（第三方VM），在VM中解释执行代码来实现热更新，与 il2cpp 的关系本质上**相当于mono中嵌入lua**。

2. VM 和 IL2CPP 是独立的，元数据系统不想通，所以类型系统不统一，为了让热更新类型能够继承 AOT 类型，**需要写适配器**，而且解释器中的类型不能为主工程点的类型系统所识别。

3. 特性不不完整、开发麻烦、运行效率底下。

   

HybridCLR 热更新：对 IL2CPP 运行时进行扩充，添加 iterpreter  模块

1. 原生的C#热更新方案
2. IL2CPP 相当于mono的 AOT 模块，HybridCLR 相当于 Mono 的 iterpreter 模块，两者合一成为 完整的Mono
3. 通过 **System.Reflection.Assembly.Load 动态加载dll**，从而支持ios平台的热更新
4. 由原生Runtime实现，可以与 主工程AOT部分类型完全等价并且无缝统一，可以任意调用、继承、反射、多线程，**不需要生成代码或者写适配器**



# HybridCLR 工作原理

HybridCLR 可以对 AOT DLL 任意增删改，会智能地让变化或者新增的类和函数以 Interpreter 模式运行，但是未改动的类和函数以 AOT 方式运行，让热更新的游戏逻辑的运行性能基本达到原生的 AOT 的水平。



扩充了 unity的 il2cpp runtime，额外提供了interpreter模块

将 纯AOT 改造为 AOT + Interpreter（直译器）的混合运行方式

**从底层彻底支持了热更新dll**



![](images\HybridCLR 工作原理.png)



- 实现了一个高效的元数据(dll)解析库
- 改造了元数据管理模块，实现了元数据的动态注册
- 实现了一个IL指令集到自定义的寄存器指令集的compiler
- 实现了一个高效的寄存器解释器
- 额外提供大量的instinct函数，提升解释器性能



# HybridCLR 实现

HybridCLR 的实现：

1. MetadataCache::LoadAssemblyFromBytes（C# 层调用 **Assembly.Load** 时触发）时加载并注册 Interpreter Assembly
2. IL2CPP 运行过程中延迟初始化类型相关的元数据，其中的关键为正确设置了 **MethodInfo 元数据中的 methodPointer 指针**
3. IL2CPP 运行时通过 methodPointer 或者 methodInvoke 指针，再经过桥接函数跳转，最终执行了 Interpreter::Execute 函数
4. **Execute 函数** 在第一次执行某 Interpreter 函数时触发 HiTransform::Transform 操作，将原始 IL 指令翻译为 HybridCLR 的寄存器指令
5. 然后执行该函数对应的 **HybridCLR 寄存器指令**



# HybridCLR AOT 泛型的问题



CLR 泛型：

1. 泛型类型
2. 泛型函数

即使没有明显包含泛型的用法，可能隐含了泛型相关的定义或者操作

例如：

1. int[] 隐含就能实现 IEnemrable<int> 之类的接口
2. 为 Async 生成状态机代码时，也会隐含生成一些对 System.Runtime.CompilerServices.AsyncTaskMethodBuilder::AwaitUnsafeOnCompleted 之类的泛型代码



AOT 泛型：

泛型类型：本身是元数据，内存可以动态创造出任意泛型类型的实例化

泛型函数（包括泛型类的普通成员）：

​	虽然解释器泛型函数没有任何限制，但是由于 AOT泛型函数 的原始函数体元数据 在 IL2CPP 翻译后已经丢失，理论上不可能根据已有的 C++ 泛型函数指针为一个新的泛型类型产生对应的泛型实例化函数（如：无法获得泛型函数 List<MyVector>.ctor 这样的 CLI 中构造函数名称 的实现），导致无法创建对象

本质上：因为 AOT 翻译导致原始 IL 指令元数据的丢失，进而无法创建出 AOT泛型函数的实例，如果原先在 AOT中已经生成对应泛型函数的代码，例如假设在 AOT 中用过 List<int>.Count ， 那么在热更部分就可以使用。



解决：

1. 基于 IL2CPP 的 **泛型共享机制**
2. 基于**补充元数据**的泛型函数实例化技术（HybridCLR的专利技术）



## IL2CPP 的 泛型代码共享

只生成一份代码，目的是避免泛型代码膨胀和节约内存



CLR 中的 泛型代码共享：

1. CLR 中认为所有引用类型实参都一样可以代码共享（List<string> 方法编译的代码可以直接用在 List<Stream> 方法，因为所有引用类型实参/变量 只是指向托管堆的一个8字节指针）
2. CLR 中认为值类型则必须每种类型都进行代码生成，因为值类型大小不一定



IL2CPP 中 泛型代码共享：

以 List<T> 举例：

1. 可以使用 AOT 中使用过的任何 List 的实例化类型，例如在 AOT里用过 List<vector3>， 则热更新里也可以用
2. 可以使用任意 List<EnumType>，只需要在AOT里实例化某一个 List<相同 隐含类型 的 Enum Type>
3. 可以使用任意引用类型的泛型参数 List<ClassType>，只需要在 AOT 里实例化 List<object>（或任意一个引用泛型参数 如 List<string>）

注意：

​	因为 **值类型无法泛型共享**，而 热更新值类型 不可能提前在AOT里 泛型实例化

​	所以 IL2CPP 泛型共享机制 不支持 List<热更新值类型>

解决：

​	提前 AOT 实例化



IL2CPP 中 **值类型不支持泛型共享** 的原因：

1、值类型 大小不定，且值类型对齐方式不一样，不同对齐会导致所在的类的布局不同

```c#
struct A //size = 4
{
    short x;
    short y;
};
struct B	//size = 4
{
	int x;   
};
struct GenericDemo<T>
{
    short x;
    T v;
    public T GetValue() => v;
}
//GenericDemo<A> size=6 alignment=2 字段v 在类中的偏移为2
//GenericDemo<B> size=8 alignment=4 字段v 在类中的偏移为4
//v的偏移不同不太可能生成一套相同的C++代码

```

2、ABI 兼容问题

​	相同大小的结构体，在 x64 ABI 是 等效的，可以用同等大小的结构体来作 共享泛型实例化

​	但是在 ARM64 ABI 却是不行的

```c#
//比如 下面这2个结构体大小一样 但是无法泛型共享

//大小12 作为函数参数传递的时候 以引用的方式传参
struct IntVec3{ int x, y, z; }
//大小12 作为函数参数传递的时候 三个字段分别放到三个浮点寄存器里
struct FloatVec3{ float x, y, z; }
```



## 共享类型计算规则

假设 泛型类T 的共享类型为 泛型：Reduce类型，则：

1. 枚举类型的值类型

   Reduce类型 为自身，如 int 的 Reduce类型 为 int

2. 枚举类型

   Reduce类型 为 隐含类型 与 它相同的枚举

   ```c#
   //enum 的 隐含类型 是 int
   //MyEnum 的 Reduce类型为 Int32Enum
   enum MyEnum
   {
       A = 1,
   }
   //MyEnum2 的 Reduce类型为 SByteEnum
   enum MyEnum2 : sbyte
   {
       A = 10,
   }
   ```

   CLI 中 没有 Int32Enum、SByteEnum 这些类型，需要在 AOT中提前创建这些枚举类型

3. Class引用类型。Reduce 类型 为 object

4. 泛型类型。

   GenericType<T1, T2, ... > 

   	1. 如果是 Class 类型，则 Reduce Type 为 object
   	1. 否则 Reduce 类型 为 GenericType<Reduce 类型<T1>,Reduce 类型<T2>... >

​	例如：

​		Dictionary<int, string> 的 Reduce 类型 为 object

​		YourValueType<int, string> 的 Reduce 类型 为 YourValueType<int, object>



## 泛型函数的共享泛型函数 计算规则

```c#
Class<C1,C2...>.Method<M1, M2, ...>(A1, A2, ...)
//的 AOT 泛型函数为
Class<reduce(C1), reduce(C2)...>.Method<reduce(M1), reduce(M2), ...>(reduce(A1), reduce(A2), ...)
```

1. List<string>.ctor 对应的共享函数为 List<object>.ctor

2. List<int>.Add(int) 对应的共享函数为 List<int>.Add(int)

3. YourGenericClass<string, int, List<int>>.Show<string, List<int>, int>(ValueTurple<int, string>,string, int) 

   的共享函数为：

   YourGenericClass<object, int, object>.Show<object, object, int>(ValueTurple<int, object>, object, int) 

   

## C#、Async 与 IEnumerate 之类的语法糖机制引发的 AOT 泛型问题

编译器为复杂语法糖生成隐含的 AOT泛型引用

解决引发的AOT反省实例化问题

如：Async 生成了 若干类 以及 状态机 以及一些代码，这些隐藏生成的代码中包含了对多个 AOT 泛型实例化问题



以下AOT泛型实例化缺失错误，使用标准的泛型 AOT的实例化规则去解决问题即可

```c#
void AsyncTaskMethodBuilder<T>::Start<TStateMachine>(ref TStateMachine stateMachine)
void AsyncTaskMethodBuilder<T>::AwaitUnsafeOnCompleted<TAwaiter, TStateMachine>(ref TAwaiter awaiter, ref TStateMachine stateMachine)
void AsyncTaskMethodBuilder<T>::SetException(Exception exception)
void AsyncTaskMethodBuilder<T>::SetResult(T result)
```



由于 Unity 使用 C# 编译器 对 Release 模式下生成的状态机 是 ValueType类型，导致无法泛型共享，但是 Debug 模式下生成的状态机是 Class类型，可以泛型共享。

因此，如果“未使用基于补充元数据的泛型函数实例化技术”，则为了能够让热更新中使用 Async语法，使用脚本编译 DLL 时，需要加上代码

```c#
scriptCompilationSettings.options = ScriptCompilationOptions.DevelopmentBuild;
```

这样编译出的状态机是 Class类型，这样才能在热更新代码中正常工作



## AOT 泛型提前实例化

Unity 默认的代码裁剪规则：

​	1、类型 public 函数默认会被保留，AOT中 new List<int>() 即可

​	2、每个泛型函数（包括泛型类的成员函数）都需要在 AOT 中提前引用（不必是真正的运行，只需要在代码中假装调用过），如果在代码中未被使用过，则不会生成泛型共享函数

​	如：要让 List<YourHotUpdateClass>  的各个函数都能够正确调用，就需要确保 List<object>（ 或者List<string> ）必须在 AOT 中已经提前调用过



HybridCLR 解决：

​	RefType.cs 包含了对常见泛型类型的实例化



## 基于补充元数据的泛型函数实例化技术（HybridCLR的专利技术）


AOT 泛型函数无法实例化的问题，本质上是 IL2CPP 翻译造成的元数据缺失导致

解决：补充原始元数据

​			使用 HuatuoApi.**LoadMetadataForAOTAssembly** 函数为 AOT 的 Assembly 补充对应的 元数据

注意：为了补充的DLL和打包时候裁剪的 DLL 精确一致

​			必须使用 Build过程中生成的裁剪后的 DLL，而不能直接复制原始 DLL

​			这些DLL在目录 {project}/Temp/StagingArea/Il2Cpp/Managed 下找到

​			只要在使用 AOT泛型前调用即可（只需要调用一次），如果未注册相应的泛型元数据，就会退回到 IL2CPP 的泛型共享机制



缺点：

​		实例化的函数**以解释方式执行**，性能不够高

​		最佳做法：提前在 AOT 中泛型实例化，提升性能



**常用的性能敏感的泛型类和函数，提前在 AOT中实例化**



# HybridCLR 指令集设计



原始 IL 指令集：基于栈的指令集

优点：指令个数少、优雅、紧凑、非常适合表示虚拟机逻辑

缺点：不适合被解释器高效解释运行

解决：需要转换为自定义的能高效解释执行的指令集，然后在解释器中运行



## IL 指令集的缺陷

- IL是基于栈的指令，运行时维护执行栈是个无谓的开销
- IL有大量单指令多功能的指令，如add指令可以用于计算int、long、float、double类型的和，导致运行时需要根据上文判断到底该执行哪种计算。不仅增加了运行时判定的开销，还增加了运行时维护执行栈数据类型的开销
- IL指令包含一些需要运行时resolve的数据，如newobj指令第一个参数是method token。token resolve是一个开销很大的操作，每次执行都进行resolve会极大拖慢执行性能
- IL是基于栈的指令，压栈退栈相关指令数较多。像a=b+c这样的指令需要4条指令完成，而如果采用基于寄存器的指令，完全可以一条指令完成。
- IL不适合做其他优化操作，如 HybridCLR 的InitOnce JIT技术。
- 其他



## HybridCLR 指令集设计目标

HybridCLR指令集设计目标就是解决原始IL的缺陷，以及做一些更高级的优化，目前主要包含以下功能

- 指令中包含要操作的目标地址，不再需要维护栈
- 对于单指令多功能指针需要为每种操作数据类型维护一个对应指令，免去运行时维护类型及判定类型的开销。如如add指令，要特例化为add_int、add_long之类的指令
- 对于涉及到token解析的指令，尽量转换指令时直接固定，省去计算或者查询的开销。如 newobj中 method token字段，直接转变成 MethodInfo* 元数据
- 对于一些常见操作如 a = b + c，需要 `ldloc b , ldloc c, add, stloc a` 这4条指令，我们希望提供专用指令折叠它
- 需要能与一些常见的优化技术配合，如函数inline、InitOnce动态jit技术
- 需要考虑到跨平台问题，如在armv7这种要求内存对齐的硬件上，也能高效执行

HybridCLR使用经典的寄存器指令集配合一些其他运行时设施实现以上目标。



https://zhuanlan.zhihu.com/p/534541768



## 运行环境

无论指令的内容如何，指令的执行结果必须产生副作用（哪怕是nop这种什么也不干的指令，也导致当前指令寄存器ip发生变化），而这些副作用，必然作用于运行环境（甚至有可能作用于指令本身，如InitOnce技术）。正因为指令执行离不开具体的执行环境，指令设计必然跟HybridCLR解释器及il2cpp运行时的运行环境紧密相关。



### HybridCLR函数帧栈

### 运行时相关



## 指令结构



## 跨平台兼容性

### Padding优化

### 指令实现

#### 空指令

#### 简单数据复制指令

#### 需要 Expand 目标数据的指令

#### 静态特例化的指令

#### 直接包含常量的指令

#### 隐含常量的指令

##### 指令共享

#### 包含 Resolved 后数据的指令

##### 直接包含 Resolved 后数据的指令

##### 间接包含 Resolved 后数据的指令

#### 分支跳转指令

#### 对象成员访问指令

#### ThreadStatic 成员访问指令

#### 数据访问相关指令

#### 函数调用指令

#### 异常机制相关指令

#### 一些额外的Instinct 指令

#### InitOnce 指令

#### 其他技术相关指令



# HybridCLR 的 接入



## 了解 - HybridCLR Settings

Hot Update Assembly Definitions :

​	配置 热更新程序集

Hot Update Assemblies：

​	不需要配置，它默认返回 Hot Update Assembly Definitions 中配置的程序集名字列表；

Output Link File：

​	HybridCLR工具自动生成代码裁剪配置文件link.xml;

Output AOT Generic Reference File：

​	HybridCLR自动生成AOT泛型元数据补充；

Patch AOT Assemblies：

​	AOT 元数据补充，用于配置AOT dll文件补充元数据



## 了解 - 项目中 程序集的划分

### 内置程序集（Builtin）

逻辑比较靠前，无法热更，必须打进包体的代码

**非热更部分(AOT)** 

如：第三方插件（LitJson）

主要包含：

 	1. 下载更新资源(包括热更dll资源)
 	2. 调用System.Reflection.Assembly.Load(hotfixDll.bytes)载入热更dll的二进制数据
 	3. 通过反射调用Hotfix程序集中的函数进入热更新逻辑



**AOT 元数据补充：（只需要出包时打进包里）**

**注意：需要考虑 AOT的部分（第三方插件）是否值类型泛型被调用**

如果被调用 则需要在游戏启动后先进行**AOT元数据补充**（ **RuntimeApi.LoadMetadataForAOTAssembly** ）



### 热更新程序集（Hotfix）

HybridCLR会把热更新程序集打成一个 热更dll文件，在游戏运行时注入这个热更dll，然后就能执行热更dll中的逻辑了



新建文件`HotFix.asmdef`并检视界面修改属性如下



![](images\HybridCLR HotFix.asmdef设置.png)









后续需要一个**HotfixEntry（热更逻辑入口）**作为进入热更逻辑的入口，使用反射的方式进入Hotfix程序集



## 了解 - AOT 元数据补充

由于编译成热更dll后会 **丢失值类型泛型** 的数据类型，所以需要把 Hotfix 中用到的值类型泛型数据提前补充到AOT里。

在 **元数据共享机制** 下，Hotfix程序就能正确识别那些值类型泛型数据



AOT元数据补充 - 保存：（**HybridCLR Settings - Patch AOT Assemblies**）

 	1. 勾选AOT Dll点击Save会自动添加到Patch AOT Assemblies列表，
 	2. 并将对应dll文件移动到Resources/AotDlls下，

AOT元数据 在游戏运行时的加载

 	1. 游戏运行时 通过Resources.LoadAll<TextAsset>("AotDlls")加载全部AOT DLL
 	2. 再手动调用 **RuntimeApi.LoadMetadataForAOTAssembly** 进行AOT元数据补充;



为AOT补充元数据，元数据补充的dll目标 一定是AOT dll而不是热更dll

例如，项目用到了LitJson插件，插件放到了非热更部分(AOT)，LitJson有很多方法用到了值类型泛型T，当热更dll中的逻辑调用了AOT里带有值类型泛型的函数就会报错，因此需要将LitJson.dll添加到补充 Patch AOT Assemblies 列表里，游戏启动后先进行AOT元数据补充，热更dll再调用值类型泛型函数就不会报错了

**注意：AOT元数据补充dll只需要出包时打进包里就行，不要作为热更资源**



## 了解 - 热更逻辑入口 HotfixEntry



因为 Hotfix程序集 以**热更资源的形式存在，不会被编译成.so进入安装包**

所以 **Builtin程序集不能直接调用Hotfix程序集**，否则编译时会报错

这就需要一个**HotfixEntry（热更逻辑入口）**作为进入热更逻辑的入口，**使用反射的方式进入Hotfix程序集**。



**（示例里 整合了GameFramework框架）在Builtin程序集初始化完热更dll后通过反射调用 HotfixEntry.StartHotfixLogic() 进入热更新逻辑**

```c#
        //加载热更新Dll完成,进入热更逻辑
        if (loadedProgress >= totalProgress)
        {
            Log.Info("热更dll加载完成, 开始进入HotfixEntry");
            loadedProgress = -1;
#if !DISABLE_HYBRIDCLR
            var hotfixDll = GFBuiltin.Hotfix.GetHotfixClass("HotfixEntry");
            if (hotfixDll == null)
            {
                Log.Error("获取热更入口类HotfixEntry失败!");
                return;
            }
            hotfixDll.GetMethod("StartHotfixLogic").Invoke(null, new object[] { true });
#else
            HotfixEntry.StartHotfixLogic(false);
#endif
```



HotfixEntry.cs

```c#
using GameFramework;
using GameFramework.Fsm;
using GameFramework.Procedure;
using UnityGameFramework.Runtime;
/// <summary>
/// 热更逻辑入口
/// </summary>
public class HotfixEntry
{
    public static void StartHotfixLogic(bool enableHotfix)
    {
        Log.Info("Hotfix Enable:{0}", enableHotfix);
        GFBuiltin.Fsm.DestroyFsm<IProcedureManager>();
        var fsmManager = GameFrameworkEntry.GetModule<IFsmManager>();
        var procManager = GameFrameworkEntry.GetModule<IProcedureManager>();
        //手动把热更新程序集的流程添加进来
        ProcedureBase[] procedures = new ProcedureBase[]
        {
            new PreloadProcedure(),
            new ChangeSceneProcedure(),
            new MenuProcedure(),
            new GameProcedure(),
            new GameOverProcedure()
        };
        procManager.Initialize(fsmManager, procedures);
        procManager.StartProcedure<PreloadProcedure>();//默认启动热更新程序集的预加载流程
    }
}
```







## 开始接入

 1. Unity 上安装 WIndows Build Support (II2CPP)模块

 2. Visual Studio 添加 C++模块

 3. 安装 Git

 4. ProjectSettings 中 勾选  Allow 'unsafe' Code ：允许不安全类型的代码

 5. ProjectSettings 中 Scripting Backend：项目工程 切换到 IL2CPP

 6. ProjectSettings 中 关闭 增量式GC(Use Incremental GC) 选项

 7. ProjectSettings 的 API Compatibility Level 选择 .NET Framework

 8. 使用 Unity Package Manager 从Git上安装 HybridCLR**（hybridclr_unity）**
     1. https://github.com/focus-creative-games/hybridclr_unity.git 

 9. 菜单栏 HybridCLR - Installer... 安装 HybridCLR

     1. 设置安装的仓库版本号：选择安装的版本 main是开发分支，1.0 是稳定的分支
     2. 直接点击安装 会直接安装最新的分支

 10. 下载 HybridCLR 工程**（hybridclr_trial ）**
      1. 下载 hybridclr_trial 项目教程，解压

      2. 提取出 Package 中的 Unity-Logs-Viewer 插件 复制到自己项目的 Package

      3. 提取出

          Assets/Editor、 Assets/HotUpdate、 Assets/Main、 Assets/Prefab、 Assets/Scene、Assets/link.xml 复制到自己的 Assets 项目

 11. 对原来项目进行划分
      1. 将自己的项目 把代码分离开，分成两个程序集，一个是**内置程序集(Builtin）**，一个是**热更程序集(Hotfix)**

      2. 在 Unity 中右键创建 Assembly Definition文件（注意，Hotfix程序集可以引用Builtin程序集，但是Builtin不能直接引用Hotfix程序集）

         修改 Hotfix程序集 中的 Use GUIDs：关联 **HotUpdateMain.cs**（热更新程序入口处代码）

      3. 菜单  HybridCLR->Settings 添加 热更新程序集（HybridCLR->Settings->Hot Update Assembly Definitions），关联 Hotfix程序集

         关联之后卸载所有 HotUpdate 文件夹下的代码都是可以热更新的

 12. 修改 LoadDll.cs 中 

       1. 将 DownLoadAssets 下载的 Assembly-CSharp.dll 修改为 Hotfix.dll.bytes
       2. 修改 StartGame 中的 Assembly-CSharp.dll 为 Hotfix.dll.bytes

 13. HybridCLR 的 Package 的版本 >1.1.0

       1. 运行菜单栏 HybridCLR - Generate - LinkXml
       2. 执行一次Build 打一个 PC 包
       3. 再运行菜单栏 HybridCLR - Generate - All：生成桥接函数

      

 14. 开始修改代码，下面步骤开始热更新

 15. 运行菜单栏 HybridCLR - Build - BuildAssetsAndCopyToSteamingAssets

       1. 把预制体打包到 SteamingAssets 下
       2. 拷贝剪辑后的 DLL到 SteamingAssets 下

 16. 手动 将 SteamingAssets 下的 Hotfix.dll.bytes、prefabs、build_info 三个文件拷贝到 PC 打包目录下的SteamingAssets 文件夹下

       1. pc包路径/xxxData/SteamingAssets 文件夹下 

      

 17. 如果修改到的是 原生层（非热更层）的 代码内容：第三方插件使用了泛型

       1. 用到这些插件的程序集，需要将这个第三方插件加入 Use GUIDs 中
       2. 需要加入 AOT 泛型补充（LoadDll.cs 中的 AOTMetaAssemblyNames），将对应插件的 DLL 添加进去
       3. 需要重新 Build 出一个 PC包
       4. 再运行菜单栏 HybridCLR - Generate - All：生成桥接函数
       5. 运行菜单栏 HybridCLR - Build - BuildAssetsAndCopyToSteamingAssets


​	





## 如何使用

1. 把 热更程序集 打成dll，作为资源以供下载更新.
2. 启动游戏后下载更新热更dll，加载热更dll，调用HybridCLR函数为AOT补充泛型元数据。
3. 通过反射进入热更逻辑，Over



https://www.bilibili.com/video/BV1z24y117xD/?spm_id_from=pageDriver&vd_source=c3b6c7523678270bec7c37b5a5b36794



# HybridCLR Demo



## 项目设置



1. 找到项目目录的下的 HybridCLRData 文件夹中的 **init_local_il2cpp_data.bat** 设置 **IL2CPP_BRANCH** 的 Unity 版本 和 **IL2CPP_PATH** 对应的 Unity 的 IL2CPP 的安装路径（IL2CPP 的原始源码）

   **因为 IL2CPP 的源码 是开源的，设置了 IL2CPP_PATH  后 HybridCLR 才能去修改源码**

   1. 对应关系：

      Unity 2020.x => 使用HybridCLR il2cpp 2020.3.33分支；IL2CPP_PATH配置为Unity2020.3.33安装目录下的il2cpp

      Unity 2021.x => 使用HybridCLR il2cpp 2021.3.1分支；L2CPP_PATH配置为Unity2021.3.1安装目录下的il2cpp

   2. 建议：

      HybridCLR魔改了Unity 2020.3.33和Unity 2021.3.1两个版本的il2cpp，分别兼容Unity 2020.x和Unity 2021.x. 

      建议il2cpp和HybridCLR魔改版il2cpp的Unity版本保持一致，即如果你是用Unity2020.x开发，就把IL2CPP_PATH设置为Unity2020.3.33安装目录下的il2cpp，使用Unity2021.x开发就把IL2CPP_PATH设置为Unity2021.3.1安装目录下的il2cpp. il2cpp版本不一致否则很有可能遇到奇怪的bug.

      为了方便可以把Unity2020.3.33和Unity2021.3.1的il2cpp文件夹提取出来放到项目里。

4. Unity 安装目录下的 Data/MonoBleedingEdge/bin 文件夹 存放了一些命令行工具，需要载入 Data/MonoBleedingEdge/bin/mono.exe  才可以被执行

3. 管理员权限运行 init_local_il2cpp_data.bat ： 

   批处理文件 做了什么

   1. 从 Git 上下载 hybridclr_repo 源码仓库
   2. 从 Git 上下载 il2cpp_plus_resp 源码仓库
   2. 将IL2CPP_PATH配置的Unity原生il2cpp库与魔改后的il2cpp文件合并
   3. 定义 Unity 的环境变量，在 Unity 启动后在 Unity 中设置，让 Unity 在打包的时候不是走 Unity 原来的 IL2CPP 的目录打包，而是走新下载下来的源码工程进行 IL2CPP 打包，执行 C# 到 C++ 的转换，转换后形成机器指令，放到 Unity 中执行

6. 重新启动进入 Unity 

7. 开始编程 设置 热更程序集

8. 修改代码加载的热更程序集



## 打包运行程序



1. 菜单 HybridCLR - Build - Win64 打包出一个 Win64.exe 的运行程序

2. 在 Unity工程中，修改热更的代码内容

   热更文件夹 Asset/HotFix 和 Asset/HotFix2 存放的是需要热更新的文件

3. 菜单 HybridCLR - BuildBundles - ActiveBuildTarget 打包 ab

4. 将打包出来生成的 ab 文件从 StreamingAssets 文件夹  复制到 Win64 的 StreamingAssets目录

5. 热更新文件替换完成，重新运行游戏就可以看到新修改的内容





# HybridCLR 打包流程



1. Build 构建操作（HybridCLR - Build - Win64）

   **注意：HybridCLR 必须要先Build工程，目的是Build时生成代码裁剪后的dll以供HybridCLR进行AOT元数据补充。**

   **首次打包获取aot dll（此次为无效打包，仅为获取aot dll）**

2. 编译生成需要热更的DLL （HybridCLR->CompileDll->ActiveBuildTarget）

   或者直接调用代码： **CompileDllCommand.CompileDll(EditorUserBuildSettings.activeBuildTarget);**

3. 将DLL 放到 AB目录 打包 

   按照自己的ab打包策略放入自己规划的目录内

4. 生成LinkXml，桥接函数 MothedBridge等等（HybridCLR->Generate->All） 

   或者直接调用代码： **HybridCLR.Editor.Commands.PrebuildCommand.GenerateAll();**

5. 将 AOT补充元数据dll 拷贝到目标平台的目录 

   按照自己的ab打包策略放入自己规划的目录内；

6. 将 DLL 打包成AB - 所有 ab资源放远程服务器待下载







# HybridCLR 游戏中热更流程

![](images\HybridCLR 热更流程.png)

下载热更资源（第2步）：

 **热更的DLL 被打包成 AB包** 进行下载下来



加载热更DLL（第3步）：

只需要加载资源，然后调用 **Assembly.Load()** 即可。

```c#
// 加载资源 （示范代码）
var dllBytes = await AssetComponent.LoadAsync<TextAsset>("Assets/Dlls/game.bytes");
// 加载程序集
Assembly gameAss = System.Reflection.Assembly.Load(dllBytes.bytes);
```



加载补充元数据DLL（第4步）：

如果热更程序集引用到了aot程序集的情况下，则一定需要 **加载补充元数据DLL**，通过dll自动补充元数据

否则会报错：**MissingMethodException**: AOT generic method isn't instantiated in aot module xxx

**使用 LoadMetadataForAOTAssembly**

```c#
// 前三个是系统自带，推荐带上，第四个是自己游戏AOT的程序集
var AotAssemblyDlls = new string[] {
    "mscorlib","System","System.Core",
    
    "MyGame.Framwork",
};

for (int i = 0; i < AotAssemblyDlls.Length; i++)
{
    var aotDllName = AotAssemblyDlls[i];
    var path = $"Assets/ResBundles_Dll/aotDll/{aotDllName}.bytes";
    var asset = AssetComponent.Load<TextAsset>($"Assets/AotDlls/{aotDllName}.bytes");
    
    // 加载assembly对应的dll，会自动为它hook。一旦aot泛型函数的native函数不存在，用解释器版本代码
    LoadImageErrorCode err = RuntimeApi.LoadMetadataForAOTAssembly(asset.bytes);
    if (err == LoadImageErrorCode.OK)
        Debug.Log($"{stateID}: LoadMetadataForAOTAssembly:{aotDllName}. ret:{err}");
    else
        Debug.LogError($"{stateID}: LoadMetadataForAOTAssembly:{aotDllName}. ret:{err}");
}
```

Hook：钩子。应用程序，包括应用触发事件和后台逻辑处理，也是根据事件流程一步步地向下执行。

而「钩子」的意思，就是在事件传送到终点前截获并监控事件的传输，像个钩子钩上事件一样，并且能够在钩上事件时，处理一些自己特定的事件。



元数据 DLL：

在构建过程中，会生成 **Strip(裁剪)过的dll**，HybirdCLR会自动帮我们把dll拷贝到目录：

项目目录/HybridCLRData/AssembliesPostIl2CppStrip/目标平台/xxx

```c#
//获取当前目标平台的目录
SettingsUtil.GetAssembliesPostIl2CppStripDir(EditorUserBuildSettings.activeBuildTarget);
```



所以如果通过ab加载，在打包assetbundle资源前，我们必须要进行一次构建操作。

然后把构建生成的需要补充的dll拷贝到项目目录中，然后 将dll打包成ab资源。



**加载热更场景，执行热更代码（第5步）：**

AOT代码无法直接调用热更代码

**推荐方案**：在所有资源都已下载，所有热更代码都已加载后，通过加载热更资源中的场景，场景上挂有热更脚本(例如InitAfterHotFix.cs)，在InitAfterHotFix的Awake方法中进行后续游戏的初始化。

**其他方案**：当然也可以通过Assembly类获取到热更程序集的启动类的启动方法，使用反射的方式实现。















