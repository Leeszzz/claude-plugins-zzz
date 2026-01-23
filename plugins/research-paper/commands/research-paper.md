---
name: research-paper
description: 学术论文检索工具。使用 Semantic Scholar MCP 检索论文，并将结果记录到指定 markdown 文件。严格按用户关键词检索，自动过滤不相关结果。
allowed-tools: ["Read", "Write", "Edit", "mcp__semanticscholar__*"]
arguments:
  - name: keywords
    description: 检索关键词（如：微电网优化、氢燃料电池）
    required: true
  - name: years
    description: 年份范围（如：2020-2025），默认近5年
    required: false
---

# Research Paper 论文检索

## 功能说明

使用 Semantic Scholar MCP 进行学术论文检索，将检索结果记录到**当前工作目录**下的 `paper/论文.md`。

## 适用领域

本工具主要面向**工科研究方向**，包括但不限于：

- **电气工程**：电力系统、电力电子、电机控制、智能电网
- **微电网**：分布式能源、储能系统、能源管理、可再生能源并网
- **轨道交通**：牵引供电、列车控制、信号系统、运行优化
- **热流体**：传热学、流体力学、热管理、换热器设计
- **新能源**：氢能、光伏、风电、燃料电池

## 核心原则

### 1. 关键词匹配

根据用户描述提取关键词，构建检索查询：

```
# 示例
用户: "/research-paper 微电网能量管理"
query = "microgrid energy management optimization"

用户: "/research-paper 轨道交通牵引供电 2020-2024"
query = "railway traction power supply system"
yearStart = 2020, yearEnd = 2024
```

### 2. 去重机制

**避免重复检索**：
1. 每次检索前，先读取输出文件中已有的论文
2. 对比新检索结果，排除已记录的论文（通过论文标题判断）
3. 只将新论文追加到文件中

### 3. 相关性判断

检索完成后检查每篇论文：
- 标题是否与用户检索意图相关
- 排除明显不相关的论文（如医学、生物等无关领域）

## 检索流程

### 步骤 1：解析用户需求

从用户输入中提取：
- **关键词**：研究主题（如"微电网优化"、"氢燃料电池"）
- **年份范围**：如 2020-2025（默认近5年）
- **特殊要求**：如特定期刊、特定作者等

### 步骤 2：读取已有记录

读取输出文件，提取已记录的论文标题列表，用于后续去重。

### 步骤 3：构建检索查询

将中文关键词转换为英文检索词：

| 中文 | 英文 |
|------|------|
| 微电网 | microgrid |
| 储能 | energy storage |
| 电力系统 | power system |
| 轨道交通 | railway / rail transit |
| 牵引供电 | traction power supply |
| 热管理 | thermal management |
| 传热 | heat transfer |
| 氢能 | hydrogen energy |
| 燃料电池 | fuel cell |
| 电解槽 | electrolyzer |
| 光伏 | photovoltaic / solar |
| 风电 | wind power |
| 优化 | optimization |
| 控制 | control |
| 仿真 | simulation |

### 步骤 4：执行检索

使用 `mcp__semanticscholar__paper-search-advanced` 工具：

```
参数：
- query: 构建的检索词
- yearStart: 起始年份（默认当前年份-5）
- yearEnd: 结束年份（默认当前年份）
- fieldsOfStudy: ["Engineering", "Physics", "Computer Science", "Environmental Science"]
- limit: 20
- sortBy: relevance 或 citationCount
```

也可使用 `mcp__semanticscholar__search-arxiv` 检索 arXiv 预印本。

### 步骤 5：过滤与去重

1. **相关性过滤**：检查每篇论文标题是否与用户意图相关
2. **去重**：排除已在文件中记录的论文

### 步骤 6：获取摘要（可选）

对于用户特别感兴趣的论文，使用 `mcp__semanticscholar__get-paper-abstract` 获取详细摘要。

### 步骤 7：记录到文件

将符合条件的**新**论文追加到输出文件。

## 输出格式

每次检索后，将结果追加到文件，使用以下格式：

```markdown
---

## [YYYY-MM-DD] 关键词: xxx | 年份: xxxx-xxxx

| 英文题目 | 中文题目 | 期刊/会议 | 年份 | 引用数 | 链接 |
|----------|----------|-----------|------|--------|------|
| English Title | 中文翻译标题 | Journal | 2024 | 50 | [链接](url) |

共检索到 X 篇新论文。
```

**中文题目翻译要求**：
- 必须将英文标题翻译为准确的中文
- 专业术语保持学术规范
- 缩写/模型名称保持英文

## 常用检索领域代码

| 领域 | fieldsOfStudy 值 |
|------|------------------|
| 工程 | Engineering |
| 物理 | Physics |
| 计算机科学 | Computer Science |
| 环境科学 | Environmental Science |
| 材料科学 | Materials Science |
| 化学 | Chemistry |

## 推荐期刊/会议参考

以下是各方向的高质量期刊，供用户参考（不作为强制筛选条件）：

### 电气/电力系统
- IEEE Transactions on Power Systems
- IEEE Transactions on Power Electronics
- IEEE Transactions on Industrial Electronics
- IEEE Transactions on Smart Grid
- Electric Power Systems Research

### 能源/可再生能源
- Applied Energy
- Energy
- Renewable Energy
- Energy Conversion and Management
- International Journal of Hydrogen Energy

### 轨道交通
- IEEE Transactions on Vehicular Technology
- IEEE Transactions on Transportation Electrification
- Railway Engineering Science

### 热流体
- International Journal of Heat and Mass Transfer
- Applied Thermal Engineering
- International Communications in Heat and Mass Transfer

## 注意事项

1. **灵活匹配**：不要过于严格，只要论文与用户主题相关即可收录
2. **中英双语**：每篇论文必须包含英文题目和中文翻译题目
3. **记录完整**：每篇论文包含题目、期刊、年份、引用数、链接
4. **追加模式**：每次检索追加到文件末尾，不覆盖已有内容
5. **询问确认**：如果用户需求不明确，主动询问具体检索方向
