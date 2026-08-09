Title: Tokens: The Building Blocks of LLMs (Part 2)
Date: 2025-12-19
Category: GenAI
Tags: GenAI, LLM, tokens, NLP, tokenization
Slug: tokens-building-blocks-of-llms-part2

Every time you type a message to ChatGPT or Claude, something happens before the model even starts "thinking": your text gets chopped up into small pieces called tokens. Tokens are the fundamental units that large language models actually read, process, and generate — and understanding them explains a surprising amount about how these systems behave, what they cost, and where their limits lie.

## What Exactly Is a Token?

A token is a chunk of text — but not necessarily a whole word. Depending on the tokenizer, a token might be:

- **A whole common word:** "the," "cat," "run"
- **A piece of a longer or rarer word:** "generat" + "ion" for "generation"
- **A single character or punctuation mark:** ",", ".", "!"
- **A prefix or suffix:** "un" + "happy" + "ness"

For example, the sentence "Tokenization is fascinating" might be split into tokens like `Token`, `ization`, `is`, `fasc`, `inating` — five tokens for three words. As a rough rule of thumb, one token is often about ¾ of an English word, though this varies by language and content.

## Why Not Just Use Whole Words?

It might seem simpler to treat each word as one unit, but that approach breaks down quickly:

- **Vocabulary size would explode.** A model would need a separate entry for every possible word, including rare words, names, typos, and slang — an impossibly large and inefficient list.
- **New or unseen words would break the model.** If "chatgptification" isn't in the dictionary, a whole-word system has no way to handle it.
- **Different languages behave differently.** Some languages don't even use spaces to separate words, so word-based splitting doesn't generalize well.

Sub-word tokenization solves all of this. By breaking rare or complex words into smaller familiar pieces, the model can represent virtually any input — including made-up words, typos, and multiple languages — using a manageable, fixed-size vocabulary (often in the tens of thousands of tokens).

## How Tokenization Actually Works

Most modern LLMs use algorithms like Byte-Pair Encoding (BPE) or similar sub-word tokenization methods. At a high level:

1. Start with individual characters as the smallest units.
2. Scan a massive training dataset and find the most frequently occurring pairs of characters or character sequences.
3. Merge those frequent pairs into single tokens.
4. Repeat this process thousands of times, gradually building up a vocabulary of common word pieces, whole words, and even common multi-word chunks.

The result is a tokenizer that represents frequent words (like "the" or "is") as single tokens, while breaking rarer or more complex words into smaller, reusable pieces.

## From Tokens to Numbers

Once text is split into tokens, each token is mapped to a unique number using the model's vocabulary — essentially a giant lookup table. These numbers are then converted into embeddings, lists of numbers that capture each token's meaning in a mathematical space the model can actually compute with. This is the format the model works in internally; it never "sees" raw letters or words the way we do.

## Tokens and Generation

When an LLM generates a response, it's not writing whole sentences at once — it's predicting one token at a time, based on everything written so far. Each new token is added to the sequence, and the model reconsiders the whole context before predicting the next one. This loop repeats until the response is complete. This is why generation can sometimes feel like it's happening "live," word by word (or piece by piece) — because it is.

## Why Tokens Matter Beyond the Technical Details

Tokens aren't just an internal implementation detail — they show up in practical, user-facing ways:

- **Context limits are measured in tokens, not words.** When a model has a "128k context window," that's 128,000 tokens — including your prompt, conversation history, and the model's response. Long conversations or documents can hit this limit.
- **API pricing is usually per token.** Both input and output tokens typically count toward cost, which is why longer prompts and responses cost more.
- **Some languages are less token-efficient than others.** Languages with different scripts or structures may require more tokens to express the same idea as English, affecting both cost and effective context length.
- **Odd model behavior can trace back to tokenization.** Struggles with tasks like counting letters in a word or reversing text often happen because the model is working with token chunks, not individual characters — it may never "see" a word broken into its individual letters at all.

## The Bottom Line

Tokens are the atomic units of language models — the bridge between human-readable text and the numbers a model actually computes with. By breaking language into flexible, reusable sub-word pieces, tokenization lets LLMs handle virtually any input efficiently, while also shaping practical realities like context limits, pricing, and even some of the quirky mistakes these models make. Once you understand tokens, a lot of LLM behavior — both impressive and puzzling — starts to make a lot more sense.
