# Chapter 4-8. Cross-Account 액세스 패턴 (IAM Role + STS AssumeRole)

## 0. 한 줄 요약

🔑 **"Cross-Account = IAM 역할 + STS AssumeRole이 정석" — 액세스 키 공유는 절대 금지, 리소스 기반 정책(S3/SQS/KMS)은 역할 전환 없이도 허용 가능한 대안**

---

## 1. Cross-Account 3대 패턴

| 패턴 | 방식 | 사용 사례 | 시험 빈도 |
|------|------|----------|----------|
| **IAM Role + AssumeRole** | 역할 전환 (임시 자격 증명) | **범용 (가장 일반적)** | ⭐⭐⭐ |
| **Resource-based Policy** | 리소스에 직접 허용 | S3, SQS, KMS, Lambda 등 | ⭐⭐ |
| **AWS RAM** | 리소스 공유 서비스 | Transit Gateway, 서브넷, License Manager | ⭐ |

---

## 2. 패턴 1: IAM Role + STS AssumeRole (정석)

### 아키텍처

```
Account A (요청자)                    Account B (리소스 소유자)
┌──────────────────┐                ┌──────────────────┐
│ IAM User/Role    │                │ IAM Role         │
│ (arn:aws:iam::   │  AssumeRole   │ (arn:aws:iam::   │
│  111:user/dev)   │ ──────────→   │  222:role/cross) │
│                  │  ← 임시 토큰   │                  │
│ 필요: sts:       │                │ Trust Policy:    │
│  AssumeRole 허용 │                │  Account A 허용  │
└──────────────────┘                └──────────────────┘
                                           │
                                    Account B 리소스 접근
                                    (S3, RDS, EC2 등)
```

### 설정 단계

**Account B (리소스 소유자) — 역할 생성:**

```json
// Trust Policy (누가 이 역할을 맡을 수 있는가)
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111111111111:root"  // Account A
  },
  "Action": "sts:AssumeRole"
}

// Permission Policy (이 역할이 무엇을 할 수 있는가)
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::bucket-b/*"
}
```

**Account A (요청자) — AssumeRole 허용:**

```json
{
  "Effect": "Allow",
  "Action": "sts:AssumeRole",
  "Resource": "arn:aws:iam::222222222222:role/cross-account-role"
}
```

### 핵심 포인트

| 포인트 | 설명 |
|--------|------|
| **양쪽 허용 필요** | Account A: AssumeRole 허용 + Account B: Trust Policy |
| **임시 자격 증명** | STS가 임시 토큰 발급 (기본 1시간, 최대 12시간) |
| **원래 권한 포기** | 역할 전환 시 원래 권한은 사용 불가 |
| **External ID** | 제3자 서비스 접근 시 혼동 대리인 문제 방지 |

---

## 3. 패턴 2: Resource-based Policy

### 아키텍처

```
Account A                           Account B
┌──────────────────┐                ┌──────────────────┐
│ IAM User/Role    │  직접 접근     │ S3 Bucket        │
│                  │ ──────────→   │ Bucket Policy:   │
│ (역할 전환 없음)  │                │  Account A 허용  │
│ 원래 권한 유지 ✅ │                │                  │
└──────────────────┘                └──────────────────┘
```

### S3 Bucket Policy 예시

```json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111111111111:role/app-role"
  },
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::bucket-b/*"
}
```

### Resource-based Policy 지원 서비스

| 서비스 | 정책 이름 | 주요 시나리오 |
|--------|----------|-------------|
| **S3** | Bucket Policy | Cross-Account 버킷 접근 |
| **SQS** | Queue Policy | Cross-Account 메시지 전송 |
| **SNS** | Topic Policy | Cross-Account 구독/발행 |
| **KMS** | Key Policy | Cross-Account 키 사용 |
| **Lambda** | Resource Policy | Cross-Account 함수 호출 |
| **ECR** | Repository Policy | Cross-Account 이미지 풀 |
| **Secrets Manager** | Resource Policy | Cross-Account 비밀값 접근 |

### AssumeRole vs Resource-based 핵심 차이

| 기준 | AssumeRole | Resource-based |
|------|-----------|----------------|
| 원래 권한 | ❌ **포기** | ✅ **유지** |
| 지원 범위 | **모든 서비스** | 지원 서비스만 |
| 설정 | 양쪽 (Trust + Permission) | 리소스 정책만 |
| 사용 사례 | 범용 | 특정 리소스 공유 |

💡 **"두 계정 리소스 모두 접근"** → Resource-based (원래 권한 유지)  
💡 **"모든 서비스 접근"** → AssumeRole (범용)

---

## 4. 패턴 3: AWS RAM (Resource Access Manager)

### 공유 가능 리소스

| 리소스 | 사용 사례 |
|--------|----------|
| **Transit Gateway** | 다중 계정 네트워크 허브 공유 |
| **VPC Subnet** | 다른 계정이 내 서브넷에 리소스 배치 |
| **Route 53 Resolver Rules** | DNS 규칙 공유 |
| **License Manager** | 라이선스 공유 |
| **Aurora DB Cluster** | Cross-Account DB 클론 |

### RAM vs 다른 방식

| 기준 | RAM | Resource-based | AssumeRole |
|------|-----|---------------|-----------|
| 목적 | **리소스 자체 공유** | 리소스 접근 허용 | 역할 전환 |
| 소유권 | 원래 계정 유지 | 원래 계정 유지 | N/A |
| 사용 예 | TGW, 서브넷 공유 | S3 버킷 접근 | 범용 접근 |

---

## 5. Organizations + Cross-Account

### Organizations 활용 시나리오

```
Management Account
├── OU: Security
│   └── Account: Security (GuardDuty, Config 중앙)
├── OU: Production
│   ├── Account: Prod-App
│   └── Account: Prod-DB
├── OU: Development
│   └── Account: Dev
└── OU: Shared Services
    └── Account: Shared (TGW, DNS, CI/CD)
```

### Organizations Cross-Account 기능

| 기능 | 설명 | 시험 키워드 |
|------|------|-----------|
| **SCP** | OU/계정 권한 제한 | "서비스 제한", "가드레일" |
| **통합 결제** | 모든 계정 비용 합산 | "비용 통합", "볼륨 할인" |
| **RAM 자동 공유** | Organizations 내 자동 리소스 공유 | "자동 공유" |
| **CloudTrail 조직 트레일** | 모든 계정 API 로그 중앙화 | "중앙 감사" |
| **Config 어그리게이터** | 모든 계정 규정 준수 중앙 뷰 | "중앙 규정 준수" |
| **GuardDuty 위임 관리자** | 보안 계정에서 전체 위협 감지 | "중앙 보안" |

---

## 6. 시험 빈출 변형 시나리오

### 시나리오 1: 기본 Cross-Account 접근

> "Account A의 EC2가 Account B의 S3 버킷에 접근해야 한다."

**정답 옵션:**
1. **AssumeRole**: Account B에 역할 생성 → EC2 역할이 AssumeRole
2. **S3 Bucket Policy**: Account A EC2 역할 허용

**오답 함정**: 액세스 키 공유 (보안 위반)

### 시나리오 2: Cross-Account + 원래 권한 유지

> "Account A의 Lambda가 Account A의 DynamoDB와 Account B의 S3 버킷 모두 접근해야 한다."

**정답**: Account B **S3 Bucket Policy**에서 Account A Lambda 역할 허용  
(Resource-based → 원래 권한 유지 → DynamoDB + S3 모두 접근)  
**오답 함정**: AssumeRole (원래 DynamoDB 권한 포기)

### 시나리오 3: 제3자 서비스 접근

> "외부 SaaS 업체가 우리 AWS 계정의 S3에 접근해야 한다."

**정답**: IAM Role + **External ID** (혼동 대리인 방지)  
**오답 함정**: IAM 사용자 생성 (장기 자격 증명 위험)

### 시나리오 4: 중앙 로깅

> "모든 계정의 CloudTrail 로그를 하나의 계정 S3에 저장."

**정답**: **CloudTrail Organization Trail** + 중앙 S3 **Bucket Policy**  
```json
{
  "Principal": {"Service": "cloudtrail.amazonaws.com"},
  "Condition": {"StringEquals": {"aws:SourceOrgID": "o-xxxxx"}}
}
```

### 시나리오 5: 공유 VPC 네트워크

> "여러 계정의 리소스가 같은 VPC 서브넷에 배치되어야 한다."

**정답**: **AWS RAM**으로 서브넷 공유 (VPC Sharing)  
**오답 함정**: VPC Peering (같은 서브넷은 불가, 네트워크 연결만)

### 시나리오 6: Cross-Account KMS 암호화

> "Account A의 Lambda가 Account B의 KMS 키로 암호화된 S3 객체를 읽어야 한다."

**정답**: 
1. S3 Bucket Policy: Account A 허용
2. **KMS Key Policy**: Account A 허용 (별도!)
3. Account A Lambda 역할: s3:GetObject + kms:Decrypt

💡 **S3 + KMS = 두 정책 모두 허용 필요** (자주 출제)

### 시나리오 7: Cross-Account AMI 공유

> "Dev 계정에서 만든 AMI를 Prod 계정에서 사용해야 한다."

**정답**: AMI **Launch Permission** 수정 (대상 계정 추가)  
**주의**: AMI의 EBS 스냅샷이 KMS 암호화되어 있으면 → KMS 키도 공유 필요

### 시나리오 8: 위임 관리자 패턴

> "보안 팀이 모든 계정의 GuardDuty를 관리해야 한다."

**정답**: Organizations **위임 관리자** (Security Account에 GuardDuty 위임)  
**오답 함정**: Management Account에서 직접 관리 (보안 모범 사례 위반)

---

## 7. 핵심 키워드 → 서비스 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "Cross-Account 접근" (범용) | **IAM Role + AssumeRole** |
| "Cross-Account" + "원래 권한 유지" | **Resource-based Policy** |
| "서브넷/TGW 공유" | **AWS RAM** |
| "External ID" | **제3자 접근 (혼동 대리인 방지)** |
| "액세스 키 공유" | ❌ **항상 오답** |
| "중앙 로깅/감사" | **Organization Trail + S3 Bucket Policy** |
| "다중 계정 관리" | **AWS Organizations** |
| "계정 권한 제한" | **SCP** |
| "KMS + Cross-Account" | **KMS Key Policy + IAM 정책 둘 다** |

---

## 8. 헷갈리는 포인트 / 함정

### 함정 1: AssumeRole은 원래 권한을 포기한다

- 역할 전환 = 원래 자격 증명의 권한 **사용 불가**
- Account A 리소스 + Account B 리소스 동시 접근 필요 → **Resource-based Policy**
- 💡 "두 계정 리소스 모두" → Resource-based

### 함정 2: S3 Cross-Account + KMS = 두 정책 필요

- S3 Bucket Policy 허용만으로는 불충분
- KMS 암호화 객체 → **KMS Key Policy에서도 허용** 필요
- 💡 "Access Denied" + "KMS 암호화 S3" → KMS Key Policy 확인

### 함정 3: External ID의 목적

- **혼동 대리인 문제 (Confused Deputy)**: 제3자가 다른 고객의 역할을 악용
- External ID = 공유 비밀값으로 추가 검증
- 💡 "제3자 SaaS" + "보안" → External ID 포함 Trust Policy

### 함정 4: RAM vs Peering vs PrivateLink

- **RAM**: 리소스 자체 공유 (서브넷, TGW)
- **VPC Peering**: 네트워크 연결 (양방향)
- **PrivateLink**: 서비스 단방향 노출
- 💡 "같은 서브넷에 리소스 배치" → RAM (VPC Sharing)

---

## 9. 검증 필요 항목 ⚠️

- [ ] STS AssumeRole 최대 세션 시간 (12시간? 역할에 따라?)
- [ ] RAM 최신 공유 가능 리소스 목록
- [ ] Organizations 위임 관리자 지원 서비스 최신 목록
- [ ] Cross-Account AMI 공유 + KMS 설정 절차
- [ ] IAM Identity Center Cross-Account 설정
- [ ] S3 Cross-Account 복제 설정 (소유권 변경)

---

## 10. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "Cross-Account 접근" → AssumeRole vs Resource-based
- **유형 2**: "원래 권한 유지 + Cross-Account" → Resource-based Policy
- **유형 3**: "제3자 접근 보안" → External ID
- **유형 4**: "KMS + Cross-Account" → 두 정책 모두 필요
- **유형 5**: "중앙 관리" → Organizations + 위임 관리자

---

## 11. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
