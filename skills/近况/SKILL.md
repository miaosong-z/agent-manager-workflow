---
name: 查看近况
description: Show project status dashboard and ongoing tasks
license: MIT
disable-model-invocation: false
allowed-tools: Read, Glob, Grep
metadata:
  author: agent-manager-workflow
  version: "1.0"
---

展示当前项目的全局近况仪表盘。

## 1. 读取关键文件

- 读取 `.guidance/INDEX.md` — 了解现有工具和决策记录
- 读取 MEMORY.md（如存在）— 了解项目记忆
- 检查项目各子目录状态（是否存在、最近修改时间）

## 2. 兜底处理

如果 `.guidance/INDEX.md` 不存在（新项目或尚未初始化），告知用户：
> 这是新项目，尚未建立指导文档索引。建议使用 `/交办` 完成第一个任务，然后用 `/存档` 建立知识沉淀体系。

## 3. 汇报内容

以简洁格式汇报：

- **活跃项目/目录**：名称 + 最后活动 + 简要状态
- **可用工具**：列出 INDEX 中的落地工具（名称 + 位置 + 用途）
- **待确认决策**：列出需要用户关注的 A/B 级决策（如有）
- **校准状态**：已完成的任务数，距下次校准还差几个任务（每 3 个任务提醒校准）

## 4. 简洁原则

如果一切正常无事，一句话带过，不要啰嗦。只汇报有信息量的内容。
