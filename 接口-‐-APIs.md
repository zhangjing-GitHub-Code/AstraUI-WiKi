# 简体中文
***or [English](#English)***

## 接口 / `APIs`
###  `Menu`
#### 新建菜单 / 构造函数

新建菜单的函数（即构造函数）是多态的。根据传入的参数不同，创建的菜单也不同。

```Cpp
explicit Menu(std::string _title);                          //创建列表类 标题为_title
Menu(std::string _title, std::vector<uint8_t> _pic);        //创建图标类 标题为_title 图标为_pic
```

通过不同的构造函数创建的菜单，`selfType` 的值会不同，该值表明了这个菜单类实例是列表页还是图标页。

####  `init()` 方法和 `deInit()` 方法

初始化/逆初始化指定菜单。

初始化方法会根据 `Camera` 当前的位置以及配置文件，确定各个元素的位置。同时执行进场动画（注意，非过渡动画）。
逆初始化方法会执行退场动画。

```Cpp
void init(std::vector<float> _camera);
void deInit();
```

##### 注意事项

执行 `init()` 方法是必须的、不可省略的过程。但是通常情况下我们无需主动执行此方法，因为在下文会提到的 `Launcher::open()` 、 `Launcher::close()` 以及 `Launcher::init()` 方法中，都调用了对应菜单的 `init()` 方法。

####  `addItem()` 方法

向指定菜单中加入子菜单。

```Cpp
bool addItem(Menu* _page);
```

##### 返回值

+ 若添加成功，返回 `True` 。
+ 若添加失败，返回 `False` 。 

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

####  `render()` 方法

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

获取指定菜单包含的子元素个数。

```Cpp
[[nodiscard]] uint8_t getItemNum() const;
```

##### 返回值

返回指定菜单包含的子元素个数。

####  `getItemPosition()` 方法

获取指定菜单中指定索引的元素位置。

```Cpp
[[nodiscard]] Position getItemPosition(uint8_t _index) const;
```

##### 返回值

返回指定菜单中指定索引的元素位置。

####  `getNext()` 方法

获取指定菜单的正在选择的子元素指针。

```Cpp
[[nodiscard]] Menu* getNext() const;
```

##### 返回值

返回指定菜单的正在选择的子元素指针。

####  `getPreview()` 方法

获取指定菜单的前序菜单指针。

```Cpp
[[nodiscard]] Menu* getPreview() const;
```

##### 返回值

返回指定菜单的前序菜单指针。

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

####  `go()` 及其一系列方法

用于将 `Camera` 移动到指定的位置。共有四个移动函数。

```Cpp
void go(float _x, float _y);                                //将Camera移动至(_x, _y) 并渲染平滑的移动动画
void goDirect(float _x, float _y);                          //直接移动 不渲染动画
void goHorizontal(float _x);                                //水平移动 不改变纵坐标
void goVertical(float _y);                                  //垂直移动 不改变横坐标
```

##### 注意事项

由于需要渲染动画，而动画的渲染并非一瞬间可以完成的，所以只有 `goDirect()` 可以在循环外执行，其余三个方法都必须在循环内执行。

同时，笔者也建议（或者说是您应该）在诸如初始化这样的仅执行一次的方法中，使用 `goDirect()` 方法对 `Camera` 进行移动，而不是其余三个方法。

#### 页面滚动方法

包含五个方法，它们用于在元素溢出视野后，对 `Camera` 进行移动。

```Cpp
void goToNextPageItem();                                    //将Camera移动到下一页 用于列表类整页滚动
void goToPreviewPageItem();                                 //将Camera移动到上一页 用于列表类整页滚动
void goToListItemPage(uint8_t _index);                      //将Camera移动到指定的页面 用于列表类整页滚动
void goToListItemRolling(std::vector<float> _posSelector);  //将Camera滚动一行或多行 用于列表类单行滚动
void goToTileItem(uint8_t _index);                          //将Camera移动到指定的元素 用于图标页居中显示选择项
```

##### 注意事项

确保您当前菜单的 `childType` 与上方注释中方法的适用页面类型一致，否则会产生意想不到的错误。

此隐患可能会在未来版本中进行规避。

####  `isMoving()` 方法

判断并返回 `Camera` 当前的状态。

```Cpp
bool isMoving();
```

##### 返回值
+ 当 `Camera` 正在移动，返回 `True` 。
+ 当 `Camera` 未在移动，返回 `False` 。

####  `reset()` 方法

将 `Camera` 移动回[初始位置](https://github.com/dcfsswindy/oled-ui-astra/wiki/%E6%8E%A5%E5%8F%A3-%E2%80%90-APIs#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9-3)。

```Cpp
void reset();
```

####  `update()` 方法

接管一切关于 `Camera` 的移动。同时更新 `Camera` 的位置。

所以，用户无需主动执行上述移动 `Camera` 的相关方法，由 `update()` 方法进行托管即可。

```Cpp
void update(Menu *_menu, Selector *_selector);
```

##### 注意事项

需放在循环内执行。

###  `Selector`

#### 新建 `Selector` / 构造函数

并无特殊处理

```Cpp
Selector() = default;
```

####  `getPosition()` 方法

获取 `Selector` 的坐标。

```Cpp
std::vector<float> getPosition();
```

##### 返回值

返回 `Selector` 的坐标容器，[参考这里](https://github.com/dcfsswindy/oled-ui-astra/wiki/%E6%8E%A5%E5%8F%A3-%E2%80%90-APIs#getposition-%E6%96%B9%E6%B3%95)。

####  `inject()` 方法

将某个菜单注入 `Selector` ，从而让 `Selector` 可以获取到对应菜单的坐标值、索引值等等。

```Cpp
bool inject(Menu* _menu); //inject menu instance to prepare for render.
```

##### 返回值

+ 若操作合法，返回 `True` 。
+ 若操作非法，如当前注入的是一个不存在的菜单，返回 `False` 。

####  `destory()` 方法

销毁并释放注入的菜单。

```Cpp
bool destroy(); //destroy menu instance.
```

##### 返回值

+ 若操作合法，返回 `True` 。
+ 若操作非法，如当前并未注入任何菜单，返回 `False` 。

####  `go()` 方法

将 `Selector` 移动到菜单的指定元素处。

同时更改当前注入了的菜单的 `selectIndex`。

```Cpp
void go(uint8_t _index);
```

##### 注意事项

`Selector` 移动是有动画的，所以该方法需要放在循环中调用。

####  `render()` 方法

渲染 `Selector` 到画布。

```Cpp
void render(std::vector<float> _camera);
```
##### 注意事项

因为涉及到渲染，需要在循环内调用。

###  `Launcher`

#### 新建 `Launcher` / 构造函数

并无特殊处理。

```Cpp
Launcher() = default;
```
####  `popInfo()` 方法

系统级弹窗。

```Cpp
void popInfo(std::string _info, uint16_t _time);
```

####  `init()` 方法

初始化方法，用于初始化 `Launcher` 。

执行此方法后， `Launcher` 就根据您传入的 `根菜单` 创建了一个新的 `菜单树` 。

```Cpp
void init(Menu* _rootPage);
```

##### 使用示例

```Cpp
astra::Launcher* astraLauncher = new astra::Launcher();
astra::Menu* rootPage = new astra::Menu("root");

astraLauncher->init(rootPage);
```

##### 注意事项

注意，此方法需要用户主动在代码的初始化阶段执行。

####  `open()` 方法

打开 `Launcher` 的 `当前页面指针` 的 `selectIndex(当前选择项)` 对应的子菜单。

比如当前处在 `根菜单` ，并且选择了 `根菜单` 的第二项，那么执行此方法便会打开 `根菜单` 的第二个子菜单。同时移动 `当前页面指针` 到新的页面。

```Cpp
bool open();
``` 

##### 执行步骤
+ 判断操作是否合法，若不合法弹出错误弹窗。
+ 调用旧菜单的 `deInit()` 方法。
+ 移动 `当前页面指针` 到新的页面。
+ 调用新菜单的 `init()` 方法。
+ 调用 `Selector` 的 `inject()` 方法，将选择器注入新菜单。

##### 返回值
+ 操作非法，返回 `False`。
+ 操作合法，返回 `True`。

##### 注意事项

若您尝试打开一个空的菜单，在默认情况下， `Launcher` 会弹出内容为"empty page!"的系统级弹窗。同时，各种指针将不会被移动。

若您尝试打开一个不存在的菜单，在默认情况下， `Launcher` 会弹出内容为"unreferenced page!"的系统级弹窗。同时，各种指针将不会被移动。

####  `close()` 方法

关闭当前页面，返回当前页面的前序页面。

同时移动 `Launcher` 的 `当前页面指针` 到新的页面。

```Cpp
bool close();
``` 

##### 执行步骤
+ 判断操作是否合法，若不合法弹出错误弹窗。
+ 调用旧菜单的 `deInit()` 方法。
+ 移动 `当前页面指针` 到新的页面。
+ 调用新菜单的 `init()` 方法。
+ 调用 `Selector` 的 `inject()` 方法，将选择器注入新菜单。

##### 返回值
+ 操作非法，返回 `False`。
+ 操作合法，返回 `True`。

##### 注意事项

若您尝试回到一个空的菜单，在默认情况下， `Launcher` 会弹出内容为"empty page!"的系统级弹窗。同时，各种指针将不会被移动。

若您尝试回到一个不存在的菜单，在默认情况下， `Launcher` 会弹出内容为"unreferenced page!"的系统级弹窗。同时，各种指针将不会被移动。

####  **`update()` 方法**

**接管一切。**

`astra UI` ，启动！

启动并渲染 `astra UI` 。

```Cpp
void update();
```

##### 注意事项

此方法应该且只能在程序主循环中调用。

---

# English
***或者 [简体中文](#简体中文)***

## Interfaces / `APIs`

###  `Menu`

#### New Menu / Constructor

The function that creates the new menu (i.e. the constructor) is polymorphic. Depending on the parameters passed in, different menus are created.

```Cpp
explicit Menu(std::string _title);                          //Create list class with title _title
Menu(std::string _title, std::vector<uint8_t> _pic);        //Create icon class Title as _title Icon as _pic
```

Menus created through different constructors will have different values for `selfType`, which indicates whether the menu class instance is a list page or an icon page.

####  `init()` method and `deInit()` method

Initializes/inverse initializes the specified menu.

The initialization method determines the position of each element based on the current position of the `Camera` and the configuration file. It also performs an entry animation (not a transition animation).
The inverse initialization method performs an exit animation.

```Cpp
void init(std::vector<float> _camera);
void deInit();
```

##### Note

Executing the `init()` method is mandatory and cannot be omitted. But usually we don't need to execute this method, because the `Launcher::open()` , `Launcher::close()` , and `Launcher::init()` methods mentioned in the following sections all call the `init()` method of the corresponding menu.

####  `addItem()` method

Adds a submenu to the specified menu.

```Cpp
bool addItem(Menu* _page);
```

##### Return Value

+ Returns `True` if the add is successful.
+ Returns `False` if addition fails. 

##### Usage Examples

```Cpp
astra::Menu* rootPage = new astra::Menu("root");            //Create a root menu
rootPage->addItem(new astra::Menu("test1"));                //Add a list class to the root menu titled "test1".
//rootPage->addItem(new astra::Menu("test1", pic_0));       //Add an icon class to the root menu with the title "test1" and the icon pic_0.
```

##### Note
+ The type `selfType` of the first submenu `Menu2` added to the empty menu `Menu1` determines the type of the `Menu1` menu, i.e. `childType` .
	+ If `Menu2` is a list class menu, then `Menu1` is determined to be a list class, and all submenu items are rendered as such.
	+ If submenus are added to `Menu1` later, they can only be of list class, and so on.
	+ If a submenu of the wrong type is added to `Menu1` , the method returns `False` . 
+ When rendering a menu, it is rendered according to the `childType` of this menu.

####  `render()` method

Used to render the current page.

```Cpp
void render(std::vector<float> _camera);                    //render all child item.
```

##### Usage Example

```Cpp
Menu* currentPage;
Camera* camera;

currentPage->render(camera->getPosition());
```

The `camera->getPosition()` method is described in more detail below.

##### Notes

The `render()` method essentially renders **all subsequent menus** of the current menu at the corresponding location, see [menu concepts](https://github.com/dcfsswindy/oled-ui-astra/wiki/astra-UI%E4%BB%8B%E7%BB%8D-%E2%80%90-astra-UI-Introduction#menu-1) for details.

####  `getItemNum()` method

Gets the number of child elements the specified menu contains.

```Cpp
[[nodiscard]] uint8_t getItemNum() const;
```

##### Return Value

Returns the number of child elements contained in the specified menu.

####  `getItemPosition()` method

Gets the position of the element at the specified index in the specified menu.

```Cpp
[[nodiscard]] Position getItemPosition(uint8_t _index) const;
```

##### Return Value

Returns the position of the element at the specified index in the specified menu.

####  `getNext()` method

Gets a pointer to the child element of the specified menu that is being selected.

```Cpp
[[nodiscard]] Menu* getNext() const;
```

##### Return Value

Returns a pointer to the child element of the specified menu that is being selected.

####  `getPreview()` method

Gets the preview menu pointer for the specified menu.

```Cpp
[[nodiscard]] Menu* getPreview() const;
```

##### Return Value

Returns a pointer to the preorder menu for the specified menu.

###  `Widget`

*(future)*

###  `Camera`

#### New `Camera` / Constructors

The `Camera` constructor is polymorphic and creates a different `Camera` depending on the parameters passed in.

```Cpp
Camera();                                                   //Create a Camera entity positioned at (0, 0)
Camera(float _x, float _y);                                 //Create a Camera entity Location at (_x, _y)
```

##### Note

When creating a Camera, if no coordinates are given, `(0, 0)` is its initial coordinates, if coordinates are given, the initial coordinates are equal to the given coordinates.

The initial coordinates are stored inside the Camera instance.

####  `outOfView()` method

Used to determine if a given point is outside the view.

```Cpp
uint8_t outOfView(float _x, float _y);
```

##### Return Value

+ Returns `0` when the specified coordinates are inside the field of view.
+ Returns `1` when the specified coordinates are above or to the left of the field of view.
+ Returns `2` when the specified coordinate is below or to the right of the field of view

####  `getPosition()` method

The most important method of `Camera` , returns the current position of `Camera` .

```Cpp
std::vector<float> getPosition();
```

##### Return Value

Returns the `{x, y}` container of the current coordinates of the `Camera` .

Where `getPosition()[0]` represents the `x` coordinate of `Camera` .

Similarly, `getPosition()[1]` represents the `y` coordinate of `Camera` .

####  `go()` and its set of methods

are used to move the `Camera` to the specified position. There are four move functions.

```Cpp
void go(float _x, float _y);                                //Moves the Camera to (_x, _y) and renders a smooth moving animation
void goDirect(float _x, float _y);                          //Direct movement No rendering animation
void goHorizontal(float _x);                                //Horizontal movement No change in vertical coordinate
void goVertical(float _y);                                  //Vertical movement No change in horizontal coordinates
```

##### Note

Since we need to render the animation, and the rendering of the animation is not done instantly, only `goDirect()` can be executed outside the loop, the other three methods must be executed inside the loop.

It is also recommended (or you should) to use the `goDirect()` method to move the `Camera` in a one-time-only method such as initialize, instead of the other three methods.

#### Page Scroll Methods

Contains five methods that are used to move `Camera` after an element has overflowed the field of view.

```Cpp
void goToNextPageItem();                                    //Move Camera to next page For list class full page scrolling
void goToPreviewPageItem();                                 //Move Camera to previous page For list class full page scrolling
void goToListItemPage(uint8_t _index);                      //Moves the Camera to the specified page Used for full page scrolling in list classes
void goToListItemRolling(std::vector<float> _posSelector);  //Scroll Camera by one or more rows For single row scrolling in list class
void goToTileItem(uint8_t _index);                          //Moves the Camera to the specified element Used to center the selection on the icon page
```

##### Note 

Make sure that the `childType` of your current menu matches the applicable page type of the method in the note above, or you will get an unexpected error.

This pitfall may be circumvented in a future release.

####  `isMoving()` method

Determines and returns the current state of the `Camera` .

```Cpp
bool isMoving();
```

##### Return Value
+ When `Camera` is moving, return `True` .
+ When `Camera` is not moving, return `False` .

#### `reset()` method

Move `Camera` back to [initial position](https://github.com/dcfsswindy/oled-ui-astra/wiki/%E6%8E%A5%E5%8F%A3-%E2%80%90-APIs#note-2).

```Cpp
void reset();
```

####  `update()` method

Takes over all movement of the `Camera`. It also updates the position of `Camera` .

So, you don't need to actively execute the above methods related to moving `Camera` , the `update()` method will take care of it.

```Cpp
void update(Menu *_menu, Selector *_selector);
```

##### Note

It needs to be executed inside a loop.

###  `Selector`

#### New `Selector` / Constructor

No special handling.

```Cpp
Selector() = default;
```

####  `getPosition()` method

Get the coordinates of the `Selector` .

```Cpp
std::vector<float> getPosition();
```

##### Return Value

Returns the coordinate container of the `Selector` , [refer here](https://github.com/dcfsswindy/oled-ui-astra/wiki/%E6%8E%A5%E5%8F%A3-%E2%80%90-APIs#getposition-method)

####  `inject()` method

Injects a menu into a `Selector` so that the `Selector` can get the coordinates, index, etc. of the corresponding menu.

```Cpp
bool inject(Menu* _menu); //inject menu instance to prepare for render.
```

##### Return Value

+ Returns `True` if the operation is legal.
+ Returns `False` if the operation is illegal, e.g., if a non-existent menu is currently injected.

####  `destory()` method

Destroys and frees the injected menu.

```Cpp
bool destroy(); //destroy menu instance.
```

##### Return Value

+ Returns `True` if the operation is legal.
+ Returns `False` if the operation is illegal, e.g. no menu is currently injected.

####  `go()` method

Moves the `Selector` to the specified element of the menu.

Also changes the `selectIndex` of the currently injected menu.

```Cpp
void go(uint8_t _index);
```

##### Notes

The `Selector` movement is animated, so this method needs to be called in a loop.

####  `render()` method

Renders the `Selector` to the canvas.

```Cpp
void render(std::vector<float> _camera);
```

##### Note

Because it involves rendering, it needs to be called inside a loop.

###  `Launcher`

#### New `Launcher` / Constructor

There is no special treatment.

```Cpp
Launcher() = default;
```

####  `popInfo()` method

System-level pop-ups.

```Cpp
void popInfo(std::string _info, uint16_t _time);
```

####  `init()` method

Initialization method to initialize the `Launcher` .

After executing this method, the `Launcher` creates a new `MenuTree` based on the `RootMenu` you passed in.

```Cpp
void init(Menu* _rootPage);
```

##### Usage Example

```Cpp
astra::Launcher* astraLauncher = new astra::Launcher();
astra::Menu* rootPage = new astra::Menu("root");

astraLauncher->init(rootPage);
```

##### Note

Note that this method requires user-initiated execution during the initialization phase of the code.

####  `open()` method

Open the submenu corresponding to `selectIndex(current selection)` of `CurrentPagePointer` of `Launcher` .

For example, if you are currently in the `root menu` and have selected the second item of the `root menu` , then this method will open the second submenu of the `root menu` . At the same time, it moves the `current page pointer' to a new page.

```Cpp
bool open();
``` 

##### Execution steps
+ Determine whether the operation is legal or not, if not, pop up an error popup.
+ Call `deInit()` method of old menu.
+ Move `current page pointer` to new page.
+ Call the `init()` method of the new menu.
+ Call the `inject()` method of `Selector` to inject the selector into the new menu.

##### Return Value
+ Returns `False` if the operation is illegal.
+ Returns `True` if the operation is legal.

##### Note

If you try to open an empty menu, by default, the `Launcher` will display a system-level pop-up window with the message "empty page!". Also, the various pointers will not be moved.

If you try to open a menu that doesn't exist, by default the `Launcher` will show a system-level popup with the message "unreferenced page!". Also, the various pointers will not be moved.

####  `close()` method

Close the current page and return to the previous page of the current page.

Also moves the `Launcher`'s `current page pointer` to a new page.

```Cpp
bool close();
``` 

##### Execution steps
+ Determine whether the operation is legal or not, if not, pop up an error popup.
+ Call `deInit()` method of old menu.
+ Move `current page pointer` to new page.
+ Call the `init()` method of the new menu.
+ Call the `inject()` method of `Selector` to inject the selector into the new menu.

##### Return Value
+ Returns `False` if the operation is illegal.
+ Returns `True` if the operation is legal.

##### Note

If you try to go back to an empty menu, by default the `Launcher` will show a system level popup with the message "empty page!". Also, the various pointers will not be moved.

If you try to go back to a menu that doesn't exist, by default the `Launcher` will show a system-level popup with the message "unreferenced page!". Also, the various pointers will not be moved.

####  **`update()` method**

**takes over everything.**

`astra UI` , start!

Starts and renders `astra UI` .

```Cpp
void update();
```

##### Note

This method should and can only be called in the main loop of the program.

:P