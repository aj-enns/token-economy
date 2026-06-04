# How-To: Think in Quality Economics (Make Every Token Count)

Most teams react to usage-based billing (UBB) by asking *"how do we spend fewer tokens?"* That's the wrong first question. Optimizing cost while the **value** of an agent's output is low is wasted effort. The better question: **"how do we get the most out of every token we spend?"** — and it turns out that higher quality usually *also* lowers spend.

> **Goal:** stop optimizing for cheap tokens. Optimize for quality, and lower spend follows.

---

## 1. The ROI lens

Think of an agent run the way you'd think of any investment:

$$\text{Agent ROI} = \frac{\text{Value of agent output} - \text{Token cost}}{\text{Token cost}} \times 100\%$$

You usually can't calculate this cleanly — quantifying the value of an agent's output is hard, and most teams (including GitHub) don't have a perfect method. But you don't need the exact number for the lesson to hold:

- If output value is near zero, **no amount of token-trimming makes the run worthwhile**.
- Raising quality raises the numerator far faster than shaving tokens shrinks the denominator.

So spend your energy making each run *land*, not making a missed run cheaper.

---

## 2. Higher quality often means *fewer* tokens anyway

The two goals aren't in tension. The most common quality problems are also the most common sources of waste:

| Quality problem | Token cost it creates |
| --- | --- |
| Stuffing irrelevant "might need it" context into the prompt | Pay for it on **every turn** (it compounds) and bias the model toward wrong answers |
| Letting a chat sprawl across unrelated tasks | Re-send a huge transcript each reply, and trigger context drift |
| Vague prompts that need several correction rounds | Extra agent loops, each generating more billable tokens |

**Trimming context is the first big lever for both better answers *and* lower spend.** You're not choosing between quality and cost — you're fixing both at once.

---

## 3. The compounding-error problem

Here's the structural reason quality matters more in agentic work than it did in single-shot chat.

LLMs are non-deterministic. They have an error margin and are never 100% accurate. In a multi-step agent workflow, those per-step errors **compound**:

| Accuracy per step | After 10 steps | After 50 steps |
| --- | --- | --- |
| 99% | ~90% | ~60% |
| 95% | ~60% | ~8% |

A 50-step workflow at a very optimistic 99%/step still only lands ~60% of the time. Drop to 95% and you're at ~8%.

This doesn't mean every agent fails — it means **every percentage point of per-step quality you add dramatically increases the odds the whole run succeeds.** And every miss isn't free: it wastes the tokens already spent, plus the bug fixes, reviews, re-runs, and human time to clean up. This is the agentic version of the classic **shift-left** principle — move quality, testing, and security as early as possible.

> Deterministic guardrails (tests, linters, scanners) are the most powerful counter to compounding errors. See [Add Deterministic Guardrails](deterministic-guardrails.md).

---

## 4. Match the effort to your maturity

How much these optimizations matter depends on how you work:

| | **AI-assisted engineer** | **AI engineer** |
| --- | --- | --- |
| Mode of work | One agent at a time, mostly synchronous | Orchestrates many asynchronous agents |
| Volume | A handful of runs per day | Dozens to hundreds per day |
| Payoff of optimizing | Low — saving 50% of a $20/mo spend is $10 | High — every 1% of tokens or quality multiplies across the fleet |

If you're on the left of this spectrum, the first few levers (model choice, clear prompts, fresh chats) are enough. The deeper techniques pay off as you scale up the number of agents you run.

---

## 5. The mindset

> **Instead of counting tokens, make every token count.**

Yes, reduce token size — but let *quality* drive the reduction, not cost. Send fewer, higher-accuracy "rockets," and the fuel savings follow automatically.

---

## Related

- [Understand AI Credits & Per-Token Pricing](ai-credits.md)
- [Add Deterministic Guardrails](deterministic-guardrails.md)
- [Work in Phases: Research → Plan → Implement](research-plan-implement.md)
- [How Context Windows Work](context-windows.md)
