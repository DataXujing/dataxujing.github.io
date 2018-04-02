---
layout: post
title: "R Interface to Python"
img: reticulated_python1.png 
date: 2018-3-30 12:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python,R,reticulate]
---

**译者:徐静**

### Getting Started

1.**安装**

可以在CRAN上安装

```r
install.packages("reticulate")
```

2.**Python版本控制**

默认情况下，reticulate使用系统变量中的Python,(i.e. Sys.which('python'))
`use_python()`函数可以用来修改python版本

```R
library(reticulate)
use_python("/usr/local/bin/python")
```

`use_virtualenv()`和`use_condaenv()` 可以设定虚拟变环境或conda环境中的python版本

```r
library(reticulate)
use_virtualenv("myenv")
```

详细的讲解可以看，后期我翻译更新的文章：Python Version Configuration

3.**python包**

熟悉python的同志们都知道可以使用`pip`或·conda`等的一些标准shell工具去安装你需要的python包，
另一种方法是，reticulate包含了一些管理和安装python包的函数集，使你可以在虚拟python环境(virtualenvs)和conda环境中安装和管理你的python包，详细的可以阅读我后期翻译的文章:Installing Python Packages 

4.**调用python**

这里提供了一些使你的Python代码插入到R代码的办法：

+ Python in R Markdown — A new Python language engine for R Markdown that supports bi-directional communication between R and Python (R chunks can access Python objects and vice-versa).

+ Importing Python modules — The import() function enables you to import any Python module and call it’s functions directly from R.

+ Sourcing Python scripts — The source_python() function enables you to source a Python script the same way you would source() an R script (Python functions and objects defined within the script become directly available to the R session).

+ Python REPL — The repl_python() function creates an interactive Python console within R. Objects you create within Python are available to your R session (and vice-versa).



### Python in R Markdown

reticulate包包含了一个R Markdown的Python引擎，有如下特点：

在您的R会话中嵌入的Python会话中运行Python块 (Python块之间的共享变量/状态)

Python的打印输出，包括从matplotlib图形输出。

使用R对象访问来自Python中的Python块中创建的对象 (e.g. py$x将访问从Python中创建的x变量)。

使用R对象访问来自Python的R块中创建的对象 (e.g. r.x会访问变量x在R从Python)

提供了许多python对象类型的内置转换，包括Numpy数组及Pandas数据库，例如：使用Pandas处理数据，R中的ggplot2绘图：

```python
import pandas as pd
flights = pd.read_csv('flights.csv')
flights = flights[flights['dest'] == 'ORD']
flights = flights['carrier','dep_delay','arr_delay']
flights = flights.dropna()
```

```r
#{r, fig.width=7,fig.height=3}
library(ggplot2)
ggplot(py$flights,aes(carrier,arr_deplay))+geom_point()+geom_jitter()
```
注释：加载安装reticulate时，默认情况下都会在R Markdown中启用Python引擎，详细的内容，将在我翻译的文章:R Markdown Python Engine 

### Importing Python modules

可以使用import()函数在R中import任何python模块，例如，下面的例子在R中import python模块os,并调用listdir()函数

```r
library(reticulate)
os <- import("os")
os$listdir(".")
```
我们看到了，这种用法就是R中S3对象的调用: 对象$对象，详细的可以看我后期翻译的文章： Calling Python from R

### Sourcing Python scripts

你可以像source R代码一样，source Python代码，使用`source_oython()`函数，例如，可以source flights.py脚本

```r
source_python("flights.py")
flights <- read_flights("flights.csv")

library(ggplot2)
ggplot(flights, aes(carrier, arr_delay)) + geom_point() + geom_jitter()
```

+ source_python()函数

Evaluate a Python script and make created Python objects available within R. The Python script is sourced within the Python main module, and so any objects defined are made available within Python as well.

source_python(file, envir = parent.frame(), convert = TRUE)
Arguments
file	Source file

envir	The environment to assign Python objects into (for example, parent.frame() or globalenv()). Specify NULL to not assign Python objects.

convert	 TRUE to automatically convert Python objects to their R equivalent. If you pass FALSE you can do manual conversion using the py_to_r() function.

### Python REPL

repl_python()在R console中运行python,exit退出

+ repl_python()

This function provides a Python REPL in the R session, which can be used to interactively run Python code. All code executed within the REPL is run within the Python main module, and any generated Python objects will persist in the Python session after the REPL is detached.

repl_python(module = NULL, quiet = getOption("reticulate.repl.quiet",
  default = FALSE))
Arguments
module	
An (optional) Python module to be imported before the REPL is launched.

quiet	
Boolean; print a startup banner when launching the REPL? If FALSE, the banner will be suppressed.

Details
When working with R and Python scripts interactively, one can activate the Python REPL with repl_python(), run Python code, and later run exit to return to the R console.

See also
py, for accessing objects created using the Python REPL.

Examples
```r
# NOT RUN {
# enter the Python REPL, create a dictionary, and exit
repl_python()
dictionary = {'alpha': 1, 'beta': 2}
exit

# access the created dictionary from R
py$dictionary
# $alpha
# [1] 1
# 
# $beta
# [1] 2

# }
```

### Why reticulate?

+ 维基百科

>网纹蟒是南洋发现的一种蟒蛇。他们是世界上最长的蛇，长的爬行动物…的具体名称，网纹，是拉丁语的意思是“网状”，或网状，并参照复杂的彩色图案。



### Reference

[1].[R Interface to Python(en)](https://rstudio.github.io/reticulate/index.html)