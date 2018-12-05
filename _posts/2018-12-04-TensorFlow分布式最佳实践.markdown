---
layout: post
title: "TensorFlow分布式最佳实践"
img: bowen62.png
date: 2018-11-29 12:00:00 
description: You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [TensorFlow,分布式,并行]
---

运行方式：在三台设备上分别启动：

```python
python mnist_replica.py --job_name="ps" --task_index=0
python mnist_replica.py --job_name="worker" --task_index=0
python mnist_replica.py --job_name="worker" --task_index=1
```

分布式训练的过程：

```python

# 首先定义一些常量

flags = tf.app.flags
flags.DEFINE_string('data_dir','/tmp/mnist-data')

# 只下载数据不做其他操作

flags.DEFINE_boolean("download_only",False,"Only perform downloading \
	of data")

# task_index从0开始，0代表用来初始化 变量的第一个任务

flags.DEFINE_integer('task_index',None,
	"worker task index,task_index=0 is the master worker task")
# 每台机器中的GPU的个数

flags.DEFINE_integer("num_gpus",0,
	"Total number of gpus for each machine")

# 在同步模式下，设置收集的工作节点的数量。默认就是工作节点的总数
flags.DEFINE_integer("replicas_to_aggregate",None,
	"Number of replicas to aggregate before paramenter update")

flas.DEFINE_integer('hidden_units',100,
	"Number of units in the hidden layer of the NN")

# 训练的次数

flags.DEFINE_integer("train_steps",200,
	"Number of gloabl training steps to perform")
flasg.DEFINE_integer("batch_size",100,"Training batch size")
flasg.DEFINE_float("learning_rate",0.01,"Learning rate")

# 使用同步训练(Sync_SGD)/异步训练(Async_SGD)

flags.DEFINE_boolean('sync_replicas',False,"Use the sync_replicas mode")

# 如果服务器已经存在采用gRPC协议通信，如果不存在，采用进程间通信
flags.DEFINE_boolean("existing_servers",False,"Weather servers already exists. If True")

# 参数服务器主机

flags.DEFINE_string("ps_hosts","localhost:2222",
	"Comma-separared list of hostname:port pairs")
# 工作节点主机
flags.DEFINE_string("worker_hosts","localhost:2223,localhost:2224")

# 本作业是工作节点还是参数服务器
flags.DEFINE_string("job_name",None,"job name: worker or ps")

FLAGS = flags.DLAGS
IMAGE_PIXELS = 28

# 读取集群的描述信息
ps_spec = FLAGS.ps_hosts.split(",")
worker_spec = FLAGS.worker_hosts.split(",")

# 创建TF集群描述对象
cluster = tf.train.ClusterSpec({
	"ps":ps_spec,
	"worker":worker_spec
	})

# 为本地执行的任务创建serevr对象
# 创建本地server对象，从tf.train.Server这个定义开始，每个节点开始不同
# 根据执行的命令的参数不同，决定了这个任务是哪个任务
# 如果作业名字是ps，进程就加入这里，作为参数更新的服务，等待其他工作节点给他提交
# 参数更新的数据。如果作业名字是worker，就执行后面的计算任务。

if not FLAGS.existing_servers:
	server = tf.train.Server(cluster,job_name=FLAGS.job_name,
		task_index=FLAGS.task_index)
	# 如果是参数服务器直接启动即可，这时候进程就会阻塞在这里
	# 下面的tf.train.replica_device_setter代码会将参数指定给ps_server保管
	if FLAGS.job_name == "ps":
		server.join()

# 找出worker的主节点，即task_index为0的节点
is_chief = (FLAGS.task_index == 0)

# 如果使用gpu
if FLAGS.num_gpus > 0:
	if FLAGS.num_gpus < num_workers:
		raise ValueError("number of gpus is less than number of workers")
	gpu = (FLAGS.task_index % FLAGS.num_gpus)

	# 分配worker到制定的gpu上运行
	worker_device = "/job:worker/task:%d/gpu:%d" % (FLAGS.task_index,gpu)

# 如果使用cpu
elif FLAGS.num_gpus == 0:
	# 把cpu分配过worker
	cpu = 0
	worker_device = "/job:worker/task:%d/cpu:%d" %(FLAGS.task_index,gpu)

# 在这个with语句下定义的参数，会自动分配到参数服务器上去定义
# 如果有多个参数服务器，就轮流循环分配

with tf.device(tf.train.replica_device_setter(worker_device=worker_device,
	ps_device="/job:ps/cpu:0",cluster=cluster)):
	# 定义全局步长，默认值为0
	global_step = tf.Variable(0,name='global_step',trainable=False)

	# 定义隐藏层参数变量，这里是全连接神经网络隐藏层
	hid_w = tf.Variable(
		tf.tuncated_normal([IMAGE_PIXELS*IMAGE_PIXELS,FLAGS.hidden_units],
			stddev=1.0/IMAGE_PIXELS),name="hid_w")
	hid_b = tf.Variable(tf.zeros([FLAGS.hidden_units]),name="hid_b")

	# 定义softmax回归层的参数变量
	
	sm_w = tf.Variable(tf.tuncated_normal([FLAGS.hidden_units,10],
		stddev=1.0/math.sqrt(FLAGS.hidden_units),name="sm_w"))
	sm_b = tf.Variable(tf.zeros([10]),name="sm_b")

	# 定义模型输入数据变量
	x = tf.placeholder(tf.float32,[None,IMAGE_PIXELS*IMAGE_PIXELS])
	y_ = tf.placeholder(tf.float32,[None,10])

	# 构建隐藏层
	hid_lin = tf.nn.xw_plus_b(x,hid_w,hid_b)
	hid = tf.nn.relu(hid_lin)

	# 构建损失函数和优化器
	y = tf.nn.softmax(tf.nn.xw_plus_b(hid,sm_w,sm_b))
	cross_entropy = -tf.reduce_sum(y_*tf.log(tf.clip_by_value(y，1e-10,1.0)))

	# 异步训练模式：自己计算完梯度就去更新参数，不同副本之间不会去协调进度
	opt = tf.train.AdamOptimizer(FLAGS.learning_rate)

	# 同步训练模式
	if FLAGS.sync_replicas:
		if FLAGS.replicas_to_aggregate is None:
			replicas_to_aggregate = num_workers
		else:
			replicas_to_aggregate = FLAGS.replicas_to_aggregate
		# 使用sync_replicasOptimizer作为优化器，并且是在图间复制的情况下
		# 在图内复制情况下将所有的梯度平均就可以了
		opt = tf.train.SyncReplicasOptimizer(
			opt,
			replicas_to_aggregate = replicas_to_aggregate,
			total_num_replicas = num_workers,
			name = "mninst_sync_replicas")

	train_step = opt.minimize(cross_entropy,global_step=global_step)

	if FLAGS.sync_replicas:
		local_init_op = opt.local_step_init_op
		if is_chief:
			# 主工作节点
			# 主工作节点负责初始化参数，模型的保存，概率的保存等
			local_init_op = opt.chief_init_op
		ready_for_local_init_op = opt.ready_for_local_init_op

		# 同步训练模式所需要的初始令牌和主队列
		chief_queue_runner = opt.get_chief_queue_runner()
		sync_init_op = opt.get_init_tokens_op()

	init_op = tf.global_variables_initializer()
	train_dir = tempfile.mkdtemp()


	if FLAGS.sync_replicas:
		# 创建一个监督管理程序，用于统计训练模型过程中的信息
		# logdir保存加载模型的路径
		# global_step的值是所有计算节点共享的
		# 在执行损失函数最小值的时候会自动加1，
		# 通过global_step能知道所有计算节点一共计算了多少步
		
		sv = tf.train.Supervisor(
			is_chief = is_chief,
			logdir = train_dir,
			init_op = init_op,
			local_init_op = local_init_op,
			ready_for_local_init_op = ready_for_local_init_op,
			recovery_wait_secs=1,
			global_step=global_step)
	else:
		sv = tf.train.Supervisor(
			is_chief=is_chief,
			logdir=train_dir,
			init_op = init_op,
			recovery_wait_secs=1,
			global_step=global_step)

	# 在创建会话时，设置属性allow_soft_placement为True
	# 所有的操作会默认使用期被指定的设备如GPU
	# 如果该操作函数没有GPU实现时，会自动使用CPU
	
	sess_config = tf.ConfigProto(
		all_soft_placement=True,
		log_device_placement=False,
		device_filters=['/job:ps','/job:worker/task:%d' %FLAGS.task_index])
	# 主工作节点（chief)即task_index=0的节点将会初始化会话
	# 其余的工作节点会等待会话被初始化后进行计算
	
	if is_chief:
		print("Worker %d:initializing sess ..." % FLAGS.task_index)
	else:
		print("Worker %d:Waiting for session to be initialized..."
			 % FLAGS.task_index)

	if FLAGS.existing_servers:
		# gRPC
		server_grpc_url = "grpc://"+worker_spec[FLAGS.task_index]
		print("Using existing server at: %s" % server_grpc_url)

		# 创建TF会话对象。用于执行图计算
		# prepare_or_wait_for_session需要参数初始化完成且主节点也准备好，才开始训练
		sess = sv.prepare_or_wait_for_session(server_grpc_url,config=sess_config)

	else:
		sess = sv.prepare_or_wit_for_session(server.target,config=sess_config)

	print("Worker %d: Session initialization complete." % FLAGS.task_index)

	if FLAGS.sync_replicas and is_chief:
		sess.run(sync_init_op)
		sv.start_queue_runners(sess,[chief_queue_runner])


	# 执行分布式模型训练
	time_begin = time.time()
	print("Training begin @ %f" % time_begin)

	local_step = 0

	while True:
		# 读入mnist训练数据
		batc_xs,batch_ys = mnist.train.next_batch(FLAGS.batch_size)
		train_feed = {x:batch_xs,y_:batch_ys}

		_,step = sess.run([train_step,global_step],feed_dict=train_feed)
		local_step += 1

		now = time.time()

		print("%f: worker %d: training step %d done (global step： %d)" 
			% (now,FLAGS.task_index,local_step,step)) 

		if step >= FLAGS.train_step:
			break

	time_end = time.time()

	print("Training ends @ %f" % time_end)
	training_time = time_end-time_begin
	print("Training elapsed time:%f s" % training_time)

	# 读入minist的验证数据，计算验证的交叉熵
	
	val_feed = {x:mnist.validation.images,y_:mnist.validation.labels}
	val_xent = sess.run(cross_entropy,feed_dict=val_feed)
	print("After %d training steps,validation cross entropy = %g" 
		% (FLAGS.train_steps,val_xent))

```

Reference: TensorFlow技术解析与实战
