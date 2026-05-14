# post-category

文章分类体系规范。本规范定义了分类的枚举值、编号前缀及分类键的使用规则。

## ADDED Requirements

### Requirement: 分类枚举

系统 SHALL 定义以下 5 个分类：

| 分类键 | 中文标签 | 编号前缀 | 说明 |
|---|---|---|---|
| `investment` | 投资理财 | `inv` | 股票、ETF、资产配置相关文章 |
| `growth` | 成长生活 | `growth` | 育儿、备孕、健康——具体的人生阶段和身心状态 |
| `tool` | 效率工具 | `tool` | AI 工具、PC/NAS/网络折腾等数字系统 |
| `insight` | 认知碎片 | `insight` | 心理探索、人生感悟——内省和认知层面的散点思考 |
| `misc` | 其他 | `misc` | 不属于以上分类的文章 |

#### Scenario: growth 分类为空
- **WHEN** 当前没有任何文章归属 `growth` 分类
- **THEN** 分类页 `/categories/growth/` 正常显示为空列表，不影响构建

> **说明**：`growth` 为预留给未来内容的分类（备孕、育儿、健康等人生阶段类文章），目前文章库中尚无此类内容属正常状态。

#### Scenario: 新文章指定分类
- **WHEN** 作者在文章 frontmatter 中写入 `category: investment`
- **THEN** 该文章归属"投资方法论"分类

#### Scenario: 未知分类键被写入
- **WHEN** 作者写入未在枚举中的 `category` 值
- **THEN** 构建系统 MAY 发出警告，但不应阻断构建

### Requirement: 分类键使用规则

文章 frontmatter 中的 `category` 字段 SHALL 为字符串类型，取值必须为预定义分类键之一。

#### Scenario: 有效分类键
- **WHEN** 文章设置 `category: "tool"`
- **THEN** 编号前缀应为 `tool`

#### Scenario: 分类键缺失
- **WHEN** 文章未设置 `category` 字段
- **THEN** 系统应视为 `misc` 分类，显示前缀 `misc`