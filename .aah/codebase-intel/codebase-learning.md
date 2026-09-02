# Codebase Learning Document

**Project**: Spoonful — Recipe Management App  
**Profiled**: 2026-09-02  
**Primary Language**: JavaScript (backend) / TypeScript (frontend)  
**Codebase Size**: 23 files, 582 lines  

---

## 1. System Purpose and Context

Spoonful is a bootcamp capstone project (Week 1): a full-stack recipe management application. Recipe creators can sign up, log in, and perform full CRUD on their recipes (with structured ingredients, instructions, and tags). Any visitor can browse and search recipes without logging in. An AI Assistant feature (using Google Gemini) lets any user submit a prompt and receive a streamed text response.

The backend is fully implemented. The React frontend is a skeleton — App.tsx currently just renders "Deloitte React Project". The main work remaining is building out the frontend UI and the AI route.

## 2. Technology Stack

| Category | Technology | Version (if known) | Notes |
|----------|------------|-------------------|-------|
| Language (BE) | JavaScript (CommonJS) | Node.js | require/module.exports pattern |
| Language (FE) | TypeScript | 5.8.3 | Strict mode via tsconfig |
| Framework (BE) | Express | 5.1.0 | Note: Express 5 (not 4 — async error handling differs) |
| Framework (FE) | React | 19.1.1 | Uses StrictMode |
| Build | Vite | 7.1.0 | ESM frontend |
| Database | MongoDB | via Mongoose 8.17 | No explicit version pinned in compose |
| Auth | JWT + bcrypt | jwt 9.0.2, bcrypt 6.0 | 6 salt rounds; 24h token expiry |
| AI | Google Gemini | gemini-3.6-flash | Streaming SSE proxy pattern |
| Containers | Docker + docker-compose | — | Separate Dockerfiles for BE and FE |

## 3. Architecture Overview

Classic MERN stack with a clean MVC backend:

- **Architectural style**: Monolith (two containers: backend API + frontend SPA)
- **Layer structure**:
  - Presentation: React SPA (not yet built)
  - API: Express routes → controller functions
  - Business logic: controllers handle ownership checks, auth, query building
  - Data: Mongoose models with embedded sub-documents
- **Key patterns**:
  - **MVC** on the backend (routes/controllers/models)
  - **JWT Bearer token** auth — token stored client-side, sent as `Authorization: Bearer <token>`
  - **Embedded sub-documents** for ingredients and instructions within a recipe (no separate collections)
  - **AI proxy pattern** — backend forwards to Gemini and pipes SSE stream directly to client; API key never exposed to browser

## 4. Codebase Structure

| Directory | Purpose | Key Files |
|-----------|---------|-----------|
| backend/ | Express REST API | server.js, package.json, Dockerfile |
| backend/routes/ | URL routing | users.js, recipes.js |
| backend/controllers/ | Request handlers | users.js (auth), recipes.js (CRUD + instruction ops) |
| backend/middleware/ | Cross-cutting concerns | verifyToken.js (JWT guard) |
| backend/models/ | Data schemas | user.js (bcrypt pre-save), recipe.js (embedded docs) |
| client/src/ | React app | main.tsx (root mount), App.tsx (skeleton) |

## 5. Data Architecture

- **Primary store**: MongoDB — two collections: `users` and `recipes`
- **Recipe document** embeds ingredients and instructions as sub-document arrays (Mongoose subdoc with `_id`). This avoids joins and is appropriate for the access pattern (always read/write the whole recipe).
- **User document** strips `password` from JSON output via `toJSON.transform` — safe pattern.
- **No migrations**: Mongoose schema changes are applied implicitly on next operation. For a dev project this is fine; in production schema evolution would need care.
- **No seeding mechanism** detected — data must be created via the API.

## 6. API Surface

| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| /api/users/signup | POST | None | Register + return JWT |
| /api/users/login | POST | None | Authenticate + return JWT |
| /api/recipes | GET | None | List/search recipes (title, tag, ingredient query params) |
| /api/recipes/:id | GET | None | Get single recipe |
| /api/recipes | POST | JWT | Create recipe |
| /api/recipes/:id | PUT | JWT + owner | Update recipe |
| /api/recipes/:id | DELETE | JWT + owner | Delete recipe |
| /api/ai/stream | POST | None (planned) | Stream Gemini AI response |

## 7. Key Abstractions

| Concept | Implementation | Files |
|---------|---------------|-------|
| JWT Auth | verifyToken middleware extracts Bearer token, verifies with JWT_SECRET, attaches decoded user to req.user | middleware/verifyToken.js |
| Ownership check | In controllers: `recipe.ownerId.toString() !== req.user._id` — string comparison of ObjectIds | controllers/recipes.js |
| Password security | bcrypt pre-save hook on User model + comparePassword instance method | models/user.js |
| Recipe search | MongoDB query with $regex for title, exact match for tags[], ingredient name lookup via dot-path | controllers/recipes.js:getAll |
| Instruction management | Sub-document ops using Mongoose `.id()` method to find nested docs by their `_id` | controllers/recipes.js:addInstruction/updateInstruction/deleteInstruction |
| AI streaming | Axios with `responseType: 'stream'`, pipe upstream SSE response directly to Express res | routes/ai.js (planned) |

## 8. Dependency Analysis

**High-impact modules** (change carefully):
- `backend/controllers/recipes.js` — 20 symbols, all core CRUD + instruction ops; most routes flow through here
- `backend/models/recipe.js` — schema changes affect all recipe operations
- `backend/middleware/verifyToken.js` — all protected routes depend on this; a bug here breaks auth for the entire app

**Well-isolated modules**:
- `backend/models/user.js` — only used by controllers/users.js; bcrypt logic is fully encapsulated
- `backend/routes/*.js` — thin wiring files; safe to add routes without side effects

**Notable gap**: `addInstruction`, `updateInstruction`, `deleteInstruction` are fully implemented in `controllers/recipes.js` but **not wired to any routes** in `routes/recipes.js`. These controller functions are dead code until routes are added.

## 9. Testing Landscape

- **Test framework**: None configured
- **Test file count**: 0 (no test files exist)
- **Coverage**: None
- **Required by capstone**: at least 4 React component tests

## 10. Build and Deployment

- **Backend**: `node server.js` (no bundler). Dev: `npm start`. Containerized via `backend/Dockerfile` + `backend/docker-compose.dev.yml`.
- **Frontend**: Vite dev server (`npm run dev`). Production build: `tsc -b && vite build`. Containerized via `client/Dockerfile` + `client/docker-compose.yml`.
- **Environment variables**:
  - Backend `.env`: `MONGO_URL`, `JWT_SECRET`, `PORT` (optional, defaults to 3000), `GEMINI_API_KEY` (planned)
  - Frontend `.env`: `VITE_BACKEND_URL` (defaults to `http://localhost:3000`)
- **No CI/CD** configured (stretch goal per README)

## 11. Technical Debt and Risks

| Area | Observation | Impact | Confidence |
|------|-------------|--------|------------|
| Instruction routes missing | addInstruction/updateInstruction/deleteInstruction controllers exist but no routes wire them | Feature gap — instruction management via API impossible | High |
| Frontend is a skeleton | App.tsx renders only a heading; no pages, no routing, no API calls | Entire frontend needs to be built | High |
| No input sanitization | Recipe creation accepts raw req.body spread into Recipe.create | Low risk in controlled bootcamp context; would need attention in production | Medium |
| JWT_SECRET env var | If not set, `process.env.JWT_SECRET` is undefined — tokens still sign but verification would also use undefined, making them trivially forgeable | Must set JWT_SECRET in .env before any real use | High |
| No error normalization | Each controller has its own error response shape | Inconsistent API error format for clients to handle | Low |
| Test coverage 0% | No tests exist in a project that requires 4 component tests | Must write tests before submission | High |

## 12. Recommendations for Modification

- **Start with environment setup**: Create `backend/.env` with `MONGO_URL`, `JWT_SECRET`, and `PORT` before running the backend.
- **Add missing instruction routes**: Wire `/api/recipes/:id/instructions` routes to the existing controller functions.
- **Build frontend in this order**: Install axios + react-router → add routing skeleton → implement Login/Signup → implement Browse Recipes → implement Dashboard (CRUD) → implement AI Assistant.
- **Be careful with**: `controllers/recipes.js` — it's the most complex file; the ownership check pattern (`ownerId.toString() !== req.user._id`) must be preserved in any modification.
- **Test early**: Run `curl -X POST http://localhost:3000/api/users/signup -H "Content-Type: application/json" -d '{"email":"test@test.com","password":"password"}'` to verify the backend before building the frontend.
- **AI route**: The full implementation is documented in README.md — copy it verbatim and register `app.use('/api/ai', require('./routes/ai'))` in server.js.
