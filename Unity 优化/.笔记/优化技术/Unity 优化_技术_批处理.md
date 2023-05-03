# 批处理

因为管线渲染可以接受一系列没有共同边的顶点

所以可以将大量任意数据块组合在一起并将它们作为单个大数据块进行处理



在内存中的不同位置来回切换内核需要时间，使用批处理后，不需要切换内核，GPU 可以充分使用多核同时处理多个任务

将多个对象的网格数据拼合到一起，在单一指令中渲染，而不是单独准备和绘制每个几何体



## 批处理的对象

网格、顶点、边、UV坐标、其他用于描述3D对象的不同数据类型的大集合



## DrawCall - SetPass Call

从 CPU 发送到 GPU 中用于绘制对象的请求 

初始化当前渲染过程之前的配置选项



请求DrawCall 之前：

1、网格和纹理数据从CPU内存（RAM）推送到 GPU 内存（VRAM）中

2、CPU 配置处理对象（DrawCall 的目标）所需要的选项和渲染特性

3、**修改渲染状态：为准备管线渲染而配置的大量设置 - 耗时的过程**

​	GPU 是 大规模的并行系统，在修改渲染状态之前一直等待，直到所有当前的作业达到同步点为止	

​	即：GPU 的最快内核需要停下，等待最慢的内核赶上，浪费了可以用于其他任务的时间

4、发送 Graphics指令 到 **Command Buffer** 的队列中

​	Command Buffer = CPU 创建的指令 + GPU 每次执行完前面的命令后从中需要提取的指令

5、取到指令后，使用 Graphics API 进行 CPU 和 GPU 之间的通信（DirectX、OpenGL、Metal、Vulkan ...）到 GPU  绘制



触发渲染状态同步的操作：

立即推送一张新纹理到 GPU 中、修改着色器、照明信息、阴影、透明度、其他任何图形设置



**新的DrawCall 不一定代表着必须配置新的渲染状态**

如果两个对象共享完全相同的渲染状态信息，那么 GPU 可以立刻开始渲染新对象，因为在最后一个对象完成渲染之后，还维护着相同的渲染状态，这消除了由于同步渲染状态而浪费的时间，也减少了需要推入 Command Buffer 中的指令数，减少了 CPU 和 GPU 上的工作负载。



## 材质

着色器的容器

**用于展示渲染状态给开发者**

每个材质必须有一个着色器



**最小化渲染状态修改的频率 = 减少场景中使用的材质数量**

1、CPU 每帧花费更少时间生成指令并传输给 GPU

2、GPU 不需要经常停止，重新同步状态的变更



## 着色器

用于定义 GPU 应该如何渲染输入的顶点和纹理数据的简短程序

本身没有状态信息

每个着色器需要一个材质



# 调试工具 - Frame Debugger

确定批处理如何影响场景

任何API调用和DrawCall 的开销都差不多，但复杂场景中的大多数API调用都采用 DrawCall的形式

所以，**通常最好先关注 DrawCall 最小化**，再去关心诸如后期处理效果等 API 通信开销



# 动态批处理

1、批处理在运行时生成（批处理动态产生）

2、批处理中包含的对象在不同的帧之间可能有所不同，取决于哪些网格在主摄像机视图中当前是可见的（批处理内容动态）

3、在场景中运动的对象也可以是批处理（对动态对象有效）



自动识别共享材质和网格信息的对象

如果使用了两个不同的材质文件，虽然材质内部设置完全相同，但渲染管线的智能无法发现这一点，还是会当成不同的材质，从而不执行动态批处理



## 用在哪里

1、到处是 石头、树木 和 灌木 的森林

2、简单而常见的元素的建筑、工厂或空间站

3、包含很多动态的非动画对象，还包含简单的几何体和粒子特效



## 顶点属性

网格文件中基于每个顶点的一段信息，每一段为一组浮点数

**<300个顶点 && <900 个顶点属性 => 触发动态批处理**

包括：

1、顶点位置：相对于网格的根

2、法线向量：一个从对象表面指向外面的向量（通常用于光照）

3、一套或多套纹理 UV坐标：用于定义一张或多张纹理如何包裹网格

4、每个顶点的颜色信息：通常用于自定义光照或扁平化着色、低多边形风格的对象



验证顶点属性数量：把网格对象拖到场景中，在 Project 窗口中找到 MeshFilter 组件，在 Inspector 窗口的 Preview 子区域中查看 verts 值



## 网格缩放

使用统一的等比缩放（x、y、z 都是相同的值）

**等比缩放才可以放到同个批处理中**



注意：不使用负数的缩放

奇数个负数缩放 和 偶数个负数缩放 无法合为一个批次

只对网格的一个轴或者3个轴进行负数缩放的网格，会放到一个不同的批处理中



对象的渲染顺序可以决定什么网格可以进行批处理，如果先前的对象出现在与当前对象不同的批处理组中，则无法对其进行批处理



# 静态批处理

静态批处理会将所有的 static 标记的可见网格数据复制到一个更大的网格数据缓冲中，并通过一个 DrawCall 传到渲染中同时忽略原始网格



## 使用规则

1、网格必须标记 static

2、每个被静态批处理的网格都需要额外的内存

3、合并到静态批处理中的顶点数量是有上限的

4、网格实例可以来自任何网格数据源，但必须使用相同的材质引用



缺点：

1、不能修改对象的变换（移动、旋转 和 缩放）

2、消耗的额外内存：取决于批处理的网格中的复制的次数

​	而因为引用了相同的网格数据，所以渲染多少个消耗的内存都是相同的



共享材质引用是减少渲染状态变更的一种方式



使用 **FrameDebugger** 验证静态批处理是否正确生成



在运行时实例化静态网格：

StaticBatchUtility.Combine 

1、不能用在有很多顶点要合并的情况，因为消耗会很大

2、即使有相同材质，也不会将给定的网格和任何预先已经存在的静态批处理合并在一起





# GPU Instancing

ALU：是专门执行算术和逻辑运算的数字电路

CPU与GPU的最大差别：CPU的ALU远远小于GPU的ALU



自动分批渲染：10000个对象显示 可以 被分成了20个批次Draw Mesh(instanced)进行显示

​	**PC 单批次最多可支持511个Instance的显示**（Unity 需要去平衡每个instance的属性和渲染batch）

相同材质 的网格对象 的 每个实例 中 **增加不同显示：添加变量**



## 合批原理

GPU 显示中的合批：

优化前：每绘制一个对象就调用一个转态转换

优化后：**把需要绘制的相同内容同时放到 CommandBuffer 中再通知GPU进行绘制**



在渲染一个模型时通过传递更多的数据让它绘制在不同的位置，从而减少 DrawCall

使用一个Draw call就能渲染具有多个 **相同材质** 的网格对象（树木或灌木丛的绘制）

网格的每个副本称为一个实例



## PC - 单批次最多 511 个实例

常量缓冲 - constant buffer，是一组计算着色器运行时不能更改的数据，用作图形程序时，常量缓冲可以是视角矩阵或颜色常量。在通用计算程序中，常量缓冲可以存放诸如信号过滤的权重和图像处理的说明等数据。



Shader中 CBUFFER_START 与 CBUFFER_END 中定义了

两个float4x4：unity_ObjectToWorldArray 和 unity_WorldToObjectArray



4×(float的size)×4(四行)×4（四列） = 64byte 两个矩阵为128byte

constant Buffer 为64KB = 65536 byte

65536btte/128byte = 512个对象

但一个 Unity_BaseInstanceID 为4个byte, 所以512个对象少一个刚好511



## 移动设备 - OpenGL 单批次最多 127 个实例

OpenGL上constant Buff只是16K

少了4倍：**127个对象**





## 插件

**Mesh Animator** 插件 中 的GPU的实例对象 可以播放动作

在 **Frame Debugger** 中如有Draw calls显示多个实例时会显示 “Darw Mesh**(Instanced)**”。



## 限制

1、不完全支持 SRP（高度可编程的渲染管线） 中 SRP Batcher

​	解决方式1：使用默认的URP Shader的时候把SRP Batcher关闭

​		在HRP-HightFidelity中选择UseSRPBatch,选择关闭

​	解决方式2：重写 Shader

2、不支持 SkinnedMeshRenderers（蒙皮网格渲染器组件）

​	另一种大场景多元素方案：Animation-Instancing：GPU的顶点动画 + SkinnedMeshRenderers

3、可以受光照的影响

4、**仅支持 MeshRender**

5、使用前提：使用相同网格 + 相同材质

6、单批次最多可支持511个Instance的显示



## 在Unity中使用实例化渲染

### 方式1：自动启用实例化渲染

使用支持实例化渲染的Shader，并勾选材质球上的启用开关，Unity便会对满足条件的物体，自动开启实例化渲染。

### 方式2：自定义Shader

### 方式3：手动实例化渲染

​	Graphics.DrawMeshInstanced

​	Graphics.DrawMeshInstancedIndirect



## 自动启用实例化渲染



1、在 Prefab命名 所使用的 Materia l中选择 **Enable GPU Instancing**

2、将 GameObject 挂载 CreateCube组件，将 Prefab 关联到 CreateCube 的 Instance Go 字段中

3、直接 Instantiate 实例化

```c#
using UnityEngine;

public class CreateCube : MonoBehaviour
{
    [SerializeField]
    private GameObject _instanceGo;//需要实例化对象
    [SerializeField]
    private int _instanceCount;//需要实例化个数
    [SerializeField]
    private bool _bRandPos = false;//是否随机的显示对象
    // Start is called before the first frame update
    void Start()
    {
        for (int i = 0; i < _instanceCount; i++)
        {
            Vector3 pos = new Vector3(i * 1.5f, 0, 0);
            GameObject pGO = GameObject.Instantiate<GameObject>(_instanceGo);
            pGO.transform.SetParent(gameObject.transform);
            if(_bRandPos)
            {
                pGO.transform.localPosition = Random.insideUnitSphere * 10.0f;
            }
            else
            {
                pGO.transform.localPosition = pos;
            }
            
        }

    }
}
```





## 自定义Shader 使用



### 1、修改 Shader 



支持 GPU Instancing 所需的 Shader关键字、变量和函数



SRP中 或 URP中 或 自己定义的Shader中：默认都不支持 Enable GPU Instancing

需要改造 让 Shader 支持 GPU Instancing



自定义 创建出来的 Shader和绑定的材质球 默认没有 Enable GPU Instancing



multi_compile_instancing

UNITY_VERTEX_INPUT_INSTANCE_ID

UNITY_SETUP_INSTANCE_ID

UNITY_TRANSFER_INSTANCE_ID



Shader改造：

```shader
//第一步： sharder 增加变体使用shader可以支持instance  
//作用：使Unity生成着色器的两个变体，一个具有GPU实例化支持，一个不具有GPU实例化支持
//Shader 如果是 片段和顶点着色器 则必需增加，如果是 曲面着色器则可选增加
#pragma multi_compile_instancing


//...

struct appdata
{
	//...
	//先启用 IINSTANCING_ON
	//第二步：instancID 加入顶点着色器输入结构 
	//SV_InstanceID语义的instanceID变量
	//instanceID主要作用是使用GPU实例化时，用作顶点属性的索引
	UNITY_VERTEX_INPUT_INSTANCE_ID
}

struct v2f
{
	//...
	
	//在顶点着色器 输入/输出结构体 中定义instance ID
	//增加一个SV_InstanceID语义的nstanceID变量，用作顶点属性的索引
	UNITY_VERTEX_INPUT_INSTANCE_ID
}

//顶点着色器：
//有了实例 就可以在顶点着色器中
//开始获取每个实例中的相关设置
v2f vert(appdata v)
{

	//...
	//第四步：instanceid在顶点的相关设置  
	//允许着色器函数访问实例ID
	//顶点着色器，开始时需要此宏
	UNITY_SETUP_INSTANCE_ID(v);
	
	//第五步：传递 instanceid 顶点到片元
	//在顶点着色器中将InstanceID从输入结构复制到输出结构。
	//如果需要访问片段着色器中的每个实例数据，请使用此宏
	UNITY_TRANSFER_INSTANCE_ID(v, o);
	
	//...
}

//片元着色器：
fixed4 frag(v2f i) : SV_Target
{
	//第六步：instanceid在片元的相关设置 片段着色器可选
	UNITY_SETUP_INSTANCE_ID(i);
	
	//...
}

```





### 2、个性化



#### C#材质属性块 

MaterialPropertyBlock

绘制相同材质，但属性有不同的多个对象时候可以使用

可以给每个实例对象通过 Render.SetPropertyBlock 设置相应的 MaterialPropertyBlock

```c#
//官方案例

using UnityEngine;

public class MaterialPropertyBlockExample : MonoBehaviour
{
    public GameObject[] objects;

    void Start()
    {
        //创建MaterialPropertyBlock
        MaterialPropertyBlock props = new MaterialPropertyBlock();
        MeshRenderer renderer;

        foreach (GameObject obj in objects)
        {
            float r = Random.Range(0.0f, 1.0f);
            float g = Random.Range(0.0f, 1.0f);
            float b = Random.Range(0.0f, 1.0f);
            //设置MaterialPropertyBlock所使用的颜色
            props.SetColor("_Color", new Color(r, g, b));
            //得到MeshRenderer
            renderer = obj.GetComponent<MeshRenderer>();
            //设置PropertyBlock
            renderer.SetPropertyBlock(props);
        }
    }
}
```



在 C# 代码中 使用 MaterialPropertyBlock 设置 需要与 Shader 沟通的字段

Setcolor 或是 SetFloat 传递到Shader中

给实例 的 meshRenderer 绑定 MaterialPropertyBlock 

```c#
using UnityEngine;

public class FunnyGPUInstance : MonoBehaviour
{
    [SerializeField]
    private GameObject _instanceGo;//初实例化对你
    [SerializeField]
    private int _instanceCount;//实例化个数
    [SerializeField]
    private bool _bRandPos = false;
 
    private MaterialPropertyBlock _mpb = null;//与buffer交换数据
    // Start is called before the first frame update
    void Start()
    {
        for (int i = 0; i < _instanceCount; i++)
        {
            Vector3 pos = new Vector3(i * 1.5f, 0, 0);
            GameObject pGO = GameObject.Instantiate<GameObject>(_instanceGo);
            pGO.transform.SetParent(gameObject.transform);
            if (_bRandPos)
            {
                pGO.transform.localPosition = Random.insideUnitSphere * 10.0f;
            }
            else
            {
                pGO.transform.localPosition = pos;
            }
            //个性化显示
            SetPropertyBlockByGameObject(pGO);

        }
    }

    //修改每个实例的PropertyBlock
    private bool SetPropertyBlockByGameObject(GameObject pGameObject)
    {
        if(pGameObject == null)
        {
            return false;
        }
        if(_mpb == null)
        {
            _mpb = new MaterialPropertyBlock();
        }

        //随机每个对象的颜色
        //_Color与_Phi是给Material中的Shader设置的参数
        _mpb.SetColor("_Color", new Color(Random.Range(0f, 1f), Random.Range(0f, 1f), Random.Range(0f, 1f), 1.0f));
        _mpb.SetFloat("_Phi", Random.Range(-40f, 40f));

        MeshRenderer meshRenderer = pGameObject.GetComponent<MeshRenderer>();
        if (meshRenderer == null)
        {
            return false;         
        }

       	//给相应的对象设置了MaterialPropertyBlock
        meshRenderer.SetPropertyBlock(_mpb);

        return true;
    }
```



Shader 获取 C# 中的 指定字段

UNITY_INSTANCING_BUFFER_START +  UNITY_INSTANCING_BUFFER_END

```shader
UNITY_INSTANCING_BUFFER_START(Props)
    UNITY_DEFINE_INSTANCED_PROP(float4,_Color)
    UNITY_DEFINE_INSTANCED_PROP(float, _Phi)
UNITY_INSTANCING_BUFFER_END(Props)
```



#### 顶点着色器

通过 UNITY_ACCESS_INSTANCED_PROP(Props, _Phi); 进行属性访问

使用Instance ID索引实例数据数组

```shader
 v2f vert (appdata v)
 {
   //...

   float phi = UNITY_ACCESS_INSTANCED_PROP(Props, _Phi);
   //修改显示 - 动起来
   v.vertex = v.vertex + sin(_Time.y + phi);
   o.vertex = UnityObjectToClipPos(v.vertex);
   o.uv = TRANSFORM_TEX(v.uv, _MainTex);
   UNITY_TRANSFER_FOG(o,o.vertex);
 }
```



#### 片元着色器

通过 UNITY_ACCESS_INSTANCED_PROP(Props, _Color); 进行属性访问

```shader
fixed4 frag (v2f i) : SV_Target
{
   //...

   //得到由CPU设置的颜色
   float4 col= UNITY_ACCESS_INSTANCED_PROP(Props, _Color);
   return col;   
}
```









