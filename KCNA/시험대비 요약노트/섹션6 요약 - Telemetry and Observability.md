# 섹션6 요약 — Telemetry and Observability

> 원본: kcna-transcripts-clean.md · 섹션6 / 시험대비 암기용
> 깊이: 상(집중, 약점 영역) · 메트릭 타입·Prometheus/Grafana 자주 출제

## 0. 시험 포인트 한눈에 (암기표)

| 주제 | 반드시 외울 것 |
| --- | --- |
| Observability 정의 | 출력(**telemetry**)을 살펴 시스템 상태를 정확히 측정하는 능력 |
| 관측성 3대 축 | **Logs · Metrics · Traces** (+ Alerts) |
| Logs | 프로그램이 내는 메시지(verbosity/log level), 파일·syslog |
| Metrics | **시간 기반** 측정값(일정 간격) |
| **Gauge** | 현재 수준, **오르내림 가능**(예: 메모리·연료계) |
| **Counter** | **누적, 증가만**(예: 방문수·API 요청수) |
| **Meter** | 이벤트 **발생률**(예: 심박수·기간당 요청수) |
| **Histogram** | 통계적 분포, **bucket/bin**(예: 연령대별 방문자) |
| Traces | 요청이 시스템을 거치는 경로 추적(분산 시스템에 유용) |
| Prometheus | CNCF **graduated** · **K8s 다음 2번째 졸업** · SoundCloud 제작 |
| Prometheus 수집 | **Pull** 중심(push도 가능) · **PromQL** · 시계열 다차원 |
| Prometheus 포트 | **9090** |
| Grafana | **시각화/대시보드** · 다중 데이터소스(Prometheus·InfluxDB·ES) |
| Node Exporter | 하드웨어/OS 메트릭, **DaemonSet** |
| Kube-State-Metrics | K8s 오브젝트 상태를 API에서 수집 |
| 비용 모델 | On-demand · Reserved · **Spot(저렴·보장 없음)** |
| Kubecost | K8s 비용 관리 도구(상용+오픈소스) |

## 1. 핵심 개념 정리
### 1-1. Observability & Telemetry (3대 축)
- **Observability** = 수집된 **출력(output)**을 살펴 시스템 상태를 정확히 측정
  하는 능력. 출력은 **telemetry** 형태로 제공.
- **Telemetry = Logs · Metrics · Traces**(관점에 따라 Alerts 포함).
- **3대 축(pillars)**:
  - **Logs**: 프로그램·앱·프로세스가 내는 메시지. verbosity/log level(정보~
    디버그). 파일 기반 또는 **syslog** 같은 로깅 엔티티 경유.
  - **Metrics**: **일정 간격의 시간 기반 측정값**. 성능 정상/과소/과다 판단.
  - **Traces**: 요청이 시스템을 **거쳐가는 과정 추적**. 분산 시스템에서 언제
    요청됐는지·컴포넌트 간 이동·각 구간 소요 시간 등 통찰 제공.
- **Alerts**: 조치가 필요한 영역 통지(예: collector 미동작). 실패를 **사전에**
  알아야 하므로 중요.

### 1-2. 메트릭 타입 (Gauge / Counter / Meter / Histogram)
- **Gauge(게이지)**: 현재 수준/내용을 표시. **오르내릴 수 있음**. 예: 연료
  게이지, 메모리 사용량, 실행 중 컴포넌트 수.
- **Counter(카운터)**: **누적**, **오직 증가**(감소 없음). 예: 웹사이트 방문수,
  API 요청수.
- **Meter(미터)**: 이벤트가 **발생하는 비율(rate)**. 예: 심박수(기간 측정 후
  평균율), 기간당 요청수.
- **Histogram(히스토그램)**: **통계적 분포**. 관측 수를 관측값 합에 대해 추적.
  값들이 **bucket/bin**에 몇 개 들어가는지 셈. 예: 연령대(0-10·11-20...)별
  방문자 수.

### 1-3. Prometheus
- 오픈소스 **CNCF graduated** 프로젝트. **Kubernetes 다음 2번째 졸업**.
  **SoundCloud**가 제작·CNCF 기증 → 현재 독립 프로젝트.
- 모니터링 + **alerting** 툴킷.
- **다차원 데이터 모델 + 시계열(time series)**: metric 이름 + key-value 쌍으로
  식별.
- **PromQL**: 유연한 쿼리 언어.
- **분산 스토리지 의존 없음** — 단일 서버 노드가 자율적.
- 수집: **push/pull 방식**(주로 **Pull**).
- 웹 UI 포트 **9090**.

### 1-4. Grafana
- 오픈소스 **모니터링·시각화·분석** 플랫폼. Prometheus와 결합 시 진가 발휘.
- **시각화**: 그래프·차트·알림으로 복잡 데이터를 한눈에.
- **데이터 소스 통합**: Prometheus, InfluxDB, Elasticsearch 등 다수.
- **관측성/모니터링**: CPU·memory·network IO 등 실시간 성능.
- **Alerting**: email·Slack·PagerDuty 등 채널.
- **커스터마이즈/확장**: 커스텀 대시보드 + 플러그인.
- 역할 연계: **SRE**의 SLA/SLO/SLI 관리에 필수.
- 기억법: **Prometheus는 두뇌(수집·저장), Grafana는 미(美)(시각화)**.

### 1-5. kube-prometheus-stack 구성요소
- Helm으로 설치하는 통합 관측성 스택. 주요 구성:
  - **Prometheus Operator**: Prometheus 서버 셋업.
  - **Prometheus 서버**: 메인 웹 UI(**9090**).
  - **Alert Manager**: 알림 처리. **StatefulSet**으로 실행.
  - **Node Exporter**: **하드웨어/OS 메트릭**(커널 노출). **DaemonSet**(모든
    노드에 롤아웃).
  - **Kube-State-Metrics**: **K8s API로 오브젝트 상태**(deployment·node·pod
    건강) 수집.
  - **Prometheus Adapter**: K8s 정보를 **Prometheus 메트릭으로 변환**(번역기).
  - **Grafana**: 시각화(포트 80).

### 1-6. Cost Management
- CN 설계는 필요에 따라 리소스 사용 → 비용 최적화(예: on-demand로 최소화).
  단, 항상 "최저가"가 목적은 아님 — **미사용 리소스** 점검도 중요.
- CN 이점: public/hybrid/private 여러 클라우드에서 실행 가능 → **lock-in 회피**.
  규제·비용 이유로 hybrid/private 선택할 수도.
- **VM 요금 모델(AWS EC2 예)**:
  - **On-demand**: 즉시 생성·폐기, 유연하나 **비쌈**.
  - **Reserved**: 기간 약정·선불 → **장기 비용 절감**.
  - **Spot**: 낮은 가격에 **입찰**, **가용성 보장 없음**(종료될 수 있음).
    리소스 손실에 적응 가능한 CN 앱에 적합.
- **Right sizing**: 클라우드 전환 시 **lift-and-shift 지양**(기존과 동일 스펙
  그대로 이전 X). **autoscaling** 활용이 더 효율적.
- **Cloud Anomaly Detection**: 비용 + 보안 크로스오버(이상 ingress/egress·CPU).
- **Kubecost**: K8s 비용 관리 도구(상용 + 오픈소스).

## 2. 헷갈리는 것 구분 (비교표)

### 관측성 3대 축
| 축 | 무엇 | 예 |
| --- | --- | --- |
| Logs | 이벤트 메시지 | 앱 로그·syslog |
| Metrics | 시간 기반 측정값 | CPU·요청수 |
| Traces | 요청 경로 추적 | 분산 호출 흐름 |
| (Alerts) | 조치 통지 | collector 실패 |

### 메트릭 타입 (가장 자주 출제)
| 타입 | 특징 | 예 |
| --- | --- | --- |
| **Gauge** | 현재값·오르내림 | 메모리·연료계 |
| **Counter** | 누적·증가만 | 방문수·API 요청 |
| **Meter** | 발생률(rate) | 심박수·기간당 요청 |
| **Histogram** | 분포·bucket | 연령대별 방문자 |

### Prometheus vs Grafana
| 구분 | Prometheus | Grafana |
| --- | --- | --- |
| 역할 | **수집·저장·쿼리(PromQL)** | **시각화·대시보드·알림** |
| 방식 | 시계열, **Pull** 중심 | 다중 데이터소스 |
| 비유 | 두뇌 | 미(美) |

### VM 비용 모델
| 모델 | 특징 |
| --- | --- |
| On-demand | 즉시·유연·비쌈 |
| Reserved | 약정·선불·절감 |
| Spot | 입찰·저렴·**가용성 보장 없음** |

## 3. 함정·키워드

- 관측성 3대 축 = **Logs · Metrics · Traces**(Alerts는 부가). 4개로 착각 주의.
- 메트릭 타입 구분(핵심 출제): **Counter=증가만**, **Gauge=오르내림**,
  **Meter=발생률**, **Histogram=분포(bucket)**.
- **Prometheus = 수집·저장(Pull·PromQL·시계열)**, **Grafana = 시각화**. 역할 혼동
  금지.
- Prometheus는 **CNCF에서 K8s 다음 2번째 졸업** 프로젝트(SoundCloud 제작).
- **Node Exporter = DaemonSet**(하드웨어/OS 메트릭), Kube-State-Metrics =
  오브젝트 상태.
- **Spot 인스턴스 = 저렴하지만 가용성 보장 없음**(종료 가능).
- 비용 최적화 = **right sizing + autoscaling**, lift-and-shift 지양.
- K8s 비용 도구 = **Kubecost**.
- Traces는 특히 **분산 시스템**에서 요청 흐름·구간 지연 파악에 유용.

## 4. 자가 점검 Q&A

1. Q: 관측성의 3대 축은?
   A: **Logs · Metrics · Traces** (+ Alerts).
2. Q: 오직 증가만 하는 메트릭 타입은?
   A: **Counter** (예: API 요청수).
3. Q: 오르내릴 수 있는 현재값 메트릭은?
   A: **Gauge** (예: 메모리 사용량).
4. Q: 값의 분포를 bucket으로 추적하는 메트릭은?
   A: **Histogram**.
5. Q: 이벤트 발생 비율을 나타내는 메트릭은?
   A: **Meter**.
6. Q: Prometheus와 Grafana의 역할 구분은?
   A: Prometheus=**수집·저장·쿼리(PromQL·Pull)**, Grafana=**시각화**.
7. Q: Prometheus는 CNCF에서 몇 번째로 졸업한 프로젝트인가?
   A: **Kubernetes 다음 2번째**.
8. Q: 하드웨어/OS 메트릭을 모든 노드에서 수집하는 컴포넌트와 실행 형태는?
   A: **Node Exporter**, **DaemonSet**.
9. Q: 저렴하지만 가용성이 보장되지 않는 EC2 요금 모델은?
   A: **Spot 인스턴스**.
10. Q: 클라우드 전환 시 지양해야 할 접근과 권장 접근은?
    A: 지양=**lift-and-shift**, 권장=**right sizing + autoscaling**.
11. Q: Kubernetes 비용 관리에 쓰는 대표 도구는?
    A: **Kubecost**.
12. Q: Prometheus 웹 UI 기본 포트는?
    A: **9090**.
