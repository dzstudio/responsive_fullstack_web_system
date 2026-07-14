# responsive_fullstack_web_system

A reusable skill for quickly building **responsive full-stack web management systems** with a unified frontend-backend architecture.

> **English** | [中文说明](#中文说明)

---

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

---

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

---

## Key Features

- **Single repository** with `frontend/` and `backend/` directories.
- **Built-in SQLite** database; schema defined in `backend/prisma/schema.prisma`.
- **Responsive PC / H5 layout** handled by `MainLayout.vue` and TailwindCSS utility classes.
- **Production single-port mode**: the Express server hosts the built frontend static assets and provides SPA fallback.
- **Docker deployment** with volume mounts for `data/`, `uploads/`, and logs.
- **Unified API response format** `{ code, message, data }` on both frontend and backend.
- **Controller singleton + route delegation** pattern on the backend.
- **Standard CRUD scaffolding** for every module: list, form, detail views plus pagination and keyword search.

---

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

---

## Usage

Once the skill is loaded, tell your agent:

> "Build a responsive full-stack web system."

The agent will then follow the architecture, directory structure, code conventions, and deployment rules defined in `SKILL.md` to generate the project.

---

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

---

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

---

## License

MIT

---

---

<a name="中文说明"></a>

# 中文说明

一个用于快速构建**前后端一体响应式全栈 Web 管理系统**的可复用技能。

## 简介

本技能定义了一套完整的架构模式，让 AI 智能体能够通过一句自然语言指令（例如**“开发一个响应式全栈 Web 系统”**）生成可直接投产的管理后台类 Web 应用。

生成的项目是**自包含**的，不需要额外安装数据库或服务：

- **内置 SQLite 数据库**，通过 Prisma ORM 管理，无需单独安装或配置数据库服务器。
- **内置 Express 后端服务**，认证、CRUD 接口、文件上传、业务逻辑开箱即用。
- **单端口发布**，生产环境下 Express 直接托管构建后的 Vue 前端静态文件，对外只暴露一个端口。
- **默认响应式**，基于 TailwindCSS v4 和响应式 `MainLayout`，同时适配 PC 和 H5 移动端。
- **支持 Docker**，多阶段 `Dockerfile` + `docker-compose.yml`，通过 volume 持久化数据库、上传文件和日志。

### 核心优势

| 优势 | 说明 |
|------|------|
| **零外部依赖** | SQLite 内嵌，无需 Postgres、MySQL、Redis 或云服务即可本地运行和部署。 |
| **一个端口、一个进程** | 生产环境前后端同源同端口，彻底解决 CORS 问题，反向代理配置更简单。 |
| **规范清晰、易于扩展** | Controller、Router、API 客户端、视图有明确约定，生成的代码易于理解和维护。 |
| **兼容多种 AI Agent** | 技能文件是带 YAML frontmatter 的标准 Markdown，支持 Claude Code、Cursor、Windsurf、Cline 等任何能加载技能/上下文文件的 Agent。 |
| **8 步模块流程** | 新增模块遵循固定路径：Prisma Schema → 后端 CRUD → 前端 API/类型 → 页面 → 路由 → 菜单入口。 |

## 技术栈

| 层级 | 技术 |
|------|------|
| 前端框架 | Vue 3 + TypeScript + Vite |
| UI 样式 | TailwindCSS v4（@tailwindcss/vite 插件） |
| 状态管理 | Pinia（localStorage 持久化） |
| 路由 | Vue Router 4（createWebHistory） |
| HTTP 客户端 | Axios（拦截器自动解包 `{ code, message, data }`） |
| 图标 | lucide-vue-next |
| 后端框架 | Express.js + TypeScript（ESM） |
| 数据库 | SQLite + Prisma ORM |
| 文件上传 | multer |
| 部署 | Docker 多阶段构建 + docker-compose |

## 核心特性

- 单仓库，前端 `frontend/` 和后端 `backend/` 目录分离。
- 内置 SQLite 数据库，模型定义在 `backend/prisma/schema.prisma`。
- 响应式 PC / H5 布局，由 `MainLayout.vue` 和 TailwindCSS 工具类处理。
- 生产单端口模式：Express 托管构建后的前端静态资源并提供 SPA fallback。
- Docker 部署，通过 volume 挂载 `data/`、`uploads/` 和日志目录。
- 前后端统一返回 `{ code, message, data }` 格式。
- 后端采用 Controller 单例 + Router 委托模式。
- 每个模块标准 CRUD 脚手架：列表页、表单页、详情页，支持分页和关键词搜索。

## 安装方式

本技能以纯 Markdown 技能文件（`SKILL.md`）形式分发，请根据你使用的 AI Agent 手动安装。

### 方式一：克隆仓库

```bash
git clone https://github.com/dzstudio/responsive_fullstack_web_system.git
cd responsive_fullstack_web_system
```

然后将 `SKILL.md` 作为技能/上下文文件加载到你的 Agent 中。

### 方式二：只下载技能文件

```bash
mkdir -p ~/.local/share/ai-skills/responsive_fullstack_web_system
curl -L https://raw.githubusercontent.com/dzstudio/responsive_fullstack_web_system/master/SKILL.md \
  -o ~/.local/share/ai-skills/responsive_fullstack_web_system/SKILL.md
```

### 在常见 Agent 中加载

- **Claude Code**：将 `SKILL.md` 放到项目 `.claude/skills/` 目录，或通过你当前版本支持的 `/add-skills` 方式引用。
- **Cursor**：将 `SKILL.md` 放入项目 `.cursor/rules/` 目录，或通过 **Settings > AI Rules** 加载。
- **Windsurf / Cline / 其他 Agent**：根据对应工具的约定，将 `SKILL.md` 加载为项目规则、上下文文件或自定义指令。

> 技能内容使用标准 Markdown 和 YAML frontmatter 编写，任何能读取项目指令的 Agent 都可以消费它。

## 使用方法

加载技能后，告诉你的 Agent：

> “开发一个响应式全栈 Web 系统。”

Agent 会按照 `SKILL.md` 中定义的架构、目录结构、代码规范和部署方式生成项目。

## 项目结构

生成后的项目结构如下：

```
project/
├── frontend/
│   ├── src/
│   │   ├── api/            # API 服务模块，每个业务一个文件
│   │   ├── components/     # 通用组件（PaginationBar, ConfirmDialog 等）
│   │   ├── composables/    # 组合式函数（useConfirm 等）
│   │   ├── layouts/        # 布局组件（MainLayout.vue - 响应式）
│   │   ├── router/         # 路由定义
│   │   ├── stores/         # Pinia store
│   │   ├── styles/         # 全局样式（CSS 变量 + TailwindCSS）
│   │   ├── types/          # TypeScript 类型定义
│   │   ├── utils/          # 工具函数（request.ts, format.ts 等）
│   │   └── views/          # 页面视图，按模块分目录
│   │       └── <module>/
│   │           ├── <Module>List.vue    # 列表页
│   │           ├── <Module>Form.vue   # 新增/编辑表单
│   │           └── <Module>Detail.vue # 详情页
│   ├── index.html
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── package.json
├── backend/
│   ├── src/
│   │   ├── controllers/    # 控制器类（业务逻辑）
│   │   ├── middleware/     # 中间件（文件上传等）
│   │   ├── routes/         # 路由定义，委托给控制器
│   │   ├── utils/          # 工具函数（response.ts, generator.ts）
│   │   └── main.ts         # Express 应用入口
│   ├── prisma/
│   │   └── schema.prisma   # 数据库模型定义
│   ├── scripts/            # 数据导入脚本
│   ├── uploads/            # 文件上传存储目录
│   └── package.json
├── Dockerfile              # 多阶段构建
├── docker-compose.yml      # 单服务 + volume 挂载
└── data/                   # SQLite 数据库文件（volume 挂载点）
```

## 部署说明

### 本地开发

```bash
# 后端
cd backend
npm install
npx prisma migrate dev
npm run dev

# 前端（另一个终端）
cd frontend
npm install
npm run dev
```

### Docker 生产部署

```bash
docker-compose up --build -d
```

后端会在配置的 `PORT`（默认 `8080`）上提供构建后的前端页面，对外只需暴露这一个端口。

## 许可证

MIT
