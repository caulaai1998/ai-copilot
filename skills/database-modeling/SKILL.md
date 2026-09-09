# Database Schema & Entity Framework Standard

## 1. Storage Separation
- Relational DB (PostgreSQL/SQL Server): Store metadata only (Users, Tenants, Documents, ChatMessages)[cite: 1].
- Vector DB (Qdrant/pgvector): Store float arrays (vectors) exclusively. DO NOT store raw vectors in the Relational DB[cite: 1]. Link them using `VectorId`[cite: 1].

## 2. Multi-Tenant Concept
- All business entities (`Documents`, `ChatSessions`, `Users`) MUST include a `TenantId` column[cite: 1].
- Entity Framework Core MUST be configured with Global Query Filters on `TenantId` to prevent cross-tenant data leakage.

## 3. Tracking & Observability
- MUST log all LLM usage (PromptTokens, CompletionTokens, CostUsd) in the `LlmUsageLogs` table to monitor AI operational costs[cite: 1].