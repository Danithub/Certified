# 📘 섹션2 정리노트 — Cloud Native Architecture

> 출처: James Spurin — Kubernetes Certified (KCNA), Section 2
> 시험 비중: Cloud Native Architecture 12% (생태계·원칙·관측성·커뮤니티) → CKA/CKAD 보유자의 실제 득점 포인트
> 아래는 강의 영상 핵심 요약. 헷갈리는 개념은 오답노트로 연계.

---

## 0. 미작성 파트 (나중에 추가 예정) 📝

> 아래 파트는 서버리스 이전에 다뤄지는 내용으로, 추후 정리해 추가할 예정.

- [ ] **Cloud Native Architecture Fundamentals** (클라우드 네이티브 아키텍처 기초)
- [ ] **Cloud Native Practices** (클라우드 네이티브 실천 방법)
- [ ] **Autoscaling** (오토스케일링)

---

## 1. Serverless (서버리스)

### 핵심 개념
- **"Serverless에도 서버는 있다"** — 다만 *남의 서버*(클라우드 제공자)라서, 우리가 관리·유지보수하지 않을 뿐.
- 제공자가 온디맨드로 리소스를 제공하고 서버·서비스 운영을 대신 담당 → 서버 유지보수·비용 감가상각 부담 제거.
- 인프라 선택(코어 수, 메모리, 네트워킹, 스토리지, OS·패치) 신경 쓸 필요 없음.
- 상호작용은 **코드** 또는 **컨테이너 이미지**로 직접 수행.

### FaaS (Function as a Service)
- 서버리스의 대표 범주. 대표 예시 = **AWS Lambda**.
- Lambda: 코드를 zip 파일 또는 컨테이너 이미지로 업로드.

### 시험 포인트 2가지 ⭐
1. **이벤트 기반 아키텍처(Event-Driven Architecture)**
   - 이벤트에 응답하여 실행 → 코드 실행 시 사용한 리소스만큼 그때그때 과금.
2. **어떤 규모에서든(At Any Scale) = 오토스케일링 내장**
   - 서버리스의 핵심 구성 요소.
   - 일반 오토스케일링은 "1개 이상 인스턴스"를 다루지만, 서버리스는 이벤트 기반이라 **Scale to Zero(0으로 축소)**가 표준.
   - 설정·구성은 신경 쓸 필요 없음(내장). 단, **비용 관점**에서 한도(limit)·예산 임계값(budget threshold) 설정 필요.

### 추가 개념
- **Provisioned Concurrency(프로비저닝된 동시성)**: 서버리스가 동시성을 자동 처리 → 자체 관리 부담 감소.
- **클라우드 네이티브 서버리스 (K8s 위 FaaS 오픈소스)**:
  - **Knative**, **OpenFaaS** → 쿠버네티스 위에서 FaaS 제공.
  - 예: K8s 위 서버리스 웹앱 → 미사용 시 자동 Scale Down, 접근 시 Knative가 자동 Scale Up.
- **도전 과제**: 많은 클라우드 제공자가 독점(proprietary) 솔루션 사용 → 표준 API 부재.
  - **CloudEvents 명세**: 이벤트 데이터를 공통 형식으로 기술하는 일관된 방법 → 서비스·플랫폼·시스템 간 상호운용성 제공. **CNCF 호스팅**. 주요 언어 SDK·공통 프로토콜 지원.

---

## 2. Community and Governance (커뮤니티와 거버넌스)

### CNCF 개요
- **CNCF(Cloud Native Computing Foundation)**: 클라우드 컴퓨팅을 위한 **벤더 중립적(vendor-neutral) 허브**.
- 역할: 클라우드 네이티브 프로젝트의 **호스팅·지원·감독·방향 설정**.
- 대표 프로젝트: Kubernetes, Envoy, Prometheus.
- **미션: "클라우드 네이티브 컴퓨팅을 어디에나 존재하게(Ubiquitous) 만드는 것"**

### 프로젝트 성숙도 3단계 ⭐ (순서 암기)
```
Sandbox → Incubating → Graduated
(샌드박스)  (인큐베이팅)   (졸업)
```
- 진입 장벽: Sandbox는 낮음 → Incubating으로 갈 때 **상당한 장벽(significant barrier)** 존재.

### Crossing the Chasm (캐즘 넘기) ⭐
- **캐즘 = 초기 기술 채택 → 주류 프로젝트 전환의 어려운 간극.**
- 강의 예시: Incubating → Graduated 전환에 해당.
- 사용자 채택 곡선 (성숙도 순):

| 단계 | 사용자 | 프로젝트 상태 |
| --- | --- | --- |
| Innovators (혁신가) | 새롭고 흥미로워서 관심 있는 기술 애호가 | Alpha / PoC |
| Early Adopters (얼리 어답터) | 요구·기회를 보는 개인·기업, 초기 문제 수용·피드백 제공 | Beta |
| **← 여기가 캐즘 (가장 큰 간극) →** | 성능·보안·기능 기대 충족이 관건 | |
| Early Majority (전기 다수, 실용주의자) | 요구 충족 솔루션을 찾음, "충분히 좋으면 OK" | RC → Release |
| Late Majority (후기 다수) | 위험 민감, 시장 채택 증가를 기다림 | |
| Laggards (지각 수용자) | 회의적, 필요·경쟁 위험·뒤처짐 우려로 마지못해 채택 | |

- Kubernetes: 졸업 시 첫 메이저 릴리스 버전으로 전환.

### 졸업 기준 (TOC에 입증) ⭐
- **TOC(Technical Oversight Committee, 기술 감독 위원회)** 에게 성숙도 입증.
- 입증 요소:
  - 채택(Adoption) 입증
  - 건강한 변경 속도(healthy rate of changes)
  - 서로 다른 조직 출신 **외부 커미터(external committers)** 보유
  - **CNCF 행동 강령(Code of Conduct)** 구현·실천
  - **CII Best Practices Badge**(Core Infrastructure Initiative) 획득·유지

### CNCF Landscape
- 클라우드 네이티브 생태계 확인용 리소스 (landscape.cncf.io).
- 프로젝트를 Graduated / Incubating / Sandbox로 필터 가능.
- 회색 박스 = 벤더 기반·CNCF 비회원일 수 있으나 랜드스케이프에 포함되는 프로젝트.

### 갈등 해결 (Conflict Resolution) 2대 축
1. **Discussion(토론)**: 핵심. Open CNCF calls에서 각 관점 논쟁 → 합의·다음 단계 도출.
2. **Elections & Voting(선거·투표)**: 토론으로 해결 안 되거나 넓은 대상 결정 시.
   - TOC 구성원도 투표로 선출.
   - **Binding Vote(구속력 있는 투표)**: 가장 중요, 실제 집계.
   - **Non-Binding Vote(구속력 없는 투표)**: 집계엔 미포함, 동의/반대 견해 표시.

### 알아둘 약어 ⭐
| 약어 | 의미 | 설명 |
| --- | --- | --- |
| **TOC** | Technical Oversight Committee | 기술 감독 위원회, 성숙도 판정 |
| **SIG** | Special Interest Group | 프로젝트 특정 영역 집중 오픈 그룹 (예: Kubernetes SIGs) |
| **TAG** | Technical Advisory Group | CNCF의 기술 자문 그룹 (구 CNCF SIG에서 명칭 변경, 투표로 결정) |

- **SIG vs TAG 혼동 주의**: 예전엔 CNCF도 "SIG" 사용 → 쿠버네티스 등 타 프로젝트 SIG와 이름 겹침 → CNCF 활동은 **TAG**로 변경.
- **TAG 역할**: 특정 도메인(스토리지, 보안, 앱 전달, 네트워크, 관측성, 런타임, 기여자 전략)에 기술 지침 제공.
  - Sandbox 제안 온보딩 안내·지원
  - Sandbox → Incubating 이상 전환 프로젝트 검토·지원
  - 해당 TAG 영역 사용자·참여자 요구 조율

---

## 3. Cloud Native Personas (클라우드 네이티브 직군/페르소나)

### 핵심 관점 ⭐
- 이 직군들은 **상호 배타적이지 않음(not mutually exclusive)** → 서로 겹치고 보완함.
- 조직은 서로 다른 전문 분야 인재를 함께 채용하는 것이 바람직.
- **DevOps 무한 루프(Infinity Loop)**: `Plan → Code → Build → Test → Release → Deploy → Operate → Monitor` (좌측=개발 / 우측=운영). 직군별로 루프의 강조 지점이 다름.

### 3-1. 직군별 핵심 요약

| 직군 | 한 줄 정의 | 핵심 키워드 / 강조점 |
| --- | --- | --- |
| **DevOps Engineer** | 개발+운영을 잇는 다리, 개발 쪽으로 약간 치우친 역량 | 인프라 프로비저닝, 시스템 관리, 자동화·스크립팅, **옹호(advocacy)**·문화 |
| **SRE (Site Reliability Engineer)** | 대규모 **신뢰성**에 집중 (구글, 2003 창시) | 가동시간·가용성·확장성·복원력, **SLA/SLO/SLI**, 장애관리(Incident Mgmt) |
| **CloudOps Engineer** | 클라우드 워크로드 운영·최적화 | 무한 루프 **우측**(Deploy·Operate·Monitor), Terraform/IaC, 클라우드 리소스 |
| **Security Engineer** | IT 보안 전문, 총체적 관점 | 공격 벡터, 네트워크 보안, 위협 탐지, 보안 문화·모범 사례(덜 hands-on) |
| **DevSecOps Engineer** | 전통 IT 보안과 DevOps를 잇는 다리 | CI/CD에 보안 자동화(코드·컨테이너 취약점 스캔), 생명주기 전반 보안 |
| **Full Stack Developer** | 프론트+백엔드 모두 담당 | FE: React/Angular/HTML/CSS · BE: Go/Rust/C·C++/Python, DB 연동 |
| **Cloud Architect** | 클라우드 앱·인프라·솔루션 설계 | 플랫폼/멀티클라우드 선택, 벤더 종속 회피, 비용·성능·관측성 평가, 협업 |
| **Data Engineer** | 데이터 조작·접근 시스템 설계·구축 | 대규모 데이터 확장, 분산 처리, 데이터 변환 알고리즘, 규정 준수 검증 |

### 3-2. SLA / SLO / SLI 구분 ⭐ (시험 단골)
| 약어 | 의미 | 예시 |
| --- | --- | --- |
| **SLA** | Service Level **Agreement** (협약, 계약) | "가동시간 99.99% 달성해야 한다" |
| **SLO** | Service Level **Objective** (목표) | "응답은 항상 200ms 미만" |
| **SLI** | Service Level **Indicator** (지표, 실측치) | "현재 가동 97%, 응답 300ms → SLA·SLO 미달" |

> 암기 팁: **A**greement(계약) ⊃ **O**bjective(목표) → **I**ndicator(측정값으로 달성 여부 확인)

### 3-3. DevOps vs CloudOps vs DevSecOps 비교
- **DevOps**: 관리형 서버·자산 제어, OS와 밀접. 배포에 Ansible/Chef/Puppet(구성관리) 사용.
- **CloudOps**: 클라우드 리소스 중심, 배포에 Terraform 등 IaC·클라우드 컴퓨팅 활용.
- **DevSecOps** = DevOps + Security Engineer 지식 → 보안을 파이프라인·생명주기에 통합.

### 3-4. Further Study — 추가 직군

| 직군 | 한 줄 정의 | 핵심 스킬 |
| --- | --- | --- |
| **FinOps Engineer** | 클라우드 재무 관리, 비용에 재무적 책임성 부여 | 클라우드 비용 관리·최적화, 재무 보고·분석, 예산·예측, CSP 지식, 소통·협업 |
| **Machine Learning Engineer** | 기계가 스스로 학습·행동하는 프로그램 제작 | Python/R/Java, 데이터 모델링·평가, ML 알고리즘, 고급 수학, 분산 컴퓨팅 |
| **Data Scientist** | 복잡한 데이터 분석·해석으로 의사결정 지원 | Python/R, 통계·ML, 데이터 랭글링·시각화, 빅데이터(Hadoop·Spark), SQL |

- **FinOps 핵심**: 재무·비즈니스·IT 부서 교차 → 속도·비용·품질 간 **비즈니스 트레이드오프** 가능하게. 비용 예측·관리 용이.
- **ML Engineer vs Data Scientist 구분**: ML Engineer는 *학습하는 시스템(모델·프로그램) 제작*에 초점 / Data Scientist는 *데이터 분석·인사이트로 비즈니스 의사결정 지원*에 초점.

---

## 4. Open Standards (오픈 표준)

### 핵심 개념
- **오픈 표준** = 누구나 자유롭게 접근·채택·구현하고, 발전 과정에 참여 가능한 표준.
- 오픈소스 컴퓨팅과 그 채택·성장의 핵심 요소.
- 아키텍트 관점: 오픈 표준 기반 제품 선택 = **벤더 종속(vendor lock-in) 회피** → 현명한 선택.
- **Docker**가 컨테이너 기술을 대중화했을 뿐 아니라, 오픈 표준·오픈 기여 창출에도 중추 역할.

### 4-1. OCI (Open Container Initiative) ⭐
- **2015년** Docker Inc + CoreOS 등 업계 협력 → **Linux Foundation** 산하로 출범.
- 목적: **컨테이너 이미지·런타임·배포(distribution)** 에 대한 오픈 표준 제정.

| 명세 | 정의 | 관련 도구/구현 |
| --- | --- | --- |
| **Image Specification** (이미지 명세) | 파일 시스템 번들을 어떻게 **이미지로 패키징**할지 정의. Docker 이미지 포맷이 채택됨 | 빌드 도구: **Buildkit, Podman, Buildah** → OCI 호환 이미지 생성 |
| **Runtime Specification** (런타임 명세) | 파일 시스템 번들을 어떻게 **다운로드·압축 해제·실행**할지 정의 | 런타임: **runC**(최초 기부, 참조 구현), Kata Containers, gVisor, Firecracker |
| **Distribution Specification** (배포 명세) | 콘텐츠 **배포 표준화**용 오픈 표준·API 프로토콜 정의 | **Docker Registry HTTP API v2** 기반 (Docker Hub 사용) |

- **runC** ⭐: Docker가 OCI에 기부한 **최초의 오픈 표준 런타임** = **참조 구현(reference implementation)**.
- 이점: OCI 포맷으로 이미지를 자유롭게 만들고(Buildkit/Podman/Buildah 등), 런타임 명세 구현 기술 어디서든 실행 가능 → 클라우드 네이티브 원칙과 부합.

### 4-2. Kubernetes 관련 오픈 표준 (인터페이스 4종) ⭐

| 약어 | 이름 | 역할 | 대표 구현/예시 |
| --- | --- | --- | --- |
| **CNI** | Container **Network** Interface | 컨테이너 **네트워크** 표준. 미설치 시 노드가 **Not Ready → Ready** 전환 안 됨 | 다양한 CNI 플러그인 (Docker Desktop·Minikube 자동 설치 / kubeadm 수동 설치) |
| **CSI** | Container **Storage** Interface | **스토리지** 솔루션 인터페이싱 표준 | **Rook**(CNCF 졸업), Portworx(상용) |
| **CRI** | Container **Runtime** Interface | **kubelet**이 다양한 컨테이너 런타임과 상호작용하는 플러그인 인터페이스 | containerd, CRI-O, Kata Containers, Firecracker |
| **SMI** | **Service Mesh** Interface | **서비스 메시** 명세 | (서비스 메시 기술 다수) |

- **CNI 세부** ⭐:
  - 노드를 Not Ready → Ready로 전환시키려면 CNI 호환 구현 설치 필수.
  - 배포판별 처리: Docker Desktop·Minikube = **자동 설치** / **kubeadm = 직접 선택·설치**.
  - 명세가 단순 → bash 스크립트로도 자체 CNI 제작 가능.
  - CNI 명령 동사 4개: **ADD, DEL, CHECK, VERSION**.
- **CRI 세부**: kubelet ↔ 런타임 엔진 연결 플러그인 → 런타임 교체 용이. kubelet 전용 아님(타 프로젝트도 사용 가능).
- 공통 이점: 오픈 표준 준수 → **벤더 종속 회피**, 손쉬운 교체.

> 암기 팁: **C_I** 4형제 → **N**etwork / **S**torage / **R**untime / **S**erviceMesh
> (CNI·CSI·CRI·SMI)

---

## 🎯 시험 대비 핵심 암기 체크
- [ ] 서버리스 = 이벤트 기반 + Scale to Zero + 사용량 과금
- [ ] FaaS 예시: AWS Lambda / K8s 위: Knative, OpenFaaS
- [ ] CloudEvents = CNCF, 이벤트 데이터 표준(상호운용성)
- [ ] 성숙도 순서: Sandbox → Incubating → Graduated
- [ ] 캐즘 = Early Adopters ↔ Early Majority 사이 최대 간극
- [ ] 졸업 판정 = TOC / 기준(채택·변경속도·외부커미터·CoC·CII Badge)
- [ ] TOC / SIG / TAG 구분 (CNCF는 TAG)
- [ ] Binding vs Non-Binding Vote 차이
- [ ] 직군은 상호 배타적 아님 → 서로 보완
- [ ] SLA(계약) / SLO(목표) / SLI(지표) 구분 — SRE 담당
- [ ] SRE = 신뢰성·가용성 중심 (구글, 2003)
- [ ] CloudOps = 루프 우측(Deploy·Operate·Monitor), Terraform/IaC
- [ ] DevSecOps = 전통 보안 ↔ DevOps 다리 (CI/CD 보안 자동화)
- [ ] FinOps = 클라우드 비용에 재무 책임성 부여
- [ ] ML Engineer(학습 시스템 제작) vs Data Scientist(데이터 분석·인사이트) 구분
- [ ] OCI = 2015년 출범, Linux Foundation 산하 (이미지·런타임·배포 표준)
- [ ] OCI 3대 명세: Image / Runtime / Distribution
- [ ] runC = OCI 최초 기부 런타임 = 참조 구현
- [ ] OCI 이미지 빌드 도구: Buildkit, Podman, Buildah
- [ ] CNI(네트워크) — 미설치 시 노드 Not Ready, kubeadm은 수동 설치
- [ ] CNI 명령 동사: ADD, DEL, CHECK, VERSION
- [ ] CSI(스토리지) — Rook(CNCF 졸업), Portworx(상용)
- [ ] CRI(런타임) — kubelet ↔ containerd/CRI-O/Kata/Firecracker
- [ ] SMI(서비스 메시) 인터페이스
- [ ] 오픈 표준의 핵심 이점 = 벤더 종속(vendor lock-in) 회피
