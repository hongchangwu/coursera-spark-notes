# Week 03

## Shuffling

Some operations, e.g. `groupBy` or `groupByKey`, returns `ShuffledRDD`:

```scala
val pairs = sc.parallelize(List((1, "one"), (2, "two"), (3, "three")))
pairs.groupByKey()
// res2 : org.apache.spark.rdd.RDD[(Int, Iterable[String])]
//   = ShuffledRDD[16] at groupByKey at <console>:37
```

### Grouping vs Reducing

Given

```scala
case class CFFPurchase(customerId: Int, destination: String, price: Double)
```

Assume we have an RDD of the purchases that users of the CFF's (Swiss train
company), we want to calculate how many trips, and how much money was spent by
each individual customer over the course of the month.

- **Grouping**: `groupByKey` moves a lot of data from one node to another to
  ge "grouped" with its key. Doing so is called "shuffling" and is expensive.

  ```scala
  val purchasesRdd: RDD[CFFPurchase] = sc.textFile(...)

  val purchasesPerMonth =
    purchasesRdd.map(p => (p.customerId, p.price)) // Pair RDD
                .groupByKey() // groupByKey returns RDD[(K, Iterable[V])]
                .map(p -> (p._1, (p._2.size, p._2.sum)))
                .collect()
  ```

- **Reducing**: `reduceByKey` reduces dataset locally first, therefore the
  amount of data sent over the network during the shuffle is greatly reduced.

  ```scala
  val purchasesRdd: RDD[CFFPurchase] = sc.textFile(...)

  val purchasesPerMonth =
    purchasesRdd.map(p => (p.customerId, p.price)) // Pair RDD
                .reduceByKey((v1, v2) => (v1._1 + v2._1, v1._2 + v2._2))
                .collect()
  ```

## Partitioning

### Partitions

**Properties of partitions:**
- Partitions never span multiple machines.
- Each machine in the cluster contains one or more partitions.
- The number of partitions to use is configurable. By default, it equals to
  the _total number of cores on all executor nodes._

**Two kinds of partitioning available in Spark:**
- Hash partitioning
- Range partitioning

###  Hash partitioning

Hash partitioning attempts to spread data evenly across partitions _based on
the key._

For example, `groupByKey` first computes per tuple `(k, v)` its partition `p`:

```scala
p = k.hashCode() % numPartitions
```

### Range partitioning

Pair RDDs may contain keys that have an _ordering_ defined (e.g. `Int`,
`Char`, `String`).

For such RDDs, _range partitioning_ may be more efficient. Using a range
partitioner, keys are partitioned according to:
1. an _ordering_ for keys
2. a set of _sorted ranges_ of keys

### Partitioning Data

Two ways to create RDDs with specific partitionings:

1. Call `partitionBy` on an RDD, providing an explicit `Partitioner`.
   Example:
   
   ```scala
   val pairs = purchasesRdd.map(p => (p.customerId, p.price))

   val tunedPartitioner = new RangePartitioner(8, pairs)
   val partitioned = paris.partitionBy(tunedPartitioner).persist()
   ```
   
   Notice the `persist` call. This is to prevent data being shuffled across
   the network over and again.
2. Using transformations that returns RDDs with specific partitions.

   **Partitioner from parent RDD:**
   Pair RDDs that are the rsult of a transformation of a _partitioned_ Pair
   RDD typically is configured to use the has partitioner that was used to
   construct the parent.

   **Automatically-set partitioners:**
   e.g.
   - When using `sortByKey`, a `RangePartitioner` is used.
   - When using `groupByKey`, a `HashPartitioner` is used by default.

### Operations on Pair RDDs that hold to (and propagate) a partitioner:

- `cogroup`
- `groupWith`
- `join`
- `leftOuterJoin`
- `rightOuterJoin`
- `groupByKey`
- `reduceByKey`
- `foldByKey`
- `combineByKey`
- `partitionBy`
- `sort`
- `mapValues` (if parent has a partitioner)
- `flatMapValues` (if parent has a partitioner)
- `filter` (if parent has a partitioner)

**All other operations will produce a result without a partitioner.**

## Optimizing with Partitioners

It is possible to optimize our previous example using range partitioners so
that it does not involve any shufflign over the network at all!

```scala
  val pairs = purchasesRdd.map(p => (p.customerId, p.price))
  val tunedPartitioner = new RangePartitioner(8, pairs)

  val partitioned = pairs.partitionBy(tunedPartitioner)
                         .persist()
                         
  val purchasesPerCust =
    partitioned.map(p => (p._1, (1, p._2)))

  val purchasesPerMonth =
    purchasesPerCust.reduceByKey((v1, v2) => (v1._1 + v2._1, v1._2 + v2._2))
                    .collect()
```

**You can figure out whether a shuffle has been planned/executed via:**
1. The return type of certain transformations, _e.g.,_

  ```scala
  org.apache.spark.rdd.RDD[(String, Int)] = ShuffledRDD[366]
  ```

2. Using function `toDebugString` to see its execution plan:

   ```scala
   partitioned.reduceByKey((v1, v2) => (v1._1 + v2._1, v1._2 + v2._2))
              .toDebugString
   res9: String =
   (8) MapPartitionsRDD[622] at reduceByKey at <console>:49 []
    |  ShuffledRDD[615] at partitionBy at <console>:48 []
    |      CachedPartitions: 8; MemorySize: 1754.8 MB; DiskSize: 0.0 B
   ```

### Operations that _might_ cause a shuffle

- `cogroup`
- `groupWith`
- `join`
- `leftOuterJoin`
- `rightOuterJoin`
- `groupByKey`
- `reduceByKey`
- `combineByKey`
- `distinct`
- `intersection`
- `repartition`
- `coalesce`

**2 Examples to avoid shuffling by partitioning:**
1. `reduceByKey` running on a pre-partitoined RDD will cause the values to be
   computed _locally_, requiring only the final reduced value to be sent from
   the worker to the driver.
2. `join` called on two RDDs that are pre-partitioned with the same
   partitioner and cached on the same machine will cause the `join` to be
   computed _locally_, with no shuffling across the network.
   the network

## Wide vs Narrow Dependencies

### Lineages

Computations on RDDs are represented as a **lineage graph**: a Directed
Acyclic Graph (DAG) representing the computations done on the RDD.

### How are RDDs represented?

RDDs are represented as:
- **Partitions**. Atomic pieces of the dataset. One or many per compute node.
- **Dependencies**. Models relationship between this RDD and its partitions
  with the RDD(s) it was derived from.

**Transformations cause shuffles**. Transformations can have two kinds of
dependencies:

1. Narrow Dependencies
   Each partition of the parent RDD is used by at most one partition of the
   child RDD.

   **Fast! No shuffle necessary. Optimizations like pipelining possible.**

   Examples:
   - `map`
   - `filter`
   - `union`
   - `join` with co-partitioned inputs

2. Wide Dependencies
   Each partition of the parent RDD may be depended on by **multiple** child
   partitions. 

   **Slow! Requires all or some data to be shuffled over the network.**

   Examples:
   - `groupByKey`
   - `join` with inputs not co-partitioned

**Transformations with narrow dependencies:**
- `map`
- `mapValues`
- `flatMap`
- `filter`
- `mapPartitions`
- `mapPartitionsWithIndex`

**Transformations with wide dependencies:** (_might cause a shuffle_)
- `cogroup`
- `groupWith`
- `join`
- `leftOuterJoin`
- `rightOuterJoin`
- `groupByKey`
- `reduceByKey`
- `combineByKey`
- `distinct`
- `intersection`
- `repartition`
- `coalesce`

### Find out the actual dependencies

Use the **dependencies** method on RDDs. It returns a sequence of `Dependency`
objects, which are used by Spark's scheduler to know how the RDD depends on
other RDDs.

**Narrow dependency objects:**
- `OneToOneDependency`
- `PruneDependency`
- `RangeDependency`

**Wide dependency objects:**
- `ShuffleDependency`

Example:

```scala
val wordRdd = sc.parallelize(largeList)
val pairs = wordRdd.map(c => (c, 1))
                   .groupByKey()
                   .dependencies
// pairs: Seq[org.apache.spark.Dependency[_]] =
// List(org.apache.spark.ShuffleDependency@4294a23d)
```

You can also the **`toDebugString`** method on RDDs as mentioned before. It
prints out a visualization of the RDD's lineage.
