# How-To: Power-User Token Tips

These are advanced techniques. They require more knowledge and testing, and several trade quality for token savings (or vice versa) — so treat them as conditional, not defaults. They pay off most for **AI engineers** orchestrating many agents, where every percent compounds across the fleet.

> **Goal:** squeeze static-token overhead and tool-call churn out of high-volume agent workflows.

---

## 1. Think in code

Prefer writing a **script** to analyze or filter data over feeding raw data to the model.

- Filter a REST API response down to the relevant fields *before* handing it to the agent, instead of dumping the full JSON into context.
- Pre-compute, grep, or transform with plain code — deterministic, free of model tokens, and free of model error.

Deterministic work belongs in deterministic tools. Reserve the model for judgment.

---

## 2. Consider CLIs vs. MCPs

CLI tools can be leaner than their MCP equivalents in some scenarios:

- A CLI like `gh`, `az`, or `kubectl` is **already known to the model** — no tool schema needs to sit in context.
- MCP tools re-send their **name + JSON schema on every turn**, which adds static tokens whether or not the tool is used.

For deterministic data fetching (a PR diff, resource JSON, logs), a CLI call is often a cleaner choice. Reserve MCP tools for steps that genuinely need the model to interpret or decide. See [Use Tools & Agents Efficiently](tools-and-agents.md).

---

## 3. Improve shell outputs

Shell output can be enormous, and all of it lands in context. Trim it to what the agent actually needs:

- Use built-in flags (`--output json`, `-o name`, `--quiet`, `| head`).
- Tools like [`rtk`](https://github.com/rtk-ai/rtk) reduce CLI output to agent-relevant information only.

---

## 4. Run `/chronicle tips` and `/chronicle cost-tips` regularly

In Copilot CLI, `/chronicle` analyzes your recent session history and surfaces opportunities to improve efficiency. Two focused variants ([source](https://docs.github.com/en/copilot/tutorials/optimize-ai-usage#7-utilize-learnings-to-be-more-efficient-at-every-turn)):

- **`/chronicle tips`** — surfaces opportunities to use Copilot more efficiently from your recent session history.
- **`/chronicle cost-tips`** — analyzes your token-usage patterns and suggests how to reduce cost.

Running them regularly turns your own usage history into a personalized optimization checklist. Use **`/context`** any time to check current context-window usage mid-session. When Chronicle identifies a recurring pattern, encode the smallest useful correction in your repository instructions; see [Externalize Custom Instructions](custom-instructions.md#7-turn-recurring-chronicle-findings-into-instructions).

---

## 5. Collapse tool calls

Each tool call is another turn, and each turn re-sends the conversation. Batching multiple tool calls into one reduces the number of loops. Plugins like [copilot-codeact-plugin](https://github.com/jsturtevant/copilot-codeact-plugin) can collapse several tool calls into a single step.

---

## 6. Model-specific context optimization

Different models respond to context differently and can be individually tweaked (prompt phrasing, context ordering, system-prompt style). This is **only** worth it for power users running thousands of agents — and it's fragile, because models change rapidly. Generally **not recommended** unless you have the scale to justify continuous re-tuning.

---

## When to reach for these

| You are…                                      | Use these tips?                           |
| --------------------------------------------- | ----------------------------------------- |
| Running a handful of agents/day               | Mostly skip — the basics matter more      |
| Orchestrating dozens–hundreds of async agents | Yes — small percentages compound at scale |

See [Think in Quality Economics](quality-economics.md) for the maturity spectrum.

---

## Related

- [Use Tools & Agents Efficiently](tools-and-agents.md)
- [Think in Quality Economics](quality-economics.md)
- [Future-Proof Your Skills](future-proofing.md)
