---
description: 批量阅读文献并生成结构化总结（表格汇总或详细分析）
argument-hint: --mode brief|detailed [--output file.md] [--outdir dir/] [--enrich] <files or directory>
allowed-tools: ["Read", "Write", "Glob", "Task", "TodoWrite", "mcp__semanticscholar__*"]
---

# /paper:summarize 命令

批量处理学术文献，生成结构化总结。支持 PDF 和 Markdown/文本文件。

## 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--mode` | 总结模式：`brief`（表格）或 `detailed`（详细） | `brief` |
| `--output` | Brief 模式输出文件路径 | 文献所在目录，默认 `paper-summary.md` |
| `--outdir` | Detailed 模式输出目录 | 文献所在目录 |
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

### 4. 逐篇处理与输出逻辑

对每篇文献，按顺序执行以下步骤：

1. **更新状态**：将当前文献任务标记为 `in_progress`。
2. **启动分析**：使用 Task 工具启动 `paper-reader` agent。
   ```
   Task 调用示例:
   - subagent_type: paper:paper-reader
   - prompt: "处理文献 [file_path]，模式: [mode]"
   ```
3. **输出处理**：
   - **Detailed 模式**：每篇处理完后，将 Agent 返回的 Markdown 内容保存为 `{原文件名}-summary.md` 到 `--outdir`，**但不在对话中展示**。暂存文件路径列表供最后汇总。
   - **Brief 模式**：解析 Agent 返回的 JSON 结果并暂存在内存中。允许在**全部文献阅读完毕后**统一展示。
4. **标记完成**：将当前文献任务标记为 `completed`。

**重要提示**：
- `detailed` 模式：逐篇生成文件但不展示，全部完成后在对话中汇总已生成的文件列表及统计信息。
- `brief` 模式：批量阅读后汇总输出（全部阅读完再一起输出总结表格）。

### 5. 生成最终汇总

所有文献处理完成后：

#### Brief 模式
1. 按照模板生成完整汇总表格。
2. **在对话中向用户展示该汇总表格**。
3. 将汇总表格写入 `--output` 指定的文件。

#### Detailed 模式
1. **在对话中向用户汇总展示**：
   - 处理的文献总数
   - 每篇文献的标题及对应生成的总结文件路径
   - 处理成功/失败的统计信息
   - 所有输出文件的存放位置

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
/paper:summarize --mode brief ./papers/

# 详细模式 - 每篇单独文件
/paper:summarize --mode detailed ./papers/*.pdf

# 指定输出并启用在线补充
/paper:summarize --mode brief --output ./reading-notes.md --enrich ./papers/

# 处理单个文件
/paper:summarize --mode detailed ./attention-is-all-you-need.pdf
```

## 注意事项

1. **上下文管理**：通过 Agent 隔离处理每篇文献，避免上下文溢出
2. **串行稳定**：一篇一篇处理，确保每篇都能充分分析
3. **格式一致**：所有输出严格遵循模板格式
4. **中文输出**：所有总结内容使用中文
