---
layout: post
title: "TensorFlow多GPU并行"
img: bowen59.png
date: 2018-11-26 20:00:00 
description: You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [TensorFlow,GPU,并行]
---

### 1.  TensorFlow使用GPU

TensorFlow程序通过tf.device函数来指定运行每一个操作的设备，这个设备可以是CPU或GPU，也可以是某一台远程的服务器，设备名称：CPU:/cup:0,GPU: /gpu:n。在配置好GPU的环境中TF会优先选择GPU。 注意不是所有的操作都可以放在GPU上，如果将无法放在GPU上的操作指定到GPU上，程序将会报错，并且不同版本的TF对GPU的支持也是不一样的。

TF提供了一个快捷的方式查看运行每一个运算的设备，在生成会话时，可以通过设置log_device_placement参数来打印运行每一个运算的设备。

```python
import tensorflow as tf

a = tf.constant([1.0,2.0,3.0],shape=[3],name="a")
b = tf.constant([1.0,2.0,3.0],shape=[3],name="b")

c = a + b

# 通过log_device_placement 参数来输出运行每一个运算的设备
gpu_options = tf.GPUOptions(per_process_gpu_memory_fraction=0.8)
sess = tf.Session(config = tf.ConfigProto(log_device_placement=True))
print(sess.run(c))

'''
a2dd: (Add): /job:localhost/replica:0/task:0/device:CPU:0

add: (Add)b/job:localhost/replica:0/task:0/device:CPU:0
/job:localhost/replica:0/task:0/device:CPU:0

b: (Const)/job:localhost/replica:0/task:0/device:CPU:0
a: (Const)/job:localhost/replica:0/task:0/device:CPU:0
[2. 4. 6.]
'''
```

如果将某些运算放在不同的GPU中或CPU中，需要通过tf.device来指定

```python
import tensorflow as tf

# 通过tf.device将运算指定到特定的设备上

with tf.device('/cpu:0'):
	a = tf.constant([1.0,2.0,3.0],shape=[3],name='a')
	b = tf.constant([1.0,2.0,3.0],shape=[3],name='b')

with tf.device('/cpu:1'):
	c = a+b

sess = tf.Session(config = tf.ConfigProto(log_device_placement=True))

print(sess.run(c))
```

为了避免指定GPU的操作而不支持GPU，TF在生成会话时可以指定allow_soft_placement参数，当allow_soft_placement参数设置为True时，如果运算无法由GPU执行，会自动放在CPU中，例子如下：

```python
import tensorflow as tf

a_cpu = tf.Variable(0,name="a_cpu")
with tf.device("/gpu:0"):
	a_gpu = tf.Variable(0,name="a_gpu")

# 通过allow_soft_placement参数自动将GPU上的操作放回CPU

sess = tf.Session(config=tf.ConfigProto(allow_soft_placement=True,
	log_device_placement=True))
sess.run(tf.global_variables_initializer())
```

### 2.多GPU并行

a. 深度学习并行的模式：同步模式和异步模式

**异步模式(模型并行)**: 在每一轮迭代时，不同设备会读取参数的最新取值，但因为不同设备在读取参数取值时的时间不一样，所以得到的值也有可能不一样。根据当前参数的取值和随机获取的一小部分训练数据，不同设备各自运行反向传播的过程并独立的更新参数。可以简单的认为异步模式就是单机模式复制了多份，每一份使用不同的训练数据进行训练。在异步模式下，不同的设备之间是完全独立的。

**同步模式(数据并行)**: 为了避免更新不同步的问题，可以使用同步模式，所有设备同时读取参数的取值，并且当反向传播算法完成之后同步更新参数的取值。单个设备不会单独对参数更新，而会·等待所有设备都完成反向传播后再统一更新参数。注意虽然所有设备使用的参数是一致的，但是因为训练数据不同，当所有设备完成反向传播的计算后，需要计算出不同设备上参数梯度的平均值，最后通过平均值对参数进行更新。但是同步模式低于异步模式，如果设备的运行速率不一致，那么每一轮训练都需要等待最慢的设备结束才能开始更新参数。

同步模式几乎适用于所有深度学习模型。使用数据并行时最好GPU的型号，速度最好一致，这样效率最高，而异步的数据并行，则不等待所有GPU都完成一次训练，而是哪个GPU完成了训练，就立即将梯度更新到共享的模型参数中，通常来说数据并行比异步并行的模式收敛速度更快，模型的精度更高。

### 3.Example

```python
import os.path
import re
import time
import numpy as np
import tensorflow as tf
import cifar10 # 训练的CNN模型

batch_size = 128
max_step  = 1000000
num_gpus = 4 # GPU个数

# 计算不同设备训练的损失
def tower_loss(scope):
	images,labels = cifar10.distorted_inputs()
	logits = cifar10.inference(images)
	_ = cifar10.loss(logits,labels)
	losses = tf.get_collection("lossess",scope)
	total_loss = tf.add_n(losses,name='total_loss')

	return total_loss


# 计算不同GPU上相同变量的梯度的平均值
# 用来更新该变量
def average_gradients(tower_grads):
	average_grads = []
	for grad_and_vars in zip(*tower_grads):
		grads = []
		for g,_ in grad_and_vars:
			expanded_g = tf.expand_dims(g,0)
			grads.append(expand_g)

		grad = tf.concat(grads, 0)
		grad = tf.reduce_mean(grad,0)

		v = grad_and_vars[0][1]
		grad_and_var = (grad,v)
		average_grads.append(grad_and_var)

	return average_grads

def train():
	with tf.Graph().as_default(),tf.device("/cpu:0"):
		global_step = tf,get_variables('global_step',[],
			initializer=tf.constant_initializer(0),
			trainable=False)

		num_batches_per_epoch = xxx
		deacy_steps = int(num_batches_per_epoch*xxxx)

		# 创建随训练步数衰减的学习率
		lr = tf.train.exponential_deacy("初始学习率",
			"全局训练的步数","每次衰减需要的步数",
			"衰减率",starcase=True)# 代表阶梯衰减

		opt = tf.train.GradientDescentOptimizer(lr)

		# 储存GPU计算结果的列表
		tower_grads = []

		for i in range(num_gpus):
			with tf.device("/gpu:%d" %i):
				with tf.name_scope("%s_%d" %(name1,i)) as scope:
					loss = tower_loss(scope)
					# 重用参数，让所有GPU公用一个
					# 模型及完全相同的参数
					tf.get_variable_scope().reuse_variables()
					# 计算梯度
					grads = opt.compute_gradient(loss)
					tower_grads.append(grads)

			grads = average_gradients(tower_grads)
			# 更新模型参数，global_step计数功能，从1开始
			apply_gradient_op = opt.apply_gradients(grads,
				global_step=global_step)


		server = tf.train.Saver(tf.all_variables())
		init = tf.global_variables_initializer()
		sess = tf.Session(config=tf.ConfigProto(allow_soft_placement=True))

		sess.run(init)
		tf.train.start_queue_runners(sess=sess)

		for step in range(max_steps):
			start_time = time.time()
			_,loss_value = sess.run([apply_gradient_op,loss])
			duration = time.time() - start_time

			if step % 10 == 0:
				# print something
			
			if step % 1000 == 0 or (step+1) == max_steps:
				# global_step计数作用
				saver.save(sess,"path/to/model/model.ckpt",
					global_step=step)
```
