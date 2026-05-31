---
name: 快速办理
description: Execute simple tasks directly without full pipeline
license: MIT
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
context: fork
metadata:
  author: agent-manager-workflow
  version: "1.0"
---

当用户要求快速处理一个任务时，你作为任务调度者执行以下流程：

## 1. 先查索引（防失忆）

在执行之前，用 `Read` 检查 `.guidance/INDEX.md` 是否存在：
- 如果文件存在，确认是否有现成的工具/脚本可复用，或已有相关决策记录可参考；有则直接使用，不需要从头开始
- 如果文件不存在，跳过此检查，直接进入下一步

## 2. 判断是否真的简单

- 如果任务确实简单（单步操作、改拼写、查简单资料、一句话问答），直接 spawn writer agent 完成
- 如果发现任务实际上不简单（需要调研、需要方案设计、涉及多个步骤），告知用户并建议走 `/交办` 完整管道

## 3. 产出

- 跳过 researcher、reviewer agent，快速产出
- 产出后直接汇报给用户

## 4. 完成后

- 简要汇报做了什么
- 提醒用户：如需质量审查，可用 `/审查` 命令
