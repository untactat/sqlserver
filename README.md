# SQL Server Grammar

# 3. 특수문자 찾기 
```
--drop table test_j
create table test_j
(
    s nvarchar(4000)
)

insert into test_j select '123'
insert into test_j select '123a'
insert into test_j select '하하하'
insert into test_j select '하하22하'
insert into test_j select '하하!@#$%^&*(){}|\`<>/하'
insert into test_j select '하하        !"#$%&''()*+,-./         하'
insert into test_j select '하하        :;<=>?@         하'
insert into test_j select '하하        [\]^_`         하'
insert into test_j select '하하        {|}~         하'

select * From test_j where s like '%[0-9]%'
select * From test_j where s like '%[!-/]%'
select * From test_j where s like '%[:-@]%'
select * From test_j where s like '%[[-`]%'
select * From test_j where s like '%[{-~]%'
select * From test_j where s like '%[ㄱ-힇]%'
--select * From test_j where s like '%[ㄱ-힇%]'
```

# 2. 계층형쿼리 
```
;WITH TEMP AS 
(
    SELECT CONVERT(NVARCHAR(8),'20220101') AS StdDate, 
           1 AS LevelSeq 

           --,CONVERT(NVARCHAR(8),DATEADD(day,1,'20210101'),112) 

    UNION ALL 

    SELECT CONVERT(NVARCHAR(8),DATEADD(day,1,A.StdDate),112) AS StdDate, 
           A.LevelSeq+1 AS LevelSeq 

      FROM TEMP AS A 
     WHERE A.StdDate <= CONVERT(NVARCHAR(8),GETDATE(),112)
)
SELECT TOP 50 
       A.*
        
  FROM TEMP AS A 
 ORDER BY A.StdDate DESC 
OPTION (maxrecursion 1000) -- The statement terminated. The maximum recursion 100 has been exhausted before statement completion.
```

# 1. 병렬처리 (parallelism)
- 병렬처리수준(MAXDOP : MAX DEGREE OF PARALLELISM)
- oracle과 달리 sql server 기본이 병렬처리로 되어있다. 실행계획이 알아서 필요만큼 CPU Core 사용한다   
```
SELECT * FROM test9000 OPTION (MAXDOP 4) -- CPU Core 4개 사용 fix 
```
