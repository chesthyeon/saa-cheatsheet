# Chapter 1-2. 스토리지

## 0. 한 줄 요약

🔑 **"접근 빈도 × 공유 범위 × 프로토콜"이 스토리지 선택 기준이다 — 객체(S3) / 블록(EBS) / 파일(EFS·FSx) / 하이브리드(Storage Gateway)**

---

## 1. 핵심 서비스 표

| 서비스 | 한 줄 정의 | 주요 키워드 | 가격 모델 |
|--------|-----------|------------|----------|
| **S3** | 무제한 객체 스토리지 | 버킷, 객체, 스토리지 클래스, 버전 관리, 수명 주기 | 저장량 + 요청 수 + 전송량 |
| **EBS** | EC2 전용 블록 스토리지 | 볼륨 타입(gp3/io2/st1/sc1), 스냅샷, 암호화 | 프로비저닝 용량(GB) + IOPS(io2) |
| **EFS** | 관리형 NFS 파일 시스템 (Linux) | 다중 AZ, 다중 EC2 마운트, POSIX | 사용량 기반 (GB) |
| **FSx for Windows** | 관리형 Windows 파일 서버 | SMB, Active Directory, NTFS | 프로비저닝 용량 |
| **FSx for Lustre** | 고성능 병렬 파일 시스템 | HPC, ML 학습, S3 연동 | 프로비저닝 용량 |
| **FSx for NetApp ONTAP** | 멀티 프로토콜 (NFS+SMB+iSCSI) | 온프레미스 NetApp 마이그레이션, 데이터 중복 제거 | 프로비저닝 용량 + SSD/HDD |
| **FSx for OpenZFS** | 고성능 NFS + 스냅샷 | Linux 워크로드, ZFS 마이그레이션, 데이터 압축 | 프로비저닝 용량 |
| **Storage Gateway - File** | 온프레미스 → S3 파일 인터페이스 | NFS/SMB → S3 백업, 로컬 캐시 | S3 비용 + 게이트웨이 VM |
| **Storage Gateway - Volume** | 온프레미스 → S3 블록 볼륨 | iSCSI, Cached/Stored 모드 | S3 비용 + 스냅샷 |
| **Storage Gateway - Tape** | 온프레미스 → S3 가상 테이프 | 백업 소프트웨어 호환, VTL | S3 Glacier 비용 |

---

## 2. 키워드 → 서비스 매핑

### S3가 정답인 키워드

| 시험 키워드 | 왜 S3인가 |
|------------|----------|
| "객체 스토리지", "무제한 저장" | S3는 용량 제한 없는 객체 스토리지 |
| "정적 웹사이트 호스팅" | S3 + CloudFront 조합 |
| "데이터 레이크", "분석용 원시 데이터" | S3가 AWS 데이터 레이크의 표준 저장소 |
| "백업", "아카이브" | S3 Glacier 클래스 활용 |
| "Cross-Region Replication" | S3 CRR로 리전 간 복제 |
| "11 9's 내구성" | S3 Standard: 99.999999999% 내구성 |
| "수명 주기 정책" | 접근 빈도에 따라 자동 클래스 전환 |
| "이벤트 기반 처리" | S3 Event → Lambda/SQS/SNS |

### S3 스토리지 클래스 매핑

| 시험 키워드 | 정답 클래스 |
|------------|-----------|
| "자주 액세스", "즉시 접근" | **S3 Standard** |
| "접근 빈도 예측 불가" | **S3 Intelligent-Tiering** |
| "30일 이상 보관, 가끔 액세스" | **S3 Standard-IA** |
| "단일 AZ + 비용 절감 + 재생성 가능" | **S3 One Zone-IA** |
| "즉시 액세스 + 아카이브 비용" | **S3 Glacier Instant Retrieval** |
| "분~시간 내 검색, 90일 이상 보관" | **S3 Glacier Flexible Retrieval** |
| "12시간 이내 검색, 180일 이상, 최저 비용" | **S3 Glacier Deep Archive** |

> ⚠️ **시험 핵심**: "최소 보관 기간"과 "검색 시간"으로 클래스를 구분한다.

### EBS가 정답인 키워드

| 시험 키워드 | 왜 EBS인가 |
|------------|-----------|
| "블록 스토리지", "EC2 디스크" | EC2의 기본 스토리지 |
| "부팅 볼륨", "루트 디바이스" | OS 설치용 블록 볼륨 |
| "고성능 IOPS", "데이터베이스 스토리지" | io2 Block Express: 최대 256,000 IOPS |
| "스냅샷", "증분 백업" | EBS 스냅샷 → S3에 저장 |
| "단일 인스턴스 연결" | 기본적으로 1 EC2에 1 EBS (Multi-Attach 제외) |
| "암호화 at rest" | EBS 암호화 (KMS 키) |

### EBS 볼륨 타입 매핑

| 시험 키워드 | 정답 타입 |
|------------|---------|
| "범용 SSD", "부팅 볼륨", "대부분의 워크로드" | **gp3** (gp2 대비 비용 효율적) |
| "최고 IOPS", "미션 크리티컬 DB" | **io2 Block Express** |
| "빅데이터", "대용량 순차 읽기/쓰기" | **st1** (처리량 최적화 HDD) |
| "최저 비용", "자주 액세스하지 않는 데이터" | **sc1** (Cold HDD) |

> ⚠️ **함정**: st1/sc1은 **부팅 볼륨으로 사용 불가**. 부팅은 gp/io만 가능.

### EFS가 정답인 키워드

| 시험 키워드 | 왜 EFS인가 |
|------------|-----------|
| "다중 EC2에서 공유 파일 시스템" | NFS 프로토콜로 다중 마운트 |
| "Linux", "POSIX 호환" | EFS = Linux 전용 |
| "Auto Scaling과 함께 공유 스토리지" | EC2가 늘어도 동일 파일 시스템 마운트 |
| "서버리스 파일 시스템" | EFS는 용량 자동 확장/축소 |
| "컨테이너 공유 볼륨" (ECS/EKS + Linux) | Fargate + EFS 조합 |

### FSx가 정답인 키워드

| 시험 키워드 | 정답 FSx 유형 |
|------------|-------------|
| "Windows 파일 서버", "SMB", "Active Directory" | **FSx for Windows File Server** |
| "HPC", "ML 학습 데이터", "S3 연동 고성능" | **FSx for Lustre** |
| "온프레미스 NetApp 마이그레이션", "NFS+SMB+iSCSI" | **FSx for NetApp ONTAP** |
| "ZFS 마이그레이션", "NFS + 스냅샷 + 압축" | **FSx for OpenZFS** |

### Storage Gateway가 정답인 키워드

| 시험 키워드 | 정답 게이트웨이 유형 |
|------------|------------------|
| "온프레미스 → S3 파일 백업", "NFS/SMB" | **File Gateway** |
| "온프레미스 → S3 블록 볼륨", "iSCSI" | **Volume Gateway** |
| "온프레미스 테이프 백업 대체", "VTL" | **Tape Gateway** |
| "하이브리드 클라우드 스토리지" | Storage Gateway (유형은 프로토콜로 판별) |
| "로컬 캐시 + 클라우드 저장" | Volume Gateway (Cached mode) 또는 File Gateway |

---

## 3. 헷갈리는 포인트 / 함정

### 함정 1: S3 Standard-IA vs S3 One Zone-IA

- ❌ "비용 절감"만 보고 One Zone-IA 선택 → **AZ 장애 시 데이터 유실 가능**
- ✅ Standard-IA: 다중 AZ 복제, 중요 데이터에 적합
- ✅ One Zone-IA: 단일 AZ, **재생성 가능한 데이터**에만 적합 (썸네일, 로그 사본 등)
- 💡 "내구성이 중요하다" → One Zone-IA 제외

### 함정 2: S3 Glacier Instant Retrieval vs Glacier Flexible Retrieval

- ❌ "아카이브"라는 단어만 보고 Glacier Flexible 선택 → **즉시 접근이 필요하면 틀림**
- ✅ Glacier Instant Retrieval: **밀리초 단위** 검색, 분기 1회 정도 접근하는 데이터
- ✅ Glacier Flexible Retrieval: **분~12시간** 검색 (Expedited/Standard/Bulk)
- ✅ Glacier Deep Archive: **12~48시간** 검색, 최저 비용
- 💡 "즉시 액세스" + "아카이브 비용" → Glacier Instant Retrieval

### 함정 3: EBS는 단일 AZ, EFS는 다중 AZ

- ❌ "고가용성 공유 스토리지" → EBS 선택 → **틀림**
- ✅ EBS: 단일 AZ에 종속, 스냅샷으로만 다른 AZ에 복원 가능
- ✅ EFS: 리전 내 다중 AZ 자동 복제
- 💡 "다중 AZ" + "공유" → EFS (Linux) 또는 FSx (Windows/특수 용도)

### 함정 4: EFS는 Linux 전용

- ❌ "Windows EC2에서 공유 파일 시스템" → EFS 선택 → **틀림**
- ✅ EFS: NFS 프로토콜 → **Linux만 지원**
- ✅ Windows: **FSx for Windows File Server** (SMB + Active Directory)
- 💡 "Windows" 또는 "SMB" 키워드 → FSx for Windows

### 함정 5: st1/sc1은 부팅 볼륨 불가

- ❌ "최저 비용 부팅 볼륨" → sc1 선택 → **틀림**
- ✅ 부팅 볼륨은 **SSD 계열(gp3, gp2, io2)만 가능**
- ✅ st1(처리량 최적화), sc1(Cold)은 **데이터 볼륨 전용**
- 💡 부팅 + 비용 절감 → gp3 (gp2보다 저렴하고 성능 독립 설정 가능)

### 함정 6: S3 버전 관리 vs Cross-Region Replication

- ❌ "재해 복구" → 버전 관리로 충분 → **틀림** (버전 관리는 같은 리전)
- ✅ 버전 관리: 실수로 삭제/덮어쓰기 방지 (같은 버킷 내)
- ✅ CRR: **다른 리전으로 자동 복제** (DR 목적)
- 💡 "리전 장애 대비" → CRR 필수, 버전 관리만으로는 부족

### 함정 7: Storage Gateway Cached vs Stored 모드

- ❌ 모드 구분 없이 Volume Gateway 선택 → **요구사항에 따라 다름**
- ✅ **Cached mode**: 자주 쓰는 데이터만 로컬 캐시, 전체 데이터는 S3 → 로컬 스토리지 절약
- ✅ **Stored mode**: 전체 데이터를 로컬에 보관, S3에 비동기 백업 → 로컬 저지연 접근
- 💡 "로컬 스토리지 최소화" → Cached, "저지연 전체 접근" → Stored

---

## 4. 단골 시나리오

### 시나리오 1: 데이터 레이크 구축

```
문제 패턴: "다양한 형식의 원시 데이터를 중앙 저장소에 모아 분석"
정답 패턴: S3 (저장) + Glue (ETL) + Athena (쿼리)
키 포인트: S3가 AWS 데이터 레이크의 사실상 표준
```

### 시나리오 2: 비용 최적화된 장기 보관

```
문제 패턴: "규정 준수를 위해 7년간 보관, 거의 접근하지 않음"
정답 패턴: S3 Glacier Deep Archive + S3 Object Lock (규정 준수 모드)
키 포인트: "규정 준수" + "장기 보관" + "거의 접근 안 함" = Deep Archive + Object Lock
```

### 시나리오 3: 고성능 데이터베이스 스토리지

```
문제 패턴: "미션 크리티컬 DB, 최대 IOPS 필요, 지연 시간 최소화"
정답 패턴: EBS io2 Block Express
키 포인트: gp3가 아닌 io2 → "미션 크리티컬", "최대 IOPS" 키워드 확인
```

### 시나리오 4: Auto Scaling 그룹의 공유 스토리지

```
문제 패턴: "여러 EC2 인스턴스가 동일 파일에 읽기/쓰기"
정답 패턴: EFS (Linux) 또는 FSx for Windows (Windows)
키 포인트: EBS는 단일 인스턴스 → 공유 불가 (Multi-Attach는 특수 케이스)
```

### 시나리오 5: 온프레미스 백업을 클라우드로

```
문제 패턴: "기존 온프레미스 백업 인프라를 AWS로 전환, 기존 백업 소프트웨어 유지"
정답 패턴: Storage Gateway - Tape Gateway (VTL)
키 포인트: "기존 백업 소프트웨어" = 테이프 호환 → Tape Gateway
```

### 시나리오 6: HPC/ML 학습 데이터 고성능 접근

```
문제 패턴: "S3의 대용량 데이터셋을 고속으로 읽어서 ML 학습"
정답 패턴: FSx for Lustre (S3와 네이티브 연동)
키 포인트: Lustre는 S3를 데이터 소스로 자동 연결 가능
```

---

## 5. 검증 필요 항목 ⚠️

> Claude의 학습 데이터가 outdated일 수 있는 항목. AWS 공식 문서에서 확인 필요.

- [ ] S3 Intelligent-Tiering 최신 액세스 티어 구성 (Archive Access, Deep Archive Access 자동 활성화 여부)
  - 공식 문서: https://docs.aws.amazon.com/AmazonS3/latest/userguide/intelligent-tiering.html
- [ ] EBS io2 Block Express 최대 IOPS (256,000) 및 처리량 한도 확인
  - 공식 문서: https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html
- [ ] gp3 기본 제공 IOPS(3,000) / 처리량(125 MiB/s) 수치 확인
- [ ] EFS 요금 모델 변경 여부 (Standard vs One Zone, IA 티어 포함)
  - 공식 문서: https://aws.amazon.com/efs/pricing/
- [ ] FSx for Lustre의 S3 연동 방식 최신 상태 확인
- [ ] Storage Gateway 유형별 최신 성능 한도 및 요금
- [ ] S3 Object Lock 규정 준수 모드 vs 거버넌스 모드 최신 동작 확인
- [ ] EBS Multi-Attach 지원 볼륨 타입 (io1/io2만 가능한지 확인)

---

## 6. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "접근 빈도별 최적 스토리지 클래스" → S3 클래스 매핑 (가장 빈출)
- **유형 2**: "다중 EC2 공유 파일 시스템" → EFS (Linux) / FSx for Windows (Windows)
- **유형 3**: "고성능 DB 스토리지" → EBS io2 vs gp3
- **유형 4**: "온프레미스 → 클라우드 백업" → Storage Gateway 유형 선택
- **유형 5**: "데이터 레이크 중앙 저장소" → S3
- **유형 6**: "아카이브 + 규정 준수" → S3 Glacier Deep Archive + Object Lock
- **유형 7**: "HPC/ML 고성능 파일 시스템" → FSx for Lustre
- **유형 8**: "Windows 파일 서버 마이그레이션" → FSx for Windows File Server
- **유형 9**: "Cross-Region 재해 복구" → S3 CRR
- **유형 10**: "비용 최적화 + 접근 빈도 예측 불가" → S3 Intelligent-Tiering

---

## 7. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
