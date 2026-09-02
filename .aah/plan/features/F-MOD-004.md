## Id
F-MOD-004

## Title
AI Assistant — Gemini Streaming Text Analyzer

## Module Ref
MOD-004

## Description
This module delivers the `/ai-assistant` route — a guest-accessible AI text analysis page — and the backend SSE proxy endpoint that powers it. The stack is React 19 + TypeScript + Vite (frontend) and Express 5 + Node.js (backend), matching the project-wide choices established in MOD-000.

**API layer — POST /api/ai/stream**

The Express route accepts `{ "prompt": "..." }` in the request body, prepends no additional system prompt (the frontend constructs the prefixed prompt), and pipes the Gemini `gemini-2.0-flash` SSE response directly back to the client as a streaming HTTP response. `GEMINI_API_KEY` is read exclusively server-side; it must never appear in any client-side bundle or response body (resolved-standards.yaml rule EXTRACTED-005, TS-SEC-001). The frontend calls `POST /api/ai/stream` through the project-wide axios instance at `localhost:3000` but consumes the SSE response via the Fetch API with `getReader` — not axios — since axios buffers the full response.

**UI/UX layer — AIAssistantPage (`/ai-assistant`)**

The route requires no authentication and is linked from the persistent Navbar (nav item "AI Assistant", `access: all`, per `.aah/architecture/design-spec.yaml` navigation contract). The page layout, visual states, and composition are specified in `.aah/architecture/applications_wireframes/07-ai-assistant.html` and must match that wireframe region by region.

Implement the "Modern Pantry" direction defined in `.aah/architecture/design-spec.yaml` (`direction.label: "Modern Pantry"`, `direction.comp: design/direction.html`) and the full token set it establishes. The design system guidance is a reference doc: `.aah/architecture/design-spec.yaml` → `guidance.ref: frontend_guidance.md`. Use only the resolved palette, type, spacing, and radius tokens from `design-spec.yaml` — no ad hoc hex values or pixel overrides. Per `install: none`, no additional design-system package installation is required.

**Regions (per `.aah/architecture/applications_wireframes/07-ai-assistant.html`)**

- **[A] Navbar** — persistent top-nav (`primary-nav`), "AI Assistant" link marked active; shows guest or creator state depending on JWT presence in localStorage.
- **[B] Page Header** — static title ("AI Assistant") and one-line subtitle ("Paste any text and choose an analysis — Summarize, Key Points, or Tone Classification.").
- **[C★] Input + Action Buttons** — dominant interactive region: a resizable `<textarea>` for user text input with a character count display, and three action buttons ("Summarize" / "Key Points" / "Classify Tone") arranged in a 3-column grid. Each button constructs a mode-specific prefixed prompt and fires `POST /api/ai/stream`. States: `idle`, `empty-validation-error`, `loading-s2-streaming-indicator`, `streaming`, `complete`, `error`.
- **[D] Streaming Output Pane** — a white output box that renders tokens progressively as they arrive from the SSE `ReadableStream` (consumed via `fetch` + `getReader` + `read` loop, not EventSource). Displays animated three-dot loading indicator (S2 pattern, per wireframe) while waiting for the first token. A blinking inline cursor tracks the end of in-progress text. States: `idle-placeholder`, `s2-loading-dots`, `streaming-with-cursor`, `complete`, `error`.
- **[E] Session History** — a React state list (not persisted, resets on reload) holding at most 3 prompt/response pairs. New completions are prepended; when the list reaches 3, the oldest pair is evicted (FIFO). Each history item renders the action label, truncated response text, and relative time metadata. States: `empty`, `has-1`, `has-2`, `has-3`.

The AIAssistantPage must have its own `.css` file (vanilla CSS + Flexbox, desktop-first, with breakpoints at 1024 / 768 / 480 px per project convention). Business logic (SSE consumption, history management) belongs in a custom hook or service module, not inline in the component (resolved-standards.yaml rule RX-ARCH-002).

Grounding user story: "As any user, I want to navigate to the AI Assistant page, paste some text, and choose an analysis (Summarize / Key Points / Tone), so I can get instant AI-powered feedback streamed progressively." (discuss-prd.md User Story 9).

**Behavioral expectations**

- Given `GEMINI_API_KEY` is absent from the environment, when the backend starts, then `env-check.cjs` reports `ERR_CDR_78_EX_CONFIG` naming `GEMINI_API_KEY` and the process exits before any module reads configuration; the error message points the user at `cp .env.example .env`. (MOD-000 registers this variable in the checker; this module's change must confirm the variable appears in `.env.example` with a safe placeholder value.)
- Given `GEMINI_API_KEY` is present, when `POST /api/ai/stream` receives `{ "prompt": "Summarize: <text>" }`, then the backend proxies the request to Gemini `gemini-2.0-flash` and streams SSE token chunks back in the HTTP response; the key value is never included in the response body or any client-visible artifact.
- Given any user (guest or authenticated), when they navigate to `/ai-assistant`, then the page renders without an auth redirect and the Navbar shows "AI Assistant" as the active link.
- Given the `/ai-assistant` page renders with no prior interaction, when region [B] displays, then the page title reads "AI Assistant" and the subtitle reads "Paste any text and choose an analysis — Summarize, Key Points, or Tone Classification."
- Given the textarea is empty, when any of the three action buttons is clicked, then a client-side validation error message ("Please enter some text before analyzing") appears in region [C★] and no HTTP request to `POST /api/ai/stream` is made.
- Given text is in the textarea, when "Summarize" is clicked, then a prompt prefixed with a summarize instruction is POSTed to `/api/ai/stream`, the three action buttons enter a loading/disabled state, and region [D] transitions to `s2-loading-dots` showing the animated three-dot indicator and "Analyzing…" label.
- Given text is in the textarea, when "Key Points" is clicked, then a prompt prefixed with a key-points instruction is POSTed to `/api/ai/stream` and the same loading sequence begins.
- Given text is in the textarea, when "Classify Tone" is clicked, then a prompt prefixed with a tone-classification instruction is POSTed to `/api/ai/stream` and the same loading sequence begins.
- Given the SSE `ReadableStream` yields its first token, when the `read` loop receives the chunk, then region [D] replaces the loading-dots indicator with the partial token text and a blinking inline cursor appended at the end.
- Given subsequent tokens arrive from the stream, when each chunk is read, then the text in region [D] grows incrementally token-by-token with the cursor tracking the new end position.
- Given the stream closes (no further chunks), when the `read` loop detects `done === true`, then the blinking cursor is removed and region [D] shows the final complete response text; the action buttons return to the interactive state.
- Given the SSE fetch or Gemini proxy returns an error, when region [D] renders the error state, then it displays "Something went wrong. The AI service could not process your request. Please try again." and the action buttons return to the interactive state.
- Given a streaming request completes successfully, when the response is finished, then the {action label, response text} pair is prepended to the React-state session history list in region [E].
- Given the session history list already contains 3 pairs, when a new request completes, then the oldest pair is removed (FIFO eviction) so the list remains at exactly 3 entries.
- Given 4 or more requests are submitted in sequence, when region [E] renders, then it shows at most 3 history items and never more; the history is not written to any backend endpoint or browser storage.
- Given the "Modern Pantry" design system tokens in `.aah/architecture/design-spec.yaml` and `direction.rules`, when each required state (`idle`, `empty-validation-error`, `loading-s2-streaming-indicator`, `streaming`, `complete`, `error`) of screen `07-ai-assistant` renders, then all colors, font weights, letter-spacing, border-radius, and spacing values are drawn exclusively from the resolved token set (primary `#3A7D5E`, bg `#F5F5F2`, surface `#FFFFFF`, muted `#6B6B6B`, border `#E0E0DC`, accent `#F2C94C`, pill radius `100px`, card radius `8px`, display weight 800 at `−0.03em`, label transform uppercase at `0.08em`) — no ad hoc hex values or pixel overrides outside those tokens appear in the component stylesheet.

## Layers
- api
- ui/ux

## Dependencies
- F-MOD-000

## Required Env Variables
- GEMINI_API_KEY — Gemini API key read by the backend AI proxy endpoint (validated at startup)

## Lint Config

## Test Config

## Constraints

## Applicable Standards
- Total rules: 21
- Critical:
  - TS-SEC-001
  - TS-SEC-002
  - RX-SEC-001
  - RX-SEC-002
- High:
  - EXTRACTED-002
  - EXTRACTED-003
  - EXTRACTED-004
  - EXTRACTED-005
  - EXTRACTED-006
  - TS-TYPE-001
  - TS-TYPE-002
  - TS-TEST-001
  - TS-ERR-001
  - RX-ARCH-001
  - RX-ARCH-002
  - RX-A11Y-001
- Medium:
  - EXTRACTED-001
  - TS-TYPE-003
  - RX-A11Y-002
  - RX-PERF-001
- Low:
  - TS-CONV-001

## Status
planned
