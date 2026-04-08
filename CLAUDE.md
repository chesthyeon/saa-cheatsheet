# CLAUDE.md

> 이 파일은 Claude Code가 이 repo에서 작업할 때 자동으로 읽는 컨텍스트 파일입니다.
> 모든 작업 전에 이 문서와 `00_규약.md`를 먼저 확인하세요.

## 프로젝트 개요

**목적**: AWS SAA-C03 (Solutions Architect Associate) 자격증 합격을 위한 키워드 기반 치트시트 작성 및 학습 관리

**1차 목표**: 2026년 7월 채용 시즌 전 SAA 합격
**2차 목표**: Cloud/DevOps/SRE 직무 전환을 위한 AWS 서비스 이해도 확보

**작성자**: Banghyeon (한국경제신문 CTS팀, 클라우드 직무 전환 준비 중)

## 사용자 프로파일

- **현재 직무**: 한국경제신문 CTS팀 (시스템 엔지니어, 1년 계약직)
- **목표 직무**: Cloud/DevOps/SRE
- **기존 지식**: AWS 3-tier 아키텍처 (Cloud4C 아카데미), 정보처리기사, SQLD, 네트워크관리사 2급
- **학습 스타일**: **객관적 분석 우선, 위로는 필요시에만**. 실행 일관성이 약점이므로 모호한 격려보다 구체적 피드백 선호.
- **언어**: 한국어 본문 + AWS 서비스명/기술 용어는 영어 원문 유지

## Repo 구조

```
saa-cheatsheet/
├── CLAUDE.md              ← 이 파일 (Claude Code 컨텍스트)
├── 00_규약.md              ← 작성 규약 (모든 챕터의 기준, 반드시 먼저 읽을 것)
├── 01_v2_목차.md           ← 전체 목차
├── 10_principles/         ← Chapter 0: Well-Architected Framework
├── 11_keyword_mapping/    ← Chapter 1: 도메인별 핵심 서비스 키워드
├── 12_purpose_pattern/    ← Chapter 2: 목적별 정답 패턴
├── 13_comparison/         ← Chapter 3: 헷갈리는 쌍 비교
├── 14_scenarios/          ← Chapter 4: 단골 아키텍처 시나리오
├── 15_traps/              ← Chapter 5: 함정 유형
└── 16_wrong_notes/        ← Chapter 6: 오답 노트 가이드
```

## 작업 시 핵심 행동 규칙

### 1. 챕터 작성 시
- **반드시** `00_규약.md`의 표준 7개 섹션 구조를 따를 것
- **반드시** "검증 필요 항목 ⚠️" 섹션을 포함할 것 (가격/한도/신규 기능은 outdated 가능)
- 분량: 챕터당 300~600줄
- 자작 모의 문제 만들지 말 것 (dump 풀이가 더 정확)
- 표/키워드/함정 3종 세트를 기본 구조로

### 2. 챕터 작성 후 (필수 워크플로우)

챕터 작성 시 아래 4단계를 **매번 순서대로** 실행할 것:

1. **`.md` 파일 작성** → 해당 챕터 폴더에 저장
2. **Git commit + push** → 커밋 메시지는 규약 7번 항목 따를 것 (`feat(ch1-1): ...`)
3. **Notion 본문 임베디드** → 학습 대시보드 DB의 해당 챕터 페이지 본문에 .md 내용 전체 삽입 (모바일 학습 대응)
4. **대시보드 상태 업데이트** → 상태를 "작성완료"로 변경, 작성일 기록, Git 경로 입력

### 3. 답변 톤
- **객관적 분석 우선**, 불필요한 격려나 칭찬 금지
- 모호한 표현 회피 ("좋은 질문이에요" 같은 preamble 금지)
- 잘못된 접근에는 직설적으로 지적 후 대안 제시
- 시험과 무관한 깊이는 의도적으로 생략 (운영 경험 디테일보다 키워드 매핑이 우선)

### 4. 정보 신뢰도
- AWS 가격, 한도, 신규 기능은 학습 데이터가 outdated일 수 있음
- 확신이 없는 항목은 반드시 "검증 필요 항목"에 명시
- 사용자가 직접 AWS 공식 문서 확인하도록 링크 제공

### 5. dump 검증 워크플로우
- 챕터 작성 → Banghyeon이 dump 5~10문제 풀이 → 오답은 Notion 오답 노트 DB에 기록
- 사용자가 "Chapter X-Y에서 [약점] 보강해줘"라고 요청하면, 해당 .md의 "학습 메모" 섹션 또는 본문을 보강

## 연결된 Notion DB

> 사용자가 "노션에 기록해줘" 요청 시 아래 DB 사용

- **AWS 자격증 페이지**: https://www.notion.so/33c5a1a2c9e8817c83e1eb636947000f
  - Page ID: `33c5a1a2-c9e8-817c-83e1-eb636947000f`
- **학습 대시보드 DB**: https://www.notion.so/1c00711686ca42c6b3f6cf5e82acb085
  - Data Source ID: `aa8d501c-1b3e-405e-8a74-88667497388b`
  - 컬럼: 챕터(title), 상태(select: 미작성/작성중/작성완료/검증완료/재학습필요), 분류(select), 작성일(date), 검증일(date), 관련 dump 풀이수(number), 오답수(number), 재학습 필요(checkbox), Git 경로(text), 메모(text)
- **오답 노트 DB**: https://www.notion.so/c5202e5195224c149c54e9922469fa36
  - Data Source ID: `d4853bb4-6760-43a4-9e39-4d36b47126fa`
  - 컬럼: 문제 요약(title), 출처(select), 문제 번호(text), 내가 고른 답(text), 정답(text), 틀린 이유 분류(select), 관련 챕터(multi_select), 핵심 키워드(text), 복습 필요(checkbox), 재시도 결과(select), 기록일(created_time), 해설/메모(text)

## 챕터 작성 권장 순서 (체득 효율 순)

1. **Chapter 1** (키워드 매핑) — 전체 그림
2. **Chapter 3** (헷갈리는 쌍) — 1을 보강
3. **Chapter 4** (시나리오 템플릿) — 1+3 응용
4. **Chapter 5** (함정) — 4까지 본 뒤 정리
5. **Chapter 2** (목적별 패턴) — 1~4의 재구성
6. **Chapter 0** (Well-Architected) — 시험 직전 정독용

## 자주 쓰는 명령 패턴

### 챕터 작성 요청
```
"Chapter 1-1 컴퓨팅 및 컨테이너 치트시트 작성해줘.
00_규약.md 표준 7개 섹션 모두 포함, 검증 필요 항목 명시"
```

### 챕터 보강 요청
```
"Chapter 1-2 스토리지에서 S3 스토리지 클래스 부분 보강해줘.
dump #47에서 Glacier Instant Retrieval과 Standard-IA를 헷갈렸음"
```

### Notion 동기화 요청
```
"방금 작성한 Chapter 1-1을 학습 대시보드 DB에서 '작성완료'로 업데이트해줘"
```

### 오답 기록 요청
```
"오답 노트 DB에 기록해줘:
- 문제: S3 Cross-Region Replication 비용 최적화
- 내가 고른 답: S3 Standard
- 정답: S3 Standard-IA
- 분류: 비용 오판
- 관련 챕터: 1-2 스토리지"
```

## 절대 금지 사항

- ❌ 챕터를 한 번에 다 작성하려 하지 말 것
- ❌ 자작 모의 문제 만들지 말 것
- ❌ "검증 필요 항목" 섹션을 비워두지 말 것
- ❌ Notion에 직접 본문 작성하지 말 것 (.md가 원본)
- ❌ 과도한 격려, "잘하고 계세요" 류 멘트 금지
- ❌ 한국어 본문에 AWS 서비스명을 한국어로 음역하지 말 것 ("람다" X, "Lambda" O)

## 참고: 사용자 학습 환경

- 시간 제약: 별도 명시 없음 (당직 근무 패턴 있으나 챕터 단위로 자율 진행)
- 장비: Galaxy S26, 데스크탑
- 협업 도구: claude.ai (대화), Claude Code (이 repo 작업), Notion (트래킹)
