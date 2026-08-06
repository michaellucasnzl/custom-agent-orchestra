# Agent Reference

What each agent does, what it expects, and what it produces.

---

## Conductor

**Purpose:** Orchestrator and single user-facing entry point. Coordinates sub-agents through the full lifecycle: Intake → Discovery → Planning → approval gate → per-phase Implementer → Reviewer → conditional Security Auditor → commit gate → Scribe.

**Tier:** Sonnet | **Writes:** Only to `plans/` and sub-agent prompts, never production code | **Directly invocable:** Yes

### Lifecycle

1. **Intake** — restates the request, asks clarifying questions, confirms scope.
2. **Discovery** — invokes Scout, receives structured findings.
3. **Planning** — invokes Architect with Scout's findings; writes the plan to `plans/<slug>-plan.md`.
4. **Mandatory stop** — presents the plan with Open Questions. Waits for explicit approval.
5. **Per-phase loop** — Implementer → Reviewer → conditional Security Auditor → commit gate. Stops after each phase for the user to commit.
6. **Failure handling** — `NEEDS_REVISION` returns to Implementer with verbatim feedback (max 2 auto-retries). `FAILED` stops and asks the user. Test/runtime errors route to Debugger.
7. **Completion** — invokes Scribe for final summary and changelog.

### Rules

- Never writes production code.
- Never dumps full sub-agent transcripts into its own context.
- Passes context explicitly between agents.
- Always announces which agent it is invoking and why.

---

## Scout

**Purpose:** Read-only discovery. Searches the codebase, identifies relevant files, patterns, conventions, test setup, dependencies, and risks. Returns structured facts only — no recommendations, no opinions.

**Tier:** Haiku | **Read-only** | **Directly invocable:** No

### Output Template

1. **Relevant Files** — path, role, and relevance.
2. **Existing Patterns & Conventions** — coding style, naming, error handling, logging, testing.
3. **Test Setup & Commands** — runner, framework, how to run full suite / single file / pattern.
4. **Dependencies & Integration Points** — key libraries, internal boundaries, API contracts, external services.
5. **Risks & Unknowns** — poorly tested areas, convention violations, unresolved questions.

Cap output at ~400 lines. Facts only.

---

## Architect

**Purpose:** Read-only planner. Takes Scout findings and a confirmed task, produces a phased implementation plan with 3–10 independently committable phases. Each phase targets 1–3 files, specifies tests to write first, and ends with acceptance criteria and rollback notes.

**Tier:** Opus | **Read-only** | **Directly invocable:** No

### Plan Structure

- **Summary** — 2–3 sentence overview.
- **Phase 1..N** — objective, files (create/modify), tests to write first, acceptance criteria, rollback notes.
- **Open Questions** — tagged `[TECHNICAL]` or `[PRODUCT]`. Product questions must be answered before implementation.
- **Risks** — likelihood + impact + mitigation per risk.

### Principles

Phases are independently committable, ordered by dependency, and follow existing conventions. New dependencies are conservative and justified.

---

## Implementer

**Purpose:** Executes a single phase of the plan using strict TDD. Stays inside the phase boundary; reports out-of-scope work rather than doing it. Never commits, never edits plan files.

**Tier:** Haiku | **Writes code and tests** | **Directly invocable:** No

### TDD Cycle

1. Write the failing test specified in the plan.
2. Run it, confirm failure.
3. Write minimal code to pass.
4. Run it, confirm green.
5. Run the full test suite to confirm no regressions.
6. Lint and format.
7. Report: files changed, tests added, results.

### Hard Constraints

- If no tests are specified in the phase, stop and request clarification.
- If the test passes before implementation, it is invalid — stop.
- If the full suite fails, stop — the Conductor routes to Debugger.
- If out-of-scope work is discovered, report it but do not execute it.

---

## Reviewer

**Purpose:** Read-only quality gate on uncommitted changes. Reviews correctness, test quality, scope adherence, conventions, error handling, and obvious performance issues. Returns exactly one verdict token.

**Tier:** Sonnet | **Read-only** | **Directly invocable:** No

### Verdicts

- **APPROVED** — no issues. No additional text.
- **NEEDS_REVISION** — numbered, actionable feedback at `file:line`. Return to Implementer.
- **FAILED** — architectural problem. Escalate to user.

No praise, no padding, no style nitpicking that passes the linter.

### Anti-Patterns Flagged

Tautological tests (assert `true === true`), empty catch blocks, hardcoded secrets, `console.log` in production paths, circular imports, functions >50 lines without clear reason.

---

## Security Auditor

**Purpose:** Read-only adversarial security review. Examines uncommitted changes for injection, authn/authz flaws, secret handling, unsafe deserialisation, SSRF, path traversal, dependency risk, insecure defaults, and sensitive data exposure.

**Tier:** Opus | **Read-only** | **Directly invocable:** Yes

### Invocation

- **Conditional:** Conductor invokes it when a phase touches auth, crypto, secrets, user input, network boundaries, new dependencies, file I/O with user-controlled paths, serialisation, or PII/payment data.
- **Direct:** User can invoke it standalone for an audit of any code.

### Output

```
### Critical/High/Medium/Low: title
- Location: file:line
- Description: what and how exploited
- Remediation: concrete fix
```

Then a summary count. States plainly when it finds nothing — clean reports are valuable.

---

## Debugger

**Purpose:** Hypothesis-driven failure analyst. Reproduces failures, isolates root causes, proves hypotheses with minimal experiments, and proposes the smallest fix.

**Tier:** Sonnet | **Writes fixes after root cause is proven** | **Directly invocable:** Yes

### Method

1. **Reproduce** — confirm the failure.
2. **Isolate** — narrow to the smallest reproducing case.
3. **Hypothesise** — state a root-cause hypothesis: "Hypothesis: <plain-language explanation>."
4. **Prove** — design and execute the smallest experiment to prove or disprove.
5. **Propose fix** — root cause in plain language + minimal code change at `file:line`.

If the root cause is architectural rather than a code defect, escalates rather than fixing.

### Forbidden

Shotgun debugging (changing multiple files hoping one works), fixing symptoms without a proven root cause, refactoring "while you're in there."

---

## Scribe

**Purpose:** Documentation agent. Produces phase-completion records, final task summaries, changelog entries, README updates, and docstrings. Works from diffs and summaries — does not explore the codebase independently.

**Tier:** Haiku | **Writes documentation** | **Directly invocable:** Yes

### Output Types

- **Phase record:** phase number, objective, files changed, tests added, verdict, notes.
- **Task summary:** all phases, verdict counts, changelog entry, follow-ups.
- **Changelog entry:** Conventional Commits format (`feat(scope): description`).
- **Docstring:** project's existing style, one-line summary, params, returns, exceptions.

### Rules

Terse and factual. No marketing adjectives. No emoji unless the surrounding file already uses them. No fabricating facts not in the provided inputs.
