## 1. 准备依赖

- [x] 1.1 日出日落算法直接内联于 extend_head.html（无需外部 SunCalc.js 文件）

## 2. Banner 样式

- [x] 2.1 创建 `static/css/sunrise-theme.css`，实现底部 slim banner 样式（flex 布局、固定底部、淡出动效）

## 3. 自动主题逻辑（extend_head.html）

- [x] 3.1 创建 `layouts/partials/extend_head.html`
- [x] 3.2 添加 `<link>` 引入 `sunrise-theme.css`
- [x] 3.3 内联日出日落算法（等价于 SunCalc.getTimes，无需外部文件依赖）
- [x] 3.4 实现内联脚本：检查 `theme-manual` → L1（缓存坐标）→ L2（时区映射）→ L3（系统偏好）
- [x] 3.5 内嵌 IANA 时区 → 近似经纬度映射表（约 200 个主要时区）

## 4. Consent Banner + 手动覆盖（extend_footer.html）

- [x] 4.1 创建 `layouts/partials/extend_footer.html`
- [x] 4.2 实现 Consent Banner HTML 结构（底部固定，含说明文字和两个按钮）
- [x] 4.3 实现 Banner 展示逻辑（检查 sessionStorage 条件）
- [x] 4.4 实现「允许」按钮：关闭 Banner → 触发 geolocation → 成功时缓存坐标并更新主题
- [x] 4.5 实现「不了」按钮和 10 秒超时自动关闭
- [x] 4.6 实现手动覆盖监听器：监听 `#theme-toggle` 点击，写入 `sessionStorage["theme-manual"]`

## 5. 验证

- [x] 5.1 `hugo --minify` 构建成功（0 错误），输出包含完整算法、TZ 映射表、Banner HTML
- [x] 5.2 构建输出验证：CSS 链接、算法常量、Banner 元素、手动覆盖监听器均正确注入
- [ ] 5.3 浏览器手动测试：Consent Banner 仅 session 首次出现，关闭后不再出现
- [ ] 5.4 浏览器手动测试：手动切换后跨页导航不被自动逻辑覆盖
- [ ] 5.5 浏览器手动测试：「允许」后 geolocation 成功时主题更新、坐标缓存
