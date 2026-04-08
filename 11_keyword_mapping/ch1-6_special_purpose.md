# Chapter 1-6. 특수 목적 서비스 (AI/ML, 마이그레이션, 비용 도구)

## 0. 한 줄 요약

🔑 **SAA 시험에서 이 영역은 "깊이"가 아니라 "어떤 서비스가 존재하는지"만 알면 된다 — 서비스명 + 한 줄 용도를 매핑하는 것이 핵심**

---

## 1. 핵심 서비스 표

### AI/ML 서비스

| 서비스 | 한 줄 정의 | 시험 출제 키워드 |
|--------|-----------|----------------|
| **SageMaker** | ML 모델 빌드·학습·배포 통합 플랫폼 | "ML 모델 학습", "커스텀 모델" |
| **Rekognition** | 이미지/비디오 분석 (얼굴, 객체, 텍스트) | "이미지 분석", "얼굴 인식", "콘텐츠 모더레이션" |
| **Transcribe** | 음성 → 텍스트 변환 (STT) | "음성 인식", "자막 생성" |
| **Polly** | 텍스트 → 음성 변환 (TTS) | "음성 합성", "텍스트 읽어주기" |
| **Translate** | 실시간 언어 번역 | "다국어 번역" |
| **Comprehend** | 자연어 처리 (NLP), 감정 분석 | "텍스트 감정 분석", "키워드 추출" |
| **Lex** | 대화형 챗봇 엔진 | "챗봇", "음성 봇" (Alexa 기반) |
| **Textract** | 문서에서 텍스트/표/양식 추출 | "문서 OCR", "양식 데이터 추출" |
| **Forecast** | 시계열 예측 | "수요 예측", "매출 예측" |
| **Personalize** | 실시간 개인화 추천 | "추천 엔진", "개인화" |
| **Kendra** | 지능형 엔터프라이즈 검색 | "사내 문서 검색", "지식 검색" |

> 💡 **시험 팁**: SAA에서 AI/ML은 "이 서비스가 뭘 하는지" 수준. 깊은 구현 문제는 안 나온다.

### 마이그레이션 서비스

| 서비스 | 한 줄 정의 | 시험 출제 키워드 |
|--------|-----------|----------------|
| **DMS** (Database Migration Service) | DB 마이그레이션 (동종/이종) | "DB 마이그레이션", "지속적 복제(CDC)" |
| **SCT** (Schema Conversion Tool) | 이종 DB 스키마 변환 | "Oracle → Aurora 스키마 변환" |
| **Application Migration Service (MGN)** | 서버 리프트 앤 시프트 마이그레이션 | "서버 마이그레이션", "리호스팅" |
| **DataSync** | 온프레미스 ↔ AWS 데이터 전송 (NFS/SMB → S3/EFS/FSx) | "대용량 파일 전송", "NFS → S3" |
| **Transfer Family** | SFTP/FTPS/FTP → S3/EFS | "SFTP 서버", "FTP → S3" |
| **Snow Family** | 물리 디바이스로 대용량 데이터 이동 | "오프라인 전송", "네트워크 대역폭 부족" |
| **Migration Hub** | 마이그레이션 진행 상황 중앙 추적 | "마이그레이션 추적 대시보드" |

### Snow Family 세부

| 디바이스 | 용량 | 사용 사례 |
|----------|------|----------|
| **Snowcone** | 8 TB HDD / 14 TB SSD | 엣지 컴퓨팅, 소규모 전송 |
| **Snowball Edge Storage Optimized** | 80 TB | 대규모 데이터 전송 |
| **Snowball Edge Compute Optimized** | 42 TB + GPU | 엣지 컴퓨팅 + 전송 |
| **Snowmobile** | 100 PB (컨테이너 트럭) | 엑사바이트급 전송 |

> 💡 **판별 기준**: 데이터 크기 + 네트워크 상황으로 선택. "수십 TB 이상 + 네트워크 느림" → Snow Family

### 비용 관리 도구

| 서비스 | 한 줄 정의 | 시험 출제 키워드 |
|--------|-----------|----------------|
| **Cost Explorer** | 비용 시각화 및 분석 | "비용 분석", "비용 추세 확인" |
| **Budgets** | 예산 설정 + 알림 | "예산 초과 알림", "비용 경고" |
| **Cost and Usage Report (CUR)** | 가장 상세한 비용 데이터 (S3로 내보내기) | "상세 비용 보고서", "항목별 비용" |
| **Compute Optimizer** | EC2/Lambda/EBS 최적 사양 추천 | "인스턴스 사양 최적화", "오버프로비저닝" |
| **Trusted Advisor** | 비용/성능/보안/내결함성 자동 검사 | "비용 절감 권장", "보안 점검" |
| **Savings Plans** | 컴퓨팅 사용량 약정 할인 (EC2/Lambda/Fargate) | "장기 약정 할인", "유연한 할인" |
| **Reserved Instances** | 특정 인스턴스 타입 약정 할인 | "특정 인스턴스 예약", "1년/3년 약정" |

---

## 2. 키워드 → 서비스 매핑

### AI/ML 키워드 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "이미지에서 얼굴 감지" | **Rekognition** |
| "음성을 텍스트로" | **Transcribe** |
| "텍스트를 음성으로" | **Polly** |
| "문서에서 데이터 추출", "OCR" | **Textract** |
| "챗봇 구축" | **Lex** |
| "텍스트 감정 분석" | **Comprehend** |
| "다국어 번역" | **Translate** |
| "수요/매출 예측" | **Forecast** |
| "개인화 추천" | **Personalize** |
| "사내 문서 검색" | **Kendra** |
| "커스텀 ML 모델 학습" | **SageMaker** |

### 마이그레이션 키워드 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "DB 마이그레이션" (동종) | **DMS** |
| "Oracle → Aurora" (이종 DB) | **DMS + SCT** |
| "지속적 복제(CDC)", "DB 동기화" | **DMS** (CDC 모드) |
| "서버 리호스팅", "리프트 앤 시프트" | **Application Migration Service (MGN)** |
| "대용량 파일 NFS/SMB → S3/EFS" | **DataSync** |
| "SFTP 서버 → S3" | **Transfer Family** |
| "네트워크 대역폭 부족" + "수십 TB 이상" | **Snow Family** |
| "페타바이트 전송" | **Snowball Edge** (복수) 또는 **Snowmobile** |
| "엣지 컴퓨팅" + "데이터 전송" | **Snowball Edge Compute Optimized** |
| "마이그레이션 진행 추적" | **Migration Hub** |

### DataSync vs Storage Gateway vs Snow Family

| 기준 | DataSync | Storage Gateway | Snow Family |
|------|----------|----------------|-------------|
| 목적 | **일회성/반복 대량 전송** | **하이브리드 상시 연결** | **오프라인 물리 전송** |
| 방향 | 온프레미스 → AWS (또는 역방향) | 온프레미스 ↔ AWS (상시) | 온프레미스 → AWS |
| 네트워크 | 온라인 (빠른 네트워크 필요) | 온라인 (캐시로 보완) | **오프라인** (네트워크 불필요) |
| 사용 사례 | 초기 마이그레이션, 정기 동기화 | 백업, 파일 공유 | 대역폭 부족, 수십 TB+ |

> ⚠️ **빈출 함정**: "대용량 데이터 전송" → 네트워크 상태에 따라 DataSync vs Snow Family 갈림

### 비용 도구 키워드 매핑

| 시험 키워드 | 정답 |
|------------|------|
| "비용 추세 분석", "비용 시각화" | **Cost Explorer** |
| "예산 초과 시 알림" | **Budgets** |
| "가장 상세한 비용 데이터" | **Cost and Usage Report (CUR)** |
| "인스턴스 사양 최적화 추천" | **Compute Optimizer** |
| "비용/보안/성능 자동 검사" | **Trusted Advisor** |
| "유연한 컴퓨팅 할인" (EC2+Lambda+Fargate) | **Savings Plans** |
| "특정 인스턴스 타입 할인" | **Reserved Instances** |

### Savings Plans vs Reserved Instances

| 기준 | Savings Plans | Reserved Instances |
|------|--------------|-------------------|
| 적용 범위 | EC2 + Lambda + Fargate (**유연**) | EC2만 (특정 인스턴스 패밀리/리전) |
| 유연성 | 인스턴스 타입/리전/OS 변경 가능 | 변경 제한적 (Convertible RI는 일부 가능) |
| 할인율 | RI와 유사 (최대 ~72%) | 최대 ~72% |
| 추천 상황 | 워크로드 변동 가능, 다양한 서비스 사용 | 특정 인스턴스 확정, 최대 할인 필요 |

> 💡 **시험 팁**: 최근 문제는 Savings Plans를 정답으로 유도하는 경향. "유연한 할인" → Savings Plans

---

## 3. 헷갈리는 포인트 / 함정

### 함정 1: DMS vs SCT — 언제 SCT가 필요한가

- ❌ "DB 마이그레이션이면 무조건 DMS만" → **이종 DB면 SCT 필요**
- ✅ **동종 DB** (MySQL → MySQL): DMS만으로 충분
- ✅ **이종 DB** (Oracle → Aurora): **SCT** (스키마 변환) + **DMS** (데이터 이동)
- 💡 "스키마 변환" 키워드 → SCT 추가

### 함정 2: DataSync vs Transfer Family

- ❌ "파일 전송"이면 무조건 DataSync → **프로토콜에 따라 다름**
- ✅ **DataSync**: 에이전트 기반, NFS/SMB → S3/EFS/FSx, 대량 전송에 최적화
- ✅ **Transfer Family**: SFTP/FTPS/FTP → S3/EFS, 기존 FTP 클라이언트 유지
- 💡 "기존 SFTP/FTP 워크플로우 유지" → Transfer Family

### 함정 3: Snow Family 선택 기준

- ❌ "오프라인 전송이면 무조건 Snowball" → **데이터 크기에 따라 다름**
- ✅ ~8 TB: **Snowcone**
- ✅ 수십~수백 TB: **Snowball Edge**
- ✅ 수십 PB 이상: **Snowmobile**
- 💡 "엣지 컴퓨팅"이 추가되면 → Snowball Edge **Compute Optimized**

### 함정 4: Cost Explorer vs Budgets vs CUR

- ❌ 비용 도구를 구분 못함 → **목적이 다름**
- ✅ **Cost Explorer**: 과거 비용 분석 + 미래 예측 시각화
- ✅ **Budgets**: 예산 설정 + 초과 시 알림/액션
- ✅ **CUR**: 가장 상세한 원시 비용 데이터 (S3로 내보내기, BI 분석용)
- 💡 "알림" → Budgets, "분석" → Cost Explorer, "상세 데이터" → CUR

### 함정 5: Savings Plans vs Reserved Instances

- ❌ "비용 절감 = Reserved Instances" → **Savings Plans가 더 유연**
- ✅ RI: 특정 인스턴스 타입에 고정, 변경 어려움
- ✅ Savings Plans: 인스턴스 타입/리전/OS 자유롭게 변경 가능
- 💡 "유연성" 키워드 → Savings Plans, "특정 인스턴스 확정" → RI

### 함정 6: Application Migration Service vs DMS

- ❌ "마이그레이션이면 DMS" → **서버 vs DB를 구분해야 함**
- ✅ **DMS**: 데이터베이스 마이그레이션
- ✅ **MGN (Application Migration Service)**: 서버 전체 마이그레이션 (리호스팅)
- 💡 "서버 리프트 앤 시프트" → MGN, "DB 마이그레이션" → DMS

---

## 4. 단골 시나리오

### 시나리오 1: Oracle → Aurora 마이그레이션

```
문제 패턴: "온프레미스 Oracle DB를 Aurora PostgreSQL로 마이그레이션"
정답 패턴: SCT (스키마 변환) + DMS (데이터 이동)
키 포인트: 이종 DB → SCT 필수
```

### 시나리오 2: 대용량 데이터센터 마이그레이션

```
문제 패턴: "100 TB 데이터, 인터넷 대역폭 100 Mbps, 빠르게 전송"
정답 패턴: Snow Family (Snowball Edge)
키 포인트: 100 TB / 100 Mbps ≈ 92일 → 네트워크로는 비현실적 → Snow Family
```

### 시나리오 3: 비용 초과 자동 알림

```
문제 패턴: "월 비용이 예산 80% 초과하면 자동 알림"
정답 패턴: AWS Budgets + SNS 알림
키 포인트: "예산 + 알림" = Budgets
```

### 시나리오 4: EC2 인스턴스 사양 최적화

```
문제 패턴: "EC2 인스턴스가 오버프로비저닝된 것 같다, 적정 사양 추천"
정답 패턴: Compute Optimizer
키 포인트: "사양 추천", "오버프로비저닝" = Compute Optimizer
```

### 시나리오 5: 기존 SFTP 워크플로우 유지하면서 S3로

```
문제 패턴: "파트너가 SFTP로 파일을 보내는데, S3에 저장하고 싶다"
정답 패턴: Transfer Family (SFTP → S3)
키 포인트: "기존 SFTP 유지" = Transfer Family
```

### 시나리오 6: 이미지 업로드 후 자동 콘텐츠 모더레이션

```
문제 패턴: "사용자 업로드 이미지에서 부적절한 콘텐츠 자동 감지"
정답 패턴: S3 Event → Lambda → Rekognition
키 포인트: "이미지 분석", "콘텐츠 모더레이션" = Rekognition
```

---

## 5. 검증 필요 항목 ⚠️

> Claude의 학습 데이터가 outdated일 수 있는 항목. AWS 공식 문서에서 확인 필요.

- [ ] Snow Family 디바이스별 최신 용량/스펙 확인
  - 공식 문서: https://docs.aws.amazon.com/snowball/latest/developer-guide/device-differences.html
- [ ] Application Migration Service (MGN) 최신 기능 확인 (CloudEndure Migration 통합 이후)
- [ ] Savings Plans 최신 할인율 및 적용 서비스 범위
- [ ] Compute Optimizer 지원 리소스 타입 최신 목록 (ECS, RDS 등 추가 여부)
- [ ] DataSync 최신 지원 소스/대상 (FSx for NetApp ONTAP 등)
- [ ] Transfer Family 최신 지원 프로토콜 (AS2 추가 여부)
- [ ] SageMaker 최신 기능 (Studio, Canvas 등) — SAA 시험 범위인지 확인
- [ ] Trusted Advisor: Business/Enterprise Support 플랜에서만 전체 검사 제공인지 확인

---

## 6. 이 챕터로 풀 수 있는 dump 유형

- **유형 1**: "이미지/비디오 분석" → Rekognition
- **유형 2**: "이종 DB 마이그레이션" → SCT + DMS
- **유형 3**: "대용량 오프라인 전송" → Snow Family
- **유형 4**: "대용량 파일 온라인 전송 (NFS → S3)" → DataSync
- **유형 5**: "SFTP → S3" → Transfer Family
- **유형 6**: "비용 초과 알림" → Budgets
- **유형 7**: "인스턴스 사양 최적화" → Compute Optimizer
- **유형 8**: "유연한 컴퓨팅 할인" → Savings Plans
- **유형 9**: "서버 리호스팅" → Application Migration Service
- **유형 10**: "문서 OCR / 데이터 추출" → Textract

---

## 7. 학습 메모 (Banghyeon 작성 영역)

> dump 풀면서 추가로 깨달은 점, 자주 틀리는 부분 등을 여기에 기록하세요.

- (아직 기록 없음)
