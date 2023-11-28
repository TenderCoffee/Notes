# Unity性能优化检查清单

[![吴帅](https://picx.zhimg.com/v2-c815f265e4b4575588fec965bcdbc987_l.jpg?source=172ae18b)](https://www.zhihu.com/people/wu-shuai-47-7)

[吴帅](https://www.zhihu.com/people/wu-shuai-47-7)

69 人赞同了该文章

如果你的游戏有性能问题，请检查以下内容：

**图形设置：**

- Camera RenderScale
- Camera AA选项
- Camera 投影设置
- Active Camera Number
- 看不到任何物体的空转相机
- Camera 近裁剪面和远裁剪面设置
- 每个Camera Renderer关联各自的配置, 如：UI和场景分开Renderer独立配置需要的Feature
- Texture 分辨率
- Texture 压缩格式
- Texture Mipmap设置
- Texture Read&Write设置
- Texture 三线性插值
- Texture POT
- Texture 图集
- Texture read/write pixels
- RT 尺寸
- RT 数量
- RT 抗锯齿格式设置
- RT 格式 RGBA vs Half
- RT 精度 F32 vs F16
- RT depth stencil color 开关
- RT Filter Mode
- RT RenderBufferLoadAction
- 后处理开关
- 后处理 Bloom贴图尺寸
- 后处理 Bloom模糊等级
- 后处理抗锯齿设置
- 运行时光源数量
- 相机，灯光，投影等 CullingMask
- 模型顶点数 & 面数
- 模型LOD设置
- 模型Mesh冗余通道
- 材质Cull Off -> Cull Back/Cull Front
- Mesh Compression
- Mesh Read/Write
- Mesh Disable rigs and BlendShapes
- URP 阴影投射设置
- 灯光投影设置
- 阴影分辨率
- 半透贴片阴影
- Planner Shadow
- 投射 & 接受影子
- CSM 设置
- 软阴影与硬阴影实现
- Shader if 条件语句优化
- Shader 反三角函数使用
- Shader discard/clip vs Pre-Z
- Shader 指令数
- Shader Texture读取数量
- Shader half & float 数据类型
- Shader LOD
- Shader Matcap
- Shader 模板测试
- Shader 多PASS
- Shader if 条件语句优化
- Shader Collection 变体预热
- Shader move PS calc to VS calc
- Shader move GPU calc to CPU calc
- Shader for 循环数量
- Shader for 循环展开
- Shader PBR -> BllinPhong -> VertexShading -> LUT or Matcap -> Lightmap -> SH
- Shader 顶点输入结构体&传入片源结构体，精度 & 冗余通道
- Shader 无意义的Normalize 操作
- Shader MaterialPropertyBlock
- Shader multi_compile -> shader_feature
- Shader grabpass
- SRP Batcher URP配置开关
- SRP Batcher Shader兼容性
- SRP Batcher 真机与PC兼容性不一致
- InstanceDraw
- 渲染状态切换 - Set Pass Call
- OverDraw
- 半透明物体渲染数量/屏幕占比
- 半透渲染有效区域与其表现模型更加贴合
- 不透明物体渲染排序 vs pre-Z
- Occlusion Culling
- MRT Render

**粒子系统：**

- 粒子系统持续时间
- 粒子系统发射间隔
- 粒子最大数量
- 粒子系统发射模型面数
- 粒子系统RingBuffer开关
- 粒子系统物理碰撞选项
- 粒子需要碰撞地面或者墙壁时，Collision中选择Planes模式
- 帧动画代替粒子系统、粒子系统表现烘培成序列帧动画

**物理设置：**

- 物理分层
- 简化物理碰撞体
- 使用LODMesh进行MeshCollider
- 使用组合简化的碰撞模型代替MeshCollider
- Fixed Timestep (Edit > Project Settings > Time)
- 避免MeshCollider
- Prebake Collision Meshes

**动画与骨骼：**

- 骨骼动画时间
- 骨骼动画压缩精度
- 骨骼动画骨骼数量
- CPU骨骼动画烘培成GPU顶点动画
- 动态骨骼数量
- 动态骨骼物理碰撞检测
- Animator Culling Mode
- Animator 状态机复杂度
- Animator 激活与关闭时机
- Animator Initialize
- Animation/Animator -> Tween 组件
- Animation 压缩误差值设置
- Animation Generic vs Humanoid Type
- GPU Skinning

**UI优化：**

- UI Canvas子节点动静分离
- UI Sprite Packer
- UI 运行时动态合并图集
- UI 自动布局组件
- UI 动态加载&剪裁
- UI 组件中自定义材质球打断合批
- UI Raycat Target
- UI 不可见元素 Cull Transparent Mesh
- UI 字体
- UI 计时器/倒计时 触发 UI 重绘
- UI Spirit 九宫格设置
- UI GameObject Active/Deactive -> UI CanvasGroup Alpha
- UI Canvas Pixel Perfect
- UI shadow 顶点和面数翻倍
- UI outline 顶点和面数翻倍
- UI 元素 position Z值不为 0 打断合批
- UI Mask vs RectMask2D
- 加载页/活动图等几乎充满屏幕的UI，关闭场景相机渲染，或渲染一张RT模糊后作背景

**音频设置：**

- Audio 双声道 -> 单声道
- Audio Compresssion Format, WAV vs Audio ogg, mp3....
- Audio 压缩比特率 or Sample Rate
- Audio ForceToMono
- Audio LoadInBackground
- Audio LoadType Decompress On Load vs 流式加载

**代码优化：**

- 对象池 or 缓存池 的初始化，使用&生命周期管理（预加载&回收复用）
- Transform.SetPositionAndRotation
- string plus operator -> string.Format -> StringBuilder
- 反射
- 正则表达式
- 装箱 & 拆箱
- LINQ
- 分帧处理 & 协程
- 逻辑代码容器选择 List，Queue，Stack...
- 优先使用数组而不是容器
- Camera.main 调用
- Start/Awake/OnEnable 执行重度逻辑
- 移除空的回调，比如：Awake，Start，Update等
- GC 频率，时机
- 多线程
- Burst
- SIMD，NEO
- ECS，DOTS
- Jobs
- IO -> 合并IO读写内容/异步读写
- 随机数 -> 伪随机数 -> 随机数池
- 使用 Dirty Flag 标记需要的信息，按照变化量执行逻辑，减少多余逻辑消耗
- 开发版本与发布版本的宏定义区分
- try catch exception -> error code
- Singleton模式/静态变量容器导致的资源常驻
- DontDestroyGameObject
- Resources.UnloadUnusedAssets
- Use OnBecameVisible() and OnBecameInvisible() callbacks
- Use sqrMagnitude for comparing vector magnitudes
- Shader.PropertyToID
- Animator.StringToHash
- AddComponent & GetComponent
- GetComponent -> Cache Compoent
- GameObject.Find
- compareString -> GameObject.CompareTag
- SendMessage
- GameObject Active&Deactive -> Renderer enable&disable -> Renderer.forceRenderingOff
- 隐藏物体的Update调用
- FixedUpdate->Update/LateUpdate -> 间隔固定时间更新
- Unity C# 与 LUA 交互调用
- Unity C# 调用第三方Jar包
- LUA 闭包
- move C# logic to native implementation
- AOI ( Area Of Interest )
- TCP/UDP -> KCP
- 序列化与反序列化
- 音频编码和解码
- 合并网络包
- 压缩网络包
- 网络包解析

**其他优化：**

- OpenGLES -> Vulkan
- AssteBundle.Load & AssetBundle Instantiate
- Asset Sync Load vs Aysnc Load
- Accelerometer Frequency
- 使用常用字精简字库减少存储空间
- 使用暗黑配色减少AMOLED耗电
- 日志开关
- 预制件子节点机型分档
- 场景结构复杂度 or 场景嵌套子节点数量
- 场景分区加载
- 场景分LOD动态加载
- 场景异步加载
- 游戏局外与战斗内 帧率，分辨率... 分离
- 小地图相机，分屏相机，缓存RT，分帧绘制 & 降低更新频率
- Forward Render vs Defer Render
- Dynamic Light vs Lightmap
- Lightmap vs Light probe
- Application.targetFrameRate
- Screen.SetResolution
- 静态合批设置
- 动态合批设置
- Mono vs il2cpp
- 渐进式GC选项
- BSDiff差异化更新
- 多线程渲染
- GPU 信息回读 - 截屏操作等
- Development Build选项开关
- URP settings: depth texture/ opaque texture
- URP settings: Transparent Receive Shadows开关
- Streaming Mipmap
- 配置表描述性字段删除
- 配置表相同内容字典压缩
- 配置表按需加载
- 配置表客户端服务器分离
- 配置表字段精度
- 配置表通用压缩，LZMA/LZ4
- 动态分辨率
- 动态锁帧率
- 动态画质
- VSync
- 移除无用目标平台CPU架构/图形API
- 更新Unity引擎到最新版本

编辑于 2023-08-03 15:55・IP 属地上海