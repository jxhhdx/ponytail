<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.png">
    <img src="assets/logo.png" width="220" alt="Ponytail，资深工程师极简模式">
  </picture>
</p>

<h1 align="center">Ponytail</h1>

<p align="center">
  <em>他什么都不说。他写了一行。它能跑。</em>
</p>

<p align="center">
  <img src="https://img.shields.io/github/stars/DietrichGebert/ponytail?style=flat-square&color=111111&label=stars" alt="Stars">
  <img src="https://img.shields.io/github/v/release/DietrichGebert/ponytail?style=flat-square&color=111111&label=release" alt="Release">
  <img src="https://img.shields.io/badge/works%20with-14%20agents-111111?style=flat-square" alt="Works with 14 agents">
  <img src="https://img.shields.io/badge/license-MIT-111111?style=flat-square" alt="MIT license">
</p>

<p align="center">
  <strong>代码少约 54%（最高 94%）&middot; 成本低约 20% &middot; 速度快约 27% &middot; 安全性 100%</strong><br>
  <sub>数据来自真实 Claude Code 会话：同一个 agent 编辑一个真实开源项目（FastAPI + React），对比未启用 skill 的情况。约 54% 是 12 个功能任务的平均值（Haiku 4.5，n=4）；当 agent 明显过度构建时可达 94%（比如日期选择器），当代码已经足够精简时接近 0。ponytail 保留所有安全护栏，而单纯的“一行代码”提示会丢掉其中一个。早期单次生成 benchmark 把 80-94% 当成固定结果；在公平的 agentic baseline 下，那是单任务上限，不是平均值。<a href="benchmarks/results/2026-06-18-agentic.md">完整报告</a> &middot; <a href="benchmarks/">复现实验</a>。</sub>
</p>

<p align="center">
  <sub><a href="README.zh.md">中文</a> &middot; <a href="README.es.md">Español</a></sub>
</p>

---

你见过他。长马尾，椭圆眼镜，在公司待得比版本控制还久。你给他看五十行代码；他看了一眼，什么都没说，然后用一行替换掉。

Ponytail 把他放进你的 AI agent。

## 前后对比

你要求做一个日期选择器。你的 agent 安装 flatpickr，写一个包装组件，加一份样式表，然后开始讨论时区。

启用 ponytail 后：

```html
<!-- ponytail: 浏览器原生就有 -->
<input type="date">
```

更多幸存案例见 [examples/](examples/)。

## 数据

比较诚实的测量方式是让真实 agent 做真实工作：一个无头 Claude Code 会话编辑 [tiangolo 的 full-stack-fastapi-template](https://github.com/fastapi/full-stack-fastapi-template)（真实 FastAPI + React 仓库），用它留下的 `git diff` 评分。12 个功能需求，同一个 agent，分别启用和不启用 skill，n=4，Haiku 4.5。

<p align="center">
  <img src="assets/benchmark-agentic.svg" width="860" alt="Each arm as a percent of the no-skill baseline across LOC, tokens, cost and time (Haiku 4.5). ponytail is lowest on every metric (LOC 46%, tokens 78%, cost 80%, time 73%); caveman rises above 100% on tokens, cost and time; yagni-oneliner LOC 67%. Safety, separate adversarial tier: baseline, caveman and ponytail 100%, yagni-oneliner 95%.">
</p>

| 相比无 skill baseline | LOC | token | 成本 | 时间 | 安全 |
|---|--:|--:|--:|--:|--:|
| **ponytail** | **-54%** | **-22%** | **-20%** | **-27%** | **100%** |
| caveman（简短表达对照组） | -20% | +7% | +3% | +2% | 100% |
| “YAGNI + 一行代码”提示 | -33% | -14% | -21% | -30% | 95% |

ponytail 是唯一一个在所有指标上都下降的方案，也是唯一一个在做到这一点时仍保持完全安全的方案。减少最明显的地方，是存在真实过度构建陷阱的任务（日期选择器从 404 行降到 23 行，颜色选择器从 287 行降到 23 行，因为它会优先使用原生 `<input>` 而不是组件）。对已经足够精简的代码，它几乎不会再减少。完整方法、逐任务表格和限制见：[benchmarks/results/2026-06-18-agentic.md](benchmarks/results/2026-06-18-agentic.md)。

<details>
<summary><strong>较早的单次生成数据（隔离生成）</strong></summary>

五个日常任务，三个模型，三个组（无 skill、[caveman](https://github.com/JuliusBrussee/caveman)、ponytail），每组运行十次，报告中位数。一个提示，一次补全，统计回答里的代码行数：

<p align="center">
  <img src="assets/benchmark-3model.svg" width="860" alt="Median lines of code per arm across Haiku, Sonnet and Opus">
</p>

这组数据展示了 **80-94% 更少代码**。[#126](https://github.com/DietrichGebert/ponytail/issues/126) 公平地指出，裸模型 baseline 会在回答里塞入说明文字和选项，所以这个差距部分来自对话 baseline 的形态。上面的 agentic 数据是修正后更站得住脚的版本。用下面命令复现单次生成测试：

```bash
npx promptfoo eval -c benchmarks/promptfooconfig.yaml
```

</details>

**规则从来不是“token 最少”。** 规则是：只写任务真正需要的东西，并且永远不删掉校验、错误处理、安全和可访问性。代码变小，是因为它必要，不是因为在玩代码高尔夫。成本和延迟下降，只是那些遵守阶梯的模型带来的副作用；如果一个简短风格的推理模型花大量 thinking token 反复琢磨这些台阶，结果也可能反过来（GPT-5.5 上就是这样）。

## 工作方式

写代码之前，agent 停在第一个成立的台阶：

```text
1. 这个东西需要存在吗？     → 不需要：跳过（YAGNI）
2. 标准库能做吗？           → 用标准库
3. 原生平台能力能覆盖吗？   → 用原生能力
4. 已安装依赖能解决吗？     → 用它
5. 一行能解决吗？           → 写一行
6. 只有到这一步：写能工作的最少代码
```

极简，不是失职：信任边界校验、防数据丢失处理、安全和可访问性永远不能被砍掉。

## 安装

ponytail 要求你做的最多也就这些：

Claude Code 和 Codex 插件会运行两个很小的 Node.js 生命周期 hook，所以 `node` 需要在你的 PATH 上（Nix/nvm 用户注意：它必须出现在非交互 shell 的 PATH 里）。如果没有，skills 仍然可用，只是 always-on 激活会安静失效，而不是每次提示都报错。

### Claude Code

```text
/plugin marketplace add DietrichGebert/ponytail
/plugin install ponytail@ponytail
```

桌面 app 没有 `/plugin` 命令。请从 UI 安装：Customize，personal plugins 旁边的 +，Create plugin and add marketplace，Add from repository，然后输入仓库 URL（感谢 @NiklasDHahn，#98）。

### Codex

```bash
codex plugin marketplace add DietrichGebert/ponytail
codex
```

打开 `/plugins`，选择 Ponytail marketplace，然后安装 Ponytail。接着打开 `/hooks`，检查并信任它的两个生命周期 hook，然后开启一个新会话。

同样的安装也适用于 Codex 桌面 app：安装后重启 app，它就会加载插件。

### GitHub Copilot CLI

```bash
copilot plugin marketplace add DietrichGebert/ponytail
copilot plugin install ponytail@ponytail
```

在交互式 Copilot CLI 会话中，使用等价的斜杠命令：

```text
/plugin marketplace add DietrichGebert/ponytail
/plugin install ponytail@ponytail
```

Copilot CLI 会按插件名给插件命令加命名空间。例如：

```text
/ponytail:ponytail ultra
/ponytail:ponytail-review
```

### Pi agent harness

```text
pi install git:github.com/DietrichGebert/ponytail
```

### OpenCode

从这个仓库的 checkout 里运行 OpenCode（插件会复用它的 `hooks/` 和 `skills/`），并在 `opencode.json` 中添加：

```json
{ "plugin": ["./.opencode/plugins/ponytail.mjs"] }
```

它会在每一轮按当前级别注入规则集，并添加 `/ponytail` 命令（见 [命令](#命令)）。OpenCode 也会自动加载本仓库的 `AGENTS.md`，所以即使没有插件，规则也会生效。插件增加了 `lite/full/ultra/off` 级别。

`./` 路径相对于你项目里的 `opencode.json` 解析；如果想在多个项目共享同一个 checkout，请改成 `.mjs` 的绝对路径（它会按自身文件位置找到对应的 `hooks/` 和 `skills/`）。

插件路径会在所有地方加载规则集，但 `/ponytail` 命令是 `.opencode/command/` 里的独立文件，OpenCode 只会从你的项目或全局 commands 目录发现它们。要在这个 checkout 之外使用这些命令，链接一次即可：

```bash
ln -sf /absolute/path/to/ponytail/.opencode/command/* ~/.config/opencode/command/
```

### Gemini CLI

```bash
gemini extensions install https://github.com/DietrichGebert/ponytail
```

它会在每个会话中把规则集作为 always-on context 加载，并注册 `/ponytail` 命令；`skills/` 也会随包分发，在任务需要时激活。

Gemini 适配器有意不提供根目录 `hooks/hooks.json`：Gemini 会自动加载这个路径，而 Ponytail 的生命周期 hook 使用的是 Claude/Codex 事件名。

### Antigravity CLI

Google 正在把 Gemini CLI 改名为 Antigravity CLI（`agy` 二进制）；同一个 extension 也可以在那里安装：

```bash
agy plugin install https://github.com/DietrichGebert/ponytail
```

它复用本仓库的 `gemini-extension.json`。一个区别是：Antigravity 会把 `/ponytail` 命令转换成 skills，所以你要把它们作为聊天消息输入（例如 `/ponytail-review`），而不是从斜杠菜单选择。在迁移完成前（约 2026 年 6 月 18 日），`gemini extensions install` 也仍然可用。若想把它作为 always-on 规则运行，可以把规则集放进 `.agents/rules/`。

### CodeWhale

它会从项目根目录读取 `AGENTS.md`，无需任何设置。把 [`AGENTS.md`](AGENTS.md) 复制到你的项目，或直接从本仓库 checkout 运行 `codewhale`。就这样。

### OpenClaw

```bash
clawhub install ponytail
```

这会从 ClawHub 把 ponytail 安装为 OpenClaw skill；review、audit、debt、gain 和 help skills 的安装方式相同（例如 `clawhub install ponytail-review`）。OpenClaw 会在编码任务中应用它，并把它暴露为 `/ponytail` 命令。没有 ClawHub 时，把 [`.openclaw/skills/ponytail`](.openclaw/skills/) 复制到 `~/.openclaw/skills/`。

就这些。他会感到骄傲。但他不会说。

它会在每个会话中生效，并提供少量命令（见 [命令](#命令)）。当代码库真的伤到你时，可以用 `/ponytail ultra`。启动和模式切换文本会显示当前模式。

可以用 `PONYTAIL_DEFAULT_MODE` 环境变量（`lite`/`full`/`ultra`/`off`）为每个新会话设置默认级别，也可以在 `~/.config/ponytail/config.json`（Windows 上是 `%APPDATA%\ponytail\config.json`）里设置 `defaultMode` 字段。默认值是 `full`。

Cursor、Windsurf、Cline、GitHub Copilot（编辑器）、Aider、Kiro、Zed、CodeWhale：从本仓库复制对应的规则文件（[`.cursor/rules/`](.cursor/rules/)、[`.windsurf/rules/`](.windsurf/rules/)、[`.clinerules/`](.clinerules/)、[`.github/copilot-instructions.md`](.github/copilot-instructions.md)、[`AGENTS.md`](AGENTS.md)、[`.kiro/steering/`](.kiro/steering/)）。

Kiro：把 `.kiro/steering/ponytail.md` 复制到 `~/.kiro/steering/`（全局）或你项目里的 `.kiro/steering/`。

GitHub Copilot CLI fallback（仅指令模式）：它会读取项目里的 `AGENTS.md` 和 `.github/copilot-instructions.md`，或者把规则复制到 `~/.copilot/copilot-instructions.md`，让 ponytail 在每个项目里运行。这个路径会保留 always-on 指令，但不会添加插件模式切换或 hook。

带 Codex 扩展的 VS Code 会读取 `AGENTS.md`，本仓库已经提供该文件，所以从仓库根目录运行无需设置（`~/.codex/AGENTS.md` 可让 Codex 全局生效）。

不同 agent 对应哪些文件：见 [Agent portability](docs/agent-portability.md)。

## 命令

| 命令 | 作用 |
|---------|--------------|
| `/ponytail [lite \| full \| ultra \| off]` | 设置强度，或关闭。无参数时报告当前级别。 |
| `/ponytail-review` | 审查当前 diff 是否过度工程，并给出删除清单。 |
| `/ponytail-audit` | 审计整个仓库里的过度工程，而不仅是当前 diff。 |
| `/ponytail-debt` | 收集你用 `ponytail:` 推迟的捷径，写进台账，避免“以后”变成“永远不会”。 |
| `/ponytail-gain` | 显示 benchmark 测得的影响记分板（更少代码、更低成本、更快速度）。 |
| `/ponytail-help` | 上面这些命令的快速参考。 |

命令需要支持 skill 的 host（Claude Code、Codex、OpenCode、Gemini、pi）。在 Codex 里它们是 skills，用 `@` 调用（例如 `@ponytail-review`）。仅指令适配器（Cursor、Windsurf、Cline、Copilot、Kiro、Antigravity）会加载 always-on 规则集，但不带这些命令。

## 开发

修改紧凑规则文本时，保持各个 agent 副本一致：

```bash
node scripts/check-rule-copies.js
npm test
```

OpenClaw skill 包（`.openclaw/skills/`）由 `skills/` 生成；修改 skill 后重新运行：

```bash
node scripts/build-openclaw-skills.js
```

如果生成结果过期，测试套件会失败。

正确性 benchmark 会启动 Python 做 email 和 CSV 检查；它会先尝试 `python3`，再尝试 `python`。CSV 检查需要本地安装 `pandas`。

## FAQ

**它需要配置文件吗？**

不需要。可选的 `~/.config/ponytail/config.json` 或 `PONYTAIL_DEFAULT_MODE` 环境变量可以设置默认级别，但没有任何必需配置。

**如果我真的需要那个 120 行缓存类呢？**

你不需要。你坚持的话，他还是会写。慢慢地，正确地，一边看着你。

**它能扩展吗？**

你没写的代码可以无限扩展。零 bug，零 CVE，自古以来 100% uptime。

**为什么叫 “ponytail”？**

你很清楚为什么。

## 许可证

[MIT](LICENSE)。能工作的最短许可证。
