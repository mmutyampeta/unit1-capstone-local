# System Architecture

## Overview
Spoonful is a full-stack recipe management web application using a MERN (MongoDB, Express, React, Node.js) architecture. The Express backend provides a REST API with JWT authentication, while the React/TypeScript frontend (currently a skeleton) consumes that API to let users browse, create, and manage recipes, plus an AI assistant powered by Google Gemini.

## Architecture Diagram

```mermaid
graph TB
    subgraph "External Systems"
        MONGO[(MongoDB Atlas<br/>or local)]
        GEMINI[Google Gemini API<br/>Streaming SSE]
    end

    subgraph "Client Browser"
        REACT[React 19 + TypeScript<br/>Vite Dev Server :5173]
    end

    subgraph "Backend - Node.js/Express :3000"
        SERVER[server.js<br/>Entry Point]

        subgraph "Middleware"
            MW[verifyToken.js<br/>JWT Auth Guard]
        end

        subgraph "Routes"
            RU[routes/users.js<br/>/api/users]
            RR[routes/recipes.js<br/>/api/recipes]
            RA[routes/ai.js<br/>/api/ai - PLANNED]
        end

        subgraph "Controllers"
            CU[controllers/users.js<br/>signup, login]
            CR[controllers/recipes.js<br/>CRUD + instructions]
            CA[controllers/ai.js<br/>Gemini proxy - PLANNED]
        end

        subgraph "Models"
            MU[models/user.js<br/>Mongoose Schema]
            MR[models/recipe.js<br/>Mongoose Schema]
        end
    end

    REACT -->|REST/JSON| SERVER
    SERVER --> RU & RR & RA
    RU --> CU
    RR --> MW --> CR
    RA --> CA
    CU --> MU
    CR --> MR
    MU & MR --> MONGO
    CA -->|SSE stream| GEMINI
```

## Layer Summary

| Layer | Purpose | Key Components |
|-------|---------|----------------|
| Presentation | React SPA — UI, routing, state | App.tsx, pages (planned), axios calls |
| Application | Express route wiring + middleware | server.js, routes/*.js, verifyToken.js |
| Business Logic | Request handling, auth, ownership checks | controllers/users.js, controllers/recipes.js |
| Data | Mongoose ODM, schema definitions | models/user.js, models/recipe.js |
| Infrastructure | Docker containers, env config | Dockerfile (x2), docker-compose files |

## External Integrations

- **MongoDB**: primary data store, connected via `MONGO_URL` env var using Mongoose
- **Google Gemini API**: streaming text generation for AI Assistant feature (PLANNED); proxied through backend; key in `GEMINI_API_KEY` env var
- **JWT**: stateless auth tokens signed with `JWT_SECRET`, 24h expiry
