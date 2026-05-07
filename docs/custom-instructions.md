# How-To: Externalize Custom Instructions

Anything in your "always-on" instructions is sent on **every chat turn**. Keep persistent instructions tiny; move the rest to on-demand skills, prompts, or per-file instruction files.

> **Goal:** stop paying for boilerplate you don't need on every call.

---

## 1. Audit your current custom instructions

**VS Code:** open `.github/copilot-instructions.md` (or your `*.instructions.md` files) and ask:

- Does **every** chat turn need this? If no → move it out.
- Is it a **role description** ("You are a senior engineer who…")? Usually trim-able.
- Is it a **style rule** that only applies to one language/folder? Move to a scoped instructions file.

---

## 2. Use scoped `.instructions.md` files with `applyTo`

Instead of one global instructions file, use multiple scoped ones. Each only loads when relevant files are in context.

Example: `.github/instructions/typescript.instructions.md`

```markdown
---
applyTo: "**/*.ts,**/*.tsx"
---

- Use `unknown` over `any`.
- Prefer named exports.
- No default exports for components.
```

This rule only loads when a `.ts`/`.tsx` file is part of the conversation.

---

## 3. Move "how to do X" knowledge into prompt files

For workflows you only run sometimes (e.g. *"write a release note"*, *"generate a PR description"*), use a `.prompt.md` file instead of stuffing the steps into global instructions.

Example: `.github/prompts/release-notes.prompt.md`

```markdown
---
mode: ask
---

You are generating release notes. Group changes under
**Features**, **Fixes**, **Chores**. Use the diff in context.
```

Invoke it on demand from the chat input — it costs **zero** tokens until used.

![Prompt file picker](images/prompt-file-picker.png)
> 📸 **Screenshot needed:** `docs/images/prompt-file-picker.png` — chat input showing the prompt-file picker.

---

## 4. Move domain knowledge into skills (`SKILL.md`)

For larger bodies of knowledge ("how our deployment works", "our coding conventions deep-dive"), use a **skill**. Skills are loaded into context **only** when their description matches the user's request.

Rough sizing guide:

| Content | Where it should live |
| --- | --- |
| 1–2 lines, applies to all chats | Global `copilot-instructions.md` |
| Language- or folder-specific rules | Scoped `*.instructions.md` with `applyTo` |
| Multi-step workflow run on demand | `*.prompt.md` |
| Larger domain knowledge / playbooks | Skill (`SKILL.md`) |

---

## 5. Quick wins to remove from global instructions

These are common bloat sources — almost always cheaper somewhere else:

- ❌ "You are an expert in X, Y, Z, A, B, C…"
- ❌ Long lists of *every* coding convention.
- ❌ Full architecture overviews.
- ❌ Sample code blocks "for reference".
- ❌ Politeness preambles ("Always be polite and thorough…").

A 300-token global instruction removed = **300 tokens saved on every chat turn, forever.**

---

## 6. Verify what's actually being sent

In VS Code, you can inspect the resolved context for a turn:

- Open the **Output panel** → channel **GitHub Copilot Chat** (or **Language Model** logs) to see what was sent.
- Look for instruction blocks; trim anything that doesn't earn its place.

![Copilot Chat output channel](images/output-channel.png)


---

[← Back to README](../readme.md)
