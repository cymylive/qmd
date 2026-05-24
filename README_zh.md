<p align="center">
  <a href="README.md">English</a> | <b>中文</b>
</p>

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/tobi/qmd/main/assets/qmd-architecture.png">
    <img alt="QMD Architecture" src="https://raw.githubusercontent.com/tobi/qmd/main/assets/qmd-architecture.png" width="600">
  </picture>
</p>

# QMD — 本地文档搜索引擎

**Query Markup Documents**

QMD 是一款**本地混合搜索引擎**，专为 Markdown 文件、文档库、知识库、会议笔记等设计。所有操作均在本地运行，无需联网，无需云端服务。

> Fork 自 [tobi/qmd](https://github.com/tobi/qmd)，原作者：Tobi Lutke（Shopify 创始人）

[![CI](https://github.com/tobi/qmd/actions/workflows/ci.yml/badge.svg)](https://github.com/tobi/qmd/actions/workflows/ci.yml)
[![npm version](https://img.shields.io/npm/v/@tobilu/qmd)](https://www.npmjs.com/package/@tobilu/qmd)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 目录

- [简介](#简介)
- [核心特性](#核心特性)
- [安装](#安装)
- [快速开始](#快速开始)
- [CLI 命令参考](#cli-命令参考)
- [查询语法](#查询语法)
- [搜索原理](#搜索原理)
  - [三种检索后端](#三种检索后端)
  - [搜索管道](#搜索管道)
  - [评分融合策略](#评分融合策略)
- [配置](#配置)
- [MCP 服务器](#mcp-服务器)
- [SDK / API 使用](#sdk--api-使用)
- [输出格式](#输出格式)
- [模型微调](#模型微调)
- [常见问题](#常见问题)

---

## 简介

QMD 是一个**终端命令行本地搜索引擎**，让你可以用自然语言搜索本地 Markdown 文件。它结合了三种搜索技术：

- **BM25（全文检索）** — 关键词精确匹配，瞬间返回
- **向量语义搜索** — 理解含义而非字面，找到概念相关的内容
- **LLM 重排序** — 交叉编码器精排，获得最佳结果

所有模型都在本地运行，自动从 HuggingFace 下载，无需 API Key，无需联网。

---

## 核心特性

- **三合一搜索** — BM25 + 向量 + LLM 重排序，兼顾速度和精度
- **完全本地** — 所有模型在本地运行，数据不出设备
- **智能分块** — 文档自动分块（~900 tokens，15% 重叠），支持 AST 感知分块（代码文件）
- **查询扩展** — LLM 自动扩展查询词，生成多种搜索变体
- **MCP 服务器** — 支持 AI Agent 通过 MCP 协议搜索本地文档
- **集合管理** — 按项目/主题组织文档集合
- **多格式输出** — CLI 彩色输出、JSON、CSV、Markdown、XML
- **Nix Flake** — 可复现构建

---

## 安装

### 前置要求

- **Node.js** >= 22 或 **Bun** >= 1.0

### 通过 npm 全局安装

```bash
npm install -g @tobilu/qmd
```

### 通过 npx 直接使用

```bash
npx @tobilu/qmd --help
```

### Nix Flake

```bash
nix run github:tobi/qmd
```

---

## 快速开始

### 1. 添加文档集合

```bash
# 添加一个 Obsidian 笔记目录
qmd collection add /Users/me/obsidian --name notes

# 添加项目文档
qmd collection add /Users/me/my-project/docs --name my-project
```

### 2. 生成向量索引

```bash
qmd embed
```

首次运行会自动下载嵌入模型（~300MB）。

### 3. 搜索

```bash
# 混合搜索（推荐）
qmd query "如何配置数据库连接"

# 关键词搜索
qmd search "数据库连接池"

# 语义搜索
qmd vsearch "数据库性能优化方法"
```

### 4. 查看索引状态

```bash
qmd status
```

---

## CLI 命令参考

### 搜索命令

| 命令 | 说明 |
|------|------|
| `qmd query <query>` | **混合搜索** — BM25 + 向量 + 查询扩展 + LLM 重排序，质量最高 |
| `qmd search <query>` | **关键词搜索** — 仅 BM25 FTS5 全文检索，速度最快 |
| `qmd vsearch <query>` | **向量搜索** — 仅语义相似度搜索 |

### 文档检索

| 命令 | 说明 |
|------|------|
| `qmd get <path>[:line]` | 获取单篇文档（按路径或 docid，例如 `#abc123`） |
| `qmd multi-get <pattern>` | 批量获取（支持 glob 或逗号分隔） |

### 集合管理

| 命令 | 说明 |
|------|------|
| `qmd collection add <path> --name <name>` | 添加/索引文档集合 |
| `qmd collection list` | 列出所有集合 |
| `qmd collection remove <name>` | 移除集合 |
| `qmd collection rename <old> <new>` | 重命名集合 |
| `qmd ls [collection[/path]]` | 列出集合中的文件 |

### 上下文管理

| 命令 | 说明 |
|------|------|
| `qmd context add [path] "text"` | 添加上下文描述 |
| `qmd context list` | 列出所有上下文 |
| `qmd context check` | 检查缺少上下文的集合 |
| `qmd context rm <path>` | 移除上下文 |

### 维护命令

| 命令 | 说明 |
|------|------|
| `qmd embed` | 生成向量嵌入（首次必须运行） |
| `qmd update [--pull]` | 重新索引所有集合 |
| `qmd cleanup` | 清理孤立文件、清缓存、VACUUM |
| `qmd doctor` | 系统健康检查 |
| `qmd status` | 索引状态、集合信息、模型信息 |

### MCP 服务器

| 命令 | 说明 |
|------|------|
| `qmd mcp` | 启动 MCP 服务器（stdio 模式） |
| `qmd mcp --http` | 启动 HTTP 模式 MCP 服务器 |
| `qmd mcp --http --daemon` | 后台 HTTP 守护进程 |
| `qmd mcp stop` | 停止后台守护进程 |

### 基准测试 & 技能

| 命令 | 说明 |
|------|------|
| `qmd bench <fixture>` | 运行基准测试查询 |
| `qmd skill show` | 显示 AI Agent 使用的 QMD 技能 |
| `qmd skill install` | 安装技能到 `.agents/skills/qmd` |

---

## 查询语法

QMD 支持结构化查询文档，每行以类型前缀开头：

| 类型前缀 | 说明 | 示例 |
|---------|------|------|
| `lex:` | BM25 关键词搜索（支持引号精确匹配和 `-` 排除） | `lex: 数据库连接池 -MySQL` |
| `vec:` | 语义向量搜索（自然语言问题） | `vec: 什么是依赖注入？` |
| `hyde:` | 假设性文档嵌入（用想象的理想文档做向量匹配） | `hyde: 本文解释了数据库索引的原理...` |
| `intent:` | 领域消歧上下文（可选） | `intent: 这是一个 Node.js 后端项目` |
| `expand:` | 让 LLM 自动扩展查询 | `expand: 数据库连接池` |

### 示例

```bash
# 结构化查询
qmd query "lex: 连接池配置
vec: 如何优化数据库连接
intent: Node.js 后端项目"

# 使用引号精确匹配
qmd query 'lex: "connection pool"'

# 排除关键词
qmd query "lex: 缓存 -Redis"
```

---

## 搜索原理

### 三种检索后端

| 后端 | 方法 | 速度 | 适用场景 |
|------|------|------|---------|
| **BM25 (FTS5)** | SQLite FTS5 关键词匹配 | 瞬间 | 精确术语、名称、代码符号 |
| **向量搜索** | sqlite-vec 语义相似度 | 快（嵌入后） | 概念、含义搜索 |
| **重排序** | 交叉编码器 LLM（Qwen3-Reranker） | 慢（LLM 推理） | 最佳质量，位置感知混合 |

### 搜索管道

```
用户查询
  ├─ BM25 探测 ── 信号足够强？ ── 跳过扩展
  ├─ LLM 查询扩展 ── 生成 lex/vec/hyde 变体
  ├─ 并行搜索：每个变体同时跑 FTS + 向量
  ├─ RRF 融合（k=60，原始查询 2x 权重，排名加分）
  ├─ 取 Top 30 候选 ── 分块文档
  ├─ LLM 重排序（按块）
  └─ 位置感知混合 ── 最终结果
```

### 评分融合策略

**RRF 融合公式：**
```
score = sum(1 / (k + rank + 1))
```
- `k = 60`
- Top 排名加分：第 1 名 +0.05，第 2~3 名 +0.02
- 原始查询列表权重为 2x

**位置感知混合（检索分数 vs 重排序分数）：**

| RRF 排名 | 检索占比 | 重排序占比 |
|----------|---------|-----------|
| 1~3 | 75% | 25% |
| 4~10 | 60% | 40% |
| 11+ | 40% | 60% |

---

## 配置

### 配置文件位置

QMD 的 YAML 配置文件位于：

- **全局配置：** `~/.config/qmd/index.yml`
- **项目配置：** `.qmd/index.yaml`（项目目录下）

### 示例配置

```yaml
# example-index.yml
collections:
  - name: my-docs
    path: /path/to/docs
    glob: "**/*.md"           # 文件匹配模式
    ignore: ["node_modules"]  # 忽略目录
```

---

## MCP 服务器

QMD 提供 MCP（Model Context Protocol）服务器，让 AI Agent（如 Claude Code）可以直接搜索你的本地文档。

### 启动

```bash
# stdio 模式（默认，供 Agent 使用）
qmd mcp

# HTTP 模式（供其他程序调用）
qmd mcp --http

# 后台守护进程
qmd mcp --http --daemon
```

HTTP 模式下，模型会保持在显存中，5 分钟无请求后自动释放。

### 暴露的工具

MCP 服务器提供 4 个工具：

1. **`query`** — 结构化搜索（支持 lex/vec/hyde + intent）
2. **`get`** — 文档检索（按路径或 docid，含模糊建议）
3. **`multi_get`** — 批量文档检索
4. **`status`** — 索引健康状态

### REST API

HTTP 模式下提供 REST 端点：

```
POST http://localhost:8181/query
Content-Type: application/json

{ "query": "搜索内容" }
```

---

## SDK / API 使用

QMD 也可以作为 Node.js 库使用：

```bash
npm install @tobilu/qmd
```

```typescript
import { createStore, type QMDStore } from '@tobilu/qmd'

const store = await createStore({
  dbPath: './index.sqlite',
  config: { /* YAML 配置 */ }
})

// 搜索
const results = await store.search({
  query: "搜索关键词"
})

// 关闭
await store.close()
```

---

## 输出格式

所有搜索命令均支持多种输出格式：

| 参数 | 格式 | 说明 |
|------|------|------|
| （默认） | CLI | 终端彩色输出，带语法高亮 |
| `--json` | JSON | 结构化 JSON 输出 |
| `--csv` | CSV | 表格形式 |
| `--md` | Markdown | Markdown 格式 |
| `--xml` | XML | XML 格式 |
| `--files` | 文件列表 | 仅输出文件路径 |

---

## 模型

QMD 自动从 HuggingFace 下载以下模型（缓存到 `~/.cache/qmd/models/`）：

| 模型 | 用途 | 大小 | 来源 |
|------|------|------|------|
| EmbeddingGemma 300M (Q8_0) | 向量嵌入 | ~300 MB | `hf:ggml-org/embeddinggemma-300M-GGUF` |
| Qwen3-Reranker 0.6B (Q8_0) | 交叉编码器重排序 | ~640 MB | `hf:ggml-org/Qwen3-Reranker-0.6B-Q8_0-GGUF` |
| QMD Query Expansion 1.7B (Q4_K_M) | 查询扩展（微调版） | ~1.1 GB | `hf:tobil/qmd-query-expansion-1.7B-gguf` |

首次运行 `qmd embed` 或 `qmd query` 时自动下载。

嵌入模型可通过环境变量 `QMD_EMBED_MODEL` 覆盖。

---

## 模型微调

`finetune/` 目录包含完整的查询扩展模型微调管线：

- **SFT**（监督微调）：基于 Qwen3-1.7B，LoRA rank 16，约 2290 条标注数据
- **奖励函数**：5 维度评分（格式 30pt、多样性 30pt、HyDE 20pt、质量 20pt、实体保留 20pt ~ -65pt）
- **转换**：训练后转换为 GGUF 格式本地部署
- **评估**：平均得分 92.0%

详见 [finetune/README.md](finetune/README.md)。

---

## 常见问题

**Q: 首次搜索很慢？**

首次运行需要下载模型（共约 2GB），取决于网络速度。后续搜索模型已缓存。

**Q: 如何只搜索关键词不要语义搜索？**

使用 `qmd search` 命令，仅 BM25 全文检索，无需模型。

**Q: 如何让 AI Agent 使用 QMD？**

启动 MCP 服务器后，在 Agent 配置中添加 MCP 工具即可。也可使用 `qmd skill install` 安装技能。

**Q: 数据存储在哪里？**

- SQLite 索引：`~/.cache/qmd/index.sqlite`
- 模型缓存：`~/.cache/qmd/models/`
- 配置文件：`~/.config/qmd/index.yml`

---

## 许可

[MIT](LICENSE)

---

<div align="center">

**QMD** — 让本地文档也能被智能搜索

</div>
