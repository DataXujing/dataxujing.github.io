---
layout: post
title: "RISE-Python也可以做漂亮的Slides"
img: hotsm.jpg 
date: 2018-1-1 9:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python,RISE]
---

**徐静**


工作日的第一天，有了有个新发现，感觉比较有意思分享给大家。以前做slide都是用ppt,latex beamer,R语言，H5等工具，今天发现用jupyter也OK。今天主要和大家分享用Python的jupyter做slides。

RISE是基于Reveal.js的一个制作slides的python模块，我们可以用jupyter的交互编辑器书写分析报告并快速的共享部署，我一直认为jupyter像极了Rmarkdown。今天的分享主要包括，RISE的安装，RISE的使用及RISE的Demo。不要担心，该模块非常非常的简单，易学，基本5分钟精通上手。

## RISE的安装

1 - Using conda (recommended):

`conda install -c damianavila82 rise`

2 - Using pip (less recommended):

`pip install RISE`

and then two more steps to install the JS and CSS in the proper places:

`jupyter-nbextension install rise --py --sys-prefix`

and enable the nbextension:

`jupyter-nbextension enable rise --py --sys-prefix`

在终端下执行以上几步就可完成RISE的安装。


## RISE的使用

使用RISE与jupyter相同，只不过需要做一些手动配置就可以完成相关slides的展示，我们主要看一下相关配置，RISE主要有两种配置方式，一种是使用Python配置，一种是使用jupyter notebook matedata配置，两种配置都比较easy。

**1.使用python**


```pyton
from traitlets.config.manager import BaseJSONConfigManager
path = "/home/damian/miniconda3/envs/
        rise_latest/etc/jupyter/nbconfig"
cm = BaseJSONConfigManager(config_dir=path)
cm.update("livereveal", {
              "theme": "sky",
              "transition": "zoom",
              "start_slideshow_at": "selected",
})
```

**2.使用jupyter notebook matedata**

我们可以使用reveal.js配置，在我们的notebook metadata (Edit->Edit Notebook Metadata)，就像这样：

```python
{
    ...
    "livereveal": {
        "theme": "serif",
        "transition": "zoom",
        ...
    },
    ...
}
```

PISE提供了很多配置选项，具体可以参照其官方的说明文档

+ theme (Choosing a theme)

+ transition (Choosing a transition)

+ tart_slideshow_at (Choosing where the slideshow begins)

+ width and height (Change the width and height of slides)

+ autolaunch (Automatically launch RISE)

+ auto_select and auto_select_fragment (Select cells based on the current slide)

+ scroll (Enable a right scroll bar)

+ backimage, header, footer, overlay (Add overlay, header, footer and background images)

+ leap_motion (Usage with Leap Motion)

具体的配置内容可以参考官方的说明文档，比较简单。

## Slide Demo



![]({{site.url}}/assets/bowen23/o1.gif){:height="100%" width="90%"}


![]({{site.url}}/assets/bowen23/o2.gif){:height="100%" width="90%"}


![]({{site.url}}/assets/bowen23/o3.gif){:height="100%" width="90%"}



+ [https://github.com/damianavila/RISE/](https://github.com/damianavila/RISE/)

+ [https://damianavila.github.io/RISE/customize.html](https://github.com/damianavila/RISE/)
