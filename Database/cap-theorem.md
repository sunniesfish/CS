# CAP 이론 (Consistency, Availability, Partition Tolerance)

## 개요

CAP 이론은 2000년 Eric Brewer가 제시한 분산 시스템의 근본적 한계를 설명하는 정리로, 일관성(Consistency), 가용성(Availability), 분할 내성(Partition tolerance) 중 최대 2개만 동시에 보장할 수 있다는 이론입니다.

## CAP의 세 가지 속성

### 1. 일관성 (Consistency)

모든 노드가 동시에 같은 데이터를 볼 수 있어야 합니다.

```python
# 강한 일관성 예시
def write_data(key, value):
    # 모든 노드에 동기적으로 쓰기
    for node in all_nodes:
        node.write(key, value)
        node.confirm_write()  # 모든 노드의 확인 대기
    return "SUCCESS"

def read_data(key):
    # 모든 노드에서 같은 값을 읽음
    values = [node.read(key) for node in all_nodes]
    assert all(v == values[0] for v in values)  # 모든 값이 동일해야 함
    return values[0]
```

### 2. 가용성 (Availability)

시스템이 항상 요청에 응답할 수 있어야 합니다.

```python
# 높은 가용성 예시
def read_data_available(key):
    for node in all_nodes:
        try:
            return node.read(key)  # 첫 번째로 응답하는 노드의 값 반환
        except NodeUnavailableError:
            continue
    raise AllNodesDownError()

def write_data_available(key, value):
    success_count = 0
    for node in all_nodes:
        try:
            node.write(key, value)
            success_count += 1
        except NodeUnavailableError:
            continue

    if success_count > 0:
        return "SUCCESS"  # 일부 노드에만 써도 성공으로 처리
    raise AllNodesDownError()
```

### 3. 분할 내성 (Partition Tolerance)

네트워크 분할이 발생해도 시스템이 계속 동작해야 합니다.

```python
# 네트워크 분할 처리 예시
class DistributedSystem:
    def __init__(self):
        self.nodes = {'A': [], 'B': []}  # 두 개의 파티션

    def handle_partition(self):
        # 네트워크 분할 발생 시
        if self.is_partitioned():
            # 각 파티션이 독립적으로 동작
            self.partition_A.continue_operation()
            self.partition_B.continue_operation()

    def is_partitioned(self):
        return not self.can_communicate(self.partition_A, self.partition_B)
```

## CAP 트레이드오프

### CA (Consistency + Availability) - 분할 내성 포기

```sql
-- 전통적인 RDBMS (단일 노드 또는 동기 복제)
-- MySQL with Synchronous Replication
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(100)
);

-- 마스터에 쓰기
INSERT INTO users VALUES (1, 'John', 'john@email.com');
-- 슬레이브에 동기적으로 복제됨 (일관성)
-- 모든 노드가 응답 가능 (가용성)
-- 하지만 네트워크 분할 시 시스템 정지 (분할 내성 부족)
```

### CP (Consistency + Partition Tolerance) - 가용성 포기

```python
# MongoDB with Strong Consistency
from pymongo import MongoClient

client = MongoClient('mongodb://localhost:27017/')
db = client.mydb

# 강한 일관성 설정
collection = db.users
collection.insert_one(
    {"name": "John", "email": "john@email.com"},
    write_concern=WriteConcern(w="majority")  # 과반수 노드 확인 대기
)

# 읽기도 과반수 노드에서 확인
result = collection.find_one(
    {"name": "John"},
    read_concern=ReadConcern("majority")
)

# 네트워크 분할 시 과반수 확보 못하면 요청 거부 (가용성 희생)
```

### AP (Availability + Partition Tolerance) - 일관성 포기

```python
# DynamoDB with Eventual Consistency
import boto3

dynamodb = boto3.resource('dynamodb', region_name='us-west-2')
table = dynamodb.Table('Users')

# 쓰기 (모든 복제본에 즉시 전파되지 않음)
table.put_item(
    Item={
        'user_id': '123',
        'name': 'John',
        'email': 'john@email.com'
    }
)

# 읽기 (최신 값이 아닐 수 있음)
response = table.get_item(
    Key={'user_id': '123'},
    ConsistentRead=False  # Eventually consistent read
)

# 높은 가용성 보장, 네트워크 분할에 강함, 하지만 일시적 불일치 가능
```

## 실제 시스템에서의 CAP 적용

### 1. 전통적인 RDBMS (CA 시스템)

```sql
-- PostgreSQL Synchronous Replication
-- postgresql.conf
synchronous_standby_names = 'standby1,standby2'
synchronous_commit = on

-- 트랜잭션 예시
BEGIN;
INSERT INTO orders (customer_id, amount) VALUES (123, 99.99);
-- 모든 동기 스탠바이가 확인할 때까지 대기
COMMIT; -- 일관성과 가용성 보장, 분할 시 정지
```

### 2. NoSQL AP 시스템

```javascript
// Cassandra (AP 시스템)
const cassandra = require("cassandra-driver");
const client = new cassandra.Client({
  contactPoints: ["127.0.0.1"],
  localDataCenter: "datacenter1",
});

// 쓰기 (Eventual Consistency)
await client.execute(
  "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
  [uuid(), "John", "john@email.com"],
  { consistency: cassandra.types.consistencies.one } // 한 노드만 성공하면 OK
);

// 읽기 (가용성 우선)
const result = await client.execute(
  "SELECT * FROM users WHERE id = ?",
  [userId],
  { consistency: cassandra.types.consistencies.one }
);
```

### 3. CP 시스템

```python
# etcd (CP 시스템) - 분산 설정 관리
import etcd3

client = etcd3.client(host='localhost', port=2379)

# 강한 일관성 보장
def update_config(key, value):
    # Raft 합의 알고리즘으로 일관성 보장
    client.put(key, value)

def read_config(key):
    # 리더에서만 읽기 (강한 일관성)
    return client.get(key)[0]

# 과반수 노드 손실 시 쓰기 불가 (가용성 희생)
```

## 현실적 CAP 해석

### 1. BASE vs ACID

```python
# BASE (Basically Available, Soft state, Eventual consistency)
class EventuallyConsistentStore:
    def __init__(self):
        self.nodes = {}
        self.version_vector = {}

    def write(self, key, value, node_id):
        # 로컬 노드에만 쓰기 (즉시 응답)
        self.nodes[node_id][key] = value
        self.version_vector[key] = time.time()

        # 백그라운드에서 다른 노드에 전파
        self.async_replicate(key, value)

    def read(self, key):
        # 가장 최근 버전 반환 (최종적 일관성)
        latest_version = max(self.version_vector.get(key, 0))
        return self.get_value_at_version(key, latest_version)
```

### 2. Quorum 기반 시스템

```python
# Amazon DynamoDB 스타일 Quorum
class QuorumSystem:
    def __init__(self, N=3, R=2, W=2):
        self.N = N  # 총 복제본 수
        self.R = R  # 읽기에 필요한 최소 노드 수
        self.W = W  # 쓰기에 필요한 최소 노드 수

    def write(self, key, value):
        success_count = 0
        for node in self.nodes:
            try:
                node.write(key, value)
                success_count += 1
                if success_count >= self.W:
                    return True  # W개 노드 성공시 반환
            except:
                continue
        return False

    def read(self, key):
        responses = []
        for node in self.nodes[:self.R]:
            try:
                responses.append(node.read(key))
            except:
                continue

        if len(responses) >= self.R:
            # 버전 벡터로 최신 값 결정
            return self.resolve_conflicts(responses)
        return None
```

## CAP 이론의 실용적 적용

### 1. 마이크로서비스에서의 CAP

```yaml
# Service Mesh에서 CAP 고려사항
apiVersion: v1
kind: ConfigMap
metadata:
  name: service-config
data:
  consistency-level: "eventual" # AP 선택
  timeout: "5s"
  retry-policy: "exponential-backoff"

  # Circuit Breaker 설정 (가용성 우선)
  circuit-breaker:
    failure-threshold: 5
    timeout: 30s
```

### 2. 데이터베이스 선택 가이드

```python
# 업무별 CAP 선택 가이드
def choose_database(requirements):
    if requirements.strong_consistency and requirements.small_scale:
        return "PostgreSQL"  # CA
    elif requirements.strong_consistency and requirements.distributed:
        return "MongoDB"     # CP
    elif requirements.high_availability and requirements.eventual_consistency_ok:
        return "Cassandra"   # AP
    elif requirements.financial_transactions:
        return "CockroachDB" # CP with high availability
```

### 3. 하이브리드 접근

```python
# Lambda Architecture - 배치와 스트림 처리
class HybridSystem:
    def __init__(self):
        self.batch_layer = PostgreSQLCluster()    # CP (정확성)
        self.speed_layer = RedisCluster()         # AP (빠른 응답)
        self.serving_layer = CassandraCluster()   # AP (확장성)

    def write(self, data):
        # 실시간 처리 (AP)
        self.speed_layer.write(data)

        # 배치 처리용 저장 (CP)
        self.batch_layer.queue_for_batch(data)

    def read(self, key):
        # Speed layer에서 먼저 확인 (빠른 응답)
        result = self.speed_layer.get(key)
        if result:
            return result

        # Serving layer에서 확인 (정확한 결과)
        return self.serving_layer.get(key)
```

## 최신 CAP 이해

### 1. PACELC 이론

```
네트워크 분할(P) 시: Availability vs Consistency
평상시(E): Latency vs Consistency
```

```python
# PACELC를 고려한 시스템 설계
class PAECLCSystem:
    def __init__(self, partition_mode='AP', normal_mode='LC'):
        self.partition_strategy = partition_mode
        self.normal_strategy = normal_mode

    def handle_request(self, request):
        if self.is_partitioned():
            if self.partition_strategy == 'AP':
                return self.serve_with_availability()  # 빠르지만 일관성 포기
            else:  # CP
                return self.serve_with_consistency()   # 느리지만 정확함
        else:
            if self.normal_strategy == 'LC':
                return self.serve_with_low_latency()   # 캐시 활용
            else:  # EC
                return self.serve_with_consistency()   # 항상 최신 데이터
```

### 2. 실무에서의 절충점

```sql
-- 읽기 복제본 활용 (읽기는 AP, 쓰기는 CP)
-- Master (쓰기용 - CP)
INSERT INTO orders (customer_id, amount, created_at)
VALUES (123, 99.99, NOW());

-- Read Replica (읽기용 - AP with eventual consistency)
SELECT * FROM orders WHERE customer_id = 123;
-- 약간의 지연은 있지만 높은 가용성 제공
```

## 결론

CAP 이론은 분산 시스템 설계의 근본적 제약을 이해하는 핵심 개념입니다. 완벽한 시스템은 존재하지 않으며, 비즈니스 요구사항에 따라 적절한 트레이드오프를 선택해야 합니다. 현대 시스템은 단순한 CAP 분류보다는 상황별로 다른 전략을 취하는 유연한 접근을 사용합니다.
