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
