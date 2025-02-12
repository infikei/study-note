# WITH 절과 공통 테이블 표현식

> 2025.01.02. 목 작성 (by 인피케이) (2025.02.12. 수 업데이트)
> 

## 1. WITH 절과 공통 테이블 표현식 (CTE, Common Table Expression)

`WITH` 절은 MySQL에서 공통 테이블 표현식(CTE)을 정의하기 위해 사용하는 구문이다.

공통 테이블 표현식(CTE)은 단일 SQL 문 내에서 임시 이름이 붙은 임시 결과 집합으로, 해당 SQL 문 내에서 여러 번 참조하여 사용할 수 있다.

---

## 2. WITH 절과 공통 테이블 표현식(CTE)의 특징

- `WITH` 절과 공통 테이블 표현식은 성능 최적화 관점에서 반복된 서브쿼리를 줄이고 가독성과 유지보수성을 높이는 데 도움을 준다.
- 중첩된 서브쿼리를 사용하지 않고도 논리적으로 쿼리를 나눌 수 있다.
- 공통 테이블 표현식은 같은 statement 내에서 여러 번 참조될 수 있으며, 여러 번 참조되더라도 한 번만 실행되어 그 결과가 메모리에 저장된다. 따라서 동일한 데이터를 여러 번 참조해야 할 때 유용하다.
- 공통 테이블 표현식은 메인 쿼리 실행 중에만 존재한다. 또한 임시 테이블과 달리, 데이터베이스에 물리적으로 생성되지 않고 메모리에서 관리된다.
- 공통 테이블 표현식은 다른 공통 테이블 표현식을 참조하여 정의될 수 있다.
- 공통 테이블 표현식은 자기 자신을 참조하여 재귀적으로 정의될 수 있다. 이때는 `RECURSIVE` 키워드를 함께 사용한다.
- 옵티마이저는 상황에 따라 공통 테이블 표현식을 메인 쿼리로 병합하여 성능을 향상시킬 수 있다.

---

## 3. 공통 테이블 표현식(CTE)의 동작 원리

- 공통 테이블 표현식은 먼저 정의된 쿼리를 실행한 후 메모리에 그 결과가 저장된다.
- 이후 메인 쿼리에서 이 임시 결과를 참조하거나, 다른 공통 테이블 표현식과 결합하여 사용한다.
- 쿼리 실행이 끝나면 공통 테이블 표현식은 메모리에서 사라지며 데이터베이스에 저장되지 않는다.

---

## 4. WITH 절의 기본 구조

메인 쿼리의 `SELECT` 절 앞에서 `WITH` 절을 사용하여 표현한다.

```sql
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
SELECT *
FROM cte_name;
```

여러 개의 공통 테이블 표현식을 정의할 경우에는 `,` 로 구분하여 작성한다.

```sql
WITH cte1 AS (
    SELECT a, b
    FROM table1
),
cte2 AS (
    SELECT c, d
    FROM table2
)
SELECT b, d
FROM cte1
JOIN cte2
WHERE cte1.a = cte2.c;
```

---

## 5. 공통 테이블 표현식(CTE)의 유형

### 비재귀적 CTE

단순히 결과 집합을 정의하는 기본적인 형태의 CTE이다.

### 재귀적 CTE

`RECURSIVE` 키워드를 사용하여 자기 자신을 참조하는 쿼리를 작성할 수 있다. 이러한 재귀적 CTE는 반드시 종료 조건을 포함해야 한다.

MySQL 공식 문서에서는 다음과 같은 예시를 제시하고 있다.

```sql
WITH RECURSIVE cte (n) AS (
    SELECT 1
    UNION ALL
    SELECT n + 1
    FROM cte
    WHERE n < 5
)
SELECT *
FROM cte;
```

- 위의 쿼리를 실행한 결과는 아래와 같다.

```
+------+
| n    |
+------+
|    1 |
|    2 |
|    3 |
|    4 |
|    5 |
+------+
```

또한 `cte_max_recursion_depth` 시스템 변수를 통해 재귀의 최대 깊이를 제한할 수 있다.

---

## 6. 참고 자료

1. https://dev.mysql.com/doc/refman/8.0/en/with.html#common-table-expressions-recursive 
2. [https://mooonstar.tistory.com/entry/MySQLWITH문을-사용하여-가상의-테이블을-만들어-사용하기](https://mooonstar.tistory.com/entry/MySQLWITH%EB%AC%B8%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%EA%B0%80%EC%83%81%EC%9D%98-%ED%85%8C%EC%9D%B4%EB%B8%94%EC%9D%84-%EB%A7%8C%EB%93%A4%EC%96%B4-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0) 
3. [https://inpa.tistory.com/entry/MYSQL-📚-WITH-임시-테이블](https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-WITH-%EC%9E%84%EC%8B%9C-%ED%85%8C%EC%9D%B4%EB%B8%94)
