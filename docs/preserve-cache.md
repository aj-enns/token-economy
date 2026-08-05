# How-To: Preserve the Cache (Stop Paying Full Price to Re-Read Context)

Caching lets a model store portions of a conversation's context so they don't have to be reprocessed on every request. In agentic coding — where the same large context (system prompt, file contents, tool definitions) is re-sent across many turns — this matters a lot: the cached portion is reused instead of reprocessed, and **cached tokens are typically billed at ~10% of the normal input price** ([source](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage)).

> **Goal:** keep the cache warm so each turn re-reads context at ~10% price instead of rebuilding it at full input price.

📚 Source: [Optimizing your AI usage to maximize efficiency and reduce cost](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage) → "Preserve the cache"

---

## 1. Why the cache is a cost lever

Under UBB you pay for three token buckets (input, cached, output). Cached input is the cheap one — roughly **10%** of the input rate. A long agent session re-sends a big, mostly-unchanged context every turn. If that context stays cached, you pay the cheap rate to re-read it. If the cache is invalidated, the **entire** context is re-sent and billed as fresh, full-price input tokens ([source](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage)).

So cache discipline is a silent multiplier on every long session: get it right and the running context is cheap; break it and you re-pay for everything.

---

## 2. What invalidates the cache

Three common actions force a full, full-price context rebuild ([source](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage)):

| Action                                                                                      | Why it breaks the cache                                                                                   |
| ------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Switching models mid-session**                                                            | One model can't reuse another model's cache — the next request rebuilds from scratch.                     |
| **Coming back to an old session**                                                           | Caches expire after inactivity — **24h for OpenAI models, ~1h for most others**. A cold session rebuilds. |
| **Changing reasoning effort, context size, or the enabled tools / MCP servers mid-session** | Each of these changes the cached prefix, so it can't be reused.                                           |

---

## 3. Practical rules

- **Pick a model and stick with it** for the session. If you need to escalate to a stronger model, do it at a natural boundary (new session), not mid-task.
- **Configure before you start.** Set reasoning effort, context size, and the tool/MCP set up front, then leave them unchanged for the session.
- **Use `Auto` — it protects your cache.** Copilot auto model selection only changes models at natural cache boundaries (a new session, or after `/compact`), never mid-task ([source](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage)).
- **Returning to old work? `/compact` first.** When a session has gone cold, run `/compact` (Copilot CLI) so what gets rebuilt is a short summary rather than the full history. See [Prune Chat History](prune-history.md).
- **Run cheap subagents in their own session.** A subagent has its own scoped context and doesn't inherit the main conversation, so assigning it a lighter model doesn't churn the main agent's cache the way a mid-session switch would. See [Scoped Agents & Handoffs](scoped-agents.md).

---

## 4. Inspect cache behavior

Use VS Code's **Cache Explorer** to compare consecutive model turns and find the first point where their prompt prefixes diverge:

1. Open the Chat view's **…** menu and select **Show Agent Debug Logs**.
2. Select the session description to open its Summary view.
3. Select **Cache Explorer**.

Each turn shows its cache-hit percentage, duration, model, and timestamp. The prompt signature breaks the request into system instructions, tool definitions, and messages; expand the first changed component to see what invalidated the remaining prefix.

The surrounding **Summary** view also shows aggregate token usage, tool calls, errors, and session duration. See [Diagnose prompt caching with the Cache Explorer](https://code.visualstudio.com/docs/agents/agent-troubleshooting/cache-explorer).

### What Copilot optimizes automatically

The Copilot harness also reduces cache and context overhead without user configuration:

- Supported OpenAI models use extended prompt-cache retention for up to 24 hours.
- Anthropic requests place cache breakpoints at stable prompt boundaries and recent cacheable messages.
- Tool search can defer full tool schemas and append discovered tools after the stable prefix, preserving the earlier cache.

These are harness implementation details, not settings you configure in VS Code. In particular, `prompt_cache_retention` is a provider request parameter configured by the harness. See [Improving token efficiency for GitHub Copilot in VS Code](https://code.visualstudio.com/blogs/2026/06/17/improving-token-efficiency-in-github-copilot).

---

## 5. Anti-patterns to watch for

- 🚩 Toggling MCP servers / tools on and off mid-conversation to "try something."
- 🚩 Bumping reasoning effort up for one hard step, then back down — each flip rebuilds the cache.
- 🚩 Leaving a session for lunch and resuming hours later on a non-OpenAI model.
- 🚩 Manually hopping between models mid-task instead of letting `Auto` route at cache boundaries.

---

## Related

- [Understand AI Credits & Per-Token Pricing](ai-credits.md)
- [Prune Chat History](prune-history.md)
- [Choose the Right Model](choose-model.md)
- [Manage Context Attachments](manage-context.md)

---

[← Back to README](../readme.md)
