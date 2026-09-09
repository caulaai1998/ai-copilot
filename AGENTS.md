# AGENTS.md

Clean architecture solution for `RagCopilot`, an AI/RAG copilot service. .NET 8.

## Projects (dependency order)

- `RagCopilot.Domain` — no dependencies (entities, interfaces)
- `RagCopilot.Application` — references Domain
- `RagCopilot.Infrastructure` — references Application; owns `Qdrant.Client` (vector DB) package
- `RagCopilot.Api` — ASP.NET Core minimal API; references Application + Infrastructure
- `RagCopilot.Worker` — `Microsoft.NET.Sdk.Worker` BackgroundService; references Application only

Follow this layering for new code: put interfaces in Domain, use-cases in Application, implementations in Infrastructure.

## Commands

```bash
dotnet build RagCopilot.sln          # build everything
dotnet run --project RagCopilot.Api  # API at http://localhost:5086 (Swagger at /swagger)
dotnet run --project RagCopilot.Worker
```

## Known state / gotchas

- **The API does not build as-is.** `RagCopilot.Api/Program.cs` references `AppDbContext`, `UseSqlServer`/`AddDbContext`, and `QdrantClient`, but `RagCopilot.Infrastructure` only has a stub `Class1.cs` — none of these types exist yet. Adding them (EF Core + Qdrant setup) is expected next work. Also note `app.UseCors(...)` is called before `var app = builder.Build();` — reorder if fixing.
- Domain/Application/Infrastructure each contain a placeholder `Class1.cs`.
- Expected external services (from `RagCopilot.Api/appsettings.json`): SQL Server (`DefaultConnection`), Qdrant (`QdrantUrl` = `http://localhost:6333`), Redis (`localhost:6379`, key `Redis:Configuration`). These aren't wired into DI yet.
- CORS policy `AllowReactApp` allows `http://localhost:3000`.
- No tests, no CI, no linter/formatter configured.

# STRICT PROHIBITIONS (CORE RULES)

1. **No Raw 3rd-Party Errors:** DO NOT return raw error URLs or messages from external services (Qdrant, OpenAI, Azure). All 3rd-party errors MUST be wrapped through an Anti-Corruption Layer (ACL) and translated into friendly internal Domain Exceptions.
2. **No Variable Alias & Casing Mapping:** Strictly read the exact key defined in the Interface/Contract/Payload. DO NOT use multiple variable aliases or fallback between different casing formats.
3. **No Implicit Default Fallbacks (Data Corruption Prevention):**
   - When input data is missing (`undefined` / `null` / missing field), STRICTLY DO NOT assign implicit default values using `||` or `??`.
   - Standard handling: If a required field is missing -> MUST throw a `ValidationError` or return a `400 Bad Request`. If an optional field is missing, preserve it as `null` / `undefined`.
4. **No Secrets Manager Fallback:**
   - Secrets and API Keys MUST be exclusively retrieved from the Secrets Manager (AWS Secrets Manager / Azure Key Vault). This is the Single Source of Truth.
   - STRICTLY PROHIBITED to write fallback logic to environment variables (`process.env`) or static constants if secret retrieval fails. MUST Fail-Fast (throw an explicit Error) and halt execution immediately.

   # CODE REVIEW & DESIGN PRINCIPLES (OOP & SOLID FIRST)

1. **Object-Oriented Programming (OOP) First:**
   - **Encapsulation:** Strongly enforce encapsulation. Business logic MUST reside inside Domain Entities/Value Objects, not scattered across Anemic Domain Models.
   - **Abstraction:** Hide implementation details behind interfaces or abstract classes. Expose only what is necessary.
   - **Inheritance vs Composition:** Favor Composition over Inheritance to avoid brittle class hierarchies.

2. **Strict SOLID Principles:**
   - **S (Single Responsibility):** Each class/service MUST have only one reason to change. Tightly separate API controllers, CQRS handlers, and infrastructure clients.
   - **O (Open/Closed):** Code must be open for extension but closed for modification. Use Strategy, Factory, or Decorator patterns when handling multiple providers (e.g., LLM Providers: OpenAI, Anthropic).
   - **L (Liskov Substitution):** Derived classes or interface implementations must be fully substitutable for their base types without altering system correctness.
   - **I (Interface Segregation):** Avoid fat interfaces. Split them into small, purpose-specific interfaces (e.g., `IReadRepository`, `IWriteRepository`).
   - **D (Dependency Inversion):** High-level modules (Domain/Application) MUST NOT depend on low-level modules (Infrastructure). Both MUST depend on Abstractions (Interfaces).

3. **Code Review Execution Rule:**
   - When reviewing or writing code, AI MUST proactively evaluate the code against OOP and SOLID principles.
   - If any violation (e.g., tight coupling, static dependencies, God classes, anemic domain models) is detected, AI MUST explicitly point it out and offer a refactored solution adhering to SOLID.

   # FRONTEND CODE REVIEW STANDARDS (REACT + TYPESCRIPT)

1. **Strict Feature-Sliced Architecture:**
   - Code MUST be organized by feature in `src/features/{feature-name}` (e.g., `chat`, `documents`, `auth`).
   - Tightly separate UI components (Presentational/Dumb) from business logic (Custom Hooks/Redux Slices/Containers). Components should only focus on rendering.

2. **Absolute Type-Safety & Contract Integrity:**
   - NEVER use `any` or loose type assertions (`as unknown`). Every API response, state, and component prop MUST have explicit TypeScript interfaces.
   - API DTOs in Frontend MUST strictly mirror Backend response contracts.

3. **Performance & Memory Management:**
   - **Stream Cleanup:** EventSource (SSE) or WebSocket connections MUST be properly closed in `useEffect` cleanup functions to avoid memory leaks.
   - **Render Optimization:** Prevent unnecessary re-renders using `useCallback`, `useMemo`, or React Compiler patterns where heavy calculations or complex object props are involved.
   - Avoid creating inline functions or objects inside jsx loops or heavily updated components.

4. **State Management Discipline (Redux Toolkit):**
   - Keep global Redux state minimal (global auth, active chat session status, document processing state). Local state (inputs, toggles) MUST remain inside `useState`.
   - Async calls MUST be handled via `createAsyncThunk` or RTK Query with standardized `loading`, `error`, and `data` states.

5. **Streaming UX Resilience:**
   - Chat UI components MUST support incremental text append (typing effect) without breaking scroll position or throwing UI layout shifts during active SSE streaming.
   - MUST handle network dropouts gracefully during stream responses with user-friendly retry states.