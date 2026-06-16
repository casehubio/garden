---
id: PP-20260616-896634
title: "Use io.qdrant:client directly for Qdrant — not quarkus-langchain4j-qdrant"
type: rule
scope: universal
applies_to: "Any Java project storing or querying embeddings in Qdrant"
severity: guidance
refs:
  - docs/protocols/universal/filesystem-watching-library.md
violation_hint: "Use of dev.langchain4j.store.embedding.EmbeddingStore<TextSegment> or quarkus-langchain4j-qdrant for Qdrant access"
created: 2026-06-16
---

Use `io.qdrant:client` (the official Qdrant Java gRPC client) for all Qdrant operations.

```xml
<dependency>
    <groupId>io.qdrant</groupId>
    <artifactId>client</artifactId>
</dependency>
```

**Why not `quarkus-langchain4j-qdrant`:**

LangChain4j's `EmbeddingStore<TextSegment>` abstraction hides Qdrant's capabilities behind a lowest-common-denominator interface:

- **No named vectors** — cannot store dense + sparse embeddings in the same point (required for hybrid search with RRF fusion)
- **No payload-scoped deletes** — `deleteDocument(sourceDocumentId)` requires filtering by payload field; `EmbeddingStore.remove(id)` only works with the point UUID
- **No scroll pagination** — `listDocuments()` requires iterating all points with a cursor; `EmbeddingStore` has no equivalent
- **No collection creation control** — cannot configure vector dimensions, distance metric, sparse vector params, or tenancy isolation strategy
- **No tenancy filtering** — cannot scope queries or writes to a tenant via payload filters
- **Random UUIDs only** — `addAll()` generates random point IDs; no support for deterministic IDs needed for idempotent upsert

The raw client provides all of these directly via the Qdrant gRPC API.

**What to keep from LangChain4j:** Use `dev.langchain4j:langchain4j` for `EmbeddingModel` (embedding generation via Ollama/OpenAI), `TextSegment`, `Document`, and `DocumentSplitter`. The embedding *model* abstraction is useful; the embedding *store* abstraction is not.

**Reference implementation:** `casehub-neural-text` `rag` module — `QdrantEmbeddingIngestor` and `HybridCaseRetriever` use `io.qdrant:client` for all Qdrant access while using LangChain4j's `EmbeddingModel` for embedding generation.
