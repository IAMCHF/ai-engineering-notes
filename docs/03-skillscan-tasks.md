# SkillScan 待办清单

> 可直接复制到 `C:\projects\skillscan\TASKS.md` 当 issue 列表用。
> 背景与判断依据见 [03-skillscan.md](03-skillscan.md)。**按 A → B → C → D → E 的顺序做**，A 不做完不要公开。
> 每项给了「位置 / 改什么 / 验收」，做完打勾即可。

---

## A · 必须先修（不做完不要公开）

### [ ] A1 重建仓库，清掉 .env 历史

- **位置**：`.env.dev` / `.env.prod` / `.env.test`，引入于 `98ce8ab`
- **改什么**：
  1. 新建空目录 → `git init`（**不要**用 `filter-branch` 修老历史）
  2. 只拷贝清理后的文件，**不拷** `.env.dev/.prod/.test`
  3. `.env.example` 里所有 key 的值改回占位符（如 `sk-your-key-here`）
  4. 首次提交
- **验收**：`git log --all --name-only | Select-String '\.env'` 只应出现 `.env.example`；在历史里 grep 不到任何 `sk-` 开头的真实 key
- **工时**：0.5h

### [ ] A2 加第三态：把「审核未完成」和「不安全」分开

- **位置**：`app/models/schemas.py`（verdict 字段）、`app/engine/scanner.py::_parse_llm_result`（`else` 分支）
- **改什么**：
  - `verdict` 增加取值 `待复核`（或 `audit_incomplete`），与 `不安全` 区分
  - `security_level` 同步区分：`安全` / `不安全` / `未知`
  - LLM 失败走 `待复核`，理由写"审核未完成，请重试或转人工"
- **连带要改**（改一处会漏，注意）：
  - `app/templates/report.html` 的结论样式（新增一个色）
  - `README.md` 的 verdict 说明
  - `app/report_renderer.py` 若有 verdict 映射
- **验收**：断掉 LLM_API_KEY 跑一次 `/scan`，返回 `待复核` 而不是 `淘汰`
- **工时**：2h

### [ ] A3 校验 LLM 返回结构，缺项不许静默降分

- **位置**：`app/engine/scanner.py::_parse_llm_result`
- **改什么**：
  - 定义 Pydantic 模型校验 `trace_scores` 的 5 个维度、每维 4 个子指标
  - 缺维度 → 标记"未评"，**不参与总分平均**（现在会当 0 分拉低总分）
  - 子指标数量不足 → 该维度标记不完整
  - 解析失败 → 返回错误，不返回"看起来正常的低分"
- **验收**：构造一个只返 3 个维度的 mock 响应，断言总分不等于 5 维平均且结果含"未评"标记
- **工时**：3h

### [ ] A4 建 `tests/` 目录，让 `make test` 真的能跑

- **位置**：新建 `tests/`；`Makefile` 的 `test:` 目标
- **改什么**：至少覆盖 12 条（见下）
- **验收**：`make test` 全绿
- **工时**：见 B2

---

## B · 最小测试集 + 评分一致性（这组直接产出简历数字）

### [ ] B1 补 12 条单测

- **位置**：`tests/test_*.py`
- **必须覆盖**：

| # | 用例 | 测什么 |
|---|---|---|
| 1 | 正常 zip → 完整结果 | 主流程 |
| 2 | SKILL.md 文件名小写 | 格式检测拦截 |
| 3 | frontmatter 未闭合 | 格式检测拦截 |
| 4 | 缺 name 字段 | 格式修复能补 |
| 5 | zip 含二进制/超大文件 | 跳过逻辑 |
| 6 | LLM 返回非法 JSON | `_parse_llm_result` 不崩 |
| 7 | LLM 缺一个维度 | A3 的"未评"生效 |
| 8 | LLM 超时 | 走 `待复核`（A2） |
| 9 | 空 zip / 非 zip | 错误路径 |
| 10 | 关键词分类命中 | 分类降级路径 |
| 11 | 报告渲染（Jinja2） | 模板不报错 |
| 12 | `_build_security_labs` | 不伪造 pass |

- **工时**：6h（用 `httpx.MockTransport` 假 LLM，不要真调）

### [ ] B2 golden set：证明分数是可复现的

- **位置**：新建 `eval/`（脚本 + 样本 + 结果表）
- **改什么**：
  1. 固定 **10–15 个样本**（安全 / 可疑 / 恶意各 3–5 个）
  2. 每样本跑 **3 次**，用 **2 个模型**（主 + 备）
  3. 记录：各维度分数、总分、verdict、`tokens_used`、`latency_ms`
  4. 产出 `eval/RESULTS.md`：各维度**标准差**、verdict **一致率**、单次成本、P50/P95
  5. 尝试 `temperature=0` 对比 `0.3`，看方差是否收敛
- **验收**：`eval/RESULTS.md` 里有一张能直接贴进简历的表，且写清"改了什么之后方差从多少降到多少"
- **工时**：8h（本职是标注 10–15 个样本，别省）

### [ ] B3 顺手修掉分类的额外 LLM 往返

- **位置**：`app/engine/classifier.py::classify_skill`、`app/engine/prompts.py::SYSTEM_PROMPT_CLASSIFY`
- **改什么**（方案 A，推荐）：
  - `classify_skill` 改为同步关键词分类，删掉那次 `client.chat`
  - 删 `SYSTEM_PROMPT_CLASSIFY`；`scanner.py` 调用处去掉 `await`
- **备选**（方案 B）：关键词置信度 < 阈值时，才补调一次 LLM
- **验收**：单次扫描的 LLM 调用次数从 2 降到 1；`scan_duration_ms` 下降；分类结果两次运行完全一致
- **工时**：A 方案 4h / B 方案 8h

---

## C · 工程化（按需做，每项都是面试可讲的点）

### [ ] C1 给 LLM 调用加重试

- **位置**：`app/engine/llm_client.py::LLMClient.chat`
- **改什么**：仅对**超时 / 5xx / 429** 做 1 次指数退避重试（0.5s → 1s）；4xx 不重试
- **验收**：mock 返回 500 一次后成功，结果正常且日志有重试记录
- **工时**：2h

### [ ] C2 批量扫描加并发

- **位置**：新增 `POST /scan-batch`，或改造现有编排
- **改什么**：`asyncio.Semaphore`（建议 4–8），复用同一个 `httpx.AsyncClient`（现在是每次调用新建，浪费连接）
- **验收**：扫 10 个技能的耗时 < 串行的 1/3；无并发打爆下游
- **工时**：3h

### [ ] C3 加结果缓存

- **位置**：`app/engine/scanner.py` 入口
- **改什么**：按 zip 内容 SHA-256 + 模型名 缓存结果（本地文件或内存 LRU 均可），命中直接返回并标记 `cached=true`
- **验收**：同一 zip 第二次扫描 `scan_duration_ms` 接近 0 且不产生 token 消耗
- **工时**：3h

### [ ] C4 成本计量

- **位置**：`scanner.py` 组装结果处 + 日志
- **改什么**：汇总 `tokens_used`，按单价算成本，写进结果对象与日志；批量扫描输出总成本
- **验收**：一次扫描的日志里能看到 token 数与估算费用
- **工时**：1.5h

### [ ] C5 接口防护

- **位置**：`app/routers/scan.py`
- **改什么**：API Key（`Depends` 一行即可）+ 简单速率限制 + 上传体积上限（如 10MB）
- **验收**：不带 key 返回 401；超限返回 413/429
- **工时**：3h

### [ ] C6 加 CI

- **位置**：`.github/workflows/ci.yml`
- **改什么**：安装依赖 → `make test`；不跑真实 LLM（用 mock）
- **验收**：PR 上能看到 CI 通过
- **工时**：1.5h

### [ ] C7 确认 zip 防护

- **位置**：`app/utils/zip_reader.py`
- **改什么**：逐项确认 **压缩比上限（zip bomb）/ 路径穿越（`..`）/ 符号链接 / 单文件大小 / 总解压体积**
- **验收**：构造一个 1000:1 压缩比的 zip，能被拒绝
- **工时**：2h

---

## D · 结构（有空再做）

### [ ] D1 拆 `skill_fixer.py`

- **位置**：`app/engine/skill_fixer.py`（478 行）
- **改什么**：拆成 `detect.py`（检测）/ `fix.py`（修复文本）/ `rezip.py`（重建 zip）
- **验收**：拆分后单测仍全绿
- **工时**：3h

### [ ] D2 报告模板瘦身

- **位置**：`app/templates/report.html`（718 行）
- **改什么**：把 if/for 之外的判断逻辑下沉到 `app/report_renderer.py`，模板只做展示
- **验收**：模板行数下降 ≥ 30%，渲染输出不变
- **工时**：3h

### [ ] D3 大文件截断策略

- **位置**：`app/engine/scanner.py`（Step 3 构建消息处）
- **改什么**：按"SKILL.md 优先 → 脚本文件 → 文档"排序，超 token 预算截断，并在结果里标注 `truncated=true`
- **验收**：给一个超大技能不会因超 context 直接失败
- **工时**：3h

---

## E · 公开前（最后一遍）

### [ ] E1 确认 TRACE 体系能否公开引用

- 若 TRACE 属于平台/公司资产：**改用自己的五维体系**（结构可同、命名与定义要换），并删除"对齐某平台官方标准"的表述

### [ ] E2 扫一遍公司资产

- grep 内部域名、平台名、账号、SkillHub 相关字段、内网接口路径
- 检查 `sample-report.html` 是否为假数据

### [ ] E3 补齐开源门面

- `README` 顶部 3 行：做什么 + 关键数字（用 B2 表里的数）+ 怎么跑
- `LICENSE`（已写 MIT，确认无冲突）
- `docs/decisions.md`：3–5 条 ADR（建议写：为什么 fail-closed、为什么静态分析不执行、为什么砍 LLM 分类、为什么加第三态）
- 1 分钟录屏：上传 zip → 报告 → 断掉 key 看 `待复核`

---

## 最短路径（只想尽快有个能公开的版本）

```
A1 → A2 → A3 → A4/B1 → B3 → E1 → E2 → E3
```

约 **18–20h**。做完这组，就有：**零泄漏的仓库 + 12 条测试 + 一个真正的语义修正（第三态）+ 少一次 LLM 调用**。

B2（评测一致性）单独 8h，但它产出的是**简历上唯一的硬数字**，建议不要跳过。
