---
description: 4-phase spec-harness-implement-verify loop for a feature/refactor, enterprise-grade
---

/goal ultracode ultrathink

Target: $ARGUMENTS

Execute the above target in 4 phases. Each phase has hard requirements, not suggestions.

PHASE 0 — TRACKER DISCOVERY
Check whether this repo has a usable issue tracker: `gh repo view` (GitHub) or an equivalent already configured (Jira/Linear MCP, etc.). If one exists, it's the Phase 1 spec's home. If none exists, ask the user where the spec should live before proceeding — do not default to writing a spec file into the repo.

PHASE 1 — SPEC (write before any code change)
Create or update an issue in the tracker found in Phase 0, containing numbered, testable rules covering:
1. Isolation — per-instance state only, no module-level mutable state. Why: shared state across instances causes cross-request data leaks and hard-to-reproduce bugs.
2. Security — no dynamic code execution (`eval`/`exec`/`pickle`/`shell=True`); no untrusted input reaches a path, command, or query unsanitized; secrets read from env only, never logged. Why: these are the actual injection/exfiltration vectors attackers exploit, not hypothetical ones.
3. Resiliency — network I/O wrapped in retry with exponential backoff and a bounded attempt count; failures degrade gracefully. Why: transient network failures are the norm, not the exception, in production.
4. Performance — explicit size/row/byte caps on unbounded payloads; heavy imports and subprocess/network startup are lazy. Why: unbounded payloads exhaust memory/context budgets; eager startup cost is paid even by callers who never use the feature.
5. Clean code — one implementation per concern; every public function/class has a one-line docstring; no dead code, no TODOs, no stubs. Why: duplicate logic and half-finished code are where regressions hide.
6. Compliance — state applicability (or explicit N/A with reason) for PCI-DSS, SOC 2, ISO 27001. Why: a silent gap here is a silent audit failure later.
Each rule ends with a `Proof:` line naming the exact test/harness gate that checks it — an unfalsifiable rule can't be enforced.
Never write a spec file into the repo (no `CONSTITUTION_*.md` or similar) — the tracker issue is the durable record; a repo file just becomes clutter to clean up after merge. Code comments cite the issue ID (`see issue #NN`), never a file path — file paths get deleted, issue IDs don't.

PHASE 2 — HARNESS (build before any implementation)
Write one script that runs, in order, and fails loud on any non-zero exit:
1. Lint (project's existing linter/config).
2. SAST scan.
3. Multi-instance isolation test — construct 2+ instances in one process, assert no shared state. This is the only way to actually prove rule 1, not just assert it.
4. Dead-code scan.
5. Unit/integration tests proving each Phase-1 rule.
6. E2E test of the real user flow (headed Playwright if browser-based) with recorded proof (screenshot/video/log).
Commit the script as permanent infra — it's the regression gate for every future change to this area, not one-time scaffolding. Write its run output to one dedicated, gitignored subfolder, added to `.gitignore` before the first run, so re-running it never dirties the working tree with logs no one will commit.
Run it now. It must fail — no implementation exists yet, and a harness that passes before you've built anything is testing nothing. Do not proceed to Phase 3 until you've confirmed this failing baseline.

PHASE 3 — IMPLEMENTATION
One module at a time — a monolithic rewrite hides which change broke what. Re-run the harness after each module. Before deleting a file/function you believe is unused, prove it via the harness (dead-code gate + full suite green), or ask the user — "looks unused" and "is unused" differ often enough to matter. Every line you write must be complete and tested on the spot; a stub or TODO left behind is a promise the next person has to keep, and they won't know it's there.

PHASE 4 — TERMINATION
Do not declare completion in prose — a claim isn't a proof, and this is the whole point of the harness. The only valid termination is: harness exits 0, AND you've closed the Phase-1 issue with a comment containing the full gate-by-gate result, so the proof outlives this session. Print that same closing comment as your last output.
