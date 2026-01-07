---
description: 批量阅读文献并生成结构化总结（表格汇总或详细分析）
argument-hint: --mode brief|detailed [--output file.md] [--outdir dir/] [--enrich] <files or directory>
allowed-tools: ["Read", "Write", "Glob", "Task", "TodoWrite", "mcp__semanticscholar__*"]
---

# /papers:summarize 命令

批量处理学术文献，生成结构化总结。支持 PDF 和 Markdown/文本文件。

## 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--mode` | 总结模式：`brief`（表格）或 `detailed`（详细） | `brief` |
| `--output` | Brief 模式输出文件路径 | `./paper-summary.md` |
| `--outdir` | Detailed 模式输出目录 | 当前目录 |
| `--enrich` | 启用 Semantic Scholar 在线补充元信息 | 禁用 |
| `<files>` | 文件路径、glob 模式或目录 | 必填 |

## 用户请求

```
$ARGUMENTS
```

## 执行流程

### 1. 解析参数

从用户输入中提取：
- `mode`: 默认 `brief`
- `output`: Brief 模式输出文件
- `outdir`: Detailed 模式输出目录
- `enrich`: 是否启用在线补充
- `files`: 待处理的文件列表

### 2. 扫描文件

使用 Glob 工具扫描指定路径，收集所有 `.pdf`、`.md`、`.txt` 文件：

```
支持的模式：
- 单个文件: ./paper.pdf
- 多个文件: ./paper1.pdf ./paper2.pdf
- Glob 模式: ./papers/*.pdf
- 目录: ./papers/（自动扫描其中的 pdf/md/txt）
```

### 3. 创建进度跟踪

使用 TodoWrite 创建任务列表，跟踪每篇文献的处理进度。

### 4. 串行处理每篇文献

对每篇文献，使用 Task 工具启动 `paper-reader` agent：

```
Task 调用示例:
- subagent_type: paper-summarizer:paper-reader
- prompt: "处理文献 [file_path]，模式: [mode]"
```

**重要**：串行处理（一篇一篇），确保稳定性，避免超时。

### 5. 收集结果

从每个 Agent 返回的结果中提取：
- Brief 模式：解析 JSON，提取 title/authors/year/abstract
- Detailed 模式：直接获取 Markdown 内容

### 6. 生成最终输出

#### Brief 模式

生成 Markdown 表格文档：

```markdown
# 文献阅读总结

> 生成时间: [当前日期时间]
> 文献数量: [N] 篇
> 总结模式: 简短

---

## 文献概览表

| # | 标题 | 作者 | 年份 | 核心摘要 |
|---|------|------|------|----------|
| 1 | [标题](./file.pdf) | 作者 | 年份 | 摘要... |
| ... | ... | ... | ... | ... |

---

## 关键词统计

- 关键词1 (出现次数)
- 关键词2 (出现次数)
- ...

---

## 统计信息

- 文献数量: N
- 年份范围: 最早 - 最新
- 成功处理: N 篇
- 处理失败: N 篇
```

将结果写入 `--output` 指定的文件（默认 `./paper-summary.md`）。

#### Detailed 模式

为每篇文献创建单独的总结文件：
- 文件名格式：`{原文件名}-summary.md`
- 存放位置：`--outdir` 指定的目录（默认当前目录）

### 7. 可选：在线补充信息

如果用户指定了 `--enrich` 参数：
1. 对每篇论文，使用 Semantic Scholar MCP 工具查询补充信息
2. 补充字段：引用数、DOI、期刊/会议、开放获取链接
3. 更新输出文档

使用的 MCP 工具：
- `mcp__semanticscholar__search-paper-title`: 按标题搜索论文
- `mcp__semanticscholar__get-paper-abstract`: 获取论文详细信息

### 8. 完成报告

处理完成后，输出统计摘要：
- 总文献数
- 成功处理数
- 失败数及原因
- 输出文件位置

## 错误处理

- 如果单篇文献处理失败，记录错误但继续处理其他文献
- 在最终报告中列出所有失败的文献及原因
- Brief 模式表格中，失败的文献标注为 "处理失败"

## 示例用法

```bash
# 简短模式 - 处理目录下所有 PDF
/papers:summarize --mode brief ./papers/

# 详细模式 - 每篇单独文件
/papers:summarize --mode detailed ./papers/*.pdf

# 指定输出并启用在线补充
/papers:summarize --mode brief --output ./reading-notes.md --enrich ./papers/

# 处理单个文件
/papers:summarize --mode detailed ./attention-is-all-you-need.pdf
```

## 注意事项

1. **上下文管理**：通过 Agent 隔离处理每篇文献，避免上下文溢出
2. **串行稳定**：一篇一篇处理，确保每篇都能充分分析
3. **格式一致**：所有输出严格遵循模板格式
4. **中文输出**：所有总结内容使用中文
