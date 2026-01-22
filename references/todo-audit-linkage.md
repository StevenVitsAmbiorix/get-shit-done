# Todo/Audit Linkage Reference

This document describes how GSD workflows link and process todo and audit files during phase planning and execution.

## Overview

The todo/audit linkage feature provides automated tracking between:
- Phase work defined in ROADMAP.md
- Todo items in `.planning/todos/pending/`
- Audit findings in `.claude/reports/audit/`

**Key benefit:** When a phase addresses a todo or audit, the system automatically tracks completion rather than requiring manual updates.

## Plan-Time Linkage

### How to Link Todos in ROADMAP.md

When defining a phase in ROADMAP.md, explicitly reference any todos it will address:

```markdown
### Phase 38: Plan-Time Todo Linkage
**Goal**: Modify plan-phase.md to discover and link todos
**Todo**: `.planning/todos/pending/2026-01-19-gsd-auto-todo-updates.md`
```

### Supported Patterns

The plan-phase.md workflow recognizes these patterns:
- `**Todo**: path/to/todo.md`
- `**Source todo**: path/to/todo.md`
- `Todo: path/to/todo.md`

### What plan-phase.md Does

When creating a PLAN.md, the workflow:
1. Scans the phase description in ROADMAP.md for todo patterns
2. Verifies the referenced todo file exists
3. Adds the path to `addresses_todos` frontmatter
4. Includes @reference to the todo in PLAN.md context

**Resulting PLAN.md frontmatter:**
```yaml
---
phase: 38-plan-time-todo-linkage
plan: 01
type: execute
wave: 1
depends_on: []
files_modified: []
autonomous: true
addresses_todos:
  - .planning/todos/pending/2026-01-19-gsd-auto-todo-updates.md
---
```

## Execute-Time Auto-Move

### What Happens When a Plan Executes

The execute-plan.md workflow processes linked todos after completing all tasks:

1. **Check for addresses_todos** in PLAN.md frontmatter
2. **Verify todo exists** in `pending/` directory
3. **Move to completed/** directory
4. **Append completion metadata**
5. **Stage changes** for commit

### Completion Metadata Format

When a todo is moved, metadata is appended:

```markdown
---
**Completed:** 2026-01-19
**Completed by:** Phase 38, Plan 01
**Summary:** .planning/phases/38-plan-time-todo-linkage/38-01-SUMMARY.md
```

This creates a clear audit trail linking the todo to its resolution.

### Example Console Output

```
Todo completed: 2026-01-19-gsd-auto-todo-updates.md
  Moved to: .planning/todos/completed/
  Linked to: 38-01-SUMMARY.md
```

## Audit Handling

### How Audits Work Differently

Unlike todos (which are binary complete/not-complete), audits contain multiple findings that may need individual resolution. Because of this:

- **Audits are NOT auto-updated**
- **A reminder is displayed** prompting manual review

### Audit Reminder Format

```
================================================================================
AUDIT UPDATE REMINDER
================================================================================

This plan addressed findings from:
.claude/reports/audit/generalization-2026-01-19.md

Consider updating:
- Mark resolved items as COMPLETE
- Update recommendation status
- Add notes about implementation approach

Files to review:
.claude/reports/audit/generalization-2026-01-19.md

(This is a reminder only - audit updates require human review)
================================================================================
```

### Linking Audits in ROADMAP.md

Same pattern as todos:

```markdown
### Phase 37: Architecture Centralization
**Goal**: Move architectures to models/architectures/
**Audit**: `.claude/reports/audit/generalization-2026-01-19.md`
```

## Best Practices

### When to Use Explicit Linkage

**Use explicit ROADMAP linkage when:**
- A phase directly addresses a documented todo
- A phase resolves specific audit findings
- You want automatic completion tracking

**Don't rely on auto-matching:**
- The workflow uses conservative extraction (explicit references only)
- No auto-matching by file path overlap or area similarity
- Better to miss a link than create a false one

### When to Let the Workflow Infer

The system does NOT auto-infer todo linkage from:
- Files modified by the phase
- Keywords in phase description
- Area/subsystem overlap

This is intentional - the ROADMAP is the source of truth.

### Directory Structure

```
.planning/
  todos/
    pending/     # Active todos awaiting resolution
    completed/   # Resolved todos with completion metadata
```

## Example: Complete ROADMAP Entry

```markdown
## Milestone v1.1.2: GSD Todo/Audit Automation

### Phase 38: Plan-Time Todo Linkage
**Goal**: Modify plan-phase.md to discover and link todos/audits
**Todo**: `.planning/todos/pending/2026-01-19-gsd-auto-todo-updates.md`
**Effort**: Small

### Phase 39: Execute-Time Auto-Update
**Goal**: Modify execute-plan.md to process linked todos
**Depends**: Phase 38
**Effort**: Small

### Phase 40: Verification & Documentation
**Goal**: Test workflow, update GSD docs
**Depends**: Phase 39
**Effort**: Small
```

## Error Handling

### Todo Not Found

If a linked todo is not found in `pending/`:
- Warning is logged
- Execution continues
- May indicate todo was moved manually

### Completed Directory Creation

If `completed/` directory cannot be created:
- Error is logged
- Todo is NOT moved
- Execution continues

---

*Feature added in GSD v1.1.2 (2026-01-19)*
*Phases 38-40: Plan-Time Todo Linkage, Execute-Time Auto-Update, Verification & Documentation*
