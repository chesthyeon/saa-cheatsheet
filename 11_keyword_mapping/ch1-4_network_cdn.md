# Chapter 1-4. 네트워크 및 콘텐츠 전송

## 0. 한 줄 요약

🔑 **"트래픽이 어디서 어디로, 어떤 경로로 흐르는가"가 네트워크 서비스 선택 기준이다 — VPC(격리) / Route 53(DNS) / CloudFront(CDN) / Global Accelerator(글로벌 가속) / PrivateLink(프라이빗 연결)**

---

## 1. 핵심 서비스 표

| 서비스 | 한 줄 정의 | 주요 키워드 | 가격 모델 |
|--------|-----------|------------|----------|
| **VPC** | AWS 내 논리적 네트워크 격리 | 서브넷, NACL, 보안 그룹, NAT Gateway, VPC Peering, Transit Gateway | NAT Gateway: 시간+데이터, 나머지 대부분 무료 |
| **Route 53** | 관리형 DNS + 헬스 체크 + 라우팅 정책 | 도메인 등록, Alias, 라우팅 정책 7종, 장애 조치 | 호스팅 존 + 쿼리 수 |
| **CloudFront** | 글로벌 CDN (엣지 캐시) | 엣지 로케이션, OAC, Lambda@Edge, 캐시 무효화 | 데이터 전송 + 요청 수 |
| **Global Accelerator** | AWS 글로벌 네트워크 경유 가속 | Anycast IP, 엔드포인트 그룹, 헬스 체크, 장애 조치 | 고정 IP + 데이터 전송 가속 |
| **PrivateLink** | VPC 간 프라이빗 연결 (인터넷 비통과) | Interface VPC Endpoint, 서비스 노출, ENI | 엔드포인트 시간 + 데이터 처리 |
| **Direct Connect** | 온프레미스 ↔ AWS 전용 물리 회선 | 1 Gbps / 10 Gbps, 일관된 지연, 하이브리드 | 포트 시간 + 데이터 전송 |
| **Site-to-Site VPN** | 온프레미스 ↔ AWS 암호화 터널 (인터넷 경유) | IPsec, Virtual Private Gateway, Transit Gateway | 연결 시간 + 데이터 전송 |
| **Transit Gateway** | 허브-스포크 네트워크 연결 허브 | 다수 VPC/VPN/Direct Connect 중앙 연결 | 연결 수 + 데이터 처리 |

---

## 2. 키워드 → 서비스 매핑

### VPC 핵심 구성 요소 매핑

| 시험 키워드 | 정답 구성 요소 |
|------------|--------------|
| "인터넷 접근 가능한 서브넷" | **퍼블릭 서브넷** (Internet Gateway 연결) |
| "인터넷 접근 불가, 내부만" | **프라이빗 서브넷** |
| "프라이빗 서브넷에서 인터넷 아웃바운드" | **NAT Gateway** (관리형) 또는 NAT Instance |
| "인스턴스 레벨 방화벽" | **보안 그룹** (Stateful, 허용만 설정) |
| "서브넷 레벨 방화벽" | **NACL** (Stateless, 허용+거부 설정) |
| "VPC 간 직접 연결" (2개) | **VPC Peering** (전이적 라우팅 불가) |
| "다수 VPC 중앙 연결" | **Transit Gateway** |
| "AWS 서비스에 프라이빗 접근" | **VPC Endpoint** (Gateway: S3/DynamoDB, Interface: 나머지) |

### Route 53 라우팅 정책 매핑

| 시험 키워드 | 정답 라우팅 정책 |
|------------|---------------|
| "단순 연결", "단일 리소스" | **Simple** |
| "가중치 기반 트래픽 분배" | **Weighted** |
| "사용자 위치 기반 (대륙/국가)" | **Geolocation** |
| "가장 가까운 리전" (지연 시간 기준) | **Latency-based** |
| "장애 조치, Active-Passive" | **Failover** |
| "여러 값 응답 + 헬스 체크" | **Multivalue Answer** |
| "특정 IP 범위 → 특정 엔드포인트" | **Geoproximity** (Traffic Flow) |

> 💡 **시험 팁**: "지연 시간 최소화" → Latency, "국가별 콘텐츠" → Geolocation, "DR" → Failover

### CloudFront가 정답인 키워드

| 시험 키워드 | 왜 CloudFront인가 |
|------------|-----------------|
| "정적 콘텐츠 가속", "글로벌 캐시" | 엣지 로케이션에서 콘텐츠 캐싱 |
| "S3 + 글로벌 배포" | CloudFront + S3 Origin |
| "DDoS 방어" + CDN | CloudFront + AWS Shield |
| "HTTPS 강제", "SSL/TLS 인증서" | CloudFront + ACM 인증서 |
| "S3 직접 접근 차단" | **OAC** (Origin Access Control) |
| "엣지에서 요청 변환" | **Lambda@Edge** 또는 **CloudFront Functions** |
| "동적+정적 콘텐츠 모두 가속" | CloudFront (동적도 지원) |

### Global Accelerator가 정답인 키워드

| 시험 키워드 | 왜 Global Accelerator인가 |
|------------|-------------------------|
| "고정 Anycast IP" | 글로벌 진입점 IP가 변하지 않음 |
| "TCP/UDP 트래픽 가속" (비HTTP 포함) | CloudFront는 HTTP/HTTPS 중심 |
| "글로벌 장애 조치" + "자동 엔드포인트 전환" | 헬스 체크 기반 자동 리전 전환 |
| "IP 화이트리스트 필요" + "글로벌" | 고정 IP로 방화벽 규칙 유지 |
| "게임 서버", "IoT" + "글로벌 가속" | 비HTTP 프로토콜 지원 |

> ⚠️ **CloudFront vs Global Accelerator**: HTTP/HTTPS 캐싱 → CloudFront, TCP/UDP 가속 + 고정 IP → Global Accelerator

### PrivateLink가 정답인 키워드

| 시험 키워드 | 왜 PrivateLink인가 |
|------------|------------------|
| "인터넷 통과하지 않고 AWS 서비스 접근" | Interface VPC Endpoint |
| "VPC 간 서비스 노출" (프라이빗) | PrivateLink + NLB |
| "SaaS 서비스 프라이빗 접근" | PrivateLink 기반 SaaS 연결 |
| "보안 규정: 인터넷 트래픽 금지" | 모든 통신을 AWS 네트워크 내부로 |

### 온프레미스 ↔ AWS 연결 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "빠른 설정", "암호화", "인터넷 경유" | **Site-to-Site VPN** |
| "일관된 지연", "전용 회선", "높은 대역폭" | **Direct Connect** |
| "Direct Connect 백업" | **Site-to-Site VPN** (보조 경로) |
| "Direct Connect + 암호화" | Direct Connect + **VPN over Direct Connect** |
| "다수 VPC + 온프레미스 중앙 연결" | **Transit Gateway** + Direct Connect/VPN |

---

## 3. 헷갈리는 포인트 / 함정

### 함정 1: 보안 그룹(Stateful) vs NACL(Stateless)

- ✅ **보안 그룹**: Stateful → 인바운드 허용하면 아웃바운드 자동 허용, **허용 규칙만**
- ✅ **NACL**: Stateless → 인바운드/아웃바운드 각각 규칙 필요, **허용+거부 규칙**
- 💡 "특정 IP 차단" → NACL (보안 그룹은 거부 규칙 없음)

### 함정 2: VPC Peering은 전이적 라우팅 불가

- ❌ VPC A↔B 피어링, B↔C 피어링 → A↔C 통신 가능 → **틀림**
- ✅ VPC Peering은 **1:1 직접 연결만**, 전이(Transitive) 불가
- 💡 "다수 VPC 상호 연결" → **Transit Gateway** (허브-스포크)

### 함정 3: NAT Gateway vs NAT Instance

- ✅ **NAT Gateway**: 관리형, 고가용성, 자동 확장 → **시험 정답은 대부분 이것**
- ✅ **NAT Instance**: EC2 기반, 직접 관리, 보안 그룹 연결 가능
- 💡 "운영 오버헤드 최소화" → NAT Gateway, "포트 포워딩 필요" → NAT Instance

### 함정 4: CloudFront vs Global Accelerator

- ❌ "글로벌 가속"이면 무조건 CloudFront → **프로토콜에 따라 다름**
- ✅ CloudFront: HTTP/HTTPS 콘텐츠 캐싱 + 가속
- ✅ Global Accelerator: TCP/UDP 모든 프로토콜 가속, 고정 IP, 캐싱 없음
- 💡 "캐싱" → CloudFront, "고정 IP" 또는 "비HTTP" → Global Accelerator

### 함정 5: VPC Endpoint Gateway vs Interface

- ✅ **Gateway Endpoint**: S3, DynamoDB만 지원, **무료**, 라우트 테이블에 추가
- ✅ **Interface Endpoint**: 대부분의 AWS 서비스, **유료** (ENI 기반, PrivateLink)
- 💡 "S3에 프라이빗 접근" → Gateway Endpoint (비용 0), 나머지 → Interface

### 함정 6: Direct Connect 설치 시간

- ❌ "즉시 연결 필요" → Direct Connect → **틀림** (설치에 수 주~수 개월)
- ✅ 즉시 필요 → **Site-to-Site VPN** (몇 분 내 설정)
- ✅ Direct Connect: 장기적 안정적 연결, 설치 기간 긴 점 감안
- 💡 "빠른 설정" → VPN, "일관된 성능" + 시간 여유 → Direct Connect

---

## 4. 단골 시나리오

### 시나리오 1: 정적 웹사이트 글로벌 배포

```
문제 패턴: "S3 정적 웹사이트, 전 세계 사용자, HTTPS"
정답 패턴: S3 + CloudFront + ACM + Route 53
키 포인트: OAC로 S3 직접 접근 차단, ACM으로 HTTPS
```

### 시나리오 2: 멀티 리전 장애 조치

```
문제 패턴: "리전 장애 시 자동으로 다른 리전으로 전환"
정답 패턴: Route 53 Failover 라우팅 + 헬스 체크
대안: Global Accelerator 엔드포인트 그룹 장애 조치
```

### 시나리오 3: 하이브리드 클라우드 네트워크

```
문제 패턴: "온프레미스 데이터센터와 AWS 안정적 연결, 다수 VPC"
정답 패턴: Direct Connect + Transit Gateway
키 포인트: Direct Connect 단독은 암호화 없음 → 암호화 필요 시 VPN 추가
```

### 시나리오 4: 프라이빗 서브넷에서 S3 접근

```
문제 패턴: "인터넷 없이 프라이빗 서브넷에서 S3 접근"
정답 패턴: S3 Gateway VPC Endpoint (무료)
함정: Interface Endpoint도 가능하지만 유료 → 비용 최적화면 Gateway
```

### 시나리오 5: 게임 서버 글로벌 가속

```
문제 패턴: "TCP/UDP 게임 트래픽, 글로벌 사용자, 고정 IP"
정답 패턴: Global Accelerator
키 포인트: "비HTTP" + "고정 IP" → CloudFront가 아닌 Global Accelerator
```

---

## 5. 검증 필요 항목 ⚠️

- [ ] NAT Gateway AZ당 처리량 한도 (현재 100 Gbps?) 확인
  - 공식 문서: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html
- [ ] CloudFront OAC vs OAI (레거시) 최신 권장 사항 확인
- [ ] Global Accelerator 최신 요금 체계 확인
- [ ] Transit Gateway 최대 연결 VPC 수 한도 확인
- [ ] Direct Connect 최신 대역폭 옵션 (100 Gbps 지원 여부)
- [ ] Route 53 Geoproximity vs Geolocation 최신 동작 차이 확인
- [ ] VPC Endpoint Gateway 지원 서비스 (S3, DynamoDB 외 추가 여부)

---

## 6. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "보안 그룹 vs NACL" → Stateful/Stateless 구분
- **유형 2**: "다수 VPC 연결" → Transit Gateway vs VPC Peering
- **유형 3**: "글로벌 콘텐츠 캐싱" → CloudFront
- **유형 4**: "TCP/UDP 글로벌 가속 + 고정 IP" → Global Accelerator
- **유형 5**: "온프레미스 ↔ AWS 연결" → Direct Connect vs VPN
- **유형 6**: "DNS 장애 조치" → Route 53 Failover
- **유형 7**: "프라이빗 서브넷 → AWS 서비스" → VPC Endpoint
- **유형 8**: "특정 IP 차단" → NACL
- **유형 9**: "인터넷 없이 S3 접근" → S3 Gateway Endpoint
- **유형 10**: "Route 53 라우팅 정책 선택" → 키워드 기반 정책 매핑

---

## 7. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
