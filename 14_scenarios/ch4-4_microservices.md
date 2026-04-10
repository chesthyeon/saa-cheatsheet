# Chapter 4-4. 마이크로서비스·디커플링 (API Gateway + SQS/SNS + Lambda/Fargate)

## 0. 한 줄 요약

🔑 **"느슨한 결합(Loose Coupling)" = 서비스 간 직접 호출 대신 SQS/SNS/EventBridge로 중간에 버퍼를 두는 것 — "디커플링" 키워드가 보이면 메시징 서비스 조합**

---

## 1. 아키텍처 다이어그램

### 패턴 A: 비동기 큐 기반 (SQS)

```
API Gateway → Lambda (주문 접수)
                  │
                  ▼
              SQS Queue (버퍼)
                  │
                  ▼
         Lambda / Fargate (주문 처리)
                  │
                  ▼
              DynamoDB (저장)
```

### 패턴 B: 팬아웃 (SNS + SQS)

```
Lambda (이벤트 발생)
    │
    ▼
SNS Topic (팬아웃)
    ├──→ SQS Queue A → Lambda (이메일 발송)
    ├──→ SQS Queue B → Lambda (재고 업데이트)
    └──→ SQS Queue C → Lambda (분석 로깅)
```

### 패턴 C: 이벤트 기반 (EventBridge)

```
AWS 서비스 / 커스텀 앱 → EventBridge (이벤트 버스)
                              │
                    ┌─ Rule 1 ─┼─ Rule 2 ─┐
                    ▼          ▼          ▼
                Lambda     Step Fn     SQS Queue
```

### 패턴 D: 오케스트레이션 (Step Functions)

```
API Gateway → Step Functions (워크플로우)
                  │
          ┌───────┼───────┐
          ▼       ▼       ▼
      Lambda   Lambda   Lambda
     (검증)   (결제)   (배송)
          │       │       │
          └───────┼───────┘
                  ▼
            SNS (완료 알림)
```

---

## 2. 디커플링 서비스 비교

| 기준 | SQS | SNS | EventBridge | Step Functions |
|------|-----|-----|-------------|----------------|
| **패턴** | **점대점** (1:1) | **팬아웃** (1:N) | **이벤트 라우팅** (규칙 기반) | **오케스트레이션** (순서 제어) |
| **통신 방식** | 비동기 (폴링) | 비동기 (푸시) | 비동기 (푸시) | 동기/비동기 (상태 머신) |
| **메시지 보존** | ✅ (최대 14일) | ❌ (즉시 전달) | ❌ (즉시 전달) | ✅ (실행 이력) |
| **재시도** | ✅ (자동) | 재전달 정책 | 재시도 + DLQ | ✅ (내장 에러 처리) |
| **순서 보장** | FIFO만 | FIFO 미지원 | 순서 미보장 | ✅ (워크플로우 순서) |
| **시험 키워드** | "버퍼링", "처리 속도 차이" | "다중 구독자", "팬아웃" | "이벤트 기반", "규칙 라우팅" | "워크플로우", "상태 관리" |

---

## 3. 핵심 디커플링 패턴 상세

### 패턴 1: SQS 큐 디커플링

**사용 시나리오**: 생산자-소비자 속도 차이 흡수

```
[빠른 생산자] → SQS Queue → [느린 소비자]
                   │
                   └─ DLQ (실패 메시지 격리)
```

| 키워드 | 구성 |
|--------|------|
| "처리 속도 차이 흡수" | **SQS Standard Queue** |
| "정확히 한 번 처리" + "순서 보장" | **SQS FIFO Queue** |
| "실패 메시지 격리" | **Dead Letter Queue (DLQ)** |
| "메시지 숨기기" | **Visibility Timeout** |
| "지연 전달" | **Delay Queue** (0~15분) |

**SQS Standard vs FIFO:**

| 기준 | Standard | FIFO |
|------|---------|------|
| 처리량 | **무제한** | 300 msg/s (배치: 3,000) |
| 순서 | 최선 노력 | **보장** |
| 중복 | 가능 (At-least-once) | **없음** (Exactly-once) |
| 사용 사례 | 로그 수집, 이메일 | **결제, 주문** |

### 패턴 2: SNS + SQS 팬아웃

**사용 시나리오**: 하나의 이벤트 → 다중 소비자

- SNS Topic에 여러 SQS Queue 구독
- 각 큐는 독립적으로 처리 (하나가 실패해도 다른 것에 영향 없음)
- 💡 "하나의 이벤트로 여러 서비스 트리거" → **SNS + SQS 팬아웃**

### 패턴 3: EventBridge 이벤트 기반

**사용 시나리오**: 복잡한 이벤트 라우팅 규칙

| EventBridge 장점 | 설명 |
|------------------|------|
| **규칙 기반 라우팅** | 이벤트 내용으로 타겟 결정 |
| **스키마 레지스트리** | 이벤트 구조 자동 검색 |
| **AWS 서비스 통합** | 100+ AWS 서비스 이벤트 자동 수신 |
| **SaaS 통합** | Zendesk, Datadog 등 외부 이벤트 |
| **크로스 계정/리전** | 이벤트 버스 간 전달 |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "이벤트 내용 기반 라우팅" | **EventBridge** |
| "AWS 서비스 상태 변경 감지" | **EventBridge** |
| "cron/스케줄 기반 실행" | **EventBridge Scheduler** |
| "단순 팬아웃" (라우팅 불필요) | **SNS** |

### 패턴 4: Step Functions 오케스트레이션

**사용 시나리오**: 복잡한 다단계 워크플로우

| 기능 | 설명 |
|------|------|
| **상태 머신** | 시각적 워크플로우 정의 (ASL) |
| **에러 처리** | Retry, Catch (내장) |
| **병렬 실행** | Parallel 상태 |
| **조건 분기** | Choice 상태 |
| **대기** | Wait 상태 (타이머) |
| **맵** | 배열 항목 반복 처리 |

**Standard vs Express:**

| 기준 | Standard | Express |
|------|---------|---------|
| 실행 시간 | **최대 1년** | **최대 5분** |
| 실행 보장 | Exactly-once | At-least-once |
| 비용 | 상태 전환당 | 실행 횟수 + 시간 |
| 사용 사례 | 주문 처리, 승인 | 대량 데이터, 스트리밍 |

---

## 4. 시험 빈출 변형 시나리오

### 시나리오 1: 서비스 간 디커플링

> "모놀리식 앱을 마이크로서비스로 전환. 서비스 간 의존성을 줄여야 한다."

**정답**: 서비스 사이에 **SQS Queue** 배치 (비동기 통신)  
**오답 함정**: 직접 HTTP 호출 (강한 결합, 장애 전파)

### 시나리오 2: 하나의 이벤트 → 다중 처리

> "주문 완료 시 이메일 발송 + 재고 업데이트 + 분석 로깅을 해야 한다."

**정답**: **SNS Topic → 3개 SQS Queue → 각각 Lambda**  
**대안**: **EventBridge → 3개 타겟**  
**오답 함정**: Lambda에서 3개 서비스 직접 호출 (강한 결합)

### 시나리오 3: 트래픽 급증 보호

> "갑작스런 트래픽 급증 시 백엔드가 과부하되지 않도록 해야 한다."

**정답**: **SQS Queue** (버퍼) + 소비자 Auto Scaling  
**대안**: API Gateway **스로틀링**  
**오답 함정**: ALB만 사용 (버퍼링 없음, 백엔드 직접 부하)

### 시나리오 4: 복잡한 워크플로우

> "주문 검증 → 결제 → 재고 확인 → 배송 순서를 관리해야 한다."

**정답**: **Step Functions** (상태 머신으로 순서 + 에러 처리)  
**오답 함정**: SQS 체인 (순서 관리 어려움, 에러 처리 복잡)

### 시나리오 5: FIFO 메시징

> "메시지 순서가 보장되어야 하고 중복 처리를 방지해야 한다."

**정답**: **SQS FIFO Queue** (순서 보장 + Exactly-once)  
**오답 함정**: SQS Standard (순서 미보장, 중복 가능)

### 시나리오 6: AWS 서비스 이벤트 반응

> "EC2 상태가 terminated로 변경되면 자동으로 슬랙 알림을 보내야 한다."

**정답**: **EventBridge Rule** (EC2 상태 변경) → Lambda (슬랙 API) 또는 SNS  
**오답 함정**: CloudWatch Events (EventBridge로 대체됨, 같은 서비스)

### 시나리오 7: 마이크로서비스 컨테이너 운영

> "마이크로서비스를 컨테이너로 운영. 서버 관리를 최소화해야 한다."

**정답**: **Fargate** (서버리스 컨테이너) + ALB  
**대안**: ECS on EC2 (서버 관리 있지만 비용 최적화 가능)  
**오답 함정**: Lambda (컨테이너 이미지 지원하지만 15분 제한)

### 시나리오 8: Saga 패턴 (보상 트랜잭션)

> "결제 후 배송 실패 시 결제를 자동으로 취소해야 한다."

**정답**: **Step Functions** (Catch → 보상 Lambda 호출)  
**오답 함정**: SQS DLQ (실패 격리만, 보상 로직 없음)

---

## 5. 핵심 키워드 → 서비스 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "디커플링", "느슨한 결합" | **SQS / SNS / EventBridge** |
| "버퍼링", "속도 차이 흡수" | **SQS Queue** |
| "팬아웃", "다중 구독자" | **SNS + SQS** |
| "이벤트 기반 라우팅" | **EventBridge** |
| "워크플로우 오케스트레이션" | **Step Functions** |
| "순서 보장" + "중복 방지" | **SQS FIFO** |
| "서버리스 컨테이너" | **Fargate** |
| "보상 트랜잭션", "Saga" | **Step Functions** |
| "실패 메시지 격리" | **SQS DLQ** |
| "스케줄 기반 실행" | **EventBridge Scheduler** |

---

## 6. 헷갈리는 포인트 / 함정

### 함정 1: SQS vs SNS 선택

- **SQS**: 1:1 (하나의 소비자가 메시지 처리 후 삭제)
- **SNS**: 1:N (모든 구독자에게 동시 전달)
- 💡 "처리 후 삭제" → SQS, "모두에게 알림" → SNS

### 함정 2: EventBridge vs SNS

- **EventBridge**: 규칙 기반 라우팅 (이벤트 내용 필터링), AWS 서비스 통합
- **SNS**: 단순 팬아웃 (필터 정책은 있으나 제한적)
- 💡 "복잡한 라우팅", "AWS 서비스 이벤트" → EventBridge
- 💡 "단순 알림", "SMS/이메일" → SNS

### 함정 3: Step Functions Standard vs Express

- 💡 "장시간 워크플로우" (주문 처리, 승인) → **Standard** (최대 1년)
- 💡 "대량 고속 처리" (IoT, 스트리밍) → **Express** (최대 5분)

### 함정 4: SQS Visibility Timeout

- 소비자가 메시지를 수신하면 다른 소비자에게 숨김
- 처리 완료 전 타임아웃 → 메시지 재출현 (중복 처리 가능)
- 💡 Visibility Timeout > 처리 시간으로 설정

### 함정 5: API Gateway → Lambda → SQS vs API Gateway → SQS 직접

- **API Gateway → SQS 직접 통합**: Lambda 불필요, 비용 절감, 지연 감소
- 💡 "단순 큐 넣기만" → API GW → SQS 직접 (서비스 프록시)
- 💡 "비즈니스 로직 필요" → API GW → Lambda → SQS

---

## 7. 검증 필요 항목 ⚠️

- [ ] SQS FIFO 최신 처리량 한도 (높은 처리량 모드?)
- [ ] EventBridge Pipes 최신 기능
- [ ] Step Functions Distributed Map 최신 기능
- [ ] API Gateway → SQS 직접 통합 설정 방법 최신
- [ ] EventBridge Scheduler vs CloudWatch Events cron 비교

---

## 8. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "디커플링/느슨한 결합" → SQS/SNS/EventBridge 조합
- **유형 2**: "팬아웃" → SNS + SQS
- **유형 3**: "워크플로우/오케스트레이션" → Step Functions
- **유형 4**: "트래픽 급증 보호" → SQS 버퍼링
- **유형 5**: "이벤트 기반 아키텍처" → EventBridge

---

## 9. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
