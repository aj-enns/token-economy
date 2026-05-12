# Token Economy: Optimizing GitHub Copilot Chat & Agents

A practical guide to reducing waste and controlling spend when using GitHub Copilot Chat and agents under usage-based billing (UBB).

> **TL;DR** — Pick the right model, keep prompts lean, scope questions narrowly, prune chat history, and let tools do the heavy lifting instead of the LLM.

---

## Table of Contents

- [Why Token Spend Matters](#why-token-spend-matters)
- [Quick Wins (Start Here)](#quick-wins-start-here)
- [Key Cost Drivers](#key-cost-drivers)
- [Strategy 1 — Trim Prompt Overhead](#strategy-1--trim-prompt-overhead)
- [Strategy 2 — Manage Memory & Context](#strategy-2--manage-memory--context)
- [Strategy 3 — Ask Focused Questions](#strategy-3--ask-focused-questions)
- [Strategy 4 — Orchestrate Tools & Agents Efficiently](#strategy-4--orchestrate-tools--agents-efficiently)
- [Strategy 5 — Pick the Right Model & Build Good Habits](#strategy-5--pick-the-right-model--build-good-habits)
- [Real-World Savings Examples](#real-world-savings-examples)
- [Walkthroughs (How-To Docs)](#walkthroughs-how-to-docs)
- [Checklist for Engineers](#checklist-for-engineers)

---

## Why Token Spend Matters

For GitHub Copilot Chat in an IDE, cost is tracked as **premium requests**: each prompt you enter counts as one premium request, multiplied by the selected model’s multiplier. In agent mode, Copilot may take follow-up actions to complete your task, but those tool calls/background steps are not charged.


---

## Quick Wins (Start Here)

If you only do five things, do these:

1. **Use the cheapest model that gets the job done.** Reserve top-tier models for genuinely complex tasks. → [How](docs/choose-model.md)
2. **Ask narrow questions.** "Show failed jobs in the last hour" beats "Tell me everything about jobs." → [How](docs/focused-questions.md)
3. **Start a new chat** when you switch topics — don't drag old context forward. → [How](docs/prune-history.md)
4. **Skip the chat for trivial code.** Prefer inline suggestions for routine edits (Copilot Free has a monthly limit for inline suggestions; paid plans include inline suggestions with included models). → [How](docs/choose-model.md#4-use-inline-suggestions-for-everyday-code)
5. **Call a tool directly** instead of asking the model to reason through what a tool already does. → [How](docs/tools-and-agents.md)

---

## Key Cost Drivers

| Driver | What It Is | Why It Costs |
| --- | --- | --- |
| **Model multiplier** | Which model you picked (or Auto) | Each prompt consumes premium requests based on model multipliers |
| **Number of prompts** | Back-and-forth and retries | Each prompt you enter is billed as a premium request |
| **Agent loops** | Repeated corrections/steering | More prompts to steer = more billed requests |
| **Tool overload** | Too many tools enabled | You can hit VS Code’s 128-tools-per-request limit and lose time to retries |

---

## Strategy 1 — Trim Prompt Overhead

**Goal:** stop paying for the same boilerplate on every call.

- **Externalize persistent instructions.** Move large style guides, role definitions, or boilerplate out of always-on instructions and into on-demand prompt files or skills.
- **Inject context dynamically.** Attach only the file or snippet the question is actually about. Don't paste an entire library when the question is about one function.
- **Be concise in your wording.** Replace multi-sentence instructions with one or two crisp sentences. Shorter prompts tend to produce more focused responses and reduce follow-up prompts.

📘 Walkthrough: [Externalize Custom Instructions](docs/custom-instructions.md)

---

## Strategy 2 — Manage Memory & Context

**Goal:** keep the context window small and relevant.

- **Prune history.** Limit how many recent turns the model sees, or summarize older exchanges into a short recap.
- **Reset when topic changes.** Start a new chat when you finish a logical task. Don't carry yesterday's conversation into today's question.
- **Summarize long histories.** Use a smaller, cheaper model to compress chat history before feeding it back to a premium model.
- **Cache and reference, don't re-send.** If the same context is reused across turns, store it once and refer to it (e.g., "use the schema from earlier") instead of pasting it again.

📘 Walkthroughs: [Prune Chat History](docs/prune-history.md) · [Manage Context Attachments](docs/manage-context.md)

---

## Strategy 3 — Ask Focused Questions

**Goal:** small input → small output.

- **Be specific.** Targeted prompts return targeted answers. Broad prompts return essays.
- **Avoid "kitchen sink" prompts.** Don't bundle unrelated questions. Break complex tasks into smaller steps and request only what you need now.
- **Constrain the output.** Use phrases like:
  - "Summarize in 5 bullet points."
  - "Just give the function signature."
  - "Return only the changed lines."

### Example

| Prompt Style | What happens |
| --- | --- |
| "Tell me everything about batch jobs" | You tend to get a long, general answer |
| "Show failed batch jobs in the past hour and why they failed" | You tend to get a short, targeted answer |

📘 Walkthrough: [Ask Focused Questions](docs/focused-questions.md)

---

## Strategy 4 — Orchestrate Tools & Agents Efficiently

**Goal:** stop making the LLM do work a tool can do directly.

- **Prefer real tools over LLM reasoning.** If a query, API, or built-in skill exists, call it directly. Don't ask the model to "think aloud" through what a tool would just answer.
- **Minimize chain-of-thought steps.** Each planning/reflection step is another model call. If one good prompt does the job, prefer that over a multi-step chain.
- **Split monolithic agents.** A single agent loaded with every domain rule pays for that whole prompt on every call. Specialized sub-agents/skills with smaller prompts only pull weight when invoked.
- **Scope your agents.** Use custom agents (`.agent.md`) and Plan → Start Implementation to reduce iteration and keep workflows predictable.
- **Offload deterministic steps.** Use plain code/logic for anything that doesn't actually need an LLM.

### Example

When a tool exists for a deterministic step, using the tool directly avoids unnecessary back-and-forth.

📘 Walkthroughs: [Use Tools & Agents Efficiently](docs/tools-and-agents.md) · [Scoped Agents & Handoffs](docs/scoped-agents.md)

---

## Strategy 5 — Pick the Right Model & Build Good Habits

**Goal:** match cost to value.

- **Right-size the model.** Use lighter models for routine tasks (renames, simple completions, doc lookups). Save premium models for complex reasoning, multi-file refactors, or architectural questions.
- **Use the right surface.** Use inline suggestions for routine edits and Copilot Chat for higher-value queries.
- **Coach your team.** Token efficiency is a habit:
  - Ask precise questions.
  - Start fresh chats when topics change.
  - Watch output verbosity.
  - Don't request a full rewrite when a one-line patch will do.

📘 Walkthroughs: [Choose the Right Model](docs/choose-model.md) · [Understand Premium Requests](docs/premium-requests.md)

---

## Real-World Savings Examples

| Pattern | Optimized Approach | Why it helps |
| --- | --- | --- |
| Overly broad query: *"Explain everything about X"* | Focused query: *"Show X's key errors and reasons"* | Less irrelevant output and fewer follow-ups |
| LLM reasoning through a tool step | Direct tool/API call | More deterministic results with fewer retries |
| Large always-on instructions | On-demand prompts/skills | Keeps always-on instructions concise |
| Growing chat history kept intact | Truncate or summarize regularly | Caps context, prevents runaway growth |
| Premium model for trivial tasks | Lighter model for routine work | Multiples cheaper per call |
| Mega-agent with every tool loaded | Plan → Start Implementation; scoped custom agents | Clearer step boundaries and fewer steering prompts |

---

## Walkthroughs (How-To Docs)

Step-by-step guides for the actions referenced above. Each is a short, screenshot-friendly walkthrough.

| Goal | Walkthrough |
| --- | --- |
| End or reset a chat to drop stale context | [Prune Chat History](docs/prune-history.md) |
| Pick the cheapest model that fits the task | [Choose the Right Model](docs/choose-model.md) |
| Understand premium-request multipliers & budgets | [Understand Premium Requests](docs/premium-requests.md) |
| Attach only the right files / selections | [Manage Context Attachments](docs/manage-context.md) |
| Get short, on-target answers | [Ask Focused Questions](docs/focused-questions.md) |
| Make agents call tools instead of reasoning aloud | [Use Tools & Agents Efficiently](docs/tools-and-agents.md) |
| Build planner→implementer agent handoffs | [Scoped Agents & Handoffs](docs/scoped-agents.md) |
| Move boilerplate out of every prompt | [Externalize Custom Instructions](docs/custom-instructions.md) |

> Screenshots live in [`docs/images/`](docs/images/README.md). The how-to docs reference them by filename — drop matching PNGs in to enable inline images.

---

## Checklist for Engineers

Before sending a Copilot Chat message, ask yourself:

- [ ] Am I using the **cheapest model** (or `Auto`) that can answer this?
- [ ] Is my prompt **as short as it can be** without losing meaning?
- [ ] Have I attached **only the relevant** file/snippet?
- [ ] Am I asking **one focused question**, not five?
- [ ] Have I **constrained the output** (length, format, schema)?
- [ ] Is this conversation still **on-topic**, or should I start a new chat?
- [ ] Could a **tool, script, or inline completion** answer this instead?
- [ ] If this is an agent task, is it **scoped** (small goal, restricted tools)?

---

**Bottom line:** Token efficiency = precision + parsimony. Trim what you send, scope what you ask, and let the right model (or tool) do the right job.
