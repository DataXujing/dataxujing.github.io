---
layout: post
title: "TensorFlow分布式并行(案例篇-1)"
img: bowen61.png
date: 2018-11-29 12:00:00 
description: You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [TensorFlow,分布式,并行]
---

### 1.本机的简单测试

**创建一个最简单的TensorFlow集群**

```python
import tensorflow as tf
c = tf.constant("Hello, Distributed TensorFlow")

# 创夹一个本地TensorFlow集群
server = tf.train.Server.create_local_server()
# 在集群上创建一个会话
sess = tf.Session(server.target)
# 输出
print(sess.run(c))
```
通过tf.train.Server.create_local_server函数在本地建立一个只有一台机器的TensorFlow集群。上面的代码只有一个任务的集群，可以做两个任务，使用tf.train.ClusterSpec来指定运行每一个任务的机器。

```python
import tensorflow as tf
c = tf.constant("Hello from server1")

# 生成一个有两个任务的集群，
# 一个任务跑在本地2222端口，一个任务跑在本地2223端口

cluster = tf.train.ClusterSpec({
    "local":['localhost:2222','localhost:2223']})

# 通过上面生成的集群配置生成server,
# 通过job_name和task_index指定当前所启动的任务。
# 因为该任务是第一个任务，所以task_index=0
server = tf.train.Server(cluster,job_name="local",task_index=0)

# 通过server.target生成会话来使用TensorFlow集群中的资源
# 通过设置log_device_placement可以看到执行每一个操作的任务
sess = tf.Session(server.target,conf=tf.VonfProto(
    log_device_place=True))

print(sess.run(c))
server.join()
```

第二个任务的代码：

```python
import tensorflow as tf
c = tf.constant("Hello from server2")

# 和第一个程序一样的集群配置。
# 集群中的每一个任务需要采取相同的配置

cluster = tf.train.ClusterSpec(
    {"local":["localhost:2222","localhost:2223"]})

# 指定task_index为1，所以这个程序将在localhost：2223启动服务
server = tf.train.Server(cluster,job_name="local",task_index=1)
# 剩下的代码和第一个任务相同

```
与使用多GPU类似，TensorFlow支持通过tf.device来指定操作运行在那个服务上，比如将第二个任务中定义计算的语句改为如下代码，计算将被调度到/job:local/replica:0/task:1/cpu:0上

```python
with tf.device("/job:local/task:1):
    c = tf.constant("Hello from server2")
```

在上面的例子中之定义了一个工作“local",但一般在训练深度学习模型时，会定义两个工作，一个工作专门负责存储，获取以及更新变量的取值，这个工作所包含的任务统称为参数服务器(ps)。另一个工作负责运行反向传播算法来获取参数梯度，这个工作所包含的任务称为计算服务器(worker),下面给出比较常用的训练深度学习模型的TensorFlow集群配置方法

```shell
tf.train.ClusterSpec({
"worker":[
    "tf-worker0:2222",
     "tf-worker1:2222",
    "tf-worker2:2222"
    ],
"ps":[
    "tf-ps0:2222",
    "tf-ps1:2222"
]
})

```

### 2.分布式TensorFlow模型训练

**异步模式样例程序**

```python
import time
import tensorflow as tf

from tensorflow.examples.tutorials.mnist import input_data
import mnist_inference # Model 文件

BATCH_SIZE = 100
LEARNING_RATE_BASE = 0.01
LEARNING_RATE_DECAY = 0.99
REGULARAZATION_RATE = 0.0001
TRAINING_STEPS = 1000

# 模型保存的路径
MODEL_SAVE_PATH = '/path/to/model'
DATA_PATH = '/path/to/data'

# 通过flags来指定运行的参数

FALAGS = tf.app.flags.FLAGS

tf.app.flags.DEFINE_string("job_name",'worker','"ps" or "worker"')

# 指定集群中的参数地址

tf.app.flasg.DEFINE_string(
	'ps_hosts','tf-ps0:2222,tf-ps1:1111',
	'Comma-separated list of hostname:port for the ps server jobs.')

# 指定及群众的计算服务器的地址

tf.app.flags.DEFINE_string(
	"worker_hosts","tf-worker0:2222,tf-worker1:1111",
	'Comma-separated list of hostname:port for the worker server jobs.')

# 指定当前程序的任务ID，TF会自动根据参数服务器/计算服务器列表中的端口号
# 来启东服务，注意参数和计算服务器的编号都是从0开始的

tf.app.flags.DEFINE_integer(
	'task_id',0,'Task ID of the worker/replica running the training')
# 定义TF的计算图，并返回每一轮迭代需运行的操作

def build_model(x,y_,is_chief):
	regularizer = tf.contrib.layers.l2_regularizer(REGULARAZATION_RATE)
	y = mnist_inference.inference(x,regularizer) # 模型调用
	global_step = tf.Variable(0,trainable=False)


	# 计算损失函数并定义反向传播过程
	cross_entropy = tf.nn.sparse_softmax_cross_entropy_with_logits(
		y,tf.argmax(y_,1))
	cross_entroy_mean= tf.reduce_mean(cross_entropy)
	loss = cross_entroy_mean + tf.add_n(tf.get_collection("lossess"))
	lerning_rate = tf.train.exponential_decay(LEARNING_RATE_BASE , 
		global_step, 6000/BATCH_SIZE, LEARNING_RATE_DEACY)

	# 定义每-轮迭代运行的操作
	train_op = tf.train.GradientDescentOptimizer(learning_rate)\
	.minimize(loss,global_step=global_step)

	return global_step,loss,train_op


# 训练分布式深度学习模型的主过程

def main(avgv=None):
	# 解析flags并通过tf.train.ClusterSpec配置tf集群
	ps_hosts = FLAGS.ps_hosts.split(",")
	worker_hosts = FLAGS.worker_hosts.split(",")
	cluster = tf.train.ClusterSpec({"ps":ps_hosts,"worker":worker_hosts})
	# 通过clusterSpec及当前任务创建Server
	
	server = tf.train.Server(
		cluster,job_name = FLAGS.job_name,task_index=FLAGS.task_id)

	# 参数服务器只需要管理TF中的变量，不需要执行训练过程
	# server.join()会一直停在 这条语句上
	if FLAGS.job_name == "ps":
		server.join()


	# 定义计算服务器运行的操作，在所有的计算服务器中
	# 有一个主计算服务器，他除了负责计算反向传播的结果‘
	# 还负责输出日志和保存模型
	
	is_chief = (FLAGS.task_id == 0)
	mnist = input_data.read_data_sets(DATA_PATH,one_hot=True)

	# 通过tf.train.replica_device_setter函数来指定执行每一个运算的
	# 设备
	# tf.train.replica_device_setter函数会讲所有的参数分配到参数服务器
	# 而计算分配到当前的计算服务器上
	
	with tf.device(tf.train.replica_device_setter(
		worker_device = "/job:worker/task:d%" %FLAGS.task_id,
		cluster = cluster)):

		x = tf.placeholder(tf.float32,[None,mnist_inference.INPUT_NODE],
			name = "x-input")
		y = tf.placeholder(tf.float32,[None,mnist_inference.OUTPUT_NODE],
			name = "y-input")

		# 定义训练模型需要的操作
		
		global_step,loss,train_op = build_model(x, y_, is_chief)

		# 定义用于保存模型的server
		saver = tf,train.Saver()
		# 定义日志输出操作
		summary_op = tf.merge_all_summaries()
		# 定义变量初始化操作
		init_op = tf.global_variables_initializer()
		# 通过tf.train.Supervisor管理训练深度学习模型的通用功能
		# 能统一管理队列操作，模型保存，日志输出以及会话额生成
		
		sv = tf.train.Supervisor(
			is_chief = is_chief, # 定义当前计算服务器是否为主计算服务器，
				# 只有主计算服务器会保存模型以及输出日志
			logdir = MODEL_SAVE_PATH, # 保存模型和输出日志的地址
			init_op = init_op, # 指定初始化操作
			summary_op = summary_op, # 指定日志生成操作
			saver = saver, # 指定用于模型保存的server
			global_step = global_step, # 指定当前迭代轮数
			save_model_secs = 60,# 指定保存模型的时间间隔
			save_summary_secs = 60,# 指定日志输出的时间间隔
			
			)
		sess_config = tf.ConfigProto(allow_soft_placement=True,
			log_device_placement=True)
		# 通过tf.train.Supervisor生成会话
		sess = sv.prepare_or_wait_for_session(server.target,config=sess_config)

		step = 0
		start_time = time.time()
		# 执行迭代过程，过程中sv会帮助输出日志和保存模型，因此不信医药直接调用这些
		# 过程
		while not sv.should_stop():
			xs,ys = mnist.train.next_batch(BATCH_SIZE)
			_.loss_value,global_step_value = sess.run(
				[train_op,loss,global_step],feed={x:xs,y_:ys})
			if global_step_value >= TRAINING_STEPS: break

			# 每隔一段时间输出训练信息
			if step > 0 and step % 100 == 0:
				duration = time.time()-start_time

				# 不同的计算服务器都会更新全局的训练轮数，所以这里使用
				# global_step_value可以直接得到训练中使用的batch的总数
				sec_per_batch = duration/global_step_value

				format_str = ("After %d training steps(%d global steps),"
					"loss on training batch is %g. " 
					"(%.3f sec/batch)")
				print(format_str %(step,global_step_value,
						loss_value,sec_per_batch))
			step += 1
		sv.stop()

if __name__ == '__main__':
	tf.app.run()
```

** 同步模式样例程序**

```python
'''
tf 分布式同步模式
'''

import time
import tensorflow as tf

from tensorflow.examples.tutorials.mnist import input_data
import mnist_inference # Model 文件

BATCH_SIZE = 100
LEARNING_RATE_BASE = 0.01
LEARNING_RATE_DECAY = 0.99
REGULARAZATION_RATE = 0.0001
TRAINING_STEPS = 1000

# 模型保存的路径
MODEL_SAVE_PATH = '/path/to/model'
DATA_PATH = '/path/to/data'

# 通过flags来指定运行的参数

FALAGS = tf.app.flags.FLAGS

tf.app.flags.DEFINE_string("job_name",'worker','"ps" or "worker"')

# 指定集群中的参数地址

tf.app.flasg.DEFINE_string(
	'ps_hosts','tf-ps0:2222,tf-ps1:1111',
	'Comma-separated list of hostname:port for the ps server jobs.')

# 指定及群众的计算服务器的地址

tf.app.flags.DEFINE_string(
	"worker_hosts","tf-worker0:2222,tf-worker1:1111",
	'Comma-separated list of hostname:port for the worker server jobs.')

# 指定当前程序的任务ID，TF会自动根据参数服务器/计算服务器列表中的端口号
# 来启东服务，注意参数和计算服务器的编号都是从0开始的

tf.app.flags.DEFINE_integer(
	'task_id',0,'Task ID of the worker/replica running the training')
# 定义TF的计算图，并返回每一轮迭代需运行的操作
# tf.train.SyncReplicasOptimizer函数处理同步更新

def build_model(x,y_,n_workers,is_chief):
	regularizer = tf.contrib.layers.l2_regularizer(REGULARAZATION_RATE)
	y = mnist_inference.inference(x,regularizer) # 模型调用
	global_step = tf.Variable(0,trainable=False)


	# 计算损失函数并定义反向传播过程
	cross_entropy = tf.nn.sparse_softmax_cross_entropy_with_logits(
		y,tf.argmax(y_,1))
	cross_entroy_mean= tf.reduce_mean(cross_entropy)
	loss = cross_entroy_mean + tf.add_n(tf.get_collection("lossess"))
	lerning_rate = tf.train.exponential_decay(LEARNING_RATE_BASE , 
		global_step, 6000/BATCH_SIZE, LEARNING_RATE_DEACY)

	# 异步
	# # 定义每-轮迭代运行的操作
	# train_op = tf.train.GradientDescentOptimizer(learning_rate)\
	# .minimize(loss,global_step=global_step)
	# 同步
	# 通过tf.train.SyncReplicasOptimizer函数处理同步更新
	opt = tf.train.SyncReplicasOptimizer(
		#定义基础的优化函数
		tf.train.GradientDescentOptimizer(learning_rate),
		# 定义每一轮更新需要多少个计算服务器得出的梯度
		replicas_to_aggregate = n_workers,
		# 指定当前服务器的编号
		replica_id = FLAGS.task_id)

	train_op = opt.minimize(loss,global_step=global_step)
	

	return global_step,loss,train_op,opt



# 训练分布式深度学习模型的主过程
# 与异步模式类似
def main(avgv=None):
	# 解析flags并通过tf.train.ClusterSpec配置tf集群
	ps_hosts = FLAGS.ps_hosts.split(",")
	worker_hosts = FLAGS.worker_hosts.split(",")
	cluster = tf.train.ClusterSpec({"ps":ps_hosts,"worker":worker_hosts})
	# 通过clusterSpec及当前任务创建Server
	n_workers = len(worker_hosts)
	
	server = tf.train.Server(
		cluster,job_name = FLAGS.job_name,task_index=FLAGS.task_id)

	# 参数服务器只需要管理TF中的变量，不需要执行训练过程
	# server.join()会一直停在 这条语句上
	if FLAGS.job_name == "ps":
		server.join()


	# 定义计算服务器运行的操作，在所有的计算服务器中
	# 有一个主计算服务器，他除了负责计算反向传播的结果‘
	# 还负责输出日志和保存模型
	
	is_chief = (FLAGS.task_id == 0)
	mnist = input_data.read_data_sets(DATA_PATH,one_hot=True)

	# 通过tf.train.replica_device_setter函数来指定执行每一个运算的
	# 设备
	# tf.train.replica_device_setter函数会讲所有的参数分配到参数服务器
	# 而计算分配到当前的计算服务器上
	
	with tf.device(tf.train.replica_device_setter(
		worker_device = "/job:worker/task:d%" %FLAGS.task_id,
		cluster = cluster)):

		x = tf.placeholder(tf.float32,[None,mnist_inference.INPUT_NODE],
			name = "x-input")
		y = tf.placeholder(tf.float32,[None,mnist_inference.OUTPUT_NODE],
			name = "y-input")

		# 定义训练模型需要的操作
		
		global_step,loss,train_op = build_model(x, y_, is_chief)

		# 定义用于保存模型的server
		saver = tf.train.Saver()
		# 定义日志输出操作
		summary_op = tf.merge_all_summaries()
		# 定义变量初始化操作
		init_op = tf.global_variables_initializer()

		# 在同步模式下，主计算服务器需要协调不同的计算服务器
		# 得到的参数梯度并最终更新参数。这需要主计算服务器完成
		# 一些额外的初始化工作。
		if is_chief:
			# 定义协调不同计算服务的队列并定义初始化操作
			chief_queue_runner = opt.get_chief_queue_runner()
			init_tokens_op = opt.get_init_tokens_op(0)

		# 通过tf.train.Supervisor管理训练深度学习模型的通用功能
		# 能统一管理队列操作，模型保存，日志输出以及会话额生成
		
		sv = tf.train.Supervisor(
			is_chief = is_chief, # 定义当前计算服务器是否为主计算服务器，
					# 只有主计算服务器会保存模型以及输出日志
			logdir = MODEL_SAVE_PATH, # 保存模型和输出日志的地址
			init_op = init_op, # 指定初始化操作
			summary_op = summary_op, # 指定日志生成操作
			saver = saver, # 指定用于模型保存的server
			global_step = global_step, # 指定当前迭代轮数
			save_model_secs = 60,# 指定保存模型的时间间隔
			save_summary_secs = 60,# 指定日志输出的时间间隔
			
			)
		sess_config = tf.ConfigProto(allow_soft_placement=True,
			log_device_placement=True)
		# 异步方式
		# # 通过tf.train.Supervisor生成会话
		# sess = sv.prepare_or_wait_for_session(server.target,config=sess_config)
		# 同步方式
		# 在开始训练模型之前，主计算服务器需要启动协调同步更新的队列
		# 并执行初始化操作
		
		if is_chief:
			sv.start_queue_runners(sess,[chief_queue_runner])
			sess.run(init_tokens_op)

		step = 0
		start_time = time.time()
		# 执行迭代过程，过程中sv会帮助输出日志和保存模型，因此不信医药直接调用这些
		# 过程
		while not sv.should_stop():
			xs,ys = mnist.train.next_batch(BATCH_SIZE)
			_.loss_value,global_step_value = sess.run(
				[train_op,loss,global_step],feed={x:xs,y_:ys})
			if global_step_value >= TRAINING_STEPS: break

			# 每隔一段时间输出训练信息
			if step > 0 and step % 100 == 0:
				duration = time.time()-start_time

				# 不同的计算服务器都会更新全局的训练轮数，所以这里使用
				# global_step_value可以直接得到训练中使用的batch的总数
				sec_per_batch = duration/global_step_value

				format_str = ("After %d training steps(%d global steps),"
					"loss on training batch is %g. "
					 "(%.3f sec/batch)")
				print(format_str %(step,global_step_value,
							loss_value,sec_per_batch))
			step += 1
		sv.stop()

if __name__ == '__main__':
	tf.app.run()
```

当一个计算服务器被卡住时，其他所有的计算服务器都需要等待这个最慢的服务器。为了解决这个问题，可以调整tf.train.SyncReplicasOptimizer函数中的replicas_to_aggregate参数。当replicas_to_aggregate小于计算服务器总数时，每一轮迭代就不需要收集所有的梯度，从而避免被最慢的计算服务器卡住。TensorFlow也支持通过调整同步队列初始化操作tf.train.SyncReplicasOptimizer.get_init_tokens_op中的参数来控制对不同计算服务之间的同步要求。当提供给初始化函数get_init_tokens_op的参数大于0时，TensorFlow支持多次使用由共同一个计算服务器得到的梯度，于是也可以缓解计算服务器性能瓶颈的问题。
