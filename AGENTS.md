# AGENTS.md

## 核心约束

1. 本仓库仅用于纯文档的项目复盘与技术学习，不存放被分析的第三方代码，不执行构建。
2. 每次 git 提交要有意义且 commit 清晰易读；每次修改完成后自动提交。删除、合并、同步等破坏性操作由用户手动执行，或每次请求用户单次授权。
3. 每次用户提出需求后，先全面分析规划；存在不清楚的地方先找用户确认，理清思路后给出完整的总体规划，待用户最终确认后再开始执行。

## 仓库结构

```
projects/    有交付物、有量化结果的项目复盘，按 <YYYY>-q<N>-<slug>/ 分目录
analyses/    针对具体仓库/论文的深度分析，扁平，文件名即索引
notes/       不绑定具体实现的原理与概念，扁平
snippets/    可复用代码片段，按语言分目录
templates/   三类文档骨架
```

顶层不按技术栈划分。

## 分类判断

写新文档前先问：**这是证据还是笔记？**

- 有交付物、能量化、能写进简历 → `projects/`
- 针对具体仓库或论文、有 commit 可追溯 → `analyses/`
- 纯概念理解、不绑定具体实现 → `notes/`
- 可复用代码，带出处与 license → `snippets/`

技术域用 frontmatter 的 `tags` 表达，不建目录。

## 写作规则

1. 结论先行：每篇开头写一句话结论。
2. 单一事实来源：跨文档只写相对链接，不复制内容。
3. 外链必须锁 commit：`https://github.com/<org>/<repo>/blob/<40位sha>/<path>#L起-L止`，禁止使用 `main` 分支链接。
4. `snippets/` 标注 license，GPL/AGPL 代码不整段抄录。
5. 结论被代码变更推翻时，把 `status` 改为 `stale` 并注明新 commit，不要删文档。

## frontmatter（每篇必填）

`type`（project/analysis/note/snippet）、`title`、`date`、`updated`、`tags`、`status`（draft/done/stale），`analyses/` 与 `snippets/` 另需 `source`。

## git 约定

- 一个 commit 只做一件事；信息说明"改了什么、为什么"，不复述文件名。
- 提交前确认没有把 `artifacts/` 里的大文件（>10MB）或敏感信息带进去。
