# Chapter 1-1. 컴퓨팅 및 컨테이너

## 0. 한 줄 요약

🔑 **"서버 관리 수준"이 선택 기준이다 — 직접 관리(EC2) → 컨테이너 위임(ECS/EKS) → 완전 서버리스(Lambda/Fargate/App Runner) → 배치 최적화(Batch)**

---

## 1. 핵심 서비스 표

| 서비스 | 한 줄 정의 | 주요 키워드 | 가격 모델 |
|--------|-----------|------------|----------|
| **EC2** | 가상 서버 (IaaS), OS~앱 전체 제어 | 인스턴스 타입, AMI, 보안 그룹, EBS, Placement Group | On-Demand / Reserved / Savings Plans / Spot |
| **Lambda** | 이벤트 기반 서버리스 함수 (FaaS) | 이벤트 트리거, 15분 제한, 동시성, 콜드 스타트 | 요청 수 + 실행 시간(ms) × 메모리 |
| **ECS** | AWS 네이티브 컨테이너 오케스트레이션 | Task Definition, Service, 클러스터 | EC2 시작 유형: EC2 비용 / Fargate 시작 유형: vCPU+메모리 |
| **EKS** | 관리형 Kubernetes | Pod, Node Group, kubectl, Helm | 클러스터 $0.10/hr + 워커 노드(EC2 또는 Fargate) |
| **Fargate** | ECS/EKS용 서버리스 컴퓨팅 엔진 | 서버리스 컨테이너, 인프라 관리 불필요 | vCPU + 메모리 (초 단위) |
| **App Runner** | 소스 코드/이미지 → 즉시 배포 | 완전관리형, 자동 스케일링, 빌드+배포 자동화 | vCPU + 메모리 (활성 시간) |
| **Batch** | 대규모 배치 작업 스케줄링·실행 | 배치 처리, 작업 큐, Spot 인스턴스 활용 | 기본 무료 (사용한 EC2/Fargate 비용만) |

---

## 2. 키워드 → 서비스 매핑

### EC2가 정답인 키워드

| 시험 키워드 | 왜 EC2인가 |
|------------|-----------|
| "특정 OS/커널 설정 필요" | Lambda/Fargate는 OS 접근 불가 |
| "GPU 인스턴스", "고성능 컴퓨팅(HPC)" | P/G 인스턴스 타입, Placement Group(Cluster) |
| "라이선스가 물리 서버에 바인딩(BYOL)" | Dedicated Host로 소켓/코어 수준 제어 |
| "상태 유지(Stateful) 애플리케이션" | 인스턴스에 로컬 상태 저장 가능 |
| "비용 절감 + 중단 허용" | Spot Instance (최대 90% 할인) |
| "예측 가능한 장기 워크로드" | Reserved Instance / Savings Plans |
| "Placement Group" | Cluster(저지연), Spread(고가용성), Partition(대규모 분산) |

### Lambda가 정답인 키워드

| 시험 키워드 | 왜 Lambda인가 |
|------------|-------------|
| "이벤트 기반", "트리거" | S3 이벤트, API Gateway, DynamoDB Streams 등 |
| "서버리스", "인프라 관리 없이" | 서버 프로비저닝 불필요 |
| "간헐적 트래픽", "요청 기반 과금" | 사용한 만큼만 과금, 유휴 비용 0 |
| "짧은 실행 시간" (수 초~수 분) | 최대 15분, 그 이상은 다른 서비스 |
| "자동 스케일링, 동시성" | 요청 단위 자동 확장 |
| "운영 오버헤드 최소화" + 짧은 작업 | 관리 포인트가 거의 없음 |
| "cron 작업", "스케줄 실행" | EventBridge 스케줄 + Lambda |

### ECS가 정답인 키워드

| 시험 키워드 | 왜 ECS인가 |
|------------|-----------|
| "컨테이너", "Docker" | AWS 네이티브 컨테이너 오케스트레이션 |
| "마이크로서비스" + AWS 친화적 | ECS Service Discovery, ALB 타겟 그룹 연동 |
| "기존 EC2 인프라 위에 컨테이너" | ECS on EC2 시작 유형 |
| "컨테이너" + "서버리스" | ECS on Fargate |

### EKS가 정답인 키워드

| 시험 키워드 | 왜 EKS인가 |
|------------|-----------|
| "Kubernetes", "K8s", "kubectl" | 관리형 K8s 컨트롤 플레인 |
| "온프레미스 K8s를 AWS로 마이그레이션" | 기존 K8s 매니페스트 그대로 사용 |
| "멀티 클라우드 컨테이너 오케스트레이션" | K8s는 클라우드 중립적 표준 |
| "EKS Anywhere" | 온프레미스에서도 EKS 운영 |

### Fargate가 정답인 키워드

| 시험 키워드 | 왜 Fargate인가 |
|------------|--------------|
| "서버리스 컨테이너" | 인스턴스 관리 없이 컨테이너 실행 |
| "컨테이너" + "운영 오버헤드 최소화" | EC2 패치/스케일링 불필요 |
| "컨테이너" + "인프라 관리 없이" | Fargate가 인프라 추상화 |
| "ECS/EKS" + "서버리스" | Fargate는 ECS/EKS 둘 다 지원 |

### App Runner가 정답인 키워드

| 시험 키워드 | 왜 App Runner인가 |
|------------|-----------------|
| "소스 코드에서 바로 배포" | GitHub 연동, 빌드~배포 자동화 |
| "컨테이너 이미지 → 즉시 실행" | ECR 이미지 직접 배포 |
| "가장 간단한 컨테이너 배포" | ECS/EKS보다 설정이 훨씬 적음 |
| "웹 앱/API" + "완전 관리형" | 로드밸런싱, TLS, 스케일링 자동 |

### Batch가 정답인 키워드

| 시험 키워드 | 왜 Batch인가 |
|------------|------------|
| "대규모 배치 처리", "수천 개 작업" | 작업 큐 + 자동 스케줄링 |
| "배치" + "비용 최적화" | Spot Instance 자동 활용 |
| "HPC", "과학 시뮬레이션", "렌더링" | 대규모 병렬 처리에 최적화 |
| "배치" + "Docker 컨테이너" | 컨테이너 기반 배치 실행 지원 |

---

## 3. 헷갈리는 포인트 / 함정 (최소 3개)

### 함정 1: "서버리스" ≠ Lambda만

- ❌ "서버리스"라고 하면 무조건 Lambda → **틀림**
- ✅ Fargate, App Runner, Aurora Serverless, DynamoDB 등도 서버리스
- 💡 **판별 기준**: "서버리스" + "컨테이너" → Fargate, "서버리스" + "짧은 함수" → Lambda

### 함정 2: Lambda 15분 제한

- ❌ "실행 시간이 30분 이상" 워크로드에 Lambda 선택 → **틀림**
- ✅ Lambda 최대 실행 시간: **15분 (900초)**
- 💡 15분 초과 → Step Functions로 Lambda 체이닝, 또는 ECS/Fargate/Batch로 전환

### 함정 3: "비용 최적화" vs "운영 오버헤드 최소화"

- ❌ 이 둘을 같은 의미로 해석 → **자주 틀리는 함정**
- ✅ **비용 최적화**: Spot Instance, Reserved Instance, Savings Plans → EC2 기반이 더 저렴할 수 있음
- ✅ **운영 오버헤드 최소화**: Lambda, Fargate, App Runner → 관리 포인트가 적은 서비스
- 💡 "가장 저렴한" ≠ "가장 관리가 편한"

### 함정 4: ECS on EC2 vs ECS on Fargate

- ❌ "ECS는 서버리스" → **시작 유형에 따라 다름**
- ✅ ECS on EC2: 사용자가 EC2 인스턴스 관리 (패치, 스케일링)
- ✅ ECS on Fargate: 인프라 관리 불필요 (서버리스)
- 💡 문제에서 "ECS" + "서버 관리 없이" → Fargate 시작 유형

### 함정 5: ECS vs EKS 선택 기준

- ❌ "컨테이너면 무조건 ECS" 또는 "무조건 EKS" → **맥락에 따라 다름**
- ✅ **기존 K8s 워크로드/팀 경험** → EKS
- ✅ **AWS 네이티브 + 간단한 구성** → ECS
- ✅ **멀티 클라우드 이식성** → EKS
- 💡 시험에서 "Kubernetes"라는 단어가 명시되면 → EKS가 거의 확정

### 함정 6: Spot Instance 중단 가능성

- ❌ "비용 절감"만 보고 Spot 선택 → **중단 허용 여부 확인 필수**
- ✅ Spot은 **2분 전 경고** 후 중단 가능
- ✅ 상태 유지가 중요한 DB, 실시간 서비스 → Spot 부적합
- 💡 "중단 허용", "내결함성 있는 워크로드" → Spot 적합

---

## 4. 단골 시나리오

### 시나리오 1: 트래픽 변동이 큰 웹 애플리케이션

```
문제 패턴: "피크 시 트래픽이 10배 증가, 비용 효율적으로..."
정답 패턴: EC2 Auto Scaling Group + ALB
          - 기본: On-Demand 또는 Reserved (베이스라인)
          - 피크: Spot Instance (Fleet 혼합)
키 포인트: Launch Template + 혼합 인스턴스 정책
```

### 시나리오 2: 이미지 업로드 후 자동 처리

```
문제 패턴: "S3에 이미지 업로드 → 자동으로 썸네일 생성"
정답 패턴: S3 Event Notification → Lambda
키 포인트: 이벤트 기반 + 짧은 실행 시간 + 서버리스 = Lambda
```

### 시나리오 3: 레거시 모놀리스 → 마이크로서비스 전환

```
문제 패턴: "기존 애플리케이션을 컨테이너화, 운영 부담 최소화"
정답 패턴: ECS on Fargate + ALB
함정 주의: "운영 부담 최소화"가 있으면 ECS on EC2가 아니라 Fargate
```

### 시나리오 4: 온프레미스 K8s → AWS 마이그레이션

```
문제 패턴: "기존 Kubernetes 워크로드를 최소 변경으로 AWS로"
정답 패턴: EKS (기존 매니페스트 재사용)
함정 주의: "최소 변경"이 핵심 → ECS가 아니라 EKS
```

### 시나리오 5: 대규모 유전체 분석 / 렌더링

```
문제 패턴: "수천 개 작업을 병렬 처리, 비용 최적화"
정답 패턴: AWS Batch + Spot Instance
키 포인트: Batch가 Spot 관리 + 작업 큐 자동 처리
```

### 시나리오 6: 간단한 웹 API 빠르게 배포

```
문제 패턴: "개발팀이 소스 코드만 푸시하면 자동 배포, 인프라 걱정 없이"
정답 패턴: App Runner
키 포인트: "가장 간단한 배포" + "완전 관리형" = App Runner
          (Elastic Beanstalk보다 더 간단)
```

---

## 5. 검증 필요 항목 ⚠️

> Claude의 학습 데이터가 outdated일 수 있는 항목입니다. AWS 공식 문서에서 확인하세요.

- [ ] Lambda 최대 메모리: 현재 10,240 MB (10 GB)인지 확인
  - 공식 문서: https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html
- [ ] Lambda 동시 실행 기본 한도: 리전당 1,000인지 확인 (상향 요청 가능)
- [ ] Fargate vCPU/메모리 조합 한도 최신 값 확인
  - 공식 문서: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-cpu-memory-error.html
- [ ] App Runner 최대 동시 요청 수, 인스턴스 크기 한도 확인
  - 공식 문서: https://docs.aws.amazon.com/apprunner/latest/dg/architecture.html
- [ ] EKS 클러스터 비용: $0.10/hr인지 확인 (가격 변동 가능)
  - 공식 문서: https://aws.amazon.com/eks/pricing/
- [ ] EC2 Savings Plans vs Reserved Instance 최신 할인율 비교
- [ ] Batch의 Fargate 지원 범위 (GPU 작업 등) 최신 상태 확인

---

## 6. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "서버리스로 전환" + "운영 오버헤드 최소화" → Lambda 또는 Fargate
- **유형 2**: "비용 최적화" + "중단 허용" → EC2 Spot Instance
- **유형 3**: "기존 Kubernetes" + "AWS 마이그레이션" → EKS
- **유형 4**: "컨테이너" + "서버 관리 없이" → ECS on Fargate
- **유형 5**: "대규모 배치 처리" + "비용 효율" → AWS Batch + Spot
- **유형 6**: "이벤트 기반" + "자동 실행" + "짧은 작업" → Lambda
- **유형 7**: "BYOL 라이선스" + "물리 서버 제어" → EC2 Dedicated Host
- **유형 8**: "소스 코드에서 바로 배포" + "가장 간단한 방법" → App Runner
- **유형 9**: "HPC" + "저지연 네트워킹" → EC2 Placement Group (Cluster)
- **유형 10**: "예측 가능한 장기 워크로드" + "비용 절감" → Reserved Instance / Savings Plans

---

## 7. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
