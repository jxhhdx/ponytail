# 分组

**任务：** “按某个键把这个对象数组分组。”

## 未启用 Ponytail

```bash
npm install lodash
```

```js
import { groupBy } from "lodash";

const byStatus = groupBy(orders, "status");
// → { pending: [...], shipped: [...], delivered: [...] }
```

或者手写版本：

```js
const byStatus = orders.reduce((acc, order) => {
  (acc[order.status] ??= []).push(order);
  return acc;
}, {});
```

## 启用 Ponytail

```js
// ponytail: Object.groupBy 就是做这个的
const byStatus = Object.groupBy(orders, order => order.status);
// → { pending: [...], shipped: [...], delivered: [...] }
```

**1 个依赖（或一次 reduce）→ 1 个内建 API。** `Object.groupBy` 已在 Chrome 117、Firefox 119、Safari 17.4、Node.js 21 中发布。如果你需要的是 `Map` 而不是普通对象：`Map.groupBy(orders, o => o.status)`。检查你的目标运行时；如果需要 IE11 或旧版 Node，那条 `reduce` 一行写法仍然是正确选择，不是 lodash。
