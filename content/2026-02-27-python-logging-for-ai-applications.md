Title: Python Logging for AI Applications
Date: 2026-02-27
Category: GenAI
Tags: GenAI, Python, logging, observability, developers
Slug: python-logging-for-ai-applications

AI systems fail in ways print statements can't capture — model drift, API timeouts, silent NaNs. Here's the minimal setup that actually works.

## 1. Basic Config (One-Time Setup)

```python
import logging, sys

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)s | %(name)s | %(message)s",
    handlers=[
        logging.StreamHandler(sys.stdout),
        logging.FileHandler("app.log")
    ]
)
logger = logging.getLogger("ai-app")
```

**Why:** `StreamHandler` sends logs to your terminal for local dev. `FileHandler` persists them for production debugging. The format includes the logger name so you can filter by component when your app grows.

## 2. Log Every Model Call

```python
logger.info("LLM request", extra={
    "model": "gpt-4o",
    "input_tokens": 150,
    "request_id": req_id
})
```

**Why:** Token counts and latency are your cost signals. Without them, you can't optimize prompts or detect rate-limit issues. The `request_id` lets you trace a single user query across model calls, vector DB lookups, and post-processing.

## 3. Capture Failures Properly

```python
try:
    response = client.chat.completions.create(...)
except Exception as e:
    logger.error("LLM API failed", exc_info=True)
```

**Why:** `exc_info=True` appends the full traceback. In AI apps, failures are often transient (rate limits, context length exceeded). The traceback tells you if it's retryable or a code bug.

## 4. Structured JSON for Production

```python
import json, logging

class JSONFormatter(logging.Formatter):
    def format(self, record):
        return json.dumps({
            "ts": self.formatTime(record),
            "level": record.levelname,
            "msg": record.getMessage(),
            **getattr(record, "extra", {})
        })

logging.getLogger().handlers[0].setFormatter(JSONFormatter())
```

**Why:** Plain text is human-friendly but machine-useless. JSON logs feed directly into Datadog, CloudWatch, or ELK. You can query `level=ERROR AND model=gpt-4o` instead of grepping strings.

## 5. Never Log Secrets

```python
# BAD
logger.info(f"Calling API with key {api_key}")

# GOOD
logger.info("Calling API", extra={"model": model, "endpoint": endpoint})
```

**Why:** Logs often ship to third-party observability platforms. API keys and PII in prompts should never leave your infrastructure. Log metadata, not content.

## Log Level Cheat Sheet

| Level | When to Use |
|-------|-------------|
| `DEBUG` | Raw API responses, tensor shapes, attention weights |
| `INFO` | Request start/end, model loaded, batch complete |
| `WARNING` | Rate limit approaching, deprecated model, fallback used |
| `ERROR` | API failure, parsing error, invalid prediction |
| `CRITICAL` | Model failed to load, OOM, data corruption |

## Bottom Line

Logging in AI isn't about prettiness — it's about debugging black boxes. One JSON log line with latency, token count, and request ID is worth a thousand print statements. Set it up once, and you'll know exactly what broke and when.
