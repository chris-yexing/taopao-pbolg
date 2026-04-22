# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Local development (serves at http://localhost:1313/, includes drafts)
hugo server -D

# Create a new post
hugo new content posts/<filename>.md

# Production build (outputs to public/)
hugo --minify
```

Deployment is automatic: pushing to `master` triggers GitHub Actions, which builds with Hugo 0.160.1 and deploys to GitHub Pages at https://chris-yexing.github.io/taopao-pbolg/

The PaperMod theme is managed as a git submodule — always use `git clone --recurse-submodules` or run `git submodule update --init` after cloning.

## Architecture

This is a Hugo static site using the PaperMod theme with no custom layouts or templates — all rendering is handled by the theme.

**Content** lives in `content/posts/` as Markdown files. The archetype at `archetypes/default.md` defines the default frontmatter for new posts:

```yaml
---
title: "文章标题"
date: 2026-04-22T10:00:00+08:00
author: "逃跑"
draft: false
tags: ["标签1", "标签2"]
---
```

**Site config** is entirely in `config.toml`. The base URL, theme, nav menu, and author are all set there.

**OpenSpec** (`openspec/`) documents the spec-driven design process used when planning this blog. It follows a proposal → design → specs → tasks workflow. It is documentation only and has no effect on the build.

## Writing Style Conventions

Articles follow these conventions (match existing posts when adding new ones):

- Do not write a `# H1` heading in the body — PaperMod renders the frontmatter `title` as the page heading automatically; a duplicate `#` causes a double title on the published site
- Sections use Chinese numeral headings: `## 一、`, `## 二、`, `## 三、`
- Use tables for comparisons and structured information
- Use blockquotes (`>`) for key definitions or callouts
- End with an italicized meta line: `*本文是本站第N篇文章，记录……*`
- Article length: tutorial ~100 lines, methodology ~140 lines, deep-dive ~220+ lines
