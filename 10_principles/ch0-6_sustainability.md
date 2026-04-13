# Chapter 0-6. 지속 가능성 (Sustainability)

## 0. 한 줄 요약

🔑 **"지속 가능성 = 최소 리소스로 최대 비즈니스 가치를 내는 것 — 서버리스 + 관리형 서비스 + Right-Sizing + 효율적 아키텍처로 에너지·자원 낭비를 줄이는 것"**

---

## 1. 핵심 설계 원칙 (6가지)

| # | 원칙 | 설명 | AWS 서비스 예시 |
|---|------|------|---------------|
| 1 | **Understand your impact** | 클라우드 워크로드의 환경 영향 측정 | Customer Carbon Footprint Tool |
| 2 | **Establish sustainability goals** | 지속 가능성 KPI 설정·추적 | CloudWatch, Cost Explorer |
| 3 | **Maximize utilization** | 리소스 활용률 극대화 | ASG, Lambda, Fargate, Spot |
| 4 | **Anticipate and adopt efficient offerings** | AWS 신규 효율적 기술 채택 | **Graviton** 프로세서, 최신 인스턴스 세대 |
| 5 | **Use managed services** | 공유 인프라 = 효율적 | RDS, DynamoDB, S3 |
| 6 | **Reduce downstream impact** | 불필요한 데이터 처리·전송 줄이기 | CloudFront 캐싱, S3 Lifecycle |

---

## 2. 키워드 → 서비스 매핑

| 시험 키워드 | 정답 서비스/전략 |
|------------|---------------|
| "탄소 발자국" / "carbon footprint" | **Customer Carbon Footprint Tool** |
| "에너지 효율" / "energy efficient" | **Graviton 인스턴스** (ARM, 최대 60% 에너지 효율↑) |
| "리소스 활용률 극대화" | **ASG** / **Lambda** / **Fargate** |
| "유휴 리소스 제거" | **Trusted Advisor** / **Compute Optimizer** |
| "데이터 불필요 보관 줄이기" | **S3 Lifecycle** / **Data Retention Policy** |
| "불필요한 데이터 전송 줄이기" | **CloudFront** / **VPC Endpoint** |
| "효율적 코드" / "optimize code" | Lambda 메모리·타임아웃 최적화, 코드 프로파일링 |
| "최신 세대 인스턴스" | **최신 세대 EC2** (C7g, M7g, R7g) |

---

## 3. 6대 영역별 최적화 전략

### 3-1. 리전 선택 (Region Selection)

| 전략 | 설명 |
|------|------|
| **재생 에너지 리전** | AWS가 재생 에너지 100% 사용 리전 우선 |
| **사용자 가까운 리전** | 데이터 전송 거리↓ = 에너지↓ |

💡 SAA 시험에서 리전 선택 문제는 주로 **지연·규정** 기준이지만, "지속 가능성" 키워드 시 재생 에너지 리전도 고려

### 3-2. 컴퓨팅 (Compute)

| 전략 | 서비스 | 효과 |
|------|--------|------|
| **Graviton 사용** | C7g, M7g, R7g | **최대 60% 에너지 효율↑**, 20% 가격↓ |
| **서버리스** | Lambda, Fargate | 요청 시에만 실행 (유휴 = 0) |
| **Right-Sizing** | Compute Optimizer | 초과 프로비저닝 제거 |
| **Spot Instances** | EC2 Spot | 유휴 용량 재활용 |
| **최신 세대** | 최신 인스턴스 | 세대↑ = 와트당 성능↑ |

💡 "**가장 에너지 효율적인 프로세서**" → **Graviton (ARM)**  
💡 "**유휴 시간 0**" → **Lambda / Fargate**

### 3-3. 스토리지 (Storage)

| 전략 | 서비스 | 효과 |
|------|--------|------|
| **Lifecycle Policy** | S3 | 오래된 데이터 → Glacier → 삭제 |
| **Intelligent-Tiering** | S3 | 자동 계층 이동 (과다 보관 방지) |
| **데이터 압축** | S3/Firehose | 저장 용량↓ = 에너지↓ |
| **불필요 데이터 삭제** | S3 Expiration | 보존 기한 후 자동 삭제 |
| **EBS 최적화** | gp3, 스냅샷 관리 | 미사용 볼륨·스냅샷 삭제 |

### 3-4. 데이터 (Data)

| 전략 | 서비스 | 효과 |
|------|--------|------|
| **캐싱** | CloudFront, ElastiCache | 반복 요청 → 원본 부하↓ |
| **효율적 쿼리** | Athena (Parquet/ORC) | 컬럼형 포맷 = 스캔 데이터↓ |
| **데이터 중복 제거** | S3 같은 객체 참조 | 불필요 복제 줄이기 |

### 3-5. 네트워크 (Network)

| 전략 | 서비스 | 효과 |
|------|--------|------|
| **CDN 캐싱** | CloudFront | 오리진 요청↓ |
| **VPC Endpoint** | Gateway/Interface | NAT 경유 트래픽↓ |
| **데이터 압축** | gzip, Brotli | 전송 데이터↓ |

### 3-6. 개발·배포 프로세스

| 전략 | 설명 |
|------|------|
| **코드 최적화** | Lambda 실행 시간↓ = 비용·에너지↓ |
| **컨테이너 이미지 최적화** | 이미지 크기↓ = 풀 시간·스토리지↓ |
| **CI/CD 효율화** | 불필요 빌드 줄이기, 캐시 활용 |

---

## 4. 시험 빈출 시나리오

### 시나리오 1: "가장 에너지 효율적인 EC2 인스턴스"

**정답**: **Graviton (ARM) 기반** (C7g, M7g, R7g)  
💡 x86 대비 최대 60% 에너지 효율↑

### 시나리오 2: "유휴 시간 동안 리소스 낭비 최소화"

**정답**: **Lambda** / **Fargate** (서버리스 = 유휴 시 비용·에너지 0)  
또는 **ASG** (수요 0이면 스케일인)

### 시나리오 3: "오래된 S3 데이터 효율적 관리"

**정답**: **S3 Lifecycle Policy** (IA → Glacier → 삭제) 또는 **Intelligent-Tiering**

### 시나리오 4: "반복 요청으로 원본 서버 과부하"

**정답**: **CloudFront 캐싱** + **ElastiCache** (원본 요청 수↓)

### 시나리오 5: "Athena 쿼리 비용 + 스캔 데이터 줄이기"

**정답**: **Parquet/ORC 포맷** + **파티셔닝** (스캔 데이터↓ = 비용↓ + 에너지↓)

### 시나리오 6: "탄소 발자국 측정"

**정답**: **Customer Carbon Footprint Tool** (AWS Console에서 확인)

### 시나리오 7: "EC2 인스턴스 세대 업그레이드"

**정답**: 구세대 → **최신 세대** (m5 → m7g) = 와트당 성능↑, 비용↓

---

## 5. 함정 / 주의사항

### 함정 1: 지속 가능성 ≠ 단순 비용 절감

- 비용 최적화와 **겹치는 부분이 많지만** 관점이 다름
- 비용: "돈을 아끼자"
- 지속 가능성: "**리소스(에너지/하드웨어)를 덜 쓰자**"
- 💡 Graviton이 정답인 이유: 성능 동일 + 에너지↓ + 비용↓

### 함정 2: SAA 시험에서 지속 가능성 출제 빈도

- 2022년 추가된 **6번째 원칙** → 아직 출제 비중 **낮음** (1~2문제)
- 하지만 "에너지 효율" / "Graviton" 키워드로 등장 가능
- 💡 깊이보다 **키워드 매핑** 정도면 충분

### 함정 3: Graviton = ARM = 호환성 확인

- 모든 소프트웨어가 ARM 지원하는 것은 아님
- 💡 "코드 변경 없이" + "지속 가능성" → Graviton이 정답이 되려면 ARM 호환 전제
- 시험에서는 보통 **호환 가능하다고 가정**

### 함정 4: 서버리스가 항상 효율적인 것은 아님

- 24/7 고부하 → 서버리스보다 **예약 인스턴스가 효율적** (단위 처리당 에너지)
- 💡 "간헐적 / 불규칙 트래픽" → 서버리스 / "24/7 고부하" → RI + Right-Sizing

---

## 6. 검증 필요 항목 ⚠️

- [ ] Graviton4 출시 여부 및 성능 개선 수치
- [ ] Customer Carbon Footprint Tool 최신 리전 지원
- [ ] AWS 재생 에너지 100% 달성 리전 목록 (2025년 목표 이후?)
- [ ] 지속 가능성 원칙 SAA-C03 실제 출제 비중 (커뮤니티 피드백)
- [ ] Graviton ARM 호환 서비스 최신 목록 (RDS, ElastiCache 등)

---

## 7. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "에너지 효율 / Graviton" → ARM 인스턴스 (C7g/M7g/R7g)
- **유형 2**: "탄소 발자국 측정" → Customer Carbon Footprint Tool
- **유형 3**: "유휴 리소스 낭비 최소" → 서버리스 (Lambda/Fargate)
- **유형 4**: "오래된 데이터 관리" → S3 Lifecycle
- **유형 5**: "리소스 활용률 극대화" → Right-Sizing (Compute Optimizer)
- **유형 6**: "최신 세대 인스턴스" → 세대 업그레이드

---

## 8. Well-Architected 6대 원칙 총정리

| # | 원칙 | 핵심 키워드 | 대표 서비스 |
|---|------|-----------|-----------|
| 1 | **운영 우수성** | IaC, 자동화, 작은 변경 | CloudFormation, SSM, CodeDeploy |
| 2 | **보안** | 최소 권한, 암호화, 탐지 | IAM, KMS, GuardDuty, WAF |
| 3 | **안정성** | 자동 복구, Multi-AZ, DR | ASG, Route 53, RDS Multi-AZ |
| 4 | **성능 효율성** | 캐싱, Right-Type, 글로벌 | CloudFront, ElastiCache, Graviton |
| 5 | **비용 최적화** | RI/SP, Spot, Lifecycle | Savings Plans, Spot, S3 Lifecycle |
| 6 | **지속 가능성** | Graviton, 서버리스, 효율 | Graviton, Lambda, Lifecycle |

---

## 9. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
