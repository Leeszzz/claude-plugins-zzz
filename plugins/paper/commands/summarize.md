---
description: "批量阅读文献并生成结构化总结(表格汇总或详细分析)。支持 PDF 和 Markdown/文本文件。"
argument-hint: --mode brief|detailed [--output file.md] [--outdir dir/] [--enrich] <files or directory>
allowed-tools: ["Read", "Write", "Glob", "Task", "TodoWrite", "mcp__semanticscholar__*"]
disable-model-invocation: true
---

调用 paper:summarize skill 并严格按照呈现的说明执行
