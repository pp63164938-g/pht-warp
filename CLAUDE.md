# PHT Warp 二次开发规范

本仓库基于 [Warp](https://github.com/warpdotdev/warp) 二次开发。  
官方代码保持可跟踪；我们自己的功能独立分支开发，确认后再合入二次开发主干。

## 远程

| 远程 | 地址 | 用途 |
| --- | --- | --- |
| `upstream` | `https://github.com/warpdotdev/warp.git` | 官方源，只读跟踪 |
| `origin` | `https://github.com/pp63164938-g/pht-warp.git` | 我们的产品仓库 |

禁止向 `upstream` 执行 `push`。  
禁止把官方 `origin` 误当成我们的仓库。

## 分支模型

| 分支 | 职责 | 谁可以改 |
| --- | --- | --- |
| `official` | 始终与官网 `master` 一致，只用来同步上游新功能 | 只允许快进合并 `upstream/master` |
| `main` | 二次开发主干。汇合官网更新和已确认的自研功能 | 只合入已确认的 `official` 更新和已批准的功能分支 |
| `feat/<功能名>` | 单个新功能的独立分支 | 日常开发都在这里进行 |

硬性规则：

1. **一个功能一个分支。** 不要在同一个功能分支里夹带无关改动。
2. **功能分支默认不合入 `main`。** 必须等用户明确同意后才能合并。
3. **`official` 禁止承载自研提交。** 规范、中文文案、新功能都不进 `official`。
4. **`main` 不要直接开发。** 先从 `main` 拉 `feat/<功能名>`，做完再申请合入。
5. **同步官网和开发功能分开提交。** 不要把 `merge official` 和新功能写进同一次提交。

### 从官网更新

```powershell
git fetch upstream
git checkout official
git merge --ff-only upstream/master
git push origin official

git checkout main
git merge official
# 解决冲突时优先保住 PHT-PATCH 块和 crates/pht_* 
git push origin main
```

`official` 只能快进。出现分叉时先停下来问用户，禁止在 `official` 上做普通功能提交后再强推。

### 新功能开发

```powershell
git checkout main
git pull origin main
git checkout -b feat/<功能名>
# 开发、中文提交
git push -u origin feat/<功能名>
```

合入 `main` 前必须：

1. 向用户说明改了什么、影响范围、有没有动官方文件。
2. 等用户明确说「可以合并」或等价批准。
3. 只合并这一个功能分支，不顺手带其他分支。

用户只说「继续做」「先提交到功能分支」，不等于批准合入 `main`。

## 语言

我们自己新增的内容优先使用中文，包括：

- Git 提交说明
- 项目规则、方案、文档
- 自研代码注释
- 自研功能的用户可见文案

边界：

- 不翻译、不改写官方已有英文，除非当前任务就是 i18n / 中文切换。
- 官方文件里为了接入自研功能而必须改的少量挂钩，注释用中文，并包进 `PHT-PATCH` 标记。
- 提交说明优先中文；不要用英文 commit message 描述自研改动。
- 功能分支名可用英文短名，例如 `feat/i18n-switch`、`feat/add-to-conversation`。

### 提交说明

格式：

```text
<动词><对象>：<补充说明，可选>
```

示例：

- `初始化二次开发规范：分支模型、中文提交和功能合入门禁`
- `新增语言切换：设置页支持中英文`
- `同步官方 master 到 official`

禁止：

- 英文为主的自研提交说明
- 一次提交混入多个无关功能
- 未获批准就把功能分支改动提交到 `main`

## 代码隔离

目标：能加自己的功能，也能持续合并官网更新。

1. **尽量加文件，少改官方文件。**
2. 自研功能优先放独立 crate，例如 `crates/pht_i18n`、`crates/pht_overlay`。
3. 必须改官方文件时，改动要短、要集中，并用标记包住：

```rust
// PHT-PATCH-START: 简要说明
...
// PHT-PATCH-END
```

4. 自研功能应可开关。关闭后尽量回到官方行为。
5. 禁止顺手重构 WarpUI、设置宏、主题系统或大规模重命名官方 crate。
6. 禁止为了中文切换或新功能全仓替换官方硬编码英文。先做最小接入点。

## 当前自研功能

| 功能 | 分支 | 合入 `main` |
| --- | --- | --- |
| 项目规范 / 二次开发工作流 | 直接在 `main` 初始化 | 初始化阶段允许 |
| 中文切换 | `feat/i18n-switch` | 需用户同意 |
| 选中文本添加到对话 / 在侧边提问 | `feat/add-to-conversation` | 需用户同意 |

选中文本交互的产品意图：

- 选中终端或编辑区文本后，右键可「添加到对话」，对这一段进行评论，支持多处选区。
- 也可「在侧边聊天中提问」。
- 交互参考 Codex：`添加到对话`、`更多详情`、`在侧边聊天中提问`。
- 该功能必须在独立分支开发，验证没问题后，只有用户明确同意才能合入 `main`。

## AI 工作约定

1. 先判断当前分支。在 `official` 上发现自研改动必须停止。
2. 新功能先建 `feat/<功能名>`，不要直接改 `main`。
3. 未获用户批准，不得把功能分支合并到 `main`，也不得把功能提交到 `official`。
4. 不要向 `upstream` 推送。
5. 不要擅自 `git add` / `git commit` / `git push`；只有用户明确要求提交或推送时才执行。
6. 用户要求提交时，提交说明用中文。
7. 官网同步产生冲突时，列出冲突文件和 `PHT-PATCH` 范围，不要静默丢自研改动。
