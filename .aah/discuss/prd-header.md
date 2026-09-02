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
