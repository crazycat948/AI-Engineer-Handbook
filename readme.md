> An AI engineer is still about 80% backend engineering and 20% LLM work, much like a steamship is still mostly a ship, with only a small but transformative part being the steam engine.

---

## Choosing the Right LLM

*Based on personal testing with GPT-4o mini and GPT-5.*

### For structured-output / constrained tasks

If your application requires the LLM to follow a strict format — generating JSON, filling a schema, following a rigid prompt template — **GPT-4o mini is sufficient**. When the output space is tightly constrained and your prompt is short and focused, a smaller model handles it well and is far cheaper.

### For open-ended tasks (RAG, agents, long-context reasoning)

For tasks like RAG pipelines, agentic workflows, or anything where the question space and expected answer format are flexible, a stronger model is strongly recommended. GPT-4-class models have known limitations here:

- **Context neglect** — when the prompt grows long, the model can start ignoring instructions or earlier context.
- **Limited long-document comprehension** — understanding lengthy retrieved passages or multi-turn reasoning chains degrades noticeably.

A common workaround is to use the LLM to rewrite/compress the query first, then feed the reformulated query alongside retrieved knowledge into the model. This helps, but in practice the improvement is limited. If your budget allows, **upgrading to a more capable model (e.g. GPT-5 or equivalent) is the most reliable fix**.

### Rule of thumb

| Task type | Recommended model |
|---|---|
| Strict JSON / schema generation | GPT-4o mini (or equivalent small model) |
| RAG retrieval + synthesis | GPT-4o or higher |
| Autonomous agents, complex reasoning | GPT-5 or the strongest available model |

Invest in a stronger model where output quality is hard to validate or where prompt length is unavoidably large — the cost difference is usually worth it.

