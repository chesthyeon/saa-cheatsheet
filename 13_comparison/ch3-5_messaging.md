# Chapter 3-5. 메시징: SQS vs SNS vs EventBridge vs Kinesis vs Step Functions

## 0. 한 줄 요약

🔑 **"1:1 큐 vs 1:N 팬아웃 vs 이벤트 라우팅 vs 실시간 스트림 vs 워크플로우 오케스트레이션" — 통신 패턴이 서비스를 결정한다**

---

## 1. 전체 비교 매트릭스

| 기준 | SQS | SNS | EventBridge | Kinesis Data Streams | Step Functions |
|------|-----|-----|-------------|---------------------|---------------|
| **패턴** | 1:1 큐 (Point-to-Point) | 1:N 팬아웃 (Pub/Sub) | 이벤트 버스 (라우팅) | 실시간 스트림 | 워크플로우 오케스트레이션 |
| **소비 방식** | **폴링** (컨슈머가 가져감) | **푸시** (구독자에게 전달) | **푸시** (규칙 매칭 → 타겟) | **폴링** (샤드에서 읽기) | 상태 머신 (자동 실행) |
| **메시지 보존** | 최대 14일 | ❌ (즉시 전달, 미저장) | ❌ (즉시 전달) | **1~365일** | 실행 이력 보존 |
| **순서 보장** | FIFO만 | FIFO 토픽만 | ❌ | ✅ **샤드 내 보장** | ✅ (상태 머신 순서) |
| **재처리** | DLQ로 실패 메시지 | ❌ (재전송 없음) | DLQ/재시도 | ✅ **원하는 시점부터 재읽기** | ✅ (재실행 가능) |
| **다중 컨슈머** | ❌ (메시지 소비 후 삭제) | ✅ (모든 구독자에게) | ✅ (여러 규칙/타겟) | ✅ **다중 컨슈머 동시 읽기** | N/A |
| **처리량** | Standard: 무제한, FIFO: 300/s | 무제한 | 무제한 | **샤드당 1 MB/s 입력** | Standard: 무제한 |
| **서버리스** | ✅ | ✅ | ✅ | 온디맨드 모드 가능 | ✅ |
| **주요 통합** | Lambda, EC2 | Lambda, SQS, Email, HTTP | 100+ AWS 서비스, SaaS | Lambda, KCL 앱 | Lambda, ECS, SNS, SQS 등 |

---

## 2. 핵심 판별 플로우차트

```
"메시징/통합 서비스 선택"
  │
  ├─ 비동기 디커플링 (1:1 큐)? ──→ SQS
  │   ├─ 순서 보장 + 정확히 1회? ──→ SQS FIFO
  │   └─ 대량 + 순서 무관? ──→ SQS Standard
  │
  ├─ 하나의 메시지 → 여러 수신자 (1:N)?
  │   ├─ 단순 팬아웃? ──→ SNS (+ SQS 조합)
  │   └─ 이벤트 필터링/라우팅 + SaaS 연동? ──→ EventBridge
  │
  ├─ 실시간 대량 스트리밍 데이터?
  │   ├─ 커스텀 처리 + 재읽기 필요? ──→ Kinesis Data Streams
  │   └─ S3/Redshift로 자동 전송? ──→ Kinesis Data Firehose
  │
  └─ 여러 단계의 복잡한 처리 흐름?
      └─ 분기/병렬/재시도/승인? ──→ Step Functions
```

---

## 3. 쌍별 상세 비교

### SQS vs SNS

| 기준 | SQS | SNS |
|------|-----|-----|
| 통신 패턴 | **1:1** (하나의 컨슈머) | **1:N** (여러 구독자) |
| 전달 방식 | 폴링 (Pull) | 푸시 (Push) |
| 메시지 보존 | 최대 14일 | 전달 후 삭제 |
| 재처리 | DLQ | 없음 |
| 조합 | SNS → **SQS 팬아웃** (단골 패턴) | SNS → SQS, Lambda, Email, HTTP |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "디커플링", "큐", "버퍼링" | **SQS** |
| "팬아웃", "1:N 알림" | **SNS** |
| "S3 업로드 → 여러 서비스 동시 처리" | **SNS + SQS 팬아웃** |

### SNS vs EventBridge

| 기준 | SNS | EventBridge |
|------|-----|-------------|
| 이벤트 소스 | AWS 서비스 + 커스텀 | **100+ AWS 서비스 + SaaS** (Zendesk, Datadog 등) |
| 필터링 | 구독 필터 정책 (속성 기반) | **이벤트 패턴 매칭** (JSON 콘텐츠 기반, 더 강력) |
| 스키마 | 없음 | **Schema Registry** (자동 탐색) |
| 스케줄 | ❌ | ✅ **EventBridge Scheduler** (cron/rate) |
| Cross-Account | 제한적 | ✅ 이벤트 버스 공유 |
| 타겟 수 | 구독당 1타겟 | 규칙당 최대 5타겟 |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "단순 팬아웃", "이메일/SMS 알림" | **SNS** |
| "이벤트 기반 아키텍처", "SaaS 연동" | **EventBridge** |
| "cron 스케줄 실행" | **EventBridge Scheduler** |
| "이벤트 콘텐츠 기반 라우팅" | **EventBridge** |

### SQS vs Kinesis Data Streams

| 기준 | SQS | Kinesis Data Streams |
|------|-----|---------------------|
| 목적 | 비동기 디커플링 | **실시간 스트리밍 수집** |
| 메시지 보존 | 최대 14일 (소비 후 삭제) | **1~365일 (소비 후에도 유지)** |
| 다중 컨슈머 | ❌ (소비 → 삭제) | ✅ **여러 앱이 동시에 같은 데이터 읽기** |
| 순서 보장 | FIFO만 (300/s 제한) | ✅ **샤드 내 보장 (높은 처리량)** |
| 재처리 | DLQ | ✅ **시점 지정 재읽기** |
| 처리량 | Standard: 무제한 | 샤드 확장 (샤드당 1 MB/s) |
| 관리 | 완전 서버리스 | 샤드 관리 필요 (온디맨드 모드 제외) |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "디커플링", "비동기 큐" | **SQS** |
| "실시간 스트리밍", "로그/클릭 수집" | **Kinesis** |
| "재처리 가능", "다중 컨슈머" | **Kinesis** |
| "순서 보장 + 대량 처리" | **Kinesis** (SQS FIFO는 300/s 제한) |

### Kinesis Data Streams vs Firehose

| 기준 | Data Streams | Firehose |
|------|-------------|----------|
| 컨슈머 | **커스텀** (Lambda, KCL 앱) | **자동 전송** (S3, Redshift, OpenSearch) |
| 관리 | 샤드 관리 (또는 온디맨드) | **완전 서버리스** |
| 지연 | **실시간** (~200ms) | **니어 리얼타임** (60초 버퍼링) |
| 데이터 변환 | 직접 구현 | Lambda 변환 내장 |
| 재처리 | ✅ | ❌ |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "실시간 커스텀 처리" | **Data Streams** |
| "로그/이벤트 → S3 자동 저장" | **Firehose** |
| "운영 최소화 + 스트림 → S3" | **Firehose** |

### Step Functions vs SQS+Lambda

| 기준 | Step Functions | SQS + Lambda |
|------|---------------|-------------|
| 복잡도 | **복잡한 흐름** (분기/병렬/루프/승인) | 단순 큐 → 처리 |
| 시각화 | ✅ 상태 머신 다이어그램 | ❌ |
| 에러 처리 | **Catch/Retry 내장** | DLQ + 수동 처리 |
| 실행 시간 | Standard: 최대 1년 | Lambda: 최대 15분/함수 |
| 사람 승인 | ✅ **콜백 패턴** | ❌ |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "워크플로우", "분기/병렬", "오케스트레이션" | **Step Functions** |
| "단순 비동기 처리" | **SQS + Lambda** |
| "사람 승인 단계 포함" | **Step Functions** |

---

## 4. 핵심 조합 패턴

### 패턴 1: SNS + SQS 팬아웃 (가장 빈출)

```
이벤트 소스 → SNS 토픽 → SQS 큐 A → Lambda A (썸네일)
                       → SQS 큐 B → Lambda B (메타데이터)
                       → SQS 큐 C → Lambda C (알림)
```

> "하나의 이벤트 → 여러 서비스가 독립적으로 처리" = **SNS + SQS 팬아웃**

### 패턴 2: Kinesis → Firehose → S3 + Analytics

```
데이터 소스 → Kinesis Data Streams → Kinesis Data Analytics (실시간 분석)
                                   → Kinesis Data Firehose → S3 (저장)
```

> "실시간 수집 + 분석 + 저장 동시에" = **Streams + Analytics + Firehose**

### 패턴 3: EventBridge → Step Functions → 다수 서비스

```
AWS 서비스 이벤트 → EventBridge 규칙 → Step Functions → Lambda/ECS/SNS 오케스트레이션
```

> "이벤트 감지 → 복잡한 처리 흐름" = **EventBridge + Step Functions**

---

## 5. 시험 빈출 시나리오 + 정답

| 시나리오 | 정답 | 오답 함정 |
|----------|------|----------|
| 주문 서비스 ↔ 결제 서비스 느슨한 결합 | **SQS** | SNS (1:1이면 SQS) |
| S3 업로드 → 썸네일 + DB + 알림 동시 | **SNS + SQS 팬아웃** | SQS만 (1:N 불가) |
| SaaS 이벤트 → AWS 서비스 연동 | **EventBridge** | SNS (SaaS 연동 부족) |
| 실시간 IoT 센서 데이터 수집 | **Kinesis Data Streams** | SQS (재읽기/다중 컨슈머 불가) |
| 로그 → S3 자동 저장, 운영 최소화 | **Firehose** | Data Streams (관리 필요) |
| 결제 → 재고 → 배송 → 알림 (재시도 포함) | **Step Functions** | SQS (복잡한 흐름 부적합) |
| 매일 자정 Lambda 실행 | **EventBridge Scheduler** | CloudWatch Events (레거시) |
| 금융 거래 메시지, 순서+정확히 1회 | **SQS FIFO** | Standard (순서 미보장) |
| 초당 수만 건 + 순서 보장 + 다중 컨슈머 | **Kinesis Data Streams** | SQS FIFO (300/s 제한) |

---

## 6. 검증 필요 항목 ⚠️

- [ ] SQS FIFO 최대 처리량 (배치 3,000/s) 변경 여부
- [ ] Kinesis Data Streams 온디맨드 모드 최신 한도
- [ ] Kinesis Data Firehose → Amazon Data Firehose 리브랜딩 확인
- [ ] Kinesis Data Analytics → Managed Apache Flink 리브랜딩 확인
- [ ] EventBridge Scheduler 최대 스케줄 수 한도
- [ ] Step Functions Express 최대 실행 시간 (5분) 변경 여부
- [ ] SNS FIFO 토픽 최신 지원 범위

---

## 7. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "1:1 디커플링" → SQS vs "1:N 팬아웃" → SNS
- **유형 2**: "이벤트 라우팅 + SaaS" → EventBridge vs SNS
- **유형 3**: "실시간 스트림 + 재처리" → Kinesis vs SQS
- **유형 4**: "Data Streams vs Firehose" → 커스텀 처리 vs 자동 전송
- **유형 5**: "워크플로우 오케스트레이션" → Step Functions vs SQS+Lambda

---

## 8. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
