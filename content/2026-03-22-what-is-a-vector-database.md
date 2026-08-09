Title: What is a Vector Database?
Date: 2026-03-22
Category: GenAI
Tags: GenAI, vector-database, RAG, embeddings, retrieval
Slug: what-is-a-vector-database

## Introduction: Databases for the AI Era

Traditional databases are designed for exact matching and range queries. They store rows of structured data and retrieve them based on precise conditions. A SQL query like `SELECT * FROM products WHERE price < 100` is fast and deterministic because the database can use indexes to narrow down the result set efficiently. But when the query is conceptual rather than exact, traditional databases fail. How do you search for documents that are "similar in meaning" to a given sentence? There is no SQL operator for semantic similarity.

Vector databases solve this problem. They are specialized storage systems designed to index and query high-dimensional vectors, the same vectors produced by text embedding models. Instead of matching keywords or filtering by columns, vector databases find the vectors that are closest to a query vector in geometric space. This operation, called approximate nearest neighbor search, is the computational primitive that makes semantic retrieval at scale possible.

## The Core Abstraction: Vectors as Data

In a vector database, the fundamental unit of storage is not a row or a document but a vector. Each vector is associated with metadata, such as the source document, chunk text, timestamp, or category. The database maintains an index structure that organizes these vectors to support fast similarity queries.

When a query arrives, the database receives a vector, searches its index for the closest vectors, and returns the associated metadata. The query itself contains no keywords, no filters, and no boolean logic. It is pure geometry. The semantic meaning of the query has already been encoded into the vector by an embedding model.

## Indexing Algorithms

The challenge of vector search is that comparing a query vector against every stored vector, known as exhaustive search, is prohibitively slow when the dataset contains millions or billions of vectors. Vector databases solve this through approximate nearest neighbor algorithms that trade a small amount of accuracy for massive speedups.

Hierarchical Navigable Small World graphs, or HNSW, is one of the most popular indexing algorithms. It constructs a multi-layer graph where vectors are nodes and edges connect nearby vectors. Search begins at a coarse layer and progressively refines through finer layers, quickly converging on the nearest neighbors without examining the entire dataset.

Other algorithms include Inverted File Index, which partitions the vector space into clusters and searches only the most relevant clusters, and Product Quantization, which compresses vectors into compact codes to reduce memory usage and comparison cost. Different databases implement different algorithms, and the choice affects the speed-accuracy-memory trade-off.

## Popular Vector Database Options

The vector database landscape has exploded with options. Chroma is an open-source, developer-friendly database ideal for prototyping and small-scale applications. It requires no external infrastructure and stores data locally. FAISS, developed by Meta, is a library rather than a standalone database, optimized for in-memory similarity search and popular in research environments.

For production, managed vector databases offer durability, replication, and horizontal scaling. Pinecone is a fully managed service with a simple API and strong performance. Weaviate combines vector search with semantic and keyword search capabilities. Milvus is designed for billion-scale deployments and supports distributed architectures. pgvector extends PostgreSQL with vector capabilities, appealing to teams that want to avoid adding another database to their stack.

## Metadata Filtering and Hybrid Search

Pure vector search is powerful but sometimes insufficient. A user might want to search for documents similar to a query but only within a specific date range or category. Vector databases support metadata filtering, where traditional exact-match constraints are applied before or alongside vector similarity search.

Hybrid search combines vector similarity with keyword matching, using fusion algorithms to rank results from both approaches. This is valuable for queries that contain specific identifiers, such as product codes or legal citations, that embedding models might not handle precisely.

## Operational Considerations

Running a vector database in production requires attention to several factors. Index rebuilds can be expensive; incremental updates are preferable. The choice of distance metric, cosine similarity versus Euclidean distance versus dot product, affects both the semantics of search and the efficiency of the index. Monitoring query latency and recall rate is essential, as index degradation or data growth can silently degrade performance over time.

```python
from langchain.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings

vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=OpenAIEmbeddings(),
    persist_directory="./chroma_db"
)

results = vectorstore.similarity_search("What is RAG?", k=5)
```

## Conclusion

Vector databases are the storage engines of the AI era. They transform the abstract geometry of embeddings into searchable, scalable indexes. By supporting approximate nearest neighbor search, metadata filtering, and hybrid queries, they provide the infrastructure layer that makes Retrieval-Augmented Generation practical at production scale.
