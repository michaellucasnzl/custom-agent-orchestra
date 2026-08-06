# Model Registry

Model assignments per agent, per harness. This is the single source of truth — edit agent files directly in each harness directory to change models.

## Assignment Table

| Agent | Tier | Copilot | Claude Code | OpenCode | Rationale |
|---|---|---|---|---|---|
| Conductor | Sonnet | `Claude Sonnet 4.6` | `sonnet` | `venice/deepseek-pro-v4` | Resident all session; routing not reasoning |
| Scout | Haiku | `Claude Haiku 4.5` | `haiku` | `venice/deepseek-v4-flash` | High-token, low-judgement summarisation |
| Architect | Opus | `Claude Opus 4.8` | `opus` | `venice/glm-5-2` | Plan quality compounds downstream |
| Implementer | Haiku | `Claude Haiku 4.5` | `haiku` | `venice/deepseek-v4-flash` | Highest call volume; tightly constrained |
| Reviewer | Sonnet | `Claude Sonnet 4.6` | `sonnet` | `venice/deepseek-pro-v4` | Judgement on small diffs |
| Security Auditor | Opus | `Claude Opus 4.8` | `opus` | `venice/glm-5-2` | Adversarial reasoning, runs rarely |
| Debugger | Sonnet | `Claude Sonnet 4.6` | `sonnet` | `venice/deepseek-pro-v4` | Hypothesis-driven, moderate context |
| Scribe | Haiku | `Claude Haiku 4.5` | `haiku` | `venice/deepseek-v4-flash` | Pure text generation |

## Cost Profiles

### Budget

Architect on Sonnet, everything else on Haiku. Roughly 70–80% cheaper than Balanced. Sufficient for well-scoped CRUD work, routine refactors, and documentation tasks.

### Balanced (Default)

The assignment table above. Premium tokens on decisions; cheap tokens on execution.

### Quality

Conductor, Architect, Reviewer, and Security Auditor on Opus; Implementer on Sonnet. Use for greenfield architecture, migrations, or code touching money or auth.

## Governing Principle

**Spend premium tokens on decisions, cheap tokens on execution.** Planning errors are expensive — a bad plan wastes every downstream call. Typing errors are cheap — they are caught by the Reviewer, test suite, and compiler.

## Model Drift

Model identifiers can go stale when providers rename models. A stale string silently falls back to whatever the model picker has selected. This is dangerous on:

- **Architect:** silently plans on a weak model → bad plans → compounding downstream waste.
- **Security Auditor:** silently reports "no findings" because a weak model missed the vulnerability.

Mitigations:

- **Claude Code:** The `opus`/`sonnet`/`haiku` aliases always resolve to the current model — immune to drift.
- **Copilot:** In VS Code, delete past the `:` on the `model` line and re-type `:` to trigger autocomplete against the live model list.
- **Copilot fallback (opt-in):** Copilot accepts an array on the `model` field. Convert just Architect and Security Auditor to `['Claude Opus 4.8', 'Claude Sonnet 4.6']` for belt-and-braces.

## Benchmarking Cheaper Models

Pick three representative past PRs. Run the Implementer at Haiku and at Sonnet against the same Architect plan and compare Reviewer verdict rates. If Haiku's `NEEDS_REVISION` rate is under ~25%, the savings are real.

## Premium-Request Warning

Opus-tier models consume premium requests at a significantly higher multiplier. The Architect is invoked **once per task, not per phase**. If you find yourself re-planning every phase, your phases are too large — split them instead.
