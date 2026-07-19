# 🗓️ KCNA 2주 맞춤 학습 계획 (진단 기반 재구성)

> 대상: CKA/CKAD 보유자 · K8s 코어 강점 / 관측성·CNCF 생태계 세부가 약점
> 시험: 약 2주 후 (D14 응시 목표)
> 가용 시간: 평일 1h × 10일 + 주말 3h × 4일 = **총 약 22h**
> 원칙: 아는 K8s 코어는 퀴즈로 자가검증만, 약점(관측성·프로젝트 매칭)에 시간 재분배. 인풋보다 아웃풋(문제풀이).

---

## 🎯 진단 요약 (2단계 결과)

- 진단 퀴즈: 8/12 → 정정 후 12/12
- 강점: K8s 코어, GitOps/Argo·Flux, Helm, CNCF 거버넌스·성숙도 순서
- 약점(집중 대상): 관측성 3대 축·트레이싱 도구 매칭, 개별 프로젝트 성숙도, 오토스케일러(KEDA 등) 구분
- 오답 추적: `오답노트.md` (Q2/Q6/Q7/Q9 → D+3 변형 재출제 예정)

---

## 참고 자료

- 메인 강의: James Spurin — Kubernetes Certified (KCNA), Udemy
- 랩 소스: https://github.com/spurin/diveintokcna
- 스터디 그룹: https://github.com/spurin/KCNA-Study-Group/discussions/1
- CNCF Glossary: https://glossary.cncf.io · Landscape: https://landscape.cncf.io
- 공식: https://training.linuxfoundation.org/certification/kubernetes-and-cloud-native-associate/

---

# 📅 Week 1 — 인풋 + 약점 집중

## D1 (평, 1h) — 섹션2 Cloud Native Architecture
- [X] 오토스케일링 4종 구분: HPA(Pod 수평) / VPA(리소스 조정) / Cluster Autoscaler(노드 수) / KEDA(이벤트 기반)
- [X] 클라우드 네이티브 원칙(복원력·확장성·자동화), 서버리스 개념
- [X] 섹션 퀴즈

## D2 (평, 1h) — 섹션3 Containers (배속)
- [ ] OCI / 컨테이너 런타임 / 이미지 계층 개념
- [ ] 아는 부분 스킵, 낯선 용어만 메모

## D3 (평, 1h) — 섹션4 Container Orchestration (개념 위주)
- [ ] 네트워킹/스토리지/서비스메시(Envoy/Istio) 개념만 확실히
- [ ] 섹션 퀴즈

## D4 (평, 1h) — 섹션5 K8s Fundamentals (자가검증)
- [ ] 강점 영역 → 강의 스킵, 섹션 퀴즈로 정답률만 확인
- [ ] 틀린 문항만 오답노트에 추가

## D5 (평, 1h) — ★ 섹션6 관측성 ①
- [ ] Prometheus(Pull 방식), Grafana(시각화), 3대 축(Metrics/Logs/Traces)
- [ ] 섹션 퀴즈

## D6 (주, 3h) — ★ 관측성 ② + 섹션7 App Delivery
- [ ] 로깅(Fluentd), 트레이싱(Jaeger/OpenTelemetry) 역할 매칭 암기
- [ ] GitOps(Argo CD/Flux), CI/CD, Helm 패키징
- [ ] 관측성 랩 실행

## D7 (주, 3h) — CNCF 프로젝트 암기 + 🧪 1차 모의고사
- [ ] 치트시트 이름↔역할 매칭 테스트
- [ ] 프로젝트 성숙도(Sandbox/Incubating/Graduated) 분류 암기
- [ ] 🧪 1차 모의고사 → 점수: ____%
- [ ] 약한 도메인 표시: ____________

---

# 📅 Week 2 — 아웃풋 + 마무리

## D8 (평, 1h) — 1차 오답 분석 + 오답노트 재출제
- [ ] 1차 모의고사 오답 해설 정독
- [ ] 오답노트 D+3 재출제 (Q2/Q6/Q7/Q9 변형)

## D9 (평, 1h) — 관측성·생태계 약점 재복습
- [ ] 정답률 낮은 섹션 재시청
- [ ] 반복 실수 패턴 메모

## D10 (평, 1h) — CNCF 프로젝트 플래시카드
- [ ] 이름↔역할 매칭 속도 테스트
- [ ] 헷갈리는 것만 카드화

## D11 (평, 1h) — 미니 퀴즈 (출제) + 오답 누적
- [ ] 오케스트레이터 출제 미니 퀴즈
- [ ] 틀린 것 오답노트 누적

## D12 (평, 1h) — 전 도메인 스피드 퀴즈
- [ ] 도메인별 빠른 자가검증
- [ ] 자신감 3점 미만 도메인 표시

## D13 (주, 3h) — 🧪 2차 모의고사 + 요약
- [ ] 🧪 2차 모의고사 90분 실전 → 점수: ____%
- [ ] 문항당 1.5분 페이스 점검
- [ ] "헷갈리는 개념 1페이지" 요약 작성

## D14 (주, 3h) — 최종 훑기 + 응시 🎯
- [ ] 요약 노트 + 치트시트 1회독 (새 개념 금지)
- [ ] 시험 환경 점검(신분증·웹캠·프록터링·조용한 공간)
- [ ] KCNA 응시 → 결과: ____________

---

## 🏁 시험 직전 체크리스트
- [ ] 관측성 3대 축·도구 매칭 완벽 암기
- [ ] CNCF 주요 프로젝트 역할 + 성숙도 단계 암기
- [ ] 모의고사 안정적으로 85%+ 달성
- [ ] 오답노트 전 항목 🟢 상태
- [ ] 프록터링 환경 준비 완료

## 📈 도메인별 자신감 & 모의고사 트래커

| 도메인 | 자신감(1-5) | 비고 |
| --- | --- | --- |
| Kubernetes Fundamentals (44%) |  | 강점 |
| Container Orchestration (28%) |  |  |
| Cloud Native App Delivery (16%) |  |  |
| Cloud Native Architecture (12%, 관측성 포함) |  | 집중 |

| 모의고사 | 날짜 | 점수 |
| --- | --- | --- |
| 1차 (D7) |  | % |
| 2차 (D13) |  | % |
