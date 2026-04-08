# Chapter 1-7. 자격 증명 및 거버넌스

## 0. 한 줄 요약

🔑 **"누가(Principal) 무엇을(Action) 어디에(Resource) 할 수 있는가"를 제어하는 것이 AWS 보안의 전부다 — IAM 정책 평가 로직을 이해하면 절반은 푼 것**

---

## 1. 핵심 서비스 표

| 서비스 | 한 줄 정의 | 주요 키워드 | 가격 모델 |
|--------|-----------|------------|----------|
| **IAM** | AWS 리소스 접근 제어 (사용자/역할/정책) | 사용자, 그룹, 역할, 정책(JSON), MFA | 무료 |
| **IAM Identity Center** (구 SSO) | 다중 AWS 계정 + SaaS 앱 SSO | 중앙 집중 로그인, SAML 2.0, 권한 세트 | 무료 |
| **STS** (Security Token Service) | 임시 보안 자격 증명 발급 | AssumeRole, 임시 토큰, Cross-Account | 무료 |
| **Organizations** | 다중 AWS 계정 중앙 관리 | OU, SCP, 통합 결제, 계정 생성 | 무료 |
| **Control Tower** | 멀티 계정 환경 자동 설정 + 거버넌스 | Landing Zone, 가드레일, 계정 팩토리 | 무료 (사용 리소스 비용만) |
| **AWS Config** | 리소스 구성 변경 추적 + 규정 준수 평가 | 구성 기록, 규칙, 규정 준수 대시보드 | 기록 항목 수 + 규칙 평가 수 |
| **CloudTrail** | API 호출 로깅 (누가 언제 무엇을 했는가) | 감사 로그, 이벤트 기록, S3 저장 | 관리 이벤트 무료 (1개 트레일), 데이터 이벤트 유료 |
| **Cognito** | 웹/모바일 앱 사용자 인증 | User Pool(인증), Identity Pool(AWS 자격 증명), 소셜 로그인 | MAU 기반 |
| **AWS RAM** (Resource Access Manager) | AWS 리소스를 다른 계정과 공유 | Cross-Account 리소스 공유, Transit Gateway/서브넷 공유 | 무료 |
| **Service Quotas** | AWS 서비스 한도 조회 및 상향 요청 | 한도 관리, 쿼터 증가 | 무료 |

---

## 2. 키워드 → 서비스 매핑

### IAM 핵심 개념 매핑

| 시험 키워드 | 정답 구성 요소 |
|------------|--------------|
| "사용자에게 권한 부여" | **IAM 사용자 + IAM 정책** |
| "여러 사용자에게 동일 권한" | **IAM 그룹** (사용자를 그룹에 추가) |
| "AWS 서비스가 다른 서비스에 접근" | **IAM 역할** (서비스 역할) |
| "EC2에서 S3 접근" | **IAM 역할** (인스턴스 프로파일), 액세스 키 ❌ |
| "임시 자격 증명" | **STS AssumeRole** |
| "Cross-Account 접근" | **IAM 역할 + STS AssumeRole** |
| "루트 사용자 보호" | **MFA 활성화** (필수) |
| "최소 권한 원칙" | 필요한 권한만 부여, 와일드카드(*) 최소화 |

### IAM 정책 유형 매핑

| 정책 유형 | 적용 대상 | 특징 |
|----------|----------|------|
| **자격 증명 기반 정책** (Identity-based) | 사용자/그룹/역할에 연결 | "이 주체가 무엇을 할 수 있는가" |
| **리소스 기반 정책** (Resource-based) | S3 버킷, SQS 큐, Lambda 등에 연결 | "이 리소스에 누가 접근 가능한가" |
| **SCP** (Service Control Policy) | OU/계정에 연결 (Organizations) | "이 계정에서 무엇이 허용/차단되는가" (최대 권한 경계) |
| **Permission Boundary** | 사용자/역할에 연결 | "이 주체의 최대 권한 범위" (교집합) |
| **세션 정책** | STS 세션에 적용 | 임시 세션의 권한 제한 |

> ⚠️ **정책 평가 로직**: 명시적 Deny > 명시적 Allow > 기본 Deny (묵시적 거부)

### IAM 역할이 정답인 키워드

| 시험 키워드 | 왜 IAM 역할인가 |
|------------|---------------|
| "EC2에서 S3 접근" | 인스턴스 프로파일 (역할 연결), 액세스 키 하드코딩 ❌ |
| "Lambda가 DynamoDB에 접근" | Lambda 실행 역할 |
| "다른 AWS 계정의 리소스 접근" | Cross-Account 역할 + AssumeRole |
| "외부 IdP 사용자가 AWS에 접근" | SAML 2.0 / OIDC Federation → 역할 |
| "임시 권한 부여" | 역할은 항상 임시 자격 증명 (STS) |

> 💡 **핵심 원칙**: AWS 서비스/외부 사용자가 AWS에 접근 → **항상 IAM 역할** (액세스 키 ❌)

### Organizations / SCP가 정답인 키워드

| 시험 키워드 | 정답 |
|------------|------|
| "다중 AWS 계정 관리" | **Organizations** |
| "통합 결제", "볼륨 할인" | **Organizations** (통합 결제) |
| "특정 서비스 사용 차단" (계정 레벨) | **SCP** |
| "특정 리전에서만 리소스 생성 허용" | **SCP** |
| "루트 사용자 행동 제한" | **SCP** (루트에도 적용됨) |
| "OU(조직 단위)별 정책" | **SCP** + OU 구조 |

> ⚠️ **SCP 핵심**: SCP는 **허용 범위를 제한**하는 것이지, 직접 권한을 부여하지 않음. IAM 정책과의 **교집합**이 실제 권한.

### Control Tower가 정답인 키워드

| 시험 키워드 | 왜 Control Tower인가 |
|------------|-------------------|
| "멀티 계정 환경 자동 설정" | Landing Zone 자동 구성 |
| "가드레일", "규정 준수 자동화" | 예방적/탐지적 가드레일 |
| "새 계정 생성 자동화 + 거버넌스" | Account Factory |
| "Organizations + 보안 기준선 자동 적용" | Control Tower = Organizations 위에 구축 |

### Cognito가 정답인 키워드

| 시험 키워드 | 정답 |
|------------|------|
| "웹/모바일 앱 사용자 인증" | **Cognito User Pool** |
| "소셜 로그인 (Google, Facebook)" | **Cognito User Pool** (외부 IdP 연동) |
| "앱 사용자가 S3/DynamoDB 직접 접근" | **Cognito Identity Pool** (AWS 임시 자격 증명 발급) |
| "사용자 가입/로그인 + MFA" | **Cognito User Pool** |
| "수백만 사용자 인증" | **Cognito** (IAM 사용자 생성 ❌) |

> 💡 **User Pool vs Identity Pool**: User Pool = "누구인가" (인증), Identity Pool = "AWS에서 무엇을 할 수 있는가" (인가/자격 증명)

### CloudTrail vs Config

| 시험 키워드 | 정답 |
|------------|------|
| "누가 이 리소스를 변경했는가" | **CloudTrail** (API 호출 기록) |
| "이 리소스의 구성이 규칙에 맞는가" | **Config** (구성 규정 준수 평가) |
| "API 감사 로그" | **CloudTrail** |
| "보안 그룹이 0.0.0.0/0 허용하면 알림" | **Config Rules** |
| "리소스 구성 변경 이력" | **Config** (타임라인) |
| "S3 버킷이 퍼블릭이면 자동 수정" | **Config Rules + 자동 수정 (SSM)** |

### RAM이 정답인 키워드

| 시험 키워드 | 왜 RAM인가 |
|------------|-----------|
| "다른 계정과 서브넷 공유" | RAM으로 VPC 서브넷 공유 |
| "Transit Gateway를 다른 계정과 공유" | RAM |
| "리소스 복제 없이 다른 계정과 공유" | RAM (리소스 자체를 공유, 복사 아님) |

---

## 3. 헷갈리는 포인트 / 함정

### 함정 1: IAM 역할 vs IAM 사용자 (액세스 키)

- ❌ "EC2에서 S3 접근" → IAM 사용자 액세스 키 → **보안 위험, 틀림**
- ✅ AWS 서비스 간 접근 → **항상 IAM 역할** (임시 자격 증명, 자동 로테이션)
- ✅ IAM 사용자 액세스 키: 외부 CLI/SDK 접근에만 (그마저도 IAM Identity Center 권장)
- 💡 시험에서 "가장 안전한 방법" → IAM 역할이 거의 확정

### 함정 2: SCP는 권한을 부여하지 않는다

- ❌ "SCP에서 S3 Allow → 계정 내 모든 사용자가 S3 접근 가능" → **틀림**
- ✅ SCP는 **최대 허용 범위**를 설정 (천장)
- ✅ 실제 권한 = SCP ∩ IAM 정책 (교집합)
- ✅ SCP에서 Deny → IAM 정책에서 Allow해도 **거부됨**
- 💡 SCP = "할 수 있는 것의 한계", IAM 정책 = "실제 할 수 있는 것"

### 함정 3: Permission Boundary vs SCP

- ❌ 둘을 같은 것으로 혼동 → **적용 레벨이 다름**
- ✅ **SCP**: Organizations의 OU/계정 레벨 (관리자가 계정 전체에 적용)
- ✅ **Permission Boundary**: 개별 IAM 사용자/역할 레벨
- 💡 둘 다 "최대 권한 범위를 제한"하지만, SCP는 계정 단위, PB는 주체 단위

### 함정 4: Cognito User Pool vs Identity Pool

- ❌ 둘을 구분 못함 → **인증 vs 인가**
- ✅ **User Pool**: 사용자 인증 (가입, 로그인, 토큰 발급)
- ✅ **Identity Pool**: AWS 자격 증명 발급 (인증된 사용자 → IAM 역할 매핑)
- 💡 "앱 사용자가 S3에 직접 접근" → User Pool (인증) + Identity Pool (AWS 자격 증명)

### 함정 5: CloudTrail vs Config vs CloudWatch

- ❌ 셋을 혼동 → **각각 역할이 다름**
- ✅ **CloudTrail**: "누가 무엇을 했는가" (API 호출 감사)
- ✅ **Config**: "리소스 구성이 규칙에 맞는가" (규정 준수)
- ✅ **CloudWatch**: "시스템이 어떻게 동작하는가" (메트릭/로그/알람)
- 💡 "감사" → CloudTrail, "규정 준수" → Config, "모니터링" → CloudWatch

### 함정 6: IAM Identity Center vs Cognito

- ❌ "SSO" → 무조건 Cognito → **대상이 다름**
- ✅ **IAM Identity Center**: AWS 계정 + 사내 SaaS 앱 SSO (직원/관리자용)
- ✅ **Cognito**: 외부 웹/모바일 앱 사용자 인증 (고객용)
- 💡 "AWS 콘솔 SSO" → Identity Center, "앱 사용자 로그인" → Cognito

---

## 4. 단골 시나리오

### 시나리오 1: Cross-Account S3 접근

```
문제 패턴: "계정 A의 EC2가 계정 B의 S3 버킷에 접근"
정답 패턴: 계정 B에 IAM 역할 생성 (S3 접근 허용) → 계정 A의 EC2가 STS AssumeRole
대안: S3 버킷 정책에서 계정 A 허용 (리소스 기반 정책)
키 포인트: "Cross-Account" = IAM 역할 + AssumeRole이 가장 안전
```

### 시나리오 2: 멀티 계정 거버넌스

```
문제 패턴: "100개 AWS 계정, 특정 리전에서만 리소스 생성 허용, 중앙 관리"
정답 패턴: Organizations + SCP (리전 제한 정책)
키 포인트: "계정 레벨 서비스 제한" = SCP
```

### 시나리오 3: 웹 앱 사용자 인증 + S3 업로드

```
문제 패턴: "모바일 앱 사용자가 로그인 후 자신의 S3 폴더에 파일 업로드"
정답 패턴: Cognito User Pool (인증) → Cognito Identity Pool (AWS 자격 증명) → S3
키 포인트: "앱 사용자 + AWS 리소스 직접 접근" = User Pool + Identity Pool 조합
```

### 시나리오 4: 다중 계정 직원 SSO

```
문제 패턴: "Active Directory 사용자가 여러 AWS 계정에 SSO로 접근"
정답 패턴: IAM Identity Center + AD Connector 또는 AWS Managed Microsoft AD
키 포인트: "직원 SSO" + "다중 계정" = Identity Center
```

### 시나리오 5: S3 퍼블릭 접근 자동 감지 + 수정

```
문제 패턴: "S3 버킷이 퍼블릭으로 설정되면 자동으로 감지하고 차단"
정답 패턴: AWS Config Rule (s3-bucket-public-read-prohibited) + 자동 수정 (SSM)
키 포인트: "구성 규정 준수 자동화" = Config
```

### 시나리오 6: API 호출 감사 및 이상 행위 탐지

```
문제 패턴: "누가 보안 그룹을 변경했는지 추적, 비정상 API 호출 감지"
정답 패턴: CloudTrail (API 로깅) + CloudTrail Insights (이상 탐지)
키 포인트: "감사" + "누가 했는가" = CloudTrail
```

---

## 5. 검증 필요 항목 ⚠️

> Claude의 학습 데이터가 outdated일 수 있는 항목. AWS 공식 문서에서 확인 필요.

- [ ] IAM Identity Center 최신 기능 (구 AWS SSO에서 리브랜딩 이후 변경 사항)
  - 공식 문서: https://docs.aws.amazon.com/singlesignon/latest/userguide/
- [ ] SCP 최대 정책 크기 및 OU 계층 한도 확인
- [ ] Control Tower 최신 가드레일 목록 및 지원 리전
- [ ] Cognito User Pool 최신 기능 (위협 보호, 고급 보안 등)
- [ ] Config 관리형 규칙 최신 목록
  - 공식 문서: https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html
- [ ] CloudTrail Lake 최신 기능 (쿼리 기반 분석)
- [ ] RAM 최신 공유 가능 리소스 타입 목록
- [ ] Permission Boundary와 세션 정책의 최신 평가 로직 확인

---

## 6. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "EC2/Lambda가 다른 서비스에 접근" → IAM 역할
- **유형 2**: "Cross-Account 접근" → IAM 역할 + STS AssumeRole
- **유형 3**: "계정 레벨 서비스 제한" → Organizations + SCP
- **유형 4**: "앱 사용자 인증 + AWS 리소스 접근" → Cognito User Pool + Identity Pool
- **유형 5**: "직원 다중 계정 SSO" → IAM Identity Center
- **유형 6**: "누가 변경했는가 감사" → CloudTrail
- **유형 7**: "리소스 구성 규정 준수" → AWS Config
- **유형 8**: "멀티 계정 거버넌스 자동화" → Control Tower
- **유형 9**: "다른 계정과 리소스 공유" → RAM
- **유형 10**: "정책 평가: 명시적 Deny > Allow > 기본 Deny" → IAM 정책 평가 로직

---

## 7. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
