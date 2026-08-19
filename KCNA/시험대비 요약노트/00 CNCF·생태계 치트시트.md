# 00 CNCF·생태계 치트시트 (전 섹션 공용)

> KCNA 시험 직전 속독용. 프로젝트·표준·구분을 한 장에 집약.

## 1. 반드시 외우는 숫자·사실

| 항목 | 값 |
| --- | --- |
| Linux Foundation 설립 | **2000** (OSDL + Free Standards Group) |
| CNCF 설립 / K8s 1.0 | **2015** / 2015.2 |
| CNCF 첫 기증 프로젝트 | **Kubernetes**(Google) |
| CNCF 졸업 1·2번째 | **Kubernetes → Prometheus** |
| cgroups | Google **2006** → 커널 병합 2008 |
| Docker | 2010 dotCloud → **2013 Docker** |
| API 서버 포트 | **6443** / Prometheus UI **9090** |
| Rolling update 기본 | maxSurge **25%** · maxUnavailable **25%** |
| CronJob 이력 보존 | **3** |
| 시험(신버전) | 60문항 / 90분 / **75%** / 무료 재응시 1회 |
| 도메인 비중 | K8s Fund **44** · Orchestration **28** · App Delivery **16** · CN Arch **12** |

## 2. CNCF 프로젝트 ↔ 역할 매핑

| 프로젝트 | 역할 |
| --- | --- |
| **Kubernetes** | 컨테이너 오케스트레이션 |
| **containerd** | 고수준 컨테이너 런타임/엔진 (graduated) |
| **etcd** | 분산 key-value store(K8s source of truth) |
| **CoreDNS** | 클러스터 DNS |
| **Prometheus** | 모니터링·알림(Pull·PromQL) |
| **Rook** | 스토리지 오케스트레이션(CSI, graduated) |
| **Helm** | 패키지 매니저(차트) |
| **Argo CD / Flux** | GitOps 지속 배포 |
| **Envoy** | 서비스 프록시(메시 데이터 플레인) |
| **Linkerd / Istio** | 서비스 메시 |
| **Knative / OpenFaaS** | 서버리스(FaaS) on K8s |
| **KEDA** | 이벤트 기반 오토스케일(scale to zero) |
| **CloudEvents** | 이벤트 데이터 공통 포맷 |
| **Jaeger / OpenTelemetry** | 분산 트레이싱/텔레메트리 |

> **참고**: Grafana·Terraform·Ansible은 유명하지만 **CNCF 그래듀에이트 목록과는 별개**입니다.

## 3. 성숙도 3단계 & 거버넌스

- 성숙도: **Sandbox → Incubating → Graduated** (crossing the chasm).
- 채택자: innovators → early adopters → 〔CHASM〕 → early majority → late
  majority → laggards.
- 졸업 요건(→ **TOC**에 증명): adoption, 건강한 변경률, 여러 조직 외부 committer,
  Code of Conduct, Core Infrastructure Initiative Best Practices Badge.
- 거버넌스 조직: **TOC**(성숙도 결정) · **SIG**(관심그룹) · **TAG**(CNCF가
  SIG 개명).
- 갈등 해결: **Discussion** + **Voting**(binding / non-binding).

## 4. 오픈표준 (OCI / 인터페이스)

| 표준 | 대상 | 메모 |
| --- | --- | --- |
| OCI **Image** | 이미지 패키징 | Buildkit·Podman·Buildah |
| OCI **Runtime** | 실행 | **runc = 참조 구현** |
| OCI **Distribution** | 배포 | Docker Registry API v2 기반 |
| **CNI** | 네트워킹 | 설치돼야 노드 Ready. ADD/DEL/CHECK/VERSION |
| **CSI** | 스토리지 | Rook·Portworx |
| **CRI** | kubelet↔런타임 | containerd·cri-o·kata 교체 |
| **SMI** | 서비스 메시 | 벤더 종속 회피 |

- OCI 출범: **2015년 Docker Inc + CoreOS 등**, Linux Foundation 산하.
- 런타임 계층: **runc(low-level) / containerd(high-level engine)**.

## 5. 오토스케일러 구분

| 도구 | 무엇을 스케일 | 메모 |
| --- | --- | --- |
| **HPA** | pod **replica 수** | Horizontal |
| **VPA** | pod **requests/limits** | Vertical |
| **Cluster Autoscaler** | **노드 수** | 클러스터 크기 |
| **KEDA** | 이벤트 기반 | **scaled objects · scale to zero** |

- 방식: **Reactive / Scheduled / Predictive(AI·ML)**.
- 방향: **Vertical(scale up) / Horizontal(scale out)** — CN은 Horizontal 선호.

## 6. 관측성 요약

- 3대 축: **Logs · Metrics · Traces** (+ Alerts).
- 메트릭: **Gauge**(오르내림) · **Counter**(증가만) · **Meter**(발생률) ·
  **Histogram**(분포·bucket).
- **Prometheus**(수집·저장·Pull·PromQL) + **Grafana**(시각화).
- Node Exporter = **DaemonSet**(HW/OS 메트릭), Kube-State-Metrics = 오브젝트 상태.

## 7. 자주 나오는 "구분" 총정리

| 헷갈리는 쌍 | 핵심 구분 |
| --- | --- |
| Continuous Delivery vs Deployment | Delivery=사람이 릴리스 / Deployment=전자동 (KCNA=Delivery) |
| Vertical vs Horizontal | up(증설) vs out(추가) — CN은 out 선호 |
| HPA vs VPA | replica 수 vs requests/limits |
| Sandbox/Incubating/Graduated | 성숙도 저→고 순서 |
| TOC vs SIG vs TAG | 성숙도 결정 vs 관심그룹 vs CNCF판 SIG |
| OCI Image/Runtime/Distribution | 패키징 / 실행(runc) / 배포 |
| low-level vs high-level runtime | runc vs containerd |
| Deployment vs StatefulSet vs DaemonSet | 무상태 vs 상태(고정ID·PVC별) vs 노드당1 |
| ClusterIP/NodePort/LoadBalancer/ExternalName | 내부 / 노드포트 / 클라우드LB / CNAME |
| ConfigMap vs Secret | 비밀아님 vs 민감(둘 다 base64·암호화 아님) |
| Labels vs Annotations | 선택 가능 vs 선택 불가(메타) |
| Startup/Liveness/Readiness | 시작완료 / 재시작 / 트래픽수신 |
| Role vs ClusterRole | namespace vs 전역 |
| NoSchedule vs NoExecute | 기존 유지 vs 기존 축출 |
| required vs preferred affinity | 필터(Pending) vs 점수(weight) |
| Retain vs Delete vs Recycle | 유지 / 삭제(동적기본) / 스크럽 |
| Ingress vs Gateway API | 단일리소스 vs 역할분리 후속표준 |
| PSS privileged/baseline/restricted | 무제한 / 기본차단 / 최대강화 |
| PSA enforce/warn/audit | 차단 / 경고 / 로그 |
| Helm vs Kustomize | 템플릿 패키지 vs kubectl 내장 오버레이 |
| Gauge/Counter/Meter/Histogram | 오르내림 / 증가만 / 발생률 / 분포 |
| On-demand/Reserved/Spot | 유연·비쌈 / 약정·절감 / 저렴·보장없음 |

