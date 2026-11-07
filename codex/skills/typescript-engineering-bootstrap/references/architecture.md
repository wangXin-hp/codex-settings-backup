# 架构与任务编排

仅在需要判断项目形态、设计目录或配置 workspace/Turbo 时读取本文件。

## 先识别项目形态

| 形态 | 识别线索 | 默认工程化选择 |
| --- | --- | --- |
| React 前端 | React 入口、Vite 配置、浏览器 API | Vite、客户端 TypeScript、React ESLint 规则；不引入 Turbo 或后端依赖 |
| NestJS 后端 | `@nestjs/*`、`nest-cli.json`、装饰器 | Nest CLI、服务端 TypeScript、Node ESLint 规则；数据服务按需求选择 |
| SDK/公共包 | `exports`、发布产物、被其他包引用 | tsup 生成 JS，`tsc` 生成声明文件，验证 ESM/CJS/类型导出 |
| monorepo | 多个独立应用或包、内部依赖 | pnpm workspace + Turbo；应用放 `apps/`，可复用包放 `packages/`，示例放 `demos/` |
| 已有混合项目 | 已有 workspace、框架或构建工具 | 保持现有布局和工具，只补足明确缺口 |

不要仅凭仓库目录多就引入 monorepo。至少存在两个需要独立安装、构建或发布的工作区时才值得采用。

## 推荐目录边界

新建 monorepo 且用户没有既定约定时采用：

```text
apps/
  frontend/<app>/
  backend/<app>/
packages/
  <shared-package>/
demos/
  <example>/
```

- `apps/` 是可部署单元，依赖 `packages/`，公共包不得反向依赖应用。
- `packages/` 按可复用能力拆分，不按随意的文件类型拆分。
- `demos/` 只用于真实演示或消费验证；没有此需求则省略。
- 每个工作区拥有自己的 `package.json`、构建入口和必要的局部配置；可共享配置放根目录或专用配置包。
- 内部依赖写为 `workspace:*`，避免开发时意外解析到注册表中的同名包。

## 根配置职责

根目录负责：

- 固定包管理器及其版本范围；
- 声明 workspace 范围；
- 托管共享 TypeScript、ESLint、Prettier、拼写和提交规范；
- 提供统一的 `dev`、`build`、`typecheck`、`lint`、`test`、`clean` 入口；
- 通过 Turbo 编排任务，而不重复实现每个工作区的实际构建命令。

工作区脚本负责真正的构建、测试和清理。根脚本应使用 Turbo 或包管理器过滤器调用它们。

## Turbo 任务原则

- `build` 使用 `dependsOn: ["^build"]`，确保依赖包先构建。
- `build.outputs` 与实际工具一致：应用常见为 `dist/**`，SDK 常见为 `build/**`。若两者并存可同时声明。
- `dev`、`build:watch` 等长期进程设置 `cache: false` 和 `persistent: true`。
- `typecheck`、`lint`、`test` 可缓存，但只有在其输入不会读取未声明外部状态时才启用。
- 测试覆盖率目录应列为输出；不得把源码目录或锁文件列为输出。
- 新增环境变量影响任务结果时，在 Turbo 配置中声明，避免错误复用缓存。

## TypeScript 分层

- 根 `tsconfig.json` 只放跨环境安全的严格性和诊断选项。
- 浏览器配置增加 DOM、JSX、bundler 模块解析和 `noEmit`。
- Node/Nest 配置增加目标模块制式、装饰器元数据和构建输出设置。
- SDK 的日常类型检查与声明生成可使用不同配置：前者 `noEmit`，后者仅生成声明文件到 `build/types`。
- 不把服务端为兼容旧代码关闭的严格性选项传播给前端或新 SDK。

## 增量改造规则

已有项目中先建立差异表：已有能力、缺失能力、冲突配置、拟议动作。默认行为：

- 有锁文件时沿用对应包管理器，不另建第二种锁文件。
- 有 ESLint/Prettier 时合并现有规则和忽略项，不替换团队风格。
- 有构建工具时先扩展它；仅在无法满足明确目标时才提出迁移。
- 有未提交修改时避免触碰重叠文件；无法安全合并则停下并说明冲突。
- 只为当前项目实际存在的语言和文件类型配置检查器。
