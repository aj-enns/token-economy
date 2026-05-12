# How-To: Win with Prompt Caching (API / Foundry / GitHub Models)

If you call models directly via API (Azure OpenAI, Microsoft Foundry, GitHub Models, OpenAI, Anthropic), **prompt caching** is one of the biggest single levers you have. Cached input tokens are billed at a deep discount (and up to **100% off** on provisioned deployments).

> **Goal:** structure every request so the long, stable prefix earns a cache hit on the next call.

This walkthrough is for API/SDK builders. Copilot Chat handles caching automatically — but the *structure* tips below also help Copilot's underlying caches hit more often.

---

## 1. How caching works (the short version)

Azure OpenAI / Foundry models GPT-4o and newer cache input prefixes automatically:

- Cache hits require **the first 1,024 tokens to be identical** to a prior request.
- After the first 1,024 tokens match, cache hits continue **every additional 128 identical tokens**.
- A single character difference in the first 1,024 tokens = **cache miss** (`cached_tokens: 0`).
- Cached tokens are billed at a steep discount; usage reports `cached_tokens` under `prompt_tokens_details`.

📚 Source: [Azure OpenAI prompt caching](https://learn.microsoft.com/azure/ai-services/openai/how-to/prompt-caching)

---

## 2. Order matters: stable content first, dynamic content last

The model hashes the **prefix** of your prompt. Anything that changes per request must come **after** anything you want to cache.

❌ Cache-miss-prone:

```text
System: You are a helpful assistant.
User: [today's question - changes every call]
System: <giant style guide, schema, tool definitions...>
```

✅ Cache-friendly:

```text
System: You are a helpful assistant.
System: <giant style guide, schema, tool definitions...>   ← STABLE, cached
User: [today's question - changes every call]              ← DYNAMIC, last
```

Same content, same total tokens — but the cache-friendly version pays the discount on every call after the first.

---

## 3. What can be cached

- **System / developer / user / assistant messages** — the full messages array.
- **Tool definitions** — both the messages array and the `tools` array.
- **Structured output schemas** — appended to the system message as a prefix.
- **Images** — both URL and base64 (same `detail` parameter required across calls).

Put tool definitions and schemas **first**, user input **last**.

---

## 4. Boost hit rate with `prompt_cache_key`

If many requests share the same long prefix (e.g. a multi-tenant app), set the `prompt_cache_key` parameter to route them to the same cache shard:

```json
{
  "model": "gpt-5.4",
  "messages": [...],
  "prompt_cache_key": "tenant-abc"
}
```

- Combines with the prefix hash so requests with the same key + prefix share the cache.
- Default routing is per-prefix; high-volume prefixes (~15+ rpm) can spill to other machines and lose hits — `prompt_cache_key` keeps them pinned.

---

## 5. Extend retention up to 24 hours

Cache entries normally evict after 5–10 minutes of inactivity. For workloads where the same prefix repeats hourly or daily, opt in to extended retention:

```json
{
  "model": "gpt-5.4",
  "input": "...",
  "prompt_cache_retention": "24h"
}
```

- Supported on `gpt-5`, `gpt-5.1+`, `gpt-5-codex`, `gpt-4.1`, and newer (default is `24h` on the newest models).
- Same price as in-memory caching — just longer-lived.

---

## 6. Verify hits in the usage payload

Every response includes `prompt_tokens_details.cached_tokens`. Add a small log line to your SDK wrapper:

```python
usage = response.usage
total = usage.prompt_tokens
cached = usage.prompt_tokens_details.cached_tokens
print(f"cache hit ratio: {cached}/{total} = {cached/total:.0%}")
```

If your ratio is under ~50% and your prefix is long, your prompt order is probably wrong — see step 2.

---

## 7. Provisioned Throughput: cached tokens can be free

On **Provisioned (PTU)** deployments, cached input tokens can be **100% discounted**. If you have steady, high-volume traffic with a long shared prefix, PTU + caching is the cheapest configuration available on Azure OpenAI.

📚 Source: [Provisioned throughput](https://learn.microsoft.com/azure/ai-foundry/openai/concepts/provisioned-throughput)

---

## 8. Checklist

- [ ] Stable content (system msg, tool defs, schema) is **at the start** of the prompt.
- [ ] Dynamic content (user query, today's data) is **last**.
- [ ] Prefix is **≥1,024 tokens** when you want caching to engage.
- [ ] High-traffic prefixes use a `prompt_cache_key`.
- [ ] Long-cycle workloads set `prompt_cache_retention: "24h"`.
- [ ] You log `cached_tokens` and watch the hit ratio in CI / dashboards.

---

[← Back to README](../readme.md)
