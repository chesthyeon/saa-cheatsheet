# Chapter 2-5. 운영 오버헤드 최소화 키워드 → 서비스 매핑

## 0. 한 줄 요약

🔑 **"Least operational overhead = 서버리스 우선 + 완전 관리형 서비스 + 자동화 도구(CloudFormation/Systems Manager) — 사람이 개입할 필요 없는 것이 정답"**

---

## 1. 운영 최소화 원칙

| 원칙 | 설명 | 예시 |
|------|------|------|
| **서버리스 우선** | 인프라 프로비저닝/스케일링 제거 | Lambda, Fargate, Aurora Serverless |
| **완전 관리형** | 패치/백업/HA를 AWS에 위임 | RDS, ElastiCache, OpenSearch, MSK |
| **자동화** | 반복 작업 코드화 | CloudFormation, Systems Manager, EventBridge Scheduler |
| **매니지드 통합** | 자체 구현 대신 서비스 활용 | Step Functions, SNS, EventBridge |
| **호환 서비스** | 기존 앱 그대로 사용 | Aurora MySQL, Amazon MQ, MSK |

---

## 2. 서버리스 우선 선택표

| 요구사항 | 서버리스 정답 | 피해야 할 것 |
|---------|-------------|------------|
| API 백엔드 | **Lambda + API Gateway** | EC2 + ALB |
| 컨테이너 | **Fargate** | ECS on EC2 |
| SQL DB (변동) | **Aurora Serverless v2** | RDS |
| NoSQL DB | **DynamoDB** | EC2 + MongoDB |
| ETL | **Glue** | EMR |
| 스트림 → S3 | **Kinesis Firehose** | Kinesis Data Streams + Consumer |
| 워크플로우 | **Step Functions** | 자체 cron/오케스트레이터 |
| 파일 스토리지 | **S3, EFS** | EC2 + NFS 서버 |
| 메시지 큐 | **SQS, SNS, EventBridge** | EC2 + RabbitMQ |
| 인증 | **Cognito** | 자체 인증 서버 |
| 데이터 웨어하우스 | **Redshift Serverless, Athena** | EMR + Presto |
| 로그/검색 | **OpenSearch Serverless** | EC2 + Elasticsearch |
| API 캐싱 | **API Gateway Cache** | 자체 캐시 레이어 |

---

## 3. 자동화 도구

### Infrastructure as Code (IaC)

| 서비스 | 용도 | 키워드 |
|--------|------|--------|
| **CloudFormation** | AWS 리소스 템플릿 | "인프라 코드화", "반복 배포" |
| **CDK** (Cloud Development Kit) | IaC를 코드로 (TypeScript/Python) | "프로그래밍 언어로 인프라" |
| **SAM** (Serverless Application Model) | 서버리스 IaC | "Lambda/API GW 배포" |
| **Terraform** | 멀티 클라우드 IaC | (3rd party) |

### Systems Manager 모음

| 기능 | 역할 | 키워드 |
|------|------|--------|
| **SSM Parameter Store** | 설정 값 저장 | "설정 관리" |
| **SSM Session Manager** | **SSH 없이** EC2 셸 접근 | "Bastion Host 제거", "SSH 키 없이" |
| **SSM Run Command** | EC2에 명령 원격 실행 | "다수 EC2 패치/명령" |
| **SSM Patch Manager** | 자동 OS 패치 | "OS 패치 자동화" |
| **SSM State Manager** | 원하는 상태 유지 | "구성 드리프트 방지" |
| **SSM Automation** | 운영 런북 자동화 | "자동 복구" |
| **SSM Inventory** | 인스턴스 자산 목록 | "소프트웨어 목록" |
| **SSM OpsCenter** | 운영 이슈 중앙 관리 | "알림 통합" |

💡 "**Bastion Host 제거**" + "SSH 없이 EC2 접근" → **SSM Session Manager**  
💡 "**OS 패치 자동화**" → **SSM Patch Manager**

### 모니터링/알림 자동화

| 서비스 | 용도 |
|--------|------|
| **CloudWatch Alarms** | 메트릭 임계값 → 조치 |
| **EventBridge** | 이벤트 기반 자동 트리거 |
| **EventBridge Scheduler** | cron/rate 스케줄 실행 |
| **Auto Scaling** | 자동 확장/축소 |
| **Lambda** | 자동 스크립트 실행 |
| **SNS** | 알림 팬아웃 |

---

## 4. 배포/CI/CD 자동화

| 서비스 | 역할 |
|--------|------|
| **CodeCommit** | Git 저장소 |
| **CodeBuild** | 빌드 (서버리스) |
| **CodeDeploy** | 무중단 배포 (Blue/Green, Canary) |
| **CodePipeline** | CI/CD 파이프라인 오케스트레이션 |
| **Elastic Beanstalk** | 앱 플랫폼 (관리형 PaaS) |
| **App Runner** | 서버리스 웹앱 호스팅 |

### 배포 전략

| 전략 | 설명 | 서비스 |
|------|------|--------|
| **In-place** | 같은 인스턴스 업데이트 | CodeDeploy |
| **Blue/Green** | 새 환경 생성 → 전환 | CodeDeploy, Elastic Beanstalk |
| **Canary** | 일부 트래픽만 신버전 | CodeDeploy (Lambda/ECS) |
| **Rolling** | 점진적 업데이트 | ASG, ECS |

💡 "무중단 배포" → **Blue/Green**  
💡 "신버전 일부만 테스트" → **Canary**

---

## 5. 관리형 서비스 vs 자체 구축

| 자체 구축 (EC2) | 관리형 서비스 | 절감되는 운영 |
|--------------|-------------|-------------|
| EC2 + MySQL | **RDS / Aurora** | 패치, 백업, 페일오버 |
| EC2 + Redis | **ElastiCache** | 클러스터 관리 |
| EC2 + Kafka | **MSK** | ZooKeeper, 브로커 관리 |
| EC2 + Elasticsearch | **OpenSearch** | 마스터 노드 관리 |
| EC2 + Jenkins | **CodePipeline + CodeBuild** | Jenkins 서버 관리 |
| EC2 + RabbitMQ | **Amazon MQ** | 큐 서버 관리 |
| EC2 + Spark | **EMR / Glue** | 클러스터 관리 |
| EC2 + cron | **EventBridge Scheduler / Lambda** | 스케줄러 서버 |
| EC2 + NFS 서버 | **EFS** | 파일 서버 관리 |
| EC2 + 웹 서버 | **App Runner / Elastic Beanstalk** | OS/런타임 |

---

## 6. 시험 빈출 시나리오

### 시나리오 1: "Bastion Host 제거 + EC2 접근"

**정답**: **SSM Session Manager** (SSH 없이 브라우저/CLI 접근)  
**오답 함정**: Bastion Host + SSH (관리 부담)

### 시나리오 2: "수백 대 EC2에 패치 적용"

**정답**: **SSM Patch Manager** (자동 패치)  
**대안**: SSM Run Command (수동 실행)

### 시나리오 3: "예약된 배치 작업 실행"

**정답**: **EventBridge Scheduler** + Lambda  
**오답 함정**: EC2 + cron (서버 관리 필요)

### 시나리오 4: "코드 변경 최소화 + 운영 최소화" + MySQL

**정답**: **Aurora MySQL** (호환 + 서버리스 옵션)  
**오답 함정**: DynamoDB (코드 대규모 변경)

### 시나리오 5: "간단한 웹앱 배포 + 운영 최소화"

**정답**: **App Runner** (서버리스 웹앱, 자동 빌드/배포)  
**대안**: Elastic Beanstalk, Amplify (프론트엔드)

### 시나리오 6: "인프라 반복 배포"

**정답**: **CloudFormation** (템플릿)  
**서버리스**: **SAM**  
**프로그래밍**: **CDK**

### 시나리오 7: "Lambda 블루/그린 배포"

**정답**: **CodeDeploy** (Lambda Canary/Linear)  
**설정**: 트래픽 점진 이동 (10% → 50% → 100%)

### 시나리오 8: "다수 계정 인프라 표준화"

**정답**: **CloudFormation StackSets** + **AWS Control Tower**  
**자동화**: **Organizations** + SCP

### 시나리오 9: "서버리스 웹 앱 모두 포함"

**정답**: **Amplify** (프론트엔드) + **API Gateway + Lambda** (백엔드) + **DynamoDB** + **Cognito**

---

## 7. 빠른 매핑 치트표

| 키워드 | 정답 |
|--------|------|
| "서버 관리 없이" | **Lambda / Fargate / DynamoDB** |
| "Bastion 제거" + "SSH 없이" | **SSM Session Manager** |
| "OS 패치 자동화" | **SSM Patch Manager** |
| "예약 작업" | **EventBridge Scheduler** |
| "인프라 코드" | **CloudFormation / CDK** |
| "서버리스 IaC" | **SAM** |
| "CI/CD" + "서버리스" | **CodePipeline + CodeBuild + CodeDeploy** |
| "간단한 웹앱 호스팅" | **App Runner / Elastic Beanstalk / Amplify** |
| "Blue/Green 배포" | **CodeDeploy** |
| "다수 계정 표준화" | **Control Tower + StackSets** |
| "ETL 서버리스" | **Glue** |
| "SQL 쿼리 서버리스" | **Athena** |
| "컨테이너 서버리스" | **Fargate** |

---

## 8. 함정 / 주의사항

### 함정 1: "운영 오버헤드 최소" ≠ "가장 저렴"

- 서버리스는 **운영 부담 적음**이지만 대량 고정 워크로드에선 비쌈
- 💡 "비용 최적화"와 충돌 시: 문제의 **주 요구사항** 확인

### 함정 2: Elastic Beanstalk vs App Runner

- **Elastic Beanstalk**: EC2 기반 (환경 관리 필요, 더 유연)
- **App Runner**: 완전 서버리스 (**더 간단**, 컨테이너 자동 빌드)
- 💡 "최소 오버헤드 웹앱" → **App Runner**

### 함정 3: SSM Session Manager 장점

- SSH 키 관리 ❌
- 22번 포트 개방 ❌
- Bastion Host ❌
- CloudTrail 감사 ✅
- IAM 권한으로 제어 ✅
- 💡 "보안 강화" + "운영 간소화" → **Session Manager**

### 함정 4: EventBridge vs EventBridge Scheduler

- **EventBridge (Rules)**: 이벤트 기반 (AWS 서비스 상태 변경)
- **EventBridge Scheduler**: **시간 기반** 스케줄링 (cron의 현대적 대체)
- 💡 "cron 대체" → EventBridge **Scheduler**

---

## 9. 검증 필요 항목 ⚠️

- [ ] App Runner 최신 지원 런타임
- [ ] SSM Session Manager 최신 기능
- [ ] EventBridge Scheduler vs CloudWatch Events cron 권장 여부
- [ ] CDK v2 최신 지원 언어
- [ ] AWS Control Tower Landing Zone 최신 가이드

---

## 10. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "운영 오버헤드 최소화" → 서버리스 조합
- **유형 2**: "Bastion 제거" → SSM Session Manager
- **유형 3**: "OS 패치 자동화" → SSM Patch Manager
- **유형 4**: "인프라 반복 배포" → CloudFormation / StackSets
- **유형 5**: "서버리스 웹앱" → App Runner / Amplify

---

## 11. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
