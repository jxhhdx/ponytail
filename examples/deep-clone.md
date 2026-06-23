# 深拷贝

**任务：** “深拷贝这个对象。”

## 未启用 Ponytail

```bash
npm install lodash
```

```js
import { cloneDeep } from "lodash";

const copy = cloneDeep(original);
```

或者经典 hack：

```js
// 脆弱：会丢失 Date、undefined、Map、Set、循环引用、函数
const copy = JSON.parse(JSON.stringify(original));
```

## 启用 Ponytail

```js
// ponytail: structuredClone 就是做这个的
const copy = structuredClone(original);
```

**1 个依赖（或一个脆弱 hack）→ 1 个内建 API。** `structuredClone` 可以处理 `Date`、`Map`、`Set`、`ArrayBuffer`、`RegExp`、循环引用等很多东西，而这些都会被 `JSON.parse/stringify` 静默丢掉。2022 年以来所有浏览器都支持，Node.js 从 v17 起支持。需要 lodash 其他能力时再引入它，不要只为一个函数引入。
