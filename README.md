# Claude Feature Workflow

Five [Claude Code](https://claude.com/claude-code) skills that cover the full lifecycle of shipping a feature — from a rough idea to a merged, independently-reviewed change — including one command that runs the entire loop by itself:

```
                        /ship-feature  (the loop, one command)
┌─────────────────────────────────────────────────────────────────────────┐
│ /plan-feature ──→ /build-feature ──→ /mr-review ──→ /address-review     │
│     (plan)           (code + MR)     (external AI      (triage, fix,    │
│                                        reviewers)        reply)         │
│                                           ▲                 │           │
│                                           └── re-review ────┘           │
│                                              until clean                │
│                                                  │                      │
│                                                  ▼                      │
│                                    MERGE GATE  --merge=ask|auto|never   │
└─────────────────────────────────────────────────────────────────────────┘
```

The loop's core idea: **the model that writes the code never reviews its own work.** Claude plans and builds; independent external models (OpenAI GPT, xAI Grok) review the MR with fresh eyes and post their findings as comments; Claude then triages those findings honestly — fixing what's real, dismissing what isn't, and replying to every thread with its reasoning — and re-reviews until the review comes back clean. Each skill works standalone; `/ship-feature` chains them.

## The skills

### `/ship-feature` — the loop, one command

Dump a feature brief and walk away:

```
/ship-feature Add CSV export to the reports page — finance needs month-end data in Excel
```

It plans (asking its clarifying questions up front — the loop's one designed touchpoint), builds and raises the MR, gets the external reviews, addresses them, and **re-reviews until convergence**: zero valid findings (or all verdicts "No blocking issues"), every thread answered, CI green. Then the merge gate:

| Flag | Behavior |
|---|---|
| `--merge=ask` *(default)* | stops at **READY TO MERGE** and waits for your approval |
| `--merge=auto` | merges when converged + CI green (`glab mr merge` / `gh pr merge`) |
| `--merge=never` | reports and leaves the MR open |
| `--max-rounds=N` *(default 3)* | honest exit if reviewers keep finding things |
| `--autonomous` | planner records defaults instead of asking questions |

Safeguards: no-progress detection (a finding surviving two rounds stops the loop instead of thrashing), honest convergence (threads are never resolved just to exit), and the sub-skills' hard stops (architectural concern, red build, red pipeline, merge conflicts) always surface to you.

### `/plan-feature` — product-owner + architect planner

Give it a few paragraphs on what you want to build. It:

1. **Studies your codebase first** — structure, conventions, the nearest existing feature, reusable pieces — and never plans from memory. Every integration point is cited as `file:line`.
2. **Frames the feature by what the user achieves** — outcome, not mechanism — and names the smallest version that delivers it.
3. **Challenges the brief** — one batched round of only the genuinely blocking questions, each with a recommended answer. Defaultable decisions are made and *recorded* so you can veto by reading.
4. **Audits every UI element** against "less is more": each button/field/label must name what the user achieves by it, or it's cut. Empty/loading/error states are designed in the plan, not discovered during coding.
5. **Proves feasibility** against real code, then writes a phased plan — small self-contained phases (goal, files, elaborate changes, verify step, commit message) that a coding agent can execute without coming back with questions.

Output: a plan file in `docs/plans/`, ready for `/build-feature`.

### `/build-feature` — disciplined plan executor

Takes the plan file (defaults to the newest in `docs/plans/`) and:

- Branches **fresh from main** — always pulls latest, never builds on a stale base.
- Reads the plan **whole, line by line**, then re-grounds every phase in the current code before writing.
- Codes with clean-code discipline: SOLID applied not recited, DRY with judgment, reuse-first, surgical changes only.
- **No unflagged hacks**: a forced workaround exists only with a `HACK:` comment and an entry in the hack ledger.
- **Async by default, synchronized by justification**: every deliberate sync point (transaction, unique constraint, idempotency key, lock) goes in a synchronization ledger with the invariant it protects.
- Gates every phase with type-check + tests, commits granularly (one commit per logical change), and finishes with a full verification sweep including a grep audit of every consumer of every changed surface.
- **Always raises the MR/PR** when green, with a fixed four-section description: **Problem / Solution / Technical details / Review notes**.

Two hard stops only: an architectural concern (it asks, with a recommendation) and a red build (it never pushes red).

### `/mr-review` — independent external-AI review

The stage where different models look at the code. It fetches the MR's title, description, and diff, sends them to each configured external reviewer, and posts each reviewer's brief as a separate MR comment — verbatim, unedited:

- **OpenAI GPT** — correctness-first implementation brief: bugs, security, data loss, races, missing tests, performance. Severity-ranked, every issue citing `file:line`, each with a recommended fix and the tests to add.
- **xAI Grok** — quality review: DRY, over/under-abstraction, naming, cohesion, coupling, testability, consistency with local patterns.

Review-only: it never modifies code, approves, or merges. Large diffs are truncated **with disclosure** — omitted files are named to the reviewers so they don't produce false "file is missing" findings. A reviewer is enabled simply by its API key being present (see [Configuration](#configuration)).

### `/address-review` — review-comment triage and resolution

Closes the loop. It:

- Pulls every comment, splits compound notes, and correlates each one **line by line against the actual diff** — never judging from the reviewer's summary alone.
- Assigns honest verdicts: **VALID** (fix it), **PARTIAL** (right instinct, wrong specifics — fix the real concern), **INVALID** (dismissed, citing the specific context the reviewer couldn't see).
- For every valid finding, answers **"why I missed it"** with a real blind-spot category — the learning signal is the point.
- Implements all valid fixes (separate commit per comment), pushes on a green build, and replies to every thread — fixed ones marked done with the commit, invalid ones explained. The only hard stop is a red build.

## Install

### Option 1: as a plugin (recommended — one command)

In Claude Code:

```
/plugin marketplace add ohyesgocool/claude-feature-workflow
/plugin install feature-workflow@claude-feature-workflow
```

### Option 2: copy the skills

```bash
git clone https://github.com/ohyesgocool/claude-feature-workflow.git
cp -r claude-feature-workflow/skills/* ~/.claude/skills/
```

New Claude Code sessions pick the skills up automatically. Verify with `/plan-feature` appearing in the slash-command list.

## Configuration

### What you need (and what you don't)

| Requirement | Needed for | Where to get it |
|---|---|---|
| **Claude Code** (subscription or API login) | everything — planning, building, triage | [claude.com/claude-code](https://claude.com/claude-code) |
| **`glab` CLI, authenticated** (GitLab) or **`gh` CLI** (GitHub) | raising the MR, posting/reading comments | `glab auth login` / `gh auth login` |
| **`OPENAI_API_KEY`** | the GPT reviewer in `/mr-review` | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) |
| **`XAI_API_KEY`** *(optional)* | the Grok quality reviewer in `/mr-review` | [console.x.ai](https://console.x.ai) |
| `jq`, `curl` | `/mr-review` API calls | preinstalled on most systems / `brew install jq` |

> **You do NOT need a separate Claude/Anthropic API key.** The skills run inside Claude Code under your existing login — Claude is the orchestrator, planner, builder, and triager. The only external keys are for the *independent reviewers*: OpenAI (this is also the key Codex-style GPT reviews run on — one OpenAI key covers it) and optionally xAI for Grok. With neither key set, `/plan-feature`, `/build-feature`, and `/address-review` still work fully; only `/mr-review` will ask you to configure a reviewer.

### Reviewer settings

| Variable | Required | Default | Purpose |
|---|---|---|---|
| `OPENAI_API_KEY` | to enable the GPT reviewer | — | correctness review |
| `OPENAI_MODEL` | no | `gpt-5.5` | which OpenAI model reviews |
| `XAI_API_KEY` | to enable the Grok reviewer | — | quality review |
| `XAI_MODEL` | no | `grok-4.3` | which xAI model reviews |
| `MR_REVIEW_MAX_DIFF_CHARS` | no | `120000` | diff size limit sent to reviewers |

A reviewer runs if — and only if — its key is set. Set one key or both.

### Where to put the keys

**Option A — Claude Code settings (recommended: applies to every project, lives with Claude Code):**

Add an `env` block to `~/.claude/settings.json`:

```json
{
  "env": {
    "OPENAI_API_KEY": "sk-...",
    "XAI_API_KEY": "xai-...",
    "OPENAI_MODEL": "gpt-5.5",
    "XAI_MODEL": "grok-4.3"
  }
}
```

**Option B — shell profile** (`~/.zshrc` / `~/.bashrc`):

```bash
export OPENAI_API_KEY="sk-..."
export XAI_API_KEY="xai-..."
```

Either way: **never commit keys to a repository**, and never paste them into MR comments or issue text. The skills only ever reference them as environment variables.

## Running the loop

The one-command way:

```text
/ship-feature Add CSV export to the reports page — finance needs month-end data in Excel
   → answers 2 planning questions up front, then: plan → build → MR !87 → OpenAI+Grok
     review → 3 findings fixed + 1 dismissed with reasons → re-review clean → CI green
   → READY TO MERGE (say "merge it", or rerun with --merge=auto to skip this gate)
```

Or stage by stage, if you want control between steps:

```text
/plan-feature  Add CSV export to the reports page — finance needs month-end data in Excel
   → plan saved to docs/plans/2026-07-03-csv-export.md (answers 2 questions on the way)

/build-feature
   → branch feat/csv-export, 4 phases, granular commits, MR !87 raised with
     Problem / Solution / Technical details / Review notes

/mr-review
   → OpenAI + Grok each post an implementation brief on MR !87

/address-review
   → triages every finding, fixes the valid ones, replies to every thread, pushes

/mr-review        # optional second pass — re-review the fixed MR until clean
```

Each skill also works standalone — you can `/address-review` a colleague's MR, or `/mr-review` a hand-written change. `SHIP_MERGE_MODE=ask|auto|never` in your environment sets `/ship-feature`'s default merge behavior without passing the flag each time.

## Automating the review step (optional)

`/mr-review` is on-demand. If you want every MR reviewed automatically on creation, the same prompts work as a GitLab webhook service: a small FastAPI app that receives `merge_request` webhooks, fetches the diff, calls OpenAI/Grok, and posts the briefs — so reviews appear without anyone asking. The prompt templates in [`skills/mr-review/SKILL.md`](skills/mr-review/SKILL.md) are drop-in for that setup; pair the webhook with `/address-review` and the loop runs itself.

## License

MIT
