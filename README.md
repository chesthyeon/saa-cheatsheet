# AWS SAA-C03 치트시트

> 2026년 7월 채용 시즌 대비 AWS Solutions Architect Associate 학습 자료

## 진행 상태

| 상태 | 의미 |
|------|------|
| ✅ | 작성 완료 |
| 🔄 | 작성 중 |
| ⬜ | 미작성 |

## Chapter 1. 도메인별 핵심 서비스 키워드

| # | 챕터 | 상태 |
|---|------|------|
| 1-1 | [컴퓨팅 및 컨테이너](11_keyword_mapping/ch1-1_compute_container.md) | ✅ |
| 1-2 | [스토리지](11_keyword_mapping/ch1-2_storage.md) | ✅ |
| 1-3 | [데이터베이스 및 캐시](11_keyword_mapping/ch1-3_database_cache.md) | ✅ |
| 1-4 | [네트워크 및 콘텐츠 전송](11_keyword_mapping/ch1-4_network_cdn.md) | ✅ |
| 1-5 | [애플리케이션 통합 및 분석](11_keyword_mapping/ch1-5_integration_analytics.md) | ✅ |
| 1-6 | [특수 목적 서비스 (AI/ML, 마이그레이션, 비용 도구)](11_keyword_mapping/ch1-6_special_purpose.md) | ✅ |
| 1-7 | [자격 증명 및 거버넌스 (IAM, Organizations, STS)](11_keyword_mapping/ch1-7_iam_governance.md) | ✅ |

## Chapter 3. 헷갈리는 쌍 완전 정리

| # | 챕터 | 상태 |
|---|------|------|
| 3-1 | [컴퓨팅: EC2 vs Lambda vs Fargate vs ECS vs EKS vs App Runner](13_comparison/ch3-1_compute.md) | ✅ |
| 3-2 | [로드밸런서: ALB vs NLB vs GLB vs CLB](13_comparison/ch3-2_load_balancer.md) | ✅ |
| 3-3 | [스토리지: S3 클래스 / EBS vs EFS vs FSx 4종](13_comparison/ch3-3_storage.md) | ✅ |
| 3-4 | [데이터베이스: RDS vs Aurora vs DynamoDB vs ElastiCache vs Redshift](13_comparison/ch3-4_database.md) | ✅ |
| 3-5 | [메시징: SQS vs SNS vs EventBridge vs Kinesis vs Step Functions](13_comparison/ch3-5_messaging.md) | ✅ |
| 3-6 | [네트워크: VPN vs Direct Connect vs Transit Gateway vs Peering vs PrivateLink](13_comparison/ch3-6_network.md) | ✅ |
| 3-7 | 보안: KMS vs CloudHSM / ACM vs Secrets Manager vs Parameter Store | ⬜ |
| 3-8 | 모니터링: CloudWatch vs CloudTrail vs Config vs X-Ray | ⬜ |
| 3-9 | IAM: 역할 vs 사용자 vs 리소스 기반 정책 vs SCP vs Permission Boundary | ⬜ |

## Chapter 4. 단골 아키텍처 시나리오 템플릿

| # | 챕터 | 상태 |
|---|------|------|
| 4-1 | 정적 웹사이트 호스팅 (S3 + CloudFront + Route 53 + ACM) | ⬜ |
| 4-2 | 3-Tier 웹 애플리케이션 (ALB + ASG + Multi-AZ RDS) | ⬜ |
| 4-3 | 완전 서버리스 (API Gateway + Lambda + DynamoDB + Cognito) | ⬜ |
| 4-4 | 마이크로서비스·디커플링 (API Gateway + SQS/SNS + Lambda/Fargate) | ⬜ |
| 4-5 | 데이터 레이크 + 분석 (S3 + Kinesis + Glue + Athena + QuickSight) | ⬜ |
| 4-6 | 재해 복구 (DR) 4대 전략 | ⬜ |
| 4-7 | 하이브리드 클라우드 (Direct Connect + Transit Gateway + Storage Gateway) | ⬜ |
| 4-8 | Cross-Account 액세스 패턴 (IAM Role + STS AssumeRole) | ⬜ |

## Chapter 5. 자주 틀리는 함정 유형

| # | 챕터 | 상태 |
|---|------|------|
| 5-1 | "가장 저렴한(Cheapest)" vs "운영 오버헤드가 적은(Least Overhead)" | ⬜ |
| 5-2 | 고가용성 vs 내구성 vs 내결함성 | ⬜ |
| 5-3 | 보안 그룹(Stateful) vs NACL(Stateless) | ⬜ |
| 5-4 | S3 버킷 정책 vs IAM 정책 우선순위 / 명시적 Deny | ⬜ |
| 5-5 | 리전 vs 글로벌 서비스 구분 | ⬜ |
| 5-6 | "애플리케이션 코드 변경 최소화" 키워드 | ⬜ |
| 5-7 | "관리형(Managed)" vs "서버리스(Serverless)" 혼동 | ⬜ |

## Chapter 2. 문제 요구사항(목적)별 정답 패턴

| # | 챕터 | 상태 |
|---|------|------|
| 2-1 | 비용 최적화 키워드 → 서비스 매핑 | ⬜ |
| 2-2 | 고가용성·내결함성 키워드 → 서비스 매핑 | ⬜ |
| 2-3 | 보안·암호화 키워드 → 서비스 매핑 | ⬜ |
| 2-4 | 성능·확장성 키워드 → 서비스 매핑 | ⬜ |
| 2-5 | 운영 오버헤드 최소화 키워드 → 서비스 매핑 | ⬜ |
| 2-6 | 마이그레이션 세분화 (DataSync vs Snow Family vs DMS/SCT vs AMS) | ⬜ |
| 2-7 | 느슨한 결합·이벤트 기반 키워드 → 서비스 매핑 | ⬜ |
| 2-8 | 실시간 처리·스트리밍 키워드 → 서비스 매핑 | ⬜ |

## Chapter 0. AWS Well-Architected Framework 6대 원칙

| # | 챕터 | 상태 |
|---|------|------|
| 0-1 | 운영 우수성 (Operational Excellence) | ⬜ |
| 0-2 | 보안 (Security) | ⬜ |
| 0-3 | 안정성 (Reliability) | ⬜ |
| 0-4 | 성능 효율성 (Performance Efficiency) | ⬜ |
| 0-5 | 비용 최적화 (Cost Optimization) | ⬜ |
| 0-6 | 지속 가능성 (Sustainability) | ⬜ |

## Chapter 6. 오답 노트

| # | 챕터 | 상태 |
|---|------|------|
| 6-1 | 오답 노트 활용 가이드 | ⬜ |
| 6-2 | 오답 패턴 분석 (월별 / 챕터별) | ⬜ |
| 6-3 | 시험 직전 1주일 복습 전략 | ⬜ |

---

## 참고 문서

- [작성 규약](00_규약.md) — 모든 챕터의 작성 기준
- [전체 목차](01_v2_목차.md) — v2 상세 목차

## 작성 권장 순서 (체득 효율 순)

> Chapter 번호 순이 아닌, 학습 효율 순서로 작성합니다.

1. **Chapter 1** (키워드 매핑) — 전체 그림 ✅
2. **Chapter 3** (헷갈리는 쌍) — 1을 보강
3. **Chapter 4** (시나리오 템플릿) — 1+3 응용
4. **Chapter 5** (함정) — 4까지 본 뒤 정리
5. **Chapter 2** (목적별 패턴) — 1~4의 재구성
6. **Chapter 0** (Well-Architected) — 시험 직전 정독용
