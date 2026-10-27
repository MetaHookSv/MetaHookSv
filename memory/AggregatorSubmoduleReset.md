---
title: AggregatorSubmoduleReset
type: note
permalink: metahooksv/aggregator-submodule-reset
tags:
- submodules
- cmake
- aggregator
- lesson
---

# Aggregator configure resets submodules to the root index

- Trigger: after committing or checking out new commits inside a submodule (MetaHook, a plugin, a vendor fork), configuring the root aggregator reverts the submodule to an older commit and leaves untracked directories behind (for example renamed submodule paths).
- Root cause: with `METAHOOKSV_INIT_SUBMODULES=ON` (the default), the root `CMakeLists.txt` runs `git submodule update --init --recursive`, which checks out the gitlink recorded in the root index. Commits that exist only in the submodule are left on their branch, but the working checkout is detached to the old gitlink.
- Correct approach: commit inside each submodule, then `git add <submodule>` in the root (staging the new gitlinks) before configuring. Alternatively configure with `-DMETAHOOKSV_INIT_SUBMODULES=OFF` while iterating locally.
- Recovery: `git -C <submodule> checkout main` (or the intended commit), remove stray untracked submodule worktrees left at old paths, then `git -C <parent> submodule update --init -- <paths>`.
- Verification: the configure log shows no `Submodule path ... checked out` lines, and `git submodule status --recursive` matches the intended commits.
- Scope: any cross-repository change in the MetaHookSv aggregator (first observed while removing VC-LTL and renaming MetaHook's SDL submodules, 2026-10-27).
