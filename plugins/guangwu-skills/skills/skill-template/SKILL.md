---
name: skill-template
description: 占位模板，不是真实流程。要新 skill 时复制这个文件夹再改，不要直接调用。
disable-model-invocation: true
---

# Skill 模板

这是仓库骨架里的占位 skill。它没有具体任务。

## 做成一个真 skill

1. 复制整个 `skill-template` 文件夹，文件夹名改成新 skill 的名字。
2. 把上面的 `name` 改成同一个名字，并把 `description` 写成「做什么，以及什么时候用」。
3. 删掉 `disable-model-invocation`，除非这个 skill 只允许手动用 `/名字` 调用。
4. 用下面的步骤换掉这段说明。

## 步骤

1. 写 agent 要做的事，按顺序，一步一行。
2. 只写可复用的做法。具体仓库、频道、账号不要写进 skill。
3. 脚本放 `scripts/`，长文档放 `references/`，静态文件放 `assets/`，并在这里用相对路径点到它们。