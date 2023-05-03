# 图片格式

## 1、ETC

- 长宽需要**POT（2的幂次方）**，可以不是正方形
- **不支持Alpha**，所以需要将ETC分离出Alpha通道，可在Override for iOS勾选上Split Alpha Channel
- 不支持单通道R和双通道RG
- 只支持OpenGL ES 2，推荐使用ETC2
- 适用于Android 和 iOS

## 2、ETC2 

- ETC2格式需要 **OpenGL ES 3.0**以上才支持（目前大部分安卓和iOS手机都支持），在不支持的设备上， Unity 在运行时将纹理解压缩为ETC2 回退指定的格式，将**增加内存使用**
- 向下兼容ETC1
- 所有运行 Vulkan 或 Metal 或 OpenGL ES 3.0 的 GPU 都支持 ETC2 格式
- 适用于Android 和 iOS33gg!B

## 3、PVRTC

- IOS Unity默认使用PVRTC,为了更好的兼容性，因为PVRTC支持旧设备
- PVRTC4 需要二次方正方形，长宽一样
- 在图形卡中无需解压即可使用压缩图像
- 该算法在具有对比度或描边+alpha通道的图像中有弱点
- PVRTC 2bits 比 PVRTC 4bits 压缩多一倍，但质量较差
- 在UI上使用会在颜色渐变到较浅区域变得模糊

## 4、ASTC

- IOS A8(2014)处理器开始支持ASTC，并且更高的版本推荐ASTC
- 4x4、5x5、6x6、8x8、10x10、12x12
- 4x4压缩比最低，质量最高(可与rgb32看起来几乎无差别)
- 12x12压缩比最大，质量最低

## 5、RGB

- 高清无压缩不带alpha通道
- 推荐在Windows、Linux、macOS、webGL平台使用

## 6、RGBA

- 高清无压缩带alpha通道
- 推荐在Windows、Linux、macOS、webGL平台使用



# 图片计算内存

假设 图片大小 512 ×512

例子1：使用RGBA 32bit真彩（Truecolor），占用内存 = 4Bytes × 512 ×512 = 1MB

RGBA = 4个byte = 4位

例子2：使用RGB ETC 4bit压缩，占用内存 = 0.5Bytes × 512× 512 = 128KB

![图片压缩](.\images\图片压缩.webp)



# 图片压缩

一、**2的N次方大小的图片**会得到引擎更大的支持，包括压缩比率，内存消耗，打包压缩大小，而且支持的力度非常大。

二、减小图片的占用大小和内存方式有:图片大小变化(Maxsize),色彩位数变化(16位，32位)，压缩(PVRC)。

三、U3D对于图片的格式是自己生成的，而并不是你给他什么格式，他就用什么格式，一张1024×1024图在无压缩格式下，它会被U3D以无压缩文件形式存放，也就是说U3D里的Texture Preview里显示的占用大小**MB不只是内存占用大小，还是空间占用大小



**RGBA32格式为无压缩最保真格式，但也是最浪费内存和空间的格式。**

RGBA16格式为无压缩16位格式，比32位节省一半的空间和内存。

RGBA Compressed PVRTC 4bits格式为PVRTC图片格式，它相当于把图片更改了压缩方式新生成了一个图片来替换原来的我们给的图片格式(比如我们给的是PNG格式)。



注意：

​	1、U3D所有图片的压缩格式都会以另一种方式来存储，不会以你给的方式来存储，只有你指定了某种格式，它才会转成你要的格式。

​	2、压缩格式在Android里并不一定有效，因为Android的机型多，GPU的渲染方式也不一样，有的是Nvidia，有的是PowerVR，**最最好的在安卓机子上启用RGBA16方式**，因为这个是适应所有机型的，并且比32位占用量少一半，但也需要因项目而异，只是推荐使用的格式，可以多用，但要权衡内存和显示效果。



# 图片优化 ETC/PVRTC/RGB +Alpha+Shader

为了优化图片的压缩大小，会采用将RGBA图分离成一张ETC压缩格式的RGB图和一张alpha图的方式

```c#
//GenAlphaTexture.cs
 
using UnityEngine;
using UnityEditor;
using System.IO;
 
public class GenAlphaTexture
{
    [MenuItem("GameTools/GenAlphaTexture")]
    public static void StartGenAlphaTexture()
    {
        var textures = Selection.GetFiltered<Texture2D>(SelectionMode.DeepAssets);
        foreach (var t in textures)
        {
            var path = AssetDatabase.GetAssetPath(t);
 
            // 如果提示图片不可读，需要设置一下isReadable为true, 操作完记得再设置为false
            // var imp = AssetImporter.GetAtPath(path) as TextureImporter;
            // imp.isReadable = true;
            // AssetDatabase.ImportAsset(path);
 
 
            var newTexture = new Texture2D(t.width, t.height, TextureFormat.RGBA32, false);
            var colors = t.GetPixels32();
            var targetColors = newTexture.GetPixels32();
            for (int i = 0, len = colors.Length; i < len; ++i)
            {
                var c = colors[i];
                targetColors[i] = new Color32(c.a, c.a, c.a, c.a);
            }
            newTexture.SetPixels32(targetColors);
 
            string fname = path.Split('.')[0] + "_a.png";
            File.WriteAllBytes(fname, newTexture.EncodeToPNG());
            
            // imp.isReadable = false;
            // AssetDatabase.ImportAsset(path);
            AssetDatabase.Refresh();
        }
    }
}
```

把原图和生成的alpha图的压缩格式都设置成ETC 4 bits，这样大小加起来比原图大小还要小一半。

如果是iOS平台，因为没有ETC压缩格式，

​		正方形尺寸的图片，可以用PVRTC的压缩格式，

​		非正方形尺寸，用RGB 16 bit的压缩格式。



| 平台       | Android    | iOS          |
| ---------- | ---------- | ------------ |
| 正方形图片 | ETC 4 bits | PVRTC 4 bits |
| 长方形图片 | ETC 4 bits | RGB 16 bit   |



Shader文件：

```
// 合并一张ETC压缩的RGB图和一张alpha图
Shader "Test/UIETC" 
{
    Properties 
        {
                 _MainTex ("Base (RGB)", 2D) = "white" { }
                 _AlphaTex("AlphaTex",2D) = "white"{}
                 }
    SubShader
        {

                 Tags
                 {
                        "Queue" = "Transparent+1"
                 }
         Pass
                 {
                        Lighting Off
                        ZTest Off
                        Blend SrcAlpha OneMinusSrcAlpha
                        Cull Off
                        CGPROGRAM
                        #pragma vertex vert
                        #pragma fragment frag

                        #include "UnityCG.cginc"

                        sampler2D _MainTex;
                        sampler2D _AlphaTex;

                        float _AlphaFactor;
                
                        struct v2f 
                        {
                                float4  pos : SV_POSITION;
                                float2  uv : TEXCOORD0;
                                float4 color :COLOR;
                        };

                        half4 _MainTex_ST;
                        half4 _AlphaTex_ST;

                        v2f vert (appdata_full v)
                        {
                                v2f o;
                                o.pos = mul (UNITY_MATRIX_MVP, v.vertex);
                                o.uv =  v.texcoord;
                                o.color = v.color;
                                return o;
                        }

                        half4 frag (v2f i) : COLOR
                        {
                                half4 texcol = tex2D (_MainTex, i.uv);

                                half4 result = texcol;

                                result.a = tex2D(_AlphaTex,i.uv)*i.color.a ;

                                return result;
                        }
                        ENDCG
                 }
    }
}
```





# 纹理

纹理：简单的图象文件，一个颜色数据的大列表，以告知插值程序，图象的每个像素应该是什么颜色

精灵：网格的2D等价物，用于渲染面向当前相机的平面



纹理文件：在运行时加载进内存，推送到 GPU 的显存，并在给定的 DrawCall 期间，由着色器渲染到目标精灵或者网格上



 纹理颜色：每个通道的位数越多，可以表示的颜色越多，更大的纹理，更多的磁盘和内存消耗



# 减小纹理文件的大小

纹理文件越大，推送纹理所消耗的 GPU 内存带宽越多

如果每秒推送的总内存超过图形卡的总内存带宽就会产生瓶颈

因为在下一个渲染过程开始之前，GPU 必须等待所有纹理都上传完毕

（纹理越大 = GPU等待时间越长 ）



# MipMap

距离摄像机很近的渲染才使用高精度细节纹理

MipMap：提前生成相同纹理的低分辨率替代品，保证占据相同的内存空间。在编辑器内通过高质量重采样和过滤方法生成（**非运行时生成**）

GPU 根据透视图中的表面大小选择相应的 MipMap 级别（答题基于对象渲染时的纹理到像素的比例）

使用 MipMap 会增大纹理文件，消耗磁盘空间和上传到 GPU 的带宽



**纹理和摄像机的距离都是固定的，在某些业务条件下 MipMap 完全没用**：

如：

​	2D 游戏纹理文件

​	UI 系统使用的纹理

​	天空盒

​	远处背景的纹理

​	一直出现在玩家周围的角色

​	以玩家为中心的粒子特效

​	只出现在玩家周围的角色对象

​	只有玩家能持有/携带的对象



# Anisotropic Filtering

在非常倾斜的角度观察纹理时提升纹理品质的特性

开启后距离摄像机越远的线也依然清晰

很消耗



不需要倾斜角度的，可以禁用 Anisotropic Filtering：

远处的背景对象、UI元素、公告板粒子效果纹理



# 考虑使用图集

图集：

​	将许多较小的、独立的纹理合并到一个较大的纹理文件中

​	当所有给定的纹理需要相同着色器时采用的一种方法，如果需要着色器应用独立的图形效果就必须分离到自己的材质中并在单独的组中打图集



最小化材质的数量 = 最小化所需使用的 DrawCall 数量 （利用了**动态批处理**）

不要将动画角色的纹理文件合并到图集中：

​	**动态批处理 效果 只影响了 非动画的网格**（MeshRenderer 而不是 SkinnedMeshRenderer）

​	如果是动画角色的纹理文件，因为 GPU 需要将每个对象的骨骼乘以当前动画状态的变换，需要为每个角色进行独立的计算，无论是否共享了材质，这个计算都会导致额外的 DrawCall

 

瓶颈 在 CPU：

减少 DrawCall = 降低 CPU工作负载、提升帧率



无论2D或者3D 的游戏类型，只要具有简单纹理的分辨率，或者是扁平着色的低多边形风格，都可以使用图集



# 调整非方形纹理的压缩率

避免非正方形 或者 非2的次幂的纹理

​	2的n次幂的格式对 GPU 更友好

​	一些GPU需要方形的纹理格式，因为Unity将会自动扩展纹理到额外的空白处进行补偿，从而适应 GPU 期望的格式，这样会消耗额外的内存带宽，将本质上用不到和没用的数据推送到 GPU



# Sparse Textures

Sparse Textures：通过许多纹理合成一个巨大的纹理文件来实现，从而可以运行时从磁盘传输纹理数据流的方式

为了避免游戏运行时的硬盘访问（硬盘访问，比 CPU 的速度要慢的很多）



跟图集的区别：

1、包含纹理的文件非常大（32768×32768，每个像素32位 - 多颜色细节）

2、纹理文件甚至可以占用 4GB 的磁盘空间



主要开销：文件大小需求 和 潜在的连续磁盘访问

需要专业的硬件和平台支持



# 程序化材质

在运行时通过自定义数学公式混合小型高质量的纹理样本，通过程序化方式生成纹理的手段

在初始化期间 **以额外的运行时内存和CPU处理为代价**，极大减少应用程序的磁盘占用，以便 **通过数学操作** 而不是静态颜色数据来生成纹理



# 异步纹理上传



开启 Reard/Write Enable：

​	模拟在画布上画画 或者 将网络上的图象数据写入到已有的纹理

​	缺点：在纹理上传之前 GPU 必须始终等待对纹理所做的修改，因为无法预测什么时候发生更改

Reard/Write Enable：一般禁用



禁用后可以使用 **异步纹理上传：将纹理上传任务移到独立线程可以节省主线程大量的CPU时间**

1、纹理从磁盘异步上传到 RAM 中，当GPU 需要纹理数据时，会在渲染线程传输，而不是主线程传输纹理

2、纹理会推送到环形缓冲区中：

​	如果缓冲区有新数据，数据会持续不断推送到 GPU

​	如果缓冲区没有新数据，提前退出处理并等待，直到请求新的纹理数据 



异步纹理上传：适用于明确导入到项目中且在构建时存在的纹理







