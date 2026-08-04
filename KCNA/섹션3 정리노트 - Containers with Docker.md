# 📘 섹션3 정리노트 — Containers with Docker

> 출처: James Spurin — Kubernetes Certified (KCNA), Section 3
> 시험 비중: Kubernetes Fundamentals 44% 중 Containerization → 컨테이너 기술의 기반 개념
> 아래는 강의 영상 핵심 요약. 헷갈리는 개념은 오답노트로 연계.

---

## 0. 강의 흐름 한눈에 보기 (서술형)

컨테이너는 인프라와 애플리케이션 설계, 나아가 일하는 방식까지 바꿔놓은 기술입니다. 특히 쿠버네티스처럼 수천 개의 컨테이너를 대규모로 운용하는 환경에서 그 영향이 두드러집니다. 그런데 컨테이너는 어느 날 갑자기 발명된 신기술이 아닙니다. 오래전부터 존재해온 격리 기술들이 하나씩 쌓여 만들어진 결과물에 가깝습니다. 그 역사를 따라가 보면 도커와 쿠버네티스가 무엇 위에 서 있는지 자연스럽게 이해할 수 있습니다.

**메인프레임 시대의 가상화 (1960년대).** 이야기는 1960년대 IBM 메인프레임에서 시작합니다. 당시 CP/CMS 운영체제는 여러 사용자가 하나의 메인프레임을 나눠 쓰게 해주었는데, 단순히 시간을 쪼개 쓰는 시분할(timesharing)을 넘어 사용자마다 독립된 가상 운영체제 인스턴스를 제공했다는 점이 특별했습니다. 초기 가상 머신 아키텍처의 대표적인 사례로 꼽히는 이유입니다.

**chroot — 파일시스템 격리의 출발점 (1979).** 1979년 Unix 버전 7에 등장한 chroot는 이름 그대로 "change root", 즉 프로세스와 그 자식 프로세스가 바라보는 루트 디렉토리를 바꿔주는 기능입니다. 이렇게 하면 프로세스는 지정된 디렉토리 안의 파일만 볼 수 있고 바깥에는 접근하지 못합니다. 다만 사용자 계정, 호스트 이름, IP 주소 같은 자원은 여전히 시스템과 공유했기 때문에 완전한 격리는 아니었습니다. 그럼에도 유용해서 오늘날의 여러 리눅스 배포판에서도 쓰이고 있습니다.

**FreeBSD Jails와 동시대의 시도들 (2000년 전후).** BSD 계열 오픈소스 운영체제인 FreeBSD는 2000년에 'Jails'를 선보였습니다. Jail은 자체 네트워크 인터페이스와 프로세스 ID를 가진 격리 환경을 만들어주는 경량 가상화 기술로, 오늘날 도커가 하는 일과 상당히 닮은 "조상격" 기술입니다. 다만 사용법이 복잡해 대중화되지는 못했습니다. 비슷한 시기에 Sun Solaris의 'Zones', HP-UX의 'Virtual Partitions(VPARs)'도 같은 목표를 좇았는데, 이는 자원을 격리하려는 시장의 요구가 그만큼 컸다는 방증입니다.

**컨테이너의 두 핵심 재료.** 리눅스가 성장하던 시기, 컨테이너의 토대가 되는 두 커널 기능이 자리 잡습니다. 하나는 프로세스가 볼 수 있는 자원의 범위를 나누는 **네임스페이스(namespaces)** 이고(2002년 Mount 네임스페이스를 시작으로 순차 도입, User는 2013년 완성), 다른 하나는 자원을 얼마나 쓸지 통제하는 **cgroups** 입니다(구글이 2006년 'Process Containers'로 시작 → 2008년 커널 병합). 네임스페이스는 무엇을 "볼 수 있는가"(가시성)를, cgroups는 무엇을 "얼마나 쓸 수 있는가"(자원)를 격리합니다. 이 둘이 합쳐질 때 비로소 온전한 자원 격리가 가능해집니다.

**가상 머신의 시대와 한계.** 이 재료들이 무르익기를 기다리는 동안, VMware ESXi 같은 하이퍼바이저를 앞세운 가상 머신(VM)의 시대가 먼저 찾아왔습니다. 하나의 하드웨어 위에서 여러 VM을 돌리고 VM마다 독립된 자원과 게스트 OS를 두는 방식인데, 자원 파편화·하이퍼바이저 의존성·게스트 OS 오버헤드·라이선스 관리 부담이라는 단점이 따라옵니다. 그럼에도 VM은 커널 수준의 완전 격리, 라이브 마이그레이션, GPU 패스스루, VDI 같은 강점 덕분에 지금도 건재합니다(예: Amazon EC2).

**도커의 등장과 대중화.** 도커는 이미 존재하던 네임스페이스와 cgroups를 가져와 컨테이너를 본격적으로 대중화했습니다. 2010년 'dotCloud'라는 클라우드 서비스로 출발해 2013년 도커로 전환됐습니다. VM과 달리 컨테이너는 호스트 커널을 공유하기 때문에 컨테이너마다 별도의 OS 레이어가 필요 없고, 그만큼 가볍고 빠릅니다. 도커가 앞서나간 결정적 이유는 새로운 기술을 발명해서가 아니라, Dockerfile·이미지 배포·`docker run` 한 줄로 요약되는 개발자 친화적 사용성 덕분이었습니다. 채택을 막던 진짜 장벽은 기술이 아니라 사용성이었고, 도커는 그 지점을 풀어냈습니다.

---

## 1. Introduction to Containers (컨테이너 입문)

### 한 줄 요약
컨테이너는 새로 발명된 기술이 아니라, **Linux 커널이 이미 갖고 있던 격리 기능(namespaces + cgroups)** 을 도커가 "쓰기 쉽게" 포장해 대중화시킨 결과물이다.

---

### 1-1. 가상화·격리 기술 연표 (시험에 순서/연도로 나올 수 있음)

| 연도 | 기술 | 핵심 의미 |
| --- | --- | --- |
| 1960년대 | IBM 메인프레임 **CP/CMS** | 최초의 실용적 VM 아키텍처. 단순 시분할(timesharing)을 넘어 **사용자별 가상 OS 인스턴스** 제공 |
| 1979 | **chroot** (Unix V7) | 프로세스+자식의 **루트 디렉토리 변경** = 파일시스템 격리. 단, 사용자/호스트명/IP 등은 여전히 공유 |
| 2000 | **FreeBSD Jails** | 경량 가상화. 자체 네트워크 인터페이스·PID 보유 → 도커의 조상급. 사용성이 나빠 대중화 실패 |
| 2000년대 초 | Solaris **Zones**, HP-UX **VPARs** | 같은 시기 다른 UNIX들도 격리를 시도 → 시장 수요가 컸다는 증거 |
| 2002~ | Linux **namespaces** | 컨테이너의 1차 재료 (아래 상세) |
| 2006 | Google **Process Containers** | 이후 **cgroups**로 개명 |
| 2008 | **cgroups** 커널 병합 | 컨테이너의 2차 재료 (자원 제어) |
| 2010 → 2013 | **dotCloud → Docker** | 재료를 UX로 포장 → 컨테이너 대중화 |

> 🎯 시험 포인트: "루트 파일시스템 격리의 시작" = **chroot**, "경량 가상화의 초기 형태" = **FreeBSD Jails**, "자원 제한" = **cgroups**, "가시성 격리" = **namespaces**.

---

### 1-2. 핵심 재료 ①: Linux Namespaces (무엇을 "보이지 않게" 하는가)

네임스페이스 = **프로세스가 볼 수 있는 시스템 자원의 범위를 분리**하는 커널 기능.

| 네임스페이스 | 격리 대상 | 컨테이너에서의 체감 |
| --- | --- | --- |
| **PID** | 프로세스 ID | 컨테이너 안에서 내 앱이 PID 1 |
| **Network (net)** | 네트워크 스택 | 컨테이너별 독립 IP·인터페이스·라우팅·포트 |
| **Mount (mnt)** | 마운트 포인트 | 컨테이너별 파일시스템 뷰 |
| **UTS** | 호스트명 / 도메인명 | 컨테이너마다 다른 hostname |
| **IPC** | 프로세스 간 통신 | POSIX 메시지큐·공유메모리 분리 |
| **User** | UID / GID 매핑 | 호스트의 일반 유저가 컨테이너 안에선 root |
| **cgroup** (별도) | cgroup 루트 뷰 | 자기 cgroup 계층만 보임 |

암기용 두문자: **P·N·M·U·I·U (+cgroup)**

> 도입 시기: 네임스페이스는 한 번에 추가된 게 아니라 순차 완성됐다. Mount(2002, 커널 2.4.19)를 시작으로 UTS·IPC(2006), PID·Network(2007~2008), 마지막으로 **User(2013, 커널 3.8)** 까지. 시험에는 "**2002년 = 네임스페이스의 시작**" 정도로 기억하면 충분하다.

---

### 1-3. 핵심 재료 ②: cgroups (무엇을 "얼마나" 쓰게 할 것인가)

Control Groups — Google이 2006년 시작, 2008년 커널 병합.

| 기능 | 설명 |
| --- | --- |
| **Resource limits** | CPU·메모리·네트워크·I/O 사용량 상한 |
| **Prioritization** | 그룹 간 자원 할당 우선순위 |
| **Accounting / Reporting** | 사용량 측정·보고 |
| **Control** | 그룹 내 프로세스 시작·중지·**freeze**·재시작 |

> 🎯 K8s 연결: Pod의 `resources.requests` / `resources.limits`가 최종적으로 **cgroups**로 구현된다. `limits` 초과 시 OOMKilled가 나는 이유가 여기 있음.

**namespaces = 시야(가시성) 격리 / cgroups = 자원(사용량) 격리** ← 이 대비가 시험 단골.

> cgroups와 namespaces는 목적이 다른 **별개의 커널 기능**이다. cgroups를 "7번째 네임스페이스"처럼 묶어 외우지 말 것. (참고: `cgroup namespace`라는 네임스페이스가 따로 있긴 하지만 — Linux 4.6, 2016 — 이건 cgroups 자체와는 다른 것이다.) 시험에서 cgroups의 역할을 물으면 **자원 제한 / 우선순위 / 모니터링·보고 / 제어**로 답한다.

---

### 1-4. VM vs 컨테이너

```
[ VM 방식 ]                        [ 컨테이너 방식 ]
App  App  App                      App   App   App
Guest OS x3   ← 오버헤드            (Guest OS 없음)
Hypervisor                         Container Runtime
Host OS                            Host OS (커널 공유)
Hardware                           Hardware
```

**VM 단점**: 하드웨어 자원 파편화 / 하이퍼바이저 의존성 / 게스트 OS마다 오버헤드 / OS 라이선스·패치 관리 부담
**VM 장점(여전히 유효)**: 라이브 마이그레이션, GPU 패스스루, VDI, **커널 수준 완전 격리** (예: Amazon EC2)
**컨테이너 장점**: 호스트 커널 공유 → OS 레이어 불필요, 기동 빠름, 자원 효율 극대화

> 검증 팁: Ubuntu 컨테이너와 Amazon Linux 컨테이너에서 각각 `uname -a` → **같은 커널** 출력. 이미지(유저스페이스)만 다르고 커널은 호스트 것을 쓴다는 결정적 증거.

---

### 1-5. Docker가 이긴 이유
재료(namespaces·cgroups)는 이미 있었다. Jails·Zones·VPARs도 있었다.
차이는 **개발자 친화적 UX**(Dockerfile, 이미지 배포, `docker run` 한 줄) — 채택을 막던 장벽이 기술이 아니라 **사용성**이었기 때문.

---

## 🔜 이 섹션에서 아직 안 나온, 시험에 반드시 나오는 것
- **OCI** (Open Container Initiative): image-spec / runtime-spec / distribution-spec
- **CRI** (Container Runtime Interface) 와 **containerd · CRI-O · runc** 의 계층 관계
- **이미지 레이어**와 유니온 파일시스템(overlayfs), copy-on-write
- Docker Engine 구성: CLI → dockerd → containerd → runc
- Kubernetes의 **dockershim 제거**(v1.24) 배경

---

## 🧪 셀프 체크 (정답은 스스로 정정 → 틀리면 오답노트로)
1. namespaces와 cgroups의 역할 차이를 한 문장씩으로 구분해 설명하시오.
2. chroot가 제공하지 **못한** 격리 3가지는?
3. 컨테이너에 게스트 OS가 없는데도 Ubuntu/CentOS 이미지를 골라 쓸 수 있는 이유는?
4. Pod의 `resources.limits`는 커널의 어떤 기능으로 구현되는가?
5. VM이 컨테이너보다 여전히 유리한 시나리오 2가지는?

---

## 🎯 시험 대비 핵심 암기 체크
- [ ] 컨테이너 = 새 발명이 아니라 Linux 커널 기능(namespaces + cgroups)을 도커가 UX로 포장한 것
- [ ] 루트 파일시스템 격리의 시작 = chroot (1979, Unix V7)
- [ ] 경량 가상화의 초기 형태 = FreeBSD Jails (2000)
- [ ] namespaces = 가시성(시야) 격리 / cgroups = 자원(사용량) 격리 ← 대비 단골
- [ ] namespaces 6종: PID / Network / Mount / UTS / IPC / User (+ cgroup)
- [ ] cgroups 역할: 자원 제한 / 우선순위 / 모니터링·보고 / 제어(start·stop·freeze)
- [ ] cgroups: Google 2006 시작 → 2008 커널 병합
- [ ] Pod의 resources.requests/limits는 최종적으로 cgroups로 구현 (초과 시 OOMKilled)
- [ ] VM은 게스트 OS·하이퍼바이저 오버헤드 / 컨테이너는 호스트 커널 공유
- [ ] VM이 여전히 유리한 경우: 커널 수준 완전 격리, 라이브 마이그레이션, GPU 패스스루, VDI
- [ ] Ubuntu·Amazon Linux 컨테이너의 uname -a가 같은 커널 → 커널은 호스트 공유
- [ ] Docker가 이긴 이유 = 기술이 아니라 개발자 친화적 사용성(UX)
- [ ] cgroups와 namespaces는 별개 커널 기능 (cgroups를 네임스페이스의 한 종류로 착각 금지)

---

## 3-2. Docker Desktop Installation and Configuration (설치와 설정)

### 한 줄 요약
Docker Desktop은 Mac·Windows·Linux 데스크톱에서 **숨겨진 VM/서브시스템 위에 Linux + Docker 런타임**을 올려주는 통합 솔루션이며, 친화적 UI·초기화 편의성·**내장 Kubernetes**까지 제공해 학습 자료로 최적이다. (개인 학습용 무료)

---

### 3-2-1. 전통적 Docker 실행 구조 (2013년 최초 출시)

아래에서 위로 쌓이는 계층 구조:

```
CLI (docker run / build)
Docker (컨테이너 런타임)
 └─ containerd → runc  ← 실제 컨테이너 실행
Operating System (커널 3.10+ 필요, 최신 리눅스면 OK)
Hardware (물리 또는 가상화 하드웨어)
```

- Docker 설치 = 컨테이너 런타임 구성 → 내부적으로 **containerd**와 **runc**로 컨테이너 실행
- 설치 후 **CLI**로 컨테이너를 편리하게 실행·빌드

---

### 3-2-2. Docker Desktop 아키텍처 (전통 방식과의 차이)

Docker Desktop은 Mac·Windows·Linux 데스크톱을 위해 Docker Inc가 만든 친화적 패키지다. 핵심은 **숨겨진 VM/서브시스템**을 활용한다는 점.

```
[ 사용자 데스크톱 ] Windows / Mac OS X / Ubuntu Desktop 등
        │
        ▼
숨겨진 VM 또는 서브시스템  ← OS·버전·아키텍처에 따라 가상화 기술이 다름
        │
        ▼
격리된 Linux OS + Docker 런타임
        │
        ▼
전통적 Docker CLI + 친화적 GUI + 호스트↔Docker 투명 네트워킹
```

- 호스트 OS·버전·아키텍처에 따라 **서로 다른 가상화 기술**로 숨겨진 인스턴스 생성
- 주 시스템과 **분리(segmented)** → 유연성이 큼: 설정에서 **클릭 한 번으로 Docker 환경 전체 초기화** 가능 (내장 K8s 클러스터도 동일하게 리셋 가능)
- **Extensions**: Docker 기반 앱을 빠르게 설치·실행하도록 번들링하는 마켓플레이스 기능

---

### 3-2-3. 설치 절차 (다운로드 → 실행)

1. docker.com 접속 → Windows / Mac / Linux 중 선택
2. Mac은 **Intel 칩** vs **Apple 칩(Apple Silicon)** 두 옵션 → 자기 칩에 맞게 다운로드
3. 설치 프로그램 실행 (Mac은 앱을 Applications로 드래그, Windows/Linux는 각자 인스톤러)
4. Docker 최초 실행 시 **튜토리얼** 제공 → 입문자는 볼 만함 (건너뛰기 가능)
5. **왼쪽 아래 Docker 아이콘 색**으로 상태 확인: **초록색 = 로드 완료**, 시작 단계마다 색이 바뀜
6. Preferences에서 다크 테마 등 커스터마이즈 → apply + restart

> 🎯 포인트: 아이콘이 **초록색**이 되어야 Docker가 사용 준비 완료 상태.

---

### 3-2-4. 첫 컨테이너 실행 실습

```bash
docker run -i -t ubuntu bash
```

| 요소 | 의미 |
| --- | --- |
| `-i` | interactive — 컨테이너와 상호작용 가능하게 함 |
| `-t` | tty 생성 — 터미널 입출력 환경 제공 |
| `ubuntu` | 사용할 이미지 이름 (Docker Hub의 Ubuntu 이미지) |
| `bash` | 실행할 명령. **Ubuntu 컨테이너의 기본 명령이라 생략 가능**(선택 사항) |

- 로컬에 이미지가 없으면 자동으로 **pull(다운로드)** 후 실행
- 컨테이너 진입 시 **hostname이 바뀜** → 격리 확인 (`hostname` 명령으로 검증)
- Ubuntu 시스템처럼 다룰 수 있음: `apt update` → `htop` 설치·실행 등
- `htop`으로 보면 프로세스가 적고, **지정한 `bash`가 PID 1**(메인 진입점)
- `exit` → 호스트 시스템으로 복귀

---

### 3-2-5. 내장 Kubernetes 활성화

- Preferences → **Kubernetes** 창에서 **enable Kubernetes** 선택
- 활성화되면 왼쪽 아래 아이콘이 **둘로 분리**: Docker 아이콘 + Kubernetes 아이콘
- Kubernetes 아이콘이 **초록색**이 되면 준비 완료
- 이후 터미널/명령 프롬프트에서 바로 `kubectl` 사용 가능

```bash
kubectl get nodes   # 사용 가능한 노드와 설치된 K8s 버전 확인
```

> 필요 시 클릭 한 번으로 초기화 가능 → 학습용으로 이상적.

---

### 3-2-6. OS별 차이 (시험보다 실습에서 체감)

| 항목 | macOS | Windows |
| --- | --- | --- |
| 가상화 방식 | **가상 머신(VM)** | **WSL2** (Windows Subsystem for Linux) |
| Resources 창 | 공유 리소스(CPU/메모리 등) **직접 커스터마이즈 가능** | VM이 아니라서 **동일한 방식의 리소스 제어 불가** (변형된 형태) |
| 설치 전제 | 인스톨러 실행 후 바로 사용 | 최초 실행 시 **WSL 업데이트 요구**될 수 있음 |

**Windows에서 WSL 업데이트가 필요할 때:**

```bash
wsl --update   # Docker Desktop 종료 후 명령 프롬프트에서 실행, 프롬프트는 수락
```

- 업데이트 후 Docker 실행 → Mac에서와 동일하게 사용 가능
- Kubernetes 활성화 절차도 Mac과 동일 (Preferences → enable Kubernetes)

---

### 🎯 3-2 시험 대비 핵심 암기 체크
- [ ] Docker 최초 출시 = 2013년
- [ ] 전통적 Docker 런타임 계층: Docker → **containerd → runc**
- [ ] Docker Desktop은 **숨겨진 VM/서브시스템** 위에 Linux+Docker 런타임을 올림
- [ ] OS·버전·아키텍처에 따라 **가상화 기술이 다름**
- [ ] macOS = **VM 기반**(리소스 직접 제어 가능) / Windows = **WSL2 기반**(리소스 제어 방식 다름)
- [ ] `docker run -i -t ubuntu bash`: `-i`=interactive, `-t`=tty, bash는 Ubuntu 기본 명령이라 생략 가능
- [ ] 컨테이너 안 지정 명령(bash)의 **PID = 1**
- [ ] 로컬에 이미지 없으면 자동 **pull**
- [ ] Docker Desktop은 개인 학습용 **무료**, 내장 K8s 제공
- [ ] K8s 활성화 후 `kubectl get nodes`로 노드·버전 확인
- [ ] 아이콘 **초록색 = 준비 완료** (Docker/K8s 각각 표시)

---

## 3-3. Container Images (컨테이너 이미지)

### 한 줄 요약
컨테이너 이미지는 소프트웨어 + 의존성을 담은 **이식 가능한 자체 완결 번들**이며, 여러 개의 **읽기 전용 레이어 스택**으로 구성된다. 컨테이너는 그 이미지로부터 실행되는 **인스턴스**이고, 실행 시 맨 위에 **얇은 쓰기 가능 레이어**가 얹힌다. 이미지의 무결성은 **다이제스트(sha256)** 로 식별·검증한다.

---

### 3-3-1. 이미지란 무엇인가 & 이미지 vs 컨테이너

- **컨테이너 이미지** = 소프트웨어와 그 의존성을 담은 이식 가능(portable)·자체 완결(self-contained) 번들 → 서로 다른 컴퓨팅 환경에서 **일관되게** 실행.
- 흔히 "Docker 이미지"라 부르지만, 더 정확한 표현은 **OCI 호환 컨테이너 이미지(OCI compliant image)**.
  - OCI 준수 덕분에 Docker에서 만든 이미지를 Kubernetes 등 OCI를 지원하는 어디서든 사용 가능.
- **이미지 vs 컨테이너**:

| 구분 | 컨테이너 이미지 | 컨테이너 |
| --- | --- | --- |
| 정체 | 소프트웨어 **번들**(정적) | 이미지로부터 실행된 **인스턴스**(동적) |
| 관계 | 하나의 이미지 | 그 이미지로부터 **여러 개** 실행 가능 |
| 예 | nginx 이미지 1개 | nginx 웹서버 인스턴스 N개 |

> 🎯 포인트: "이미지 = 클래스, 컨테이너 = 인스턴스"로 비유하면 쉽다.

---

### 3-3-2. 레지스트리 & 태그 (latest의 함정)

- **컨테이너 레지스트리(Registry)** = 이미지를 호스팅·배포하는 저장소. 이미지를 만들면 **push**해서 공유.
  - 대표: **Docker Hub**(클라우드 기반, 대다수 도구의 **기본 레지스트리**)
  - **quay.io**: Red Hat의 자체 레지스트리 (Red Hat 도구들은 Docker Hub를 기본으로 가정하지 않는 흐름)
- **태그(Tag)** = 이미지의 버전을 구분하는 **라벨**. 유연하게 사용 가능.
  - 버전 표기(`1.0`, `v2`)로도, 베이스 OS 표기(`alpine`, `ubuntu`)로도 사용.
- **`latest` 태그의 함정 (시험 단골)**:
  - `latest`는 **반드시 최신 버전을 의미하지 않는다.**
  - 태그를 지정하지 않았을 때 Docker가 붙이는 **기본(default) 태그**일 뿐.
  - `docker pull image` == `docker pull image:latest` (동일 동작).

> 🎯 시험 포인트: "`latest`는 항상 최신 버전이다" → **거짓(False)**. 단지 기본 태그.

---

### 3-3-3. docker pull & CLI 구문 변형

**기본 pull**
```bash
docker pull spurin/funbox            # :latest 자동 적용
docker pull spurin/funbox:latest     # 위와 동일
```

- 명령에 레지스트리 주소가 없어도 됨 → **Docker Hub가 기본 레지스트리**라서.
- 레지스트리를 명시하고 싶으면 전체 경로로:
```bash
docker pull docker.io/spurin/funbox:latest
```

**CLI 구문 두 가지 (둘 다 유효)**

| 스타일 | 형식 | 예시 |
| --- | --- | --- |
| 전통(traditional) | `docker <동사>` | `docker pull ...` |
| 신형(noun-verb) | `docker <명사> <동사>` | `docker image pull ...` |

- `docker pull` == `docker image pull` (완전 동등). 취향껏 사용, 다만 **두 형태가 존재함을 인지**.
- pull 시 여러 줄이 **동시에 병렬 다운로드**되는데, 이는 이미지를 구성하는 **레이어들**을 각각 받는 것.

---

### 3-3-4. 레이어(Layers) & 유니온 파일시스템

**레이어의 생성 원리**
- 이미지 빌드의 **각 단계(명령)** 가 레이어를 만든다. (`FROM`, `RUN`, `USER`, `ENV`, `ADD`, `CMD` 등)
- 단, **0바이트인 명령(메타데이터성)** 은 0바이트가 아닌 레이어에 **묶여 병합**된다.
  - 그래서 Dockerfile 단계는 많아도 실제 **다운로드 레이어는 5개**처럼 적게 보인다.

**Dockerfile 주요 지시어(강의 예시)**

| 지시어 | 의미 |
| --- | --- |
| `MAINTAINER` | 관리자 정보 |
| `RUN` | 빌드 중 명령 실행 |
| `USER` | 실행 유저 지정 (예: john → **root로 안 돌리는 게 good practice**) |
| `ENV` | 환경 변수 설정 |
| `ADD` | 파일/디렉터리 추가 |
| `CMD` | 컨테이너 실행 시 **기본 명령** (예: `/menu.sh`) |

**유니온 파일시스템(union filesystem)**
- 이미지는 **읽기 전용 레이어들의 스택** → 유니온 파일시스템이 이를 **하나의 뷰로 병합**.
- 컨테이너 실행 시 맨 위에 **얇은 쓰기 가능 레이어(thin writeable layer)** 가 얹힘. 모든 쓰기는 여기에 기록.

**Copy-on-Write & 삭제의 동작**
- 실행 중 이미지 내 파일을 삭제해도, 실제로는 **쓰기 가능 레이어에 "삭제됨" 표시**만 남긴다.
  - 하위 레이어의 원본은 **여전히 존재**하지만, 컨테이너 시점에서는 **보이지 않을 뿐**.
  - ⚠️ 보안 주의: 비밀번호·API 키 같은 민감 파일은 특정 레이어를 targeting해 **복구 가능** → 이미지에 넣지 말 것.
- **저장 효율**: 같은 이미지로 컨테이너를 여러 개 띄워도, 변하는 건 **각자의 얇은 쓰기 레이어**뿐.
  - 나머지 이미지 레이어는 **공유** → 공간 효율 극대화. 변경/삭제는 해당 컨테이너의 쓰기 레이어에만 적용.

> 🎯 시험 포인트: 이미지 레이어 = **읽기 전용 + 공유**, 컨테이너별 변경 = **맨 위 쓰기 레이어(CoW)**.

---

### 3-3-5. 다이제스트(Digest) & 이미지 ID

- **다이제스트(Digest)** = 레지스트리에서 온 이미지의 **안전하고 고유한 식별자**.
  - 이미지 **매니페스트(manifest)의 해시(sha256)** 로 계산 → **무결성 검증**에 사용.
  - 로컬 이미지가 Docker Hub의 것과 일치하는지 대조 가능.
- **다이제스트로 pull** 도 가능 (버전을 정확히 고정하고 싶을 때):
```bash
docker pull spurin/funbox@sha256:<digest>
```
- 다이제스트는 이미지의 **최상위(top-level) 식별자** → 모든 아키텍처(386/amd64/arm64…)와 그 하위 레이어 전체를 포괄.
  - 그래서 **어느 아키텍처에서 봐도 동일**한 ID. (멀티아키텍처 이미지 전체를 대표)

**BuildX로 원시 매니페스트 확인**
```bash
docker buildx imagetools inspect spurin/funbox --raw
```
- 아키텍처별 개별 다이제스트, **attestation manifests**(빌드 방법·검증 정보를 담은 아티팩트) 확인 가능.
- 이 `--raw` 출력을 `sha256`에 통과시키면 → 앞서 본 **다이제스트 ID와 동일** (검증 완료).

---

### 3-3-6. (심화) 이미지 내부 구조 뜯어보기 — `docker save`

> 시험 필수는 아니지만, 다이제스트 → 매니페스트 → 레이어 관계를 눈으로 이해하는 심화 실습.

```bash
mkdir /tmp/funbox && cd /tmp/funbox
docker save spurin/funbox -o funbox.tar   # 이미지를 tar로 저장(다른 곳 import용)
tar xvf funbox.tar                         # blobs/ + 파일들 추출
```

**추출된 구조 따라가기**

| 파일 | 역할 |
| --- | --- |
| `index.json` | **최상위 시작점/인덱스**. 이미지 ID와 일치하는 다이제스트 포함 |
| `manifest.json` | 앞서 본 **5개 레이어** + **config 파일** 참조 |
| `blobs/sha256/<hash>` | 실제 데이터. **파일명 = 자기 내용의 sha256 체크섬** (`cat 파일 | sha256` → 파일명과 일치) |
| config (json) | Dockerfile 빌드 정보(USER, ADD 등) — 이미지가 어떻게 만들어졌는지 |

**따라가는 흐름**: `index.json`(최상위 다이제스트) → 멀티아키텍처 목록에서 **내 아키텍처(예: arm64)** 매칭 → 해당 blob → **config + 레이어 1~5** 참조.

> 🎯 핵심: 이미지 ID(다이제스트)는 **최상위 메타데이터**이고, 그 아래로 blobs 디렉터리 → 아키텍처별 매니페스트 → config + 개별 레이어로 **트리처럼** 연결된다.

---

### 🎯 3-3 시험 대비 핵심 암기 체크
- [ ] 컨테이너 이미지 = 소프트웨어+의존성의 이식 가능·자체 완결 번들, 정확한 명칭은 **OCI 호환 이미지**
- [ ] 이미지 = 정적 번들 / 컨테이너 = 이미지로부터 실행된 **인스턴스** (1 이미지 → N 컨테이너)
- [ ] 레지스트리 = 이미지 호스팅·배포 저장소. **Docker Hub = 기본 레지스트리**, Red Hat = **quay.io**
- [ ] 태그 = 이미지 버전을 구분하는 라벨 (버전/베이스 OS 등 자유)
- [ ] **`latest`는 최신 버전이 아니라 기본(default) 태그** ← 단골 함정
- [ ] `docker pull img` == `docker pull img:latest`, 레지스트리 생략 시 Docker Hub
- [ ] CLI 두 형태: `docker pull`(전통) == `docker image pull`(명사-동사)
- [ ] 이미지 = 읽기 전용 레이어 스택, 실행 시 맨 위 **얇은 쓰기 가능 레이어** 추가
- [ ] 유니온 파일시스템이 레이어를 **하나의 뷰로 병합**
- [ ] 파일 삭제 = 쓰기 레이어에 "삭제 표시"만, 하위 원본은 남음 → **민감정보 이미지 포함 금지**
- [ ] 여러 컨테이너가 이미지 레이어 **공유**, 각자 쓰기 레이어만 별도 (저장 효율)
- [ ] 0바이트 명령은 병합됨 → Dockerfile 단계 多여도 다운로드 레이어는 적음(예: 5개)
- [ ] 다이제스트 = 매니페스트의 **sha256 해시**, 무결성 검증·`@sha256:`로 pull 가능
- [ ] 다이제스트는 멀티아키텍처 전체를 대표 → 아키텍처 무관하게 동일 ID
- [ ] `docker buildx imagetools inspect --raw`로 원시 매니페스트·attestation 확인

---

### 🧪 셀프 체크 (틀리면 오답노트로)
1. 이미지와 컨테이너의 차이를 한 문장으로 설명하시오.
2. `latest` 태그에 대한 흔한 오해와 실제 의미는?
3. 실행 중 컨테이너에서 파일을 지우면 실제로 어떤 일이 일어나는가? (레이어 관점)
4. 다이제스트는 무엇의 해시이며 어디에 쓰이는가?
5. Dockerfile 단계 수보다 다운로드되는 레이어 수가 적은 이유는?
