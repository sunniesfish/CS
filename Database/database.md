# 데이터베이스 전공 지식 모음

이 폴더는 데이터베이스 이론과 실무에 필요한 전공 수준의 상세한 내용을 담고 있습니다.

## 📚 목차

### 기본 개념

- [데이터 독립성 (논리적, 물리적)](data-independence.md)
- [관계형 모델의 기본 개념 (릴레이션, 튜플, 애트리뷰트 등)](relational-model-basics.md)
- [키의 종류 (기본 키, 후보 키, 외래 키 등)](key-types.md)
- [무결성 제약조건 (개체 무결성, 참조 무결성 등)](integrity-constraints.md)

### 설계와 모델링

- [ER 모델과 관계형 모델 변환](er-to-relational-model.md)
- [정규화 1NF, 2NF, 3NF, BCNF (정규화의 실용적 이해)](normalization.md)
- [정규화 vs 반정규화 (Denormalization)](normalization-vs-denormalization.md)

### SQL 기초

- [SQL 기본 문법 (SELECT, INSERT, UPDATE, DELETE)](sql-basics.md)
- [WHERE, ORDER BY, GROUP BY, HAVING 절](sql-clauses.md)
- [조인 (INNER, OUTER, SELF, CROSS JOIN)](sql-joins.md)
- [서브쿼리와 IN, EXISTS](subqueries.md)

### 성능과 최적화

- [인덱스(Index)의 개념과 성능 영향](indexes.md)
- [실행 계획(Execution Plan)과 쿼리 최적화](execution-plan-optimization.md)

### 트랜잭션과 동시성

- [트랜잭션과 ACID](transactions-and-acid.md)
- [트랜잭션 격리 수준과 현상 (Dirty Read, Phantom Read 등)](transaction-isolation-levels.md)
- [동시성 제어 (Lock, MVCC 등)](concurrency-control.md)
- [데드락(Deadlock) 발생 조건 및 해결 방법](deadlock.md)

### 고급 개념

- [저장 프로시저와 트리거 (간단한 이해만)](stored-procedures-triggers.md)
- [분산 데이터베이스 개념 (샤딩, 파티셔닝, 레플리케이션)](distributed-databases.md)
- [CAP 이론 (Consistency, Availability, Partition Tolerance)](cap-theorem.md)
- [NoSQL의 기본 개념 (Key-Value, Document 중심)](nosql-basics.md)

### 운영과 관리

- [백업과 복구 전략 (논리/물리 백업, 트랜잭션 로그 등)](backup-recovery.md)

---

## 🎯 학습 가이드

이 문서들은 데이터베이스 전공 지식을 체계적으로 습득할 수 있도록 구성되어 있습니다:

1. **기본 개념**부터 시작하여 관계형 모델의 이론적 기초를 다집니다
2. **설계와 모델링**에서 실제 데이터베이스를 설계하는 방법을 학습합니다
3. **SQL 기초**에서 실무에 필요한 쿼리 작성 능력을 기릅니다
4. **성능과 최적화**에서 효율적인 데이터베이스 운영 방법을 익힙니다
5. **트랜잭션과 동시성**에서 안전한 데이터 처리 방법을 이해합니다
6. **고급 개념**에서 현대적인 데이터베이스 기술을 탐구합니다
7. **운영과 관리**에서 실제 운영 환경에서의 관리 방법을 학습합니다

각 문서는 이론적 배경부터 실제 구현까지 포괄적으로 다루고 있으며, 실무에서 바로 활용할 수 있는 예제와 베스트 프랙티스를 포함하고 있습니다.

---

_"데이터베이스는 단순한 저장소가 아니라, 데이터의 무결성과 일관성을 보장하는 지적 시스템입니다."_
