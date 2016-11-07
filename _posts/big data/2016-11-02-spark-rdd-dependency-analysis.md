---
layout: post
title: Spark RDD源码解析 - RDD依赖关系
category: BigData
tags: [spark, RDD]
---
{% include JB/setup %}

## RDD依赖关系

RDD作为Spark计算模型的核心，被设计成包含以下几种信息的实体：

- A list of partitions
- A function for computing each split
- A list of dependencies on other RDDs
- Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)
- Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file)

其中的依赖关系指该RDD的每个分区计算时依赖哪些父RDD的分区，这些信息对于计算任务的调度、容错都非常重要。而根据不同的依赖关系，Spark将其划分为两种类型：窄依赖（narrow dependencies），父RDD的分区最多被一个子RDD的分区依赖；宽依赖（wide dependencies），子RDD的分区依赖于多个父RDD分区。进行依赖关系区分的意义在于：①窄依赖允许在一个集群结点上流水执行所有的计算，而宽依赖则需要首先计算好所有的父RDD数据，然后再在结点之间进行Shuffle操作；②在结点失效后进行恢复时，窄依赖会更加高效，只需要计算丢失的父分区即可，并且不同分区可以在多个结点上并行计算，而一个宽依赖的分区丢失时可能会导致整个过程的重新计算。

![窄依赖与宽依赖](/assets/post_img/bigdata/spark-dependencies.png)

在实现上，RDD对象的内部有一个Depedency对象的列表，而每个Depedency对象内部会存储一个RDD对象，对应一个父RDD。通过遍历RDD内部的Dependency列表即可获取该RDD所有依赖的父RDD。

```java
/**
 * :: DeveloperApi ::
 * Base class for dependencies.
 */
@DeveloperApi
abstract class Dependency[T] extends Serializable {
    def rdd: RDD[T]
}
```

RDD抽象类提供`dependencies`和`partitions`接口供开发者调用获取RDD对象的依赖及分区。同时这两个外部接口会调用内部接口`getDependencies`和`getPartitions`，RDD的子类需要实现这两个内部接口。


```java
abstract class RDD[T: ClassTag](
    @transient private var _sc: SparkContext,
    @transient private var deps: Seq[Dependency[_]]
  ) extends Serializable with Logging {

  ...

  // Our dependencies and partitions will be gotten by calling subclass's methods
  // below, and will be overwritten when we're checkpointed
  private var dependencies_ : Seq[Dependency[_]] = null
  @transient private var partitions_ : Array[Partition] = null

  ...

  /**
   * Implemented by subclasses to return the set of partitions in this RDD. This
   * method will only be called once, so it is safe to implement a time-consuming
   * computation in it.
   */
  protected def getPartitions: Array[Partition]

  /**
   * Implemented by subclasses to return how this RDD depends on parent RDDs. This
   * method will only be called once, so it is safe to implement a time-consuming
   * computation in it.
   */
  protected def getDependencies: Seq[Dependency[_]] = deps

  ...

  /**
   * Get the list of dependencies of this RDD, taking into account whether the
   * RDD is checkpointed or not.
   */
  final def dependencies: Seq[Dependency[_]] = {
    checkpointRDD.map(r => List(new OneToOneDependency(r))).getOrElse {
      if (dependencies_ == null) {
        dependencies_ = getDependencies
      }
      dependencies_
    }
  }

  /**
   * Get the array of partitions of this RDD, taking into account whether the
   * RDD is checkpointed or not.
   */
  final def partitions: Array[Partition] = {
    checkpointRDD.map(_.partitions).getOrElse {
      if (partitions_ == null) {
        partitions_ = getPartitions
      }
      partitions_
    }
  }
}
```

## 依赖类

Spark中用Dependency类来表示在经过转换操作后，父RDD与子RDD之间的联系（依赖关系）。如上所述，Dependency是一个只包含一个`def rdd: RDD[T]`方法的抽象类。它有如下几种派生类：

- NarrowDependency
    + OneToOneDependency
    + PruneDependency
    + RangeDependency
- ShuffleDependency

### ShuffleDependency

ShuffleDependency是一种会导致shuffle map stage（是一个在执行DAG时会执行shuffle操作的中间阶段，是后续阶段的输入）的依赖关系。

一个ShuffleDependency应该属于一个key-value对RDD（其rdd成员的类型为`RDD[Product2[K, V]]`）。而每一个ShuffleDependency对象都有一个**shuffleId**。

ShuffleDependency使用`partitiner`分割shuffle的输出。使用`ShuffleManager`来进行注册，以及`ContextCleaner`来注册自己进行清洗。每一个ShuffleDependency对象都使用其shuffleId和RDD的分区数目向`MapOutputTrackerMaster`注册。

会使用到ShuffleDependency的场景包括：

- 当多个RDD的`partitioner`不同时，生成`CoGroupedRDD`和`SubtractedRDD`
- 通过shuffle操作生成的`ShuffledRDD`或`ShuffledRowRDD`

可能用到也可能用不到上述RDD，而会进行shuffle的RDD操作有：

- coalesce
    + repartition
- cogroup
    + intersection
- subtractByKey
    + substract
- sortByKey
    + sortBy
- repartitionAndSortWithinPartitions
- combineByKeyWithClassTag
    + combineByKey
    + aggregateByKey
    + foldByKey
    + reduceByKey
    + countApproxDistinctByKey
    + groupByKey
- partitionBy

## 内部RDD类

Spark内部实现的RDD派生类有ParallelCollectionRDD, CoGroupedRDD, HadoopRDD, MapPartitionsRDD, CoalescedRDD, ShuffledRDD, PairRDD, UnionRDD等。

### ParallelCollectionRDD

ParallelCollectionRDD是一个拥有numSlices个分区，以及可选的locationPrefs选项的元素集合，这些元素分散存储在numSlices个分区中。一个ParallelCollectionRDD对象通常是`SparkContext.parallelize`和`SparkContext.makeRDD`方法的运行结果。因此，我们可以将其看作一个应用中**最初的RDD对象**。

### MapPartitionsRDD

MapPartitionsRDD是在父RDD的每个分区上执行给定函数f后得到的结果类型，对应其compute函数为对给定Partition调用f，进行计算。默认情况下，MapPartitionsRDD不会保留原始RDD的分区，当构造函数的最后一个参数preservesPartitioning是true的时候，保留原始分区。

```java
/**
 * An RDD that applies the provided function to every partition of the parent RDD.
 */
private[spark] class MapPartitionsRDD[U: ClassTag, T: ClassTag](
    var prev: RDD[T],
    f: (TaskContext, Int, Iterator[T]) => Iterator[U],  // (TaskContext, partition index, iterator)
    preservesPartitioning: Boolean = false)
  extends RDD[U](prev) {

  override val partitioner = if (preservesPartitioning) firstParent[T].partitioner else None

  override def getPartitions: Array[Partition] = firstParent[T].partitions

  override def compute(split: Partition, context: TaskContext): Iterator[U] =
    f(context, split.index, firstParent[T].iterator(split, context))

  override def clearDependencies() {
    super.clearDependencies()
    prev = null
  }
}
```

结果是MapPartitionsRDD的转换操作有：map, flatMap, filter, glom, mapPartitions, mapPartitionsWithIndex, PairRDDFunctions.mapValues, PairRDDFunctions.flatMapValues。

我们选择观察map和filter函数的实现，分析MapPartitionsRDD的生成过程。在RDD类中有map及filter函数的实现，其代码如下。可见，其生成过程为：创建一个MapPartitionsRDD对象，并将函数f作为参数传入，**并将f变形为对一个分区的元素迭代调用f（map地调用或filter地调用）**， 在真正（lazy compute）计算时，调用compute函数，对每个分区执行f计算。

```java
/**
 * Return a new RDD by applying a function to all elements of this RDD.
 */
def map[U: ClassTag](f: T => U): RDD[U] = withScope {
  val cleanF = sc.clean(f)
  new MapPartitionsRDD[U, T](this, (context, pid, iter) => iter.map(cleanF))
}

...

/**
 * Return a new RDD containing only the elements that satisfy a predicate.
 */
def filter(f: T => Boolean): RDD[T] = withScope {
  val cleanF = sc.clean(f)
  new MapPartitionsRDD[T, T](
    this,
    (context, pid, iter) => iter.filter(cleanF),
    preservesPartitioning = true)
}
```
