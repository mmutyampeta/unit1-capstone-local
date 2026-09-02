## Id
F-MOD-005

## Title
Component Tests & Responsive CSS Polish

## Module Ref
MOD-005

## Description
This module delivers two parallel workstreams for the Spoonful React SPA (React 19 + TypeScript + Vite): (1) a Vitest + React Testing Library test suite covering five UI components, and (2) a final responsive CSS audit and repair pass across all seven application screens, bringing every layout into conformance with the desktop-first breakpoint scheme at 1024 px, 768 px, and 480 px.

**Testing workstream.** Five test files are written in `client/src/` alongside (or near) the components they cover. Each file uses `@testing-library/react` `render()` against the real component tree rendered in jsdom — no mock frameworks, no stubs beyond plain prop fixtures. The components under test, their locations, and the behaviors each suite must assert are defined in the `testing` layer of `.aah/architecture/module-map.yaml` (MOD-005 entry). Tests are executed via `npm run test` in the `client/` directory (Vitest runner). The test framework decision (`vitest + React Testing Library`) and the five target components are recorded in `.aah/discuss/discuss-prd.md` under `testing-framework` and `test-targets`.

**Responsive polish workstream.** All seven wireframe screens defined in `.aah/architecture/design-spec.yaml` (screen ids `01-landing` through `07-ai-assistant`, files at `.aah/architecture/applications_wireframes/`) are audited at three viewport widths (1024 px / 768 px / 480 px). Layout defects — overflow, hidden interactive elements, broken grid stacking, illegible nav — are fixed in the per-component CSS files (one `.css` file per component, as established in MOD-001 through MOD-004). No new components or routes are introduced; this module only repairs existing CSS.

The design token system is defined in `.aah/architecture/design-spec.yaml` (`tokens` block, `direction.rules`). Every CSS repair must draw exclusively from those tokens: `color.primary` `#3A7D5E`, `color.bg` `#F5F5F2`, `color.surface` `#FFFFFF`, `color.muted` `#6B6B6B`, `color.border` `#E0E0DC`, `color.accent` `#F2C94C`, `color.dark` `#1A1A1A`, `color.primary_tint` `rgba(58,125,94,0.12)`; `radius.pill` `100px`, `radius.card` `8px`, `radius.modal` `12px`, `radius.tag_small` `4px`; spacing values `8px / 16px / 24px / 40px / 64px / 80px`; typeface `'Segoe UI', system-ui, -apple-system, sans-serif`; display weight 800, tracking `−0.03em`; label transform `uppercase`, tracking `0.08em`, weight 700.

**Behavioral expectations**

- Given the five test files (`Navbar.test.tsx`, `RecipeCard.test.tsx`, `RecipeForm.test.tsx`, `SearchBar.test.tsx`, `LoginForm.test.tsx`) are present in the `client/` package, when `npm run test` is run from the `client/` directory, then all five suites exit with status 0 and zero failing assertions.

- Given `Navbar.test.tsx` renders the Navbar component with props representing guest state and creator state, when the rendered output is queried, then all navigation link labels defined in `.aah/architecture/design-spec.yaml` `navigation.nav_items` (`Recipes`, `AI Assistant`, `Login` for guest; `Recipes`, `AI Assistant`, `Dashboard`, `Log Out` for creator) are present in the DOM, and the active-state class or attribute is applied to the link matching the current route prop.

- Given `RecipeCard.test.tsx` renders the RecipeCard component with a fixture recipe object (title, imageUrl, description, tags array), when the rendered output is queried, then the recipe title text, an `<img>` element with the correct `src` and non-empty `alt` attribute, and at least one tag chip are present in the DOM.

- Given `RecipeForm.test.tsx` renders the RecipeForm component (used for create and edit flows), when the rendered output is queried, then input fields for title, image URL, description, and tags are present, and at least one ingredient row and one instruction row are rendered; when a submit event is fired on the form, then the `onSubmit` callback prop is invoked with the current field values.

- Given `SearchBar.test.tsx` renders the SearchBar component with an `onChange` prop callback and an initial empty value, when the rendered text input is present in the DOM and a change event is simulated with a search string, then the `onChange` callback is invoked with that string as its argument.

- Given `LoginForm.test.tsx` renders the LoginForm component with an `onSubmit` prop callback, when the email and password fields are populated and the submit button is activated, then the `onSubmit` callback is invoked with the entered email and password values; when either field is empty and submit is triggered, then the callback is not invoked (client-side required-field constraint).

- Given the `01-landing` screen (`.aah/architecture/applications_wireframes/01-landing.html`) defines a 50/50 hero split (region B) and a 3-column discovery strip (region C), when the viewport width is 768 px, then the hero panels stack to a single column without horizontal overflow; when the viewport is 480 px, then all CTA buttons and text remain fully visible and tappable (minimum 44 px touch target height).

- Given the `02-browse-recipes` screen defines a 3-column recipe card grid (region C) and a pill search input with tag-chip filter row (region B), when the viewport is 768 px, then the grid reflows to 2 columns; when the viewport is 480 px, then the grid is 1 column and the search input spans full width with no clipping.

- Given the `03-recipe-detail` screen defines a 2-column ingredients/instructions layout (region D) and a full-width hero image (region B), when the viewport is 768 px or 480 px, then the 2-column content collapses to single column and the hero image maintains its aspect ratio without overflow.

- Given the `04-login` and `05-signup` screens each define a centered card form (region B), when rendered at 480 px, then the card does not exceed the viewport width, all form fields are full-width within the card, and the pill submit button is fully visible.

- Given the `06-dashboard` screen defines a 3-column creator card grid (region C) and a centered modal overlay (region D), when the viewport is 768 px, then the grid reflows to 2 columns; when the viewport is 480 px, then the grid is 1 column and the modal width is constrained to the viewport with internal padding preserved.

- Given the `07-ai-assistant` screen defines a 3-column action button row (region C) and a streaming output pane (region D), when the viewport is 480 px, then the three action buttons stack vertically or wrap without clipping, the textarea spans full width, and the output pane scrolls vertically within the viewport.

- Given the Navbar (region A across all screens) uses the pill-CTA style defined in `direction.rules` in `.aah/architecture/design-spec.yaml`, when the viewport is 480 px, then navigation links remain legible (no overflow, no hidden items behind layout edges) and the Navbar does not obscure page content.

- Given the design system defined in `.aah/architecture/design-spec.yaml` (`tokens` block, `direction.rules`, `token_source: generated`), when any required state from any screen's `states` fields (e.g., `loading-skeleton`, `populated`, `empty`, `error-fallback`, `idle`, `streaming`, `creator-authenticated`) renders after the responsive fixes are applied, then only tokens from the system's resolved palette, type scale, spacing scale, and radius scale are used — no ad hoc hex values, px literals, or font stacks outside those defined in the `tokens` block.

## Layers
- testing
- ui/ux

## Dependencies
- F-MOD-001
- F-MOD-002
- F-MOD-003
- F-MOD-004

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