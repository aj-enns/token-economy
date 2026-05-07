# Token Economy: Optimizing GitHub Copilot Chat & Agents

A practical guide to reducing token spend when using GitHub Copilot Chat and agents under usage-based billing (UBB).

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

Under usage-based billing, **every token counts** — input prompts, output responses, and conversation/context history all show up on the bill. A few small habits compound across thousands of daily interactions and can produce **80%+ cost reductions** without sacrificing quality.

---

## Quick Wins (Start Here)

If you only do five things, do these:

1. **Use the cheapest model that gets the job done.** Reserve top-tier models for genuinely complex tasks. → [How](docs/choose-model.md)
2. **Ask narrow questions.** "Show failed jobs in the last hour" beats "Tell me everything about jobs." → [How](docs/focused-questions.md)
3. **Start a new chat** when you switch topics — don't drag old context forward. → [How](docs/prune-history.md)
4. **Skip the chat for trivial code.** Inline ghost-text completions are unmetered. → [How](docs/choose-model.md#4-use-unmetered-inline-completions-for-everyday-code)
5. **Call a tool directly** instead of asking the model to reason through what a tool already does. → [How](docs/tools-and-agents.md)

---

## Key Cost Drivers

| Driver | What It Is | Why It Costs |
| --- | --- | --- |
| **Model selection** | GPT-4 / Opus-tier vs. lighter models | Premium models cost multiples per token |
| **Prompt size** | Custom instructions, pasted code, attachments | Every token is billed on every call |
| **Output length** | Verbose answers from broad questions | Output tokens are billed too |
| **Conversation history** | Long chats replay prior turns | Context tokens compound each turn |
| **Multi-call agents** | Agents that plan → reason → act in many steps | One user request = many model calls |

---

## Strategy 1 — Trim Prompt Overhead

**Goal:** stop paying for the same boilerplate on every call.

- **Externalize persistent instructions.** Move large style guides, role definitions, or boilerplate out of the default prompt and into on-demand skills/commands. A 200-token instruction removed from the default prompt saves 200 tokens **on every interaction**.
- **Inject context dynamically.** Attach only the file or snippet the question is actually about. Don't paste an entire library when the question is about one function.
- **Be concise in your wording.** Replace multi-sentence instructions with one or two crisp sentences. Shorter prompt = fewer input tokens.

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

| Prompt Style | Approx. Tokens |
| --- | --- |
| "Tell me everything about batch jobs" | ~5,000 output |
| "Show failed batch jobs in the past hour and why they failed" | ~800 output |

**~84% reduction** for the same useful answer.

📘 Walkthrough: [Ask Focused Questions](docs/focused-questions.md)

---

## Strategy 4 — Orchestrate Tools & Agents Efficiently

**Goal:** stop making the LLM do work a tool can do directly.

- **Prefer real tools over LLM reasoning.** If a query, API, or built-in skill exists, call it directly. Don't ask the model to "think aloud" through what a tool would just answer.
- **Minimize chain-of-thought steps.** Each planning/reflection step is another model call. If one good prompt does the job, prefer that over a multi-step chain.
- **Split monolithic agents.** A single agent loaded with every domain rule pays for that whole prompt on every call. Specialized sub-agents/skills with smaller prompts only pull weight when invoked.
- **Offload deterministic steps.** Use plain code/logic for anything that doesn't actually need an LLM.

### Example

| Approach | Tokens |
| --- | --- |
| LLM reasons through and writes a KQL query | ~6,000 |
| Direct call to a pre-built "exceptions" tool | ~1,200 |

**~80% reduction**, faster answer.

📘 Walkthrough: [Use Tools & Agents Efficiently](docs/tools-and-agents.md)

---

## Strategy 5 — Pick the Right Model & Build Good Habits

**Goal:** match cost to value.

- **Right-size the model.** Use lighter models for routine tasks (renames, simple completions, doc lookups). Save premium models for complex reasoning, multi-file refactors, or architectural questions.
- **Use unmetered features.** Inline ghost-text code completions are unmetered under current Copilot plans — use them for everyday coding and reserve Chat for high-value queries.
- **Coach your team.** Token efficiency is a habit:
  - Ask precise questions.
  - Start fresh chats when topics change.
  - Watch output verbosity.
  - Don't request a full rewrite when a one-line patch will do.

📘 Walkthrough: [Choose the Right Model](docs/choose-model.md)

---

## Real-World Savings Examples

| Pattern | Optimized Approach | Savings |
| --- | --- | --- |
| Overly broad query: *"Explain everything about X"* | Focused query: *"Show X's key errors and reasons"* | ~84% fewer output tokens |
| LLM reasoning through a tool step | Direct tool/API call | ~80% fewer tokens |
| Persistent ~300-token instruction every turn | On-demand skill/agent | Removes ~300 tokens per call |
| Growing chat history kept intact | Truncate or summarize regularly | Caps context, prevents runaway growth |
| Premium model for trivial tasks | Lighter model for routine work | Multiples cheaper per call |

---

## Walkthroughs (How-To Docs)

Step-by-step guides for the actions referenced above. Each is a short, screenshot-friendly walkthrough.

| Goal | Walkthrough |
| --- | --- |
| End or reset a chat to drop stale context | [Prune Chat History](docs/prune-history.md) |
| Pick the cheapest model that fits the task | [Choose the Right Model](docs/choose-model.md) |
| Attach only the right files / selections | [Manage Context Attachments](docs/manage-context.md) |
| Get short, on-target answers | [Ask Focused Questions](docs/focused-questions.md) |
| Make agents call tools instead of reasoning aloud | [Use Tools & Agents Efficiently](docs/tools-and-agents.md) |
| Move boilerplate out of every prompt | [Externalize Custom Instructions](docs/custom-instructions.md) |

> Screenshots live in [`docs/images/`](docs/images/README.md). The how-to docs reference them by filename — drop matching PNGs in to enable inline images.

---

## Checklist for Engineers

Before sending a Copilot Chat message, ask yourself:

- [ ] Am I using the **cheapest model** that can answer this?
- [ ] Is my prompt **as short as it can be** without losing meaning?
- [ ] Have I attached **only the relevant** file/snippet?
- [ ] Am I asking **one focused question**, not five?
- [ ] Have I **constrained the output** (length, format)?
- [ ] Is this conversation still **on-topic**, or should I start a new chat?
- [ ] Could a **tool, script, or inline completion** answer this instead?

---

**Bottom line:** Token efficiency = precision + parsimony. Trim what you send, scope what you ask, and let the right model (or tool) do the right job.
