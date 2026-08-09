Title: Choosing the Right Chunk Size
Date: 2026-03-19
Category: GenAI
Tags: GenAI, RAG, chunking, embeddings, retrieval
Slug: choosing-the-right-chunk-size

## Introduction: The Chunking Dilemma

In a Retrieval-Augmented Generation system, documents must be divided into pieces before they can be embedded and stored. This division, called chunking, is one of the most consequential design decisions in the entire pipeline. Choose chunks that are too small, and you lose semantic coherence. The meaning of a sentence depends on the paragraph that surrounds it, and a fragment may become ambiguous or useless when isolated. Choose chunks that are too large, and you dilute the embedding's precision. A chunk that contains ten unrelated topics will produce an embedding that is semantically blurry, matching poorly against focused queries.

Chunk size is not a single number to be optimized in isolation. It interacts with the embedding model's context window, the LLM's context limit, the nature of the source documents, and the expected query patterns. A legal contract requires different chunking than a codebase, which requires different chunking than a collection of tweets.

## How Chunk Size Affects Embeddings

Embedding models convert text into dense vectors by capturing semantic meaning. When a chunk is small and focused, the embedding accurately represents that specific concept. When a chunk is large and diffuse, the embedding becomes an average of many concepts, losing the sharpness that makes similarity search effective.

Consider a fifty-page technical manual. If chunked into paragraphs of one hundred tokens, each chunk likely covers a single procedure or definition. A query about that procedure will retrieve a highly relevant chunk. If chunked into ten-thousand-token sections, each chunk covers dozens of topics. The embedding for such a chunk is semantically close to many different queries, but not strongly aligned with any particular one. The retrieval precision drops.

## How Chunk Size Affects Generation

Chunk size also constrains the generation phase. Retrieved chunks must fit into the LLM's context window alongside the system prompt and the user's query. If chunks are too large, fewer can be included in the prompt. You might retrieve five chunks, but only have room for two. This reduces the diversity of context available to the model and increases the risk that critical information was in the excluded chunks.

Conversely, if chunks are too small, the model receives many fragments without sufficient surrounding context. A single sentence like "Set the threshold to zero" is meaningless without knowing what system is being configured and why. The model may hallucinate context or fail to answer accurately.

## Document-Specific Considerations

Different document types have natural semantic boundaries that should guide chunk size. Source code chunks well at the function or class level, because functions are self-contained units of logic. Legal documents chunk well at the clause or section level. Academic papers chunk well at the paragraph level, with special handling for section headers that provide topical context. Conversational data, such as support tickets, might chunk at the message or thread level.

The chunking strategy should preserve these natural boundaries rather than splitting arbitrarily at fixed token counts. A chunk that cuts off mid-sentence or mid-theorem is less useful than one that respects the document's structure.

## The Trade-Off Matrix

| Chunk Size | Retrieval Precision | Context Completeness | Context Window Efficiency |
|-----------|-------------------|---------------------|--------------------------|
| Small (100-300 tokens) | High | Low | Many chunks fit |
| Medium (500-1000 tokens) | Balanced | Balanced | Moderate |
| Large (1500+ tokens) | Low | High | Few chunks fit |

## Empirical Tuning

There is no universal optimal chunk size. The right size must be determined empirically for each application. The standard approach is to build an evaluation dataset of representative queries with known relevant passages. Then, chunk the documents at various sizes, index them, and measure retrieval accuracy. The size that maximizes the proportion of queries for which the relevant passage appears in the top retrieved results is the winner.

This evaluation should also consider the downstream generation quality. A chunk size that improves retrieval precision but provides insufficient context for accurate answers is not truly optimal. End-to-end evaluation, measuring whether the final generated answer is correct, is the ultimate metric.

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    separators=["\n\n", "\n", ".", " "]
)
chunks = splitter.split_documents(documents)
```

## Conclusion

Chunk size is a lever that directly controls the quality of both retrieval and generation in RAG systems. Too small, and context is lost. Too large, and precision degrades. The optimal size depends on document structure, query patterns, and model constraints, and it must be validated through empirical evaluation rather than guessed.
