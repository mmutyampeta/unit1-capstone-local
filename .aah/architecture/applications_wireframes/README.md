# Spoonful — UI Wireframes & Design Contract

Generated during the `aah-ux` port of the `/aah-arch` architecture phase.

## What Was Decided

- **7 screens** confirmed in Checkpoint 4 of 6 with all 4 signature UX moments included (S1–S4).
- **Direction B — "Modern Pantry"** chosen in Checkpoint 5 of 6: bold geometric sans, sage-green primary, off-white background, split two-panel layouts, pill CTAs.
- **No structural changes** were requested during Checkpoint 6 of 6 — wireframes approved on first review.
- **Create Recipe flow**: modal/overlay on Dashboard (not a separate page).
- **Recipe Detail**: read-only for all users (Edit/Delete only from Dashboard).

---

## Screen List

| File | Screen | Spine |
|------|--------|-------|
| `01-landing.html` | Landing | What can I discover today? |
| `02-browse-recipes.html` | Browse Recipes | What's available to cook? |
| `03-recipe-detail.html` | Recipe Detail | What's in this recipe and how do I make it? |
| `04-login.html` | Login | How do I access my recipes? |
| `05-signup.html` | Signup | How do I become a recipe creator? |
| `06-dashboard.html` | Dashboard | What are my recipes, and how do I manage them? |
| `07-ai-assistant.html` | AI Assistant | How can I analyze this text? |

---

## IA / Nav Model

**Model:** Persistent top navbar on every screen.

**Entry screen:** `01-landing` (`/`).

**Nav items:**
| Item | Route | Visible |
|------|-------|---------|
| Spoonful (logo) | `/` | Always |
| Recipes | `/recipes` | Always |
| AI Assistant | `/ai-assistant` | Always |
| Login | `/login` | Guest only (no JWT) |
| Dashboard | `/dashboard` | Creator only (JWT present) |
| Log Out | — (clears localStorage) | Creator only |

**Protected routes:** `/dashboard` — wrapped in `RequireAuth` component; redirects to `/login` if no JWT.

**Auth flows:**
- Login → redirect to `/dashboard`
- Signup → redirect to `/dashboard`
- Log out → clear `localStorage` → redirect to `/`

---

## Viewport Targets

All screens: **desktop-first**, minimum 1280px. Responsive breakpoints (per decision registry):
- 1024px — tablet landscape
- 768px — tablet portrait
- 480px — mobile

---

## Signature UX Moments

| # | Moment | Screen | Description |
|---|--------|--------|-------------|
| S1 | Hero image | Recipe Detail | 16:7 aspect-ratio food photo, prominent crop |
| S2 | Streaming indicator | AI Assistant | Animated dots while waiting for first SSE token |
| S3 | Optimistic UI | Dashboard | Card appears on create, fades on delete before server confirms |
| S4 | Tag-chip filter pills | Browse Recipes | One-click filter by ingredient/tag alongside text SearchBar |

---

## Screen → Backend Hints

| Screen | Region | Data Dependency | Candidate Endpoint | Assumed? |
|--------|--------|-----------------|-------------------|----------|
| Landing [B] hero right | Preview cards | Latest 3 recipes | `GET /api/recipes?limit=3` | No |
| Landing [C] featured | Recipe grid | Latest 3 recipes | `GET /api/recipes?limit=3` | No |
| Browse [B] search | Text query | Recipe search | `GET /api/recipes?search=<q>` | No |
| Browse [B] filter pills | Tag filter | Tag filter | `GET /api/recipes?tag=<t>` | Yes (query param TBD) |
| Browse [C] grid | All recipes | Full list | `GET /api/recipes` | No |
| Recipe Detail [B] | Hero image | Recipe by ID | `GET /api/recipes/:id` | No |
| Recipe Detail [C] | Meta | Recipe by ID | `GET /api/recipes/:id` | No |
| Recipe Detail [D] | Ingredients + steps | Recipe by ID | `GET /api/recipes/:id` | No |
| Login [B] | Auth form | Login | `POST /api/users/login` | No |
| Signup [B] | Auth form | Registration | `POST /api/users/signup` | No |
| Dashboard [C] | Creator's recipes | Owner-filtered list | `GET /api/recipes` (JWT-filtered server-side) | Yes (filter by owner in middleware) |
| Dashboard [D] modal | Create recipe | New recipe | `POST /api/recipes` | No |
| Dashboard [D] modal | Edit recipe | Update recipe | `PUT /api/recipes/:id` | No |
| Dashboard [D] modal | Delete confirm | Delete recipe | `DELETE /api/recipes/:id` | No |
| AI Assistant [C] | 3 action buttons | AI stream | `POST /api/ai/stream { prompt }` | No |
| AI Assistant [D] | Streaming pane | SSE stream | `POST /api/ai/stream` (ReadableStream) | No |

---

## Screen → Module Mapping

| Screen | Module(s) | Responsibility |
|--------|-----------|----------------|
| `01-landing` | MOD-000, MOD-001 | App scaffold + recipe discovery entry point |
| `02-browse-recipes` | MOD-001 | Recipe browse + search/filter |
| `03-recipe-detail` | MOD-001 | Full recipe display |
| `04-login` | MOD-002 | Creator authentication (login) |
| `05-signup` | MOD-002 | Creator authentication (registration) |
| `06-dashboard` | MOD-002, MOD-003 | Auth context + full recipe CRUD |
| `07-ai-assistant` | MOD-004 | Gemini SSE text analyzer |

---

## Open Questions

### Resolved
| Question | Resolution |
|----------|------------|
| Create Recipe flow: modal or separate page? | Modal/overlay on Dashboard |
| Recipe Detail: edit/delete buttons for owner? | Read-only for all — CRUD from Dashboard only |
| Search: client-side or server-side? | Server-side (`GET /api/recipes?search=`) |
| Auth state storage | JWT → `localStorage`, `AuthContext` populated on load |
| Tag filter endpoint | Assumed `?tag=<t>` query param (to be confirmed with backend) |
| Dashboard recipe filtering | Assumed server-side owner filter via JWT middleware |

### Still Open (minor)
| Question | Notes |
|----------|-------|
| Does GET /api/recipes support `?tag=` filtering? | Backend routes need verification; S4 tag pills depend on it |
| Dashboard: does GET /api/recipes return only owner's recipes when JWT provided, or all? | Need to confirm backend filter behavior |

---

## Knowledge References Used

| File | Used For |
|------|---------|
| `knowledge/DESIGN.md` | Screen list, route definitions, API endpoints, AI feature spec |
| `knowledge/project-brief.md` | Feature scope, development approach |
| `.aah/discuss/decision-registry.yaml` | JWT strategy, CSS approach, routing library, AI rendering, auth state |
| `.aah/discuss/discuss-prd.md` | User stories mapped to screens |
| `.aah/architecture/module-map.yaml` | Module assignments per screen |

---

## Design System

| Property | Value |
|----------|-------|
| `ui_system` | `frontend-design` |
| `mode` | `light` |
| `token_source` | `generated` |
| `component_model` | `composition` |
| `guidance` | `frontend_guidance.md` |
| `install` | `none` |
| `binding.raw_hex` | `within-tokens` |

**Direction:** `Modern Pantry` — see `design/direction.html` for the hi-fi entry-screen comp.

Direction rules applied to all screens:
1. White top-nav with pill-shaped CTA; logo with sage-green accent letters
2. Two-panel split layouts for content+preview screens; centered single-column for forms
3. Bold 800-weight headings, tight tracking (−0.03em), uppercase wide-tracking section labels
4. Sage #3A7D5E as sole interactive color; off-white #F5F5F2 background; yellow #F2C94C for badges only
