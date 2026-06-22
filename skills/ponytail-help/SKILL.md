---
name: ponytail-help
description: >
  ponytail 所有模式、skills 和命令的快速参考卡片。一次性展示，不是持久模式。
  触发：/ponytail-help、“ponytail help”、“ponytail 有哪些命令”、“怎么使用 ponytail”。
---

# Ponytail Help

调用时展示这张参考卡片。一次性，不要改变模式，不要写 flag 文件，不要持久化任何东西。

## 级别

| 级别 | 触发 | 变化 |
|-------|---------|-------------|
| **Lite** | `/ponytail lite` | 构建用户要求的东西，并用一句话指出更懒的替代方案。 |
| **Full** | `/ponytail` | 强制执行阶梯：YAGNI → 标准库 → 原生能力 → 一行 → 最小实现。默认。 |
| **Ultra** | `/ponytail ultra` | YAGNI 极端模式。删除先于新增。构建前先挑战需求。 |

级别会持续到被改变或会话结束。

## Skills

| Skill | 触发 | 作用 |
|-------|---------|--------------|
| **ponytail** | `/ponytail` | 懒模式本体。采用能工作的最简单方案。 |
| **ponytail-review** | `/ponytail-review` | 过度工程 review：`L42: yagni: factory, one product. Inline.` |
| **ponytail-gain** | `/ponytail-gain` | 实测影响记分板：更少代码、更低成本、更快速度。 |
| **ponytail-help** | `/ponytail-help` | 当前卡片。 |

Codex 使用 `@ponytail`、`@ponytail-review` 和 `@ponytail-help`；Claude Code 和 OpenCode 使用上面的斜杠命令形式（OpenCode 提供 `/ponytail` 和 `/ponytail-review`）。

## 停用

说 “stop ponytail” 或 “normal mode”。随时用 `/ponytail` 恢复。`/ponytail off` 也可用。

## 配置默认模式

默认模式 = `full`，每个会话自动激活。修改方式：

**环境变量**（最高优先级）：
```bash
export PONYTAIL_DEFAULT_MODE=ultra
```

**配置文件**（`~/.config/ponytail/config.json`，Windows：`%APPDATA%\ponytail\config.json`）：
```json
{ "defaultMode": "lite" }
```

设置为 `"off"` 可禁用会话开始时的自动激活，需要时再用 `/ponytail` 手动激活。

解析优先级：环境变量 > 配置文件 > `full`。

## 更新

启用一次自动更新：打开 `/plugin`，进入 Marketplaces，选择 ponytail，启用自动更新。之后 Claude Code 会在启动时拉取新版本（提示时运行 `/reload-plugins`）。手动刷新：先运行 `/plugin marketplace update ponytail`，再运行 `/reload-plugins`。

如果 `/plugin` 无法识别，说明你的 Claude Code 版本太旧。更新它（`npm install -g @anthropic-ai/claude-code@latest`，或 `brew upgrade claude-code`）并重启。其他 host 使用各自的更新流程。

## 更多

完整文档和示例：https://github.com/DietrichGebert/ponytail
