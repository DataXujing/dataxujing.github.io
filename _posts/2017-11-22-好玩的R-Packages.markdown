---
layout: post
title: "三个好玩的R包"
img: qiao.jpg # Add image post (optional)
date: 2017-11-22 9:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [R]
---
**徐静**

最近在逛知乎的时候，无意间发现三个比较好玩的R包，这三个包都是很早出来的，顿觉自己对R的了解甚浅。

## 1.花式自动称赞包

看到这个包时我笑了，于是看着文档玩了好一会。。。。。。
这个包叫：praise,可以直接在CRAN上安装。知乎上的大牛说，这个包的功能就是**赞你**，**赞你**，**赞你**重要的事情说出三遍。

这个包很简单，但让我玩了好一会（这篇文章不会被和谐掉吧）

![]({{site.url}}/assets/bowen12/output_0.png)

+ 直接赞

![]({{site.url}}/assets/bowen12/output_1.png)

+ 自定义赞

![]({{site.url}}/assets/bowen12/output_2.png)

其中 `${EXCLAMATION}` 以及 `${adverb_manner }`你可以理解为包中的词库，分别表示感叹和情态，每个词库里面都含有数量不等的用于称赞你的词语。

具体词库见可通过names()查看：

![]({{site.url}}/assets/bowen12/output_3.png)

+ 设定为打开程序，花式自动赞   

R语言当中，我们是可以自定义我们的启动环境。R在启动时，会到R_Home\etc目录下找Rprofile.site文件进行加载（其中R_Home指的是我们R的安装路径，例如徐静的目录：D:\R\R-3.4.0\etc）。在这个文件里，设置的内容包括默认编辑器，CRAN镜像选取，自动加载包等等，要实现我们的花式自动赞，只需要打开Rprofile.site文件，在最后加上如下代码：

```r
.First <- function(){  
    library(praise)  
    cat(praise("${EXCLAMATION}! ${EXCLAMATION}! 徐静相信自己你一定能成为大牛 
    ${adverb_manner}!"),"\n",praise("相信自己，相信自己"),
    "\n",praise("重要的事情说三遍"),"\n",date(),"\n")
}
```
Wow,被加油打气了,你可以按照自己的需求替换。

![]({{site.url}}/assets/bowen12/output_4.png)



## 2.fun 包

这是一个R游戏和其他有趣的东西的集合，是大牛Yihui Xie, Taiyun Wei and Yixuan Qiu写的，GitHub的项目地址为：
https://github.com/yihui/fun

+ demo(ChinaHeart2D)

![]({{site.url}}/assets/bowen12/output_5.png){:height="50%" width="90%"}

+ demo(ChinaHeart3D)

![]({{site.url}}/assets/bowen12/output_6.png){:height="50%" width="90%"}

+ 还有其他奇葩的demo()绘图，在这里不演示了

+ 扫雷游戏

```r

library(fun)
if (.Platform$OS.type == "windows") x11() else x11(type = "Xlib")
mine_sweeper()

```

![]({{site.url}}/assets/bowen12/output_7.png)

+ 五子棋

```r
library(fun)
gomoku()
```
![]({{site.url}}/assets/bowen12/output_8.png)

+  light out 游戏

```r
library(fun)
if (.Platform$OS.type == "windows") x11() else x11(type = "Xlib")
lights_out()
```
![]({{site.url}}/assets/bowen12/output_9.png)

+ 还有一些其他的小函数

+ 记住不要运行：shutdown()函数。。。。。。


## 3.数独 sudoku

```r
library(sudoku)
playSudoku() #玩一个random的数独游戏

```

最近也看到有些大牛用R做游戏，R语言又多了一项技能--娱乐。
