## 1. 分类与编号规划

- [x] 1.1 确认最终分类列表（investment/growth/tool/insight/misc）和编号格式（`{prefix}-001`）
- [x] 1.2 为现有 8 篇文章手动分配分类和初始序号
- [x] 1.3 为 2-3 篇最新文章添加 frontmatter `category` 和 `number` 字段作为试点

## 2. Hugo 配置

- [x] 2.1 更新 `config.toml`，注册 `category` 为 Hugo taxonomy
- [x] 2.2 验证 `/categories/investment/` 等分类页可正常访问

## 3. 顶栏导航与 About 页面

- [x] 3.1 更新顶栏导航为：首页 | 投资理财 | 成长生活 | 效率工具 | 认知碎片 | 其他 | 关于
- [x] 3.2 创建 `content/about.md` 页面占位（暂时留白）

## 4. 首页定制（Hero + 缩略图网格）

- [x] 4.1 添加 Hero 区域（博客名 + 一句话简介）
- [x] 4.2 实现 3 列缩略图网格布局（PC），2 列（Tablet），单列堆叠（Mobile < 768px）
- [x] 4.3 缩略图：有 `cover` 用 cover，无 `cover` 用分类纯色块（inv 深蓝 / tool 深橙 / insight 深紫 / growth 深绿 / misc 深灰）
- [x] 4.4 首页只展示最新 8 篇，底部加「浏览更多 →」引导到分类页
- [x] 4.5 网格卡中显示分类编号（右下角编号标签）

## 5. 分类页定制（纯列表）

- [x] 5.1 分类页改为纯列表（无缩略图），每条目：标题 + 日期 + 分类编号
- [x] 5.2 日期倒序，每页 15 篇，加翻页控件
- [x] 5.3 验证各分类页（investment/growth/tool/insight/misc）显示正常

## 6. 编号展示

- [x] 6.1 文章详情页：在标题附近显示编号
- [x] 6.2 列表页（分类/标签页）：每条目右侧显示编号
- [x] 6.3 CSS 样式：编号小字号、灰色，不喧宾夺主

## 7. 存量文章补录

- [x] 7.1 为全部历史文章添加 `category` 和 `number` 字段
- [x] 7.2 运行 `hugo` 验证构建无报错
- [x] 7.3 本地 `hugo server` 预览，确认所有页面显示正常

## 8. 收尾

- [ ] 8.1 提交代码，触发 CI/CD 部署到 GitHub Pages
- [ ] 8.2 访问线上页面最终验证
- [ ] 8.3 记录本次变更到归档文件