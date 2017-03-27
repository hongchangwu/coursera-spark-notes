#  Week 04

## Structured vs Unstructured Data

When we perform operations on Spark datasets, there are often many different possible 
approaches. For example, if we want to count the number of records that
satifies conditions from two pair RDDs.

1. Join the two RDDs. first, and then filter on the conditions (**Slower**)
2. Filter each of the RDD first, and then join them together (**Fastest**)
3. Get the cartesian product of the two RDDs, filter on keys, and the filter on
   the conditions (**Slowest**)

In most cases we have to do the optimizaiton by hand.

All data isn't equal, structurally. It falls on a spectrum from unstructured
to structured.

- **Unstructured:**
  - Log files
  - Images
- **Semi-structured:**
  - JSON
  - XML
- **Structured:**
  - Database tables

### Structured Data vs RDDs

Spark + regular RDDs don't know anything about the **schema** of the data.

- **Spark RDDs:** 
  - Do **functional transformations** on data
  - Not much structure. Difficult to aggressively optimize.
- **Database/Hive:** 
  - Do **declarative transformations** on data
  - Lots of structure. Lots of optimization opportunities.
  
## Spark SQL

### Overview

**Three main goals:**
1. Support **relational processing** both with Spark programs (on RDDs) and on
   external data sources with a friendly API.
2. High performance, achieved by using techniques from research in databases.
3. Easily support new data sources such as semi-structured data on external
   databases.
   
**Three main APIs:**
- SQL literal syntax
- `DataFrame`s
- `Dataset`s

**Two specialized backend components:**
- **Catalyst**, query optimizer
- **Tungsten**, off-heap serializer

### Relational Queries (SQL)

Terminologies:
- A _relation_ is just a table.
- _Attributes_ are columns.
- Rows are _records_ or _tuples_.

**DataFrame** is Spark SQL's core abstraction. Conceptually, they are RDDs
full of records with a **known schema**.

DataFrames are **untyped**!

Transformations on DataFrames are also known as **untyped transformations**.

### SparkSession

To get started using Spark SQL, everything starts with the SparkSession

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession
  .builder()
  .appName("My App")
  //.config("spark.some.config.option", "some-value")
  .getOrCreate()
```

### Creating DataFrames

`DataFrame`s can be created in two ways:

1. From an existing RDD
   1. From tuples

   ```scala
   val tupleRDD = ... // Assume RDD[(Int, String, String, String)]
   val tupleDF = tupleRDD.toDF("id", "name", "city", "country") // column names
   ```

   2. From case classes
   ```scala
   case class Person(id: Int, name: String, city: String)
   val peopleRDD = ... // Assume RDD[Person]
   val peopleDF = peopleRDD.toDF
   ```

   3. With explicit schema

2. Reading in a specific **data source** from file

```scala
// 'spark' is the SparkSession object we created a few slides back
val df = spark.read.json("examples/src/main/resources/people.json")
```

Some supported data sources:
- JSON
- CSV
- Parquet
- JDBC

### SQL Literals

```scala
// Register the DataFrame as a SQL temporary view
peopleDF.createOrReplaceTempView("people")
// This essentially gives a name to our DataFrame in SQL
// so we can refer to it in an SQL FROM statement

// SQL literals can be passed to Spark SQL's sql method
val adultsDF
  = spark.sql("SELECT * FROM people WHERE age > 17")
```

- Supported Spark SQL syntax:
  https://docs.datastax.com/en/datastax_enterprise/4.6/datastax_enterprise/spark/sparkSqlSupportedSyntax.html
- For a HiveQL cheat sheet:
  https://hortonworks.com/blog/hive-cheat-sheet-for-sql-users/
- For an updated list of supported Hive features in Spark SQL, the official
  Spark SQL docs enumerate:
  https://spark.apache.org/docs/latest/sql-programming-guide.html#supported-hive-features

## DataFrames

### Overview

DataFrames are 
- A relational API over Spark's RDDs
- Able to be automatically aggresively optimzied
- Untyped!

**DataFrame API:** Similar-looking to SQL. Example methods include:
- `select`
- `where`
- `limit`
- `orderBy`
- `groupBy`
- `join`

Helper methods:
- **`show()`** pretty-prints DataFrame in tabular form. Shows first 20 elements.
- **`printSchema()`** prints the schema of your DataFrame in a tree format.

### Specifying Columns

1. Using $-notation:

   ```scala
   // requires import spark.implicits._
   df.filter($"age" > 18)
   ```

2. Referring to the DataFrame

   ```scala
   df.filter(df("age") > 18)
   ``` 

3. Using SQL query string

   ```scala
   df.filter("age > 18")
   ```

**Example:** Rewrite the example SQL query from previous session using
DataFrame API.

```scala
case class Employee(id: Int, fname: String, lname: String, age: Int, city:
String)
val employeeDF = sc.parallelize(...).toDF

val sydneyEmployeesDF = employeeDF.select("id", "lname")
                                  .where("city == 'Sydney'")
                                  .orderBy("id")
```

### Grouping and Aggregating

For grouping & aggregating, Spark SQL provides:
- a `groupBy` function which returns a `RelationalGroupedDataset`
- which has several standard aggregation funcitons defined on it like `count`,
  `sum`, `max`, `min`, and `avg`

**Examples:**

```scala
df.groupBy($"attribute1")
  .agg(sum($"attribute2"))

df.groupBy($"attribute1")
  .count($"attribute2")
```

**References:**
- `RelationalGroupedDataSet`:
  http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.RelationalGroupedDataset
- Methods within `agg`:
  http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$

### Clearing Data

**Dropping records with unwanted values:**
- **`drop()`** drops rows that contain `null` or `NaN` values in **any** column
  and returns a new `DataFrame`.
- **`drop("all")`** drops rows that contain `null` or `NaN` values in **all**
  columns and retruns a new `DataFrame`.
- **`drop(Array("id", "name"))`** drops rows that contain `null` or `NaN`
  values in the **specified** columns and returns a new `DataFrame`.

**Replacing unwanted values:**
- **`fill(0)`** replaces all occurrences of `null` or `NaN` in **numeric
  columns** with **specified value** and returns a new `DataFrame`.
- **`fill(Map("minBalance" -> 0))`** replaces all occurrences of `null` or
  `NaN` in **specified column** with **specified value** and returns a new
  `DataFrame`.
- **`replace(Array("id"), Map(1234 -> 8923))`** replaces **specified value**
  in **sepcified column** with **specified replacement value** and returns a
  new `DataFrame`.

### Common Actions on DataFrames

- `collect(): Array[Row]`
- `count(): Long`
- `first(): Row / head(): Row`
- `show(): Unit`
- `take(n: Int): Array[Row]`

### Joins on DataFrames

Types of joins available: `inner`, `outer`, `left_outer`, `right_outer`, `leftsemi`

**Examples:**

```scala
df1.join(df2, $"df1.id" === $"df2.id")

df1.join(df2, $"df1.id" === $"df2.id", "right_outer")
```

### Optimizations

**Catalyst:** Spark SQL's query optimizer

Catalyst compiles Spark SQL programs down to an RDD. Having full knowledge about the 
data and the computations makes it possible to do optmizations like:

- **Reordering operations**
- **Reduce the amount of data we must read**
- **Pruning unneeded partitioning**

**Tungsten:** Spark SQL's off-heap data encoder

Since our data types are restricted to Spark SQL data types, Tungsten can
provide:

- highly-specialized data encoders
- **column-based**
- off-heap (free from garbage colleciton overhead!)

### Limitations of DataFrames

- Untyped!
- Limited Data Types
- Requires Semi-Structured/Structured Data

## Datasets

`DataFrame`s are actually `Dataset`s!

```scala
type DataFrame = Dataset[Row]
```

- `Dataset`s can be thought of as **typed** distributed collections of data.
- `Dataset` API unifies the `DataFrame` and `RDD` APIs.
- `Dataset`s requires structured/semi-structured data.

**Example:**

```scala
listingsDS.groupByKey(l => l.zip)        // looks like groupByKey on RDD
          .agg(avg($"price").as[Double]) // looks like DataFrame operators
```

### Creating Datasets

**From a DataFrame**

```scala
myDF.toDS // requires import spark.implicits._
```

**From a file**

```scala
val myDS = spark.read.json("people.json").as[Person]
```

**From an RDD**

```scala
myRDD.toDS // requires import spark.implicits._
```

**From common Scala types**

```scala
List("yay", "ohnoes", "hooray!").toDS // requires import spark.implicits._
```

### Typed Columns

On `Dataset`s, typed operations act on `TypedColumn`. To create a
`TypedColumn`:

```scala
$"price".as[Double] // now a TypedColumn
```

### Transformations on Datasets

The `Dataset` API includes both **untyped** and **typed** transformations.

**Common typed transformations:**

- `map[U](f: T => U): Dataset[U]`
- `flatMap[U](f: T => TraversableOnce[U]): Dataset[U]`
- `filter(pred: T => Boolean): Dataset[T]`
- `distinct(): Dataset[T]`
- `groupByKey[K](f: T => K): KeyValueGroupedDataset[K, T]`
- `coalesce(numPartitions: Int): Dataset[T]`
- `repartition(numPartitions: Int): Dataset[T]`

Some `KeyValueGroupedDataset` Aggregation Operations:

- `reduceGroups(f: (V, V) => V): Dataset[(K, V)]`
- `agg[U](col: TypedColumn[V, U]): Dataset[(K, U])`
- `mapGroups[U](f: (K, Iterator[V]) => U): Dataset[U]`
- `flatMapGroups[U](f: (K, Iterator[V]) => TraversableOnce[U]): Dataset[U]`

**Example:**  emulate `reduceByKey` using typed transformations:

```scala
val keyValues = 
  List((3, "Me"), (1, "Thi"), (2, "Se"), (3, "ssa"), (1, "sIsA"), (3, "ge:"), (3, "-)"), (2, "cre"), (2, "t"))

val keyValuesDS = keyValues.toDS

keyValuesDS.groupByKey(p => p._1)
           .mapGroups((k, vs) => (k, vs.foldLeft("")((acc, p) => acc + p._2)))
           .sort($"_1").show()
```

### Aggregators

```scala
class Aggregator[-IN, BUF, OUT]
```

- **`IN`** is the input to the aggregator.
- **`BUF`** is the intermediate type during aggregation.
- **`OUT`** is the type of the output of the aggregation.

To define a custom `Aggregator`:

```scala
val myAgg = new Aggregator[IN, BUF, OUT] {
  def zero: BUF = ...                     // The initial value
  def reduce(b: BUF, a: IN): BUF = ...    // Add an element to the running total
  def merge(b1: BUF, b2: BUF): BUF = ...  // Merge intermediate values
  def finish(b: BUF): OUT = ...           // Return the final result
}.toColumn
```

**Example:** emulate `reduceByKey` with an `Aggregator`

```scala
val keyValues = 
  List((3, "Me"), (1, "Thi"), (2, "Se"), (3, "ssa"), (1, "sIsA"), (3, "ge:"), (3, "-)"), (2, "cre"), (2, "t"))

val keyValuesDS = keyValues.toDS

val strConcat = new Aggregator[(Int, String), String, String]{
  def zero: String = ""
  def reduce(b: String, a: (Int, String)): String = b + a._2
  def merge(b1: String, b2: String): String = b1 + b2
  def finish(r: String): String = r
  override def bufferEncoder: Encoder[String] = Encoders.STRING
  override def outputEncoder: Encoder[String] = Encoders.STRING
}.toColumn

keyValuesDS.groupByKey(pair => pair._1)
           .agg(strConcat.as[String])
```

### Encoders

`Encoder`s are what convert your data between JVM objects and Spark SQL's
specialized internal (tabular) representation. **They are required by all
`Dataset`s!**

Two ways to introduce encoders:
- **Automatically** (generally the case) via implicits from a `SparkSession`,
  `import spark.implicits._`
- **Explicitly** via `org.apache.spark.sql.Encoder`, which contains a large
  selection of methods for creating `Encoder`s from Scala primitive types and
  `Product`s.

### Common Dataset Actions

- `collect(): Array[T]`
- `count(): Long`
- `first(): T / head(): T`
- `foreach(f: T => Unit): Unit`
- `reduce(f: (T, T) => T): T`
- `show(): Unit`
- `take(n: Int): Array[T]`

### Choose between Datasets vs DataFrames vs RDDs

**Use Datasets when...**
- you have structured/semi-structured data
- you want type safety
- you need to work with functional APIs
- you need good performance, but it doesn't have to be the best

**Use DataFrames when...**
- you have structured/semi-structured data
- you want the best possible performance, automatically optimized for you

**Use RDDs when...**
- you have unstructured data
- you need to fine-tune and manage low-level details of RDD computations
- you have complex data types that cannot be serialized with `Encoder`s

### Limitations of Datasets

- **Catalyst Can't Optimize All Operations**, e.g. it can optimize relational
filter operations, but cannot optimize functional filter operations.
- **Limited Data Types**
- **Requires Semi-Structured/Structured Data**
