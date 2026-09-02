# Dependency Map

## External Dependencies

### Backend

| Package | Version | Purpose | Category |
|---------|---------|---------|----------|
| express | ^5.1.0 | HTTP server and routing framework | Framework |
| mongoose | ^8.17.0 | MongoDB ODM — schema definition and querying | Database |
| bcrypt | ^6.0.0 | Password hashing with salt rounds | Security |
| jsonwebtoken | ^9.0.2 | JWT sign and verify for stateless auth | Auth |
| dotenv | ^17.2.1 | Load env vars from .env file | Config |
| cors | ^2.8.5 | Enable cross-origin requests from React client | Networking |
| morgan | ^1.10.1 | HTTP request logging middleware | Observability |
| eslint | ^9.0.0 | Static code linting | Dev Tools |

### Client

| Package | Version | Purpose | Category |
|---------|---------|---------|----------|
| react | ^19.1.1 | UI component library | Framework |
| react-dom | ^19.1.1 | DOM renderer for React | Framework |
| vite | ^7.1.0 | Fast build tool and dev server | Build |
| typescript | ~5.8.3 | Type checking | Language |
| @vitejs/plugin-react | ^4.7.0 | React Fast Refresh + JSX transform for Vite | Build |
| eslint | ^9.32.0 | Static code linting | Dev Tools |
| typescript-eslint | ^8.39.0 | TypeScript rules for ESLint | Dev Tools |
| @types/react | ^19.1.9 | React TypeScript type definitions | Types |
| @types/react-dom | ^19.1.7 | React DOM TypeScript type definitions | Types |

### Planned (per DESIGN.md / README.md)

| Package | Purpose |
|---------|---------|
| axios | HTTP client for API calls from React frontend |
| react-router-dom | Client-side routing for SPA navigation |

## Internal Module Dependencies

| Module | Depends On | Depended By |
|--------|-----------|-------------|
| backend/server.js | express, mongoose, morgan, cors, routes/users, routes/recipes | — (entry point) |
| backend/routes/users.js | express, controllers/users | server.js |
| backend/routes/recipes.js | express, controllers/recipes, middleware/verifyToken | server.js |
| backend/controllers/users.js | models/user, jsonwebtoken | routes/users.js |
| backend/controllers/recipes.js | models/recipe | routes/recipes.js |
| backend/middleware/verifyToken.js | jsonwebtoken | routes/recipes.js |
| backend/models/user.js | mongoose, bcrypt | controllers/users.js |
| backend/models/recipe.js | mongoose | controllers/recipes.js |
| client/src/main.tsx | react, react-dom, App.tsx, index.css | — (entry point) |
| client/src/App.tsx | — | main.tsx |
