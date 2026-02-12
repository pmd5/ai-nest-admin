# AI 助手行为准则 (Copilot Instructions)

> **Important**: This file is referenced by `.github/copilot-instructions.md`.

## 1. 技术规范 (Technology Standards)

- **核心框架**: NestJS 11+ (Fastify Adapter).
- **语言**: TypeScript (Strict Mode).
- **数据库 ORM**: TypeORM (Active Record Pattern 优先).
- **验证**: `class-validator` + `class-transformer`.
- **API 文档**: `@nestjs/swagger`.

### 1.1 代码风格
- **控制器 (Controller)**: 仅处理 HTTP 请求/响应，不要包含复杂业务逻辑。
- **服务 (Service)**: 包含核心业务逻辑。
- **Entity**: 使用 TypeORM 装饰器定义数据库模型，继承 `CommonEntity`。
- **DTO**: 数据传输对象必须使用 `class-validator` 装饰器进行验证。

## 2. API 定义与Swagger

- **Swagger装饰器**: 所有 Controller 方法**必须**包含详细的 Swagger 描述：
  - `@ApiOperation({ summary: '...' })`: 接口描述。
  - `@ApiOkResponse({ type: MyResponseDto })`: 成功响应类型。
  - `@ApiResult()`: 自定义装饰器，用于统一包装响应结构。

- **URL 规范**: 使用 RESTful 风格，例如 `GET /system/users` 而非 `GET /system/user/list`.

## 3. 模块化意识 (Modularity)

- **模块结构**: 遵循 NestJS 模块化设计。每个业务领域应是一个独立的 Module (e.g., `UserModule`, `AuthModule`)。
- **依赖注入**: 始终使用依赖注入 (Constructor Injection) 获取 Service 或 Repository。
- **Circular Dependency**: 避免循环依赖，必要时使用 `forwardRef()`。

## 4. 数据库操作 (Database)

- **查询**: 简单查询使用 Find Options (`repo.findOne({ where: { ... } })`)。
- **复杂查询**: 使用 `createQueryBuilder`，注意 SQL 注入风险。
- **事务**: 涉及多表更新时，**必须**使用 EntityManager 或 QueryRunner 的事务机制。

## 5. 语言

- 除非用户另有要求，否则请使用**中文**进行解释和注释。
- 代码中的注释 (Comments) 也应尽可能使用清晰的中文。

## 6. 自我文档化 (Self-Documentation)

- **保持文档同步**:
  - 每当你创建了新的 Top-Level Module (在 `src/modules` 下创建新文件夹) 时，**必须**检查并更新 `openspec/SPEC.md` 中的 **"8. 业务模块映射"** 表格。

## 7. 常见编程模式 (AI Skills)

### 7.1 DTO 与 Validation
- **必须**为 Request Body 定义 DTO 类。
- **严禁**使用 `any` 作为参数类型。
- 示例:
  ```typescript
  export class CreateUserDto {
    @ApiProperty()
    @IsString()
    @MinLength(4)
    username: string;
  }
  ```

### 7.2 异常处理
- 不要直接抛出 Error，使用 NestJS 内置的 HTTP 异常或自定义异常。
- 示例: `throw new BusinessException(ErrorEnum.USER_NOT_FOUND);` 或 `throw new NotFoundException('User not found');`

### 7.3 配置文件
- 不要硬编码配置 (Secrets, URLs)，始终通过 `ConfigService` 读取环境变量。
- 示例: `this.configService.get('database.host')`

## 8. 专项技能 (Skills)

### 8.1 🗄️ 数据库专家 (TypeORM)
- **场景**: 定义实体或编写迁移。
- **要求**:
  - 所有的 Entity 应继承 `src/common/entity/common.entity.ts` (包含 created_at, updated_at)。
  - 谨慎处理仅仅为了显示使用的虚拟字段 (`@VirtualColumn` 或 `@AfterLoad`)。
  - 在修改表结构后，提醒用户运行 Migration 生成命令。

### 8.2 ⚡ 缓存专家 (Redis)
- **场景**: 性能优化或临时存储。
- **要求**:
  - 使用 `@liaoliaots/nestjs-redis` 或 `InjectRedis()` 注入 Redis 客户端。
  - Key 命名必须规范，通常格式为 `project:module:id` (e.g., `nest-admin:auth:token:123`).
  - 始终设置过期时间 (TTL) 以防止内存泄漏。

### 8.3 🔒 安全卫士 (Security)
- **场景**: 鉴权与数据访问。
- **要求**:
  - 敏感操作必须检查 `Current User` 的权限。
  - 密码**必须**通过 `bcrypt` 或 `argon2` 哈希存储，**严禁**明文存储。
  - 敏感字段 (如 password, salt) 在 Entity 中应标记 `ws-select: false` 或在 Transformer 中排除。

### 8.4 📝 Git 提交专家 (Git Commit)
- **必须**遵循 Angular Commit Convention。
- **Type 类型**:
  - `feat`: 新功能 (Features)
  - `fix`: 问题修复 (Bug Fixes)
  - `docs`: 文档 (Documentation)
  - `style`: 格式 (不影响代码运行)
  - `refactor`: 重构
  - `perf`: 性能
  - `test`: 测试
  - `chore`: 构建/依赖/杂项
- **Commit 示例**:
  - `feat(user): 新增用户注册接口`
  - `fix(auth): 修复 JWT 过期时间计算错误`
  - `chore(deps): 升级 nestjs 核心库`

## 9. 代码质量与重构 (Refactoring)

- **DRY 原则**: 避免在 Controller 和 Service 中重复逻辑。通用逻辑应提取到 Utils, Decorators 或 Interceptors。
- **类型安全**: 尽量减少 `as any` 的使用，利用 TypeScript 的类型推断。
