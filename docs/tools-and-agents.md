# How-To: Use Tools & Agents Efficiently

Agents shine when they call **tools** to do real work. They waste tokens — and your AI Credits — when they "think aloud" through what a tool already does. Under usage-based billing, every under-the-hood model call in an agent run generates billable input + output tokens.

> **Goal:** prefer tool calls over LLM reasoning, and keep agent chains short.

Primary references:

- Tool picker, tool limits, and best practices in VS Code: https://code.visualstudio.com/docs/copilot/agents/agent-tools
- Copilot UBB billing model: https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals

---

## 1. Prefer a real tool over LLM reasoning

If a tool, command, or API exists for the task, ask the agent to use it directly instead of inventing the answer.

| ❌ Token-heavy ask | ✅ Tool-first ask |
| --- | --- |
| "Figure out which tests fail" | "Run the test task and report failures" |
| "Walk me through what files changed" | "Run `git diff --stat` and summarize" |
| "Reason about which functions call `foo`" | "Find usages of `foo` and list them" |
| "Write and explain a KQL query for X" | "Use the App Insights tool to fetch X" |

The LLM stops narrating; it just orchestrates.

---

## 2. Keep agent chains short

Each planning / reflection step is another model call. Watch for runaway chains:

- 🚩 The agent restates the plan every turn.
- 🚩 The agent re-reads the same file repeatedly.
- 🚩 The agent runs the same test multiple times without changing code.

When you see this, **stop the agent**, summarize where you are, and start a focused new prompt.

![Agent stop / cancel control](images/agent-stop.png)


---

## 3. Give the agent a small, well-scoped goal

Agent prompts that begin with **"do everything for…"** explode into many tool calls.

❌ "Set up the whole project."

✅ "Create a `src/utils/date.ts` file exporting `formatIso(date: Date): string`. Add a unit test."

The smaller the scope, the fewer tools fire, the fewer tokens you spend.

---

## 4. Disable tools you don't need

Most chat surfaces let you turn individual tools on/off for a session. Select only the tools relevant to your prompt.

![Tool toggle UI](images/tool-toggles.png)

---

## 5. Use specialized agents/skills for specialized work

A single mega-agent loaded with every domain rule pays for that whole prompt **on every call**. Specialized sub-agents or skills only pull weight when invoked.

| Pattern | Cost shape |
| --- | --- |
| One monolith agent (auth + billing + ops + docs) | Fat prompt every call |
| Specialized skills (`/auth`, `/billing`, …) invoked on demand | Thin prompt by default |

In VS Code, define each role as a **custom `.agent.md`** with a restricted `tools` list and optional handoffs. See [Scoped Agents & Handoffs](scoped-agents.md) for the full pattern.

---

## 6. Watch the 128-tools-per-request limit

A chat request can have a maximum of **128 tools enabled** at a time. If you see an error about exceeding 128 tools per request, deselect tools or entire MCP servers in the tools picker.

---

## 7. Prune unused MCP tools (huge inefficiency)

Because LLM APIs are stateless, agent runtimes typically **re-send every enabled tool's name and JSON schema with every request**. For a GitHub-style MCP server with ~40 tools, that can add **10–15 KB of schema per turn**. If the agent only ever calls 2 of them, the other 38 are pure overhead on every call ([source](https://github.blog/ai-and-ml/github-copilot/improving-token-efficiency-in-github-agentic-workflows/)).

The GitHub Next team measured this on production agentic workflows: removing unused MCP tool registrations reduced per-call context by **8–12 KB**, saving **several thousand tokens per run** with no change in behavior. In one workflow (Smoke Claude), aggressive MCP pruning combined with a model-tier switch drove a **−79% token reduction**.

**What to do:**

- Start with a full tool-set if you must, then **prune to the narrow set the agent actually calls**. Audit which tools are invoked vs. registered.
- In VS Code, **disable whole MCP servers** you don't need for this chat in the tools picker — not just individual tools.
- For custom agents, set an explicit `tools:` allowlist in `.agent.md` rather than inheriting the workspace default.
- 🚩 Watch for a tool registered but **called zero times** across runs — it's costing you on every turn for no benefit.

> **Counter-example:** removing 8 unused GitHub MCP tools from one workflow yielded **no measurable ET savings** because the tool manifests were a small fraction of that workflow's context. Pruning helps most when tool schemas are a meaningful share of input tokens — measure before/after.

---

## 8. Prefer a CLI over MCP for data fetching

Removing unused tools is the cheap win. The bigger structural win for repetitive workflows is **replacing MCP data-fetch calls with CLI calls** ([source](https://github.blog/ai-and-ml/github-copilot/improving-token-efficiency-in-github-agentic-workflows/)).

Why MCP is expensive for plain data retrieval:

- An MCP tool call is **a reasoning step on top of the data fetch**. The model has to decide to call it, formulate arguments, and consume the response as context — a full LLM round-trip that bills input + output tokens for the tool-use schema, the arg block, and the result.
- A CLI call (`gh pr diff`, `az ... --output json`, `kubectl get ... -o json`) is a **deterministic HTTP request with no LLM involvement**.

Two patterns the GitHub team uses:

1. **Pre-fetch before the agent starts.** If you always need a PR diff, the list of changed files, recent logs, etc., run a `gh` / `az` / `kubectl` command in a setup step and write the result to a workspace file. The agent reads the file instead of making an MCP call. Removes the tool-call overhead entirely and lets the agent use its strong shell/text-processing training on the static artifact.
2. **In-agent CLI substitution** for runtime-determined fetches. When the agent decides at runtime what to pull, point it at the CLI (`gh pr view --json`, etc.) instead of an MCP tool that does the same thing. Same data, no schema overhead in context, no extra reasoning step.

**Rule of thumb:**

| Task | Prefer |
| --- | --- |
| Deterministic data fetch (diff, file contents, issue body, logs, resource list) | **CLI** (`gh`, `az`, `kubectl`, `git`, …) |
| Action that requires the model to interpret/decide (triage, summarize, route) | **MCP tool** or model reasoning |
| Data you'll always need | **Pre-fetch** before the agent runs |
| Data the agent picks at runtime | **CLI inside the agent**, not MCP |

The principle: **the cheapest LLM call is the one you don't make**. Move deterministic reads out of the LLM reasoning loop.

---

## 9. Remember what is billed in agent mode (UBB)

Under **usage-based billing** (starting June 1, 2026), every model call an agent makes generates billable **tokens** — input, output, and cached — converted to AI Credits ([source](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)). A complex agentic session working across a large codebase will consume significantly more than a quick chat question, because it involves many under-the-hood model calls.

> **Compare with the prior model:** under request-based billing, only the prompts *you* entered counted as premium requests; tool calls / background steps in agent mode were not charged. Under UBB, those calls do generate tokens, so keeping the chain short matters more than ever.

---

## 10. Offload deterministic work to plain code

Anything that doesn't need an LLM, shouldn't use one:

- Date math → use `date-fns`, not the model.
- JSON shape changes → use a script, not a prompt.
- Lookups → use a database query, not "what is X in the docs?".

Save the LLM for genuine reasoning.

---

[← Back to README](../readme.md)
