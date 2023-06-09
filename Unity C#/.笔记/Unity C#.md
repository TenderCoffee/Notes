# 认识C#

https://blog.csdn.net/sinat_34014668/article/details/127039207



## TODO 字符串

### 查看变量的内存地址

https://blog.csdn.net/xiaouncle/article/details/87832198



## TODO 堆和栈

![](images\堆和栈.png)



### CLR 的 new 操作符完成的 四步操作

1. 它计算类型以及所有基类型（一直到System.Object,虽然它没有定义自己的实例字段）中定义的所有实例字段**需要的字节数**。

   堆上的每个对象都需要一些额外的成员---即**“类型对象指针”和“同步块索引”**。

   这些成员由CLR用于管理对象。

   这些额外成员的字节数会计入对象大小。

2. 它**从托管堆中分配指定类型要求的字节数**，从而分配对象的内存，分配的所有字节都设为0.

3. 它**初始化对象的“类型对象指针”和“同步块索引”成员**。

4. 调用类型的实例构造器，向其传入在对new的调用中指定的任何实参。

   **大多数编译器都在构造器重自动生成代码来调用一个基类构造。**

   每个类型的构造在调用时，都要负责初始化由这个类型定义的实例字段。

   最终调用的说System.Object的构造器，该构造器只是简单地返回，不会做其他任何事情。

   （为了证明这一点，可使用ILDasm.exe加载MSCorLib.dll，检查System.Object的构造器。）



开发人员是没有办法显式释放为对象分配的内存，但是CLR采用了垃圾回收机制，能够自动检测到一个对象是否可达，并且自动释放资源。



## 引用类型和值类型

CLR支持两种类型，引用类型和值类型。

1. 引用类型总是从托管堆分配，每次我们通过使用new操作符返回对象内存地址——即指向对象数据的内存地址，而后把这个内存地址pop进线程栈中。引用类型总是处于已装箱状态。
2. 轻量级类型——值类型。避免了每次实例化对象都要进行一次内存分配，值类型的实例一般在线程栈上直接分配，值类型变量中直接就包含了实例本身的字段。



**托管堆**：进程初始化时，CLR会自动划出一个地址空间区域。每个托管进程都有一个托管堆，进程中的所有线程都在同一堆上分配对象记忆。

**NextObjPtr 指针 **：由CLR进行维护，该指针指向下一个对象在堆中的分配位置。对于托管堆而言，**分配一个对象只是修改NextObjPtr指针的指向**，这个速度是非常快的。

**事实上，在托管堆上分配一个对象和在线程栈上分配内存的速度很接近。**



### 装箱和拆箱

装箱是隐式的；拆箱是显式的

装箱：值类型 转 引用类型（装箱是将值类型转换为  object 类型 或由此值类型实现的任何接口类型的过程）

​	值类型进行装箱时，必须分配并构造一个新对象

拆箱：引用类型 转 值类型

​	拆箱所需的强制转换也需要进行大量的计算，只是程度较轻



#### 装箱转换的步骤（3步）

```c#
int a = 100;
Objec t obj1 = (Object)a;
```

1. 在堆上为新的对象实例分配内存，准备用于存放值类型数据，该对象实例包含数据，但没有名称；
2. 在栈上值类型变量的值复制到堆上的对象中；
3. 将堆上创建的对象的地址返回给引用类型变量；

#### 拆箱步骤（2步）

```c#
int b = (int)obja2;
```

1. 获取已装箱的对象的地址；
2. 将值从堆上的对象中复制到堆栈上的值变量中；



#### 性能消耗

装箱、拆箱时生成的是全新的对象，不断地分配和销毁内存不但会大量消耗CPU，同时也会增加内存碎片，降低性能。

**装箱 需要消耗的 托管堆内存，如果有大量的对象产生，会增加 GC 的压力**



#### 装箱出现的场景

1. 当程序、逻辑或接口为了更加通用把参数定义为object，一个值类型（如Int32）传入时，就需要装箱。

2. 一个非泛型的容器为了保证通用，而将元素类型定义为object，当值类型数据加入容器时，就需要装箱。

3. 当结构体实现接口，而接口又持有该结构体时会发生装箱。

4. 值类型调用基类Object中的方法可能会装箱。

   Object 的 GetType方法 返回 System.Type 是非虚方法，值类型实例调用GetType方法一定会装箱。int重写了ToString方法，所以int调用ToString不会装箱。结构体直接调用ToString，GetHashCode会装箱，如果重写了方法可以避免装箱。Dictionary、HashSet, 如果Key是结构体，对其进行操作会触发Equals方法和GetHashCode方法，是会发生装箱的，解决方法是实现lEqualityComparer。

   



## 四种引用（Java举例）

强引用 > 软引用 > 弱引用 > 虚引用

![](images\java4种引用.png)



### 强引用 - 内存炸了都不回收

把一个对象赋给一个引用变量，引用变量就是强引用

```java
//  强引用
MikeChen mikechen=new MikeChen();
```

在一个方法的内部有一个强引用，这个引用保存在Java栈中，而真正的引用内容(MikeChen)保存在Java堆中。

一个对象具有强引用，垃圾回收器不会回收该对象，当内存空间不足时，JVM 宁愿抛出 OutOfMemoryError异常

不需要的时候 弱化让 GC 及时回收

```java
//帮助垃圾收集器回收此对象
mikechen=null;
```

```java
package com.mikechen.java.refenence;
 
/**
* 强引用举例
*
* @author mikechen
*/
public class StrongRefenenceDemo {
 
    public static void main(String[] args) {
        Object o1 = new Object();        
        Object o2 = o1;
        o1 = null;
        System.gc();

        //StrongRefenenceDemo 中尽管 o1已经被回收，但是 o2 强引用 o1，一直存在，所以不会被GC回收。
        System.out.println(o1);  //null
        System.out.println(o2);  //java.lang.Object@2503dbd3
    }
}
```



### 软引用 - 只要内存不够

```java
String str=new String("abc");                                     // 强引用
SoftReference<String> softRef=new SoftReference<String>(str);     // 软引用
```

如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它，

如果内存空间不足了，就会回收这些对象的内存。

用在对内存敏感的程序中，比如高速缓存就有用到软引用，内存够用的时候就保留，不够用就回收。

```java
/**
* 弱引用举例
*
* @author mikechen
*/
Object obj = new Object();
SoftReference softRef = new SoftReference<Object>(obj);
//删除强引用
obj = null;

//调用gc
System.gc();

// 对象依然存在
System.out.println("gc之后的值：" + softRef.get());


//软引用可以和一个引用队列(ReferenceQueue)联合使用,
//如果软引用所引用对象被垃圾回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。
ReferenceQueue<Object> queue = new ReferenceQueue<>();
Object obj = new Object();
SoftReference softRef = new SoftReference<Object>(obj,queue);
//删除强引用
obj = null;
//调用gc
System.gc();
System.out.println("gc之后的值: " + softRef.get()); // 对象依然存在

//申请较大内存使内存空间使用率达到阈值，强迫gc
byte[] bytes = new byte[100 * 1024 * 1024];
//如果obj被回收，则软引用会进入引用队列
Reference<?> reference = queue.remove();
if (reference != null){
    System.out.println("对象已被回收: "+ reference.get());  // 对象为null
}
```



### 弱引用 - 只要发生 GC 

弱引用的特点是不管内存是否足够，只要发生 GC，都会被回收

```java
package com.mikechen.java.refenence;
 
import java.lang.ref.WeakReference;
 
/**
* 弱引用
*
* @author mikechen
*/
public class WeakReferenceDemo {
    public static void main(String[] args) {
        Object o1 = new Object();
        WeakReference<Object> w1 = new WeakReference<Object>(o1);
 
        System.out.println(o1);
        System.out.println(w1.get());
 
        o1 = null;
        System.gc();
 
        System.out.println(o1);
        System.out.println(w1.get());
    }
}
```



### 虚引用 - 仅持有虚引用，任何时候都可能被垃圾回收

最弱的一种引用关系

```java
A a = new A();
ReferenceQueue<A> rq = new ReferenceQueue<A>();
PhantomReference<A> prA = new PhantomReference<A>(a, rq);
```

主要用来跟踪对象被垃圾回收器回收的活动

虚引用必须和引用队列 （ReferenceQueue）联合使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之 关联的引用队列中





## C# 的运行机制

![](images\C# 是怎么执行的.png)

### CIL

​	公共中间语言 - DLL文件	

​	包含 元信息 和 IL 汇编



ILSpy：反汇编工具查看

1. 元信息：DLL包含的信息类型的定义，方法的定义（反汇编工具中的方法的定义）
2. IL 汇编：方法体实际内容（反汇编工具中每个方法具体的汇编内容）

MSDN网站有每个指令的作用



### Mono.Ceil

1. 用于读取C#编译的DLL 的开源第三方库
2. 可以读取 DLL 中的类型和方法元信息
3. 可以读取方法体的 IL汇编指令
4. 可以读取 PDB 调试符号表文件
5. 可以修改 DLL 中的 元信息 和 方法体内容 并写回 DLL
6. 广泛应用于各自热更方案，如 toLua、xLua



## CLR和.Net对象生存周期

公共语言运行时（CLR）是一个可以由多种编程语言使用的运行时，如同java的JVM（Java Virtual Machine）。CLR的核心功能包括**内存管理，程序集加载，类型安全，异常处理和线程同步**，而且还负责对代码实施严格的类型安全检查，保证代码的准确性，这些功能都可以提供给面向CLR的所有语言（C#，F#等）使用。

CLR并不关心开发人员使用什么语言来进行编程，只要我们使用的编译器（充当语法检查器和‘正确代码’分析器）是面向CLR的就行。常见的语言编译器包括C++/CLI，C#，F#，VB和一个中间语言汇编器（Intermediate Language，IL） ，以下是编译器编译代码的过程，可以看到最终都是生成包含中间代码（IL）和托管数据（可进行垃圾回收的数据类型）的**托管模块**。

托管模块是标准的32位或64位Microsoft Windows可移植执行体文件，主要由以下几部分组成

- PE32或PE32+
- CLR头
- **元数据**
- **IL代码（基于栈，也称为托管代码）**

![img](images\CLR和.Net对象生存周期.png)



托管代码：由公共语言运行库环境（而不是直接由操作系统）执行的代码。托管代码应用程序可以获得公共语言运行库服务，例如自动垃圾回收、运行库类型检查和安全支持等。这些服务帮助提供独立于平台和语言的、统一的托管代码应用程序行为。

非托管代码：在公共语言运行库环境的外部，由操作系统直接执行的代码。**非托管代码必须提供自己的垃圾回收、类型检查、安全支持等服务**；它与托管代码不同，后者从公共语言运行库中获得这些服务。例如COM/COM++组件,ActiveX控件,API函数,指针运算,自制的资源文件，一般情况下我们会采取手动回收，如**调用Dispose接口或使用using包裹逻辑块**，



### CLR 的 垃圾回收（CC）

垃圾回收器（基于 **代** 的 分代式垃圾回收器）采用引用跟踪算法，在CLR中用作自动内存管理器，用于控制的分配和释放的托管内存。

GC 解决了可能存在的**内存泄漏**和因为访问被释放内存而造成的**内存损坏**的问题。

垃圾回收器 使用引用计数算法（该算法只关心引用类型变量），跟踪并回收托管内存中分配的对象。垃圾回收器**会定期执行垃圾回收**来回收内存分配给对象没有有效的引用。**当无法满足内存要求，使用可用的可用内存（如new 时发现内存占满），垃圾回收时会自动发生**。或者，应用程序**可以强制垃圾收集**使用 Collect 方法。



当CLR在托管堆上为非垃圾对象分配地址空间时，**总是分配出新的地址空间，且呈连续分配**。

对于未装箱的值类型对象而言，由于其不在堆上分配，一旦定义了该类型的一个实例的方法不再活动，为它们分配的存储资源就会被释放，而不是等着进行垃圾回收



#### 代的设计思路（降低GC对性能影响）

- 对象越新，生命周期越短，反之也成立
- 回收托管堆的一部分，速度快于回收整个堆

托管堆中的每个对象都可以被分为0、1、2三个代（System.GC.MaxGeneration=2）：

- 第 0 代： 从没有被标记为回收的新分配对象
- 第 1 代： 在上一次垃圾回收中没有被回收的对象
- 第 2 代： 在一次以上的垃圾回收后仍然没有被回收的对象.





引用类型变量 称为 根

1. 所有的全局和静态对象指针是应用程序的根对象，
2. 在线程栈上的局部变量/参数也是应用程序的根对象，
3. CPU寄存器中的指向托管堆的对象也是根对象



当满足以下条件之一时CLR将发生垃圾回收：

1. 系统具有低的物理内存。
2. **由托管堆上已分配的对象使用的内存超出了可接受的阈值**。随着进程的运行，此阈值会不断地进行调整。
3. 强制调用 GC.Collect 方法。
4. CLR正在卸载应用程序域（AppDomain）
5. CLR正在关闭。



整个垃圾回收过程包括以下步骤 ︰

- 垃圾回收器搜索托管代码中引用的托管对象。
- 垃圾回收器尝试完成未被引用的对象。
- 垃圾回收器释放未被引用的对象，并回收它们的内存。



#### 垃圾回收做的事情

- GC的准备阶段
  在这个阶段，**CLR会暂停进程中的所有线程**，这是为了防止线程在CLR检查根期间访问堆。

- GC的标记阶段
  当GC开始运行时，它会假设托管堆上的所有对象都是垃圾。也就是说，假定没有根对象，也没有根对象引用的对象，

  然后 **GC开始遍历根对象** 并构建一个由所有和根对象之间有引用关系对象构成的**对象图**，然后，GC会挨个遍历根对象和引用对象，假如一个根包含null，GC会忽略这个根并继续检查下个根（这很关键）。反之，假**如根引用了堆上的对象，GC就会标记那个对象并加入对象图中**。如果GC发现一个对象已经在图中就会换一个路径继续遍历。这样做有两个目的：一是提高性能，二是避免无限循环。

**注意：将引用赋值为null并不意味着强制GC立即启动并把对象从堆上移除，唯一完成的事情是显式取消了引用和之前 引用所指向对象之间的连接**



NextObjPtr对象始终保持指向最后一个对象放入托管堆的地址

被标记的对象至少被一个根引用，我们把这种对象称为**可达**（也称为幸存），反之称为**不可达**



- **GC的碎片整理阶段**
  所有的根对象都检查完之后，GC构建的对象图中就有了应用程序中所有的可达对象。托管堆上所有不在这个图上的对象就是要做回收的垃圾对象了。同时，CLR会对堆中非垃圾对象进行位置上的整理，使它们覆盖占用连续的内存空间（这个动作还伴随着对根返回新的内存地址的行为），这样一方面恢复了引用的“局部化”，压缩了工作集，同时空出了空间给其他对象入住，另外也解决了本机堆的空间碎片化问题。
- **GC恢复阶段**
  完成了综上的所有操作后，CLR也恢复了原先暂停的所有线程，使这些线程可以继续访问对象。



#### 垃圾回收的缺点

会有显著的性能损失，这是使用托管堆的一个明显的缺点

代就是一种为了降低GC对性能影响的机制



#### 代的工作原理

1、托管堆在程序初始化时不包含对象，这时候添加到堆的对象就是第 0 代对象，这些对象并未经历过GC检查。一段时间后，C，F，H对象被标记为不可达

![](images\代的工作原理_1.png)

2、CLR初始化时为第0代对象选择一个预算容量，假如这时分配一个新对象造成第0代超过预算，此时CLR就会触发一次GC操作。比如说A-H对象正好用完了第 0 代的空间，此时再操作时就会引发一次GC操作。GC后第 0 代对象不包括任何对象，并且第一代对象也已经被压缩整理到连续的地址空间中。

垃圾回收发生于第 0 代满的时候

![](images\代的工作原理_2.png)

3、每次新对象仍然会被分配到第 0 代中，如下图所示，CLR又重新分配了I-N对象，一段时间后，第 0 代和第 1 代都产生了新的垃圾对象

CLR不仅为第 0 代对象选择了预算，也为第 1 代，第 2 代对象选择了预算。
不过由于GC是自调节的，这意味着GC可能会根据应用程序构造对象的实际情况调整每代的预算（每次GC后，发现对象多存活增加预算，发现少存活减少预算），这样进程工作集大小也会实时不同，进一步优化了GC性能。

![](images\代的工作原理_3.png)

4、此时CLR再为第 0 代对象加入新对象时造成超过第 0 代预算的情况，GC将重新开启。GC将检查第 1 代预算使用情况，**假如第 1 代占用内存远少于预算，GC将只检查第 0 代对象，即便此时原来的第 1 代对象中也出现了垃圾对象**。这符合假设中的第一点，同时GC也不用再遍历整个托管堆，从而优化了GC操作性能。

![](images\代的工作原理_4.png)

5、此后，CLR仍然是按照规则对第 0 代分配对象，直到第 0 代预算被塞满才会发生垃圾回收，把对象补充到第 1 代中，此时分两种情况，假如第 1 代对象空间仍然小于预算，此时第 1 代中的垃圾对象仍然不会进行回收（如4图中所示）。

**假如第 1 代对象在某个时间段增长到超过预算的阶段，那么CLR将在下一次进行GC回收时，检查第 1 代对象，然后统一回收第 0 代和第 1 代中的垃圾对象**。回收以后，第 0 代的幸存对象提升到第 1 代，第 1 代的幸存对象提升到了第 2 代。此时第 0 代回归空余状态

至此，CLR已经进行了数次GC操作才最终将对象分配到了第 2 代中

![](images\代的工作原理_5.png)



#### **使用System.GC类控制垃圾回收**

```c#
托管堆上分配字节数：GC.GetTotalMemory(false)
当前系统支持的最大代数：GC.MaxGeneration
未被回收对象的代：GC.GetGeneration(实例对象)
GC回收： GC.Collect();
```



### 非托管资源的回收（CLR无法回收 需要自己手动）

需要显式释放非托管对象：

	1. FileStream 文件句柄
	1. 数据库连接信息
	1. ...



两种方式：可终结对象（Finalize）和可处置对象（IDisposable）



#### Finalize：终结器 - 不提倡使用

允许对象在判定为垃圾之后，在对象内存在回收之前执行一些代码

当一个对象被判定不可达后，对象将终结它自己，并释放包装着的本机资源，之后，GC再从托管堆中回收对象。

重写Object基类的Finalize方法，GC判定对象不可达后，会调用重写的该方法

析构函数：System.Object中, 的 Finalize()的虚方法。在GC完成以后才调用，所以这些对象的内存不是被马上回收的，并且会被提升到下一**代**，这增大了内存损耗（**析构函数的调用将导致GC对对象回收的效率降低**），并且Finalize方法的执行时间无法控制（所以 **析构函数中只能释放非托管资源而不能对任何托管的对象/资源进行操作**）。

重写Finalize方法的必要原因就是C#类通过平台调用或复杂的COM组件任务使用了非托管资源

确保Finalize方法尽可能快的执行，要避免所有可能引起阻塞的操作，包括任何线程同步操作，同时也要确保Finalize方法不会引起任何异常，如果有异常垃圾回收器会继续执行其他对象的Finalize方法直接忽略掉异常

Finalize方法的执行时间是无法控制的，只能被碎片搜集程序调用



如果已经完成了析构函数该干的事情（例如释放非托管资源），就应当使用 **SuppressFinalize** 方法告诉GC不需要再执行某个对象的析构函数。



#### IDisposable：可处置对象

尽快地开释一些不再运用的数量有限的非可管理性资源（如文件句柄）

调用Dispose方法只是**控制这个清理动作的发生时间**而已

Dispose方法也不会将托管对象从托管堆中删除



如果一个类拥有一个实现了 IDispose 接口类型的成员，并创建（注意是创建，而不是接收，必须是由类自己创建）它的实例对象，则 这个类也应该实现 IDispose 接口，并在 Dispose方法中调用所有实现了 IDispose接口的成员的 Dispose 方法。
只有这样的才能保证所有实现了 IDispose 接口的类的对象的 Dispose 方法能够被调用到，确保可以手动释放任何需要释放的资源。



在正常情况下，只有在GC之后，托管堆中的内存才能得以释放。我们的习惯用法是将Dispose方法放入try finally的finally块中，以确保代码的顺利执行



using语法糖，查看IL代码也发现具有相同的try finally块，只适用于那些实现了IDisposable接口的类型





#### Finalize 和 IDisposable 相同点

​	**都是为了确保非托管资源得到释放。**

#### Finalize 和 IDisposable 不同点

- finalize 由垃圾回收器调用；dispose 由对象调用。

- finalize 无需担心因为没有调用 finalize 而使非托管资源得不到释放，而 dispose 必须手动调用。

- finalize 虽然无需担心因为没有调用finalize 而使非托管资源得不到释放，但因为由垃圾回收器管理，不能保证立即释放非托管资源；而 dispose 一调用便释放非托管资源。

- 只有类类型才能重写 finalize，而结构不能；类和结构都能实现IDispose.原因请看Finalize()特性。




### 值类型不需要GC



#### 原因：栈帧

栈帧包含代码的执行顺序以及函数的调用顺序

栈上会保留值类型数据，以及指向堆的指针

值类型数据 和 指向引用类型的指针 都可以超出被系统删除或者缓存

![](images\栈帧.png)



# C# Mono和IL2CPP JIT和AOT

![](images\C#_Mono和IL2CPP_JIT和AOT.png)





# C# Mono 三类组件和垃圾回收机制

![](images\C#_Mono_三类组件和垃圾回收机制.png)



# Unity的垃圾回收

## 贝姆垃圾回收器

​	Unity 2019 以前	

​	会在执行垃圾回收时暂停运行中的程序，当完成垃圾回收后才恢复运行状态。

​	可能会导致程序在运行过程中随时会出现延迟执行的现象

​	

## 贝姆垃圾回收器 + 增量式垃圾回收

​	Unity 2019 开始

​	将任务分解为多个部分。用多个短时间的中断来完成垃圾回收。

​	通过分配工作量到多个帧，显著减少GC峰值对动画流畅性的影响问题。

​	Player Settings窗口的“Other settings”部分，勾选Use incremental GC (Experimental)



缺点：需要根据自己的项目考虑是否使用

1、在增量式垃圾回收分解任务时，分解的部分是标记阶段，此时它会扫描所有托管对象，寻找对象引用的其它对象，从而跟踪仍在使用的对象。

1. 这个过程会假设对象间的多数引用不会在任务分解时改变。
1. 当引用改变时，修改的对象需要在下一次迭代过程重新扫描。该过程可能会造成增量式回收无法完成，因为它总会增加更多任务。
1. 这种情况下，**垃圾回收会回退为完整的非增量式回收**，这种情况下增量式垃圾回收会比非增量式垃圾回收的效果更差。

2、在使用增量式垃圾回收器时，**Unity需要生成额外代码，用于在引用变化时通知垃圾回收器**，从而使垃圾回收器知道是否需要重新扫描对象。在引用变化时，该做法会**增加性能开销**，在部分托管代码中造成显著的性能影响。



## 贝姆垃圾收集器的缺点

1、无法区分指针和非指针（比较保守）

2、会占用更大的堆内存，

3、导致有些原本已经可以释放的内存无法释放。



# Unity 进程、线程、协程





![](images\Unity 进程_线程_协程.png)



1、CPU高速轮询调度，当调度到进程的时候，会把资源的使用交给进程使用

2、进程在使用的过程中，如果是有开辟多个线程，会把主线程和子线程一起分配在CPU上，让CPU同时运行线程



## 进程 Process 

1、程序的一次运行活动，是系统进行资源分配和调度的基本单位

​	**CPU 的执行是高速轮询调度，交替运行多个进程**

2、程序的基本执行实体

3、进程是线程的容器

​	没有进程 无法执行线程



Unity 程序运行在主线程，单线程



多进程：

多进程之间可以进行更多CPU的占用

有多进程，其中一个宕掉的情况下，可以动态匹配到另外一个进程去让客户端使用



## 线程 thread

1、操作系统能够进行运算调度的最小单位

​	主线程可以开辟多个子线程	

​	线程是为了提升CPU的使用率，如果只是单个执行，对CPU的占用率很高，

​	目的：解决类似网络访问，加载资源（IO），大数据运算 的造成的CPU的卡顿 

2、是进程中的实际运作单位

3、线程可以并发并行



必须有一个主线程，对象的生命周期都是在主线程中调用



多线程缺点：发热



**多核 = 能同时处理线程的数量也越多**

单核的电脑进行多线程开发也有意义：虽然同时处理线程是单核处理，但是多线程的时候可以把密集运算（磁盘IO）用线程去执行，这些线程在CPU中也是有专门处理单元处理并发问题，也是提高了处理效率。



线程单独执行自己的任务，之间互相不影响

主线程被杀死不会影响子线程，除非 程序的进程 被杀死

但是如果子线程在创建的时候定义为后台线程（守护线程），主线程关闭的时候会随着一起被关闭



休眠：Thread.Sleep(毫秒数)

终止：Abort 注意关闭游戏主线程的时候，也要把其他子线程终止关闭



问题1：

**Unity的 API 只能被主线程调用，子线程无法调用 Unity 的 API**

解决：把子线程的任务合并到主线程中（loom插件）

在主线程中创建一个 List<Action> 的 委托集合，主线程一直去遍历这个集合，注意要加锁



问题2：

**多个线程同时访问同一个变量或资源（竞争使用）**，没有按照正常结果输出

解决：加锁 或者 使用线程安全的类型

1、线程安全类型

```c#
ArrayList list = ArrayList.Synchronized(new ArrayList(1000000));
```

2、对变量加锁 lock

加锁会出现的问题：死锁

死锁：一个线程访问2个资源，另一个线程同时访问这2个资源，但这时候两个线程是并发执行。当第一个线程把第一个资源占了，而第二个线程把第二个资源占了，第一个线程在等待第二个线程释放掉第二个资源才可以执行下去，第二个线程在等待第一个线程释放掉第一个资源才可以执行下去。**两个线程都占了各自必要的资源，都需要等对方释放，造成死锁，无止境的等待下去。**



锁的类型

互斥锁（Mutex）：性能消耗远高于 lock，但好处是可以在进程之间或者线程之间有个预加锁概念（实际上锁住的调用方），一旦锁上后，其他的线程都不能访问，也不能访问别的，还必须要释放，不释放会出现死锁（需要手动释放）

lock：锁住的是被调用方，锁住的只是一段，不用手动释放





## 协程 

1、具有多返回点的方法

​	yield return 可以有多个

​	协程类似于迭代器，每迭代一次，下一次迭代的点就会指向下一个 yield return

2、time slicing ：时间分片（帧）

​	在指定的时间分片执行后 yield return，下一次执行指向下一个时间分片

3、单线程



**协程类似多线程，运行在主线程里**



Unity 本身是单线程，**Unity 的 API 和 控件都只能在主线程使用，无法在子线程使用**

把运算放在 Unity的同一帧运算会造成卡顿，使用协程将运算放在不同的帧上分散计算（yield return null 等待1帧）

从而可以在Unity中得到类似于多线程的解决方案



1、协程具有延迟的功能

2、使用协程www加载AB包

3、使用协程等待结果的完成（挂机程序）





