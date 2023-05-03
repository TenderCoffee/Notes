# 汇编指令

内存数据 -> 缓存 -> 寄存器 -> ALU进行计算

程序员 -> 汇编指令 -> 编译器 -> 机器码 -> 计算机



mov 指令：将内存中的数据传递到寄存器中

​	例如： mov ax 8 ( 寄存器 数据 );

​				将寄存器中的数据传递到另一个寄存器中

​				mov ax bx ( 寄存器ax 寄存器bx );

​				mov ax 123VBH ( 寄存器 内存单元 )

加法：

​	add ax, [0]

​	将数据段偏移地址0 加到 ax中

​	ax -= 23VBH[0];



减法：

​	sub ax, 9

​	将数据9被 ax 减去

​	ax -= 9;



乘法：mul

除法：div



# SIMD 汇编指令

1、CPU 需要提供 128位 或者 256位寄存器（手机普遍都支持128位寄存器）

2、SIMD：**单条CPU 一次执行计算多条数据**

​		如：一个乘法指令同时计算多个数据的乘法

3、SIMD：基本可以在当今所有的CPU上使用

4、移动平台上有 ARM neon 指令集都支持 SIMD

5、PC平台上有 SSE 和 AVX 指令集 都支持 SIMD

6、C/C++/C#/Java 都原生支持 SIMD 指令



原本：

​	两个数据相加，CPUI一次只能执行一次运算

SIMD：
	CPU 一次执行 多个两个数据相加的运算



```c#
public static float DotExample(float3 a, float3 b)
{
    return math.dot(a, b);
}
/*
float3 a = (1,2,3)
float3 b = (1,2,3)
DotExample(a, b) 的结果应该是 1*1 + 2*2 + 3*3 = 14
真正4个浮点数一起相乘，只有第一条指令mulps，而且只用到了3个通道
剩下的汇编指令都是在做数据拷贝
做了两次addss 就是为了计算出它们相加的结果
*/
```



## SIMD 指令优化



```c#
public static float DotExample(float ax, float ay, float az, float bx, float by,float bz)
{
    return ax * bx + ay * by + az * bz;
}
/*
一个 SIMD 指令都没有
一共五个标量操作，效率最低
*/

//优化：
/*
使用float4可以保存4个独立的值
就可以充分利用 SIMD 指令
全都是 PS汇编指令（mulps addps）
这样的效率是最高的
*/
public static float DotExample(float4 ax, float4 ay, float4 az, float4 bx, float4 by,float4 bz)
{
    return ax * bx + ay * by + az * bz;
}

```



## 使用总结

1、避免代码出现分支预测，因为会打断 SIMD 的向量化指令

​	但是可以通过使用 math.select 和 math.lerp 代表分支预测

2、使用 float4 bool4 等代替 float 和 bool

3、使用 Unity 提供的数学库

4、使用 m128 自己组织 128位数据 ，效率上可以自己灵活控制

5、编译后尽量使用v开头指令，结尾尽量是 ps的指令，避免使用 ss 指令



## C# 平台 SIMD

System.Numerics 命名空间中，Matrix3x2、Matrix4x4、Plane、Quaternion、Vector2、Vector3、Vector4 类型，允许向量化 SIMD 计算。

System.Runtime.Intrinsics 命名空间，可以自由的发挥 SIMD指令。



```c#
//同个写法的不同实现
public int Naive()
{
    int result = 0;
    foreach(int i in Array)
    {
        result += i;
    }
    return result;
}

//基于 Linq的 实现
public long LINQ() => Array.Aggregate<int, long>(0, (current, i) => current + i);

//基于 System.Numerics 中的向量的 实现
public int Vector3()
{
    int vectorSize = Vector<int>.Count;
    var accVector = Vector<int>.Zero;
    int i;
    var array = Array;
    for(i=0; i<=array.Length - vectorSize; i+= vectorSize)
    {
        var v = new Vector<int>(array, i);
        accVector = Vector.Add(accVector, v);
    }
    int result = Vector.Dot(accVector, Vector<int>.One);
    for(; i < array.Length; i++)
    {
		result += array[i];
	}
    return result;
}

//基于 System.Runtime.Intrinsics 的代码实现
public unsafe int Intrinsics()
{
    int vectorSize = 256 / 8 / 4;
    var accVector = Vector256<int>.Zero;
    int i;
    var array = Array;
    fixed(int* prt = array){
        for( i=0; i<=array.Length - vectorSize; i += vectorSize){
            var v = Avx2.LoadVector256(ptr + i);
            accVector = Avx2.Add(accVector, v);
		}
	}
    int result = 0;
    var temp = stackalloc int[vectorSize];
    Avx2.Store(temp, accVector);
    for(int j=0; j<vectorSize; j++)
    {
        result += temp[j];
    }
    for(; i<array.Length; i++)
    {
        result += array[i];
    }
    return result;
}
```



# Unity.Mathematics 数学库

1、Unity.Mathematics 提供矢量类型（float4、float3...），可以直接映射到硬件 SIMD 寄存器

2、Unity.Mathematics  的 Math 类中也提供了直接映射到硬件 SIMD寄存器

3、原本CPU需要一个个计算，有了 SIMD 后就可以一次性计算完毕

4、注意： Unity之前的 Math类默认不支持 SIMD寄存器



```c#
public struct Position : IComponentData
{
    //不建议用 Vector3，使用 Vector3
    //public Vector3 Value;
    public float3 Value;
}

public struct Rotation : IComponentData
{
    //不建议 Quaternion
    //public Quaternion Value;
    public quaternion Value;
}


```



# LLVM - 编译器堆栈

动态编译技术

编译器分为3步：（和 GCC 编译器一样的策略）

1、前端：

	1. 检查源代码 是否存在错误
	1. 源代码 进行抽象语法树分析
	1. 生成中间 IR语言

2、优化

 	1. 对中间 IR 语言进行优化
 	2. 消除冗余计算
 	3. 内联
 	4. 变量折叠 等

3、后端

​	1. 将 IR 中间语言 生成目标机器 所能执行的代码



## LLVM IR 中间语言

LLVM：提供了一套 **LLVM IR 的中间语言**

前端无论是什么语言，只要通过语法分析生成 LLVM 所规定的 IR 中间语言

LLVM IR 的优化 和 后端，都可以最终生成目标平台的机器码

只能给值类型用，通过转换可以减少 CPU的运算量



# Burst 编译器

Burst 到最后是编译成本机的机器码

1、以 **LLVM** 为基础的后端编译技术

2、编译器的原理分为3个步骤：

​	源代码 -> 前端 -> 优化器 -> 后端 -> 机器码

3、LLVM 定义了 **抽象语言IR**，前端负责将源代码（C#）编译为 IR

​	优化器 负责 优化 IR

​	后端 负责 将 IR 生成目标语言（机器码）

4、LLVM 支持很多语言，也方便扩展

5、LLVM 代码开源，可以用来作为 Burst 的编译



**缺点：LLVM 对 C# 的 GC 做的不好，所以 burst 只支持值类型的数据编译，不支持引用类型数据编译**



1、使用原型块可以提供最优的内存布局

2、在系统中使用 遍历数组代替原先的访问游戏对象，更新时缓存行为的相关数据

3、利用 CPU 缓存机制，尽量从缓存中取数据，其次从内存中取数据

4、JobSystem 充分利用所有 CPU内核

5、Burst编译器 产生高度优化的机器代码

6、Burst编译器 可以在不同的CPU架构下的设备运行



## 不进行 Burst编译 - [BurstDiscard]

[BurstDiscard]：标记了不进行 Burst编译的方法

```c#
[BurstCompile]
struct Job : IJobParallerFor
{
    [WriteOnly] public NativeArray<float> values;
    [ReadOnly] public float offset;
    public void Execute(int index)
    {
        MethodToDiscard(index);
    }
}

//标记了不进行 Burst编译的方法
[BurstDiscard]
static void MethodToDiscard(int i)
{
    Debug.Log(i);
}

```



## 指定 Burst编译 - [BurstCompile] 

1、在 Job 上加上标签

2、在 静态类 和 静态方法 上

3、在结构体上

最终编译出来的是 SIMD 的 ps指令

