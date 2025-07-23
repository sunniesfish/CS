# 동시성 제어 (Lock, MVCC 등)

## 개요

동시성 제어(Concurrency Control)는 다중 사용자 환경에서 동시에 실행되는 트랜잭션들이 데이터베이스의 일관성을 해치지 않으면서 최대한의 동시성을 제공하는 메커니즘입니다.

## 동시성 제어의 필요성

### 동시 실행으로 인한 문제점

```sql
-- 예시: 계좌 이체 상황
-- Transaction T1: A계좌 → B계좌로 100원 이체
-- Transaction T2: A계좌 → C계좌로 50원 이체

-- T1: 읽기(A) = 1000
-- T2: 읽기(A) = 1000  (동시 읽기)
-- T1: A = 1000 - 100 = 900, 쓰기(A)
-- T2: A = 1000 - 50 = 950, 쓰기(A)
-- 결과: A = 950 (T1의 -100이 손실됨)
```

## 락 기반 동시성 제어

### 1. 기본 락 유형

#### 공유 락 (Shared Lock, S-Lock)

```sql
-- 읽기 락: 여러 트랜잭션이 동시에 읽기 가능
-- PostgreSQL
BEGIN;
SELECT * FROM accounts WHERE account_id = 'A001' FOR SHARE;
-- 다른 트랜잭션도 같은 데이터를 읽을 수 있음
-- 하지만 쓰기는 대기해야 함
COMMIT;

-- MySQL
BEGIN;
SELECT * FROM accounts WHERE account_id = 'A001' LOCK IN SHARE MODE;
COMMIT;
```

#### 배타 락 (Exclusive Lock, X-Lock)

```sql
-- 쓰기 락: 하나의 트랜잭션만 접근 가능
-- PostgreSQL
BEGIN;
SELECT * FROM accounts WHERE account_id = 'A001' FOR UPDATE;
-- 다른 트랜잭션의 읽기/쓰기 모두 대기
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
COMMIT;

-- SQL Server
BEGIN TRANSACTION;
SELECT * FROM accounts WITH (XLOCK) WHERE account_id = 'A001';
-- 배타적 잠금으로 다른 모든 접근 차단
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
COMMIT;
```

### 2. 락 호환성 행렬

```
        | None | S-Lock | X-Lock
--------|------|--------|--------
None    |  ✓   |   ✓    |   ✓
S-Lock  |  ✓   |   ✓    |   ✗
X-Lock  |  ✓   |   ✗    |   ✗
```

### 3. 락 단위 (Granularity)

#### 행 수준 락 (Row-Level Lock)

```sql
-- InnoDB에서 행 단위 락
BEGIN;
SELECT * FROM users WHERE user_id = 123 FOR UPDATE;
-- user_id = 123인 행만 락됨
-- 다른 행들은 자유롭게 접근 가능
UPDATE users SET last_login = NOW() WHERE user_id = 123;
COMMIT;
```

#### 테이블 수준 락 (Table-Level Lock)

```sql
-- 테이블 전체 락
LOCK TABLES accounts READ;
-- 모든 세션이 읽기만 가능

LOCK TABLES accounts WRITE;
-- 현재 세션만 읽기/쓰기 가능, 다른 세션은 대기

UNLOCK TABLES;
```

#### 페이지 수준 락 (Page-Level Lock)

```sql
-- SQL Server에서 페이지 락
-- 보통 자동으로 결정되지만 힌트 사용 가능
SELECT * FROM accounts WITH (PAGLOCK) WHERE balance > 1000;
```

### 4. 2단계 락킹 프로토콜 (Two-Phase Locking)

#### 확장 단계와 축소 단계

```python
# 2PL 프로토콜 구현 예시
class TwoPhaseLocking:
    def __init__(self):
        self.locks_held = set()
        self.growing_phase = True

    def acquire_lock(self, resource, lock_type):
        if not self.growing_phase:
            raise Exception("Cannot acquire lock in shrinking phase")

        if self.can_acquire_lock(resource, lock_type):
            self.locks_held.add((resource, lock_type))
            return True
        else:
            # 대기 또는 데드락 감지
            return False

    def release_lock(self, resource, lock_type):
        self.growing_phase = False  # 축소 단계 시작
        self.locks_held.remove((resource, lock_type))

    def commit_or_rollback(self):
        # 모든 락 해제
        self.locks_held.clear()
        self.growing_phase = True
```

### 5. 의도 락 (Intent Locks)

```sql
-- 계층적 락킹을 위한 의도 락
-- 테이블 수준에서 행 수준 락의 존재를 표시

-- IS (Intent Shared): 하위 레벨에 공유 락 존재
-- IX (Intent Exclusive): 하위 레벨에 배타 락 존재
-- SIX (Shared Intent Exclusive): 공유 락 + 하위에 배타 락 존재

-- 호환성 행렬
```

     | IS | IX | S  | SIX| X

-----|----|----|----|----|----
IS | ✓ | ✓ | ✓ | ✓ | ✗
IX | ✓ | ✓ | ✗ | ✗ | ✗
S | ✓ | ✗ | ✓ | ✗ | ✗
SIX | ✓ | ✗ | ✗ | ✗ | ✗
X | ✗ | ✗ | ✗ | ✗ | ✗

```

```

## MVCC (Multi-Version Concurrency Control)

### 1. MVCC 기본 원리

각 데이터 항목에 대해 여러 버전을 유지하여 읽기 작업이 쓰기 작업을 기다리지 않도록 합니다.

```python
# MVCC 개념적 구현
class MVCCDatabase:
    def __init__(self):
        self.data = {}  # key -> [(value, transaction_id, timestamp, is_deleted)]
        self.active_transactions = {}

    def begin_transaction(self, txn_id):
        self.active_transactions[txn_id] = {
            'start_time': time.time(),
            'read_set': set(),
            'write_set': set()
        }

    def read(self, key, txn_id):
        versions = self.data.get(key, [])
        txn_start_time = self.active_transactions[txn_id]['start_time']

        # 트랜잭션 시작 이전에 커밋된 가장 최신 버전 찾기
        visible_versions = [
            v for v in versions
            if v[2] < txn_start_time and not v[3]  # 삭제되지 않은 버전
        ]

        if visible_versions:
            latest_version = max(visible_versions, key=lambda x: x[2])
            self.active_transactions[txn_id]['read_set'].add(key)
            return latest_version[0]
        return None

    def write(self, key, value, txn_id):
        if key not in self.data:
            self.data[key] = []

        # 새 버전 추가 (커밋 시까지 임시)
        current_time = time.time()
        self.data[key].append((value, txn_id, current_time, False))
        self.active_transactions[txn_id]['write_set'].add(key)
```

### 2. PostgreSQL의 MVCC 구현

#### 튜플 헤더 정보

```sql
-- PostgreSQL의 각 행은 다음 정보를 포함
-- xmin: 행을 생성한 트랜잭션 ID
-- xmax: 행을 삭제한 트랜잭션 ID
-- cmin: 명령 ID (트랜잭션 내 명령 순서)
-- cmax: 삭제 명령 ID

-- 행 버전 정보 확인
SELECT xmin, xmax, cmin, cmax, * FROM accounts;
```

#### 가시성 규칙

```sql
-- 트랜잭션이 행을 볼 수 있는 조건:
-- 1. xmin이 현재 트랜잭션보다 작고 커밋되었음
-- 2. xmax가 0이거나 현재 트랜잭션보다 크거나 롤백되었음

-- 예시: 동시 업데이트
-- T1: BEGIN; UPDATE accounts SET balance = 900 WHERE id = 1;
-- T2: BEGIN; SELECT * FROM accounts WHERE id = 1;
-- T2는 T1이 커밋되기 전까지 기존 값을 봄
```

### 3. Oracle의 읽기 일관성

```sql
-- Oracle의 Undo 세그먼트를 이용한 읽기 일관성
-- SCN (System Commit Number) 기반 버전 관리

-- 특정 시점으로의 플래시백 쿼리
SELECT * FROM accounts
AS OF SCN 1000000;

-- 시간 기반 플래시백
SELECT * FROM accounts
AS OF TIMESTAMP (SYSTIMESTAMP - INTERVAL '5' MINUTE);
```

### 4. MySQL InnoDB의 MVCC

```sql
-- InnoDB의 언두 로그를 이용한 MVCC
-- 트랜잭션 ID와 롤백 포인터 사용

-- 현재 활성 트랜잭션 확인
SELECT
    trx_id,
    trx_state,
    trx_started,
    trx_mysql_thread_id
FROM INFORMATION_SCHEMA.INNODB_TRX;

-- 언두 로그 상태 확인
SHOW ENGINE INNODB STATUS;
```

## 낙관적 동시성 제어

### 1. 타임스탬프 기반 제어

```sql
-- 버전 번호를 이용한 낙관적 락
CREATE TABLE accounts (
    account_id VARCHAR(10) PRIMARY KEY,
    balance DECIMAL(10,2),
    version INTEGER DEFAULT 1,
    last_modified TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 낙관적 업데이트
BEGIN;
SELECT balance, version FROM accounts WHERE account_id = 'A001';
-- balance = 1000, version = 5

-- 비즈니스 로직 수행 후
UPDATE accounts
SET balance = 900,
    version = version + 1,
    last_modified = CURRENT_TIMESTAMP
WHERE account_id = 'A001'
AND version = 5;  -- 버전 확인

-- 영향받은 행이 0이면 다른 트랜잭션이 먼저 수정한 것
GET DIAGNOSTICS affected_rows = ROW_COUNT;
IF affected_rows = 0 THEN
    ROLLBACK;
    -- 충돌 처리 (재시도 또는 오류)
ELSE
    COMMIT;
END IF;
```

### 2. 체크섬을 이용한 변경 감지

```python
import hashlib

class OptimisticConcurrency:
    def __init__(self, db):
        self.db = db

    def read_with_checksum(self, account_id):
        data = self.db.execute(
            "SELECT * FROM accounts WHERE account_id = ?",
            (account_id,)
        ).fetchone()

        # 데이터의 체크섬 계산
        checksum = hashlib.md5(str(data).encode()).hexdigest()
        return data, checksum

    def update_with_validation(self, account_id, new_balance, expected_checksum):
        # 현재 데이터 다시 읽기
        current_data, current_checksum = self.read_with_checksum(account_id)

        if current_checksum != expected_checksum:
            raise ConcurrencyConflictError("Data has been modified by another transaction")

        # 업데이트 수행
        self.db.execute(
            "UPDATE accounts SET balance = ? WHERE account_id = ?",
            (new_balance, account_id)
        )
```

## 데드락 처리

### 1. 데드락 감지

```python
# 대기 그래프를 이용한 데드락 감지
class DeadlockDetector:
    def __init__(self):
        self.wait_graph = {}  # 트랜잭션 대기 관계

    def add_wait_edge(self, waiting_txn, blocking_txn):
        if waiting_txn not in self.wait_graph:
            self.wait_graph[waiting_txn] = []
        self.wait_graph[waiting_txn].append(blocking_txn)

    def has_cycle(self):
        visited = set()
        rec_stack = set()

        def dfs(node):
            if node in rec_stack:
                return True  # 사이클 발견
            if node in visited:
                return False

            visited.add(node)
            rec_stack.add(node)

            for neighbor in self.wait_graph.get(node, []):
                if dfs(neighbor):
                    return True

            rec_stack.remove(node)
            return False

        for txn in self.wait_graph:
            if dfs(txn):
                return True
        return False

    def select_victim(self):
        # 가장 적은 작업을 한 트랜잭션을 희생자로 선택
        return min(self.wait_graph.keys(),
                  key=lambda txn: self.get_work_done(txn))
```

### 2. 데드락 방지

```sql
-- 항상 같은 순서로 리소스 접근 (Ordered Locking)
-- 나쁜 예:
-- T1: LOCK TABLE A, LOCK TABLE B
-- T2: LOCK TABLE B, LOCK TABLE A

-- 좋은 예: 항상 알파벳 순서로 락 획득
-- T1: LOCK TABLE A, LOCK TABLE B
-- T2: LOCK TABLE A, LOCK TABLE B

-- 타임아웃을 이용한 데드락 해결
SET innodb_lock_wait_timeout = 50; -- 50초 후 타임아웃
```

### 3. 데드락 모니터링

```sql
-- MySQL에서 데드락 정보 확인
SHOW ENGINE INNODB STATUS;

-- PostgreSQL에서 락 대기 상황
SELECT
    blocked_locks.pid AS blocked_pid,
    blocked_activity.query AS blocked_statement,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.query AS blocking_statement
FROM pg_locks blocked_locks
JOIN pg_stat_activity blocked_activity ON blocked_locks.pid = blocked_activity.pid
JOIN pg_locks blocking_locks ON blocking_locks.transactionid = blocked_locks.transactionid
JOIN pg_stat_activity blocking_activity ON blocking_locks.pid = blocking_activity.pid
WHERE NOT blocked_locks.granted;
```

## 성능 최적화

### 1. 락 경합 최소화

```sql
-- 길게 지속되는 락 피하기
-- 나쁜 예:
BEGIN;
SELECT * FROM large_table WHERE complex_condition = true; -- 오래 걸리는 쿼리
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
COMMIT;

-- 좋은 예: 락 시간 최소화
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A001';
COMMIT;
-- 복잡한 쿼리는 별도로 실행
```

### 2. 락 단위 최적화

```sql
-- 적절한 락 단위 선택
-- 소수 행 업데이트: 행 수준 락
UPDATE users SET last_login = NOW() WHERE user_id = 123;

-- 대량 업데이트: 테이블 수준 락이 더 효율적일 수 있음
LOCK TABLE users WRITE;
UPDATE users SET status = 'inactive' WHERE last_login < '2023-01-01';
UNLOCK TABLES;
```

### 3. 인덱스를 이용한 락 최적화

```sql
-- 인덱스가 없으면 테이블 스캔으로 많은 행이 락됨
CREATE INDEX idx_accounts_status ON accounts(status);

-- 인덱스 사용으로 필요한 행만 락됨
UPDATE accounts SET balance = balance * 1.1
WHERE status = 'ACTIVE' AND account_id = 'A001';
```

## 실무 적용 가이드

### 1. 동시성 제어 방식 선택

```python
def choose_concurrency_control(workload_type, consistency_requirement):
    if workload_type == "read_heavy" and consistency_requirement == "eventual":
        return "MVCC"
    elif workload_type == "write_heavy" and consistency_requirement == "strong":
        return "Two-Phase Locking"
    elif workload_type == "mixed" and consistency_requirement == "medium":
        return "Optimistic Concurrency Control"
    else:
        return "Serializable Isolation"
```

### 2. 애플리케이션 레벨 최적화

```java
// 재시도 로직을 포함한 낙관적 락
@Retryable(value = OptimisticLockingFailureException.class, maxAttempts = 3)
@Transactional
public void updateAccountBalance(String accountId, BigDecimal amount) {
    Account account = accountRepository.findById(accountId);
    account.setBalance(account.getBalance().add(amount));
    account.setVersion(account.getVersion() + 1);
    accountRepository.save(account);
}

// 비관적 락 사용
@Transactional
public void transferMoney(String fromId, String toId, BigDecimal amount) {
    Account fromAccount = accountRepository.findByIdWithLock(fromId);
    Account toAccount = accountRepository.findByIdWithLock(toId);

    fromAccount.withdraw(amount);
    toAccount.deposit(amount);

    accountRepository.save(fromAccount);
    accountRepository.save(toAccount);
}
```

## 결론

동시성 제어는 데이터베이스 성능과 일관성 사이의 균형을 맞추는 핵심 기술입니다. 락 기반 제어는 강한 일관성을 제공하지만 성능 저하를 가져올 수 있고, MVCC는 읽기 성능을 향상시키지만 저장 공간과 메모리 사용량이 증가합니다. 낙관적 제어는 충돌이 적은 환경에서 좋은 성능을 보이지만 충돌 처리 로직이 복잡해집니다. 애플리케이션의 특성에 맞는 적절한 동시성 제어 방식을 선택하고, 데드락 방지와 성능 최적화를 함께 고려해야 합니다.
