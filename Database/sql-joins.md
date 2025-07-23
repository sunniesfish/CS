# SQL 조인 (INNER, OUTER, SELF, CROSS JOIN)

## 개요

조인(JOIN)은 관계형 데이터베이스에서 둘 이상의 테이블을 연결하여 데이터를 조회하는 핵심 기능입니다. 정규화된 데이터베이스에서 분산된 정보를 통합하여 의미있는 결과를 얻기 위해 필수적입니다.

## 조인의 역사와 이론적 배경

- Codd의 관계형 모델에서 관계 대수의 조인 연산에서 유래
- 카테시안 곱(Cartesian Product)을 기반으로 한 필터링 연산
- 함수적 종속성과 정규화 이론과 밀접한 연관

## 기본 테이블 구조 (예시)

```sql
-- 직원 테이블
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    department_id INT,
    manager_id INT,
    salary DECIMAL(10,2)
);

-- 부서 테이블
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100),
    location_id INT
);

-- 위치 테이블
CREATE TABLE locations (
    location_id INT PRIMARY KEY,
    city VARCHAR(50),
    country VARCHAR(50)
);
```

## INNER JOIN

### 기본 개념

INNER JOIN은 두 테이블에서 조인 조건을 만족하는 행들만 반환합니다. 가장 일반적으로 사용되는 조인 유형입니다.

### 기본 문법

```sql
-- ANSI 표준 문법
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;

-- 암시적 조인 (구식 문법, 권장하지 않음)
SELECT columns
FROM table1, table2
WHERE table1.column = table2.column;
```

### 실제 예시

```sql
-- 직원과 부서 정보 조인
SELECT
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- 다중 테이블 조인
SELECT
    e.first_name,
    e.last_name,
    d.department_name,
    l.city
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN locations l ON d.location_id = l.location_id;
```

### 조건이 있는 INNER JOIN

```sql
-- WHERE 절과 함께
SELECT
    e.first_name,
    e.last_name,
    d.department_name,
    e.salary
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 10000
AND d.department_name = 'IT';

-- 복합 조인 조건
SELECT *
FROM order_details od
INNER JOIN products p ON od.product_id = p.product_id
INNER JOIN orders o ON od.order_id = o.order_id AND o.order_date >= '2023-01-01';
```

### INNER JOIN 성능 최적화

```sql
-- 적절한 인덱스 생성
CREATE INDEX idx_employees_dept ON employees(department_id);
CREATE INDEX idx_departments_location ON departments(location_id);

-- 조인 순서 고려 (옵티마이저가 보통 자동으로 처리)
SELECT /*+ USE_INDEX(e, idx_employees_dept) */
    e.first_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;
```

## OUTER JOIN

### LEFT OUTER JOIN (LEFT JOIN)

왼쪽 테이블의 모든 행을 반환하고, 오른쪽 테이블에서 매칭되는 행이 없으면 NULL로 채웁니다.

```sql
-- 모든 직원과 그들의 부서 정보 (부서가 없는 직원도 포함)
SELECT
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- 부서가 할당되지 않은 직원 찾기
SELECT
    e.employee_id,
    e.first_name,
    e.last_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE d.department_id IS NULL;
```

### RIGHT OUTER JOIN (RIGHT JOIN)

오른쪽 테이블의 모든 행을 반환하고, 왼쪽 테이블에서 매칭되는 행이 없으면 NULL로 채웁니다.

```sql
-- 모든 부서와 해당 부서의 직원 정보 (직원이 없는 부서도 포함)
SELECT
    d.department_id,
    d.department_name,
    e.first_name,
    e.last_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- 직원이 없는 부서 찾기
SELECT
    d.department_id,
    d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id
WHERE e.employee_id IS NULL;
```

### FULL OUTER JOIN

양쪽 테이블의 모든 행을 반환합니다. 매칭되지 않는 행은 NULL로 채웁니다.

```sql
-- PostgreSQL, SQL Server, Oracle
SELECT
    COALESCE(e.employee_id, 0) as employee_id,
    COALESCE(e.first_name, 'No Employee') as first_name,
    COALESCE(d.department_name, 'No Department') as department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;

-- MySQL (FULL OUTER JOIN 미지원 시 UNION 사용)
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
UNION
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;
```

## SELF JOIN

### 기본 개념

같은 테이블을 별칭을 사용하여 자기 자신과 조인하는 방법입니다. 계층적 데이터나 같은 테이블 내의 다른 행과의 관계를 찾을 때 사용합니다.

### 관리자-직원 관계 조회

```sql
-- 직원과 그들의 관리자 정보
SELECT
    emp.employee_id,
    emp.first_name || ' ' || emp.last_name as employee_name,
    emp.salary,
    mgr.first_name || ' ' || mgr.last_name as manager_name,
    mgr.salary as manager_salary
FROM employees emp
LEFT JOIN employees mgr ON emp.manager_id = mgr.employee_id
ORDER BY emp.employee_id;

-- 관리자보다 급여가 높은 직원 찾기
SELECT
    emp.first_name || ' ' || emp.last_name as employee_name,
    emp.salary as employee_salary,
    mgr.first_name || ' ' || mgr.last_name as manager_name,
    mgr.salary as manager_salary
FROM employees emp
INNER JOIN employees mgr ON emp.manager_id = mgr.employee_id
WHERE emp.salary > mgr.salary;
```

### 계층적 쿼리 (재귀적 접근)

```sql
-- PostgreSQL의 재귀 CTE를 이용한 조직도
WITH RECURSIVE org_hierarchy AS (
    -- 최고 경영진 (관리자가 없는 직원)
    SELECT
        employee_id,
        first_name,
        last_name,
        manager_id,
        1 as level,
        CAST(first_name || ' ' || last_name AS VARCHAR(1000)) as path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- 하위 직원들
    SELECT
        e.employee_id,
        e.first_name,
        e.last_name,
        e.manager_id,
        oh.level + 1,
        oh.path || ' -> ' || e.first_name || ' ' || e.last_name
    FROM employees e
    INNER JOIN org_hierarchy oh ON e.manager_id = oh.employee_id
)
SELECT
    REPEAT('  ', level - 1) || first_name || ' ' || last_name as org_chart,
    level,
    path
FROM org_hierarchy
ORDER BY path;
```

### 동일 테이블에서 비교 조건

```sql
-- 같은 부서에서 급여 차이가 큰 직원들
SELECT DISTINCT
    e1.first_name || ' ' || e1.last_name as employee1,
    e1.salary as salary1,
    e2.first_name || ' ' || e2.last_name as employee2,
    e2.salary as salary2,
    ABS(e1.salary - e2.salary) as salary_diff
FROM employees e1
INNER JOIN employees e2 ON e1.department_id = e2.department_id
WHERE e1.employee_id < e2.employee_id  -- 중복 제거
AND ABS(e1.salary - e2.salary) > 5000
ORDER BY salary_diff DESC;
```

## CROSS JOIN

### 기본 개념

두 테이블의 카테시안 곱(Cartesian Product)을 생성합니다. 첫 번째 테이블의 각 행이 두 번째 테이블의 모든 행과 결합됩니다.

### 기본 문법과 예시

```sql
-- 명시적 CROSS JOIN
SELECT
    e.first_name,
    d.department_name
FROM employees e
CROSS JOIN departments d;

-- 암시적 CROSS JOIN (WHERE 절 없는 조인)
SELECT
    e.first_name,
    d.department_name
FROM employees e, departments d;

-- 실제 사용 예: 모든 직원에 대한 모든 프로젝트 할당 가능성
SELECT
    e.employee_id,
    e.first_name,
    p.project_id,
    p.project_name,
    'Available' as assignment_status
FROM employees e
CROSS JOIN projects p
WHERE e.department_id IN (SELECT department_id FROM project_departments WHERE project_id = p.project_id);
```

### CROSS JOIN의 실용적 활용

#### 1. 시간 시리즈 데이터 생성

```sql
-- 날짜 범위와 직원의 모든 조합 생성
WITH date_range AS (
    SELECT DATE '2023-01-01' + (n || ' days')::INTERVAL as work_date
    FROM generate_series(0, 364) n  -- PostgreSQL
)
SELECT
    e.employee_id,
    e.first_name,
    dr.work_date,
    CASE
        WHEN EXTRACT(DOW FROM dr.work_date) IN (0, 6) THEN 'Weekend'
        ELSE 'Workday'
    END as day_type
FROM employees e
CROSS JOIN date_range dr
WHERE e.department_id = 60
ORDER BY e.employee_id, dr.work_date;
```

#### 2. 조합 분석

```sql
-- 모든 가능한 팀 조합 (2명씩)
SELECT
    e1.first_name || ' ' || e1.last_name as member1,
    e2.first_name || ' ' || e2.last_name as member2,
    e1.department_id
FROM employees e1
CROSS JOIN employees e2
WHERE e1.employee_id < e2.employee_id  -- 중복 제거 및 순서 고정
AND e1.department_id = e2.department_id
AND e1.department_id = 60;
```

## 고급 조인 기법

### 1. 다중 조인 조건

```sql
-- 복합 키를 이용한 조인
SELECT *
FROM order_details od
INNER JOIN product_prices pp ON od.product_id = pp.product_id
    AND od.order_date BETWEEN pp.effective_date AND pp.end_date;

-- 범위 조인
SELECT
    e.first_name,
    sg.grade,
    sg.min_salary,
    sg.max_salary
FROM employees e
INNER JOIN salary_grades sg ON e.salary BETWEEN sg.min_salary AND sg.max_salary;
```

### 2. 조건부 조인

```sql
-- CASE문을 이용한 동적 조인
SELECT
    e.first_name,
    e.employee_type,
    CASE
        WHEN e.employee_type = 'FULL_TIME' THEN ft.benefit_package
        WHEN e.employee_type = 'PART_TIME' THEN pt.hourly_rate::VARCHAR
        ELSE 'N/A'
    END as employment_details
FROM employees e
LEFT JOIN full_time_benefits ft ON e.employee_id = ft.employee_id
    AND e.employee_type = 'FULL_TIME'
LEFT JOIN part_time_rates pt ON e.employee_id = pt.employee_id
    AND e.employee_type = 'PART_TIME';
```

### 3. 집계와 조인

```sql
-- 서브쿼리와 조인
SELECT
    d.department_name,
    dept_stats.employee_count,
    dept_stats.avg_salary,
    dept_stats.total_salary
FROM departments d
INNER JOIN (
    SELECT
        department_id,
        COUNT(*) as employee_count,
        AVG(salary) as avg_salary,
        SUM(salary) as total_salary
    FROM employees
    GROUP BY department_id
) dept_stats ON d.department_id = dept_stats.department_id
WHERE dept_stats.employee_count > 5;
```

### 4. 윈도우 함수와 조인

```sql
-- 부서별 급여 순위와 함께 조인
SELECT
    e.first_name,
    e.last_name,
    d.department_name,
    e.salary,
    RANK() OVER (PARTITION BY e.department_id ORDER BY e.salary DESC) as dept_salary_rank,
    AVG(e.salary) OVER (PARTITION BY e.department_id) as dept_avg_salary
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE e.salary IS NOT NULL
ORDER BY d.department_name, dept_salary_rank;
```

## 조인 성능 최적화

### 1. 인덱스 전략

```sql
-- 조인 컬럼에 인덱스 생성
CREATE INDEX idx_employees_dept_id ON employees(department_id);
CREATE INDEX idx_employees_manager_id ON employees(manager_id);

-- 복합 인덱스 (조인 + 필터 조건)
CREATE INDEX idx_employees_dept_salary ON employees(department_id, salary);

-- 조인과 정렬을 위한 복합 인덱스
CREATE INDEX idx_employees_dept_name ON employees(department_id, last_name, first_name);
```

### 2. 조인 알고리즘 이해

```sql
-- Nested Loop Join (소규모 데이터셋)
SELECT /*+ USE_NL(e, d) */ e.first_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- Hash Join (대규모 데이터셋)
SELECT /*+ USE_HASH(e, d) */ e.first_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- Sort Merge Join (정렬된 데이터)
SELECT /*+ USE_MERGE(e, d) */ e.first_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;
```

### 3. 실행 계획 분석

```sql
-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS)
SELECT e.first_name, d.department_name, l.city
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN locations l ON d.location_id = l.location_id
WHERE e.salary > 10000;

-- 확인할 포인트:
-- 1. Join Type (Nested Loop, Hash, Merge)
-- 2. Index usage
-- 3. Row estimates vs actual
-- 4. Memory usage
-- 5. I/O operations
```

### 4. 조인 순서 최적화

```sql
-- 가장 선택적인 조건부터 적용
SELECT e.first_name, d.department_name, l.city
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INNER JOIN locations l ON d.location_id = l.location_id
WHERE e.employee_id = 100  -- 가장 선택적
AND d.department_name = 'IT'
AND l.country = 'USA';
```

## 안티패턴과 해결책

### 1. 잘못된 조인 사용

```sql
-- 안티패턴: 불필요한 CROSS JOIN
SELECT COUNT(*)
FROM employees e, departments d
WHERE e.salary > 10000;  -- 조인 조건 누락

-- 해결책: 명시적 조인
SELECT COUNT(DISTINCT e.employee_id)
FROM employees e
WHERE e.salary > 10000;
```

### 2. N+1 문제 해결

```sql
-- 문제: 반복적인 단일 행 조회
-- SELECT * FROM employees WHERE department_id = ?; (반복)

-- 해결책: 한 번에 조인으로 처리
SELECT
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE e.department_id IN (10, 20, 30);
```

## 최신 SQL 표준과 확장

### 1. LATERAL JOIN (PostgreSQL, Oracle)

```sql
-- 각 부서별 상위 2명의 고액 연봉자
SELECT d.department_name, top_earners.*
FROM departments d
CROSS JOIN LATERAL (
    SELECT e.first_name, e.last_name, e.salary
    FROM employees e
    WHERE e.department_id = d.department_id
    ORDER BY e.salary DESC
    LIMIT 2
) top_earners;
```

### 2. APPLY 연산자 (SQL Server)

```sql
-- CROSS APPLY (INNER JOIN과 유사)
SELECT d.department_name, e.first_name, e.salary
FROM departments d
CROSS APPLY (
    SELECT TOP 1 first_name, salary
    FROM employees
    WHERE department_id = d.department_id
    ORDER BY salary DESC
) e;

-- OUTER APPLY (LEFT JOIN과 유사)
SELECT d.department_name, e.first_name, e.salary
FROM departments d
OUTER APPLY (
    SELECT TOP 1 first_name, salary
    FROM employees
    WHERE department_id = d.department_id
    ORDER BY salary DESC
) e;
```

## 결론

SQL 조인은 관계형 데이터베이스의 핵심 기능으로, 각 유형의 특성과 적절한 사용 시나리오를 이해하는 것이 중요합니다. 성능 최적화를 위해서는 적절한 인덱스 설계, 조인 순서, 실행 계획 분석이 필수적입니다. 현대의 SQL 확장 기능들도 활용하여 더욱 효율적인 쿼리를 작성할 수 있습니다.
