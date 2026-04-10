# Chapter 5-6. "애플리케이션 코드 변경 최소화" 키워드

## 0. 한 줄 요약

🔑 **"Minimum code changes" = 앱 코드를 건드리지 않고 인프라/서비스 레벨에서 해결하라는 뜻 — Lift-and-Shift, 프록시 패턴, 관리형 서비스 교체가 정답 방향**

---

## 1. 핵심 원칙

"코드 변경 최소화" 문제에서 **오답**이 되는 패턴:
- 애플리케이션 로직 수정
- SDK 교체 / API 변경
- 새로운 라이브러리 도입
- 코드에 AWS SDK 직접 호출 추가

"코드 변경 최소화" 문제에서 **정답**이 되는 패턴:
- 인프라 교체 (EC2 → Fargate 등)
- 환경 변수 / 엔드포인트 변경만으로 전환
- 프로토콜 호환 서비스로 교체 (MySQL → Aurora MySQL)
- 프록시/미들웨어로 중간 계층 추가

---

## 2. 서비스별 "코드 변경 없는" 전환

### 데이터베이스

| 현재 | 전환 대상 | 코드 변경 | 이유 |
|------|----------|----------|------|
| MySQL (EC2) | **Aurora MySQL** | ❌ 없음 | MySQL 호환 (엔드포인트만 변경) |
| PostgreSQL (EC2) | **Aurora PostgreSQL** | ❌ 없음 | PostgreSQL 호환 |
| MySQL (EC2) | **RDS MySQL** | ❌ 없음 | 동일 엔진 |
| MySQL (EC2) | DynamoDB | ✅ 대규모 | SQL → NoSQL 전환 필요 |
| Redis (EC2) | **ElastiCache Redis** | ❌ 없음 | Redis 프로토콜 호환 |
| Memcached (EC2) | **ElastiCache Memcached** | ❌ 없음 | Memcached 호환 |

��� **"코드 변경 최소"** + "DB 마이그레이션" → **동일 엔진의 관리형 서비스** (Aurora/RDS)

### 컴퓨팅

| 현재 | 전환 대상 | 코드 변경 | 이유 |
|------|----------|----------|------|
| EC2 | **같은 OS의 EC2** (다른 타입) | ❌ 없음 | 인스턴스 타입만 변경 |
| EC2 (컨테이너) | **ECS/Fargate** | 최소 | Dockerfile 있으면 거의 변경 없음 |
| EC2 | Lambda | ✅ 대규모 | 이벤트 핸들러로 리팩토링 |
| 온프레미스 VM | **EC2** (Lift-and-Shift) | ❌ 없음 | VM Import/Export |

💡 **"코드 변경 최소"** + "컴퓨팅 마이그레이션" → **Lift-and-Shift (EC2)** 또는 **같은 엔진 관리형**

### 스토리지

| 현재 | 전환 대상 | 코드 변경 | 이유 |
|------|----------|----------|------|
| NFS 파일 서버 | **EFS** | ❌ 없음 | NFS 프로토콜 호환 |
| SMB 파일 서버 | **FSx for Windows** | ❌ 없음 | SMB 프로토콜 호환 |
| Lustre | **FSx for Lustre** | ❌ 없음 | Lustre 호환 |
| 파일 서버 → S3 | **Storage Gateway** | ❌ 없음 | NFS/SMB → S3 투명 |
| 로컬 디스크 | S3 API | ✅ 필요 | API 호출 코드 필요 |

💡 **"코드 변경 최소"** + "파일 스토리지" → **프로토콜 호환 서비스** (EFS/FSx)

### 메시징/통합

| 현재 | 전환 대상 | 코드 변경 | 이유 |
|------|----------|----------|------|
| RabbitMQ | **Amazon MQ (RabbitMQ)** | ❌ 없음 | 프로토콜 호환 |
| ActiveMQ | **Amazon MQ (ActiveMQ)** | ❌ 없음 | 프로토콜 호환 |
| RabbitMQ | SQS/SNS | ✅ 필요 | API 다름 |
| Kafka | **MSK (Managed Kafka)** | ❌ 없음 | Kafka 호환 |

💡 **"코드 변경 최소"** + "기존 메시징" → **Amazon MQ** 또는 **MSK** (호환 서비스)  
💡 "코드 변경 OK" + "클라우드 네이티브" → **SQS/SNS** (더 저렴, 관리형)

---

## 3. 시험 빈출 시나리오

### 시나리오 1: MySQL EC2 → 관리형 전환

> "EC2의 MySQL을 관리형으로 전환. 코드 변경을 최소화."

**정답**: **Aurora MySQL** 또는 **RDS MySQL** (MySQL 호환)  
**오답 함정**: DynamoDB (NoSQL, 코드 대규모 변경)

### 시나리오 2: 온프레미스 앱 → AWS

> "온프레미스 앱을 AWS로 빠르게 이전. 코드 변경 없이."

**정답**: **EC2 (Lift-and-Shift)** + VM Import/Export 또는 AWS MGN  
**오답 함정**: Lambda로 리팩토링 (코드 대규모 변경)

### 시나리오 3: 기존 RabbitMQ → AWS

> "온프레미스 RabbitMQ를 AWS로. 코드 변경 최소화."

**정답**: **Amazon MQ for RabbitMQ** (프로토콜 호환)  
**오답 함정**: SQS (API 다름, 코드 변경 필요)

### 시나리오 4: NFS 파일 서버 → AWS

> "온프레미스 NFS 파일 서버를 AWS로. 앱 변경 없이."

**정답**: **EFS** (NFS 호환) 또는 **S3 File Gateway** (NFS 인터페이스)  
**오답 함정**: S3 직접 (NFS → S3 API 변경 필요)

### 시나리오 5: 세션 관리 외부화

> "EC2 로컬 세션을 외부화. 코드 변경 최소화."

**정답**: **ElastiCache Redis** (세션 라이브러리만 설정 변경)  
**대안**: **DynamoDB** (세션 스토어 라이브러리)  
💡 대부분 프레임워크가 Redis 세션 스토어를 설정 변경만으로 지원

### 시나리오 6: Kafka → AWS

> "온프레미스 Kafka를 AWS로. 프로듀서/컨슈머 코드 변경 없이."

**정답**: **Amazon MSK** (Managed Kafka, 100% 호환)  
**오답 함정**: Kinesis Data Streams (API 다름)

---

## 4. 빠른 판별표

| "코드 변경 최소" + 키워드 | 정답 |
|-------------------------|------|
| + MySQL/PostgreSQL | **Aurora** / **RDS** (같은 엔진) |
| + Redis/Memcached | **ElastiCache** (같은 엔진) |
| + RabbitMQ/ActiveMQ | **Amazon MQ** |
| + Kafka | **Amazon MSK** |
| + NFS | **EFS** / **S3 File Gateway** |
| + SMB/Windows | **FSx for Windows** |
| + 온프레미스 VM | **EC2 (Lift-and-Shift)** |
| + 컨테이너 | **ECS/Fargate** |

---

## 5. 헷갈리는 포인트 / 함정

### 함정 1: "코드 변경 최소화" vs "운영 오버헤드 최소화"

- **코드 변경 최소**: 호환 서비스 선택 (Amazon MQ > SQS)
- **운영 오버헤드 최소**: 클라우드 네이티브 (SQS > Amazon MQ)
- 💡 두 키워드가 동시에 나오면 → 코드 변경 최소가 우선 (보통)

### 함정 2: Lift-and-Shift는 최적이 아니다

- "코드 변경 최소" → Lift-and-Shift (EC2) 정답
- "비용 최적화" → Lift-and-Shift ❌ → 리팩토링 (서버리스)
- 💡 문제가 무엇을 요구하는지 정확히 파악

### 함정 3: "엔드포인트만 변경" = "코드 변경 없음"

- DB 연결 문자열 변경 = 코드 변경이 아님 (환경 변수/설정)
- 💡 MySQL → Aurora MySQL은 "코드 변경 없음"으로 분류

---

## 6. 검증 필요 항목 ⚠️

- [ ] Amazon MQ 최신 지원 엔진 (RabbitMQ/ActiveMQ 버전)
- [ ] MSK Serverless 최신 기능
- [ ] AWS MGN (Application Migration Service) 최신 기능
- [ ] Aurora MySQL/PostgreSQL 호환 버전

---

## 7. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "코드 변경 최소" + "DB 전환" → Aurora/RDS
- **유형 2**: "코드 변경 최소" + "메시징 전환" → Amazon MQ/MSK
- **유형 3**: "코드 변경 최소" + "마이그레이션" → Lift-and-Shift
- **유형 4**: "코드 변경 최소" vs "운영 최소" 구분

---

## 8. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
