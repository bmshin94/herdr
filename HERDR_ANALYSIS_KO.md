# herdr 전수조사 분석 정리 (한국어)

> 작성일: 2026-09-28
> 분석 대상 커밋: `01f25d4` (브랜치 `claude/gallant-gauss-pgttta`)
> 이 문서는 herdr 레포지토리를 전수조사한 결과와, 그로부터 도출한 활용/수익화 방안을 정리한 개인 분석 노트입니다.

---

## 0. 링크 모음

| 구분 | 주소 |
| --- | --- |
| 이 포크 (작업 중인 레포) | https://github.com/bmshin94/herdr |
| 원본(업스트림) 레포 | https://github.com/herdrdev/herdr |
| 공식 웹사이트 | https://herdr.dev |
| 공식 문서 | https://herdr.dev/docs/ |
| 퀵스타트 | https://herdr.dev/docs/quick-start/ |
| 소켓 API 문서 | https://herdr.dev/docs/socket-api/ |
| 플러그인 문서 | https://herdr.dev/docs/plugins/ |
| 마켓플레이스 (런치 예정) | https://herdr.dev/plugins/ |
| 에이전트 스킬 문서 | https://herdr.dev/docs/agent-skill/ |
| AI용 온보딩 가이드 | https://herdr.dev/agent-guide.md |
| 릴리스 바이너리 | https://github.com/herdrdev/herdr/releases |
| Homebrew Formula | https://formulae.brew.sh/formula/herdr |
| X (트위터) | https://x.com/herdrdev |
| 문의 | hey@herdr.dev |

---

## 1. herdr는 무엇인가

한 줄 정의: **코딩 에이전트(AI)들이 살아가는 런타임 = AI 코딩 에이전트 전용 터미널 멀티플렉서.**

tmux/zellij와 같은 계열이지만, 처음부터 "AI 에이전트 여러 개를 동시에 굴리는 상황"을 위해 설계되었다.
공식 슬로건은 `the runtime your coding agents live on`.

### 기본 스펙 (실측)

| 항목 | 값 |
| --- | --- |
| 언어 | Rust 100% |
| 코드 규모 | `.rs` 파일 366개 / 약 266,854줄 |
| 버전 | 0.9.1 |
| 라이선스 | Apache-2.0 (상업적 이용 가능) |
| GitHub 스타 | 41,154 |
| 포크 | 3,163 |
| 열린 이슈 | 370 |
| 레포 생성일 | 2026-03-27 |
| 기본 브랜치 | master |
| 지원 OS | Linux / macOS / Windows(beta) |
| 배포 형태 | 단일 바이너리 (Electron 없음) |
| 지원 에이전트 | 22종 |

### 지원 에이전트 22종

```
amp, antigravity, claude, cline, codex, cursor, devin, droid,
gemini, github-copilot, grok, hermes, kilo, kimi, kiro,
letta, maki, muse, opencode, pi, qodercli, qwen
```

---

## 2. 폴더 구조 (전수조사 결과)

```
herdr/
├── src/                      Rust 본체
│   ├── server/               백그라운드 서버(데몬). PTY와 에이전트 프로세스의 실제 소유자
│   ├── client/               TUI 클라이언트 (화면). 서버와 소켓으로 통신
│   ├── protocol/             wire.rs / endpoint.rs — 서버·클라이언트 와이어 프로토콜
│   ├── api/                  소켓 JSON API (외부 제어 진입점) + 스키마
│   ├── cli/                  herdr pane / agent / workspace ... CLI 구현
│   ├── detect/               ★ 에이전트 상태 감지 엔진
│   │   └── manifests/        에이전트별 감지 규칙 22개 (.toml)
│   ├── integration/          에이전트에 상태 보고 훅을 설치하는 로직
│   │   └── assets/           claude/codex/grok... 각 에이전트용 보고 스크립트(.sh/.ps1/.ts/.py)
│   ├── app/                  state / actions / input 분리된 앱 계층
│   ├── pane/, workspace/     패널·탭·워크스페이스 모델
│   ├── pty/                  의사 터미널(PTY) 처리
│   ├── platform/             OS별 코드 격리 (linux / macos / windows)
│   ├── remote/               SSH 원격 머신 연결
│   ├── persist/              세션 스냅샷 저장·복원
│   ├── ui/, input/, config/  렌더링, 마우스/키보드, 설정
│   └── plugin_command.rs     플러그인 실행 진입점
│
├── skills/herdr/SKILL.md     ★ AI 에이전트용 herdr 사용 설명서(Agent Skill)
├── .agents/skills/           개발용 내부 스킬 (triage / throwaway-repro / pre-release-audit)
├── distribution/
│   ├── install.sh/.ps1/.cmd  설치 스크립트
│   ├── agent-detection/      원격 배포용 감지 카탈로그 (앱 업데이트 없이 규칙 갱신)
│   ├── latest.json           stable 채널 업데이트 매니페스트
│   ├── preview.json          preview 채널 매니페스트
│   └── agent-guide.md        AI가 읽는 온보딩 가이드
├── docs/
│   ├── next/                 다음 릴리스용 문서 초안 (en / ja / zh-cn)
│   ├── preview/              preview 채널 문서 스냅샷 (CI 소유, 수동 편집 금지)
│   └── versions/             0.5.11 ~ 0.9.0 버전별 문서 보관
├── vendor/                   libghostty-vt 등 벤더링된 외부 소스 + 로컬 패치 인덱스
├── tests/                    통합 테스트 (detach_reattach, multi_client, machine_api 등)
├── CLAUDE.md / AGENTS.md     AI 에이전트 작업 규칙 (이 포크에는 페르소나 추가됨)
├── CHANGELOG.md              약 133,917자
└── justfile                  just test / just check / just preview / just release ...
```

---

## 3. 핵심 기능 5가지

### 3.1 에이전트 상태 자동 감지 (킬러 기능)

herdr는 에이전트 내부에 접속하지 않는다. **터미널에 그려진 문자를 정규식으로 읽어서** 상태를 판정한다.

상태 4종 + 1:

| 상태 | 의미 |
| --- | --- |
| `working` | 작업 중 |
| `blocked` | 승인/질문 UI 감지 — 사용자 입력 대기 중 |
| `idle` | 입력 대기 (이미 확인됨) |
| `done` | 완료 (아직 확인 안 됨) |
| `unknown` | 에이전트는 있으나 분류 불가. 완료를 증명하지 않음 |

`src/detect/manifests/claude.toml` 실제 예시:

```toml
[[rules]]
id = "live_turn_working"
state = "working"
priority = 970
region = "bottom_non_empty_lines(12)"   # 화면 하단 12줄만 검사
visible_working = true
any = [
  { line_regex = ['^\s*[⏸⏵].*esc to interrupt(?:\s|·|$)'] },
]
```

설계 포인트:

- `region`으로 검사 영역을 하단으로 좁혀 오탐 방지 (상단 대화 내용이 상태를 사칭하지 못하게)
- `any` = OR 게이트, `not` = 제외 조건 (예: `do you want to proceed?` 가 있으면 working 아님)
- `priority`로 규칙 충돌 해소
- 사용자 스크롤에 영향받는 뷰포트 대신 **detection 소스(하단 버퍼 스냅샷)** 를 사용
- `distribution/agent-detection/` 로 규칙을 원격 배포 → **herdr 업데이트 없이 감지 규칙 갱신 가능**
- 로컬 오버라이드: `~/.config/herdr/agent-detection/<agent>.toml` + `herdr server reload-agent-manifests` 로 핫리로드
- 판정 근거 확인: `herdr agent explain <pane> --json`

정확도 보강 경로가 2단계로 있다:

| 방식 | 설치 | 정확도 |
| --- | --- | --- |
| detect (화면 읽기) | 불필요 | 높음 |
| integration (에이전트에 훅 설치) | 필요 | 더 높음 — 에이전트가 직접 상태 보고 |

### 3.2 디태치 — 터미널을 닫아도 작업이 죽지 않음

서버/클라이언트가 분리되어 있다. 에이전트 프로세스는 **서버**가 소유하고, TUI는 단지 창이다.

```bash
herdr              # 접속
ctrl+b q           # 디태치 (서버는 계속 동작)
herdr              # 재접속 — 그대로 이어짐
herdr server stop  # 세션 종료 (패널 프로세스도 종료됨)
```

- 터미널 창을 닫아도, SSH가 끊겨도 에이전트는 계속 작동
- 서버/머신 재시작 후에는 **레이아웃을 복원**하고 지원 에이전트는 세션 resume 가능
- 단, 원본 프로세스 자체는 재시작을 넘어 살아남지 않음 (공식 문서에 명시)

### 3.3 Agent-native — AI가 herdr를 직접 조종

`skills/herdr/SKILL.md` 를 읽은 에이전트는 CLI로 herdr를 제어한다.

```bash
# 계산: 넓은 패널은 right, 좁거나 긴 패널은 down으로 분할
herdr pane split --current --direction right --cwd "$PWD" --no-focus
herdr agent start reviewer --kind codex --pane w1:p2
herdr agent prompt reviewer "현재 diff를 리뷰하고 실행 가능한 지적만 보고해줘" --wait --timeout 120000
herdr agent read reviewer --source recent-unwrapped --lines 120
```

즉 **AI가 다른 AI를 고용해 병렬로 일을 시키는 멀티에이전트 오케스트레이션**이 CLI 한 줄 단위로 가능하다.

안전장치: 스킬은 먼저 `test "${HERDR_ENV:-}" = 1` 로 herdr 내부 실행 여부를 확인하고,
아니면 즉시 중단한다 (외부에서 남의 세션을 건드리는 것을 방지).

### 3.4 여러 머신을 한 창에서

```bash
herdr --machine <label-or-id> agent list
herdr --machine <label-or-id> agent prompt <name> "..." --wait --timeout 120000
```

- 로컬 + 저장된 SSH 머신을 한 화면에서 통합 관리, 재연결은 독립적
- ID와 에이전트 이름은 **서버 단위 스코프** (두 머신이 모두 `w1:p1`을 가질 수 있음)
- 양쪽 herdr가 API 호환이어야 하고, 포워딩은 서버를 설치/시작/재시작하지 않음

### 3.5 소켓 API + 플러그인

100개가 넘는 소켓 메서드가 점 표기법으로 제공된다.

```
ping, server.stop, server.reload_config, server.reload_agent_manifests
session.snapshot
workspace.create/list/get/focus/rename/move/move_block/report_metadata/close
tab.create/list/get/focus/rename/move/close
pane.split/swap/move/zoom/layout/process_info/neighbor/edges/focus_direction/
     resize/list/current/get/rename/send_text/send_keys/send_input/read/
     graphics.*/report_agent/report_agent_session/report_metadata/
     release_agent/close/wait_for_output
agent.list/get/read/explain/send_keys/prompt/wait/rename/focus/start/view.*
events.subscribe, events.wait
integration.install/uninstall
plugin.link/list/unlink/enable/disable/action.list/action.invoke/log.list/pane.*
worktree.list/create/open/remove
layout.export/apply/set_split_ratio
notification.show, popup.close, client.window_title.*
```

스키마는 바이너리가 직접 출력한다: `herdr api schema --json`, 현재 상태는 `herdr api snapshot`.

플러그인은 `herdr-plugin.toml` 매니페스트 + 실행 파일 조합. 언어 제약 없음(Bash/JS/Lua/Rust 등).
별도 SDK 없이 **herdr CLI 전체가 플러그인 API**이며, `HERDR_BIN_PATH`로 herdr를 호출한다.

---

## 4. 설치 및 사용법

### 4.1 설치

```bash
# macOS / Linux 원라이너
curl -fsSL https://herdr.dev/install.sh | sh

# Homebrew
brew install herdr

# mise
mise use -g herdr

# Windows (PowerShell) — beta
powershell -ExecutionPolicy Bypass -c "irm https://herdr.dev/install.ps1 | iex"

# 소스 빌드 (이 포크 사용 시)
git clone https://github.com/bmshin94/herdr
cd herdr
cargo build --release
```

Windows용 배포물은 `herdr-windows-x86_64.zip` 이며 `herdr.exe` + app-local ConPTY 런타임을 포함한다.
릴리스 에셋은 총 5종: linux-x86_64 / linux-aarch64 / macos-x86_64 / macos-aarch64 / windows-x86_64.zip.

### 4.2 기본 사용

```bash
cd ~/myproject
herdr          # 워크스페이스 없으면 자동 생성. 소켓 관리 불필요
claude         # 또는 codex, gemini, opencode, grok ... 자동 감지됨
```

마우스 우선(mouse-first) TUI:

| 동작 | 방법 |
| --- | --- |
| 패널 포커스 | 클릭 |
| 크기 조절 | 경계선 드래그 |
| 분할 / 새 탭 | 우클릭 메뉴 |
| 복사 | 드래그 후 놓으면 자동 복사 (Ctrl+C 불필요) |
| 단어 선택 | 더블클릭 (두 번째 누른 채 드래그 = 단어 단위 확장) |
| 링크 열기 | Ctrl+클릭 (Ctrl hover 시 밑줄). macOS도 Ctrl+클릭 |

키보드 (prefix = `ctrl+b`, 선택사항):

| 액션 | 키 |
| --- | --- |
| 오른쪽 분할 | `prefix+v` |
| 아래 분할 | `prefix+minus` |
| 새 탭 | `prefix+c` |
| 다음 / 이전 탭 | `prefix+n` / `prefix+p` |
| 워크스페이스 이동 | `prefix+w` |
| 새 워크스페이스 | `prefix+shift+n` |
| 디태치 | `prefix+q` |
| 전체 키 목록 | `prefix+?` |
| 복사 모드 | `prefix+[` |

### 4.3 자주 쓰는 CLI

```bash
herdr --help
herdr status
herdr workspace list
herdr pane list --workspace w1
herdr pane split --current --direction right --cwd "$PWD" --no-focus
herdr pane run w1:p2 "just test"
herdr pane wait-output w1:p2 --match "test result" --timeout 120000
herdr pane read w1:p2 --source recent-unwrapped --lines 120
herdr agent list
herdr agent start reviewer --kind codex --pane w1:p2
herdr agent prompt reviewer "리뷰해줘" --wait --timeout 120000
herdr agent wait reviewer --until blocked --timeout 120000
herdr agent explain w1:p1 --json
herdr api schema --json
herdr api snapshot
herdr --machine <label> agent list
herdr channel set preview && herdr update
```

ID 체계: 워크스페이스 `w1` / 탭 `w1:t1` / 패널 `w1:p1`.
닫힌 ID는 재사용되지 않으며, 패널을 다른 워크스페이스로 옮기면 새 ID가 부여된다.

읽기 소스 4종:

| 소스 | 용도 |
| --- | --- |
| `visible` | 현재 렌더된 뷰포트 |
| `recent` | 최근 출력 (소프트랩 포함) |
| `recent-unwrapped` | 최근 출력, 소프트랩 결합 — 로그/트랜스크립트에 권장 |
| `detection` | 감지에 쓰이는 하단 버퍼 평문 스냅샷 |

### 4.4 AI에게 온보딩 맡기기

```text
Help me understand and set up Herdr.
Read https://herdr.dev/agent-guide.md first, then walk me through it step by step.
```

---

## 5. 플러그인? 스킬? MCP? — 정체 정리

**정답: 셋 다 아니다. herdr는 독립 실행 프로그램(단일 Rust 바이너리 앱)이다.**
다만 이 세 가지를 모두 품고 있어서 혼동하기 쉽다.

| 질문 | 답 | 근거 |
| --- | --- | --- |
| 플러그인인가? | 아니다. herdr는 **플러그인을 받는 호스트**다 | `herdr-plugin.toml` 매니페스트 규격, `plugin.*` 소켓 메서드 |
| 스킬인가? | 아니다. herdr는 **스킬을 제공하는 쪽**이다 | `skills/herdr/SKILL.md` (Claude Skill 포맷) |
| MCP인가? | **아니다.** 자체 소켓 JSON API를 쓴다 | 코드 전체 grep 결과 MCP 서버 구현 없음 |

MCP 관련 언급은 딱 두 군데뿐이었다.

1. `claude.toml` / `qwen.toml` 의 화면 문자열 감지 규칙 (`N MCP tasks still running` 등)
2. `src/detect/mod.rs` 테스트의 더미 경로 `/tmp/mcp/bin/codex`

herdr 소켓 API vs MCP 비교:

| 항목 | herdr 소켓 API | MCP |
| --- | --- | --- |
| 전송 | Unix 소켓 / Windows 명명 파이프 | stdio / HTTP+SSE |
| 포맷 | 자체 JSON (점 표기법 메서드) | JSON-RPC 2.0 기반 MCP 스펙 |
| 스키마 | `herdr api schema --json` | MCP capabilities |
| 인코딩 | JSON + bincode (렌더 프레임) | JSON |

플러그인 매니페스트 예시:

```toml
id = "example.layout"
name = "Layout"
version = "0.1.0"
min_herdr_version = "0.7.0"
platforms = ["linux", "macos", "windows"]

[[build]]
command = ["npm", "ci"]

[[actions]]
id = "apply"
title = "Apply layout"
contexts = ["workspace"]
command = ["node", "dist/apply.js"]

[[events]]
on = "worktree.created"
command = ["herdr", "workspace", "list"]

[[panes]]
id = "board"
title = "Project board"
placement = "overlay"
command = ["herdr-board"]
```

Agent Skill 설치:

```bash
npx skills add herdrdev/herdr --skill herdr -g
```

> 참고: 플러그인 커맨드는 셸을 거치지 않는 argv 배열이며, herdr는 플러그인 코드를 **검수하거나 샌드박싱하지 않는다.**
> 플러그인은 사용자 권한으로 실행되고 전체 herdr CLI를 호출할 수 있으므로, 신뢰하는 저자의 것만 설치해야 한다.

---

## 6. API 토큰이 필요한가

**herdr 자체는 토큰이 전혀 필요 없다. 무료, 가입 없음, 인증 레이어 자체가 없다.**

코드베이스를 `api_key` / `token` / `ANTHROPIC_API_KEY` 로 검색한 결과:

- `parse_api_key()`, `normalize_api_key_alias()` → **키보드 키** 파싱 함수 (`ctrl+h` 등). 인증과 무관
- `normalize_metadata_tokens()` → 사이드바 표시용 메타데이터 토큰. 인증과 무관

이유는 구조적이다. herdr는 AI를 대신 호출하지 않고 터미널만 소유한다.

```
[사용자 API 키] → [Claude Code / Codex CLI] → 이 프로세스를 herdr가 담는다
                         ↑ 토큰은 여기서만 쓰임 (herdr는 관여하지 않음)
```

보안 모델:

| 항목 | 방식 |
| --- | --- |
| 로컬 통신 | Unix 소켓(파일 권한) / Windows 명명 파이프(SDDL) |
| 원격 | SSH 신뢰 모델 (herdr 자체 인증 없음) |
| 인증/인가 | 없음 — OS 권한에 위임 |
| Git 신뢰 | `--trust-repository` 플래그 (워크트리 명령 한정) |

주의점 2가지:

1. 소켓에 접근 가능하면 세션을 완전히 제어할 수 있다 (동일 머신·동일 유저)
2. 플러그인 샌드박싱이 없다

원격(`--machine`)에서 문제가 되는 것은 토큰이 아니라 **프로토콜 버전**(`src/protocol/wire.rs::PROTOCOL_VERSION`) 호환성이다.

---

## 7. 왜 GitHub에서 유명한가 (스타 41,154 / 6개월)

1. **타이밍** — "AI 에이전트 여러 개 동시 운용"이 보편화된 시점에 정확히 그 빈칸으로 들어왔다.
   tmux는 AI를 모르고, IDE는 무겁고 디태치가 안 되고, 생터미널은 창을 닫으면 죽는다.
2. **진짜 아픈 곳** — "AI 5개 돌려놓고 돌아왔더니 1개가 처음 5초에 y/n 물어보고 29분 55초를 놀았다"는 경험.
   슬로건 `never hunt for the stuck one` 이 이 고통을 정확히 겨눈다.
3. **기술 선택이 취향 저격** — Rust 단일 바이너리(Electron 피로감 대안), `curl | sh` 한 줄 설치,
   기존 터미널 그대로 사용, tmux 프리픽스(`ctrl+b`) 호환 + 마우스 우선.
   결과적으로 **tmux 유저와 tmux를 못 쓰는 사람을 동시에** 확보했다.
4. **에이전트 커버리지** — 22종 지원 + 원격 감지 카탈로그로 신규 에이전트도 앱 업데이트 없이 추가.
   "내가 쓰는 AI는 지원 안 되겠지"가 성립하지 않는다.
5. **개발 속도와 완성도** — CHANGELOG 13만 자, 6개월에 0.5.x→0.9.1,
   문서 3개국어(en/ja/zh-cn) + 버전별 문서 전량 보관, stable/preview 2채널 릴리스 자동화,
   Homebrew·Nix·mise 패키징.
6. **AI 시대 바이럴 구조** — `agent-guide.md`와 Agent Skill 덕분에 **AI가 herdr를 학습하고 사용자에게 추천**한다.
   AI가 AI 도구를 전파하는 확산 경로.

---

## 8. 로컬 에이전트 구축에 도움이 되는가

결론: **매우 도움이 된다. 단, 역할을 정확히 구분해야 한다.**

```
herdr ≠ Ollama / LM Studio / vLLM  (모델 추론 엔진이 아니다)
herdr = 그 위에서 도는 에이전트들의 작업장·런타임
```

### 도움 되는 부분

| 항목 | 내용 |
| --- | --- |
| 오케스트레이션 레이어 | 프로세스 생명주기, 상태 판정, 출력 캡처, 복구를 이미 해결해둠 |
| 이벤트 기반 대기 | `agent.wait`는 서버 소유 + 이벤트 구동. 패널 점유자를 pin해서 교체된 에이전트가 wait를 만족시키지 못함 |
| 감지 엔진 재사용 | 22종 규칙 + `agent explain --json` + 커스텀 TOML + 핫리로드 |
| 상태 보고 API | `pane.report_agent`, `pane.report_agent_session`, `pane.report_metadata` 로 자체 에이전트도 상태 보고 가능 |
| 설계 교과서 | 상태/런타임 분리, 순수 렌더, 영속성, 하위호환 계약, 성능 곱셈 경로 |
| 멀티머신 | `--machine` 으로 GPU 서버 등 원격 에이전트까지 CLI 제어 |

설계 원칙(CLAUDE.md/AGENTS.md에서 정리):

- **상태와 런타임 분리** — `AppState`는 순수 데이터, `PaneState`와 `PaneRuntime`을 분리.
  `AppState::test_new()` / `Workspace::test_new()` 로 PTY 없이 테스트 가능
- **렌더는 순수** — `compute_view()`가 기하/변경을 담당, `render()`는 `&AppState`를 받아 그리기만 함
- **God object 금지** — 모듈이 커지면 분할 (`app/`은 state/actions/input으로 이미 분리)
- **플랫폼 코드 격리** — OS별 코드는 `src/platform/<os>.rs`, 코어에 `#[cfg(target_os)]` 금지
- **감지 디커플링** — 디텍터는 화면 스냅샷만 읽고 파서/뷰포트 상태를 만지지 않음
- **성능은 곱셈 경로로** — 렌더/레이아웃/PTY 파싱/감지/클라이언트 팬아웃은
  (바이트·이벤트·렌더) × (패널·탭·워크스페이스) × (접속 클라이언트)로 증폭.
  `just bench-render-scale` 로 패널 1개 vs 15개 이상 스케일링 델타 확인
- **하위호환은 계약** — 명명된 코덱은 불변, 새 JSON 필드는 optional,
  새 enum 값은 `Unknown` 폴백 필요, 프리즌 픽스처/bincode 다이제스트/와이어 태그 테스트로 강제.
  Generation 1이 Local·SSH·Cloud 호환 floor

### 도움 안 되는 부분 (herdr 범위 밖)

로컬 LLM 추론, 웹 UI, 멀티유저·권한 관리, 인증/인가, 에이전트 메모리/RAG,
MCP 통합, 작업 큐/스케줄러 — 전부 없음. (→ 이것이 곧 수익화 빈칸이다.)

### 권장 아키텍처

```
[내가 만들 오케스트레이터 / 대시보드]   React · Node · (선택) PHP
              ↕ herdr 소켓 API (JSON)
[herdr 서버]  프로세스 생명주기 · 상태감지 · 출력캡처 · 영속성
              ↕
[Claude Code] [Codex] [Ollama] [커스텀 에이전트]
```

핵심: **바닥부터 만들지 말고 herdr를 런타임으로 쓰고 그 위에 로직만 얹는다.**

---

## 9. React / PHP로 만들 수 있는가

- **herdr 본체를 React/PHP로 재구현: 비추천.** PTY 저수준 제어, 초당 수만 바이트 파싱,
  백그라운드 데몬, 단일 바이너리 배포, Windows ConPTY, 터미널 에뮬레이터(libghostty-vt 벤더링) —
  26만 줄 Rust를 다시 쓰는 것은 비합리적이다.
- **herdr 위에 얹는 것: 완전히 가능하다.** 소켓 JSON API가 열려 있어 어떤 언어든 클라이언트가 될 수 있다.

### 방법 1 — React 웹 대시보드 (권장)

```
React (브라우저)  ──WebSocket──▶  Node 브리지 (얇게)  ──Unix 소켓──▶  herdr 서버
```

Node 브리지 골격:

```js
import net from 'node:net';
import { WebSocketServer } from 'ws';

const SOCKET = process.env.HERDR_SOCKET_PATH;
const wss = new WebSocketServer({ port: 8080 });

wss.on('connection', (ws) => {
  const sock = net.connect(SOCKET);
  sock.on('data', (buf) => ws.send(buf.toString()));
  ws.on('message', (msg) => sock.write(msg + '\n'));

  // 순서 중요: 먼저 구독하고 ACK를 받은 뒤 스냅샷을 요청해야 부트스트랩 갭이 없다
  sock.write(JSON.stringify({ method: 'events.subscribe' }) + '\n');
  sock.write(JSON.stringify({ method: 'session.snapshot' }) + '\n');
});
```

React 쪽:

```jsx
function AgentBoard() {
  const [agents, setAgents] = useState([]);
  useEffect(() => {
    const ws = new WebSocket('ws://localhost:8080');
    ws.onmessage = (e) => {
      const msg = JSON.parse(e.data);
      if (msg.result?.agents) setAgents(msg.result.agents);
    };
    return () => ws.close();
  }, []);
  return agents.map(a => <AgentCard key={a.pane_id} {...a} />);
}
```

킬러 기능: **폰에서 `blocked` 상태 에이전트에 바로 답변** → AI 유휴 시간 제거.

> 보안: herdr에 인증이 없으므로 브리지 계층에서 **토큰 인증 + TLS(WSS)** 를 반드시 직접 구현해야 한다.

### 방법 2 — herdr 플러그인 (Node.js)

```js
const { execFileSync } = require('node:child_process');
const herdr = process.env.HERDR_BIN_PATH;
const agents = JSON.parse(execFileSync(herdr, ['agent', 'list', '--json'])).result.agents;
const blocked = agents.filter(a => a.state === 'blocked');
if (blocked.length) { /* Slack 알림 등 */ }
```

### 방법 3 — PHP (배치·집계·리포팅에 적합)

```php
$socket = stream_socket_client('unix://' . getenv('HERDR_SOCKET_PATH'), $errno, $errstr, 5);
fwrite($socket, json_encode(['method' => 'agent.list']) . "\n");
$response = json_decode(fgets($socket), true);
foreach ($response['result']['agents'] as $agent) {
    if ($agent['state'] === 'blocked') { /* 알림 · DB 기록 */ }
}
```

PHP는 Laravel 기반 팀 SaaS 백엔드, 작업 이력 DB, 관리자 페이지, 멀티머신 중앙 집계에 잘 맞는다.
실시간 스트리밍은 Node가 유리하므로 역할을 분리하는 편이 좋다.

### 권장 스택

```
React + Tailwind (프론트)
Node.js + TypeScript (브리지 / 실시간)
(선택) Laravel/PHP (이력 DB · 리포팅)
herdr (Rust 코드 수정 없이 런타임으로 사용)
```

---

## 10. 수익화 아이디어

### 10.1 법적 기반

Apache-2.0 이므로 상업적 이용·수정·재배포·사설 사용이 모두 허용되고 특허 사용권까지 부여된다(GPL보다 안전).
의무는 저작권 고지 + 라이선스 사본 포함 + 변경 사항 명시. 단, "herdr" 상표 사용은 별개 이슈이므로
독자 브랜드로 가는 것이 안전하다.

### 10.2 빈칸(Gap) 분석

| # | 빈칸 | 수요 | 난이도 | 수익 |
| --- | --- | --- | --- | --- |
| 1 | MCP 서버 브리지 | 매우 높음 | 낮음 | 간접 |
| 2 | 웹 / 모바일 UI | 매우 높음 | 중 | 직접 |
| 3 | 팀 협업 · 멀티유저 | 높음 | 높음 | 직접 |
| 4 | 작업 큐 · 스케줄러 | 매우 높음 | 중 | 직접 |
| 5 | 비용 추적 · 분석 | 매우 높음 | 낮음 | 직접 |
| 6 | 알림 통합 (Slack 등) | 높음 | 매우 낮음 | 간접 |
| 7 | 인증 · 감사 로그 | 높음 | 높음 | 직접 |
| 8 | 매니지드 호스팅 | 높음 | 매우 높음 | 직접 |

### 10.3 아이디어 상세

#### 1) herdr MCP 서버 브리지 — 최우선 추천

herdr 소켓 API를 MCP 서버로 노출해 Claude Desktop / Cursor 등 모든 MCP 클라이언트가 herdr를 도구로 쓰게 한다.

```
Claude Desktop ──MCP──▶ herdr-mcp ──소켓──▶ herdr 서버
```

툴 설계: `herdr_list_agents`, `herdr_spawn_agent`, `herdr_prompt_agent`,
`herdr_read_output`, `herdr_wait_blocked`, `herdr_pane_run`.

가치: Claude Desktop은 터미널을 쓸 수 없는데, 이 브리지가 있으면
"데스크톱에서 대화하면서 로컬 Claude Code 5개를 병렬로 굴리기"가 가능해진다.

- 스택: TypeScript + `@modelcontextprotocol/sdk` + `node:net`
- MVP 1~2주, 1,000~2,000줄
- 수익: OSS 명성 → 취업/컨설팅, Pro 버전 $5~15/월, GitHub Sponsors, 컨설팅 $100~300/시

#### 2) 웹 / 모바일 대시보드 (SaaS) — 수익성 1위

에이전트 상태 보드 + 원격 프롬프트 + 푸시 알림 + 이력/통계.
킬러 기능은 **폰으로 blocked 답변**(AI 유휴 시간 제거).

가격안: Free $0 (1머신, 알림 3/일) / Pro $9월 / Team $19유저월 / Enterprise 문의.
유료 100명 = 월 약 $900, 1,000명 = 월 약 $9,000.
MVP 4~6주, 프로덕션 3개월. React + Node + Postgres + Stripe.

#### 3) 플러그인 마켓플레이스 선점

공식 문서가 "마켓플레이스 런치 시 등재되도록 레포에 태그하라"고 안내 중 = **아직 런치 전**.

| 플러그인 | 가격 |
| --- | --- |
| Cost Tracker (토큰·비용 추적, 예산 알림) | $5/월 |
| Notify Pro (Slack/Discord/Telegram/이메일) | $3/월 |
| Layout Manager (프로젝트별 레이아웃) | 무료 — 유입용 |
| Scheduler (정시 에이전트 실행) | $7/월 |
| Session Recorder (작업 기록 → 리포트) | $5/월 |
| Git Flow (PR 생성까지 자동화) | $9/월 |
| Test Runner (변경 감지 → 자동 테스트) | $5/월 |
| Theme Pack | $9 1회 |

전략: 무료 1개로 유입 → Pro 번들 $15/월 전환. 플러그인 1개당 3~7일(JS).

#### 4) 에이전트 오케스트레이션 SaaS

herdr를 엔진으로 쓰는 "AI 개발팀 관리 플랫폼". YAML 워크플로우 + DAG 의존성 +
실패 재시도/롤백 + 모델별 비용 최적화 + 팀 승인 워크플로우.

```yaml
steps:
  - agent: claude
    task: "기능 구현"
  - agent: codex
    task: "테스트 작성"
    depends_on: [1]
  - agent: gemini
    task: "코드 리뷰"
    depends_on: [1, 2]
```

가격: Starter $29 / Team $99 / Business $299. 팀 50개 × $99 = 월 약 $4,950.
MVP 2~3개월. Node 또는 Go + React + Postgres + Redis.

#### 5) 비용 추적 & 분석 도구

`events.subscribe` 로 에이전트 활동을 추적하고 각 CLI 사용량 로그 + 모델 단가로 집계.
프로젝트별·에이전트별 비용, 예산 알림, 절감 제안("docs 작업은 저가 모델로").

가격: Free / Pro $7월 / Team $15유저월. "월 $7로 $50 절약" → ROI가 명확해 전환율이 높다.
MVP 2~3주. Node + React + SQLite/Postgres.

#### 6) 교육 · 콘텐츠 (진입 난이도 최저, 한국어 자료 거의 없음)

| 상품 | 가격 | 예상 |
| --- | --- | --- |
| YouTube 시리즈 | 무료 | 광고 + 유입 |
| 유데미 강의 | ₩50,000 | 500명 = 약 ₩2,500만 |
| 전자책 | ₩20,000 | 1,000부 = 약 ₩2,000만 |
| 유료 뉴스레터 | $5/월 | 200명 = 월 약 $1,000 |
| 기업 워크샵 | ₩200만/회 | 월 2회 = 약 ₩400만 |

다른 제품의 마케팅 채널로도 동시에 작동한다.

#### 7) 컨설팅 / SI

포지셔닝: "AI 에이전트 개발 환경 구축 전문가".
셋업 컨설팅 ₩300만 / 워크플로우 설계 ₩500만 / 커스텀 플러그인 ₩200~800만 /
팀 온보딩 ₩200만 / 유지보수 리테이너 월 ₩150만. 초기 자본 0, 즉시 현금화.

#### 8) 매니지드 호스팅

클라우드에서 herdr + 에이전트를 관리형으로 제공. Hobby $19 / Pro $49 / Team $149.
200명 × $49 = 월 약 $9,800. 인프라 비용·API 키 위탁 보안·기존 개발환경 서비스와의 경쟁이 리스크.
4~6개월, K8s + Docker + Terraform.

### 10.4 실행 로드맵

**Phase 1 — 씨 뿌리기 (1~2개월)**
1. herdr MCP 서버 (TypeScript, 2주) → OSS 공개
2. 한국어 콘텐츠 시작 (블로그/유튜브) → 한국어 1인자 포지션 선점
3. 무료 플러그인 1개 (Layout Manager) → 생태계 진입
목표: 인지도 + 포트폴리오

**Phase 2 — 첫 수익 (3~4개월)**
4. 비용 추적 도구 출시 (Free / Pro $7)
5. 컨설팅 시작 (Phase 1의 명성 활용)
목표: 월 $500~2,000

**Phase 3 — 스케일 (5~10개월)**
6. 웹 대시보드 SaaS (메인 제품)
7. 유료 플러그인 번들 $15/월
8. 유데미 강의
목표: 월 $3,000~10,000

**Phase 4 — 확장 (12개월+)**
9. 오케스트레이션 플랫폼 또는 매니지드 호스팅
목표: 월 $10,000+

### 10.5 리스크

| 리스크 | 대응 |
| --- | --- |
| herdr 팀이 직접 구현 (웹 UI·비용추적) | 틈새(한국 시장, 특정 워크플로우)에 집중 |
| API 변경 | Generation 1이 호환 floor로 보장되고 프리즌 픽스처로 강제 → 비교적 안전 |
| 시장 규모 | 스타 4만 ≠ 유료 4만. 전환율 0.5~2% 가정 |
| AI CLI 변화로 감지 규칙 파손 | 감지 규칙 유지보수는 herdr 팀 책임 영역 |
| 보안 사고 | herdr에 인증 없음 → 자체 레이어에 토큰 + TLS 필수 |

### 10.6 최종 추천 3선

1. **herdr MCP 서버** (2주, 무료 공개) — 빈칸 확실, 난이도 낮음, 임팩트 큼
2. **비용 추적 도구** (3주, $7/월) — 첫 수익, ROI 명확
3. **웹 대시보드** (6주, $9~19/월) — 메인 제품, React 강점 활용

각 단계가 다음 단계의 마케팅으로 이어지는 순서다.

---

## 11. 이 포크에 대한 참고사항

- 현재 포크: `https://github.com/bmshin94/herdr` (기준 커밋 `01f25d4`)
- 업스트림 대비 차이: `CLAUDE.md` 에 페르소나 가이드 추가 (커밋 `610a031`, 353줄 추가)
- 업스트림 규칙(`AGENTS.md` / `CONTRIBUTING.md`)상, `.github/MAINTAINERS` 에 없는 계정은
  **외부 컨트리뷰터**로 분류된다. 초청 없는 구현 PR은 자동 클로즈되며,
  이슈는 재현 가능한 실제 버그에 한해 지정 템플릿으로만 제출할 수 있다.
- 따라서 이 포크는 개인 실험·학습·파생 제품 개발용으로 쓰는 것이 적절하다.
- 업스트림 규칙 요약(참고): 커밋은 소문자 conventional commit, 이모지 금지, AI co-author 라인 금지,
  이슈 연결 시 본문에 `refs #<번호>` (`fixes` 등 클로징 키워드 금지),
  커밋 전 `just check` 실행, 릴리스 파일(`docs/next/CHANGELOG.md` 등)은 일반 작업에서 수정하지 않음.

---

## 12. 한 장 요약

| 질문 | 답 |
| --- | --- |
| 무엇인가 | AI 코딩 에이전트 전용 터미널 멀티플렉서 (Rust 단일 바이너리) |
| 어떨 때 쓰나 | AI 2개 이상 동시 운용 / 디태치 필요 / SSH 작업 / 멀티에이전트 협업 |
| 정체 | 독립 앱 (플러그인 아님, 스킬 아님, MCP 아님 — 단 셋 다 품고 있음) |
| 토큰 | 불필요. herdr 자체는 무료·인증 없음 |
| 왜 유명한가 | 타이밍 + 실제 고통 해결 + Rust 단일 바이너리 + 22종 지원 + 개발속도 + AI 바이럴 |
| 로컬 에이전트에 도움? | 매우 도움. 단 추론 엔진이 아니라 런타임 레이어 |
| React/PHP 가능? | 본체 재구현은 비추천, 소켓 API 위에 얹는 것은 완전히 가능 |
| 수익화 | MCP 브리지 → 비용 추적 → 웹 대시보드 순서 추천 |

---

*이 문서는 herdr 레포지토리(커밋 `01f25d4`) 전수조사를 기반으로 작성되었습니다.*
