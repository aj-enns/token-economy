# Token Economy: Optimizing GitHub Copilot Chat & Agents under Usage-Based Billing

A practical guide to controlling spend when using GitHub Copilot Chat and agents under **usage-based billing (UBB)** — the billing model GitHub is moving Copilot to on **June 1, 2026** ([source](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)).

> **TL;DR** — Under UBB, you pay for **tokens** (input + output + cached) converted to **GitHub AI Credits** (1 credit = $0.01). Pick a cheaper model, keep prompts lean, ask focused questions to shorten outputs, prune history, scope agents, and let tools do deterministic work instead of the LLM.

---

## Table of Contents

- [Token Economy: Optimizing GitHub Copilot Chat \& Agents under Usage-Based Billing](#token-economy-optimizing-github-copilot-chat--agents-under-usage-based-billing)
  - [Table of Contents](#table-of-contents)
  - [How UBB Bills Copilot](#how-ubb-bills-copilot)
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

Each Copilot plan ships with a **monthly AI Credits allowance**:

| Plan | Monthly AI Credits | Notes |
| --- | --- | --- |
| Copilot Free | Limited | Plus 2,000 inline suggestions/month |
| Copilot Pro | 1,000 | |
| Copilot Pro+ | 3,900 | |
| Copilot Business | 1,900 per license | Pooled at billing-entity level |
| Copilot Enterprise | 3,900 per license | Pooled at billing-entity level |

When the allowance is exhausted, additional usage is billed at per-token rates (1 credit = $0.01), subject to budget policy.

**Code completions and next edit suggestions are not billed in AI Credits** — they stay unlimited on paid plans.

> **Was it different before?** Yes. Before June 1, 2026, Copilot used **request-based billing** — each prompt counted as one *premium request* multiplied by a model multiplier. Under UBB, multipliers go away (except for legacy annual subscribers who stay on request-based billing), and **the actual token volume** of every interaction is what determines cost.

Sources: [Usage-based billing for individuals](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals) · [Usage-based billing for organizations and enterprises](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises) · [Models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)

---

## Quick Wins (Start Here)

If you only do five things, do these:

1. **Use the cheapest model that meets the bar.** Per-token prices span roughly 25× across the lineup. → [How](docs/choose-model.md)
2. **Ask narrow questions to keep output short.** Under UBB, output tokens are priced ~4–8× higher than input tokens. → [How](docs/focused-questions.md)
3. **Start a new chat** when topics change — old turns get re-sent as input tokens on every reply. → [How](docs/prune-history.md)
4. **Skip Chat for trivial code.** Inline suggestions and next edit suggestions don't draw AI Credits at all on paid plans. → [How](docs/choose-model.md#4-use-inline-suggestions-for-everyday-code)
5. **Let tools do deterministic work.** Each agent step is more tokens; a single tool call costs much less than several rounds of reasoning. → [How](docs/tools-and-agents.md)

---

## Key Cost Drivers

| Driver | What It Is | Why It Costs in UBB |
| --- | --- | --- |
| **Model choice** | Which model you picked (or Auto) | Each model has its own per-token price for input, output, and cached tokens |
| **Output length** | How verbose the response is | Output tokens are typically the most expensive bucket (~4–8× input) |
| **Input length** | Prompt + attached context + chat history | Every turn re-sends the running conversation as input tokens |
| **Agent loops** | Multiple under-the-hood model calls per task | Long agentic sessions generate many tokens; complex agent runs can dwarf a single chat |
| **Tool overload** | Too many tools enabled | You can hit VS Code's 128-tools-per-request limit and lose time (and tokens) to retries |

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

📘 Walkthroughs: [Prune Chat History](docs/prune-history.md) · [Manage Context Attachments](docs/manage-context.md)

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

| Prompt Style | What happens |
| --- | --- |
| "Tell me everything about batch jobs" | A long, general answer — lots of output tokens |
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

- **Right-size the model.** Per-token prices span roughly 25× between lightweight (e.g. GPT-5 mini, Grok Code Fast, Gemini Flash) and powerful (e.g. GPT-5.5, Claude Opus, Gemini Pro) models. Save the powerful ones for genuinely hard problems.
- **Use Auto for routine work.** Copilot auto model selection picks based on real-time health and gets a **10% multiplier discount** on paid plans (for legacy request-based subscribers); under UBB, Auto still helps avoid pinning an expensive model when you don't need it.
- **Use the right surface.** Inline suggestions and next edit suggestions don't draw AI Credits on paid plans — use them for routine edits. Reserve Chat for higher-value queries.
- **Coach your team.** Token economy is a habit:
  - Ask precise questions.
  - Start fresh chats when topics change.
  - Watch output verbosity.
  - Don't request a full rewrite when a one-line patch will do.

📘 Walkthroughs: [Choose the Right Model](docs/choose-model.md) · [Understand AI Credits & Per-Token Pricing](docs/ai-credits.md)

---

## Real-World Savings Examples

| Pattern | Optimized Approach | Why it helps under UBB |
| --- | --- | --- |
| Overly broad query: *"Explain everything about X"* | Focused query: *"Show X's key errors and reasons"* | Drastically fewer **output tokens** (the priciest bucket) |
| LLM reasoning through a tool step | Direct tool/API call | Avoids many extra model calls and the tokens they generate |
| Large always-on instructions | On-demand prompts/skills | Keeps input tokens small on every turn |
| Growing chat history kept intact | `/clear`, new chat, or `/compact` | Caps the running input the model re-reads each turn |
| Powerful model for trivial tasks | Lightweight model for routine work | Up to ~25× cheaper per token |
| Mega-agent with every tool loaded | Plan → Start Implementation; scoped custom agents | Fewer model calls per task; smaller, focused prompts |
| 40-tool MCP server enabled, only 2 tools used | Prune unused MCP tools / disable whole MCP servers | Removes 8–12 KB of schema re-sent on **every** turn ([source](https://github.blog/ai-and-ml/github-copilot/improving-token-efficiency-in-github-agentic-workflows/)) |
| Agent calls an MCP tool to fetch a PR diff / file contents / logs | Run `gh pr diff` / `az ... --output json` / `kubectl get ... -o json` and feed the result to the agent | Replaces an LLM reasoning round-trip with a deterministic HTTP request ([source](https://github.blog/ai-and-ml/github-copilot/improving-token-efficiency-in-github-agentic-workflows/)) |

---

## Walkthroughs (How-To Docs)

Step-by-step guides for the actions referenced above. Each is a short, screenshot-friendly walkthrough.

| Goal | Walkthrough |
| --- | --- |
| End or reset a chat to drop stale context | [Prune Chat History](docs/prune-history.md) |
| Pick the cheapest model that fits the task | [Choose the Right Model](docs/choose-model.md) |
| Understand AI Credits, per-token pricing & budgets | [Understand AI Credits & Per-Token Pricing](docs/ai-credits.md) |
| Attach only the right files / selections | [Manage Context Attachments](docs/manage-context.md) |
| Get short, on-target answers | [Ask Focused Questions](docs/focused-questions.md) |
| Make agents call tools instead of reasoning aloud | [Use Tools & Agents Efficiently](docs/tools-and-agents.md) |
| Build planner→implementer agent handoffs | [Scoped Agents & Handoffs](docs/scoped-agents.md) |
| Move boilerplate out of every prompt | [Externalize Custom Instructions](docs/custom-instructions.md) |

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

---

## Tools and Extensions

- [Chat Customizations Evaluations](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-chat-customizations-evaluations) — VS Code extension for evaluating and iterating on Copilot Chat customizations (instructions, prompts, and agents).

---
  
**Bottom line:** Under UBB, tokens = money. Trim what you send, scope what you ask, shorten what comes back, and let the right model (or tool) do the right job.
