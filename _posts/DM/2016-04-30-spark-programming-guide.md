---
layout: post
title: Spark Programming Guide 翻译
catergory: data_mining
tags: [spark]
---
{% include JB/setup %}

## # Overview
从一个较高的角度来讲，每个Spark应用都包含一个驱动程序(driver program)，用来运行用户的main函数，并在集群上执行各种并行操作(parallel operations)。Spark中最主要的抽象概念是弹性数据集（Resilient Distributed Dataset, RDD），它是指分布在集群节点上，可以进行并行操作的元素集合。RDD可以从Hadoop文件系统（或者其他任何支持Hadoop文件系统的系统）上的文件开始创建，或者从driver程序中已经存在Scala数据集合上创建，然后进行转化。用户可以让Spark去持久化(persist)内存中的RDD，使其可以在并行操作中重用。此外，当节点失效时，节点上的RDD会自动恢复。

Spark提供的另外一个抽象概念是可以在并行操作中共享使用的共享变量(shared variables)。默认情况下，Spark会在不同的节点上并行的运行一个函数，也就是任务，并且会把函数中用到的变量拷贝一份分发到各个任务中。但是有时候，有一些变量需要在任务间共享，或者在任务和driver程序之间共享。Spark支持两种类型的共享变量：广播变量(broadcast variables)可以在每个节点的内存中缓存一个变量的值；累加器(accumulators)是一种只能进行“加”操作的变量，例如计数器和求和器。

该手册将会用所有Spark支持的语言展示上述特性。学习本文最简单的方式是启动Spark的交互Shell，不论bin/spark-shell或bin/pyspark，跟随文章内容进行练习。

P.s. 本文(翻译)将采用Python作为示例语言

## # Linking with Spark
Spark 1.6.1可以在Python 2.6+和Python 3.4+上工作。可以使用标准的CPython解释器，所以C语言库，例如NumPy都可以使用，同时Spark还可以和PyPy 2.3+一起工作。

当在Python中运行Spark应用时，使用Spark目录中的bin/spark-submit脚本。这个脚本会加载Spark的Java/Scala库，并允许你向集群提交应用。你还可以使用bin/pyspark来启动交互式的Python Shell。

如果你希望访问HDFS数据，你需要使用一个与你的HDFS版本绑定构建的PySpark。为常用的HDFS版本构建的预构建包可以在Spark主页[下载](http://spark.apache.org/downloads.html)。

最后，你需要在你的程序中导入(import)一些Spark类，并添加如下代码：

```py
from pyspark import SparkContext, SparkConf
```

PySpark要求driver和worker上运行的Python的次版本(minor version)是相同的。它使用PATH变量中设置的默认Python版本，你可以通过PYSPARK_PYTHON变量来指明你想要使用的Python的版本，例如：

```sh
$ PYSPARK_PYTHON=python3.4 bin/pyspark
$ PYSPARK_PYTHON=/opt/pypy-2.5/bin/pypy bin/spark-submit example/src/main/python/pi.py
```

## # Initializing Spark
Spark程序必须做的第一件事是创建一个[SparkContext](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkContext)对象，这个SparkContext对象会告诉Spark怎么去访问集群。为了创建SparkContext，首先需要创建一个[SparkConf](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkConf)对象，它将包含应用的信息。

```py
conf = SparkConf().setAppName(appName).setMaster(master)
sc = SparkContext(conf=conf)
```

appName参数是你的应用的名字，这个名字将会显示在集群的UI。master是一个[Spark, Mesos或YARN集群的URL](http://spark.apache.org/docs/latest/submitting-applications.html#master-urls)，或者是一个特殊的“local”字符串表示运行在本地模式。在实际应用中，当运行一个集群式，你不会希望将master硬编码在程序中，而是通过[spark-submit启动应用](http://spark.apache.org/docs/latest/submitting-applications.html)，并从命令参数中接收master。当然，对于本地测试和单元测试，你可以传入“local”在进程中运行Spark。

## # Using the Shell
在PySpark Shell中，一个特殊的，解释器已知的SparkContext对象已经创建好了，叫做sc。这时，你自己创建的SparkContext对象是不会工作的。你可以通过--master参数来设置sc要连接的集群，你还可以通过--py-files参数添加.zip, .egg或者.py等Python文件到运行路径上(多个文件需要用逗号分隔)。你还可以通过--packages参数和一个逗号分隔的maven坐标列表向你的Shell会话添加依赖（Spark包等）。另外的资源库(repositories)可以通过--repositories参数添加。必要的时候，所有Spark包依赖的Python包(Spark包中的requirements.txt文件列出的)都必须用pip进行手动安装。例如，在四个内核上运行bin/pyspark，使用命令：

```sh
$ ./bin/pyspark --master local[4]
```

或者，把code.py添加到搜索路径上（为了稍后可以导入code），使用：

```sh
$ ./bin/pyspark --master local[4] --py-files code.py
```

如果想看到完整的选项列表，运行`pyspark --help`命令，而pyspark实际上是在后面调用了更加通用的[spark-submit脚本](http://spark.apache.org/docs/latest/submitting-applications.html)。

同时，在增强版的Python解释器[IPython](http://ipython.org/)中启动PySpark Shell也是可以得。PySpark可以在1.0.0以上版本的IPython中工作。想要使用IPython，需要在运行bin/pyspark是设置PYSPARK_DRIVER_PYTHON变量为ipython：

```sh
$ PYSPARK_DRIVER_PYTHON=ipython ./bin/pyspark
```

你可以通过设置PYSPARK_DRIVER_PYTHON_OPTS变量来自定义ipython命令。例如，基于PyLab画图的支持启动[IPython Notebook](http://ipython.org/notebook.html)：

```sh
$ PYSPARK_DRIVER_PYTHON=ipython PYSPARK_DRIVER_PYTHON_OPTS="notebook" ./bin/pyspark
```

在IPython Notebook服务器启动后，你可以在“File”菜单中创建一个“Python 2”记事本。在记事本中，你可以输入命令`%pylab inline`作为记事本的一部分，然后从IPython记事本启动Spark。

## # Resilient Distributed Datasets (RDDs)
Spark的核心是弹性数据集(RDD)，RDD是一个可以进行并行操作的高容错性元素集合。共有两种方式创建RDD：将driver程序中已存的数据集合并行化(parallelizing)，或者引用外部存储系统中的数据集，例如，共享文件系统、HDFS、HBase，或者任何提供Hadoop InputFormat的数据源。

### # Parallelized Collections
并行数据集合通过在driver程序中可迭代的数据集上调用SparkContext的parallelize方法创建。该数据集的元素将会被拷贝去构建可以并行操作的分布式数据集。例如，下面是如何创建一个包含数字1到5的并行数据集的示例：

```py
data = [1, 2, 3, 4, 5]
distData = sc.parallelize(data)
```

创建之后，分布式数据集(distData)就可以进行并行操作。例如，我们可以调用`distData.reduce(lambda a, b: a + b)`把列表中的所有元素相加求和。关于分布式数据集的操作，我们稍后讨论。

并行数据集的一个重要参数是分割的块(partitions)的数量。Spark会在集群上为每一个块运行一个任务。通常，划分块的时候，你会按照集群中平均每个CPU 2-4块来划分。一般，Spark会根据集群的情况自动设置数据块的数量。但是，你也可以通过向parallelize传如第二个参数来手动设置（比如，sc.parallelize(data, 10))。

注意：在代码中的有些地方会使用分片（slices）（块partitions的同义词）来保持后向兼容性。

### # External Datasets
PySpark可以从任何Hadoop支持的存储系统创建分布式数据集，包括本地文件系统、HDFS、Cassandra、HBase、[Amazon S3](http://wiki.apache.org/hadoop/AmazonS3)等。
