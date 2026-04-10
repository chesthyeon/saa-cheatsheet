# Chapter 2-8. 실시간 처리·스트리밍 키워드 → 서비스 매핑

## 0. 한 줄 요약

🔑 **"Streaming = 데이터가 '끝없이 흐르는' 형태로 들어올 때 — 수집(Kinesis Data Streams/KDS) → 변환(Lambda/KDA/Flink) → 적재(Firehose/S3/Redshift) → 분석(OpenSearch/Athena)"**

---

## 1. 배치 vs 스트리밍

| 기준 | 배치 | 스트리밍 |
|------|------|---------|
| 데이터 단위 | 파일/테이블 | **이벤트/레코드** |
| 처리 시기 | 주기적 (시간/일) | **실시간/거의 실시간** |
| 지연 | 분~시간 | **밀리초~초** |
| 대표 서비스 | Glue, EMR, Batch | **Kinesis**, MSK, Flink |
| 예시 | 야간 ETL, 월말 정산 | IoT 센서, 로그, 클릭스트림, 금융 거래 |

💡 시험 키워드: "**real-time**", "**near real-time**", "**streaming**", "**clickstream**", "**IoT telemetry**", "**log ingestion**" → 전부 **Kinesis 계열**

---

## 2. Kinesis 4형제 (가장 중요)

| 서비스 | 역할 | 관리 수준 | 지연 | 용도 |
|--------|------|----------|------|------|
| **Kinesis Data Streams (KDS)** | **수집 + 저장** (샤드 기반) | 샤드 관리 필요 | ~200ms | 커스텀 실시간 처리, 재처리 필요 |
| **Kinesis Data Firehose (KDF)** | **배달 서비스** (S3/Redshift/OpenSearch/Splunk) | **완전 관리** (서버리스) | **~60초 버퍼** | 간단한 적재/ETL, 코드 없이 |
| **Kinesis Data Analytics (KDA)** / **Managed Service for Apache Flink** | **SQL/Flink 실시간 분석** | 관리형 | 초 단위 | 실시간 집계, 이상 탐지 |
| **Kinesis Video Streams** | 비디오/오디오 스트리밍 | 관리형 | - | CCTV, ML 추론 |

### ⚡ 핵심 구분: KDS vs Firehose

| 기준 | **KDS** (Data Streams) | **Firehose** |
|------|----------------------|--------------|
| 모델 | **Pull** (소비자가 가져감) | **Push** (자동 배달) |
| 저장 | ✅ **1~365일 보관** | ❌ (버퍼만, 즉시 배달) |
| 재처리 | ✅ 가능 (shard iterator) | ❌ 불가 |
| 스케일링 | **샤드 수동 관리** (또는 On-Demand) | **자동** |
| 지연 | **~200ms** (실시간) | **~60초** (near real-time) |
| 타겟 | 커스텀 (Lambda/KDA/EC2) | S3/Redshift/OpenSearch/Splunk만 |
| 변환 | 소비자 코드 | **Lambda 변환** (선택적) |
| 비용 | 샤드 시간 | 데이터량 기반 |

💡 **가장 헷갈리는 포인트**:
- "**real-time 분석 + 재처리 필요**" → **KDS**
- "**S3/Redshift로 자동 적재만**" → **Firehose**
- "서버리스 / 관리 오버헤드 최소" → **Firehose**
- "200ms 이하 지연" → **KDS** (Firehose는 최소 60초 버퍼)

---

## 3. Kinesis Data Streams 심화

### 샤드 (Shard)

| 항목 | 한도 |
|------|------|
| **쓰기** | 1 MB/s **또는** 1,000 records/s |
| **읽기** | 2 MB/s (fan-out 없으면 모든 소비자 공유) |
| **Enhanced Fan-Out** | 소비자별 **2 MB/s 전용** (최대 20 소비자) |
| **보관** | 1일 (기본) ~ **365일** (확장) |

### 용량 모드

| 모드 | 설명 |
|------|------|
| **Provisioned** | 샤드 수 수동 관리, 예측 가능한 트래픽 |
| **On-Demand** | **자동 확장**, 예측 불가 트래픽, 관리 오버헤드↓ |

💡 "트래픽 예측 불가 / 운영 오버헤드 최소" → **On-Demand**

### 파티션 키 함정

- 파티션 키 해시 → 샤드 결정
- **핫 샤드**(특정 키 집중) → 성능 저하
- 💡 "uneven distribution / hot shard" → **파티션 키 분산 설계**

### 소비 방식

| 방식 | 설명 |
|------|------|
| **Classic (Shared)** | 모든 소비자가 2 MB/s 공유 (폴링) |
| **Enhanced Fan-Out** | 소비자당 2 MB/s **전용** + HTTP/2 푸시 |
| **Lambda Event Source** | 자동 폴링 → Lambda 호출 |
| **Kinesis Client Library (KCL)** | EC2 소비자, 체크포인트 자동 관리 |

💡 "**여러 소비자 + 지연 최소**" → **Enhanced Fan-Out**

---

## 4. Kinesis Data Firehose 심화

### 버퍼링 (핵심!)

- **크기 기준**: 1~128 MB
- **시간 기준**: 60~900초 (**최소 60초**)
- 💡 "**실시간(<1초) 불가능**" = Firehose 최소 60초 버퍼
- 둘 중 먼저 채워지는 쪽으로 flush

### 타겟

| 타겟 | 용도 |
|------|------|
| **S3** | 데이터 레이크 (가장 흔함) |
| **Redshift** | 분석용 DW (S3 경유 COPY) |
| **OpenSearch** | 로그 검색/대시보드 |
| **Splunk** | 보안/운영 로그 |
| **HTTP Endpoint** | Datadog/MongoDB/New Relic 등 |

### 변환 옵션

- **Lambda 변환**: 레코드 단위 커스텀 변환
- **Format 변환**: JSON → Parquet/ORC (Athena 비용 절감에 유용)
- **압축**: GZIP, Snappy
- **암호화**: KMS
- **동적 파티셔닝**: 속성 기반 S3 prefix 생성

💡 "**Parquet 변환 + S3 파티셔닝 + 코드 없이**" → **Firehose 동적 파티셔닝 + Parquet 변환**

---

## 5. Kinesis Data Analytics / Managed Apache Flink

| 기능 | 설명 |
|------|------|
| **입력** | KDS, KDF, MSK |
| **처리** | SQL (KDA-SQL deprecated) / **Apache Flink** (현재 권장) |
| **윈도우** | Tumbling / Sliding / Session |
| **출력** | KDS, KDF, Lambda |

💡 "**실시간 집계 / 이상 탐지 / 윈도우 쿼리**" → **Managed Service for Apache Flink**

---

## 6. MSK (Managed Streaming for Kafka)

| 기준 | MSK | KDS |
|------|-----|-----|
| 프로토콜 | **Kafka (표준)** | AWS 전용 |
| 마이그레이션 | **기존 Kafka 앱 그대로** | 코드 변경 필요 |
| 파티션 | Kafka Partition | Shard |
| 관리 | MSK (Broker) / **MSK Serverless** | 샤드 or On-Demand |
| 생태계 | Kafka Connect, Streams, Schema Registry | AWS 네이티브 |

💡 "**기존 Kafka 워크로드 리프트&시프트**" → **MSK**  
💡 "**Kafka 호환 + 관리 부담 최소**" → **MSK Serverless**  
💡 "처음부터 AWS로 구축" → **KDS** (더 간단)  
💡 "코드 변경 최소화" 키워드 → **MSK**

---

## 7. 실시간 처리 대표 파이프라인

### 파이프라인 1: 클릭스트림 → 실시간 분석

```
Web/App → KDS → Managed Flink → KDS → Lambda → DynamoDB (실시간 대시보드)
                             → Firehose → S3 (장기 저장)
```

### 파이프라인 2: IoT 텔레메트리

```
IoT Device → IoT Core → IoT Rule → KDS → Lambda → DynamoDB
                                       → Firehose → S3 → Athena
```

### 파이프라인 3: 로그 수집 (ELK 대체)

```
EC2/App → CloudWatch Logs → Subscription Filter → Firehose → OpenSearch
                                                          → S3 (아카이브)
```

### 파이프라인 4: 데이터 레이크 적재 (최저 운영 오버헤드)

```
Producer → Firehose (Parquet 변환 + 동적 파티셔닝) → S3 → Athena/Glue/QuickSight
```

### 파이프라인 5: 금융 거래 실시간 이상 탐지

```
Transaction → KDS → Managed Flink (윈도우 집계) → Lambda → SNS 알림
                                                        → DynamoDB
```

---

## 8. 시험 빈출 시나리오

### 시나리오 1: "실시간(<1초) 처리 + 재처리 가능"

**정답**: **KDS** (Enhanced Fan-Out, 보관 기간 확장)  
**오답**: Firehose (최소 60초 버퍼)

### 시나리오 2: "서버리스로 S3에 자동 적재 + Parquet 변환"

**정답**: **Firehose** (Lambda 변환 + Format 변환)  
**오답**: KDS + Lambda (관리 오버헤드 더 큼)

### 시나리오 3: "기존 Kafka 앱을 AWS로 이전, 코드 변경 최소"

**정답**: **Amazon MSK** (또는 MSK Serverless)  
**오답**: KDS (코드 재작성 필요)

### 시나리오 4: "OpenSearch로 실시간 로그 분석"

**정답**: **CloudWatch Logs → Firehose → OpenSearch**

### 시나리오 5: "10초 윈도우 평균 계산, 이상치 탐지"

**정답**: **KDS → Managed Service for Apache Flink (윈도우 SQL)**

### 시나리오 6: "IoT 센서 수백만 개 수집"

**정답**: **IoT Core → KDS / Firehose**

### 시나리오 7: "샤드 수 예측 어려움 + 관리 최소화"

**정답**: **KDS On-Demand** 또는 **Firehose**

### 시나리오 8: "Kinesis 소비자 여러 개 + 지연 민감"

**정답**: **Enhanced Fan-Out** (소비자당 2 MB/s 전용)

### 시나리오 9: "CCTV 비디오 스트림 → ML 추론"

**정답**: **Kinesis Video Streams + Rekognition Video**

### 시나리오 10: "로그를 S3에 저장 + Athena로 쿼리, 코드 작성 불가"

**정답**: **Firehose → S3 (Parquet) → Athena**

---

## 9. 빠른 매핑 치트표

| 키워드 | 정답 |
|--------|------|
| "real-time (<1초)" | **KDS** |
| "near real-time (60초 OK)" | **Firehose** |
| "자동 S3/Redshift/OpenSearch 적재" | **Firehose** |
| "재처리 / 장기 보관 (365일)" | **KDS** |
| "Kafka 호환" | **MSK** |
| "실시간 SQL 윈도우 집계" | **Managed Flink** |
| "Parquet 변환 + 동적 파티셔닝" | **Firehose** |
| "관리 오버헤드 최소 + 스트리밍" | **Firehose** 또는 **KDS On-Demand** |
| "여러 소비자 + 지연 민감" | **KDS Enhanced Fan-Out** |
| "IoT 텔레메트리" | **IoT Core → KDS** |
| "로그 → ELK" | **Firehose → OpenSearch** |
| "CCTV / 비디오 ML" | **Kinesis Video Streams** |
| "Kafka Serverless" | **MSK Serverless** |

---

## 10. 함정 / 주의사항

### 함정 1: "실시간"인데 Firehose 선택

- Firehose 최소 버퍼 = **60초** → 초 단위 실시간 **불가**
- "real-time" 키워드 → **KDS**
- "near real-time" / "close to real-time" → Firehose OK
- 💡 키워드 정확히 구분

### 함정 2: 샤드 한도 초과

- 1 샤드 = **1 MB/s 쓰기, 1,000 records/s**
- 초과 시 `ProvisionedThroughputExceededException`
- 해결: **샤드 분할** 또는 **On-Demand 모드**
- 💡 "throughput exception" → 샤드 추가

### 함정 3: 모든 소비자가 2 MB/s 공유

- 기본(Classic) 모드: 소비자 N명이 샤드당 **2 MB/s 공유**
- 지연 증가 → **Enhanced Fan-Out** (소비자당 전용)
- 💡 "multiple consumers / low latency" → Enhanced Fan-Out

### 함정 4: Firehose = "스트리밍"이지만 ETL 아님

- Firehose는 **배달** 서비스 (변환은 Lambda 옵션)
- 복잡한 ETL은 **Glue** 또는 **Flink**
- 💡 "complex transformation" → Firehose ❌

### 함정 5: KDS vs SQS 혼동

| 기준 | KDS | SQS |
|------|-----|-----|
| 데이터 모델 | **스트림** (순서 + 재처리) | **큐** (소비 후 삭제) |
| 보관 | 1~365일 | 최대 14일 |
| 소비자 | **여러 명 동시** | 1명 (삭제됨) |
| 순서 | 샤드 내 순서 | FIFO Queue만 |
| 용도 | 실시간 분석 | 작업 분산 |

💡 "**여러 소비자 + 재처리**" → KDS / "작업 분산" → SQS

### 함정 6: MSK vs KDS 선택

- "**코드 변경 없이**" / "**기존 Kafka**" → **MSK**
- 처음부터 AWS → **KDS** (더 간단, 저렴)
- 💡 "minimize code changes" 키워드 핵심

### 함정 7: Firehose 비용 vs Lambda 직접 S3 쓰기

- Firehose는 데이터량 기반 과금이지만 **배치 쓰기**로 S3 PUT 요청 절감
- Lambda로 매 레코드 S3 쓰기 → **PUT 요청 폭발** (비쌈)
- 💡 "S3 요청 비용 절감" → Firehose 배치

---

## 11. 검증 필요 항목 ⚠️

- [ ] Firehose 최소 버퍼 시간 최신값 (60초 → 0초 near real-time 옵션?)
- [ ] KDS On-Demand 최대 처리량 한도
- [ ] MSK Serverless 리전 가용성
- [ ] Managed Service for Apache Flink (구 KDA) 최신 기능
- [ ] Kinesis Data Analytics for SQL 종료 시점 및 마이그레이션 경로

---

## 12. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "Real-time vs Near real-time" → KDS vs Firehose
- **유형 2**: "Existing Kafka migration" → MSK
- **유형 3**: "Least operational overhead + streaming" → Firehose / KDS On-Demand
- **유형 4**: "Multiple consumers + low latency" → KDS Enhanced Fan-Out
- **유형 5**: "Real-time aggregation / windowed analytics" → Managed Flink
- **유형 6**: "Log ingestion to OpenSearch" → CloudWatch Logs → Firehose → OpenSearch

---

## 13. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
