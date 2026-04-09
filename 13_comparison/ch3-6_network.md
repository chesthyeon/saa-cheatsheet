# Chapter 3-6. 네트워크: VPN vs Direct Connect vs Transit Gateway vs VPC Peering vs PrivateLink

## 0. 한 줄 요약

🔑 **"어디와 어디를 연결하는가(온프레미스↔AWS / VPC↔VPC / VPC↔서비스)" × "성능·보안·비용 요구"가 네트워크 연결 선택 기준이다**

---

## 1. 전체 비교 매트릭스

| 기준 | Site-to-Site VPN | Direct Connect | Transit Gateway | VPC Peering | PrivateLink |
|------|-----------------|---------------|----------------|-------------|-------------|
| **연결 대상** | 온프레미스 ↔ AWS | 온프레미스 ↔ AWS | **다수 VPC/VPN/DX 허브** | VPC ↔ VPC (1:1) | VPC → AWS 서비스/다른 VPC 서비스 |
| **경로** | **인터넷** (암호화) | **전용 물리 회선** | AWS 네트워크 | AWS 네트워크 | AWS 네트워크 (ENI) |
| **암호화** | ✅ IPsec | ❌ (기본 미암호화) | VPN 연결 시 ✅ | ❌ (AWS 내부) | ❌ (AWS 내부) |
| **대역폭** | ~1.25 Gbps | 1 / 10 / 100 Gbps | 최대 50 Gbps | 제한 없음 (VPC 대역폭) | 제한 없음 |
| **지연 시간** | 변동 (인터넷 경유) | **일관적·저지연** | 낮음 | 낮음 | 낮음 |
| **설치 시간** | **분~시간** | **수 주~수 개월** | 분~시간 | 분 | 분 |
| **이중화** | 2개 터널 자동 | 별도 구성 필요 | 자동 | 자동 | 자동 |
| **전이적 라우팅** | ❌ | ❌ | ✅ **허브-스포크** | ❌ | ❌ |
| **비용** | 연결 시간 + 전송 | 포트 시간 + 전송 | 연결 수 + 전송 | 전송만 | 엔드포인트 시간 + 전송 |

---

## 2. 핵심 판별 플로우차트

```
"네트워크 연결 선택"
  │
  ├─ 온프레미스 ↔ AWS?
  │   ├─ 빠른 설정 / 백업 경로? ──→ Site-to-Site VPN
  │   ├─ 일관된 성능 / 대용량? ──→ Direct Connect
  │   ├─ DX + 암호화? ──→ Direct Connect + VPN over DX
  │   └─ 다수 VPC + 온프레미스? ──→ Transit Gateway + DX/VPN
  │
  ├─ VPC ↔ VPC?
  │   ├─ 2개 VPC 직접 연결? ──→ VPC Peering
  │   └─ 다수 VPC 상호 연결? ──→ Transit Gateway
  │
  └─ VPC → AWS 서비스 (프라이빗)?
      ├─ S3 / DynamoDB? ──→ Gateway VPC Endpoint (무료)
      └─ 그 외 서비스? ──→ Interface VPC Endpoint (PrivateLink)
```

---

## 3. 쌍별 상세 비교

### Site-to-Site VPN vs Direct Connect

| 기준 | VPN | Direct Connect |
|------|-----|---------------|
| 경로 | 인터넷 (암호화) | 전용 물리 회선 |
| 설치 | **분~시간** (즉시 사용) | **수 주~수 개월** |
| 대역폭 | ~1.25 Gbps | **1 / 10 / 100 Gbps** |
| 지연 | 변동 (인터넷 품질 의존) | **일관적·저지연** |
| 암호화 | ✅ 기본 IPsec | ❌ (별도 VPN 추가 필요) |
| 비용 | 저렴 | 높음 (포트 비용 + 전용선) |
| 이중화 | 2개 터널 자동 | 2개 DX 연결 별도 구성 권장 |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "빠른 설정", "즉시 연결" | **VPN** |
| "일관된 성능", "대용량 전송", "전용 회선" | **Direct Connect** |
| "Direct Connect 백업" | **VPN** (보조 경로) |
| "DX + 암호화 필요" | **Direct Connect + VPN over DX** |

### VPC Peering vs Transit Gateway

| 기준 | VPC Peering | Transit Gateway |
|------|-------------|----------------|
| 연결 구조 | **1:1 직접 연결** | **허브-스포크** (중앙 허브) |
| 전이적 라우팅 | ❌ (A↔B, B↔C → A↔C 불가) | ✅ (허브 경유 모두 통신) |
| VPC 수 증가 | 연결 수 = N×(N-1)/2 (폭발적) | 연결 수 = N (선형) |
| 비용 | 데이터 전송만 | 연결당 + 데이터 처리 |
| 리전 간 | ✅ (Inter-Region Peering) | ✅ (Inter-Region Peering) |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "2개 VPC 직접 연결" | **VPC Peering** |
| "다수 VPC (3개 이상) 상호 연결" | **Transit Gateway** |
| "온프레미스 + 다수 VPC 중앙 관리" | **Transit Gateway** |
| "전이적 라우팅 필요" | **Transit Gateway** |

### PrivateLink vs VPC Peering

| 기준 | PrivateLink | VPC Peering |
|------|-------------|-------------|
| 목적 | **특정 서비스만 프라이빗 노출** | **전체 VPC 네트워크 연결** |
| 노출 범위 | 서비스 단위 (NLB 뒤 서비스) | VPC 전체 CIDR |
| 방향 | 단방향 (소비자 → 제공자) | 양방향 |
| CIDR 겹침 | ✅ 허용 (ENI 기반) | ❌ CIDR 겹치면 불가 |
| 사용 사례 | SaaS 서비스 노출, AWS 서비스 프라이빗 접근 | VPC 간 전체 통신 |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "인터넷 없이 AWS 서비스 접근" | **PrivateLink (Interface Endpoint)** |
| "특정 서비스만 다른 VPC에 노출" | **PrivateLink** |
| "VPC 전체 통신" | **VPC Peering** |
| "CIDR 겹침" | **PrivateLink** (Peering 불가) |

### Gateway Endpoint vs Interface Endpoint

| 기준 | Gateway Endpoint | Interface Endpoint |
|------|-----------------|-------------------|
| 지원 서비스 | **S3, DynamoDB만** | **대부분 AWS 서비스** |
| 구현 방식 | 라우트 테이블 항목 추가 | ENI 생성 (서브넷에) |
| 비용 | **무료** | 유료 (시간 + 데이터) |
| 접근 범위 | 같은 리전만 | 같은 리전 (Cross-Region은 Peering 경유) |

> 💡 "S3 프라이빗 접근" + "비용 최적화" → **Gateway Endpoint** (무료)

---

## 4. 시험 빈출 시나리오 + 정답

| 시나리오 | 정답 | 오답 함정 |
|----------|------|----------|
| 온프레미스 ↔ AWS 즉시 연결 | **Site-to-Site VPN** | Direct Connect (수 주 소요) |
| 온프레미스 ↔ AWS 안정적 대용량 | **Direct Connect** | VPN (대역폭 부족) |
| DX + 데이터 암호화 | **DX + VPN over DX** | DX만 (암호화 없음) |
| 50개 VPC + 온프레미스 중앙 관리 | **Transit Gateway** | VPC Peering (1,225개 연결 필요) |
| 프라이빗 서브넷 → S3 (비용 최적화) | **S3 Gateway Endpoint** | Interface Endpoint (유료) |
| SaaS 서비스 프라이빗 노출 | **PrivateLink** | VPC Peering (전체 노출) |
| 2개 VPC 간 단순 통신 | **VPC Peering** | Transit Gateway (과도) |
| CIDR 겹치는 VPC 간 서비스 공유 | **PrivateLink** | Peering (CIDR 겹침 불가) |

---

## 5. 검증 필요 항목 ⚠️

- [ ] Direct Connect 100 Gbps 지원 리전 및 가격
- [ ] Transit Gateway 최대 VPC 연결 수 (기본 5,000?)
- [ ] VPC Peering 최대 연결 수 한도 (리전당)
- [ ] Gateway Endpoint 지원 서비스 (S3, DynamoDB 외 추가?)
- [ ] PrivateLink 최신 지원 서비스 목록
- [ ] Transit Gateway Inter-Region Peering 최신 대역폭 한도

---

## 6. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "온프레미스 연결" → VPN vs Direct Connect
- **유형 2**: "다수 VPC 연결" → Transit Gateway vs Peering
- **유형 3**: "프라이빗 서비스 접근" → PrivateLink vs Peering
- **유형 4**: "S3 프라이빗 접근" → Gateway vs Interface Endpoint
- **유형 5**: "DX + 암호화" → DX + VPN over DX

---

## 7. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
