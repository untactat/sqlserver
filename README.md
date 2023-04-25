# SQL Server Grammar

# 1. 병렬처리 (parallelism)
- 병렬처리수준(MAXDOP : MAX DEGREE OF PARALLELISM)
```
SELECT * FROM test9000 OPTION (MAXDOP 4) -- CPU Core 4개 사용 fix 
```
