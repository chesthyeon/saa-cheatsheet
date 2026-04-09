# Chapter 3-4. 데이터베이스: RDS vs Aurora vs DynamoDB vs ElastiCache vs Redshift

## 0. 한 줄 요약

🔑 **"데이터 모델(관계형/키-값/인메모리/분석) × 워크로드 패턴(OLTP/OLAP/캐시)"으로 DB가 결정된다 — 관계형 안에서도 RDS vs Aurora는 엔진과 성능 요구로 갈린다**

---

## 1. 전체 비교 매트릭스

| 기준 | RDS | Aurora | DynamoDB | ElastiCache | Redshift |
|------|-----|--------|----------|-------------|----------|
| **유형** | 관계형 (RDBMS) | 관계형 (AWS 설계) | NoSQL (키-값/문서) | 인메모리 캐시 | 데이터 웨어하우스 |
| **워크로드** | OLTP | OLTP (고성능) | OLTP (대규모) | 캐시/세션 | **OLAP** (분석) |
| **엔진** | MySQL, PostgreSQL, MariaDB, **Oracle, SQL Server** | MySQL, PostgreSQL **호환만** | 독자 엔진 | Redis OSS, Memcached | 독자 엔진 (컬럼형) |
| **서버리스** | ❌ (RDS Proxy만) | ✅ **Serverless v2** | ✅ 온디맨드 모드 | ❌ | ✅ Serverless |
| **스토리지** | 수동 확장 (최대 64 TB) | **자동 확장** (최대 128 TB) | 자동 (무제한) | 인메모리 | 수동 확장 (RA3) |
| **Read Replica** | 최대 **5개** | 최대 **15개** | 해당 없음 | Redis: Read Replica | 해당 없음 |
| **Multi-AZ** | ✅ 동기 복제 (장애 조치) | ✅ 6개 사본/3개 AZ | ✅ 자동 (다중 AZ) | ✅ Redis Multi-AZ | ✅ Multi-AZ |
| **글로벌** | Cross-Region Read Replica | **Global Database** (1초 미만) | **Global Tables** (다중 마스터) | Global Datastore | ❌ |
| **지연 시간** | 밀리초 | 밀리초 (RDS보다 낮음) | **단일 자릿수 밀리초** | **마이크로초** | 초~분 (쿼리 규모 따라) |
| **비용 모델** | 인스턴스 + 스토리지 | 인스턴스 + I/O (Serverless: ACU) | RCU/WCU 또는 온디맨드 | 노드 시간 | 노드 시간 |

---

## 2. 핵심 판별 플로우차트

```
"DB 선택"
  │
  ├─ 관계형 (SQL, ACID, 조인)?
  │   ├─ Oracle / SQL Server / MariaDB? ──→ RDS
  │   ├─ MySQL/PostgreSQL + 고성능? ──→ Aurora
  │   ├─ MySQL/PostgreSQL + 서버리스? ──→ Aurora Serverless v2
  │   └─ MySQL/PostgreSQL + 기본 기능? ──→ RDS (비용 절감)
  │
  ├─ NoSQL (키-값, 유연한 스키마)?
  │   ├─ 밀리초 지연 + 자동 확장? ──→ DynamoDB
  │   └─ 마이크로초 지연 (DynamoDB 캐시)? ──→ DAX
  │
  ├─ 인메모리 캐시?
  │   ├─ DB 읽기 부하 경감? ──→ ElastiCache
  │   ├─ 리더보드/세션? ──→ ElastiCache for Redis
  │   └─ 단순 캐시 + 멀티스레드? ──→ ElastiCache for Memcached
  │
  └─ 분석 (OLAP, BI)?
      ├─ 대규모 반복 분석? ──→ Redshift
      └─ 서버리스 즉석 쿼리? ──→ Athena (Ch1-5 참고)
```

---

## 3. 쌍별 상세 비교

### RDS vs Aurora

| 기준 | RDS | Aurora |
|------|-----|--------|
| 지원 엔진 | MySQL, PostgreSQL, MariaDB, **Oracle, SQL Server** | MySQL, PostgreSQL **호환만** |
| 성능 | 기본 | MySQL 5배, PostgreSQL 3배 |
| 스토리지 | 수동 확장 (최대 64 TB) | **자동 확장** (최대 128 TB) |
| 복제본 | 최대 5개 Read Replica | 최대 **15개** Read Replica |
| 장애 조치 | ~60초 | **~30초** (빠름) |
| 글로벌 복제 | Cross-Region Read Replica (분 단위 지연) | **Global Database** (1초 미만 지연) |
| 서버리스 | ❌ | ✅ Serverless v2 |
| 비용 | Aurora보다 저렴 (동일 사양) | RDS보다 ~20% 비쌈 (성능 대비 효율적) |

**판별 핵심:**

| 시험 키워드 | 정답 |
|------------|------|
| "Oracle", "SQL Server" | **RDS** (Aurora 미지원) |
| "MySQL/PostgreSQL" + "고성능" | **Aurora** |
| "자동 확장 스토리지" | **Aurora** |
| "15개 Read Replica" | **Aurora** |
| "Global Database" (1초 미만 복제) | **Aurora** |
| "서버리스 관계형 DB" | **Aurora Serverless v2** |
| "비용 절감" + "기본 MySQL" | **RDS** (Aurora보다 저렴) |

### DynamoDB vs RDS/Aurora

| 기준 | DynamoDB | RDS/Aurora |
|------|----------|-----------|
| 데이터 모델 | 키-값 / 문서 (NoSQL) | 관계형 (SQL) |
| 스키마 | **유연** (항목마다 다를 수 있음) | 고정 (테이블 스키마) |
| 조인 | ❌ | ✅ |
| 트랜잭션 | 제한적 (TransactWriteItems) | 완전한 ACID |
| 확장성 | **무제한** 자동 확장 | 수직 확장 (인스턴스 크기) + Read Replica |
| 서버 관리 | 없음 (완전 관리형) | 인스턴스 관리 (패치 등) |

**판별 핵심:**

| 시험 키워드 | 정답 |
|------------|------|
| "SQL", "조인", "복잡한 쿼리", "ACID" | **RDS/Aurora** |
| "키-값", "유연한 스키마", "무제한 확장" | **DynamoDB** |
| "밀리초 지연" + "NoSQL" | **DynamoDB** |
| "서버리스 DB" + "NoSQL" | **DynamoDB 온디맨드** |

### ElastiCache vs DAX

| 기준 | ElastiCache | DAX |
|------|-------------|-----|
| 대상 DB | **범용** (RDS, Aurora 등) | **DynamoDB 전용** |
| API 변경 | 애플리케이션 코드 수정 필요 | **코드 변경 최소** (DynamoDB API 호환) |
| 프로토콜 | Redis/Memcached | DynamoDB 호환 |
| 사용 사례 | DB 읽기 캐시, 세션, 리더보드 | DynamoDB 읽기 가속 |

> 💡 **판별**: "DynamoDB" + "캐시" → **DAX**, 그 외 DB + "캐시" → **ElastiCache**

### ElastiCache: Redis vs Memcached

| 기준 | Redis OSS | Memcached |
|------|-----------|-----------|
| 데이터 구조 | String, Hash, Set, **Sorted Set**, List | 단순 키-값 |
| 복제/HA | ✅ Multi-AZ, Read Replica | ❌ |
| 지속성 | ✅ AOF/RDB 백업 | ❌ (순수 캐시) |
| Pub/Sub | ✅ | ❌ |
| 멀티스레드 | ❌ (단일 스레드) | ✅ |
| **시험 정답 비율** | **~95%** | ~5% |

> 💡 **판별**: 대부분 **Redis**가 정답. "단순 캐시" + "멀티스레드" 명시 시에만 Memcached.

### Redshift vs Athena

| 기준 | Redshift | Athena |
|------|----------|--------|
| 인프라 | 전용 클러스터 (또는 Serverless) | **서버리스** (인프라 없음) |
| 데이터 위치 | Redshift 클러스터에 적재 | **S3에서 직접 쿼리** |
| 쿼리 패턴 | 반복적 대규모 분석, BI 대시보드 | 즉석(ad-hoc) 쿼리, 간헐적 |
| 성능 | 대규모 조인/집계에 최적화 | 단순 스캔/필터에 적합 |
| 비용 | 노드 시간 (항상 비용 발생) | **스캔 데이터량** (쿼리할 때만) |
| 데이터 적재 | 필요 (COPY 명령) | **불필요** |

**판별 핵심:**

| 시험 키워드 | 정답 |
|------------|------|
| "서버리스 쿼리", "간헐적 분석", "S3 직접" | **Athena** |
| "반복 BI", "페타바이트 분석", "데이터 웨어하우스" | **Redshift** |
| "S3 데이터를 Redshift에서 쿼리" | **Redshift Spectrum** |

---

## 4. 시험 빈출 시나리오 + 정답

| 시나리오 | 정답 | 오답 함정 |
|----------|------|----------|
| Oracle DB를 AWS 관리형으로 | **RDS for Oracle** | Aurora (Oracle 미지원) |
| MySQL + 고성능 + 글로벌 복제 | **Aurora Global Database** | RDS (성능/글로벌 기능 부족) |
| 간헐적 관계형 워크로드, 유휴 시 비용 0 | **Aurora Serverless v2** | RDS (항상 인스턴스 비용) |
| 초당 수만 건 쓰기, 유연한 스키마 | **DynamoDB** | RDS (확장성 한계) |
| DynamoDB 읽기 지연 → 마이크로초 필요 | **DAX** | ElastiCache (코드 변경 필요) |
| 웹 앱 세션 스토어, 실시간 리더보드 | **ElastiCache for Redis** | Memcached (기능 부족) |
| 여러 데이터 소스 → BI 대시보드 반복 분석 | **Redshift** | Athena (반복 분석에 비효율) |
| CloudTrail 로그 S3에서 간헐적 SQL 쿼리 | **Athena** | Redshift (과도) |
| 글로벌 NoSQL, 다중 리전 쓰기 | **DynamoDB Global Tables** | Aurora Global (단일 Writer) |
| 읽기 80% 워크로드, DB 부하 경감 | **Read Replica + ElastiCache** | Multi-AZ (성능 향상 아님) |

---

## 5. 헷갈리는 포인트 / 함정

### 함정 1: Multi-AZ ≠ 성능 향상

- ❌ "성능 향상" → Multi-AZ → **틀림** (대기 인스턴스는 읽기 불가)
- ✅ Multi-AZ = **고가용성** (장애 조치)
- ✅ 성능 향상 = **Read Replica**
- 💡 "고가용성" → Multi-AZ, "읽기 성능" → Read Replica

### 함정 2: Aurora Global Database vs DynamoDB Global Tables

- ✅ **Aurora Global**: 1개 Writer 리전 + 여러 Reader 리전 (단일 쓰기 지점)
- ✅ **DynamoDB Global Tables**: **다중 리전 다중 마스터** (어디서든 쓰기 가능)
- 💡 "다중 리전 쓰기" → DynamoDB Global Tables

### 함정 3: DynamoDB 온디맨드 vs 프로비저닝

- ✅ **온디맨드**: 예측 불가 트래픽, 간헐적 → 편리하지만 단가 높음
- ✅ **프로비저닝 + Auto Scaling**: 예측 가능 → 비용 절감
- 💡 "비용 최적화" + "안정적 트래픽" → 프로비저닝

### 함정 4: Redshift vs RDS (OLTP vs OLAP)

- ❌ "대규모 데이터 분석" → RDS → **틀림** (OLTP에 최적화된 DB)
- ✅ OLTP (트랜잭션 처리) → RDS/Aurora/DynamoDB
- ✅ OLAP (분석/집계) → **Redshift**
- 💡 "분석", "BI", "집계", "OLAP" → Redshift

---

## 6. 검증 필요 항목 ⚠️

- [ ] Aurora 스토리지 최대 128 TB 변경 여부
- [ ] Aurora Serverless v2 최소/최대 ACU, 0까지 축소 가능 여부
- [ ] DynamoDB 온디맨드 최신 요금
- [ ] ElastiCache for Valkey (Redis 대체) 변경 사항
- [ ] Redshift Serverless 최신 기능/요금
- [ ] RDS Multi-AZ 클러스터 (2개 읽기 가능 대기 인스턴스) 최신 지원 엔진

---

## 7. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "관계형 DB 엔진 선택" → RDS vs Aurora (엔진으로 판별)
- **유형 2**: "관계형 vs NoSQL" → RDS/Aurora vs DynamoDB
- **유형 3**: "인메모리 캐시 선택" → ElastiCache vs DAX
- **유형 4**: "OLTP vs OLAP" → RDS/Aurora vs Redshift
- **유형 5**: "글로벌 복제" → Aurora Global vs DynamoDB Global Tables

---

## 8. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
