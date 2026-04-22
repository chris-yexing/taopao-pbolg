---
title: "通过openspec构建个人博客网站"
date: 2026-04-09T15:20:00+08:00
author: "逃跑"
draft: false
tags: ["AI", "科技"]
---

记录一下用 OpenSpec 规划、Hugo 搭建博客的实际步骤，没什么废话，直接上。

## 1. 创建 OpenSpec change

在项目根目录运行：

```bash
openspec new change "personal-article-blog"
```

这会生成变更目录和基础文件。

## 2. 撰写 proposal

编辑 `openspec/changes/personal-article-blog/proposal.md`，说明：

- 博客要实现的目标
- 功能范围
- 需要的能力
- 影响范围

## 3. 撰写 design

编辑 `openspec/changes/personal-article-blog/design.md`，说明实现方案：

- 使用 Hugo 生成静态站点
- 选择主题和风格
- 使用 GitHub Pages 部署
- 明确不做动态评论、不做数据库

## 4. 撰写 specs

为每个能力编写规格文件：

- `specs/article-display/spec.md`
- `specs/blog-search/spec.md`
- `specs/rss-feed/spec.md`
- `specs/social-sharing/spec.md`

每个规格写清楚需求和场景。

## 5. 撰写 tasks

编辑 `openspec/changes/personal-article-blog/tasks.md`，列出实施步骤：

- 安装 Hugo
- 创建站点结构
- 添加主题
- 配置部署
- 实现首页、文章列表、标签、搜索、RSS、分享
- 调整响应式样式

## 6. 搭建 Hugo 项目

在项目目录执行：

```bash
hugo new site .
```

然后将主题放入 `themes/`，并在 `config.toml` 中设置主题。

## 7. 写文章

把内容写到 `content/posts/` 目录下面的 `.md` 文件。

示例：

```md
---
title: "通过openspec构建一个个人博客网站"
date: 2026-04-09T15:20:00+08:00
draft: false
tags: ["AI", "科技"]
---

这里写文章正文。
```

## 8. 本地预览

运行：

```bash
hugo server -D
```

然后访问 `http://localhost:1313/`。

## 9. 发布到外网

将代码推送到 GitHub 仓库，使用 GitHub Pages 或其他静态站点服务。

## 10. 标签自定义

标签在 front matter 里写：

```md
tags: ["AI", "科技"]
```

要增加标签，直接修改该字段即可。

*本文是本站第一篇文章，记录从 OpenSpec 规划到 Hugo 上线的完整步骤。*