Welcome to the oled-ui-astra wiki!

![image](https://github.com/dcfsswindy/oled-ui-astra/assets/59963050/159bd518-976e-4fa5-bb5c-66452e0d698a)

# 序言 - Preamble
最初产生创建这个项目的想法，是三个月前。当时受到稚晖君 `Mono UI` 和 b站@音游玩的人 的 `Wouo UI` 的启发，我决定做一个属于自己的、基于 `Cpp` 的、以更科学的方式组织的OLED多级菜单UI。从这个想法诞生时到现在，`astra UI` 的仓库进行了五十余次Commit，我也为此奋战了无数个日夜。当然，最终的结果是好的，我也如约将我的心血结晶开源出来，供更多的人学习使用。

我目前还是一名在校大学生，学习成绩算不上优秀，其实也可以说是比较烂。能支持我走到现在的，靠的是在我心底里这些有大有小的爱好。我喜欢编程、喜欢琢磨电子、喜欢画电路板、喜欢编曲、喜欢弹琴也喜欢摄影。我也经常感到迷茫，迷茫我的人际关系；迷茫我那并不好看的学习成绩；迷茫我的未来人生路途又或者是迷茫我内心的答案。但我相信，这些迷茫都会在未来被回答，有可能是由我亲自来回答，当然也有可能是某个贵人替我回答。当然，我还是希望由我自己主导我的人生旅程。

说了这么多，略显矫情。只是想记录一下这个项目开发到这样一个相对比较完善的阶段时，我的内心感受。写 `astra UI` 真的让我学到了很多，感谢它，这是我目前做过**最牛b**的项目。

谢谢大家的支持，愿你们玩得开心，有疑问的话，可以参考本Wiki，或者提交 `Issues`，等待你的来信。

dcfsswindy 2024.03.31 10:14 

于 北京

---
The idea of creating this project first came to me three months ago. Inspired by PengZhiHui's `Mono UI` and bilibili@音游玩的人's `Wouo UI`, I decided to make my own `Cpp` based OLED multilevel menu UI which is organized in a more scientific way. from the birth of this idea to now, the repository of `astra UI` has been committed for more than 50 times, and I have been working on it for countless days and nights. Of course, the end result is good, and I have open-sourced my brainchild as promised, so that more people can learn and use it.

I am still a college student, academic performance can not be considered excellent, in fact, can also be said to be relatively bad. Can support me to go to the present, relying on these big and small hobbies in my heart. I like programming, I like to think about electronics, I like to draw circuit boards, I like to arrange music, I like to play the piano and I like photography. I'm often confused about my relationships, my not-so-good academic performance, my future path in life, or my inner answers. But I believe that all these confusions will be answered in the future, possibly by me personally, or of course, by someone precious for me. Of course, I still want to lead my life's journey by myself.

Having said all this, it's slightly pretentious. I just want to record how I feel when this project has reached such a relatively advanced stage of development. Writing `astra UI` has really taught me a lot, and thanks to it, this is the **best** project I've done so far.

Thank you all for your support, may you have fun, and if you have any questions, please refer to this Wiki, or submit `Issues`, waiting for your letter.

dcfsswindy. 2024.03.31 10:14

Beijing

---
# Wiki 目录 - Wiki Contents
+ `astra UI` 介绍 - `astra UI` Introduction
	+ 简介 - Synopsis
	+ 硬件要求 - Hardware Requirements
	+ 文件结构 - File Structure
	+ 关于 `Item` - About `Item`
		+ 基本概念 - Basic Concept
		+ `Menu`
		+ `Widget` *(future)*
		+ `Camera`
		+ `Selector`
		+ `Launcher`
	+ 接口 - `APIs`
		+ `Menu`
		+ `Widget` *(future)*
		+ `Camera`
		+ `Selector`
		+ `Launcher`
	+ 关于 `HAL` - About `HAL`
+ **部署教程 - Deployment Tutorial**
	+ 编写派生 `HAL` - Write Derived `HAL`
		+ 继承 `HAL` 类 - Inherit the `HAL` Class
		+ 链接图形库 *（可选的）* - Link Graphics Library *(optional)*
		+ 编写 `_xx_init()` 方法 - Write `_xx_init()` Method
		+ 重写 `init()` - 方法 Override `init()` Method
		+ 重写其他方法 - Override Other Method
		+ 编写其他方法 *（可选的）* - Write Other Method *(optional)*
	+ 注入派生 `HAL` - Inject Derived `HAL`
	+ 运行 `HAL` 测试程序 - Run the HAL Test Program
	+ Have Fun! 
+ 例程 - Example 
+ 更新计划 - Update Program
+ 更新日志 - Update Log
+ FAQ