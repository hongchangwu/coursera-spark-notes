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

Word Count

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
