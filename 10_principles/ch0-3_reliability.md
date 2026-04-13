# Chapter 0-3. 안정성 (Reliability)

## 0. 한 줄 요약

🔑 **"안정성 = 장애는 반드시 발생한다는 전제 하에, 자동 복구(Auto Recovery) + 수평 확장(Scale Out) + 멀티 AZ/리전 분산으로 서비스를 중단 없이 유지하는 것"**

---

## 1. 핵심 설계 원칙 (5가지)

| # | 원칙 | 설명 | AWS 서비스 예시 |
|---|------|------|---------------|
| 1 | **Automatically recover from failure** | 자동 장애 감지 + 복구 | ASG, Route 53 Health Check, RDS Multi-AZ |
| 2 | **Test recovery procedures** | 복구 절차를 미리 테스트 | FIS, GameDay |
| 3 | **Scale horizontally** | 단일 대형 리소스 대신 다수 소형 리소스 | ALB + ASG, DynamoDB |
| 4 | **Stop guessing capacity** | 수요에 따라 자동 확장/축소 | ASG, Aurora Serverless, Lambda |
| 5 | **Manage change in automation** | 인프라 변경을 자동화로 관리 | CloudFormation, CodeDeploy |

---

## 2. 키워드 → 서비스 매핑

| 시험 키워드 | 정답 서비스 |
|------------|-----------|
| "고가용성" / "high availability" | **Multi-AZ** 배포 (RDS, ELB, ASG) |
| "내결함성" / "fault tolerant" | **Multi-AZ + 자동 장애 조치** (Aurora, DynamoDB Global Tables) |
| "자동 복구" / "self-healing" | **ASG** (unhealthy 인스턴스 교체) |
| "수평 확장" / "scale out" | **ASG + ALB** |
| "용량 예측 불가" / "unpredictable traffic" | **ASG** / **Lambda** / **Aurora Serverless** |
| "단일 장애 지점 제거" / "eliminate SPOF" | **Multi-AZ** + **ELB** + **ASG** |
| "데이터 내구성" / "durability" | **S3 (11 9s)** / **EBS 스냅샷** |
| "재해 복구" / "DR" / "RPO/RTO" | **Multi-Region** + DR 4대 전략 |
| "백업 자동화" | **AWS Backup** |
| "DNS 장애 조치" / "failover routing" | **Route 53 Failover** |
| "데이터베이스 자동 장애 조치" | **RDS Multi-AZ** / **Aurora Multi-AZ** |
| "글로벌 데이터베이스" | **Aurora Global Database** / **DynamoDB Global Tables** |
| "서비스 쿼터 관리" | **Service Quotas** + **Trusted Advisor** |

---

## 3. 4대 영역별 핵심 서비스

### 3-1. 기반 (Foundations)

| 요소 | 서비스 | 설명 |
|------|--------|------|
| **네트워크 설계** | VPC, Subnet, AZ | 멀티 AZ 서브넷 분산 |
| **서비스 한도** | Service Quotas | 한도 모니터링 + 사전 증가 요청 |
| **네트워크 용량** | Direct Connect, VPN | 온프레미스 연결 이중화 |

💡 "**서비스 한도 초과 방지**" → **Service Quotas** + **Trusted Advisor**

### 3-2. 워크로드 아키텍처 (Workload Architecture)

| 패턴 | 설명 | 서비스 |
|------|------|--------|
| **분산 시스템** | 단일 컴포넌트 장애가 전체에 전파되지 않음 | SQS, SNS, EventBridge |
| **멱등성** | 동일 요청 여러 번 = 같은 결과 | SQS + DynamoDB Conditional Write |
| **Graceful Degradation** | 일부 장애 시 핵심 기능만 유지 | Circuit Breaker 패턴 |

### 3-3. 변경 관리 (Change Management)

| 도구 | 역할 |
|------|------|
| **CloudWatch Alarms** | 메트릭 기반 자동 스케일링 트리거 |
| **ASG** | EC2 자동 확장/축소 + unhealthy 교체 |
| **CloudFormation** | 인프라 변경 추적·롤백 |
| **AWS Config** | 구성 변경 이력 |

### 3-4. 장애 관리 (Failure Management)

| 전략 | 서비스 | 설명 |
|------|--------|------|
| **백업** | AWS Backup, EBS Snapshot, RDS Snapshot | 자동 백업 정책 |
| **Multi-AZ** | RDS, ELB, ASG, ElastiCache | AZ 장애 시 자동 전환 |
| **Multi-Region** | Route 53, Aurora Global, DynamoDB Global Tables, S3 CRR | 리전 장애 대비 |
| **장애 조치** | Route 53 Failover, RDS Auto Failover | DNS/DB 자동 전환 |

---

## 4. Multi-AZ vs Multi-Region 비교 (시험 핵심)

| 기준 | Multi-AZ | Multi-Region |
|------|----------|-------------|
| 보호 범위 | **AZ 장애** | **리전 장애** |
| 비용 | 중간 | **높음** |
| 복잡도 | 낮음 | 높음 |
| RTO/RPO | 짧음 (초~분) | 전략에 따라 다름 |
| 대표 서비스 | RDS Multi-AZ, ALB Cross-AZ | Aurora Global DB, DynamoDB Global Tables |

💡 "**대부분의 시험 문제는 Multi-AZ로 충분**" — Multi-Region은 "리전 장애" / "글로벌 사용자" 키워드일 때만

### DR 4대 전략 요약 (Ch4-6 참조)

| 전략 | RTO | RPO | 비용 |
|------|-----|-----|------|
| **Backup & Restore** | 시간 | 시간 | 💰 |
| **Pilot Light** | 10분~시간 | 분 | 💰💰 |
| **Warm Standby** | 분 | 초~분 | 💰💰💰 |
| **Active-Active (Multi-Site)** | **~0** | **~0** | 💰💰💰💰 |

---

## 5. 서비스별 안정성 기능 매핑

### EC2 / ASG

| 기능 | 설명 |
|------|------|
| **ASG Health Check** | EC2 상태 / ELB 헬스 체크 기반 unhealthy 교체 |
| **ASG Multi-AZ** | 여러 AZ에 인스턴스 분산 |
| **Launch Template** | 인스턴스 구성 버전 관리 |
| **Scaling Policy** | Target Tracking / Step / Simple / Scheduled / Predictive |

💡 "**self-healing**" = ASG가 unhealthy 인스턴스 종료 + 새 인스턴스 시작

### RDS / Aurora

| 기능 | RDS | Aurora |
|------|-----|--------|
| **Multi-AZ** | Standby (동기 복제, 읽기 불가) | **리더 복제본이 Standby** (읽기 가능) |
| **읽기 확장** | Read Replica (비동기, 수동 승격) | 최대 15개 리더 (자동 Failover) |
| **Failover 시간** | ~60초 | **~30초** |
| **스토리지** | 단일 AZ EBS | **6개 복사본 / 3 AZ** |
| **글로벌** | Cross-Region Read Replica | **Aurora Global Database** (<1초 복제) |

💡 "**읽기 확장 + 자동 장애 조치 + 최소 다운타임**" → **Aurora**

### S3

| 기능 | 설명 |
|------|------|
| **내구성** | **99.999999999% (11 9s)** |
| **가용성** | Standard: 99.99%, IA: 99.9% |
| **버전 관리** | 실수 삭제 복구 |
| **CRR (Cross-Region Replication)** | 리전 간 복제 (DR) |
| **SRR (Same-Region Replication)** | 같은 리전 내 복제 (로그 집계) |
| **Object Lock** | WORM (삭제/수정 방지) |

### DynamoDB

| 기능 | 설명 |
|------|------|
| **기본 Multi-AZ** | 3 AZ에 자동 복제 (설정 불필요) |
| **Global Tables** | **Multi-Region Active-Active** (Active-Active 읽기/쓰기) |
| **On-Demand 백업** | 수동 스냅샷 |
| **PITR** | 최대 35일 시점 복구 |
| **DAX** | 읽기 캐시 (마이크로초 지연) |

💡 "**Multi-Region Active-Active DB**" → **DynamoDB Global Tables** (유일한 Active-Active DB)

### ELB

| 기능 | 설명 |
|------|------|
| **Cross-Zone Load Balancing** | AZ 간 균등 분배 |
| **Health Check** | 비정상 타겟 자동 제외 |
| **ALB** | HTTP/HTTPS, 경로 기반 라우팅 |
| **NLB** | TCP/UDP, **고정 IP**, 초저지연 |
| **Connection Draining** | 기존 연결 완료 후 인스턴스 제거 |

### Route 53

| 라우팅 정책 | 안정성 용도 |
|------------|-----------|
| **Failover** | Primary 장애 → Secondary 자동 전환 |
| **Weighted** | 점진적 트래픽 이동 (Blue/Green) |
| **Latency** | 가장 빠른 리전으로 라우팅 |
| **Multi-value** | 여러 IP 반환 + 헬스 체크 (간단 LB) |
| **Geolocation** | 사용자 위치 기반 라우팅 |

---

## 6. 시험 빈출 시나리오

### 시나리오 1: "단일 장애 지점(SPOF) 제거"

**정답**: **Multi-AZ** + **ELB** + **ASG** (모든 계층 이중화)  
- Web: ALB + ASG (Multi-AZ)  
- DB: RDS Multi-AZ 또는 Aurora

### 시나리오 2: "리전 장애에도 서비스 유지"

**정답**: **Multi-Region Active-Active** (DynamoDB Global Tables + Route 53 Failover + CloudFront)  
또는 **Warm Standby** / **Pilot Light** (비용 절감)

### 시나리오 3: "예측 불가 트래픽 + 자동 확장"

**정답**: **ASG** (Target Tracking) + **ALB** / **Lambda** / **Aurora Serverless**

### 시나리오 4: "데이터베이스 읽기 부하 분산"

**정답**: **Aurora Read Replica** (최대 15개) 또는 **ElastiCache** / **DAX**

### 시나리오 5: "S3 데이터 리전 간 복제"

**정답**: **S3 CRR (Cross-Region Replication)** + 버전 관리 활성화 필수

### 시나리오 6: "RPO ~0 + RTO ~0"

**정답**: **Active-Active (Multi-Site)** — DynamoDB Global Tables + Route 53 + CloudFront  
💡 가장 비싸지만 가장 빠른 복구

### 시나리오 7: "EC2 장애 자동 교체"

**정답**: **ASG** (Min=1, Desired=1이라도 self-healing)

### 시나리오 8: "백업 정책 중앙 관리"

**정답**: **AWS Backup** (EBS, RDS, DynamoDB, EFS, FSx 등 통합 백업)

### 시나리오 9: "Aurora Failover 시간 최소화"

**정답**: **Aurora Reader Endpoint** + **Custom Endpoint** (애플리케이션 연결 문자열 변경 불필요)

### 시나리오 10: "글로벌 사용자 + 낮은 지연"

**정답**: **CloudFront** (정적) + **Aurora Global Database** (동적) + **Route 53 Latency Routing**

---

## 7. 함정 / 주의사항

### 함정 1: "고가용성(HA)" vs "내결함성(FT)" vs "내구성(Durability)"

| 개념 | 의미 | 예시 |
|------|------|------|
| **HA** | 서비스 중단 시간 최소화 | RDS Multi-AZ (Failover 있지만 짧은 중단) |
| **FT** | 장애 시에도 **중단 없음** | DynamoDB Global Tables Active-Active |
| **Durability** | 데이터가 **손실되지 않음** | S3 11 9s |

💡 "Multi-AZ RDS = **HA**, not FT" (Failover 중 몇 초 중단)

### 함정 2: RDS Multi-AZ Standby는 읽기 불가

- RDS Multi-AZ Standby = **동기 복제, 읽기 불가** (순수 장애 조치용)
- 읽기 분산 → **Read Replica** (비동기, 수동 승격)
- Aurora는 리더 = Standby 겸용 (**읽기 가능 + 자동 Failover**)
- 💡 "읽기 분산 + 장애 조치" → **Aurora** 선택

### 함정 3: ASG "self-healing" = Min 1이라도 동작

- Min=1, Desired=1 → 인스턴스 장애 시 새 인스턴스 시작
- 💡 "단일 인스턴스 + 자동 복구" → ASG (Min=1)

### 함정 4: S3 CRR은 버전 관리 필수

- CRR/SRR 활성화 → **소스 + 대상 버킷 모두 버전 관리 필수**
- 💡 "S3 복제" 문제 → 버전 관리 활성화 여부 확인

### 함정 5: Route 53 Failover vs Multi-value

- **Failover**: Primary/Secondary 1:1 (DR)
- **Multi-value**: 최대 8 IP, 각각 헬스 체크 (간단 LB 대체, DNS 레벨)
- 💡 "간단한 DNS 레벨 로드 밸런싱" → Multi-value
- 💡 "DR 장애 조치" → Failover

### 함정 6: Aurora Global Database vs DynamoDB Global Tables

| 기준 | Aurora Global | DynamoDB Global Tables |
|------|-------------|----------------------|
| 모델 | **1 Writer + N Reader** | **Active-Active** (모든 리전 R/W) |
| 복제 지연 | ~1초 | ~1초 |
| Failover | 수동 승격 (~1분) | **자동** (장애 리전 제외) |
| 용도 | 글로벌 읽기 + DR | **글로벌 읽기/쓰기** |

💡 "**모든 리전에서 쓰기**" → DynamoDB Global Tables  
💡 "**RDS 호환 + 글로벌 읽기**" → Aurora Global Database

---

## 8. 검증 필요 항목 ⚠️

- [ ] Aurora Global Database 최신 Failover 시간 (Managed Planned Failover?)
- [ ] DynamoDB Global Tables v2 최신 기능 (2019 버전 이후 변경?)
- [ ] ASG Predictive Scaling 최신 지원 범위
- [ ] AWS Backup 최신 지원 서비스 목록
- [ ] RDS Multi-AZ Cluster (읽기 가능 Standby) 최신 GA 상태
- [ ] Route 53 Application Recovery Controller 최신 기능

---

## 9. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "HA / SPOF 제거" → Multi-AZ + ELB + ASG
- **유형 2**: "DR / RPO / RTO" → 4대 전략 선택
- **유형 3**: "자동 복구 / self-healing" → ASG
- **유형 4**: "글로벌 Active-Active DB" → DynamoDB Global Tables
- **유형 5**: "읽기 확장 + 자동 Failover" → Aurora
- **유형 6**: "백업 중앙 관리" → AWS Backup
- **유형 7**: "DNS 장애 조치" → Route 53 Failover

---

## 10. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
