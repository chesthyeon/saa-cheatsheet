# Chapter 2-1. 비용 최적화 키워드 → 서비스 매핑

## 0. 한 줄 요약

🔑 **"비용 최적화 = 예약(RI/SP) + 스팟(Spot) + 계층화(S3 Lifecycle) + 오프-피크 축소(ASG) + 데이터 전송 최소화(CloudFront/VPC Endpoint)" 5대 축**

---

## 1. 비용 최적화 5대 원칙

| 원칙 | 핵심 | 주요 서비스 |
|------|------|-----------|
| **1. 예약 할인** | 장기 약정으로 단가↓ | Reserved Instance, Savings Plan |
| **2. 스팟 활용** | 여유 용량 저가 사용 | Spot Instance, Spot Fleet |
| **3. 계층화** | 접근 빈도별 스토리지 변경 | S3 Lifecycle, Glacier |
| **4. 오프-피크 축소** | 유휴 리소스 축소/중지 | ASG 스케줄, RDS 중지 |
| **5. 데이터 전송 최소화** | OUT 트래픽 줄이기 | CloudFront, VPC Endpoint |

---

## 2. 컴퓨팅 비용 최적화

### EC2 구매 옵션 비교

| 옵션 | 할인 | 약정 | 사용 사례 |
|------|------|------|----------|
| **On-Demand** | 0% | 없음 | 단기/예측 불가 |
| **Reserved Instance (1년)** | ~40% | 1년 | 안정적 워크로드 |
| **Reserved Instance (3년)** | ~60% | 3년 | 장기 안정 |
| **Savings Plan (Compute)** | ~66% | 1/3년 | **유연한 EC2/Fargate/Lambda** |
| **Savings Plan (EC2 Instance)** | ~72% | 1/3년 | 특정 패밀리 고정 |
| **Spot Instance** | **최대 90%** | 없음 | **중단 허용** 배치/분석 |
| **Dedicated Host** | - | 1/3년 | BYOL, 규정 준수 |

### Savings Plan vs Reserved Instance

| 기준 | Savings Plan | Reserved Instance |
|------|-------------|------------------|
| 대상 | EC2, Fargate, **Lambda** | EC2만 |
| 유연성 | **높음** (리전/패밀리 변경 OK) | 낮음 (고정) |
| 권장 | **우선 고려** | 레거시 |

💡 시험에서 "**유연한 할인**" → Savings Plan, "고정 EC2 할인" → RI

### Spot Instance 활용

| 시나리오 | 적합 | 주의 |
|---------|------|------|
| 배치 처리 (MapReduce, ETL) | ✅ | 중단 내성 필요 |
| Stateless 웹 서버 (ASG 혼합) | ✅ | ASG with Mixed Instances |
| CI/CD 빌드 | ✅ | 빌드 재시도 가능 |
| 실시간 프로덕션 | ❌ | 중단 리스크 |
| 상태 저장 DB | ❌ | 데이터 손실 |

**ASG Mixed Instances**:
- On-Demand 30% (기본 용량) + Spot 70% (확장 용량)
- 💡 "비용 + 가용성 균형" → 혼합 ASG

### 컴퓨팅 비용 플로우차트

```
"EC2 비용 최적화"
  │
  ├─ 24/7 고정 부하 (1년 이상)
  │   └─ Savings Plan 또는 Reserved Instance
  │
  ├─ 간헐적 / 단기 (수일~수주)
  │   └─ On-Demand
  │
  ├─ 배치 / 중단 허용
  │   └─ Spot Instance
  │
  └─ 간헐적 이벤트 (ms~min)
      └─ Lambda (실행 시간만 과금)
```

---

## 3. 스토리지 비용 최적화

### S3 스토리지 클래스 비용 (저렴한 순)

| 클래스 | 저장 비용 | 검색 비용 | 최소 기간 | 사용 사례 |
|--------|----------|----------|---------|---------|
| S3 Standard | 💰💰💰 | 무료 | 없음 | 자주 접근 |
| **S3 Intelligent-Tiering** | 💰💰 (자동) | 무료 | 없음 | **접근 패턴 불명** |
| S3 Standard-IA | 💰💰 | 있음 | 30일 | 드물게 접근 |
| S3 One Zone-IA | 💰 | 있음 | 30일 | 재생성 가능 |
| S3 Glacier Instant Retrieval | 💰 | 있음 | 90일 | 분기별 접근 |
| S3 Glacier Flexible Retrieval | 💰 (저가) | 있음 (분~시간) | 90일 | 아카이브 |
| **S3 Glacier Deep Archive** | **💰 최저** | 있음 (12시간) | 180일 | **장기 보관** (규정) |

### S3 Lifecycle 정책

```
업로드 (Standard)
  │ 30일
  ▼
Standard-IA
  │ 90일
  ▼
Glacier Flexible Retrieval
  │ 180일
  ▼
Glacier Deep Archive
  │ 7년
  ▼
삭제 (만료)
```

💡 "접근 패턴 불명" → **Intelligent-Tiering** (자동 이동)  
💡 "정의된 정책" → **Lifecycle Rules** (수동 설계)

### EBS 비용 최적화

| 방법 | 설명 |
|------|------|
| **gp3** (vs gp2) | 20% 저렴 + 성능 독립 설정 |
| 사용하지 않는 볼륨 삭제 | Detached 볼륨은 계속 과금 |
| 스냅샷 라이프사이클 | DLM으로 오래된 스냅샷 자동 삭제 |
| 스냅샷 아카이브 | **EBS Snapshot Archive** (75% 저렴) |

💡 "gp2 비용 절감" → **gp3로 전환** (즉시 20% 절감)

---

## 4. 데이터베이스 비용 최적화

| 전략 | 설명 | 시험 키워드 |
|------|------|-----------|
| **RDS Reserved Instance** | 1/3년 약정 (~60% 할인) | "RDS 비용 절감" |
| **Aurora Serverless v2** | 사용한 만큼 과금 (0.5~ ACU) | "변동 트래픽 DB" |
| **RDS 스토리지 Auto Scaling** | 과잉 할당 방지 | "스토리지 낭비 방지" |
| **DynamoDB Provisioned + Auto Scaling** | On-Demand 대비 저렴 | "예측 가능 DynamoDB" |
| **DynamoDB Reserved Capacity** | RCU/WCU 약정 | "DynamoDB RI" |
| **RDS 중지** (dev/test) | 최대 7일 중지 가능 | "개발 DB 비용" |
| **Read Replica 축소** | 필요한 시점만 |  |

---

## 5. 데이터 전송 비용 최적화

**데이터 전송 비용 원칙:**
- IN (인터넷 → AWS): **무료**
- OUT (AWS → 인터넷): **유료** (~$0.09/GB)
- 같은 리전 AZ 간: 유료 (~$0.01/GB)
- 같은 AZ 내 프라이빗 IP: **무료**
- CloudFront → 인터넷: 할인

### 비용 절감 방법

| 방법 | 효과 |
|------|------|
| **CloudFront** | 오리진(S3/ALB)으로 OUT 무료, 엣지 캐싱 |
| **VPC Gateway Endpoint** (S3, DynamoDB) | **무료** (NAT GW 우회) |
| **VPC Interface Endpoint** | NAT GW 비용 회피 |
| **같은 AZ 배치** | AZ 간 비용 회피 |
| **프라이빗 IP 사용** | 퍼블릭 IP 간 통신보다 저렴 |

💡 "NAT GW 비용 절감" → **VPC Gateway Endpoint (S3)** 또는 **Interface Endpoint**  
💡 "데이터 OUT 비용" → **CloudFront** (캐싱 + 할인 요금)

---

## 6. 네트워크 비용 최적화

| 서비스 | 비용 함정 | 해결책 |
|--------|----------|--------|
| **NAT Gateway** | 시간당 + 데이터 처리 (비쌈) | VPC Endpoint 사용 |
| **Elastic IP** | Unattached 시 과금 | 사용 안 하면 릴리스 |
| **로드밸런서** | 시간당 + LCU | 불필요한 ALB 정리 |
| **VPN** | 시간당 + 데이터 | 필요 시에만 |

---

## 7. 비용 분석/관리 도구

| 도구 | 역할 | 시험 키워드 |
|------|------|-----------|
| **AWS Cost Explorer** | 비용 시각화, 예측 | "비용 분석", "예측" |
| **AWS Budgets** | 예산 알림, 임계값 | "예산 초과 알림" |
| **Cost and Usage Report (CUR)** | 상세 청구 데이터 → S3 | "상세 청구" |
| **Trusted Advisor** | 유휴 리소스 탐지 | "비용 권장사항" |
| **Compute Optimizer** | 인스턴스 크기 추천 | "적정 크기" |
| **S3 Storage Lens** | S3 사용량 분석 | "S3 비용 분석" |

---

## 8. 시험 빈출 시나리오

### 시나리오 1: "24/7 프로덕션 EC2 비용 절감"

**정답**: **Savings Plan** 또는 **Reserved Instance** (1/3년)  
**오답 함정**: Spot (중단 위험), Lambda (대량 24/7은 비쌈)

### 시나리오 2: "접근 패턴 예측 불가 S3 객체"

**정답**: **S3 Intelligent-Tiering** (자동 계층 이동)  
**오답 함정**: Lifecycle Rules (수동 설계 필요)

### 시나리오 3: "장기 규정 준수 보관 (7년)"

**정답**: **S3 Glacier Deep Archive**  
**오답 함정**: S3 Standard (과도한 비용)

### 시나리오 4: "NAT Gateway 비용 절감"

**정답**: **VPC Gateway Endpoint** (S3/DynamoDB 트래픽을 NAT 우회)  
**대안**: VPC Interface Endpoint (다른 AWS 서비스)

### 시나리오 5: "내결함성 배치 처리 + 비용 최소"

**정답**: **EC2 Spot Instance** + Auto Scaling (체크포인트)  
**대안**: **AWS Batch + Spot**

### 시나리오 6: "개발/테스트 DB 비용"

**정답**: **RDS 중지** (최대 7일) + **작은 인스턴스** + **Aurora Serverless**

### 시나리오 7: "gp2 EBS 비용 절감"

**정답**: **gp3로 볼륨 타입 변경** (즉시 20% 절감 + 성능 독립)

### 시나리오 8: "사용하지 않는 리소스 탐지"

**정답**: **Trusted Advisor** (유휴 탐지) + **Compute Optimizer** (적정 크기)

---

## 9. 빠른 매핑 치트표

| 키워드 | 정답 |
|--------|------|
| "장기 고정 EC2 비용" | **Savings Plan / RI** |
| "배치 + 저비용" | **Spot Instance** |
| "S3 접근 패턴 모름" | **Intelligent-Tiering** |
| "S3 장기 아카이브" | **Glacier Deep Archive** |
| "NAT 비용 절감" | **VPC Endpoint** |
| "데이터 OUT 비용" | **CloudFront** |
| "비용 시각화" | **Cost Explorer** |
| "예산 알림" | **AWS Budgets** |
| "유휴 리소스 탐지" | **Trusted Advisor** |
| "인스턴스 적정 크기" | **Compute Optimizer** |
| "변동 DB 트래픽" | **Aurora Serverless** |
| "간헐적 컴퓨팅" | **Lambda** (유휴 비용 없음) |

---

## 10. 함정 / 주의사항

### 함정 1: Lambda가 항상 저렴하지 않다

- 소량 호출: 거의 무료
- 대량 24/7: EC2 RI보다 비쌈 (분기점 있음)
- 💡 "24/7 고부하" → Lambda ❌

### 함정 2: Spot은 프로덕션 DB에 부적합

- Spot은 **2분 전 중단 통지**만 제공
- 상태 저장 서비스에 사용 ❌
- 💡 "프로덕션 DB + 비용" → Spot ❌ → RDS RI

### 함정 3: Cost Explorer ≠ Budgets

- **Cost Explorer**: 과거 분석, 예측
- **Budgets**: 예산 설정, 임계값 알림
- 💡 "알림" → Budgets, "분석" → Cost Explorer

---

## 11. 검증 필요 항목 ⚠️

- [ ] Savings Plan vs RI 최신 할인율
- [ ] S3 Glacier 클래스 최신 요금
- [ ] gp3 vs gp2 성능/비용 최신 비교
- [ ] VPC Endpoint 최신 요금

---

## 12. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "EC2 비용 절감" → Savings Plan / RI / Spot
- **유형 2**: "S3 비용 최적화" → 클래스 선택 / Lifecycle / Intelligent-Tiering
- **유형 3**: "네트워크 비용 절감" → VPC Endpoint / CloudFront
- **유형 4**: "비용 관리 도구" → Cost Explorer / Budgets / Trusted Advisor

---

## 13. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
