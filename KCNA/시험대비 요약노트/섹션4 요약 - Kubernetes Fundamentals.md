# 섹션4 요약 — Kubernetes Fundamentals

> 원본: kcna-transcripts-clean.md · 섹션4 / 시험대비 암기용
> 깊이: 상(자세히) · 최대 도메인(K8s Fundamentals 44%)

## 0. 시험 포인트 한눈에 (암기표)

| 주제 | 반드시 외울 것 |
| --- | --- |
| CRD | Custom Resource Definition = 코어 밖 기능으로 **K8s API 확장** |
| 저수준/고수준 런타임 | runc(참조구현) / containerd(엔진, CNCF graduated) |
| kubelet | **control plane·node 양쪽** 실행. pod 유지. `/etc/kubernetes/manifests` 감시 → **static pod** |
| etcd | 강한 일관성 **key-value store**, source of truth. prod **홀수(이상적 5)** |
| kube-apiserver | 클러스터 중심 게이트웨이, RESTful, 데이터는 etcd에 |
| kube-scheduler | 노드 배치 결정. 없으면 pod **Pending** |
| kube-proxy | **DaemonSet**(static pod 아님). 모든 노드/CP에서 서비스 포워딩 |
| CoreDNS | 클러스터 DNS. **Deployment**로 실행 |
| Pod | **컴퓨트 최소 배포 단위**. 1+ 컨테이너, **localhost 공유·pod당 고유 IP** |
| restartPolicy | **Always(기본)** / OnFailure / Never |
| Deployment 전략 | RollingUpdate 기본 **maxSurge 25% · maxUnavailable 25%** |
| DaemonSet | **노드당 1 pod**, 신규 노드 자동 배치 |
| Service 유형 | ClusterIP(기본)·NodePort·LoadBalancer·ExternalName **(+Headless)** |
| DNS 이름 | `service.namespace.svc.cluster.local` |
| Secret | **base64 인코딩(암호화 아님)**, etcd에 평문 저장 → 접근 제한 필수 |
| Labels vs Annotations | Labels=**선택(selector)** / Annotations=**메타데이터(선택 불가)** |
| Probe 순서 | **Startup 먼저** → 완료 후 Readiness·Liveness 동시 |

## 1. 핵심 개념 정리
### 1-1. Container Orchestration Introduction
- 오케스트레이션 = 컨테이너 운영(provisioning·deployment·scaling 등) **자동화**.
  네트워킹·스토리지·보안·오토스케일링 통합 프레임워크 제공.
- 뛰어난 영역: 프로비저닝/배포, 가용성/self-healing, 스케줄링/리소스 효율,
  서비스 노출, 인가/보안, 스토리지, 오토스케일링, 확장 기능(**CRD**).
- **CRD**(Custom Resource Definition) = 코어 밖 기능으로 K8s API 확장(예:
  MySQL CRD로 MySQL 인스턴스 관리).
- 경쟁자: OpenShift, Docker Swarm, Nomad, Kubernetes → **Kubernetes 승리**.
  OpenShift는 코어를 K8s로 피벗(+ 프레임워크).

### 1-2. Kubernetes Architecture (컴포넌트)
- **저수준 런타임**: **runc**(OCI 참조 구현). 대안 crun·kata·gVisor.
- **고수준 런타임/엔진**: **containerd**(Docker 제작→CNCF 기증·graduated).
  설치 시 runc 자동 설치.
- **kubelet**: **control plane과 node 양쪽**에서 실행. pod 유지 담당. pod
  spec(YAML/JSON) 사용. `/etc/kubernetes/manifests` 디렉토리 감시 → **static
  pod** 생성. API로도 pod 생성 지시 받음.
- **static pod**: manifests 디렉토리의 파일로 생성·관리되는 pod.
- **etcd**: 강한 일관성 분산 **key-value store**. K8s의 **source of truth /
  backing store**. prod는 **홀수 멤버(이상적 5)** + 백업 중요.
- **kube-apiserver**: 클러스터 중심·주 게이트웨이. **RESTful API**. 모든
  데이터를 **etcd**에 저장.
- **kube-scheduler**: static pod. 제약·리소스 기반 노드 배치 결정. 없으면
  pod가 **Pending**에서 멈춤.
- **kube-proxy**: **DaemonSet**(static pod 아님·일반 pod)으로 모든 CP·노드에
  실행. TCP/UDP/SCTP 포워딩 구성(Service 동작).
- **CoreDNS**: 유연한 DNS 서버. **Deployment**로 실행. 친화적 DNS 이름 제공.
- **Controller-Manager(c-m)**: control loop로 desired state 유지. Replication
  Controller·Node Controller·Deployment Controller 등.
- **Cloud-Controller-Manager(CCM)**: 클라우드 기능 브리지(route·LB). 예: AWS
  LoadBalancer 서비스 → ELB 자동 연결.
- **HA 구성**: control node 3개, **etcd 클러스터(홀수)**, **Raft consensus**로
  상태 합의. API는 load balancer 앞단.

### 1-3. Pods
- **Pod = 컴퓨트의 최소 배포 단위**. 1개 이상 컨테이너 그룹.
- 네트워킹: 컨테이너들이 **localhost로 통신**, pod마다 **고유 IP**. 컨테이너
  간 **IPC** 가능. pod IP는 클러스터 내 어느 노드에서든 접근 가능.
- **restartPolicy**: **Always(기본)** / OnFailure / Never.
- **kubectl explain**으로 spec 필드 조회.
- **Imperative vs Declarative**: create/replace/delete = **imperative**(create
  는 1회만), **apply** = **declarative**(변경 예상 시 apply 권장).
- **멀티 컨테이너 pod / sidecar**: 부가 작업 컨테이너. 특정 컨테이너 로그는
  `logs -c <name>`, 이전 로그는 `-p`/`--previous`.
- 접근: `kubectl port-forward`(로컬), 다른 pod에서 curl(pod IP·DNS).

### 1-4. Pods Troubleshooting
- **Pod phase**(공식): Pending / Running / Succeeded / Failed / Unknown.
- 상태 신호: ContainerCreating, ErrorImagePull, **ImagePullBackOff**,
  **CrashLoopBackOff**, InvalidImageName, RunContainerError,
  CreateContainerConfigError.
- 진단 법칙:
  - **Pending + 노드 미할당** → 스케줄링 문제(리소스·nodeSelector·taint·커스텀
    스케줄러).
  - **Pending + 노드 할당됨 + 컨테이너 이슈** → 이미지/컨테이너 문제.
  - **ImagePullBackOff / ErrorImagePull** → 이미지 pull 실패(레포/권한/네트워크).
  - **CrashLoopBackOff** → 컨테이너 반복 종료, backoff 지연 증가.
- 로그 규칙: 컨테이너가 **시작 못 하면 앱 로그 없음**(kubectl logs는 에러만).
  시작 후 크래시면 이전 로그는 **`logs -p`**로. **`-f`**(follow)·**`--tail=N`**·
  **`--all-containers`** 유용. 진단 도구: describe(events)·logs·exec·events.

### 1-5. Namespaces
- 클러스터 리소스를 분할. 이름은 **namespace 내에서만 유일**하면 됨(다른 ns에
  동일 이름 pod 가능). 멀티테넌시에 유용.
- namespace 단위로 **ResourceQuota·LimitRange·RBAC** 적용 가능.
- **기본 namespace 4종**:
  - `default`(기본), `kube-system`(K8s 시스템 객체), `kube-node-lease`(노드
    heartbeat lease), `kube-public`(모두 읽기 가능·비인증 포함).
- **namespaced vs cluster-level**: Pod=namespaced, **Node=cluster-level**.
  `kubectl api-resources`의 NAMESPACED 열로 확인(ns 단축어).

### 1-6. Deployments & ReplicaSets
- **Deployment** = 애플리케이션의 **선언적 업데이트**. 기능: **replication,
  rolling update, rollback**.
- Deployment는 **ReplicaSet을 관리**. 이미지 변경 시 **새 ReplicaSet** 생성,
  이전 ReplicaSet은 롤백용으로 유지. pod 이름 = ReplicaSet ID 포함.
- **RollingUpdate 기본 전략**: **maxSurge 25%**(초과 허용), **maxUnavailable
  25%**(불가 허용) → 가용성 유지.
- rollout: `rollout status`(실시간)·`rollout history`(revision)·`rollout undo`
  (--to-revision=N). change-cause는 `kubernetes.io/change-cause` 어노테이션.
- 스케일: `kubectl scale`, 이미지 변경 `set image`.

### 1-7. DaemonSets
- **모든 노드(또는 특정 부분집합)에 pod 1개 보장**. 신규 노드 추가 시 자동 배치.
- 용도: 로그 수집(fluentd/filebeat), 모니터링(node exporter), 네트워킹(CNI).
- **replicas 개념 없음**, rolling update 전략 없음. ReplicaSet 없이 이름 부여.
- 비유: "노드용 Deployment" = **1 pod per node**. 특정 노드에 안 뜨면 보통
  **taint** 때문.

### 1-8. kubectl set image & patch
- **set image**: pod/deployment의 컨테이너 이미지 갱신. 컨테이너명에 **와일드카드
  (*)** 가능. Deployment 대상이면 **rolling update** 유발.
- **patch**: 리소스 일부만 부분 갱신. 기본은 **strategic merge**(구조 인지,
  컨테이너 merge key = **name**), 그 외 **JSON patch**(op/path). replicas·
  labels·annotations·env·image 등 변경 가능.

### 1-9. Services
- Service = pod 집합을 **네트워크 서비스로 노출**. label **selector** 기반.
- **4대 유형**:
  - **ClusterIP**(기본): 클러스터 내부 IP.
  - **NodePort**: 각 노드 포트로 외부 접근(노드 IP:NodePort).
  - **LoadBalancer**: 클라우드 LB 연동.
  - **ExternalName**: 외부 DNS의 **CNAME/별칭**(프록시 아님).
- **Headless(5번째)**: `clusterIP: None`. IP·프록시 없이 **DNS로 pod에 직접**
  (round-robin). "서비스 유형 몇 개?" 질문의 정답은 **4개**지만 Headless도 기억.
- **Endpoints/EndpointSlices** = 서비스가 가리키는 **pod IP** 집합(트러블슈팅에
  유용).
- DNS: `service.namespace.svc.cluster.local`. pod의 resolv.conf search로
  이름만으로도 해석.

### 1-10. Jobs & CronJobs
- **Job** = 배치 작업 pod의 **supervisor**. 완료까지 실행·실패 시 재시도.
  - **completions**(총 완료 수), **parallelism**(동시 실행 수). 예 completions
    20·parallelism 5 = 총 20개 pod, 동시 최대 5개.
- **CronJob** = 시간 기반 스케줄러(**cron 문법**). 스케줄마다 Job 생성.
  - **successfulJobsHistoryLimit 기본 3**(완료/실패 이력 보존 수).

### 1-11. ConfigMaps
- **비밀 아닌 설정 데이터**(설정 파일·환경변수) 저장. 코드에서 설정 분리.
- 생성: `--from-literal`(key=value), `--from-env-file`. 주입: **envFrom +
  configMapRef**.
- **immutable: true**(1.21+) 설정 시 변경 불가(삭제 후 재생성만 가능).
- 클러스터의 **중앙 설정 저장소** 역할.

### 1-12. Secrets
- **민감 정보**(암호·토큰·키) 저장. 사용법은 ConfigMap과 거의 동일하나
  **secretRef** 사용, 타입 **generic**.
- **핵심: 암호화가 아니라 base64 "인코딩"**. `base64 -d`로 즉시 디코딩 가능.
  etcd에 **평문(인코딩 상태)** 저장 → **etcd 접근 제한 필수**.
- 사용 지침: secret-like는 **Secret**, 비밀 아닌 설정은 **ConfigMap**.

### 1-13. Labels & Annotations
- **Labels**: 식별·조직화용 key-value **메타데이터**. **selector로 선택 가능**.
  Service·ReplicaSet·Deployment가 selector로 대상 pod를 고름. `kubectl run`은
  `run=<이름>` 라벨 자동 부여. `-l`/`--selector`로 조회.
- **Annotations**: 서술적/지시적 **메타데이터**. **selector로 선택 불가**.
  컨트롤러(ingress·cert-manager·external-dns·service mesh)나 도구가 소비.
  관례상 `company.org/key` 형식. **pod 템플릿의 어노테이션 변경도 rollout 유발**.

### 1-14. Probes (Startup / Liveness / Readiness)
- **Startup probe**: 앱이 다 떴는가? **Startup이 진행/실패 중이면 Readiness·
  Liveness는 실행 안 됨**(느린 시작 앱 보호).
- **Liveness probe**: 컨테이너를 재시작해야 하나? 충분히 실패하면 kubelet이
  **재시작**(restartPolicy에 따름).
- **Readiness probe**: 트래픽을 받아야 하나? 실패하면 **서비스 백엔드에서 제외**
  (준비될 때까지).
- **순서: Startup 먼저 → 완료 후 Readiness·Liveness 동시 실행**.
- 방식: command, HTTP, TCP, gRPC(상호 교체 가능). `describe`는 실패만 보여줌
  (성공은 안 보임).

## 2. 헷갈리는 것 구분 (비교표)

### static pod vs 일반 pod / 컴포넌트 실행 형태
| 컴포넌트 | 실행 형태 |
| --- | --- |
| etcd·apiserver·scheduler·controller-manager | **static pod** |
| kube-proxy | **DaemonSet** (일반 pod) |
| CoreDNS | **Deployment** |
| kubelet | 프로세스(각 노드·CP) |

### 워크로드 오브젝트
| 오브젝트 | 용도 |
| --- | --- |
| Deployment | 무상태 앱, rolling update/rollback, ReplicaSet 관리 |
| ReplicaSet | replica 수 유지(보통 Deployment가 관리) |
| DaemonSet | **노드당 1 pod** |
| Job / CronJob | 완료형 배치 / 스케줄 배치 |

### Service 유형
| 유형 | 접근 범위 |
| --- | --- |
| ClusterIP(기본) | 클러스터 내부 |
| NodePort | 노드 IP:포트로 외부 |
| LoadBalancer | 클라우드 LB |
| ExternalName | 외부 DNS CNAME(프록시 없음) |
| Headless(clusterIP None) | DNS로 pod 직접(프록시 없음) |

### ConfigMap vs Secret
| 구분 | ConfigMap | Secret |
| --- | --- | --- |
| 대상 | 비밀 아닌 설정 | 민감 정보 |
| 저장 | 평문 | **base64 인코딩(암호화 아님)** |
| 주입 | configMapRef | secretRef |

### Labels vs Annotations
| 구분 | Labels | Annotations |
| --- | --- | --- |
| 목적 | 식별·**선택(selector)** | 서술 메타데이터 |
| 선택 사용 | 가능 | **불가** |
| 소비자 | Service·Deployment 등 | 컨트롤러·도구 |

### Probe 3종
| Probe | 질문 | 실패 시 |
| --- | --- | --- |
| Startup | 다 떴나? | readiness/liveness 대기 |
| Liveness | 살아있나? | 컨테이너 **재시작** |
| Readiness | 트래픽 받나? | 서비스에서 **제외** |

## 3. 함정·키워드

- **kubelet은 control plane에도 있다**(node 전용 아님). static pod를 통해 코어
  컴포넌트 실행.
- **kube-proxy = DaemonSet**(static pod 아님). CoreDNS = **Deployment**.
- Scheduler 없으면 pod는 **Pending**(에러 아님).
- **Secret = 인코딩(base64), 암호화 아님.** etcd 평문 저장 → 접근 제한.
- **Labels=선택 가능 / Annotations=선택 불가**(핵심 구분).
- restartPolicy 기본 = **Always**.
- Rolling update 기본 = **maxSurge 25% / maxUnavailable 25%**.
- DaemonSet은 **replicas 없음**, 노드당 1 pod. 특정 노드 누락 = **taint** 의심.
- 서비스 유형 "4개"(ClusterIP·NodePort·LoadBalancer·ExternalName), Headless는
  ClusterIP 변형.
- **Startup probe 실패/진행 중엔 liveness·readiness 미실행**.
- CronJob 이력 기본 보존 = **3**(successfulJobsHistoryLimit).
- ImagePullBackOff=이미지 pull 실패, CrashLoopBackOff=반복 종료.

## 4. 자가 점검 Q&A

1. Q: kubelet은 어디에서 실행되나?
   A: **control plane과 node 양쪽**.
2. Q: kube-proxy와 CoreDNS의 실행 형태는?
   A: kube-proxy=**DaemonSet**, CoreDNS=**Deployment**.
3. Q: 스케줄러가 없으면 pod는 어떤 상태가 되나?
   A: **Pending**(배치 불가).
4. Q: Secret은 암호화되나?
   A: 아니오. **base64 인코딩**이며 etcd에 평문 저장 → 접근 제한 필요.
5. Q: Labels와 Annotations의 핵심 차이는?
   A: Labels=**selector로 선택 가능**, Annotations=선택 불가(메타데이터).
6. Q: Deployment의 기본 롤링 업데이트 파라미터 두 값은?
   A: **maxSurge 25% / maxUnavailable 25%**.
7. Q: 노드마다 pod 하나를 보장하는 오브젝트는?
   A: **DaemonSet**.
8. Q: 서비스 4대 유형은?
   A: **ClusterIP·NodePort·LoadBalancer·ExternalName** (+Headless).
9. Q: 서비스의 FQDN 형식은?
   A: `service.namespace.svc.cluster.local`.
10. Q: Startup probe가 진행/실패 중일 때 다른 프로브는?
    A: **Readiness·Liveness는 실행되지 않음**.
11. Q: Liveness 실패 시와 Readiness 실패 시 각각 무슨 일이?
    A: Liveness=**컨테이너 재시작**, Readiness=**서비스 백엔드에서 제외**.
12. Q: 기본 namespace 4종은?
    A: default·kube-system·kube-node-lease·kube-public.
13. Q: Node는 namespaced 리소스인가?
    A: 아니오. **cluster-level**.
14. Q: CronJob의 기본 이력 보존 개수는?
    A: **3**.

