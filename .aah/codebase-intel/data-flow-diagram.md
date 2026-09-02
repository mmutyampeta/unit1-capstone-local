# Data Flow

## Overview
Three primary data flows: (1) auth — signup/login issuing JWT tokens; (2) recipe CRUD — authenticated write paths and public read paths; (3) AI streaming — client prompt forwarded to Gemini, SSE stream piped back.

## Data Flow Diagram

```mermaid
flowchart LR
    subgraph "Client"
        UI[React UI]
    end

    subgraph "Backend :3000"
        SERVER[Express server.js]
        AUTH_MW[verifyToken middleware]
        UC[users controller]
        RC[recipes controller]
        AC[ai controller<br/>PLANNED]
    end

    subgraph "Data Stores"
        MONGO[(MongoDB)]
    end

    GEMINI[Google Gemini<br/>SSE API]

    UI -->|"POST /api/users/signup<br/>POST /api/users/login"| UC
    UC -->|"find/create User"| MONGO
    UC -->|"JWT token"| UI

    UI -->|"GET /api/recipes"| RC
    UI -->|"POST/PUT/DELETE /api/recipes<br/>Authorization: Bearer token"| AUTH_MW
    AUTH_MW -->|"decoded user on req.user"| RC
    RC -->|"find/create/update/delete Recipe"| MONGO
    MONGO -->|"Recipe docs"| RC
    RC -->|"JSON response"| UI

    UI -->|"POST /api/ai/stream<br/>{ prompt }"| AC
    AC -->|"SSE streamGenerateContent"| GEMINI
    GEMINI -->|"token-by-token SSE"| AC
    AC -->|"piped SSE stream"| UI
```

## Data Pipelines

| Pipeline | Source | Processing | Destination |
|----------|--------|-----------|-------------|
| Auth signup | Client POST /api/users/signup | bcrypt hash password, create User, sign JWT | Client receives `{ token }` |
| Auth login | Client POST /api/users/login | Lookup user by email, comparePassword, sign JWT | Client receives `{ token }` |
| Recipe read (public) | Client GET /api/recipes | Optional query filter (title/tag/ingredient), MongoDB find | JSON array of recipes |
| Recipe write | Client POST/PUT/DELETE with JWT | verifyToken → decode user → CRUD with ownership check | Updated recipe doc or confirmation |
| AI streaming | Client POST /api/ai/stream with prompt | Validate prompt, proxy to Gemini SSE endpoint | SSE stream piped directly to client |

## Integration Points

- **MongoDB** (`MONGO_URL` env): inbound — all persistent read/write for users and recipes
- **Google Gemini** (`GEMINI_API_KEY` env): outbound streaming — AI assistant feature; backend is proxy, key never reaches client
- **JWT** (`JWT_SECRET` env): inline — token issued at login/signup, verified on protected routes
