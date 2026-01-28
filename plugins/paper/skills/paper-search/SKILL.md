---
name: paper-search
description: Use when needing to search academic papers in engineering fields (electrical, microgrid, railway, thermal, renewable energy). Searches Semantic Scholar, filters results, and saves to markdown with deduplication.
---

# 学术论文检索

## 概述

使用 Semantic Scholar MCP 进行学术论文检索,将结果记录到 markdown 文件。**核心原则：严格关键词匹配、自动去重、只收录相关论文。**

## 适用场景

- 用户需要检索工科研究论文(电气工程、微电网、轨道交通、热流体、新能源)
- 用户需要批量收集特定主题的最新研究成果
- 用户提到需要避免重复检索

**不适用于：**
- 非工科领域(医学、生物、人文社科)
- 单篇论文的详细分析(应使用 paper-reader Agent)

## 核心模式

### 处理流程
```
用户描述 → 提取关键词 → 中英文转换 → 检索 → 去重 → 追加保存

示例:
用户: "帮我找微电网能量管理的最新论文,2020年到现在的"
处理:
1. 提取: 关键词=微电网能量管理, 年份=2020-2024
2. 转换: query="microgrid energy management optimization"
3. 检索: paper-search-advanced(query, yearStart=2020, yearEnd=2024)
4. 去重: 读取已有记录,排除重复
5. 保存: 追加到 paper/论文.md
```

### 去重机制
```
每次检索前:
1. 读取输出文件 (paper/论文.md)
2. 提取已记录论文标题列表
3. 对比新检索结果
4. 只追加新论文
```

## 快速参考

### 关键词转换

中文关键词需转换为英文进行检索。

> 📚 **完整映射表**：请参考 [`references/keyword-mapping.md`](../../references/keyword-mapping.md)

**常用示例**：
- 微电网 → microgrid
- 储能 → energy storage
- 轨道交通 → railway / rail transit
- 氢能 → hydrogen energy
- 优化 → optimization

### MCP 工具

**主要检索:**
- `mcp__semanticscholar__paper-search-advanced`
  - query: 检索词
  - yearStart, yearEnd: 年份范围
  - fieldsOfStudy: 领域限定
  - limit: 结果数量(建议 20)
  - sortBy: relevance / citationCount

**补充工具:**
- `mcp__semanticscholar__search-arxiv`: arXiv 预印本
- `mcp__semanticscholar__get-paper-abstract`: 获取详细摘要

### 领域代码

| 领域 | fieldsOfStudy 值 |
|------|------------------|
| 工程 | Engineering |
| 物理 | Physics |
| 计算机科学 | Computer Science |
| 环境科学 | Environmental Science |
| 材料科学 | Materials Science |
| 化学 | Chemistry |

## 执行流程

### 1. 解析用户需求
从用户描述中提取:
- **关键词**: 研究主题(如"微电网优化"、"氢燃料电池")
- **年份范围**: 默认近5年,或用户指定
- **特殊要求**: 特定期刊、作者、会议等

### 2. 读取已有记录
```
1. 读取输出文件 (默认: paper/论文.md)
2. 提取已记录的论文标题列表
3. 用于后续去重
```

### 3. 构建检索查询
- 将中文关键词转换为英文(使用关键词转换表)
- 组合多个同义词提高召回率
- 设置合适的领域限定(fieldsOfStudy)

### 4. 执行检索
调用 `mcp__semanticscholar__paper-search-advanced`:
- query: 构建的英文检索词
- yearStart/yearEnd: 年份范围
- fieldsOfStudy: ["Engineering", "Physics", "Computer Science"]
- limit: 20
- sortBy: relevance 或 citationCount

### 5. 过滤与去重
**相关性过滤:**
- 检查每篇论文标题是否与用户意图相关
- 排除明显不相关的论文(如医学、生物等)

**去重:**
- 对比已记录的论文标题列表
- 排除重复论文

### 6. 翻译与记录
- 将英文标题翻译为准确的中文
- 专业术语保持学术规范
- 追加到输出文件

## 输出格式

```markdown
---

## [YYYY-MM-DD] 关键词: xxx | 年份: xxxx-xxxx

| 英文题目 | 中文题目 | 期刊/会议 | 年份 | 引用数 | 链接 |
|----------|----------|-----------|------|--------|------|
| English Title | 中文翻译标题 | Journal | 2024 | 50 | [链接](url) |

共检索到 X 篇新论文。
```

**翻译要求:**
- 必须准确翻译英文标题为中文
- 专业术语保持学术规范
- 缩写/模型名称保持英文

## 使用场景示例

### 场景 1: 基础检索
```
用户: "帮我找微电网能量管理的最新论文"
Claude:
1. 提取关键词: 微电网、能量管理
2. 转换为英文: "microgrid energy management"
3. 调用 paper-search-advanced
4. 读取 paper/论文.md 提取已有标题
5. 过滤不相关论文,去重
6. 翻译标题并追加到文件
7. 向用户报告: "共检索到 X 篇新论文"
```

### 场景 2: 指定年份范围
```
用户: "检索 2020-2024 年轨道交通牵引供电的论文"
Claude:
1. 提取: 关键词=轨道交通+牵引供电, 年份=2020-2024
2. 转换: "railway traction power supply system"
3. 设置 yearStart=2020, yearEnd=2024
4. 执行检索并去重
5. 追加到文件
```

### 场景 3: 多轮检索避免重复
```
用户第一次: "检索氢能相关论文"
Claude: 检索并保存 15 篇论文

用户第二次: "再找一些氢燃料电池的"
Claude:
1. 读取 paper/论文.md,提取已有的 15 篇标题
2. 检索"hydrogen fuel cell"
3. 对比新结果,排除重复
4. 只追加新论文(假设 8 篇)
5. 报告: "共检索到 8 篇新论文"
```

### 场景 4: 需求不明确时询问
```
用户: "帮我找一些论文"
Claude:
"请问您需要检索哪个领域的论文?我主要擅长工科领域,例如:
- 电气工程(电力系统、电力电子、微电网)
- 轨道交通(牵引供电、列车控制)
- 新能源(氢能、光伏、风电)
- 热流体(传热、热管理)
请告诉我具体的研究主题和年份范围。"
```

## 常见错误

### 重复检索
**问题:** 多次检索相同关键词导致文件中论文重复

**解决:** 每次检索前必须读取输出文件,提取已有标题,去重后再追加

### 不相关论文
**问题:** 检索结果包含无关领域论文

**解决:**
- 使用 fieldsOfStudy 参数限定领域
- 检索后人工检查每篇论文标题相关性
- 排除明显不相关的论文

### 检索词不精准
**问题:** 中文关键词直接使用或机器翻译不准确

**解决:**
- 使用关键词转换表
- 不确定时询问用户确认专业术语英文表达
- 结合多个同义词提高召回率

## 高质量期刊参考

**电气/电力系统:**
- IEEE Transactions on Power Systems
- IEEE Transactions on Power Electronics
- IEEE Transactions on Smart Grid

**能源/可再生能源:**
- Applied Energy, Energy, Renewable Energy
- International Journal of Hydrogen Energy

**轨道交通:**
- IEEE Transactions on Vehicular Technology
- IEEE Transactions on Transportation Electrification

**热流体:**
- International Journal of Heat and Mass Transfer
- Applied Thermal Engineering

## 最佳实践

1. **灵活匹配**: 不要过于严格,论文与主题相关即可收录
2. **中英双语**: 每篇论文必须包含英文和中文题目
3. **记录完整**: 包含题目、期刊、年份、引用数、链接
4. **追加模式**: 每次追加到文件末尾,不覆盖已有内容
5. **主动询问**: 需求不明确时主动询问具体方向
6. **去重检查**: 每次检索前必须读取已有记录并去重
