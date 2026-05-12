---
name: "fact-check"
description: "Fact-check GitHub Copilot claims against official sources (docs.github.com first), flag unsourced statements, and propose safer rewrites. Use when: verifying Copilot pricing, premium requests, model availability, settings keys, agent mode behavior, or any Copilot-related guidance."
argument-hint: "Paste a claim, or run with a selection. Optionally: target=<file path or section name>"
agent: "Copilot Fact-Checker"
tools: [read, search, web, todo]
---

Fact-check the GitHub Copilot claim(s) provided in this prompt and/or the current editor selection.

## Inputs
- **Selection (preferred):** `${selection}`
- **Optional target hint:** `${input:target:File path, URL, or section name (optional)}`

If the selection is empty and no claim text is provided, ask me to paste the claim(s) to review.

## Hard rules
- You MUST cite primary sources for every factual sentence about GitHub Copilot:
  - `https://docs.github.com/en/copilot/**` (highest authority)
  - `https://github.blog/changelog/label/copilot/`
  - `https://code.visualstudio.com/docs/copilot/**` (VS Code UX/customization)
- Do NOT use Microsoft Learn (`learn.microsoft.com`) to justify GitHub Copilot product claims.
- Do NOT speculate. If you can't cite it, mark it as **not documented; cannot confirm**.
- Treat any **numbers** (pricing, multipliers, limits, dates) as suspect unless sourced.
- Do NOT propose file edits unless explicitly asked. If asked, present a diff-style suggestion first.

## Process
1. Extract every distinct factual claim (numbered).
2. For each claim:
   - Quote it verbatim.
   - Classify: `product` / `pricing` / `feature-availability` / `api-contract` / `model-availability` / `config-syntax` / `ui-behavior`.
   - Use `#tool:web/fetch` to retrieve the primary source.
   - Determine verdict: ✅ Verified | ⚠️ Partially supported | ❌ Contradicted | 🔍 No primary source found.
   - Provide: URL, cited section heading/line, and a short supporting quote.
3. Provide a summary table and (if needed) suggested rewrites.

## Output format
Use exactly this format per-claim:

```
### Claim
> "<verbatim claim>"

**Classification:** <product | pricing | feature | api | model | config | ui>
**Verdict:** ✅ Verified | ⚠️ Partially supported | ❌ Contradicted | 🔍 No primary source found
**Primary source:** <URL>
**Cited section:** "<exact heading or quoted line>"
**Delta:** <what the claim says vs. what the source says>
**Suggested rewrite (if needed):**
> "<text the source actually supports>"
```

Then end with:

| # | Claim (truncated) | Verdict | Source |
|---|---|---|---|
