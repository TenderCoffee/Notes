

# 物理引擎的内部工作情况



## Unity 的物理引擎

3D - Nvidia 的 PhysX 和 2D - Box2D



## 物理和时间

物理引擎在时间按固定值前进的假设下运行的

物理引擎只使用非常特定的时间值来处理每个时间步长，与渲染的上一帧所花费的时间无关

时间步长 = 默认 20ms = 每秒50次更新



FixedUpdate 在内部的物理引擎执行之前处理

FixedUpdate 是一个用于放置任何期望独立于帧率的游戏行为的好地方

FixedUpdate  下一步执行后 是 协程的执行

FixedUpdate -> yield WaitForFixedUpdate -> （内部物理引擎更新） -> OnTriggerXXX  -> OnCollisionXXX

**注意：**

**1、如果自从上次 FixedUpdate 执行后，如果经过的时间小于设置的时间步长（20ms），则会跳过当前的这次 FixedUpdat  不去执行，此时输入、游戏逻辑和渲染将正常进行。完成这一次循环后，Unity 会检查处理下一次的 FixedUpdate 更新**

**2、在高帧率下，渲染更新可能在物理引擎更新自身之前完成多次更新**



物理引擎在对象的状态之间的可见位置做了插值，确保了对象在固定更新之间平稳移动



### 最大允许的时间步长 - 保证了渲染和逻辑正常执行

**注意：如果从上次 FixedUpdate 以后已经过了很长一段时间，那么 FixedUpdate 会继续在相同的固定更新循环中计算，直到物理引擎赶上当前时间**



最大允许的时间步长 设置：Edit - Project Settings - Time - Maximum Allowed Timestep

默认设置消耗最大值是 0.333s 

​	超过 = 帧率下降（仅 3FPS）

​	**用尽其他优化方法后有需要再调整这个值**



FixedUpdate 的死亡螺旋：

物理活动较多时，物理引擎处理 FixedUpdate 的时间可能比 列为基准的时间要长

比如：制作游戏的时候指定20ms作为一次FixedUpdate 的更新，但是 FixedUpdate的逻辑执行后，实际上花费了30ms，为了解决 FixedUpdate 落后，会去处理更多的时间步长去跟上，但是 FixedUpdate 内部执行越执行越落后，需要更多的时间步长，物理引擎永远无法摆脱 FixedUpdate 的更新循环，无法允许另一帧进行渲染。

解决：

​	加一个判定允许的最大时间步长：存在允许物理引擎处理每个 FixedUpdate 的最长时间

​	如果当前一批固定更新的处理时间太长 则会停止并放弃进一步的处理 直到下一次渲染更新完成

​	**这样就保证了渲染管线至少将当前状态进行渲染，并保证了用户输入以及游戏逻辑在物理引擎出现异常的时候还能正常执行**



### 物理更新和运行时变化 - 写在 FixedUpdate 中

因为引擎会在给定的时间步长时间内移动激活的 **刚体对象（即 带有 Rigidbody 的 GameObject）**，检测新的碰撞并调用相应对象的碰撞回调，所以需要在 FixedUpdate 中才能执行对物理处理对刚体的修改



为了保持平滑、一致的帧率，需要最小化物理引擎处理在给定时间步长消耗所需的时间，为渲染争取尽可能多的时间



## 静态碰撞器 和 动态碰撞器

动态碰撞器：GameObject 包含 Collider组件 和 Rigidbody组件

作用：对外部的力与Rigidbody的碰撞做出反应。

动态碰撞器 + 动态碰撞器 = 碰撞 - 牛顿运动定律



静态碰撞器：GameObject 包含 Collider组件 但没有包含 Rigidbody组件

作用：无形屏障 - 无穷大质量的墙

静态碰撞器 + 静态碰撞器 = 无法解析碰撞



## 碰撞检测

Rigidbody - Collision Detection 

​	Discrete：离散 - 每个时间步长将对象传送一小段时间，移动后对重叠执行边界立体检查检测碰撞，小对象移动太快会丢失碰撞

​			动态碰撞器 与 动态碰撞器之间的碰撞

​	Continuous：连续 - 从当前时间步长的起始和结束位置插入碰撞器，在时间段之间检测碰撞，更精确，但CPU 开销更高

​			在给定碰撞器 与 静态碰撞器之间启用连续碰撞检测

​	ContinuousDynamic：连续动态，开销最高

​			碰撞器 与 静态碰撞器 + 动态碰撞器 都能进行连续碰撞检测

​			

## 碰撞器类型

3D

​	球体

​	胶囊体

​	立方体

​	网格

​		Convex：凸

​		Concave：凹 - 只能做静态碰撞器或触发器



2D

​	圆

​	方框

​	多边形



碰撞器 碰撞：触发另外一个碰撞器的 OnCollisionEnter、OnCollisionStay、OnCollisionExit



### 触发

IsTrigger 属性：勾选 = 表示为非物理，但进入或离开仍然有物理

勾选 IsTrigger 后，碰撞器 碰撞：触发另外一个 触发器的 OnTriggerEnter、OnTriggerStay、OnTriggerExit



## 碰撞矩阵 - 定制碰撞规则

定义允许哪些对象与哪些对象发生碰撞

Edit - Project Settings - （Physics/Physics2D） - Layer Collision Matrix

通过 Layer 层判断

矩阵表示层与层每个可能的组合，启用表示在碰撞检测期间检查两个层中的碰撞器



## Rigidbody 激活和休眠

休眠：CPU 不会执行更新 FixedUpdate 中的逻辑，除非被外力或碰撞事件唤醒

物理引擎内部有自己确定静止状态的测量值

物体的速度在指定时间内没有超过阈值 = 物理引擎认为不再需要移动 = 启用休眠

修改休眠阈值：Edit - Project Settings - Physics - Sleep Threshold

Profiler - Physics Area ：活动的 Rigidbody 对象总数



移动的 Rigidbody 接近休眠对象会强制执行检查是否发生碰撞，再次唤醒休眠对象



## 射线和对象投射

射线投射：物理引擎的常见特征

​	将射线从一个点投射到另一个点，并用路径中的一个或者多个对象生成碰撞信息



Physics.OverlapSphere：检查空间中固定点的有限距离内获得目标列表 - 模拟爆炸

Physics.SphereCast、Physics.CapsuleCast：在空间中向前投射整个对象 - 模拟光束



## 调试物理

1、

Physics Debugger：从 Scene 窗口中过滤出不同类型的碰撞器

Window - Physics Debugger



2、

OnCollisionEnter、OnTriggerEnter 回调打断点



3、

将用户输入的处理要写在 Update，不能为了让用户输入控制刚体行为就写在 FixedUpdate 中，会导致输入等待或者延迟，因为在物理引擎响应被按下的键之前需要等待 0-20ms（固定的更新时间步长）

Input.GetKeyDown() ：“当前帧” 的按下 才会返回 true，否则为 false



# 物理性能优化



## 场景设置

场景设置降低物理引擎的不稳定性



1、物理物体缩放尽可能 (1,1,1)

​	缩放过大会导致重力移动物体的速度比预期要慢

​	修改重力强度调整世界隐含的缩放：Edit - Project Settings - Physics/Physics2D - Gravity 

​	注意：任何浮点运算在数值接近0时都会更加精确



2、在世界空间的位置接近(0,0,0)

​	会有更好的浮点数精度

​	如：人物固定位置(0,0,0)，移动其他对象模拟旅行造成移动错觉



3、保持质量的相对差异可以让碰撞更真实不会对引擎造成更大压力

​	质量值保持在1.0左右

​	可以指定一个人的质量是 1.0（~130kg），相对差异下汽车质量值是 10.0（~1300kg）

​	碰撞质量产生的动量差会由于冲量而变成突然巨大的速度变化导致不稳定的物理现象和浮点精度的潜在丢失

​	物体下落的空气阻力的逼真行为：为对象自定义drag属性或者禁用 Use Gravity 并在 FixedUpdate 中应用自定义的重力



## 适当使用静态碰撞器

1、实例化

避免在游戏中实例化新的静态碰撞器

在运行时将新对象引入静态碰撞器数据结构，需要重新生成，会导致 CPU 峰值



2、移动、旋转、缩放

移动、旋转、缩放 静态碰撞器 也会触发重新生成，会导致 CPU 峰值



碰撞器不与其他物体发生碰撞下移动： - 如 玩家角色 的物理 Kinematic 碰撞器设置

​	1、附加一个 Rigidbody，先转换成 动态碰撞器

​	2、开启 Kinematic：防止对象对来自对象间碰撞的外部冲击做出反应，会变成类似于静态碰撞器 但 可以通过 Transform组件 或者通过施加 Rigidbody 的力 进行移动

​	因为不会对撞击它的物体做出反应，所以运动时会简单把其他动态碰撞器推开



## 适当使用触发体积

不要在 OnTriggerXXX 中对碰撞做出反应，这时候获取的碰撞的信息（相交信息）会不准确



## 优化碰撞矩阵

碰撞矩阵定义了物理引擎关心的对象碰撞对

忽略其他每一个层对可以最小化物理引擎工作负载，减少每次 FixedUpdate 必须检查的边界体积的数量，以及在应用程序的生命周期中需要处理的碰撞数量

可以延长移动设备电池寿命



## 首选离散碰撞检测

离散碰撞检测消耗低

只有极端情况下使用连续碰撞检测设置



## 修改 FixedUpdate 频率

为离散碰撞检测系统提供更好的捕获碰撞

FixedUpdate 和 物理时间步长处理 是强耦合的

修改 FixedUpdate 检查的频率 = 更改物理引擎 计算和处理 下一个回调的频率 = 更改调用 FixedUpdate 回调 和 协程 的频率

Edit - Project Settings - Time - Fixed Timestep 或者代码中通过 Time.fixedDeltaTime 属性



减小后可以增加频率迫使物理引擎更加高频处理，更容易捕获离散碰撞检测，但会增加 CPU 成本

增加这个值 = 减少频率 = 降低CPU成本，但是 会降低物体移动的最大速度，导致无法使用离散碰撞检测，不再类似现实行为



## 最小化射线发射和边界体积检查

射线投射（CapsuleCast 和 SphereCast ）：

​	持久的投射检查

​	消耗较大，要避免在 Update 回调或者协程中定期使用这些方法，只在脚本代码中的关键事件中调用



尽可能使用简单的触发体积：持续的线、射线或者区域效果碰撞区域（安全激光、持续燃烧的火焰、光束武器），并且对象保持相对静止

无法使用简单触发体积替换的情况下：层遮罩（LayerMask） 最小化每个射线投射的处理量

```c#
[SerializeField] LayerMask _layerMask;
void PerformRaycast(){
    RaycastHit[] hits;
  	//Physics.RaycastAll 重载 传入 LayerMask
    hits = Physics.RaycastAll(transform.position, transform.forward, 100.0f, _layerMask);
    for(int i=0; i < hits.Length; i++)
    {
        //...
    }
}
```



注意：RaycastHit 和 Ray 类 被 Unity引擎 本地内存空间管理，不会导致受垃圾回收器关注的内存分配



## 避免复杂的网格碰撞器

两个对象的复杂性 决定了碰撞的数学工作量

一个对象的静态碰撞器 比 两个对象构成的动态碰撞器 更好处理

立方体图形最简单（8个顶点+12个三角形），但碰撞检测和解决碰撞更消耗（数学计算取决于 面、边、角 或 混合条件是否发生碰撞），最有效的是球形碰撞器



1、组合：使用更简单的基本体组合起来表示复杂图形对象

2、简化：使用更简单的网格替换复杂的网格



## 避免复杂的物理组件

避免使用 TerrainCollider、Cloth、WheelCollider 这些比网格碰撞器更消耗



## 尽可能使物理对象休眠



1、减少刚体

休眠情况下，增加1倍的刚体数量 ≠ 成本只增加一倍

碰撞频率 和 活动物体的总累计时间 是指数形式增加



2、运行时不要修改 Rigidbody 的任何属性

mass、drag、useGravity 会重新唤醒对象



3、自定义重力解决方案中

避免每次 FixedUpdate 中使用重力，这会导致无法休眠

解决方案：检查质量归一化动能（velocity.sqrMagnitude 的值），这个值足够小的时候再手动禁用自定义重力



4、降低场景复杂度，大量刚体之间不要相互接触

否则会出现链式唤醒反应

一瞬间大量对象重新进入物理模拟产生CPU 峰值，如果对象太近会在对象再次休眠之前有大量潜在的碰撞对需要处理



## 修改处理器迭代次数

适用于 复杂的模拟

复杂的模拟：关节、弹簧、其他的刚体连接，会产生相互依赖的交互作用

当物体链的任何一部分的速度产生变化的时候需要用这种多迭代方法计算精确的结果



处理器允许尝试的最大迭代次数：Solver Iteration Count

Edit - Project Settings - Physics - Default Solver Iterations：默认6次迭代

增大这个值 = 防止不稳定的 CharacterJoint 行为，让布娃娃这种基于关节的对象更符合物理规则 = 消耗更多CPU

减小这个值 = 避免过高的计算

修改方式：

1、运行时修改 Physics.defaultSolverIterations：不会影响之前存在的刚体

2、刚体构造后通过 Rigidbody.solverIterations 修改



布娃娃被碰撞吸收太多能量后，出现迭代计算结束之前被处理器强制防止

Edit - Project Settings - Physics - Default Solver Velocity Iterations：适用于布娃娃受到碰撞物理

增大这个值 = 处理器更多机会在基于关节的对象碰撞期间计算合理的速度

修改方式：

1、运行时修改 Physics.defaultSolverVelocityIterations：不会影响之前存在的刚体

2、刚体构造后通过 Rigidbody.solverVelocityIterations 修改



## 优化布娃娃



1、只是用7个碰撞器，减少关节和碰撞器，可以降低消耗成本

GameObject - 3D Object - Ragdoll ：简单的布娃娃生成工具

在给定的对象中创建布娃娃

七个碰撞器：骨盆、胸部、头部、每个肢体

代价：布娃娃没有真实性

将角色关节的 connectedBody 属性重新指定给适当的父关节：手臂碰撞器连接到胸部，腿部碰撞器连接到骨盆



2、避免布娃娃之间的碰撞

允许布娃娃之间的碰撞会使性能指数级增长

解决：简单的使用碰撞矩阵



3、更换、禁用 或者 移除不活跃的布娃娃

不再需要布娃娃的时候可以禁用销毁，或者用更简单的代替品替换（只包含7个关节的更简单版本）

Rigidbody.IsSleeping：布娃娃是否休眠



## 尽量避免使用物理

流星雨：使用高度图进行基本碰撞检测，手动调整 Transform.position 来简化对象移动模拟，从而不需要物理组件

拾取对象：使用 Physics.OverlapSphere，在按下时获取附近的对象从结果中找出最近拾取

