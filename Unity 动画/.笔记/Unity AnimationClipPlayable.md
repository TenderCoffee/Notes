

**使用 AnimationClipPlayable 按需加载**

**PlayableGraph 相关所有成员都是 struct**，框架所有函数参数传递都必须用 ref 关键字，这样就避免 struct 拷贝构造



# 优化



## 1、禁用 AnimationMixerPlayable.GetInput 

会有CPU 的性能消耗，每次都会触发一次 struct 的拷贝过程

优化：使用 List 下标索引的方式获取 AnimationClipPlayable



## 2、尽可能少的 AnimationClipPlayable 执行 connect 和 discount

connect  到 AnimationMixerPlayable 节点

从 AnimationMixerPlayable 节点 discount

这些都会有性能消耗



## 3、AnimationMixerPlayable.SetInputWeight 中的权重值很重要

会有数量级的性能差距

要 设置为 1  和 0

不要设置为 0.001 这样的数值



**动画不再使用就要及时将 weight 设置为 0，避免性能开销**

SetDuration、SetDone、Pause 无用



## 4、使用 Generic 而不是 Humanoid

Generic 比 Humanoid 性能更好，效率高近50%

非必须不使用 Humanoid

Humanoid  增加了 EvaluateClip 调用次数，增加了开销



## 5、性能上：AnimationClipPlayable > Animator

**DirectorUpdateAnimationBegin 的 PrepareFrame 中**  ，AnimationClipPlayable  的CPU 耗时 比 Animator 少了一半

**Animator中的 state 的数量**影响了每帧中 PrepareFrame 的耗时开销



## 6、AnimationClipPlayable 和 Animator 都是多线程

Animator 的 ProcessGraph() 的调用次数每帧都在变动

这个次数是 Unity 分配给主线程的计算量，会每帧变动



## 7、Unity 是以 Graph 为单位进行多线程的任务分配

当只有一个 Graph 的时候，就会固定分配给主线程调用，无法多线程分配



## 8、隐藏 GameObject

GameObject 隐藏后，AnimationClipPlayable 和 Animator 的相关开销都会停止



## 9、Graph.Stop 停止动作 效率更高

AnimationClipPlayable.Puase 性能开销没有减少

Discount 性能开销减少但动画处于 T-Pose

**Graph.Stop 性能开销减少，并且动画处于 Pause**





