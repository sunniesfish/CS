# NoSQL의 기본 개념

## 개요

NoSQL(Not Only SQL)은 전통적인 관계형 데이터베이스의 한계를 극복하기 위해 등장한 비관계형 데이터베이스의 총칭입니다. 스키마 유연성, 수평 확장성, 대용량 데이터 처리에 최적화되어 있습니다.

## NoSQL의 등장 배경

### 관계형 데이터베이스의 한계

- **스케일 아웃의 어려움**: 수직 확장(Scale-up)에 의존
- **스키마 경직성**: 구조 변경의 어려움
- **복잡한 조인**: 대용량 데이터에서 성능 저하
- **ACID의 오버헤드**: 분산 환경에서 성능 제약

### 웹 2.0 시대의 요구사항

- **빅데이터**: 페타바이트 급 데이터 처리
- **실시간 처리**: 낮은 지연시간 요구
- **유연한 데이터 모델**: 비정형 데이터 증가
- **글로벌 분산**: 지리적 분산 서비스

## NoSQL의 주요 특징

### 1. 스키마리스 (Schema-less)

```json
// 동일 컬렉션에 다른 구조의 문서 저장 가능
{
  "id": 1,
  "name": "John",
  "email": "john@email.com"
}

{
  "id": 2,
  "name": "Jane",
  "email": "jane@email.com",
  "phone": "555-1234",
  "address": {
    "city": "Seoul",
    "country": "Korea"
  }
}
```

### 2. 수평 확장성 (Horizontal Scaling)

```javascript
// MongoDB 샤딩 예시
sh.enableSharding("mydb");
sh.shardCollection("mydb.users", { user_id: "hashed" });

// 데이터가 여러 샤드에 자동 분산
// Shard 1: user_id hash 0-333333
// Shard 2: user_id hash 333334-666666
// Shard 3: user_id hash 666667-999999
```

### 3. BASE 특성

- **BA**: Basically Available (기본적으로 사용 가능)
- **S**: Soft state (유연한 상태)
- **E**: Eventual consistency (최종적 일관성)

## NoSQL 데이터베이스 유형

### 1. Key-Value 스토어

#### 특징

가장 단순한 NoSQL 모델로, 고유한 키와 값의 쌍으로 데이터를 저장합니다.

#### Redis 예시

```bash
# 기본 Key-Value 연산
SET user:1001 "John Doe"
GET user:1001

# 복합 데이터 타입
HSET user:1001 name "John Doe" email "john@email.com" age 30
HGET user:1001 name

# 리스트 타입
LPUSH user:1001:orders "order-123"
LPUSH user:1001:orders "order-456"
LRANGE user:1001:orders 0 -1

# Set 타입
SADD user:1001:interests "technology" "music" "travel"
SMEMBERS user:1001:interests
```

#### DynamoDB 예시

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

# 아이템 저장
table.put_item(
    Item={
        'user_id': '1001',
        'name': 'John Doe',
        'email': 'john@email.com',
        'preferences': {
            'language': 'en',
            'timezone': 'UTC'
        }
    }
)

# 아이템 조회
response = table.get_item(
    Key={'user_id': '1001'}
)
```

#### 활용 사례

- **세션 관리**: 웹 애플리케이션 세션 데이터
- **캐싱**: 데이터베이스 쿼리 결과 캐시
- **설정 관리**: 애플리케이션 구성 정보
- **실시간 추천**: 사용자별 추천 아이템 목록

### 2. Document 스토어

#### 특징

JSON, BSON, XML 형태의 문서로 데이터를 저장하며, 스키마가 유연합니다.

#### MongoDB 예시

```javascript
// 문서 삽입
db.users.insertOne({
  _id: ObjectId(),
  name: "John Doe",
  email: "john@email.com",
  age: 30,
  address: {
    street: "123 Main St",
    city: "Seoul",
    zipcode: "12345",
  },
  hobbies: ["reading", "swimming", "coding"],
  created_at: new Date(),
});

// 복잡한 쿼리
db.users.find({
  age: { $gte: 25, $lte: 35 },
  "address.city": "Seoul",
  hobbies: { $in: ["coding"] },
});

// 집계 파이프라인
db.users.aggregate([
  { $match: { age: { $gte: 25 } } },
  {
    $group: {
      _id: "$address.city",
      count: { $sum: 1 },
      avgAge: { $avg: "$age" },
    },
  },
  { $sort: { count: -1 } },
]);
```

#### CouchDB 예시

```javascript
// 문서 생성
{
  "_id": "user001",
  "_rev": "1-967a00dff5e02add41819138abb3284d",
  "type": "user",
  "name": "John Doe",
  "email": "john@email.com",
  "profile": {
    "bio": "Software Developer",
    "interests": ["technology", "music"]
  }
}

// 뷰 정의 (MapReduce)
{
  "_id": "_design/users",
  "views": {
    "by_city": {
      "map": "function(doc) { if(doc.type == 'user' && doc.address) { emit(doc.address.city, doc); } }"
    }
  }
}

// 뷰 쿼리
GET /mydb/_design/users/_view/by_city?key="Seoul"
```

#### 활용 사례

- **콘텐츠 관리**: 블로그, 뉴스 기사
- **상품 카탈로그**: 전자상거래 상품 정보
- **사용자 프로필**: 소셜 미디어 프로필
- **로깅**: 애플리케이션 로그 저장

### 3. Column Family (Wide Column)

#### Cassandra 예시

```sql
-- Keyspace 생성
CREATE KEYSPACE ecommerce
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 3
};

-- 테이블 생성
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    email TEXT,
    name TEXT,
    created_at TIMESTAMP
);

-- 시계열 데이터를 위한 복합 키
CREATE TABLE user_activities (
    user_id UUID,
    activity_date DATE,
    activity_time TIMESTAMP,
    activity_type TEXT,
    details MAP<TEXT, TEXT>,
    PRIMARY KEY ((user_id, activity_date), activity_time)
);

-- 쿼리
INSERT INTO user_activities (user_id, activity_date, activity_time, activity_type, details)
VALUES (uuid(), '2024-01-15', now(), 'login', {'ip': '192.168.1.1', 'device': 'mobile'});

SELECT * FROM user_activities
WHERE user_id = ? AND activity_date = '2024-01-15'
ORDER BY activity_time DESC;
```

#### HBase 예시

```java
// HBase Java API
Configuration conf = HBaseConfiguration.create();
Connection connection = ConnectionFactory.createConnection(conf);
Table table = connection.getTable(TableName.valueOf("users"));

// Put 연산
Put put = new Put(Bytes.toBytes("user001"));
put.addColumn(Bytes.toBytes("profile"), Bytes.toBytes("name"), Bytes.toBytes("John Doe"));
put.addColumn(Bytes.toBytes("profile"), Bytes.toBytes("email"), Bytes.toBytes("john@email.com"));
put.addColumn(Bytes.toBytes("stats"), Bytes.toBytes("login_count"), Bytes.toBytes("42"));
table.put(put);

// Get 연산
Get get = new Get(Bytes.toBytes("user001"));
Result result = table.get(get);
String name = Bytes.toString(result.getValue(Bytes.toBytes("profile"), Bytes.toBytes("name")));
```

### 4. Graph Database

#### Neo4j 예시

```cypher
// 노드 생성
CREATE (john:Person {name: "John Doe", age: 30})
CREATE (jane:Person {name: "Jane Smith", age: 28})
CREATE (company:Company {name: "Tech Corp"})

// 관계 생성
CREATE (john)-[:WORKS_FOR {since: date('2020-01-01')}]->(company)
CREATE (jane)-[:WORKS_FOR {since: date('2021-05-15')}]->(company)
CREATE (john)-[:KNOWS {since: date('2019-03-10')}]->(jane)

// 그래프 탐색 쿼리
MATCH (p:Person)-[:WORKS_FOR]->(c:Company {name: "Tech Corp"})
RETURN p.name, p.age

// 친구의 친구 찾기 (2단계 관계)
MATCH (john:Person {name: "John Doe"})-[:KNOWS*1..2]-(friend)
RETURN DISTINCT friend.name

// 최단 경로 찾기
MATCH path = shortestPath((john:Person {name: "John Doe"})-[*]-(target:Person {name: "Target Person"}))
RETURN path
```

## NoSQL과 SQL 비교

### 데이터 모델링

```javascript
// SQL 방식 (정규화)
// users 테이블
{id: 1, name: "John", email: "john@email.com"}

// orders 테이블
{id: 101, user_id: 1, amount: 99.99, date: "2024-01-15"}

// order_items 테이블
{order_id: 101, product_id: 201, quantity: 2, price: 49.99}

// NoSQL 방식 (비정규화)
{
  "_id": "order_101",
  "customer": {
    "id": 1,
    "name": "John",
    "email": "john@email.com"
  },
  "order_date": "2024-01-15",
  "total_amount": 99.99,
  "items": [
    {
      "product_id": 201,
      "product_name": "Laptop",
      "quantity": 1,
      "price": 99.99
    }
  ]
}
```

### 쿼리 복잡성

```sql
-- SQL: 복잡한 조인
SELECT u.name, o.order_date, oi.product_name, oi.quantity
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
WHERE u.email = 'john@email.com'
AND o.order_date >= '2024-01-01';
```

```javascript
// MongoDB: 단일 컬렉션 쿼리
db.orders.find({
  "customer.email": "john@email.com",
  order_date: { $gte: "2024-01-01" },
});
```

## 일관성 모델

### 강한 일관성 (Strong Consistency)

```javascript
// MongoDB with majority write concern
db.users.insertOne(
  { name: "John", email: "john@email.com" },
  { writeConcern: { w: "majority", j: true } }
);

// 읽기도 majority에서 확인
db.users.find({ name: "John" }).readConcern("majority");
```

### 최종적 일관성 (Eventual Consistency)

```python
# DynamoDB eventually consistent read
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

# 쓰기
table.put_item(Item={'user_id': '123', 'name': 'John'})

# Eventually consistent read (기본값)
response = table.get_item(
    Key={'user_id': '123'},
    ConsistentRead=False  # 빠르지만 최신이 아닐 수 있음
)

# Strong consistent read
response = table.get_item(
    Key={'user_id': '123'},
    ConsistentRead=True   # 느리지만 최신 보장
)
```

## 성능과 확장성

### 샤딩 전략

```javascript
// MongoDB 샤딩
// Hash-based 샤딩
sh.shardCollection("mydb.users", { user_id: "hashed" });

// Range-based 샤딩
sh.shardCollection("mydb.orders", { order_date: 1 });

// Compound 샤딩
sh.shardCollection("mydb.events", { user_id: 1, timestamp: 1 });
```

### 복제 전략

```bash
# Redis Cluster 설정
redis-cli --cluster create \
  127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 \
  127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 \
  --cluster-replicas 1
```

## NoSQL 선택 가이드

### 사용 사례별 선택

```python
def choose_nosql_database(requirements):
    if requirements.simple_key_value and requirements.high_performance:
        return "Redis"  # 캐싱, 세션

    elif requirements.document_structure and requirements.flexible_schema:
        return "MongoDB"  # CMS, 프로필 관리

    elif requirements.time_series and requirements.high_write_throughput:
        return "Cassandra"  # IoT 데이터, 로깅

    elif requirements.relationship_analysis and requirements.graph_traversal:
        return "Neo4j"  # 소셜 네트워크, 추천

    elif requirements.serverless and requirements.auto_scaling:
        return "DynamoDB"  # AWS 기반 웹앱
```

### 관계형 DB와 NoSQL 하이브리드

```python
# Polyglot Persistence 예시
class UserService:
    def __init__(self):
        self.postgres = PostgreSQLConnection()  # 트랜잭션 데이터
        self.mongodb = MongoDBConnection()      # 프로필 데이터
        self.redis = RedisConnection()          # 캐시
        self.neo4j = Neo4jConnection()          # 친구 관계

    def create_user(self, user_data):
        # 1. 핵심 데이터는 관계형 DB
        user_id = self.postgres.insert_user(user_data['basic'])

        # 2. 확장 프로필은 문서 DB
        self.mongodb.save_profile(user_id, user_data['profile'])

        # 3. 캐시 업데이트
        self.redis.set(f"user:{user_id}", user_data)

        return user_id

    def get_user_with_friends(self, user_id):
        # 캐시에서 먼저 확인
        cached = self.redis.get(f"user:{user_id}")
        if cached:
            return cached

        # 기본 정보는 PostgreSQL
        basic_info = self.postgres.get_user(user_id)

        # 프로필은 MongoDB
        profile = self.mongodb.get_profile(user_id)

        # 친구 관계는 Neo4j
        friends = self.neo4j.get_friends(user_id)

        result = {**basic_info, **profile, 'friends': friends}

        # 캐시에 저장
        self.redis.setex(f"user:{user_id}", 3600, result)

        return result
```

## 결론

NoSQL은 관계형 데이터베이스를 완전히 대체하는 것이 아니라, 특정 사용 사례에서 더 적합한 솔루션입니다. 각 NoSQL 유형의 특성을 이해하고, 애플리케이션의 요구사항에 맞는 적절한 데이터베이스를 선택하는 것이 중요합니다. 현대적인 아키텍처에서는 여러 데이터베이스를 조합하여 사용하는 폴리글랏 퍼시스턴스가 일반적입니다.
