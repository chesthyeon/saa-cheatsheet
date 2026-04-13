# Chapter 0-5. 비용 최적화 (Cost Optimization)

## 0. 한 줄 요약

🔑 **"비용 최적화 = 필요한 만큼만 쓰고(Right-Sizing), 미리 약속하면 싸고(RI/SP), 안 쓸 때 끄고(스케줄링), 싼 계층으로 내리고(Lifecycle), 데이터 이동을 줄이는 것"**

---

## 1. 핵심 설계 원칙 (5가지)

| # | 원칙 | 설명 | AWS 서비스 예시 |
|---|------|------|---------------|
| 1 | **Implement cloud financial management** | 비용 가시성·책임 조직 운영 | Cost Explorer, Budgets, CUR |
| 2 | **Adopt a consumption model** | 사용한 만큼만 과금 | Lambda, Fargate, S3 |
| 3 | **Measure overall efficiency** | 비즈니스 산출물 대비 비용 추적 | CloudWatch + Cost Explorer |
| 4 | **Stop spending money on undifferentiated heavy lifting** | 관리형/서버리스 서비스 활용 | RDS, Aurora, DynamoDB |
| 5 | **Analyze and attribute expenditure** | 태그 기반 비용 배분 | Cost Allocation Tags, Organizations |

---

## 2. 키워드 → 서비스 매핑

| 시험 키워드 | 정답 서비스 |
|------------|-----------|
| "가장 비용 효율적" / "most cost-effective" | **RI/Savings Plans** / **Spot** / **서버리스** |
| "예측 가능한 워크로드 + 비용 절감" | **Reserved Instances** / **Savings Plans** |
| "중단 허용 + 최대 할인" | **Spot Instances** (최대 90% 할인) |
| "사용하지 않는 리소스 식별" | **Trusted Advisor** / **Cost Explorer** |
| "인스턴스 크기 최적화" | **Compute Optimizer** / **Cost Explorer Right-Sizing** |
| "S3 비용 절감" | **S3 Lifecycle Policy** / **Intelligent-Tiering** |
| "데이터 전송 비용" | **VPC Endpoint** / **CloudFront** / **같은 AZ 배치** |
| "비용 알림" | **AWS Budgets** (예산 초과 알림) |
| "비용 분석" | **Cost Explorer** |
| "상세 비용 보고서" | **CUR (Cost and Usage Report)** |
| "태그 기반 비용 배분" | **Cost Allocation Tags** |
| "멀티 계정 비용 통합" | **Organizations Consolidated Billing** |
| "유휴 리소스" / "underutilized" | **Trusted Advisor** + **Compute Optimizer** |

---

## 3. 5대 비용 최적화 축

### 축 1: 적정 크기 (Right-Sizing)

| 도구 | 기능 |
|------|------|
| **Compute Optimizer** | EC2/ASG/Lambda/EBS 크기 추천 (ML 기반) |
| **Cost Explorer Right-Sizing** | EC2 인스턴스 다운사이징 추천 |
| **Trusted Advisor** | 유휴 EC2, 유휴 RDS, 미사용 EBS/EIP 식별 |

💡 "**인스턴스 크기 줄이기**" / "**underutilized**" → **Compute Optimizer**

### 축 2: 예약 / 약정 (Commitment)

| 유형 | 할인율 | 기간 | 유연성 |
|------|-------|------|--------|
| **Reserved Instances (RI)** | ~72% | 1년/3년 | Standard(변경 제한) / Convertible(변경 가능) |
| **Savings Plans** | ~72% | 1년/3년 | **Compute SP**: EC2/Fargate/Lambda 모두 적용 |
| **EC2 Instance SP** | 최대 할인 | 1년/3년 | 인스턴스 패밀리+리전 고정 |

| RI 종류 | 변경 | 할인율 |
|---------|------|--------|
| **Standard RI** | 인스턴스 크기만 변경 | **최대** |
| **Convertible RI** | 패밀리·OS·테넌시 변경 가능 | 중간 |

💡 "**가장 큰 할인**" → **Standard RI / EC2 Instance SP (All Upfront, 3년)**  
💡 "**유연성 + 할인**" → **Compute Savings Plans** (EC2/Fargate/Lambda 통합)  
💡 "**인스턴스 타입 변경 가능 + 할인**" → **Convertible RI** 또는 **Compute SP**

### 축 3: Spot Instances

| 항목 | 설명 |
|------|------|
| **할인율** | 최대 **90%** |
| **중단** | AWS가 2분 전 알림 후 회수 가능 |
| **적합** | 배치 처리, CI/CD, HPC, 빅데이터, 무상태 웹 |
| **부적합** | DB, 상태 유지 서버, 장시간 단일 작업 |

| Spot 전략 | 설명 |
|-----------|------|
| **Spot Fleet** | 여러 인스턴스 풀에서 최적 조합 |
| **ASG Mixed Instances** | On-Demand + Spot 혼합 (베이스라인 On-Demand, 추가분 Spot) |
| **Spot Block** | (종료됨, 시험에 나올 수 있음) |

💡 "**중단 허용 + 최대 비용 절감**" → **Spot**  
💡 "**베이스라인 보장 + 추가 Spot**" → **ASG Mixed Instances Policy**

### 축 4: 스토리지 비용 최적화

| 전략 | 서비스 | 설명 |
|------|--------|------|
| **Lifecycle Policy** | S3 | Standard → IA → Glacier → Deep Archive 자동 전환 |
| **Intelligent-Tiering** | S3 | **자동** 접근 패턴 분석 → 계층 이동 (관리비 $0.0025/1K 객체) |
| **gp3 전환** | EBS | gp2 → **gp3** (같은 성능, ~20% 저렴) |
| **스냅샷 관리** | EBS | 오래된 스냅샷 삭제, DLM (Data Lifecycle Manager) |
| **EFS IA** | EFS | 자주 안 쓰는 파일 자동 이동 |

#### S3 스토리지 클래스 비용 순서

```
Standard > Standard-IA > One Zone-IA > Glacier Instant > Glacier Flexible > Glacier Deep Archive
(비용 높음)                                                                      (비용 낮음)
```

💡 "**접근 패턴 예측 불가**" → **S3 Intelligent-Tiering**  
💡 "**30일 후 IA, 90일 후 Glacier**" → **Lifecycle Policy**  
💡 "**가장 저렴한 아카이브**" → **Glacier Deep Archive** (복원 12시간)

### 축 5: 데이터 전송 비용

| 방향 | 비용 |
|------|------|
| **인터넷 → AWS (Inbound)** | **무료** |
| **AWS → 인터넷 (Outbound)** | 유료 (GB당) |
| **같은 AZ 내** | **무료** (Private IP 사용 시) |
| **AZ 간** | 유료 (소액) |
| **리전 간** | 유료 (더 높음) |
| **S3 → CloudFront** | **무료** |

💡 "**데이터 전송 비용 줄이기**":
1. **VPC Endpoint** (S3/DynamoDB → NAT Gateway 비용 제거)
2. **CloudFront** (S3 Origin → CloudFront 무료)
3. **같은 AZ 배치** (Private IP 사용)
4. **Direct Connect** (대량 전송 시 인터넷보다 저렴)

---

## 4. 비용 관리 도구 비교

| 도구 | 역할 | 핵심 기능 |
|------|------|----------|
| **Cost Explorer** | **분석** | 비용 트렌드, RI/SP 추천, Right-Sizing |
| **AWS Budgets** | **알림** | 예산 초과 시 알림 (이메일/SNS), 자동 액션 |
| **CUR (Cost and Usage Report)** | **상세 보고서** | 가장 세밀한 비용 데이터 (S3로 배달, Athena 분석) |
| **Trusted Advisor** | **추천** | 유휴 리소스, 보안, 한도, 성능 체크 |
| **Compute Optimizer** | **ML 추천** | EC2/ASG/Lambda/EBS 최적 크기 |
| **Cost Allocation Tags** | **비용 배분** | 태그별 비용 그룹핑 |
| **Organizations** | **통합 청구** | 멀티 계정 볼륨 할인, RI 공유 |

💡 "**비용 분석 + 트렌드**" → Cost Explorer  
💡 "**예산 초과 알림**" → Budgets  
💡 "**가장 상세한 비용 데이터**" → CUR  
💡 "**유휴 리소스 식별**" → Trusted Advisor  
💡 "**인스턴스 크기 추천**" → Compute Optimizer

---

## 5. 시험 빈출 시나리오

### 시나리오 1: "예측 가능한 24/7 워크로드 비용 절감"

**정답**: **Reserved Instances** 또는 **Savings Plans** (1년/3년)  
**오답**: Spot (중단 위험), On-Demand (비쌈)

### 시나리오 2: "배치 처리 + 중단 허용 + 최대 절감"

**정답**: **Spot Instances**

### 시나리오 3: "인스턴스 타입 자주 변경 + 할인"

**정답**: **Compute Savings Plans** (EC2/Fargate/Lambda 통합, 타입 무관)  
또는 **Convertible RI**

### 시나리오 4: "S3 비용 절감 + 접근 패턴 모름"

**정답**: **S3 Intelligent-Tiering**

### 시나리오 5: "90일 후 아카이브 + 복원 12시간 OK"

**정답**: **S3 Lifecycle → Glacier Deep Archive**

### 시나리오 6: "NAT Gateway 비용 절감"

**정답**: **S3/DynamoDB → VPC Gateway Endpoint** (무료)  
기타 서비스 → **VPC Interface Endpoint** (NAT보다 저렴)

### 시나리오 7: "유휴 EC2 인스턴스 식별"

**정답**: **Trusted Advisor** (유휴 EC2 체크) + **Compute Optimizer**

### 시나리오 8: "멀티 계정 비용 통합 + RI 공유"

**정답**: **AWS Organizations Consolidated Billing**

### 시나리오 9: "비용 예산 초과 시 자동 알림 + EC2 중지"

**정답**: **AWS Budgets** (Budget Actions → EC2 중지)

### 시나리오 10: "gp2 EBS 비용 절감"

**정답**: **gp3로 전환** (~20% 저렴, 동일 기본 성능)

---

## 6. 함정 / 주의사항

### 함정 1: "가장 저렴한(Cheapest)" vs "가장 비용 효율적(Most cost-effective)"

- **Cheapest**: 순수 가격만 비교 (Spot, 가장 작은 인스턴스)
- **Most cost-effective**: **요구사항 충족 + 가격** (RI/SP, Right-Sizing)
- 💡 "cheapest"라도 "고가용성 유지" 조건이 있으면 Spot이 아닐 수 있음

### 함정 2: Standard RI vs Convertible RI vs Savings Plans

| | 변경 가능 | 할인 |
|--|----------|------|
| Standard RI | 크기만 | **최대** |
| Convertible RI | 패밀리/OS | 중간 |
| Compute SP | **무관** ($/hr 약정) | 중간~높음 |

💡 "유연성" 키워드 → Convertible RI 또는 **Compute SP**

### 함정 3: Spot 중단 = 2분 알림

- **2분 전 알림** → Graceful Shutdown 필요
- ASG에서 Spot 사용 시 **Mixed Instances Policy** 권장
- 💡 "DB" / "상태 유지" → Spot ❌

### 함정 4: S3 Glacier 복원 시간

| 계층 | 복원 | 비용 |
|------|------|------|
| Glacier Instant Retrieval | **밀리초** | 높음 |
| Glacier Flexible Retrieval | 1~5분 (Expedited) / 3~5시간 (Standard) / 5~12시간 (Bulk) | 중간 |
| Glacier Deep Archive | **12시간** (Standard) / 48시간 (Bulk) | **최저** |

💡 "빈번하지 않지만 즉시 필요" → **Glacier Instant Retrieval**  
💡 "연 1~2회 + 12시간 OK" → **Deep Archive**

### 함정 5: 데이터 전송 = 숨은 비용

- NAT Gateway: **처리 데이터 GB당 과금** + 시간당 과금
- 💡 S3/DynamoDB → **Gateway Endpoint** (무료)로 NAT 비용 제거
- 💡 CloudFront Origin → S3 전송 **무료**

### 함정 6: Organizations RI 공유

- Consolidated Billing → **RI/SP가 전체 계정에서 공유**
- 특정 계정에만 적용하려면 → **RI 공유 비활성화** 가능
- 💡 "특정 계정만 RI 적용" → RI Sharing 끄기

---

## 7. 검증 필요 항목 ⚠️

- [ ] Savings Plans 최신 할인율 (Compute SP vs EC2 Instance SP)
- [ ] Spot Block 완전 종료 확인 (2021년 종료 발표)
- [ ] S3 Intelligent-Tiering 최신 계층 (Archive Access Tier 자동 활성화?)
- [ ] Compute Optimizer 최신 지원 리소스 (ECS? RDS?)
- [ ] AWS Budgets Actions 최신 지원 액션 타입
- [ ] gp3 vs gp2 최신 가격 비교

---

## 8. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "비용 절감 + 예측 가능 워크로드" → RI / Savings Plans
- **유형 2**: "중단 허용 + 최대 할인" → Spot
- **유형 3**: "S3 비용 최적화" → Lifecycle / Intelligent-Tiering
- **유형 4**: "유휴 리소스 식별" → Trusted Advisor / Compute Optimizer
- **유형 5**: "데이터 전송 비용" → VPC Endpoint / CloudFront
- **유형 6**: "비용 알림" → Budgets
- **유형 7**: "멀티 계정 할인" → Organizations Consolidated Billing

---

## 9. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
