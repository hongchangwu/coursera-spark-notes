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

- **Grouping**: `groupByKey` moves a lot of data from one node to another to
  ge "grouped" with its key. Doing so is called "shuffling" and is expensive.
- **Reducing**: `reduceByKey` reduces dataset locally first, therefore the
  amount of data sent over the network during the shuffle is greatly reduced.
