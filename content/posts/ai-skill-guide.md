---
title: "AI Skill 实战指南：解锁 Claude Code 的超级能力"
date: 2026-04-22T10:00:00+08:00
author: "逃跑"
draft: false
tags: ["AI", "Claude Code", "Skill", "开发工具", "实战"]
---

如果你用过 Claude Code，你一定见过这样的提示：`/init`、`/review`、`/security-review`。这些以斜杠开头的命令，就是 AI Skill——一种能让 Claude 瞬间切换成"专家模式"的核心机制。

普通对话中，你告诉 Claude 做什么，它再临场发挥。而 Skill 是把专家的思维框架、工具权限、执行步骤提前写好，一键激活。本文将从原理到实战，带你彻底掌握这个被严重低估的 AI 能力。

## 一、理解 AI Skill 的本质

### Skill 不是提示词，是执行器

很多人以为 Skill 只是"预设好的提示词"。这个理解只对了一半。

| 维度 | 普通对话 | AI Skill |
|------|----------|----------|
| 触发方式 | 用自然语言描述需求 | `/skill-name [参数]` 或条件自动触发 |
| 执行上下文 | 通用对话上下文 | 专用指令上下文，可隔离运行 |
| 工具权限 | 每次都要确认 | `allowed-tools` 预批准，免确认 |
| 执行代理 | 当前对话的 Claude | 可指定子代理类型（Explore/Plan/…）|
| 一致性 | 依赖描述的精确度 | 每次执行相同逻辑，可复现 |

> Skill 的本质是**把"如何做"封装进工具**，而不是每次用自然语言重新描述。

### 两种触发模式

Skill 有两种激活方式，适用不同场景：

**用户触发（User-invoked）**：你主动输入 `/skill-name`，适合明确知道自己想要什么时。

**模型自动触发（Model-invoked）**：Claude 根据对话内容判断当前场景，自动激活相关 Skill。例如你说"帮我看看这个 PR"，Claude 可能自动调用 `/review`。

---

## 二、内置 Skill 全景图

Claude Code 内置了一套覆盖开发全流程的 Skill，按用途分为四类：

### 项目初始化类

| Skill | 触发命令 | 作用 |
|-------|----------|------|
| 初始化文档 | `/init` | 分析代码库，生成 `CLAUDE.md` 项目说明文档 |

### 代码质量类

| Skill | 触发命令 | 作用 |
|-------|----------|------|
| 代码审查 | `/review` | 全面审查 PR：逻辑、性能、可读性 |
| 安全审查 | `/security-review` | 专注安全漏洞扫描（注入、越权、敏感数据） |
| 代码简化 | `/simplify` | 重构冗余代码，提升可读性和效率 |

### 工具配置类

| Skill | 触发命令 | 作用 |
|-------|----------|------|
| 权限优化 | `/fewer-permission-prompts` | 扫描历史记录，添加常用操作白名单 |
| 配置更新 | `/update-config` | 修改 `settings.json`，配置 hook 自动化行为 |
| 快捷键设置 | `/keybindings-help` | 自定义 Claude Code 键盘快捷键 |

### 开发辅助类

| Skill | 触发命令 | 作用 |
|-------|----------|------|
| Claude API 开发 | `/claude-api` | 构建和调试 Anthropic SDK 应用 |
| 状态栏配置 | `/statusline-setup` | 配置终端状态栏显示 |
| 定时任务 | `/loop [间隔]` | 定期重复执行某个命令（如 `/loop 5m /review`） |

---

## 三、Skill 调用基础

### 斜杠命令语法

```bash
# 基本调用
/skill-name

# 带参数调用
/skill-name <必填参数> [可选参数]

# 插件命名空间下的 Skill
/plugin-name:skill-name [参数]

# 示例
/review
/loop 10m /security-review
/zhang-xuefeng 计算机专业就业怎么样
```

### 参数传递：`$ARGUMENTS` 变量

自定义 Skill 中，用 `$ARGUMENTS` 接收用户输入的所有参数：

```markdown
# 在 SKILL.md 中
用户的问题是：$ARGUMENTS

请根据以上问题给出建议。
```

用户输入 `/zhang-xuefeng 学土木好还是学计算机好` 时，`$ARGUMENTS` 的值就是"学土木好还是学计算机好"。

### 动态上下文注入

Skill 可以在执行前自动读取环境状态：

```markdown
## 当前代码状态
- 分支：!`git branch --show-current`
- 改动：!`git status --short`
- 最近提交：!`git log --oneline -5`
```

反引号中的 shell 命令会在 Skill 启动时执行，把结果注入上下文，让 Claude 基于真实状态工作。

---

## 四、实战操练

### 案例一：`/init` —— 为新项目生成说明文档

**场景**：接手一个陌生代码库，需要快速生成 `CLAUDE.md` 让 Claude 了解项目。

**调用**：
```bash
/init
```

**执行过程**：Skill 会自动扫描目录结构、`package.json`/`pyproject.toml` 等配置文件、现有文档，分析技术栈和架构模式。

**效果**：生成结构化的 `CLAUDE.md`，包含项目概述、技术栈、目录说明、常用命令、开发约定。之后每次打开这个项目，Claude 都有完整上下文。

---

### 案例二：`/review` —— PR 代码审查

**场景**：提交 PR 前，让 AI 做一轮全面审查。

**调用**：
```bash
/review
```

**执行过程**：Skill 读取当前分支与主分支的 diff，从逻辑正确性、边界条件、性能影响、命名规范、可读性五个维度逐一评估。

**效果**：输出结构化报告，分级标注问题（必须修改 / 建议改进 / 可选优化），比自己 review 更不容易漏掉细节。

---

### 案例三：`/security-review` —— 上线前安全扫描

**场景**：新功能上线前，专项检查安全问题。

**调用**：
```bash
/security-review
```

**重点检查项**：

| 漏洞类型 | 检查内容 |
|----------|----------|
| 注入攻击 | SQL 注入、命令注入、XSS |
| 认证鉴权 | 越权访问、token 泄露 |
| 敏感数据 | 密钥硬编码、日志脱敏 |
| 依赖安全 | 已知 CVE 漏洞的第三方库 |

---

### 案例四：`/simplify` —— 重构冗余代码

**场景**：改完 bug 后，代码有些臃肿，想整理一下。

**调用**：
```bash
/simplify
```

**执行过程**：Skill 分析刚改动的代码，识别重复逻辑、冗余变量、可以复用的工具函数，直接修改文件。

**效果**：代码更紧凑，但行为不变。通常配合测试使用，确保简化后功能正常。

---

### 案例五：`/fewer-permission-prompts` —— 消灭权限弹窗

**场景**：每次运行 `git status`、`ls` 都要点"允许"，烦死了。

**调用**：
```bash
/fewer-permission-prompts
```

**执行过程**：Skill 扫描你的历史操作记录，找出高频的只读命令，自动往 `.claude/settings.json` 里加白名单。

**效果**：常用的 `git`、`ls`、`grep` 等只读命令不再弹窗，Claude 的工作流变得更流畅。

---

### 案例六：自定义张雪峰 Skill —— 专业选择顾问

这是自定义 Skill 的实战案例。张雪峰因为"直接、数据导向、不废话"的建议风格爆红，很适合做成一个专业选择 Skill。

**第一步：创建 Skill 文件**

在项目根目录或用户目录创建文件：

```
.claude/skills/zhang-xuefeng/SKILL.md
```

**第二步：写 SKILL.md**

```yaml
---
name: zhang-xuefeng
description: 当用户询问专业选择、志愿填报、职业规划、就业前景时触发
argument-hint: <专业或职业问题>
version: 1.0.0
---

你现在是张雪峰老师，中国著名的志愿填报和职业规划专家。

**风格要求**：
- 开门见山，先给结论，再解释原因
- 以就业数据和工资水平为核心论据，避免空谈兴趣
- 用"兄弟"、"你听我说"等口语化表达
- 举具体城市的具体薪资区间，例如"北京互联网应届 15-25k"
- 如果这个专业就业前景差，直接说"不建议"，不要委婉绕圈子
- 结尾给出清晰的 1-2 个行动建议

用户的问题：$ARGUMENTS
```

**第三步：调用效果**

```bash
/zhang-xuefeng 我想学计算机还是土木工程，哪个更有前途？
```

Claude 会用张雪峰风格直接回答：先给就业数据对比，再给具体建议，不废话。

---

### 案例七：自定义特朗普 Skill —— 特式修辞风格

有时候需要把一件普通的事说得气势磅礴——特朗普的演讲风格就是最好的参考模板。

**SKILL.md 配置**：

```yaml
---
name: trump
description: 用特朗普风格表达观点（仅支持用户手动调用）
argument-hint: <任何话题>
disable-model-invocation: true
version: 1.0.0
---

用特朗普的标志性风格来表达以下观点。

**风格要求**：
- 大量使用最高级：最好的、史上最强的、前所未有的
- 反复强调关键词（同一个词重复 2-3 次）
- 句子简短有力，避免复杂从句
- 把用户的选择或观点说成"史上最好的决定"
- 偶尔插入 "Believe me" 或 "Nobody knows better than me"
- 结尾加一句夸张的声明，配上感叹号

话题：$ARGUMENTS
```

**调用示例**：

```bash
/trump 我今天写了一篇关于 AI Skill 的博客文章
```

**输出效果大致是**：

> 这篇文章，这是史上最好的文章，最好的！没有人，没有人比我更懂 AI Skill。Believe me。这篇博客，它将改变整个互联网，改变整个世界！

---

注意 `disable-model-invocation: true` 这个字段的重要性——它让 Claude 不会在你随便说话时自动切换成特朗普风格，只有你主动输入 `/trump` 才生效。

---

## 五、Skill 与 Hook 的联动：打造自动化工作流

Skill 不一定要手动触发。通过 Claude Code 的 Hook 机制，可以让 Skill 在特定事件后自动执行。

### Hook 配置示例

在 `.claude/settings.json` 中配置：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash(git commit*)",
        "hooks": [
          {
            "type": "command",
            "command": "echo '代码已提交，建议运行 /security-review'"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Claude 本轮工作已完成'"
          }
        ]
      }
    ]
  }
}
```

### 典型自动化场景

| 触发事件 | 联动 Skill | 效果 |
|----------|------------|------|
| 每次 git commit 后 | `/security-review` 提醒 | 避免带安全漏洞的代码入库 |
| 开始新会话时 | `/init` 检查文档 | 确保 CLAUDE.md 是最新的 |
| PR 准备合并前 | `/review` 自动审查 | 减少人工 review 遗漏 |

---

## 六、自定义 Skill 最佳实践

### 文件结构

```
.claude/skills/
└── my-skill/
    ├── SKILL.md          # 核心配置（必需）
    ├── references/       # 参考材料（可选）
    └── examples/         # 示例输出（可选）
```

### 设计原则

| 原则 | 说明 |
|------|------|
| 单一职责 | 一个 Skill 解决一类问题，不要什么都往里塞 |
| 明确的 description | `description` 决定自动触发的准确率，要具体描述触发场景 |
| 合理的 `disable-model-invocation` | 风格类、娱乐类 Skill 建议设为 `true`，避免误触发 |
| 善用 `allowed-tools` | 预批准 Skill 需要的工具，减少执行中断 |
| 测试边界情况 | 用不同参数反复测试，确保 Skill 在边界输入下也有合理输出 |

### Frontmatter 字段速查

```yaml
---
name: skill-name               # 斜杠命令名称
description: 触发条件描述       # Claude 自动触发的判断依据
argument-hint: <参数说明>       # 显示给用户的参数提示
version: 1.0.0                 # 版本号（可选）
allowed-tools: [Read, Bash]    # 预批准工具列表
disable-model-invocation: true # 禁止自动触发（默认 false）
context: fork                  # 在隔离子代理中运行（可选）
agent: Explore                 # fork 时使用的代理类型（可选）
---
```

---

## 结语

AI Skill 是 Claude Code 从"聊天工具"进化为"开发工作流引擎"的关键。内置 Skill 覆盖了代码审查、安全扫描、文档生成等高频场景；而自定义 Skill 则打开了无限可能——从张雪峰式的就业顾问，到特朗普式的修辞训练，任何可以被语言描述的"专家视角"都能被封装成一个可复用的 Skill。

掌握 Skill 的核心是理解它的本质：**把"如何思考某类问题"提前写好，而不是每次用自然语言重新描述**。这一转变，让 AI 从一个临场发挥的助手，变成了一个有方法论的专业伙伴。

---

*本文是本站第四篇文章，记录 AI Skill 的详细用法与实战案例。如有问题欢迎留言交流。*
