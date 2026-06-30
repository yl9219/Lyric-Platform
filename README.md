# Lyric Platform

歌词分享与管理平台后端 API。

## 技术栈

| 层 | 技术 |
|---|---|
| 框架 | NestJS 11 + TypeScript 5.7 |
| 数据库 | PostgreSQL 15 + Drizzle ORM |
| 缓存/队列 | Redis (ioredis) + BullMQ |
| 认证 | JWT 双 Token + Passport.js + bcrypt |
| 权限 | CASL (RBAC: ADMIN / MODERATOR / USER) |
| 文档 | Swagger / OpenAPI |
| 安全 | Helmet + express-rate-limit |
| 日志 | Pino (结构化 JSON) |
| CI/CD | GitHub Actions + Docker |

## 快速开始

```bash
# 安装依赖
pnpm install

# 启动 PostgreSQL + Redis
docker compose -f apps/api/docker-compose.yml up -d

# 配置环境变量
cp apps/api/.env.example apps/api/.env

# 推送数据库 Schema
pnpm --filter api db:push

# 插入种子数据（admin/moderator/user 账号）
pnpm --filter api seed

# 启动开发服务器
pnpm --filter api start:dev
```

- API: http://localhost:3000
- Swagger: http://localhost:3000/api/docs
- Drizzle Studio: `pnpm --filter api db:studio`

## 项目结构

```
lyric-platform/
├── apps/api/src/
│   ├── auth/           # 认证（JWT 双 token + 防暴力）
│   ├── users/          # 用户管理 + 封禁/解封
│   ├── roles/          # 角色查询（只读）
│   ├── songs/          # 歌曲 CRUD + 审核
│   ├── comments/       # 评论 CRUD + 审核
│   ├── collections/    # 收藏管理
│   ├── categories/     # 分类管理
│   ├── analyses/       # AI 歌词分析（BullMQ）
│   ├── admin-logs/     # 审计日志
│   ├── casl/           # CASL 权限工厂
│   ├── database/       # Drizzle ORM + Schema + Seed
│   ├── redis/          # Redis 服务 + 常量
│   └── common/         # 拦截器、过滤器、装饰器、DTO
├── Dockerfile          # 多阶段构建
├── .github/workflows/  # CI/CD
└── IMPLEMENTATION_PROMPT.md  # 完整项目规范
```

## API 端点概览（43 个）

| 模块 | 端点数 | 说明 |
|------|--------|------|
| Auth | 5 | 注册、登录、登出、刷新、改密码 |
| Users | 8 | 列表、详情、更新、删除、封禁、解封、改角色 |
| Roles | 2 | 角色列表、角色详情（含权限树） |
| Songs | 8 | 公开列表/详情、CRUD、审核通过/拒绝 |
| Categories | 5 | CRUD + 公开列表 |
| Comments | 7 | 公开列表/详情、CRUD、隐藏 |
| Collections | 4 | 列表、检查、收藏、取消 |
| Admin Logs | 1 | 审计日志列表 |
| Analyses | 2 | 获取分析结果、重试失败分析 |

## 测试

```bash
# 单元测试
pnpm --filter api test

# E2E 测试（需要真实 DB + Redis）
pnpm --filter api test:e2e
```

## Docker

```bash
# 构建镜像
docker build -t lyric-platform .

# 运行
docker run -p 3000:3000 --env-file apps/api/.env lyric-platform
```
