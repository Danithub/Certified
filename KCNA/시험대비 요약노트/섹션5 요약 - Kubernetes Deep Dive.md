# 섹션5 요약 — Kubernetes Deep Dive

> 원본: kcna-transcripts-clean.md · 섹션5 / 시험대비 암기용
> 깊이: 상(자세히) · Container Orchestration(28%)·Security 다수 포함

## 0. 시험 포인트 한눈에 (암기표)

| 주제 | 반드시 외울 것 |
| --- | --- |
| API 요청 순서 | **Authentication → Authorization → Admission control** (그 후 validation) |
| Authorization 모드 | `--authorization-mode` (Node·**RBAC**·Webhook·ABAC). 기본 AlwaysAllow |
| API 포트 | HTTPS **6443** |
| RBAC 범위 | **Role/RoleBinding=namespace** / **ClusterRole/ClusterRoleBinding=클러스터** |
| RBAC 주체 | user·group(외부·인증서 CN/O) · **service account(K8s 객체)** |
| nodeName | 스케줄러 **우회**(직접 노드 지정) |
| Taint effect | **NoSchedule / NoExecute(축출) / PreferNoSchedule(soft)** |
| Affinity | required=**필터**(불충족 시 Pending) / preferred=**점수**(weight) |
| topologyKey | pod affinity 범위(예: kubernetes.io/hostname, zone) |
| reclaim 정책 | **Retain**(수동·데이터 유지) / **Delete**(동적 기본) / Recycle |
| StatefulSet | 안정 network ID·**pod별 PVC**·순서 보장. headless service 필요 |
| NetworkPolicy | 기본은 **allow all**, 정책 적용 시 격리. CNI 지원 필요 |
| Ingress vs Gateway | Gateway API = Ingress **후속**(역할 분리). Ingress API는 **폐기 아님** |
| PDB | **voluntary disruption** 시 가용성 보장(replicas와 다름) |
| 보안 4계층 | AuthN/AuthZ · Admission · Runtime · Network/Data |
| PSS 레벨 | **privileged / baseline / restricted** |
| PSA 모드 | **enforce / warn / audit** (namespace 라벨) |
| Helm vs Kustomize | Helm=패키지·**템플릿** / Kustomize=**kubectl 내장(-k)**·base+overlay |
| Service Mesh | mTLS·관측성·신뢰성. 표준 = **SMI** |

## 1. 핵심 개념 정리
### 1-1. Kubernetes API & CRD
- kube-apiserver = 사용자·컴포넌트 공통 인터페이스(RESTful, HTTPS **6443**).
  사용자(kubectl·Helm·client lib), 모니터링, 내부 컴포넌트(scheduler·kubelet·
  controller-manager), Admission Controller가 사용.
- **요청 처리 경로(핵심 3단계)**: 도착 → 라우팅 → **Authentication(AuthN)** →
  **Authorization(AuthZ)** → **Admission control** → validation → 처리 → 응답.
  - AuthN: 신원 확인(인증서·토큰·OIDC·service account).
  - AuthZ: 권한 확인. **`--authorization-mode`**(Node·RBAC·Webhook·ABAC).
    미지정 시 **AlwaysAllow**.
  - Admission control: 정책으로 **accept/reject/modify**(quota·기본값·검증).
- kubectl은 API의 **wrapper**. `--v`로 verbosity, `kubectl proxy`는 인증을
  붙인 HTTP 프록시.
- **CRD** = API 확장(코어 외 리소스 타입 정의).

### 1-2. RBAC
- Role Based Access Control = 리소스 접근 규제. **K8s는 user/group을 관리하지
  않음** → CA가 서명한 인증서의 subject(**CN=user, O=group**)로 신원 판단.
- **주체(subject) 3종**: **user**(외부), **group**(외부), **service account**
  (K8s 객체·namespace 소속).
- **역할·바인딩 4종**:
  - **Role**(namespace) + **RoleBinding**(namespace).
  - **ClusterRole**(클러스터 전역) + **ClusterRoleBinding**(전역).
- 기본 슈퍼유저: **cluster-admin** ClusterRole ↔ **system:masters** group.
- verbs: get·list·watch·create·delete 등. 확인: `kubectl auth can-i`.

### 1-3. Scheduler & nodeName
- kube-scheduler 3단계: **filtering(적합 노드 선별) → scoring(점수) →
  binding(바인딩)**. 바인딩되면 pod의 **nodeName** 필드에 노드 기록.
- **schedulerName**: 커스텀 스케줄러 지정(없으면 Pending).
- **nodeName** 직접 지정 = **스케줄러 우회**(특정 노드에 강제 배치).

### 1-4. Taints & Tolerations
- **Taint = 노드에** 적용(pod를 밀어냄). **Toleration = pod에** 적용(taint된
  노드에 허용).
- 형식: **key=value:effect**.
- **effect 3종**:
  - **NoSchedule**: 미허용 pod 신규 스케줄 안 됨. **기존 pod는 유지**.
  - **NoExecute**: 미허용 pod **즉시 축출(evict)** + 신규 금지.
  - **PreferNoSchedule**: soft(가능하면 회피).
- control plane 기본 taint: `node-role.kubernetes.io/control-plane:NoSchedule`
  (K3s·minikube 등은 제거/미적용). toleration operator: Exists / Equal.

### 1-5. Affinity (Node / Pod / Anti)
- 스케줄링 = **필터 + 점수**. 공통 규칙 이해가 핵심.
  - **required**DuringScheduling... = **필터(hard)**. 매칭 노드 없으면 **Pending**.
  - **preferred**DuringScheduling... = **점수(soft)**, **weight** 부여(합산).
- **Node Affinity**: **노드 라벨** 기준 배치.
- **Pod Affinity**: 이미 실행 중인 **pod 라벨** 기준 **같은 곳에 배치**(co-locate).
- **Pod Anti-Affinity**: **분리 배치**(복제본 분산·단일 장애점 감소).
- **topologyKey**: 규칙 적용 범위. 예: `kubernetes.io/hostname`(같은 호스트),
  `topology.kubernetes.io/zone`(같은 zone).

### 1-6. Storage (PV / PVC / StorageClass)
- **Ephemeral vs Persistent**: ephemeral은 재시작 후 소멸. 예 **emptyDir**
  (`medium: Memory` → 메모리 기반 캐시). Persistent는 컨테이너 삭제 후에도 유지.
- 3계층: **StorageClass → PV(PersistentVolume) → PVC(PersistentVolumeClaim)**.
  pod가 PVC로 볼륨 사용.
- 생성 방식: **Manual**(PV·PVC 직접 생성) vs **Dynamic**(PVC만 → PV 자동 생성).
- **Reclaim 정책**:
  - **Retain**: 수동 회수, 데이터 유지(수동 PV 기본).
  - **Delete**: 볼륨 자산 삭제(**동적 프로비저닝 기본**).
  - **Recycle**: 기본 스크럽(구식).
- CNCF graduated 스토리지 오케스트레이션 = **Rook**.

### 1-7. StatefulSets
- Stateful 앱용. **sticky identity**: pod 이름 `name-0`, `name-1`...(순서·고정).
- **ReplicaSet 없음**. 각 pod가 **자기 PVC(→ 자기 PV)** 보유(Deployment는 볼륨
  공유). pod 삭제·StatefulSet 삭제 후에도 **PVC/PV 유지**(state 기억).
- **안정적 network ID** 위해 **headless service** 필요:
  `hostname.servicename.namespace.svc.cluster.local`.
- **순서 보장**(ordered deploy/scale), updateStrategy **partition**(단계/카나리
  롤아웃).
- 용도: 다음 중 하나 이상 필요 시 — 안정적 고유 network ID, 안정적 영구
  스토리지, 순서 있는 배포·스케일, 순서 있는 롤링 업데이트.

### 1-8. NetworkPolicies
- pod를 **isolated**로 분류해 접근 제한(허용 pod·namespace·IP block 지정).
- **기본은 open**(대부분 CNI 기본 allow all). 정책을 적용해야 격리 시작.
- **NetworkPolicy를 지원하는 CNI**가 있어야 동작(예: Calico·Cilium).
- label selector로 대상 pod와 허용 ingress/egress 규칙 지정.

### 1-9. Ingress
- **HTTP/HTTPS 라우팅**(host·path 기반). 여러 라우팅 규칙을 하나의 리소스로
  통합, TLS termination 제공.
- 동작하려면 **Ingress Controller** 필요(nginx 등). 클러스터 기본 미포함일 수
  있음(설치 필요). **ingressClass**로 컨트롤러 지정, 기본 클래스 지정 가능.
- **주의**: ingress-nginx(커뮤니티) 컨트롤러는 은퇴 예정(~2026.3)이나,
  **Kubernetes Ingress API 자체는 폐기(deprecated)가 아님**(GA·지원 중).
- pathType: Prefix / Exact 등.

### 1-10. Gateway API
- Ingress의 **후속(successor)**. **역할 지향(role-oriented)** 설계로 책임 분리
  (인프라팀 / 운영팀 / 앱팀).
- **리소스 계층**:
  - **GatewayClass**: 클러스터 범위, 어떤 **컨트롤러**가 구현할지(인프라팀).
  - **Gateway**: namespace 범위, **listener**(port·protocol·hostname·TLS)
    정의(운영팀).
  - **HTTPRoute**(및 기타 Route): 라우팅 규칙(match·backendRefs),
    **parentRef**로 Gateway에 연결(앱팀).
  - Policy/Filter: 인증·rate limit·redirect 등(선택).
- Ingress 대비 강점: 다중 listener·TLS·다중 컨트롤러 공존, 어노테이션 의존 축소,
  트래픽 분할(weight)·리다이렉트(type 필터) 등 모듈식.

### 1-11. Pod Disruption Budgets (PDB)
- **자발적 중단(voluntary disruption)** 시 가용성 보장(유지보수·업그레이드·노드
  오토스케일). "voluntary"가 핵심 키워드.
- **replicas와 다름**: replicas는 정상 운영 중 가용성, PDB는 **자발적 중단으로
  부터 보호**.
- **minAvailable** / **maxUnavailable** 지정. `kubectl drain`이 PDB를 존중.
  관련 명령: **cordon**(스케줄 금지)·**uncordon**·**drain**(비우기).

### 1-12. Security 4계층 개요
- **① AuthN & AuthZ**(누가·무엇을): TLS·OIDC·service account·**RBAC**.
  AuthN=인증(N=authenticatioN), AuthZ=인가(Z=authoriZation).
- **② Admission control**(무엇을 생성/수정 허용): **Pod Security Admission**
  (privileged/baseline/restricted), OPA Gatekeeper·Kyverno(webhook).
- **③ Workload/Runtime settings**(컨테이너가 어떻게 실행): **security context**,
  Seccomp·AppArmor·SELinux, user namespaces, **gVisor**(하드닝 런타임).
- **④ Network & Data plane**(트래픽·데이터): **NetworkPolicy**, Ingress/Gateway,
  **encryption at rest**(etcd secret 암호화).

### 1-13. Security Contexts
- pod/컨테이너의 **권한·접근 제어** 설정.
- 주요 필드: **runAsUser·runAsGroup**, **runAsNonRoot**,
  **allowPrivilegeEscalation: false**, **capabilities**(drop all),
  **privileged**, seccompProfile.
- 예: runAsNonRoot로 root 실행 차단, allowPrivilegeEscalation false로 권한
  상승(rogue binary) 차단.

### 1-14. Pod Security Admission (PSA)
- API 서버의 **Admission Controller**. pod 생성/수정 시 보안 표준 검사.
- **Pod Security Standards(PSS) 3레벨**:
  - **privileged**: 제한 없음(시스템/신뢰 워크로드).
  - **baseline**: 알려진 권한 상승 차단(대부분 앱). privileged·hostPID/IPC 등 거부.
  - **restricted**: 최대 강화(비-root 강제 등 best practice).
- **적용 모드(namespace 라벨)**:
  - **enforce**(위반 pod 차단), **warn**(허용+경고), **audit**(허용+로그).
  - 라벨에 level + version 지정.

### 1-15. Resource Quotas
- **namespace 단위** 리소스 소비 제한(CPU·memory·storage·pod 수). 멀티테넌시에서
  공정 분배·starvation 방지.
- **Admission plugin**(ResourceQuota)으로 동작(기본 활성).
- **requests vs limits** 모두 쿼터 가능. 쿼터가 걸린 namespace에선 pod가
  requests/limits를 **명시해야** 생성 가능.
- **LimitRange**: pod/컨테이너 **개별** 최소·최대 제한(쿼터와 별개).

### 1-16. Helm vs Kustomize
- **Helm**: K8s **패키지 매니저**(= apt/yum 등가). **Chart**(사전 구성 리소스
  패키지)로 배포·관리·업데이트. Chart.yaml·values.yaml·templates. 명령
  install/package/uninstall/list. **템플릿 언어 사용**.
- **Kustomize**: 구성 관리 도구, **kubectl 내장(`-k`)**. **템플릿 언어 없음**,
  평문 YAML을 **base + overlays**로 커스터마이즈. `kustomization.yaml`.
  namePrefix/suffix·라벨·이미지·**strategic merge/JSON patch**. dev/staging/
  prod 오버레이.

### 1-17. Service Meshes
- 마이크로서비스 간 통신 관리. 구성:
  - **Data Plane**: 트래픽 처리 프록시. **sidecar 패턴**(서비스마다 프록시,
    예 Istio·Linkerd) 또는 **per-node 프록시**(예 traefik mesh).
  - **Control Plane**: 프록시 구성·지시 관리 허브.
- 이점: **mTLS(mutual TLS)**, access control, **observability**(트레이싱·모니터링),
  reliability(rate limiting·circuit breaking).
- 표준: **SMI(Service Mesh Interface)** — traffic management·access control·
  metrics의 공통 인터페이스(벤더 종속 회피).

## 2. 헷갈리는 것 구분 (비교표)

### API 3단계
| 단계 | 질문 | 비고 |
| --- | --- | --- |
| Authentication | 너 누구? | 인증서·토큰·OIDC·SA |
| Authorization | 뭘 해도 돼? | `--authorization-mode`(RBAC 등) |
| Admission | 이 요청 허용? | accept/reject/modify |

### RBAC 역할·바인딩
| 리소스 | 범위 |
| --- | --- |
| Role / RoleBinding | **namespace** |
| ClusterRole / ClusterRoleBinding | **클러스터 전역** |

### 스케줄 제어 3종
| 기법 | 방향 | 성격 |
| --- | --- | --- |
| nodeSelector | pod→노드 라벨 | 스케줄러 사용(가이드) |
| nodeName | pod→노드 | **스케줄러 우회** |
| Taint/Toleration | 노드가 pod 밀어냄 / pod가 허용 | 반발/허용 |
| Affinity/Anti | 라벨 기반 유인/분리 | required=필터, preferred=점수 |

### Taint effect
| effect | 신규 pod | 기존 pod |
| --- | --- | --- |
| NoSchedule | 차단 | 유지 |
| NoExecute | 차단 | **축출** |
| PreferNoSchedule | 회피(soft) | 유지 |

### Reclaim 정책
| 정책 | 동작 | 기본 |
| --- | --- | --- |
| Retain | 데이터 유지(수동 회수) | 수동 PV |
| Delete | 볼륨 삭제 | **동적** |
| Recycle | 스크럽(구식) | - |

### Deployment vs StatefulSet
| 구분 | Deployment | StatefulSet |
| --- | --- | --- |
| ID | 랜덤 pod명 | **고정(name-0..)** |
| 스토리지 | 공유 | **pod별 PVC** |
| 관리 | ReplicaSet | ReplicaSet 없음·순서 보장 |

### Ingress vs Gateway API
| 구분 | Ingress | Gateway API |
| --- | --- | --- |
| 성격 | 단일 리소스·컨트롤러 어노테이션 의존 | **역할 분리**·모듈식 |
| 리소스 | Ingress | GatewayClass/Gateway/HTTPRoute |
| 위상 | GA(폐기 아님) | **후속 표준** |

### PSS 레벨 / PSA 모드
| PSS | 강도 | PSA 모드 | 동작 |
| --- | --- | --- | --- |
| privileged | 제한 없음 | enforce | 차단 |
| baseline | 권한상승 차단 | warn | 허용+경고 |
| restricted | 최대 강화 | audit | 허용+로그 |

### Helm vs Kustomize
| 구분 | Helm | Kustomize |
| --- | --- | --- |
| 방식 | 패키지·**템플릿** | **kubectl 내장**·base+overlay |
| 언어 | 템플릿 언어 | 순수 YAML(패치) |

## 3. 함정·키워드

- API 순서 = **AuthN → AuthZ → Admission**(순서 바꾸면 오답). AuthZ는
  `--authorization-mode`.
- **nodeName = 스케줄러 우회**(nodeSelector는 우회가 아니라 가이드).
- **NoExecute = 기존 pod 축출**, NoSchedule = 기존 pod 유지.
- Affinity **required=필터(Pending 가능), preferred=점수(weight)**.
- **동적 프로비저닝 reclaim 기본 = Delete**, 수동 PV = Retain.
- **StatefulSet = pod별 PVC + 안정 network ID + headless service**.
- **NetworkPolicy 기본 open**; 지원 CNI 필요.
- **Ingress API는 폐기 아님**(ingress-nginx 컨트롤러 은퇴와 혼동 금지).
  Gateway API는 Ingress의 후속.
- **PDB = voluntary disruption** 보호(replicas와 목적 다름).
- 보안 4계층 순서: AuthN/AuthZ → Admission → Runtime → Network/Data.
- **PSS = privileged/baseline/restricted**, **PSA 모드 = enforce/warn/audit**.
- **Helm=템플릿 패키지 / Kustomize=kubectl 내장·오버레이**.
- Service Mesh 핵심 이점 = **mTLS**, 표준 = **SMI**.

## 4. 자가 점검 Q&A

1. Q: API 요청 처리 3단계 순서는?
   A: **Authentication → Authorization → Admission control**.
2. Q: Authorization 방식을 정하는 API 서버 플래그는?
   A: **`--authorization-mode`** (RBAC·Node·Webhook·ABAC).
3. Q: Role과 ClusterRole의 차이는?
   A: Role=**namespace 범위**, ClusterRole=**클러스터 전역**.
4. Q: 스케줄러를 우회해 특정 노드에 pod를 배치하는 필드는?
   A: **nodeName**.
5. Q: NoExecute와 NoSchedule의 기존 pod 처리 차이는?
   A: NoExecute=**축출**, NoSchedule=**유지**.
6. Q: required와 preferred affinity의 차이는?
   A: required=**필터(불충족 시 Pending)**, preferred=**점수(weight)**.
7. Q: 동적 프로비저닝 PV의 기본 reclaim 정책은?
   A: **Delete** (수동 PV는 Retain).
8. Q: StatefulSet이 안정적 network ID를 위해 필요한 서비스는?
   A: **headless service**.
9. Q: NetworkPolicy가 없을 때 기본 통신 정책은?
   A: **allow all**(개방).
10. Q: Ingress API는 폐기되었나?
    A: 아니오. ingress-nginx **컨트롤러**만 은퇴 예정, Ingress API는 유지.
    Gateway API가 후속.
11. Q: PDB가 보호하는 중단 유형은?
    A: **voluntary disruption**(자발적 중단).
12. Q: Pod Security Standards 3레벨은?
    A: **privileged / baseline / restricted**.
13. Q: PSA의 3가지 적용 모드는?
    A: **enforce / warn / audit**.
14. Q: Helm과 Kustomize의 근본 차이는?
    A: Helm=**템플릿 기반 패키지 매니저**, Kustomize=**kubectl 내장·오버레이
    (템플릿 없음)**.
15. Q: 서비스 메시의 대표 보안 이점과 표준 인터페이스는?
    A: **mTLS** / **SMI**.

