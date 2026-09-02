## Id
F-MOD-003

## Title
Recipe CRUD — Create, Edit & Delete

## Module Ref
MOD-003

## Description
MOD-003 implements creator-only recipe management from the authenticated dashboard. It adds three API-backed mutations — create (POST /api/recipes), edit (PUT /api/recipes/:id), and delete (DELETE /api/recipes/:id) — and the UI flows that trigger them, all updating the dashboard grid without a full page reload. The module extends the dashboard screen established by MOD-002 and relies on the axios instance, AuthContext, and RequireAuth scaffold from MOD-000.

**Stack:** React 19 + TypeScript + Vite (frontend), Express 5 / MongoDB backend (pre-existing). Vanilla CSS + Flexbox, one `.css` file per component, desktop-first responsive at 1024 / 768 / 480 px breakpoints. All mutating API calls go through the shared axios instance, which attaches the JWT Authorization header via its interceptor.

**Architecture references (read these directly):**
- `.aah/architecture/module-map.yaml` — MOD-003 entry: description, demo_criteria, layers, depends_on
- `.aah/architecture/design-spec.yaml` — `ui_system: frontend-design`, `token_source: generated`, `component_model: composition`, `direction.comp: design/direction.html`, `direction.rules`; screen `06-dashboard` regions C and D (the two regions owned by MOD-003); `intake.feature_appetite: recommend-standouts`
- `.aah/architecture/applications_wireframes/06-dashboard.html` — region [C★] creator recipe grid with S3 optimistic UI behavior (card appears on create, fades on delete before server confirms); region [D] create/edit modal overlay with title, description, imageUrl, dynamic ingredient rows (name + quantity), dynamic instruction step rows, tags field, Save/Cancel pill buttons, and the confirm-delete dialog; confirm-dialog with recipe-title copy and Cancel / Delete pill buttons
- `design/direction.html` — "Modern Pantry" direction: sage-green `#3A7D5E` primary, off-white `#F5F5F2` body background, pure-white surface, pill-shaped buttons (radius 100px), 800-weight bold headings at −0.03em tracking, uppercase wide-tracking section labels, sage-tint `rgba(58,125,94,.12)` pill tags

**Components to implement:**

1. **RecipeForm** — centered overlay modal (`.modal`, border-radius 12px per token `radius.modal`) containing labeled fields: Title (text input), Description (textarea), Image URL (text input), Ingredients (dynamic list — each row has Name + Quantity inputs and a remove button; "+ Add ingredient" appends a row), Instructions (dynamic list — each row is a step input with a remove button; "+ Add step" appends a row), Tags (comma-separated text input). Save and Cancel pill buttons (radius 100px). Form fields carry uppercase 700-weight labels at 0.07–0.08em tracking per the direction. In **create** mode the modal title is "New Recipe" and all fields are empty. In **edit** mode the modal title is "Edit Recipe" and all fields are prefilled from the selected recipe's current data. The Save button disables and shows a loading indicator while the request is in flight. On server success the modal closes and the dashboard grid updates in-place. On server error an inline error message renders inside the modal without closing it.

2. **Create flow** — the "New Recipe" pill button in the dashboard header [B] opens RecipeForm in create mode. On submit, an optimistic card (S3, styled with `.card-new-badge` "Just added" badge) is inserted at the head of the recipe grid [C★] immediately, before the POST /api/recipes response arrives. On server success the optimistic card is replaced with the confirmed server-returned card. On server error the optimistic card is removed and an inline error message is shown inside the still-open modal.

3. **Edit flow** — each recipe card's "Edit" button opens RecipeForm in edit mode prefilled with that card's data. On submit, PUT /api/recipes/:id is called. On success the card in the grid updates in-place (title, description, image, tags) without a page reload. On error an inline error renders inside the modal and the card is unchanged.

4. **Delete flow** — each recipe card's "Delete" button opens a confirmation dialog (`.confirm-dialog`, border-radius 8px per token `radius.card`) showing the recipe title and the message "This will permanently remove '{title}'. This action cannot be undone." with Cancel and Delete pill buttons. On confirm, the card immediately transitions to `.card-deleting` (S3 optimistic fade) and DELETE /api/recipes/:id is called. On server success the card is removed from the DOM. On server error the card's opacity is restored and an error message is shown.

**Behavioral expectations:**

- Given a logged-in creator is on /dashboard, when they click "New Recipe", then the RecipeForm modal opens in create mode (modal title "New Recipe") with all fields empty and the Save button enabled.
- Given RecipeForm is open in create mode with all required fields filled, when the creator clicks Save, then the Save button disables (loading state), an optimistic card with a "Just added" badge appears immediately in the [C★] grid, and POST /api/recipes is called with the form payload.
- Given POST /api/recipes succeeds, when the response arrives, then the optimistic card is replaced with the server-confirmed card and the modal closes.
- Given POST /api/recipes returns an error, when the response arrives, then the optimistic card is removed from the grid, the modal remains open, the Save button re-enables, and an inline error message is displayed inside the modal.
- Given RecipeForm is open in create mode, when the creator clicks Cancel, then the modal closes with no API call made and the grid is unchanged.
- Given a logged-in creator is on /dashboard with at least one recipe, when they click "Edit" on a card, then the RecipeForm modal opens in edit mode (modal title "Edit Recipe") with all fields prefilled from that recipe's current data.
- Given RecipeForm is open in edit mode with modified data, when the creator clicks Save, then the Save button disables (loading state) and PUT /api/recipes/:id is called with the updated payload.
- Given PUT /api/recipes/:id succeeds, when the response arrives, then the card in the [C★] grid updates in-place with the new values and the modal closes.
- Given PUT /api/recipes/:id returns an error, when the response arrives, then the modal remains open, the Save button re-enables, an inline error message is displayed inside the modal, and the card in the grid is unchanged.
- Given a logged-in creator is on /dashboard with at least one recipe, when they click "Delete" on a card, then the confirmation dialog opens showing the recipe's title and "This action cannot be undone." copy, with Cancel and Delete buttons.
- Given the confirmation dialog is open, when the creator clicks Delete, then the card immediately transitions to the `.card-deleting` faded state (S3 optimistic) and DELETE /api/recipes/:id is called.
- Given DELETE /api/recipes/:id succeeds, when the response arrives, then the faded card is removed from the DOM and the grid count decrements.
- Given DELETE /api/recipes/:id returns an error, when the response arrives, then the faded card is restored to its normal appearance and an error message is shown.
- Given the confirmation dialog is open, when the creator clicks Cancel, then the dialog closes with no API call made and the card is unchanged.
- Given the dashboard is mounted and fetching the creator's recipes, when the GET /api/recipes (owner-filtered) request is in flight, then region [C★] shows a loading state (skeleton or spinner); when populated, then a 3-column card grid renders with each recipe's image, title, description, and Edit/Delete action buttons; when the creator has no recipes, then the empty state "You haven't added any recipes yet. Click 'New Recipe' to get started." is displayed.
- Given the generated token system defined in `.aah/architecture/design-spec.yaml` and the `direction.rules` (pill buttons, bold sans headings, sage-green primary, off-white body background), when any required state from the screen spec's regions renders (region C: loading, populated, empty-no-recipes; region D: closed, open-create, open-edit-prefilled, saving, save-error, save-success), then only tokens from the system's resolved palette, type scale, spacing, and radius are applied — no ad hoc hex values or px values outside the token set.

## Layers
- api
- ui/ux

## Dependencies
- F-MOD-002

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
