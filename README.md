# SQL Server Grammar

# 4. xml 
```
--drop table test_j
create table test_j( a int, b int )
insert into test_j select 1,2
insert into test_j select 1,1
insert into test_j select 1,3
insert into test_j select 2,2
insert into test_j select 2,null
insert into test_j select 3,null

select B.a, 
       (SELECT A.b FROM test_j AS A WHERE A.a = B.a FOR XML AUTO, ELEMENTS),
       replace(replace( replace( replace( (SELECT Z.b FROM test_j AS Z 
                                            WHERE Z.a = B.a 
                                            ORDER BY Z.b
                                              FOR XML AUTO, ELEMENTS), 
        '</b></Z><Z><b>', ',' ), 
        '<Z><b>', '' ), 
        '</b></Z>', '' ),
        '<Z/>', '')
from test_j as B
group by B.a 

-- <A><b>2</b></A><A><b>1</b></A><A><b>3</b></A><A><b>2</b></A>

SELECT * 
  FROM test_j    
   FOR XML RAW ('DataBlock1'), ROOT('ROOT')

--<ROOT><DataBlock1 a="1" b="2"/><DataBlock1 a="1" b="1"/><DataBlock1 a="1" b="3"/><DataBlock1 a="2" b="2"/><DataBlock1 a="2"/><DataBlock1 a="3"/></ROOT>

SELECT * 
  FROM test_j    
   FOR XML RAW ('DataBlock1'), ROOT('ROOT'), ELEMENTS  

--<ROOT><DataBlock1><a>1</a><b>2</b></DataBlock1><DataBlock1><a>1</a><b>1</b></DataBlock1><DataBlock1><a>1</a><b>3</b></DataBlock1><DataBlock1><a>2</a><b>2</b></DataBlock1><DataBlock1><a>2</a></DataBlock1><DataBlock1><a>3</a></DataBlock1></ROOT>

SELECT * 
  FROM test_j    
   FOR XML PATH('DataBlock1') 

--<DataBlock1><a>1</a><b>2</b></DataBlock1><DataBlock1><a>1</a><b>1</b></DataBlock1><DataBlock1><a>1</a><b>3</b></DataBlock1><DataBlock1><a>2</a><b>2</b></DataBlock1><DataBlock1><a>2</a></DataBlock1><DataBlock1><a>3</a></DataBlock1>

```

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
