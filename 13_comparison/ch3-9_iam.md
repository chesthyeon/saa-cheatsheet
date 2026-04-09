# Chapter 3-9. IAM: 역할 vs 사용자 vs 리소스 기반 정책 vs SCP vs Permission Boundary

## 0. 한 줄 요약

🔑 **"누구에게 붙이는가(자격 증명 vs 리소스) × 어디까지 제한하는가(허용 vs 최대 경계)"로 IAM 정책 유형이 결정된다 — 명시적 Deny > 모든 Allow가 시험의 핵심 공식**

---

## 1. IAM 정책 유형 전체 비교

| 기준 | Identity-based (자격 증명 기반) | Resource-based (리소스 기반) | SCP | Permission Boundary |
|------|-------------------------------|---------------------------|-----|-------------------|
| **부착 대상** | 사용자/그룹/역할 | S3, SQS, Lambda, KMS 등 | **OU/계정** (Organizations) | 사용자/역할 |
| **질문** | "이 주체가 무엇을 할 수 있는가?" | "이 리소스에 누가 접근하는가?" | "이 계정에서 무엇이 허용되는가?" | "이 주체의 최대 권한은?" |
| **Principal 명시** | ❌ (부착 대상이 주체) | ✅ **필수** (누가 접근하는지 명시) | ❌ (계정/OU에 적용) | ❌ (부착 대상이 주체) |
| **Cross-Account** | 역할 전환 (AssumeRole) | ✅ **직접 허용** (역할 전환 불필요) | N/A | N/A |
| **허용/거부** | Allow + Deny | Allow + Deny | **Deny 목록** 또는 **Allow 목록** | **최대 허용 범위** (교집합) |
| **기본값** | 모든 것 거부 | 모든 것 거부 | 모든 것 허용 (FullAWSAccess) | 연결 시 교집합 적용 |

---

## 2. 핵심 판별 플로우차트

```
"IAM 정책 유형 선택"
  │
  ├─ 사용자/역할에게 권한 부여?
  │   └─ Identity-based Policy (IAM 정책)
  │
  ├─ 리소스에 접근 제어 설정?
  │   ├─ S3 버킷 접근 제어? ──→ S3 Bucket Policy (리소스 기반)
  │   ├─ Cross-Account 접근 (역할 전환 없이)? ──→ Resource-based Policy
  │   └─ KMS 키 접근 제어? ──→ KMS Key Policy (리소스 기반)
  │
  ├─ 계정/OU 전체 권한 제한?
  │   └─ "특정 리전만 허용", "특정 서비스 차단" ──→ SCP
  │
  └─ 사용자/역할의 최대 권한 제한?
      └─ "위임된 관리자가 과도한 권한 부여 방지" ──→ Permission Boundary
```

---

## 3. 쌍별 상세 비교

### IAM 사용자 vs IAM 역할

| 기준 | IAM 사용자 | IAM 역할 |
|------|----------|---------|
| 자격 증명 | **영구** (비밀번호, 액세스 키) | **임시** (STS 토큰) |
| 사용 주체 | 사람 (콘솔/CLI) | **서비스, 다른 계정, 페더레이션 사용자** |
| 보안 | 키 유출 위험 | ✅ 임시 토큰 (자동 만료) |
| Cross-Account | ❌ (키 공유 = 보안 위반) | ✅ **AssumeRole** |
| EC2 → S3 접근 | ❌ 액세스 키 저장 (위험) | ✅ **인스턴스 프로파일** (역할 연결) |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "EC2에서 S3 접근" | **IAM 역할** (인스턴스 프로파일) |
| "다른 계정 리소스 접근" | **IAM 역할 + STS AssumeRole** |
| "Lambda가 DynamoDB 접근" | **IAM 역할** (실행 역할) |
| "액세스 키 제거", "보안 강화" | **IAM 역할로 전환** |

### Identity-based vs Resource-based 정책

| 기준 | Identity-based | Resource-based |
|------|---------------|---------------|
| 부착 위치 | 사용자/그룹/역할 | 리소스 (S3, SQS, Lambda 등) |
| Principal | 암묵적 (정책이 붙은 주체) | **명시적** (JSON에 Principal 필드) |
| Cross-Account | AssumeRole 필요 (원래 권한 **포기**) | ✅ **직접 허용** (원래 권한 **유지**) |
| 모든 리소스 지원 | ✅ (어떤 리소스든) | ❌ (지원 리소스만: S3, SQS, KMS, Lambda 등) |

**Cross-Account 핵심 차이:**

```
시나리오: Account A의 사용자가 Account B의 S3에 접근

방법 1: IAM 역할 (AssumeRole)
  Account A 사용자 → AssumeRole(Account B 역할) → Account B 권한으로 S3 접근
  ⚠️ Account A의 원래 권한을 "내려놓음"

방법 2: Resource-based Policy (S3 Bucket Policy)
  Account B S3 버킷 정책에 Account A 사용자 허용
  ✅ Account A 원래 권한 "유지" + Account B 리소스 접근
```

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "Cross-Account" + "원래 권한 유지" | **Resource-based Policy** |
| "Cross-Account" + "역할 전환" | **IAM 역할 + AssumeRole** |
| "S3 버킷에 다른 계정 접근 허용" | **S3 Bucket Policy** (리소스 기반) |
| "Lambda를 다른 계정에서 호출" | **Lambda Resource-based Policy** |

### SCP vs IAM 정책

| 기준 | SCP | IAM 정책 |
|------|-----|---------|
| 적용 범위 | **계정/OU 전체** | 개별 사용자/그룹/역할 |
| 목적 | **최대 권한 경계** (가드레일) | 실제 권한 부여 |
| 대상 | Organizations 멤버 계정 | IAM 주체 |
| 관리 계정 영향 | ❌ **관리 계정에는 미적용** | ✅ |
| 권한 부여 | ❌ (허용 범위만 제한, 직접 부여 안 함) | ✅ 직접 부여 |

**핵심 원리:**
- SCP는 **가드레일** (울타리) = "이 안에서만 놀 수 있다"
- IAM 정책은 **실제 권한** = "이것을 할 수 있다"
- **최종 권한 = SCP ∩ IAM 정책** (교집합)

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "모든 계정에서 특정 리전만 허용" | **SCP** |
| "특정 서비스 사용 금지 (조직 전체)" | **SCP** |
| "개별 사용자 권한 부여" | **IAM 정책** |
| "관리 계정 권한 제한" | ❌ SCP 불가 → **IAM 정책** |

### Permission Boundary vs SCP

| 기준 | Permission Boundary | SCP |
|------|-------------------|-----|
| 적용 대상 | **개별 사용자/역할** | **계정/OU** |
| 목적 | 위임 관리자가 부여할 수 있는 최대 권한 제한 | 계정 전체 최대 권한 제한 |
| 사용 시나리오 | "개발자가 역할 생성 가능, 하지만 S3만" | "이 OU의 모든 계정에서 EU 리전만" |
| 필수 여부 | 선택 | Organizations 사용 시 |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "개발자가 IAM 역할 생성 가능 + 권한 제한" | **Permission Boundary** |
| "위임된 관리 + 과도한 권한 방지" | **Permission Boundary** |
| "조직 전체 서비스 제한" | **SCP** |

---

## 4. IAM 정책 평가 로직 (시험 필수)

```
요청 도착
  │
  ├─ 1. 명시적 Deny 있는가? ──→ ❌ 거부 (최우선)
  │
  ├─ 2. SCP 허용 범위 안인가? ──→ 아니면 ❌ 거부
  │
  ├─ 3. Resource-based Policy Allow? ──→ ✅ 허용 (단독 충분)
  │
  ├─ 4. Identity-based Policy Allow?
  │   ├─ Permission Boundary 범위 안? ──→ 아니면 ❌ 거부
  │   └─ 세션 정책 범위 안? ──→ 아니면 ❌ 거부
  │   └─ ✅ 허용
  │
  └─ 위 어디에도 Allow 없음 ──→ ❌ **묵시적 거부** (기본)
```

**핵심 공식:**
- **명시적 Deny > 모든 Allow** (어디서든 Deny면 거부)
- **최종 권한 = SCP ∩ Permission Boundary ∩ Identity-based Policy**
- Resource-based Policy는 자격 증명 기반과 **합집합** (독립적으로 허용 가능)

---

## 5. 시험 빈출 시나리오 + 정답

| 시나리오 | 정답 | 오답 함정 |
|----------|------|----------|
| EC2 → S3 접근 권한 | **IAM 역할** (인스턴스 프로파일) | IAM 사용자 액세스 키 (보안 위험) |
| Account A → Account B S3 접근 | **S3 Bucket Policy** 또는 **AssumeRole** | IAM 사용자 키 공유 (금지) |
| 조직 전체 us-east-1만 허용 | **SCP** (리전 제한) | IAM 정책 (개별 적용 = 비효율) |
| 개발자가 Lambda 역할 생성, S3만 허용 | **Permission Boundary** | SCP (계정 전체에 영향) |
| S3 버킷 퍼블릭 접근 차단 | **S3 Block Public Access** + **Bucket Policy** | IAM 정책만 (리소스 레벨 제어 필요) |
| IAM 정책 Allow + SCP Deny | **거부** (명시적 Deny > Allow) | 허용 (SCP Deny가 우선) |
| 임시 자격 증명으로 Cross-Account | **STS AssumeRole** | 장기 액세스 키 공유 |
| 웹 앱 사용자 → AWS 리소스 접근 | **Cognito Identity Pool** (임시 자격 증명) | IAM 사용자 생성 (확장 불가) |
| 관리 계정 권한 제한 | **IAM 정책** (SCP는 관리 계정 미적용) | SCP (관리 계정 예외) |
| Cross-Account + 원래 권한 유지 | **Resource-based Policy** | AssumeRole (원래 권한 포기) |

---

## 6. 헷갈리는 포인트 / 함정

### 함정 1: SCP는 관리 계정에 미적용

- SCP는 **멤버 계정**에만 적용
- **관리 계정** (Management Account)은 SCP 영향 없음
- 💡 "관리 계정 제한" → IAM 정책 사용

### 함정 2: SCP는 권한을 부여하지 않는다

- SCP Deny = 서비스 차단 ✅
- SCP Allow = "이 범위 안에서만 허용 **가능**" (실제 부여는 IAM 정책)
- 💡 SCP Allow 있어도 IAM 정책 없으면 → 여전히 거부

### 함정 3: Resource-based Policy의 Cross-Account 이점

- AssumeRole: 원래 권한 포기 → 역할의 권한만 사용
- Resource-based: 원래 권한 유지 + 리소스 접근 추가
- 💡 "두 계정 리소스 모두 접근 필요" → Resource-based Policy

### 함정 4: Cognito User Pool vs Identity Pool

- **User Pool**: 인증 (로그인/회원가입) → JWT 토큰 발급
- **Identity Pool**: AWS 자격 증명 발급 → S3, DynamoDB 직접 접근
- 💡 "웹 앱 로그인" → User Pool, "웹 앱에서 S3 직접 접근" → Identity Pool

---

## 7. 검증 필요 항목 ⚠️

- [ ] SCP 최대 정책 크기 (5,120 바이트?)
- [ ] Permission Boundary 지원 서비스 범위
- [ ] IAM Identity Center (구 SSO) 최신 기능
- [ ] Cognito Identity Pool v2 변경 사항
- [ ] Resource-based Policy 지원 서비스 전체 목록
- [ ] IAM Access Analyzer 최신 기능 (정책 검증, 미사용 접근)

---

## 8. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "IAM 사용자 vs 역할" → 영구 키 vs 임시 자격 증명
- **유형 2**: "Identity-based vs Resource-based" → Cross-Account 판별
- **유형 3**: "SCP vs IAM 정책" → 조직 가드레일 vs 개별 권한
- **유형 4**: "Permission Boundary" → 위임 관리자 최대 권한 제한
- **유형 5**: "정책 평가 로직" → 명시적 Deny > Allow

---

## 9. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
