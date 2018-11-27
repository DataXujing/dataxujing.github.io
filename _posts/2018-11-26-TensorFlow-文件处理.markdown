---
layout: post
title: "TensorFlow文件处理"
img: bowen58.png
date: 2018-11-26 12:00:00 
description: You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [TensorFlow,多线程]
---

### 1.TFRecord输入数据格式

TensorFlow提供了一种统一的格式来存储数据，这个格式就是TFRecord.

TFRecord格式介绍： TFRecord文件中的数据都是通过tf.train.Example Protocol Buffer的格式存储的，一下代码给出了tf,train.Example的定义

```python
message Example {
 Features features = 1;
};

message Features {
    map<string,Feature> feature = 1;
};

meaasge Feature {
    oneof kind {
        ByteList byte_list = 1;
        FlaotList float_list = 2;
        Int64List int64_list = 3;
    }
};
```

### 2.TFRecord样例程序

**将MNIST数据集转化成TFRecord格式：**

```python
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data
import numpy as np

# 生成整数型的属性

def _int64_feature(value):
    return tf.train.Feature(int64_list = tf.train.Int64List(value=[value]))

# 生成字符型的属性

def _bytes_feature(value):
    return tf.train.Feature(bytes_list=tf.train.BytesList(value=[value]))


mnist = input_data.read_data_sets(
 "path/to/mnist/data",dtypes=tf.uint8,one_hot=True)

images = mnist.train.images

# 训练数据所对应的正确答案。可以作为一个属性保存在TFRecord中
labels = mnist.train.labels
# 训练数据的图像分辨率。可以作为Example中的一个属性
pixels  = images.shape[1]
number_examples = mnist.train.num_examples

# 输出TFRecord文件的地址
filename = "path/to/output.tfrecords"
# 传建一个writer来写TFRecord文件
writer = tf.python_io.TFRecordWriter(filename)
for index in range(num_examples):
    # 将图像矩阵转化成一个字符串
    img_raw = images[index].tostring()
    # 讲一个样例转化为Example Protocol Buffer，并将所有的信息写入这个数据结构
    example = tf.train.Example(features=tf.train.Features(feature={
                'pixels':_int64_feature(pixels)
                'label':_int64_feature(np.argmax(label[index])
                'image_raw':_bytes_feature(image_raw)}))
    # 将一个Example写入TFRecord文件
    writer.write(example.SerializeToString())

writer.close()
```

这样就讲MNIST数据存储到一个TFRecord文件中，如果数据量比较大，也可以写入多个TFRecord文件

**读取TFRecord文件：**

```python

import tensorflow as tf

# 创建一个reader来读取TFRecord文件

reader = tf.TFRecordReader()
# 创建一个列表来维护输入文件列表
# tf.train.string_input_producer函数

filename_queue = tf.train.string_input_producer(
	["path/to/output.tfrecords"])

# 从文件中读出一个样例，也可以使用read_up_to
# 一次性读取多个样例

_,serialized_example = reader.read(filename_queue)

# 解析读入的一个样例，如果需要解析多个样例，
# 可以用parse_example函数

features = tf.parse_single_example(serialized_example,
	features={
	# TF提供了两种不同的属性解析方法，一种方法是tf.FixedLenFeature
	# 这种方法解析的结果为一个tensor,另一种方法时tf.varLenFeature
	# 这种方法得到的为sparseTensor，用于处理稀疏数据。这里解析数据的
	# 个数需要与上面写入的格式一致
	'image_raw':tf.FixedLenFeature([],tf.string),
	'pixels':tf.FixedLenFeature([],tf.int64),
	'label':tf.FixedLenFeature([],tf.int64)
	})

# tf.decode_raw可以将字符解析成图像对应的像素数组

images = tf.decode_raw(feature['image_raw'], tf.uint8)
labels = tf.cast(feature['label'], tf.int32) # 格式转化
pixels = tf.cast(feature['pixels'], tf.int32)

sess = tf.Session()

# 启动多个线程处理数据

coord = tf.train.Coordinator()
threads = tf.train.start_queue_runners(sess=sess,coord=coord)

# 每次运行可以读取TFRecord文件中的一个样例。当所有样例都读完后
# 程序会重头读取

for i  in range(10):
	image,label,pixel = sess.run([images,labels,pixels])
```

### 3.输入文件队列

首先假设我们已经将数据存储为多个TFRecord文件，TF提供了tf.train.match_filenames_once()函数来获取符合一个正则表达式的所有文件，得到的文件列表可以通过tf.train.string_input_producer函数进行有效管理。tf.train.string_input_producer函数会使用初始化时提供的文件列表创建一个输入队列，输入队列的原始的元素为文件列表中的所有文件。通过设置shuffle参数，tf.train.string_input_producer函数支持随机打乱文件列表的出队顺序

**生成样例数据**

```python
import tensorflow as tf

# 创建TFRecord文件的帮助函数

def _int64_feature(value):
	return tf.train.Feature(int64_list = tf.train.Int64List(value=[value]))

# 模拟海量数据情况下将数据写入不同的文件，num_shareds
# 定义了总共写入多少文件
# instance_per_shard定义了每个文件中有多少数据


num_shards = 2
instance_per_shard = 2

for i in range(num_shards):
	# 文件命名规则可以是;0000n-of-000m。
	# 其中m表示了数据被存在了多少个文件
	# n表示当前文件的编号
	# 目的是方便正则表达式获取文件列表
	filename = ("/path/to/data.tfrecords-%.5d-of-%.5d" % (i,num_shards))
	writer = tf.python_io.TFRecordWriter(filename)
	# 将数据封装成Example结构并写入TFRecord文件
	
	for j in range(instance_per_shard):
		# Example结构仅包含当前样例属于第几个文件
		# 以及当前文件的第几个样本
		example = tf.train.Example(features=tf.train.Features(
			feature = {
			   "i":_int64_feature(i),
			   "j":_int64_feature(j)
			}))
		writer.write(example.SerializeToString())
	writer.close()
```

tf.train.match_filenames_once和tf.train.string_input_producer函数使用

```python
import tensorflow as tf


# 使用tf.train.match_filename_once 获取文件列表
files = tf.train.match_filenames_once("/path/to/data.tfrecords-*")

# 通过tf.train.string_input_producer创建队列

filename_queue = tf.train.string_input_producer(files,shuffle=False)

# 读取并解析样本

reader = tf.TFRecordReader()

_,serialized_example = reader.read(filename_queue)
features = tf.parse_single_example(
	serialized_example, features={
		"i":tf.FixedLenFeature([],tf.int64),
		'j':tf.FixedLenFeature([],tf.int64),
	})

with tf.Session() as sess:
	tf.global_variables_initializer().run()
	print(sess.run(files))

	# 声明tf.train.Coordinator()来协同不同线程
	coord = tf.train.Coordinator()
	threads = tf.train.start_queue_runners(sess=sess,coord=coord)

	# 多次获取数据的操作
	for i in range(6):
		print(sess.run([fetches['i'],features['j']])
	coord.request_stop()
	coord.join(threads)
```

### 4.组合训练数据(batching)

TF提供了tf.train.batch和tf.train.shuffle_batch函数来讲单个样例组织成batch的形式输出，这两个函数都会生成一个队列。

```python
example,label = features['i'],features['j']
batch_size = 3
capacity = 1000+38batch_size  #队列的大小

example_batch,label_batch = tf.train.batch([example,label],batch_szie=batch_size,
	capacity=capacity)
```

tf.train.batch和tf.train.shuffle_batch还可以做并行化，设置参数nun_threads可以指定多个线程同时执行入队操作，当这个参数大于1时，多个线程会同时读取一个文件中的不同样例并进行预处理，除此之外还有tf.train,shuffle_batch_join()也可以完成多线程并行的方式来进行数据预处理。对于tf.train.shuffle_batch函数不同线程会读取同一个文件，使用tf.train.shuffle_batch_join（）不同线程会读取不同文件。






