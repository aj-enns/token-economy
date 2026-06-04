# How-To: Work in Phases — Research → Plan → Implement

Doing research, planning, and implementation in one long chat carries a pile of irrelevant context through every turn — degrading quality (context rot) and re-billing tokens (even cached ones) on each loop. Splitting the work into phases with **fresh context windows** keeps each step lean and accurate.

> **Goal:** carry only the context each phase needs, so quality stays high and tokens stay low.

---

## 1. The three phases

| Phase | What it does | Why it bloats context |
| --- | --- | --- |
| **Research** | Explores the codebase to find what's relevant ("I want to change X — what files matter?") | Loads many files, most of which won't matter for implementation |
| **Plan** | Turns findings into a precise spec — a detailed to-do list that front-loads the thinking | Benefits from a reasoning model viewing the plan from every angle |
| **Implement** | Executes the spec | Only needs the spec + the few relevant files |

If you do all three in one session, the implementation phase is still dragging the entire research dump behind it.

---

## 2. Start a fresh window between phases

Yes, this duplicates a little context across windows — but you trade a small, one-time re-send for **dramatically less compounding context** and better focus in each phase.

```text
Window 1 (Research)   → produces a short findings note / file list
Window 2 (Plan)       → consumes findings, produces a precise spec
Window 3 (Implement)  → consumes the spec, makes the changes
```

Pass the *output* of each phase forward (a findings note, then a spec file), not the raw transcript.

---

## 3. Match the model to the phase

- **Research / Plan / debug:** use a **reasoning model** (something with strong reasoning — it doesn't always have to be the top-tier model). Reasoning models excel at viewing a plan from every angle and finding gaps.
- **Implement:** once the spec is tight, a **mid- or low-tier model** is often *better* — a reasoning model may re-open the plan, second-guess the spec, and "go rogue." See [Choose the Right Model](choose-model.md).

> Use **Plan mode → Start Implementation** in Copilot Chat to get a reviewable plan before any edits. See [Scoped Agents & Handoffs](scoped-agents.md).

---

## 4. The payoff: parallel implementation

A precise spec unlocks something you can't do with a vague prompt — **splitting the work across multiple agents in parallel**:

- Split by architecture layer (frontend, backend, database).
- Define the contracts between components in the spec.
- Each agent works with **only its relevant context**.

This saves both wall-clock time and tokens, because no agent is carrying knowledge it doesn't need for its slice of the work.

---

## 5. Why this saves tokens

| Without phases | With phases |
| --- | --- |
| Research files re-sent on every implementation turn | Research context discarded before implementation starts |
| One giant window drifts past 50% → recency bias | Each window stays small and focused |
| Reasoning model used end-to-end (priciest) | Reasoning only where it adds value; cheaper model for execution |

---

## Related

- [Choose the Right Model](choose-model.md)
- [How Context Windows Work](context-windows.md)
- [Scoped Agents & Handoffs](scoped-agents.md)
- [Prune Chat History](prune-history.md)
