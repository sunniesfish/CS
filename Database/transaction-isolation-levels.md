# 트랜잭션 격리 수준과 현상

## 개요

트랜잭션 격리 수준(Transaction Isolation Level)은 동시에 실행되는 트랜잭션들 사이의 상호작용을 제어하는 메커니즘입니다. ACID의 Isolation을 구현하는 방법으로, 데이터 일관성과 성능 사이의 균형을 조절합니다.

## 동시성 문제 현상

### 1. Dirty Read (더티 리드)

다른 트랜잭션에서 커밋되지 않은 데이터를 읽는 현상

```sql
-- Transaction A
BEGIN;
UPDATE accounts SET balance = 1000 WHERE account_id = 'A001';
-- 아직 커밋하지 않음

-- Transaction B (다른 세션에서)
BEGIN;
SELECT balance FROM accounts WHERE account_id = 'A001'; -- 1000 읽음 (Dirty Read)

-- Transaction A (롤백)
ROLLBACK; -- balance는 실제로 원래 값으로 돌아감

-- Transaction B는 존재하지 않았던 값을 읽은 것
```

### 2. Non-Repeatable Read (반복 불가능 읽기)

같은 트랜잭션 내에서 같은 쿼리를 두 번 실행했을 때 다른 결과가 나오는 현상

```sql
-- Transaction A
BEGIN;
SELECT balance FROM accounts WHERE account_id = 'A001'; -- 500 읽음

-- Transaction B (다른 세션에서)
BEGIN;
UPDATE accounts SET balance = 1000 WHERE account_id = 'A001';
COMMIT;

-- Transaction A (계속)
SELECT balance FROM accounts WHERE account_id = 'A001'; -- 1000 읽음 (다른 결과!)
COMMIT;
```

### 3. Phantom Read (팬텀 리드)

같은 조건의 쿼리를 두 번 실행했을 때 첫 번째에는 없던 행이 두 번째에는 나타나는 현상

```sql
-- Transaction A
BEGIN;
SELECT COUNT(*) FROM accounts WHERE balance > 1000; -- 5개 행

-- Transaction B (다른 세션에서)
BEGIN;
INSERT INTO accounts VALUES ('A999', 1500); -- 새로운 고액 계좌
COMMIT;

-- Transaction A (계속)
SELECT COUNT(*) FROM accounts WHERE balance > 1000; -- 6개 행 (Phantom Read)
COMMIT;
```

### 4. Lost Update (갱신 손실)

두 트랜잭션이 같은 데이터를 동시에 수정할 때 하나의 갱신이 손실되는 현상

```sql
-- Transaction A
BEGIN;
SELECT balance FROM accounts WHERE account_id = 'A001'; -- 1000
-- balance에 100 더하기 계획

-- Transaction B (동시에)
BEGIN;
SELECT balance FROM accounts WHERE account_id = 'A001'; -- 1000
UPDATE accounts SET balance = 800 WHERE account_id = 'A001'; -- 200 차감
COMMIT;

-- Transaction A (계속)
UPDATE accounts SET balance = 1100 WHERE account_id = 'A001'; -- 1000 + 100
COMMIT;
-- 결과: 1100 (Transaction B의 -200은 손실됨)
```

## SQL 표준 격리 수준

### 1. READ UNCOMMITTED (레벨 0)

가장 낮은 격리 수준으로, 커밋되지 않은 데이터도 읽을 수 있습니다.

```sql
-- 설정
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- 특징
-- ✗ Dirty Read 발생 가능
-- ✗ Non-Repeatable Read 발생 가능
-- ✗ Phantom Read 발생 가능
-- ✓ 최고 성능 (락 없이 읽기)

-- 예시
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT * FROM accounts; -- 다른 트랜잭션의 미커밋 데이터도 읽음
COMMIT;
```

### 2. READ COMMITTED (레벨 1)

커밋된 데이터만 읽을 수 있는 격리 수준 (대부분 DBMS의 기본값)

```sql
-- 설정
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 특징
-- ✓ Dirty Read 방지
-- ✗ Non-Repeatable Read 발생 가능
-- ✗ Phantom Read 발생 가능

-- PostgreSQL 예시
BEGIN;
SELECT balance FROM accounts WHERE account_id = 'A001'; -- 커밋된 데이터만 읽음

-- 다른 세션에서 UPDATE 후 COMMIT

SELECT balance FROM accounts WHERE account_id = 'A001'; -- 새로운 값 읽음 (Non-Repeatable)
COMMIT;
```

### 3. REPEATABLE READ (레벨 2)

트랜잭션 시작 시점의 스냅샷을 유지하여 반복 읽기를 보장

```sql
-- 설정
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- 특징
-- ✓ Dirty Read 방지
-- ✓ Non-Repeatable Read 방지
-- ✗ Phantom Read 발생 가능 (표준에서는, 구현마다 다름)

-- MySQL InnoDB 예시
BEGIN;
SELECT * FROM accounts WHERE balance > 1000; -- 스냅샷 기준으로 읽기

-- 다른 세션에서 INSERT/UPDATE/DELETE 후 COMMIT

SELECT * FROM accounts WHERE balance > 1000; -- 여전히 같은 결과 (스냅샷)
COMMIT;
```

### 4. SERIALIZABLE (레벨 3)

가장 높은 격리 수준으로, 트랜잭션들이 순차적으로 실행되는 것과 동일한 결과 보장

```sql
-- 설정
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- 특징
-- ✓ Dirty Read 방지
-- ✓ Non-Repeatable Read 방지
-- ✓ Phantom Read 방지
-- ✗ 성능 저하, 데드락 가능성 증가

-- 예시
BEGIN;
SELECT SUM(balance) FROM accounts; -- 범위 락 설정

-- 다른 세션의 INSERT/UPDATE는 이 트랜잭션이 끝날 때까지 대기
COMMIT;
```

## DBMS별 구현 차이

### PostgreSQL

```sql
-- PostgreSQL의 MVCC (Multi-Version Concurrency Control)
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- 스냅샷 시점 확인
SELECT txid_current_snapshot();

-- 다른 세션의 변경사항은 보이지 않음
SELECT * FROM accounts;
COMMIT;

-- 시리얼라이저블 충돌 감지
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM accounts WHERE balance > 1000;
-- 다른 트랜잭션과 충돌 시 serialization failure 오류 발생
```

### MySQL InnoDB

```sql
-- InnoDB의 락 기반 격리
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Gap Lock과 Next-Key Lock으로 Phantom Read 방지
SELECT * FROM accounts WHERE id BETWEEN 10 AND 20;

-- FOR UPDATE를 사용한 명시적 락
SELECT * FROM accounts WHERE account_id = 'A001' FOR UPDATE;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
COMMIT;
```

### SQL Server

```sql
-- SQL Server의 격리 수준
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
-- 또는
SELECT * FROM accounts WITH (NOLOCK);

-- 스냅샷 격리 (낙관적 동시성)
ALTER DATABASE mydb SET ALLOW_SNAPSHOT_ISOLATION ON;
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;

BEGIN TRANSACTION;
SELECT * FROM accounts WHERE account_id = 'A001';
-- 다른 트랜잭션이 수정해도 스냅샷 버전을 읽음
COMMIT;
```

### Oracle

```sql
-- Oracle의 읽기 일관성
-- READ COMMITTED가 기본값이지만 문장 수준 읽기 일관성 제공

-- 명시적 격리 수준 설정
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- 플래시백 쿼리로 특정 시점 읽기
SELECT * FROM accounts AS OF TIMESTAMP (SYSTIMESTAMP - INTERVAL '5' MINUTE);
```

## 실제 구현 메커니즘

### 1. 락 기반 동시성 제어

```sql
-- 공유 락 (Shared Lock)
SELECT * FROM accounts WHERE account_id = 'A001' LOCK IN SHARE MODE;

-- 배타 락 (Exclusive Lock)
SELECT * FROM accounts WHERE account_id = 'A001' FOR UPDATE;

-- 의도 락 (Intent Lock)
-- 테이블 수준에서 행 수준 락의 존재를 알려줌
-- IS (Intent Shared), IX (Intent Exclusive), SIX (Shared Intent Exclusive)
```

### 2. MVCC (Multi-Version Concurrency Control)

```python
# MVCC 개념적 구현
class MVCCStorage:
    def __init__(self):
        self.versions = {}  # key -> [(transaction_id, value, timestamp)]

    def read(self, key, transaction_id, start_time):
        versions = self.versions.get(key, [])

        # 트랜잭션 시작 시점 이전에 커밋된 가장 최근 버전 찾기
        valid_versions = [
            v for v in versions
            if v[2] <= start_time and v[0] != transaction_id
        ]

        if valid_versions:
            return max(valid_versions, key=lambda x: x[2])[1]
        return None

    def write(self, key, value, transaction_id, timestamp):
        if key not in self.versions:
            self.versions[key] = []
        self.versions[key].append((transaction_id, value, timestamp))
```

### 3. 타임스탬프 정렬

```sql
-- PostgreSQL의 타임스탬프 기반 순서 확인
SELECT
    xact_start,
    now() - xact_start as duration,
    query,
    state
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY xact_start;
```

## 성능과 격리 수준

### 벤치마크 예시

```sql
-- 동시 트랜잭션 수에 따른 성능 측정

-- READ UNCOMMITTED (가장 빠름)
-- TPS: 1000, 평균 응답시간: 10ms, 데이터 정확성: 낮음

-- READ COMMITTED (균형)
-- TPS: 800, 평균 응답시간: 15ms, 데이터 정확성: 보통

-- REPEATABLE READ (안전)
-- TPS: 600, 평균 응답시간: 25ms, 데이터 정확성: 높음

-- SERIALIZABLE (가장 안전하지만 느림)
-- TPS: 300, 평균 응답시간: 50ms, 데이터 정확성: 최고
```

### 격리 수준별 락 경합

```sql
-- 락 대기 상황 모니터링
-- PostgreSQL
SELECT
    blocked_locks.pid AS blocked_pid,
    blocked_activity.query AS blocked_statement,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.query AS blocking_statement
FROM pg_locks blocked_locks
JOIN pg_stat_activity blocked_activity ON blocked_locks.pid = blocked_activity.pid
JOIN pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
JOIN pg_stat_activity blocking_activity ON blocking_locks.pid = blocking_activity.pid
WHERE NOT blocked_locks.granted;
```

## 실무 적용 가이드

### 1. 격리 수준 선택 기준

```python
def choose_isolation_level(use_case):
    if use_case.requires_exact_consistency:
        return "SERIALIZABLE"    # 금융, 회계
    elif use_case.requires_repeatable_reads:
        return "REPEATABLE READ" # 보고서, 분석
    elif use_case.allows_slight_inconsistency:
        return "READ COMMITTED"  # 일반적인 웹 애플리케이션
    elif use_case.prioritizes_performance:
        return "READ UNCOMMITTED" # 실시간 모니터링, 로그 분석
```

### 2. 애플리케이션별 설정

```sql
-- 온라인 거래 시스템
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN;
-- 계좌 잔액 확인
-- 거래 처리
-- 감사 로그 기록
COMMIT;

-- 실시간 대시보드
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT COUNT(*), AVG(amount), MAX(timestamp) FROM transactions;

-- 배치 처리 시스템
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN;
-- 대량 데이터 처리
-- 일관성 검증
COMMIT;
```

### 3. 하이브리드 접근

```python
class TransactionManager:
    def __init__(self):
        self.read_db = ReadOnlyDatabase()   # READ UNCOMMITTED
        self.write_db = TransactionalDatabase()  # REPEATABLE READ

    def read_dashboard_data(self):
        # 빠른 읽기, 약간의 불일치 허용
        return self.read_db.query("SELECT * FROM dashboard_stats")

    def process_payment(self, payment_data):
        # 강한 일관성 필요
        with self.write_db.transaction(isolation="SERIALIZABLE"):
            # 결제 처리
            pass
```

## 모니터링과 튜닝

### 격리 수준 관련 메트릭

```sql
-- 데드락 발생 빈도 확인
SELECT deadlock_count FROM pg_stat_database WHERE datname = 'mydb';

-- 트랜잭션 대기 시간 분석
SELECT
    waiting_queries.query AS waiting_query,
    blocking_queries.query AS blocking_query,
    now() - waiting_activity.query_start AS waiting_duration
FROM pg_locks waiting
JOIN pg_locks blocking ON waiting.locktype = blocking.locktype
JOIN pg_stat_activity waiting_activity ON waiting.pid = waiting_activity.pid
JOIN pg_stat_activity blocking_activity ON blocking.pid = blocking_activity.pid
WHERE NOT waiting.granted AND blocking.granted;
```

## 결론

트랜잭션 격리 수준은 데이터 일관성과 시스템 성능 사이의 핵심적인 트레이드오프입니다. 애플리케이션의 요구사항에 따라 적절한 격리 수준을 선택하고, MVCC나 락 메커니즘의 특성을 이해하여 최적의 동시성을 달성해야 합니다. 현실적으로는 READ COMMITTED가 가장 많이 사용되며, 특별한 일관성 요구사항이 있는 경우에만 더 높은 격리 수준을 적용합니다.
