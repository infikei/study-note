# IN과 EXISTS의 차이

> 2024.12.31. 화 작성 (by 인피케이)

### 1. `IN`

- 서브쿼리의 결과를 모두 가져온 후, 메인 쿼리의 WHERE 절에서 결과 집합과 비교하는 방식이다.
- 서브쿼리의 결과에 `NULL` 이 포함되는 경우에는 결과가 의도와 다르게 동작할 수 있으므로 주의해야 한다.

```sql
SELECT *
FROM a
WHERE a.key IN (
    SELECT b.key
    FROM b
);
```

- `b` 테이블에 먼저 접근한다.
- `b.key` 를 `IN` 리스트에 나열한 후 `a.key` 에 공급한다.
- 즉, 이 쿼리에서 `b` 테이블은 공급자 역할을 수행한다.

---

### 2. `EXISTS`

- 서브쿼리에서 조건을 만족하는 행이 존재하는지 여부를 확인하는 방식이다.
- 즉, 서브쿼리에서 결과 집합을 반환하는 것이 아니라, 서브쿼리에서 조건을 만족하는 행을 찾는 즉시 반환하는 것이다. 따라서 대량의 데이터 처리에서 더 효율적일 수 있다.
- `NULL` 값 문제를 피할 수 있다는 장점이 있다.
- `EXISTS` 뒤에 오는 SELECT 절에서 `*` 이나 `1` 대신 무엇을 넣어도 상관없다. 어차피 조건을 만족하는 행이 존재하는지 여부만 확인하기 때문이다.
- 서브쿼리에서 조건을 만족하는 행이 최소 하나가 있다면 바로 `TRUE` 로 판단한다.

```sql
SELECT *
FROM a
WHERE EXISTS (
    SELECT 1
    FROM b
    WHERE b.key = a.key
);
```

- `a` 테이블에 먼저 접근한다.
- `a` 테이블의 각 행에 대하여 `EXISTS` 가 `TRUE` 인지 아닌지를 체크한다. 서브쿼리에서 조건을 만족하는 행이 존재한다면 `EXISTS` 는 `TRUE` 가 되고, 그렇지 않다면 `EXISTS` 는 `FALSE` 가 된다.
- 즉, 이 쿼리에서 `b` 테이블은 확인자 역할을 수행한다.

---

### 3. `IN` 과 `EXISTS` 의 차이

- `IN` 과 `EXISTS` 는 둘 다 WHERE 절에서 사용되며, 조건에 따라 데이터를 걸러내어 결과를 조회할 때 사용된다.
- `IN` 에서는 서브쿼리 → 메인 쿼리의 방향으로 진행된다. 반면 `EXISTS` 에서는 메인 쿼리 → 서브쿼리의 방향으로 진행된다.
- `IN` 은 값이 특정 목록에 존재하는지 확인하기 위한 방식이다. 반면 `EXISTS` 는 서브쿼리 조건을 만족하는 행이 존재하는지 확인하기 위한 방식이다.
- `IN` 은 작은 데이터 집합에 적합하다. 반면 `EXISTS` 는 큰 데이터 집합에 적합하다.
- `IN` 방식에서는 서브쿼리에 `NULL` 이 있는 경우 결과가 의도와 다를 수 있다. 반면 `EXISTS` 방식은 `NULL` 에 영향을 받지 않는다.
- `IN` 방식에서는 서브쿼리 결과를 가져와서 모두 비교한다. 반면 `EXISTS` 방식에서는 서브쿼리에서 조건을 만족하는 즉시 반환하므로 더 빠를 가능성이 있다.
- 따라서 서브쿼리 결과가 크거나 `NULL` 문제를 방지해야 하는 경우 `EXISTS` 방식이 적합하다.

---

### 4. `IN` 에서 서브쿼리 결과에 `NULL` 이 포함되는 경우, 왜 문제가 되는가?

- 간단히 말하면, 서브쿼리 결과에 `NULL` 이 포함되는 경우, `IN` 조건은 `NULL` 로 평가될 수 있으며, 이로 인해 메인 쿼리의 결과에도 영향을 미칠 수 있다.
    - 이는 `WHERE ... IN (...)` 이 내부적으로 조건을 `OR` 연산으로 풀어서 평가하기 때문이다.
    - 예를 들어, `car_id IN (1, NULL)` 은 내부적으로 `car_id = 1 OR car_id = NULL` 로 평가된다.
    - `car_id = 1` → `TRUE` 또는 `FALSE`
    - `car_id = NULL` → `NULL`
    - 따라서 위의 `IN` 조건은 다음과 같이 평가된다.
        - `TRUE OR NULL` → `TRUE`
        - `FALSE OR NULL` → `NULL`
    - 즉, 특정 상황에서는 `NULL` 이라는 결과로 인해 그 이후의 과정에서 의도와 다른 결과가 나올 수 있다.
- SQL의 WHERE 절은 조건이 `TRUE` 로 평가된 행만 결과에 포함하며, 조건이 `FALSE` 나 `NULL` 로 평가된 행은 결과에서 제외한다.
- 따라서 `IN` 조건으로 인해 의도와 다르게 일부 행이 제외될 수 있고, 경우에 따라서는 메인 쿼리의 결과에서 0개의 행이 반환될 수도 있다.

---

### 5. `NOT IN` 과 `NOT EXISTS` 의 차이를 보여주는 예제

1. 데이터 준비

    ```sql
    CREATE TABLE car (car_id INT, car_name VARCHAR(50));
    
    INSERT INTO car VALUES
    (1, '세단'),
    (2, 'SUV'),
    (3, '트럭');
    
    CREATE TABLE rental_history (history_id INT, car_id INT, start_date DATE, end_date DATE);
    
    INSERT INTO rental_history VALUES
    (1, 1, '2024-11-24', '2024-11-27'),
    (2, NULL, '2024-12-07', '2024-12-16');
    ```

2. `NOT IN` 

    ```sql
    SELECT *
    FROM car
    WHERE car_id NOT IN (
        SELECT car_id
        FROM rental_history
    );
    ```

    - 서브쿼리 결과

        ```sql
        car_id
        ------
        1
        NULL
        ```

    - 메인 쿼리의 WHERE 절 평가 과정
        - `NOT IN` 은 각 행에 대해 서브쿼리 결과와 비교한다.
        - `car_id = 1` 행 : `1 NOT IN (1, NULL)` → `NOT (TRUE OR NULL)` → `NOT TRUE` → `FALSE` (1이 서브쿼리에 존재) → 메인 쿼리 결과에서 제외
        - `car_id = 2` 행 : `2 NOT IN (1, NULL)` → `NOT (FALSE OR NULL)` → `NOT NULL` → `NULL` → 메인 쿼리 결과에서 제외
        - `car_id = 3` 행 : `3 NOT IN (1, NULL)` → `NOT (FALSE OR NULL)` → `NOT NULL` → `NULL` → 메인 쿼리 결과에서 제외
    - 메인 쿼리 결과 (결과 없음)

        ```markdown
        car_id | car_name
        ------ | --------
        
        ```

3. `NOT EXISTS` 

    ```sql
    SELECT *
    FROM car c
    WHERE NOT EXISTS (
        SELECT 1
        FROM rental_history h
        WHERE h.car_id = c.car_id
    );
    ```

    - 평가 과정
        - `NOT EXISTS` 는 메인 테이블의 각 행에 대해 서브쿼리 조건을 평가한다.
        - `car_id = 1` 행 : 서브쿼리 조건 `TRUE` (조건 만족하는 행이 존재함) → 메인 쿼리 결과에서 제외
        - `car_id = 2` 행 : 서브쿼리 조건 `FALSE` (조건 만족하는 행이 존재하지 않음) → 메인 쿼리 결과에 포함
        - `car_id = 3` 행 : 서브쿼리 조건 `FALSE` (조건 만족하는 행이 존재하지 않음) → 메인 쿼리 결과에 포함
    - 메인 쿼리 결과

        ```markdown
        car_id | car_name
        ------ | --------
        2      | SUV
        3      | 트럭
        ```

---

### 6. 참고 자료

1. [https://velog.io/@wogud9675/MySQL-NOT-IN-과-NOT-EXISTS의-차이점](https://velog.io/@wogud9675/MySQL-NOT-IN-%EA%B3%BC-NOT-EXISTS%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90) 
2. [https://inpa.tistory.com/entry/MYSQL-📚-서브쿼리-연산자-EXISTS-총정리-성능-비교](https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-%EC%84%9C%EB%B8%8C%EC%BF%BC%EB%A6%AC-%EC%97%B0%EC%82%B0%EC%9E%90-EXISTS-%EC%B4%9D%EC%A0%95%EB%A6%AC-%EC%84%B1%EB%8A%A5-%EB%B9%84%EA%B5%90)
