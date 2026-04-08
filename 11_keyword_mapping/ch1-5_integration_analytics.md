# Chapter 1-5. 애플리케이션 통합 및 분석

## 0. 한 줄 요약

🔑 **"컴포넌트 간 결합을 어떻게 느슨하게 만들 것인가(통합)" + "데이터를 어떻게 수집·변환·쿼리할 것인가(분석)"가 이 챕터의 두 축이다**

---

## 1. 핵심 서비스 표

| 서비스 | 한 줄 정의 | 주요 키워드 | 가격 모델 |
|--------|-----------|------------|----------|
| **SQS** | 완전 관리형 메시지 큐 | 디커플링, 비동기, Standard/FIFO, DLQ, 폴링 | 요청 수 (백만 건당) |
| **SNS** | 완전 관리형 Pub/Sub 메시징 | 팬아웃, 토픽, 구독(Lambda/SQS/Email/HTTP), 필터링 | 게시 수 + 전송 수 |
| **EventBridge** | 서버리스 이벤트 버스 | 이벤트 규칙, SaaS 통합, 스케줄, Schema Registry | 이벤트 수 |
| **Step Functions** | 서버리스 워크플로우 오케스트레이션 | 상태 머신, 분기/병렬/재시도, Standard/Express | 상태 전환 수 (Standard) / 실행 시간 (Express) |
| **Kinesis Data Streams** | 실시간 데이터 스트리밍 수집 | 샤드, 실시간 수집, 순서 보장, 1~365일 보존 | 샤드 시간 + PUT 페이로드 |
| **Kinesis Data Firehose** | 스트림 → 대상 자동 전송 (ETL) | S3/Redshift/OpenSearch 전송, 자동 배치, 변환 | 처리 데이터량 (GB) |
| **Kinesis Data Analytics** | 스트림 데이터 실시간 SQL/Flink 분석 | 실시간 분석, Apache Flink, 윈도우 함수 | 처리 시간 (KPU) |
| **Kinesis Video Streams** | 비디오 스트리밍 수집·저장 | 비디오 수집, ML 분석, 재생 | 수집 데이터량 + 소비 데이터량 |
| **Athena** | S3 데이터 서버리스 즉석 쿼리 | 서버리스 SQL, S3 쿼리, Presto 기반 | 스캔 데이터량 (TB당) |
| **Glue** | 서버리스 ETL + 데이터 카탈로그 | ETL 작업, 크롤러, 데이터 카탈로그, Spark 기반 | DPU 시간 (ETL) + 크롤러 실행 시간 |

---

## 2. 키워드 → 서비스 매핑

### SQS가 정답인 키워드

| 시험 키워드 | 왜 SQS인가 |
|------------|-----------|
| "디커플링", "느슨한 결합" | 프로듀서-컨슈머 분리 |
| "비동기 처리", "큐" | 메시지를 큐에 넣고 비동기 소비 |
| "순서 보장" + "정확히 한 번 처리" | **SQS FIFO** |
| "버퍼링", "트래픽 급증 흡수" | SQS가 버퍼 역할 |
| "Dead Letter Queue (DLQ)" | 처리 실패 메시지 격리 |
| "폴링 기반 메시지 소비" | 컨슈머가 직접 메시지를 가져감 |
| "메시지 지연 전송" | Delay Queue (최대 15분) |

### SQS Standard vs FIFO

| 기준 | Standard | FIFO |
|------|----------|------|
| 순서 | 최선 노력 (보장 안 됨) | **엄격한 순서 보장** |
| 중복 | 최소 1회 전송 (중복 가능) | **정확히 1회 전송** |
| 처리량 | **무제한** | 초당 300건 (배치 시 3,000) |
| 사용 사례 | 대량 처리, 순서 무관 | 금융 거래, 주문 처리 |

> 💡 "순서 보장" 또는 "정확히 한 번" → FIFO, 그 외 대부분 → Standard

### SNS가 정답인 키워드

| 시험 키워드 | 왜 SNS인가 |
|------------|-----------|
| "팬아웃", "1:N 메시지 전송" | 하나의 메시지를 여러 구독자에게 |
| "알림", "이메일/SMS 전송" | SNS 구독 프로토콜 |
| "SNS + SQS 팬아웃 패턴" | SNS 토픽 → 여러 SQS 큐 |
| "이벤트 알림" (단순한 경우) | S3 이벤트 → SNS → 구독자들 |
| "메시지 필터링" | 구독자별 필터 정책 |

### SNS + SQS 팬아웃 패턴

```
S3 이벤트 → SNS 토픽 → SQS 큐 A (썸네일 생성)
                     → SQS 큐 B (메타데이터 저장)
                     → Lambda C (알림 전송)
```

> 💡 "하나의 이벤트를 여러 서비스가 각각 처리" → **SNS + SQS 팬아웃**이 단골 정답

### EventBridge가 정답인 키워드

| 시험 키워드 | 왜 EventBridge인가 |
|------------|------------------|
| "이벤트 기반 아키텍처" (고급) | 이벤트 버스 + 규칙 기반 라우팅 |
| "SaaS 이벤트 통합" | Zendesk, Datadog 등 SaaS → AWS |
| "스케줄 기반 실행" (cron) | EventBridge Scheduler |
| "이벤트 스키마 자동 탐색" | Schema Registry |
| "다수 AWS 서비스 이벤트 라우팅" | 100+ AWS 서비스 이벤트 소스 |
| "Cross-Account 이벤트 전달" | 이벤트 버스 간 공유 |

> ⚠️ **SNS vs EventBridge**: 단순 팬아웃 → SNS, 이벤트 필터링/라우팅/SaaS 연동 → EventBridge

### Step Functions가 정답인 키워드

| 시험 키워드 | 왜 Step Functions인가 |
|------------|---------------------|
| "워크플로우 오케스트레이션" | 여러 단계를 시각적으로 조합 |
| "분기/병렬/재시도 로직" | 상태 머신으로 복잡한 흐름 제어 |
| "Lambda 체이닝" | 여러 Lambda 함수를 순차/병렬 실행 |
| "장시간 실행 워크플로우" | Standard: 최대 1년 실행 |
| "에러 핸들링 + 자동 재시도" | Catch/Retry 내장 |
| "사람 승인 단계 포함" | 수동 승인 태스크 (콜백 패턴) |

### Kinesis가 정답인 키워드

| 시험 키워드 | 정답 Kinesis 유형 |
|------------|-----------------|
| "실시간 스트리밍 수집", "순서 보장 스트림" | **Kinesis Data Streams** |
| "스트림 → S3/Redshift 자동 전송" | **Kinesis Data Firehose** |
| "스트림 데이터 실시간 SQL 분석" | **Kinesis Data Analytics** |
| "비디오 스트리밍 수집" | **Kinesis Video Streams** |
| "로그 실시간 수집 → S3" | **Firehose** (가장 간단) |
| "IoT 센서 데이터 실시간 수집 + 분석" | **Data Streams** + **Data Analytics** |
| "클릭스트림 분석" | **Data Streams** → **Data Analytics** |

> 💡 **Kinesis 선택 흐름**: 수집만 → Data Streams, 자동 전송 → Firehose, 실시간 분석 → Data Analytics

### Athena가 정답인 키워드

| 시험 키워드 | 왜 Athena인가 |
|------------|-------------|
| "S3 데이터 직접 쿼리" | 데이터 적재 불필요 |
| "서버리스 SQL 분석" | 인프라 관리 없음 |
| "간헐적/즉석 쿼리" (ad-hoc) | 사용한 만큼만 과금 |
| "로그 분석" (S3에 저장된) | CloudTrail/VPC Flow Logs → Athena |
| "비용 효율적 S3 분석" | 스캔 데이터량 기반 과금 → Parquet/ORC로 절감 |

### Glue가 정답인 키워드

| 시험 키워드 | 왜 Glue인가 |
|------------|-----------|
| "ETL", "데이터 변환" | 서버리스 ETL 서비스 |
| "데이터 카탈로그" | 스키마 자동 탐색 (크롤러) |
| "S3 데이터 스키마 탐색" | Glue Crawler → Glue Data Catalog |
| "Athena + 스키마 관리" | Athena가 Glue 카탈로그 참조 |
| "Spark 기반 데이터 처리" | 서버리스 Apache Spark |

---

## 3. 헷갈리는 포인트 / 함정

### 함정 1: SQS vs SNS vs EventBridge

- ❌ "메시지 전달"이면 무조건 SQS → **패턴에 따라 다름**
- ✅ **SQS**: 1:1 큐, 비동기 디커플링, 폴링 기반
- ✅ **SNS**: 1:N 팬아웃, 푸시 기반
- ✅ **EventBridge**: 이벤트 기반 라우팅, SaaS 연동, 고급 필터링
- 💡 "디커플링" + "큐" → SQS, "팬아웃" → SNS, "이벤트 규칙" → EventBridge

### 함정 2: Kinesis Data Streams vs SQS

- ❌ "실시간 데이터 수집" → SQS → **순서+재읽기 필요하면 Kinesis**
- ✅ **SQS**: 메시지 소비 후 삭제, 단순 디커플링
- ✅ **Kinesis Data Streams**: 데이터 보존(1~365일), 순서 보장, 다중 컨슈머, 재읽기 가능
- 💡 "재처리 가능", "다중 컨슈머가 같은 데이터", "실시간 스트리밍" → Kinesis

### 함정 3: Kinesis Data Streams vs Firehose

- ❌ "실시간 수집" → 무조건 Data Streams → **대상에 따라 Firehose가 더 간단**
- ✅ **Data Streams**: 커스텀 컨슈머 필요, 실시간 처리, 샤드 관리
- ✅ **Firehose**: S3/Redshift/OpenSearch에 자동 전송, 관리 불필요 (서버리스)
- 💡 "S3로 자동 전송" + "간단하게" → Firehose, "커스텀 처리" → Data Streams

### 함정 4: Athena vs Redshift

- ❌ "S3 데이터 분석"이면 무조건 Athena → **반복 대규모 분석은 Redshift**
- ✅ **Athena**: 서버리스, 즉석 쿼리, 간헐적, 스캔량 과금
- ✅ **Redshift**: 전용 클러스터, 반복 분석, 대규모 조인, 데이터 적재
- 💡 "서버리스" + "간헐적" → Athena, "반복 BI" + "페타바이트" → Redshift

### 함정 5: Step Functions Standard vs Express

- ✅ **Standard**: 최대 1년 실행, 실행 이력 보존, 상태 전환당 과금
- ✅ **Express**: 최대 5분 실행, 대량 이벤트 처리, 실행 시간당 과금
- 💡 "장시간 워크플로우" → Standard, "대량 짧은 이벤트" → Express

### 함정 6: SQS FIFO 처리량 제한

- ❌ "대량 메시지" + "순서 보장" → FIFO → **처리량 한계 확인 필요**
- ✅ FIFO: 초당 300건 (배치 시 3,000) → 대량이면 부족할 수 있음
- ✅ Standard: 처리량 무제한
- 💡 "초당 수만 건" + "순서 보장" → Kinesis Data Streams가 더 적합

---

## 4. 단골 시나리오

### 시나리오 1: 마이크로서비스 디커플링

```
문제 패턴: "주문 서비스와 결제 서비스를 느슨하게 결합"
정답 패턴: SQS (주문 큐) 또는 SNS + SQS 팬아웃
키 포인트: "디커플링" = SQS가 거의 확정
```

### 시나리오 2: 이벤트 팬아웃

```
문제 패턴: "S3 업로드 시 썸네일 생성 + DB 기록 + 알림 전송 동시에"
정답 패턴: S3 → SNS 토픽 → SQS 큐 A/B + Lambda C
키 포인트: "하나의 이벤트 → 여러 처리" = SNS + SQS 팬아웃
```

### 시나리오 3: 실시간 로그 수집 → S3 저장

```
문제 패턴: "애플리케이션 로그를 실시간으로 S3에 저장, 운영 최소화"
정답 패턴: Kinesis Data Firehose → S3
키 포인트: "실시간 수집 → S3 자동 전송" = Firehose (가장 간단)
```

### 시나리오 4: 실시간 클릭스트림 분석

```
문제 패턴: "웹사이트 클릭 데이터를 실시간으로 분석, 이상 탐지"
정답 패턴: Kinesis Data Streams → Kinesis Data Analytics (Flink)
키 포인트: "실시간 분석" = Data Streams + Data Analytics
```

### 시나리오 5: S3 로그 데이터 즉석 쿼리

```
문제 패턴: "CloudTrail 로그를 S3에 저장, 필요할 때 SQL로 분석"
정답 패턴: S3 + Glue Crawler (카탈로그) + Athena (쿼리)
키 포인트: "서버리스" + "즉석 쿼리" + "S3" = Athena
```

### 시나리오 6: 복잡한 주문 처리 워크플로우

```
문제 패턴: "결제 확인 → 재고 확인 → 배송 → 알림, 실패 시 자동 재시도"
정답 패턴: Step Functions (상태 머신)
키 포인트: "분기/병렬/재시도" + "여러 서비스 조합" = Step Functions
```

---

## 5. 검증 필요 항목 ⚠️

> Claude의 학습 데이터가 outdated일 수 있는 항목. AWS 공식 문서에서 확인 필요.

- [ ] SQS FIFO 최대 처리량 (배치 시 3,000/초) 최신 수치 확인
  - 공식 문서: https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-fifo.html
- [ ] Kinesis Data Streams 최대 보존 기간 (365일) 확인
- [ ] Kinesis Data Firehose 이름 변경 여부 (Amazon Data Firehose로 리브랜딩?)
  - 공식 문서: https://docs.aws.amazon.com/firehose/
- [ ] Kinesis Data Analytics → Amazon Managed Service for Apache Flink 리브랜딩 확인
- [ ] EventBridge Scheduler 최신 기능 (1회성 스케줄 등)
- [ ] Step Functions Express 최대 실행 시간 (5분) 변경 여부
- [ ] Athena 엔진 v3 최신 기능 및 성능 개선 사항
- [ ] Glue 최신 버전 (4.0+) 지원 Spark 버전 확인

---

## 6. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "디커플링" + "비동기" → SQS
- **유형 2**: "1:N 팬아웃" → SNS + SQS
- **유형 3**: "이벤트 기반 라우팅" + "SaaS 연동" → EventBridge
- **유형 4**: "워크플로우 오케스트레이션" + "재시도" → Step Functions
- **유형 5**: "실시간 스트리밍 수집" → Kinesis Data Streams
- **유형 6**: "실시간 수집 → S3 자동 전송" → Kinesis Data Firehose
- **유형 7**: "스트림 실시간 SQL 분석" → Kinesis Data Analytics
- **유형 8**: "S3 서버리스 즉석 쿼리" → Athena
- **유형 9**: "ETL + 데이터 카탈로그" → Glue
- **유형 10**: "순서 보장 메시지 큐" → SQS FIFO

---

## 7. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
