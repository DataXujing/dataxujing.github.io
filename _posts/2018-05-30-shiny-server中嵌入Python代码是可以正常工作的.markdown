---
layout: post
title: "shiny server与Python的结合"
img: bowen43.jpg
date: 2018-05-30 12:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [R,shiny server]
---

5月份的时候在公司做了一个智能分配的小项目，底层的算法是推荐系统，配合业务配置项。为了展示效果和测试线上结果，做了个shiny,然而我的算法是使用Python开发的，使用我在博客中介绍的reticulate包在线下是能够实现的，然而当我部署在服务器上的时候，我发现问题来了，shiny server根本无法解析我的Python代码，原因在哪呢？

我们知道在Python中我们用Django开发一些Web应用时是要做一个Python的虚拟环境的，在这个虚拟环境中我会安装所有这个Django应用的模块。而在部署时我会告诉Django你要在哪个Python环境中解析你的代码。实际上reticulate说的很清楚，我们通过use_virtualenv()函数指定R代码在解析Python代码时使用的环境，好了到这里这个问题就解决了。


Python中有个比较牛逼的模块叫virtualenv，我们可以通过pip全装，他是干什么的呢？　virtualenv 是一个创建隔绝的Python环境的工具。virtualenv创建一个包含所有必要的可执行文件的文件夹，用来使用Python工程所需的包。我么只需要基本的使用就可以创建一个virtualenv Python环境，然后在这个环境中安装必要的模块，把这个虚拟环境拷贝我们的shiny项目中，我们指定R解释Python的环境是该环境，问题就解决了。

我们需要怎么做呢？

<pre>
$ cd my_project
$ virtualenv my_shiny_py　　#muy_shiny_py为虚拟环境目录名，目录名自定义
</pre>

virtualenv my_shiny_py 将会在当前的目录中创建一个文件夹，包含了Python可执行文件，以及 pip 库的一份拷贝，这样就能安装其他包了。虚拟环境的名字（此例中是 my_shiny_py ）可以是任意的；若省略名字将会把文件均放在当前目录。

在任何你运行命令的目录中，这会创建Python的拷贝，并将之放在叫做 my_shiny_py 的文件中。

你可以选择使用一个Python解释器：

<pre>
$ virtualenv -p /usr/bin/python3.5 my_shiny_py　　　　# -p参数指定Python解释器程序路径
</pre>

这将会使用 /usr/bin/python3.5 中的Python解释器。


要开始使用虚拟环境，其需要被激活：
$ source my_shiny_py/bin/activate　　　
从现在起，任何你使用pip安装的包将会放在 my_shiny_py 文件夹中，与全局安装的Python隔绝开。

像平常一样安装包，比如：
<pre>
$ pip install numpy
</pre>

如果你在虚拟环境中暂时完成了工作，则可以停用它：
<pre>
$ . my_shiny_py/bin/deactivate
</pre>
这将会回到系统默认的Python解释器，包括已安装的库也会回到默认的。

要删除一个虚拟环境，只需删除它的文件夹。（执行 rm -rf my_shiny_py ）。

这里virtualenv 有些不便，因为virtual的启动、停止脚本都在特定文件夹，可能一段时间后，你可能会有很多个虚拟环境散落在系统各处，你可能忘记它们的名字或者位置。可以使用virtualenvwrapper，鉴于virtualenv不便于对虚拟环境集中管理，所以推荐直接使用virtualenvwrapper。 virtualenvwrapper提供了一系列命令使得和虚拟环境工作变得便利。它把你所有的虚拟环境都放在一个地方。（这里不是我们的重点）


那么R代码要做什么呢？

实际上加上一句就OK了

<pre>

library(reticulate)
use_virtualenv("my_shiny_py")

</pre>

在未来感觉R和Python会走的更近，之间的相互调用会更方便。
