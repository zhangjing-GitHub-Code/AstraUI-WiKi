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
```

#### 编写 `MyHAL` 的 `_xx_init()` 和 `init` 方法


# English
***或者 [简体中文](#简体中文)***