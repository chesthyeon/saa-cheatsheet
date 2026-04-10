# Chapter 4-3. 완전 서버리스 (API Gateway + Lambda + DynamoDB + Cognito)

## 0. 한 줄 요약

🔑 **"서버 관리 없이 API 구축" = API Gateway(진입) + Lambda(로직) + DynamoDB(저장) + Cognito(인증) — "운영 오버헤드 최소화" 키워드가 보이면 이 조합**

---

## 1. 아키텍처 다이어그램

```
사용자 (웹/모바일)
  │
  ├─ 인증: Cognito User Pool (회원가입/로그인 → JWT)
  │
  ▼
API Gateway (REST/HTTP API)
  │  ├─ Cognito Authorizer (JWT 검증)
  │  ├─ 스로틀링 / API 키 / 사용량 계획
  │  └─ 스테이지: dev / staging / prod
  │
  ▼
Lambda (비즈니스 로직)
  │  ├─ 실행 역할 (IAM Role)
  │  ├─ 환경 변수 (KMS 암호화)
  │  └─ 동시성: Reserved / Provisioned
  │
  ├──→ DynamoDB (NoSQL 데이터 저장)
  ├──→ S3 (파일 업로드/다운로드)
  ├──→ SQS (비동기 처리)
  └──→ SNS (알림 발송)
```

---

## 2. 구성 요소별 역할 + 키워드 매핑

| 구성 요소 | 역할 | 시험 키워드 |
|----------|------|-----------|
| **API Gateway** | API 진입점 (REST/HTTP/WebSocket) | "API 엔드포인트", "스로틀링" |
| **Lambda** | 이벤트 기반 컴퓨팅 (서버 없음) | "서버리스", "이벤트 처리", "15분 제한" |
| **DynamoDB** | 서버리스 NoSQL (ms 지연) | "키-값 저장", "서버리스 DB" |
| **Cognito User Pool** | 사용자 인증 (JWT 발급) | "회원가입/로그인", "소셜 로그인" |
| **Cognito Identity Pool** | AWS 자격 증명 발급 | "S3 직접 접근", "임시 자격 증명" |
| **S3** | 정적 자산 / 파일 저장 | "파일 업로드", "프론트엔드 호스팅" |

---

## 3. API Gateway 유형 비교

| 기준 | REST API | HTTP API | WebSocket API |
|------|---------|---------|---------------|
| 기능 | 풀 기능 (캐싱, WAF, 사용량 계획) | 경량화 (프록시 전용) | 양방향 실시간 |
| 비용 | 높음 | **~70% 저렴** | 연결 시간 + 메시지 수 |
| 인증 | Cognito, Lambda, IAM | Cognito, Lambda, IAM, JWT | Lambda |
| 캐싱 | ✅ | ❌ | ❌ |
| WAF 통합 | ✅ | ❌ | ❌ |
| 사용 사례 | 엔터프라이즈 API | **단순 프록시/CRUD** | **채팅, 알림, 실시간** |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "비용 최적화" + "단순 API" | **HTTP API** |
| "API 캐싱", "WAF 필요" | **REST API** |
| "실시간 양방향", "채팅" | **WebSocket API** |

---

## 4. Lambda 핵심 제약 + 설정

### 리소스 제한

| 항목 | 제한 |
|------|------|
| 최대 실행 시간 | **15분** (900초) |
| 메모리 | 128 MB ~ 10,240 MB |
| /tmp 디스크 | 512 MB ~ 10,240 MB |
| 동시 실행 | 리전당 기본 1,000 (증가 가능) |
| 배포 패키지 | 50 MB (압축) / 250 MB (비압축) |
| 환경 변수 | 4 KB |

### 동시성 유형

| 유형 | 설명 | 시험 키워드 |
|------|------|-----------|
| **Unreserved** | 기본 (공유 풀) | 기본값 |
| **Reserved** | 함수에 동시성 할당량 예약 | "다른 함수 영향 방지" |
| **Provisioned** | 사전 초기화 (Cold Start 제거) | "Cold Start 제거", "일관된 지연" |

### Cold Start 대응

| 방법 | 설명 |
|------|------|
| **Provisioned Concurrency** | 가장 확실 (비용 발생) |
| SnapStart (Java) | JVM 스냅샷 (Java 전용) |
| 경량 런타임 | Python/Node.js (Go는 네이티브 바이너리) |
| 💡 "Cold Start" + "비용 무관" → **Provisioned Concurrency** |

---

## 5. DynamoDB 핵심 설정

### 용량 모드

| 모드 | 설명 | 시험 키워드 |
|------|------|-----------|
| **Provisioned** | RCU/WCU 사전 할당 | "예측 가능한 워크로드", "비용 최적화" |
| **On-Demand** | 자동 확장 (사용한 만큼) | "예측 불가", "운영 오버헤드 최소" |

### 주요 기능

| 기능 | 설명 | 시험 키워드 |
|------|------|-----------|
| **DAX** | DynamoDB 인메모리 캐시 (μs 지연) | "마이크로초 지연", "읽기 캐싱" |
| **Global Tables** | 멀티 리전 복제 (Active-Active) | "글로벌 사용자", "멀티 리전" |
| **DynamoDB Streams** | 변경 데이터 캡처 (CDC) | "변경 감지", "이벤트 트리거" |
| **TTL** | 자동 항목 만료 삭제 | "자동 삭제", "세션 만료" |
| **Point-in-Time Recovery** | 초 단위 백업 복원 (35일) | "실수로 삭제", "복원" |

---

## 6. 시험 빈출 변형 시나리오

### 시나리오 1: 기본 서버리스 API

> "운영 오버헤드를 최소화하면서 CRUD API를 구축해야 한다."

**정답**: API Gateway (HTTP API) + Lambda + DynamoDB  
**오답 함정**: EC2 + RDS (서버 관리 필요)

### 시나리오 2: 사용자 인증이 필요한 API

> "서버리스 API에 회원가입/로그인 기능을 추가해야 한다."

**정답**: **Cognito User Pool** (인증) + API Gateway **Cognito Authorizer**  
**오답 함정**: 자체 인증 Lambda (운영 오버헤드)

### 시나리오 3: Lambda 15분 초과 작업

> "대용량 파일 처리가 15분을 초과한다."

**정답 옵션:**
1. **Step Functions** + Lambda (작업 분할 + 오케스트레이션)
2. **AWS Batch** 또는 **Fargate** (장시간 작업)
3. S3 Multipart Upload + 분할 처리

**오답 함정**: Lambda 단독 (15분 제한 초과 불가)

### 시나리오 4: 대규모 파일 업로드

> "사용자가 대용량 파일을 서버리스 아키텍처로 업로드해야 한다."

**정답**: **S3 Pre-signed URL** (Lambda가 URL 생성 → 클라이언트 직접 S3 업로드)  
**오답 함정**: API Gateway 경유 (페이로드 10 MB 제한)

### 시나리오 5: DynamoDB 읽기 성능

> "DynamoDB 읽기 지연 시간을 마이크로초로 줄여야 한다."

**정답**: **DAX** (DynamoDB Accelerator)  
**오답 함정**: ElastiCache (DynamoDB 전용이 아님, DAX가 더 통합적)

### 시나리오 6: 서버리스 웹앱 전체 구성

> "프론트엔드 + 백엔드 + DB를 서버리스로 구축."

**정답**: S3 (정적 호스팅) + CloudFront + API Gateway + Lambda + DynamoDB + Cognito  
**핵심**: 모든 계층이 서버리스 = 운영 오버헤드 제로

### 시나리오 7: Lambda 동시성 폭증

> "이벤트 급증으로 Lambda 동시 실행이 한도를 초과한다."

**정답 옵션:**
1. **Reserved Concurrency** (중요 함수 보호)
2. **SQS** 앞에 배치 (버퍼링, Lambda 이벤트 소스 매핑 배치)
3. Service Quotas에서 한도 증가 요청

### 시나리오 8: API 요청 제한

> "특정 클라이언트의 API 호출을 제한해야 한다."

**정답**: API Gateway **Usage Plans + API Keys** (스로틀링)  
**대안**: API Gateway 기본 스로틀링 (계정 레벨)

---

## 7. 핵심 키워드 → 서비스 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "운영 오버헤드 최소화" + "API" | **서버리스 (API GW + Lambda + DynamoDB)** |
| "서버 관리 없이" | **서버리스 아키텍처** |
| "15분 초과 작업" | Lambda ❌ → **Step Functions** / **Fargate** |
| "Cold Start 제거" | **Lambda Provisioned Concurrency** |
| "DynamoDB 마이크로초 지연" | **DAX** |
| "대용량 파일 업로드" + "서버리스" | **S3 Pre-signed URL** |
| "실시간 양방향 통신" | **API Gateway WebSocket** |
| "예측 불가 트래픽" + "DB" | **DynamoDB On-Demand** |
| "웹/모바일 인증" | **Cognito User Pool** |
| "글로벌 DynamoDB" | **Global Tables** |

---

## 8. 헷갈리는 포인트 / 함정

### 함정 1: API Gateway 페이로드 제한

- REST API: **10 MB** 페이로드 제한
- 💡 대용량 파일 → API Gateway 경유 ❌ → **S3 Pre-signed URL** 사용

### 함정 2: Lambda + VPC = Cold Start 증가

- Lambda를 VPC에 배치하면 ENI 생성으로 Cold Start 증가 (과거)
- 현재는 Hyperplane ENI로 개선되었으나 여전히 약간의 오버헤드
- 💡 "VPC 내 RDS 접근" → Lambda를 VPC Private Subnet에 배치 + NAT GW (인터넷)

### 함정 3: DynamoDB Provisioned vs On-Demand 비용

- **Provisioned**: 예측 가능하면 저렴 (+ Auto Scaling)
- **On-Demand**: 예측 불가하면 편리하지만 **단가 ~6배 비쌈**
- 💡 "비용 최적화" + "예측 가능" → **Provisioned + Auto Scaling**
- 💡 "운영 최소화" → **On-Demand**

### 함정 4: Cognito User Pool vs Identity Pool

- **User Pool**: 인증 (로그인 → JWT) + 사용자 관리
- **Identity Pool**: AWS 자격 증명 발급 (S3, DynamoDB 직접 접근)
- 💡 "API Gateway 인증" → **User Pool** (JWT)
- 💡 "S3에 직접 업로드" → **Identity Pool** (임시 자격 증명)

---

## 9. 검증 필요 항목 ⚠️

- [ ] Lambda /tmp 최대 크기 (10,240 MB 확인)
- [ ] API Gateway HTTP API 최신 기능 (JWT 인증 등)
- [ ] DynamoDB On-Demand 최신 단가
- [ ] Lambda SnapStart 지원 런타임 (Java 이외?)
- [ ] API Gateway WebSocket API 최신 제한
- [ ] Lambda 동시 실행 기본 한도 (리전별 1,000? 증가?)

---

## 10. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "서버리스 API 설계" → API GW + Lambda + DynamoDB
- **유형 2**: "Lambda 제한 초과" → Step Functions / Fargate
- **유형 3**: "Cold Start" → Provisioned Concurrency
- **유형 4**: "대용량 업로드" → S3 Pre-signed URL
- **유형 5**: "DynamoDB 성능" → DAX / Global Tables

---

## 11. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
