# 섹션7 요약 — Cloud Native Application Delivery

> 원본: kcna-transcripts-clean.md · 섹션7 / 시험대비 암기용
> 깊이: 중 · 도메인 Cloud Native Application Delivery(16%)

## 0. 시험 포인트 한눈에 (암기표)

| 주제 | 반드시 외울 것 |
| --- | --- |
| App Delivery | 빠르고·확장성·복원력 있는 배포. 컨테이너화·마이크로서비스·동적 오케스트레이션 활용 |
| CI | 잦은 커밋 + **자동 테스트** |
| CD (KCNA 기준) | **Continuous Delivery**(릴리스는 사람이 개시). Deployment(전자동)와 구분 |
| GitOps 핵심 | **Git = 단일 진실 원천(single source of truth)** · 선언적 인프라·앱 |
| Argo CD | 선언적 GitOps 도구 · **self heal · prune · auto sync** |
| 조정(reconcile) | 실제 상태 ≠ Git → **Git 상태로 자동 복구**(예: namespace 삭제해도 재생성) |
| 다른 GitOps 도구 | **Flux**(Argo CD와 함께 대표 GitOps 도구) |

## 1. 핵심 개념 정리
### 1-1. Cloud Native Application Delivery
- 클라우드 환경에서 **빠르고·확장성·복원력** 있는 앱 배포를 위한 접근.
- **컨테이너화 + 마이크로서비스 + 동적 오케스트레이션**을 활용해 신속 배포·
  효율 관리.

### 1-2. CI/CD (App Delivery 관점)
- **CI(Continuous Integration)**: 잦은 코드 커밋 + **자동 테스트**(build →
  unit test → integration test). 릴리스 전 자동 검증.
- **CD 두 의미**:
  - **Continuous Delivery**: 릴리스 준비까지 자동, **릴리스는 사람이 개시**.
  - **Continuous Deployment**: prod까지 **전자동**(+ 프로덕션 테스트·자동 롤백).
- **KCNA에서 CI/CD의 CD는 Delivery로 간주**(섹션2와 동일 포인트).

### 1-3. GitOps
- **Git을 단일 진실 원천(single source of truth)** 으로 삼아 **선언적 인프라·
  애플리케이션**을 관리하는 방식.
- 쉽게 말해 **"Git으로 인프라와 앱을 관리"**. 원하는 상태를 Git에 선언 →
  도구가 클러스터를 그 상태로 유지.
- 대표 도구: **Argo CD**, **Flux**(둘 다 CNCF 계열 GitOps 도구).

### 1-4. Argo CD
- Cloud Native **애플리케이션 딜리버리 도구**(선언적 GitOps).
- 설치: `kubectl create namespace argocd` → 설치 YAML `apply`.
- 앱 설정 옵션: **sync policy = automatic**, **prune**(Git에 없는 리소스 제거),
  **self heal**(수동 변경 시 Git 상태로 복구), auto-create namespace,
  directory recurse.
- **핵심 동작 = reconciliation(조정)**: 실제 환경이 Git과 다르면 감지 후
  **Git 상태로 자동 복구**. 예: namespace를 통째로 삭제해도 Argo가 **재생성**.
- 초기 접속: username **admin**, 비밀번호는 secret에서 조회(**base64 -d**).

## 2. 헷갈리는 것 구분 (비교표)

### Continuous Delivery vs Deployment
| 구분 | Delivery | Deployment |
| --- | --- | --- |
| 릴리스 개시 | **사람** | **자동** |
| prod 반영 | 수동 승인 후 | 전자동 |
| KCNA 기본 해석 | **이것(Delivery)** | - |

### CI vs CD
| 구분 | 초점 |
| --- | --- |
| CI(Integration) | 잦은 커밋 + 자동 테스트 |
| CD(Delivery/Deployment) | 릴리스/배포 자동화 |

### GitOps 도구
| 도구 | 비고 |
| --- | --- |
| Argo CD | 선언적·self heal·prune·자동 sync |
| Flux | 또 다른 대표 GitOps 도구 |

## 3. 함정·키워드

- CI/CD의 **CD = Continuous Delivery**(KCNA 기준). Deployment(전자동)와 구분.
- **GitOps = Git이 single source of truth**. 선언적 상태를 Git에 두고 자동 조정.
- **Argo CD self heal + prune**: 수동 변경/이탈을 Git 상태로 되돌림
  (namespace 삭제해도 재생성).
- GitOps 대표 도구 = **Argo CD, Flux**.
- App Delivery는 컨테이너화·마이크로서비스·**동적 오케스트레이션** 기반.

## 4. 자가 점검 Q&A

1. Q: GitOps의 핵심 원칙 한 문장은?
   A: **Git을 단일 진실 원천**으로 삼아 선언적 인프라·앱을 관리.
2. Q: CI/CD에서 CD의 KCNA 기준 의미는?
   A: **Continuous Delivery**(릴리스는 사람이 개시).
3. Q: Argo CD에서 수동 변경을 Git 상태로 되돌리는 기능은?
   A: **self heal**(+ prune으로 Git에 없는 리소스 제거).
4. Q: Argo CD로 관리하는 namespace를 삭제하면?
   A: Git과 불일치를 감지해 **자동 재생성**(reconcile).
5. Q: Argo CD 외 대표 GitOps 도구는?
   A: **Flux**.
6. Q: CI가 강조하는 것은?
   A: 잦은 커밋 + **자동 테스트**.
