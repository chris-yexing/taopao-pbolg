## MODIFIED Requirements

### Requirement: 手动切换按钮保留且可见
The system SHALL 在所有页面保留 PaperMod 原有的手动主题切换按钮（`#theme-toggle`），不受自动主题功能的影响。

#### Scenario: 任意页面加载
- **GIVEN** `site.Params.disableThemeToggle` 未设置或为 false
- **WHEN** 读者加载任意页面
- **THEN** 页面头部显示月亮/太阳图标切换按钮

---

### Requirement: 手动切换在 session 内优先于自动逻辑
The system SHALL 当读者点击手动切换按钮时，在 sessionStorage 中记录手动覆盖标记，使当前 session 内后续所有页面加载时跳过自动主题设置。

#### Scenario: 读者点击手动切换按钮
- **WHEN** 读者点击 `#theme-toggle` 按钮
- **THEN** PaperMod 原有逻辑正常执行（切换主题 + 写 localStorage）
- **AND** 系统额外写入 `sessionStorage["theme-manual"] = "1"`

#### Scenario: 手动切换后导航至下一页
- **GIVEN** 读者已在本 session 内手动切换过主题
- **WHEN** 读者点击文章链接进入新页面
- **THEN** 新页面主题由 PaperMod 从 localStorage 读取，不被自动日出日落逻辑覆盖

---

### Requirement: 不修改 PaperMod 原有 localStorage 行为
The system SHALL 不修改、不清除 PaperMod 写入 localStorage 的 `pref-theme` 值，确保跨 session 的用户偏好记忆继续正常工作。

#### Scenario: 读者关闭并重新打开浏览器
- **GIVEN** 读者在上一个 session 中手动切换为 dark 主题
- **WHEN** 读者重新访问博客（新 session）
- **THEN** PaperMod 从 localStorage 读取 `pref-theme = "dark"` 并应用
- **AND** 自动日出日落逻辑重新生效（新 session 无 `theme-manual` 标记），可能再次覆盖为日出日落对应主题
