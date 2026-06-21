# Playbook — the 8 competencies (practice · measure · reuse)

The skills an agent engineer is on the hook for (per the widely-shared list, and
borne out by real builds). For each: **what good looks like**, **what to measure**,
and **what to reuse** instead of hand-rolling. The "lesson" lines are from an actual
24-hour build (ClaimLens) — including its mistakes.

> Orchestration failures > model failures. Seven of the eight below are
> orchestration. Budget your time accordingly.

## 1. Tool / function-calling design
- **Practice:** few, well-named, typed tools; one clear job each; validate inputs;
  return structured results + errors (not prose). Treat tool output as *untrusted*.
  Keep the tool surface small — more tools = more misfires.
- **Measure:** tool-call success rate, wrong-tool rate, arg-validation failures,
  schema-conformance of outputs.
- **Reuse:** your framework's tool/`@tool` decorators + JSON-schema validation
  (Pydantic). MCP for cross-app tools.

## 2. Planning / workflow orchestration
- **Practice:** make control flow **explicit and deterministic** where you can
  (DAG/state machine); reserve model-driven planning for genuinely dynamic subtasks.
  Pick the simplest [pattern](./PATTERNS.md) that fits.
- **Measure:** end-to-end task success, step-level success, replan/iteration count.
- **Reuse:** LangGraph (state machines/DAGs, control + HITL), OpenAI Agents SDK /
  Claude Agent SDK (simple tool loops), CrewAI (role teams), smolagents (code
  agents). See [OSS-LANDSCAPE.md](./OSS-LANDSCAPE.md).
- **Lesson:** the fixed DAG was the right call for a money decision — auditable and
  testable beats clever-and-dynamic when correctness matters.

## 3. Memory / context management
- **Practice:** the context window is RAM — *curate* it. Distill, don't dump (send
  compact evidence, not raw blobs). Tag provenance/trust on each block. Persist
  durable facts *outside* the window. "Context engineering > prompt engineering."
- **Measure:** tokens in/out per task, context-window utilization, retrieval
  hit-rate, cost attributable to context.
- **Reuse:** mem0 (framework-agnostic), Letta/MemGPT (tiered runtime), Zep
  (temporal), Cognee (docs). Don't build a memory store from scratch.

## 4. State machines / multi-step execution
- **Practice:** model states + transitions explicitly with guards; make steps
  **idempotent** and the run **resumable** (persist state between steps). Cap loops.
- **Measure:** completion rate, stuck/loop rate, resume-after-failure success.
- **Reuse:** LangGraph for in-process state; **durable execution** engines
  (Temporal, Inngest, Restate) when runs are long/critical and must survive crashes.

## 5. Retry / fallback / recovery (self-healing)
- **Practice:** retry-with-backoff on transient errors; a **fallback chain**
  (provider A → B → cheap/deterministic backstop) with a **circuit breaker**;
  **fail closed** to the safe outcome; **log every heal** (no silent degradation).
- **Measure:** retry rate, fallback-trigger rate, % runs that still return a *valid*
  result under failure, mean-time-to-recover.
- **Reuse:** provider SDK retries; durable-execution engines for retry semantics;
  your router for model fallback.
- **Lesson:** a capability **router with fallback kept the system running when one
  provider's credits ran out mid-build** — recovery logic is not optional polish.

## 6. Agent evals / reliability testing
- **Practice:** an eval set from day one; multi-strategy comparison; **regression-gate
  in CI**; be **honest about variance and small n**. Test at the **real boundary**
  (HTTP/tool I/O), not just internals.
- **Measure:** task accuracy / pass-rate, per-field accuracy, confusion matrix,
  run-to-run variance, calibration (ECE) for confidence.
- **Reuse:** MLflow / Langfuse / Braintrust / Arize Phoenix (tracing + eval),
  DeepEval / Ragas / promptfoo (eval harnesses), AgentOps (agent-session replay).
- **Lesson (the expensive one):** a green test suite **hid a P0** — routes 422'd in
  reality because tests only checked route *paths* and the e2e test bypassed HTTP. A
  staff-level review caught it. **If it's an HTTP agent, a test must POST to it.**
  Also: n=20 with selection-on-test is *directional* — say so, don't headline it.

## 7. Cost / latency optimization
- **Practice:** tier the work (free deterministic → cheap model → strong model only
  for the real decision); **cache** idempotent calls (content-hash); bounded
  concurrency; one expensive call where it changes the outcome. Make cost a
  **dial** (cheap vs accurate config), not a fixed property.
- **Measure:** cost per task, p50/p95 latency, cache-hit rate, $/accuracy-point.
- **Reuse:** provider prompt-caching/batch APIs; semantic/exact caches; the eval
  harness's cost tracking.
- **Lesson:** same pipeline ran **0.80 acc @ ~$0.63 (strong judge)** vs **0.60 @
  ~$0.06 (cheap judge)** — naming the trade-off is more useful than a single number.

## 8. Human-in-the-loop patterns
- **Practice:** make "escalate to human" a **first-class output**; define *which*
  actions always need approval; surface confidence + a grounded rationale so the
  human decides fast; never let the human become a rubber stamp.
- **Measure:** escalation rate, human-override rate, time-to-decision, post-HITL
  error rate.
- **Reuse:** HumanLayer (approval/escalation as tool calls); your framework's
  interrupt/checkpoint (LangGraph interrupts).
- **Lesson:** abstaining (`needs_review`) on low confidence is a feature, not a miss —
  it's how a money decision stays safe.

---

## Process lessons (the meta-competency, from a real build)
These aren't in the eight, but they decided whether the build held together:

- **Plan first, share, then build.** A written plan + a few clarifying questions up
  front saved a wrong-folder/wrong-scope start.
- **Fan out, then converge.** Parallel workers on **disjoint files**; the orchestrator
  does the shared-file glue + integration. Independent reviewers catch what builders
  miss.
- **Guardrail with CI, not vibes.** Lint + tests + an output-contract check + a
  **docs-sync gate** (docs must change with the code) + a static check on the UI.
- **Keep an honest backlog.** A prioritized ROADMAP of what's thin (cost, evals,
  guardrails, provenance) beats pretending it's done.
- **Scoped commits.** `git add -A` once swept an unrelated UI redesign into a docs
  commit — stage deliberately; keep commits coherent.
- **Secrets hygiene.** Keys in a gitignored `.env`; verify the shipped artifact/zip
  contains **no** real secrets before sending it.
- **Log the trail.** A per-turn transcript made decisions auditable and made *this
  document* possible.
</content>
