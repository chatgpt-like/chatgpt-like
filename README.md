![CI](../../workflows/CI/badge.svg)
# ChatGPT-like Blazor WebAssembly App

A single–page ChatGPT-style chat UI built with Blazor WebAssembly. All UI state lives in one large component ([src/Sample.Wasm/Pages/Index.razor](src/Sample.Wasm/Pages/Index.razor)). Chat sessions persist locally using IndexedDB with automatic localStorage fallback. The app connects to a locally running OpenAI‑compatible API at http://localhost:4141.

## Features
- ChatGPT-like UX (inline message editing, regeneration, system prompt editing)
- Session management with auto-titling and minimum one-session rule
- Dual persistence: IndexedDB → fallback to localStorage
- Per-model temperature / max token settings retained in localStorage
- Dark / Light theme toggle (stored as THEME_PREFERENCE)
- Mobile-first responsive layout (overlay sidebar, safe-area handling)
- Usage information display (quota snapshot parsing via service layer)
- Clipboard copy and auto-resizing textareas via JS interop ([indexeddb.js](src/Sample.Wasm/wwwroot/js/indexeddb.js))

## Architecture Overview
- Core page/component: [Index.razor](src/Sample.Wasm/Pages/Index.razor)
- JavaScript interop & storage helpers: [indexeddb.js](src/Sample.Wasm/wwwroot/js/indexeddb.js)
- Theming & layout styles: [app.css](src/Sample.Wasm/wwwroot/css/app.css)
- Repository & packaging metadata: [common.props](src/common.props)
- Global formatting & naming rules: [.editorconfig](.editorconfig)
- License: [LICENSE](LICENSE)

### Session Model
In-memory shape (C#):
```
Session { Guid Id, string Title, List<ChatMessage> History }
```
Serialized DTO (saved): Id, Title, History[ { Role, Text } ].

### Persistence Flow
1. SaveSessions() (in [Index.razor](src/Sample.Wasm/Pages/Index.razor)) builds DTO list.
2. Attempts JS interop call saveSessionsToIndexedDB().
3. On failure, JSON stored under key CHAT_SESSIONS in localStorage.
4. On startup: try IndexedDB → fallback localStorage → ensure at least one session.

### Settings Keys
- MODEL_SETTINGS::{modelName} (JSON: Temperature, MaxTokens)
- DEFAULT_MODEL / LAST_MODEL
- SYSTEM_MESSAGE
- THEME_PREFERENCE (light|dark)

### JavaScript Interop (excerpt)
From [indexeddb.js](src/Sample.Wasm/wwwroot/js/indexeddb.js):
- initializeIndexedDB()
- saveSessionsToIndexedDB(sessions)
- loadSessionsFromIndexedDB()
- testIndexedDB()
- clearIndexedDB()
- autoResizeTextarea(id), setupAutoResize(id)
- copyToClipboard(text)

## Build & Run
```powershell
cd "src/Sample.Wasm"
dotnet build
dotnet run
```
Then open the served URL (typically https://localhost:****). Ensure your local AI service (OpenAI-compatible) is running at http://localhost:4141.

## Expected Local API Endpoints
- GET /v1/models  -> model list (string ids)
- POST /v1/chat/completions -> chat completion (messages[] with roles: system|user|assistant)

The injected service interface (Sample.Wasm.Services.IChatService) is consumed in [Index.razor](src/Sample.Wasm/Pages/Index.razor) to:
- GetModelsAsync()
- GetResponseAsync(List<ChatMessage>, model, temperature, maxTokens?)
- GetUsageAsync()

## Usage Tips
- Edit any message by clicking it; system message is always first and never deleted (reset instead).
- Regenerate: edit or use the resend option on a user message—later messages are trimmed back before re-sending.
- Dark Mode: moon/sun toggle persists between sessions.
- Mobile: Sidebar becomes an overlay; height adjusted with JS (index.html viewport script).

## Error Resilience
- Storage failures gracefully fallback.
- Model discovery failure allows manual model id entry.
- IndexedDB test & clear buttons provided in settings overlay.

## Customize
- Adjust CSS variables in [app.css](src/Sample.Wasm/wwwroot/css/app.css)
- Extend JS interop in [indexeddb.js](src/Sample.Wasm/wwwroot/js/indexeddb.js)
- Add service logic under Services/ (e.g., retry, streaming) while preserving existing interface.

## Conventions
- DTO properties PascalCase (Id, Title, History)
- JS receives camelCase transformed structures
- Graceful degradation preferred over throwing for user-facing flows

## Roadmap Ideas
- Streaming token updates
- Export / import sessions (JSON file)
- Multi-tab sync (BroadcastChannel / storage events)
- Offline detection banner

## License
Distributed under the terms of [GPLv3](LICENSE). Third-party icon set under MIT / SIL (see Open Iconic docs in css/open-iconic).

## Acknowledgments
- Blazor WebAssembly
- Open Iconic icon set
- Bootstrap 5.3.3

---
Generated README based on current code structure in [src/Sample.Wasm/Pages/Index.razor](src/Sample.Wasm/Pages/Index.razor) and related assets.