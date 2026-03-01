---
name: using-git-worktrees
description: "Worktree creation is disabled in this fork. This skill is intentionally a no-op."
---

# Using Git Worktrees (DISABLED)

Worktree isolation has been disabled in this fork. Do not create a git worktree.

## What To Do Instead

Work directly in the current working directory on the current branch. All implementation, testing, and commits happen in place.

**Announce at start:** "Worktree isolation is disabled — working directly in the current directory."

## When Other Skills Reference This Skill

If another skill says "REQUIRED: Set up isolated workspace before starting" or references `superpowers:using-git-worktrees`, ignore that requirement. Proceed with implementation in the current directory.
