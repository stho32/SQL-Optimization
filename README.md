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
