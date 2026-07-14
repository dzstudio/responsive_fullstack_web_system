---
name: responsive_fullstack_web_system
description: 当用户说"开发一个响应式全栈Web系统"时，采用前后端一体、SQLite 内置、响应式 PC/H5、单端口部署、Docker 支持的 Vue 3 + Express + Prisma 架构模式进行系统构建。
---

# 响应式全栈 Web 系统

前后端一体响应式 Web 管理系统架构技能。当用户说"开发一个响应式全栈Web系统"时，采用此架构模式。

## 架构特征

- 前后端在同一仓库，`frontend/` + `backend/` 目录分离
- 内置 SQLite 数据库，通过 Prisma ORM 管理
- 同时支持 PC 和 H5 移动端访问（响应式布局）
- 生产环境后端服务同时托管前端静态文件，对外只暴露一个端口
- 支持 Docker 单容器部署，数据库通过 volume 持久化

## 技术栈

| 层 | 技术 |
|---|------|
| 前端框架 | Vue 3 + TypeScript + Vite |
| UI 样式 | TailwindCSS v4（@tailwindcss/vite 插件） |
| 状态管理 | Pinia（localStorage 持久化） |
| 路由 | Vue Router 4（createWebHistory） |
| HTTP 客户端 | Axios（拦截器自动解包响应） |
| 图标 | lucide-vue-next |
| 后端框架 | Express.js + TypeScript（ESM） |
| 数据库 | SQLite + Prisma ORM |
| 文件上传 | multer |
| 部署 | Docker 多阶段构建 + docker-compose |

## 目录结构

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
│   ├── scripts/            # 数据导入脚本（tsx 执行）
│   ├── uploads/            # 文件上传存储目录
│   └── package.json
├── Dockerfile              # 多阶段构建
├── docker-compose.yml      # 单服务 + volume 挂载
└── data/                   # SQLite 数据库文件（volume 挂载点）
```

## 后端代码规范

### 入口文件 main.ts

```ts
import express from 'express'
import cors from 'cors'
import path from 'path'
import { fileURLToPath } from 'url'
import dotenv from 'dotenv'

const __filename = fileURLToPath(import.meta.url)
const __dirname = path.dirname(__filename)
import { PrismaClient } from '@prisma/client'

dotenv.config()

const app = express()
const PORT = process.env.PORT || 8080
export const prisma = new PrismaClient()

// 中间件
app.use(cors())
app.use(express.json())
app.use(express.urlencoded({ extended: true }))
app.use('/uploads', express.static(path.resolve(process.cwd(), 'uploads')))

// API 路由挂载
// app.use('/api/v1/<module>', <module>Routes)

// 健康检查
app.get('/api/health', (req, res) => {
  res.json({ code: 0, message: 'ok', data: null })
})

// 生产环境：serve 前端 + SPA fallback
if (process.env.NODE_ENV === 'production') {
  const staticDir = path.resolve(__dirname, '../../frontend/dist')
  app.use(express.static(staticDir))
  app.get('*', (_req, res) => {
    res.sendFile(path.join(staticDir, 'index.html'))
  })
}

async function start() {
  await prisma.$connect()
  console.log('Database connected')
  app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`)
  })
}

start()
```

关键点：
- Prisma 实例在 main.ts 中创建并 export，控制器从 `../main.js` 导入
- ESM 模式：import 路径必须带 `.js` 后缀
- 生产环境 Express 同时 serve 前端静态文件，SPA fallback 用 `app.get('*')`

### 统一响应格式 (utils/response.ts)

```ts
import { Response } from 'express'

interface ApiResponse<T = any> {
  code: number
  message: string
  data: T | null
}

export function success<T>(res: Response, data: T): void {
  res.json({ code: 0, message: 'success', data })
}

export function error(res: Response, message: string, code: number = 500): void {
  res.json({ code, message, data: null })
}

export function paginate<T>(res: Response, list: T[], total: number, page: number, pageSize: number): void {
  success(res, { list, total, page, pageSize })
}
```

所有接口返回 `{ code, message, data }` 格式。`code === 0` 表示成功。分页用 `paginate()` 包装。

### 控制器模式 (controllers/<module>Controller.ts)

```ts
import { Request, Response } from 'express'
import { prisma } from '../main.js'
import { success, error, paginate } from '../utils/response.js'
import { generateNo } from '../utils/generator.js'

export class <Module>Controller {
  async getList(req: Request, res: Response) {
    try {
      const { keyword, page = 1, pageSize = 20 } = req.query
      const where: any = {}
      if (keyword) {
        where.OR = [
          { field1: { contains: keyword as string } },
          { field2: { contains: keyword as string } },
        ]
      }
      const skip = (Number(page) - 1) * Number(pageSize)
      const take = Number(pageSize)
      const [list, total] = await Promise.all([
        prisma.<module>.findMany({ where, orderBy: { createTime: 'desc' }, skip, take }),
        prisma.<module>.count({ where }),
      ])
      paginate(res, list, total, Number(page), Number(pageSize))
    } catch (err) {
      error(res, '获取列表失败')
    }
  }

  async getDetail(req: Request, res: Response) { /* prisma.<module>.findUnique */ }
  async create(req: Request, res: Response)    { /* generateNo + prisma.<module>.create */ }
  async update(req: Request, res: Response)    { /* prisma.<module>.update */ }
  async delete(req: Request, res: Response)    { /* prisma.<module>.delete */ }
}

export const <module>Controller = new <Module>Controller()
```

每个控制器导出单例实例。getList 标准模式：keyword 多字段 OR 搜索 + 分页。

### 路由模式 (routes/<module>.ts)

```ts
import { Router } from 'express'
import { <module>Controller } from '../controllers/<module>Controller.js'

const router = Router()

router.get('/', (req, res) => <module>Controller.getList(req, res))
router.get('/:id', (req, res) => <module>Controller.getDetail(req, res))
router.post('/', (req, res) => <module>Controller.create(req, res))
router.put('/:id', (req, res) => <module>Controller.update(req, res))
router.delete('/:id', (req, res) => <module>Controller.delete(req, res))

export default router
```

路由文件只做委托，不含业务逻辑。

### Prisma Schema 模式

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model <Module> {
  id         Int      @id @default(autoincrement())
  <module>No String   @unique           // 业务编号
  // ... 业务字段
  createTime DateTime @default(now())
  updateTime DateTime @updatedAt
}
```

### 编号生成 (utils/generator.ts)

```ts
import dayjs from 'dayjs'

export function generateNo(type: string, sequence: number = 1): string {
  const prefixMap: Record<string, string> = { issue: 'AS', repair: 'RP', ... }
  const prefix = prefixMap[type] || 'XX'
  const date = dayjs().format('YYYYMMDD')
  const seq = String(sequence).padStart(4, '0')
  return `${prefix}${date}${seq}`
}
```

创建记录时先统计当日已有记录数，传入 sequence + 1。

## 前端代码规范

### API 请求封装 (utils/request.ts)

```ts
import axios from 'axios'
import { useAuthStore } from '@/stores/auth'
import router from '@/router'
import type { ApiResponse } from '@/types/api'

const instance = axios.create({
  baseURL: '/api/v1',
  timeout: 30000,
})

// 请求拦截：自动附加 Authorization 和 X-User-Name
instance.interceptors.request.use((config) => {
  const authStore = useAuthStore()
  if (authStore.token) {
    config.headers.Authorization = `Bearer ${authStore.token}`
  }
  if (authStore.userInfo?.username) {
    config.headers['X-User-Name'] = authStore.userInfo.username
  }
  return config
})

// 响应拦截：解包 ApiResponse，code !== 0 抛错，401 跳登录
instance.interceptors.response.use(
  (response) => {
    const data = response.data as ApiResponse
    if (data.code !== 0) {
      return Promise.reject(new Error(data.message || '请求失败'))
    }
    return data.data  // 直接返回 data 字段，调用方无需再 .data
  },
  (error) => {
    if (error.response?.status === 401) {
      const authStore = useAuthStore()
      authStore.logout()
      router.push('/login')
    }
    return Promise.reject(error)
  }
)

export default instance
```

关键：拦截器已解包，API 调用返回的直接是业务数据，不是 AxiosResponse。

### API 服务模块 (api/<module>.ts)

```ts
import request from '@/utils/request'
import type { PaginatedResponse, <Module>, <Module>QueryParams } from '@/types/api'

export const <module>Api = {
  getList(params: <Module>QueryParams) {
    return request.get<any, PaginatedResponse<<Module>>>('/<modules>', { params })
  },
  getDetail(id: number) {
    return request.get<any, <Module>>(`/<modules>/${id}`)
  },
  create(data: Partial<<Module>>) {
    return request.post<any, <Module>>('/<modules>', data)
  },
  update(id: number, data: Partial<<Module>>) {
    return request.put<any, <Module>>(`/<modules>/${id}`, data)
  },
  delete(id: number) {
    return request.delete<any, null>(`/<modules>/${id}`)
  },
}
```

### 类型定义 (types/api.ts)

```ts
// 通用分页参数
export interface PaginationParams {
  page?: number
  pageSize?: number
}

// 通用 API 响应
export interface ApiResponse<T = any> {
  code: number
  message: string
  data: T
}

// 分页响应
export interface PaginatedResponse<T> {
  list: T[]
  total: number
  page: number
  pageSize: number
}

// 业务实体接口
export interface <Module> {
  id: number
  <module>No: string
  // ... 字段
  createTime: string
  updateTime: string
}

// 查询参数
export interface <Module>QueryParams extends PaginationParams {
  keyword?: string
  // ... 筛选字段
}
```

### 认证 Store (stores/auth.ts)

```ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useAuthStore = defineStore('auth', () => {
  const token = ref(localStorage.getItem('<app>_token') || '')
  const userInfo = ref(JSON.parse(localStorage.getItem('<app>_userInfo') || 'null'))

  const isLoggedIn = computed(() => !!token.value)

  function setAuth(t: string, user: any) {
    token.value = t
    userInfo.value = user
    localStorage.setItem('<app>_token', t)
    localStorage.setItem('<app>_userInfo', JSON.stringify(user))
  }

  function logout() {
    token.value = ''
    userInfo.value = null
    localStorage.removeItem('<app>_token')
    localStorage.removeItem('<app>_userInfo')
  }

  return { token, userInfo, isLoggedIn, setAuth, logout }
})
```

### 路由定义 (router/index.ts)

```ts
import { createRouter, createWebHistory } from 'vue-router'
import { useAuthStore } from '@/stores/auth'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/login', component: () => import('@/views/login/Login.vue'), meta: { public: true } },
    {
      path: '/',
      component: () => import('@/layouts/MainLayout.vue'),
      children: [
        { path: '', redirect: '/dashboard' },
        { path: 'dashboard', component: () => import('@/views/dashboard/Dashboard.vue') },
        { path: '<module>', component: () => import('@/views/<module>/<Module>List.vue') },
        { path: '<module>/create', component: () => import('@/views/<module>/<Module>Form.vue') },
        { path: '<module>/:id', component: () => import('@/views/<module>/<Module>Detail.vue') },
        { path: '<module>/:id/edit', component: () => import('@/views/<module>/<Module>Form.vue') },
        // ... 更多模块
      ],
    },
  ],
})

// 路由守卫：未登录跳登录页，已登录访问登录页跳首页
router.beforeEach((to, _from, next) => {
  const authStore = useAuthStore()
  if (to.meta.public) {
    if (authStore.isLoggedIn && to.path === '/login') return next('/dashboard')
    return next()
  }
  if (!authStore.isLoggedIn) return next('/login')
  next()
})

export default router
```

所有业务路由是 `/` 的 children，使用 MainLayout 包裹。懒加载所有组件。

### 响应式布局 (layouts/MainLayout.vue)

PC 端：可收起侧边栏 + 顶部 header + 主内容区
H5 端：顶部 header + 底部 tab 栏 + 滑出式菜单

通过 `window.innerWidth < 768` 判断移动端，监听 resize 事件。

```vue
<script setup lang="ts">
const isMobile = ref(false)
const checkMobile = () => { isMobile.value = window.innerWidth < 768 }
onMounted(() => { checkMobile(); window.addEventListener('resize', checkMobile) })
onUnmounted(() => { window.removeEventListener('resize', checkMobile) })
</script>
```

PC 端侧边栏支持收起/展开，收起后只显示图标。

### 页面视图模式

每个业务模块三个文件：

**List.vue** — 列表页
- 搜索筛选栏（keyword + 下拉筛选）
- PC 端：表格 + 行内编辑（select 下拉直接修改状态）
- H5 端：卡片列表
- 分页组件 PaginationBar
- 删除确认弹窗 ConfirmDeleteDialog

**Form.vue** — 新增/编辑表单
- 通过 `route.params.id` 判断新增/编辑模式
- 表单验证
- 返回确认（未保存数据提醒）

**Detail.vue** — 详情页
- 数据展示
- 编辑/删除操作按钮

### 分页组件 (components/PaginationBar.vue)

通用分页组件，支持：
- 页码按钮（带省略号）
- 首页/上一页/下一页/末页
- 每页条数选择器（10/20/50/100）
- 显示总条数

### 确认弹窗 (composables/useConfirm.ts)

自定义确认弹窗替代浏览器原生 confirm()，居中显示：

```ts
import { createApp, h, ref } from 'vue'
import ConfirmDialog from '@/components/ConfirmDialog.vue'

export function confirm(message: string): Promise<boolean> {
  return new Promise((resolve) => {
    const container = document.createElement('div')
    document.body.appendChild(container)
    const visible = ref(true)
    const app = createApp({
      setup() {
        return () => h(ConfirmDialog, {
          visible: visible.value,
          message,
          onCancel: () => { visible.value = false; cleanup(); resolve(false) },
          onConfirm: () => { visible.value = false; cleanup(); resolve(true) },
        })
      },
    })
    const cleanup = () => { setTimeout(() => { app.unmount(); container.remove() }, 200) }
    app.mount(container)
  })
}
```

使用方式：`if (!await confirm('确定要删除吗？')) return`

### 样式系统 (styles/index.css)

使用 CSS 变量定义主题色，TailwindCSS v4 通过 `@tailwindcss/vite` 插件引入：

```css
@import "tailwindcss";

:root {
  --color-primary: #005D9C;
  --color-primary-light: #0EA5E9;
  --color-bg: #F8FAFC;
  --color-text-primary: #1E293B;
  --color-text-secondary: #64748B;
  --color-border: #E2E8F0;
  --color-danger: #EF4444;
  --color-success: #22C55E;
}
```

组件中使用 `style="color: var(--color-text-primary)"` 而非 Tailwind 色值类，保持主题一致性。

### 图标规范

使用 `lucide-vue-next`，禁止 emoji 或文字符号充当图标：

```vue
import { Search, Plus, Trash2, Edit } from 'lucide-vue-next'
<Search :size="16" />
```

## Docker 部署

### Dockerfile（三阶段构建）

1. **frontend-build**：构建前端 `npm ci && npm run build`
2. **backend-build**：构建后端 `npm ci && npx prisma generate && npm run build`
3. **backend**：生产镜像，只装生产依赖 + tsx，复制构建产物和前端 dist

```dockerfile
FROM node:22-slim AS backend
WORKDIR /app/backend
COPY --from=backend-build /build/backend/package*.json ./
RUN npm ci --omit=dev && npm install tsx@^4
COPY --from=backend-build /build/backend/node_modules/.prisma ./node_modules/.prisma
COPY --from=backend-build /build/backend/node_modules/prisma ./node_modules/prisma
COPY --from=backend-build /build/backend/node_modules/@prisma/engines ./node_modules/@prisma/engines
COPY --from=backend-build /build/backend/dist ./dist
COPY --from=backend-build /build/backend/prisma ./prisma
COPY --from=frontend-build /build/frontend/dist ../frontend/dist
RUN mkdir -p /app/data /app/uploads /app/logs
ENV NODE_ENV=production
ENV PORT=5174
EXPOSE 5174
CMD ["./docker-entrypoint.sh"]
```

注意：Prisma 相关目录必须完整复制（.prisma, prisma, @prisma/engines, @prisma/config）。

### docker-compose.yml

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: backend
    ports:
      - "5174:5174"
    volumes:
      - ./data:/app/data         # SQLite 数据库持久化
      - ./uploads:/app/uploads   # 上传文件持久化
      - ./logs:/app/logs         # 日志持久化
    environment:
      - NODE_ENV=production
      - DATABASE_URL=file:/app/data/app.db
      - PORT=5174
    restart: unless-stopped
```

### 数据脚本执行

在 Docker 容器内执行 seed 脚本（使用 tsx 运行 .ts 文件）：

```bash
docker exec <container> npx tsx scripts/seed-<module>.ts
```

脚本通过 Prisma Client 直接写入数据库，支持重复执行（去重逻辑）。

## 新增业务模块步骤

1. **Prisma Schema**：在 `backend/prisma/schema.prisma` 添加模型，运行 `npx prisma generate`
2. **后端控制器**：`backend/src/controllers/<module>Controller.ts`，实现 CRUD
3. **后端路由**：`backend/src/routes/<module>.ts`，挂载到 `main.ts`
4. **前端类型**：`frontend/src/types/api.ts`，添加实体和查询参数接口
5. **前端 API**：`frontend/src/api/<module>.ts`
6. **前端页面**：`frontend/src/views/<module>/` 下创建 List/Form/Detail
7. **前端路由**：`frontend/src/router/index.ts` 添加子路由
8. **菜单**：`MainLayout.vue` 的 menuGroups 和 mobileTabs 添加入口

## Vite 开发代理

```ts
// vite.config.ts
server: {
  port: 5173,
  proxy: {
    '/api': { target: 'http://localhost:8080', changeOrigin: true },
  },
}
```

开发时前端 5173 端口，后端 8080 端口，`/api` 请求代理到后端。生产环境由 Express 直接 serve 前端，无需代理。
