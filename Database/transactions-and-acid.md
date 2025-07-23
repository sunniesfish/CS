# 트랜잭션과 ACID

## 개요

트랜잭션은 데이터베이스에서 하나의 논리적 작업 단위를 나타내며, ACID 속성을 통해 데이터 일관성과 무결성을 보장합니다.

## 트랜잭션의 정의

데이터베이스 상태를 변화시키는 하나의 논리적 기능을 수행하기 위한 작업의 단위입니다.

```sql
-- 계좌 이체 트랜잭션 예시
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
    UPDATE accounts SET balance = balance + 100 WHERE account_id = 'A002';
    INSERT INTO transaction_log VALUES (NOW(), 'A001', 'A002', 100);
COMMIT;
```

## ACID 속성

### 1. 원자성 (Atomicity)

트랜잭션의 모든 연산이 완전히 수행되거나 전혀 수행되지 않아야 합니다.

```sql
-- 성공 시: 모든 작업이 커밋됨
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'A002';
COMMIT; -- 모든 변경사항이 영구적으로 저장됨

-- 실패 시: 모든 작업이 롤백됨
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
-- 오류 발생 (예: 잔액 부족)
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'INVALID';
ROLLBACK; -- 모든 변경사항이 취소됨
```

### 2. 일관성 (Consistency)

트랜잭션이 실행을 성공적으로 완료하면 언제나 일관성 있는 데이터베이스 상태로 변환합니다.

```sql
-- 제약조건을 통한 일관성 보장
CREATE TABLE accounts (
    account_id VARCHAR(10) PRIMARY KEY,
    balance DECIMAL(10,2) CHECK (balance >= 0), -- 잔액은 항상 0 이상
    status VARCHAR(10) CHECK (status IN ('ACTIVE', 'CLOSED'))
);

-- 일관성 위반 시 트랜잭션 실패
BEGIN;
UPDATE accounts SET balance = -50 WHERE account_id = 'A001'; -- 제약조건 위반
ROLLBACK; -- 자동으로 롤백됨
```

### 3. 격리성 (Isolation)

동시에 실행되는 트랜잭션들이 서로에게 영향을 주지 않아야 합니다.

```sql
-- 트랜잭션 격리 수준 설정
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- 트랜잭션 1
BEGIN;
SELECT balance FROM accounts WHERE account_id = 'A001'; -- 1000원
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';

-- 트랜잭션 2 (동시 실행)
BEGIN;
SELECT balance FROM accounts WHERE account_id = 'A001'; -- 여전히 1000원 (격리됨)
```

### 4. 지속성 (Durability)

성공적으로 수행된 트랜잭션은 영원히 반영되어야 합니다.

```sql
-- WAL (Write-Ahead Logging)을 통한 지속성 보장
-- 트랜잭션 로그에 먼저 기록 후 데이터 파일에 반영
BEGIN;
UPDATE accounts SET balance = 500 WHERE account_id = 'A001';
COMMIT; -- 로그에 기록되어 시스템 장애에도 복구 가능
```

## 트랜잭션 상태

### 트랜잭션 생명주기

```
Active → Partially Committed → Committed
  ↓              ↓
Failed ← → Aborted
```

```sql
-- PostgreSQL에서 트랜잭션 상태 확인
SELECT
    pid,
    state,
    query,
    xact_start
FROM pg_stat_activity
WHERE state IN ('active', 'idle in transaction');
```

## 동시성 제어

### 1. 락(Lock) 기반 제어

```sql
-- 배타적 락 (Exclusive Lock)
SELECT * FROM accounts WHERE account_id = 'A001' FOR UPDATE;

-- 공유 락 (Shared Lock)
SELECT * FROM accounts WHERE account_id = 'A001' FOR SHARE;

-- 테이블 락
LOCK TABLE accounts IN ACCESS EXCLUSIVE MODE;
```

### 2. 타임스탬프 기반 제어

```sql
-- 버전 관리를 통한 동시성 제어
CREATE TABLE accounts (
    account_id VARCHAR(10) PRIMARY KEY,
    balance DECIMAL(10,2),
    version INTEGER DEFAULT 1,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 낙관적 동시성 제어
UPDATE accounts
SET balance = 900, version = version + 1, updated_at = CURRENT_TIMESTAMP
WHERE account_id = 'A001' AND version = 1;
```

## 트랜잭션 관리

### 1. 명시적 트랜잭션

```sql
-- PostgreSQL/SQL Server
BEGIN TRANSACTION;
-- 작업 수행
COMMIT; -- 또는 ROLLBACK;

-- MySQL
START TRANSACTION;
-- 작업 수행
COMMIT; -- 또는 ROLLBACK;
```

### 2. 자동 커밋 모드

```sql
-- 자동 커밋 비활성화
SET AUTOCOMMIT = 0; -- MySQL
SET AUTOCOMMIT OFF; -- Oracle

-- 각 문장이 자동으로 트랜잭션이 됨
UPDATE accounts SET balance = 900 WHERE account_id = 'A001'; -- 자동 커밋됨
```

### 3. 세이브포인트 (Savepoint)

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
SAVEPOINT sp1;

UPDATE accounts SET balance = balance + 50 WHERE account_id = 'A002';
SAVEPOINT sp2;

-- 부분 롤백
ROLLBACK TO SAVEPOINT sp1; -- sp2는 취소, sp1까지만 유지

UPDATE accounts SET balance = balance + 100 WHERE account_id = 'A003';
COMMIT; -- 최종 커밋
```

## 분산 트랜잭션

### 2단계 커밋 (2PC - Two-Phase Commit)

```sql
-- Phase 1: Prepare
PREPARE TRANSACTION 'tx_001';

-- Phase 2: Commit/Abort
COMMIT PREPARED 'tx_001';
-- 또는
ROLLBACK PREPARED 'tx_001';
```

### XA 트랜잭션

```java
// Java XA 예시
XADataSource xaDs1 = new MysqlXADataSource();
XADataSource xaDs2 = new PostgresXADataSource();

XAConnection xaCon1 = xaDs1.getXAConnection();
XAConnection xaCon2 = xaDs2.getXAConnection();

XAResource xaRes1 = xaCon1.getXAResource();
XAResource xaRes2 = xaCon2.getXAResource();

Xid xid = new MyXid();

// Phase 1
xaRes1.start(xid, XAResource.TMNOFLAGS);
xaRes2.start(xid, XAResource.TMNOFLAGS);

// 작업 수행
// ...

xaRes1.end(xid, XAResource.TMSUCCESS);
xaRes2.end(xid, XAResource.TMSUCCESS);

int prepare1 = xaRes1.prepare(xid);
int prepare2 = xaRes2.prepare(xid);

// Phase 2
if (prepare1 == XA_OK && prepare2 == XA_OK) {
    xaRes1.commit(xid, false);
    xaRes2.commit(xid, false);
}
```

## 성능 최적화

### 1. 트랜잭션 크기 최적화

```sql
-- 나쁜 예: 너무 큰 트랜잭션
BEGIN;
UPDATE large_table SET status = 'PROCESSED'; -- 백만 행 업데이트
COMMIT;

-- 좋은 예: 배치 처리
DO $$
DECLARE
    batch_size INTEGER := 1000;
    affected_rows INTEGER;
BEGIN
    LOOP
        UPDATE large_table SET status = 'PROCESSED'
        WHERE status = 'PENDING'
        AND ctid IN (
            SELECT ctid FROM large_table
            WHERE status = 'PENDING'
            LIMIT batch_size
        );

        GET DIAGNOSTICS affected_rows = ROW_COUNT;
        EXIT WHEN affected_rows = 0;

        COMMIT;
    END LOOP;
END $$;
```

### 2. 락 최소화

```sql
-- 락 시간 최소화
BEGIN;
-- 1. 읽기 작업 먼저
SELECT balance FROM accounts WHERE account_id = 'A001';

-- 2. 계산 수행 (락 없이)
-- calculate new_balance

-- 3. 업데이트는 가능한 늦게
UPDATE accounts SET balance = new_balance WHERE account_id = 'A001';
COMMIT;
```

## 모니터링

### 트랜잭션 상태 모니터링

```sql
-- PostgreSQL
SELECT
    pid,
    state,
    query_start,
    now() - query_start as duration,
    query
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY query_start;

-- 장기 실행 트랜잭션 확인
SELECT
    pid,
    now() - xact_start as transaction_duration,
    query
FROM pg_stat_activity
WHERE xact_start IS NOT NULL
AND now() - xact_start > interval '5 minutes';
```

### 락 모니터링

```sql
-- 현재 락 상태 확인
SELECT
    l.locktype,
    l.database,
    l.relation,
    l.mode,
    l.granted,
    a.query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE NOT l.granted;
```

## 결론

트랜잭션과 ACID 속성은 데이터베이스의 신뢰성을 보장하는 핵심 개념입니다. 적절한 트랜잭션 설계와 동시성 제어를 통해 데이터 일관성을 유지하면서도 높은 성능을 달성할 수 있습니다.
