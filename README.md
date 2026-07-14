# responsive_fullstack_web_system

English | [简体中文](./README.zh-CN.md)

A reusable skill for quickly building **responsive full-stack web management systems** with a unified frontend-backend architecture.

## Overview

This skill defines a complete architecture pattern for AI agents to generate production-ready admin-style web applications from a single natural-language request such as **"build a responsive full-stack web system"**.

Instead of scaffolding yet another template that requires you to install databases, configure services, and wire up deployment pipelines, the generated project is **self-contained**:

- **Built-in SQLite database** managed by Prisma ORM — no separate database server to install or configure.
- **Built-in backend services** using Express.js — authentication, CRUD APIs, file uploads, and business logic run out of the box.
- **Single-port publishing** — in production, the Express backend serves the built Vue frontend as static files, so only one external port is exposed.
- **Responsive by default** — the UI adapts to both desktop and mobile (H5) screens using TailwindCSS v4 and a responsive `MainLayout`.
- **Docker-ready** — multi-stage `Dockerfile` + `docker-compose.yml` with volume persistence for the database, uploads, and logs.

### Why this skill?

| Advantage | What it means for you |
|-----------|----------------------|
| **Zero external dependencies** | SQLite is embedded; no Postgres, MySQL, Redis, or cloud service is required to run locally or in production. |
| **One port, one process** | Frontend and backend share the same origin and port in production, eliminating CORS issues and simplifying reverse-proxy setup. |
| **Opinionated but flexible** | Clear conventions for controllers, routes, API clients, and views make generated code easy to extend. |
| **AI-agent friendly** | The skill file is plain Markdown with structured rules, so it works with Claude Code, Cursor, Windsurf, Cline, and any other agent that can load a skill/context file. |
| **8-step module flow** | Adding a new module follows a repeatable path: Prisma schema → backend CRUD → frontend API/types → pages → routes → menu entry. |

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend framework | Vue 3 + TypeScript + Vite |
| Styling | TailwindCSS v4 (`@tailwindcss/vite` plugin) |
| State management | Pinia (with `localStorage` persistence) |
| Routing | Vue Router 4 (`createWebHistory`) |
| HTTP client | Axios (with interceptors that unwrap `{ code, message, data }`) |
| Icons | lucide-vue-next |
| Backend framework | Express.js + TypeScript (ESM) |
| Database | SQLite + Prisma ORM |
| File uploads | multer |
| Deployment | Docker multi-stage build + docker-compose |

## Key Features

- **Single repository** with `frontend/` and `backend/` directories.
- **Built-in SQLite** database; schema defined in `backend/prisma/schema.prisma`.
- **Responsive PC / H5 layout** handled by `MainLayout.vue` and TailwindCSS utility classes.
- **Production single-port mode**: the Express server hosts the built frontend static assets and provides SPA fallback.
- **Docker deployment** with volume mounts for `data/`, `uploads/`, and logs.
- **Unified API response format** `{ code, message, data }` on both frontend and backend.
- **Controller singleton + route delegation** pattern on the backend.
- **Standard CRUD scaffolding** for every module: list, form, detail views plus pagination and keyword search.

## Installation

This skill is distributed as a plain Markdown skill file (`SKILL.md`). Install it manually in whichever AI agent or IDE you use.

### Option 1: Clone the repository

```bash
git clone https://github.com/dzstudio/responsive_fullstack_web_system.git
cd responsive_fullstack_web_system
```

Then load `SKILL.md` as a skill/context file in your agent.

### Option 2: Download only the skill file

```bash
mkdir -p ~/.local/share/ai-skills/responsive_fullstack_web_system
curl -L https://raw.githubusercontent.com/dzstudio/responsive_fullstack_web_system/master/SKILL.md \
  -o ~/.local/share/ai-skills/responsive_fullstack_web_system/SKILL.md
```

### Loading the skill in popular agents

- **Claude Code**: place `SKILL.md` in your project's `.claude/skills/` directory, or reference it with `/add-skills` / the skills system your version supports.
- **Cursor**: add `SKILL.md` to your project's `.cursor/rules/` directory, or load it via **Settings > AI Rules**.
- **Windsurf / Cline / other agents**: load `SKILL.md` as a project rule, context file, or custom instruction depending on the agent's convention.

> The skill content is written in standard Markdown with YAML frontmatter, so any agent that reads project instructions can consume it.

## Usage

Once the skill is loaded, tell your agent:

> "Build a responsive full-stack web system."

The agent will then follow the architecture, directory structure, code conventions, and deployment rules defined in `SKILL.md` to generate the project.

## Project Structure

After generation, the project looks like this:

```
project/
├── frontend/
│   ├── src/
│   │   ├── api/            # API modules (one file per business domain)
│   │   ├── components/     # Shared components (PaginationBar, ConfirmDialog, etc.)
│   │   ├── composables/    # Composables (useConfirm, etc.)
│   │   ├── layouts/        # Layout components (responsive MainLayout.vue)
│   │   ├── router/         # Route definitions
│   │   ├── stores/         # Pinia stores
│   │   ├── styles/         # Global styles (CSS variables + TailwindCSS)
│   │   ├── types/          # TypeScript type definitions
│   │   ├── utils/          # Utilities (request.ts, format.ts, etc.)
│   │   └── views/          # Page views grouped by module
│   │       └── <module>/
│   │           ├── <Module>List.vue
│   │           ├── <Module>Form.vue
│   │           └── <Module>Detail.vue
│   ├── index.html
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── package.json
├── backend/
│   ├── src/
│   │   ├── controllers/    # Controller classes (business logic)
│   │   ├── middleware/     # Middleware (file upload, etc.)
│   │   ├── routes/         # Route definitions delegating to controllers
│   │   ├── utils/          # Utilities (response.ts, generator.ts)
│   │   └── main.ts         # Express application entry
│   ├── prisma/
│   │   └── schema.prisma   # Database models
│   ├── scripts/            # Data import scripts
│   ├── uploads/            # Uploaded files
│   └── package.json
├── Dockerfile              # Multi-stage build
├── docker-compose.yml      # Single service + volume mounts
└── data/                   # SQLite database file (volume mount)
```

## Deployment

### Local development

```bash
# Backend
cd backend
npm install
npx prisma migrate dev
npm run dev

# Frontend (in another terminal)
cd frontend
npm install
npm run dev
```

### Production with Docker

```bash
docker-compose up --build -d
```

The backend serves the built frontend on the configured `PORT` (default `8080`). Only this single port needs to be exposed externally.

## License

MIT
