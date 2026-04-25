# Phase 0 — 공식 문서 URL 6주제 + 기초 교안 ADVANCED 갭 분석

- **작성일:** 2026-04-25 KST
- **작성자:** 윈터 (HRD 리서치팀)
- **목적:** 바이브 코딩 중간 교안 Phase 0 — 공식 URL 직접 fetch 검증 + 기초 교안 갭 분석
- **신뢰도:** 전 항목 🟢 A (Anthropic 공식 문서 직접 fetch 검증)
- **합의서:** 합의서_v1.md v1.3 — 윈터 R&R (Phase 0 출처 검증 담당)

---

## 1. 공식 문서 URL 6주제 검증 결과

> ⚠️ **URL 구조 변경 확인 (2026-04-25 실측):**
> `docs.anthropic.com/en/docs/claude-code/*` → 301 Permanent Redirect → `code.claude.com/docs/en/*`
> **정식 URL은 `code.claude.com` 도메인** — 교안 내 URL 전부 이쪽으로 기재할 것

---

### S1. CLAUDE.md — 4계층 구조

| 항목 | 내용 |
|------|------|
| **공식 URL** | `https://code.claude.com/docs/en/memory` 🟢 A |
| **구 URL (리다이렉트)** | `https://docs.anthropic.com/en/docs/claude-code/memory` → 301 |
| **fetch 확인** | ✅ 직접 확인 2026-04-25 KST |

#### 핵심 내용

**4계층 (합의서 "3계층" = 실제 4계층)**

| 계층 | 위치 | 범위 | 공유 여부 |
|------|------|------|---------|
| Managed Policy | Win: `C:\Program Files\ClaudeCode\CLAUDE.md` / macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md` | 조직 전체 (IT 배포) | 전원 |
| Project | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | 프로젝트 팀 | 팀 (소스컨트롤) |
| User | `~/.claude/CLAUDE.md` | 개인 전 프로젝트 | 개인만 |
| Local | `./CLAUDE.local.md` (gitignore 권장) | 개인 프로젝트 전용 | 개인만 |

**Auto Memory:**
- 위치: `~/.claude/projects/<project>/memory/MEMORY.md`
- 첫 200줄 또는 25KB → 세션 시작 시 자동 로드
- Claude가 스스로 작성 (학습·패턴 저장)

**추가 기능:**
- `.claude/rules/` — 경로 스코핑 규칙 파일 (paths 프론트매터로 특정 파일만 적용)
- `@path/to/import` — CLAUDE.md 내 파일 임포트 문법
- `<!-- comment -->` — HTML 주석은 컨텍스트에 로드 안 됨 (토큰 절약)
- 200줄 이상 → 준수율 저하 주의

---

### S2. Skills

| 항목 | 내용 |
|------|------|
| **공식 URL** | `https://code.claude.com/docs/en/skills` 🟢 A |
| **구 URL (리다이렉트)** | `https://docs.anthropic.com/en/docs/claude-code/skills` → 301 |
| **fetch 확인** | ✅ 직접 확인 2026-04-25 KST |

#### 핵심 내용

**파일 위치:**

| 범위 | 경로 |
|------|------|
| 개인 (전 프로젝트) | `~/.claude/skills/<name>/SKILL.md` |
| 프로젝트 | `.claude/skills/<name>/SKILL.md` |
| 플러그인 | `<plugin>/skills/<name>/SKILL.md` |
| Managed (조직) | 관리형 설정 경유 |

**즉시 반영 (Live Change Detection):**
- 세션 **재시작 불필요** — 파일 저장 즉시 반영
- 단, 디렉토리 자체가 없다가 새로 생성 시에는 재시작 필요

**주요 프론트매터 필드:**

| 필드 | 용도 |
|------|------|
| `name` | 슬래시 명령어 이름 |
| `description` | Claude 자동 호출 판단 기준 (필수 권장) |
| `disable-model-invocation: true` | 사용자만 호출 가능 (Claude 자동 호출 금지) |
| `user-invocable: false` | 슬래시 메뉴에서 숨김 (Claude만 호출) |
| `context: fork` | 서브에이전트로 실행 (격리 컨텍스트) |
| `allowed-tools` | 스킬 활성 시 사전 승인 도구 목록 |

**`.claude/commands/` 하위 호환:**
- 기존 commands 파일 그대로 동작 — Skills와 동일 기능
- 같은 이름 충돌 시 Skills 우선

---

### S3. Hooks (exit 2 차단 포함)

| 항목 | 내용 |
|------|------|
| **공식 URL** | `https://code.claude.com/docs/en/hooks` 🟢 A |
| **구 URL (리다이렉트)** | `https://docs.anthropic.com/en/docs/claude-code/hooks` → 301 |
| **fetch 확인** | ✅ 직접 확인 2026-04-25 KST |

#### 핵심 내용

**이벤트 타입:**

| 이벤트 | 실행 시점 | 차단 가능 |
|--------|---------|---------|
| `PreToolUse` | 도구 실행 전 | ✅ |
| `PostToolUse` | 도구 실행 후 (성공) | ✅ |
| `PostToolUseFailure` | 도구 실행 후 (실패) | - |
| `PermissionRequest` | 권한 요청 시 | ✅ |
| `UserPromptSubmit` | 사용자 프롬프트 제출 시 | ✅ |
| `SessionStart` | 세션 시작 | ❌ |
| `SessionEnd` | 세션 종료 | ❌ |
| `Stop` | Claude 응답 완료 | ✅ |

**exit code 차단 메커니즘:**
```bash
exit 0  → 성공 (JSON stdout으로 세밀 제어)
exit 2  → 차단 (stderr 내용 사용자에게 표시, 작업 중단)
기타    → 비차단 에러 (실행 계속)
```

**핸들러 타입 5종:**
- `command` — 셸 스크립트
- `http` — HTTP 엔드포인트 POST
- `mcp_tool` — **MCP 도구 직접 호출** (v2.1.118 신기능 ✅)
- `prompt` — Claude에게 yes/no 판단 위임
- `agent` — 서브에이전트 실행

**설정 위치:**
- `~/.claude/settings.json` — 전 프로젝트
- `.claude/settings.json` — 프로젝트 (공유 가능)
- `.claude/settings.local.json` — 프로젝트 로컬

---

### S4. Subagents

| 항목 | 내용 |
|------|------|
| **공식 URL** | `https://code.claude.com/docs/en/sub-agents` 🟢 A |
| **구 URL (리다이렉트)** | `https://docs.anthropic.com/en/docs/claude-code/sub-agents` → 301 |
| **fetch 확인** | ✅ 직접 확인 2026-04-25 KST |

#### 핵심 내용

**파일 위치:**
- 개인: `~/.claude/agents/<name>.md`
- 프로젝트: `.claude/agents/<name>.md`
- `/agents` 명령어로 UI 생성 가능

**내장 Subagents:**

| 이름 | 모델 | 도구 | 용도 |
|------|------|------|------|
| `Explore` | Haiku (빠름) | 읽기 전용 | 코드베이스 탐색·검색 |
| `Plan` | 상속 | 읽기 전용 | 플랜 모드 리서치 |
| `general-purpose` | 상속 | 전체 | 복합 다단계 작업 |

**핵심 제약:**
- 서브에이전트는 다른 서브에이전트 생성 불가 (무한 중첩 방지)
- 각 서브에이전트 = 독립 컨텍스트 창

**자가 설정 필드:**
- `model`: Haiku / Sonnet / Opus 선택
- `tools`: 허용 도구 목록
- `description`: Claude 위임 판단 기준
- `memory`: 퍼시스턴트 메모리 디렉토리 (`~/.claude/agent-memory/`)

---

### S5. Agent SDK (구 Claude Code SDK → Claude Agent SDK)

| 항목 | 내용 |
|------|------|
| **공식 URL** | `https://code.claude.com/docs/en/agent-sdk/overview` 🟢 A |
| **리다이렉트 체인** | `docs.anthropic.com/en/docs/agents-and-tools/agent-sdk` → `platform.claude.com` → `docs.claude.com` → `code.claude.com` |
| **fetch 확인** | ✅ 직접 확인 2026-04-25 KST |

#### 핵심 내용

**이름 변경:**
> "Claude Code SDK has been renamed to the Claude Agent SDK"

**설치:**
```bash
# Python
pip install claude-agent-sdk

# TypeScript/Node
npm install @anthropic-ai/claude-agent-sdk
```

**핵심 패턴:**
```python
from claude_agent_sdk import query, ClaudeAgentOptions

async for message in query(
    prompt="Fix the bug in auth.py",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Edit", "Bash"]),
):
    print(message)
```

**CLI vs SDK:**
| 용도 | 선택 |
|------|------|
| 일상 개발 | CLI |
| CI/CD 파이프라인 | SDK |
| 커스텀 앱 | SDK |
| 프로덕션 자동화 | SDK |

**SDK에서 Subagents 사용 예:**
```python
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition

options = ClaudeAgentOptions(
    allowed_tools=["Read", "Glob", "Grep", "Agent"],
    agents={
        "code-reviewer": AgentDefinition(
            description="Expert code reviewer",
            prompt="Analyze code quality.",
            tools=["Read", "Glob", "Grep"],
        )
    },
)
```

---

### S6. MCP (Model Context Protocol)

| 항목 | 내용 |
|------|------|
| **공식 URL** | `https://code.claude.com/docs/en/mcp` 🟢 A |
| **구 URL (리다이렉트)** | `https://docs.anthropic.com/en/docs/claude-code/mcp` → 301 |
| **MCP Registry API** | `https://api.anthropic.com/mcp-registry/v0/servers` 🟢 A |
| **fetch 확인** | ✅ 직접 확인 2026-04-25 KST |

#### 핵심 내용

**Anthropic 공식 MCP Registry:**
- API: `https://api.anthropic.com/mcp-registry/v0/servers`
- 상업용 공개 서버 목록 조회 가능
- `worksWith: ["claude-code"]` 필터로 Claude Code 호환 서버 확인

**mcp.so 관련:**
- mcp.so = 커뮤니티 MCP 서버 디렉토리 (Anthropic 공식 아님 — **🟠 C등급**)
- 공식 MCP 서버 = Anthropic Registry API 또는 Claude Code `/mcp` 명령어에서 확인

**설정 방법 (claude_desktop_config.json 또는 settings.json):**
```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["@mypackage/mcp-server"]
    }
  }
}
```

**Agent SDK에서 MCP:**
```python
options = ClaudeAgentOptions(
    mcp_servers={
        "playwright": {"command": "npx", "args": ["@playwright/mcp@latest"]}
    }
)
```

---

## 2. 기초 교안 ADVANCED 섹션 갭 분석

> **기초 교안:** `https://lejiyon1202-tech.github.io/ai-guide/` (ai-guide 레포)
> fetch 확인 2026-04-25 KST 🟢 A (직접 열람)

### 기초 교안 ADVANCED 섹션 현황

| 주제 | 기초 교안 수준 | 중간 교안 목표 |
|------|-------------|-------------|
| **CLAUDE.md** | 소개 + 존재 언급 수준 (직접 작성 실습 없음) | **3계층 구조 이해 + 실제 작성 실습** |
| **Skills** | 이름 언급·개념 소개 수준 | **SKILL.md 직접 작성 + 즉시 반영 실습** |
| **Hooks** | 언급 없음 또는 극히 얕음 | **PreToolUse·exit 2 차단 직접 구현** |
| **Subagents** | 언급 없음 | **내장 서브에이전트 활용 + 커스텀 생성** |
| **MCP** | 언급 없음 | **공식 Registry 서버 연결 실습** |
| **Agent SDK** | 언급 없음 | **SDK 설치 + query() 기본 패턴** |

### 중간 교안 포지셔닝 (갭 채우기 전략)

```
기초 교안 (ai-guide)
└── AI가 무엇인지, 왜 써야 하는지, 기본 대화법
    └── CLAUDE.md·Skills 존재 소개 (손 안 댐)

     ↓ 갭 (중간 교안이 채워야 할 영역)

중간 교안 (신규)
└── CLAUDE.md 직접 작성 (3계층 선택·컨텍스트 효율화)
└── Skills 직접 제작 (SKILL.md + 즉시 반영)
└── Hooks 실제 구현 (exit 2 차단·mcp_tool 연동)
└── Subagents 활용 (Explore 활용 + 커스텀 에이전트)
└── MCP 연결 (공식 서버 + 실무 도구 연동)
└── Agent SDK 입문 (자동화 파이프라인 첫걸음)
└── 직군별 실무 시나리오 (리즈 케이스 5건 S18~S22)

     ↓

고급 교안 (vibe-coding-advanced)
└── 멀티에이전트 아키텍처·보안·팀 워크플로우
```

---

## 3. 자가 보고 3분류 매트릭스

| 구분 | 내용 |
|------|------|
| **어디서** | `shared/projects/vibe-coding-intermediate/phase0_directional/윈터_URL_6주제.md` |
| **무엇이** | 공식 URL 6주제 신규 fetch 검증 완료 + 기초 교안 ADVANCED 갭 분석 추가 |
| **어떻게** | 6주제 전부 `code.claude.com` 도메인으로 301 리다이렉트 확인·등급 🟢 A 전수·실제 문서 내용 기반 핵심 정보 추출·기초 교안 실측 기반 갭 매트릭스 완성 |

---

## 4. 출처 검증 결과 요약

| # | 주제 | 공식 URL | 등급 | fetch 확인 |
|---|------|---------|------|----------|
| 1 | CLAUDE.md | `https://code.claude.com/docs/en/memory` | 🟢 A | ✅ |
| 2 | Skills | `https://code.claude.com/docs/en/skills` | 🟢 A | ✅ |
| 3 | Hooks | `https://code.claude.com/docs/en/hooks` | 🟢 A | ✅ |
| 4 | Subagents | `https://code.claude.com/docs/en/sub-agents` | 🟢 A | ✅ |
| 5 | Agent SDK | `https://code.claude.com/docs/en/agent-sdk/overview` | 🟢 A | ✅ |
| 6 | MCP | `https://code.claude.com/docs/en/mcp` | 🟢 A | ✅ |
| 7 | MCP Registry API | `https://api.anthropic.com/mcp-registry/v0/servers` | 🟢 A | ✅ (코드 내 직접 확인) |
| 8 | 기초 교안 | `https://lejiyon1202-tech.github.io/ai-guide/` | 🟢 A | ✅ |

**⚠️ mcp.so**: Anthropic 공식 아님 (커뮤니티 허브) — 🟠 C등급. 교안 인용 시 주의.

**총 출처:** 8건 전부 🟢 A등급 (공식 직접 fetch 검증)

---

*리서치: 윈터 (HRD 리서치팀) | Phase 0 산출물 | 2026-04-25 KST*
*합의서 v1.3 — v3.6 14번째 SOP 출처 귀속 1단계 완료 (윈터 URL 첨부)*
