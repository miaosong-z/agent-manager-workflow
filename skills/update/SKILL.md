---
name: 更新检查
description: Check for plugin updates and merge workflow changes
license: MIT
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
context: fork
metadata:
  author: agent-manager-workflow
  version: "2.1.2"
---

当用户执行 `/update` 时，按以下步骤检查插件更新。

## 1. 版本对比

1. 读取 `~/.claude/agent-manager/config.json` 中的 `version` 字段
2. 读取 `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json` 中的 `version` 字段
3. 如一致，告知用户「当前已是最新版本」，结束
4. 如不一致，进入步骤 2

## 2. 计算差异

对以下用户级配置文件，逐文件对比「插件默认版本」与「用户当前版本」：
- `TASK_PIPELINE.md`
- （未来可扩展更多文件）

对比方式：
1. 读取 `${CLAUDE_PLUGIN_ROOT}/guidance/TASK_PIPELINE.md`（新版本）
2. 读取 `~/.claude/agent-manager/TASK_PIPELINE.md`（用户版本）
3. 分析差异，用自然语言总结变更点（如「新增阶段 X」「调整了阶段 Y 的门禁位置」「删除了阶段 Z」）

跳过 `config.json` 中 `skipped_updates` 已记录的版本对应文件。

## 3. 展示 + 询问

```
🔄 工作流更新检查

插件版本：v{new}
当前生效：v{old}

发现以下变更：

📋 TASK_PIPELINE.md
   - 新增：阶段 X ...
   - 调整：...
   - 删除：...

如何处理？
  1. 全部覆盖 — 用新版本替换当前配置
  2. 保持当前 — 不更新，不再提醒这些变更
  3. 逐项确认 — 逐个文件决定覆盖/跳过
  4. 自定义流程 — 以新版本为基础，手动调整流程
```

## 4. 根据用户选择执行

### 全部覆盖
- 用插件新版本文件覆盖 `~/.claude/agent-manager/` 对应文件
- 更新 `config.json` 中 `version` 为新版本号

### 保持当前
- 将所有本次变更的文件名写入 `config.json` 的 `skipped_updates["{新版本号}"]` 数组
- 更新 `config.json` 中 `version` 为新版本号（标记"已知此版本"）

### 逐项确认
- 对每个变更文件单独询问「覆盖/跳过」
- 覆盖 → 覆盖文件
- 跳过 → 写入 `skipped_updates`
- 完成后更新 `version`

### 自定义流程
- 展示新版本的阶段列表
- 用户用自然语言描述调整（增删改阶段、调整门禁位置等）
- 根据用户描述重写 `TASK_PIPELINE.md`
- 展示新流程并请用户确认
- 确认后写入，更新 `version`

## 5. 完成

告知用户更新结果。
