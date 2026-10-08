# Claude Code 오케스트레이션 완전 가이드 (초보자용)

> **한 줄 요약**
> 오케스트레이션은 **여러 AI 작업자(에이전트)에게 일을 나눠 맡기고 결과를 모아 하나의 결과물로 만드는 방식**입니다.
> Claude Code에서는 별도 설치 없이 바로 쓸 수 있습니다. 메인 대화를 **Opus**로 두고, 일을 **Sonnet**·**Haiku** 서브에이전트에게 나눠 맡기면 됩니다.
>
> ```text
> 계획은 네가 세우고, 파일 탐색은 Haiku 서브에이전트에게,
> 구현은 Sonnet 서브에이전트에게 맡긴 뒤 결과를 종합해줘.
> ```

- 공식 문서: <https://code.claude.com/docs/en/agents> (병렬 에이전트 개요)
- 이 문서 기준 버전: **Claude Code 2.1.294** (실제 실행 검증에 쓴 버전)
- 작성일: 2026-10-08

---

## 목차

0. [이 문서를 어떻게 검증했나](#0-이-문서를-어떻게-검증했나)
1. [오케스트레이션이란 무엇인가](#1-오케스트레이션이란-무엇인가)
2. [Claude Code의 오케스트레이션 기능 5가지](#2-claude-code의-오케스트레이션-기능-5가지)
3. [설치 전 준비물](#3-설치-전-준비물)
4. [Claude Code 설치하기](#4-claude-code-설치하기)
5. [모델 이해하기: Opus, Sonnet, Haiku](#5-모델-이해하기-opus-sonnet-haiku)
6. [기본 사용법 ① 서브에이전트](#6-기본-사용법--서브에이전트)
7. [실습: Opus 지휘자 + Sonnet·Haiku 작업자 팀 만들기](#7-실습-opus-지휘자--sonnethaiku-작업자-팀-만들기)
8. [기본 사용법 ② 동적 워크플로](#8-기본-사용법--동적-워크플로)
9. [기본 사용법 ③ 에이전트 팀 (실험 기능)](#9-기본-사용법--에이전트-팀-실험-기능)
10. [데스크톱 앱 Code 탭에서 쓰기](#10-데스크톱-앱-code-탭에서-쓰기)
11. [응용: 상황별 실전 시나리오](#11-응용-상황별-실전-시나리오)
12. [응용: 고급 설정](#12-응용-고급-설정)
13. [비용 관리](#13-비용-관리)
14. [끄기와 삭제하기](#14-끄기와-삭제하기)
15. [문제 해결](#15-문제-해결)
16. [자주 묻는 질문](#16-자주-묻는-질문)
17. [출처](#17-출처)

---

## 0. 이 문서를 어떻게 검증했나

정확성을 위해 확인한 범위와 확인하지 못한 범위를 먼저 밝힙니다.

| 대상 | 확인 방법 | 결과 |
|---|---|---|
| 서브에이전트, 에이전트 팀, 동적 워크플로, 설치·삭제, 모델 설정, 비용, 데스크톱 앱 | 공식 문서(code.claude.com) 원문을 직접 읽음 | ✅ 확인 |
| 오케스트레이션 개념과 수치 | Anthropic 엔지니어링 블로그 2편을 직접 읽음 | ✅ 확인 |
| 서브에이전트 파일 만들기 → Haiku 모델로 호출 | Claude Code 2.1.294로 **실제 실행**. 결과 JSON에서 Haiku가 실제로 호출된 것을 확인 | ✅ 실행 검증 |
| `--agents` 플래그로 Opus 메인 + Sonnet 서브에이전트 | **실제 실행**. Opus와 Sonnet 사용 내역이 각각 기록됨 | ✅ 실행 검증 |
| 서브에이전트 파일 삭제 후 사라지는지 | **실제 실행**. 삭제 후 새 세션에서 해당 에이전트가 목록에 없음 | ✅ 실행 검증 |
| 에이전트 팀 | 대화형(interactive) 세션이 필요해서 이 작업 환경에서는 실행하지 못함 | ⚠️ 문서 기준 |
| 동적 워크플로 실행, 데스크톱 앱 화면 | 직접 실행하거나 화면을 보지 못함 | ⚠️ 문서 기준 |

> ⚠️ **버전 주의:** Claude Code는 업데이트가 매우 잦습니다. 메뉴 이름, 단축키, 기본값은 버전에 따라 바뀔 수 있습니다. 이 문서와 화면이 다르면 공식 문서를 기준으로 하세요.

---

## 1. 오케스트레이션이란 무엇인가

### 1.1 쉬운 비유: 오케스트라 지휘자

| 오케스트라 | Claude Code |
|---|---|
| 지휘자 | **메인 대화** (예: Opus). 계획하고, 나눠주고, 결과를 합침 |
| 연주자들 | **서브에이전트** (예: Sonnet, Haiku). 맡은 부분만 처리 |
| 악보 | 내 요청, 에이전트 정의 파일, 워크플로 스크립트 |
| 완성된 연주 | 최종 결과물 (코드, 보고서, 리뷰) |

지휘자는 직접 연주하지 않습니다. 오케스트레이터도 직접 모든 파일을 읽는 대신 **일을 쪼개서 맡기고, 돌아온 결과를 판단하고 합칩니다.**

### 1.2 실제 구조

```
            ┌───────────────────────────┐
  나 ──요청──▶│  메인 대화 (오케스트레이터)   │
            │  모델: Opus                 │
            └──────┬──────────┬─────────┘
          작업 위임 │          │ 작업 위임
     ┌─────────────▼──┐   ┌───▼──────────────┐
     │ 서브에이전트 A    │   │ 서브에이전트 B      │
     │ 모델: Haiku      │   │ 모델: Sonnet       │
     │ 자기만의 컨텍스트  │   │ 자기만의 컨텍스트    │
     └───────┬────────┘   └────────┬─────────┘
             │ 요약만 반환            │ 요약만 반환
             └──────────┬──────────┘
                        ▼
              메인 대화가 종합 → 나에게 답변
```

핵심은 세 가지입니다 (공식 문서 기준).

1. **각 서브에이전트는 자기만의 컨텍스트 창에서 일합니다.** 파일 수백 개를 뒤져도 메인 대화에는 **요약만** 돌아옵니다. 그래서 메인 대화가 덜 어수선해집니다.
2. **서브에이전트마다 모델, 사용할 도구, 권한을 따로 정할 수 있습니다.** 탐색은 저렴한 Haiku, 판단은 Opus처럼 나눌 수 있습니다.
3. **서브에이전트는 메인 대화의 이전 내용을 모릅니다.** 지시를 구체적으로 줘야 결과가 좋습니다.

### 1.3 업계에서 쓰는 오케스트레이션 패턴

Anthropic 엔지니어링 블로그 *"Building Effective Agents"*(2024-12-19)의 분류입니다.

| 패턴 | 설명 | 언제 쓰나 |
|---|---|---|
| 프롬프트 체이닝 | A → B → C 순서로 앞 단계 결과를 다음 단계에 넘김 | 단계가 고정된 작업 |
| 라우팅 | 입력을 분류해서 전문 처리기로 보냄 | 유형별로 처리가 달라야 할 때 |
| 병렬화 | 여러 작업을 동시에 돌리거나, 같은 작업을 여러 번 돌려 투표 | 속도, 신뢰도 향상 |
| **오케스트레이터-워커** | 중앙 LLM이 작업을 나눠 워커에게 맡기고 결과를 종합 | 하위 작업을 미리 알 수 없는 복잡한 작업 |
| 평가자-최적화기 | 하나가 만들고 다른 하나가 평가하며 반복 | 평가 기준이 명확할 때 |
| 에이전트 | LLM이 도구를 쓰며 스스로 루프를 돔 | 단계 수를 예측할 수 없는 열린 문제 |

같은 글의 핵심 조언: **"가장 단순한 해법부터 찾고, 필요할 때만 복잡도를 높여라."**

### 1.4 실제로 얼마나 효과가 있나

Anthropic이 자사 Research 기능의 구조를 공개한 글 *"How we built our multi-agent research system"*(2025-06-13)에 나온 수치입니다.

| 항목 | 수치 |
|---|---|
| 성능 | Opus 4 리더 + Sonnet 4 서브에이전트 구성이 Opus 4 단독보다 내부 리서치 평가에서 **90.2% 높은 성능** |
| 토큰 사용량 | 에이전트는 일반 채팅보다 **약 4배**, 멀티에이전트는 **약 15배** 많은 토큰 사용 |
| 성능 요인 | BrowseComp 평가에서 **토큰 사용량 하나가 성능 차이의 80%를 설명** |

> 💡 **해석:** 멀티에이전트는 "토큰을 더 써서 더 넓게 탐색하는" 방식입니다. 리서치처럼 **병렬로 나눌 수 있는 일**에서 효과가 크고, 같은 글에 따르면 **대부분의 코딩 작업은 리서치보다 병렬로 나눌 부분이 적어서** 효과가 작습니다.

---

## 2. Claude Code의 오케스트레이션 기능 5가지

공식 문서는 여러 작업을 동시에 하는 방법을 다섯 가지로 정리합니다.

| 기능 | 한 줄 설명 | 상태 | 이럴 때 쓰세요 |
|---|---|---|---|
| **서브에이전트** | 한 세션 안에서 Claude가 작업자를 띄워 일을 맡기고 요약을 받음 | 정식 기능 | 검색 결과나 로그가 메인 대화를 어지럽힐 때. **가장 기본** |
| **동적 워크플로** | Claude가 짠 스크립트가 수십~수백 개 서브에이전트를 돌리고 결과를 교차 검증 | 유료 요금제 전부 (Pro는 `/config`에서 켜야 함) | 저장소 전체 감사, 대량 마이그레이션, 교차 검증 리서치 |
| **에이전트 팀** | 리더 세션이 팀원 세션들을 관리. 팀원끼리 직접 메시지를 주고받음 | **실험 기능, 기본 꺼짐**, CLI 전용 | 팀원끼리 토론하거나 서로 검증해야 할 때 |
| **에이전트 뷰** | 백그라운드 세션 여러 개를 한 화면에서 관리 (`claude agents`) | 연구 미리보기 | 독립적인 일 여러 개를 맡기고 나중에 확인할 때 |
| **프로젝트** | claude.ai/code나 데스크톱 앱에서 하나의 대화가 여러 병렬 스레드를 관리 | 공개 베타 (Pro, Max) | 며칠~몇 주에 걸친 큰 작업 |

### 2.1 어떤 걸 골라야 하나?

```
작업을 나눠야 하나요?
 │
 ├─ 아니오 → 메인 대화 하나로 충분 (가장 저렴)
 │
 └─ 예 → 하위 작업이 몇 개인가요?
          │
          ├─ 몇 개 (1~10) ──────────────▶ ① 서브에이전트  ← 이 문서의 중심
          │
          ├─ 수십~수백 개, 또는 교차 검증 필요 ─▶ ② 동적 워크플로
          │
          └─ 작업자끼리 토론·협업이 필요 ────▶ ③ 에이전트 팀 (CLI, 실험)
```

> 이 문서는 **"Opus, Sonnet, Haiku로 구성해서 일을 나눠 맡기기"**가 목표이므로 **서브에이전트**를 가장 자세히 다루고, 워크플로와 에이전트 팀은 핵심만 다룹니다.

---

## 3. 설치 전 준비물

| 항목 | 요구 사항 (공식 문서 기준) |
|---|---|
| 계정 | **Pro, Max, Team, Enterprise, Console(API)** 중 하나. **무료 플랜은 Claude Code를 쓸 수 없습니다.** Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry도 가능 |
| 운영체제 | macOS 13.0+, Windows 10 1809+ / Windows Server 2019+, Ubuntu 20.04+, Debian 10+, Alpine Linux 3.19+ |
| 하드웨어 | RAM 4GB 이상, x64 또는 ARM64 |
| 네트워크 | 인터넷 연결 필수 |
| 지역 | Anthropic 지원 국가 (한국 포함) |
| Windows 권장 | [Git for Windows](https://git-scm.com/downloads/win) (없으면 PowerShell로 명령 실행) |

> ✅ **오케스트레이션 기능은 별도 설치가 없습니다.** 서브에이전트와 워크플로는 Claude Code에 내장되어 있습니다. Claude Code만 설치하면 됩니다.

---

## 4. Claude Code 설치하기

### 4.1 방법 고르기

| 방법 | 추천 대상 | 자동 업데이트 |
|---|---|---|
| **네이티브 설치 (추천)** | 대부분의 사용자 | ✅ 자동 |
| Homebrew | macOS에서 brew를 쓰는 분 | ❌ 직접 (`brew upgrade`) |
| WinGet | Windows에서 winget을 쓰는 분 | ❌ 직접 (`winget upgrade`) |
| npm | Node.js 22+ 환경 | ❌ 직접 |
| **데스크톱 앱** | 터미널이 낯선 분 | 앱이 관리 |

### 4.2 네이티브 설치 (추천)

**macOS, Linux, WSL** (터미널에 붙여넣기):

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell** (프롬프트가 `PS C:\`로 시작):

```powershell
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD** (프롬프트가 `C:\`로 시작, `PS` 없음):

```batch
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

> 💡 설치 중에는 진행 표시가 없습니다. 멈춘 것처럼 보여도 기다리세요.
> 💡 `'irm' is not recognized` 오류 → CMD에서 PowerShell 명령을 쳤습니다. CMD 명령을 쓰세요.
> 💡 `The token '&&' is not a valid statement separator` 오류 → PowerShell에서 CMD 명령을 쳤습니다. PowerShell 명령을 쓰세요.

### 4.3 다른 설치 방법

```bash
# macOS Homebrew (안정 채널. 최신 채널은 claude-code@latest)
brew install --cask claude-code

# Windows WinGet
winget install Anthropic.ClaudeCode

# npm (Node.js 22 이상 필요, sudo 쓰지 마세요)
npm install -g @anthropic-ai/claude-code
```

### 4.4 데스크톱 앱 (터미널 없이)

1. [macOS](https://claude.ai/api/desktop/darwin/universal/dmg/latest/redirect) 또는 [Windows](https://claude.com/download)용 앱을 내려받아 설치합니다. (Linux는 베타)
2. 앱을 열고 로그인합니다.
3. 상단의 **Code** 탭을 누릅니다.

데스크톱 앱 Code 탭은 CLI와 **같은 엔진**을 씁니다. 자세한 차이는 [10장](#10-데스크톱-앱-code-탭에서-쓰기)을 보세요.

### 4.5 설치 확인

**새 터미널 창**을 열고 실행합니다.

```bash
claude --version
```

`2.1.294 (Claude Code)`처럼 버전이 나오면 성공입니다. 더 자세히 점검하려면:

```bash
claude doctor
```

설치 상태, 설정 파일 오류, 자동 업데이트 상태를 보여줍니다.

> ❗ `command not found: claude`가 나오면 설치 폴더가 PATH에 없는 것입니다. 공식 문서의 [Fix your PATH](https://code.claude.com/docs/en/troubleshoot-install)를 참고하세요.

### 4.6 로그인

작업할 프로젝트 폴더로 이동해서 실행합니다.

```bash
cd 내-프로젝트-폴더
claude
```

처음 실행하면 브라우저가 열리고 로그인 안내가 나옵니다. 따라서 로그인하면 끝입니다.

---

## 5. 모델 이해하기: Opus, Sonnet, Haiku

### 5.1 모델 별칭(alias)

Claude Code에서는 모델을 짧은 **별칭**으로 부릅니다. 별칭은 항상 그 계열의 **최신 버전**을 가리킵니다.

| 별칭 | 의미 (공식 문서) | 이 문서 작성 시점의 실제 모델 (Anthropic API 기준) |
|---|---|---|
| `opus` | 복잡한 추론 작업용 최신 Opus | Opus 5.5 |
| `sonnet` | 일상적인 코딩 작업용 최신 Sonnet | Sonnet 5.5 |
| `haiku` | 단순 작업용 빠르고 효율적인 Haiku | Haiku 5.5 |
| `fable` | 가장 어렵고 오래 걸리는 작업용 Fable | Fable 5.1 |
| `opusplan` | **계획 모드에서는 Opus, 실행할 때는 Sonnet**으로 자동 전환 | Opus + Sonnet |
| `inherit` | (서브에이전트 전용) 메인 대화의 모델을 그대로 사용 | 메인 모델 |

> 다른 클라우드 제공자(Bedrock 등)에서는 별칭이 이전 버전을 가리킬 수 있습니다.

### 5.2 역할별 권장 배치

| 역할 | 권장 모델 | 이유 |
|---|---|---|
| 지휘자 (계획, 판단, 종합) | **Opus** | 판단 품질이 전체 결과를 좌우함 |
| 구현, 리팩터링, 테스트 작성 | **Sonnet** | 품질, 속도, 비용의 균형. 공식 문서도 "대부분의 코딩 작업은 Sonnet으로 충분"하다고 안내 |
| 파일 찾기, 검색, 로그 요약 | **Haiku** | 빠르고 저렴. 공식 문서도 단순 서브에이전트 작업에 `model: haiku`를 권장 |

### 5.3 메인 대화(지휘자) 모델 바꾸는 법

우선순위가 높은 순서입니다.

| 방법 | 예시 | 적용 범위 |
|---|---|---|
| 대화 중 명령 | `/model opus` (인자 없이 `/model`만 치면 선택 화면) | 즉시 전환. 기본값으로도 저장됨 |
| 시작할 때 | `claude --model opus` | 이번 세션 |
| 환경 변수 | `ANTHROPIC_MODEL=opus` | 해당 환경 |
| 설정 파일 | `~/.claude/settings.json`에 `"model": "opus"` | 항상 |
| 데스크톱 앱 | 보내기 버튼 옆 **모델 드롭다운** | 세션 중에도 변경 가능 |

> 💡 `/model` 선택 화면에서 `Enter`는 "전환 + 기본값으로 저장", `s`는 "이번 세션만 전환"입니다.

### 5.4 서브에이전트의 모델은 어떻게 정해지나 (중요)

공식 문서의 결정 순서입니다. **위에서부터 먼저 해당하는 것이 적용됩니다.**

1. Claude가 서브에이전트를 부를 때 **그 자리에서 지정한 모델** (예: "Haiku로 돌려줘"라고 요청한 경우)
2. 서브에이전트 정의 파일의 `model:` 항목
3. 환경 변수 `CLAUDE_CODE_SUBAGENT_MODEL`
4. 메인 대화의 모델

> ⚠️ 정의 파일에 `model`을 적지 않으면 **메인 모델(예: Opus)을 그대로 따라갑니다.** Opus로 바꾸면 서브에이전트도 전부 Opus가 되어 비용이 커질 수 있습니다. 저렴한 모델로 고정하려면 `model:`을 꼭 적으세요.

---

## 6. 기본 사용법 ① 서브에이전트

### 6.1 이미 들어 있는 기본 서브에이전트

아무 설정 없이도 Claude Code에는 다음 서브에이전트가 들어 있습니다.

| 이름 | 하는 일 |
|---|---|
| **Explore** | 빠른 읽기 전용 코드 탐색. 파일 수정(Write, Edit) 불가 |
| **Plan** | 계획 모드에서 읽기 전용 조사 |
| **general-purpose** | 탐색과 수정이 모두 필요한 다단계 작업 |
| **claude** | 어디에도 맞지 않는 작업을 맡는 범용 에이전트 |
| **statusline-setup** | `/statusline` 설정 (Sonnet) |
| **claude-code-guide** | Claude Code 사용법 질문에 답함 (Haiku) |

### 6.2 가장 쉬운 방법: 말로 요청하기

메인 모델을 Opus로 두고 그냥 이렇게 요청하면 됩니다.

```text
이 저장소에서 인증 관련 코드를 찾아서 정리해줘.
탐색은 Haiku 서브에이전트에게 맡기고, 결과는 네가 종합해줘.
```

```text
서브에이전트 3개를 Sonnet으로 띄워서
src/api, src/db, src/ui 폴더를 각각 리뷰하게 하고, 결과를 하나로 정리해줘.
```

> 💡 **"서브에이전트"라는 단어를 넣는 게 확실합니다.** 그냥 "찾아줘"라고만 하면 Claude가 직접 처리할 수도 있습니다.

### 6.3 서브에이전트를 파일로 정의하기 (반복해서 쓸 때 추천)

매번 말로 설명하기 번거로우면 **역할을 파일로 저장**합니다. 마크다운 파일 하나가 서브에이전트 하나입니다.

#### 저장 위치

| 위치 | 적용 범위 | 우선순위 |
|---|---|---|
| 조직 관리 설정 (managed settings) | 조직 전체 | 1 (가장 높음) |
| `claude --agents '...'` 플래그 | 이번 세션만 (디스크에 저장 안 됨) | 2 |
| **`.claude/agents/`** (프로젝트 폴더 안) | 이 프로젝트. **git에 커밋하면 팀원과 공유** | 3 |
| **`~/.claude/agents/`** (홈 폴더) | 내 모든 프로젝트 | 4 |
| 플러그인의 `agents/` 폴더 | 플러그인이 켜진 곳 | 5 (가장 낮음) |

같은 이름이 여러 곳에 있으면 **우선순위가 높은 쪽**이 이깁니다.

#### 파일 형식

```markdown
---
name: code-reviewer
description: 코드 품질과 보안을 리뷰한다. 코드를 작성하거나 수정한 직후에 사용.
tools: Read, Glob, Grep
model: sonnet
---

너는 시니어 코드 리뷰어다. 호출되면 코드를 분석해서
품질, 보안, 모범 사례에 대해 구체적이고 실행 가능한 피드백을 줘라.
```

- `---` 사이(frontmatter)는 **설정**입니다.
- 그 아래 본문은 서브에이전트의 **시스템 프롬프트(업무 지침)**입니다.

#### 자주 쓰는 설정 항목

필수는 `name`, `description` 두 개뿐입니다. 항목 이름은 **대소문자를 구분**하고, 모르는 항목은 **조용히 무시**됩니다(오타 주의).

| 항목 | 필수 | 설명 | 예시 |
|---|---|---|---|
| `name` | ✅ | 고유 이름. `:` 사용 불가, `-`로 시작 불가 | `code-reviewer` |
| `description` | ✅ | **언제 이 에이전트에게 맡길지** Claude가 판단하는 기준. 짧고 구체적으로 | `코드 수정 직후 리뷰` |
| `tools` | | 허용 도구 목록. 생략하면 쓸 수 있는 도구 전부 | `Read, Grep, Glob` |
| `disallowedTools` | | 금지 도구 목록 | `Write, Edit` |
| `model` | | `opus`, `sonnet`, `haiku`, `fable`, 전체 모델 ID, `inherit` | `haiku` |
| `effort` | | 추론 노력 수준 `low`~`max` | `low` |
| `permissionMode` | | 권한 모드 (`default`, `acceptEdits`, `plan` 등) | `plan` |
| `maxTurns` | | 최대 작업 턴 수. 넘으면 멈춤 | `20` |
| `isolation` | | `worktree`면 별도 git 작업 복사본에서 일함 (파일 충돌 방지) | `worktree` |
| `memory` | | `user`, `project`, `local`. 영구 메모리 사용 | `project` |
| `background` | | `true`면 항상 백그라운드에서 실행 | `true` |
| `skills` | | 시작할 때 미리 불러올 스킬 | 목록 |
| `color` | | 화면 표시 색 | `blue` |

> 전체 항목은 [공식 문서의 Frontmatter reference](https://code.claude.com/docs/en/sub-agents)를 보세요.

#### 파일이 반영되는 시점

Claude Code는 이 폴더들을 감시하므로 **파일을 수정하면 몇 초 안에 반영**됩니다. 단, **새 `agents` 폴더에 첫 파일을 만든 경우**에는 Claude Code를 다시 시작해야 합니다.

#### Claude에게 파일을 만들어 달라고 하기

파일을 직접 쓰기 어렵다면 Claude에게 시키면 됩니다 (공식 문서가 안내하는 방법).

```text
.claude/agents/ 폴더에 test-runner라는 서브에이전트를 만들어줘.
테스트를 실행하고 실패한 것만 요약하는 역할이고, Haiku를 쓰고, 파일 수정은 못 하게 해줘.
```

> ℹ️ **`/agents` 명령 변경 안내:** 예전 버전(v2.1.197 이하)에서는 `/agents`가 생성·편집·삭제 마법사를 열었습니다. 현재 버전에서는 **파일 위치를 알려주는 안내만** 출력합니다. 인터넷의 옛 글과 다를 수 있습니다.

### 6.4 서브에이전트 부르는 방법 4가지

| 방법 | 예시 | 특징 |
|---|---|---|
| **자동** | 그냥 일을 시킴 | Claude가 `description`을 보고 알아서 맡김. description에 "use proactively"(적극 사용)를 넣으면 더 자주 맡김 |
| **자연어로 지목** | `test-runner 서브에이전트로 실패한 테스트를 고쳐줘` | 가장 쉬움 |
| **@멘션** | `@`를 치고 목록에서 선택, 또는 `@agent-code-reviewer 인증 변경사항 봐줘` | **그 에이전트가 반드시 실행됨** |
| **세션 전체를 그 에이전트로** | `claude --agent code-reviewer` | 메인 대화 자체가 그 역할로 동작 |

### 6.5 파일 없이 한 번만 쓰기: `--agents` 플래그

이번 세션에서만 쓸 에이전트를 JSON으로 즉석 정의합니다. **디스크에 저장되지 않아서 끝나면 사라집니다.**

```bash
claude --model opus --agents '{
  "code-reviewer": {
    "description": "코드 리뷰 전문가. 코드 변경 후 적극적으로 사용.",
    "prompt": "너는 시니어 코드 리뷰어다. 품질, 보안, 모범 사례에 집중하라.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

`prompt`가 본문(시스템 프롬프트) 역할을 하고, 나머지는 frontmatter 항목과 같습니다.

### 6.6 포그라운드와 백그라운드

| 구분 | 동작 | 권한 요청 |
|---|---|---|
| **포그라운드** | 서브에이전트가 끝날 때까지 메인 대화가 기다림 | 나에게 바로 표시 |
| **백그라운드** | 메인 대화와 동시에 실행 | 메인 세션에 "어느 서브에이전트가 요청했는지"와 함께 표시 |

- 실행 중인 작업을 백그라운드로 보내기: **Ctrl+B**
- 백그라운드 작업 목록 보기, 확인, 중지: **`/tasks`**
- 대화형 세션에서는 기본적으로 **포크 모드**가 켜져 있어서, Claude가 띄우는 서브에이전트가 백그라운드에서 실행됩니다.
- 항상 포그라운드로 실행하려면 `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1`

### 6.7 이전 서브에이전트 이어서 쓰기

서브에이전트는 부를 때마다 새로 시작합니다. 이전 작업을 이어가고 싶으면:

```text
아까 그 code-reviewer 서브에이전트한테 이어서 테스트 파일도 봐달라고 해줘.
```

Claude가 `SendMessage`로 그 에이전트를 다시 깨우고, 에이전트는 **이전 대화 기록을 그대로 가진 채** 이어서 일합니다.

- 기본 제공 Explore, Plan은 일회용이라 이어갈 수 없습니다.
- 서브에이전트 기록은 `~/.claude/projects/{프로젝트}/{세션ID}/subagents/`에 저장되고, 기본 **30일** 뒤 삭제됩니다.

---

## 7. 실습: Opus 지휘자 + Sonnet·Haiku 작업자 팀 만들기

이 장을 그대로 따라 하면 **Opus가 계획하고 → Haiku가 찾고 → Sonnet이 구현하고 → Opus가 검토하는** 구조가 완성됩니다.

### 7.1 완성 구조

```
내 프로젝트/
└── .claude/
    └── agents/
        ├── scout.md        ← Haiku: 파일 찾기 (읽기 전용)
        ├── implementer.md  ← Sonnet: 코드 구현
        └── reviewer.md     ← Opus: 최종 검토 (읽기 전용)
```

### 7.2 1단계: 폴더 만들기

프로젝트 폴더에서 실행합니다.

```bash
# macOS, Linux, WSL
mkdir -p .claude/agents
```

```powershell
# Windows PowerShell
New-Item -ItemType Directory -Force .claude\agents
```

### 7.3 2단계: 탐색 담당 (Haiku) 만들기

`.claude/agents/scout.md` 파일을 만들고 아래 내용을 붙여넣습니다.

```markdown
---
name: scout
description: 코드베이스에서 관련 파일, 함수, 사용처를 빠르게 찾아 위치 목록으로 보고하는 읽기 전용 탐색 담당. 넓은 범위의 파일 탐색이 필요할 때 사용.
tools: Read, Grep, Glob
model: haiku
---

너는 읽기 전용 탐색 담당이다.

규칙:
- 파일을 절대 수정하지 마라.
- 찾은 결과는 `파일경로:줄번호 - 한 줄 설명` 형식으로 보고하라.
- 추측하지 말고, 실제로 열어서 확인한 것만 보고하라.
- 찾지 못한 것은 "찾지 못함"이라고 분명히 적어라.
- 보고는 최대 30줄로 요약하라.
```

**왜 이렇게 설정했나:**
- `tools: Read, Grep, Glob` → 읽기만 가능. 실수로 파일을 고칠 위험이 없습니다.
- `model: haiku` → 단순 검색에 비싼 모델을 쓰지 않습니다.
- "최대 30줄" → 메인 대화로 돌아오는 내용을 줄여 Opus의 토큰을 아낍니다.

### 7.4 3단계: 구현 담당 (Sonnet) 만들기

`.claude/agents/implementer.md`:

```markdown
---
name: implementer
description: 확정된 계획에 따라 코드를 수정하고 관련 테스트를 실행하는 구현 담당. 계획이 정해진 뒤 실제 코드 변경이 필요할 때 사용.
tools: Read, Edit, Write, Grep, Glob, Bash
model: sonnet
---

너는 구현 담당이다.

규칙:
- 전달받은 계획의 범위 안에서만 수정하라. 범위 밖 수정이 필요하면 하지 말고 보고하라.
- 기존 코드의 스타일과 패턴을 따르라.
- 수정 후 관련 테스트나 린트를 실행하고, 실행한 명령과 결과를 그대로 보고하라.
- 테스트가 실패하면 "성공"이라고 보고하지 마라.

보고 형식:
1. 수정한 파일 목록
2. 실행한 검증 명령과 결과
3. 남은 문제
```

### 7.5 4단계: 검토 담당 (Opus) 만들기

`.claude/agents/reviewer.md`:

```markdown
---
name: reviewer
description: 변경된 코드의 정확성, 보안, 경계 조건을 검토하는 읽기 전용 시니어 리뷰어. 구현이 끝난 직후 사용.
tools: Read, Grep, Glob, Bash
model: opus
---

너는 시니어 코드 리뷰어다. 파일을 수정하지 마라.

호출되면:
1. `git diff`로 변경 사항을 확인하라.
2. 변경된 파일에 집중하라.

점검 항목:
- 로직 오류, 경계 조건, null 처리
- 보안 문제 (입력 검증, 비밀 정보 노출)
- 오류 처리 누락
- 테스트 누락

우선순위별로 보고하라:
- 반드시 고쳐야 함 (근거와 재현 경로 포함)
- 고치는 게 좋음
- 참고
```

> ⚠️ `Bash`를 허용했기 때문에 리뷰어가 명령을 실행할 수는 있습니다. 완전히 읽기 전용으로 만들려면 `Bash`를 빼세요. 대신 `git diff`는 메인 대화가 대신 실행해서 넘겨줘야 합니다.

### 7.6 5단계: 메인을 Opus로 시작하기

```bash
claude --model opus
```

이미 실행 중이면 `/model opus`. 데스크톱 앱은 모델 드롭다운에서 Opus를 고릅니다.

> 💡 새 `agents` 폴더를 방금 만들었다면 **Claude Code를 다시 시작**해야 에이전트가 인식됩니다.

### 7.7 6단계: 지휘하기

이렇게 요청합니다.

```text
로그인 실패 시 5회 이상이면 계정을 잠그는 기능을 추가하고 싶어.

진행 방식:
1. scout 서브에이전트로 로그인, 인증, 사용자 모델 관련 코드를 찾아줘.
2. 그 결과를 바탕으로 네가 구현 계획을 세워서 나에게 먼저 보여줘.
3. 내가 승인하면 implementer 서브에이전트에게 계획을 넘겨 구현시켜줘.
4. 구현이 끝나면 reviewer 서브에이전트로 검토하고,
   "반드시 고쳐야 함"이 있으면 implementer에게 다시 고치게 해줘.
5. 마지막에 전체 결과를 요약해줘.
```

**이 요청이 잘 동작하는 이유:**
- 단계마다 **누가 무엇을 할지** 분명합니다.
- 2단계에서 **사람이 확인하는 지점**이 있어서 엉뚱한 방향으로 구현할 위험이 줄어듭니다.
- 4단계에 **검증 → 수정 반복**(평가자-최적화기 패턴)이 들어 있습니다.

### 7.8 실제로 동작하는지 확인하기 (실행 검증 결과)

이 문서를 쓰면서 Claude Code 2.1.294로 직접 실행해 확인했습니다.

**시험 1. 파일로 정의한 Haiku 서브에이전트**

```bash
# .claude/agents/file-scout.md 에 model: haiku 로 정의한 뒤
claude -p "file-scout 서브에이전트를 사용해서 이 폴더의 파일 목록을 받아와 그대로 알려줘." \
  --model sonnet --output-format json
```

결과 JSON의 모델별 사용 내역:

| 모델 | 역할 | 비용 (정가 기준 추정치) |
|---|---|---|
| claude-sonnet-5-5 | 메인 대화 | 약 $0.089 |
| claude-haiku-5-5 | file-scout 서브에이전트 | 약 $0.0013 |

`subagent_stats.by_type`에 `{"file-scout": 1}`이 기록되어, **정의한 서브에이전트가 Haiku로 실제 실행된 것**을 확인했습니다.

**시험 2. `--agents` 플래그로 Opus 메인 + Sonnet 서브에이전트**

| 모델 | 비용 (정가 기준 추정치) |
|---|---|
| claude-opus-5-5 (메인) | 약 $0.155 |
| claude-sonnet-5-5 (word-counter 서브에이전트) | 약 $0.011 |
| claude-haiku-5-5 | 약 $0.0001 |

> ℹ️ 시험 2에서 Haiku 사용 내역이 아주 조금 나왔는데, 정의한 서브에이전트는 Sonnet이었습니다. Claude Code의 **내부 보조 요청으로 추정**되며, 정확한 용도는 확인하지 못했습니다.

**확인 방법 (직접 해보기):**
- 대화형 세션: 서브에이전트가 실행되면 화면에 에이전트 이름이 표시됩니다. `/usage`의 사용량 분석에서 서브에이전트 비중도 볼 수 있습니다.
- 비대화형: 위처럼 `--output-format json`을 붙이면 `modelUsage`에 **모델별 사용량**이 나옵니다.

---

## 8. 기본 사용법 ② 동적 워크플로

서브에이전트가 몇 개를 넘어 **수십~수백 개**가 필요하거나, 결과를 **서로 교차 검증**해야 할 때 씁니다.

### 8.1 서브에이전트와 무엇이 다른가

| | 서브에이전트 | 동적 워크플로 |
|---|---|---|
| 다음에 뭘 할지 누가 정하나 | Claude가 턴마다 판단 | **스크립트**가 정함 |
| 중간 결과는 어디에 | Claude의 컨텍스트 | 스크립트 변수 (Claude 컨텍스트에는 최종 결과만) |
| 규모 | 한 번에 몇 개 | 실행당 수십~수백 개 |
| 다시 실행 | 에이전트 정의를 재사용 | **오케스트레이션 자체를 저장해서 재실행** |
| 중단되면 | 턴을 다시 시작 | 같은 세션 안에서 이어서 실행 가능 |

### 8.2 사용 조건

- **모든 유료 요금제**, Anthropic API, Bedrock, Agent Platform, Foundry에서 사용 가능
- **Pro 요금제**는 `/config`의 **Dynamic workflows** 항목을 켜야 함
- CLI, 데스크톱 앱, IDE 확장, `claude -p`, Agent SDK 모두 지원

### 8.3 가장 쉬운 시작: 내장 워크플로 `/deep-research`

```text
/deep-research Node.js 권한 모델이 v20과 v22 사이에 어떻게 바뀌었나?
```

1. 실행 허용 여부를 묻습니다 → **Yes** 선택
2. 백그라운드에서 여러 에이전트가 웹 검색, 출처 교차 확인, 종합을 진행합니다.
3. 끝나면 **출처가 달린 보고서**가 나옵니다. 교차 검증을 통과하지 못한 주장은 빠집니다.

### 8.4 내 작업을 워크플로로 실행하기

프롬프트에 **`ultracode`** 키워드를 넣거나, "워크플로로 해줘"라고 직접 요청합니다.

```text
ultracode: src/routes/ 아래 모든 API 엔드포인트에서 인증 검사가 빠진 곳을 감사해줘
```

```text
워크플로를 써서 src/components/ 아래 모든 컴포넌트를 JavaScript에서 TypeScript로 옮겨줘.
각 파일은 격리된 복사본에서 작업하고, 단순 변환 단계는 Haiku를, 검증은 Sonnet을 써줘.
```

> 💡 **모델 지정:** 워크플로 에이전트도 서브에이전트와 같은 순서로 모델을 정합니다. 요청할 때 "이 단계는 Haiku로"처럼 말하면 비용을 줄일 수 있습니다. 아무것도 지정하지 않으면 **메인 세션 모델**로 실행됩니다.

### 8.5 진행 상황 보기: `/workflows`

```text
/workflows
```

| 키 | 동작 |
|---|---|
| `↑` / `↓` | 단계나 에이전트 선택 |
| `Enter` 또는 `→` | 선택한 단계 → 에이전트 상세로 들어가기 |
| `Esc` 또는 `←` | 한 단계 나오기 |
| `f` | 상태별 필터 |
| `p` | 일시정지 / 재개 |
| `x` | 선택한 에이전트 중지 (실행 전체에 초점이 있으면 워크플로 전체 중지) |
| `r` | 선택한 실행 중 에이전트 재시작 |
| `s` | 이 실행의 스크립트를 **명령으로 저장** |

### 8.6 저장해서 재사용하기

`/workflows`에서 실행을 고르고 `s`를 누르면 저장 위치를 고를 수 있습니다 (Tab으로 전환).

| 저장 위치 | 범위 |
|---|---|
| `.claude/workflows/` | 이 프로젝트. git으로 팀과 공유 |
| `~/.claude/workflows/` | 내 모든 프로젝트, 나만 사용 |

저장한 워크플로는 이후 **`/이름`** 명령으로 실행됩니다.

### 8.7 규모 조절: 크기 가이드라인

Claude가 워크플로를 짤 때 목표로 삼는 에이전트 수입니다. **강제 상한이 아니라 권고**입니다.

| 값 | 목표 에이전트 수 |
|---|---|
| `small` | 5개 미만 |
| `medium` | 10개 미만 (기본값. Pro 요금제는 `small`이 기본) |
| `large` | 50개 미만 |
| `unrestricted` | 작업에 맞게 |

```text
/config workflowSizeGuideline=small
```

**런타임 고정 한도:** 동시 실행 에이전트 기본 16개, 실행당 에이전트 총 1,000개.

### 8.8 ultracode 모드 (주의)

```text
/effort ultracode
```

켜면 Claude가 **모든 큰 작업을 워크플로로 처리**합니다. 요청 하나가 여러 워크플로로 이어질 수 있어 **토큰을 훨씬 많이 씁니다.** 끄려면 `/effort ultracode off`.

---

## 9. 기본 사용법 ③ 에이전트 팀 (실험 기능)

> ⚠️ **실험 기능이고 기본으로 꺼져 있습니다.** 세션 재개, 작업 조정, 종료 동작에 알려진 한계가 있습니다. **CLI 전용**이며 데스크톱 앱에서는 쓸 수 없습니다. 이 장은 공식 문서 기준이며 직접 실행해 보지는 못했습니다.

### 9.1 서브에이전트와의 차이

| | 서브에이전트 | 에이전트 팀 |
|---|---|---|
| 소통 | 결과를 부른 쪽에 보고 | **팀원끼리 직접 메시지** |
| 조정 | 메인이 전부 관리 | 공유 작업 목록 + 자율 조정 |
| 적합한 일 | 결과만 중요한 집중 작업 | 토론, 상호 검증이 필요한 복잡한 작업 |
| 토큰 비용 | 낮음 | **높음** (팀원마다 독립된 Claude 인스턴스) |

### 9.2 켜기

`~/.claude/settings.json` (또는 프로젝트 `.claude/settings.json`):

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

> ⚠️ 켜면 Claude가 **이름을 붙인 서브에이전트가 팀원으로 실행**되어, 요청하지 않아도 팀이 생길 수 있습니다.

### 9.3 팀 만들기 (모델 지정 포함)

```text
이 모듈들을 병렬로 리팩터링하려고 해. 팀원 3명을 만들어줘.
각 팀원은 Sonnet을 쓰고, 이름은 api, db, ui로 해줘.
각자 자기 폴더의 파일만 수정해.
```

```text
security-reviewer 에이전트 타입으로 팀원을 하나 만들어서 auth 모듈을 감사하게 해줘.
```

두 번째 예처럼 **6~7장에서 만든 서브에이전트 정의를 팀원 역할로 재사용**할 수 있습니다. 이때 정의의 `model`, `tools`가 적용됩니다.

### 9.4 조작법 (기본 in-process 모드)

| 키 | 동작 |
|---|---|
| `↑` / `↓` | 팀원 선택 |
| `Enter` | 팀원 기록 열기, 직접 메시지 보내기 |
| `Esc` | 선택 해제 (팀원 기록을 보는 중이면 그 팀원의 현재 턴 중단) |
| `x` | 선택한 팀원 중지 |
| `Ctrl+T` | 작업 목록 열고 닫기 |

팀원마다 별도 창(split pane)으로 보려면 tmux나 iTerm2가 필요합니다 (`"teammateMode": "auto"`).

### 9.5 팀원 종료

```text
researcher 팀원에게 종료하라고 해줘
```

세션이 끝나면 팀 설정 폴더(`~/.claude/teams/...`)는 **자동으로 정리**됩니다.

### 9.6 알아둘 한계

- 세션 하나에 팀 하나만. 팀원이 또 팀을 만들 수 없음
- `/resume`, `/rewind`로 in-process 팀원이 복구되지 않음
- 두 팀원이 **같은 파일을 수정하면 덮어쓰기** 발생 → 파일을 나눠 맡기세요
- 권장 규모: **팀원 3~5명**, 팀원당 작업 5~6개
- 공식 비용 문서: 팀원이 계획 모드로 실행되면 일반 세션보다 **약 7배** 토큰 사용

---

## 10. 데스크톱 앱 Code 탭에서 쓰기

데스크톱 앱 Code 탭은 CLI와 **같은 엔진**을 쓰고 설정을 공유합니다.

### 10.1 공유되는 것 (공식 문서 기준)

- `CLAUDE.md` 파일
- MCP 서버 설정 (`~/.claude.json`, `.mcp.json`)
- 훅(hooks)과 스킬(skills)
- `~/.claude/settings.json`의 설정과 권한 규칙
- 모델 목록

### 10.2 기능별 사용 가능 여부

| 기능 | Code 탭 | 방법 |
|---|---|---|
| 메인 모델 선택 | ✅ | 보내기 버튼 옆 **모델 드롭다운** (단축키 macOS `Cmd+Shift+I`) |
| 서브에이전트 | ✅ | 자연어로 요청. **Background tasks** 목록에서 항목을 누르면 **subagent 패널**에 출력 표시 |
| 백그라운드 작업 보기 | ✅ | 제목 표시줄 **⋮** 메뉴 → **Background tasks** (서브에이전트, 백그라운드 명령, 워크플로 표시. 클릭해서 출력 보기·중지) |
| 동적 워크플로 | ✅ | 승인 카드에 **Once / Always / Deny** 버튼. 진행 상황은 Background tasks 패널 |
| 에이전트 팀 | ❌ | **CLI 전용.** 데스크톱에서는 대신 동적 워크플로를 쓰라고 공식 문서가 안내 |
| 여러 세션 병렬 | ✅ | 사이드바의 세션 목록 |

### 10.3 `.claude/agents/` 파일도 Code 탭에서 쓸 수 있나?

- **확인된 사실:** 공식 문서는 Code 탭이 CLI와 "같은 엔진"을 쓰고 settings, skills, hooks, CLAUDE.md를 공유한다고 명시합니다.
- **추론:** 서브에이전트 정의 파일도 같은 엔진이 읽으므로 로컬 세션에서 동작할 것으로 보입니다. 단, 데스크톱 문서가 이 점을 **따로 명시하지는 않아서** 직접 확인하지는 못했습니다.
- **확인 방법:** Code 탭에서 `@`를 입력해 목록에 내 에이전트 이름이 나오는지 보거나, "scout 서브에이전트가 정의되어 있는지 확인해줘"라고 물어보세요.

### 10.4 클라우드 세션 (claude.ai/code)

클라우드 세션도 서브에이전트를 지원합니다. 이 문서를 작성한 클라우드 세션에서도 메인 에이전트가 서브에이전트를 띄울 때 `opus`, `sonnet`, `haiku` 중에서 모델을 고를 수 있었습니다. 저장소에 커밋한 `.claude/agents/` 파일은 클론된 저장소에 함께 들어갑니다.

---

## 11. 응용: 상황별 실전 시나리오

### 시나리오 1. 처음 보는 큰 저장소 파악하기

```text
이 저장소 구조를 파악하고 싶어.
Haiku 서브에이전트 3개를 병렬로 띄워서 각각
(1) 진입점과 라우팅, (2) 데이터 모델과 DB, (3) 테스트와 빌드 설정을 조사하게 하고,
결과를 네가 한 페이지 아키텍처 요약으로 정리해줘.
```

**효과:** 수백 개 파일을 읽는 일이 Haiku 쪽 컨텍스트에서 끝나고, Opus 메인에는 요약만 남습니다.

### 시나리오 2. 기능 구현 (계획 → 구현 → 검토)

[7장](#7-실습-opus-지휘자--sonnethaiku-작업자-팀-만들기)의 scout / implementer / reviewer 구조를 그대로 씁니다.

> 💡 **더 간단한 대안:** 모델 별칭 `opusplan`을 쓰면 **계획 모드에서는 Opus, 실행할 때는 Sonnet**으로 자동 전환됩니다. 서브에이전트 파일 없이도 "Opus로 계획, Sonnet으로 구현"을 할 수 있습니다.
> ```text
> /model opusplan
> ```
> 그다음 `Shift+Tab`으로 계획 모드에 들어가서 계획을 세우고, 승인한 뒤 실행하면 됩니다.

### 시나리오 3. 다각도 코드 리뷰

```text
현재 브랜치의 변경 사항을 리뷰해줘.
Sonnet 서브에이전트 3개를 병렬로 띄워서 각각 보안, 성능, 테스트 누락만 보게 하고,
결과를 중복 없이 심각도 순으로 합쳐줘. 각 지적에는 파일:줄번호를 붙여줘.
```

### 시나리오 4. 원인 모를 버그 (경쟁 가설)

```text
앱이 메시지 하나 보낸 뒤 연결이 끊기는 버그가 있어.
서브에이전트 3개를 띄워서 각각 다른 가설(네트워크 타임아웃, 세션 만료, 이벤트 핸들러 누수)을
검증하게 하고, 증거가 가장 강한 가설을 골라줘. 수정은 아직 하지 마.
```

> 가설끼리 **서로 반박**하게 하려면 에이전트 팀(CLI, 실험)이 더 적합합니다. 공식 문서도 이 용도를 대표 예시로 듭니다.

### 시나리오 5. 테스트 로그처럼 출력이 큰 작업 떼어내기

```text
test-runner 서브에이전트로 전체 테스트를 돌리고, 실패한 테스트 이름과 에러 메시지 첫 줄만 보고하게 해줘.
```

수천 줄의 테스트 출력이 메인 대화에 들어오지 않습니다. 공식 문서가 서브에이전트의 대표 용도로 꼽는 경우입니다.

### 시나리오 6. 저장소 전체 대량 작업 (워크플로)

```text
워크플로를 써서 src/ 아래 모든 파일에서 deprecated된 logger.warn 호출을 logger.warning으로 바꿔줘.
먼저 src/utils 폴더 하나에만 시험 실행하고 결과를 보여줘.
```

**작은 범위로 먼저 시험**하는 것은 공식 문서가 권장하는 비용 관리 방법입니다.

---

## 12. 응용: 고급 설정

### 12.1 모든 서브에이전트의 기본 모델 정하기

`settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_SUBAGENT_MODEL": "haiku"
  }
}
```

모델을 따로 지정하지 않은 서브에이전트, 팀원, 워크플로 에이전트가 Haiku로 실행됩니다. 정의 파일의 `model:`이 있으면 그쪽이 우선입니다.

**모든 서브에이전트를 강제로 한 모델로:** `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1`을 함께 설정하면 정의 파일이나 요청 시 지정보다 이 값이 우선합니다.

### 12.2 특정 서브에이전트 금지하기

```json
{
  "permissions": {
    "deny": ["Agent(Explore)"]
  }
}
```

`Agent` 도구 자체를 금지하면 서브에이전트 위임이 전부 막힙니다.

### 12.3 서브에이전트가 또 서브에이전트를 부르는 깊이

기본적으로 메인 아래 **3단계까지** 가능합니다.

```json
{
  "env": {
    "CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH": "2"
  }
}
```

`1`로 설정하면 중첩이 꺼집니다. 특정 에이전트만 막으려면 그 정의의 `tools`에서 `Agent`를 빼거나 `disallowedTools`에 넣습니다.

**동시 실행 서브에이전트 수:** 기본 20개. `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`로 변경.

### 12.4 어떤 서브에이전트를 부를 수 있는지 제한하기

```yaml
tools: Agent(scout, implementer), Read, Bash
```

이 에이전트는 `scout`, `implementer`만 띄울 수 있습니다.

### 12.5 파일 충돌 막기: worktree 격리

```yaml
isolation: worktree
```

서브에이전트가 별도의 git 작업 복사본에서 일합니다. 변경이 없으면 자동으로 정리됩니다. 여러 서브에이전트가 동시에 파일을 고칠 때 유용합니다.

### 12.6 서브에이전트 시작·종료 시 자동 동작 (훅)

`settings.json`의 `SubagentStart`, `SubagentStop` 훅은 에이전트 이름으로 대상을 고를 수 있습니다 (예: `"matcher": "implementer"`). 정의 파일 안에 `hooks:`를 넣으면 **그 서브에이전트가 실행되는 동안만** 동작합니다. 공식 문서 예시는 Bash 명령을 검사해 DB 쓰기 쿼리를 막는 `PreToolUse` 훅입니다.

### 12.7 팀에 배포하기

- **프로젝트 공유:** `.claude/agents/`, `.claude/workflows/`를 git에 커밋
- **여러 저장소에 배포:** 플러그인으로 묶기. 플러그인 에이전트는 `플러그인이름:에이전트이름`으로 불림. 단, 보안상 플러그인 에이전트에서는 `hooks`, `mcpServers`, `permissionMode`가 무시됨

---

## 13. 비용 관리

### 13.1 꼭 알아야 할 사실

- 서브에이전트, 워크플로 에이전트, 팀원은 **각자 따로 요청을 보냅니다.** 메인 대화 비용에 **더해집니다.**
- 구독 요금제(Pro, Max 등)에서는 이 사용량이 **사용 한도에서 차감**됩니다.
- 멀티에이전트는 채팅 대비 약 **15배**, 에이전트 팀은 계획 모드 기준 약 **7배** 토큰을 쓴다는 공식 수치가 있습니다 (1.4절, 9.6절).

### 13.2 비용을 줄이는 방법

| 방법 | 효과 |
|---|---|
| 단순 작업 서브에이전트에 `model: haiku` 명시 | 7.8절 시험에서 Haiku 서브에이전트 비용은 Sonnet 메인의 약 1/70 |
| 서브에이전트 보고 길이 제한 ("최대 30줄") | 메인(Opus)으로 돌아오는 토큰 감소 |
| `tools`를 꼭 필요한 것만 | 불필요한 탐색 감소 |
| 작은 작업은 서브에이전트 없이 메인에서 | 서브에이전트는 맥락을 새로 읽어야 해서 작은 일엔 오히려 손해 |
| 워크플로는 작은 범위로 먼저 시험 | 공식 권장 |
| 다 쓴 팀원은 바로 종료 | 활성 팀원은 계속 토큰 사용 |
| 관련 없는 작업 사이에 `/clear` | 쌓인 컨텍스트 비용 제거 |

### 13.3 사용량 확인

```text
/usage
```

구독 요금제에서는 **스킬, 서브에이전트, 플러그인, MCP 서버별 사용 비중**도 보여줍니다. `d`/`w`로 최근 24시간/7일 전환.

---

## 14. 끄기와 삭제하기

필요한 범위만 골라서 하세요. **위에서 아래로 갈수록 영향이 큽니다.**

### 14.1 실행 중인 작업 멈추기

| 대상 | 방법 |
|---|---|
| 현재 진행 중인 응답 | `Esc` |
| 백그라운드 서브에이전트 | `/tasks`에서 선택 후 중지. 데스크톱은 Background tasks 패널에서 중지 |
| 워크플로 | `/workflows`에서 선택 후 `x` |
| 에이전트 팀 팀원 | 팀원 선택 후 `x`, 또는 "OO 팀원 종료해줘" |
| 남은 tmux 세션 (팀 split-pane 모드) | `tmux ls` → `tmux kill-session -t <세션이름>` |

### 14.2 서브에이전트 하나 삭제하기

**정의 파일을 지우면 됩니다.** (현재 버전에는 별도 삭제 명령이 없습니다.)

```bash
# 프로젝트용
rm .claude/agents/scout.md

# 개인용 (모든 프로젝트)
rm ~/.claude/agents/scout.md
```

```powershell
# Windows PowerShell
Remove-Item .claude\agents\scout.md
Remove-Item "$env:USERPROFILE\.claude\agents\scout.md"
```

> ✅ **실행 검증:** 파일을 지운 뒤 새 세션에서 "file-scout가 정의되어 있는지" 물었더니 **"없다"**고 답했고, 사용 가능한 에이전트 목록에서도 사라졌습니다.

> ⚠️ 프로젝트 폴더의 파일을 git에 커밋했다면, 지운 것도 커밋해야 팀원에게서도 사라집니다.
> ```bash
> git rm .claude/agents/scout.md
> git commit -m "Remove scout subagent"
> ```

**만든 서브에이전트 전부 지우기:**

```bash
rm -rf .claude/agents        # 이 프로젝트의 서브에이전트 전부
rm -rf ~/.claude/agents      # 개인 서브에이전트 전부
```

**플러그인으로 설치한 에이전트:** `/plugin`에서 해당 플러그인을 비활성화하거나 삭제합니다 (데스크톱은 **+** → **Plugins** → **Manage plugins**).

### 14.3 저장한 워크플로 삭제하기

```bash
rm .claude/workflows/<이름>.js      # 프로젝트용
rm ~/.claude/workflows/<이름>.js    # 개인용
```

같은 세션에 바로 반영하려면 `/reload-skills`를 실행합니다.

### 14.4 기능 자체를 끄기

| 끌 대상 | 방법 |
|---|---|
| **동적 워크플로** | `/config`의 Dynamic workflows 끄기, 또는 `settings.json`에 `"disableWorkflows": true`, 또는 환경 변수 `CLAUDE_CODE_DISABLE_WORKFLOWS=1` |
| ultracode 모드만 | `/effort ultracode off` |
| `ultracode` 키워드 자동 인식 | `/config`의 Ultracode keyword trigger 끄기 |
| **에이전트 팀** | `settings.json`에 `"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "0"` (또는 줄 삭제). 새 세션 없이 저장하면 바로 반영 |
| 기본 Explore, Plan 서브에이전트 | `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS=1` |
| 서브에이전트 위임 전체 | `permissions.deny`에 `Agent` 추가 |
| 서브에이전트 기본 모델 설정 원복 | `settings.json`에서 `CLAUDE_CODE_SUBAGENT_MODEL` 줄 삭제 |

### 14.5 Claude Code 완전히 삭제하기

**1단계. 프로그램 삭제** (설치한 방법에 맞게)

```bash
# 네이티브 설치 (macOS, Linux, WSL)
rm -f ~/.local/bin/claude
rm -rf ~/.local/share/claude
```

```powershell
# 네이티브 설치 (Windows PowerShell)
Remove-Item -Path "$env:USERPROFILE\.local\bin\claude.exe" -Force
Remove-Item -Path "$env:USERPROFILE\.local\share\claude" -Recurse -Force
```

```bash
brew uninstall --cask claude-code            # Homebrew (최신 채널은 claude-code@latest)
winget uninstall Anthropic.ClaudeCode        # WinGet
npm uninstall -g @anthropic-ai/claude-code   # npm
```

**2단계. 설정과 데이터 삭제 (선택)**

> ⚠️ **되돌릴 수 없습니다.** 모든 설정, 허용 도구, MCP 서버 설정, 세션 기록, 개인 서브에이전트와 워크플로가 삭제됩니다. 필요하면 먼저 백업하세요.
> ⚠️ VS Code 확장, JetBrains 플러그인, 데스크톱 앱도 `~/.claude/`를 씁니다. 이것들이 설치되어 있으면 폴더가 다시 생깁니다. 완전히 지우려면 이것들을 먼저 삭제하세요.

```bash
# macOS, Linux, WSL
rm -rf ~/.claude
rm ~/.claude.json

# 프로젝트별 설정 (프로젝트 폴더에서 실행)
rm -rf .claude
rm -f .mcp.json
```

```powershell
# Windows PowerShell
Remove-Item -Path "$env:USERPROFILE\.claude" -Recurse -Force
Remove-Item -Path "$env:USERPROFILE\.claude.json" -Force
Remove-Item -Path ".claude" -Recurse -Force
Remove-Item -Path ".mcp.json" -Force
```

**3단계. 확인**

```bash
claude --version
```

`command not found`가 나오면 삭제 완료입니다. 여전히 실행되면 다른 방법으로 설치된 사본이 남아 있는 것입니다 ([공식 안내](https://code.claude.com/docs/en/troubleshoot-install)).

---

## 15. 문제 해결

| 증상 | 원인 | 해결 |
|---|---|---|
| 만든 서브에이전트를 못 찾음 | 새 `agents` 폴더에 첫 파일을 만든 뒤 재시작 안 함 | Claude Code 재시작 |
| 서브에이전트 설정이 무시됨 | frontmatter 항목 오타 (모르는 항목은 조용히 무시됨) 또는 대소문자 틀림 | `disallowedTools`처럼 정확한 camelCase로 |
| 같은 이름 에이전트가 다르게 동작 | 우선순위가 높은 위치에 같은 이름 파일 존재 | 6.3절 우선순위 표 확인 |
| 서브에이전트가 Opus로 돌아서 비쌈 | 정의에 `model:` 없음 → 메인 모델 상속 | `model: haiku` 또는 `sonnet` 명시 |
| Claude가 서브에이전트를 안 씀 | `description`이 모호하거나 요청에 지목이 없음 | description을 구체적으로, 또는 `@agent-이름`으로 지목 |
| 시작할 때 description 경고 | 모든 커스텀 서브에이전트 설명 합계가 15,000토큰 초과 | 설명은 짧게, 자세한 내용은 본문으로 |
| 원하는 모델로 안 바뀜 | 조직의 `availableModels` 제한 | 관리자에게 문의 |
| 팀원 대신 서브에이전트가 생김 | Claude 판단 | "에이전트 팀을 만들어줘"라고 명시 |
| 서브에이전트 대신 팀원이 생김 | 에이전트 팀이 켜져 있음 | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`를 `0`으로 |
| 데스크톱에서 에이전트 팀이 안 됨 | CLI 전용 기능 | 터미널에서 실행, 또는 동적 워크플로 사용 |
| `/agents`에 마법사가 안 뜸 | v2.1.198부터 안내 메시지만 출력 | 파일을 직접 만들거나 Claude에게 요청 |
| 권한 요청이 너무 많음 | 서브에이전트, 팀원의 권한 요청이 메인으로 올라옴 | 자주 쓰는 명령을 `settings.json`의 allow 규칙에 미리 추가 |

---

## 16. 자주 묻는 질문

**Q1. 오케스트레이션을 쓰려면 따로 설치할 게 있나요?**
아니요. 서브에이전트와 동적 워크플로는 Claude Code에 내장되어 있습니다. 에이전트 팀만 설정 한 줄로 켜야 합니다.

**Q2. Opus, Sonnet, Haiku만으로 구성할 수 있나요?**
네. 메인을 `opus`로 두고, 서브에이전트 정의에 `model: sonnet`, `model: haiku`를 적으면 됩니다. 7장의 실습이 정확히 이 구성입니다. 7.8절에서 실제로 모델이 나뉘어 실행되는 것을 확인했습니다.

**Q3. 무조건 멀티에이전트가 더 좋은가요?**
아닙니다. Anthropic도 "가장 단순한 해법부터"를 권장합니다. 리서치처럼 병렬로 나눌 수 있는 일에서 효과가 크고, 서로 의존하는 작업이나 같은 파일을 고치는 작업은 메인 하나가 낫습니다. 작은 작업은 서브에이전트를 쓰면 오히려 비용과 시간이 늘어납니다.

**Q4. 서브에이전트에게 이전 대화 내용이 전달되나요?**
기본적으로 아닙니다. 서브에이전트는 맥락 없이 시작하므로 지시에 목적, 범위, 보고 형식을 담아야 합니다. 전체 대화 맥락을 물려받는 **포크(fork) 서브에이전트**도 있습니다 (`/subtask` 명령).

**Q5. `opusplan`과 서브에이전트 구성은 무엇이 다른가요?**
`opusplan`은 **한 대화 안에서** 계획 모드일 때 Opus, 실행할 때 Sonnet으로 모델만 바꿉니다. 서브에이전트 구성은 **별도 작업자**에게 일을 나눠서 컨텍스트를 분리합니다. 간단하게 시작하려면 `opusplan`, 역할 분담과 컨텍스트 절약이 필요하면 서브에이전트를 쓰세요.

**Q6. 무료 플랜에서도 되나요?**
아니요. Claude Code 자체가 Pro, Max, Team, Enterprise, Console 계정이 필요합니다.

**Q7. 서브에이전트가 파일을 망가뜨리면요?**
`tools`로 읽기 전용(`Read, Grep, Glob`)으로 만들거나, `isolation: worktree`로 별도 작업 복사본에서 일하게 하세요. 메인 대화의 체크포인트는 `/rewind`(또는 `Esc` 두 번)로 되돌릴 수 있습니다.

---

## 17. 출처

### Claude Code 공식 문서 (2026-10-08 확인)

- [Run agents in parallel](https://code.claude.com/docs/en/agents): 병렬 작업 5가지 비교
- [Create custom subagents](https://code.claude.com/docs/en/sub-agents): 서브에이전트 생성, frontmatter, 호출, 모델 결정 순서
- [Orchestrate subagents at scale with dynamic workflows](https://code.claude.com/docs/en/workflows): 동적 워크플로
- [Orchestrate teams of Claude Code sessions](https://code.claude.com/docs/en/agent-teams): 에이전트 팀
- [Advanced setup](https://code.claude.com/docs/en/setup): 설치, 업데이트, 삭제
- [Model configuration](https://code.claude.com/docs/en/model-config): 모델 별칭, `opusplan`, `CLAUDE_CODE_SUBAGENT_MODEL`
- [Manage costs effectively](https://code.claude.com/docs/en/costs): 비용, 에이전트 팀 토큰 비용
- [Desktop application](https://code.claude.com/docs/en/desktop): 데스크톱 앱 Code 탭

### Anthropic 엔지니어링 블로그

- [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) (2024-12-19): 워크플로와 에이전트 패턴
- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) (2025-06-13): 오케스트레이터-워커 구조와 성능, 토큰 수치

### 직접 실행 검증

- Claude Code 2.1.294, Linux 환경, `claude -p ... --output-format json`으로 서브에이전트 모델 분리와 삭제 동작 확인 (7.8절, 14.2절)
