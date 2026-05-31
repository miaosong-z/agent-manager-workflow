---
name: 交办任务
description: Accept and execute tasks through the full agent pipeline
license: MIT
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch
context: fork
metadata:
  author: agent-manager-workflow
  version: "1.0"
---

当用户交办一个任务时，你作为任务调度者执行以下流程：

## 1. 判断复杂度

- **简单任务**（单步操作、改拼写、查简单资料、一句话问答）→ 自动降级为「快办」模式，只 spawn writer agent 产出
- **中等任务**（写笔记、写文档、做调研）→ spawn researcher agent 调研 → 等结果 → spawn writer agent 撰写 → spawn reviewer agent 审查 → 汇总汇报
- **复杂任务**（搭建项目、重构、需要方案设计）→ spawn researcher agent 调研 → 使用 Plan agent（架构师）设计方案，通过 Agent tool 指定 subagent_type: 'Plan' → **向用户确认方案** → spawn writer agent 实施 → spawn reviewer agent 审查 → 汇总汇报

## 2. 执行规则

- 使用 Agent tool spawn 子 Agent，子 Agent 分别配置在 `${CLAUDE_PLUGIN_ROOT}/agents/` 目录下
- researcher 用 Explore agent 类型
- planner 用 Plan agent 类型
- writer、reviewer、curator 用 general-purpose agent 类型 + 各自角色提示
- 审查不通过时，将审查意见传给 writer agent 修正，循环直到通过
- 最终汇总结果，简洁汇报给用户

## 3. 决策上报

- 遇到方向性分歧、取舍、正确性问题（A 级决策）必须暂停并询问用户
- 常规协调、冲突（B 级）自行决定，校准期则上报
- 执行细节（C 级）直接决定
- 拿不准 A/B 时，宁可上报。上报时提供：问题描述 + 可选方案 + 你的建议

## 4. 完成后

- 简要汇报做了什么、发现了什么问题（如有）
- 提醒用户：如果需要持久化本次产出，建议使用 `/存档` 命令

---

**注意**：本 Skill 是自包含的任务入口，不依赖用户是否配置了 CLAUDE.md，独立可用。插件安装后直接使用 `/交办` 即可。
