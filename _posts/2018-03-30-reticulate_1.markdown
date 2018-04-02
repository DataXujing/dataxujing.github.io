---
layout: post
title: "reticulate: R interface to Python"
img: reticulated_python1.png 
date: 2018-3-30 9:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python,R,reticulate]
---

转自R语言中文社区

>牛B了我的Rstudio,reticulate正式我想要的，我已经告别了`rPython`
>
>(徐静)

### 一. 基础介绍

CRAN已于2018年3月21日收录reticulat包（1.6版本)，它包含了用于Python和R之间协同操作的全套工具，在R和Rstudio中均可使用。主要包括：

1）在R中支持多种方式调用Python。包括R Markdown、加载Python脚本、导入Python模块以及在R会话中交互式地使用Python。

2）实现R和Python对象之间的转换(例如R和Python数据框、R矩阵与NumPy数组之间)。

3）灵活绑定到不同版本的Python，包括虚拟环境和Conda环境。

在R会话中嵌入了Python会话，从而实现了无缝的、高性能的互操作性。如果你是使用Python进行某些工作的R开发人员或使用两种语言的数据科学团队的成员，那么reticulate包可以极大地简化你的工作流程!

### 二. 操作说明

#### 1. reticulate包安装

install.packages("reticulate")

library("reticulate")

#### 2. R Markdown中应用Python

reticulate包含一个用于R Markdown的Python引擎，具有以下功能：

1）可在R会话中嵌入的单个Python会话中，运行Python块。同时Python块之间的共享变量/状态。

2）可打印Python输出，包括matplotlib的图形输出。

3）可使用py对象访问R中Python块中创建的对象。

4）使用r对象从Python中访问在R块中创建的对象。

同时，内置了许多用于Python对象类型转换的方法，包括NumPy数组和Pandas数据框。 例如，你可以使用Pandas读取和操作数据，然后使用ggplot2轻松绘制Pandas数据框：

```python
#python
import pandas as pd
flights = pd.read_csv('flights.csv')
flights = flights[flights['dest'] == 'ORD']
flights = flights[['carrier','dep_delay','arr_delay']]
flights = flights.dropna()
```

```r
#{r,fig.width=7,fig.height=3}
library(ggplot2)
ggplot(py$flights,aes(carrier,arr_delay))+geom_point() + geom_jitter()
```
注释：加载安装reticulate时，默认情况下都会在R Markdown中启用Python引擎

#### 3. 加载Python模块

可以使用函数：import() 导入任何Python模块并从R中调用它。例如，此代码导入Python os模块并调用函数：listdir()

library(reticulate) 

os <- import("os") 

os$listdir(".")


#### 4. 载入Python脚本

可以使用函数：`source_python()`获取任何Python脚本，就像使用R脚本一样。如果你有以下Python脚本：

```python
#flights.py

import pandas

def read_flights(file):  
	flights = pandas.read_csv("flights.csv")  

	flights = flights[flights['dest'] == "ORD"]  

	flights = flights[['carrier', 'dep_delay', 'arr_delay']] 

	flights = flights.dropna()  

	return flights
```

然后，你可以编写脚本源代码并按如下所示调用函数：`read_flights()`

```r
source_python("flights.py") 

flights <- read_flights("flights.csv")

library(ggplot2) 

ggplot(flights, aes(carrier, arr_delay)) + geom_point() + geom_jitter()
```

在Python REPL中输入exit以返回到R提示符。同时，Python代码还可以使用r对象访问R会话中的对象（例如r.flights）

当调用Python时，R数据类型会自动转换为它们等效的Python类型。 当值从Python返回到R时，它们会被转换回R类型。如果返回自定义类的Python对象，则返回该对象的R引用。

#### 7. 更多学习

reticulate网站(https://rstudio.github.io/reticulate/) 包括了使用该软件包的详细文档，主要包括：

1）Calling Python from R : 介绍从R访问Python对象的各种方法以及可用于更高级的交互和转换行为的函数。

2）R Markdown Python Engine : 提供有关在R Markdown文档中使用Python块的详细信息，包括如何从R块调用Python代码，反之亦然。

3）Python Version Configuration Python版本配置 : 描述用于确定R会话中使用哪个版本的Python的工具。

4）Installing Python Packages : 有关从PyPI或Conda安装Python软件包的文档，以及使用virtualenvs和Conda环境管理软件包安装的文档。

5）Using reticulate in an R Package : 在R软件包中使用reticulate的准则和最佳实践。

6）Arrays in R and Python : 深层次讨论R和Python中数组之间的差异以及对转换和互操作性的影响。



### Reference

[1].[reticulate: R interface to Python](https://mp.weixin.qq.com/s?__biz=MzA3MTM3NTA5Ng==&mid=2651057816&idx=1&sn=07b5489a138daa5350d267574372d54d&chksm=84d9cf0fb3ae46198813f79d479dd212f8b3dde3e6d686f6d885b78066c365d6203a9d4fc885&mpshare=1&scene=23&srcid=0328NEhaSVuaEDUAucadHSBn#rd)

[2].[Github](https://github.com/rstudio/reticulate)

[3].[CRAN](https://CRAN.R-project.org/package=reticulate)

[4].[reticulate网站](https://rstudio.github.io/reticulate/)
