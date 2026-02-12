# 项目规格说明书 (SPEC.md)

## 1. 技术栈概览

本项目的核心技术选型如下：

- **核心框架:** Vue 3.5+ (Composition API, `<script setup>`)
- **语言:** TypeScript 5.x
- **构建工具:** Vite 6+
- **UI 组件库:** Ant Design Vue 4.x
- **样式方案:** UnoCSS (原子化 CSS) + Less
- **状态管理:** Pinia 3.x (使用 Setup Store 语法)
- **网络请求:** Axios (位于 `src/utils/request`), API 类型通过 `@umijs/openapi` 自动生成
- **Mock 数据:** MSW (Mock Service Worker)
- **包管理器:** pnpm (v10+, 使用 Workspaces)

## 2. 项目架构 (Monorepo)

本项目采用 **Monorepo** 结构，通过 pnpm workspaces 管理。

- **根目录应用 (`src/`)**: 主要的 Vue 3 管理后台应用。
- **`packages/` 目录**:
  - `components`: 共享业务组件库。
  - `vite-plugin-*`: 各种定制的 Vite 插件。
- **`mocks/` 目录**: MSW 服务端 Mock 定义文件。

## 3. 核心开发约定

### 3.1 API 层

- **自动生成**: 后端 API 定义位于 `src/api/backend`，由 `openapi.config.ts` 配置并通过 `pnpm openapi` 命令自动生成。
- **禁止修改**: **切勿** 手动修改 `src/api/backend` 目录下的文件。
- **手动 Mock**: 如果需要手动编写 API 包装函数，请放置在 `src/api` 下的其他目录（如 `src/api/demo`）。

### 3.2 状态管理 (Pinia)

- 必须使用 **Setup Stores** (函数式写法) 而非 Options Stores。
- 示例: `export const useUserStore = defineStore('user', () => { ... })`

### 3.3 样式编写

- **优先 UnoCSS**: 布局、间距、排版优先使用 UnoCSS 工具类 (如 `flex items-center mb-4 text-gray-500`)。
- **样式覆盖**: 只有在处理复杂动画或深度覆盖 Ant Design 样式时，才使用 `<style scoped lang="less">`。

### 3.4 Vue 组件

- 统一使用 `<script setup lang="ts">`。
- 使用 `defineProps<Props>()` 定义类型安全的 Props。

## 4. 常用命令

- `pnpm install`: 安装依赖。
- `pnpm dev`: 启动主应用开发服务器。
- `pnpm build`: 构建生产环境应用。
- `pnpm openapi`: 根据 Swagger 文档生成 API 代码。

## 5. 核心功能 (Features)

### 5.1 权限控制 (Permission / RBAC)

- **实现方式**: 基于 `src/permission` 和 `useUserStore`。
- **颗粒度**: 支持细粒度的按钮/操作级权限。
- **使用**:
  - 指令: `<a-button v-auth="'sysUser.add'">新增</a-button>`
  - 函数: `if (hasPermission('sysUser.add')) { ... }`

### 5.2 国际化 (I18n)

- **库**: `vue-i18n` with lazy loading.
- **文件位置**: `src/locales/lang/*.ts`。
- **配置**: 通过 `useLocaleStore` 管理当前语言。

### 5.3 Mock 数据

- **方案**: MSW (Mock Service Worker)。
- **规则**: 开发环境默认开启拦截。
- **新增 Mock**: 在 `mocks/` 目录下添加 handler，并在 `mocks/index.ts` 中注册（如适用）。

## 6. 开发工作流 (Workflow)

### 6.1 代码提交规范

- 本项目使用 **Husky** + **Commitlint**。
- **Format**: `type(scope): subject` (Angular Convention).
  - 示例: `feat(user): add login page`
  - 常用 Types: `feat`, `fix`, `docs`, `style`, `refactor`, `chore`.
- 提交时会自动触发 Lint 和 Prettier 检查 (`lint-staged`).

### 6.2 API 开发流程

1. 后端更新 Swagger 文档。
2. 运行 `pnpm openapi` 自动更新 `src/api/backend` 下的接口定义。
3. 如果是新业务，直接调用生成的 Hook 或函数；如果是旧业务或特殊需求，在 component 中引入使用。

### 6.3 Monorepo 构建

- 使用 **Nx** 进行任务调度加速构建。
- `packages/` 下的包变更会触发增量构建。
- 运行 `pnpm bootstrap` 初始化依赖。

## 7. 目录结构规范 (Directory Breakdown)

AI 助手在创建新文件时，必须严格遵守以下目录归属规则：

### `src/` (核心源码)

| 路径                     | 说明          | 规则                                                              |
| :----------------------- | :------------ | :---------------------------------------------------------------- |
| **`src/api/backend/`**   | 自动生成 API  | **禁止手动编辑**。由 `pnpm openapi` 生成。                        |
| **`src/api/*/`**         | 手动 Mock/API | 用于存放非自动生成的 API 请求定义。                               |
| **`src/components/`**    | 全局组件      | 见下文 [3.1 组件晋升规则]。                                        |
### `src/views/` (页面视图)

| 路径                     | 说明          | 规则                                                                     |
| :----------------------- | :------------ | :----------------------------------------------------------------------- |
| **`views/*/index.vue`**  | 页面入口      | 如果是复杂页面，必须拆分为子组件。                                       |
| **`views/*/components`** | 私有组件      | 仅在此页面使用的组件。禁止跨视图引用。                                   |
| **`views/*/hooks`**      | 私有 Hook     | 仅在此页面使用的逻辑复用。                                               |
| **`views/*/utils`**      | 私有工具      | 仅在此页面使用的数据处理函数。                                           |

### `src/hooks/` (全局 Composables)
| **`src/store/modules/`** | Pinia 状态    | 业务模块状态，单文件管理 (e.g. `user.ts`)。                       |
| **`src/utils/`**         | 工具函数      | 纯函数工具。`request.ts` 位于此处核心位置。                       |
| **`src/enums/`**         | 枚举定义      | 所有的固定常量枚举应定义在此，禁止在组件内散落。                  |

### 3.1 组件晋升规则 (Avoiding Duplication)

为了避免组件重复，AI 助手在创建组件时必须遵循**"三级晋升"**规则：

1.  **Level 1: 页面级 (Private)**
    *   **位置**: `src/views/xxx/components/`
    *   **适用**: 仅在当前页面使用的独有组件。
    *   **规则**: AI 默认应将新组件创建在此，除非明确要求复用。

2.  **Level 2: 业务通用级 (Shareable Business)**
    *   **位置**: `src/components/business/`
    *   **适用**: 包含特定业务逻辑（如依赖 Store、调用特定 API），且被 **2个以上** 页面模块使用的组件。
    *   **命名**: 建议带业务前缀，如 `SysUserSelect.vue`。

3.  **Level 3: 通用基础级 (Generic UI)**
    *   **位置**: `src/components/basic/`
    *   **适用**: 纯 UI 组件，**严禁** 包含任何业务 API 调用或 Store 依赖。它只接受 `props` 和发出 `emits`。
    *   **规则**: 必须是完全解耦的，可以在任何项目中使用。

### `packages/` (内部私有库)

| 路径                          | 说明       | 规则                                 |
| :---------------------------- | :--------- | :----------------------------------- |
| **`packages/components/`**    | 纯净组件库 | 不依赖主应用业务逻辑的高度通用组件。 |
| **`packages/vite-plugin-*/`** | 构建插件   | 自研 Vite 插件，独立编译。           |

### `mocks/` (Mock 服务)

- 所有 MSW 的 Handler 定义都在这里。
- 按业务模块拆分文件 (e.g. `user.ts`, `list.ts`)。

## 8. 业务模块映射 (Business Domain Map)

为了快速定位业务代码，请参考以下核心模块映射表：

| 业务领域 | 核心路径 (`src/views/...`) | 关联 Store (`src/store/modules/...`) | 备注 |
| :--- | :--- | :--- | :--- |
| **系统管理 (System)** | `system/` | `user.ts`, `dict.ts` | 包含用户、角色、菜单、部门管理等 RBAC 核心逻辑。 |
| &nbsp;&nbsp;- 用户管理 | `system/user/` | `user.ts` | 用户列表、增删改查。 |
| &nbsp;&nbsp;- 角色权限 | `system/role/` | - | 角色与权限分配。 |
| &nbsp;&nbsp;- 菜单管理 | `system/menu/` | - | 路由菜单配置。 |
| &nbsp;&nbsp;- 数据字典 | `system/dict-type/`, `dict-item/` | `dict.ts` | 系统字典配置。 |
| **个人中心 (Account)** | `account/` | `user.ts` | 个人设置、密码修改。 |
| **仪表盘 (Dashboard)** | `dashboard/` | - | 首页分析图表。 |
| **网盘/文件 (Netdisk)** | `netdisk/` | - | 文件上传与管理系统。 |
| **登录认证 (Auth)** | `login/` | `user.ts` | 登录页。 |
| **消息推送 (SSE)** | - | `sse.ts` | 服务端推送相关状态。 |

> **提示**: 在进行新功能开发时，请首先判断归属上述哪个模块。如果是全新的顶级业务，请在 `src/views` 下创建新目录并更新此表。
