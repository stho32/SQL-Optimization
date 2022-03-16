# SQL-Optimization
Notes on optimizing SQL

## General Procedure

### Before you start

1. Create/choose a representative test case that you can execute multiple times
  1. Also check if the sql is REALLY the bottleneck. The execution of the sql should take up the most time.
2. Choose how often you execute your test case
  - As a rule of thumb a number between 3 and 10 should be fine. After this point we will call this, by which we mean executing your test case n times, "taking a sample".
  - When you take your sample without changing your code the distribution of execution durations you get should be about the same.
3. Choose the kind of calculation you want to do to the durations in the sample to calculate your guiding value.

## DROP Caches on SQL Server

You may not want that the caches of the SQL Server deminish the value of your measured durations:

```sql
CHECKPOINT; 
GO 
DBCC DROPCLEANBUFFERS; 
GO
```

## Things to look out for

- scalar functions --> is a view better?, can you cache values in variables?
- GETDATE()? can it be cached in a variable?
- having a look at the real execution plan: is an index recommended? does it work?
- join hints: does a join hint like LEFT HASH JOIN LEFT MERGE JOIN or something like that change the performance?
