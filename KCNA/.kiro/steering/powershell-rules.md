---
inclusion: always
---

# Script Execution Rules (.ps1, .py)

## 0. 핵심 원칙 (업데이트)
- `.py`/`.ps1` 스크립트는 **필요하면 만들어서 실행해도 된다.** 과거에 생성/실행/삭제가
  잘 안 되던 문제는 해결되었고, 아래 절차대로 하면 안정적으로 동작한다(실사례 검증됨).
- 다만 **불필요한 워크스페이스 오염은 피한다.** 짧으면 인라인, 길거나 재사용/검증이
  필요하면 지정된 스크래치 위치에 파일로 만든다.
- 실행 방법 우선순위:
  1. **인라인** (`python -c` / `powershell -Command`) — 짧은 1회성 코드
  2. **stdin 파이핑** — 파일 없이 여러 줄 실행
  3. **스크립트 파일** — 길거나 재사용/검증용. `fs_write`+`fs_append`로 생성
     - 특정 spec 동작용 스크립트 → 해당 spec의 `.kiro/specs/<feature>/scripts/`
     - 그 외 범용 스크래치 → `.kiro/tmp/`
  4. `%TEMP%` — 셸로만 생성 가능(아래 6절 참고). fs 도구로는 접근 불가

## 1. 짧은 코드 → 인라인 실행
- Python: `python -c "코드"`
- PowerShell: `powershell -NoProfile -NonInteractive -ExecutionPolicy Bypass -Command "코드"`
- 여러 명령 연결 시 **`&&` 금지** (Windows PowerShell 미지원). `;`를 쓰거나 도구 호출을 분리한다.
  - 나쁨: `python a.py && echo OK`
  - 좋음: `python a.py; if ($?) { echo OK }`

## 2. 긴 코드 → stdin 파이핑 (파일 없이)
파일을 만들지 않고 여러 줄을 실행할 수 있다.

```
powershell -NoProfile -ExecutionPolicy Bypass -Command "@'
print('여러 줄')
for i in range(3):
    print(i)
'@ | python -"
```

- `python -` 의 `-` 는 "stdin에서 코드를 읽으라"는 의미이며 디스크에 파일을 남기지 않는다.
- 단, 한글/한자 등 비-ASCII가 많거나 따옴표가 복잡하면 인용 처리가 깨지기 쉽다.
  이럴 때는 3절의 스크래치 파일 방식이 더 안전하고 검증도 쉽다.

## 3. 워크스페이스 스크래치 파일 (권장 — 길거나 검증이 필요할 때)
길거나 비-ASCII/따옴표가 많거나, 재실행·검증이 필요한 코드는 스크립트 파일로 만든다.

### 3-1. 생성 위치
- **`fs_write`/`fs_append`는 워크스페이스 안에서만 동작한다.** (`%TEMP%` 등 외부 경로는 "Access denied"로 거부됨.)
- 스크립트는 용도에 따라 아래 위치에 둔다:
  - **특정 spec의 동작을 위해 만드는 스크립트** → 반드시 해당 spec 폴더 아래
    `.kiro/specs/<feature>/scripts/`에 둔다(없으면 생성). 이렇게 하면 스크립트가
    어느 spec에 속하는지 명확해지고 함께 보관·추적된다.
  - **특정 spec에 묶이지 않는 범용 스크래치** → `.kiro/tmp/`(없으면 생성).
- 앱 소스 트리(`src/main/...`)에는 스크립트를 만들지 않는다.

### 3-2. 생성 방법 (청크로 나눠 저장)
- 한 번의 `fs_write`/`fs_append` 호출이 너무 크면 잘릴 수 있으므로,
  **뼈대 → `fs_append`로 분할 저장**한다(한 청크 40줄 이내 권장).
- `fs_write`는 UTF-8로 저장되며, Python3는 소스를 UTF-8로 해석하므로 한글/한자 문자열도 안전하다.
- properties 등 `\uXXXX` 이스케이프가 필요한 산출물은, 스크립트 안에서 네이티브 문자열을
  받아 이스케이프를 **코드로 생성**하게 하면 손이스케이프 오류를 피할 수 있다.

### 3-3. 실행·검증
- 문법 먼저 검사: `python -m py_compile "경로"` (성공 시 조용히 종료)
- 실행: `python "경로"`
- 스크립트는 가능하면 **멱등(idempotent)** 하게 작성한다(이미 처리된 항목은 skip).
  재실행해도 중복이 생기지 않아 검증에 그대로 재사용할 수 있다.
- 실행 후 별도 조회 명령으로 **결과를 검증**한다(예: 등록한 키가 대상 파일에 실제로 존재하는지).

### 3-4. 정리
- 스크립트는 더 이상 필요 없으면 `delete_file`로 삭제해도 된다.
- 재실행·검증·재현에 재사용할 가치가 있으면 보관한다. 보관할 경우 특정 spec용은
  `.kiro/specs/<feature>/scripts/`에, 범용 스크래치는 `.kiro/tmp/`에 둔다.

## 4. (선택) %TEMP% 사용
- 워크스페이스를 전혀 건드리고 싶지 않을 때만 사용한다.
- **`fs_write`로는 불가**하므로 셸로 생성해야 한다. 예:
  ```
  powershell -NoProfile -Command "$f=Join-Path $env:TEMP ('kiro_'+[guid]::NewGuid().ToString('N')+'.py'); Set-Content -Encoding UTF8 $f 'print(123)'; python $f"
  ```
- 비-ASCII를 다룰 때는 `-Encoding UTF8`을 반드시 지정한다(기본 인코딩은 한글이 깨질 수 있음).
- `%TEMP%` 파일은 OS가 정리하므로 굳이 즉시 삭제하지 않아도 된다.

## 5. 터미널 출력 판독 주의 (중요)
- 이 환경의 대화형 셸은 명령을 **문자 단위로 반복 에코**해 출력이 매우 지저분해 보일 수 있다.
  이는 PSReadLine 렌더링 아티팩트일 뿐 **실패가 아니다.**
- 성공 여부는 지저분한 에코가 아니라 **실제 프로그램 출력(예: `added 50 keys`, `COMPILE_OK`)** 으로 판단한다.
- 프롬프트에 표시되는 `Exit Code: -1` 역시 이 아티팩트로 인한 것일 수 있으니,
  실제 출력이 기대대로면 성공으로 간주한다.

### 5-1. 에코 깨짐 근본 예방 (PSReadLine 제거)
- 증상: 명령 실행 시 입력 줄이 한 글자씩 자라며 같은 줄이 수십 번 중복 출력된다.
- 원인: 상주 PowerShell 호스트에 PSReadLine이 로드돼 입력 줄을 매 글자 재렌더링하기 때문이다.
- 잘못된 처방: 자식 셸에 `-NoProfile -NonInteractive`를 줘도 소용없다(에코는 바깥 상주 호스트 문제이므로).
- 해결: 세션당 1회, 첫 명령으로 아래를 실행한다. 같은 터미널이 재사용되므로 이후 명령은 깨끗해진다.
  ```
  Remove-Module PSReadLine -ErrorAction SilentlyContinue
  ```
- 근본 해결: 통합 터미널을 `powershell -NoProfile`로 기동하면 PSReadLine이 아예 로드되지 않는다.

## 6. 요약 체크리스트
- [ ] 짧은 코드인가 → `python -c` / `powershell -Command` 인라인
- [ ] 여러 줄이지만 1회성인가 → stdin 파이핑(`... | python -`)
- [ ] 길거나 비-ASCII/검증이 필요한가 → 파일로 생성(`fs_write`+`fs_append`, 청크)
- [ ] 특정 spec용 스크립트는 `.kiro/specs/<feature>/scripts/`에, 범용은 `.kiro/tmp/`에 두었는가
- [ ] 명령 연결에 `&&` 대신 `;` 를 썼는가
- [ ] 실행 전 `py_compile`로 문법을 확인했는가 (저장 시 `python-syntax-check` hook이 자동 검사)
- [ ] 스크립트를 멱등하게 작성하고, 실행 후 결과를 조회로 검증했는가
- [ ] 지저분한 에코/`Exit Code: -1`이 아니라 실제 출력으로 성공을 판단했는가

## 7. 과거 규칙과의 차이 (변경 이력)
- 이전: "워크스페이스에 스크립트 파일 생성 절대 금지, %TEMP%에만 허용".
- 변경: 생성/실행이 안정적으로 동작함을 확인하여, **워크스페이스 내 스크립트 파일 생성을 허용**한다.
  `%TEMP%`는 `fs_write`로 접근 불가하다는 점도 명시.
- 추가: **특정 spec용 스크립트는 `.kiro/specs/<feature>/scripts/`에 배치**하고, 범용 스크래치만
  `.kiro/tmp/`에 둔다.
