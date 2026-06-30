# Lyric Platform API - 完整项目规范

> **用途**：这是一份自包含的项目规范文档。任何 AI 模型都可以基于此文档生成一致的代码。
> **工作方式**：按阶段逐步执行。每完成一个模块，等待用户确认后再继续下一个。

---

## 使用说明

### 给 AI 的指令

你是一个后端工程师，正在从零构建一个歌词分享平台的 API。严格遵循本文档中的所有规范、代码模式和文件结构。

**工作流程**：
1. 按阶段顺序执行（第 1 阶段 → 第 5 阶段）
2. 每个阶段内按模块逐个实现
3. 每完成一个模块，输出该模块的所有文件，等待用户确认
4. 用户确认后再开始下一个模块
5. 不要跳步、不要合并模块、不要提前实现后续阶段的功能

**代码质量要求**：
- 每个文件必须能通过严格的 ESLint 检查（no-explicit-any: error）
- 每个文件必须有清晰的中文注释说明职责
- 遵循本文档中的所有代码模式，不要发明新模式

---

## 一、技术栈

| 层 | 技术 | 版本 |
|---|---|---|
| 框架 | NestJS | ^11.0.1 |
| 语言 | TypeScript | ^5.7.3 |
| ORM | Drizzle ORM | ^0.45.1 |
| 数据库 | PostgreSQL | 15 |
| 缓存 | Redis (ioredis) | ^5.9.1 |
| 认证 | Passport + JWT | ^11.0.5 / ^11.0.2 |
| 权限 | CASL | ^6.8.0 |
| 校验 | class-validator + class-transformer | ^0.14.3 / ^0.5.1 |
| 文档 | Swagger | ^11.2.4 |
| 安全 | Helmet + express-rate-limit | ^8.1.0 / ^8.2.1 |
| 日志 | nestjs-pino + pino-pretty | ^4.0.0 / ^13.0.0 |
| 密码 | bcrypt | ^6.0.0 |
| 环境校验 | Joi | ^17.12.0 |
| 包管理 | pnpm | 9.0.0 |
| Monorepo | Turborepo | ^2.7.2 |
| 运行时 | Node.js | >=18 |

---

## 二、项目结构

```
lyric-platform/                          # monorepo 根目录
├── turbo.json
├── pnpm-workspace.yaml
├── package.json
└── apps/
    └── api/                             # 本项目
        ├── .env.example
        ├── docker-compose.yml
        ├── drizzle.config.ts
        ├── eslint.config.mjs
        ├── nest-cli.json
        ├── package.json
        ├── tsconfig.json
        ├── tsconfig.build.json
        └── src/
            ├── main.ts
            ├── app.module.ts
            ├── app.controller.ts
            │
            ├── @types/                  # TypeScript 类型扩展
            │   └── express/
            │       └── index.d.ts     # Express Request 扩展（添加 requestId 字段）
            │
            ├── config/                  # 环境配置
            │   └── env.validation.ts
            │
            ├── database/               # 数据库
            │   ├── database.module.ts
            │   ├── database.service.ts
            │   ├── schema.ts           # 12 张表 + 7 个枚举，单文件
            │   └── seed.ts
            │
            ├── redis/                   # Redis
            │   ├── redis.module.ts
            │   ├── redis.service.ts
            │   └── redis.constants.ts
            │
            ├── common/                  # 公共层
            │   ├── filters/
            │   │   └── all-exceptions.filter.ts
            │   ├── interceptors/
            │   │   └── transform.interceptor.ts
            │   ├── middleware/
            │   │   └── request-id.middleware.ts
            │   ├── dto/
            │   │   └── pagination-query.dto.ts
            │   ├── decorators/
            │   │   ├── index.ts
            │   │   ├── public.decorator.ts
            │   │   ├── current-user.decorator.ts
            │   │   └── roles.decorator.ts
            │   ├── interfaces/
            │   │   ├── response.interface.ts
            │   │   └── pagination.interface.ts
            │   └── utils/
            │       ├── slug.util.ts
            │       └── hash.util.ts
            │
            ├── auth/                    # 认证
            │   ├── auth.module.ts
            │   ├── auth.controller.ts
            │   ├── auth.service.ts
            │   ├── strategies/
            │   │   └── jwt.strategy.ts
            │   ├── guards/
            │   │   ├── jwt-auth.guard.ts
            │   │   ├── user-status.guard.ts
            │   │   └── roles.guard.ts
            │   ├── dto/
            │   │   ├── register.dto.ts
            │   │   ├── login.dto.ts
            │   │   ├── refresh-token.dto.ts
            │   │   └── change-password.dto.ts
            │   ├── interfaces/
            │   │   ├── jwt-payload.interface.ts
            │   │   ├── login-response.interface.ts
            │   │   └── authenticated-user.interface.ts
            │   └── utils/
            │       └── ensure-user-active.util.ts
            │
            ├── users/                   # 用户
            ├── roles/                   # 角色（只读展示，GET /roles）
            ├── casl/                    # 资源归属检查
            ├── songs/                   # 歌曲
            ├── categories/              # 分类（新）
            ├── comments/                # 评论
            ├── collections/             # 收藏
            ├── admin-logs/              # 审计日志（新）
            └── analyses/                # AI 分析（新）
```

每个业务模块的内部结构统一为：
```
模块名/
├── 模块名.module.ts
├── 模块名.controller.ts         # 公开端点（@Public 或需登录）
├── admin-模块名.controller.ts   # 管理员端点（仅有 admin/* 路由的模块才有此文件）
├── 模块名.service.ts
├── dto/
│   ├── create-模块名.dto.ts
│   ├── update-模块名.dto.ts
│   ├── query-模块名.dto.ts      # 公开列表查询（不含 status 过滤）
│   └── query-admin-模块名.dto.ts # 管理员列表查询（含 status 过滤，仅有 admin/* 路由的模块才有）
│   # 注：songs 模块额外有 reject-song.dto.ts（审核拒绝必须提供 reason 字段）
└── interfaces/
    └── 模块名.interface.ts
```

> **admin controller 设计原则**：公开端点和管理员端点是两个不同的业务场景，不共用同一个路由。
> 公开接口永远返回确定状态（APPROVED / VISIBLE），管理员接口可按 status 过滤任意状态。
> 当前有 admin controller 的模块：songs（`/admin/songs`）、comments（`/admin/comments`）。

---

## 三、编码规范

### 3.1 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 文件名 | kebab-case | `create-song.dto.ts` |
| 类名 | PascalCase | `CreateSongDto` |
| 变量/函数 | camelCase | `findBySlug` |
| 常量 | UPPER_SNAKE_CASE | `SAFE_USER_SELECT` |
| 数据库表名 | snake_case | `admin_logs` |
| 数据库列名 | snake_case | `created_at` |
| API 路径 | kebab-case | `/admin-logs` |
| 枚举值 | UPPER_SNAKE_CASE | `APPROVED` |
| Redis Key | 冒号分隔 | `user:session:{userId}` |

### 3.2 注释规范

- 每个文件顶部：一句话说明职责
- 每个 public 方法：JSDoc 说明参数和返回值
- 复杂逻辑：行内注释说明"为什么"而不是"做什么"
- 中文注释

### 3.3 错误处理

- 使用 NestJS 内置异常：`NotFoundException`, `ConflictException`, `ForbiddenException`, `UnauthorizedException`
- 不要捕获异常后静默吞掉
- 数据库唯一约束冲突由全局 `AllExceptionsFilter` 统一处理
- 业务层预检查 + 数据库约束兜底（双层防护）

### 3.4 import 顺序

```typescript
// 1. NestJS 核心
import { Injectable, NotFoundException } from '@nestjs/common';
// 2. 第三方库
import { eq, and, desc, count, sql } from 'drizzle-orm';
// 3. 内部模块（按层级从外到内）
import { DatabaseService } from '@database/database.service';
import { songs, users } from '@database/schema';
// 4. 同模块文件
import { CreateSongDto } from './dto/create-song.dto';
import { SongListResponse } from './interfaces/song.interface';
```

---

## 四、代码模式（所有模块必须严格遵循）

### 模式 1：Schema 定义

所有表和枚举定义在单文件 `src/database/schema.ts` 中，按"用户体系 → 内容体系 → 辅助体系"分区注释。

```typescript
// src/database/schema.ts
import { pgTable, pgEnum, uuid, text, integer, boolean, timestamp, jsonb, unique, primaryKey, index } from 'drizzle-orm/pg-core';

// ==================== 枚举 ====================
export const userStatusEnum = pgEnum('user_status', ['ACTIVE', 'BANNED', 'DELETED']);
export const songStatusEnum = pgEnum('song_status', ['PENDING', 'APPROVED', 'REJECTED', 'DELETED']);
// ...

// ==================== 用户体系 ====================
export const users = pgTable('users', { /* ... */ });
export const roles = pgTable('roles', { /* ... */ });
// ...

// ==================== 内容体系 ====================
export const songs = pgTable('songs', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull().references(() => users.id, { onDelete: 'restrict' }),
  // ...
}, (table) => ({
  userIdIdx: index('songs_user_id_idx').on(table.userId),
  statusCreatedAtIdx: index('songs_status_created_at_idx').on(table.status, table.createdAt),
}));
// ...

// ==================== 辅助体系 ====================
export const analyses = pgTable('analyses', { /* ... */ });
export const adminLogs = pgTable('admin_logs', { /* ... */ });
```

### 模式 2：字段选择器常量

列表查询排除大字段（如 lyrics），使用 `as const` 确保类型推断。

```typescript
const SONG_LIST_SELECT = {
  id: songs.id,
  userId: songs.userId,
  slug: songs.slug,
  title: songs.title,
  artist: songs.artist,
  // ... 不包含 lyrics
  uploaderId: users.id,
  uploaderUsername: users.username,
  uploaderAvatar: users.avatar,
} as const;
```

### 模式 3：Service 查询

```typescript
@Injectable()
export class SongsService {
  constructor(private readonly database: DatabaseService) {}

  // 列表查询：并行 data + count，分页，条件构建
  async findAll(queryDto: QuerySongsDto): Promise<SongListResponse> {
    const db = this.database.db;
    const { keyword, status } = queryDto;
    const page = queryDto.page ?? 1;
    const pageSize = queryDto.pageSize ?? 20;
    const offset = (page - 1) * pageSize;

    const conditions: SQL[] = [];
    if (status) conditions.push(eq(songs.status, status));
    if (keyword) {
      conditions.push(or(
        like(songs.title, `%${keyword}%`),
        like(songs.artist, `%${keyword}%`),
      )!);
    }
    const whereClause = conditions.length > 0 ? and(...conditions) : undefined;

    const [items, [{ total }]] = await Promise.all([
      db.select(SONG_LIST_SELECT).from(songs)
        .leftJoin(users, eq(songs.userId, users.id))
        .where(whereClause)
        .orderBy(desc(songs.createdAt))
        .limit(pageSize).offset(offset),
      db.select({ total: count() }).from(songs).where(whereClause),
    ]);

    const totalPages = Math.ceil(total / pageSize);
    return {
      items: items.map(item => this.mapToSongWithUploader(item)),
      total, page, pageSize, totalPages,
      hasNext: page < totalPages,
      hasPrev: page > 1,
    };
  }

  // 详情查询：单条 + NOT FOUND 异常
  async findBySlug(slug: string): Promise<SongWithUploader> {
    const db = this.database.db;
    const [song] = await db.select({ ...SONG_LIST_SELECT, lyrics: songs.lyrics })
      .from(songs).leftJoin(users, eq(songs.userId, users.id))
      .where(eq(songs.slug, slug)).limit(1);
    if (!song) throw new NotFoundException('歌曲不存在');
    return this.mapToSongWithUploader(song);
  }

  // 映射函数：平铺 JOIN 结果 → 嵌套对象
  private mapToSongWithUploader(song: Record<string, unknown>): SongWithUploader {
    return {
      id: song.id, /* ... */
      uploader: {
        id: song.uploaderId ?? 'deleted-user',
        username: song.uploaderUsername ?? '已删除用户',
        avatar: song.uploaderAvatar,
      },
    };
  }
}
```

### 模式 4：事务 + 原子计数器

所有涉及多表修改的操作必须用事务。计数器用 SQL 原子操作，减法用 `GREATEST(..., 0)` 防负数。

```typescript
async create(userId: string, dto: CreateSongDto) {
  return this.database.db.transaction(async (tx) => {
    const [newSong] = await tx.insert(songs).values({ userId, ...dto }).returning();
    await tx.update(users).set({
      contributionCount: sql`${users.contributionCount} + 1`,
    }).where(eq(users.id, userId));
    return newSong;
  });
}

async remove(id: string) {
  return this.database.db.transaction(async (tx) => {
    const [deleted] = await tx.update(songs).set({ status: 'DELETED', updatedAt: new Date() })
      .where(and(eq(songs.id, id), ne(songs.status, 'DELETED')))  // 乐观锁
      .returning({ userId: songs.userId });
    if (!deleted) throw new NotFoundException('歌曲不存在或已被删除');

    // 级联软删除评论
    await tx.update(comments).set({ status: 'DELETED', updatedAt: new Date() })
      .where(eq(comments.songId, id));

    // 原子计数器 - 减法防负数
    await tx.update(users).set({
      contributionCount: sql`GREATEST(${users.contributionCount} - 1, 0)`,
    }).where(eq(users.id, deleted.userId));

    return { message: '歌曲删除成功' };
  });
}
```

### 模式 5：DTO 校验

所有文本字段必须有 `@MaxLength`，数组必须有 `@ArrayMaxSize`，URL 必须有 `@IsUrl`。

```typescript
export class CreateSongDto {
  @ApiProperty({ description: '歌曲标题', example: '晴天' })
  @IsString()
  @IsNotEmpty()
  @MaxLength(200)
  title!: string;

  @ApiPropertyOptional({ description: '标签', example: ['流行', '华语'] })
  @IsOptional()
  @IsArray()
  @ArrayMaxSize(10)
  @IsString({ each: true })
  @MaxLength(30, { each: true })
  tags?: string[];

  @ApiPropertyOptional({ description: '封面图片URL' })
  @IsOptional()
  @IsUrl()
  coverImage?: string;
}
```

### 模式 6：分页基类

所有列表查询的 QueryDto 继承此基类。

```typescript
// src/common/dto/pagination-query.dto.ts
export class PaginationQueryDto {
  @ApiPropertyOptional({ default: 1, minimum: 1 })
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @ApiPropertyOptional({ default: 20, minimum: 1, maximum: 100 })
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  pageSize?: number = 20;
}
```

### 模式 7：统一响应

成功：
```json
{ "code": 200, "message": "success", "data": { ... }, "timestamp": "...", "requestId": "..." }
```

错误：
```json
{ "code": 404, "message": "歌曲不存在", "data": null, "timestamp": "...", "requestId": "..." }
```

### 模式 8：Guard 链

```
请求 → JwtAuthGuard（@Public 跳过）
     → UserStatusGuard（检查 ACTIVE）
     → RolesGuard（检查 @Roles）
     → Controller（CASL 检查资源归属）
```

### 模式 9：权限装饰器

```typescript
// 公开路由
@Public()
@Get()
findAll() {}

// 需要特定角色
@Roles(Role.MODERATOR, Role.ADMIN)
@Patch(':id/approve')
approve() {}

// 需要登录但无需特定角色（默认行为，不加装饰器）
@Post()
create() {}
```

### 模式 10：CASL Action 枚举 + 资源归属检查

**Action 枚举定义**（`src/casl/casl-ability.factory.ts`）：

```typescript
export enum Action {
  Manage  = 'manage',   // 超级权限，匹配所有操作（仅 ADMIN）
  Create  = 'create',
  Read    = 'read',
  Update  = 'update',
  Delete  = 'delete',
  Approve = 'approve',  // 审核通过 / 审核拒绝（ADMIN + MODERATOR）
}
```

**权限分层**：
- `ADMIN`：`can(Action.Manage, 'all')` — 无限制
- `MODERATOR`：`Approve` + `Read/Create` + `Update/Delete`（仅自己资源）
- `USER`：`Create/Read` + `Update/Delete`（仅自己资源，通过 `{ userId: user.id }` 条件）

在 Controller 中，查出资源后用 CASL 判断归属权：

```typescript
@Patch(':id')
async update(@Param('id') id: string, @Body() dto: UpdateSongDto, @Request() req) {
  const song = await this.songsService.findOne(id);
  const ability = this.caslAbilityFactory.createForUser(req.user);
  if (!ability.can(Action.Update, song)) {
    throw new ForbiddenException('权限不足');
  }
  return this.songsService.update(id, dto);
}
```

---

## 五、数据库设计

### 5.1 枚举

```
user_status:     ACTIVE | BANNED | DELETED
song_status:     PENDING | APPROVED | REJECTED | DELETED
comment_status:  PENDING | VISIBLE | HIDDEN | DELETED
analysis_status: PENDING | PROCESSING | COMPLETED | FAILED
permission_type: MENU | BUTTON
admin_action:    APPROVE_SONG | REJECT_SONG | BAN_USER | UNBAN_USER | HIDE_COMMENT | DELETE_COMMENT
target_type:     SONG | USER | COMMENT
```

### 5.2 表定义（14 张）

#### users
| 列 | 类型 | 约束 |
|---|---|---|
| id | UUID | PK, defaultRandom |
| email | text | NOT NULL, UNIQUE |
| username | text | NOT NULL, UNIQUE |
| password | text | NOT NULL |
| avatar | text | nullable |
| bio | text | nullable |
| status | user_status | NOT NULL, default ACTIVE |
| contribution_count | integer | NOT NULL, default 0 |
| collection_count | integer | NOT NULL, default 0 |
| created_at | timestamptz | NOT NULL, defaultNow |
| updated_at | timestamptz | NOT NULL, defaultNow |

索引：`users_status_idx`

#### roles
| 列 | 类型 | 约束 |
|---|---|---|
| id | UUID | PK |
| code | text | NOT NULL, UNIQUE |
| name | text | NOT NULL |
| description | text | nullable |
| is_system | boolean | NOT NULL, default false |
| created_at / updated_at | timestamptz | NOT NULL |

索引：`roles_code_idx`

#### permissions
> **不驱动运行时授权**。此表仅存储权限条目供前端展示，授权由 `@Roles()` + CASL 控制（见 6.2 节）。

| 列 | 类型 | 约束 |
|---|---|---|
| id | UUID | PK |
| code | text | NOT NULL, UNIQUE |
| name | text | NOT NULL |
| module | text | NOT NULL |
| type | permission_type | NOT NULL |
| parent_id | UUID | FK → permissions.id, CASCADE |
| path | text | nullable |
| icon | text | nullable |
| sort | integer | NOT NULL, default 0 |
| is_system | boolean | NOT NULL, default false |
| created_at / updated_at | timestamptz | NOT NULL |

索引：`permissions_code_idx`, `permissions_module_type_idx`, `permissions_parent_id_idx`

#### user_roles
> 登录时查询此表获取用户角色，写入 JWT payload。当前业务下每个用户只持有一个角色，多对多结构为未来独立多角色场景预留（无需加表加迁移即可支持）。

| 列 | 类型 | 约束 |
|---|---|---|
| id | UUID | PK |
| user_id | UUID | FK → users.id, CASCADE |
| role_id | UUID | FK → roles.id, CASCADE |
| created_at | timestamptz | NOT NULL |

UNIQUE(user_id, role_id)。索引：`user_roles_user_id_idx`, `user_roles_role_id_idx`

#### role_permissions
> **不驱动运行时授权**。此表仅记录角色与权限的关联供前端展示，授权由 `@Roles()` + CASL 控制（见 6.2 节）。

同 user_roles 模式。UNIQUE(role_id, permission_id)。

#### songs
见模式 1 中的完整定义。

**onDelete 策略**：
- `songs.userId` → `restrict`（用户不可删除如果有歌曲）
- `songs.auditedBy` → `set null`（审核人删除不影响歌曲）

#### categories
> `slug` 为语义化标识符（如 `pop`、`rock`），由管理员创建时显式提供。API 路由用 `id` 不用 `slug`，slug 供前端渲染样式标识（如 CSS class）或未来 SEO 路由扩展用。

| 列 | 类型 | 约束 |
|---|---|---|
| id | UUID | PK |
| name | text | NOT NULL |
| slug | text | NOT NULL, UNIQUE（格式：小写字母、数字、连字符） |
| description | text | nullable |
| order | integer | NOT NULL, default 0 |
| is_visible | boolean | NOT NULL, default true |
| created_at / updated_at | timestamptz | NOT NULL |

#### song_categories
| 列 | 类型 | 约束 |
|---|---|---|
| song_id | UUID | FK → songs.id, CASCADE |
| category_id | UUID | FK → categories.id, RESTRICT |
| created_at | timestamptz | NOT NULL |

PK(song_id, category_id)

#### collections
| 列 | 类型 | 约束 |
|---|---|---|
| id | UUID | PK |
| user_id | UUID | FK → users.id, CASCADE |
| song_id | UUID | FK → songs.id, CASCADE |
| created_at | timestamptz | NOT NULL |

UNIQUE(user_id, song_id)

#### comments
| 列 | 类型 | 约束 |
|---|---|---|
| id | UUID | PK |
| song_id | UUID | FK → songs.id, **CASCADE**（从旧版 restrict 修改） |
| user_id | UUID | FK → users.id, RESTRICT |
| content | text | NOT NULL |
| status | comment_status | NOT NULL, default VISIBLE |
| created_at / updated_at | timestamptz | NOT NULL |

索引：`comments_song_id_status_idx`, `comments_user_id_idx`, `comments_created_at_idx`

#### analyses
| 列 | 类型 | 约束 |
|---|---|---|
| id | UUID | PK |
| song_id | UUID | FK → songs.id, CASCADE |
| sentiment | jsonb | NOT NULL |
| themes | text[] | NOT NULL, default [] |
| quality | jsonb | NOT NULL |
| summary | text | NOT NULL |
| status | analysis_status | NOT NULL, default PENDING |
| error_message | text | nullable |
| created_at / updated_at | timestamptz | NOT NULL |

#### admin_logs
| 列 | 类型 | 约束 |
|---|---|---|
| id | UUID | PK |
| admin_id | UUID | FK → users.id, RESTRICT |
| action | admin_action | NOT NULL |
| target_type | target_type | NOT NULL |
| target_id | UUID | NOT NULL |
| reason | text | nullable |
| metadata | jsonb | nullable |
| created_at | timestamptz | NOT NULL |

索引：`admin_logs_admin_id_idx`, `admin_logs_target_type_id_idx`, `admin_logs_created_at_idx`

---

## 六、RBAC 权限设计

### 6.1 运行时权限查询流程

```
JWT Access Token 包含 { sub: userId, roles: string[] }
（登录时写入，角色固定，无需每次请求查权限表）

每次认证请求：
1. JwtStrategy.validate(payload):
   - DB 查询用户基本信息（仅查 users 表，获取 status/username/email）
   - 结果: { id, username, email, status, roles: string[] }
   - 返回 AuthenticatedUser 对象
   （无 Redis 权限缓存，无内存映射，零额外转换）

2. RolesGuard:
   - 读 @Roles() 装饰器
   - 检查 user.roles.some(r => requiredRoles.includes(r))
   - 无装饰器 → 放行

角色枚举：
  export enum Role {
    ADMIN     = 'ADMIN',
    MODERATOR = 'MODERATOR',
    USER      = 'USER',
  }

权限变更说明：
  调整角色访问控制需修改对应 @Roles() 装饰器并重新部署
```

### 6.2 授权方案选择与决策记录

> **给 AI 的说明**：本节是设计决策记录，不是实现指令。阅读后你将理解：
> 本项目选了哪种授权方案、为什么选它、以及在什么业务场景下应该换成另一种方案。

---

#### 两种授权方案对比

| 维度 | 方案 A：DB 驱动权限 | 方案 B：角色枚举 + CASL（**本项目采用**）|
|------|--------------------|-----------------------------------------|
| 适用系统类型 | 运营后台、内部管理系统 | 用户产品、社区平台 |
| 典型场景 | CMS、ERP、OA、多租户 SaaS | 内容社区、音乐平台、论坛、博客 |
| 角色特征 | 动态创建，随组织架构变化，数量多 | 固定 3-5 个，代表产品中的用户身份 |
| 权限变更方式 | 管理员在 UI 配置，无需部署 | 修改 `@Roles()` 装饰器，重新部署 |
| 运行时开销 | 每次请求查 permissions 表（或 Redis 缓存） | JWT 携带 roles，无额外 DB 查询 |
| 实现复杂度 | 高：权限表、中间表、缓存、同步 | 低：装饰器 + CASL 工厂函数 |

---

#### 本项目选择方案 B 的理由

歌词分享平台是典型的**用户产品**：

- 角色只有 3 个（USER / MODERATOR / ADMIN），是产品概念而非组织配置
- "版主可以审核歌曲"是产品规则，不需要非技术人员通过 UI 修改
- 无多租户、无动态权限分配、无中间层角色的需求

**实际生效的授权规则**（完全由代码中的 `@Roles()` 装饰器 + CASL 控制）：

| 端点范围 | ADMIN | MODERATOR | USER（已登录） | 未登录 |
|---------|:-----:|:---------:|:-------------:|:------:|
| `@Public()` 标记的端点 | ✅ | ✅ | ✅ | ✅ |
| 仅需登录的端点 | ✅ | ✅ | ✅ | ❌ |
| `GET /users`（用户列表） | ✅ | ✅ | ❌ | ❌ |
| `PATCH /users/:id`（改自己信息） | ✅ | ✅ | ✅（仅自己） | ❌ |
| `PATCH /users/:id/role` | ✅ | ❌ | ❌ | ❌ |
| `DELETE /users/:id` / ban / unban | ✅ | ❌ | ❌ | ❌ |
| 创建歌曲 / 评论 / 收藏 | ✅ | ✅ | ✅ | ❌ |
| 修改 / 删除自己的资源 | ✅ | ✅ | ✅（仅自己） | ❌ |
| `GET /admin/songs`（审核队列） | ✅ | ✅ | ❌ | ❌ |
| `PATCH /songs/:id/approve` / reject | ✅ | ✅ | ❌ | ❌ |
| `POST/PATCH/DELETE /categories` | ✅ | ❌ | ❌ | ❌ |
| `GET /admin/comments`（审核队列） | ✅ | ✅ | ❌ | ❌ |
| `PATCH /comments/:id/hide` | ✅ | ✅ | ❌ | ❌ |
| `GET /admin-logs` | ✅ | ✅ | ❌ | ❌ |
| `POST /analyses/:id/retry` | ✅ | ❌ | ❌ | ❌ |
| `GET /roles` | ✅ | ❌ | ❌ | ❌ |

> **CASL 归属检查**：USER / MODERATOR 修改或删除资源时，只能操作自己创建的内容（`{ userId: user.id }`）。ADMIN 不受此限制（`can(Action.Manage, 'all')`）。

---

#### permissions / role_permissions 表的定位

Schema 中保留了 `permissions` 和 `role_permissions` 两张表，种子数据写入了权限条目与角色-权限关联。

**这些数据不参与运行时授权**，其保留价值有两点：

1. **管理后台展示**：`GET /roles/:id` 可以返回"该角色拥有哪些权限"，供管理后台 UI 渲染权限树
2. **升级路径预留**：若业务扩张需要从方案 B 切换到方案 A，基础设施已就位，不需要加表加迁移

```
<!--
  以下权限数据为展示用途，不驱动运行时授权。
  AI 复刻时：如果目标系统是用户产品（社区/平台），按当前方案 B 实现即可，此列表仅供参考。
  如果目标系统是运营后台/多租户 SaaS（方案 A），需要：
    1. 将此权限列表改为运行时查询依据（JwtStrategy 查 permissions 表）
    2. 去掉固定的 Role 枚举和 @Roles() 装饰器
    3. RolesGuard 改为匹配 permission code 而非 role code
    4. 引入 Redis 缓存权限结果（建议 TTL 5 分钟）
-->

权限列表（22 个，仅展示用）：
users:list, users:view, users:update, users:delete, users:role
roles:list
permissions:list
songs:list, songs:view, songs:create, songs:update, songs:delete, songs:approve
categories:manage
comments:create, comments:update, comments:delete, comments:moderate
collections:create, collections:delete
admin-logs:list
analyses:manage

角色-权限对应（仅展示用，与 @Roles() 装饰器保持一致）：
ADMIN（22 权限）：全部
MODERATOR（10 权限）：users:list,
                      songs:list, songs:view, songs:create, songs:approve,
                      comments:create, comments:moderate,
                      collections:create, collections:delete, admin-logs:list
USER（6 权限）：songs:list, songs:view, songs:create,
               comments:create, collections:create, collections:delete
```

### 6.3 Redis 使用范围

权限不走 Redis。Redis 仅用于以下场景：

| Key 格式 | 类型 | 用途 | TTL |
|---------|------|------|-----|
| `refresh_token:{userId}` | String | Refresh Token JTI 防重放 | 7 天 |
| `login:attempts:{username}` | String | 登录防暴力计数 | 900 秒 |
| `user:session:{userId}` | String (JSON) | 用户信息缓存（第 5 阶段） | 300~360 秒 |
| `user:banned` | Set | 封禁用户黑名单（第 5 阶段） | 永久 |

---

**第 5 阶段 Redis 缓存方案（完整设计）**

**背景**：当前 `JwtStrategy.validate()` 每次认证请求都查一次 DB，获取 `id / username / email / status`。status 用于封禁检查，其余字段挂载到 `request.user`。低流量下完全合理，高并发时是不必要的 IO 放大。

**核心设计：两个关注点彻底分离**

```
user:session:{userId}  →  "这个用户是谁"（id/username/email）
user:banned Set        →  "这个用户被封禁了吗"
```

**user:session 策略**：
- 缓存内容：`{ id, username, email }`，**不含 status**（status 交给黑名单管理）
- TTL：`300 + Math.floor(Math.random() * 60)` 秒，随机抖动防止缓存雪崩
- 加载策略：Lazy Loading（首次请求 miss 时写入，这是正常冷启动，不是问题）
- **不需要主动删除**：user:session 不含 status，ban/unban 操作不影响它，TTL 到期自然刷新

**user:banned 黑名单策略**：
- 类型：Redis Set，支持 O(1) 的 SISMEMBER 查询
- `ban(userId)` → `SADD user:banned {userId}`（即时生效）
- `unban(userId)` → `SREM user:banned {userId}`（即时生效）
- `remove(userId)` → `SADD user:banned {userId}`（软删除同样拉黑）

**第 5 阶段认证请求完整流程**：

```
携带 token 的请求
  → 验证 JWT 签名 + 过期时间
  → SISMEMBER user:banned {userId}     ← O(1)，即时封禁检查
      是 → 403 Forbidden
      否 → GET user:session:{userId}
              命中 → 直接返回用户信息
              未命中 → 查 DB → 写 Redis（TTL 随机）→ 返回
```

**为什么不需要分布式锁**：

缓存击穿的触发条件是"key 被删除后并发 miss"。`user:session` 不主动删除，只靠 TTL 自然过期。TTL 加了随机抖动，各用户 key 过期时间分散。单个用户的并发量不足以构成热点。因此整个方案从根源上消灭了击穿的触发条件，无需引入分布式锁的复杂度。

---

## 七、认证设计

### 7.1 双 Token

- **Access Token**：15 分钟，JWT payload `{ sub: userId, roles: string[] }`
- **Refresh Token**：7 天，JWT payload `{ sub: userId, type: 'refresh', jti: randomUUID() }`
- Refresh Token 的 JTI 存入 Redis：`refresh_token:{userId}` → `jti值`，TTL 7 天

### 7.2 JTI 轮转防盗

```
刷新时：
1. 验证 refresh token 签名
2. Redis GET refresh_token:{userId}
3. 比对 JTI：
   - 匹配 → 签发新的 access + refresh，更新 Redis 中的 JTI
   - 不匹配 → 检测到重放攻击，删除 Redis 中该用户的所有会话，返回 401
```

### 7.3 登录防暴力

```
Redis key: login:attempts:{username}，TTL 900 秒
失败 → INCR
>= 5 → 返回 429 "账号暂时锁定，请 15 分钟后重试"
登录成功 → DEL key
```

### 7.4 密码修改

```
POST /auth/change-password { oldPassword, newPassword }
1. bcrypt.compare 验证旧密码
2. 新密码 != 旧密码
3. bcrypt.hash 新密码，UPDATE users
4. DEL refresh_token:{userId}（强制登出，权限随 token 失效）
```

### 7.5 JWT 数据分层设计决策

Access Token payload 只存 `{ sub, roles }`，不存 status、username、email 等字段。这是一个有意为之的权衡：

| 数据 | 存放位置 | 原因 |
|------|---------|------|
| `roles` | JWT payload | 变动极少（需重新部署才能调整），15 分钟的过期窗口可接受，省去每次请求查权限表 |
| `status` | 每次请求实时获取 | 封禁操作必须立即生效，不能有延迟 |
| `username/email` | 每次请求实时获取 | 用户信息可能更新，保持 request.user 的数据准确性 |

第 5 阶段引入 Redis 缓存后，status 改由黑名单 Set 实时管控，username/email 由 user:session 缓存提供，DB 查询频率大幅降低，同时保留了各字段原有的一致性保证。

`type: 'refresh'` 字段：Access Token 和 Refresh Token 使用同一个 JWT_SECRET 签发，type 字段是显式的语义边界——防止 Access Token 被误提交到 `/auth/refresh` 接口，也防止 Refresh Token 被当作 Access Token 使用。

### 7.6 角色变更的一致性处理

roles 存在 JWT payload 里，Access Token 有效期 15 分钟，角色变更后存在一个过期窗口。两种方向的危害不同，处理策略也不同：

| 变更方向 | 危害 | 处理策略 |
|---------|------|---------|
| 升级（USER → MODERATOR） | 延迟 15 分钟获得新权限 | 只更新 DB，接受延迟，低风险 |
| 降级（ADMIN → USER） | 15 分钟内仍可执行高权限操作 | 更新 DB + 立即 `DEL refresh_token:{userId}` |

**降级处理流程**：

```
PATCH /users/:id/role（仅 ADMIN，在 UsersService 中实现）→
  1. 并行读取：确认用户存在、获取新角色 ID、记录当前角色（用于降级检测）
     （必须在事务前读取当前角色，事务后 user_roles 已是新状态）
  2. 事务：DELETE 旧 user_roles 记录 → INSERT 新角色行（替换语义，非添加）
  3. 若新角色权限低于旧角色 → DEL refresh_token:{userId}
     （Redis 操作在事务外：Redis 不参与 DB 事务，最坏情况旧 token 多用 15 分钟，可接受）

效果：
  用户当前 Access Token  → 最多再用 15 分钟（无法强制撤销，JWT 无状态）
  用户尝试刷新 Token     → 失败，被迫重新登录
  重新登录               → 获得降级后角色的新 JWT
```

这与密码修改的处理模式完全一致，复用现有机制，无需引入新的 Redis key。

**为什么不彻底关闭 15 分钟窗口**：

彻底关闭需要给 Access Token 加 JTI 并在每次请求时查黑名单，Access Token 因此变为有状态的，违背 JWT 无状态的设计初衷，且增加每次请求的 Redis 查询。对本项目而言，角色降级是极低频操作，15 分钟窗口风险可控，不值得引入这一复杂度。

---

## 八、API 端点清单（43 个）

### Auth（5 个）
| 方法 | 路径 | 访问控制 | 说明 |
|------|------|---------|------|
| POST | /auth/register | Public | 用户注册 |
| POST | /auth/login | Public（5/min限流） | 用户登录 |
| POST | /auth/logout | 已登录 | 用户登出 |
| POST | /auth/refresh | Public | 刷新 token |
| POST | /auth/change-password | 已登录 | 修改密码 |

### Users（8 个）
| 方法 | 路径 | 访问控制 | 说明 |
|------|------|---------|------|
| GET | /users | ADMIN / MODERATOR | 用户列表（分页+搜索） |
| GET | /users/me | 已登录 | 当前用户信息 |
| GET | /users/:id | Public | 用户主页 |
| PATCH | /users/:id | 已登录（本人 或 ADMIN） | 更新用户信息 |
| DELETE | /users/:id | ADMIN | 软删除用户 |
| PATCH | /users/:id/ban | ADMIN | 封禁用户 |
| PATCH | /users/:id/unban | ADMIN | 解封用户 |
| PATCH | /users/:id/role | ADMIN | 替换用户角色（语义是替换而非添加，降级时强制删除 refresh token） |

### Roles（2 个）

> 角色为业务常量（ADMIN / MODERATOR / USER），不通过 API 动态创建。
> 这两个接口仅供前端管理后台展示角色及其权限树。

| 方法 | 路径 | 访问控制 | 说明 |
|------|------|---------|------|
| GET | /roles | ADMIN | 角色列表（含各角色已分配的权限） |
| GET | /roles/:id | ADMIN | 角色详情（含权限列表） |

### Songs（8 个）
| 方法 | 路径 | 访问控制 | 说明 |
|------|------|---------|------|
| GET | /songs | Public | 歌曲列表（分页+搜索+分类过滤，永远只返回 APPROVED） |
| GET | /admin/songs | ADMIN / MODERATOR | 歌曲列表（可按 status 过滤任意状态，内容审核场景） |
| GET | /songs/:slug | Public | 歌曲详情（含分类、上传者信息） |
| POST | /songs | 已登录 | 创建歌曲（可选 categoryIds） |
| PATCH | /songs/:id | 已登录（本人 或 ADMIN） | 更新歌曲 |
| DELETE | /songs/:id | 已登录（本人 或 ADMIN） | 软删除歌曲 |
| PATCH | /songs/:id/approve | ADMIN / MODERATOR | 审核通过（触发 AI 分析） |
| PATCH | /songs/:id/reject | ADMIN / MODERATOR | 审核拒绝（需填写原因） |

### Categories（5 个）
| 方法 | 路径 | 访问控制 | 说明 |
|------|------|---------|------|
| GET | /categories | Public | 分类列表 |
| GET | /categories/:id | Public | 分类详情 |
| POST | /categories | ADMIN | 创建分类 |
| PATCH | /categories/:id | ADMIN | 更新分类 |
| DELETE | /categories/:id | ADMIN | 删除分类（有歌曲关联时拒绝） |

### Comments（7 个）
| 方法 | 路径 | 访问控制 | 说明 |
|------|------|---------|------|
| GET | /comments | Public | 评论列表（按 songId 过滤，永远只返回 VISIBLE） |
| GET | /admin/comments | ADMIN / MODERATOR | 评论列表（可按 status 过滤任意状态，内容审核场景） |
| GET | /comments/:id | Public | 评论详情 |
| POST | /comments | 已登录 | 发表评论（默认 VISIBLE，事后审核模式） |
| PATCH | /comments/:id | 已登录（本人 或 ADMIN/MODERATOR） | 编辑评论 |
| DELETE | /comments/:id | 已登录（本人 或 ADMIN/MODERATOR） | 软删除评论 |
| PATCH | /comments/:id/hide | ADMIN / MODERATOR | 隐藏评论 → HIDDEN（发现问题才介入） |

### Collections（4 个）
| 方法 | 路径 | 访问控制 | 说明 |
|------|------|---------|------|
| GET | /collections | 已登录 | 我的收藏列表（分页） |
| GET | /collections/check/:songId | 已登录 | 检查是否已收藏 |
| POST | /collections | 已登录 | 收藏歌曲 |
| DELETE | /collections/:id | 已登录（本人） | 取消收藏 |

### Admin Logs（1 个）
| 方法 | 路径 | 访问控制 | 说明 |
|------|------|---------|------|
| GET | /admin-logs | ADMIN / MODERATOR | 审计日志列表（按 action/targetType/日期过滤） |

### Analyses（2 个）
| 方法 | 路径 | 访问控制 | 说明 |
|------|------|---------|------|
| GET | /analyses/song/:songId | Public | 获取歌曲分析结果 |
| POST | /analyses/:id/retry | ADMIN | 重试失败的分析 |

---

## 九、实施阶段

### 第 1 阶段：基础设施

按以下顺序，逐个模块实现，每个等待确认：

1. **项目初始化** — package.json, tsconfig.json(含 path aliases), nest-cli.json, eslint.config.mjs, .env.example, docker-compose.yml, drizzle.config.ts
2. **ConfigModule** — env.validation.ts（Joi 环境变量校验）
3. **DatabaseModule** — database.module.ts + database.service.ts
4. **Schema** — schema.ts 单文件（12 张表 + 7 个枚举）
5. **Seed** — seed.ts（bcrypt 实时哈希，打印密码到控制台）
6. **RedisModule** — redis.module.ts + redis.service.ts + redis.constants.ts
7. **Common 层** — filters, interceptors, middleware, dto, decorators, utils, interfaces
8. **应用入口** — main.ts（Pino 日志, helmet, 分级限流, CORS, Swagger）+ app.module.ts + app.controller.ts（健康检查）
9. **修复 turbo.json** — outputs 加 `dist/**`

### 第 2 阶段：认证 + 用户 + RBAC

1. **Auth 基础设施** — guards, strategies, decorators, utils, interfaces
2. **AuthModule** — auth.module.ts + auth.service.ts + auth.controller.ts + DTOs
3. **UsersModule** — users.module.ts + users.service.ts + users.controller.ts + DTOs
   - 含 `PATCH /users/:id/role`（ADMIN 分配角色，降级时强制 DEL refresh_token）
4. **CaslModule** — casl.module.ts + casl-ability.factory.ts
5. **RolesModule** — roles.module.ts + roles.service.ts + roles.controller.ts
   - 仅 GET /roles 和 GET /roles/:id（含权限列表），供前端展示，无增删改
   - PermissionsModule 不建：权限数据已通过 GET /roles/:id 的 JOIN 查询暴露，无独立调用场景
6. **注册全局守卫** — 更新 app.module.ts

### 第 3 阶段：业务模块

1. **AdminLogsModule** — admin-logs.module.ts + admin-logs.service.ts + admin-logs.controller.ts（先建，其他模块会注入它）
2. **CategoriesModule** — categories.module.ts + categories.service.ts + categories.controller.ts + DTOs
3. **SongsModule** — songs.module.ts + songs.service.ts + songs.controller.ts + admin-songs.controller.ts + DTOs（含 query-admin-songs.dto.ts；公开接口永远 APPROVED，管理员接口走 /admin/songs；含 categoryIds 集成 + adminLogs 集成）
4. **CommentsModule** — comments.module.ts + comments.service.ts + comments.controller.ts + admin-comments.controller.ts + DTOs（含 query-admin-comments.dto.ts；事后审核：默认 VISIBLE，公开接口永远 VISIBLE，管理员审核走 /admin/comments，版主只可 hide；含 adminLogs 集成）
5. **CollectionsModule** — collections.module.ts + collections.service.ts + collections.controller.ts + DTOs

### 第 4 阶段：AI 分析 + 测试

1. **AnalysesModule** — analyses.module.ts + analyses.service.ts + analyses.processor.ts + analyses.controller.ts + openai/
2. **单元测试** — 所有 Service 的 spec 文件
3. **集成测试** — 所有 Controller 的 spec 文件
4. **E2E 测试** — auth, songs, rbac 流程测试

### 第 5 阶段：生产就绪

1. **Swagger 完善** — 所有端点添加完整的 @ApiOperation, @ApiResponse
2. **Dockerfile** — 多阶段构建
3. **CI/CD** — GitHub Actions
4. **Redis 缓存** — 热点路径缓存（分类列表、首页歌曲）+ user:session 信息缓存 + user:banned 黑名单 Set（见 6.3 节）
5. **README**

---

## 十、安全规范

### 强制要求

1. **越权防护**：所有用户数据操作从 JWT token 获取用户身份，禁止信任客户端传递的 userId
2. **密码存储**：bcrypt，rounds=10
3. **密码传输**：通过请求体传输，禁止 URL 参数
4. **UUID 主键**：防止资源枚举攻击
5. **认证通用错误**：登录失败统一返回"用户名或密码错误"，不透露用户是否存在
6. **日志脱敏**：禁止在日志中记录密码、token、密钥
7. **生产环境**：关闭调试模式，Swagger 可选关闭
8. **请求校验**：ValidationPipe 开启 whitelist + forbidNonWhitelisted
9. **SQL 注入**：全部使用 Drizzle ORM 参数化查询，禁止字符串拼接
10. **会话安全**：登录成功后生成新 token，登出后强制 token 过期，密码修改后所有会话失效

### 禁止

- 硬编码密码、密钥、token
- 在响应中返回 password 字段
- 使用 eval() 执行用户输入
- 直接执行 SQL 语句的调试接口
- 绕过认证的隐藏入口

---

## 附录：实现细节（精确配置值与完整代码）

> 以下内容提供本项目的精确实现细节，确保 100% 复刻。覆盖：环境变量、路径别名、BullMQ 配置、AI 分析完整实现、Slug 算法、Redis 常量、限流配置、种子数据、测试模式、Docker/CI、ESLint 规则。

## 附录.1 环境变量（.env.example 完整清单）

```env
# ===========================================
# Lyric Platform API - Environment Variables
# ===========================================
# 复制此文件为 .env 并填入实际值：cp .env.example .env

# --- Database ---
DATABASE_URL=postgresql://postgres:password@localhost:5432/lyric_platform
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DB=lyric_platform
POSTGRES_PORT=5432
# DB_MAX_CONNECTIONS=10

# --- Server ---
PORT=3000
NODE_ENV=development

# --- JWT ---
JWT_SECRET=change-me-to-a-random-string-at-least-32-chars

# --- Redis ---
REDIS_HOST=localhost
REDIS_PORT=6379
# REDIS_PASSWORD=
# REDIS_DB=0

# --- CORS ---
# CORS_ORIGIN=http://localhost:3000,http://localhost:5173

# --- AI Analysis (Optional) ---
# OPENAI_API_KEY=sk-...
# OPENAI_MODEL=gpt-4
```

**环境变量校验（Joi Schema）**：

| 变量 | 类型 | 必填 | 校验规则 |
|------|------|------|---------|
| DATABASE_URL | string | 是 | URI scheme: postgresql/postgres |
| DB_MAX_CONNECTIONS | number | 否 | 1-100 |
| PORT | number | 否 | 默认 3000 |
| NODE_ENV | string | 否 | development/production/test |
| JWT_SECRET | string | 是 | 最少 32 字符 |
| REDIS_HOST | string | 否 | 默认 localhost |
| REDIS_PORT | number | 否 | 默认 6379 |
| REDIS_PASSWORD | string | 否 | 允许空 |
| REDIS_DB | number | 否 | 默认 0 |
| CORS_ORIGIN | string | 否 | - |
| OPENAI_API_KEY | string | 否 | - |
| OPENAI_MODEL | string | 否 | - |

---

## 附录.2 TypeScript 配置（路径别名）

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "target": "ES2023",
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": "./",
    "incremental": true,
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "paths": {
      "@database/*": ["src/database/*"],
      "@common/*": ["src/common/*"],
      "@auth/*": ["src/auth/*"]
    }
  }
}
```

**关键点**：
- `baseUrl: "./"` — 路径别名的基准
- `@database/*` → `src/database/*`
- `@common/*` → `src/common/*`
- `@auth/*` → `src/auth/*`
- `strict: true` — 严格模式
- `target: "ES2023"` — 使用现代 JS 特性

---

## 附录.3 nest-cli.json

```json
{
  "$schema": "https://json.schemastore.org/nest-cli",
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": { "deleteOutDir": true }
}
```

---

## 附录.4 BullMQ 队列配置

技术栈表补充：

| 层 | 技术 | 版本 | 用途 |
|---|---|---|---|
| 消息队列 | BullMQ | ^5.79.1 | AI 分析异步处理 |
| 队列集成 | @nestjs/bullmq | ^11.0.4 | NestJS BullMQ 装饰器 |

**队列名称**：`analyse-lyrics`

**BullMQ 根配置**（复用 Redis 连接）：

```typescript
// app.module.ts imports 中
BullModule.forRootAsync({
  inject: [ConfigService],
  useFactory: (config: ConfigService) => ({
    connection: {
      host: config.get<string>('REDIS_HOST', 'localhost'),
      port: config.get<number>('REDIS_PORT', 6379),
      password: config.get<string>('REDIS_PASSWORD') || undefined,
      db: config.get<number>('REDIS_DB', 0),
    },
  }),
}),
```

**模块队列注册**：

```typescript
// analyses.module.ts
BullModule.registerQueue({ name: ANALYSIS_QUEUE })
```

**任务入队参数**：

```typescript
await this.analysisQueue.add("analyse", { analysisId, songId }, {
  attempts: 3,
  backoff: { type: "exponential", delay: 5000 },
});
```

---

## 附录.5 AI 分析模块完整实现

### 5.1 模块结构

```
analyses/
├── analyses.module.ts       # BullMQ 队列注册
├── analyses.controller.ts   # GET /analyses/song/:songId, POST /analyses/:id/retry
├── analyses.service.ts      # 查询 + 入队 + 重试
├── analyses.processor.ts    # BullMQ Worker
├── analyses.constants.ts    # ANALYSIS_QUEUE = "analyse-lyrics"
├── interfaces/
│   └── analysis.interface.ts
└── openai/
    └── openai.service.ts    # AI 调用 + Mock 降级
```

### 5.2 接口定义

```typescript
export interface SentimentResult {
  overall: string;   // positive / negative / neutral / mixed
  score: number;     // 0-1
  details?: Record<string, number>;
}

export interface QualityResult {
  overall: number;   // 0-100
  rhyme?: number;
  imagery?: number;
  structure?: number;
}

export interface AnalysisResult {
  sentiment: SentimentResult;
  themes: string[];
  quality: QualityResult;
  summary: string;
}

export interface AnalysisResponse {
  id: string;
  songId: string;
  sentiment: SentimentResult | Record<string, never>;
  themes: string[];
  quality: QualityResult | Record<string, never>;
  summary: string;
  status: string;
  errorMessage: string | null;
  createdAt: Date;
  updatedAt: Date;
}
```

### 5.3 OpenAI Service（含 Mock 降级）

```typescript
@Injectable()
export class OpenAIService {
  private readonly apiKey: string | undefined;
  private readonly model: string;

  constructor(private readonly configService: ConfigService) {
    this.apiKey = this.configService.get<string>("OPENAI_API_KEY");
    this.model = this.configService.get<string>("OPENAI_MODEL") ?? "gpt-4o-mini";
    if (!this.apiKey) {
      this.logger.warn("OPENAI_API_KEY 未配置，AI 分析将使用 mock 结果");
    }
  }

  async analyzeLyrics(lyrics: string, title: string, artist: string): Promise<AnalysisResult> {
    if (!this.apiKey) return this.mockAnalysis(title, artist);
    // 使用 fetch 调用 OpenAI API
    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${this.apiKey}`,
      },
      body: JSON.stringify({
        model: this.model,
        messages: [
          { role: "system", content: SYSTEM_PROMPT },
          { role: "user", content: `请分析以下歌词：\n\n歌曲：${title}\n艺术家：${artist}\n\n歌词内容：\n${lyrics}` },
        ],
        temperature: 0.3,
        response_format: { type: "json_object" },
      }),
    });
    // 解析并返回 JSON
  }

  private mockAnalysis(title: string, artist: string): AnalysisResult {
    return {
      sentiment: { overall: "positive", score: 0.75, details: { joy: 0.6, sadness: 0.2, nostalgia: 0.4 } },
      themes: ["爱情", "回忆", "成长"],
      quality: { overall: 78, rhyme: 80, imagery: 75, structure: 78 },
      summary: `「${title}」（${artist}）是一首情感细腻的作品，通过丰富的意象表达了对生活的感悟。[Mock 分析结果]`,
    };
  }
}
```

### 5.4 AI System Prompt（固定常量）

```
你是一个专业的歌词分析助手。请对给定的歌词进行以下分析，并以 JSON 格式返回：

{
  "sentiment": {
    "overall": "positive|negative|neutral|mixed",
    "score": 0.0-1.0,
    "details": { "情感维度": 0.0-1.0 }
  },
  "themes": ["主题1", "主题2", ...],
  "quality": {
    "overall": 0-100,
    "rhyme": 0-100,
    "imagery": 0-100,
    "structure": 0-100
  },
  "summary": "一句话总结这首歌词的核心内容和艺术特点"
}

要求：
- themes 最多 5 个标签
- summary 控制在 100 字以内
- quality 各项评分客观公正
- 严格返回 JSON，不要附加其他文本
```

### 5.5 Processor（BullMQ Worker）

```typescript
export interface AnalysisJobData {
  analysisId: string;
  songId: string;
}

@Processor(ANALYSIS_QUEUE)
export class AnalysesProcessor extends WorkerHost {
  async process(job: Job<AnalysisJobData>): Promise<void> {
    const { analysisId, songId } = job.data;
    // 1. UPDATE analyses SET status='PROCESSING'
    // 2. SELECT title, artist, lyrics FROM songs WHERE id=songId
    // 3. 歌曲不存在 → FAILED + "关联歌曲不存在"
    // 4. openAIService.analyzeLyrics(lyrics, title, artist)
    // 5. UPDATE analyses SET sentiment, themes, quality, summary, status='COMPLETED'
  }
}
```

### 5.6 触发时机

歌曲审核通过时（`SongsService.approve()`）：
1. 事务内：INSERT analyses 记录（status=PENDING，sentiment=`{}`，quality=`{}`，summary=''，themes=[]）
2. 事务外：`analysesService.enqueue(analysisId, songId)`

---

## 附录.6 Slug 生成算法

```typescript
// src/common/utils/slug.util.ts
export function generateSlug(title: string, artist: string): string {
  const timestamp = Date.now();
  return `${title}-${artist}-${timestamp}`
    .toLowerCase()
    .replace(/\s+/g, '-')
    .replace(/[^\w\u4e00-\u9fa5-]/g, '');
}
```

**规则**：格式 `{title}-{artist}-{毫秒时间戳}`，全小写，空白→连字符，保留中文+字母数字+连字符，去除其他字符。

---

## 附录.7 Redis Key 全局常量

```typescript
// src/redis/redis.constants.ts
export const REDIS_KEYS = {
  REFRESH_TOKEN: 'refresh_token',      // refresh_token:{userId} → jti值
  LOGIN_ATTEMPTS: 'login:attempts',    // login:attempts:{username} → 计数
  USER_SESSION: 'user:session',        // user:session:{userId} → JSON
  USER_BANNED: 'user:banned',          // Set<userId>
  CATEGORIES_LIST: 'categories:list',  // JSON（分类列表缓存）
  SONGS_HOMEPAGE: 'songs:homepage',    // JSON（首页歌曲缓存）
} as const;

export const HOTPATH_CACHE = {
  CATEGORIES_TTL: 3600,    // 分类列表：1 小时
  SONGS_HOMEPAGE_TTL: 300, // 首页歌曲：5 分钟
} as const;

export const LOGIN_LOCK = {
  MAX_ATTEMPTS: 5,         // 最大连续失败 5 次
  LOCK_DURATION: 15 * 60,  // 锁定 900 秒
} as const;

export const SESSION_CACHE = {
  BASE_TTL: 300,           // 基础 300s
  JITTER: 60,              // 随机 0-60s 防雪崩
} as const;
```

---

## 附录.8 Redis Service 完整 API

```typescript
@Injectable()
export class RedisService implements OnModuleDestroy {
  constructor(configService: ConfigService) {
    this.client = new Redis({
      host: configService.get('REDIS_HOST', 'localhost'),
      port: configService.get('REDIS_PORT', 6379),
      password: configService.get('REDIS_PASSWORD') || undefined,
      db: configService.get('REDIS_DB', 0),
    });
  }

  async set(key: string, value: string, ttlSeconds?: number): Promise<void>
  async get(key: string): Promise<string | null>
  async del(key: string): Promise<void>
  async exists(key: string): Promise<boolean>
  async incr(key: string, ttlSeconds?: number): Promise<number>  // 首次 incr 时设 TTL
  async deleteByPattern(pattern: string): Promise<number>        // SCAN + DEL
  async ping(): Promise<boolean>
  async sAdd(key: string, member: string): Promise<void>         // Set 添加
  async sRem(key: string, member: string): Promise<void>         // Set 移除
  async sIsMember(key: string, member: string): Promise<boolean> // Set 判存
}
```

---

## 附录.9 限流配置

```typescript
// main.ts - 全局限流：100 次 / 15 分钟
app.use(rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: { code: 429, message: '请求过于频繁，请稍后再试', data: null },
  standardHeaders: true,
  legacyHeaders: false,
}));

// main.ts - 登录单独限流：5 次 / 1 分钟
app.use('/auth/login', rateLimit({
  windowMs: 60 * 1000,
  max: 5,
  message: { code: 429, message: '登录尝试过于频繁，请1分钟后再试', data: null },
  standardHeaders: true,
  legacyHeaders: false,
}));
```

---

## 附录.10 种子数据完整内容

### 10.1 角色

| code | name | description | isSystem |
|------|------|-------------|----------|
| ADMIN | 管理员 | 系统管理员，拥有所有权限 | true |
| MODERATOR | 版主 | 版主，可以审核内容 | true |
| USER | 普通用户 | 普通用户，可以创建和管理自己的内容 | true |

### 10.2 权限（22 个，全部 type=BUTTON，isSystem=true）

```
users:list(1), users:view(2), users:update(3), users:delete(4), users:role(5)
roles:list(6), permissions:list(7)
songs:list(8), songs:view(9), songs:create(10), songs:update(11), songs:delete(12), songs:approve(13)
categories:manage(14)
comments:create(15), comments:update(16), comments:delete(17), comments:moderate(18)
collections:create(19), collections:delete(20)
admin-logs:list(21), analyses:manage(22)
```

### 10.3 角色-权限分配

- **ADMIN（22）**：全部
- **MODERATOR（10）**：users:list, songs:list, songs:view, songs:create, songs:approve, comments:create, comments:moderate, collections:create, collections:delete, admin-logs:list
- **USER（6）**：songs:list, songs:view, songs:create, comments:create, collections:create, collections:delete

### 10.4 测试账号

| 角色 | email | username | password | bcrypt rounds |
|------|-------|----------|----------|:---:|
| ADMIN | admin@lyric-platform.com | admin | Admin@123456 | 10 |
| MODERATOR | moderator@lyric-platform.com | moderator | Moderator@123456 | 10 |
| USER | user@lyric-platform.com | testuser | User@123456 | 10 |

三个哈希并行生成（`Promise.all`），密码明文打印到控制台。

### 10.5 分类

| name | slug | order |
|------|------|:-----:|
| 流行 | pop | 1 |
| 摇滚 | rock | 2 |
| 民谣 | folk | 3 |
| 说唱 | rap | 4 |
| 电子 | electronic | 5 |

### 10.6 示例歌曲（3 首，userId=admin，status=APPROVED）

| slug | title | artist | album | tags |
|------|-------|--------|-------|------|
| ye-qu | 夜曲 | 周杰伦 | 十一月的萧邦 | ["周杰伦","抒情","经典"] |
| qing-tian | 晴天 | 周杰伦 | 叶惠美 | ["周杰伦","青春","怀旧"] |
| dao-xiang | 稻香 | 周杰伦 | 魔杰座 | ["周杰伦","励志","温暖"] |

### 10.7 数据清除顺序（按外键依赖）

```
rolePermissions → userRoles → songCategories → collections
→ comments → analyses → adminLogs → songs → categories
→ permissions → roles → users
```

---

## 附录.11 测试模式与规范

### 11.1 测试框架

- **测试运行器**：Jest 29
- **NestJS 集成**：`@nestjs/testing` 的 `Test.createTestingModule`
- **描述语言**：中文 `describe` + `it`
- **断言风格**：`expect(...).toThrow(ExceptionType)`, `expect(...).toHaveBeenCalledWith(...)`

### 11.2 Service 测试模式

```typescript
describe("SongsService", () => {
  let service: SongsService;
  let mockDb: any;

  beforeEach(async () => {
    // 模拟 Drizzle 链式调用
    mockDb = {
      db: {
        select: jest.fn().mockReturnThis(),
        from: jest.fn().mockReturnThis(),
        where: jest.fn().mockReturnThis(),
        limit: jest.fn().mockResolvedValue([]),
        leftJoin: jest.fn().mockReturnThis(),
        innerJoin: jest.fn().mockReturnThis(),
        orderBy: jest.fn().mockReturnThis(),
        offset: jest.fn().mockResolvedValue([]),
        insert: jest.fn().mockReturnThis(),
        values: jest.fn().mockReturnThis(),
        returning: jest.fn().mockResolvedValue([]),
        update: jest.fn().mockReturnThis(),
        set: jest.fn().mockReturnThis(),
        delete: jest.fn().mockReturnThis(),
        transaction: jest.fn(),
      },
    };

    const module = await Test.createTestingModule({
      providers: [
        SongsService,
        { provide: DatabaseService, useValue: mockDb },
        { provide: AdminLogsService, useValue: { create: jest.fn() } },
        { provide: AnalysesService, useValue: { enqueue: jest.fn() } },
        { provide: RedisService, useValue: { get: jest.fn().mockResolvedValue(null), set: jest.fn(), del: jest.fn() } },
      ],
    }).compile();

    service = module.get<SongsService>(SongsService);
  });

  // 通过控制 mockReturnThis/mockResolvedValue 模拟不同场景
  it("歌曲不存在时抛出 NotFoundException", async () => {
    mockDb.db.limit.mockResolvedValueOnce([]);
    await expect(service.findById("x")).rejects.toThrow(NotFoundException);
  });
});
```

### 11.3 Controller 测试模式

```typescript
describe("SongsController", () => {
  let controller: SongsController;
  let mockService: any;
  let mockCasl: any;

  beforeEach(async () => {
    mockService = {
      findAll: jest.fn().mockResolvedValue({ items: [], total: 0 }),
      findBySlug: jest.fn().mockResolvedValue({ id: "s1" }),
      create: jest.fn().mockResolvedValue({ id: "s1" }),
      findById: jest.fn().mockResolvedValue({ id: "s1", userId: "u1", status: "PENDING" }),
      update: jest.fn().mockResolvedValue({ id: "s1" }),
      remove: jest.fn().mockResolvedValue({ message: "ok" }),
    };
    mockCasl = {
      createForUser: jest.fn().mockReturnValue({ can: jest.fn().mockReturnValue(true) }),
    };

    const module = await Test.createTestingModule({
      controllers: [SongsController],
      providers: [
        { provide: SongsService, useValue: mockService },
        { provide: CaslAbilityFactory, useValue: mockCasl },
      ],
    }).compile();

    controller = module.get<SongsController>(SongsController);
  });

  it("权限不足时抛出 ForbiddenException", async () => {
    mockCasl.createForUser.mockReturnValue({ can: jest.fn().mockReturnValue(false) });
    await expect(controller.update("s1", {}, { id: "u2", roles: ["USER"] })).rejects.toThrow(ForbiddenException);
  });
});
```

### 11.4 Auth Service 测试要点

- 登录失败次数 ≥ 5 → 抛出 HttpException（429）
- 用户不存在 → UnauthorizedException（统一错误，不泄露用户存在性）
- JTI 不匹配 → UnauthorizedException（重放攻击检测）
- type !== 'refresh' → UnauthorizedException
- 注册邮箱已存在 → ConflictException
- 注册用户名已存在 → ConflictException
- 事务模式：注册时 mock `transaction` 内含 insert + select + insert

---

## 附录.12 Docker 多阶段构建

```dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
RUN corepack enable && corepack prepare pnpm@9.0.0 --activate
WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/api/package.json ./apps/api/
RUN pnpm install --frozen-lockfile --filter api

# Stage 2: Build
FROM node:20-alpine AS build
RUN corepack enable && corepack prepare pnpm@9.0.0 --activate
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/apps/api/node_modules ./apps/api/node_modules
COPY apps/api ./apps/api
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml turbo.json ./
RUN pnpm --filter api build

# Stage 3: Production
FROM node:20-alpine AS prod
RUN corepack enable && corepack prepare pnpm@9.0.0 --activate
WORKDIR /app
ENV NODE_ENV=production
COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/apps/api/node_modules ./apps/api/node_modules
COPY --from=build /app/apps/api/dist ./apps/api/dist
COPY apps/api/package.json ./apps/api/
EXPOSE 3000
CMD ["node", "apps/api/dist/main.js"]
```

---

## 附录.13 CI/CD（GitHub Actions）

```yaml
name: CI
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }

jobs:
  lint-test-build:
    runs-on: ubuntu-latest
    services:
      redis:
        image: redis:7-alpine
        ports: ["6379:6379"]
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9.0.0 }
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - name: Lint
        run: pnpm --filter api lint
      - name: Unit Tests
        run: pnpm --filter api test
      - name: Build
        run: pnpm --filter api build

  docker:
    needs: lint-test-build
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## 附录.14 docker-compose.yml（本地开发）

```yaml
services:
  postgres:
    image: postgres:15-alpine
    container_name: lyric-api-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-password}
      POSTGRES_DB: ${POSTGRES_DB:-lyric_platform}
    ports: ["${POSTGRES_PORT:-5432}:5432"]
    volumes: [postgres-data:/var/lib/postgresql/data]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:alpine
    container_name: lyric-api-redis
    restart: unless-stopped
    ports: ["${REDIS_PORT:-6379}:6379"]
    volumes: [redis-data:/data]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres-data: { driver: local }
  redis-data: { driver: local }
```

---

## 附录.15 ESLint 配置

```javascript
// eslint.config.mjs
export default tseslint.config(
  { ignores: ['eslint.config.mjs', 'dist/**'] },
  eslint.configs.recommended,
  ...tseslint.configs.recommendedTypeChecked,
  eslintPluginPrettierRecommended,
  {
    languageOptions: {
      globals: { ...globals.node, ...globals.jest },
      sourceType: 'commonjs',
      parserOptions: { projectService: true, tsconfigRootDir: import.meta.dirname },
    },
  },
  {
    rules: {
      '@typescript-eslint/no-explicit-any': 'warn',
      '@typescript-eslint/no-floating-promises': 'error',
      '@typescript-eslint/no-unsafe-argument': 'warn',
      '@typescript-eslint/no-unsafe-call': 'warn',
      '@typescript-eslint/no-unsafe-assignment': 'warn',
      '@typescript-eslint/no-unsafe-member-access': 'warn',
      '@typescript-eslint/no-unsafe-return': 'warn',
      'no-console': ['warn', { allow: ['warn', 'error'] }],
    },
  },
);
```

---

## 附录.16 Drizzle 配置

```typescript
// drizzle.config.ts
import type { Config } from 'drizzle-kit';
import * as dotenv from 'dotenv';
dotenv.config();

export default {
  schema: './src/database/schema.ts',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: { url: process.env.DATABASE_URL! },
  verbose: true,
  strict: true,
} satisfies Config;
```

---

## 附录.17 DatabaseService 连接池配置

```typescript
// 生产：10 连接，开发：1 连接
const maxConnections = configService.get<number>('DB_MAX_CONNECTIONS') || (isProduction ? 10 : 1);

this.client = postgres(connectionString, {
  max: maxConnections,
  idle_timeout: 20,
  connect_timeout: 10,
  prepare: false,  // pgBouncer 兼容
});
```

---

## 附录.18 JwtStrategy 认证流程（含缓存）

```
请求 → JWT 签名验证 → validate(payload)
  → SISMEMBER user:banned {userId}     // O(1) 封禁检查
      命中 → 403 "账号已被封禁"
  → GET user:session:{userId}
      命中 → 返回缓存的 { id, username, email } + roles from payload
      未命中 → SELECT id, username, email, status FROM users
               → status 非 ACTIVE → 403
               → SET user:session:{userId} JSON (TTL=300+random(60))
               → 返回 AuthenticatedUser
```

---

## 附录.19 全局异常过滤器数据库错误映射

```typescript
const UNIQUE_CONSTRAINT_MESSAGES: Record<string, string> = {
  users_email_unique: '邮箱已被注册',
  users_username_unique: '用户名已被占用',
  collections_user_id_song_id_unique: '已经收藏过该歌曲',
  user_roles_user_id_role_id_unique: '用户已拥有该角色',
  role_permissions_role_id_permission_id_unique: '角色已拥有该权限',
  songs_slug_unique: '歌曲标识已存在',
  categories_slug_unique: '分类标识已存在',
};

// PostgreSQL 错误码处理
// 23505 UNIQUE_VIOLATION → 409 + 友好消息
// 23503 FOREIGN_KEY_VIOLATION → 400 "关联数据不存在或已被删除"
// 23502 NOT_NULL_VIOLATION → 400 "必填字段不能为空"
// 其他 23xxx → 500 "数据库操作失败"
```

---

## 附录.20 main.ts 完整启动序列

```typescript
async function bootstrap() {
  // 1. 创建应用（bufferLogs: true）
  const app = await NestFactory.create(AppModule, { bufferLogs: true });

  // 2. Pino Logger 接管
  app.useLogger(app.get(Logger));

  // 3. Helmet 安全头
  app.use(helmet());

  // 4. CORS
  //    - 有 CORS_ORIGIN → 逗号分隔的白名单
  //    - development → true（允许所有）
  //    - production 无配置 → false（拒绝跨域）
  app.enableCors({ origin: ..., credentials: true });

  // 5. 全局限流（100/15min）
  // 6. 登录限流（5/1min）
  // 7. ValidationPipe（whitelist + forbidNonWhitelisted + transform）
  // 8. Swagger（仅非 production）
  //    - 路径：/api/docs
  //    - Bearer Auth scheme: 'access-token'
  //    - persistAuthorization: true

  await app.listen(port);
}
```

---

## 附录.21 Express Request 类型扩展

```typescript
// src/@types/express/index.d.ts
declare namespace Express {
  interface Request {
    requestId?: string;
  }
}
```

---

## 附录.22 快速启动命令

```bash
# 1. 启动基础设施
cd apps/api && docker compose up -d

# 2. 安装依赖
pnpm install

# 3. 推送 Schema 到数据库
pnpm --filter api db:push

# 4. 写入种子数据
pnpm --filter api seed

# 5. 启动开发服务器
pnpm --filter api start:dev

# 6. 访问 Swagger
open http://localhost:3000/api/docs
```
