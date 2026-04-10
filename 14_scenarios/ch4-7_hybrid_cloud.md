# Chapter 4-7. 하이브리드 클라우드 (Direct Connect + Transit Gateway + Storage Gateway)

## 0. 한 줄 요약

🔑 **"온프레미스 ↔ AWS 연결" = 네트워크(VPN/DX/TGW) + 스토리지(Storage Gateway/DataSync) + ID(AD/IAM Identity Center) 세 축으로 구성된다 — 대역폭·보안·비용 요구사항이 서비스를 결정**

---

## 1. 아키텍처 다이어그램

```
┌─── 온프레미스 데이터센터 ───┐
│                              │
│  앱 서버 / DB / 파일 서버     │
│  Active Directory             │
│  Storage Gateway (VM)         │
│                              │
│  ┌─ 연결 옵션 ─────────┐    │
│  │ ① Site-to-Site VPN  │────│──→ VPN Gateway ──→ VPC
│  │ ② Direct Connect    │────│──→ DX Gateway ──→ VPC / TGW
│  │ ③ VPN over DX       │────│──→ DX + VPN (암호화)
│  └─────────────────────┘    │
└──────────────────────────────┘
                                         │
                              ┌──────────┼──────────┐
                              ▼          ▼          ▼
                           VPC-A      VPC-B      VPC-C
                              └──────────┼──────────┘
                                   Transit Gateway
                                   (중앙 허브)
```

---

## 2. 네트워크 연결 옵션 비교

| 기준 | Site-to-Site VPN | Direct Connect (DX) | DX + VPN |
|------|-----------------|--------------------|---------| 
| **연결 방식** | 인터넷 경유 (IPsec 터널) | **전용 물리 회선** | DX 위에 VPN 터널 |
| **대역폭** | 인터넷 속도 의존 (~1.25 Gbps) | **1 Gbps / 10 Gbps / 100 Gbps** | DX 대역폭 |
| **지연** | 변동 (인터넷) | **일관된 저지연** | 일관된 저지연 |
| **암호화** | ✅ IPsec (기본) | ❌ 기본 미암호화 | ✅ **IPsec over DX** |
| **설정 시간** | **분~시간** | **수주~수개월** | 수주~수개월 |
| **이중화** | 2개 터널 (기본) | 2개 DX 연결 권장 | DX 이중화 + VPN |
| **비용** | 저렴 (시간당 + 데이터) | 고가 (포트 시간 + 데이터 OUT) | 가장 비쌈 |
| **시험 키워드** | "빠른 설정", "암호화", "임시" | "안정적", "대용량", "전용선" | "DX + 암호화" |

### 연결 방식 판별 플로우차트

```
"온프레미스 ↔ AWS 연결"
  │
  ├─ "빠르게 연결" / "임시" / "백업 연결"
  │   └─ Site-to-Site VPN
  │
  ├─ "안정적" / "대용량" / "일관된 지연"
  │   └─ Direct Connect
  │
  ├─ "DX + 암호화 필요"
  │   └─ DX 위에 Site-to-Site VPN
  │
  └─ "DX 설치 전 임시 연결"
      └─ VPN 먼저 → DX 완료 후 전환
```

---

## 3. Transit Gateway (TGW) 상세

### 역할

- **중앙 허브**: 여러 VPC + VPN + DX를 하나의 허브로 연결
- Full Mesh 대신 Hub-and-Spoke 토폴로지

### TGW 없이 vs 있을 때

```
[TGW 없이] N개 VPC = N(N-1)/2 피어링     [TGW 있을 때] 모든 VPC → TGW 연결
VPC-A ←→ VPC-B                              VPC-A ──┐
VPC-A ←→ VPC-C                              VPC-B ──┼── TGW
VPC-B ←→ VPC-C                              VPC-C ──┘
(3개 피어링)                                 (3개 연결, 중앙 관리)
```

### TGW 핵심 기능

| 기능 | 설명 | 시험 키워드 |
|------|------|-----------|
| **라우트 테이블** | VPC 간 트래픽 제어 | "VPC 간 격리", "공유 서비스" |
| **멀티캐스트** | TGW 멀티캐스트 도메인 | "멀티캐스트" |
| **Inter-Region Peering** | TGW 간 리전 연결 | "멀티 리전 네트워크" |
| **RAM 공유** | 다른 계정과 TGW 공유 | "Cross-Account VPC 연결" |

### TGW 시험 판별

| 키워드 | 정답 |
|--------|------|
| "다수 VPC 연결" (3개 이상) | **Transit Gateway** |
| "VPC 2개 연결" | **VPC Peering** (더 간단) |
| "온프레미스 + 다수 VPC" | **TGW + DX Gateway** |
| "VPC 간 전이적 라우팅" | **TGW** (Peering은 전이적 ❌) |

---

## 4. Storage Gateway 상세

### 유형 비교

| 유형 | 프로토콜 | 사용 사례 | 백엔드 스토리지 | 시험 키워드 |
|------|---------|----------|---------------|-----------|
| **S3 File Gateway** | NFS / SMB | 파일 공유 → S3 | **S3** | "파일 서버 → S3", "NFS" |
| **FSx File Gateway** | SMB | Windows 파일 → FSx | **FSx for Windows** | "Windows 파일 서버", "SMB" |
| **Volume Gateway (Cached)** | iSCSI | 블록 스토리지 (자주 사용 = 로컬 캐시) | **S3 + EBS 스냅샷** | "자주 접근 데이터 로컬 캐시" |
| **Volume Gateway (Stored)** | iSCSI | 블록 스토리지 (전체 로컬 저장) | **로컬 + S3 백업** | "전체 로컬 저장 + S3 백업" |
| **Tape Gateway** | iSCSI VTL | 테이프 백업 대체 | **S3 Glacier** | "테이프 백업", "VTL" |

### Storage Gateway 판별 플로우차트

```
"온프레미스 → AWS 스토리지 통합"
  │
  ├─ 파일 기반 (NFS/SMB)?
  │   ├─ Linux (NFS) 또는 범용 → S3 File Gateway
  │   └─ Windows (SMB) + AD 통합 → FSx File Gateway
  │
  ├─ 블록 기반 (iSCSI)?
  │   ├─ 자주 접근 데이터 캐시 → Volume Gateway (Cached)
  │   └─ 전체 로컬 유지 + 백업 → Volume Gateway (Stored)
  │
  └─ 테이프 백업 대체?
      └─ Tape Gateway
```

---

## 5. 데이터 전송 서비스 비교

| 서비스 | 유형 | 사용 사례 | 시험 키워드 |
|--------|------|----------|-----------|
| **DataSync** | 온라인 전송 (에이전트) | 온프레미스 → S3/EFS/FSx **마이그레이션** | "데이터 이전", "일회성/주기적 동기화" |
| **Storage Gateway** | 하이브리드 스토리지 | 온프레미스 앱이 **지속적으로** AWS 스토리지 사용 | "하이브리드", "지속적 접근" |
| **Transfer Family** | SFTP/FTPS/FTP → S3 | 외부 파트너 파일 수신 | "SFTP", "파트너 파일 전송" |
| **Snow Family** | 오프라인 물리 전송 | 대용량 (TB~PB), 네트워크 제한 | "대용량", "네트워크 불가" |

**DataSync vs Storage Gateway 핵심 구분:**

| 기준 | DataSync | Storage Gateway |
|------|---------|----------------|
| 목적 | **데이터 이전** (마이그레이션) | **하이브리드 접근** (지속 사용) |
| 패턴 | 일회성 / 스케줄 동기화 | 상시 연결 |
| 프로토콜 | 에이전트 기반 (최적화 전송) | NFS/SMB/iSCSI |
| 완료 후 | 에이전트 제거 가능 | **계속 운영** |

💡 "마이그레이션" → DataSync, "하이브리드 지속 사용" → Storage Gateway

---

## 6. 시험 빈출 변형 시나리오

### 시나리오 1: 안정적 온프레미스 연결

> "온프레미스 데이터센터와 AWS를 안정적으로 연결. 대용량 데이터 전송."

**정답**: **Direct Connect**  
**이중화**: DX 2개 또는 DX + VPN (백업)  
**오답 함정**: VPN만 (인터넷 의존, 대역폭 불안정)

### 시나리오 2: DX 설치 전 임시 연결

> "DX를 주문했지만 설치에 수주 걸린다. 당장 연결이 필요하다."

**정답**: **Site-to-Site VPN** (즉시 설정) → DX 완료 후 전환  
**오답 함정**: DX 기다리기 (비즈니스 지연)

### 시나리오 3: DX + 암호화

> "Direct Connect로 연결하되, 전송 중 데이터 암호화가 필요하다."

**정답**: **DX 위에 Site-to-Site VPN** (IPsec 암호화)  
**오답 함정**: DX만 (기본 미암호화)  
**대안**: MACsec (DX 전용 L2 암호화, 10/100 Gbps 포트)

### 시나리오 4: 다수 VPC + 온프레미스 연결

> "10개 VPC와 온프레미스를 모두 연결해야 한다."

**정답**: **Transit Gateway** + DX Gateway  
**오답 함정**: VPC Peering 45개 (N(N-1)/2, 관리 불가)

### 시나리오 5: 온프레미스 파일 서버 → S3

> "온프레미스 NFS 파일 서버의 데이터를 S3에서 접근 가능하게."

**정답**: **S3 File Gateway** (NFS → S3, 로컬 캐시)  
**마이그레이션만**: **DataSync** (일회성 전송)  
**오답 함정**: S3 직접 업로드 (앱 수정 필요)

### 시나리오 6: 테이프 백업 클라우드 전환

> "온프레미스 테이프 백업을 클라우드로 전환. 기존 백업 소프트웨어 유지."

**정답**: **Tape Gateway** (VTL → S3 Glacier)  
**오답 함정**: S3 직접 (기존 백업 소프트웨어 호환 ❌)

### 시나리오 7: Windows 파일 서버 통합

> "온프레미스 Windows 파일 서버를 AWS로 확장. Active Directory 통합 필요."

**정답**: **FSx File Gateway** (SMB + AD) 또는 **FSx for Windows File Server**  
**오답 함정**: S3 File Gateway (SMB 지원하지만 AD 통합은 FSx가 적합)

### 시나리오 8: DX 이중화

> "Direct Connect의 고가용성을 확보해야 한다."

**정답 옵션:**
1. **2개 DX** (다른 DX 로케이션) — 최고 가용성
2. **DX + VPN** (백업) — 비용 절감
3. **DX LAG** (Link Aggregation Group) — 같은 로케이션 대역폭 결합

---

## 7. 핵심 키워드 → 서비스 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "안정적/전용선/대용량 연결" | **Direct Connect** |
| "빠른 설정/임시/암호화 연결" | **Site-to-Site VPN** |
| "DX + 암호화" | **VPN over DX** |
| "다수 VPC 중앙 연결" | **Transit Gateway** |
| "NFS/파일 → S3" | **S3 File Gateway** |
| "Windows SMB + AD" | **FSx File Gateway** |
| "테이프 백업 대체" | **Tape Gateway** |
| "데이터 마이그레이션 (일회성)" | **DataSync** |
| "하이브리드 스토리지 (지속)" | **Storage Gateway** |
| "SFTP 파일 수신" | **Transfer Family** |

---

## 8. 헷갈리는 포인트 / 함정

### 함정 1: VPN 대역폭 한계

- Site-to-Site VPN: 터널당 최대 **1.25 Gbps**
- 여러 VPN 터널 = ECMP (TGW에서 지원, VGW는 미지원)
- 💡 "10 Gbps 이상" → VPN ❌ → **Direct Connect**

### 함정 2: DX Gateway vs TGW

- **DX Gateway**: DX 연결을 여러 리전의 VGW에 연결 (VPC 간 라우팅 ❌)
- **TGW**: VPC 간 라우팅 ✅ + DX 연결 가능
- 💡 "VPC 간 통신 필요" → TGW, "온프레미스 → 여러 리전 VPC" → DX Gateway + TGW

### 함정 3: Volume Cached vs Stored

- **Cached**: 자주 사용 데이터만 로컬, 전체는 S3 → **로컬 스토리지 절약**
- **Stored**: 전체 데이터 로컬, S3에 비동기 백업 → **낮은 지연**
- 💡 "로컬 스토리지 절약" → Cached, "전체 로컬 접근" → Stored

### 함정 4: PrivateLink vs VPN/DX

- **PrivateLink**: 특정 서비스 접근 (VPC 간 또는 AWS 서비스)
- **VPN/DX**: 네트워크 전체 연결
- 💡 "특정 서비스만 노출" → PrivateLink, "전체 네트워크 연결" → VPN/DX

---

## 9. 검증 필요 항목 ⚠️

- [ ] DX MACsec 지원 포트 (10 Gbps, 100 Gbps 확인)
- [ ] TGW 최대 연결 수 (5,000 VPC?)
- [ ] Storage Gateway 최신 VM 요구사항
- [ ] DataSync 최신 전송 속도 (10 Gbps?)
- [ ] DX 최신 요금 (포트 시간 + 데이터 전송)
- [ ] Snow Family 최신 모델 (Snowcone SSD?)

---

## 10. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "온프레미스 연결" → VPN vs DX 판별
- **유형 2**: "다수 VPC 연결" → TGW
- **유형 3**: "파일 서버 → S3" → Storage Gateway 유형 선택
- **유형 4**: "데이터 마이그레이션 vs 하이브리드" → DataSync vs Storage Gateway
- **유형 5**: "DX 이중화" → 2개 DX / DX + VPN

---

## 11. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
