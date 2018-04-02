---
layout: post
title: " Calling Python from R"
img: reticulated_python1.png 
date: 2018-3-30 15:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python,R,reticulate]
---

**译者:徐静**

### OverView

reticulate包提供了一个导入python模块，类，方法的R接口，例如：下面的例子是在R中导入Python模块并使用模块中的函数。

```r
library(reticulate)
os <- import("os")
os$listdir(".")
```
Python模块和类中的函数和其他数据可以通过$操作符访问（类似于与R列表、环境或引用类交互的方式）。

牛B的是，reticulate包支持所有版本的python，只是对某些python包的版本有要求，比如Numpy >= 1.6

### Python Version

默认情况下，reticulate首先在环境变量中寻找Python(i.e.Sys.which('python'))，可以使用`use_python()`函数来更换你的python版本，例如：

```r
library(reticulate)
use_python('user/locla/bin/python')
```

同时可以使用`use_virtualenv()`和`use_condaenv()`设定虚拟环境和conda环境的版本，例如:

```r
library(reticulate)
use_virtualenv('myenv')
```

### Importing Modules

inport()函数可以用来导入Python模块

```r
difflib <- import("difflib")
difflib$ndiff(foo, bar)

filecmp <- import("filecmp")
filecmp$cmp(dir1, dir2)

```

`import_main()`和`import_builtins()`可以用来访问主模块（默认加载的模块，及python自建的模块函数

```r
main <- import_main()

builtins <- import_builtins()
builtins$print('foo')
```

### Sourcing Scripts

`source_python()`函数可以用来在R环境下source Python脚本

```python
#add.py
def add(x, y):
  return x + y
```
```r
source_python('add.py')
add(5,10)
```

### Executing Code

可以使用主模块中的`py_run_file`和`py_run_string`函数，这样就可以使用任何用Python创建的对象

```r
library(reticulate)

py_run_file("script.py")

py_run_string("x = 10")

# access the python main module via the 'py' object
py$x

```

### Object Conversion

默认情况下，当python独享返回到R时，需要转化成R对象，然而，如果不想转化可以在import python函数时加入参数`convert = FALSE`，此python对象就不能够转化成R对象。

```r

# import numpy and specify no automatic Python to R conversion
np <- import("numpy", convert = FALSE)

# do some array manipulations with NumPy
a <- np$array(c(1:4))
sum <- a$cumsum()

# convert to R explicitly at the end
py_to_r(sum)

```
如果你需要在计算最后转化成R对象，可以调用`py_to_r()`函数

### Getting Help

打印Python的帮助文档函数:`py_help()`

```r
os <- import("os")
py_help(os$chdir)

```


### Lists,Tuples, and Dictionaries


Python API需要一个list,需要用R中list()函数转化

```r
foo$bar(indexes = list(42L))
```
相似的，Python API需要一个tuple，此时你可以使用tuple()函数

```r
tuple('a','b','c')

```

可以用dict()函数，创建一个Python字典

```r
dict(foo = "bar", index = 42L)
```

### Numeric Types and Indexes

R和python有不同的默认数值类型，比如说42在R中默认会被转化为一个浮点数，而在Python中会默认被转化为整数。

这就意味着，如果Python API需要一个整型，在R中需要在结尾加上一个L,例如：

```r
foo$bar(index = 42L)
```

Python索引是以0开始的，而R中是以1开始的，因此如果在R中索引数组额第一个元素，需要

```r
items[[1]]
```

然而，如果你想通过reticulate去调用一个Python方法，你需要这样调用第一个索引：

```r
items$get(0L)
```

需要注意的是使用0作为第一个索引并且加上L作为结尾来表示是一个整型。


### Arrays

R矩阵或数组会自动与Numpy array相互转化。

R array to Numpy using `np_array()`

```r
a <- np_array(c(1:8), dtype = "float16")
a <- np_array(c(1:8), order = "C")
```

### DataFrame

R data frames 自动与pandas DataFrame相互转化

注：

|R|	Python|
|--------|-----------------------------------------|
|Factor|	Categorical Variable|
|POSIXt|	NumPy array with dtype = datetime64[ns]


### With Contexts

带有泛型函数的r可以用于与Python上下文管理器对象交互（在Python中，使用关键字）。例如:

```r
py <- import_builtins()
with(py$open("output.txt", "w") %as% file, {
  file$write("Hello, there!")
})
```

此示例打开一个文件，并确保它在该块的结尾处自动关闭。注意使用%as%操作符来别名由上下文管理器创建的对象别名。

### Iterrators

如果Python API返回的是一个迭代器或生成器，你可以在R中使用`iterate()`函数,实现迭代器的功能。

```r
iterate(iter,print)
```
如果不传递函数进行迭代，结果将被收集到R向量中

```r
results <- iterate(iter)
```
注意：

```r
a <- iterate(iter) # results are not empty
b <- iterate(iter) # results are empty since items have already been drained
```

### Element Level Iteration

```r
while (TRUE) {
  item <- iter_next(iter)
  if (is.null(item))
    break
}
```


```r
while (TRUE) {
  item <- iter_next(iter, completed = NA)
  if (is.na(item))
    break
}
```


### Generators

```r
# define a generator function
sequence_generator <-function(start) {
  value <- start
  function() {
    value <<- value + 1
    value
  }
}

# convert the function to a python iterator
iter <- py_iterator(sequence_generator(10))
```

### Python Objects

|Function|	Description|
|-------------|--------------------------------------------------|
|py_has_attr()|	Check if an object has a specified attribute.|
|py_get_attr()|	Get an attribute of a Python object.|
|py_set_attr()|	Set an attribute of a Python object.|
|py_list_attributes()|	List all attributes of a Python object.|
|py_len()|	Length of Python object.|
|py_call()|	Call a Python callable object with the specified arguments.|
|py_to_r()|	Convert a Python object to it’s R equivalent|
|r_to_py()|	Convert an R object to it’s Python equivalent|


### Pickle

You can save and load Python objects (via pickle) using the py_save_object and py_load_object functions:

|Function|	Description|
|-------------------|-----------------------------------------------|
|py_save_object()|	Save a Python object to a file with pickle.|
|py_load_object()|	Load a previously saved Python object from a file.|


### Configuration

|Function|	Description|
|-------------------|------------------------------------------------------|
|py_available()|	Check whether a Python interface is available on this system.|
|py_numpy_available()|	Check whether the R interface to NumPy is available (requires NumPy >= 1.6)|
|py_module_available()|	Check whether a Python module is available on this system.|
|py_config()|	Get information on the location and version of Python in use.|


### Output Control

这些函数使您能够捕获或抑制来自Python的输出：

|Function|	Description|
|---------------------|-----------------------------------------------------------------------------|
|py_capture_output()|	Capture Python output for the specified expression and return it as an R character vector.|
|py_suppress_warnings()|	Execute the specified expression, suppressing the display Python warnings.|


### Miscellaneous(其他)

|Function|	Description|
|------------------|---------------------------------------------------------------|
|py_set_seed()|	Set Python and NumPy random seeds.|
|py_unicode()|	Convert a string to a Python unicode object.|
|py_str()|	Get the string representation of Python object.|
|py_id()|	Get a unique identifier for a Python object|
|py_is_null_xptr()|	Check whether a Python object is a null externalptr.|
|py_validate_xptr()|Check whether a Python object is a null externalptr and throw an error if it is.|

### Reference

[1].[Calling Python from R (en)](https://rstudio.github.io/reticulate/articles/calling_python.html)

