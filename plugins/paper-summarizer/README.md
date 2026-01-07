# Paper Summarizer

批量阅读学术文献并生成结构化总结的 Claude Code 插件。

## 功能特点

- 📚 **批量处理**：一次性处理多篇文献
- 📊 **表格汇总**：Brief 模式生成概览表格，快速浏览
- 📝 **详细分析**：Detailed 模式生成学术标准的深度分析
- 🔄 **上下文隔离**：每篇文献独立处理，避免上下文溢出
- 🌐 **在线补充**：可选集成 Semantic Scholar 补充元信息

## 支持格式

| 格式 | 扩展名 |
|------|--------|
| PDF | `.pdf` |
| Markdown | `.md` |
| 文本文件 | `.txt` |

## 安装

插件已安装在：`~/.claude/plugins/paper-summarizer/`

确保 Claude Code 加载了此插件目录。

## 使用方法

### 命令格式

```bash
/papers:summarize [选项] <文件或目录>
```

### 选项说明

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `--mode brief` | 表格汇总模式 | ✅ 默认 |
| `--mode detailed` | 详细分析模式 | - |
| `--output <file>` | Brief 模式输出文件 | `./paper-summary.md` |
| `--outdir <dir>` | Detailed 模式输出目录 | 当前目录 |
| `--enrich` | 启用 Semantic Scholar 在线补充 | 禁用 |

### 使用示例

```bash
# 简短模式 - 处理目录下所有 PDF，生成表格
/papers:summarize --mode brief ./papers/

# 详细模式 - 每篇生成独立的分析报告
/papers:summarize --mode detailed ./papers/*.pdf

# 指定输出文件
/papers:summarize --mode brief --output ./reading-notes.md ./papers/

# 启用在线信息补充
/papers:summarize --mode brief --enrich ./papers/

# 处理单个文件
/papers:summarize --mode detailed ./attention-is-all-you-need.pdf
```

## 输出格式

### Brief 模式（表格汇总）

生成一个 Markdown 文件，包含：

```markdown
# 文献阅读总结

> 生成时间: 2024-01-07
> 文献数量: 5 篇

## 文献概览表

| # | 标题 | 作者 | 年份 | 核心摘要 |
|---|------|------|------|----------|
| 1 | [标题](./file.pdf) | 作者 | 年份 | 摘要... |

## 关键词统计
## 统计信息
```

### Detailed 模式（详细分析）

为每篇文献生成单独的分析文件 `{filename}-summary.md`：

```markdown
# 文献详细分析：论文标题

## 📋 元信息
## 1. 研究背景与动机
## 2. 研究方法
## 3. 主要结果
## 4. 讨论与结论
## 5. 局限性与未来工作
## 📝 个人笔记
```

## 目录结构

```
paper-summarizer/
├── .claude-plugin/           # 插件标识
├── plugin.json               # 插件清单
├── README.md                 # 本文档
├── commands/
│   └── summarize.md          # /papers:summarize 命令
├── agents/
│   └── paper-reader.md       # 单篇文献处理 Agent
└── templates/
    ├── brief-table.md        # 表格输出模板参考
    └── detailed-report.md    # 详细报告模板参考
```

## 工作原理

```
┌─────────────────────────────────────────┐
│         /papers:summarize               │
│         (命令入口)                       │
└──────────────────┬──────────────────────┘
                   │
                   ▼
         ┌─────────────────┐
         │  扫描文献文件    │
         │  (Glob)         │
         └────────┬────────┘
                  │
    ┌─────────────┼─────────────┐
    │             │             │
    ▼             ▼             ▼
┌───────┐   ┌───────┐   ┌───────┐
│Agent 1│   │Agent 2│   │Agent 3│  (串行处理)
│文献 A │   │文献 B │   │文献 C │
└───┬───┘   └───┬───┘   └───┬───┘
    │           │           │
    └───────────┼───────────┘
                │
                ▼
       ┌────────────────┐
       │  汇总结果       │
       │  生成输出文档   │
       └────────────────┘
```

**关键设计**：每篇文献在独立的 Agent 上下文中处理，处理完成后只返回总结结果，原始文献内容不会累积在主对话中，从而避免上下文溢出。

## 可选功能：Semantic Scholar 集成

使用 `--enrich` 参数时，插件会通过 Semantic Scholar API 补充：

- 引用数
- DOI
- 期刊/会议名称
- 开放获取链接

**注意**：需要网络连接，且依赖 Semantic Scholar MCP 服务。

## 常见问题

### Q: PDF 无法解析怎么办？

部分扫描版 PDF 可能无法直接提取文本。建议：
1. 使用 OCR 工具预处理
2. 将 PDF 转换为文本/Markdown 格式
3. 使用支持 OCR 的 PDF 阅读器导出文本

### Q: 处理速度慢？

为保证稳定性，插件采用串行处理。如需加速：
1. 减少单次处理的文献数量
2. 优先使用 Brief 模式快速筛选

### Q: 总结质量不理想？

- 确保文献内容清晰可读
- Detailed 模式会提供更完整的分析
- 可手动编辑生成的总结文档

## 版本

- **0.1.0** - 初始版本
  - 支持 PDF、Markdown、文本文件
  - Brief 表格模式
  - Detailed 分析模式
  - Semantic Scholar 可选集成

## 许可

MIT License
