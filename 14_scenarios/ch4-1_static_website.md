# Chapter 4-1. 정적 웹사이트 호스팅 (S3 + CloudFront + Route 53 + ACM)

## 0. 한 줄 요약

🔑 **"정적 콘텐츠 호스팅" = S3 Origin + CloudFront CDN + Route 53 DNS + ACM HTTPS — 서버 없이 글로벌 배포하는 SAA 시험 단골 1번 아키텍처**

---

## 1. 아키텍처 다이어그램

```
사용자 (브라우저)
  │
  ▼
Route 53 (DNS: example.com → CloudFront)
  │
  ▼
CloudFront (CDN, 엣지 로케이션)
  │  ├─ ACM 인증서 (HTTPS, us-east-1 필수)
  │  └─ OAC (Origin Access Control)
  │
  ▼
S3 버킷 (정적 웹사이트: HTML/CSS/JS/이미지)
  └─ 퍼블릭 접근 차단 (OAC 통해서만 접근)
```

---

## 2. 구성 요소별 역할 + 키워드 매핑

| 구성 요소 | 역할 | 시험 키워드 |
|----------|------|-----------|
| **S3** | 정적 파일 저장 (Origin) | "정적 웹사이트", "HTML/CSS/JS" |
| **CloudFront** | 글로벌 캐싱 + HTTPS 종료 | "지연 시간 감소", "글로벌 배포", "CDN" |
| **Route 53** | DNS 라우팅 (도메인 → CloudFront) | "커스텀 도메인", "Alias 레코드" |
| **ACM** | SSL/TLS 인증서 (무료) | "HTTPS", "인증서 자동 갱신" |
| **OAC** | S3 직접 접근 차단, CloudFront만 허용 | "S3 퍼블릭 접근 차단", "보안" |

---

## 3. 단계별 구성 순서

### Step 1: S3 버킷 생성

- 버킷 이름: 도메인과 동일할 필요 없음 (CloudFront가 라우팅)
- **S3 정적 웹사이트 호스팅 활성화**: 시험에서는 활성화가 정답일 때도 있지만, CloudFront + OAC 사용 시 **불필요**
- **퍼블릭 접근 차단**: ✅ Block all public access 유지
- 💡 S3 웹사이트 엔드포인트 vs S3 REST API 엔드포인트 구분 필요

### Step 2: CloudFront 배포 생성

- **Origin**: S3 버킷 (REST API 엔드포인트)
- **OAC (Origin Access Control)** 설정 → S3 버킷 정책 자동 업데이트
- **Default Root Object**: `index.html`
- **Viewer Protocol Policy**: Redirect HTTP to HTTPS
- **캐시 동작**: TTL 설정 (정적 자산은 긴 TTL)

### Step 3: ACM 인증서 발급

- ⚠️ **반드시 us-east-1 (N. Virginia)에서 발급** (CloudFront 요구사항)
- 검증 방식: DNS 검증 (Route 53 자동 CNAME 추가) 또는 이메일 검증
- 자동 갱신: ✅

### Step 4: Route 53 설정

- **Alias 레코드** (A/AAAA): 도메인 → CloudFront 배포
- 💡 CNAME이 아닌 **Alias** 사용 (Zone Apex = naked domain 지원)

---

## 4. 시험 빈출 변형 시나리오

### 시나리오 1: 기본 정적 웹사이트

> "회사가 정적 웹사이트를 최소 비용으로 호스팅하려 한다. 글로벌 사용자에게 빠르게 제공해야 한다."

**정답**: S3 + CloudFront + Route 53 + ACM  
**오답 함정**: EC2 + ALB (과도한 인프라, 서버 관리 필요)

### 시나리오 2: S3 직접 접근 차단

> "S3에 호스팅된 웹사이트가 있다. S3 URL로 직접 접근을 차단하고 CloudFront만 허용하려 한다."

**정답**: **OAC** (Origin Access Control) + S3 Bucket Policy  
**오답 함정**: OAI (Origin Access Identity) — 레거시, OAC가 권장  
**오답 함정**: S3 퍼블릭 접근 허용 (보안 위반)

### 시나리오 3: HTTPS 적용

> "CloudFront 배포에 커스텀 도메인으로 HTTPS를 설정해야 한다."

**정답**: **ACM (us-east-1)** + CloudFront 대체 도메인(CNAME) 설정  
**오답 함정**: ACM을 다른 리전에서 발급 (CloudFront는 us-east-1만 허용)

### 시나리오 4: SPA (Single Page Application) 라우팅

> "React SPA를 S3+CloudFront로 호스팅. /about 등 서브 경로에서 404 에러 발생."

**정답**: CloudFront **Custom Error Response** — 403/404 → `/index.html` (200) 반환  
**오답 함정**: S3 리다이렉트 규칙 (SPA 라우팅에 부적합)

### 시나리오 5: 콘텐츠 업데이트 반영

> "S3에 새 파일을 업로드했지만 CloudFront에서 이전 버전이 보인다."

**정답**: **CloudFront Invalidation** (`/*` 또는 특정 경로)  
**대안**: 파일명에 버전 포함 (캐시 버스팅: `app.v2.js`)  
**오답 함정**: S3 버전 관리 활성화 (캐시 문제 해결 안 됨)

### 시나리오 6: Zone Apex (naked domain) 설정

> "example.com (www 없이)을 CloudFront로 연결해야 한다."

**정답**: Route 53 **Alias 레코드** (A 타입)  
**오답 함정**: CNAME (Zone Apex에 CNAME 사용 불가 — DNS 표준)

### 시나리오 7: 다중 Origin (정적 + API)

> "프론트엔드는 S3, 백엔드 API는 ALB. 하나의 도메인으로 서비스."

**정답**: CloudFront **Behavior** (경로 패턴)
- `/api/*` → ALB Origin
- `Default (*)` → S3 Origin

---

## 5. 핵심 키워드 → 서비스 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "정적 웹사이트" + "비용 최소" | **S3 + CloudFront** |
| "글로벌 지연 시간 감소" | **CloudFront** |
| "HTTPS" + "CloudFront" | **ACM (us-east-1)** |
| "커스텀 도메인" + "Zone Apex" | **Route 53 Alias** |
| "S3 직접 접근 차단" | **OAC + Bucket Policy** |
| "캐시 무효화" | **CloudFront Invalidation** |
| "SPA 404 에러" | **CloudFront Custom Error Response** |
| "서버 없이 호스팅" | **S3 + CloudFront** (서버리스) |

---

## 6. 헷갈리는 포인트 / 함정

### 함정 1: ACM 인증서 리전

- CloudFront용 ACM 인증서는 **반드시 us-east-1**
- ALB용 ACM 인증서는 **ALB가 있는 리전**
- 💡 "CloudFront + HTTPS" 문제에서 리전 확인

### 함정 2: OAI vs OAC

- **OAI** (Origin Access Identity): 레거시 방식
- **OAC** (Origin Access Control): 현재 권장 (SSE-KMS 지원, 더 세밀한 제어)
- 시험에서 OAI가 보이면 정답일 수 있으나, OAC가 더 최신

### 함정 3: S3 웹사이트 엔드포인트 vs REST API 엔드포인트

| 엔드포인트 | 형식 | HTTPS | CloudFront Origin |
|-----------|------|-------|-------------------|
| 웹사이트 | `bucket.s3-website-region.amazonaws.com` | ❌ HTTP만 | Custom Origin |
| REST API | `bucket.s3.region.amazonaws.com` | ✅ | **S3 Origin (OAC 사용)** |

- 💡 CloudFront + OAC → **REST API 엔드포인트** 사용 (S3 Origin)
- 💡 S3 정적 웹사이트 호스팅 활성화 → 웹사이트 엔드포인트 (HTTP만)

### 함정 4: CloudFront 지리적 제한

- **Geo Restriction**: 특정 국가 허용/차단 가능
- 시험에서 "특정 국가에서만 접근" → CloudFront Geo Restriction

---

## 7. 검증 필요 항목 ⚠️

- [ ] OAC가 시험에서 OAI 대신 정답으로 나오는 빈도 (최신 문제풀 확인)
- [ ] CloudFront Functions vs Lambda@Edge 최신 비교
- [ ] S3 Transfer Acceleration과 CloudFront 차이점 최신 정리
- [ ] CloudFront 최신 캐시 정책 (Cache Policy / Origin Request Policy)
- [ ] Route 53 Alias 레코드 지원 대상 최신 목록

---

## 8. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "정적 웹사이트 호스팅" → S3 + CloudFront 조합
- **유형 2**: "HTTPS 설정" → ACM (us-east-1) + CloudFront
- **유형 3**: "S3 직접 접근 차단" → OAC/OAI
- **유형 4**: "SPA 라우팅 에러" → CloudFront Custom Error Response
- **유형 5**: "Zone Apex 도메인" → Route 53 Alias

---

## 9. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
