---
name: using-git-worktrees
description: Deprecated compatibility skill for older workflows that still ask for git worktrees - forwards to ensure-isolated-workspace and prefers worktree creation when isolation is needed
---

# Using Git Worktrees

This skill remains only as a compatibility entrypoint for older prompts, plans, and documentation.

**New primary skill:** `ensure-isolated-workspace`

**Announce at start:** "I'm using the using-git-worktrees skill as a compatibility path. I'll use ensure-isolated-workspace to choose the right workspace mode."

## Compatibility Policy

When this skill is invoked:

1. Treat the request as a request for safe workspace isolation
2. Immediately invoke `ensure-isolated-workspace`
3. If isolation is needed and the user has not said otherwise, prefer creating a worktree
4. If an existing linked worktree is already present, reuse it
5. If a clean in-place workspace is sufficient and the user explicitly prefers not to use worktrees, allow that path

Do not maintain a second, separate worktree-creation policy here. The authoritative behavior now lives in `ensure-isolated-workspace`.

## Migration Note

For new workflows:

- use `ensure-isolated-workspace`
- do not introduce new references to `using-git-worktrees`

For old workflows:

- keep this skill working as a compatibility alias
- bias toward worktree creation only when isolation is actually needed

## Why This Changed

The project now distinguishes between:

- reusing an existing isolated workspace
- allowing safe in-place work for small clean tasks
- creating a worktree when risk or coordination requires it

This is broader than "always create a new worktree", so the old name is no longer the preferred concept.

## Red Flags

**Never:**

- add new workflow references to this deprecated skill
- keep a second full procedure here that can drift from `ensure-isolated-workspace`
- assume the user wants a new worktree in every environment

**Always:**

- route new logic through `ensure-isolated-workspace`
- keep older prompts working
- make the compatibility behavior explicit

## Integration

**Called by:**

- Older prompts, plans, and docs that still name this skill directly

**Pairs with:**

- **ensure-isolated-workspace** - Authoritative workspace-selection policy
