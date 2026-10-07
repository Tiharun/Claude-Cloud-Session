# Chrome DevTools MCP 완전 가이드 (초보자용)

> **한 줄 요약**
> Chrome DevTools MCP(`chrome-devtools-mcp`)는 **Google Chrome DevTools 팀이 만든 무료(Apache-2.0) 도구**로, AI 코딩 에이전트(Claude Code, Cursor, Copilot, Gemini CLI 등)가 **Chrome 브라우저를 직접 열고, 보고, 클릭·입력하고, 스크린샷을 찍고, 성능을 측정**하게 해 줍니다.
> Claude Code에서는 아래 한 줄로 설치하고, "localhost:3000 열어서 로그인 버튼 눌러보고 스크린샷 찍어줘"처럼 말로 시키면 됩니다.
>
> ```bash
> claude mcp add chrome-devtools --scope user -- npx -y chrome-devtools-mcp@latest
> ```

- 공식 GitHub: <https://github.com/ChromeDevTools/chrome-devtools-mcp>
- 공식 문서 홈: <https://developer.chrome.com/docs/devtools/agents> (`plugin.json`의 homepage)
- npm 패키지: <https://www.npmjs.com/package/chrome-devtools-mcp>
- 이 문서 기준 버전: **v1.10.1** (2026-09-23 릴리스, main 브랜치 커밋 `cf2eab9`)
- 작성일: 2026-10-05

---

## 목차

0. [이 문서를 어떻게 검증했나](#0-이-문서를-어떻게-검증했나)
1. [무엇인가: 들은 내용이 맞나?](#1-무엇인가-들은-내용이-맞나)
2. [동작 원리](#2-동작-원리)
3. [할 수 있는 일: 도구 목록](#3-할-수-있는-일-도구-목록)
4. [설치 전 준비물](#4-설치-전-준비물)
5. [설치하기](#5-설치하기)
   - [5.5 Claude 데스크톱 앱 Code 탭에서 쓰기](#55-claude-데스크톱-앱-code-탭에서-쓰기)
6. [설치 확인과 첫 사용](#6-설치-확인과-첫-사용)
7. [사용법: 이렇게 말하면 됩니다](#7-사용법-이렇게-말하면-됩니다)
8. [응용: 실전 시나리오](#8-응용-실전-시나리오)
9. [응용: 설정 옵션 (플래그)](#9-응용-설정-옵션-플래그)
10. [응용: 내 Chrome에 연결하기](#10-응용-내-chrome에-연결하기)
11. [응용: 터미널 CLI로 쓰기](#11-응용-터미널-cli로-쓰기)
12. [보안과 개인정보](#12-보안과-개인정보)
13. [업데이트하기](#13-업데이트하기)
14. [삭제하기](#14-삭제하기)
15. [문제 해결](#15-문제-해결)
16. [토큰 비용](#16-토큰-비용)
17. [자주 묻는 질문](#17-자주-묻는-질문)
18. [출처](#18-출처)

---

## 0. 이 문서를 어떻게 검증했나

| 대상 | 확인 방법 | 결과 |
|---|---|---|
| GitHub 저장소 | `git clone` 후 README, `docs/` 10개 문서, 번들 스킬 7개, `SECURITY.md`, `CHANGELOG.md`, 매니페스트(`plugin.json`, `mcp.json`, `.claude-plugin/`)를 읽음 | ✅ |
| MCP 서버 실제 동작 | npm에서 v1.10.1을 설치하고 **Chromium을 실제로 띄워서** 페이지 열기 → 구조 읽기 → 입력 → 클릭 → 대기 → 스크린샷 → 콘솔·네트워크 확인 → JS 실행 → 성능 측정 → Lighthouse → 네트워크 느리게 하기까지 실행 | ✅ 모두 성공 |
| Claude Code 등록·확인·삭제 | `claude mcp add` → `list`(Connected 확인) → `get` → `remove` 실행 | ✅ |
| Claude Code 플러그인 방식 | GitHub에서 바로 설치하는 것은 **이 작업 환경의 네트워크 정책 때문에 실패**(하위 모듈을 `chromium.googlesource.com`에서 받아야 하는데 차단됨). 로컬 사본으로 설치, `details`, 삭제는 성공 | ⚠️ 부분 확인 |
| CLI (`chrome-devtools`) | `start` → `status` → `new_page` → `take_screenshot` → `list_pages` → `stop` 실행 | ✅ |
| 데스크톱 앱 Code 탭 | Claude Code 공식 문서(Desktop application)로 확인. **앱을 직접 실행해 보지는 않음** | ⚠️ 문서 기준 |
| 외부 사이트 접속 | 작업 환경이 외부 웹사이트를 막아서 **내가 만든 로컬 테스트 페이지로만** 시험 | ⚠️ 여러분 PC에서는 일반 사이트 가능 |

시험은 **Linux 클라우드 환경**에서 했습니다. **Windows 관련 내용은 공식 문서 기준**이며 Windows PC에서 직접 실행해 보지는 않았습니다.

---

## 1. 무엇인가: 들은 내용이 맞나?

### 1.1 "구글에서 직접 만든 것"이 맞나?

**맞습니다.** 근거는 다음과 같습니다.
- GitHub 조직이 `ChromeDevTools`이고, 플러그인 매니페스트의 작성자가 **"Google Chrome"**, 소유자가 **"Chrome DevTools Team (devtools-dev@chromium.org)"** 입니다.
- 소스 코드 머리말에 `Copyright 2026 Google LLC`라고 적혀 있습니다.
- npm 패키지 관리자에 Google 계정(`google-wombot`)이 포함되어 있습니다.
- 공식 홈페이지가 `developer.chrome.com`입니다.
- 라이선스는 **Apache-2.0** 오픈소스입니다. 첫 커밋은 2025-09-11이고 현재까지 60여 차례 릴리스됐습니다.

### 1.2 "AI가 브라우저를 열어 직접 보고 조작하고 캡처한다"가 맞나?

**대부분 맞고, 세 가지는 보충이 필요합니다.**

| 들은 내용 | 실제 |
|---|---|
| AI가 브라우저를 연다 | ✅ AI가 도구를 처음 쓰는 순간 **Chrome을 자동으로 실행**합니다 |
| 직접 본다 | ✅ 화면을 두 가지 방식으로 봅니다: ① **스냅샷**(페이지 구조를 텍스트로 읽기, 주로 이 방식) ② **스크린샷**(이미지) |
| 조작한다 | ✅ 클릭, 입력, 폼 채우기, 키 누르기, 드래그, 파일 업로드, 팝업 처리 |
| 자유롭게 캡처한다 | ✅ 화면 전체, 특정 요소, 전체 페이지 스크롤 캡처. 실험 기능으로 동영상 녹화도 가능 |
| "구글 창" | ⚠️ "구글 검색창"이 아니라 **Chrome 브라우저**입니다. 공식 지원은 Google Chrome과 Chrome for Testing이고, 다른 Chromium 계열 브라우저는 보장하지 않습니다 |
| 내 Chrome을 쓴다? | ⚠️ 기본값은 **별도의 전용 프로필**로 새 Chrome을 띄웁니다. 평소 쓰는 Chrome(로그인 상태 포함)에 연결하려면 따로 설정해야 합니다 (10장) |
| 그냥 조작 도구? | ⚠️ 원래 목적은 **웹 개발자용 디버깅·성능 분석**입니다. 콘솔 에러, 네트워크 요청, 성능 측정(LCP 등), Lighthouse 감사, 메모리 누수 분석이 핵심 기능입니다 |

### 1.3 쉬운 비유

AI에게 **원격 조종이 가능한 Chrome과 개발자 도구(F12 화면)를 통째로 쥐여 주는 것**입니다. 사람이 F12를 눌러 보는 콘솔, 네트워크, 성능 탭을 AI가 직접 열어 보고, 마우스와 키보드도 대신 움직입니다.

---

## 2. 동작 원리

```
 ┌──────────────┐  ① "로그인 눌러봐"    ┌──────────────────────┐
 │  사용자      │ ───────────────────► │ AI 에이전트            │
 └──────────────┘                      │ (Claude Code 등)      │
                                       └──────────┬───────────┘
                                ② MCP 도구 호출 (click, take_screenshot …)
                                                  ▼
                                       ┌──────────────────────┐
                                       │ chrome-devtools-mcp  │  ← 내 PC에서 실행되는
                                       │ (Node.js 프로그램)    │    작은 서버
                                       └──────────┬───────────┘
                         ③ Puppeteer + Chrome DevTools Protocol(CDP)
                                                  ▼
                                       ┌──────────────────────┐
                                       │ Chrome 브라우저        │
                                       └──────────────────────┘
```

- **MCP(Model Context Protocol):** AI가 외부 도구를 쓰도록 연결하는 표준 규격입니다. 특정 AI 회사 전용이 아니라서 여러 AI 도구에서 똑같이 쓸 수 있습니다.
- **Puppeteer:** Google이 만든 Chrome 자동화 라이브러리입니다. 클릭한 뒤 결과가 나올 때까지 **자동으로 기다려 줘서** 동작이 안정적입니다.
- **CDP(Chrome DevTools Protocol):** 개발자 도구(F12)가 Chrome과 통신하는 내부 규약입니다.
- **브라우저 실행 시점:** MCP 서버에 연결만 해서는 Chrome이 뜨지 않습니다. AI가 브라우저가 필요한 도구를 **처음 호출할 때** 자동으로 실행됩니다.
- **AI가 페이지를 "보는" 방법:** 주로 `take_snapshot`을 씁니다. 페이지를 접근성 트리(화면 낭독기가 읽는 구조)로 바꿔서 요소마다 `uid`라는 번호를 붙여 줍니다. AI는 그 번호로 클릭하거나 입력합니다. 이미지보다 빠르고 토큰이 적게 듭니다.

실제 테스트에서 AI가 받은 스냅샷 예시입니다.
```
uid=1_0 RootWebArea "테스트 로그인" url="http://localhost:8765/"
  uid=1_1 heading "테스트 로그인 페이지" level="1"
  uid=1_3 textbox "이름 "
  uid=1_4 button "로그인"
```
→ AI가 `fill(uid=1_3, "홍길동")`, `click(uid=1_4)`를 호출한 뒤 `take_screenshot`으로 찍은 결과:

![AI가 이름을 입력하고 로그인 버튼을 누른 뒤 찍은 스크린샷](assets/chrome-devtools-mcp-demo.png)

---

## 3. 할 수 있는 일: 도구 목록

기본 설정에서는 **30개** 도구가 켜집니다 (실측). 나머지는 옵션 플래그로 켭니다.

### 3.1 기본으로 켜지는 도구 (30개)

| 분류 | 도구 | 하는 일 |
|---|---|---|
| **입력 (9)** | `click` | 요소 클릭 (더블클릭 가능) |
| | `fill` | 입력칸에 글자 넣기, 드롭다운 선택 |
| | `fill_form` | 폼의 여러 칸을 한 번에 채우기 |
| | `type_text` | 포커스된 입력칸에 키보드로 타이핑 |
| | `press_key` | 키나 단축키 누르기 (예: `Control+A`, `Enter`) |
| | `hover` | 마우스 올리기 |
| | `drag` | 끌어다 놓기 |
| | `upload_file` | 파일 업로드 |
| | `handle_dialog` | `alert`/`confirm` 같은 팝업 수락·거절 |
| **이동 (6)** | `new_page` | 새 탭 열기 |
| | `navigate_page` | URL 이동, 뒤로, 앞으로, 새로고침 |
| | `list_pages` / `select_page` / `close_page` | 탭 목록, 탭 선택, 탭 닫기 |
| | `wait_for` | 특정 글자가 나타날 때까지 대기 |
| **에뮬레이션 (2)** | `emulate` | 느린 네트워크(예: Slow 3G), CPU 느리게, 위치, 다크모드, 모바일 화면 흉내 |
| | `resize_page` | 창 크기 변경 |
| **성능 (3)** | `performance_start_trace` / `performance_stop_trace` | 성능 기록. LCP, INP, CLS 등 Core Web Vitals 분석 |
| | `performance_analyze_insight` | 성능 문제 항목을 자세히 분석 |
| **네트워크 (2)** | `list_network_requests` / `get_network_request` | 요청 목록, 요청·응답 헤더와 본문 |
| **디버깅 (7)** | `take_snapshot` | 페이지 구조를 텍스트로 읽기 (요소 `uid` 포함) |
| | `take_screenshot` | 스크린샷 (화면, 특정 요소, 전체 페이지. PNG/JPEG/WebP) |
| | `evaluate_script` | 페이지 안에서 JavaScript 실행 |
| | `list_console_messages` / `get_console_message` | 콘솔 로그와 에러 (소스맵 적용된 스택 트레이스) |
| | `get_css_styles` | 요소에 적용된 CSS 규칙 |
| | `lighthouse_audit` | Lighthouse 감사: 접근성, SEO, 모범 사례, 에이전트 브라우징 (성능 제외) |
| **메모리 (1)** | `take_heapsnapshot` | 메모리 스냅샷 저장 |

### 3.2 옵션으로 켜는 도구

| 켜는 방법 | 추가되는 도구 |
|---|---|
| `--memoryDebugging` | 메모리 누수 분석 도구 (스냅샷 비교, 누가 객체를 붙잡고 있는지 추적 등). 실측 결과 도구가 30개에서 42개로 늘어남 |
| `--categoryExtensions` | Chrome 확장 프로그램 설치, 목록, 새로고침, 실행, 삭제 |
| `--experimentalScreencast` | `screencast_start`, `screencast_stop`: 동영상 녹화 (**ffmpeg 필요**) |
| `--experimentalVision` | `click_at(x, y)`: 좌표로 클릭 (화면을 보고 좌표를 고를 수 있는 모델용) |
| `--categoryPwa` | PWA 설치, 실행, 삭제 |
| `--categoryExperimentalWebmcp` | WebMCP 도구 (Chrome 150 이상 + 플래그 필요) |
| `--categoryExperimentalThirdParty` | 페이지가 직접 제공하는 개발자 도구 실행 |

### 3.3 slim 모드 (도구 3개)

`--slim` 옵션을 주면 `navigate`(이동), `evaluate`(JS 실행), `screenshot`(캡처) **3개만** 남습니다. 간단한 작업만 할 때 토큰을 크게 아낄 수 있습니다 (16장).

---

## 4. 설치 전 준비물

| 준비물 | 버전 | 확인 명령 |
|---|---|---|
| Node.js | LTS 버전 | `node --version` |
| npm | Node.js에 포함 | `npm --version` |
| Google Chrome | 현재 안정 버전 이상 | Chrome 메뉴 → 도움말 → Chrome 정보 |
| AI 클라이언트 | Claude Code 등 | `claude --version` |

Node.js가 없으면 <https://nodejs.org>에서 **LTS** 버전을 설치하세요. 설치 후 터미널(PowerShell)을 **새로 열어야** 인식됩니다.

---

## 5. 설치하기

### 어떤 방법을 고를까?

| 방법 | 설치되는 것 | 추천 대상 |
|---|---|---|
| **A. `claude mcp add`** | MCP 서버만 (도구 30개) | **가장 간단. 처음 쓰는 분께 추천** |
| B. Claude Code 플러그인 | MCP 서버 + 사용법 스킬 7개 | AI가 도구를 더 능숙하게 쓰길 원할 때 |
| C. 다른 AI 도구 | 도구별로 다름 | Cursor, VS Code, Gemini CLI 등 |

> 🖥️ **Claude 데스크톱 앱의 Code 탭을 쓰시나요?** A나 B로 설치하면 Code 탭에서도 **그대로** 쓸 수 있습니다. 데스크톱 앱 화면만으로 설치하는 방법은 [5.5절](#55-claude-데스크톱-앱-code-탭에서-쓰기)을 보세요.

> ⚠️ **A와 B를 동시에 하지 마세요.** 같은 서버가 두 번 등록됩니다. 공식 문서도 "플러그인을 설치하기 전에 기존 MCP 설치를 먼저 제거하라"고 안내합니다.

---

### 방법 A. `claude mcp add` (추천, 실제 실행 확인)

터미널(Windows는 PowerShell)에서 실행합니다.
```bash
claude mcp add chrome-devtools --scope user -- npx -y chrome-devtools-mcp@latest
```

실제 실행 결과:
```
Added stdio MCP server chrome-devtools with command: npx -y chrome-devtools-mcp@latest to user config
```

각 부분의 의미:

| 부분 | 의미 |
|---|---|
| `chrome-devtools` | Claude Code 안에서 부를 서버 이름 |
| `--scope user` | 내 **모든 프로젝트**에서 사용. `project`로 하면 현재 프로젝트의 `.mcp.json`에 저장돼서 팀과 공유됨. 생략하면 `local`(현재 프로젝트, 나만) |
| `--` | 여기부터는 Claude가 아니라 실행할 명령이라는 구분자 |
| `npx -y chrome-devtools-mcp@latest` | 실행할 때마다 **최신 버전**을 받아서 실행. `-y`는 설치 확인 질문 자동 수락 |

> 💡 **Windows에서 연결이 안 되면** `cmd /c`를 앞에 붙여서 다시 등록하세요. 공식 문서의 Windows 문제 해결법입니다 (15장).
> ```powershell
> claude mcp remove chrome-devtools --scope user
> claude mcp add chrome-devtools --scope user -- cmd /c npx -y chrome-devtools-mcp@latest
> ```

공식 README에 적힌 형식(`claude mcp add chrome-devtools --scope user npx chrome-devtools-mcp@latest`)도 있습니다. 제가 직접 시험한 것은 위의 `--` 구분자가 있는 형식입니다.

---

### 방법 B. Claude Code 플러그인 (MCP 서버 + 스킬 7개)

Claude Code 안에서:
```
/plugin marketplace add ChromeDevTools/chrome-devtools-mcp
/plugin install chrome-devtools-mcp@chrome-devtools-plugins
```
설치 후 **Claude Code를 재시작**하고 `/skills`로 스킬이 보이는지 확인합니다.

터미널 명령으로 해도 같습니다.
```bash
claude plugin marketplace add ChromeDevTools/chrome-devtools-mcp
claude plugin install chrome-devtools-mcp@chrome-devtools-plugins
```

**함께 설치되는 스킬 7개.** 스킬은 AI가 도구를 언제, 어떤 순서로 쓸지 알려 주는 사용 설명서입니다.

| 스킬 | 언제 쓰이나 |
|---|---|
| `chrome-devtools` | 기본 사용법: 이동 → 대기 → 스냅샷 → 조작 순서, 도구 고르는 법 |
| `chrome-devtools-cli` | 셸 스크립트로 브라우저 자동화할 때 |
| `a11y-debugging` | 접근성: 시맨틱 HTML, ARIA, 포커스, 키보드, 색 대비 |
| `cookie-debugging` | 쿠키, 세션, 로그인 문제, 401/403, SameSite, 쿠키 동의 배너 |
| `debug-optimize-lcp` | "페이지가 느리게 떠요", LCP 최적화 |
| `memory-leak-debugging` | 메모리 누수, 메모리 부족(OOM) |
| `troubleshooting` | 연결이 안 되거나 페이지가 안 열릴 때 |

로컬 사본으로 설치해서 확인한 구성: **스킬 7개, MCP 서버 1개, 훅 0개, 에이전트 0개**

> ⚠️ **`Failed to clone repository` 오류가 나면:** 회사 방화벽 등으로 GitHub나 Chromium 저장소 접속이 막힌 경우입니다. 저도 작업 환경에서 같은 이유로 실패했습니다. 이때는 **방법 A**를 쓰세요 (스킬은 빠지지만 도구는 똑같이 쓸 수 있습니다).
>
> 💡 플러그인 방식은 서버 버전이 고정됩니다 (현재 `chrome-devtools-mcp@1.10.1`). 방법 A의 `@latest`는 실행할 때마다 최신 버전을 씁니다.

---

### 방법 C. 다른 AI 도구

대부분의 도구는 아래 **표준 설정**을 각자의 MCP 설정 화면이나 파일에 넣으면 됩니다.
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

| 도구 | 설치 방법 (공식 문서) |
|---|---|
| Cursor | `Cursor Settings` → `MCP` → `New MCP Server`에 위 설정 |
| VS Code / Copilot | 명령 팔레트(`Ctrl+Shift+P`) → **Chat: Install Plugin From Source** → `ChromeDevTools/chrome-devtools-mcp` (MCP + 스킬). 또는 `code --add-mcp` |
| Copilot CLI | `/mcp add` → 이름 `chrome-devtools`, 종류 Local, 명령 `npx -y chrome-devtools-mcp@latest` |
| Gemini CLI | `gemini mcp add -s user chrome-devtools npx chrome-devtools-mcp@latest`<br>또는 확장(MCP + 스킬): `gemini extensions install --auto-update https://github.com/ChromeDevTools/chrome-devtools-mcp` |
| Codex | `codex mcp add chrome-devtools -- npx chrome-devtools-mcp@latest` (Windows 11은 `.codex/config.toml`에 `cmd /c`와 `startup_timeout_ms = 20_000` 설정) |
| Antigravity | MCP 설정에 위 내용 + `--browser-url=http://127.0.0.1:9222` (Antigravity 내장 브라우저에 연결) |
| Windsurf, Cline, JetBrains, Kiro, Warp, OpenCode, Amp 등 | 각 도구의 MCP 설정에 위 표준 설정 |

---

### 5.5 Claude 데스크톱 앱 Code 탭에서 쓰기

> **이 절의 근거와 한계**
> - 근거: Claude Code 공식 문서 [Desktop application](https://code.claude.com/docs/en/desktop) (2026-10-07 확인)
> - 한계: 데스크톱 앱을 **직접 실행해 보지는 않았습니다** (작업 환경에 화면이 없음). 메뉴 이름이 앱 버전에 따라 조금 다를 수 있습니다.

#### 핵심: Code 탭과 터미널 Claude Code는 설정을 공유합니다

공식 문서의 표현은 이렇습니다.
> "Desktop runs the same underlying engine with a graphical interface."
> "Desktop and CLI read the same configuration files, so your setup carries over."

Code 탭은 **터미널 Claude Code와 같은 엔진을 화면으로 감싼 것**입니다. 그래서 아래 설정이 그대로 공유됩니다.

| 공유되는 것 | 의미 |
|---|---|
| `~/.claude.json`, `.mcp.json`의 MCP 서버 | **방법 A(`claude mcp add`)로 등록한 서버가 Code 탭에서도 보임** |
| 플러그인과 스킬 | **방법 B(플러그인)로 설치한 것도 Code 탭에서 쓸 수 있음** (scope는 user, project, local 모두 지원) |
| `~/.claude/settings.json` | 권한 규칙 등 설정 공유 |
| `CLAUDE.md` | 프로젝트 메모리 공유 |

**즉, 이미 터미널에서 설치했다면 Code 탭에서 따로 할 일이 없습니다.** 앱을 재시작한 뒤 바로 쓰면 됩니다.

> ⚠️ **로컬(Local) 세션에서만 됩니다.** Code 탭에서 세션을 시작할 때 실행 환경을 **Local**(내 컴퓨터)로 고르세요. 공식 문서 기준으로 **Cloud** 세션과 **WSL** 세션에서는 데스크톱 앱에서 설치한 플러그인을 쓸 수 없습니다. 이 서버는 내 PC의 Chrome을 띄우는 도구라서 Local이 맞습니다.

#### 터미널 없이 데스크톱 앱 화면만으로 설치하기

**① 플러그인으로 설치 (MCP 서버 + 스킬 7개)**

1. Claude 데스크톱 앱 → **Code** 탭 → 실행 환경 **Local**로 새 세션을 시작합니다.
2. 입력창 옆의 **+** 버튼 → **Plugins** → **Add plugin**을 누르면 플러그인 브라우저가 열립니다.
3. 목록에서 `chrome-devtools-mcp`를 찾아 설치하고, 범위(나만 / 이 프로젝트 / 이 프로젝트에서 나만)를 고릅니다.
4. 설치된 것은 **+ → Plugins → Manage plugins**에서 켜고 끄거나 삭제할 수 있습니다.

> 💡 **목록에 안 보이면:** 플러그인 브라우저는 **등록된 마켓플레이스**의 플러그인만 보여 줍니다 (Anthropic 공식 마켓 포함). Chrome DevTools 마켓이 등록되어 있지 않으면, Code 탭의 **내장 터미널**(제목 표시줄의 **Terminal** 또는 `` Ctrl+` ``)을 열고 한 번만 등록하세요. 이후 2번부터 다시 하면 됩니다.
> ```powershell
> claude plugin marketplace add ChromeDevTools/chrome-devtools-mcp
> ```
> 이 명령에는 터미널용 Claude Code(`claude` 명령)가 설치되어 있어야 합니다. 데스크톱 앱 화면에서 마켓플레이스를 직접 추가하는 메뉴가 있는지는 공식 문서에서 확인하지 못했습니다.

**② MCP 서버만 설치 (설정 파일)**

공식 문서 기준으로 Code 탭은 아래 **세 곳**의 MCP 설정을 모두 읽습니다.

| 설정 파일 | 위치 (Windows) | 만드는 방법 |
|---|---|---|
| `~/.claude.json` | `C:\Users\<이름>\.claude.json` | 방법 A의 `claude mcp add` (직접 편집은 비추천) |
| `.mcp.json` | 프로젝트 폴더 | 직접 작성. 팀과 공유할 때 |
| `claude_desktop_config.json` | 보통 `%APPDATA%\Claude\claude_desktop_config.json`. 데스크톱 앱 **설정 → 개발자(Developer) → 설정 편집(Edit Config)** 으로 열 수 있음 (이 경로와 메뉴는 일반적인 MCP 안내 기준이며, 이번에 확인한 Code 탭 문서에는 나오지 않음) | 직접 작성. 데스크톱 앱의 **일반 채팅(Chat 탭)에서도 함께** 쓰고 싶을 때 |

`claude_desktop_config.json`에 넣는 내용은 표준 설정과 같습니다.
```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--isolated", "--no-usage-statistics"]
    }
  }
}
```
저장한 뒤 데스크톱 앱을 **완전히 종료**(작업 표시줄 트레이 아이콘에서 종료)하고 다시 실행하세요.

> ⚠️ **같은 이름을 여러 곳에 등록했을 때:** 공식 문서에 따르면 Code 탭은 같은 이름의 서버를 **한 번만 연결**합니다. `claude_desktop_config.json`과 `~/.claude.json`에 같은 이름이 있으면 **`claude_desktop_config.json` 쪽 설정을 사용**합니다. 헷갈리지 않게 **한 곳에만 등록**하세요. 플러그인(B)과도 동시에 쓰지 마세요 (5장 경고 참고).
>
> 참고: 터미널 Claude Code는 `claude_desktop_config.json`을 읽지 않습니다. 터미널과 Code 탭 양쪽에서 쓰려면 방법 A나 B가 편합니다.

#### Code 탭에서 확인하고 사용하기

1. 앱을 재시작하고 Code 탭에서 **Local** 세션을 엽니다.
2. 아래 방법 중 하나로 연결을 확인합니다.
   - 입력창에 "chrome-devtools 도구 쓸 수 있어? 도구 목록 보여줘"라고 물어보기
   - 내장 터미널에서 `claude mcp list` 실행 (방법 A, 또는 `claude_desktop_config.json`이 아닌 설정 파일로 등록한 경우)
   - 플러그인이면 **+ → Plugins**에서 `chrome-devtools-mcp`와 스킬 확인. 입력창에 `/`를 치면 스킬 목록이 나옵니다
3. 이후 사용법은 7장과 같습니다. 말로 시키면 됩니다.
   ```
   localhost:3000 열어서 로그인 버튼 눌러보고 스크린샷 찍어줘
   ```
4. 처음 도구를 쓸 때 권한 확인 카드가 나오면 허용합니다. 입력창 옆 **모드 선택기**(Manual / Accept edits / Plan / Auto)로 얼마나 자주 물어볼지 정할 수 있습니다.

Code 탭에는 파일 창(**⋮ → Files**)이 있어서, Claude가 프로젝트 폴더에 저장한 스크린샷이나 보고서를 앱 안에서 찾아보기 편합니다. 이미지가 앱 안에서 바로 미리보기 되는지는 확인하지 못했습니다.

#### Code 탭 내장 Browser 창과의 관계

Code 탭에는 Anthropic이 만든 **내장 Browser 창**(`Ctrl+Shift+B`)이 따로 있습니다. 공식 문서 기준으로 Claude가 이 창에서 페이지를 읽고 클릭할 수 있고, 내 앱을 띄워서 검증할 때도 씁니다. **chrome-devtools-mcp와는 별개의 도구**입니다.

| 비교 | 내장 Browser 창 | chrome-devtools-mcp |
|---|---|---|
| 설치 | 필요 없음 (기본 제공. 설정 → Claude Code → Browser tools에서 끌 수 있음) | 설치 필요 |
| 화면 | 앱 안의 창으로 바로 보임 | 별도 Chrome 창이 뜸 |
| 안전장치 | 외부 사이트 조작 시 안전 분류기 검사, 사이트별 승인 카드 | 클라이언트 권한 확인에 의존 |
| 강점 | 앱 미리보기와 검증, 간단한 탐색과 조작 | **성능 측정(LCP 등), 네트워크·콘솔 상세 분석, Lighthouse, 메모리 분석, 기기·네트워크 흉내** |
| 로그인 상태 | 깨끗한 별도 프로필 | 기본은 별도 프로필, `--autoConnect`로 내 Chrome 연결 가능 |

**추천 (추론):** 내 앱을 띄워서 눈으로 확인하는 정도면 **내장 Browser 창으로 충분**합니다. 성능, 네트워크, Lighthouse, 메모리 같은 **개발자 도구 수준의 분석이 필요할 때** chrome-devtools-mcp를 쓰세요. 둘 다 켜져 있으면 Claude가 어느 쪽을 쓸지 헷갈릴 수 있으니, "chrome-devtools로 성능 측정해줘"처럼 도구를 지정하면 확실합니다.

#### Code 탭에서 문제가 생기면

| 증상 | 해결 (공식 문서 기준) |
|---|---|
| `npx`나 `node`를 못 찾음 | 일반 터미널에서 `node --version`이 되는지 확인 후 **앱 재시작**. Windows 앱은 사용자·시스템 환경변수를 상속하지만 **PowerShell 프로필은 읽지 않습니다.** Node.js를 방금 설치했다면 앱을 껐다 켜야 PATH가 반영됩니다 |
| Windows에서 MCP 서버가 연결되지 않거나 토글이 반응하지 않음 | 설정 파일 확인 → 앱 재시작 → 작업 관리자에서 서버 프로세스(node)가 실행 중인지 확인 → 로그 확인. 그래도 안 되면 15장의 `cmd /c` 방식으로 등록 |
| 플러그인 메뉴가 안 보임 | Cloud나 WSL 세션인지 확인 → **Local** 세션으로 다시 시작 |
| `/permissions` 같은 명령이 "isn't available in this environment" | Code 탭에서는 터미널 대화상자 명령이 동작하지 않습니다. 설정 파일을 직접 고치거나 내장 터미널에서 실행하세요 |
| 로그 위치 | Windows: 이벤트 뷰어 → Windows 로그 → 응용 프로그램 |

#### Code 탭에서 삭제하기

| 설치 방법 | 삭제 |
|---|---|
| 플러그인 | **+ → Plugins → Manage plugins**에서 uninstall. 또는 내장 터미널에서 `claude plugin uninstall chrome-devtools-mcp@chrome-devtools-plugins` |
| 방법 A (`claude mcp add`) | 내장 터미널에서 `claude mcp remove chrome-devtools --scope user` |
| `claude_desktop_config.json` | 파일에서 `chrome-devtools` 항목을 지우고 앱 완전 종료 후 재시작 |

남는 파일(브라우저 전용 프로필 등) 정리는 14.4절과 같습니다.

---

## 6. 설치 확인과 첫 사용

### 6.1 연결 상태 확인 (실제 실행 확인)

```bash
claude mcp list
```
```
chrome-devtools: npx -y chrome-devtools-mcp@latest - √ Connected
```
Claude Code 안에서는 `/mcp`를 입력하면 연결 상태와 도구 목록을 볼 수 있습니다.

자세히 보기:
```bash
claude mcp get chrome-devtools
```
```
chrome-devtools:
  Scope: User config (available in all your projects)
  Status: √ Connected
  Type: stdio
  Command: npx
  Args: -y chrome-devtools-mcp@latest
```

### 6.2 첫 프롬프트

Claude Code를 실행하고 공식 문서의 첫 예제를 입력합니다.
```
Check the performance of https://developers.chrome.com
```
한국어로 해도 됩니다.
```
https://developers.chrome.com 성능 측정해줘
```
→ Chrome 창이 뜨고, 페이지가 열리고, 성능 기록 결과(LCP 등)가 나오면 성공입니다.

> 💡 처음 실행할 때는 `npx`가 패키지를 내려받느라 몇십 초 걸릴 수 있습니다.
> 💡 도구를 처음 쓸 때 Claude Code가 **권한을 물어봅니다.** 내용을 확인하고 허용하세요.

---

## 7. 사용법: 이렇게 말하면 됩니다

도구 이름을 몰라도 됩니다. 원하는 것을 말하면 AI가 알맞은 도구를 고릅니다.

| 하고 싶은 일 | 이렇게 말하기 | AI가 쓰는 도구 (예상) |
|---|---|---|
| 페이지 열기 | "localhost:3000 열어줘" | `new_page` |
| 화면 캡처 | "지금 화면 스크린샷 찍어줘" / "전체 페이지 캡처해서 shot.png로 저장해줘" | `take_screenshot` |
| 클릭·입력 | "이메일 칸에 test@example.com 넣고 로그인 버튼 눌러줘" | `take_snapshot` → `fill` → `click` |
| 폼 채우기 | "회원가입 폼 테스트 데이터로 채워서 제출해줘" | `fill_form` |
| 에러 찾기 | "이 페이지 콘솔 에러 확인해줘" | `list_console_messages` |
| API 확인 | "로그인할 때 어떤 API 호출하는지, 응답 코드 뭔지 보여줘" | `list_network_requests` → `get_network_request` |
| 성능 측정 | "이 페이지 왜 느린지 분석해줘" | `performance_start_trace` → `performance_analyze_insight` |
| 품질 점검 | "Lighthouse로 접근성이랑 SEO 점검해줘" | `lighthouse_audit` |
| 모바일 테스트 | "아이폰 크기로 바꾸고 메뉴 깨지는지 봐줘" | `emulate` / `resize_page` → `take_screenshot` |
| 느린 네트워크 | "느린 3G 환경에서 로딩 화면 잘 나오는지 확인해줘" | `emulate` |
| 스타일 문제 | "이 버튼 왜 파란색이 아닌지 CSS 확인해줘" | `get_css_styles` |
| 값 읽기 | "장바구니 총액 값이 뭔지 읽어줘" | `evaluate_script` / `take_snapshot` |

**실제 테스트 결과 예시** (로컬 테스트 페이지):
```
list_console_messages
  msgid=3 [error] demo error: 일부러 낸 콘솔 에러

list_network_requests
  reqid=1 GET http://localhost:8765/ [200]
  reqid=2 GET http://localhost:8765/favicon.ico [404]
  reqid=3 GET http://localhost:8765/api.json [200]

performance_start_trace
  LCP: 277 ms (TTFB 5 ms, Render delay 273 ms)
  CLS: 0.00

lighthouse_audit
  Accessibility: 95 / Best Practices: 100 / SEO: 90
  보고서: report.json, report.html
```

### 결과물 파일

스크린샷, 성능 기록(`trace.json.gz`), Lighthouse 보고서(`report.html`)는 **파일로 저장**할 수 있습니다. 공식 설계 원칙상 큰 데이터는 파일 경로로 돌려주는 것이 기본입니다. "바탕화면에 저장해줘"처럼 위치를 말하세요.

> ⚠️ **파일 저장 위치 제한:** AI 클라이언트가 허용 폴더(MCP roots)를 알려 주지 않으면, 파일 쓰기는 기본적으로 **OS 임시 폴더로 제한**됩니다. 다른 폴더에 저장하려면 `--filesystem-root=<폴더>` 옵션을 추가하세요 (9장).

---

## 8. 응용: 실전 시나리오

### 시나리오 1. 내가 만든 웹앱 자동 점검

```
npm run dev로 띄운 localhost:5173을 열어서
1) 회원가입 → 로그인 → 글쓰기까지 실제로 눌러보고
2) 각 단계 스크린샷을 screenshots/ 폴더에 저장하고
3) 콘솔 에러와 실패한 네트워크 요청(4xx/5xx)이 있으면 원인을 코드에서 찾아 고쳐줘.
```
AI가 고친 뒤 **다시 브라우저로 확인까지** 할 수 있다는 것이 핵심입니다. 코드만 보고 "고쳤을 거예요"라고 추측하는 대신 실제 화면으로 검증합니다.

### 시나리오 2. "페이지가 느려요"

```
https://내사이트.com 메인 페이지 LCP가 느린 이유 찾아서 개선 방법 알려줘.
모바일 + 느린 4G 조건으로 측정해줘.
```
플러그인 방식이면 `debug-optimize-lcp` 스킬이 함께 동작합니다. LCP를 TTFB, 리소스 로딩 지연, 렌더링 지연 같은 구간으로 나눠서 분석합니다.

### 시나리오 3. 로그인·쿠키 문제

```
로그인 후 새로고침하면 로그아웃되는 문제가 있어.
로그인 요청과 응답의 Set-Cookie 헤더, 쿠키 속성(SameSite, Secure, 만료) 확인해줘.
```

### 시나리오 4. 접근성 점검

```
이 페이지를 키보드만으로 쓸 수 있는지, 이미지 대체 텍스트와 색 대비 문제 있는지 점검해줘.
```

### 시나리오 5. 반응형 디자인 확인

```
375x812(모바일), 768x1024(태블릿), 1440x900(데스크톱) 세 크기로 메인 페이지 스크린샷 찍고
레이아웃 깨지는 곳 알려줘.
```

### 시나리오 6. 메모리 누수 (고급)

`--memoryDebugging` 옵션을 켜고:
```
목록 페이지를 열고 '더보기'를 20번 누른 전후로 힙 스냅샷을 찍어서 비교하고,
안 풀리는 객체가 뭔지 찾아줘.
```

### 시나리오 7. Chrome 확장 프로그램 개발 (고급)

`--categoryExtensions` 옵션을 켜고:
```
./my-extension 폴더의 확장을 설치하고, 팝업을 열어서 버튼이 동작하는지 확인해줘.
```

### 시나리오 8. 반복 작업 자동화

```
이 표의 1~5페이지를 차례로 넘기면서 각 행의 제목과 가격을 CSV로 정리해줘.
```
> ⚠️ 다른 사람의 웹사이트를 자동으로 수집할 때는 그 사이트의 이용약관과 robots.txt, 관련 법을 지키세요.

---

## 9. 응용: 설정 옵션 (플래그)

옵션은 설치 명령 끝에 붙입니다.
```bash
claude mcp add chrome-devtools --scope user -- npx -y chrome-devtools-mcp@latest --isolated --no-usage-statistics
```
이미 등록했다면 `claude mcp remove chrome-devtools --scope user` 후 다시 등록하세요.

### 자주 쓰는 옵션

| 옵션 | 효과 | 추천 상황 |
|---|---|---|
| `--headless` | 창을 띄우지 않고 백그라운드에서 실행 | 화면을 볼 필요 없을 때, 서버 |
| `--isolated` | 실행할 때마다 **임시 프로필** 사용, 끝나면 자동 삭제 | **보안상 추천.** 쿠키나 로그인 흔적이 남지 않음 |
| `--slim` | 도구 3개만 (이동, JS, 캡처) | 간단한 작업, 토큰 절약 |
| `--no-usage-statistics` | Google에 사용 통계를 보내지 않음 (기본값은 보냄) | 개인정보가 신경 쓰일 때 |
| `--no-performance-crux` | 성능 측정 시 URL을 Google CrUX API로 보내지 않음 | 사내 URL 등 외부로 보내면 안 될 때 |
| `--viewport=1280x720` | 시작 창 크기 | 스크린샷 크기를 맞출 때 |
| `--channel=canary` | Chrome Canary/Dev/Beta 사용 | 새 Chrome 기능 테스트 |
| `--executablePath=<경로>` | 특정 Chrome 실행 파일 사용 | Chrome이 기본 위치에 없을 때 |
| `--filesystem-root=<폴더>` | 파일 저장·읽기를 허용할 폴더 (여러 번 지정 가능) | 스크린샷을 프로젝트 폴더에 저장할 때 |
| `--screenshot-format=jpeg` / `--screenshot-max-width=1024` | 스크린샷 형식, 최대 크기 | 이미지 토큰 절약 |
| `--accept-insecure-certs` | 자체 서명·만료 인증서 무시 | 로컬 HTTPS 개발. **주의해서 사용** |
| `--blocked-url-pattern` / `--allowed-url-pattern` | 접속 차단·허용 목록 (allowed는 Chrome 149 이상) | AI가 엉뚱한 사이트에 못 가게 제한 |
| `--no-javascript-evaluation` | JS 실행 도구 끄기 | 더 안전하게 쓰고 싶을 때 |
| `--no-file-navigations` | `file://` 주소 접근 금지 | 내 PC 파일을 브라우저로 못 열게 |
| `--redact-network-headers` | 민감한 네트워크 헤더 가리기 | 인증 토큰 노출 방지 |
| `--memoryDebugging` | 메모리 분석 도구 추가 | 메모리 누수 조사 |
| `--categoryExtensions` | 확장 프로그램 도구 추가 | 확장 개발 |
| `--experimentalScreencast` | 동영상 녹화 (ffmpeg 필요) | 동작 영상 기록 |
| `--log-file=<경로>` | 디버그 로그 저장 (`NODE_DEBUG=*`과 함께) | 문제 보고용 |

전체 옵션 보기:
```bash
npx chrome-devtools-mcp@latest --help
```

### 설정 파일로 관리하기

옵션이 많으면 JSON 파일로 관리할 수 있습니다. 키는 camelCase로 씁니다.
```json
{
  "headless": true,
  "isolated": true,
  "usageStatistics": false,
  "blockedUrlPattern": ["*://*.example.com/*"]
}
```
서버는 아래 순서로 **처음 찾은 파일 하나만** 사용합니다 (여러 파일을 합치지 않음). 명령줄 옵션이 파일보다 우선합니다.
1. `--config=<경로>`로 지정한 파일
2. 현재 폴더의 `cd4a.config.json`
3. 플러그인 데이터 폴더의 `cd4a.config.json` (플러그인 설치 시)
4. 전역 설정 파일
   - Windows: `%LOCALAPPDATA%\Google\cd4a\config.json` (없으면 `~/.config/cd4a/config.json`)
   - macOS/Linux: `~/.config/cd4a/config.json`

---

## 10. 응용: 내 Chrome에 연결하기

### 기본 동작

- 새 Chrome을 **전용 프로필**로 실행합니다.
  - Windows: `%USERPROFILE%\.cache\chrome-devtools-mcp\chrome-profile`
  - macOS/Linux: `~/.cache/chrome-devtools-mcp/chrome-profile`
- 이 프로필은 **실행 사이에 유지됩니다.** 여기서 한 로그인은 다음에도 남아 있습니다. 남기기 싫으면 `--isolated`를 쓰세요.
- 한 번에 한 브라우저만 이 프로필을 쓸 수 있습니다.

### 방법 1. 자동 연결 `--autoConnect` (Chrome 144 이상)

평소 쓰는 Chrome의 로그인 상태를 그대로 쓰고 싶을 때 사용합니다.
1. Chrome 주소창에 `chrome://inspect/#remote-debugging`을 입력하고 원격 디버깅을 켭니다.
2. 옵션을 붙여 등록합니다.
   ```bash
   claude mcp add chrome-devtools --scope user -- npx -y chrome-devtools-mcp@latest --autoConnect
   ```
3. Chrome을 먼저 켜 둔 상태에서 AI에게 작업을 시키면, Chrome에 **허용 여부를 묻는 창**이 뜹니다. **Allow**를 누릅니다.

> 🚨 **주의:** 이 방식에서는 AI가 그 프로필의 **열린 모든 창과 탭**(메일, 은행, 업무 사이트 등)에 접근할 수 있습니다 (공식 문서 명시). 민감한 탭은 닫고 쓰거나, 테스트 전용 Chrome 프로필을 따로 만들어 쓰세요. 프로필이 여러 개면 기본 프로필에 연결됩니다.

### 방법 2. 디버깅 포트로 수동 연결 `--browser-url`

샌드박스·가상머신 안의 AI가 바깥 Chrome에 붙어야 할 때 사용합니다.
1. 열려 있는 Chrome을 **모두 닫고** 디버깅 포트를 열어 실행합니다 (Windows).
   ```powershell
   & "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="$env:TEMP\chrome-profile-stable"
   ```
   - Chrome 보안 정책상 `--user-data-dir`로 **기본이 아닌 프로필 폴더**를 지정해야 합니다.
2. 등록합니다.
   ```bash
   claude mcp add chrome-devtools --scope user -- npx -y chrome-devtools-mcp@latest --browser-url=http://127.0.0.1:9222
   ```

> 🚨 디버깅 포트가 열려 있는 동안에는 **내 PC의 어떤 프로그램이든** 그 Chrome을 조종할 수 있습니다. 민감한 사이트를 열지 말고, 끝나면 그 Chrome을 닫으세요.

### Android Chrome 디버깅

저장소의 `docs/debugging-android.md`에 안내가 있습니다 (USB 디버깅 + 포트 포워딩).

---

## 11. 응용: 터미널 CLI로 쓰기

AI 없이 터미널에서 직접 쓰는 **실험적** 명령어 `chrome-devtools`가 함께 들어 있습니다. 셸 스크립트로 자동화할 때 유용합니다. (실제 실행 확인)

```bash
npm i chrome-devtools-mcp@latest -g     # 전역 설치
chrome-devtools status                   # 동작 확인

chrome-devtools new_page "https://example.com"              # 새 탭 (페이지 번호가 나옴)
chrome-devtools navigate_page 1 --url "https://web.dev"     # 1번 페이지 이동
chrome-devtools take_screenshot 1 --filePath screenshot.png # 1번 페이지 캡처
chrome-devtools list_pages --output-format=json             # JSON으로 결과 받기
chrome-devtools lighthouse_audit 1 --mode snapshot          # Lighthouse
chrome-devtools stop                                         # 백그라운드 종료
```

- 처음 명령을 실행하면 **백그라운드 데몬과 브라우저가 자동으로 시작**되고, 이후 명령은 같은 브라우저를 계속 씁니다.
- CLI의 기본값은 **headless(창 없음) + isolated(임시 프로필)** 입니다. MCP 방식과 다릅니다.
- ⚠️ CLI는 기본적으로 **파일 접근 제한이 없습니다.** 제한하려면 `chrome-devtools start --workspace=<폴더>`로 시작하세요.
- 일부 도구(`wait_for`, `fill_form`, 확장 관련)는 CLI에서 쓸 수 없습니다.
- 멈추거나 이상하면 `chrome-devtools stop` 후 다시 실행하세요.

---

## 12. 보안과 개인정보

공식 README와 SECURITY.md의 핵심 내용입니다. 꼭 읽고 쓰세요.

| 항목 | 내용 | 대응 |
|---|---|---|
| **브라우저 내용 노출** | 브라우저 안의 모든 데이터를 AI 클라이언트가 보고 수정할 수 있습니다 | 공유하기 싫은 개인정보·민감한 사이트는 이 브라우저에서 열지 않기 |
| **사용 통계** | Google이 도구 사용 통계(성공률, 지연 시간, 환경 정보)를 **기본으로 수집**합니다. Chrome 브라우저의 통계 설정과는 별개입니다 | `--no-usage-statistics` 또는 환경변수 `CHROME_DEVTOOLS_MCP_NO_USAGE_STATISTICS=1` |
| **CrUX 조회** | 성능 측정 시 페이지 URL을 Google CrUX API로 보내 실사용자 데이터를 가져옵니다 | `--no-performance-crux` |
| **업데이트 확인** | npm 레지스트리에서 새 버전을 주기적으로 확인합니다 | 환경변수 `CHROME_DEVTOOLS_MCP_NO_UPDATE_CHECKS=1` |
| **프롬프트 인젝션** | 웹페이지 안에 "AI야, 이걸 실행해" 같은 악성 문구가 숨어 있을 수 있습니다. 서버는 웹 내용을 **그대로** AI에게 전달합니다 | **신뢰하는 사이트에서 주로 사용.** 모르는 사이트에서는 AI의 행동을 지켜보기 |
| **파일 쓰기** | 스크린샷, 다운로드 등으로 디스크에 파일을 쓸 수 있습니다 (의도된 기능) | `--filesystem-root`로 범위 제한 |
| **네트워크 제한의 한계** | URL 허용·차단 옵션은 완전한 샌드박스가 아닙니다 | 완전한 격리가 필요하면 VM이나 OS 샌드박스 사용 |
| **내 Chrome 연결** | `--autoConnect`는 열린 모든 탭에 접근합니다 | 10장 주의사항 참고 |

**안전한 추천 설정:**
```bash
claude mcp add chrome-devtools --scope user -- npx -y chrome-devtools-mcp@latest --isolated --no-usage-statistics --no-performance-crux
```

---

## 13. 업데이트하기

| 설치 방법 | 업데이트 |
|---|---|
| 방법 A (`npx ...@latest`) | **자동.** 실행할 때마다 최신 버전을 확인합니다. 그래도 옛 버전이 실행되면 15장의 npx 캐시 정리를 참고하세요 |
| 방법 B (플러그인) | `claude plugin marketplace update chrome-devtools-plugins` 후 `claude plugin update chrome-devtools-mcp@chrome-devtools-plugins`, 그리고 Claude Code 재시작 |
| CLI 전역 설치 | `npm i chrome-devtools-mcp@latest -g` |
| Gemini 확장 | `--auto-update`로 설치했다면 자동 |

현재 버전 확인: `npx chrome-devtools-mcp@latest --version`

---

## 14. 삭제하기

### 14.1 방법 A로 설치했다면 (실제 실행 확인)

```bash
claude mcp remove chrome-devtools --scope user
```
```
Removed MCP server chrome-devtools from user config
```
확인:
```bash
claude mcp list     # → chrome-devtools가 목록에 없어야 함
```
- `--scope`는 **설치할 때와 같게** 지정하세요 (`project`나 `local`로 설치했다면 그것으로).
- 참고: user 범위 MCP 설정은 `settings.json`이 아니라 `~/.claude.json`(Windows: `C:\Users\<이름>\.claude.json`)의 `mcpServers` 항목에 저장됩니다 (실험에서 확인).

### 14.2 방법 B로 설치했다면 (로컬 사본으로 실행 확인)

```bash
claude plugin uninstall chrome-devtools-mcp@chrome-devtools-plugins
claude plugin marketplace remove chrome-devtools-plugins
```
Claude Code 안에서는 `/plugin` 관리 화면에서도 할 수 있습니다.

### 14.3 잠시 끄기만 하려면

- Claude Code 안에서 `/mcp`를 열고 해당 서버를 비활성화할 수 있습니다.
- 플러그인이면 `claude plugin disable chrome-devtools-mcp@chrome-devtools-plugins`

### 14.4 남는 것 정리 (선택)

| 남는 것 | 위치 | 정리 방법 |
|---|---|---|
| **브라우저 전용 프로필** (쿠키, 로그인 기록 포함) | Windows: `%USERPROFILE%\.cache\chrome-devtools-mcp`<br>macOS/Linux: `~/.cache/chrome-devtools-mcp` | 폴더 삭제. **개인정보 정리 차원에서 권장** |
| CLI 전역 설치 | npm 전역 | `npm uninstall -g chrome-devtools-mcp` |
| CLI 백그라운드 프로세스 | — | 삭제 전에 `chrome-devtools stop` |
| npx 캐시 | npm 캐시 폴더 | 보통 그대로 둬도 됩니다. 정리하려면 `npm cache clean --force` (**다른 패키지 캐시도 함께 지워짐**) |
| 설정 파일 | `cd4a.config.json`, 전역 `cd4a/config.json` | 만들었다면 직접 삭제 |
| 프로젝트 공유 설정 | 프로젝트의 `.mcp.json` | `--scope project`로 설치했다면 해당 항목 삭제 |
| 저장한 스크린샷, 보고서 | 저장한 위치 | 직접 삭제 |

Windows PowerShell 예시:
```powershell
Remove-Item -Recurse -Force "$env:USERPROFILE\.cache\chrome-devtools-mcp"
```

---

## 15. 문제 해결

| 증상 | 원인 | 해결 |
|---|---|---|
| **Windows: `MCP error -32000: Connection closed`** | Windows에서 `npx`를 다른 프로그램이 바로 실행하지 못함 | `cmd /c`로 감싸서 등록: `claude mcp add chrome-devtools --scope user -- cmd /c npx -y chrome-devtools-mcp@latest` (공식 해결법) |
| `ERR_MODULE_NOT_FOUND` | Node.js 버전이 낮거나 클라이언트가 다른 Node를 사용 | Node.js LTS 설치, 터미널 재시작. AI 클라이언트와 터미널이 같은 Node를 쓰는지 확인 |
| Chrome이 안 뜸 / `Target closed` | Chrome 미설치, 경로 문제, 같은 프로필을 다른 Chrome이 사용 중 | Chrome 설치 확인, `--executablePath` 지정, `--isolated` 사용 |
| Linux 컨테이너에서 Chrome이 바로 꺼짐 | **root 계정에서는 Chrome이 실행되지 않음** (실험에서 같은 오류 확인) | 일반 사용자 계정으로 실행 |
| WSL에서 안 됨 | WSL의 알려진 문제 | WSL 안에 Chrome 설치, 또는 PowerShell이나 Git Bash 사용 |
| 플러그인 설치 시 `Failed to clone repository` | GitHub이나 Chromium 저장소 접속 차단 (방화벽, 프록시) | 방법 A(`claude mcp add`)로 설치 |
| `--autoConnect`에서 시간 초과 | Chrome 미실행, 원격 디버깅 꺼짐, 허용 창을 안 누름, 다른 도구가 포트 사용 중 | Chrome 144 이상을 켜 두고 `chrome://inspect/#remote-debugging`에서 켠 뒤 Allow 클릭. 탭이 수백 개면 느릴 수 있음 |
| macOS: Web Bluetooth 사용 시 Chrome 충돌 | 알려진 문제 | 공식 troubleshooting 문서 참고 |
| 저장한 파일이 안 보임 | 파일 쓰기가 임시 폴더로 제한됨 | `--filesystem-root=<폴더>` 추가 |
| 옛 버전이 계속 실행됨 | npx 캐시 | `npx -y chrome-devtools-mcp@latest --version`으로 확인, 필요하면 npm 캐시 정리 |
| 원인을 모르겠음 | — | `npx chrome-devtools-mcp@latest --help`가 실행되는지 먼저 확인. 로그: `NODE_DEBUG=*`와 `--log-file=<경로>` |

플러그인 방식이면 `troubleshooting` 스킬이 있으니 "chrome devtools 연결이 안 돼, 원인 찾아줘"라고 요청해도 됩니다.

---

## 16. 토큰 비용

| 항목 | 비용 | 근거 |
|---|---|---|
| 기본 모드 도구 30개의 설명(스키마) | 약 6,400 토큰 (25,691자 ÷ 4로 추정) | 실측 글자 수 |
| `--slim` 모드 도구 3개 | 약 200 토큰 (846자) | 실측 글자 수 |
| 플러그인 스킬 7개 (상시) | 약 806 토큰 | `claude plugin details` 출력 |
| 스킬 1개 실행 시 | 약 1.5k ~ 4.4k 토큰 | 같은 출력 |
| 스크린샷 | 이미지 크기에 비례 | `--screenshot-max-width`, JPEG로 절약 |
| 스냅샷 | 페이지가 복잡할수록 큼 | 큰 결과는 `filePath`로 파일 저장 권장 (공식 스킬 안내) |

- **추론:** Claude Code는 MCP 도구가 많으면 도구 설명을 처음부터 다 넣지 않고 필요할 때 불러오는 방식(도구 검색)을 쓸 수 있습니다. 그런 경우 실제 상시 비용은 위 추정보다 작습니다. `/context` 명령으로 실제 사용량을 확인하세요.
- **절약 팁:**
  1. 간단한 캡처와 이동만 필요하면 `--slim`
  2. 필요 없는 분류는 끄기 (예: `--no-category-performance`는 30개에서 27개로, `--no-category-emulation`은 28개로 줄어듦. 실측)
  3. 스크린샷 대신 스냅샷(텍스트) 위주로 요청
  4. 안 쓸 때는 `/mcp`에서 끄기

---

## 17. 자주 묻는 질문

**Q. 무료인가요?**
A. 네. Apache-2.0 오픈소스입니다. AI 서비스의 토큰 비용은 별도입니다.

**Q. 코딩을 몰라도 쓸 수 있나요?**
A. 설치 명령 한 줄과 Node.js, Chrome만 있으면 됩니다. 이후에는 말로 시키면 됩니다. 다만 원래 웹 개발자용 도구라서 결과(콘솔 에러, 네트워크, 성능 지표)를 해석하는 데는 약간의 배경지식이 도움이 됩니다.

**Q. 내 Chrome의 로그인 정보를 AI가 가져가나요?**
A. 기본 설정에서는 **별도 프로필**로 새 Chrome을 띄우므로 평소 쓰는 Chrome의 로그인 정보에 접근하지 않습니다. `--autoConnect`나 `--browser-url`로 연결하면 접근할 수 있습니다 (10장).

**Q. Edge, Whale, Brave에서도 되나요?**
A. 공식 지원은 Google Chrome과 Chrome for Testing뿐입니다. 다른 Chromium 계열은 "동작할 수도 있지만 보장하지 않음"입니다. 제 테스트는 Chromium으로 했고 동작했습니다.

**Q. 아무 웹사이트나 자동으로 조작해도 되나요?**
A. 기술적으로는 가능하지만, 사이트 이용약관, 자동화 금지 규정, 개인정보 관련 법을 지켜야 합니다. 일부 사이트는 자동화된 브라우저의 로그인을 막습니다 (공식 문서 언급). 결제, 송금, 계정 설정처럼 되돌리기 어려운 작업은 AI에게 맡기지 마세요.

**Q. agent-skills의 `browser-testing-with-devtools` 스킬과는 무슨 관계인가요?**
A. 그 스킬이 바로 이 도구를 쓰는 방법을 안내하는 절차서입니다. agent-skills 가이드 10.4절의 설정과 이 문서의 방법 A를 함께 쓰면 됩니다.

**Q. Claude 데스크톱 앱에서도 쓸 수 있나요?**
A. 네. **Code 탭**은 터미널 Claude Code와 설정을 공유하므로, 터미널에서 설치했다면 그대로 쓸 수 있고, 앱 화면(+ → Plugins)에서 설치할 수도 있습니다. Local 세션에서 쓰세요. 일반 채팅(Chat 탭)에서는 `claude_desktop_config.json`에 등록합니다. 자세한 내용은 [5.5절](#55-claude-데스크톱-앱-code-탭에서-쓰기)을 보세요.

**Q. Claude in Chrome이나 Playwright MCP와는 뭐가 다른가요?**
A. 모두 AI가 브라우저를 조작하는 도구입니다. Chrome DevTools MCP는 **Google이 만들었고, 성능 측정, 네트워크·콘솔 분석, Lighthouse, 메모리 분석 같은 개발자 도구 기능이 강점**입니다. 다른 도구들은 이번에 상세히 비교하지 않았습니다. 필요하면 별도로 정리해 드리겠습니다.

---

## 18. 출처

- 공식 GitHub (1차 근거): <https://github.com/ChromeDevTools/chrome-devtools-mcp>
  - `README.md`, `SECURITY.md`, `CHANGELOG.md`, `LICENSE`
  - `docs/client-configurations.md`, `docs/configuration.md`, `docs/tool-reference.md`, `docs/slim-tool-reference.md`, `docs/advanced-usage.md`, `docs/cli.md`, `docs/troubleshooting.md`, `docs/design-principles.md`, `docs/debugging-android.md`, `docs/third-party-developer-tools.md`
  - `skills/*/SKILL.md` (7개), `plugin.json`, `mcp.json`, `.claude-plugin/marketplace.json`
- Claude Code 공식 문서 Desktop application (Code 탭, 설정 공유, 플러그인, 내장 Browser 창): <https://code.claude.com/docs/en/desktop>
- npm: <https://www.npmjs.com/package/chrome-devtools-mcp> (v1.10.1, `npm view`로 확인)
- 실측: chrome-devtools-mcp 1.10.1 + Chromium(headless) + MCP SDK 클라이언트로 도구 호출, Claude Code 2.1.289의 `claude mcp`, `claude plugin` 명령 실행 결과
