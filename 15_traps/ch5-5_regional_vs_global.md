# Chapter 5-5. 리전 vs 글로벌 서비스 구분

## 0. 한 줄 요약

🔑 **"대부분 서비스는 리전 범위 — IAM/Route 53/CloudFront/WAF(CloudFront용)/S3(네임스페이스)만 글로벌" — DR/멀티리전 문제에서 '이 서비스를 다른 리전에 별도 생성해야 하는가?'를 판별하는 핵심**

---

## 1. 글로벌 서비스 (리전 선택 불필요)

| 서비스 | 범위 | 설명 |
|--------|------|------|
| **IAM** | 글로벌 | 사용자/역할/정책 = 전 리전 동일 |
| **Route 53** | 글로벌 | DNS = 전 세계 |
| **CloudFront** | 글로벌 | CDN = 엣지 로케이션 |
| **WAF** (CloudFront 연결) | 글로벌 | CloudFront용 WAF = us-east-1 |
| **ACM** (CloudFront용) | us-east-1 | CloudFront 인증서는 반드시 us-east-1 |
| **S3** (버킷 이름) | 글로벌 네임스페이스 | 이름 고유, **데이터는 리전에 저장** |
| **Organizations** | 글로벌 | 계정/OU/SCP 관리 |
| **STS** | 글로벌 엔드포인트 | 리전 엔드포인트도 존재 |
| **Shield Advanced** | 글로벌 | 계정 레벨 DDoS 보호 |

---

## 2. 리전 서비스 (리전마다 별도 생성)

| 카테고리 | 서비스 | 주의사항 |
|---------|--------|---------|
| **컴퓨팅** | EC2, Lambda, ECS, EKS, Fargate | AMI는 리전 범위, 복사 필요 |
| **스토리지** | S3 (데이터), EBS, EFS, FSx | S3 데이터는 리전 내, CRR로 복제 |
| **데이터베이스** | RDS, Aurora, DynamoDB, ElastiCache | Read Replica/Global로 멀티리전 |
| **네트워크** | VPC, ALB/NLB, Direct Connect | VPC는 리전 범위, 서브넷은 AZ 범위 |
| **보안** | KMS, Secrets Manager, Config | KMS 키는 리전 범위 (멀티리전 키 존재) |
| **분석** | Athena, Redshift, Glue, EMR | 리전별 별도 |
| **모니터링** | CloudWatch, CloudTrail | 리전별 수집, Organization Trail로 통합 |
| **메시징** | SQS, SNS, EventBridge | 리전별 별도, EventBridge 글로벌 엔드포인트 존재 |

---

## 3. "거의 글로벌" — 멀티리전 기능이 있는 리전 서비스

| 서비스 | 멀티리전 기능 | 시험 키워드 |
|--------|-------------|-----------|
| **S3** | Cross-Region Replication (CRR) | "멀티리전 복제" |
| **DynamoDB** | **Global Tables** (Active-Active) | "글로벌 테이블" |
| **Aurora** | **Global Database** (~1초 복제) | "글로벌 DB" |
| **RDS** | Cross-Region Read Replica | "Cross-Region 읽기" |
| **KMS** | Multi-Region Keys | "멀티리전 키" |
| **Secrets Manager** | 멀티리전 비밀 복제 | "비밀 복제" |
| **ECR** | Cross-Region Replication | "이미지 복제" |
| **EventBridge** | Global Endpoint (자동 페일오버) | "이벤트 글로벌" |

---

## 4. AZ 범위 서비스

| 서비스 | 범위 | 주의사항 |
|--------|------|---------|
| **EC2 인스턴스** | AZ | 다른 AZ로 이동 불가 (AMI → 새 인스턴스) |
| **EBS 볼륨** | AZ | 다른 AZ에서 사용 불가 (스냅샷 → 새 볼륨) |
| **서브넷** | AZ | 하나의 AZ에만 존재 |
| **NAT Gateway** | AZ | AZ당 1개 권장 (HA) |
| **ElastiCache 노드** | AZ | Multi-AZ 클러스터로 HA |
| **RDS Primary** | AZ | Multi-AZ로 다른 AZ Standby |

---

## 5. 시험 빈출 시나리오

### 시나리오 1: DR 리전에 필요한 것

> "리전 장애 대비 DR. 무엇을 DR 리전에 별도 생성해야 하는가?"

**별도 생성 필요:**
- VPC + 서브넷 + 보안 그룹
- EC2 AMI 복사 + ASG + ALB
- RDS Read Replica 또는 Aurora Global
- KMS 키 (Multi-Region Key 또는 별도 생성)
- Lambda 함수 배포

**별도 생성 불필요 (글로벌):**
- IAM 역할/정책
- Route 53 레코드
- CloudFront 배포

### 시나리오 2: CloudFront + ACM 리전

> "CloudFront에 HTTPS 적용. ACM 인증서를 어디서 생성?"

**정답**: **us-east-1** (N. Virginia)  
💡 CloudFront = 글로벌 → ACM은 us-east-1 필수

### 시나리오 3: S3 데이터 리전

> "S3 버킷 이름은 글로벌 고유. 데이터는 어디에 저장?"

**정답**: **선택한 리전** (데이터는 리전 내 AZ에 복제)  
💡 "S3는 글로벌" = 이름만 글로벌, **데이터는 리전에 저장**

### 시나리오 4: KMS 키 멀티리전

> "여러 리전에서 같은 KMS 키로 암호화/복호화해야 한다."

**정답**: **KMS Multi-Region Key**  
**오답 함정**: 같은 키 ID를 다른 리전에서 사용 (불가)

### 시나리오 5: Lambda@Edge 리전

> "Lambda@Edge 함수를 어디에 배포?"

**정답**: **us-east-1**에 생성 → CloudFront가 엣지에 자동 복제  
💡 Lambda@Edge = us-east-1에서만 생성 가능

---

## 6. 빠른 판별표

| 문제 키워드 | 범위 | 서비스 |
|------------|------|--------|
| "다른 리전에서도 같은 IAM 역할" | 글로벌 | IAM (자동) |
| "다른 리전의 S3에 복제" | 리전 (CRR 필요) | S3 CRR |
| "EBS를 다른 AZ로 이동" | AZ | 스냅샷 → 새 볼륨 |
| "VPC를 다른 리전에" | 리전 | 새로 생성 |
| "AMI를 다른 리전에" | 리전 | AMI 복사 |
| "CloudFront 인증서 리전" | us-east-1 | ACM |
| "글로벌 DynamoDB" | 리전 (Global Tables) | DynamoDB |
| "DNS 전 세계" | 글로벌 | Route 53 |

---

## 7. 헷갈리는 포인트 / 함정

### 함정 1: S3 "글로벌" 오해

- 버킷 이름: 글로벌 고유
- **데이터 저장**: 선택한 리전 (명시적 리전 선택)
- 💡 "S3 데이터가 자동으로 다른 리전에 복제" → ❌ (CRR 설정 필요)

### 함정 2: WAF 글로벌 vs 리전

- CloudFront에 연결: **글로벌** (us-east-1에서 생성)
- ALB/API Gateway에 연결: **리전** (ALB와 같은 리전)
- 💡 같은 WAF라도 연결 대상에 따라 범위가 다름

### 함정 3: EBS → 다른 AZ 이동

1. EBS 스냅샷 생성 (S3에 저장)
2. 스냅샷에서 새 EBS 볼륨 생성 (다른 AZ 선택)
3. 새 볼륨을 인스턴스에 연결

💡 EBS는 직접 이동 불가, **스냅샷 경유** 필수

---

## 8. 검증 필요 항목 ⚠️

- [ ] KMS Multi-Region Key 최신 지원 리전
- [ ] EventBridge Global Endpoint 최신 기능
- [ ] Lambda@Edge vs CloudFront Functions 배포 리전 차이
- [ ] Aurora Global Database 최신 지원 엔진/리전

---

## 9. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "글로벌 vs 리전 서비스" 판별
- **유형 2**: "DR 리전에 필요한 리소스" 식별
- **유형 3**: "ACM/WAF 리전" → CloudFront = us-east-1
- **유형 4**: "EBS/AMI 리전 이동" → 스냅샷/복사

---

## 10. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
