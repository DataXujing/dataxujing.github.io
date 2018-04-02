---
layout: post
title: "Installing Python Packages"
img: reticulated_python1.png 
date: 2018-3-31 16:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python,R,reticulate]
---

**译者:徐静**


### Conda installation

下面这些函数可以管理Conda环境

|Function|	Description|
|----------------|------------------------------------------|
|conda_list()|	List all available conda environments|
|conda_create()|	Create a new conda environment|
|conda_install()|	Install a package within a conda environment|
|conda_remove()|	Remove individual packages or an entire conda environment|

例子：

```r
library(reticulate)

# create a new environment 
conda_create("r-reticulate")

# install SciPy
conda_install("r-reticulate", "scipy")

# import SciPy (it will be automatically discovered in "r-reticulate")
scipy <- import("scipy")


library(reticulate)

# indicate that we want to use a specific condaenv
use_condaenv("r-reticulate")

# import SciPy (will use "r-reticulate" as per call to use_condaenv)
scipy <- import("scipy")
```


### 虚拟环境安装

下面这些函数管理Python虚拟环境

|Function|	Description|
|-------------------|----------------------------------------------|
|virtualenv_list()|	List all available virtualenvs|
|virtualenv_create()|	Create a new virtualenv|
|virtualenv_install()|	Install a package within a virtualenv|
|virtualenv_remove()|	Remove individual packages or an entire virtualenv|

例子：

```r
library(reticulate)

# create a new environment 
virtualenv_create("r-reticulate")

# install SciPy
virtualenv_install("r-reticulate", "scipy")

# import SciPy (it will be automatically discovered in "r-reticulate")
scipy <- import("scipy")

library(reticulate)

# indicate that we want to use a specific virtualenv
use_virtualenv("r-reticulate")

# import SciPy (will use "r-reticulate" as per call to use_virtualenv)
scipy <- import("scipy")


```


### Shell安装

```r
# install into system level Python
$ sudo pip install SciPy

# install into active Conda environment
$ conda install SciPy
```


### Reference

[1].[Installing Python Packages(en)](https://rstudio.github.io/reticulate/articles/python_packages.html)

