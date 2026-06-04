# How-To: Add Deterministic Guardrails

LLMs are non-deterministic, so every step in an agent run has some chance of being wrong — and across a multi-step task those odds *multiply*, they don't average. A run that's right only if all 50 steps are right has a success rate of (step accuracy)⁵⁰: ~60% at 99%/step, collapsing to ~8% at 95%/step (see [Quality Economics](quality-economics.md)). The most powerful counter is a **deterministic control**: something that either passes or fails, every time, with no model judgment involved. Tests are the canonical example.

> **Goal:** give the agent objective checkpoints that catch errors *early*, before they stack into an incident.

---

## 1. Why a test resets the error compound

A test is deterministic: it fails or it doesn't. When the agent runs it, the result is unambiguous — and that **restarts the accuracy** of the run.

**With a unit test in place:**

```
buggy change → FAILING TEST → "stop, you can't continue"
            → correction → stable base → next change → PASSING TESTS → done
```

The failing test tells the agent *"stop and fix this"* before it builds further. Each green test is a known-good base to build on. If after 10 steps the agent has drifted to ~50% accuracy, the test snaps it back toward ~99%.

**Without tests:**

```
buggy change → buggy change 2 → buggy change 3 → buggy change 4 → 🔥
```

The agent stacks bad changes on bad changes. It may finish *faster* and with *fewer* tokens — but what you've produced is a **bug or an incident**, and the real cost shows up downstream:

- Wasted CI/CD minutes
- Extra Copilot review cycles and agent re-runs
- Human time to diagnose and fix

---

## 2. It's not just tests

Any guardrail you can encode in **code** and have the agent execute counts:

- **Unit / integration tests** — the highest-leverage control
- **Linters** — style and correctness, deterministically
- **Type checkers** — catch whole classes of errors before runtime
- **Security scanners** — flag vulnerabilities the model won't reliably notice
- **Build / compile steps** — a clean build is a pass/fail signal

The pattern is always the same: a **deterministic check the agent runs itself**, turning "I think this is right" into "this provably passes."

---

## 3. Make the guardrails discoverable

Guardrails only help if the agent actually runs them. Wire them in:

- Mention the exact test/build/lint commands in `copilot-instructions.md` (see [Externalize Custom Instructions](custom-instructions.md)).
- Add **stop signals** to your prompt: *"Once the bug is fixed and tests pass, stop."* (see [Ask Focused Questions](focused-questions.md)).
- Prefer a workflow that runs the checks automatically (pre-commit hooks, CI on every push).

---

## 4. This is shift-left, applied to agents

Teams that ship heavily with agents lean on tests precisely because they're deterministic — a large share of a healthy agentic codebase is test code, by design. The classic shift-left movement (move quality, testing, and security as early as possible) is *more* important in agentic systems, not less, because compounding errors punish late detection so harshly.

---

## Related

- [Think in Quality Economics](quality-economics.md)
- [Externalize Custom Instructions](custom-instructions.md)
- [Ask Focused Questions](focused-questions.md)
