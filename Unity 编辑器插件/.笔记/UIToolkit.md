# UIToolkit 版本更新不够稳定

还在开发中 v2023.2 版本

技术选型：不适用于需要支持或重用旧的UI内容（API 的支持）



# UIToolkit 与 UGUI



UGUI：UI 在 3D 世界中定位和显示、可以自定义特效着色器和材质、轻松引用 MonoBehavior

UIToolkit：在多屏幕分辨率上运行的屏幕覆盖UI、美术可以参与、无纹理的UI渲染



## UIToolkit  与 UGUI 的 使用区别

UIToolkit 支持、UGUI  不支持：

	1. 可以有全局样式的修改
	1. 可以有 动态纹理图集
	1. 可以有 无纹理元素
	1. 可以实现 用户界面抗锯齿
	1. 支持 UI转换动画

UIToolkit 不支持、 UGUI  支持：

 1. 不支持序列化的事件

 2. 不支持可视化脚本

 3. 不支持 世界空间（3D）渲染

 4. 不支持 自定义材质和着色器

 5. 不支持 遮罩剪裁

 6. 不支持 与动画剪辑（Animation Clips）和时间线（Timeline）集成使用

    

# UIToolkit 与 IMGUI



UI Toolkit  优势 **- 适合制作复杂的编辑器工具**

1、更好的可重用性和解耦性

2、用于编写UI的可视化工具

3、更好的代码维护和性能可扩展性



IMGUI 的优势：

1、对编辑器可扩展功能的无限制访问（更灵活）

2、用于在屏幕上快速渲染UI的Light API



## UIToolkit  与 IMGUI 的 使用区别



UIToolkit 支持、IMGUI 不支持：

 1. 所见即所得

 2. 嵌套可重用组件

 3. 布局和样式调试器

 4. 可缩放文本

    

**IMGUI 支持的，UIToolkit 都支持**



UI Toolkit 更适合 网页 开发经验，支持样式表以及动态和上下文事件处理



# UI Toolkit 的 UI 系统

- **[视觉树](https://docs.unity3d.com/2023.2/Documentation/Manual/UIE-VisualTree.html)
  ：**一个对象图，由轻量级节点组成，包含窗口或面板中的所有元素。它定义了您使用 UI 工具包构建的每个 UI。
- **[控件](https://docs.unity3d.com/2023.2/Documentation/Manual/UIE-Controls.html)：**标准 UI 控件库，例如按钮、弹出窗口、列表视图和颜色选择器。您可以按原样使用它们、自定义它们或创建您自己的控件。
- **[数据绑定系统](https://docs.unity3d.com/2023.2/Documentation/Manual/UIE-Binding.html)：**系统将属性链接到修改其值的控件。
- **[布局引擎](https://docs.unity3d.com/2023.2/Documentation/Manual/UIE-LayoutEngine.html)：**基于 CSS Flexbox 模型的布局系统。它根据布局和样式属性定位元素。
- **[事件系统](https://docs.unity3d.com/2023.2/Documentation/Manual/UIE-Events.html)
  ：**系统将用户交互传达给元素，例如输入、触摸和指针交互、拖放操作和其他事件类型。该系统包括一个调度程序、一个处理程序、一个合成器和一个事件类型库。
- **[UI 渲染器](https://docs.unity3d.com/2023.2/Documentation/Manual/UIE-ui-renderer.html)：**直接构建在 Unity 图形设备层之上的渲染系统。
- **[编辑器 UI 支持](https://docs.unity3d.com/2023.2/Documentation/Manual/UIE-support-for-editor-ui.html)：**一组用于创建编辑器 UI 的组件。
- **[运行时 UI 支持](https://docs.unity3d.com/2023.2/Documentation/Manual/UIE-support-for-runtime-ui.html)：**一组用于创建运行时 UI 的组件。



## 开发UI

- **[UXML 文档](https://docs.unity3d.com/2023.2/Documentation/Manual/UIE-UXML.html)：**类 HTML 和 XML
- **[Unity Style Sheets (USS)](https://docs.unity3d.com/2023.2/Documentation/Manual/UIE-USS.html)：**类 CSS



## UI 工具 和 资源

创建和调试 UI，并学习如何使用 UI 工具包：

- **[UI 调试器](https://docs.unity3d.com/2023.2/Documentation/Manual/UIB-testing-ui.html#ui-toolkit-debugger)：**在Window >  UI Toolkit  >  Debugger
- **[UI Builder](https://docs.unity3d.com/2023.2/Documentation/Manual/UIBuilder.html)：** 以可视化方式创建和编辑 UI （UXML 和 USS 文件）



# 使用



## 创建自定义编辑器窗口

Create > UI Toolkit > Editor Window



## 添加 UI 控件

### UI Builder（以前名为UIElements）

​	https://docs.unity3d.com/Packages/com.unity.ui.builder@1.0/manual/index.html

Unity 2020 版本 安装 UI Builder

PackageManager - + - Add Package from git URL 

​	com.unity.ui

​	com.unity.ui.builder



### UXML



### C# 脚本









