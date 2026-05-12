# How-To: Understand Premium Requests & Model Multipliers

GitHub Copilot's usage-based billing (UBB) meters **premium requests** for many Copilot features. Each model has a different **multiplier**, so a single chat turn can consume different amounts of your allowance depending on the model.

> **Goal:** know what each call actually costs you, and prefer models with low multipliers when the task allows.

---

## 1. What is a premium request?

- Each Copilot plan includes a **monthly allowance** of premium requests (see the plan table for current amounts).
- In Copilot Chat (including agent and plan modes), **each prompt you enter** counts as one premium request multiplied by the model’s rate.
- Overage is purchased at a flat per-request rate ($0.04/request at time of writing).

Inline suggestions are metered differently from Chat. Copilot Free includes a monthly limit for real-time code suggestions, while paid plans include real-time code suggestions with included models.

📚 Source: [GitHub Copilot plans](https://docs.github.com/en/copilot/get-started/plans)

---

## 2. Lighter models cost dramatically less

Multipliers change over time and by plan, but the **tier rule** is stable:

| Tier | Typical models | Use for |
| --- | --- | --- |
| **Included / lowest multiplier** | GPT-5 mini, Claude Haiku, Grok Code Fast, Raptor mini, GPT-4.1 | Renames, syntax help, doc Q&A, simple edits |
| **Mid** | GPT-5.2, GPT-5.4 mini, Claude Sonnet, Gemini Flash | Single-file edits, small refactors, tests |
| **Premium** (high multiplier) | GPT-5.4 / 5.5, Claude Opus, Gemini Pro | Multi-file refactors, architectural design, deep debugging |

To see the exact multipliers, refer to the model multipliers table in GitHub’s billing docs.

---

## 3. Use the `auto` model picker for routine work

Copilot auto model selection chooses models based on real-time system health and performance. On paid plans, using auto model selection qualifies models for a 10% multiplier discount.

- Pin a premium model only when you know the next few turns need it.
- Reset to `Auto` (or a light model) after the hard part is done.

---

## 4. Watch agent / cloud-agent runs carefully

Agentic and cloud-agent runs can spawn **many** model calls per user request:

- In Copilot Chat agent mode, each prompt you enter counts as one premium request multiplied by the model’s multiplier; tool calls/background steps are not charged.
- **Copilot cloud agent**: one premium request is used per session, multiplied by the model’s rate. In addition, each real-time steering comment made during an active session uses one premium request per session, multiplied by the model’s rate.
- **Copilot CLI**: each invocation is metered.

Mitigations:

- Give agents **small, scoped goals** (see [Use Tools & Agents Efficiently](tools-and-agents.md)).
- Cancel runs early when you see the agent looping or re-reading the same files.
- Pair a **planning agent** (light model) with an **implementation agent** (premium model). See [Scoped Agents & Handoffs](scoped-agents.md).

---

## 5. Set a spending limit

Both individuals and organizations can cap overage spending:

- In your **GitHub billing settings**, set a **budget** for premium requests and enable **Stop usage when budget limit is reached** (if available) to block additional usage.
- Organizations can centrally manage per-user allowances and policy.

This is your safety net — turn it on **before** you experiment with agent mode.

---

## 6. Quick checklist before each chat

- [ ] Is the model picker on **Auto** (or a light tier)?
- [ ] Will my prompt fit in a single turn, or am I about to fire an agent that runs for 20 steps?
- [ ] Have I removed unused attachments / tools that would expand the prompt?
- [ ] Do I have a **premium-request budget** configured?

---

[← Back to README](../readme.md)
