# Copilot harness

Ten custom agents for GitHub Copilot in VS Code and Visual Studio, coordinated by the Conductor.
The lifecycle matches the OpenCode harness exactly: discovery → plan → **mandatory plan-approval
gate** → implementation phases → one end-of-task verification gate → one end-of-task review gate
→ conditional security audit → summary.

## Install

Workspace:

```bash
cp -r harnesses/copilot/.github .github/
cp -r harnesses/copilot/.vscode .vscode/
```

User scope (all projects): copy `.github/agents/*.agent.md` to `~/.copilot/agents/`
(VS Code) or `%USERPROFILE%\.github\agents\` (Visual Studio).

Reload the window, then pick **Conductor** in the agent picker. Every other agent is
subagent-only except Debugger, Security Auditor, and Scribe, which stay user-invocable.

## Default agent

`.vscode/settings.json` sets `"chat.newSession.defaultMode": "conductor"` so new VS Code chat
sessions start in the orchestrator. Visual Studio has no equivalent setting — select
Conductor once from the agent picker; the choice is remembered per workspace.

## Models

Each agent pins a Copilot model in its frontmatter, mapped to the tier its OpenCode
counterpart used (OpenAI/Anthropic only, so no Gemini, Kimi, Grok, or BYOK providers):

| Tier | OpenCode model it replaces | Copilot model | Agents |
|---|---|---|---|
| High-cost | `kimi-k3.1-pro` | `Claude Opus 5.5 (copilot)` | Architect, Security Auditor |
| Mid-tier | `glm-5.2`, `glm-5.3` | `Claude Sonnet 5.5 (copilot)` | Conductor, Reviewer, Debugger |
| Fast/cheap | `deepseek-v4.1-flash*`, `glm-5.3-flash` | `Claude Haiku 5.5 (copilot)` | Scout, Image Reader, Implementer, Verifier, Scribe |

Model names are in the `Display Name (vendor)` form Copilot uses for agent frontmatter.
Nothing else needs configuring — these ship with Copilot, so no BYOK setup is required.

Two notes:

- These are the newest GA Claude models at the time of writing. Copilot retires models on a
  rolling basis (Claude Opus 5, Claude Opus 4.8, and GPT-5.6 Terra are deprecated from
  16 November 2026), so if a model disappears from your picker, update the `model:` line in
  the affected files.
- `Claude Haiku 5.5` is used for the Image Reader because it accepts image input. If you swap
  that agent to a text-only model, image reads will fail — the agent reports this rather than
  guessing from filenames.

To rebalance cost, change a single agent's `model:` line, or delete it so the agent inherits
whatever model is selected in the picker. OpenAI equivalents by tier are
`GPT-6.1 Sol` (high), `GPT-6 Sol` (mid), and `GPT-5.6 Luna` (fast).

## Behaviour notes specific to this harness

- **No background subagents.** Copilot returns sub-agent results inside the same turn.
  The Conductor therefore batches independent invocations in a single turn (up to 3) instead
  of `background: true`, and never batches dependent work.
- **Plan approval is a real stop.** The Conductor waits for an explicit approval message
  before any implementation phase runs.
- **Tool boundaries come from `tools:`** in each agent file: Scout/Architect are read-only,
  Implementer/Debugger can edit and run terminals, Verifier is execute-only, Reviewer and
  Security Auditor are read-only with terminal inspection, Scribe can edit documentation.
- **Model tiers are per-file**, not global: change one agent's `model:` line to move it to a
  stronger or cheaper Copilot model without touching the others.
