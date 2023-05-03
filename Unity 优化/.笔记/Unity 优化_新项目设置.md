

# Profiler

**注意：监视 CPU 峰值的时候需要关闭 垂直同步（VSync）**

Edit - Project Settings - Quality



# 物理

详细见 Unity 优化_物理.md



## 最大允许的时间步长

最大允许的时间步长 设置：Edit - Project Settings - Time - Maximum Allowed Timestep

​	**用尽其他优化方法后有需要再调整这个值**



## 碰撞矩阵 - 定制碰撞规则

定义允许哪些对象与哪些对象发生碰撞

Edit - Project Settings - （Physics/Physics2D） - Layer Collision Matrix

尽可能忽略其他每一个层对，可以最小化物理引擎工作负载



## Rigidbody 激活和休眠

修改休眠阈值：Edit - Project Settings - Physics - Sleep Threshold



## 场景设置

修改重力强度调整世界隐含的缩放：Edit - Project Settings - Physics/Physics2D - Gravity 



## 修改 FixedUpdate 频率

修改 FixedUpdate 检查的频率 = 更改物理引擎 计算和处理 下一个回调的频率 = 更改调用 FixedUpdate 回调 和 协程 的频率

Edit - Project Settings - Time - Fixed Timestep 或者代码中通过 Time.fixedDeltaTime 属性



减小后可以增加频率迫使物理引擎更加高频处理，更容易捕获离散碰撞检测，但会增加 CPU 成本

增加这个值 = 减少频率 = 降低CPU成本，但是 会降低物体移动的最大速度，导致无法使用离散碰撞检测，不再类似现实行为



## 修改处理器迭代次数

适用于 复杂的模拟

处理器允许尝试的最大迭代次数：Solver Iteration Count

Edit - Project Settings - Physics - Default Solver Iterations：默认6次迭代

Edit - Project Settings - Physics - Default Solver Velocity Iterations：适用于布娃娃受到碰撞物理



