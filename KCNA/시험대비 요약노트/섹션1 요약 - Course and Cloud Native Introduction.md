# 섹션1 요약 — Course and Cloud Native Introduction

> 원본: kcna-transcripts-clean.md · 섹션1 / 시험대비 암기용
> 깊이: 중 (정의·기관·시험 개요 암기 중심)

## 0. 시험 포인트 한눈에 (암기표)

| 항목 | 반드시 외울 것 |
| --- | --- |
| Cloud Native 본질 | 0/1 이분법 아님 → **철학(philosophy)들의 집합**. 하나 이상에 "정렬(align)"되는 것 |
| "Cloud"의 의미 | 특정 퍼블릭 클라우드 편애 없음 → **public·private·hybrid 모두** 포함 |
| "Native"의 의미 | 특정 시스템에 맞게 **설계되거나 내장된(designed for / built into)** |
| 컨테이너 = 클라우드 네이티브? | **아니오**. 컨테이너 사용·클라우드 실행은 좋은 방향일 뿐, 그 자체로 CN 아님 |
| Linux Foundation | **2000년** 설립. OSDL + Free Standards Group **병합** |
| CNCF 첫 기증 프로젝트 | **Kubernetes** (Google 기증, 최초 제출 프로젝트) |
| K8s 1.0 | **2015년 2월** stable·production ready |
| CNCF 모회사 | **Linux Foundation** |
| KCNA 위치 | Kubernetes/Cloud Native 입문 관문 → **CKA·CKAD의 선행(precursor)** |
| Kubestronaut | K8s 자격 **5개 전부** 합격(KCNA·KCSA·CKA·CKAD·CKS) |
| 시험(신버전) | 객관식 60문항 / 90분 / **75% 합격** / 온라인 프록터링 / 무료 재응시 1회 |

## 1. 핵심 개념 정리

### 1-1. Cloud Native란
- 이분법(있다/없다)이 아니라 **여러 철학의 집합**. 어떤 것이 하나 이상의 철학에
  "정렬"될 수 있다.
- 대표 철학: **Architecture, Culture, Community & Governance, Personas,
  Open Standards**.
- "클라우드에서 돈다"·"컨테이너로 돈다"는 **CN의 충분조건이 아니다**. 올바르게
  적용돼야 함.
- CN 여부 판단 질문(핵심): ① 셋업·배포가 **자동화**되었나 ② **복원력
  (resilience)** 있게 설계됐나 ③ 워크로드에 따라 **자동 확장(autoscale)**
  되나 ④ **기본이 보안(secure by default)** 인가.
- 표현 팁: "내 앱은 CN이다/아니다"가 아니라 "내 앱은 다음 CN 접근을 쓴다"로.

### 1-2. Linux Foundation (LF)
- **2000년** 설립. **Open Source Development Labs(OSDL) + Free Standards
  Group** 의 병합.
- 목적: Linux 표준화, 성장 지원, 상업적 채택 촉진.
- 후원: Linus Torvalds(창시자), Greg Kroah-Hartman(리드 메인테이너) 지원.
  Google·Intel·Meta·Microsoft·Samsung 등 참여.
- 호스팅: **Linux Kernel, Kubernetes, CNCF**.

### 1-3. CNCF (Cloud Native Computing Foundation)
- Cloud Native 기술을 **촉진·개발하는 핵심 기관**. 벤더 중립 허브.
- **2015년** 설립, K8s 1.0(2015.2)의 성공으로 Google이 Kubernetes를
  **첫 기증 프로젝트**로 기증.
- 수백 개 프로젝트 관리, 1,000만+ 기여. **모회사 = Linux Foundation**.

### 1-4. KCNA & 시험 개요
- KCNA = Kubernetes and Cloud Native Associate. **입문 관문**,
  **CKA·CKAD의 선행 학습**에 적합.
- 5개 자격 전부 합격 = **Kubestronaut**(KCNA·KCSA·CKA·CKAD·CKS).
- KCNA 커리큘럼은 보안(→ CKS), Prometheus(→ 전용 Prometheus 자격),
  telemetry/observability를 부분 포함.
- 형식: 객관식. **온라인 프록터링**(화면·카메라 감시) 또는 시험센터.
- 신버전 기준: **60문항 / 90분 / 75% 합격 / 무료 재응시 1회**.
- 공식 커리큘럼은 **GitHub**에서 확인(변경사항 확인처).
- 할인 코드 **DIVEINTO30**(30% 할인), 다양성 그룹 지원 가능.

### 1-5. 도메인 비중 (신버전, 반드시 암기)
| 도메인 | 비중 |
| --- | --- |
| Kubernetes Fundamentals | 44% |
| Container Orchestration | 28% |
| Cloud Native Application Delivery | 16% |
| Cloud Native Architecture | 12% |

## 2. 헷갈리는 것 구분 (비교표)

### Linux Foundation vs CNCF
| 구분 | Linux Foundation | CNCF |
| --- | --- | --- |
| 설립 | 2000 | 2015 |
| 관계 | **모회사** | LF **산하** 재단 |
| 역할 | Linux 표준화·오픈소스 전반 | **Cloud Native** 기술 촉진·프로젝트 관리 |
| 대표 호스팅 | Linux Kernel, K8s, CNCF | K8s 등 수백 CN 프로젝트 |

### 자격 라인업 (KCNA의 위치)
| 자격 | 성격 |
| --- | --- |
| **KCNA** | 입문(associate)·객관식·CN 전반 |
| KCSA | 보안 associate |
| CKA / CKAD | 실기(관리자 / 개발자) |
| CKS | 보안 전문(실기) |
| Kubestronaut | 위 5개 전부 합격 |

## 3. 함정·키워드

- **"컨테이너로 실행 = Cloud Native"는 틀림.** 방향은 맞지만 충분조건 아님.
- **"클라우드에서 실행 = Cloud Native"도 틀림.** public/private/hybrid 다 포함.
- Cloud Native는 **이분법이 아니라 철학의 집합** (핵심 함정 포인트).
- Kubernetes는 **CNCF 최초 기증/제출 프로젝트** (Google 기증).
- Linux Foundation은 CNCF의 **모회사** (반대로 쓰면 오답).
- CN 판단 4요소 키워드: **automated · resilient · autoscale · secure by
  default**.

## 4. 자가 점검 Q&A

1. Q: Cloud Native는 있다/없다의 이분법인가?
   A: 아니오. **여러 철학의 집합**이며 하나 이상에 정렬되는 개념.
2. Q: "Cloud Native"의 cloud는 퍼블릭 클라우드만 의미하나?
   A: 아니오. **public·private·hybrid 모두** 포함(편애 없음).
3. Q: Linux Foundation은 어떤 두 조직의 병합으로 몇 년에 설립됐나?
   A: **OSDL + Free Standards Group**, **2000년**.
4. Q: CNCF에 최초로 기증된 프로젝트와 기증사는?
   A: **Kubernetes**, **Google**.
5. Q: Kubernetes 1.0 정식 릴리스 시점은?
   A: **2015년 2월**.
6. Q: CNCF와 Linux Foundation의 관계는?
   A: Linux Foundation이 **모회사(CNCF는 산하)**.
7. Q: Kubestronaut 조건은?
   A: **KCNA·KCSA·CKA·CKAD·CKS 5개 전부 합격**.
8. Q: 앱이 Cloud Native한지 판단하는 핵심 질문 4가지는?
   A: 자동화 여부 · 복원력 설계 · 자동 확장 · secure by default.
9. Q: KCNA 신버전 합격 기준 점수는?
   A: **75%** (60문항 / 90분).

