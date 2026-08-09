Title: Python Functions for AI Development
Date: 2026-02-13
Category: GenAI
Tags: GenAI, Python, LLM, developers, tutorial
Slug: python-functions-for-ai-development

Once a generative AI script grows past a handful of lines, functions stop being optional structure and start being the thing that keeps the whole project sane. They're how you turn "a prompt I typed once" into "a reusable piece of AI logic anyone on the team can call." This post looks at Python functions specifically through the lens of building generative AI applications — what to actually know, and where functions show up constantly in real AI code.

## Why Functions Matter More in AI Code, Specifically

Generative AI scripts tend to repeat the same basic shape over and over: build a prompt, call a model, handle the response, maybe retry on failure. Without functions, that pattern gets copy-pasted every time you need it — and copy-pasted code is exactly where inconsistency and bugs creep in, especially once you're adjusting prompts (as covered throughout the prompt engineering posts in this series) and need every call site to reflect the update. Functions turn that repeated pattern into a single, reliable, callable unit — write it once, fix it once, use it everywhere.

## The Basic Anatomy, Applied to AI Calls

A function packages up a block of logic behind a name, optionally takes inputs (parameters), and optionally returns a result:

```python
def summarize(text):
    prompt = f"Summarize the following text in 3 bullet points:\n\n{text}"
    response = call_model(prompt)
    return response
```

Once defined, this function can be called anywhere in the script, as many times as needed, with different input each time:

```python
summary_1 = summarize(article_one)
summary_2 = summarize(article_two)
```

This is the direct, practical realization of the prompt templates concept covered earlier in this series — the fixed instruction ("summarize the following text in 3 bullet points") lives in one place, and only the variable part (`text`) changes per call.

## Parameters: Making Functions Flexible Without Duplicating Logic

Parameters are what let one function handle a whole family of related requests, rather than needing a separate function for every slight variation:

```python
def summarize(text, max_bullets=3, tone="neutral"):
    prompt = f"Summarize the following text in {max_bullets} {tone} bullet points:\n\n{text}"
    return call_model(prompt)
```

Default arguments (`max_bullets=3`, `tone="neutral"`) are especially useful in AI functions specifically, because they let you set a sensible default behavior while still allowing any individual call to override it when needed:

```python
summarize(article)                                  # uses the defaults
summarize(article, max_bullets=5, tone="casual")     # overrides both
```

This mirrors exactly the kind of flexible-but-consistent behavior covered in the reusable prompt templates post — a stable core with room to adjust specific details per use.

## Keyword Arguments: Clarity When Calls Get Complex

As AI functions accumulate more parameters — model name, temperature, max tokens, system prompt — calling them with plain positional arguments gets error-prone and hard to read. Keyword arguments solve this directly:

```python
def generate_response(prompt, model="claude-sonnet-4-6", temperature=0.7, max_tokens=500):
    # API call logic would go here
    pass

# Clear, even with many parameters
generate_response(
    prompt="Explain quantum computing simply.",
    temperature=0.2,
    max_tokens=300
)
```

This connects directly to the temperature and top-p post — functions like this are typically where those sampling parameters actually get set and passed through to an API call, and keyword arguments make it immediately obvious which value controls what, rather than relying on remembering argument order.

## Return Values: Getting Structured Data Back Out

A function's return statement is how the result of an AI call gets handed back to whatever called it — and in AI code, that returned value is often more than just raw text:

```python
def classify_sentiment(review_text):
    prompt = f"Classify the sentiment of this review as positive, negative, or neutral: {review_text}"
    response = call_model(prompt)
    return response.strip().lower()
```

For tasks needing structured output (as covered in the JSON output post), it's common to return a parsed dictionary rather than raw text:

```python
import json

def extract_review_data(review_text):
    prompt = f"Extract sentiment and rating as JSON from: {review_text}"
    response = call_model(prompt)
    return json.loads(response)  # returns a dictionary, not a raw string
```

Returning structured data rather than a raw string makes the function's output immediately usable by whatever code calls it next — no repeated parsing logic scattered throughout the script.

## Error Handling Inside Functions: Containing Failure

As covered in the Python fundamentals post, API calls fail — and wrapping that failure handling inside the function itself, rather than requiring every caller to remember to do it, is one of the most valuable habits in AI development:

```python
def call_model_safely(prompt, retries=2):
    for attempt in range(retries + 1):
        try:
            return call_model(prompt)
        except Exception as e:
            if attempt == retries:
                return f"Error after {retries} retries: {e}"
```

This connects directly to the "design for fallback behavior" principle covered in the developer-focused prompt engineering post — building that resilience once, inside a shared function, rather than hoping every call site handles failure correctly on its own.

## Functions That Wrap Prompt Templates

This is one of the most common patterns in real AI codebases: a function whose entire job is to fill in a prompt template and send it off, tying together the prompt templates and functions concepts from earlier in this series:

```python
def draft_email(recipient_type, topic, tone="professional", word_limit=150):
    prompt = f"""
    Write a {tone} email to a {recipient_type} about {topic}.
    Keep it under {word_limit} words.
    """
    return call_model(prompt)
```

Every parameter here maps directly to a placeholder in the underlying template — the function is the template, expressed in code rather than as a standalone string with {curly braces} to fill in manually.

## Higher-Order Functions: Processing Many Inputs Consistently

A function that takes another function as an argument — a higher-order function — is a genuinely useful pattern for applying the same AI logic across a batch of inputs, connecting to the loop-based batch processing covered in the fundamentals post:

```python
def process_batch(items, process_fn):
    results = []
    for item in items:
        results.append(process_fn(item))
    return results

reviews = ["Great service!", "Very disappointing.", "It was okay."]
sentiments = process_batch(reviews, classify_sentiment)
```

This pattern separates "how do I loop over a batch" from "what do I actually do to each item" — letting the same `process_batch` function get reused with completely different AI tasks, just by swapping which function gets passed in.

## Type Hints: Optional, But Genuinely Useful in AI Code

Python doesn't require you to specify types, but type hints make AI functions significantly easier to understand and debug — especially since so many AI function signatures involve dictionaries, lists, and optional values that aren't obvious from the name alone:

```python
def summarize(text: str, max_bullets: int = 3) -> str:
    prompt = f"Summarize the following text in {max_bullets} bullet points:\n\n{text}"
    return call_model(prompt)

def get_conversation_history() -> list[dict]:
    return conversation_history
```

Type hints don't change how the code runs, but they make it immediately clear — to you later, or to a teammate — what a function expects and what it hands back, which matters a lot once a codebase has dozens of these small AI-calling functions.

## Docstrings: Documenting What a Prompt-Wrapping Function Actually Does

Since AI functions often hide meaningful prompt engineering decisions inside them — a specific phrasing, a particular format requirement — a short docstring explaining why a function is built the way it is pays off, echoing the "document the reasoning" principle from the prompt versioning post:

```python
def classify_sentiment(review_text: str) -> str:
    """
    Classifies review sentiment as 'positive', 'negative', or 'neutral'.
    Uses a zero-shot prompt; tested against a 50-review sample set (see test_sentiment.py).
    """
    prompt = f"Classify the sentiment of this review as positive, negative, or neutral: {review_text}"
    return call_model(prompt).strip().lower()
```

This is a small habit, but it directly supports the kind of prompt testing and versioning discipline covered earlier in this series — the reasoning behind a specific prompt doesn't just live in someone's memory.

## A Realistic Example Putting It Together

```python
import json

def extract_and_summarize(document: str, max_words: int = 100) -> dict:
    """
    Summarizes a document and extracts key topics as structured data.
    Returns a dictionary with 'summary' and 'topics' keys.
    """
    prompt = f"""
    Summarize the following document in under {max_words} words,
    and list up to 5 key topics.
    Return your answer as JSON with keys: summary, topics.

    Document:
    \"\"\"
    {document}
    \"\"\"
    """
    try:
        response = call_model(prompt)
        return json.loads(response)
    except Exception as e:
        return {"summary": None, "topics": [], "error": str(e)}
```

This single function combines a parameterized template, structured prompting (the delimited document section), a JSON output request, error handling, and a docstring — several techniques from across this series, expressed together in one reusable unit.

## The Bottom Line

Functions are what turn one-off AI prompts into dependable, reusable tools — packaging a prompt template, its parameters, and its error handling behind a single callable name, rather than copy-pasting the same pattern throughout a script. Comfort with parameters and default arguments, meaningful return values (especially structured ones), contained error handling, and small habits like type hints and docstrings are what let a generative AI codebase grow from a quick experiment into something genuinely maintainable — one well-designed function at a time, reused everywhere it's needed rather than rebuilt from scratch.
