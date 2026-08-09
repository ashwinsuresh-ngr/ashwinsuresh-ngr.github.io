Title: Improving RAG Retrieval
Date: 2026-03-26
Category: GenAI
Tags: GenAI, RAG, retrieval, reranking, embeddings
Slug: improving-rag-retrieval

## Introduction: The Retrieval Bottleneck

The generation phase of RAG receives most of the attention. Developers obsess over prompt engineering, model selection, and temperature settings. But generation quality is fundamentally bounded by retrieval quality. If the retrieved chunks do not contain the answer, no amount of prompt optimization will help the model hallucinate correctly. The retrieval phase is the bottleneck, and improving it yields disproportionate returns.

Retrieval improvement is a multifaceted problem. It involves chunking strategy, embedding model selection, query preprocessing, retrieval algorithms, and post-retrieval reranking. Each of these levers can be tuned independently, and their interactions are complex. A better embedding model might be wasted on poorly chunked documents. A perfect retrieval might be undermined by a query that uses terminology different from the indexed documents.

## Query Rewriting and Expansion

One of the most effective retrieval improvements is query preprocessing. Users often ask questions using terminology that differs from the documents. A user might ask "How do I fix a flat tire?" while the manual refers to "puncture repair procedures." Without bridging this vocabulary gap, retrieval fails.

Query rewriting uses an LLM to reformulate the user's question into multiple variants that cover different phrasings, synonyms, and related concepts. Each variant is used to retrieve chunks, and the results are merged. Query expansion adds related terms to the query before embedding it, pushing the query vector into regions of the embedding space that contain relevant documents even if the exact words do not match.

## Hybrid Search

Pure vector search excels at semantic similarity but struggles with exact matches for specific identifiers, dates, or technical codes. Hybrid search combines vector similarity with traditional keyword search, typically using a fusion algorithm to merge the ranked lists from both approaches.

The keyword component ensures that documents containing exact terms are not missed, while the vector component ensures that conceptually related documents are found even without keyword overlap. LangChain supports hybrid search through integrations with databases like Weaviate and Elasticsearch, which can execute both search types in a single query.

## Reranking with Cross-Encoders

Embedding models used for initial retrieval are typically bi-encoders: they embed the query and documents independently, then compare the vectors. This is fast but coarse, because the query and document never interact during encoding. Cross-encoders, by contrast, process the query and document together, allowing fine-grained attention-based comparison.

Cross-encoders are too slow for initial retrieval across millions of documents, but they are perfect for reranking a small candidate set. The pipeline retrieves one hundred candidates quickly using the bi-encoder, then the cross-encoder scores these candidates precisely and returns the top five. This retrieve-then-rerank pattern is the state of the art for high-precision RAG.

## Parent Document Retrieval

A persistent tension in RAG is between chunk size and context. Small chunks embed precisely but lack surrounding context. Large chunks provide context but embed poorly. Parent document retrieval offers a compromise. Documents are chunked into small, precise pieces for embedding and retrieval. But each small chunk stores a reference to its parent, the larger document or section from which it came.

When a small chunk is retrieved, the system replaces it with its parent document for injection into the prompt. The retrieval benefits from the precision of small chunks, while the generation benefits from the completeness of the parent context.

## Metadata Enrichment

Retrieval can be improved by enriching chunks with contextual metadata. A chunk from a research paper might be prepended with its section title and the paper's abstract. A chunk from a support ticket might include the product version and issue category. This metadata, embedded along with the chunk text, improves the semantic signal and enables more precise filtering.

## Iterative Evaluation

Improvements to retrieval must be measured. Build an evaluation dataset of representative queries with annotated relevant passages. For each change, measure recall at various cutoffs, mean reciprocal rank, and end-to-end answer accuracy. Without this measurement loop, optimization becomes guesswork.

## Conclusion

RAG retrieval is not a solved problem; it is an optimization target. Query rewriting, hybrid search, reranking, parent document retrieval, and metadata enrichment each offer significant gains. The key is measuring impact systematically and understanding that retrieval and generation are coupled: improving the former is often the fastest way to improve the latter.
