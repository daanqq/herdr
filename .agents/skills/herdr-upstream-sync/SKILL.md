---
name: herdr-upstream-sync
description: Sync a Herdr fork branch with upstream/master while preserving uncommitted work and resolving merge or stash-replay conflicts. Use when asked to update the fork from upstream, merge upstream master, or catch a fork branch up to the canonical repository.
---

# Herdr upstream sync

Use this workflow only inside the Herdr repository. Integrate `upstream/master` with a merge so fork commits retain their identity. Do not rebase, push, discard changes, or clean untracked files unless the user explicitly requests that separate action.

## Establish the sync boundary

Inspect the current branch, worktrees, status including staged and untracked files, remotes, HEAD, and any in-progress Git operation. Confirm that `upstream` is the canonical Herdr repository and that the checked-out branch is the fork branch the user wants updated.

Stop before changing anything when:

- a merge, rebase, cherry-pick, or revert is already in progress;
- the requested branch is checked out in another worktree;
- the remote or target branch is ambiguous;
- existing changes cannot be safely distinguished from the sync.

Record the pre-sync status and the names of staged, unstaged, and untracked paths. Completion criterion: every existing local change has a known preservation path, and the exact source and target refs are known.

## Fetch and classify

Fetch `upstream master` without pulling. Re-read status, then inspect the ahead/behind relationship and the commits unique to each side.

- If `upstream/master` is already an ancestor of HEAD, do not create an empty merge.
- If HEAD can fast-forward and the branch has no fork-only commits, fast-forward it.
- If both sides have commits, merge `upstream/master` into the fork branch.

Before creating a merge commit, follow the repository's commit-message alignment rule. Do not hide a required approval inside an auto-committing `git merge` command.

## Preserve local work

When the worktree or index is dirty, create a clearly named stash including untracked files before integrating upstream. Record the stash identity and keep it until restoration has been verified. Never use a temporary commit for the user's unfinished work.

The stash is a safety copy, not permission to overwrite files. If stashing fails or leaves a dirty tree, stop and report the state.

## Integrate upstream

Merge the fetched remote-tracking ref, not an unfetched name or a moving network ref. Resolve each conflict from its surrounding code and both parents:

1. Identify the fork behavior and the upstream replacement or architectural change.
2. Preserve both when they remain compatible.
3. Adapt the fork behavior to the new upstream interface when upstream supersedes the old structure.
4. Search for renamed fields, callers, tests, fixtures, and protocol contracts before declaring the resolution complete.

Do not choose all `ours` or all `theirs`, and do not resolve generated or compatibility files by intuition. Regenerate them through the repository workflow when applicable. Completion criterion: Git reports no unmerged paths, no conflict markers remain in resolved text files, and `upstream/master` is an ancestor of HEAD.

## Restore local work

Apply the recorded stash with its index state after upstream integration. If replay conflicts, resolve them as local work rebased onto the merged tree: preserve the user's intent while adapting it to upstream changes.

Compare the resulting staged, unstaged, and untracked paths with the pre-sync record. Restore the original staging boundary when possible. Do not silently stage the user's previously unstaged files. Drop the stash only after the restored work has been inspected and accounted for; otherwise retain it and report its identity.

Completion criterion: every pre-sync local change is present or has an explicitly reported conflict, and no unrelated local artifact was removed.

## Validate

Run these checks in order:

1. `git diff --check` and a conflict-marker search over files involved in conflicts.
2. Verify `upstream/master` is an ancestor of HEAD and report the final ahead/behind counts.
3. Run formatting and the cheapest focused checks for code touched during conflict resolution.
4. Run the repository-required check when the environment supports it. Report missing tools and any unrun stages precisely.

Do not fix unrelated pre-existing failures as part of the sync.

## Report

Report the fetched upstream commit, resulting HEAD, conflicts resolved, local-work restoration status, checks and blockers, and whether anything was pushed. A sync is not complete if unmerged paths remain or preserved work is unaccounted for.
