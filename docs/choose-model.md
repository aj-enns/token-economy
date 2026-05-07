# How-To: Choose the Right Model

Premium models can cost **multiples per token** vs. lighter ones. Matching the model to the task is one of the biggest cost levers you have.

> **Goal:** use the cheapest model that produces a good-enough answer.

---

## 1. Switch the model in VS Code Chat

**VS Code:**

- At the bottom of the Chat input, click the **model picker** (shows current model name).
- Pick a lighter model for routine work, premium for hard problems.

![Model picker in the Chat input](images/model-picker.png)


---

## 2. Quick model-selection guide

| Task | Suggested tier |
| --- | --- |
| Renames, syntax help, doc lookups, simple Q&A | **Light** (e.g. GPT-4.1-mini, Haiku-class) |
| Single-file edits, small refactors, test scaffolding | **Mid** (e.g. GPT-4.1, Sonnet-class) |
| Multi-file refactors, architectural design, deep debugging | **Premium** (e.g. GPT-5, Opus-class) |
| Repetitive code completions while typing | **Inline ghost text** (unmetered) |

> Names change over time — apply the *tier* rule, not specific model names.

---

## 3. Try the cheap model first

Workflow:

1. Ask your question with a **light** model.
2. If the answer is wrong or shallow, escalate to a **mid** or **premium** model in the same chat (the model picker switches mid-conversation).
3. Note the pattern — next time, start at the right tier.

---

## 4. Use unmetered inline completions for everyday code

Inline ghost-text completions (the gray suggestions while you type) are **unmetered** under current Copilot plans. For routine code, accept inline suggestions instead of opening Chat.

![Inline ghost-text completion](images/inline-completion.png)
> 📸 **Screenshot needed:** `docs/images/inline-completion.png` — editor showing gray ghost-text suggestion mid-line.

---

## 5. Set a default model per workspace (optional)

VS Code remembers the last-used model per chat. If your workspace is mostly routine, leaving the default on a light model nudges you toward cheaper calls.

---

[← Back to README](../readme.md)
