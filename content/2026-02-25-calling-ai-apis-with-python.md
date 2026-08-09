Title: Calling AI APIs with Python
Date: 2026-02-25
Category: GenAI
Tags: GenAI, Python, LLM, API, developers
Slug: calling-ai-apis-with-python

Most AI APIs look simple in the docs. In production, you need retries, error handling, streaming, and batching. Here's the distilled playbook.

## The Modern Stack

| Library | Best For |
|--------|----------|
| `openai` | OpenAI, Azure OpenAI |
| `anthropic` | Claude |
| `google-genai` | Gemini |
| `httpx` | Generic REST APIs |
| `tenacity` | Retry logic |
| `pydantic` | Response validation |

## Basic Call (That You Should Never Ship)

```python
from openai import OpenAI

client = OpenAI(api_key="sk-...")  # Hardcoded key — don't

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)
```

## Production-Ready Pattern

```python
import os
from openai import OpenAI, RateLimitError, APIError
from tenacity import retry, stop_after_attempt, wait_exponential

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10),
    retry=(RateLimitError, APIError)
)
def call_llm(prompt: str, model: str = "gpt-4o") -> str:
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        temperature=0.7,
        max_tokens=500
    )
    return response.choices[0].message.content
```

- **Env var** for the key
- **Retry with backoff** for rate limits
- **Explicit parameters** — no defaults drifting

## Streaming Responses

```python
def stream_llm(prompt: str):
    stream = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )
    for chunk in stream:
        content = chunk.choices[0].delta.content
        if content:
            yield content

# Usage
for token in stream_llm("Tell me a story"):
    print(token, end="", flush=True)
```

Lower time-to-first-token. Essential for chat UIs.

## Structured Output with Pydantic

```python
from pydantic import BaseModel

class Summary(BaseModel):
    title: str
    key_points: list[str]
    sentiment: str

def structured_call(text: str) -> Summary:
    completion = client.beta.chat.completions.parse(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "Extract structured summary."},
            {"role": "user", "content": text}
        ],
        response_format=Summary,
    )
    return completion.choices[0].message.parsed
```

No regex parsing. No JSON `eval()`. Type-safe outputs.

## Multi-Provider Abstraction

```python
from abc import ABC, abstractmethod

class LLMClient(ABC):
    @abstractmethod
    def complete(self, prompt: str) -> str: ...

class OpenAIClient(LLMClient):
    def __init__(self):
        self.client = OpenAI()

    def complete(self, prompt: str) -> str:
        r = self.client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": prompt}]
        )
        return r.choices[0].message.content

class AnthropicClient(LLMClient):
    def __init__(self):
        from anthropic import Anthropic
        self.client = Anthropic()

    def complete(self, prompt: str) -> str:
        r = self.client.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=1024,
            messages=[{"role": "user", "content": prompt}]
        )
        return r.content[0].text
```

Swap providers without rewriting business logic.

## Error Handling Checklist

| Error | Cause | Handle By |
|-------|-------|-----------|
| `RateLimitError` | Too many requests | Exponential backoff |
| `APIConnectionError` | Network blip | Retry with jitter |
| `BadRequestError` | Invalid payload | Log and fix input |
| `AuthenticationError` | Bad/expired key | Alert immediately |
| `Timeout` | LLM hanging | Set `timeout` param, fallback |

## Quick Reference

```python
# One-liner with all guards
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[...],
    temperature=0.3,      # Lower = more deterministic
    max_tokens=1024,      # Cap cost
    top_p=1.0,
    frequency_penalty=0,  # Reduce repetition
    presence_penalty=0,
    timeout=30.0          # Don't hang forever
)
```

## Bottom Line

Calling an AI API is easy. Calling it **reliably** means retries, streaming, structured outputs, and clean error handling. Use a client library, wrap it in retry logic, validate outputs with Pydantic, and never hardcode a key. Ship that.
