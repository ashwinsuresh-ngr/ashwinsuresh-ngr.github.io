Title: How Large Language Models Work
Date: 2025-12-16
Category: GenAI
Tags: GenAI, LLM, transformers, deep-learning
Slug: how-large-language-models-work

Large Language Models (LLMs) like GPT-4, Claude, and Gemini can write essays, debug code, and hold surprisingly natural conversations. But under the hood, they're not "thinking" the way humans do — they're performing an extraordinarily sophisticated version of pattern prediction. Here's how it actually works.

## The Core Idea: Predicting the Next Token

At their heart, LLMs do one thing: predict the next piece of text, given everything that came before it. That "piece" is called a token — which might be a whole word, part of a word, or even punctuation.

Given the phrase "The cat sat on the ___," the model calculates probabilities for what comes next — "mat" might be highly likely, "moon" less likely, "banana" very unlikely — and picks one accordingly. Generate one token, add it to the text, repeat. String enough of these predictions together, and you get coherent paragraphs, code, or conversations.

The impressive part isn't the concept — it's the scale and nuance behind it.

## Step 1: Tokenization

Before a model can process text, it breaks it into tokens using a process called tokenization. Rather than working with whole words, models often split text into smaller sub-word chunks. This lets them handle rare words, typos, and multiple languages more efficiently than a word-by-word approach would.

## Step 2: Embeddings — Turning Words into Numbers

Computers don't understand words directly — they need numbers. Each token gets converted into an embedding: a list of numbers (a vector) that captures its meaning in a mathematical space. Words with similar meanings end up with similar embeddings, so "king" and "queen" sit closer together than "king" and "banana."

## Step 3: The Transformer Architecture

This is the real breakthrough behind modern LLMs. Introduced in a 2017 paper, the transformer architecture allows models to weigh the relationships between all the words in a passage simultaneously, rather than processing text strictly left to right.

The key mechanism is called self-attention. For every word, the model asks: "Which other words in this sentence matter most for understanding me?" In the sentence "The trophy didn't fit in the suitcase because it was too big," attention helps the model figure out that "it" refers to the trophy, not the suitcase — by weighing the relevance of every other word in context.

Transformers stack many of these attention layers on top of each other, allowing the model to build increasingly abstract and nuanced understanding of language, context, and meaning as information flows through the network.

## Step 4: Training on Massive Datasets

LLMs learn by processing enormous amounts of text — books, websites, articles, code repositories — often hundreds of billions or trillions of words. During training, the model repeatedly tries to predict the next token in real text, compares its guess to the actual answer, and adjusts billions of internal parameters (weights) slightly to improve. This happens billions of times, gradually tuning the model to capture grammar, facts, reasoning patterns, and style.

This initial phase is called pretraining, and it's what gives the model its broad general knowledge and language ability.

## Step 5: Fine-Tuning and Alignment

A freshly pretrained model is powerful but raw — it's good at predicting text, not necessarily at being a helpful assistant. That's where additional training steps come in:

- **Supervised fine-tuning:** The model is trained on curated examples of good question-answer pairs, teaching it to follow instructions.
- **Reinforcement learning from human feedback (RLHF):** Human reviewers rank different model responses, and the model is further trained to produce outputs people rate as more helpful, accurate, and safe.

This is the stage that transforms a raw text predictor into something that behaves like a useful assistant with a consistent tone and behavior.

## Step 6: Generating a Response

When you type a prompt, the model doesn't "look up" an answer — it processes your input through all its trained layers, calculates probabilities for the next token, samples one, then repeats the process token by token until it produces a full response. Small amounts of randomness (controlled by a setting often called "temperature") let the model vary its phrasing rather than always giving identical, robotic answers.

## Why This Matters: Strengths and Limitations

Understanding this mechanism explains a lot about LLM behavior:

- **They're excellent at fluent, context-aware language** — because attention lets them track relationships across long passages.
- **They can "hallucinate"** — since they're predicting plausible-sounding text, not retrieving verified facts, they can confidently generate incorrect information.
- **They don't have real-time knowledge by default** — their knowledge comes from training data up to a certain cutoff, unless connected to external tools like search.
- **They don't truly "understand" like humans** — they're recognizing and reproducing extremely sophisticated statistical patterns in language, not reasoning from lived experience.

## The Bottom Line

Large Language Models are, at their core, incredibly advanced next-token predictors — built on transformer architectures, trained on massive datasets, and refined through human feedback to be genuinely useful. They don't "know" things the way people do, but by learning the deep statistical structure of human language, they've become remarkably capable at writing, reasoning, and conversing. Understanding this mechanism — prediction, not true comprehension — is key to using these tools effectively and knowing where to trust them, and where to double-check.
