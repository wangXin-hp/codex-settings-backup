# 依赖选择指南

仅在决定安装哪些包、如何选择版本或评估依赖是否必要时读取本文件。

## 版本策略

1. 读取 `engines.node`、`packageManager`、锁文件和已安装框架的主版本。
2. 已有项目优先使用同一主版本及其兼容版本，不在工程化任务中顺带做大版本迁移。
3. 新项目通过包管理器或官方发布信息选择与目标 Node 兼容的稳定版本，避开 preview、beta 和 RC，除非用户明确要求。
4. 遵循项目已有的精确版本或范围风格；无约定时工具链可保存精确版本，运行时库可采用兼容范围，并始终提交锁文件。
5. 安装后检查 peer dependency 警告、弃用提示和实际解析版本。存在冲突时调整组合，不使用忽略 peer dependency 的参数掩盖问题。

## 核心与任务编排

| 能力 | 常用依赖 | 使用条件 |
| --- | --- | --- |
| TypeScript | `typescript`, `@types/node` | 所有 TypeScript 项目；Node 类型只在确有 Node API 时加入 |
| workspace | `pnpm` | 多包项目，或项目已经使用 pnpm |
| 任务编排 | `turbo` | 多工作区存在依赖构建或重复质量任务 |
| 跨平台清理 | `rimraf` | 脚本需要在 Windows/macOS/Linux 一致删除产物 |
| 串并行脚本 | `npm-run-all2` | 原生脚本难以清晰表达多个子任务时；不要仅为一个命令引入 |

不要在单应用中默认安装 Turbo，也不要同时保留多个包管理器的锁文件。

## 代码质量与提交规范

基础组合可包括：

- ESLint Flat Config：`eslint`、`@eslint/js`、`typescript-eslint`、`globals`。
- 格式化：`prettier`。可以独立运行 Prettier；仅当团队明确要求把格式错误呈现为 ESLint 错误时加入 `eslint-plugin-prettier`。
- 导入排序：`eslint-plugin-simple-import-sort`，仅在团队接受自动排序时启用。
- React 规则：`eslint-plugin-react-hooks`、`eslint-plugin-react-refresh`，仅用于 React/Vite 源码。
- 暂存文件检查：`lint-staged` + `husky`。
- Conventional Commits 强制校验：`@commitlint/cli`、`@commitlint/config-conventional`。
- 新项目要求完整、易用的提交规范时，同时加入 `commitizen`、`cz-git`，提供 `pnpm commit` 交互入口；cz-git 不能代替 Commitlint，因为开发者仍可绕过交互命令直接执行 `git commit`。
- 拼写检查：`cspell`；仅扫描源文件和文档，并提供项目自定义词典。
- CSS 检查：`stylelint` 加一个明确的共享配置；只有项目确实需要独立 CSS 规则时加入。

每一项都必须同时具有配置、脚本和明确扫描范围。不要为了“依赖齐全”安装未启用的工具。

提交规范启用后至少验证三条路径：交互入口能读取 cz-git 配置、合法 type/scope 能通过 Commitlint、未知 type/scope 会被 `commit-msg` 门禁拒绝。

## React + Vite 前端

核心组合：`react`、`react-dom`、`vite`、`@vitejs/plugin-react` 及匹配的 React 类型包。

按需求选择：

- 路由：`react-router-dom`
- 服务端状态：`@tanstack/react-query`
- HTTP：优先平台 `fetch`；需要拦截器等能力时使用 `axios`
- 表单与校验：`react-hook-form`、`@hookform/resolvers`、`zod`
- 样式：`tailwindcss`、`postcss`、`autoprefixer`；动画、合并类名和组件原语仅在 UI 方案需要时加入
- 图表、日期、图标等属于产品依赖，不作为工程化默认项

不自动复制代理端口、路径别名或 UI 组件库；它们必须来自目标项目需求。

## NestJS 后端

核心组合：`@nestjs/common`、`@nestjs/core`、目标 HTTP 平台适配器、`reflect-metadata`、`rxjs`，开发时配套 `@nestjs/cli`、`@nestjs/testing`。

按需求选择：

- 配置：`@nestjs/config`
- DTO 校验：`class-validator`、`class-transformer`
- OpenAPI：`@nestjs/swagger` 及对应 UI
- 数据库：选择一种 ORM/客户端及目标驱动，例如 TypeORM + PostgreSQL；没有持久化需求则不安装
- 鉴权：Passport/JWT 仅在身份验证需求存在时加入
- 队列、定时任务、邮件、上传、Kafka、ClickHouse 均为业务能力，不是后端工程化基线

## SDK 与公共包

推荐 `tsup` 转译 ESM/CJS，使用 `tsc` 单独生成声明文件。只有确需发布双模块消费者时才同时输出两种格式；纯 ESM 项目可简化为 ESM。

发布包至少验证：

- `package.json` 的模块制式与生成文件扩展名一致；
- `main`、`module`、`types` 和 `exports` 路径存在；
- 外部依赖没有被意外打包或遗漏；
- sourcemap 与声明映射符合发布策略；
- 从临时消费者执行 ESM 导入、CJS 引用和 TypeScript 类型解析。

## 测试

- Vite 前端和普通 TypeScript 包优先考虑 Vitest；React 组件按需配套 Testing Library 和 DOM 环境。
- NestJS 项目可使用 Jest + `@nestjs/testing`；HTTP 集成测试按需加入 `supertest` 及类型包。
- 无论选择何者，都要提供测试配置、`test` 脚本、必要环境和至少一个能证明配置可运行的测试，或明确说明本次不建立测试体系。

## 可选基础设施

Docker Compose、PostgreSQL、Redis、Kafka、ClickHouse 只在需求明确时加入。应用客户端库与容器服务应一一对应；本地环境的版本、端口、卷和健康检查需要显式配置。秘密只通过未提交的环境文件或秘密管理系统注入，并提供不含真实值的 `.env.example`。
