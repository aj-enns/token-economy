# How-To: Scope Agents & Use Plan → Implement Handoffs

Scoped workflows reduce mistakes and wasted iterations by giving Copilot the right tools and guardrails for each step (planning, implementing, reviewing).

This doc focuses on *documented* mechanisms:

- **Custom agents** (`.agent.md`) with tool restrictions and optional **handoffs** ([Custom agents in VS Code](https://code.visualstudio.com/docs/copilot/customization/custom-agents))
- **Plan mode → Start Implementation** in Copilot Chat ([Asking GitHub Copilot questions in your IDE](https://docs.github.com/en/copilot/how-tos/chat-with-copilot/chat-in-ide))

---

## 1. Why scope agents

### 1.1 Least privilege (reduce accidental edits)

Custom agents let you set up a read-only planning role and a separate implementation role with edit tools, so the planning step cannot modify files by accident ([Custom agents in VS Code](https://code.visualstudio.com/docs/copilot/customization/custom-agents)).

### 1.2 Stay under tool limits

VS Code enforces a hard limit of **128 tools per request**. If you exceed the limit, you need to deselect tools or whole MCP servers in the tools picker ([Use tools with agents](https://code.visualstudio.com/docs/copilot/agents/agent-tools)).

### 1.3 Billing mental model (Copilot Chat under UBB)

Under **usage-based billing** (starting June 1, 2026), every model call in an agent run generates billable tokens — input, output, and cached — converted to AI Credits ([Usage-based billing for individuals](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)). Scoping agents with restricted tools and a smaller goal directly reduces:

- **Input tokens** per call (smaller agent prompt, fewer attached tools)
- **Number of calls** per task (less re-planning and re-reading)
- **Output tokens** per call (focused goal → focused response)

> **Compare with the prior model:** under request-based billing, only the prompts *you* entered counted as premium requests; tool calls / background steps in agent mode were not charged. Under UBB, those calls do generate tokens, so scoping matters more.

---

## 2. Option A (recommended): Plan mode → Start Implementation

The built-in **Plan** agent is designed to produce an implementation plan before any edits. After you review the plan, click **Start Implementation** to switch into agent mode and implement it ([Asking GitHub Copilot questions in your IDE](https://docs.github.com/en/copilot/how-tos/chat-with-copilot/chat-in-ide)).

Workflow:

1. Switch the agent dropdown to **Plan**
2. Ask for a plan
3. Review and clarify
4. Click **Start Implementation**

---

## 3. Option B: Custom agents + handoffs

Workspace-level custom agents live in `.github/agents/` ([Custom agents in VS Code](https://code.visualstudio.com/docs/copilot/customization/custom-agents)).

### 3.1 Example: Planner (read-only)

Create `.github/agents/planner.agent.md`:

```md
---
name: Planner
description: Produce an implementation plan. Do not edit files.
tools: ['read', 'search', 'web']
handoffs:
  - label: Start Implementation
    agent: implementer
    prompt: Implement the approved plan above. Keep changes minimal.
    send: false
---

Produce a concise, numbered plan.

Rules:
- Do not edit files or run terminal commands.
- Ask up to 3 clarifying questions if needed.
- Stop after the plan; wait for handoff.
```

### 3.2 Example: Implementer (editing + execution)

Create `.github/agents/implementer.agent.md`:

```md
---
name: Implementer
description: Implement an approved plan using edit + execution tools.
tools: ['edit', 'execute', 'read', 'search']
---

Follow the approved plan precisely.

Rules:
- Keep scope minimal.
- Prefer small diffs over rewrites.
- If a step fails, report the error and propose the smallest fix.
```

---

## 4. Checklist

- [ ] Use **Plan → Start Implementation** for complex tasks
- [ ] Keep a read-only planner separate from an implementer
- [ ] Keep tools under the **128 tools/request** limit
- [ ] Run **subagents on cheaper models** — they have their own scoped context and don't churn the main agent's cache ([Choose the Right Model](choose-model.md), [Preserve the Cache](preserve-cache.md))
- [ ] Remember: under UBB, every model call in an agent run generates billable tokens — a small, scoped goal keeps total tokens (and AI Credits) down

---

[← Back to README](../readme.md)
