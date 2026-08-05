# How-To: Understand How Context Windows Work

You can't optimize what you don't understand. Most token waste comes from a few mechanics that aren't obvious: agents are **stateless**, conversations are **re-sent in full every turn**, and models **don't read the whole window evenly**. This doc is the mental model behind every other optimization in this repo.

> **Goal:** understand what actually gets sent to the model on each turn, and why bigger context is not better.

---

## 1. An agent is just an app talking to a stateless model

There's no magic — just text. Two pieces:

- **The harness** (a.k.a. the agent): the app you interact with. VS Code Chat, Copilot CLI, the Copilot cloud agent, or third-party harnesses like Claude Code and Codex.
- **The LLM**: the model itself — GPT-5.5, Claude Opus, Gemini Pro, etc.

The harness talks to the LLM **on your behalf, many times**, before it returns to you. And critically, the LLM is **stateless**:

- It does **not** store your conversation.
- "Having a conversation" really means **re-sending the entire ordered history** — every prior input and output — on every single turn.
- So tokens and context **compound** as the session grows.

Your levers to influence the result: **your prompt**, the **files in your project**, and your **agent configs** (instructions, skills, MCPs). See [Agent Configs Explained](agent-configs.md).

---

## 2. What gets sent on each loop

On every loop the harness sends the *whole* conversation to the LLM again:

**Loop 1**

```
[ system prompt + tool descriptions ]   ← input tokens
[ your prompt ]                          ← input tokens
[ file references / attachments ]        ← input tokens
        → model responds                 ← output tokens
```

**Loop 2 and beyond**

```
[ everything from Loop 1 ]
[ previous response(s) ]
[ new input ]                            ← all re-sent as input tokens
        → model responds                 ← output tokens
```

Some of those input tokens may be **cached** (cheaper, but not guaranteed). The takeaway: don't get lost in tokenization details you can't control. Think at the level you *can* control — **prompts, files, and responses all consume tokens, and they compound with each loop.**

### The prompt prefix

Much of the beginning of an agent request repeats across turns: system instructions, tool definitions, repository context, and conversation history. This stable beginning is the **prompt prefix**. When consecutive requests share the same prefix, the model provider can reuse cached model state rather than process that portion again.

The match stops at the first divergence. A changed instruction, model, reasoning setting, or tool configuration near the beginning can make everything after it a cache miss. Keep stable configuration at the front of the session and add volatile material, such as attachments and command output, later. See [Preserve the Cache](preserve-cache.md) and the official [VS Code explanation of prompt-prefix caching](https://code.visualstudio.com/blogs/2026/06/17/improving-token-efficiency-in-github-copilot#how-agentic-requests-spend-tokens).

A rough sense of scale:

- 1 token ≈ ¾ of an English word (a fine mental model is "1 token ≈ 1 word").
- Context limits vary: smaller models handle ~50–200K tokens; larger ones (Claude Opus, GPT-5.5) handle ~1M.
- 1M tokens ≈ the *Lord of the Rings* trilogy plus *The Hobbit*.

---

## 3. Just because you *can* fill the window doesn't mean you should

Models do **not** read the context window evenly. They bias toward the **beginning** and the **end**, and discount the **middle** — an effect often called **context rot** ([Context Rot, Product Talk](https://www.producttalk.org/context-rot/)).

### 3.1 "Lost in the Middle" — below ~50% full

The model favors the start and end of the window. Usually that's fine:

- **Beginning** = your instructions, goals, and plan (you *want* these prioritized).
- **End** = the current work stream (also important).
- **Middle** = older work (less relevant).

The problem shows up when you **switch tasks mid-session**. Start with a bug fix, then say "now implement a feature." As the window grows, the model may snap back to the *bug fix*, because it over-weights the original statement at the top.

> **Fix:** use a **new context window for each distinct task.**

### 3.2 Recency bias — above ~50% full

Past roughly half capacity, the model starts favoring **only the end** of the conversation. It begins forgetting your system prompt, custom instructions, and original goal — and **drifts**, doing things that seem to come from nowhere.

> **Fix:** try not to let a working session grow past **~60–70%** of the window unless you really need to. Divide-and-conquer your tasks from the start (see [Research → Plan → Implement](research-plan-implement.md)); compacting can help as a last resort, but it loses information.

---

## 4. Don't deal in absolutes

Bias is a *tendency*, not a guarantee. The model won't always forget the middle, and there are plenty of legitimate reasons to run at 60–80% of the window. Treat these as optimization knobs you reach for deliberately — not hard rules.

---

## Related

- [Prune Chat History](prune-history.md)
- [Manage Context Attachments](manage-context.md)
- [Work in Phases: Research → Plan → Implement](research-plan-implement.md)
- [Agent Configs Explained](agent-configs.md)
