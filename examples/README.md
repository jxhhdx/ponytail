# 示例

这些是真实模型输出，来自 benchmark 运行：同一个任务由同一个模型分别在未启用 skill（`## 未启用 Ponytail`）和启用 ponytail（`## 启用 Ponytail`）时回答，方便并排比较。模型：Claude Haiku 4.5，temperature 1，来源 `benchmarks/output.json`。

这些不是手写样例。你可以自己复现：
`npx promptfoo@latest eval -c benchmarks/promptfooconfig.yaml`。方法、全部三个模型和 10 次运行中位数见：[../benchmarks/](../benchmarks/)。

| 示例 | 未启用（LOC） | 启用后（LOC） |
|---|--:|--:|
| [邮箱校验](email-validation.md) | 75 | 3 |
| [Debounce](debounce.md) | 116 | 10 |
| [CSV 求和](csv-sum.md) | 20 | 3 |
| [倒计时组件](react-countdown.md) | 267 | 9 |
| [速率限制](rate-limit.md) | 128 | 10 |
