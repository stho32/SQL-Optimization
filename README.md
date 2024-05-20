# SQL-Optimization
This guide provides notes and techniques for optimizing SQL queries and views to improve performance.

## General Procedure

### 1. Profile Your Application

Before you even look at your SQL query, use a profiler tailored to your development environment (e.g., dotTrace for .NET, Java profilers for Java applications) to:

- Analyze the overall performance of your application's use case.
- Verify if SQL queries are a significant bottleneck:
  - If yes, proceed with SQL query optimization.
  - If not, address other performance issues first.

### 2. Optimize the SQL

1. **Choose a Representative Test Case**

    - Select a query or set of queries that accurately reflects typical usage.
    - When you are about to test a stored procedure, do a code review first. Be sure that you know which branches of the code you want to optimize and that you cover them all.

2. **Sampling and Measurement**

    - Choose your metric, e.g., duration or logical reads
        - a lot of times, just measuring the duration of the execution in the query analyzer might be sufficient
        - sometimes, the caching behavior of the SQL server will obscure your results. You can do two things then:
            - Using `SET STATISTICS IO ON` and the `logical reads` as a value for orientation will be very constant, even if the duration varies. A smaller value is better.
            - Include a drop of the caches in your test case `CHECKPOINT; GO; DBCC DROPCLEANBUFFERS; GO;` Be aware that dropping the cache can also negatively impact your measurement as it may take a long and variable amount of time.
    - Select a calculation method for your guiding value (average, average with standard deviation, or trimmed mean).
    - Taking a sample means running the test case multiple times (3-10) to capture variability in execution times.
    - Avoid changing the repetition count and calculation method after this point because your results will not be comparable.
    - You know your sampling method is suitable when you take your sample without changing your code, and the distribution of execution durations you get are about the same.
    - Keep the most overarching test case around because this one marks your starting point and is later needed to verify your overall result. Without it, you have nothing but wasted time. You may have done something but may have slowed down the system unintentionally, and you cannot prove your progress to your colleagues.

3. **Document your optimization process**

    - Maintain a log of each change and the samples you have taken.
    - Do not forget to primarily have a starting sample (baseline) because whatever you do, in the end, you might want to have a result statement like "we improved the performance from ... to ...". And you might want to withstand a test of your superior that asks you about what you have changed. Also, performance tests can be time-consuming. Your superior will be much more satisfied with you if you can report the 20 small tests you performed that might not have improved the result but still show that you were systematically working through all possible ideas.
    - It is ok to document directly in the query analyzer. E.g.:

```SQL
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
    5. Decide based on the results whether to keep that change or to roll it back.
    6. Repeat from 1. until satisfied with the performance improvement.

There might be a necessity to dive deeper into separate elements. For example, you start with a view but find out that a specific scalar function used is the bottleneck. In that case, you can apply the optimization process recursively (begin again at 1. "Choose a representative test" for the subquery, scalar function, ...).


## Usual suspects that create bad performance

#### 1. Scalar Functions

- **Why They Are Slow:**
  - Applying scalar functions to many rows results in row-by-row processing, which could be more efficient.

- **Ideas to Improve Performance:**
  - Convert scalar functions to views or table-valued functions where possible.
  - Cache values in variables to avoid repeated calculations.

#### 2. CASE Statements

- **Why They Are Slow:**
  - CASE statements applied to many rows can lead to complex and slow query execution.

- **Ideas to Improve Performance:**
  - Convert CASE statements to subselects using UNIONs to simplify and speed up execution.

#### 3. Join Types

- **Why They Are Slow:**
  - Inefficient join types can lead to sizeable intermediate result sets and slow performance.

- **Ideas to Improve Performance:**
  - Optimize join operations by using join hints like LEFT HASH JOIN or LEFT MERGE JOIN.
  - Ensure indexes on join columns are adequately defined.

#### 4. Indexes

- **Why They Are Slow:**
  - Missing or improperly defined indexes cause full table scans instead of quick index lookups.

- **Ideas to Improve Performance:**
  - Review the execution plan to identify missing indexes.
  - Implement recommended indexes and regularly maintain them.

#### 5. Dynamic SQL

- **Why They Are Slow:**
  - Dynamic SQL can bypass query optimization and lead to inefficient execution plans.

- **Ideas to Improve Performance:**
  - Use parameterized queries instead of dynamic SQL where possible.
  - Ensure dynamic SQL is necessary and adequately indexed.

#### 6. Subselects

- **Why They Are Slow:**
  - Subselects, especially in UNION statements, can cause the query to process extensive data sets multiple times.

- **Ideas to Improve Performance:**
  - Test each subselect individually and optimize the slowest ones first.
  - Combine subselects judiciously to minimize redundant data processing.

#### 7. CLR Functions

- **Why They Are Slow:**
  - CLR functions share a thread and can be slower than native SQL functions due to the overhead of managed code execution.

- **Ideas to Improve Performance:**
  - Avoid using CLR functions unless necessary.
  - Optimize CLR functions or replace them with equivalent T-SQL functions.

#### 8. Large Binary Fields

- **Why They Are Slow:**
  - Retrieving large binary fields can consume significant I/O bandwidth and memory.

- **Ideas to Improve Performance:**
  - Optimize queries to minimize the retrieval of large binary fields.
  - Retrieve only necessary data and avoid fetching large fields unless needed.

#### 9. Order of Joins

- **Why They Are Slow:**
  - Inefficient join order can lead to sizeable intermediate result sets and excessive processing.

- **Ideas to Improve Performance:**
  - Join tables with the most effective filtering conditions first to reduce the result row count early.
  - Review and adjust join order based on query execution plans.

#### 10. Inefficient WHERE Clauses

- **Why They Are Slow:**
  - Complex and poorly optimized WHERE clauses can lead to full table scans and slow performance.

- **Ideas to Improve Performance:**
  - Simplify WHERE clauses and ensure they use indexed columns effectively.
  - Use query hints and review execution plans to optimize filtering conditions.

#### 11. GETDATE() Function

- **Why They Are Slow:**
  - Repeatedly calling GETDATE() in a query can cause redundant calculations and slow execution.

- **Ideas to Improve Performance:**
  - Cache the result of GETDATE() in a variable if used multiple times within a query or procedure.

#### 12. Caches

- **Why They Are Slow:**
  - SQL Server caching mechanisms can mask the actual performance of queries, making it difficult to measure improvements.

- **Ideas to Improve Performance:**
  - Use `CHECKPOINT` and `DBCC DROPCLEANBUFFERS` commands to clear caches before taking performance measurements for accurate results.

