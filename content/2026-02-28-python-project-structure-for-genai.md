Title: Python Project Structure for GenAI
Date: 2026-02-28
Category: GenAI
Tags: GenAI, Python, project-structure, developers, best-practices
Slug: python-project-structure-for-genai

A messy GenAI project becomes unmaintainable fast. Prompts scattered in notebooks, API keys hardcoded, and no separation between inference logic and business code. Here's a structure that scales.

## The Layout

```
genai-app/
├── src/
│   ├── __init__.py
│   ├── config.py          # Centralized settings
│   ├── clients/           # LLM provider wrappers
│   │   ├── __init__.py
│   │   ├── openai_client.py
│   │   └── anthropic_client.py
│   ├── prompts/           # Version-controlled prompts
│   │   ├── __init__.py
│   │   └── templates.py
│   ├── models/            # Pydantic schemas
│   │   ├── __init__.py
│   │   └── schemas.py
│   ├── services/          # Business logic
│   │   ├── __init__.py
│   │   └── rag_service.py
│   └── utils/             # Helpers
│       ├── __init__.py
│       └── logger.py
├── tests/
├── notebooks/             # Exploratory only
├── .env
├── .env.example
├── requirements.txt
├── Dockerfile
└── main.py
```

## Key Files Explained

### `config.py` — Single Source of Truth

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    openai_api_key: str
    model_name: str = "gpt-4o"
    temperature: float = 0.7

    class Config:
        env_file = ".env"

settings = Settings()
```

**Why:** One place for all env vars. Type-safe. Fails on startup if a key is missing.

### `clients/openai_client.py` — Provider Isolation

```python
from openai import AsyncOpenAI
from src.config import settings

class OpenAIClient:
    def __init__(self):
        self.client = AsyncOpenAI(api_key=settings.openai_api_key)

    async def complete(self, prompt: str) -> str:
        r = await self.client.chat.completions.create(
            model=settings.model_name,
            messages=[{"role": "user", "content": prompt}],
            temperature=settings.temperature
        )
        return r.choices[0].message.content
```

**Why:** Swap providers later without touching business logic. Settings injected, not imported ad-hoc.

### `prompts/templates.py` — Prompts as Code

```python
SUMMARIZER_PROMPT = """Summarize the following text in {style} tone.

Text: {text}

Summary:"""

def format_summarizer(text: str, style: str = "neutral") -> str:
    return SUMMARIZER_PROMPT.format(text=text, style=style)
```

**Why:** Prompts are logic, not config. Version-control them. Reuse templates. No string concatenation scattered across files.

### `models/schemas.py` — Structured I/O

```python
from pydantic import BaseModel

class SummaryOutput(BaseModel):
    summary: str
    key_points: list[str]
    sentiment: str

class RAGQuery(BaseModel):
    question: str
    top_k: int = 3
```

**Why:** Validate inputs before hitting the API. Parse outputs into types, not raw strings.

### `services/rag_service.py` — Business Logic Layer

```python
from src.clients.openai_client import OpenAIClient
from src.prompts.templates import format_rag_prompt

class RAGService:
    def __init__(self):
        self.llm = OpenAIClient()

    async def answer(self, query: str, context: list[str]) -> str:
        prompt = format_rag_prompt(query, context)
        return await self.llm.complete(prompt)
```

**Why:** Orchestrates clients, prompts, and data. Testable in isolation. No framework lock-in.

## The Rules

| Rule | Reason |
|------|--------|
| `src/` layout | Prevents import path chaos as the project grows |
| `.env` in `.gitignore` | Secrets never hit version control |
| Notebooks outside `src/` | Exploration stays separate from production code |
| Pydantic for all I/O | Catches bad data before it reaches the LLM |
| One client per provider | Easy to mock in tests, swap in production |
| Prompts in `prompts/` | Diff-friendly, reviewable, reusable |

## Quick Start Template

```bash
mkdir genai-app && cd genai-app
python -m venv venv && source venv/bin/activate

# Core deps
pip install openai pydantic pydantic-settings python-dotenv
```

**`main.py`:**

```python
import asyncio
from src.services.rag_service import RAGService

async def main():
    service = RAGService()
    answer = await service.answer(
        "What is RAG?",
        context=["RAG stands for Retrieval-Augmented Generation."]
    )
    print(answer)

asyncio.run(main())
```

## Bottom Line

GenAI projects rot when prompts, keys, and provider logic leak everywhere. Separate concerns: config in one place, prompts as code, clients behind interfaces, and business logic in services. Start with this structure on day one, and you won't be refactoring a tangled mess on day thirty.
