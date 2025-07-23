# SQL 절 (WHERE, ORDER BY, GROUP BY, HAVING)

## 개요

SQL 절(Clauses)들은 SELECT 문에서 데이터를 필터링, 정렬, 그룹화하는 핵심 구성 요소입니다. 각 절의 실행 순서와 특성을 이해하는 것이 효율적인 쿼리 작성의 기초입니다.

## SQL 문의 논리적 처리 순서

```sql
-- 작성 순서
SELECT column_list
FROM table_name
WHERE condition
GROUP BY column_list
HAVING condition
ORDER BY column_list

-- 실제 처리 순서
-- 1. FROM - 테이블 지정
-- 2. WHERE - 행 필터링
-- 3. GROUP BY - 그룹화
-- 4. HAVING - 그룹 필터링
-- 5. SELECT - 컬럼 선택/계산
-- 6. ORDER BY - 정렬
```

## WHERE 절

### 기본 비교 연산자

```sql
-- 동등 비교
SELECT * FROM employees WHERE department_id = 60;
SELECT * FROM employees WHERE job_id = 'IT_PROG';

-- 부등호 비교
SELECT * FROM employees WHERE salary > 10000;
SELECT * FROM employees WHERE salary <= 5000;
SELECT * FROM employees WHERE hire_date >= '2020-01-01';

-- NULL 비교 (= NULL이 아닌 IS NULL 사용)
SELECT * FROM employees WHERE commission_pct IS NULL;
SELECT * FROM employees WHERE phone_number IS NOT NULL;
```

### 논리 연산자

```sql
-- AND 연산자
SELECT * FROM employees
WHERE department_id = 60 AND salary > 8000;

-- OR 연산자
SELECT * FROM employees
WHERE department_id = 60 OR department_id = 90;

-- NOT 연산자
SELECT * FROM employees
WHERE NOT (department_id = 60 OR department_id = 90);

-- 연산자 우선순위 (괄호로 명시하는 것이 좋음)
SELECT * FROM employees
WHERE (department_id = 60 OR department_id = 90)
AND salary > 8000;
```

### 범위 및 패턴 조건

```sql
-- BETWEEN (포함)
SELECT * FROM employees
WHERE salary BETWEEN 5000 AND 10000;

-- IN 연산자
SELECT * FROM employees
WHERE department_id IN (10, 20, 30);

-- LIKE 패턴 매칭
SELECT * FROM employees WHERE first_name LIKE 'J%';      -- J로 시작
SELECT * FROM employees WHERE first_name LIKE '%son';    -- son으로 끝남
SELECT * FROM employees WHERE first_name LIKE '%ar%';    -- ar 포함
SELECT * FROM employees WHERE first_name LIKE 'J__n';    -- J_n (4글자)

-- 대소문자 구분하지 않는 검색 (PostgreSQL)
SELECT * FROM employees WHERE first_name ILIKE 'j%';

-- 정규표현식 (PostgreSQL)
SELECT * FROM employees WHERE email ~ '^[a-zA-Z0-9]+@[a-zA-Z0-9]+\.(com|org)$';
```

### 날짜 조건

```sql
-- 날짜 범위
SELECT * FROM employees
WHERE hire_date BETWEEN '2020-01-01' AND '2020-12-31';

-- 날짜 함수 활용
SELECT * FROM employees
WHERE EXTRACT(YEAR FROM hire_date) = 2020;

SELECT * FROM employees
WHERE hire_date >= CURRENT_DATE - INTERVAL '1 year';

-- 특정 월/일 조건
SELECT * FROM employees
WHERE EXTRACT(MONTH FROM hire_date) = 12; -- 12월 입사자

SELECT * FROM employees
WHERE EXTRACT(DOW FROM hire_date) = 1; -- 월요일 입사자 (PostgreSQL)
```

### 서브쿼리 조건

```sql
-- 단일 값 서브쿼리
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- 다중 값 서브쿼리
SELECT * FROM employees
WHERE department_id IN (
    SELECT department_id FROM departments
    WHERE location_id = 1700
);

-- EXISTS 조건
SELECT * FROM employees e
WHERE EXISTS (
    SELECT 1 FROM departments d
    WHERE d.department_id = e.department_id
    AND d.location_id = 1700
);

-- ANY/ALL 조건
SELECT * FROM employees
WHERE salary > ANY (
    SELECT salary FROM employees WHERE department_id = 60
);

SELECT * FROM employees
WHERE salary > ALL (
    SELECT salary FROM employees WHERE department_id = 60
);
```

## ORDER BY 절

### 기본 정렬

```sql
-- 오름차순 정렬 (기본값)
SELECT * FROM employees ORDER BY salary;
SELECT * FROM employees ORDER BY salary ASC;

-- 내림차순 정렬
SELECT * FROM employees ORDER BY salary DESC;

-- 다중 컬럼 정렬
SELECT * FROM employees
ORDER BY department_id ASC, salary DESC;
```

### 표현식으로 정렬

```sql
-- 계산 결과로 정렬
SELECT first_name, last_name, salary, salary * 12 AS annual_salary
FROM employees
ORDER BY salary * 12 DESC;

-- 함수 결과로 정렬
SELECT * FROM employees
ORDER BY LENGTH(first_name), first_name;

-- CASE문으로 사용자 정의 정렬
SELECT * FROM employees
ORDER BY
    CASE department_id
        WHEN 90 THEN 1  -- 경영진이 먼저
        WHEN 60 THEN 2  -- IT 부서가 다음
        ELSE 3          -- 나머지 부서
    END,
    salary DESC;
```

### NULL 값 처리

```sql
-- PostgreSQL, Oracle: NULLS FIRST/LAST
SELECT * FROM employees
ORDER BY commission_pct NULLS LAST;

-- MySQL: NULL 값 처리
SELECT * FROM employees
ORDER BY commission_pct IS NULL, commission_pct;

-- 일반적인 방법 (모든 DB)
SELECT * FROM employees
ORDER BY COALESCE(commission_pct, -1);
```

### 성능 최적화

```sql
-- 인덱스를 활용한 정렬
CREATE INDEX idx_employees_dept_salary ON employees(department_id, salary);

-- ORDER BY절과 일치하는 인덱스 사용
SELECT * FROM employees
WHERE department_id = 60
ORDER BY salary; -- idx_employees_dept_salary 활용

-- LIMIT와 함께 사용 (페이징)
SELECT * FROM employees
ORDER BY employee_id
LIMIT 10 OFFSET 20; -- 21-30번째 레코드
```

## GROUP BY 절

### 기본 그룹화

```sql
-- 단일 컬럼 그룹화
SELECT department_id, COUNT(*) as employee_count
FROM employees
GROUP BY department_id;

-- 다중 컬럼 그룹화
SELECT department_id, job_id, COUNT(*) as count, AVG(salary) as avg_salary
FROM employees
GROUP BY department_id, job_id
ORDER BY department_id, job_id;
```

### 집계 함수

```sql
-- 기본 집계 함수
SELECT
    department_id,
    COUNT(*) as total_employees,
    COUNT(commission_pct) as commissioned_employees, -- NULL 제외
    SUM(salary) as total_salary,
    AVG(salary) as avg_salary,
    MIN(salary) as min_salary,
    MAX(salary) as max_salary,
    STDDEV(salary) as salary_stddev  -- 표준편차
FROM employees
GROUP BY department_id;

-- DISTINCT와 함께 사용
SELECT
    department_id,
    COUNT(DISTINCT job_id) as unique_jobs
FROM employees
GROUP BY department_id;
```

### 조건부 집계

```sql
-- CASE문을 이용한 조건부 집계
SELECT
    department_id,
    COUNT(*) as total_count,
    COUNT(CASE WHEN salary > 10000 THEN 1 END) as high_salary_count,
    COUNT(CASE WHEN hire_date >= '2020-01-01' THEN 1 END) as recent_hires,
    AVG(CASE WHEN job_id LIKE '%MANAGER%' THEN salary END) as avg_manager_salary
FROM employees
GROUP BY department_id;

-- FILTER 절 (PostgreSQL)
SELECT
    department_id,
    COUNT(*) as total_count,
    COUNT(*) FILTER (WHERE salary > 10000) as high_salary_count,
    AVG(salary) FILTER (WHERE job_id LIKE '%MANAGER%') as avg_manager_salary
FROM employees
GROUP BY department_id;
```

### 고급 그룹화

#### ROLLUP

```sql
-- 부분합 계산
SELECT
    department_id,
    job_id,
    COUNT(*) as employee_count,
    SUM(salary) as total_salary
FROM employees
GROUP BY ROLLUP(department_id, job_id)
ORDER BY department_id, job_id;

-- 결과에는 다음이 포함됨:
-- 1. (department_id, job_id) 조합별 집계
-- 2. department_id별 집계 (job_id는 NULL)
-- 3. 전체 집계 (둘 다 NULL)
```

#### CUBE

```sql
-- 모든 가능한 조합의 집계
SELECT
    department_id,
    job_id,
    COUNT(*) as employee_count
FROM employees
GROUP BY CUBE(department_id, job_id);

-- 결과에는 ROLLUP + 추가로 다음이 포함됨:
-- job_id별 집계 (department_id는 NULL)
```

#### GROUPING SETS

```sql
-- 특정 그룹화 집합만 지정
SELECT
    department_id,
    job_id,
    COUNT(*) as employee_count
FROM employees
GROUP BY GROUPING SETS (
    (department_id),
    (job_id),
    (department_id, job_id),
    () -- 전체 집계
);
```

### GROUP BY 제한사항

```sql
-- 오류: GROUP BY에 없는 컬럼은 SELECT할 수 없음
-- SELECT department_id, job_id, first_name, COUNT(*)
-- FROM employees GROUP BY department_id;

-- 올바른 방법 1: GROUP BY에 모든 컬럼 포함
SELECT department_id, job_id, first_name, COUNT(*)
FROM employees
GROUP BY department_id, job_id, first_name;

-- 올바른 방법 2: 집계함수 사용
SELECT department_id, job_id, STRING_AGG(first_name, ', ') as names, COUNT(*)
FROM employees
GROUP BY department_id, job_id;
```

## HAVING 절

### 기본 HAVING

```sql
-- 그룹 집계 결과에 대한 조건
SELECT department_id, COUNT(*) as employee_count, AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;  -- 직원 수가 5명보다 많은 부서만

-- 복합 조건
SELECT department_id, COUNT(*) as employee_count, AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5 AND AVG(salary) > 8000;
```

### WHERE vs HAVING

```sql
-- WHERE: 그룹화 전 행 필터링
-- HAVING: 그룹화 후 그룹 필터링

-- 비효율적: 모든 행을 그룹화한 후 필터링
SELECT department_id, COUNT(*) as employee_count
FROM employees
GROUP BY department_id
HAVING department_id IN (10, 20, 30);

-- 효율적: 먼저 행을 필터링한 후 그룹화
SELECT department_id, COUNT(*) as employee_count
FROM employees
WHERE department_id IN (10, 20, 30)
GROUP BY department_id;

-- 올바른 HAVING 사용
SELECT department_id, COUNT(*) as employee_count, AVG(salary) as avg_salary
FROM employees
WHERE hire_date >= '2020-01-01'  -- WHERE로 먼저 필터링
GROUP BY department_id
HAVING COUNT(*) >= 3;            -- HAVING으로 그룹 필터링
```

### 복잡한 HAVING 조건

```sql
-- 서브쿼리 사용
SELECT department_id, AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > (
    SELECT AVG(salary) FROM employees
);

-- 여러 집계 함수 조건
SELECT
    department_id,
    COUNT(*) as employee_count,
    AVG(salary) as avg_salary,
    MAX(salary) - MIN(salary) as salary_range
FROM employees
GROUP BY department_id
HAVING COUNT(*) >= 3
   AND AVG(salary) > 8000
   AND MAX(salary) - MIN(salary) > 5000;
```

## 절들의 조합과 최적화

### 복합 쿼리 예시

```sql
-- 모든 절을 사용한 복합 쿼리
SELECT
    d.department_name,
    e.job_id,
    COUNT(*) as employee_count,
    AVG(e.salary) as avg_salary,
    MIN(e.hire_date) as earliest_hire,
    MAX(e.hire_date) as latest_hire
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.salary IS NOT NULL
  AND e.hire_date >= '2015-01-01'
GROUP BY d.department_name, e.job_id
HAVING COUNT(*) >= 2
   AND AVG(e.salary) > 7000
ORDER BY d.department_name, avg_salary DESC;
```

### 성능 최적화 팁

```sql
-- 1. 적절한 인덱스 생성
CREATE INDEX idx_employees_complex
ON employees(hire_date, department_id, job_id, salary);

-- 2. WHERE 조건 최적화 (선택도가 높은 조건을 앞에)
SELECT * FROM employees
WHERE employee_id = 100        -- 매우 선택적
  AND department_id = 60       -- 보통 선택적
  AND salary > 5000;           -- 덜 선택적

-- 3. EXISTS vs IN 최적화
-- EXISTS가 일반적으로 더 효율적
SELECT * FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e
    WHERE e.department_id = d.department_id
    AND e.salary > 10000
);
```

### 윈도우 함수와의 비교

```sql
-- GROUP BY 방식
SELECT department_id, AVG(salary) as dept_avg
FROM employees
GROUP BY department_id;

-- 윈도우 함수 방식 (행별 정보 유지)
SELECT
    employee_id,
    first_name,
    department_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) as dept_avg
FROM employees;

-- 랭킹과 함께
SELECT
    employee_id,
    first_name,
    department_id,
    salary,
    RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) as salary_rank
FROM employees
WHERE department_id IS NOT NULL
ORDER BY department_id, salary_rank;
```

## 실행 계획 분석

```sql
-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS)
SELECT department_id, COUNT(*), AVG(salary)
FROM employees
WHERE hire_date >= '2020-01-01'
GROUP BY department_id
HAVING COUNT(*) >= 3
ORDER BY AVG(salary) DESC;

-- 성능 모니터링 포인트:
-- 1. Sequential Scan vs Index Scan
-- 2. Hash Aggregate vs Group Aggregate
-- 3. Sort 연산의 메모리 사용량
-- 4. 전체 실행 시간과 I/O 비용
```

## 결론

WHERE, ORDER BY, GROUP BY, HAVING 절은 SQL 쿼리의 핵심 구성 요소로, 각각의 실행 순서와 특성을 이해하여 효율적인 쿼리를 작성해야 합니다. 특히 WHERE와 HAVING의 차이점, 인덱스 활용 방법, 그리고 성능 최적화 기법을 숙지하는 것이 중요합니다.
