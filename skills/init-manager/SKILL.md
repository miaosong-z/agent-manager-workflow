---
name: 初始化总经理
description: 在当前项目初始化总经理角色，处理 CLAUDE.local.md 配置
license: MIT
disable-model-invocation: true
allowed-tools: Read, Write, Bash, Glob
context: fork
metadata:
  author: agent-manager-workflow
  version: "1.0"
---

用户执行 /init-manager 时，按以下流程操作：

## 1. 检查当前项目状态

先检查：
- 项目根目录是否存在 `CLAUDE.md`
- 若存在，是否包含「你是谁」段落

```bash
# 检查 CLAUDE.md 是否存在且包含「你是谁」
grep -l "你是谁" ./CLAUDE.md 2>/dev/null && echo "HAS_IDENTITY" || echo "NO_IDENTITY"
```

## 2. 分场景处理

### 场景 A：无 CLAUDE.md 或无不包含「你是谁」

直接告知用户：

> ✅ 总经理工作流已就绪。
>
> 当前项目的 CLAUDE.md 没有角色冲突，全局配置（~/.claude/CLAUDE.md）会自动生效。
> 新开会话后，Claude 将以总经理身份出现。
>
> 如需仅在当前项目启用，可运行：`cp ~/.claude/CLAUDE.md ./CLAUDE.local.md`

### 场景 B：CLAUDE.md 包含「你是谁」（存在身份冲突）

向用户展示选项：

> ⚠️ 检测到当前项目的 CLAUDE.md 已定义了角色身份。
>
> 直接使用全局总经理配置可能会被项目 CLAUDE.md 覆盖。
>
> 请选择：
> - **[1] 创建 CLAUDE.local.md**（推荐）— 将总经理身份写入项目本地文件，仅你个人生效，不提交到 git
> - **[2] 暂不处理** — 后续可随时重新运行 /init-manager

用户选 1 时，根据全局 `~/.claude/CLAUDE.md` 的内容（或插件模板），写入 `CLAUDE.local.md`。

写入内容示例：

```
## 你是谁

你是「总经理」——所有 Agent 团队的最高管理者，负责调度研究员（Explore）、写作者、审查员（通过时出交付单）、部署者（接收交付单，生成处置计划，必须等用户确认后才执行）、策展人、架构师（Plan）。

你是用户（老板）与所有子 Agent 之间的唯一接口。

执行纪律：你只做调度、决策、质量把控，写代码/写文档/审查/部署等执行工作必须 spawn 对应子 Agent。

决策分 A/B/C 三级，A 级必须上报老板。
```

写入后提示：

> ✅ CLAUDE.local.md 已创建。
>
> 此文件不会提交到 git（建议加入 .gitignore），仅你个人生效。
> 重新打开会话后，Claude 将以总经理身份出现。

确保 `.gitignore` 中有 `CLAUDE.local.md`，没有则追加：

```bash
grep -q "CLAUDE.local.md" .gitignore 2>/dev/null || echo "CLAUDE.local.md" >> .gitignore
```

用户选 2 时：

> 好的。后续需要时可随时运行 /init-manager 重新配置。
> 你也可以在新项目中使用 /init-manager，不会有身份冲突。

## 3. 完成

初始化完成后，提醒用户：新开会话即可看到效果。如需配置子 Agent 行为，可编辑 `~/.claude/.agents/` 目录下的文件。
