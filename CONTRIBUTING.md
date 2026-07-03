# Contributing

This repo is one file that matters: [`commands/harnessed-build.md`](commands/harnessed-build.md). Everything else supports it.

## Reporting an issue

Open a GitHub issue. Useful reports include:
- A task/repo where Phase 0 (tracker discovery) picked the wrong home for the spec, or should support a tracker it doesn't yet (Jira, Linear, GitLab Issues).
- A Phase 1 rule that didn't fit your language/stack, or a rule that's missing a `Proof:` line worth adding.
- A Phase 2 harness step that didn't make sense for your project type (no browser, no network I/O, etc.) and how you'd word the fallback.
- Any case where the agent declared completion without the harness passing — that's the one thing this command should never allow.

## Proposing a change

1. Open an issue first for anything beyond a typo fix — this command is short and dense; a change to one line changes what every user's agent does.
2. Keep edits minimal. Every phase is intentionally terse: each rule gets one instruction plus one short "Why:" clause for model reasoning, not more.
3. If you add or reword a rule, test it by running `/harnessed-build` against a real (ideally toy) repo and confirming the resulting behavior — spec written correctly, harness fails before implementation, harness passes after, issue closed with proof.
4. Send a PR against `commands/harnessed-build.md`. Explain what behavior changes and why in the PR description.

## Style

- No filler. Every sentence should change what the agent does.
- Keep the "Why:" reasoning — token-efficiency should never come at the cost of removing the rationale a model needs to apply a rule correctly in edge cases.
- Don't reintroduce repo-committed spec files (`CONSTITUTION_*.md` or similar) as a pattern — the whole point of this command is that specs live in the tracker, not the codebase.
