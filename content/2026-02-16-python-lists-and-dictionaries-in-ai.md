Title: Python Lists and Dictionaries in AI
Date: 2026-02-16
Category: GenAI
Tags: GenAI, Python, LLM, developers, tutorial
Slug: python-lists-and-dictionaries-in-ai

If there's one pattern that shows up in nearly every line of generative AI code, it's this: lists and dictionaries, nested inside each other, moving data in and out of API calls. They've been referenced throughout this series as "the core data shapes of AI work," but they deserve a dedicated, practical look — because getting genuinely comfortable with them is probably the single highest-leverage Python skill for this kind of development. Here's a closer look at how they actually get used.

## Why These Two, Specifically

Generative AI APIs are built around structured, key-value data — a prompt has a role and content, a request has a model and parameters, a response has content and metadata. That shape maps almost perfectly onto Python dictionaries. And nearly everything in AI work happens in batches or sequences — a conversation is a sequence of messages, a dataset is a sequence of examples, a batch job is a sequence of inputs — which maps just as naturally onto lists. Once you see this, a huge amount of AI code stops looking mysterious and starts looking like the same few patterns repeated.

## Dictionaries: The Shape of a Single Exchange

A dictionary maps keys to values — and it's almost always how a single request or response gets represented:

```python
message = {
    "role": "user",
    "content": "What's the capital of Japan?"
}
```

Accessing a specific piece of that data is direct and readable:

```python
print(message["role"])     # "user"
print(message["content"])  # "What's the capital of Japan?"
```

API request configurations, covered in the Python fundamentals post, follow the same shape:

```python
request_config = {
    "model": "claude-sonnet-4-6",
    "max_tokens": 500,
    "temperature": 0.7
}
```

## Lists: The Shape of a Sequence

A list is an ordered collection — and in AI work, it's almost always either a conversation history or a batch of items to process:

```python
conversation_history = [
    {"role": "user", "content": "What's the capital of Japan?"},
    {"role": "assistant", "content": "The capital of Japan is Tokyo."},
    {"role": "user", "content": "What's its population?"}
]
```

This is exactly the structure referenced throughout the series' prompt-vs-completion and context engineering posts: each new turn in a conversation gets appended to this list, and the whole list gets sent along with every new request so the model has full context.

## Nesting: Where the Real Shape of AI Data Lives

Almost nothing in AI work is a flat list or a flat dictionary — it's lists of dictionaries, dictionaries containing lists, several layers deep. Getting comfortable reading and building these nested structures is the real skill here:

```python
api_response = {
    "id": "msg_123",
    "model": "claude-sonnet-4-6",
    "content": [
        {"type": "text", "text": "The capital of Japan is Tokyo."}
    ],
    "usage": {
        "input_tokens": 15,
        "output_tokens": 8
    }
}

# Pulling the actual text out requires stepping through the nesting
response_text = api_response["content"][0]["text"]
token_count = api_response["usage"]["output_tokens"]
```

This is a very typical shape for a real API response — a dictionary, containing a list, containing another dictionary. Reading it means going layer by layer: get the "content" list, get its first item (a dictionary), get that dictionary's "text" key.

## Building Conversation History: Appending to a List of Dictionaries

This exact pattern — a list of message dictionaries, growing over time — is how conversational state gets managed in code, connecting directly to the `ConversationManager` and `ChatSession` classes covered in the earlier Python classes post:

```python
conversation_history = []

def add_message(role, content):
    conversation_history.append({"role": role, "content": content})

add_message("user", "Explain gradient descent simply.")
add_message("assistant", "Gradient descent is like walking downhill to find the lowest point.")
add_message("user", "Can you give an analogy?")
```

Each call to `add_message` appends a new dictionary onto the list — the list grows, in order, exactly matching the order of the actual conversation.

## Looping Over Lists of Dictionaries: The Batch-Processing Pattern

A huge share of practical AI scripts follow this exact shape: a list of dictionaries representing items to process, looped over, with results collected into a new list:

```python
reviews = [
    {"id": 1, "text": "Great product, fast shipping!"},
    {"id": 2, "text": "Terrible experience, item arrived broken."},
    {"id": 3, "text": "It was okay, nothing special."}
]

results = []
for review in reviews:
    sentiment = classify_sentiment(review["text"])
    results.append({"id": review["id"], "sentiment": sentiment})
```

Each iteration pulls the "text" field out of one review dictionary, processes it, and builds a new dictionary combining the original "id" with the new result — a pattern that comes up constantly whenever you're running the same AI task across many pieces of data, as covered in the functions post's `process_batch` example.

## List Comprehensions With Dictionaries

Once the loop version feels comfortable, the same pattern often gets written more compactly as a list comprehension:

```python
sentiments = [classify_sentiment(r["text"]) for r in reviews]

# Building a new list of dictionaries, compactly
results = [{"id": r["id"], "sentiment": classify_sentiment(r["text"])} for r in reviews]
```

This does the same thing as the loop above, just more concisely — genuinely useful once you're comfortable reading it, though the explicit loop version is often clearer while you're still learning the pattern.

## Filtering: Pulling Out What You Actually Need

A common task is narrowing a list of dictionaries down to just the ones matching some condition — for example, pulling only negative reviews out of a larger batch:

```python
negative_reviews = [r for r in results if r["sentiment"] == "negative"]
```

Or filtering conversation history down to just the user's messages:

```python
user_messages = [m["content"] for m in conversation_history if m["role"] == "user"]
```

This kind of filtering shows up constantly when post-processing AI output — separating results by category, pulling out just the fields you need, discarding ones that failed or don't meet some criteria.

## Safely Accessing Dictionary Values

Directly indexing a dictionary with `["key"]` raises an error if that key doesn't exist — which happens more often than you'd expect with AI API responses, especially when a field is optional or a call partially fails. The `.get()` method handles this gracefully:

```python
rating = review.get("rating", None)  # returns None instead of raising an error if missing
topics = api_result.get("topics", [])  # returns an empty list as a safe default
```

This connects directly to the error-handling habits covered in the functions post — defensively accessing dictionary data with `.get()` and a sensible default is a small habit that prevents a missing field from crashing an entire batch job partway through.

## Merging and Updating Dictionaries

AI configuration often involves starting from a set of defaults and overriding just a few values per call:

```python
default_config = {"model": "claude-sonnet-4-6", "temperature": 0.7, "max_tokens": 500}

# Override just temperature for this specific call
custom_config = {**default_config, "temperature": 0.2}
```

The `{**default_config, ...}` pattern creates a new dictionary combining the defaults with any overrides — a compact way to implement exactly the kind of "stable core with adjustable specifics" behavior covered in the reusable prompt templates post, expressed directly in code.

## Converting Between JSON and Python Structures

As covered in the JSON output and fundamentals posts, JSON text and Python dictionaries/lists convert directly into each other:

```python
import json

json_text = '{"sentiment": "negative", "issues": ["cold food", "slow service"]}'
data = json.loads(json_text)   # dictionary, with issues as a list

print(data["issues"][0])  # "cold food"

# And the reverse — Python structure back into JSON text
output_json = json.dumps(data)
```

A JSON object maps to a Python dictionary; a JSON array maps to a Python list — including nested combinations of both, which is exactly the shape of most structured LLM output covered earlier in this series.

## A Realistic Combined Example

Pulling several of these patterns together — a batch job that processes a list of documents, builds structured results, and filters them:

```python
documents = [
    {"id": 1, "text": "The new policy takes effect next quarter..."},
    {"id": 2, "text": "Customer complaints have risen sharply this month..."},
    {"id": 3, "text": "The product launch exceeded expectations..."}
]

def analyze(doc):
    prompt = f"Classify sentiment (positive/negative/neutral) and give a one-line summary: {doc['text']}"
    response = call_model(prompt)
    return json.loads(response)  # e.g. {"sentiment": "negative", "summary": "..."}

results = []
for doc in documents:
    analysis = analyze(doc)
    results.append({**analysis, "id": doc["id"]})

negative_docs = [r for r in results if r.get("sentiment") == "negative"]
```

This one script uses nested dictionaries, a list of inputs, dictionary merging, safe access via `.get()`, and filtering — the same small set of patterns, combined, that underlies a large fraction of real-world AI scripting.

## The Bottom Line

Lists and dictionaries aren't just basic Python data structures — in generative AI development, they're the actual shape of almost everything: prompts, conversation history, API requests, structured responses, and batch results all take the form of dictionaries, lists, and nested combinations of both. Getting genuinely fluent in building, looping over, filtering, and safely accessing these structures — rather than treating them as something to look up each time — is what makes reading real AI code, debugging a broken script, or building your own batch-processing pipeline feel straightforward rather than confusing.
