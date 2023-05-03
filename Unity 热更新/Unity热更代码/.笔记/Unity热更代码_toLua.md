# 简单原理 - Unity静态绑定lua

ToLua 框架主要是通过 **静态绑定** 来实现 C# 与 Lua 之间的交互



基本原理：

​		通过建立一个 **Lua 虚拟机来映射 C# 脚本**，然后再通过这个虚拟机来运行 Lua 脚本，Lua 脚本在运行时可以通过虚拟机反过来调用 C# 脚本里注册过的物体，虚拟机变相地实现了一个**解释器的功能**。

![](images\ToLua基本原理.jpg)



https://github.com/topameng/tolua



tolua是**Unity静态绑定lua的一个解决方案**

Wrap.cs文件：C#通过反射生成的包装类，和 toLua C 交互

**Lua 调用 C#：**

​	LuaBinder 调用 Wrap文件 生成的 Register代码，

​	Register 将 C# 通过 luaapi 调用 lua 注册，将 C# 方法注册到 lua 虚拟机中

​	在 toLuaC# 中 OjbectTranslator 将对象包装成 **userdata** 传递给 lua 并设置 元表

​	C# 暴露给 Lua 后 ，Lua 可以在内部访问到 C# 的方法/变量

**C# 调用 Lua：**

​	C# 调用 toLua C#（Wrap文件） 与 toLua C 交互，因为 C 属于 非托管代码，需要使用 DllImport 调用 toLua C 中的接口，通过Marshal等与非托管代码交互



Unity C# <==> **toLua C# <==> toLua C** <==> lua ==> luaSocket、cjson、sqlite3 ...



toLua 的库：

- windows平台叫做`tolua.dll`，

- android叫做`libtolua.so`，

- mac平台叫`tolua.bundle`，

- 而iOS平台由于不允许使用动态库，所以会编译成静态库`libtolua.a`。

  

# 编译tolua

https://blog.csdn.net/linxinfa/article/details/90046840



# C# 调用 Lua 代码 - 虚拟机 

1. 创建虚拟机 

   ```c#
   LuaState state = new LuaState() 
   ```

   

2. 启动虚拟机

   ```c#
    state.Start()
   ```

   

3. 可以开始调用 Lua 代码。

​	

## 执行一段 Lua 代码

- state.DoString(string chunk, string chunkName = "LuaState.cs")：

  比较少用这种方式加载代码，无法避免代码重复加载覆盖等，需调用者自己保证。第二个参数用于调试信息，或者 error 消息，用于提示出错代码所在文件名称。

- state.DoFile(string fileName)：

  加载一个 lua 文件，fileName 需要扩展名，可反复执行，后面的变量会覆盖之前的 DoFile 加载的变量。

  （需要 手动给目标文件添加一个文件搜索位置  state.AddSearchPath(string fullPath)）

- state.Require(string fileName)：

  同 lua require(modname)

  （需要 手动给目标文件添加一个文件搜索位置  state.AddSearchPath(string fullPath)）

  

```c#
LuaState state = new LuaState();
state.Start();
string hello =
    @"                
        print('hello tolua#')                                  
    ";
state.DoString(hello, "HelloWorld.cs");

//如果移动了ToLua目录，需要自己手动，这里只是例子就不做配置了
string fullPath = Application.dataPath + "/ToLua/Examples/02_ScriptsFromFile";
state.AddSearchPath(fullPath);  
state.DoFile("ScriptsFromFile.lua"); 
state.Require("ScriptsFromFile"); 

// 垃圾回收, 对于被自动 gc 的 LuaFunction, LuaTable, 以及委托减掉的 LuaFunction, 延迟删除的 Object 之类
// 等等需要延迟处理的回收, 都在这里自动执行
state.Collect();
// 检查是否堆栈是否平衡，一般放于 update 中，c# 中任何使用 lua 堆栈操作，
// 都需要调用者自己平衡堆栈（参考 LuaFunction 以及 LuaTable 代码）, 
// 当 CheckTop 出现警告时其实早已经离开了堆栈操作范围，这时需自行 review 代码
state.CheckTop();
// 释放LuaState 以及其资源
state.Dispose();
state= null;
```



## 调用Lua 变量/函数

state[string]：通过 LuaState [string] 的形式就可以直接获取到，也可以通过这个表达式来直接赋值，取到的是 object 类型。



## ToLua table 方法

这里完全可以将 table 看一个 LuaState 来进行操作，ToLua 对 table 的数据结构进行了解析，实现了非常多的方法：

- state.GetTable(string)：从 state 中获取一个 table, 可以串式访问比如 state.GetTable("varTable.map.name") 等于 varTable->map->name（table["map.name"] 不支持串式访问，"map.name" 只是一个key，不是table->map->name）。
- table.GetMetaTable()：可以获取当前 table 的 metatable。
- table.ToArray()：获取表中的数组部分。
- table.GetTable(key)：获取 table[key] 值，类似于 lua_gettable。
- table.SetTable(key, value)：等价于 table[k] = v，类似于lua_settable。
- table.RawGet(key)：获取 table[key]值，类似于 lua_rawget。
- table.RawSet(key, value)：等价于 table[k] = v，类似于lua_rawset。



## 调用 Lua 的函数

- state.Invoke：临时调用一个 lua function 并返回一个值，这个操作并不缓存 lua function，适合频率非常低的函数调用，可带参数。
- state.Call：看起来和 Invoke 类似，只是无返回值。
- state.GetFunction(string)： 获取并缓存一个 lua 函数，string 支持串式操作。
- LuaFunction.Call：不需要返回值的函数调用操作。
- LuaFunction.Invoke：有一个返回值的函数调用操作。
- LuaFunction.BeginPCall()：开始函数调用。
- LuaFunction.Push：压入函数调用需要的参数。
- LuaFunction.PCall()：调用 lua 函数。
- LuaFunction.CheckNumber()：提取函数返回值, 并检查返回值为 lua number 类型。
- LuaFunction.EndPCall()：结束 lua 函数调用，清楚函数调用造成的堆栈变化。
- luaFunc.ToDelegate：创建一个委托，后续直接调用委托即可。
- LuaFunction.Dispose()：释放 LuaFunction，递减引用计数，如果引用计数为 0，则从 _R 表删除该函数。



# ToLua 调用 C# 方法/变量 - 方法名绑定

C# 或者其他语言中来调用 Lua ：提前约定特定方法名然后载入脚本



但 C# 是需要提前编译的，解释器来调用 C# 中的对象就是主要的难点了。

uLua ：反射，但是反射的效率非常低，虽然确实可以实现，但问题还是非常明显。

ToLua：**方法名绑定的方式来实现这个映射**的，

	1. 构造一个 Lua 虚拟机，在虚拟机启动后对所需的方法进行绑定，
	1. 在虚拟机运行时可以在 Lua 中调用特定方法，虚拟机变相地实现了一个解释器的功能，
	1. 在 Lua 调用特定方法和对象时，虚拟机会在已绑定的方法中找到对应的 C# 方法和对象进行操作，并且 ToLua 已经自动实现了一些绑定的方法。



# LuaBinder.Bind()

LuaBinder 由 GenLuaBinder() 自动生成，主要分为以下几个部分：

- LuaState.BeginModule/EndModule()：用于绑定命名空间，可以逐层嵌套。
- wrap.Register()：注册相应的 wrap 文件，每个 wrap 文件都是对一个 c# 类的包装，实现了将变量/方法绑定进入虚拟机中。在 lua 中，通过对 wrap 类中的函数调用，间接的对 c# 实例进行操作。
- LuaState.BeginPreLoad/AddPreLoad/EndPreLoad()：预加载相关。



# Wrap 文件 - 反射生成（c#类通过wrap类在lua中进行使用）

每个wrap文件都是对一个c#类的包装，在lua中，通过对wrap类中的函数调用，间接的对c#实例进行操作。



## wrap类文件生成和使用的总体流程

![](images\wrap类文件生成和使用的总体流程.png)



CustomSettings.cs 文件

GenerateClassWrap

GenLuaBinder

LuaBinder.Bind(LuaState L)



## 主要通过分析类的反射信息生成一个wrap文件

![](images\生成一个wrap文件的流程.png)

```c#
//这部分代码由GenRegisterFunction()生成，可以看到，这些代码分为了4部分：
public static void Register(LuaState L)
{
    //1.BeginClass部分，负责类在lua中的初始化部分
    /*
    ①用于创建类和类的元表,如果类的元表的元表（类的元表是承载每个类方法和属性的实体，类的元表的元表就是类的父类）
    ②将类添加到loaded表中。
    ③设置每个类的元表的通用的元方法和属性，__gc,name,ref,__cal,__index,__newindex。
    */
    L.BeginClass(typeof(UnityEngine.GameObject), typeof(UnityEngine.Object));
    //2.RegFunction部分，负责将函数注册到lua中
    //每一个RefFunction做的事都很简单，将每个函数转化为一个指针，然后添加到类的元表中去，与将一个c函数注册到lua中是一样的。
    L.RegFunction("CreatePrimitive", CreatePrimitive);
    L.RegFunction("GetComponent", GetComponent);
    L.RegFunction("GetComponentInChildren", GetComponentInChildren);
    L.RegFunction("GetComponentInParent", GetComponentInParent);
    L.RegFunction("GetComponents", GetComponents);
    L.RegFunction("GetComponentsInChildren", GetComponentsInChildren);
    L.RegFunction("GetComponentsInParent", GetComponentsInParent);
    L.RegFunction("SetActive", SetActive);
    L.RegFunction("CompareTag", CompareTag);
    L.RegFunction("FindGameObjectWithTag", FindGameObjectWithTag);
    L.RegFunction("FindWithTag", FindWithTag);
    L.RegFunction("FindGameObjectsWithTag", FindGameObjectsWithTag);
    L.RegFunction("Find", Find);
    L.RegFunction("AddComponent", AddComponent);
    L.RegFunction("BroadcastMessage", BroadcastMessage);
    L.RegFunction("SendMessageUpwards", SendMessageUpwards);
    L.RegFunction("SendMessage", SendMessage);
    L.RegFunction("New", _CreateUnityEngine_GameObject);
    L.RegFunction("__eq", op_Equality);
    L.RegFunction("__tostring", ToLua.op_ToString);
    //3.RegVar部分，负责将变量和属性注册到lua中
    //每一个变量或属性或被包装成get_xxx,set_xxx函数注册添加到类的元表的gettag，settag表中去，用于调用和获取。
    L.RegVar("transform", get_transform, null);
    L.RegVar("layer", get_layer, set_layer);
    L.RegVar("activeSelf", get_activeSelf, null);
    L.RegVar("activeInHierarchy", get_activeInHierarchy, null);
    L.RegVar("isStatic", get_isStatic, set_isStatic);
    L.RegVar("tag", get_tag, set_tag);
    L.RegVar("scene", get_scene, null);
    L.RegVar("gameObject", get_gameObject, null);
    //4.EndClass部分，负责类结束注册的收尾工作
    //①设置类的元表
    //②把该类加到所在模块代表的表中（如将GameObject加入到UnityEngine表中）
    L.EndClass();
}

```



## 通过Wrap文件注册的函数

### 函数的实体部分

构造函数，this[]，get_xxx，set_xxx的原理都差不多，都是通过反射的信息生成的



这里使用GameObject的GetComponent函数进行说明：

​		通过反射分析GetComponent的重载个数，每个重载的参数个数，类型生成的。

```c#
[MonoPInvokeCallbackAttribute(typeof(LuaCSFunction))]
static int GetComponent(IntPtr L)
{
    try
    {
        //获取栈中参数的个数
        int count = LuaDLL.lua_gettop(L);
        //根据栈中元素的个数和元素的类型判断该使用那一个重载
        if (count == 2 && TypeChecker.CheckTypes<string>(L, 2))
        {
            //将栈底的元素取出来，这个obj在栈中是一个fulluserdata，需要先将这个fulluserdata转化成对应的c#实例，也就是调用这个GetComponent函数的GameObject实例
            UnityEngine.GameObject obj = (UnityEngine.GameObject)ToLua.CheckObject(L, 1, typeof(UnityEngine.GameObject));
            //将栈底的上一个元素取出来，也就是GetComponent(string type)的参数
            string arg0 = ToLua.ToString(L, 2);
            //通过obj，arg0直接第调用GetCompent(string type)函数
            UnityEngine.Component o = obj.GetComponent(arg0);
            //将调用结果压栈
            ToLua.Push(L, o);
            //返回参数的个数
            return 1;
        }
        //另一个GetComponent的重载，跟上一个差不多，就不详细说明了
        else if (count == 2 && TypeChecker.CheckTypes<System.Type>(L, 2))
        {
            UnityEngine.GameObject obj = (UnityEngine.GameObject)ToLua.CheckObject(L, 1, typeof(UnityEngine.GameObject));
            System.Type arg0 = (System.Type)ToLua.ToObject(L, 2);
            UnityEngine.Component o = obj.GetComponent(arg0);
            ToLua.Push(L, o);
            return 1;
        }
        //参数数量或类型不对，没有找到对应的重载，抛出错误
        else
        {
            return LuaDLL.luaL_throw(L, "invalid arguments to method: UnityEngine.GameObject.GetComponent");
        }
    }
    catch (Exception e)
    {
        return LuaDLL.toluaL_exception(L, e);
    }
}
```



### 函数实际的调用过程

```lua
local tempGameObject = UnityEngine.GameObject("temp")
--[[
1.先去tempGameObject的元表GameObject元表中尝试去取GetComponent函数，取到了。
2.调用取到的GetComponent函数，调用时会将tempGameObject,"Transform"作为参数先压栈，然后调用GetComponent函数。

3.接下来就进入GetComponent函数内部进行操作，因为生成了新的ci，所以此时栈中只有tempGameOjbect,"Transfrom"两个元素。
4.根据参数的数量和类型判断需要使用的重载。
5.通过tempGameObject代表的c#实例的索引，在objects表中找到对应的实例。同时取出"Transform"这个参数，准备进行真正的函数调用。
6.执行obj.GetComponent(arg0)，将结果包装成一个fulluserdata后压栈，结束调用。
7.lua中的transfrom变量赋值为这个压栈的fulluserdata。

8.结束。
其中3-7的操作都在c#中进行，也就是wrap文件中的GetComponent函数。
]]
local transform = tempGameObject.GetComponent("Transform")
```



## 通过Wrap文件注册的类

一个类通过Wrap文件注册进lua虚拟机后的样子

![](images\一个类通过Wrap文件注册进lua虚拟机后的样子.png)



GameObject 的所有功能都是通过一个元表实现的

1. 通过这个元表可以调用 GameObjectWrap 文件中的各个函数来实现对 GameObject实例的操作

2. 这个元表对使用者来说是不可见的，因为我们平时只会在代码中调用 GameObject 类，GameObject 实例，并不会直接引用到这个元表，

   

GameObject实例与这个元表的关系：

1. GameObject 类：其实只是一个放在_G表中供人调用的一个充当索引的表，我们通过它来触发GameObject元表的各种元方法，实现对c#类的使用。
2.  GameObject的实例：是一个fulluserdata,内容为一个整数，这个整数代表了这个实例在objects表中的索引（objects是一个用list实现的回收链表，lua中调用的c#类实例都存在这个里面，后面会讲这个objects表），每次在lua中调用一个c#实例的方法时，都会通过这个索引找到这个索引在c#中对应的实例，然后进行操作，最后将操作结果转化为一个fulluserdata（或lua的内建类型，如bool等）压栈，结束调用。



# Lua 调用 C#实例 的函数或变量的过程

```lua
local tempGameObject = UnityEngine.GameObject("temp")
local instanceID = tempGameObject.GetInstanceID()
```

![](images\在lua中调用一个c#实例中的函数或变量的过程.png)



# Lua 中 C#实例 的真正存储位置

1. 一个c#实例在lua中是一个内容为整数索引的fulluserdata
2. 在进行函数调用时，通过这个整数索引查找和调用这个索引代表的实例的函数和变量。

```lua
local tempGameObject = UnityEngine.GameObject("temp")
local transform = tempGameObject.GetComponent("Transform")
```



lua中调用和创建的c#实例实际都是存在c#中的objects表中

lua中的变量只是一个持有该c#实例索引位置的fulluserdata，并没有直接对c#实例进行引用

对c#实例进行函数的调用和变量的修改都是通过元表调用操作wrap文件中的函数进行的

![](images\lua中c#实例的真正存储位置.png)



# Tolua框架：LuaFramework_UGUI



1、生成注册文件： Wrap类

根据 `CustomSettings.cs`脚本中的`_GT(typeof(XXXXX))` 生成Wrap类

会将`Unity`常用的`C#`类生成`Wrap`类（Assets/LuaFramework/ToLua/Source/Generate）并注册到`lua`虚拟机中



即：执行了菜单中的 Lua - Gen Lua Wrap Files

```
GT`就是`Genrate Table`的意思，在`lua`中，类其实就是`table
```



2、  生成 Lua委托 和 LuaBinder文件

即：执行了菜单中的 Lua - Gen Lua Delegates 和 菜单中的 Lua - Gen LuaBinder File



生成的`Wrap`类需要在`LuaBinder`中注册到`lua`虚拟机中，

生成的`lua`委托需要在`DelegateFactory`中注册到`lua`虚拟机中，



Lua / Generate Al

1 根据 CustomSettings 中的 customDelegateList，生成lua委托并在 DelegateFactory 中注册到lua虚拟机中；
2 根据 CustomSettings 中的 customTypeList，生成Wrap类；
3 在 LuaBinder 中生成Wrap类的注册逻辑。



3、指定的Wrap不注册

1. 取消原来工具中的注册，特定的Wrap移动到BaseType中
2. 因为`Wrap`是工具生成的，如果手动修改`Wrap`类，下次重新生成的时候会被覆盖回去，就又会报错
3. 解决办法是把它移到`Assets / LuaFramework / ToLua / BaseType`目录中
4. 并且 在`CustomSettings.cs`中把对应的类的`_GT`调用注释掉，
5. 手动在`LuaState.cs`脚本中注册Wrap类，在`LuaState`的`OpenBaseLibs`函数中添加上`Register`调用，以`BeginModule`和`EndModule`来包裹



## ToLua 框架工作流程（LuaManager 是核心管理）

1、进入游戏后，会初始化管理器，将管理器挂到 GameManager 物体上

2、LuaManager 是核心管理

`LuaManager`是整个`tolua`框架的核心，它的三个核心成员如下：

1. LuaState ：lua虚拟机

2. LuaLoader：lua文件加载器

3. LuaLopper：lua生命周期控制

   

### GameManager

#### 释放资源

![](images\toLua GameManager 释放资源.png)

GameManager启动时，会先检测资源路径（Util.DataPath）中是否有lua文件

	1. 如果没有，则将StreamingAssets目录中的files.txt文件拷贝到资源路径（Util.DataPath）中，其中files.txt记录了StreamingAssets目录中所有lua文件和资源文件的md5。
	1. 遍历files.txt文件，把StreamingAssets目录中的lua文件和资源文件拷贝到资源路径（Util.DataPath）中，这个过程叫做释放资源。



#### 更新资源

![](images\toLua GameManager 更新资源.png)

根据 AppConst.UpdateMode 决定要不要执行更新资源。

1. 如果需要更新，则访问Web服务器地址AppConst.WebUrl，下载最新的files.txt。
2. 遍历最新的files.txt，检查本地文件是否缺少或者MD5是否不相等，
3. 去Web服务器下载lua代码或资源（线程管理器启动独立线程进行下载）。



#### 执行Lua 代码

更新完`lua`代码和资源后会调用`GameManager`的`OnInitialize`，到这里就可以启动`lua`虚拟机执行`lua`代码了。

```c#
// 启动lua虚拟机 
LuaManager.InitStart();
```

执行`lua`代码：

```lua
-- 加载Game.lua脚本
LuaManager.DoFile("Logic/Game");  
-- 执行lua的Game.OnInitOK方法
Util.CallMethod("Game", "OnInitOK"); 
```



#### Lua业务代码的结构

![](images\toLua GameManager lua业务代码结构.png)

预设要挂上 LuaBehaviour,cs 才可以去加载显示





### LuaState lua虚拟机（`LuaManager -> LuaState -> LuaDLL`）

`lua`代码需要经过`lua`解释器进行解释才能执行，`lua`解释器是使用`c`语言写的，它在各个平台下有对应的库文件



库函数的声明在`LuaDLL.cs`中

![](images\LuaState LuaDLL.png)



`LuaState`中封装了很多对`LuaDLL`的调度，比如调用某个`lua`的方法，

![](images\LuaState中 LuaDLL的调度.png)





![](images\LuaState做了什么.png)



### LuaLoader Lua文件加载器

继承`LuaFileUtils`，主要提供`lua`文件的读取、查找功能。

```c#
public bool beZip = false;
protected List<string> searchPaths = new List<string>();
protected Dictionary<string, AssetBundle> zipMap = new Dictionary<string, AssetBundle>();
```

![](images\LuaLoader Lua文件加载器.png)





### LuaLooper：lua生命周期控制

```c#
// LuaDLL.cs

[DllImport(LUADLL, CallingConvention = CallingConvention.Cdecl)]
public static extern int tolua_update(IntPtr L, float deltaTime, float unscaledDelta);

[DllImport(LUADLL, CallingConvention = CallingConvention.Cdecl)]
public static extern int tolua_lateupdate(IntPtr L);

[DllImport(LUADLL, CallingConvention = CallingConvention.Cdecl)]
public static extern int tolua_fixedupdate(IntPtr L, float fixedTime);
```

如果没有`LuaLooper`，则`lua`的协程会无法正常执行

![](images\LuaLooper 生命周期.png)





# 扩展 ToLua 框架



## CS2Lua



CS2Lua：CSharp代码转lua，适用于使用lua实现热更新而又想有一个强类型检查的语言的场合

https://github.com/dreamanlan/Cs2Lua

c#编译器开源工程Rosyln 

在ios平台上运行lua，在android平台上直接将用于转lua的c#工程编译为dll并动态加载，这样在不同平台实现热更新的机制不同，运行效率不同，但开发时只需要进行一次开发



翻译过程：

1、两阶段翻译

![](images\CS2Lua 两阶段翻译.png)

2、翻译过程

![](images\CS2Lua 翻译过程.png)



基本思路：

1、语法制导方式翻译到DSL（可以理解为简化了语法特性的中间语言），再由DSL经由生成器转换为lua。（c#语法、语义直接使用Rosyln工程）

2、C# class/struct -> lua table + metatable

3、inherit/property/event interface implementation -> metatable __index/__newindex

4、lambda/delegate/event -> 函数对象

5、generic -> 为每个generic实例生成一份代码，generic类本身不生成代码

6、interface -> 直接忽略

7、数组、集合 -> lua table

8、表达式 -> lua表达式 + 匿名函数调用

9、c#语句 -> lua语句 + 匿名函数调用



## ScriptBehaviourBinder 绑定 Lua 脚本



用于协助脚本层存储绑定的对象和转发MonoBehaviour事件



proxy：
为了尽可能的稳定TKPlugins层，此模块的绝大部分逻辑转移到Proxy里处理。
proxy实例通过工厂代理(ProxyCreator)创建，由系统初始化的时候在App层指定。
proxy的事件将会尽可能的和MonoBehaviour保持一致



对象绑定:
编辑时，由编辑器插件工具维护bindItems和脚本层“名字-位置(name-index)”的一致性。
运行时,脚本对象先根据自己类型绑定属性的描述找到正确的位置(Index),再通过绑定对象的位置快速获取到指定的对象，必要的时候可以使用名字进行校验



事件转发:
为了不必要的性能损失，MonoBehaviour事件被分作两种：基础事件和Notifier事件
基础事件直接由ScriptBehaivourBinder从Unity系统接受，并转发到proxy层，最终由proxy转发到具体的Lua函数里
Notifier事件：由编辑器插件工具维护需要的Notifier组件列表，运行时通过Notifier接受系统事件，并转发到proxy层











# ToLua 与 C# 交互以及泄漏的整理



```c#
gameobj.transform.position = pos
```

gameobj是GameObject类型，pos是Vector3



**第一步：.transform.position（10步）**

```
1. UnityEngine_GameObjectWrap.get_transform    lua想从gameobj拿到transform，对应gameobj.transform
2. LuaDLL.luanet_rawnetobj         把lua中的gameobj变成c#可以辨认的id
3. ObjectTranslator.TryGetValue    用这个id，从ObjectTranslator中获取c#的gameobject对象
4. gameobject.transform            准备这么多，这里终于真正执行c#获取gameobject.transform了
 
5. ObjectTranslator.AddObject      给transform分配一个id，这个id会在lua中用来代表这个transform，
                                transform要保存到ObjectTranslator供未来查找
6. LuaDLL.luanet_newudata          在lua分配一个userdata，把id存进去，用来表示即将返回给lua的transform
7. LuaDLL.lua_setmetatable         给这个userdata附上metatable，让你可以transform.position这样使用它
8. LuaDLL.lua_pushvalue            返回transform，后面做些收尾
9. LuaDLL.lua_rawseti
10. LuaDLL.lua_remove
```



**第二步：= pos（7步）**

```
1. TransformWrap.set_position              lua想把pos设置到transform.position
2. LuaDLL.luanet_rawnetobj                 把lua中的transform变成c#可以辨认的id
3. ObjectTranslator.TryGetValue            用这个id，从ObjectTranslator中获取c#的transform对象
4. LuaDLL.tolua_getfloat3                  从lua中拿到Vector3的3个float值返回给c#
5. lua_getfield + lua_tonumber 3次         拿xyz的值，退栈
6. lua_pop
7. transform.position = new Vector3(x,y,z) 准备了这么多，终于执行transform.position = pos赋值了
```



泄漏1：

## **1、table 作为 Key**

重复性执行的逻辑中需要一直创建table，而这些新创建的table又在存储table中作为索引key，当这个创建的table不再使用时，存储table还持有新建的table，泄漏就发生了。



解决：将存储table设置为弱引用table，这样lua在GC的时候就不会判定存储table还在持有已经不用的新建table



## **2、C# 持有 Lua对象使用完毕不执行释放接口**

件的使用周期完毕不对luaFunction进行释放，是有可能引起内存泄漏的

lua的匿名回调函数



解决：

1.C#工具使用周期结束执行luaFunction or luaTable的Dispose接口。

2.注册函数尽可能少的使用匿名函数



## **3、数学运算（Vector3/Quaternion）**

不加处理的数学运算，且调用频繁的话，会大大增加内存的增长率，使得GC操作变得相对频繁起来



## Lua 的 GC 算法

Lua采用的是 **Mark-sweep算法：（标记-清除算法）**

 1. 每次 GC 的时候，对所有对象进行一次扫描，如果该对象不存在引用，则被回收，反之则保存。

 2. 版本 <= Lua5.0，Lua的GC是一次性不可被打断的过程，**双色标记算法**(Two color mark)，

    这样系统中对象的非黑即白，要么被引用，要么不被引用

    出现的问题：**在GC的过程中如果新加入对象**，这时候新加入的对象无论怎么设置都会带来问题，

    1. 如果设置为白色，则如果处于回收阶段，则该对象会在没有遍历其关联对象的情况下被回收；
    2. 如果标记为黑色，那么没有被扫描就被标记为不可回收，是不正确的。

 3.  Lua5.1 开始，**分布回收 + 三色增量标记清除算法**

    黑色：被垃圾回收器扫描过，并且这个对象的所有引用已经全扫描过，安全的状态

    白色：从未被垃圾回收器扫描过，如果一直是白色就等待被垃圾回收期回收，初始状态默认都是白色

    （**新增 灰色：代表未完成的工作，暂时安全，至少还有一个引用没被扫描过**）

    1. 每个新创建的对象颜色设置为白色

    2. 初始化阶段。遍历root节点中引用的对象，从白色置为灰色，并且放入到灰色节点列表中

    3. 标记阶段

       while（灰色链表中还有未扫描的元素）

       {

       ​	从中取出一个对象，将其置为黑色

       ​	遍历这个对象关联的其他所有对象

       ​	{

       ​		if（白色）

       ​		{

       ​			标记为灰色，加入到灰色链表中(insert to the head)

       ​		}

       ​	}

       }

    4. 回收阶段

       遍历所有对象

       {

       ​	if （白色）

       ​	{

       ​		没有被引用的对象，执行回收

       ​	}

       ​	else{

       ​		重新塞入到对象链表中，等待下一轮GC

       ​	}

       }











