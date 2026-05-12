# How-To: Use the Batch API for 50% Off

If your workload doesn't need an answer in seconds — bulk evals, dataset generation, document summarization, content review — run it through the **Global Batch API**. Azure OpenAI charges **50% less than Global Standard** for batch, with a 24-hour target turnaround.

> **Goal:** never pay interactive prices for non-interactive work.

📚 Source: [Azure OpenAI batch deployments](https://learn.microsoft.com/azure/ai-services/openai/how-to/batch)

---

## 1. When batch wins (and when it doesn't)

| Use case | Use batch? |
| --- | --- |
| Score 100K customer tickets overnight | ✅ Yes |
| Generate product descriptions for a catalog | ✅ Yes |
| Bulk evaluation runs against an eval set | ✅ Yes |
| Translate / summarize a documentation corpus | ✅ Yes |
| Pre-compute embeddings | ❌ No (batch doesn't support embeddings yet) |
| Anything user-facing (chat, IDE assist) | ❌ No (24-hour SLO) |
| Fine-tuned models | ❌ No (not yet supported) |

---

## 2. Batch deployment quick start

1. In **Microsoft Foundry portal → Build → Models**, create a deployment with type **`Global-Batch`** (or **`Data Zone Batch`** for regional residency).
2. Enable **dynamic quota** so the deployment can opportunistically use spare capacity.
3. Build a `.jsonl` file — one request per line, with a `custom_id` so you can correlate responses:

```jsonl
{"custom_id":"task-0","method":"POST","url":"/v1/chat/completions","body":{"model":"gpt-5-batch","messages":[{"role":"system","content":"You are a classifier."},{"role":"user","content":"..."}]}}
{"custom_id":"task-1","method":"POST","url":"/v1/chat/completions","body":{"model":"gpt-5-batch","messages":[{"role":"system","content":"You are a classifier."},{"role":"user","content":"..."}]}}
```

4. Upload the file and **create batch job**.
5. Poll status (`validating → in_progress → finalizing → completed`) and download the output file.

> All rows in a single file must target the **same deployment**. Split files if you need different models.

---

## 3. Limits to know

| Limit | Value |
| --- | --- |
| Max requests per file | 100,000 |
| Max input file size (direct upload) | 200 MB |
| Max input file size (BYO storage) | 1 GB |
| Active input files (no expiration) | 500 |
| Target SLO | 24 hours (jobs don't expire; you can cancel anytime) |

Enqueued-token quotas per model are large (often billions/day) — see the [batch quotas table](https://learn.microsoft.com/azure/ai-services/openai/how-to/batch#batch-quota).

---

## 4. Stack with prompt caching for compounding savings

Batch + prompt caching = double win. If your batch rows all share the same system prompt / schema / few-shot examples:

- Put the shared prefix **first** in every row.
- The cache warms within the batch; subsequent rows hit it.
- You pay **50% off list × cached-token discount** on most input.

See [Win with Prompt Caching](prompt-caching.md) for prompt structure rules.

---

## 5. Queue overflow with exponential backoff

If your job is so big it hits the enqueued-token limit, Foundry now supports **batch job queuing** with exponential backoff in select regions — submit, walk away, and jobs kick off as quota frees up.

---

## 6. Don't have a batch workload? Make one.

Many "interactive" tasks are secretly batchable:

- **Nightly eval runs** of agent regressions.
- **Weekly summaries** of internal docs / Slack threads.
- **Pre-generation** of common LLM responses you cache in your app.
- **Bulk labeling** of historical data for training / fine-tuning.

Identify any LLM workload that doesn't truly need <60s latency and pipe it to batch.

---

## 7. Checklist

- [ ] Workload tolerates a 24-hour SLO.
- [ ] You created a `Global-Batch` (or `Data Zone Batch`) deployment.
- [ ] Dynamic quota is **on**.
- [ ] Each row uses the same model deployment name; one model per file.
- [ ] Shared prefix is **at the start** of each row to earn cache discounts.
- [ ] You log per-row outcomes via `custom_id`.

---

[← Back to README](../readme.md)
