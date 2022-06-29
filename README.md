# SQL 정리노트
## 1. SQL 활용
</br>

### 서브쿼리?
</br>**하나의 SQL 문안에 포함되어 있는 또 다른 SQL**


#### 동작되는 방식에 따른 서브쿼리 분류

1. Un_Correlated(비연관) 서브쿼리
    </br>서브쿼리가 메인쿼리 **칼럼을 갖고 있지 않은 형태**, 메인쿼리에 값을 제공하기 위한 목적
    
3. Correlatied(연관) 서브쿼리
    </br>서브쿼리가 메인쿼리 **칼럼을 갖고있는 형태**, 메인쿼리가 서브쿼리에서 조건이 맞는지 확인하기 위한 목적



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
    
5. 그 밖의 위치에서 사용하는 서브쿼리

    1. SELECT 절에 서브쿼리 (스칼라 서브쿼리, Scalar Subquery)
        </br>스칼라 서브쿼리는 **한 행, 한 칼럼만을 반환**하는 서브쿼리
        </br>`단일행 서브쿼리, 결과가 2건 이상이면 오류 반환`
      
    2. FROM 절에 서브쿼리 사용 (인라인 뷰, Inline View)
       </br>인라인 뷰를 사용하면 서브쿼리의 결과를 테이블처럼 사용 가능
       </br>`SELECT문을 객체로서 저장하여 테이블처럼 사용하는 View와 달리, 인라인 뷰는 쿼리 내에서 즉시 처리`

    3. HAVING 절에서 서브쿼리 사용


### 뷰?

테이블은 실제로 데이터를 갖고있지만, 뷰는 실제 데이터를 갖고있지 않고  **뷰 정의(View Definition)**  만을 갖음

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

### 집합 연산자?

</br>**여러 개의 결과 집합 간의 연산을 통해 결합**하는 방식

1. UNION
</br>합집한 연산 수행, **중복 제거** 
    
2. UNION ALL
</br>합집합 연산 수행, **중복 허용**
  
3. INTERSECT
  </br>교집합 연산 수행, **중복 제거**
  
4. EXCEPT
  </br>차집합 연산 수행, **중복 제거**

### 그룹함수

ANSI/ISO SQL 표준은 데이터 분석을 위해 다음의 3가지 함수를 정의

- **AGGREGATE FUNCTION** (= GROUP AGGREGATE FUNCTION)
    </br>COUNT, SUM, AVG, MAX, MIN 
    
- **GROUP FUNCTION**
    </br>ROLLUP, CUBE, GROUPING SETS
    
- **WINDOW FUNCTION**
    </br>ANALYTIC FUNCTION, RANK FUNCTION

1. ROLLUP 함수
    - ROLLUP에 지정된 Grouping Columns의 List는 Subtotal을 생성하기 위함
    - Grouping Columns의 수가 N이면, **N+1 Level**의 Subtotal 생성
    - 인수 순서가 바뀌면 수행결과도 바뀜
    
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
 
2. CUBE 함수
    - 명시한 모든 칼럼에 대해 Subtotal 생성
    - GROUPING COLUMNS의 수가 N, 2^N승 Subtotal 생성
    - ROLLUP에 비해 시스템에 부하를 줌
    - 인수의 순서가 바뀌는 경우, 행간에 정렬 순서는 바뀔 수 있어도 데이터 결과는 같음
    
    ```SQL
    -- 예제1 : CUBE 함수 + ORDER BY 절
    SELECT B.DNAME, A.JOB, COUNT(*) AS EMP_CNT, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    GROUP BY CUBE (B.DNAME, A.JOB)
    ORDER BY B.DNAME, A.JOB;
    
    
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


3. GROUPING SETS 함수
    - 표시된 인수들에 대한 개별 집계를 구할수 있음
    - 인수의 순서가 바뀌어도 결과 같음
    
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

#### 윈도우 함수?

</br>기존 관계형 데이터베이스는 칼럼과 칼럼 간의 연산·비교·연결, 집합에 대한 집계는 쉬웠지만 행과 행간의 관계를 정의·비교·연산하는 것을 하나의 SQL문으로 처리하는것은 어려움.
따라서 **INLINE VIEW를 이용해 복잡한 SQL문을 작성하던 것을 행과 행간의 관계를 쉽게 정의하기 위해 만든 함수**

#### 윈도우 함수 종류

- 그룹 내 **순위** 관련 함수 (RANK)
    </br>RANK, DENSE_RANK, ROW_NUMBER
    
- 그룹 내 **집계** 관련 함수 (AGGREGATE)
    </br>SUM, MAX, MIN, AVG, COUNT
    
- 그룹 내 **행순서** 관련 함수
    </br>FIRST_VALUE, LAST_VALUE, LAG, LEAD, FIRST_VALUE(=MAX), LAST_VALUE(=MIN)

- 그룹 내 **비율** 관련 함수
    </br>CUME_DIST, PERCENT_RANK, NTILE, RATIO_TO_REPORT
 
#### 그룹 내 순위 함수

1. RANK 함수
    </br>ORDER BY를 포함한 QUERY문에서 특정 항목에 대한 순위를 구하는 함수 
    </br>`동일한 값에 대해 동일한 순위 부여`

2. DENSE_RANK 함수
    </br>RANK와 유사하지만, `동일한 순위를 하나의 건수로 취급`
    
3. ROW_NUMBER 함수
    </br>RANK나 DENSE_RANK는 동일한 값에 대해서는 동일한 순위를 부여하지만,
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


#### 일반 집계 함수

#### 그룹 내 비율 함수

1. RATIO_TO_REPORT
    </br>파티션 내 전체 칼럼 값에 대한 행별 칼럼 값의 백분율을 소수점으로 구함
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
    
2. PERCENT_RANK
    </br>파티션별 윈도우에서 제일 먼저 나오는것은 0, 제일 늦게 나오면 1로 하여 행의 순서별 백분율을 구함 (SQL Server에서 지원안함)
    </br>같은 값은 같은 ORDER로 취급
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

3. CUME_DIST
    </br>파티션별 윈도우의 전체건수에서 현재 행보다 작거나 같은 건수에 대한 누적백분율을 구함
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
 
4. NTILE 함수
   </br>파티션별 전체 건수를 ARGUMENT값으로 N등분한 결과
   
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
   
6. SUM 함수

---

<h4> 5) Top N 쿼리 </h4>
<details>
</details>

---

<h4> 6) 계층형 질의와 셀프 조인 </h4>
<details>
</details>

---

<h4> 7) PIVOT 절과 UNPIVOT 절 </h4>
<details>
</details>

---

<h4> 8) 정규 표현식 </h4>
<details>
</details>

---
### Reference
- SQL 전문가 가이드
