# Chapter 0-2. 보안 (Security)

## 0. 한 줄 요약

🔑 **"보안 = 최소 권한(Least Privilege) + 모든 계층 보호(Defense in Depth) + 데이터는 항상 암호화(전송 중 + 저장 시) + 사람 접근 제거(자동화)"**

---

## 1. 핵심 설계 원칙 (7가지)

| # | 원칙 | 설명 | AWS 서비스 예시 |
|---|------|------|---------------|
| 1 | **Strong identity foundation** | 최소 권한, 의무 분리 | IAM, SCP, Permission Boundary |
| 2 | **Enable traceability** | 모든 행위 로깅·감사 | CloudTrail, Config, VPC Flow Logs |
| 3 | **Apply security at all layers** | 네트워크~앱 모든 계층 보호 | SG, NACL, WAF, Shield, GuardDuty |
| 4 | **Automate security best practices** | 보안 점검 자동화 | Security Hub, Config Rules, Inspector |
| 5 | **Protect data in transit and at rest** | 전송 중(TLS) + 저장 시(KMS) 암호화 | KMS, ACM, SSE-S3/SSE-KMS |
| 6 | **Keep people away from data** | 직접 접근 제거, 자동화 | SSM Session Manager, Lambda |
| 7 | **Prepare for security events** | 사고 대응 준비 | GuardDuty, Detective, Macie |

---

## 2. 키워드 → 서비스 매핑

| 시험 키워드 | 정답 서비스 |
|------------|-----------|
| "최소 권한" / "least privilege" | **IAM Policy** (필요한 Action만 허용) |
| "계정 전체 권한 제한" / "prevent" | **SCP (Organizations)** |
| "역할의 최대 권한 범위 제한" | **Permission Boundary** |
| "Cross-Account 접근" | **IAM Role + STS AssumeRole** |
| "API 호출 감사" / "누가 무엇을" | **CloudTrail** |
| "리소스 구성 변경 추적" | **AWS Config** |
| "위협 탐지" / "악의적 IP" | **GuardDuty** |
| "민감 데이터(PII) 탐지" / "S3 스캔" | **Macie** |
| "취약점 스캔" / "CVE" | **Inspector** |
| "보안 표준 준수" / "중앙 보안 대시보드" | **Security Hub** |
| "사고 조사" / "근본 원인 분석" | **Detective** |
| "DDoS 방어" | **Shield (Standard/Advanced)** |
| "SQL Injection / XSS 차단" | **WAF** |
| "봇 차단" / "IP 기반 차단" | **WAF (IP Set / Bot Control)** |
| "SSL/TLS 인증서" | **ACM** |
| "데이터 암호화 키 관리" | **KMS** |
| "전용 HSM" / "FIPS 140-2 Level 3" | **CloudHSM** |
| "DB 자격 증명 자동 로테이션" | **Secrets Manager** |
| "설정 값 중앙 관리" | **SSM Parameter Store** |
| "VPC 내부 트래픽 로그" | **VPC Flow Logs** |
| "DNS 방화벽" | **Route 53 Resolver DNS Firewall** |

---

## 3. 5대 보안 축 상세

### 축 1: 접근 제어 (Identity & Access)

| 계층 | 도구 | 범위 |
|------|------|------|
| **AWS 계정** | SCP (Organizations) | 계정 전체 최대 권한 |
| **IAM 역할/사용자** | IAM Policy | 특정 주체 권한 |
| **역할 경계** | Permission Boundary | 역할의 최대 권한 상한 |
| **리소스** | Resource-based Policy | 리소스 자체에 누가 접근 가능한지 |
| **세션** | STS Session Policy | 임시 자격 증명 추가 제한 |

**정책 평가 순서**: 명시적 Deny → SCP → Permission Boundary → Session Policy → Identity-based/Resource-based Allow

💡 "**명시적 Deny는 모든 Allow를 이긴다**" — SAA 핵심 문장

### 축 2: 탐지 (Detection)

| 서비스 | "무엇"을 탐지 | 데이터 소스 |
|--------|------------|-----------|
| **GuardDuty** | **위협** (악의적 IP, 비정상 API 호출, 암호화폐 채굴) | CloudTrail, VPC Flow Logs, DNS Logs, S3 Data Events |
| **Inspector** | **취약점** (CVE, 네트워크 접근성) | EC2, ECR, Lambda |
| **Macie** | **민감 데이터** (PII, 신용카드 번호) | S3 객체 |
| **Security Hub** | **중앙 집계** (여러 서비스 결과 통합) | GuardDuty, Inspector, Macie, Config, Firewall Manager |
| **Detective** | **사고 조사** (근본 원인 분석) | GuardDuty Findings |
| **Config** | **구성 변경** (리소스 설정 변경 이력) | 모든 AWS 리소스 |

💡 외우기: **GuardDuty(위협) → Inspector(취약점) → Macie(데이터) → Security Hub(통합) → Detective(조사)**

### 축 3: 네트워크 보호

| 계층 | 도구 | 특징 |
|------|------|------|
| **인스턴스** | Security Group (SG) | **Stateful**, Allow만, 인스턴스 레벨 |
| **서브넷** | NACL | **Stateless**, Allow+Deny, 서브넷 레벨 |
| **VPC 경계** | VPC Endpoints (Gateway/Interface) | 인터넷 경유 없이 AWS 서비스 접근 |
| **리전/엣지** | WAF | HTTP 계층 (L7) 필터링 |
| **리전/엣지** | Shield | DDoS 방어 (L3/L4) |
| **DNS** | Route 53 Resolver DNS Firewall | 악성 도메인 차단 |
| **VPC 간** | AWS Network Firewall | 상태 기반 패킷 검사 (IPS/IDS) |
| **전체** | Firewall Manager | WAF/SG/Network Firewall 중앙 관리 |

### 축 4: 데이터 보호 (암호화)

| 구분 | 방식 | 서비스 |
|------|------|--------|
| **저장 시 (At Rest)** | SSE-S3, SSE-KMS, SSE-C, EBS 암호화, RDS 암호화 | **KMS** (키 관리 허브) |
| **전송 중 (In Transit)** | TLS/SSL, VPN, PrivateLink | **ACM** (인증서 무료 발급·갱신) |
| **키 유형** | AWS Managed Key / Customer Managed Key (CMK) / CloudHSM | **KMS** + **CloudHSM** |

| 암호화 방식 | 키 관리 | 설명 |
|------------|--------|------|
| **SSE-S3** | AWS 완전 관리 | 가장 간편 |
| **SSE-KMS** | **CMK** (감사 + 키 정책) | **CloudTrail에 키 사용 로그** |
| **SSE-C** | 고객이 키 제공 | AWS에 키 저장 안 함 |
| **CSE** (Client-Side) | 고객이 암호화·복호화 | AWS에 평문 전송 안 함 |

💡 "**키 사용 감사**" / "**키 정책 제어**" → SSE-KMS  
💡 "**자체 키 + AWS에 키 안 맡김**" → SSE-C 또는 CSE  
💡 "**FIPS 140-2 Level 3**" → CloudHSM

### 축 5: 사고 대응 (Incident Response)

| 단계 | 서비스 |
|------|--------|
| **탐지** | GuardDuty → EventBridge 자동 알림 |
| **격리** | Lambda → SG 변경 (의심 인스턴스 격리) |
| **조사** | Detective (근본 원인), CloudTrail (API 이력) |
| **복구** | 백업/스냅샷 복원, SSM Automation |
| **사후 분석** | Security Hub 대시보드, Config 타임라인 |

---

## 4. 시험 빈출 시나리오

### 시나리오 1: "S3 버킷에 민감 데이터가 있는지 스캔"

**정답**: **Macie**  
**오답**: Inspector (EC2/ECR/Lambda 취약점이지 S3 콘텐츠 아님)

### 시나리오 2: "비정상적인 API 호출 탐지 (예: root 사용, 이상 지역 접속)"

**정답**: **GuardDuty**  
**오답**: CloudTrail (로깅만, 탐지/분석은 GuardDuty)

### 시나리오 3: "여러 보안 서비스 결과를 하나의 대시보드로"

**정답**: **Security Hub**

### 시나리오 4: "EC2 인스턴스 CVE 취약점 스캔"

**정답**: **Inspector**

### 시나리오 5: "DDoS 방어 + WAF 자동 대응 + 비용 보호"

**정답**: **Shield Advanced** (WAF 자동 적용, DDoS 비용 크레딧, 24/7 DRT 지원)  
**오답**: Shield Standard (무료이지만 L3/L4만, 자동 대응·비용 보호 없음)

### 시나리오 6: "SQL Injection 차단"

**정답**: **WAF** (ALB/CloudFront/API Gateway에 연결)

### 시나리오 7: "DB 비밀번호 주기적 자동 교체"

**정답**: **Secrets Manager** (자동 로테이션 내장)  
**오답**: Parameter Store SecureString (자동 로테이션 없음, Lambda 직접 구현 필요)

### 시나리오 8: "모든 계정에 WAF 규칙 일괄 적용"

**정답**: **Firewall Manager** + Organizations

### 시나리오 9: "VPC에서 S3 접근 시 인터넷 경유 안 함"

**정답**: **S3 Gateway Endpoint** (무료, 라우팅 테이블에 추가)  
**오답**: NAT Gateway (유료, 인터넷 경유)

### 시나리오 10: "GuardDuty 위협 탐지 → 자동 격리"

**정답**: **GuardDuty → EventBridge → Lambda** (SG 변경으로 인스턴스 격리)

---

## 5. 함정 / 주의사항

### 함정 1: GuardDuty vs Inspector vs Macie 혼동

| 서비스 | 대상 | 탐지 내용 |
|--------|------|----------|
| GuardDuty | **계정 전체** | 위협 (악의적 활동) |
| Inspector | **EC2/ECR/Lambda** | 취약점 (CVE) |
| Macie | **S3** | 민감 데이터 (PII) |

💡 "**위협**=GuardDuty, **취약점**=Inspector, **데이터**=Macie"

### 함정 2: Shield Standard vs Advanced

| 기능 | Standard | Advanced |
|------|----------|----------|
| 비용 | **무료** | $3,000/월 |
| 보호 | L3/L4 | L3/L4 + **L7 (WAF 연동)** |
| DRT 지원 | ❌ | ✅ 24/7 |
| 비용 보호 | ❌ | ✅ (DDoS로 인한 스케일링 비용 환불) |
| 자동 WAF 규칙 | ❌ | ✅ |

💡 "**DDoS 비용 보호**" / "**DRT 지원**" → Shield **Advanced**

### 함정 3: Secrets Manager vs Parameter Store

| 기능 | Secrets Manager | Parameter Store |
|------|----------------|----------------|
| 자동 로테이션 | ✅ **내장** | ❌ (Lambda 직접 구현) |
| 비용 | $0.40/비밀/월 | Standard: **무료** |
| Cross-Account | ✅ | ❌ |
| 용도 | DB 비밀번호, API 키 | 설정 값, 플래그 |

💡 "**자동 로테이션**" → Secrets Manager  
💡 "**비용 최소 + 설정 관리**" → Parameter Store

### 함정 4: VPC Endpoint 종류

| 종류 | 대상 | 비용 | 원리 |
|------|------|------|------|
| **Gateway Endpoint** | **S3, DynamoDB만** | **무료** | 라우팅 테이블 |
| **Interface Endpoint** | 대부분 AWS 서비스 | **유료** (ENI 비용) | PrivateLink (ENI) |

💡 "**S3 비공개 접근 + 비용 최소**" → **Gateway Endpoint**

### 함정 5: KMS Key Policy vs IAM Policy

- KMS는 **Key Policy가 기본** (IAM Policy만으로 접근 불가, Key Policy에서 허용해야 함)
- 💡 "KMS 키 접근 = Key Policy + IAM Policy **둘 다 필요**" (Key Policy에서 IAM 허용을 활성화한 경우)

---

## 6. 검증 필요 항목 ⚠️

- [ ] GuardDuty 최신 탐지 유형 (EKS, RDS, Lambda 지원 범위)
- [ ] Inspector v2 최신 지원 범위 (Lambda 코드 스캔?)
- [ ] Security Hub 자동 교정(Auto-remediation) 최신 기능
- [ ] Shield Advanced 최신 가격 ($3,000/월 유지?)
- [ ] KMS Multi-Region Keys 최신 제약 사항
- [ ] Macie 최신 탐지 패턴 (커스텀 데이터 식별자)

---

## 7. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "위협 탐지 / 이상 활동" → GuardDuty
- **유형 2**: "취약점 / CVE / 패치" → Inspector
- **유형 3**: "PII / 민감 데이터 / S3 스캔" → Macie
- **유형 4**: "중앙 보안 대시보드" → Security Hub
- **유형 5**: "암호화 키 감사 / 키 정책" → KMS (SSE-KMS)
- **유형 6**: "DDoS + 비용 보호" → Shield Advanced
- **유형 7**: "자동 로테이션 / DB 비밀번호" → Secrets Manager
- **유형 8**: "S3 비공개 접근" → VPC Gateway Endpoint

---

## 8. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
