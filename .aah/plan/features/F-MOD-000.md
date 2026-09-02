## Id
F-MOD-000

## Title
Walking Skeleton — Backend Verification & Frontend Scaffold

## Module Ref
MOD-000

## Description
This module has two parallel concerns: (1) confirm the existing Express 5 / MongoDB backend is fully operational before any frontend code is written, and (2) scaffold the React 19 + TypeScript + Vite frontend with every shared structural piece the remaining modules depend on.

**Backend verification (db + api layers)**

The backend server runs at `localhost:3000` with MongoDB connected via Mongoose. The existing codebase defines two Mongoose models — `User` and `Recipe` — and exposes three route groups: auth (`/api/users`), recipes (`/api/recipes`), and AI proxy (`/api/ai`). This module verifies those models and every route are reachable and return correct HTTP status codes and schema-conformant JSON payloads. No backend code is changed; only tests and the seed script are added. Authoritative model shapes are in `knowledge/backend-README.md` and `knowledge/DESIGN.md`; the running server is the source of truth.

**Frontend scaffold (ui/ux layer)**

A Vite 7 + React 19 + TypeScript project is initialized under `client/`. `tsconfig.json` must have `strict: true` (standard TS-TYPE-002). The scaffold delivers, and only these pieces — no page implementations:

- **React Router 6 route tree** — seven route stubs: `/` (LandingPage), `/recipes` (RecipesPage), `/recipes/:id` (RecipeDetailPage), `/login` (LoginPage), `/signup` (SignupPage), `/dashboard` (DashboardPage, wrapped in RequireAuth), `/ai-assistant` (AIAssistantPage).
- **AuthContext** — React Context + `useContext` holding `{ user, token, login, logout }`. JWT stored in `localStorage`. `login()` saves the token; `logout()` clears it and redirects to `/`.
- **RequireAuth** — wrapper component that reads AuthContext; redirects unauthenticated visitors to `/login`.
- **Axios instance** — configured with `baseURL: 'http://localhost:3000'` and a request interceptor that attaches `Authorization: Bearer <token>` from AuthContext when a token is present.
- **Navbar shell** — renders the persistent top-nav per the navigation contract in `.aah/architecture/design-spec.yaml` (navigation section): logo linking to `/`, Recipes, AI Assistant links visible to all; Login visible to guests only; Dashboard and Log Out visible to authenticated creators only. Styled to the "Modern Pantry" direction defined in `.aah/architecture/design/direction.html` — white surface nav, sage-green (`#3A7D5E`) accent, pill-shaped CTA, bold sans display font — using only the token variables defined in `.aah/architecture/design-spec.yaml` (tokens block). The Navbar shell appears as region A of screen `01-landing` in `.aah/architecture/applications_wireframes/01-landing.html`.
- **env-check.cjs** — CommonJS module (runs via Node `--require` or imported at server/app entry) that reads `MONGODB_URI`, `JWT_SECRET`, and `GEMINI_API_KEY` from `process.env`; if any are missing it exits with code 1 and prints `ERR_CDR_78_EX_CONFIG: missing required env vars: [VAR_NAME, …]. Copy .env.example → .env and fill in values.` It is invoked from the backend entrypoint before any Mongoose or route initialization.
- **Seed script** (`scripts/seed.js` or equivalent runnable via `npm run seed`) — connects to MongoDB, creates at least one User document (hashed password, valid email) and at least one Recipe document (title, description, imageUrl, ingredients array, instructions array, tags array) so all other modules can run functional tests against real data without manual setup.

Design authority for the Navbar and all future screens: `.aah/architecture/design-spec.yaml` (ui_system, tokens, direction, per-screen regions) and `.aah/architecture/design/direction.html` (visual comp). No screen-level layouts are implemented in this module; those belong to MOD-001 through MOD-004.

**Behavioral expectations**

- Given the backend is running with a valid MONGODB_URI, when `GET http://localhost:3000/api/recipes` is called, then the response is HTTP 200 with a JSON array (may be empty) and `Content-Type: application/json`.
- Given the backend is running, when `POST http://localhost:3000/api/users/signup` is called with a unique email and password, then the response is HTTP 201 with a JSON body containing a `token` field.
- Given a valid JWT from signup or login, when `POST http://localhost:3000/api/users/login` is called with correct credentials, then the response is HTTP 200 with a `token` field.
- Given a valid creator JWT, when `POST http://localhost:3000/api/recipes` is called with a complete recipe payload (title, description, imageUrl, ingredients, instructions, tags), then the response is HTTP 201 with the created recipe document including `_id`.
- Given a valid creator JWT and an existing recipe `_id`, when `PUT http://localhost:3000/api/recipes/:id` is called, then the response is HTTP 200 with the updated recipe.
- Given a valid creator JWT and an existing recipe `_id`, when `DELETE http://localhost:3000/api/recipes/:id` is called, then the response is HTTP 200 or 204.
- Given the backend is running, when `POST http://localhost:3000/api/ai/stream` is called with `{ "prompt": "Hello" }` and a valid GEMINI_API_KEY configured, then the response streams with `Content-Type: text/event-stream` and emits at least one data chunk before closing.
- Given the MongoDB connection string is valid, when the backend starts, then Mongoose connects without error and the `users` and `recipes` collections are accessible (User and Recipe models resolve without schema errors).
- Given `MONGODB_URI`, `JWT_SECRET`, or `GEMINI_API_KEY` is absent from the environment, when env-check.cjs runs, then the process exits with code 1 and logs `ERR_CDR_78_EX_CONFIG` naming every missing variable and referencing `.env.example`.
- Given all three required env vars are present, when env-check.cjs runs, then the process does not exit and prints no error.
- Given the `client/` project is initialized, when `npm run dev` is executed inside `client/`, then Vite starts without TypeScript errors and the dev server is reachable at `http://localhost:5173`.
- Given the React app is running at `http://localhost:5173`, when a browser visits `/`, `/recipes`, `/recipes/test-id`, `/login`, `/signup`, and `/ai-assistant`, then each route renders a stub page (no crashes, no blank screen, no uncaught exceptions in the console).
- Given the React app is running, when a browser visits `/dashboard` without a JWT in localStorage, then the user is redirected to `/login`.
- Given the React app is running, when a browser visits `/dashboard` with a valid JWT in localStorage, then the DashboardPage stub renders without redirect.
- Given the Navbar shell renders, when the user is not authenticated (no JWT in localStorage), then the Navbar displays the Recipes, AI Assistant, and Login links and does not display Dashboard or Log Out.
- Given the Navbar shell renders, when the user is authenticated (JWT present in localStorage), then the Navbar displays Dashboard and Log Out links and does not display Login.
- Given the axios instance is imported, when a request is made while a JWT is present in AuthContext, then the request carries an `Authorization: Bearer <token>` header; when no JWT is present, no Authorization header is added.
- Given the seed script (`npm run seed`) is executed against a running MongoDB instance, then at least one User and at least one Recipe document exist in the database afterward, and subsequent calls to `GET /api/recipes` return a non-empty array.
- Given the `client/` TypeScript project, when the TypeScript compiler runs (`tsc --noEmit`), then it exits 0 with `strict: true` active and no `any` type usages in scaffold files.
- Given the resolved design system (`frontend-design`, tokens from `.aah/architecture/design-spec.yaml`) and direction.rules ("Modern Pantry"), when the Navbar shell renders in any of its required states (`guest`, `creator`), then only token values from the spec's color/type/spacing/radius blocks are used in its CSS — no ad hoc hex values or bare pixel measurements outside the spacing scale appear in `Navbar.css`.
- Given `.env.example` exists in the project root, when any developer clones the repository, then `MONGODB_URI`, `JWT_SECRET`, and `GEMINI_API_KEY` are listed as named entries with safe placeholder values (no real secrets), so `cp .env.example .env` gives them a complete template.

## Layers
- db
- api
- ui/ux

## Dependencies

## Required Env Variables
- MONGODB_URI — MongoDB connection string for data storage (validated at startup)
- JWT_SECRET — JWT signing secret for authentication (validated at startup)
- GEMINI_API_KEY — Gemini API key for AI feature (validated at startup)

## Lint Config
Before writing any application code, for each root below: create its manifest first, then run
`aah run core.scaffold.project ensure-lint-config --project-path "$PROJECT_DIR" --package-root <root> --install`
— it reads the manifest to pick the linter, writes the config, adds the linter to dev dependencies
and installs it, and never clobbers an existing config. Where a source path is given, copy that file
into the root first, then run the same command. Commit the configs with this module.

- client — default

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