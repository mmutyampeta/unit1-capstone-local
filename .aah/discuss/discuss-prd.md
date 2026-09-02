# Discuss PRD — unit1-capstone

## Problem Statement

A bootcamp capstone project ("Spoonful") has a fully working Express/MongoDB backend but a completely unbuilt React frontend. The goal is to deliver a polished, working application end-to-end: verify the backend, build and connect the React SPA to every API endpoint, implement responsive design (matching the Figma spec) for mobile, tablet, and desktop, write component tests for 5 UI components, and add an AI-powered text-analyzer feature that streams responses from Google Gemini. Development follows a quick, iterative cycle — small features built and verified one at a time.

## Solution

The backend (Express 5, MongoDB/Mongoose, JWT auth) is already implemented and will be verified first via direct API testing. The React 19 + TypeScript frontend will be built with Vite, using `react-router-dom` v6+ for client-side routing across six routes: `/` (Landing), `/login`, `/signup`, `/recipes` (Browse), `/recipes/:id` (Detail), `/dashboard` (creator CRUD), and `/ai-assistant`. Authentication state will be held in `AuthContext` (React Context + `useContext`) with the JWT stored in `localStorage`; protected routes use a `RequireAuth` wrapper component. API calls go through an axios instance configured with the backend base URL. CSS is vanilla + Flexbox, desktop-first with breakpoints at 1024px / 768px / 480px, with one `.css` file per component. Tests for 5 components (Navbar, RecipeCard, RecipeForm, SearchBar, LoginForm) will be written with Vitest + React Testing Library. The AI feature lives at `/ai-assistant` (no auth required, linked from nav): a text-analyzer where the user pastes text and clicks one of three action buttons (Summarize / Extract Key Points / Classify Tone), each sending a prefixed prompt to `POST /api/ai/stream` on the backend, which proxies to Gemini's SSE endpoint; the response streams token-by-token into the UI, and the last 3 prompt/response pairs are kept in React state (not persisted).

## User Stories

1. As a visitor, I want to land on a welcome page with CTAs to browse recipes or log in, so I can orient myself immediately.
2. As a new user, I want to sign up with email and password, so I can become a recipe creator.
3. As a creator, I want to log in and be redirected to my dashboard, so I can manage my recipes.
4. As any user, I want to browse all recipes and search by title, tag, or ingredient, so I can find something to cook.
5. As any user, I want to click a recipe card and see its full details (ingredients, instructions, image), so I can follow the recipe.
6. As a creator, I want to create a new recipe from a form on my dashboard, so I can share it with others.
7. As a creator, I want to edit one of my recipes, so I can fix mistakes or improve it.
8. As a creator, I want to delete one of my recipes with a confirmation prompt, so I can manage my content.
9. As any user, I want to navigate to the AI Assistant page, paste some text, and choose an analysis (Summarize / Key Points / Tone), so I can get instant AI-powered feedback streamed progressively.
10. As a developer, I want the app to look correct on mobile, tablet, and desktop, so it passes the responsive design requirement.

---

## Overview & Intent

| Field | Value |
|-------|-------|
| Project | `unit1-capstone` |
| Complexity tier | `mvp` |
| Business domain(s) | _None selected._ |
| Technical archetype(s) | _None selected._ |
| Discovery started | 2026-09-02 |

## Scope

**In scope** — every decision recorded below.

No deferred ideas.

## Decisions Made

| slug_id | Area | Decision | Rationale |
|---------|------|----------|-----------|
| `cloud-vs-local` | Infrastructure | local | Running locally with npm only, no Docker, no cloud |
| `data-storage-present` | Data | yes | MongoDB via Mongoose already wired in server.js |
| `custom-ui-required` | Frontend | yes | Entire target area is building the React SPA frontend |
| `compliance-regimes` | Compliance | none | Bootcamp capstone project, no regulatory constraints |
| `docker-installed` | Infrastructure | no | User: "don't worry about Docker, we don't need any container" |
| `business-domain` | Domain & Archetype | none | No harness domain brief matches a recipe management web app |
| `technical-archetype` | Domain & Archetype | none | Standard CRUD web app with light AI feature — no archetype overlay needed |
| `routing-library` | Frontend Routing & Auth | react-router-dom v6+ | DESIGN.md specifies 6 routes; react-router-dom is the standard for React SPAs |
| `jwt-storage` | Frontend Routing & Auth | localStorage | Simple; XSS risk acceptable for local bootcamp project |
| `global-state` | Frontend Routing & Auth | React Context + useContext | Built-in, no extra deps, sufficient for this scale |
| `css-organization` | CSS & Responsive Design | one CSS file per component | Clean separation, compatible with Vite |
| `responsive-breakpoints` | CSS & Responsive Design | desktop-first: 1024px / 768px / 480px | Desktop-first, shrink down with media queries |
| `css-approach` | CSS & Responsive Design | vanilla CSS + Flexbox | Brief explicitly requires CSS/Flexbox; no framework |
| `testing-framework` | Testing | Vitest + React Testing Library | Native Vite integration, zero extra config, fast |
| `test-targets` | Testing | Navbar, RecipeCard, RecipeForm, SearchBar, LoginForm | Covers main UI surface area across all major user flows |
| `ai-rendering` | AI Feature | streaming token-by-token (SSE ReadableStream) | Progressive display, matches backend SSE spec |
| `ai-ux-controls` | AI Feature | three action buttons: Summarize / Key Points / Classify Tone | Explicit UX; each button sends a prefixed prompt to Gemini |
| `ai-session-history` | AI Feature | last 3 prompt/response pairs in-memory (React state, not persisted) | DESIGN.md specifies last 3 pairs; no persistence required |
| `ai-route` | AI Feature | /ai-assistant (linked from main nav, no auth required) | DESIGN.md specifies GET /ai-assistant, accessible to all users |

### Revisions (surgical re-float)

| slug_id | Prior response | Revised response | Reason |
|---------|---------------|-----------------|--------|
| `ai-route` | Shell path expansion artifact | `/ai-assistant (linked from main nav, no auth required)` | Shell path expansion corrupted the leading slash; corrected |

## Pre-Resolved (brownfield / inferred)

| slug_id | Value | Source |
|---------|-------|--------|
| `backend-language` | Node.js/JavaScript (CommonJS) | codebase |
| `backend-framework` | Express 5 | codebase |
| `data-store-choice` | MongoDB via Mongoose | codebase |
| `auth-strategy` | JWT Bearer tokens (24h expiry, JWT_SECRET env var) | codebase |
| `password-security` | bcrypt with 6 salt rounds via Mongoose pre-save hook | codebase |
| `api-style` | REST JSON | codebase |
| `frontend-language` | TypeScript | codebase |
| `frontend-framework` | React 19 + Vite 7 | codebase |
| `ai-provider` | Google Gemini (gemini-3.6-flash, SSE streaming) | inferred |
| `ai-backend-proxy` | Backend-only API key; frontend calls /api/ai/stream only | inferred |
| `deployment-target` | Local npm scripts only — no Docker, no cloud | user |

## Constraints & Compliance

Constraint gate: **PASS** (no blocking violations).

No compliance regimes apply. No flags or warnings.

## Open Questions for Analysis

_None — resolved during discovery._

## Constraint Audit — Checks A-F

**Overall:** ✅ PASS  
**Ran at:** 2026-09-02

| Check | Result | Detail |
|-------|--------|--------|
| A — Mandatory coverage | PASS | clean |
| B — Forbidden options | PASS | clean |
| C — Whitelist adherence | PASS | clean |
| D — Service scope | PASS | clean |
| E — Mandated options | PASS | clean |
| F — Elimination provenance | PASS | clean |
