# Chapter 4-5. 데이터 레이크 + 분석 (S3 + Kinesis + Glue + Athena + QuickSight)

## 0. 한 줄 요약

🔑 **"수집 → 저장 → 변환 → 분석 → 시각화" 파이프라인 = Kinesis(수집) + S3(저장) + Glue(ETL/카탈로그) + Athena(쿼리) + QuickSight(대시보드) — 각 단계별 서비스를 매핑하면 끝**

---

## 1. 아키텍처 다이어그램

```
[데이터 소스]
  ├─ 실시간: IoT, 클릭스트림, 로그
  ├─ 배치: DB 덤프, CSV, 로그 파일
  └─ DB: RDS, DynamoDB
       │
       ▼
┌── 수집 (Ingestion) ──┐
│ Kinesis Data Streams  │ ← 실시간 스트리밍
│ Kinesis Data Firehose │ ← 준실시간 전달 (S3/Redshift)
│ AWS DMS              │ ← DB 마이그레이션/CDC
│ S3 Transfer / Snow   │ ← 대용량 배치
└───────────────────────┘
       │
       ▼
┌── 저장 (Storage) ────┐
│ Amazon S3             │ ← 데이터 레이크 중심
│ (Raw / Processed /    │
│  Curated 계층)        │
└───────────────────────┘
       │
       ▼
┌── 변환 (ETL) ────────┐
│ AWS Glue              │ ← 서버리스 ETL + 카탈로그
│ Glue Data Catalog     │ ← 메타데이터 저장소
│ EMR                   │ ← 대규모 Spark/Hadoop
│ Lambda                │ ← 경량 변환
└───────────────────────┘
       │
       ▼
┌── 분석 (Analytics) ──┐
│ Athena                │ ← S3 직접 SQL 쿼리 (서버리스)
│ Redshift              │ ← 데이터 웨어하우스 (대규모)
│ Redshift Spectrum     │ ← S3 데이터 Redshift에서 쿼리
│ OpenSearch            │ ← 로그/텍스트 검색 분석
└───────────────────────┘
       │
       ▼
┌── 시각화 (BI) ───────┐
│ QuickSight            │ ← 서버리스 BI 대시보드
└───────────────────────┘
```

---

## 2. 단계별 서비스 상세

### 2-1. 수집 (Ingestion)

| 서비스 | 유형 | 설명 | 시험 키워드 |
|--------|------|------|-----------|
| **Kinesis Data Streams** | 실시간 | 커스텀 소비자, 샤드 기반 | "실시간 처리", "커스텀 소비자" |
| **Kinesis Data Firehose** | 준실시간 | S3/Redshift/OpenSearch 자동 전달 | "S3로 전달", "가장 간단한 수집" |
| **DMS** | 배치/CDC | DB → DB 또는 DB → S3 마이그레이션 | "DB 마이그레이션", "CDC" |
| **S3 Transfer Acceleration** | 배치 | 글로벌 업로드 가속 | "원격지 업로드 속도" |
| **Snow Family** | 오프라인 | TB~PB 물리 전송 | "대용량", "네트워크 불가" |
| **IoT Core** | 실시간 | IoT 디바이스 데이터 수집 | "IoT 센서", "MQTT" |

**Kinesis Data Streams vs Firehose 핵심 비교:**

| 기준 | Data Streams | Firehose |
|------|-------------|----------|
| 관리 | 샤드 수동 관리 (On-Demand 모드 가능) | **완전 관리형** |
| 지연 | **~200ms** (실시간) | **60초~** (버퍼링) |
| 소비자 | **커스텀** (Lambda, KCL, Spark) | **S3, Redshift, OpenSearch, Splunk** |
| 변환 | ❌ (소비자에서 처리) | ✅ Lambda 변환 내장 |
| 보존 | 24시간~365일 | ❌ (전달 후 삭제) |
| 사용 사례 | 커스텀 실시간 처리 | **S3로 자동 적재** |

💡 "S3로 가장 간단하게 스트리밍 데이터 저장" → **Kinesis Data Firehose**

### 2-2. 저장 (Storage) — S3 데이터 레이크 계층

```
S3 데이터 레이크
├── Raw Zone (원본)      ← 수집된 그대로 (JSON, CSV, 로그)
├── Processed Zone (변환) ← Glue ETL 후 (Parquet, ORC)
└── Curated Zone (정제)   ← 분석용 최종 데이터
```

| 설정 | 설명 | 시험 키워드 |
|------|------|-----------|
| **S3 Lifecycle** | 계층 간 자동 전환 (IA → Glacier) | "비용 최적화", "자동 아카이빙" |
| **S3 Versioning** | 실수 복구 | "데이터 보호" |
| **S3 Object Lock** | WORM (쓴 후 수정 불가) | "규정 준수", "변조 방지" |
| **Parquet/ORC 포맷** | 컬럼형 포맷 (Athena 최적화) | "쿼리 비용 절감", "성능 최적화" |

### 2-3. 변환 (ETL)

| 서비스 | 설명 | 시험 키워드 |
|--------|------|-----------|
| **Glue ETL** | 서버리스 Spark 기반 ETL | "서버리스 ETL", "변환" |
| **Glue Data Catalog** | 메타데이터 저장소 (Hive 호환) | "스키마", "메타데이터", "카탈로그" |
| **Glue Crawlers** | 자동 스키마 탐색 | "자동 스키마 검색" |
| **EMR** | 대규모 Spark/Hadoop/Presto | "빅데이터", "Hadoop", "Spark" |
| **Lake Formation** | 데이터 레이크 권한 관리 | "데이터 레이크 보안", "세밀한 접근 제어" |

**Glue vs EMR:**

| 기준 | Glue | EMR |
|------|------|-----|
| 관리 | **서버리스** | 클러스터 관리 (EC2) |
| 사용 사례 | 일반 ETL | **대규모 빅데이터**, 커스텀 처리 |
| 비용 | DPU 시간당 | 인스턴스 시간 |
| 복잡도 | 낮음 | 높음 (설정 자유도↑) |

💡 "서버리스 ETL" → Glue, "대규모 Spark/Hadoop" → EMR

### 2-4. 분석 (Analytics)

| 서비스 | 설명 | 시험 키워드 |
|--------|------|-----------|
| **Athena** | S3 직접 SQL 쿼리 (서버리스) | "S3 쿼리", "서버리스 분석", "ad-hoc" |
| **Redshift** | 데이터 웨어하우스 (페타바이트급) | "데이터 웨어하우스", "OLAP", "복잡한 조인" |
| **Redshift Spectrum** | Redshift에서 S3 직접 쿼리 | "Redshift + S3 쿼리" |
| **OpenSearch** | 로그/텍스트 검색 + 시각화 | "로그 분석", "검색", "Kibana" |

**Athena vs Redshift:**

| 기준 | Athena | Redshift |
|------|--------|----------|
| 인프라 | **서버리스** | 클러스터 (Serverless 옵션 있음) |
| 비용 | **스캔 데이터량** ($5/TB) | 노드 시간 |
| 사용 사례 | **Ad-hoc 쿼리**, 가끔 분석 | **지속적 분석**, 복잡한 조인 |
| 성능 | 쿼리당 수초~분 | 매우 빠름 (사전 로드) |
| 데이터 위치 | **S3** (이동 불필요) | Redshift 내부 (로드 필요) |

💡 "S3 데이터를 가끔 분석" → **Athena**, "지속적 대규모 분석" → **Redshift**

### 2-5. 시각화 (BI)

| 서비스 | 설명 | 시험 키워드 |
|--------|------|-----------|
| **QuickSight** | 서버리스 BI 대시보드 | "대시보드", "시각화", "BI" |
| SPICE | QuickSight 인메모리 엔진 | "빠른 대시보드", "캐시" |

---

## 3. 시험 빈출 파이프라인 조합

### 조합 1: 실시간 로그 분석

```
CloudWatch Logs → Kinesis Data Firehose → S3 → Athena
```

### 조합 2: 클릭스트림 실시간 처리

```
클릭스트림 → Kinesis Data Streams → Lambda (변환) → DynamoDB (실시간)
                                                   → S3 (배치 분석)
```

### 조합 3: DB → 데이터 레이크

```
RDS → DMS (CDC) → S3 (Raw) → Glue ETL → S3 (Processed/Parquet) → Athena
```

### 조합 4: IoT 데이터 파이프라인

```
IoT 센서 → IoT Core → Kinesis Data Firehose → S3 → Athena + QuickSight
```

### 조합 5: 풀 데이터 레이크

```
다양한 소스 → S3 (Raw)
                │
                ▼
         Glue Crawlers (스키마 탐색)
                │
                ▼
         Glue Data Catalog (메타데이터)
                │
                ▼
         Glue ETL (변환 → Parquet)
                │
                ▼
         S3 (Processed) → Athena → QuickSight
                        → Redshift Spectrum
```

---

## 4. 시험 빈출 변형 시나리오

### 시나리오 1: S3 데이터 SQL 분석

> "S3에 저장된 CSV 데이터를 SQL로 분석해야 한다. 인프라 관리 없이."

**정답**: **Athena** (서버리스, S3 직접 쿼리)  
**최적화**: CSV → **Parquet** 변환 (Glue ETL) → 스캔량 ↓ 비용 ↓  
**오답 함정**: Redshift (인프라 관리, 가끔 분석에 과도)

### 시나리오 2: 실시간 스트리밍 → S3

> "실시간 로그를 S3에 자동 저장해야 한다. 가장 간단한 방법."

**정답**: **Kinesis Data Firehose** → S3  
**오답 함정**: Kinesis Data Streams (커스텀 소비자 개발 필요)

### 시나리오 3: Athena 비용 최적화

> "Athena 쿼리 비용이 높다. 어떻게 줄일 수 있는가?"

**정답 조합:**
1. **Parquet/ORC** 컬럼형 포맷 변환 (스캔량 90%↓)
2. **파티셔닝** (year/month/day 폴더 구조)
3. 압축 (Snappy, GZIP)
4. 필요한 컬럼만 SELECT

### 시나리오 4: 대규모 ETL

> "S3의 페타바이트 데이터를 Spark으로 변환해야 한다."

**정답**: **EMR** (대규모 Spark 클러스터)  
**대안**: Glue ETL (서버리스, 규모 제한)  
**비용 최적화**: EMR + **Spot Instances** (태스크 노드)

### 시나리오 5: 로그 검색 + 시각화

> "애플리케이션 로그를 검색하고 실시간 대시보드로 모니터링해야 한다."

**정답**: **OpenSearch** (구 Elasticsearch) + Kibana  
**수집**: Kinesis Data Firehose → OpenSearch  
**오답 함정**: Athena (텍스트 검색에 비적합, 배치 분석용)

### 시나리오 6: 데이터 레이크 접근 제어

> "데이터 레이크에서 부서별로 접근 가능한 데이터를 제한해야 한다."

**정답**: **Lake Formation** (세밀한 열/행 수준 접근 제어)  
**오답 함정**: S3 Bucket Policy (파일 단위, 열/행 제어 불가)

### 시나리오 7: Redshift + S3 연합 쿼리

> "Redshift에 적재된 데이터와 S3의 히스토리 데이터를 함께 조인해야 한다."

**정답**: **Redshift Spectrum** (S3 외부 테이블 → Redshift에서 조인)  
**오답 함정**: S3 데이터를 Redshift에 모두 로드 (비용 + 시간)

### 시나리오 8: DB 변경 데이터 캡처

> "RDS의 변경 데이터를 실시간으로 S3 데이터 레이크에 동기화해야 한다."

**정답**: **DMS + CDC (Change Data Capture)** → S3  
**대안**: DynamoDB Streams → Lambda → S3

---

## 5. 핵심 키워드 → 서비스 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "S3 SQL 쿼리" + "서버리스" | **Athena** |
| "스트리밍 → S3 자동 전달" | **Kinesis Data Firehose** |
| "실시간 커스텀 처리" | **Kinesis Data Streams** |
| "서버리스 ETL" | **Glue** |
| "메타데이터 카탈로그" | **Glue Data Catalog** |
| "데이터 웨어하우스" | **Redshift** |
| "로그 검색 + 대시보드" | **OpenSearch** |
| "BI 대시보드" + "서버리스" | **QuickSight** |
| "대규모 Spark/Hadoop" | **EMR** |
| "데이터 레이크 권한" | **Lake Formation** |

---

## 6. 헷갈리는 포인트 / 함정

### 함정 1: Kinesis Data Streams vs Firehose

- 💡 "S3로 자동 전달" → **Firehose** (소비자 개발 불필요)
- 💡 "실시간 커스텀 처리" + "여러 소비자" → **Data Streams**
- 💡 "Lambda로 실시간 처리" → 둘 다 가능 (Data Streams 더 실시간)

### 함정 2: Athena 비용 = 스캔 데이터량

- CSV: 전체 파일 스캔 → 비용 높음
- **Parquet**: 필요한 컬럼만 스캔 → **비용 최대 90% 절감**
- 💡 Athena 비용 문제 → **Parquet 변환 + 파티셔닝**

### 함정 3: Glue Data Catalog = Athena 메타데이터 소스

- Athena는 Glue Data Catalog을 메타데이터 저장소로 사용
- Glue Crawlers → Data Catalog (테이블 정의) → Athena 쿼리
- 💡 "Athena에서 테이블이 안 보임" → Glue Crawler 실행

### 함정 4: Redshift Serverless vs Provisioned

- **Provisioned**: 노드 수 고정, 예측 가능한 워크로드
- **Serverless**: 자동 확장, 사용한 만큼 비용
- 💡 "Redshift" + "운영 오버헤드 최소화" → **Redshift Serverless**

---

## 7. 검증 필요 항목 ⚠️

- [ ] Kinesis Data Firehose 최신 이름 변경 (Amazon Data Firehose?)
- [ ] Athena 최신 엔진 버전 (v3?) 기능
- [ ] Glue ETL 최신 요금 (DPU 시간당)
- [ ] Redshift Serverless 최신 요금 체계
- [ ] OpenSearch Serverless 최신 기능
- [ ] Lake Formation 행/열 수준 보안 최신 지원 범위

---

## 8. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "데이터 파이프라인 설계" → 수집 → 저장 → 변환 → 분석
- **유형 2**: "S3 쿼리" → Athena
- **유형 3**: "실시간 수집" → Kinesis Streams vs Firehose
- **유형 4**: "ETL" → Glue vs EMR
- **유형 5**: "Athena 비용 최적화" → Parquet + 파티셔닝

---

## 9. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
