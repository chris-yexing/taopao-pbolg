# 逃跑的笔记

一个基于 Hugo + PaperMod 主题构建的个人博客网站。

🔗 **在线访问**: https://chris-yexing.github.io/taopao-pbolg/

## 项目简介

本项目是一个静态个人博客，用于分享技术文章和个人想法。采用 **OpenSpec 规范驱动开发** 方法论，结合 **Qoder AI 编程助手** 完成构建。

## 技术栈

| 技术 | 用途 |
|------|------|
| [Hugo](https://gohugo.io/) | 静态站点生成器 |
| [PaperMod](https://github.com/adityatelange/hugo-PaperMod) | Hugo 主题 |
| [OpenSpec](https://openspec.dev/) | 规范驱动开发方法论 |
| [Qoder](https://qoder.dev/) | AI 编程助手 |
| GitHub Pages | 免费托管部署 |

## 功能特性

- 📝 文章发布（Markdown 格式）
- 🏷️ 标签分类
- 🔍 站内搜索（Fuse.js）
- 📡 RSS 订阅
- 🔗 社交分享
- 📱 响应式设计
- 🌓 暗/亮模式切换
- 📑 文章目录导航

## 项目结构

```
.
├── config.toml          # Hugo 站点配置
├── content/posts/       # 博客文章目录
├── themes/PaperMod/     # Hugo 主题
├── openspec/            # OpenSpec 规范文档
│   └── changes/         # 变更记录
│       └── archive/2026-04-09-personal-article-blog/
│           ├── proposal.md    # 项目提案
│           ├── design.md      # 设计方案
│           ├── tasks.md       # 实施任务
│           └── specs/         # 功能规格
└── README.md            # 本文件
```

## 本地开发

### 前置要求

- [Hugo](https://gohugo.io/installation/) (Extended 版本)
- Git

### 启动开发服务器

```bash
hugo server -D
```

访问 http://localhost:1313/ 预览站点。

### 新建文章

```bash
hugo new content posts/文章名称.md
```

### 构建站点

```bash
hugo --minify
```

构建结果输出到 `public/` 目录。

## 写作规范

文章使用 Markdown 格式，头部包含 YAML frontmatter：

```markdown
---
title: "文章标题"
date: 2026-04-10T12:00:00+08:00
author: "逃跑"
draft: false
tags: ["标签1", "标签2"]
---

文章内容...
```

## 部署

本项目使用 GitHub Pages 自动部署：

1. 推送代码到 GitHub 仓库
2. GitHub Actions 自动构建并部署
3. 访问 `https://<username>.github.io/taopao-pbolg/`

## 开发方法论

本项目实践 **OpenSpec 规范驱动开发**：

1. **Proposal** - 明确"为什么"和"做什么"
2. **Design** - 确定"怎么做"，记录决策
3. **Specs** - 详细定义功能规格
4. **Tasks** - 拆解可执行的任务
5. **Implementation** - 编码实现

配合 Qoder AI 助手，实现高效的人机协作开发。

## 相关文章

- [通过 OpenSpec 构建个人博客网站](https://chris-yexing.github.io/taopao-pbolg/posts/my-first-post/)
- [OpenSpec 与 Qoder：AI 驱动的规范开发实践](https://chris-yexing.github.io/taopao-pbolg/posts/openspec-qoder-intro/)

## 许可证

本项目采用 [MIT License](LICENSE) 开源协议。

主题 [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 由 Aditya Telange 开发，遵循 MIT 协议。

---

© 2026 逃跑的笔记. All rights reserved.
