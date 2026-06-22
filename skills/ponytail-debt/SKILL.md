---
name: ponytail-debt
description: >
  收集代码库里所有 `ponytail:` 注释，整理成技术债台账，让 ponytail 留下的有意捷径和延期项被追踪，
  而不是腐烂成“以后就是永远不会”。当用户说“ponytail debt”、“/ponytail-debt”、
  “ponytail 推迟了什么”、“列出这些捷径”、“ponytail ledger” 或“我们标了哪些以后再做”时使用。
  一次性报告，不修改任何东西。
---

每个有意的 ponytail 捷径都用 `ponytail:` 注释标记，写明它的上限和升级路径。这个 skill 会把它们收集到一个台账里，避免延期项悄悄变成永久状态。

## 扫描

在仓库中 grep 注释标记，跳过 `node_modules`、`.git` 和构建输出：

`grep -rnE '(#|//) ?ponytail:' .`  (add other comment prefixes if your stack uses them)

每个命中项是一条台账记录。注释前缀可以避免把只是提到这个约定的普通文字收进台账。

## 输出

每个标记一行，按文件分组：

`<file>:<line>, <what was simplified>. ceiling: <the limit named>. upgrade: <the trigger to revisit>.`

约定格式是 `ponytail: <ceiling>, <upgrade path>`，所以直接从注释里提取上限和触发条件。也想给每行加 owner？使用 `git blame -L<line>,<line>`。

标出腐烂风险：任何没有写升级路径或触发条件的 `ponytail:` 注释，都加上 `no-trigger` 标签；这些最容易悄悄烂掉。

结尾写：`<N> markers, <M> with no trigger.` 没找到时：`No ponytail: debt. Clean ledger.`

## 边界

只读取和报告，不修改任何东西。若用户要求持久化，再把台账写入文件（例如 `PONYTAIL-DEBT.md`）。一次性执行。说 “stop ponytail-debt” 或 “normal mode” 可恢复。
