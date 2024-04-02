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

若您还是有些迷惑，笔者在这里附上 `HAL` 类的原型代码：

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
  static void *getCanvasBuffer() { return get()->_getCanvasBuffer(); }

  virtual void *_getCanvasBuffer() { return nullptr; }
  static uint8_t getBufferTileHeight() { return get()->_getBufferTileHeight(); }
  virtual uint8_t _getBufferTileHeight() { return 0; }
  static uint8_t getBufferTileWidth() { return get()->_getBufferTileWidth(); }
  virtual uint8_t _getBufferTileWidth() { return 0; }
  static void canvasUpdate() { get()->_canvasUpdate(); }
  virtual void _canvasUpdate() {}
  static void canvasClear() { get()->_canvasClear(); }
  virtual void _canvasClear() {}
  static void setFont(const uint8_t *_font) { get()->_setFont(_font); }
  virtual void _setFont(const uint8_t *_font) {}
  static uint8_t getFontWidth(std::string &_text) { return get()->_getFontWidth(_text); }
  virtual uint8_t _getFontWidth(std::string &_text) { return 0; }
  static uint8_t getFontHeight() { return get()->_getFontHeight(); }
  virtual uint8_t _getFontHeight() { return 0; }
  static void setDrawType(uint8_t _type) { get()->_setDrawType(_type); }
  virtual void _setDrawType(uint8_t _type) {}
  static void drawPixel(float _x, float _y) { get()->_drawPixel(_x, _y); }
  virtual void _drawPixel(float _x, float _y) {}
  static void drawEnglish(float _x, float _y, const std::string &_text) { get()->_drawEnglish(_x, _y, _text); }
  virtual void _drawEnglish(float _x, float _y, const std::string &_text) {}
  static void drawChinese(float _x, float _y, const std::string &_text) { get()->_drawChinese(_x, _y, _text); }
  virtual void _drawChinese(float _x, float _y, const std::string &_text) {}
  static void drawVDottedLine(float _x, float _y, float _h) { get()->_drawVDottedLine(_x, _y, _h); }
  virtual void _drawVDottedLine(float _x, float _y, float _h) {}
  static void drawHDottedLine(float _x, float _y, float _l) { get()->_drawHDottedLine(_x, _y, _l); }
  virtual void _drawHDottedLine(float _x, float _y, float _l) {}
  static void drawVLine(float _x, float _y, float _h) { get()->_drawVLine(_x, _y, _h); }
  virtual void _drawVLine(float _x, float _y, float _h) {}
  static void drawHLine(float _x, float _y, float _l) { get()->_drawHLine(_x, _y, _l); }
  virtual void _drawHLine(float _x, float _y, float _l) {}
  static void drawBMP(float _x, float _y, float _w, float _h, const uint8_t *_bitMap) {
    get()->_drawBMP(_x,
                    _y,
                    _w,
                    _h,
                    _bitMap);
  }

  virtual void _drawBMP(float _x, float _y, float _w, float _h, const uint8_t *_bitMap) {}
  static void drawBox(float _x, float _y, float _w, float _h) { get()->_drawBox(_x, _y, _w, _h); }
  virtual void _drawBox(float _x, float _y, float _w, float _h) {}
  static void drawRBox(float _x, float _y, float _w, float _h, float _r) {
    get()->_drawRBox(_x,
                     _y,
                     _w,
                     _h,
                     _r);
  }
  virtual void _drawRBox(float _x, float _y, float _w, float _h, float _r) {}
  static void drawFrame(float _x, float _y, float _w, float _h) { get()->_drawFrame(_x, _y, _w, _h); }
  virtual void _drawFrame(float _x, float _y, float _w, float _h) {}
  static void drawRFrame(float _x, float _y, float _w, float _h, float _r) {
    get()->_drawRFrame(_x,
                       _y,
                       _w,
                       _h,
                       _r);
  }

  virtual void _drawRFrame(float _x, float _y, float _w, float _h, float _r) {}
  static void printInfo(std::string _msg) { get()->_printInfo(std::move(_msg)); }
  virtual void _printInfo(std::string _msg);

  /**
   * @brief system timers.
   */
public:
  static void delay(unsigned long _mill) { get()->_delay(_mill); }
  virtual void _delay(unsigned long _mill) {}
  static unsigned long millis() { return get()->_millis(); }
  virtual unsigned long _millis() { return 0; }
  static unsigned long getTick() { return get()->_getTick(); }
  virtual unsigned long _getTick() { return 0; }
  static unsigned long getRandomSeed() { return get()->_getRandomSeed(); }

  /**optional**/
  virtual unsigned long _getRandomSeed() { return 0; }

  /**
   * @brief buzzer.
   * */
public:
  static void beep(float _freq) { get()->_beep(_freq); }
  virtual void _beep(float _freq) {}
  static void beepStop() { get()->_beepStop(); }
  virtual void _beepStop() {}
  static void setBeepVol(uint8_t _vol) { get()->_setBeepVol(_vol); }
  virtual void _setBeepVol(uint8_t _vol) {}
  static void screenOn() { get()->_screenOn(); }
  virtual void _screenOn() {}
  static void screenOff() { get()->_screenOff(); }
  virtual void _screenOff() {}

  /**
   * @brief key.
   */
public:
  static bool getKey(key::KEY_INDEX _keyIndex) { return get()->_getKey(_keyIndex); }
  virtual bool _getKey(key::KEY_INDEX _keyIndex) { return false; }
  static bool getAnyKey() { return get()->_getAnyKey(); }
  virtual bool _getAnyKey();

protected:
  key::keyAction key[key::KEY_NUM] = {static_cast<key::keyAction>(0)};

public:
  static void keyScan() { get()->_keyScan(); }
  virtual void _keyScan();
  static void keyTest() { return get()->_keyTest(); }
  virtual void _keyTest();

  /**
   * @brief system config.
   */
public:
  static sys::config &getSystemConfig() { return get()->config; }
  static void setSystemConfig(sys::config _cfg) { get()->config = _cfg; }
  static void updateConfig() { get()->_updateConfig(); }
  virtual void _updateConfig() {}
};
```

# English
***或者 [简体中文](#简体中文)***