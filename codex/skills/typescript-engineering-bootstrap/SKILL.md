---
name: typescript-engineering-bootstrap
description: 自适应初始化或完善 TypeScript 项目的工程化配置，适用于 React/Vite 前端、NestJS 后端、tsup SDK/公共包以及 pnpm/Turborepo monorepo。用于项目脚手架、构建、类型检查、代码质量、提交规范、测试和可选容器环境；不用于仅实现业务功能或强制替换既有技术栈。
---

# TypeScript 工程化初始化

为目标项目建立可维护、可执行的工程化基线。借鉴成熟组合，但根据目标项目裁剪，不复刻来源项目的品牌、业务和历史缺陷。

## 工作方式

1. 先检查目标目录中的 `package.json`、锁文件、Node 版本声明、workspace 配置、构建工具、TypeScript 配置、质量工具、测试和 Git hooks。已有项目还要检查未提交修改，避免覆盖用户工作。
2. 判断项目属于单体前端、单体后端、SDK/公共包或 monorepo。需要选择目录和任务编排时，读取 [references/architecture.md](references/architecture.md)。
3. 保留用户已经选定的包管理器、框架、模块制式、格式风格和目录约定。只有新建多包 TypeScript 项目且用户未指定时，才默认 pnpm workspace + Turborepo。
4. 只选择当前需求实际需要的依赖。确定依赖组合和版本时，读取 [references/dependencies.md](references/dependencies.md)。
5. 在修改前说明将新增或调整的配置。对已有项目做增量合并；不要用模板整体覆盖配置。
6. 需要具体配置形态时，读取 [references/config-recipes.md](references/config-recipes.md)，并按目标项目改写路径、脚本和模块语法。
7. 安装依赖并生成对应包管理器的锁文件。运行实际存在的 `typecheck`、`lint`、`test` 和 `build`，修复由本次工程化变更引入的问题。
8. 最后检查变更范围、可复现性和敏感信息，并向用户说明启用的能力、刻意未启用的可选项以及验证结果。

## 必须保持的约束

- 单应用不因“工程化”而自动升级成 monorepo，也不引入无消费者的 Turbo、NestJS、数据库或消息队列。
- 新增脚本必须有对应依赖和配置；新增依赖必须能说明消费者。尤其不要留下不可执行的 `lint:style`、`test` 或提交检查脚本。
- 新项目启用完整提交规范时，成套配置 `commitizen` + `cz-git` 交互入口、Commitlint 强制校验、Husky hooks 和 lint-staged；不得只安装依赖或只生成提示界面。已有项目尊重其现有提交工具。
- Commitlint 必须由 `.husky/commit-msg` 调用；暂存文件检查由 `.husky/pre-commit` 调用。钩子命令应复用 `package.json` 脚本，并明确 cz-git 负责生成、Commitlint 负责强制约束。
- Turbo 的构建任务必须声明真实产物，如 `dist/**` 或 `build/**`；持久开发任务禁用缓存。内部包使用 `workspace:*`，并让任务通过 `dependsOn: ["^build"]` 按依赖顺序构建。
- SDK 的 JavaScript 转译与声明文件生成都要可验证。`main`、`module`、`types` 和 `exports` 必须指向真实产物，并同时验证 ESM 导入、CJS 引用和类型解析。
- 测试框架、配置、测试脚本和测试环境必须成套出现。没有测试需求时不要只安装测试依赖制造虚假能力。
- `.gitignore` 不得用 `*.yaml` 等宽泛规则误伤锁文件或 CI 配置。锁文件必须提交。
- 容器配置通过环境变量和 `.env.example` 表达可配置项；不得写入口令、令牌、固定内网地址、个人路径或生产凭据。
- 不复制项目名、私有 scope、版权头、作者、许可证或业务端口。新项目缺少这些信息时保持中性或向用户确认。
- 动态选取与目标 Node、包管理器和框架兼容的稳定版本；遵循现有版本策略，并用锁文件固定解析结果。不要盲目沿用参考项目中的历史版本。

## 默认完成标准

- 全新项目可从干净检出安装依赖，并能执行其声明的检查与构建脚本。
- 已有项目的原有命令和模块解析行为不被无意破坏。
- 配置之间的路径、脚本名、输出目录和依赖相互一致。
- Git diff 中没有业务模板残留、生成产物、缓存、秘密或机器专用值。
