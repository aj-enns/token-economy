# How-To: Agent Configs Explained

"Context engineering" is, in practice, mostly about **agent configs** — the markdown files and controls an agent takes into account automatically. They differ in one crucial way: **when they're loaded into the context window.** Always-on configs cost tokens on *every* turn; lazy-loaded ones only cost tokens when they're actually needed. Knowing which is which is how you keep context lean.

> **Goal:** put the right guidance in the right config so it's present when needed and absent when not.

---

## 1. The taxonomy at a glance

| Config | Loaded… | Best for | Token impact |
| --- | --- | --- | --- |
| **Persistent instructions** (`copilot-instructions.md`, `AGENT.md`) | **Always on** | Non-negotiables, agent-miss fixes, output trimming | High — re-sent every turn, keep tiny |
| **Custom agents** (`*.agent.md`) | When you invoke them | Forcing a role/workflow; restricting tools | Medium — manual, scoped |
| **Skills** (`SKILL.md`) | **Lazy** — when the task matches | Capabilities the agent lacks | Low — description always on, body on demand |
| **MCP servers** | When activated | 3rd-party tools / external APIs | Medium–High — tool schemas bloat context |
| **Subagents** | When spawned | Offloading task-specific context (e.g. research) | Trade-off — spend tokens to protect main session |
| **Scoped instructions** (`*.instructions.md`) | By file-path match | Monoliths with distinct sections | Conditional |
| **Prompt files** (`*.prompt.md`) | Manually invoked | Reusable "starting point" prompts | Low |
| **Copilot Memory** | Always on (auto-learned) | Small patterns learned from your behavior | Low |

---

## 2. Persistent instructions — your always-on guidance

These sit in the window for **every** session, so they have outsized influence (and outsized cost). Keep them tiny.

**What belongs:**

- **Non-negotiables** — guardrails every agent must follow.
- **Agent-miss prevention** — recurring corrections (wrong test framework, wrong build command).
- **Output trimming** — *"be concise," "drop niceties," "only return code."* Output tokens are the priciest bucket.

**Best practices:**

1. **Don't use AI to generate them.** This is your human chance to fill gaps the model can't know; AI-generated instructions are wordy and imprecise.
2. **Write them yourself**, from real observed agent behavior.
3. **Iterate** — add corrections as you spot misses; it needn't be perfect at first.
4. **Recreate them periodically.** Models and your project change; stale instructions compound useless context. Treat it as a living document.

> Research note: a blunt *"be concise"* yields almost the same trimming as a much longer style skill. Short wins.

See [Externalize Custom Instructions](custom-instructions.md).

---

## 3. Custom agents — force a role, restrict tools

A custom agent (`*.agent.md`) is best thought of as **manually invoked by you** to make an agent behave a specific way — e.g. a TDD agent scoped to only write failing ("red") tests, which an agent wouldn't do on its own.

When you invoke one (e.g. `/tdd-red add API endpoint`), the harness loads the agent file and **adjusts the available tools**. The real benefit isn't the small token savings from fewer tools — it's **preventing the agent from going down a path you don't intend**. If it should only *read* an issue, don't give it the write tool. See [Scoped Agents & Handoffs](scoped-agents.md).

---

## 4. Skills — lazy-loaded capabilities

A skill is like a custom agent's instruction set, but **offered to the agent based on the task** rather than always on. The harness keeps only the **skill description** in context; when the LLM detects a matching task ("work on the API"), it asks the harness to **load the full skill**.

**Best practices:**

- Don't overdo it — you don't need hundreds.
- Avoid redundant skills (does the model really need a "React skill" when it's already proficient in React?).
- Use skills only for capabilities the agent **wouldn't otherwise have**.
- Maintain them — as models improve, some skills become unnecessary.

---

## 5. MCP servers — external tools, used rigorously

MCPs add external API calls and dynamic tools. Once active, their **tool descriptions go into context** and stay there. Example: the GitHub MCP's `get_issue` tool lets *"read issue #45"* trigger a real API call.

**Be rigorous:**

- They **bloat tool descriptions** — pure token overhead on every turn.
- More importantly, extra tools can **lure the agent into calling things you didn't want**.
- Deactivate MCPs you don't always need, or scope them inside a custom agent.

> **Example — Playwright:** powerful for frontend work (it can view and autocorrect against a live page), but costly — screenshots and page reads burn tokens. Left always-on, it may trigger needless work (reading a webpage for a one-line CSS change). Better: enable it only inside a custom agent, when actually needed.

See [Use Tools & Agents Efficiently](tools-and-agents.md).

---

## 6. Subagents — a second window for offloaded work

A subagent opens a **separate context window** for a specific task (e.g. research), so the main session never fills with irrelevant detail. The subagent reads the documents, builds a **summary**, and returns only that summary to the main session.

This improves main-session quality, at the cost of the tokens spent inside the subagent. Use it **cautiously** — it's a conditional optimization. The agent often spawns one automatically, but you can also invoke one explicitly for research.

---

## 7. The rest (lower impact, still worth knowing)

- **Scoped instructions** (`*.instructions.md`) — apply by file-path pattern; offered to the agent non-deterministically (like skills). Useful for monoliths with distinct sections. Start with static instructions; only move to scoped if they get too long.
- **Prompt files** (`*.prompt.md`) — reusable, manually invoked prompts. A good "manual starting point" to kick off custom agents or skills with a common prompt. (Not supported in Copilot CLI; often a skill or custom agent is the better choice.)
- **Copilot Memory** — small, always-on instructions auto-learned from your behavior and team patterns, applied uniformly across Copilot surfaces. Little to proactively tune, but worth checking periodically.

---

## Related

- [Externalize Custom Instructions](custom-instructions.md)
- [Scoped Agents & Handoffs](scoped-agents.md)
- [Use Tools & Agents Efficiently](tools-and-agents.md)
- [How Context Windows Work](context-windows.md)
