---
name: 交办任务
description: 将用户请求按6阶段工作流执行（需求澄清→技术方案→Spec→实现→审查→交付）。当用户说"帮我做/实现/开发/重构/优化/写..."或提到"工作流/走流程/交办"时触发。
license: MIT
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch
context: fork
metadata:
  author: agent-manager-workflow
  version: "2.1.2"
---

当用户交办一个任务时，按以下步骤执行。

## 0. 初始化检查（首次运行）

检查 `~/.claude/agent-manager/` 目录是否存在。不存在则：

1. 从 `${CLAUDE_PLUGIN_ROOT}/templates/user-config/` 复制所有文件到 `~/.claude/agent-manager/`
2. 将 `${CLAUDE_PLUGIN_ROOT}/guidance/TASK_PIPELINE.md` 复制为 `~/.claude/agent-manager/TASK_PIPELINE.md`
3. 告知用户：用户级配置已初始化在 `~/.claude/agent-manager/`，包含默认流程、偏好、校准、决策记录。可通过 `/update` 命令管理升级。可通过对话优化流程，优化时会询问"当前项目生效 or 全部项目生效"。

存在则：
1. 读取 `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json` 获取插件版本号
2. 读取 `~/.claude/agent-manager/config.json` 中的 `version` 字段
3. 对比两个版本号：
   - 一致 → 继续
   - 不一致 → 提醒用户：「插件已更新到 v{new}，你的配置基于 v{old}。运行 `/update` 查看更新内容。」

## 1. 读取流程定义

按优先级加载 TASK_PIPELINE.md：
1. `.claude/guidance/TASK_PIPELINE.md`（项目级覆盖）
2. `~/.claude/agent-manager/TASK_PIPELINE.md`（用户级）
3. `${CLAUDE_PLUGIN_ROOT}/guidance/TASK_PIPELINE.md`（插件默认）

读取 `~/.claude/agent-manager/preferences.md` 了解用户偏好。

## 2. 任务初始化

1. 从用户需求中提取简短中文 task-name
2. 创建任务目录 `.claude/tasks/<task-name>/`
3. 创建 `流程执行状态.json`，初始化 6 阶段全部为 pending：

```json
{
  "task_name": "<task-name>",
  "status": "in_progress",
  "current_stage": 1,
  "stages": {
    "1": {"name": "需求澄清", "status": "in_progress", "started_at": "<ISO时间>"},
    "2": {"name": "技术方案", "status": "pending"},
    "3": {"name": "Spec撰写", "status": "pending"},
    "4": {"name": "代码实现", "status": "pending"},
    "5": {"name": "代码审查", "status": "pending"},
    "6": {"name": "交付通知", "status": "pending"}
  },
  "created_at": "<ISO时间>",
  "updated_at": "<ISO时间>"
}
```

4. 展示任务初始化信息：`📋 任务「{task-name}」已创建，开始 6 阶段流程。`

## 3. 执行阶段流水线

所有任务统一走 6 阶段流程，不判断复杂度。

**状态更新规则**：进入每个阶段时，将当前阶段的 status 改为 `in_progress`，记录 `started_at`，更新 `current_stage` 和 `updated_at`。阶段通过 ⛔ 确认后，改为 `completed`，记录 `completed_at`。

每个 ⛔ 标记的确认点，必须等待用户明确说"通过"/"确认"/"没问题"/"继续"等，才能进入下一阶段。用户说"等等"/"不对"/"重新来"，则根据反馈调整后再次确认。

### 阶段 1：需求澄清

- 目的：对齐需求理解，澄清模糊点
- 产出：`.claude/tasks/<task-name>/需求确认单.md`，包含功能点、边界条件、与现有系统的关系
- ⛔ 展示需求确认单，等待用户确认
- 确认后：更新 `流程执行状态.json` 阶段 1 为 `completed`，阶段 2 为 `in_progress`，`current_stage` 改为 2

### 阶段 2：技术方案

- 目的：设计实现方案，拆解任务
- 使用 Plan agent（`subagent_type: 'Plan'`）设计方案
- 产出：`.claude/tasks/<task-name>/技术方案.md`（数据库变更、接口设计、文件清单、编码顺序）和 `.claude/tasks/<task-name>/接口协议.md`（如涉及接口）
- ⛔ 展示技术方案，等待用户确认
- 确认后：更新 `流程执行状态.json` 阶段 2 为 `completed`，阶段 3 为 `in_progress`，`current_stage` 改为 3

### 阶段 3：Spec 撰写

- 目的：将技术方案转化为 OpenSpec 变更提案，作为代码实现的规格约束
- **先检查 OpenSpec 是否可用**：执行 `which openspec` 或等效检测
  - 可用 → 直接进入 Spec 撰写
  - 不可用 → 告知用户阶段 3 需要 OpenSpec 工具，询问是否自动安装。用户确认后执行安装，验证成功后继续。用户拒绝则等待用户指示。
- 产出：`openspec/` 下的变更提案（提案 + 设计 + 规格 + 任务清单）
- ⛔ 展示 Spec 概要，等待用户确认
- 确认后：更新 `流程执行状态.json` 阶段 3 为 `completed`，阶段 4 为 `in_progress`，`current_stage` 改为 4

### 阶段 4：代码实现

- 目的：按 Spec 任务清单逐条编码
- spawn writer agent 实施
- 产出：代码变更（commit）
- 完成后：更新 `流程执行状态.json` 阶段 4 为 `completed`，阶段 5 为 `in_progress`，`current_stage` 改为 5，自动进入阶段 5

### 阶段 5：代码审查

- 目的：发现功能、编译、安全、风格问题
- spawn reviewer agent 审查
- 审查 → 发现问题 → writer 修正 → 再审，多轮循环直到通过
- ⛔ 展示审查结论，等待用户确认
- 确认后：更新 `流程执行状态.json` 阶段 5 为 `completed`，阶段 6 为 `in_progress`，`current_stage` 改为 6

### 阶段 6：交付通知

- 目的：汇总变更内容和审查结论
- 产出：`.claude/tasks/<task-name>/交付通知.md`
- 展示变更摘要 + 审查结论
- 完成后：更新 `流程执行状态.json` 阶段 6 为 `completed`，任务 `status` 改为 `completed`，`current_stage` 清空

## 4. 决策记录

遇到 A 级决策（方向、取舍、正确性）时：
1. 暂停，向用户提供：问题描述 + 可选方案 + 你的建议
2. 用户决策后，记录到 `~/.claude/agent-manager/decisions/`，文件名格式 `YYYY-MM-DD-决策主题.md`

## 5. 偏好自动积累

识别用户在对话中表达的长期偏好，自动追加到 `~/.claude/agent-manager/preferences.md`。触发条件：
- 用户说"以后都..." "每次..." "默认..." "优先..." 等长期性表述
- 用户反复做出同一选择（3 次以上）
- 追加前简要确认："我会记住：以后新项目优先用 Java。可以吗？"

## 6. 流程优化

用户对话中提出流程变更（如"以后方案阶段不需要我确认了"、"加一个部署阶段"）时：
1. 识别这是流程优化请求
2. 询问：「这个调整只对当前项目生效，还是对所有项目？」
   - 当前项目 → 写入 `.claude/guidance/TASK_PIPELINE.md`
   - 所有项目 → 写入 `~/.claude/agent-manager/TASK_PIPELINE.md`
3. 写入后告知用户变更内容及生效范围

## 7. 任务文档管理

每个任务在 `.claude/tasks/<task-name>/` 下创建独立目录，所有阶段文档存放于此。task-name 从用户需求中提取简短中文名。

## 8. 完成后

- 简要汇报做了什么
- 提醒用户：如需持久化本次产出到记忆，使用 `/存档`；如需检查更新，使用 `/update`

---

**注意**：本 Skill 是自包含的任务入口，独立可用。安装后直接使用 `/交办` 即可，不依赖 CLAUDE.md。
