# 백업과 복구 전략 (Backup and Recovery Strategies)

## 개요

백업과 복구는 데이터베이스 시스템의 가용성과 내구성을 보장하는 핵심 메커니즘입니다. 시스템 장애, 인적 오류, 자연재해 등으로부터 데이터를 보호하고 신속한 복구를 가능하게 합니다.

## 백업의 필요성과 배경

### 데이터 손실 원인

- **하드웨어 장애**: 디스크 크래시, 메모리 오류
- **소프트웨어 오류**: OS 크래시, DBMS 버그
- **인적 오류**: 잘못된 DELETE, DROP 문 실행
- **자연재해**: 화재, 홍수, 지진
- **보안 침해**: 랜섬웨어, 데이터 변조

### 복구 목표

- **RTO (Recovery Time Objective)**: 허용 가능한 최대 복구 시간
- **RPO (Recovery Point Objective)**: 허용 가능한 최대 데이터 손실량
- **가용성 목표**: 99.9%, 99.99%, 99.999% 등

## 백업 유형

### 1. 논리적 백업 (Logical Backup)

```sql
-- PostgreSQL pg_dump
pg_dump -h localhost -U postgres -d mydb > backup.sql
pg_dump -h localhost -U postgres -d mydb -t employees > employees_backup.sql

-- MySQL mysqldump
mysqldump -u root -p mydb > backup.sql
mysqldump -u root -p mydb employees > employees_backup.sql

-- SQL Server SSMS Export
-- 또는 BCP 유틸리티
bcp mydb.dbo.employees out employees.txt -S server -U user -P password
```

### 2. 물리적 백업 (Physical Backup)

```bash
# PostgreSQL 물리적 백업
pg_basebackup -D /backup/pgdata -Ft -z -P -U postgres

# MySQL 물리적 백업 (InnoDB Hot Backup)
mysql> FLUSH TABLES WITH READ LOCK;
# 파일 시스템 복사
cp -r /var/lib/mysql/mydb /backup/mysql/

# SQL Server 백업
BACKUP DATABASE mydb TO DISK = 'C:\backup\mydb.bak'
WITH COMPRESSION, CHECKSUM;
```

### 3. 전체 백업 vs 증분 백업

```sql
-- 전체 백업 (Full Backup)
BACKUP DATABASE mydb TO DISK = 'C:\backup\mydb_full.bak';

-- 차등 백업 (Differential Backup)
BACKUP DATABASE mydb TO DISK = 'C:\backup\mydb_diff.bak'
WITH DIFFERENTIAL;

-- 트랜잭션 로그 백업 (Transaction Log Backup)
BACKUP LOG mydb TO DISK = 'C:\backup\mydb_log.trn';
```

## 트랜잭션 로그와 WAL

### Write-Ahead Logging (WAL) 원리

```sql
-- PostgreSQL WAL 설정
-- postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /archive/%f'
max_wal_size = 1GB
min_wal_size = 80MB

-- WAL 파일 확인
SELECT pg_current_wal_lsn();
SELECT pg_walfile_name(pg_current_wal_lsn());
```

### 로그 기반 복구

```sql
-- Point-in-Time Recovery (PITR)
-- 1. 기본 백업 복원
pg_basebackup -D /recovery/pgdata -Ft -z -P

-- 2. recovery.conf 설정
restore_command = 'cp /archive/%f %p'
recovery_target_time = '2024-01-15 14:30:00'

-- 3. PostgreSQL 시작 (복구 모드)
pg_ctl start -D /recovery/pgdata
```

## 복구 전략

### 1. 완전 복구 (Complete Recovery)

```sql
-- 모든 커밋된 트랜잭션 복구
-- SQL Server 예시
RESTORE DATABASE mydb FROM DISK = 'C:\backup\mydb_full.bak'
WITH REPLACE;

RESTORE LOG mydb FROM DISK = 'C:\backup\mydb_log1.trn'
WITH NORECOVERY;

RESTORE LOG mydb FROM DISK = 'C:\backup\mydb_log2.trn'
WITH RECOVERY;
```

### 2. 불완전 복구 (Incomplete Recovery)

```sql
-- 특정 시점까지만 복구
RESTORE DATABASE mydb FROM DISK = 'C:\backup\mydb_full.bak'
WITH NORECOVERY, REPLACE;

RESTORE LOG mydb FROM DISK = 'C:\backup\mydb_log.trn'
WITH STOPAT = '2024-01-15 14:30:00', RECOVERY;
```

### 3. 테이블 레벨 복구

```sql
-- PostgreSQL 테이블 단위 복구
pg_restore -d mydb -t employees backup.dump

-- MySQL 테이블 복구
mysql -u root -p mydb < employees_backup.sql

-- 선택적 복구
mysql> SOURCE employees_backup.sql;
```

## 고가용성 구성

### 1. 마스터-슬레이브 복제

```sql
-- MySQL 마스터 설정
-- my.cnf
[mysqld]
log-bin=mysql-bin
server-id=1
binlog-format=row

-- 슬레이브 설정
-- my.cnf
[mysqld]
server-id=2
relay-log=relay-log

-- 슬레이브 시작
mysql> CHANGE MASTER TO
  MASTER_HOST='master_host',
  MASTER_USER='replication_user',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=154;

mysql> START SLAVE;
```

### 2. 클러스터링

```sql
-- PostgreSQL 스트리밍 복제
-- postgresql.conf (Primary)
wal_level = hot_standby
max_wal_senders = 3
wal_keep_segments = 32

-- pg_hba.conf
host replication replicator standby_ip/32 md5

-- Standby 서버 설정
standby_mode = 'on'
primary_conninfo = 'host=primary_ip port=5432 user=replicator'
```

## 백업 스케줄링

### 1. 자동화 스크립트

```bash
#!/bin/bash
# PostgreSQL 자동 백업 스크립트
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/postgres"
DB_NAME="mydb"

# 전체 백업 (매일 자정)
if [ "$(date +%H)" = "00" ]; then
    pg_dump -h localhost -U postgres -d $DB_NAME >
    $BACKUP_DIR/${DB_NAME}_full_$DATE.sql

    # 오래된 백업 삭제 (7일 이상)
    find $BACKUP_DIR -name "*_full_*.sql" -mtime +7 -delete
fi

# WAL 아카이빙
cp /var/lib/postgresql/14/main/pg_wal/* $BACKUP_DIR/wal/
```

### 2. 백업 검증

```sql
-- 백업 무결성 확인
-- SQL Server
RESTORE VERIFYONLY FROM DISK = 'C:\backup\mydb.bak';

-- PostgreSQL 백업 테스트
pg_restore --list backup.dump
psql -d testdb < backup.sql 2>&1 | grep -i error
```

## 클라우드 백업

### 1. AWS RDS 백업

```bash
# 자동 백업 설정
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00"

# 스냅샷 생성
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot-$(date +%Y%m%d)
```

### 2. 하이브리드 백업

```bash
# 로컬 + 클라우드 백업
pg_dump mydb | gzip | aws s3 cp - s3://my-backup-bucket/db-backup-$(date +%Y%m%d).sql.gz

# 백업 보존 정책
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-backup-bucket \
  --lifecycle-configuration file://lifecycle.json
```

## 재해 복구 계획

### 1. DR 사이트 구성

```sql
-- 지리적 복제 설정
-- PostgreSQL 물리적 복제
# Primary (Seoul)
postgresql.conf:
wal_level = hot_standby
archive_mode = on
archive_command = 'rsync %p dr-site:/archive/%f'

# DR Site (Busan)
recovery.conf:
standby_mode = 'on'
restore_command = 'cp /archive/%f %p'
primary_conninfo = 'host=primary-seoul port=5432'
```

### 2. 장애 조치 절차

```bash
#!/bin/bash
# 자동 장애 조치 스크립트
check_primary() {
    pg_isready -h $PRIMARY_HOST -p 5432
}

failover() {
    echo "Primary 장애 감지, 장애 조치 시작"

    # Standby를 Primary로 승격
    pg_ctl promote -D $PGDATA

    # 애플리케이션 연결 전환
    update_app_config.sh

    # 모니터링 알림
    send_alert.sh "DR activated"
}

if ! check_primary; then
    failover
fi
```

## 성능 최적화

### 1. 백업 성능 튜닝

```sql
-- 병렬 백업
pg_dump -j 4 -Fd -f backup_dir mydb

-- 압축 백업
pg_dump -Z 9 mydb > backup.sql.gz

-- 백업 시 I/O 제한
ionice -c3 pg_dump mydb > backup.sql
```

### 2. 복구 성능 최적화

```sql
-- 병렬 복구
pg_restore -j 4 -d mydb backup_dir

-- 인덱스 재생성 지연
pg_restore --disable-triggers -d mydb backup.dump
-- 복구 후 인덱스 수동 재생성
```

## 백업 보안

### 1. 암호화 백업

```bash
# GPG 암호화
pg_dump mydb | gpg --cipher-algo AES256 --compress-algo 2 \
  --symmetric --output backup.sql.gpg

# 복호화 및 복구
gpg --decrypt backup.sql.gpg | psql -d mydb
```

### 2. 접근 제어

```sql
-- 백업 전용 사용자 생성
CREATE USER backup_user WITH PASSWORD 'secure_password';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO backup_user;
GRANT USAGE ON SCHEMA public TO backup_user;
```

## 모니터링과 알림

### 1. 백업 상태 모니터링

```sql
-- 마지막 백업 시간 확인
SELECT name, backup_finish_date
FROM msdb.dbo.backupset
WHERE database_name = 'mydb'
ORDER BY backup_finish_date DESC;

-- PostgreSQL WAL 상태
SELECT pg_current_wal_lsn(), pg_current_wal_insert_lsn();
```

### 2. 알림 시스템

```bash
#!/bin/bash
# 백업 실패 알림
if [ $? -ne 0 ]; then
    echo "백업 실패: $(date)" | mail -s "DB Backup Failed" admin@company.com
    curl -X POST -H 'Content-type: application/json' \
      --data '{"text":"Database backup failed"}' \
      $SLACK_WEBHOOK_URL
fi
```

## 결론

효과적인 백업과 복구 전략은 비즈니스 연속성을 보장하는 핵심 요소입니다. 조직의 RTO/RPO 요구사항에 맞는 적절한 백업 방식을 선택하고, 정기적인 복구 테스트를 통해 전략의 실효성을 검증해야 합니다. 클라우드 기반 솔루션과 자동화를 활용하여 보다 안정적이고 효율적인 백업 체계를 구축하는 것이 중요합니다.
