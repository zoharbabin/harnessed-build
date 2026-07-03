# `/harnessed-build`

A [Claude Code](https://claude.com/claude-code) slash command that forces an AI coding agent to spec, harness, implement, and verify before it's allowed to call anything done.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-slash%20command-006efa)](https://claude.com/claude-code)

## Install

```bash
curl -o ~/.claude/commands/harnessed-build.md \
  https://raw.githubusercontent.com/zoharbabin/harnessed-build/main/commands/harnessed-build.md
```

Or clone the repo and symlink it in:

```bash
git clone https://github.com/zoharbabin/harnessed-build.git
ln -s "$(pwd)/harnessed-build/commands/harnessed-build.md" ~/.claude/commands/harnessed-build.md
```

Personal commands in `~/.claude/commands/` are available across every project. Drop the file in a repo's `.claude/commands/` instead if you want it scoped to just that project.

## Use

```
/harnessed-build migrate the payments module off the deprecated SDK to the new async client
```

The agent then, in order:
1. Checks for an issue tracker and writes the spec there.
2. Builds a verification script and confirms it fails (nothing's built yet).
3. Implements module by module, re-running the script after each one.
4. Closes the issue with the passing run log — the only way the task ends.

## Why it exists

Most feature requests to an AI coding agent end the same way. The agent writes code, runs it once, and declares success. Nothing forces it to define "done" before it starts. Nothing stops it from claiming victory on a hunch instead of a test result.

`/harnessed-build` removes that option. It's a four-phase process — spec, harness, implementation, verified close — and each phase has to produce something checkable before the next one can begin. The agent can't skip to "it works," because the only way to end the run is a script exiting 0.

## Why it beats the alternatives

**Vs. "just ask for the feature":** you get code, but no record of what it should do and no proof it does it. Six months later, nobody — human or agent — can tell if a change respected the original intent.

**Vs. a design doc you write yourself:** this shifts that work onto the agent. It also forces a testable format — numbered rules, each with a `Proof:` line — instead of prose that sounds thorough but nobody can check.

**Vs. writing spec files into the repo** (e.g. `CONSTITUTION_FEATURE.md`): those pile up as clutter you clean up later. This command puts the spec in a GitHub issue (or your existing tracker) instead. That's a durable record that lives outside the codebase and closes with the proof attached.

## Vs. the heavier frameworks

There's a real category of more serious spec/harness-driven process frameworks and execution harnesses out there: GitHub's own [Spec Kit](https://github.com/github/spec-kit), [OpenSpec](https://github.com/Fission-AI/OpenSpec), [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD), and agent-execution platforms like [OpenHands](https://github.com/OpenHands/OpenHands) and [mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent). Here's where `/harnessed-build` sits relative to them, and when to reach for each.

- **Spec Kit and OpenSpec keep a living spec; `/harnessed-build` keeps a closed record.** Spec Kit's `spec.md`/`plan.md`/`tasks.md` and OpenSpec's `openspec/specs/` stay open. You reopen and edit them across a feature's life, sometimes a project's whole life. `/harnessed-build`'s GitHub issue covers one task only, and closes the moment the harness passes. Both approaches use plain, git-trackable, team-visible files — the difference is shape, not durability. Reach for Spec Kit or OpenSpec when the spec itself needs to keep evolving alongside the code.
- **BMAD-METHOD brings purpose-built personas; `/harnessed-build` stays generic.** BMAD ships 12+ role-specific skill files — PM, architect, QA, dev — plus a "Party Mode" where multiple personas collaborate in one session. That's real depth for specific SDLC scenarios. None of these tools, BMAD included, actually coordinate multiple *humans*. BMAD's "team" features are one operator directing many AI personas, not access control between human roles. So `/harnessed-build`'s single generic role isn't a scaling gap — it's a depth-of-persona one. It leans on you defining the task well upfront, instead of picking a ready-made template.
- **OpenHands and mini-swe-agent run headless in CI; `/harnessed-build` runs inside a conversation with your agent.** OpenHands has a real GitHub Action — label an issue `fix-me` and it opens a PR with zero human in the loop — plus a headless mode built for pipelines. mini-swe-agent has an unattended `--yolo` flag and container support for sandboxed, scaled runs; that's what powers its SWE-bench submissions. `/harnessed-build` isn't a pre-built script at all. It's an instruction set you hand your agent. The agent then researches and writes a bespoke harness for the task at hand, and runs it interactively, in one sitting.
- **The red-before-green gate isn't unique to this command — BMAD-METHOD has one too.** Its `bmad-dev-story` skill spells out an actual red-phase/green-phase step sequence with stop-on-failure checks. Like `/harnessed-build`'s "confirm it fails first," the agent enforces this by following instructions, not by external code. Neither is a true programmatic gate. Spec Kit and OpenHands only suggest test-first in prose; OpenSpec skips gates by design. `/harnessed-build`'s real difference: one portable slash command, no framework to install, no persona to pick — and it ties its ending to a closed tracker comment so the proof outlives the session.

**When to graduate to one of these instead:** pick Spec Kit or OpenSpec when the spec needs to keep evolving across a project's life. Pick BMAD when a task fits one of its purpose-built SDLC personas better than a generic one. Pick OpenHands or mini-swe-agent for unattended CI-triggered fixes at scale with no human review step. `/harnessed-build` covers one task, one sitting, no install — you trade real capability for zero setup.

## Where it can fall short

- **No GitHub repo.** If the repo has no GitHub remote and no other configured tracker, the agent stops and asks you where the spec should live. It won't guess.
- **Fuzzy or exploratory asks.** This is built for "build X to spec," not "what should we build?" Have that design conversation first, then hand the agent a concrete target.
- **Harness gaps mean false confidence.** The process is only as strict as the checks it writes. A shallow test suite still passes. The ritual doesn't replace an agent reasoning about test quality — it just forces a testable ritual in the first place.
- **Cost.** Four phases, a generated test suite, and possibly a live e2e pass — that's real token and time spend. For a five-line bug fix, save this for changes worth the ceremony.
- **Nothing here is a mechanical gate — the agent could skip a step.** Every phase transition — spec before harness, a confirmed-failing baseline before implementation, a green harness before closing the issue — relies on the agent choosing to follow instructions. No code blocks it. Nothing stops it from writing the spec after the code, skipping the failing-baseline check, or closing the issue on a hunch. This is the one place a real harness with enforced script gates reliably beats this command. Wire the same checks into a Claude Code [hook](https://docs.claude.com/en/docs/claude-code/hooks) instead (`PreToolUse`/`Stop` in `settings.json`). Have it inspect real repo state: an open tracker issue exists, a harness script newer than the changed files exists, that script last exited non-zero before any implementation edit. Block the tool call if any check fails. OpenHands' Stop Hooks work the same way. The tradeoff: that gate becomes a project-specific script you write and maintain, not one portable file you drop into `~/.claude/commands/`. You trade this command's zero-install pitch for actual enforcement. Reach for it when a task is worth that setup cost — this command is for when it isn't.

## Contributing

Issues and PRs welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Good places to start: sharpen the Phase-1 rule set, add a tracker-discovery path for non-GitHub trackers (Jira, Linear, GitLab Issues), or report a repo/language where the harness phase needed a different shape.

## License

[MIT](LICENSE)
