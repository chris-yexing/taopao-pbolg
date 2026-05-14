# post-numbering

文章编号体系规范。本规范定义了两段式编号的格式、序号规则及展示方式。

## ADDED Requirements

### Requirement: 编号格式

每篇文章的编号 SHALL 由分类前缀和 3 位序号组成，格式为 `{前缀}-{3位序号}`，例如 `inv-001`、`tool-012`。

#### Scenario: 编号正确格式
- **WHEN** 文章 `category: investment` 且 `number: "003"`
- **THEN** 完整编号为 `inv-003`

#### Scenario: insight 分类编号
- **WHEN** 文章 `category: insight` 且 `number: "007"`
- **THEN** 完整编号为 `insight-007`

#### Scenario: 序号前导零
- **WHEN** 序号为 1 到 9
- **THEN** 编号中应显示为 `001` 到 `009`

### Requirement: 序号分配

序号 SHALL 由作者手动分配，同一分类下序号唯一，不得重复。

#### Scenario: 序号唯一性
- **WHEN** 同一分类下已有 `inv-001`
- **THEN** 新文章的序号不得为 `001`

#### Scenario: 序号范围
- **WHEN** 分类下文章数量超过 9 但不足 1000
- **THEN** 序号应为 `010` 到 `999`

### Requirement: frontmatter 字段

文章 frontmatter SHALL 包含 `number` 字段，格式为字符串（带前导零），例如 `number: "001"`。

#### Scenario: frontmatter 字段完整
- **WHEN** frontmatter 包含 `category: investment` 和 `number: "001"`
- **THEN** 系统生成编号 `inv-001`

#### Scenario: number 字段缺失
- **WHEN** 文章缺少 `number` 字段
- **THEN** 系统 MAY 使用默认值 `001`，并发出警告

### Requirement: 编号展示

编号 SHALL 在文章详情页和列表页中显示，显示位置为文章标题附近，视觉样式为次要文字（较小字号或较浅颜色）。

#### Scenario: 详情页展示
- **WHEN** 用户访问投资类文章
- **THEN** 页面在标题附近显示编号如 `inv-001`

#### Scenario: 列表页展示
- **WHEN** 用户浏览分类列表或标签页面
- **THEN** 每篇文章条目旁显示其编号