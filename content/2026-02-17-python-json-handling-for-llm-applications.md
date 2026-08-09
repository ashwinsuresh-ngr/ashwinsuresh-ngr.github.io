Title: Python JSON Handling for LLM Applications
Date: 2026-02-17
Category: GenAI
Tags: GenAI, Python, LLM, JSON, developers
Slug: python-json-handling-for-llm-applications

Every structured piece of data that moves between your code and an LLM API eventually passes through JSON — it's the universal handshake format for AI APIs, tool calls, and structured model output. Python's relationship with JSON is close to seamless, but getting genuinely comfortable with the specific ways it shows up in LLM applications — parsing responses, handling malformed output, converting between formats — is what separates scripts that work on the happy path from ones that hold up in practice. Here's a practical, focused look.

## Why JSON Specifically

As covered in the earlier JSON output post, LLM APIs need a predictable, parseable way to exchange structured data — requests, responses, tool calls, extracted fields. JSON's simple, universal structure (objects, arrays, strings, numbers, booleans, null) maps almost one-to-one onto Python's own dictionaries, lists, strings, numbers, booleans, and None — which is exactly why Python's built-in `json` module feels so natural to use in this context.

## The Two Core Operations: loads and dumps

Nearly all JSON handling in LLM applications comes down to two functions from Python's built-in `json` module:

```python
import json

# JSON text -> Python object
json_text = '{"sentiment": "negative", "confidence": 0.87}'
data = json.loads(json_text)
print(data["sentiment"])  # "negative"

# Python object -> JSON text
python_dict = {"sentiment": "positive", "confidence": 0.95}
json_text = json.dumps(python_dict)
```

`loads` ("load string") parses JSON text into a Python object — usually a dictionary or list. `dumps` ("dump string") does the reverse, converting a Python object back into JSON text. Nearly everything else in this post is a variation or complication of these two operations.

## Parsing a Real LLM Response

When a model returns structured output, it typically arrives as a string containing JSON — even though it represents structured data, it needs to be explicitly parsed before you can work with it like a dictionary:

```python
response_text = model_response.content[0].text
# response_text is a STRING that looks like: '{"summary": "...", "topics": ["AI", "Python"]}'

data = json.loads(response_text)
print(data["topics"])  # now an actual Python list: ["AI", "Python"]
```

This is a distinction worth being precise about: before `json.loads()`, you have a string that merely contains JSON-formatted text — you can't index into it with `["topics"]` the way you can a real dictionary. Only after parsing does it become a genuine Python object you can work with directly.

## The Problem: Models Don't Always Return Clean JSON

As covered in the JSON output post, even with good prompting, models can wrap their JSON in conversational text, add trailing commentary, or produce near-miss syntax errors:

```python
raw_response = 'Sure! Here\'s the JSON you requested:\n\n{"sentiment": "negative", "rating": 2}\n\nLet me know if you need anything else!'
```

Passing this directly to `json.loads()` fails, because it isn't valid JSON on its own — it's JSON embedded inside surrounding prose.

## Defensive Parsing: Extracting JSON From Messy Output

A common, practical pattern is stripping away the non-JSON wrapper before parsing:

```python
import re
import json

def extract_json(text):
    # Look for the first { to the last } as a simple heuristic
    match = re.search(r'\{.*\}', text, re.DOTALL)
    if match:
        return json.loads(match.group())
    raise ValueError("No JSON object found in response")

data = extract_json(raw_response)
```

This isn't bulletproof — deeply nested or unusual responses can still trip it up — but it handles a large share of the "the model added a friendly wrapper" cases that show up constantly in practice, connecting back to the "hope is not a reliability strategy" point from the building reliable prompts post: this kind of defensive parsing is a mechanical safeguard, not a request the model might ignore.

## Handling Parse Failures Gracefully

As covered in the functions post's error-handling patterns, a JSON parse failure shouldn't crash an entire batch job. Wrapping the parse step in a try/except is essential, not optional:

```python
def safe_parse(response_text, default=None):
    try:
        return json.loads(response_text)
    except json.JSONDecodeError as e:
        print(f"Failed to parse JSON: {e}")
        return default

data = safe_parse(response_text, default={})
```

Catching `json.JSONDecodeError` specifically (rather than a bare `except Exception`) is worth doing here — it keeps the error handling targeted to the actual failure mode you're expecting, rather than silently swallowing unrelated bugs elsewhere in the function.

## Validating Structure, Not Just Syntax

Valid JSON syntax doesn't guarantee the shape you actually need — a response might parse successfully but be missing an expected key, or have a value of the wrong type. This connects directly to the "validate model output, don't just trust it" principle from the developer-focused prompt engineering post:

```python
def validate_sentiment_response(data):
    required_keys = {"sentiment", "confidence"}
    if not required_keys.issubset(data.keys()):
        raise ValueError(f"Missing required keys: {required_keys - data.keys()}")
    if data["sentiment"] not in {"positive", "negative", "neutral"}:
        raise ValueError(f"Unexpected sentiment value: {data['sentiment']}")
    return data

data = json.loads(response_text)
validated = validate_sentiment_response(data)
```

A syntactically valid JSON object can still contain a hallucinated field, an out-of-range value, or a subtly wrong structure — parsing successfully is necessary but not sufficient for trusting the result.

## Schema Validation With Pydantic

For anything beyond a quick script, hand-writing validation checks for every field gets tedious and error-prone fast. The `pydantic` library is widely used in LLM applications specifically for this — defining an expected schema once, then validating (and getting helpful errors) automatically:

```python
from pydantic import BaseModel
from typing import Literal

class SentimentResult(BaseModel):
    sentiment: Literal["positive", "negative", "neutral"]
    confidence: float
    key_issues: list[str] = []

raw = json.loads(response_text)
result = SentimentResult(**raw)  # raises a clear error if anything doesn't match

print(result.sentiment)  # accessed as an attribute, not a dictionary key
```

If the model returns a confidence as a string instead of a number, or an unexpected sentiment value, Pydantic raises a specific, readable validation error immediately — rather than that bad data silently propagating further into your application. Many modern LLM SDKs integrate with Pydantic-style schemas directly for exactly this reason, tying back to the schema-constrained generation features covered in the JSON output post.

## Nested Structures: Parsing Multi-Level JSON

LLM responses are often nested several levels deep — a common shape for anything involving lists of extracted items:

```python
response_text = '''
{
  "document_summary": "Quarterly sales report showing growth across all regions.",
  "regions": [
    {"name": "North America", "growth": 0.12, "highlights": ["record Q4", "new store openings"]},
    {"name": "Europe", "growth": 0.08, "highlights": ["stable demand"]}
  ]
}
'''

data = json.loads(response_text)

for region in data["regions"]:
    print(f"{region['name']}: {region['growth']:.0%}")
    for highlight in region["highlights"]:
        print(f"  - {highlight}")
```

This is the same nested list-of-dictionaries pattern covered in the previous post — `json.loads()` reconstructs the full nested shape automatically, so once parsed, it behaves exactly like any other Python nested structure.

## Serializing Python Objects Back Into JSON

The reverse direction matters too — building a Python structure and sending it as JSON, whether as part of a request payload or for storage:

```python
request_payload = {
    "model": "claude-sonnet-5",
    "messages": conversation_history,
    "max_tokens": 500
}

json_payload = json.dumps(request_payload)
```

Most AI provider SDKs handle this serialization step internally when you pass a Python dictionary directly, but understanding it's happening underneath is useful for debugging — and essential when you're manually constructing requests with `requests` rather than a dedicated SDK.

## Pretty-Printing for Readability

When debugging or logging structured output, the default `json.dumps()` output is compact and hard to read. The `indent` parameter fixes that:

```python
print(json.dumps(data, indent=2))
```

```json
{
  "sentiment": "negative",
  "confidence": 0.87,
  "key_issues": [
    "cold food",
    "slow service"
  ]
}
```

This is a small habit, but it's genuinely useful constantly while developing and debugging — a wall of compact JSON is much harder to visually scan than the same data indented clearly.

## Handling Non-Standard Values: NaN, Dates, and Custom Objects

Standard JSON doesn't support Python-specific types like `datetime` objects — attempting to serialize one directly raises an error:

```python
from datetime import datetime

data = {"timestamp": datetime.now()}
json.dumps(data)  # raises TypeError: Object of type datetime is not JSON serializable
```

A common fix is a custom default handler, converting unsupported types into something JSON-compatible:

```python
def json_default(obj):
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Type {type(obj)} not serializable")

json.dumps(data, default=json_default)
```

This comes up more often than expected once real application data — timestamps, IDs, custom classes — starts flowing alongside LLM-generated content in the same structures.

## A Realistic Combined Example

Pulling several of these patterns together — parsing a batch of LLM responses defensively, validating them, and collecting only the valid results:

```python
import json
from pydantic import BaseModel, ValidationError
from typing import Literal

class ReviewAnalysis(BaseModel):
    sentiment: Literal["positive", "negative", "neutral"]
    summary: str

def analyze_review(review_text: str) -> dict:
    prompt = f"Analyze sentiment and summarize: {review_text}. Return JSON only."
    raw_response = call_model(prompt)

    try:
        parsed = extract_json(raw_response)
        validated = ReviewAnalysis(**parsed)
        return {"success": True, "data": validated.model_dump()}
    except (json.JSONDecodeError, ValidationError) as e:
        return {"success": False, "error": str(e), "raw": raw_response}

reviews = ["Great service, fast delivery!", "Package arrived damaged, very upset."]
results = [analyze_review(r) for r in reviews]

successful = [r["data"] for r in results if r["success"]]
failed = [r for r in results if not r["success"]]

print(f"{len(successful)} succeeded, {len(failed)} failed")
```

This combines defensive extraction, schema validation, structured error capture, and list-based batch processing — the same handful of JSON-handling patterns, composed together, that cover the large majority of real LLM application code.

## The Bottom Line

JSON handling in LLM applications isn't really about learning new syntax — Python's `json` module is small and simple — it's about building the defensive habits that account for how real model output actually behaves: wrapped in conversational text, occasionally malformed, syntactically valid but structurally wrong. Comfort with `loads` and `dumps`, defensive extraction and error handling around parsing, schema validation with tools like Pydantic, and small habits like pretty-printing for debugging are what turn "the model sort of returns JSON" into a dependable, structured data pipeline you can actually build an application on top of.
