# Chapter 2-3. 보안·암호화 키워드 → 서비스 매핑

## 0. 한 줄 요약

🔑 **"보안 = IAM(접근) + KMS(키) + 네트워크 격리(VPC/SG/NACL) + 감사(CloudTrail/Config) + 위협 탐지(GuardDuty)" 5대 축**

---

## 1. 보안 5대 축

| 축 | 목적 | 주요 서비스 |
|----|------|-----------|
| **1. 접근 제어** | 누가 무엇을 하는가 | IAM, Cognito, STS, SCP |
| **2. 암호화** | 데이터 보호 (전송/저장) | KMS, CloudHSM, ACM, Secrets Manager |
| **3. 네트워크 격리** | 경계 보안 | VPC, SG, NACL, PrivateLink, WAF, Shield |
| **4. 감사/로깅** | 추적 가능성 | CloudTrail, Config, VPC Flow Logs |
| **5. 위협 탐지** | 실시간 탐지/대응 | GuardDuty, Security Hub, Inspector, Macie, Detective |

---

## 2. 암호화 키워드 매핑

### 저장 중 암호화 (Encryption at Rest)

| 키워드 | 정답 |
|--------|------|
| "S3 암호화" (기본) | **SSE-S3** (AWS 관리) |
| "S3 키 사용 감사" | **SSE-KMS** (CloudTrail 로그) |
| "S3 고객 키 제공" | **SSE-C** |
| "S3 업로드 전 암호화" | **Client-Side Encryption** |
| "EBS 암호화" | **KMS 키 기본/CMK** |
| "RDS 암호화" | **KMS** (생성 시에만, 이후 변경 불가) |
| "FIPS 140-2 Level 3" | **CloudHSM** |
| "키 관리" + "로테이션" | **KMS CMK** (자동 로테이션) |

### 전송 중 암호화 (Encryption in Transit)

| 키워드 | 정답 |
|--------|------|
| "HTTPS" + "ALB/CloudFront" | **ACM** (무료 인증서) |
| "ALB → EC2 HTTPS" | End-to-End TLS 또는 SSL Offloading |
| "DX 위 암호화" | **VPN over DX** 또는 **MACsec** |
| "VPC 간 암호화" | **TLS** (앱 레벨) 또는 **VPN** |

---

## 3. 비밀/인증서 관리

| 키워드 | 정답 |
|--------|------|
| "DB 비밀번호 자동 로테이션" | **Secrets Manager** |
| "앱 설정값 + 비용 최소화" | **Parameter Store** (Standard 무료) |
| "API 키 저장" | **Secrets Manager** 또는 **Parameter Store SecureString** |
| "SSL 인증서 + 자동 갱신" | **ACM** (ALB/CloudFront) |
| "프라이빗 인증서" | **ACM Private CA** |

---

## 4. 네트워크 보안 키워드

| 키워드 | 정답 |
|--------|------|
| "웹 공격 차단 (SQL Injection, XSS)" | **WAF** |
| "DDoS 보호" | **Shield Standard** (무료) / **Shield Advanced** (유료) |
| "인스턴스 방화벽" | **Security Group** |
| "서브넷 방화벽" / "IP 차단" | **NACL** |
| "프라이빗 서비스 노출" | **PrivateLink (VPC Endpoint Service)** |
| "공개 IP 없이 AWS 서비스 접근" | **VPC Endpoint** |
| "VPC 간 연결" | **VPC Peering / Transit Gateway** |
| "온프레미스 DNS 통합" | **Route 53 Resolver** |

### WAF 핵심 규칙

| 규칙 유형 | 설명 |
|----------|------|
| IP Set | IP 허용/차단 |
| Rate-based | 요청 빈도 제한 (DDoS) |
| SQL Injection | SQL 공격 탐지 |
| XSS | 스크립트 삽입 탐지 |
| Geo Match | 국가별 차단 |
| AWS Managed Rules | 사전 정의 규칙 세트 |

💡 WAF는 **ALB, CloudFront, API Gateway, AppSync**에 연결 가능  
💡 CloudFront에 연결한 WAF = **글로벌** (us-east-1)

---

## 5. 감사/규정 준수 키워드

| 키워드 | 정답 |
|--------|------|
| "API 호출 감사" | **CloudTrail** |
| "리소스 구성 변경 추적" | **AWS Config** |
| "규정 준수 자동 평가" | **Config Rules** |
| "VPC 네트워크 트래픽 감사" | **VPC Flow Logs** |
| "S3 객체 접근 감사" | **CloudTrail 데이터 이벤트** |
| "다중 계정 중앙 감사" | **CloudTrail Organization Trail** |
| "민감 데이터 탐지 (S3)" | **Macie** |
| "취약점 스캔" (EC2/ECR) | **Inspector** |
| "중앙 보안 대시보드" | **Security Hub** |
| "위협 탐지" | **GuardDuty** |
| "보안 사고 조사" | **Detective** |

### GuardDuty vs Security Hub vs Inspector

| 서비스 | 역할 | 데이터 소스 |
|--------|------|-----------|
| **GuardDuty** | **위협 탐지** (의심 행위) | VPC Flow Logs, DNS, CloudTrail |
| **Security Hub** | 보안 **통합 대시보드** | GuardDuty + Inspector + Macie + Config |
| **Inspector** | **취약점 스캔** (CVE) | EC2, ECR, Lambda |
| **Macie** | **민감 데이터** 발견 (S3) | S3 객체 (ML 기반) |
| **Detective** | 보안 사고 **조사/분석** | CloudTrail, VPC Flow, GuardDuty |

💡 "악성 IP 접근 탐지" → **GuardDuty**  
💡 "모든 보안 알림 통합" → **Security Hub**  
💡 "S3에 PII 있는지 탐지" → **Macie**  
💡 "EC2 패치 취약점" → **Inspector**

---

## 6. 접근 제어 (IAM 심화)

| 키워드 | 정답 |
|--------|------|
| "사용자 MFA 강제" | **IAM Policy + Condition (aws:MultiFactorAuthPresent)** |
| "임시 자격 증명" | **STS AssumeRole** |
| "Cross-Account 접근" | **IAM Role + AssumeRole** 또는 **Resource-based Policy** |
| "조직 전체 서비스 제한" | **SCP** (Organizations) |
| "위임 관리자 권한 제한" | **Permission Boundary** |
| "최소 권한 원칙 자동 분석" | **IAM Access Analyzer** |
| "미사용 권한 탐지" | **IAM Access Analyzer (Unused Access)** |
| "Active Directory 통합" | **IAM Identity Center** (구 SSO) |
| "웹/모바일 앱 인증" | **Cognito** (User Pool) |
| "소셜 로그인" | **Cognito User Pool** (Federation) |

---

## 7. 시험 빈출 시나리오

### 시나리오 1: "S3 객체 암호화 + 키 사용 감사"

**정답**: **SSE-KMS** + CloudTrail 데이터 이벤트  
**오답 함정**: SSE-S3 (키 사용 감사 불가)

### 시나리오 2: "RDS 비밀번호 자동 로테이션"

**정답**: **Secrets Manager** (RDS 네이티브 통합)  
**오답 함정**: Parameter Store (자동 로테이션 없음)

### 시나리오 3: "웹 앱 SQL Injection 차단"

**정답**: **WAF** on ALB + AWS Managed Rules (SQL Injection 규칙)  
**오답 함정**: Security Group (L4, 페이로드 검사 불가)

### 시나리오 4: "DDoS 대규모 공격 보호"

**정답**: **Shield Advanced** + CloudFront + Route 53  
**추가**: WAF (L7 필터) + Auto Scaling (용량 확장)  
**비용 기본**: Shield Standard (자동 적용, 무료)

### 시나리오 5: "다중 계정 중앙 보안 모니터링"

**정답**: **Security Hub + GuardDuty** (위임 관리자 패턴)  
**추가**: Config Aggregator, Organization Trail

### 시나리오 6: "S3에 PII 데이터 있는지 탐지"

**정답**: **Amazon Macie** (ML 기반 PII 탐지)  
**오답 함정**: Inspector (EC2/ECR 취약점, S3 데이터 아님)

### 시나리오 7: "EC2에 임베딩된 AWS 자격 증명 제거"

**정답**: **IAM Role** (인스턴스 프로파일)  
**오답 함정**: IAM 사용자 액세스 키 (보안 위반)

### 시나리오 8: "조직 전체 us-east-1 리전만 허용"

**정답**: **SCP** + Condition (aws:RequestedRegion)

### 시나리오 9: "VPC에서 S3 접근 시 인터넷 경유 X"

**정답**: **VPC Gateway Endpoint (S3)** + Bucket Policy로 VPCE 강제  
```json
{
  "Condition": {"StringNotEquals": {"aws:sourceVpce": "vpce-xxx"}},
  "Effect": "Deny"
}
```

---

## 8. 빠른 매핑 치트표

| 키워드 | 정답 |
|--------|------|
| "암호화 키 관리" | **KMS** |
| "FIPS 140-2 Level 3" | **CloudHSM** |
| "비밀값 자동 로테이션" | **Secrets Manager** |
| "SSL 인증서" | **ACM** |
| "웹 방화벽" | **WAF** |
| "DDoS 보호" | **Shield** |
| "API 감사" | **CloudTrail** |
| "구성 규정 준수" | **Config** |
| "위협 탐지" | **GuardDuty** |
| "민감 데이터 탐지" | **Macie** |
| "취약점 스캔" | **Inspector** |
| "중앙 보안 대시보드" | **Security Hub** |
| "SSO" | **IAM Identity Center** |
| "앱 사용자 인증" | **Cognito** |
| "Cross-Account" | **IAM Role + AssumeRole** |
| "조직 권한 제한" | **SCP** |
| "VPC에서 S3 프라이빗 접근" | **VPC Gateway Endpoint** |

---

## 9. 함정 / 주의사항

### 함정 1: KMS vs Secrets Manager vs Parameter Store

- **KMS**: **암호화 키** 관리
- **Secrets Manager**: **비밀값** (자동 로테이션)
- **Parameter Store**: **설정값** (비밀값도 가능, 무료)
- 💡 혼동 주의: KMS는 "키를 저장"이 아니라 "키를 관리"

### 함정 2: RDS 암호화는 생성 시에만

- 암호화되지 않은 RDS → 직접 암호화 불가
- **해결**: 스냅샷 생성 → 스냅샷 **복사 시 암호화** → 복원
- 💡 "기존 RDS 암호화" → 스냅샷 복사 경로

### 함정 3: Shield Standard vs Advanced

- **Standard**: 자동, 무료, L3/L4 기본 보호
- **Advanced**: 유료 (월 $3,000), L7 보호, 비용 보호, 24/7 DRT 지원
- 💡 "대규모 공격" + "비용 보호" → **Advanced**

### 함정 4: WAF는 L7 방화벽

- WAF = HTTP/HTTPS 레벨 (SQL Injection, XSS 등)
- IP/포트 차단은 NACL/SG
- 💡 "IP 차단" → NACL, "SQL Injection 차단" → WAF

---

## 10. 검증 필요 항목 ⚠️

- [ ] Secrets Manager 최신 요금 (비밀당 $0.40?)
- [ ] Macie 최신 PII 탐지 항목
- [ ] GuardDuty 최신 위협 유형
- [ ] Security Hub 통합 서비스 최신 목록
- [ ] IAM Access Analyzer 최신 기능

---

## 11. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "암호화 키 관리" → KMS vs CloudHSM
- **유형 2**: "비밀값 관리" → Secrets Manager vs Parameter Store
- **유형 3**: "웹 보안" → WAF + Shield + CloudFront
- **유형 4**: "위협 탐지/규정" → GuardDuty / Config / Macie / Inspector
- **유형 5**: "네트워크 격리" → VPC Endpoint / PrivateLink

---

## 12. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
