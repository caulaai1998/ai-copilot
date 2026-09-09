# Strict Type Safety & No Silent Fallbacks

## 1. Fail-Fast Mechanism
- When mapping data from 3rd-party SDKs (OpenAI, Qdrant), if a required field is missing or invalid, throw an Exception immediately. DO NOT assign implicit `null` or empty string values.
- All external exceptions must be caught and transformed into standard internal error codes.

## 2. Nullable Reference Types (C#)
- `<Nullable>enable</Nullable>` MUST be enabled across all `.csproj` files.
- Explicitly declare nullable variables (e.g., `string?`). Variables without `?` must be strictly initialized and never hold a null value.