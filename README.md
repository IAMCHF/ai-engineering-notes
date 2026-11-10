# 项目复盘与技术学习库

纯文档仓库：记录**我交付过什么**、**我理解过什么**、**我掌握了什么**。不存放被分析的第三方代码，只存结论与证据。

## 三秒判断：这篇该放哪

新写一篇文档前，问一句：**这是"证据"还是"笔记"？**

| 文档性质 | 去向 |
|---|---|
| 有交付物、有量化结果、能写进简历 | `projects/` |
| 针对某个具体仓库/论文，有 commit 可追溯 | `analyses/` |
| 纯概念理解，不绑定具体实现 | `notes/` |
| 可复用代码片段，带出处与 license | `snippets/` |

顶层**不按技术栈分**。技术栈两年后会过时，证据类型不会。"按技术域检索"交给 frontmatter 的 `tags`，比目录分层灵活。

## 目录

```
projects/                 # 按 <YYYY>-q<N>-<slug>/ 分目录，时间序是回看成长轨迹的主轴
  └── 2026-q4-rag-service/
      ├── README.md       # 结论先行；跨类引用写成相对链接
      └── artifacts/      # 截图 / 数据 / 度量，可选
analyses/                 # 扁平，文件名即索引
  └── smolagents.md
notes/                    # 扁平
  └── kv-cache.md
snippets/                 # 按语言
  └── py/retry-with-backoff.md
templates/                # 三类文档骨架，复制后改 frontmatter 即可开写
```

## 命名规范

统一小写、kebab-case。

| 位置 | 规则 | 示例 |
|---|---|---|
| `projects/` | `<YYYY>-q<N>-<slug>/` | `2026-q4-rag-service/` |
| `analyses/` | `<repo>-<topic>.md`，单仓库单文档可省 topic | `smolagents.md`、`ragflow-hybrid-search.md` |
| `notes/` | `<concept>.md` | `kv-cache.md` |
| `snippets/` | `<lang>/<name>.md` | `py/retry-with-backoff.md` |

只有 `projects/` 带时间前缀，因为只有它需要按时间排序。

## frontmatter

每篇必填：

```yaml
---
type: analysis            # project | analysis | note | snippet
title: smolagents 架构拆解
date: 2026-11-10          # 创建日
updated: 2026-11-10
tags: [agent, tool-use]   # 技术域只打这里，不建目录
status: draft             # draft | done | stale
source: https://github.com/huggingface/smolagents/blob/<40位sha>/src/agents.py#L1-L40
---
```

`status: stale` 表示"结论所依据的代码已变、待复核"。复盘仓库里标记比删除更有价值。

## 外链仓库规则

本仓库**不克隆也不存放**任何外部代码，所有引用靠 permalink 追溯：

1. 必须是锁定 commit 的永久链接：`https://github.com/<org>/<repo>/blob/<40位sha>/<path>#L起-L止`
2. 禁止使用 `main` / `master` 分支链接，指向会漂移，半年后行号全错
3. 私有仓库拿不到 permalink 时，写 `commit: <完整 SHA>` + 拉取日期
4. `snippets/` 额外标注 `license`；GPL/AGPL 代码不整段抄录

## 链接约定

三类文档之间用相对链接串联，**不复制内容**：

- 在项目复盘中写"检索层参考了 [混合检索](analyses/ragflow-hybrid-search.md)"，而不是把那篇的结论再抄一遍
- 把可迁移的原理抽到 `notes/`，分析文档里只留链接
- 单一事实来源：同一结论只在一处详述，其余引用

## 检索

```bash
rg -l 'tags:.*\brag\b'          # 按标签找
rg '混合检索' --type md          # 全文找
```

数量到百篇以上再考虑生成索引，现阶段按目录和文件名找足够。
