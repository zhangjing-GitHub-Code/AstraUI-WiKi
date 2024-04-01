# 简体中文 
***or [English](#English)***

## 简介

[完整演示视频](https://www.bilibili.com/video/BV16x421S7qc)

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

![image](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/8303724e-a0c0-4699-8371-411fbd45d6f3)


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

#### 基本概念

![menu-tree](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/82a3f222-345b-435e-a26d-f0e551f423d2)

上图即为 `Menu` 类的组织框架图。`astra UI` 使用 `树` 来进行多级菜单管理，笔者称其为 `菜单树` 。

`菜单树` 的每个节点，都是一个实例化的菜单类。每个菜单类会包含一个前序指针 `*parent` 以及一个后序指针容器 `std::vector<Menu*> child` 。

使用 `树` 来管理多级菜单的好处是，每个节点都是一个独立的菜单，拥有自己独立的坐标值和参数。保证了每个菜单互不干扰，同时极大地简化了编程难度，也提高了代码可读性。

可以说，使用 `树` 来管理多级菜单，绝对是一个相当符合直觉的思路。

**比如** ，您想在某个菜单（假设其名为 `root` ）里显示"test1"、"test2"、"test3"三个选项，只需要先建立菜单 `root` 。然后调用下文中提到的相应方法，将这三个选项（他们也是菜单类）添加进 `root` 即可。在这个例子中，一共出现了四个新建的菜单类，他们分别是 `root`， `test1`， `test2` 和 `test3` 。而后面三个菜单类被添加进了 `root` 类中，成为了 `root` 类后续指针容器 `std::vector<Menu*> child` 中的一员。相应地，`root` 类也成为了后面三个菜单类的前序指针 `*parent` 。

在渲染 `root` 菜单时，`astra UI` 会遍历 **当前菜单（也就是 `root` ）** 的后序指针容器中的所有菜单类（也就是 `test1`， `test2` 和 `test3`），并在相应的位置显示他们（每个菜单类中都存储了自身的坐标值，这个坐标就是其在前序菜单中的位置）

至此，`test1`， `test2` 和 `test3` 已经成功显示在了 `root` 菜单中。 

当然，每个菜单类中还存储了坐标值（也就是该菜单在父级菜单中显示的位置）、选择值（也就是用户选择了当前菜单的哪一项）等等。具体您可查阅 `astra UI` 的源代码。此部分内容出现在 `item.h` 和 `item.cpp` 中。

#### 两种菜单形式

菜单类有两种形式，在 `childType == Menu::LIST` 和 `childType == Menu::TILE` 时，会呈现不同的形式。

下图为 `childType == Menu::TILE` ，呈现的大图标模式。

![image](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/61dda18a-125f-4a79-915f-7ad1f609d1ac)

下图为 `childType == Menu::LIST` ，呈现的列表类模式。

![image](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/e61cc0e4-e689-4de5-8142-363f8e51037b)

#### 动画效果

在菜单被初始化时，您可以根据自己的喜好和需求决定是否让所有选项从头展开，如下图。

![CPT2404011121-228x123](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/e7bdd466-0c21-4544-96c6-cb9a9d582791)

![CPT2404011116-228x122](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/b482efbf-159f-4f4d-8913-b989a556797c)

分别是图标类和列表类的展开动画。

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

###  `Selector`

#### 基本概念

![image](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/bfa9a4d4-2825-4c1d-b125-9f27ce443e49)

上图中高亮显示在"测试测试测试5"上的就是 `Selector(选择器)` 。

选择器在两种菜单模式下有两种不同的形式，当处在列表类时，外观如上图。

当处在大图标类时，外观如下图。

![image](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/be6538c9-2ba2-4dc0-8748-2d52a05d506c)

围绕在屏幕中间图标的四个"L"型边框，以及页面下方的标题显示，都是 `Selector` 的一部分。

#### 作用

`Selector` 承担如下作用：
+ 高亮显示当前选择的项
+ 更改当前菜单中选择的项（即改变菜单 `selectIndex` 的值）

#### 动画效果

##### 过渡动画

在两个不同类型页面进行切换时， `Selector` 会绘制过渡动画，如下图。

![CPT2404011116-228x122](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/3832eef8-3336-427a-adfc-fa953c637584)

![CPT2404011125-228x122](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/5a5e6c8f-611b-4ac7-9773-ac5bf1428acc)

分别是从列表类进入图标类和从图标类进入列表类的过渡动画。

##### 滚动动画

![CPT2404011133-228x120](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/dc854c1b-d2c2-4a77-8edb-79a63802f291)

![CPT2404011136-228x122](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/8ec05741-02be-4d81-989e-25716f45ff26)

分别是在图标类中，选择器快速和慢速滚动的动画效果。

![CPT2404011142-228x127](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/00a6ab62-27c8-4b9d-9d87-768253d7c8bc)

是在列表类中，选择器快速和慢速滚动的动画效果。

#### 注意事项

在正常情况下，与 `Selector` 有关的操作，应该只在 `Launcher` 中出现。`Selector` 属于系统级组件，用户不需要主动管理。

###  `Camera`

#### 基本概念

在 `astra UI` 中， `Camera` 会作为所有渲染函数的参数。用于控制渲染函数渲染对应元素的坐标原点。实现控制视角范围的效果。

在此之前，我们先介绍一个新的概念， `前景(Foreground) ` 和 `背景(Background)` 。

![image](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/8781fff0-3f0c-40f6-9123-044d6cffe382)

如图所示，`前景`表示那些无论页面怎么移动，都不会改变在视线中位置的元素，而 `背景` 表示那些会被视角的改变而影响，会随着页面滚动而滚动的元素。

在概念抽象层面，我们可以理解为，无论页面怎么滚动，视角怎么切换，各种处于 `背景` 的 `Items` 的位置都是不变的，变化的只有 `Camera` 的坐标。

![image](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/eb49f445-0e87-4d69-bf8c-f1eb55f8aba7)

但是在实际的程序层面，所有处于 `背景` 的 `Items` ，在渲染时其坐标都会加上 `Camera` 的坐标。 `Camera` 的坐标确定了其渲染的参考系和坐标原点，如图所示。

![image](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/ba913203-0444-4e5a-a372-563402807253)

`Camera` 只是为了方便移动视角而诞生的概念，如上图所示，实际的渲染过程中，处于 `背景` 的 `Items` 的坐标还是在不断改变的。

#### 动画效果

##### 视角转移动画

![CPT2404011116-228x122](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/9520f440-46a3-4be8-84c6-63e0e47f04bb)

`Camera` 的视角转移与页面类型无关。这里拿图标类做演示。如上图所示，`Camera` 从图标类的最后一个元素移动到了第一个元素。

##### 页面滚动动画

在图标类中， `Camera` 将永远**追随**当前选择的元素（注意，追随并不意味着位置时刻相等， `Camera` 的移动速度应慢于 `Selector` ）

在列表类中，当 `Camera` 检测到当前选择项超出视野，则会触发页面滚动，有两种滚动方式。

###### `goToListItemPage()` 整页滚动



###### `goToListItemRolling()` 单项滚动

![CPT2404011210-228x122](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/08e2c2e8-7b29-42d9-a4f1-3fa0a103132e)

#### 注意事项

在正常情况下，与 `Camera` 有关的操作，应该只在 `Launcher` 中出现。`Camera` 属于系统级组件，用户不需要主动管理。

## 关于 `Launcher`

`Launcher` 是 `astra UI` 最为重要的组成部分。

`Launcher` 的作用：
+ 接收并处理用户输入的信息，如按键等
+ 管理和更新 `Selector` 和 `Camera`
+ 管理 `菜单树` 
+ 打开和关闭菜单
+ 渲染系统级弹窗 `popInfo(std::string _info, uint16_t _time)`
+ 调用当前正在显示的菜单实例的 `render()` 方法，渲染当前菜单

### 关于系统级弹窗 `popInfo(std::string _info, uint16_t _time)`

是优先级最高的渲染操作，即可以无视UI渲染，直接渲染于UI图层上方。

可以用于弹出错误信息和提示信息。

效果如下图：

![CPT2404012129-228x126](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/a69f44fa-7002-4578-9905-37a5d1886ff8)

### 注意事项

在正常情况下，您只需要在引导程序中手动创建一个 `Launcher` 实例，并将您创建好的 `root` 菜单实例作为参数传入 `Launcher` 的 `init()` 成员函数即可。

下面给出一个示例：

```Cpp
astra::Launcher* astraLauncher = new astra::Launcher();     //实例化Launcher
astra::Menu* rootPage = new astra::Menu("root");

rootPage->addItem(new astra::Menu("test1"));
astraLauncher->init(rootPage);                              //初始化Launcher实体

for (;;) astraLauncher->update();                           //在主循环中执行此句即可开始渲染
```

## 接口 / `APIs`
### `Menu`
#### 新建菜单 / 构造函数

新建菜单的函数（即构造函数）是多态的。根据传入的参数不同，创建的菜单也不同。

```Cpp
explicit Menu(std::string _title);                          //创建列表类 标题为_title
Menu(std::string _title, std::vector<uint8_t> _pic);        //创建图标类 标题为_title 图标为_pic
```

通过不同的构造函数创建的菜单，`selfType` 的值会不同，该值表明了这个菜单类实例是列表页还是图标页。

#### `init()` 方法和 `deInit()` 方法

初始化/逆初始化指定菜单。

初始化方法会根据 `Camera` 当前的位置以及配置文件，确定各个元素的位置。同时执行进场动画（注意，非过渡动画）。
逆初始化方法会执行退场动画。

```Cpp
void init(std::vector<float> _camera);
void deInit();
```

##### 注意事项

执行 `init()` 方法是必须的、不可省略的过程。但是通常情况下我们无需主动执行此方法，因为在下文会提到的 `Launcher::open()` 、 `Launcher::close()` 以及 `Launcher::init()` 方法中，都调用了对应菜单的 `init()` 方法。

#### `addItem()` 方法

向指定菜单中加入子菜单。

```Cpp
bool addItem(Menu* _page);
```

若添加成功，则返回 `True` ，否则返回 `False` 。 

##### 使用示例

```Cpp
astra::Menu* rootPage = new astra::Menu("root");            //创建一个根菜单
rootPage->addItem(new astra::Menu("test1"));                //向根菜单中添加一个列表类 标题为"test1"
//rootPage->addItem(new astra::Menu("test1", pic_0));       //向根菜单中添加一个图标类 标题为"test1" 图标为pic_0
```

##### 注意事项
+ 给空菜单 `Menu1` 添加的第一个子菜单 `Menu2` 的类型 `selfType` ，决定了 `Menu1` 菜单的类型，即 `childType` 。
	+ 如果 `Menu2` 是列表类菜单，那么 `Menu1` 就被判定为列表类，以列表类的方式对所有子菜单项进行渲染。
	+ 若之后再向 `Menu1` 中添加子菜单，只能是列表类，以此类推。
	+ 如果向 `Menu1` 中添加了错误类型的子菜单，该方法就会返回 `False` 。 
+ 渲染菜单时，会根据此菜单的 `childType` 进行渲染。

#### `render()` 方法

用于渲染当前页面。

```Cpp
void render(std::vector<float> _camera);                    //render all child item.
```

##### 使用示例

```Cpp
Menu* currentPage;
Camera* camera;

currentPage->render(camera->getPosition());
```

其中，`camera->getPosition()` 方法会在下文中有更详尽的介绍。

##### 注意事项

`render()` 方法本质上是在对应位置渲染当前菜单的**所有后继菜单**，详见[菜单的概念](https://github.com/dcfsswindy/oled-ui-astra/wiki/astra-UI%E4%BB%8B%E7%BB%8D-%E2%80%90-astra-UI-Introduction#menu)

####  `getItemNum()` 方法

返回获取指定菜单包含的子元素个数。

```Cpp
[[nodiscard]] uint8_t getItemNum() const;
```

####  `getItemPosition()` 方法

返回指定菜单中指定索引的元素位置。

```Cpp
[[nodiscard]] Position getItemPosition(uint8_t _index) const;
```

####  `getNext()` 方法

返回指定菜单的正在选择的子元素指针。

```Cpp
[[nodiscard]] Menu* getNext() const;
```

####  `getPreview()` 方法

返回指定菜单的前序菜单指针。

```Cpp
[[nodiscard]] Menu* getPreview() const;
```

###  `Widget`

*(future)*

###  `Camera`

#### 新建 `Camera` / 构造函数

`Camera` 的构造函数是多态的，根据传入的参数不同，会创建不同的 `Camera` 。

```Cpp
Camera();                                                   //创建一个Camera实体 位置在(0, 0)
Camera(float _x, float _y);                                 //创建一个Camera实体 位置在(_x, _y)
```

##### 注意事项

在创建Camera时，若未给出坐标， `(0, 0)` 即是其初始坐标，若给出了坐标值，初始坐标就等于给定的坐标值。

初始坐标存储于Camera实例内部。

####  `outOfView()` 方法

用于判断某个给定点是否位于视野之外。

```Cpp
uint8_t outOfView(float _x, float _y);
```

##### 返回值

+ 当指定坐标位于视野内时，返回 `0`
+ 当指定坐标位于视野上方或左侧时，返回 `1`
+ 当指定坐标位于视野下方或右侧时，返回 `2`

####  `getPosition()` 方法

`Camera` 最重要的一个方法，返回 `Camera` 当前的坐标。

```Cpp
std::vector<float> getPosition();
```

##### 返回值

返回 `Camera` 当前的坐标容器 `{x, y}` 。

其中， `getPosition()[0]` 代表 `Camera` 的 `x` 坐标。

同理， `getPosition()[1]` 代表 `Camera` 的 `y` 坐标。

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
