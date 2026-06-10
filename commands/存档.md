---
name: 存档会话
description: Scan session outputs and persist valuable content to memory and guidance
license: MIT
disable-model-invocation: false
allowed-tools: Read, Write, Edit, Glob, Grep
metadata:
  author: agent-manager-workflow
  version: "1.0"
---

将会话中有价值的内容持久化到项目记忆中。

## 1. 回顾本次会话的产出

扫描会话历史，识别以下类型的有价值内容：

- **新工具/脚本/系统搭建** → 记录到 `.guidance/` 并在 `INDEX.md` 中注册为落地工具
- **重要决策及原因** → 记录到 `.guidance/` 决策记录（为什么选 A 不选 B，上下文是什么）
- **用户偏好/项目新认知** → 写入 memory（`.claude/memory/` 或用户配置的 memory 目录）
- **新发现的外部资源/参考** → 写入 reference 类型 memory

## 2. 更新索引

更新 `.guidance/INDEX.md`：
- 「落地工具」表格中添加新记录
- 「决策记录」表格中添加新决策
- 「最近更新」列表更新日期

如果 `.guidance/INDEX.md` 不存在，使用 `${CLAUDE_PLUGIN_ROOT}/templates/guidance-index-template.md` 模板创建。

## 3. 汇报

向用户汇报：存档了哪些内容，分别存在哪里。

## 4. 规则

- 不要存中间讨论、草稿、废弃方案
- 不要存代码模式、架构细节（这些在代码里）
- 不要重复存已有的内容（先读 INDEX 确认不重复）
- 每条记忆应简洁、独立、可检索
