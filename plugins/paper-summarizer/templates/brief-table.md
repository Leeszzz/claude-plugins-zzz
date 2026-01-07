# Brief 模式输出模板

此模板定义了简短模式（表格汇总）的输出格式。

---

## 文档结构

```markdown
# 文献阅读总结

> 生成时间: {timestamp}
> 文献数量: {count} 篇
> 总结模式: 简短

---

## 文献概览表

| # | 标题 | 作者 | 年份 | 核心摘要 |
|---|------|------|------|----------|
| 1 | [{title}]({file_path}) | {authors} | {year} | {abstract} |
| 2 | ... | ... | ... | ... |

---

## 关键词统计

- {keyword_1} ({count_1})
- {keyword_2} ({count_2})
- ...

---

## 统计信息

| 指标 | 值 |
|------|-----|
| 文献数量 | {total} |
| 年份范围 | {min_year} - {max_year} |
| 成功处理 | {success_count} |
| 处理失败 | {fail_count} |

---

## 处理详情

### 成功处理

{success_list}

### 处理失败

| 文件 | 错误原因 |
|------|----------|
| {file} | {error_message} |

```

---

## 字段说明

| 字段 | 来源 | 说明 |
|------|------|------|
| `title` | Agent 返回 JSON | 论文标题，作为链接文本 |
| `file_path` | 输入参数 | 原始文件相对路径，作为链接地址 |
| `authors` | Agent 返回 JSON | 作者列表，超过 3 人用 "et al." |
| `year` | Agent 返回 JSON | 发表年份 |
| `abstract` | Agent 返回 JSON | 200 字以内摘要 |
| `keyword_N` | 汇总所有 keywords | 所有论文关键词的统计 |

---

## 表格格式规范

1. **标题列**：使用 Markdown 链接指向原文件
2. **作者列**：保持简洁，3 人以上用 "第一作者 et al."
3. **摘要列**：控制在 200 字以内，如过长自动截断
4. **对齐**：表格列默认左对齐

---

## 示例输出

```markdown
# 文献阅读总结

> 生成时间: 2024-01-07 14:30:00
> 文献数量: 3 篇
> 总结模式: 简短

---

## 文献概览表

| # | 标题 | 作者 | 年份 | 核心摘要 |
|---|------|------|------|----------|
| 1 | [Attention Is All You Need](./attention.pdf) | Vaswani et al. | 2017 | 提出 Transformer 架构，完全基于注意力机制，在机器翻译任务上取得显著提升... |
| 2 | [BERT: Pre-training of Deep Bidirectional Transformers](./bert.pdf) | Devlin et al. | 2018 | 提出双向预训练语言模型 BERT，通过掩码语言建模和下一句预测任务... |
| 3 | [GPT-3: Language Models are Few-Shot Learners](./gpt3.pdf) | Brown et al. | 2020 | 展示了大规模语言模型的少样本学习能力，1750 亿参数的 GPT-3... |

---

## 关键词统计

- Transformer (3)
- 自然语言处理 (3)
- 预训练 (2)
- 注意力机制 (2)
- 语言模型 (2)

---

## 统计信息

| 指标 | 值 |
|------|-----|
| 文献数量 | 3 |
| 年份范围 | 2017 - 2020 |
| 成功处理 | 3 |
| 处理失败 | 0 |
```
