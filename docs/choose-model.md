# How-To: Choose the Right Model

In Copilot Chat, different models consume premium requests at different multipliers. Picking the right model is one of the biggest cost controls you have. 

> **Goal:** use the cheapest model that produces a good-enough answer.

---

## 1. Switch the model in VS Code Chat

**VS Code:**

- At the bottom of the Chat input, click the **model picker** (shows current model name).
- Pick a lighter model for routine work, premium for hard problems.
- For mixed-complexity sessions, choose **`Auto`** — it lets Copilot auto model selection choose the best available model. On paid plans, auto model selection qualifies for a 10% premium-request multiplier discount.

![Model picker in the Chat input](images/model-picker.png)


> 💡 GitHub Copilot meters chat via **premium requests** with per-model multipliers — the model you pick directly controls how much of your monthly allowance each turn consumes. See [Understand Premium Requests](premium-requests.md).

---

## 2. Quick model-selection guide

| Task | Suggested tier |
| --- | --- |
| Renames, syntax help, doc lookups, simple Q&A | **Light** (e.g. GPT-5 mini, Haiku-class) |
| Single-file edits, small refactors, test scaffolding | **Mid** (e.g. GPT-4.1, Sonnet-class) |
| Multi-file refactors, architectural design, deep debugging | **Premium** (e.g. GPT-5.4 / 5.5, Opus-class) |
| Repetitive code completions while typing | **Inline suggestions** (limited on Copilot Free) |

> Names change over time — apply the *tier* rule, not specific model names.

---

## 3. Try the cheap model first

Workflow:

1. Ask your question with a **light** model.
2. If the answer is wrong or shallow, escalate to a **mid** or **premium** model in the same chat (the model picker switches mid-conversation).
3. Note the pattern — next time, start at the right tier.

---

## 4. Use inline suggestions for everyday code

Inline ghost-text completions (the gray suggestions while you type) are a separate surface from Chat.

- Copilot Free includes up to **2,000** real-time code suggestions per month.
- Paid Copilot plans include real-time code suggestions with the included models.

For routine code, accept inline suggestions instead of opening Chat.

![Inline ghost-text completion](images/inline-completion.png)
> 📸 **Screenshot needed:** `docs/images/inline-completion.png` — editor showing gray ghost-text suggestion mid-line.

---

## 5. Set a default model per workspace (optional)

VS Code remembers the last-used model per chat. If your workspace is mostly routine, leaving the default on a light model (or `Auto`) nudges you toward cheaper calls.



[← Back to README](../readme.md)
