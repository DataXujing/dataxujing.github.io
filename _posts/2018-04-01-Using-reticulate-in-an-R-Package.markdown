---
layout: post
title: "Using reticulate in an R Package"
img: reticulated_python1.png 
date: 2018-04-01 12:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python,R,reticulate]
---

**译者:徐静**


### Delay Loading Python Modules

如果你写一个R包，会包含的一个或多个Python包，很可能你会导入Python模块通过你的包的OnLoad方法，这样你可以在包的源代码的其余部分有方便的访问它们。

当你打算这么做时，你应该使用import()函数的delay_load标志，例如：

```r
# global reference to scipy (will be initialized in .onLoad)
scipy <- NULL

.onLoad <- function(libname, pkgname) {
  # use superassignment to update global reference to scipy
  scipy <<- reticulate::import("scipy", delay_load = TRUE)
}
```

使用delay_load参数有两个优点：

+ 在没有Python或Python环境的情况下，也可以成功加载，非常方便CRAN测试
+ 允许用户在与你的包交互前更改Python环境
```r
library(mypackage)
reticulate::use_virtualenv("~/pythonenvs/userenv")
# call functions from mypackage

```

### Checking and Testing on CRAN

如何通过CRAN的测试呢？

+ 使用delay_load选项

```r
# python 'scipy' module I want to use in my package
scipy <- NULL

.onLoad <- function(libname, pkgname) {
  # delay load foo module (will only be loaded when accessed via $)
  scipy <<- import("scipy", delay_load = TRUE)
}
```

+ 测试时，如果使用testthat包，可以这样做

```r
# helper function to skip tests if we don't have the 'foo' module
skip_if_no_scipy <- function() {
  have_scipy <- py_module_available("scipy")
  if (!have_scipy)
    skip("scipy not available for testing")
}

# then call this function from all of your tests
test_that("Things work as expected", {
  skip_if_no_scipy()
  # test code here...
})
```


### Implementing S3 Methods

```r
method.MyModule.MyPythonClass <- function(x, y, ...) {
  if (py_is_null_xptr(x))
    # whatever is appropriate
  else 
    # interact with the object
}
```

### Using Trabis-CI


.travis.yml中放入

```r
before_install:
  - pip install numpy any_other_dependencies go_here
```

### Reference

[1].[Using reticulate in an R Package](https://rstudio.github.io/reticulate/articles/package.html)