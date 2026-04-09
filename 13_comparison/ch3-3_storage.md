# Chapter 3-3. 스토리지: S3 클래스 / EBS vs EFS vs FSx 4종

## 0. 한 줄 요약

🔑 **"객체 vs 블록 vs 파일"로 1차 분류, "접근 빈도 / 공유 여부 / OS·프로토콜"로 2차 선택 — 이 두 단계만 거치면 정답이 나온다**

---

## 1. 스토리지 유형 1차 분류

| 유형 | 서비스 | 접근 방식 | 대표 사용 사례 |
|------|--------|----------|--------------|
| **객체** | S3 | HTTP API (PUT/GET) | 데이터 레이크, 백업, 정적 웹, 아카이브 |
| **블록** | EBS | EC2에 마운트 (디스크) | OS 부팅, DB 스토리지, 트랜잭션 I/O |
| **파일** | EFS, FSx | NFS/SMB/Lustre 마운트 | 다중 인스턴스 공유, HPC, Windows 파일 서버 |

> 💡 **1차 판별**: "EC2 디스크" → EBS, "다중 인스턴스 공유" → EFS/FSx, "무제한 객체 저장" → S3

---

## 2. S3 스토리지 클래스 전체 비교

| 클래스 | 최소 보관 | 검색 시간 | 가용성 | AZ | 핵심 키워드 |
|--------|----------|----------|--------|-----|-----------|
| **Standard** | 없음 | 즉시 | 99.99% | 다중 | 자주 액세스, 기본값 |
| **Intelligent-Tiering** | 없음 | 즉시 | 99.9% | 다중 | 접근 패턴 예측 불가, 자동 전환 |
| **Standard-IA** | **30일** | 즉시 | 99.9% | 다중 | 가끔 액세스, 중요 데이터 |
| **One Zone-IA** | **30일** | 즉시 | 99.5% | **단일** | 재생성 가능, 비용 절감 |
| **Glacier Instant Retrieval** | **90일** | **밀리초** | 99.9% | 다중 | 분기 1회 접근, 즉시 필요 |
| **Glacier Flexible Retrieval** | **90일** | **분~12시간** | 99.99% | 다중 | 연 1~2회, 검색 시간 유연 |
| **Glacier Deep Archive** | **180일** | **12~48시간** | 99.99% | 다중 | 최저 비용, 규정 준수 장기 보관 |

### S3 클래스 판별 플로우

```
"접근 빈도는?"
  ├─ 자주 (일/주) ──→ Standard
  ├─ 예측 불가 ──→ Intelligent-Tiering
  ├─ 가끔 (월) ──→ "데이터 중요도?"
  │   ├─ 중요 (유실 불가) ──→ Standard-IA
  │   └─ 재생성 가능 ──→ One Zone-IA
  └─ 거의 안 함 (아카이브) ──→ "검색 속도?"
      ├─ 즉시 (밀리초) ──→ Glacier Instant Retrieval
      ├─ 분~시간 OK ──→ Glacier Flexible Retrieval
      └─ 12시간+ OK ──→ Glacier Deep Archive
```

### S3 클래스 시험 빈출 함정

| 함정 | 설명 |
|------|------|
| Standard-IA vs One Zone-IA | "비용 절감"만 보고 One Zone 선택 → **AZ 장애 = 데이터 유실**. "재생성 가능"이 아니면 Standard-IA |
| Glacier Instant vs Flexible | "아카이브"만 보고 Flexible → **즉시 접근 필요하면 Instant**. "검색 시간"이 판별 기준 |
| Intelligent-Tiering 비용 | 모니터링 비용 발생 → 객체 수가 매우 많고 작으면 오히려 비쌀 수 있음 |
| 최소 보관 기간 위반 | 30일 미만에 IA 삭제 → **30일분 비용 청구됨**. 수명 주기 정책 설계 시 주의 |

---

## 3. EBS 볼륨 타입 전체 비교

| 타입 | 카테고리 | IOPS | 처리량 | 부팅 | 핵심 키워드 |
|------|---------|------|--------|------|-----------|
| **gp3** | 범용 SSD | 기본 3,000 (최대 16,000) | 기본 125 MiB/s (최대 1,000) | ✅ | 대부분의 워크로드, 비용 효율 |
| **gp2** | 범용 SSD | 용량 비례 (3 IOPS/GB) | 최대 250 MiB/s | ✅ | 레거시 (gp3 권장) |
| **io2 Block Express** | 프로비저닝 SSD | **최대 256,000** | 최대 4,000 MiB/s | ✅ | 미션 크리티컬 DB, 최대 성능 |
| **st1** | 처리량 HDD | N/A | 최대 500 MiB/s | ❌ | 빅데이터, 순차 읽기/쓰기 |
| **sc1** | Cold HDD | N/A | 최대 250 MiB/s | ❌ | 최저 비용, 자주 접근 안 함 |

### EBS 판별 핵심

| 시험 키워드 | 정답 |
|------------|------|
| "범용", "대부분 워크로드", "부팅 볼륨" | **gp3** |
| "최대 IOPS", "미션 크리티컬 DB" | **io2 Block Express** |
| "빅데이터", "로그 처리", "순차 I/O" | **st1** |
| "최저 비용 데이터 볼륨" | **sc1** |

> ⚠️ **핵심 함정**: st1/sc1은 **부팅 볼륨 불가**. gp3 vs gp2는 gp3가 비용 효율적 (IOPS/처리량 독립 설정).

---

## 4. EBS vs EFS vs FSx 전체 비교

| 기준 | EBS | EFS | FSx for Windows | FSx for Lustre |
|------|-----|-----|----------------|---------------|
| **유형** | 블록 | 파일 (NFS) | 파일 (SMB) | 파일 (병렬) |
| **프로토콜** | EC2 직접 연결 | **NFS v4** | **SMB** | **Lustre** |
| **OS 지원** | Linux/Windows | **Linux만** | **Windows** (+ Linux SMB) | Linux |
| **다중 인스턴스** | ❌ (Multi-Attach 예외) | ✅ **다중 AZ, 다중 EC2** | ✅ 다중 EC2 | ✅ 다중 EC2 |
| **AZ** | **단일 AZ** | 다중 AZ (또는 One Zone) | 단일/다중 AZ | 단일 AZ |
| **용량 관리** | 프로비저닝 (수동) | **자동 확장/축소** | 프로비저닝 | 프로비저닝 |
| **성능** | IOPS 기반 | 처리량 모드 선택 | 수백 MB/s | **수백 GB/s** |
| **비용** | 프로비저닝 용량 | 사용량 기반 | 프로비저닝 | 프로비저닝 |
| **백업** | 스냅샷 → S3 | AWS Backup | AWS Backup | S3 자동 연동 |
| **사용 사례** | DB, 부팅 디스크 | 웹 콘텐츠 공유, CMS | AD 환경, 홈 디렉토리 | HPC, ML 학습 |

### FSx 4종 비교

| 기준 | FSx for Windows | FSx for Lustre | FSx for NetApp ONTAP | FSx for OpenZFS |
|------|----------------|---------------|---------------------|----------------|
| **프로토콜** | SMB | Lustre | **NFS + SMB + iSCSI** | NFS |
| **OS** | Windows (+ Linux) | Linux | Linux/Windows | Linux |
| **핵심 기능** | AD 통합, NTFS | S3 네이티브 연동 | 데이터 중복 제거, SnapMirror | 스냅샷, 압축, 클론 |
| **시험 키워드** | "Windows 파일 서버", "AD", "SMB" | "HPC", "ML", "S3 고속 연동" | "NetApp 마이그레이션", "멀티 프로토콜" | "ZFS 마이그레이션", "NFS + 스냅샷" |

---

## 5. 핵심 판별 플로우차트

```
"스토리지 선택"
  │
  ├─ 객체 저장 (API 기반)? ──→ S3 (클래스는 접근 빈도로)
  │
  ├─ EC2 디스크 (단일 인스턴스)? ──→ EBS (타입은 IOPS/처리량으로)
  │
  └─ 다중 인스턴스 공유 파일 시스템?
      ├─ Linux + NFS? ──→ EFS
      ├─ Windows + SMB + AD? ──→ FSx for Windows
      ├─ HPC/ML + S3 연동? ──→ FSx for Lustre
      ├─ NetApp 마이그레이션? ──→ FSx for NetApp ONTAP
      └─ ZFS 마이그레이션? ──→ FSx for OpenZFS
```

---

## 6. 시험 빈출 시나리오 + 정답

| 시나리오 | 정답 | 오답 함정 |
|----------|------|----------|
| Auto Scaling 그룹의 EC2들이 동일 파일 공유 (Linux) | **EFS** | EBS (공유 불가) |
| Windows EC2 + Active Directory 파일 서버 | **FSx for Windows** | EFS (Linux 전용) |
| S3 대용량 데이터셋 → ML 학습 고속 읽기 | **FSx for Lustre** | EFS (성능 부족) |
| 미션 크리티컬 DB, 최대 IOPS | **EBS io2 Block Express** | gp3 (IOPS 한계) |
| 접근 빈도 예측 불가, 비용 최적화 | **S3 Intelligent-Tiering** | Standard-IA (패턴 예측 필요) |
| 7년 규정 준수 보관, 거의 접근 안 함 | **S3 Glacier Deep Archive + Object Lock** | Glacier Flexible (비용 높음) |
| 즉시 접근 가능한 아카이브 | **S3 Glacier Instant Retrieval** | Glacier Flexible (분~시간) |
| Fargate 컨테이너 공유 볼륨 (Linux) | **EFS** | EBS (Fargate 미지원) |
| 온프레미스 NetApp → AWS 마이그레이션 | **FSx for NetApp ONTAP** | EFS (NetApp 기능 부족) |
| 최저 비용 데이터 볼륨, 자주 접근 안 함 | **EBS sc1** | st1 (더 비쌈) |

---

## 7. 검증 필요 항목 ⚠️

- [ ] gp3 기본 IOPS(3,000) / 처리량(125 MiB/s) 최신 확인
- [ ] io2 Block Express 최대 IOPS(256,000) 변경 여부
- [ ] EBS Multi-Attach 지원 타입 (io1/io2만?) 확인
- [ ] EFS One Zone 클래스 최신 요금
- [ ] S3 Intelligent-Tiering Archive Access 자동 활성화 여부
- [ ] S3 각 클래스 최소 보관 기간 / 최소 객체 크기 최신 확인
- [ ] FSx for Lustre S3 연동 방식 최신 (자동 임포트/익스포트)
- [ ] FSx for NetApp ONTAP 최신 기능 (FlexClone, SnapMirror)

---

## 8. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "S3 스토리지 클래스 선택" → 접근 빈도 + 검색 시간으로 판별
- **유형 2**: "EBS 볼륨 타입 선택" → IOPS vs 처리량 vs 비용
- **유형 3**: "다중 EC2 공유 파일" → EFS vs FSx (OS/프로토콜로)
- **유형 4**: "HPC/ML 고성능 파일" → FSx for Lustre
- **유형 5**: "Windows 파일 서버" → FSx for Windows

---

## 9. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
