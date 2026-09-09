# React + Redux Toolkit Architecture Standard

## 1. Core Technology
- STRICTLY use React 18, TypeScript (Strict Mode), and TailwindCSS[cite: 1].
- State Management: Use Redux Toolkit for global state (chat history, user session, document processing status)[cite: 1].

## 2. API Communication & Streaming
- REST APIs: Use Axios or Fetch API with Interceptors to attach JWT tokens.
- Chat Streaming: MUST use Server-Sent Events (SSE) or WebSockets to consume real-time AI responses. DO NOT use polling or wait for the full response[cite: 1].

## 3. Directory Structure (Feature-Sliced Design)
- Group by features: `src/features/chat`, `src/features/documents`, `src/features/auth`[cite: 1].
- Strictly separate UI Components (dumb/presentational) from Logic/Hooks (smart/container).