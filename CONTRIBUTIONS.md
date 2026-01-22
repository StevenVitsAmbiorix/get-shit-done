# Custom GSD Contributions

This document isolates specific features developed independently from upstream that could potentially be contributed back.

## Feature 1: Todo/Audit Linkage System

**Problem solved:** Manual tracking of which todos and audits are resolved by which phases is error-prone and tedious. Todos often get forgotten in `pending/` after they're addressed.

**Solution:** Automatic linking at plan-time, automatic completion at execute-time.

### Files Involved

| File | Purpose |
|------|---------|
| `references/todo-audit-linkage.md` | Full documentation (203 lines) |
| `workflows/plan-phase.md` | Plan-time extraction (lines 210-275) |
| `workflows/execute-plan.md` | Execute-time processing (lines 1409-1490) |
| `templates/phase-prompt.md` | Frontmatter fields definition |

### How It Works

#### 1. ROADMAP.md Declaration

When defining a phase in ROADMAP.md, explicitly reference todos/audits:

```markdown
### Phase 38: Plan-Time Todo Linkage
**Goal**: Modify plan-phase.md to discover and link todos
**Todo**: `.planning/todos/pending/2026-01-19-gsd-auto-todo-updates.md`
**Audit**: `.claude/reports/audit/generalization-2026-01-19.md`
```

#### 2. Plan-Phase Extraction (plan-phase.md)

When creating PLAN.md, the workflow:
1. Scans phase description in ROADMAP.md for patterns:
   - `**Todo**: path/to/todo.md`
   - `**Source todo**: path/to/todo.md`
   - `**Audit**: path/to/audit.md`
2. Verifies referenced files exist
3. Adds to PLAN.md frontmatter:

```yaml
---
phase: 38-plan-time-todo-linkage
plan: 01
addresses_todos:
  - .planning/todos/pending/2026-01-19-gsd-auto-todo-updates.md
addresses_audits:
  - .claude/reports/audit/generalization-2026-01-19.md
---
```

**Code location in plan-phase.md (lines 205-275):**

```markdown
**6. Extract explicitly-linked todos from ROADMAP:**

Check if the current phase description in ROADMAP.md explicitly references a todo:

Pattern to look for:
- `**Todo**: .planning/todos/pending/...`
- `**Source todo**: .planning/todos/pending/...`
- `Todo: .planning/todos/pending/...`

If found:
1. Verify the todo file exists
2. Add path to `addresses_todos` frontmatter in PLAN.md
3. Include @reference to the todo in PLAN.md context section

**Conservative approach:** Only link what's explicitly referenced. The ROADMAP is the
source of truth for which todos a phase addresses. Don't auto-match by file overlap
or area - better to miss a link than create a false one.
```

#### 3. Execute-Plan Processing (execute-plan.md)

After completing all tasks, the workflow:

**For todos (auto-complete):**
```markdown
<step name="process_linked_todos">

For each todo path in addresses_todos:
1. Verify the todo file exists in `pending/`
2. Move to `completed/` directory
3. Append completion metadata:
   ---
   **Completed:** 2026-01-19
   **Completed by:** Phase 38, Plan 01
   **Summary:** .planning/phases/38-plan-time-todo-linkage/38-01-SUMMARY.md
4. Stage the changes
</step>
```

**For audits (reminder only):**
```markdown
<step name="process_linked_audits">

Audits require human judgment (multiple findings, nuanced status).
Display reminder:

================================================================================
AUDIT UPDATE REMINDER
================================================================================

This plan addressed findings from:
.claude/reports/audit/generalization-2026-01-19.md

Consider updating:
- Mark resolved items as COMPLETE
- Update recommendation status
- Add notes about implementation approach

(This is a reminder only - audit updates require human review)
================================================================================
</step>
```

### Design Decisions

| Decision | Rationale |
|----------|-----------|
| Conservative extraction | Only explicit ROADMAP references - no auto-matching by filename or area |
| Todos auto-complete | Binary complete/not-complete state is automatable |
| Audits reminder-only | Multiple findings require human judgment on resolution status |
| Append metadata | Creates audit trail linking resolution to specific plan |

---

## Feature 2: Context Trimming Workflow

**Problem solved:** GSD planning files (STATE.md, ROADMAP.md) accumulate historical context over time, causing context bloat that slows down Claude's responses.

**Solution:** Periodic archival of historical content to STATE-ARCHIVE.md while preserving current milestone context.

### Files Involved

| File | Purpose |
|------|---------|
| `workflows/trim-context.md` | Full workflow (290 lines) |

### How It Works

#### 1. Measure Current State

```bash
wc -l .planning/STATE.md .planning/ROADMAP.md
```

**Thresholds:**
- STATE.md: Target ≤120 lines (warn at >150)
- ROADMAP.md: Target ≤150 lines (warn at >200)

#### 2. Archive Historical Content

Create/append to `.planning/STATE-ARCHIVE.md`:

```markdown
## Archive Entry: 2026-01-22 (After v1.5.0)

Trimmed by `/gsd:trim-context`

### Shipped Milestones (archived from STATE.md)
[All shipped milestones except last 2]

### Accumulated Context (archived from STATE.md)
[All milestone summaries except current in-progress]

### Performance Metrics Detail
[Any detailed per-phase metrics tables]
```

#### 3. Trim STATE.md

Reduced structure (~100-120 lines):

```markdown
# Project State

## Project Reference
[Keep - 5 lines]

## Current Position
[Keep entire section - phase, plan, status, branch]

## Performance Metrics
**Velocity:** [summary line only]

## Recent Milestones
[Last 2 shipped only, one line each]
*Full history: .planning/STATE-ARCHIVE.md*

## Current Milestone Context
[ENTIRE current milestone section - decisions, blockers, status]

## Session Continuity
[Keep entire section]
```

#### 4. Trim ROADMAP.md

- Keep full details for current + next planned milestone
- Collapse completed milestones:

```markdown
## v1.5.0 (Shipped: 2026-01-22)

**Archive:** [milestones/v1.5.0-ROADMAP.md](milestones/v1.5.0-ROADMAP.md)

<details>
<summary>Phases 62-63 (click to expand)</summary>
[Original phase list - collapsed by default]
</details>
```

### Integration Points

**After `/gsd:complete-milestone`:**

The trim-context workflow can be triggered automatically or via reminder:

```markdown
💡 **Context maintenance:** Planning files may have grown during this milestone.

Run `/gsd:trim-context` to optimize for next milestone.
```

### Expected Results

| File | Before | After | Typical Reduction |
|------|--------|-------|-------------------|
| STATE.md | ~700 lines | ~100 lines | 85% |
| ROADMAP.md | ~400 lines | ~150 lines | 62% |

**Token savings:** ~15,000 tokens freed from context window.

---

## Contributing Back to Upstream

These features are designed to be modular and could be contributed back to [glittercowboy/get-shit-done](https://github.com/glittercowboy/get-shit-done):

1. **Todo/Audit Linkage:** Requires changes to plan-phase.md and execute-plan.md templates
2. **Trim Context:** Self-contained workflow file, easy to add

Note: The upstream structure differs (nested directories, Node.js tooling), so PRs would need adaptation.
