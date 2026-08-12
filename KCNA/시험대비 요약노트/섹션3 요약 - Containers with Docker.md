# 섹션3 요약 — Containers with Docker

> 원본: kcna-transcripts-clean.md · 섹션3 / 시험대비 암기용
> 깊이: 중 · 컨테이너 기반 개념(역사·namespaces/cgroups·이미지·OCI)

## 0. 시험 포인트 한눈에 (암기표)

| 주제 | 반드시 외울 것 |
| --- | --- |
| 컨테이너의 두 재료 | **namespaces**(가시성) + **cgroups**(리소스) |
| namespaces 종류 | User, PID, Network, Mount, UTS(hostname), IPC |
| cgroups 유래 | **Google 2006** "process containers" → 2008 커널 병합. 리소스 limit/우선순위/accounting/control |
| chroot | 1979 Unix V7. 루트 디렉토리 변경(파일시스템 격리). user/hostname/IP는 공유 |
| FreeBSD Jails | 2000. 자체 network·PID. 컨테이너 조상격 |
| VM vs Container | VM=게스트 OS별 존재(무거움) / Container=**호스트 커널 공유**(가벼움) |
| Docker 대중화 시점 | 2010 dotCloud → **2013 Docker**. 신기술 발명 아닌 **사용성**으로 성공 |
| 이미지 vs 컨테이너 | 이미지=번들 / 컨테이너=이미지의 **실행 인스턴스**(1 이미지 → N 컨테이너) |
| 기본 레지스트리 | **Docker Hub** (Red Hat은 quay.io) |
| **latest 태그** | 기본 태그일 뿐 **"최신 버전" 보장 아님** (대표 함정) |
| 이미지 구조 | **레이어 스택 + union filesystem** + 최상단 thin writable layer |
| digest | 이미지 manifest의 **sha256 해시** = 무결성 검증용 고유 ID |
| 저수준/고수준 런타임 | low-level=**runc**(참조구현) / high-level·engine=**containerd** |
| CMD vs ENTRYPOINT | CMD=기본 명령(완전 오버라이드) / ENTRYPOINT=실행파일(인자로 전달) |

## 1. 핵심 개념 정리
### 1-1. 가상화·격리 기술 연표
| 연도 | 기술 | 핵심 |
| --- | --- | --- |
| 1960s | IBM mainframe **CP/CMS** | 최초 실용 VM. 시분할 넘어 **사용자별 가상 OS** |
| 1979 | **chroot** (Unix V7) | 루트 디렉토리 변경 = 파일시스템 격리. user/hostname/IP는 공유 |
| 2000 | **FreeBSD Jails** | 자체 network·PID. 컨테이너 조상격. 사용성 나빠 대중화 실패 |
| 2000s | Solaris **Zones**, HP-UX **VPARs** | 동시대 격리 시도 |
| 2002~ | Linux **namespaces** | 가시성 격리(순차 도입) |
| 2006 | Google **cgroups** | 리소스 격리(2008 커널 병합) |
| 2013 | **Docker** | dotCloud(2010)→Docker. 컨테이너 대중화 |

### 1-2. Namespaces & cgroups (컨테이너의 두 재료)
- **Namespaces = "무엇을 볼 수 있는가"(가시성)**. 종류:
  - **User**(uid/gid, 네임스페이스 내 root 가능), **PID**(독립 프로세스 ID),
    **Network**(독립 IP·스택), **Mount**(독립 마운트), **UTS**(독립 hostname/
    domain), **IPC**(프로세스 간 통신).
- **cgroups = "얼마나 쓸 수 있는가"(리소스)**. Google 2006 시작, 2008 커널 병합.
  - 기능: **resource limits, prioritization, accounting, control**(start/
    stop/frozen/restart).
- 둘이 합쳐질 때 완전한 리소스 격리 가능. Docker가 이를 쉽게 포장.

### 1-3. VM vs Container / Docker
- **VM**: 하이퍼바이저(VMware ESXi 등) 위 각 VM이 **자체 게스트 OS** 보유.
  - 단점: 리소스 분할·하이퍼바이저 의존·게스트 OS 오버헤드·라이선스 부담.
  - 장점: 커널 수준 완전 격리, live migration, GPU passthrough, VDI(예: EC2).
- **Container**: 호스트 **커널 공유** → OS 레이어 불필요, 가볍고 빠름.
  - 모든 컨테이너가 같은 커널 참조(`uname -a` 동일). namespaces로 자체 user/
    hostname/IP/mount, cgroups로 리소스 제어.
- **Docker**: 2010 dotCloud → 2013 Docker. 차별점은 신기술이 아니라 **사용성**
  (Dockerfile·이미지 배포·`docker run` 한 줄).

### 1-4. Docker vs Docker Desktop
- **Docker(전통)**: 컨테이너 런타임(내부에 **containerd + runc**). 최소 커널
  3.10 초과 필요.
- **Docker Desktop**: Mac/Windows/Linux 데스크톱용. **숨은 VM/서브시스템**으로
  격리 실행. Windows는 **WSL** 사용. K8s 노드 내장, Extensions, 친화적 UI.
  containerd가 CNCF graduated, runc는 OCI 기증 참조 구현.

### 1-5. Container Images (레이어·tag·digest)
- **이미지 = 소프트웨어+의존성의 이식 가능한 번들**(OCI 호환). = "OCI 호환
  컨테이너 이미지"(Docker 이미지라고도 부름).
- **이미지 vs 컨테이너**: 이미지=번들, 컨테이너=이미지의 **실행 인스턴스**.
  1개 이미지로 N개 컨테이너 실행.
- **Registry**: 이미지 호스팅/공유. 기본 = **Docker Hub**(대부분 툴의 기본).
  Red Hat은 **quay.io**.
- **Tag**: 이미지 버전 라벨. **`latest`는 태그 미지정 시 기본값일 뿐 "최신
  버전" 아님**(핵심 함정).
- **레이어 & Union Filesystem**: 이미지는 레이어 스택. 실행 시 union fs가
  레이어를 단일 뷰로 병합 + 최상단 **thin writable layer**. 파일 삭제는
  writable layer에만 표시(하위 레이어엔 잔존 → 시크릿 노출 위험).
- **Digest**: 이미지 manifest의 **sha256 해시** = 무결성 검증 고유 ID.
  digest로 pull 가능.
- CLI 문법 2형식: `docker pull`(verb) = `docker image pull`(noun verb).

### 1-6. Dockerfile & 이미지 빌드 베스트프랙티스
- **Dockerfile** = 이미지 조립 명령 텍스트 파일. 각 명령 = 레이어 생성(0바이트
  명령은 다음 비-0 레이어에 묶임).
- 주요 지시어:
  - **FROM**(베이스 이미지), **LABEL**(MAINTAINER 대신 권장), **RUN**(빌드 중
    실행), **WORKDIR**(디렉토리 생성/이동, 유지됨), **COPY**, **ENV**,
    **USER**(비-root 실행 = 권장), **CMD**(기본 명령), **ENTRYPOINT**.
- **베스트프랙티스**: RUN 명령 합치기(레이어 축소), **multi-stage build**
  (빌드 단계 + 최소 런타임 단계, `COPY --from=builder`), 비-root USER,
  경량 베이스(**Alpine** <10MB, **apk** 패키지 매니저).
- **buildx**: 멀티아키(amd64·arm64) 빌드 후 레지스트리 push.
- `docker system prune` = 미사용 리소스 정리.

## 2. 헷갈리는 것 구분 (비교표)

### VM vs Container
| 구분 | VM | Container |
| --- | --- | --- |
| OS | 게스트 OS **별도** | 호스트 커널 **공유** |
| 무게 | 무거움 | 가벼움 |
| 격리 | 커널 수준 완전 격리 | namespaces+cgroups |
| 강점 | live migration·GPU·VDI | 밀도·속도·이식성 |

### namespaces vs cgroups
| 구분 | namespaces | cgroups |
| --- | --- | --- |
| 역할 | **가시성**(무엇을 보나) | **리소스**(얼마나 쓰나) |
| 예 | PID·Net·Mount·UTS·IPC·User | CPU·mem limit·우선순위 |

### 이미지 vs 컨테이너 / tag vs digest
| 구분 | 의미 |
| --- | --- |
| Image | 소프트웨어 번들(정적) |
| Container | 이미지의 실행 인스턴스 |
| Tag | 사람이 붙인 라벨(latest=기본값, 최신 아님) |
| Digest | manifest의 sha256(불변·무결성) |

### 런타임 계층 / CMD vs ENTRYPOINT
| 구분 | 내용 |
| --- | --- |
| low-level runtime | **runc**(참조 구현), crun·kata·gVisor |
| high-level runtime/engine | **containerd**(이미지 관리·런타임 호출) |
| CMD | 기본 명령, 실행 시 완전 오버라이드 |
| ENTRYPOINT | 컨테이너를 실행파일처럼, CMD/args는 인자로 |

## 3. 함정·키워드

- **latest ≠ 최신 버전**. 태그 미지정 시 쓰이는 **기본 태그**일 뿐.
- 컨테이너 두 재료 = **namespaces + cgroups** (하나만 답하면 함정).
- **cgroups = Google 2006**, 커널 병합 2008. namespaces와 역할 혼동 주의
  (namespaces=가시성, cgroups=리소스).
- 컨테이너는 **호스트 커널 공유** → 모든 컨테이너 `uname` 동일.
- **runc = 저수준 런타임(참조 구현)**, **containerd = 고수준 엔진**.
- 삭제한 파일도 **하위 레이어엔 남음**(union fs) → 시크릿을 이미지에 넣지 말 것.
- Docker의 성공 요인 = **사용성**(신기술 발명 아님).
- Docker 기본 레지스트리 = **Docker Hub**.

## 4. 자가 점검 Q&A

1. Q: 컨테이너를 가능하게 하는 리눅스 커널 두 기능은?
   A: **namespaces**(가시성) + **cgroups**(리소스).
2. Q: cgroups는 누가 언제 시작했고 언제 커널에 병합됐나?
   A: **Google 2006**(process containers) → **2008** 커널 병합.
3. Q: `latest` 태그의 의미는?
   A: 태그 미지정 시 **기본 태그**. 최신 버전을 보장하지 않음.
4. Q: 이미지와 컨테이너의 차이는?
   A: 이미지=정적 번들, 컨테이너=이미지의 **실행 인스턴스**.
5. Q: 이미지 무결성 검증에 쓰는 sha256 식별자는?
   A: **digest**(manifest의 해시).
6. Q: 저수준 런타임의 참조 구현과 대표 고수준 엔진은?
   A: **runc** / **containerd**.
7. Q: VM과 컨테이너의 가장 큰 구조적 차이는?
   A: VM=게스트 OS 별도, 컨테이너=**호스트 커널 공유**.
8. Q: chroot의 한계는?
   A: 파일시스템은 격리하나 **user·hostname·IP는 공유**.
9. Q: CMD와 ENTRYPOINT 차이는?
   A: CMD=기본 명령(오버라이드), ENTRYPOINT=실행파일화(인자로 전달).
10. Q: 이미지 크기를 줄이는 대표 빌드 기법은?
    A: **multi-stage build** + 경량 베이스(Alpine) + RUN 합치기.

