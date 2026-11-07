# 配置配方

仅在实际创建或修改对应配置时读取。以下是中性基线，必须根据目标项目的包管理器、模块制式、目录和现有风格改写，不能整段盲目复制。

## pnpm workspace

```yaml
packages:
  - "apps/*"
  - "packages/*"
  - "demos/*"
```

不存在某类目录时删除对应 glob。内部包依赖使用 `workspace:*`。根 `package.json` 标记为 `private: true`，并用 `packageManager` 固定实际采用的 pnpm 版本。

## Turbo

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", "build/**"]
    },
    "typecheck": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    "lint": {
      "outputs": []
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "build:watch": {
      "cache": false,
      "persistent": true
    }
  },
  "cacheDir": ".turbo/cache"
}
```

删除不存在的任务和产物。若构建工具使用其他输出目录，必须同步修改 `outputs`。

根脚本可采用以下语义，不要求逐字一致：

```json
{
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "typecheck": "turbo typecheck",
    "lint": "turbo lint",
    "test": "turbo test",
    "clean": "turbo clean && rimraf .turbo/cache",
    "prepare": "husky"
  }
}
```

确保每个被 Turbo 调用的工作区都定义相应脚本，或通过任务过滤策略排除。

## TypeScript 配置层次

通用根配置示例：

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitReturns": true,
    "isolatedModules": true,
    "sourceMap": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

浏览器配置在此基础上增加 `lib: ["ES2022", "DOM", "DOM.Iterable"]`、`module: "ESNext"`、`moduleResolution: "Bundler"`、`jsx: "react-jsx"` 和 `noEmit: true`。

NestJS 配置根据实际运行时选择 CommonJS 或 NodeNext，并启用 `experimentalDecorators`、`emitDecoratorMetadata`、声明输出目录和增量构建。不要为了兼容参考项目而默认关闭 `strictNullChecks` 或 `noImplicitAny`。

SDK 声明配置示例：

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "declaration": true,
    "declarationMap": true,
    "emitDeclarationOnly": true,
    "outDir": "build/types"
  },
  "include": ["src"]
}
```

## ESLint Flat Config

基础 TypeScript 配置示例：

```js
import eslint from '@eslint/js'
import globals from 'globals'
import tseslint from 'typescript-eslint'

export default tseslint.config(
    { ignores: ['**/dist/**', '**/build/**', '**/coverage/**'] },
    eslint.configs.recommended,
    ...tseslint.configs.recommended,
    {
        files: ['**/*.{ts,tsx}'],
        languageOptions: { globals: globals.node },
    }
)
```

仅服务端文件使用 `globals.node`；浏览器文件改用 `globals.browser`。React 项目再加入 hooks 和 refresh 插件，并限制到前端目录。若项目需要类型感知规则，配置 `parserOptions.projectService` 并验证性能和项目边界。

不要在全局忽略所有 `.js` 或 `.mjs`，否则配置文件和 JavaScript 源码可能完全绕过检查。

## Prettier 与暂存检查

Prettier 风格应优先继承现有项目。新项目可从小型配置开始：

```json
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "es5",
  "endOfLine": "lf"
}
```

`lint-staged` 可以放在根 `package.json`：

```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,yaml,yml,css,scss}": ["prettier --write"]
  }
}
```

只有安装并配置 Stylelint 后才把 `stylelint --fix` 加入 CSS 流程。暂存检查中不要运行会修改整个仓库的命令。

## Husky 与提交规范

新项目需要完整提交体验时安装：

```sh
pnpm add -D -w husky lint-staged @commitlint/cli @commitlint/config-conventional commitizen cz-git
```

根 `package.json` 同时提供交互命令和 Commitizen adapter：

```json
{
  "scripts": {
    "prepare": "husky",
    "lint:staged": "lint-staged",
    "commit": "git-cz"
  },
  "config": {
    "commitizen": {
      "path": "node_modules/cz-git"
    }
  }
}
```

`commitlint.config.cjs` 可同时承载强制规则和 cz-git prompt。scope 必须根据目标项目的真实边界生成，不复制示例值：

```js
const scopes = ['repo', 'frontend', 'backend', 'docs', 'deps']

/** @type {import('cz-git').UserConfig} */
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'scope-enum': [2, 'always', scopes],
    'header-max-length': [2, 'always', 100],
  },
  prompt: {
    types: [
      { value: 'feat', name: 'feat: 新增功能' },
      { value: 'fix', name: 'fix: 修复缺陷' },
      { value: 'docs', name: 'docs: 文档变更' },
      { value: 'refactor', name: 'refactor: 代码重构' },
      { value: 'test', name: 'test: 测试变更' },
      { value: 'build', name: 'build: 构建或依赖变更' },
      { value: 'ci', name: 'ci: 持续集成变更' },
      { value: 'chore', name: 'chore: 其他维护变更' },
    ],
    scopes,
    allowCustomScopes: false,
    allowEmptyScopes: true,
    useEmoji: false,
    maxHeaderLength: 100,
  },
}
```

`.husky/pre-commit`：

```sh
pnpm lint:staged
```

`.husky/commit-msg`：

```sh
pnpm exec commitlint --edit "$1"
```

根脚本提供 `lint:staged`，`prepare` 执行 `husky`。Commitlint 从 `@commitlint/config-conventional` 开始；只有 scope 能从当前 workspace 稳定推导时才限制 scope 枚举。`pnpm commit` 负责引导输入，`commit-msg` hook 才是不可绕过的项目内校验层。

Windows 项目还应使用 `.gitattributes` 保证 Shell hooks 保持 LF，例如 `* text=auto eol=lf`，并仅为 `.bat`、`.cmd` 指定 CRLF。

非交互验证至少覆盖：

```sh
echo "feat(frontend): add login form" | pnpm exec commitlint
echo "feature(unknown): invalid message" | pnpm exec commitlint
```

第一条应通过，第二条应失败；另外实际运行一次 `pnpm commit`，确认能看到配置的 type 和 scope。

## cspell

```json
{
  "version": "0.2",
  "useGitignore": true,
  "caseSensitive": false,
  "files": ["apps/**/*.{ts,tsx,js,jsx,css,md}", "packages/**/*.{ts,tsx,js,jsx,md}"],
  "ignorePaths": ["**/node_modules/**", "**/dist/**", "**/build/**", "**/coverage/**"],
  "dictionaryDefinitions": [
    {
      "name": "project-words",
      "path": "./.cspell/project-words.txt",
      "addWords": true
    }
  ],
  "dictionaries": ["project-words"]
}
```

根据真实目录缩减 `files`；不要通过忽略所有文档或业务源码来让检查“通过”。

## SDK 双模块构建

`tsup.config.ts` 示例：

```ts
import { defineConfig } from 'tsup'

export default defineConfig({
    entry: ['src/index.ts'],
    format: ['esm', 'cjs'],
    bundle: true,
    clean: true,
    sourcemap: true,
    dts: false,
    outDir: 'build',
})
```

若实际生成 `build/index.js` 和 `build/index.cjs`，包清单必须精确引用它们；若采用 `build/esm` 与 `build/cjs` 子目录，则相应调整 tsup 输出。声明文件使用单独的 `tsc -p tsconfig.types.json` 生成。

包清单的导出结构示意：

```json
{
  "main": "./build/index.cjs",
  "module": "./build/index.js",
  "types": "./build/types/index.d.ts",
  "exports": {
    ".": {
      "types": "./build/types/index.d.ts",
      "import": "./build/index.js",
      "require": "./build/index.cjs"
    },
    "./package.json": "./package.json"
  }
}
```

不要假设文件名；构建后检查磁盘产物再确定导出映射。

## Git 忽略项

至少按实际工具考虑：

```gitignore
node_modules/
dist/
build/
coverage/
.turbo/
*.tsbuildinfo
*.log
.env
.env.*
!.env.example
```

不要忽略 `pnpm-lock.yaml`、CI 工作流、全部 YAML、源映射或声明文件，除非它们确实是构建产物且发布策略另有安排。

## Docker Compose

仅在用户需要本地基础设施时添加。配置原则：

- 镜像使用明确且受支持的版本，不使用无约束的 `latest`。
- 容器名称、网络和端口从目标项目命名，不复制参考项目值。
- 密码写为 `${SERVICE_PASSWORD}` 等变量，并在 `.env.example` 提供空值或安全的非秘密说明。
- 为依赖服务配置健康检查；应用服务等待健康状态，而不是只依赖启动顺序。
- 持久卷目录加入 `.gitignore`，但保留 Compose YAML。
- 开发容器与部署 Compose 分开；开发便利配置不得被误当作生产安全配置。

## 验证顺序

从干净依赖状态可行时执行：安装依赖、类型检查、lint、测试、构建。随后检查：

- 锁文件已生成且未被忽略；
- hooks 可执行且引用现有脚本；
- Turbo 命中的工作区和缓存产物符合预期；
- SDK 导出路径全部存在；
- `git diff` 不包含缓存、构建产物、秘密或来源项目标识。
