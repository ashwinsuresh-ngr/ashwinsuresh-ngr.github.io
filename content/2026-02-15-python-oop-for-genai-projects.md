Title: Python OOP for GenAI Projects
Date: 2026-02-15
Category: GenAI
Tags: GenAI, Python, LLM, OOP, developers
Slug: python-oop-for-genai-projects

The previous post covered classes as a practical tool for managing state in AI applications — conversations, sessions, agents. This one zooms out to the bigger picture: object-oriented programming (OOP) as a design philosophy, and specifically the four core principles — encapsulation, abstraction, inheritance, and polymorphism — that make it genuinely useful for structuring larger generative AI projects, not just a single conversation class.

## Why OOP Matters More As a GenAI Project Grows

A single script calling a model once doesn't need much structure. But real GenAI projects tend to grow quickly in complexity: multiple assistant types, different prompt strategies, various tools an agent can call, several API providers to support, evaluation pipelines, retry logic. Without some organizing structure, that complexity turns into a tangle of loosely related functions and duplicated logic. OOP's core principles exist specifically to manage that kind of growing complexity — and they map onto GenAI development more naturally than you might expect.

## Encapsulation: Hiding Complexity Behind a Clean Interface

Encapsulation means bundling data and the logic that operates on it together, while hiding the messy internal details behind a simple, predictable interface. The person using a class shouldn't need to know how it works internally — just what it does.

```python
class RetryableAIClient:
    def __init__(self, api_key, max_retries=3):
        self._api_key = api_key  # underscore signals "internal, not for external use"
        self._max_retries = max_retries
        self._client = anthropic.Anthropic(api_key=api_key)

    def generate(self, prompt, model="claude-sonnet-4-6"):
        for attempt in range(self._max_retries):
            try:
                return self._call(prompt, model)
            except Exception as e:
                if attempt == self._max_retries - 1:
                    return f"Failed after {self._max_retries} attempts: {e}"

    def _call(self, prompt, model):
        response = self._client.messages.create(
            model=model, max_tokens=500,
            messages=[{"role": "user", "content": prompt}]
        )
        return response.content[0].text
```

Anyone using `RetryableAIClient` just calls `.generate(prompt)` — they don't need to know or care about retry counts, internal client objects, or how failures get handled underneath. That complexity is encapsulated, connecting directly to the "design for fallback behavior" principle from the developer-focused prompt engineering post — the resilience lives in one place, hidden behind a simple call.

## Abstraction: Designing Around "What," Not "How"

Abstraction is closely related to encapsulation but focuses specifically on defining a consistent interface that different implementations can share, letting the rest of your code work with "an AI client" in general, without caring which specific one it's actually using.

```python
from abc import ABC, abstractmethod

class AIProvider(ABC):
    @abstractmethod
    def generate(self, prompt: str) -> str:
        pass


class AnthropicProvider(AIProvider):
    def __init__(self, api_key):
        self.client = anthropic.Anthropic(api_key=api_key)

    def generate(self, prompt: str) -> str:
        response = self.client.messages.create(
            model="claude-sonnet-4-6", max_tokens=500,
            messages=[{"role": "user", "content": prompt}]
        )
        return response.content[0].text


class OpenAIProvider(AIProvider):
    def __init__(self, api_key):
        self.client = OpenAI(api_key=api_key)

    def generate(self, prompt: str) -> str:
        response = self.client.chat.completions.create(
            model="gpt-4", messages=[{"role": "user", "content": prompt}]
        )
        return response.choices[0].message.content
```

Any code written against `AIProvider` works identically regardless of which concrete provider is actually plugged in:

```python
def summarize(provider: AIProvider, text: str) -> str:
    return provider.generate(f"Summarize this text:\n\n{text}")
```

This is genuinely valuable in practice — it's what makes it realistic to support multiple model providers, swap one out for testing, or add a new one later, without rewriting the logic that actually uses them.

## Inheritance: Sharing Structure Across Related Assistants

The previous post touched on inheritance briefly — this is where it earns its keep at project scale. Rather than building several assistant classes from scratch, each with duplicated conversation-handling logic, a shared base class captures the common behavior once:

```python
class BaseAssistant:
    def __init__(self, system_prompt, temperature=0.7):
        self.system_prompt = system_prompt
        self.temperature = temperature
        self.history = []

    def send(self, message):
        self.history.append({"role": "user", "content": message})
        response = call_model(self.system_prompt, self.history, self.temperature)
        self.history.append({"role": "assistant", "content": response})
        return response

    def reset(self):
        self.history = []


class CodeReviewAssistant(BaseAssistant):
    def __init__(self):
        super().__init__(
            system_prompt="You are a senior engineer reviewing code for bugs and readability.",
            temperature=0.2  # lower temperature for consistent, precise review
        )

    def review(self, code):
        return self.send(f"Review this code:\n\n{code}")


class BrainstormAssistant(BaseAssistant):
    def __init__(self):
        super().__init__(
            system_prompt="You are a creative brainstorming partner, generating varied ideas.",
            temperature=1.0  # higher temperature for more varied, creative output
        )

    def generate_ideas(self, topic, count=5):
        return self.send(f"Generate {count} creative ideas about: {topic}")
```

Both subclasses inherit conversation management, history tracking, and reset behavior from `BaseAssistant` — while each sets its own system prompt and temperature (connecting directly to the temperature and top-p post: precision-focused review wants low temperature, creative brainstorming wants higher) and adds its own specialized method. Fix a bug in `send()` once, and every assistant type benefits, rather than needing the same fix applied in five different places.

## Polymorphism: Treating Different Things the Same Way

Polymorphism means different classes can be used interchangeably through a shared interface, each responding to the same method call in its own way. This follows naturally from the abstraction example above — code that works with an `AIProvider` doesn't need to know or care which specific provider it actually received:

```python
def run_comparison(providers: list[AIProvider], prompt: str):
    for provider in providers:
        result = provider.generate(prompt)
        print(f"{type(provider).__name__}: {result}\n")

run_comparison(
    [AnthropicProvider(api_key=key1), OpenAIProvider(api_key=key2)],
    "Explain entropy in one sentence."
)
```

The same loop calls `.generate()` on every provider without any special-casing — each object handles the call according to its own internal implementation, but from the outside, they're interchangeable. This is exactly the kind of pattern that makes A/B testing prompt strategies across providers, or swapping a production model for a cheaper one during testing, a small configuration change rather than a rewrite.

## A Combined Example: Putting the Four Principles Together

A small but realistic illustration of all four principles working together — an evaluation harness that tests multiple assistant configurations against multiple providers:

```python
class Evaluator:
    def __init__(self, provider: AIProvider):
        self.provider = provider  # abstraction: works with any AIProvider
        self._results = []        # encapsulation: internal state, accessed via methods

    def run(self, test_prompts: list[str]):
        for prompt in test_prompts:
            output = self.provider.generate(prompt)  # polymorphism: works regardless of provider type
            self._results.append({"prompt": prompt, "output": output})

    def results(self):
        return self._results


anthropic_eval = Evaluator(AnthropicProvider(api_key=key1))
openai_eval = Evaluator(OpenAIProvider(api_key=key2))

test_set = ["Summarize the theory of relativity.", "Explain what a REST API is."]
anthropic_eval.run(test_set)
openai_eval.run(test_set)
```

This connects directly back to the prompt testing strategies post — running the same test set against different configurations is exactly the kind of comparison that discipline calls for, and OOP structure is what makes that comparison clean to set up rather than a tangle of parallel scripts.

## When OOP Structure Is Worth It — and When It Isn't

OOP adds real design overhead, and it's not free. It's worth reaching for specifically when:

- **You have genuinely related variations** — several assistant types, multiple providers, different agent behaviors — that share common structure worth factoring out
- **State needs managing across a session or multi-step process** — as covered in the previous classes post
- **The project will grow and be maintained by more than just you**, where a clear, consistent structure pays off in reduced confusion later
- **You want to swap implementations** — different providers, different prompt strategies — without rewriting the code that uses them

It's often not worth it for a quick script, a one-off experiment, or a task you'll run once and discard. As covered in the functions post, plain functions are simpler and perfectly sufficient for straightforward, one-shot AI calls — OOP earns its complexity specifically when a project's shape genuinely benefits from it, not by default.

## The Bottom Line

Object-oriented programming gives generative AI projects a set of proven tools for managing complexity as they grow beyond a single script: encapsulation hides messy internals behind clean interfaces, abstraction lets code work with "an AI provider" or "an assistant" in general rather than one specific implementation, inheritance shares common structure across related variations instead of duplicating it, and polymorphism lets different implementations be swapped in and out interchangeably. None of this replaces the fundamentals covered earlier in this series — clear prompts, tested reliability, sound context — but it's what keeps a GenAI project's code itself maintainable, extensible, and comprehensible as it grows from a single API call into a real, evolving application.
