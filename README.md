<div align="center">

# ⚡ debloat

<p><strong>Your coding agent loads thousands of tokens before your first word.<br/>See the bill. Cut the dead weight. One command.</strong></p>

<p>
<a href="https://www.npmjs.com/package/debloat"><img src="https://img.shields.io/npm/v/debloat?style=for-the-badge&color=CB3837&logo=npm" alt="npm"></a>
<a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-22C55E?style=for-the-badge" alt="MIT"></a>
<img src="https://img.shields.io/badge/node-%3E%3D18-4FC3F7?style=for-the-badge&logo=node.js&logoColor=white" alt="Node >=18">
</p>

<p>
<a href="#quick-start"><kbd> &nbsp; Quick start &nbsp; </kbd></a>
<a href="#see-the-whole-bill"><kbd> &nbsp; The receipt &nbsp; </kbd></a>
<a href="#method-honestly"><kbd> &nbsp; How it counts &nbsp; </kbd></a>
</p>

</div>

---

**Claude Code loads dozens of skills into your context every session — and most you've never used.** `debloat` reads your real usage, finds the skills that never fire, and cuts them. Reversibly. Lighter context, every session.

## Quick start

One command, zero install:

```bash
npx debloat
```

![skills --pick: the burn graph, the skills you've never run, and the receipt for what you trimmed](https://raw.githubusercontent.com/katrinalaszlo/debloat/main/assets/skills-pick.png)

It shows what's costing you, recommends a safe cut (based on which skills have actually fired in your session logs), lets you choose how deep to go, and celebrates what you reclaimed — all reversible with `skills --enable`.

## Why care

Every always-loaded token brings compaction closer and competes with your actual work for the model's attention. Most setups accrete this invisibly — old memory entries, skills you installed once, rules files you forgot.

- **See the bill** — everything loaded before you type: system prompt, CLAUDE.md, memory, skills, agents.
- **Evidence, not vibes** — usage comes from your real session logs, not guesses.
- **Cut safely** — disabled skills move to a folder, never deleted. Restore with one command.
- **Find dead weight** — files you *think* are loaded that aren't (`AGENTS.md` is ignored unless `@imported`).
- **Scriptable** — `--json` output and `--budget` exit codes for CI.

> ⭐ **Reclaimed some tokens? [A star](https://github.com/katrinalaszlo/debloat) helps the next cluttered setup find this.**

## See the whole bill

Trimming skills is the actionable part. To see *everything* loaded before you type, run the plain receipt:

```bash
npx debloat receipt
```

```
 ⚡ 24.9k tokens loaded every session before your first word
 claude code · vault · 2026-07-18

   claude code base  ██████████                   ~9.2k
   instructions      ████████                      7.5k
   memory            ████                          3.8k
   skills + agents   █████                         4.4k

 DEAD WEIGHT — on disk, NOT loaded
   AGENTS.md                           17.8k chars
     ignored unless @imported from CLAUDE.md

 MCP, loads on use: supabase, gbrain

  ┌───────────────────────────────────────┐
  │                                       │
  │~24.9k tokens  ·  12% of a 200k window │
  │                                       │
  │     ~4.0k of it is skill listings     │
  │     trim the unused: npx debloat      │
  │                                       │
  └───────────────────────────────────────┘
   * chars÷4 estimate · reconcile with /context
```

## skills — cost vs usage, and disable what you never run

The receipt tells you skills cost tokens every session. It doesn't tell you *which* to cut. `skills` does, by reading your actual usage out of the session logs:

```bash
npx debloat skills            # print the cost-vs-usage table
npx debloat skills --pick     # recommend a safe cut, then one y/n
```

`--pick` shows your token burn as before/after bars (context now vs. after the cut), lists exactly which never-invoked skills it recommends disabling (excluding safety rails like `git-safety` that fire on their own), and asks a single yes/no. Nothing is cut sight-unseen, and nothing is deleted.

```
 SKILLS — cost vs usage (user scope)
 ──────────────────────────────────────────────────────
 SKILL                             cost  last used
 printing-press-amend               233  never
 plan-tune                          201  never
 humanize                            90  20d ago
 last30days                          48  today
 ──────────────────────────────────────────────────────
 96/115 never invoked · ~8.4k tokens reclaimable
```

Usage is real, not guessed: it counts Skill-tool calls and slash-command invocations across `~/.claude/projects/**/*.jsonl`. "Never invoked" is evidence, not a verdict — a safety skill you keep loaded may rarely fire, so **you** decide, not the tool.

Disabling is **reversible and never deletes**: chosen skills move to `~/.claude/skills-disabled/`. Restore any time:

```bash
npx debloat skills --disable plan-tune printing-press-amend
npx debloat skills --enable plan-tune
```

## explain — which instructions apply to one file

`/context` tells you what's loaded *now*. It can't tell you which instructions apply when Claude touches a specific file:

```bash
npx debloat explain src/auth/login.ts
```

```
 src/auth/login.ts
 inherits 7 instruction layers (~2.0k tokens est)

 1. CLAUDE.md (global)            [1.5k]   always (startup)
 5. CLAUDE.md (project)           [15]     always (startup)
 6. @import docs/extra.md         [4]      always (startup)
 7. CLAUDE.md (src)               [17]     when Claude works in this directory

 OVERLAPPING SECTIONS — same heading in multiple layers,
 all in context at once; review for contradictions:
   "git" — CLAUDE.md, src/CLAUDE.md
```

It resolves the global + project `CLAUDE.md` chain, nested per-directory `CLAUDE.md` files (which load lazily), and one level of `@imports` — then flags headings that appear in more than one layer, since those are where contradictory rules hide. Claude Code loads all applicable layers **simultaneously**; there is no enforced cascade, so `explain` reports layers and overlaps rather than pretending a precedence engine exists.

## Isn't this just /context?

`/context` is the ground truth inside a live session — use it. debloat is for what `/context` can't do: it works without opening a session, it knows which skills you've actually invoked (from your session logs), and it does the cut for you. Run both — the receipt should land within ~8% of `/context`'s total.

## All commands

```bash
npx debloat                  # interactive trimmer (on a terminal)
npx debloat receipt          # the full itemized receipt
npx debloat receipt ~/proj   # receipt for another project
npx debloat skills           # cost-vs-usage table
npx debloat skills --pick    # recommended cut, one y/n
npx debloat explain <file>   # instruction layers for one file
npx debloat --json           # write debloat.json (diffable, CI-able)
npx debloat --budget 30000   # exit 1 if the estimate exceeds the budget

# Piped, scripted, or agent-run? The bare command prints the static receipt —
# it never prompts without a keyboard.
```

## Method, honestly

- **Static scan** of the same files Claude Code reads: the global and project `CLAUDE.md` chain (walking up from the project directory), `~/.claude/rules/`, one level of `@imports`, the project's auto-memory `MEMORY.md`, skill frontmatter (name + description are listed in context at startup), and agent frontmatter.
- **Token counts are estimates** (chars ÷ 4). Reconcile against `/context` inside a live session — that's the ground truth this tool approximates without needing a session.
- The Claude Code base system prompt and built-in tool schemas can't be derived statically; they're included as constants measured via `/context` on v2.1.215.
- Calibration: on the author's setup the static estimate landed ~17% under the live `/context` total (22.5k estimated vs 27.2k measured). Treat the receipt as a floor — your real bill is higher.

## Roadmap

- Real tokenizer, `/context` auto-reconciliation
- `debloat diff` between two receipts (did that PR add 4k always-loaded tokens?)
- Usage-aware pruning for agents and plugin skills too
- Codex CLI, Cursor, Gemini CLI receipts

## License

MIT. One file, zero dependencies — free to use, copy, and make yours.

<div align="center">

**Built by [Kat Laszlo](https://github.com/katrinalaszlo)** · a star helps other cluttered setups find this

</div>
