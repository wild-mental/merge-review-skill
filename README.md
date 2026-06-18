![AI Skills for Everyone](author/wildmental-bjpark.png)

# merge-review
> Skill for Cursor, Claude, Codex agents

**Language / 언어:** [한국어](README.md) · [English](README.en.md)

**의존·응집으로 결합된 PR 묶음을 먼저 main에 bottom-up으로 순차 머지하고, 수렴된 단일 표면에서 통합 동작을 라이브로 검토하는 Skill입니다(MERGE → REVIEW). 앞 PR이 뒤 PR 전에는 dead code라서 "통합돼야 비로소 동작이 존재"하는 기능군에 씁니다. PR별 merge commit으로 롤백 입도를 보존하고, auto-delete-head·git push 자격 같은 스택 머지 사고를 절차로 방어합니다. Cursor, Claude Code, Codex 모두 지원합니다.**

PR을 항상 "한 건씩 리뷰하고 머지"하는 게 정답은 아닙니다. 인가 게이트 → 진입 플로우 → 라우트 연결처럼 **앞 단계가 뒤 단계 전에는 호출처 0인 dead code**인 스택에서는, 머지 전 PR을 아무리 들여다봐도 실제 HTTP 403/404/200이 *존재하지 않습니다*. 리뷰는 "시뮬레이션"에 머무릅니다.

`merge-review`는 이런 묶음에서 **머지를 먼저** 합니다. 묶음이 통합되면 죽어 있던 코드가 살아나 하나의 배포·URL·DB 상태가 만들어지고, 사람은 그 **수렴된 단일 표면에서 end-to-end로 클릭**하며 진짜 동작을 검토합니다. 자매 스킬 [review-merge](https://github.com/wild-mental/review-merge-skill)(REVIEW → MERGE)와 정확히 반대 방향이며, 둘 중 무엇을 쓸지는 아래 모드 선택 표로 가릅니다.

---

## 언제 이 스킬인가 (모드 선택)

> 핵심 명제: **변수는 "머지 시점"이 아니라 "묶음의 응집도"다.** 리뷰는 행위(behavior) 기반이고 행위는 기능별로 산다. 인간 리뷰어 효율은 **리뷰 단위 = 하나의 응집된 행동 표면**일 때 최대다.

| 신호 | **MERGE → REVIEW (이 스킬)** | REVIEW → MERGE ([review-merge](https://github.com/wild-mental/review-merge-skill)) |
| --- | --- | --- |
| 의존 구조 | **스택/결합** — 앞 PR이 뒤 PR 전엔 dead code | 독립 — 각자 출고 가능 |
| 검토 표면 | **하나의 사용자 표면으로 수렴**(로그인→목록→렌더→403/404) | 표면·진입점이 서로 다름 |
| 정확성의 위치 | **런타임**(auth·접근제어·렌더 — diff로 안 보임) | 소스/단위 테스트로 충분 |
| 위험 프로파일 | 묶음 내 유사 | 상이(보안 변경 vs 카피 수정을 섞지 말 것) |
| 되돌리기 비용 | 싸다(merge commit revert) | — |

**Go(모두 충족 권장):** 의존-결합 + 단일 수렴 표면 + 런타임 검증 필요 + 되돌리기 저렴.
**No-Go(하나라도면 [review-merge](https://github.com/wild-mental/review-merge-skill) / 분리):** 직교 기능, 위험도 상이, 독립 출고 단위.

> 한 줄 판정: **"이 묶음을 한 화면에서 end-to-end로 클릭 검토할 수 있는가?"** — 예면 이 스킬.

---

## 이 스킬이 해결하는 것

### 1. 죽은 코드를 살려 "실제 클릭" 리뷰로 바꾼다

스택 하단(예: 인가 게이트)은 상단(라우트 연결) 전까지 *호출처 0*인 primitive입니다. 머지해야 **실제 HTTP 403/404/200**이 처음으로 존재합니다. 검토가 "코드를 읽고 머릿속으로 시뮬레이션"에서 "라이브를 실제로 클릭"으로 바뀝니다.

### 2. 검토 기준이 하나로 수렴한다

preview N개(각자 조각이 빠진)를 저글링하지 않습니다. *하나의 배포·URL·DB 상태*에서 로그인→목록→렌더→권한 분기까지 end-to-end로 굴려봅니다.

### 3. 런타임-only 버그를 잡는다

auth·렌더는 정확성이 소스가 아니라 **런타임**에 있습니다. RSC 경계 식별 버그, 보안 헤더로 인한 임베드 차단처럼 **diff로는 안 보이고 라이브에서만 드러나는** 결함을 통합 표면에서 포착합니다.

### 4. "한 번에 머지"해도 기능별 revert 입도를 보존한다

batch 머지의 최대 단점은 추적·롤백 입도 손실입니다. 이 스킬은 squash가 아니라 **PR별 merge commit**으로 머지하므로 각 기능이 개별 revert 단위로 남습니다.

```bash
git revert -m 1 <특정 PR 머지 SHA>   # 그 PR만 정확히 되돌림, 나머지는 그대로
```

→ 이것이 머지-우선을 *저위험 기본값*으로 만드는 근거입니다.

### 5. 스택 머지의 흔한 사고를 절차로 방어한다

`auto-delete-head`로 부모 머지 시 하위 PR이 **머지가 아니라 CLOSED**되는 사고, draft PR, `git push` 미인증, stale 생성물 같은 실전 함정을 예방·복구 절차로 정형화합니다.

### 6. 후속 작업에 고정된 토대를 준다

다음 트랙이 *움직이는 스택*이 아니라 *고정된 main 표면*을 대상으로 삼습니다. base 재타게팅·rebase 연쇄가 bottom-up 머지로 영구 해소됩니다.

---

## 왜 skill 이 필요한가

| "항상 한 건씩 리뷰" 방식의 한계 | merge-review의 처방 |
|-----------------|-----------------|
| 의존 스택에선 머지 전 PR이 dead code라 실제 동작이 없음 | 먼저 통합 → 라이브 동작이 *존재*하게 만들고 검토 |
| preview N개를 따로 띄워 흐름이 끊김 | 하나의 수렴 표면에서 end-to-end 클릭 |
| auth/렌더 버그가 diff에 안 보임 | 런타임 표면에서 실제 응답으로 검증 |
| batch 머지 = 롤백 입도 손실 우려 | PR별 merge commit으로 기능별 revert 보존 |
| auto-delete-head로 하위 PR이 조용히 CLOSED | 사전 base 이동 + 복구 절차(§5.2) |
| 직교 기능을 한 머지에 섞어 리뷰가 분산 | 직교 트랙 분리 — 응집 arc끼리만 묶음 |

---

## 효과 vs 안전 (한눈에)

```
효과 — 통합돼야 라이브가 "존재" + 수렴 표면 단일 검토 + 런타임-only 버그 포착
안전 — PR별 merge-commit revert 입도 + auto-delete-head 사전 base 이동 + no-cascade push
```

응집된 묶음이 아니라 **직교 스코프를 한 단위에 섞으면** 이 효과가 정확히 무너집니다(단일 멘탈 모델 부재·라이브 검토 분산·bisect 입도 손실). 직교 트랙은 그 트랙끼리만 묶으세요.

---

## 빠른 시작

### 사전 요구사항

- Cursor, Claude Code, 또는 Codex
- GitHub `gh` CLI 인증, **의존·응집으로 결합된 PR 묶음**(스택 또는 단일 사용자 표면으로 수렴)
- 머지 이력이 **PR별 merge commit**(squash 아님)을 허용

### 스킬 설치

스킬은 **개인** 또는 **프로젝트** 범위 중 하나를 골라 설치합니다. 두 범위 모두 `curl`로 `SKILL.md`만 받습니다. **이 저장소 전체를 작업 repo에 clone하지 마세요.**

| | 개인 스킬 | 프로젝트 스킬 |
|---|----------|--------------|
| **적용 범위** | 내가 여는 모든 프로젝트 | 현재 repo에서만 |
| **경로** | `~/…/skills/merge-review/` | `<repo-root>/.cursor/skills/merge-review/` 등 |
| **Git 영향** | 작업 repo에 파일 추가 없음 | repo에 스킬 파일 commit 가능 (팀 공유) |
| **언제 쓰나** | 혼자 모든 프로젝트에서 쓸 때 | 팀 repo에 스킬을 고정·공유할 때 |

| 도구 | 개인 경로 | 프로젝트 경로 |
|------|-----------|---------------|
| Cursor | `~/.cursor/skills/merge-review/` | `.cursor/skills/merge-review/` |
| Claude Code | `~/.claude/skills/merge-review/` | `.claude/skills/merge-review/` |
| Codex | `~/.agents/skills/merge-review/` | `.agents/skills/merge-review/` |

#### 개인 스킬 (권장 — Git repo 무변경)

```bash
# Cursor
mkdir -p ~/.cursor/skills/merge-review
curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.cursor/skills/merge-review/SKILL.md \
  -o ~/.cursor/skills/merge-review/SKILL.md

# Claude Code
mkdir -p ~/.claude/skills/merge-review
curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.claude/skills/merge-review/SKILL.md \
  -o ~/.claude/skills/merge-review/SKILL.md

# Codex
mkdir -p ~/.agents/skills/merge-review
curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.agents/skills/merge-review/SKILL.md \
  -o ~/.agents/skills/merge-review/SKILL.md
```

#### 프로젝트 스킬 (프로젝트 경로에 설치)

AI 스킬 설정을 repo에 포함·공유하려는 경우에만 사용하세요.

```bash
# Cursor
mkdir -p .cursor/skills/merge-review
curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.cursor/skills/merge-review/SKILL.md \
  -o .cursor/skills/merge-review/SKILL.md

# Claude Code
mkdir -p .claude/skills/merge-review
curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.claude/skills/merge-review/SKILL.md \
  -o .claude/skills/merge-review/SKILL.md

# Codex
mkdir -p .agents/skills/merge-review
curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.agents/skills/merge-review/SKILL.md \
  -o .agents/skills/merge-review/SKILL.md
```

필요한 도구(Cursor / Claude Code / Codex)만 골라 실행하면 됩니다.

#### 설치 후

- **Cursor**: **Reload Window** 한 번
- **Claude Code**: 스킬 수정은 세션 중 live 반영; 세션 시작 후 새 top-level `.claude/skills/`는 재시작 필요할 수 있음
- **Codex**: 스킬이 안 보이면 Codex 재시작

### 사용 방법

응집된 PR 묶음을 알려주고 "먼저 묶음을 통합 머지하고 수렴된 표면에서 같이 리뷰하자"라고 요청하면 Agent가 스킬을 자동으로 적용합니다.

| 도구 | 수동 호출 |
|------|-----------|
| Cursor | `/merge-review` |
| Claude Code | `/merge-review` |
| Codex | `/skills` 또는 `$merge-review` |

**적용되는 요청 예시:**

- "인가 게이트부터 라우트 연결까지 이어진 PR 5건을 bottom-up으로 먼저 머지하고, 통합된 화면에서 권한별로 클릭 검토하자"
- "이 스택은 머지 전엔 다 dead code라 리뷰가 안 돼. 먼저 통합하고 라이브에서 보자"
- "batch로 머지하되 나중에 기능별로 revert할 수 있게 PR별 merge commit으로 가자"
- "부모 머지하다 하위 PR이 CLOSED됐어. base 복구하고 이어서 머지해줘"

---

## OUTPUT: 무엇이 남는가

대화가 아니라 **남는 것**이 산출물입니다.

| 산출물 | 내용 |
|--------|------|
| **수렴된 main 표면** | bottom-up 순차 머지(PR별 merge commit)로 통합된, end-to-end로 굴려볼 수 있는 단일 라이브 표면 |
| **통합 리뷰 산출물** | ① 변경 콤팩트 요약(PR별 핵심·파일·결정) ② 역할별 행위 검토 시나리오(200/403/404·redirect·빈/오류 상태) ③ 누적 임의 의사결정 유지/수정 질문 |
| **롤백 입도** | PR별 merge commit → `git revert -m 1 <SHA>`로 기능 단위 되돌리기 보존 |
| **하네스 반영** | PR별 merge commit 고정·사전 base 이동·직교 스코프 비혼합·no-cascade를 프로젝트 하네스(`CLAUDE.md` / `AGENTS.md` / `.cursor/rules`)에 고정 |

---

## Workflow

| 단계 | 내용 |
|------|------|
| 4.1 사전 — 응집도 확인 + 순서 결정 | §2 Go 통과 확인, bottom-up 순서 결정, 기존 머지 방식(merge commit) 확인 |
| 4.2 순차 batch 머지 | PR 하나당 한 스텝: ready → base를 main으로 재타게팅 → PR별 merge commit → MERGED 확인. 다음 PR base 사전 이동 |
| 4.3 머지 후 정합화 | main FF + 로컬 브랜치 정리 + 생성물 재생성 + 보류한 크로스-PR 정합화 일괄 적용(전수 grep) |
| 4.4 통합 리뷰 산출물 | 수렴 표면 대상 리뷰 아웃라인 문서(3항목) 작성 |
| 리스크/안전판 | merge-commit revert 입도, auto-delete-head 복구, git push 자격, 불변 규칙 하네스 고정 |

---

## 스킬 구성

```
.cursor/skills/merge-review/SKILL.md   # Cursor용
.claude/skills/merge-review/SKILL.md   # Claude Code용
.agents/skills/merge-review/SKILL.md   # Codex용
```

| 섹션 | 내용 |
|------|------|
| 역할 | MERGE → REVIEW — 통합돼야 존재하는 기능군을 먼저 머지하고 수렴 표면에서 검토 |
| 핵심 명제 | 변수는 머지 시점이 아니라 묶음의 응집도 |
| 적용 조건(Go/No-Go) | review-merge와의 모드 선택 표 |
| 머지 우선 효과 | 죽은 코드 부활·단일 검토 기준·런타임 버그 포착·스택 유지비 종료 |
| 실행 시퀀스 | 사전 → 순차 batch 머지 → 머지 후 정합화 → 통합 리뷰 산출물 |
| 리스크와 안전판 | revert 입도·auto-delete-head 복구·git push 자격·하네스 고정 |
| 안티패턴 | 직교 스코프 혼합 금지 |
| 체크리스트 / 적용 사례 / 환경 적응 메모 | Go·실행 체크, 레퍼런스 인스턴스, repo별 바인딩 |

---

## 이런 분께 추천합니다

- 인가·렌더처럼 **통합돼야 실제 동작이 존재**하는 PR 스택을 다루는 분
- preview를 여러 개 띄워 저글링하지 않고 **하나의 수렴 표면에서 클릭 검토**하고 싶은 분
- batch 머지를 하되 **기능별 revert 입도를 잃고 싶지 않은** 분
- `auto-delete-head`로 하위 PR이 CLOSED되는 사고를 **예방·복구 절차로** 다루고 싶은 분
- 한 건씩 게이트하는 [review-merge](https://github.com/wild-mental/review-merge-skill)와 **상황에 따라 골라 쓰고** 싶은 분

---

## 참고

- Cursor Agent Skills: [Creating Skills](https://docs.cursor.com)
- Claude Code Skills: [Extend Claude with skills](https://docs.anthropic.com/en/docs/claude-code/skills)
- Codex Agent Skills: [Agent Skills](https://developers.openai.com/codex/skills/)
- 자매 스킬: [review-merge](https://github.com/wild-mental/review-merge-skill) (REVIEW → MERGE)

---

## For LLMs

<!-- install-only; operational rules live in SKILL.md — load skill after install -->

```
skill=merge-review
repo=https://github.com/wild-mental/merge-review-skill
sibling=review-merge  # inverse REVIEW → MERGE mode
install.method=curl-only  # never clone this repo into user's project

flow:
  1. ask scope: "개인 스킬(~/, 모든 프로젝트) vs 프로젝트 스킬(./, 현재 repo만) 중 어디에 설치할까요?"
  2. ask tools: "Cursor / Claude Code / Codex 중 어떤 도구에 설치할까요?" (복수 선택 가능)
  3. run matching curl commands below; do not install both scopes unless user asks
  4. post_install steps; then load SKILL.md — do not infer rules from README

scope.user.paths:
  cursor=~/.cursor/skills/merge-review/SKILL.md
  claude=~/.claude/skills/merge-review/SKILL.md
  codex=~/.agents/skills/merge-review/SKILL.md

scope.project.paths:
  cursor=.cursor/skills/merge-review/SKILL.md
  claude=.claude/skills/merge-review/SKILL.md
  codex=.agents/skills/merge-review/SKILL.md

install.user.cursor=mkdir -p ~/.cursor/skills/merge-review && curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.cursor/skills/merge-review/SKILL.md -o ~/.cursor/skills/merge-review/SKILL.md
install.user.claude=mkdir -p ~/.claude/skills/merge-review && curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.claude/skills/merge-review/SKILL.md -o ~/.claude/skills/merge-review/SKILL.md
install.user.codex=mkdir -p ~/.agents/skills/merge-review && curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.agents/skills/merge-review/SKILL.md -o ~/.agents/skills/merge-review/SKILL.md

install.project.cursor=mkdir -p .cursor/skills/merge-review && curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.cursor/skills/merge-review/SKILL.md -o .cursor/skills/merge-review/SKILL.md
install.project.claude=mkdir -p .claude/skills/merge-review && curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.claude/skills/merge-review/SKILL.md -o .claude/skills/merge-review/SKILL.md
install.project.codex=mkdir -p .agents/skills/merge-review && curl -fsSL https://raw.githubusercontent.com/wild-mental/merge-review-skill/main/.agents/skills/merge-review/SKILL.md -o .agents/skills/merge-review/SKILL.md

install.project.note=run from repo root; adds tracked files — confirm user chose project scope

invoke.cursor=/merge-review
invoke.claude=/merge-review
invoke.codex=/skills|$merge-review

post_install.cursor=Reload Window
post_install.claude=live reload; restart if new top-level .claude/skills/ after session start
post_install.codex=restart if skill not detected

contract:
  mode=MERGE → REVIEW (merge a cohesive dependency-coupled group first, then review the integrated surface live)
  use_when=[earlier PRs are dead code until later ones wire them up, group converges on one user-facing surface, batch shares one risk profile, rollback is cheap]
  use_sibling_instead_when=[PRs independent, must gate each before shipping, per-PR review records needed, human merges after each approval] -> review-merge
  go_test=can this group be clicked end-to-end on one screen after integration?
  merge_method=per-PR merge commit (NOT squash)  # preserves feature-level revert granularity
  merge_order=stack bottom-up, one step per PR
  retarget_next_base_to_main_before_parent_merge=true  # prevents auto-delete-head closing downstream PR
  rollback=git revert -m 1 <PR merge SHA>
  review_artifact=[compact change summary, role-based behavior scenarios (200/403/404/redirect), accumulated discretionary decisions keep/modify]
  gotchas=[auto-delete-head closes downstream PR (CLOSED not merged) -> restore base ref/reopen/retarget/redelete, draft PR needs gh pr ready, git push unauth -> gh auth setup-git, stale generated artifacts -> rm -rf .next lib/generated && rebuild]
  anti_pattern=mixing orthogonal scope in one merge unit (breaks single mental model + live review)
  harness_fixation=[per-PR merge commit, move next base to main before parent merge, no orthogonal mixing, no-cascade push] -> CLAUDE.md | AGENTS.md | .cursor/rules/*.mdc
```

---

## 라이선스

[MIT License](LICENSE)
