---
name: ponytail-gain
description: >
  用紧凑记分板展示 ponytail 的实测影响：更少代码、更低成本、更快速度，数据来自 benchmark 中位数。
  一次性展示，不是持久模式，也不是当前仓库的专属数字。触发：/ponytail-gain、
  “ponytail gain”、“ponytail 节省了什么”、“展示 ponytail 影响”、“ponytail scoreboard”。
---

# Ponytail Gain

调用时展示这个记分板。一次性：不要改变模式，不要写 flag 文件，不要持久化任何东西。

这些数字是已发布 benchmark 的中位数（5 个日常任务：email validator、debounce、CSV sum、countdown timer、rate limiter；3 个模型：Haiku、Sonnet、Opus）。它们是实测数据，不是从当前仓库计算出来的。来源：`benchmarks/` 和 README。

## 记分板

渲染纯 ASCII 条形图。条形长度表示测量范围；标签给出精确数字：

```
  ponytail gain                     benchmark median · 5 tasks · 3 models

  Lines of code   no-skill  ████████████████████  100%
                  ponytail  ██▌·················    6–20%   ▼ 80–94%
  Cost            no-skill  ████████████████████  100%
                  ponytail  █████▌··············   23–53%  ▼ 47–77%
  Speed           ponytail  ▸ 3–6× faster

  This repo:  /ponytail-debt  (shortcuts you deferred)
              /ponytail-audit (what's still cuttable)
```

## 诚实边界

这些是 benchmark 中位数，不是当前仓库的数据。永远不要输出单仓库节省数字（例如“这里节省了 X 行/token”）：未构建版本从来没有被写出来，所以在真实仓库里没有可相减的真实 baseline。唯一真实的单仓库数字来自 `/ponytail-debt`（可计数台账），这个卡片应该指向那里，而不是编造数字。

## 边界

一次性展示。不编辑任何东西，不改变模式。
说 “stop ponytail” 或 “normal mode” 可恢复。
