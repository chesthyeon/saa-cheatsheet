# Chapter 0-1. 운영 우수성 (Operational Excellence)

## 0. 한 줄 요약

🔑 **"운영 우수성 = 코드로 운영하고(IaC), 작고 자주 변경하며, 실패에서 배우고, 자동화로 사람 실수를 제거하는 것"**

---

## 1. 핵심 설계 원칙 (6가지)

| # | 원칙 | 설명 | AWS 서비스 예시 |
|---|------|------|---------------|
| 1 | **Operations as Code** | 인프라·운영 절차를 코드로 정의 | CloudFormation, CDK, SAM |
| 2 | **Make frequent, small, reversible changes** | 소규모 배포로 롤백 용이 | CodeDeploy (Blue/Green), CloudFormation Rollback |
| 3 | **Refine operations procedures frequently** | 운영 절차를 지속적으로 개선 | SSM Runbook, Automation |
| 4 | **Anticipate failure** | 장애를 미리 예상하고 테스트 | FIS (Fault Injection Simulator) |
| 5 | **Learn from all operational failures** | 장애 후 개선 (Post-mortem) | CloudWatch Dashboards, X-Ray |
| 6 | **Use managed services** | 관리형 서비스로 운영 부담↓ | RDS, Aurora, Lambda, Fargate |

---

## 2. 키워드 → 서비스 매핑

| 시험 키워드 | 정답 서비스 |
|------------|-----------|
| "운영 오버헤드 최소화" / "least operational overhead" | **서버리스** (Lambda, Fargate, Aurora Serverless, DynamoDB) |
| "자동화된 배포" / "automated deployment" | **CodePipeline + CodeDeploy** |
| "인프라를 코드로" / "IaC" | **CloudFormation** / CDK / SAM |
| "롤백 가능한 배포" / "rollback" | **CodeDeploy Blue/Green** / CloudFormation Rollback |
| "Bastion 없이 EC2 접속" | **SSM Session Manager** |
| "패치 자동화" | **SSM Patch Manager** |
| "운영 절차 자동화" / "runbook" | **SSM Automation / Runbook** |
| "장애 시뮬레이션" / "chaos engineering" | **FIS (Fault Injection Simulator)** |
| "변경 추적" / "drift detection" | **CloudFormation Drift Detection** / **AWS Config** |
| "중앙 집중 로깅" | **CloudWatch Logs** + **CloudTrail** |
| "애플리케이션 성능 추적" | **X-Ray** |
| "대시보드 / 알림" | **CloudWatch Dashboards + Alarms** |
| "이벤트 기반 자동 대응" | **EventBridge → Lambda / SSM Automation** |

---

## 3. 4대 영역별 핵심 서비스

### 3-1. 준비 (Prepare)

| 서비스 | 역할 |
|--------|------|
| **CloudFormation / CDK** | IaC — 인프라 정의·버전 관리 |
| **AWS Config** | 리소스 구성 규칙 준수 확인 |
| **AWS Config Rules** | 규정 위반 자동 탐지 |
| **SSM Document** | 운영 절차 문서화 |

### 3-2. 운영 (Operate)

| 서비스 | 역할 |
|--------|------|
| **CloudWatch** | 메트릭/로그/알람 (모니터링 허브) |
| **CloudTrail** | API 호출 감사 (누가 무엇을 했나) |
| **X-Ray** | 분산 추적 (어디서 느린가) |
| **SSM Session Manager** | Bastion 없는 EC2 접속 |
| **SSM Patch Manager** | OS 패치 자동화 |
| **EventBridge** | 이벤트 기반 자동 대응 |

### 3-3. 발전 (Evolve)

| 서비스 | 역할 |
|--------|------|
| **CodePipeline** | CI/CD 파이프라인 |
| **CodeDeploy** | Blue/Green, Canary, Rolling 배포 |
| **FIS** | 장애 주입 테스트 (Chaos Engineering) |
| **CloudFormation StackSets** | 멀티 계정·멀티 리전 일괄 배포 |

### 3-4. 최소 권한 운영

| 도구 | 용도 |
|------|------|
| **SSM Parameter Store** | 설정 값 중앙 관리 (암호화 옵션) |
| **Secrets Manager** | DB 자격 증명 자동 로테이션 |
| **Systems Manager** | 운영 허브 (Inventory, Compliance, OpsCenter) |

---

## 4. 시험 빈출 시나리오

### 시나리오 1: "EC2에 SSH 없이 접속, 보안 그룹 22번 포트 열지 않음"

**정답**: **SSM Session Manager** (IAM 권한만으로 접속, CloudTrail 감사)  
**오답**: Bastion Host + SG 22 오픈

### 시나리오 2: "인프라 변경 시 드리프트 감지 + 자동 교정"

**정답**: **AWS Config Rules** + **SSM Automation** (자동 교정)  
또는 **CloudFormation Drift Detection** (탐지만)

### 시나리오 3: "배포 실패 시 자동 롤백"

**정답**: **CodeDeploy Blue/Green** (헬스 체크 실패 → 자동 이전 버전으로)  
또는 **CloudFormation Rollback** (스택 업데이트 실패 시)

### 시나리오 4: "운영 부담 최소화 + DB 관리"

**정답**: **Aurora Serverless** (자동 스케일링, 관리 최소) 또는 **DynamoDB**  
**오답**: EC2에 자체 DB 설치

### 시나리오 5: "장애 복원력 테스트"

**정답**: **AWS FIS** (Fault Injection Simulator)  
- EC2 중지, AZ 장애 시뮬레이션, 네트워크 지연 주입

### 시나리오 6: "여러 계정에 동일 인프라 배포"

**정답**: **CloudFormation StackSets** + **AWS Organizations**

### 시나리오 7: "Lambda 함수 배포 자동화 (서버리스)"

**정답**: **SAM (Serverless Application Model)** + **CodePipeline**  
또는 CDK

### 시나리오 8: "EC2 인스턴스 패치 + 컴플라이언스 보고"

**정답**: **SSM Patch Manager** + **SSM Compliance**

---

## 5. 함정 / 주의사항

### 함정 1: "Monitoring" ≠ "Observability"

- **모니터링**: CloudWatch 메트릭/알람 (정해진 지표 감시)
- **관찰 가능성(Observability)**: CloudWatch + X-Ray + CloudTrail (메트릭 + 트레이스 + 로그) **3축**
- 💡 시험에서 "observability" → X-Ray 포함 여부 확인

### 함정 2: SSM Session Manager vs Bastion Host

- Session Manager: **포트 22 불필요**, 키 페어 불필요, CloudTrail 감사
- 시험에서 "**가장 안전한(most secure) EC2 접속**" → **Session Manager**
- Bastion은 아직 정답이 될 수 있음 (Private Subnet 접근 경유지로)
- 💡 "SSH 키 관리 부담 없이" / "포트 22 닫기" → Session Manager

### 함정 3: CloudFormation vs Terraform

- SAA 시험은 **AWS 서비스만** 다룸 → 정답은 항상 **CloudFormation / CDK / SAM**
- Terraform은 오답 보기로 등장 가능 (AWS 네이티브 아님)
- 💡 IaC 키워드 → CloudFormation 먼저 고려

### 함정 4: CodeDeploy 배포 전략 혼동

| 전략 | 설명 | 롤백 |
|------|------|------|
| **In-place** | 기존 인스턴스에 새 코드 | 느림 (재배포) |
| **Blue/Green** | 새 인스턴스 세트 생성 → 전환 | **빠름** (이전 세트로 전환) |
| **Canary** | 소수(10%)에 먼저 → 전체 | 빠름 |
| **Linear** | 일정 비율씩 점진 | 중간 |

💡 "**최소 다운타임 + 빠른 롤백**" → **Blue/Green**

### 함정 5: "자동화" 키워드의 다양한 해석

- 인프라 자동화 → CloudFormation/CDK
- 배포 자동화 → CodePipeline/CodeDeploy
- 운영 절차 자동화 → SSM Automation/Runbook
- 이벤트 대응 자동화 → EventBridge → Lambda
- 💡 "무엇을" 자동화하는지에 따라 서비스가 다름

---

## 6. 검증 필요 항목 ⚠️

- [ ] FIS 지원 장애 유형 최신 목록 (AZ 장애 시뮬레이션 GA 여부)
- [ ] SSM Session Manager Advanced Tier 최신 기능
- [ ] CodeDeploy + ECS Blue/Green 최신 지원 범위
- [ ] CloudFormation StackSets 자동 배포 최대 계정/리전 수
- [ ] CDK vs SAM 최신 권장 사항 (AWS 공식 입장 변화?)

---

## 7. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "운영 오버헤드 최소" → 서버리스 서비스 선택
- **유형 2**: "SSH 없이 EC2 접속" → SSM Session Manager
- **유형 3**: "IaC / 인프라 자동화" → CloudFormation / CDK
- **유형 4**: "배포 롤백 / 다운타임 최소" → CodeDeploy Blue/Green
- **유형 5**: "장애 복원력 테스트" → FIS
- **유형 6**: "패치 자동화" → SSM Patch Manager

---

## 8. 학습 메모 (Banghyeon 작성 영역)

- (아직 기록 없음)
