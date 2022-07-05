## [SQL 활용]
</br>

### [제1절] 서브쿼리

#### 서브쿼리?

하나의 SQL 문안에 포함되어 있는 또 다른 SQL

</br>

#### 동작되는 방식에 따른 서브쿼리 분류

1. Un_Correlated(비연관) 서브쿼리
    </br>서브쿼리가 메인쿼리 **칼럼을 갖고 있지 않은 형태**, 메인쿼리에 값을 제공하기 위한 목적
    
2. Correlatied(연관) 서브쿼리
    </br>서브쿼리가 메인쿼리 **칼럼을 갖고있는 형태**, 메인쿼리가 서브쿼리에서 조건이 맞는지 확인하기 위한 목적
</br>


#### 반환되는 데이터의 형태에 따른 서브쿼리 분류

1. 단일행 서브쿼리
    </br>단일 행 비교 연산자(=, <, <=, >, >=, <>)와 함께 사용되며, 서브쿼리의 결과 건수는 반드시 **1건 이하**일것 (2건 이상 반환시 Run Time 오류)
    
    ```SQL
    SELECT PLAYER_NAME AS 선수명, POSITION AS 포지션, BACK_NO AS 백넘버
    FROM PLAYER
    WHERE TEAM_ID = (SELECT TEAM_ID
                       FROM PLAYER
                      WHERE PLAYER_NAME = '윤가영')
    ORDER BY PLAYER_NAME;
    ```
</br>

2. 다중행 서브쿼리
    </br>서브쿼리의 **결과가 2건 이상 반환** 된다면, 다중행 비교 연산자(IN, ALL, ANY, SOME)과 함께 사용
     
    ```SQL
    -- IN : 서브쿼리의 결과에 존재하는 임의의 값과 동일한 조건
    -- ALL : 서브 쿼리의 결과에 존재하는 모든값을 만족하는 조건
    -- ANY(=SOME) : 서브쿼리의 결과에 존재하는 어느 하나의 값이라도 만족하는 조건
    -- EXISTS : 서브쿼리의 결과를 만족하는 값이 존재하는지 여부 확인

    SELECT REGION_NAME AS 연고지명, TEAM_NAME AS 팀명, E_TEAM_NAME AS 영문팀명
    FROM TEAM
    WHERE TEAM_ID IN (SELECT TEAM_ID
                        FROM PLAYER
                       WHERE PLAYER_NAME = '윤가영')
    ORDERY BY TEAM_NAME;
    ```
</br>

3. 다중 칼럼 서브쿼리
    </br>**여러 개의 칼럼이 반환**되어 메인쿼리의 조건과 동시에 비교됨. (SQL Server에서는 지원안함)
    
    ```SQL
    SELECT TEAM_ID AS 팀코드, PLAYER_NAME AS 선수명, POSITION AS 포지션
        ,  BACK_NO AS 백넘버, HEIGHT AS 키
    FROM PLAYER
    WHERE (TEAM_ID, HEIGHT) IN (SELECT TEAM_ID, MIN(HEIGHT)
                                 FROM PLAYER
                                GROUP BY TEAM_ID)
    ORDER BY TEAM_ID, PLAYER_NAME;
    ```
</br>    
   
4. 연관 서브쿼리
    </br>서브쿼리 내에  **메인쿼리 칼럼이 사용된** 서브쿼리
    
    ```SQL
    SELECT B.TEAM_NAME AS 팀명, A.PLAYER_NAME AS 선수명, A.POSITION AS 포지션
        , A.BACK_NO AS 백넘버, A.HEIGHT AS 키
    FROM PLAYER A, TEAM B
    WHERE A.HEIGHT < (SELECT AVG(X.HEIGHT)
                        FROM PLAYER X
                        WHERE X.TEAM_ID = A.TEAM_ID -- 메인쿼리 칼럼 사용
                      GROUP BY X.TEAM_ID)
    ORDER BY TEAM_ID, PLAYER_NAME;



    -- EXISTS 서브쿼리는 항상 연관 서브쿼리로 사용됨. 아무리 조건을 만족하는 건이 여러 건이더라고 1건만 찾으면 추가적인 검색 진행 안함
    SELECT A.STADIUM_ID AS ID, A.STADIUM_NAME AS 경기장명
    FROM STADIUM A
    WHERE EXISTS IN (SELECT 1
                      FROM SCHEDULE X
                     WHERE X.STADIUM_ID = A.STADIUM_ID
                       AND X.SCHE_DATE BETWEEN '20120501' AND '20120502');
    ```
</br>    
    
5. 그 밖의 위치에서 사용하는 서브쿼리

    1. SELECT 절에 서브쿼리 (**스칼라 서브쿼리**, Scalar Subquery)
        
        </br>: 스칼라 서브쿼리는 **한 행, 한 칼럼만을 반환**하는 서브쿼리
        </br>`단일행 서브쿼리, 결과가 2건 이상이면 오류 반환`
      
    2. FROM 절에 서브쿼리 사용 (**인라인 뷰**, Inline View)
       
       </br>: 인라인 뷰를 사용하면 서브쿼리의 결과를 테이블처럼 사용 가능
       </br>`SELECT문을 객체로서 저장하여 테이블처럼 사용하는 View와 달리, 인라인 뷰는 쿼리 내에서 즉시 처리`

    3. HAVING 절에서 서브쿼리 사용
</br>


#### 뷰?

테이블은 실제로 데이터를 갖고있지만, 뷰는 실제 데이터를 갖고있지 않고  **뷰 정의(View Definition)**  만을 갖음
</br>

#### 뷰 특징

- 독립성 : 테이블 구조가 변경돼도 뷰를 사용하는 응용 프로그램은 변경하지 않음

- 편리성 : 복잡한 질의를 뷰로 생성함으로써 관련 질의를 단순하게 작성 가능

- 보안성 : 숨기고 싶은 정보가 있다면, 뷰를 생성할 때 해당 칼럼을 제외하고 생성 가능

```SQL
-- VIEW 생성
CREATE VIEW V_PLAYER_TEAM AS
SELECT A.PLAYER_NAME, A.POSITION, A.BACK_NO, B.TEAM_ID, B.TEAM_NAME
FROM PLAYER A, TEAM B
WHERE B.TEAM_ID = A.TEAM_ID;

-- VIEW 활용
SELECT PLAYER_NAME, POSITION, BACK_NO, TEAM_ID, TEAM_NAME
FROM V_PLAYER_TEAM
WHERE PLAYER_NAME = '윤%';
```

---

### [제2절] 집합 연산자

#### 집합 연산자? 

**여러 개의 결과 집합 간의 연산을 통해 결합**하는 방식
</br>

1. UNION
</br>합집한 연산 수행, **중복 제거** 
    
2. UNION ALL
</br>합집합 연산 수행, **중복 허용**
  
3. INTERSECT
</br>교집합 연산 수행, **중복 제거**
  
1. EXCEPT
</br>차집합 연산 수행, **중복 제거**
</br>

---

### [제3절] 그룹함수

#### 그룹함수?

ANSI/ISO SQL 표준은 데이터 분석을 위해 다음의 3가지 함수를 정의

- **AGGREGATE FUNCTION** (= GROUP AGGREGATE FUNCTION)
    </br>COUNT, SUM, AVG, MAX, MIN 
    
- **GROUP FUNCTION**
    </br>ROLLUP, CUBE, GROUPING SETS
    
- **WINDOW FUNCTION**
    </br>ANALYTIC FUNCTION, RANK FUNCTION
</br>

1. ROLLUP 함수
    - ROLLUP에 지정된 Grouping Columns의 List는 Subtotal을 생성하기 위함
    - Grouping Columns의 수가 N이면, **N+1 Level**의 Subtotal 생성
    - 인수 순서가 바뀌면 수행결과도 바뀜
    </br>
    
    ```SQL
    -- 예제1 : GROUP BY 절 + ORDER BY 절
    SELECT B.DNAME, A.JOB, COUNT(*) AS EMP_CNT, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    GROUP BY B.DNAME, A.JOB
    ORDER BY B.DNAME, A.JOB;
    ```
    |    DNAME   |    JOB    | EMP_CNT | SALSUM |
    |:----------:|:---------:|:-------:|:------:|
    | ACCOUNTING |   CLERK   |    1    |   1300 |
    | ACCOUNTING |  MANAGER  |    1    |   2450 |
    | ACCOUNTING | PRESIDENT |    1    |   5000 |
    |  RESEARCH  |  ANALYST  |    2    |   6000 |
    |  RESEARCH  |   CLERK   |    2    |   1900 |
    |  RESEARCH  |  MANAGER  |    1    |   2975 |
    |    SALES   |   CLERK   |    1    |    950 |
    |    SALES   |  MANAGER  |    1    |   2850 |
    |    SALES   |  SALESMAN |    4    |   5600 |
    </br>
    
    ```SQL
    -- 예제2 : ROLLUP 함수 + ORDER BY 절
    SELECT B.DNAME, A.JOB, COUNT(*) AS EMP_CNT, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    GROUP BY ROLLUP (B.DNAME, A.JOB)
    ORDER BY B.DNAME, A.JOB;
    ```
    |    DNAME   |    JOB    | EMP_CNT | SALSUM |
    |:----------:|:---------:|:-------:|:------:|
    | ACCOUNTING |   CLERK   |    1    |   1300 | -- L1
    | ACCOUNTING |  MANAGER  |    1    |   2450 | -- L1
    | ACCOUNTING | PRESIDENT |    1    |   5000 | -- L1
    | ACCOUNTING |           |    3    |   8750 | -- **L2**
    |  RESEARCH  |  ANALYST  |    2    |   6000 | -- L1
    |  RESEARCH  |   CLERK   |    2    |   1900 | -- L1
    |  RESEARCH  |  MANAGER  |    1    |   2975 | -- L1
    |  RESEARCH  |           |    5    |  10875 | -- **L2**
    |    SALES   |   CLERK   |    1    |    950 | -- L1
    |    SALES   |  MANAGER  |    1    |   2850 | -- L1
    |    SALES   |  SALESMAN |    4    |   5600 | -- L1
    |    SALES   |           |    6    |   9400 | -- **L2**
    |            |           |    14   | 29025  | -- **L3**
 </br>
 
2. CUBE 함수
    - 명시한 모든 칼럼에 대해 Subtotal 생성
    - GROUPING COLUMNS의 수가 N, 2^N승 Subtotal 생성
    - ROLLUP에 비해 시스템에 부하를 줌
    - 인수의 순서가 바뀌는 경우, 행간에 정렬 순서는 바뀔 수 있어도 데이터 결과는 같음
    </br>
    
    ```SQL
    -- 예제1 : CUBE 함수 + ORDER BY 절
    SELECT B.DNAME, A.JOB, COUNT(*) AS EMP_CNT, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    GROUP BY CUBE (B.DNAME, A.JOB)
    ORDER BY B.DNAME, A.JOB;
    </br>
    


    -- 예제2 : CUBE 함수의 UNION ALL 버전
    SELECT B.DNAME, A.JOB, COUNT(*) AS EMP_CNT, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    GROUP BY B.DNAME, A.JOB
    UNION ALL
    SELECT B.DNAME, '' AS A.JOB, COUNT(*) AS EMP_CNT, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    GROUP BY B.DNAME
    UNION ALL
    SELECT '' AS B.DNAME, A.JOB, COUNT(*) AS EMP_CNT, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    GROUP BY A.JOB
    UNION ALL
    SELECT '' AS B.DNAME, '' AS A.JOB, COUNT(*) AS EMP_CNT, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    ```
    |    DNAME   |    JOB    | EMP_CNT | SALSUM |
    |:----------:|:---------:|:-------:|:------:|
    | ACCOUNTING |   CLERK   |    1    |   1300 |
    | ACCOUNTING |  MANAGER  |    1    |   2450 |
    | ACCOUNTING | PRESIDENT |    1    |   5000 |
    | ACCOUNTING |           |    3    |   8750 |
    |  RESEARCH  |  ANALYST  |    2    |   6000 |
    |  RESEARCH  |   CLERK   |    2    |   1900 |
    |  RESEARCH  |  MANAGER  |    1    |   2975 |
    |  RESEARCH  |           |    5    |  10875 |
    |    SALES   |   CLERK   |    1    |    950 |
    |    SALES   |  MANAGER  |    1    |   2850 |
    |    SALES   |  SALESMAN |    4    |   5600 |
    |    SALES   |           |    6    |   9400 |
    |            |  ANALYST  |    2    |   6000 |
    |            |   CLERK   |    4    |   4150 |
    |            |  MANAGER  |    3    |   8275 |
    |            | PRESIDENT |    1    |   5000 |
    |            |  SALESMAN |    4    |   5600 |
    |            |           |    14   |  29025 |
</br>

3. GROUPING SETS 함수
    - 표시된 인수들에 대한 개별 집계를 구할수 있음
    - 인수의 순서가 바뀌어도 결과 같음
    </br>
    
    ```SQL
    -- 예제1 : GROUPING SETS UNION ALL 버전
    SELECT B.DNAME, '' AS A.JOB, COUNT(*) AS EMP_CNT, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    GROUP BY B.DNAME
    UNION ALL
    SELECT '' AS B.DNAME, A.JOB, COUNT(*) AS EMP_CNT, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    GROUP BY A.JOB;
    
    

    -- 예제2 : GROUPING SETS + ORDER BY 절
    SELECT B.DNAME, A.JOB, COUNT(*) AS EMP_CNT, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    GROUP BY GROUPING SETS (B.DNAME, A.JOB)
    ORDER BY B.DNAME, A.JOB;
    ```
    |    DNAME   |    JOB    | EMP_CNT | SALSUM |
    |:----------:|:---------:|:-------:|:------:|
    | ACCOUNTING |           |    3    |   8750 |
    |  RESEARCH  |           |    5    |  10875 |
    |    SALES   |           |    6    |   9400 |
    |            |  ANALYST  |    2    |   6000 |
    |            |   CLERK   |    4    |   4150 |
    |            |  MANAGER  |    3    |   8275 |
    |            | PRESIDENT |    1    |   5000 |
    |            |  SALESMAN |    4    |   5600 |
    
---

### [제4절] 윈도우 함수

#### 윈도우 함수?

기존 관계형 데이터베이스는 칼럼과 칼럼 간의 연산·비교·연결, 집합에 대한 집계는 쉬웠지만 
</br>행과 행간의 관계를 정의·비교·연산하는 것을 하나의 SQL문으로 처리하는것은 어려움.
</br>따라서 `INLINE VIEW를 이용해 복잡한 SQL문을 작성하던 것을 행과 행간의 관계를 쉽게 정의하기 위해 만든 함수`
</br>
</br>

#### 윈도우 함수 종류

- 그룹 내 `순위` 관련 함수 (RANK)
    </br>: RANK, DENSE_RANK, ROW_NUMBER
    
- 그룹 내 `집계` 관련 함수 (AGGREGATE)
    </br>: SUM, MAX, MIN, AVG, COUNT
    
- 그룹 내 `행순서` 관련 함수
    </br>: FIRST_VALUE, LAST_VALUE, LAG, LEAD, FIRST_VALUE(=MAX), LAST_VALUE(=MIN)

- 그룹 내 `비율` 관련 함수
    </br>: CUME_DIST, PERCENT_RANK, NTILE, RATIO_TO_REPORT
</br>

##### 그룹 내 순위 함수

1. RANK 함수
    </br>: ORDER BY를 포함한 QUERY문에서 특정 항목에 대한 순위를 구하는 함수 
    </br>`동일한 값에 대해 동일한 순위 부여`

2. DENSE_RANK 함수
    </br>: RANK와 유사하지만, `동일한 순위를 하나의 건수로 취급`
    
3. ROW_NUMBER 함수
    </br>: RANK나 DENSE_RANK는 동일한 값에 대해서는 동일한 순위를 부여하지만,
    </br>`동일한 값이라도 고유한 순위 부여`
    
|  SAL | RANK() | DENSE_RANK() | ROW_NUMBER() |
|:----:|:------:|:------------:|:------------:|
| 5000 |    1   |       1      |       1      |
| 3000 |    2   |       2      |       2      |
| 3000 |    2   |       2      |       3      |
| 2000 |    4   |       3      |       4      |
| 1500 |    5   |       4      |       5      |
| 1500 |    5   |       4      |       6      |
| 1000 |    7   |       5      |       7      |
| 50   |    8   |       6      |       8      |
</br>

##### 그룹 내 집계 함수

1. SUM 함수
    </br>: 파티션별 위도우의 합
    
2. MAX 함수
    </br>: 파티션별 윈도우의 최댓값
    
3. MIN 함수
    </br>: 파티션별 윈도우의 최댓값
    
4. AVG 함수
    </br>: 파티션별 윈도우의 평균값, ROWS 윈도우를 통해 원하는 조건에 맞는 데이터에 대한 통계값 구할 수 있음
    
5. COUNT 함수
    </br>: 파티션별 윈도우의 카운트값 
</br>

##### 그룹 내 행순서 관련 함수

1. FIRST_VALUE 함수
    </br>: 파티션 별 윈도우에서 가장 먼저 나온 값 구함 (SQL Server에서 지원 안함)
    </br>  공동 등수를 인정하지 않고 처음 나온 행만 처리한다.
    
    ```SQL
    SELECT DEPTNO, ENAME, SAL
        , FIRST_VALUE (ENAME) OVER (PARTITION BY DEPTNO ORDER BY SAL DESC
                                    ROWS UNBOUNDED PRECEDING) AS ENAME_FV -- 현재 행을 기준으로 파티션 내 첫 번째 행까지의 범위를 지정
      FROM EMP;
    ``` 
    | DEPTNO |  ENAME |  SAL | ENAME_FV |
    |:------:|:------:|:----:|:--------:|
    |     10 |  KING  | 5000 |   KING   |
    |     10 |  CLARK | 2450 |   KING   |
    |     10 | MILLER | 1300 |   KING   |
    |     20 |  SCOTT | 3000 |   SCOTT  | -- SCOTT
    |     20 |  FORD  | 3000 |   SCOTT  |
    |     20 |  JONES | 2975 |   SCOTT  |
    |     20 |  ADAMS | 1100 |   SCOTT  |
    |     20 |  SMITH |  800 |   SCOTT  |
    |     30 |  BLAKE | 2850 |   BLAKE  |
    |     30 |  ALLEN | 1600 |   BLAKE  |
    |     30 | TURNER | 1500 |   BLAKE  |
    |     30 | MARTIN | 1250 |   BLAKE  |
    |     30 |  WARD  | 1250 |   BLAKE  |
    |     30 |  JAMES |  950 |   BLAKE  |
    </br>

    ```SQL
    -- 의도적으로 세부 항목을 정렬하고 싶다면, 
    -- 별도의 정렬조건을 가진 INLINE VIEW 혹은 OVER() 내의 ORDER BY 절에 칼럼 추가
    SELECT DEPTNO, ENAME, SAL
        , FIRST_VALUE (ENAME) OVER (PARTITION BY DEPTNO ORDER BY SAL DESC, ENAME
                                    ROWS UNBOUNDED PRECEDING) AS ENAME_FV 
      FROM EMP;
    ```
    | DEPTNO |  ENAME |  SAL | ENAME_FV |
    |:------:|:------:|:----:|:--------:|
    |     10 |  KING  | 5000 |   KING   |
    |     10 |  CLARK | 2450 |   KING   |
    |     10 | MILLER | 1300 |   KING   |
    |     20 |  FORD  | 3000 |   FORD   | -- FORD
    |     20 |  SCOTT | 3000 |   FORD   |
    |     20 |  JONES | 2975 |   FORD   |
    |     20 |  ADAMS | 1100 |   FORD   |
    |     20 |  SMITH |  800 |   FORD   |
    |     30 |  BLAKE | 2850 |   BLAKE  |
    |     30 |  ALLEN | 1600 |   BLAKE  |
    |     30 | TURNER | 1500 |   BLAKE  |
    |     30 | MARTIN | 1250 |   BLAKE  |
    |     30 |  WARD  | 1250 |   BLAKE  |
    |     30 |  JAMES |  950 |   BLAKE  |
</br>

2. LAST_VALUE 함수
    </br>: 파티션별 윈도우에서 가장 나중에 온 값 구함 (SQL Server에서 지원 안함)

    ```SQL
    -- FIRST_VALUE 와 마찬가지로 공동 등수를 인정하지 않기 때문에, 의도적으로 INLINE VIEW 사용 혹은 ORDER BY 절에 칼럼을 추가한다.
    SELECT DEPTNO, ENAME, SAL
        , LAST_VALUE (ENAME) OVER (PARTITION BY DEPTNO ORDER BY SAL DESC
                                    ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS ENAME_FV -- 현재 행을 포함하여 파티션 내의 마지막 행까지의 범위를 지정
      FROM EMP;
    ```
    | DEPTNO |  ENAME |  SAL | ENAME_FV |
    |:------:|:------:|:----:|:--------:|
    |     10 |  KING  | 5000 |  MILLER  |
    |     10 |  CLARK | 2450 |  MILLER  |
    |     10 | MILLER | 1300 |  MILLER  |
    |     20 |  FORD  | 3000 |   SMITH  |
    |     20 |  SCOTT | 3000 |   SMITH  |
    |     20 |  JONES | 2975 |   SMITH  |
    |     20 |  ADAMS | 1100 |   SMITH  |
    |     20 |  SMITH |  800 |   SMITH  |
    |     30 |  BLAKE | 2850 |   JAMES  |
    |     30 |  ALLEN | 1600 |   JAMES  |
    |     30 | TURNER | 1500 |   JAMES  |
    |     30 | MARTIN | 1250 |   JAMES  |
    |     30 |  WARD  | 1250 |   JAMES  |
    |     30 |  JAMES |  950 |   JAMES  |
</br>

3. LAG 함수
    </br>: 파티션별 윈도우에서 이전 몇 번째 행의 값을 가져옴 (SQL Server에서 지원 안함)

    ```SQL
    SELECT ENAME, HIREDATE, SAL
         , LAG (SAL) OVER (ORDER BY HIREDATE) AS LAG_SAL -- 본인보다 입사일자가 한 명 앞선 사원의 급여를 본의 급여와 함께 출력
      FROM EMP
     WHERE JOB = 'SALESMAN';
    ```
    |  ENAME |  HIREDATE  |  SAL | LAG_SAL |
    |:------:|:----------:|:----:|:-------:|
    |  ALLEN | 1981-02-20 | 1600 |         |
    |  WARD  | 1981-02-22 | 1250 |    1600 |
    | TURNER | 1981-09-08 | 1500 |    1250 |
    | MARTIN | 1981-09-28 | 1250 |    1500 |
    </br>
    
    ```SQL
    -- 두번째 인자 : 몇 번째 앞의 행을 가져올 것인지
    -- 세번째 인자 : NVL, ISNULL 기능과 같음
    SELECT ENAME, HIREDATE, SAL
         , LAG (SAL, 2, 0) OVER (ORDER BY HIREDATE) AS LAG_SAL -- 본인보다 입사일자가 두 명 앞선 사원의 급여를 본의 급여와 함께 출력하는데, NULL이면 0으로 출력
      FROM EMP
     WHERE JOB = 'SALESMAN';
    ```
    |  ENAME |  HIREDATE  |  SAL | LAG_SAL |
    |:------:|:----------:|:----:|:-------:|
    |  ALLEN | 1981-02-20 | 1600 |       0 |
    |  WARD  | 1981-02-22 | 1250 |       0 |
    | TURNER | 1981-09-08 | 1500 |    1600 |
    | MARTIN | 1981-09-28 | 1250 |    1250 |
</br>

4. LEAD 함수
    </br>: 파티션별 윈도우에서 이후 몇 번째 행의 값을 가져옴 (SQL Server에서 지원 안함)


    ```SQL
    -- 두번째 인자 : 몇 번째 후의 행을 가져올 것인지
    -- 세번째 인자 : NVL, ISNULL 기능과 같음
    SELECT ENAME, HIREDATE, SAL
         , LEAD (HIREDATE, 1) OVER (ORDER BY HIREDATE) AS LEAD_HIREDATE -- 입사일자가 빠른순으로 정렬하고, 바로 다음에 입사한 직원의 입사일자 출력
      FROM EMP
     WHERE JOB = 'SALESMAN';
    ```
    |  ENAME |  HIREDATE  | LEAD_HIREDATE |
    |:------:|:----------:|:-------------:|
    |  ALLEN | 1981-02-20 |   1981-02-22  |
    |  WARD  | 1981-02-22 |   1981-09-08  |
    | TURNER | 1981-09-08 |   1981-09-28  |
    | MARTIN | 1981-09-28 |       0       |
</br>

##### 그룹 내 비율 함수

1. RATIO_TO_REPORT
    </br>: 파티션 내 전체 칼럼 값에 대한 행별 칼럼 값의 백분율을 소수점으로 구함
    </br>`0 < 결과 값 <= 1`
    
    ```SQL
    -- 1600 / 5600
    -- 1250 / 5600
    -- 1250 / 5600
    -- 1500 / 5600
    SELECT ENAME, SAL, ROUND(RATIO_TO_REPORT (SAL) OVER (),2) AS RR
    FROM EMP
    WHERE JOB = 'SALESMAN';
    ```
    |  ENAME |  SAL |  RR  |
    |:------:|:----:|:----:|
    |  ALLEN | 1600 | 0.29 |
    |  WARD  | 1250 | 0.22 |
    | MARTIN | 1250 | 0.22 |
    | TURNER | 1500 | 0.27 |
</br>

2. PERCENT_RANK
    </br>: 파티션별 윈도우에서 제일 먼저 나오는것은 0, 제일 늦게 나오면 1로 하여 행의 순서별 백분율을 구함 (SQL Server에서 지원안함)
    </br>  같은 값은 같은 ORDER로 취급
    </br>`0 <= 결과 값 <= 1`
    
    ```SQL
    -- DEPTNO 10의 경우 3건이므로 구간은 2개 → 0, 0.5, 1
    -- DEPTNO 20의 경우 5건이므로 구간은 4개 → 0, 0.25, 0.5, 0.75, 1
    SELECT DEPTNO, ENAME, SAL, PERCENT_RANK() OVER (PARTITION BY DEPTNO ORDER BY SAL DESC) AS PR
    FROM EMP;
    ```
    | DEPTNO |  ENAME |  SAL |  PR  |
    |--------|:------:|:----:|:----:|
    |   10   |  KING  | 5000 | 0.00 |
    |   10   |  CLARK | 2450 | 0.50 |
    |   10   | MILLER | 1300 | 1.00 |
    |   20   |  SCOTT | 3000 | 0.00 |
    |   20   |  FORD  | 3000 | 0.00 |
    |   20   |  JONES | 2975 | 0.50 |
    |   20   |  ADAMS | 1100 | 0.75 |
    |   20   |  SMITH |  800 | 1.00 |
</br>

3. CUME_DIST
    </br>: 파티션별 윈도우의 전체건수에서 현재 행보다 작거나 같은 건수에 대한 누적백분율을 구함
    </br>`0 < 결과 값 <= 1`
    
    ```SQL
    -- DEPTNO가 10인 경우 전체 3건이므로 0.3333단위 간격
    -- DEPTNO가 20인 경우 전체 5건이므로 0.2000단위 간격
    SELECT DEPTNO, ENAME, SAL, CUM_DIST() OVER (PARTITION BY DEPTNO ORDER BY SAL DESC) AS CD
    FROM EMP;
    ```
    | DEPTNO |  ENAME |  SAL |  CD  |
    |--------|:------:|:----:|:----:|
    |   10   |  KING  | 5000 | 0.33 |
    |   10   |  CLARK | 2450 | 0.67 |
    |   10   | MILLER | 1300 | 1.00 |
    |   20   |  SCOTT | 3000 | 0.40 |
    |   20   |  FORD  | 3000 | 0.40 |
    |   20   |  JONES | 2975 | 0.60 |
    |   20   |  ADAMS | 1100 | 0.80 |
    |   20   |  SMITH |  800 | 1.00 |
</br> 

4. NTILE 함수
   </br>: 파티션별 전체 건수를 ARGUMENT값으로 N등분한 결과
   
   ```SQL
   -- NTILE(4)SMS 14명의 팀원을 4개조로 나눈다는 의미. 14/4 = 3, 14%3=2 이므로 나머지 두 명은 앞의 조부터 할당
   -- 즉 4명 + 4명 + 3명 + 3명
   SELECT ENAME, SAL, NTILE(4) OVER (PARTITION BY SAL DESC) AS NT
   FROM EMP;
   ```
    |  ENAME |  SAL | NT |
    |:------:|:----:|:--:|
    |  KING  | 5000 |  1 |
    |  FORD  | 3000 |  1 |
    |  SCOTT | 3000 |  1 |
    |  JONES | 2975 |  1 |
    |  BLAKE | 2850 |  2 |
    |  CLARK | 2450 |  2 |
    |  ALLEN | 1600 |  2 |
    | TURNER | 1500 |  2 |
    | MILLER | 1300 |  3 |
    |  WARD  | 1250 |  3 |
    | MARTIN | 1250 |  3 |
    |  ADAMS | 1100 |  4 |
    |  JAMES |  950 |  4 |
    |  SMITH |  800 |  4 |

---

### [제5절] Top N 쿼리

#### ROWNUM 슈도 칼럼

Pseudo Column으로서 SQL 처리 결과 집합의 각 행에 대해 임시로 부여되는 일련번호. 테이블이나 집합에서 원하는 만큼의 행만 가져오고 싶을때 WHERE 절에서 행의 개수를 제한하는 목적
</br>Oracle의 경우 정렬이 완료된 후 데이터의 일부가 출력되는 것이 아닌, `데이터의 일부가 먼저 추출된 후 데이터에 대한 정렬 작업`이 일어나므로 주의
</br>

#### 한 건의 행만 가져오기

```SQL
-- 한 건의 행만 가져오기
SELECT PLAYER_NAME FROM PLAYER WHERE ROWNUM <= 1;
SELECT PLAYER_NAME FROM PLAYER WHERE ROWNUM < 2;

-- 두 건 이상의 N행을 가져오고 싶을때
SELECT PLAYER_NAME FROM PLAYER WHERE ROWNUM <= N;
SELECT PLAYER_NAME FROM PLAYER WHERE ROWNUM < N+1;
```
</br>

#### TOP 절

SQL Server는 TOP절을 사용해 결과 집합으로 출력되는 행의 수를 제한.
</br>

```SQL
SELECT TOP(2)
       ENAME, SAL
  FROM EMP
ORDER BY SAL DESC;
```
| ENAME |  SAL |
|:-----:|:----:|
|  KING | 5000 |
| SCOTT | 3000 |
</br>

```SQL
SELECT TOP(2) WITH TIES -- 동일 수치의 데이터를 추가로 더 추출
  FROM EMP
ORDER BY SAL DESC;
```
| ENAME |  SAL |
|:-----:|:----:|
|  KING | 5000 |
| SCOTT | 3000 |
|  FORD | 3000 |
</br>

#### ROW LIMITING 절

Oracle 12.1, SQL Server 2012 이상부터 ROW LIMITING으로 TOP N 쿼리를 작성할 수 있음. ORDER BY 절 다음에 기술하며 ORDER BY 절과 함께 수행됨.
</br>

`[OFFSET offset {ROW | ROWS}]`

`[FETCH {FIRST | NEXT} [{rowcount | percent PERCENT}] {ROW | ROWS} {ONLY | WITH TIES}]`

</br>
- OFFSET offset : 건너뛸 행의 개수 지정
- FETCH : 반환할 행의 개수나 백분율 지정
- ONLY : 지정된 행의 개수나 백분율 만큼 행 반환
- WITH TIES : 마지막 행에 대한 동순위를 포함해서 반홪
</br>

```SQL
SELECT EMPNO, SAL 
  FROM EMP
ORDER BY SAL, EMPNO FETCH FIRST 5 ROWS ONLY;
```
| EMPNO |  SAL |
|:-----:|:----:|
|  7369 |  800 |
|  7900 |  950 |
|  7876 | 1100 |
|  7654 | 1250 |
|  7521 | 1250 |
</br>

```SQL
SELECT EMPNO, SAL
  FROM EMP
ORDER BY SAL, EMPNO OFFSET 5 ROWS; -- 상위 5개 행을 건너뜀
```
| EMPNO |  SAL |
|:-----:|:----:|
|  7934 | 1300 |
|  7844 | 1500 |
|  7499 | 1600 |
|  ...  |  ... |

---

### [제6절] 계층형 질의와 셀프 조인

#### 계층형 데이터?

동일 테이블에 계층적으로 상위와 하위 데이터가 포함된 데이터 (관리자-사원 관계, 상위조직-하위조직 관계)
</br>

#### 셀프조인
동일 테이블 사이의 조인. FROM 절에 동일 테이블이 두 번 이상 나타남.
</br>동일 테이블 사이의 조인을 수행하기 위해서는 `테이블 식별을 위해 Alias 사용 필수` 
</br>셀프조인은 동일한 테이블이지만, 개념적으로는 두 개의 서로 다른 테이블을 사용하는 것과 동일함
</br>

```SQL
-- JONES의 자식노드를 조회하는 쿼리
SELECT WORKER.EMPNO, WORKER.ENAME, WORKER.MGR
FROM EMP MANAGER, EMP WORKER -- 동일한 테이블처럼 처리하기위해서는 Alias 사용필수
WHERE MANAGER.ENAME = 'JONES'
  AND WORKER.MGR = MANAGER.EMPNO;
```
| EMPNO | ENAME | MGR  |
|:-----:|:-----:|------|
|  7788 | SCOTT | 7566 |
|  7902 |  FORD | 7566 |
</br>

```SQL
-- JONES의 자식노드의 자식노드를 조회하는 쿼리
-- 순방향 전개
SELECT WORKER.EMPNO, WORKER.ENAME, WORKER.MGR
 FROM EMP MANAGER, EMP MID, EMP WORKER
WHERE MANAGER.ENAME = 'JONES'
  AND MID.MGR = A.EMPNO
  AND WORKER.MGR = MID.EMPNO;
```
| EMPNO | ENAME |  MGR |
|:-----:|:-----:|:----:|
|  7369 | SMITH | 7902 |
|  7876 | ADAMS | 7788 |
</br>

```SQL
-- SMITH의 부모노드를 조회하는 쿼리
SELECT MANAGER.EMPNO, MANAGER.ENAME, MANAGER.MGR 
  FROM EMP WORKER, EMP MANAGER
 WHERE WORKER.ENAME = 'SMITH'
   AND MANAGER.EMPNO = WORKER.MGR;
```
| EMPNO | ENAME |  MGR |
|:-----:|:-----:|:----:|
|  7902 |  FORD | 7566 |
</br>

```SQL
-- SMITH의 부모노드의 부모노드를 조회하는 쿼리
SELECT MANAGER.EMPNO, MANAGER.ENAME, MANAGER.MGR
  FROM EMP WORKER, EMP MID, EMP MANAGER
 WHERE WORKER.ENAME = 'SMITH'
   AND MID.EMPNO = WORKER.MGR
   AND MANAGER.EMPNO = MID.MGR;
```
| EMPNO | ENAME |  MGR |
|:-----:|:-----:|:----:|
|  7566 | JONES | 7839 |
</br>

#### 계층형 질의

1. Oracle 계층형 질의
    
    - START WITH 절 : 계층구조전개의 시작위치를 지정하는 구문
    
    - CONNECT BY 절 : 다음에 전개될 자식 데이터를 지정하는 구문
    
    - PRIOR 절 : CONNECT BY 절에 사용되며, 현재 읽은 칼럼을 지정
        - (FK) = PRIOR (PK) : 순방향 전개
        - (PK) = PRIOR (FK) : 역방향 전개
    
    - NOCYCLE : 이미 나타났던 동일한 데이터가 전개 중에 다시 나타날때 사이클(CYCLE)이 발생했다고 함. 사이클이 발생하면 런타임 오류가 발생하게 되는데, `NOCYCLE을 추가하면 오류를                  발생시키지 않고 사이클이 발생한 이후의 데이터를 전개하지 않음`
    
    - ORDER SIBLINGS BY : 형제 노드(동일 LEVEL) 사이에서 정렬 수행
    
    - WHERE : 모든 전개를 수행한 후 지정된 조건을 만족하는 데이터만 추출(필터링)
</br>

2. Oracle 계층형 질의에서 사용되는 가상 칼럼

    - LEVEL : 루트 테이터는 1, 그 하위 데이터는 2. 리프(LEAF) 데이터까지 1씩 증가
    
    - CONNECT_BY_ISLEAF : 전개 과정ㅁ에서 해당 데이터가 리프 데이터면 1, 그렇지 않으면 0
    
    - CONNECT_BY_ISCYCLE : 전개 과정에서 자식을 갖는데, 해당 데이터가 조상으로서 존재하면 1, 그렇지 않으면 0 (CYCLE 옵션 사용할때만 사용 가능)
</br>

```SQL
SELECT  LEVEL AS LV, LPAD (' ', (LEVEL - 1) * 2) || EMPNO AS EMPNO, MGR
        , CONNECT_BY_ISLEAF AS ISLEAF
  FROM EMP
START WITH MGR IS NULL
CONNECT BY MGR = PRIOR EMPNO;
```
| LV | EMPNO |  MGR | ISLEAF |
|:--:|:-----:|:----:|:------:|
|  1 |  7839 |      |      0 |
|  2 |  7566 | 7839 |      0 |
|  3 |  7788 | 7566 |      0 |
|  4 |  7876 | 7788 |      1 |
|  3 |  7902 | 7566 |      0 |
|  4 |  7369 | 7902 |      1 |
|  2 |  7698 | 7839 |      0 |
|  3 |  7499 | 7698 |      1 |
|  3 |  7521 | 7698 |      1 |
|  3 |  7654 | 7698 |      1 |
|  3 |  7844 | 7698 |      1 |
|  3 |  7900 | 7698 |      1 |
|  2 |  7782 | 7839 |      0 |
|  3 |  7934 | 7782 |      1 |
</br>

<img src="./src/forward.png" alt="순방향 예시" width="500" height="600">
</br>

```SQL
SELECT  LEVEL AS LV, LPAD (' ', (LEVEL - 1) * 2) || EMPNO AS EMPNO, MGR
        , CONNECT_BY_ISLEAF AS ISLEAF
  FROM EMP
START WITH MGR = 7876
CONNECT BY EMPNO = PRIOR MGR;
```
| LV | EMPNO |  MGR | ISLEAF |
|:--:|:-----:|:----:|:------:|
|  1 |  7876 | 7788 |    0   |
|  2 |  7788 | 7566 |    0   |
|  3 |  7566 | 7839 |    0   |
|  4 |  7839 |      |    1   |
</br>

<img src="./src/backward.png" alt="역방향 예시" width="500" height="600">

---

### [제7절] PIVOT 절과 UNPIVOT 절

#### PIVOT 절

행을 열로 전환
</br>

```SQL
SELECT *
  FROM (SELECT JOB, DEPTNO, SAL FROM EMP
 PIVOT (SUM(SAL) FOR DEPTNO IN (10, 20, 30))
ORDER BY 1;
```
|    JOB    |  10  |  20  |  30  |
|:---------:|:----:|:----:|:----:|
|  ANALYST  |      | 6000 |      |
|   CLERK   | 1300 | 1900 |  950 |
|  MANAGER  | 2450 | 2975 | 2850 |
| PRESIDENT | 5000 |      |      |
|  SALESMAN |      |      | 5600 |
</br>

```SQL
-- 집계함수와 IN 절에 별칭 지정. 별칭을 지정하면 결과 집합의 열 명이 변경됨
SELECT *
  FROM (SELECT JOB, DEPTNO, SAL FROM EMP)
 PIVOT (SUM(SAL) AS SAL FOR DEPTNO IN (10 AS D10, 20 AS D20, 30 AS D30))
ORDER BY 1;
```
|    JOB    | D10_SAL | D20_SAL | D30_SAL |
|:---------:|:-------:|:-------:|:-------:|
|  ANALYST  |         |    6000 |         |
|   CLERK   |    1300 |    1900 |     950 |
|  MANAGER  |    2450 |    2975 |    2850 |
| PRESIDENT |    5000 |         |         |
|  SALESMAN |         |         |    5600 |
</br>

```SQL
-- PIVOT 절을 사용할 수 없는 경우, 집계함수와 CASE 표현식으로 PIVOT 수행
SELECT  JOB
      , SUM (CASE DEPTNO WHEN 10 THEN SAL END) AS D10_SAL
      , SUM (CASE DEPTNO WHEN 20 THEN SAL END) AS D20_SAL
      , SUM (CASE DEPTNO WHEN 30 THEN SAL END) AS D30_SAL
  FROM EMP
GROUP BY JOB
ORDER BY JOB;
```
</br>

#### UNPIVOT 절

열을 행으로 전환

</br>
만약, 다음과 같이 조회되는 테이블이 있을때 UNPIVOT을 수행해보자.

```SQL
SELECT * FROM T1 ORDER BY JOB;
```
|   JOB   | D10_SAL | D20_SAL | D10_CNT | D20_CNT |
|:-------:|:-------:|:-------:|:-------:|:-------:|
| ANALYST |         |    6000 |       0 |       2 |
|  CLERK  |    1300 |    1900 |       1 |       2 |
</br>

```SQL
-- D10_SAL, D20_SAL 열이 행으로 전환
SELECT JOB, DEPTNO, SAL
  FROM T1
UNPIVOT (SAL FOR DEPTNO IN (D10_SAL, D20_SAL))
ORDER BY 1, 2;
```
|   JOB   |  DEPTNO |  SAL |
|:-------:|:-------:|:----:|
| ANALYST | D20_SAL | 6000 |
|  CLERK  | D10_SAL | 1900 |
|  CLERK  | D20_SAL | 1900 |
</br>

```SQL
-- IN 절에 별칭을 지정하면 지정한 열의 값 변경 가능
SELECT JOB, DEPTNO, SAL
  FROM T1
UNPIVOT (SAL FOR DEPTNO IN (D10_SAL AS 10, D20_SAL AS 20)
ORDER BY 1M2;
```
|   JOB   | DEPTNO |  SAL |
|:-------:|:------:|:----:|
| ANALYST |     20 | 6000 |
|  CLERK  |     10 | 1300 |
|  CLERK  |     20 | 1900 |
</br>

```SQL
-- UNPIVOT 절을 사용할 수 없는 경우, 카티션 곱을 사용해 수행할 수 있다.
-- UNPIVOT 할 열의 개수만큼 행을 복제하고, CASE 표현식으로 UNPIVOT할 열을 선택하는 방식
SELECT A.JOB
     , CASE B.LV WHEN 1 THEN 10         WHEN 2 THEN 20          END AS DEPTNO
     , CASE B.LV WHEN 1 THEN A.D10_SAL  WHEN 2 THEN A.D20_SAL   END AS SAL
     , CASE B.LV WHEN 1 THEN A.D10_CNT  WHEN 2 THEN A.D20_CNT   END AS CNT
  FROM T1 A
        , (SELECT LEVEL AS LV FROM DUAL CONNECT BY LEVEL <= 2 ) B
ORDER BY 1, 2;
```
|   JOB   | DEPTNO |  SAL | CNT |
|:-------:|:------:|:----:|-----|
| ANALYST |     10 |      |   0 |
| ANALYST |     20 | 6000 |   2 |
|  CLERK  |     10 | 1300 |   1 |
|  CLERK  |     20 | 1900 |   2 |
---

### [제8절] 정규 표현식 </h4>

#### 정규 표현식?

문자열의 규칙을 표현하는 검색 패턴으로 주로 문자열 검색과 치환에 사용됨
</br>

1. POSIX 연산자
```BASH
. : 모든 문자와 일치
| : 대체 문자를 구분
\ : 다음 문자를 일반 문자로 취급

^ : 문자열의 시작
$ : 문자열의 끝
? : 0회 또는 1회 일치
* : 0회 또는 그 이상의 횟수로 일치
+ : 1회 또는 그 이상의 횟수로 일치

{m}         : m회 일치
{m,}        : 최소 m회 일치
{,m}        : 최대 m회 일치
{m,n}       : 최소 m회, 최대 n회 일치

(expr)      : 괄호 안의 표현식을 하나의 단위로 취급
\n          : n번째 서브 표현식과 일치, n은 1에서 9 사이의 정수
[char···]   : 문자 리스트 중 한 문자와 일치
[^char···]  : 문자 리스트에 포함되지 않은 한 문자와 일치

[:digit:]   : (= [0-9])         숫자              
[:lower:]   : (= [a-z])         소문자            
[:upper:]   : (= [A-Z])         대문자            
[:alpha:]   : (= [a-zA-Z])      영문자            
[:alnum:]   : (= [0-9a-zA-Z])   영문자와 숫자     
[:xdigit:]  : (= [0-9a-fA-F])   16진수            
[:punct:]   : (= ^[:alnum:][:cntrl:]) 구두점 기호       
[:blank:]   : 공백 문자
[:space:]   : 공간 문자 (space, enter, tab)
```
</br>

2. PERL 정규표현식 연산자

```BASH
\d : (= [[:digit:]])    숫자                       
\D : (= [^[:digit:]])   숫자가 아닌 모든 문자
\w : (= [[:alnum:]_])   숫자와 영문자 (underbar 포함)
\W : (= [^[:alnum:]_])  숫자와 영문자가 아닌 모든 문자 (underbar 제외)
\s : (= [[:space:]])    공백 문자
\S : (= [^[:space:]])   공백 문자가 아닌 모든 문자

?? : 0회 또는 1회 일치
*? : 0회 또는 그 이상의 횟수로 일치
+? : 1회 또는 그 이상의 횟수로 일치
{m}? : m회 일치
{m,}? : 최소 m회 일치
{,m}? : 최대 m회 일치
{m,n}? : 최소 m회, 최대 n회 일치
```
</br>

#### 정규 표현식 조건과 함수

1. REGEXP_LIKE 조건
    </br>: source_char가 pattern과 일치하면 True, 아니면 False
    </br>`REGEXP_LIKE (source_char, pattern [, match_param])`
    </br>

    - source_char   : 검색 문자열 지정
    - pattern       : 검색 패턴 지정
    - mamtch_param  : 일치 옵션 지정 (기본값은 c)
      - i : 대소문자 무시
      - c : 대소문자 구분
      - n : dot(.)을 개행 문자와 일치
      - m : 다중행 모드
      - x : 검색 패턴의 공백 문자 무시
</br>

2. REGEXP_REPLACE 함수
    </br>: source_char에서 일치한 pattern을 replace_string으로 변경한 문자값을 반환
    </br>`REGEXP_REPLACE (source_char, pattern [, replace_string [, position [, occurence [, match_param]]]])`
    </br>

    - replace_string    : 변경 문자열 지정
    - position          : 검색 시작 위치 지정 (기본값은 1)
    - occurrence        : 패턴 일치 횟수 지정 (기본값은 1)
</br>

3. REGEXP_SUBSTR 함수
    </br>: source_char에서 일치한 pattern반환
    </br>`REGEXP_SUBSTR (source_char, pattern [, position [, occurence [, match_param [, subexpr]]]])`
    </br>

    - substr : 서브 표현식을 지정 (0은 전체 패턴 / 1 이상은 서브표현식 / 기본값은 0)
</br>

4. REGEXP_INSTR 함수
    </br>: source_char에서 일치한 pattern의 시작위치를 정수로 반환
    </br>`REGEXP_INSTR (source_char, pattern [, position [, occurence [, return_opt [, match_param [, subexpr]]]]])`
    </br>

    - return_opt : 반환 옵션 지정 (0은 시작 위치 / 1은 다음 위치 / 기본값은 0)
</br>

5. REGEXP_COUNT 함수 
    </br>: source_char에서 일치한 pattern의 횟수를 반환
    </br>`REGEXP_COUNT (source_char, pattern [, position [, match_param]])`
    </br>

---

### Reference
- SQL 전문가 가이드
