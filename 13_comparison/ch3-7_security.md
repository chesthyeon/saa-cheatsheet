# Chapter 3-7. 보안: KMS vs CloudHSM / ACM vs Secrets Manager vs Parameter Store

## 0. 한 줄 요약

🔑 **"암호화 키 관리 vs 인증서 관리 vs 비밀값 관리" — 보호 대상이 무엇인지(키/인증서/비밀)와 규정 준수 수준(FIPS 140-2 Level 2 vs 3)이 서비스를 결정한다**

---

## 1. 전체 비교 매트릭스

| 기준 | KMS | CloudHSM | ACM | Secrets Manager | Parameter Store |
|------|-----|----------|-----|-----------------|-----------------|
| **목적** | **암호화 키 관리** | **전용 HSM으로 키 관리** | **SSL/TLS 인증서** 관리 | **비밀값** (DB 비밀번호, API 키) | **구성 값** (설정, 비밀값 포함) |
| **관리 수준** | AWS 관리 | **고객 전용 하드웨어** | 완전 관리형 | 완전 관리형 | 완전 관리형 |
| **규정 준수** | FIPS 140-2 **Level 2** | FIPS 140-2 **Level 3** | N/A | KMS로 암호화 | KMS로 암호화 (선택) |
| **키 소유** | AWS + 고객 공유 | **고객 단독** | AWS 관리 | AWS 관리 | AWS 관리 |
| **자동 로테이션** | ✅ (1년 기본) | 고객 직접 관리 | ✅ (자동 갱신) | ✅ **내장** (30~365일) | ❌ (Lambda로 구현 가능) |
| **비용** | 키당 $1/월 + API 호출 | 인스턴스당 ~$1.50/시간 | **무료** (공개 인증서) | 비밀당 $0.40/월 | 무료 (Standard) / 유료 (Advanced) |
| **주요 통합** | S3, EBS, RDS 등 거의 모든 AWS | KMS 커스텀 키 스토어 | ALB, CloudFront, API Gateway | RDS, Redshift, Lambda | Lambda, ECS, EC2 |

---

## 2. 핵심 판별 플로우차트

```
"보안 서비스 선택"
  │
  ├─ 암호화 키 관리?
  │   ├─ AWS 관리형 키 OK? ──→ KMS (AWS managed key)
  │   ├─ 고객 관리 키 (CMK) 필요? ──→ KMS (Customer managed key)
  │   └─ FIPS 140-2 Level 3 / 전용 HSM? ──→ CloudHSM
  │
  ├─ SSL/TLS 인증서?
  │   ├─ ALB/CloudFront/API Gateway용? ──→ ACM (무료)
  │   └─ EC2에 직접 설치? ──→ ACM ❌ (자체 인증서 관리)
  │
  └─ 비밀값 / 설정값 저장?
      ├─ DB 비밀번호 + 자동 로테이션? ──→ Secrets Manager
      ├─ 단순 설정값 (비암호화 OK)? ──→ Parameter Store (Free Tier)
      └─ 비밀값 + 비용 최소화? ──→ Parameter Store SecureString
```

---

## 3. 쌍별 상세 비교

### KMS vs CloudHSM

| 기준 | KMS | CloudHSM |
|------|-----|----------|
| 인프라 | **AWS 공유** 인프라 | **고객 전용** 하드웨어 |
| 규정 준수 | FIPS 140-2 **Level 2** | FIPS 140-2 **Level 3** |
| 키 접근 | AWS가 일부 접근 가능 | **고객만 접근** (AWS도 불가) |
| 관리 부담 | 낮음 (완전 관리형) | 높음 (클러스터 운영) |
| 비용 | 저렴 (키당 $1/월) | **고가** (~$1.50/시간 ≒ $1,080/월) |
| 사용 사례 | 대부분의 암호화 | 규정상 전용 HSM 필수, 오라클 TDE |
| KMS 통합 | 기본 | ✅ **Custom Key Store** (KMS API + CloudHSM 백엔드) |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "암호화", "SSE-KMS", "CMK" | **KMS** |
| "FIPS 140-2 Level 3" | **CloudHSM** |
| "전용 하드웨어 키 관리" | **CloudHSM** |
| "고객 단독 키 접근" | **CloudHSM** |
| "CloudHSM + KMS API 사용" | **KMS Custom Key Store** |

### Secrets Manager vs Parameter Store

| 기준 | Secrets Manager | Parameter Store |
|------|-----------------|-----------------|
| 목적 | **비밀값 전용** (DB 자격 증명, API 키) | **범용 설정값** (비밀 + 일반 설정) |
| 자동 로테이션 | ✅ **내장** (Lambda 자동 호출) | ❌ (직접 Lambda 구성 필요) |
| RDS 통합 | ✅ **네이티브** (RDS 비밀번호 자동 로테이션) | ❌ |
| 암호화 | **필수** (KMS) | 선택 (SecureString = KMS 암호화) |
| 비용 | **유료** ($0.40/비밀/월 + API) | Standard: **무료** / Advanced: 유료 |
| 값 크기 | 최대 64 KB | Standard: 4 KB / Advanced: 8 KB |
| 계층 구조 | ❌ | ✅ (경로 기반: /app/prod/db-password) |
| Cross-Account | ✅ | ✅ (Advanced) |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "DB 비밀번호 자동 로테이션" | **Secrets Manager** |
| "RDS 자격 증명 관리" | **Secrets Manager** |
| "비용 최소화" + "비밀값 저장" | **Parameter Store SecureString** |
| "앱 설정값 + 비밀값 혼합" | **Parameter Store** |
| "자동 로테이션 필수" | **Secrets Manager** |

### ACM vs 자체 인증서 관리

| 기준 | ACM (AWS Certificate Manager) | 자체 인증서 |
|------|-------------------------------|------------|
| 발급 | **무료** (공개 인증서) | 유료 (CA에서 구매) |
| 갱신 | **자동 갱신** | 수동 갱신 (만료 위험) |
| 설치 대상 | ALB, CloudFront, API Gateway, NLB | **EC2 직접** (ACM 미지원) |
| 프라이빗 인증서 | ACM Private CA (유료) | 자체 CA |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "HTTPS 설정" + "ALB/CloudFront" | **ACM** |
| "인증서 자동 갱신" | **ACM** |
| "EC2에 SSL 인증서 설치" | **자체 관리** (ACM은 EC2 직접 미지원) |
| "프라이빗 내부 인증서" | **ACM Private CA** |

### S3 암호화 옵션 비교

| 기준 | SSE-S3 | SSE-KMS | SSE-C | 클라이언트 암호화 |
|------|--------|---------|-------|-----------------|
| 키 관리 | AWS 완전 관리 | **KMS** (CMK) | **고객 제공 키** | 고객 완전 관리 |
| 감사 로그 | ❌ | ✅ **CloudTrail** | ❌ | ❌ |
| 키 접근 제어 | ❌ | ✅ **키 정책** | ❌ | ❌ |
| 추가 비용 | 무료 | KMS API 호출 비용 | 무료 | 무료 |
| 기본값 | ✅ (2023~ 기본 활성화) | 선택 | 선택 | 선택 |

**시험 판별:**

| 키워드 | 정답 |
|--------|------|
| "S3 기본 암호화" | **SSE-S3** |
| "키 접근 감사", "CloudTrail 키 사용 로그" | **SSE-KMS** |
| "키 로테이션 제어" | **SSE-KMS (CMK)** |
| "고객이 키를 직접 제공" | **SSE-C** |
| "업로드 전 암호화" | **클라이언트 암호화** |

---

## 4. 시험 빈출 시나리오 + 정답

| 시나리오 | 정답 | 오답 함정 |
|----------|------|----------|
| S3 + EBS 데이터 암호화, 키 관리 | **KMS (CMK)** | CloudHSM (과도) |
| 규정 준수: FIPS 140-2 Level 3 필수 | **CloudHSM** | KMS (Level 2만) |
| RDS 비밀번호 자동 로테이션 | **Secrets Manager** | Parameter Store (자동 로테이션 없음) |
| 앱 설정값 저장, 비용 최소화 | **Parameter Store Standard** | Secrets Manager (유료) |
| ALB에 HTTPS 설정, 인증서 자동 갱신 | **ACM** | 자체 인증서 (수동 갱신) |
| EC2에 직접 SSL 인증서 설치 | **자체 관리** (ACM 불가) | ACM (EC2 직접 미지원) |
| S3 객체 암호화 + 키 사용 감사 | **SSE-KMS** | SSE-S3 (감사 불가) |
| Cross-Account S3 암호화 키 공유 | **KMS 키 정책** (다른 계정 허용) | SSE-S3 (키 제어 불가) |
| DB 비밀번호 + API 키 + 앱 설정 통합 관리 | **Parameter Store** (혼합) + Secrets Manager (DB) | 하나만 사용 |
| CloudHSM + KMS API 통합 | **KMS Custom Key Store** | CloudHSM 직접 (API 호환 필요) |

---

## 5. 헷갈리는 포인트 / 함정

### 함정 1: KMS 키 유형 혼동

- **AWS 관리형 키** (aws/s3, aws/ebs): 자동 생성, 무료, 관리 불가
- **고객 관리형 키** (CMK): $1/월, 키 정책 설정 가능, 로테이션 제어
- **AWS 소유 키**: 완전 투명, 고객이 볼 수 없음
- 💡 "키 정책 제어", "Cross-Account 키 공유" → **고객 관리형 키 (CMK)**

### 함정 2: Secrets Manager vs Parameter Store 비용

- Secrets Manager: 비밀당 $0.40/월 (비밀 10개 = $4/월)
- Parameter Store Standard: **완전 무료** (최대 10,000개)
- 💡 "비용 최적화" + "비밀값" → Parameter Store SecureString 먼저 고려
- 💡 "자동 로테이션 필수" → Secrets Manager (비용 무관)

### 함정 3: ACM 인증서 ≠ EC2 직접 설치

- ACM 인증서는 ALB, CloudFront, API Gateway, NLB에만 연결 가능
- **EC2에 직접 SSL 설정**하려면 자체 인증서 필요
- 💡 "EC2 + HTTPS" → ALB(ACM) + EC2 구성이 정답인 경우가 많음

### 함정 4: SSE-KMS 처리량 제한

- SSE-KMS는 **KMS API 호출 한도** 적용 (리전당 5,500~30,000 req/s)
- 대량 S3 작업 시 쓰로틀링 가능
- 💡 "S3 대량 업로드" + "암호화" → SSE-S3가 처리량 제한 없음

---

## 6. 검증 필요 항목 ⚠️

- [ ] KMS 자동 키 로테이션 주기 (1년 → 변경 여부, 커스텀 주기 지원?)
- [ ] CloudHSM 최신 시간당 비용
- [ ] Secrets Manager 자동 로테이션 최소 주기 (30일?)
- [ ] Parameter Store Advanced 최신 요금 및 한도
- [ ] ACM Private CA 최신 요금
- [ ] SSE-KMS 리전별 API 호출 한도 최신값
- [ ] S3 기본 암호화 SSE-S3 자동 활성화 시점 (2023년 1월?)

---

## 7. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "암호화 키 관리" → KMS vs CloudHSM (규정 준수 수준)
- **유형 2**: "비밀값 저장 + 로테이션" → Secrets Manager vs Parameter Store
- **유형 3**: "SSL/TLS 인증서" → ACM vs 자체 관리
- **유형 4**: "S3 암호화 방식" → SSE-S3 vs SSE-KMS vs SSE-C
- **유형 5**: "FIPS 140-2 Level 3" → CloudHSM

---

## 8. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
