# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

VeloKit 是一个基于 Turborepo 的 monorepo 全栈应用脚手架，集成了 Next.js 15 (App Router)、Supabase Auth、Prisma ORM 和 shadcn/ui。适用于需要快速搭建具有用户认证、权限管理和现代化 UI 的 Web 应用。

## 常用命令

### 项目级别（根目录）

```bash
# 安装所有依赖
pnpm install

# 启动开发服务器（所有包）
pnpm dev

# 构建所有包
pnpm build

# 代码检查
pnpm lint

# 代码格式化
pnpm format
```

### 数据库操作（packages/db）

```bash
cd packages/db

# 生成 Prisma Client
pnpm db:generate

# 运行数据库迁移（开发环境）
pnpm db:migrate

# 部署迁移（生产环境）
pnpm db:deploy
```

### Web 应用（apps/web）

```bash
cd apps/web

# 启动开发服务器（Turbopack）
pnpm dev

# 构建生产版本
pnpm build

# 启动生产服务器
pnpm start

# 类型检查
pnpm typecheck

# 代码检查
pnpm lint

# 添加 shadcn/ui 组件
pnpm dlx shadcn@latest add [组件名]
```

## 项目架构

### Monorepo 结构

```
velokit/
├── apps/
│   └── web/                 # 主 Web 应用（Next.js 15 + App Router）
└── packages/
    ├── db/                  # 数据库层（Prisma + Supabase）
    ├── ui/                  # 共享 UI 组件库（shadcn/ui + Tailwind CSS v4）
    ├── eslint-config/       # 共享 ESLint 配置
    └── typescript-config/   # 共享 TypeScript 配置
```

### apps/web 核心目录

```
apps/web/
├── app/                     # Next.js App Router
│   ├── actions/            # Server Actions（表单提交等服务端操作）
│   ├── api/                # API 路由
│   ├── login/              # 登录页面
│   ├── signup/             # 注册页面
│   ├── forgot-password/    # 忘记密码
│   └── private/            # 受保护页面（需认证）
├── lib/
│   ├── dal/                # Data Access Layer（数据访问层）
│   ├── definitions/        # Zod 验证 schema（表单验证）
│   └── dto/                # Data Transfer Objects
├── modules/                # 功能模块（landing, users）
├── components/             # 应用特定组件
├── utils/                  # 工具函数（Supabase 客户端等）
└── types/                  # TypeScript 类型定义
```

### 关键架构模式

**1. Data Access Layer (DAL)**
- 位置：`apps/web/lib/dal/`
- 作用：封装所有数据库操作，使用 Supabase Client
- 示例：`user.ts` 提供 `getUser`、`createUser` 等函数
- 注意：使用 `cache` from React 进行查询缓存

**2. Server Actions**
- 位置：`apps/web/app/actions/`
- 作用：处理表单提交，调用 DAL 进行数据操作
- 验证：使用 Zod schemas（`lib/definitions/`）进行表单验证

**3. Supabase 集成**
- 认证：使用 `@supabase/ssr` 的 `createServerClient`
- 客户端创建：`utils/supabase/server.ts`（服务端）、`utils/supabase/client.ts`（客户端）
- Session 验证：`lib/dal/verifySession.ts`

**4. Prisma + Supabase 混合使用**
- Prisma schema：`packages/db/prisma/schema.prisma`
- Supabase 迁移：`packages/db/supabase/`（包含 RLS 策略、触发器）
- Prisma Client 输出：`packages/db/generated/prisma`（无 Rust 引擎）

**5. UI 组件组织**
- 共享组件：`packages/ui/src/components/`（通过 `@workspace/ui/components/*` 导入）
- 应用组件：`apps/web/components/`（应用特定的业务组件）

## 环境配置

在 `apps/web/.env.local` 中配置：

```bash
NEXT_PUBLIC_SUPABASE_URL=your-project-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

参考 `apps/web/.env.example`。

## Turborepo 依赖关系

根据 `turbo.json`：
- `build` 任务依赖：`^build` 和 `^db:generate`
- `dev` 任务依赖：`^db:generate`
- `db:generate` 任务不缓存（Prisma 生成）

这意味着：
1. 构建前会自动生成 Prisma Client
2. 启动开发服务器前会确保数据库 schema 最新

## 技术栈版本

- Next.js: 16.0.1（App Router）
- React: 19.2.0
- Node: >=22.21.1
- pnpm: 10.22.0
- TypeScript: 5.9.3
- Tailwind CSS: 4.1.17
- Prisma: 6.17.1
- Supabase JS: 2.81.1

## 开发注意事项

**添加新的 shadcn/ui 组件**
```bash
cd apps/web
pnpm dlx shadcn@latest add [组件名]
```
组件会自动添加到 `apps/web/components/`。

**数据库 schema 更改**
1. 修改 `packages/db/prisma/schema.prisma`
2. 运行 `pnpm db:migrate` 创建迁移
3. Prisma Client 会自动重新生成

**认证流程**
- 用户注册/登录通过 `app/actions/auth/` 处理
- Session 验证通过 `lib/dal/verifySession.ts`
- 受保护页面检查 session 后才渲染

**类型导入**
- Supabase 类型：`@/types/model`（从 `packages/db` 生成）
- 共享 UI：`@workspace/ui/components/*`
- 数据库：`@workspace/db`
