# /gsd:trim-context

Trim planning files to maintain fast context loading. Archives historical content to STATE-ARCHIVE.md.

**When to run:**
- Manually when context feels slow
- Automatically after `/gsd:complete-milestone`

---

<step name="measure_current_state">

Measure current file sizes and determine if trimming is needed.

```bash
wc -l .planning/STATE.md .planning/ROADMAP.md 2>/dev/null
```

**Thresholds:**
- STATE.md: Target ≤120 lines (warn at >150)
- ROADMAP.md: Target ≤150 lines (warn at >200)

**If both within targets:**

```
✅ Planning files already optimized

STATE.md: [X] lines (target: ≤120)
ROADMAP.md: [X] lines (target: ≤150)

No trimming needed.
```

Exit workflow.

**If either exceeds threshold:**

```
📊 Current planning file sizes:

STATE.md: [X] lines (target: ≤120) [⚠️ OVER if applicable]
ROADMAP.md: [X] lines (target: ≤150) [⚠️ OVER if applicable]

Proceeding with trim...
```

Continue to next step.

</step>

<step name="read_current_files">

Read the files that need trimming:

```bash
cat .planning/STATE.md
cat .planning/ROADMAP.md
```

Parse STATE.md to identify:
- Project Reference section
- Current Position section
- Performance Metrics section
- Shipped Milestones section (identify which to keep vs archive)
- Accumulated Context section (identify current vs old milestone summaries)
- Session Continuity section

Parse ROADMAP.md to identify:
- Current milestone (status: in_progress or planned)
- Completed milestones (already archived to milestones/)

</step>

<step name="create_archive_entry">

Read or create STATE-ARCHIVE.md:

```bash
cat .planning/STATE-ARCHIVE.md 2>/dev/null || echo "Creating new archive"
```

**If file doesn't exist, create with header:**

```markdown
# State Archive

Historical context trimmed from STATE.md to maintain fast context loading.
Full history preserved here for reference.

**Usage:** This file is append-only. Each trim adds a dated entry below.

---
```

**Append new archive entry:**

```markdown

## Archive Entry: [YYYY-MM-DD] (After [milestone version])

Trimmed by `/gsd:trim-context`

### Shipped Milestones (archived from STATE.md)

[Copy all shipped milestone entries EXCEPT the last 2]

### Accumulated Context (archived from STATE.md)

[Copy all milestone summaries EXCEPT the current in-progress milestone]

### Performance Metrics Detail (archived from STATE.md)

[Copy any detailed per-phase metrics tables if present]

---
```

Write the updated STATE-ARCHIVE.md.

</step>

<step name="trim_state_md">

Create trimmed STATE.md with this structure:

```markdown
# Project State

## Project Reference

See: .planning/PROJECT.md (updated [date from original])

**Core value:** [keep from original]
**Current focus:** [keep from original]

## Current Position

[Keep entire section as-is - phase, plan, status, progress bar, branch]

## Performance Metrics

**Velocity:** [total] plans completed across [N] phases (~[X] min/plan, ~[Y] hours total)

## Recent Milestones

[Keep only last 2 shipped milestones, one line each:]
- **v[X.Y]** [Name] — [N] phases, [M] plans (shipped [date])
- **v[X.Y-1]** [Name] — [N] phases, [M] plans (shipped [date])

*Full history: .planning/STATE-ARCHIVE.md*

## Current Milestone Context

[Keep the ENTIRE current/in-progress milestone section as-is, including:]
- Goal
- Key decisions
- Blockers
- Phase status
- Any other current milestone details

## Session Continuity

[Keep entire section as-is]
```

**Validation:**
- Count lines in new STATE.md
- Should be ~100-120 lines
- If significantly over, check if current milestone section is unusually large

Write the trimmed STATE.md.

</step>

<step name="trim_roadmap_md">

For ROADMAP.md, apply these rules:

**Keep full details for:**
- Current milestone (in_progress status)
- Next planned milestone (if any)

**Collapse completed milestones to:**

```markdown
## [Milestone Name] (Shipped: [date])

**Archive:** [milestones/vX.Y-name-ROADMAP.md](milestones/vX.Y-name-ROADMAP.md)

<details>
<summary>Phases [X]-[Y] (click to expand)</summary>

[Original phase list - collapsed by default]

</details>
```

**If milestone archive file exists:** Just link to it, don't duplicate content.

**If no archive exists:** Keep details in collapsed `<details>` block.

Write the trimmed ROADMAP.md.

</step>

<step name="verify_and_report">

Verify the trim succeeded:

```bash
wc -l .planning/STATE.md .planning/ROADMAP.md .planning/STATE-ARCHIVE.md
```

**Report:**

```
✅ Context trimmed successfully

| File | Before | After | Reduction |
|------|--------|-------|-----------|
| STATE.md | [X] | [Y] | [Z]% |
| ROADMAP.md | [X] | [Y] | [Z]% |

Archived to: .planning/STATE-ARCHIVE.md (+[N] lines)

💡 Next trim recommended after next milestone completion.
```

</step>

<step name="git_status">

Show git status for user to review changes:

```bash
git diff --stat .planning/STATE.md .planning/ROADMAP.md
git status .planning/STATE-ARCHIVE.md
```

**Note:** Do NOT auto-commit. User should review diffs before committing.

```
📝 Changes ready for review. Run `git diff .planning/` to inspect.

When satisfied:
git add .planning/STATE.md .planning/ROADMAP.md .planning/STATE-ARCHIVE.md
git commit -m "chore: trim planning context for performance"
```

</step>

---

## Checklist

- [ ] Current file sizes measured
- [ ] STATE-ARCHIVE.md created/updated with archived content
- [ ] STATE.md trimmed to ≤120 lines
- [ ] ROADMAP.md trimmed to ≤150 lines (completed milestones collapsed)
- [ ] Line counts verified
- [ ] Git status shown for user review

---

## Integration with complete-milestone

To auto-trigger after milestone completion, add to `complete-milestone.md` at the end of `offer_next` step:

```markdown
---

💡 **Context maintenance:** Planning files may have grown during this milestone.

Run `/gsd:trim-context` to optimize for next milestone.
```

Or for fully automatic execution, add a new step after `git_commit_milestone`:

```markdown
<step name="auto_trim_context">

Automatically trim planning context after milestone completion.

Run: `/gsd:trim-context`

(This step runs the trim workflow inline)

</step>
```
