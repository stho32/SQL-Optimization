# SQL-Optimization
Notes on optimizing SQL

## General Procedure

### Before you start

1. Create/choose a representative test case that you can execute multiple times
  1. Also check if the sql is REALLY the bottleneck. The execution of the sql should take up the most time.
2. Choose how often you execute your test case
  - As a rule of thumb a number between 3 and 10 should be fine. After this point we will call this, by which we mean executing your test case n times, "taking a sample".
  - When you take your sample without changing your code the distribution of execution durations you get should be about the same.
3. Choose the kind of calculation you want to do to the durations in the sample to calculate your guiding value. Examples:
  - Average
  - Average with standard distribution (+/-)
  - Eliminate the most extreme n outliers and then calculate the average of the remaining durations

### Executing the optimization

- Ensure you keep the code for the currently running version of the code. You will need it to ensure, that the results of your new code are the same. You only want to change the performance.

1. Write down a quick note of what you changed
2. Take the sample and write down the results
3. Think about what you learned and change the code a little bit again
4. Goto 1 until satisfied with the result


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
