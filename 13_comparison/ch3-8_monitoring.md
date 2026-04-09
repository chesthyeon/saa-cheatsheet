# Chapter 3-8. 모니터링: CloudWatch vs CloudTrail vs Config vs X-Ray

## 0. 한 줄 요약

🔑 **"무엇을 관찰하는가" — 성능 메트릭(CloudWatch) / API 감사 로그(CloudTrail) / 리소스 구성 변경(Config) / 분산 추적(X-Ray)으로 4분할된다**

---

## 1. 전체 비교 매트릭스

| 기준 | CloudWatch | CloudTrail | Config | X-Ray |
|------|-----------|------------|--------|-------|
| **관찰 대상** | **성능 메트릭 + 로그** | **API 호출 (누가/언제/무엇)** | **리소스 구성 상태/변경** | **분산 애플리케이션 추적** |
| **질문** | "리소스가 잘 작동하는가?" | "누가 무엇을 했는가?" | "리소스 설정이 규정에 맞는가?" | "요청이 어디서 느려지는가?" |
| **데이터 유형** | 메트릭, 로그, 이벤트 | 이벤트 기록 (JSON) | 구성 항목 (Configuration Item) | 트레이스, 세그먼트 |
| **기본 보존** | 메트릭: 15개월 / 로그: 무기한 | 90일 (이벤트 기록) | 구성 기록기 활성화 시 S3 저장 | 30일 |
| **알림** | ✅ **CloudWatch Alarms** | ❌ (EventBridge 연동) | ❌ (SNS 연동) | ❌ |
| **대시보드** | ✅ **커스텀 대시보드** | ❌ | ✅ 규정 준수 대시보드 | ✅ 서비스 맵 |
| **자동 조치** | ✅ **Auto Scaling, EC2 복구** | ❌ | ✅ **자동 수정 (Remediation)** | ❌ |
| **비용** | 메트릭/로그/알람 별 | 관리 이벤트 무료 (1트레일) | 기록 항목 + 규칙 평가 | 트레이스 기록 수 |

---

## 2. 핵심 판별 플로우차트

```
"모니터링/감사 서비스 선택"
  │
  ├─ 성능 모니터링 (CPU, 메모리, 에러율)?
  │   ├─ 메트릭 수집 + 알림? ──→ CloudWatch Metrics + Alarms
  │   ├─ 로그 수집 + 분석? ──→ CloudWatch Logs
  │   └─ 커스텀 메트릭 (메모리, 디스크)? ──→ CloudWatch Agent
  │
  ├─ 감사/보안 (누가 무엇을 했는가)?
  │   ├─ API 호출 기록? ──→ CloudTrail
  │   ├─ 장기 보관 (S3)? ──→ CloudTrail + S3 Trail
  │   └─ 데이터 이벤트 (S3 객체, Lambda 호출)? ──→ CloudTrail 데이터 이벤트
  │
  ├─ 리소스 구성 규정 준수?
  │   ├─ 설정 변경 추적? ──→ Config
  │   ├─ 규정 위반 탐지 + 자동 수정? ──→ Config Rules + Remediation
  │   └─ "이 리소스가 과거에 어떤 설정이었나?" ──→ Config 타임라인
  │
  └─ 애플리케이션 성능 추적?
      └─ 마이크로서비스 병목/지연 분석? ──→ X-Ray
```

---

## 3. 쌍별 상세 비교

### CloudWatch vs CloudTrail

| 기준 | CloudWatch | CloudTrail |
|------|-----------|------------|
| 목적 | **성능 모니터링** (리소스 상태) | **감사 로깅** (API 활동) |
| 관찰 대상 | CPU, 네트워크, 에러율, 로그 | 누가 어떤 API를 호출했는가 |
| 시간 기준 | 실시간 메트릭 (~1분/5분) | 이벤트 발생 후 ~15분 내 기록 |
| 알림 | ✅ CloudWatch Alarms | ❌ (EventBridge로 감지) |
| 사용 사례 | 오토스케일링 트리거, 장애 감지 | 보안 감사, 규정 준수, 포렌식 |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "CPU 사용률 모니터링" | **CloudWatch** |
| "알림", "경보" | **CloudWatch Alarms** |
| "누가 S3 버킷을 삭제했는가" | **CloudTrail** |
| "API 감사 로그" | **CloudTrail** |
| "보안 그룹 변경 감지" (누가 변경?) | **CloudTrail** |
| "보안 그룹 규정 준수 확인" (올바른 설정?) | **Config** |

### CloudTrail vs Config

| 기준 | CloudTrail | Config |
|------|------------|--------|
| 기록 내용 | **API 활동** (누가 무엇을 했는가) | **리소스 구성 상태** (어떤 설정인가) |
| 질문 | "누가 보안 그룹을 변경했나?" | "보안 그룹이 0.0.0.0/0을 허용하나?" |
| 시간 관점 | 이벤트 시점 기록 | **구성 타임라인** (과거 상태 추적) |
| 규칙/평가 | ❌ | ✅ **Config Rules** (규정 준수 평가) |
| 자동 수정 | ❌ | ✅ **Auto Remediation** (SSM 연동) |
| 관계 탐색 | ❌ | ✅ 리소스 관계 시각화 |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "감사", "누가 했는가", "API 로그" | **CloudTrail** |
| "규정 준수", "구성 평가", "규칙 위반" | **Config** |
| "리소스 변경 이력" (설정이 어떻게 바뀌었나) | **Config** |
| "리소스 변경 이력" (누가 바꿨나) | **CloudTrail** |
| "비준수 리소스 자동 수정" | **Config Rules + Remediation** |

### CloudWatch Logs vs CloudTrail Logs

| 기준 | CloudWatch Logs | CloudTrail Logs |
|------|----------------|----------------|
| 데이터 소스 | **애플리케이션/OS 로그** (커스텀) | **AWS API 호출** (자동) |
| 수집 방식 | CloudWatch Agent / SDK | 자동 (AWS 서비스 연동) |
| 검색 | Logs Insights (쿼리) | Athena (S3 Trail) / CloudTrail Lake |
| 보존 | 커스텀 (1일~무기한) | 이벤트 기록: 90일 / S3: 무기한 |
| 사용 사례 | 앱 에러 로그, 웹 서버 로그 | 보안 감사, 규정 준수 |

### X-Ray vs CloudWatch

| 기준 | X-Ray | CloudWatch |
|------|-------|-----------|
| 관점 | **요청 흐름** (End-to-End) | **리소스 메트릭** (개별) |
| 시각화 | **서비스 맵** (의존성 그래프) | 메트릭 대시보드 |
| 병목 식별 | ✅ 어떤 서비스에서 지연 발생 | 개별 서비스 메트릭만 |
| 적용 방식 | 코드에 SDK/데몬 추가 | 자동 (기본 메트릭) + Agent |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "분산 추적", "서비스 맵" | **X-Ray** |
| "마이크로서비스 병목 분석" | **X-Ray** |
| "Lambda 실행 시간 모니터링" | **CloudWatch** (메트릭) |
| "Lambda 내부 함수 호출 추적" | **X-Ray** (세그먼트) |

---

## 4. CloudWatch 핵심 기능 정리

### 주요 구성 요소

| 구성 요소 | 역할 | 시험 키워드 |
|----------|------|-----------|
| **Metrics** | 수치 데이터 수집 (CPU, 네트워크 등) | "메트릭", "모니터링" |
| **Alarms** | 임계값 초과 시 알림/조치 | "경보", "Auto Scaling 트리거" |
| **Logs** | 로그 수집·저장·검색 | "로그 분석", "Logs Insights" |
| **Events/EventBridge** | 상태 변경 감지 → 타겟 실행 | "이벤트 기반", "cron" |
| **Agent** | EC2 메모리/디스크 커스텀 메트릭 | "메모리 사용률", "디스크 사용률" |
| **Dashboards** | 커스텀 시각화 | "운영 대시보드" |

### CloudWatch Agent 핵심

- 기본 EC2 메트릭: CPU, 네트워크, 디스크 I/O, 상태 체크
- **기본 미포함**: 메모리 사용률, 디스크 사용률 → **CloudWatch Agent 필요**
- 💡 "메모리 모니터링" → CloudWatch Agent (반드시 설치)

---

## 5. 시험 빈출 시나리오 + 정답

| 시나리오 | 정답 | 오답 함정 |
|----------|------|----------|
| EC2 CPU 80% 초과 시 알림 | **CloudWatch Alarm + SNS** | CloudTrail (성능이 아닌 감사) |
| EC2 메모리 사용률 모니터링 | **CloudWatch Agent** | 기본 CloudWatch (메모리 미포함) |
| "누가 보안 그룹을 변경했나?" | **CloudTrail** | Config (누구인지가 아닌 설정 상태) |
| "보안 그룹이 0.0.0.0/0 허용하면 알림" | **Config Rule + SNS** | CloudTrail (규정 평가 아님) |
| "비준수 리소스 자동 수정" | **Config Rule + Auto Remediation** | CloudWatch (구성 평가 불가) |
| S3 객체 삭제 감사 | **CloudTrail 데이터 이벤트** | CloudWatch (API 감사 불가) |
| 마이크로서비스 지연 병목 분석 | **X-Ray** | CloudWatch (개별 메트릭만) |
| 리소스 구성 변경 타임라인 | **Config** | CloudTrail (API 이벤트, 구성 상태 아님) |
| 모든 리전 API 감사 로그 중앙화 | **CloudTrail Organization Trail** | 리전별 개별 Trail (관리 복잡) |
| CPU 기반 Auto Scaling 트리거 | **CloudWatch Alarm + ASG 정책** | Config (스케일링 아님) |

---

## 6. 헷갈리는 포인트 / 함정

### 함정 1: "변경 감지" 문맥 주의

- "누가 변경했나" → **CloudTrail** (감사)
- "무엇이 변경되었나" → **Config** (구성 상태)
- "변경으로 성능이 떨어졌나" → **CloudWatch** (메트릭)

### 함정 2: CloudTrail 이벤트 유형

- **관리 이벤트**: API 호출 (CreateBucket, RunInstances) → 기본 무료 (1개 트레일)
- **데이터 이벤트**: S3 GetObject, Lambda Invoke → **유료, 별도 활성화**
- 💡 "S3 객체 접근 감사" → CloudTrail **데이터 이벤트** (기본 비활성화)

### 함정 3: Config ≠ 실시간 차단

- Config은 **사후 평가** (변경 발생 후 규정 위반 탐지)
- **사전 차단**이 필요하면 → **SCP** 또는 **IAM 정책**
- 💡 "규정 위반 방지" → SCP/IAM, "규정 위반 탐지 + 수정" → Config

---

## 7. 검증 필요 항목 ⚠️

- [ ] CloudWatch 메트릭 기본 보존 기간 (5분: 63일, 1시간: 455일?)
- [ ] CloudTrail Lake 최신 요금 및 쿼리 기능
- [ ] Config Conformance Packs 최신 사전 정의 팩 목록
- [ ] X-Ray → CloudWatch ServiceLens 통합 변경 사항
- [ ] CloudWatch Agent 최신 지원 메트릭 목록
- [ ] Config Auto Remediation SSM 자동화 문서 최신 목록

---

## 8. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "성능 모니터링 + 알림" → CloudWatch
- **유형 2**: "API 감사/보안" → CloudTrail
- **유형 3**: "구성 규정 준수" → Config
- **유형 4**: "분산 추적/병목 분석" → X-Ray
- **유형 5**: "변경 감지" 문맥 판별 → CloudTrail vs Config vs CloudWatch

---

## 9. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
