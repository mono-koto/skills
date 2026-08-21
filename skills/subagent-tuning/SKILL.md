---
name: subagent-tuning
description: Use whenever the harness can launch subagents and may let you choose a model, provider, or reasoning effort. Inspect the available controls, then route each subtask to the least costly capable agent.
---

# Tune Subagents

Choose the least costly agent that can complete and verify the work.

## Workflow

1. Inspect the harness before delegating. Confirm subagent support and which launch controls exist: provider, model, reasoning effort, agent template, tool profile, and concurrency. Do not claim to use an unavailable control.
2. Read reliable runtime state. In Pi, read `PI_PROVIDER`, `PI_MODEL`, and `PI_REASONING_LEVEL` from bash. Otherwise, use documented runtime state or the model catalog. Do not rely on the parent prompt, UI label, or a guess. Prefer the current provider unless another is needed for tools, modality, context, reliability, or an explicit user preference.
3. Check the local or harness model catalog. Confirm exact model IDs, supported efforts, modalities, tools, context limits, and provider availability. Query live provider metadata only when the catalog is incomplete or stale.
4. Define the task before choosing a model. State the outcome, allowed tools, inputs, expected output, constraints, and stopping condition. Tighten an underspecified prompt before raising capability or effort.
5. Use the routing gates below. Choose a model only after it passes the tool and modality gate. Pin the exact provider/model ID when reproducibility matters.
6. Run independent, bounded tasks in parallel when supported. Keep synthesis, approval, risky changes, and conflict resolution with a stronger agent.
7. Verify results against primary sources, repository state, or tests. Confidence is not evidence.

If subagents are unavailable, do the work directly. If model or effort overrides are unavailable, use the harness default and compensate with a better bounded prompt, staged review, and verification.

## Prompt contract

Every delegated task needs:

- The goal, relevant inputs, and exact question to answer or change to make.
- Allowed tools, repositories, files, and source-quality requirements.
- Output format, evidence to return, acceptance criteria, and a stopping condition.

## Routing gates

### 1. Tool and modality fit is a hard gate

Reject models that lack a required tool or modality. This includes browser access for web research, image/PDF understanding for visual evidence, repository tools for implementation, and parallel tool calls for wide retrieval.

Do not assign a text-only model to a visual task and expect prose to compensate.

### 2. Choose a provider for a reason

Stay on the current provider when it clears the tool/modality gate and offers a suitable tier. Change providers only for a concrete reason:

- The current provider lacks a required tool, modality, context window, or reliability property.
- A user explicitly requests a provider or model.
- The available catalog shows a distinct, better fit for the task, and the result can be verified.

Record the reason in the launch prompt or delegation note when it affects reproducibility or cost. Do not switch providers merely to chase a newer or more popular model.

### 3. Match the agent to the work

| Work | Prompt shape and autonomy | Default route |
| --- | --- | --- |
| Explore or read-only lookup | Narrow question, evidence and file paths required, no edits | Cheapest capable text/tool agent; low effort |
| Web research | Sources, recency, and comparison criteria specified | Browser-capable research agent; medium effort unless retrieval is trivial |
| Bounded implementation | Explicit files, behavior, tests, and stop condition | Coding-capable agent; medium effort, with review |
| Plan, review, or debugging | Ambiguous causes or tradeoffs; evidence must be weighed | Strong reasoning/coding agent; high effort for hard cases |
| Deep autonomous work or orchestration | Multiple dependent steps, delegation, recovery, and judgment | Flagship long-horizon agent; high when supervised, xhigh or max only for exceptional independent work |

Use agent templates or styles as inputs to this decision. A repository explorer benefits from fast retrieval and explicit evidence requirements. An implementation template needs coding tools and a narrow acceptance test. A planner or reviewer needs enough reasoning capacity to compare alternatives. An autonomous orchestrator needs reliable tool use, recovery, and context management.

### 4. Choose capability, then the lowest useful effort

Consider risk, autonomy, prompt boundedness, and consequence of error:

- Lower the tier for a read-only, reversible, well-bounded task with easy verification.
- Raise the tier for unclear requirements, non-local changes, security or data risk, hard debugging, and decisions that steer other agents.
- Use low effort for constrained, cheaply checked tasks. Use medium for routine coding and structured research. Use high for difficult reasoning or review. Reserve xhigh or max for exceptional deep independent work.
- Fan out cheap workers for independent extraction or reconnaissance. Give a stronger agent the synthesis, prioritization, and final judgment.
- Do not blindly copy the parent model or effort. The parent may be overpowered for lookup work or underpowered for a risky decision.
- Do not use popularity, leaderboard position, or vendor marketing as evidence of capability. Use observed task fit, supported tools and modalities, constraints, and verification results.

## Current provider examples

These examples are an **August 2026 snapshot**, not a permanent catalog. Exact IDs, capabilities, tool support, and effort levels change. Query the harness or local model catalog before launch, use live provider metadata only when needed, pin the exact available ID, and never invent an ID or pass an unsupported effort. Fall back to the closest available capability tier with a supported effort.

### OpenAI/Codex

| Model | Useful route | Effort |
| --- | --- | --- |
| `openai-codex/gpt-5.6-luna` | Targeted exploration and extraction; broad web research | low; medium for broad web research |
| `openai-codex/gpt-5.6-terra` | Everyday coding; difficult planning and debugging | medium; high for difficult work |
| `openai-codex/gpt-5.6-sol` | Complex or high-stakes reasoning | high; xhigh only for deep independent orchestration |

### OpenRouter

| Model | Distinct routing role | Effort |
| --- | --- | --- |
| `openrouter/qwen/qwen3.7-flash` | Very cheap mechanical lookup, extraction, and multimodal retrieval with easy verification | Omit the effort override; disable reasoning for pure extraction when supported |
| `openrouter/qwen/qwen3.8-27b` | Broad exploration, research, and multimodal synthesis at modest cost | low for lookup; medium for research |
| `openrouter/z-ai/glm-5.3` | Brent's preferred text-only option for complex software engineering and long-horizon agents; always reasons and exposes low/high/max | low for bounded work; high for serious coding/reasoning; max for truly deep orchestration |
| `openrouter/moonshotai/kimi-k2.7-code` | Cheaper coding-specialized work with parallel tool calls; always reasons but exposes no named effort levels in this snapshot | Omit the effort override |
| `openrouter/qwen/qwen3.8-max` | Qwen flagship for multimodal coding and reasoning with granular effort control | medium for routine work; high or xhigh for difficult independent work |
| `openrouter/moonshotai/kimi-k3` | Latest Moonshot flagship for multimodal, long-horizon coding and reasoning | low for bounded work; high for long-horizon work; max only for exceptional cases |

Do not infer that an OpenRouter model has browser, repository, image, or tool access from its base model description. Those capabilities are supplied by the harness and provider integration and must be inspected at launch time.

## Launch example

When the parent is `openai-codex/gpt-5.6-sol` at `high`, use `openai-codex/gpt-5.6-luna` at `low` for targeted exploration. Use Luna at `medium` for bounded web research when browser tools are available.

Keep deep orchestration and final judgment on Terra or Sol at `high`. Use Sol `xhigh` only for genuinely long-horizon, independently autonomous work.

## Fallbacks

- No subagent launcher: work directly and keep the same staged workflow.
- No model override: use the selected harness default; improve the task contract and verify the result.
- No effort override: use the model's default supported effort. Do not send an unsupported value.
- Unknown catalog entry: choose a known, tool-compatible model or omit the override. Do not guess aliases or capability claims.
- No browser or modality support: narrow the task to available evidence, ask for inputs, or route to a compatible harness.
- Model failure or weak output: first fix scope, inputs, and stopping conditions. Escalate capability only when the task still demands it.

## Common mistakes

- Claiming a launch, provider, effort, or tool setting that the harness does not expose.
- Reading model state from conversation context instead of reliable runtime state.
- Spending a stronger model on an ambiguous prompt instead of specifying deliverables and boundaries.
- Using text-only agents for images, PDFs, websites, or tools they cannot access.
- Giving cheap agents final authority over synthesis, security decisions, or risky edits.
- Treating max effort as a default instead of an exceptional orchestration setting.
- Copying the parent model and effort to every worker.
- Passing a guessed model ID or unsupported effort value.

## Sources

- OpenAI Codex model selection: <https://developers.openai.com/codex/models/>
- OpenAI GPT-5.6 model pages: <https://developers.openai.com/api/docs/models/gpt-5.6-luna>, <https://developers.openai.com/api/docs/models/gpt-5.6-terra>, and <https://developers.openai.com/api/docs/models/gpt-5.6-sol>
- OpenAI reasoning guidance: <https://developers.openai.com/api/docs/guides/reasoning>
- OpenRouter live models API: <https://openrouter.ai/api/v1/models>
- Z.ai GLM-5.3 documentation: <https://docs.z.ai/guides/llm/glm-5.3>
- Moonshot agent support: <https://platform.moonshot.ai/docs/guide/agent-support.en-US>
- Qwen3.8 post: <https://qwen.ai/blog?id=qwen3.8>
