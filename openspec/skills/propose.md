# openspec-propose

当用户想要提出新功能、新变更，或者描述想要构建的东西时，按以下步骤执行。

## 目标

在 `openspec/changes/` 下创建一个新变更目录，并依次生成 proposal.md、design.md、specs/、tasks.md 四个制品，完成后变更即可进入实施阶段。

## 执行步骤

### 1. 确认变更名称

如果用户已提供名称（kebab-case），直接使用。否则询问用户想要构建什么，根据描述推导出 kebab-case 名称（例如"添加搜索功能" → `add-search`）。

**未明确需求前不得继续。**

### 2. 创建变更目录

在 `openspec/changes/<name>/` 下创建以下结构：

```
openspec/changes/<name>/
└── .openspec.yaml
```

`.openspec.yaml` 内容：

```yaml
schema: spec-driven
created: <YYYY-MM-DD>
```

### 3. 按顺序创建制品

依照 `openspec/config.yaml` 中定义的规则和上下文，按以下顺序创建制品：

**a. proposal.md**

参考 `openspec/config.yaml` 中的规则，结合用户需求，创建 `openspec/changes/<name>/proposal.md`。

必须包含：
- `## Why` — 为什么要做这个变更
- `## What Changes` — 具体改变什么
- `## Capabilities` — 新增/修改了哪些能力
- `## Impact` — 对现有系统的影响

**向用户展示 proposal.md，等待明确确认后再继续。**

**b. design.md**

读取 proposal.md 作为上下文，创建 `openspec/changes/<name>/design.md`。

必须包含：
- 技术方案选择和理由
- 关键设计决策
- 实现路径概述

**c. specs/**

为 proposal.md 中每个 Capability 在 `openspec/changes/<name>/specs/<capability-name>/spec.md` 创建规格文件。

每个 spec.md 必须包含：
- 能力描述
- 可验证的行为条件（Given/When/Then 或等价形式）

**d. tasks.md**

读取 design.md 和 specs/ 作为上下文，创建 `openspec/changes/<name>/tasks.md`。

格式为可勾选的任务列表：
```markdown
- [ ] 任务描述（对应 spec: <capability-name>）
```

### 4. 完成后输出摘要

```
变更已就绪：<name>
位置：openspec/changes/<name>/

已创建：
- proposal.md — <一句话描述>
- design.md — <一句话描述>
- specs/<n> 个能力规格
- tasks.md — <N> 个任务

可以开始实施了。
```

## 注意事项

- 每个制品依赖前一个，必须按顺序创建
- proposal.md 必须经过用户确认才能继续
- `openspec/config.yaml` 中的 `context` 和 `rules` 是给你的约束，不要复制进制品文件
- 如果同名变更已存在，询问用户是继续还是重新创建
