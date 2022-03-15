# SQL-Optimization
Notes on optimizing SQL

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
