# Contributing

Thank you for considering a contribution to Feature Loop.

One thing makes this project unusual to contribute to: **the "code" is Markdown**. Each skill
is a plain-text operating procedure that an AI coding agent executes. That means reading a
skill is understanding it, improving one is a text edit — and "does it work" can only be
proven by *running* it, not by unit tests. Contributions are judged accordingly: on clarity,
on the behavior they produce in real runs, and on whether they respect the project's contracts
(below).

## What contributions are welcome

### Bug reports

A "bug" here is a skill doing the wrong thing in a real run: a loop that games its own
convergence, a reviewer prompt that produces noise, a step that breaks on GitHub when it works
on GitLab, an instruction agents consistently misread.

- Search existing issues first. Duplicates will be closed with a pointer to the original.
- State: **which skill**, **which agent/harness** you ran it in, **what it did**, and **what
  it should have done**. A sanitized excerpt of the run (the skill's report output, the MR
  comment it posted) is worth more than a paragraph of description.
- Reports without enough detail to reproduce the behavior may be closed. That's not hostility
  — a behavior nobody can reproduce is a behavior nobody can fix.

### Feature suggestions

- Check [`docs/plans/`](docs/plans/) first — the roadmap lives there, and several ideas
  (plan-review stage, feature queue, parallel worktrees, post-landing watch) are already
  designed and deliberately deferred. A suggestion that's already on the roadmap will be
  linked there and closed.
- Search existing issues for duplicates.
- The burden of the case is on the requester: what gap does it close for an engineering team
  using the loop, and why does it belong in the core rather than in your own fork of a skill?
  Whether a feature fits the project's scope is ultimately the maintainer's call.

### Improvements that are especially wanted

- **Sharper review prompts** — measurably better findings per token (bring before/after
  evidence from real MRs).
- **More reviewer backends** — any OpenAI-compatible endpoint qualifies; keep the contract
  (see below).
- **Forge support** — better `gh` equivalences, other forges (Gitea, Bitbucket) as "Forge
  note" mappings.
- **Portability notes** — how to load the skills in more agent harnesses, with graceful
  fallbacks for missing features.
- **Docs** — corrections, clearer explanations, honest FAQ entries.

## The contracts (non-negotiable in review)

Every PR is checked against these; they are what make the loop trustworthy:

1. **Agent-agnostic.** No model-specific or harness-specific instruction without a graceful
   fallback ("your harness's task tracker, or a plain list"). Nothing vendor-locked.
2. **Reviewer independence.** Reviewers are *different models* from the author agent; their
   output is posted **verbatim** — never edited, filtered, or softened. Context given to
   reviewers informs, it never defends the diff.
3. **Honest exits.** No change may let the loop game its own convergence: threads are never
   resolved without a fix or a stated dismissal, "merged/shipped" is only claimed after
   verification, caps and truncations are always disclosed.
4. **Noise-aversion.** Nothing posts a comment, report line, or file the team didn't need.
   "No issues found" comments, decorative output, and speculative configuration are all
   rejected on sight.
5. **Keys via environment only.** Never in files, never in prompts sent to reviewers, never
   echoed.

## Pull requests

- **Open an issue first** for anything that changes a skill's behavior, and wait for a nod
  before investing serious effort. Typo- and docs-level fixes can go straight to a PR.
- **Small and focused.** One concern per PR. A PR that rewrites three skills to fix one bug
  will be asked to shrink.
- **Bring evidence.** For behavior changes, include what a real run produced before and after
  (sanitized report output or MR comments). The skills cannot be unit-tested; your run *is*
  the test.
- **Follow the project's own commit rules** — they're in
  [`skills/build-feature/SKILL.md`](skills/build-feature/SKILL.md) and this repo practices
  them: `<prefix>: <imperative summary>`, ≤ 50 characters, no trailing period, body only when
  the *why* needs room. PRs with a messy history may be squashed on merge.
- Review takes real maintainer time; acceptance is discretionary. A declined PR is not a
  judgment of you — it's a scope decision.

## Be respectful

- No aggression, no demands, no deadlines. Maintainers and contributors here owe you nothing;
  everything you receive is given freely.
- Read the [README](README.md) and search issues and [`docs/plans/`](docs/plans/) before
  asking a question that's already answered.
- Don't nitpick or bikeshed in threads — if a discussion has been settled, relitigating it
  costs everyone.
- If the project's direction doesn't suit you, the MIT license is your friend: fork it, bend
  it to your needs, and godspeed.

## Licensing

By contributing, you agree that your contributions are licensed under the [MIT License](LICENSE),
the same license as the project.
