Title: Python Exception Handling in AI Projects
Date: 2026-02-18
Category: GenAI
Tags: GenAI, Python, LLM, error-handling, developers
Slug: python-exception-handling-in-ai-projects

API calls fail. Rate limits get hit, networks hiccup, models return malformed output, timeouts happen mid-generation. None of this is exotic — it's the normal, expected texture of building anything that talks to an external AI service. What separates a fragile AI script from a dependable one is almost never "does it work on a clean run" — it's how deliberately it handles the moment something goes wrong. Here's a practical look at exception handling specifically through that lens.

## Why This Matters More in AI Projects Specifically

Most traditional software failure modes are at least somewhat predictable — a file exists or it doesn't, a database is reachable or it isn't. AI applications add a distinct layer of failure on top of that: network and API failures (rate limits, timeouts, service outages), plus a second category unique to this space — a call can succeed at the HTTP level and still return something unusable, like malformed JSON (covered in the previous post) or a response that doesn't match what your application expected. As covered in the developer-focused prompt engineering post, production systems need explicit fallback handling rather than assuming the happy path always holds — exception handling is the concrete mechanism that makes that possible.

## The Basics: try / except

The core tool is familiar from general Python, but it's worth grounding directly in an AI call:

```python
try:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=500,
        messages=[{"role": "user", "content": prompt}]
    )
except Exception as e:
    print(f"API call failed: {e}")
    response = None
```

This prevents a single failed call from crashing an entire script — but a bare `except Exception` catches everything, including bugs in your own code that you'd actually want to know about. That's a reasonable starting point, but it's worth outgrowing quickly.

## Catching Specific Exceptions, Not Just "Something Went Wrong"

Most AI provider SDKs expose specific exception types for specific failure modes — rate limits, authentication errors, invalid requests, service errors — and catching them specifically lets you respond appropriately to each, rather than treating every failure identically:

```python
import anthropic

try:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=500,
        messages=[{"role": "user", "content": prompt}]
    )
except anthropic.RateLimitError:
    print("Rate limited — should back off and retry")
except anthropic.AuthenticationError:
    print("Invalid API key — not something a retry will fix")
except anthropic.APIConnectionError:
    print("Network issue — worth retrying")
except anthropic.APIStatusError as e:
    print(f"API returned an error status: {e.status_code}")
```

This distinction matters practically: a rate limit or connection error is usually worth retrying; an authentication error never is, no matter how many times you try again. Treating them the same way — as one generic "it failed" case — means either retrying something that will never succeed, or giving up on something that would have worked a moment later.

## Retry Logic: Handling Transient Failures

Since a lot of API failures are transient — a momentary rate limit, a brief network blip — retrying with a short delay is often the right response, rather than failing immediately:

```python
import time

def call_with_retry(prompt, max_retries=3, base_delay=1):
    for attempt in range(max_retries):
        try:
            return client.messages.create(
                model="claude-sonnet-4-6",
                max_tokens=500,
                messages=[{"role": "user", "content": prompt}]
            )
        except anthropic.RateLimitError:
            if attempt == max_retries - 1:
                raise
            wait = base_delay * (2 ** attempt)  # exponential backoff
            print(f"Rate limited, retrying in {wait}s...")
            time.sleep(wait)
        except anthropic.AuthenticationError:
            raise  # never worth retrying, fail immediately
```

The exponential backoff pattern here (`base_delay * (2 ** attempt)`) — waiting progressively longer between each retry — is a standard, well-established way to handle rate limits specifically, since retrying too aggressively right after being rate limited often just triggers the limit again.

## Custom Exceptions: Making Your Own Failure Modes Explicit

Beyond what the SDK raises, it's often useful to define your own exception types for AI-specific failure modes your application cares about — like a response that parses fine but fails validation, connecting directly to the schema validation covered in the previous post:

```python
class InvalidModelOutputError(Exception):
    """Raised when a model response doesn't match the expected structure."""
    pass

def get_sentiment(review_text):
    response = call_model(f"Classify sentiment: {review_text}")
    try:
        data = json.loads(response)
    except json.JSONDecodeError:
        raise InvalidModelOutputError(f"Model returned non-JSON output: {response[:100]}")

    if data.get("sentiment") not in {"positive", "negative", "neutral"}:
        raise InvalidModelOutputError(f"Unexpected sentiment value: {data.get('sentiment')}")

    return data
```

Custom exceptions like this make the calling code's error handling more precise and readable — catching `InvalidModelOutputError` specifically communicates exactly what kind of problem occurred, rather than a generic `Exception` that could mean anything from a network failure to a typo in your own code.

## finally: Cleanup That Always Runs

`finally` runs regardless of whether an exception occurred — useful for cleanup that needs to happen either way, like closing a resource or logging that an attempt was made:

```python
def process_document(doc):
    start_time = time.time()
    try:
        result = call_model(f"Summarize: {doc}")
        return result
    except Exception as e:
        log_error(doc, e)
        return None
    finally:
        elapsed = time.time() - start_time
        log_timing(doc, elapsed)  # logged whether it succeeded or failed
```

This pattern is especially useful for the kind of monitoring covered in the prompt injection prevention post — logging that a call happened, and how long it took, regardless of outcome, gives you visibility into both successes and failures rather than only ever seeing the successful runs.

## Fallback Values: Degrading Gracefully Instead of Crashing

For a lot of AI application logic, the right response to a failure isn't to halt everything — it's to fall back to a sensible default and keep going, especially in a batch job where one bad item shouldn't take down the rest:

```python
def classify_sentiment_safe(text, default="unknown"):
    try:
        response = call_model(f"Classify sentiment: {text}")
        return json.loads(response)["sentiment"]
    except Exception:
        return default

reviews = ["Great product!", "", "Terrible service", None]
sentiments = [classify_sentiment_safe(r) for r in reviews]
# a bad or empty input returns "unknown" rather than crashing the whole batch
```

This directly supports the batch-processing patterns covered in the lists and dictionaries post — a single malformed input or failed call becomes one "unknown" entry in the results, not a reason the entire job dies partway through.

## Handling Partial Failures in Batch Jobs

When processing many items, it's worth explicitly tracking which succeeded and which failed, rather than silently dropping failures or letting one bad item stop everything:

```python
def process_batch(items, process_fn):
    successes = []
    failures = []
    for item in items:
        try:
            result = process_fn(item)
            successes.append({"item": item, "result": result})
        except Exception as e:
            failures.append({"item": item, "error": str(e)})
    return successes, failures

successes, failures = process_batch(reviews, classify_sentiment)
print(f"{len(successes)} succeeded, {len(failures)} failed")
```

This connects back to the higher-order function pattern from the earlier functions post, but adds explicit failure tracking — giving you a clear, inspectable record of exactly what didn't work, which is essential both for debugging and for the kind of production monitoring covered earlier in this series.

## Timeouts: Failing Fast Instead of Hanging Indefinitely

AI API calls can occasionally hang far longer than expected. Most SDKs support an explicit timeout, and it's worth setting one deliberately rather than relying on defaults:

```python
try:
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=500,
        messages=[{"role": "user", "content": prompt}],
        timeout=30.0  # seconds
    )
except anthropic.APITimeoutError:
    print("Request timed out — treating as a transient failure")
```

A hung request with no timeout can silently stall an entire batch job or user-facing feature; an explicit timeout turns that indefinite wait into a specific, catchable, recoverable failure.

## What Not to Do: Common Anti-Patterns

**Silently swallowing exceptions with no logging.**

```python
# Avoid this
try:
    response = call_model(prompt)
except Exception:
    pass  # failure vanishes with no trace
```

This makes failures invisible — no log, no metric, nothing to debug later when results look wrong and you have no idea why.

**Catching exceptions too broadly, too early.**

Wrapping an entire large block of logic in one `try/except Exception` obscures exactly which line failed and why — narrower, more targeted try blocks around the specific operation that can actually fail (the API call, the JSON parse) make debugging dramatically easier.

**Retrying failures that will never succeed.**

Retrying an `AuthenticationError` or a malformed-request error just wastes time and API calls — as covered above, it's worth distinguishing transient failures (worth retrying) from permanent ones (worth failing immediately and surfacing clearly).

## The Bottom Line

Exception handling in AI projects isn't a defensive afterthought bolted on at the end — it's a core part of making an AI application actually dependable, given how many distinct ways a call to an external model can fail: network issues, rate limits, authentication problems, malformed or invalid output. Catching specific exception types rather than everything generically, retrying transient failures with backoff, defining custom exceptions for AI-specific failure modes, tracking partial failures in batch jobs, and setting explicit timeouts are what turn a script that works great in a demo into one that keeps working when real usage — and real failures — actually show up.
