---
type: module-spec
version: 1.0
source: src/memory/
files_analyzed: 28
loc: 7065
last_updated: 2026-02-09
---

# Memory Spec

## 1. Intent (意图)
向量记忆搜索模块，为 Agent 提供长期记忆能力：
- 文件和 session transcript 的向量索引
- 多 embedding 提供商支持（Voyage、OpenAI、Gemini）
- 混合搜索（向量 + BM25 全文）
- SQLite + sqlite-vec 本地向量数据库
- 增量同步和文件变更监听

## 2. Interface (接口定义)
- `MemoryIndexManager` — 主管理器类，负责索引构建和搜索
- `getMemorySearchManager(cfg, agentId)` → `MemorySearchManager` — 获取搜索管理器
- `MemorySearchResult` — 搜索结果类型
- `MemoryEmbeddingProbeResult` — embedding 探测结果

## 3. Business Logic (业务逻辑)
- **索引构建**: 扫描 workspace 文件 → markdown chunking → batch embedding → 存入 SQLite
- **混合搜索**: 向量相似度搜索 + BM25 关键词搜索 → 合并排序
- **增量同步**: chokidar 监听文件变更 → 计算 hash 差异 → 仅更新变化的 chunks
- **Session 索引**: 自动索引 Agent 对话 transcript
- **Embedding 提供商**: Voyage（默认）、OpenAI、Gemini，支持 batch API

## 4. Data Structures (数据结构)
- `MemoryIndexMeta` — 索引元数据（model, provider, chunkTokens, vectorDims）
- `MemoryChunk` — 文本块（path, content, hash, embedding）
- `MemoryFileEntry` — 文件条目
- `MemorySource` — 记忆来源类型
- `MemorySyncProgressUpdate` — 同步进度

## 5. Constraints (约束)
- 依赖 SQLite（node:sqlite）和 sqlite-vec 扩展
- 向量维度由 embedding 模型决定
- 本地存储，不支持远程向量数据库

## 6. Edge Cases (边界条件)
- sqlite-vec 扩展不可用时的降级
- Embedding API 调用失败的重试
- 大文件的分块处理
- 空 workspace 的处理

## 7. Technical Debt (技术债务)
- 依赖 node:sqlite（Node.js 实验性 API）
- 多 provider batch 代码有重复

## 8. Dependencies (依赖)
- **内部**: agents, config, logging, sessions
- **外部**: chokidar, node:sqlite, sqlite-vec
