# ER 모델과 관계형 모델 변환

## 개요

ER(Entity-Relationship) 모델은 데이터베이스 설계의 개념적 모델링 단계에서 사용되는 모델이며, 이를 실제 데이터베이스로 구현하기 위해서는 논리적 모델인 관계형 모델로 변환해야 합니다. 이 변환 과정은 데이터베이스 설계의 핵심 단계입니다.

## ER 모델의 기본 구성 요소

### 1. 엔터티 (Entity)

현실 세계에서 독립적으로 존재하는 객체나 개념

### 2. 애트리뷰트 (Attribute)

엔터티의 특성이나 성질

### 3. 관계 (Relationship)

엔터티들 간의 연관성

## 변환 규칙

### 1. 강한 엔터티 → 릴레이션

```sql
-- ER: Student(student_id, name, age, major)
CREATE TABLE Student (
    student_id VARCHAR(10) PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    age INT,
    major VARCHAR(30)
);
```

### 2. 약한 엔터티 → 릴레이션

```sql
-- 의존하는 강한 엔터티의 기본 키 + 부분 키 = 복합 기본 키
CREATE TABLE Dependent (
    emp_id INT,
    dependent_name VARCHAR(50),
    relationship VARCHAR(20),
    birth_date DATE,

    PRIMARY KEY (emp_id, dependent_name),
    FOREIGN KEY (emp_id) REFERENCES Employee(emp_id)
);
```

### 3. 1:1 관계 변환

**방법 1: 외래 키 추가**

```sql
-- Employee와 Parking_Spot의 1:1 관계
ALTER TABLE Employee
ADD COLUMN parking_spot_id INT UNIQUE,
ADD FOREIGN KEY (parking_spot_id) REFERENCES Parking_Spot(spot_id);
```

**방법 2: 테이블 병합**

```sql
CREATE TABLE Employee (
    emp_id INT PRIMARY KEY,
    name VARCHAR(50),
    parking_spot_number VARCHAR(10),
    parking_location VARCHAR(50)
);
```

### 4. 1:N 관계 변환

```sql
-- Department(1) : Employee(N)
CREATE TABLE Employee (
    emp_id INT PRIMARY KEY,
    name VARCHAR(50),
    dept_id INT NOT NULL,  -- 외래 키는 'N'쪽에 추가
    FOREIGN KEY (dept_id) REFERENCES Department(dept_id)
);
```

### 5. M:N 관계 변환

```sql
-- Student와 Course의 M:N 관계
CREATE TABLE Enrollment (
    student_id VARCHAR(10),
    course_id VARCHAR(10),
    enrollment_date DATE,
    grade CHAR(2),

    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES Student(student_id),
    FOREIGN KEY (course_id) REFERENCES Course(course_id)
);
```

## 복합 애트리뷰트 처리

### 1. 단순 분해

```sql
-- Address(street, city, state, zip) → 개별 컬럼
CREATE TABLE Customer (
    customer_id INT PRIMARY KEY,
    street VARCHAR(100),
    city VARCHAR(50),
    state VARCHAR(20),
    zip_code VARCHAR(10)
);
```

### 2. JSON 타입 활용 (현대적 접근)

```sql
CREATE TABLE Customer (
    customer_id INT PRIMARY KEY,
    address JSONB  -- {"street": "123 Main St", "city": "Seoul"}
);
```

## 다중값 애트리뷰트 처리

```sql
-- Employee의 Skills (다중값) → 별도 테이블
CREATE TABLE Employee_Skill (
    emp_id INT,
    skill VARCHAR(50),
    proficiency_level VARCHAR(20),

    PRIMARY KEY (emp_id, skill),
    FOREIGN KEY (emp_id) REFERENCES Employee(emp_id)
);
```

## 유도 애트리뷰트 처리

### 1. 계산된 컬럼 (MySQL, SQL Server)

```sql
CREATE TABLE Employee (
    emp_id INT PRIMARY KEY,
    birth_date DATE,
    age INT AS (YEAR(CURDATE()) - YEAR(birth_date)) STORED
);
```

### 2. 뷰 활용

```sql
CREATE VIEW Employee_With_Age AS
SELECT emp_id, name, birth_date,
       EXTRACT(YEAR FROM AGE(birth_date)) AS age
FROM Employee;
```

## 특수한 변환 사례

### 1. ISA 관계 (상속) 변환

**방법 1: 단일 테이블 (Table per Hierarchy)**

```sql
CREATE TABLE Person (
    person_id INT PRIMARY KEY,
    name VARCHAR(50),
    person_type VARCHAR(20), -- 'Student', 'Employee'

    -- Student 속성
    student_id VARCHAR(10),
    major VARCHAR(30),

    -- Employee 속성
    salary DECIMAL(10,2),
    department VARCHAR(30)
);
```

**방법 2: 클래스별 테이블 (Table per Class)**

```sql
CREATE TABLE Person (
    person_id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE Student (
    person_id INT PRIMARY KEY,
    student_id VARCHAR(10),
    major VARCHAR(30),
    FOREIGN KEY (person_id) REFERENCES Person(person_id)
);

CREATE TABLE Employee (
    person_id INT PRIMARY KEY,
    salary DECIMAL(10,2),
    department VARCHAR(30),
    FOREIGN KEY (person_id) REFERENCES Person(person_id)
);
```

### 2. 카테고리 (Category) 변환

```sql
-- Owner는 Person 또는 Company가 될 수 있음
CREATE TABLE Owner (
    owner_id INT PRIMARY KEY,
    owner_type VARCHAR(20), -- 'Person' or 'Company'
    person_id INT,
    company_id INT,

    FOREIGN KEY (person_id) REFERENCES Person(person_id),
    FOREIGN KEY (company_id) REFERENCES Company(company_id),

    CONSTRAINT chk_owner_type CHECK (
        (owner_type = 'Person' AND person_id IS NOT NULL AND company_id IS NULL) OR
        (owner_type = 'Company' AND company_id IS NOT NULL AND person_id IS NULL)
    )
);
```

## 변환 시 고려사항

### 1. 정규화 수준 결정

```sql
-- 높은 정규화: 데이터 중복 최소화
CREATE TABLE Order_Item (
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2), -- Product 테이블에서 참조
    PRIMARY KEY (order_id, product_id)
);

-- 낮은 정규화: 성능 최적화
CREATE TABLE Order_Item (
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),  -- 중복 저장으로 JOIN 최소화
    product_name VARCHAR(100), -- 중복 저장
    PRIMARY KEY (order_id, product_id)
);
```

### 2. 제약조건 설정

```sql
CREATE TABLE Employee (
    emp_id INT PRIMARY KEY,
    manager_id INT,
    salary DECIMAL(10,2) CHECK (salary > 0),
    hire_date DATE CHECK (hire_date <= CURRENT_DATE),

    FOREIGN KEY (manager_id) REFERENCES Employee(emp_id)
);
```

## 최적화 전략

### 1. 인덱스 설계

```sql
-- 외래 키에 인덱스 생성
CREATE INDEX idx_employee_dept ON Employee(dept_id);
CREATE INDEX idx_enrollment_student ON Enrollment(student_id);
CREATE INDEX idx_enrollment_course ON Enrollment(course_id);

-- 복합 인덱스
CREATE INDEX idx_order_customer_date ON Order(customer_id, order_date);
```

### 2. 파티셔닝 고려

```sql
-- 날짜 기반 파티셔닝
CREATE TABLE Sales (
    sale_id SERIAL,
    sale_date DATE,
    customer_id INT,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (sale_date);

CREATE TABLE Sales_2023 PARTITION OF Sales
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');
```

## 변환 검증

### 1. 데이터 무결성 검증

```sql
-- 참조 무결성 검사
SELECT COUNT(*) FROM Employee e
LEFT JOIN Department d ON e.dept_id = d.dept_id
WHERE d.dept_id IS NULL; -- 0이어야 함

-- 도메인 무결성 검사
SELECT COUNT(*) FROM Employee
WHERE salary <= 0 OR hire_date > CURRENT_DATE; -- 0이어야 함
```

### 2. 완전성 검증

```sql
-- 모든 엔터티가 변환되었는지 확인
-- 모든 관계가 적절히 표현되었는지 확인
-- 모든 제약조건이 구현되었는지 확인
```

## 현대적 변환 기법

### 1. 스키마리스 접근

```sql
-- JSON을 활용한 유연한 스키마
CREATE TABLE Entity (
    id SERIAL PRIMARY KEY,
    entity_type VARCHAR(50),
    attributes JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- JSON 스키마 검증
ALTER TABLE Entity ADD CONSTRAINT chk_student_schema
CHECK (
    entity_type != 'Student' OR (
        attributes ? 'student_id' AND
        attributes ? 'major' AND
        jsonb_typeof(attributes->'student_id') = 'string'
    )
);
```

### 2. 그래프 데이터베이스 고려

```sql
-- 복잡한 관계는 그래프 DB가 더 적합할 수 있음
-- Neo4j 예시 (Cypher 쿼리)
-- CREATE (s:Student {student_id: '2021001', name: 'Kim'})
-- CREATE (c:Course {course_id: 'CS101', title: 'Database'})
-- CREATE (s)-[:ENROLLED_IN {grade: 'A'}]->(c)
```

## 결론

ER 모델에서 관계형 모델로의 변환은 체계적인 규칙을 따르되, 성능, 유지보수성, 확장성을 고려한 최적화가 필요합니다. 현대적인 데이터베이스는 JSON, 배열 등의 확장 타입을 지원하므로 이를 활용한 유연한 설계도 가능합니다.
