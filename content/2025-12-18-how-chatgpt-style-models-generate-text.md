Title: How ChatGPT-Style Models Generate Text
Date: 2025-12-18
Category: GenAI
Tags: GenAI, LLM, ChatGPT, transformers
Slug: how-chatgpt-style-models-generate-text

Ever wonder what's actually happening between the moment you hit "enter" on a prompt and the moment text starts appearing on your screen? It's not retrieval, and it's not magic — it's a very fast, very structured process of prediction happening one small piece at a time. Here's what's really going on.

## It All Comes Down to One Word at a Time

ChatGPT-style models don't write a full response in one shot. They generate text token by token — where a token is roughly a word, part of a word, or a punctuation mark. Each token is chosen based on everything written so far, including your prompt and every token the model has already generated in its own response.

So if the model has written "The weather today is," it now calculates: given this exact sequence, what's the most probable next token? Maybe "sunny," maybe "cold," maybe "unpredictable." It picks one, appends it to the text, and repeats the process — using the new, longer sequence as the basis for the next prediction.

This loop continues until the model decides the response is complete (by generating a special "stop" signal) or hits a length limit.

## Step 1: Your Prompt Gets Tokenized

The moment you send a message, it's broken down into tokens — the same units the model was trained on. "Generating text" might become tokens like "Gener," "ating," and "text," for example. This lets the model handle any input, including rare words, typos, and multiple languages, without needing a fixed dictionary of whole words.

## Step 2: The Model Builds Context

Each token is converted into a numerical representation (an embedding) that captures its meaning. Then, through the transformer architecture's self-attention mechanism, the model looks at how every token in the input relates to every other token — figuring out what's relevant to what, resolving ambiguity, and building a rich contextual understanding of the entire conversation so far.

This is why the model can correctly track pronouns, follow multi-turn conversations, and stay on topic even in long exchanges — it's constantly weighing the full context, not just the last sentence.

## Step 3: Calculating Probabilities

For every possible next token — out of tens of thousands of options in its vocabulary — the model calculates a probability score based on the patterns it learned during training. This produces something like a ranked list: "sunny" might get a 40% probability, "cold" 25%, "great" 10%, and so on down a long tail of less likely options.

## Step 4: Sampling the Next Token

Here's where a bit of controlled randomness comes in. Rather than always picking the single highest-probability token (which would make responses repetitive and robotic), the model samples from that probability distribution. Settings like temperature control how much randomness is involved — lower temperature makes output more focused and predictable, higher temperature makes it more varied and creative.

## Step 5: Repeat, Repeat, Repeat

That new token gets added to the sequence, and the entire process — attention, probability calculation, sampling — happens again for the next token. And the next. And the next. This continues at high speed until a full response has been built, token by token, each one shaped by everything that came before it.

## Why Responses Feel Coherent

Because the model reconsiders the entire context every single time it generates a new token, its output stays consistent with earlier parts of the response and the conversation. It's effectively "re-reading" everything so far before deciding on each next word — which is part of why responses can maintain tone, follow instructions, and stay on-topic across long answers.

## Why This Explains Some Quirks

- **Hallucinations happen** because the model is optimizing for plausible-sounding continuations, not verified truth — a very confident wrong answer can have a high probability score too.
- **Responses can vary between runs** because of the sampling step — ask the same question twice and you may get slightly different phrasing.
- **Long responses can drift** if earlier tokens nudge the probability distribution in a direction that compounds over many steps.
- **It "thinks" as it writes, not before** — there's no separate planning stage where it drafts the whole answer first, though some newer models do generate intermediate reasoning steps before producing a final answer.

## The Bottom Line

ChatGPT-style models generate text through a rapid, repeating cycle: read the context, calculate probabilities for the next token, sample one, and repeat — thousands of times per response, often in a matter of seconds. There's no lookup table and no hidden "understanding" in the human sense — just an extraordinarily well-trained prediction engine, built on the transformer architecture, generating language one carefully weighted choice at a time.
