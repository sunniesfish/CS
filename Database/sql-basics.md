# SQL 기본 문법 (SELECT, INSERT, UPDATE, DELETE)

## 개요

SQL(Structured Query Language)은 관계형 데이터베이스에서 데이터를 조작하고 정의하기 위한 표준 언어입니다. 기본 DML(Data Manipulation Language) 명령인 SELECT, INSERT, UPDATE, DELETE는 모든 데이터베이스 작업의 핵심입니다.

## SQL의 역사와 표준화

- 1970년대 IBM System R 프로젝트에서 SEQUEL로 시작
- 1982년 IBM SQL/DS에서 첫 상용화
- 1986년 ANSI SQL-86 (SQL1) 표준 제정
- 현재 SQL:2016이 최신 표준

## SELECT 문

### 기본 구조

```sql
SELECT [DISTINCT] 컬럼목록
FROM 테이블명
[WHERE 조건]
[GROUP BY 컬럼목록]
[HAVING 조건]
[ORDER BY 컬럼목록 [ASC|DESC]]
[LIMIT 개수];
```

### 기본 SELECT

```sql
-- 모든 컬럼 선택
SELECT * FROM employees;

-- 특정 컬럼 선택
SELECT employee_id, first_name, last_name FROM employees;

-- 컬럼 별칭 사용
SELECT
    employee_id AS emp_id,
    first_name AS 이름,
    salary * 12 AS annual_salary
FROM employees;

-- 중복 제거
SELECT DISTINCT department_id FROM employees;
```

### 계산 및 함수

```sql
-- 산술 연산
SELECT
    product_name,
    price,
    price * 0.9 AS discounted_price,
    price * quantity AS total_value
FROM products;

-- 문자열 함수
SELECT
    UPPER(first_name) AS upper_name,
    LOWER(last_name) AS lower_name,
    LENGTH(first_name) AS name_length,
    CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;

-- 날짜 함수
SELECT
    hire_date,
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    CURRENT_DATE - hire_date AS days_employed,
    AGE(hire_date) AS employment_duration
FROM employees;
```

### 조건부 표현식

```sql
-- CASE 문
SELECT
    employee_id,
    salary,
    CASE
        WHEN salary >= 10000 THEN 'High'
        WHEN salary >= 5000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_grade
FROM employees;

-- COALESCE (NULL 처리)
SELECT
    employee_id,
    COALESCE(commission_pct, 0) AS commission,
    COALESCE(phone_number, 'No Phone') AS contact
FROM employees;
```

## INSERT 문

### 기본 INSERT

```sql
-- 모든 컬럼에 값 삽입
INSERT INTO employees VALUES
(207, 'John', 'Doe', 'john.doe@email.com', '2023-01-15', 'IT_PROG', 6000, NULL, NULL, 60);

-- 특정 컬럼에만 값 삽입
INSERT INTO employees (employee_id, first_name, last_name, email, hire_date, job_id, salary, department_id)
VALUES (208, 'Jane', 'Smith', 'jane.smith@email.com', '2023-02-01', 'IT_PROG', 6500, 60);
```

### 다중 행 INSERT

```sql
-- MySQL, PostgreSQL 스타일
INSERT INTO products (product_id, product_name, price, category_id) VALUES
(101, 'Laptop', 1200.00, 1),
(102, 'Mouse', 25.00, 1),
(103, 'Keyboard', 75.00, 1);

-- 표준 SQL VALUES 구문
INSERT INTO products (product_id, product_name, price, category_id)
VALUES
    (104, 'Monitor', 300.00, 1),
    (105, 'Headphones', 150.00, 2);
```

### SELECT를 이용한 INSERT

```sql
-- 다른 테이블에서 데이터 복사
INSERT INTO employees_backup
SELECT * FROM employees WHERE department_id = 60;

-- 선택적 컬럼 복사
INSERT INTO high_salary_employees (employee_id, name, salary)
SELECT
    employee_id,
    CONCAT(first_name, ' ', last_name) AS name,
    salary
FROM employees
WHERE salary > 8000;
```

### INSERT 성능 최적화

```sql
-- 배치 INSERT (PostgreSQL)
INSERT INTO logs (timestamp, message, level)
SELECT * FROM UNNEST(
    ARRAY['2023-01-01 10:00:00', '2023-01-01 10:01:00'],
    ARRAY['System started', 'User login'],
    ARRAY['INFO', 'INFO']
) AS t(timestamp, message, level);

-- ON CONFLICT (PostgreSQL UPSERT)
INSERT INTO products (product_id, product_name, price)
VALUES (101, 'Updated Laptop', 1300.00)
ON CONFLICT (product_id)
DO UPDATE SET
    product_name = EXCLUDED.product_name,
    price = EXCLUDED.price;
```

## UPDATE 문

### 기본 UPDATE

```sql
-- 단일 컬럼 업데이트
UPDATE employees
SET salary = 7000
WHERE employee_id = 207;

-- 다중 컬럼 업데이트
UPDATE employees
SET
    salary = salary * 1.1,
    last_modified = CURRENT_TIMESTAMP
WHERE department_id = 60;
```

### 조건부 UPDATE

```sql
-- CASE를 이용한 조건부 업데이트
UPDATE employees
SET salary = CASE
    WHEN performance_rating = 'A' THEN salary * 1.15
    WHEN performance_rating = 'B' THEN salary * 1.10
    WHEN performance_rating = 'C' THEN salary * 1.05
    ELSE salary
END
WHERE department_id IN (10, 20, 30);

-- 서브쿼리를 이용한 업데이트
UPDATE employees
SET salary = (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.job_id = employees.job_id
)
WHERE salary < (
    SELECT AVG(salary) * 0.8
    FROM employees e3
    WHERE e3.job_id = employees.job_id
);
```

### JOIN을 이용한 UPDATE

```sql
-- PostgreSQL 스타일
UPDATE employees
SET department_name = d.department_name
FROM departments d
WHERE employees.department_id = d.department_id;

-- ANSI 표준 (일부 DBMS)
UPDATE employees
SET salary = employees.salary * (1 + d.budget_increase_rate)
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE d.budget_increase_rate > 0;
```

### 안전한 UPDATE 방법

```sql
-- 트랜잭션 사용
BEGIN;
UPDATE employees SET salary = salary * 1.1 WHERE department_id = 60;
-- 결과 확인 후
SELECT COUNT(*) FROM employees WHERE department_id = 60; -- 예상 개수 확인
COMMIT; -- 또는 ROLLBACK;

-- UPDATE 전 백업
CREATE TABLE employees_backup AS SELECT * FROM employees;
UPDATE employees SET salary = salary * 1.1 WHERE department_id = 60;
```

## DELETE 문

### 기본 DELETE

```sql
-- 조건부 삭제
DELETE FROM employees
WHERE employee_id = 207;

-- 다중 조건 삭제
DELETE FROM employees
WHERE department_id = 60
AND hire_date < '2020-01-01';
```

### 서브쿼리를 이용한 DELETE

```sql
-- 평균 급여 이하인 직원 삭제
DELETE FROM employees
WHERE salary < (SELECT AVG(salary) FROM employees);

-- 주의: 위 쿼리는 실행 중 평균이 변할 수 있으므로 아래와 같이 작성
DELETE FROM employees
WHERE salary < (
    SELECT avg_salary
    FROM (SELECT AVG(salary) AS avg_salary FROM employees) AS temp
);

-- 다른 테이블과 관련된 레코드 삭제
DELETE FROM order_items
WHERE order_id IN (
    SELECT order_id FROM orders
    WHERE order_date < CURRENT_DATE - INTERVAL '1 year'
);
```

### JOIN을 이용한 DELETE

```sql
-- PostgreSQL 스타일
DELETE FROM employees
USING departments
WHERE employees.department_id = departments.department_id
AND departments.department_name = 'Marketing';

-- MySQL 스타일
DELETE e FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE d.department_name = 'Marketing';
```

### 안전한 DELETE 방법

```sql
-- 1. SELECT로 먼저 확인
SELECT COUNT(*) FROM employees WHERE department_id = 60;

-- 2. 트랜잭션으로 감싸기
BEGIN;
DELETE FROM employees WHERE department_id = 60;
-- 결과 확인 후 COMMIT 또는 ROLLBACK

-- 3. 소프트 삭제 패턴
ALTER TABLE employees ADD COLUMN deleted_at TIMESTAMP;

-- 삭제 대신 마킹
UPDATE employees
SET deleted_at = CURRENT_TIMESTAMP
WHERE employee_id = 207;

-- 조회 시 삭제된 것 제외
SELECT * FROM employees WHERE deleted_at IS NULL;
```

## 고급 DML 기법

### 1. WITH 절 (CTE - Common Table Expression)

```sql
-- 재귀적이지 않은 CTE
WITH high_salary_employees AS (
    SELECT employee_id, first_name, last_name, salary, department_id
    FROM employees
    WHERE salary > 10000
),
department_stats AS (
    SELECT
        department_id,
        COUNT(*) as high_earner_count,
        AVG(salary) as avg_high_salary
    FROM high_salary_employees
    GROUP BY department_id
)
SELECT
    d.department_name,
    ds.high_earner_count,
    ds.avg_high_salary
FROM department_stats ds
JOIN departments d ON ds.department_id = d.department_id;

-- 재귀 CTE (조직도 예시)
WITH RECURSIVE org_chart AS (
    -- 기본 케이스: 최고 관리자
    SELECT employee_id, first_name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- 재귀 케이스
    SELECT e.employee_id, e.first_name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.employee_id
)
SELECT * FROM org_chart ORDER BY level, employee_id;
```

### 2. 윈도우 함수와 DML

```sql
-- 순위 기반 업데이트
WITH ranked_employees AS (
    SELECT
        employee_id,
        ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) as rank
    FROM employees
)
UPDATE employees
SET is_top_performer = true
WHERE employee_id IN (
    SELECT employee_id FROM ranked_employees WHERE rank <= 3
);

-- 누적 계산
SELECT
    employee_id,
    salary,
    SUM(salary) OVER (ORDER BY hire_date) as running_total,
    AVG(salary) OVER (PARTITION BY department_id) as dept_avg
FROM employees;
```

### 3. 대용량 데이터 처리

```sql
-- 배치 삭제 (성능 고려)
DELETE FROM logs
WHERE id IN (
    SELECT id FROM logs
    WHERE created_at < CURRENT_DATE - INTERVAL '30 days'
    LIMIT 1000
);

-- 파티션 기반 삭제 (PostgreSQL)
DELETE FROM sales_2022 WHERE sale_date < '2022-01-01'; -- 전체 파티션 삭제

-- 청크 단위 업데이트
DO $$
DECLARE
    batch_size INTEGER := 1000;
    affected_rows INTEGER;
BEGIN
    LOOP
        UPDATE employees
        SET updated_at = CURRENT_TIMESTAMP
        WHERE id IN (
            SELECT id FROM employees
            WHERE updated_at IS NULL
            LIMIT batch_size
        );

        GET DIAGNOSTICS affected_rows = ROW_COUNT;
        EXIT WHEN affected_rows = 0;

        COMMIT; -- 각 배치마다 커밋
    END LOOP;
END $$;
```

## 성능 최적화 팁

### 1. 인덱스 활용

```sql
-- 인덱스 생성
CREATE INDEX idx_employees_dept_salary ON employees(department_id, salary);
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- 인덱스를 활용한 쿼리
SELECT * FROM employees
WHERE department_id = 60 AND salary > 5000; -- 복합 인덱스 활용

-- 함수 기반 인덱스 (PostgreSQL, Oracle)
CREATE INDEX idx_employees_upper_lastname ON employees(UPPER(last_name));
SELECT * FROM employees WHERE UPPER(last_name) = 'SMITH';
```

### 2. 쿼리 최적화

```sql
-- EXISTS vs IN
-- 일반적으로 EXISTS가 더 효율적
SELECT * FROM employees e
WHERE EXISTS (
    SELECT 1 FROM departments d
    WHERE d.department_id = e.department_id
    AND d.location_id = 1700
);

-- 대신 IN 사용 시
SELECT * FROM employees
WHERE department_id IN (
    SELECT department_id FROM departments WHERE location_id = 1700
);
```

### 3. 실행 계획 분석

```sql
-- PostgreSQL
EXPLAIN ANALYZE SELECT * FROM employees WHERE salary > 10000;

-- MySQL
EXPLAIN FORMAT=JSON SELECT * FROM employees WHERE salary > 10000;

-- SQL Server
SET STATISTICS IO ON;
SELECT * FROM employees WHERE salary > 10000;
```

## 트랜잭션과 동시성

### 1. 기본 트랜잭션

```sql
BEGIN; -- 또는 START TRANSACTION;
INSERT INTO orders (customer_id, order_date) VALUES (1, CURRENT_DATE);
INSERT INTO order_items (order_id, product_id, quantity) VALUES (LAST_INSERT_ID(), 101, 2);
COMMIT;
```

### 2. 롤백과 세이브포인트

```sql
BEGIN;
INSERT INTO customers (name, email) VALUES ('John Doe', 'john@example.com');
SAVEPOINT sp1;

UPDATE customers SET email = 'john.doe@example.com' WHERE name = 'John Doe';
SAVEPOINT sp2;

-- 오류 발생 시
ROLLBACK TO sp1; -- sp2는 취소하고 sp1로 돌아감
-- 또는 전체 롤백
-- ROLLBACK;

COMMIT;
```

### 3. 락과 동시성 제어

```sql
-- 명시적 락 (PostgreSQL)
SELECT * FROM accounts WHERE account_id = 1 FOR UPDATE;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;

-- 공유 락
SELECT * FROM products WHERE category_id = 1 FOR SHARE;
```

## 결론

SQL의 기본 DML 명령들은 데이터베이스 작업의 핵심이며, 각각의 고급 기능과 최적화 기법을 이해하는 것이 효율적인 데이터베이스 프로그래밍의 기초입니다. 성능, 안전성, 유지보수성을 고려한 SQL 작성이 중요합니다.
