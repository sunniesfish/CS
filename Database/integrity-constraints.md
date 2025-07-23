# 무결성 제약조건 (Integrity Constraints)

## 개요

무결성 제약조건은 데이터베이스에 저장된 데이터의 정확성, 일관성, 신뢰성을 보장하기 위한 규칙들의 집합입니다. 이는 부정확하거나 일관성 없는 데이터가 데이터베이스에 저장되는 것을 방지하고, 비즈니스 규칙을 데이터베이스 레벨에서 강제하는 중요한 메커니즘입니다.

## 탄생 배경

초기 데이터 관리 시스템에서는 데이터 품질 관리가 주로 응용 프로그램 수준에서 이루어졌습니다:

- **일관성 부족**: 서로 다른 응용 프로그램에서 동일한 데이터에 대해 다른 검증 규칙 적용
- **중복 검증**: 모든 응용 프로그램에서 동일한 검증 로직을 반복 구현
- **우회 가능성**: 응용 프로그램을 거치지 않고 직접 데이터베이스에 접근하여 부정확한 데이터 입력
- **유지보수 문제**: 비즈니스 규칙 변경 시 모든 응용 프로그램 수정 필요

이러한 문제를 해결하기 위해 데이터베이스 레벨에서 무결성을 보장하는 제약조건 개념이 도입되었습니다.

## 무결성 제약조건의 분류

### 1. 개체 무결성 (Entity Integrity)

#### 정의

개체 무결성은 기본 키에 대한 제약조건으로, 기본 키의 어떤 애트리뷰트도 NULL 값을 가질 수 없으며, 릴레이션 내에서 기본 키 값은 유일해야 한다는 규칙입니다.

#### 이론적 배경

관계형 모델에서 각 튜플은 고유하게 식별 가능해야 하며, 이를 위해서는 기본 키가 명확하게 정의되어야 합니다. NULL 값은 '알 수 없는 값'을 의미하므로, 식별자 역할을 하는 기본 키에는 적합하지 않습니다.

#### 구현 방법

```sql
-- 단일 컬럼 기본 키
CREATE TABLE Student (
    student_id VARCHAR(10) PRIMARY KEY,  -- NULL 불가, 유일성 보장
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100)
);

-- 복합 기본 키
CREATE TABLE Enrollment (
    student_id VARCHAR(10),
    course_id VARCHAR(10),
    semester VARCHAR(10),
    grade CHAR(2),

    PRIMARY KEY (student_id, course_id, semester)  -- 조합이 유일해야 함
);

-- 대안적 구현 (명시적 제약조건)
CREATE TABLE Product (
    product_id SERIAL,
    name VARCHAR(100) NOT NULL,

    CONSTRAINT pk_product PRIMARY KEY (product_id)
);
```

#### 위반 시 발생하는 문제들

1. **튜플 식별 불가**: NULL 기본 키로 인한 튜플 구별 불가
2. **참조 무결성 침해**: 다른 테이블에서 NULL 기본 키 참조 시 문제
3. **인덱스 효율성 저하**: NULL 값으로 인한 인덱스 성능 저하

### 2. 참조 무결성 (Referential Integrity)

#### 정의

참조 무결성은 외래 키에 대한 제약조건으로, 외래 키 값은 참조하는 릴레이션의 기본 키에 존재하는 값이거나 NULL이어야 한다는 규칙입니다.

#### 이론적 배경

관계형 데이터베이스에서 릴레이션 간의 관계는 외래 키를 통해 표현됩니다. 참조 무결성은 이러한 관계의 일관성을 보장하여 "참조하는 대상이 실제로 존재한다"는 것을 보증합니다.

#### 구현 방법

```sql
-- 기본적인 외래 키 제약조건
CREATE TABLE Order (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,

    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id)
);

-- 참조 동작 지정
CREATE TABLE OrderDetail (
    detail_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,

    FOREIGN KEY (order_id) REFERENCES Order(order_id)
        ON DELETE CASCADE      -- 주문 삭제 시 상세도 삭제
        ON UPDATE CASCADE,     -- 주문 ID 변경 시 상세도 함께 변경

    FOREIGN KEY (product_id) REFERENCES Product(product_id)
        ON DELETE RESTRICT     -- 상품 삭제 금지 (참조하는 주문 상세 있으면)
        ON UPDATE CASCADE
);
```

#### 참조 동작 (Referential Actions)의 상세 분석

##### CASCADE

```sql
-- 예시: 고객 삭제 시 해당 고객의 모든 주문도 함께 삭제
DELETE FROM Customer WHERE customer_id = 100;
-- 자동으로 실행됨: DELETE FROM Order WHERE customer_id = 100;
```

##### RESTRICT vs NO ACTION

```sql
-- RESTRICT: 즉시 제약조건 검사
DELETE FROM Customer WHERE customer_id = 100;  -- 주문이 있으면 즉시 오류

-- NO ACTION: 트랜잭션 끝에서 제약조건 검사 (지연 가능)
SET CONSTRAINTS fk_order_customer DEFERRED;
DELETE FROM Customer WHERE customer_id = 100;  -- 일단 허용
-- 트랜잭션 커밋 시점에 오류 발생 (관련 주문이 정리되지 않았다면)
```

##### SET NULL vs SET DEFAULT

```sql
-- SET NULL: 참조되는 행 삭제 시 외래 키를 NULL로 설정
ALTER TABLE Order ADD CONSTRAINT fk_order_customer
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id)
    ON DELETE SET NULL;

-- SET DEFAULT: 참조되는 행 삭제 시 기본값으로 설정
ALTER TABLE Order ALTER COLUMN customer_id SET DEFAULT 0;
ALTER TABLE Order ADD CONSTRAINT fk_order_customer
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id)
    ON DELETE SET DEFAULT;
```

### 3. 도메인 무결성 (Domain Integrity)

#### 정의

도메인 무결성은 애트리뷰트 값이 정의된 도메인(데이터 타입, 형식, 범위 등) 내에서만 허용된다는 제약조건입니다.

#### 구성 요소

##### 데이터 타입 제약조건

```sql
CREATE TABLE Product (
    product_id SERIAL,
    name VARCHAR(100) NOT NULL,           -- 문자열, 최대 100자
    price DECIMAL(10,2) NOT NULL,         -- 숫자, 소수점 2자리
    manufacture_date DATE,                -- 날짜 형식
    is_active BOOLEAN DEFAULT TRUE        -- 참/거짓
);
```

##### CHECK 제약조건

```sql
CREATE TABLE Employee (
    emp_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    age INT CHECK (age >= 18 AND age <= 65),           -- 나이 범위
    salary DECIMAL(10,2) CHECK (salary > 0),           -- 급여는 양수
    email VARCHAR(100) CHECK (email LIKE '%@%.%'),     -- 이메일 형식
    department VARCHAR(20) CHECK (department IN ('HR', 'IT', 'SALES', 'MARKETING')),
    hire_date DATE CHECK (hire_date <= CURRENT_DATE),  -- 입사일은 현재일 이전

    -- 복합 체크 제약조건
    CONSTRAINT chk_salary_age CHECK (
        (age < 30 AND salary <= 50000) OR
        (age >= 30 AND salary <= 100000)
    )
);
```

##### 정규 표현식을 이용한 복잡한 패턴

```sql
CREATE TABLE Customer (
    customer_id SERIAL PRIMARY KEY,
    phone VARCHAR(20) CHECK (phone ~ '^[0-9]{3}-[0-9]{3,4}-[0-9]{4}$'),  -- 전화번호 형식
    ssn CHAR(13) CHECK (ssn ~ '^\d{6}-\d{7}$'),                          -- 주민번호 형식
    postal_code VARCHAR(10) CHECK (postal_code ~ '^\d{3}-\d{3}$'),       -- 우편번호 형식
    credit_score INT CHECK (credit_score BETWEEN 300 AND 850)            -- 신용점수 범위
);
```

### 4. 키 무결성 (Key Integrity)

#### 정의

키 무결성은 슈퍼 키, 후보 키, 기본 키, 외래 키 등 모든 종류의 키에 대한 제약조건을 포괄하는 개념입니다.

#### 구성 요소

##### UNIQUE 제약조건

```sql
CREATE TABLE User (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(30) UNIQUE NOT NULL,    -- 중복 불가
    email VARCHAR(100) UNIQUE NOT NULL,      -- 중복 불가
    ssn CHAR(13) UNIQUE,                     -- 중복 불가 (NULL 허용)

    -- 복합 유니크 제약조건
    first_name VARCHAR(30),
    last_name VARCHAR(30),
    birth_date DATE,
    CONSTRAINT uk_user_name_birth UNIQUE (first_name, last_name, birth_date)
);
```

##### 부분 키 제약조건

```sql
-- 약한 엔터티의 부분 키
CREATE TABLE Dependent (
    emp_id INT,                    -- 직원 ID (외래 키)
    dependent_name VARCHAR(50),    -- 부양가족 이름 (부분 키)
    relationship VARCHAR(20),
    birth_date DATE,

    PRIMARY KEY (emp_id, dependent_name),  -- 복합 기본 키
    FOREIGN KEY (emp_id) REFERENCES Employee(emp_id)
        ON DELETE CASCADE
);
```

## 고급 무결성 제약조건

### 1. 어설션 (Assertion)

#### 정의

어설션은 데이터베이스의 상태에 대한 일반적인 제약조건으로, 여러 테이블에 걸친 복잡한 비즈니스 규칙을 표현할 수 있습니다.

#### 예시 (개념적 - 모든 DBMS가 지원하지는 않음)

```sql
-- 개념적 예시: 부서별 직원 수 제한
CREATE ASSERTION max_dept_employees
CHECK (NOT EXISTS (
    SELECT dept_id
    FROM Employee
    GROUP BY dept_id
    HAVING COUNT(*) > 50
));

-- 실제 구현 대안: 트리거 사용
CREATE OR REPLACE FUNCTION check_dept_employee_limit()
RETURNS TRIGGER AS $$
BEGIN
    IF (SELECT COUNT(*) FROM Employee WHERE dept_id = NEW.dept_id) > 50 THEN
        RAISE EXCEPTION 'Department cannot have more than 50 employees';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER tg_dept_employee_limit
    BEFORE INSERT OR UPDATE ON Employee
    FOR EACH ROW
    EXECUTE FUNCTION check_dept_employee_limit();
```

### 2. 트리거를 이용한 복잡한 무결성

#### 비즈니스 규칙 구현

```sql
-- 예시: 주문 취소 시 재고 복구
CREATE OR REPLACE FUNCTION restore_inventory()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.status = 'confirmed' AND NEW.status = 'cancelled' THEN
        UPDATE Product
        SET stock_quantity = stock_quantity + OLD.quantity
        WHERE product_id = OLD.product_id;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER tg_restore_inventory
    AFTER UPDATE ON OrderItem
    FOR EACH ROW
    EXECUTE FUNCTION restore_inventory();
```

#### 감사 로그 생성

```sql
-- 중요 데이터 변경 추적
CREATE TABLE audit_log (
    log_id SERIAL PRIMARY KEY,
    table_name VARCHAR(50),
    operation VARCHAR(10),
    old_values JSONB,
    new_values JSONB,
    user_name VARCHAR(50),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION audit_changes()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, operation, old_values, new_values, user_name)
    VALUES (
        TG_TABLE_NAME,
        TG_OP,
        CASE WHEN TG_OP = 'DELETE' THEN to_jsonb(OLD) ELSE NULL END,
        CASE WHEN TG_OP IN ('INSERT', 'UPDATE') THEN to_jsonb(NEW) ELSE NULL END,
        current_user
    );
    RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

-- 중요 테이블에 감사 트리거 적용
CREATE TRIGGER tg_audit_employee
    AFTER INSERT OR UPDATE OR DELETE ON Employee
    FOR EACH ROW EXECUTE FUNCTION audit_changes();
```

## 무결성 제약조건의 성능 영향

### 1. 인덱스와의 관계

```sql
-- 제약조건은 자동으로 인덱스 생성
CREATE TABLE Order (
    order_id SERIAL PRIMARY KEY,           -- 자동으로 클러스터드 인덱스
    customer_id INT UNIQUE,                -- 자동으로 유니크 인덱스
    order_number VARCHAR(20) UNIQUE        -- 자동으로 유니크 인덱스
);

-- 외래 키에는 수동으로 인덱스 생성 권장
CREATE INDEX idx_orderitem_product ON OrderItem(product_id);
```

### 2. 대용량 데이터에서의 고려사항

```sql
-- 체크 제약조건 최적화: 함수 대신 단순 비교 사용
-- 비효율적
ALTER TABLE Product ADD CONSTRAINT chk_price_range
    CHECK (calculate_price_category(price) IN ('low', 'medium', 'high'));

-- 효율적
ALTER TABLE Product ADD CONSTRAINT chk_price_range
    CHECK (price >= 0 AND price <= 10000);
```

### 3. 배치 작업 시 제약조건 관리

```sql
-- 대량 데이터 삽입 시 임시로 제약조건 비활성화
ALTER TABLE OrderItem DISABLE TRIGGER ALL;
-- 대량 삽입 수행
COPY OrderItem FROM '/path/to/data.csv' CSV;
-- 제약조건 재활성화 및 검증
ALTER TABLE OrderItem ENABLE TRIGGER ALL;

-- 지연 제약조건 활용
SET CONSTRAINTS ALL DEFERRED;
-- 복잡한 데이터 조작 수행
COMMIT;  -- 트랜잭션 끝에서 모든 제약조건 검사
```

## 무결성 제약조건의 설계 원칙

### 1. 비즈니스 규칙의 우선순위

```sql
-- 핵심 비즈니스 규칙: 데이터베이스 레벨에서 강제
CREATE TABLE Account (
    account_id SERIAL PRIMARY KEY,
    balance DECIMAL(15,2) NOT NULL CHECK (balance >= 0),  -- 잔액은 항상 양수
    account_type VARCHAR(10) CHECK (account_type IN ('CHECKING', 'SAVINGS')),
    status VARCHAR(10) DEFAULT 'ACTIVE' CHECK (status IN ('ACTIVE', 'CLOSED', 'SUSPENDED'))
);

-- 부가적 규칙: 응용 프로그램 레벨에서 처리
-- 예: 복잡한 할인 정책, 개인화된 추천 등
```

### 2. 성능과 무결성의 균형

```sql
-- 트레이드오프 예시: 정규화 vs 성능
-- 정규화된 구조 (무결성 우선)
CREATE TABLE OrderSummary (
    order_id INT PRIMARY KEY,
    total_amount DECIMAL(10,2),
    item_count INT,
    FOREIGN KEY (order_id) REFERENCES Order(order_id)
);

-- 비정규화된 구조 (성능 우선, 무결성은 트리거로 관리)
ALTER TABLE Order ADD COLUMN total_amount DECIMAL(10,2);
ALTER TABLE Order ADD COLUMN item_count INT;

-- 무결성 유지를 위한 트리거
CREATE OR REPLACE FUNCTION update_order_summary()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE Order
    SET
        total_amount = (SELECT SUM(quantity * price) FROM OrderItem WHERE order_id = NEW.order_id),
        item_count = (SELECT COUNT(*) FROM OrderItem WHERE order_id = NEW.order_id)
    WHERE order_id = NEW.order_id;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

## 현대적 무결성 관리 기법

### 1. 마이크로서비스에서의 무결성

```sql
-- 사가 패턴 (Saga Pattern)을 위한 보상 트랜잭션
CREATE TABLE compensation_log (
    log_id SERIAL PRIMARY KEY,
    saga_id UUID,
    step_name VARCHAR(50),
    compensation_data JSONB,
    status VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 이벤트 소싱을 위한 이벤트 저장소
CREATE TABLE event_store (
    event_id SERIAL PRIMARY KEY,
    aggregate_id UUID,
    event_type VARCHAR(50),
    event_data JSONB,
    version INT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT uk_event_version UNIQUE (aggregate_id, version)
);
```

### 2. JSON/NoSQL 데이터의 무결성

```sql
-- JSON 스키마 검증
CREATE TABLE product_catalog (
    product_id SERIAL PRIMARY KEY,
    product_data JSONB NOT NULL,

    -- JSON 스키마 검증
    CONSTRAINT chk_product_schema CHECK (
        jsonb_typeof(product_data->'name') = 'string' AND
        jsonb_typeof(product_data->'price') = 'number' AND
        (product_data->'price')::numeric > 0
    )
);

-- JSON 경로 인덱스
CREATE INDEX idx_product_category ON product_catalog
    USING GIN ((product_data->'category'));
```

### 3. 분산 데이터베이스에서의 무결성

```sql
-- 분산 트랜잭션을 위한 2단계 커밋 상태 테이블
CREATE TABLE distributed_transaction (
    txn_id UUID PRIMARY KEY,
    coordinator VARCHAR(100),
    participants JSONB,
    status VARCHAR(20) CHECK (status IN ('PREPARING', 'PREPARED', 'COMMITTED', 'ABORTED')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 무결성 제약조건 위반 처리

### 1. 오류 메시지 사용자 정의

```sql
-- PostgreSQL 예시
CREATE OR REPLACE FUNCTION custom_constraint_message()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.age < 18 THEN
        RAISE EXCEPTION 'CUSTOM_ERROR: 직원 나이는 18세 이상이어야 합니다. 입력된 나이: %', NEW.age;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 응용 프로그램에서 사용자 친화적 메시지 제공
```

### 2. 제약조건 위반 복구 전략

```sql
-- 자동 복구 메커니즘
CREATE OR REPLACE FUNCTION auto_fix_constraint_violation()
RETURNS TRIGGER AS $$
BEGIN
    -- 예: 재고 부족 시 자동으로 backorder 생성
    IF NEW.quantity > (SELECT stock_quantity FROM Product WHERE product_id = NEW.product_id) THEN
        INSERT INTO BackOrder (customer_id, product_id, quantity)
        VALUES (NEW.customer_id, NEW.product_id, NEW.quantity);

        NEW.quantity = (SELECT stock_quantity FROM Product WHERE product_id = NEW.product_id);
        NEW.status = 'PARTIAL';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

## 결론

무결성 제약조건은 데이터베이스의 데이터 품질과 일관성을 보장하는 핵심 메커니즘입니다. 적절한 제약조건 설계는 시스템의 안정성과 신뢰성을 크게 향상시키지만, 성능과 유연성에 대한 고려도 필요합니다. 현대의 복잡한 시스템에서는 전통적인 제약조건뿐만 아니라 트리거, 저장 프로시저, 응용 프로그램 레벨 검증을 적절히 조합하여 포괄적인 무결성 관리 전략을 수립하는 것이 중요합니다.
