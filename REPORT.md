Superpowers Fork Retirement Investigation
=========================================

> **Provenance:** Requested from AgenC mission `ae362b33` (session `2bf4d7a9`). Investigation
> performed in AgenC mission `f49d7997-79ef-452a-a806-c67ea973408a`. Upstream state captured by
> fetching `obra/superpowers` on 2026-05-28; upstream HEAD `f2cbfbe` (Release v5.1.0, 2026-04-30).

TL;DR
-----

**Recommendation: Retire the fork.** Switch back to upstream `obra/superpowers` (and the upstream
marketplace `obra/superpowers-marketplace`), and re-express every load-bearing carry as a thin config
layer — primarily a few lines in your global CLAUDE.md/AgenC instructions plus `edge-case-discovery`
living in your own skills collection. Upstream has done the work that makes this clean:

1. The hooks.json bug that triggered this investigation **is already fixed upstream** (and more besides).
2. Upstream now **explicitly honors user CLAUDE.md instructions over skill behavior** (a documented
   priority hierarchy as of v5.0.0), and the rewritten worktree skill **honors a declared worktree
   preference without asking** — which is exactly the mechanism your biggest carry needs.
3. You are 154 commits and a **major version (v4.3.1 → v5.1.0)** behind. The fork is accumulating
   real cost: you're missing the zero-dependency brainstorm server, multi-platform support, the
   worktree rewrite, review-loop refinements, and a pile of cross-platform hook fixes.

The one carry that does *not* collapse cleanly into config is the **brainstorming-flow edits**
(batched questions + three-phase design review), because upstream rewrote that same file heavily.
Those are soft preferences, partly obsoleted by upstream's own rework — drop them or re-apply as
small CLAUDE.md nudges rather than maintaining a forked SKILL.md that will conflict on every pull.

---

Where things stand
------------------

- **Upstream:** `obra/superpowers` (Jesse Vincent / fsck.com). Confirmed from the README fork note.
- **Fork point:** merge-base `e4a2375` — you forked right after upstream's **v4.3.1** release.
- **Your fork carries 11 commits** on top of that point.
- **Upstream has added 154 commits** since, spanning releases **v5.0.0 → v5.0.1 → v5.0.2 → v5.0.4 →
  v5.0.5 → v5.0.6 → v5.0.7 → v5.1.0**. v5.0.0 was a breaking major bump.
- **Marketplace fork (`mieubrisse/superpowers-marketplace`):** carries only 2 fork-specific commits —
  it repoints the `superpowers` and `superpowers-dev` plugin `source.url` at your fork and adds a
  README note. It is pinned to a stale `4.3.1`. Every other plugin entry still points at `obra/*`.
  **Its entire reason to exist is to redirect to your superpowers fork.** Retire the superpowers
  fork and this fork becomes pure redundancy — use `obra/superpowers-marketplace` directly.

---

Question 1 — What has upstream added since you forked?
------------------------------------------------------

### The trigger bug is already fixed

Your fork's `hooks/hooks.json` still has:

```json
"command": "'${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd' session-start"
```

The single quotes stop `/bin/sh` from expanding `${CLAUDE_PLUGIN_ROOT}`, which is what broke your
SessionStart hook after the Claude Code 2.1.119 → 2.1.145 upgrade.

Upstream's current `hooks/hooks.json`:

```json
"matcher": "startup|clear|compact",
"command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start"
```

So upstream fixed it **by switching to escaped double quotes — not the 2.1.139 `args[]` exec form.**
This landed in **v5.0.1** (PR #585, issues #577/#529/#644), explicitly described as "single quotes
around `${CLAUDE_PLUGIN_ROOT}` ... on Linux single quotes prevent variable expansion ... replaced
with escaped double quotes." Verified across macOS bash, Windows cmd.exe, Git Bash, and Linux dash.

Upstream **also dropped `resume` from the matcher** (your fork still has `startup|resume|clear|compact`;
upstream is `startup|clear|compact`). That was an intentional v5.0.3 fix: the startup hook was
re-injecting context on resumed sessions that already had it in history. Worth picking up regardless.

There were *several* other hook robustness fixes in v5.0.3 you're missing: Bash 5.3+ heredoc hang on
Homebrew bash (replaced heredoc with printf), POSIX-safe `$0` instead of `${BASH_SOURCE[0]}` for dash,
and `#!/usr/bin/env bash` shebangs for NixOS/FreeBSD. **Given you run heavy custom config on macOS
with Homebrew, the Bash 5.3+ heredoc fix is directly relevant to you.**

### Major features and architecture changes (v5.0.0+)

Touching surfaces you care about (hooks, multi-platform, config-steerability):

- **Instruction priority hierarchy (v5.0.0)** — `using-superpowers` now documents an explicit order:
  (1) user's explicit instructions (CLAUDE.md/AGENTS.md) — highest, (2) Superpowers skills, (3) default
  system prompt. *"If CLAUDE.md says 'don't use TDD' and a skill says 'always use TDD,' the user's
  instructions win."* **This is the linchpin that makes the config-layer approach viable.**
- **`<SUBAGENT-STOP>` gate (v5.0.0)** — dispatched subagents skip the skill bootstrap instead of
  re-triggering full workflows. Relevant to your AgenC subagent-heavy flows.
- **Worktree skills rewrite (v5.1.0, PRI-974)** — `using-git-worktrees` and
  `finishing-a-development-branch` now: detect existing isolation (`GIT_DIR != GIT_COMMON`) with a
  submodule guard; **ask consent before creating a worktree and honor a declared preference without
  asking**; prefer the harness's native worktree tool; only clean up worktrees under `.worktrees/`.
  Fixes #991 (SDD was auto-creating worktrees without consent) — the exact behavior you forked to kill.
- **Zero-dependency brainstorm server (v5.0.2)** — removed ~1,200 lines of vendored node_modules;
  pure built-in `http`/`fs`/`crypto`. Plus owner-PID lifecycle hardening (v5.0.6) and a security fix
  separating served `content/` from private `state/`.
- **Visual brainstorming companion (v5.0.0)** — optional browser companion for mockups/diagrams.
- **Review-loop evolution** — v5.0.0 added subagent spec/plan review loops; v5.0.4 refined them
  (single whole-plan pass, calibrated blocking); **v5.0.6 replaced them with inline self-review**
  after measuring that the subagent loop added ~25 min with no quality gain.
- **Execution-choice flip-flop** — v5.0.0 made subagent-driven-development *mandatory* on capable
  harnesses; **v5.0.5 restored user choice** (SDD recommended, not mandatory). Net: current upstream
  *offers* the choice. (Bears on your "always subagent-driven" carry — see Q2.)
- **Multi-platform support** — Cursor (v5.0.3), Gemini CLI (v5.0.1), GitHub Copilot CLI (v5.0.7),
  OpenCode fixes throughout, Codex plugin mirror tooling. Cross-platform hook formats (`hooks-cursor.json`).
- **Breaking (v5.0.0):** specs/plans now save to `docs/superpowers/specs/` and `docs/superpowers/plans/`
  (was `docs/plans/`). User preference overrides. Migration is just moving files.
- **Removals (v5.1.0):** legacy slash commands (`/brainstorm`, `/write-plan`, `/execute-plan`) deleted;
  the `superpowers:code-reviewer` named agent removed (merged into a `general-purpose` dispatch template).

### Contributor guidance that matters for your strategy

v5.1.0 added a CLAUDE.md/AGENTS.md section addressed to AI agents, after an audit found a 94% PR
rejection rate. **They explicitly will not accept: project-specific configuration, domain-specific
skills, fork-specific changes, or "compliance" rewrites of skill content.** Translation: most of your
carries (worktree-disable, AgenC-specific behavior, edge-case-discovery) **would be rejected upstream
by policy.** That's not an argument for keeping the fork — it's an argument for the **config-layer**,
since the whole point of the v5.0.0 priority hierarchy is to let you steer without upstreaming.

---

Question 2 — Can you retire the fork?
-------------------------------------

### What you carry, and whether it is still load-bearing

Your 11 commits touch 15 files. Grouped by intent:

| # | Carry | Files | Still load-bearing? | Reframe as config? |
|---|-------|-------|---------------------|--------------------|
| 1 | **Worktrees disabled** (no-op skill + removals in 4 skills) | `using-git-worktrees` (−216), `finishing-a-development-branch` (−48), `executing-plans`, `subagent-driven-development`, `writing-plans` | **Partly.** Upstream now asks consent + honors declared preference. AgenC clones (not linked worktrees) so upstream *would* offer to create one — but only with consent. | **Yes — fully.** One CLAUDE.md line: "Never create git worktrees; work in place on the current branch." Upstream honors it without asking. |
| 2 | **Always subagent-driven** (remove execution choice) | `writing-plans` | **Weakly.** On Claude Code, SDD is already the recommended default; upstream offers the choice rather than forcing it. | **Yes.** CLAUDE.md line: "Always use subagent-driven-development; don't offer inline execution." |
| 3 | **Batched brainstorming questions** (plain text vs AskUserQuestion) | `brainstorming` | **Soft preference.** AskUserQuestion works fine; this was a stylistic choice. | **Partial.** Expressible as a CLAUDE.md nudge, but upstream rewrote this file heavily — a forked copy will conflict every pull. |
| 4 | **Three-phase design review** (vs upstream's rounds) | `brainstorming` | **Largely obsoleted.** Upstream reworked brainstorming (scope assessment, inline self-review, visual companion) since you forked. Your condensation predates all of it. | **Drop or re-derive.** Not worth carrying against a moving file. |
| 5 | **edge-case-discovery** (new skill, 6 files) | `skills/edge-case-discovery/*` + a hook in `brainstorming` | **Yes — genuinely additive and yours.** | **Yes, with a caveat.** The *skill* drops cleanly into your own skills collection. The *auto-integration* into the brainstorming flow needs either a one-line brainstorming hook (lost on a clean upstream) or a CLAUDE.md instruction telling the agent to run it after architecture approval. |
| 6 | **Metadata / branding** (plugin.json ×2, README fork note) | `.claude-plugin/`, `.cursor-plugin/`, `README.md` | **No.** Exists *only because* it's a fork. | **N/A — disappears on retirement.** |

### The reframe, concretely

The combination that makes this clean:

- **v5.0.0 priority hierarchy** — upstream skills now defer to your CLAUDE.md by design.
- **v5.1.0 worktree skill** — honors a declared worktree preference without prompting.

So carries #1, #2, and the integration trigger for #5 become **a handful of lines in your global
AgenC/Claude instructions**. Carry #5's skill content lives in your own skills directory (where your
dozens of other skills already live), layered alongside upstream. Carries #3 and #4 are soft/obsoleted
— let them go. Carry #6 vanishes.

This is strictly better than a fork: you stop merging 154-commit deltas by hand, you inherit every
future hook/platform fix automatically, and your customizations sit in a layer upstream is explicitly
built to respect.

### Costs and caveats of retiring

- **edge-case-discovery auto-integration is the one real loss.** In the fork it's wired into the
  brainstorming SKILL.md so it fires automatically. On clean upstream you reproduce that via a
  CLAUDE.md instruction ("after architecture approval in brainstorming, run edge-case-discovery") —
  reliable given the priority hierarchy, but a notch less seamless than an edited SKILL.md. If that
  auto-fire is important enough, the fallback is a *thin wrapper plugin* that adds only the skill +
  a brainstorming nudge, not a full fork.
- **Spec/plan path change (v5.0.0)** — `docs/plans/` → `docs/superpowers/plans/`. Cosmetic; override
  via preference if you care, otherwise migrate or ignore.
- **One-time setup cost** — writing the CLAUDE.md lines, moving edge-case-discovery into your skills
  collection, repointing the marketplace to `obra/superpowers-marketplace`, reinstalling the plugin.
  A few hours, paid once, versus open-ended per-pull merge tax.

### What I'd want to confirm before pulling the trigger (not done in this mission)

- That AgenC's clone-per-mission model plays well with upstream's Step 0 isolation detection. A clone
  has `GIT_DIR == GIT_COMMON`, so upstream treats it as a normal repo and *would offer* a worktree —
  hence the CLAUDE.md "work in place" line is **required**, not optional, for fire-and-forget runs.
- That `edge-case-discovery` as a standalone skill (outside the brainstorming SKILL.md edit) actually
  triggers when you want it. Quick to validate with the existing skill content.

---

Recommendation
--------------

**Retire both forks.** Return to `obra/superpowers` + `obra/superpowers-marketplace`, and carry your
customizations as a config layer:

1. **CLAUDE.md / AgenC global instructions** — add: never create worktrees (work in place); always use
   subagent-driven-development; after architecture approval in brainstorming, run edge-case-discovery.
2. **Your own skills collection** — move `edge-case-discovery` in alongside your other skills.
3. **Drop** the brainstorming batched-questions and three-phase-review edits (soft, and obsoleted by
   upstream's rework). Re-add as small CLAUDE.md nudges only if you miss them.
4. **Marketplace** — install from `obra/superpowers-marketplace`; delete the fork repointing.

**Confidence: high** that the fork is retireable and the carries reframe as config. The crux that
would change my mind: if you discover the edge-case-discovery auto-fire genuinely *must* be embedded
in the brainstorming flow and a CLAUDE.md instruction proves unreliable, then a **thin wrapper plugin**
(skill + one nudge, tracking upstream as the real dependency) — not a full fork — is the fallback.

*Per the task scope, no retirement action was taken in this mission. This report ends at the recommendation.*
