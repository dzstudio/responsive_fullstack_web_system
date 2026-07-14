# responsive_fullstack_web_system

一个 Claude Code 技能，用于快速构建**前后端一体响应式全栈 Web 管理系统**。

当用户说"开发一个响应式全栈Web系统"时，Claude Code 会按此技能定义的架构模式执行：

- **前端**：Vue 3 + TypeScript + Vite + TailwindCSS v4 + Pinia
- **后端**：Express.js + TypeScript（ESM）+ Prisma ORM + SQLite
- **部署**：Docker 多阶段构建 + docker-compose，单端口对外暴露
- **响应式**：同时适配 PC 和 H5 移动端

## 安装

```bash
npx skills add <your-github-username>/<your-repo>@responsive_fullstack_web_system
```

例如：

```bash
npx skills add dillon-zhang/responsive_fullstack_web_system@responsive_fullstack_web_system
```

## 使用

安装后，在 Claude Code 中说：

> "开发一个响应式全栈Web系统"

Claude 就会按本技能定义的架构、目录结构、代码规范和部署方式为你生成系统。

## 内容概览

本技能包含：

- 架构定义（前后端一体、SQLite 内置、响应式 PC/H5、单端口部署、Docker 支持）
- 完整代码规范
  - 后端：Express + Prisma + ESM，统一响应格式 `{ code, message, data }`，Controller 单例 + Router 委托模式
  - 前端：Vue 3 + Vite + TailwindCSS v4，Axios 拦截器解包，Pinia 认证 Store，响应式 MainLayout
  - 部署：三阶段 Dockerfile，Prisma 引擎完整复制，volume 持久化数据/上传/日志
- 新增模块清单
  - 从 Prisma Schema → 后端 CRUD → 前端 API/类型 → 页面视图 → 路由注册 → 菜单入口的完整 8 步流程

详见 [SKILL.md](./SKILL.md)。

## 发布到 skills.sh

本仓库符合 Claude Code 技能标准格式：

```
responsive_fullstack_web_system/
├── SKILL.md      # 技能定义（必需）
├── README.md     # 仓库说明
└── .gitignore    # Git 忽略规则
```

要发布到 [skills.sh](https://skills.sh/)，请：

1. 将本目录推送到 GitHub 仓库
2. 访问 https://skills.sh/ 提交你的仓库
3. 用户即可通过 `npx skills add <owner>/<repo>@responsive_fullstack_web_system` 安装

## License

MIT
