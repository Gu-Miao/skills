---
name: antfu
description: 在 Cursor 里编写或修改 Vue 3、Nuxt、TypeScript 项目时使用，包括新项目脚手架、组件、ESLint 和提交前检查。按 Anthony Fu 的约定：script setup、显式 import、shallowRef、@antfu/eslint-config。已有项目先跟仓库里的现有写法。
---

# Antfu 约定

这是给 Cursor Agent 的 skill。来源是 Anthony Fu 的 [antfu skill](https://github.com/antfu/skills/blob/main/skills/antfu/SKILL.md)（2026.09.30）和其中的 [app-development](https://github.com/antfu/skills/blob/main/skills/antfu/references/app-development.md)。

先看当前仓库已经有的配置和写法。已有项目跟现有约定。下面这套只用在新项目，或用户明确要求按这套改的时候。

搭一个新的 TypeScript 项目时，对照 [antfu/starter-ts](https://github.com/antfu/starter-ts)，对齐它的 `package.json` scripts、`eslint.config.js`、`knip.json`、`tsdown.config.ts` 和工作流，不要另起一套目录。

## 代码组织

- 每个源文件只做一件清楚的事。文件变大或管得太宽时拆开。
- 类型和 interface 放进 `types.ts` 或 `types/*.ts`，不要散在实现文件里。
- 常量放进 `constants.ts`。

## 运行环境

能同时在 Node、浏览器和 worker 里跑的代码，就写成与运行环境无关的。

代码绑死在某个环境时，在文件顶部注明：

```ts
// @env node
// @env browser
```

## TypeScript

- 能标明返回类型就标明。
- 复杂类型不要写在行内，抽成 `type` 或 `interface`。
- 新项目的 `tsconfig` 用下面这份：

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true
  }
}
```

## 显式，不要靠魔法

人和 agent 都应该不用跑工具就能看出每个名字从哪来。

- 用显式 `import`。框架如果提供自动导入（Nuxt、Nitro），新项目关掉，见下面的 Nuxt 一节。
- 默认用相对路径（`./foo`、`../bar`）。`@/`、`~/`、`#imports` 这类别名只在项目里已经配好时使用。不要为新代码再加别名。

## 注释

代码能看懂就不要注释。注释写为什么，不写代码在做什么。

## 测试（Vitest）

- 测试文件跟源文件放在同一目录：`foo.ts` 对应 `foo.test.ts`。
- 用 `describe` / `it`，不用 `test`。
- 复杂输出用 `toMatchSnapshot`。跟语言相关的快照用 `toMatchFileSnapshot`，并写明路径。

## 包管理命令

用 `@antfu/ni`，不要把 pnpm、npm、yarn 写死。

| 命令 | 作用 |
| --- | --- |
| `ni` | 安装依赖 |
| `ni <pkg>` / `ni -D <pkg>` | 加依赖 / 开发依赖 |
| `nr <script>` | 跑脚本 |
| `nu` | 升级依赖 |
| `nun <pkg>` | 卸依赖 |
| `nci` | 按锁文件干净安装 |
| `nlx <pkg>` | 执行一个包 |

查 npm 上的最新版本时用 `fast-npm-meta`，不要为了看一个版本号去拉整个 registry 包。

```bash
nlx fast-npm-meta version vite
nlx fast-npm-meta version "nuxt@^3.5"
nlx fast-npm-meta version vite nuxt vue
```

只想看最新版本时，优先用它，而不是 `npm view <pkg> version`。

## ESLint

```js
// eslint.config.js
import antfu from '@antfu/eslint-config'

export default antfu({
  type: 'lib', // 应用用 'app'（默认）
  pnpm: true, // package.json 里强制使用 pnpm catalog
  antislop: true, // 标出重复、啰嗦的 AI 风格代码，并禁止显式 any
})
```

`antislop: true` 一直打开。它需要开发依赖 `eslint-plugin-slop` 和 `eslint-plugin-sonarjs`。

更细的选项以 [antfu 的 ESLint 参考](https://github.com/antfu/skills/blob/main/skills/antfu/references/antfu-eslint-config.md) 为准，不要自己发明规则开关。

## Knip

用 Knip 查没用到的文件、导出和依赖。ESLint 一次只看一个文件，看不全。

```json
{
  "$schema": "https://unpkg.com/knip@6/schema.json",
  "project": ["src/**/*.ts"],
  "ignoreDependencies": []
}
```

Knip 报出来的，优先删掉。只有在 `package.json` 脚本之外被调用的工具（例如 `taze`）才放进 `ignoreDependencies`。

## 提交前必须过的检查

每个项目都要有一个 `ci` 脚本，把检查串在一起：

```json
{
  "scripts": {
    "build": "tsdown",
    "lint": "eslint --cache",
    "typecheck": "tsc",
    "knip": "knip",
    "test": "pnpm run build && vitest",
    "ci": "pnpm run lint && pnpm run typecheck && pnpm run knip && pnpm run test --run"
  }
}
```

在 Cursor 里准备提交之前，先跑 `nr lint --fix` 做格式化，再跑 `nr ci`，通过了才能提交。`ci` 失败时修原因，不要把规则关掉，也不要加 ignore。

## Git hooks

用 `simple-git-hooks` 加 `nano-staged`。pre-commit 跑完整的 `ci`。`prepare` 里同时装 agent skills：

```json
{
  "scripts": {
    "prepare": "git config core.hooksPath .githooks && simple-git-hooks && skills-npm"
  },
  "simple-git-hooks": {
    "pre-commit": "pnpm i --frozen-lockfile --ignore-scripts --offline && pnpm run ci && pnpx nano-staged"
  },
  "nano-staged": {
    "*": "eslint --fix --no-warn-ignored"
  }
}
```

把 `.githooks` 放进 `.gitignore`。在 `pnpm-workspace.yaml` 的 `allowBuilds` 里加上 `simple-git-hooks: true`。

`skills-npm` 会在安装时把依赖包里的 skill 链到 agent 的 skills 目录。把它装成开发依赖，按上面的 `prepare` 接上。把 `skills/npm-*` 放进 `.gitignore`。生成的 `skills-npm-lock.json` 要提交。

## 发布和版本目录

发布优先用 npm Trusted Publishing（OIDC），不要在 CI 里放 `NPM_TOKEN`。`v*` tag 触发 `release.yml`，并给 `id-token: write`。`nr release`（`bumpp`）只负责升版本、打 tag、推上去。细节以 [setting-up](https://github.com/antfu/skills/blob/main/skills/antfu/references/setting-up.md) 为准。

pnpm 的版本放在 `pnpm-workspace.yaml` 的具名 catalog 里，不要用默认 catalog。

| Catalog | 用途 |
| --- | --- |
| `prod` | 生产依赖 |
| `inlined` | 会被打包进去的依赖 |
| `dev` | 开发工具 |
| `frontend` | 前端库 |

名字可以按项目改，但不要把所有版本堆进一个默认 catalog。

## 选 Vite 还是 Nuxt

| 场景 | 选择 |
| --- | --- |
| SPA、只在客户端跑、库的 playground | Vite + Vue |
| SSR、SSG、要 SEO、文件路由、API 路由 | Nuxt |

## Nuxt：新项目关掉自动导入

新 Nuxt 项目在 `nuxt.config.ts` 里关掉应用侧和 Nitro 侧的自动导入：

```ts
export default defineNuxtConfig({
  imports: {
    autoImport: false,
  },
  components: {
    dirs: [],
  },
  nitro: {
    imports: false,
  },
})
```

框架 API 从 `#imports` 显式导入：

```ts
import { computed, ref } from '#imports'
```

| 选项 | 效果 |
| --- | --- |
| `imports.autoImport: false` | 不再自动导入 composable、utils，以及 `ref` 这类框架 API |
| `components.dirs: []` | 不再自动导入组件 |
| `nitro.imports: false` | 服务端也不再自动导入 |

单独的 Nitro 项目默认就是 `imports: false`，保持关掉，不要打开。

Nuxt 自带的 `~/`、`@/`、`#imports` 已经配好，可以用。除此之外不要再加新的路径别名，其他地方用相对路径。

## Vue 3 组件

| 约定 | 做法 |
| --- | --- |
| script | 一律 `<script setup lang="ts">` |
| 状态 | `shallowRef()` 优先于 `ref()` |
| 对象 | 用 `ref()`，不用 `reactive()` |
| 样式 | UnoCSS |
| 工具函数 | VueUse，有现成的就不要手写 |

props 和 emits 用 interface。默认值用 `withDefaults`，不要用响应式 props 解构。

```vue
<script setup lang="ts">
interface Props {
  title: string
  count?: number
}

interface Emits {
  (e: 'update', value: number): void
  (e: 'close'): void
}

const props = withDefaults(defineProps<Props>(), {
  count: 0,
})

const emit = defineEmits<Emits>()
</script>
```

组件用 Storybook 把每种状态写成一个 story，让组件没有副作用、状态可预期。story 测试放进 CI，优先用 `@storybook/addon-vitest`，让 story 跟着现有的 vitest 一起跑。