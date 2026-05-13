## ADDED Requirements

### Requirement: 首次访问展示 Consent Banner
The system SHALL 在读者每个 session 内首次访问时展示一条底部 slim banner，告知地理位置权限的用途，并在读者做出选择后消失。

#### Scenario: Session 内首次加载页面
- **GIVEN** sessionStorage 中 `geo-consent-asked` 不存在
- **AND** sessionStorage 中 `geo-coords` 不存在
- **AND** 浏览器支持 Geolocation API
- **WHEN** 页面完成加载
- **THEN** 系统在页面底部展示 Consent Banner
- **AND** Banner 包含说明文字、「允许」和「不了」两个按钮

#### Scenario: Session 内已作过选择
- **GIVEN** sessionStorage 中 `geo-consent-asked` 已存在
- **WHEN** 读者访问任意页面
- **THEN** 系统不展示 Consent Banner

---

### Requirement: Banner 说明文字透明告知用途
The system SHALL 在 Banner 中展示说明文字，明确告知读者：位置信息仅用于计算日出日落时间，数据不离开浏览器，不上报任何服务器。

#### Scenario: Banner 展示时文案可读
- **WHEN** Consent Banner 可见
- **THEN** Banner 包含文案"位置仅用于计算日出日落时间，数据不离开您的浏览器"

---

### Requirement: 读者点击「允许」后触发地理位置请求
The system SHALL 在读者点击「允许」按钮后，立即关闭 Banner 并触发浏览器原生地理位置权限请求。权限授予后缓存坐标并重新计算主题。

#### Scenario: 读者点击「允许」且浏览器授权
- **WHEN** 读者点击「允许」按钮
- **THEN** Banner 立即关闭
- **AND** 浏览器弹出系统地理位置权限对话框
- **AND** 若读者在系统对话框中授权，坐标缓存至 `sessionStorage["geo-coords"]`
- **AND** 系统用 L1 坐标重新计算并更新当前页面主题

#### Scenario: 读者点击「允许」但浏览器拒绝授权
- **WHEN** 读者点击「允许」按钮后，在系统对话框中拒绝
- **THEN** 系统静默保持当前 L2/L3 主题，不报错

---

### Requirement: 读者点击「不了」或超时后静默关闭
The system SHALL 在读者点击「不了」按钮、或 Banner 展示超过 10 秒无操作时，自动关闭 Banner 并继续使用 L2/L3 主题。

#### Scenario: 读者点击「不了」
- **WHEN** 读者点击「不了」按钮
- **THEN** Banner 立即关闭
- **AND** 设置 `sessionStorage["geo-consent-asked"] = "1"`
- **AND** 系统继续使用已生效的 L2/L3 主题，不再提示

#### Scenario: Banner 展示超过 10 秒无操作
- **GIVEN** Banner 已展示
- **WHEN** 10 秒内读者未点击任何按钮
- **THEN** Banner 自动关闭（动效淡出）
- **AND** 设置 `sessionStorage["geo-consent-asked"] = "1"`
