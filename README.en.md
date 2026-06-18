![AI Skills for Everyone](author/wildmental-bjpark.png)

# merge-review
> Skill for Cursor, Claude, Codex agents

**Language / 언어:** [한국어](README.md) · [English](README.en.md)

**A Skill that first merges a dependency- and cohesion-coupled group of PRs into main bottom-up, sequentially, and then reviews the integrated behavior live on the converged single surface (MERGE → REVIEW). Use it for feature groups where earlier PRs are dead code before the later ones, so that "behavior only comes into existence once integrated." It preserves rollback granularity via a per-PR merge commit, and procedurally defends against stack-merge accidents such as auto-delete-head and missing git push credentials. Supports Cursor, Claude Code, and Codex alike.**

Reviewing and merging PRs "one at a time, always" is not the only right answer. In a stack like authorization gate → entry flow → route wiring, where **the earlier stage is dead code with zero call sites before the later stage**, no matter how hard you stare at a pre-merge PR, an actual HTTP 403/404/200 *does not exist*. The review stays a "simulation."

`merge-review` **merges first** in such groups. Once the group is integrated, the dead code comes alive into a single deployment·URL·DB state, and a human **clicks through end-to-end on that converged single surface** to review the real behavior. It is the exact opposite direction of the sister skill [review-merge](https://github.com/wild-mental/review-merge-skill) (REVIEW → MERGE), and the mode-selection table below decides which of the two to use.

---

## When this skill (mode selection)

> Core thesis: **The variable is not the "merge timing" but the "cohesion of the group."** Review is behavior-based, and behavior lives per feature. Human-reviewer efficiency is maximized when the **review unit = one cohesive behavior surface**.

| Signal | **MERGE → REVIEW (this skill)** | REVIEW → MERGE ([review-merge](https://github.com/wild-mental/review-merge-skill)) |
| --- | --- | --- |
| Dependency structure | **Stack/coupled** — earlier PR is dead code before the later one | Independent — each can ship on its own |
| Review surface | **Converges into one user surface** (login→list→render→403/404) | Surfaces·entry points differ from each other |
| Where correctness lives | **Runtime** (auth·access control·render — invisible in diff) | Source/unit tests are enough |
| Risk profile | Similar within the group | Differing (don't mix security changes with copy edits) |
| Cost of reverting | Cheap (merge commit revert) | — |

**Go (recommended when all are met):** dependency-coupled + single converged surface + runtime verification needed + cheap to revert.
**No-Go (if even one applies, use [review-merge](https://github.com/wild-mental/review-merge-skill) / separate):** orthogonal features, differing risk levels, independent shipping units.

> One-line verdict: **"Can this group be clicked through end-to-end for review on one screen?"** — if yes, this skill.

---

## What this skill solves

### 1. Revives dead code and turns it into "actual click" review

The bottom of the stack (e.g., the authorization gate) is a primitive with *zero call sites* until the top (route wiring). Only after merging does an **actual HTTP 403/404/200** first exist. Review shifts from "read the code and simulate it in your head" to "actually click the live thing."

### 2. The review criterion converges into one

You don't juggle N previews (each missing a piece). On *a single deployment·URL·DB state*, you run end-to-end from login→list→render→permission branching.

### 3. Catches runtime-only bugs

For auth·render, correctness lives in the **runtime**, not the source. You catch, on the integration surface, defects that are **invisible in the diff and only surface live**, such as RSC boundary identification bugs or embed blocking caused by security headers.

### 4. Preserves per-feature revert granularity even when "merging all at once"

The biggest downside of batch merging is loss of traceability·rollback granularity. This skill merges via a **per-PR merge commit**, not squash, so each feature remains an individual revert unit.

```bash
git revert -m 1 <merge SHA of a specific PR>   # reverts exactly that PR, leaving the rest untouched
```

→ This is the rationale that makes merge-first a *low-risk default*.

### 5. Procedurally defends against common stack-merge accidents

It formalizes, into prevention·recovery procedures, real-world pitfalls such as: the accident where `auto-delete-head` makes a downstream PR get **CLOSED rather than merged** when the parent is merged, draft PRs, an unauthenticated `git push`, and stale artifacts.

### 6. Gives follow-up work a fixed foundation

The next track targets a *fixed main surface* rather than a *moving stack*. Base re-targeting·rebase chains are permanently resolved by bottom-up merging.

---

## Why a skill is needed

| Limits of the "always review one at a time" approach | merge-review's prescription |
|-----------------|-----------------|
| In a dependency stack, pre-merge PRs are dead code, so there is no real behavior | Integrate first → make live behavior *exist*, then review |
| Spinning up N previews separately breaks the flow | End-to-end clicks on one converged surface |
| Auth/render bugs are invisible in the diff | Verify with real responses on the runtime surface |
| Batch merge = concern over loss of rollback granularity | Preserve per-feature revert with a per-PR merge commit |
| auto-delete-head silently CLOSES a downstream PR | Move the base beforehand + recovery procedure (§5.2) |
| Mixing orthogonal features into one merge scatters the review | Separate orthogonal tracks — group only cohesive arcs |

---

## Effect vs safety (at a glance)

```
Effect — live "exists" only once integrated + single review of the converged surface + catches runtime-only bugs
Safety — per-PR merge-commit revert granularity + auto-delete-head pre-base move + no-cascade push
```

If you **mix orthogonal scope into one unit** rather than a cohesive group, this effect collapses exactly (absence of a single mental model·scattered live review·loss of bisect granularity). Group orthogonal tracks only with their own kind.

---

## Quick start

### Prerequisites

- Cursor, Claude Code, or Codex
- GitHub `gh` CLI authentication, **a dependency- and cohesion-coupled group of PRs** (a stack, or converging onto a single user surface)
- A merge history that allows a **per-PR merge commit** (not squash)

### Installing the skill

You install the skill by choosing one of two scopes: **personal** or **project**. Both scopes fetch only `SKILL.md` via `curl`. **Do not clone this entire repository into your working repo.**

| | Personal skill | Project skill |
|---|----------|--------------|
| **Scope** | Every project I open | Only the current repo |
| **Path** | `~/…/skills/merge-review/` | `<repo-root>/.cursor/skills/merge-review/`, etc. |
| **Git impact** | No files added to the working repo | Skill files can be committed to the repo (team sharing) |
| **When to use** | When using it solo across all projects | When pinning·sharing the skill in a team repo |

| Tool | Personal path | Project path |
|------|-----------|---------------|
| Cursor | `~/.cursor/skills/merge-review/` | `.cursor/skills/merge-review/` |
| Claude Code | `~/.claude/skills/merge-review/` | `.claude/skills/merge-review/` |
| Codex | `~/.agents/skills/merge-review/` | `.agents/skills/merge-review/` |

#### Personal skill (recommended — no Git repo changes)

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

#### Project skill (install into the project path)

Use this only when you want to include·share the AI skill configuration in the repo.

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

Just pick and run only the tool(s) you need (Cursor / Claude Code / Codex).

#### After installation

- **Cursor**: **Reload Window** once
- **Claude Code**: skill edits apply live during a session; a new top-level `.claude/skills/` added after the session started may require a restart
- **Codex**: if the skill doesn't show up, restart Codex

### How to use

Tell the agent about the cohesive PR group and request, "let's integrate-merge the group first and review together on the converged surface," and the agent applies the skill automatically.

| Tool | Manual invocation |
|------|-----------|
| Cursor | `/merge-review` |
| Claude Code | `/merge-review` |
| Codex | `/skills` or `$merge-review` |

**Examples of requests it applies to:**

- "Merge first, bottom-up, the 5 PRs running from authorization gate to route wiring, and click through the integrated screen per permission for review"
- "This stack can't be reviewed because everything is dead code before merge. Integrate first and let's look at it live"
- "Merge as a batch, but let's go with a per-PR merge commit so we can revert per feature later"
- "A downstream PR got CLOSED while merging the parent. Restore the base and continue the merge for me"

---

## OUTPUT: what remains

The deliverable is **what remains**, not the conversation.

| Deliverable | Content |
|--------|------|
| **Converged main surface** | A single live surface, integrated by bottom-up sequential merge (per-PR merge commit), that can be run end-to-end |
| **Integration review artifact** | ① compact change summary (per-PR essentials·files·decisions) ② per-role behavior review scenarios (200/403/404·redirect·empty/error states) ③ accumulated discretionary-decision keep/modify questions |
| **Rollback granularity** | Per-PR merge commit → preserves per-feature reverting via `git revert -m 1 <SHA>` |
| **Harness fixation** | Pin per-PR merge commit·pre-base move·no orthogonal scope mixing·no-cascade into the project harness (`CLAUDE.md` / `AGENTS.md` / `.cursor/rules`) |

---

## Workflow

| Step | Content |
|------|------|
| 4.1 Pre — confirm cohesion + decide order | Confirm §2 Go passes, decide bottom-up order, confirm the existing merge method (merge commit) |
| 4.2 Sequential batch merge | One step per PR: ready → retarget base to main → per-PR merge commit → confirm MERGED. Move the next PR's base beforehand |
| 4.3 Post-merge reconciliation | main FF + local branch cleanup + artifact regeneration + apply the held-back cross-PR reconciliation in bulk (full grep) |
| 4.4 Integration review artifact | Write a review-outline document (3 items) targeting the converged surface |
| Risks/safeguards | merge-commit revert granularity, auto-delete-head recovery, git push credentials, pin invariant rules in the harness |

---

## Skill structure

```
.cursor/skills/merge-review/SKILL.md   # for Cursor
.claude/skills/merge-review/SKILL.md   # for Claude Code
.agents/skills/merge-review/SKILL.md   # for Codex
```

| Section | Content |
|------|------|
| Role | MERGE → REVIEW — merge first the feature group that only exists once integrated, and review on the converged surface |
| Core thesis | The variable is not merge timing but the cohesion of the group |
| Applicability conditions (Go/No-Go) | The mode-selection table vs review-merge |
| Merge-first effect | Reviving dead code·single review criterion·catching runtime bugs·ending stack maintenance cost |
| Execution sequence | Pre → sequential batch merge → post-merge reconciliation → integration review artifact |
| Risks and safeguards | revert granularity·auto-delete-head recovery·git push credentials·harness fixation |
| Anti-pattern | No mixing of orthogonal scope |
| Checklist / case studies / environment-adaptation notes | Go·execution checks, reference instances, per-repo bindings |

---

## Recommended for

- Those who deal with PR stacks where, like authorization·render, **actual behavior only exists once integrated**
- Those who want to **click through for review on one converged surface** instead of juggling multiple previews
- Those who batch-merge but **don't want to lose per-feature revert granularity**
- Those who want to handle the accident of `auto-delete-head` making downstream PRs CLOSED **as a prevention·recovery procedure**
- Those who want to **choose by situation** between this and [review-merge](https://github.com/wild-mental/review-merge-skill), which gates one at a time

---

## References

- Cursor Agent Skills: [Creating Skills](https://docs.cursor.com)
- Claude Code Skills: [Extend Claude with skills](https://docs.anthropic.com/en/docs/claude-code/skills)
- Codex Agent Skills: [Agent Skills](https://developers.openai.com/codex/skills/)
- Sister skill: [review-merge](https://github.com/wild-mental/review-merge-skill) (REVIEW → MERGE)

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

## License

[MIT License](LICENSE)
