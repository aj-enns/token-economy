# How-To: Apply FinOps to Production Agents

Optimizing a Copilot chat and operating a production agent system are different problems. Production systems need to attribute usage, evaluate complete workflows, and enforce runtime controls across models, tools, agents, and infrastructure.

> **Goal:** optimize cost per successful task without sacrificing quality, latency, or reliability.

This guide summarizes architecture principles from [Token economics: The new FinOps for agentic AI](https://techcommunity.microsoft.com/blog/azuredevcommunityblog/token-economics-the-new-finops-for-agentic-ai/4533743). Product-specific controls must still be verified against the platform that runs the application.

---

## 1. Measure successful outcomes, not isolated calls

Tokens per request miss the cost of retries, tool calls, failed runs, and human recovery. Use **cost per successful task** as the primary economic measure, supported by:

| Dimension  | Measures                                                    |
| ---------- | ----------------------------------------------------------- |
| Cost       | Cost per completed and successful task                      |
| Tokens     | Input, output, cached, and cache-write tokens               |
| Quality    | Pass rate, evaluation score, and regression rate            |
| Efficiency | End-to-end latency, model calls, tool calls, and retries    |
| Governance | Budget breaches, throttles, circuit breakers, and rollbacks |

A cheaper workflow with a materially lower success rate is not an optimization.

---

## 2. Compress context into structured artifacts

Do not pass every source document or prior transcript to every step. Extract the state the next step needs into JSON, a table, or a typed schema:

- Remove duplicated, stale, and irrelevant content.
- Preserve source references for audit and recovery.
- Validate required fields and schema fidelity.
- Keep a fallback path to the original source when compression loses information.

Measure prompt reduction together with task success. Compression that drops a required constraint merely moves cost into rework.

---

## 3. Reuse stable work with explicit invalidation

Agent pipelines often repeat extraction and retrieval over unchanged documents. Cache reusable artifacts by content hash or another stable key, then define:

- The conditions for a cache hit.
- Expiration appropriate to the source data.
- Invalidation when the source or extraction schema changes.
- Monitoring for stale-cache failures.

Share structured artifacts through a memory or object store instead of copying full transcripts between agents.

---

## 4. Route by task, complexity, and risk

Use the least expensive model that reliably satisfies each step, with an escalation path when it does not:

| Workload                                | Typical route                        |
| --------------------------------------- | ------------------------------------ |
| Classification and keyword matching     | Rules, code, or a small model        |
| Summarization and structured extraction | Mid-tier model                       |
| Coding and multi-step orchestration     | High-capability or specialized model |
| Deterministic validation                | Tests, linters, scanners, or scripts |

Track routing accuracy, quality by route, escalation frequency, and end-to-end success. Unit price alone does not reveal whether a route is economical.

---

## 5. Define handoff contracts

Each agent should receive the next useful artifact, not every earlier conversation. A handoff contract should identify:

- The user's goal and accepted plan.
- Inputs, outputs, and source references.
- Completed and pending actions.
- Validation criteria and known failure reasons.
- The schema version of the handoff artifact.

For a coding workflow, a requirements agent can produce a specification, an implementation agent can consume that specification, and a testing agent can consume changed files plus acceptance criteria. See [Research → Plan → Implement](research-plan-implement.md).

---

## 6. Make metering a runtime control

Reporting after the bill arrives is insufficient for autonomous workflows. Meter every model call and attribute it to the scenario, agent, model, and task. Then define actions at budget thresholds:

1. Warn as a task approaches its budget.
2. Route eligible work to a cheaper model or deterministic implementation.
3. Throttle repeated or low-priority work.
4. Stop retry loops with a circuit breaker.
5. Preserve state and roll back partial work when the budget is exceeded.

Treat budgets as one guardrail among quality, security, and reliability controls. A cost limit should stop work predictably, not leave an untraceable partial result.

---

## 7. Evaluate the complete pipeline

Compare a baseline and optimized workflow using representative tasks. Record quality, latency, retries, tool calls, token categories, and total cost. Promote an optimization only when it meets the required quality and reliability thresholds.

Platform features such as Microsoft Foundry model router, Toolbox, or Agent Optimizer are implementation options for Foundry applications; they are not GitHub Copilot controls. Keep product behavior and citations separate when applying these principles.

---

## Related

- [Think in Quality Economics](quality-economics.md)
- [Add Deterministic Guardrails](deterministic-guardrails.md)
- [Use Tools & Agents Efficiently](tools-and-agents.md)
- [Power-User Token Tips](power-user-tips.md)

---

[← Back to README](../readme.md)
