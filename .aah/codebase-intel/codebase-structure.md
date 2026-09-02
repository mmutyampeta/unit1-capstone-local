# Codebase Structure

## Directory Layout

```
unit1-capstone/
├── backend/                        # Node.js/Express REST API
│   ├── server.js                   # Entry point — Express app, MongoDB connect, route registration
│   ├── package.json                # Backend deps (express 5, mongoose, bcrypt, jsonwebtoken, etc.)
│   ├── Dockerfile                  # Backend container image
│   ├── docker-compose.dev.yml      # Dev compose config for backend
│   ├── eslint.config.mjs           # ESLint config
│   ├── controllers/
│   │   ├── recipes.js              # Recipe CRUD + instruction sub-document management
│   │   └── users.js                # Auth — signup, login, JWT creation
│   ├── middleware/
│   │   └── verifyToken.js          # JWT Bearer token verification
│   ├── models/
│   │   ├── recipe.js               # Recipe Mongoose schema (embeds ingredients + instructions)
│   │   └── user.js                 # User Mongoose schema (bcrypt pre-save hook)
│   └── routes/
│       ├── recipes.js              # /api/recipes — GET public, POST/PUT/DELETE protected
│       └── users.js                # /api/users/signup, /api/users/login
├── client/                         # React 19 + TypeScript + Vite SPA
│   ├── src/
│   │   ├── main.tsx                # React root — mounts App into #root
│   │   ├── App.tsx                 # Root component (skeleton — "Deloitte React Project")
│   │   ├── index.css               # Global styles
│   │   └── vite-env.d.ts           # Vite env type shim
│   ├── package.json                # Client deps (react 19, vite 7, typescript 5.8)
│   ├── Dockerfile                  # Client container image
│   ├── docker-compose.yml          # Client docker-compose
│   ├── tsconfig.json               # Base TS config
│   ├── tsconfig.app.json           # App TS config
│   ├── tsconfig.node.json          # Node TS config (for vite.config)
│   ├── vite.config.ts              # Vite build config with React plugin
│   └── eslint.config.js            # ESLint config
├── DESIGN.md                       # Full product spec — user stories, flows, AI feature spec
├── README.md                       # Capstone instructions and requirements
└── .dockerignore                   # Docker ignore rules
```

## Entry Points

| File | Role | Description |
|------|------|-------------|
| backend/server.js | Backend entry | Initializes Express, connects Mongoose, registers all routes, starts HTTP listener |
| client/src/main.tsx | Frontend entry | Mounts React app into `#root` DOM element |

## API Routes

| Route | Method | Handler | Auth | Description |
|-------|--------|---------|------|-------------|
| /api/users/signup | POST | usersCtrl.signup | None | Create account, return JWT |
| /api/users/login | POST | usersCtrl.login | None | Authenticate, return JWT |
| /api/recipes | GET | recipesCtrl.getAll | None | List all recipes; supports ?title, ?tag, ?ingredient query filters |
| /api/recipes/:id | GET | recipesCtrl.getOne | None | Get single recipe by ID |
| /api/recipes | POST | recipesCtrl.create | JWT | Create recipe (ownerId set from token) |
| /api/recipes/:id | PUT | recipesCtrl.update | JWT + owner | Update recipe |
| /api/recipes/:id | DELETE | recipesCtrl.delete | JWT + owner | Delete recipe |
| /api/ai/stream | POST | ai proxy | None (planned) | Stream Gemini response for given prompt |

## Module Organization

The backend is organized by MVC layer (routes → controllers → models). The client is currently a bare skeleton — all planned pages (Landing, Browse, Dashboard, AI Assistant) are yet to be implemented following the spec in DESIGN.md.
