# /assembly — Orchestrator

**Agents:** `assembly-planner` → `assembly-executor` → `assembly-reviewer`

---

## Purpose
Drive ≤2‑SP Jira tickets end‑to‑end **without Git, patches, or snapshots**. The **Executor** uses an Edit Tool to apply changes directly to the working tree. The **Reviewer** inspects the **current files listed in `CHANGED.PATHS`** and returns a decision with **reasons** (plain language). On revision, the **Planner** decides what to change based on those reasons. The orchestrator holds **no persistent state** between cycles.

---

## Parameters
- **ticket_key** *(required)* — Jira key like `PROJ-123`
- **mode** *(optional)* — `auto` (default) or `interactive`
  - `interactive`: preview the proposed edits (`EDITS.V1`) and `CHANGED.PATHS` and apply only after confirmation
- **max_cycles** *(optional)* — default `3`

---

## Agent I/O contracts (inline blocks)
- **Planner → Orchestrator/Executor/Reviewer**
  - `---BEGIN TODO.V{n}--- … ---END TODO.V{n}---` *(n starts at 1; increment on replans)*
- **Executor → Orchestrator/Reviewer**
  - `---BEGIN EDITS.V1--- … ---END EDITS.V1---` *(declarative edit plan; may be applied immediately in `auto` mode)*
  - `---BEGIN APPLY_LOG.MD--- … ---END APPLY_LOG.MD---` *(per‑edit success/errors)*
  - `---BEGIN CHANGED.PATHS--- … ---END CHANGED.PATHS---` *(newline list)*
  - `---BEGIN SUMMARY.MD--- … ---END SUMMARY.MD---` *(rationale, notes)*
- **Reviewer → Orchestrator**
  - `---BEGIN REVIEW.MD--- … ---END REVIEW.MD---` *(first line `Decision: APPROVED|REVISE`; contains `reasons:` list of text items)*
- **Orchestrator → Planner (for replans)**
  - `---BEGIN REASONS.V1--- … ---END REASONS.V1---` *(plain list of reasons from reviewer and/or tools; no diffs/hunks)*

All agent data is exchanged inline. **No snapshots, no patch files, no state tracking.**

---

## Orchestration steps

### 0) Preflight
1. Fetch Jira via Atlassian MCP: title, description, acceptance criteria (ACs), and story points (SP).
2. **Gate:** if SP > 2 → stop and ask to split the ticket. Allow if SP is missing and the scope appears ≤2 SP.
3. Verify working directory is repo root and writable.

### 1) Plan
- Call **`assembly-planner`** with Jira context **(+ optional `REASONS.V1` if this is a replan)**.
- Capture solely `TODO.V{n}` (tasks, constraints, test_strategy, expected_paths).

### 2) Execute (direct edits)
- Call **`assembly-executor`** with Jira + `TODO.V{n}`.
- **`mode=interactive`:** executor emits `EDITS.V1` without applying; orchestrator displays `EDITS.V1` + `CHANGED.PATHS` preview and, on confirmation, tells executor to apply.  
- **`mode=auto`:** executor may apply immediately.
- Capture: `EDITS.V1`, `APPLY_LOG.MD`, `CHANGED.PATHS`, `SUMMARY.MD`.

### 3) Review (no snapshots)
- Provide **`assembly-reviewer`** with: ACs, `TODO.V{n}`, `CHANGED.PATHS` (and it may read those files from disk), plus `SUMMARY.MD` for context.
- Receive `REVIEW.MD`:
  - `Decision: APPROVED` → go to Finalize.
  - `Decision: REVISE` → collect the reviewer’s **`reasons:`** (plain language).

### 4A) Finalize *(if APPROVED)*
Run validators/linters/tests (scoped where possible):
- `bundle exec packwerk check`
- `bundle exec rubocop -A` on changed Ruby files
- `npx eslint --fix` on changed `client/**` paths
- Run targeted tests from `TODO.V{n}.test_strategy` (RSpec/Jest/Playwright)

If all green → emit **Finalize Report** (tool outputs + `CHANGED.PATHS`).  
If any fail → treat the failures as **additional reasons** and replan.

### 4B) Replan *(if REVISE or finalize failures)*
1. Build **`REASONS.V1`** by aggregating:
   - Reviewer `reasons:` (plain text, may reference files/lines),
   - Validator/linter/test failure summaries.
2. Increase attempt. If `attempt > max_cycles`, stop and emit an unresolved summary.
3. Return to **Plan** with `REASONS.V1` (Planner decides the changes).

---

## `REASONS.V1` format
```
---BEGIN REASONS.V1---
- AC2 not met: deleting a user returns 500 on nil org_id (see packs/users/.../delete_user.rb).
- Missing unit tests for FooService error branch.
- FE import alias used; must import via "fe-design-base/...".
---END REASONS.V1---
```

---

## Boundaries & conventions
- **No Git / no VCS.** Only direct edits via Edit Tool.
- **No snapshots.** Reviewer inspects current files under `CHANGED.PATHS`.
- **Packs‑first:** put new domain code under `/packs/<domain>/...`; avoid `/app/` business logic.
- **FE imports:** use `fe-design-base/...` (no `@` aliases).
- Each attempt is independent; the **Planner** interprets `REASONS.V1` and updates `TODO.V{n+1}`.
