# Agent Types — what you can build well with this playbook

"Agent" is a broad word. This maps the common **kinds** to the right
[pattern](./PATTERNS.md), autonomy level, and the guardrail that matters most — so
the playbook applies whether you're building a support copilot or a research swarm.

| Type | What it does | Default shape · autonomy | Watch most |
|---|---|---|---|
| **Conversational assistant / copilot** | Q&A, support chat, draft replies | routing + RAG · HITL/supervised | injection via user text; hallucinated answers → cite sources |
| **RAG / knowledge agent** | answer over your docs/KB | workflow (retrieve→answer) · supervised | retrieval quality; stale/contaminated context |
| **Tool / function-calling agent** | takes actions in apps (CRM, calendar, payments) | single-LLM + tools · HITL on writes | **least-privilege tools**; irreversible actions gated |
| **Workflow / back-office automation** | run a known multi-step process | DAG / state-machine · supervised-auto | idempotent, resumable steps; partial-failure recovery |
| **Verification / judgment agent** (e.g. **ClaimLens**) | decide/adjudicate from evidence | DAG · **HITL, abstains** | **fail-closed**; image/data as source of truth, not user words |
| **Research / deep-research agent** | gather + synthesize across many sources | **orchestrator–workers** (parallel subagents) · supervised | cost (≈10–15× tokens); citation/grounding pass |
| **Coding agent** (Devin / Claude-Code style) | write, run, fix code | long-horizon **single-threaded harness** + context engineering · HITL on merge | context management; sandbox; never auto-merge |
| **Computer-use / browser agent** | drive a GUI/website | state-machine · HITL on consequential actions | **sandbox + allowlists**; injection from page content |
| **Data-extraction / pipeline agent** | structure messy inputs at scale | workflow, deterministic · autonomous (cheap-to-redo) | schema validation; cost at volume; drift |
| **Multi-agent system** | many specialists coordinated | orchestrator + workers OR a supervised loop · varies | only when work is *parallelizable*; see [ARCHITECTURE-REFERENCES.md](./ARCHITECTURE-REFERENCES.md) |

## How to choose (don't over-build)
- **Most "agents" are workflows.** If the path is known, wire it in code — cheaper,
  testable, safer. Reach for a dynamic/multi-agent design only when the task is
  genuinely open-ended or parallelizable.
- **Stakes set autonomy.** Money/safety/irreversible → HITL with a hard gate. Cheap
  and reversible → supervised-autonomous → autonomous *after* it's evaled.
- **Single-thread before swarm.** Shared-context tasks (most coding) do better
  single-threaded; multi-agent shines on independent, parallel breadth (research).
  ([ARCHITECTURE-REFERENCES.md](./ARCHITECTURE-REFERENCES.md))

## Worked example — ClaimLens (a verification/judgment agent)
A DAG, HITL (abstains to `needs_review`), fail-closed, image-is-truth — *not* a
multi-agent system, because the work isn't parallelizable and the stakes demand a
human on the ambiguous cases. The playbook would steer most claims/insurance,
returns/e-commerce, and document-verification agents to this same quadrant.
</content>
