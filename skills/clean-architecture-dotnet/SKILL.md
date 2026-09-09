# .NET Clean Architecture Standard

## 1. Domain Layer
- STRICTLY NO external dependencies (No Entity Framework, No ASP.NET Core, No 3rd-party SDKs).
- MUST only contain Entities, Value Objects, Domain Events, and Core Interfaces (e.g., Repositories).

## 2. Application Layer
- Define all Use Cases utilizing the CQRS Pattern (via MediatR).
- Interact with the infrastructure strictly through Interfaces. DO NOT include any direct database access or API calling logic here.

## 3. Infrastructure Layer
- Implement Interfaces defined in the Application/Domain layers (e.g., EF Core DbContext, Qdrant Client)[cite: 1].
- Handle all 3rd-party integrations and external services (OpenAI/Anthropic SDKs)[cite: 1].

## 4. API Layer (Presentation)
- Acts ONLY as the entry point to receive HTTP requests, validate JWT authentication, and dispatch commands/queries to the Application Layer[cite: 1].
- MUST utilize Server-Sent Events (SSE) or WebSockets for chat stream functionalities[cite: 1].