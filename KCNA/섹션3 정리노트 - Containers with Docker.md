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

---

## 3-4. Running Containers (컨테이너 실행하기)

### 한 줄 요약
Docker는 **클라이언트-서버 아키텍처**로 동작하며(엔진=containerd, 런타임=runc), `docker run [옵션] 이미지 [명령]` 한 줄로 컨테이너를 실행한다. `-i`(대화형)·`-t`(터미널)·`--rm`(종료 시 자동 삭제)가 핵심 옵션이고, 이미지의 **기본 명령(CMD)** 은 명령줄에서 **재정의(override)** 할 수 있다.

---

### 3-4-1. docker version & 클라이언트-서버 아키텍처

```bash
docker version   # 클라이언트 + 서버 버전/구성 확인
```

- Docker는 **클라이언트-서버 아키텍처** → 클라이언트와 서버(daemon)를 **독립적으로** 실행 가능.
- **Docker Desktop**은 클라이언트와 서버를 **함께** 실행.
- **containerd**: Docker가 사용하는 **컨테이너 엔진**. Docker가 **CNCF에 기증** → 현재 **graduated(졸업)** 프로젝트.
- **runc**: **컨테이너 런타임의 표준 참조 구현(reference implementation)**. Docker가 **OCI(Open Container Initiative)에 기증**.

> 🎯 포인트: 기증처 구분 — **containerd → CNCF**, **runc → OCI**.

---

### 3-4-2. help 구문 활용

```bash
docker --help          # Docker 전체 파라미터 목록
docker run --help      # 특정 명령(run)의 세부 옵션 확인
```

- `docker --help` → 사용 가능한 **모든 명령/파라미터** 조회.
- 명령줄 파라미터 뒤에 `--help`를 붙이면 **해당 명령의 세부 옵션**까지 파고들 수 있음.

---

### 3-4-3. docker run & 핵심 옵션

```bash
docker pull spurin/funbox            # 이미지가 없으면 먼저 받기
docker run -i -t --rm spurin/funbox  # funbox readme의 실행 예시
```

| 옵션 | 의미 |
| --- | --- |
| `-i` | interactive — 컨테이너와 상호작용 |
| `-t` | terminal(tty) — 터미널 입출력 환경 |
| `--rm` | 컨테이너가 **종료되면 자동으로 제거** |

- 이름을 지정하지 않으면 컨테이너에 **무작위 이름**이 부여됨 (시스템마다 다름).
- funbox 실행 시 **메뉴 시스템**이 뜸 → 옵션 선택(예: 6=nyancat, 10=train)으로 다양한 데모 실행. `CTRL-C`로 중단.
- **핵심 체감**: 필요한 모든 것이 이미지로 번들 → 소프트웨어를 **빠르고 쉽게** 실행.

---

### 3-4-4. 컨테이너 라이프사이클 — ps / ps -a / rm

```bash
docker ps         # 실행 중(running) 컨테이너만 표시
docker ps -a      # 종료(exited) 포함 모든 컨테이너 표시
docker rm <id|name>   # 컨테이너 삭제 (ID 또는 이름 사용)
```

- `--rm` **없이** 실행하면, 종료 후에도 컨테이너가 **exited 상태로 남음** → `docker ps -a`로 확인 가능.
- `docker ps -a` 출력 읽기:
  - **COMMAND** = 컨테이너가 실행한 명령 (funbox 기본값 = **`/menu.sh`**, 이미지의 **기본 명령/CMD**)
  - **왼쪽 = 컨테이너 ID**, **오른쪽 = 무작위 이름**
- 남은 컨테이너는 `docker rm`으로 정리 (ID/이름 둘 다 가능).
- `--rm`을 붙이면 종료 시 **자동 정리** → 남는 컨테이너 없음.

> 🎯 포인트: 나중에 재사용/상호작용하려면 `--rm` **없이**, 일회성이면 `--rm` **붙여서** 실행.

---

### 3-4-5. 기본 명령(CMD) 재정의 & non-root 사용자

```bash
docker run -i -t --rm spurin/funbox nyancat   # CMD(/menu.sh)를 nyancat으로 덮어쓰기
docker run -i -t --rm spurin/funbox bash      # bash 셸 실행
```

- 이미지의 **기본 명령(CMD, funbox=`/menu.sh`)** 은 명령줄 인자로 **재정의(override)** 가능.
- 컨테이너 안에서 실행되는 것은 결국 **하나의 명령/바이너리**일 뿐 (bash, nyancat, cmatrix, asciiaquarium 등 직접 실행 가능).
- `bash`로 진입하면 사용자 ID가 **`john`** → Dockerfile의 **`USER` 레이어**가 지정한 값.
  - ⭐ **비root(non-root) 사용자로 실행 = good practice** (보안상 권장).

> 🎯 시험 포인트: 컨테이너를 **root가 아닌 사용자**로 돌리는 것이 권장 관행.

---

### 🎯 3-4 시험 대비 핵심 암기 체크
- [ ] Docker = **클라이언트-서버 아키텍처** (클라이언트/서버 분리 실행 가능, Desktop은 함께)
- [ ] 컨테이너 엔진 = **containerd** (Docker → **CNCF** 기증, **graduated**)
- [ ] 컨테이너 런타임 참조 구현 = **runc** (Docker → **OCI** 기증)
- [ ] `docker run` 옵션: `-i`=interactive, `-t`=terminal(tty), `--rm`=종료 시 자동 삭제
- [ ] 이름 미지정 시 **무작위 이름** 부여
- [ ] `docker ps`=실행 중만 / `docker ps -a`=종료 포함 전체
- [ ] funbox 기본 명령(CMD) = **`/menu.sh`**, 명령줄로 **override 가능**
- [ ] `docker rm`은 컨테이너 **ID 또는 이름**으로 삭제
- [ ] `docker --help` / `docker run --help`로 옵션 조회
- [ ] 컨테이너는 **non-root 사용자**(예: john)로 실행 = 권장 관행

---

### 🧪 셀프 체크 (틀리면 오답노트로)
1. `docker run`의 `-i`, `-t`, `--rm` 각각의 역할을 설명하시오.
2. containerd와 runc는 각각 어느 재단(CNCF/OCI)에 기증되었는가?
3. `docker ps`와 `docker ps -a`의 차이는?
4. 이미지의 기본 명령(CMD)을 명령줄에서 어떻게 바꾸는가? funbox의 기본 CMD는?
5. 컨테이너를 non-root 사용자로 실행하는 것이 왜 권장되는가?

---

## 3-5. Container Network Services and Volumes (네트워크 서비스와 볼륨)

### 한 줄 요약
컨테이너가 포트에서 리스닝하더라도, 호스트에서 접근하려면 포트를 **게시(publish)** 해야 한다. `-P`(대문자)는 이미지의 **EXPOSE** 를 근거로 **모든 포트를 무작위 호스트 포트**에 매핑하고, `-p 호스트:컨테이너`(소문자)는 **특정 포트**를 지정 매핑한다. 실행 중 컨테이너에는 `docker exec`로 **또 다른 프로세스(셸 등)** 를 붙일 수 있고, 컨텐츠는 컨테이너 내부를 직접 고치는 대신 **볼륨(-v)** 으로 주입하는 것이 올바른 방식이다.

---

### 3-5-1. nginx로 실습하는 이유 & 포그라운드 vs 백그라운드(-d)

- **nginx 웹서버 이미지**는 **네트워킹 서비스**와 **볼륨**을 한 번에 테스트하기 좋은 편리한 이미지.
- 웹서버라 터미널과 상호작용할 필요가 없음 → `-i -t` **불필요**.

```bash
docker run --rm nginx        # 포그라운드 실행 (터미널 점유), CTRL-C로 중단
docker run --rm -d nginx     # -d = detach, 백그라운드 실행 → 컨테이너 ID 반환
```

| 옵션 | 의미 |
| --- | --- |
| `--rm` | 종료 시 자동 정리(clean up) |
| `-d` | detach — 백그라운드 실행. 실행 결과로 **컨테이너 ID** 출력 |

- 이미지가 로컬에 없으면 run 과정에서 자동 **pull**(레이어·다이제스트 확인 가능).
- 백그라운드 실행 후 상태 확인:

```bash
docker ps        # 실행 중 컨테이너 확인 (무작위 이름 부여됨)
```

> 🎯 포인트: 웹서버처럼 상호작용이 필요 없는 서비스는 `-d`로 백그라운드 실행.

---

### 3-5-2. 포트 게시(publish) — 리스닝 ≠ 접근 가능

- `docker ps`의 포트 표기가 **`80/tcp`** 이면, 컨테이너가 80에서 **리스닝**은 하지만 **게시(publish)되지 않은** 상태 → 호스트에서 쉽게 접근 불가.
- 컨테이너 중지: 컨테이너 ID를 **식별 가능한 만큼만** 입력해도 됨. `--rm`이라 중지 시 자동 삭제.

```bash
docker stop <id-앞부분>   # --rm이면 중지와 동시에 자동 정리
```

**두 가지 게시 방법**

| 옵션 | 동작 | 결과 표기 |
| --- | --- | --- |
| `-P` (대문자) | 이미지의 **EXPOSE** 지시어 기반으로 **모든 포트**를 **무작위 호스트 포트**에 매핑 | `0.0.0.0:무작위->80/tcp` |
| `-p 호스트:컨테이너` (소문자) | **특정 포트**를 지정 매핑 | `0.0.0.0:12345->80/tcp` |

```bash
docker run --rm -d -P nginx              # EXPOSE 기반 전체 포트 무작위 게시
docker run --rm -d -p 12345:80 nginx     # 호스트 12345 → 컨테이너 80
```

- `-P`는 **매번 무작위 포트** 사용. `0.0.0.0` 바인딩 = **모든 IPv4 주소**에서 접근 가능.
- `EXPOSE`는 Docker Hub의 nginx 이미지 instruction에서 확인 가능.
- 접근 확인은 브라우저 또는 `curl`:

```bash
curl localhost:12345    # nginx 기본 환영 페이지 확인
```

> 🎯 시험 포인트: **`-P`(대문자)=EXPOSE 기반 전체 무작위 게시**, **`-p`(소문자)=특정 포트 지정**. 리스닝만으로는 호스트 접근 불가 → **publish 필요**.

---

### 3-5-3. 실행 중 컨테이너 안으로 진입 — docker exec

- 실행 중 컨테이너에는 **또 다른 프로세스**를 붙여 실행할 수 있음. nginx가 메인 프로세스(**PID 1**)로 돌아가는 상태에서 셸을 추가로 띄우는 식.

```bash
docker exec -it <컨테이너ID> bash    # 실행 중 컨테이너에서 bash 실행
```

| 요소 | 의미 |
| --- | --- |
| `docker exec` | **실행 중인** 컨테이너에서 명령 실행 |
| `-i -t` | interactive + tty (터미널 상호작용) |
| `bash` | 컨테이너 안에서 띄울 프로세스 |

- 컨테이너 안에서 기본 웹페이지 위치 찾기 (Unix `find`):

```bash
find / -name index.html 2>/dev/null    # 에러는 /dev/null로 버림
```

- 찾은 index.html을 덮어써 즉시 반영 확인:

```bash
echo hello > <index.html 경로>    # 새로고침하면 페이지 변경됨
```

> ⚠️ 주의: **컨테이너 안에 직접 들어가 파일을 수정하는 방식은 비효율적이고 좋은 전략이 아니다.** → 더 나은 방법은 다음 항목의 **볼륨**.

---

### 3-5-4. 볼륨(Volume)으로 컨텐츠 주입 — -v

- 웹사이트(컨텐츠)를 담은 **볼륨을 컨테이너에 전달**하는 것이 올바른 방식. 컨테이너 내부를 직접 고치지 않는다.

```bash
mkdir website
echo "hello from James" > website/index.html

docker run --rm -d -p 12345:80 \
  -v <호스트 전체경로>/website:/usr/share/nginx/html nginx
```

| 형식 | 의미 |
| --- | --- |
| `-v 호스트경로:컨테이너경로` | 호스트 디렉터리를 컨테이너 경로에 마운트(덮어씀) |

- 호스트 경로는 반드시 **전체 시스템 경로(full path)** 여야 함.
- **Windows(명령줄/WSL)** 는 볼륨 경로 **문법이 다름** → 강의 화면의 예시 문법 참고.
- 볼륨의 강력함: 컨테이너를 건드리지 않고 **호스트의 로컬 파일만 수정**해도 즉시 반영.

```bash
echo Spurin >> website/index.html    # 로컬 파일 수정 → 새로고침 시 바로 반영
```

> 🎯 시험 포인트: 컨텐츠는 컨테이너 내부 수정이 아니라 **볼륨(-v)** 으로 주입. 호스트 파일 변경이 컨테이너에 실시간 반영됨.

---

### 🎯 3-5 시험 대비 핵심 암기 체크
- [ ] nginx = 네트워킹 서비스 + 볼륨 테스트에 편리한 이미지, 웹서버라 `-i -t` 불필요
- [ ] `-d` = detach(백그라운드 실행), 실행 결과로 **컨테이너 ID** 반환
- [ ] `docker ps` 포트가 **`80/tcp`** = 리스닝은 하나 **게시 안 됨** → 호스트 접근 불가
- [ ] 컨테이너 중지는 **ID 앞부분만**으로도 가능, `--rm`이면 중지 시 자동 정리
- [ ] **`-P`(대문자)** = 이미지 **EXPOSE 기반 전체 포트를 무작위 호스트 포트**에 매핑
- [ ] **`-p 호스트:컨테이너`(소문자)** = 특정 포트 지정 매핑 (예: `-p 12345:80`)
- [ ] `0.0.0.0` 바인딩 = 모든 IPv4 주소에서 접근 가능
- [ ] `docker exec -it <id> bash` = **실행 중** 컨테이너에 또 다른 프로세스(셸) 붙이기
- [ ] 컨테이너 안 직접 수정은 나쁜 관행 → **볼륨(-v)** 으로 컨텐츠 주입이 정석
- [ ] `-v 호스트전체경로:컨테이너경로`, 호스트 경로는 **full path**, Windows는 문법 다름
- [ ] 볼륨 사용 시 **호스트 로컬 파일 수정이 컨테이너에 실시간 반영**

---

### 🧪 셀프 체크 (틀리면 오답노트로)
1. `-P`(대문자)와 `-p`(소문자)의 차이를 설명하시오. `-P`는 무엇을 근거로 매핑하는가?
2. `docker ps`에서 `80/tcp`로만 표시될 때 호스트에서 접근이 안 되는 이유는?
3. `docker exec`는 어떤 상태의 컨테이너에 무엇을 하는 명령인가?
4. 웹 컨텐츠를 바꿀 때 컨테이너 내부를 직접 수정하면 안 되는 이유와 올바른 대안은?
5. 볼륨 마운트 시 호스트 경로에 대한 제약(그리고 Windows 특이사항)은?
