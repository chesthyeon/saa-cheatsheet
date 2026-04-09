# Chapter 4-2. 3-Tier 웹 애플리케이션 (ALB + ASG + Multi-AZ RDS)

## 0. 한 줄 요약

🔑 **"3-Tier = 프레젠테이션(ALB) + 애플리케이션(ASG) + 데이터(Multi-AZ RDS)" — 고가용성·확장성·내결함성 시험 문제의 80%가 이 아키텍처 변형**

---

## 1. 아키텍처 다이어그램

```
사용자
  │
  ▼
Route 53 (DNS)
  │
  ▼
┌──────────── ALB (Application Load Balancer) ────────────┐
│  ├─ 리스너: HTTPS (443) → ACM 인증서                      │
│  └─ 헬스 체크 → 비정상 인스턴스 자동 제거                   │
└──────────────────────────────────────────────────────────┘
        │                    │
        ▼                    ▼
┌─── AZ-a ───┐      ┌─── AZ-b ───┐
│ Public SN   │      │ Public SN   │
│ (NAT GW)    │      │ (NAT GW)    │  ← 선택: NAT GW 이중화
├─────────────┤      ├─────────────┤
│ Private SN  │      │ Private SN  │
│ EC2 (ASG)   │      │ EC2 (ASG)   │  ← Auto Scaling Group
├─────────────┤      ├─────────────┤
│ Private SN  │      │ Private SN  │
│ RDS Primary │ ←──→ │ RDS Standby │  ← Multi-AZ (동기 복제)
└─────────────┘      └─────────────┘
```

---

## 2. 3개 계층 상세

### Tier 1: 프레젠테이션 (ALB)

| 구성 요소 | 역할 | 시험 키워드 |
|----------|------|-----------|
| **ALB** | HTTP/HTTPS 트래픽 분산 (L7) | "로드밸런싱", "경로 기반 라우팅" |
| **ACM** | HTTPS 종료 (SSL Offloading) | "HTTPS", "SSL 종료" |
| **WAF** | 웹 방화벽 (SQL Injection, XSS 차단) | "보안", "공격 차단" |
| **Shield** | DDoS 보호 | "DDoS", "보호" |

**핵심 설정:**
- ALB는 **Public Subnet**에 배치
- **Cross-Zone Load Balancing**: 기본 활성화 (ALB)
- **Sticky Sessions**: 세션 유지 필요 시 (하지만 확장성 제한 → ElastiCache 권장)

### Tier 2: 애플리케이션 (ASG + EC2)

| 구성 요소 | 역할 | 시험 키워드 |
|----------|------|-----------|
| **ASG** | 자동 확장/축소 | "트래픽 변동", "Auto Scaling" |
| **EC2** | 애플리케이션 서버 | "웹 서버", "앱 서버" |
| **Launch Template** | EC2 인스턴스 설정 템플릿 | "AMI", "인스턴스 유형" |
| **NAT Gateway** | Private 인스턴스 → 인터넷 (아웃바운드) | "패치 다운로드", "API 호출" |

**핵심 설정:**
- EC2는 **Private Subnet**에 배치 (보안)
- ASG **다중 AZ** 설정 (최소 2개 AZ)
- **스케일링 정책**: Target Tracking (CPU 70%) 또는 Step Scaling

**ASG 스케일링 유형:**

| 유형 | 설명 | 시험 키워드 |
|------|------|-----------|
| **Target Tracking** | 목표 메트릭 유지 (CPU 70%) | "자동으로 유지", "가장 간단" |
| **Step Scaling** | 임계값별 단계적 조정 | "세밀한 제어" |
| **Scheduled** | 예약 시간에 조정 | "예측 가능한 트래픽 패턴" |
| **Predictive** | ML 기반 사전 확장 | "예측 기반", "사전 대비" |

### Tier 3: 데이터 (Multi-AZ RDS)

| 구성 요소 | 역할 | 시험 키워드 |
|----------|------|-----------|
| **RDS Multi-AZ** | DB 고가용성 (자동 페일오버) | "고가용성", "자동 장애 조치" |
| **Read Replica** | 읽기 부하 분산 | "읽기 성능", "보고서 쿼리" |
| **ElastiCache** | DB 쿼리 캐싱 | "반복 쿼리", "세션 저장" |

**핵심 설정:**
- RDS는 **Private Subnet** (데이터 계층 격리)
- **Multi-AZ**: 동기 복제, 자동 페일오버 (60~120초)
- **Read Replica**: 비동기 복제, 읽기 전용 (Cross-Region 가능)

---

## 3. VPC 네트워크 설계

```
VPC (10.0.0.0/16)
├── Public Subnet AZ-a  (10.0.1.0/24)  ← ALB, NAT GW, Bastion
├── Public Subnet AZ-b  (10.0.2.0/24)  ← ALB, NAT GW
├── Private Subnet AZ-a (10.0.11.0/24) ← EC2 (App)
├── Private Subnet AZ-b (10.0.12.0/24) ← EC2 (App)
├── Private Subnet AZ-a (10.0.21.0/24) ← RDS Primary
└── Private Subnet AZ-b (10.0.22.0/24) ← RDS Standby
```

**보안 그룹 체인:**

| SG | 인바운드 허용 | 설명 |
|----|-------------|------|
| **ALB SG** | 0.0.0.0/0 : 443 (HTTPS) | 인터넷에서 ALB로 |
| **App SG** | ALB SG : 80 | ALB에서 앱으로만 |
| **DB SG** | App SG : 3306 (MySQL) | 앱에서 DB로만 |

💡 **보안 그룹 참조** (SG Chaining): IP 대신 SG ID로 인바운드 허용 → 확장 시 자동 적용

---

## 4. 시험 빈출 변형 시나리오

### 시나리오 1: 기본 고가용성 웹앱

> "웹 애플리케이션의 고가용성과 확장성을 확보해야 한다."

**정답**: ALB + ASG (Multi-AZ) + RDS Multi-AZ  
**오답 함정**: 단일 EC2 + 단일 RDS (SPOF)

### 시나리오 2: DB 읽기 부하 분산

> "읽기 트래픽이 많아 DB 성능이 저하된다."

**정답**: **RDS Read Replica** + ALB 라우팅 또는 앱 레벨 분리  
**대안**: **ElastiCache** (반복 쿼리 캐싱)  
**오답 함정**: RDS 인스턴스 크기 업그레이드 (수직 확장, 한계 있음)

### 시나리오 3: 세션 관리

> "ASG로 확장 시 사용자 세션이 유실된다."

**정답 옵션:**
1. **ElastiCache (Redis)** — 세션 외부화 (가장 권장)
2. **DynamoDB** — 세션 테이블
3. ALB **Sticky Sessions** — 간단하지만 확장성 제한

**오답 함정**: 로컬 디스크 세션 (인스턴스 종료 시 유실)

### 시나리오 4: HTTPS 종료 위치

> "EC2 인스턴스의 CPU 부하를 줄이면서 HTTPS를 제공해야 한다."

**정답**: **ALB에서 SSL 종료** (SSL Offloading) + ALB → EC2는 HTTP  
**오답 함정**: 각 EC2에 SSL 인증서 설치 (관리 부담, CPU 낭비)

### 시나리오 5: 예측 가능한 트래픽 패턴

> "매일 오전 9시에 트래픽이 급증한다."

**정답**: ASG **Scheduled Scaling** (예약) + **Predictive Scaling** (ML 예측)  
**오답 함정**: Target Tracking만 (반응형 = 트래픽 도착 후 확장 시작)

### 시나리오 6: 비용 최적화

> "3-Tier 아키텍처의 비용을 줄여야 한다."

**정답 조합:**
- EC2: **Reserved Instances** (기본 부하) + **Spot Instances** (ASG 혼합)
- RDS: **Reserved Instance**
- NAT Gateway: **단일 AZ** (비용↓ but 가용성↓)
- ElastiCache: Reserved Nodes

### 시나리오 7: DB 장애 복구

> "RDS 프라이머리가 장애 발생. 자동으로 복구되어야 한다."

**정답**: **Multi-AZ** (자동 페일오버, DNS 엔드포인트 불변)  
**오답 함정**: Read Replica (자동 페일오버 아님, 수동 승격)  
**오답 함정**: 스냅샷 복원 (시간 소요, RTO 높음)

### 시나리오 8: Cross-Region DR

> "리전 장애 시에도 서비스가 가능해야 한다."

**정답**: 
- Route 53 **Failover Routing** (Active-Passive)
- **Cross-Region Read Replica** 승격
- AMI를 다른 리전에 복사
- S3 Cross-Region Replication

---

## 5. 핵심 키워드 → 서비스 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "고가용성 웹 애플리케이션" | **ALB + ASG + Multi-AZ RDS** |
| "자동 확장" | **ASG** (Auto Scaling Group) |
| "DB 고가용성" + "자동 장애 조치" | **RDS Multi-AZ** |
| "읽기 부하 분산" | **Read Replica** 또는 **ElastiCache** |
| "세션 유실 방지" | **ElastiCache Redis** 또는 **DynamoDB** |
| "SSL 종료" + "CPU 부하 감소" | **ALB SSL Offloading** |
| "예약된 트래픽 패턴" | **ASG Scheduled Scaling** |
| "Private 인스턴스 인터넷 접근" | **NAT Gateway** |
| "DB 계층 격리" | **Private Subnet + DB SG** |

---

## 6. 헷갈리는 포인트 / 함정

### 함정 1: Multi-AZ vs Read Replica

| 기준 | Multi-AZ | Read Replica |
|------|---------|-------------|
| 목적 | **고가용성** (자동 페일오버) | **읽기 성능** (부하 분산) |
| 복제 | **동기** | **비동기** |
| 읽기 가능 | ❌ Standby는 읽기 불가 | ✅ 읽기 전용 |
| 페일오버 | ✅ **자동** (60~120초) | ❌ **수동 승격** |
| Cross-Region | ❌ (같은 리전) | ✅ |

💡 "고가용성" → Multi-AZ, "읽기 성능" → Read Replica, "둘 다" → **둘 다 사용**

### 함정 2: NAT Gateway 이중화 vs 비용

- NAT GW: AZ당 1개 권장 (고가용성)
- 비용: ~$32/월/개 + 데이터 처리 비용
- 💡 "비용 최적화" → 단일 NAT GW (단, 해당 AZ 장애 시 아웃바운드 불가)
- 💡 "고가용성" → AZ마다 NAT GW (비용 증가)

### 함정 3: ASG Launch Template vs Launch Configuration

- **Launch Configuration**: 레거시 (수정 불가, 새로 생성 필요)
- **Launch Template**: 현재 권장 (버전 관리, 혼합 인스턴스 지원)
- 💡 시험에서는 **Launch Template**이 정답

### 함정 4: ALB vs NLB 선택

- 3-Tier **웹** 앱 → **ALB** (HTTP/HTTPS, L7)
- TCP/UDP, 초저지연 → **NLB** (L4)
- 💡 "경로 기반 라우팅", "호스트 기반 라우팅" → ALB

---

## 7. 검증 필요 항목 ⚠️

- [ ] RDS Multi-AZ 페일오버 시간 최신값 (60~120초?)
- [ ] ASG Predictive Scaling 지원 메트릭 범위
- [ ] NAT Gateway 최신 요금
- [ ] ALB 최신 기능 (가중치 기반 타겟 그룹 등)
- [ ] RDS Multi-AZ Cluster (2 standby) 최신 지원 엔진

---

## 8. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "고가용성 웹 애플리케이션 설계" → ALB + ASG + Multi-AZ RDS
- **유형 2**: "DB 읽기 부하 분산" → Read Replica / ElastiCache
- **유형 3**: "세션 관리" → ElastiCache / DynamoDB
- **유형 4**: "비용 최적화" → Reserved + Spot 혼합
- **유형 5**: "보안 설계" → SG Chaining + Private Subnet

---

## 9. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
