---
layout: post
title: "TensorFlow多线程输入数据处理框架"
img: bowen57.png
date: 2018-11-25 12:00:00 
description: You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [TensorFlow,多线程]
---


在TensorFlow中，队列不仅仅是一种数据结构，他更提供了多线程的机制。

### 1.队列与多线程

**队列**
在TF中队列与变量类似，都是计算图上有状态的节点，其他计算节点可以修改他们额状态，对于变量可以通过赋值修改，对于队列可以通过：Enqueue,EnqueueMany,Dequeue，下面面代码展示这些函数操作队列的过程

```python
import tensorflow as tf

# 创建一个先进先出的队列，指定队列中最多可以保存2个元素，类型为整型
q = tf.FIFOQueue(2,"int32")

# 使用enqueue_many函数来初始化队列中的元素。和变量初始化类似，
# 在使用队列之前需要明确的调用这个初始化过程
init = q.enqueue_many(([0,10]))

# 使用Dequeue函数将队列中的第一个元素出对列，并复制给x
x = q.dequeue()

#将得到的值加1

y = x + 1

# 将加1后的值重新加入对列
q_inc  = q.enqueue([y])

with tf.Session() as sess:
    # 运行初始化队列的操作
    init.run()
    for _ in range(5):
        # 运行q_inc将至型数据出对列，出队列的元素+1.重新加入队列的过程
        v,_ = sess.run([x,q_inc])
        print(v)

```

TF中提供了两种对列FIFOQueue(先进先出),RandomShuffleQueue(随机出),TF对列不仅仅是一种数据结构，也是异步计算张量的一个重要机制，多个线程可以同时向一个队列中写元素或读元素

**多线程**

TF提供了tf.Coordinator和tf.QueueRunner两个类来完成多线程协同的功能，tf.Coordinator主要用于协同多个线程一起停止，并提供了should_stop,request_stop和join三个函数。should_stop这个函数返回值为True时，则当前线程需要退出。每一个启动的线程都可以通过调用request_stop函数来通知其他线程退出。

```python
import tensorflow as tf
import numpy as np
import threading
import time

# 线程中运行的程序，这个程序每隔1s判断是否需要停止并打印自己的ID
def MyLoop(coord,worker_id):
    # 使用tf.coordinator类提供的协同工具判断当前线程是否终止
    while nor coord.should_stop():
        # 随机停止所有的线程
        if np.random.rand() < 0.1:
            # 调用coord.request_stop()通知其他线程终止
            coord.request_stop()
        else:
            print("working on id:%d \n" % worker_id)
        time.sleep(1)\

# 声明一个tf.train.Coordinator类来协同过个线程
coord = tf.train.Coordinator()

# 声明创建5个线程
threads = [
 threading.Thread(target=MyLoop,args(coord,i)) for i in range(5)
]

# 启动所有的线程
for t in threads: t.start()

# 等待所有线程退出
coord.join(threads)

```

tf.QueueRunner主要用于启动多个线程来操作同一个队列，启动的这些线程可以通过上面介绍的tf.Coordinator类来统一管理。

```python
import tensorflow as tf

# 声明一个先进先出的队列。队列中最多100个元素，类型为实数
queue = tf.FIFOQueue(100,"float")
# 定义队列的入队操作
enqueue_op = queue.enqueue([tf.random_normal([1])])

# 使用tf.train.QueueRunner来穿件多个线程运行队列的入队操作
# 第一个参数给出了被操作的队列
# [enqueue_op]*5表示启动5个线程，每个鲜橙汁那个运行的是enqueue_op操作

qr = tf,train.QueueRunner(queue,[enqueue_op]*5)

# 将定义的QueueRunner加入TF计算图上指定的集合
# tf.train.add_queue_runner函数没有指定集合。
# 则默认加入集合tf.GraphKeys.QUEUE_RUNNERS

tf.train.add_queue_runner(qr)
# 定义出队操作
out_tensor = queue.dequeue()

with tf.Session() as sess:
    # 使用tf.train.Coordinator来协同启动线程
    coord = tf.trainCoordintor()]
    # 使用tf.train.QueueRunners时，需要明确调用tf.train.start_queue_runners
    # 来启动所有线程。否则没有线程做入队操作，当调用出队操作时
    # 程序将一直等待入队操做进行。
    # tf.train.start_queue_runners将默认启动tf.GraphKeys.QUEUE_RUNNERS中
    # 所有的QueueRunners

    threads =  tf.train.start_queue_runners(sess=sess,coord=coord)
    
    # 获取队列中的取值
    for _ in range(3):
        print(sess.run(out_tensor)[0]
    # 使用tf.train.Coordinator来停止所有线程
    coord.request_stop()
    coord.join(threads)
```
