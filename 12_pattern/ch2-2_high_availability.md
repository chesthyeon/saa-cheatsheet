# Chapter 2-2. 고가용성·내결함성 키워드 → 서비스 매핑

## 0. 한 줄 요약

🔑 **"고가용성 = Multi-AZ(기본) → Multi-Region(DR) → Active-Active(FT)" 3단계 — 요구 가용성 수준과 RTO/RPO에 따라 아키텍처 레벨이 결정된다**

---

## 1. 가용성 설계 3단계

| 단계 | 수준 | 아키텍처 | 가용성 목표 |
|------|------|---------|-----------|
| **Level 1: Multi-AZ** | 기본 HA | 2~3 AZ 분산 | 99.9% ~ 99.99% |
| **Level 2: Multi-Region DR** | 재해 복구 | Primary + DR 리전 | 99.99%+ |
| **Level 3: Multi-Region Active-Active** | 내결함성 | 양쪽 모두 활성 | 99.999%+ |

---

## 2. 서비스별 고가용성 옵션

### 컴퓨팅

| 서비스 | HA 방법 | 키워드 |
|--------|--------|--------|
| **EC2** | Auto Scaling Group (Multi-AZ) | "자동 복구", "장애 교체" |
| **EC2** | **EC2 Auto Recovery** | "상태 체크 실패 시 복구" |
| **Lambda** | 자동 Multi-AZ | 기본 HA |
| **ECS/Fargate** | Multi-AZ 태스크 배치 | "컨테이너 HA" |

### 로드밸런싱

| 서비스 | HA 특징 | 키워드 |
|--------|--------|--------|
| **ALB/NLB** | 다중 AZ 자동 | "Cross-Zone Load Balancing" |
| **Route 53** | 글로벌 DNS + Health Check | "DNS 페일오버" |
| **Global Accelerator** | 글로벌 Anycast IP | "리전 간 페일오버 (빠름)" |

### 데이터베이스

| 서비스 | HA 옵션 | RPO | RTO |
|--------|--------|-----|-----|
| **RDS Multi-AZ** | 동기 Standby | 0 (같은 리전) | 60~120초 |
| **RDS Multi-AZ Cluster** | 2개 Standby (읽기 가능) | 0 | **35초 미만** |
| **Aurora** | 6 copies across 3 AZ | 0 | 빠른 페일오버 |
| **Aurora Global DB** | Cross-Region 복제 | ~1초 | <1분 |
| **DynamoDB** | 자동 3 AZ 복제 | 0 | 즉시 |
| **DynamoDB Global Tables** | Active-Active 멀티리전 | ~1초 | 0 |
| **ElastiCache Multi-AZ** | Replica 자동 페일오버 | ~초 | ~분 |

### 스토리지

| 서비스 | 가용성 | 내구성 |
|--------|--------|--------|
| **S3 Standard** | 99.99% | 11 nines (3 AZ) |
| **S3 Standard-IA** | 99.9% | 11 nines (3 AZ) |
| **S3 One Zone-IA** | 99.5% | 11 nines (**1 AZ**) |
| **EBS** | AZ 범위 | 99.999% |
| **EFS Standard** | 99.99% | 여러 AZ |
| **EFS One Zone** | 99.9% | 단일 AZ |

💡 **One Zone** 클래스는 AZ 파괴 시 **데이터 손실 가능**

### 메시징

| 서비스 | HA |
|--------|-----|
| **SQS** | 자동 Multi-AZ (메시지 복제) |
| **SNS** | 자동 Multi-AZ |
| **Kinesis Data Streams** | 자동 3 AZ 복제 |

---

## 3. Route 53 라우팅 정책 (HA/페일오버)

| 정책 | 설명 | 시험 키워드 |
|------|------|-----------|
| **Simple** | 단일 레코드 | 기본 |
| **Failover** | Primary/Secondary (Health Check) | "DR 페일오버" |
| **Weighted** | 가중치 분산 (예: 80/20) | "카나리 배포", "A/B 테스트" |
| **Latency** | 가장 낮은 지연 리전 | "글로벌 사용자 성능" |
| **Geolocation** | 사용자 위치 기반 | "국가별 컨텐츠" |
| **Geoproximity** | 지리적 근접 + 편향 | "리전 선호" |
| **Multi-Value Answer** | 여러 IP 반환 + Health Check | "간단한 HA" |

💡 "Active-Passive DR" → **Failover**  
💡 "Active-Active 글로벌" → **Latency** 또는 **Geolocation**

---

## 4. Auto Scaling 깊이

### ASG 스케일링 유형

| 유형 | 설명 | 키워드 |
|------|------|-------|
| **Target Tracking** | 목표 메트릭 유지 (CPU 70%) | "가장 간단", "자동" |
| **Step Scaling** | 임계값 단계별 조정 | "세밀한 제어" |
| **Scheduled** | 예약 시간 기반 | "예측 가능한 패턴" |
| **Predictive** | ML 기반 사전 확장 | "트래픽 예측", "사전 대비" |

### ASG Health Check

- **EC2 Health Check** (기본): 인스턴스 상태
- **ELB Health Check**: 앱 레벨 (HTTP 체크)
- 💡 "앱이 죽었는데 교체 안 됨" → **ELB Health Check 활성화** 필요

### Termination Policy

- 기본: 오래된 Launch Configuration 먼저 종료
- **OldestInstance / NewestInstance / OldestLaunchTemplate**
- 💡 "가장 오래된 인스턴스 교체" → OldestInstance

---

## 5. 시험 빈출 시나리오

### 시나리오 1: "99.99% 가용성 웹 앱"

**정답**: ALB + ASG (Multi-AZ 최소 2개) + RDS Multi-AZ  
**오답 함정**: 단일 AZ (SPOF)

### 시나리오 2: "리전 장애에도 서비스"

**정답 옵션:**
- **Route 53 Failover** + Cross-Region Read Replica 승격 (DR)
- **Aurora Global Database** + Route 53 (빠른 복구)
- **DynamoDB Global Tables** + **Global Accelerator** (무중단)

### 시나리오 3: "글로벌 사용자 지연 최소화 + HA"

**정답**: **Route 53 Latency Routing** + 리전별 인프라  
**대안**: **Global Accelerator** (Anycast IP, 빠른 페일오버)

### 시나리오 4: "DB 페일오버 시간 최소화"

**정답**:
- 같은 리전: **RDS Multi-AZ Cluster** (35초 미만) 또는 **Aurora**
- 멀티리전: **Aurora Global Database**

### 시나리오 5: "파일 공유 + Multi-AZ"

**정답**: **EFS** (자동 Multi-AZ)  
**Windows**: **FSx for Windows Multi-AZ**  
**오답 함정**: EBS (단일 AZ)

### 시나리오 6: "ASG에서 비정상 앱 교체"

**정답**: ASG **ELB Health Check** 활성화 (HTTP 응답 기반)  
**오답 함정**: EC2 Health Check만 (OS는 살았지만 앱이 죽은 경우 감지 불가)

### 시나리오 7: "예측 가능한 트래픽 패턴 + 자동 확장"

**정답**: ASG **Scheduled Scaling** 또는 **Predictive Scaling**

### 시나리오 8: "무중단 DB 페일오버 (FT)"

**정답**: **DynamoDB Global Tables** (Active-Active) 또는 **Aurora Multi-Master**  
**오답 함정**: RDS Multi-AZ (60~120초 중단 = HA, FT 아님)

---

## 6. 빠른 매핑 치트표

| 키워드 | 정답 |
|--------|------|
| "고가용성" + "웹 앱" | **ALB + ASG (Multi-AZ)** |
| "자동 장애 복구" | **ASG + ELB Health Check** |
| "DB 자동 페일오버" | **RDS Multi-AZ / Aurora** |
| "글로벌 DB 무중단" | **DynamoDB Global Tables / Aurora Global** |
| "DNS 페일오버" | **Route 53 Failover + Health Check** |
| "글로벌 지연 최소" | **Route 53 Latency / Global Accelerator** |
| "리전 간 빠른 페일오버" | **Global Accelerator** |
| "파일 공유 HA" | **EFS** |
| "Stateless 세션" | **ElastiCache / DynamoDB** |
| "예측 기반 확장" | **Predictive Scaling** |
| "S3 HA" | **기본** (Standard, 99.99%) |

---

## 7. 함정 / 주의사항

### 함정 1: Multi-AZ는 HA, Active-Active는 FT

- Multi-AZ RDS: 60~120초 중단 → HA
- Aurora Multi-Master / DynamoDB Global: 무중단 → FT
- 💡 "무중단", "제로 다운타임" → Multi-AZ ❌

### 함정 2: EBS는 AZ 범위

- EBS 볼륨은 **단일 AZ**에 존재
- 다른 AZ 접근 → 스냅샷 → 복원 필요
- 💡 "파일 공유 + Multi-AZ" → EBS ❌ → **EFS**

### 함정 3: Global Accelerator vs CloudFront

- **CloudFront**: HTTP/HTTPS 캐싱 (정적 콘텐츠에 효과)
- **Global Accelerator**: 모든 TCP/UDP (비 HTTP, 게임, 실시간)
- 💡 "정적 웹" → CloudFront
- 💡 "TCP 기반 앱 + 글로벌 페일오버" → Global Accelerator

### 함정 4: One Zone 스토리지 클래스

- S3 One Zone-IA, EFS One Zone: **단일 AZ** (AZ 파괴 시 손실)
- 💡 "HA 필수" → One Zone ❌

---

## 8. 검증 필요 항목 ⚠️

- [ ] RDS Multi-AZ Cluster 페일오버 시간 최신값
- [ ] Aurora Global Database 페일오버 시간
- [ ] Route 53 Health Check 최소 간격
- [ ] Global Accelerator 최신 기능

---

## 9. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "HA 수준 판별" → Multi-AZ vs Multi-Region vs Active-Active
- **유형 2**: "DB 페일오버 최소화" → Aurora / RDS Multi-AZ Cluster
- **유형 3**: "글로벌 페일오버" → Route 53 / Global Accelerator
- **유형 4**: "스토리지 HA" → EFS / S3 Standard

---

## 10. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
