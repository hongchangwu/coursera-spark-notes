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
