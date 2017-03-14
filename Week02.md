# Week 02

## Reduction Operations

Walk through a collection and combine neighboring elements of the collection
together to produce a single combine result.

- Some are not parallelizable, e.g. `foldLeft`, `foldRight`
- Some are parallelizable, e.g. `fold`, `reduce`, `aggregate`

## Pair RDDs

In data science it's more common to operate on data in the form of key-value
pairs. In fact manipulating key-value pairs is a key choice in the design of
MapReduce.

### Creating a Pair RDD

```scala
val rdd: RDD[WikipediaPage] = ...

// Has type: RDD[(String, String)]
val pairRdd = rdd.map(page => (page.title, page.text))
```

## Transformations and Actions on Pair RDDs

**Transformations**
- `groupByKey`
- `reduceByKey`
- `mapValues`
- `keys`
- `joins`
- `leftOuterJoin/rightOuterJoin`

**Actions**
- `countByKey`

## Joins

- Inner joins (`join`)
- Outer joins (`leftOuterJoin/rightOuterJoin`)
