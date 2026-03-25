# Research with Receipts

A Claude Code plugin for structured, verified research. Every claim comes with a receipt.

## What This Does

Most AI research tools generate reports from scratch. This one does the opposite — it **extracts and verifies** information from documents you already have, ensuring every data point is traceable to a specific source.

**The problem:** LLMs hallucinate. Even RAG systems like NotebookLM, while significantly better, are not hallucination-free. When you're extracting data from annual reports, proxy statements, or research papers, a single fabricated data point can invalidate your entire analysis.

**The solution:** A structured 7-step workflow that separates extraction from verification, uses multi-round validation, and produces a Word document where every claim has a traceable source.

## How It Works

```
Step 1: Source Preparation    → Get documents into a local folder
Step 2: Import to NotebookLM  → Build a queryable knowledge base
Step 3: Define Dimensions     → Specify exactly what to extract
Step 4: Pre-flight Check      → Test one object to validate feasibility
Step 5: Batch Extraction      → Query NotebookLM for every object × dimension
Step 6: Multi-Round Verify    → Three rounds of independent verification
Step 7: Word Output           → Generate traceable report
```

### Multi-Round Verification

The core value. Three independent rounds catch different types of errors:

- **Round 1 (Self-audit):** Does the quoted source actually support the claim?
- **Round 2 (Re-query):** Ask the same question differently — do answers match?
- **Round 3 (Spot-check):** Random 20% sample — if any errors found, full re-check.

### Confidence Ratings

Every data point gets a status:

| Status | Meaning |
|--------|---------|
| ✅ | Direct evidence with source citation |
| ⚠️ | Indirect evidence — reasoning documented |
| ❌ | No data available in provided sources |

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- Node.js >= 18
- A Google account (free tier works, 50 queries/day)
- Chrome browser (for NotebookLM authentication)

### Install the plugin

```bash
# Install NotebookLM MCP (required)
claude mcp add notebooklm npx notebooklm-mcp@latest

# Install python-docx (for Word output)
pip install python-docx

# Install this plugin
claude plugin add github:Kewanvk/research-with-receipts
```

### First-time setup

```bash
# Authenticate with NotebookLM (opens Chrome)
nlm login
```

## Usage

In Claude Code, trigger the skill with natural language:

```
"Help me extract data from these annual reports"
"I have some PDFs in ~/reports/, help me research them"
"Verify the claims in this document"
"Collect data from these proxy statements"
```

The skill will guide you through each step interactively.

## Why Not Just Use NotebookLM Directly?

NotebookLM is an excellent "librarian" — you ask a question, it finds the answer in your documents. But a librarian is not a researcher. Here's what this skill adds on top:

|  | NotebookLM alone | Research with Receipts |
|--|-----------------|----------------------|
| **Extraction** | One question at a time, manually | Batch extraction: dozens of objects × dimensions, automated |
| **Output** | Free-text answers in chat | Structured Word document with source traceability |
| **Self-check** | No built-in verification | Three independent rounds of verification |
| **Source grading** | All sources treated equally | Sources graded A–D; indirect evidence flagged |
| **When it doesn't know** | May infer or speculate | Hard rule: no data = "no data", never guess |
| **Error patterns** | No awareness of common mistakes | 8 documented anti-hallucination rules from real failures |
| **Consistency** | Same question may yield different phrasing | Re-query with rephrased questions to check consistency |

**In short:** NotebookLM is the engine; this skill is the methodology. You wouldn't submit raw database query results as a research report — same idea here.

## Project Structure

```
research-with-receipts/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── research-with-receipts/
│       ├── SKILL.md                        # Core workflow (7 steps)
│       └── references/
│           ├── verification-rules.md       # Source grading + 8 error patterns
│           └── agent-workflow.md           # Multi-round verification details
├── README.md
└── LICENSE
```

## License

MIT

---

# Research with Receipts（中文说明）

一个 Claude Code 插件，用于结构化、可验证的研究。每条数据都有据可查。

## 这东西解决什么问题

市面上大多数 AI 研究工具都是"生成式"的——给个话题，它帮你写报告。但在真实的研究场景里，你需要的往往不是从零生成，而是**从已有资料中提取信息，并确保每条数据都是对的**。

**痛点：** LLM 会幻觉。即便是 NotebookLM 这样的 RAG 系统，虽然比直接用大模型好很多，但仍然不是零幻觉。当你在从年报、代理声明、研究论文中提取数据时，一条编造的数据就能毁掉整个分析。

**解法：** 一套 7 步工作流，把"提取"和"验证"分开，通过多轮校验确保准确性，最终输出一份每条数据都能追溯来源的 Word 文档。

## 工作流程

```
第一步：资料准备        → 把文档放进本地文件夹
第二步：导入 NotebookLM  → 建立可查询的知识库
第三步：定义提取维度      → 明确要提取什么信息
第四步：预检            → 拿一个对象试跑，验证可行性
第五步：批量提取         → 对每个对象 × 每个维度向 NotebookLM 提问
第六步：多轮验证         → 三轮独立验证，层层过滤错误
第七步：输出 Word 文档    → 生成可追溯的研究报告
```

### 多轮验证（核心价值）

三轮验证，每轮抓不同类型的错误：

- **第一轮（自查）：** 引用的原文真的支持这个结论吗？
- **第二轮（换个问法再问）：** 同一个问题换一种方式问 NotebookLM，答案一致吗？
- **第三轮（抽查）：** 随机抽查 20% 的数据，发现错误则全量复查。

### 置信度标记

每条数据都有明确状态：

| 状态 | 含义 |
|------|------|
| ✅ | 有直接证据，附原文引用 |
| ⚠️ | 间接证据，附推断依据 |
| ❌ | 资料中无相关数据 |

## 和直接用 NotebookLM 有什么区别？

NotebookLM 是一个很好的"图书管理员"——你问它什么，它帮你从资料里找答案。但图书管理员不是研究员。这个 skill 在 NotebookLM 之上加了什么：

|  | 直接用 NotebookLM | Research with Receipts |
|--|-------------------|----------------------|
| **提取方式** | 一次问一个问题，手动操作 | 批量提取：几十个对象 × 多个维度，自动化执行 |
| **输出格式** | 聊天窗口里的自然语言回答 | 结构化 Word 文档，每条数据标注来源 |
| **自我校验** | 没有内置验证机制 | 三轮独立验证 |
| **来源分级** | 所有来源一视同仁 | 来源按 A–D 分级，间接证据单独标记 |
| **查不到时** | 可能推测或编造 | 硬规则：没有数据就写"无数据"，绝不猜测 |
| **错误模式** | 无感知 | 8 条反幻觉规则，来自真实项目的血泪教训 |
| **一致性检查** | 无 | 换种问法重新提问，检查答案是否一致 |

**一句话总结：NotebookLM 是引擎，这个 skill 是方法论。** 你不会把数据库查询结果直接当研究报告交出去——同理。

## 安装

### 前置条件

- 已安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- Node.js >= 18
- Google 账号（免费版即可，每天 50 次查询）
- Chrome 浏览器（用于 NotebookLM 认证）

### 安装步骤

```bash
# 安装 NotebookLM MCP（必需）
claude mcp add notebooklm npx notebooklm-mcp@latest

# 安装 python-docx（用于生成 Word 文档）
pip install python-docx

# 安装本插件
claude plugin add github:Kewanvk/research-with-receipts
```

### 首次使用

```bash
# 登录 NotebookLM（会打开 Chrome 浏览器）
nlm login
```

## 使用方式

在 Claude Code 中用自然语言触发：

```
"帮我从这些年报里提取数据"
"~/reports/ 里有些 PDF，帮我整理一下"
"验证一下这份文档里的信息"
"从这些代理声明中收集数据"
```

Skill 会引导你一步步完成整个流程。

