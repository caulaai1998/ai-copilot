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