# Chapter 2-6. 마이그레이션 세분화 (DataSync vs Snow Family vs DMS/SCT vs MGN/AMS)

## 0. 한 줄 요약

🔑 **"데이터량 + 네트워크 가용 여부 + 대상(파일/DB/서버)로 서비스가 결정된다 — DataSync(온라인 파일) / Snow(오프라인 대용량) / DMS(DB) / MGN(서버 전체)"**

---

## 1. 마이그레이션 6R 전략

| 전략 | 설명 | 예시 |
|------|------|------|
| **Rehost** (Lift-and-Shift) | 그대로 이전 | **AWS MGN** |
| **Replatform** (Lift-Tinker-Shift) | 약간 수정 | MySQL → Aurora |
| **Repurchase** | SaaS 전환 | CRM → Salesforce |
| **Refactor** | 클라우드 네이티브 재설계 | 모놀리식 → 마이크로서비스 |
| **Retire** | 더 이상 필요 없음 | 사용 중단 |
| **Retain** | 유지 (마이그레이션 제외) | 규정상 온프레미스 유지 |

---

## 2. 데이터 마이그레이션 서비스 비교

| 서비스 | 유형 | 데이터량 | 네트워크 필요 | 대상 |
|--------|------|---------|-------------|------|
| **DataSync** | 온라인 (에이전트) | TB급 | ✅ 필수 (최소 100 Mbps 권장) | **파일** (NFS/SMB) → S3/EFS/FSx |
| **Storage Gateway** | 하이브리드 (지속) | 지속 동기화 | ✅ | 파일/블록/테이프 |
| **Snowcone** | 오프라인 | **~8 TB** | ❌ | S3 (물리 배송) |
| **Snowball Edge** | 오프라인 | **~80 TB** | ❌ | S3 (물리 배송) |
| **Snowmobile** | 오프라인 | **~100 PB** | ❌ | S3 (트럭 배송) |
| **Transfer Family** | 온라인 (SFTP/FTPS/FTP/AS2) | 중소 | ✅ | S3/EFS |
| **S3 Transfer Acceleration** | 온라인 가속 | 대용량 가능 | ✅ | S3 직접 |
| **DMS** (Database Migration Service) | 온라인 DB | DB 크기 | ✅ | DB → DB / S3 |
| **SCT** (Schema Conversion Tool) | 스키마 변환 | - | - | 이기종 DB 간 |
| **MGN** (Application Migration Service) | 온라인 서버 | 서버 블록 | ✅ | EC2 (Lift-and-Shift) |

---

## 3. 데이터량/네트워크 기준 선택 플로우

```
"온프레미스 → AWS 데이터 전송"
  │
  ├─ 데이터량?
  │   ├─ ~8 TB + 네트워크 제한 → Snowcone
  │   ├─ ~80 TB → Snowball Edge
  │   ├─ PB급 → Snowmobile
  │   └─ 지속적 (GB~TB) → DataSync / Storage Gateway
  │
  ├─ 네트워크 가용?
  │   ├─ No / 느림 → **Snow Family** (오프라인)
  │   └─ Yes + 빠름 → DataSync / Direct Connect
  │
  ├─ 데이터 유형?
  │   ├─ 파일 (NFS/SMB) → DataSync / Storage Gateway
  │   ├─ DB → DMS
  │   ├─ 서버 전체 → MGN
  │   └─ SFTP 수신 → Transfer Family
  │
  └─ 일회성 vs 지속?
      ├─ 일회성/주기 동기화 → DataSync
      └─ 지속 접근 → Storage Gateway
```

---

## 4. 서비스별 상세

### DataSync

| 항목 | 내용 |
|------|------|
| **사용 사례** | 온프레미스 → S3/EFS/FSx 마이그레이션 또는 주기 동기화 |
| **속도** | 최대 ~10 Gbps (에이전트당) |
| **기능** | 증분 동기화, 검증, 암호화, 대역폭 제한 |
| **대상** | NFS, SMB, HDFS, S3, EFS, FSx, Azure Files, Google Cloud Storage |

💡 "**데이터 마이그레이션**" (일회성/주기) → DataSync  
💡 "**지속 하이브리드 접근**" → Storage Gateway

### Snow Family

| 모델 | 용량 | 컴퓨팅 | 사용 사례 |
|------|------|-------|----------|
| **Snowcone** | 8 TB HDD / 14 TB SSD | 2 vCPU, 4 GB | **엣지/소용량**, 배낭에 들어감 |
| **Snowball Edge Storage Optimized** | **80 TB** | 40 vCPU, 80 GB | **대용량 이전** |
| **Snowball Edge Compute Optimized** | 42 TB | 52 vCPU, 208 GB + GPU | **엣지 컴퓨팅** (원격지 ML) |
| **Snowmobile** | **100 PB** | - | **초대용량** (EB급) |

💡 "**네트워크 없음/느림**" + "대용량" → Snow Family  
💡 "**엣지 컴퓨팅**" (원격지 데이터 처리) → Snowball Edge Compute

### DMS (Database Migration Service)

| 기능 | 설명 |
|------|------|
| **소스** | Oracle, MySQL, PostgreSQL, SQL Server, MongoDB, DB2 등 |
| **타겟** | RDS, Aurora, DynamoDB, S3, Redshift, OpenSearch 등 |
| **모드** | 일회성 / **CDC** (Change Data Capture, 지속 복제) |
| **복제 인스턴스** | DMS가 EC2 기반 복제 인스턴스 생성 |

**DMS 시나리오:**

| 시나리오 | 정답 |
|---------|------|
| Oracle → Aurora PostgreSQL | **DMS + SCT** (스키마 변환 + 데이터 이전) |
| MySQL → Aurora MySQL | **DMS** (동종, SCT 불필요) |
| 온프레미스 DB → S3 (분석) | **DMS** (CDC → S3) |
| 최소 다운타임 DB 이전 | **DMS CDC** (초기 풀 로드 + 변경 추적) |

### SCT (Schema Conversion Tool)

- **이기종 DB** 간 스키마/코드 변환
- 예: Oracle PL/SQL → PostgreSQL PL/pgSQL
- 💡 "Oracle → PostgreSQL/Aurora" → **SCT + DMS**

### AWS MGN (Application Migration Service)

| 항목 | 내용 |
|------|------|
| **사용 사례** | 온프레미스/다른 클라우드 **서버 전체** → EC2 (Lift-and-Shift) |
| **방식** | 블록 레벨 지속 복제 → EC2 인스턴스 변환 |
| **이전 이름** | CloudEndure Migration |
| **장점** | 최소 다운타임, 재구성 불필요 |

💡 "**서버를 그대로 AWS로**" → MGN (6R의 Rehost)

### AWS Migration Hub

- 마이그레이션 진행 상황 **중앙 추적**
- 여러 마이그레이션 도구(DMS, MGN 등) 통합 뷰

---

## 5. 시험 빈출 시나리오

### 시나리오 1: "100 TB 데이터 + 느린 네트워크"

**정답**: **Snowball Edge** (80 TB 여러 대) 또는 **Snowmobile**  
**오답 함정**: DataSync (네트워크 의존, 수개월 소요)

### 시나리오 2: "10 TB 데이터 + 빠른 네트워크"

**정답**: **DataSync** (온라인 전송, ~1~2일)  
**대안**: Direct Connect + DataSync

### 시나리오 3: "Oracle → Aurora PostgreSQL"

**정답**: **SCT** (스키마 변환) + **DMS** (데이터 이전)  
**오답 함정**: DMS만 (이기종 DB 스키마 변환 필요)

### 시나리오 4: "최소 다운타임 DB 마이그레이션"

**정답**: **DMS CDC** (초기 풀 로드 후 변경 추적 → 컷오버)  
**오답 함정**: mysqldump + 복원 (긴 다운타임)

### 시나리오 5: "수백 대 온프레미스 VM을 AWS로"

**정답**: **AWS MGN** (Lift-and-Shift)  
**대안**: VM Import/Export (소수 VM)

### 시나리오 6: "NFS 파일 서버 → S3 (일회성)"

**정답**: **DataSync** (NFS → S3)  
**대안**: Storage Gateway (지속 하이브리드인 경우)

### 시나리오 7: "외부 파트너 SFTP 파일 수신"

**정답**: **AWS Transfer Family** (SFTP → S3)  
**오답 함정**: EC2 + SFTP 서버 (운영 부담)

### 시나리오 8: "엣지 위치에서 데이터 처리 + 수집"

**정답**: **Snowball Edge Compute Optimized** (로컬 컴퓨팅 + 데이터 수집 → AWS)

### 시나리오 9: "마이그레이션 진행 상황 통합 추적"

**정답**: **AWS Migration Hub**

---

## 6. 빠른 매핑 치트표

| 키워드 | 정답 |
|--------|------|
| "파일 (NFS/SMB) → S3" (일회성/주기) | **DataSync** |
| "파일 하이브리드 (지속)" | **Storage Gateway** |
| "수 PB + 네트워크 없음" | **Snowmobile** |
| "~80 TB + 느린 네트워크" | **Snowball Edge** |
| "~8 TB + 엣지" | **Snowcone** |
| "DB 마이그레이션" | **DMS** |
| "이기종 DB 스키마" | **SCT + DMS** |
| "최소 다운타임 DB" | **DMS CDC** |
| "서버 Lift-and-Shift" | **AWS MGN** |
| "SFTP 수신" | **Transfer Family** |
| "S3 글로벌 업로드 가속" | **Transfer Acceleration** |
| "마이그레이션 추적" | **Migration Hub** |

---

## 7. 함정 / 주의사항

### 함정 1: DataSync vs Storage Gateway

- **DataSync**: **마이그레이션** (일회성/주기) — 완료 후 에이전트 제거 가능
- **Storage Gateway**: **지속 하이브리드** — 계속 운영
- 💡 "마이그레이션" → DataSync, "하이브리드" → Storage Gateway

### 함정 2: SCT는 데이터 이전 도구가 아니다

- **SCT**: **스키마 + 저장 프로시저 변환**만 담당
- **DMS**: 실제 **데이터 이전** (행 복사, CDC)
- 💡 "이기종 DB" → **둘 다** 필요

### 함정 3: Snow Family는 단방향이 아니다

- **Export from S3**도 지원 (AWS → 온프레미스)
- "S3 데이터를 온프레미스로 가져오기" → Snowball

### 함정 4: MGN vs DMS

- **MGN**: **서버 전체** (OS, 앱, DB 모두)
- **DMS**: **DB만** (서버 이전 아님)
- 💡 "VM/서버 전체" → MGN, "DB만" → DMS

### 함정 5: Transfer Acceleration 비용 주의

- S3 Transfer Acceleration은 **추가 비용** 발생 ($0.04/GB)
- 원격지 업로드 속도가 현저히 느릴 때만 권장
- 💡 대안: **Multipart Upload** (대용량 파일)

---

## 8. 검증 필요 항목 ⚠️

- [ ] Snow Family 최신 모델 (Snowcone SSD 용량)
- [ ] DMS Serverless 최신 기능
- [ ] MGN 최신 지원 소스 (온프레미스 OS 버전)
- [ ] DataSync 최신 지원 스토리지 (Google Cloud, Azure Files)
- [ ] SCT 최신 지원 DB 엔진

---

## 9. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "데이터 전송 서비스 선택" → 데이터량 + 네트워크
- **유형 2**: "DB 마이그레이션" → DMS (+ SCT 이기종)
- **유형 3**: "서버 전체 이전" → MGN
- **유형 4**: "DataSync vs Storage Gateway" → 일회성 vs 지속

---

## 10. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
