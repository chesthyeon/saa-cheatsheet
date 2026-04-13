# Chapter 0-4. 성능 효율성 (Performance Efficiency)

## 0. 한 줄 요약

🔑 **"성능 효율성 = 워크로드에 맞는 리소스 타입·크기를 선택하고, 캐싱·엣지·서버리스로 지연을 줄이며, 수요 변화에 자동 대응하는 것"**

---

## 1. 핵심 설계 원칙 (5가지)

| # | 원칙 | 설명 | AWS 서비스 예시 |
|---|------|------|---------------|
| 1 | **Democratize advanced technologies** | 복잡한 기술을 관리형 서비스로 사용 | Aurora, OpenSearch, SageMaker |
| 2 | **Go global in minutes** | 글로벌 배포로 지연 시간↓ | CloudFront, Global Accelerator, Aurora Global |
| 3 | **Use serverless architectures** | 서버 관리 없이 실행 | Lambda, Fargate, Aurora Serverless |
| 4 | **Experiment more often** | 다양한 리소스 타입 실험 | EC2 인스턴스 유형 변경, A/B 테스트 |
| 5 | **Consider mechanical sympathy** | 워크로드 특성에 맞는 기술 선택 | GPU 인스턴스, io2 EBS, DAX |

---

## 2. 키워드 → 서비스 매핑

| 시험 키워드 | 정답 서비스 |
|------------|-----------|
| "지연 시간 감소" / "latency" | **CloudFront** / **ElastiCache** / **DAX** / **Global Accelerator** |
| "정적 콘텐츠 가속" | **CloudFront** |
| "동적 콘텐츠 가속" | **CloudFront** (Dynamic Content) / **Global Accelerator** |
| "TCP/UDP 가속 + 고정 IP" | **Global Accelerator** |
| "DB 읽기 성능 향상" | **ElastiCache** (Redis/Memcached) / **DAX** (DynamoDB 전용) |
| "마이크로초 지연" | **DAX** / **ElastiCache** |
| "높은 IOPS" | **io2 Block Express** (EBS) / **Provisioned IOPS** |
| "처리량 중심 스토리지" | **st1** (EBS HDD) / **S3** |
| "HPC / 고성능 컴퓨팅" | **EC2 Cluster Placement Group** / **EFA** (Elastic Fabric Adapter) |
| "GPU 워크로드" | **P/G 인스턴스** (ML/그래픽) |
| "메모리 집약적" | **R/X 인스턴스** |
| "컴퓨팅 집약적" | **C 인스턴스** |
| "글로벌 사용자 + 지연 최소" | **CloudFront + Route 53 Latency Routing** |
| "세션 캐싱" | **ElastiCache Redis** |
| "API 응답 캐싱" | **API Gateway Cache** / **CloudFront** |

---

## 3. 4대 리소스 타입별 최적화

### 3-1. 컴퓨팅 (Compute)

| 선택 기준 | 서비스 | 설명 |
|----------|--------|------|
| **서버 관리 최소** | Lambda, Fargate | 서버리스 |
| **커스텀 OS/런타임** | EC2 | 완전 제어 |
| **컨테이너** | ECS/EKS + Fargate/EC2 | 오케스트레이션 |
| **고성능 컴퓨팅** | EC2 Cluster Placement Group + EFA | 노드 간 초저지연 |

#### EC2 인스턴스 패밀리 (시험 단골)

| 패밀리 | 용도 | 예시 |
|--------|------|------|
| **C** (Compute) | CPU 집약 (배치, 인코딩) | c6i, c7g |
| **R** (RAM) | 메모리 집약 (인메모리 DB, 캐시) | r6i, r7g |
| **X** | 초대형 메모리 (SAP HANA) | x2idn |
| **M** (General) | 범용 (웹, 앱 서버) | m6i, m7g |
| **I** (I/O) | 스토리지 집약 (NoSQL, DW) | i4i |
| **P/G** | GPU (ML 추론/학습, 그래픽) | p5, g5 |
| **T** (Burstable) | 버스트 가능 (개발, 소규모) | t3, t4g |
| **Inf/Trn** | ML 추론/학습 전용 칩 | inf2, trn1 |

💡 "**CPU 집약적 배치 처리**" → **C** / "**인메모리 캐시**" → **R** / "**ML 학습**" → **P**

#### Placement Group

| 종류 | 배치 | 용도 |
|------|------|------|
| **Cluster** | 같은 AZ, 같은 랙 | **초저지연 HPC** (노드 간 10 Gbps+) |
| **Spread** | 각기 다른 하드웨어 | **HA** (최대 7 인스턴스/AZ) |
| **Partition** | 논리적 파티션별 분리 | **대규모 분산 시스템** (Hadoop, Cassandra) |

💡 "**HPC / 노드 간 최소 지연**" → **Cluster Placement Group + EFA**

### 3-2. 스토리지 (Storage)

| 유형 | 서비스 | IOPS | 처리량 | 용도 |
|------|--------|------|-------|------|
| **블록 (고IOPS)** | **io2 Block Express** | 256,000 | 4,000 MB/s | DB, 트랜잭션 |
| **블록 (범용)** | **gp3** | 16,000 | 1,000 MB/s | 부트 볼륨, 일반 |
| **블록 (처리량)** | **st1** | 500 | 500 MB/s | 로그, DW |
| **파일 (NFS)** | **EFS** | 자동 확장 | 모드 선택 | Linux 공유 파일 |
| **파일 (Windows)** | **FSx for Windows** | - | - | Windows, SMB |
| **파일 (HPC)** | **FSx for Lustre** | 수백만 | 수백 GB/s | **HPC, ML** |
| **객체** | **S3** | - | 무제한 | 데이터 레이크, 백업 |

💡 "**HPC 파일 시스템**" → **FSx for Lustre** (S3 연동 가능)  
💡 "**고IOPS DB**" → **io2 Block Express**  
💡 "**처리량 중심 빅데이터**" → **st1** 또는 **S3**

### 3-3. 데이터베이스 (Database)

| 요구 | 서비스 |
|------|--------|
| **관계형 + 읽기 확장** | Aurora (최대 15 Read Replica) |
| **마이크로초 KV 조회** | DynamoDB + **DAX** |
| **세션/캐시** | **ElastiCache Redis** |
| **전문 검색** | **OpenSearch** |
| **그래프 쿼리** | **Neptune** |
| **시계열** | **Timestream** |
| **원장 (불변)** | **QLDB** |

### 3-4. 네트워크 (Network)

| 요구 | 서비스 |
|------|--------|
| **정적 콘텐츠 캐싱** | CloudFront |
| **API 캐싱** | API Gateway Cache / CloudFront |
| **글로벌 TCP/UDP 가속** | **Global Accelerator** |
| **리전 내 초저지연** | **Cluster Placement Group + EFA** |
| **VPN 없이 AWS 백본 사용** | **PrivateLink** / **Global Accelerator** |

#### CloudFront vs Global Accelerator

| 기준 | CloudFront | Global Accelerator |
|------|------------|-------------------|
| 계층 | **L7 (HTTP/HTTPS)** | **L4 (TCP/UDP)** |
| 캐싱 | ✅ | ❌ |
| 고정 IP | ❌ (도메인 기반) | ✅ **Anycast IP 2개** |
| 용도 | 웹 콘텐츠, API | **게임, IoT, VoIP, 비HTTP** |
| 장애 조치 | Origin Failover | **엔드포인트 헬스 체크 + 즉시 전환** |

💡 "**고정 IP + TCP 가속**" → **Global Accelerator**  
💡 "**캐싱 + HTTP**" → **CloudFront**

---

## 4. 캐싱 전략 총정리 (시험 빈출)

| 캐싱 위치 | 서비스 | 대상 |
|----------|--------|------|
| **엣지 (CDN)** | CloudFront | 정적/동적 웹 콘텐츠 |
| **API 레벨** | API Gateway Cache | API 응답 |
| **애플리케이션** | ElastiCache | DB 쿼리 결과, 세션 |
| **DB 앞단** | DAX | DynamoDB 읽기 (마이크로초) |
| **DNS** | Route 53 (TTL) | DNS 응답 |

💡 "**캐싱 어디서?**" → 사용자에 가까울수록 효과적: **CloudFront > API GW Cache > ElastiCache/DAX**

### ElastiCache: Redis vs Memcached

| 기준 | Redis | Memcached |
|------|-------|-----------|
| 데이터 구조 | 다양 (String, Hash, List, Set, Sorted Set) | Key-Value만 |
| 복제 | ✅ Multi-AZ | ❌ |
| 백업 | ✅ 스냅샷 | ❌ |
| Pub/Sub | ✅ | ❌ |
| 멀티스레드 | ❌ (단일 스레드) | ✅ |
| 용도 | **세션, 리더보드, Pub/Sub, 복잡한 캐시** | **단순 캐시, 멀티스레드 필요** |

💡 시험에서 **대부분 Redis가 정답** (기능 풍부, HA 지원)

---

## 5. 시험 빈출 시나리오

### 시나리오 1: "글로벌 사용자 지연 시간 최소화 (웹)"

**정답**: **CloudFront** + **Route 53 Latency Routing** + **Aurora Global Database**

### 시나리오 2: "DynamoDB 읽기 지연 마이크로초"

**정답**: **DAX** (DynamoDB 전용 캐시)  
**오답**: ElastiCache (가능하지만 DAX가 더 간단, DynamoDB 네이티브)

### 시나리오 3: "HPC 워크로드, 노드 간 최소 지연"

**정답**: **Cluster Placement Group** + **EFA** (Enhanced Networking)

### 시나리오 4: "게임 서버, TCP 가속 + 고정 IP"

**정답**: **Global Accelerator**

### 시나리오 5: "DB 쿼리 반복 + 읽기 부하 분산"

**정답**: **ElastiCache Redis** (또는 Aurora Read Replica)

### 시나리오 6: "ML 학습 대규모 데이터셋 읽기"

**정답**: **FSx for Lustre** (S3 연동, 고처리량 파일 시스템)

### 시나리오 7: "API 응답 캐싱 + 서버리스"

**정답**: **API Gateway Cache** (Stage 레벨 TTL 설정)

### 시나리오 8: "높은 IOPS DB 워크로드"

**정답**: **io2 Block Express** EBS 또는 **Provisioned IOPS**

### 시나리오 9: "세션 저장소 (상태 유지 + 확장)"

**정답**: **ElastiCache Redis** (또는 DynamoDB)  
💡 Redis = 세션 + TTL + Multi-AZ

### 시나리오 10: "비HTTP 프로토콜 글로벌 가속"

**정답**: **Global Accelerator** (TCP/UDP)

---

## 6. 함정 / 주의사항

### 함정 1: CloudFront vs Global Accelerator 혼동

- "캐싱" / "HTTP" → CloudFront
- "고정 IP" / "TCP/UDP" / "게임" → Global Accelerator
- 💡 둘 다 AWS 글로벌 네트워크 사용하지만 **계층이 다름**

### 함정 2: DAX vs ElastiCache

- DAX: **DynamoDB 전용**, 코드 변경 최소 (SDK 엔드포인트만 변경)
- ElastiCache: **범용** 캐시 (RDS, API 결과 등)
- 💡 "DynamoDB 캐시" 키워드 → **DAX** 먼저

### 함정 3: gp3 vs io2 선택

- **gp3**: 3,000 IOPS 기본 (최대 16,000), **가성비 최고**
- **io2**: 높은 IOPS 보장 (최대 64,000, Block Express 256,000)
- 💡 "**consistent high IOPS**" / "**mission-critical DB**" → io2
- 💡 "**일반 부트 볼륨**" / "**비용 효율**" → gp3

### 함정 4: Cluster Placement Group = 같은 AZ

- 초저지연이지만 **AZ 장애 시 전체 영향**
- HA가 중요하면 → **Spread Placement Group**
- 💡 "HPC + 최소 지연" → Cluster / "HA + 분리" → Spread

### 함정 5: ElastiCache Redis 대부분 정답

- SAA 시험에서 Memcached가 정답인 경우는 극히 드묾
- "단순 캐시 + 멀티스레드" 명시적 키워드가 없으면 → **Redis**
- 💡 Redis = HA + 백업 + Pub/Sub + 다양한 자료구조

### 함정 6: T 인스턴스 (Burstable) 함정

- CPU 크레딧 소진 시 **성능 급락** (Unlimited 모드 아니면)
- 💡 "지속적으로 높은 CPU" → T 인스턴스 ❌, C/M 인스턴스 ✅

---

## 7. 검증 필요 항목 ⚠️

- [ ] io2 Block Express 최신 IOPS 한도 (256K 유지?)
- [ ] gp3 기본 IOPS/처리량 최신값
- [ ] Global Accelerator 최신 가격 모델
- [ ] EFA 지원 인스턴스 타입 최신 목록
- [ ] ElastiCache Serverless 최신 기능 (Serverless 모드?)
- [ ] DAX vs ElastiCache Serverless 비교

---

## 8. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "지연 시간 최소" → CloudFront / ElastiCache / DAX / Global Accelerator
- **유형 2**: "HPC / 노드 간 최소 지연" → Cluster Placement Group + EFA
- **유형 3**: "DB 읽기 캐싱" → ElastiCache Redis / DAX
- **유형 4**: "EC2 인스턴스 타입 선택" → C/R/M/P/I 패밀리
- **유형 5**: "고정 IP + TCP 가속" → Global Accelerator
- **유형 6**: "높은 IOPS 스토리지" → io2 Block Express

---

## 9. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
