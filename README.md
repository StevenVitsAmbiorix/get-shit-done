# Get Shit Done (Custom Fork)

A fork of [get-shit-done](https://github.com/glittercowboy/get-shit-done) with custom extensions.

## Attribution

This project is based on **get-shit-done v1.9.4** by [glittercowboy](https://github.com/glittercowboy).

## Custom Extensions

Two custom features have been integrated on top of upstream:

1. **Todo/Audit Linkage System** - Automatic tracking of which todos/audits are resolved by which phases
2. **Context Trimming Workflow** - Periodic archival to prevent context bloat

See [CONTRIBUTIONS.md](CONTRIBUTIONS.md) for detailed documentation of these features.

## Update Workflow

When updating from upstream:

```bash
# 1. Backup custom files
cp workflows/trim-context.md /tmp/gsd-backup/
cp references/todo-audit-linkage.md /tmp/gsd-backup/

# 2. Run upstream update
npx get-shit-done-cc@latest

# 3. Re-initialize git
cd ~/.claude/get-shit-done
git init
git remote add origin https://github.com/StevenVitsAmbiorix/get-shit-done.git
git add -A && git commit -m "chore: fresh install from npx"

# 4. Restore custom files
cp /tmp/gsd-backup/* workflows/  # or references/

# 5. Re-apply execute-plan.md changes (see CONTRIBUTIONS.md)

# 6. Commit
git add -A && git commit -m "feat: reintegrate custom contributions"
git push origin main --force
```

## Files Modified from Upstream

| File | Change |
|------|--------|
| `workflows/trim-context.md` | **Added** - Context trimming workflow |
| `references/todo-audit-linkage.md` | **Added** - Todo linkage documentation |
| `workflows/execute-plan.md` | **Modified** - Added process_linked_todos/audits steps |
| `templates/phase-prompt.md` | **Modified** - Added addresses_todos/audits frontmatter |
