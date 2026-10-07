# 🐹 Mole 전수조사 & 활용 전략 리포트 (한국어)

> 이 문서는 Claude Code 세션에서 Mole 저장소를 전수조사하고,
> 설치/사용법 · 성격(플러그인/스킬/MCP) · AI 에이전트 활용 · 수익화 · 구현 스택 ·
> 콘텐츠 제작 가능성까지 논의한 내용을 정리한 기록입니다.
>
> 작성일: 2026-10-07

---

## 🔗 GitHub 주소

| 구분 | 주소 |
| --- | --- |
| **원본(Upstream)** | https://github.com/tw93/mole |
| **이 저장소(Fork)** | https://github.com/bmshin94/Mole |
| 작업 브랜치 | `claude/beautiful-wozniak-o5imcl` |
| 릴리스 | https://github.com/tw93/mole/releases |
| 유료 Mac 앱 (원작자) | https://mole.fit/ |
| Windows 실험 브랜치 | https://github.com/tw93/Mole/tree/windows |
| 튜토리얼 영상 (중국어) | https://www.youtube.com/watch?v=UEe9-w4CcQ0 |

---

## 1. 프로젝트 정체

**macOS 전용 올인원 청소/최적화 CLI 도구.**
CleanMyMac + AppCleaner + DaisyDisk + iStat Menus 를 터미널 명령어 한 세트로 대체합니다.

| 항목 | 내용 |
| --- | --- |
| 원작자 | `tw93` (Twitter @HiTw93) |
| 이 저장소 | `bmshin94/Mole` (포크) |
| 버전 | `1.54.0` |
| 라이선스 | **GPL v3** (상업 이용 가능하나 소스 공개 의무) |
| 상표 | `TRADEMARK.md` 존재 — 이름 상업적 사용 주의 |
| 지원 환경 | macOS 12+, Intel / Apple Silicon |
| 커밋 히스토리 | 50개 (포크 시점 shallow) |
| 비즈니스 모델 | 오픈코어 — CLI 무료 OSS + `Mole for Mac` 유료 앱 |

> ⚠️ 이 세션은 Linux 컨테이너라서 `mo` 실행은 불가능합니다. 코드 열람/수정만 가능.
> `mole` 11행에서 root(EUID 0) 실행을 명시적으로 차단합니다.

### 포크 상태 특이사항

원본에서 `CLAUDE.md`는 `AGENTS.md`의 **심볼릭 링크**였지만,
이 포크에서는 삭제 후 실제 파일로 재업로드되어 **링크가 끊어진 상태**입니다.
(한글 가이드 + 페르소나 추가를 유지하려면 이 상태가 맞지만,
`AGENTS.md`를 수정해도 `CLAUDE.md`에 반영되지 않고 upstream 머지 시 충돌 가능.)

---

## 2. 폴더 구조 & 규모

```
Mole/
├── mole                  🚦 CLI 진입점 (라우터 전용)
├── mo                    짧은 별칭
├── install.sh            설치 스크립트 (65KB, 체크섬/attestation 검증)
│
├── bin/                  명령어 오케스트레이터 (셸)
│   ├── clean.sh (2,107줄) / uninstall.sh (1,993줄) / installer.sh (873줄)
│   ├── purge.sh / optimize.sh / touchid.sh / completion.sh / history.sh
│   └── analyze.sh · status.sh → Go 바이너리 래퍼
│
├── lib/                  🧠 비즈니스 로직 (46,897줄)
│   ├── core/   file_ops.sh(3,682) ⭐ 삭제 깔때기
│   │           app_protection.sh(2,299) ⭐ 보호 판정
│   │           app_protection_data.sh(627) 보호 번들ID 데이터
│   │           base / common / ui / sudo / timeout / history / log / bundle_resolver
│   ├── clean/  dev.sh(5,355) user.sh(2,869) project.sh(2,715)
│   │           apps.sh(2,101) app_caches.sh(1,952) system.sh(1,478)
│   │           caches.sh(803) hints.sh(518) brew.sh purge_shared.sh
│   ├── uninstall/ batch.sh(2,728) brew.sh steam.sh
│   ├── optimize/  tasks.sh(2,150) diagnostics.sh(694) catalog / maintenance / outcomes
│   ├── ui/        menu_paginated.sh(1,011, Bash 3.2 호환) menu_simple / app_selector
│   └── manage/    update.sh(1,353) remove.sh whitelist.sh(501) purge_paths.sh
│
├── cmd/                  🐹 Go (24,071줄)
│   ├── analyze/  Bubble Tea TUI 디스크 탐색기
│   │             scanner.go(34KB) update.go(34KB) cache.go view.go delete.go json.go
│   │             analyze_test.go(108KB!) delete_fuzz_test.go
│   └── status/   실시간 모니터
│                 metrics_{cpu,gpu,memory,disk,network,battery,bluetooth,process,health}.go
│                 view.go(34KB) view_test.go(52KB) watch.go prefs.go
├── internal/units/       바이트 포맷 유틸
│
├── tests/                🧪 Bats 70개 + expect(.exp) PTY 테스트 + 퍼즈 코퍼스
├── scripts/              check.sh / test.sh / 감사 3종
│                         audit_destructive_sinks.py (# SAFE 주석 강제)
│                         audit_function_duplication.py (본문 해시 중복 감지)
│                         audit_bundle_drift.sh (월간 번들ID 드리프트 감사)
├── docs/SECURITY_DESIGN.md   보안 설계 문서
├── SECURITY_AUDIT.md         보안 감사 노트 (25KB)
│
├── AGENTS.md (45KB)      🤖 멀티 AI 공통 계약서 (단일 진실 소스)
├── CLAUDE.md (48KB)      Claude Code용 (이 포크에선 실제 파일)
├── GEMINI.md             Gemini용
├── .claude/              스킬 4 + 서브에이전트 2 + 훅 1 + settings.json
├── .agents/skills/       Codex용 상대 심볼릭 링크
├── .cursor/rules/        Cursor용 규칙
└── .github/workflows/    CI 5개 (check / test / codeql / release / bundle_audit)
```

| 지표 | 값 |
| --- | --- |
| 셸 스크립트 | 50,681줄 |
| Go 코드 | 24,071줄 |
| Bats 테스트 파일 | 70개 |
| Go 의존성 | bubbletea, lipgloss, gopsutil, xxhash, golang.org/x/sys |
| Go 버전 | 1.26.0 |

---

## 3. 명령어 12종

| 명령어 | 기능 |
| --- | --- |
| `mo` | 인터랙티브 메뉴 (↑↓ + Enter) |
| `mo clean` | 캐시·로그·임시파일 + **삭제된 앱의 잔여물** 청소 |
| `mo uninstall` | 앱 + 숨은 잔여물(LaunchAgent, plist, 캐시) 완전 삭제 |
| `mo optimize` | DNS 플러시, Spotlight 재색인, 아이콘 캐시 재구축, DB 최적화 |
| `mo analyze` / `analyse` | 터미널 디스크 탐색기 (Vim 키, 휴지통 경유 삭제) |
| `mo status` | 실시간 CPU/GPU/메모리/디스크/네트워크/배터리 + 건강점수 |
| `mo purge` | `node_modules`, `target`, `dist`, `build`, `.build` 등 재생성 가능 산출물 |
| `mo installer` | DMG/PKG/MPKG/ISO/XIP/설치ZIP 정리 |
| `mo history` | 삭제 이력 (`--json`) |
| `mo touchid` | sudo 지문인식 설정 |
| `mo update` | 자체 업데이트 (`--force`, `--nightly`) |
| `mo remove` | Mole 자체 제거 |

---

## 4. 설치 및 사용법

### 설치

```bash
# ① Homebrew (권장)
brew install mole

# ② 스크립트
curl -fsSL https://raw.githubusercontent.com/tw93/mole/main/install.sh | bash

# ③ 유저 디렉토리 (이후 update 시 비밀번호 불필요)
mkdir -p "$HOME/.local/bin"
curl -fsSL https://raw.githubusercontent.com/tw93/mole/main/install.sh | bash -s -- --prefix "$HOME/.local/bin"
export PATH="$HOME/.local/bin:$PATH"   # ~/.zshrc 에도 추가

# 특정 버전 / 개발 버전
curl -fsSL .../install.sh | bash -s -- 1.51.0
curl -fsSL .../install.sh | bash -s -- main     # 미릴리스 main (rough edges)
```

### 설치 후 첫 세팅 권장 순서

```bash
mo --version          # 확인
mo completion         # 탭 자동완성
mo touchid            # sudo 지문인식
mo clean --dry-run    # ⭐ 반드시 미리보기 먼저
mo clean              # 실제 실행
```

### 자주 쓰는 플래그

```bash
mo clean --dry-run --debug      # 미리보기 + 상세 로그
mo clean --whitelist            # 보호 캐시 지정 (~/.config/mole/whitelist)
mo optimize --whitelist         # 제외 작업/경로 패턴
mo purge --paths                # 스캔 디렉토리 (~/.config/mole/purge_paths)
mo purge --yes                  # 비대화형 (CI 필수 플래그)
mo analyze /Volumes             # 외장 드라이브
mo analyze --json ~/Documents   # JSON 리포트
mo status --json                # 스냅샷
mo status --watch --interval 2s # NDJSON 스트리밍
mo status | jq '.health_score'  # 파이프 시 자동 JSON 전환
```

### 환경변수 (API 토큰 아님)

```bash
MOLE_DRY_RUN=1        # 미리보기 강제
MOLE_TEST_NO_AUTH=1   # 인증 프롬프트 없이 테스트
MO_NO_OPLOG=1         # 작업 로그 비활성화
MO_DEBUG=1            # 상세 로그
MOLE_ENABLE_DISK_VERIFY=1
```

### TUI 키 조작

- `mo purge`: `↑↓`/`jk` 이동, `PgUp/PgDn`/`hl` 페이지, `[` `]` 프로젝트 점프,
  `X` 프로젝트 건너뛰기, `/` 검색, `n` 다음 결과, `Enter` 최종 리뷰
- `mo status`: `k` 고양이 토글, `c` CPU 코어 수 순환, `q` 종료

### 안전 수칙

- ❌ `sudo mo` 금지 (코드에서 차단, 필요 시 Mole이 직접 권한 요청)
- ✅ 새 명령어는 항상 `--dry-run` 먼저
- 로그: `~/Library/Logs/mole/operations.log` → `mo history`
- `mo analyze` 삭제는 휴지통 경유(복구 가능) / `mo purge`는 영구 삭제(재빌드로 복구)

### Raycast / Alfred 런처

```bash
curl -fsSL https://raw.githubusercontent.com/tw93/Mole/main/scripts/setup-quick-launchers.sh | bash
```
Raycast는 수동 한 단계: Settings > Extensions > Script Commands 에
`~/Library/Application Support/Raycast/script-commands` 추가 → Reload Script Directories

---

## 5. 안전성 설계 (이 프로젝트의 진짜 가치 ①)

### 4중 레이어

1. **단일 삭제 깔때기** — 모든 삭제는 `mole_delete` / `safe_remove` / `safe_sudo_remove`
   를 통과. 날것의 `rm -rf`, `find -delete` 는 같은 줄에 `# SAFE: <이유>` 주석이 있어야
   하고, `scripts/audit_destructive_sinks.py` 가 `check.sh` 와 CI에서 이를 강제.
2. **경로 검증** — `validate_path_for_deletion` 의 6개 독립 검사.
   `/System`, `/bin`, `/sbin`, `/usr`(단 `/usr/local` 제외), `/etc`, `/var/db`,
   `/Library/Extensions`, `com.apple.*` 금지. 심볼릭 링크 해석 후 재검사.
   `/Applications`, `/Library`, `/Volumes`, `/Users`, 홈 루트 등 **최상위 자체** 삭제 금지
   → `"$dir/$name"` 에서 `$name` 이 비는 **빈 변수 붕괴 사고**를 구조적으로 차단.
3. **증거 기반 판정 (fail-closed)** — 프로세스 상태 tri-state
   (`0` 실행중 / `1` 아님 / `2` 알 수 없음)에서 **`2`는 거부**.
   번들 ID·경로 정확 매칭만 허용, 벤더 접두사·일반명 와일드카드 금지.
   앱 캐시는 소유 프로세스 상태 확인 후에만, 열린 파일 핸들(`lsof`) 확인.
4. **복구 가능성 + 감사** — 사용자 파일은 휴지통 경유,
   `~/Library/Logs/mole/operations.log` 에 전량 기록, `--dry-run` 미리보기.

### 테스트 전략

- **퍼징**: `cmd/analyze/delete_fuzz_test.go`, `tests/path_validation_fuzz.bats`
  (+ `tests/fuzz_corpus/`) — 기계가 생성한 괴상한 경로로 검증 로직 공격
- **PTY/expect**: `timeout_tty_*.exp`, `uninstall_tty_foreground.exp`, `color_tty.exp`
  — ESC 타임아웃, 터미널 복구, 백그라운드/동시성 동작까지 검증
- **사고 회귀 테스트**: 과거 이슈별 전용 테스트 고정
  (`#1459` purge, `#1505` 시뮬레이터 런타임, `#1520` 앱 표시명,
  `#1558`/`#1579` brew 소유권, `#1584` 리프트오버 타임아웃, `#1348` lockf 부재 등)

### 시스템 통합 (왜 셸이어야 하는가)

`mdfind` `mdls` `lsof` `launchctl` `pmset` `ioreg` `diskutil` `df` `sysctl` `vm_stat`
`osascript` `PlistBuddy` `xcrun simctl` `brew` — 동작의 대부분이 macOS 네이티브 바이너리 호출.

---

## 6. AI 에이전트 인프라 (이 프로젝트의 진짜 가치 ②)

### 멀티 AI 도구 단일 진실 소스

```
AGENTS.md (45KB)  ← 원본 계약서
   ├── CLAUDE.md        (원본: 심볼릭 링크 / 이 포크: 실제 파일)
   ├── GEMINI.md        Gemini
   ├── .cursor/rules/   Cursor
   └── .agents/skills/  Codex (상대 심볼릭 링크)
```

### AGENTS.md 7단 구조 (그대로 차용 가능한 템플릿)

1. **Product Direction** — 무엇을 만들고 무엇을 안 만드는가
2. **What Should NOT Do** — ⭐ 명시적 거부 목록 (AI의 기능 폭주 방지)
3. **Product Decision Filter** — 새 기능 수용 판단 5개 질문
4. **Repository Map** — 파일별 책임
5. **Critical Safety Rules** — 이슈 번호 포함 금지 사항
6. **Hotspot Ownership** — "이 파일 건드리면 이 테스트를 돌려라"
7. **Verification** — 검증 방법 (+ 잘못된 검증 방법까지 명시)

특히 `No` 리스트의 문구가 인상적:
> "A new flag, environment variable, or config key is the same weight as a new setting:
> it passes only when no single default is right for everyone."

### `.claude/` 구성

| 종류 | 이름 | 역할 |
| --- | --- | --- |
| 스킬 | `mole` | 에이전트가 사용자 Mac에서 `mo`를 안전하게 쓰는 규약 — 추측 금지, TUI에 결정 위임 금지, 사용자가 목록을 못 본 파괴적 명령 금지 |
| 스킬 | `bugs` | 과거 사고 카탈로그 (라우터형 — 증거에 해당하는 reference만 로드) |
| 스킬 | `release-flow` | 릴리스 런북 (대문자 `V` 태그, 배포 채널) |
| 스킬 | `release-notes` | 릴리스 노트 (`disable-model-invocation: true`) |
| 에이전트 | `safety-reviewer` | 파괴적 동작 회귀 감사 — `tools: Read, Grep, Glob, Bash` (**수정 권한 없음**) |
| 에이전트 | `bash32-portability-reviewer` | macOS Bash 3.2 / errexit / TTY / BSD 툴 호환성 감사 |
| 훅 | `format-on-edit.sh` | `PostToolUse` + `Edit\|MultiEdit\|Write` → `shfmt -i 4 -ci -sr -w` / `goimports` 자동 실행. 심볼릭 링크 비추적, 레포 외부 경로 거부 |

### 배울 핵심 패턴 5가지

1. 계층형 컨텍스트 설계 (7단 구조)
2. **"No" 리스트가 "Yes" 리스트보다 중요** — AI의 기능 추가 폭주 억제
3. **사고 → 규칙 → 테스트 삼종 세트** — AI가 안전장치를 "최적화"로 지우면 테스트가 빨개짐
4. **읽기 전용 감사 서브에이전트** — 감사관에게 수정 권한을 주지 않는 권한 분리
5. 훅으로 포맷팅 자동화 → AI 토큰을 로직에만 사용

> "Treat specialist or AI review output as a claim to verify, never as approval."

---

## 7. 성격 분류: 플러그인? 스킬? MCP?

**정답: 세 개 다 아님. 독립 실행형 CLI 도구** (`git`, `brew`, `ffmpeg` 와 같은 범주).

| 구분 | Mole은? | 근거 |
| --- | --- | --- |
| CLI 도구 | ✅ **본체가 이것** | `mole` 라우터 + `bin/` + `lib/` + Go 바이너리 |
| Claude Skill | ✅ 레포 안에 4개 포함 | `.claude/skills/*/SKILL.md` (도구의 보조물) |
| Claude Plugin | ❌ 아님 | `plugin.json` 없음, 플러그인 패키징 아님 |
| MCP 서버 | ❌ 아님 | MCP/JSON-RPC 코드 전무 |

**API 토큰: 불필요.** 100% 로컬 오프라인 동작.
네트워크는 설치/업데이트 시 GitHub 공개 엔드포인트 익명 접근뿐.
대신 "권한"이 필요 — 일반 유저로 시작해 시스템 파일에서만 좁게 sudo 요청(`mo touchid`로 지문 대체).
(단, 이 레포로 Claude Code를 쓰는 비용은 별개 사안.)

---

## 8. AI 에이전트 구축에 도움되는가

### (A) 운영 레퍼런스 — ⭐⭐⭐⭐⭐
위 6절의 7단 구조 · No 리스트 · 사고→규칙→테스트 루프 · 읽기전용 감사 에이전트 · 훅 자동화.
**바로 훔쳐 쓸 것 3개**: ① AGENTS.md 7단 구조 ② safety-reviewer 패턴
③ 사고→규칙→테스트 루프.

### (B) 에이전트의 도구(Tool) — ⭐⭐⭐
```bash
mo status --json / mo analyze --json / mo history --json   # 읽기 전용
mo clean --dry-run                                        # 계획 단계
mo status --watch                                         # NDJSON 스트리밍
```
좋은 도구 설계 요소: 파이프 시 자동 JSON 전환(isatty), 파괴적 작업의 dry-run,
의미 있는 종료 코드, 비대화형 `--yes` 강제.

### (C) 불가한 것
MCP 서버 아님(직접 래퍼 작성 필요) · LLM 호출 코드 없음 · macOS 전용(Linux 에이전트 불가).

### MCP 래핑 설계안
```
mole_status / mole_analyze / mole_history   → 읽기 전용 (안전)
mole_clean_plan (--dry-run)                 → 계획만
실제 삭제                                   → 사람 승인 게이트 필수
```

---

## 9. React / PHP 구현 가능성

**결론: "껍데기는 가능, 엔진은 불가능."**
동작의 90%가 macOS 네이티브 바이너리 호출이므로 OS 접근 권한이 전제.

| 스택 | 가능? | 비고 |
| --- | --- | --- |
| 브라우저 React 단독 | ❌ 영구 불가 | 샌드박스가 파일시스템 접근 금지 |
| Electron + React | ✅ 가능 | `child_process` 로 CLI 호출, 번들 ~150MB |
| **Tauri + React** | ✅ **권장** | Rust 백엔드, 번들 ~10MB |
| React Native macOS | ⚠️ 어려움 | 생태계 미성숙 |
| PHP 데스크톱 | ❌ 비추 | GUI 생태계 없음, macOS 12+ 에 PHP 미번들 |
| **PHP 서버 대시보드** | ✅ **적합** | `mo status --json` 수집 → Laravel + 웹 대시보드 (B2B와 연결) |

### 언어별 적합도
🥇 Bash + Go(현재) → 🥈 Swift/SwiftUI → 🥉 Rust(Tauri) → Go → Node/Electron → Python
→ ❌ 브라우저 React, PHP 데스크톱

### 추천 React 사이드 프로젝트
1. **Mole Web Dashboard** — `mo status --json` → Recharts 실시간 차트 (1주, 포트폴리오)
2. **History Visualizer** — `mo history --json` → 타임라인/히트맵
3. **Mac Fleet Monitor** — 다수 Mac 수집 → 통합 대시보드 (수익화 가능)

> ⚖️ Mole을 별도 프로세스로 호출하는 GUI는 "느슨한 결합" 해석이 다수지만 논쟁적.
> 상업화 전 변호사 상담 필요. 또한 원작자가 이미 `mole.fit` 유료 앱 판매 중 → 직접 경쟁 비권장.

---

## 10. 유튜브 강의 영상 제작

**가능하며, 법적 리스크가 가장 낮은 수익화 경로.**
GPL v3는 코드 배포를 규율 — 교육/영상 콘텐츠는 해당 없음.
(원작자 `tw93` 와 레포 URL 크레딧은 예의. "공식 채널"처럼 보이는 브랜딩은 상표 문제.)

**시장 검증**: README에 중국어권 유튜버(PAPAYA 電腦教室) 영상이 이미 등재 →
**한국어 콘텐츠는 공백 = 블루오션.**

### 3트랙 기획

**트랙 A — 일반 Mac 유저 (조회수)**
1. 맥북 용량 부족? 명령어 하나로 20GB 확보 (8분)
2. CleanMyMac 연 4만원 → 무료 대안 (10분)
3. 맥에서 앱 지워도 찌꺼기가 남는다 — 완전 삭제법 (7분)
4. 터미널로 맥 실시간 모니터링 (iStat 대체) (6분)
5. 개발자 맥북에 숨어있는 50GB 찾기 (12분)

**트랙 B — 개발자 심화 (충성도)**
1. 5만줄 Bash 프로젝트 해부
2. 파일 삭제 도구를 안전하게 만드는 4가지 레이어
3. Bubble Tea로 터미널 UI 만들기 (Go)
4. 퍼징 테스트로 경로 검증 뚫어보기
5. Bash 3.2 호환성 지옥 — macOS가 2007년 Bash를 쓰는 이유

**트랙 C — AI 에이전트 (권장 시작점 🔥)**
1. **AGENTS.md 하나로 Claude/Cursor/Codex/Gemini 전부 관리하기**
2. **Claude Code가 안전장치를 지우지 못하게 만드는 법**
3. Claude 서브에이전트 실전: 읽기 전용 감사관 만들기
4. Claude Code 훅으로 자동 포맷팅 (토큰 절약)
5. 45KB AGENTS.md 분석 — 프로덕션 AI 컨텍스트 설계

### 제작 체크리스트
- 녹화 환경: macOS 필수 / iTerm2·Ghostty, 폰트 18pt+, 다크 테마
- **프라이버시**: 녹화 전 더미 유저 계정 생성 (파일명·앱 목록 노출 방지)
- 시연: 반드시 `--dry-run` 먼저 보여주고 권장
- 극적 효과: 새 계정에 쓰레기 파일 생성 → Before/After
- 설명란: 원작자 크레딧 + 레포 URL + 설치 명령어 / 챕터 마커

---

## 11. 수익화 아이디어 10선

### GPL v3 수익화 지도

| 모델 | GPL | 상표 | 종합 |
| --- | :---: | :---: | :---: |
| 유튜브/강의/블로그 | ✅ | 🟡 | ✅ 안전 |
| 컨설팅/교육 | ✅ | ✅ | ✅ 안전 |
| Mole 실행 SaaS (AGPL 아님) | ✅ | 🟡 | ✅ 안전 |
| 패턴 차용 재작성 | ✅ | ✅ | ✅ 안전 |
| MCP 래퍼(별도 작성) | 🟡 | 🟡 | ⚠️ 변호사 |
| 코드 포함 유료 제품 | ❌ | ❌ | ❌ 비추 |
| 클로즈드 소스 포크 | ❌ | ❌ | ❌ 금지 |

### 종합 평가 매트릭스

| # | 아이디어 | 수익성 | 난이도 | 법적 | 속도 | 순위 |
| --- | --- | :---: | :---: | :---: | :---: | :---: |
| 1 | 콘텐츠 + 교육 제국 | ⭐⭐⭐⭐ | 🟢 | ✅ | 즉시 | 🥇 |
| 6 | AGENTS.md 템플릿 / AI 거버넌스 | ⭐⭐⭐⭐ | 🟢 | ✅ | 1주 | 🥈 |
| 7 | AI 안전 감사 컨설팅 | ⭐⭐⭐⭐⭐ | 🟡 | ✅ | 3개월 | 🥉 |
| 2 | SafeOps 프레임워크 (SaaS) | ⭐⭐⭐⭐⭐ | 🔴 | ✅ | 6~12개월 | 4 |
| 3 | B2B Mac 함대 관리 SaaS | ⭐⭐⭐⭐ | 🟡 | ✅ | 3~6개월 | 5 |
| 5 | Mole MCP 서버 | ⭐ (직접) | 🟢 | ⚠️ | 2주 | 6 (명성) |
| 4 | Tauri+React GUI 앱 | ⭐⭐ | 🟡 | ❌ | 2개월 | 7 |
| 8 | 크로스플랫폼 포크 | ⭐⭐ | 🔴 | ⚠️ | — | 하위 |
| 9 | Mole Pro 플러그인 생태계 | ⭐⭐ | 🟡 | ❌ | — | 하위 |
| 10 | 안전성 테스트 SaaS (퍼징) | ⭐⭐⭐ | 🔴 | ✅ | — | 하위 |

### ① 콘텐츠 + 교육 제국 (1위)
4단 퍼널: 유튜브(유입) → 블로그/뉴스레터(리스트) → 유료 강의(7~10만원) → 기업 교육/컨설팅(300~1,000만원).
초기 투자 0원, 법적 리스크 0, 영상이 영구 자산, 다른 모델 전부의 마케팅 엔진.

### ② SafeOps 프레임워크 (최고 잠재력)
Mole 코드를 쓰지 않고 **설계 패턴만 차용해 새로 작성**.

| Mole 패턴 | 범용화 |
| --- | --- |
| `mole_delete` 단일 깔때기 | 모든 파괴적 작업의 단일 게이트 |
| `validate_path_for_deletion` | 플러그형 검증 체인 |
| `--dry-run` | 계획/실행 분리 (Terraform plan/apply) |
| `# SAFE:` + CI 감사 | 우회 시 명시 사유 + 자동 감사 |
| `operations.log` | 감사 로그 + 롤백 |
| 휴지통 경유 | 소프트 삭제 / 격리 영역 |
| 화이트리스트 | 보호 리소스 정책 |
| tri-state `2` = 거부 | **"모르면 거부" fail-closed** ⭐ |

수익: OSS 코어(MIT) → Cloud($29~199/월) → Enterprise($10k+/년).
타이밍: AI 에이전트가 프로덕션을 망가뜨리는 사고가 급증 중.

### ③ B2B Mac 함대 관리 SaaS
```
각 Mac: mo status --json (LaunchAgent, 1시간) → HTTPS POST
   → 수집 API (Laravel/Node) → PostgreSQL/TimescaleDB
   → React 대시보드: 건강점수 히트맵 / 회수가능공간 Top10 /
     배터리 교체 필요 기기 / 디스크 90% Slack 알림 / 승인 후 원격 청소
```
가격: Free 5대 / Team $3·대·월 / Business $5 + SSO / Enterprise 온프렘.
500대 = 월 $1,500~2,500 (ARR ~$25k).
리스크: Jamf·Kandji·Mosyle 경쟁 → "정리/최적화"로 좁게 포지셔닝. MVP는 Mole, 이후 자체 에이전트.

### ④~⑩ 요약
- ④ **Tauri+React GUI**: GPL 논쟁 + 원작자 유료앱과 직접 경쟁 → 수익화 대신 **포트폴리오·튜토리얼**로
- ⑤ **MCP 서버**: 직접 수익 적지만 명성 → "안전한 MCP 서버 설계" 전문가 포지션 → 컨설팅 연결
- ⑥ **AGENTS.md 템플릿 팩**: Gumroad $29~99부터. **가장 빠른 첫 매출**
- ⑦ **AI 안전 감사 컨설팅**: 건당 500~2,000만원. 단가 최고, 레퍼런스 필요
- ⑧ 크로스플랫폼 포크: GPL 상속으로 소스 공개 필수 → 후원 수준
- ⑨ Mole Pro 플러그인: 원작자와 직접 경쟁 → 비권장
- ⑩ 안전성 테스트 SaaS: 퍼징/PTY 패턴 서비스화

### 실행 로드맵

| 시점 | 할 일 | 예상 수익 |
| --- | --- | --- |
| 1주 | 영상 1개(AGENTS.md 멀티 AI 관리) + 블로그 1개 | — |
| 1개월 | 영상 4~6개, AGENTS.md 템플릿 팩 출시, 뉴스레터 | 첫 매출 |
| 3개월 | 구독 3~5천, 유료 강의 1개, MCP 서버 OSS 공개 | 30~80만원/월 |
| 6개월 | B2B MVP 또는 SafeOps OSS 시작 | 200~500만원/월 |
| 12개월 | SaaS 유료 고객 확보 → 반복 수익 | 500~1,500만원/월 |

### 최종 권고
- ❌ 하지 말 것: 코드 포크 유료 앱(GPL 위반 + 원작자 경쟁), 고객 0명 상태로 SaaS 선개발
- ✅ 할 것: 콘텐츠 → 디지털 상품(템플릿) → 시장 반응 확인 → 검증된 방향으로 제품화

> **핵심 인사이트**: 이 저장소의 진짜 가치는 "맥 청소 도구"가 아니라
> **"AI 에이전트를 안전하게 운영하는 방법론"** 이다. 많은 팀이 이 문제를 겪고 있고,
> 이 저장소는 그에 대한 살아있는 모범 사례다.

---

## 12. 개발 환경 참고 (이 저장소에서 작업할 때)

```bash
./scripts/check.sh --format                      # 셸 포맷 + 린트
MOLE_TEST_NO_AUTH=1 ./scripts/test.sh            # 전체 Bats
TERM=xterm-256color MOLE_TEST_NO_AUTH=1 ./scripts/test.sh   # 백그라운드 실행 시 TERM 필수
MOLE_TEST_NO_AUTH=1 bats tests/clean_core.bats   # 개별
make build                                       # Go 바이너리
go test ./...                                    # Go 테스트
make verify                                      # check + go test
```

주의사항:
- 테스트/CI 출력을 `tail`·`head` 로 파이프하지 말 것 (파이프의 종료 코드가 보고되어 빨간 실행이 녹색으로 읽힘)
- `cancelled` CI 실행은 통과가 아님 (`cancel-in-progress: true`)
- 전체 스위트 개수는 러너의 요약 라인을 읽을 것 (직접 센 개수는 부정확)
- 커밋에 AI 귀속(attribution) 트레일러 추가 금지 (프로젝트 규칙)

---

*이 문서는 Claude Code 세션의 분석 대화를 정리한 것입니다. 원작자: tw93 — https://github.com/tw93/mole*
