# Architecture References — how the well-known agents are actually built

Grounded, cited references so you can copy *proven shapes* instead of inventing
them. Read these primary sources; the takeaways below are summaries, not gospel.

## Single-agent vs multi-agent — the central debate (read both)
Two famous posts with opposite titles, both right in their context:

- **Anthropic — [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)** —
  an **orchestrator–worker** design: a *Lead Researcher* plans and spawns **parallel
  subagents**, then a separate **citation pass**. Reported **+90.2%** over single-agent
  on breadth-heavy research — but uses **~15× the tokens** of a chat. *Use multi-agent
  when the task is genuinely parallel and the outcome is worth the spend.*
- **Cognition — [Don't Build Multi-Agents](https://cognition.ai/blog/dont-build-multi-agents)** —
  argues for **single-threaded** agents on **shared-context** tasks (most coding):
  naive multi-agent fragments context and produces inconsistent results. Coined the
  framing **context engineering**. *Use a single agent when subtasks share heavy context.*

**Synthesis** (also [LangChain: how & when to build multi-agent systems](https://www.langchain.com/blog/how-and-when-to-build-multi-agent-systems)):
*match architectural complexity to business value.* Multi-agent for parallel breadth
+ context that exceeds one window; single-thread for shared-context, dependent work.
Do the token math before committing.

## Reference shapes & primary sources
| System / idea | Shape worth copying | Source |
|---|---|---|
| **Anthropic Research** | orchestrator + parallel subagents + citation pass; lead writes a plan to memory | [anthropic.com](https://www.anthropic.com/engineering/multi-agent-research-system) |
| **Anthropic — Building Effective Agents** | the 5 workflow patterns + workflows-vs-agents; start simple | [anthropic.com](https://www.anthropic.com/engineering/building-effective-agents) |
| **Anthropic — Context engineering** | design the *information environment*, not just the prompt; curate the window | [anthropic.com](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) |
| **Anthropic — Harnesses for long-running agents** | the harness (memory, tools, recovery) is the engineering for long tasks | [anthropic.com](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) |
| **Cognition Devin** | "agent-native", **compound** system; gives the agent the *same tools a human engineer uses* | [cognition.ai](https://cognition.ai/blog/dont-build-multi-agents) |
| **12-Factor Agents** | own your prompt/context/control-flow; the agentic loop; HITL as a factor | [github](https://github.com/humanlayer/12-factor-agents) |
| **OWASP Secure Agent Playbook** | security plays/skills agents *run*, + securing agents (MCP review, injection testing, multi-agent threat modeling) | [github](https://github.com/OWASP/secure-agent-playbook) |

## What to copy vs. what's situational
- **Copy broadly:** the agentic loop (LLM picks step → code executes → append result),
  context curation, a verification/citation pass, retries/fallback, HITL gates.
- **Situational (decide per task):** multi-agent fan-out (only for parallel breadth),
  long-horizon autonomy (only when evaled + reversible), heavy frameworks (only when
  you need their machinery).
- **The honest meta-lesson** across all of them: *reliability is an
  orchestration/harness property.* The frontier labs spend their effort on the
  environment around the model — so should you.

> Use this file to justify an architecture choice in your `AGENT_SPEC.md` ("we use an
> orchestrator–worker like Anthropic Research because the subtasks are independent and
> the answer is worth ~10× tokens") rather than defaulting to the most complex option.
</content>
