# Token Economy: Optimizing GitHub Copilot Chat & Agents under Usage-Based Billing

A practical guide to controlling spend when using GitHub Copilot Chat and agents under **usage-based billing (UBB)** — the billing model GitHub is moving Copilot to on **June 1, 2026** ([source](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)).

> **TL;DR** — Under UBB, you pay for **tokens** (input + output + cached) converted to **GitHub AI Credits** (1 credit = $0.01). Pick a cheaper model, keep prompts lean, ask focused questions to shorten outputs, prune history, scope agents, and let tools do deterministic work instead of the LLM.

---

## Table of Contents

- [Token Economy: Optimizing GitHub Copilot Chat \& Agents under Usage-Based Billing](#token-economy-optimizing-github-copilot-chat--agents-under-usage-based-billing)
  - [Table of Contents](#table-of-contents)
  - [How UBB Bills Copilot](#how-ubb-bills-copilot)
  - [Why Quality Beats Cost (Read This First)](#why-quality-beats-cost-read-this-first)
  - [Foundations: How Agents \& Context Windows Work](#foundations-how-agents--context-windows-work)
  - [Quick Wins (Start Here)](#quick-wins-start-here)
  - [Key Cost Drivers](#key-cost-drivers)
  - [Strategy 1 — Trim Prompt Overhead](#strategy-1--trim-prompt-overhead)
  - [Strategy 2 — Manage Memory \& Context](#strategy-2--manage-memory--context)
  - [Strategy 3 — Ask Focused Questions (Shrink Output)](#strategy-3--ask-focused-questions-shrink-output)
    - [Example](#example)
  - [Strategy 4 — Orchestrate Tools \& Agents Efficiently](#strategy-4--orchestrate-tools--agents-efficiently)
    - [Example](#example-1)
  - [Strategy 5 — Pick the Right Model \& Build Good Habits](#strategy-5--pick-the-right-model--build-good-habits)
  - [Real-World Savings Examples](#real-world-savings-examples)
  - [Walkthroughs (How-To Docs)](#walkthroughs-how-to-docs)
  - [Checklist for Engineers](#checklist-for-engineers)
  - [Tools and Extensions](#tools-and-extensions)

---

## How UBB Bills Copilot

Starting **June 1, 2026**, every Copilot interaction in Chat, CLI, cloud agent, Spaces, Spark, and third-party coding agents is metered by **tokens** — input + output + cached — priced per model and converted to **GitHub AI Credits** (1 credit = $0.01 USD).

Each Copilot plan ships with a **monthly AI Credits allowance**. Paid individual plans split it into **base credits** (fixed) plus a **flex allotment** (a variable top-up GitHub tunes as model economics change):

| Plan               | Monthly AI Credits | Notes                               |
| ------------------ | ------------------ | ----------------------------------- |
| Copilot Free       | Limited            | Plus 2,000 inline suggestions/month |
| Copilot Pro        | 1,500              | 1,000 base + 500 flex ($10/mo)      |
| Copilot Pro+       | 7,000              | 3,900 base + 3,100 flex ($39/mo)    |
| Copilot Max        | 20,000             | 10,000 base + 10,000 flex ($100/mo) |
| Copilot Business   | 1,900 per license  | Pooled at billing-entity level      |
| Copilot Enterprise | 3,900 per license  | Pooled at billing-entity level      |

When the allowance is exhausted, additional usage is billed at per-token rates (1 credit = $0.01), subject to budget policy.

**Code completions and next edit suggestions are not billed in AI Credits** — they stay unlimited on paid plans.

> **Was it different before?** Yes. Before June 1, 2026, Copilot used **request-based billing** — each prompt counted as one *premium request* multiplied by a model multiplier. Under UBB, multipliers go away (except for legacy annual subscribers who stay on request-based billing), and **the actual token volume** of every interaction is what determines cost.

Sources: [Usage-based billing for individuals](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals) · [Usage-based billing for organizations and enterprises](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises) · [Models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)

---

## Why Quality Beats Cost (Read This First)

The instinct under UBB is to ask *"how do we spend fewer tokens?"* That's the wrong first question. Optimizing cost while an agent's output value is near zero is wasted effort — and the worst token waste (bloated context, sprawling chats, vague prompts that need re-runs) is a **quality** problem first and a cost problem second.

- **Fix quality and spend drops with it.** Trimming irrelevant context is the single biggest lever for *both* better answers and lower bills.
- **Errors compound.** LLMs are non-deterministic, so every step in an agent run has some chance of being wrong — and across a multi-step task those odds *multiply*, they don't average. A run that's right only if all 50 steps are right has a success rate of (step accuracy)⁵⁰: at **99%/step that's ~60%**, and at **95%/step it collapses to ~8%**. So a tiny drop in per-step quality tanks the whole run, and small quality gains pay off hugely. Every miss also re-bills tokens plus fixes, reviews, and human time.
- **Effort should scale with maturity.** If you run a handful of agents a day, the [Quick Wins](#quick-wins-start-here) are enough. If you orchestrate dozens or hundreds, every percent compounds across the fleet.

> **Instead of counting tokens, make every token count.** Send fewer, higher-accuracy runs and the fuel savings follow.

📘 Walkthroughs: [Think in Quality Economics](docs/quality-economics.md) · [Add Deterministic Guardrails](docs/deterministic-guardrails.md)

---

## Foundations: How Agents & Context Windows Work

You can't optimize what you don't understand. A few non-obvious mechanics drive most token waste:

- **Agents are stateless.** An agent (the *harness* — VS Code Chat, Copilot CLI, cloud agent, Claude Code, Codex) talks to a stateless LLM on your behalf, many times per task. "Having a conversation" means **re-sending the entire ordered history on every turn** — so tokens compound with each loop.
- **Models don't read the window evenly.** They bias toward the **beginning** and **end** and discount the **middle** ("context rot"). Below ~50% full you get *Lost in the Middle*; above ~50% **recency bias** kicks in and the model starts forgetting your original goal.
- **Practical rules:** use a **new context window per distinct task**, and try to keep working sessions under **~60–70%** of the window.

📘 Walkthroughs: [How Context Windows Work](docs/context-windows.md) · [Agent Configs Explained](docs/agent-configs.md)

---

## Quick Wins (Start Here)

If you only do five things, do these — they raise quality *and* cut tokens at the same time:

1. **Right-size the model.** Use the cheapest model that meets the bar (prices span ~25×); save reasoning models for planning and hard problems, not typo fixes. → [How](docs/choose-model.md)
2. **Give clear, focused guidance.** Precise prompts with explicit scope and stop signals steer the agent right the first time and keep output short (output tokens cost ~4–8× input). → [How](docs/focused-questions.md)
3. **Work in phases and start fresh.** Split work into Research → Plan → Implement with a new context window per task, so stale context can't drift or re-bill on every turn. → [How](docs/research-plan-implement.md)
4. **Add deterministic guardrails.** Tests, linters, and scanners catch agent errors before they compound — turning a future incident into an early, cheap correction. → [How](docs/deterministic-guardrails.md)
5. **Keep a concise, human-written `copilot-instructions.md`.** Capture your non-negotiables and recurring agent misses, and tell it to be concise to trim output. → [How](docs/custom-instructions.md)

---

## Key Cost Drivers

| Driver            | What It Is                                                         | Why It Costs in UBB                                                                           |
| ----------------- | ------------------------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| **Model choice**  | Which model you picked (or Auto)                                   | Each model has its own per-token price for input, output, and cached tokens                   |
| **Output length** | How verbose the response is                                        | Output tokens are typically the most expensive bucket (~4–8× input)                           |
| **Input length**  | Prompt + attached context + chat history                           | Every turn re-sends the running conversation as input tokens                                  |
| **Cache misses**  | Model switches, reasoning/tool changes, or resuming a cold session | Cached input bills at ~10%; breaking the cache re-sends the whole context at full input price |
| **Agent loops**   | Multiple under-the-hood model calls per task                       | Long agentic sessions generate many tokens; complex agent runs can dwarf a single chat        |
| **Tool overload** | Too many tools enabled                                             | You can hit VS Code's 128-tools-per-request limit and lose time (and tokens) to retries       |

---

## Strategy 1 — Trim Prompt Overhead

**Goal:** shrink the **input tokens** that get re-sent on every turn.

- **Externalize persistent instructions.** Move large style guides, role definitions, or boilerplate out of always-on `copilot-instructions.md` and into on-demand prompt files (`*.prompt.md`) or skills (`SKILL.md`). Always-on instructions are sent on every turn; on-demand prompts and skills are only sent when invoked.
- **Inject context dynamically.** Attach only the file or snippet the question is actually about. Don't paste an entire library when the question is about one function.
- **Be concise in your wording.** Replace multi-sentence instructions with one or two crisp sentences. Shorter prompts tend to produce shorter, more focused responses — and output tokens are the most expensive bucket.

📘 Walkthrough: [Externalize Custom Instructions](docs/custom-instructions.md)

---

## Strategy 2 — Manage Memory & Context

**Goal:** keep the running conversation small so each new reply re-sends fewer input tokens.

- **Prune history.** Use `/clear` or start a new chat when you switch tasks. Every prior turn becomes input tokens on the next reply.
- **Reset when the topic changes.** A focused 5-turn chat is cheaper than a sprawling 50-turn one carrying yesterday's work.
- **Summarize long histories.** Use `/compact` (or ask the model for a 5-bullet summary, copy it, start a new chat, paste it). You keep the decisions without re-sending the full transcript.
- **Reference, don't repaste.** If you already shared a schema or file earlier in the chat, refer back to it ("using the schema from earlier…") rather than pasting it again.
- **Keep the cache warm.** Cached context bills at ~10% of input — but switching models mid-session, changing reasoning/tools, or resuming a cold session forces a full-price rebuild. Pick one model per session (or use `Auto`, which switches only at cache boundaries) and configure reasoning/tools up front.

📘 Walkthroughs: [Prune Chat History](docs/prune-history.md) · [Manage Context Attachments](docs/manage-context.md) · [Preserve the Cache](docs/preserve-cache.md)

---

## Strategy 3 — Ask Focused Questions (Shrink Output)

**Goal:** small input → small output. Output tokens are the priciest tokens you'll generate.

- **Be specific.** Targeted prompts return targeted answers. Broad prompts return essays.
- **Avoid "kitchen sink" prompts.** Don't bundle unrelated questions. Break complex tasks into smaller steps and request only what you need now.
- **Constrain the output explicitly.** Use phrases like:
  - "Summarize in 5 bullet points."
  - "Just give the function signature."
  - "Return only the changed lines as a diff."
  - "Code only, no commentary."

### Example

| Prompt Style                                                  | What happens                                       |
| ------------------------------------------------------------- | -------------------------------------------------- |
| "Tell me everything about batch jobs"                         | A long, general answer — lots of output tokens     |
| "Show failed batch jobs in the past hour and why they failed" | A short, targeted answer — far fewer output tokens |

📘 Walkthrough: [Ask Focused Questions](docs/focused-questions.md)

---

## Strategy 4 — Orchestrate Tools & Agents Efficiently

**Goal:** stop making the LLM do work a tool can do directly. Under UBB, every under-the-hood model call in an agent run generates billable tokens — long agentic sessions can consume far more than a single chat.

- **Prefer real tools over LLM reasoning.** If a query, API, or built-in skill exists, call it directly. Don't ask the model to "think aloud" through what a tool would just answer.
- **Prune unused MCP tools.** Every enabled tool's name + JSON schema is re-sent on every turn. A ~40-tool MCP server can add 10–15 KB of schema per call; removing the ones the agent never invokes saved GitHub's team several thousand tokens per workflow run with no behavior change ([source](https://github.blog/ai-and-ml/github-copilot/improving-token-efficiency-in-github-agentic-workflows/)).
- **Prefer a CLI over MCP for data fetching.** Calling `gh pr diff`, `az ... --output json`, or `kubectl get ... -o json` is a deterministic HTTP request — no LLM round-trip, no tool-schema overhead in context. Reserve MCP tools for steps that actually need the model to interpret or decide ([source](https://github.blog/ai-and-ml/github-copilot/improving-token-efficiency-in-github-agentic-workflows/)).
- **Minimize chain-of-thought steps.** Each planning/reflection step is another model call generating more tokens. If one good prompt does the job, prefer that over a multi-step chain.
- **Split monolithic agents.** A single agent loaded with every domain rule re-sends that whole prompt on every call. Specialized sub-agents and skills with smaller prompts only pull weight when invoked.
- **Scope your agents.** Use custom agents (`.agent.md`) and Plan → Start Implementation so each step has a small goal and a restricted tool set.
- **Offload deterministic steps.** Use plain code/logic for anything that doesn't actually need an LLM.

### Example

When a tool exists for a deterministic step, using the tool directly avoids many rounds of model reasoning (and the tokens that come with them).

📘 Walkthroughs: [Use Tools & Agents Efficiently](docs/tools-and-agents.md) · [Scoped Agents & Handoffs](docs/scoped-agents.md)

---

## Strategy 5 — Pick the Right Model & Build Good Habits

**Goal:** match per-token cost to the value of the answer.

- **Right-size the model.** Per-token prices span roughly 25× between lightweight (e.g. GPT-5 mini, GPT-5.6 Luna, MAI-Code-1-Flash, Gemini Flash) and powerful (e.g. GPT-5.5, GPT-5.6 Sol, Claude Opus, Gemini Pro) models. Save the powerful ones for genuinely hard problems.
- **Match reasoning to the task.** Some models expose a configurable **reasoning level** (VS Code + CLI); higher reasoning burns more tokens, so keep it regular by default and raise it only for hard problems.
- **Use Auto for routine work.** Copilot auto model selection routes by task intent, **protects your cache** (switches only at natural boundaries), and earns a **10% discount on model costs on any paid plan**; under UBB it also avoids pinning an expensive model when you don't need it.
- **Use the right surface.** Inline suggestions and next edit suggestions don't draw AI Credits on paid plans — use them for routine edits. Reserve Chat for higher-value queries.
- **Coach your team.** Token economy is a habit:
  - Ask precise questions.
  - Start fresh chats when topics change.
  - Watch output verbosity.
  - Don't request a full rewrite when a one-line patch will do.

📘 Walkthroughs: [Choose the Right Model](docs/choose-model.md) · [Understand AI Credits & Per-Token Pricing](docs/ai-credits.md)

---

## Real-World Savings Examples

| Pattern                                                           | Optimized Approach                                                                                     | Why it helps under UBB                                                                                                                                                                  |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Overly broad query: *"Explain everything about X"*                | Focused query: *"Show X's key errors and reasons"*                                                     | Drastically fewer **output tokens** (the priciest bucket)                                                                                                                               |
| LLM reasoning through a tool step                                 | Direct tool/API call                                                                                   | Avoids many extra model calls and the tokens they generate                                                                                                                              |
| Large always-on instructions                                      | On-demand prompts/skills                                                                               | Keeps input tokens small on every turn                                                                                                                                                  |
| Growing chat history kept intact                                  | `/clear`, new chat, or `/compact`                                                                      | Caps the running input the model re-reads each turn                                                                                                                                     |
| Powerful model for trivial tasks                                  | Lightweight model for routine work                                                                     | Up to ~25× cheaper per token                                                                                                                                                            |
| Mega-agent with every tool loaded                                 | Plan → Start Implementation; scoped custom agents                                                      | Fewer model calls per task; smaller, focused prompts                                                                                                                                    |
| 40-tool MCP server enabled, only 2 tools used                     | Prune unused MCP tools / disable whole MCP servers                                                     | Removes 8–12 KB of schema re-sent on **every** turn ([source](https://github.blog/ai-and-ml/github-copilot/improving-token-efficiency-in-github-agentic-workflows/))                    |
| Agent calls an MCP tool to fetch a PR diff / file contents / logs | Run `gh pr diff` / `az ... --output json` / `kubectl get ... -o json` and feed the result to the agent | Replaces an LLM reasoning round-trip with a deterministic HTTP request ([source](https://github.blog/ai-and-ml/github-copilot/improving-token-efficiency-in-github-agentic-workflows/)) |
| Switching models mid-session / resuming a cold chat               | One model per session (or `Auto`); `/compact` before resuming                                          | Keeps the cache warm so context re-reads at ~10% instead of full input price                                                                                                            |
| Uncapped agent run that loops for many steps                      | Set an **AI credit session limit** in Copilot CLI/SDK                                                  | Agent stops cleanly at the cap instead of silently burning credits                                                                                                                      |

---

## Walkthroughs (How-To Docs)

Step-by-step guides for the actions referenced above. Each is a short, screenshot-friendly walkthrough.

| Goal                                                                      | Walkthrough                                                      |
| ------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Understand why quality (not cost) is the right first lever                | [Think in Quality Economics](docs/quality-economics.md)          |
| Understand how harnesses, the stateless LLM & token loops work            | [How Context Windows Work](docs/context-windows.md)              |
| End or reset a chat to drop stale context                                 | [Prune Chat History](docs/prune-history.md)                      |
| Keep the cache warm so context re-reads at ~10% price                     | [Preserve the Cache](docs/preserve-cache.md)                     |
| Pick the cheapest model that fits the task                                | [Choose the Right Model](docs/choose-model.md)                   |
| Understand AI Credits, per-token pricing & budgets                        | [Understand AI Credits & Per-Token Pricing](docs/ai-credits.md)  |
| Attach only the right files / selections                                  | [Manage Context Attachments](docs/manage-context.md)             |
| Get short, on-target answers                                              | [Ask Focused Questions](docs/focused-questions.md)               |
| Split work into fresh windows per phase                                   | [Research → Plan → Implement](docs/research-plan-implement.md)   |
| Counter compounding errors with tests & linters                           | [Add Deterministic Guardrails](docs/deterministic-guardrails.md) |
| Make agents call tools instead of reasoning aloud                         | [Use Tools & Agents Efficiently](docs/tools-and-agents.md)       |
| Build planner→implementer agent handoffs                                  | [Scoped Agents & Handoffs](docs/scoped-agents.md)                |
| Move boilerplate out of every prompt                                      | [Externalize Custom Instructions](docs/custom-instructions.md)   |
| Know which config to use (instructions, agents, skills, MCPs, subagents…) | [Agent Configs Explained](docs/agent-configs.md)                 |
| Squeeze static-token overhead at scale                                    | [Power-User Token Tips](docs/power-user-tips.md)                 |
| Apply FinOps controls to production agent systems                         | [Apply FinOps to Production Agents](docs/agent-finops.md)        |
| Build durable habits for agentic development                              | [Future-Proof Your Skills](docs/future-proofing.md)              |

> Screenshots live in [`docs/images/`](docs/images/README.md). The how-to docs reference them by filename — drop matching PNGs in to enable inline images.

---

## Checklist for Engineers

Before sending a Copilot Chat message, ask yourself:

- [ ] Am I using the **cheapest model** (or `Auto`) whose per-token pricing matches the value of this answer?
- [ ] Is my prompt **as short as it can be** without losing meaning?
- [ ] Have I attached **only the relevant** file/snippet (not `#codebase` for a narrow question)?
- [ ] Am I asking **one focused question**, not five?
- [ ] Have I **constrained the output** (length, format, schema) to keep output tokens down?
- [ ] Is this conversation still **on-topic**, or should I start a new chat (or `/compact`)?
- [ ] Could a **tool, script, or inline completion** answer this without an LLM?
- [ ] If this is an agent task, is it **scoped** (small goal, restricted tools) so it doesn't loop into a long, token-heavy session?
- [ ] Are there **deterministic guardrails** (tests, linters, scanners) in place so the agent catches its own errors before they compound?

---

## Tools and Extensions

- [Optimizing your AI usage to maximize efficiency and reduce cost](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage) — GitHub's **official** companion tutorial to this guide (model choice, lean context, cache preservation, session limits, guardrails).
- [Chat Customizations Evaluations](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-chat-customizations-evaluations) — VS Code extension for evaluating and iterating on Copilot Chat customizations (instructions, prompts, and agents).

---
  
**Bottom line:** Under UBB, tokens = money. Trim what you send, scope what you ask, shorten what comes back, and let the right model (or tool) do the right job.
