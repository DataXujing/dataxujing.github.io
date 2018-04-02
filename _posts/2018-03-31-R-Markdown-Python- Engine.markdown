---
layout: post
title: "R Markdown Python Engine"
img: reticulated_python1.png 
date: 2018-3-31 09:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python,R,reticulate]
---

**译者:徐静**

### Overview

+ reticulate包提供了一个针对于R markdown的Python引擎，使得R与Python块很好的交互
+ Python块R块非常相似的行为（包括从matplotlib图形输出）和两种语言有充分的访问彼此的对象。建了很多Python对象类型转换设置，包括NumPy数组和pandas数据框
+ 如果你使用的是knitr版本1.18或更高版本，然后reticulate Python引擎将被默认打开，reticulate的安装并没有进一步的设置要求。如果您运行的是较早版本的knitr或要禁用的reticulate引擎的使用见下面的引擎设置部分


### Python Version

默认情况下，使用默认路径的Python。如果你想使用另一个版本，你应该添加一个功能use_python()来设置，例如：

```r
#'''{r setup,include=FALSE}
library(reticulate)
use_python('usr/local/bin/python')
'''
```

### Python Chunks

+ Python代码块的工作原理与R代码块的工作原理完全一样，Python代码执行任何打印或图形（matplotlib)输出包含在文档
+ Python块的运行开启的是Python相同的会话，因此可以访问以前块中创建的所有对象，块的选项包括：echo,include,etc.
+ 下面给出一个R Markdown的例子：

```python
#python chunks
import pandas as pd
flights = pd.read_csv('flights.csv')
flights = flights[flights['dest']='ORD']
flights = flights[['carrier','dep_delay'.'arr_delay']]
flights = flights.dropna()
print(flight.head())

```

```python
#python chunks
import matplotlib.pyplot as plt
import numpy as np

t = np.arange(0.0,2.0,0.01)
s = 1+ np.sin(2*np.pi*t)

plt.plot(t,s)
plt.xlabel('time(s)')
plt.ylabel('voltage(mV)')
plt.grid(True)
plt.savefig('test.png')
plt.show()
```

### Calling Python from R

在Python块中创建的所有对象都可以使用py对象在R代码块中使用，比如下面的例子：

```python
#python chunks
import pandas as pd
flights = pd.read_csv('flights.csv')
flights = flights[flights['dest']='ORD']
flights = flights[['carrier','dep_delay'.'arr_delay']]
flights = flights.dropna()


```

```r
#R chunks
library(ggplot2)

ggplot(py$floghts,aes(carrier,arr_delay)) + geom_point() + geom_jitter()

```

### Calling R from Python

类似的，你可以可以在Python块中通过r对象访问R块中的对象，例如：

```r
#R chunks
library(tidyverse)
flights <- read_csv('flights.csv') %>%
  filter(dest=='ORD') %>%
  select(carrier,dep_delay,arr_delay) %>%
  na.omit()
```

```python
#Python chunks

print(r.flights.head())
```


### Engine Setup

如果你的Knitr是1.18之前的版本，可以通过以下setup来开启你的python引擎

```r
'''{r setup, include=FALSE}
knitr::knit_engines$set(python=reticulate::eng_python)
'''
```

如果你不想使用reticulate 开启Python引擎，可以做如下设置

```r
'''{r setup, include=FALSE}
knitr::opts_chunk$set(python.reticulate=FALSE)
'''
```

### Reference

[1].[R Markdown Python Engine（en)](https://rstudio.github.io/reticulate/articles/r_markdown.html)