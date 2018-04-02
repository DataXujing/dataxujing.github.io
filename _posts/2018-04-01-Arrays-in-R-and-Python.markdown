---
layout: post
title: "Arrays in R and Python"
img: reticulated_python1.png 
date: 2018-04-01 09:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python,R,reticulate]
---

**译者:徐静**

密集的数据连续存储在内存中，由一个单一的指标处理（内存地址）。数组内存排序策略将单个索引转换成与数组坐标对应的多个索引。例如，矩阵有两个索引：行和列。三维数组有三，等等.


### 列主序

列主序是用Fortran语言，MATLAB，R，和最底层核心的线性代数库（BLA）。顺序地址位置被转换为数组坐标i，j，k，……这样第一个数组坐标随地址变化最快，下一个数组坐标不那么快，等等。

### 行主序

行主要排序是顺序地址索引和数组坐标之间的一种不同的转换，而不是在矩阵中跨行放置内存中的顺序数据。行主排序有时被称为“C”样式排序和列主要排序“FORTRAN”样式。Python / NumPy是指在阵旗c_contiguous和f_contiguous顺序，分别。

### Python

Python NumPy库很一般。它可以使用行为主或列为主的有序数组，但默认为**行主排序**。


### What about going from R column-major arrays to Python?

```r
(y <- array(1:24, c(4, 3, 2)))  # In R

## , , 1
##      [,1] [,2] [,3]
## [1,]    1    5    9
## [2,]    2    6   10
## [3,]    3    7   11
## [4,]    4    8   12
## 
## , , 2
##      [,1] [,2] [,3]
## [1,]   13   17   21
## [2,]   14   18   22
## [3,]   15   19   23
## [4,]   16   20   24

(x <- np$array(y))              # and now in Python

## [[[ 1 13]
##   [ 5 17]
##   [ 9 21]]
## 
##  [[ 2 14]
##   [ 6 18]
##   [10 22]]
## 
##  [[ 3 15]
##   [ 7 19]
##   [11 23]]
## 
##  [[ 4 16]
##   [ 8 20]
##   [12 24]]]
```

```r
x$flags

##   C_CONTIGUOUS : False
##   F_CONTIGUOUS : True
##   OWNDATA : True
##   WRITEABLE : True
##   ALIGNED : True
##   UPDATEIFCOPY : False

```

**在array转换的时候，一定要注意调整，列主序和行主序**

### Reference

[1].[Arrays in R and Python(en)](https://rstudio.github.io/reticulate/articles/arrays.html)