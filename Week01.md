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
