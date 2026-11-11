# 仓库规划与 P0 骨架

> 状态：2026-11-11 建立。对应主规划 [§6 排期](00-master-plan.md)。

---

## 1. 现状盘点（`C:\projects`）

| 目录 | 是什么 | 怎么用 |
|---|---|---|
| `ai-engineering-notes` | 本仓，纯文档 | 规划与复盘 |
| `sites-workbench` | **案例 A** 内网建站 MCP（161 提交 / 1.4 万行） | **只读参考**，不公开、不复刻代码（AGENTS 第 4 条） |
| `skillscan` | **案例 B** SkillScan TRACE（2,227 行） | 按 [待办清单](03-skillscan-tasks.md) 改造后开源 |
| `smolagents-study` | 上游 `huggingface/smolagents` 的 clone | **只读学习**（学习路径 #1），不提交改动、不公开 |
| `leetcode` | 空目录 | 可选：每日 2 题 |
| `trade-study` | 仅有 `agents.md`，无内容 | 暂不投入 |

## 2. 新建两个仓

### 2.1 `agent-platform`（主线，W1–W10）

**定位**：一内核三出口里的**内核 + HTTP/SSE 出口 + Eval CLI**；RAG 是它的一个模块，不单开仓库。

```
agent-platform/
├── src/agent_platform/
│   ├── config.py            # pydantic-settings，启动校验
│   ├── llm/gateway.py       # 主备切换 + 熔断 + 错误分类 + 计量
│   ├── obs/trace.py         # jsonl trace，token/成本/延迟
│   ├── api/app.py           # FastAPI 入口
│   ├── api/sse.py           # thinking/tool_call/tool_result/answer 四类事件
│   ├── graph/               # P1 起：intent/plan/exec/reflect/replan/fallback
│   ├── tools/               # P1 起：注册/超时/重试/幂等/故障注入
│   └── memory/              # P2 起：messages + 摘要 + 长期向量
├── evals/                   # 用例集 + 4 臂对比脚本 + 结果表
├── tests/
├── docs/decisions.md        # ADR
├── Makefile                 # demo / test / eval / fmt
├── .env.example
└── README.md                # 顶部 3 行 + 数字
```

**P0 范围（本周，W1）**

| # | 任务 | 验收 |
|---|---|---|
| P0-1 | 仓骨架 + `pyproject` + Makefile + `.env.example` + `.gitignore` | `make install` 通过 |
| P0-2 | `config.py`：pydantic-settings，缺 key 启动即报错 | 不配 key 启动失败且提示明确 |
| P0-3 | `llm/gateway.py`：主备 + 熔断 + 错误分类（可重试/不可重试）+ token 计量 | 单测：主模型 5xx → 自动切备用；4xx 不重试 |
| P0-4 | `obs/trace.py`：每次调用落 jsonl（步数/token/成本/延迟） | `make demo` 后 `traces/` 有文件 |
| P0-5 | `api/sse.py` + `api/app.py`：`/chat` 流式返回四类事件 | `curl -N` 能看到 4 种 event |
| P0-6 | `tests/` 最少 6 条 + CI | `make test` 全绿；PR 上 CI 通过 |
| P0-7 | `docs/decisions.md` 前 2 条 ADR（为什么自写网关、为什么 SSE 而非 WebSocket） | —— |

> **P0 不做**：图状态机、工具、记忆、评测。这些是 P1 起的事，别提前塞。

### 2.2 `mcp-replica`（P4，W11–W13）

**定位**：案例 A 的**机制复刻版**——只保留机制，用本地假后端重写，不搬任何原代码。

```
mcp-replica/
├── src/mcp_replica/
│   ├── server.py            # MCP 工具层（stdio + Streamable HTTP）
│   ├── plan.py              # 计划—确认—执行三阶段，摘要绑定
│   ├── confirm_ui.py        # 回环确认页
│   ├── executor.py          # 四态结果 + 未知不重试 + 有界等待
│   ├── backend/             # DeploymentBackend 协议 + LocalBackend
│   └── records.py           # 记录 / 锁 / 恢复
├── tests/
├── docs/decisions.md
├── Makefile
└── README.md
```

**P4 任务**：见 [02-mcp-open-source.md](02-mcp-open-source.md) §5（W11 骨架 / W12 机制 / W13 收口）。

> **W11 之前不要开工**，也**不要打开 `sites-workbench` 的 `.py` 文件**——只看[案例 A 归档](internal/01-内网建站MCP.md)的机制描述，避免思维搬运。

## 3. 三个仓的分工

| 仓 | 讲什么能力 | 公开 | 时间 |
|---|---|---|---|
| `agent-platform` | Agent 编排 + 评测方法 + 工程化 | ✅ | W1–W10 |
| `mcp-replica` | 有副作用的系统：确认、幂等、未知结果 | ✅ | W11–W13 |
| `skillscan` | 判定与可复现性：fail-closed、一致性 | ✅ | 用户自行安排 |
| `sites-workbench` | 真实约束下的全链路交付 | ❌ 只讲 | —— |

## 4. 跨仓约定

- 每个仓自己的 `AGENTS.md` 继承本仓约定（自动提交、不破坏性操作）。
- 每个仓 README 顶部 3 行：**做什么 + 关键数字 + 怎么跑**。
- Makefile 统一目标：`install / demo / test / eval / fmt / up`。
- 依赖全部 pin；协议类决策以官方 spec 为准（不复用旧笔记）。
- 不引入 LangChain 全家桶；不建数据库迁移 / Redis。

## 5. Gate 0（P0 完成标准）

- [ ] `make install && make demo` 在干净环境一次通过
- [ ] `curl -N` 能看到 thinking / tool_call / tool_result / answer 四类 SSE 事件
- [ ] 不配 `LLM_API_KEY` 启动即失败，报错明确指出缺哪个变量
- [ ] 主模型故障时自动切备用，日志与 trace 都留痕
- [ ] `make test` 全绿（≥ 6 条）
- [ ] 2 条 ADR 已写
