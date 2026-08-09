Title: Asyncio in LLM Applications
Date: 2026-02-24
Category: GenAI
Tags: GenAI, Python, async, LLM, performance
Slug: asyncio-in-llm-applications

LLM APIs are high-latency, I/O-bound black holes. A single GPT-4 call takes 1–10 seconds. Do that synchronously in a loop, and you're burning wall-clock time watching network requests finish one by one. Asyncio fixes this by letting Python juggle hundreds of in-flight requests on a single thread.

## The Core Idea

```python
import asyncio
from openai import AsyncOpenAI

client = AsyncOpenAI()

async def ask(prompt: str) -> str:
    r = await client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    return r.choices[0].message.content

async def main():
    prompts = ["Explain RAG", "Write a regex", "Summarize this"]
    results = await asyncio.gather(*[ask(p) for p in prompts])
    print(results)

asyncio.run(main())
```

`await` hands control back to the event loop while the API responds. `asyncio.gather` fires them all at once. Three 3-second calls finish in ~3 seconds total, not 9.

## Patterns That Actually Matter

### 1. Rate-Limited Bulk Processing

```python
semaphore = asyncio.Semaphore(20)

async def safe_ask(prompt: str):
    async with semaphore:
        return await ask(prompt)
```

Raw concurrency gets you HTTP 429'd instantly. Cap it.

### 2. Streaming Without Blocking

```python
async def stream_response(prompt: str):
    stream = await client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )
    async for chunk in stream:
        yield chunk.choices[0].delta.content or ""
```

Emit tokens as they arrive. No memory bloat from buffering full responses.

### 3. Parallel Tool Calls

```python
async def multi_tool_query(user_input: str):
    tasks = [
        classify_intent(user_input),
        extract_entities(user_input),
        check_safety(user_input),
    ]
    intent, entities, safe = await asyncio.gather(*tasks)
    return intent, entities, safe
```

Run independent LLM sub-tasks concurrently, not sequentially.

### 4. Timeout Handling

```python
async def ask_with_timeout(prompt: str, seconds: float = 10):
    return await asyncio.wait_for(ask(prompt), timeout=seconds)
```

LLMs hang. Don't let them take your server down with them.

## What Breaks Asyncio

| Mistake | Why It Dies | Fix |
|--------|-------------|-----|
| Calling sync LLM clients | Blocks the event loop | Use `AsyncOpenAI`, `AsyncAnthropic`, etc. |
| CPU-heavy post-processing | Freezes all concurrent tasks | Offload to `asyncio.to_thread()` or `ProcessPoolExecutor` |
| Unbounded `gather()` | Rate limits + memory explosion | Use `asyncio.Semaphore` |
| Forgetting `await` | Returns a coroutine object, not a result | Lint with `ruff` or `mypy` |

## Quick Snippet: Full Pipeline

```python
import asyncio
from openai import AsyncOpenAI

client = AsyncOpenAI(api_key=os.getenv("OPENAI_API_KEY"))
sem = asyncio.Semaphore(10)

async def process_batch(prompts: list[str]) -> list[str]:
    async def _one(p: str):
        async with sem:
            r = await client.chat.completions.create(
                model="gpt-4o",
                messages=[{"role": "user", "content": p}]
            )
            return r.choices[0].message.content
    return await asyncio.gather(*[_one(p) for p in prompts])
```

## Bottom Line

Asyncio won't make an LLM think faster. It will make your **application** handle more users, process bigger batches, and stream responses without choking. Use async clients, cap concurrency, and keep CPU work out of the event loop. That's it.
