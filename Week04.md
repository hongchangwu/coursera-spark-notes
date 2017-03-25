# Week 04

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
  

