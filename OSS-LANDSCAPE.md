# OSS Landscape — reuse, don't rebuild

Curated, source-cited map of mature options per competency (verified mid-2026).
**Read the repo before you write a blank file.** Positioning is summarized from the
linked comparisons; pick by your *dominant constraint*, not hype. Always confirm
licence + current status for your use.

> Anthropic's caution applies: many patterns are a few lines against the raw model
> API — use a framework when you genuinely need its machinery, and **understand its
> underlying code** either way. ([Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents))

## Foundations to read first (concepts, not deps)
- **Anthropic — Building Effective Agents** — the 5 workflow patterns + workflows-vs-agents. <https://www.anthropic.com/engineering/building-effective-agents>
- **12-Factor Agents (HumanLayer)** — reliability principles: own your prompt/context/control-flow, HITL as a factor, the agentic loop. <https://github.com/humanlayer/12-factor-agents>
- **OpenTelemetry GenAI semantic conventions** — standard spans/attrs for agent traces (vendor-neutral observability). <https://opentelemetry.io/docs/specs/semconv/gen-ai/>

## Orchestration frameworks
| Tool | Shape | Pick when (dominant constraint) | Link |
|---|---|---|---|
| **LangGraph** | graph / state machine, conditional edges | you need **control** — explicit branching, retries, HITL, checkpoints; lowest latency in tests | <https://github.com/langchain-ai/langgraph> |
| **OpenAI Agents SDK** | agent + explicit handoffs | a single agent calling 1–2 tools; vendor-native tracing/memory, low abstraction tax | <https://github.com/openai/openai-agents-python> |
| **Claude Agent SDK** | tool-use chain + sub-agents | Anthropic-native production agents | <https://docs.claude.com> |
| **CrewAI** | role-based "crews" | **team velocity** — fastest idea→prototype for multi-role agents | <https://github.com/crewAIInc/crewAI> |
| **AutoGen / AG2** | conversational GroupChat | conversational/research multi-agent (note: MS AutoGen v0.4+ vs community AG2) | <https://github.com/microsoft/autogen> · <https://github.com/ag2ai/ag2> |
| **smolagents (HF)** | code-writing agent, sandboxed | **simplest** single-agent loop | <https://github.com/huggingface/smolagents> |
| **Pydantic AI** | typed agent | **type safety** / structured outputs | <https://github.com/pydantic/pydantic-ai> |
| **LlamaIndex** | data-centric agents | RAG / **data layer** is the core | <https://github.com/run-llama/llama_index> |
| **Google ADK** | agent dev kit | Google/Gemini-centric stacks | <https://github.com/google/adk-python> |
| **Semantic Kernel** | enterprise SDK (.NET/Py) | Microsoft/enterprise stack | <https://github.com/microsoft/semantic-kernel> |
| **DSPy** | programmatic prompt/optimizer | you want to *compile/optimize* prompts vs hand-tune | <https://github.com/stanfordnlp/dspy> |

Comparisons: [LangChain's 2026 roundup](https://www.langchain.com/resources/ai-agent-frameworks) · [Firecrawl OSS frameworks](https://www.firecrawl.dev/blog/best-open-source-agent-frameworks) · [ZenML LangGraph alternatives](https://www.zenml.io/blog/langgraph-alternatives).

## Memory
| Tool | Strength | Link |
|---|---|---|
| **mem0** | framework-agnostic drop-in persistent memory | <https://github.com/mem0ai/mem0> |
| **Letta (MemGPT)** | tiered memory + full agent runtime (core/recall/archival) | <https://github.com/letta-ai/letta> |
| **Zep** | strong **temporal** retrieval | <https://github.com/getzep/zep> |
| **Cognee** | unstructured-document ingestion → knowledge | <https://github.com/topoteretes/cognee> |

Benchmarks differ by task (personalization vs temporal vs long-horizon) — see [mem0-vs-Letta](https://vectorize.io/articles/mem0-vs-letta) / [Mem0 vs Zep vs Letta vs Cognee](https://particula.tech/blog/agent-memory-frameworks-tested-mem0-zep-letta-cognee-2026). Match the tool to *your* memory access pattern.

## Evals + observability (do not hand-roll)
| Tool | Strength | Link |
|---|---|---|
| **MLflow** | OSS, broad (tracing + eval + prompt + governance), huge adoption | <https://github.com/mlflow/mlflow> |
| **Langfuse** | OSS, self-hostable tracing + evals | <https://github.com/langfuse/langfuse> |
| **Arize Phoenix** | OSS tracing/eval, OTel-native | <https://github.com/Arize-ai/phoenix> |
| **Braintrust** | eval + experimentation + monitoring | <https://www.braintrust.dev> |
| **AgentOps** | agent-session lifecycle + tool-call replay | <https://github.com/AgentOps-AI/agentops> |
| **LangSmith** | best if all-in on LangChain/Graph | <https://www.langchain.com/langsmith> |
| **DeepEval** · **Ragas** · **promptfoo** | eval/test harnesses (assertions, RAG metrics, red-team) | <https://github.com/confident-ai/deepeval> · <https://github.com/explodinggradients/ragas> · <https://github.com/promptfoo/promptfoo> |

(LLM-observability is a real, fast-growing category — [overview](https://mlflow.org/articles/top-llm-observability-tools-in-2026-a-pro-guide/).)

## Durable execution / retries / long-running
| Tool | Strength | Link |
|---|---|---|
| **Temporal** | durable workflows — crash-safe, resumable, built-in retries/timeouts | <https://github.com/temporalio/temporal> |
| **Inngest** | event-driven durable functions/steps | <https://github.com/inngest/inngest> |
| **Restate** | durable execution / resilient handlers | <https://github.com/restatedev/restate> |

Use these when an agent run is long, must survive process death, or needs exactly-once-ish step semantics.

## Human-in-the-loop
| Tool | Strength | Link |
|---|---|---|
| **HumanLayer** | approval/escalation/human-contact as tool calls | <https://github.com/humanlayer/humanlayer> |
| **LangGraph interrupts** | pause for human input at a node, then resume | <https://github.com/langchain-ai/langgraph> |

## Provenance / safety (domain-specific; relevant if you verify media)
- **C2PA / Content Credentials** — cryptographic content provenance standard. <https://c2pa.org>
- Plus your own input-security layer: path/format/size guards, prompt-injection
  handling, fail-closed output coercion (build the *thin* domain glue yourself).

---
**Selection heuristic:** start from your dominant constraint → *control* = LangGraph ·
*Anthropic-native* = Claude Agent SDK · *velocity* = CrewAI · *simplest* = smolagents ·
*type-safety* = Pydantic AI · *data/RAG* = LlamaIndex · *durability* = Temporal ·
*evals* = MLflow/Langfuse. When unsure, prototype the smallest version against the raw
API first, then adopt the framework whose machinery you actually needed.
</content>
