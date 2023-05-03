# 概述

## Editor目录

	1、Unity的特殊目录
	2、Editor目录下的脚本和素材都不会最终打包到游戏包体中 
	3、在Editor目录下哪怕包含 Resources 目录，也不会被打包进游戏包体中
	4、Editor目录就是Unity提供给开发者用来存放编辑器扩展代码的

## API

1、调用系统打开文件夹目录

```c#
UnityEditor.EditorUtility.RevealInFinder("path")
```



2、勾选 MenuItem

```c#
Menu.SetChecked("menupath", true/false)
```



3、MenuItem 快捷键

```c#
[MenuItem("menupath 快捷键码")]
```
快捷键码：
	# - Shift
	& - Alt
	% - Ctrl/Command
	_a-zA-Z - a-zA-Z



4、MenuItem 启用验证方法

```c#
[MenuItem("menupath", validate=true)]
```
启用后方法的返回值返回的true还是false决定了 是否启用这个MenuItem
 一个 MenuItem 对应一个 Validate 验证方法



 5、MenuItem的复用

```c#
UnityEditor.EditorApplication.ExecuteMenuItem("path")
```

静态构造函数调用 - 这里可以用于编辑器编译后 校正 和 初始化操作
1、默认情况下，编辑器每次编译后，再次点击菜单然后回到非菜单区域，才有调用静态构造函数
2、如果需要每次编译后就执行静态构造函数 需要在类上加一个Attribute - [InitializeOnLoad]



6、EditorWindow

```c#
GetWindow<GUILayoutExample>().Show();
```

EditorWindow 是 IMGUI 的载体
在OnGUI中 使用 IMGUI(即时模式的GUI渲染) 的 API进行绘制



7、IMGUI 的渲染 API
	1、GUILayout - 自动布局、支持运行时
	2、GUI - 需要设置坐标和尺寸、支持运行时
	3、EditorGUILayout - 自动布局、仅支持编辑器
	4、EditorGUI - 需要设置坐标和尺寸、仅支持编辑器



# 反射 + Attribute

项目用到的 Assemblies 都放在了项目工程 Library 下

项目工程\Library\ScriptAssemblies



通过 reflector 或者 resharper 可以直接查看对应的反编译后的内容

如：

已知 EditorWindow 有个 m_parent 的 internal 成员变量，可以获取到他的父节点，可以得到所有编辑器内置窗口

而 internal 无法直接访问

这时候可以使用反射的方式

![Snipaste_2022-03-24_16-38-11](images\Snipaste_2022-03-24_16-38-11.png)

```c#

		var editorWindowType = typeof(EditorWindow);
        var parent = editorWindowType.GetField("m_Parent", BindingFlags.Instance | BindingFlags.NonPublic);
        //找到所有继承自 EditorWindow的 非接口子类
        if (!editorWindowType.IsInterface)
        {
            //Linq语法
            //当前的领域 获取所有的程序集
            var assemblies = AppDomain.CurrentDomain.GetAssemblies();
            //获取每一个程序集 过滤一些不需要的程序集 得到所有的类型 然后组成一个大的数组
            var typeArr = assemblies.SelectMany(assembly => assembly.GetTypes()).Where(.Where(
                assembly => assembly.FullName.StartsWith("Assembly") 
                            );
            //找出所属 EditorWindow 的子类 和 实现了EditorExWindowAttribute 这个Attribute 的子类 
            var subEditorWindowClassArr = typeArr.Where(type => type.IsSubclassOf(editorWindowType)).Where(type => type.GetCustomAttribute<EditorExWindowAttribute>() != null);
            foreach (Type type in subEditorWindowClassArr)
            {
                Debug.Log(type.Name.ToString());
                //打开窗口
                //GetWindow(type).Show();
            }
        }

```

可以给自定义窗口添加 Attribute标签，从而搭配反射，可以实现过滤掉一些非指定Attribute 的窗口



# 数据



## ScriptableObject 把数据当成资产（asset）

可以像 MonoBehaviour 在 Inspector 操作数据而不用挂到 GameObject（Prefab也是 GameObject） 上

可以把数据对象当作 asset 进行加载和卸载

![](images\Snipaste_2023-02-02_15-22-12.png)

使用方式

​	1、继承 ScriptableObject

​	2、[CreateAssetMenu] 标记创建 或者 通过 ScriptableObject.CreateInstance 进行创建

提供一些生命周期函数

```c#
[CreateAssetMenu]
public class ScriptableObjectExample : ScriptableObject
{
    public int IntValue;
    public string StringValue;
}
```



## EditorPrefs 编辑器存储

```
编辑器扩展版本的 PlayerPrefs

保存：EditorPrefs.SetString(key, value);

读取：EditorPrefs.GetString(key);

```



# 运行时载体

MonoBehaviour 的 OnGUI 中绘制

编辑器脚本：在Editor文件夹下

运行时脚本不能引用编辑器脚本，但是可以把编辑器脚本放到非Editor文件夹下，改装为运行时脚本。GUILayout 和 GUI 都是支持在运行时使用。

编辑器脚本可以引用运行时脚本



# IMGUI API模式



## IMGUI API模式1 - GUILayout 常用 UI 组件

```
Label - 文本标签
TextField/TextAra/PasswordField - 输入框
Button/RepeatButton - 按钮
	区别 - RepeatButton - 按下和抬起都会触发点击逻辑
Horizontal/Vertical - 方向布局
Box - 自动布局框
ScrollView - 滚动视图
Horizontal/VerticalSlider - 滑动条
Area - GUI区域
Window - 窗口
Toolbar - 工具栏
Toggle - 打开/关闭的开关按钮
Space/FlexibleSpace - 空白
Width/Height/MinWidth/MinHeight/MaxWidth/MaxHeight - 宽高控制
SelectionGrid - 选择网格
...
```



影响GUILayout 渲染的API

```
1、GUI.enabled - 是否可交互
2、GUIUtility.RotateAroundPivot - 根据一个点 旋转接下来的 GUI 组件
3、GUIUtility.ScaleAroundPivot - 根据一个点 缩放接下来的 GUI 组件
4、GUI.color - 修改颜色
```



## IMGUI API模式2 - GUI 常用 UI 组件

**无法自动布局，相比较 GUILayout 需要传入 Rect，组件之间位置设置错误会相互叠加** 

```
Label：文本标签
TextField/TextArea/PasswordField：文本输入框
Button/RepeatButton：按钮
Toggle：打开/关闭的开关按钮
Toolbar：工具栏
Box：框
ScrollView：滚动视图
VerticalSlider/HorizontalSlider
Group：组
Window：窗口
...
```



## IMGUI API模式3 - EditorGUI 常用 UI 组件

https://docs.unity3d.com/cn/current/ScriptReference/EditorGUI.html

这些方法的运行方式与常规 GUI 函数十分相似，并且在 [EditorGUILayout](https://docs.unity3d.com/cn/current/ScriptReference/EditorGUILayout.html) 中也有匹配实现

**需要指定 Rect**

**EditorGUI 针对编辑器制定的 API，有更多自己的东西（ColorField、BoundField）**

Unity 界面本身使用的也是 EditorGUI

```
EditorGUI.BeginDisabledGroup + EditorGUI.EndDisabledGroup
EditorGUI.LabelField
EditorGUI.TextField
EditorGUI.TextArea
EditorGUI.PasswordField
EditorGUI.DropdownButton
EditorGUI.Toggle
EditorGUI.ToggleLeft
EditorGUI.HelpBox
EditorGUI.ColorField - 颜色设置面板
EditorGUI.BoundField
EditorGUI.CurveField - 动画曲线面板
EditorGUI.DelayedDoubleField、DelayedFloatField、DelayedIntField、DelayedTextField - 延迟 - 鼠标焦点放到别的地方后，才会真正生效改变
EditorGUI.EnumFlagsField - 枚举多选
EditorGUI.EnumPopup - 枚举单选
EditorGUI.Foldout - 折叠
EditorGUI.GradientField - 颜色渐变
EditorGUI.ObjectField - 对象字段
...
```

如果没有合适的UI组件，就用 GUI 的 UI 组件来替代，如 GUI.Button



## IMGUI API模式4 - EditorGUILayout 常用 UI 组件

https://docs.unity3d.com/cn/current/ScriptReference/EditorGUILayout.html

EditorGUI 的自动布局版本

```
BeginBuildTargetSelectionGrouping - 开始构建目标组并返回所选 BuildTargetGroup
BeginFadeGroup - 开始一个可隐藏/显示的组，并且过渡将生成动画
BeginFoldoutHeaderGroup - 创建一个左侧带有折叠箭头的标签
BeginToggleGroup - 开始一个垂直组，带有可一次性启用或禁用所有控件的开关
...
```



# IMGUI:GUIContent 和 GUIStyle

GUIContent  定义渲染内容

​	可以是文本

​	可以是贴图

​	可以是复合类型（文本 + 贴图）



GUIStyle 定义渲染方式

​	可以直接用字符串赋值

​	字体、颜色、大小、对齐、背景 等等

​	https://docs.unity3d.com/cn/current/ScriptReference/GUIStyle.html



​	默认 GUIStyle：

​		box ： 带边框的盒子

```c#
private GUIStyle mBoxStyle = "box"
```

​		

​		label：字

```c#
private Lazy<GUIStyle> mFontStyle = new Lazy<GUIStyle>(()=>{
  	var retStyle = new GUIStyle("label");
  	//字体
  	retStyle.fontSize = 30;
  	retStyle.fontStyle = FontStyle.BoldAndItalic;
  	//设置颜色
  	retStyle.normal.textColor = Color.green;
  	retStyle.hover.textColor = Color.blue;
  	retStyle.focused.textColor = Color.red;
  	retStyle.active.textColor = Color.cyan;
  	retStyle.normal.background = Texture2D.whiteTexture;
  	return retStyle;
});

GUILayout.Label("Style font", mFontStyle.Value);
```

​	**注意：如果没有 new 的情况下设置了 字体，哪怕后面改了回来，原来的字体也不会发生变化，需要重启一遍Unity 才会恢复**

​		

​		button：按钮

```c#
private Lazy<GUIStyle> mButtonStyle = new Lazy<GUIStyle>(()=>{
  	//默认 - 从皮肤上获取按钮
  	var retStyle = new GUIStyle(GUI.skin.button); //new GUIStyle("button");
  
  	//字体
  	retStyle.fontSize = 30;
  	retStyle.fontStyle = FontStyle.BoldAndItalic;

  	//设置颜色
  	retStyle.normal.textColor = Color.green;
  	retStyle.hover.textColor = Color.blue;
  	retStyle.focused.textColor = Color.red;
  	retStyle.active.textColor = Color.cyan;
  	retStyle.normal.background = Texture2D.whiteTexture;
  	return retStyle;

});

GUILayout.Button("Button font", mButtonStyle.Value);
```





# 进阶IMGUI 组件



## ReorderableList 可排序列表

Editor 文件夹下

UnityEditorInternal 命名空间下

```c#
private ReorderableList mList;

private List<Vector2> mData = new List<Vector2>();

private void OnGUI()
{
    mList.DoLayoutList();
}

private void OnEnable()
{
	mList = new ReorderableList(mData, typeof(Vector2));
    mList.elementHeight = 30;
    mList.drawHeaderCallback += DrawHeader;
    mList.drawNoneElementCallback += DrawNoneElement;
    mList.drawElementCallback += DrawElement;
    mList.drawElementBackgroundCallback += DrawElementBG;
}

private void DrawElementBG(Rect rect, int index, bool isactive, bool isfocused)
{
    GUI.DrawTexture(rect, Texture2D.whiteTexture);
}

private void DrawElement(Rect rect, int index, bool isactive, bool isfocused)
{
    mData[index] = EditorGUI.Vector2Field(rect, "", mData[index]);
}

private void DrawNoneElement(Rect rect)
{
    
}

private void DrawHeader(Rect rect)
{
    GUI.Box(rect, "header");
}


```



![](images\Snipaste_2023-02-02_18-34-26.png)





## PopupWindow 弹窗组件

1、新建类 继承 PopupWindowContent，重写 OnGUI 方法

2、GetWindowSize 得到弹出窗口的固定宽高

**3、GUILayoutUtility.GetLastRect() 得到前面绘制控件的最后的 Rect**

4、PopupWindow.Show(mButtonRect, new PopupWindowContentExample()); 在指定的位置显示 PopupWindowContent 弹窗内容

```c#
public class PopupWindowContentExample : PopupWindowContent
{
    private bool mToggle1 = true;
    private bool mToggle2 = true;
    private bool mToggle3 = true;
    
    public override Vector2 GetWindowSize()
    {
		return new Vector2(200, 150);
	}
    
    public override void OnGUI(Rect rect)
    {
        GUILayout.Label("Popup Options Example", EditorStyles.boldLabel);
        mToggle1 = EditorGUILayout.Toggle("Toggle 1", mToggle1);
        mToggle2 = EditorGUILayout.Toggle("Toggle 2", mToggle2);
        mToggle3 = EditorGUILayout.Toggle("Toggle 3", mToggle3);
    }
    
    public override void OnOpen()
    {
        Debug.Log("OnOpen");
    }
    
    public override void OnClose()
    {
        Debug.Log("OnClose");
	}
    
}

public class PopupWindowExample : EditorWindow
{
    [MenuItem("Open")]
    static void Open()
    {
        GetWindow<PopupWindowExample>().Show();
	}
    
    private Rect mButtonRect;
    
    private void OnGUI()
    {
        if(GUILayout.Button("Popup Options", GUILayout.Width(200)))
        {
            //在指定的区域位置 显示弹窗
            PopupWindow.Show(mButtonRect, new PopupWindowContentExample());
        }
        
        if(Event.current.type == EventType.Repaint)
            mButtonRect = GUILayoutUtility.GetLastRect();
        
    }
    
}


```



![](images\Snipaste_2023-02-03_16-58-17.png)



## AdvancedDropdown 高级下拉菜单

Editor文件夹下

1、创建类 继承 AdvancedDropdown

2、重写 BuildRoot

**3、GUILayoutUtility.GetRect：获取一个接下来要绘制的方块**

```c#
public class WeekdaysDropdown : AdvancedDropdown
{
    public WeekdaysDropdown(AdvancedDropdownState state) : base(state)
    {
	}
    
    public override AdvancedDropdownItem BuildRoot()
    {
        var root = new AdvancedDropdownItem("Weekdays");
        var firstHalf = new AdvancedDropdownItem("First half");
        var secondHalf = new AdvancedDropdownItem("Second half");
        var weekend = new AdvancedDropdownItem("Weekend");
        
        firstHalf.AddChild(new AdvancecdDropdownItem("Monday"));
        firstHalf.AddChild(new AdvancecdDropdownItem("Tuesday"));
        
        secondHalf.AddChild(new AdvancecdDropdownItem("Wednesday"));
        secondHalf.AddChild(new AdvancecdDropdownItem("Thursday"));
        
        weekend.AddChild(new AdvancecdDropdownItem("Friday"));
        weekend.AddChild(new AdvancecdDropdownItem("Saturday"));
        weekend.AddChild(new AdvancecdDropdownItem("Sunday"));
        
        root.AddChild(firstHalf);
        root.AddChild(secondHalf);
        root.AddChild(weekend);
        
        return root;
    }
}

public class AdvancedDropdownExample : EditorWindow
{
    [MenuItem("Open")]
    static void Open()
    {
        CreateInstance<AdvancedDropdownExample>().Show();
    }
    
    private void OnGUI()
    {
        //GetRect
		var rect = GUILayoutUtility.GetRect(new GUIContent("Show"), EditorStyles.toolbarButton);
        if(GUI.Button(rect, new GUIContent("Show"), EditorStyles.toolbarButton))
        {
            var dropdown = new WeekdaysDropdown(new AdvancedDropdownState());
            dropdown.Show(rect);
        }
    }
    
}


```

![](images\Snipaste_2023-02-03_16-25-09.png)



## TreeView 树视图组件

Editor 文件夹下

1、创建新类 继承 TreeView

2、TreeView.OnGUI 渲染

3、构造要加个 Reload 方法

```c#
public class TreeViewExample : EditorWindow
{
    [MenuItem(Open)]
    static void Open()
    {
        GetWindow<TreeViewExample>().Show();
	}
    
    [SerializeField]
    private TreeViewState mTreeViewState;
    
    private SimpleTreeView mSimpleTreeView;
    private SearchField mSearchField;
    
    private void OnEnable()
    {
        if(mTreeViewState == null)
        {
			mTreeViewState = new TreeViewState();
        }
        
        mTreeViewState = new SimpleTreeView(mTreeViewState);
        mSearchField = new SearchField;
        mSearchField.downOrUpArrowKeyPressed += mSimpleTreeView.SetFocusAndEnsureSelectedItem;
        
	}
    
	private void OnGUI()
    {
        GUILayout.BeginHorizontal(EditorStyles.toolbar);
        GUILayout.Space(100);
        GUILayout.FlexibleSpace();
        mSimpleTreeView.searchString = mSearchField.OnToolbarGUI(mSimpleTreeView.searchString);
        GUILayout.EndHorizontal();
        
		var rect = GUILayoutUtility.GetRect(0, 10000, 0, 10000);
        mSimpleTreeView.OnGUI(rect);
    }
    
    
    public class SimpleTreeView : TreeView
    {
        public SimpleTreeView(TreeViewState state) : base(state)
        {
            //创建之后需要重新加载 才会执行BuildRoot操作
            Relaod();
        }
        
        public SimpleTreeView(TreeViewState state, MultiColumnHeader multiColumnHeader) : base(state, multiColumnHeader)
        {
            
        }
        
        protected override TreeViewItem BuildRoot()
        {
            //创建根节点
            //id，深度值，显示名
            var root = new TreeViewItem(0, -1, "Root");
            var allItems = new List<TreeViewItem>()
            {
                new TreeViewItem(1, 0, "Animals"),
                new TreeViewItem(2, 1, "Mammals"),
                new TreeViewItem(3, 2, "Triger"),
                new TreeViewItem(4, 2, "Elephant"),
                new TreeViewItem(5, 2, "Okapi"),
			};
            
            //根据深度去创建
            SetupParentsAndChildrenFromDepths(root， allItems);
            
            return root;
		}

    }
    
}
```

![](images\Snipaste_2023-02-03_17-16-58.png)



# Event

监听拖拽

​	Event.current.DragPerform - 按下鼠标开始

```c#
//标记拖拽开始
DragAndDrop.AcceptDrag();
```

​	Event.current.DragUpdated

Rect.Contains - 检测鼠标是否在指定区域内 

```c#
	if(rect.Contains(Event.current.mousePosition)){
      	//鼠标变拖拽模式 + 号
		DragAndDrop.visualMode = DragAndDropVisualMode.Generic;
      	//标记占用
      	Event.current.Use()
	}
```

​	Event.current.DragExited - 脱离拖拽框

```c#
//获取路径
DragAndDrop.paths

//拖拽的对象的引用
DragAndDrop.objectReferences
```



Event.current：检测鼠标输入

EditorWindow 下的 OnGUI 中绘制

```c#
private  void OnGUI()
{
    var e = Event.current;
    
    EditorGUILayout.Toggle("alt", e.alt);
    EditorGUILayout.IntField("button", e.button);
    EditorGUILayout.Toggle("capsLock", e.capsLock);
    EditorGUILayout.IntField(nameof(e.clickCount), e.clickCount);
    EditorGUILayout.Toggle(nameof(e.command), e.command);
    EditorGUILayout.LabelField(nameof(e.commandName), e.commandName);
    EditorGUILayout.Toggle("control", e.control);
    EditorGUILayout.Vector2Field("delta", e.delta);
    EditorGUILayout.IntField("displayIndex", e.displayIndex);
    EditorGUILayout.Toggle("functionKey", e.functionKey);
    EditorGUILayout.Toggle("isKey", e.isKey);
    EditorGUILayout.Toggle("isMouse", e.isMouse);
    EditorGUILayout.Toggle("isScrollWheel", e.isScrollWheel);
    EditorGUILayout.EnumPopup("keyCode", e.keyCode);
    
    EditorGUILayout.EnumPopup("modifiers", e.modifiers);
    EditorGUILayout.Vector2Field("mousePosition", e.mousePosition);
    EditorGUILayout.Toggle("numeric", e.numeric);
    EditorGUILayout.EnumPopup("pointerType", e.pointerType);
    EditorGUILayout.FloatField("pressure", e.pressure);
    EditorGUILayout.EnumPopup("rawType", e.rawType);
    EditorGUILayout.Toggle("shift", e.shift);
    EditorGUILayout.EnumPopup("type", e.type);
    
    //每次都重新绘制 数据实时显示
    Repaint();
}
```



![](images\Snipaste_2023-02-02_18-11-16.png)







# Attribute

## 常用内置 Attribute

https://docs.unity3d.com/cn/current/ScriptReference/HideInInspector.html

```
SerializeField - 序列化，会再 Inspector 中显示
HideInspector - 不在 Inspector 中显示
Tooltip
Header - 标题 [Header("xx")]
Space - 空格 [Space(20)]
Multiline - 多行 [Multiline(5)] - 超出不会有滚动条
TextArea - 文本域 - 超出会有滚动条
Range - 滑动条 - [Range(5,10)]
HelpURLAttribute - [HelpURL("http://www.baidu.com")] - 问号打开的网址
```



## 自定义 Attribute 渲染器

```
1、Attribute 继承 PropertyAttribute
2、在 Editor文件夹下写渲染器，渲染器继承 PropertyDrawer
3、使用 CustomPropertyDrawerAttribute 标记
4、在 OnGUI 渲染
5、使用 GetPropertyHeight 重新计算高度
```



```c#
// Unity提供的 PropertyAttribute
public class MyAttribute : PropertyAttribute
{
  	
}


//Editor文件夹下：新建渲染器脚本
//渲染器脚本标签
[CustomPropertyDrawer(typeof(MyAttribute))]
public class MyAttributeDrawer : PropertyDrawer
{
	//重写 OnGUI
	public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
    {
    	//property.intValue = 得到属性绑定的值
    	GUI.Label(new Rect(position.position, new Vector2(position.Width, 20)), "使用了MyAttribute：" + property.intValue);
    	//base.OnGUI(position, property, label);
    	EditorGUI.PropertyField(new Rect(position.x, position.y,position.width, position.height)， property, label);
    }
		
	//修改原来的高度
	public override float GetPropertyHeight(SerializedProperty property, GUIContent label)
    {
    	return base.GetPropertyHeight(property, label) + 20;
    }
}

  
//使用属性 不需要加上 Attribute 关键字
[My]
public int test;
  
```



## 编辑器事件 Attribute

Editor 文件夹下的脚本

```
静态构造方法（第一次被获取时生效）
InitializeOnload - 加载脚本时初始化（编辑器重新编译代码 或者 运行场景） - 标记类，触发静态构造函数
InitializeOnLoadMethod - 用于标记静态方法（编辑器重新编译代码 或者 运行场景） - 标记静态方法

DidReloadScripts - 脚本编译完成后生效
PostProcessScene - 加载场景时，系统会在进入运行时
PostProcessBuild - Build 之后会收到一个回调
OnOpenAsset - 打开某个资源时获得回调
```





# Unity 内置 Icon

```c#
if (GUI.Button(rect, EditorGUIUtility.IconContent("Folder Icon")))
{
}
```



# Inspector 检视面板自定义



## ContextMenu - 上下文菜单中添加命令

```c#
[ContextMenu("Hello Context")]
void HelloContext()
{
  	Debug.Log("Hello Context");
}

```

将脚本挂载在游戏对象上，右键脚本组件，执行 ContextMenu 的内容

 

## ContextMenuItemAttribute - 将上下文菜单添加到调用命名方法的字段

将脚本挂载在游戏对象上，右键脚本组件上的 mContextMenuItemValue 变量，可以看到菜单“Print Value”，执行 HelloContextMenuItem 的内容

``` C#
[ContextMenuItem("Print Value", "HelloContextMenuItem")]
[SerializeField]
private string mContextMenuItemValue;

void HelloContextMenuItem()
{
  	Debug.Log(mContextMenuItemValue);
}
```



示例：给所有的 MonoBehaviour 脚本

```c#
#if UNITY_EDITOR

	//给所有的 MonoBehaviour 组件类型 绑定指定 FindScript菜单
	private const string mFindScriptPath = "CONTEXT/MonoBehaviour/FindScript";
	
	[UnityEditor.MenuItem(mFindScriptPath)]
	static void FindScript(UnityEditor.MenuCommand command)
    {
      	UnityEditor.Selection.activeObject = UnityEditor.MonoScript.FromMonoBehaviour(command.context as MonoBehaviour);
	}

	//================================

	//给所有的 Camera 组件类型 绑定指定 LogSelf菜单
	private const string mCameraScriptPath = "CONTEXT/Camera/LogSelf";
	
	[UnityEditor.MenuItem(mCameraScriptPath)]
	static void LogSelf(UnityEditor.MenuCommand command)
    {
      	Debug.Log(command.context);
	}
	
#endif
```



## Editor - 复写 场景游戏对象 的 Inspector 上的内容

```
创建 Editor文件夹下的 Inspector 脚本文件，继承 Editor 类
1、使用 CustomEditorAttribute 标记类 => [CustomEditor(typeof(xxx))]
2、类继承 UnityEditor.Editor
3、在绘制的时候用 OnInspectorGUI
4、获取复写的源脚本的内容 => target as 类型

其他方式：
可以用 Editor.CreateEditor 等 API
```

```c#
public class CustomEditorExample
{
  	public int HP;
  	public string RoleName;
  	public GameObject OtherObj;
}

[CustomEditor(typeof(CustomEditorExample))]
public class CustomEditorExampleInspector : Editor
{
	CustomEditorExample mTarget
	{
      get{
		return target as CustomEditorExample;
      }
	}
	
	public override void OnInspectorGUI()
	{
		//base.OnInspectorGUI();
		
		var rect = GUILayoutUtility.GetRect(18, 18, "TextField");
		EditorGUI.ProgressBar(rect, mTarget.HP, "HP");
		
		EditorGUILayout.BeginHorizontal("box");
		//这里的输入框太短了 因此指定宽度
		EditorGUILayout.LabelField("角色名", GUILayout.Width(100));
		mTarget.RoleName = EditorGUILayout.TextArea(mTarget.RoleName);
		EditorGUILayout.EndHorizontal();
		
		//在这里面不需要注意排版的问题  EditorGUILayout 和 EditorGUI 可以一起使用
		//var serializedObject = new SerializedObject(target);
		//Editor 下有直接提供 serializedObject 不需要new 直接取即可
		var otherObjProperty = serializedObject.FindProperty("OtherObj");
		EditorGUILayout.ObjectField(otherObjProperty);
		//提交修改属性 - 调用修改 这样的话拖拽对象才有更新到属性里面
		serializedObject.ApplyModifiedProperties();
	}

}
```



## ObjectPreview 预览渲染

```
1、创建脚本放在 Editor 目录下，继承 ObjectPreview
2、HasPreviewGUI 返回 true
3、在 OnPreviewGUI 渲染
4、使用 CustomPreview Attribute 标记
```

```c#

//对指定类型 进行 预览渲染
[CustomPreview(typeof(GameObject))]
public class ObjectPreviewExample : ObjectPreview
{
	public override bool HasPreviewGUI()
	{
		return true;
	}
	
	public override void OnPreviewGUI(Rect r, GUIStyle background)
    {
    	//渲染
    	GUI.Label(r, target.name + " 预览了");
    }
}

```



# Hierarchy 场景结构自定义

```
EditorApplication.hierarchyChanged：发生变化时
EditorApplication.hierarchyWindowItemOnGUI：绘制
EditorApplication.RepaintHierarchyWindow：重绘，如果不执行重绘，就需要将鼠标焦点放到窗体后才会有刷新
```

在 Editor文件夹下新建脚本

```c#

[InitializeOnLoad]
public class HierarchyExample
{
  	static HierarchyExample()
    {
      	RegisterHierarchy();
	}
  
  	static void RegisterHierarchy()
    {
      	EditorApplication.hierarchyWindowItemOnGUI += OnHierarchyGUI;
      	EditorApplication.hierarchyChanged += OnHierarchyChanged;
    }
  
  	static void UnRegisterHierarchy()
    {
      	EditorApplication.hierarchyWindowItemOnGUI -= OnHierarchyGUI;
      	EditorApplication.hierarchyChanged -= OnHierarchyChanged;
    }
  	
  	private static void OnHierarchyChanged()
    {
      	//修改游戏对象的时候触发
      	EditorApplication.RepaintHierarchyWindow();
	}
  
  	private static void OnHierarchyGUI(int instanceid, Rect selectionrect)
    {
      	//通过实例ID获取到对应的对象
      	var obj = EditorUtility.InstanceIDToObject(instanceid) as GameObject;
      	
      	if(obj){
        	var tagLabelRect = selectionrect;
            tagLabelRect.x += 100;
            GUI.Label(tagLabelRect, obj?.tag);

            var layerLabelRect = tagLabelRect;
            layerLabelRect.x += 100;
            GUI.Label(layerLabelRect, LayerMask.LayerToName(obj.layer));
      	}
      	
	}
  
}
```



# Project 项目结构自定义

```
EditorApplication.projectChanged：发生变化时
EditorApplication.projectWindowItemOnGUI：绘制
EditorApplication.RepaintProjectWindow：重绘

ProjectWindowUtil.CreateFolder()：创建目录等待玩家输入文件夹名称
```

在 Editor文件夹下新建脚本

```c#
[InitializeOnLoad]
public class ProjectExample
{
  	static ProjectExample()
    {
      	RegisterProject();
	}
  
  	static void RegisterProject()
    {
      	EditorApplication.projectWindowItemOnGUI += OnProjectGUI;
      	EditorApplication.projectChanged += OnProjectChanged;
    }
  
  	static void UnRegisterProject()
    {
      	EditorApplication.projectWindowItemOnGUI -= OnProjectGUI;
      	EditorApplication.projectChanged -= OnProjectChanged;
    }
  	
  	private static void OnProjectChanged()
    {
      	//修改项目的时候触发
      	EditorApplication.RepaintProjectWindow();
	}
  
  	private static void OnProjectGUI(string guid, Rect selectionrect)
    {
      	try{

          	//通过GUID获取到对应的项目资源
          	var assetPath = AssetDatabase.GUIDToAssetPath(guid);
          	var files = Directory.GetFiles(assetPath);

          	var countLabelRect = selectionrect;
          	countLabelRect.x += 100;
          	GUI.Label(countLabelRect, files.Length.ToString());
		}
        catch(Exception e)
        {

        }
      	
	}
  
}
```



# Project - AssetDatabase 资产数据库

创建目录、创建材质、查询是否存在、GUID/路径转换、加载等

https://docs.unity3d.com/cn/current/ScriptReference/AssetDatabase.html

创建 Editor 文件夹

```c#
AssetDatabase.CreateFolder - 创建文件夹  返回 guid
string.IsNullOrEmpty(AssetDatabase.GUIDToAssetPath(guid)) - 判断文件/文件夹 是否存在

//创建材质到 NewMaterialPath 指定路径下
Material material = new Material(Shader.Find("Specular"));
AssetDatabase.CreateAsset(material, NewMaterialPath);

//判断资源是否存在/是否创建成功 
AssetDatabase.Contains(material)
 
//路径转换为GUID - 用于后面操作资源
var guid = AssetDatabase.AssetPathToGUID(NewMaterialPath);

//加载材质
AssetDatabase.LoadAssetAtPath<Material>(NewMaterialPath);

//重命名
AssetDatabase.RenameAsset(NewMaterialPath, "NewName");

//移动
AssetDatabase.MoveAsset(NewMaterialPath, "Assets/move.mat");

//复制
AssetDatabase.CopyAsset(NewMaterialPath, "Assets/copy.mat");

//删除
AssetDatabase.DeleteAsset(NewMaterialPath); //- 通过路径直接删除（不放回收站）
AssetDatabase.MoveAssetToTrash(NewMaterialPath);// - 删除 放到回收站

//刷新
AssetDatabase.Refresh();
```



# AssetProcessor 资源处理回调

## AssetModificationProcessor - 资产操作的各种回调、判断、操作限制等

使用方式：继承 AssetModificationProcessor

API文档：

https://docs.unity3d.com/cn/current/ScriptReference/AssetModificationProcessor.html

```c#

public class AssetModificationProcessorExample : UnityEditor.AssetModificationProcessor
{
	private static void OnWillCreateAsset(string assetName)
    {
        Debug.Log($"OnWillCreateAsset{assetName}");
    }
	
	//...
}


```



## AssetPostprocessor 资源导入前后回调

在Project窗口中导入资源（贴图、模型...）前后可以获得回调

如：贴图

​	OnPreprocessTexture：导入贴图前处理贴图

​	OnPostprocessTexture：导入（处理）贴图之后回调

API文档：

https://docs.unity3d.com/cn/current/ScriptReference/AssetPostprocessor.html



```c#

//继承 AssetPostprocessor
public class AssetPostprocessorExample : AssetPostprocessor
{
	private void OnPreprocessTexture()
    {
    	//导入资源前
    }
    
    private void OnPostprocessTexture(Texture2D texture)
    {
    	//导入资源之后
    }
}

```



# Gizmos 辅助绘制

可以在场景中绘制当前选择物体的一些辅助图

在MonoBehaviour 的 OnDrawGizmos 方法中渲染

API 文档：

https://docs.unity3d.com/ScriptReference/Gizmos.html

```c#
//绘制颜色
Gizmos.color
Gizmos.DrawCube
Gizmos.DrawWireCube

//绘制贴图
Gizmos.DrawGUITexture
```

```c#

//绑定在指定的游戏对象上
public class GizmosExample : MonoBehaviour
{

  //绘制 Gizmos
  private void OnDrawGizmos()
  {
  }

  //选中这个 MonoBehaviour 的时候显示
  private void OnDrawGizmosSelected()
  {

  }
  
	//用静态的方式写在其他地方
  #if UNITY_EDITOR
  [UnityEditor.DrawGizmo(UnityEditor.GizmoType.Active | UnityEditor.GizmoType.Selected)]
  private static void MyCustomOnDrawGizmos(GizmosExample target, UnityEditor.GizmoType gizmoType)
  {

  }

  #endif
}
  

```



# Handles 自定义 3D  GUI 空间 和 绘制

1、写一个 MonoBehaviour 脚本文件

2、在 Editor 文件夹下创建脚本对应的 自定义编辑器脚本文件

3、在 OnSceneGUI 中绘制

4、target 可以获取得到

```c#
public class HandlesExample : MonoBehaviour
{
  	public float Radius = 0.7f;
}

[CustomEditor(typeof(HandlesExample))]
public class HandlesExampleEditor : Editor
{
	private void OnSceneGUI()
    {
    	var t = target as HandlesExample;
    	//Handles.color - 修改颜色
    	//Handles.DrawWireDisc - 绘制一个环
    	//Handles.Label - 绘制一个Label
    	
    	//Handles.BeginGUI - 绘制 GUILayout、GUI 内容
    	//Handles.EndGUI
    	//Handles.DrawAAPolyLine
    	
    }

}
```



# EditorTool 工具栏QWER扩展

![Snipaste_2023-01-18_17-05-53.png](images\Snipaste_2023-01-18_17-05-53.png)

```
1、[EditorTool("Name")]
2、继承EditorTool
3、在 OnToolGUI 里用 Handles 等 API 绘制
```

```c#

[EditorTool("Hello EditorTool")]
public class HelloEditorTest : EditorTool
{
	public override void OnToolGUI(EditorWindow window)
    {
    	window.ShowNotification(new GUIContent("Hello EditorTool"));
    	
    	Handles.BeginGUI();
    	if(GUILayout.Button("Hello EditorTool"))
        {
        	Debug.Log("Hello EditorTool");
        }
    	Handles.EndGUI();
    }

}


//只显示右移动的句柄
[EditorTool("Move Right")]
public class MoveRightTest : EditorTool
{
	public override void OnToolGUI(EditorWindow window)
    {
    	window.ShowNotification(new GUIContent("Move Right"));
    	
    	//检查是否有发生滑动修改
    	EditorGUI.BeginChangeCheck();
    	Vector3 position = Tools.handlePosition;
    	using(new Handles.DrawingScope(Color.green))
        {
        	position = Handles.Slider(position, Vector3.right);
        }
        if(EditorGUI.EndChangeCheck())
        {
        	Vector3 delta = position - Tools.handlePosition;
        	Undo.RecordObjects(Selection.transforms, "Move Platforms");
        	foreach(var transform in Selection.transforms)
            {
            	transform.position += delta;
            }
        }
    }

}


```



# 使用XML去写编辑器界面：比如 Unity 中的 UIElement UIToolKit



用XML 去渲染 Label TextField TextArea Button

在 OnEnable 中 使用 XMLDocument 去 LoadXml 加载 XML文件 并 解析XML

在 OnGUI 中 进行绘制



# Undo：撤销操作注册 Ctrl+Z

Editor 文件夹下

```c#

var newObj = new GameObject("Undo");
Undo.RegisterCreatedObjectUndo(newObj, "CreateObj");

var trans = Selection.activeGameObject.transform;
if(trans)
{
    Undo.RecordObject(trans, "MoveObj");
    trans.position += Vector3.up;
}

var selectionObj = Selection.activeGameObject;
if(selectionObj)
{
    Undo.AddComponent(selectionObj, typeof(Rigidbody));
}

var selectionObj = Selection.activeGameObject;
if(selectionObj)
{
    Undo.DestroyObjectImmediate(selectionObj);
}

var trans = Selection.activeGameObject.transform;
var root = Camera.main.transform;
if(trans)
{
    Undo.SetTransformParent(trans, root, trans.name);
}




```



# GenericMenu 右键菜单

Editor文件夹下

EditorWindow 的 OnGUI 中 

```c#
private void OnGUI()
{
    var e = Event.current;
    if(e != null)
    {
		if(e.type == EventType.MouseDown && e.button == 1)
        {
            var genericMenu = new GenericMenu();
            genericMenu.AddItem(new GUIContent("功能1"), false, ()=>{ Debug.Log("功能1"); });
            genericMenu.AddSeparator("功能合集/");
            genericMenu.AddItem(new GUIContent("功能合集/功能4"), false, ()=>{ Debug.Log("功能4"); });
            genericMenu.ShowAsContext();
            
            //还可以给邮件菜单设置颜色
            
        }
        
    } 
}
```



# EditorUtility 和 EditorApplication 常用方法

```c#
EditorUtility.DisplayProgressBar - 显示进度条

EditorUtility.ClearProgressBar - 清空进度条

EditorUtility.DisplayDialog - 显示对话框

EditorApplication.Beep - 播放系统声音

EditorApplication.EnterPlaymode - 进入运行模式

EditorApplication.ExitPlaymode - 退出运行模式

EditorApplication.Step - 下一帧

EditorApplication.isPaused - 设置/是否暂停

```













