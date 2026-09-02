## Id
F-MOD-002

## Title
Creator Auth Flow and Protected Dashboard

## Module Ref
MOD-002

## Description
MOD-002 delivers the full authentication loop for recipe creators and the protected dashboard screen that lists their own recipes. It adds three new React pages — `LoginPage` (`/login`), `SignupPage` (`/signup`), and `DashboardPage` (`/dashboard`) — to the frontend scaffold already established by MOD-000, wires them into the existing Express 5 JWT auth endpoints on the backend, and gates `/dashboard` behind the `RequireAuth` wrapper from the scaffold.

**Stack and layout.** The frontend is React 19 + TypeScript + Vite (from MOD-000). Auth pages follow the design contract in `.aah/architecture/design-spec.yaml` (ui_system: `frontend-design`, direction: "Modern Pantry", comp at `.aah/architecture/design/direction.html`). Both `LoginPage` and `SignupPage` use the single-column centered-card layout dictated by the direction rules. `DashboardPage` uses a two-panel dashboard layout — header bar with welcome greeting and "New Recipe" CTA, followed by a 3-column recipe card grid. All CSS is vanilla + Flexbox, one `.css` file per component, desktop-first with breakpoints at 1024 / 768 / 480 px. Wireframes are the authoritative visual spec for all three screens; read them before implementing:

- **04-login** — `.aah/architecture/applications_wireframes/04-login.html`
- **05-signup** — `.aah/architecture/applications_wireframes/05-signup.html`
- **06-dashboard** — `.aah/architecture/applications_wireframes/06-dashboard.html`

Per-screen design contracts (regions, states, data) are declared in `.aah/architecture/design-spec.yaml` under screen ids `04-login`, `05-signup`, and `06-dashboard`. The `06-dashboard` screen is shared with MOD-003; this module owns regions A (nav), B (dashboard header + New Recipe button), and C (creator recipe grid). Region D (create/edit/delete modal) is built by MOD-003.

**API surface.** No backend changes are needed — the Express routes exist. The frontend calls them through the axios instance (baseURL `http://localhost:3000`) scaffolded in MOD-000:

- `POST /api/users/login` — submits `{ email, password }`, receives `{ token, user }` on 200.
- `POST /api/users/signup` — submits `{ email, password }`, receives `{ token, user }` on 201.
- `GET /api/recipes` — called from `DashboardPage` with the JWT Authorization header; the backend filters by the authenticated owner.

**JWT lifecycle.** On successful login or signup the JWT is written to `localStorage` and the `AuthContext` (from MOD-000 scaffold) is updated with `{ token, user }`. The navbar conditionally shows "Dashboard" and "Log Out" for authenticated users and "Login" for guests based on `AuthContext` state. Log Out clears `localStorage`, resets `AuthContext`, and redirects to `/`.

**Ground truth references:** module-map entry `id: MOD-002` in `.aah/architecture/module-map.yaml`; user stories 2 and 3 in `.aah/discuss/discuss-prd.md`; applicable standards from `.aah/plan/resolved-standards.yaml` (TS-SEC-002, RX-SEC-001, RX-SEC-002, TS-TYPE-001, TS-TYPE-002, RX-ARCH-001, RX-ARCH-002, RX-A11Y-001, RX-A11Y-002).

**Behavioral expectations:**

- Given the user navigates to `/login`, when the page renders in the idle state, then a centered card is shown with an "Email" field, a "Password" field, a full-width pill "Log In" button, and a "Sign up" text link to `/signup`; the navbar shows no "Login" button and no "Dashboard" link, matching region A of `04-login.html`.
- Given the user fills valid credentials on `/login` and submits, when `POST /api/users/login` returns 200 with `{ token, user }`, then the JWT is stored in `localStorage`, `AuthContext` is updated with the token and user, and the router navigates to `/dashboard`.
- Given the login form is in-flight, when the request is pending, then the "Log In" button is disabled and a loading indicator is visible; the form cannot be double-submitted.
- Given the user submits incorrect credentials on `/login`, when `POST /api/users/login` returns a 4xx response, then the inline error "Invalid email or password" is displayed within region B without a page reload; the form returns to idle state on correction.
- Given the user navigates to `/signup`, when the page renders in the idle state, then a centered card is shown with "Email", "Password", and "Confirm Password" fields, a full-width pill "Create Account" button, and a "Log in" text link to `/login`, matching region B of `05-signup.html`.
- Given the user submits `/signup` with mismatched password fields, when client-side validation fires before any network call, then the error "Passwords do not match" is displayed inline and no `POST /api/users/signup` request is made.
- Given the user submits `/signup` with a valid new email and matching password, when `POST /api/users/signup` returns 201 with `{ token, user }`, then the JWT is stored in `localStorage`, `AuthContext` is updated, and the router navigates to `/dashboard`.
- Given the user submits `/signup` with an email already in the database, when `POST /api/users/signup` returns a 409 or 4xx response, then the inline error "Email already in use" is displayed and the form remains on `/signup`.
- Given an unauthenticated user navigates directly to `/dashboard`, when `RequireAuth` checks `AuthContext` and finds no JWT, then the user is redirected to `/login` with no dashboard content rendered.
- Given an authenticated creator navigates to `/dashboard`, when `RequireAuth` finds a valid JWT in `AuthContext`, then `GET /api/recipes` is called with the `Authorization: Bearer <token>` header and the response is filtered to the creator's own recipes.
- Given `GET /api/recipes` returns one or more recipes on `/dashboard`, when the data loads, then region C renders a 3-column grid of recipe cards each showing title, image placeholder, description, and "Edit" / "Delete" action buttons, plus a recipe count label, matching region C of `06-dashboard.html`.
- Given `GET /api/recipes` returns an empty array for the creator on `/dashboard`, when the empty state renders, then region C shows "You haven't added any recipes yet. Click 'New Recipe' to get started." with no recipe cards.
- Given `/dashboard` is loading recipes, when the fetch is in-flight, then a loading state (skeleton or spinner) is shown in region C and no partial or stale data is rendered.
- Given the authenticated creator's dashboard navbar (region A), when the creator is logged in, then "Dashboard" is the active nav link and a "Log Out" control is visible; "Login" nav item is hidden.
- Given the creator clicks "Log Out" in the navbar, when the action fires, then the JWT is removed from `localStorage`, `AuthContext` is reset to unauthenticated, the navbar reverts to guest state, and the router navigates to `/`.
- Given the "Modern Pantry" design system resolved from `.aah/architecture/design-spec.yaml` and the direction rules in `.aah/architecture/design/direction.html`, when each required state for screens `04-login` (idle, loading, error-invalid-credentials, success-redirect), `05-signup` (idle, loading, error-duplicate-email, error-password-mismatch, success-redirect), and `06-dashboard` (loading, populated, empty-no-recipes) renders, then only tokens from the resolved palette (`#3A7D5E`, `#F2C94C`, `#1A1A1A`, `#F5F5F2`, `#FFFFFF`, `#6B6B6B`, `#E0E0DC`), type stack (Segoe UI, display-weight 800 / label-weight 700), spacing scale (8 / 16 / 24 / 40 / 64 / 80 px), and radius scale (pill 100px, card 8px, modal 12px) are applied — no ad hoc hex values or pixel values outside these tokens.

## Layers
- api
- ui/ux

## Dependencies
- F-MOD-000

## Required Env Variables

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