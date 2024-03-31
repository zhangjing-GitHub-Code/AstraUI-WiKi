# 简体中文 
***or [English](#English)***

## 简介
### 基本信息
`astra UI` 是一个基于 `Cpp` 语言的，面向对象开发的多级菜单 UI 框架。
`astra UI` 由各种 `item` 组成，其中包括：
+ 用户层元素
	+ 列表类菜单 `List`
	+ 图标类菜单 `Tile`
	+ 控件 `Widget` *（下个版本）*
+ 系统级元素
	+ 选择器 `Selector`
	+ 摄像机 `Camera` 

## 硬件要求
### 处理器/开发板
`astra UI` 可以支持目前主流的硬件平台，包括 `STM32` 、`Arduino` 、`ESP32` 等。
若您的硬件平台默认情况下不支持 `C++` 编程（如 `STM32` ），可在网络上搜索对应的支持方式。只要硬件平台可以支持 `C++` 编程，`astra UI` 都可以成功部署并运行。
但仍有一些推荐的硬件平台配置，若您的硬件平台满足以下配置，可能会获得比较优秀的使用体验：
+ 主频率大于等于 40 MHz
+ 显示数据传输总线速率（如 `SPI` ）大于 1 Mbps *即将优化*
	+ 在 `astra UI v0.0.2-alpha` 中，数据传输速率会影响各种动画的速度
	+ 后续的版本更新会解决这一问题（详见“更新计划”）
	+ 下个版本后，普通的软件模拟低速 `I2C` 协议即可得到正常的使用体验
	+ 若您的传输速率达不到此要求，动画速度会变慢，但**不会影响使用**
+ RAM 大于等于 128 KB *即将优化*
	+ 后续的版本更新会优化、裁剪程序结构
	+ 经测试，在 `astra UI v0.0.2-alpha` 中：
		+ 不添加任何菜单，单纯部署 `astra UI` 到硬件平台，约占用 80 KB
		+ 添加两级菜单，其中包括一页图标页和一页列表页，约占用 90 KB
		+ 添加两级具有七个元素的菜单，包括一页图标和一页列表，约占用 100 KB
+ 至少一个 `ADC` 端口（可选的，用于生成真随机数，您也可以用其他方式生成）
	+ 每次开机动画背景星星生成的位置使用了真随机数
	+ 以下情况需要您特别配置 `HAL`，详见“移植（部署）教程”
		+ 若您想更改随机数生成方式
		+ 若您不想星星的位置随机生成
		+ 若您使用的硬件平台没有 `ADC` 端口
		+ 若您不想使用 `ADC`
	+ 若您不想在开机时显示 `astra UI` 开机动画，请与作者取得联系获得 `Pro` 版
+ 两个按键，分别用于 `选择上一项/返回上一级` 和 `选择下一项/确认`
	+ 当然您也可以在 `派生HAL` 中自定义按键处理的方式
	+ 关于 `HAL` ，在下文中会有提及

> 在 `astra UI` 测试阶段，笔者使用的硬件平台是 `STM32F103CBT6` 。
> 其具有 128 KB 的 RAM；72 MHz 的主频率。
> 如果您的硬件平台与笔者相同，可以直接刷入仓库中的源码，无需另外移植。
> 笔者的具体接线方式，可参考下文“例程”内容。
### 支持的屏幕
从理论上讲，`astra UI` 不限制使用屏幕的分辨率、颜色和刷新率等。
但是，推荐的屏幕条件如下：
+ 硬件刷新率大于 50 帧每秒（`SSD1306` 的硬件刷新率是 106 帧每秒）
+ 屏幕高度（纵向）大于等于 32 像素
+ 屏幕宽度（横向）大于等于 64 像素

## 文件结构
```BASH
└─astra-ui-v0-0-2-alpha           # astra UI主文件夹
    ├─astra                       # UI文件夹
    │  ├─app                      # app文件夹 后期会merge进另一个项目
    │  │  ├─astra_app.h         
    │  │  ├─astra_app_i.h
    │  │  └─astra_app.cpp
    │  ├─config                   # 设置文件夹
    │  │  └─config.h              # astra UI设置参数结构体
    │  ├─ui                       # 存放各种类型的item
    │  │  ├─element
    │  │  │  ├─page          
    │  │  │  │  ├─item.h          # 各种item类的原型
    │  │  │  │  └─item.cpp        # 各种item类的实现
    │  │  │  └─widget             # 下个版本会更新的控件
    │  │  │     ├─widget.h    
    │  │  │     └─widget.cpp
    │  │  ├─launcher.h            # astra UI启动器 渲染器
    │  │  └─launcher.cpp
    │  ├─astra_logo.h             
    │  ├─astra_logo.cpp
    │  ├─astra_rocket.h           # 负责引导单片机进入astra UI
    │  └─astra_rocket.cpp         # 负责引导单片机进入astra UI
    └─hal                         # 硬件抽象层文件夹
       ├─hal.h                    # HAL父类
       └─hal.cpp                  # 存放了缺省的HAL类方法
```

***若您只是想部署 `astra UI` ，而不是理解它，您可以直接跳过下文，直接阅读“部署教程”。***

## 关于 `Item`

### 基本概念
如前文中所述， `astra UI` 内含很多 `Item` ，包括：
+ 用户层元素
	+ 列表类菜单 `List` ：纵向的列表类型菜单，与下方的 `Tile` 同属于 `Class Menu`
	+ 图标类菜单 `Tile` ：横向的大图标类型菜单
	+ 控件 `Widget` *（下个版本）*：存在于列表类或图标类菜单中的元素，可以改变系统的某些参数/设置
+ 系统级元素

	+ 选择器 `Selector` ：选择器，在列表类中表现为高亮显示的选择框；在图标类中表现为选择方框以及下方的标题文本
	+ 摄像机 `Camera` ：用于设定视野范围，**只会影响背景元素，而不会影响前景元素**，这一点下文中会有提及

下面，就来分别讲述每一个元素。

###  `Menu`

![menu-tree](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/82a3f222-345b-435e-a26d-f0e551f423d2)

上图即为 `Menu` 类的组织框架图。`astra UI` 使用 `树` 来进行多级菜单管理，笔者称其为 `菜单树` 。

`菜单树` 的每个节点，都是一个实例化的菜单类。每个菜单类会包含一个前序指针 `*parent` 以及一个后序指针容器 `std::vector<Menu*> child` 。

使用 `树` 来管理多级菜单的好处是，每个节点都是一个独立的菜单，拥有自己独立的坐标值和参数。保证了每个菜单互不干扰，同时极大地简化了编程难度，也提高了代码可读性。

可以说，使用 `树` 来管理多级菜单，绝对是一个相当符合直觉的思路。

**比如** ，您想在某个菜单（假设其名为 `root` ）里显示"test1"、"test2"、"test3"三个选项，只需要先建立菜单 `root` 。然后调用下文中提到的相应方法，将这三个选项（他们也是菜单类）添加进 `root` 即可。在这个例子中，一共出现了四个新建的菜单类，他们分别是 `root`， `test1`， `test2` 和 `test3` 。而后面三个菜单类被添加进了 `root` 类中，成为了 `root` 类后续指针容器 `std::vector<Menu*> child` 中的一员。相应地，`root` 类也成为了后面三个菜单类的前序指针 `*parent` 。

在渲染 `root` 菜单时，`astra UI` 会遍历 **当前菜单（也就是 `root` ）** 的后序指针容器中的所有菜单类（也就是 `test1`， `test2` 和 `test3`），并在相应的位置显示他们。

至此，`test1`， `test2` 和 `test3` 已经成功显示在了 `root` 菜单中。 

当然，每个菜单类中还存储了坐标值（也就是该菜单在父级菜单中显示的位置）、选择值（也就是用户选择了当前菜单的哪一项）等等。具体您可查阅 `astra UI` 的源代码。此部分内容出现在 `item.h` 和 `item.cpp` 中。

###  `Widget` *(future)*

控件。可以被添加到对应的菜单项中，从而在菜单项的尾部显示。用户可以通过控件来更改某些系统参数的值。

控件有以下几种形式：
+ 互斥复选框。几个选项中，只允许同时打开其中一个
+ 复选框。控制某个参数为0或者1
+ 开关。另一种外观形式的复选框，控制某个参数为0或者1
+ 数值。可以调整某个参数的具体取值
+ 消息弹窗。可以与上面的三种控件进行组合，用于提示用户已经应用更改或者应用失败
+ 确认弹窗。用于进行二次确认，可以应用在有风险的选项中，比如“关机”

*`Widget` 将在下个版本进行更新。在做了在做了~*

## 关于 `Launcher`

`Launcher` 是 `astra UI` 最为重要的组成部分。

`Launcher` 的作用：
+ 管理
+ 接受用户输入的信息，如按键等
+ 调用当前正在显示的菜单实例的 `render()` 方法，渲染当前菜单
+ 

## 接口 / `APIs`
### `Menu`
#### 新建菜单 / 构造函数

新建菜单的函数（即构造函数）是多态的。根据传入的参数不同，创建的菜单也不同。

```Cpp
explicit Menu(std::string _title);                      //创建列表类 标题为_title
Menu(std::string _title, std::vector<uint8_t> _pic);    //创建图标类 标题为_title 图标为_pic
```

通过不同的构造函数创建的菜单，`selfType` 的值会不同，该值表明了这个菜单类实例是列表页还是图标页。

#### `addItem()` 方法

向指定菜单中加入子菜单。

```Cpp
bool addItem(Menu* _page);
```

若添加成功，则返回 `True` ，否则返回 `False` 。 

##### 使用示例

```Cpp
astra::Menu* rootPage = new astra::Menu("root");          //创建一个根菜单
rootPage->addItem(new astra::Menu("test1"));              //向根菜单中添加一个列表类 标题为"test1"
//rootPage->addItem(new astra::Menu("test1", pic_0));     //向根菜单中添加一个图标类 标题为"test1" 图标为pic_0
```

#### `render()` 方法
用于渲染当前页面。请注意， `render()` 方法本质上是在渲染当前菜单

---

# English
***或者 [简体中文](#简体中文)***

## Introduction
### Basic Information
`astra UI` is a multi-level menu UI framework for object-oriented development based on the `Cpp` language.

`astra UI` consists of various `items`, including: 
+ User-Level Element
	+ list menu `List`
	+ icon menu `Tile`
	+ controls `Widgets` *(next version)*.
+ System-Level Elements
	+ selector `Selector`
	+ camera `Camera`

## Hardware Requirement
### MCU / Board
`astra UI` can support current mainstream hardware platforms, including `STM32`, `Arduino`, `ESP32` and so on.

If your hardware platform does not support `C++` programming by default (e.g. `STM32`), you can search for the corresponding support on the web. As long as the hardware platform supports `C++` programming, `astra UI` can be successfully deployed and run.

However, there are still some recommended hardware platform configurations. If your hardware platform meets the following configurations, you may get a better experience:
+ Mains frequency greater than or equal to 40 MHz
+ Display data transfer bus rates (e.g. `SPI`) greater than 1 Mbps *Optimization coming soon*.
	+ In `astra UI v0.0.2-alpha`, the data transfer rate affects the speed of various animations.
	+ This issue will be addressed in a subsequent update (see "Update Program" for details)
	+ After the next release, normal software will be able to emulate the low-speed `I2C` protocol for a normal experience.
	+ If your transfer rate does not meet this requirement, the animation will be slower, but **it won't affect the usage**.
+ RAM greater than or equal to 128 KB *Optimization coming soon*.
	+ The structure of the program will be optimized and trimmed in subsequent updates.
	+ Tested in `astra UI v0.0.2-alpha`:
		+ Deploying `astra UI` to the hardware platform without adding any menus takes about 80 KB.
		+ Add two levels of menus, including a page of icons and a page of lists, which takes up about 90 KB.
		+ Add two levels of menus with seven elements, including a page of icons and a page of lists, approx. 100 KB
+ At least one `ADC` port (optional, for generating true random numbers, you can also generate them in other ways)
	+ A true random number is used to generate the position of the background star for each boot animation.
	+ If you want to change the random number generation, you need to configure the `HAL` port (optional), you can also do it in other ways.
		+ If you want to change the random number generation method
		+ If you don't want the star positions to be randomized
		+ If you are using a hardware platform that does not have an `ADC` port.
		+ If you don't want to use the `ADC` port.
	+ If you don't want to show the `astra UI` startup animation at boot time, please contact the author for the `Pro` version.
+ Two buttons for `Select Previous/Back to Previous` and `Select Next/Confirm`.
	+ Of course you can customize the way the keys are handled in the `Derived HAL`.
	+ The `HAL` is mentioned below.

> In the `astra UI` testing phase, I used the `STM32F103CBT6` hardware platform.
> It has 128 KB of RAM and 72 MHz mains frequency.
> If your hardware platform is the same as mine, you can directly flash the source code from the repository without additional porting.
> For more details about the wiring, please refer to the "Example" below.
### Supported Screens
Theoretically, `astra UI` does not restrict the use of the screen resolution, colors, refresh rate, etc.

However, the recommended screen conditions are as follows:
+ Hardware refresh rate greater than 50 frames per second (106 frames per second for `SSD1306`)
+ Screen height (portrait) greater than or equal to 32 pixels
+ Screen height (portrait) greater than or equal to 32 pixels + Screen width (landscape) greater than or equal to 64 pixels

## File Structure
```BASH
└─astra-ui-v0-0-2-alpha           # astra UI Main Folder
    ├─astra                       # UI Folder
    │  ├─app                      # The app folder will be merged into another project at a later date
    │  │  ├─astra_app.h         
    │  │  ├─astra_app_i.h
    │  │  └─astra_app.cpp
    │  ├─config                   # Setting Up Folders
    │  │  └─config.h              # astra UI Settings Parameter Structures
    │  ├─ui                       # Stores Various Types of Items
    │  │  ├─element
    │  │  │  ├─page          
    │  │  │  │  ├─item.h          # Prototypes For Various Item Classes
    │  │  │  │  └─item.cpp        # Various Implementations of The Item Class
    │  │  │  └─widget             # Controls That Will Be Updated in The Next Version
    │  │  │     ├─widget.h    
    │  │  │     └─widget.cpp
    │  │  ├─launcher.h            # astra UI Launcher And Renderer
    │  │  └─launcher.cpp
    │  ├─astra_logo.h             
    │  ├─astra_logo.cpp
    │  ├─astra_rocket.h           # Responsible for Guiding the Microcontroller Into astra UI
    │  └─astra_rocket.cpp         # Responsible for Guiding the Microcontroller Into astra UI
    └─hal                         # Hardware Abstraction Layer Folder
       ├─hal.h                    # HAL Parent Class
       └─hal.cpp                  # Stores the Default HAL Class Methods
```
## About `Item`

### Basic Concept
todo
###  `Menu`
![menu-tree](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/82a3f222-345b-435e-a26d-f0e551f423d2)
