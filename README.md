# SQL-Optimization
Notes on optimizing SQL

## General Procedure

### 1. Profile Your Application

Before you even look at your SQL query: Use a profiler tailored to your development environment (e.g., dotTrace for .NET, Java profilers for Java applications) to:

- Analyze the overall performance of your application.
- Verify if SQL queries are a significant bottleneck:
  - If yes, proceed with SQL query optimization.
  - If not, address other performance issues first.

### 2. Optimize the SQL

1. **Choose a Representative Test Case that can be executed multiple times**
  - Select a query or set of queries that accurately reflects typical usage.
  - When getting more sophisticated and detailed you might create additional sub-test cases later, but you have to keep the most overarching one because this one marks your starting point and is later needed to verify your overall result. Without it you have nothing but wasted time. You might have optimized but who knows, maybe you have slowed down the system instead.

2. **Sampling and Measurement**
  - Choose your metric (e.g. duration or logical reads using SET STATISTICS IO ON, the simple duration of a query can and can not be a valid orientation because auf SQL Server Caching methods, you might prefer logical reads instead).
  - Select a calculation method for your guiding value (average, average with standard deviation, or trimmed mean).
  - Taking a sample means you run the test case multiple times (3-10)
    - to capture variability in execution times.
    - avoid changing the repetition count and calculation method after this point, because your results will not be comparable then.
  - You know your sampling method is good when you take your sample without changing your code and the distribution of execution durations you get are about the same.

3. **Document your optimization process**
  - Maintain a detailed log of each change as well as the samples you have taken.
  - Do not forget to especially have a starting sample (baseline) because whatever you do, in the end you might want to have a result statement like "we improved the performance from ... to ...".
  - It is ok to document directly in the query analyzer. E.g.:

```sql
-- - Baseline: 24s 26s 25s = 25s average
-- - Removed unnecessary columns: 22s 23s 21s = 22s average
-- - Added index on column X: 10s 12s 9s = 10.3s average
-- ...
```

4. **Execute the optimization: Iterative approach**
  1. Isolate the slowest part of the query.
  2. Change it.
  3. Document what you have changed.
  4. Take the sample for that change.
  5. Decide based on the results wether to keep that change or to roll it back.
  6. Repeat from 1. until satisfied with the performance improvement.

There might be a necessity to dive deeper into separate elements. E.g. it can be that you start out with a view but find out, that a specific scalar function that is used is the bottleneck. In that case you can apply the optimization process recursivly (start again at 1. "Choose a representative test" for the subquery, scalar function, ...).


- 
