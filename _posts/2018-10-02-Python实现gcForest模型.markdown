---
layout: post
title: "Python实现gcForest模型"
img: bowen54.png
date: 2018-10-02 12:00:00 
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python, gcForest]
---

## 1.介绍

gcForest v1.1.1是gcForest的一个官方托管在GitHub上的版本，是由Ji Feng(Deep Forest的paper的作者之一)维护和开发，该版本支持Python3.5,且有类似于Scikit-Learn的API接口风格，在该项目中提供了一些调用例子，目前支持的基分类器有RandomForestClassifier,XGBClassifer,ExtraTreesClassifier,LogisticRegression,SGDClassifier如果采用XGBoost的基分类器还可以使用GPU,如果想增加其他基分类器，可以在模块中的`lib/gcforest/estimators/__init__.py`中添加,使用该模块需要依赖安装如下模块：

+ argparse
+ joblib
+ keras
+ psutil
+ scikit-learn>=0.18.1
+ scipy
+ simplejson
+ tensorflow
+ xgboost


## 2.API调用样例

这里先列出gcForest提供的API接口：

+ fit_tranform(X_train,y_train) 是gcForest模型最后一层每个估计器预测的概率concatenated的结果

+ fit_transform(X_train,y_train,X_test=x_test,y_test=y_test) 测试数据的准确率在训练的过程中也会被记录下来

+ set_keep_model_mem(False) 如果你的缓存不够，把该参数设置成False(默认为True),如果设置成False,你需要使用fit_transform(X_train,y_train,X_test=x_test,y_test=y_test)来评估你的模型

+ predict(X_test) # 模型预测

+ transform(X_test)


最简单的调用gcForest的方式如下：

```python

# 导入必要的模块
from gcforest.gcforest import GCForest

# 初始化一个gcForest对象
gc = GCForest(config) # config是一个字典结构

# gcForest模型最后一层每个估计器预测的概率concatenated的结果
X_train_enc = gc.fit_transform(X_train,y_train)

# 测试集的预测
y_pred = gc.predict(X_test)
```

下面我们使用MNIST数据集来演示gcForest的使用及代码的详细说明：

```python
# 导入必要的模块

import argparse # 命令行参数调用模块
import numpy as np 
import sys
from keras.datasets import mnist # MNIST数据集
import pickle
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score
sys.path.insert(0, "lib")

from gcforest.gcforest import GCForest
from gcforest.utils.config_utils import load_json


def parse_args():
	'''
	解析终端命令行参数(model)
	'''
    parser = argparse.ArgumentParser()
    parser.add_argument("--model", dest="model", type=str, default=None, 
	help="gcfoest Net Model File")
    args = parser.parse_args()
    return args


def get_toy_config():
	'''
	生成级联结构的相关结构
	'''
    config = {}
    ca_config = {}
    ca_config["random_state"] = 0
    ca_config["max_layers"] = 100
    ca_config["early_stopping_rounds"] = 3
    ca_config["n_classes"] = 10
    ca_config["estimators"] = []
    ca_config["estimators"].append(
            {"n_folds": 5, "type": "XGBClassifier", "n_estimators": 10, 
		"max_depth": 5,"objective": "multi:softprob", "silent": 
		True, "nthread": -1, "learning_rate": 0.1} )
    ca_config["estimators"].append({"n_folds": 5, "type": "RandomForestClassifier", 
	"n_estimators": 10, "max_depth": None, "n_jobs": -1})
    ca_config["estimators"].append({"n_folds": 5, "type": "ExtraTreesClassifier",
	 "n_estimators": 10, "max_depth": None, "n_jobs": -1})
    ca_config["estimators"].append({"n_folds": 5, "type": "LogisticRegression"})
    config["cascade"] = ca_config
    return config

# get_toy_config()生成的结构，如下所示：

'''
{
"cascade": {
    "random_state": 0,
    "max_layers": 100,
    "early_stopping_rounds": 3,
    "n_classes": 10,
    "estimators": [
        {"n_folds":5,"type":"XGBClassifier","n_estimators":10,"max_depth":5,
		"objective":"multi:softprob", "silent":true, 
		"nthread":-1, "learning_rate":0.1},
        {"n_folds":5,"type":"RandomForestClassifier","n_estimators":10,
		"max_depth":null,"n_jobs":-1},
        {"n_folds":5,"type":"ExtraTreesClassifier","n_estimators":10,
		"max_depth":null,"n_jobs":-1},
        {"n_folds":5,"type":"LogisticRegression"}
    ]
}
}
'''

if __name__ == "__main__":
    args = parse_args()
    if args.model is None:
        config = get_toy_config()
    else:
        config = load_json(args.model)

    gc = GCForest(config)
    # 如果模型消耗太大内存，可以使用如下命令使得gcforest不保存在内存中
    # gc.set_keep_model_in_mem(False), 默认情况下是True.

    (X_train, y_train), (X_test, y_test) = mnist.load_data()
    # X_train, y_train = X_train[:2000], y_train[:2000]
    # np.newaxis相当于增加了一个维度
    X_train = X_train[:, np.newaxis, :, :]
    X_test = X_test[:, np.newaxis, :, :]


    X_train_enc = gc.fit_transform(X_train, y_train)
    # X_enc是gcForest模型最后一层每个估计器预测的概率concatenated的结果
    # X_enc.shape =
    #   (n_datas, n_estimators * n_classes): 如果是级联结构
    #   (n_datas, n_estimators * n_classes, dimX, dimY): 如果只有多粒度扫描结构

    # 可以在fit_transform方法中加入X_test,y_test,这样测试数据的准确率在训练的过程中
    # 也会被记录下来。
    # X_train_enc, X_test_enc = 
	gc.fit_transform(X_train, y_train, X_test=X_test, y_test=y_test)

    # 注意: 如果设置了gc.set_keep_model_in_mem(True),必须使用
    # gc.fit_transform(X_train, y_train, X_test=X_test, y_test=y_test)
    # 评估模型

    # 测试集预测与评估
    y_pred = gc.predict(X_test)
    acc = accuracy_score(y_test, y_pred)
    print("Test Accuracy of GcForest = {:.2f} %".format(acc * 100))

    # 可以使用gcForest得到的X_enc数据进行其他模型的训练比如xgboost/RF
    # 数据的concat
    X_test_enc = gc.transform(X_test)
    X_train_enc = X_train_enc.reshape((X_train_enc.shape[0], -1))
    X_test_enc = X_test_enc.reshape((X_test_enc.shape[0], -1))
    X_train_origin = X_train.reshape((X_train.shape[0], -1))
    X_test_origin = X_test.reshape((X_test.shape[0], -1))
    X_train_enc = np.hstack((X_train_origin, X_train_enc))
    X_test_enc = np.hstack((X_test_origin, X_test_enc))

    print("X_train_enc.shape={}, X_test_enc.shape={}".format(X_train_enc.shape,
	 X_test_enc.shape))

    # 训练一个RF
    clf = RandomForestClassifier(n_estimators=1000, max_depth=None, n_jobs=-1)
    clf.fit(X_train_enc, y_train)
    y_pred = clf.predict(X_test_enc)
    acc = accuracy_score(y_test, y_pred)
    print("Test Accuracy of Other classifier using 
	gcforest's X_encode = {:.2f} %".format(acc * 100))

    # 模型写入pickle文件
    with open("test.pkl", "wb") as f:
        pickle.dump(gc, f, pickle.HIGHEST_PROTOCOL)

    # 加载训练的模型
    with open("test.pkl", "rb") as f:
        gc = pickle.load(f)
    y_pred = gc.predict(X_test)
    acc = accuracy_score(y_test, y_pred)
    print("Test Accuracy of GcForest (save and load) = {:.2f} %".format(acc * 100))

```

这里需要注意的是gcForest不但可以对传统的结构化的2维数据建模，还可以对非结构化的数据比如图像，序列化的文本数据，音频数据等进行建模，但要注意数据维度的设定：

+ 如果仅使用级联结构，X_train,X_test对于2-D数组其维度为(n_samples,n_features);3-D或4-D数组会自动reshape为2-D，例如MNIST数据(60000,28,28)会reshape为(60000,784),(60000,3,28,28)会reshape为(60000,2352)。

+ 如果使用多粒度扫描结构，X_train,X_test必须是4—D的数组,图像数据其维度是(n_samples,n_channels,n_height,n_width)；序列数据其维度为(n_smaples,n_features,seq_len,1),例如对于IMDB数据，n_features为1，对于音频MFCC特征，其n_features可以为13,26等。


上述代码可以通过两种方式运行：

+ 一种方式是通过json文件定义模型结构，比如级联森林结构，只需要写一个json文件如代码中显示的结构，然后通过命令行运行`python examples/demo_mnist.py --model examples/demo_mnist-gc.json`就可以完成训练；如果既使用多粒度扫面又使用级联结构，那么需要同时把多粒度扫描的结构定义出来。
+ 定义好的json可以通过模块中的load_json()方法加载，然后作为参数初始化模型，如下：

```python
config = load_json(your_json_file)
gc = GCForest(config) 
```

+ 另一种方式是直接通过Python代码定义模型结构，实际上模型结构就是一个字典数据结构，即是上述代码中的`get_toy_config()`方法。


## 3.小结

通过gcForest v1.1.1 Python模块，结合gcForest模型的结构，演示了如何使用gcForest模块构建Deep Forest模型。gcForest不论是处理传统的机器学习任务还是非结构化数据的挖掘都有不错的表现。关于gcForest模块的详细介绍和源码，读者可以参考参考文献中所列的项目地址进行详细的学习。

## 4.参考文献

[1].<https://github.com/kingfengji/gcForest>



