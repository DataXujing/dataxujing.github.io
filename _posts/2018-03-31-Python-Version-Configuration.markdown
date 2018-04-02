---
layout: post
title: "Python Version Configuration"
img: reticulated_python1.png 
date: 2018-3-31 12:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python,R,reticulate]
---

**译者:徐静**


### 定位Python

一个系统中会有多版本的Python环境，比如conda,virtualenv等

```r
library(reticulate)
scipy <- import('scipy')
scipy$amin(c(1,2,3,7
```

+ 搜寻哪一个环境中有scipy，使用第一个找到该模块的环境。
+ 默认首先搜索环境变量中的环境



### Providing Hints（提供线索）

有两种方法可以提供关于应该使用哪种版本的Python的提示：

|Function|	Description|
|----------------|-------------------------------------------------|
|use_python()|Specify the path a specific Python binary.|
|use_virtualenv()|Specify the directory containing a Python virtualenv.|
|use_condaenv()|Specify the name of a Conda environment.|

例子：

```r
library(reticulate)
use_python("/usr/local/bin/python")
use_virtualenv("~/myenv")
use_condaenv("myenv")
```

`use_condaenv`可以加载系统变量之外的python环境

```r
use_condaenv(condaenv = "r-nlp", conda = "/opt/anaconda3/bin/conda")
use_virtualenv("~/myenv", required = TRUE)

```

You can add the required parameter to ensure that the specified version of Python is always used !!!


### 配置信息

You can use the py_config() function to query for information about the specific version of Python in use as well as a list of other Python versions discovered on the system:

```r
py_config()
```
You can also use the py_discover_config() function to see what version of Python will be used without actually loading Python:
```r
py_discover_config()
```

### Reference

[1].[Python Version Configuration](https://rstudio.github.io/reticulate/articles/versions.html)