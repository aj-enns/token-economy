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

Each paid individual plan's monthly allowance is split into **base credits** (fixed, matched to your subscription price) plus a **flex allotment** (a variable top-up GitHub tunes as model economics change). Base credits are consumed first; the flex allotment applies automatically after ([source](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)).

| Plan                   | Base credits | Flex allotment | Total monthly AI Credits                      | Notes                                                      |
| ---------------------- | ------------ | -------------- | --------------------------------------------- | ---------------------------------------------------------- |
| Copilot Free / Student | —            | —              | Limited allowance (auto model selection only) | Free: 2,000 completions/mo; Student: unlimited completions |
| Copilot Pro ($10/mo)   | 1,000        | 500            | **1,500**                                     | Individual                                                 |
| Copilot Pro+ ($39/mo)  | 3,900        | 3,100          | **7,000**                                     | Individual                                                 |
| Copilot Max ($100/mo)  | 10,000       | 10,000         | **20,000**                                    | Individual — highest allowance                             |
| Copilot Business       | —            | —              | 1,900 per license                             | Pooled at billing-entity level                             |
| Copilot Enterprise     | —            | —              | 3,900 per license                             | Pooled at billing-entity level                             |

> **Copilot Max** is an individual plan added under UBB with the largest allowance — worth it if you regularly exhaust Pro+ and would otherwise pay for additional usage.

> **Promotional period** (June 1 – September 1, 2026): existing Copilot Business / Enterprise customers receive higher allowances (3,000 and 7,000 respectively) for the first three months.

> Included credits **do not carry over** — the allowance resets to the full monthly amount at 00:00:00 UTC on the first of each month.

### What's **not** billed in AI Credits

Code completions and next edit suggestions are **not** billed in AI Credits. They remain unlimited on paid plans and use their existing counting mechanism.

---

## 2. The three token buckets (and why they cost differently)

| Bucket           | What it is                                                    | Typical relative cost                       |
| ---------------- | ------------------------------------------------------------- | ------------------------------------------- |
| **Input**        | Your prompt, attached context, instructions, prior chat turns | Baseline                                    |
| **Cached input** | Context the model has already seen and stored                 | Roughly 10% of input cost (model-dependent) |
| **Output**       | What the model generates back to you                          | Roughly **4–8× the input rate**             |

**Implications:**

- **Output tokens dominate cost** when responses are long. Constraining output (see [Ask Focused Questions](focused-questions.md)) is one of the highest-leverage levers.
- **Chat history is input** — every turn re-sends prior turns. Pruning history (see [Prune Chat History](prune-history.md)) directly reduces input tokens on every subsequent reply.
- **Attached files are input** — only attach what you actually need (see [Manage Context Attachments](manage-context.md)).
- **Cached input is the cheap bucket (~10%)** — but only while the cache stays warm. Switching models mid-session, changing reasoning/tools, or resuming a cold session forces a full-price rebuild. See [Preserve the Cache](preserve-cache.md).

---

## 3. Per-token prices (per 1M tokens)

Prices below are per **1 million tokens**, in USD, per the official [Models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing) page. Prices change over time — use the link as the source of truth.

> **Long-context tiers:** several models now charge a higher rate once a request crosses a size threshold (e.g. GPT-5.x above 272K input tokens, Gemini/Luna above 200K). The rates below are the **Default** (below-threshold) tier — big-context requests cost more. This is another reason to keep context lean.

### OpenAI

| Model         | Tier        | Input | Cached input | Output |
| ------------- | ----------- | ----- | ------------ | ------ |
| GPT-5 mini    | Lightweight | $0.25 | $0.025       | $2.00  |
| GPT-5.4 nano  | Lightweight | $0.20 | $0.02        | $1.25  |
| GPT-5.4 mini  | Lightweight | $0.75 | $0.075       | $4.50  |
| GPT-5.6 Luna  | Lightweight | $1.00 | $0.10        | $6.00  |
| GPT-5.4       | Versatile   | $2.50 | $0.25        | $15.00 |
| GPT-5.6 Terra | Versatile   | $2.50 | $0.25        | $15.00 |
| GPT-5.3-Codex | Powerful    | $1.75 | $0.175       | $14.00 |
| GPT-5.5       | Powerful    | $5.00 | $0.50        | $30.00 |
| GPT-5.6 Sol   | Powerful    | $5.00 | $0.50        | $30.00 |

> **GPT-5.6 family:** Luna (lightweight, lowest-cost), Terra (balanced default), Sol (highest reasoning ceiling). Match the variant to the job.

### Anthropic (includes a cache-write cost in addition to cached input)

| Model                             | Tier      | Input  | Cached input | Cache write | Output |
| --------------------------------- | --------- | ------ | ------------ | ----------- | ------ |
| Claude Haiku 4.5                  | Versatile | $1.00  | $0.10        | $1.25       | $5.00  |
| Claude Sonnet 5                   | Versatile | $2.00  | $0.20        | $2.50       | $10.00 |
| Claude Sonnet 4 / 4.5 / 4.6       | Versatile | $3.00  | $0.30        | $3.75       | $15.00 |
| Claude Opus 4.5 / 4.6 / 4.7 / 4.8 | Powerful  | $5.00  | $0.50        | $6.25       | $25.00 |
| Claude Fable 5                    | Powerful  | $10.00 | $1.00        | $12.50      | $50.00 |

### Google

| Model                    | Tier        | Input | Cached input | Output |
| ------------------------ | ----------- | ----- | ------------ | ------ |
| Gemini 3 Flash (preview) | Lightweight | $0.50 | $0.05        | $3.00  |
| Gemini 3.5 Flash         | Lightweight | $1.50 | $0.15        | $9.00  |
| Gemini 2.5 Pro           | Powerful    | $1.25 | $0.125       | $10.00 |
| Gemini 3.1 Pro (preview) | Powerful    | $2.00 | $0.20        | $12.00 |

### Fine-tuned (GitHub) & Microsoft

| Model            | Provider  | Tier        | Input | Cached input | Output |
| ---------------- | --------- | ----------- | ----- | ------------ | ------ |
| Raptor mini      | GitHub    | Versatile   | $0.25 | $0.025       | $2.00  |
| MAI-Code-1-Flash | Microsoft | Lightweight | $0.75 | $0.075       | $4.50  |

### Open-weight (Moonshot AI)

| Model          | Tier      | Input | Cached input | Output |
| -------------- | --------- | ----- | ------------ | ------ |
| Kimi K2.7 Code | Versatile | $0.95 | $0.19        | $4.00  |

> **Kimi K2.7 Code** is the **first open-weight model** selectable in the Copilot model picker — a lower-cost option, hosted by GitHub on Azure. It is **off by default** for Business/Enterprise; admins must enable it and should review open-weight models against their own security, compliance, and data-governance requirements first ([source](https://github.blog/changelog/2026-07-01-kimi-k2-7-is-now-available-in-github-copilot/)).

**Tier rule of thumb (stable even if model names change):**

| Tier            | Use for                                                    |
| --------------- | ---------------------------------------------------------- |
| **Lightweight** | Renames, syntax help, doc Q&A, simple edits                |
| **Versatile**   | Single-file edits, small refactors, tests                  |
| **Powerful**    | Multi-file refactors, architectural design, deep debugging |

---

## 4. Auto model selection

[Copilot auto model selection](https://docs.github.com/en/copilot/concepts/auto-model-selection) chooses a model in real time based on the intent of your task, reserving expensive reasoning models for complex problems and avoiding models that burn a token budget quickly. **If you're on any paid Copilot plan, you get a 10% discount on model costs while using Auto** in Copilot Chat, Copilot CLI, the GitHub Copilot app, or Copilot cloud agent ([source](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage)).

- Pin a powerful model only when you know the next few turns need it.
- Switch back to `Auto` (or a lightweight model) after the hard part is done.
- **Auto also protects your cache** — it only changes models at natural cache boundaries (a new session or after `/compact`), never mid-task, so you don't pay to rebuild context. See [Preserve the Cache](preserve-cache.md).

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

## 6. Set a budget, cap sessions, and watch spend

**Budgets** (cap spend beyond the included allowance):

- In **GitHub billing settings**, set a **budget** in US dollars for additional Copilot usage. AI Credits draw down at $0.01 each (e.g., a $10 budget = 1,000 AI Credits).
- For organizations and enterprises, budgets can be set at **enterprise, organization, cost-center, or user level**. A **$0 user-level budget = no access at all** for that user.
- There is **no automatic fallback to lower-cost models** when a budget is exhausted — usage stops.

**New enterprise spend controls (rolled out June–July 2026, REST API first):**

- **Cost-center user-level budgets** — one per-user budget scoped to a cost center that *follows membership* automatically, so you don't reconfigure budgets when people move teams. It counts *included* usage too, so it can stop a user before the shared pool is even exhausted ([source](https://github.blog/changelog/2026-06-30-per-user-ai-credit-budgets-available-for-cost-centers/)).
- **Cost-center AI credit pools (included-usage caps)** — cap how much of the enterprise's shared *included* credit pool a cost center can draw, so one team can't spend credits another team's licenses paid for. GitHub computes the cap from the licenses assigned to that cost center ([source](https://github.blog/changelog/2026-07-02-cost-centers-now-support-included-usage-caps/)).

**AI credit session limits** (cap a single task):

- In **Copilot CLI and the Copilot SDK**, set an **AI credit session limit** before starting a task. When the limit is reached, the agent **stops cleanly, notifies you, and lets you continue or raise the limit**. These are *soft* limits — great for capping runaway agent runs or tuning the minimum credits that still gets a good result. They don't replace budgets/spending limits ([source](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage)).

**Watch where credits go:**

- The **AI usage** page under [github.com/settings/billing](https://github.com/settings/billing) breaks down consumption across every feature and model.
- In VS Code, hover over a chat response to see the AI credits consumed by that turn.
- Hover over or select the **context window control** in the chat input to see the cumulative token breakdown and total AI credits for the session.
- Open the **Copilot status dashboard** from the Status Bar to see the percentage of your monthly allowance used.
- Use **Agent Debug Logs** for aggregate token usage, tool calls, errors, and duration, and use **Cache Explorer** to inspect cache hit rates and reused input tokens.

See [Optimize AI credit usage in VS Code](https://code.visualstudio.com/docs/agents/guides/optimize-usage#_monitor-your-usage) and [Debug chat interactions](https://code.visualstudio.com/docs/agents/agent-troubleshooting/chat-debug-view).

📚 Sources: [Budgets for usage-based billing](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing) · [Optimizing your AI usage](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage)

---

## 7. Quick checklist before each chat

- [ ] Is the model picker on **Auto** (or a lightweight tier) for routine work?
- [ ] Is my prompt focused enough to keep **output tokens** small?
- [ ] Have I removed unused attachments / tools that would balloon **input tokens**?
- [ ] Am I keeping the **cache warm** (one model per session; reasoning/tools set up front)?
- [ ] Am I about to fire an agent that may run for many steps — should I set an **AI credit session limit**?
- [ ] Do I have an **AI Credits budget** configured for the entity that pays?

---

## Appendix: How this compares to request-based billing (legacy)

Before June 1, 2026, Copilot billed by **premium requests**: each prompt you entered counted as one premium request multiplied by the model's multiplier; tool calls / background steps in agent mode were **not** charged. Some annual subscribers may stay on this legacy model after June 1; for them, the [Model multipliers for annual plans staying on request-based billing](https://docs.github.com/en/copilot/reference/copilot-billing/model-multipliers-for-annual-plans) page is the source of truth.

For everyone else, **AI Credits replace premium requests**, and tokens (input + output + cached) are what's actually metered.

---

[← Back to README](../readme.md)
