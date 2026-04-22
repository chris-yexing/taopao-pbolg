## ADDED Requirements

### Requirement: 页面加载时自动设置主题
The system SHALL 在每次页面加载时，根据读者当地的日出/日落时间自动将主题设置为 `light` 或 `dark`，并在 PaperMod 初始化之后立即覆盖，不产生可见的主题切换闪烁（FOUC）。

#### Scenario: 白天加载页面
- **GIVEN** 当前时间在日出之后、日落之前
- **WHEN** 读者加载任意页面
- **THEN** 系统将 `<html data-theme>` 设为 `light`

#### Scenario: 夜间加载页面
- **GIVEN** 当前时间在日落之后或日出之前
- **WHEN** 读者加载任意页面
- **THEN** 系统将 `<html data-theme>` 设为 `dark`

---

### Requirement: L1 — 使用地理位置精确计算
The system SHALL 当 sessionStorage 中存有缓存坐标（`geo-coords`）时，使用该坐标通过 SunCalc.js 计算当地精确日出日落时间。

#### Scenario: Session 内存在缓存坐标
- **GIVEN** sessionStorage 中 `geo-coords` 存有 `{lat, lng}`
- **AND** sessionStorage 中 `theme-manual` 不存在
- **WHEN** 页面加载，extend_head.html 脚本执行
- **THEN** 系统使用缓存坐标计算日出日落，应用对应主题

---

### Requirement: L2 — 时区推算降级
The system SHALL 当无缓存坐标时，通过 `Intl.DateTimeFormat().resolvedOptions().timeZone` 获取 IANA 时区名，查询内嵌映射表得到近似坐标，再通过 SunCalc.js 计算日出日落时间。

#### Scenario: 无缓存坐标但时区可识别
- **GIVEN** sessionStorage 中 `geo-coords` 不存在
- **AND** 读者时区在内嵌映射表中有对应条目
- **AND** sessionStorage 中 `theme-manual` 不存在
- **WHEN** 页面加载，extend_head.html 脚本执行
- **THEN** 系统使用时区近似坐标计算日出日落，应用对应主题
- **AND** 精度误差在 ±30 分钟以内（时区范围内的最大偏差）

---

### Requirement: L3 — 系统偏好兜底
The system SHALL 当时区也无法映射时，回退至读取 `window.matchMedia('(prefers-color-scheme: dark)')` 来设置主题。

#### Scenario: 时区不在映射表中
- **GIVEN** 读者时区不在内嵌映射表中
- **AND** sessionStorage 中 `theme-manual` 不存在
- **WHEN** 页面加载，extend_head.html 脚本执行
- **THEN** 系统根据系统深色模式偏好设置主题

---

### Requirement: 手动切换优先
The system SHALL 当 sessionStorage 中存在 `theme-manual` 标记时，完全跳过自动主题逻辑，不覆盖当前主题。

#### Scenario: 读者已手动切换主题
- **GIVEN** 读者在当前 session 内点击过手动切换按钮
- **WHEN** 读者导航至下一个页面（同 session）
- **THEN** 系统不执行自动主题设置，保持 PaperMod 从 localStorage 读取的主题
