# 정규화 (1NF, 2NF, 3NF, BCNF)

## 개요

정규화(Normalization)는 관계형 데이터베이스 설계에서 데이터 중복을 최소화하고 데이터 무결성을 최대화하기 위해 테이블을 분해하는 과정입니다. Edgar F. Codd가 제시한 이론으로, 함수적 종속성을 기반으로 합니다.

## 정규화의 목적과 배경

### 탄생 배경

초기 데이터베이스 설계에서 발생하는 문제들:

- **데이터 중복**: 동일한 정보가 여러 곳에 저장
- **갱신 이상(Update Anomaly)**: 중복된 데이터 중 일부만 수정되어 일관성 깨짐
- **삽입 이상(Insert Anomaly)**: 불필요한 정보를 함께 저장해야 하는 문제
- **삭제 이상(Delete Anomaly)**: 필요한 정보까지 함께 삭제되는 문제

### 정규화의 목적

1. **데이터 무결성 보장**: 일관성 있는 데이터 유지
2. **저장 공간 최적화**: 중복 제거로 공간 절약
3. **유지보수성 향상**: 구조 변경의 영향 최소화
4. **이상 현상 제거**: 삽입, 갱신, 삭제 이상 방지

## 함수적 종속성 (Functional Dependency)

### 기본 개념

X → Y (X가 Y를 함수적으로 결정한다)

- X 값이 결정되면 Y 값이 유일하게 결정됨
- X를 결정자(Determinant), Y를 종속자(Dependent)라고 함

### 함수적 종속성의 유형

```sql
-- 예시 테이블: Student
CREATE TABLE Student_Info (
    student_id VARCHAR(10),
    student_name VARCHAR(50),
    birth_date DATE,
    advisor_id VARCHAR(10),
    advisor_name VARCHAR(50),
    department_id VARCHAR(10),
    department_name VARCHAR(50),
    course_id VARCHAR(10),
    course_name VARCHAR(50),
    grade CHAR(2)
);

-- 함수적 종속성 예시:
-- student_id → student_name, birth_date (완전 함수 종속)
-- advisor_id → advisor_name, department_id (완전 함수 종속)
-- department_id → department_name (완전 함수 종속)
-- course_id → course_name (완전 함수 종속)
-- (student_id, course_id) → grade (완전 함수 종속)

-- 부분 함수 종속 (문제가 되는 경우):
-- (student_id, course_id) → student_name (부분 종속 - student_id만으로도 결정됨)
-- (student_id, course_id) → course_name (부분 종속 - course_id만으로도 결정됨)
```

### 종속성의 종류

#### 1. 완전 함수 종속 (Full Functional Dependency)

```sql
-- 복합 키의 모든 구성 요소가 필요한 경우
-- (student_id, course_id) → grade
-- student_id 또는 course_id 하나만으로는 grade를 결정할 수 없음
```

#### 2. 부분 함수 종속 (Partial Functional Dependency)

```sql
-- 복합 키의 일부만으로도 결정되는 경우
-- (student_id, course_id) → student_name
-- student_id만으로도 student_name이 결정됨
```

#### 3. 이행적 함수 종속 (Transitive Functional Dependency)

```sql
-- X → Y, Y → Z일 때 X → Z인 경우
-- student_id → advisor_id → advisor_name
-- 즉, student_id → advisor_name (이행적 종속)
```

## 제1정규형 (1NF - First Normal Form)

### 정의

- 모든 속성 값이 원자값(Atomic Value)이어야 함
- 반복 그룹이나 다중값 속성이 없어야 함

### 1NF 위반 예시

```sql
-- 1NF 위반 테이블
CREATE TABLE Student_Courses_Violation (
    student_id VARCHAR(10),
    student_name VARCHAR(50),
    courses VARCHAR(200),  -- 'Math,Physics,Chemistry' (다중값)
    grades VARCHAR(50)     -- 'A,B+,A-' (다중값)
);

-- 데이터 예시
INSERT INTO Student_Courses_Violation VALUES
('S001', 'John', 'Math,Physics,Chemistry', 'A,B+,A-'),
('S002', 'Jane', 'Biology,Chemistry', 'A+,A');
```

### 1NF로 변환

```sql
-- 1NF 준수 테이블
CREATE TABLE Student_Courses_1NF (
    student_id VARCHAR(10),
    student_name VARCHAR(50),
    course VARCHAR(50),
    grade CHAR(2),
    PRIMARY KEY (student_id, course)
);

-- 정규화된 데이터
INSERT INTO Student_Courses_1NF VALUES
('S001', 'John', 'Math', 'A'),
('S001', 'John', 'Physics', 'B+'),
('S001', 'John', 'Chemistry', 'A-'),
('S002', 'Jane', 'Biology', 'A+'),
('S002', 'Jane', 'Chemistry', 'A');
```

### 1NF의 문제점

```sql
-- 여전히 존재하는 문제들:
-- 1. 데이터 중복: student_name이 반복됨
-- 2. 갱신 이상: John의 이름을 바꾸려면 여러 행을 수정해야 함
-- 3. 삽입 이상: 수강하지 않은 학생은 등록할 수 없음
-- 4. 삭제 이상: 모든 과목을 취소하면 학생 정보도 사라짐
```

## 제2정규형 (2NF - Second Normal Form)

### 정의

- 1NF를 만족하면서
- 모든 비주요 속성이 기본 키에 대해 완전 함수 종속이어야 함
- 즉, 부분 함수 종속이 없어야 함

### 2NF 위반 분석

```sql
-- 1NF 테이블 (2NF 위반)
-- 기본 키: (student_id, course)
-- 부분 함수 종속:
-- student_id → student_name (course 없이도 결정됨)

-- 문제 상황 시뮬레이션
UPDATE Student_Courses_1NF
SET student_name = 'John Smith'
WHERE student_id = 'S001';
-- 3개 행을 모두 수정해야 함 (갱신 이상)
```

### 2NF로 변환

```sql
-- 학생 테이블 (주제별 분리)
CREATE TABLE Students (
    student_id VARCHAR(10) PRIMARY KEY,
    student_name VARCHAR(50) NOT NULL
);

-- 수강 테이블
CREATE TABLE Enrollments (
    student_id VARCHAR(10),
    course VARCHAR(50),
    grade CHAR(2),
    PRIMARY KEY (student_id, course),
    FOREIGN KEY (student_id) REFERENCES Students(student_id)
);

-- 데이터 삽입
INSERT INTO Students VALUES
('S001', 'John'),
('S002', 'Jane');

INSERT INTO Enrollments VALUES
('S001', 'Math', 'A'),
('S001', 'Physics', 'B+'),
('S001', 'Chemistry', 'A-'),
('S002', 'Biology', 'A+'),
('S002', 'Chemistry', 'A');
```

### 2NF의 장점과 한계

```sql
-- 장점:
-- 1. 갱신 이상 해결: 학생 이름 변경이 한 곳에서만 발생
UPDATE Students SET student_name = 'John Smith' WHERE student_id = 'S001';

-- 2. 삽입 이상 해결: 수강 없이도 학생 등록 가능
INSERT INTO Students VALUES ('S003', 'Bob');

-- 여전한 한계:
-- 이행적 종속성이 있다면 여전히 이상 현상 발생 가능
```

## 제3정규형 (3NF - Third Normal Form)

### 정의

- 2NF를 만족하면서
- 모든 비주요 속성이 기본 키에 대해 이행적으로 종속되지 않아야 함

### 3NF 위반 예시

```sql
-- 2NF이지만 3NF 위반인 테이블
CREATE TABLE Students_2NF (
    student_id VARCHAR(10) PRIMARY KEY,
    student_name VARCHAR(50),
    advisor_id VARCHAR(10),
    advisor_name VARCHAR(50),
    department VARCHAR(50)
);

-- 이행적 종속성:
-- student_id → advisor_id → advisor_name
-- student_id → advisor_id → department
-- 즉, student_id → advisor_name, department (이행적 종속)

-- 문제 상황
INSERT INTO Students_2NF VALUES
('S001', 'John', 'A001', 'Dr. Smith', 'Computer Science'),
('S002', 'Jane', 'A001', 'Dr. Smith', 'Computer Science'),
('S003', 'Bob', 'A002', 'Dr. Jones', 'Mathematics');

-- 갱신 이상: 지도교수 이름 변경시
UPDATE Students_2NF
SET advisor_name = 'Dr. John Smith'
WHERE advisor_id = 'A001';
-- 여러 행을 수정해야 함
```

### 3NF로 변환

```sql
-- 학생 테이블
CREATE TABLE Students_3NF (
    student_id VARCHAR(10) PRIMARY KEY,
    student_name VARCHAR(50),
    advisor_id VARCHAR(10),
    FOREIGN KEY (advisor_id) REFERENCES Advisors(advisor_id)
);

-- 지도교수 테이블
CREATE TABLE Advisors (
    advisor_id VARCHAR(10) PRIMARY KEY,
    advisor_name VARCHAR(50),
    department VARCHAR(50)
);

-- 데이터 삽입
INSERT INTO Advisors VALUES
('A001', 'Dr. Smith', 'Computer Science'),
('A002', 'Dr. Jones', 'Mathematics');

INSERT INTO Students_3NF VALUES
('S001', 'John', 'A001'),
('S002', 'Jane', 'A001'),
('S003', 'Bob', 'A002');
```

### 3NF 조인 예시

```sql
-- 학생과 지도교수 정보 조회
SELECT
    s.student_id,
    s.student_name,
    a.advisor_name,
    a.department
FROM Students_3NF s
JOIN Advisors a ON s.advisor_id = a.advisor_id;
```

## BCNF (Boyce-Codd Normal Form)

### 정의

- 3NF의 강화된 형태
- 모든 결정자가 후보 키여야 함
- 3NF에서 놓칠 수 있는 예외 상황을 해결

### BCNF 위반 예시

```sql
-- 3NF이지만 BCNF 위반인 테이블
CREATE TABLE Course_Instructor (
    course_id VARCHAR(10),
    instructor_id VARCHAR(10),
    instructor_name VARCHAR(50),
    classroom VARCHAR(20),
    PRIMARY KEY (course_id, instructor_id)
);

-- 함수적 종속성:
-- (course_id, instructor_id) → instructor_name, classroom
-- instructor_id → instructor_name (결정자이지만 후보 키가 아님)

-- 문제 상황
INSERT INTO Course_Instructor VALUES
('CS101', 'I001', 'Dr. Brown', 'Room 101'),
('CS102', 'I001', 'Dr. Brown', 'Room 102'),
('CS101', 'I002', 'Dr. White', 'Room 103');

-- 갱신 이상: 교수 이름 변경
UPDATE Course_Instructor
SET instructor_name = 'Dr. John Brown'
WHERE instructor_id = 'I001';
-- 여러 행 수정 필요
```

### BCNF로 변환

```sql
-- 교수 테이블
CREATE TABLE Instructors (
    instructor_id VARCHAR(10) PRIMARY KEY,
    instructor_name VARCHAR(50)
);

-- 과목-교수 배정 테이블
CREATE TABLE Course_Assignments (
    course_id VARCHAR(10),
    instructor_id VARCHAR(10),
    classroom VARCHAR(20),
    PRIMARY KEY (course_id, instructor_id),
    FOREIGN KEY (instructor_id) REFERENCES Instructors(instructor_id)
);

-- 데이터 삽입
INSERT INTO Instructors VALUES
('I001', 'Dr. Brown'),
('I002', 'Dr. White');

INSERT INTO Course_Assignments VALUES
('CS101', 'I001', 'Room 101'),
('CS102', 'I001', 'Room 102'),
('CS101', 'I002', 'Room 103');
```

## 정규화 과정의 실제 예시

### 비정규화 테이블 (0NF)

```sql
CREATE TABLE University_Records (
    student_id VARCHAR(10),
    student_name VARCHAR(50),
    student_email VARCHAR(100),
    courses_and_grades TEXT,  -- 'Math:A,Physics:B+,Chemistry:A-'
    advisor_id VARCHAR(10),
    advisor_name VARCHAR(50),
    advisor_phone VARCHAR(20),
    department_id VARCHAR(10),
    department_name VARCHAR(50),
    department_building VARCHAR(50)
);
```

### 1NF 적용

```sql
CREATE TABLE Records_1NF (
    student_id VARCHAR(10),
    student_name VARCHAR(50),
    student_email VARCHAR(100),
    course_name VARCHAR(50),
    grade CHAR(2),
    advisor_id VARCHAR(10),
    advisor_name VARCHAR(50),
    advisor_phone VARCHAR(20),
    department_id VARCHAR(10),
    department_name VARCHAR(50),
    department_building VARCHAR(50),
    PRIMARY KEY (student_id, course_name)
);
```

### 2NF 적용

```sql
-- 학생 정보
CREATE TABLE Students_2NF (
    student_id VARCHAR(10) PRIMARY KEY,
    student_name VARCHAR(50),
    student_email VARCHAR(100),
    advisor_id VARCHAR(10),
    advisor_name VARCHAR(50),
    advisor_phone VARCHAR(20),
    department_id VARCHAR(10),
    department_name VARCHAR(50),
    department_building VARCHAR(50)
);

-- 성적 정보
CREATE TABLE Grades_2NF (
    student_id VARCHAR(10),
    course_name VARCHAR(50),
    grade CHAR(2),
    PRIMARY KEY (student_id, course_name),
    FOREIGN KEY (student_id) REFERENCES Students_2NF(student_id)
);
```

### 3NF 적용

```sql
-- 학생
CREATE TABLE Students_3NF (
    student_id VARCHAR(10) PRIMARY KEY,
    student_name VARCHAR(50),
    student_email VARCHAR(100),
    advisor_id VARCHAR(10)
);

-- 지도교수
CREATE TABLE Advisors_3NF (
    advisor_id VARCHAR(10) PRIMARY KEY,
    advisor_name VARCHAR(50),
    advisor_phone VARCHAR(20),
    department_id VARCHAR(10)
);

-- 학과
CREATE TABLE Departments_3NF (
    department_id VARCHAR(10) PRIMARY KEY,
    department_name VARCHAR(50),
    department_building VARCHAR(50)
);

-- 성적
CREATE TABLE Grades_3NF (
    student_id VARCHAR(10),
    course_name VARCHAR(50),
    grade CHAR(2),
    PRIMARY KEY (student_id, course_name),
    FOREIGN KEY (student_id) REFERENCES Students_3NF(student_id)
);

-- 외래 키 제약조건
ALTER TABLE Students_3NF
ADD FOREIGN KEY (advisor_id) REFERENCES Advisors_3NF(advisor_id);

ALTER TABLE Advisors_3NF
ADD FOREIGN KEY (department_id) REFERENCES Departments_3NF(department_id);
```

## 정규화의 실용적 고려사항

### 1. 성능과의 트레이드오프

```sql
-- 3NF 조인 쿼리 (복잡하지만 정규화됨)
SELECT
    s.student_name,
    s.student_email,
    a.advisor_name,
    d.department_name,
    g.course_name,
    g.grade
FROM Students_3NF s
JOIN Advisors_3NF a ON s.advisor_id = a.advisor_id
JOIN Departments_3NF d ON a.department_id = d.department_id
JOIN Grades_3NF g ON s.student_id = g.student_id
WHERE s.student_id = 'S001';

-- 비정규화된 테이블 쿼리 (단순하지만 중복 존재)
SELECT * FROM Records_1NF WHERE student_id = 'S001';
```

### 2. 정규화 수준 결정 기준

```sql
-- 읽기 중심 애플리케이션: 적당한 비정규화 고려
CREATE TABLE Student_Summary (
    student_id VARCHAR(10) PRIMARY KEY,
    student_name VARCHAR(50),
    advisor_name VARCHAR(50),  -- 비정규화
    department_name VARCHAR(50),  -- 비정규화
    total_courses INT,
    gpa DECIMAL(3,2)
);

-- 쓰기 중심 애플리케이션: 높은 수준의 정규화 유지
-- 위의 3NF 구조 유지
```

### 3. 인덱스 전략

```sql
-- 정규화된 테이블의 인덱스
CREATE INDEX idx_students_advisor ON Students_3NF(advisor_id);
CREATE INDEX idx_advisors_department ON Advisors_3NF(department_id);
CREATE INDEX idx_grades_student ON Grades_3NF(student_id);
CREATE INDEX idx_grades_course ON Grades_3NF(course_name);

-- 복합 인덱스 (조인 성능 향상)
CREATE INDEX idx_grades_student_course ON Grades_3NF(student_id, course_name);
```

## 정규화 검증

### 1. 함수적 종속성 확인

```sql
-- 현재 테이블의 함수적 종속성 검사
SELECT
    student_id,
    COUNT(DISTINCT student_name) as name_count,
    COUNT(DISTINCT advisor_id) as advisor_count
FROM Students_3NF
GROUP BY student_id
HAVING COUNT(DISTINCT student_name) > 1
    OR COUNT(DISTINCT advisor_id) > 1;
-- 결과가 없어야 함 (함수적 종속성 만족)
```

### 2. 이상 현상 테스트

```sql
-- 갱신 이상 테스트
-- 3NF: 한 곳만 수정
UPDATE Advisors_3NF
SET advisor_name = 'Dr. John Smith'
WHERE advisor_id = 'A001';

-- 삽입 이상 테스트
-- 3NF: 독립적 삽입 가능
INSERT INTO Students_3NF VALUES ('S004', 'Alice', 'alice@email.com', NULL);

-- 삭제 이상 테스트
-- 3NF: 안전한 삭제
DELETE FROM Grades_3NF WHERE student_id = 'S001';
-- 학생 정보는 Students_3NF에 그대로 유지됨
```

## 비정규화 (Denormalization)

### 언제 비정규화를 고려할까?

```sql
-- 1. 자주 조인되는 테이블들
CREATE TABLE Student_Extended (
    student_id VARCHAR(10) PRIMARY KEY,
    student_name VARCHAR(50),
    advisor_id VARCHAR(10),
    advisor_name VARCHAR(50),  -- 비정규화
    department_name VARCHAR(50)  -- 비정규화
);

-- 2. 집계 데이터 저장
CREATE TABLE Department_Stats (
    department_id VARCHAR(10) PRIMARY KEY,
    department_name VARCHAR(50),
    total_students INT,  -- 비정규화된 집계
    total_advisors INT,  -- 비정규화된 집계
    avg_gpa DECIMAL(3,2)  -- 비정규화된 집계
);

-- 3. 트리거로 일관성 유지
CREATE OR REPLACE FUNCTION update_dept_stats()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE Department_Stats
    SET total_students = (
        SELECT COUNT(*) FROM Students_3NF s
        JOIN Advisors_3NF a ON s.advisor_id = a.advisor_id
        WHERE a.department_id = NEW.department_id
    )
    WHERE department_id = NEW.department_id;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

## 현대적 접근: 다형성 테이블

### JSON을 활용한 유연한 설계

```sql
-- 하이브리드 접근: 핵심은 정규화, 확장 정보는 JSON
CREATE TABLE Students_Modern (
    student_id VARCHAR(10) PRIMARY KEY,
    student_name VARCHAR(50),
    advisor_id VARCHAR(10),
    metadata JSONB,  -- 유연한 확장 정보

    FOREIGN KEY (advisor_id) REFERENCES Advisors(advisor_id)
);

-- JSON 데이터 예시
INSERT INTO Students_Modern VALUES (
    'S001',
    'John',
    'A001',
    '{"email": "john@email.com", "phone": "555-1234", "interests": ["AI", "Database"]}'
);

-- JSON 쿼리
SELECT
    student_name,
    metadata->>'email' as email,
    jsonb_array_elements_text(metadata->'interests') as interest
FROM Students_Modern
WHERE student_id = 'S001';
```

## 마이크로서비스와 정규화

### 서비스별 데이터 분할

```sql
-- Student Service Database
CREATE TABLE Students (
    student_id VARCHAR(10) PRIMARY KEY,
    student_name VARCHAR(50),
    email VARCHAR(100)
);

-- Academic Service Database
CREATE TABLE Enrollments (
    student_id VARCHAR(10),
    course_id VARCHAR(10),
    grade CHAR(2),
    PRIMARY KEY (student_id, course_id)
);

-- Faculty Service Database
CREATE TABLE Advisors (
    advisor_id VARCHAR(10) PRIMARY KEY,
    advisor_name VARCHAR(50),
    department_id VARCHAR(10)
);
```

## 결론

정규화는 데이터 무결성과 일관성을 보장하는 중요한 설계 원칙입니다. 하지만 실제 시스템에서는 성능, 복잡성, 비용 등을 종합적으로 고려하여 적절한 수준의 정규화를 적용해야 합니다. 3NF가 일반적으로 권장되는 수준이며, 특별한 경우에만 BCNF나 비정규화를 고려합니다. 현대적 접근에서는 핵심 관계는 정규화하고, 확장성을 위해 NoSQL 스타일의 유연한 구조를 병행하는 하이브리드 접근이 널리 사용됩니다.
