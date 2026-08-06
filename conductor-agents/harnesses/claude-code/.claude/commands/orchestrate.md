Execute the full Conductor orchestration lifecycle for: $ARGUMENTS

The Conductor will:
1. Restate the task and ask clarifying questions.
2. Invoke Scout for codebase discovery.
3. Invoke Architect to produce a phased plan.
4. Present the plan for your approval.
5. Execute each phase: Implementer → Reviewer → conditional Security Auditor.
6. Stop after each phase for you to commit.
7. Invoke Scribe for final documentation when all phases are complete.

Paste a task description after `/orchestrate`. Be specific about scope, constraints, and acceptance criteria. Vague requests will be bounced back for clarification.
