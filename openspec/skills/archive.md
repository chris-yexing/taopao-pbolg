# openspec-archive

当用户想要归档一个已完成的 OpenSpec 变更时，按以下步骤执行。

## 目标

将 `openspec/changes/<name>/` 移动到 `openspec/changes/archive/YYYY-MM-DD-<name>/`，完成变更的生命周期。

## 执行步骤

### 1. 确定目标变更

如果用户指定了变更名称，直接使用。否则：
- 列出 `openspec/changes/` 下的所有目录（不含 `archive/`）
- **不要自动猜测，必须让用户选择**

### 2. 检查制品完成情况

检查变更目录下是否存在以下文件：
- `proposal.md`
- `design.md`
- `tasks.md`
- `specs/` 目录下至少一个 spec.md

如有缺失，列出未完成的制品，询问用户是否确认继续归档。

### 3. 检查任务完成情况

读取 `tasks.md`，统计 `- [ ]`（未完成）和 `- [x]`（已完成）的数量。

如有未完成任务，提示用户："`<N> 个任务尚未完成"，询问是否确认继续归档。

### 4. 检查 delta specs 同步状态

检查 `openspec/changes/<name>/specs/` 下是否有规格文件。

如果存在，与 `openspec/specs/` 下对应的全局规格比较，判断是否需要同步。向用户展示差异摘要，询问：
- "立即同步到全局 specs/（推荐）"
- "跳过同步，直接归档"

### 5. 执行归档

生成目标路径：`openspec/changes/archive/YYYY-MM-DD-<name>/`（使用当前日期）

检查目标路径是否已存在：
- 若已存在：报错，提示用户处理冲突后重试
- 若不存在：将整个变更目录移动到目标路径

### 6. 输出归档摘要

```
归档完成：<name>
归档位置：openspec/changes/archive/YYYY-MM-DD-<name>/
Specs 同步：已同步 / 已跳过 / 无 delta specs
```

如有警告（不完整制品或未完成任务），一并列出。

## 注意事项

- 不要自动猜测或自动选择变更，必须明确用户意图
- 有未完成项时只警告不阻止，用户确认后继续
- `.openspec.yaml` 随目录一起移动，无需特殊处理
