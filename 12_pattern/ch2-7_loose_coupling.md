# Chapter 2-7. 느슨한 결합·이벤트 기반 키워드 → 서비스 매핑

## 0. 한 줄 요약

🔑 **"Decoupling = 생산자와 소비자를 직접 호출하지 않도록 중간에 큐/토픽/이벤트 버스를 끼워 넣는 것 — SQS(버퍼) / SNS(팬아웃) / EventBridge(라우팅) / Step Functions(오케스트레이션)"**

---

## 1. 왜 느슨한 결합인가

| 문제 (강결합) | 해결 (느슨한 결합) |
|-------------|----------------|
| 소비자 장애 → 생산자도 실패 | 큐가 메시지 보관 (SQS) |
| 트래픽 스파이크 → 소비자 과부하 | 큐가 버퍼 역할 |
| 새 소비자 추가 = 생산자 코드 수정 | 토픽/이벤트 버스에 구독만 추가 |
| 동기 호출로 지연 누적 | 비동기 이벤트 |
| 여러 단계 워크플로우 관리 어려움 | Step Functions |

💡 시험 키워드: "**decouple**", "**loosely coupled**", "**independently scale**", "**asynchronous**", "**buffer spikes**" → 전부 **SQS/SNS/EventBridge/Step Functions**

---

## 2. 4대 디커플링 서비스 비교

| 서비스 | 모델 | 메시지 보관 | 소비자 수 | 주 용도 |
|--------|------|-----------|---------|--------|
| **SQS** | 큐 (Pull) | ✅ 최대 14일 | **1:1** (하나만 가져감) | **버퍼링**, 작업 분산 |
| **SNS** | 토픽 (Push) | ❌ (구독자에게 즉시) | **1:N** 팬아웃 | **알림**, 팬아웃 |
| **EventBridge** | 이벤트 버스 (Push) | ❌ (아카이브 가능) | 1:N + **규칙 기반 라우팅** | **이벤트 라우팅**, SaaS 연동 |
| **Step Functions** | 상태 머신 | ✅ (실행 이력) | - | **워크플로우 오케스트레이션** |

### 한눈 매핑

| 키워드 | 정답 |
|--------|------|
| "트래픽 스파이크 흡수" / "버퍼" | **SQS** |
| "하나의 이벤트를 여러 구독자에게" | **SNS** 또는 **EventBridge** |
| "SaaS 이벤트 수신 (Zendesk/Shopify 등)" | **EventBridge** |
| "스키마/필터링/규칙 기반 라우팅" | **EventBridge** |
| "순차 / 분기 / 재시도 워크플로우" | **Step Functions** |
| "AWS 서비스 상태 변경 트리거" | **EventBridge** (기본 버스) |
| "이메일/SMS/모바일 푸시" | **SNS** |

---

## 3. SQS 심화

### 큐 종류

| 종류 | 순서 | 중복 | 처리량 | 용도 |
|------|------|------|-------|------|
| **Standard** | **Best-effort** | At-least-once (중복 가능) | 무제한 | 일반 작업 분산 |
| **FIFO** | **엄격한 순서** | Exactly-once | 300 TPS (배치 3,000) | 주문/결제처럼 순서 중요 |

💡 "**주문 처리 순서 보장**" → **FIFO**  
💡 "최고 처리량 + 순서 무관" → **Standard**

### 핵심 파라미터

| 파라미터 | 설명 | 시험 포인트 |
|---------|------|-----------|
| **Visibility Timeout** | 메시지 수신 후 숨김 시간 (기본 30s, 최대 12h) | **너무 짧으면 중복 처리**, 너무 길면 장애 회복 지연 |
| **Message Retention** | 큐 보관 기간 (1분~14일, 기본 4일) | "최대 14일" |
| **Long Polling** | 메시지 없으면 대기 (최대 20s) | **비용 절감** + API 호출 감소 |
| **Delay Queue** | 메시지 지연 (최대 15분) | 예약 작업 |
| **DLQ** (Dead Letter Queue) | 실패 메시지 격리 | **처리 실패 추적/재처리** |

### SQS 시험 빈출 시나리오

| 시나리오 | 정답 |
|---------|------|
| "메시지가 여러 번 처리됨" | **Visibility Timeout 늘리기** |
| "메시지가 Lost" | **DLQ 설정** + 소비자 에러 확인 |
| "순서 보장 + 중복 제거" | **FIFO Queue** |
| "처리 실패 메시지 보관" | **DLQ** |
| "빈 응답 비용 절감" | **Long Polling** (20s) |
| "EC2 소비자 확장" | **ASG + SQS 큐 길이 CloudWatch 메트릭** |

---

## 4. SNS 심화

| 기능 | 설명 |
|------|------|
| **Topic** | 메시지 발행 대상 |
| **Subscription** | 구독자 (SQS/Lambda/HTTP/Email/SMS/Mobile Push/Firehose) |
| **Fanout** | 1개 메시지 → 여러 SQS 큐로 (팬아웃 패턴) |
| **Message Filtering** | 구독자별 필터 (속성 기반) |
| **FIFO Topic** | SQS FIFO와 조합해 순서 보장 |

### 팬아웃 패턴 (시험 단골)

```
Publisher → SNS Topic → SQS1 (서비스 A)
                     → SQS2 (서비스 B)
                     → SQS3 (서비스 C)
                     → Lambda (실시간 처리)
```

💡 "**하나의 이벤트를 여러 서비스가 독립적으로 처리**" → **SNS + 여러 SQS 팬아웃**

---

## 5. EventBridge 심화

| 구성 요소 | 설명 |
|----------|------|
| **Event Bus** | 기본 / 커스텀 / SaaS 파트너 |
| **Rule** | 이벤트 패턴 매칭 → 타겟 라우팅 |
| **Target** | Lambda, SQS, SNS, Step Functions, Kinesis, ECS Task 등 20+ |
| **Schema Registry** | 이벤트 스키마 자동 탐색 |
| **Archive + Replay** | 이벤트 재생 (디버깅/복구) |
| **Pipes** | Source → Filter → Enrich → Target 파이프라인 |
| **Scheduler** | cron/rate 스케줄 (EventBridge Rules에서 분리) |

### EventBridge vs SNS

| 기준 | SNS | EventBridge |
|------|-----|-------------|
| 구독자 수 | 12.5M / 토픽 | 5 타겟 / 규칙 |
| 처리량 | 매우 높음 | 보통 |
| 필터링 | 단순 속성 | **JSON 패턴 매칭** (풍부) |
| SaaS 이벤트 | ❌ | ✅ **Partner Event Source** |
| 스키마 레지스트리 | ❌ | ✅ |
| 지연 시간 | **낮음** (즉시) | 약간 높음 (~0.5s) |
| 용도 | **알림/팬아웃** | **통합/라우팅** |

💡 "**Zendesk/Shopify/Datadog 이벤트 수신**" → EventBridge (SaaS Partner)  
💡 "단순 팬아웃 + 고처리량" → SNS  
💡 "**AWS 서비스 상태 변경**" (EC2 상태, S3 객체 생성 등) → EventBridge 기본 버스

---

## 6. Step Functions 심화

| 종류 | 용도 | 실행 |
|------|------|------|
| **Standard** | 장기 워크플로우 (최대 1년), 정확히 1회 실행 | ~2,000 실행/초 |
| **Express** | 고처리량 단기 (최대 5분), At-least-once | **100,000 실행/초** |

### 상태 타입

| 상태 | 설명 |
|------|------|
| **Task** | Lambda/ECS/서비스 호출 |
| **Choice** | 분기 |
| **Parallel** | 병렬 실행 |
| **Map** | 배열 반복 |
| **Wait** | 지연 |
| **Pass/Succeed/Fail** | 흐름 제어 |

💡 "**ETL 여러 단계 + 에러 처리/재시도**" → Step Functions  
💡 "**수작업 승인 (Human Approval)**" → Step Functions + Task Token 패턴  
💡 "**Lambda만으로 복잡한 조건 분기**" 코드 대신 → Step Functions Choice

---

## 7. 시험 빈출 시나리오

### 시나리오 1: "주문이 유실되지 않도록 + 처리 지연 허용"

**정답**: **SQS** (메시지 큐 버퍼)  
**오답 함정**: SNS (저장 없음, 구독자 장애 시 유실)

### 시나리오 2: "하나의 주문 이벤트 → 재고, 결제, 알림 각각 처리"

**정답**: **SNS + SQS 팬아웃** (각 서비스가 자체 큐로 받아 독립 처리)

### 시나리오 3: "Shopify 주문 이벤트를 AWS로 수신"

**정답**: **EventBridge Partner Event Source**

### 시나리오 4: "S3에 파일 업로드 → 여러 Lambda 체인 실행 + 에러 시 재시도 + 분기"

**정답**: **Step Functions** (S3 이벤트 → EventBridge → Step Functions 시작)  
**오답 함정**: Lambda가 Lambda를 직접 호출 (강결합, 에러 처리 어려움)

### 시나리오 5: "EC2 상태 변경 시 슬랙 알림"

**정답**: **EventBridge Rule** (기본 버스, EC2 state-change 패턴) → **SNS/Lambda → Slack**

### 시나리오 6: "매일 새벽 3시 배치 작업"

**정답**: **EventBridge Scheduler** → Lambda/Step Functions  
(구: CloudWatch Events cron rule)

### 시나리오 7: "소비자 처리 실패 메시지 보관 + 재처리"

**정답**: **SQS DLQ** (maxReceiveCount 초과 시 이동)

### 시나리오 8: "주문 순서 엄격 보장"

**정답**: **SQS FIFO Queue** (MessageGroupId 기준 순서)

### 시나리오 9: "Lambda 장애 시 이벤트 재생"

**정답**: **EventBridge Archive + Replay**

### 시나리오 10: "수작업 승인 후 다음 단계 진행"

**정답**: **Step Functions + Task Token** (Callback 패턴)

---

## 8. 빠른 매핑 치트표

| 키워드 | 정답 |
|--------|------|
| "버퍼링 / 스파이크 흡수" | **SQS** |
| "팬아웃 (1:N)" | **SNS** |
| "SaaS 이벤트 통합" | **EventBridge** |
| "AWS 서비스 상태 변경" | **EventBridge 기본 버스** |
| "cron 스케줄" | **EventBridge Scheduler** |
| "워크플로우 + 분기 + 재시도" | **Step Functions** |
| "수작업 승인" | **Step Functions Task Token** |
| "순서 보장" | **SQS FIFO** |
| "실패 메시지 격리" | **SQS DLQ** |
| "빈 응답 비용 절감" | **Long Polling** |
| "이메일/SMS 알림" | **SNS** |
| "이벤트 재생 (디버깅)" | **EventBridge Archive/Replay** |
| "고처리량 단기 워크플로우" | **Step Functions Express** |
| "여러 소스 → 필터 → 타겟 파이프" | **EventBridge Pipes** |

---

## 9. 함정 / 주의사항

### 함정 1: SNS는 메시지 저장하지 않는다

- 구독자가 받지 못하면 **유실** (재전송만 제한적)
- 안전하게 받으려면 → **SNS → SQS 팬아웃** (SQS가 보관)
- 💡 "유실 방지" = SQS가 필요

### 함정 2: Visibility Timeout 설정 실수

- **Lambda 처리 시간 > Visibility Timeout** → 다른 소비자가 중복 처리
- 권장: **Lambda 함수 타임아웃 × 6** (AWS 공식 권장)
- 💡 "메시지 중복 처리됨" → Visibility Timeout 늘리기

### 함정 3: SQS Standard = At-Least-Once (중복 가능)

- 소비자는 **멱등성(idempotent)** 보장해야 함
- 정확히 1회 필요 → **FIFO Queue**
- 💡 "exactly-once" 키워드 → FIFO

### 함정 4: EventBridge vs SNS 선택

- SNS: **속도 + 높은 팬아웃 수 + 단순 알림**
- EventBridge: **복잡한 라우팅 + SaaS + 스키마**
- 💡 단순 "알림 팬아웃" → SNS / 복잡한 "이벤트 라우팅" → EventBridge

### 함정 5: Step Functions Standard vs Express

- **Standard**: 정확 1회, 1년, 비쌈, 실행 이력 보존
- **Express**: At-least-once, 5분, 싸고 빠름, CloudWatch Logs만
- 💡 "고처리량 IoT/스트림 처리" → Express / "중요 비즈니스 프로세스" → Standard

### 함정 6: Lambda → Lambda 직접 호출 = 강결합

- 동기 호출은 호출자 타임아웃/리트라이 관리 필요
- 💡 "loosely coupled" → 중간에 **SQS/SNS/EventBridge/Step Functions** 삽입

---

## 10. 검증 필요 항목 ⚠️

- [ ] SQS FIFO 최신 처리량 한도 (300 → 3,000 배치 → 고처리량 모드?)
- [ ] EventBridge Pipes 최신 Source/Target 지원 목록
- [ ] Step Functions Express 최대 실행 시간 (5분 유지?)
- [ ] SNS FIFO 토픽 최대 처리량
- [ ] EventBridge Scheduler vs CloudWatch Events Rule 권장

---

## 11. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "Decouple two components" → SQS / SNS / EventBridge 중 선택
- **유형 2**: "순서 보장 / 중복 제거" → SQS FIFO
- **유형 3**: "SaaS 이벤트 / AWS 서비스 이벤트" → EventBridge
- **유형 4**: "복잡한 워크플로우 + 재시도" → Step Functions
- **유형 5**: "팬아웃 (1개 이벤트 → N개 소비자)" → SNS + SQS
- **유형 6**: "메시지 유실 방지 / 스파이크 버퍼" → SQS

---

## 12. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
