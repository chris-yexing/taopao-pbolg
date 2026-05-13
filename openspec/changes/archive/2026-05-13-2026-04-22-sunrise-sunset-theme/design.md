## Context

博客使用 Hugo + PaperMod 主题，已有手动亮/暗模式切换。PaperMod 的主题初始化逻辑在 `head.html` 中同步执行（读取 `localStorage["pref-theme"]` 或系统 `prefers-color-scheme`），之后调用 `extend_head.html`。手动切换逻辑（写 `localStorage`）在 `footer.html` 中，`extend_footer.html` 在其之前调用。两个扩展点目前均为空占位符，在项目 `layouts/partials/` 下创建同名文件即可覆盖，**无需修改主题文件**。

**执行顺序（每次页面加载）：**
1. PaperMod `head.html` 初始化主题（`localStorage` → 系统偏好）
2. 我们的 `extend_head.html` 覆盖为日出日落主题
3. 页面渲染
4. 我们的 `extend_footer.html` 注册手动覆盖监听 + 展示 Consent Banner
5. PaperMod `footer.html` 注册原有 toggle 监听（两者共存，互不干扰）

## Goals / Non-Goals

**Goals:**
- 页面加载时根据读者当地日出/日落时间自动设置亮/暗主题，不产生视觉闪烁（FOUC）
- 三层降级：L1 geolocation（精确）→ L2 时区推算（近似）→ L3 系统 `prefers-color-scheme`（兜底）
- 首次访问时展示非阻断式 Consent Banner，告知位置用途后再触发浏览器权限弹窗
- 保留 PaperMod 手动切换按钮；手动切换后 session 内自动逻辑不再覆盖
- 不修改 PaperMod 主题任何文件

**Non-Goals:**
- 不跨 session 持久化手动切换偏好（不改变 PaperMod 现有 `localStorage` 行为）
- 不做实时监听（如每隔一段时间重新判断日出日落），仅在页面加载时计算一次
- 不覆盖所有 IANA 时区（L2 仅覆盖约 400 个主要时区，偏远/不常见时区降至 L3）

## Decisions

### D1: 日出日落计算 — SunCalc.js 本地托管
使用 SunCalc.js（minified ~7KB），下载至 `static/js/suncalc.min.js` 本地托管。

在 `extend_head.html` 中用同步 `<script src>` 加载，确保后续内联脚本可以立即调用。

- 排除 CDN：外部 CDN 不可用时功能失效，影响主题初始化
- 排除自行实现：日出日落涉及精密天文计算，SunCalc 经过广泛验证

### D2: 自动主题逻辑 — extend_head.html 内联同步脚本
将自动主题设置逻辑放在 `extend_head.html` 的内联 `<script>` 中，紧跟 SunCalc.js 加载之后，在 PaperMod 初始化之后、页面渲染之前运行，避免 FOUC。

**extend_head.html 执行流程：**
```
sessionStorage["theme-manual"] 存在？
  → 是：跳过（尊重手动覆盖），结束
  → 否：
      sessionStorage["geo-coords"] 存在？
        → 是：用缓存坐标走 L1 计算
        → 否：走 L2（时区推算）
      根据结果设置 html.dataset.theme
```

### D3: Consent Banner — 页面底部 slim 条，非阻断式
在 `extend_footer.html` 注入一个固定在页面底部的细长 Banner（高度约 48px），仅在满足以下条件时展示：
- `sessionStorage["geo-consent-asked"]` 不存在（每 session 只问一次）
- `sessionStorage["geo-coords"]` 不存在（已有缓存坐标则不再问）

**Banner 文案：** "位置仅用于计算日出日落时间，数据不离开您的浏览器。"

**Banner 交互：**
- 点击「允许」→ 调用 `navigator.geolocation.getCurrentPosition()` → 成功：缓存坐标至 `sessionStorage["geo-coords"]`，重新计算并更新主题；失败：静默降级
- 点击「不了」→ 直接关闭
- 10 秒无操作 → 自动关闭
- 任意操作后设置 `sessionStorage["geo-consent-asked"] = "1"`

Banner 样式写入 `static/css/sunrise-theme.css`，在 extend_head.html 中 `<link>` 引入。

### D4: 手动覆盖 — sessionStorage flag
在 `extend_footer.html` 对 `#theme-toggle` 添加第二个点击监听器（与 PaperMod 原有监听器共存），点击时写入 `sessionStorage["theme-manual"] = "1"`。

extend_head.html 的自动逻辑在判断流程第一步检查此 flag，存在则完全跳过。不影响 PaperMod 写 `localStorage` 的行为。

### D5: L2 时区推算 — 内嵌精简映射表
使用 `Intl.DateTimeFormat().resolvedOptions().timeZone` 获取 IANA 时区名（如 `Asia/Shanghai`），查询内嵌在脚本中的精简映射表，得到该时区代表城市的经纬度，再交给 SunCalc.js 计算。

映射表作为 JS 对象字面量内嵌在 `extend_head.html` 内联脚本中，约 400 条主要时区，gzip 后约 3KB。

## Risks / Trade-offs

- **[SunCalc.js 加载失败]**: 同步加载失败（网络/文件缺失）会导致后续内联脚本报错，主题回退至 PaperMod 的 auto 逻辑（L3）→ 可接受，有自然兜底
- **[首次访问主题二次切换]**: 首次访问走 L2，用户在 Banner 点击「允许」后 L1 坐标更精确可能导致主题切换一次 → 仅首次访问出现，属已知 UX 取舍，L2 精度已足够大多数场景
- **[Geolocation 需用户手势触发]**: 部分浏览器要求 geolocation 在用户手势后调用，Banner 的「允许」按钮点击即满足此要求
- **[时区映射表静态维护]**: IANA 时区定义偶有更新，但影响极小；不匹配的时区降级至 L3
- **[性能]**: 同步加载 SunCalc.js (~7KB) 轻微增加首屏阻塞时间，远小于常规图片资源，可接受

## File Structure

```
layouts/partials/
  extend_head.html       # SunCalc 引入 + 自动主题逻辑（L1/L2/L3）+ CSS 引入
  extend_footer.html     # Consent Banner HTML/JS + 手动覆盖监听器

static/
  js/
    suncalc.min.js       # SunCalc.js 库文件
  css/
    sunrise-theme.css    # Consent Banner 样式
```
