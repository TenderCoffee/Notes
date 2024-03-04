1、如果你们项目把Assembly-CSharp作为AOT程序集，强烈建议关闭热更新程序集的`auto reference`选项。因为Assembly-CSharp是最顶层assembly，开启此选项后会自动引用剩余所有assembly，包括热更新程序集，很容易就出现失误引用热更新程序集导致打包失败的情况。



2、

配置PlayerSettings

- 如果你用的hybridclr包**低于v4.0.0版本**，需要关闭增量式GC(Use Incremental GC) 选项
- `Scripting Backend` 切换为 `IL2CPP`
- `Api Compatability Level` 切换为 `.Net 4.x`(Unity 2019-2020) 或 `.Net Framework`（Unity 2021+）



3、

- 暂时不支持在热更新脚本中定义extern函数，但可以调用AOT中extern函数。
- 支持2022的dots技术，需要小幅修改dots代码以支持动态注册component和system类型（**需要购买商业化版本获得实现细节**），而且无法利用burst加速。如果burst部分在AOT，则仍然原生方式执行；如果burst部分在热更部分，则虽然是Jobs并发执行，但以解释方式执行。
- 不支持`System.Runtime.InteropServices.Marshal`中 `Marshal.StructureToPtr`之类序列化结构的函数，但普通Marshal函数如`Marshal.PtrToStringAnsi`都是能正常工作的。
- 不支持[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.xxx)]。纯粹是时机问题，Unity收集这些函数的时机很早，此时热更新dll还没加载。一个推荐的办法是你使用反射收集这些函数，在合适的时机主动调用它们。
- 不支持对解释代码部分进行C#级别调试，因为没暂时没时间写调试器
- `RequireComponent(typeof(AAA))` 要求AAA必须已经在别处资源中实例化或者AddComponent过，否则Unity无法识别AAA为脚本而忽略处理。



4、

主工程不能直接引用热更新代码

多种方式可以从主工程调用热更新程序集中的代码

https://hybridclr.doc.code-philosophy.com/docs/basic/runhotupdatecodes

方式1 反射来调用热更新代码
方式2 通过反射创造出Delegate后运行
方式3 通过反射创建出对象后，再调用接口函数
方式4 通过动态AddComponent运行脚本代码
**方式5 通过初始化从打包成assetbundle的prefab或者scene还原挂载的热更新脚本 - 推荐**



5、

打包运行

- 运行菜单 `HybridCLR/Generate/All` 进行必要的生成操作。**这一步不可遗漏**!!!
- 将`{proj}/HybridCLRData/HotUpdateDlls/StandaloneWindows64(MacOS下为StandaloneMacXxx)`目录下的HotUpdate.dll复制到`Assets/StreamingAssets/HotUpdate.dll.bytes`，**注意**，要加`.bytes`后缀！！！
- 打开`Build Settings`对话框，点击`Build And Run`，打包并且运行热更新示例工程。











