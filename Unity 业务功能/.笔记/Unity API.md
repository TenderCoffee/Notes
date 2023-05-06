# HideFlags

位掩码，用于控制对象的销毁、保存和在 Inspector 中的可见性



HideFlags.DontSave ：Object对象将在新场景中被保留下来

（1）如果GameObject对象被HideFlags.DontSave标识，则在新scene中GameObject的所有组件将被保留下来，但其子类GameObject对象不会被保留到新scene中。

（2）不可以对GameObject对象的某个组件如Transform进行HideFlags.DontSave标识，否则无效。

（3）即使程序已经退出，被HideFlags.DontSave标识的对象会一直存在于程序中，造成内存泄漏，对HideFlags.DontSave标识的对象在不需要或程序退出时需要使用DestroyImmediate手动销毁。

    //当程序退出时用DestroyImmediate()销毁被HideFlags.DontSave标识的对象
    //否则即使程序已经退出，被HideFlags.DontSave标识的对象依然在Hierarchy面板中
    //即每运行一次程序就会产生多余对象，造成内存泄漏
    void OnApplicationQuit()
    {
    	//...
    }



# Time



## realtimeSinceStartup - 不受timeScale影响（真实）

realtimesincestartup表示的是从程序开始以来的真实时间。不依赖游戏时间。

Time.time，或者Time.deltaTime，又或者使用粒子或者动画系统，它们是依赖于游戏时间的。

当我们加速游戏的时候，那么就会变更。

UI 即使游戏速度再慢，也不能被影响。ngui上有一个就是不受时间速度影响的 itweener 的 ignoreTimeScale 属性。



**timeScale只会影响FixedUpdate的速度**
设置 Time.timeScale ＝ 0；即可让游戏暂停。 其实我们暂停的主要是 人物动画，还有技能特效，比如一个火球打了一半。
