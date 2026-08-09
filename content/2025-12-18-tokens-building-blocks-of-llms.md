Title: Tokens: The Building Blocks of LLMs
Date: 2025-12-18
Category: GenAI
Tags: GenAI, LLM, tokens, NLP
Slug: tokens-building-blocks-of-llms

If you've ever noticed that AI pricing is measured in "tokens," or seen a model get confused by an unusual word, you've bumped into one of the most fundamental — and least discussed — parts of how LLMs work. Before a model can predict, reason, or write a single sentence, it first has to break language down into tokens. Here's what they are and why they matter so much.

## What Exactly Is a Token?

A token is the basic unit of text that a language model reads and generates. It's tempting to assume a token equals a word, but that's not quite right. Depending on the word, a token might be:

- **A whole word** — "cat" is one token
- **Part of a word** — "unbelievable" might split into "un," "believ," and "able"
- **A single character or symbol** — punctuation, emojis, or rare characters often become their own tokens
- **A few characters combined** — common chunks like "ing" or "tion" often exist as single tokens

As a rough rule of thumb, in English, one token is about 4 characters or ¾ of a word on average. So a 100-word paragraph might translate to roughly 130–150 tokens.

## Why Not Just Use Whole Words?

It seems simpler to just treat every word as one unit. But that approach breaks down fast:

- **Vocabulary size would explode.** Human language has an effectively unlimited number of words, including names, slang, typos, and technical jargon. A whole-word vocabulary would need to be enormous — and would still miss words it's never seen before.
- **Rare and made-up words would break the model.** If "un-word-like-this" never appeared in training data, a whole-word system wouldn't know what to do with it.
- **Multiple languages would be a nightmare.** Different languages have wildly different word structures — some barely use spaces at all.

Breaking text into smaller sub-word pieces solves this. Even a word the model has never seen before can usually be broken into familiar chunks it does recognize, letting it make a reasonable guess.

## How Tokenization Actually Works

Most modern LLMs use a method called byte-pair encoding (BPE) or similar sub-word tokenization algorithms. Here's the basic idea:

1. Start by treating every individual character as a token.
2. Scan a massive amount of training text and find the most frequently occurring pairs of adjacent tokens.
3. Merge that pair into a single new token.
4. Repeat this process thousands of times.

Over many iterations, common chunks like "ing," "tion," or entire frequent words like "the" or "and" become single tokens, while rarer words stay broken into smaller pieces. The result is a fixed vocabulary — often 50,000 to 200,000 tokens — that balances efficiency with flexibility.

## Tokens Aren't Just for Text Input

Every part of an LLM's process runs through tokens:

- Your prompt is broken into tokens before the model processes it.
- The model's internal predictions are calculated one token at a time.
- The response you see is generated token by token, then reassembled into readable text.

This is also why LLMs have a context window — a maximum number of tokens they can "see" and reason about at once, encompassing both your input and the model's own output. A model with a 128,000-token context window can hold a very large conversation or document in memory; once that limit is hit, older content has to be dropped or summarized.

## Why Tokens Matter to You

Understanding tokens explains a few practical things:

- **Pricing** — Most AI APIs charge based on the number of tokens processed, both for input and output, which is why long conversations or documents cost more.
- **Context limits** — If a conversation feels like it's "forgetting" earlier details, it may be because older tokens fell outside the context window.
- **Odd behavior with unusual words** — Rare words, made-up terms, or non-English text can get split into unfamiliar token combinations, sometimes leading to less accurate results.
- **Response length limits** — Output is capped by a maximum token count, which is why very long responses sometimes get cut off.

## A Quick Example

The sentence:

"ChatGPT is transforming how we work."

Might tokenize into something like:

`Chat` `G` `PT` `is` `transform` `ing` `how` `we` `work` `.`

That's 10 tokens for an 8-word sentence — showing how tokenization doesn't map neatly onto whole words, especially for less common terms like "ChatGPT."

## The Bottom Line

Tokens are the invisible building blocks behind every LLM interaction — the units models actually read, think in, and generate. By breaking language into flexible sub-word pieces rather than whole words, tokenization lets models handle any input, from common English sentences to rare technical terms to entirely new languages, all with a manageable, fixed vocabulary. It's a small design choice with outsized consequences — shaping everything from how much a query costs to how well a model handles the words you throw at it.
