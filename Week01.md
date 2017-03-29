# Week 01

## Book & Resources

- [Learning Spark](http://shop.oreilly.com/product/0636920028512.do)
- [Spark in Action](https://www.manning.com/books/spark-in-action)
- [High Performance Spark](http://shop.oreilly.com/product/0636920046967.do)
- [Advanced Analytics with Spark](http://shop.oreilly.com/product/0636920035091.do)
- [Mastering Apache Spark 2](https://www.gitbook.com/book/jaceklaskowski/mastering-apache-spark/details)

## Tools

- [Databricks Community Edition](https://community.cloud.databricks.com/)

## Data Parallelism

Scala's Parallel Collections is an abstraction over shared memory
  data-parallel execution.

Like parallel collections, we can keep collections abstraction over
_distributed_ data-parallel execution.

- **Shared memory case:** Data-parallel programming model. Data partitioned in
memory and operated upon in parallel.
- **Distributed case:** Data-parallel programming model. Data partitioned
between machines, network in between, operated upon in parallel.

Spark implements a distributed data parallel model called **Resilient
Distributed Datasets (RDDs)**

## Latency

- Reading/writing to disk: **100x slower** than in-memory
- Network communication: **1,000,000x slower** than in-memory

Why Spark?
- Retains fault-tolerance
- Different strategy for handling latency

**Idea:** Keep all data **immutable and in-memory**.

## RDDs

### Word Count

```scala
// Create an RDD
val rdd = spark.textFile("hdfs://...")

val count = rdd.flatMap(line => line.split(" ")) // separate line into words
               .map(word => (word, 1))           // include something to count
               .reduceByKey(_ + _)               // sum up
```

RDDs can be created in two ways:
- Transforming an existing RDD.
- From a `SparkContext` (or `SparkSession`) object.

### Transformations and Actions

- **Transformations.** Return new RDDs as result -  ***Lazy***
- **Actions.** Compute a result based on an RDD, and either returned or saved
  to an external storage system (e.g. HDFS) - ***Eager***

**Laziness/eagerness** is how Spark limits network communications using the
programming model.

## Evaluation

By default, RDDs are recomputed each time you run an action on them. But Spark
allows you to contorl what is cached in memory -  use `persist()` or `cache()`.

- **`cache`**
Shorthand for using the default storage level, which is in memory only as
regular Java objects.

- **`persist`**
Persistence can be customized with this method. Pass the storage level you
like as a parameter.

**Key takeaway:**
Despite similar-looking API to Scala Collections, the deferred semantics of
Spark's RDDs are very unlike Scala Collections.

## Topology

- **Driver Program**
Runs the main program. Holds the Spark Context. Creates RDDs.

- **Worker Node**
Executes the actual computations.

They communicate with each other via a Cluster Manager (e.g. YARN/Mesos).
