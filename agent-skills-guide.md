# agent-skills 완전 가이드 (초보자용)

> **한 줄 요약**
> agent-skills는 AI 코딩 에이전트(Claude Code, Codex, Cursor 등)가 **선배 엔지니어처럼 일하도록** 만드는 무료(MIT) 작업 절차서 모음입니다.
> Claude Code에서는 아래 두 줄로 설치하고, `/spec` → `/plan` → `/build` → `/test` → `/review` → `/ship` 순서로 쓰면 됩니다.
>
> ```
> /plugin marketplace add addyosmani/agent-skills
> /plugin install agent-skills@addy-agent-skills
> ```

- 공식 사이트: <https://skills.addy.ie/>
- 공식 GitHub: <https://github.com/addyosmani/agent-skills>
- 이 문서 기준 버전: **v0.6.12** (main 브랜치 커밋 `1401c8b`, 2026-10-03)
- 작성일: 2026-10-05

---

## 목차

0. [이 문서를 어떻게 검증했나](#0-이-문서를-어떻게-검증했나)
1. [agent-skills란 무엇인가](#1-agent-skills란-무엇인가)
2. [무엇이 들어 있나 (구성요소)](#2-무엇이-들어-있나-구성요소)
3. [설치 전 준비물](#3-설치-전-준비물)
4. [설치하기](#4-설치하기)
5. [설치가 잘 됐는지 확인하기](#5-설치가-잘-됐는지-확인하기)
6. [기본 사용법: 9개 명령어](#6-기본-사용법-9개-명령어)
7. [25개 스킬 전체 목록](#7-25개-스킬-전체-목록)
8. [에이전트(전문가 페르소나) 4개](#8-에이전트전문가-페르소나-4개)
9. [응용: 상황별 실전 시나리오](#9-응용-상황별-실전-시나리오)
10. [응용: 고급 설정과 커스터마이즈](#10-응용-고급-설정과-커스터마이즈)
11. [업데이트하기](#11-업데이트하기)
12. [삭제하기](#12-삭제하기)
13. [문제 해결](#13-문제-해결)
14. [토큰 비용](#14-토큰-비용)
15. [자주 묻는 질문](#15-자주-묻는-질문)
16. [출처](#16-출처)

---

## 0. 이 문서를 어떻게 검증했나

정확성을 위해 확인한 범위와 확인하지 못한 범위를 먼저 밝힙니다.

| 대상 | 확인 방법 | 결과 |
|---|---|---|
| GitHub 저장소 전체 | `git clone` 후 README, `docs/` 17개 문서, 스킬 25개, 명령어 9개, 에이전트 4개, `hooks/`, `references/`, `evals/`, 매니페스트를 직접 읽음 | ✅ 확인 |
| Claude Code 플러그인 설치, 비활성화, 업데이트, 삭제 | 격리된 임시 설정 폴더에서 Claude Code 2.1.289로 **실제 실행** | ✅ 실행 검증 |
| `npx skills` 설치, 목록, 업데이트, 삭제 | 임시 프로젝트에서 `skills` CLI 1.7.0으로 **실제 실행** | ✅ 실행 검증 |
| 공식 사이트 skills.addy.ie | curl, WebFetch, Chromium(headless) 모두 시도 | ❌ **작업 환경의 네트워크 정책이 차단.** 검색 엔진 결과(제목과 요약)로 페이지 구조만 확인 |

**사이트에 대해 확인한 내용** (검색 결과 기준):
- 페이지 구성: 홈, `/docs/getting-started/`, `/skills/<스킬이름>/` (스킬별 페이지), `/tutorials/`
- 튜토리얼 3개
  1. **새 앱 만들기** (`/tutorials/new-app/`): 빈 폴더에서 습관 추적 앱(habit tracker) 만들기
  2. **남이 작성한 코드에 기능 추가** (`/tutorials/existing-app/`): 코드베이스 파악 → 기존 패턴에 맞춘 스펙 → 낯선 코드 검증 → 기능 플래그 뒤에 배포
  3. **루프 엔지니어링** (`/tutorials/loop-engineering/`): 밤새 코드베이스를 개선하고 아침에 사람의 승인을 받는 "가장 작고 정직한 소프트웨어 공장"
- 저장소 문서(`docs/getting-started.md`)도 이 튜토리얼을 "Claude Code, Codex 등에서 복사해 쓸 수 있는 프롬프트가 있는 인터랙티브 튜토리얼"이라고 소개합니다.

> ⚠️ **불일치 주의:** 검색 결과에 나온 사이트 요약에는 "명령어 8개", "스킬 24개"라는 표현이 있습니다. 저장소 최신본에는 **명령어 9개**(`/constraints` 추가), **스킬 25개**(메타 스킬 포함)입니다. 검색 엔진에 저장된 사이트 내용이 오래됐을 가능성이 높으므로, 이 문서는 **저장소 최신본을 기준**으로 씁니다.
> 사이트 튜토리얼의 세부 단계는 직접 읽지 못했습니다. 9장의 시나리오는 저장소 문서를 바탕으로 같은 주제를 재구성한 것입니다.

---

## 1. agent-skills란 무엇인가

### 1.1 쉬운 비유

AI 코딩 에이전트는 **실력은 좋지만 지름길을 좋아하는 신입 개발자**와 비슷합니다. 시키면 빨리 만들지만 스펙 정리, 테스트, 보안 점검, 리뷰를 건너뛰기 쉽습니다.

agent-skills는 이 신입에게 건네는 **팀의 업무 매뉴얼**입니다.
- "기능을 만들기 전에 스펙부터 써라"
- "테스트가 실패하는 걸 먼저 확인하고 구현해라"
- "100줄 넘게 한 번에 쓰지 말고 잘게 나눠서 커밋해라"
- "'아마 될 거예요'는 안 된다. 테스트 결과로 증명해라"

### 1.2 실제 구조

- 스킬 하나는 `SKILL.md`라는 **마크다운 파일 하나**입니다. 프로그램이 아니라 에이전트가 읽고 따르는 **작업 절차서**입니다.
- Claude Code는 세션을 시작할 때 각 스킬의 **이름과 짧은 설명(description)만** 읽어 둡니다. 작업 내용이 설명과 맞으면 그때 **본문 전체를 불러와서** 그 절차대로 일합니다. 이 방식을 progressive disclosure(필요할 때만 펼치기)라고 하며, 덕분에 스킬이 많아도 평소 토큰 소모가 작습니다.
- 모든 스킬이 같은 뼈대를 가집니다.

```
SKILL.md
├─ Frontmatter        : name, description ("Use when…" 사용 조건)
├─ Overview           : 이 스킬이 하는 일
├─ When to Use        : 써야 할 때, 쓰지 말아야 할 때
├─ Process            : 단계별 작업 절차
├─ Rationalizations   : 에이전트가 단계를 건너뛸 때 대는 핑계와 반박
├─ Red Flags          : 절차를 어기고 있다는 신호
└─ Verification       : 완료 조건 (증거 요구)
```

**Rationalizations(핑계-반박 표)** 가 이 팩의 특징입니다. 예를 들어 "테스트는 나중에 추가할게요"라는 핑계에 대한 반박이 미리 적혀 있어서, 에이전트가 스스로 핑계를 대고 단계를 건너뛰는 일을 줄여 줍니다.

### 1.3 설계 원칙 (README 기준)

- **Process, not prose:** 읽을거리가 아니라 따라 하는 절차
- **Anti-rationalization:** 단계를 건너뛰는 핑계를 미리 차단
- **Verification is non-negotiable:** "맞아 보인다"는 완료가 아님. 테스트 통과, 빌드 출력, 실행 데이터 같은 증거가 필요
- **Progressive disclosure:** 필요할 때만 상세 내용 로드
- Google 엔지니어링 문화(『Software Engineering at Google』, Google eng-practices)의 개념을 반영했습니다. 예: Hyrum's Law, Beyonce Rule, 테스트 피라미드, 약 100줄 단위 변경, Chesterton's Fence, 트렁크 기반 개발, Shift Left

### 1.4 개발 생명주기와의 대응

```
  DEFINE        PLAN         BUILD        VERIFY       REVIEW        SHIP
 (정의)        (계획)        (구현)        (검증)        (리뷰)        (배포)
  /spec   →   /plan   →   /build   →   /test   →   /review   →   /ship
```

---

## 2. 무엇이 들어 있나 (구성요소)

| 구성요소 | 개수 | 위치 | 역할 |
|---|---|---|---|
| 스킬 | 25개 (작업 스킬 24 + 메타 스킬 1) | `skills/` | 작업 절차서. 상황에 맞으면 자동 활성화 |
| 슬래시 명령어 | 9개 | `.claude/commands/` | 사용자가 직접 입력하는 진입점 (`/spec` 등) |
| 에이전트(페르소나) | 4개 | `agents/` | 리뷰어, 보안 감사관, 테스트 엔지니어, 웹 성능 감사관 |
| 참고 체크리스트 | 7개 | `references/` | 스킬이 필요할 때 참조하는 상세 체크리스트 |
| 선택형 훅 | 3종 | `hooks/` | **기본 비활성.** 직접 연결해야 동작 (10장) |
| 평가(evals) | 스킬당 케이스 파일 | `evals/` | 스킬이 제대로 발동하고 동작하는지 측정 |

참고 체크리스트 7개:

| 파일 | 내용 | 함께 쓰는 스킬 |
|---|---|---|
| `definition-of-done.md` | 모든 변경이 넘어야 하는 공통 완료 기준 | 전체 |
| `testing-patterns.md` | 테스트 구조, 이름, 모킹, React/API/E2E 예시 (JS/TS) | test-driven-development |
| `security-checklist.md` | 커밋 전 점검, 인증, 입력 검증, 헤더, CORS, OWASP Top 10 | security-and-hardening |
| `performance-checklist.md` | Core Web Vitals 목표, 프론트/백엔드 체크리스트 | performance-optimization |
| `accessibility-checklist.md` | 키보드 탐색, 스크린 리더, ARIA | frontend-ui-engineering |
| `observability-checklist.md` | 로깅, RED/USE 지표, 트레이싱, 알림 | observability-and-instrumentation |
| `orchestration-patterns.md` | 여러 에이전트를 조합하는 공인 패턴과 금지 패턴 | doubt-driven-development |

---

## 3. 설치 전 준비물

| 준비물 | 필요한 경우 | 확인 명령 |
|---|---|---|
| Claude Code | Claude Code에서 쓸 때 (가장 추천) | `claude --version` |
| Git | 마켓플레이스 등록 시 저장소를 내려받음 | `git --version` |
| Node.js | `npx skills` 방식으로 설치할 때 | `node --version` |
| jq, curl, bash | 선택형 훅을 쓸 때만 | `jq --version` |

> 💡 Windows에서 선택형 훅(bash 스크립트)을 쓰려면 Git Bash나 WSL이 필요합니다. 스킬과 명령어만 쓸 때는 필요 없습니다.

---

## 4. 설치하기

### 어떤 방법을 고를까?

| 방법 | 설치되는 것 | 추천 대상 |
|---|---|---|
| **A. Claude Code 플러그인** | 스킬 25 + 명령어 9 + 에이전트 4 + references | **Claude Code 사용자 (추천)** |
| B. `npx skills` | 스킬만 (명령어, 에이전트 없음) | 여러 AI 도구에 같이 깔고 싶을 때, 일부 스킬만 원할 때 |
| C. 수동 복사 (git clone) | 원하는 파일만 | 직접 관리하고 싶을 때 |
| D. 다른 도구 전용 | 도구별로 다름 | Codex, Gemini CLI, Cursor 등 |

---

### 방법 A. Claude Code 플러그인 (추천)

#### A-1. Claude Code 안에서 설치하기

1. 터미널에서 Claude Code를 실행합니다.
   ```bash
   claude
   ```
2. 마켓플레이스(플러그인 상점 목록)를 등록합니다.
   ```
   /plugin marketplace add addyosmani/agent-skills
   ```
3. 플러그인을 설치합니다.
   ```
   /plugin install agent-skills@addy-agent-skills
   ```
   - `agent-skills`는 **플러그인 이름**, `addy-agent-skills`는 **마켓플레이스 이름**입니다. 둘이 다르니 그대로 입력하세요.
4. 설치 범위를 묻는 화면이 나오면 고릅니다 (아래 표 참고).
5. Claude Code를 **재시작**합니다.

#### A-2. 터미널 명령으로 설치하기 (같은 결과, 실제 실행 확인)

```bash
claude plugin marketplace add addyosmani/agent-skills
claude plugin install agent-skills@addy-agent-skills                 # 기본: user 범위
claude plugin install agent-skills@addy-agent-skills --scope project # 이 프로젝트에만
```

실제 실행 결과:
```
√ Successfully added marketplace: addy-agent-skills (declared in user settings)
√ Successfully installed plugin: agent-skills@addy-agent-skills (scope: user)
```

#### 설치 범위(scope) 고르기

| 범위 | 적용 대상 | 추천 상황 |
|---|---|---|
| `user` (기본값) | 내 컴퓨터의 **모든 프로젝트** | 개인적으로 늘 쓸 때 |
| `project` | **현재 프로젝트**. 설정이 프로젝트의 `.claude/`에 저장돼서 git으로 팀과 공유 가능 | 팀 전체가 같은 규칙을 쓸 때 |
| `local` | 현재 프로젝트, **나만** (공유 안 됨) | 팀 공유 없이 이 프로젝트에서만 시험해 볼 때 |

#### SSH 오류가 날 때

`git@github.com: Permission denied (publickey)` 같은 오류가 나면 둘 중 하나를 쓰세요.

```
/plugin marketplace add https://github.com/addyosmani/agent-skills.git
```
또는 GitHub SSH 주소를 HTTPS로 바꿔 쓰도록 git을 설정합니다 (README 공식 우회법).
```bash
git config --global url."https://github.com/".insteadOf git@github.com:
```

---

### 방법 B. `npx skills` (여러 AI 도구 공용)

[vercel-labs/skills](https://github.com/vercel-labs/skills) CLI를 사용합니다. Claude Code, Cursor, Codex, Copilot, Cline 등 70개 이상의 에이전트에 스킬을 설치합니다.

```bash
# 설치 전에 어떤 스킬이 있는지 구경하기
npx skills add addyosmani/agent-skills --list

# 전체 25개 설치 (어느 에이전트에 넣을지 대화형으로 물어봄)
npx skills add addyosmani/agent-skills

# Claude Code에만, 확인 없이 설치 (실제 실행 확인)
npx skills add addyosmani/agent-skills -a claude-code -y

# 특정 스킬만 설치
npx skills add addyosmani/agent-skills --skill code-review-and-quality
npx skills add addyosmani/agent-skills --skill interview-me
npx skills add addyosmani/agent-skills --skill test-driven-development

# 모든 프로젝트에서 쓰도록 전역 설치
npx skills add addyosmani/agent-skills -g
```

실제로 설치해 보면 이렇게 됩니다.
- 프로젝트 폴더에 `.claude/skills/<스킬이름>/` 폴더 25개가 생깁니다.
- 어디서 무엇을 설치했는지 기록한 `skills-lock.json`이 생깁니다.

> ⚠️ **알아둘 점**
> - 이 방법은 **스킬만** 설치합니다. `/spec`, `/build` 같은 **슬래시 명령어와 에이전트 4개는 설치되지 않습니다.** 대신 "SPEC.md를 먼저 작성해줘"처럼 평소 말로 요청하면 스킬이 자동으로 발동합니다.
> - `--skill`로 하나만 설치하면 저장소 최상위의 `references/` 체크리스트가 복사되지 않습니다. 스킬은 동작하지만 체크리스트 경로가 깨집니다(공식 이슈 [#361](https://github.com/addyosmani/agent-skills/issues/361)). 필요하면 체크리스트를 스킬 폴더 안 `references/`에 직접 복사하세요.

---

### 방법 C. 수동 설치 (git clone)

```bash
git clone https://github.com/addyosmani/agent-skills.git ~/tools/agent-skills

# Claude Code를 이 폴더를 플러그인으로 불러와 실행 (개발/시험용)
claude --plugin-dir ~/tools/agent-skills
```

또는 필요한 스킬 폴더만 복사합니다.
```bash
mkdir -p .claude/skills
cp -R ~/tools/agent-skills/skills/test-driven-development .claude/skills/
```

> ⚠️ 저장소 최상위의 `AGENTS.md`와 `CLAUDE.md`는 **agent-skills 개발 기여자용 설정**입니다. 여러분 프로젝트에 복사하지 마세요 (`docs/getting-started.md` 명시).

---

### 방법 D. 다른 AI 도구 (공식 문서 요약)

| 도구 | 설치 명령 | 비고 |
|---|---|---|
| Codex CLI (v0.122+) | `codex plugin marketplace add addyosmani/agent-skills`<br>`codex plugin add agent-skills@agent-skills` | 스킬은 `@spec-driven-development`처럼 `@`로 호출 |
| Gemini CLI | `gemini skills install https://github.com/addyosmani/agent-skills.git --path skills` | `.gemini/commands/`에 명령어 9개 제공 |
| Antigravity CLI | `agy plugin install https://github.com/addyosmani/agent-skills.git` | 일부 버전에서 명령어 래퍼가 보이지 않음. 스킬을 직접 호출 |
| Cursor | 스킬을 `.cursor/skills/`에 복사, 짧은 규칙만 `.cursor/rules/*.mdc`에 | 스킬 전체를 rules에 붙여넣지 말 것 |
| GitHub Copilot | `agents/`를 페르소나로, 스킬 내용을 `.github/copilot-instructions.md`에 | Copilot CLI는 플러그인으로 설치 가능 |
| OpenCode | `.opencode/skills/` 또는 `~/.config/opencode/skills/`에 복사 | |
| Command Code | `cmd skills add addyosmani/agent-skills` | `--global`, `-s <스킬>` 지원 |
| Windsurf | 스킬 내용을 Windsurf rules 설정에 추가 | |
| Kiro | `.kiro/skills/` (프로젝트 또는 전역) | |
| Oh My Pi | `omp plugin marketplace add addyosmani/agent-skills` 후 `omp plugin install agent-skills@addy-agent-skills` | 메인테이너가 최신 버전으로 검증하지 않은 호스트 |

자세한 내용은 저장소 `docs/<도구>-setup.md`를 보세요.

---

## 5. 설치가 잘 됐는지 확인하기

### 5.1 플러그인 목록 확인

```bash
claude plugin list
```
```
  > agent-skills@addy-agent-skills
    Version: 0.6.12
    Scope: user
    Status: √ enabled
```
Claude Code 안에서는 `/plugin`을 입력해서 관리 화면을 열어도 됩니다.

### 5.2 구성요소와 토큰 비용 확인

```bash
claude plugin details agent-skills@addy-agent-skills
```
실제 출력 요약:
```
Component inventory
  Skills (34)  ...        ← 스킬 25개 + 명령어 9개 (명령어도 스킬 형태로 집계됨)
  Agents (4)   security-auditor, web-performance-auditor, code-reviewer, test-engineer
  Hooks (0)               ← 훅은 기본으로 연결되지 않음
  MCP servers (0)
Projected token cost
  Always-on:   ~3,622 tok   added to every session
```

### 5.3 실제로 동작하는지 시험

Claude Code에서 아래처럼 입력해 보세요.
```
/spec 할 일 목록 웹앱을 만들고 싶어
```
목표, 사용자, 기술 스택, 경계 조건 같은 **질문을 먼저 하면** 정상입니다. 질문 없이 바로 코드를 쓰기 시작하면 설치나 재시작 상태를 다시 확인하세요.

---

## 6. 기본 사용법: 9개 명령어

### 6.1 전체 흐름 한눈에 보기

```
/spec ──► SPEC.md 생성 (무엇을 만들까)
  │
/plan ──► tasks/plan.md, tasks/todo.md 생성 (어떻게 쪼갤까)
  │
/build ─► 작업 1개씩: 실패하는 테스트 → 구현 → 전체 테스트 → 빌드 → 커밋
  │        (/build auto = 계획 1회 승인 후 전체 작업 자동 진행)
/test ──► TDD / 버그 재현 테스트 (Prove-It 패턴)
  │
/review ► 5가지 축 코드 리뷰
  │
/ship ──► 리뷰어 + 보안 + 테스트 전문가 병렬 점검 → GO / NO-GO 판정
```

추가 명령어:
- `/constraints`: 프로젝트 품질 기준을 정하고 강제
- `/code-simplify`: 동작은 그대로 두고 코드 단순화
- `/webperf`: 웹 성능 감사

> 💡 **명령어 이름 충돌 시:** Claude Code에는 `/review`처럼 이름이 같은 다른 명령어나 다른 플러그인이 있을 수 있습니다. 엉뚱한 명령이 실행되면 플러그인 이름을 붙인 `/agent-skills:review` 형태로 입력하세요. (CLI의 `details` 출력에 명령어가 플러그인 스킬로 등록된 것을 확인했습니다. 충돌 시 어느 쪽이 우선하는지는 확인하지 못했습니다.)

### 6.2 `/spec`: 스펙(요구사항 문서) 작성

- **사용하는 스킬:** `spec-driven-development`
- **하는 일:**
  1. 질문을 먼저 합니다: 목표와 대상 사용자, 핵심 기능과 완료 기준, 기술 스택, 경계 조건(항상 할 것 / 먼저 물어볼 것 / 절대 하지 말 것)
  2. 6개 영역을 담은 스펙을 만듭니다: 목표, 명령어, 프로젝트 구조, 코드 스타일, 테스트 전략, 경계
  3. 독립적으로 테스트할 수 있는 기능이 여러 개 섞여 있으면 먼저 **기능 지도**(모듈, 의존 방향, 만드는 순서)를 제안하고 승인을 받습니다.
  4. 프로젝트 루트에 `SPEC.md`로 저장하고 확인을 받습니다.
- **입력 예시:**
  ```
  /spec 팀원들이 점심 메뉴를 투표하는 작은 웹앱. Next.js + SQLite로 하고 싶어.
  ```
- **쓰지 않아도 되는 경우:** 한 줄 수정, 오타 수정, 요구사항이 명확한 작은 변경

### 6.3 `/plan`: 작업 계획 세우기

- **사용하는 스킬:** `planning-and-task-breakdown`
- **하는 일:**
  1. `SPEC.md`와 관련 코드를 **읽기만 합니다** (계획 모드, 코드 수정 없음).
  2. 구성요소 사이의 의존 관계를 파악합니다.
  3. **세로로 자릅니다.** 화면, API, DB를 층별로 나누지 않고, 기능 하나가 처음부터 끝까지 동작하는 단위로 나눕니다.
  4. 작업마다 완료 기준과 검증 방법을 적고, 단계 사이에 체크포인트를 둡니다.
  5. `tasks/plan.md`(계획)와 `tasks/todo.md`(체크리스트)로 저장합니다.
- **안전장치:** 다른 작업의 미완료 계획 파일이 이미 있으면 덮어쓰지 않고 먼저 물어봅니다.

### 6.4 `/build`: 구현하기

- **사용하는 스킬:** `incremental-implementation` + `test-driven-development`
- **`/build` (기본, 작업 1개)** 의 순서:
  1. 다음 작업의 완료 기준을 읽음
  2. 관련 코드, 패턴, 타입 파악
  3. **실패하는 테스트 먼저 작성 (RED)**
  4. 테스트를 통과하는 최소한의 코드 작성 (GREEN)
  5. 전체 테스트로 회귀(기존 기능 고장) 확인
  6. 빌드 확인
  7. 커밋
  8. 작업 완료 표시 후 **멈춤**
- **`/build auto` (자동, 전체 작업)**:
  - 스펙이 정해진 위치(`SPEC.md`, `docs/SPEC.md`, `spec/`)에 있어야 합니다. 없으면 `/spec`부터 하라고 멈춥니다. README 같은 다른 문서는 스펙으로 인정하지 않습니다.
  - `git status`로 관련 없는 미커밋 변경이 있는지 확인하고, 있으면 멈춥니다.
  - 계획을 보여주고 **분명한 승인**("approve", "go", "yes")을 기다립니다. "괜찮아 보이네요" 같은 애매한 답은 승인으로 보지 않습니다. 사람이 개입하는 지점은 이 한 번뿐입니다.
  - 승인 후에는 작업마다 RED → GREEN → 회귀 → 빌드 → **작업당 커밋 1개**를 반복합니다. `git add -A`로 관계없는 파일까지 담지 않습니다.
  - 아래 상황에서는 **멈추고 사람에게 묻습니다.**
    - 테스트를 통과시킬 수 없거나 빌드가 깨질 때
    - 스펙이 모호할 때
    - 위험하거나 되돌릴 수 없는 작업: 인증·권한, 데이터 삭제 마이그레이션, 결제, 삭제, 배포, 비밀값, `git revert`로 되돌릴 수 없는 모든 것
  - 문제를 해결한 뒤 `/build auto`를 다시 입력하면 다음 작업부터 이어갑니다.
  - 끝나면 완료한 작업, 추가한 테스트, 커밋, 건너뛴 항목을 요약합니다.

> 💡 `/build auto`는 **작업당 속도가 빨라지는 기능이 아닙니다.** 작업 사이에 사람이 "다음"을 누르는 단계만 없앱니다. 검증은 그대로 수행합니다.

### 6.5 `/test`: 테스트 주도 개발

- **사용하는 스킬:** `test-driven-development`
- **새 기능:** 실패하는 테스트 작성 → 통과하도록 구현 → 테스트를 녹색으로 유지하며 정리(리팩터)
- **버그 수정 (Prove-It 패턴):**
  1. 버그를 재현하는 테스트 작성
  2. 실패하는지 확인
  3. 수정
  4. 통과하는지 확인
  5. 전체 테스트로 회귀 확인
- 브라우저 관련 문제면 `browser-testing-with-devtools`도 함께 사용합니다 (Chrome DevTools MCP 필요, 10.4절).
- **입력 예시:**
  ```
  /test 장바구니에 수량 0을 넣으면 합계가 NaN이 되는 버그
  ```

### 6.6 `/review`: 5가지 축 코드 리뷰

- **사용하는 스킬:** `code-review-and-quality`
- **검토 축:**
  1. **정확성:** 스펙과 맞는가, 예외 상황, 테스트 충분성
  2. **가독성:** 이름, 로직의 단순함, 구성
  3. **아키텍처:** 기존 패턴, 경계, 추상화 수준
  4. **보안:** 입력 검증, 비밀값, 권한
  5. **성능:** N+1 쿼리, 무제한 반복
- **결과:** Critical / Important / Suggestion으로 분류하고, `파일:줄` 위치와 수정 제안을 붙입니다.
- 스테이징된 변경이나 최근 커밋을 대상으로 합니다. diff를 대화에 붙여넣어도 됩니다.

### 6.7 `/ship`: 배포 전 최종 점검

- **사용하는 스킬:** `shipping-and-launch`
- **동작:**
  - **A단계 (병렬):** `code-reviewer`, `security-auditor`, `test-engineer` 서브에이전트 3개를 **동시에** 실행합니다.
  - **B단계 (종합):** 메인 에이전트가 코드 품질, 보안, 성능, 접근성, 인프라(환경변수, 마이그레이션, 모니터링, 기능 플래그), 문서를 종합합니다.
  - **C단계 (판정):** `GO` 또는 `NO-GO`를 결정하고 차단 이슈, 권장 수정, 감수하는 위험, **롤백 계획**을 냅니다.
- **규칙:**
  - 롤백 계획 없이는 GO가 나오지 않습니다.
  - Critical 발견이 있으면 사용자가 위험을 명시적으로 수락하지 않는 한 기본값은 NO-GO입니다.
  - 병렬 점검은 아래 조건을 **모두** 만족할 때만 생략합니다: 2개 파일 이하, diff 50줄 미만, 인증·결제·데이터 접근·설정 미변경
- **팁:** 내 `.claude/agents/` 또는 `~/.claude/agents/`에 같은 이름의 에이전트를 만들면 플러그인 버전 대신 내 버전이 사용됩니다 (10.2절).

### 6.8 `/constraints`: 품질 기준 정하기

- **사용하는 스킬:** `constraint-driven-development`
- **하는 일:**
  1. 프로젝트 파일(package.json, CI 설정, 린트, 커버리지)을 먼저 읽어서, 읽어서 알 수 있는 것은 묻지 않습니다.
  2. 질문은 **최대 4개**, 하나씩, 추천 기본값과 함께 합니다. "모르겠어요"라고 답해도 동작하는 설정이 나옵니다.
  3. `CONSTRAINTS.md`를 작성합니다: 항상 강제하는 최저선, 숫자로 강제할 항목, 측정만 할 항목, 예외(담당자, 만료일)
  4. 검사 도구를 설치합니다: Semgrep, gitleaks(`--redact`), osv-scanner, axe-core, Lighthouse, size-limit, dependency-cruiser, Stryker 등. package.json에 `check:fast`, `check:task`, `check:full` 스크립트를 추가합니다.
  5. 검사를 비용별로 배치합니다: 타입·린트·비밀값은 편집할 때마다, 관련 테스트는 작업 끝에(90초 이내), 나머지는 리뷰나 CI에서
  6. `CLAUDE.md`에 "CONSTRAINTS.md를 읽고, 통과시키려고 기준을 낮추지 말 것"을 추가합니다.
- **하위 명령:**
  - `/constraints check`: 현재 브랜치를 기준에 맞춰 검사
  - `/constraints guard`: diff에서 기준이 약해진 흔적 탐지 (임계값 하향, 테스트 삭제·스킵, `@ts-ignore`·`eslint-disable` 추가, 미완성 stub)
  - `/constraints ratchet`: 오늘 측정값을 "더 떨어지면 안 되는 최저선"으로 기록
- **추천 시점:** `/build auto`나 자동 루프를 돌리기 **전**에 하세요. 에이전트가 테스트도 직접 작성하므로, 그 테스트가 유일한 방어선이 되는 상황을 막아 줍니다.

### 6.9 `/code-simplify`: 코드 단순화

- **사용하는 스킬:** `code-simplification`
- **대상:** 최근 변경한 코드, 또는 지정한 범위
- **단순화 항목:** 깊은 중첩 → 조기 반환, 긴 함수 → 책임별 분리, 중첩 삼항 연산자 → if/switch, 모호한 이름 → 구체적 이름, 중복 → 공통 함수, 죽은 코드 → 확인 후 삭제
- **안전장치:** 바꿀 때마다 테스트를 돌리고, 실패하면 그 변경을 되돌립니다.
- **원칙:** Chesterton's Fence. 왜 있는지 모르는 코드는 이해하기 전에 지우지 않습니다.

### 6.10 `/webperf`: 웹 성능 감사

- **사용하는 에이전트:** `web-performance-auditor`
- 웹 애플리케이션 전용입니다. 라이브러리, CLI, 서버 전용 코드에는 쓰지 마세요.
- **두 가지 모드:**
  - **Quick (기본):** 소스 코드에서 구조적 문제를 찾습니다. 모든 결과에 "potential impact(잠재적 영향)" 표시가 붙습니다.
  - **Deep:** Lighthouse JSON, PageSpeed Insights, CrUX 데이터, DevTools 트레이스, 또는 Chrome DevTools MCP가 연결된 실제 URL이 있을 때 실측값으로 감사합니다.
  ```bash
  # Deep 모드용 Lighthouse 보고서 만들기
  npx lighthouse https://localhost:3000 --output json --output-path ./report.json
  ```
  ```
  /webperf 상품 상세 페이지. 보고서는 ./report.json
  ```
- 실측 근거가 있는 값만 점수표에 넣습니다 (metric-honesty 규칙).

### 6.11 명령어 없이 쓰기 (자동 활성화)

명령어를 몰라도 됩니다. 평소처럼 말하면 Claude Code가 description을 보고 맞는 스킬을 골라 씁니다.

| 이렇게 말하면 | 발동하는 스킬 (예상) |
|---|---|
| "로그인 API 설계해줘" | api-and-interface-design |
| "이 버튼 컴포넌트 접근성 맞춰서 만들어줘" | frontend-ui-engineering |
| "테스트가 갑자기 깨졌어" | debugging-and-error-recovery |
| "이 함수 너무 복잡해, 정리해줘" | code-simplification |
| "나한테 질문해서 요구사항 정리해줘" / "grill me" | interview-me |
| "이 아이디어 다듬어줘" | idea-refine |

특정 스킬을 확실히 쓰고 싶으면 이름을 직접 말하세요.
```
test-driven-development 스킬 절차대로 이 버그를 고쳐줘.
```

> ⚠️ 자동 선택은 description을 근거로 한 확률적인 판단이라 항상 원하는 스킬이 고르지는 않습니다. 중요한 작업은 명령어나 스킬 이름으로 명시하세요.

---

## 7. 25개 스킬 전체 목록

`호출 예시`는 자연어 요청 예시입니다. 스킬 이름을 직접 말해도 됩니다.

### 메타 (스킬 고르기)

| 스킬 | 하는 일 | 언제 |
|---|---|---|
| `using-agent-skills` | 작업을 맞는 스킬로 안내하는 순서도 + 공통 행동 규칙 6가지 (아래 참고) | 세션 시작 시, 어떤 스킬을 쓸지 모를 때 |

공통 행동 규칙 6가지:
1. **가정을 드러내라:** "제가 가정하는 것: … 지금 정정해 주세요"
2. **혼란을 적극적으로 관리하라:** 모순을 발견하면 멈추고 질문
3. **필요하면 반대하라:** 무조건 "네"라고 하지 않음. 비용을 수치로 제시
4. **단순함을 강제하라:** 100줄이면 될 것을 1000줄로 만들면 실패
5. **범위를 지켜라:** 요청받지 않은 정리, 리팩터, 삭제 금지
6. **확인하라, 가정하지 마라:** 증거(테스트, 빌드, 실행 데이터) 필수

> ⚠️ Claude Code처럼 **자체적으로 스킬을 고르는 도구**에서는 이 메타 스킬을 항상 켜 둔 규칙 파일(CLAUDE.md 등)에 붙여넣지 마세요. 스킬 선택기가 둘이 되어 충돌합니다 (공식 문서 명시). 플러그인으로 설치하면 다른 스킬처럼 필요할 때만 로드되므로 괜찮습니다.

### DEFINE: 무엇을 만들지 정하기

| 스킬 | 하는 일 | 언제 | 호출 예시 |
|---|---|---|---|
| `interview-me` | 질문을 한 번에 하나씩 해서, 사용자가 "원해야 한다고 생각하는 것"이 아니라 **실제로 원하는 것**을 약 95% 확신이 들 때까지 파악 | 요청이 막연할 때 ("X 만들어줘"인데 누구를, 왜가 없을 때) | "만들기 전에 나 인터뷰해줘" |
| `idea-refine` | 발산(아이디어 넓히기) → 수렴(검증하고 좁히기) → 1페이지 문서(`docs/ideas/<이름>.md`) | 아이디어가 아직 흐릿할 때 | "이 아이디어 다듬어줘", "내 계획 스트레스 테스트해줘" |
| `spec-driven-development` | 코드 전에 PRD(목표, 명령어, 구조, 스타일, 테스트, 경계) 작성 | 새 프로젝트·기능, 30분 이상 걸릴 변경 | `/spec` |
| `constraint-driven-development` | 품질 기준을 `CONSTRAINTS.md`로 문서화하고, 기준을 몰래 낮추는 행위 감시 | 품질 기준이 문서로 없을 때, 자동 루프 전 | `/constraints` |

### PLAN: 계획 세우기

| 스킬 | 하는 일 | 언제 | 호출 예시 |
|---|---|---|---|
| `planning-and-task-breakdown` | 스펙을 작고 검증 가능한 작업으로 나누고 의존 순서 정리 | 스펙이 있고 구현 단위가 필요할 때, 병렬 작업 계획 | `/plan` |

### BUILD: 구현하기

| 스킬 | 하는 일 | 언제 | 호출 예시 |
|---|---|---|---|
| `incremental-implementation` | 얇은 세로 조각 단위로 구현 → 테스트 → 검증 → 커밋. 기능 플래그, 안전한 기본값 | 파일 2개 이상을 건드리는 변경, 한 번에 100줄 넘게 쓰고 싶을 때 | `/build` |
| `test-driven-development` | Red-Green-Refactor, 테스트 피라미드(80/15/5), DAMP > DRY, Beyonce Rule | 로직 구현, 버그 수정, 동작 변경 | `/test` |
| `context-engineering` | 규칙 파일, 컨텍스트 구성, 세션 재시작 경계, 컨텍스트 예산 관리 | 세션 시작, 결과물 품질이 떨어질 때, 작업 전환 | "이 프로젝트용 CLAUDE.md 만들어줘" |
| `source-driven-development` | 프레임워크 관련 결정을 **공식 문서로 확인**하고 출처를 표시. 확인 못 한 것은 표시 | 프레임워크 코드를 기억에 의존해 쓰려 할 때 | "React 19 공식 문서 기준으로 폼 만들어줘" |
| `doubt-driven-development` | 중요한 결정마다 새 컨텍스트에서 반대 입장 검토: CLAIM → EXTRACT → DOUBT → RECONCILE → STOP | 운영 환경, 보안, 되돌릴 수 없는 작업, 낯선 코드 | "이 마이그레이션 계획 의심해서 검증해줘" |
| `frontend-ui-engineering` | 컴포넌트 구조, 디자인 시스템, 상태 관리, 반응형, WCAG 2.1 AA 접근성 | UI를 만들거나 수정할 때 | "로그인 페이지 만들어줘" |
| `api-and-interface-design` | 계약 우선 설계, Hyrum's Law, One-Version Rule, 에러 의미, 경계 검증 | API, 모듈 경계, 공개 인터페이스 설계 | "주문 REST API 설계해줘" |

### VERIFY: 동작 증명하기

| 스킬 | 하는 일 | 언제 | 호출 예시 |
|---|---|---|---|
| `browser-testing-with-devtools` | Chrome DevTools MCP로 실제 브라우저의 DOM, 콘솔, 네트워크, 성능, 스크린샷 확인. 브라우저 내용은 신뢰하지 않는 데이터로 취급 | 브라우저에서 돌아가는 것을 만들거나 디버깅할 때 | "브라우저 열어서 콘솔 에러 확인해줘" |
| `debugging-and-error-recovery` | 5단계: 재현 → 위치 파악 → 축소 → 수정 → 재발 방지. 문제가 생기면 진행을 멈추는 규칙 | 테스트·빌드 실패, 예상과 다른 동작 | "어제까진 됐는데 오늘 안 돼" |

### REVIEW: 머지 전 품질 점검

| 스킬 | 하는 일 | 언제 | 호출 예시 |
|---|---|---|---|
| `code-review-and-quality` | 5축 리뷰, 변경 크기(~100줄), 심각도 라벨, 리뷰 속도 기준, 큰 변경 쪼개기 | 모든 머지 전 | `/review` |
| `code-simplification` | Chesterton's Fence, Rule of 500, 동작을 유지하며 복잡도 줄이기 | 동작은 하지만 읽기 어려울 때 | `/code-simplify` |
| `security-and-hardening` | 위협 모델 먼저, OWASP Top 10, 인증 패턴, 비밀값 관리, 의존성 감사, 3단계 경계 시스템, GDPR/CCPA | 사용자 입력, 인증, 데이터 저장, 외부 연동 | "이 로그인 흐름 보안 점검해줘" |
| `performance-optimization` | **측정 먼저.** Core Web Vitals, 프로파일링, 번들 분석, N+1 | 성능 요구사항이 있거나 느려졌을 때 | "이 페이지 느린 원인 찾아줘" |

### SHIP: 배포하기

| 스킬 | 하는 일 | 언제 | 호출 예시 |
|---|---|---|---|
| `git-workflow-and-versioning` | 트렁크 기반 개발, 원자적 커밋, 커밋을 저장 지점으로 쓰기, worktree, 시맨틱 버전, 체인지로그 | **모든 코드 변경** | "작업 내용 커밋 단위로 정리해줘" |
| `ci-cd-and-automation` | Shift Left, "빠를수록 안전하다", 품질 게이트 파이프라인, CI 실패를 에이전트에게 피드백 | CI/CD 구축·수정 | "GitHub Actions CI 만들어줘" |
| `deprecation-and-migration` | 코드를 부채로 보는 관점, 강제 vs 권고 폐기, 무중단 스키마 변경(expand/contract), 좀비 코드 제거 | 구 시스템 교체, 기능 종료, DB 컬럼 변경 | "이 컬럼 무중단으로 이름 바꾸는 계획 세워줘" |
| `documentation-and-adrs` | ADR(아키텍처 결정 기록), API 문서, **왜**를 기록하는 주석 | 중요한 설계 결정, 공개 API 변경 | "DB 선택 이유를 ADR로 남겨줘" |
| `observability-and-instrumentation` | 구조화 로깅, RED 지표, OpenTelemetry 트레이싱, 증상 기반 알림 | 운영 환경에서 돌아갈 기능 | "이 결제 API에 로깅이랑 메트릭 붙여줘" |
| `shipping-and-launch` | 출시 전 체크리스트, 기능 플래그 수명주기, 단계적 배포, 롤백 절차, 에러 예산 게이트 | 운영 배포 준비 | `/ship` |

---

## 8. 에이전트(전문가 페르소나) 4개

### 8.1 목록

| 에이전트 | 역할 | 관점 |
|---|---|---|
| `code-reviewer` | 시니어 스태프 엔지니어 | "스태프 엔지니어가 승인할까?" 기준의 5축 리뷰 |
| `test-engineer` | QA 전문가 | 테스트 전략, 커버리지 분석, Prove-It 패턴 |
| `security-auditor` | 보안 엔지니어 | 취약점 탐지, 위협 모델링, OWASP 평가 |
| `web-performance-auditor` | 웹 성능 엔지니어 | Core Web Vitals 감사 (Quick/Deep 모드) |

### 8.2 개념 정리

| 층 | 의미 | 예시 | 비유 |
|---|---|---|---|
| 스킬 | 단계와 완료 조건이 있는 절차 | code-review-and-quality | **어떻게** 할지 |
| 에이전트 | 하나의 역할과 관점, 출력 형식 | code-reviewer | **누가** 할지 |
| 명령어 | 사용자가 입력하는 진입점 | `/review`, `/ship` | **언제** 할지 |

### 8.3 직접 호출하기

Claude Code에서 평소 말로 요청합니다.
```
code-reviewer 에이전트로 이번 변경 리뷰해줘
security-auditor 에이전트로 src/auth.ts 보안 점검해줘
test-engineer 에이전트로 결제 흐름에 빠진 테스트 찾아줘
```

### 8.4 규칙

- **에이전트는 다른 에이전트를 부르지 않습니다.** 조합은 사용자나 명령어(`/ship`)가 합니다. Claude Code에서도 "서브에이전트는 다른 서브에이전트를 만들 수 없다"는 제약이 있습니다.
- 공식으로 인정하는 조합 패턴은 `/ship`처럼 **서로 독립적인 점검을 병렬로 돌리고 메인 에이전트가 합치는 방식** 하나뿐입니다.
- "어떤 에이전트를 부를지 결정하는 에이전트" 같은 중간 관리자 구조는 금지 패턴입니다. 정보가 손실되고 토큰이 두 배로 듭니다.

---

## 9. 응용: 상황별 실전 시나리오

> 공식 사이트의 튜토리얼 3개(새 앱 / 기존 코드 / 루프)와 같은 주제를 저장소 문서(`adoption-guide.md`, `getting-started.md`) 기준으로 정리했습니다. 사이트 튜토리얼의 원문 프롬프트는 직접 확인하지 못했습니다.

### 시나리오 1. 빈 폴더에서 새 앱 만들기 (Greenfield)

```bash
mkdir habit-tracker && cd habit-tracker && git init
claude
```
```
1) /spec 매일 습관을 등록하고, 완료 체크하고, 연속 달성일(streak)을 보는 웹앱
   → 질문에 답하고 SPEC.md 확인 후 승인

2) /constraints
   → 품질 기준 설정 (모르면 기본값 수락)

3) /plan
   → tasks/plan.md 검토. 작업이 너무 크면 "3번 작업 더 쪼개줘"

4) /build auto
   → 계획을 보고 "approve"라고 입력. 이후 작업별 TDD와 커밋이 자동 진행

5) /review
6) /ship
```

처음부터 늘 켜 둘 것: TDD, git 워크플로, 보안, ADR. 프로젝트가 자라면서 추가할 것:

| 시점 | 추가 스킬 |
|---|---|
| 첫 공개 API | api-and-interface-design |
| 첫 UI | frontend-ui-engineering (+ browser-testing-with-devtools) |
| 첫 CI | ci-cd-and-automation |
| 첫 운영 배포 | observability-and-instrumentation, shipping-and-launch |
| 성능 요구 발생 | performance-optimization |

**피해야 할 것:** "프로토타입이니까" 스펙 건너뛰기, 모든 스킬을 규칙 파일에 상시 로드, 관측성(로깅·메트릭)을 나중으로 미루기

### 시나리오 2. 남이 만든 기존 코드에 기능 추가 (Brownfield)

**핵심: 바꾸기 전에 먼저 읽고 보호하라.** 4단계로 천천히 도입합니다.

| 단계 | 목표 | 사용할 스킬 |
|---|---|---|
| 1. 맥락 파악 (읽기 전용) | 에이전트가 코드를 이해 | context-engineering(실제 규칙을 CLAUDE.md에), code-review-and-quality, debugging-and-error-recovery, doubt-driven-development |
| 2. 변경 전 테스트 | 건드릴 곳에 안전망 | test-driven-development(**특성화 테스트**: 현재 동작을 맞든 틀리든 그대로 고정하는 테스트), code-simplification, git-workflow-and-versioning |
| 3. 새 작업은 전체 사이클 | 신규 기능만 정식 절차 | `/spec` → `/plan` → `/build` → `/review`, api-and-interface-design(신구 코드 경계), security-and-hardening |
| 4. 부채 정리 | 레거시 축소 | deprecation-and-migration, observability-and-instrumentation, performance-optimization |

예시 프롬프트:
```
이 저장소 구조와 실제 코딩 관례를 파악해서 CLAUDE.md 초안을 만들어줘.
빌드/테스트 명령어, 디렉터리 의미, 건드리면 위험한 곳을 포함해. 코드는 수정하지 마.
```
```
/spec 주문 목록에 CSV 내보내기 추가.
경계: legacy/billing 폴더는 수정 금지, 기능 플래그 뒤에 배포.
```

**가장 비싼 실수:** 테스트 없는 레거시 코드를 리팩터링하는 것. **특성화 테스트 없이는 리팩터링하지 않는다**가 원칙입니다.

### 시나리오 3. 버그 하나 고치기

```
/test 회원가입 시 이메일에 대문자가 있으면 중복 체크를 통과하는 버그
```
→ 재현 테스트(실패 확인) → 수정 → 통과 → 전체 회귀 테스트

원인을 모르면:
```
테스트가 갑자기 실패해. debugging-and-error-recovery 절차로 원인 찾아줘.
```

### 시나리오 4. 리뷰만 받기

```
/review
```
또는 다른 사람의 PR diff를 붙여넣고:
```
code-reviewer 에이전트로 아래 diff 리뷰해줘
<diff 붙여넣기>
```

### 시나리오 5. 여러 세션에 걸쳐 큰 작업 이어가기

대화 내용이 아니라 **파일**이 작업을 이어주는 연결고리입니다.

**세션을 바꾸기 전** (요청 예시):
```
지금까지 결정 사항, 승인된 범위, 남은 질문, 다음 작업, 검증 상태(어떤 테스트를 무엇 기준으로 돌렸는지)를
SPEC.md와 tasks/todo.md에 반영해줘.
```

**새 세션에서** (공식 문서의 프롬프트):
```
Read SPEC.md, tasks/plan.md and tasks/todo.md, then check where things actually stand —
git status, plus re-running whatever checks the recorded verification state no longer covers.
Tell me the next unchecked task and anything still open, then stop:
I'll confirm the scope before you start it.
```
한국어로:
```
SPEC.md, tasks/plan.md, tasks/todo.md를 읽고 git status로 실제 상태를 확인해.
기록된 검증이 지금 코드에 안 맞으면 다시 돌려.
다음 미완료 작업과 남은 질문을 알려주고 멈춰. 범위는 내가 확인한 다음 시작해.
```

**원칙:**
- 기록된 "테스트 통과"는 **특정 시점 기준의 주장**입니다. 코드가 바뀌었으면 다시 확인합니다.
- 이전 대화에서 승인했다고 가정하지 않습니다. 파일에 기록된 승인만 인정합니다.
- 큰 작업은 단계마다 새 세션을 쓰면 컨텍스트가 깔끔하게 유지됩니다 (spec → plan → build → review).

### 시나리오 6. 자동 루프 (밤새 개선, 아침에 승인)

공식 사이트의 `loop-engineering` 튜토리얼 주제입니다. 저장소 문서 기준 원칙:
- `/build auto`는 한 세션 안에서 전체 계획을 실행합니다. 작업마다 상태 갱신, 검증, 커밋을 남기므로 **각 완료 작업이 재시작 가능한 경계**가 됩니다.
- 셸에서 반복 실행하는 "Ralph loop"는 스킬이 아니라 **실행 환경(하네스)의 동작**입니다. 쓴다면:
  - 기록된 작업 경계에서만 재시작합니다.
  - 재시작하면 대화 기록이 아니라 파일과 저장소 상태를 읽습니다.
  - 프로세스가 종료됐다고 작업이 통과한 것은 아닙니다.
  - 재시작이 승인 단계를 건너뛰면 안 됩니다.
- 사전 준비: `/constraints`로 기준을 정하고 `/constraints guard`로 감시합니다. 에이전트가 테스트를 지우거나 기준을 낮춰서 통과시키는 것을 막기 위해서입니다.

> ⚠️ 무인 자동화는 비용(토큰)과 위험이 큽니다. 처음에는 사람이 지켜보는 상태에서 `/build auto`부터 익히세요.

---

## 10. 응용: 고급 설정과 커스터마이즈

### 10.1 프로젝트 규칙 파일과 함께 쓰기

```markdown
# CLAUDE.md (예시)
## 스택
Next.js 15, TypeScript, Prisma, PostgreSQL

## 명령어
- 테스트: pnpm test
- 타입체크: pnpm typecheck
- 빌드: pnpm build

## 경계
- 항상: 변경마다 테스트 추가, CONSTRAINTS.md 준수
- 먼저 물어볼 것: DB 스키마 변경, 새 의존성 추가
- 절대 금지: .env 커밋, 실패하는 테스트 삭제

## 작업 방식
- 기능 작업은 /spec → /plan → /build 순서
- CONSTRAINTS.md를 읽고, 통과시키려고 기준을 낮추지 말 것
```

### 10.2 에이전트를 내 방식으로 바꾸기

플러그인 에이전트는 Claude Code에서 **우선순위가 가장 낮습니다.** 같은 이름으로 내 에이전트를 만들면 내 것이 이깁니다. `/ship`도 자동으로 내 버전을 사용합니다.

```bash
mkdir -p .claude/agents
# 플러그인 설치 위치에서 원본을 복사해 수정 (버전 폴더명은 설치 버전에 따라 다름)
cp ~/.claude/plugins/cache/addy-agent-skills/agent-skills/0.6.12/agents/code-reviewer.md .claude/agents/
# 원하는 대로 수정: 예) "모든 리뷰는 한국어로", "우리 팀 컨벤션 문서 X를 기준으로"
```

### 10.3 스킬을 로컬 사본으로 커스터마이즈 (Claude Code 전용 필드)

공개 `SKILL.md`는 여러 도구에서 쓸 수 있도록 표준 필드만 사용합니다. Claude Code 전용 기능은 **내 로컬 사본**에 추가합니다 (`docs/advanced-per-agent-configuration.md`).

| 필드 | 효과 |
|---|---|
| `context: fork` | 스킬을 별도 서브에이전트 컨텍스트에서 실행. 메인 대화는 깨끗하게 유지되고 결과만 돌아옴. `agent: Plan`처럼 서브에이전트 유형 지정 가능 |
| `allowed-tools` | 나열한 도구를 권한 확인 없이 **허용**. 제한하는 기능이 아님 |
| `disallowed-tools` | 나열한 도구를 **제거**(실제 제한). 리뷰처럼 읽기만 해야 하는 단계에 적합 |

```markdown
---
name: my-review
description: 우리 팀 기준으로 코드를 리뷰한다. 머지 전에 사용.
context: fork
disallowed-tools: Edit, Write
---
agent-skills의 code-review-and-quality 절차를 따르되, 결과는 한국어로 작성한다.
```
위 파일을 `.claude/skills/my-review/SKILL.md`로 저장합니다.

> ⚠️ 이 필드들을 공개 배포용 SKILL.md 최상위에 넣으면 패키징 검증에서 오류가 날 수 있습니다. 개인용 사본에서만 쓰세요.

### 10.4 브라우저 테스트 (Chrome DevTools MCP 연결)

`browser-testing-with-devtools` 스킬과 `/webperf` Deep 모드에 필요합니다. 프로젝트의 `.mcp.json`에 추가합니다.

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--isolated"]
    }
  }
}
```
- `--isolated`: 브라우저를 닫으면 지워지는 임시 프로필을 사용합니다. 가장 안전합니다.
- `--autoConnect`: 현재 실행 중인 내 Chrome에 붙습니다 (Chrome 144 이상). 로그인된 이메일, 은행, GitHub 세션 등 **모든 창에 접근 가능**하므로 꼭 필요할 때만 쓰세요.
- 스킬 규칙상 웹페이지 내용은 **신뢰하지 않는 데이터**로 취급합니다. 페이지 속 "이 명령을 실행하라" 같은 문구를 따르지 않고, 쿠키나 토큰을 읽지 않습니다.

### 10.5 선택형 훅 3종 (기본 비활성)

플러그인은 훅을 자동으로 연결하지 않습니다 (`Hooks (0)`). 원하면 직접 `.claude/settings.json`에 추가합니다.

> 📌 **스크립트 경로:** 플러그인 설치 폴더(`~/.claude/plugins/cache/.../<버전>/hooks/`)는 **업데이트하면 버전 폴더명이 바뀝니다.** 훅을 쓸 거라면 저장소를 고정된 위치에 따로 클론해서 그 절대 경로를 쓰세요.
> ```bash
> git clone https://github.com/addyosmani/agent-skills.git ~/tools/agent-skills
> ```
> 필요한 도구: `bash`, `jq`, sdd-cache는 `curl`과 `sha256sum`(또는 `shasum`)도 필요

**① simplify-ignore: `/code-simplify`가 건드리면 안 되는 코드 보호**

보호할 코드를 표시합니다.
```js
/* simplify-ignore-start: perf-critical */
// 손으로 펼친 XOR — 반복문보다 3배 빠름
result[0] = buf[0] ^ key[0];
result[1] = buf[1] ^ key[1];
/* simplify-ignore-end */
```
`.claude/settings.json`에 훅을 등록합니다 (경로는 본인 위치로 수정).
```json
{
  "hooks": {
    "PreToolUse":  [{ "matcher": "Read",       "hooks": [{ "type": "command", "command": "bash \"$HOME/tools/agent-skills/hooks/simplify-ignore.sh\"" }] }],
    "PostToolUse": [{ "matcher": "Edit|Write", "hooks": [{ "type": "command", "command": "bash \"$HOME/tools/agent-skills/hooks/simplify-ignore.sh\"" }] }],
    "Stop":        [{ "hooks": [{ "type": "command", "command": "bash \"$HOME/tools/agent-skills/hooks/simplify-ignore.sh\"" }] }]
  }
}
```
- 모델은 보호된 블록 대신 `/* BLOCK_xxxxxxxx: perf-critical */` 자리표시자만 봅니다. 편집이 끝나면 원래 코드로 복원됩니다.
- `.claude/.simplify-ignore-cache/`를 `.gitignore`에 추가하세요.

**② sdd-cache: 공식 문서 조회(WebFetch) 캐시**

`source-driven-development`가 같은 문서를 반복해서 가져오는 낭비를 줄입니다. 저장된 내용을 쓸 때마다 서버에 "바뀌었나?"를 물어보고(`ETag`, `Last-Modified`), **304(변경 없음)** 응답일 때만 캐시를 사용하므로 오래된 문서를 쓸 위험이 없습니다.
```json
{
  "hooks": {
    "PreToolUse":  [{ "matcher": "WebFetch", "hooks": [{ "type": "command", "command": "bash \"$HOME/tools/agent-skills/hooks/sdd-cache-pre.sh\"",  "timeout": 10 }] }],
    "PostToolUse": [{ "matcher": "WebFetch", "hooks": [{ "type": "command", "command": "bash \"$HOME/tools/agent-skills/hooks/sdd-cache-post.sh\"", "async": true, "timeout": 10 }] }]
  }
}
```
- `.claude/sdd-cache/`를 `.gitignore`에 추가하세요.
- 캐시를 사용할 때 WebFetch가 "오류"처럼 막히는 것은 의도된 동작입니다. 캐시된 내용이 그 오류 메시지 안에 담겨 에이전트에게 전달됩니다.

**③ session-start: 메타 스킬 자동 주입**

**Claude Code와 Codex에서는 쓰지 마세요.** 이 도구들은 스스로 스킬을 고르므로, 이 훅을 켜면 스킬 선택기가 둘이 됩니다. Gemini CLI처럼 자체 스킬 선택 기능이 없는 도구 전용입니다.

### 10.6 필요한 스킬만 골라 쓰기 (토큰 절약)

플러그인은 25개를 한꺼번에 설치합니다. 몇 개만 원하면 `npx skills`로 골라서 설치하세요.
```bash
# 공식 문서가 추천하는 최소 3종
npx skills add addyosmani/agent-skills -a claude-code -y \
  --skill spec-driven-development \
  --skill test-driven-development \
  --skill code-review-and-quality
```
공백으로 나열하는 형식(`--skill idea-refine interview-me`)도 동작합니다. 두 형식 모두 직접 실행해서 확인했습니다.

### 10.7 나만의 스킬 만들기

agent-skills의 형식(`docs/skill-anatomy.md`)을 따르면 됩니다.
```bash
mkdir -p .claude/skills/our-release-process
```
```markdown
---
name: our-release-process
description: 우리 팀의 릴리스 절차를 안내한다. Use when 릴리스 태그를 만들거나 배포 노트를 작성할 때.
---

# Our Release Process

## Overview
## When to Use
## Process
1. ...
## Common Rationalizations
| 핑계 | 반박 |
|---|---|
| "작은 변경이라 체인지로그는 생략" | 작은 변경이 장애 원인을 찾는 단서가 된다 |
## Red Flags
## Verification
- [ ] ...
```
규칙:
- `name`은 소문자와 하이픈만 사용하고 폴더명과 같아야 합니다.
- `description`에는 **무엇을 하는지 + 언제 쓰는지("Use when…")** 를 담고, 1024자 이하로 씁니다.
- description에 작업 단계를 요약하지 마세요. 에이전트가 본문 대신 요약만 보고 따라 할 수 있습니다.
- `npx skills init my-skill` 명령으로 뼈대를 만들 수도 있습니다 (skills CLI 기능).

### 10.8 스킬 평가하기 (고급)

저장소를 클론하면 스킬이 잘 발동하는지 측정할 수 있습니다.
```bash
cd ~/tools/agent-skills
node scripts/run-evals.js                       # 2단계: 발동·라우팅 검사 (토큰 소모 없음)
node scripts/run-evals.js --behavioral test-driven-development --dry-run   # 3단계 계획만 출력
node scripts/run-evals.js --behavioral test-driven-development             # 3단계 실제 실행 (토큰 소모)
```

| 단계 | 검사 내용 | 비용 |
|---|---|---|
| 1. 구조 | 형식, 이름, 필수 섹션 | 무료 |
| 2. 발동·라우팅 | 맞는 요청에 맞는 스킬이 1순위로 뽑히는지, 설명끼리 겹치지 않는지 (TF-IDF 근사) | 무료 |
| 3. 행동 | 실제로 Claude를 실행해서 스킬이 약속한 대로 행동하는지 채점 | 토큰 소모 |

Claude Code 내장 평가도 있습니다: `claude plugin eval agent-skills@addy-agent-skills`. 신뢰하는 플러그인에만 사용하세요.

---

## 11. 업데이트하기

### Claude Code 플러그인 (실제 실행 확인)

```bash
claude plugin marketplace update addy-agent-skills   # 1) 목록 갱신
claude plugin update agent-skills@addy-agent-skills  # 2) 플러그인 갱신
```
```
√ Successfully updated marketplace: addy-agent-skills
√ agent-skills is already at the latest version (0.6.12).
```
- 업데이트를 적용하려면 Claude Code를 **재시작**해야 합니다 (CLI 도움말 명시).
- `project`나 `local` 범위로 설치했다면 `--scope project`처럼 범위를 지정하세요.
- Claude Code 안에서는 `/plugin` 관리 화면에서도 할 수 있습니다.

### `npx skills` (실제 실행 확인)

```bash
npx skills update            # 범위를 물어봄
npx skills update -p -y      # 프로젝트 범위, 확인 없이
npx skills update -g -y      # 전역 범위
```

### 수동 설치

```bash
cd ~/tools/agent-skills && git pull
```
복사해서 쓰는 경우에는 다시 복사해야 합니다.

### 다른 도구

| 도구 | 업데이트 |
|---|---|
| Command Code | `cmd skills add addyosmani/agent-skills --force` |
| Antigravity | `agy update` |
| Cursor | 원본을 다시 동기화 (`docs/cursor-setup.md`의 "After upstream updates") |

---

## 12. 삭제하기

### 12.1 Claude Code 플러그인

#### 잠시 끄기 (삭제하지 않음, 실제 실행 확인)

```bash
claude plugin disable agent-skills@addy-agent-skills   # 끄기 → Status: × disabled
claude plugin enable  agent-skills@addy-agent-skills   # 다시 켜기
```
끄면 상시 토큰 비용(약 3.6k)도 사라집니다. 다시 쓸 가능성이 있으면 삭제보다 이 방법이 편합니다.

#### 완전히 삭제 (실제 실행 확인)

```bash
# 1) 플러그인 삭제
claude plugin uninstall agent-skills@addy-agent-skills
#    project 범위로 설치했다면:
claude plugin uninstall agent-skills@addy-agent-skills --scope project

# 2) 마켓플레이스 등록 해제 (다시 설치할 일이 없다면)
claude plugin marketplace remove addy-agent-skills

# 3) 확인
claude plugin list               # → No plugins installed (또는 목록에 없음)
claude plugin marketplace list   # → addy-agent-skills 없음
```
Claude Code 안에서는 `/plugin` 관리 화면에서 같은 작업을 할 수 있습니다.

#### 남는 것과 정리 방법

| 남는 것 | 위치 | 정리 |
|---|---|---|
| 플러그인 캐시 폴더 | `~/.claude/plugins/cache/addy-agent-skills/` | 실험에서 uninstall 후에도 **남아 있었음.** 신경 쓰이면 `rm -rf ~/.claude/plugins/cache/addy-agent-skills` |
| 플러그인 데이터 폴더 | `~/.claude/plugins/data/<id>/` | uninstall이 기본으로 삭제 (`--keep-data`를 주면 보존) |
| 내가 직접 연결한 훅 | `.claude/settings.json`의 `hooks` | 해당 항목 직접 삭제 |
| 작업 산출물 | `SPEC.md`, `tasks/`, `CONSTRAINTS.md`, `docs/ideas/` | 프로젝트 문서이므로 자동 삭제되지 않음. 필요 없으면 직접 삭제 |
| CLAUDE.md에 추가한 줄 | 예: "CONSTRAINTS.md를 읽을 것" | 직접 삭제 |
| 내가 만든 에이전트·스킬 사본 | `.claude/agents/`, `.claude/skills/` | 직접 삭제 |
| 훅 캐시 | `.claude/sdd-cache/`, `.claude/.simplify-ignore-cache/` | 직접 삭제 |
| `/constraints`가 설치한 도구 | package.json의 `check:*` 스크립트, devDependencies | 직접 삭제 |

> ⚠️ Windows에서는 `~/.claude`가 `%USERPROFILE%\.claude`입니다.

### 12.2 `npx skills` (실제 실행 확인)

```bash
# 설치된 목록 확인
npx skills list

# 하나만 삭제
npx skills remove -s idea-refine -y
```

agent-skills의 25개만 골라서 삭제 (bash):
```bash
npx skills remove -y -s \
  using-agent-skills interview-me idea-refine spec-driven-development \
  constraint-driven-development planning-and-task-breakdown \
  incremental-implementation test-driven-development context-engineering \
  source-driven-development doubt-driven-development frontend-ui-engineering \
  api-and-interface-design browser-testing-with-devtools \
  debugging-and-error-recovery code-review-and-quality code-simplification \
  security-and-hardening performance-optimization git-workflow-and-versioning \
  ci-cd-and-automation deprecation-and-migration documentation-and-adrs \
  observability-and-instrumentation shipping-and-launch
# 전역(-g)으로 설치했다면 -g 추가
```
(이름 여러 개를 한 번에 나열해서 지우는 형식을 직접 실행해서 확인했습니다. 지정한 이름만 삭제되고 나머지는 남았습니다.)

> 🚨 **`npx skills remove --all`은 agent-skills뿐 아니라 이 CLI로 설치한 모든 스킬을 지웁니다.** 다른 스킬도 설치했다면 쓰지 마세요.

실험 결과: 삭제 후 `.claude/skills/`가 비고 `skills-lock.json`이 `"skills": {}`로 갱신됐습니다. `skills-lock.json` 파일 자체는 남으므로 필요 없으면 직접 지우세요.

### 12.3 수동 설치

```bash
rm -rf .claude/skills/<복사한-스킬-이름>
rm -rf ~/tools/agent-skills     # 클론한 원본
```
`claude --plugin-dir ...`로 실행하던 방식은 그 옵션을 빼고 실행하면 끝입니다.

### 12.4 다른 도구

| 도구 | 삭제 |
|---|---|
| Command Code | `cmd skills remove <스킬>` (전역: `--global`) |
| Codex, Gemini, Antigravity 등 | 각 도구의 플러그인·스킬 삭제 명령. 저장소 문서에는 명시되지 않았으니 해당 도구의 공식 문서를 확인하세요 |
| Cursor, Copilot, Windsurf, OpenCode, Kiro | 복사한 폴더와 규칙 파일 내용을 직접 삭제 |

---

## 13. 문제 해결

| 증상 | 원인 | 해결 |
|---|---|---|
| `Permission denied (publickey)` | 마켓플레이스가 SSH로 클론 | HTTPS 주소로 추가하거나 `git config --global url."https://github.com/".insteadOf git@github.com:` |
| `Default commands/ folder is ignored because the manifest sets 'commands'` 경고 | 루트 `commands/`는 Antigravity용 | **무시해도 됨** (공식 문서: 표시만 되는 경고) |
| `/review` 등이 다른 명령으로 실행됨 | 이름 충돌 | `/agent-skills:review` 형태로 입력 |
| 설치했는데 명령어가 안 보임 | 재시작 안 함, 또는 `npx skills`로 설치 (명령어 미포함) | 재시작, 또는 플러그인 방식으로 설치 |
| 스킬이 자동으로 발동하지 않음 | description 매칭은 확률적 | 명령어나 스킬 이름을 직접 지정 |
| 단일 스킬 설치 후 체크리스트 경로 오류 | `references/` 미포함 (이슈 #361) | 체크리스트를 스킬 폴더 안 `references/`로 복사 |
| 훅 실행 시 `jq is required` | jq 미설치 | `brew install jq` / `apt-get install jq` |
| 업데이트 후 훅이 동작 안 함 | 플러그인 캐시의 버전 폴더명 변경 | 고정 위치에 클론한 경로 사용 (10.5절) |
| `/build auto`가 바로 멈춤 | 스펙 없음, 또는 관련 없는 미커밋 변경 | `/spec` 실행, 변경 커밋 또는 stash |
| `/build auto`가 계획 후 진행 안 함 | 승인이 애매함 | "approve", "go", "yes"로 분명하게 답하기 |
| 컨텍스트가 빨리 참 | 규칙 파일에 스킬 전문을 붙여넣음 | 스킬은 필요할 때만 로드. 규칙 파일에는 짧은 정책만 |

---

## 14. 토큰 비용

`claude plugin details`의 실측 추정치(v0.6.12)입니다. 실제 사용량과 다를 수 있습니다.

| 항목 | 비용 |
|---|---|
| **상시 비용** (켜져 있는 동안 모든 세션에 추가) | **약 3,622 토큰** |
| 스킬 1개 실행 시 (본문 로드) | 약 2.8k ~ 7.3k 토큰 |
| 가장 큰 스킬 | code-review-and-quality 약 7.3k, constraint-driven-development 약 7.1k |
| 명령어 1개 | 약 0.2k ~ 1.6k (스킬 비용은 별도) |
| 에이전트 1개 | 약 1.1k ~ 4.3k (서브에이전트 컨텍스트에서 사용) |

절약 팁:
1. 안 쓸 때는 `claude plugin disable`로 끄기
2. 몇 개만 쓴다면 `npx skills --skill`로 골라 설치
3. 규칙 파일에 스킬 전문을 붙여넣지 않기
4. `/ship`은 서브에이전트 3개를 병렬로 돌리므로 작은 변경에는 `/review`만 쓰기
5. 큰 작업은 단계마다 새 세션 쓰기 (시나리오 5)

---

## 15. 자주 묻는 질문

**Q. 무료인가요?**
A. 네. MIT 라이선스입니다. 다만 사용하는 AI 서비스의 토큰 비용은 별도입니다.

**Q. 코딩을 잘 몰라도 쓸 수 있나요?**
A. 네. 오히려 에이전트에게 스펙, 테스트, 리뷰를 강제해 주므로 초보자에게 더 유용합니다. `/spec`으로 시작해서 질문에 답하기만 해도 됩니다.

**Q. 기존 프로젝트를 바꿔야 하나요?**
A. 아니요. 공식 문서 기준으로 마이그레이션은 필요 없습니다. 프로젝트 루트에서 설치하고 그대로 쓰면 됩니다. 단계적 도입은 시나리오 2를 참고하세요.

**Q. SPEC.md와 tasks/ 파일은 계속 남겨야 하나요?**
A. 작업 중에는 git에 두고 계속 갱신하는 **살아 있는 문서**로 쓰세요. 장기 보관이 필요 없으면 머지 전에 지우거나 `.gitignore`에 추가해도 됩니다.

**Q. 다른 스킬 팩(Superpowers, ECC 등)과 같이 써도 되나요?**
A. 역할이 겹치는 팩을 함께 쓰면 같은 요청에 여러 스킬이 경쟁하고 토큰도 낭비됩니다. 하나를 고르는 것을 권합니다. 저장소의 `docs/comparison.md`에 Superpowers, Matt Pocock skills와의 비교가 있습니다.

**Q. 한국어로 요청해도 되나요?**
A. 됩니다. 다만 스킬 설명이 영어라서 자동 선택 정확도는 영어 요청보다 낮을 수 있습니다 (추론, 검증하지 않음). 확실히 하려면 명령어나 스킬 이름을 직접 쓰세요.

---

## 16. 출처

- 공식 GitHub (모든 내용의 1차 근거): <https://github.com/addyosmani/agent-skills>
  - `README.md`, `docs/getting-started.md`, `docs/adoption-guide.md`, `docs/agents.md`, `docs/skill-anatomy.md`, `docs/advanced-per-agent-configuration.md`, `docs/*-setup.md`, `docs/other-hosts.md`
  - `.claude/commands/*.md` (명령어 9개), `skills/*/SKILL.md` (25개), `agents/*.md` (4개)
  - `hooks/SIMPLIFY-IGNORE.md`, `hooks/SDD-CACHE.md`, `hooks/session-start.sh`, `evals/README.md`
- 공식 사이트 (검색 결과로 구조만 확인):
  - <https://skills.addy.ie/>
  - <https://skills.addy.ie/docs/getting-started/>
  - <https://skills.addy.ie/tutorials/>
  - <https://skills.addy.ie/tutorials/new-app/>
  - <https://skills.addy.ie/tutorials/existing-app/>
  - <https://skills.addy.ie/tutorials/loop-engineering/>
  - <https://skills.addy.ie/skills/using-agent-skills/>
- skills CLI: <https://github.com/vercel-labs/skills> (npm `skills` 1.7.0, `--help` 출력으로 확인)
- Claude Code CLI 2.1.289: `claude plugin --help`, `install|uninstall|update|details|marketplace --help` 출력과 실제 실행 결과
