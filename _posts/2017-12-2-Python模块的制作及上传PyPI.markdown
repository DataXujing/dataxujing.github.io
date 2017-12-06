---
layout: post
title: "用Python自己造轮子--Python模块封装及上传PyPI"
img: titian.jpg # Add image post (optional)
date: 2017-12-01 9:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python]
---
**徐静**

之前开发过六七个R语言的packages并且已经应用到业务中（第一个R包的开发是因为自己的一篇论文，想造个轮子供别人使用），感觉R语言的package的书写中规中矩，详细的包部分托管在我的github上，感兴趣的读者可以去clone。最近打算在业务系统中植入一些其他的算法，想着用Python去实现，算法封装成模块，运维在配置服务器时直接在网上pip我的算法就OK了，所以花了些许时间做了一些实验。下面把我的实验过程及结果展示给大家，以后就可以按照这个步骤‘造轮子’了（自行车轮子）。

首先我强调的是任何Python程序都可以当做模块引入，也就是都可以import myfunc,前提是你要指定你代码存放的路径，告诉Python解释器要在哪里引入你的代码，Python默认会在安装Python模块的系统路径去引入，可以查看这些路径并添加

```python
import sys
>>> sys.path
['', 'E:\\Project\\Python\\ModuleAndPackage', 'C:\\Windows\\system32\\python27.zip', 
'C:\\Python\\DLLs', 'C:\\Python\\lib', 'C:\\Python\\lib\\plat-win', 
'C:\\Python\\lib\\lib-tk', 'C:\\Python', 'C:\\Python\\lib\\site-packages']
>>>
```

也可以通过sys模块的append方法在Python环境中增加搜索路径。

```python
>>> sys.path.append("E:\\Project\\Python\\ModuleAndPackage2")
>>> sys.path
```

所以你写的代码不需要封装也可以当做模块引入，前提是你要告诉你代码的位置，然后import myfunc。

但是我要做的是像其他pip安装的模块一样，可以在有网络的地方pip安装我的模块，有完整的模块说明文档和函数说明文档，有严格的包版本管理协议。下面是我的实验结果，后期可以愉快开发自己的算法，造轮子了。


+ Pytestxj的GitHub地址:[https://github.com/DataXujing/Pytestxj](https://github.com/DataXujing/Pytestxj)

+ Pytestxj的PyPI地址：[https://pypi.python.org/pypi/Pytestxj](https://pypi.python.org/pypi/Pytestxj)

## Pytestxj的包结构：

首先贴上Pytestxj包的整个文件结构, 后面的内容都是以此为例进行说明：Python模块大体包括：.py的源码程序，MANIFEST.in文件，README.rst文件，setup.py,doc文件夹(这个文件夹你放你的documents吧，不过要用心写文档真是个难事，所以这个文件夹基本是不存在的——为自己的懒惰可耻一把。),COPYING.txt 就是授权文件，里面是你关于这个包的授权，比如：MIT license，那么你里面放入MIT License全文即可，当然，如果你不清楚这个，你完全可以不要这个文件,等等其他的文件。


![]({{ site.url }}/assets/bowen16/output_5.png)

在Python的世界里，有个叫Distutils的工具模块可以帮我们轻松的解决这个问题， setuptools是Distutils的加强版，既然这样，让我们开始打包之旅吧。

分别分析一下每个文件的意义，具体如何写可参考官方教程。

+ **.py的源文件**：这个就是你的算法代码，如果有若干个.py文件可以放在一个文件夹下，且需要写一个`__init__.py`在python模块的每一个包中，都有一个__init__.py文件（这个文件定义了包的属性和方法）。这里我们想实现数字和字符的加法运算，并且写好函数或类的说明文档。

+ **README.rst文件**：这个文件主要是对Python包的描述，不要使用.md的markdown语法，因为PyPI暂不支持Markdown语法，模块上传PyPI时，该文件会被渲染成html网页，作为模块的说明文档。（reStructureText的语法规则可参考官方文档:Quick reStructuredText
其实还有一种方法就是使用pandoc将markdown转换成rst格式，一种省事的方式就是使用pyandoc模块在发布的时候自动转换。
具体方法可以参考：Use Markdown README's in Python modules）因为语法问题，我在提交我的模块时PyPI总是渲染不好，改了好半天，浪费了我的时间。

+ **MANIFEST.in文件**：此文件在打包的时候告诉setuptools（Distutils）还需要额外打包哪些文件，例如我Pytestxj中的文件我就使用README.rst这个文件将其包含进来。当然LICENSE,COPYING.txt,doc/这些也可以通过它来一起打包进来。
下面是我自己的MANIFEST.in的内容：include README.rst。具体的语法规则可以参考：The MANIFEST.in template。

+ **setup.py文件**：核心文件（python代码），看一下我的setup.py文件的写法，具体的其他参数可以参考官方文档去写，网上有大量的模块可用，比较简单。

```python

import codecs
import os
import sys

try:
	from setuptools import setup
except:
	from distutils.core import setup



def read(fname):
	return codecs.open(os.path.join(os.path.dirname(__file__), fname)).read()

long_des = read("README.rst")
    
platforms = ['linux/Windows']
classifiers = [
    'Development Status :: 3 - Alpha',
    'Topic :: Text Processing',
    'License :: OSI Approved :: MIT License',
    'Programming Language :: Python :: 2',
    'Programming Language :: Python :: 2.7',
    'Programming Language :: Python :: 3',
    'Programming Language :: Python :: 3.5',
]

install_requires = [
    'numpy>=1.11.1',
    'pandas>=0.19.0'
]

    
setup(name='Pytestxj',
      version='0.6.1',
      description='A test module for DataXujing',
      long_description=long_des,
      py_modules=['pytestxu'],
      author = "DataXujing",  
      author_email = "xujing@inter-credit.net" ,
      url = "https://dataxujing.github.io" ,
      license="Apache License, Version 2.0",
      platforms=platforms,
      classifiers=classifiers,
      install_requires=install_requires
      
      )   
```

+ **docs/文件夹**：，这个文件夹你放你的documents，不过要用心写文档真是个难事，所以这个文件夹基本是不存在的——为自己的懒惰可耻一把。

+ **COPYING.txt文件**：就是授权文件，里面是你关于这个包的授权，比如：Apache License，MIT license，那么你里面放入Apache License全文即可，当然，如果你不清楚这个，你完全可以不要这个文件。

还有其他你需要的文件，测试代码，数据等内容不一一介绍了，足够封装你的模块。有了这些文件，开始生成自己的模块并上传至PyPI.


## 生成模块并上传PyPI

首先需要在PyPI中注册账号，然后在电脑的主文件目录下做如下配置,配置文件的后缀为.pypirc：

![]({{ site.url }}/assets/bowen16/output_2.png)

![]({{ site.url }}/assets/bowen16/output_3.png)

做好这一步就可以打开你的cmd输入如下代码：

![]({{ site.url }}/assets/bowen16/output_4.png)

中途会需要选择使用已存在的账号登录，初次可能需要输入登录名和密码，最后成功后会出现；200，OK的结果。

完成后原文件夹中多了几个文件：


![]({{ site.url }}/assets/bowen16/output_5.png)

去自己的账号下看一下

![]({{ site.url }}/assets/bowen16/output_6.png){:height="100%" width="90%"}

恭喜你成功了。

## 测试安装模块

pip在不同版本的python中安装并测试是否正常运行：


![]({{ site.url }}/assets/bowen16/output_8.png)

说明pip安装成功。

![]({{ site.url }}/assets/bowen16/output_9.png)

python环境模块成功运行。

![]({{ site.url }}/assets/bowen16/output_7.png)

是不是像其他模块一样，也具有函数的说明文档。

## 小结

通过实现字符串拼接和数值加法计算的小函数进行模块封装，并且上传PyPI供其他用户pip安装使用，初步完成了我们造轮子的梦想。


## 学习资料：

【0】[https://pypi.python.org/pypi/Pytestxj](https://pypi.python.org/pypi/Pytestxj)

【1】[https://python-packaging.readthedocs.io/en/latest/minimal.html#creating-the-scaffolding](https://python-packaging.readthedocs.io/en/latest/minimal.html#creating-the-scaffolding)

【2】[http://www.diveintopython3.net/table-of-contents.html](http://www.diveintopython3.net/table-of-contents.html)

【3】[http://blog.csdn.net/crisschan/article/details/51840552](http://blog.csdn.net/crisschan/article/details/51840552)

【4】[http://www.jb51.net/article/92789.htm](http://www.jb51.net/article/92789.htm)

【5】[http://blog.csdn.net/hyman_c/article/details/53445755](http://blog.csdn.net/hyman_c/article/details/53445755)
