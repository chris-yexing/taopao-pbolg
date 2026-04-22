---
title: "如何在这个博客手动写文章并发布"
date: 2026-04-22T21:00:00+08:00
author: "逃跑"
draft: false
tags: ["博客", "Hugo", "教程", "写作"]
---

这个博客之前都是 AI 自动生成文章的。从现在开始，我想自己写，所以记录一下整个流程，也方便以后回头查。

## 一、先确认环境

博客用 [Hugo](https://gohugo.io/) 驱动，写文章之前先确认本地装了 Hugo。打开终端（Windows 用 Git Bash 或 PowerShell）执行：

```bash
hugo version
```

正常的话会输出类似这样的内容：

```
hugo v0.160.1+extended ...
```

如果提示命令找不到，去 [Hugo 官方安装页](https://gohugo.io/installation/) 下载对应系统的版本安装好再继续。

Git 也是必须的，用来最后推送代码。验证方式一样：

```bash
git --version
```

## 二、创建新文章文件

在项目根目录执行这条命令：

```bash
hugo new content posts/my-article.md
```

`my-article` 换成你想要的文件名，建议用英文小写加连字符，比如 `reading-notes-2026.md`、`thoughts-on-minimalism.md`。文件名会出现在文章的 URL 里，所以中文文件名也能用，但不太雅观。

命令执行后，Hugo 会在 `content/posts/` 目录下生成这个文件，并自动填入 frontmatter 模板：

```yaml
---
title: "My Article"
date: 2026-04-22T21:00:00+08:00
author: "逃跑"
draft: false
tags: []
---
```

## 三、填写 Frontmatter

Frontmatter 是文件顶部两行 `---` 之间的部分，控制文章的标题、时间、标签等元数据。字段说明：

| 字段 | 说明 | 示例 |
|------|------|------|
| `title` | 文章页面显示的标题 | `"我为什么开始写博客"` |
| `date` | 发布时间，保留 `+08:00` 时区后缀 | `2026-04-22T21:00:00+08:00` |
| `author` | 固定填 `"逃跑"`，不用改 | `"逃跑"` |
| `draft` | `false` 才会正式发布；写作中可先设 `true` | `false` |
| `tags` | 分类标签，数组格式，可以多个 | `["读书", "随笔"]` |

`title` 就是页面的大标题，PaperMod 主题会自动把它渲染成 H1，所以**正文里不要再写 `# 标题`**，否则页面会出现两个标题。

## 四、写正文

### 章节结构

章节标题统一用中文数字：

```markdown
## 一、背景

这里写内容……

## 二、核心内容

这里写内容……

## 三、结尾

这里写内容……
```

用二级标题 `##` 划分主要章节，三级标题 `###` 用于章节内的子话题。不要堆太多层级，结构保持扁平，读起来更顺。

### 表格和引用块

比较多个选项、列对照关系的内容用表格：

```markdown
| 列A | 列B | 列C |
|-----|-----|-----|
| 内容 | 内容 | 内容 |
```

需要突出的关键定义或重要引用，用 `>` 引用块：

```markdown
> 这里放需要特别强调的内容。
```

### 代码块

写命令或代码时用三个反引号包裹，并标注语言：

````markdown
```bash
git push origin master
```
````

````markdown
```yaml
draft: false
```
````

### 关于写作风格

说实话，这部分比格式更重要。我设了一些对自己的要求，写的时候尽量遵守：

- 用"我"，不用"我们"——这是我一个人的博客，没有我们
- 不用"综上所述"、"值得注意的是"、"首先……其次……最后……"这类套话，读起来像机器写的
- 连贯的想法用段落，不要什么都拆成条目列表
- 可以有自己的立场和判断，不用全程中立客观
- 口语化没问题，"说实话"、"我觉得"、"老实讲"都行
- 不确定的事情可以说不确定，"也许是……"比假装什么都懂要诚实
- 结尾不用再把全文总结一遍，读者刚读完，不需要

## 五、本地预览

文章写好后，先在本地看效果，再推送：

```bash
hugo server -D
```

`-D` 参数让 `draft: true` 的草稿文章也显示出来。命令执行后访问 `http://localhost:1313`，可以实时看到文章效果，编辑文件保存后页面会自动刷新。

确认排版、标题、标签都没问题，按 `Ctrl+C` 停止服务器。

## 六、用 Git 提交并推送到仓库

Git 是这套流程里最后也是最关键的一步。本地写好的文章，要通过 Git 推送到 GitHub 仓库，才能触发自动部署。

### 第一步：确认当前状态

在提交之前，先看看 Git 认为哪些文件发生了变化：

```bash
git status
```

输出大概是这样：

```
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        content/posts/my-article.md
```

`Untracked files` 表示这是新文件，Git 还没有追踪它。如果是修改了已有文章，会显示 `modified` 而不是 `Untracked`。

### 第二步：暂存文件

"暂存"是 Git 的一个中间步骤，相当于告诉 Git"我要提交这个文件"。只暂存文章文件本身：

```bash
git add content/posts/my-article.md
```

> 不要用 `git add .` 或 `git add -A`，这两个命令会把当前目录所有变动的文件都加进去，容易把不相关的临时文件或配置一起提交进去。

暂存之后再执行 `git status`，会看到文件状态变成了绿色的 `new file`：

```
Changes to be committed:
        new file:   content/posts/my-article.md
```

### 第三步：创建提交

提交（commit）是把暂存的改动正式记录到 Git 历史里。`-m` 后面跟提交信息：

```bash
git commit -m "post: 文章标题简述"
```

提交信息建议用 `post:` 开头，后面跟文章主题，简短清晰就行，比如：

```bash
git commit -m "post: 如何在博客写文章并发布"
git commit -m "post: 2026年四月读书笔记"
```

提交成功后会输出类似：

```
[master 86789d7] post: 如何在博客写文章并发布
 1 file changed, 100 insertions(+)
 create mode 100644 content/posts/my-article.md
```

括号里那串字母数字（`86789d7`）是这次提交的唯一 ID，Git 用它追踪历史记录。

### 第四步：推送到远程仓库

到这步为止，改动还只在本地。推送才是真正把文章送到 GitHub 的操作：

```bash
git push origin master
```

`origin` 是远程仓库的别名（默认名称），`master` 是分支名。推送成功后输出类似：

```
To https://github.com/你的用户名/pblog.git
   d2f28f8..86789d7  master -> master
```

### 第五步：等待自动部署

推送完成后，GitHub Actions 会自动检测到新的提交，触发构建和部署流程。整个过程大约 1-2 分钟。

可以在 GitHub 仓库页面点 **Actions** 标签，查看部署进度——绿色勾表示成功，红色叉表示构建出了问题。

部署成功后，刷新博客网站，应该能看到新文章出现在列表里。

### 如果推送失败

偶尔会遇到推送被拒绝的情况，报错信息类似：

```
! [rejected]  master -> master (fetch first)
```

这通常是因为远程仓库有你本地没有的提交。先把远程的更新拉下来再推送：

```bash
git pull origin master
git push origin master
```

如果 `git pull` 后出现合并冲突（conflict），终端会提示哪些文件有冲突，手动打开文件解决冲突标记后，重新 `git add` 和 `git commit`，再推送。

## 七、用 VSCode 图形界面提交（不想敲命令的话）

如果觉得命令行麻烦，VSCode 内置了 Git 图形界面，鼠标点几下就能完成暂存、提交和推送，和命令行效果完全一样。

### 打开源代码管理面板

点击左侧活动栏的**分支图标**（Source Control），或者按快捷键 `Ctrl+Shift+G`。

面板会列出所有有变动的文件。新写的文章会出现在"**更改**"（Changes）分组里，图标是 `U`（Untracked，未追踪的新文件）；修改过的旧文章图标是 `M`（Modified）。

### 暂存文件

鼠标悬停在文章文件名上，右侧会出现几个小图标，点击 **`+`**（暂存更改）。

文件会从"更改"移动到"**已暂存的更改**"（Staged Changes）分组，这和命令行里的 `git add` 是同一个操作。

> 不要点"更改"分组标题旁边的那个 **`+`**，那个是"暂存所有更改"，相当于 `git add -A`，会把所有文件都加进去。只点文章文件那一行的 `+`。

### 填写提交信息并提交

面板顶部有一个输入框，提示文字是"消息（按 Ctrl+Enter 提交...）"。

在里面输入提交信息，格式和命令行一样：

```
post: 文章标题简述
```

输入完成后，点输入框下方的蓝色 **"提交"** 按钮，或者按 `Ctrl+Enter`。提交成功后"已暂存的更改"列表会清空。

### 推送到远程仓库

提交完成后，面板底部状态栏会显示一个云朵图标，旁边有向上的箭头和数字（表示有 1 个提交待推送）。

有两种方式推送：

**方式一**：点击状态栏那个云朵/箭头图标，直接推送。

**方式二**：点面板右上角的 **`···`** 菜单，选择 **"推送"**（Push）。

推送成功后状态栏的数字消失，说明远程仓库已经收到了这次提交，GitHub Actions 随即开始自动部署。

### 如果遇到"需要先拉取"的提示

VSCode 有时会弹出提示说"无法推送，因为远端包含你本地没有的工作"，点 **"同步更改"** 按钮，它会自动先 pull 再 push，等同于命令行的：

```bash
git pull origin master
git push origin master
```

## 八、完整流程速查

贴一个最简版本，熟练了之后大概就是这几步：

```bash
# 1. 新建文章
hugo new content posts/my-article.md

# 2. 编辑文章内容（填 frontmatter，写正文）

# 3. 本地预览
hugo server -D
# 浏览器打开 http://localhost:1313

# 4. 提交推送
git add content/posts/my-article.md
git commit -m "post: 文章标题"
git push origin master
```

部署完成后去网站确认文章正常显示，就算发布成功了。
