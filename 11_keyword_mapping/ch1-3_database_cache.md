# Chapter 1-3. 데이터베이스 및 캐시

## 0. 한 줄 요약

🔑 **"데이터 모델(관계형/키-값/문서/그래프) × 관리 수준(자체 운영/관리형/서버리스) × 읽기 패턴(캐시 필요 여부)"이 DB 선택 기준이다**

---

## 1. 핵심 서비스 표

| 서비스 | 한 줄 정의 | 주요 키워드 | 가격 모델 |
|--------|-----------|------------|----------|
| **RDS** | 관리형 관계형 DB (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server) | Multi-AZ, Read Replica, 자동 백업, 스냅샷 | 인스턴스 시간 + 스토리지 + I/O |
| **Aurora** | AWS 자체 설계 고성능 관계형 DB (MySQL/PostgreSQL 호환) | 6개 복제본, 자동 확장 스토리지, Global Database, Serverless v2 | 인스턴스 시간 + 스토리지 + I/O (Serverless: ACU) |
| **DynamoDB** | 완전 관리형 키-값/문서 NoSQL DB | 밀리초 지연, DAX, Global Tables, TTL, Streams | 온디맨드 / 프로비저닝 (RCU+WCU) |
| **ElastiCache** | 인메모리 캐시 (Redis OSS / Memcached) | 마이크로초 지연, 세션 스토어, 리더보드 | 노드 시간 + 데이터 전송 |
| **Neptune** | 관리형 그래프 DB | 소셜 네트워크, 추천 엔진, 사기 탐지, Gremlin/SPARQL | 인스턴스 시간 + 스토리지 + I/O |
| **DocumentDB** | 관리형 문서 DB (MongoDB 호환) | MongoDB 워크로드 마이그레이션, JSON 문서 | 인스턴스 시간 + 스토리지 + I/O |
| **Redshift** | 페타바이트급 데이터 웨어하우스 | OLAP, 컬럼형 저장, Spectrum(S3 직접 쿼리), RA3 | 노드 시간 (+ Spectrum: 스캔 데이터량) |

---

## 2. 키워드 → 서비스 매핑

### RDS가 정답인 키워드

| 시험 키워드 | 왜 RDS인가 |
|------------|-----------|
| "관계형 DB", "SQL", "ACID" | 전통적 RDBMS 관리형 서비스 |
| "Multi-AZ 배포" | 자동 장애 조치, 동기식 복제 |
| "Read Replica" | 읽기 부하 분산 (비동기 복제) |
| "Oracle", "SQL Server", "MariaDB" | Aurora가 지원 안 하는 엔진 |
| "기존 RDB를 최소 변경으로 AWS에" | 동일 엔진 → RDS 마이그레이션 |
| "자동 백업", "35일 보존" | RDS 자동 백업 (최대 35일) |

### Aurora가 정답인 키워드

| 시험 키워드 | 왜 Aurora인가 |
|------------|-------------|
| "MySQL/PostgreSQL 호환" + "고성능" | RDS 대비 MySQL 5배, PostgreSQL 3배 성능 |
| "자동 확장 스토리지" (최대 128 TB) | 10 GB 단위 자동 확장, 관리 불필요 |
| "6개 복제본", "3개 AZ" | 데이터가 3개 AZ에 6개 사본 |
| "Global Database" | 리전 간 1초 미만 복제, 글로벌 DR |
| "서버리스 DB" + "관계형" | Aurora Serverless v2 (자동 ACU 스케일링) |
| "간헐적 워크로드" + "관계형 DB" | Aurora Serverless (유휴 시 0까지 축소 가능) |
| "Writer + 최대 15개 Reader" | Aurora는 Read Replica 최대 15개 (RDS는 5개) |

### DynamoDB가 정답인 키워드

| 시험 키워드 | 왜 DynamoDB인가 |
|------------|---------------|
| "키-값 스토어", "NoSQL" | 완전 관리형 NoSQL |
| "밀리초 단위 일관된 응답 시간" | 단일 자릿수 밀리초 지연 |
| "마이크로초 지연" + NoSQL | DynamoDB + **DAX** (인메모리 캐시) |
| "서버리스 NoSQL" | 용량 관리 불필요 (온디맨드 모드) |
| "Global Tables" | 다중 리전 다중 마스터 복제 |
| "TTL (Time To Live)" | 자동 만료 삭제 |
| "DynamoDB Streams" | 변경 이벤트 캡처 → Lambda 트리거 |
| "세션 상태 저장" + "서버리스" | 서버리스 세션 스토어로 활용 |
| "무제한 확장성" + "NoSQL" | 자동 파티셔닝, 용량 제한 없음 |

### ElastiCache가 정답인 키워드

| 시험 키워드 | 왜 ElastiCache인가 |
|------------|------------------|
| "인메모리 캐시" | Redis OSS / Memcached 관리형 |
| "DB 읽기 부하 경감" | 자주 조회되는 데이터를 캐시 |
| "세션 스토어" | 웹 앱 세션을 인메모리에 저장 |
| "리더보드", "실시간 순위" | Redis Sorted Set 활용 |
| "마이크로초 응답 시간" (범용) | 인메모리 특성 |
| "Pub/Sub 메시징" (간단한 경우) | Redis Pub/Sub |

### ElastiCache: Redis OSS vs Memcached

| 기준 | Redis OSS | Memcached |
|------|-----------|-----------|
| 데이터 구조 | 다양 (String, Hash, Set, Sorted Set, List) | 단순 키-값 |
| 복제 | 지원 (Multi-AZ, Read Replica) | 미지원 |
| 지속성 | AOF/RDB 백업 가능 | 미지원 (순수 캐시) |
| 클러스터링 | Redis Cluster | 멀티스레드, 수평 확장 |
| 사용 사례 | 세션, 리더보드, Pub/Sub, 지속성 필요 | 단순 캐시, 멀티스레드 필요 |

> 💡 **시험 팁**: 대부분 Redis가 정답. "단순 캐시 + 멀티스레드" 명시 시에만 Memcached.

### Neptune이 정답인 키워드

| 시험 키워드 | 왜 Neptune인가 |
|------------|--------------|
| "그래프 DB", "관계 탐색" | 네이티브 그래프 엔진 |
| "소셜 네트워크", "추천 엔진" | 노드-엣지 관계 모델 |
| "사기 탐지", "패턴 매칭" | 복잡한 관계 그래프 분석 |
| "Gremlin", "SPARQL" | 그래프 쿼리 언어 |
| "지식 그래프" | RDF 트리플 스토어 |

### DocumentDB가 정답인 키워드

| 시험 키워드 | 왜 DocumentDB인가 |
|------------|-----------------|
| "MongoDB 호환", "MongoDB 마이그레이션" | MongoDB API 호환 |
| "JSON 문서 저장" + "관리형" | 문서 DB 관리형 서비스 |
| "MongoDB" + "AWS 관리형" | 자체 MongoDB 관리 부담 제거 |

### Redshift가 정답인 키워드

| 시험 키워드 | 왜 Redshift인가 |
|------------|---------------|
| "데이터 웨어하우스", "OLAP" | 분석 전용 컬럼형 저장소 |
| "페타바이트 규모 분석" | 대용량 분석 최적화 |
| "비즈니스 인텔리전스(BI)" | QuickSight + Redshift 조합 |
| "S3에서 직접 쿼리" (대용량) | Redshift Spectrum |
| "컬럼형 저장", "MPP" | 병렬 처리 아키텍처 |
| "ETL 후 분석" | 데이터 적재 → 분석 파이프라인 |

> ⚠️ **Redshift vs Athena 구분**: Athena = S3 위 즉석 쿼리(서버리스), Redshift = 전용 클러스터에 데이터 적재 후 반복 분석

---

## 3. 헷갈리는 포인트 / 함정

### 함정 1: RDS Multi-AZ vs Read Replica

- ❌ "읽기 성능 향상" → Multi-AZ 선택 → **틀림**
- ✅ **Multi-AZ**: 고가용성 목적, 동기식 복제, 자동 장애 조치 → **성능 향상 아님**
- ✅ **Read Replica**: 읽기 부하 분산, 비동기 복제 → **성능 향상 목적**
- 💡 "고가용성" → Multi-AZ, "읽기 성능" → Read Replica, 둘 다 가능

### 함정 2: Aurora vs RDS 선택 기준

- ❌ "관계형 DB면 무조건 Aurora" → **엔진에 따라 다름**
- ✅ Aurora: MySQL / PostgreSQL만 지원
- ✅ Oracle, SQL Server, MariaDB → **RDS만 가능**
- 💡 "Oracle 라이선스" 또는 "SQL Server" 키워드 → Aurora 제외

### 함정 3: DynamoDB 온디맨드 vs 프로비저닝

- ❌ "비용 최적화" → 무조건 온디맨드 → **예측 가능하면 프로비저닝이 더 저렴**
- ✅ **온디맨드**: 트래픽 예측 불가, 간헐적 워크로드
- ✅ **프로비저닝**: 예측 가능한 트래픽, 비용 절감 (+ Auto Scaling)
- 💡 "예측 불가능한 트래픽" → 온디맨드, "안정적 트래픽" + "비용 절감" → 프로비저닝

### 함정 4: ElastiCache vs DAX

- ❌ "DynamoDB 읽기 성능 향상" → ElastiCache → **DAX가 더 적합**
- ✅ **DAX**: DynamoDB 전용 인메모리 캐시, API 변경 불필요
- ✅ **ElastiCache**: 범용 인메모리 캐시, RDS/Aurora 등에 적합
- 💡 "DynamoDB" + "캐시" → DAX, 그 외 DB + "캐시" → ElastiCache

### 함정 5: Redshift vs Athena

- ❌ "S3 데이터 분석" → 무조건 Athena → **데이터 규모와 패턴에 따라 다름**
- ✅ **Athena**: 서버리스, 즉석 쿼리, 간헐적 분석, 스캔 데이터량 과금
- ✅ **Redshift**: 전용 클러스터, 반복적 대규모 분석, 데이터 적재 필요
- ✅ **Redshift Spectrum**: Redshift 클러스터에서 S3 직접 쿼리 (하이브리드)
- 💡 "서버리스 쿼리" + "간헐적" → Athena, "반복 분석" + "대규모" → Redshift

### 함정 6: Aurora Serverless의 적용 범위

- ❌ "관계형 DB + 서버리스" → 무조건 Aurora Serverless → **안정적 트래픽이면 불필요**
- ✅ Aurora Serverless v2: 간헐적/예측 불가 워크로드에 적합
- ✅ 안정적 고트래픽: 프로비저닝 Aurora가 비용 효율적
- 💡 "간헐적", "개발/테스트 환경", "트래픽 급증" → Aurora Serverless

---

## 4. 단골 시나리오

### 시나리오 1: 글로벌 웹 애플리케이션 DB

```
문제 패턴: "전 세계 사용자, 로컬 읽기 성능, 리전 간 자동 복제"
정답 패턴: Aurora Global Database (리전 간 1초 미만 복제)
대안: DynamoDB Global Tables (NoSQL이 적합한 경우)
키 포인트: "글로벌" + "관계형" → Aurora Global, "글로벌" + "NoSQL" → DynamoDB Global Tables
```

### 시나리오 2: 읽기 집중 웹 애플리케이션

```
문제 패턴: "읽기 80%, 쓰기 20%, DB 부하 경감"
정답 패턴: RDS/Aurora Read Replica + ElastiCache
키 포인트: Read Replica만으로 부족하면 ElastiCache 추가
```

### 시나리오 3: 실시간 게임 리더보드

```
문제 패턴: "실시간 순위 업데이트, 밀리초 응답"
정답 패턴: ElastiCache for Redis (Sorted Set)
키 포인트: "리더보드" = Redis가 거의 확정
```

### 시나리오 4: IoT 센서 데이터 대량 수집

```
문제 패턴: "초당 수만 건 쓰기, 가변적 스키마, 자동 확장"
정답 패턴: DynamoDB (온디맨드 모드)
키 포인트: 대량 쓰기 + 가변 스키마 + 자동 확장 = DynamoDB
```

### 시나리오 5: 소셜 네트워크 친구 추천

```
문제 패턴: "사용자 간 관계 탐색, 친구의 친구 추천"
정답 패턴: Neptune (그래프 DB)
키 포인트: "관계 탐색", "추천" = 그래프 DB = Neptune
```

### 시나리오 6: MongoDB → AWS 마이그레이션

```
문제 패턴: "기존 MongoDB, 관리 부담 줄이면서 AWS로"
정답 패턴: DocumentDB (MongoDB 호환)
키 포인트: "MongoDB" + "관리형" = DocumentDB
```

### 시나리오 7: BI 대시보드용 대규모 분석

```
문제 패턴: "여러 데이터 소스 → 집계 → 반복 대시보드 쿼리"
정답 패턴: Redshift (+ QuickSight 시각화)
키 포인트: "반복 분석" + "BI" → Redshift, 즉석 쿼리면 Athena
```

---

## 5. 검증 필요 항목 ⚠️

> Claude의 학습 데이터가 outdated일 수 있는 항목. AWS 공식 문서에서 확인 필요.

- [ ] Aurora 스토리지 최대 한도: 128 TB인지 확인
  - 공식 문서: https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_Limits.html
- [ ] Aurora Read Replica 최대 개수: 15개인지 확인
- [ ] Aurora Serverless v2 최소/최대 ACU 범위 확인
- [ ] DynamoDB 온디맨드 vs 프로비저닝 최신 요금 비교
- [ ] DynamoDB Global Tables 최신 버전 (2019.11.21) 기능 확인
- [ ] DAX 최신 지원 리전 및 가격
- [ ] Redshift RA3 인스턴스 타입 최신 사양 확인
- [ ] Redshift Serverless 최신 상태 (정식 출시 여부, 요금)
- [ ] ElastiCache for Redis vs ElastiCache for Valkey 변경 사항 확인
- [ ] Neptune Serverless 지원 여부 확인
- [ ] RDS Multi-AZ 클러스터 (db.r5d 기반) 최신 지원 엔진 확인

---

## 6. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "관계형 DB + 고가용성" → RDS Multi-AZ 또는 Aurora
- **유형 2**: "읽기 성능 향상" → Read Replica + ElastiCache
- **유형 3**: "서버리스 NoSQL" → DynamoDB 온디맨드
- **유형 4**: "글로벌 DB 복제" → Aurora Global Database / DynamoDB Global Tables
- **유형 5**: "인메모리 캐시 + 리더보드" → ElastiCache for Redis
- **유형 6**: "DynamoDB 읽기 캐시" → DAX
- **유형 7**: "그래프 DB / 관계 탐색" → Neptune
- **유형 8**: "MongoDB 마이그레이션" → DocumentDB
- **유형 9**: "데이터 웨어하우스 / OLAP" → Redshift
- **유형 10**: "간헐적 관계형 워크로드 + 서버리스" → Aurora Serverless v2

---

## 7. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
