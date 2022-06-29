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
    - Grouping Columns의 수가 N이면, N+1 Level의 Subtotal 생성
    - 인수 순서가 바뀌면 수행결과도 바뀜
    
    ```SQL
    SELECT B.DNAME, A.JOB, COUNT(*) AS EMP_CND, SUM(A.SAL) AS SAL_SUM
    FROM EMP A, DEPT B
    WHERE B.DEPTNO = A.DEPTNO
    GROUP BY B.DNAME, A.JOB
    ORDER BY B.NAME, A.JOB
    ```
    ```VM
    DDDDDDD
    ```
 
3. CUBE 함수
4. GROUPING SETS 함수

---

<h4> 4) 윈도우 함수 </h4>
<details>
</details>

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
