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

将 `Camera` 移动回[初始位置](https://github.com/dcfsswindy/oled-ui-astra/wiki/astra-UI%E4%BB%8B%E7%BB%8D-%E2%80%90-astra-UI-Introduction#%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9-1)。

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

##### 注意事项

`Selector` 移动是有动画的，所以该方法需要放在循环中调用。

```Cpp
void go(uint8_t _index);
```

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

# English
***或者 [简体中文](#简体中文)***
