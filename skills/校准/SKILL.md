---
name: 校准决策权限
description: Review recent decisions and calibrate authorization levels
license: MIT
disable-model-invocation: false
allowed-tools: Read, Write, Edit
metadata:
  author: agent-manager-workflow
  version: "1.0"
---

执行决策权限校准，帮助用户调整哪些事可以自主决定，哪些需要上报。

## 1. 回顾 B 级决策

回顾本次会话中，你在任务调度过程中做过的 B 级决策（常规协调类决策），例如：
- "我判断这个任务是中等复杂度，按中等流程处理"
- "审查员的意见我觉得合理，直接让写作者改了"

## 2. 列出决策清单

逐条列出：决策内容 + 当时的处理方式

格式：
```
决策 N：<决策简述>
当时处理：<你怎么处理的>
```

## 3. 逐条询问用户

对每条决策，询问用户态度：

- **"这个决策以后你自己定"** → 标记为 C 级，以后不再上报
- **"继续上报"** → 保持 B 级
- **"这个应该问我"** → 升级为 A 级，永远上报

## 4. 记录授权变更

将用户的授权决定写入 memory（feedback 类型），格式如：
```
用户授权：<决策类型> 从此由调度者自行决定，不再上报。
```

## 5. 汇报

汇报校准结果：N 项决策中，X 项授权下放，Y 项保持，Z 项升级。

---

如果没有做过 B 级决策，告知用户"本次会话尚无需要校准的决策"。
