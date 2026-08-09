Title: Building an LLM Client in Python
Date: 2026-02-26
Category: GenAI
Tags: GenAI, Python, LLM, OOP, developers
Slug: building-an-llm-client-in-python

Don't let vendor SDKs leak into your business logic. Build one clean client. Swap providers later without touching your app.

## The Interface

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class LLMResponse:
    text: str
    tokens_used: int
    model: str

class LLMClient(ABC):
    @abstractmethod
    def complete(self, prompt: str, **kwargs) -> LLMResponse: ...

    @abstractmethod
    def stream(self, prompt: str): ...
```

One contract. Any backend.

## OpenAI Implementation

```python
from openai import OpenAI, RateLimitError
from tenacity import retry, stop_after_attempt, wait_exponential
import os

class OpenAILLM(LLMClient):
    def __init__(self, model: str = "gpt-4o"):
        self.client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
        self.model = model

    @retry(stop=stop_after_attempt(3), wait=wait_exponential(1, 2, 10))
    def complete(self, prompt: str, temperature: float = 0.7) -> LLMResponse:
        r = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": prompt}],
            temperature=temperature
        )
        return LLMResponse(
            text=r.choices[0].message.content,
            tokens_used=r.usage.total_tokens,
            model=self.model
        )

    def stream(self, prompt: str):
        stream = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": "user", "content": prompt}],
            stream=True
        )
        for chunk in stream:
            if chunk.choices[0].delta.content:
                yield chunk.choices[0].delta.content
```

## Anthropic Implementation

```python
from anthropic import Anthropic

class AnthropicLLM(LLMClient):
    def __init__(self, model: str = "claude-3-5-sonnet-20241022"):
        self.client = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))
        self.model = model

    def complete(self, prompt: str, **kwargs) -> LLMResponse:
        r = self.client.messages.create(
            model=self.model,
            max_tokens=4096,
            messages=[{"role": "user", "content": prompt}]
        )
        return LLMResponse(
            text=r.content[0].text,
            tokens_used=r.usage.input_tokens + r.usage.output_tokens,
            model=self.model
        )
```

## Factory + Usage

```python
class LLMFactory:
    _registry = {
        "openai": OpenAILLM,
        "anthropic": AnthropicLLM,
    }

    @classmethod
    def create(cls, provider: str, **kwargs) -> LLMClient:
        if provider not in cls._registry:
            raise ValueError(f"Unknown provider: {provider}")
        return cls._registry[provider](**kwargs)

# Usage
llm = LLMFactory.create("openai", model="gpt-4o")
response = llm.complete("Explain async programming")
print(response.text)
```

Change one string. Switch providers instantly.

## Adding Structured Output

```python
from pydantic import BaseModel

class ExtractedData(BaseModel):
    name: str
    age: int

class OpenAILLM(LLMClient):
    def structured(self, prompt: str, schema: type[BaseModel]) -> BaseModel:
        r = self.client.beta.chat.completions.parse(
            model=self.model,
            messages=[{"role": "user", "content": prompt}],
            response_format=schema,
        )
        return r.choices[0].message.parsed

# Usage
data = llm.structured("John is 30", ExtractedData)
```

## The Checklist

| Feature | Why |
|--------|-----|
| Abstract base class | Vendor lock-in protection |
| Retry decorator | API flakiness handled silently |
| `LLMResponse` dataclass | Consistent return shape |
| Factory pattern | Runtime provider switching |
| Streaming generator | Low latency for chat UIs |
| Structured output | No regex parsing ever |

## Bottom Line

Build the client once. Use it everywhere. When OpenAI hikes prices or Anthropic drops a better model, you change one line — not fifty. That's the point of abstraction.
