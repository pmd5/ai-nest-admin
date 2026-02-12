# 项目规格说明书 (SPEC.md)

## 1. 技术栈概览

本项目的核心后端技术选型如下：

- **核心框架:** NestJS 11.x (Fastify Adapter)
- **语言:** TypeScript 5.x
- **数据库:** MySQL 8.0+
- **ORM:** TypeORM
- **缓存:** Redis (使用 `@liaoliaots/nestjs-redis`)
- **消息队列:** Bull (基于 Redis)
- **API 文档:** Swagger (`@nestjs/swagger`)
- **包管理器:** pnpm
- **构建工具:** Nest CLI / Webpack

## 2. 项目架构 (Modular Monolith)

本项目采用模块化单体架构，核心逻辑位于 `src/modules`。

### 2.1 目录结构 (`src/`)

| 路径 | 说明 | 规则 |
| :--- | :--- | :--- |
| **`modules/`** | 业务模块 | 按领域划分 (e.g. `user`, `system`, `auth`)。包含 Controller, Service, Module 定义。 |
| **`common/`** | 公共库 | 全局通用的 DTO, Entity, Fitlers, Guards, Interceptors, Pipes。 |
| **`config/`** | 配置中心 | 所有的配置加载逻辑 (Database, Redis, OSS 等)。 |
| **`shared/`** | 共享模块 | 封装第三方服务的全局模块 (Logger, Mailer, Helper)。 |
| **`utils/`** | 工具函数 | 纯函数工具类 (Crypto, Date, File)。 |
| **`main.ts`** | 入口文件 | 应用启动、全局中间件配置。 |

## 3. 核心开发约定

### 3.1 实体定义 (Entities)

- 所有实体必须放在模块下的 `*.entity.ts` 文件中，或 `common/entity`。
- **必须继承** `CommonEntity` (位于 `src/common/entity/common.entity.ts`)，它提供了 `id`, `created_at`, `updated_at`。
- 示例:
  ```typescript
  @Entity({ name: 'sys_user' })
  export class UserEntity extends CommonEntity { ... }
  ```

### 3.2 控制器 (Controllers)

- 使用 `@Controller('path')` 定义路由前缀。
- 所有接口方法需使用 `@ApiOperation` 和 `@ApiResult` (自定义装饰器) 标注。
- 尽量保持 Controller 轻量，逻辑下沉到 Service。

### 3.3 服务 (Services)

- 业务逻辑的核心载体。
- 数据库操作通过注入 Repository 完成。
- 抛出业务异常时，优先使用 `src/common/exceptions/biz.exception.ts`。

## 4. 常用命令

- `pnpm install`: 安装依赖。
- `pnpm dev`: 启动开发服务器 (Watch Mode)。
- `pnpm build`: 构建生产环境代码 (`dist/`)。
- `pnpm test`: 运行单元测试。
- `pnpm test:e2e`: 运行端对端测试。
- `pnpm migration:generate`: 根据 Entity 变更生成迁移文件。
- `pnpm migration:run`: 执行数据库迁移。
- `docker:up`: 启动 Docker 容器 (Redis, MySQL)。

## 5. 核心功能 (Features)

### 5.1 认证与授权 (Auth & RBAC)

- **JWT 认证**: 基于 `@nestjs/jwt` 和 `passport-jwt`。
- **权限控制**:
  - 使用 Guard (`JwtAuthGuard`, `RolesGuard`) 进行拦截。
  - 使用自定义装饰器 `@Perm('sys:user:add')` 标记需要的权限。
  - 权限数据通常存储在 Redis 中以提高性能。

### 5.2 异步任务 (Async Tasks)

- **队列**: 使用 `@nestjs/bull` 处理异步任务（如发送邮件、生成报表）。
- **定时任务**: 使用 `@nestjs/schedule` 处理 Cron Job。

### 5.3 文件上传

- 基于 Fastify Multipart 支持。
- 实现了 OSS (Object Storage Service) 抽象，支持本地存储或云存储 (七牛云/MinIO 等)。

## 6. 开发工作流 (Workflow)

### 6.1 新增模块流程

1.  **定义实体**: 在 `src/modules/xxx/entities/` 下创建 Entity。
2.  **生成迁移**: 运行 `pnpm migration:generate` 并应用 `pnpm migration:run`。
3.  **定义 DTO**: 创建 `xxx.dto.ts`，定义增删改查参数并添加校验。
4.  **编写 Service**: 实现业务逻辑。
5.  **编写 Controller**: 暴露 API 接口。
6.  **注册 Module**: 将 Controller 和 Service 注册到 `XxxModule`，并在 `AppModule` 中引入。

### 6.2 代码提交

- 严格遵循 Conventional Commits 规范。
- 提交前会自动运行 Lint 检查。

## 7. 业务模块映射 (Business Modules)

核心业务逻辑位于 `src/modules` 目录下：

| 业务领域 | 核心路径 (`src/modules/...`) | 说明 |
| :--- | :--- | :--- |
| **系统管理 (System)** | `system/` | 包含用户、角色、菜单、部门、参数配置等核心管理功能。 |
| &nbsp;&nbsp;- 用户管理 | `system/user/` | 管理员账号管理。 |
| &nbsp;&nbsp;- 角色权限 | `system/role/`, `system/menu/` | RBAC 权限体系。 |
| **认证中心 (Auth)** | `auth/` | 登录、注册、Token 刷新、验证码逻辑。 |
| **网盘存储 (Netdisk)** | `netdisk/` | 文件上传、管理、目录结构。 |
| **任务调度 (Tasks)** | `tasks/` | 系统计划任务管理。 |
| **待办事项 (Todo)** | `todo/` | 简单的待办示例模块。 |
| **监控检查 (Health)** | `health/` | 系统健康检查 endpoint。 |
| **SSE 推送** | `sse/` | Server-Sent Events 服务端推送。 |

## 8. 工具与技巧

- **Repl 模式**: 使用 `pnpm repl` 进入交互式环境调试 Service。
- **Swagger**: 访问 `http://localhost:xxxx/api-docs` 查看自动生成的 API 文档。
