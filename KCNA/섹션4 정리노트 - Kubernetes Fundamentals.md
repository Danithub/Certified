# 📘 섹션4 정리노트 — Kubernetes Fundamentals

> 출처: James Spurin — Kubernetes Certified (KCNA), Section 4
> 시험 비중: Kubernetes Fundamentals 46% (KCNA 최대 도메인)
> 아래는 강의 영상 핵심 요약. 헷갈리는 개념은 오답노트로 연계.

---

## 목차
- 0. 강의 흐름 한눈에 보기 (서술형)
- 4-1. Container Orchestration Introduction (컨테이너 오케스트레이션 입문)
  - 4-1-1. 왜 오케스트레이션이 필요한가 — 스케일의 문제
  - 4-1-2. 컨테이너 오케스트레이션이란
  - 4-1-3. 오케스트레이션이 뛰어난 영역 (8가지)
  - 4-1-4. CRD (Custom Resource Definition)
  - 4-1-5. 오케스트레이터 경쟁과 Kubernetes의 승리
  - 4-1-6. OpenShift — Kubernetes로의 피벗
- 4-2. Kubernetes Architecture (쿠버네티스 아키텍처)
  - 4-2-1. 아키텍처 개요 — Control Plane과 Node
  - 4-2-2. 저수준 컨테이너 런타임 (Low Level Container Runtime, runc)
  - 4-2-3. 고수준 컨테이너 런타임 / 컨테이너 엔진 (containerd)
  - 4-2-4. kubelet — Control Plane과 Node 양쪽에서 실행
  - 4-2-5. Static Pods와 manifests 디렉터리
  - 4-2-6. etcd
  - 4-2-7. kube-apiserver
  - 4-2-8. kube-scheduler
  - 4-2-9. Service와 kube-proxy
  - 4-2-10. CoreDNS와 Deployment
  - 4-2-11. Controller-Manager
  - 4-2-12. Cloud-Controller-Manager
  - 4-2-13. 고가용성(HA) 구성과 Raft
  - 4-2-14. 컴포넌트 배치 요약표
- 4-3. Kubernetes Pods - Part 1 (쿠버네티스 파드 - 1부)
  - 4-3-1. Pod란 무엇인가 — 컴퓨트의 최소 배포 단위
  - 4-3-2. Pod 내부의 네트워킹과 통신 (localhost · 고유 IP · IPC)
  - 4-3-3. 단일 컨테이너 Pod 실행하기 (kubectl run)
  - 4-3-4. Pod 로그와 상세 정보 보기 (logs · get pods -o wide)
  - 4-3-5. Pod IP 접근성 — 어디서 접근 가능한가
  - 4-3-6. kubectl port-forward — 로컬에서 서비스 접근
  - 4-3-7. 다른 Pod에서 접근하기 (curl 파드)
  - 4-3-8. 실행 중인 Pod에 명령 실행 (kubectl exec · ubuntu 파드)
  - 4-3-9. Pod 정리 (kubectl delete --now)
  - 4-3-10. 명령어 요약표
- 4-4. Kubernetes Pods - Part 2 (YAML로 파드 만들기)
  - 4-4-1. YAML이란 & kubectl로 YAML 빠르게 생성하기 (--dry-run)
  - 4-4-2. restartPolicy와 kubectl explain
  - 4-4-3. create vs apply (명령형 vs 선언형)
  - 4-4-4. 하나의 YAML에 여러 선언 담기 (--- 구분자)
  - 4-4-5. 파일로 적용·삭제 (-f 옵션)
  - 4-4-6. 명령어 요약표
- 🎯 시험 대비 핵심 암기 체크
- 🧪 셀프 체크

---

## 0. 강의 흐름 한눈에 보기 (서술형)

컨테이너의 이점과 활용이 폭발적으로 늘면서, 그 컨테이너들을 "어떻게 운영할 것인가"라는 새로운 문제가 따라왔습니다. 소수의 컨테이너를 직접 손으로 돌리는 것은 어렵지 않습니다. 하지만 인프라가 수백, 수천, 수만 개 이상의 컨테이너로 확장되면 사람 손으로는 감당할 수 없습니다. 바로 이 "대규모 운영(Containers at Scale)"의 문제를 풀어주는 것이 컨테이너 오케스트레이터입니다.

**오케스트레이션 = 컨테이너 운영의 자동화.** 컨테이너 오케스트레이션은 본질적으로 컨테이너를 운영하는 데 필요한 작업들, 즉 프로비저닝·배포·스케일링 같은 일들을 자동화하는 것입니다. 여기에 더해 복잡한 애플리케이션 배포를 표준화할 수 있는 표준(Standards)과 프레임워크(Frameworks)를 제공하고, 네트워킹·스토리지·보안·오토스케일링 같은 구성 요소와의 통합까지 함께 제공합니다.

**경쟁을 뚫고 표준이 된 Kubernetes.** 이 매력적인 시장 기회를 두고 한때 OpenShift, Docker Swarm, Nomad, 그리고 Kubernetes가 경쟁했습니다. 결과적으로 Kubernetes가 오케스트레이터 경쟁에서 승리해 컨테이너 오케스트레이션의 "표준(gold standard)"이 되었고, 가장 인기 있는 오픈소스 프로젝트 중 하나가 되었습니다. 경쟁자들 중 일부는 여전히 존재하지만 점유율은 작고, 일부는 사라졌으며, 일부는 아예 자신들의 솔루션을 Kubernetes 위로 방향 전환(pivot)했습니다. 그 대표적인 예가 OpenShift로, 그 핵심에는 Kubernetes가 자리 잡고 있습니다. 이 강의는 Kubernetes에 초점을 두므로, 이제부터 컨테이너 오케스트레이터로서의 Kubernetes를 본격적으로 다루게 됩니다.

**아키텍처는 두 축으로 나뉜다 — Control Plane과 Node.** 4-2에서는 Kubernetes를 아키텍처 관점에서 해부합니다. 크게 주요 컴포넌트 대부분이 실행되는 **컨트롤 플레인(Control Plane)**과, 실제 워크로드가 실행되는 **노드(Node)**로 나뉩니다. 가장 아래에는 컨테이너를 실제로 띄우는 **저수준 컨테이너 런타임(runc)**이 있고, 그 위에 이를 관리하는 **고수준 컨테이너 런타임/컨테이너 엔진(containerd)**이 있습니다. 그리고 파드를 유지·관리하는 **kubelet**이 컨트롤 플레인과 노드 **양쪽 모두**에서 돌며, manifests 디렉터리를 감시해 **스태틱 파드**로 핵심 컴포넌트(etcd·apiserver·scheduler·controller-manager 등)를 띄웁니다. 이 조각들이 어떻게 맞물려 하나의 클러스터가 되는지, 그리고 고가용성(HA) 구성에서는 어떻게 확장되는지를 순서대로 따라갑니다.

---

## 4-1. Container Orchestration Introduction (컨테이너 오케스트레이션 입문)

### 한 줄 요약
컨테이너를 **대규모(수백~수만 개 이상)** 로 운영하려면 손으로는 불가능하다. **컨테이너 오케스트레이터**가 프로비저닝·배포·스케일링 등 운영 작업을 **자동화**하고, 네트워킹·스토리지·보안·오토스케일링과의 **통합**을 표준화된 프레임워크로 제공한다. 이 경쟁의 승자가 **Kubernetes**다.

---

### 4-1-1. 왜 오케스트레이션이 필요한가 — 스케일의 문제

- 컨테이너의 이점·활용이 늘수록 오케스트레이션 수요·필요성도 함께 증가.
- **소수의 컨테이너**를 직접 운영하는 것은 비교적 간단(straightforward).
- 그러나 인프라가 **수백 → 수천 → 수만 개 이상**으로 확장되면 사람 손으로 관리 불가.
- 이 **"Containers at Scale(대규모 컨테이너 운영)"** 문제를 풀어주는 것이 **Container Orchestrator**.

> 🎯 포인트: 오케스트레이션의 존재 이유 = **스케일**. 컨테이너 수가 폭증할 때 운영을 자동화하기 위함.

---

### 4-1-2. 컨테이너 오케스트레이션이란

- **정의**: 컨테이너 운영에 필요한 작업들을 **자동화(automating the operational needs)** 하는 것.
  - 대표 작업: **프로비저닝(Provisioning), 배포(Deployment), 스케일링(Scaling)** 등.
- **표준·프레임워크 제공**: 복잡한 애플리케이션의 배포를 **표준화**하는 데 손쉽게 활용할 수 있는 Standards & Frameworks 제공.
- **통합(Integration) 제공**: 다음 구성 요소들과의 통합을 함께 제공.
  - **네트워킹(Networking), 스토리지(Storage), 보안(Security), 오토스케일링(Autoscaling)**

> 🎯 포인트: 오케스트레이션 = **운영 자동화 + 배포 표준화 + 주변 구성요소 통합**.

---

### 4-1-3. 오케스트레이션이 뛰어난 영역 (8가지)

컨테이너 오케스트레이션이 특히 강점을 보이는 영역:

| # | 영역 | 설명 |
| --- | --- | --- |
| 1 | **프로비저닝 & 배포** | 컨테이너의 provisioning과 deployment |
| 2 | **가용성 & 자가 치유** | Container Availability & Self Healing |
| 3 | **스케줄링 & 자원 효율** | 클러스터 내 Compute Resources의 효과적 사용 |
| 4 | **서비스 노출** | 컨테이너 서비스를 **네트워크 접근 가능**하게(Exposing) 함 |
| 5 | **인증 & 보안** | Authorisation & Security |
| 6 | **스토리지 활용** | 공유(Shared) 및 영속(Persistent) 워크로드용 스토리지 |
| 7 | **오토스케일링** | Autoscaling |
| 8 | **확장 기능** | Extended Functionality (예: **CRD** 활용) |

> 🎯 시험 포인트: 이 8가지는 "오케스트레이터가 왜 필요한가"의 답이자, Kubernetes 기능 전반과 그대로 매핑된다.

---

### 4-1-4. CRD (Custom Resource Definition)

- Kubernetes에서 **CRD = Custom Resource Definition(커스텀 리소스 정의)**.
- **의미**: Kubernetes의 **핵심(core) 기능 밖의 기능**을 갖도록 **확장(extend)** 하는 수단.
- **예시**: **MySQL CRD 객체** → MySQL 인스턴스를 **관리·제어**하는 전용 리소스로 사용.
- 4-1-3의 "확장 기능(Extended Functionality)"을 실현하는 대표 메커니즘.

> 🎯 시험 포인트: **CRD = Kubernetes를 기본 기능 너머로 확장하는 커스텀 리소스**. 특정 애플리케이션(예: MySQL)을 위한 전용 오브젝트를 정의할 수 있다.

---

### 4-1-5. 오케스트레이터 경쟁과 Kubernetes의 승리

- 과거 컨테이너 오케스트레이션 시장은 **엄청난 기회**를 두고 여러 솔루션이 경쟁.
- 경쟁했던 오케스트레이터:

| 오케스트레이터 | 비고 |
| --- | --- |
| **Kubernetes** | 경쟁의 **승자**, 오케스트레이션의 **표준(gold standard)** |
| **Docker Swarm** | 여전히 존재하나 점유율 작음 |
| **Nomad** | (HashiCorp) 여전히 존재하나 점유율 작음 |
| **OpenShift** | Kubernetes 위로 **피벗**(아래 4-1-6) |

- **Kubernetes**는 경쟁에서 승리 → 컨테이너 오케스트레이션의 **표준**이자 **가장 인기 있는 오픈소스 프로젝트 중 하나**.
- 대안들의 운명: 일부는 **존속(점유율 작음)**, 일부는 **소멸**, 일부는 **Kubernetes로 방향 전환(pivot)**.

> 🎯 시험 포인트: **Kubernetes = 오케스트레이션의 사실상 표준(de facto standard)**. 경쟁자(Swarm/Nomad/OpenShift)를 함께 기억.

---

### 4-1-6. OpenShift — Kubernetes로의 피벗

- OpenShift는 솔루션을 Kubernetes 쪽으로 **피벗한 대표 사례**.
- ⭐ **OpenShift의 핵심(core)은 Kubernetes**.
- 그 위에 **OpenShift 프레임워크**가 얹혀 **추가 기능(additional functionality)** 과 **지원 모델(support model)** 을 제공.
- 이 지원 모델은 **대규모 조직(larger organizations)** 들이 선호.

> 🎯 포인트: **OpenShift = Kubernetes 코어 + 부가 프레임워크·엔터프라이즈 지원**. "Kubernetes를 대체"가 아니라 "Kubernetes 위에 구축".

> 강의 마무리: 이 강의는 Kubernetes에 초점 → 이후 영상들에서 Kubernetes를 컨테이너 오케스트레이터로서 **심층적으로** 다룬다.

---

## 4-2. Kubernetes Architecture (쿠버네티스 아키텍처)

### 한 줄 요약
Kubernetes는 **Control Plane(주요 컴포넌트 실행)** 과 **Node(워크로드 실행)** 로 구성된다. 최하위에 **runc(저수준 런타임)**, 그 위에 **containerd(고수준 런타임/엔진)**, 그리고 파드를 관리하는 **kubelet**이 **양쪽 모두**에서 동작한다. 핵심 컴포넌트(**etcd, kube-apiserver, kube-scheduler, controller-manager**)는 manifests 디렉터리 기반의 **Static Pod**로 뜨고, **kube-proxy**는 DaemonSet, **CoreDNS**는 Deployment로 실행된다.

---

### 4-2-1. 아키텍처 개요 — Control Plane과 Node

- Kubernetes 아키텍처는 크게 **두 영역**으로 나뉜다.
  - **Control Plane(컨트롤 플레인)**: 주요 컴포넌트 대부분이 실행되는 곳.
  - **Node(노드)**: 일반적으로 **워크로드(사용자 애플리케이션)** 가 실행되는 곳.
- 설명 순서: 먼저 **단일 Control Plane + 단일 Node**로 이해 → 이후 **고가용성(HA)** 구성으로 확장.
- 그림 기준: 왼쪽 = Control Plane, 오른쪽 = Node.

> 🎯 포인트: "어디서 실행되는가"를 항상 의식하라. 컴포넌트마다 **Control Plane 전용 / Node 전용 / 양쪽 공통**이 구분된다.

---

### 4-2-2. 저수준 컨테이너 런타임 (Low Level Container Runtime, runc)

- **역할**: 컨테이너를 **실제로 생성하고 실행(spawning & running)** 하는 가장 낮은 수준의 컴포넌트.
- 컨테이너화를 가능하게 하는 **저수준 리눅스 기능**과 직접 상호작용.
  - **Linux Namespaces(네임스페이스)** + **Cgroups(제어 그룹)**
- **참조 구현체 = `runc`** ⭐ (기억 필수)
  - Docker가 **OCI(Open Container Initiative)** 에 **기증(donated)**.
  - **OCI 호환** 컨테이너 런타임.
- **대체 OCI 호환 런타임들**:
  - **crun** — C로 작성된 컨테이너 런타임
  - **kata** 런타임
  - **gVisor**
- "저수준(low level)"이라는 용어는 **고수준 컨테이너 런타임(= 컨테이너 엔진)** 과 구분하기 위해 붙인다.
- 보통 저수준 런타임을 **직접 설치하지 않는다** → 고수준 런타임/엔진을 설치하면 그것이 runc를 설치·사용.

> 🎯 시험 포인트: **runc = OCI 참조 구현 저수준 런타임(Docker → OCI 기증)**. 대체재 = crun·kata·gVisor. 실제로 Namespaces·Cgroups를 다룬다.

---

### 4-2-3. 고수준 컨테이너 런타임 / 컨테이너 엔진 (containerd)

- **용어 정리**: **High Level Container Runtime = Container Engine(컨테이너 엔진)** → 같은 것.
- **대표 예 = `containerd`** ⭐
  - **Docker**가 만들고 내부에서 사용 → **CNCF에 기증** → 현재 CNCF **졸업(graduated) 프로젝트**.
- **역할**: **전체 컨테이너 라이프사이클**을 고수준에서 관리.
  - 컨테이너 **이미지 pull 및 저장**
  - 실행·감독(supervision)·네트워킹을 위해 **runc 같은 저수준 런타임과 상호작용**
- 리눅스에서 `yum`/`apt`로 **containerd 설치 시 → runc(저수준 런타임)도 자동 설치**.

> 🎯 시험 포인트: **containerd = 고수준 런타임(컨테이너 엔진), CNCF 졸업 프로젝트**. 이미지 관리 등 라이프사이클 전반을 맡고, 실제 실행은 runc에 위임.

---

### 4-2-4. kubelet — Control Plane과 Node 양쪽에서 실행

- 흔히 "kubelet = Kubernetes **노드**"로 언급되지만, ⭐ **kubelet은 노드 전용이 아니다.**
  - **Control Plane과 Node 양쪽 모두에서 실행**된다. (강사가 특히 강조한 포인트)
  - 많은 다이어그램이 kubelet을 노드 안에만 그려서 오해를 부른다.
- **역할**: Kubernetes에서 **파드(pod)를 유지·관리**하는 컴포넌트.
  - **Pod = 컴퓨트의 최소 단위**, 하나 이상의 컨테이너를 실행하는 **수단(vehicle)**.
- **Pod Spec 사용**: YAML/JSON으로 된 파드 명세를 받아, 해당 파드가 **실제 실행 중이고 healthy 한지 보장**.
- Control Plane의 **핵심 컴포넌트들도 kubelet이 실행** → 그래서 양쪽에 존재.
- **파드 명세를 받는 방식(2가지)**:
  1. **API 호출**을 통해 (kube-apiserver와 통신)
  2. **디렉터리 직접 감시** → `/etc/kubernetes/manifests`의 YAML 파일들

> 🎯 시험 포인트: **kubelet은 Control Plane + Node 양쪽에서 실행**되며 **파드의 생성·헬스를 책임**진다. 명세 입력 경로 = ①API ②manifests 디렉터리.

---

### 4-2-5. Static Pods와 manifests 디렉터리

- **manifests 디렉터리**: 일반적으로 **`/etc/kubernetes/manifests`**.
  - Kubernetes 실행에 필요한 **필수 Control Plane 서비스**의 YAML 파일들이 채워짐.
- **흐름(강의의 파란 화살표)**: 디렉터리의 YAML을 kubelet이 감시 →
  생성 요청을 **containerd(고수준)** → **runc(저수준)** 로 전달 → **Static Pod** 생성.
- **Static Pod(스태틱 파드)** = **디렉터리 내 파일을 기반으로 생성·관리되는 파드**.
- 아래에서 다룰 **etcd, kube-apiserver, kube-scheduler, controller-manager**가 이렇게 Static Pod로 뜬다.

> 🎯 시험 포인트: **Static Pod = manifests 디렉터리(파일) 기반으로 kubelet이 직접 띄우는 파드**. API 서버를 거치지 않고 kubelet이 로컬 파일로 관리.

---

### 4-2-6. etcd

- **정의**: **강한 일관성(strongly consistent)** 을 갖는 **분산 키-값 저장소(distributed key-value store)**.
- **특성**: 리더 선출(leader election), 네트워크 파티션 처리, **머신 장애 감내**(HA 설계).
- **역할**: Kubernetes의 **진실의 원천(source of truth)** → 클러스터의 **모든 데이터를 저장하는 백업 저장소(backing store)**.
- **프로덕션 권장**:
  - **홀수 개 멤버**로 여러 인스턴스 운영 → 이상적으로 **5개 멤버 클러스터**.
  - **etcd 백업**을 프로덕션 구현의 일부로 반드시 고려.
- Static Pod로 실행됨.

> 🎯 시험 포인트: **etcd = 분산 키-값 저장소, 클러스터의 source of truth**. 프로덕션은 **홀수 멤버(권장 5개)** + **백업 필수**.

---

### 4-2-7. kube-apiserver

- **위치/성격**: 클러스터의 **중심점(central point)**, 접근의 **주요 관문(main gateway)**. Static Pod로 실행.
- **누가 쓰나**: **모든 사용자**의 접근 + 클러스터 **내부 컴포넌트들**의 접근 모두 이 API 서버 경유.
- **인터페이스**: **RESTful API** 제공.
- **데이터 저장**: 모든 데이터를 **영구 저장 백엔드 = etcd** 에 저장.
- **통신(강의 색상 규칙)**:
  - **주황 화살표** = Control Plane 내부 컴포넌트 간 상호 통신.
  - **초록 화살표** = kubelet↔API 서버 간 파드 생성 통신(대체 워크플로).
  - **보라 화살표** = 인스턴스 간(cross instance) 트래픽 (Control Plane ↔ Node).
- **파드 생성 경로**: kubelet이 API 서버 요청을 받아 → containerd → runc 로 파드 생성 (Node에서도 동일).

> 🎯 시험 포인트: **kube-apiserver = 클러스터의 관문/중심, RESTful API, 데이터는 etcd에 저장**. 사용자·내부 컴포넌트의 모든 접근이 여기로 모인다.

---

### 4-2-8. kube-scheduler

- **문제 상황(강의 시연)**: kubectl로 nginx 파드 요청 → API 서버 → etcd 저장까지는 되지만,
  **스케줄러가 없으면 파드는 결코 생성되지 않고 실패**한다.
- **kube-scheduler = Control Plane 프로세스**, 역시 **Static Pod**(manifests 디렉터리의 텍스트 파일)로 시작.
- **역할**: **어느 노드가 유효한 배치 위치인지 결정** → 클러스터의 **제약 조건(constraints)과 리소스**에 따라 스케줄링.
- 노드가 하나면 그 노드에, 여러 개면 **현재 워크로드 기준 가장 적절한 노드**를 선택.

> 🎯 시험 포인트: **kube-scheduler = 파드를 어느 노드에 놓을지 결정**. 없으면 파드가 pending으로 남아 생성되지 않는다.

---

### 4-2-9. Service와 kube-proxy

- **Service(서비스)**: 애플리케이션에 **네트워크 접근·연결성**을 제공. (예: nginx를 포트 80으로 노출)
  - 생성 요청 → API 서버 → etcd 등록.
- **문제 상황**: **kube-proxy가 없으면 서비스는 동작하지 않는다.**
- **kube-proxy**:
  - **DaemonSet**으로 실행 → **모든 Control Plane 인스턴스 + 모든 Node**에서 **일반 파드(normal pod)** 로 실행.
  - ⚠️ **Static Pod가 아니다** → Kubernetes가 관리하는 **정상 파드**.
  - API 서버와 통신하며, 실행되는 모든 시스템에서 **TCP/UDP/SCTP 포워딩을 동적으로 구성**.
- kube-proxy가 뜨면 서비스가 **클러스터 어디에서나** 기대대로 동작.

> 🎯 시험 포인트: **kube-proxy = DaemonSet(모든 노드+컨트롤플레인), 일반 파드**, TCP/UDP/SCTP 포워딩 구성 → **서비스 네트워킹 담당**. Static Pod 아님에 주의.

---

### 4-2-10. CoreDNS와 Deployment

- **DNS 서비스**: Kubernetes 서비스는 자동으로 **DNS 해석(resolution)** 서비스를 받아,
  클러스터 어디서나 **친숙한 DNS 이름**으로 다른 컴포넌트와 통신 가능.
- 이를 담당하는 컴포넌트 = **CoreDNS** (유연하고 확장 가능한 DNS 서버).
- ⭐ **CoreDNS는 Kubernetes Deployment**로 실행됨 → "Deployment"라는 점이 중요.
- **Pod vs Deployment**:
  - 지금까지 다룬 것들은 대부분 **단순 Pod**.
  - **Deployment = 애플리케이션이 어떻게 실행되어야 하는지 기술** → **replicas(복제본 수)** 같은 개체 보유.
  - 예: "3개 인스턴스(3개 파드)가 떠 있어야 성공" 처럼 원하는 상태를 선언.

> 🎯 시험 포인트: **CoreDNS = 클러스터 DNS, Deployment로 실행**. Deployment는 replicas 등 **원하는 상태(desired state)** 를 선언하는 상위 개념.

---

### 4-2-11. Controller-Manager (c-m)

- Deployment가 기대대로 돌려면 **컨트롤러(controllers)를 처리·유지하는 컴포넌트**가 필요 → **Controller-Manager**.
- **Controller = 제어 루프(control loop)**: 클러스터 상태를 **모니터링**하고 필요한 변경을 요청.
  - 각 컨트롤러는 클러스터를 **원하는 상태(desired state)** 로 이동시키려 시도.
- **대표 컨트롤러**:
  - **Replication Controller** — 레플리카(복제본) 수 관리
  - **Node Controller** — 노드의 온라인/오프라인 상태 확인
  - **Deployment Controller** — 디플로이먼트 관리

> 🎯 시험 포인트: **Controller-Manager = 여러 컨트롤러(control loop)를 구동** → 현재 상태를 원하는 상태로 수렴. (Replication/Node/Deployment 컨트롤러 등)

---

### 4-2-12. Cloud-Controller-Manager (CCM)

- 있을 수도, 없을 수도 있는 컴포넌트 → 보통 **퍼블릭 클라우드가 제공하는 Kubernetes**에서 보임.
- **역할**: 해당 **클라우드 제공자의 기능을 Kubernetes 클러스터에 연결(bridge)**.
  - 예: 클라우드 컴포넌트 간 **경로 처리(route handling) 자동화**.
  - **로드 밸런서를 클라우드 제공자 offering과 직접 통합**.
- **예시**: AWS에서 LoadBalancer 서비스 사용 시 → **AWS Elastic Load Balancer**가 생성되며,
  이를 **CCM이 투명하게 처리**.

> 🎯 시험 포인트: **CCM = 클라우드 제공자 기능(라우팅·로드밸런서 등)을 Kubernetes에 연결**하는 브리지. 매니지드 K8s(EKS 등)에서 등장.

---

### 4-2-13. 고가용성(HA) 구성과 Raft

- **k8s** = kubernetes의 약어(가운데 8글자 → k**8**s).
- **HA 구성**: **Control 노드 3개** + etcd가 **클러스터로 동작**(서로 인식·통신).
- **Raft consensus(합의 프로토콜)**:
  - 분산 시스템에서 **각 노드가 장애 상황에서도 동일한 상태에 동의**하도록 보장하는 프로토콜.
  - Raft GitHub 페이지에 대화형 예제 있음(학습 추천).
- **API 서버 앞단 로드 밸런서**: 외부 연결(사용자·다른 노드)을 담당.
  - **어느 Control Plane 인스턴스에 접근하든 무관** → 모든 정보는 etcd에서 참조되고, 합의 프로토콜 덕에 **항상 올바른 상태**.
- HA 뷰에서는 nginx 파드가 **노드 중 하나에 스케줄링**된 모습을 확인.

> 🎯 시험 포인트: **HA = Control Plane 다중화 + etcd 클러스터(홀수) + Raft 합의 + API 앞 로드밸런서**. 어느 인스턴스에 붙어도 상태 일관성 보장.

---

### 4-2-14. 컴포넌트 배치 요약표

| 컴포넌트 | 실행 형태 | 실행 위치 | 핵심 역할 |
| --- | --- | --- | --- |
| **runc** | 저수준 런타임 | 모든 인스턴스 | 컨테이너 실제 생성(Namespaces/Cgroups) |
| **containerd** | 고수준 런타임/엔진 | 모든 인스턴스 | 이미지 관리·라이프사이클, runc 호출 |
| **kubelet** | 프로세스 | **Control Plane + Node** | 파드 유지·헬스 보장, manifests 감시 |
| **etcd** | Static Pod | Control Plane | 분산 KV 저장소, source of truth |
| **kube-apiserver** | Static Pod | Control Plane | 클러스터 관문, RESTful API |
| **kube-scheduler** | Static Pod | Control Plane | 파드를 어느 노드에 놓을지 결정 |
| **controller-manager** | Static Pod | Control Plane | 컨트롤러(control loop) 구동 |
| **cloud-controller-manager** | Static Pod | Control Plane | 클라우드 기능 브리지(옵션) |
| **kube-proxy** | **DaemonSet(일반 파드)** | 모든 인스턴스 | TCP/UDP/SCTP 포워딩(서비스) |
| **CoreDNS** | **Deployment** | 클러스터 | 클러스터 DNS 해석 |

> 🎯 암기 팁: **Static Pod 4형제 = etcd · apiserver · scheduler · controller-manager**(+옵션 CCM). **kube-proxy=DaemonSet**, **CoreDNS=Deployment**, **kubelet=양쪽 공통**.

## 4-3. Kubernetes Pods - Part 1 (쿠버네티스 파드 - 1부)

### 한 줄 요약
**Pod = Kubernetes에서 배포 가능한 최소 컴퓨트 단위**이자 **하나 이상의 컨테이너 그룹**이다. 같은 파드 안 컨테이너들은 **네트워킹을 공유**하며 **localhost**로 통신하고, 각 파드는 클러스터 내 **고유 IP**를 가진다. 파드 간에는 별도 NAT 없이 **IP만으로 상호 통신**이 가능하며, `kubectl run/logs/get/port-forward/exec/delete` 로 파드를 다룬다.

---

### 4-3-1. Pod란 무엇인가 — 컴퓨트의 최소 배포 단위

- **Pod = Kubernetes에서 배포 가능한 가장 작은 컴퓨트 단위(smallest deployable unit).**
- 본질적으로 **하나 이상(one or more)의 컨테이너 그룹**.
- 파드는 애플리케이션과 그 **의존성 · 공유 스토리지 · 네트워킹**을 **하나의 배포 단위**로 캡슐화할 수 있다.
- 따라서 파드 하나에 **애플리케이션 전체**를 담아 실행하는 것도 가능하다.

> 🎯 시험 포인트: **Pod = 배포 가능한 최소 단위 = 1개 이상 컨테이너의 묶음.** 애플리케이션 + 의존성 + 스토리지 + 네트워킹을 하나로 캡슐화.

---

### 4-3-2. Pod 내부의 네트워킹과 통신 (localhost · 고유 IP · IPC)

- **네트워킹 공유**: 같은 파드 안의 컨테이너들은 파드가 제공하는 **네트워킹을 공유**한다.
- **localhost 통신**: 파드 내부 컨테이너끼리는 **localhost**로 서로 통신한다.
- **고유 IP**: 각 파드는 클러스터 내에서 **자신만의 고유 IP 주소**를 할당받는다.
- **IPC 통신**: 파드 내 컨테이너들은 **IPC(Inter Process Communication, 프로세스 간 통신)** 로도 통신 가능.

> 🎯 시험 포인트: 파드 **내부** 컨테이너 통신 = **localhost + IPC**(네트워킹 공유). 파드마다 **고유 IP** 1개.

---

### 4-3-3. 단일 컨테이너 Pod 실행하기 (kubectl run)

- 예시: **nginx 웹 서버** 컨테이너 하나로 간단한 파드를 실행.
- 실행 명령:

```bash
kubectl run nginx --image=nginx
```

- `run` = 파드 실행 파라미터, `nginx` = **파드 이름**, `--image=nginx` = 공식 **nginx 컨테이너 이미지** 사용.
- 실행 후 파드 목록 확인:

```bash
kubectl get pods
```

> 🎯 포인트: `kubectl run <파드이름> --image=<이미지>` → 단일 컨테이너 파드가 뜬다.

---

### 4-3-4. Pod 로그와 상세 정보 보기 (logs · get pods -o wide)

- `get pods`만으로는 파드 정보나 기동 과정 가시성이 부족 → **로그**로 확인.

```bash
kubectl logs nginx        # 파드 이름이 nginx인 파드의 로그
```

- 강사 팁: 평소엔 `name/component` 형태(예: `pod/nginx`)를 표준으로 쓰지만,
  **파드는 Kubernetes의 기본 개체**라 그냥 `kubectl logs nginx`로도 조회 가능.
- 더 자세한 정보는 `-o wide` 옵션으로:

```bash
kubectl get pods -o wide  # 파드의 IP 주소, 실행 중인 노드 등 추가 정보 표시
```

> 🎯 포인트: `kubectl logs <파드>` = 로그 확인 / `kubectl get pods -o wide` = **IP·노드** 등 상세 정보.

### 4-3-5. Pod IP 접근성 — 어디서 접근 가능한가

- 파드 IP에 접근 가능한지는 **어떻게, 어디서** Kubernetes와 kubectl을 실행하는지에 따라 달라진다.
  - kubectl을 **Kubernetes 인프라(클러스터 노드) 위에서** 직접 실행하는가?
  - 아니면 **데스크톱 등 외부**에서 실행하는가?
- 강의 예시: kubectl을 **클러스터 인프라 위에서** 직접 실행 → 시스템이 클러스터의 일부이므로 **파드 IP로 ping 가능**.
- **파드 IP는 클러스터 내 어느 노드에서든 접근 가능** → 다른 노드로 `ssh` 후 그 노드에서 `ping` 해도 동작.
- 랩 환경에는 편의용 **리버스 프록시**가 있어, 특정 Kubernetes 인스턴스(Control Plane / Worker1 / Worker2) 시점에서 웹 서비스에 접속해볼 수 있다.

> 🎯 시험 포인트: **파드 IP는 클러스터 내부(모든 노드)에서 접근 가능.** 외부(데스크톱)에서의 접근 가능 여부는 실행 위치·환경에 따라 다르다.

---

### 4-3-6. kubectl port-forward — 로컬에서 서비스 접근

- 클러스터가 어디에 있든(로컬·클라우드) **kubectl로 네트워크 서비스에 접근**하는 강력한 방법.
- 도움말 예시가 유용:

```bash
kubectl port-forward --help
kubectl port-forward nginx 8080:80   # 로컬 8080 → 파드의 80 포트로 터널
```

- kubectl을 실행 중인 시스템의 **8080 포트**에서 리슨 → 파드의 **80 포트**로 **포트 포워딩 터널** 생성.
- 리슨 IP가 `127.0.0.1`(localhost)로 표시됨 → 로컬 머신이라면 브라우저로
  `localhost:8080` 또는 `127.0.0.1:8080` 접속 시 **nginx 환영 페이지** 확인 가능.
- 종료: `Ctrl-C`로 터널을 닫는다.

> 🎯 시험 포인트: **port-forward = 로컬 포트를 파드 포트로 터널링**. 클러스터 위치와 무관하게 로컬에서 서비스에 접근하는 표준 기법.

### 4-3-7. 다른 Pod에서 접근하기 (curl 파드)

- 실행 중인 파드를 **다른 파드**에서 조회하는 방법. `curl`이 담긴 이미지를 활용.
- `curl` = 명령줄에서 웹사이트와 상호작용하는 CLI 도구. 편리한 `curlimages/curl` 이미지 사용.

```bash
kubectl run -it --rm curl \
  --image=curlimages/curl \
  --restart=Never \
  -- curl <nginx 파드 IP>
```

- 옵션 의미:
  - `-it` = 인터랙티브, `--rm` = 종료 후 **자동 정리(clean up)**.
  - `curl` = 파드 이름, `--image=curlimages/curl` = 사용할 이미지.
  - `--restart=Never` = 실패해도 재시작하지 않음.
  - `--` = **파라미터 구분자**, 뒤에 오는 것은 컨테이너에 전달할 명령/인자.
  - 이 이미지의 기본 명령이 `curl`이므로, 인자로 **조회할 nginx 파드 IP**를 전달.
- 결과: **nginx 환영 페이지 HTML**이 출력됨.
- **핵심**: 서로 독립된 **두 파드가 통신**했다는 점. 두 파드가 **어느 노드에 있든**,
  **NAT 같은 복잡한 네트워크 설정 없이 파드 IP만으로 통신**할 수 있다.
- `--rm` 덕분에 종료 후 curl 파드는 자동 삭제되어 목록엔 다시 nginx만 남는다.

> 🎯 시험 포인트: **파드 간 통신 = 파드 IP로 직접, NAT 불필요.** `--rm`은 종료 시 파드 자동 정리, `--restart=Never`는 재시작 안 함.

### 4-3-8. 실행 중인 Pod에 명령 실행 (kubectl exec · ubuntu 파드)

- 백그라운드로 도는 **ubuntu 파드**를 하나 실행 (sleep로 계속 유지):

```bash
kubectl run ubuntu --image=ubuntu -- sleep infinity
```

- `sleep infinity` = 무한정 sleep → 파드를 계속 살려 둠. `get pods`에서 실행 확인.
- 이제 실행 중인 파드 안에서 **추가 프로세스**를 실행할 수 있다 → `kubectl exec`:

```bash
kubectl exec -it ubuntu -- bash   # ubuntu 파드 안에서 bash 셸 실행
```

- 컨테이너 안으로 진입 → 프롬프트 색상·호스트명이 바뀜.
- `ps -ef` 확인 시:
  - **PID 1** = 파드 기동 시 시작된 프로세스 = `sleep infinity`.
  - `kubectl exec`로 스폰된 **bash 셸**이 보이며, 그 bash가 부모 프로세스로 잡힌다.
- 컨테이너 내부는 표준 Ubuntu처럼 다룰 수 있음:

```bash
apt update && apt install curl   # 컨테이너 안에서 패키지 설치
curl <nginx 파드 IP>             # 설치한 curl로 nginx 조회
```

> 🎯 시험 포인트: `kubectl exec -it <파드> -- <명령>` = **실행 중인 파드 안에서 추가 프로세스 실행**. 파드의 최초 프로세스가 **PID 1**.

### 4-3-9. Pod 정리 (kubectl delete --now)

- 셸에서 `exit` 후 파드들을 삭제:

```bash
kubectl delete pod/nginx pod/ubuntu --now
```

- **버전별 차이**:
  - **구버전** Kubernetes: 지연 없이 즉시 제거하려면 `--grace-period=0` 사용.
  - **신버전** Kubernetes: `--now` 만으로 충분 (타이핑·기억이 더 쉬움).

> 🎯 포인트: `kubectl delete pod/<이름> --now` = 지연 없이 즉시 삭제. (구버전은 `--grace-period=0`)

> 강의 마무리: 지금까지는 전부 **명령줄로 만든 단일 컨테이너 파드**였다.
> 다음(Part 2 예고)에는 **YAML 선언(declaration)** 으로 파드를 만드는 방법을 다룬다.

---

### 4-3-10. 명령어 요약표

| 명령어 | 용도 |
| --- | --- |
| `kubectl run nginx --image=nginx` | 단일 컨테이너 파드 생성 |
| `kubectl get pods` | 파드 목록 확인 |
| `kubectl get pods -o wide` | 파드 IP·노드 등 상세 정보 |
| `kubectl logs nginx` | 파드 로그 확인 |
| `kubectl port-forward nginx 8080:80` | 로컬 8080 → 파드 80 터널 |
| `kubectl run -it --rm curl --image=curlimages/curl --restart=Never -- curl <IP>` | 다른 파드에서 조회(1회성) |
| `kubectl run ubuntu --image=ubuntu -- sleep infinity` | 백그라운드 유지용 파드 |
| `kubectl exec -it ubuntu -- bash` | 실행 중 파드에서 셸 실행 |
| `kubectl delete pod/nginx pod/ubuntu --now` | 파드 즉시 삭제 |

> 🎯 암기 팁: **run(생성) · get -o wide(상세) · logs(로그) · port-forward(터널) · exec(진입) · delete --now(삭제)** 6개 흐름.

## 4-4. Kubernetes Pods - Part 2 (YAML로 파드 만들기)

### 한 줄 요약
파드는 **YAML 선언**으로도 만들 수 있다. `kubectl run ... --dry-run=client -o yaml`로 **YAML 초안을 빠르게 생성**하고, `kubectl explain`으로 스펙을 조회한다. 적용은 **명령형 `create`**(한 번만 가능)와 **선언형 `apply`**(반복 적용·수정 가능)로 나뉘며, 실무는 대개 **apply**를 쓴다. 하나의 YAML 파일에 **`---` 구분자**로 여러 파드를 담아 `-f`로 한 번에 생성·삭제할 수 있다.

---

### 4-4-1. YAML이란 & kubectl로 YAML 빠르게 생성하기 (--dry-run)

- **YAML** = 설정 파일 정의에 자주 쓰이는 간단한 **데이터 직렬화 언어**.
- kubectl로 필요한 **YAML 초안을 빠르게 확보**할 수 있다.
- 앞서 쓴 `kubectl run nginx --image=nginx`를 변형:

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml | tee nginx.yaml
```

- `--dry-run=client` = 실제로 만들지 않고 **결과만 시뮬레이션**.
- `-o yaml` = 출력을 **YAML 형식**으로.
- `| tee <파일>` = **T 자 분기**처럼 화면 출력 + 파일 저장을 **동시에** 수행.

> 🎯 포인트: `--dry-run=client -o yaml | tee 파일` = **파드 YAML 초안을 즉석에서 생성·저장**하는 표준 기법.

### 4-4-2. restartPolicy와 kubectl explain

- 생성된 YAML에는 많은 정보가 담기며, 그중 **`restartPolicy`** 에 주목.
- **`restartPolicy` 기본값 = `Always`** → 파드 실패 시 **항상 재시작 시도**.
- 유효한 값 3가지: **`Always` · `Never` · `OnFailure`**.
- 스펙의 특정 항목을 더 알고 싶으면 **`kubectl explain`** 사용:

```bash
kubectl explain pod.spec.restartPolicy
```

- `pod` = kind, `.spec.restartPolicy` = 조회할 경로 → 가능한 값과 참고 링크를 보여준다.

> 🎯 시험 포인트: **restartPolicy = Always(기본) / Never / OnFailure.** 스펙 조회는 `kubectl explain <kind>.<경로>`.

---

### 4-4-3. create vs apply (명령형 vs 선언형)

- YAML 파일은 **`kubectl create`** 또는 **`kubectl apply`** 로 실행할 수 있다.
- **`create` = 명령형(Imperative)**.
  - `create` · `replace` · `delete` 가 모두 명령형으로 분류됨.
  - 이미 만든 것을 **다시 create 하면 에러** (한 번만 생성 가능).
- **`apply` = 선언형(Declarative)**.
  - "이 개체가 **이런 모습이길 원한다**"고 선언 → 없으면 만들고 있으면 맞춰 갱신.
  - 앞으로 YAML을 **수정할 계획이면 처음부터 apply로 시작**하는 것이 좋다.
  - `create`로 만든 리소스에 `apply`하면 **경고(warning)** 가 뜨지만, 이후엔 경고 없이 계속 apply 가능.
- 실무에서는 **대부분 apply**를 사용. (둘 다 개체가 없을 때는 동작함)

```bash
kubectl create -f nginx.yaml   # 두 번째 실행 시 이미 존재 → 에러
kubectl apply  -f nginx.yaml   # create했던 리소스면 첫 apply에 경고, 이후엔 조용히 적용
```

> 🎯 시험 포인트: **create/replace/delete = 명령형(한 번만)**, **apply = 선언형(원하는 상태 선언, 반복·수정 가능)**. 수정 예정이면 apply로 시작.

### 4-4-4. 하나의 YAML에 여러 선언 담기 (--- 구분자)

- ubuntu 파드도 같은 방식으로 YAML 생성:

```bash
kubectl run ubuntu --image=ubuntu --dry-run=client -o yaml -- sleep infinity | tee ubuntu.yaml
kubectl apply -f ubuntu.yaml
```

- 현재는 nginx.yaml, ubuntu.yaml **두 개의 파일**이지만, **하나의 YAML 파일에 여러 선언**을 담을 수 있다.
- **`---`** = YAML에서 항목(엔트리) 사이를 나누는 **구분자(separator)**.
- 두 파일을 하나로 합치기 (중괄호로 명령을 그룹화):

```bash
{ cat nginx.yaml; echo ---; cat ubuntu.yaml; } | tee combined.yaml
```

- `{ ... }` = 명령들을 그룹화, 각 명령은 `;`로 구분, 중간의 `echo ---`로 두 선언을 분리.

> 🎯 시험 포인트: **YAML의 `---` = 여러 리소스 선언을 한 파일에 담는 구분자.**

---

### 4-4-5. 파일로 적용·삭제 (-f 옵션)

- 합쳐진 단일 파일을 apply하면 **두 개체가 동시에 생성**된다:

```bash
kubectl apply -f combined.yaml    # nginx + ubuntu 동시 생성
```

- 삭제도 파일을 지정(`-f`)해 한 번에:

```bash
kubectl delete -f combined.yaml --now   # 파일 내 모든 개체 삭제, --now로 즉시
```

- `-f <파일>` = 해당 파일에 정의된 리소스들을 대상으로 지정. `--now`로 지연 없이 삭제.

> 🎯 포인트: **`-f 파일` = 파일 기반 적용/삭제.** combined 파일로 여러 파드를 **한 번에 생성·삭제**.

---

### 4-4-6. 명령어 요약표

| 명령어 | 용도 |
| --- | --- |
| `kubectl run nginx --image=nginx --dry-run=client -o yaml \| tee nginx.yaml` | 파드 YAML 초안 생성·저장 |
| `kubectl explain pod.spec.restartPolicy` | 스펙 항목 설명 조회 |
| `kubectl create -f nginx.yaml` | 명령형 생성(한 번만) |
| `kubectl apply -f nginx.yaml` | 선언형 적용(반복·수정 가능) |
| `{ cat nginx.yaml; echo ---; cat ubuntu.yaml; } \| tee combined.yaml` | 여러 선언을 한 파일로 결합 |
| `kubectl apply -f combined.yaml` | 파일 내 모든 개체 동시 생성 |
| `kubectl delete -f combined.yaml --now` | 파일 내 모든 개체 즉시 삭제 |

> 🎯 암기 팁: **dry-run(초안) → explain(조회) → create/apply(적용) → `---`(결합) → -f(파일 적용/삭제)** 흐름.

---

## 🎯 시험 대비 핵심 암기 체크

### 4-1 Container Orchestration
- [ ] 오케스트레이션의 존재 이유 = **스케일**(수백~수만 개 이상 컨테이너를 손으로 못 다룸)
- [ ] 컨테이너 오케스트레이션 = 컨테이너 **운영 작업의 자동화**(프로비저닝·배포·스케일링 등)
- [ ] 오케스트레이션은 배포 **표준화용 Standards & Frameworks** 제공
- [ ] 통합 대상: **네트워킹 · 스토리지 · 보안 · 오토스케일링**
- [ ] 오케스트레이션 강점 8영역: 프로비저닝/배포 · 가용성/자가치유 · 스케줄링/자원효율 · 서비스 노출 · 인증/보안 · 스토리지 · 오토스케일링 · 확장기능(CRD)
- [ ] **CRD = Custom Resource Definition** = Kubernetes **핵심 기능 밖으로 확장**하는 수단 (예: MySQL CRD)
- [ ] 오케스트레이터 경쟁자: **Kubernetes · Docker Swarm · Nomad · OpenShift**
- [ ] **Kubernetes = 경쟁 승자, 오케스트레이션의 표준(gold standard)**, 대표적 오픈소스 프로젝트
- [ ] 대안들: 일부 존속(점유율 작음) / 일부 소멸 / 일부 Kubernetes로 **피벗**
- [ ] **OpenShift의 코어 = Kubernetes** + 부가 프레임워크·지원 모델(대규모 조직 선호)

### 4-2 Kubernetes Architecture
- [ ] 아키텍처 두 축 = **Control Plane(주요 컴포넌트) + Node(워크로드)**
- [ ] **runc = OCI 참조 저수준 런타임**(Docker→OCI 기증), Namespaces·Cgroups 직접 제어 / 대체재 crun·kata·gVisor
- [ ] **containerd = 고수준 런타임(컨테이너 엔진), CNCF 졸업 프로젝트**, 이미지 관리 + runc 호출
- [ ] **kubelet은 Control Plane + Node 양쪽에서 실행** ⭐, 파드 유지·헬스 보장 / 명세 입력 = API 또는 manifests 디렉터리
- [ ] **Static Pod = manifests 디렉터리(`/etc/kubernetes/manifests`) 파일 기반**으로 kubelet이 직접 띄움
- [ ] **etcd = 분산 KV 저장소, source of truth** / 프로덕션 = 홀수 멤버(권장 5개) + 백업
- [ ] **kube-apiserver = 클러스터 관문/중심, RESTful API, 데이터는 etcd 저장**
- [ ] **kube-scheduler = 파드 배치 노드 결정**(없으면 파드 생성 실패)
- [ ] **kube-proxy = DaemonSet(모든 노드+컨트롤플레인), 일반 파드**, TCP/UDP/SCTP 포워딩 (Static Pod 아님)
- [ ] **CoreDNS = 클러스터 DNS, Deployment로 실행** / Deployment는 replicas 등 원하는 상태 선언
- [ ] **Controller-Manager = 컨트롤러(control loop) 구동**(Replication/Node/Deployment 컨트롤러)
- [ ] **Cloud-Controller-Manager = 클라우드 기능 브리지**(예: AWS ELB 자동 연동)
- [ ] **HA = Control Plane 다중화 + etcd 클러스터 + Raft 합의 + API 앞 로드밸런서**
- [ ] **Static Pod 4형제 = etcd · apiserver · scheduler · controller-manager**

### 4-3 Kubernetes Pods
- [ ] **Pod = 배포 가능한 최소 컴퓨트 단위 = 1개 이상 컨테이너의 그룹**
- [ ] 파드는 앱 + 의존성 + **공유 스토리지 + 네트워킹**을 하나의 단위로 캡슐화
- [ ] 파드 **내부** 컨테이너 통신 = **localhost + IPC**(네트워킹 공유)
- [ ] 각 파드는 클러스터 내 **고유 IP** 1개 보유
- [ ] **파드 IP는 클러스터 내부(모든 노드)에서 접근 가능**, 외부 접근은 실행 위치·환경에 따라 다름
- [ ] **파드 간 통신 = 파드 IP로 직접, NAT 불필요**
- [ ] `kubectl run <이름> --image=<이미지>` = 단일 컨테이너 파드 생성
- [ ] `kubectl get pods -o wide` = **IP·노드** 등 상세 정보
- [ ] `kubectl port-forward <파드> 8080:80` = **로컬 포트 → 파드 포트 터널**
- [ ] `--rm` = 종료 시 파드 자동 정리, `--restart=Never` = 재시작 안 함, `--` = 파라미터 구분자
- [ ] `kubectl exec -it <파드> -- bash` = 실행 중 파드에서 추가 프로세스 실행, 최초 프로세스는 **PID 1**
- [ ] `kubectl delete pod/<이름> --now` = 즉시 삭제 (구버전은 `--grace-period=0`)

### 4-4 Kubernetes Pods (YAML)
- [ ] **YAML = 데이터 직렬화 언어**, 설정 파일 정의에 자주 사용
- [ ] `--dry-run=client -o yaml | tee 파일` = **YAML 초안 생성·저장** (dry-run은 실제 생성 안 함)
- [ ] **restartPolicy = Always(기본) / Never / OnFailure**
- [ ] 스펙 조회 = `kubectl explain <kind>.<경로>` (예: `pod.spec.restartPolicy`)
- [ ] **create/replace/delete = 명령형(Imperative)**, 같은 것 두 번 create 시 에러
- [ ] **apply = 선언형(Declarative)**, 원하는 상태 선언 → 반복·수정 가능 (실무 기본)
- [ ] **YAML `---` = 한 파일에 여러 리소스 선언을 구분**하는 구분자
- [ ] **`-f 파일`** = 파일 기반 적용/삭제, `apply -f`/`delete -f`로 여러 개체 동시 처리

---

## 🧪 셀프 체크 (정답은 스스로 정정 → 틀리면 오답노트로)

### 4-1
1. 컨테이너 오케스트레이션이 필요해지는 근본 이유는 무엇인가?
2. 컨테이너 오케스트레이션이 자동화하는 대표적인 운영 작업 3가지는?
3. 오케스트레이션이 통합을 제공하는 구성 요소 4가지를 말하시오.
4. CRD의 정식 명칭과 역할을 설명하고, 강의에서 든 예시를 말하시오.
5. 컨테이너 오케스트레이터 경쟁자 4가지와, 그중 승자는?
6. OpenShift와 Kubernetes의 관계를 한 문장으로 설명하시오.

### 4-2
7. runc는 무엇이며 어떤 저수준 리눅스 기능을 다루는가? OCI와의 관계는?
8. containerd는 어떤 종류의 런타임이고 runc와 어떤 관계인가?
9. kubelet이 실행되는 위치는 어디인가? (강사가 강조한 포인트)
10. Static Pod란 무엇이며 어느 디렉터리를 기반으로 하는가?
11. Static Pod로 실행되는 Control Plane 핵심 컴포넌트 4가지를 말하시오.
12. kube-proxy의 실행 형태와 역할은? Static Pod인가?
13. CoreDNS는 어떤 워크로드 형태로 실행되는가? Deployment의 특징은?
14. kube-scheduler가 없으면 파드 생성 시 무슨 일이 벌어지는가?
15. HA 구성의 핵심 요소들과 Raft 합의의 목적을 설명하시오.

### 4-3
16. Pod의 정의를 한 문장으로 말하시오. 파드가 캡슐화하는 4가지는?
17. 같은 파드 안 컨테이너들은 어떻게 통신하는가? (2가지 방식)
18. 파드 IP는 어디에서 접근 가능한가? 외부 접근 가능 여부는 무엇에 좌우되는가?
19. 두 파드가 통신할 때 NAT가 필요한가? 그 이유는?
20. `kubectl port-forward nginx 8080:80` 명령이 하는 일을 설명하시오.
21. `--rm`, `--restart=Never`, `--`(더블 대시)의 각 의미는?
22. 실행 중인 파드 안에서 bash 셸을 실행하는 명령은? 파드의 최초 프로세스 PID는?
23. 파드를 지연 없이 즉시 삭제하는 최신 옵션과 구버전 옵션은 각각 무엇인가?

### 4-4
24. `--dry-run=client -o yaml`의 역할과, `tee`를 함께 쓰는 이유는?
25. restartPolicy의 기본값과 유효한 3가지 값은?
26. 특정 스펙 항목의 설명을 조회하는 명령은? (예: restartPolicy)
27. create와 apply의 차이를 명령형/선언형 관점에서 설명하시오.
28. 이미 존재하는 리소스를 다시 create하면 어떻게 되는가? create한 리소스에 apply하면?
29. YAML에서 여러 선언을 한 파일에 담을 때 쓰는 구분자는 무엇인가?
30. 하나의 combined.yaml로 여러 파드를 동시에 생성·삭제하는 명령은?
