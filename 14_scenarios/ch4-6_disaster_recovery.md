# Chapter 4-6. 재해 복구 (DR) 4대 전략

## 0. 한 줄 요약

🔑 **"RTO/RPO가 짧을수록 비용↑" — Backup & Restore(시간) → Pilot Light(분) → Warm Standby(분~초) → Multi-Site Active-Active(초) 순으로 4단계를 이해하면 DR 문제는 풀린다**

---

## 1. RTO / RPO 정의

| 용어 | 정의 | 질문 |
|------|------|------|
| **RPO** (Recovery Point Objective) | 데이터 손실 허용 범위 | "얼마나 오래된 데이터까지 잃어도 되는가?" |
| **RTO** (Recovery Time Objective) | 복구 시간 허용 범위 | "얼마나 빨리 서비스를 재개해야 하는가?" |

```
시간 흐름 →
──────────[마지막 백업]────────────[장애 발생]────────────[복구 완료]──
          │←───── RPO ─────→│         │←───── RTO ─────→│
          (데이터 손실 구간)            (서비스 중단 구간)
```

---

## 2. 4대 DR 전략 비교

| 전략 | RTO | RPO | 비용 | 핵심 | 시험 키워드 |
|------|-----|-----|------|------|-----------|
| **Backup & Restore** | **시간** (수시간) | **시간** | 💰 가장 저렴 | 백업 → 장애 시 복원 | "비용 최소", "느린 복구 OK" |
| **Pilot Light** | **10분~수십분** | **분** | 💰💰 | 핵심 구성만 최소 실행 | "최소 핵심만 실행", "빠른 시작" |
| **Warm Standby** | **분** | **초~분** | 💰💰💰 | 축소 버전 상시 가동 | "축소 실행", "빠른 확장" |
| **Multi-Site Active-Active** | **초** (거의 0) | **초** (거의 0) | 💰💰💰💰 최고 | 양쪽 모두 활성 | "무중단", "제로 다운타임" |

---

## 3. 전략별 상세 아키텍처

### 전략 1: Backup & Restore

```
[Primary Region]                    [DR Region]
  EC2 + RDS                           (없음)
     │
     ├─ S3 Cross-Region Replication ──→ S3 (백업)
     ├─ RDS 자동 스냅샷 → 복사 ───────→ RDS 스냅샷
     ├─ EBS 스냅샷 → 복사 ───────────→ EBS 스냅샷
     └─ AMI 복사 ────────────────────→ AMI

[장애 발생 시]
  DR Region에서 스냅샷/AMI로 인프라 재생성 (수시간)
```

| 구성 요소 | 설명 |
|----------|------|
| S3 CRR | 데이터 자동 복제 |
| RDS 스냅샷 복사 | Cross-Region 자동 복사 |
| AMI 복사 | EC2 이미지 복제 |
| CloudFormation/Terraform | 인프라 재생성 자동화 |

**장점**: 비용 최저 (저장 비용만)  
**단점**: RTO 가장 길음 (인프라 생성 시간)

### 전략 2: Pilot Light

```
[Primary Region]                    [DR Region]
  ALB + ASG(EC2) + RDS Primary        RDS Read Replica (최소)
                                      AMI 준비됨
                                      Launch Template 준비됨
                                      (EC2 = 0대, ASG 꺼짐)

[장애 발생 시]
  1. RDS Read Replica → Primary 승격
  2. ASG 활성화 (EC2 시작)
  3. Route 53 DNS 전환
  → 10분~수십분
```

| 구성 요소 | 평시 상태 | 장애 시 |
|----------|----------|--------|
| RDS | **Read Replica 실행 중** | Primary로 승격 |
| EC2/ASG | **꺼짐** (AMI만 준비) | 시작 |
| ALB | **꺼짐** | 생성 |
| Route 53 | Primary 가리킴 | DR로 전환 |

**핵심**: 데이터 계층(DB)만 최소 실행 → "불씨(Pilot Light)"

### 전략 3: Warm Standby

```
[Primary Region]                    [DR Region]
  ALB + ASG (min:4)                   ALB + ASG (min:1) ← 축소 실행
  RDS Primary                         RDS Read Replica
  Full Traffic                         소량 트래픽 가능

[장애 발생 시]
  1. ASG min/max 확장 (1 → 4)
  2. RDS Read Replica → Primary 승격
  3. Route 53 DNS 전환 (가중치 변경)
  → 수분
```

| 구성 요소 | 평시 상태 | 장애 시 |
|----------|----------|--------|
| RDS | Read Replica 실행 | Primary 승격 |
| EC2/ASG | **축소 실행** (최소 인스턴스) | **스케일 업** |
| ALB | **실행 중** | 트래픽 수용 |
| Route 53 | Primary 가리킴 | DR로 전환 |

**핵심**: 전체 인프라가 축소 버전으로 "예열(Warm)" 상태

### 전략 4: Multi-Site Active-Active

```
[Region A]                          [Region B]
  ALB + ASG (Full)                    ALB + ASG (Full)
  Aurora Global DB (Primary)          Aurora Global DB (Secondary)
  Full Traffic (50%)                  Full Traffic (50%)

Route 53 (Latency/Weighted Routing → 양쪽 분산)

[장애 발생 시]
  1. Route 53 헬스 체크 → 자동 페일오버
  2. Aurora Global DB Secondary → Primary 승격
  → 초 단위
```

| 구성 요소 | 평시 상태 | 장애 시 |
|----------|----------|--------|
| 양쪽 모두 | **Full 실행** | 한쪽이 전체 흡수 |
| Aurora Global | **양방향 복제** | 자동 승격 |
| Route 53 | **양쪽 라우팅** | 장애 쪽 제거 |

**핵심**: 양쪽 모두 프로덕션 트래픽 처리 중

---

## 4. DR 전략 판별 플로우차트

```
"DR 전략 선택"
  │
  ├─ "비용 최소" + "느린 복구 OK" (RTO: 시간)
  │   └─ Backup & Restore
  │
  ├─ "빠른 복구" + "비용 적당" (RTO: 10~30분)
  │   └─ Pilot Light
  │
  ├─ "매우 빠른 복구" + "비용 높아도 OK" (RTO: 분)
  │   └─ Warm Standby
  │
  └─ "무중단" + "비용 무관" (RTO: 초)
      └─ Multi-Site Active-Active
```

---

## 5. 시험 빈출 변형 시나리오

### 시나리오 1: 비용 최소 DR

> "가장 저렴한 DR 전략. 복구 시간 몇 시간은 허용."

**정답**: **Backup & Restore** (S3 CRR + 스냅샷 복사)  
**오답 함정**: Pilot Light (비용 더 높음)

### 시나리오 2: 핵심 DB만 보호

> "DR 비용을 줄이되, DB 복구 시간을 최소화해야 한다."

**정답**: **Pilot Light** (RDS Read Replica만 DR 리전에 유지)  
**오답 함정**: Warm Standby (EC2도 상시 실행 = 비용↑)

### 시나리오 3: 빠른 페일오버

> "장애 시 수분 내 전체 서비스 복구. 평시에 DR 리전도 소량 트래픽 처리."

**정답**: **Warm Standby**  
**오답 함정**: Pilot Light (EC2 꺼져 있어 시작 시간 필요)

### 시나리오 4: 무중단 서비스

> "글로벌 사용자 대상. 리전 장애 시에도 무중단."

**정답**: **Multi-Site Active-Active** + Route 53 Latency Routing  
**DB**: **Aurora Global Database** (1초 미만 복제 지연)  
**오답 함정**: Warm Standby (페일오버에 수분 소요)

### 시나리오 5: RPO 최소화

> "데이터 손실을 최소화해야 한다."

**정답 순서** (RPO 짧은 순):
1. **Aurora Global DB** (RPO ~1초)
2. **RDS Cross-Region Read Replica** (RPO 수초~분)
3. **S3 CRR** (RPO 분~시간)
4. **RDS 스냅샷** (RPO = 스냅샷 주기)

### 시나리오 6: Route 53 페일오버

> "Primary 리전 장애 시 자동으로 DR 리전으로 전환."

**정답**: Route 53 **Failover Routing** + **Health Check**  
**설정**: Primary Record (헬스 체크 연결) + Secondary Record (DR)

### 시나리오 7: RTO 1시간 이내, 비용 효율적

> "RTO 1시간 이내를 요구하지만 비용을 최적화해야 한다."

**정답**: **Pilot Light** (RTO 10~30분, Warm Standby보다 저렴)  
**오답 함정**: Backup & Restore (RTO 수시간 = 요구사항 미달)

### 시나리오 8: 데이터베이스별 DR 옵션

| DB | DR 옵션 | RPO |
|----|---------|-----|
| **Aurora** | Global Database | ~1초 |
| **RDS** | Cross-Region Read Replica | 수초~분 |
| **DynamoDB** | Global Tables (Active-Active) | ~1초 |
| **S3** | Cross-Region Replication | 분 |
| **EFS** | Cross-Region Replication | 분 |

---

## 6. 핵심 키워드 → 전략 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "가장 저렴한 DR" | **Backup & Restore** |
| "핵심 구성만 최소 실행" | **Pilot Light** |
| "축소 버전 상시 가동" | **Warm Standby** |
| "양쪽 모두 활성", "무중단" | **Multi-Site Active-Active** |
| "RTO 수시간 OK" | **Backup & Restore** |
| "RTO 분 이내" | **Warm Standby** 또는 **Active-Active** |
| "RPO 최소" | **Aurora Global DB** / **DynamoDB Global Tables** |
| "자동 페일오버 DNS" | **Route 53 Failover Routing** |

---

## 7. 헷갈리는 포인트 / 함정

### 함정 1: Pilot Light vs Warm Standby 구분

- **Pilot Light**: EC2 **꺼져 있음** (DB만 실행) → 장애 시 EC2 시작
- **Warm Standby**: EC2 **켜져 있음** (축소 실행) → 장애 시 스케일 업
- 💡 "최소 인스턴스가 실행 중인가?" → Yes: Warm Standby, No: Pilot Light

### 함정 2: Multi-AZ ≠ Multi-Region DR

- **Multi-AZ**: 같은 리전 내 고가용성 (AZ 장애 대응)
- **Multi-Region DR**: 리전 장애 대응 (별도 전략 필요)
- 💡 "리전 장애" → Multi-AZ로는 불가, DR 전략 필요

### 함정 3: Aurora Global DB vs RDS Read Replica

- **Aurora Global DB**: 전용 복제 인프라, RPO ~1초, 1분 내 승격
- **RDS Read Replica**: 비동기 복제, RPO 수초~분, 수동 승격
- 💡 "가장 빠른 DB 페일오버" → **Aurora Global Database**

### 함정 4: S3 CRR 지연

- S3 Cross-Region Replication은 **비동기** (수분 지연 가능)
- 새 객체는 자동 복제, 기존 객체는 **S3 Batch Replication** 필요
- 💡 "즉시 복제" 필요 → S3 CRR로는 보장 불가

---

## 8. 검증 필요 항목 ⚠️

- [ ] Aurora Global Database 페일오버 시간 최신값 (~1분?)
- [ ] RDS Cross-Region Read Replica 승격 시간
- [ ] Route 53 Health Check 최소 간격 (10초? 30초?)
- [ ] S3 Replication Time Control (RTC) SLA (15분?)
- [ ] DynamoDB Global Tables 복제 지연 최신값
- [ ] AWS Elastic Disaster Recovery (DRS) 최신 기능

---

## 9. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "DR 전략 선택" → RTO/RPO + 비용으로 4대 전략 판별
- **유형 2**: "비용 최소 DR" → Backup & Restore
- **유형 3**: "무중단 글로벌 서비스" → Active-Active
- **유형 4**: "DB 페일오버" → Aurora Global vs RDS Read Replica
- **유형 5**: "Route 53 페일오버" → Failover Routing + Health Check

---

## 10. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
