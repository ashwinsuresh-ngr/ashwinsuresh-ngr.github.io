Title: Prompt vs Completion
Date: 2025-12-29
Category: GenAI
Tags: GenAI, LLM, prompt-engineering, tokens
Slug: prompt-vs-completion

If you've worked with any AI language model — through an app, an API, or a chatbot — you've encountered these two terms, even if you didn't know their formal names. Every interaction with an LLM boils down to two halves: what you send in, and what the model sends back. Understanding this split clarifies a lot about how these systems actually work.

## The Basic Definitions

- **Prompt:** The input you give the model — your question, instruction, or starting text. It's everything the model sees before it starts generating a response.
- **Completion:** The output the model generates in response to that prompt — the text it produces, token by token, based on the patterns it learned during training and the context you provided.

In the simplest terms: prompt is what you ask, completion is what the model answers.

## Where the Terms Come From

The terminology traces back to how LLMs fundamentally work — as next-token predictors. Early language model APIs (like OpenAI's original GPT-3 API) were literally framed as "text completion" tools: you'd give the model a chunk of text, and it would predict a plausible continuation. Ask it to complete "Once upon a time," and it would generate a story. The model wasn't "answering a question" in a structured sense — it was just continuing whatever text it was given.

Modern chat-style models like ChatGPT and Claude still work this way under the hood, even though the interface feels more like a conversation. Your message, the system instructions, and the conversation history all get combined into one long prompt, and the model generates a completion in response — it just happens to be trained to complete that prompt in the style of a helpful reply rather than a raw continuation.

## What Actually Makes Up "The Prompt"

In modern LLM applications, the prompt is often more than just what you typed. It can include several layered components:

- **System instructions** — hidden guidance that shapes the model's behavior, tone, or role (e.g., "You are a helpful coding assistant")
- **Conversation history** — previous messages in the chat, included so the model has context
- **Your actual message** — the specific question or instruction you just sent
- **Any additional context** — uploaded documents, retrieved search results, or tool outputs added into the mix

All of this gets combined into a single sequence of tokens that the model reads before generating anything. This is why very long conversations or large documents can "fill up" a model's context window — the whole thing counts as part of the prompt.

## What Actually Makes Up "The Completion"

The completion is simply the new tokens the model generates after processing the prompt. It's built one token at a time — the model predicts the next token, appends it, then re-reads the entire sequence (prompt + everything generated so far) to predict the next one, continuing until it reaches a natural stopping point or hits a length limit.

Importantly, the completion is not fetched or retrieved — it's generated fresh, based purely on patterns learned during training, shaped by the specific prompt provided.

## Why This Distinction Matters

Understanding prompt vs. completion isn't just semantics — it has real practical implications:

- **Prompt engineering exists because of this relationship.** Since completions are generated entirely in response to the prompt, the way you phrase, structure, and provide context in your prompt directly shapes the quality of the output. Clearer, more specific prompts generally produce more useful completions.
- **API pricing is usually split by prompt and completion tokens**, often at different rates. Providers frequently charge less for prompt (input) tokens than completion (output) tokens, since generation is more computationally expensive than reading input.
- **Context windows are shared between the two.** A model's total context limit covers prompt tokens and completion tokens combined — a long prompt leaves less room for a long completion, and vice versa.
- **Debugging AI behavior often starts here.** If a model gives an odd or incomplete answer, the first place to look is usually the prompt — was necessary context included? Was the instruction ambiguous? The completion is only ever as good as what it had to work with.

## Prompt vs. Completion in Chat-Style Interfaces

In apps like ChatGPT or Claude, the conversational back-and-forth can obscure this structure, but it's still happening underneath. Every time you send a new message, the entire conversation so far — your messages and the model's previous completions — gets bundled back into a fresh prompt, and the model generates a new completion in response. Each "turn" is technically a brand-new prompt-completion cycle, even though it feels like one continuous conversation.

## The Bottom Line

Prompt and completion are the two halves of every interaction with a language model: the prompt is the input — instructions, context, and conversation history — and the completion is the model's generated output, built one token at a time in response. This simple input/output relationship underlies everything from casual chatbot conversations to complex AI applications, and understanding it is often the first step toward getting better, more reliable results out of any LLM.
