Title: FastAPI + LLM
Date: 2026-05-22
Category: GenAI
Tags: GenAI, FastAPI, Python, LLM, tutorial
Slug: fastapi-plus-llm

The previous post covered why FastAPI fits GenAI applications well in principle. This one is the practical follow-through: a closer, more complete look at actually wiring an LLM into a FastAPI application — from a single endpoint to a more realistic setup with conversation history, streaming, structured output, and error handling working together.

## A Complete, Realistic Example

Pulling together several patterns covered throughout this series into one working structure:

```python
from fastapi import FastAPI, HTTPException
from fastapi.responses import StreamingResponse
from pydantic import BaseModel
import anthropic
import json

app = FastAPI()
client = anthropic.AsyncAnthropic()

class ChatRequest(BaseModel):
    user_id: str
    message: str
    max_tokens: int = 500

# Simple in-memory store — a real app would use a database
conversations: dict[str, list] = {}

@app.post("/chat")
async def chat(request: ChatRequest):
    history = conversations.get(request.user_id, [])
    history.append({"role": "user", "content": request.message})

    try:
        response = await client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=request.max_tokens,
            messages=history
        )
    except anthropic.RateLimitError:
        raise HTTPException(status_code=429, detail="Rate limited, please retry shortly")
    except anthropic.APIError as e:
        raise HTTPException(status_code=502, detail=f"Model provider error: {e}")

    reply = response.content[0].text
    history.append({"role": "assistant", "content": reply})
    conversations[request.user_id] = history

    return {"reply": reply}
```

This connects directly to the agent memory architecture and Python classes posts earlier in this series — per-user conversation history, stored and reassembled on each request, is exactly the pattern covered there, just applied within a FastAPI endpoint instead of a standalone script.

## Structured Output Endpoints

For tasks needing structured, machine-parseable results — connecting directly to the JSON output and Python JSON handling posts — FastAPI's Pydantic integration lets you define both the request and the expected response shape:

```python
from typing import Literal

class SentimentRequest(BaseModel):
    text: str

class SentimentResponse(BaseModel):
    sentiment: Literal["positive", "negative", "neutral"]
    confidence: float

@app.post("/classify", response_model=SentimentResponse)
async def classify(request: SentimentRequest):
    prompt = f"Classify sentiment as JSON with keys sentiment, confidence: {request.text}"
    response = await client.messages.create(
        model="claude-sonnet-4-6", max_tokens=100,
        messages=[{"role": "user", "content": prompt}]
    )
    data = json.loads(response.content[0].text)
    return SentimentResponse(**data)  # validated against the declared schema
```

Declaring `response_model=SentimentResponse` means FastAPI validates the outgoing response too — if the model's output doesn't match the declared schema, the mismatch surfaces immediately as a clear error, rather than silently returning malformed data to the client.

## Streaming with Server-Sent Events

Building on the streaming pattern from the previous post, a more complete implementation using proper Server-Sent Events (SSE) formatting, which most frontend streaming clients expect:

```python
@app.post("/chat/stream")
async def chat_stream(request: ChatRequest):
    async def event_generator():
        async with client.messages.stream(
            model="claude-sonnet-4-6",
            max_tokens=request.max_tokens,
            messages=[{"role": "user", "content": request.message}]
        ) as stream:
            async for text in stream.text_stream:
                yield f"data: {json.dumps({'text': text})}\n\n"
        yield "data: [DONE]\n\n"

    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

## Background Tasks for Longer Agentic Work

Connecting to the autonomous agents and Vercel deployment posts earlier in this series, a genuinely long-running agentic task doesn't fit well within a single blocking request-response cycle. FastAPI's BackgroundTasks (or a proper task queue for more serious production use) lets you acknowledge a request immediately and process it asynchronously:

```python
from fastapi import BackgroundTasks
import uuid

task_results: dict[str, dict] = {}

async def run_agent_task(task_id: str, goal: str):
    # a longer-running agentic loop, as covered in the AI agent posts
    result = await run_agent(goal)
    task_results[task_id] = {"status": "complete", "result": result}

@app.post("/agent/start")
async def start_agent(goal: str, background_tasks: BackgroundTasks):
    task_id = str(uuid.uuid4())
    task_results[task_id] = {"status": "running"}
    background_tasks.add_task(run_agent_task, task_id, goal)
    return {"task_id": task_id}

@app.get("/agent/status/{task_id}")
async def agent_status(task_id: str):
    return task_results.get(task_id, {"status": "not_found"})
```

The client can then poll `/agent/status/{task_id}` for the eventual result — a practical pattern for the kind of multi-step, potentially long-running agentic workflows covered in the AI agent posts earlier in this series.

## Dependency Injection for Shared Resources

FastAPI's dependency injection system is genuinely useful for AI applications specifically — cleanly sharing a configured model client, a database connection, or an authentication check across many endpoints without repeating setup logic in each one:

```python
from fastapi import Depends

def get_client():
    return anthropic.AsyncAnthropic()

@app.post("/chat")
async def chat(request: ChatRequest, client: anthropic.AsyncAnthropic = Depends(get_client)):
    ...
```

This connects directly to the encapsulation principles covered in the Python OOP post — the endpoint doesn't need to know how the client is configured, just that it can ask for one.

## Testing an LLM-Backed API

Connecting directly to the prompt testing strategies post, testing a FastAPI + LLM application benefits from the same discipline applied to any AI system: a representative test set of requests, clear pass/fail or quality criteria, and — since real model calls are slow and costly to run in every test — mocking the LLM client during most automated tests, reserving real model calls for a smaller set of integration tests run less frequently.

## The Bottom Line

Wiring an LLM into FastAPI in practice means combining several patterns covered throughout this series: Pydantic models for both request validation and structured output enforcement, async calls to avoid blocking on model latency, streaming for responsive chat interfaces, background tasks for longer agentic work, and explicit error handling for the AI-specific failure modes covered in the exception handling post. Together, these turn a simple model API call into a genuinely production-shaped backend — reliable, observable, and structured enough to support a real application rather than just a working demo.
