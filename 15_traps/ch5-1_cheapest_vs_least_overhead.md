# Chapter 5-1. "가장 저렴한(Cheapest)" vs "운영 오버헤드가 적은(Least Overhead)"

## 0. 한 줄 요약

🔑 **"Most cost-effective"는 돈을 아끼라는 뜻이고, "Least operational overhead"는 관리를 줄이라는 뜻 — 같은 시나리오에서 정답이 완전히 달라지는 SAA 최대 함정**

---

## 1. 핵심 구분

| 키워드 (영문) | 의미 | 정답 방향 |
|--------------|------|----------|
| **Most cost-effective** / Cheapest / Minimize cost | **비용 최소화** | 저렴한 서비스, 예약 인스턴스, 수동 설정 OK |
| **Least operational overhead** / Minimum management | **관리 최소화** | 관리형/서버리스, 자동화, 비용 높아도 OK |
| **Most operationally efficient** | 관리 최소 + 효율적 | 서버리스/완전 관리형 |

---

## 2. 동일 시나리오, 다른 정답

### 예시 1: 웹 애플리케이션 호스팅

| 질문 변형 | 정답 | 이유 |
|----------|------|------|
| "가장 **비용 효율적**인 방법" | **EC2 Reserved Instance + ALB** | RI가 On-Demand보다 ~72% 저렴 |
| "**운영 오버헤드 최소화**" | **Fargate + ALB** 또는 **App Runner** | 서버 관리 제로 |

### 예시 2: 배치 데이터 처리

| 질문 변형 | 정답 | 이유 |
|----------|------|------|
| "가장 **저렴한** 배치 처리" | **EC2 Spot Instance + AWS Batch** | Spot = 최대 90% 할인 |
| "**관리 최소화** 배치 처리" | **Lambda** 또는 **Fargate** | 인프라 관리 없음 |

### 예시 3: 데이터베이스

| 질문 변형 | 정답 | 이유 |
|----------|------|------|
| "가장 **비용 효율적**인 DB" | **RDS Reserved Instance** | RI 할인 |
| "**운영 오버헤드 최소화** DB" | **Aurora Serverless** 또는 **DynamoDB** | 자동 스케일링, 패치 자동화 |

### 예시 4: ETL 파이프라인

| 질문 변형 | 정답 | 이유 |
|----------|------|------|
| "가장 **저렴한** ETL" | **EMR + Spot Instance** | Spot 노드로 비용↓ |
| "**관리 최소화** ETL" | **AWS Glue** | 서버리스, 인프라 관리 없음 |

### 예시 5: 파일 스토리지

| 질문 변형 | 정답 | 이유 |
|----------|------|------|
| "가장 **저렴한** 파일 저장" | **S3 Glacier Deep Archive** | $0.00099/GB/월 |
| "**관리 최소화** 파일 공유" | **EFS** 또는 **FSx** | 자동 확장, 관리형 |

---

## 3. 비용 최적화 키워드 → 정답 패턴

| 비용 키워드 | 정답 패턴 |
|------------|----------|
| "비용 절감" + EC2 | **Reserved Instance** (1년/3년) 또는 **Savings Plan** |
| "변동 워크로드" + 비용 | **Spot Instance** (중단 허용) |
| "사용하지 않는 리소스" | **ASG 축소** / **스케줄링** / **인스턴스 중지** |
| "스토리지 비용" | **S3 Lifecycle** (IA → Glacier) |
| "데이터 전송 비용" | **CloudFront** (오리진 전송 무료) / **VPC Endpoint** |
| "DB 비용" | **Reserved Instance** / **Aurora Serverless** (가변 트래픽) |
| "다중 계정 비용" | **Organizations 통합 결제** (볼륨 할인) |

---

## 4. 운영 오버헤드 최소화 키워드 → 정답 패턴

| 관리 키워드 | 정답 패턴 |
|------------|----------|
| "서버 관리 없이" | **서버리스** (Lambda, Fargate, DynamoDB) |
| "패치/업데이트 자동화" | **관리형 서비스** (RDS, ElastiCache, OpenSearch) |
| "자동 확장" | **Auto Scaling** / **서버리스** |
| "가장 간단한 방법" | **완전 관리형** (Firehose, Glue, App Runner) |
| "코드 변경 없이" | **관리형 서비스로 전환** / **마이그레이션 도구** |
| "운영 팀 부담 감소" | **서버리스** / **관리형** / **자동화** |

---

## 5. 서비스별 비용 vs 관리 스펙트럼

```
비용 저렴 ←──────────────────────────────────→ 관리 편리

EC2 Spot     EC2 RI     EC2 On-Demand     Fargate     Lambda
  💰            💰💰        💰💰💰          💰💰💰💰    💰(소량)~💰💰💰💰(대량)
  🔧🔧🔧       🔧🔧🔧     🔧🔧🔧          🔧           🔧

EMR+Spot     EMR RI      EMR On-Demand     Glue
  💰            💰💰        💰💰💰          💰💰💰💰
  🔧🔧🔧       🔧🔧🔧     🔧🔧🔧          🔧

RDS RI       RDS On-Demand   Aurora Serverless   DynamoDB On-Demand
  💰            💰💰💰          💰💰💰💰           💰💰💰💰(소량 저렴)
  🔧🔧          🔧🔧            🔧                  🔧
```

💰 = 비용, 🔧 = 관리 부담

---

## 6. 시험 빈출 시나리오

### 시나리오 1: "가장 비용 효율적으로 정적 웹사이트 호스팅"

**정답**: S3 + CloudFront (서버 비용 제로)  
**오답 함정**: EC2 (서버 비용 발생)  
💡 이 경우 비용 효율 = 운영 최소화 = 같은 정답

### 시나리오 2: "비용 최소화하면서 간헐적 배치 작업 실행"

**정답**: **Lambda** (실행 시간만 과금, 15분 이내) 또는 **EC2 Spot**  
💡 "비용 최소화" + "간헐적" → Lambda가 정답 (서버리스 + 유휴 비용 제로)

### 시나리오 3: "24/7 고정 워크로드, 비용 최적화"

**정답**: **EC2 Reserved Instance** 또는 **Compute Savings Plan**  
**오답 함정**: Lambda (24/7 실행 시 비쌈), Spot (중단 위험)

### 시나리오 4: "개발팀의 운영 부담을 줄이면서 컨테이너 실행"

**정답**: **Fargate** (서버리스 컨테이너)  
**오답 함정**: ECS on EC2 (인스턴스 관리 필요)

### 시나리오 5: 키워드 조합 판별

| 문제 키워드 조합 | 정답 |
|----------------|------|
| "cost-effective" + "steady-state" | **Reserved Instance** |
| "cost-effective" + "fault-tolerant batch" | **Spot Instance** |
| "least overhead" + "container" | **Fargate** / **App Runner** |
| "least overhead" + "database" | **Aurora Serverless** / **DynamoDB** |
| "least overhead" + "ETL" | **Glue** |
| "cost-effective" + "variable traffic" | **ASG + Spot 혼합** |

---

## 7. 헷갈리는 포인트 / 함정

### 함정 1: "Cost-effective"가 항상 "가장 싼" 것은 아니다

- Cost-effective = 가성비 (비용 대비 효과)
- 매우 싸지만 요구사항 미달 → ❌
- 💡 **요구사항 만족하는 것 중 가장 저렴한 것**

### 함정 2: 서버리스가 항상 저렴한 것은 아니다

- Lambda: 소량 호출 = 거의 무료, 대량 24/7 = **EC2보다 비쌈**
- DynamoDB On-Demand: 소량 = 저렴, 대량 = **Provisioned보다 6배 비쌈**
- 💡 "비용 최적화" + "예측 가능한 대규모" → 서버리스 ❌ → **예약/프로비저닝**

### 함정 3: "Least effort"와 "Least overhead"는 다르다

- **Least effort**: 초기 설정 노력 최소 (마이그레이션 등)
- **Least overhead**: 지속적 운영 부담 최소
- 💡 "코드 변경 최소" = 마이그레이션 effort, "관리 최소" = 운영 overhead

---

## 8. 검증 필요 항목 ⚠️

- [ ] EC2 Savings Plan vs Reserved Instance 최신 할인율
- [ ] Lambda 비용 분기점 (EC2보다 비싸지는 호출량)
- [ ] Fargate vs EC2 비용 비교 최신 벤치마크
- [ ] Aurora Serverless v2 최신 요금

---

## 9. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "cost-effective" vs "least overhead" 판별 → 정답 방향 결정
- **유형 2**: "24/7 고정 워크로드 비용" → RI / Savings Plan
- **유형 3**: "서버리스 vs 관리형 vs EC2" 선택
- **유형 4**: "비용 + 관리" 복합 조건 판별

---

## 10. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
