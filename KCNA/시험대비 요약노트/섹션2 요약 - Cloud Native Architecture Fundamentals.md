# 섹션2 요약 — Cloud Native Architecture Fundamentals

> 원본: kcna-transcripts-clean.md · 섹션2 / 시험대비 암기용
> 깊이: 상(집중) · 도메인 Cloud Native Architecture(12%) 핵심 + 출제 빈도 높음

## 0. 시험 포인트 한눈에 (암기표)

| 주제 | 반드시 외울 것 |
| --- | --- |
| CNCF 정의 문장 | loosely coupled · **resilient · manageable · observable** + robust automation → high impact changes **frequently and predictably with minimal toil** |
| Monolithic | tightly coupled → 한 곳 변경이 전체에 영향, 배포/관리 어려움 |
| Microservices | 독립 단위·자체 네트워킹·교체 용이·유연 |
| 자가치유 | Deployment→ReplicaSet→desired replica 수 유지(실패 시 자동 대체) |
| CI/CD | **CD = Continuous Delivery**(사람이 릴리스 개시). Deployment(자동 prod)와 구분. **KCNA는 Delivery로 간주** |
| Zero Trust | "never trust, always verify" · 존(zone) 무관 mutual auth |
| Least Privilege | 최소 권한만 부여 |
| 오토스케일 방식 | **Reactive / Scheduled / Predictive(AI·ML)** |
| 스케일 방향 | Vertical=scale up(기존 리소스 증설) / Horizontal=scale out(인스턴스 추가). **CN은 Horizontal 선호** |
| K8s 스케일러 | Cluster Autoscaler(노드) · HPA(replica 수) · VPA(pod requests/limits) · **KEDA(event-driven·scaled objects·scale to zero)** |
| Serverless | 이벤트 기반 · **scale to zero** · FaaS(AWS Lambda) · Knative/OpenFaaS · **CloudEvents**(CNCF, 이벤트 표준) |
| 성숙도 | **Sandbox → Incubating → Graduated** (crossing the chasm) |
| 거버넌스 조직 | **TOC**(성숙도 결정) · **SIG**(관심그룹) · **TAG**(CNCF, SIG 개명) |
| SRE 지표 | **SLA**(합의) · **SLO**(목표) · **SLI**(실측 지표) |
| 오픈표준 | OCI(Image/Runtime/Distribution) · **runc=Runtime 참조구현** · CNI · CSI · CRI · SMI |

## 1. 핵심 개념 정리
### 1-1. Architecture Fundamentals (Monolithic vs Microservices)
- CN Architecture는 레거시 설계의 문제(비효율·긴밀 결합)에서 진화. 초점:
  **가용성·비용관리·효율·신뢰성**.
- CNCF 헌장 핵심 문구(암기): loosely coupled, **resilient·manageable·
  observable** + robust automation → 엔지니어가 high impact 변경을
  **frequently and predictably with minimal toil**로 수행.
- **Monolithic(모놀리식)**: tightly coupled. 한 컴포넌트 변경이 전체에 영향.
  UI·business logic·data가 같은 OS/파일시스템에 설치돼 리소스·라이브러리 공유
  → 라이브러리 버전 충돌 등 의존성 문제. 앱 늘수록 악화.
- **Microservices(마이크로서비스)**: 작은 독립 단위로 분해. 각자 설치/설정
  캡슐화, **자체 네트워킹**(같은 포트 443도 독립 사용 가능), 팀별 독립
  유지보수. Ingress 라우팅으로 URL별 다른 서비스 매핑 가능.
- 데이터 계층 유연: 1:1 DB, replication/sharding, 또는 분산 KV store(**etcd**,
  K8s의 store of truth). 컴포넌트 교체(swap)가 쉬움.

### 1-2. Cloud Native Practices (실천 원칙)
- **Resilience / Self-healing**: 실패를 전제로 설계. K8s 예: Deployment →
  ReplicaSet → replica 5개면 pod 5개 유지. 하나 실패 시 자동 대체 = 자가치유.
- **Automation**: 속도·민첩성. 수동 지양. 예: **Ansible**(구성관리, KubeSpray로
  K8s 배포), **Terraform**(IaC, 선언적·재현 가능).
- **CI/CD**:
  - CI(Continuous Integration): 잦은 커밋 + 자동 테스트. 흐름 = commit(git) →
    pipeline 트리거 → build → unit test → integration test → review/release.
  - CD: **Continuous Delivery**(릴리스는 사람이 개시) vs Continuous
    Deployment(prod까지 자동). **시험에서 CI/CD의 CD는 Delivery로 간주**.
- **Secure by default**: **Zero Trust** = "never trust, always verify".
  신뢰 존(사내 LAN)이라도 동일하게 mutual authentication. 서비스 간 secure
  channel(예: kubeadm은 소규모 단일노드도 secure channel 구성).
- **Least privilege**: 필요한 최소 권한만 → 침해 시 피해 최소화.
- **Speed·Efficiency·Cost saving**: autoscaling(상·하), serverless(scale to
  zero). 예: Interflora(기념일 피크). 상시 확장 대신 필요 시 확장 → 비용 절감.
- **Service discovery**: 수동 설정 지양, 자동 탐지. K8s는 **환경변수 + 내장
  DNS**로 서비스 디스커버리.

### 1-3. Autoscaling
- 정의: 메트릭/요구에 따라 인프라·앱 컴포넌트를 **자동 확장**. auto=automatic/
  automated, scaling=up/down/across. 클라우드에서 미사용 리소스=비용이라 중요.
- 메트릭: CPU·memory가 대표지만 앱마다 다름(예: Netflix 트랜스코딩=GPU 의존).
- **접근 방식 3종**:
  - **Reactive**: 임계값 도달 시 반응. 지연 없이 빠르게 대응 가능한 워크로드에 적합.
  - **Scheduled**: 알려진 피크(월말 배치, 오전 9시 등)에 미리 예약 확장.
  - **Predictive**: AI/ML로 패턴 학습 → 선제 확장(reactive/scheduled보다 유리).
- **스케일 방향**:
  - **Vertical(scale up)**: 기존 인프라 스펙 증설(CPU/mem/disk). 주로 VM. VMware
    ESXi는 무중단 증설 가능하나 축소 시 재부팅 필요할 수 있음. 물리는 비현실적.
  - **Horizontal(scale out)**: 인스턴스/리소스 추가·제거. **CN은 Horizontal
    선호**. 단, 데이터 공유·동시성 등 복잡도 증가.
- **K8s 오토스케일 도구**:
  - **Cluster Autoscaler**: 워크로드 기준 **클러스터 노드 수** 조정.
  - **HPA**(Horizontal Pod Autoscaler): **replica 수** 조정.
  - **VPA**(Vertical Pod Autoscaler): pod의 **resource requests/limits** 조정.
  - **KEDA**: **event-driven**. **scaled objects** 사용, **scale to zero** 가능.
- 오토스케일 시 **테스트** 중요(확장 후 정상 동작 검증).

### 1-4. Serverless
- 서버는 존재하나 **provider가 관리**. 사용자는 코어·메모리·네트워킹·스토리지·
  OS·패치를 신경 쓸 필요 없음. 코드/컨테이너 이미지로 상호작용.
- **AWS Lambda = FaaS**(Function as a Service). zip 또는 컨테이너 이미지 업로드.
- **이벤트 기반(event-driven)**: 이벤트에 반응해 실행. **실행된 만큼(리소스)
  과금**.
- **any scale** → 오토스케일이 핵심. serverless는 보통 **scale to zero** 전제.
  설정은 자동이지만 **비용 한도/budget threshold**는 사용자가 고려해야 함.
- concurrency(동시성)는 provider가 처리(provisioned concurrency).
- CN 오픈소스: **Knative, OpenFaaS** = K8s 위 FaaS. 미사용 시 축소, 접근 시 확대.
- 문제점: provider마다 독점 방식·표준 API 부재 → **CloudEvents** 스펙 등장.
  이벤트 데이터를 공통 포맷으로 기술 → 상호운용성. **CNCF 호스팅**, 다수 언어
  SDK·공통 프로토콜 지원.

### 1-5. Community & Governance
- CNCF = **vendor-neutral hub**. 미션: **make Cloud Native Computing
  ubiquitous**.
- **성숙도 단계**: **Sandbox → Incubating → Graduated**. 진행 = "crossing
  the chasm".
- **Chasm(캐즘)**: 초기 채택 → 주류 전환의 어려운 간극(특히 Incubating→
  Graduated). 채택자 스펙트럼:
  - **Innovators**(alpha/PoC) → **Early adopters**(beta) → 〔CHASM〕 →
    **Early majority/pragmatists**(release) → **Late majority**(위험회피) →
    **Laggards**(마지막).
- **졸업(Graduated) 요건 — TOC에 증명**: adoption(채택), 건강한 변경률,
  **여러 조직의 외부 committer**, CNCF **Code of Conduct** 준수, **Core
  Infrastructure Initiative Best Practices Badge** 획득.
- Sandbox=진입장벽 낮음, Incubating→Graduated=높은 장벽.
- **CNCF Landscape**: graduated/incubating/sandbox 필터. 회색 박스=벤더/비CNCF.
- **갈등 해결**: **Discussion**(논의) + **Elections/Voting**(투표). 투표는
  **binding**(구속력 있음·중요) vs **non-binding**(찬반 의견 표시).
- **핵심 약어**:
  - **TOC** = Technical Oversight Committee → **프로젝트 성숙도 결정**.
  - **SIG** = Special Interest Group → 프로젝트 특정 영역 집중, 개방·투명.
  - **TAG** = Technical Advisory Group → CNCF가 SIG 명칭을 개명(K8s SIG와
    혼동 방지). 도메인: storage, security, app delivery, network,
    observability, runtime, contributor strategy. sandbox 온보딩·심사 지원.

### 1-6. Cloud Native Personas
- **DevOps Engineer**: dev+ops 스킬(개발 쪽 bias). 인프라 프로비저닝·시스템
  관리·스토리지/네트워킹·자동화/스크립팅. **advocacy**(CN 문화 전파)도 핵심.
  "DevOps infinity loop" 연상.
- **SRE(Site Reliability Engineer)**: **Google에서 2003년 시작**. reliability
  at scale에 집중(uptime·availability·scalability·resilience). **SLA/SLO/SLI**
  생성·구현, 인시던트 관리 + 사후 교훈.
- **CloudOps Engineer**: infinity loop의 **오른쪽(deploy·operate·monitor)**
  에 집중, 클라우드 중심(Terraform·클라우드 컴퓨트).
- **Security Engineer**: IT 보안 전반(공격 벡터·OS/네트워크 보안). 덜 hands-on,
  보안 best practice·문화 전파 중심.
- **DevSecOps Engineer**: DevOps + 보안 지식. **전통 IT 보안과 DevOps의 다리**.
  CI/CD에 보안 자동화(예: 취약점 스캔) 추가.
- **Full Stack Developer**: front end(React·Angular·HTML/CSS·iOS/Android) +
  back end(Go·Rust·C·C++·Python·데이터스토어).
- **Cloud Architect**: 클라우드 앱/인프라/솔루션 설계. 타깃 플랫폼·툴링 선택,
  **vendor lock-in 회피**, 비용·성능·관측성·자동화 평가. 대인 협업 스킬.
- **Data Engineer**: 데이터 조작·접근 시스템 설계. 대규모/분산 처리·알고리즘·
  데이터 컴플라이언스.
- **핵심 takeaway**: 역할들은 **상호 배타적이지 않고 상호 보완**적이다.

### 1-7. Open Standards
- 정의: 공개 접근·자유 채택·누구나 개발 참여 가능. 오픈소스 성장의 핵심,
  **vendor lock-in 회피**.
- **OCI(Open Container Initiative)**: 2015년 Docker Inc + CoreOS 등이
  **Linux Foundation** 산하로 출범. 컨테이너 이미지·런타임·배포 표준화.
  - **Image Specification**: 파일시스템 번들을 이미지로 패키징하는 방법.
    빌드 도구: Buildkit·Podman·Buildah → OCI 호환 이미지.
  - **Runtime Specification**: 파일시스템 번들 실행(download/unpack/run) 방법.
    Docker가 **runc** 기증 = **최초 오픈표준 = 참조 구현(reference impl)**.
    다른 런타임: Kata Containers·gVisor·Firecracker.
  - **Distribution Specification**: 콘텐츠 배포 표준·API. Docker Registry
    HTTP API v2 기반.
- **K8s 관련 오픈표준 인터페이스**(암기):
  - **CNI**(Container Network Interface): 네트워킹. 설치돼야 노드가
    NotReady→Ready. verbs: **ADD/DEL/CHECK/VERSION**.
  - **CSI**(Container Storage Interface): 스토리지. 예 Rook(CNCF graduated),
    Portworx(상용).
  - **CRI**(Container Runtime Interface): kubelet ↔ 런타임 엔진 플러그인
    (containerd·cri-o·kata·Firecracker 교체 가능).
  - **SMI**(Service Mesh Interface): 서비스 메시 표준.

## 2. 헷갈리는 것 구분 (비교표)

### CI/CD의 CD
| 약어 | 의미 | 트리거 |
| --- | --- | --- |
| Continuous **Delivery** | 릴리스 준비까지 자동, **릴리스는 사람이 개시** | 사람 |
| Continuous **Deployment** | prod까지 **전자동** 배포 | 자동 |

> KCNA에서 CI/CD의 CD는 **Delivery**로 간주합니다.

### 오토스케일링 방식
| 방식 | 트리거 | 특징 |
| --- | --- | --- |
| Reactive | 임계값 도달 시 반응 | 지연 없이 대응 가능한 워크로드 |
| Scheduled | 정해진 시각 | 알려진 피크(월말·오전9시) 사전 예약 |
| Predictive | AI/ML 예측 | 선제 확장, 가장 스마트 |

### K8s 오토스케일러
| 도구 | 무엇을 스케일 |
| --- | --- |
| Cluster Autoscaler | 클러스터 **노드 수** |
| HPA | pod **replica 수** |
| VPA | pod **requests/limits** |
| KEDA | **event-driven**, scaled objects, **scale to zero** |

### 스케일 방향
| 구분 | 별칭 | 방법 | CN 선호 |
| --- | --- | --- | --- |
| Vertical | scale up | 기존 리소스 증설 | 덜 선호 |
| Horizontal | scale out | 인스턴스 추가/제거 | **선호** |

### CNCF 성숙도
| 단계 | 채택자 | 상태 예 |
| --- | --- | --- |
| Sandbox | innovators | alpha/PoC, 진입장벽 낮음 |
| Incubating | early adopters | beta |
| Graduated | early majority~ | release, chasm 통과 후 |

### 거버넌스 조직
| 약어 | 역할 |
| --- | --- |
| TOC | Technical Oversight Committee, **성숙도 결정** |
| SIG | Special Interest Group, 프로젝트 영역별 그룹 |
| TAG | Technical Advisory Group, **CNCF가 SIG를 개명** |

### SRE 지표
| 약어 | 의미 | 예 |
| --- | --- | --- |
| SLA | 합의(Agreement) | uptime 99.99% |
| SLO | 목표(Objective) | 응답 < 200ms |
| SLI | 지표(Indicator, 실측) | 실제 uptime 97%·응답 300ms |

### OCI 3대 스펙 & 오픈표준 인터페이스
| 스펙/IF | 대상 |
| --- | --- |
| OCI Image | 이미지 패키징 |
| OCI Runtime | 실행(참조구현 **runc**) |
| OCI Distribution | 배포(Registry API v2 기반) |
| CNI | 네트워킹(노드 Ready 전환) |
| CSI | 스토리지 |
| CRI | kubelet↔런타임 |
| SMI | 서비스 메시 |

## 3. 함정·키워드

- CI/CD의 **CD = Delivery**(사람이 릴리스 개시). Deployment(전자동)와 혼동 주의.
- **KEDA** = event-driven · **scaled objects**(밑줄 강조 포인트) · **scale to
  zero**. Knative도 scale to zero(서버리스).
- **HPA=replica 수 / VPA=requests·limits / Cluster Autoscaler=노드 수** 구분.
- CN은 **Horizontal(scale out)** 선호.
- **runc** = OCI **Runtime** 스펙의 **참조 구현**(최초 기증 오픈표준).
- **CNI**가 설치돼야 노드가 **Ready**로 전환. verbs: ADD/DEL/CHECK/VERSION.
- 성숙도 순서 **Sandbox → Incubating → Graduated** (역순·중간 생략 오답 주의).
- **TOC**가 성숙도 판정. CNCF는 K8s와의 혼동 방지로 **SIG를 TAG로 개명**.
- **Zero Trust** = never trust, always verify. 사내 LAN도 동일 취급.
- **SLA(합의)/SLO(목표)/SLI(실측)** 세 개 매칭이 자주 출제.
- **CloudEvents** = 이벤트 데이터 공통 포맷 표준(CNCF), 서버리스 상호운용성.

## 4. 자가 점검 Q&A

1. Q: CI/CD에서 CD가 두 뜻을 가질 때 KCNA 기준 정답은?
   A: **Continuous Delivery**(릴리스는 사람이 개시).
2. Q: replica 수를 늘리는 오토스케일러는? pod의 requests/limits를 조정하는 것은?
   A: **HPA** / **VPA**.
3. Q: 이벤트 기반이며 scaled objects와 scale to zero를 지원하는 K8s 스케일러는?
   A: **KEDA**.
4. Q: CN에서 선호되는 스케일 방향과 그 별칭은?
   A: **Horizontal (scale out)**.
5. Q: CNCF 프로젝트 성숙도 3단계를 순서대로.
   A: **Sandbox → Incubating → Graduated**.
6. Q: 프로젝트 성숙도를 최종 판정하는 위원회는?
   A: **TOC (Technical Oversight Committee)**.
7. Q: CNCF가 K8s와의 혼동을 피하려고 SIG를 무엇으로 바꿨나?
   A: **TAG (Technical Advisory Group)**.
8. Q: OCI Runtime 스펙의 참조 구현체는?
   A: **runc**.
9. Q: 노드를 NotReady에서 Ready로 만들려면 설치해야 하는 오픈표준 계층은?
   A: **CNI (Container Network Interface)**.
10. Q: SLA / SLO / SLI를 각각 한 단어로.
    A: 합의 / 목표 / (실측) 지표.
11. Q: 서버리스의 이벤트 데이터 상호운용을 위한 CNCF 표준은?
    A: **CloudEvents**.
12. Q: Zero Trust의 핵심 문구는?
    A: **never trust, always verify**.

