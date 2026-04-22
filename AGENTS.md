# AGENTS.md

## ⚠ 强制约束：所有功能变更必须遵循 OpenSpec 工作流

**在做任何功能新增或修改之前，必须完成以下四步，不得跳过：**

1. **Proposal** — 在 `openspec/changes/<YYYY-MM-DD>-<slug>/proposal.md` 起草提案，经用户确认后才能继续
2. **Design** — 在 `design.md` 说明技术方案
3. **Specs** — 在 `specs/<capability>/spec.md` 为每个能力写可验证规格
4. **Tasks** — 在 `tasks.md` 列出可执行任务列表

**未经 proposal 确认，不得写任何实现代码。** 完整规则见 `openspec/config.yaml`。

---

## 技能

以下技能按需触发。当用户请求匹配触发条件时，读取对应文件并执行：

| 触发条件 | 技能文件 |
|---|---|
| 用户要提出新功能、新变更，或描述想要构建的东西 | [openspec/skills/propose.md](openspec/skills/propose.md) |
| 用户要开始实施、写代码、或继续执行某个变更的任务 | [openspec/skills/apply.md](openspec/skills/apply.md) |
| 用户要归档、完结某个已完成的变更 | [openspec/skills/archive.md](openspec/skills/archive.md) |
| 用户想探索想法、分析问题、或在开始提案前厘清需求 | [openspec/skills/explore.md](openspec/skills/explore.md) |

---

## 命令

```bash
# 本地开发（http://localhost:1313/，含草稿）
hugo server -D

# 新建文章
hugo new content posts/<filename>.md

# 生产构建（输出到 public/）
hugo --minify
```

部署自动完成：推送到 `master` 触发 GitHub Actions，使用 Hugo 0.160.1 构建并部署到 GitHub Pages。

PaperMod 主题以 git submodule 管理，克隆时使用 `git clone --recurse-submodules`，或克隆后运行 `git submodule update --init`。

## 架构

Hugo 静态站点，使用 PaperMod 主题，无自定义 layout 或模板，所有渲染由主题处理。

**内容**位于 `content/posts/`，Markdown 文件。新文章默认 frontmatter 模板见 `archetypes/default.md`：

```yaml
---
title: "文章标题"
date: 2026-04-22T10:00:00+08:00
author: "逃跑"
draft: false
tags: ["标签1", "标签2"]
---
```

**站点配置**全部在 `config.toml`。

## 写作规范

写作或编辑任何文章内容时，以下规则自动生效，无需用户提示：

**格式约束**
- 正文不写 `# H1` 标题——PaperMod 自动将 frontmatter `title` 渲染为页面标题，重复写会导致双标题
- 章节使用中文数字标题：`## 一、`、`## 二、`、`## 三、`
- 比较和结构化信息用表格
- 关键定义或重点用引用块（`>`）
- 文章长度：教程约 100 行，方法论约 140 行，深度分析 220 行以上

**语言风格约束**（详细规则见 [agents/skills/humanize.md](agents/skills/humanize.md)）
- 不使用"综上所述"、"值得注意的是"、"不难发现"等机械过渡词
- 不使用"首先……其次……最后……"排比式展开
- 不把所有内容拆成条目列表，连贯观点用自然段落
- 用"我"而非"我们"，加入作者立场和判断
- 不在结尾重复总结全文
