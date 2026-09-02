## Id

F-MOD-001

## Title

Guest Recipe Discovery — Landing, Browse & Detail

## Module Ref

MOD-001

## Description

MOD-001 implements the guest-accessible front door to Spoonful: a LandingPage at `/`, a RecipesPage at `/recipes`, and a RecipeDetailPage at `/recipes/:id`. All three routes are publicly accessible without authentication (per `discuss-prd.md` user stories 1, 4, 5 and `resolved-standards.yaml` rule EXTRACTED-004). The module connects the React 19 + TypeScript frontend scaffolded by MOD-000 to the existing Express backend via two endpoints: `GET /api/recipes` (with optional `search` and `tag` query params) and `GET /api/recipes/:id`. All API calls use the axios instance configured in MOD-000 (baseURL `localhost:3000`, JWT header interceptor).

**Visual design:** The "Modern Pantry" direction governs all three screens. The authoritative token set, direction rules, and per-screen region contracts are defined in `.aah/architecture/design-spec.yaml` (screens `01-landing`, `02-browse-recipes`, `03-recipe-detail`). The visual source of truth for the direction comp is `.aah/architecture/design/direction.html`. Design guidance is provided via the `ref: frontend_guidance.md` slot resolved in `design-spec.yaml`. CSS is vanilla + Flexbox, one `.css` file per component, desktop-first with breakpoints at 1024px / 768px / 480px (per `discuss-prd.md` decision `responsive-breakpoints` and `css-approach`).

**Screens and region contracts (consult wireframes and design-spec for exact layout):**

- **01-landing** — wireframe at `.aah/architecture/applications_wireframes/01-landing.html`, screen contract in `design-spec.yaml` id `01-landing`: Region [A] persistent top-nav — white surface, pill-shaped Login CTA in sage green, logo with bold sans + em-colored accent, auth-aware links (Login shown for guest); Region [B] 50/50 split hero — static headline "Find your next favorite recipe." with "Explore Recipes" (primary pill) and "Login" (ghost pill) CTAs on the left, 3 live recipe preview cards (title + tags, fetched from `GET /api/recipes?limit=3`) on the right — states: `loading-skeleton`, `populated`; Region [C] "Recently Added" 3-column discovery strip (same `GET /api/recipes?limit=3` call) with "View all recipes →" link — states: `loading`, `populated`, `empty`; Region [D] dark footer bar with logo and nav links — state: `default`.

- **02-browse-recipes** — wireframe at `.aah/architecture/applications_wireframes/02-browse-recipes.html`, screen contract in `design-spec.yaml` id `02-browse-recipes`: Region [A] top-nav with "Recipes" link active; Region [B] pill search input + tag-chip filter row — real-time filter against `GET /api/recipes?search=<term>&tag=<tag>` — states: `empty`, `typing`, `results-loading`, `filtered`; Region [C] 3-column RecipeCard grid (image, title, description, tag chips) — states: `loading-skeleton`, `populated`, `empty-no-results`; Region [D] dark footer.

- **03-recipe-detail** — wireframe at `.aah/architecture/applications_wireframes/03-recipe-detail.html`, screen contract in `design-spec.yaml` id `03-recipe-detail`: Region [A] top-nav; Region [B] full-width 16:7 aspect-ratio hero image — states: `loading`, `loaded`, `error-fallback`; Region [C] back breadcrumb ("← Back to Recipes"), large title, tag chips, creator attribution — from `GET /api/recipes/:id` — states: `loading`, `populated`, `not-found-404`; Region [D] 2-column content area — ingredients list (name + quantity per row) left, numbered instruction steps right — states: `loading`, `populated`, `empty-list`.

**Components:** `LandingPage`, `RecipesPage`, `RecipeDetailPage`, `RecipeCard` (title, image, description, tags), `SearchBar` (real-time filter by title/tag/ingredient).

**Behavioral expectations:**

- Given the app is running and a guest visits `/`, when the LandingPage mounts, then the top-nav [A] renders with the pill Login CTA and the Spoonful logo; the hero [B] right panel fetches `GET /api/recipes?limit=3`, shows a loading skeleton during the request, then renders up to 3 recipe preview cards (title + tags) once resolved; the "Explore Recipes" and "Login" CTA buttons are visible in the hero left panel.
- Given the LandingPage has mounted, when the "Recently Added" section [C] resolves its `GET /api/recipes?limit=3` call, then 3 RecipeCard components render in a 3-column grid; when the API returns an empty array, then an empty state is displayed in region [C].
- Given a guest clicks "Explore Recipes" on the LandingPage, when the click event fires, then the router navigates to `/recipes`.
- Given a guest visits `/recipes`, when the RecipesPage mounts, then it calls `GET /api/recipes` and renders a 3-column grid of RecipeCard components with a loading skeleton while in-flight; when the API returns an empty array, then an empty-state message "No recipes match your search" is displayed in region [C].
- Given a guest is on `/recipes`, when they type into the SearchBar input, then the component calls `GET /api/recipes?search=<term>` (on input or debounced) and the card grid updates to only show matching recipes; when the input is cleared, then the full recipe list is shown.
- Given a guest is on `/recipes`, when they click a tag-chip filter pill, then the component calls `GET /api/recipes?tag=<tag>` and only cards matching that tag are displayed; when the "All" chip is selected, then the full recipe list is shown.
- Given a guest is on `/recipes`, when they click a RecipeCard, then the router navigates to `/recipes/:id` for that recipe's ID.
- Given a guest visits `/recipes/:id` with a valid recipe ID, when the RecipeDetailPage mounts and `GET /api/recipes/:id` resolves, then: region [B] renders the recipe hero image at a 16:7 aspect ratio with a loading state while in-flight and a fallback placeholder if the image URL fails to load; region [C] renders the recipe title, tag chips, and creator attribution; region [D] renders the ingredients list (each row: ingredient name + quantity) in the left column and numbered instruction steps in the right column.
- Given a guest visits `/recipes/:id` with an invalid or non-existent ID, when `GET /api/recipes/:id` returns a 404, then a not-found error state is displayed in regions [C] and [D] in place of the recipe content.
- Given the RecipeDetailPage is displayed, when the user clicks the "← Back to Recipes" breadcrumb in region [C], then the router navigates back to `/recipes`.
- Given the app is rendered at a 1024px viewport width, when any of the three pages is displayed, then the layout uses its full desktop configuration: 3-column RecipeCard grids, the 50/50 hero split on the landing page, and the 2-column ingredient/instructions layout on the detail page.
- Given the app is rendered at a 768px viewport width, when any of the three pages is displayed, then card grids reflow to 2 columns or stack, the landing hero stacks vertically, and all interactive elements remain reachable and visible without overflow.
- Given the app is rendered at a 480px viewport width, when any of the three pages is displayed, then card grids collapse to single-column, the Navbar adapts, and all text and buttons remain legible without horizontal scrolling.
- Given the "Modern Pantry" design system defined in `.aah/architecture/design-spec.yaml` and direction rules in `.aah/architecture/design/direction.html`, when any required state for screens `01-landing` (loading-skeleton, populated, empty), `02-browse-recipes` (empty, typing, results-loading, filtered, loading-skeleton, populated, empty-no-results), and `03-recipe-detail` (loading, loaded, error-fallback, populated, not-found-404, empty-list) renders, then all colors, type weights, letter-spacing, spacing, and border-radius values come exclusively from the resolved token set in `design-spec.yaml` (palette: `#3A7D5E`, `#F2C94C`, `#1A1A1A`, `#F5F5F2`, `#FFFFFF`, `#6B6B6B`, `#E0E0DC`, `rgba(58,125,94,0.12)`; type: 800-weight headings at −0.03em, uppercase labels at 0.08em/700-weight; spacing: 8/16/24/40/64/80px; radius: pill 100px, card 8px) — no ad hoc hex values or arbitrary pixel values outside the token set are present in any component stylesheet.

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
