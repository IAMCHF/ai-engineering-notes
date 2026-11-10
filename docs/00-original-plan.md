# 原规划（留存）

> 来源：外部 Agent 于 2026-11-10 给出的规划，用户直接粘贴。
> 本文件**仅做标题层级规整，内容未做任何删改**，用于后续对照与追溯。
> 评估与修正见 [主规划](00-master-plan.md)。

---

## 项目规划

### 3 个项目

#### 1. Agent 平台（核心深度，3 周）

- LangGraph 实现状态图（intent/plan/tool-exec/reflect/replan/fallback）
- 5 个工具（含副作用），含失败重试 + 降级
- 流式输出（SSE：thinking/tool_call/tool_result/answer）
- 短期记忆(messages) + 长期记忆(向量库) + 摘要压缩
- 30 条评测用例，对比"单 prompt vs ReAct vs 自研图"成功率
- 模型降级（主模型挂了切备用）

#### 2. RAG 评测系统（2 周）

- 不部署 RAGFlow，自己写核心链路
- 对比实验矩阵：chunk(256/512/semantic) × embedding(2 个) × 检索(向量/BM25/混合) × rerank(有/无)
- 50 条中文 query + ground truth
- 指标：Recall@5 / MRR / P50 延迟
- 接入 Agent 项目作为检索工具

#### 3. MCP 建站（2 周，差异化王牌）

- FastMCP SDK + 5 个真实工具（代码分析类）
- API Key 鉴权 + 限流 + 审计日志
- stdio + SSE 双 transport
- 能被 Claude Desktop 直接接入

### 技术栈原则

- 不用 RuoYi、不用 full-fastapi-template
- FastAPI 裸写 + pydantic-settings + langgraph + httpx
- MCP 项目用 FastMCP SDK
- 每个项目 src 代码控制在 1500 行内
- 鉴权用 API Key（一行 Depends），不要 RBAC

### 不做的事

- 不做多租户（Skill 广场故事已覆盖，面试现场设计即可）
- 不做前端管理后台
- 不引入 LangChain 全家桶
- 不加数据库迁移 / Redis 消息队列等非核心组件

### 每个仓库必须有

- README（顶部：3 行说清做什么 + 数字结果）
- docs/decisions.md（至少 5 个 ADR）
- 架构图（mermaid）
- 评测结果表
- demo GIF 或 1 分钟录屏
- Makefile（make demo / make eval）

### 时间线（下班后，每天 2h × 5 天）

1. MCP 建站 2 周（先做，差异化最大）
2. Agent 平台 3 周（核心）
3. RAG 2 周（后期接入 Agent）
4. 博客 3 天（TRACE 评测体系 / MCP 实战）

总计约 7 周。

### 学习路径（按顺序）

1. **smolagents**（HuggingFace）— 最小 ReAct Agent，1500 行
2. **FastMCP** + **modelcontextprotocol/servers** — MCP 协议实现
3. **LangGraph examples** — Agent 状态图设计
4. **Pydantic AI** — 结构化输出可靠性
5. **Langfuse** — LLM 工程化标杆（评估 + trace）
6. **LiteLLM** — 模型路由 + 降级
7. **Letta** — 长记忆设计
8. **Haystack** — RAG pipeline 抽象
9. **RAGFlow**（看设计不看代码）— chunk 策略
10. **Dify**（只看 api/core/）— RAG + Agent 串联

**学习方法**：跑起来 → 找入口 → 打断点 → 画架构图 → 写 1000 字拆解笔记

### 背景说明（用户原话）

> 我准备更新简历找工作。但是我做的项目都在内网无法搬出来。所以准备在外网复现部分核心功能，同时学习一些确实的能力项。上面是其他 agent 给我的规划。我的目标是拿到 28k 的工作，我有一年半的经验。之后我会把内网的一些项目用文档的方式总结给你，你现在先把我这些整理分析一下。然后放到一个文档里面吧，方便后续更新。
