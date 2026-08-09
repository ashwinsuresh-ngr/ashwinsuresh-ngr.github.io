Title: Python Classes for AI Applications
Date: 2026-02-14
Category: GenAI
Tags: GenAI, Python, LLM, developers, tutorial
Slug: python-classes-for-ai-applications

Functions are great for a single, self-contained piece of logic — summarize this text, classify this sentiment. But a lot of real generative AI applications need something functions alone don't handle well: state that persists and evolves over time — an ongoing conversation, an agent's memory, a running record of tool calls. That's where classes come in. Here's how they show up in practical AI development.

## Why Classes, When Functions Got You This Far

As covered in the previous post, functions are excellent at packaging a repeatable piece of logic behind a name. But functions are stateless by default — call one twice, and it has no memory of the first call unless you explicitly pass that history back in every time. For a chatbot that needs to remember earlier messages, or an agent that accumulates results across several steps, constantly passing the entire state into and out of every function call gets unwieldy fast. Classes solve this by bundling data (state) and the functions that operate on it (methods) together into a single object that can hold onto that state between calls.

## The Basic Anatomy

A class is a blueprint for creating objects that carry their own data and behavior:

```python
class ConversationManager:
    def __init__(self):
        self.history = []

    def add_message(self, role, content):
        self.history.append({"role": role, "content": content})

    def get_history(self):
        return self.history
```

`__init__` is the constructor — it runs once when a new object is created, setting up its initial state. `self` refers to the specific object being worked with, letting each instance keep its own independent data.

```python
conversation = ConversationManager()
conversation.add_message("user", "What's the capital of France?")
conversation.add_message("assistant", "The capital of France is Paris.")

print(conversation.get_history())
```

This connects directly to the conversation history management ideas from the context engineering post — a class like this is exactly how that "growing list of prior turns" typically gets managed in real code, rather than as a loose variable passed manually through every function.

## Why This Matters Specifically for Conversational AI

A chatbot interaction isn't really one API call — it's a sequence of calls, each one needing the full conversation so far as context (as covered in the prompt vs. completion post). A class is a natural fit for managing that:

```python
class ChatSession:
    def __init__(self, system_prompt):
        self.system_prompt = system_prompt
        self.history = []

    def send(self, user_message):
        self.history.append({"role": "user", "content": user_message})
        response = call_model(self.system_prompt, self.history)
        self.history.append({"role": "assistant", "content": response})
        return response

chat = ChatSession(system_prompt="You are a helpful coding tutor.")
reply_1 = chat.send("What's a Python list comprehension?")
reply_2 = chat.send("Can you give me an example?")  # still has context from reply_1
```

Each call to `.send()` automatically has access to everything that came before, because it's stored on `self.history` — no need to manually thread the growing conversation through every function call by hand.

## Attributes: The Data a Class Carries

Attributes are the variables attached to an object — the state it holds onto between method calls:

```python
class ChatSession:
    def __init__(self, system_prompt, model="claude-sonnet-4-6", temperature=0.7):
        self.system_prompt = system_prompt
        self.model = model
        self.temperature = temperature
        self.history = []
```

Here, `model` and `temperature` (covered in the earlier sampling parameters post) are stored once when the session is created, rather than needing to be passed into every single call — the object remembers its own configuration.

## Methods: The Behavior a Class Provides

Methods are just functions defined inside a class, operating on that object's own data via `self`:

```python
class ChatSession:
    # ... (init as above)

    def reset(self):
        self.history = []

    def last_response(self):
        for message in reversed(self.history):
            if message["role"] == "assistant":
                return message["content"]
        return None
```

Grouping related behavior — sending a message, resetting the session, retrieving the last response — under one object keeps everything about "a single conversation" logically together, rather than scattered across loose functions that all happen to take the same history list as an argument.

## A Practical Pattern: Wrapping an API Client

Classes are also a natural fit for wrapping repeated API configuration and error handling (covered in the previous post) into a single reusable object:

```python
class AIClient:
    def __init__(self, api_key, default_model="claude-sonnet-4-6"):
        self.client = anthropic.Anthropic(api_key=api_key)
        self.default_model = default_model

    def generate(self, prompt, model=None, max_tokens=500):
        try:
            response = self.client.messages.create(
                model=model or self.default_model,
                max_tokens=max_tokens,
                messages=[{"role": "user", "content": prompt}]
            )
            return response.content[0].text
        except Exception as e:
            return f"Error: {e}"

ai = AIClient(api_key=os.environ.get("ANTHROPIC_API_KEY"))
result = ai.generate("Explain recursion in one sentence.")
```

This bundles the API key, default settings, and error handling into one object, created once and reused everywhere — rather than re-specifying the model name and repeating try/except logic in every individual function.

## Managing Agent-Style State

As AI applications move toward more agentic behavior — calling tools, taking multiple steps, tracking progress (touched on in the prompt injection posts) — classes become especially useful for holding onto everything an agent needs to track across a multi-step task:

```python
class Agent:
    def __init__(self, goal):
        self.goal = goal
        self.steps_taken = []
        self.completed = False

    def take_step(self, action, result):
        self.steps_taken.append({"action": action, "result": result})

    def mark_complete(self):
        self.completed = True

    def summary(self):
        return f"Goal: {self.goal}\nSteps: {len(self.steps_taken)}\nCompleted: {self.completed}"
```

This gives a multi-step process a clear, inspectable record of what's happened so far — useful both for the agent's own logic (deciding what to do next based on prior steps) and for the kind of monitoring and logging covered in the prompt injection prevention post.

## Inheritance: Sharing Behavior Across Related Classes

Inheritance lets one class build on another, reusing shared behavior while customizing specific pieces — useful when you have several related but slightly different AI-calling objects:

```python
class BaseAssistant:
    def __init__(self, system_prompt):
        self.system_prompt = system_prompt
        self.history = []

    def send(self, message):
        self.history.append({"role": "user", "content": message})
        response = call_model(self.system_prompt, self.history)
        self.history.append({"role": "assistant", "content": response})
        return response


class CodingAssistant(BaseAssistant):
    def __init__(self):
        super().__init__(system_prompt="You are an expert Python coding assistant.")

    def review_code(self, code):
        return self.send(f"Review this code for bugs and style issues:\n\n{code}")
```

`CodingAssistant` inherits the core conversation-handling logic from `BaseAssistant` (via `super().__init__()`), while adding its own specialized method. This avoids duplicating the same conversation-management logic across several slightly different assistant types.

## When Classes Are Worth the Extra Structure

Classes add a bit of upfront overhead compared to a plain function, so it's worth knowing when that trade-off pays off:

- **State needs to persist across multiple calls** — conversation history, agent progress, accumulated results
- **Multiple related pieces of configuration travel together** — API key, model name, temperature, system prompt, all tied to the same session or client
- **You have several related variations of similar behavior** — different assistant types that share a common core (a good fit for inheritance)
- **You're managing something with a clear lifecycle** — start a session, use it repeatedly, reset or close it

For a single, one-off script — call a model once, get a result, done — a plain function is simpler and perfectly sufficient. Reach for a class when you notice yourself passing the same growing bundle of state into function after function.

## The Bottom Line

Classes matter in generative AI development specifically because so much of it involves state that accumulates and persists — conversation history, agent progress, session configuration — rather than the one-shot input-output pattern a simple function handles well. Bundling that state together with the methods that operate on it keeps conversational and agentic AI code organized and easy to reason about, especially as an application grows from a single API call into something that manages an ongoing interaction or a multi-step task. Combined with the functions and fundamentals covered earlier in this series, classes round out the core Python toolkit worth having before diving deeper into AI frameworks and agent-building libraries.
