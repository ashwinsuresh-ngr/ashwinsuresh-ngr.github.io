Title: Python Variables for AI Applications
Date: 2026-02-12
Category: GenAI
Tags: GenAI, Python, LLM, developers, tutorial
Slug: python-variables-for-ai-applications

Every generative AI script — no matter how simple or sophisticated — is built on the most basic unit in Python: the variable. It's easy to skim past variables as "too basic to matter" when you're eager to get to prompts and API calls, but the way you use variables directly shapes how clean, reliable, and reusable your AI code actually is. Here's a practical look at variables specifically through the lens of building generative AI applications.

## What a Variable Actually Is, Quickly

A variable is simply a named reference to a value stored in memory — a label you can use to store, retrieve, and pass around data. In Python, you don't need to declare a variable's type explicitly; you just assign a value, and Python figures out the type on its own.

```python
model_name = "claude-sonnet-4-6"
max_tokens = 1000
temperature = 0.7
```

This might look trivial, but in AI development, variables are doing something specific and important: they're holding the exact pieces of information — prompts, parameters, responses — that flow through every API call you make.

## Why Variables Matter More Than They Seem To in AI Work

Unlike a lot of traditional software, generative AI scripts are often built around a small number of key values that get reused, modified, and passed into API calls repeatedly: the prompt text, the model name, sampling parameters, the conversation history. Getting comfortable with how these values are stored, updated, and referenced isn't a preliminary step before "the real work" — it more or less is the real work, at least at the level of actually wiring a script together.

## Naming Variables Clearly Matters More Than You'd Think

In AI scripts specifically, unclear variable names create real confusion fast, because so many values look superficially similar — strings, numbers, dictionaries — but mean very different things.

```python
# Unclear
x = "Summarize this article in 3 bullet points."
y = 0.3
z = call_model(x, y)

# Clear
summary_prompt = "Summarize this article in 3 bullet points."
temperature = 0.3
summary_response = call_model(summary_prompt, temperature)
```

The second version is barely longer, but it's dramatically easier to read, debug, and hand off to someone else — which matters a lot once a script grows beyond a quick, throwaway experiment.

## The Core Variable Types You'll Use Constantly

**Strings** — for prompts, model responses, and any text data:

```python
system_prompt = "You are a helpful coding assistant."
user_question = "How do I reverse a list in Python?"
```

**Numbers (int and float)** — for sampling parameters, token limits, and any numeric configuration:

```python
max_tokens = 500
temperature = 0.7  # float, controls randomness (covered in the temperature and top-p post)
retry_count = 3
```

**Booleans** — for flags controlling behavior, like whether to stream a response or whether a task succeeded:

```python
stream_response = True
task_completed = False
```

**None** — Python's way of representing "no value yet," commonly used as a starting point for a variable that will be filled in later, like an API response before the call has actually happened:

```python
response = None
try:
    response = call_ai_api(user_question)
except Exception as e:
    print(f"Call failed: {e}")
```

## Variables Holding Structured Data: Where AI Work Gets Real

As covered in the previous post on Python fundamentals, dictionaries and lists are the real workhorses of generative AI scripting — and they're just variables holding more complex, structured values rather than a single string or number.

```python
request_config = {
    "model": "claude-sonnet-4-6",
    "max_tokens": 1000,
    "temperature": 0.5
}

conversation_history = [
    {"role": "user", "content": "What's the capital of France?"},
    {"role": "assistant", "content": "The capital of France is Paris."}
]
```

Here, `request_config` and `conversation_history` are still just variables — but they're holding rich, nested data rather than a single value, which is the shape almost all real AI application data actually takes.

## Mutability: A Subtlety Worth Understanding Early

One Python quirk that trips people up in AI scripts specifically: lists and dictionaries are mutable, meaning they can be changed in place, while things like strings and numbers are immutable. This matters a lot when managing something like conversation history, where you're typically appending to an existing list rather than creating a new one each time:

```python
conversation_history = []

def add_message(role, content):
    conversation_history.append({"role": role, "content": content})

add_message("user", "Hello!")
add_message("assistant", "Hi there! How can I help?")
```

Because `conversation_history` is mutable, `.append()` modifies it directly, in place — no need to reassign it. Understanding this distinction avoids a common source of confusing bugs, especially once functions start passing these structures around.

## Variable Scope: Why It Matters in Scripts and Functions

Where a variable is defined determines where it can be accessed — a concept called scope. This becomes practically relevant fast once you start organizing AI logic into functions:

```python
api_key = os.environ.get("ANTHROPIC_API_KEY")  # module-level, broadly accessible

def summarize(text):
    prompt = f"Summarize this: {text}"  # local to this function only
    return call_model(prompt)
```

`api_key` is defined at the top level and accessible throughout the script; `prompt` only exists inside the `summarize` function and disappears once the function finishes running. Understanding this prevents a common early mistake: trying to access a variable outside the function where it was actually created.

## Constants: Signaling "This Shouldn't Change"

Python doesn't have true constants in the way some languages do, but by convention, values meant to stay fixed throughout a script are written in all caps — a signal to yourself and anyone else reading the code that this value is meant to be configuration, not something that changes as the script runs:

```python
DEFAULT_MODEL = "claude-sonnet-4-6"
MAX_RETRIES = 3
SYSTEM_PROMPT = "You are a concise, helpful assistant."
```

This becomes especially useful in AI scripts, where a handful of configuration values — model name, default parameters, a reused system prompt — often need to stay consistent across many function calls throughout a project.

## F-Strings: Variables Feeding Directly Into Prompts

This connects directly to the prompt templates concept covered earlier in this series: f-strings let you insert variable values directly into a string, which is exactly how dynamic prompts get built in code.

```python
user_name = "Alex"
topic = "neural networks"
difficulty = "beginner"

prompt = f"Explain {topic} to {user_name} at a {difficulty} level, using a simple analogy."
```

Every variable referenced inside the `{}` gets inserted directly into the resulting string — this is the most common, practical way variables and prompt construction intersect in real AI code.

## A Common Pitfall: Overwriting Variables You Still Need

A frequent mistake in AI scripts, especially ones built by iterating quickly, is reusing the same variable name for genuinely different things, losing an earlier value you actually needed later:

```python
# Risky pattern
response = call_model(prompt_1)
response = call_model(prompt_2)  # the first response is now gone
```

If both responses matter, they need distinct variable names — or should be collected into a list or dictionary rather than repeatedly overwriting the same name:

```python
response_1 = call_model(prompt_1)
response_2 = call_model(prompt_2)

# or, for many calls:
responses = []
for prompt in prompts:
    responses.append(call_model(prompt))
```

This is a small habit, but it's a very common source of confusing, hard-to-debug mistakes in scripts that grew quickly from a quick experiment into something more substantial.

## Environment Variables: A Special, Security-Relevant Case

As covered in the previous post, API keys and other sensitive configuration values should generally live in environment variables rather than being hardcoded directly into a script as a regular variable:

```python
import os
api_key = os.environ.get("ANTHROPIC_API_KEY")
```

This is technically still "a variable holding a value," but the value comes from outside the script itself, rather than being written directly into the code — an important habit for keeping sensitive credentials out of version-controlled files.

## The Bottom Line

Variables are the most basic building block in any Python script, but in generative AI development specifically, they're doing real, central work — holding prompts, parameters, responses, and conversation state that flow through every API call. Clear naming, an understanding of mutability and scope, comfort with f-strings for building dynamic prompts, and good habits around configuration and sensitive credentials might seem like small fundamentals, but they're exactly the foundation that keeps AI scripts readable, debuggable, and reliable as they grow from a quick experiment into something you actually depend on.
