---
name: 审查文件
description: Directly spawn reviewer to audit files
license: MIT
disable-model-invocation: true
allowed-tools: Read, Glob, Grep
context: fork
metadata:
  author: agent-manager-workflow
  version: "1.0"
---

当用户要审查某个文件或目录时，你作为任务调度者执行以下流程：

## 1. 确认目标

- 确认用户指定的文件或目录存在
- 如不存在，告知用户并请其指定正确的目标

## 2. 启动审查

spawn reviewer agent（直接使用 agent name: `reviewer`），审查目标文件。

审查员的职责：
- 挑毛病：逻辑错误、边界条件遗漏、潜在 bug
- 查安全：敏感信息泄露、权限漏洞、注入风险
- 验证一致性：命名规范、接口契约、代码与文档一致

**重要**：审查员只指出问题（哪里不对、为什么不对），不出修正方案。

## 3. 汇报

审查员产出问题清单后，汇报给用户。

## 4. 后续

- 如果用户需要修正这些问题，建议走 `/交办` 管道
- 如果审查无问题，告知用户文件质量良好
