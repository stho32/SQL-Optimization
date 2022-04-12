# SQL-Optimization
Notes on optimizing SQL

## General Procedure

### Before you start

1. Create/choose a representative test case that you can execute multiple times. When getting more sophisticated and detailed you might create additional sub-test cases later, but you have to keep the most overarching one because this one marks your starting point and is later needed to verify your overall result. Without it you have nothing but wasted time. You might have optimized but who knows, maybe you have slowed down the system instead.
  1. Also check if the sql is REALLY the bottleneck. The execution of the sql should take up the most time. E.g. in .Net applications you have the option to use a profiler.
2. Choose how often you execute your test case
  - As a rule of thumb a number between 3 and 10 should be fine. After this point we will call this, by which we mean executing your test case n times, "taking a sample".
  - When you take your sample without changing your code the distribution of execution durations you get should be about the same.
3. Choose the kind of calculation you want to do to the durations in the sample to calculate your guiding value. Examples:
  - Average
  - Average with standard distribution (+/-)
  - Eliminate the most extreme n outliers and then calculate the average of the remaining durations
4. Choose what to measure. Although we speak of duration in this text it might actually be not the right value to measure because optimizations might kick in that render the test useless. USE `SET STATISTICS TIME, IO ON` instead. While the end user cares for performance - so you should give the information about performance improvement in duration - the thing that you actually care for are **logical reads**. While most other information will change between calls a lot the logical reads will remain about constant.
5. Check the logic of the thing you measure. For example if you need to test a stored procedure, do all calls to that thing really execute all the same if-branches in the procedure. (My current optimization target as I write this is a stored procedure that assigns a new task to an employee unless he/she already has one. That means that there are two different branches and the second execution will be faster because a task is probably assigned the first time it runs).

6. Ensure that you document each step of the way. Especially your starting point, as it will help you to tell the other people around you what you did all day and what the benefit was.

7. Ensure you keep the code for the currently running version of the code. You will need it to ensure, that the results of your new code are the same. You only want to change the performance. A good rule of thumb is to create the new version with a versioning number at the end. E.g. if "HelloWorldView" is your original view then "HelloWorldView2" is the next new implementation. Also this makes sure that you do not break any running code. Even if you change the output, old code will be able to use your old implementation until explicitly told to do otherwise.

8. You should have a version control ready for your database. You can create one by extracting all code like every day into a git repository for example and then automatically commit and push every day. Thus, when you stumble, you can always roll back to an earlier version.

### Executing the optimization in general

1. Write down a quick note of what you changed
2. Take the sample and write down the results
3. Think about what you learned and change the code a little bit again
4. Goto 1 until satisfied with the result

The top of your script should read like about this:
```sql
-- Baseline: 24s 26s 25s = 25s
-- Commented out everything besides FROM, its columns and where statements
-- new measurement: 1s 1s 0s = 1s
-- Commented in join a (first join) and all its columns and where-statements
-- new measurement: 19s 18s 19s = 19s
-- optimized ...
-- new measurement: 5s 6s 4s = 5s
-- ...
```

That is your optimization protocol and the thing that you throw at people that ask you what you did that day.

### Algorithm for optimizing Views

```sql
CREATE VIEW abc
    AS
    SELECT a.one,
           a.two,
           dbo.SomeFunction(a.One, a.Two) AS OneTwoFunction,
           b.three,
           b.four
      FROM a
      JOIN b ON b.SomeAId = a.Id
     WHERE a.five=99
```

1. add the code for dropping the cache
2. comment out ´CREATE VIEW´ AND ´AS´
3. Comment out every join and every column that is selected besides the from and its columns and where-elements. 
4. Is the execution fast now? No -> restart the process using the code of view a. The problem starts there.
5. The execution is fast now! Yes:
6. while there are joins still left comment in one of the joins you have commented out. Every time you do that also comment in every column that should work now. Also comment in every part of the where statement that should work now. The columns and where-statments are important as they define the load transfered and the indexes used.
7. Is the execution slow now? No: Goto 6
8. Is the execution slow now? Yes: 
9. Change things until the query is fast again.
10. Goto 6 

#### Look out for

- Scalar functions can be bad when applied to lots of rows, can you convert them to views?
- CASE-Statements are bad when applied to lots of rows, can you convert them to subselects using unions?
- When the joined table or view only returns a few rows but the join takes forever, try LEFT HASH JOIN. 
- Does the real execution plan recommend an index you might be missing?

## DROP Caches on SQL Server

You may not want that the caches of the SQL Server deminish the value of your measured durations:

```sql
CHECKPOINT; 
GO 
DBCC DROPCLEANBUFFERS; 
GO
```

## Things to look out for

- do not simply guess - test! When you find an sql statement to be slow, start by commenting things out until it becomes fast.

- scalar functions --> is a view better?, can you cache values in variables?
- GETDATE()? can it be cached in a variable?
- having a look at the real execution plan: is an index recommended? does it work?
- join hints: does a join hint like LEFT HASH JOIN LEFT MERGE JOIN or something like that change the performance?
- CLR functions share a thread, so they might be even more of a pain than scalar functions
- performance can be impacted by the amount and size of data you get back. This is especially true for big binary fields. In a simple flat table retrieving 200tsd. lines of content takes almost no time. So when it is slow, there is a reason.
- when having multiple tables, the order of joined tables in a sql statement should be in a way, so that the tables with the most effective filtering / reduction of the result row count should be joined/handled first. This way the follow up pipelines have less things to do. Be aware that the sql server optimizer is already capable to do this by himself to a certain degree, so this might yield no result or it might take some effort.
- 
