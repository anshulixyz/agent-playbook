# Patterns — agent shapes and when each fits

Two families: **workflows** (LLM steps orchestrated by *code* on predefined paths)
and **agents** (the LLM *dynamically* directs its own control flow / tool use).
Prefer the simplest that works; escalate only when it doesn't.
Sources: Anthropic [Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents),
[12-Factor Agents](https://github.com/humanlayer/12-factor-agents).

## A. The five workflow patterns (Anthropic)
| Pattern | Shape | Use when | Watch out for |
|---|---|---|---|
| **Prompt chaining** | step → step → step | task splits into fixed, ordered subtasks | error compounding down the chain — validate between steps |
| **Routing** | classify → specialized handler | distinct input categories | misroute on ambiguous input; measure routing accuracy |
| **Parallelization** | fan-out → gather (sectioning or voting) | independent subtasks; latency matters; or N-vote for reliability | cost ×N; aggregation logic |
| **Orchestrator–workers** | planner delegates dynamic subtasks → synthesizes | subtasks unpredictable (multi-file edits, research) | planner errors cascade; bound the fan-out |
| **Evaluator–optimizer** | generate → critique → revise (loop) | clear eval signal; iterating improves output | infinite loops — cap iterations + a stop test |

## B. The structural shapes (your vocabulary)
| Shape | What it is | Best for | Failure mode |
|---|---|---|---|
| **DAG** (directed acyclic graph) | fixed nodes + edges, no cycles; explicit branches | most production agents — auditable, testable, parallelizable | rigidity if the real task needs loops |
| **State machine** | explicit states + transitions (a DAG that can loop, with guards) | precise control over branching, retries, **HITL gates**; resumable | state explosion if under-modeled |
| **Closed loop** | observe → decide → act → **observe the effect** → correct | the agent must react to outcomes of its own actions (control systems, ops) | runaway loops without a stop condition or a human gate |
| **Self-healing** | detect failure → recover automatically (retry, fallback, reroute, degrade) | unreliable deps (model outages, flaky APIs); high uptime needs | masking real bugs; silent degradation — *log every heal* |
| **Autonomous** | model drives end-to-end, minimal human gating | cheap-to-reverse, high-volume, **well-evaled** tasks | unverified action at scale — the most expensive failure |
| **Human-in-the-loop** | agent proposes; human approves/edits the risky step | money/safety/irreversible; low error tolerance | becoming a rubber stamp; make the human's call meaningful |

> **The agentic loop** underneath most of these: *LLM picks the next step →
> deterministic code executes it → append the result to context → repeat until a
> stop condition.* Own that loop, your prompts, and your context — don't outsource
> them to a black box (12-Factor Agents).

## C. Reliability/recovery sub-patterns (compose into any shape)
- **Retry with backoff** on transient errors (5xx/429); cap attempts.
- **Fallback chain** — try best option, fall back to the next on permanent failure
  (e.g. model A → model B → cheap deterministic backstop). A *circuit breaker* stops
  hammering a dead dependency.
- **Verification layer** — never trust raw model output: validate/coerce to a schema,
  **fail closed** (default to the safe outcome, e.g. "needs review", never toward an
  irreversible "approve").
- **Adversarial/independent checks** — a second, independent signal (rules, a
  different model, a non-LLM detector) cross-checks the primary; gaming one isn't enough.
- **Bounded concurrency + caching** — stay under provider rate limits; content-hash
  cache idempotent work so re-runs are ~free.

## D. Choosing — a quick heuristic
```
Is the path fixed?            → workflow (chaining / routing / parallel)
Path dynamic, bounded?        → orchestrator–workers or a DAG/state-machine
Needs to react to its acts?   → closed loop (+ a stop condition)
Unreliable dependencies?      → add self-healing (fallback + circuit breaker)
Irreversible/high-stakes?     → HITL gate on those actions (always)
Cheap-to-reverse + evaled?    → supervised-autonomous → autonomous
```

## E. Worked mapping — ClaimLens (one honest example)
- **Shape:** a **DAG**, not a dynamic agent — Tier-0 CV → perception (per image) →
  tools → judge → fuse → coerce. Chosen for auditability on a money decision.
- **Routing + parallelization:** scenario packs route by object type; per-image
  perception fans out.
- **Self-healing:** a capability **router** falls back model-A→model-B→mock on
  credit/auth/outage (this kept it alive when one provider's credits ran out).
- **HITL:** abstains to `not_enough_information` / `manual_review_required` on low
  confidence — *not* autonomous, by design, because the action (pay/deny) is costly.
- **Verification:** schema coercion **fails closed** to "needs info"; independent
  CLIP object-check cross-checks the VLM.
- **What it deliberately is *not*:** a closed loop or fully autonomous — there's no
  benefit to the agent acting on the world and re-observing here, and the stakes
  demand a human on the ambiguous cases.
</content>
