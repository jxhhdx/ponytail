---
name: ponytail-review
description: >
  专门关注过度工程的代码 review。找出能删除的东西：重新发明标准库、不必要依赖、
  推测性抽象、失效的灵活性。每个发现一行：位置、删什么、用什么替代。
  当用户说“review 过度工程”、“有什么能删”、“这是不是过度工程”、“简化 review”，
  或调用 /ponytail-review 时使用。它补充正确性 review；这个 skill 只猎杀复杂度。
---

审查 diff 中不必要的复杂度。每个发现一行：位置、删什么、用什么替代。一个 diff 最好的结果，是变得更短。

## 格式

`L<line>: <tag> <what>. <replacement>.`，多文件 diff 使用 `<file>:L<line>: ...`。

标签：

- `delete:` 死代码、未使用的灵活性、推测性功能。替代物：无。
- `stdlib:` 手写了标准库已有的东西。说明对应函数。
- `native:` 依赖或代码在做平台原生已经支持的事。说明对应特性。
- `yagni:` 只有一个实现的抽象、没人设置的配置、只有一个调用方的层。
- `shrink:` 同样逻辑，更少行数。展示更短写法。

## 示例

❌ “这个 EmailValidator 类可能比必要的更复杂，你是否考虑过现阶段是否真的需要所有这些校验规则？”

✅ `L12-38: stdlib: 27-line validator class. "@" in email, 1 line, real validation is the confirmation mail.`

✅ `L4: native: moment.js imported for one format call. Intl.DateTimeFormat, 0 deps.`

✅ `repo.py:L88: yagni: AbstractRepository with one implementation. Inline it until a second one exists.`

✅ `L52-71: delete: retry wrapper around an idempotent local call. Nothing replaces it.`

✅ `L30-44: shrink: manual loop builds dict. dict(zip(keys, values)), 1 line.`

## 评分

以唯一重要的指标结尾：`net: -<N> lines possible.`

如果没东西可删，说 `Lean already. Ship.` 然后停止。

## 边界

范围：只看过度工程和复杂度。正确性 bug、安全漏洞和性能问题明确不在范围内；把它们交给普通 review，而不是这里。单个冒烟测试或基于 `assert` 的自检是 ponytail 的最低要求，不是臃肿，永远不要标记为删除。不会应用修复，只列出它们。
说 “stop ponytail-review” 或 “normal mode” 可恢复为详细 review 风格。
