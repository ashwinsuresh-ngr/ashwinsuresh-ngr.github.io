Title: Self-Attention in Transformers
Date: 2026-01-18
Category: GenAI
Tags: GenAI, LLM, attention, transformers, deep-learning
Slug: self-attention-in-transformers

Self-attention is the mechanism that made transformers possible — and by extension, made models like GPT and Claude possible. It's been touched on in a few earlier posts in this series, but it deserves its own deep dive, since it's genuinely the core computational idea the entire modern AI boom is built on. Here's a closer, more technical look at how it actually works.

## What "Self" Means Here

Attention, broadly, is a mechanism for letting a model weigh the relevance of different pieces of information when processing something. Self-attention specifically means every element in a sequence attends to every other element within that same sequence — words in a sentence relate to other words in that same sentence, rather than to some separate external source. It's the model referencing itself internally, word against word, to build context.

## The Three Vectors: Query, Key, Value

Every token in the input gets transformed into three separate vectors, each learned during training via its own weight matrix:

- **Query (Q)** — represents what this token is "looking for" from the rest of the sequence
- **Key (K)** — represents what this token "advertises" about itself, for other tokens to compare against
- **Value (V)** — represents the actual information content this token contributes, once it's deemed relevant

These aren't three arbitrary copies of the same data — they're each produced by multiplying the token's embedding by a separate learned weight matrix (Wq, Wk, Wv), so the model learns distinct, purpose-specific representations for "what am I looking for," "what do I offer," and "what do I actually contain."

## The Calculation, Step by Step

For a given token, self-attention works through the following sequence:

1. **Compute similarity scores.** Take that token's Query vector and compare it against the Key vector of every other token in the sequence (including itself), typically using a dot product. This produces a raw relevance score for every pair.
2. **Scale the scores.** The scores are divided by the square root of the key vector's dimension — a normalization step that keeps the values in a stable range, preventing extremely large dot products from destabilizing training. This is why you'll often see the technique called scaled dot-product attention.
3. **Apply softmax.** The scaled scores are passed through a softmax function, converting them into a clean probability distribution — a set of weights between 0 and 1 that sum to 1 across all tokens. This is the step that translates raw similarity scores into "how much attention should I actually pay to each token."
4. **Compute the weighted sum.** Multiply each token's Value vector by its corresponding attention weight, then sum them all together. The result is a new representation for the original token — one that's been enriched by pulling in exactly the right amount of context from every other token in the sequence, based on the weights just calculated.

This entire process happens for every single token simultaneously, which is part of what makes it so efficient on parallel hardware like GPUs.

## A Concrete Example

Take: "The cat sat on the mat because it was tired."

When computing the self-attention output for "it," the model's process looks like:

- "it"'s Query vector gets compared against the Key vector of every word in the sentence — "The," "cat," "sat," "on," "the," "mat," "because," "was," "tired."
- The dot products produce raw scores — likely highest between "it" and "cat," since that's the more plausible referent given the overall sentence pattern.
- Softmax converts these into weights — maybe 0.7 for "cat," 0.1 for "mat," and small residual weights spread across the rest.
- The final representation of "it" becomes a weighted blend of all the Value vectors, dominated by "cat"'s contribution — meaning "it" now carries strong contextual information pointing to "cat," even though no explicit rule was ever coded for pronoun resolution.

## Why Multi-Head Self-Attention Exists

Running this process just once captures only one "type" of relationship. Language contains many kinds of relationships simultaneously — grammatical dependencies, coreference, thematic association, tone — and a single attention calculation can't cleanly capture all of them at once.

Multi-head attention solves this by running several self-attention calculations in parallel, each with its own separate set of Q, K, V weight matrices. Each "head" can specialize in learning a different type of relationship during training — without being explicitly told to. The outputs of all heads are then concatenated and passed through another learned transformation, combining these different perspectives into one richer representation. Most modern LLMs use dozens of attention heads per layer.

## Self-Attention Is Permutation-Invariant (and Why That's a Problem)

One quirk worth understanding: the self-attention calculation itself has no inherent sense of word order. Mathematically, it treats the input as a set, not a sequence — shuffle the words, and the raw attention mechanism would produce the same relevance scores between the same pairs, regardless of their position.

This is why transformers add positional encoding — extra numerical information mixed into each token's embedding before attention is applied, indicating where in the sequence that token sits. Without it, "the dog bit the man" and "the man bit the dog" would be indistinguishable to the self-attention mechanism, since both sentences contain the exact same words.

## Self-Attention vs. Cross-Attention

It's worth distinguishing self-attention from a related mechanism called cross-attention, which shows up in encoder-decoder models (like translation systems). In self-attention, Query, Key, and Value all come from the same sequence. In cross-attention, the Query comes from one sequence (say, the output being generated) while the Key and Value come from a different sequence (say, the original input being translated). Most modern decoder-only LLMs like GPT and Claude rely primarily on self-attention, since they're generating text based on their own accumulating context rather than translating between two distinct sequences.

## Masked Self-Attention: Why Generation Only Looks Backward

There's an important variant used specifically during text generation, called masked self-attention. Since a model generating text token by token shouldn't be allowed to "cheat" by looking at tokens it hasn't generated yet, the attention calculation is masked so that each token can only attend to itself and the tokens that came before it — never ones that come later. This is what keeps generation causal and consistent with how the model actually produces text: strictly left to right, one token building on the ones before it.

## Why Self-Attention Scales So Well — and Its One Real Cost

Self-attention's biggest advantage is that it directly connects any two tokens in a sequence in a single computational step, regardless of how far apart they are — no information decay from passing through a long chain of intermediate steps, unlike older recurrent architectures.

The trade-off: computing attention between every pair of tokens means the computation and memory required grow quadratically with sequence length — double the input length, and the attention calculation roughly quadruples in cost. This is precisely why long context windows are computationally expensive, and why a lot of research effort has gone into more efficient attention variants that reduce this cost for very long sequences.

## The Bottom Line

Self-attention works by converting every token into a Query, Key, and Value vector, then using those to calculate how much every token should "attend to" every other token in the same sequence — producing a new, context-enriched representation for each one. Layered across multiple heads and many stacked layers, this simple pairwise mechanism is what allows transformers to capture nuanced, long-range relationships in language efficiently and in parallel. It's a remarkably elegant piece of math sitting underneath everything from autocomplete suggestions to full conversational AI — turning "which words matter to which other words" into a calculation a model can actually learn and apply.
