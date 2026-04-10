# Chapter 5-4. S3 버킷 정책 vs IAM 정책 우선순위 / 명시적 Deny

## 0. 한 줄 요약

🔑 **"명시적 Deny > 모든 Allow" — S3 접근은 IAM 정책 + 버킷 정책 + ACL의 합집합이지만, 어디서든 Deny가 있으면 무조건 거부**

---

## 1. S3 접근 제어 레이어

```
요청 도착
  │
  ├─ 1. Block Public Access (계정/버킷 레벨)
  │     └─ 활성화 시 → 퍼블릭 접근 무조건 차단
  │
  ├─ 2. Bucket Policy (리소스 기반)
  │     └─ 버킷에 연결된 JSON 정책
  │
  ├─ 3. IAM Policy (자격 증명 기반)
  │     └─ 사용자/역할에 연결된 정책
  │
  ├─ 4. ACL (레거시)
  │     └─ 버킷/객체 레벨 접근 목록
  │
  └─ 5. VPC Endpoint Policy (해당 시)
        └─ VPC Endpoint 경유 시 추가 제어
```

---

## 2. 정책 평가 로직

### 같은 계정 (Same Account) 접근

```
요청
  │
  ├─ 명시적 Deny 있는가? (어떤 정책에서든)
  │   └─ Yes → ❌ 거부
  │
  ├─ IAM Policy Allow OR Bucket Policy Allow?
  │   └─ 둘 중 하나라도 Allow → ✅ 허용
  │
  └─ 어디에도 Allow 없음
      └─ ❌ 묵시적 거부
```

**핵심**: 같은 계정 = IAM + Bucket Policy **합집합** (OR)

### Cross-Account 접근

```
요청 (Account A → Account B S3)
  │
  ├─ 명시적 Deny 있는가?
  │   └─ Yes → ❌ 거부
  │
  ├─ Bucket Policy Allow AND IAM Policy Allow?
  │   └─ **둘 다** Allow → ✅ 허용
  │
  └─ 하나라도 Allow 없음
      └─ ❌ 거부
```

**핵심**: Cross-Account = IAM + Bucket Policy **교집합** (AND)  
💡 Cross-Account는 **양쪽 모두 허용** 필요

---

## 3. 명시적 Deny의 위력

### 예시: IAM Allow + Bucket Deny

```json
// IAM Policy (Allow)
{"Effect": "Allow", "Action": "s3:*", "Resource": "arn:aws:s3:::my-bucket/*"}

// Bucket Policy (Deny)
{"Effect": "Deny", "Action": "s3:DeleteObject", "Resource": "arn:aws:s3:::my-bucket/*"}
```

**결과**: s3:DeleteObject → **거부** (명시적 Deny > Allow)  
**결과**: s3:GetObject → **허용** (Deny가 없으므로)

### 예시: Bucket Allow + IAM 없음 (같은 계정)

```json
// Bucket Policy (Allow)
{"Effect": "Allow", "Principal": {"AWS": "arn:aws:iam::111:user/dev"},
 "Action": "s3:GetObject", "Resource": "arn:aws:s3:::my-bucket/*"}

// IAM Policy: 없음
```

**결과**: s3:GetObject → **허용** (같은 계정: 합집합, Bucket Policy Allow만으로 충분)

---

## 4. Block Public Access

| 설정 | 효과 |
|------|------|
| **BlockPublicAcls** | 퍼블릭 ACL 설정 차단 |
| **IgnorePublicAcls** | 기존 퍼블릭 ACL 무시 |
| **BlockPublicPolicy** | 퍼블릭 버킷 정책 설정 차단 |
| **RestrictPublicBuckets** | 퍼블릭 버킷 정책의 접근 제한 |

💡 **Block All Public Access**: 4개 모두 활성화 → 퍼블릭 접근 완전 차단  
💡 Block Public Access는 **ACL/정책보다 우선** (Override)

---

## 5. 시험 빈출 시나리오

### 시나리오 1: IAM 허용인데 접근 거부

> "IAM 정책에서 S3 접근을 허용했는데 403 Forbidden."

**확인 순서:**
1. **Bucket Policy에 명시적 Deny?** → 제거
2. **Block Public Access?** → 확인
3. **SCP (Organizations)?** → Deny 확인
4. **VPC Endpoint Policy?** → 허용 확인
5. **KMS 키 정책?** → 암호화 객체인 경우

### 시나리오 2: 퍼블릭 접근 차단

> "S3 버킷의 퍼블릭 접근을 확실히 차단."

**정답**: **S3 Block Public Access** (계정 레벨 + 버킷 레벨)  
**추가**: Bucket Policy에서 퍼블릭 접근 Deny  
**오답 함정**: ACL만 수정 (Block Public Access가 더 강력)

### 시나리오 3: 특정 IP만 S3 접근 허용

> "특정 IP 대역에서만 S3 버킷에 접근 가능하도록."

**정답**: Bucket Policy + **Condition** (aws:SourceIp)
```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::bucket/*",
  "Condition": {
    "NotIpAddress": {"aws:SourceIp": "203.0.113.0/24"}
  }
}
```
💡 **Deny + NotIpAddress** = 해당 IP가 아니면 거부 (화이트리스트)

### 시나리오 4: VPC 내에서만 S3 접근

> "VPC 내부 리소스만 S3에 접근 가능하도록."

**정답**: **VPC Endpoint (Gateway)** + Bucket Policy Condition
```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::bucket/*",
  "Condition": {
    "StringNotEquals": {"aws:sourceVpce": "vpce-xxxxx"}
  }
}
```

### 시나리오 5: Cross-Account S3 접근

> "Account A가 Account B의 S3에 접근."

**정답**: Bucket Policy (Account A 허용) **AND** Account A IAM Policy (s3 허용)  
💡 Cross-Account = 양쪽 모두 허용 필요

### 시나리오 6: S3 객체 소유권

> "Cross-Account 업로드 시 버킷 소유자가 객체에 접근 못 함."

**정답**: **Bucket owner enforced** (S3 Object Ownership 설정)  
**대안**: 업로드 시 `--acl bucket-owner-full-control` 강제  
💡 2023년 이후 기본값: Bucket owner enforced (ACL 비활성화)

---

## 6. 정책 우선순위 요약 표

| 순위 | 정책 | 설명 |
|------|------|------|
| **1** | **명시적 Deny** | 어디서든 Deny → 무조건 거부 |
| **2** | **SCP** | Organizations 계정 최대 권한 |
| **3** | **Block Public Access** | 퍼블릭 접근 Override |
| **4** | **Bucket Policy / IAM Policy** | 같은 계정: 합집합, Cross-Account: 교집합 |
| **5** | **ACL** | 레거시 (Bucket owner enforced면 무시) |
| **6** | **묵시적 거부** | Allow가 없으면 기본 거부 |

---

## 7. 헷갈리는 포인트 / 함정

### 함정 1: 같은 계정 vs Cross-Account 평가 차이

- **같은 계정**: IAM Allow **또는** Bucket Allow → 허용
- **Cross-Account**: IAM Allow **그리고** Bucket Allow → 허용
- 💡 Cross-Account 403 → 양쪽 정책 모두 확인

### 함정 2: Deny는 어디에 있어도 우선

- IAM Deny + Bucket Allow → **거부**
- Bucket Deny + IAM Allow → **거부**
- SCP Deny + IAM Allow + Bucket Allow → **거부**

### 함정 3: S3 ACL은 레거시

- 2023년 이후 기본: **Bucket owner enforced** (ACL 비활성화)
- 시험에서 ACL이 보이면 → 대부분 오답 or 레거시 시나리오
- 💡 현대적 접근 제어 = **Bucket Policy + IAM Policy**

### 함정 4: VPC Endpoint와 S3 접근

- **Gateway Endpoint** (S3, DynamoDB): 무료, 라우트 테이블에 추가
- Bucket Policy에서 VPC Endpoint 조건 걸면 → **인터넷 경유 접근 차단 가능**

---

## 8. 검증 필요 항목 ⚠️

- [ ] S3 Object Ownership 기본값 최신 확인 (Bucket owner enforced)
- [ ] S3 Block Public Access 계정 레벨 기본 활성화 여부
- [ ] Cross-Account S3 복제 시 소유권 변경 설정
- [ ] VPC Gateway Endpoint vs Interface Endpoint 최신 S3 지원

---

## 9. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "명시적 Deny > Allow" 정책 평가
- **유형 2**: "S3 403 트러블슈팅" → 정책 확인 순서
- **유형 3**: "퍼블릭 접근 차단" → Block Public Access
- **유형 4**: "Cross-Account S3" → 양쪽 정책 필요
- **유형 5**: "특정 IP/VPC만 S3 접근" → Condition

---

## 10. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
