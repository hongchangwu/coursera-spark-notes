# Week 04

## Structured & Unstructured Data

When we perform operations on Spark datasets, there are often many different possible 
approaches. For example, if we want to count the number of records that
satifies conditions from two pair RDDs.

1. Join the two RDDs. first, and then filter on the conditions (**Slower**)
2. Filter each of the RDD first, and then join them together (**Fastest**)
3. Get the cartesian product of the two RDDs, filter on keys, and the filter on
   the conditions (**Slowest**)
