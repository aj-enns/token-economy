# How-To: Understand AI Credits & Per-Token Pricing

Starting **June 1, 2026**, GitHub Copilot moves from request-based billing to **usage-based billing (UBB)**. Under UBB, Copilot meters **tokens** for every interaction in Chat, CLI, cloud agent, Spaces, Spark, and third-party coding agents, then converts them to **GitHub AI Credits** (1 AI Credit = **$0.01 USD**).

> **Goal:** understand how each Copilot interaction is priced so you can keep spend predictable.

📚 Sources: [Usage-based billing for individuals](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals) · [Usage-based billing for organizations and enterprises](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises) · [Models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)

---

## 1. What is an AI Credit?

- **1 AI Credit = $0.01 USD.**
- Every Copilot interaction consumes **tokens**: input (what's sent to the model), output (what the model generates), and cached (context reused from earlier).
- Each token is priced per the model used, and the total is converted into AI Credits.
- Your plan includes a **monthly allowance** of AI Credits; usage beyond that allowance is billed at the same per-token rates, subject to your budget policy.

### Plan allowances

| Plan | Monthly AI Credits | Notes |
| --- | --- | --- |
| Copilot Free | Limited (specific allowance not published in the [individuals UBB doc](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)) | Plus 2,000 inline suggestions/month |
| Copilot Pro | 1,000 | Individual |
| Copilot Pro+ | 3,900 | Individual |
| Copilot Business | 1,900 per license | Pooled at billing-entity level |
| Copilot Enterprise | 3,900 per license | Pooled at billing-entity level |

> **Promotional period** (June 1 – September 1, 2026): existing Copilot Business / Enterprise customers receive higher allowances (3,000 and 7,000 respectively) for the first three months.

### What's **not** billed in AI Credits

Code completions and next edit suggestions are **not** billed in AI Credits. They remain unlimited on paid plans and use their existing counting mechanism.

---

## 2. The three token buckets (and why they cost differently)

| Bucket | What it is | Typical relative cost |
| --- | --- | --- |
| **Input** | Your prompt, attached context, instructions, prior chat turns | Baseline |
| **Cached input** | Context the model has already seen and stored | Roughly 10–25% of input cost (model-dependent) |
| **Output** | What the model generates back to you | Roughly **4–8× the input rate** |

**Implications:**

- **Output tokens dominate cost** when responses are long. Constraining output (see [Ask Focused Questions](focused-questions.md)) is one of the highest-leverage levers.
- **Chat history is input** — every turn re-sends prior turns. Pruning history (see [Prune Chat History](prune-history.md)) directly reduces input tokens on every subsequent reply.
- **Attached files are input** — only attach what you actually need (see [Manage Context Attachments](manage-context.md)).

---

## 3. Per-token prices (per 1M tokens)

Prices below are per **1 million tokens**, in USD, per the official [Models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing) page. Prices change over time — use the link as the source of truth.

### OpenAI

| Model | Tier | Input | Cached input | Output |
| --- | --- | --- | --- | --- |
| GPT-5 mini | Lightweight | $0.25 | $0.025 | $2.00 |
| GPT-5.4 nano | Lightweight | $0.20 | $0.02 | $1.25 |
| GPT-5.4 mini | Lightweight | $0.75 | $0.075 | $4.50 |
| GPT-4.1 | Versatile | $2.00 | $0.50 | $8.00 |
| GPT-5.2 / 5.2-Codex / 5.3-Codex | Versatile/Powerful | $1.75 | $0.175 | $14.00 |
| GPT-5.4 | Versatile | $2.50 | $0.25 | $15.00 |
| GPT-5.5 | Powerful | $5.00 | $0.50 | $30.00 |

### Anthropic (includes a cache-write cost in addition to cached input)

| Model | Tier | Input | Cached input | Cache write | Output |
| --- | --- | --- | --- | --- | --- |
| Claude Haiku 4.5 | Versatile | $1.00 | $0.10 | $1.25 | $5.00 |
| Claude Sonnet 4 / 4.5 / 4.6 | Versatile | $3.00 | $0.30 | $3.75 | $15.00 |
| Claude Opus 4.5 / 4.6 / 4.7 | Powerful | $5.00 | $0.50 | $6.25 | $25.00 |

### Google

| Model | Tier | Input | Cached input | Output |
| --- | --- | --- | --- | --- |
| Gemini 3 Flash | Lightweight | $0.50 | $0.05 | $3.00 |
| Gemini 2.5 Pro / Gemini 3.1 Pro | Powerful | $1.25–$2.00 | $0.125–$0.20 | $10.00–$12.00 |

### xAI / Fine-tuned

| Model | Tier | Input | Cached input | Output |
| --- | --- | --- | --- | --- |
| Grok Code Fast 1 | Lightweight | $0.20 | $0.02 | $1.50 |
| Raptor mini | Versatile | $0.25 | $0.025 | $2.00 |
| Goldeneye | Powerful | $1.25 | $0.125 | $10.00 |

**Tier rule of thumb (stable even if model names change):**

| Tier | Use for |
| --- | --- |
| **Lightweight** | Renames, syntax help, doc Q&A, simple edits |
| **Versatile** | Single-file edits, small refactors, tests |
| **Powerful** | Multi-file refactors, architectural design, deep debugging |

---

## 4. Auto model selection

[Copilot auto model selection](https://docs.github.com/en/copilot/concepts/auto-model-selection) chooses a model in real time based on system health and performance. Under UBB, it still helps avoid pinning an expensive model when you don't need one. (Legacy request-based subscribers also get a 10% multiplier discount when using Auto.)

- Pin a powerful model only when you know the next few turns need it.
- Switch back to `Auto` (or a lightweight model) after the hard part is done.

---

## 5. Watch agent / cloud-agent runs carefully

Under UBB, agentic features generate **multiple model calls per task**, and every one consumes tokens:

- **Copilot Chat agent mode**: each model call in the agent loop generates input + output tokens.
- **Copilot cloud agent**: a single session can span many model calls across files — a complex run can dwarf a typical chat interaction.
- **Copilot CLI**: each invocation is metered.

> **Compare with the prior model:** under request-based billing (before June 1, 2026), only the prompts *you* entered counted as premium requests, and tool calls / background steps were not charged. Under UBB, those under-the-hood calls **do** generate tokens, so agent loops materially affect spend.

Mitigations:

- Give agents **small, scoped goals** (see [Use Tools & Agents Efficiently](tools-and-agents.md)).
- Cancel runs early when you see the agent looping or re-reading the same files.
- Pair a **planning agent** (lightweight model) with an **implementation agent** (powerful model). See [Scoped Agents & Handoffs](scoped-agents.md).

---

## 6. Set a budget

Both individuals and organizations can cap additional usage beyond the included allowance:

- In **GitHub billing settings**, set a **budget** in US dollars for additional Copilot usage. AI Credits draw down at $0.01 each (e.g., a $10 budget = 1,000 AI Credits).
- For organizations and enterprises, budgets can be set at **enterprise, organization, cost-center, or user level**. A **$0 user-level budget = no access at all** for that user.
- There is **no automatic fallback to lower-cost models** when a budget is exhausted — usage stops.

📚 Source: [Setting up budgets to control spending on metered products](https://docs.github.com/en/billing/managing-your-billing/using-budgets-control-spending)

---

## 7. Quick checklist before each chat

- [ ] Is the model picker on **Auto** (or a lightweight tier) for routine work?
- [ ] Is my prompt focused enough to keep **output tokens** small?
- [ ] Have I removed unused attachments / tools that would balloon **input tokens**?
- [ ] Am I about to fire an agent that may run for many steps?
- [ ] Do I have an **AI Credits budget** configured for the entity that pays?

---

## Appendix: How this compares to request-based billing (legacy)

Before June 1, 2026, Copilot billed by **premium requests**: each prompt you entered counted as one premium request multiplied by the model's multiplier; tool calls / background steps in agent mode were **not** charged. Some annual subscribers may stay on this legacy model after June 1; for them, the [Model multipliers for annual plans staying on request-based billing](https://docs.github.com/en/copilot/reference/copilot-billing/model-multipliers-for-annual-plans) page is the source of truth.

For everyone else, **AI Credits replace premium requests**, and tokens (input + output + cached) are what's actually metered.

---

[← Back to README](../readme.md)
