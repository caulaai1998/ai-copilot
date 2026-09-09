# RAG Query Engine & Ingestion Standard

## 1. Chunking Strategy (Ingestion)
- Chunk text into segments of ~300-500 tokens[cite: 1].
- MUST include an overlap of ~50 tokens between consecutive chunks to preserve semantic context[cite: 1].
- Use `text-embedding-3-small` (OpenAI) or `all-MiniLM` for vector embeddings[cite: 1].

## 2. Retrieval Strategy
- Retrieve the Top-K closest chunks (K=5) using cosine similarity via Qdrant[cite: 1].
- SECURITY: MUST filter search results by `tenantId` and document access permissions[cite: 1].

## 3. Prompt & Response Management
- Prompt Construction: System prompt + Context (retrieved chunks) + Chat history + Current user query[cite: 1].
- The API response MUST include a citations/sources array indicating exactly which chunks or documents were used by the LLM[cite: 1].