# Fork Customization Guide

This fork intentionally changes the default workspace-isolation behavior of `superpowers`.

The upstream project historically treated `using-git-worktrees` as the primary way to establish an implementation workspace. This fork changes that default:

- `ensure-isolated-workspace` is the preferred primary workflow
- `using-git-worktrees` remains only as a compatibility shim
- clean in-place execution is allowed in explicitly safe cases

## Why This Fork Exists

The main reason for this fork is compatibility with environments where workspace isolation may already be provided externally, or where forcing a new worktree on every task creates unnecessary friction.

Examples:

- Codex-managed worktree environments
- small single-task changes in a clean branch
- repositories whose tests or generated files are sensitive to path changes

## High-Conflict Files

These files are likely to conflict with upstream changes and should be reviewed carefully during merges or rebases:

- `README.md`
- `docs/README.codex.md`
- `skills/writing-plans/SKILL.md`
- `skills/executing-plans/SKILL.md`
- `skills/subagent-driven-development/SKILL.md`
- `skills/finishing-a-development-branch/SKILL.md`
- `skills/using-git-worktrees/SKILL.md`
- `skills/using-superpowers/references/codex-tools.md`

This fork also adds:

- `skills/ensure-isolated-workspace/SKILL.md`
- `docs/superpowers/specs/2026-03-27-workspace-isolation-migration-design.md`
- `docs/superpowers/plans/2026-03-27-workspace-isolation-migration-implementation.md`

## Merge Policy

When syncing upstream:

1. Rebase or merge upstream into the fork branch
2. Review every conflict in the files listed above manually
3. Preserve this fork's workspace-isolation semantics unless there is a deliberate decision to change them
4. Re-run targeted searches for stale references to `using-git-worktrees` as the primary workflow

## Conflict Resolution Rule

When upstream and this fork disagree, use this priority:

1. Preserve upstream improvements that are unrelated to workspace-isolation behavior
2. Preserve this fork's decision that `ensure-isolated-workspace` is the primary workflow
3. Preserve compatibility support for older `using-git-worktrees` references
4. Keep cleanup behavior compatible with both in-place and worktree execution

## Customization Boundary

To reduce future merge pain, prefer this order for new customization work:

1. Add new docs
2. Add new skills
3. Adjust compatibility shims
4. Modify core upstream workflow skills only when necessary

Avoid broad edits to upstream core files unless the behavior must actually change.

## Future Strategy

If this fork accumulates many more workflow differences, consider moving additional custom behavior into a separate local skill pack instead of continuing to deepen the fork.

That would reduce merge pressure, but it would also weaken automatic integration with upstream `superpowers` workflows. For now, this fork chooses tighter integration over lower maintenance cost.

## Verification Checklist After Upstream Sync

After each upstream sync, check:

- `ensure-isolated-workspace` still exists and is discoverable
- `using-git-worktrees` is still only a compatibility shim
- `writing-plans`, `executing-plans`, and `subagent-driven-development` still reference `ensure-isolated-workspace`
- `finishing-a-development-branch` still handles in-place workspaces correctly
- entry docs still describe `ensure-isolated-workspace` as the preferred workflow
