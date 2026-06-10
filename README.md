# agent-manager-workflow — AI 总经理工作流

> 一键安装，即刻拥有一个懂调度、会审查、能持久化的 AI 总经理团队。

## 1. 这是什么？

你是**老板**，Claude 是你的**总经理**，总经理下面有一群**员工**：

```
你（老板）
  │
  └── 总经理（任务调度、质量把控、向你汇报）
        │
        ├── 研究员   —— 查资料、做调研
        ├── 写作者   —— 写代码、写文档
        ├── 审查员   —— 挑毛病、查安全
        ├── 部署者   —— 处置交付单（独立调用）
        └── 策展人   —— 维护知识库、发现关联
```

你只需自然描述需求，总经理自动走 6 阶段流水线，每阶段确认后才推进。

- **1 个总经理**：负责任务调度、需求拆解、质量把控
- **5 个员工 Agent**：研究员、写作者、审查员、部署者、策展人
- **8 个命令**：agent-manager-workflow:交办 / 快办 / 审查 / 存档 / 校准 / 近况 / 脑暴 / update

## 2. v2.0 新特性

- **6 阶段固定流程**：需求澄清 → 技术方案 → Spec 撰写 → 代码实现 → 代码审查 → 交付通知，每阶段 ⛔ 确认门禁
- **用户级配置**：`~/.claude/agent-manager/` 存放流程、偏好、决策、校准，换项目跟你走
- **流程可覆盖**：项目级 `.claude/guidance/TASK_PIPELINE.md` 优先于用户级
- **更新命令**：`agent-manager-workflow:update`，插件升级时对比差异，支持全部覆盖/保持/逐项/自定义
- **流程执行状态追踪**：每个任务维护 `流程执行状态.json`，中断后可续接
- **偏好自动积累**：manager 自动识别你的长期偏好并记录

## 3. 安装与维护

### 3.1. 首次安装

在终端运行：

```bash
claude plugin marketplace add https://github.com/miaosong-z/agent-manager-workflow.git
claude plugin install agent-manager-workflow
```

安装后在对话中使用命令或自然语言触发：

```
agent-manager-workflow:交办 "帮我调研 MCP 协议并写一篇笔记"
```

```
帮我重构这个模块，提高性能  ← 自然语言也能触发
```

首次运行会自动初始化 `~/.claude/agent-manager/`，包含默认流程和用户配置。

### 3.2. 从 v1.x 升级

旧版（v1.x）用户无法直接 update，需先迁移到 marketplace 安装方式：

```bash
rm -rf ~/.claude/plugins/agent-manager-workflow
claude plugin marketplace add https://github.com/miaosong-z/agent-manager-workflow.git
claude plugin install agent-manager-workflow@agent-manager-workflow-marketplace
```

### 3.3. 更新

在终端运行：

```bash
claude plugin update agent-manager-workflow@agent-manager-workflow-marketplace
```

更新后重启 Claude Code，下次交办时会检测到版本变化并提醒运行 update。

### 3.4. 卸载

```bash
claude plugin uninstall agent-manager-workflow@agent-manager-workflow-marketplace
claude plugin marketplace remove agent-manager-workflow-marketplace
rm -rf ~/.claude/agent-manager ~/.claude/plugins/agent-manager-workflow
```

## 4. 使用方式

### 4.1. 自然语言触发

不需要记命令。以下自然语言都能触发交办流程：

- "帮我写一个登录接口"
- "帮我重构这个模块"
- "帮我实现用户管理功能"
- "帮我优化这段代码"
- "帮我做一个..."
- "帮我开发..."
- "走工作流处理..."

### 4.2. 命令列表

所有命令格式为 `agent-manager-workflow:<命令名>`：

| 命令 | 用途 | 示例 |
|------|------|------|
| agent-manager-workflow:交办 | 走完整 6 阶段流程 | "帮我写一篇深度笔记" |
| agent-manager-workflow:快办 | 简单任务，不走完整管道 | "改掉这个拼写错误" |
| agent-manager-workflow:审查 | 拉审查员审计指定文件 | src/main.py |
| agent-manager-workflow:存档 | 会话有价值内容持久化 | |
| agent-manager-workflow:校准 | 回顾决策调整授权级别 | |
| agent-manager-workflow:近况 | 展示项目仪表盘 | |
| agent-manager-workflow:脑暴 | 纯发散讨论 | "AI agent 记忆系统怎么设计？" |
| agent-manager-workflow:update | 检查更新、对比差异 | |

## 5. 工作流

```
自然语言触发
  → 创建任务
    → 阶段1：需求澄清   ⛔ 确认
    → 阶段2：技术方案   ⛔ 确认
    → 阶段3：Spec 撰写  ⛔ 确认（缺 OpenSpec → 按需安装）
    → 阶段4：代码实现
    → 阶段5：代码审查   ⛔ 确认（多轮循环）
    → 阶段6：交付通知
```

每阶段产出存入 `.claude/tasks/<task-name>/`，`流程执行状态.json` 实时追踪进度。

## 6. 用户级 vs 项目级

| 层级 | 位置 | 存放内容 |
|------|------|---------|
| 用户级 | `~/.claude/agent-manager/` | 默认流程、决策记录、偏好、校准 |
| 项目级 | `.claude/guidance/` | 架构约定、指导索引、可选流程覆盖 |
| 项目级 | `.claude/tasks/<task-name>/` | 任务阶段文档、流程执行状态 |

流程优先级：项目级 > 用户级 > 插件默认。对话中优化流程时，manager 会询问"当前项目 or 全部项目"。

## 7. 子 Agent 团队

| Agent | 模型 | 职责 |
|-------|------|------|
| researcher | sonnet | 深度调研、查资料、标注来源 |
| writer | sonnet | 撰写代码和文档、产出成品 |
| reviewer | sonnet | 挑毛病、查安全、验证逻辑 |
| deployer | sonnet | 话题终结者，接收交付单、确认后执行（独立调用） |
| curator | haiku | 知识关联、索引维护 |

> 架构师角色使用内置 Plan agent 类型（通过 Agent tool 指定 subagent_type: 'Plan'），不需要独立的 planner agent 文件。

## 8. 决策分级

| 级别 | 范围 | 处理 |
|------|------|------|
| A 级 | 方向、取舍、正确性 | 必须上报用户，记录到 decisions/ |
| B 级 | 常规协调 | 校准后授权 |
| C 级 | 执行细节 | 自行决定 |

## 9. 进阶用法：让 Claude 成为常驻总经理

### 9.1. 默认模式 vs 进阶模式

**默认模式**——你是老板，需要时用命令或自然语言触发：

```
你说：帮我写一篇 Redis 持久化笔记
       → Claude 自动识别为任务，走 6 阶段流程
       → 任务完成，Claude 回到普通模式

你说：今天天气怎么样？
       → 普通 Claude，直接回答，不走管道
```

**进阶模式**——Claude 永久扮演总经理，说话就是交办：

在对话中运行：
```
agent-manager-workflow:交办 "帮我初始化总经理角色，写入 CLAUDE.local.md"
```

效果：
- 说"帮我..."自动走流程，不需要命令
- 每个阶段主动向你确认
- 流程优化时自动询问生效范围

**一句话总结**：默认模式是"按需叫总经理"，进阶模式是"Claude 就是总经理"。

### 9.2. 进阶模式的代价

- Claude 会更有"管理感"——更多汇报、更多确认、更结构化
- 建议在**主力工作项目**中启用，临时项目保持默认模式

## 10. 常见问题

**Q: 命令为什么带命名空间前缀？**
A: 这是 Claude Code 插件系统的标准行为，`agent-manager-workflow:交办` 确保不发生命名冲突。也可以用自然语言直接触发，不需要记命令格式。

**Q: 可以只用部分功能吗？**
A: 可以。每个 Skill 和 Agent 都是独立的。

**Q: 能自定义子 Agent 吗？**
A: 可以。编辑 `${CLAUDE_PLUGIN_ROOT}/agents/*.md` 调整角色提示、模型、工具权限。

**Q: 能修改流程吗？**
A: 可以。对话中说"以后 XX 阶段不需要确认了"，manager 会自动识别并询问生效范围。也可通过 `agent-manager-workflow:update` 在插件升级时自定义流程。

**Q: 换项目后偏好还在吗？**
A: 在。用户级配置（流程、偏好、决策）存于 `~/.claude/agent-manager/`，跟人不跟项目。

## 11. 许可证

MIT
