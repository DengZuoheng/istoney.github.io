---
layout: post
title: Spark Programming Guide 编程指南 v1.6.1
category: big data
tags: [spark]
---
{% include JB/setup %}

## Overview
从一个较高的角度来讲，每个Spark应用都包含一个驱动程序(driver program)，用来运行用户的main函数，并在集群上执行各种并行操作(parallel operations)。Spark中最主要的抽象概念是弹性数据集（Resilient Distributed Dataset, RDD），它是指分布在集群节点上，可以进行并行操作的元素集合。RDD可以从Hadoop文件系统（或者其他任何支持Hadoop文件系统的系统）上的文件开始创建，或者从driver程序中已经存在Scala数据集合上创建，然后进行转化。用户可以让Spark去持久化(persist)内存中的RDD，使其可以在并行操作中重用。此外，当节点失效时，节点上的RDD会自动恢复。

Spark提供的另外一个抽象概念是可以在并行操作中共享使用的共享变量(shared variables)。默认情况下，Spark会在不同的节点上并行的运行一个函数，也就是任务，并且会把函数中用到的变量拷贝一份分发到各个任务中。但是有时候，有一些变量需要在任务间共享，或者在任务和driver程序之间共享。Spark支持两种类型的共享变量：广播变量(broadcast variables)可以在每个节点的内存中缓存一个变量的值；累加器(accumulators)是一种只能进行“加”操作的变量，例如计数器和求和器。

该手册将会用所有Spark支持的语言展示上述特性。学习本文最简单的方式是启动Spark的交互Shell，不论bin/spark-shell或bin/pyspark，跟随文章内容进行练习。

P.s. 本文(翻译)将采用Python作为示例语言

## Linking with Spark
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

## Initializing Spark
Spark程序必须做的第一件事是创建一个[SparkContext](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkContext)对象，这个SparkContext对象会告诉Spark怎么去访问集群。为了创建SparkContext，首先需要创建一个[SparkConf](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkConf)对象，它将包含应用的信息。

```py
conf = SparkConf().setAppName(appName).setMaster(master)
sc = SparkContext(conf=conf)
```

appName参数是你的应用的名字，这个名字将会显示在集群的UI。master是一个[Spark, Mesos或YARN集群的URL](http://spark.apache.org/docs/latest/submitting-applications.html#master-urls)，或者是一个特殊的“local”字符串表示运行在本地模式。在实际应用中，当运行一个集群式，你不会希望将master硬编码在程序中，而是通过[spark-submit启动应用](http://spark.apache.org/docs/latest/submitting-applications.html)，并从命令参数中接收master。当然，对于本地测试和单元测试，你可以传入“local”在进程中运行Spark。

### Using the Shell
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

## Resilient Distributed Datasets (RDDs)
Spark的核心是弹性数据集(RDD)，RDD是一个可以进行并行操作的高容错性元素集合。共有两种方式创建RDD：将driver程序中已存的数据集合并行化(parallelizing)，或者引用外部存储系统中的数据集，例如，共享文件系统、HDFS、HBase，或者任何提供Hadoop InputFormat的数据源。

### Parallelized Collections
并行数据集合通过在driver程序中可迭代的数据集上调用SparkContext的parallelize方法创建。该数据集的元素将会被拷贝去构建可以并行操作的分布式数据集。例如，下面是如何创建一个包含数字1到5的并行数据集的示例：

```py
data = [1, 2, 3, 4, 5]
distData = sc.parallelize(data)
```

创建之后，分布式数据集(distData)就可以进行并行操作。例如，我们可以调用`distData.reduce(lambda a, b: a + b)`把列表中的所有元素相加求和。关于分布式数据集的操作，我们稍后讨论。

并行数据集的一个重要参数是分割(partitions)的数量。Spark会在集群上为每一个分割运行一个任务。通常，分割的时候，你会按照集群中平均每个CPU 2-4个分割来划分。一般，Spark会根据集群的情况自动设置数据分割的数量。但是，你也可以通过向parallelize传入第二个参数来手动设置（比如，sc.parallelize(data, 10))。

注意：在代码中的有些地方会使用分片（slices）（partitions的同义词）来保持后向兼容性。

### External Datasets
PySpark可以从任何Hadoop支持的存储系统创建分布式数据集，包括本地文件系统、HDFS、Cassandra、HBase、[Amazon S3](http://wiki.apache.org/hadoop/AmazonS3)等。Spark支持文本文件，[SequenceFiles](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html)，以及任何其他Hadoop输入格式（[InputFormat](http://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapred/InputFormat.html)）。

文本文件的RDD可以使用SparkContext的`textFile`方法创建。这个方法需要一个文件的URI（本地机器的文件路径，或者是`hdfs://`，`s3n://`等URI），然后读入该文件，作为以行为基本单位的数据集合。下面是一个调用示例：

```py
>>> distFile = sc.textFile("data.txt")
```

创建之后就可以在`distFile`上进行数据集操作了。例如，我们可以使用`map`和`reduce`操作把所有行的大小加起来：`distFile.map(lambda s: len(s)).reduce(lambda a, b: a + b)`。

在Spark中读取文件需要注意的几个地方：

- 如果使用本地文件系统路径，必须保证在worker节点上的相同路径上该文件也是可以访问的。或者把该文件拷贝到所有worker节点上，或者使用网络挂载的共享文件系统。
- Spark所有基于文件的输入方法，包括`textFile`，都持支目录、压缩文件和通配符。例如，你可以使用`textFile("/my/directory")`，`textFile("/my/directory/*.txt")`和`textFile("/my/directory/*.gz")`。
- `textFile`方法还支持一个可选的参数来控制文件分割的数量。默认情况下，Spark会为文件的每个块（block）创建一个分割（HDFS中默认的文件块大小为64MB），但是你也可以通过传入一个更大的值使其分割的数量更多。需要注意的是，分割的数量不能小于文件块的数量。

除了文本文件，Spark的Python API还支持几种其他的格式：

- `SparkContext.wholeTextFiles`可以读取一个包含多个小的文本文件的目录，然后为每个文件返回一个(filename, content)的值对。这个方法和`textFile`形成一个对比，`textFile`方法将每个文件的每一行作为一条记录返回。
- `RDD.saveAsPickleFile`和`SparkContext.pickleFile`支持将RDD以序列化的Python对象的简单格式保存起来。序列化操作会被批量执行，批量处理的默认数量为10.
- 序列文件（SequenceFile）和Hadoop的输入/输出格式（Input/Output Formats）。

注意：这一特性目前被标记为“实验的”（`Experimental`）并向高级用户提供的。将来可能会被基于Spark SQL的读写（read/write）支持而取代，在这种情景下Spark SQL是首选的方式。

#### # Writable Support
PySpark SequenceFile支持在Java中载入一个RDD的键值对，将可写类型转换成Java基本类型，以及使用[Pyrolite](https://github.com/irmen/Pyrolite/)序列化Java的结果对象。当把一个RDD键值对保存为SequenceFile时，PySpark会指向上述过程的反过程。它把Python对象反序列化成Java对象，然后将其转换成可写类型。下列可写类型将会自动转化：

| Writable Type | Python Type |
|:--------------|:------------|
|Text|unicode str|
|IntWritable    |int|
|FloatWritable  |float|
|DoubleWritable |float|
|BooleanWritable |bool|
|BytesWritable  |bytearray|
|NullWritable   |None|
|MapWritable    |dict|

数组类型并不支持自动转换。当读写数组时，用户需要自定义`ArrayWritable`的子类型，以及写入时将数组转换为自定义的`ArrayWritable`的转换器，和读入时将`ArrayWritable`转换为Java对象数组，并序列化为Python元组的转换器。想要从主要类型数组得到Python `array.array`，用户需要自定义转换器。

#### # Saving and Loading SequenceFiles
和文本文件类似，序列文件可以在指明的路径上保存和加载。键和值的类型都可以指定，但是对于标准的可写类型不需要指定。

```py
>>> rdd = sc.parallelize(range(1, 4)).map(lambda x: (x, "a" * x))
>>> rdd.saveAsSequenceFile("path/to/file")
>>> sorted(sc.sequenceFile("path/to/file").collect())
[(1, u'a'), (2, u'aa'), (3, u'aaa')]
```

#### # Saving and Loading Other Hadoop Input/Output Formats
PySpark还可以读入任何hadoop输入格式（InputFormat），以及写入任何Hadoop输出格式（OutputFormat），不管是新的，还是旧的Hadoop MapReduce API都可以。如果需要，可以以Python dict的形式传入一个Hadoop配置。下面是使用Elasticsearch ESInputFormat的例子：

```py
$ SPARK_CLASSPATH=/path/to/elasticsearch-hadoop.jar ./bin/pyspark
>>> conf = {"es.resource" : "index/type"}   # assume Elasticsearch is running on localhost defaults
>>> rdd = sc.newAPIHadoopRDD("org.elasticsearch.hadoop.mr.ESInputFormat", \
          "org.apache.hadoop.io.NullWritable", "org.elasticsearch.hadoop.mr.LinkedMapWritable", conf=conf)
>>> rdd.first()     # the result is a MapWritable that is converted to a Python dict
(u'Elasticsearch ID',
 {u'field1': True,
  u'field2': u'Some Text',
  u'field3': 12345})
```

注意到，如果输入格式只是简单的依赖于Hadaoop配置或者输入路径，，并且键和值的类型可以根据上表简单的转换，那么对于这种情形，这种方法可以很好的应对。

如果你有自定义的串行二进制文件（比如从Cassandra或HBase导入），那么你首先需要在Scala/Java方面将其转换成Pyrolite序列化器可以处理的类型。Spark为这种情况提供了一个[Converter](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.api.python.Converter)接口。只要继承这个接口，并且在`convert`方法中实现你的转换代码就可以了。不过，一定要记得把这个类，还有访问你的`InputFormat`需要的任何依赖，打包添加到你的Spark任务的jar包中，然后把他们添加到PySpark的classpath上。

对于Cassandra/HBase `InputFormat`和包含自定义转换器的`OutputFormat`的使用，请查看[Python 示例](https://github.com/apache/spark/tree/master/examples/src/main/python)和[Converted 示例](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples/pythonconverters)。

### RDD Operations
RDD支持两种类型的操作：转化（`transformations`），从一个已存的数据集创建新的数据集；动作（`actions`），在数据集上运行一个计算之后想driver程序返回一个结果。例如，`map`是一个转化，它将所有数据集元素传入一个函数，并生成一个新RDD表示所有的结果。另一方面，`reduce`是一个动作，它通过一些函数收集所有RDD的元素，然后将最终结果返回给driver程序（尽管还有一个并行的`reduceByKey`操作是返回值是分布式数据集）。

Spark中的所有转化都是懒惰的（lazy），即所有计算并不会被立即执行，而是记录下载基础数据集（例如文件）上执行的转化，只有当一个动作被调用，并且需要有一个结果返回给driver程序时这些转化才会执行。这样的设计使得Spark更加高效，例如，当我们发现一个通过`map`创建的数据集将会执行`reduce`操作，这时只需要返回`reduce`的结果，而不需要将`map`计算得到的大数据集返回。

默认情况下，每一个转化的RDD都会在执行动作是重新计算。但是，你可以使用`persist`(或`cache`)方法在内存中持久化（persist）一个RDD，这时，Spark就会在集群中保留这些元素，以便下次查询是可以快速访问。另外，还支持在硬盘上持久化RDD，或者在多个节点上保存副本。

#### Basics

##### Passing Functions to Spark

##### Understanding closures

###### Example

###### Local vs. cluster modes

###### Printing elements of an RDD

##### Working with Key-Value Pairs

###### Transformations

###### Actions

###### Shuffle operations

某些Spark操作会触发shuffle（洗牌）。Shuffle是Spark的一个数据再分配机制，使分区之间的数据重新分组。这一操作通常会在executor（执行器）或机器之间拷贝数据，导致shuffle成为复杂而费时的操作。

####### Background

为了理解在shuffle中发生了什么，我们以reduceByKey操作作为例子进行观察。reduceByKey操作会生成一个新的RDD，在其中同一个key的所有value组合成一个元组——key和在与所有与key相关的value上执行reduce函数的结果。挑战在于，与一个key相关的所有value并不会全部都在同一个分区内，甚至不在一个机器上，但是它们必须同地协作来计算结果。

在Spark中，数据通常不会为了某个特别的操作而在区分之间进行特别的分布。在计算中，单个任务只在单个分区上进行，因此，为了单个reduceByKey任务组织所有数据，Spark需要进行一个all-to-all操作。该操作从所有的分区中读取所有key的value，并将其组织到一起为每个key计算最终结果，这一过程称为shuffle。

尽管新shuffle的数据的每个分区的元素集合是确定的，以及分区的顺序，但是这些元素之间并没有顺序。如果在shuffle时需要数据保持一定的顺序，你可能需要：

- mapPartitions，使用如.sorted对每个分区进行排序
- repartitionAndSortWithinPartitions在并发再分区时对分区进行排序
- sortBy来生成全局有序的RDD

会引发shuffle的操作包括再分区操作，如`repartition`和`coalesce`，·ByKey操作（出了计数），比如`groupByKey`和`reduceByKey`，以及join操作，例如`cogroup`和`join`。
