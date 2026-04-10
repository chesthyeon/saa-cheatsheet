# Chapter 5-7. "관리형(Managed)" vs "서버리스(Serverless)" 혼동

## 0. 한 줄 요약

🔑 **"Managed = AWS가 인프라 관리(패치/백업), Serverless = 서버 개념 자체가 없음(프로비저닝/스케일링 자동)" — 모든 서버리스는 관리형이지만, 모든 관리형이 서버리스는 아니다**

---

## 1. 핵심 구분

| 기준 | Self-managed | Managed (관리형) | Serverless (서버리스) |
|------|-------------|-----------------|---------------------|
| **인프라** | 고객 관리 | **AWS 관리** (패치, HA) | **개념 없음** |
| **용량 계획** | 고객 결정 | 고객 결정 (인스턴스 크기) | **자동** |
| **스케일링** | 수동/ASG | 수동/제한적 자동 | **완전 자동** |
| **과금** | 인스턴스 시간 | 인스턴스 시간 | **사용량** (요청/시간/데이터) |
| **유휴 비용** | ✅ 있음 | ✅ 있음 | ❌ **없음** (대부분) |
| **예시** | EC2에 MySQL 설치 | RDS MySQL | Aurora Serverless |

### 관계도

```
Self-managed ⊂ Managed ⊂ Serverless

EC2 + 자체 MySQL    →  RDS MySQL        →  Aurora Serverless
EC2 + 자체 Redis    →  ElastiCache      →  ElastiCache Serverless
EC2 + 자체 Kafka    →  MSK              →  MSK Serverless
EC2 + 컨테이너      →  ECS on EC2       →  Fargate
EC2 + 자체 ETL      →  EMR              →  Glue
```

---

## 2. 서비스별 관리형 vs 서버리스 매핑

### 컴퓨팅

| Self-managed | Managed | Serverless |
|-------------|---------|-----------|
| EC2 (직접 관리) | ECS on EC2 | **Lambda** |
| EC2 (컨테이너) | ECS on EC2 | **Fargate** |
| - | - | **App Runner** |

### 데이터베이스

| Self-managed | Managed | Serverless |
|-------------|---------|-----------|
| EC2 + MySQL | **RDS MySQL** | **Aurora Serverless** |
| EC2 + PostgreSQL | **RDS PostgreSQL** | **Aurora Serverless** |
| EC2 + DynamoDB 대안 | - | **DynamoDB** (항상 서버리스) |
| EC2 + Redis | **ElastiCache** | **ElastiCache Serverless** |
| EC2 + Redshift | **Redshift Provisioned** | **Redshift Serverless** |

### 분석/ETL

| Self-managed | Managed | Serverless |
|-------------|---------|-----------|
| EC2 + Spark | **EMR** | **Glue** |
| EC2 + Presto | **EMR** | **Athena** |
| EC2 + Elasticsearch | **OpenSearch** | **OpenSearch Serverless** |

### 메시징/스트리밍

| Self-managed | Managed | Serverless |
|-------------|---------|-----------|
| EC2 + Kafka | **MSK** | **MSK Serverless** |
| EC2 + RabbitMQ | **Amazon MQ** | SQS/SNS (네이티브 서버리스) |

### 스토리지

| Self-managed | Managed | Serverless |
|-------------|---------|-----------|
| EC2 + NFS | **EFS** (서버리스) | **EFS** |
| EC2 + 파일 서버 | **FSx** | - |
| - | - | **S3** (항상 서버리스) |

---

## 3. 시험에서의 판별 로직

```
"서비스 선택"
  │
  ├─ "코드 변경 최소화" (기존 앱 마이그레이션)
  │   └─ Managed (호환 서비스): RDS, ElastiCache, Amazon MQ, MSK
  │
  ├─ "운영 오버헤드 최소화"
  │   └─ Serverless: Lambda, Fargate, Aurora Serverless, DynamoDB
  │
  ├─ "비용 최소화" + "예측 가능한 워크로드"
  │   └─ Managed + Reserved: RDS RI, ElastiCache RI
  │
  └─ "비용 최소화" + "예측 불가 워크로드"
      └─ Serverless: Lambda, DynamoDB On-Demand, Aurora Serverless
```

---

## 4. 시험 빈출 시나리오

### 시나리오 1: "운영 오버헤드 최소화" + DB

> "DB 운영 부담을 최소화해야 한다."

**정답**: **Aurora Serverless** 또는 **DynamoDB**  
**오답 함정**: RDS (인스턴스 크기 결정 필요, 패치 윈도우 관리)

### 시나리오 2: "관리형" vs "서버리스" 비용

| 워크로드 패턴 | 관리형 비용 | 서버리스 비용 | 정답 |
|-------------|-----------|-------------|------|
| 24/7 고정 부하 | 💰💰 (RI 할인) | 💰💰💰 | **Managed + RI** |
| 간헐적/변동 | 💰💰💰 (유휴 낭비) | 💰 | **Serverless** |
| 예측 불가 급증 | 💰💰💰💰 (오버프로비저닝) | 💰💰 (자동 확장) | **Serverless** |

### 시나리오 3: Fargate vs ECS on EC2

> "컨테이너 운영. 운영 오버헤드 최소화."

**정답**: **Fargate** (서버리스, 인스턴스 관리 없음)  
**비용 최적화라면**: **ECS on EC2** (Spot/RI 활용)

### 시나리오 4: EMR vs Glue

> "ETL 작업. 서버 관리 없이."

**정답**: **Glue** (서버리스 ETL)  
**대규모 커스텀이라면**: **EMR** (Spark 클러스터 직접 관리)

### 시나리오 5: RDS vs Aurora Serverless

> "예측 불가능한 트래픽 패턴. DB 자동 확장."

**정답**: **Aurora Serverless v2** (0.5~128 ACU 자동 스케일링)  
**오답 함정**: RDS (수동 스케일링, 다운타임 필요)

---

## 5. 자주 혼동되는 서비스 분류

| 서비스 | 관리형? | 서버리스? | 주의사항 |
|--------|--------|----------|---------|
| **RDS** | ✅ | ❌ | 인스턴스 크기 선택 필요 |
| **Aurora Provisioned** | ✅ | ❌ | 인스턴스 크기 선택 |
| **Aurora Serverless** | ✅ | ✅ | ACU 자동 스케일링 |
| **DynamoDB** | ✅ | ✅ | 항상 서버리스 |
| **ElastiCache** | ✅ | 노드: ❌ / Serverless: ✅ | 두 모드 존재 |
| **ECS on EC2** | ✅ | ❌ | EC2 인스턴스 관리 필요 |
| **Fargate** | ✅ | ✅ | 인스턴스 개념 없음 |
| **Lambda** | ✅ | ✅ | 완전 서버리스 |
| **API Gateway** | ✅ | ✅ | 완전 서버리스 |
| **S3** | ✅ | ✅ | 완전 서버리스 |
| **EFS** | ✅ | ✅ | 자동 확장, 프로비저닝 없음 |
| **EMR** | ✅ | ❌ | 클러스터 관리 (EMR Serverless 존재) |
| **Glue** | ✅ | ✅ | 서버리스 ETL |
| **MSK** | ✅ | ❌ / Serverless: ✅ | 두 모드 존재 |
| **Redshift** | ✅ | ❌ / Serverless: ✅ | 두 모드 존재 |

---

## 6. 헷갈리는 포인트 / 함정

### 함정 1: "Managed"가 "운영 부담 없음"은 아니다

- RDS: AWS가 패치하지만 → **인스턴스 크기, 스토리지, 패치 윈도우** 결정 필요
- 💡 "운영 오버헤드 최소화" → Managed ❌ → **Serverless**

### 함정 2: Serverless가 항상 저렴하지 않다

- Lambda: 소량 = 거의 무료, 24/7 대량 = EC2 RI보다 비쌈
- DynamoDB On-Demand: 예측 가능하면 Provisioned이 6배 저렴
- 💡 "비용 최적화" + "고정 워크로드" → Serverless ❌

### 함정 3: Aurora Serverless v1 vs v2

- **v1**: 0으로 스케일 다운 가능 (완전 중지) → 개발/테스트용
- **v2**: 0.5 ACU 최소 (완전 중지 불가) → 프로덕션용
- 💡 시험에서는 주로 v2 (프로덕션 시나리오)

### 함정 4: "서버리스"라고 이름 붙은 서비스

- 일부 서비스가 "Serverless" 모드를 추가: Redshift Serverless, MSK Serverless, OpenSearch Serverless, ElastiCache Serverless
- 기존 Provisioned 모드와 공존
- 💡 시험에서 "운영 최소화" → Serverless 모드 선택

---

## 7. 검증 필요 항목 ⚠️

- [ ] Aurora Serverless v2 최소 ACU (0.5?)
- [ ] ElastiCache Serverless 최신 요금/기능
- [ ] EMR Serverless 최신 기능
- [ ] MSK Serverless 최신 제한사항
- [ ] Redshift Serverless 최신 요금 체계

---

## 8. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "Managed vs Serverless" 선택 → 키워드 판별
- **유형 2**: "운영 오버헤드 최소화" → Serverless
- **유형 3**: "비용 최적화" + "고정 워크로드" → Managed + RI
- **유형 4**: "Serverless 비용 함정" → 대규모 고정 워크로드

---

## 9. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
