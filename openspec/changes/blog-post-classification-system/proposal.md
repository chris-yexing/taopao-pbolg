## Why

目前博客的文章仅依赖 Hugo 默认的分类（categories）和标签（tags）体系，缺乏系统性的编号机制。随着文章数量增长，读者难以快速定位系列文章，也难以判断文章之间的关联和阅读顺序。通过为每篇文章建立两段式编号体系，可以增强内容的组织性、可发现性和系列感。

## What Changes

- **引入分类体系**：将文章归入预定义的几个大类（如「投资方法论」「工具效率」「读书笔记」等）
- **两段式编号规则**：每篇文章获得一个编号 `分类前缀-序号`，例如 `inv-001`、`tool-012`，编号随文章在分类中的位置动态或手动确定
- **修订文章 frontmatter**：为现有文章和新文章添加 `category` 和 `number` 字段
- **可选：Hugo Taxonomy 配置**：将 `category` 注册为 Hugo 分类法（taxonomy），与默认的 categories 并存或替代

## Capabilities

### New Capabilities

- **post-category**: 定义博客文章的分类体系，包括分类名称、中文标签、分类前缀（用于编号）等属性
- **post-numbering**: 定义两段式编号的具体规则，包括编号格式、序号分配方式（手动还是自动递增）、编号展示位置等
- **page-layout**: 重新定义首页与分类页的职责边界和视觉形态——首页为 Hero + 缩略图网格，分类页为纯列表
- **category-list**: 定义博客的预定义分类列表（5 个分类），作为 post-category 的具体枚举

### Modified Capabilities

- 无 - 这是全新的能力，不修改已有能力

## Impact

- **content/posts/*.md**：需为每篇文章 frontmatter 添加 `category` 和 `number` 字段
- **hugo.toml / hugo.yaml**：如启用自定义 taxonomy，需新增配置
- **主题模板**（extend_head.html / extend_footer.html）：编号在文章列表和详情页的展示方式（如有定制需求）
- **CI/CD**：无影响，纯静态内容变更