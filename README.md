# guangwu skills

guangwu 自用的 Cursor skill 集合。只给 Cursor 用。

## 目录

```text
.cursor-plugin/marketplace.json
plugins/guangwu-skills/.cursor-plugin/plugin.json
plugins/guangwu-skills/skills/<skill-name>/SKILL.md
```

每个 skill 是一个文件夹，里面有 `SKILL.md`。文件夹名必须和 frontmatter 里的 `name` 一致：小写字母、数字、连字符。

可选子目录（需要时再加，不要空着占位）：

- `scripts/`：agent 可以运行的脚本
- `references/`：按需再读的长说明
- `assets/`：模板、图片等静态文件


## 已有 skill

- `antfu`：Vue 3、Nuxt 和 TypeScript 的写法，按 Anthony Fu 的约定改成给 Cursor 用的简体中文说明。见 `plugins/guangwu-skills/skills/antfu/SKILL.md`。

## 新增一个 skill

1. 复制 `plugins/guangwu-skills/skills/skill-template`。
2. 把文件夹改成新的 `name`。
3. 改 `SKILL.md` 的 `name` 和 `description`。`description` 写清什么时候该用它，agent 靠这一行决定要不要用。
4. 把正文换成真正的步骤。`skill-template` 只是占位，不要当正式 skill 用。

## Cursor 怎么加载

这个仓库本身不会被 Cursor 自动扫到。两种用法：

- 装成插件：在 Cursor 的 Customize 里，用 From GitHub Repository 导入 `https://github.com/Gu-Miao/skills`。仓库根上的 `.cursor-plugin/marketplace.json` 是给这次导入用的。
- 只在本机用某一个 skill：把那个 skill 文件夹放到 `~/.cursor/skills/<name>/`。Cursor 启动时会从这里加载，不依赖这个仓库是否被打开。

格式以 Cursor 的 [Agent Skills](https://cursor.com/docs/skills) 为准。