# 인덱스(Index)의 개념과 성능 영향

## 개요

인덱스는 데이터베이스에서 데이터 검색 속도를 향상시키기 위한 자료구조로, 테이블의 특정 컬럼에 대한 정렬된 포인터 집합입니다. 책의 색인과 같은 역할을 하며, 전체 테이블을 스캔하지 않고도 원하는 데이터를 빠르게 찾을 수 있게 합니다.

## 인덱스의 탄생 배경

- **선형 검색의 한계**: 데이터가 증가할수록 검색 시간이 비례적으로 증가 (O(n))
- **정렬된 데이터의 활용**: 정렬된 데이터에서는 이진 검색으로 O(log n) 성능 달성 가능
- **저장 공간과 검색 속도의 트레이드오프**: 추가 공간을 사용하여 검색 성능 향상
- **B-Tree 자료구조의 도입**: 디스크 기반 저장소에 최적화된 균형 트리 구조

## 인덱스의 기본 원리

### 물리적 구조

```sql
-- 테이블 데이터 (힙 구조)
Table: employees
Row ID | employee_id | name      | department_id | salary
1      | 107        | Johnson   | 60           | 9000
2      | 103        | Smith     | 20           | 8000
3      | 101        | Adams     | 10           | 5000
4      | 105        | Wilson    | 30           | 7000

-- 인덱스 구조 (employee_id 컬럼 기준)
Index: idx_employees_id
Key Value | Row ID
101      | 3
103      | 2
105      | 4
107      | 1
```

### 검색 과정

1. **인덱스 검색**: 인덱스에서 원하는 키 값을 찾음
2. **Row ID 획득**: 해당 키에 대응하는 Row ID를 얻음
3. **데이터 페이지 접근**: Row ID를 이용하여 실제 데이터에 접근

## 인덱스 유형

### 1. 클러스터드 인덱스 (Clustered Index)

#### 특징

- 테이블 데이터 자체가 인덱스 키 순서로 물리적으로 정렬됨
- 테이블당 하나만 존재 가능
- 일반적으로 기본 키에 자동 생성

```sql
-- SQL Server에서 클러스터드 인덱스
CREATE CLUSTERED INDEX idx_employees_id
ON employees(employee_id);

-- 데이터 물리적 정렬 상태
Row ID | employee_id | name      | department_id | salary
3      | 101        | Adams     | 10           | 5000
2      | 103        | Smith     | 20           | 8000
4      | 105        | Wilson    | 30           | 7000
1      | 107        | Johnson   | 60           | 9000
```

#### 장점과 단점

```sql
-- 장점: 범위 검색에 매우 효율적
SELECT * FROM employees
WHERE employee_id BETWEEN 102 AND 106;
-- → 연속된 데이터 페이지만 읽으면 됨

-- 단점: INSERT 시 데이터 재정렬 오버헤드
INSERT INTO employees VALUES (104, 'Brown', 40, 6000);
-- → 기존 데이터들이 물리적으로 이동해야 할 수 있음
```

### 2. 비클러스터드 인덱스 (Non-Clustered Index)

#### 특징

- 인덱스와 테이블 데이터가 별도로 저장
- 테이블당 여러 개 생성 가능
- 대부분의 보조 인덱스가 이 유형

```sql
-- 비클러스터드 인덱스 생성
CREATE INDEX idx_employees_dept ON employees(department_id);
CREATE INDEX idx_employees_salary ON employees(salary);

-- 인덱스 구조 (department_id 기준)
Index: idx_employees_dept
Key Value | Row Pointer
10       | Page:100, Slot:2
20       | Page:100, Slot:1
30       | Page:101, Slot:3
60       | Page:101, Slot:1
```

### 3. 복합 인덱스 (Composite Index)

#### 기본 개념

여러 컬럼을 조합한 인덱스로, 컬럼 순서가 매우 중요합니다.

```sql
-- 복합 인덱스 생성
CREATE INDEX idx_employees_dept_salary
ON employees(department_id, salary);

-- 인덱스 구조
Key Value (dept_id, salary) | Row Pointer
(10, 5000)                 | Page:100, Slot:2
(20, 8000)                 | Page:100, Slot:1
(30, 7000)                 | Page:101, Slot:3
(60, 9000)                 | Page:101, Slot:1
```

#### 활용 패턴

```sql
-- 효율적 사용 (Leading Column 사용)
SELECT * FROM employees WHERE department_id = 20;
SELECT * FROM employees WHERE department_id = 20 AND salary > 7000;

-- 비효율적 사용 (Non-Leading Column만 사용)
SELECT * FROM employees WHERE salary > 7000;
-- → 인덱스 전체를 스캔해야 함
```

### 4. 유니크 인덱스 (Unique Index)

#### 특징과 용도

```sql
-- 유니크 인덱스 생성
CREATE UNIQUE INDEX idx_employees_email
ON employees(email);

-- 자동으로 유니크 제약조건 강제
INSERT INTO employees VALUES (108, 'Davis', 'john@email.com', 40, 6500);
INSERT INTO employees VALUES (109, 'Miller', 'john@email.com', 50, 7000);
-- Error: Duplicate key value violates unique constraint

-- 복합 유니크 인덱스
CREATE UNIQUE INDEX idx_employees_name_dept
ON employees(first_name, last_name, department_id);
```

### 5. 부분 인덱스 (Partial Index)

#### 조건부 인덱스 (PostgreSQL, SQLite)

```sql
-- 활성 직원만 인덱싱
CREATE INDEX idx_active_employees
ON employees(department_id)
WHERE status = 'ACTIVE';

-- 고액 연봉자만 인덱싱
CREATE INDEX idx_high_salary_employees
ON employees(salary)
WHERE salary > 10000;

-- 최근 데이터만 인덱싱
CREATE INDEX idx_recent_orders
ON orders(customer_id, order_date)
WHERE order_date >= CURRENT_DATE - INTERVAL '1 year';
```

### 6. 함수 기반 인덱스 (Function-based Index)

#### 계산된 값에 대한 인덱싱

```sql
-- PostgreSQL, Oracle
CREATE INDEX idx_employees_upper_name
ON employees(UPPER(last_name));

-- 사용 예시
SELECT * FROM employees WHERE UPPER(last_name) = 'SMITH';

-- 표현식 인덱스
CREATE INDEX idx_employees_full_name
ON employees((first_name || ' ' || last_name));

-- JSON 속성 인덱스 (PostgreSQL)
CREATE INDEX idx_employee_skills
ON employees USING GIN ((data->'skills'));
```

## B-Tree 인덱스 구조

### B-Tree의 특징

- **균형 트리**: 모든 리프 노드가 같은 레벨에 위치
- **높은 분기도**: 노드당 많은 키를 저장하여 트리 높이 최소화
- **순차 접근 지원**: 리프 노드들이 연결되어 범위 검색에 효율적

### B-Tree 구조 예시

```
                [50|100]
               /    |    \
         [20|30]  [70|80] [120|150]
        /  |  \   / |  \   /   |   \
    [10] [25] [35][60][75][90][110][140][180]
```

### 검색 과정

```sql
-- employee_id = 75 검색
-- 1. Root: 75 > 50, 75 < 100 → 중간 자식으로 이동
-- 2. Internal: 75 > 70, 75 < 80 → 중간 자식으로 이동
-- 3. Leaf: 75 찾음 → Row Pointer 반환
```

### B+Tree vs B-Tree

```sql
-- B+Tree 특징 (대부분의 DBMS에서 사용)
-- 1. 모든 데이터가 리프 노드에만 저장
-- 2. 리프 노드들이 연결 리스트 형태
-- 3. 범위 검색에 더 효율적

-- 범위 검색 예시
SELECT * FROM employees WHERE employee_id BETWEEN 70 AND 90;
-- B+Tree에서는 첫 번째 값(70)만 찾은 후 연결된 리프를 순차 탐색
```

## 해시 인덱스

### 특징과 한계

```sql
-- PostgreSQL 해시 인덱스
CREATE INDEX idx_employees_hash_id
ON employees USING HASH(employee_id);

-- 장점: 등가 검색에서 O(1) 성능
SELECT * FROM employees WHERE employee_id = 105;

-- 단점: 범위 검색 불가
SELECT * FROM employees WHERE employee_id > 100;
-- → 해시 인덱스 사용 불가, 전체 테이블 스캔
```

### 해시 충돌 처리

```sql
-- 해시 함수: hash(key) % bucket_count
-- 충돌 발생 시 체이닝 또는 오픈 어드레싱 사용

-- 메모리 내 해시 테이블 (Redis 스타일)
Hash Bucket 0: [key:101, value:ptr1] → [key:201, value:ptr2]
Hash Bucket 1: [key:102, value:ptr3]
Hash Bucket 2: [key:103, value:ptr4] → [key:203, value:ptr5]
```

## 비트맵 인덱스

### 특징 (Oracle, PostgreSQL)

```sql
-- 카디널리티가 낮은 컬럼에 효율적
CREATE INDEX idx_employees_gender ON employees(gender);

-- 비트맵 구조
Row ID | Gender | Bitmap_M | Bitmap_F
1      | M      | 1        | 0
2      | F      | 0        | 1
3      | M      | 1        | 0
4      | F      | 0        | 1
```

### 비트맵 연산

```sql
-- AND 연산
SELECT * FROM employees WHERE gender = 'F' AND department_id = 20;
-- Bitmap_F AND Bitmap_Dept20 = 결과 비트맵

-- OR 연산
SELECT * FROM employees WHERE gender = 'F' OR department_id = 20;
-- Bitmap_F OR Bitmap_Dept20 = 결과 비트맵
```

## 인덱스 성능 영향

### 1. SELECT 성능 향상

#### Before/After 비교

```sql
-- 인덱스 없는 경우
SELECT * FROM employees WHERE department_id = 60;
-- Execution Plan: Table Scan (Full Scan)
-- Cost: O(n) - 전체 테이블 스캔

-- 인덱스 있는 경우
CREATE INDEX idx_employees_dept ON employees(department_id);
SELECT * FROM employees WHERE department_id = 60;
-- Execution Plan: Index Seek → Key Lookup
-- Cost: O(log n) + 적은 수의 페이지 접근
```

#### 통계 정보 예시

```sql
-- PostgreSQL 통계
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM employees WHERE department_id = 60;

-- 인덱스 없을 때:
-- Seq Scan on employees (cost=0.00..15.00 rows=6 width=68)
--   (actual time=0.10..0.15 rows=6 loops=1)
-- Buffers: shared hit=8

-- 인덱스 있을 때:
-- Index Scan using idx_employees_dept (cost=0.28..8.30 rows=6 width=68)
--   (actual time=0.02..0.04 rows=6 loops=1)
-- Buffers: shared hit=3
```

### 2. DML 성능 영향

#### INSERT 성능 저하

```sql
-- 인덱스가 많을수록 INSERT 느려짐
CREATE TABLE test_table (
    id SERIAL PRIMARY KEY,
    col1 INT,
    col2 INT,
    col3 INT
);

-- 인덱스 추가시마다 INSERT 시간 증가
CREATE INDEX idx1 ON test_table(col1);  -- INSERT 시간 +10%
CREATE INDEX idx2 ON test_table(col2);  -- INSERT 시간 +20%
CREATE INDEX idx3 ON test_table(col3);  -- INSERT 시간 +30%

-- 원인: 각 INSERT마다 모든 인덱스 업데이트 필요
```

#### UPDATE/DELETE 성능

```sql
-- 인덱스 컬럼 UPDATE 시 인덱스 재구성 필요
UPDATE employees SET salary = 12000 WHERE employee_id = 101;
-- 1. salary 인덱스에서 기존 값 제거
-- 2. salary 인덱스에 새 값 추가
-- 3. 필요시 인덱스 페이지 분할/병합

-- 비인덱스 컬럼 UPDATE는 영향 적음
UPDATE employees SET phone_number = '555-1234' WHERE employee_id = 101;
-- → salary 인덱스는 변경되지 않음
```

### 3. 인덱스 선택도와 효율성

#### 선택도 (Selectivity) 계산

```sql
-- 선택도 = 고유한 값의 수 / 전체 행의 수
-- 예시: employees 테이블 1000행

-- 높은 선택도 (좋음)
SELECT COUNT(DISTINCT employee_id) / COUNT(*) FROM employees;
-- Result: 1000/1000 = 1.0 (100% 고유)

-- 중간 선택도
SELECT COUNT(DISTINCT department_id) / COUNT(*) FROM employees;
-- Result: 10/1000 = 0.01 (1% 고유)

-- 낮은 선택도 (나쁨)
SELECT COUNT(DISTINCT gender) / COUNT(*) FROM employees;
-- Result: 2/1000 = 0.002 (0.2% 고유)
```

#### 인덱스 효율성 기준

```sql
-- 효율적인 인덱스 사용
SELECT * FROM employees WHERE employee_id = 101;
-- → 1개 행 반환 예상, 높은 선택도

-- 비효율적인 인덱스 사용
SELECT * FROM employees WHERE gender = 'M';
-- → 500개 행 반환 예상, 낮은 선택도
-- 옵티마이저가 전체 테이블 스캔 선택할 수 있음
```

## 인덱스 설계 전략

### 1. 컬럼 선택 기준

```sql
-- 1. WHERE 절에 자주 사용되는 컬럼
CREATE INDEX idx_orders_customer ON orders(customer_id);
SELECT * FROM orders WHERE customer_id = 12345;

-- 2. JOIN 조건에 사용되는 컬럼
CREATE INDEX idx_order_details_order ON order_details(order_id);
SELECT * FROM orders o JOIN order_details od ON o.order_id = od.order_id;

-- 3. ORDER BY에 사용되는 컬럼
CREATE INDEX idx_employees_salary_desc ON employees(salary DESC);
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;

-- 4. GROUP BY에 사용되는 컬럼
CREATE INDEX idx_sales_date_product ON sales(sale_date, product_id);
SELECT product_id, SUM(amount) FROM sales GROUP BY product_id;
```

### 2. 복합 인덱스 컬럼 순서

```sql
-- 원칙: 선택도가 높은 컬럼을 앞에 배치
-- 예외: 쿼리 패턴을 고려하여 조정

-- Case 1: 개별 검색이 많은 경우
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
-- customer_id로 검색 → 효율적
-- customer_id + order_date로 검색 → 매우 효율적
-- order_date만으로 검색 → 비효율적

-- Case 2: 범위 검색을 고려한 순서
CREATE INDEX idx_sales_date_amount ON sales(sale_date, amount);
SELECT * FROM sales
WHERE sale_date BETWEEN '2023-01-01' AND '2023-12-31'
AND amount > 1000;
```

### 3. 인덱스 커버링

```sql
-- 커버링 인덱스: 쿼리에 필요한 모든 컬럼을 인덱스에 포함
CREATE INDEX idx_employees_covering
ON employees(department_id, salary, first_name, last_name);

-- 커버링 되는 쿼리 (Table Lookup 불필요)
SELECT first_name, last_name, salary
FROM employees
WHERE department_id = 60;

-- INCLUDE 컬럼 (SQL Server, PostgreSQL)
CREATE INDEX idx_employees_dept_include
ON employees(department_id) INCLUDE (salary, first_name, last_name);
```

## 인덱스 유지보수

### 1. 인덱스 통계 정보 업데이트

```sql
-- PostgreSQL
ANALYZE employees;
REINDEX INDEX idx_employees_dept;

-- SQL Server
UPDATE STATISTICS employees;
ALTER INDEX idx_employees_dept ON employees REBUILD;

-- MySQL
ANALYZE TABLE employees;
```

### 2. 인덱스 단편화 해결

```sql
-- 단편화 확인 (SQL Server)
SELECT
    object_name(i.object_id) as table_name,
    i.name as index_name,
    ips.avg_fragmentation_in_percent,
    ips.page_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'DETAILED') ips
JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id;

-- 단편화 해결
-- 10% 미만: 유지
-- 10-30%: REORGANIZE
-- 30% 이상: REBUILD
ALTER INDEX idx_employees_dept ON employees REORGANIZE;
ALTER INDEX idx_employees_dept ON employees REBUILD;
```

### 3. 사용하지 않는 인덱스 식별

```sql
-- PostgreSQL: 인덱스 사용 통계
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY tablename, indexname;

-- SQL Server: 누락된 인덱스 제안
SELECT
    mid.statement,
    mids.avg_total_user_cost,
    mids.avg_user_impact,
    mid.equality_columns,
    mid.inequality_columns,
    mid.included_columns
FROM sys.dm_db_missing_index_details mid
JOIN sys.dm_db_missing_index_group_stats mids
    ON mid.index_handle = mids.group_handle
ORDER BY mids.avg_total_user_cost * mids.avg_user_impact DESC;
```

## 특수한 인덱스 기법

### 1. 파티셔닝과 인덱스

```sql
-- 파티션별 로컬 인덱스
CREATE TABLE sales_partitioned (
    sale_id SERIAL,
    sale_date DATE,
    customer_id INT,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (sale_date);

-- 각 파티션에 자동으로 인덱스 생성
CREATE INDEX idx_sales_customer ON sales_partitioned(customer_id);
```

### 2. 병렬 인덱스 생성

```sql
-- PostgreSQL
CREATE INDEX CONCURRENTLY idx_large_table_col ON large_table(column_name);
-- 테이블 락 없이 백그라운드에서 인덱스 생성

-- Oracle
CREATE INDEX idx_large_table_col ON large_table(column_name) PARALLEL 4;
-- 4개 프로세스로 병렬 생성
```

## 인덱스 모니터링과 튜닝

### 1. 인덱스 효율성 측정

```sql
-- 인덱스 사용률 모니터링
SELECT
    t.table_name,
    i.index_name,
    s.num_rows,
    s.distinct_keys,
    s.clustering_factor,
    s.blevel  -- B-Tree 높이
FROM user_indexes i
JOIN user_tables t ON i.table_name = t.table_name
JOIN user_ind_statistics s ON i.index_name = s.index_name;
```

### 2. 실행 계획 분석

```sql
-- 인덱스 사용 여부 확인
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT * FROM employees WHERE department_id = 60 AND salary > 8000;

-- 주요 확인 포인트:
-- 1. Index Scan vs Seq Scan
-- 2. Rows Removed by Filter (인덱스 효율성)
-- 3. Buffers Hit vs Read (캐시 효율성)
-- 4. 실제 실행 시간
```

## 결론

인덱스는 데이터베이스 성능 최적화의 핵심 도구이지만, 잘못 설계하면 오히려 성능 저하를 초래할 수 있습니다. 쿼리 패턴 분석, 적절한 컬럼 선택, 정기적인 유지보수를 통해 효과적인 인덱스 전략을 수립해야 합니다. 현대 데이터베이스의 다양한 인덱스 유형과 최적화 기법을 이해하고 활용하는 것이 고성능 데이터베이스 시스템 구축의 기초입니다.
