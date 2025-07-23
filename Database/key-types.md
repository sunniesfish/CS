# 키의 종류 (Types of Keys)

## 개요

키(Key)는 관계형 데이터베이스에서 튜플을 고유하게 식별하거나 관계를 표현하기 위해 사용되는 애트리뷰트 또는 애트리뷰트의 집합입니다. 키는 데이터의 무결성을 보장하고 효율적인 데이터 접근을 제공하는 핵심적인 개념입니다.

## 키의 탄생 배경

초기 데이터 관리 시스템에서는 레코드를 식별하기 위한 명확한 체계가 부족했습니다:

- **데이터 중복**: 동일한 엔터티가 여러 번 저장되는 문제
- **일관성 부족**: 데이터 업데이트 시 모든 중복 데이터를 찾기 어려운 문제
- **관계 표현 한계**: 엔터티 간의 관계를 명확히 표현할 방법 부족
- **검색 효율성**: 특정 레코드를 빠르게 찾을 수 없는 문제

이러한 문제를 해결하기 위해 수학의 함수 이론과 집합 이론을 바탕으로 키 개념이 도입되었습니다.

## 키의 분류 체계

### 1. 슈퍼 키 (Super Key)

#### 정의

슈퍼 키는 릴레이션 내에서 튜플을 유일하게 식별할 수 있는 애트리뷰트 또는 애트리뷰트의 집합입니다.

#### 특징

- **유일성**: 동일한 슈퍼 키 값을 가진 두 개의 서로 다른 튜플은 존재하지 않음
- **포함성**: 슈퍼 키에 임의의 애트리뷰트를 추가해도 여전히 슈퍼 키
- **최대성**: 가장 큰 슈퍼 키는 모든 애트리뷰트의 집합

#### 수학적 정의

릴레이션 R의 애트리뷰트 집합 A에 대해, A가 슈퍼 키가 되려면:

```
∀ t₁, t₂ ∈ R : t₁[A] = t₂[A] ⟹ t₁ = t₂
```

#### 예시

```
Student(StudentID, Name, Email, Phone, Address)

슈퍼 키의 예:
- {StudentID}
- {Email}
- {StudentID, Name}
- {StudentID, Name, Email}
- {StudentID, Name, Email, Phone, Address} (모든 애트리뷰트)
```

### 2. 후보 키 (Candidate Key)

#### 정의

후보 키는 슈퍼 키 중에서 더 이상 줄일 수 없는 최소한의 애트리뷰트 집합입니다.

#### 특징

- **유일성**: 튜플을 고유하게 식별
- **최소성**: 애트리뷰트를 하나라도 제거하면 유일성을 잃음
- **복수성**: 하나의 릴레이션에 여러 개의 후보 키가 존재할 수 있음

#### 최소성 검사 알고리즘

```
For each attribute A in candidate key K:
    If (K - {A}) can uniquely identify tuples:
        K is not minimal (not a candidate key)
    Else:
        Continue to next attribute
If all attributes are necessary:
    K is a candidate key
```

#### 예시

```
Student(StudentID, SSN, Email, Name, Phone)

후보 키들:
- {StudentID} ✓
- {SSN} ✓
- {Email} ✓
- {StudentID, Name} ✗ (최소성 위반: Name 제거 가능)
```

### 3. 기본 키 (Primary Key)

#### 정의

기본 키는 후보 키 중에서 릴레이션을 대표하기 위해 선택된 키입니다.

#### 특징

- **유일성**: NULL이 아닌 고유한 값
- **불변성**: 한번 할당된 후 변경되지 않음
- **최소성**: 필요 최소한의 애트리뷰트로 구성
- **단일성**: 하나의 릴레이션에는 하나의 기본 키만 존재

#### 선택 기준

1. **안정성**: 값이 자주 변경되지 않는 애트리뷰트
2. **단순성**: 가능한 한 적은 수의 애트리뷰트
3. **의미성**: 비즈니스적 의미가 있는 애트리뷰트
4. **성능**: 인덱스 생성과 검색에 유리한 구조

#### 대리 키 vs 자연 키

```sql
-- 자연 키 (Natural Key): 비즈니스 의미가 있는 키
CREATE TABLE Student (
    student_id VARCHAR(10) PRIMARY KEY,  -- 학번
    name VARCHAR(50) NOT NULL
);

-- 대리 키 (Surrogate Key): 인위적으로 생성된 키
CREATE TABLE Student (
    id SERIAL PRIMARY KEY,               -- 자동 증가 숫자
    student_id VARCHAR(10) UNIQUE NOT NULL,
    name VARCHAR(50) NOT NULL
);
```

### 4. 대체 키 (Alternate Key)

#### 정의

대체 키는 후보 키 중에서 기본 키로 선택되지 않은 나머지 키들입니다.

#### 특징

- **유일성**: 기본 키와 동일한 유일성 보장
- **선택성**: 기본 키 대신 사용될 수 있는 잠재력
- **제약조건**: UNIQUE 제약조건으로 구현

#### 활용

```sql
CREATE TABLE Student (
    id SERIAL PRIMARY KEY,                    -- 기본 키
    student_id VARCHAR(10) UNIQUE NOT NULL,  -- 대체 키 1
    email VARCHAR(100) UNIQUE NOT NULL,      -- 대체 키 2
    ssn CHAR(13) UNIQUE NOT NULL,           -- 대체 키 3
    name VARCHAR(50) NOT NULL
);
```

### 5. 외래 키 (Foreign Key)

#### 정의

외래 키는 다른 릴레이션의 기본 키를 참조하는 애트리뷰트 또는 애트리뷰트 집합입니다.

#### 특징

- **참조 무결성**: 참조하는 값은 반드시 피참조 릴레이션에 존재해야 함
- **NULL 허용**: 외래 키는 NULL 값을 가질 수 있음
- **관계 표현**: 릴레이션 간의 관계를 나타냄

#### 참조 무결성 제약조건

```sql
-- 외래 키 정의
CREATE TABLE Enrollment (
    enrollment_id SERIAL PRIMARY KEY,
    student_id VARCHAR(10) NOT NULL,
    course_id VARCHAR(10) NOT NULL,
    grade CHAR(2),

    -- 외래 키 제약조건
    FOREIGN KEY (student_id) REFERENCES Student(student_id)
        ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (course_id) REFERENCES Course(course_id)
        ON DELETE RESTRICT ON UPDATE CASCADE
);
```

#### 참조 동작 (Referential Actions)

1. **CASCADE**: 참조되는 튜플이 삭제/수정되면 참조하는 튜플도 삭제/수정
2. **SET NULL**: 참조되는 튜플이 삭제/수정되면 외래 키를 NULL로 설정
3. **SET DEFAULT**: 참조되는 튜플이 삭제/수정되면 외래 키를 기본값으로 설정
4. **RESTRICT/NO ACTION**: 참조하는 튜플이 있으면 삭제/수정 금지

### 6. 복합 키 (Composite Key)

#### 정의

복합 키는 두 개 이상의 애트리뷰트를 조합하여 만든 키입니다.

#### 특징

- **조합 유일성**: 개별 애트리뷰트는 중복 가능하지만 조합은 유일
- **부분 함수 종속**: 일부 애트리뷰트에만 함수 종속되는 경우 발생 가능
- **정규화 고려**: 2NF 위반 가능성 검토 필요

#### 예시

```sql
-- 수강 테이블의 복합 기본 키
CREATE TABLE Enrollment (
    student_id VARCHAR(10),
    course_id VARCHAR(10),
    semester VARCHAR(10),
    grade CHAR(2),

    PRIMARY KEY (student_id, course_id, semester)
);

-- 주문 상세의 복합 기본 키
CREATE TABLE OrderDetail (
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10,2),

    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

## 키 설계 원칙과 전략

### 1. 기본 키 설계 원칙

#### 안정성 (Stability)

```sql
-- 좋은 예: 변하지 않는 식별자
CREATE TABLE Employee (
    emp_id CHAR(10) PRIMARY KEY,  -- 사번 (변경 불가)
    ssn CHAR(11) UNIQUE NOT NULL, -- 주민번호 (대체 키)
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100)
);

-- 나쁜 예: 변할 수 있는 데이터
CREATE TABLE Employee (
    email VARCHAR(100) PRIMARY KEY,  -- 이메일은 변경 가능
    name VARCHAR(50) NOT NULL,
    department VARCHAR(50)
);
```

#### 단순성 (Simplicity)

```sql
-- 좋은 예: 단일 컬럼 기본 키
CREATE TABLE Product (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    category_id INT REFERENCES Category(id)
);

-- 복잡한 예: 복합 키 (필요한 경우에만)
CREATE TABLE ProductVariant (
    product_id INT,
    size VARCHAR(10),
    color VARCHAR(20),
    price DECIMAL(10,2),

    PRIMARY KEY (product_id, size, color)
);
```

### 2. 외래 키 설계 전략

#### 순환 참조 해결

```sql
-- 문제: 순환 참조
CREATE TABLE Employee (
    emp_id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    manager_id INT REFERENCES Employee(emp_id)  -- 자기 참조
);

-- 해결: 계층 구조 테이블
CREATE TABLE Department (
    dept_id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    parent_dept_id INT REFERENCES Department(dept_id)
);
```

#### 다중 관계 처리

```sql
-- 다대다 관계 해결
CREATE TABLE StudentCourse (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    grade CHAR(2),

    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES Student(id),
    FOREIGN KEY (course_id) REFERENCES Course(id)
);
```

### 3. 성능 최적화 고려사항

#### 인덱스 전략

```sql
-- 기본 키는 자동으로 클러스터드 인덱스 생성
-- 외래 키에는 별도 인덱스 생성 고려
CREATE INDEX idx_enrollment_student ON Enrollment(student_id);
CREATE INDEX idx_enrollment_course ON Enrollment(course_id);

-- 복합 인덱스 순서 고려
CREATE INDEX idx_order_date_customer ON Orders(order_date, customer_id);
```

#### 파티셔닝 고려

```sql
-- 기본 키를 이용한 파티셔닝
CREATE TABLE Orders (
    order_id SERIAL,
    customer_id INT,
    order_date DATE,
    ...
    PRIMARY KEY (order_id, order_date)
) PARTITION BY RANGE (order_date);
```

## 키 관련 무결성 제약조건

### 1. 개체 무결성 (Entity Integrity)

- 기본 키는 NULL 값을 가질 수 없음
- 기본 키는 릴레이션 내에서 유일해야 함

### 2. 참조 무결성 (Referential Integrity)

- 외래 키는 참조하는 릴레이션의 기본 키에 존재하는 값이어야 함
- 또는 NULL 값을 가져야 함

### 3. 도메인 무결성 (Domain Integrity)

- 키 애트리뷰트는 정의된 도메인 범위 내의 값만 가져야 함

## 현대적 키 관리 기법

### 1. UUID (Universally Unique Identifier)

```sql
-- UUID를 이용한 분산 시스템 친화적 키
CREATE TABLE User (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);
```

### 2. 순차 생성 키 (Sequential Keys)

```sql
-- 자동 증가 키
CREATE TABLE Order (
    order_id SERIAL PRIMARY KEY,           -- PostgreSQL
    -- id INT AUTO_INCREMENT PRIMARY KEY, -- MySQL
    customer_id INT NOT NULL,
    order_date DATE NOT NULL
);
```

### 3. 복합 비즈니스 키

```sql
-- 비즈니스 규칙을 반영한 키
CREATE TABLE Invoice (
    invoice_number VARCHAR(20) PRIMARY KEY,  -- INV-2024-001234 형식
    customer_id INT NOT NULL,
    issue_date DATE NOT NULL,

    CONSTRAINT chk_invoice_format
        CHECK (invoice_number ~ '^INV-\d{4}-\d{6}$')
);
```

## 키 설계 시 고려사항

### 1. 비즈니스 요구사항

- 자연 키 vs 대리 키 선택
- 키의 의미적 중요성
- 사용자 인터페이스에서의 표시 방식

### 2. 기술적 요구사항

- 성능 (인덱스, 파티셔닝)
- 확장성 (분산 시스템)
- 호환성 (다른 시스템과의 연동)

### 3. 유지보수성

- 키 값 변경의 어려움
- 마이그레이션 복잡성
- 백업 및 복구 전략

## 결론

키는 관계형 데이터베이스의 핵심 개념으로, 데이터의 무결성과 성능을 보장하는 중요한 역할을 합니다. 각 키 유형의 특성을 이해하고 적절히 설계하는 것은 효율적이고 안정적인 데이터베이스 시스템을 구축하는 기초가 됩니다. 현대의 분산 시스템과 클라우드 환경에서는 전통적인 키 개념에 새로운 고려사항들이 추가되고 있지만, 기본적인 키의 원리와 특성은 여전히 유효하며 중요합니다.
