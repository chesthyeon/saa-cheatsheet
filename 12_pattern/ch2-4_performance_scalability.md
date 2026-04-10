# Chapter 2-4. 성능·확장성 키워드 → 서비스 매핑

## 0. 한 줄 요약

🔑 **"성능 최적화 = 캐싱(CloudFront/ElastiCache/DAX) + 수평 확장(ASG/Read Replica) + 인스턴스 타입 선택 + 스토리지 최적화(gp3/io2/FSx)"**

---

## 1. 성능 최적화 4대 축

| 축 | 설명 | 주요 서비스 |
|----|------|-----------|
| **1. 캐싱** | 반복 요청 비용/지연 감소 | CloudFront, ElastiCache, DAX, API GW 캐시 |
| **2. 수평 확장** | 부하 분산 + 자동 증가 | ASG, ALB, Read Replica, DynamoDB 샤딩 |
| **3. 적정 리소스** | 올바른 인스턴스/스토리지 타입 | EC2 패밀리, EBS 타입, Compute Optimizer |
| **4. 글로벌 배포** | 지연 최소화 (엣지) | CloudFront, Global Accelerator, Route 53 Latency |

---

## 2. 캐싱 계층별 서비스

| 계층 | 서비스 | 용도 |
|------|--------|------|
| **엣지** | CloudFront | 정적 콘텐츠, HTTPS 종료 |
| **엣지** | CloudFront Functions / Lambda@Edge | 요청/응답 변환 |
| **API** | API Gateway 캐시 | REST API 응답 캐싱 |
| **앱** | ElastiCache (Redis/Memcached) | DB 쿼리 결과, 세션 |
| **DB** | **DAX** (DynamoDB Accelerator) | DynamoDB 읽기 (μs) |
| **DNS** | Route 53 + TTL | DNS 응답 캐싱 |

### ElastiCache Redis vs Memcached

| 기준 | Redis | Memcached |
|------|-------|-----------|
| 데이터 구조 | 풍부 (List, Set, Hash, Sorted Set) | Key-Value만 |
| 복제 | ✅ Multi-AZ | ❌ |
| 영속성 | ✅ RDB/AOF | ❌ |
| 트랜잭션 | ✅ | ❌ |
| Pub/Sub | ✅ | ❌ |
| Multi-threaded | ❌ | ✅ |
| 사용 사례 | 세션, 실시간 랭킹 | **단순 캐싱** (멀티 스레드 성능) |

💡 "세션 저장", "HA", "복잡 구조" → **Redis**  
💡 "단순 캐싱", "멀티 스레드" → **Memcached**

---

## 3. EC2 인스턴스 패밀리

| 패밀리 | 용도 | 예시 |
|--------|------|------|
| **T** | 버스트 가능 (개발, 소규모) | t3, t4g |
| **M** | 범용 (균형) | m5, m6i |
| **C** | **컴퓨팅 최적화** (CPU) | c5, c6i, c7g |
| **R** | **메모리 최적화** | r5, r6i (인메모리 DB) |
| **X** | **고메모리** (SAP HANA) | x1e, x2 |
| **I** | **I/O 최적화** (로컬 NVMe SSD) | i3, i4i |
| **D** | **HDD 밀도** (빅데이터) | d2, d3 |
| **P/G** | **GPU** (ML, 그래픽) | p4, g5 |
| **F** | FPGA | f1 |
| **Inf/Trn** | ML 추론/학습 | inf2, trn1 |

**빠른 매핑:**

| 키워드 | 패밀리 |
|--------|--------|
| "CPU 집약적" | **C** |
| "메모리 집약적" (인메모리 DB, 캐시) | **R** |
| "SAP HANA", "고메모리" | **X** |
| "I/O 집약적", "로컬 SSD" | **I** |
| "ML 학습" | **P** (GPU) |
| "ML 추론" + "비용 최적화" | **Inf** (Inferentia) |

---

## 4. EBS 볼륨 타입

| 타입 | 용도 | 성능 | 시험 키워드 |
|------|------|------|-----------|
| **gp3** | 범용 SSD | 3,000~16,000 IOPS | "**기본 권장**", "비용 효율" |
| **gp2** | 범용 SSD (레거시) | 100~16,000 IOPS | (gp3로 전환 권장) |
| **io2 / io2 Block Express** | 고성능 SSD | 최대 256,000 IOPS | "고성능 DB", "프로덕션 DB" |
| **io1** | 고성능 SSD (레거시) | 최대 64,000 IOPS | (io2로 전환) |
| **st1** | 처리량 HDD | 500 MB/s | "빅데이터", "로그 처리" (스트리밍) |
| **sc1** | 콜드 HDD | 250 MB/s | "저빈도 액세스" |

💡 **기본**: gp3 (가장 좋은 가성비)  
💡 **고IOPS DB**: io2 Block Express  
💡 **빅데이터 순차 I/O**: st1  
💡 **저비용 콜드**: sc1

### EBS vs 인스턴스 스토어

| 기준 | EBS | Instance Store |
|------|-----|---------------|
| 지속성 | ✅ 유지 | ❌ 인스턴스 중지 시 **손실** |
| 성능 | 높음 | **최고** (로컬 NVMe) |
| 사용 사례 | 범용 | 캐시, 임시 버퍼 |

💡 "인스턴스 중지/종료 시 데이터 유지" → **EBS**  
💡 "가장 빠른 로컬 스토리지" + "임시 데이터" → **Instance Store** (i3 패밀리)

---

## 5. FSx 4종 성능 비교

| FSx 종류 | 대상 | 성능 특징 | 시험 키워드 |
|---------|------|---------|-----------|
| **FSx for Windows** | Windows 앱 | SMB, AD 통합 | "Windows 파일 서버" |
| **FSx for Lustre** | HPC, ML | **초고성능** (수백 GB/s) | "HPC", "ML 학습", "고성능 병렬" |
| **FSx for NetApp ONTAP** | NetApp 사용자 | 멀티 프로토콜 (NFS/SMB/iSCSI) | "NetApp 호환" |
| **FSx for OpenZFS** | Linux 고성능 | NFS, 고 IOPS | "ZFS 스냅샷", "Linux 고성능" |

💡 "HPC / ML 학습" → **FSx for Lustre**

---

## 6. DB 성능 확장 패턴

### 읽기 확장

| 방법 | 서비스 | 키워드 |
|------|--------|--------|
| **Read Replica** | RDS, Aurora | "읽기 부하 분산" |
| **ElastiCache** | Redis/Memcached | "반복 쿼리 캐싱" |
| **DAX** | DynamoDB | "DynamoDB μs 지연" |
| **CloudFront** | 정적 API 응답 | "API 응답 캐싱" |

### 쓰기 확장

| 방법 | 서비스 | 키워드 |
|------|--------|--------|
| **샤딩** | DynamoDB (파티션 키 설계) | "DynamoDB 스케일" |
| **수직 확장** | 인스턴스 크기 업그레이드 | 한계 있음 |
| **Aurora Multi-Master** | Aurora (레거시) | "다중 쓰기 노드" |
| **큐 버퍼링** | SQS + 배치 쓰기 | "쓰기 스파이크 흡수" |

### DynamoDB 성능

| 기능 | 설명 | 키워드 |
|------|------|-------|
| **GSI** (Global Secondary Index) | 대체 파티션/정렬 키 | "다른 키로 쿼리" |
| **LSI** (Local Secondary Index) | 동일 파티션, 다른 정렬 키 | 테이블 생성 시만 |
| **DAX** | 인메모리 캐시 | "μs 지연" |
| **Streams** | 변경 캡처 | "이벤트 트리거" |
| **Adaptive Capacity** | 핫 파티션 자동 조정 | "핫 키" |

---

## 7. 글로벌 지연 최소화

| 서비스 | 역할 | 키워드 |
|--------|------|--------|
| **CloudFront** | HTTP/HTTPS 엣지 캐싱 | "정적 콘텐츠", "지연 감소" |
| **Global Accelerator** | 글로벌 Anycast IP (TCP/UDP) | "비 HTTP", "리전 페일오버" |
| **Route 53 Latency Routing** | DNS로 가장 빠른 리전 | "DNS 기반 글로벌" |
| **S3 Transfer Acceleration** | 글로벌 S3 업로드 가속 | "원격지 업로드" |
| **DynamoDB Global Tables** | 멀티리전 Active-Active | "글로벌 DB" |
| **Aurora Global Database** | 멀티리전 읽기 | "글로벌 읽기 DB" |

### CloudFront vs Global Accelerator

| 기준 | CloudFront | Global Accelerator |
|------|-----------|-------------------|
| 프로토콜 | HTTP/HTTPS | **TCP/UDP** (모든 포트) |
| 캐싱 | ✅ | ❌ |
| 용도 | **정적/동적 웹** | 게임, IoT, 비 HTTP 앱 |
| 페일오버 속도 | 느림 (DNS TTL) | **빠름** (Anycast IP) |

💡 "HTTP 웹앱" → **CloudFront**  
💡 "TCP 게임 서버", "비 HTTP + 글로벌 페일오버" → **Global Accelerator**

---

## 8. 시험 빈출 시나리오

### 시나리오 1: "DB 읽기 성능 향상"

**정답 옵션:**
1. **ElastiCache** (쿼리 결과 캐싱) — 가장 효과적
2. **Read Replica** (읽기 분산)
3. **DAX** (DynamoDB만)

### 시나리오 2: "웹앱 전 세계 지연 감소"

**정답**: **CloudFront** (엣지 캐싱)  
**대안**: Global Accelerator (비 HTTP) + 리전 배포

### 시나리오 3: "ML 학습 병렬 파일 시스템"

**정답**: **FSx for Lustre** (수백 GB/s)

### 시나리오 4: "DynamoDB 마이크로초 읽기 지연"

**정답**: **DAX**

### 시나리오 5: "CPU 집약 워크로드"

**정답**: **C 패밀리** (Compute Optimized)

### 시나리오 6: "메모리 집약 워크로드 (인메모리 DB)"

**정답**: **R 패밀리** (Memory Optimized)

### 시나리오 7: "EBS 볼륨 선택"

| 워크로드 | 정답 |
|---------|------|
| 기본 범용 | **gp3** |
| 고성능 DB (24K+ IOPS) | **io2 Block Express** |
| 빅데이터 처리 (순차) | **st1** |
| 저빈도 콜드 | **sc1** |

### 시나리오 8: "Stateless ASG 세션 유지"

**정답**: **ElastiCache Redis** (세션 외부화)  
**대안**: DynamoDB  
**오답 함정**: EBS (인스턴스 로컬)

---

## 9. 빠른 매핑 치트표

| 키워드 | 정답 |
|--------|------|
| "정적 콘텐츠 가속" | **CloudFront** |
| "DB 쿼리 캐싱" | **ElastiCache** |
| "DynamoDB μs" | **DAX** |
| "읽기 부하 분산" | **Read Replica** |
| "HPC/ML 고성능 파일 시스템" | **FSx for Lustre** |
| "CPU 집약" | **C 패밀리** |
| "메모리 집약" | **R 패밀리** |
| "고 IOPS DB" | **io2 Block Express** |
| "기본 SSD" | **gp3** |
| "글로벌 TCP/UDP 가속" | **Global Accelerator** |
| "S3 글로벌 업로드 가속" | **Transfer Acceleration** |
| "API 응답 캐시" | **API Gateway Cache** |
| "세션 저장" | **ElastiCache Redis** |
| "DynamoDB 쓰기 스파이크" | **SQS 버퍼** 또는 **On-Demand** |

---

## 10. 함정 / 주의사항

### 함정 1: ElastiCache Redis = Multi-AZ, Memcached ≠ Multi-AZ

- Redis: ✅ Multi-AZ, 복제, 영속성
- Memcached: ❌ Multi-AZ, 단순 캐시만
- 💡 "HA 캐시" → Redis

### 함정 2: CloudFront는 HTTP만

- 게임, IoT, 비 HTTP 앱에 CloudFront ❌
- → **Global Accelerator** 사용

### 함정 3: gp2 → gp3 전환 권장

- gp3가 **20% 저렴 + 더 빠른 기본 성능** (3,000 IOPS)
- 💡 "gp2 사용 중" → gp3로 전환 (즉시 개선)

### 함정 4: Instance Store는 영속적이지 않다

- 인스턴스 중지/종료 시 **데이터 손실**
- 캐시, 임시 처리에만 사용
- 💡 "지속 데이터" → EBS

---

## 11. 검증 필요 항목 ⚠️

- [ ] gp3 최대 성능 (16K IOPS? 변경?)
- [ ] io2 Block Express 최대 IOPS (256K?)
- [ ] FSx for Lustre 최대 처리량
- [ ] Graviton3 (c7g/m7g/r7g) 비용 효율 최신 벤치마크
- [ ] DAX 최신 지원 API

---

## 12. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "캐싱 전략" → CloudFront / ElastiCache / DAX
- **유형 2**: "EC2 인스턴스 패밀리 선택"
- **유형 3**: "EBS 볼륨 타입 선택"
- **유형 4**: "DB 성능 확장" → Read Replica / 캐싱 / 샤딩
- **유형 5**: "글로벌 성능" → CloudFront vs Global Accelerator

---

## 13. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
