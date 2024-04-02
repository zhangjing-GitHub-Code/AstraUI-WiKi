# 简体中文 
***or [English](#English)***

## 关于代码

您可以直接 `git clone` 或者通过本仓库 `Code` 分页内 `Download Zip` 下载笔者的代码。

您可以参考这份代码作为自己部署 `astra UI` 的依据。这是一份非常不错的资料，有助于您理解 `astra UI` 的逻辑。

但请注意，当且仅当您的硬件配置与笔者完全相同时，才可以直接刷入笔者编译好的固件，否则可能会产生意想不到的后果。

下面，笔者将会介绍代码配套的硬件平台。

## 笔者的硬件配置

+ MCU: STM32F103CBT6
+ Key: 
	+ Key 0 -> PA6
	+ Key 1 -> PA7
+ OLED 
	+ IC型号: SSD1306
	+ 分辨率: 128 * 64
		+ 水平 128 Pixels 
		+ 垂直 64 Pixels
	+ Hardware SPI + DMA: 
		+ SCK -> PB13
		+ MOSI -> PB15
		+ CS -> PA2
		+ RST -> PA3

---

# English
***或者 [简体中文](#简体中文)***

## About the code

You can download the author's code via `git clone` or `Download Zip` in the `Code` section of this repository.

You can refer to this code as the basis for your own deployment of `astra UI` . This is a very good resource to help you understand the logic of `astra UI` .

However, please note that if and only if your hardware configuration is exactly the same as the author, you can directly flash the firmware compiled by the author, otherwise there may be unexpected consequences.

In the following, I will introduce the code matching hardware platform.

## Author's hardware configuration

+ MCU: STM32F103CBT6
+ Key. 
	+ Key 0 -> PA6
	+ Key 1 -> PA7
+ OLED 
	+ IC Model: SSD1306
	+ Resolution: 128 * 64
		+ Horizontal 128 Pixels 
		+ Vertical 64 Pixels
	+ Hardware SPI + DMA. 
		+ SCK -> PB13
		+ MOSI -> PB15
		+ CS -> PA2
		+ RST -> PA3