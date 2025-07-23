# 서브쿼리와 IN, EXISTS

## 개요

서브쿼리(Subquery)는 다른 SQL 문 내부에 포함된 SELECT 문으로, 복잡한 데이터 검색과 조건부 논리를 구현하는 강력한 도구입니다. IN과 EXISTS는 서브쿼리와 함께 자주 사용되는 중요한 연산자입니다.

## 서브쿼리의 분류

### 1. 위치에 따른 분류

- **스칼라 서브쿼리**: SELECT 절에서 사용
- **인라인 뷰**: FROM 절에서 사용
- **중첩 서브쿼리**: WHERE/HAVING 절에서 사용

### 2. 결과에 따른 분류

- **단일 행 서브쿼리**: 하나의 행만 반환
- **다중 행 서브쿼리**: 여러 행 반환
- **다중 열 서브쿼리**: 여러 컬럼 반환

### 3. 의존성에 따른 분류

- **독립 서브쿼리**: 외부 쿼리와 무관하게 실행
- **상관 서브쿼리**: 외부 쿼리의 값에 의존

## 스칼라 서브쿼리 (Scalar Subquery)

### 기본 개념

SELECT 절에서 사용되며 정확히 하나의 값을 반환해야 합니다.

```sql
-- 각 직원의 급여와 전체 평균 급여 비교
SELECT
    employee_id,
    first_name,
    salary,
    (SELECT AVG(salary) FROM employees) as avg_salary,
    salary - (SELECT AVG(salary) FROM employees) as salary_diff
FROM employees;

-- 부서별 직원 수 포함
SELECT
    e.employee_id,
    e.first_name,
    d.department_name,
    (SELECT COUNT(*)
     FROM employees e2
     WHERE e2.department_id = e.department_id) as dept_employee_count
FROM employees e
JOIN departments d ON e.department_id = d.department_id;
```

### CASE문과 스칼라 서브쿼리

```sql
-- 복잡한 조건부 계산
SELECT
    employee_id,
    first_name,
    salary,
    CASE
        WHEN salary > (SELECT AVG(salary) FROM employees WHERE department_id = e.department_id)
        THEN 'Above Department Average'
        WHEN salary > (SELECT AVG(salary) FROM employees)
        THEN 'Above Company Average'
        ELSE 'Below Average'
    END as performance_level
FROM employees e;
```

## 인라인 뷰 (Inline View)

### 기본 사용법

FROM 절에서 테이블처럼 사용되는 서브쿼리입니다.

```sql
-- 부서별 통계와 조인
SELECT
    d.department_name,
    dept_stats.employee_count,
    dept_stats.avg_salary,
    dept_stats.max_salary
FROM departments d
JOIN (
    SELECT
        department_id,
        COUNT(*) as employee_count,
        AVG(salary) as avg_salary,
        MAX(salary) as max_salary
    FROM employees
    GROUP BY department_id
) dept_stats ON d.department_id = dept_stats.department_id
WHERE dept_stats.employee_count > 5;
```

### 복잡한 집계와 순위

```sql
-- 각 부서의 상위 3명 고액 연봉자
SELECT *
FROM (
    SELECT
        e.*,
        d.department_name,
        ROW_NUMBER() OVER (PARTITION BY e.department_id ORDER BY e.salary DESC) as salary_rank
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
) ranked_employees
WHERE salary_rank <= 3
ORDER BY department_name, salary_rank;
```

## 중첩 서브쿼리 (Nested Subquery)

### 단일 행 서브쿼리

```sql
-- 가장 높은 급여를 받는 직원
SELECT *
FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees);

-- 평균 급여보다 높은 급여를 받는 직원
SELECT employee_id, first_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- 특정 부서의 평균 급여보다 높은 모든 직원
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = 60
);
```

### 다중 행 서브쿼리

```sql
-- 관리자인 직원들의 정보
SELECT *
FROM employees
WHERE employee_id IN (
    SELECT DISTINCT manager_id
    FROM employees
    WHERE manager_id IS NOT NULL
);

-- 각 부서의 최고 급여와 같은 급여를 받는 직원
SELECT *
FROM employees
WHERE (department_id, salary) IN (
    SELECT department_id, MAX(salary)
    FROM employees
    GROUP BY department_id
);
```

## IN 연산자

### 기본 사용법

```sql
-- 리터럴 값들과 비교
SELECT *
FROM employees
WHERE department_id IN (10, 20, 30);

-- 서브쿼리와 함께 사용
SELECT *
FROM employees
WHERE job_id IN (
    SELECT job_id
    FROM job_history
    WHERE end_date > CURRENT_DATE - INTERVAL '1 year'
);
```

### 다중 컬럼 IN

```sql
-- 여러 컬럼을 동시에 비교
SELECT *
FROM employees
WHERE (department_id, job_id) IN (
    SELECT department_id, job_id
    FROM job_history
    WHERE end_date IS NULL  -- 현재 직위
);

-- 복잡한 조건
SELECT *
FROM order_details
WHERE (product_id, order_date) IN (
    SELECT product_id, MAX(order_date)
    FROM order_details
    GROUP BY product_id
);
```

### NOT IN의 주의사항

```sql
-- 위험: NULL 값이 있으면 예상과 다른 결과
SELECT *
FROM employees
WHERE department_id NOT IN (
    SELECT department_id
    FROM departments
    WHERE location_id = 1700
);

-- 안전한 방법: NULL 처리
SELECT *
FROM employees
WHERE department_id NOT IN (
    SELECT department_id
    FROM departments
    WHERE location_id = 1700
    AND department_id IS NOT NULL
);

-- 더 안전한 방법: NOT EXISTS 사용
SELECT *
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM departments d
    WHERE d.department_id = e.department_id
    AND d.location_id = 1700
);
```

## EXISTS 연산자

### 기본 개념

EXISTS는 서브쿼리가 하나 이상의 행을 반환하는지 확인합니다.

```sql
-- 부하직원이 있는 관리자 찾기
SELECT *
FROM employees mgr
WHERE EXISTS (
    SELECT 1
    FROM employees emp
    WHERE emp.manager_id = mgr.employee_id
);

-- 주문이 있는 고객만 조회
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

### NOT EXISTS

```sql
-- 주문이 없는 고객 찾기
SELECT *
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);

-- 부하직원이 없는 직원 (일반직원) 찾기
SELECT *
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM employees sub
    WHERE sub.manager_id = e.employee_id
);
```

### EXISTS vs IN 성능 비교

```sql
-- EXISTS: 일반적으로 더 효율적
SELECT *
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
    AND e.salary > 10000
);

-- IN: 서브쿼리 결과를 메모리에 저장
SELECT *
FROM departments
WHERE department_id IN (
    SELECT department_id
    FROM employees
    WHERE salary > 10000
);
```

## 상관 서브쿼리 (Correlated Subquery)

### 기본 개념

외부 쿼리의 각 행에 대해 서브쿼리가 실행됩니다.

```sql
-- 각 직원보다 급여가 높은 동료 수
SELECT
    e1.employee_id,
    e1.first_name,
    e1.salary,
    (SELECT COUNT(*)
     FROM employees e2
     WHERE e2.department_id = e1.department_id
     AND e2.salary > e1.salary) as higher_paid_colleagues
FROM employees e1;

-- 부서 내에서 평균보다 높은 급여를 받는 직원
SELECT *
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

### 복잡한 상관 서브쿼리

```sql
-- 각 고객의 최근 주문보다 더 큰 주문이 있는 고객
SELECT DISTINCT c.*
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o1
    WHERE o1.customer_id = c.customer_id
    AND EXISTS (
        SELECT 1
        FROM orders o2
        WHERE o2.customer_id = c.customer_id
        AND o2.order_date > o1.order_date
        AND o2.total_amount > o1.total_amount
    )
);
```

## ANY, ALL 연산자

### ANY (SOME) 연산자

```sql
-- 어떤 부서의 최대 급여보다 높은 급여를 받는 직원
SELECT *
FROM employees
WHERE salary > ANY (
    SELECT MAX(salary)
    FROM employees
    GROUP BY department_id
);

-- 위는 다음과 같음
SELECT *
FROM employees
WHERE salary > (
    SELECT MIN(MAX(salary))
    FROM employees
    GROUP BY department_id
);
```

### ALL 연산자

```sql
-- 모든 부서의 최대 급여보다 높은 급여를 받는 직원
SELECT *
FROM employees
WHERE salary > ALL (
    SELECT MAX(salary)
    FROM employees
    GROUP BY department_id
);

-- 위는 다음과 같음
SELECT *
FROM employees
WHERE salary > (
    SELECT MAX(MAX(salary))
    FROM employees
    GROUP BY department_id
);
```

## 고급 서브쿼리 기법

### 1. 다중 레벨 서브쿼리

```sql
-- 3단계 중첩 서브쿼리
SELECT *
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE location_id IN (
        SELECT location_id
        FROM locations
        WHERE country_id IN ('US', 'UK', 'CA')
    )
);
```

### 2. 서브쿼리와 집계 함수

```sql
-- 각 부서에서 두 번째로 높은 급여
SELECT
    department_id,
    (SELECT DISTINCT salary
     FROM employees e2
     WHERE e2.department_id = e1.department_id
     ORDER BY salary DESC
     LIMIT 1 OFFSET 1) as second_highest_salary
FROM (SELECT DISTINCT department_id FROM employees) e1;

-- 윈도우 함수를 이용한 더 효율적인 방법
SELECT DISTINCT
    department_id,
    NTH_VALUE(salary, 2) OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
        ROWS UNBOUNDED PRECEDING
    ) as second_highest_salary
FROM employees;
```

### 3. 조건부 서브쿼리

```sql
-- CASE와 서브쿼리 조합
SELECT
    employee_id,
    first_name,
    CASE
        WHEN department_id = 60 THEN
            (SELECT 'Tech: ' || job_title
             FROM jobs j
             WHERE j.job_id = employees.job_id)
        ELSE
            (SELECT 'Business: ' || job_title
             FROM jobs j
             WHERE j.job_id = employees.job_id)
    END as categorized_job
FROM employees;
```

## 서브쿼리 최적화

### 1. 서브쿼리를 JOIN으로 변환

```sql
-- 비효율적인 서브쿼리
SELECT *
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE location_id = 1700
);

-- 효율적인 JOIN
SELECT DISTINCT e.*
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
WHERE d.location_id = 1700;
```

### 2. EXISTS vs IN 선택 기준

```sql
-- 큰 테이블에서 작은 테이블 확인: EXISTS가 유리
SELECT *
FROM large_table lt
WHERE EXISTS (
    SELECT 1
    FROM small_lookup sl
    WHERE sl.id = lt.lookup_id
    AND sl.active = 'Y'
);

-- 작은 테이블에서 큰 테이블 확인: IN이 유리할 수 있음
SELECT *
FROM small_table st
WHERE st.id IN (
    SELECT large_id
    FROM large_table
    WHERE status = 'ACTIVE'
);
```

### 3. 상관 서브쿼리 최적화

```sql
-- 비효율적: 각 행마다 서브쿼리 실행
SELECT
    e.employee_id,
    (SELECT COUNT(*) FROM employees e2 WHERE e2.department_id = e.department_id) as dept_count
FROM employees e;

-- 효율적: JOIN과 윈도우 함수 사용
SELECT
    e.employee_id,
    COUNT(*) OVER (PARTITION BY e.department_id) as dept_count
FROM employees e;
```

## CTE (Common Table Expression)와 서브쿼리

### 기본 CTE

```sql
-- 복잡한 서브쿼리를 CTE로 단순화
WITH dept_stats AS (
    SELECT
        department_id,
        AVG(salary) as avg_salary,
        COUNT(*) as employee_count
    FROM employees
    GROUP BY department_id
),
high_performing_depts AS (
    SELECT department_id
    FROM dept_stats
    WHERE avg_salary > 8000 AND employee_count >= 5
)
SELECT e.*, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.department_id IN (SELECT department_id FROM high_performing_depts);
```

### 재귀 CTE

```sql
-- 조직 계층 구조
WITH RECURSIVE org_tree AS (
    -- 기본 케이스: 최고 관리자
    SELECT employee_id, first_name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- 재귀 케이스
    SELECT e.employee_id, e.first_name, e.manager_id, ot.level + 1
    FROM employees e
    JOIN org_tree ot ON e.manager_id = ot.employee_id
)
SELECT * FROM org_tree ORDER BY level, employee_id;
```

## 실행 계획과 성능 분석

```sql
-- PostgreSQL 실행 계획
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM employees e
WHERE salary > (SELECT AVG(salary) FROM employees);

-- 서브쿼리 최적화 확인 포인트:
-- 1. SubPlan vs HashAggregate
-- 2. Materialize 노드
-- 3. 실행 시간과 메모리 사용량
-- 4. 인덱스 사용 여부
```

## 안티패턴과 해결책

### 1. 불필요한 서브쿼리

```sql
-- 안티패턴
SELECT *
FROM employees
WHERE department_id = (
    SELECT department_id
    FROM departments
    WHERE department_name = 'IT'
);

-- 개선: 직접 조인
SELECT e.*
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE d.department_name = 'IT';
```

### 2. 중복 서브쿼리 실행

```sql
-- 안티패턴: 같은 서브쿼리 반복
SELECT
    employee_id,
    salary,
    CASE
        WHEN salary > (SELECT AVG(salary) FROM employees) THEN 'High'
        ELSE 'Low'
    END as grade,
    salary - (SELECT AVG(salary) FROM employees) as diff
FROM employees;

-- 개선: CTE 사용
WITH avg_salary AS (
    SELECT AVG(salary) as company_avg FROM employees
)
SELECT
    employee_id,
    salary,
    CASE WHEN salary > company_avg THEN 'High' ELSE 'Low' END as grade,
    salary - company_avg as diff
FROM employees, avg_salary;
```

## 결론

서브쿼리는 복잡한 데이터 검색 로직을 구현하는 강력한 도구이지만, 성능을 고려한 적절한 사용이 중요합니다. IN, EXISTS, ANY, ALL 연산자의 특성을 이해하고, 상황에 따라 JOIN이나 윈도우 함수로 대체하는 것을 고려해야 합니다. 현대 SQL의 CTE와 윈도우 함수를 활용하면 더욱 읽기 쉽고 효율적인 쿼리를 작성할 수 있습니다.
