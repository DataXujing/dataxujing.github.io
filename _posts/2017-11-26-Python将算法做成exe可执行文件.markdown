---
layout: post
title: "Python将算法转化成可执行文件"
img: weiexueshan.jpg # Add image post (optional)
date: 2017-11-26 9:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python, Pyinstaller]
---
**徐静**

Python打包成可执行文件方案有很多种，我们主要使用pyinstaller把上一次的爬虫封装成可执行文件。
PyInstaller可以用来打包python应用程序，打包完的程序就可以在没有安装Python解释器的机器上运行了。PyInstaller支持Python 2.7和Python 3.3+。可以在Windows、Mac OS X和Linux上使用，但是并不是跨平台的，而是说你要是希望打包成.exe文件，需要在Windows系统上运行PyInstaller进行打包工作；打包成mac app，需要在Mac OS上使用，在这里我们权且称之为半跨平台。

![]({{ site.url }} /assets/bowen14/output_2.png)


### 安装

Windows上运行PyInstaller还需要PyWin32或者pypiwin32，其中pypiwin32在你安装PyInstaller的时候会自动安装。

`pip install pyinstaller`

`pip install --upgrade pyinstaller`

当然，也可以通过下载源代码编译方式安装：

`python setup.py install`

安装完成后，使用如下命令可以确认版本号：

`pyinstaller --version`

```
pyinstaller
usage: pyinstaller-script.py [-h] [-v] [-D] [-F] [--specpath DIR] [-n NAME]
                             [-p DIR] [--hidden-import MODULENAME]
                             [--additional-hooks-dir HOOKSPATH]
                             [--runtime-hook RUNTIME_HOOKS]
                             [--exclude-module EXCLUDES] [--key KEY] [-d] [-s]
                             [--noupx] [-c] [-w]
                             [-i <FILE.ico or FILE.exe,ID or FILE.icns>]
                             [--version-file FILE] [-m <FILE or XML>]
                             [-r RESOURCE] [--uac-admin] [--uac-uiaccess]
                             [--win-private-assemblies]
                             [--win-no-prefer-redirects]
                             [--osx-bundle-identifier BUNDLE_IDENTIFIER]
                             [--distpath DIR] [--workpath WORKPATH] [-y]
                             [--upx-dir UPX_DIR] [-a] [--clean]
                             [--log-level LEVEL] [--upx UPX]
                             scriptname [scriptname ...]
pyinstaller-script.py: error: the following arguments are required: scriptname
```

可以看到PyInstaller的信息，说明安装完成，可以使用了；详细帮助可以pyinstaller -h查看。
正常安装之后就可以使用了。

关于pyinstaller的使用教程可参见官网教程及相关优秀的网络资源教程。

### 爬虫程序封装成exe可执行文件

把上次的爬虫程序做成exe可执行程序，中途走了不少弯路，把python爬虫源码拷贝至文件夹E:/appexe并把chromedriver拷贝至该文件夹，

`cd /d E:/appexe`

`pyinstaller -F --icon-icon.ico spyder.py`

如果该过程无错误正常运行，则在E:/appexe文件夹下会出现：spyder.spec,build(临时文件最后可删除)，dist文件，dist文件中有封装好的可执行文件，如下图所示：


![]({{ site.url }} /assets/bowen14/output_3.png){:height="50%" width="90%"}


![]({{ site.url }} /assets/bowen14/output_4.png){:height="50%" width="90%"}


单击dist文件下，spyder.exe即可脱离python环境镜像模拟登录爬虫，非常炫酷。

踩过的坑：

+ 开始时代码读入数据使用pandas模块，该模块是C++缩写，pyistaller需做相关配置才可正常使用该模块，为了快速实现exe可执行文件把pandas模块替换成了xlrd模块。

+ chromedriver必须放在E:/appexe文件下selenuim才可正常使用。

+ selenuim在shell下可正常运行，但在pynstaller做成exe后，运行exe发现编码有问题，后来经过不断尝试，最后正常。

+ 如果exe闪退可在代码最后添加 `raw_input()`后按任意键退出。

+ 不同系统须在不同系统下pyinstaller，成功后可拷入其他电脑测试脱离python环境后的效果

这样我们就可以正常把我们的模拟登录爬虫代码写成exe可执行文件，dist下点开spyder.exe按要求可以在脱离Python环境下进行网络爬虫，放心交付給业务部门。


### 参考文献

【1】 [pyinstaller手册](http://blog.csdn.net/mr__fang/article/details/7176731)

【2】 [PYINSTALLER打包PYTHON脚本的一些心得](https://zhengzexin.com/2016/11/08/pyinstaller-da-bao-python-jiao-ben-de-yi-xie-xin-de/)

【3】[pyinstaller简洁教程](http://legendtkl.com/2015/11/06/pyinstaller/)

【4】[Creating an Executable from a Python Script](https://mborgerson.com/creating-an-executable-from-a-python-script/)

【5】[Add openpyxl hook](https://github.com/pyinstaller/pyinstaller/pull/2066)

【6】[pyinstaller官方教程](http://pythonhosted.org/PyInstaller/)

【7】[COMPILING PYTHON USING PYINSTALLER](https://metac0rtex.com/compiling-python-using-pyinstaller/)
