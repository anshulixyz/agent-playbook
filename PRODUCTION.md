# Production — operating an agent, not just building one

The rest of Groundwork is *"build the agent right."* This is *"run it in production."*
Vendor-neutral; it names the decisions and points at tools, it is not a Kubernetes
tutorial. (Stage 11 of the [learning path](./LEARNING-PATH.md).)

## 1. Serving the model
- **Managed APIs** (Anthropic/OpenAI/Google) — default; no infra, built-in scaling,
  prompt-caching/batch. Start here.
- **Self-host** with [vLLM](https://github.com/vllm-project/vllm) or
  [SGLang](https://github.com/sgl-project/sglang) — when you need open-weights, data
  residency, cost-at-scale, or latency control. Adds GPU ops you now own.
- **Decide by constraint:** data/compliance or volume economics → self-host; speed to
  market + elasticity → managed. (Your capability **router** can mix both — see
  [PATTERNS](./PATTERNS.md) self-healing/fallback.)

## 2. Scaling & runtime shape
- **Stateless workers + externalized state** (DB / cache / vector store) so you can
  scale horizontally (k8s, serverless, queues).
- **Bounded concurrency + a queue** to stay under provider TPM/RPM; backpressure, not
  drop.
- **Long-running / multi-step?** use **durable execution** (Temporal / Inngest /
  Restate, see [OSS-LANDSCAPE](./OSS-LANDSCAPE.md)) so a run survives a crash and
  resumes — don't hold a 10-minute agent loop in one web request.

## 3. CI/CD for agents (the part that's different)
- **Eval-gated deploys:** your eval set ([BENCHMARKING](./BENCHMARKING.md)) is the test
  suite — a regression fails the build, same as a unit test.
- **Pin versions:** prompt version, model id, tool schemas — a prompt change is a code
  change; review + version it.
- **Canary / shadow / rollback:** release to a slice, compare quality+cost, roll back
  fast. **Shadow-run** a new model/judge on live traffic (no user impact) before
  switching.
- **Feature-flag the model tier** — this is the **cost/accuracy dial** in prod
  (cheap-judge vs strong-judge) flipped without a redeploy.

## 4. Reliability in production
- Retry-with-backoff + circuit breaker + **fallback chain** (model A→B→deterministic),
  every heal logged — see [PLAYBOOK](./PLAYBOOK.md) #5.
- **Idempotency** on actions (so retries/redeliveries don't double-act).
- **Graceful degradation:** always return a *valid* result (even a "needs review") under
  partial failure; never 500 a money decision.

## 5. Cost & latency at scale
- Caching (exact + semantic), batching, prompt-caching APIs.
- **Budget caps + a kill switch**; **per-tenant metering**; alert on cost spikes.
- SLOs: **p95 latency**, availability, cost/request, error budget.

## 6. Observability & alerting (run it like a service)
- Traces + **cost dashboards** + latency monitoring ([OSS-LANDSCAPE](./OSS-LANDSCAPE.md):
  Langfuse/Phoenix/MLflow/AgentOps; OpenTelemetry GenAI semconv).
- **Quality/drift monitoring** in prod — sampled live eval + human spot-checks; alert
  when accuracy/confidence/override-rate drifts.
- **Security alerting:** attack-success-rate, anomalous tool calls, egress
  ([GUARDRAILS-AND-SECURITY](./GUARDRAILS-AND-SECURITY.md)).

## 7. Security & compliance in prod
- Secrets in a manager (not env files on disk); **PII redaction** in logs/traces;
  **audit trail** of decisions + tool calls; data **retention/deletion** policy.
- For SOC 2 / regulated work: the per-decision trace is your audit backbone; auth +
  least-privilege + monitoring are the controls. (Most of this is *organizational*, not
  architectural.)

## What to measure (prod SLIs)
`availability · p95 latency · cost/request · eval-regression (gated) · drift ·
attack-success-rate · HITL/override rate · error budget burn`

## Pre-deploy gate (add to [CHECKLIST](./CHECKLIST.md))
- [ ] Eval-gated CI; prompt/model/tool versions pinned.
- [ ] Canary + rollback path; new models shadow-tested first.
- [ ] Durable execution for long runs; idempotent actions.
- [ ] Budget caps + kill switch; cost + latency dashboards + alerts.
- [ ] Secrets manager; PII redaction; audit trail; retention policy.

> Honest note: this chapter was previously **out of scope** for Groundwork (the build
> playbook). It's here as a *reference for the operating decisions* — for the actual
> infra (k8s manifests, vLLM tuning), follow each tool's docs.
</content>
