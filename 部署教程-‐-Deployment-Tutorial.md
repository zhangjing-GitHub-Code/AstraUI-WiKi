# 简体中文 
***or [English](#English)***

## 部署教程

### 关于 `HAL`

特别感谢 @Forairaaaaa ，笔者基于其开源的 `monica` ，学习到了很多相关知识。

在正式开始部署教程前，笔者需要先向您介绍 `astra UI` 配套的 `HAL` ，即 `硬件抽象层` 。

在 `astra UI` 中，系统配置、屏幕驱动、图形绘制、I/O口驱动和生成随机数都集成在了 `HAL` 中。

`HAL` 保证了 `astra UI` 的运行不受硬件平台的限制。**同时，您绝对不会在任何 `astra UI` 的代码文件中看到任何一行与硬件平台有关的代码。**如 `digitalWrite()` 或 `HAL_GPIOWritePin()` 等。

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
class MyHAL : public HAL {
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

#### 注意事项

在编写 `MyHAL` 的过程中，如果需要额外的自定义函数，请直接在 `MyHAL` 类中声明 `protected` 类型的成员函数。

# English
***或者 [简体中文](#简体中文)***