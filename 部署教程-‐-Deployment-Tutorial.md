# 简体中文 
***or [English](#English)***

## 部署教程

### 前言

在一切开始之前，您需要确保您的 `编译器` ，或者说 `开发环境` 支持 `C++` 编程。

下面将给出一些已明确的支持 `C++` 编程的 `开发环境` ：

1. Keil
2. CLion
3. Arduino IDE (原生支持 `C++` )
4. STM32CubeIDE
5. IAR
6. Eclipse
7. ...

如果您正在使用上述的某一种 `开发环境` ，您还需要通过某些修改，使得您的 `开发环境` 可以编译并烧录 `C++` 程序到您的硬件平台。

下面给出使用 `CLion` + `STM32` 的一个[范例教程](https://blog.csdn.net/weixin_44934226/article/details/124783825)

如果您使用的是其他 `开发环境` 或者 `硬件平台` ， 可以自行检索具体的方法。**相信我，如果您可以看到这篇文章，那您就一定可以做好这一步。**

进行了上述操作后，如果您的 `开发环境` 并非 `原生支持C++` 。您还需要编写以下两个文件，放入工程文件夹内。

```Cpp
//test.h

#ifndef __TEST_H_
#define __TEST_H_
#ifdef __cplusplus
extern "C" {
#endif

/*---c---*/
#include "main.h"
#include "gpio.h"

//此处添加其它C头文件 add other c header there.

void CppMain();  //主程序函数 main program method.

#ifdef __cplusplus
}

/*---c++---*/
#include "LED.h"

//此处添加其他Cpp头文件 add other cpp header there.

#endif
#endif
```

```Cpp
//test.cpp

//主程序入口 main program
void CppMain() {
  for(;;) {
    //主循环 main loop
  }
}
```

最后一步，您需要在您的 `main.c` 中包含您编写好的头文件，并按下文方式调用您编写好的引导函数。

```Cpp
//main.c

#include "test.h"

.
.
.

CppMain();  
//在进入原本的主循环之前调用该函数 接管程序
//Call this function to take over the program before entering the original main loop.

while(1) {
  //弃用 deprecated.
}
```

万事开头难，您已经做完了最难的一步，接下来让我们开始吧！

### 关于 `HAL`

特别感谢 @Forairaaaaa ，笔者基于其开源的 `monica` ，学习到了很多相关知识。

在正式开始部署教程前，笔者需要先向您介绍 `astra UI` 配套的 `HAL` ，即 `硬件抽象层` 。

在 `astra UI` 中，系统配置、屏幕驱动、图形绘制、I/O口驱动和生成随机数都集成在了 `HAL` 中。

`HAL` 保证了 `astra UI` 的运行不受硬件平台的限制。 **同时，您绝对不会在任何 `astra UI` 的代码文件中看到任何一行与硬件平台有关的代码。** 如 `digitalWrite()` 或 `HAL_GPIOWritePin()` 等。

例如，当您想让系统延时500ms时，您无需执行 `delay(500); (Arduino)` 或 `HAL_Delay(500); (STM32)` ，您只需要执行 `HAL::Delay(500);` 即可。前者的代码不能在不同平台执行，也非常不易于移植，但采用了 `HAL` 的后者，与硬件平台无关，可以在任何平台执行。

这使得 `astra UI` 几乎可以在任何主流硬件平台上部署和运行，只需要您针对您自己的硬件平台 `重载` 对应的 `HAL` 函数即可。

若您还是有些迷惑，篇幅所限，笔者在这里附上一部分 `HAL` 类的原型代码：

```Cpp
class HAL {
private:
  static HAL *hal;

public:
  static HAL *get();    //get hal instance.
  static bool check();  //check if there is a hal instance.

  static bool inject(HAL *_hal);  //inject HAL instance and run hal_init.
  static void destroy();  //destroy HAL instance.

  virtual ~HAL() = default;

  virtual std::string type() { return "Base"; }

  virtual void init() {}

protected:
  sys::config config;

public:
  static void canvasUpdate() { get()->_canvasUpdate(); }
  virtual void _canvasUpdate() {}

  static void canvasClear() { get()->_canvasClear(); }
  virtual void _canvasClear() {}

  static void drawPixel(float _x, float _y) { get()->_drawPixel(_x, _y); }
  virtual void _drawPixel(float _x, float _y) {}

  static void drawEnglish(float _x, float _y, const std::string &_text) { get()->_drawEnglish(_x, _y, _text); }
  virtual void _drawEnglish(float _x, float _y, const std::string &_text) {}

  static void drawChinese(float _x, float _y, const std::string &_text) { get()->_drawChinese(_x, _y, _text); }
  virtual void _drawChinese(float _x, float _y, const std::string &_text) {}

  /**
   * @brief system timers.
   */
public:
  static void delay(unsigned long _mill) { get()->_delay(_mill); }
  virtual void _delay(unsigned long _mill) {}

  //...
};
```

可以发现， `HAL` 是由很多组函数构成，分别是一个静态成员函数和一个名字以下划线开头的虚成员函数。

每组函数中，静态成员函数的任务就是调用 `get()` 方法，获取到当前存储在 `HAL` 类中的 `HAL` 实例（也就是下文中您自己编写的派生 `HAL` ）。而后调用 `HAL` 实例中的相关方法。

下面是 `inject()` 方法的实现，它用于将派生 `HAL` 注入回 `HAL` 。并将派生 `HAL` 指针存储于 `HAL` 中。

```Cpp
bool HAL::inject(HAL *_hal) {
  if (_hal == nullptr) {
    return false;
  }

  _hal->init();
  hal = _hal;
  return true;
}
```

所以不难发现，您需要做的事情有三个：

1. 继承此 `HAL` 类，编写属于您自己硬件平台的派生 `HAL`
2. 在派生 `HAL` 中重写所有您需要用到的虚函数
3. 将您编写的派生 `HAL` 通过 `inject()` 方法注入回 `HAL`

下面，笔者将以更详尽的描述，带您一步步部署 `astra UI` 到您的硬件平台。

### 第一步：编写派生 `HAL`

#### 编写框架

首先您需要在您的 `.h` 文件中包含 `hal.h` 头文件。

继承并编写派生 `HAL` ，同时给您的派生 `HAL` 命名，这里以 `MyHAL` 为例，下文中都用 `MyHAL` 指代派生 `HAL` 。

顺带编写一个默认的构造函数。 ^_^

```Cpp
//MyHAL.h

#pragma once
#ifndef MYHAL_H_
#define MYHAL_H_

#include "hal.h"

class MyHAL : public HAL {
public:
  MyHAL() = default;
};

#endif
```

#### 编写 `MyHAL` 的 `_xx_init()` 方法并重写 `init` 方法

这一步是因人而异的，取决于您的硬件平台和您接入的各种设备。

`init()` 方法会在您调用 `HAL::inject()` 方法时自动被调用。

`_xx_init()` 方法仅存在于您的派生 `HAL` 中，并应该在您派生类的 `init()` 方法中调用。

其用于初始化您的所有个性化设备。

请注意，如果您需要加入图形库（笔者也推荐您这么做），请同时也在 `MyHAL.h` 中包含您图形库的 `.h` 文件，并为其编写 `xx_init()` 方法。

下面以笔者使用的 `STM32` 和 `u8g2` 图形库为例，给出一段代码：

```Cpp
//MyHAL.h

class MyHAL : public HAL {
private:
  void _stm32_hal_init();
  void _sys_clock_init();
  void _gpio_init();
  void _dma_init();
  void _timer_init();
  void _spi_init();
  void _adc_init();

  void _ssd1306_init();
  void _key_init();
  void _buzzer_init();
  void _u8g2_init();

public:
  inline void init() override {
    _stm32_hal_init();
    _sys_clock_init();
    _gpio_init();
    _dma_init();
    _timer_init();
    _spi_init();
    _adc_init();

    _ssd1306_init();
    _key_init();
    _buzzer_init();
    _u8g2_init();
  }
};
```

再给出上面某些函数的实现，供您参考：

```Cpp
//MyHAL_STM32.cpp

void MyHAL::_gpio_init() {
  MX_GPIO_Init();
}

void MyHAL::_adc_init() {
  MX_ADC1_Init();
}

void MyHAL::_spi_init() {
  MX_SPI2_Init();
}
```

请注意，这些方法的实现需要在 `MyHAL.cpp` 文件中编写，当然，笔者建议您把不同种类的设备放在不同的 `.cpp` 文件中。

如 `MyHAL_key.cpp` 、 `MyHAL_oled.cpp` 和 `MyHAL_STM32.cpp` 等。

#### 重写其他方法

接下来是重头戏。

您需要根据您选择的图形库（或者在不借助图形库的情况下），**重写**绘制像素、绘制直线、绘制矩形等一系列方法。

您需要根据您的屏幕，重写刷新画布、清空屏幕等一系列方法。

您需要根据您的硬件平台，重写延时、获取开机时间、生成真随机数等一系列方法。

您需要根据您的外设类型，重写蜂鸣器发声、按键扫描等一系列方法。

当然，如果您明确您的需求，可以选择不重写某些方法，使用 `HAL` 父类中默认的方法实现。

比如，如果您不需要生成真随机数，您就无需重写 `_getRandomSeed()` 方法，或者直接让其返回一个固定值即可。

比如，您不需或没有蜂鸣器，您就无需重写 `_beep()` 及其一系列方法。

下面是笔者所有重写了的函数，供您参考，完整的可以重写的函数列表，您可以查阅 `hal.h`

```Cpp
//MyHAL.h

class MyHAL : public HAL {
protected:
  u8g2_t canvasBuffer {};

public:
  void _screenOn() override;
  void _screenOff() override;

public:
  void* _getCanvasBuffer() override;
  uint8_t _getBufferTileHeight() override;
  uint8_t _getBufferTileWidth() override;
  void _canvasUpdate() override;
  void _canvasClear() override;
  void _setFont(const uint8_t * _font) override;
  uint8_t _getFontWidth(std::string& _text) override;
  uint8_t _getFontHeight() override;
  void _setDrawType(uint8_t _type) override;
  void _drawPixel(float _x, float _y) override;
  void _drawEnglish(float _x, float _y, const std::string& _text) override;
  void _drawChinese(float _x, float _y, const std::string& _text) override;
  void _drawVDottedLine(float _x, float _y, float _h) override;
  void _drawHDottedLine(float _x, float _y, float _l) override;
  void _drawVLine(float _x, float _y, float _h) override;
  void _drawHLine(float _x, float _y, float _l) override;
  void _drawBMP(float _x, float _y, float _w, float _h, const uint8_t* _bitMap) override;
  void _drawBox(float _x, float _y, float _w, float _h) override;
  void _drawRBox(float _x, float _y, float _w, float _h, float _r) override;
  void _drawFrame(float _x, float _y, float _w, float _h) override;
  void _drawRFrame(float _x, float _y, float _w, float _h, float _r) override;

public:
  void _delay(unsigned long _mill) override;
  unsigned long _millis() override;
  unsigned long _getTick() override;
  unsigned long _getRandomSeed() override;

public:
  void _beep(float _freq) override;
  void _beepStop() override;
  void _setBeepVol(uint8_t _vol) override;

public:
  bool _getKey(key::KEY_INDEX _keyIndex) override;

public:
  void _updateConfig() override;
};
```

同时，下面以笔者使用的 `STM32` 和 `u8g2` 图形库为例，给出一段重写的函数实现代码：

```Cpp
//MyHAL_oled.cpp

void HALDreamCore::_drawPixel(float _x, float _y) {
  u8g2_DrawPixel(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y));
}

void HALDreamCore::_drawChinese(float _x, float _y, const std::string &_text) {
  u8g2_DrawUTF8(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), _text.c_str());
}

void HALDreamCore::_drawVDottedLine(float _x, float _y, float _h) {
  for (uint8_t i = 0; i < (uint8_t)std::round(_h); i++) {
    if (i % 8 == 0 | (i - 1) % 8 == 0 | (i - 2) % 8 == 0) continue;
    u8g2_DrawPixel(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y) + i);
  }
}
```

#### 注意事项

在编写 `MyHAL` 的过程中，如果需要额外的自定义函数，请直接在 `MyHAL` 类中声明 `protected` 类型的成员函数。

至此，派生 `HAL` 编写完成。

### 第二步：注入派生 `HAL`

在您的引导程序的初始化部分，编写如下代码（笔者假设您已经包含了 `MyHAL.h` ）：

```Cpp
HAL::inject(new MyHAL);
```

至此，您已经可以调用 `HAL` 中对应的静态成员函数了。关于 `HAL` 的程序皆已编写完毕。

下一步，运行 `HAL` 测试程序，测试您的代码是否正确。

### 第三步：运行 `HAL` 测试程序

您可以在注入派生 `HAL` 后运行一段简单的测试程序，来检验您代码的正确性。

```Cpp
HAL::delay(100);
HAL::printInfo("loading...");
HAL::delay(500);
HAL::printInfo("astra UI by dcfsswindy.");
HAL::delay(100);
```

如果屏幕中出现了如下的提示信息，恭喜您！ `HAL` 部署成功。

![CPT2404021245-228x114](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/c4e6e167-0abd-42d9-a5a5-0acfee856c54)

### 测试 `astra UI`

一般情况下， `HAL` 部署完成后，`astra UI` 就已经可以正常工作了。

但还是建议您运行下方的测试程序，用于测试 `astra UI` 是否正常工作（假设您已经正确包含了相关头文件）。

```Cpp
//初始化部分
void setup() {
  astra::Launcher* astraLauncher = new astra::Launcher();
  astra::Menu* rootPage = new astra::Menu("root");

  rootPage->addItem(new astra::Menu("test1"));

  astraLauncher->init(rootPage);

  astra::drawLogo(1000);
}

//主循环部分
void loop() {
  astraLauncher->update();
}
```

若一切正常，您的屏幕将在显示 `astra UI` 开机动画后进入一个只有一个子元素 `test1` 的菜单。

**至此， `astra UI` 部署完毕。**

# English
***或者 [简体中文](#简体中文)***

## Deployment tutorial

### About `HAL`

Special thanks to @Forairaaaaa, I have learned a lot about `monica` based on his open source `monica` .

Before we start the deployment tutorial, I would like to introduce you to the `astra UI` companion `HAL` , the `Hardware Abstraction Layer` .

In `astra UI` , system configuration, screen driver, graphics drawing, I/O port driver and random number generation are all integrated in `HAL` .

The `HAL` ensures that the `astra UI` runs regardless of the hardware platform. **At the same time, you will never see a single line of hardware-related code in any `astra UI` code file.**  Such as `digitalWrite()` or `HAL_GPIOWritePin()` .

For example, when you want to delay the system by 500ms, you don't need to execute `delay(500); (Arduino)` or `HAL_Delay(500); (STM32)` , you just need to execute `HAL::Delay(500);` . The former code cannot be executed on different platforms and is very non-portable, but the latter with `HAL` is hardware platform independent and can be executed on any platform.

This allows `astra UI` to be deployed and run on almost any major hardware platform, you just need to `overload` the corresponding `HAL` functions for your hardware platform.

If you are still a bit confused, due to the limitation of space, I attach part of the prototype code of `HAL` class here:

```Cpp
class HAL {
private:
  static HAL *hal;

public:
  static HAL *get();    //get hal instance.
  static bool check();  //check if there is a hal instance.

  static bool inject(HAL *_hal);  //inject HAL instance and run hal_init.
  static void destroy();  //destroy HAL instance.

  virtual ~HAL() = default;

  virtual std::string type() { return "Base"; }

  virtual void init() {}

protected:
  sys::config config;

public:
  static void canvasUpdate() { get()->_canvasUpdate(); }
  virtual void _canvasUpdate() {}

  static void canvasClear() { get()->_canvasClear(); }
  virtual void _canvasClear() {}

  static void drawPixel(float _x, float _y) { get()->_drawPixel(_x, _y); }
  virtual void _drawPixel(float _x, float _y) {}

  static void drawEnglish(float _x, float _y, const std::string &_text) { get()->_drawEnglish(_x, _y, _text); }
  virtual void _drawEnglish(float _x, float _y, const std::string &_text) {}

  static void drawChinese(float _x, float _y, const std::string &_text) { get()->_drawChinese(_x, _y, _text); }
  virtual void _drawChinese(float _x, float _y, const std::string &_text) {}

  /**
   * @brief system timers.
   */
public:
  static void delay(unsigned long _mill) { get()->_delay(_mill); }
  virtual void _delay(unsigned long _mill) {}

  //...
};
```

As you can see, `HAL` is made up of a number of groups of functions, a static member function and a dummy member function whose name begins with an underscore.

In each group, the static member function's task is to call the `get()` method to get the current instance of `HAL` stored in the `HAL` class (i.e., the derived `HAL` that you write below). You then call the relevant methods in the `HAL` instance.

Here is the implementation of the `inject()` method, which is used to inject the derived `HAL` back into the `HAL` . and stores the derived `HAL` pointer in `HAL` .

```Cpp
bool HAL::inject(HAL *_hal) {
  if (_hal == nullptr) {
    return false;
  }

  _hal->init();
  hal = _hal;
  return true;
}
```

So it's easy to see that there are three things you need to do:

1. Inherit this `HAL` class and write your own hardware platform derived `HAL`
2. rewrite all the virtual functions you need to use in the derived `HAL`
3. inject your derived `HAL` back into `HAL` via `inject()` method

In the following, I will take you step by step to deploy `astra UI` to your hardware platform with more detailed description.

### The first step: writing the derived `HAL`

#### Writing the Framework

First you need to include the `hal.h` header file in your `.h` file.

Inherit and write a derived `HAL`, and give your derived `HAL` a name, here `MyHAL` for example, hereinafter all use `MyHAL` to refer to the derived `HAL` .

Incidentally, write a default constructor. ^_^

```Cpp
//MyHAL.h

#pragma once
#ifndef MYHAL_H_
#define MYHAL_H_

#include "hal.h"

class MyHAL : public HAL {
public:
  MyHAL() = default;
};

#endif
```

#### Write the `_xx_init()` method of `MyHAL` and override the `init` method

This step varies from person to person, depending on your hardware platform and the various devices you have access to.

The `init()` method is automatically called when you call the `HAL::inject()` method.

The `_xx_init()` method exists only in your derived `HAL` and should be called in the `init()` method of your derived class.

It is used to initialize all your personalization devices.

Please note that if you need to include a graphics library (and I recommend you do), please also include the `.h` file for your graphics library in `MyHAL.h` and write the `xx_init()` method for it.

The following code is an example of the `STM32` and `u8g2` graphics libraries that I use:

```Cpp
//MyHAL.h

class MyHAL : public HAL {
private:
  void _stm32_hal_init();
  void _sys_clock_init();
  void _gpio_init();
  void _dma_init();
  void _timer_init();
  void _spi_init();
  void _adc_init();

  void _ssd1306_init();
  void _key_init();
  void _buzzer_init();
  void _u8g2_init();

public:
  inline void init() override {
    _stm32_hal_init();
    _sys_clock_init();
    _gpio_init();
    _dma_init();
    _timer_init();
    _spi_init();
    _adc_init();

    _ssd1306_init();
    _key_init();
    _buzzer_init();
    _u8g2_init();
  }
};
```

The implementation of some of the above functions is given again for your reference:

```Cpp
//MyHAL_STM32.cpp

void MyHAL::_gpio_init() {
  MX_GPIO_Init();
}

void MyHAL::_adc_init() {
  MX_ADC1_Init();
}

void MyHAL::_spi_init() {
  MX_SPI2_Init();
}
```

Please note that the implementation of these methods needs to be written in the `MyHAL.cpp` file, but of course, I recommend that you put different kinds of devices in different `.cpp` files.

Of course, I recommend you to put different kinds of devices in different `.cpp` files, such as `MyHAL_key.cpp` , `MyHAL_oled.cpp` and `MyHAL_STM32.cpp` .

#### Rewriting other methods

Next comes the heavy lifting.

Depending on your graphics library of choice (or without the aid of a graphics library), You need to **override** draw pixels, draw lines, draw rectangles, and a host of other methods.

You need to **override** the methods for refreshing the canvas, clearing the screen, etc., depending on your screen.

You need to **override** a series of methods such as delay, get boot time, generate true random number, etc. according to your hardware platform.

According to your peripheral type, you need to rewrite a series of methods such as buzzer sound, key scanning and so on.

Of course, if you are clear about your needs, you can choose not to rewrite some methods and use the default method implementations in the `HAL` parent class.

For example, if you don't need to generate true random numbers, you don't need to override the `_getRandomSeed()` method, or just have it return a fixed value.

For example, if you don't need or have a buzzer, you don't need to rewrite `_beep()` and its set of methods.

For reference, here are all the functions that I have rewritten. For a complete list of functions that can be rewritten, see `hal.h` .

```Cpp
//MyHAL.h

class MyHAL : public HAL {
protected:
  u8g2_t canvasBuffer {};

public:
  void _screenOn() override;
  void _screenOff() override;

public:
  void* _getCanvasBuffer() override;
  uint8_t _getBufferTileHeight() override;
  uint8_t _getBufferTileWidth() override;
  void _canvasUpdate() override;
  void _canvasClear() override;
  void _setFont(const uint8_t * _font) override;
  uint8_t _getFontWidth(std::string& _text) override;
  uint8_t _getFontHeight() override;
  void _setDrawType(uint8_t _type) override;
  void _drawPixel(float _x, float _y) override;
  void _drawEnglish(float _x, float _y, const std::string& _text) override;
  void _drawChinese(float _x, float _y, const std::string& _text) override;
  void _drawVDottedLine(float _x, float _y, float _h) override;
  void _drawHDottedLine(float _x, float _y, float _l) override;
  void _drawVLine(float _x, float _y, float _h) override;
  void _drawHLine(float _x, float _y, float _l) override;
  void _drawBMP(float _x, float _y, float _w, float _h, const uint8_t* _bitMap) override;
  void _drawBox(float _x, float _y, float _w, float _h) override;
  void _drawRBox(float _x, float _y, float _w, float _h, float _r) override;
  void _drawFrame(float _x, float _y, float _w, float _h) override;
  void _drawRFrame(float _x, float _y, float _w, float _h, float _r) override;

public:
  void _delay(unsigned long _mill) override;
  unsigned long _millis() override;
  unsigned long _getTick() override;
  unsigned long _getRandomSeed() override;

public:
  void _beep(float _freq) override;
  void _beepStop() override;
  void _setBeepVol(uint8_t _vol) override;

public:
  bool _getKey(key::KEY_INDEX _keyIndex) override;

public:
  void _updateConfig() override;
};
```

Meanwhile, a rewritten function implementation code is given below as an example of the `STM32` and `u8g2` graphic libraries used by the author:

```Cpp
//MyHAL_oled.cpp

void HALDreamCore::_drawPixel(float _x, float _y) {
  u8g2_DrawPixel(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y));
}

void HALDreamCore::_drawChinese(float _x, float _y, const std::string &_text) {
  u8g2_DrawUTF8(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y), _text.c_str());
}

void HALDreamCore::_drawVDottedLine(float _x, float _y, float _h) {
  for (uint8_t i = 0; i < (uint8_t)std::round(_h); i++) {
    if (i % 8 == 0 | (i - 1) % 8 == 0 | (i - 2) % 8 == 0) continue;
    u8g2_DrawPixel(&canvasBuffer, (int16_t)std::round(_x), (int16_t)std::round(_y) + i);
  }
}
```

#### Notes

In the process of writing `MyHAL` , if you need extra customized functions, please declare `protected` type member functions in `MyHAL` class directly.

At this point, the derivation of `HAL` is complete.

### Step 2: Injecting the derived `HAL`

In the initialization section of your bootloader, write the following code (I assume you have included `MyHAL.h` ):

```Cpp
HAL::inject(new MyHAL);
```

At this point, you are ready to call the corresponding static member functions of `HAL`. The program for `HAL` has been written.

Next, run the `HAL` test program to see if your code is correct.

### Step 3: Run the `HAL` test program

You can run a simple test program after injecting the derived `HAL` to check the correctness of your code.

```Cpp
HAL::delay(100);
HAL::printInfo("loading...");
HAL::delay(500);
HAL::printInfo("astra UI by dcfsswindy.");
HAL::delay(100);
```

If the following message appears in the screen, congratulations! `HAL` deployment was successful.

![CPT2404021245-228x114](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/c4e6e167-0abd-42d9-a5a5-0acfee856c54)

### Testing `astra UI`

Normally, `astra UI` is ready to work once `HAL` has been deployed.

However, it is recommended that you run the test program below to test that `astra UI` is working (assuming you have included the headers correctly).

```Cpp
void setup() {
  astra::Launcher* astraLauncher = new astra::Launcher();
  astra::Menu* rootPage = new astra::Menu("root");

  rootPage->addItem(new astra::Menu("test1"));

  astraLauncher->init(rootPage);

  astra::drawLogo(1000);
}

void loop() {
  astraLauncher->update();
}
```

If all is well, your screen will enter a menu with only one child element, `test1`, after displaying the `astra UI` startup animation.

**This completes the deployment of `astra UI` . **

